# 前言

**目录**

*   1\. 序言
*   2\. 本书的阅读对象
*   3\. 感谢
*   4\. 文档约定
*   5\. 如何获得本书最新版本？
*   6\. 反馈

## 1\. 序言

Wireshark 是一种适合网络管理员使用的程序。但因为一直缺乏足够的文档资料阻碍了它的广泛流传。 提高 Wireshark 易用性，Wireshark 团队作出诸多努力，本书即是其中的的一部分。

我们希望本书能对您有所帮助，同时希望您能提出宝贵意见

## 2\. 本书的阅读对象

任何人只要有兴趣或者有需要都可以阅读本书！

本书将会讲解 Wireshark 的所有基本功能以及一些特性。 与原先的版本相比，今天 Wireshark 已经拥有非常丰富的功能，当然也变得更加复杂了。也许有些功能在本书中会略过不做讲解。！

本书不尝试揭示网络嗅探(Network sniffing)的原理（译者注：此处翻译不尽人意）。 很多有用的信息可以在 Wireshark 的 WiKI 网站上找到 [`wiki.wireshark.org`](http://wiki.wireshark.org) [1]

通过阅读本书，你将会了解如何安装 Wireshark，如何使用图形界面的各单元（比如菜单）， 以及一些不能直接发现的高级功能。本书将会初学者（有时甚至一些高手）解决使用中碰到的一些常见问题.

[1] 译者注：个人认为该网站还比较简陋

## 3\. 感谢

答谢的，此处就不译了,附原文如下

The authors would like to thank the whole Wireshark team for their assistance. In particular, the authors would like to thank:

*   Gerald Combs, for initiating the Wireshark project and funding to do this documentation.
*   Guy Harris, for many helpful hints and a great deal of patience in reviewing this document.
*   Gilbert Ramirez, for general encouragement and helpful hints along the way.

The authors would also like to thank the following people for their helpful feedback on this document:

*   Pat Eyler, for his suggestions on improving the example on generating a backtrace.
*   Martin Regner, for his various suggestions and corrections.
*   Graeme Hewson, for a lot of grammatical corrections.

The authors would like to acknowledge those man page and README authors for the Wireshark project from who sections of this document borrow heavily:

*   Scott Renfro from whose mergecap man page Section D.7, “mergecap: Merging multiple capture files into one ” is derived.
*   Ashok Narayanan from whose text2pcap man page Section D.8, “text2pcap: Converting ASCII hexdumps to network captures ” is derived.
*   Frank Singleton from whose README.idl2wrs Section D.9, “idl2wrs: Creating dissectors from CORBA IDL files ” is derived.

## 4\. 文档约定

本书由 Wireshark 基金会提供经费，Richard Sharpe 编写。经 ED Warnicke 更新，最近大多数更新，重新排版由 UlfLamping 进行。

本书使用 DocBook/XML 编写[2]

阅读本书时你会遇到一些特殊的标记

> ![](img/000057.png)
> 
> 警告
> 
> 看到警告时你需要加以注意了，可能会有数据丢失发生
> 
> ![](img/000066.png)
> 
> 注意
> 
> 指出一些常见错误，或者一些不明显的东西
> 
> ![](img/000068.png)
> 
> 提示
> 
> 提示对日常使用 Wireshark 很有用

[2] 译者注：未来的文档标准，适合书籍，资料之类的发布，有兴趣的可以 google 搜索 一下，译文不是用这个写的

## 5\. 从哪里可以得到 Wireshark

你可以从我们的网站下载最新版本的[Wireshark http://www.wireshark.org/download.html](http://www.wireshark.org/download.html).网站上您可以选择适合您的镜像站点。

Wireshark 通常在 4-8 周内发布一次新版本

如果您想获得 Wireshark 发布的消息通知，你可以订阅 Wireshark-announce 邮件列表。详见第 1.6.4 节 “邮件列表”

## 6\. 反馈

对本书的反馈信息可以通过[wireshark-dev[AT]wireshark.org.](mailto:wireshark-dev@wireshark.gor)发送给作者。