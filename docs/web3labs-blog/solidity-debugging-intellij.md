# Solidity 调试器来到 IntelliJ

> 原文：<https://blog.web3labs.com/web3development/solidity-debugging-intellij>

合著者:[克里斯蒂安·费尔德](/web3development/author/christian-felde)

许多年前，作为一名开发人员需要在卡片上打孔。这个过程既困难又乏味。后来情况有所好转，你可以开始直接在电脑上输入代码了。但是您仍然需要依靠打印语句和日志输出来理解发生了什么，如果您没有或不能添加输出文本，这并不总是可能的。

在 Web3 实验室，我们一直关注 Java 和 JVM 开发人员，从我们的 Web3j 开源库开始。作为现代主流语言的开发人员，我们总是被奇妙的工具和开发人员体验所宠坏。我们已经能够连接到正在运行的进程，检查虚拟机的每个部分，并确切地看到代码在做什么。

进行 Solidity smart 契约开发感觉有点像回到过去，回到那些工具已经从我们身边移走的地方。即使与智能合同执行相关的一切都可以在公共、私有或本地区块链节点上获得，我们还没有工具来利用这一点。很长一段时间，我们甚至不能在 Solidity 中轻松地进行任何形式的登录。

我们在 Web3 实验室的部分使命是通过提供最好的 Web3 技术和服务，让开发者和企业加速采用分散式系统。因此，对我们来说，投资构建一个可靠性调试器是非常有意义的，这也是我们现在自豪地宣布的。

现在，在[https://github.com/web3j/intellij-solidity-debugger](https://github.com/web3j/intellij-solidity-debugger)可用的，是我们 IntelliJ 的 Solidity 调试器插件。如果您是 Java 开发人员，您可能已经熟悉 IntelliJ 及其为 Java 和 JVM 调试提供的出色工具。有了这个插件，类似的功能现在可以在 IDE 中设置断点、检查内存和遍历 Solidity 智能契约代码。所有这些都是通过我们的调试器插件在 IntelliJ 中集成体验的一部分。

我们结束了吗？不，还可以做更多的事情，但是我们认为它已经达到了一个成熟的水平，对我们来说把它作为开源软件与更广泛的社区共享是有意义的。

我们将感谢您的反馈，并帮助推进此事！

![IntelliJ Solidity Debugger in action](img/66e53456922b16e654ffc375132d3e76.png)