# 宣布我们的区块链浏览器

> 原文：<https://blog.web3labs.com/web3development/announcing-our-blockchain-explorer>

blk.io 这几个月一直很忙，支持里程碑式的 web3j3 . x 版本，并构建了我们的第一个企业产品。

我们在与私人以太坊或 [Quorum](https://github.com/jpmorganchase/quorum/) 区块链网络合作时一直面临的一个挑战是缺乏像[以太扫描](https://etherscan.io/)这样的工具来查看区块链发生了什么。

我们决定建造一个区块链探索者来解决这个缺点。

## blk.io 浏览器

我们的 block Explorer 支持以太坊和法定网络，并提供搜索和查看最近的块、交易和部署的智能合同的能力。

![Blockchain Explorer](img/bce5c7c4ba5fb8f10a66fbdde9c83a17.png)

*Latest blocks view*

## 快的

它的响应速度非常快，是大规模事件——我们已经在跨数百万个数据块的数百万次交易的实时网络上进行了测试。

## 简单

我们尽可能保持浏览器体验的简单性，包括将其与您的网络相集成。启动并运行它所需要的只是您想要同步的节点的详细信息，剩下的就交给 Explorer 了。

## 交易隐私

Quorum 的交易隐私也得到了很好的迎合。当遇到私有事务时，Explorer 会将其标记为私有，并根据需要从 Quorum enclave 请求事务的详细信息，从而保持您的数据私有。

![Blockchain Explorer](img/ac77dc3b78c264e94e06de141068497d.png)

*Viewing a private transaction in Quorum*

## 智能合同注册

使用智能合约时的一个挑战是，当您与它们交互时，理解它们正在执行什么操作。在区块链上创建[日志事件是记录智能合约中状态变化的推荐方法。然而，弄清楚这些事件所代表的动作是很重要的。](https://media.consensys.net/technical-introduction-to-events-and-logs-in-ethereum-a074d65dd61e)

为了解决这个问题，我们的 Explorer 提供了一个合同注册表。使用此注册表，您可以将有关智能合约的信息与浏览器相关联，因此当您查看与事务相关联的事件时，可以看到被调用事件的详细信息及其参数值。

例如，在下面的例子中，我们演示了查看一个 [ERC20 传输](https://theethereum.wiki/w/index.php/ERC20_Token_Standard#The_ERC20_Token_Standard_Interface)事件。

![Viewing an ERC20 Transfer event](img/de5a84bd8cc7bab2f0dc435b81d99770.png)

ERC20 Transfer event

### 下一步是什么

我们仍在大力投资开发我们的浏览器，我们还在开发更多的功能。最终，我们希望达到这样一种境界，即用户认为包含在其中的信息不像一个区块链，而更像是它所支持的底层业务流程。

我们正在开发一个免费的浏览器版本，将在未来几周内发布。与此同时，如果你有一个私人以太坊或定额区块链网络，你想使用探索者，请[联系](mailto:hi@web3labs.com)了解更多信息。