# 如何避免鼠患

> 原文：<https://www.social-engineer.com/how-to-avoid-a-rat-infestation/>

这不是讨论如何防止或摆脱那些大型啮齿动物，当它们在地板上奔跑或从天花板上掉下来时，会导致成年人像小孩子一样尖叫。我们指的是远程访问特洛伊木马(rat ),它正在全球范围内感染计算机和网络。这些 rat 是一种恶意软件，让黑客获得对您的计算机的管理控制。根据 Check Point 2018 年 10 月的全球威胁指数，这种威胁的受欢迎程度持续上升，其中一种形式 flawedAmmyy 进入了前十大最常见的恶意软件威胁。我们如何避免或防止鼠患？

### **![How To Avoid a RAT Infestation](img/6039af7f516fc44510b5143647a757dd.png)**

### 什么是老鼠？

远程访问特洛伊木马(RAT)是一种恶意软件程序，它打开后门来完全控制受害者的计算机。rat 通常通过用户请求的程序以不可见的方式下载，或者嵌入在电子邮件附件中。一旦安装，它可以通过间谍软件(如键盘记录器)监控用户行为，访问机密信息，激活系统的网络摄像头，分发病毒和其他恶意软件，更改文件和文件系统，等等。有些变得有自我意识，并能够逃避几种常用的恶意软件检测技术。他们有能力掩盖自己的存在，默默工作。一只名为 GravityRAT 的老鼠可以测量 CPU 的温度，并确定系统是否正在进行激烈的活动，并据此采取行动以逃避检测。

如前所述，rat 通常通过电子邮件发送，并以附件的形式出现，格式不限，如 Word、Excel、Publisher 和 PowerPoint 等 MS Office 产品。它们也可以是 Adobe Acrobat 文件或音频和视频文件。虽然大多数 rat 用于鱼叉式网络钓鱼活动(直接针对个人的攻击)，但 FlawedAmmyy 等 rat 已用于大规模网络钓鱼攻击。据报道，有一起针对 [Godiva 巧克力、酸奶和 Pinkberry 的大型攻击事件。](https://www.scmagazine.com/home/security-news/pied-piper-phishing-scheme-infests-victims-with-flawedammyy-rms-rats/)发送的附件可以伪装成发票或装运单据。土耳其的一个[鱼叉式网络钓鱼活动](https://www.riskiq.com/blog/labs/spear-phishing-turkish-defense-contractors/)声称是土耳其税务局的官方通信，如果填写了附加文件，接收者可能会免税。为了查看文档中的内容，受害者必须启用宏。所附文档中嵌入了代码，用于下载和安装一个名为 Remcos 的 RAT(基于同名的远程管理工具)。

除了上面提到的，其他一些需要注意的老鼠有 JBIFrost、DarkComet、Adwind、Coldroot 和 njRAT。所有这些都有相同的目的，即给予攻击者对受害者机器的远程管理权限。既然我们知道了老鼠是什么，它能做什么，我们如何检测、补救和预防鼠患呢？

### 如何检测、补救和预防

虽然这种形式的攻击并不新鲜，因为它的形式可以追溯到 2002 年，但它仍在继续发展，以领先于入侵检测系统(IDS)和高级持续威胁(APT)检测工具的发展。检测并不容易，但也不是不可能。如果是你的个人系统，你注意到一个性能问题，并且在你打开一个附件后有奇怪的事情发生，打电话给专业人员。

如果你是技术专家，并且你不想雇佣专业人员，那么你可以随意调查，看看是否有任何奇怪的进程在运行。此外，使用 Wireshark 等工具或在命令提示符下使用 Netstat 来查找来自计算机或网络的奇怪流量。如果您使用的是公司拥有的设备，并且怀疑您的系统可能有问题，请向您的网络管理员或网络安全部门报告。如果系统上有 RAT，则补救可能会因设备而异。推荐的方法是确保备份所有数据，擦除系统，然后重新安装操作系统。

说到预防，最好采用分层方法。其中应包括:

*   安装了防病毒和防恶意软件程序的系统，并确保它们始终是最新的；
*   全面修补和更新系统和安装的应用程序；
*   管理员应用严格的应用程序白名单，阻止未使用的端口，关闭未使用的服务，并监控传出流量以防止感染发生；

避免下载程序或打开来源不可靠的附件。

*   Avoid enabling macros without verifying that attachments are actually from trusted sources.

由于 rat 的初始感染机制通常是通过网络钓鱼电子邮件，因此您可以通过阻止这些网络钓鱼电子邮件到达您的用户、帮助用户识别和报告网络钓鱼电子邮件以及实施安全控制来帮助防止感染，以便恶意电子邮件不会危害您的设备。帮助教育用户和提高意识的一种方法是通过为您的用户群量身定制的[可靠的网络钓鱼程序](https://www.social-engineer.com/not-all-phishing-programs-are-created-equal/)，例如[社会工程师的网络钓鱼即服务](https://www.social-engineer.com/services/phishing-as-a-service-phaas/) (PHaaS)程序。如果你对防止鼠患感兴趣，那么在我们的网页【https://www.social-engineer.com/contact. T4】联系我们获取更多信息

资源:
[https://www . sc magazine . com/home/security-news/pied-piper-phishing-scheme-infest-victims-with-flawedammyy-rms-RATs/](https://www.scmagazine.com/home/security-news/pied-piper-phishing-scheme-infests-victims-with-flawedammyy-rms-rats/)
[https://www . zdnet . com/article/Ukrainian-police-arrest-hacker-who-infected-over-2000-users-with-dark comet-rat/](https://www.zdnet.com/article/ukrainian-police-arrest-hacker-who-infected-over-2000-users-with-darkcomet-rat/)
[https://www . zdnet . com/article/this-remote-access-trojan](https://www.zdnet.com/article/this-remote-access-trojan-just-popped-up-on-malwares-most-wanted-list/)