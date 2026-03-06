# 宣布 Web3j 支持 EIP-1559

> 原文：<https://blog.web3labs.com/web3development/announcing-web3j-support-for-eip-1559>

最值得期待的以太坊伦敦叉升级来了！其中包括改进建议 [EIP-1559](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md) 。升级计划在 mainnet 上进行，区块号 [12，965，000](https://etherscan.io/block/countdown/12965000) ，预计 8 月 5 日完成。

以太坊的传统交易模式采用拍卖方式，出价最高的人首先购买有限大小的地块。这种拍卖方式给许多用户带来了麻烦，因为他们经常要等待油价下跌，这可能需要几个小时甚至几天的时间！

这种模式的另一个缺点是，恶意矿商的人为需求造成了高油价。我们也看到网络变得拥挤，2020 年夏天 DeFi 的成功见证了以太坊[天然气价格](https://etherscan.io/chart/gasprice)飙升，有时单笔交易的总费用高达 500 美元。

EIP-1559 旨在通过提高交易费用的可预测性来改进这一旧模式，并通过算法来计算费用，而不是基于矿商的拍卖系统。为了实现这一点，协议引入了两个关键变化:

*   一种称为基本费用的交易市场价格:一种已知的天然气价格，有助于用户估计天然气价格，避免支付过多费用。额外的提示可以提供给矿工，以优先考虑你的交易。

*   动态块大小:现在块大小与以前一样有限制(15M 气体大小)，但具有增加到 30M 气体大小的灵活性。

[Web3j 4.8.7](https://github.com/web3j/web3j/releases/tag/v4.8.7) 版本包括支持新事务类型的更改，增加了 JSON RPC API 所需的字段。

下一节是如何用 Web3j 发送 EIP-1559 的演练。

## 带 Web3j 的 EIP-1559

开始使用 Maven 向项目添加库:-

```java
<dependency>

  <groupId>org.web3j</groupId>

  <artifactId>core</artifactId>

  <version>4.8.7</version>

</dependency>
```

或使用 Gradle:-

```java
dependencies {

   implementation "org.web3j:core:4.8.7",

          ……

}
```

配置 http 服务和凭据:-

```java
Web3j web3 = Web3j.build(new HttpService());

Credentials credentials = 

WalletUtils.loadCredentials("password", "/path/to/walletfile");
```

使用[传输类](https://github.com/web3j/web3j/blob/master/core/src/main/java/org/web3j/tx/Transfer.java)发送带有 EIP-1559 新字段的以太网:-

```java
TransactionReceipt transactionReceipt = Transfer.sendFundsEIP1559(

        web3j, credentials,

        "0x<address>|<ensName>", //toAddress

        BigDecimal.ONE.valueOf(1), //value

        Convert.Unit.ETHER, //unit

        BigInteger.valueOf(8_000_000), gasLimit

        DefaultGasProvider.GAS_LIMIT,//maxPriorityFeePerGas

        BigInteger.valueOf(3_100_000_000L)//maxFeePerGas

  ).send();
```

通过 TransactionReceipt 访问新属性 effectiveGasPrice 和 type

```java
transactionReceipt.getType();

transactionReceipt.effectiveGasPrice();
```

使用 Web3j 获取块并访问新字段:-

```java
  EthBlock.Block block = web3j.ethGetBlockByNumber(

            DefaultBlockParameter.valueOf(“<block number>”),

            true  //returnFullTransactionObjects

            ).send().get().getBlock();

  block.getBaseFeePerGas();
```

其他选择得到块或得到叔叔喜欢:-

```java
EthBlock.Block block = web3j.ethGetBlockByHash(String blockHash, boolean returnFullTransactionObjects);

EthBlock.Block block = web3j.ethGetUncleByBlockHashAndIndex(String blockHash, BigInteger transactionIndex);

EthBlock.Block block = web3j.ethGetUncleByBlockNumberAndIndex(

DefaultBlockParameter defaultBlockParameter, BigInteger transactionIndex);
```

通过 Web3j 访问交易新字段:-

```java
EthTransaction ethTransaction = web3j.ethGetTransactionByHash(String transactionHash).send();

EthTransaction ethTransaction =   web3j.ethGetTransactionByBlockHashAndIndex(String blockHash,        BigInteger transactionIndex).send();

EthTransaction ethTransaction =     web3j.ethGetTransactionByBlockNumberAndIndex(DefaultBlockParameter defaultBlockParameter, BigInteger transactionIndex).send();
```

访问新交易:-

```java
ethTransaction .getTransaction().get().getType();

ethTransaction .getTransaction().get().getMaxFeePerGas();

ethTransaction .getTransaction().get().getMaxPriorityFeePerGas();

ethTransaction.getTransaction().get().getAccessList();
```

更多详情请参考 web3j [文档](http://docs.web3j.io/)。

EIP-1559 被认为是以太坊最重要的改进方案之一。预计这将对用户体验和网络经济产生重大影响。燃气费可能会减少 90%，燃烧机制据说会使以太坊通货紧缩。

随着以太网从工作验证走向利益验证，这一升级对于网络安全和能源消耗也很重要。要了解更多信息，请阅读我们的[博客，这里的](/blockchain-myths-energy-consumption)解释了被称为工作证明及其竞争对手利益证明的共识算法。

感谢所有为 Web3j 做出贡献和帮助 EIP-1559 发展的人。我们鼓励所有用户和开发者为我们的 Web3j github [repo](https://github.com/web3j/web3j) 做出贡献。

你对 EIP-1559 有什么看法？请在评论中告诉我们！与此同时，为了及时了解区块链的所有趋势，请通过( [watch)收听或观看本周的区块链。](https://www.youtube.com/playlist?list=PLqTg_CuyCcDgeuG5NNXivJdg80nxqyb1f))[weekinblockchain.com](https://www.weekinblockchain.com/)，或者通过你喜欢的播客平台。