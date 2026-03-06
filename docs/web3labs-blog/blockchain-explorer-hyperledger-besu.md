# 如何在 Hyperledger Besu 上探索区块链

> 原文：<https://blog.web3labs.com/web3development/blockchain-explorer-hyperledger-besu>

自从我们创建了 Sirato 区块链浏览器，我们希望确保它与几个不同的企业以太坊(当然还有普通以太坊)区块链客户端无缝地工作。

直到最近，Quorum 还是我们完全支持的唯一一款企业客户端及其隐私功能。我们很高兴地说，从 2019 年 9 月开始，Sirato 也支持 [Hyperledger Besu](https://www.hyperledger.org/projects/besu) (以前的 Pantheon)项目。

![Epirus Supports Hyperledger Besu](img/c384a428019d5cabc8b8ead2c62860c4.png)

我们很高兴在 Hyperledger 中看到 Besu。Web3 实验室的团队已经为 Besu 贡献了一段时间，致力于他们的隐私功能，并确保在 [Web3j](http://web3j.io/) 中的一流支持。

我们试图让我们的用户尽可能简单——Sirato 开箱即用 Besu 或 Quorum——当你启动它时，你不需要告诉它你正在使用哪个。如果您想检查它，只需运行:

```java
git clone https://github.com/blk-io/epirus-free.git

cd epirus-free

NODE_ENDPOINT=http://<node_endpoint> docker-compose up
```

或者，如果你想在 Azure 上运行完整版，你可以点击[这里](https://w3l.cc/medium-azure)开始。

如果您想查看交易隐私支持，请找到您的区块链节点参与的合同或交易。

![Epirus Enterprise Blockchain Explorer](img/4ce3781128b578e20dc5a8159a52ad05.png)

然后，当您在 Sirato 中打开特定的事务时，您将能够查看该事务的输入字节码。

![Epirus Enterprise Blockchain Explorer](img/4c979c6c4146a321775fcfce86335e66.png)

当然，如果 Besu 节点不参与私人事务，则该信息将不可用。

目前就这些，请继续关注 Sirato 和 Web3j 的更多重大更新。