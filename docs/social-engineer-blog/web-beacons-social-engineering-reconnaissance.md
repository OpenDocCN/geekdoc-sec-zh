# 社会工程侦察网络信标

> 原文：<https://www.social-engineer.org/general-blog/web-beacons-social-engineering-reconnaissance/>

你们大多数人都听说过互联网上的 cookies，但是网络信标跟踪你的浏览习惯更进一步。网络信标(又名:跟踪像素、网络漏洞和跟踪信标)主要由营销人员用来跟踪电子邮件、广告或文章被受众接收的程度。通常这些 1×1 像素的图像隐藏在电子邮件或页面的背景中，告诉营销人员观众是否打开了电子邮件或访问了特定页面。

![Web Beacons for Social Engineering Reconnaissance](img/07dbc265496165947bdb3990ba9a8863.png)

## 看起来无害，对吗？

人们不禁会想，“谁在乎亚马逊是否知道我对他们即将到来的销售感兴趣？”，或者，“我不介意 CNN 知道我读了他们的一篇文章。”然而，你需要担心的不仅仅是内容提供商的信标。例如，如果你访问 CNN 的一个页面，上面也有一个脸书的“喜欢”按钮，它很可能有自己的信标，打电话给总部，让他们知道你访问了这个页面。这些信息可以包括日期、时间、IP 地址和浏览器详细信息。我最近读到的一篇 CNN 报道，仅在一页上就有 19 个追踪器！因此，即使你不是脸书或 Twitter 用户，他们也很清楚你(或至少你的 IP 地址)喜欢在互联网上访问什么。

## 如果你仍然不在乎，以下是你应该在乎的原因

嵌入在页面和电子邮件中的信标可被用作社会工程攻击的侦察工具。除了收集您的 IP 地址和时间戳，这些信标还可以报告您的操作系统、主机名和电子邮件地址。如果攻击者可以让你打开一封电子邮件(通过[网络钓鱼](https://www.social-engineer.org/framework/attack-vectors/phishing-attacks-2/))或说服你访问一个网站(通过[视觉)](https://www.social-engineer.org/framework/attack-vectors/vishing/)，那么这些信标将有助于提供你的组织的技术足迹。如果电子邮件地址有效，这还可以为潜在的攻击者提供信息，以及哪些用户更容易受到网络钓鱼和欺诈攻击。您的组织还可以在内部使用信标来跟踪电子邮件是否被转发，或者谁打开了特定的文档。事实上，维基解密报道称[中情局正在利用网络文件中的信标](https://wikileaks.org/vault7/#Scribbles)来追踪敏感文件在整个组织中的传播。

## 你如何最大限度地减少暴露于信标？

电子前沿基金会(EFF)有一个优秀的浏览器插件叫做[隐私獾](https://www.eff.org/deeplinks/2016/12/new-and-improved-privacy-badger-20-here)。专为 Firefox、Opera 和 Chrome 打造；*“**如果同一个第三方域名似乎在三个或更多不同的网站上跟踪你，隐私獾会断定该第三方域名是一个跟踪器，并阻止未来与它的连接。”这是一个易于使用的工具，向您显示嵌入在页面中的每个信标的详细信息。*

至于邮件，确保你的邮件客户端设置为不自动下载图片；并且始终只打开来自可信发件人的电子邮件。Gmail 对 beacons 的解决方案是首先通过他们的服务器提供任何传入的图片。这意味着“打开”仍然会被跟踪，但只会用谷歌的 IP 地址而不是你自己的地址给总部打电话。如果你想进一步阅读，RedAnt 提供了一个非常好的关于 [Gmail 缓存过程的技术分析。](http://redant.com.au/how-we-do/cache-busting-gmail-new-image-caching/)

对于 Microsoft Office 文档，请确保您的信任中心设置为仅在[受保护视图](https://support.office.com/en-us/article/What-is-Protected-View-d6f09ac7-e6b9-4495-8e43-2bbcdbcb6653)中打开文档，并且仅在您完全确定其来源可靠时启用宏。

对于互联网搜索， [DuckDuckGo](https://duckduckgo.com/) 为谷歌提供了一个免费的跟踪选择，只是要注意，在你搜索点击的页面上可能仍然有信标。

几乎不可能 100%避免网络信标，但上述工具可以帮助您将暴露给企业和潜在攻击者的信息量降至最低。浏览愉快！

*来源:*[](https://en.wikipedia.org/wiki/Web_beacon)[](https://wikileaks.org/vault7/#Scribbles)[*https://www . eff . org/deep links/2016/12/new-and-improved-privacy-badger-20-here*](https://www.eff.org/deeplinks/2016/12/new-and-improved-privacy-badger-20-here) [*http://blog . check point . com/2017/04/17/look-files-cloud-looking-back/*](https://blog.checkpoint.com/2017/04/17/look-files-cloud-looking-back/)