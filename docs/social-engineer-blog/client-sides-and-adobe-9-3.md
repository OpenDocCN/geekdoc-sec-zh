# 客户端和 Adobe 9.3

> 原文：<https://www.social-engineer.org/social-engineering/client-sides-and-adobe-9-3/>

感谢我们的好朋友杜基，他给了我们一个关于 Adobe 9.3 新漏洞的 CVE 的链接。好吧，我应该说它不适合 9.3，但它声明:
漏洞利用在禁用 Adobe Javascript 的情况下工作。
已测试:在 Adobe Reader 9.1/9.2/9.3 操作系统 Windows XP(SP2、SP3 任何语言)上成功测试，也可与 Adobe 浏览器插件配合使用

一个名叫 [villy 的黑客制作了一个 python 脚本](https://bugix-security.blogspot.com/)，它将创建一个 pdf 文件，在装有最新版本 Adobe Reader 的 WinXP SP2 机器上启动 calc.exe，即使关闭 Java。

在玩了它之后，我们用一个 Windows 反向外壳替换了外壳代码，然后在一个完全补丁的系统上进行测试！砰——又是炮弹。

我们把 PDF 文件上传到 Virus Total，然后一个惊人的 0/42 被返回，这还是在我们使用 Shakata Ganai 对它进行编码之前。

当然，我们记录了这次冒险，并在我们网站的资源页面上发布了一个名为[全新 Adobe 9.3 Exploit](https://www.social-engineer.org/blog/resources/) 的新视频

敬请期待更多精彩内容。