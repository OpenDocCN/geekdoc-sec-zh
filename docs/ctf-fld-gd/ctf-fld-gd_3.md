# 三、漏洞利用程序的构建

# 3.1、二进制的漏洞利用（1）

## 3.1、二进制的漏洞利用（1）

二进制的漏洞利用是破坏编译程序的过程，令程序违反自身的可信边界从而有利于你——攻击者。本部分中我们将聚焦于内存错误。通过利用漏洞来制造软件内存错误，我们可以用某种方式重写恶意程序静态数据，从而提升特定程序的权限（像远程桌面服务器）或通过劫持控制流完成任意操作和运行我们所用的代码。

如果你尝试在已编译的 C 程序中找 bug，知晓你要找的东西是很重要的。从认识你发送的数据被程序用在什么地方开始，如果你的数据被存储在一个缓冲区中，要注意到它们的大小。编写没有错误的 C 程序是非常难的，[CERT C Coding Standard](https://www.securecoding.cert.org/confluence/display/seccode/CERT+C+Coding+Standard)手册汇总了许多错误出现的方式。对[commonly misused APIs](http://stackoverflow.com/questions/4588581/which-functions-in-the-c-standard-library-commonly-encourage-bad-practice)稍加注意是可以加快了解的捷径。

一旦一个漏洞被确认，它就可以被用来威胁程序的完整性，然而，有各种不同的方式可以达到这个目标。对于像 Web 服务器这样的程序，获取另一个用户的信息可能是最终目标。另一方面，修改你的权限可能是有用的，比如修改一个本地用户权限为管理员。

**课程**

第一套课程是《Memory Corruption 101》，提供了 Windows 环境下缓冲区溢出利用一步一步的解释和相关背景。第二套课程《Memory Corruption 102》，涵盖了更多高级主题，包括 Web 浏览器漏洞利用。这两套课程都是针对 Windows 的例子，但技术和过程可以用于使用 x86 指令集的其他操作系统。记住，当你处理 UNIX/Linux 二进制时，函数名称和有时候的调用约定是不一样的。

*   [Memory Corruption 101](http://vimeo.com/31348274)

*   [Memory Corruption 102](http://vimeo.com/31831062)

**工具**

我们建议使用 GDB 来调试该部分的挑战题，因为它们都是在 32 位 Linux 下编译的，然而，GDB 更适合用来调试源代码，而不是没有标识符和调试信息的纯粹二进制程序。诸如[gdbinit](https://github.com/gdbinit/gdbinit)、[peda](https://code.google.com/p/peda/)和[voltron](https://github.com/snarez/voltron)可以使 gdb 在调试无源码的程序时更有用。我们建议在你的 home 目录下创建一个至少包含以下命令的.gdbinit 文件：

```
set disassembly-flavor intel

set follow-fork-mode child 
```

**挑战工坊**

为了运行这些挑战题，你需要安装[Ubuntu 14.01（32-bit）](http://www.ubuntu.com/download/desktop/thank-you?country=US&version=14.04&architecture=i386)虚拟机。我们建议使用[VMware Player](https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_player/6_0)，因为它免费且支持性很好。在你运行虚拟机后，打开终端并运行 sudo apt-get install socat 来安装 socat。

在此次挑战工坊中共有三道挑战题，当你在 git 中 clone 该手册的 repository 后，每道题都包含在其目录中，每道题的最终目标是操纵执行流以读取 flag。对于每道题，请尝试将反汇编转换为 C 代码。在尝试之后，你可以通过查看提供的实际 C 源码来确认你的猜测，然后，尝试利用漏洞来读取 flag。

**挑战题：Easy**

确认目录中 easy 程序的 flag。当你执行 easy 后，它会在 12346 端口监听指令。

**挑战题：Social**

和 easy 类似，确认目录中 social 程序的 flag 和作为 social 程序的 host.sh。当你执行 social 后，它将在 12347 端口监听指令。

**资源**

*   [Using GDB to Develop Exploits – A Basic Run Through](http://www.exploit-db.com/papers/13205/)

*   [Exploiting Format String Vulnerabilities](https://trailofbits.github.io/ctf/exploits/references/formatstring-1.2.pdf)

*   [Low-level Software Security: Attacks and Defenses](https://trailofbits.github.io/ctf/exploits/references/tr-2007-153.pdf)

# 3.2、二进制的漏洞利用（2）

## 3.2、二进制的漏洞利用（2）

在该部分内容中，我们继续可利用漏洞的本地应用检查之路，并关注使用返回导向编程（ROP）来达到此目的。ROP 是在代码结尾的返回指令中整合现有可执行片段的过程。通过创建这些“玩意儿”地址链可以在不引入任何新代码的情况下写新程序。

记住，在可利用程序的漏洞识别方法上你需要灵活应变。有时候有必要在漏洞利用开发过程中对一个漏洞多次利用。有时，你可能仅想用 ROP 来让你的 shellcode 执行，其他情况下，你可能想在 ROP 中完整写一个攻击载荷。偶尔，内存布局能使非常规的漏洞利用方法可行，例如，你可曾考虑过用 ROP 来构造一个不受控的格式化字符串漏洞？

**课程**

本部分的课程将讨论返回导向编程（ROP）和绕过数据不可执行保护的代码重用。这些在漏洞利用细节上会更具体，且基于上部分所讨论的内容。

*   [Return Oriented Exploitation](http://vimeo.com/54941772)

*   [Payload already inside data re-use for ROP exploits](http://www.youtube.com/watch?v=GIZziAOniBE)（特指 Linux）

**挑战工坊**

和前一个部分的挑战一样，在你 clone repository 后文件夹中有两个可执行文件。每个程序的漏洞利用都需要使用返回导向编程以读取 flag。此次的挑战题没有提供源代码的访问。你需要对二进制程序进行逆向工程来发掘漏洞，并需要将漏洞利用的技术。同样，请使用相同的 Ubuntu 14.04（32-bit）虚拟机。

**挑战题：brute_cookie**

运行 bc 程序，它会监听 12345 端口。

**挑战题：space**

运行作为 space 程序的 host.sh，它会监听 12348 端口。

**挑战题：rop_mixer**

运行作为 rop_mixer 程序的 host.sh，它会监听 12349 端口。

**工具**

参考前面所提到的工具。如果你还没有准备，你需要熟悉一下*NIX 的 binutils 套件。像 readelf、strings、objdump、objcopy 和 nm 都是常用的有用工具。请使用软件包管理器和帮助页面安装和阅读它们的使用。

有几个现有的工具专门用来创建可重用代码的漏洞，它们比一般的反汇编器更加专业，因为它们会寻找合适的可执行代码段作为 ROP 目标（在指令、.rodata 等中）。

*   [RP](https://github.com/0vercl0k/rp)

*   [ROPGadget](https://github.com/JonathanSalwan/ROPgadget)

*   [BISC](https://github.com/trailofbits/bisc/)——适用于课程的简单工具

**资源**

*   [x86-64 buffer overflow exploits and the borrowed code chunks exploitation technique](https://trailofbits.github.io/ctf/exploits/references/no-nx.pdf)

*   [Surgically returning to randomized lib(c)](https://trailofbits.github.io/ctf/exploits/references/acsac09.pdf)

*   [Extensive security reference](https://code.google.com/p/it-sec-catalog/wiki/Exploitation)

*   [Dartmouth College: Useful Security and Privacy links](http://althing.cs.dartmouth.edu/secref/resources/buffer_overflows.shtml)

*   [Corelan Team Blog](https://www.corelan.be/index.php/articles/)

# 3.3、Web 应用的漏洞利用

## 3.3、Web 应用的漏洞利用

该部分承接前面的 Web 应用审计部分。在该部分中我们将关注于漏洞的利用，在结束时你应该能够熟练地识别和利用[OWASP Top 10](https://www.owasp.org/index.php/Top_10_2013-Top_10)。

**课程**

前面的内容中我们已经介绍了 Web 安全的基础部分，所以现在在该部分中我们可以更深一步到一些能够获得更大效果的合适工具。学习掌握 Burp Suite 和 Chrome 开发者工具能够更好的理解和你交互的应用程序。BeEF 是一个 XSS 代理的例子，通读它的源码学习它怎样工作将会受益匪浅。

*   [Burp Suite Training](http://www.youtube.com/watch?v=L4un5IppoY4)

*   [Chrome Dev Tools](http://www.youtube.com/watch?v=BaneWEqNcpE)

*   [From XSS to reverse shell with BeEF](https://vimeo.com/82779965)

**挑战工坊**

你被指派对一个小而糙的 Web 应用程序[Gruyere](http://google-gruyere.appspot.com/)进行审计。Gruyere 是托管于 Google 的应用，它能够进行许多类型的 Web 方面漏洞利用练习，包括 XSS、SQL 注入、CSRF、路径穿越和其他。对于每一个挑战你都能够找到提示、漏洞和进行漏洞修复的方法。

**参考**

*   [Google Chrome Console](https://developers.google.com/chrome-developer-tools/docs/console)

*   [OWASP Top 10 Tools and Tactics](http://resources.infosecinstitute.com/owasp-top-10-tools-and-tactics/)

*   [The Tangled Web: Chapter 3](http://www.nostarch.com/download/tangledweb_ch3.pdf)

*   [PHP Primer](http://www2.astro.psu.edu/users/sdb210/documents/phpprimer_v0.1.pdf)

**工具**

SQLMap 和 BeEF 都是强大且有趣的工具，但是对于挑战题基本上用不着。如果你坚持要使用 BeEF，请不要尝试拦截进行应用审计的其他用户。

*   [Burp Suite](http://portswigger.net/burp/download.html)

*   [SQL Map](http://sqlmap.org/)

*   [The Browser Exploitation Framework (BeEF)](http://beefproject.com/)