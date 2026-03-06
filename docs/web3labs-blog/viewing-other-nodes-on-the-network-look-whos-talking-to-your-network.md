# 查看网络上的其他节点

> 原文：<https://blog.web3labs.com/web3development/viewing-other-nodes-on-the-network-look-whos-talking-to-your-network>

我们倾向于想当然地认为区块链网络就在那里，但是你有没有想过你的节点还在和谁说话呢？有一个可以连接的节点当然很好，但是您如何知道您的节点正在与可靠的来源对话呢？虽然很容易忽略这个细节，但潜在的影响是严重的。

您如何知道节点的正确数量是多少？

![Viewing other nodes Blog ](img/e8db0357a0f66531eed82bb0777c8c7c.png)

## 概观

以太坊是一个去中心化的应用平台；它是由许多节点组成的分布式非结构化对等系统。因为它是非结构化的，这意味着每个节点都必须拥有网络上所有数据的完整副本。这也意味着一个节点将充当网络的服务器(对其他对等体)和客户端(从对等体获取数据)。

因为每个节点都必须获取该区块链的所有数据，这意味着它容易受到恶意行为者的攻击，这些恶意行为者可能会向您传播损坏的数据。为了缓解这个问题，中本聪(在比特币白皮书中)提出，可以通过引入“工作证明”共识算法来解决这个问题。

![Ethereum Nodes](img/5e0905062c51552fc91f326aac835384.png)谷歌地球上的以太坊节点来自 [@peter_szilagyi](https://twitter.com/peter_szilagyi)

绿色= geth，橙色=奇偶校验，白色=其他

## 这和对等计算有什么关系？

简单来说，你需要有同行，才能有人脉。否则，你只是在自言自语！您可以运行一个只有一个连接到更大网络的对等节点的节点。缺点是你必须完全信任那个人。在另一个极端，你可以尝试连接到所有可用的对等点，问题是它不能扩展。你不能及时和每个人交谈。你需要的是一个合理数量的对等体，其中一半以上是恶意的可能性很低。

## 你如何防止自己被劫持？

你发现同伴的方式需要随机。问题是你需要从某个地方开始，以太坊 mainnet 通过 bootnodes 来实现。它们由以太坊基金会维护，并作为对其他节点的查找。一旦你有了一些同伴，你可以要求更多的同伴(邻居)。您的客户端可以在网络中漫游，找到足够随机的对等点进行连接。

## 检查你的同伴

确保您所连接的对等点分布充分是一个好主意。连接到属于同一演员的 15 个对等点是没有意义的。没有真正万无一失的方法，但是你可以从 geth 控制台检查`admin.peers`或者使用 [epirus](https://docs.epirus.io/features/#peers) 来开始。

现在你知道了，下次你第一次连接到一个节点时，也许从检查它的对等点开始。如果您看到至少 15 个不同的对等点，您应该可以确信您的节点正在与真实的网络对话！

### 参考资料:

[https://github.com/ethereum/devp2p/blob/master/rlpx.md](https://github.com/ethereum/devp2p/blob/master/rlpx.md)

[https://github.com/ethereum/devp2p/wiki/Discovery-Overview](https://github.com/ethereum/devp2p/wiki/Discovery-Overview)

[https://medium . com/loom-network/understanding-区块链-基础-第一部分-拜占庭-容错-245f46fe8419](https://medium.com/loom-network/understanding-blockchain-fundamentals-part-1-byzantine-fault-tolerance-245f46fe8419)