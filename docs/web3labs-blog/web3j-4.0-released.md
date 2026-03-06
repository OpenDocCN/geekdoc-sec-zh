# web3j 4.0 发布

> 原文：<https://blog.web3labs.com/web3development/web3j-4.0-released>

基于我们社区的出色工作，我们从 alpha 中取出了 web3j-4.0.0，并发布了完整版。

您可能还记得 [web3j-4.0.0-alpha](https://medium.com/blk-io/web3j-4-0-0-preview-is-available-just-in-time-for-devcon-4-b4f37067a541) 是在 Devcon4 发布时发布的，为我们的社区提供了一些受欢迎的功能，例如 Reactive Streams 2.0，一个 BIP44 实现，加上对 PegaSys 的 [Pantheon 客户端](https://github.com/PegaSysEng/pantheon)的支持以及许多其他简洁的变化。

 ![Rx Java 2](img/706a06d74ae0b7e9102373c14006e35c.png)

*web3j is now using Rx Java 2*

## 变更集

除了 alpha 版本变更集之外，web3j-4.0.3 还包括以下内容

*   实体代码生成的改进: [#672](https://github.com/web3j/web3j/pull/672)
*   移除不推荐使用的方法: [#508](https://github.com/web3j/web3j/pull/508)
*   改进助记符的实现: [#665](https://github.com/web3j/web3j/pull/665)
*   CLI 用法字符串更新: [#660](https://github.com/web3j/web3j/pull/660)
*   更新了 OkHttp CipherSuites 以包括所有的 in fura cipher suites:[# 757](https://github.com/web3j/web3j/pull/757)
*   删除*。合同包装生成的 bin 文件要求: [#408](https://github.com/web3j/web3j/pull/408)
*   根据 eth_sign 前缀消息签名: [#761](https://github.com/web3j/web3j/pull/761)
*   支持特定于万神殿的 RPC 调用 [#767](https://github.com/web3j/web3j/pull/767) ， [#792](https://github.com/web3j/web3j/pull/792)
*   异常记录和格式 [#746](https://github.com/web3j/web3j/pull/746) 、 [#784](https://github.com/web3j/web3j/pull/784) 、 [#785](https://github.com/web3j/web3j/pull/785)
*   无主题日志的 NPE[# 708](https://github.com/web3j/web3j/pull/708)
*   设置监听器之前遇到错误时的 NPE[# 731](https://github.com/web3j/web3j/pull/731)
*   允许使用 Java 关键字作为可靠性函数名 [#776](https://github.com/web3j/web3j/pull/776)
*   添加对明文 HTTP 通信的支持 [#718](https://github.com/web3j/web3j/issues/718)
*   发布/订阅 [#458](https://github.com/web3j/web3j/pull/458) — [#768](https://github.com/web3j/web3j/pull/768) 的文档
*   修复节点断开时的无效索引错误 [#713](https://github.com/web3j/web3j/pull/713)
*   减少“加密”模块对第三方库的依赖(简化 Android 集成) [#618](https://github.com/web3j/web3j/pull/618)
*   改变一些方法的可见性，以简化硬件集成

感谢我们所有的社区和贡献者让这一切成为可能。

## 我要做什么呢？

如果您以前升级到 4.0.0-alpha，那么这个过程很简单，不需要任何更改；只需在 web3j-4.0.3 上添加一个依赖即可。

如果您使用的是 3.6 或稍旧的版本，那么除了按照 RxJava 2.0 部分迁移方法调用以使用更新的函数名之外，您还需要重新生成您的智能契约包装器以使用该版本；更多细节请参考关于 4.0.0-alpha 版本的博客文章。

## 下一步是什么？

我们已经为即将到来的 4.x 版本制定了一些令人兴奋的计划，包括各种社区驱动的[拉请求](https://github.com/web3j/web3j/pulls)和 [blk.io](http://blk.io/) 贡献。

我们也在与我们的社区合作，将 web3j 的 Android 版本升级到 4.0。

对于 5.x 和更高版本，我们将寻求精简我们的库，并使用 [Java 9 的流程](https://docs.oracle.com/javase/9/docs/api/index.html?java/util/concurrent/Flow.html)。

### 谢谢

非常感谢[以太坊社区基金](https://ecf.network/)让这个正在进行的开发得以发生，感谢 [Gitcoin](https://gitcoin.co/profile/web3j) 提供发行奖金。

敬请关注，更多精彩消息即将发布！