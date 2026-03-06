# Web3j ENS 第 2 层和离线数据支持

> 原文：<https://blog.web3labs.com/web3development/web3j-ens-layer2-and-offchain-data-support>

web3j 版本 4.9.2 中增加了新的重要 ENS 特性。这些功能旨在定义一个使用 L2 解决方案和外链数据的通用标准。

*   ENSIP-10:通配符解析(EIP-2544)
*   EIP3668: CCIP 读取:安全离线数据检索

## ENS 通配符解析

许多应用程序表达了通过共享父域上的自定义子域为其用户发布 ENS 名称的愿望。然而，这样做的成本对于大型用户群来说是相当昂贵的，因为每一条记录都应该在 ENS 注册中心进行设置。

启用对 ENS 的通配符支持为构建更高级的解析器打开了大门，这些解析器可以确定性地为未分配的子域生成地址，并对 L2 的采用具有重要意义。

例如，web3j.eth 解析所有*.web3j.eth ENS 名称。这意味着对于任何给定的 ENS 名称，例如 my-test.web3j.eth，一切都将正常工作，不需要任何更改，这意味着所有者不需要添加任何内容。

好消息是，您不需要在代码中添加或更改任何东西就可以开始使用这个特性。不需要修改现有的 ENS 注册管理机构合同或任何现有的解析器，它们将使用现有的 ENS 记录以相同的旧方式工作，没有任何问题。传统的 ENS 客户端将无法解析通配符记录。

要开始使用这个很酷的特性，你只需将 web3j 升级到 4.9.2 版或最新版本。

## ENS L2/非连锁整合

L2 支持是当今区块链社区最重要的领域之一，它减少了汽油费，并使互动更快。

有很多技术可以将数据移出链外，它们都采用一种思想，即只存储最少量的信息，以便在需要时验证外部数据。问题是每个应用程序都有自己的解决方案和方法。更糟糕的是，每个解决方案都使用一些特定的数据存储，并且不容易切换到另一个。

下面我建立了一个简单的例子来调用这个新特性的新功能。

| public static void**main**(String[]args){
web 3j web 3j = web 3j . build(new HttpService(HTTP _ MAINNET _ URL))；
ens resolver ens resolver = new ens resolver(web 3j)；
String address = ens resolver . resolve(" 1 . off chain example . eth ")；

(地址)system . out . println；
}

 |

| > 0x 41563129 cdbbd 0 C5 D3 E1 c 86 cf 9563926 b 243834d |

有任何问题或意见吗？我们希望收到您的来信！如果你想更多地了解区块链，它的发展和最新的发展，那就去看看我们的 [dev 博客](/web3development)或者听听我们启发性的 [Web3 创新者播客](https://podcast.web3labs.com/)。