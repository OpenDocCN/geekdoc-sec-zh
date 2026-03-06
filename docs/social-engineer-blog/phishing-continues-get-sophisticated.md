# 网络钓鱼变得越来越复杂

> 原文：<https://www.social-engineer.com/phishing-continues-get-sophisticated/>

网络钓鱼变得越来越复杂。最近，我们看到了几起复杂的网络钓鱼攻击，它们试图获取目标的 Gmail 凭据。下面，我们将对每一项进行快速分析，并告诉你如何在未来继续保护自己。虽然下面的攻击示例是针对 Gmail 帐户的，但这些攻击也可以很容易地在许多不同的平台上使用。

![Phishing Continues to Get More Sophisticated](img/5157df1d6d3b990d7d8af5f70e924259.png)

### 通过混淆进行网络钓鱼

以下老练的网络钓鱼者正在四处传播。像任何其他网络钓鱼一样，它可能看起来来自可信的人(或者实际上可能来自被黑客攻击的可信的人)，并且通常包含您想要查看的附件图像。我们中的许多人都知道要小心谨慎，在输入凭证之前要检查 URL，对吗？好吧，在这种情况下，它并不是那么一成不变的，它成功地愚弄了精通网络钓鱼攻击的用户。

Wordfence 的 Mark Maunder 解释了这种方法是如何愚弄技术娴熟的用户的。*你点击图片，期待 Gmail 给你一个附件的预览。相反，会打开一个新标签，Gmail 会提示您重新登录。你扫了一眼地址栏，看到**accounts.google.com**在那里。看起来是这样的…*

![picture2](img/2b6616da563f7a54c24b72af03797009.png)

在忙碌的一天，在移动设备上，或者对当时分心的人；快速浏览一下网址并不会发出正常的危险信号。地址前没有*不安全*警告，通常在任何非 SSL 站点都会发出警告。此外，它似乎没有指向一个假的网址，如 security-google.co

 **登录后，您会看到一个真实的文档，但是您的帐户凭证已经被破坏了。当他们看到 Gmail 登录页面时，许多人错过了打开凭证收集页面的混乱代码。如果用户查看完整的 URL(通过滚动所有的空白)，他们会看到这不是一个合法的登录页面。

![picture3](img/8f93707749e64240add1674998fed891.png)

### 通过建立融洽关系进行网络钓鱼

最近，[另一个类似的网络钓鱼攻击](https://www.forbes.com/sites/thomasbrewster/2017/02/14/safeena-malik-qatar-fake-cyberespionage-hacking-campaign/#1f3c927b12af)是通过另一个假冒的 Gmail 登录页面窃取凭据。不同的是，这些网络钓鱼更有针对性，攻击者在发动攻击之前与目标建立了融洽的关系。攻击者为一个名叫 Safeena Malik 的女性创建了一个完整的在线存在。她有一个 LinkedIn 帐户，有 500 多个联系人，还有脸书和谷歌帐户，在网络钓鱼发送之前，这些帐户都是活跃的，并定期与目标通信。

在“信任”建立了一段时间后，Safeena 发送了一封电子邮件，承诺附件中包含有价值的信息。同样，为了查看文件，目标会被提示登录一个伪造的 Gmail 页面，在那里他们的证书被盗。然而，由于这来自一个看起来合法的来源，许多用户可能在登录前都懒得验证这个 URL。

### 不要成为这些攻击的受害者

*   在点击和/或输入信息之前，一定要仔细检查网址。它可能看起来来自可信的来源，但该来源可能已被破坏或完全是伪造的。最佳做法是手动导航到一个已知的好的 URL 进行登录。
*   在任何可用的帐户上激活双因素身份验证(2FA)。攻击者可能会通过这些攻击之一获得您的密码，但 2FA 将有助于防止他们登录和窃取数据或接管您的帐户。
*   不要跨站点重复使用密码。如果你的密码确实被泄露了，至少它不能被用来入侵其他账户。

*来源:*
*[https://www . Forbes . com/sites/Thomas Brewster/2017/02/14/safe ENA-Malik-Qatar-fake-网络间谍-黑客-campaign/# 1 F3 c 927 b 12 af](https://www.forbes.com/sites/thomasbrewster/2017/02/14/safeena-malik-qatar-fake-cyberespionage-hacking-campaign/#1f3c927b12af)*
[*https://www . word fence . com/blog/2017/01/Gmail-phishing-data-uri/*](https://www.wordfence.com/blog/2017/01/gmail-phishing-data-uri/)**