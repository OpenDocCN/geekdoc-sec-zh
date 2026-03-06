# 您的设备受到攻击

> 原文：<https://www.social-engineer.org/general-blog/your-appliances-are-on-the-attack/>

10 月 21 日星期五对许多互联网用户来说是艰难的一天。这一天发生了针对 Dyn DNS 的大规模 DDoS 攻击；关闭 Twitter、亚马逊、纽约时报等网站。你的设备被攻击的那一天。随着我们最喜欢的社交媒体工具的关闭，OSINT 变得越来越小，所以我们不得不寻找其他方式来打发时间(谢谢 Candy Crush)

你可能会问，“对 Dyn 的攻击如何影响我的 Reddit 浏览？”简单来说，Dyn 为许多顶级网站提供域名系统(DNS)。因此，当你在浏览器中输入 www.reddit.com 时，你会被自动指向该网站的正确 IP 地址。如果没有像 Dyn 这样的服务，我们必须知道我们想要访问的网站的确切 IP 地址，这需要记住很多数字。那天，一些东西开始用流量淹没 Dyn 的服务器，直到故障点，或者被称为分布式拒绝服务(DDOS)。这就相当于欺骗优步派遣数千辆汽车到你家造成交通堵塞，然后阻止任何合法的交通通过街道。

最终，人们发现 Mirai 未来组合僵尸网络已经感染了数十万台联网设备。这些大多是安全摄像头、DVR 和其他未加保护或使用出厂默认密码的设备。更大的问题是，这些设备中的许多都有不可更改的硬编码密码，如果没有手动固件升级，仍然容易受到更多攻击。

![Your Appliances Are On The Attack](img/b65725dd4759f28a7eb5bc2b05ef4ed0.png)

《大西洋月刊》的一些记者最近试图测试他们的物联网设备有多脆弱，所以他们[制作了一个假的网络烤面包机](https://www.theatlantic.com/technology/archive/2016/10/we-built-a-fake-web-toaster-and-it-was-hacked-in-an-hour/505571/)并放到网上。他们认为可能需要几天或一周的时间来获得一次黑客攻击尝试，但令他们震惊的是，第一次黑客攻击是在上线后不到一小时内发生的。10 个小时后，超过 300 个不同的 IP 地址试图侵入“烤面包机”。

在物联网设备缺乏安全性的情况下，需要考虑的一件事是跟踪您的房子是否/何时被占用的能力。当你不在工作的时候，许多人会设定他们的自动调温器在这段时间内调节温度。随着我们接触到更多的物联网设备，如智能灯、联网冰箱、锁和车库门开门器，想象一下如果它们在家中无人时被黑客攻击并报告回来。这将是一个小偷的天堂，拥有这些信息，并能够随意盗窃；你的摄像系统也捕捉不到它，因为它也被黑了。

在为您的家庭购买物联网设备之前，请确保您研究了该设备，以确保您可以更改所有密码。然后在设备安装后立即更改所有密码。如果你发现一个不安全因素，写信给公司抱怨缺乏安全性并报告漏洞。

*来源:*
*[https://www . thealantic . com/technology/archive/2016/10/we-build-a-fake-web-toaster-and-it-is-hacked-in-hour/505571/](https://www.theatlantic.com/technology/archive/2016/10/we-built-a-fake-web-toaster-and-it-was-hacked-in-an-hour/505571/)*