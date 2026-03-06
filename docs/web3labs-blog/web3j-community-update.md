# Web3j 社区更新

> 原文：<https://blog.web3labs.com/web3development/web3j-community-update>

我们已经花了 2020 年的大部分时间在 Web3j 上持续投资，以改善 Java 和 Android 开发人员的以太坊开发体验。这一直是我非常感兴趣的一个领域，我们很幸运地从以太坊基金会获得了资金来帮助推进这项工作。

在这篇文章中，我将提供我们在过去几个月中所做的所有令人敬畏的事情的更新！

## Web3j 文档更新

我们花时间浏览了 Web3j 文档，对其进行了重组，使其尽可能对用户友好。我们经常问自己的一个问题是，如何确保那些刚接触 Web3j 的人在开始使用时有最好的体验。关于以太坊，人们需要学习很多不同的概念，以及如何使用 Web3j。

我们现在添加了一个新的[快速启动](https://docs.web3j.io/quickstart/)页面，它由一个[入门](https://docs.web3j.io/getting_started/run_node_locally/)部分提供支持，允许新用户浏览许多核心概念的流程，如运行节点、部署智能合同并与之交互以及处理事件。

我们还添加了涵盖其他补充 Web3j 项目的页面，包括 [Web3j OpenAPI](https://docs.web3j.io/web3j_openapi/) 、Web3j Unit、Web3j EVM、Sokt 以及 Gradle 和 Maven 构建插件。更复杂的主题现在包含在它们自己的类别中，涵盖了隐私技术等概念和一些其他以太坊概念，如 ENS、RLP 编码和管理 API。

为了补充文档更新，我们还开始发布 Web3j [Javadocs](https://docs.web3j.io/javadoc-api/) 。我们发现 Javadocs 是一个颇有争议的话题——开发人员要么喜欢它们，要么忽视它们，但不管怎样，作为一个严肃的开发人员框架，在这里满足尽可能多的需求是很重要的。

我们希望您会发现这个更新的文档对您的 Web3j 之旅非常有帮助。

## Web3j Solidity 库依赖管理

对于为 Java 和 Android 编写智能合约代码的开发人员来说，真正令人烦恼的一个特性是缺乏对 Solidity 库的依赖管理。例如，当您想要使用一个参考 ERC20 实现时，比如[openzeplin 的](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)，您需要手动将相关的库依赖项复制并粘贴到您的项目中，然后确保它们在正确的位置进行编译。

为了解决这个问题，我们现在增加了对使用' @ '注释提取 Solidity 依赖项的支持，这样你就可以使用任何已经发布到 NPM 资源库的 Solidity 源代码。

这意味着，要编写一个 ERC20 令牌，您只需提供以下可靠性代码:

```java
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract ERC20Token is ERC20 { 
     constructor() ERC20(“My Token”, “MT”) public {
     }
}
```

然后使用 Web3j Gradle 插件，它将负责下载所有 Solidity 依赖项，下载 Solidity 编译器，生成 Java 智能合同包装器，甚至为它创建一个 OpenAPI 服务，如果你将它与 Web3j OpenAPI 插件结合使用的话！！！

我们真的很高兴所有这些智能现在都在幕后进行。如果你想了解更多关于依赖管理特性的信息，你可以看看这篇[博客文章](/solidity-dependency-management-comes-to-web3j)和 web3j [文档](https://docs.web3j.io/4.8.7/)。

## Web3j Eth2

随着 Eth2 信标链的推出，我们也希望开始支持新的 Eth2 网络。考虑到 Eth2 规范不断发展的本质，等到规范达到一定程度的稳定性是很重要的，这在过去的几个月里实现了。

因此，我们创建了 Web3j Eth2 项目，这是一个完整的 JVM 客户端实现，使用开发人员非常熟悉的干净友好的 Web3j 语法来实现 [Eth2 信标节点 API](https://ethereum.github.io/eth2.0-APIs/) 。你可以在[的后续文章](/announcing-web3j-eth2-beacon-node-api-client)和[的项目文档](https://github.com/web3j/web3j-eth2#web3j---ethereum-20)中了解更多信息。

## Web3j OpenAPI

虽然这不是一项新的工作，但我想提醒大家关于 Web3j OpenAPI 项目，该项目采用 Solidity 代码并生成一个可运行的 OpenAPI 服务，供客户端部署并与 Solidity 智能合同交互。这是一个令人敬畏的工程，你可以在下面的[帖子](https://blog.web3labs.com/web3j-open-api)中读到更多，并参考[文档](https://docs.web3j.io/4.8.7/web3j_openapi/)。

## Web3j 社区

最后，我想把大家的注意力吸引到 [Web3 实验室社区论坛](https://community.web3labs.com/)上，我们鼓励人们讨论与 Web3j 有关的任何事情。在过去的几年里有一个非常活跃的 [Gitter 社区](https://gitter.im/web3j/web3j)，但是如果用户通过聊天渠道使用论坛，我们会很高兴，因为它使讨论能够围绕特定主题形成和发展，使每个人都更容易为讨论做出贡献并从中受益。

### 下一步是什么？

在接下来的几个月里，我们为 Web3j 计划了更多的工作，但是我希望你喜欢更多地了解我们一直在做的事情——看到我们已经走了这么远真是令人惊讶。再次感谢以太坊基金会为这项工作做出的贡献——它真的帮助我们增强了开发者的体验。