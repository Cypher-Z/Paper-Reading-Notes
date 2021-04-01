# Bulletproof - SSL and TLS 

[Last Updated at 2021-02-28]

## SSL, TLS and Cryptograghy

本章主要介绍背景及密码学相关知识。

密码学可以帮助解决重要的三个安全问题：confidentiality（机密性）、authenticity（权威性）、integrity（完整性）。

通常用来衡量加密强度的指标是**key的长度**。而密码的安全性，通常是指的**计算安全性**（computationally secure）。

* **流密码**。是最简单的，密钥最好只用一次。
* **块密码**。和流密码相比，最大的区别是，输入的一个小的变动可以对输出造成相当大的改变。最popular的块密码是AES。快密码还需要输入数据是block的整数倍，所以需要padding，通常是将最后一个byte用来表示padding的长度，而其它需要填充的byte也都填成该长度值（如下图所示）。

![image-20210228185101317](/Users/chris/Documents/E-books/Notes/image/image-20210228185101317.png)

对称加密只能在单独的pair之间进行，如果某个规模巨大的group之间两两都需要通信，对称密钥所需的key的数量是巨大的，于是就引出了：

* **非对称加密**。非对称加密比对称加密的速度要慢很多，因此，通常只将其用于authentication以及negotiation of shared secrets，而对于后续大量数据的传输还是选择使用对称加密。RSA是目前最popular的非对称加密算法。

**数字签名**，是用来进行身份验证的密码学机制。具体操作方式为：

1. 发送方对document进行hash计算，得到固定长度字符串，用私钥进行加密得到签名。
2. 接收方使用同样的hash算法对收到的document进行计算，随后用公钥对接收方发送的签名进行解密，对比两个hash结果是否一致。

⭐️**协议的设计**——如何利用上述套件完成一次安全会话？

假设：Alice，Bob是通话的双方，Mallory是攻击者。

思考：Mallory都有哪些方法破坏安全对话，对应的，Alice和Bob应采用什么方法来防御？

1. 从最简单的角度考虑，如果会话是明文的，Mallory很容易就进行监听，所以，最基础的需求是对会话进行加密（按照之前所述，该步骤使用对称加密）（遗留问题1：对称加密的密钥如何确定？）
2. 即使Mallory无法获取数据包的明文内容，其可以想办法篡改数据包，于是数据包本身需要增加完整性校验，即MAC值。
3. Mallory除了修改数据包之外，还可以直接扔掉数据包、或者是重放数据包，为解决此问题，可以为数据包增添序列号（serial number），以判断是否出现包丢失或者包重放。并且，需要为会话的结束专门设置一个通知信息，以免Mallory提前中断会话。
4. Mallory可能会企图假冒为通信的某一方，所以，在通信开始时，还需要对双方进行身份验证（authentication）（遗留问题2：如何完成身份验证？）
5. 此时，我们来解决两个遗留问题：对于2）身份验证，可以采用公钥密码学的方式解决，即，双方各自产生一个随机数，利用私钥进行加密，交给对方使用公钥解密，完成身份验证；对于1）对称加密的密钥协商，可以利用密钥交换协议，例如Alice生成密钥并用自己的公钥加密发给Bob，此外，还可以使用Diffie-Hellman（DH）协议，尽管开销更大一些，但具有better security properties。

讨论至此，关于Alice和Bob之间应如何完成安全对话，我们已经形成了一个大致的协议：

1. 首先进行身份验证和密钥交换（handshake phase）
2. 其次利用密码学机制完成机密性、完整性得到保证的数据会话（data exchange phase）
3. 最后，通过某个shut down sequence结束会话。

以上也就是high-level的，SSL/TLS协议的雏形。

## TLS1.2

<!--Let's first go through TLS 1.2 to further understand TLS 1.3, which is expected to be widely deployed to replace the former due to real-world demands and the strength of cryptogragy.--> 

Key idea: 没有必要搞清楚其中的每一个feature（time-consumming）。最好的学习方式是通过wireshark观察TLS流量。几个基础的相关权威文档包括：

> [[1] RFC 5246](http://tools.ietf.org/html/rfc5246)
>
> [[2] TLS working group document](https://datatracker.ietf.org/wg/tls/document)
>
> [[3] TLS working group mailing list](https://mailarchive.ietf.org/arch/browse/tls)





