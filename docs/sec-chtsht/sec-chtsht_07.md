# 密码存储备忘单

> 原文：[Password Storage Cheat Sheet](https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet)
> 
> [密码存储备忘单](http://cheatsheets.hackdig.com/?7.htm)

## 介绍

媒体几乎每天都会报道一些窃取密码的新闻。媒体报道的密码窃取大多数是密码存储方案泄露、存储方案存在漏洞。通常会有大量的凭据遭到破坏，从而影响大量的 WEB 站点或者其他应用程序。本文提供了一个正确存储密码、密码问题及答案和类似凭据信息的指导。合适的存储方案能防止证书被盗、被泄露和恶意使用。信息系统通过各种保护形式来存储密码和其他凭据。常见的漏洞让窃密者能通过 SQL 注入等攻击向量来窃取被保护的密码。受保护的密码也有可能被攻击者通过其他形式（如日志、转储和备份文件等）窃取。

这份指导会教你如何防止凭据被盗，但是大部分内容是关于防止密钥泄露的。这个指导同样能帮助你设计抵御用户凭据被盗或防止窃密者访问凭据信息的系统。你可以参阅 [`goo.gl/Spvzs`](http://goo.gl/Spvzs) 以获取更多信息。

## 指导

### 不限制字符集和设置最大凭据长度

一些系统会做如下限制：1)特定的字符类型。2)系统能够接受的凭据长度（因为这有助于防止 SQL 注入、跨站脚本、命令执行和其他形式的注入攻击）。这些善意的限制让系统更易于受到暴力破解这类的攻击。

不要在登陆入口或存储凭据时使用过短、没有长度限制的、只限制字符集或编码的密码。除此之外，还要使用编码、转码、隐藏、忽略和其他最佳实践来消除注入攻击的风险。

合理的长密码的长度是 160，太长的密码可能会让系统出现 DDOS 攻击漏洞[1].

### 使用一个强大的加密凭据专用的盐(salt)

盐是固定长度的、用于加强密码学可靠性的随机值。将凭据数据加入盐中并将其作为保护函数的输入。形式如下：

```
[protected form] = [salt] + protect([protection func], [salt] + [credential]); 
```

遵循如下的实践来实现凭据专用的盐的生成：

为每一个凭据生成唯一的盐（而不是每个用户或每个系统生成一个盐）；

使用有强密码学可靠性的随机[*3]数据；

根据存储许可要求，使用 32 位或 64 位盐（实际大小依赖于保护函数）；

安全机制并不取决于隐藏、拆分或隐藏盐；

加盐是为了达到两个目的：1)防止两个相同凭据的生成同样的加密数据；2)增加熵值让保护功能不再依赖凭据的复杂度。第二个还能让个人凭据免遭彩虹表攻击。

### 利用攻击者不可行验证

用来保护存储凭据的函数应该平衡攻击者和防卫验证。防御者需要一个即使在访问高峰时也能接受的验证用户凭据的响应时间。但是要知道，生成“凭据<=>保护数据”的映射表的时间与攻击者的硬件（GPU、FPGA）和技术（基于字典、暴力破解等）能力有关。

下面两种方法都不完美。

使用可以自适应的单向函数

自适应的单向函数进行单向的不可逆转换。每个函数都可以配置“工作因子”。如何实现不可逆性、支配工作因子（如时间、空间和并行度）将不在这里讨论。

选择:

PBKDF2 [*4]是 FIPS 认证的，已经获得很多企业支持。

scrypt [*5]能抵御任何/全部硬件加速攻击，但支持并不好。

bcrypt 不支持 PBKDF2 和 scrypt 加密算法。

protect()的伪代码如下：

```
return [salt] + pbkdf2([salt], [credential], c=10000); 
```

设计师选择单向自适应函数来实现 protect()函数，因为这些函数相对于哈希函数，能够通过修改配置改变它的执行耗时（线性或指数方式）。防卫者调整工作因子来与攻击者持续增长的硬件能力赛跑。这些自适应的单向函数的实现必须工作因子调整，从而在阻碍攻击者的同事提供可接受的用户体验。

此外，自适应的单向函数不能有效的阻止常见的基于字典的凭据破解，用户规模和加盐对于此丝毫没有帮助。

#### 工作因子

一般而言，资源是有限的，一个常见的调整工作因子（或耗时）的法则是：让 protect()函数在不影响用户的体验和增加额外的超预算硬件的前提下，尽可能的降低速度。因此，在注册和身份验证时，你可以改变 protect()的参数，让这个计算在你的硬件上耗时 1 秒钟。这样一来，它就不会因为过慢影响到你的用户，同时又能有效阻止攻击者的尝试轻轻。

虽然有推荐的最小迭代次数来确保数据的安全，但是由于技术在发展，这个值至少每年需要改变一次。一个知名的迭代次数的例子是苹果公司，他们加密 iTunes 密码（使用 PBKDF2）的迭代次数是 10000。但是你要知道，使用单一的工作因子并不适合所有的情况。实验很重要[*6]

#### 杠杆键函数

键函数，如 HMACs 算法，使用私钥和输入参数计算单向（不可逆）转换。对于 HMACs，它继承了哈希函数的属性，包括：速度、允许即时验证。密钥的长度影响不可行长度和/或空间要求——甚至是常见的凭据。设计者使用键函数来保护存储的凭据：

使用一个“站点级别”的 key；

像保护所有密钥一样用最佳实践来保护这个 key；

将这个 key 和凭据分开存储（不要存在数据库中）；

使用强密码学可靠性的伪随机数据生成密钥；

不要担心输出数据块的大小(如 SHA-256 vs. SHA-512).

protect()伪代码如下：

```
return [salt] + HMAC-SHA-256([key], [salt] + [credential]); 
```

盐方案的持续改进依赖于正确的密钥管理。

### 设计能处理密码被泄露的密码存储的方法

实际上，受保护凭据频繁被盗的现状逼迫着我们做“失败设计”。如果发现凭据被盗，一个好的凭据存储方案能轻松的对被盗凭据执行安全操作并使用备选的凭据验证流程：

保护用户的账号

使用第 2 因素或者密码问题来进行登陆，不再支持快捷登陆。

不允许用户修改账户信息，如编辑密码问题和修改账户多因子认证配置。

使用新的保护方案

使用新的更强大的密钥保护函数

包含版本信息的存储形式；

设置账号被盗或疑似被盗的标示，让用户重置凭据；

修改密钥和/或调整保护函数参数（迭代次数）；

增加方案版本号

当用户登陆时：

根据存储的版本验证凭据（新的或者旧的凭据）；如果是旧密码，则要求进行多因子认证或询问密码问题。

提示用户凭据已经变化，向用户道歉并引导用户进行其他身份确认。

当用户成功登陆后，将存储的凭据转换为新方案。

## 引用

[1] Morris, R. Thompson, K., Password Security: A Case History, 04/03/1978, p4:[`cm.bell-labs.com/cm/cs/who/dmr/passwd.ps`](http://cm.bell-labs.com/cm/cs/who/dmr/passwd.ps)

[2] Space-based (Lookup) attacks: Space-time Tradeoff: Hellman, M., Crypanalytic Time-Memory Trade-Off, Transactions of Information Theory, Vol. IT-26, No. 4, July, 1980[`www-ee.stanford.edu/~hellman/publications/36.pdf`](http://www-ee.stanford.edu/~hellman/publications/36.pdf) Rainbow Tables -[`ophcrack.sourceforge.net/tables.php`](http://ophcrack.sourceforge.net/tables.php)

[3] For example: SecureRandom.html.

[4] Kalski, B., PKCS #5: Password-Based Cryptography Specification Version 2.0, IETF RFC 2898, September, 2000, p9 [`www.ietf.org/rfc/rfc2898.txt`](http://www.ietf.org/rfc/rfc2898.txt)

[5] Percival, C., Stronger Key Derivation Via Sequential Memory-Hard Functions, BSDCan ‘09, May, 2009 [`www.tarsnap.com/scrypt/scrypt.pdf`](http://www.tarsnap.com/scrypt/scrypt.pdf)

[6] For instance, one might set work factors targeting the following run times: (1) Password-generated session key - fraction of a second; (2) User credential - ~0.5 seconds; (3) Password-generated site (or other long-lived) key - potentially a second or more.

[7]php hmac hash function:[`www.php.net/manual/en/function.hash-hmac.php`](http://www.php.net/manual/en/function.hash-hmac.php)

## 作者和主编

John Steven - john.steven[at]owasp.org (author)

Jim Manico - jim[at]owasp.org (editor)

## 原文属性

最后修改时间：This page was last modified on 25 March 2014, at 09:53.