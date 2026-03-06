# 正好赶上敌无双，web3j 3.0！

> 原文：<https://blog.web3labs.com/web3development/just-in-time-for-devcon-web3j-3.0>

经过几个月的开发， [web3j](https://github.com/web3j/web3j) 3.0 里程碑版本已经发布！这个版本提供了许多增强功能，旨在使使用 JVM 语言的以太坊和智能合约变得更加容易。

下面是所有伟大的新功能的完整纲要。

blk.io 将参加 Devcon3，所以如果你想聊天，请[给我们发消息](http://hi@web3labs.com/)。

## 模块化的

web3j 现在是一个模块化项目。对于每个 web3j 版本，都会发布一些工件:

*   实用程序-实用程序类的最小集合
*   rlp —递归长度前缀(rlp)编码器
*   abi —应用二进制接口(abi)编码器
*   核心——很像前面的 web3j 核心工件，没有代码生成器
*   加密-以太坊加密库
*   geth — Geth 特定的 JSON-RPC 模块
*   奇偶校验—特定于奇偶校验的 JSON-RPC 模块
*   元组——简单元组库
*   codegen —代码生成器
*   infura — Infura 特定的 HTTP 头支持

这里的动机是给开发者(尤其是 Android)更多的选择来决定他们想要使用库的哪一部分。我还希望这将鼓励那些基于 JVM 的开发人员在整个以太坊生态系统中实现标准化和重用，这样这些库就不会在不同的项目中一次又一次地被重新实现。

对于几乎所有的用例，如果你以前使用的是 web3j core，那么在 3.0 core 中你应该不会有任何问题。在幕后，有一些重构，但大部分相同的功能仍然存在。您将需要重新生成您的智能协定包装器以使用此版本。

## 智能契约包装器中的本机 Java 类型

web3j 的智能契约包装器现在默认使用原生 Java 类型。这意味着下面的 Java 到 Solidity 类型的转换在后台进行:

布尔-> bool
big integer->uint/int
byte[]->bytes
String->String 和地址类型
List < > - >动态/静态数组

这大大减少了您编写的代码量，但仍然在幕后提供了与 Java 中的原生可靠性类型相同的保护。

## 更新了远程调用的 API

web3j 以前在其返回未来的智能契约包装器上提供了异步方法。现在返回了一个新的 RemoteCall 类型，这使得您可以很容易地选择是同步地、异步地(通过 Java 中的 CompletableFuture 或 Android 中的 Future)还是可观察地(Observable)对以太坊客户端进行远程调用。

如前所述，这确实意味着您需要重新生成您的智能合约包装器来使用 web3j 3.0，但是好处是您有了一个更干净的 API，并且可以利用 3.0 版本中的许多智能合约包装器增强功能。

## 迁移到 OkHttp

web3j 现在使用优秀的 [OkHttp](https://github.com/square/okhttp) 库进行所有的 Http 通信。

## 奇偶校验和 Geth JSON-RPC 支持

所有特定于 Geth 和奇偶校验的个人模块调用现在都可以在 web3j 中使用。此外，web3j 还支持奇偶校验跟踪模块。

感谢 [@iikirilov](https://github.com/iikirilov) 提交此变更。

## 智能协定中多个返回值的元组

web3j 现在提供了一个简单的元组类型，它由返回多个值的智能契约包装器使用。以前，如果方法返回了不同的类型，列表就需要转换为正确的类型。

## 交易收据处理器

web3j 提供了一个处理以太坊事务的抽象层，当您向网络提交一个事务时，它会不断地轮询您的以太坊节点以获得一个事务收据，一旦有了收据，就将它返回给调用者，表明该事务已经被挖掘并放入网络上的一个块中。

在 3.0 版本中，您可以修改 web3j 轮询事务的方式。默认行为仍然保持不变。然而，如果您想要处理大量事务，web3j 提供了一个队列，可以用来轮询这些事务，从而减少库创建的线程数量。

你可以在这里阅读我们的[文档](https://docs.web3j.io/4.8.7/transactions/transactions_and_smart_contracts/)。

## 通过第三方签署交易

交易签名逻辑已经重构，允许通过第三方进行交易签名(以前必须在 web3j 中执行)。这意味着，如果你想从 web3j 呼叫 HSM 或钱包，你可以。

感谢 [@eztierney](https://github.com/eztierney) 提交此变更。

## 文档更新

所有 web3j 文档都已更新，以反映对库的更改。如果您发现一个断开的链接或任何差异，请提出问题，或者更好地创建一个 PR:)详细说明什么是错误的。我想保持这个库的文档的高标准(已经在这方面投入了大量的资金)，所以任何反馈都会受到感激。

## 其他变化

*   被 [@dmitrychaban](https://github.com/dmitrychaban) 移除了 Scrypt 依赖( [#165](https://github.com/web3j/web3j/pull/165) )
*   支持[@马文鹏](https://github.com/mawenpeng)在 HttpService ( [#200](https://github.com/web3j/web3j/pull/200) )中提供自定义头
*   Android 上潜在的 SecureRandom 漏洞( [#146](https://github.com/web3j/web3j/issues/146) )突出显示 [@ligi](https://github.com/ligi)
*   更好的静态数组支持( [#183](https://github.com/web3j/web3j/pull/183) )由 [@jaycarey](https://github.com/jaycarey)
*   由 [@the-wastl](https://github.com/the-wastl) 将历史过滤器限制为 LogFilter ( [#154](https://github.com/web3j/web3j/pull/154) )
*   由 [@IgorPerikov](https://github.com/igorperikov) 在请求( [#210](https://github.com/web3j/web3j/pull/210) )上增加 id 字段
*   由 [@iikirilov](https://github.com/iikirilov) 允许距离可观测量发射一个值( [#184](https://github.com/web3j/web3j/pull/184)
*   删除 [@mochalovv](https://github.com/mochalovv) 安装和卸载过滤器的重复调用( [#212](https://github.com/web3j/web3j/pull/212)
*   如果关联的方法包含 payable 修饰符，智能协定包装器只需要 Ether 参数的规范(以前必须提供零值)
*   链条 Id 已符合 [EIP-155 规格](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md)
*   添加了事务实用程序方法来生成事务哈希

如果我错过了你对列表的贡献，请接受我的道歉——拉请求和问题是从用户那里获得关于库的反馈的主要机制之一，所以我感谢人们花时间提交它们。

有关其他变更的详细信息，请参考[发布页面](https://github.com/web3j/web3j/releases/tag/v3.0.1)。