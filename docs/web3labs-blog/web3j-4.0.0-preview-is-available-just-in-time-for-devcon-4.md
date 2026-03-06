# web3j-4.0.0 预览版现已推出，正好赶上敌无双 4

> 原文：<https://blog.web3labs.com/web3development/web3j-4.0.0-preview-is-available-just-in-time-for-devcon-4>

At [blk.io](http://blk.io/) we’ve had some exciting times recently; our team has been growing in order to bring to fruition our plans for web3j. ![Web3j - Where Java meets the Blockchain](img/801d1f955638c11cf788fc03ded3b1f7.png)

本月早些时候，我们发布了带有一些关键修复的 web3j 3.6.0，现在我们很高兴地宣布发布 4.0.0-alpha-1，包括内部和社区贡献。

4.0.x 仍在积极开发中，但我们认为我们会给你一个阿尔法版本的味道。请继续关注，我们不打算让它成为阿尔法太久！关于这个版本的全部细节可以在下面找到。

像去年一样 [blk.io](http://blk.io/) 将会在敌无双 4，所以如果你想聊天，请[给我们发消息](https://www.web3labs.com/contact)。

## 反应流 2.0

此次发布的两个主要特性中的第一个是从 RxJava 1.0 升级到 RxJava 2.0，关于这两个版本之间主要差异的讨论可以在[这里](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)找到。

这对 web3j 意味着什么？名字中包含 Observable 的所有方法都被重命名为使用 Flowable。

我们还清理了受此次 API 升级影响的冗长的方法名称。

举个例子，

![method names](img/562fbf3c45a49e734cad4ef2190ead74.png)

已被重命名为

replayPastAndFutureTransactionsFlowable

更多信息请参考 [Web3jRx](https://github.com/web3j/web3j/blob/release/4.0/core/src/main/java/org/web3j/protocol/rx/Web3jRx.java) 接口。

完整的变更集可以在 [#753](https://github.com/web3j/web3j/pull/753) 找到，感谢 [@vpriscan](https://github.com/vpriscan) 的宝贵贡献。

## BIP44 实施

这个版本的第二个头条特性是 BIP44 实现，这要感谢[@ AmiMobilechain](https://github.com/AmiMobilechain)；完整的变更集可以在 [#686](https://github.com/web3j/web3j/pull/686) 找到。

该规范建立在 BIP43 中描述的方案和 BIP32 中的算法的基础上，为钱包提供了一个逻辑层次结构，从而能够从一个种子生成层次结构中的密钥。这样做有很多好处，比如易于维护和密钥再生。更多信息请参考 BIP44 [规范](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)和 [Bip44WalletUtils](https://github.com/web3j/web3j/blob/release/4.0/crypto/src/main/java/org/web3j/crypto/Bip44WalletUtils.java) 源代码。

## 各种其他修复和改进

同样重要的是我们希望在每个版本中包含的各种修复和改进。一些突出的修复包括:

*   由 [@franz 对可靠性代码生成器(](https://github.com/franz-see) [#672](https://github.com/web3j/web3j/pull/672) )的改进——见
*   [@咏春](https://github.com/iongchun)对折旧方法( [#508](https://github.com/web3j/web3j/pull/508) )的清理
*   由 [@serso](https://github.com/serso) 改进助记符( [#665](https://github.com/web3j/web3j/pull/665) )的实现
*   [@taivokasper](https://github.com/taivokasper) 更新 CLI 用法字符串( [#660](https://github.com/web3j/web3j/pull/660) )
*   由 [@gaiazov](https://github.com/gaiazov) 更新了 OkHttp CipherSuites 以包含所有 in fura cipher suites([# 757](https://github.com/web3j/web3j/pull/757))
*   删除*。 [@yuriymyronovyc](https://github.com/yuriymyronovych) 生成合同包装( [#408](https://github.com/web3j/web3j/pull/408) )的 bin 文件要求
*   由 [@ricmoo](https://github.com/ricmoo) 根据 eth_sign ( [#650](https://github.com/web3j/web3j/pull/650) )对消息进行前缀签名

如果我错过了你对列表的贡献，请接受我的道歉——拉请求和问题是从用户那里获得关于库的反馈的主要机制之一，所以我感谢人们花时间提交它们。

有关其他变更的详细信息，请参考[发布页面](https://github.com/web3j/web3j/releases/tag/v4.0.0-alpha-1)。

## 我要做什么呢？

对于几乎所有的用例，如果你以前使用的是 web3j core，那么在 4.0 core 中你应该不会有任何问题。

除了按照上面的 RxJava 2.0 小节在方法调用之间进行迁移以使用更新的函数名之外，您还需要重新生成您的智能契约包装器以使用这个版本。

### 下一步是什么？

我们已经为即将到来的 4.x 版本制定了一些令人兴奋的计划，包括各种社区驱动的[拉请求](https://github.com/web3j/web3j/pulls)，构建和流程改进，以及 [blk.io](http://blk.io/) 贡献。

对于 5.x 和更高版本，我们将寻求精简我们的库，并使用 [Java 9 的流程](https://docs.oracle.com/javase/9/docs/api/index.html?java/util/concurrent/Flow.html)。

我们正在积极开发 web3j，请关注此空间，了解更多令人兴奋的消息！