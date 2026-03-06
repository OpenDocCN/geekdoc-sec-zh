# 法定上的 Azure ERC-20 令牌服务

> 原文：<https://blog.web3labs.com/web3development/azure-erc-20-token-service-on-quroum>

我们最近在 Azure Marketplace 上发布了我们的第一个区块链解决方案。

这个示例环境为在 [Quorum](https://github.com/jpmorganchase/quorum) 之上创建和管理 ERC-20 令牌提供了 RESTful 服务。

它提供了一个 3 节点仲裁环境和 RESTful 服务端点，用于通过 HTTP 与每个节点进行交互。这些服务通过使用 [web3j](https://web3j.io/) 和 [Spring Boot](https://projects.spring.io/spring-boot/) 的 [erc20-rest-service](https://github.com/blk-io/erc20-rest-service) 提供。

由于 Quorum 的存在，完全的交易隐私得到了支持。

![ERC-20 API](img/593eb101a95f13ffd9b82875c9041826.png)

## **入门**

按照通常的 Azure 虚拟机映像创建机器映像，我们建议您使用 *DS1 v2* 或更好的主机，因为在单个虚拟机上运行仲裁集群需要内存。

您应该使用 *ubuntu* 作为用户名，然后按照普通的 Azure VM 部署主机。

一旦在 Azure 上可用，登录到主机，并使用以下命令启动应用程序:

```java
cd erc20-quorum-vm-example
./init.sh
```

这将启动一个使用 RAFT 共识的 3 节点仲裁集群，以及我们的 ERC 20 RESTful 服务的 3 个实例。

## **部署我们的 ERC 20 合同**

[ERC-20 令牌标准](https://theethereum.wiki/w/index.php/ERC20_Token_Standard)是一个以太坊标准，用于实现在以太坊网络上提供令牌的智能合约。它提供了这些契约应该实现的一组公共方法。

这些服务均可从以下网址获得:

[http:// <主机名> :8080/swagger-ui.html](http://52.178.114.17:8082/swagger-ui.html)

[http:// <主机名> :8081/swagger-ui.html](http://52.178.114.17:8082/swagger-ui.html)

[http:// <主机名> :8082/swagger-ui.html](http://52.178.114.17:8082/swagger-ui.html)

在每个网址应该有一个大摇大摆的用户界面，看起来像下面。

![Deploying our ERC-20 contract](img/ed049e52b25a5fb21b75433f465a2ca9.png)

现在，您已经准备好体验服务本身了。

我们将从创建自己的名为 *QToken* 的令牌开始。从 Swagger UI 中选择 *POST /deploy* 操作(可以使用 URL[http://<hostname>:8080/Swagger-UI . html #！/controller/deployUsingPOST](http://52.178.114.17:8080/swagger-ui.html#!/controller/deployUsingPOST) 可选)。

![Deploying our ERC-20 contract](img/7ce3a43c61c7afdbc7445c8408384f0f.png)

提供以下参数值

*合同规格:*

```java
{
    "initialAmount": 1000000,

    "tokenName": "QToken",

    "decimalUnits": 0,

    "tokenSymbol": "Q$"

}
```

*专用于:*

```java
1iTZde/ndBHvzhcl7V68x44Vx7pl8nwx9LqnM/AfJUg=
```

然后点击*试试看！*按钮。

您应该会看到如下所示的响应:

![Deploying our ERC-20 contract](img/039ff23b33c44a6dad566500bf3fa052.png)

恭喜您，您刚刚将智能合同部署到法定网络！它使用 Quorum 的交易隐私来确保交易仅在三方网络中的两方之间可见。

智能合约在网络上的地址是响应体:

在这种情况下，其:

```java
0x938781b9796aea6376e40ca158f67fa89d5d8a18
```

## **证明 Quorum 的交易隐私**

现在让我们证明该契约只对网络中的一部分参与者可用。我们可以通过调用智能契约上的任何方法来实现这一点。如果我们被允许查看智能合同，我们将得到一个结果，否则我们不会。

我们将从每个节点调用契约上的 symbol 方法。

我们从第一个节点开始:

[http:// <主机名> :8080/swagger-ui.html#！/控制器/符号获取](http://52.178.114.17:8080/swagger-ui.html#!/controller/symbolUsingGET)

输入我们提供的合同地址(*0x 938781 b 9796 ea 6376 e 40 ca 158 f 67 fa 89 D5 D8 a 18*)

点击*试试看！*

您应该在响应体中看到符号名称 *Q$* ,表明我们的第一个节点可以访问契约。

![Demonstrating Quorum’s transaction privacy](img/85bb7991cc4ff7444bf0a61f84944fdf.png)

现在，尝试在我们的第二个节点上做同样的事情:

[http:// <主机名> :8081/swagger-ui.html#！/控制器/符号获取](http://52.178.114.17:8080/swagger-ui.html#!/controller/symbolUsingGET)

这一次，您将得到不同的响应，因为第二个节点并不参与该事务。

![Demonstrating Quorum’s transaction privacy](img/cc07187d78300b89f0bc0df73242b669.png)

因此，请求无法返回 HTTP 500 响应:

{
【时间戳】:1505302053178，

【状态】:500，

【错误】:“内部服务器错误”，

【异常】:“java.lang.RuntimeException”，

【消息】:“调用返回空值”，

【路径】“/0x 938781 b 9796 aa 6376 e 40 ca 158 f 67 fa 89d 5

最后，我们尝试使用第三个节点，这是成功的，因为该节点参与了事务。

[http:// <主机名> :8082/swagger-ui.html#！/控制器/符号获取](http://52.178.114.17:8080/swagger-ui.html#!/controller/symbolUsingGET)

![Demonstrating Quorum’s transaction privacy](img/dda01a355bf1d565d2b8443706d9d843.png)

### **结束语**

这证明了 Quorum 的事务私密性。

或者，您可以在不利用仲裁的事务隐私的情况下执行这些操作，将 privateFor 字段留空。然后，这个演示就像在一个普通的以太网上一样运行，但是使用了 Quorum 的快速 RAFT 共识机制。

根据 ERC20 令牌标准，您可以调用许多其他方法。我们鼓励你演一出戏。

如果您有任何问题或想聊天，请随时[联系我们](mailto:hi@web3labs.com)。