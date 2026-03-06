# 使用 Sirato 审计智能合同

> 原文：<https://blog.web3labs.com/web3development/blockchain-explorer-auditing-smart-contracts-with-epirus>

智能合约中的错误会导致问题。从最初的 DAO 黑客攻击，到奇偶校验的 multisig 钱包问题，再到整数溢出和下溢导致的较小错误，软件错误已经导致了现实世界的后果。其中一些已经严重影响了对各种公司的信任，以及更广泛的以太坊生态系统。

对于新开发人员来说，在 Solidity 中编写无 bug 代码可能很困难，即使是经验丰富的区块链工程师也会不时出错。漏洞通常很微妙，在它们被提交之前可能不会被注意到。虽然代码审查和审计可以成功地用于消除大多数这类错误，但两者都是不完美的，而且通常是昂贵的:前者花费开发人员的时间，后者花费金钱。

## **解决方案**

Sirato CLI 的目标是让新开发人员更轻松、更容易地开发智能合约，为此，我们将智能合约静态分析工具直接集成到应用程序中。使用静态分析，可以识别常见的错误，如可重入性、不安全的算法和拒绝服务的漏洞。Sirato 可以提供关于检测到的功能和潜在漏洞的特定信息，包括相关文件和行号。

## **演示**

首先，[安装 Sirato](https://www.web3labs.com/web3j) 如果你还没有的话。

然后，为了在现有智能合同上测试此功能，只需运行:

```java
epirus audit <filename> 
```

考虑[下面的 Solidity 代码](https://gist.github.com/josh-richardson/d03f2ad51b0ee4a1e6ad0ff82b098e45)，它实现了一个简单的众筹活动，对它的审计产生以下输出:

![Smart Contracts with Epirus](img/8402d78277613bfb245ba5c255f3f9b8.png)

已经生成了四行输出，每一行表示静态分析工具检测到的一个特征。第一列显示与检测到的功能相关联的行和字符，第二列显示严重性(1 表示信息，2 表示警告，3 表示严重漏洞)，第三列提供检测到的问题的详细信息，最后第四列提供被触发的规则的标识符。

严重性为 1 的检测往往是信息性的，或者是使用最佳实践或帮助可读性的建议，但是严重性为 2 的项目通常需要注意。严重性为 3 的警告是一个关键问题，从输出的第一行可以看出，该合同的作者未能包括从众筹基金中提款的方法，从而将资金锁定在合同中，实际上使它们无法使用。为了解决这个问题，应该创建一个提取资金的函数。

这样的函数可以这样实现，并且应该从投票函数内部调用:

```java
function payStage() internal inState(State.Funded) returns (bool) {

     uint256 totalRaised = currentBalance;

     currentBalance = currentBalance.sub(balancePerStage);

     if (creator.send(balancePerStage)) {

         emit CreatorPaidStage(creator);

         return true;

     } else {

         currentBalance = totalRaised;

         state = State.Successful;

     }

     return false;

}
```

本合同有一个[固定版本](https://gist.github.com/josh-richardson/95cca5b07919230b6e7472218f2fa3fa)。在检查并修复此问题后，运行另一个审计应该只显示严重性为 1 的问题——在这种情况下，它们是信息性的，不会对合同的安全性构成威胁。

## **与您的工作流程整合**

Sirato 审计功能本身对于审计合同是有用的，但是代码分析工具通常在集成到开发过程本身时才能得到最有效的应用。因为如果检测到任何严重性高于 1 的问题，audit 命令将返回非零退出代码，所以它可以很容易地与预提交 git 挂钩或甚至 Jenkins/Travis/etc 上的 CI 管道结合使用。未来的博客文章将记录如何将 Sirato 的审计功能集成到您的开发过程中。

智能合同审计由 [SmartCheck](https://github.com/smartdec/smartcheck) 提供，并在本地执行——没有文件上传到任何第三方服务器。

开发健壮的智能合约是困难的，这就是为什么我们想在 Sirato 中提供所有的工具来尽可能简单地开发区块链。你知道吗，Sirato 还可以为你自动生成单元测试，并提供嵌入式区块链环境。阅读 [Sirato 文档](https://docs.epirus.io/)了解更多信息。我保证你不会后悔的！