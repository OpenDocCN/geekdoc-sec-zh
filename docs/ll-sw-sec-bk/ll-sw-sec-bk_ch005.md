# 5 编译器引入了安全漏洞

编译器引入的安全漏洞有着悠久的历史。Thompson [@Thompson1984] 在这个领域提供了最古老和最受欢迎的例子之一。在他的论文中，他谈到了一个能够检测到正在编译登录程序并可以插入后门的编译器，这样他就可以以任何用户的身份使用系统。然而，最常见的案例是编译器在生成的二进制文件中无意中添加了安全漏洞。

解释一下，为什么导致未定义行为的代码往往可以像程序员预期的那样工作，直到应用了一些优化，也许还可以谈谈为什么编译器在某些情况下会依赖于未定义行为的缺失，这看起来有些激进。[#202](https://github.com/llsoftsec/llsoftsecbook/issues/202)

当讨论编译器引入的安全漏洞时，未定义的行为起着重要作用。它在各种作品中得到了充分的讨论，如[@wang2012undefined] [@d2015correctness] [@dusilent]。通过阅读这些作者的作品，可以看到即使是经过仔细测试的项目，如 Linux、FreeBSD 或 PostgreSQL，也无法摆脱这类漏洞。为了更好地理解它们，本章包含了一些此类漏洞的例子、它们的影响以及如何修复它们。

第一个例子是一个 15 年前的漏洞，影响了 Mac OS X 中的随机数生成器（RNG）[@Wang2015]。在过去某个时刻，这个漏洞影响了所有*BSD 操作系统，因为它们与 Mac OS 有一个共同的祖先。

在系统的随机数生成器中，更具体地说在`srandomdev(3)`中，我们可以找到以下用于种子逻辑的代码片段：

```asm
struct timeval tv;
unsigned long junk;

gettimeofday(&tv, NULL);
srandom((getpid() << 16) ^ tv.tv_sec ^ tv.tv_usec ^ junk);
```

为了生成 RNG 的种子，代码使用了当前时间和栈中的一个未初始化的值，即`junk`。由于 C 标准对于未初始化加载没有明确的语义，这会触发未定义的行为。正因为如此，两个不同版本的 Mac OS X 生成的汇编代码之间存在巨大的差异。

在 Mac OS X 10.6 中，生成的代码看起来是这样的：

```asm
leaq    0xe0(%rbp),%rdi
xorl    %esi,%esi
callq   0x001422ca      ; symbol stub for: _gettimeofday
callq   0x00142270      ; symbol stub for: _getpid
movq    0xe0(%rbp),%rdx
movl    0xe8(%rbp),%edi
xorl    %edx,%edi
shll    $0x10,%eax
xorl    %eax,%edi
xorl    %ebx,%edi
callq   0x00142d68      ; symbol stub for: _srandom
```

对于 Mac OS X 10.7，代码看起来是这样的：

```asm
leaq    0xd8(%rbp),%rdi
xorl    %esi,%esi
callq   0x000a427e      ; symbol stub for: _gettimeofday
callq   0x000a3882      ; symbol stub for: _getpid
callq   0x000a4752      ; symbol stub for: _srandom
```

在生成的汇编代码的简短版本中，编译器为了优化而丢弃了`srandom`的整个参数。虽然优化后的代码遵守了标准，但它为攻击者留下了可以利用系统的空间，因为现在可以预测 RNG 的种子。

同时，这个问题已在 FreeBSD [@FbsdJunk] 和 OpenBSD [@ObsdJunk] 中得到解决。

当前用于检测这类漏洞的解决方案包括 LLVM 的 MemorySanitizer 和 Valgrind。

下一个示例涵盖了可以轻易引入安全漏洞的新类型未定义行为。这次我们讨论的是对空指针的解引用以及这种操作可能出现的错误。以下代码片段来自 Linux，通过在检查指针是否有效之前解引用`tun`指针来引入一个漏洞：

```asm
unsigned int
tun_chr_poll(struct file *file, poll_table * wait)
{
  struct tun_file *tfile = file->private_data;
  struct tun_struct *tun = __tun_get(tfile);
  struct sock *sk = tun->sk;
  if (!tun)
    return POLLERR;
  ...
}
```

通常情况下，这会导致内核崩溃或如果地址 0 映射到地址空间中，函数将返回 POLLERR。然而，编译器在执行到 if 语句时假设`tun`是一个有效的指针。这是因为它在 if 语句之前看到了一个较早的解引用。在这种情况下，检查被认为是多余的，并从最终的二进制文件中删除。这允许攻击者在地址 0 映射时继续从`tun_chr_poll`执行代码。

为了减轻这种情况，GCC 开发者添加了一个名为`-fno-delete-null-pointer-checks`的标志，Linux 将其集成到其编译器配置中。

Linux 并不是唯一遭受这种问题的项目。Chromium [@ChromiumIssue] 和 Mozilla [@MozillaIssue] 过去也遇到过这个问题。

也存在一些安全漏洞，它们不是由未定义行为引入的，以下代码片段就是一个例子。这是从 Linux 内核中取出的。因为编译器看到指针哈希在此之后从未被使用，它决定删除 memset 操作。我们称这种优化为死存储优化（DSO）。这具有严重的安全影响，因为程序员的意图是从内存中删除`hash`信息。

```asm
static void extract_buf(struct entropy_store *r, __u8 *out) {
  ...
  - memset(&hash, 0, sizeof(hash));
  + memzero_explicit(&hash, sizeof(hash));
}
```

Linux 提供的解决方案是添加一个名为`memzero_explicit`的新函数，其底层实现如下：

```asm
void memzero_explicit(void *s, size_t count)
{
  memset(s, 0, count);
  OPTIMIZER_HIDE_VAR(s);
}
```

它仍然使用`memset`来删除相关的安全敏感数据，但它还试图通过使用`OPTIMIZER_HIDE_VAR`宏来消除 DSO 的风险。然而，这并不足以完全消除死存储[@MemZeroBarrier]。在 LTO 使用的情况下，缓冲区`s`仍然存在漏洞。因此，Linux 维护者通过使用编译器屏障添加了进一步的加固机制：

```asm
void memzero_explicit(void *s, size_t count)
{
  memset(s, 0, count);
  - OPTIMIZER_HIDE_VAR(s);
  + barrier();
}
```

在引入的屏障[@MemZeroDataBarrier]方面，仍有改进的空间。如果缓冲区的内容存在于寄存器中，那么编译器会再次盲目地证明 DSO 可以被触发，`memset`也将再次被删除。为了减轻这种情况，提出了以下补丁：

```asm
+ #define barrier_data(ptr) \
+  __asm__ __volatile__("": :"r"(ptr) :"memory")

void memzero_explicit(void *s, size_t count)
{
  memset(s, 0, count);
  - barrier();
  + barrier_data(s);
}
```

在这个补丁中，我们创建了一个新的屏障，将确保将缓冲区的内容放入内存，这样 DSO 就无法再产生影响。

在其他项目如 OpenSSL[@OpenSSLMemClr]中也进行了类似的工作。OpenSSL 使用的方法相当不同，但它达到了相同的目标，即消除 DSO 的影响。通过将`memset_func`设置为指向`memset`实际实现的 volatile 指针，编译器被迫解引用该指针以获取实际的`memset`，从而消除了优化掉它的风险。

C23 委员会决定从另一个角度解决这个问题，即通过添加一个名为`memset_explicit`的库函数[@MemSetProposal]。这个函数要求编译器不要优化内存覆盖操作。然而，实现这样的功能并不简单，正如 GNU 在[@GNUMemSet]中提出的。即使信息已经被从内存中删除，它可能仍然存在于机器的某个地方。
