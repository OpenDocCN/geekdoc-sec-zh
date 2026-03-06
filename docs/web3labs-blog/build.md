# //build/的 Sirato

> 原文：<https://blog.web3labs.com/web3development/block-explorer-epirus-at-/build>

Web3 实验室最近几周很忙。继上周我们的 Sirato 区块链浏览器在 [Azure](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/blk-technologies.azure-blockchain-explorer-template) 上[发布](https://medium.com/web3labs/azure-blockchain-service-explorer-495e6702d762)之后，我们还有幸被微软 Azure 首席技术官 [Mark Russinovich](https://mybuild.techcommunity.microsoft.com/speaker/545398) 选中用于他在 Build 上的演示。Build 是微软的首要开发者大会，于本周早些时候举行——查看该活动的视频[简化区块链开发](https://www.youtube.com/watch?v=7TcLb94NxFc&t=32m7s)(我们在 32:02)。

 [![Azure Blockchain Service](img/b0c3ebfcdee8ee8f3a2dc5a9e610e844.png)](https://mybuild.techcommunity.microsoft.com/sessions/77268) 

## 企业区块链浏览器

对我们来说，支持业务用户一直是一个重要的考虑因素，这一问题不能仅靠工程来解决。因此，去年年底，我们开始全面改进我们的浏览器用户体验，以支持商业用户。sira to Enterprise block chain Explorer 是最终的成果，我们对我们取得的成就感到非常自豪。

![Epirus Enterprise Blockchain Explorer](img/f4eb7861d1c4220b9d2647dd31d21eb5.png)

正如你从上面看到的，我们已经对界面进行了彻底的检查。它有一个全新的外观，恰好符合我们的品牌身份。

## **简单**

新的体验强调用户的简单性，我们不想给他们增加他们不需要经常参考的信息的负担。当然，如果需要的话，细节仍然存在，但是我们希望确保我们在屏幕上提供合理的信息默认值。

![Epirus Enterprise Blockchain Explorer](img/70b2c07e51cf9b8229ad774dc68222fa.png)

## **(业务)用户体验**

业务用户需要有意义的业务指标来支持区块链正在帮助解决的业务问题。如果没有更广泛的上下文，块和事务是没有意义的。因此，Sirato 优先考虑合同的观点。

## **合同登记处**

Sirato 提供了一个智能合同注册表，允许我们的用户注册他们的合同 ABI 数据。这极大地增强了浏览器中可见的合同数据。契约方法和事件名称以及它们的参数都变得可见，允许深入了解它们的应用程序正在做什么。

![Epirus Enterprise Blockchain Explorer](img/df2c722b6feb5ea6596dc287f1d0f747.png)

## **代币**

我们真正感到兴奋的另一个功能是能够查看网络中部署的所有令牌的详细信息，并为可替换或不可替换的令牌(ERC20 或 ERC721)分别贴上标签。

![Epirus Enterprise Blockchain Explorer](img/94b460ef4e48ced3ac76ca0801965c64.png)

考虑到企业环境中对令牌的兴趣有多大(参见企业以太坊联盟最近关于[令牌分类框架](https://entethalliance.org/enterprise-ethereum-alliance-launches-blockchain-neutral-token-taxonomy-initiative-to-accelerate-a-token-powered-blockchain-future/)的公告)，我们相信这将有助于不太懂技术的用户理解在他们网络上运行的令牌应用程序。

## **高级排序和过滤器**

另一个关键特性是在 Sirato 中增加了排序和过滤器。这使我们的用户能够通过按关键属性对各种视图进行排序来找到非常活跃的合同。合同可以按交易日期、交易计数或事件合同按交易计数排序。

![Epirus Enterprise Blockchain Explorer](img/73acdd2edda1a19b8e2cdbb1ac3b239d.png)

*Filter by multiple attributes*

但这还不是全部！提供的过滤器允许我们的用户查看可用数据的子集，例如只显示合同创建交易或可替换的令牌。

![Epirus Enterprise Blockchain Explorer](img/879beaa464bab16b0af7cd96c26052ff.png)

## **商业智能**

我们真的为新的用户体验感到骄傲。我们知道，提取商业智能数据用于报告是任何业务中的另一个重要考虑因素。Sirato 也提供了 RESTful API，可以用来提取你在界面中看到的所有数据。这与过滤功能和合同注册相结合，使我们的用户能够对其应用程序执行定制报告，例如与特定令牌相关的所有转让事件的详细信息。

![Epirus Explorer API](img/7186635a5571c0bd97c0b594da8e4077.png)

## **入门**

Sirato 支持许多不同的主机选项，这些选项都来自 Web3 实验室团队的支持。你可以在 Azure Marketplace 找到它，免费试用，我们还提供托管的 SaaS 版本，并提供认证访问。

![Epirus Azure Blockchain Service Explorer](img/d5580df275a69a29d3fc0ee073bd0b51.png)

## **下一步是什么**

这只是冰山一角。我们对 Sirato 有宏伟的计划，它将为商业用户提供更大的简单性，为开发者提供更集成的体验。如果您想了解更多信息，请加入我们的[邮件列表](https://blk.us17.list-manage.com/subscribe/post?u=412696652858d5fc58dd705c9&id=b629184709)或进入[触摸](mailto:hi@web3labs.com)！否则，请在接下来的几周继续关注！