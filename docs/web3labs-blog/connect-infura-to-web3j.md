# 如何将 Infura 连接到 Web3j

> 原文：<https://blog.web3labs.com/web3development/connect-infura-to-web3j>

以太坊是一个节点网络，在一个完全去中心化的世界里，你将在本地运行自己的节点，并在进行交易或查询数据时连接到那个节点。

然而，并不是每个人都可以或想要运行自己的节点，因此我们有像 [Infura](https://infura.io/) 这样的基础设施提供商，允许您使用他们的节点连接到以太坊区块链网络。

使用 Infura 而不是自己的节点超级简单。因为您保留了自己的钱包私钥，所以在这些服务之间切换就像将 Web3j 指向另一个节点 URL 一样简单。

你可以免费注册 Infura，使用他们的免费核心账户。完成之后，您将能够创建一个项目来获得一个项目 ID。Infura URL 将遵循以下结构:`https://<network>.infura.io/v3/YOUR-PROJECT-ID`

在我们的[部署者示例项目](https://github.com/web3j/web3j-deployer-demo/blob/1ea29066f7919302f3f0e852468dac27a1121114/src/main/java/demo/deploy/MyDeploymentLogic.java#L44)中可以找到这样的例子:

![Infura web3j sample project](img/893c9450c5ef8795a063d56efe98ba56.png)

这实际上就是全部，非常简单。

## 使用项目机密

以前的 URL 只需要项目 ID 就可以实现连接，但是 Infura 也支持项目秘密。这样的项目机密是使用 HTTP 基本认证传递给 Infura 的。

我们也可以在 Web3j 中支持这一点，方法是[传入一个预先配置好的 OkHttpClient](https://github.com/web3j/web3j-deployer-demo/blob/1ea29066f7919302f3f0e852468dac27a1121114/src/main/java/demo/deploy/MyDeploymentLogic.java#L75) 来验证自己，如下所示:

![OkHttpClient](img/883ffd8f5f9fe68fce75f5c82bf61578.png)

虽然从钱包安全的角度来看，确保 Infura 端点的安全并不重要，因为您总是保持您的私钥是私有的，但它确实会阻止其他人使用您的 Infura 请求权限。