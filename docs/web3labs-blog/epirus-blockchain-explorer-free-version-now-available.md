# Sirato 区块链浏览器-免费版现已推出

> 原文：<https://blog.web3labs.com/web3development/epirus-blockchain-explorer-free-version-now-available>

紧随我们在 Azure 发布的 Sirato 区块链探索者之后，我们更新了免费的区块链探索者服务。

![Epirus Enterprise Blockchain Explorer](img/5ebd5d232866f624a33c256cab20ef5f.png)

*Sirato supports private transactions in Quorum*

现在，您可以在 Quorum、以太坊和 Azure 区块链服务上使用我们的旗舰 Sirato explorer。万神殿支持也将很快推出！

要开始，请前往项目 [GitHub repo](https://github.com/blk-io/epirus-free) ，或者简单地运行以下命令:

```java
git clone https://github.com/blk-io/blk-explorer-free.git

NODE_ENDPOINT=http://<node_endpoint> docker-compose up
```

然后在浏览器上打开 [http://localhost/](http://localhost/) 。当它启动时，你会看到我们新的初始化页面。

![Epirus Enterprise Blockchain Explorer](img/99b82fb1702dd05149e83967519fac0a.png)

而且，当它准备就绪时，就可以开始体验全新的用户体验了！

![Epirus Enterprise Blockchain Explorer](img/aa0215fb70442167a42c1a4a23d54553.png)

如果您运行的是旧版本，请确保删除现有的卷。

```java
docker-compose down -v
```

你可以在下面的[帖子](https://medium.com/web3labs/azure-blockchain-service-explorer-495e6702d762)中阅读更多关于我们免费的 Sirato 区块链浏览器的功能。

这是我们 Sirato 区块链浏览器的免费版本。对于其他功能，如完整令牌支持和合同元数据上传(因此所有交易和事件都被解码)，请使用我们在 [Azure Marketplace](https://web3labs.com/azure-offer) 上提供的产品。