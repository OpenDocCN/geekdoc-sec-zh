# 6 隐蔽代码

**隐蔽代码** 是看起来像在执行一件事情但实际上做了另一件事情的代码。它通常指的是以这种方式恶意编写的代码，以在代码审查中隐藏恶意行为。然而，当隐蔽行为是意外添加的，这也适用——事实上，理想的攻击也会给攻击者提供声称这是一个诚实错误的可能性。

[@Wheeler2020] 提供了一个比这里给出的更系统的对隐蔽代码的视角。本章通过一些示例概述了攻击方式以及编译器可以（和不能）如何帮助，尽管这可能不归类为“低级”软件安全。

## 6.1 赋值和相等混淆

在 C 风格语言中，最经典的隐蔽代码示例可能是利用赋值（`=`）和相等运算符（`==`）看起来非常相似的事实。

```asm
void somefunction(UserPermissionLevel permission_level) {

   if (permission_level = LEVEL_ADMIN) {
       do_admin_action();
       return;
   }
   report_access_violation();
}
```

在这里，`permission_level` 并不是与 `LEVEL_ADMIN` 进行比较，而是被赋予该值。这意味着如果 `LEVEL_ADMIN` 是一个非零值，if 条件将评估为真。

Clang 和 GCC 都会在带有警告选项 `-Wparentheses` 的情况下检测到这个示例。在赋值周围添加一对括号可以消除这个警告。在这些特定的例子中，额外的括号可能会引起审阅者的注意，但如果条件更复杂，包含多个由 `||` 和 `&&` 组合的子表达式，括号看起来就正常了。

这可能被归咎于语言设计问题。传统上，Python 不支持将赋值用作表达式，从而有效地绕过了这个特定问题。当语言中添加了赋值表达式 [@pep572]（在 Python 3.8 中）时，更具有视觉区别的“walrus 运算符” `:=` 被用来避免与比较运算符混淆的风险。即使在现有的语言中，编译器也可以提供选项来禁止那些希望这样做的人使用有风险的构造，他们可以选择加入，从而创建一个实际上是原始语言子集的新语言。

## 6.2 goto fail

“goto-fail” 错误，官方称为 [CVE-2014-1266](https://nvd.nist.gov/vuln/detail/CVE-2014-1266)，导致苹果设备在 TLS 连接中无法正确验证证书。它实际上禁用了函数应该进行的验证，并且在代码审查中很容易被忽略。这很可能是没有故意添加的错误，但仍然具有隐蔽代码的特征。

这个错误是由重复的“goto fail” 行引起的，该行被缩进，看起来像它被之前的 if 子句保护，但实际上它总是被执行。由于 `err` 变量在 if 语句内被设置为 0（即没有错误），函数总是会返回 0。

```asm
OSStatus
SSLVerifySignedServerKeyExchange(...)
{
    OSStatus err;
    ...

    if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
        goto fail;
    if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
        goto fail;
        goto fail;
    if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
        goto fail;
    ...

fail:
    return err;
}
```

在出现这个错误的时候，它们并没有，但现在 GCC 和 Clang 都有一个 `-Wmisleading-indentation` 选项，可以检测到这类问题。

此外，自动代码格式化工具，如 clang-format，将有助于发现这个问题，因为它将修复误导性的缩进。

## 6.3 特洛伊木马源

由[@Boucher2023]描述的“特洛伊木马源”攻击是另一种通过做一些使编辑器（或其他向用户显示代码的东西）以不同于编译器解析的方式渲染代码的事情来实现隐蔽代码的方法。

这是通过使用 Unicode 功能来实现的。支持双向文本是一个这样的功能，其中它有特殊字符来标记文本区域，使其从右到左或从左到右，以便在同一文档中混合，例如阿拉伯语和英语。如果这些功能被巧妙地使用，它们可以使文本以这种方式渲染，使得一个单词看起来像它是在**注释**内部，但对于编译器来说，它按照文件中出现的顺序解析它，看到的是它在注释**之后**。

GCC 提供了[`-Wbidi-chars`](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wbidi-chars_003d)选项来检测书写方向标记的使用。Clang 编译器[不提供](https://bugs.chromium.org/p/llvm/issues/detail?id=11#c9)此类警告，但在[clang-tidy](https://clang.llvm.org/extra/clang-tidy/checks/misc/misleading-bidirectional.html)中有一个检查。

“特洛伊木马源”的另一种变体是使用同形异义字符，这些字符看起来相似，甚至完全相同（取决于字体）。例如，Unicode 包含字符“除号”（U+2215），它与普通的 ASCII 斜杠“正斜杠”（U+002F）不同。

```asm
/* Important comment */
check_permissions();
/* Another comment */
```

如果第一行最后的斜杠不是一个斜杠，而是一个看起来相似但编译器不将其视为注释结束的字符，那么整个代码片段就被注释掉了。
