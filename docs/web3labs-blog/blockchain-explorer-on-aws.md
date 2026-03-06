# 探索 AWS 上的区块链

> 原文：<https://blog.web3labs.com/web3development/blockchain-explorer-on-aws>

自从今年早些时候在 Azure Marketplace 上推出 Sirato 区块链探索者以来，许多人问我们它是否会在 AWS Marketplace 上提供。等待终于结束了！我们很高兴地宣布，从今天起，Sirato 可以在 [AWS 市场](https://w3l.cc/aws-offer)买到。

![Epirus Available on AWS Marketplace](img/37a2f8e7c3046dd60677c9fe15333faf.png)

Sirato 区块链浏览器和智能合同注册是企业和技术用户使用区块链网络的首选平台。它支持 Hyperledger Besu，Quorum 和其他基于以太坊的网络。

![Epirus available on AWS marketplace](img/48b955e8ebcf58c235c5b3c3472e8160.png)

Sirato 区块链浏览器和智能合同注册是企业和技术用户使用区块链网络的首选平台。它支持 Hyperledger Besu，Quorum 和其他基于以太坊的网络。

![Epirus available on AWS marketplace](img/ae9fe6834245087fa922ad8790a15756.png)

要在 AWS 上开始使用 Sirato，您可以在这里注册该服务[，并启动一个针对 AWS 优化的 Sirato 实例。您将需要您的节点的 RPC 连接的详细信息。关于这方面的说明，你可以去 Sirato 获取 AWS](https://w3l.cc/aws-offer) [文档](https://docs.epirus.io/getting_started/#aws)。

在 AWS 上，Sirato 提供开箱即用的认证访问，当您设置它时，会提示您输入用户名和密码来控制对实例的访问。这是通过`epirus setup`命令完成的。

```java
$ sudo epirus setup

Configuring Epirus instance

Please enter a username: <enter username>

New password: <enter password>

Re-type new password: <re-enterpassword>

Adding password for user <username>

Please enter node URL: http://<your-service-url>

Successfully connected to http://<your-service-url>

Configuration written to /usr/local/src/epirus/epirus.conf
```

但是，如果您希望在事后添加额外的凭证，您可以使用新的`epirus passwd`命令。

`$ sudo epirus passwd <new or existing username> `

优惠还包括 7 天免费试用，你还在等什么？前往[此处](https://w3l.cc/aws-offer)开始与 Sirato 在 AWS 上合作！