# 最近进出口银行邮件服务器漏洞分析| Pentest-Tools.com

> 原文：<https://pentest-tools.com/blog/exim-server-rce-vulnerabilities>

在过去的几个月里，进出口银行的邮件服务器中发现了多个严重漏洞，这些漏洞可能允许攻击者获得远程访问并执行恶意活动: [**CVE-2019-16928**](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16928) 、 [**CVE-2019-15846**](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-15846) 和 [**CVE-2019-10149**](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-10149) 。

在本文中，我们将分析这些漏洞，并试图了解它们的根本原因。此外，我们将找出哪些计算机受到影响，以及为什么进出口邮件服务器需要立即打补丁。

使用下面的链接快速浏览并发现有关这些漏洞的更多信息:

## **1。什么是 Exim？**

根据最近的[邮件(MX)服务器调查](http://www.securityspace.com/s_survey/data/man.201905/mxsurvey.html)，Exim 是在 Unix 系统上使用的最流行的开源邮件传输代理(MTA)软件之一，它被部署在所有互联网邮件服务器的 **57%上。这使得 Exim 成为黑客非常有吸引力的目标。**

作为免费且高度可配置的软件，运行 Exim 的服务器广泛应用于 Linux、Mac OSX 或 Solaris 等操作系统。Shodan 搜索引擎的一份报告总结道，活跃服务器的数量估计超过 500 万。

## **2。Exim 漏洞分析**

### **CVE-2019-16928**

9 月份，Exim 项目的维护者不得不第二次发布了一个紧急补丁，修补邮件服务器中发现的一个严重的安全漏洞。

该漏洞被追踪为 [CVE-2019-16928](https://nvd.nist.gov/vuln/detail/CVE-2019-16928) ，最初由 QAX-A-TEAM 的[报告](https://bugs.exim.org/show_bug.cgi?id=2449)，并被描述为一个*基于堆的溢出漏洞*。这可能会让攻击者对受影响的邮件服务器发起拒绝服务(DoS)攻击或远程代码执行攻击。

该漏洞是由于 EHLO(或 HELO)命令处理程序组件中使用的 string _ v format(string . c 的一部分)中的[基于堆的缓冲区溢出](https://cwe.mitre.org/data/definitions/122.html)(内存损坏)造成的。

基本上，这可能让未经授权的远程黑客通过向目标邮件服务器发送特制的 EHLO 字符串来执行任意系统命令，或者使接收邮件的 Exim 进程崩溃。进出口银行的公告称:

*虽然在这种操作模式下，Exim 已经放弃了它的特权，但是可能存在到达易受攻击代码的其他路径。远程代码执行似乎是可能的。*

如果黑客成功获得目标服务器的访问权限，他可以安装特定的程序，查看、更改或删除敏感数据，甚至创建具有完全用户权限的新帐户。

Exim 维护者已经[发布了](https://www.exim.org/static/doc/security/CVE-2019-16928.txt)针对该漏洞的安全补丁，该补丁包含在 Exim 4.92.3 版本中。作为修复的一部分，一个[概念验证](https://git.exim.org/exim.git/patch/478effbfd9c3cc5a627fc671d4bf94d13670d65f)也可以用来利用这个漏洞。

### **CVE-2019-15846**

上个月在进出口邮件服务器中发现了另一个严重的漏洞。它被称为 [CVE-2019-15846](https://nvd.nist.gov/vuln/detail/CVE-2019-15846) ，它可以让恶意黑客获得系统的本地访问权限(作为无权限用户)，或以根权限远程执行程序。

该漏洞最初是由一位名叫“Zerons”的研究人员在 2019 年 7 月 21 日报告的，随后由 Qualys 的研究人员进行了分析，他们对这一漏洞提出了警告。

Exim 服务器易受攻击的条件是**接受 [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) 连接**并且这“不依赖于 TLS 库，所以 GnuTLS 和 OpenSSL(协议)都受到影响”，[Exim 团队说](https://exim.org/static/doc/security/CVE-2019-15846.txt)。

为了利用该漏洞，Exim 维护人员解释说，攻击者需要“在初始 TLS 握手期间发送一个以反斜杠-null 序列结尾的 SNI(服务器名称指示)”。这可能导致 SMTP(简单邮件传输协议)处理过程中的[缓冲区溢出](https://lists.exim.org/lurker/message/20190906.102039.7eeb3210.en.html)。

在此披露之后，2019 年 9 月 4 日，Exim 团队[警告](https://www.openwall.com/lists/oss-security/2019/09/04/1)系统管理员和用户，其即将发布的安全补丁会影响版本，包括 Exim 4.92.1 版本。

两天后，2019 年 9 月 6 日，Exim 4.92.2 版本的安全更新已经发布，修复了 [RCE 漏洞](https://pentest-tools.com/exploit-helpers/sniper)，并敦促所有用户尽快升级。

Exim 团队证实了一个基本概念验证(POC)的存在，但目前**还没有可用的公开利用**。

### **CVE-2019-10149**

几个月前报道了一个影响 Exim 服务器[的类似严重漏洞。该漏洞被追踪为](https://www.tenable.com/blog/cve-2019-10149-critical-remote-command-execution-vulnerability-discovered-in-exim) [CVE-2019-10149](https://www.cert.be/en/vulnerability-exim-mail-server) ，并被命名为“向导的回归”，由 Qualys 的研究人员在 6 月份对 Exim 进行代码审查时发现。

被披露几天后，安全研究人员[检测到](https://www.helpnetsecurity.com/2019/06/14/exploiting-cve-2019-10149/)黑客试图利用漏洞“通过 SSH 获得对目标 Linux 服务器的完全根访问权限”。

关于这个漏洞，Exim 团队[声明](https://www.exim.org/static/doc/security/CVE-2019-10149.txt)“严重性取决于您的配置。这取决于您的 Exim 运行时配置与标准配置的接近程度。越近越好”。

在默认配置下，本地攻击者可以利用该漏洞，通过发送一封特制的电子邮件，以*根用户*的身份执行命令，“该邮件将由`deliver_message()`函数中的`expand_string`函数解释。”在这种默认配置下，可以远程执行命令。点击阅读更多技术细节。

## **3。受影响的系统**

从(包括)4.92 到(包括)4.92.2 的所有版本都容易受到最新 **CVE-2019-16928** 漏洞的攻击。

另一个关键缺陷， **CVE-2019-15846** ，影响了 Exim 服务器的旧版本:4.80 版本到 4.92.1(含 4 . 92 . 1)。

值得注意的是，4.80 之前的 Exim 版本不受 CVE-2019-15846 的影响，但它们可能容易受到其他严重的 RCE 漏洞的影响，例如去年的 [CVE-2018-6789](https://nvd.nist.gov/vuln/detail/CVE-2018-6789) 远程代码执行漏洞。

**CVE-2019-10149** 存在于从 4.87 到 4.91 的旧的 Exim 版本中。

## **4。缓解**

敦促所有使用 Exim 服务器的系统管理员和家庭用户尽快升级到最新(且已修复)的版本 4.92.3。

然而，对于那些不能下载和安装最新版本的人，Exim 维护人员建议“要求一个包含后端口补丁的版本”。根据请求和我们的资源，我们将支持您进行修复。”

对于 **CVE-2019-15846** 不建议禁用 TLS，即使它确实减轻了漏洞。应该通过对[邮件访问控制列表](https://www.exim.org/exim-html-current/doc/html/spec_html/ch-access_control_lists.html) (ACL)配置一些规则来缓解，这样可以防止攻击。

Exim 建议在您的邮件 ACL(ACL*SMTP*mail main config 引用的 ACL)前添加以下两行代码片段:

```
deny    condition = ${if eq{\\}{${substr{-1}{1}{$tls_in_sni}}}}
```

```
deny    condition = ${if eq{\\}{${substr{-1}{1}{$tls_in_peerdn}}}}
```

如果您的服务器属于暴露于 **CVE-2019-15846 漏洞**的服务器，我们建议升级到[最新版本](http://exim.org/index.html)。

## **5。最终想法**

打补丁仍然是一项重要的安全措施，用于防止出现上述严重的安全缺陷。黑客不会停止利用漏洞，这可能会让他们完全控制你的系统和敏感数据。