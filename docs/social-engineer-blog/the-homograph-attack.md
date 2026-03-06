# 同形异义攻击

> 原文：<https://www.social-engineer.com/the-homograph-attack/>

同形异义攻击。想象一下，坐在你的电脑前，当你查看你的 r 电子邮件 ， 你碰到 一条信息 广告 大量关于 苹果 iPad 。你一直想得到一个 ，所以你可以把你的旧手机给你的孩子 。于是你 点击链接，就到了[【https://www.apple.com】](https://www.xn--80ak6aa92e.com/)。 

![The Homograph Attack](img/29f40de6960f6ee5eb0c276bf123e249.png)

现在你查一下； 安全吗？你看 网址 中的绿色锁和 https 。 好了，安全了。 是真的吗？https://www . apple . com 就是你在浏览器里看到的。 所以，它一定是真实的。  

如果你仔细看，你会看到[https://www . xn-80ak6aa92e.com/](https://www.xn--80ak6aa92e.com/)而不是[https://www.apple.com](https://www.apple.com)。  

这怎么可能呢？  

### 是魔法吗？

据研究人员旭东、【郑、【如果你使用的是 Chrome ( 预版本 58.0.3029.81 )、Firefox，或者 Opera 、 的网页浏览器，你就有被这种攻击打个措手不及的危险。但是 为什么是 ？  

It comesdown这些浏览器如何显示 Unicode 字符。简而言之，“Unicode 为每个字符提供了一个唯一的编号，不管是什么平台，什么程序，什么语言”([Unicode Consortium](https://unicode.org/standard/WhatIsUnicode.html)))。 换句话说，每种语言字母表中的每个字符都会被赋予一个不同的数字 ，用+十六进制 (其中 U+表示 Unicode，十六进制为十六进制表示) 。所以，如果你把希腊字母、西里尔字母、亚美尼亚字母、、拉丁字母等看似相似的字母组合起来。 字母表 ，你可以创建一个视觉上相似的域名作为网址，如在 apple.com 的例子中。(如果你想随便玩玩就明白我的意思，访问这里[https://www.irongeek.com/homoglyph-attack-generator.php](https://www.irongeek.com/homoglyph-attack-generator.php))

这种安全问题并不新鲜。早在 2001 年，叶夫根尼 加布利洛维奇 和亚历克斯 贡特马克 在他们的论文 [“同形异义攻击”中就发现了这一点](http://www.gabrilovich.com/publications/papers/Gabrilovich2002THA.pdf) 由于一些 Unicode 字符看起来可能相同 ，为了扩大域名中允许的字符数量，ICANN 使用了 Punycode 而不是 Unicode。Punycode 由“ xn —”表示，后面是对 Punycode 的 Unicode 翻译。起初，浏览器默认读取 Punycode URL 并将其转换回 Unicode(这产生了另一个安全问题) 。 你现在可以 用希腊语、西里尔语或者另一种语言 生成一个 域名，然后 接受Punycode翻译，并使用它作为链接 。 W 当它在浏览器中显示时 会将 钓鱼网站伪装成合法 网站 通过显示 Unicode 翻译 。 如 中的例子:[https://www . xn-80ak6aa92e.com/](https://www.xn--80ak6aa92e.com/)vs[https://www.apple.com](https://www.apple.com)。  

(现在，不要去买域名)  

### 你如何保护自己？

像 Edge、IE、Safari 和其他浏览器已经解决了这个问题，如果 URL 包含多种语言的字符，则使用过滤器来显示 Punycode URL，而不是 Unicode。如果您的 d 默认 语言 设置为除 英语、 之外的另一种语言 ，浏览器 将以 Unicode 显示该语言。Chrome (pre- 版本580 . 3029 . 81)，Firefox，Opera 都没做过 这个还没 。他们仍然 将双关语 翻译成拉丁字母字符，然后显示出来。T32
T34】

但是 如果 你用火狐，你能怎么办？一切都还没完。即使 Mozilla 仍然没有发布 更新来修复 这个问题， 您可以执行以下操作:  

第一步:在火狐的地址栏中输入 about:config 并点击回车。

第二步:键入 网络。IDN _ 秀 _punycode 在搜索栏 中双击该选项设置为“真”。  

如果使用 Chrome， 这个已经在稳定的 58 更新中解决了 。所以 ， 只要检查你的版本，确保你是最新的更新。  

如果你使用 Opera… w ell ， you [点击 URL](https://forums.opera.com/discussion/1883468/can-opera-be-made-to-show-punycode/p1) 旁边的 锁图标，会显示一个带有 Punycode 的窗口(见下图)。 但仅此而已(所以，如果你想检查每一个网址，看看 Punycode，以确保它是真实的交易，这就是如何) 。

![image1](img/2c1cfbf3c344771b0b2cba8b70002d11.png)

这一切都归结于意识到并知道该做什么。这里有一个简短的清单来确保你的安全:

*   确保您没有点击电子邮件或文档中的链接
*   确保 Chrome 和你所有的浏览器都是最新的
*   如果你使用 Firefox，那么做上面列出的建议更改  

下次再见，注意安全。