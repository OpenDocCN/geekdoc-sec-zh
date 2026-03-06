# web3j 3.1.0 版本中的 ENS 和 Truffle 支持！

> 原文：<https://blog.web3labs.com/web3development/ens-and-truffle-support-in-web3j-3.1.0-release>

几周前，在我飞往[敌无双 3](https://ethereumfoundation.org/devcon3/) 的航班上(顺便说一下，这真是太棒了)，我有幸坐在了[尼克·强森](https://twitter.com/nicksdjohnson)的旁边，他是[以太坊域名服务(ENS)](https://ens.domains/) 的创始人。

考虑到起飞后我们都拿出了笔记本电脑，屏幕上显示着代码，很明显我们要去坎昆，并开始互相谈论我们的项目。不久前，我看到 Nick 在伦敦以太坊会议上谈论 ENS，并在我的待办事项列表中的 [web3j](https://github.com/web3j/web3j) 中与它集成。然而，鉴于我有尼克坐在我旁边，在我面前有 9 个小时的飞行，这是一个没有大脑的工作就开始了！

## 实体

web3j 中的 ENS 支持意味着，以前在任何需要使用合同或钱包地址的地方，现在都可以使用 ENS 域名。

例如，对于您的智能合同，您可以使用:

```java
YourSmartContract contract = YourSmartContract.load(
        "<name>.ens", 
        web3j, credentials, GAS_PRICE, GAS_LIMIT);
```

web3j 的[命令行工具](https://docs.web3j.io/command_line.html)也支持它:

```java
$ web3j wallet send <walletfile> <name>.ens
```

web3j 实现提供符合 [UTS #46](http://unicode.org/reports/tr46/) 的输入标准化和验证。Nick 坚持认为任何 ENS 实现都会这样做:)。

## 块菌支架

对于 JavaScript 和 Solidity 开发者来说， [Truffle 框架](http://truffleframework.com/)是一个非常受欢迎的选择。Truffle 在 JSON 中提供了一个[契约模式](https://github.com/trufflesuite/truffle-contract-schema)来表示部署到各种以太坊网络的契约。

web3j 现在可以使用 Truffle JSON 文件来生成智能契约包装器。这意味着，如果您目前或已经使用 Truffle 进行智能合约开发，您可以从它们生成 Java 智能合约包装器。这种方法的优点是，您的智能合约将知道它们以前部署在哪里。

```java
YourSmartContract contract = YourSmartContract.load(
 YourSmartContract.getDeployedAddress(<network id>), 
        web3j, credentials, GAS_PRICE, GAS_LIMIT);
```

其中网络 id 根据 [EIP 155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md#list-of-chain-ids) 确定。

为了生成包装文件， [web3j 命令行工具](https://docs.web3j.io/smart_contracts.html#smart-contract-wrappers)有一个新的*松露*参数:

$ web3j **松露**生成[-Java types |-solidity types]
/path/to/<松露-智能-契约-输出>。JSON-o
/path/to/src/main/Java-p com . your . organization . name

Ezra Epstein 应该为这一改变受到赞扬，因为他有这个想法并实施了它。

## 其他变化

有关此版本的其他修复的详细信息，请参考 [v3.1.0 发布页面](https://github.com/web3j/web3j/releases/tag/v3.1.0)。

## 你用的是 web3j 吗？

如果您正在使用 web3j，并且乐意将您的项目或公司添加到 web3j 的[用户列表中，请让我们](https://github.com/web3j/web3j#companies-using-web3j)[知道](mailto:hi@blk.io)！