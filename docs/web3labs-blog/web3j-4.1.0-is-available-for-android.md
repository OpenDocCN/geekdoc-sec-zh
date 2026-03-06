# web3j 4.1.0 适用于 Android

> 原文：<https://blog.web3labs.com/web3development/web3j-4.1.0-is-available-for-android>

在 blk.io，我们一直在努力确保与以太坊合作的 Android 开发者拥有最新、最棒的 [web3j](https://github.com/web3j) 功能，这是一个挑战。无论是开发新功能还是修复错误，人们都需要知道如何编写能够在所有支持的平台上工作的代码。

上一次发布的 web3j 的 android 特定版本是 web3j-3.3.1-android，从那以后，我们有了许多版本，其中包含一些我们的社区渴望在 android 上使用的出色功能和关键修复。

大多数最近的 web3j 贡献大量使用了 Java 8 的特性和构造，比如 Streams 和 CompletableFutures，但是我们仍然希望支持运行 Android 4.0.3(冰激凌三明治，api 级别 15)的设备，所以使用 Java 8 并不容易。

![web3j ](img/801d1f955638c11cf788fc03ded3b1f7.png)

进入 [Gitcoin](https://gitcoin.co/) ，他慷慨地支持我们用[赏金](https://github.com/web3j/web3j/issues/769)进行必要的改变，将最新版本的 web3j 带到 android 4.0.3。

感谢 [Sergey](https://github.com/serso) 为交付这个优秀的[作品](https://github.com/web3j/web3j/pull/809)所做的辛勤工作。因此，我们为那些希望在 android 上预览最新 web3j 的人提供了一个快照。

## 我需要做什么？

把你对 web3j 的依赖更新到 *4.1.0-android-SNAPSHOT* 瞧。

请注意，web3j-android 自上一个版本以来已经有了很大的发展，所以随着 API 的变化，您必须对代码进行一些修改。看看我们之前的文章。

### 下一步是什么？

我们将很快发布一个 4.1.x 版本，其中包含一些其他修复和功能，但与此同时，我们希望听到您关于 web3j Android 冒险的消息，[联系](mailto:hi@web3labs.com)！