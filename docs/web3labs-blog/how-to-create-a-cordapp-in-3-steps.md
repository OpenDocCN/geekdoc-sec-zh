# 如何通过 3 个步骤创建 CorDapp

> 原文：<https://blog.web3labs.com/web3development/how-to-create-a-cordapp-in-3-steps>

当你开始写你的第一个 CorDapp 时，这是第一次有意义的工作。有许多移动部分和概念需要学习，其中一些可能是完全陌生的，如分布式系统和密码学。

在 Web3 实验室，我们努力让开发者的体验尽可能的流畅。通过利用多年来为[以太坊](https://www.web3labs.com/web3j)构建开发者工具的经验，我们现在已经将我们的专业知识应用于 Corda，创建了 web3j-corda。web3j-corda 极大地简化了开始创建第一个 CorDapp 的步骤。

在不到 30 分钟的时间里，这个博客将带你了解你第一个 corDapp 建立和运行之前需要知道的一切。

## 步骤 1:安装

首先，我们需要确保安装了所有先决条件，它们是:

*   JDK8
*   码头工人

然后让我们使用

```java
curl -L https://getcorda.web3j.io | bash
```

如果您在 windows 上运行:

```java
Set-ExecutionPolicy Bypass -Scope Process -Force;iex ((New-Object 
System.Net.WebClient).DownloadString(‘https://raw.githubusercontent.
com/web3j/corda/master/scripts/win_installer.ps1'))
```

这将把最新版本的 web3j-corda 安装到$HOME/.web3j-corda 目录中。它将被直接添加到您的$PATH 中。因此，下次您启动终端时，它将立即可供您使用。但是如果您想在当前终端中使用它，只需运行

source $ HOME/. web3j _ corda/source . sh

## 步骤 2:创建你的 CorDapp

现在让我们使用 web3j-corda 创建我们的第一个 CorDapp。为此，我们必须运行新命令。这为您创建了一个模板 CorDapp。以下是需要的不同选项:

1.  -n —我们要创建的 cordapp 的名称

2.  -o —我们希望生成 cordapp 的输出目录

3.  -p—corda PP 客户端的包名

让我们运行以下代码:

```java
web3j-corda new -n test -o ~/clients/corda/test/t1 -p org.example
```

这将使用给定的输入生成一个 Gradle 项目。然后，我们可以在我们选择的 IDE 中打开创建的项目。

![Gradle project](img/c4c9ff184e29e534a12a2dc7912eaa8a.png)

*Generated Gradle project with 3 modules.*

该项目包含 3 个主要模块:

1.  合同:它定义了 CorDapp 的合同有效性。你可以在 R3 的技术文档中了解更多信息。[状态](https://docs.corda.net/key-concepts-states.html)是在同一个模块中创建的，它们是存储在总账中的实际信息。
2.  工作流:流程支持自动同意分类帐状态。详细文档可在 [Corda](https://docs.corda.net/key-concepts-flows.html) 网站上找到。
3.  客户端:客户端定义可用于在生产环境中与 CorDapp 交互的类。

所有上述模块都已经由 web3j-corda 预定义，因此它可以开箱即用，无需对单个类或方法进行任何更改。

对于上面的例子，我们已经生成了 CorDapp 的所有代码，包括默认的合同、状态和工作流。我们甚至定义了可以使用的客户端，我们可以与定义的 CorDapp 进行交互。

![Generated client with the REST API annotations](img/8dad779a46cbe202b2375f949b579665.png)

*The generated client with the REST API annotations*

生成的客户端代码用 REST API 定义进行了注释，这样就可以用来连接到 Corda 网络。

我们已经定义了启动工作流模块中定义的流程所需的所有接口。客户端是由 Corda 应用程序公开的 REST 端点的门面。您还可以找到所有已定义的流以及该流各自的输入和输出。

![Generated client list](img/a8cc06963a3d6a7e968337a4341773ec.png)

*The generated client test*

我们使用 docker 定义了一个测试网络来启动一个 party 节点。和一个[网络图](https://docs.corda.net/network-map.html)来创建一个兼容区域。此外，我们为每个 party 节点公开了一个 [braid](https://gitlab.com/bluebank/braid) 服务器，它公开了 OpenAPI 定义，帮助我们通过 REST 端点进行通信。

## 步骤 3:运行 CorDapp

既然我们理解了生成的一切，我们就有了第一个符合我们需求的 CorDapp。我们可以通过运行来运行测试。/gradlew test(或通过使用 Intellij IDEA)来验证网络的启动并启动一个流—它对该流进行最基本的调用并验证输出。

确保您已经安装并运行了 docker。在这里的测试中，我们将启动一个 party 节点。启动节点、使用网络图验证证书并启动网络需要一些时间。

![Successful execution of the workflow test](img/8e720146b04b3b63d74e6831b8b2d0e5.png)

*Successful execution of the workflow test being run.*

我们已经开始测试了。万岁，你也有了你的第一个 corDapp！如果使用正确的工具，使用 Corda 提高工作效率并不困难。我们演示的三个步骤将让您理解核心概念，最重要的是为在 Corda 平台上高效工作提供正确的基础。

享受您的 Corda 之旅，下次再见，您可以使用 web3j-corda 做更多的事情。