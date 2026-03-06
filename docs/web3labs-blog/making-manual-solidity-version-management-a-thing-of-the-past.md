# 让手动可靠性版本管理成为过去

> 原文：<https://blog.web3labs.com/web3development/making-manual-solidity-version-management-a-thing-of-the-past>

当你是一名区块链开发者时，管理 Solidity 的安装很糟糕。事情不应该是这样的，所以我们决定做点什么。任何一个称职的区块链图书馆都会承担许多与智能合约集成的重担。在 [Web3j SDK](https://www.web3labs.com/web3j) 中，当您创建一个新项目时，您的智能合约代码会在构建时自动为您编译，并生成用于部署和处理该智能合约的 JVM 绑定。为了确保最终用户体验尽可能无缝，我们发现自己不断地从一个版本的 Solidity 切换到另一个版本，然后又是另一个版本。

任何一个称职的区块链图书馆都会承担许多与智能合约集成的重担。在 [Web3j SDK](https://www.web3labs.com/web3j) 中，当您创建一个新项目时，您的智能合约代码会在构建时自动为您编译，并生成用于部署和处理该智能合约的 JVM 绑定。为了确保最终用户体验尽可能无缝，我们发现自己不断地从一个版本的 Solidity 切换到另一个版本，然后又是另一个版本。

## 不要被落下

Solidity 的发布速度非常快，在一个版本下可能有效的东西在另一个版本下可能无效。有时我们希望使用 Solidity 0.5，有时我们开始运行新发布的 Solidity 0.6 的新项目，有时我们支持仍然使用 0.4 的遗留项目。在我们的开发机器上，在这些之间切换通常是一个相当烦人的过程，包括处理多个本机安装，或者为每个项目使用 dockerized Solidity 版本。

在 Web3j 中，我们也需要针对最终用户选择的任何版本的 Solidity 编译智能合约的能力。以前，我们通过使用单一的 Solidity 版本来支持这样做，并允许最终用户使用不同的版本，如果他们需要的话，通过重新配置我们的编译器插件来使用本地可执行文件。这是功能性的，但是对于新用户来说很困难，并且导致了我们的问题跟踪器的相当大的混乱。

## 要是有一个图书馆就好了

进入 [Sokt](https://github.com/web3j/web3j-sokt) ，这是我们的新库，用于管理多个本地 Solidity 安装，并以编程方式编译几乎任何 Solidity 文件。给 Sokt 任何一个 Solidity 文件，它都会自动决定编译它的最佳 Solidity 版本。如果没有安装该版本，将为当前平台(Linux、macOS 或 Windows)下载一个 solc 二进制文件——不需要从源代码编译，也不依赖 Docker。Sokt 然后可以在你的源文件上调用 Solidity，创建任何指定的输出。下面的示例演示了如何使用 Sokt 来编译智能协定:

```java
val fileName = “<path to your solidity file>”

println("sokt Processing $fileName")

val solidityFile = SolidityFile(fileName)

 

println("Resolving compiler version for $fileName")

val compilerInstance = solidityFile.getCompilerInstance()

 

println("Resolved ${compilerInstance.solcRelease.version} for $fileName")

 

val result = compilerInstance.execute(

    SolcArguments.OUTPUT_DIR.param { "/tmp" },

    SolcArguments.AST,

    SolcArguments.BIN,

    SolcArguments.OVERWRITE

)

 

println("Solc exited with code: ${result.exitCode}")

println("Solc standard output:\n${result.stdOut}")

println("Solc standard error:\n${result.stdErr}")
```

Sokt 是用 Kotlin 编写的，可以与任何 Java、Kotlin 或 Groovy 项目一起工作。它现在也与 Web3j 完全集成(通过 [Gradle 插件](https://github.com/web3j/solidity-gradle-plugin))，这导致使用该库时更好的兼容性和开发者体验。 [Sokt 现在也是一个已发布的工件](https://mvnrepository.com/artifact/org.web3j/web3j-sokt/0.1.0)——所以它也可以在你的项目中使用，只需添加一个 Gradle 依赖项:

```java
compile group: 'org.web3j', name: 'web3j-sokt', version: '0.1.0'
```

通过利用 Sokt 的强大功能，您不再需要自己管理 Solidity 安装。希望这能让你有机会用宝贵的时间做其他事情，而不是在你意识到电脑从你的道路上拿起了错误版本的 Solidity 时咒骂你的电脑。