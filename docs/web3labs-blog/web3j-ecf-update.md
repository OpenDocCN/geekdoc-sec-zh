# web3j ECF 最新消息

> 原文：<https://blog.web3labs.com/web3development/web3j-ecf-update>

以太坊社区基金(ECF)[最近宣布](https://medium.com/ecf-review/meet-the-grantees-ecf-class-of-2018-part-ii-ff46a284a0b1)资助 [web3j](https://github.com/web3j/web3j) 。在这里，很高兴得到 ECF 的支持，帮助推动图书馆的发展。

![_ECF web3j Blog 2](img/4abc331a57fd807453a6d962f6410853.png)

## 一点历史

早在 2016 年 9 月，我就开始开发 web3j，最初的发布与上海的 Devcon2 相吻合。

![reddit ecf blog](img/ac9c15495349e85c1041619890051931.png)

*The original web3j announcement on Reddit, September 2016*

图书馆社区已经越来越强大，但是在没有任何直接支持的情况下继续发展图书馆总是一个困难的平衡行为。来自 ECF 的资助意义重大，因为它使我们能够在图书馆投入全职资源。

![web3j2018](img/6a1bfca2533fef660362ae42b2263a95.png)

*web3j in 2018*

因此，我们现在的情况是，我们将所有的后端基础设施整合在一起，以支持非常精简的发布流程和常规的发布节奏。在开始一些全新的 web3j 计划之前，我们也将能够开始清理积压的拉取请求和问题！

## 快照构建

我们很高兴地宣布，如果您需要已经合并到库中但尚未发布的功能，现在可以使用快照构建。例如，要使用 web3j 4.0 快照，只需将以下内容添加到 Gradle 构建文件中:

> T2
> 
> ```java
> repositories {
>    maven {
>       url "https://oss.sonatype.org/content/repositories/snapshots"
>    }
> }// ...dependencies {
>     compile "org.web3j:core:4.0.0-SNAPSHOT",
>             // ...
> }
> ```

## web3j 3.6

web3j 3.6 里程碑也于上周发布，你可以阅读更多信息。

## 前进

web3j 的未来是光明的，这个项目现在:

*   在 GitHub 上拥有超过 2000 颗星星
*   每月下载超过 20，000 次(如果算上单个模块的话，超过 100，000 次)
*   其 [Gitter 社区](https://gitter.im/web3j/web3j)拥有超过 600 名成员
*   它是 GitHub 上第二受欢迎的 Java 区块链库——第一名仍然由 [BitcoinJ](https://github.com/bitcoinj/bitcoinj) 占据，考虑到它的年龄和比特币的整体受欢迎程度，这并不令人惊讶。

随着敌无双 4 即将推出，我们又有一个重大消息发布，请继续关注。

### 最后一件事…

我们一直在寻找优秀的 Java/Kotlin/Golang 开发人员加入我们的[blk . io](https://blk.io/)——如果您想聊天，请[联系](mailto:hi@web3labs.com)。