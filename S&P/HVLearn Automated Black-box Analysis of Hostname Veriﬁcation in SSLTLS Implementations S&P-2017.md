### HVLearn: Automated Black-box Analysis of Hostname Veriﬁcation in SSL/TLS Implementations S&P-2017

### 内容摘要

HVLearn - 基于Automata Learning的黑盒差分测试模型。Hostname verification是证书验证的重要环节之一。然而各证书验证应用的实现可能与RFC规定有所差异，且有些差异会导致安全问题。

本文的核心思想是：Application实现的hostname verification过程应该都是模版化（正则化）的表达式规则，所以可以通过Automata Learning的方式对其实现的表达式进行学习（得到某个DFA表示），再通过与RFC标准进行比较以及差分分析，发现application在hostname verification过程中的不合规。

本文的主要贡献可概括为：

- 首次通过学习host verification的DFA模型对其安全性进行分析

- 做了许多domain-specific的优化（不是简单地应用算法和模型）

- 对6个popular的library和2个应用做了测试，代码覆盖率比已有的黑盒/灰盒fuzzing平均提高了11.21%，并且找到了8个从未被发现过的RFC rule violation

### 背景补充

关于Automata Learning：已有诸多算法可以运用，可在state数量指数倍的时间内，通过黑盒请求有效地学习出DFA[1]。

Hostname需要符合严格的格式规范，X.509证书中有两个字段可能用来验证hostname，一个是Subject中的CN（格式要求较为宽泛），一个是SAN 扩展（其中不同类型的entity有不同的格式规范要求）。

RFC标准中关于hostname verification的规则（人工总结）包括：

- Matching order：SAN优先级高于CN

- Wildcard：*出现的位置、匹配的规则等有诸多规范要求

- IDN-related：IDN自身有一定的要求，比如需要先转换成xn--开头的punycode再验证；其次，IDN在CN和在SAN中各有一定允许的字符集范围（SAN的范围相对更窄）和长度要求；IDN与wildcard结合的情况没有特别具体的要求， 但是RFC不推荐对wildcard出现在IDN中的情况进行匹配

- IP：长度、编码格式，不允许wildcard

- Email-related：当IDN、wildcard、IP等出现在email address中时各自有一些处理规范

- Other：还有URI等格式，但是application一般不支持验证

### Methodology概述

为了能够更广泛地检测（不同编程语言）的工具，选择了黑盒测试的方式（而不是基于白盒做代码分析）。

如何根据学习的结果发现不合规：

1. RFC的部分rule可直接以正则表达式表示，由此HVLearn可直接对单个application是否符合规范进行检查。

2. 此外，还有很多corner case是RFC很难直接通过明确规则说清楚的，而这种corner case往往会出现问题（不能忽略）。于是，作者还对不同的application实现存在差异的部分进行差分分析，以更全面地找到安全问题。

### Automata Learning Algorithm

**Learning model**: 采用的是一种主动学习（Active Learning）的模型，i.e., exact learning from queries. 

**学习过程**：目标是学习一个target system（理论上应该是个DFA model），方式是向该system发送membership query，得到accept或者reject的响应；学习一段时间后得到某个model M，再向equivalence oracle发送请求，问M是不是对的，要么得到肯定的结论，要么得到某个counter-example（model M和要学习 的target model在该example上结果不一致）；学习算法会根据counter-example继续调整model M；该过程持续迭代至model M得到equivalence oracle的肯定。

![img](https://api2.mubu.com/v3/document_image/0c6a97d6-eb54-4008-911c-67b011beb6d9-1707530.jpg)

**Automata learning in practice**：

- 算法选择：本文采用的是Kearns-Vazirani (KV)算法[2]进行主动学习

- 构建equivalence oracle：由于本文均为黑盒检测，测试者不可能建立完备的oracle。因此，作者选用了Wp-method[3]算法对oracle进行近似模拟。该方法的input是一个超参数depth parameter，i.e., upper bound of the number of states of target model，以及学习算法得到的DFA model M，随后Wp-method会生成一些测试input并对照model M 和 target system的返回结果

- 具体地，本工作使用了Learnlib项目[4]（包含了KV和Wp）



### HVLEARN 的具体工作流程

**Overview** ：如下图所示。基本流程为，针对RFC的某个rule，生成某种test certificate template case，作为输入问implementation 该hostname是否与test case match，得到是或者否的结论；想要学习的就是implementation针对此rule的regular expression（用DFA表示）。

![img](https://api2.mubu.com/v3/document_image/8ea5ed8e-f5ea-44bb-9ac8-c8d23e26e3f8-1707530.jpg)

**Generating certificate templates**: 共设计了23个test样例（基于人工总结的RFC规则），使用GnuTLS生成自签名的test case（原因是GnuTLS对于Subject以及SAN内容的定制支持性比较好）。

**Performing membership queries**: 即实现Learnlib与待测试的implementation之间的查询接口函数。由于Learnlib基于java而implementation可能是C/C++/Python，此处使用了JNI（Java Native Interface）技术进行交互。

**Automata learning parameters and optimizations**:

- **Alphabet size**. 控制字符集的范围是很重要的，因为这直接决定了要测试的状态数，若字符集太大则overhead过于严重，不可行。作者采取的方式是选取representive的一个set（13个字符），包含了大小写字母、数字、wildcard、-、\s、Null、不可打印的unicode字符以及某些特殊字符，基本是最小程度地cover所有可能的测试范围

- **Caching membership queries**. 为了避免在迭代学习过程中重复地用同一个input向test model发起query，作者利用了Learnlib的DFALearningCache来缓存查询结果（能够较好地提升equivalence query check过程的效率）

- **Optimizing equivalence queries**. 大概意思是，若完全不干预该过程（不做任何优化），大概率开始时生成的是一个reject everything的DFA，拉低模型效率。所以本工作优化了该过程：通过一系列的规则首先把一些能够被accept的hostname放进种子集合

**Analysis and comparison of inferred DFA models**:（如何根据HVLearn的学习结果判断implementation的合规性）

- **Analyzing a single DFA model**. 判断单个模型是否合规，即将其与RFC规则进行对比。有两种方法：
  - 如果RFC规则可以将accepted hostname写成regular expression，HVLearn就可以自动完成对比
  - 否则，HVLearn会遍历生成模型所有accept的domain集合，由研究人员人工完成判断

* **Comparing unique differences between DFA models**. 比较不同的生成模型间的差异，也就是比较DFA的差异，有算法可以直接利用，本文采用的算法见[5]

**Specification extraction**: RFC并没有（也不可能）对所有case都给出明确的accepted set，也就是有vague的部分。本文采取的方法是，对于这些vague case，取所有implementation生成的DFA的交集作为formal specification。此处运用了已有的DFA交集计算算法[6] 注意，虽然计算多个DFA的交集的时间复杂度很高，好在本文生成出的这些DFA差异不是特别大，时间复杂度在可以接受的范围之内

### Evaluation

被测试的implementation列举如下：

![img](https://api2.mubu.com/v3/document_image/8d6ae3f8-20f1-4bab-926b-89a35bba8882-1707530.jpg)

HVLearn找到的violation（为单个model分析和差分分析的结果综合）如下：

![img](https://api2.mubu.com/v3/document_image/8ac0d16d-2caa-42db-b348-e0b549b82786-1707530.jpg)



值得一提的是，作者还比较了HVLearn和黑盒fuzzing（随机生成字符串，自己实现）以及Coverage-guided fuzzing（同样为自己实现）在同等Query数量下，对于测试的implementation用于hostname的function的代码覆盖率。代码覆盖率的计算由工具Gcov完成。

其余Evaluation就是对参数选择、优化效率的提升等等做分析。

### Related work

提到Automata Learning还被用来做过银行卡、电子护照、TLS协议、TCP/IP实现的问题发现，找机会看一看。

### My Comment

1. 本文采用的自动化测试方法，即，对于某个可表达为DFA的规则实现，利用Automata Learning学习自动机、并对学到的自动机模型进行分析，是非常值得学习和借鉴的。
2. 几种自动测试方法：Fuzzing test, differential test，以及Automata Learning，各自有何优缺点、适用于哪些场景？需要进一步讨论学习。
3. 测试对象：为何选择了certificate 验证过程中hostname verification进行研究？（balance，finding和难度如何把握）以及，针对协议的自动化测试，和这种针对验证过程的自动化测试有何异同？之后再进行总结讨论。

### 参考文献

[1] Angluin, Dana. "Learning regular sets from queries and counterexamples." Information and computation 75.2 (1987): 87-106.

[2] Kearns, Michael J., Umesh Virkumar Vazirani, and Umesh Vazirani. An introduction to computational learning theory. MIT press, 1994.

[3] Khendek, Fujiwara Bochmann, et al. "Test selection based on finite state models." IEEE Transactions on software engineering 17.591-603 (1991): 10-1109.

[4] Raffelt, Harald, Bernhard Steffen, and Therese Berg. "Learnlib: A library for automata learning and experimentation." Proceedings of the 10th international workshop on Formal methods for industrial critical systems. 2005.

[5] Argyros, George, et al. "Sfadiff: Automated evasion attacks and fingerprinting using black-box differential automata learning." Proceedings of the 2016 ACM SIGSAC conference on computer and communications security. 2016.

[6] Sipser, Michael. "Introduction to the Theory of Computation." ACM Sigact News 27.1 (1996): 27-29.