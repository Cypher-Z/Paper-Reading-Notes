### Discussion of Ethics in Internet Measurement Paper



#### The Menlo Report - Ethical Principles Guiding Information and Communication Technology Research



**Four principle guidelines**

- **Respect for persons** 参与者必须是自愿的，拥有决定权；注意对非自愿参与的人是否产生影响。
- **Informed consent** 向用户提供consent，说明研究的内容、可能的负面影响，让用户来选择。不能用利益来诱惑受试者同意。
  - Notice，comprehension，voluntariness 要clearly地说明所有细节，内容要易于理解，强调自愿参与
  - Post hoc notification & debriefing 如果实验中有欺骗用户的行为（网络犯罪相关研究中很常见），需要做事后解释，这很重要。
- **Beneficence** 最大化实验的正面作用，最小化实验的负面影响。
  - Identification of potential benefits and harms.
  - Mitigation of Realized Harms. 需要告知affected parties并采取mitigation actions；需要考虑到最坏的状况，并做好相应的准备。
- **Justice** 受益面、负担等都要对所有人是equally的。
- **Respect for Law and Public Interest** 遵守法律，公开方法和结果，对自己的行为负责。

注意降低披露漏洞的负面影响（例如，不要给malicious actors利用论文内容进行攻击的机会）。

如果研究人员无法直接获得用户的知情同意，network operator以及platform provider也许可以代为提供。



#### Addressing Ethical Considerations in Network Measurement Papers

目前网络测量领域的文章缺少主动描述ethics consideration的习惯，这不仅对论文评审造成困扰，更重要的是，也许作者本身是做了ethics考虑的，但没有写到文章里，那么后人在复现论文tech的时候可能就忽略了这些actions，从而导致出现道德问题。所以，只要论文是产生human impact的，就应该有ethics consideration的相关内容。

Ethics主要包含两方面：

- 以人为本的价值观。包括考虑人的生命、健康、安全、幸福。
- 计算机/软件行业的职业道德规范。

本文关注的是那些会对人（无论是直接还是间接）产生影响的ethics问题。（例如，占用带宽过多之类的问题本文是不涉及的，除非其将会以某种形式直接对human being造成影响）。

网络测量的数据分成两种：

1. When and how long two parties communicate. 或者扩展来看，可以用metadata来描述。
2. The contents of the conversation. 涉及到**交流的具体内容**，和第一类数据相比，此类数据在privacy方面具有更高的要求（根据美国的法律规定）。

**Fact**：metadata 与 content之间的界限越来越模糊（因为根据metadata推测content的技术在不断发展），e.g.，某些情况下可以直接根据header推测加密通信的具体内容。



- **A spectrum of harm**. Harm是很难进行准确定义的，一般都是“光谱状”。即使如此，光谱的两端（最好的情况和最差的情况）依旧能够帮助对该工作的harm进行评估。
- **Indirect Harm**. 即使harm是indirect的，也要进行讨论。
- **Potential Harm**. 有时实验并没有造成harm（实际上），而只是有潜在的可能性。



数据收集的ethics问题：

被动收集的数据一定是与道德问题无关吗？不是的。考虑如下两个case：

1）一个public的数据集，即使其收集方式是不太道德的（比如compromised）

2）一个non-public的数据集，但是以某种方式被researcher获取了

以上两种情况，后者将面临更严峻的道德问题，因为事实是：**如果不是因为这项工作，公众是无法获知关于该数据集的信息的（而且获知这些信息对human being可能是harmful的）。**



关于公开数据集：

只要对数据做了“匿名化”处理就可以公开数据集吗？不是的。如今人们“deanonymization”的技术越来越强了，例如1980s年代的数据包，由于其采用的password和加密算法都弱，也许现在就可以破解了。

不能要求研究者彻底解决这个问题，但是研究者必须考虑这个问题可能引发的相关风险。



几个基本ethics问题，供参考：

1. 如果data是由author直接收集的，收集该数据会对任何人造成任何harm的影响吗？如果有，讨论可以缓解该harm的方法。
2. 如果数据不是直接由author收集的，该收集过程的道德问题有在其他任何地方（例如发表的文章、公开的技术报告等）中讨论过吗？ 如果讨论过，请提供citation。如果没有，则作者需要在本文中既讨论数据收集的ethics的问题，也需要讨论使用该数据的道德问题（尤其是针对non-public dataset）。
3. 利用当前的技术，本研究使用的数据能否披露有关个人的隐私信息？如果有，请讨论为了保护数据被不正当地披露或者误用而采取的措施。
4. 讨论任何没有被以上三个问题cover到，但是对当前工作其效果的具体的ethics问题。