# Citadel 特洛伊木马使用阴险的社会工程形式

> 原文：<https://www.social-engineer.org/social-engineering/citadel-trojan-uses-insidious-forms-of-social-engineering/>

宙斯木马已经够糟糕了，但攻击者并没有停止他们的努力。今天有报道称这种木马出现了一个新变种，叫做“城堡”。

Citadel 是宙斯木马、勒索软件、恐吓软件和社交网络的混合体。这个木马特别恶意。它出现在一个“路过”网站上，然后被下载到受害者的电脑上。

它偷偷安装后，发送信号下载并安装名为 Reveton 的勒索软件。然后这会冻结受害者的计算机，并提供一个看起来非常合法的弹出窗口:

[![](img/4ddc0f4c34d27f159c842125e99d4b24.png "police_trojan_screenshot")](https://www.social-engineer.org/interesting-se-articles/citadel-trojan-uses-insidious-forms-of-social-engineering/attachment/police_trojan_screenshot/)

这个弹出窗口是[恐吓软件和勒索软件](https://www.social-engineer.org/framework/general-discussion/categories-social-engineers/hackers/ "Social Engineer Hackers")的一部分。它告诉受害者在他们的电脑上发现了儿童色情内容。它记录下他们的 IP 地址，并威胁要将此事报告给相关部门，除非他们支付“罚款”。与坐牢、失业和被贴上恋童癖的标签相比，区区 100 美元似乎微不足道，不是吗？

我真的希望我能说就此打住。但是，这种特别邪恶的木马的创造者使用一种社交网络连接与受感染的机器进行通信，并窃取银行信息或其他个人信息来进一步攻击。

我们能学到什么？

除了恶意攻击者变得越来越邪恶，越来越不关心他们的同伴这一事实之外，这里还有一些严重的社会工程学方面的因素在起作用。

首先，悉尼大学进行的一项关于色情成瘾的研究表明，66%的被调查者承认经常看色情电影。担心你的互联网缓存中有一个流浪的图像或东西，意味着这个木马有 66%的机会击中某人，他可能会想，“Hrm，在那个页面上有每个 18 岁以下的人吗？我不想进监狱，让我来付 100 美元的罚款。”。

对于另外 33%的人来说，害怕被政府机构举报仍然是合理的。约翰·弗洛伊德，一位为被错误指控的人争取权利的著名律师，讲述了那些因虚假指控而在监狱度过几十年的人的故事。这些故事再次充斥互联网，导致许多人害怕自己会被监禁，即使他们是无辜的。

这种恐惧让人们把支付“罚款”合理化。埃默里大学的格雷戈里·伯恩斯博士进行了一项研究，基本上陈述了“恐惧和害怕导致我们做出糟糕的决定”。这是这些攻击者利用的心理学原理。害怕被抓，害怕被诬告，害怕你的电脑被损坏或无法使用。

它也是成功的。一份关于 malwaresurvival.net 的报告声称有超过 360 万宙斯感染(Citadel 的前身),想想看，如果这些人中的每一个人，甚至其中一些人支付了 100 美元的“罚款”……那就是 1.5 亿到 3.6 亿美元。的确是一笔大生意。

如果你觉得你被感染了，你能做什么？

**删除**
如果您受到此特洛伊木马的影响，您可能需要执行以下说明来手动删除它:

1.  按 CTRL+O
2.  在打开的对话框中，按原样键入以下内容，然后按 Enter 键:
3.  cmd.exe
4.  在命令提示符窗口中，键入以下内容，然后按 Enter 键:
5.  CD“% user profile % \开始菜单\程序\启动”
6.  仍然在命令提示符窗口中，按原样键入以下内容，然后按 Enter 键:
7.  del *.dll.lnk 文件
8.  仍然在命令提示符窗口中，按原样键入以下内容，然后按 Enter 键:
9.  关闭-r -t 0

在[Microsoft.com](https://www.microsoft.com/security/portal/Threat/Encyclopedia/Entry.aspx?Name=Trojan%3aWin32%2fReveton.A "Citadel Mitigation")上找到更多信息和缓解提示

下次见，注意安全。