# 第六章. 将限制操作限制在缓冲区边界内（避免缓冲区溢出）

> 原文：[`dwheeler.com/secure-programs/Secure-Programs-HOWTO/buffer-overflow.html`](https://dwheeler.com/secure-programs/Secure-Programs-HOWTO/buffer-overflow.html)

|   |  **敌人将横扫土地；他将摧毁你的坚固堡垒，掠夺你的要塞。**  |
| --- | --- |
|   | *阿摩司书 3:11 (NIV)* |

**目录**

6.1\. C/C++中的危险

6.2\. C/C++中的库解决方案

6.2.1\. 标准 C 库解决方案

6.2.2\. 静态和动态分配的缓冲区

6.2.3\. strlcpy 和 strlcat

6.2.4\. asprintf 和 vasprintf

6.2.5\. libmib

6.2.6\. Safestr 库（Messier 和 Viega）

6.2.7\. C++ std::string 类

6.2.8\. Libsafe

6.2.9\. 其他库

6.3\. C/C++中的编译解决方案

6.4\. 其他语言

程序通常使用内存缓冲区来捕获输入和处理数据。在某些情况下（尤其是在 C 或 C++ 程序中），可能可以进行操作，但要么从，要么写入一个超出缓冲区预期边界的内存位置。在许多情况下，这可能导致极其严重的安全漏洞。这是一个如此普遍的问题，以至于它有一个 CWE 标识符，[CWE-119](http://cwe.mitre.org/data/definitions/119.html)。超出缓冲区边界是程序内部实现的问题，但它是一个如此普遍且严重的问题，所以我将此信息放在了自己的章节中。

缺乏对缓冲区边界操作的限制有许多变体。超出缓冲区边界的一个子类别是*缓冲区溢出*。缓冲区溢出这个术语有几个不同的定义。就我们的目的而言，如果程序试图在缓冲区中写入比其能容纳更多的数据，或者写入缓冲区边界之外的内存区域，则发生缓冲区溢出。一个特别常见的情况是在缓冲区末尾之外写入字符数据（通过复制或生成）。当从用户读取输入到缓冲区时，可能会发生缓冲区溢出，但它也可能在程序的其他处理中发生。缓冲区溢出也被称为*缓冲区越界*。这个子类别是一个如此普遍的问题，以至于它有自己的 CWE 标识符，[CWE-120](http://cwe.mitre.org/data/definitions/120.html)。

缓冲区溢出是一种极其常见且危险的安全漏洞，在许多情况下，缓冲区溢出会导致攻击者立即完全控制受漏洞程序。为了给您一个这个主题重要性的概念，在 CERT 中，1998 年的 13 条警告中有 9 条涉及缓冲区溢出，1999 年的警告中至少有一半涉及缓冲区溢出。1999 年在 Bugtraq 上进行的一项非正式调查发现，大约 2/3 的受访者认为缓冲区溢出是系统安全漏洞的主要原因（其余受访者将“配置错误”视为主要原因）[Cowan 1999]。这是一个老生常谈的问题，但仍然不断出现[McGraw 2000]。

利用缓冲区溢出漏洞的攻击通常根据缓冲区的位置命名，例如，“栈溢出”攻击攻击栈上的缓冲区，而“堆溢出”攻击攻击堆上的缓冲区（由 malloc 和 new 等操作员分配的内存）。更多详情可参考 Aleph1 [1996]、Mudge [1995]、LSD [2001]，或 Nathan P. Smith 的*栈溢出安全漏洞*网站[`destroy.net/machines/security/`](http://destroy.net/machines/security/)。Crispin Cowan 等人于 2000 年对问题及其一些应对方法进行了讨论，可在[`immunix.org/StackGuard/discex00.pdf`](http://immunix.org/StackGuard/discex00.pdf)找到。Pierre-Alain Fayolle 和 Vincent Glaume 在[`www.enseirb.fr/~glaume/indexen.html`](http://www.enseirb.fr/~glaume/indexen.html)对 Linux 中的问题及其一些应对方法进行了讨论。

允许攻击者读取超出缓冲区边界的数据也可能导致漏洞，这种弱点有自己的标识符（[CWE-125](http://cwe.mitre.org/data/definitions/125.html)）。例如，[Heartbleed](http://www.dwheeler.com/essays/heartbleed.html) 漏洞就是这种弱点。OpenSSL 中的 Heartbleed 漏洞允许攻击者提取关键数据，如私钥，然后使用它们（例如，以便他们可以冒充受信任的站点）。

**图 6-1. 物理缓冲区溢出：1895 年蒙帕纳斯铁路出轨事件**

![缓冲区溢出示意图](img/ce0530ce73c9a6f366736ca33f810910.png)

大多数高级编程语言本质上对超出缓冲区边界是免疫的，要么是因为它们会自动调整数组大小（这适用于大多数语言，如 Perl），要么是因为它们通常能够检测并防止缓冲区溢出（例如，Ada95）。然而，C 语言没有提供针对此类问题的保护，而 C++也容易被用于引发此类问题。汇编语言和 Forth 也没有提供保护，而且一些通常包含此类保护的编程语言（例如，C#、Ada 和 Pascal）可能已经禁用了这种保护（出于性能考虑）。即使你的大部分程序是用其他语言编写的，许多库例程是用 C 或 C++编写的，以及调用它们的“粘合”代码，因此其他语言通常不会提供你期望的那么完整的缓冲区溢出保护。

# 6.1. C/C++中的危险

> 原文：[`dwheeler.com/secure-programs/Secure-Programs-HOWTO/dangers-c.html`](https://dwheeler.com/secure-programs/Secure-Programs-HOWTO/dangers-c.html)

C 语言用户必须避免使用未检查边界的危险函数，除非他们已经确保边界永远不会被超过。在大多数情况下应避免的函数（或确保保护）包括 strcpy(3)、strcat(3)、sprintf(3)（与 vsprintf(3)的堂兄弟）、gets(3)。这些应该分别用 strncpy(3)、strncat(3)、snprintf(3)和 fgets(3)替换，但请参见下面的讨论。如果可以确保存在一个终止的 NIL 字符，则应避免使用 strlen(3)。scanf()家族（scanf(3)、fscanf(3)、sscanf(3)、vscanf(3)、vsscanf(3)和 vfscanf(3)通常很危险；不要在没有控制最大长度的情况下将其用于向字符串发送数据（格式%s 尤其是一个常见问题）。其他可能允许缓冲区溢出的危险函数（取决于其使用方式）包括 realpath(3)、getopt(3)、getpass(3)、streadd(3)、strecpy(3)和 strtrns(3)。在使用 getwd(3)时必须小心；发送给 getwd(3)的缓冲区必须至少 PATH_MAX 字节长。select(2)辅助宏 FD_SET()、FD_CLR()和 FD_ISSET()不会检查索引 fd 是否在边界内；请确保 fd >= 0 且 fd <= FD_SETSIZE（这个特定的宏在 pppd 中被利用过）。

不幸的是，snprintf()的变体还有其他问题。官方上，snprintf()不是 ISO 1990（ANSI 1989）标准中的标准 C 函数，尽管 sprintf()是，因此一些非常旧的系统不包括 snprintf()。更糟糕的是，一些系统的 snprintf()实际上并不保护缓冲区溢出；它们只是直接调用 sprintf。Linux 的旧版 libc4 依赖于一个“libbsd”，它做了这样可怕的事情，我被告知一些旧的 HP 系统也做了同样的事情。Linux 当前版本的 snprintf 是已知可以正确工作的，也就是说，它确实尊重请求的边界。snprintf()的返回值也有所不同；Single Unix Specification (SUS)版本 2 和 C99 标准在 snprintf()返回的内容上有所不同。最后，似乎至少某些版本的 snprintf()不能保证其字符串以 NIL 结尾；如果字符串太长，它根本不会包含 NIL。请注意，glib 库（GTK 的基础，与 GNU C 库 glibc 不同）有一个 g_snprintf()，它有一个一致的返回语义，总是以 NIL 结尾，最重要的是总是尊重缓冲区长度。

当然，问题不仅仅是调用字符串函数不当。以下是一些额外的缓冲区溢出问题类型的例子，由 Timo Sirainen 慷慨提供，涉及操纵数字以导致缓冲区溢出。

首先，存在符号位的问题。如果你读取影响缓冲区大小的数据，例如“要读取的字符数”，务必检查该数字是否小于零或一。否则，负数可能会被转换为无符号数，从而导致产生一个很大的正数，这可能会允许缓冲区溢出问题。请注意，有时攻击者可以提供一个很大的正数，并导致相同的事情发生；在某些情况下，大值会被解释为负数（如果没有检查小于一的价值，就会通过大数检查），然后在稍后解释为一个大正数。

```
  /* 1) signedness - DO NOT DO THIS. */
 char *buf;
 int i, len;

 read(fd, &len, sizeof(len));

 /* OOPS!  We forgot to check for < 0 */
 if (len > 8000) { error("too large length"); return; }

 buf = malloc(len);
 read(fd, buf, len); /* len casted to unsigned and overflows */
```

这是 Timo Sirainen 识别出的第二个例子，涉及整数大小截断。有时整数的不同大小可以被利用来导致缓冲区溢出。基本上，确保你不截断用于计算缓冲区大小的任何整数结果。以下是 Timo 为 64 位架构提供的示例：

```
  /* An example of an ERROR for some 64-bit architectures,
    if "unsigned int" is 32 bits and "size_t" is 64 bits: */

 void *mymalloc(unsigned int size) { return malloc(size); }

 char *buf;
 size_t len;

 read(fd, &len, sizeof(len));

 /* we forgot to check the maximum length */

 /* 64-bit size_t gets truncated to 32-bit unsigned int */
 buf = mymalloc(len);
 read(fd, buf, len);
```

这是 Timo Sirainen 提供的第三个例子，涉及整数溢出。当与 malloc()结合使用时，这尤其令人讨厌；攻击者可能能够创建一个计算出的缓冲区大小小于要放入其中的数据的情况。以下是 Timo 的示例：

```
  /* 3) integer overflow */
 char *buf;
 size_t len;

 read(fd, &len, sizeof(len));

 /* we forgot to check the maximum length */

 buf = malloc(len+1); /* +1 can overflow to malloc(0) */
 read(fd, buf, len);
 buf[len] = '\0';
```

# 6.2\. C/C++中的库解决方案

> 原文：[`dwheeler.com/secure-programs/Secure-Programs-HOWTO/library-c.html`](https://dwheeler.com/secure-programs/Secure-Programs-HOWTO/library-c.html)

C/C++ 中的一种部分解决方案是使用没有缓冲区溢出问题的库函数。下一小节描述了“标准 C 库”解决方案，它可以工作但有其缺点。下一小节描述了固定长度和动态重新分配缓冲区方法的一般安全问题。以下小节描述了各种替代库，例如 `strlcpy` 和 `libmib`。请注意，这些库并不能解决所有问题；您仍然需要在 C/C++ 中非常小心地编写代码，以避免所有缓冲区溢出情况。

## 6.2.1\. 标准 C 库解决方案

在 C 中防止缓冲区溢出的“标准”解决方案是使用标准 C 库调用，这些调用可以防御这些问题。这种方法在很大程度上依赖于标准库函数 `strncpy(3)` 和 `strncat(3)`。如果您选择这种方法，请注意：这些调用具有一些令人惊讶的语义，并且很难正确使用。函数 `strncpy(3)` 如果源字符串长度至少等于目标字符串，则不会在目标字符串末尾添加 NULL 终止符，因此请确保在调用 `strncpy(3)` 后将目标字符串的最后一个字符设置为 NULL。如果您打算多次重用相同的缓冲区，一种有效的方法是在使用前告诉 `strncpy()` 缓冲区实际上比实际短一个字符，并在使用前将最后一个字符设置为 NULL。`strncpy(3)` 和 `strncat(3)` 都要求您传递剩余可用的空间量，这是一个容易出错（并且出错可能导致缓冲区溢出攻击）的计算。它们都不提供简单的机制来确定是否发生了溢出。最后，与它所取代的 `strcpy(3)` 相比，`strncpy(3)` 具有显著的性能惩罚，因为它会在目标字符串的剩余部分填充 NULL。我收到过一些关于这个最后点的电子邮件表示惊讶，但这一点在 Kernighan 和 Ritchie 第二版 [Kernighan 1988，第 249 页] 中有明确说明，并且这种行为在 Linux、FreeBSD 和 Solaris 的 man 页面中有明确的文档。这意味着仅仅从 `strcpy` 更改为 `strncpy` 就可能导致性能严重下降，在大多数情况下没有合理的理由。

警告！！函数 `strncpy(s1, s2, n)` 也可以用作仅复制 s2 部分内容的方法，其中 n 小于 `strlen(s2)`。以这种方式使用时，`strncpy()` 本身基本上不提供防止缓冲区溢出的保护 - 您必须采取单独的措施来确保 n 小于 s1 的缓冲区。此外，以这种方式使用时，`strncpy()` 通常不会在复制 n 个字符后添加尾随的 NULL。这使得确定使用 `strncpy()` 的程序是否安全变得更加困难。

你也可以在防止缓冲区溢出的同时使用 sprintf()，但这样做时需要非常小心；它很容易被误用，因此很难推荐。sprintf 控制字符串可以包含各种转换说明符（例如，"%s"），并且控制说明符可以有可选的字段宽度（例如，"%10s"）和精度（例如，"%.10s"）说明。它们看起来相当相似（唯一的区别是一个点），但它们非常不同。字段宽度仅指定一个*最小*长度，对于防止缓冲区溢出来说完全无用。相比之下，精度说明指定了当用作字符串转换说明符时，该特定字符串在其输出中可能具有的最大长度——因此它可以用来防止缓冲区溢出。请注意，精度说明仅指定处理字符串时的总最大长度；对于其他转换操作，它有不同的含义。如果大小以"*"精度给出，则可以将最大大小作为参数传递（例如，sizeof()操作的结果）。这可以通过一个例子最简单地展示——以下是如何错误和正确地使用 sprintf()来防止缓冲区溢出的示例：

```
  char buf[BUFFER_SIZE];
 sprintf(buf, "%*s",  sizeof(buf)-1, "long-string");  /* WRONG */
 sprintf(buf, "%.*s", sizeof(buf)-1, "long-string");  /* RIGHT */
```

理论上，sprintf()应该非常有帮助，因为你可以用它来指定复杂的格式。遗憾的是，使用 sprintf()很容易出错。如果格式复杂，你需要确保目标足够大，可以容纳整个格式的最大可能大小，但精度字段仅控制一个参数的大小。在创建复杂输出时，“最大可能”的值往往很难确定。如果一个程序没有为最长可能的组合分配足够的空间，可能会打开缓冲区溢出漏洞。此外，sprintf()在操作完成后会在目标后追加一个 NUL 字符——这个额外的字符很容易忘记，并创造了发生偏移量错误的机会。因此，虽然这可以工作，但在某些情况下使用起来可能会很痛苦。

此外，关于上面代码的简要说明——请注意，sizeof()操作使用了数组的大小。如果代码被修改，使得“buf”是指向某些已分配内存的指针，那么所有“sizeof()”操作都必须更改（或者 sizeof 将只测量指针的大小，这对于大多数值来说空间不足）。

scanf()系列函数也有些令人困惑。一个明显的问题是，是否可以在%s 中使用最大宽度值来防止这些攻击。对于 scanf()有多个官方规范；一些明确指出宽度参数是绝对最大的字符数，而另一些则不是很明确。最大的问题是实现；我所知道的现代实现都支持最大宽度，但我不能肯定所有库都正确实现了最大宽度。在这种情况下，最安全的做法是自己处理。然而，如果你只是使用 scanf 并在格式字符串中包含宽度，很少有人会责怪你（但别忘了计算\0，否则你会得到错误长度）。如果你确实使用 scanf，最好在安装脚本中包含一个测试，以确保库正确地限制了长度。

## 6.2.2\. 静态和动态分配的缓冲区

像 strncpy 这样的函数对于处理静态分配的缓冲区很有用。这是一种编程方法，其中为“最长有用大小”分配一个缓冲区，然后它保持固定大小。另一种选择是按需动态重新分配缓冲区大小。结果发现，这两种方法都有安全影响。

使用固定长度缓冲区时存在一个一般的安全问题：缓冲区固定长度的事实可能被利用。这是 strncpy(3)、strncat(3)、snprintf(3)、strlcpy(3)、strlcat(3)和其他类似函数的问题。基本思想是攻击者设置一个非常长的字符串，使得当字符串被截断时，最终结果将是攻击者想要的（而不是开发者想要的）。也许这个字符串是由几个较小的部分连接而成的；攻击者可能会使第一部分与整个缓冲区一样长，这样所有后续尝试连接字符串都不会起作用。以下是一些具体的例子：

+   想象一下这样的代码：调用 gethostbyname(3) 函数，如果成功，立即使用 strncpy 或 snprintf 将 hostent->h_name 复制到一个固定大小的缓冲区中。使用 strncpy 或 snprintf 可以防止过长的完全限定域名（FQDN）导致的溢出，因此你可能认为这样就完成了。然而，这可能会导致截断 FQDN 的末尾。这可能会非常不理想，具体取决于接下来会发生什么。

+   想象一下使用 strncpy、strncat、snprintf 等函数将文件系统对象的完整路径复制到某个缓冲区的代码。进一步想象原始值是由不可信的用户提供的，并且复制是传递最终计算结果到函数的过程的一部分。听起来很安全，对吧？现在想象一下攻击者在一个路径的开头添加了大量斜杠 '/’。这可能导致未来的操作在文件“/”上执行。如果程序在相信结果将是安全的情况下追加值，程序可能存在可利用性。或者，攻击者可以设计一个接近缓冲区长度的长文件名，这样尝试追加到文件名就会静默失败（或者只部分发生，这可能存在可利用性）。

当使用静态分配的缓冲区时，你确实需要考虑源和目标参数的长度。对输入和中间计算进行合理性检查也可能解决这个问题。

另一种替代方案是动态重新分配所有字符串，而不是使用固定大小的缓冲区。GNU 编程指南推荐这种通用方法，因为它允许程序处理任意大小的输入（直到它们耗尽内存）。当然，动态分配字符串的主要问题是可能会耗尽内存。内存可能在其他程序部分耗尽，而不是你担心缓冲区溢出的部分；任何内存分配都可能失败。此外，由于动态重新分配可能会导致内存分配效率低下，即使技术上程序有足够的虚拟内存可用以继续运行，也可能耗尽内存。此外，在耗尽内存之前，程序可能会使用大量的虚拟内存；这很容易导致“颠簸”，即计算机花费所有时间在磁盘和内存之间传输信息（而不是进行有用的工作）。这可能会产生拒绝服务攻击的效果。一些合理的输入大小限制可以在这里有所帮助。一般来说，如果你使用动态分配的字符串，程序必须设计为在内存耗尽时安全失败。

## 6.2.3\. strlcpy 和 strlcat

一种替代方案，由 OpenBSD 采用，是 Miller 和 de Raadt 的 strlcpy(3) 和 strlcat(3) 函数 [Miller 1999]。这是一个最小化、静态大小的缓冲区方法，它提供了与不同（且更不易出错）接口的 C 字符串复制和连接。这些函数的源代码和文档可以在新的 BSD 风格开源许可证下获得，网址为 ftp://ftp.openbsd.org/pub/OpenBSD/src/lib/libc/string/strlcpy.3。

首先，这里是他们的一些原型：

```
 size_t strlcpy (char *dst, const char *src, size_t size);
size_t strlcat (char *dst, const char *src, size_t size);
```

strlcpy 和 strlcat 都接受目标缓冲区的完整大小作为参数（而不是要复制的最大字符数），并保证结果以 NIL 结尾（只要大小大于 0）。请记住，你应该在大小中包含一个用于 NIL 的字节。

strlcpy 函数从以 NUL 结尾的字符串 src 复制最多 size-1 个字符到 dst，并使结果以 NIL 结尾。strlcat 函数将 NUL 结尾的字符串 src 追加到 dst 的末尾。它将追加最多 size - strlen(dst) - 1 个字节，并使结果以 NIL 结尾。

strlcpy(3) 和 strlcat(3) 的小缺点之一是它们默认情况下并未安装在大多数类 Unix 系统中。在 OpenBSD 中，它们是 `$1` 的一部分。这并不是一个特别困难的问题；由于它们是小型函数，你甚至可以将它们包含在你自己的程序源代码中（至少作为一个选项），并创建一个单独的小型包来加载它们。你甚至可以使用 autoconf 自动处理这种情况。如果更多程序使用这些函数，那么它们很快就会成为 Linux 发行版和其他类 Unix 系统的标准部分。此外，这些函数最近已被添加到“glib”库中（我提交了补丁以实现这一点），因此使用 glib 的最新版本可以使它们可用。在 glib 中，这些函数被命名为 g_strlcpy 和 g_strlcat（而不是 strlcpy 或 strlcat），以符合 glib 库的命名约定。

此外，当提供的 size 为 0 或目标字符串 dst（在给定的字符数内）中没有 NIL 字符时，strlcat(3) 的语义略有不同。在 OpenBSD 中，如果大小为 0，则目标字符串的长度被认为是 0。此外，如果大小不为零，但目标字符串中没有 NIL 字符（在大小个字符内），则认为目标长度等于大小。这些规则使得处理没有嵌入 NIL 的字符串是一致的。不幸的是，至少在当前，Solaris 不遵守这些规则，因为它们在原始文档中没有指定。我与 Todd Miller 聊过，我们一致认为 OpenBSD 的语义是正确的（而 Solaris 是错误的）。推理很简单：在任何情况下，strlcat 或 strlcpy 都不应检查大小范围之外的目标中的字符；这种访问可能会引起核心转储（从访问超出范围的内存）甚至硬件交互（通过内存映射 I/O）。因此，给定：

```
   a = strlcat ("Y", "123", 0);
```

正确答案是 3（0+3=3），但 Solaris 会声称答案是 4，因为它错误地查看目标中的“大小”长度之外的字符。目前，我建议避免大小为 0 或目标没有 NIL 字符的情况。glib 的未来版本将隐藏这种差异，并始终使用 OpenBSD 的语义。

## 6.2.4\. asprintf 和 vasprintf

当使用 C（不是 C++）并且可以使用动态内存分配方法时，asprintf 和 vasprintf 函数是解决安全避免缓冲区溢出问题的特别有用的方法。asprintf() 和 vasprintf() 函数是 sprintf(3) 和 vsprintf(3) 的类似物，不同之处在于它们会自动分配一个新的 C 字符串，并返回该字符串的指针。它们具有以下形式：

```
   int asprintf(char **strp, const char *fmt, ...);
  int vasprintf(char **strp, const char *fmt, va_list ap); 
```

由于这些函数分配内存，你必须传递指针给 free(3) 以进行释放。这些函数返回“打印”的字节数（如果有错误则返回 -1）。

这些函数相对简单易用，特别是它们不会在中间截断结果（这是任何固定长度缓冲区解决方案的问题）。以下是一个示例：

```
   char *result;
  asprintf(&result, "x=%s and y=%s\n", x, y);
```

asprintf() 和 vasprintf() 函数在 C 语言中广泛用于完成任务，而不会出现缓冲区溢出问题。它们的问题在于实际上并不是标准函数（它们不在 C11 标准中）。尽管如此，它们被广泛实现；它们存在于 GNU C 库中，以及 *BSDs（包括苹果的）中。在其他系统上重新创建它们相对简单，例如，在 Windows 上用不到 20 行代码就可以重新实现。在错误处理方面存在一些差异；FreeBSD 在出错时将 strp 设置为 NULL，而其他系统则不会；用户不应依赖于任何一种行为。另一个问题是，它们的广泛使用很容易导致内存泄漏；就像任何分配内存的 C 函数一样，你必须手动释放分配的内存。

## 6.2.5\. libmib

一个用于 C 的工具集是 Forrest J. Cavalier III 的“libmib 分配字符串函数”，可在 [`www.mibsoftware.com/libmib/astring`](http://www.mibsoftware.com/libmib/astring) 找到。libmib 有两种变体；“libmib-open”似乎在其自带的类似 X11 许可证下是明确的开源，允许修改和重新分发，但重新分发时必须选择不同的名称。然而，开发者表示“可能并未完全测试”。要持续获得成熟的 libmib，你必须支付订阅费。文档不是开源的，但它可以免费获取。如果你正在考虑使用这个库，你也应该看看 Messier 和 Viega 的 Safestr 库（下文将讨论）。

## 6.2.6\. Safestr 库（Messier 和 Viega）

Messier 和 Viega 开发的 Safe C String (Safestr) 库可在 [`www.zork.org/safestr`](http://www.zork.org/safestr) 获取。Safestr 为 C 提供了一组字符串函数，这些函数会根据需要自动重新分配字符串。Safestr 字符串可以轻松转换为常规的 C "char *" 字符串，使用的是大多数 malloc() 实现所用的相同技巧：safestr 在传递的指针“之前”存储重要信息——因此，在现有程序中使用 safestr 更容易。Safestr 支持将字符串设置为只读，并支持“可信”的字符串值，这有助于检测问题。Safestr 在开源 BSD 风格许可下发布。请注意，safestr 需要 XXL 库，这是一个添加 C 中异常处理和资产管理支持的库。

## 6.2.7\. C++ std::string 类

C++ 开发者可以使用内置在语言中的 std::string 类。这是一种动态方法，因为存储空间会根据需要增长。然而，需要注意的是，如果将该类的数据转换为“char *”（例如，通过使用 data() 或 c_str()），缓冲区溢出的可能性就会再次出现，因此在使用这些方法时需要小心。请注意，c_str() 总是返回一个以 NULL 结尾的字符串，但 data() 可能是也可能不是（这取决于实现，大多数实现不包括 NULL 终止符）。避免使用 data()，如果必须使用它，不要依赖于其格式。

许多 C++ 开发者也使用其他字符串库，例如那些包含在其他大型库中的库，甚至是一些自制的字符串库。使用这些库时，请特别小心——许多替代的 C++ 字符串类包括将类自动转换为“char *”类型的例程。因此，它们可能会无声地引入缓冲区溢出漏洞。

## 6.2.8\. Libsafe

Arash Baratloo、Timothy Tsai 和 Navjot Singh（来自 Lucent Technologies）开发了 Libsafe，这是一个已知容易受到堆栈破坏攻击的几个库函数的包装器。这个包装器（他们称之为一种“中间件”）是一个简单的动态加载库，其中包含修改后的 C 库函数版本，如 strcpy(3)。这些修改后的版本实现了原始功能，但以一种确保任何缓冲区溢出都包含在当前堆栈帧内的方式。他们的初步性能分析表明，这个库的开销非常小。Libsafe 论文和源代码可在 [`www.research.avayalabs.com/project/libsafe`](http://www.research.avayalabs.com/project/libsafe) 获取。Libsafe 源代码在完全开源的 LGPL 许可证下提供。

Libsafe 的方法似乎有些有用。Linux 发行商当然应该考虑将其包含在内，其他人也应该考虑其方法。例如，我知道 Linux 的 Mandrake 发行版（版本 7.1）包括它。然而，作为一个软件开发者，Libsafe 是一个支持深度防御的有用机制，但它并不能真正防止缓冲区溢出。以下是一些你不应该在代码开发期间仅仅依赖 Libsafe 的原因：

+   Libsafe 仅保护一组已知具有明显缓冲区溢出问题的函数。在撰写本文时，这个列表比已知存在此问题的本书中函数列表要短得多。它也不会保护你编写的代码（例如，在 while 循环中）导致的缓冲区溢出。

+   即使 libsafe 被安装在一个发行版中，其安装方式也会影响其使用。文档建议设置 LD_PRELOAD 以启用 libsafe 的保护，但问题是用户可以取消设置此环境变量...从而导致他们执行的程序的保护被禁用！

+   Libsafe 仅保护堆栈到返回地址的缓冲区溢出；你仍然可以超过该过程框架中的堆或其他变量。

+   除非你能确保所有部署的平台都将使用 libsafe（或类似的东西），否则你必须像它不存在一样保护你的程序。

+   LibSafe 似乎假设保存的帧指针位于每个堆栈帧的开头。这并不总是正确的。编译器（如 gcc）可以优化某些内容，特别是选项“-fomit-frame-pointer”会移除 libsafe 似乎需要的信息。因此，libsafe 可能无法为某些程序工作。

libsafe 的开发者自己承认，软件开发者不应仅仅依赖 libsafe。用他们的话说：

> 人们普遍认为，解决缓冲区溢出攻击的最佳方案是修复有缺陷的程序。然而，修复有缺陷的程序需要知道某个特定程序是有缺陷的。使用 libsafe 和其他替代安全措施的真实好处是保护尚未被发现有漏洞的程序免受未来攻击。

## 6.2.9. 其他库

glib（不是 glibc）库是一个广泛可用的开源库，为 C 程序员提供了一系列有用的函数。例如，GTK+和 GNOME 都使用 glib。正如我之前提到的，在 glib 版本 1.3.2 中，通过我提交的补丁添加了 g_strlcpy()和 g_strlcat()。这将使得在 glib 的这些后续版本广泛可用后，便携式使用这些函数变得更加容易。目前，我没有分析表明 glib 库函数可以防止缓冲区溢出。然而，许多 glib 函数会自动分配内存，并且这些函数在失败时没有合理的拦截方式（例如，尝试其他操作）。因此，在许多情况下，大多数 glib 函数不能在大多数安全程序中使用。GNOME 指南建议使用 g_strdup_printf()等函数，只要程序在发生内存不足条件时立即崩溃是可以接受的。然而，如果你不能接受这一点，那么使用此类例程是不合适的。

# 6.3\. C/C++中的编译解决方案

> 原文：[`dwheeler.com/secure-programs/Secure-Programs-HOWTO/compilation-c.html`](https://dwheeler.com/secure-programs/Secure-Programs-HOWTO/compilation-c.html)

完全不同的方法是使用执行边界检查的编译方法（参见[Sitaker 1999]以获取列表）。在我看来，这类工具在多层防御中非常有用，但将此技术作为唯一的防御手段并不明智。许多此类工具仅提供部分防御。更全面的防御往往速度较慢（并且通常人们选择使用 C/C++，因为性能对于他们的应用程序来说很重要）。此外，对于开源程序，你无法确定将使用哪些工具来编译程序；使用给定系统的默认“正常”编译器可能会突然打开安全漏洞。

历史上，“StackGuard”是一个非常重要的工具，它是标准 GNU C 编译器 gcc 的修改版。StackGuard 通过在返回地址前插入一个“保护值”（称为“金丝雀”）来工作；如果缓冲区溢出覆盖了返回地址，金丝雀的值（希望）会改变，系统会在使用它之前检测到这一点。这非常有价值，但请注意，这并不能防止缓冲区溢出覆盖其他值（它们可能仍然能够用来攻击系统）。有工作正在扩展 StackGuard，使其能够向其他数据项添加金丝雀，称为“PointGuard”。PointGuard 将自动保护某些值（例如，函数指针和 longjump 缓冲区）。然而，使用 PointGuard 保护其他变量类型需要特定的程序员干预（程序员必须标识哪些数据值必须用金丝雀进行保护）。这可能很有价值，但很容易不小心遗漏保护那些你认为不需要保护的数据值——但实际上它们需要保护。关于 StackGuard、PointGuard 和其他替代方案的信息，请参阅 Cowan [1999]。StackGuard 启发了许多其他运行时机制的开发，以检测和对抗攻击。

[IBM 已经开发了一个基于 StackGuard 思想的堆栈保护系统，称为 ProPolice](http://www.trl.ibm.com/projects/security/ssp)。IBM 在其当前网站上不包括 ProPolice 的名称——它只是被称为“用于保护应用程序免受堆栈破坏攻击的 GCC 扩展”。然而，不使用名称很难谈论某事，所以我将继续使用 ProPolice 这个名字。与 StackGuard 一样，ProPolice 是 GCC（Gnu 编译器集合）的一个扩展，用于保护应用程序免受堆栈破坏攻击。用 C 编写的应用程序在编译时自动插入保护代码以获得保护。然而，ProPolice 与 StackGuard 略有不同，它增加了三个功能：（1）重新排序局部变量，将缓冲区放在指针之后（以避免进一步破坏任意内存位置的指针），（2）将函数参数中的指针复制到局部变量缓冲区之前的一个区域（以防止进一步破坏任意内存位置的指针），（3）省略某些函数的仪器代码（它基本上假设只有字符数组是危险的；虽然这并不完全正确，但大多数情况下是正确的，因此 ProPolice 在保持大部分保护能力的同时，性能更好]。

2005 年，红帽工程师基于从 ProPolice 学到的经验，在 GCC 中重新实现了缓冲区溢出防护措施。他们实现了 GCC 标志-fstack-protector（仅保护一些易受攻击的函数），以及-fstack-protector-all（保护所有函数）。2012 年，谷歌工程师添加了-fstack-protector-strong 标志，试图达到更好的平衡（它比-fstack-protector 保护更多函数，但不像-fstack-protector-all 那样保护所有函数）。许多 Linux 发行版将这些标志之一作为默认设置，或者至少为某些软件包使用，以加固应用程序程序。

在 Windows 上，微软的编译器包括/GS 选项，以包含类似于 StackGuard 的保护措施来防止缓冲区溢出。然而，值得注意的是，至少在[Microsoft Windows 2003 Server 上，这些保护机制可以被克服。](http://www.nextgenss.com/papers/defeating-w2k3-stack-protection.pdf)

一种特别强大的加固方法是"地址检查器"（ASan，也称为 ASAN 和 AddressSanitizer）。ASan 作为 LLVM 和 gcc 编译器的"-fsanitize=address"标志可用。ASan 可以对抗缓冲区溢出（全局/栈/堆）、使用后释放和双重释放的攻击。它还可以检测使用后返回和内存泄漏。它还可以对抗一些其他 C/C++内存问题，但由于其设计，它无法检测读在写之前。它的平均 CPU 开销为 73%（通常是 2 倍），内存开销为 2 倍至 4 倍；与以前的方法相比，这已经很低了，但仍然很重要。尽管如此，对于部署来说，这有时是可以接受的开销，对于测试，包括模糊测试，通常也是完全可以接受的。例如，Chromium 和 Firefox 的开发流程就使用了 ASan。关于 ASan 如何工作的详细信息可在[`code.google.com/p/address-sanitizer/`](http://code.google.com/p/address-sanitizer/)找到，特别是在 Konstantin Serebryany、Derek Bruening、Alexander Potapenko 和 Dmitry Vyukov（谷歌）撰写的论文"AddressSanitizer: A Fast Address Sanity Checker"中。从根本上说，ASan 使用"影子字节"来记录内存地址性。ASan 跟踪内存的地址性，其中地址性意味着是否允许读取或写入。所有内存分配（全局、栈和堆）都对齐到（至少）8 字节，每 8 字节内存的地址性由一个"影子字节"表示。在影子字节中，0 表示所有 8 字节可寻址，1..7 表示只有下一个 N 个字节可寻址，而负数（高位）表示没有字节可寻址。所有分配都被不可访问的"红色区域"（默认大小为 128 字节）包围。栈和堆中的每个分配/释放都会操作影子字节，每个读取/写入首先检查影子字节以查看是否允许访问。这种对策非常强大，尽管如果计算出的地址在另一个有效区域中，它可能会被欺骗。话虽如此，ASan 是针对用 C 或 C++编写的应用程序的非常强大的防御措施，在这些开销可以接受的情况下。

"不可执行段"方法是由 Ingo Molnar 开发的，称为[Exec Shield](http://lwn.net/Articles/31032/)。Molnar 的执行盾限制可执行代码存在的区域，并将可执行代码移至该区域下方。如果代码被移至必须出现零字节的区域，那么它就难以被利用，因为许多基于 ASCII 的攻击无法插入零字节。这并不是万无一失的，但它确实可以防止某些攻击。然而，许多程序调用的库在总体上非常大，它们的地址可能包含非零值，这使得它们更加脆弱。

一种不同的方法是限制控制权的转移；这并不能阻止所有缓冲区溢出攻击（例如，那些攻击数据的攻击），但它可以使其他攻击更难[Kiriansky 2002]

简而言之，最好是首先开发一个能够防御缓冲区溢出的正确程序。然后，在你完成这项工作之后，无论如何都要使用像 StackGuard 这样的技术和工具作为额外的安全网。如果你已经在代码本身中努力消除了缓冲区溢出，那么 StackGuard（以及类似工具）可能会更加有效，因为将会有更少的“盔甲上的缝隙”需要 StackGuard 来保护。

# 6.4. 其他语言

> [`dwheeler.com/secure-programs/Secure-Programs-HOWTO/other-languages.html`](https://dwheeler.com/secure-programs/Secure-Programs-HOWTO/other-languages.html)

缓冲区溢出的问题是一个很好的理由去使用其他编程语言，例如 Perl、Python、Java 和 Ada95。毕竟，今天使用的几乎所有其他编程语言（除了汇编语言）都能防止缓冲区溢出。当然，使用那些其他语言并不能消除所有问题；特别是请参阅第 8.3 节中关于 NIL 字符的讨论。还有确保那些其他语言的底层设施（例如，运行时库）可用且安全的问题。尽管如此，当开发安全程序以防止缓冲区溢出时，你当然应该考虑使用其他编程语言。
