# 密钥交换和 DHKE

> 原文：[`cryptobook.nakov.com/key-exchange`](https://cryptobook.nakov.com/key-exchange)

在密码学中，[**密钥建立**](http://cacr.uwaterloo.ca/hac/about/chap12.pdf)（**密钥交换**，**密钥协商**）是一个过程或协议，通过这个过程或协议，两个参与者可以获取一个**共享密钥**，用于后续的加密使用，通常用于加密通信。建立技术可以是**密钥协商**或**密钥传输**方案。

+   在**密钥协商**方案中，双方都参与共享密钥的协商。密钥协商方案的例子包括 Diffie-Hellman（**DHKE**）和椭圆曲线 Diffie-Hellman（**ECDH**）。

+   在**密钥传输**方案中，只有一方参与共享密钥，而另一方从它那里获取密钥。密钥传输方案通常通过**公钥密码学**实现，例如在**RSA 密钥交换**中，客户端使用其私钥加密一个随机会话密钥，并将其发送到服务器，在那里使用客户端的公钥进行解密。

按设计，**密钥交换**方案在两个参与者之间安全地交换加密密钥，使得其他人无法获得密钥的副本。通常，在加密对话开始时（例如，在**TLS 握手**阶段），参与者首先协商要用于对话的加密密钥（共享密钥）。**密钥交换方案**是现代密码学中的一个重要主题，因为密钥在互联网中的数百万台设备和服务器之间交换了数百次。

每当笔记本电脑连接到 Wi-Fi 网络或通过`https://`协议打开网站时，都会执行一个**密钥协商**（**密钥建立**）方案。密钥协商可以基于匿名密钥交换协议（如 DHKE）、密码或预共享密钥（PSK）、数字证书或许多元素的组合。一些通信协议只建立一次共享密钥，而其他则随着时间的推移不断更改密钥。

**认证密钥交换**（AKE）是在密钥交换协议中交换会话密钥，同时也**验证参与方的身份**（例如，通过密码、公钥或数字证书）。例如，如果您连接到受密码保护的 Wi-Fi 网络，通常使用认证密钥协商协议，在大多数情况下是**密码认证密钥协商**（PAKE）。如果您连接到公共 Wi-Fi 网络，则进行**匿名密钥协商**。

## [](#密钥交换-密钥协商算法)密钥交换/密钥协商算法

存在许多用于密钥交换和密钥建立的**加密算法**。一些使用公钥密码系统，其他使用简单的密钥交换方案（如迪菲-赫尔曼密钥交换），一些涉及服务器身份验证，一些涉及客户端身份验证，一些使用密码，一些使用数字证书或其他身份验证机制。

密钥交换方案的例子包括：[**迪菲-赫尔曼密钥交换**（Diffie–Hellman key exchange，DHКЕ）](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange)、[**椭圆曲线迪菲-赫尔曼**（Elliptic-curve Diffie–Hellman，ECDH）](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie–Hellman)、[**RSA-OAEP**](https://en.wikipedia.org/wiki/Optimal_asymmetric_encryption_padding)、[**RSA-KEM**](https://tools.ietf.org/html/rfc5990)（RSA 密钥传输）、[**预共享密钥**（PSK）](https://en.wikipedia.org/wiki/Pre-shared_key)、[**安全远程密码协议**（SRP）](https://en.wikipedia.org/wiki/Secure_Remote_Password_protocol)、[**FHMQV**](https://www.cryptopp.com/wiki/Fully_Hashed_Menezes-Qu-Vanstone)（完全哈希的 Menezes-Qu-Vanstone）、[**ECMQV**](https://www.cryptopp.com/wiki/Elliptic_Curve_Menezes-Qu-Vanstone)（椭圆曲线 Menezes-Qu-Vanstone）和[**CECPQ1**](https://en.wikipedia.org/wiki/CECPQ1)（量子安全密钥协议）。

让我们从经典的**迪菲-赫尔曼密钥交换**（Diffie–Hellman Key Exchange，DHКЕ）方案开始，这是最早的公钥协议之一。
