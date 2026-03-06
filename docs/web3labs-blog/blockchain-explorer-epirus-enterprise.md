# Sirato 企业区块链平台

> 原文：<https://blog.web3labs.com/web3development/blockchain-explorer-epirus-enterprise>

在过去的几个月里，blk.io 的团队一直在努力工作。我们一直致力于一系列计划，现在我们准备将这些计划整合在一起，推出我们的企业区块链平台 Sirato。

![Epirus Enterprise Blockchain Platform](img/9efee43ac86a0dd97bea77aa8d0524c0.png)

Sirato 由许多核心组件组成，包括:

*   [法定人数](https://github.com/jpmorganchase/quorum)区块链
*   [症结](https://github.com/blk-io/crux)交易飞地
*   [区块链浏览器](https://github.com/blk-io/blk-explorer-free/)

这得到了其他组件的支持，如 [web3j](https://web3j.io/) 以提供直接的应用程序集成，身份验证服务以根据您组织的策略提供对组件的访问权限，以及最终的本地、云或 SaaS 部署选项。

![Epirus Enterprise Blockchain Platform Blog (2)](img/4c641362e365536d7dbce63e743c40a2.png)

然而，我们想让我们的平台在很多人的手中，所以我们提供了一个免费版本，你可以通过运行:

```java
git clone https://github.com/blk-io/epirus.git
cd epirus
./epirus.sh
```

这将运行一个 4 节点许可的法定网络和我们的区块链浏览器，因此您有一个完整的法定环境来工作。

![4-node permissioned Quorum network, and Epirus Blockchain Explorer](img/cb51738490fdb756422cb633c92bfdcd.png)

一切启动后，浏览器窗口将会打开，您可以使用我们的浏览器直接开始浏览区块链！

免费平台节点端点是固定的，以保持简单，它们在下面和[这里](https://github.com/blk-io/epirus#getting-started)是可用的。

![The 4-node Quorum endpoints](img/e2a7cab57d7dcea7f91ef75657fd7707.png)

*The 4-node Quorum endpoints*

如果您想了解更多信息，请进入 [touch](mailto:hi@blk.io) ，或者前往我们新更新的[网站](https://blk.io/)了解我们的 SaaS 产品。

我们仍在忙于招聘，所以如果你想加入我们的团队，请点击这里查看我们的职位空缺！