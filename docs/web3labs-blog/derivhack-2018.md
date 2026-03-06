# 德里维克 2018

> 原文：<https://blog.web3labs.com/web3development/derivhack-2018>

上个月，blk.io 的成员参加了首届 [Derivhack 2018 黑客马拉松](https://www.creativeservices.barclays/ehome/barclays-derivhack/invitation/)，该活动由 ISDA、巴克莱、德勤和 Thompson Reuters 在 Rise London 主办。

![Prize Giving DerivHack](img/ec123054453c88d442daae85c39e0c61.png)

*blk . io 团队从巴克莱银行的 Jeremy Wilson 和 Lee Braine 手中领奖*

我们基于自己的 [Epirus 平台](https://medium.com/blk-io/epirus-enterprise-blockchain-platform-979458227a14)赢得了最佳解决方案架构的奖项。获得认可的感觉很棒，尤其是我们是以太坊区块链唯一的团队建筑(几乎其他所有人都在 Corda 上建筑)。

![Pitching Epirus Platform](img/3b27d3997bcf00a25fbb1a5d86ce5e91.png)

*推销我们的平台*

**黑客马拉松**

该活动的目的是利用 ISDA 的公共领域模型和分布式分类帐技术，找到更有效的方法来处理衍生产品。

我们 blk.io 相信，场外交易市场可以从区块链/分布式账本技术中获得巨大收益，这就是为什么我们对这次黑客马拉松感到非常兴奋。

![DerivHack 2018](img/52771a657dc9d71c7f1c3aa780e1551e.png)

*黑客攻击*

对于那些不知道我到底在说什么的人，下面是一个简短的介绍。

**场外交易市场初级读本**

当有人购买或出售一家公司(例如巴克莱银行)的股票时，这是通过证券交易所如伦敦证券交易所进行的。像任何市场一样，交易所将买方和卖方聚集在一起，买方和卖方将以固定的价格购买或出售一定数量的股票。

当你有一个很好理解的、标准化的产品，用少量的属性来描述它，比如一只股票，这是很好的；但有时需要更复杂的产品，由于它们的要求，你不能去一个已建立的交易所购买或出售它们，即它们服务于市场中的特定利基。因此，你可以直接去另一家了解具体情况的银行或金融机构，你可以创造一个更定制的产品，其中有更多关于参与者之间贸易条款的细节。

所有这些金融参与者之间的定制协议构成了场外(OTC)衍生品市场。你可能在金融危机的背景下听说过这个市场——抵押贷款支持证券(MBS)和债务抵押债券(CDO)是场外衍生品的一种。更常见的是，你会听到利率互换(IRS)，这使得拥有固定利率贷款的人可以与他人交换浮动利率，因为他们希望锁定利率，就像房主可以抵押贷款一样。此外，你有信用违约互换，这实际上是一种保险，防止贷款人无力偿还贷款。

这些产品对于想要控制财务风险的企业来说是有用的。然而，金融危机后颁布的一项关键变革是提高透明度。

**ISDA**

进入 ISDA，它是场外交易市场的标准机构，推动了一系列降低风险、增加透明度和改善运营基础设施的计划。

其中一种方法是使用金融产品标记语言( [FpML](http://www.fpml.org/) )。FpML 是一个基于 XML 的模式，用于描述 OTC 产品。它已存在多年，并在整个行业中用于交易后生命周期事件，包括向监管机构报告交易的详细信息。

![Organisation of the FpML Standard](img/c903b982f789d565e09f085184095c1c.png)

FpML 标准的组织(“FpML 规范”)

[http://www.fpml.org/docs/FpML5-at-a-glance.pdf](http://www.fpml.org/docs/FpML5-at-a-glance.pdf)

去年，ISDA 宣布了公共领域模型(CDM ),该模型以衍生品在整个交易生命周期中的交易和管理方式为模型。清洁发展机制的目的是，遵循该机制的参与者将产生相同的结果，帮助标准化参与者产生的数据。

![High-level CDM Concept](img/6034d51da14c1ac49a7861ccf3fd8d02.png)

*高级清洁发展机制概念*

这种标准化数据的概念非常适合智能合约和 DLT，这就是人们对该领域产生浓厚兴趣的原因。

**我们的解决方案**

黑客马拉松的重点是使用 CDM 的一个版本来处理特定的贸易生命周期事件，并探索 DLT。使用我们的 Epirus 平台，它使用 Quorum 作为其底层区块链，我们创建了一个接收交易数据的解决方案，然后创建了一个底层交易的以太坊智能合约表示。

![Simplified solution architecture](img/76fd9f0cda000fb7fd05e8fe856fbc9e.png)

*简化的解决方案架构*

我们的网络配置如下所示，其中有许多代表交易参与者(如银行)的节点，另外还有一个监管节点，该节点参与所有交易。

![Our network deployment](img/9aed9c74e8971fd20bc42e16c6dc3b68.png)

*我们的网络部署*

通过利用 Quorum 的隐私功能以及以太坊智能合约，我们能够提供:

*   参与者之间交易的隐私，不排除监管者
*   跟踪交易历程(历史)以及直接对应于真实基础交易的事件散列
*   加强贸易合同状态逻辑，促进参与者之间的单一真实来源

![Trade contracts were browsable in our Blockchain Explorer](img/6a62a371ef84f292ee0d8250edfb34fd.png)

*贸易合同可以在我们的区块链浏览器中浏览*

我们还接入了 Thompson Reuters 的 BlockOneIQ 服务，以获取一些交易付款的历史利率数据。

对于我们来说，接触 CDM 真的很令人兴奋，我们期待继续使用标准支持的以太坊开源技术，帮助推动衍生品市场与分布式账本技术向前发展。

要了解有关我们的 Epirus 平台的更多信息，以及它如何解决运行和管理您自己的区块链平台的挑战，请[联系我们](https://www.web3labs.com/contact)了解更多信息！

**呼喊**

弗朗西丝和[海伦](https://twitter.com/_helenrobinson_)为组织这次活动所做的出色工作，以及[托尼](https://twitter.com/MargiottaTony)在开始时让每个人都热情高涨！

还要感谢 [Wojtek](https://twitter.com/wojtekhejna) 对 BlockOneIQ 的帮助，最后感谢 [Luke MacGregor](http://www.lukemacgregor.com/) 拍摄的精彩照片记录了这一事件。