# 扩展以太坊网络:第 2 层入门

> 原文：<https://blog.web3labs.com/scaling-the-ethereum-network-a-primer-on-layer-2>

对区块链，尤其是以太坊最常见的批评之一是它们无法扩展。换句话说，它们的事务吞吐量有限。这可能导致高使用率的破坏性网络拥塞。对此有许多不同的解决方案，本文将介绍一种称为第 2 层解决方案的解决方案。

## **缩放状态**

根据 [Etherscan](https://etherscan.io/) 的说法，在撰写本文时，以太坊网络每秒可以处理 13.2 个事务(TPS)。这可能高于比特币的 5 TPS，但仍然非常低。区块链的变化包括交易速度，分为第 1 层和第 2 层解决方案:

*   **第 1 层**用于描述底层区块链。这意味着要有所作为就意味着要对整个区块链进行彻底改革。目前，以太坊正在转向名为 Eth2 的新版本网络。此外，通过将工作负载分成称为碎片的子区块链，它将有助于提高基本事务吞吐量。然而，预计至少在一两年内不会发生这种情况，在此期间，需要其他解决方案。

*   **第 2 层**指的是上层网络，它改善了区块链的某些方面。可以说，这些解决方案并不属于该方案的一部分——如果您愿意，您可以使用它们，但您没有义务这样做。由于它们不需要改变整个区块链，因此创建起来相对简单。现在，有几个第 2 层解决方案可以解决以太坊的伸缩问题。

在 Eth2 推出之前——甚至可能是之后的一段时间——以太坊网络的用户需要比它目前所能提供的更多的东西。对于许多人来说，网络不仅太慢，而且高使用率会导致更高的交易费用，因为用户试图以更高的价格相互出价，以便他们的交易得到优先处理。第二层解决方案提供了另一种选择:不仅仅是临时的权宜之计，而是提高区块链人生活质量的一种方式。

## **新兴以太坊扩展解决方案**

**![Examples of Ethereum L2 Solutions](img/645231cbc88353e3df3d448b2f3094b2.png)**

现在有许多以太坊的第二层解决方案正在开发中。最有前途的方法之一是卷装。这些是第 2 层解决方案，将资金放在区块链上，但将大部分工作转移到侧链，以减轻主链的工作量。以太坊的联合创始人和发明者 Vitalik Buterin 称 Rollups 是他最喜欢的第 2 层解决方案。两个主要类别是 ZK 汇总和乐观汇总。

而 ZK 汇总使用称为零知识证明的加密概念进行状态转换。然而，这些是复杂的，并且仍然需要大量的工作来应用于整个网络。然而，乐观汇总只是假设状态转换是正确的，而不是每次都检查它。如果不正确，可以恢复数据块，但整个过程要比网络的当前状态快得多。

下面，我们将看看一些现有的扩展解决方案。

*   ****多边形(原 Matic 网络)。****Matic 网络是一个简单的扩展解决方案。在 2021 年 2 月[更名为 Polygon](https://polygontech.medium.com/matic-network-becomes-polygon-ethereums-internet-of-blockchains-expands-mission-and-tech-scope-364932c02cd0) 后，它现在是一个可互操作的区块链扩展框架，用于构建互连以太坊兼容的区块链网络。Polygon 旨在成为一个完整的平台，用于启动可互操作的区块链。它旨在通过利用各种技术，包括 POS 链、等离子体链、ZK 汇总和乐观汇总，克服单个区块链的一些限制，如高费用、可扩展性差和安全性有限。

*   由**物质实验室提供的 zkSync** 。自 2020 年中期以来，该解决方案已经在 mainnet 上运行，并以其低交易成本和无信任协议而自豪。顾名思义，这是一种 ZK 汇总。它的融资轮是由联合广场风险投资(USV)牵头的，值得注意的是对比特币基地和 Dapper Labs 的投资，尽管投资金额没有披露。

*   **乐观网**(原**等离子网**)。Plasma 曾经研究以太坊的可伸缩性，但是随着重新命名为乐观主义，该团队投入到一个可行的可伸缩性解决方案中——一种乐观主义的汇总。预计 mainnet 将于今年 7 月推出，而今年 1 月将开始试运行(有增量版本)。由安德森·霍洛维茨领投的价值 2500 万美元的[首轮融资](https://www.coindesk.com/andreessen-horowitz-leads-25m-round-in-ethereum-scaling-solution)最近结束。

*   **SKALE 网络**。这个项目不是一个汇总——它是自己的技术，称为弹性。它们的链是可配置的和特定于应用的。区块链应用可以租用 SKALE shards 作为以太坊兼容的智能合约平台，吞吐量更好。其 mainnet 于 2020 年 6 月推出，年底前将推出几项预先计划的更新。该项目由阿灵顿 XRP 资本、Blockchange、ConsenSys Labs 等[出资 700 万美元，母公司 SKALE Labs 出资 1000 万美元。](https://www.coindesk.com/ethereum-scaling-project-skale-raises-17-1-million-for-mainnet-launch)

*   Starkware 。另一个 ZK 汇总项目，Starkware 的解决方案被称为 StarkEx，其 mainnet 也在 2020 年年中推出，仅在五个月后就推出了第二个版本。其[最近的 B 轮融资](https://www.theblockcrypto.com/post/99211/starkware-funding-round-ethereum-paradigm)，价值 7500 万美元，由 Paradigm 领投，还有红杉、创始人基金和潘迪拉。

*   **阿兹特克**。Aztec 也是一个基于 ZK 汇总的解决方案，它专注于开放和无许可的区块链网络上的“银行级”隐私。你可以使用可识别的地址别名，类似于用户名，但你的交易活动是加密的——你的交易历史不会向全世界广播。该公司的 210 万美元[融资轮](https://www.coindesk.com/consensys-backs-2-1-million-funding-round-for-ethereum-privacy-startup)由 ConsenSys Labs 牵头，投资者包括 Entrepreneur First、Samos Investments、Jeffrey Tarrant (Mov37)和 Charlie Songhurst。

*   **Loopring** 。Loopring 也是基于 ZK 上卷。他们将以太坊的安全性与低费用和高网络速度结合在一起，吞吐量约为以太坊的 1000 倍，成本仅为其百分之一。除了协议之外，该项目还有一个具备所有这些优势的交换和支付平台。除了其 [2017 年 ICO](https://www.scmp.com/business/article/2109191/blockchain-start-loopring-raises-us45m-ico-regulator-intensifies-scrutiny) ，Loopring 没有其他融资轮次。

*   **POA 网**。POA 是一个开放的以太坊侧链，其前沿是[互操作性](https://medium.com/poa-network/tokenbridge-ethereum-to-binance-chain-interoperability-1c93c399da73)，致力于在不同的智能合约平台之间建立桥梁。它最初是作为自己的区块链，这意味着它不仅仅是侧链技术。该网络使用权威共识算法证明，验证者必须通过完整的 KYC 过程，使其可信。

### **总结**

区块链开发者很早就意识到对更快、更便宜的交易的需求。对于以太坊来说尤其如此，因为它的可编程性允许许多受其当前状态阻碍的实现。虽然这些解决方案中没有一个是现成的，但它们中的许多都可以在某种程度上使用。此外，一旦网络转向其 Eth2 版本，可扩展性问题将得到显著缓解。

您是否希望了解有关部署区块链解决方案的更多信息？您可以在区块链播客([weekinblockchain.com](https://www.weekinblockchain.com/)或您喜欢的播客平台)中查看我们的本周。如果你想现场讨论，我们每周一美国东部时间中午 12 点/格林威治时间下午 5 点在俱乐部会所。期待在那里见到你！