# 不是你的普通士兵乔

> 原文：<https://www.social-engineer.org/general-blog/not-average-g-joe/>

早在 2015 年，我们就报道了物联网网络连接的 Hello Barbie 玩偶中的[可能存在的漏洞，以及儿童云存储通信中隐私和安全漏洞的可能性。两周前，德国禁止商店出售互动的凯拉娃娃，并向购买过它的父母发出警告。凯拉娃娃被发现是不安全的，而且](https://www.social-engineer.org/general-blog/hello-barbie-the-doll-that-really-listens-2/)[违反了德国禁止隐藏监控设备的法律](https://www.bundesnetzagentur.de/SharedDocs/Pressemitteilungen/EN/2017/17022017_cayla.html)。

![Not Your Average (G.I.) Joe](img/ecf573ef612b31741ce7bdb8e05a5ea6.png)

周一，据宣布，最近对 CloudPets 毛绒动物的攻击再次引起了人们的关注。CloudPets 允许父母利用智能手机应用程序在毛绒玩具上给他们的孩子留下信息，孩子们也可以给他们的父母发信息作为回报。电子邮件地址和密码等个人资料都存储在不安全的 MongoDB 上；事实上，它非常不安全，既没有密码也没有防火墙保护。虽然帐户密码被散列，但它们很容易被破解，因为许多密码就像“cloudpets”或“1234567”一样简单。实际的信息被发现存储在亚马逊服务器上，该服务器也不需要认证，攻击者所要做的就是猜测或稍微改变 URL 以获得对录音的访问。

在写这篇博客的时候，发现了更多关于 CloudPets 漏洞的令人震惊的数据。蓝牙范围内的任何人都可以连接到毛绒玩具并记录它当时听到的内容，或者上传任何性质的消息供它播放给孩子听。它基本上可以把玩具变成你家里的窃听装置。

安全研究人员一直试图联系 CloudPets 的制造商，但没有成功，数据泄露至少从去年年底就已经在网上曝光。不过，截至本文撰写之时，这家玩具制造商尚未回应或解决这一问题。如果您的孩子拥有这些玩具中的一个，您可能应该考虑更改您的帐户密码以及可能使用相同密码的任何其他帐户的密码(如您的银行)，并关闭设备，直到问题得到解决。

随着物联网在我们家中的物品中变得越来越普遍，消费者需要向制造商要求更好的安全性。像这样的漏洞也说明了为什么不建议在多个网站上使用相同的密码，以及为什么不建议在连接到互联网的每个帐户上使用复杂的密码。此外，请务必与您的非技术朋友讨论这些问题和违规行为，他们可能没有意识到玩具放在客厅的危险。如果你仍然想要一个可以给你的孩子播放信息的玩具，在 EBay 上仍然可以买到安全的免于黑客攻击的玩具。

*来源:*
*[https://www . social-engineer . org/general-blog/hello-Barbie-the-doll-that-really-listens-2/](https://www.social-engineer.org/general-blog/hello-barbie-the-doll-that-really-listens-2/)*
*[https://www . bundesnetzagentur . de/SharedDocs/Pressemitteilungen/EN/2017/170222017 _ cayla . html](https://www.bundesnetzagentur.de/SharedDocs/Pressemitteilungen/EN/2017/17022017_cayla.html)*
*[https://https](https://www.nytimes.com/2017/02/17/technology/cayla-talking-doll-hackers.html)*