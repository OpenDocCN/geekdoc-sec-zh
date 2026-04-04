# 2021 漏洞研究人员黑客概述

> 原文：<https://www.mitnicksecurity.com/blog/an-overview-of-the-2021-vulnerability-researchers-hack>

如果你认为 2020 年是社会工程计划[](https://www.mitnicksecurity.com/blog/the-top-5-most-famous-social-engineering-attacks-of-the-last-decade)的重要一年，那就要小心了！221 正在火热进行中。

新年不到四周，谷歌宣布了我们国家的最新漏洞:朝鲜黑客针对网络安全专家。

**没错。黑客盯上了** ***网络安全专家。***

让我们回顾一下这一 2021 年初的社会工程攻击，并发现即使是网络安全专家自己也是如何感染恶意软件的。

## 发生了什么事？

谷歌的*威胁分析小组*，简称 TAG，是世界上最大的搜索引擎安全团队。

这个谷歌团队的工作是跟踪和阻止高级持续威胁(APT)团体。就像他们是行业领袖一样，当 TAG 发现一个重大的网络威胁时， [他们迅速在他们的博客上宣布](https://blog.google/threat-analysis-group/new-campaign-targeting-security-researchers/) 。

![](img/486893f60aeb9020036bd53823ffd421.png)

该帖子宣布，来自朝鲜黑客行动的坏人名为 *Lazarus Group* ，设计了一个偷偷摸摸的 [社会工程漏洞](https://www.mitnicksecurity.com/blog/social-engineering-attacks) 来针对从事漏洞研究的网络安全专业人员。

黑客们模仿有相似兴趣的研究员，通过虚假的社交媒体资料和电子邮件联系真正的漏洞研究员。

在一些最初的接触之后，这些坏演员建立了一些友谊，黑客们发送了一封钓鱼电子邮件，询问研究人员是否有兴趣合作。这些假冒的研究人员随后发送了一个 Visual Studio 项目的链接，当该项目打开时，就会在受害者的系统上安装恶意软件。

但这不是拉扎勒斯集团唯一的攻击途径。在某些情况下，真正的研究人员会点击他们的假 Twitter、LinkedIn 等。社交媒体帖子。这些帖子包含指向坏演员博客的链接——通过访问这个受感染的页面，安全研究人员感染了恶意软件(在可疑的 [驾车下载](https://www.kaspersky.com/resource-center/definitions/drive-by-download) )。

## 黑客是如何进入的

这次朝鲜网络攻击背后的精英社会工程师是彻底的。他们花了很大力气建立虚假的“研究博客”，并创建了多个虚拟的社交媒体账户，他们经常在这些账户上发帖——所有这些都是为了欺骗研究人员，让他们相信自己是可信的研究员。

谷歌解释说，他们“使用这些 Twitter 个人资料发布他们博客的链接，发布他们声称的行为的视频，并放大和转发他们控制的其他账户的帖子”。 他们的博客甚至包括来自安全实验的“客座博文”，以进一步支持他们的虚假恶名。一个社会工程最好的真实例子。

一些黑客甚至分享了一个假的 YouTube 视频(他们自己上传的)，声称他们找到了一个绕过最近修补的 Windows Defender 漏洞的方法:CVE-2021-1647。这可能是为了让自己成为新漏洞的前沿研究者，而事实上，这些都不是真的。

为了获得目标的信任，一些漏洞研究人员通过电子邮件或社交媒体点击受感染的链接并下载 Visual Studio 项目。

“Visual Studio 项目中会有利用漏洞的源代码，”谷歌解释道。这个漏洞被怀疑是 Windows 10 或 Chrome 浏览器中的“ [零日”漏洞](https://us.norton.com/internetsecurity-emerging-threats-how-do-zero-day-vulnerabilities-work-30sectech.html)——意思是，它是由坏演员*独家发现的漏洞，*在黑客社区之外仍不广为人知。事实上，谷歌仍在试图找到这个安全漏洞，并提供 [奖励支付](https://www.google.com/about/appsecurity/chrome-rewards/) 帮助识别它。

不管*什么*这个零日漏洞是*，*安全漏洞让坏人可以在完全修补的最新数字防御周围注入恶意软件。一个真正可怕的想法。

谷歌分享说，Visual Studio Build 还包含一个额外的 DLL，这给了坏人可乘之机。DLL，或 [动态链接库](https://www.cyberbit.com/blog/endpoint-security/malware-terms-non-techies-dll-hijacking/) ，是一种定制的恶意软件，当黑客“进入”时，它会向远程命令和控制服务器发出警报，允许他们从自己的计算机上舒适地渗透系统。

## 社会工程的演变

社会工程正在发展，逐年变得越来越复杂。

“这场运动很有趣，因为它利用了研究人员合作的愿望，包括与我们不认识的人合作，以推进我们的工作，”红色金丝雀的情报总监凯蒂·尼克尔斯(Katie Nickels)告诉 TechRepublic，她也是朝鲜社会工程运动的目标。它利用了人类对其“社区”的基本信任，并操纵了研究人员在进一步发展脆弱性研究中的更大目的感。

当然，这次攻击的最大“惊喜”是工程师们创建可信的社交档案和虚假的研究人员网站和博客所经历的复杂过程。

这证明，即使是打满补丁的最新系统也可能被精明的黑客利用，他们会不遗余力地利用与政府的关系钓上“大网络钓鱼”。事实上，如此之长，以至于他们愿意冒着暴露鲜为人知的零日漏洞的风险来危及他们的目标。

## 关键要点

从这种社会工程攻击中可以吸取一些有价值的教训:

*   如果你不知道你在和谁说话，就不要假设合法性。
    这些黑客创建了强大的虚假个人资料和在线存在来愚弄研究人员。这强调了除了快速谷歌搜索之外，验证某人是否是他们所声称的那个人的重要性。

*   真正检查社交媒体档案。

While the bad actors had social media profiles and were frequently posting, most of them did not contain profile pictures of the individuals themselves. In this picture below from Google of (now banned) users from the exploit, you can see there’s no face behind the profile. This is an immediate sign of suspicion. Educate your team to beware the faceless user!

![](img/3b00c566f0835698e001e19abcc79ab4.png)

*   小心不要接受朋友或听从陌生人的要求。

We get LinkedIn and Facebook requests all the time from people we’ve never met. But these strangers could be bad actors in disguise. Think before allowing access to valuable private data in your feeds and profiles.

*   **大力投资安全意识培训。** 你的员工和管理层都需要知道在面对巧妙的社会工程漏洞时要注意什么。通过要求 [安全培训，让他们掌握常见黑客技术和操作的知识。](https://www.mitnicksecurity.com/kevin-mitnick-security-awareness-training)
    T8】

## 提高安全性的几个步骤

黑客变得越来越狡猾。这意味着你也需要。

**提高您公司的网络安全始于对您面临的威胁的认识。**

下载我们的 [*避免网络威胁的 5 步指南*](https://www.mitnicksecurity.com/lp-easy-steps-to-avoid-cyber-threats) ，了解一些提升您安全状态的有效方法，包括了解最新安全更新和新闻的一些资源。

[![New call-to-action](img/95ee2efaa0b0e1050f47338da41f7869.png)](https://cta-redirect.hubspot.com/cta/redirect/3875471/7f9b1de1-cf7c-4700-8892-cdf9402b32cf)