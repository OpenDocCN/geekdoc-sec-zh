# 二、漏洞挖掘

# 2.1、源代码审计

## 2.1、源代码审计

这个部分是关于熟悉应用程序编译为本地代码时显现的漏洞。对一门编译语言编写应用程序时的精准和完整理解，在没有学习编译器怎样转换源代码为机器语言和处理器怎么执行代码前是无法达到的。一种简单的获得这些转换经验的方式是通过逆向工程你自己的代码或源码可见的项目。在这个部分结束时你将会识别用诸如 C 和 C++编译语言编写的常见漏洞。

大型软件包由于使用第三方软件库导致漏洞普遍存在。常见的例子包括像 libxml、libpng、libpoppler 和用来解析已编译文件格式和协议的 libfreetype 等这样的库。这些库中的每一个都曾在历史上被发现过易于攻击的漏洞。即便当新的版本发布之后，也无助于绝大多数软件的未更新，在这些情况下很容易发现易于发现的已知漏洞。

**课程**

*   [Source Code Auditing I](http://vimeo.com/30001189)

*   [Source Code Auditing II](http://vimeo.com/29702192)

**挑战工坊**

为了锻炼你的技能，我们建议通过一个故意留有漏洞的程序过一遍尽可能多漏洞发掘之旅，然后转移到实际应用中做同样的事情。

Newspaper 应用是一个用 C 语言编写的小型服务应用，允许认证的用户读取和写入文章到一个远程文件系统。Newspaper 编写的方式使得它对于许多不同的攻击都有漏洞可以利用。你应该能够通过源码发现其中至少 10 个 bug 或可能存在的漏洞。

*   [Newspaper App](https://trailofbits.github.io/ctf/vulnerabilities/source_workshop/news_server.c)

*   [Newspaper App Installer](https://trailofbits.github.io/ctf/vulnerabilities/source_workshop/news_install.sh)

Wireshark，无论怎样，是一款从 1998 年开始持续开发的行业标准级别的网络协议分析器。相比漏洞诸多的 Newspaper 应用，Wireshark 的漏洞少之又少。查看[wireshark 安全页面](http://wireshark.org/security)，找到一个协议解析器的名字并测试是否你可以在没有查看漏洞细节的情况下发现漏洞。解析器位于/epan/dissectors/目录。

*   [Wiireesakk 1.8.5](http://www.wireshark.org/download/src/all-versions/wireshark-1.8.5.tar.bz2)

**工具**

当进行源码审计时，使用一款用户分析和引导代码库的工具是很有帮助的。下面是两款工具，Source Navigator 和 Understand，通过收集和展现相关数据关系、API 使用、设计模式和控制流的信息来帮助分析员快速熟悉代码。一个有用的对比工具的例子同样列在下方。其中一个免费、开源的审计工具例子是 Clang Static Analyzer，该工具可以帮助你在常见 API 和漏洞编码形式中跟踪编程错误。

*   [Source Navigator](http://sourcenav.sourceforge.net/)
*   [Scitools Understand](http://www.scitools.com/)
*   [Meld](http://meldmerge.org/)
*   [Clang Static Analyzer](http://clang-analyzer.llvm.org/)

**资源**

要确定你对分析目标所用的编程语言非常地熟悉。发现漏洞的审计员相比初始开发软件的程序员要更加熟悉语言和代码库。在一些案例中，这种级别的理解可以简单通过关注可选编译器警告或通过第三方分析工具帮助跟踪常见编程错误而达到。计算机安全等同于计算机精通。对你的目标没有严苛的理解，也就没有抵御攻击的希望。

*   [Essential C](https://trailofbits.github.io/ctf/vulnerabilities/references/EssentialC.pdf)——C 语言编程入门

*   [TAOSSA Chapter 6: C Language Issues](https://trailofbits.github.io/ctf/vulnerabilities/references/Dowd_ch06.pdf)——强烈推荐阅读

*   [Integer Overflow](http://en.wikipedia.org/wiki/Integer_overflow)

*   [Wireshark Security](https://wireshark.org/security/)——许多漏洞的例子

*   [Gera's Insecure Programming by Example](http://community.coresecurity.com/~gera/InsecureProgramming/)——小型漏洞 C 程序的例子

# 2.2、二进制审计

## 2.2、二进制审计

现在你已经深入到原生层，这是你撕扯下所有遮掩后的软件。今天我们所关注的原生代码形式是 Intel X86 下的 32 位代码。Intel 处理器从上世纪 80 年代开始在个人计算机市场有着强劲的表现，现在支配着桌面和服务器市场。理解这些指令集可以帮助你以内部视角看到程序每天是如何运行的，也可以在你遇到诸如 ARM、MIPS、PowerPC 和 SPARC 等其他指令集时提供一种参考。

这部分内容我们将要逐渐熟悉原生层和开发策略，以理解、分析和解释本地代码。在该部分结束时你应该能够完成一项“逆向编译”——从汇编分段到高级语言状态——以及处理过程、派生意义和程序员意图。

**课程**

学习 X86 初看起来令人生畏且需要一些专业性的学习才能掌握。我们建议阅读《深入理解计算机系统》的第三章学习 C 程序是怎样编译成机器指令的。当你有了一些基础的、这种过程的应用知识，就在手边随时备着像弗吉尼亚大学 x86 汇编指南这样的参考指南。我们还发现了来自 Quinn Liu 的系列视频作为快速介绍。

*   《深入理解计算机系统》第三章: [Machine-Level Representation of Programs](https://picoctf.com/docs/asmhandout.pdf)

*   [x86 Assembly Guide](http://www.cs.virginia.edu/~evans/cs216/guides/x86.html)

*   [Introduction to x86 Assembly](https://www.youtube.com/watch?v=qn1_dRjM6F0&list=PLPXsMt57rLthf58PFYE9gOAsuyvs7T5W9)

**挑战工坊**

下面的程序都是“二进制炸弹”。逆向工程这些 Linux 程序并确定输入序列就可以“拆除“炸弹。炸弹的每个连接层关注于原生代码的不同层面。例如，CMU 实验室中的程序（CMU Binary Bomb Lab）中你会看到不同数据结构（链表、树）以及控制流结构（切换、循环）在原生代码层面怎样表现的。在逆向这些程序时你可以发现将程序执行流程转换为 C 或者其他高级语言的有用之处。

你应该将目标聚焦于解决这两个实验室程序的八个段。CMU 炸弹程序有一个隐藏段，RPI 炸弹程序有一个段包含有内存错误，你能找到并解决么？

*   [CMU Binary Bomb Lab](http://csapp.cs.cmu.edu/public/bomb.tar)

*   [RPI Binary Bomb Lab](http://www.cs.rpi.edu/academics/courses/spring10/csci4971/rev2/bomb)

**工具**

处理原生代码的两个至关重要的工具是调试器和反汇编器。我们建议你熟悉下行业标准反汇编工具：IDA Pro。IDA 会分割代码为独立的块以对应程序源码的定义函数。每个函数进一步被分割为修改控制流的指令定义的“基础块”。这样很容易一眼识别循环、条件和其他控制流指令。

调试器允许你与设置了断点的运行中代码进行交互和状态检查，以及内存检查和寄存器内容查看。你会发现如果你的输入没有产生预期的结果，这些查看功能是很有用的，但是一些程序会使用反调试技术在调试时改变程序行为。对于大多数 Linux 系统来说 GNU 调试器（gdb）是标准的调试工具。gdb 可以通过你所用 Linux 版本的软件包管理器获得。

*   [IDA Pro Demo](https://www.hex-rays.com/products/ida/support/download_demo.shtml)

*   [gdb](http://www.sourceware.org/gdb/)

**资源**

已经有许多好的资源用来学习 x86 汇编和 CTF 题目中的技巧。除了以上资源，x86 维基手册和 AMD 指令集帮助都是更加完整的参考供你参考（我们发现 AMD 帮助手册没有 Intel 帮助手册那么吓人）。

*   AMD64 程序员帮助手册: [General-Purpose and System Instructions](http://amd-dev.wpengine.netdna-cdn.com/wordpress/media/2008/10/24594_APM_v3.pdf)

*   [x86 Assembly Wikibook](https://en.wikibooks.org/wiki/X86_Assembly)

*   [Computer Systems: A Programmer's Perspective](http://csapp.cs.cmu.edu/) (《深入理解计算机系统》)

一些逆向工程工具使用起来和汇编语言自身一样复杂。下面列出的是常用的命令行工具的命令表单：

*   [gdb Quick Reference](http://www.refcards.com/docs/peschr/gdb/gdb-refcard-a4.pdf)

*   [IDA Quick Reference](https://www.hex-rays.com/products/ida/support/freefiles/IDA_Pro_Shortcuts.pdf)

*   [WinDBG x86 Cheat Sheet](https://trailofbits.github.io/vulnerabilities/references/X86_Win32_Reverse_Engineering_Cheat_Sheet.pdf)

最后，许多 CTF 挑战会使用反调试技术和反汇编技术隐藏或混淆目标。上面的炸弹程序就使用了其中的几种技术，但是你可能想要更全面的参考资料。

*   [Linux anti-debugging techniques](http://vxheavens.com/lib/vsc04.html)

*   [The "Ultimate" Anti-Debugging Reference](http://pferrie.host22.com/papers/antidebug.pdf)

# 2.3、Web 应用审计

## 2.3、Web 应用审计

欢迎来到 Web 渗透部分，在这个部分中将会熟悉 Web 应用中发现的常见漏洞。在该部分结束时你应该能够使用各种测试方法和源码级别审计识别基于 Web 的常见应用漏洞。课程资源中将会给出对挑战工坊资源进行成功审计的所有工具。

**课程**

*   [Web Hacking Part I](http://vimeo.com/32509769)

*   [Web Hacking Part II](http://vimeo.com/32550671)

**挑战工坊**

为了锻炼你的技能，我们建议你实践一遍 Damn Vulnerable Web App（DVWA）和 Sibria Exploit Kit 的漏洞发掘与利用。DVWA 是基于 PHP 并汇总了各类漏洞的一套测试环境，在其中能够看到 Web 应用中许多常见的错误。Siberia Exploit Kit 是一个被许多犯罪份子用来完成大量攻击的“犯罪套件”，它包括了一个浏览器利用包和一个用来管理受害主机的控制面板。Siberia 包含的几种基于 POST 的身份认证漏洞允许攻击者获得管理员权限并接管服务器所在的主机。

下载并在 VMware 虚拟机中运行[OWASP Broken Web Apps](https://code.google.com/p/owaspbwa/)开始此次挑战工坊。BWA 包含许多用来安全测试的 Web 应用，其中包括 DVWA。如果你搞定了 DVWA，那么放轻松再试试其他 Web 应用漏洞！尝试审计 Siberia 的源码来找寻漏洞，要将注意力集中在 PHP 输入的代码上。

*   [OWASP Broken Web Apps](https://code.google.com/p/owaspbwa/)

*   [Siberia Crimeware Pack](https://trailofbits.github.io/ctf/web/workshop/siberia.zip) (口令: infected)

**工具**

Burp Suite 是一个用来安全测试的本地 HTTP 代理。Burp Suite 是针对 Web 渗透测试人员开发的，并将许多常见功能集成在一个 GUI 界面中。免费版本的可用功能足以完成上面的挑战和许多其他 Web 安全挑战。

*   [Burp Suite](http://portswigger.net/burp/download.html)

**资源**

许多简单的测试方法和常见的 Web 应用漏洞都可以在该部分的挑战题中遇到。在做这些挑战题前请先确定你理解了 HTTP、HTML 和 PHP 的基础知识。

*   [OWASP Top 10 Tools and Tactics](http://resources.infosecinstitute.com/owasp-top-10-tools-and-tactics/)

*   [The Tangled Web: Chapter 3](http://www.nostarch.com/download/tangledweb_ch3.pdf)

*   [PHP Primer](http://www2.astro.psu.edu/users/sdb210/documents/phpprimer_v0.1.pdf)