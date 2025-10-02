# 七、参与者

该手册的编写建立在许多艰难工作之上，绝大多数难点是在其他地方。未经允许不得重新发布这些文件，因为该手册尚在更新过程中，我们非常感谢和感激大家的支持。

所以，读者们，当你完成一些 CTF 比赛准备参加更多的比赛，请参与到该手册的编写中来。那些有着雄心壮志的人们或许知道一些需要如此技能的人。

*   [Andrew Ruef](http://www.mimisbrunnr.net/~munin/blog/) 提供了倡议和 PPT。

*   [Evan Jensen](https://github.com/wontonSlim) 开发和优化了几乎有所的课程

*   [Nick Anderson](https://github.com/PoppySeedPlehzr) 校对了所有课程并进行了许多改进

*   [Alex Sotirov](http://www.phreedom.org/) 贡献了 ROP 课程并提供了反馈意见

*   [Jay Little](https://twitter.com/computerality) 重新审阅了二进制漏洞利用部分的内容

*   [Brandon Edwards](https://twitter.com/drraid) 贡献了源码审计课程和 newspaper 应用

*   [Marcin W](https://twitter.com/marcinw) 和[Gotham Digital Science](http://www.gdssecurity.com/) 贡献了 Web 安全课程

*   [Dino Dai Zovi](http://www.theta44.org/main.html) 贡献了漏洞利用课程的介绍

如果你对这样的课程感兴趣，请参考[NYU Poly](http://engineering.nyu.edu/academics/departments/computer/)。他们专注于网络安全且经常通过他们的[Hacker in Residence](http://www.isis.poly.edu/hackers-in-residence)活动与我们合作。

**参与进来**

如果你想参与该手册的编写，只需简单地将你新的 markdown 提交到 master 分支即可，我们可以从那里获取到提交。Gitbook 有一个出色的[编辑器](https://github.com/GitbookIO/editor/releases)用来帮助你预览更改。我们一直在找寻新或精妙的内容，所以请给我们提供建议！

**Gitbook 的使用**

《CTF 领域指南》是用[Gitbook](https://github.com/GitbookIO/gitbook)创建的，它是一个用 Git 和 Markdown 创建电子书的命令行工具。你可以用 Gitbook 将《CTF 领域指南》导出为 PDF 格式、一个 eBook 或一个单独可打印的 HTML 页面。在安装 Gitbook 和它的少量插件前请确保你的电脑上安装有[NodeJS](http://nodejs.org/)和 npm：

```
npm install gitbook gitbook-plugin-ga gitbook-pdf ebook-convert -g 
```

在 Gitbook 安装后，你可以在书籍目录内运行下面的任一命令：

*   创建一个互动、静态站点：`gitbook build ./myrepo`

*   创建一个独立页面站点：`gitbook build ./myrepo -f page`

*   创建一个 PDF 文件：`gitbook pdf ./myrepo`（需要[gitbook-pdf](https://github.com/GitbookIO/gitbook-pdf)）

（全文待续）