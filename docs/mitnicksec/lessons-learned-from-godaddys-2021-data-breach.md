# 从 go daddy 2021 年数据泄露事件中吸取的教训

> 原文：<https://www.mitnicksecurity.com/blog/lessons-learned-from-godaddys-2021-data-breach>

美国互联网域名注册商和网络托管公司 [GoDaddy](https://www.godaddy.com/) 最近制造了最新的 2021 安全漏洞新闻。

这次重大的网络攻击影响了 120 万现在和以前的托管客户，包括有托管计划的 WordPress 用户。

让我们看看数字隐私灾难的影响，从这次攻击中吸取有力的教训:

## 发生了什么事？

2021 年 11 月 22 日， [GoDaddy 宣布](https://www.sec.gov/Archives/edgar/data/1609711/000160971121000122/gddyblogpostnov222021.htm) 一个影响他们托管 WordPress 服务的安全事件。互联网域名注册公司告诉美国证券交易委员会(SEC)，他们在 11 月 17 日的五天前发现了对其“托管 WordPress 托管环境”的“未经授权的第三方访问”。

网络犯罪分子通过在他们的供应系统中泄露密码来入侵 GoDaddy 的数据库，他们通过向客户分配服务器空间、用户名和密码来为他们提供新的托管服务。

GoDaddy 黑客攻击的结果是，120 万拥有托管计划的 WordPress 用户的电子邮件地址和客户号被暴露。

此外，GoDaddy 还面临其他数据的泄露，包括:

*   【WordPress 管理员级别的原始密码
*   安全 FTP (sFTP)用户名和密码
*   活跃客户的数据库用户名和密码
*   活跃客户子集的 SSL 私钥 

根据 [Wordfence 安全专家](https://www.wordfence.com/blog/2021/11/godaddy-breach-plaintext-passwords/) 的调查，GoDaddy 的托管 WordPress 主机以不符合行业最佳实践的方式存储 sFTP 用户名和密码。

Wordfence 解释说，“GoDaddy 以这样一种方式存储 sFTP 密码，即密码的明文版本可以被检索，而不是存储这些密码的盐散列，或提供公钥认证。”由于将用户名和密码存储在未加密的纯文本中，不良行为者能够访问 GoDaddy 的托管 WordPress 遗留代码库中的供应系统。

更糟糕的是，违规事件本身发生在 2021 年 9 月 6 日，比 GoDaddy 团队在 2021 年 11 月 17 日正式发现它早了两个月。

## 谁会受到影响？

根据 GoDaddy 的说法，多达 120 万活跃和不活跃的托管 WordPress 用户的电子邮件地址和客户号码被暴露，但违规行为的蔓延并没有就此结束。

在漏洞公布的第二天， [GoDaddy 分享了](https://www.techradar.com/uk/news/godaddy-isnt-the-only-web-hosting-firm-caught-up-in-mega-breach) 转售 GoDaddy 管理的 WordPress 的品牌也受到了影响，包括 tsoHost、Media Temple、123Reg、Domain Factory、Heart Internet、Host Europe。调查仍在进行中，以确定受损数据的全部范围。

## GoDaddy 的下一步

在宣布数据泄露的同时，GoDaddy 还分享了他们为从大规模数据泄露中恢复所做的工作。域名注册商立即从他们的系统中阻止了未经授权的第三方，重置了受影响帐户的密码，并警告了受影响用户可能面临的威胁。

他们暴露的客户面临的最大风险之一是[](https://www.mitnicksecurity.com/blog/spear-phishing-targeted-email-scams-what-you-need-to-know-about-this-hacking-technique)的网络钓鱼攻击。他们的电子邮件地址暴露给了坏人直接访问他们的收件箱的机会。**通过一点开源情报研究和正确的借口，社会工程师可以编制一封高度针对性的钓鱼电子邮件，诱骗用户采取行动并下载** [**恶意软件**](https://www.mitnicksecurity.com/blog/5-common-hacking-techniques-for-2020) **。**

GoDaddy 的 CISO 总结了他的攻击声明，他说:“我们将从这次事件中吸取教训，并已采取措施，以加强我们的供应系统的额外保护。”但是这些额外的保护层是什么，我们还不确定...

## 经验教训

网络攻击正变得越来越频繁，并放大了影响，因为不良行为者继续将目标对准存储大量私人数据网络的大品牌。

在当今这个时代，没有一家公司可以免受网络攻击，所以在攻击发生之前经常评估你的安全性是至关重要的。

在 GoDaddy 的案例中，当公司发现他们的系统被入侵时，已经过去了整整两个月。这意味着对手花了 60 多天时间在他们的网络中横向移动。如果没有安全专业人员的彻底调查，很难确定他们的危害程度。考虑到这一点， **GoDaddy 修改被攻破账号密码的补救行动是不够的；他们必须执行** [**漏洞扫描**](https://www.mitnicksecurity.com/vulnerability-assessment) **和** [**渗透测试**](https://www.mitnicksecurity.com/penetration-testing) **来准确识别攻击的真实范围。**

现在，这些有新闻价值的网络攻击比以往任何时候都更加提醒我们，遵循存储敏感信息的安全最佳实践是多么重要。

## 您是否免受网络威胁？

现实情况是，许多组织对其真正的威胁形势和漏洞没有一个现实的想法。

虽然投资进行漏洞扫描或渗透测试是确定的最佳方式，但是您可以做一些事情来评估自己的安全状况。

立即下载我们的[*5-1/2 避免网络威胁的简单步骤*](https://www.mitnicksecurity.com/lp-easy-steps-to-avoid-cyber-threats) 电子书，学习保护您的组织免受内部和外部威胁。[![New call-to-action](img/95ee2efaa0b0e1050f47338da41f7869.png)](https://cta-redirect.hubspot.com/cta/redirect/3875471/7f9b1de1-cf7c-4700-8892-cdf9402b32cf)