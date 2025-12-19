# 2 基于内存漏洞的攻击

## 2.1 关于内存漏洞的一些背景信息

内存访问错误描述了虽然程序允许但并非程序员所意图的内存访问。这些类型的错误通常通过明确列出它们的类型来定义，包括：

+   缓冲区溢出

+   空指针解引用

+   释放后使用

+   未初始化内存的使用

+   非法释放

内存漏洞是由于这些类型的错误而出现的重要漏洞类别，它们最常见于使用 C/C++等语言时的编程错误。这些语言默认不提供保护内存访问错误的机制。攻击者可以利用这些漏洞泄露敏感数据或覆盖关键内存位置，从而控制受漏洞影响的程序。

内存漏洞有着悠久的历史。1988 年的[莫里斯蠕虫](https://en.wikipedia.org/wiki/Morris_worm)是第一个广泛公开的利用缓冲区溢出的攻击。后来，在 90 年代中期，出现了一些描述缓冲区溢出的著名文章 [@AlephOne1996]。通过堆栈保护和不可执行堆栈缓解了堆栈缓冲区溢出。应对策略是更巧妙地绕过这些缓解措施：代码重用攻击，从像返回到 libc 这样的攻击开始 [@Solar1997]。代码重用攻击后来演变为返回导向编程（ROP） [@Shacham2007]以及更复杂的技术。

为了防御代码重用攻击，引入了地址空间布局随机化（ASLR）和控制流完整性（CFI）措施。这种攻击性和防御性安全研究之间的互动对于提高安全性至关重要，并且至今仍在继续。每次部署新的缓解措施都会导致尝试绕过它，或者采用替代的、更复杂的利用技术，甚至自动化这些技术的工具。

内存安全 [@Hicks2014] 语言在设计时考虑了防止此类漏洞，并使用诸如边界检查和自动内存管理等技术。如果这些语言承诺消除内存漏洞，为什么我们还在讨论这个话题？

一方面，C 和 C++仍然是非常流行的语言，尤其是在实现低级软件方面。另一方面，用内存安全语言编写的程序本身也可能因为它们实现中的错误而容易受到内存错误的影响，例如，编译器中的错误。我们能否通过也将内存安全语言用于编译器和运行时实现来解决这个问题？即使这听起来很简单，不幸的是，这些语言无法保护某些类型的编程错误。例如，内存安全语言编译器或运行时实现中的逻辑错误可能导致内存访问错误未被检测到。我们将在后续章节中看到这类逻辑错误在编译器优化中的例子。后续章节。

考虑到内存漏洞和缓解措施的丰富历史以及该领域的积极发展，编译器开发者在其职业生涯中可能会遇到这些问题。本章旨在作为该领域的入门指南。我们首先讨论利用原语，这在分析威胁模型时可能很有用。在其他章节中讨论威胁模型，并在此处引用该部分[编号 #161](https://github.com/llsoftsec/llsoftsecbook/issues/161)。然后，我们继续更详细地讨论各种类型的漏洞及其缓解措施，按它们出现的粗略时间顺序和复杂性顺序呈现。

## 2.2 利用原语

软件安全领域的初学者可能会发现自己迷失在许多描述特定内存漏洞及其利用方法的博客文章和其他出版物中。在这些出版物中出现的两个非常常见但对初学者来说可能不熟悉的术语是**读取原语**和**写入原语**。为了理解内存漏洞并能够设计有效的缓解措施，了解这些术语的含义、攻击者如何获取这些原语以及它们如何被使用是很重要的。

一种**利用原语**是一种允许攻击者在受害程序的内存空间中执行特定操作的机制。这是通过向受害程序提供特别定制的输入来实现的。

一种**写入原语**赋予攻击者对受害程序内存空间的一定程度的写入访问权限。写入的值和写入的地址可能被攻击者以不同程度地控制。例如，原语可能允许：

+   将固定值写入攻击者控制的地址，或者

+   将由固定基址和攻击者控制的、限制在特定范围内的偏移量（例如，32 位偏移量）写入地址。请更详细地描述为什么范围限制很重要[编号 #162](https://github.com/llsoftsec/llsoftsecbook/issues/162)，或者

+   将固定偏移量写入攻击者控制的基址。

根据更详细的属性，原语可以进一步分类。参见[@Miller2012]的第 11 张幻灯片以获取示例。

写原语中最强大的版本是**任意写**原语，其中地址和值都完全受攻击者控制。

相应地，**读原语**为攻击者提供了对受害程序内存空间的读取访问。访问的内存位置地址将在一定程度上受攻击者控制，就像写原语一样。一个特别有用的原语是**任意读**原语，其中地址完全受攻击者控制。

写原语的效果可能更容易理解，因为它有明显的副作用：将值写入受害程序的内存。但攻击者如何观察读原语的结果？

这取决于攻击是交互式还是非交互式[@Hu2016]。

+   在**交互式攻击**中，攻击者向受害程序提供恶意输入。恶意输入导致受害程序执行攻击者指示的读取操作，并输出该读取的结果。这种输出可以是任何类型的输出，例如受害程序传输的网络包。攻击者可以通过查看这种输出（例如解析这个网络包）来观察读原语的结果。然后这个过程重复：攻击者向受害程序发送更多的恶意输入，观察输出并准备下一个输入。您可以在[@Beer2020]中看到这种类型攻击的示例，它描述了一种零点击无线电接近攻击。

+   在**非交互式（一次性）攻击**中，攻击者一次性向受害程序提供所有恶意输入。恶意输入触发多个原语依次执行，并且原语能够通过受害程序的状态观察先前操作的效果。输入可以是，例如，JavaScript 程序[@Groß2020]，或者伪装成 GIF 的 PDF 文件[@Beer2021]。

本节中的参考文献描述了复杂的现代攻击。请考虑链接到更简单的攻击示例，以及一些教程级别的材料。[#163](https://github.com/llsoftsec/llsoftsecbook/issues/163)

攻击者最初如何获得这些类型的原语？细节各不相同，在某些情况下，需要结合许多技术，其中一些超出了本书的范围。但我们将在本章中描述其中的一些。例如，当输入大小超过程序预期时，堆栈缓冲区溢出会导致（受限的）写原语。

作为攻击的一部分，攻击者将希望多次执行每个原语，因为单个读取或写入操作很少足以实现他们的最终目标（关于这一点稍后还会讨论）。如何组合原语以执行多个读取/写入操作？

在交互式攻击的情况下，准备和发送输入到受害程序以及解析受害程序的输出通常是在一个外部程序中完成的，该程序驱动漏洞利用。攻击者可以自由选择他们喜欢的编程语言，只要他们能够在这个语言中与受害程序交互。例如，假设有一个用 C 语言编写的漏洞利用程序，通过 TCP 与受害程序通信。在这种情况下，原语被抽象为 C 函数，这些函数准备并发送数据包到受害程序，并解析受害程序的响应。使用原语的方法就是调用这些函数。这些调用可以很容易地与任意计算结合，所有这些计算都用 C 语言编写，以形成漏洞利用。

为了使这个重复输入/输出交互周期正常工作，受害程序的当前状态必须在提供输入和观察输出的不同迭代之间保持不变。换句话说，受害进程不应该被重新启动。

值得注意的是，虽然读写原语由精心构造的输入组成，攻击者可以将这些输入视为对受害程序的*指令*。受害程序实际上无意中实现了一个解释器，攻击者可以向这个解释器发送指令。这一点在[@Dullien2020]中进一步探讨。

在非交互式攻击的情况下，所有计算都在受害程序内部进行。在这种情况下，输入数据和代码的二重性更为明显，因为对受害程序的恶意输入可以被视为漏洞代码。也有情况是，输入显然被受害应用程序解释为代码，例如将 JavaScript 程序作为输入提供给 JavaScript 引擎的情况。在这种情况下，读写原语将被编写为 JavaScript 函数，当调用时，会产生访问 JavaScript 程序不应访问的任意内存的意外副作用。这些原语可以与任意计算一起链接，这些计算也用 JavaScript 表示。

然而，有些情况下，数据和代码之间的对应关系并不那么明显。例如，在[@Beer2021]中，恶意输入由一个 PDF 文件组成，伪装成 GIF。由于 PDF 解码器中的整数溢出漏洞，恶意输入导致无界缓冲区访问，因此到任意读写原语。在 JavaScript 引擎利用的情况下，攻击者通常能够使用 JavaScript 操作并执行任意计算，使利用过程更加直接。在这种情况下，没有官方支持脚本能力。然而，攻击者利用压缩格式的复杂性来实现一个小型计算机架构，通过数千条简单命令传递给解码器。通过这种方式，他们有效地*引入*了脚本能力，并能够将他们的利用表达为针对该架构的程序。

到目前为止，我们已经描述了读写原语。我们也讨论了攻击者可能如何执行任意计算：

+   在交互式攻击的情况下，在外部程序中，或者

+   通过在非交互式攻击中使用脚本能力（无论是原始支持的还是由攻击者引入的）。

假设攻击者已经获得了这些能力，他们如何利用它们来实现他们的目标？

攻击者的最终目标可能各不相同：它可能是获取系统访问权限、泄露敏感信息或使服务崩溃等。通常，实现这些更广泛目标的第一步是在受害进程中执行任意代码。我们已经提到，攻击者此时通常具有任意计算能力，但任意代码执行还涉及调用任意库函数和执行系统调用等事项。

一些攻击者可能如何使用所获得原语的例子：

+   泄露信息，例如指向特定数据结构或代码的指针，或栈指针。

+   覆盖栈内容，例如执行返回导向编程（ROP）攻击。

+   覆盖非控制数据，例如授权状态。有时这一步就足以实现攻击者的目标，绕过执行任意代码的需求。

一旦实现了任意代码执行，攻击者可能需要利用额外的漏洞来逃离进程沙盒、提升权限等。这种漏洞链是常见的，但为了本章的目的，我们将专注于：

+   首先防止内存漏洞，从而阻止攻击者获得强大的读写原语。

+   通过维护控制流完整性（CFI）等机制来减轻读写原语的影响。

## 2.3 栈缓冲区溢出

缓冲区溢出发生在从数据缓冲区[数据缓冲区](https://en.wikipedia.org/wiki/Data_buffer)读取或写入时超出其边界。这通常会导致访问相邻的数据结构，从而有可能泄露或破坏相邻数据的完整性。

当缓冲区在堆上分配时，我们称之为堆栈缓冲区溢出。在本节中，我们专注于堆栈缓冲区溢出，因为在没有缓解措施的情况下，它们是一些最简单的缓冲区溢出，易于利用。

函数的[堆栈帧](https://en.wikipedia.org/wiki/Call_stack)包括重要的控制信息，如保存的返回地址和保存的帧指针。无意中覆盖这些值通常会导致崩溃，但攻击者可以精心选择溢出的值来控制程序执行的控制权。

这里有一个程序易受堆栈缓冲区溢出攻击 1 的简单示例：

```asm
#include <stdio.h>
#include <string.h>

void copy_and_print(char* src) {
  char dst[16];

  for (int i = 0; i < strlen(src) + 1; ++i)
    dst[i] = src[i];
  printf("%s\n", dst);
}

int main(int argc, char* argv[]) {
  if (argc > 1) {
    copy_and_print(argv[1]);
  }
}
```

在上面的代码中，由于在将参数复制到`dst`之前没有检查参数的长度，所以我们存在缓冲区溢出的潜在风险。

当查看使用 GCC 11.22 为 AArch64 生成的代码时，堆栈布局如下所示：

![堆栈缓冲区溢出示例的堆栈帧布局](img/file0.svg)

堆栈缓冲区溢出示例的堆栈帧布局

堆栈帧布局的详细情况，包括变量的顺序和存储的确切控制信息，将取决于您使用的特定编译器版本和编译的架构。

如堆栈图所示，`copy_and_print`函数中的溢出写入可以覆盖`main`帧中的保存帧指针（FP）和链接寄存器（LR）。当`copy_and_print`返回时，执行继续在`main`中。然而，当`main`返回时，执行从保存的 LR 地址继续，该地址已被覆盖。因此，当攻击者可以选择覆盖保存 LR 的值时，就可以控制程序在从`main`返回后继续执行的位置。

在非可执行堆栈成为主流之前，利用这些漏洞的一种常见方式是利用溢出同时将 shellcode3 写入堆栈并覆盖返回地址，使其指向 shellcode。[@AlephOne1996]是这种技术的经典例子。

解决这个问题的明显方法是使用处理器的内存保护功能，将堆栈（以及其他数据段）标记为不可执行 4。然而，即使堆栈不可执行，也可以使用更高级的技术来利用覆盖返回地址的溢出。这些技术利用可执行文件或库代码中已经存在的代码，将在下一节中描述。

栈 canary 是针对栈缓冲区溢出的另一种缓解措施。一般思路是在缓冲区和控制信息（例如示例中的保存的 FP 和 LR）之间存储一个已知值，称为栈 canary，并在离开函数之前检查这个值。由于将要覆盖返回地址的溢出首先会覆盖 canary，因此通过栈缓冲区溢出破坏返回地址将被检测到。

这种技术有一些局限性：首先，它专门旨在保护栈缓冲区溢出，对更强大的原语（例如任意写入原语）没有任何保护措施。下一节中描述的控制流完整性技术旨在保护存储的代码指针免受任何修改。

其次，由于编译器需要生成额外的指令以确保 canary 的完整性，通常会采用启发式方法来确定哪些函数被认为是易受攻击的。然后只为被认为是易受攻击的函数生成额外的指令。由于启发式方法并不总是完美的，这给技术带来了另一个潜在的局限性。为了解决这个问题，编译器可以引入各种级别的启发式方法，从仅对一小部分函数应用缓解措施，到普遍应用。例如，GCC 和 Clang 都提供了`-fstack-protector`、`-fstack-protector-strong`和`-fstack-protector-all`选项，具体请参阅[GCC](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)和[Clang](https://clang.llvm.org/docs/ClangCommandLineReference.html#cmdoption-clang-fstack-protector)的文档。

另一个局限性是 canary 值的泄露可能性。canary 值通常在程序开始时随机化，但在程序执行期间保持不变。因此，如果攻击者能够在某个时刻获取 canary 值，他们可能能够重新使用泄露的 canary 值并破坏控制信息，同时避免被检测到。选择包含空字节（C 风格字符串终止符）的 canary 值可能有助于限制来自字符串操作函数的溢出造成的损害，即使值被泄露。

许多缓冲区溢出漏洞源于使用不安全的库函数，例如`gets`，或者由于不安全地使用库函数，如`strcpy`。关于编写安全的 C/C++代码有大量的文献，例如[@Seacord2013]和[@Dowd2006]。限制溢出影响的一种不同方法是库函数加固，其目的是检测缓冲区溢出并优雅地终止程序。这涉及到引入特征宏，如`_FORTIFY_SOURCE`[@Sharma2014]。

最后，重要的是要提到，并非所有缓冲区溢出都旨在覆盖保存的返回地址。有许多情况，缓冲区溢出可以覆盖缓冲区附近的其他数据，例如一个决定授权是否成功的相邻变量，或者一个函数指针，当修改后，可以按照攻击者的意愿修改程序的流程控制。

这些漏洞中的一些可以通过本节中描述的措施来缓解，但通常需要更通用的措施来确保内存安全或 控制流完整性。例如，除了强化特定库函数外，编译器还可以为可以静态确定数组边界的数组实现自动边界检查（`-fsanitize=bounds`），以及各种其他“sanitizers”。我们将在接下来的章节中描述这些措施。

## 2.4 使用后释放 (UaF)

当一个变量在释放后被使用（读取和/或写入）时，会发生 *use after free* (UaF)。尽管这个描述假设使用 malloc/new 和 free/delete（堆分配）进行手动内存管理，但如果我们将内存视为一种资源，那么可以将同样的想法更广泛地应用。例如，获取栈中变量的引用并在其作用域结束后使用它，或者以某种方式从垃圾收集器访问已释放的内存。需要注意的是，尽管看似相关，一些作者更喜欢不混淆定义。例如，通用弱点枚举（CWE）页面 [仅提供了示例](https://cwe.mitre.org/data/definitions/416.html)，使用原始的 malloc/frees，没有提及栈或垃圾收集器的任何情况。除非明确说明，本节其余部分假设使用 malloc/new 和 free/delete 进行原始内存管理。

虽然某些 UaF 情况可能只会导致软件行为异常或崩溃，但其他情况可能使攻击者能够毒化数据，从而改变程序流程。这种情况可能发生的可能性有很多。其中一些取决于内存分配器如何管理其数据。例如，如果攻击者能够欺骗分配器为两个不同的分配返回相同的地址，那么这可能导致可控制的数据。这更详细地展示在示例 @ex:use-after-free 中。关于堆利用技术的概述，请参阅 [@dhavalkapil2022]。

```asm
struct auth_t {
  char name[32];
  int logged_in;
};
 
int main(int argc, char** argv) {
  char line[50];
 
  while(1) {
    printf("[ auth = %p, service = %p ]\n", auth, service);
 
    if (fgets(line, sizeof(line), stdin) == NULL) break;
 
    // Usage: auth <name>
    if (strncmp(line, "auth ", 5) == 0) {
      auth = (struct auth_t*)malloc(sizeof(struct auth_t));

      // Memory is only set to 0 here, not on free[1]
      memset(auth, 0, sizeof(struct auth_t));

      if (strlen(line + 5) < 31) {
        strcpy(auth->name, line + 5);
      }
    }
 
    // Usage: reset
    else if (strncmp(line, "reset", 5) == 0) {
      // [1]
      free(auth);
    }

    // Usage: service <service-name>
    else if (strncmp(line, "service ", 8) == 0) {
      service = strdup(line + 8);
    }

    // Usage: login
    else if (strncmp(line, "login", 5) == 0) {
      // Possible use-after-free:
      if (auth && auth->logged_in) {
        printf("You are already logged in!\n");
      } else {
        printf("NOT AUTHORIZED!\n");
      }
    }
  }

  return 0;
}
```

下面的利用可能不会在所有系统上工作，因为它假设调用 malloc+free+malloc 将导致两次 malloc 调用返回相同的指针。下面的示例执行通过利用 UaF 来改变布尔值，使软件误以为用户已登录：

```asm
> auth admin                                   
[ auth = 0x78322e1000, service = 0x0 ]       
> reset                                        
[ auth = 0x78322e1000, service = 0x0 ]       
> service aaaaaaaaa0aaaaaaaaa0aaaaaaaaa0121    
[ auth = 0x78322e1000, service = 0x78322e1000 ]
> login                                        
You are already logged in!
```

在这个例子中，发出了四个命令：

1.  `auth admin` 将首次分配 `auth_t` 结构。此时，用户有一个名字，但尚未授权（`logged_in` 为 `false`）。

1.  `reset` 将释放内存（但 `auth` 的指针没有被设置为 `nullptr`）。`service (...)` 将分配一个新的字符串，可能是在 `auth_t` 结构之前分配的同一地址。

1.  `service aaaaaaaaa0aaaaaaaaa0aaaaaaaaa0121` 将导致之前指向 `auth_t` 的内存被设置。

1.  `login` 将使用悬垂的 `auth_t` 指针。如果此内存已被重新分配到攻击者控制的字符串，则字段 `name` 将显示为 `aaaaaaaaa0aaaaaaaaa0aaaaaaaaa012`，布尔值 `logged_in` 为 `true`。

检测 UaF 通常不是一项容易的任务，因为它不仅取决于用户输入，有时还取决于执行流程。在多线程环境中，这可能会变得更加复杂。因此，已经构建了许多不同的 UaF 检测工具，基于多种不同的方法：

一些检测器拦截对 delete / free 的调用，向变量注入已知值，然后运行软件寻找包含该值的崩溃。用于检测 UaF 的另一个有用工具是 Arm 的 MTE（内存标记扩展），在第 @sec:preventing-and-detecting-memory-errors 节中讨论。

模糊测试（生成随机输入）也可能很有用，可能与其他工具同时使用。

还可以包括不同的算法来降低在分配器中利用 use-after-free 的概率，作为缓解策略，垃圾收集器或基于引用计数的内存分配器。

防止 UaF 发生可能涉及多种方法，这取决于上下文。从简单的代码更改，例如初始化分配的变量，到更复杂的更改，例如更改内存分配器的工作方式以避免重用特定的内存位置。除此之外，降低 UaF 对攻击者的相关性也可以是一个有趣的视角（即，即使存在 UaF，降低其可利用性的可能性）。例如，MTE 同步模式可以在 UaF 发生时立即强制应用程序崩溃，而指针认证（PAC）可以用来签名指针，即使它们被毒化，也无法使用（更多细节请参阅第 @sec:pointer-authentication 节）。

深入了解 UaF 的检测和缓解。建议的起点：

+   使用不同算法的分配器来降低利用 use-after-free 的概率（这可能是它自己的一个完整章节？）

+   类型感知的分配和释放函数，这解释了在 https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2719r5.html#a-concrete-use-case 中引入此功能的原因。

+   使用垃圾回收或引用计数的语言

这些并不是检测、缓解和预防 UaF 的工具的详尽列表。但本节的目标是简要介绍这一主题。

## 2.5 代码重用攻击

在内存漏洞利用的早期，攻击者只需将他们选择的 shellcode 放置在可执行内存中，然后跳转到它。随着非可执行栈和堆成为主流，攻击者开始重新使用应用程序的二进制文件和链接库中已经存在的代码。为此目的，出现了各种不同的技术。

这些技术中最简单的是返回到库函数[@Solar1997]。与返回到攻击者注入的 shellcode 不同，返回地址被修改为返回到一个库函数，例如`system`或`exec`。当参数也通过栈传递并且可以因此通过用于修改地址的相同的栈缓冲区溢出进行控制时，这种技术更易于使用。

### 2.5.1 返回导向编程

返回到库函数攻击将攻击者限制在库函数的整体功能上。虽然这可能导致强大的攻击，但已经证明，通过组合一系列以间接控制转移指令结束的短指令序列，即所谓的**gadgets**，可以实现任意计算。间接控制转移指令使得攻击者能够通过控制提供每个控制转移指令目标的内存或寄存器，轻松地依次执行 gadgets。

在返回导向编程（ROP）[@Shacham2007]中，每个 gadget 执行一个简单的操作，例如设置一个寄存器，然后从栈中弹出一个返回地址并返回到它。攻击者构建一个假的调用栈（通常称为 ROP 链），以确保一系列 gadgets 依次执行，以执行更复杂的操作。

通过一个例子可能会更清楚地说明这一点：一个用于 AArch64 Linux 的 ROP 链，通过调用`execve`并使用`"/bin/sh"`作为参数来启动 shell。`execve`库函数的原型[链接](https://man7.org/linux/man-pages/man2/execve.2.html)，它封装了 exec 系统调用，如下所示：

```asm
 int execve(const char *pathname, char *const argv[],
             char *const envp[]);
```

对于 AArch64，`pathname`将通过`x0`寄存器传递，`argv`将通过`x1`传递，`envp`将通过`x2`传递。为了启动 shell，只需做以下操作：

+   使`x0`包含指向`"/bin/sh"`的指针。

+   使`x1`包含指向一个包含两个元素的指针数组的指针：

    +   第一个元素是指向`"/bin/sh"`的指针。

    +   第二个元素是零（`NULL`）。

+   使`x2`包含零（`NULL`）。

这可以通过将 gadgets 链接起来，设置寄存器`x0`、`x1`、`x2`，然后返回到 C 库中的`execve`来实现。

假设我们有以下 gadgets：

1.  一个从栈中加载`x0`和`x1`的 gadget：

```asm
 gadget_x0_x1:
    ldp x0, x1, [sp]
    ldp x20, x19, [sp, #64]
    ldp x29, x30, [sp, #32]
    ldr x21, [sp, #48]
    add sp, sp, #0x50
    ret
```

1.  一个将`x2`设置为零，但作为副作用也清除`x0`的 gadget：

```asm
 gadget_x2:
    mov x2, xzr
    mov x0, x2
    ldp x20, x19, [sp, #32]
    ldp x29, x30, [sp]
    ldr x21, [sp, #16]
    add sp, sp, #0x30
    ret
```

解释这些 gadgets 如何从 C/C++代码中产生。当前版本经过轻微的手动调整，以获得更可管理的偏移量。[编号#164](https://github.com/llsoftsec/llsoftsecbook/issues/164)

这两个 gadget 还破坏了几个无趣的寄存器，但由于`gadget_x2`还清除了`x0`，因此很明显我们应该使用一个 ROP 链：

1.  返回到`gadget_x2`，将`x2`设置为零。

1.  返回到`gadget_x0_x1`，将`x0`和`x1`设置为所需的值。

1.  返回到`execve`。

图@fig:rop-control-flow 展示了这个控制流。

![ROP 示例控制流](img/file1.svg)

ROP 示例控制流

![ROP 示例伪造调用栈](img/file2.svg)

ROP 示例伪造调用栈

我们可以通过构建图@fig:rop-call-stack 中所示的伪造调用栈来实现这一点，其中“原始帧”标记了`gadget_x2`的地址替换了未来将被加载和返回的保存的返回地址的帧。作为替代，攻击者可以将这个伪造的调用栈放在其他地方，例如堆上，并使用一个改变栈指针值的原始操作。这被称为栈指针劫持。

注意，即使不考虑各种返回地址的确切值，这个伪造的调用栈也包含零字节。基于 C 风格字符串操作的溢出漏洞不会允许攻击者一次性用这个伪造的调用栈替换栈内容，因为 C 风格字符串是[空终止的](https://en.wikipedia.org/wiki/Null-terminated_string)，并且复制伪造的栈内容会在遇到第一个零字节时停止。因此，ROP 链需要调整，以便它不包含零字节，例如，最初用不同的字节替换零字节，并在 ROP 链中添加一些将零写入这些栈位置的 gadget。

当查看栈图时，一个问题会浮现出来：“我们如何知道这些 gadget 的地址”？我们将在下一节中对此进行更多讨论。

类似这里使用的 ROP（Return-Oriented Programming）工具可能通过检查拆解的二进制文件来轻松识别，但攻击者通常使用“gadget 扫描器”工具来自动发现大量 gadget。这些工具对于从事代码重用攻击缓解的编译器工程师来说也很有用，因为它们可以指出应该受到保护但被遗漏的代码序列。

可执行内存中的任何内容都可能被用作 ROP 设备，即使编译器没有打算将其作为代码。这包括与代码混合的文本池，以及在具有可变长度指令编码的架构上返回到指令的中间。在攻击者可能影响生成哪些文本的即时编译器中，这尤其强大。例如，在 x86 上，编译器可能发出了指令`mov $0xc35f, %ax`，该指令编码为四个字节`66 b8 5f c3`。如果攻击者可以将执行流程偏离该 4 字节指令的两个字节，它将执行`5f c3`。这些字节对应于两个单字节指令`pop %rdi; ret`，这是一个有用的 ROP 设备。

### 2.5.2 跳转导向编程

跳转导向编程（JOP）[@Bletsch2011]是 ROP 的一种变体，其中设备也可以以间接分支指令而不是返回指令结束。攻击者通过一个调度设备链接着多个这样的设备，该设备依次从一个指针数组中加载指针，并返回到每个指针。使用的设备必须设置成在完成操作后分支或返回到调度设备。这在图@fig:jop 中有演示。

![JOP 示例](img/file3.svg)

JOP 示例

图中的设备是虚构的，选择它们是为了突出每个设备可以以不同类型的间接控制流转移指令结束。考虑用更现实的设备替换它们。[#165](https://github.com/llsoftsec/llsoftsecbook/issues/165)

在图@fig:jop 中，`x4`最初指向“调度表”，该表已被攻击者修改，包含他们想要执行的三个设备的地址。调度设备逐个加载调度表中的地址，并分支到它们。第一个设备从堆栈中加载`x0`和`x1`，攻击者已经将他们的选择输入放置在这里。然后它加载其返回地址，该地址也被攻击者修改，使其指向调度设备，并返回到它。调度设备分支到下一个设备，该设备将`x0`和`x1`相加，并将结果留在`x0`中，通过从堆栈中加载另一个值到`x2`来返回到调度设备。最后一个设备将加法的结果，仍然在`x0`中，存储到堆栈中，在分支到仍然指向调度设备的`x2`之前。

### 2.5.3 伪造面向对象编程

伪造面向对象编程（COOP）[@Schuster2015]是一种利用 C++虚函数调用的代码重用技术。COOP 攻击利用现有的虚函数和[vtables](https://en.wikipedia.org/wiki/Virtual_method_table)，并创建指向这些现有 vtables 的假对象。在攻击中用作 gadgets 的虚函数称为 vfgadgets。为了将 vfgadgets 链接在一起，攻击者使用一个“主循环 gadget”，类似于 JOP 的调度器 gadget，它本身是一个虚拟函数，它遍历一个指向 C++对象指针的容器，并在这些对象上调用虚拟函数。[@Schuster2015]更详细地描述了这种攻击。这里特别提到它作为一个攻击示例，这种攻击不依赖于直接替换返回地址和代码指针，就像 ROP 和 JOP 所做的那样。在考虑针对代码重用攻击的缓解措施时，考虑这些特定语言攻击是很重要的，这将是下一节的主题。

很想有一个 COOP 攻击的小例子，类似于前一部分中 JOP 例子的那样。[#261](https://github.com/llsoftsec/llsoftsecbook/issues/261)

### 2.5.4 Sigreturn-oriented programming

值得一提的最后一个代码重用攻击的例子是 sigreturn-oriented programming（SROP）[@Bosman2014]。它是一种特殊的 ROP，攻击者创建一个假的信号处理程序帧并调用`sigreturn`。`sigreturn`是许多 UNIX 类型系统上的系统调用，通常在从信号处理程序返回时调用，并基于内核之前在信号处理程序的堆栈上保存的状态恢复进程的状态。伪造信号处理程序帧并调用`sigreturn`的能力为攻击者提供了一个简单的方式来控制程序的状态。

## 2.6 针对代码重用攻击的缓解措施

在讨论针对代码重用攻击的缓解措施时，重要的是要记住，攻击者必须具备两种能力才能使此类攻击生效：

+   覆写返回地址、函数指针或其他代码指针的能力。

+   知道要覆盖的目标地址（例如，libc 函数入口点）。

当代码重用攻击最初被描述时，程序通常包含绝对代码指针，需要加载到固定地址。栈基址是可预测的，库在可预测的内存位置加载。这使得代码重用攻击变得简单，因为所有成功利用所需的地址都很容易发现。在本节中，我们将讨论使攻击者更难获得这些能力的缓解措施。

攻击者能够覆盖代码指针的能力通常归结为在它们存储在内存中而不是在机器寄存器中时覆盖它们。直接在机器寄存器中覆盖值通常是不可能的。攻击者利用内存漏洞来能够在内存中覆盖指针。考虑到这一点，人们可能会认为对于在内存安全语言中编写的程序，代码重用缓解措施是不必要的，因为它们不应该有任何内存漏洞。然而，大多数现实生活中的内存安全语言编写的程序仍然包含至少部分在非安全语言中编写的二进制代码。攻击者可以在程序的非安全部分获得写原语，并使用它来覆盖程序内存安全部分的代码指针。因此，针对代码重用攻击的缓解措施对于在内存安全语言中编写的程序仍然相关。

攻击者能够在内存安全程序中获得写原语的另一个原因是编译器或运行时中的错误。这对于基于 JIT 的语言尤其如此，有关更多详细信息，请参阅第 @sec:jit-compiler-vulnerabilities 节。

### 2.6.1 地址空间布局随机化（ASLR）

[地址空间布局随机化（ASLR）](https://en.wikipedia.org/wiki/Address_space_layout_randomization) 通过随机化包含可执行文件、加载的库、栈和堆的内存区域的位置，使得这更加困难。ASLR 要求代码是无位置依赖的。在足够熵的情况下，攻击者成功猜测一个或多个地址以实施成功的攻击的机会将大大降低。

这是否意味着 ASLR 已经使代码重用攻击变得过时？不幸的是，并非如此。攻击者有多种方式可以发现受害程序的内存布局。这通常被称为“信息泄露”[@Serna2012]。

由于我们无法仅通过使地址难以猜测来排除代码重用攻击，我们还需要考虑防止攻击者覆盖返回地址和其他代码指针的缓解措施。一些之前描述的缓解措施 earlier，如栈防篡改和库函数加固，可以在特定情况下有所帮助，但对于攻击者已经获得了任意读写原语的一般情况，我们需要更多。

### 2.6.2 控制流完整性（CFI）

[控制流完整性（CFI）](https://en.wikipedia.org/wiki/Control-flow_integrity) 是一系列旨在保护程序预期控制流的缓解措施。这是通过限制间接分支和返回的可能目标来实现的。一种保护间接跳转和调用的方案被称为前向边 CFI，而一种保护返回的方案则被称为实现后向边 CFI。

理想情况下，CFI 方案不应允许任何在正确程序执行中不会发生的控制流转移。然而，不同的方案具有不同的粒度。一般来说，合法的分支目标将被划分为类别，每个类别中的目标在安全目的上被视为等效。一个分支被允许将其控制权转移到其目标类中的任何成员。

CFI 方案有时被分类为粗粒度或细粒度。粗粒度 CFI 方案是使用少量大型等价类的方案，而细粒度方案使用更多的小等价类，这样从给定位置的可能分支目标就更加受限（可能以更高的性能成本为代价）。

例如，一个允许间接函数调用在任何函数开始处继续执行的 CFI 方案会被认为是粗粒度的。如果它限制在具有适当类型签名的函数子集，则它是细粒度的。

正向边 CFI 方案通常依赖于函数类型检查或使用静态分析（指针分析）来识别潜在的控制流转移目标。[@Burow2017]根据精度比较了多种可用的 CFI 方案。例如，对于正向边 CFI 方案，方案根据是否执行，以及其他因素，如流敏感分析、上下文敏感分析和类层次分析进行分类。

下几节将进一步详细介绍常见的 CFI 方案。这些 CFI 方案在生产中被用来强化特定类型的控制流转移。它们包括：

+   [Clang CFI](https://clang.llvm.org/docs/ControlFlowIntegrityDesign.html)

+   arm64e，参见 @McCall2019 和 pauthabi，参见 @Korobeynikov2024

+   [kcfi](https://reviews.llvm.org/D119296)

+   各种影子栈

+   pac-ret，参见 @Cheeseman2019

+   [Arm BTI, Intel IBT](https://en.wikipedia.org/wiki/Indirect_branch_tracking)

+   [Microsoft 控制流保护 (CFG)](https://learn.microsoft.com/en-us/windows/win32/secbp/control-flow-guard)

最常见 CFI 方案的一些关键特性。

| 名称 | 正向边？ | 反向边？ | 精细粒度？ | 基于硬件？ |
| --- | --- | --- | --- | --- |
| Clang CFI | 是 | 否 | 是 | 否 |
| arm64e/pauthabi | 是 | 是 | 是 | 是 |
| kcfi | 是 | 否 | 是 | 否 |
| shadow stack | 否 | 是 | 是 | 取决于 |
| pac-ret | 否 | 是 | 是 | 是 |
| BTI, IBT | 是 | 否 | 否 | 是 |
| 控制流保护 | 是 | 否 | 否 | 否 |

有许多更多的 CFI 方法，通常是学术性的，但其中许多在生产中并不广泛使用。本书主要关注已部署的 CFI 方案。

#### 2.6.2.1 一般 CFI 原则

##### 2.6.2.1.1 保护（正向）间接函数调用

在实践中，大多数生产中的 CFI 方案通过将程序中存在的所有函数划分为等价类来强化间接函数调用。每个函数被分配一个单独的等价类。

对于 C 代码，大多数 CFI 方案要么将所有函数放入一个单一等价类中，要么根据它们的签名对函数进行分区。

例如，arm64e 和 pauthabi 将所有 C 函数放入一个单一等价类中，参见@McCall2019。基于签名对 C 函数进行分区的 CFI 方案示例包括[kcfi](https://reviews.llvm.org/D119296)和[clang cfi](https://clang.llvm.org/docs/ControlFlowIntegrityDesign.html#forward-edge-cfi-for-indirect-function-calls)。

在 C 语言中，考虑以下三个函数：

```asm
void f1(int a) { /* ... */ }
void f2(int* b) { /* ... */ }
void f3(int c) { /* ... */ }
```

函数`f1`和`f3`具有相同的签名，但`f2`具有不同的签名。基于签名对函数进行分区的 CFI 方案会将`f1`和`f3`分配到同一个等价类，而将`f2`分配到不同的等价类。

可能是某些 CFI 方案将所有 C 函数放入单一等价类中的主要原因，因为现实世界的 C 代码经常隐式地将一个 C 函数指针类型转换为另一个。这在技术上是不正确的 C 代码，但恰好能在大多数不使用细粒度 CFI 的平台 上工作。示例@ex:qsort-cfi 说明了这一点。

```asm
#include <stdlib.h>
int cmp_long(const long *a, const long *b) { return *a < *b; }
long sort_array(long *arr, long size) {
  /* The prototype of qsort is:
 void qsort(void *base, size_t nmemb, size_t size,
 int (*compar)(const void *, const void *)); */
  qsort(arr, size, sizeof(long), &cmp_long);
  return arr[0];
}
```

在这个例子中，函数`cmp_long`的签名与`qsort`期望的函数指针类型不同。

此代码将在将所有 C 函数放入单一等价类的 CFI 方案下运行，但在基于签名对 C 函数进行分区的 CFI 方案下将失败。

##### 2.6.2.1.2 保护（前向）虚函数调用

许多 CFI 方案检查 C++虚函数调用是否发生在正确动态类型的对象上。一些例子包括：clang-cfi、arm64e、pauthabi。

```asm
struct A {
  virtual void f();
};
struct B : public A {
  virtual void f();
};
struct C : public A {
  virtual void f();
};
void call_foo(A* a, B* b){
  a->f();
  b->f();
}
struct D : public B {
  virtual void f();
};
```

在这个例子中，一个非常细粒度的 CFI 方案应该允许当`a`是`A`、`B`、`C`或`D`的实例时调用`a->f()`。换句话说，它应该确保调用`A::f`、`B::f`、`C::f`或`D::f`中的任何一个，而不是其他函数。同样，只有当最终调用的是`B::f`或`D::f`而不是`A::f`或`C::f`时，才应该允许调用`b->f()`。

当启用`[`-fsanitize=cfi-cast-strict`选项](https://clang.llvm.org/docs/ControlFlowIntegrity.html#strictness)时，clang-cfi 实现了非常细粒度的 CFI 方案，而 arm64e 和 pauthabi 实现了更粗粒度的 CFI 方案，该方案仅（概率性地）检查对方法`f`的任何调用是否是来自`A::f`的重载函数之一，即`A::f`、`B::f`、`C::f`或`D::f`。这在上述`b->f()`调用上不够精确。

##### 2.6.2.1.3 保护（前向）跳转

对于具有许多值密集排列的 case 的 switch 语句，通常使用[跳转表](https://en.wikipedia.org/wiki/Branch_table)来实现，这是一个指向每个 case 代码的指针或偏移量的数组。最终，跳转到哪个地址是通过从跳转表中加载来计算的，然后执行一个指向计算出的地址的间接跳转。如果攻击者可以控制用于索引跳转表的值，他们可以使跳转目标指向不同的地址，从而导致攻击者接管控制流。

大多数 CFI 方案都不提供对此的保护，但 arm64e 和 pauthabi 做到了，如下面的例子所示。这也在[@McCall2019, slide 39-40]中有所解释。

```asm
  switch (b) {
    case 0:
      return a+1;
    case 1:
      return a-5;
    case 2:

      ... /* cases 3-13 omitted for brevity */

    case 14:
      return a % 4;
    case 15:
      return a & 3;
    }
```

Arm64 为跳转表生成以下汇编代码。为了清晰起见，注释是手动添加的。

```asm
  ;; x0 contains the value of b, which is the switch value.
  adrp  x8, lJTI0_0@PAGE
  add   x8, x8, lJTI0_0@PAGEOFF
  ;; x8 now contains the address of the jump table.
  adr   x9, LBB0_2
  ldrb  w10, [x8, x0]
  ;; w10 now contains the offset in words from LBB0_2
  ;; to the target instruction to jump to.
  add   x9, x9, x10, lsl #2
  ;; x9 now contains the address of the instruction to jump to.
  br    x9
  ;; code emitted for brevity

lJTI0_0:
  .byte (LBB0_2-LBB0_2)>>2
  .byte (LBB0_6-LBB0_2)>>2
  .byte (LBB0_11-LBB0_2)>>2
  .byte (LBB0_10-LBB0_2)>>2
  .byte (LBB0_9-LBB0_2)>>2
  ;; more cases omitted for brevity
```

在这个序列中，如果`x0`中的值是从内存加载的，那么它可能被攻击者控制。如果攻击者可以控制这个值，他们可以通过从进程内存空间中的任何可读位置加载一个字偏移量值，使跳转目标指向几乎任意的地址。

为了防止这种情况，arm64e 和 pauthabi 在加载跳转表偏移量之前检查`x0`中的值是否在范围内：

```asm
  mov   x16, x0
  ;; check that x0 is in range
  cmp   x16, #15
  ;; if x0 is out of range, set switch value to zero (in x16)
  ;; this guarantees that the value will be loaded from the jump
  ;; table which is read-only and cannot be modified by an attacker
  csel  x16, x16, xzr, ls
  adrp  x17, lJTI0_0@PAGE
  add   x17, x17, lJTI0_0@PAGEOFF
  ldrsw x16, [x17, x16, lsl #2]
Ltmp1:
  adr   x17, Ltmp1
  add    x16, x17, x16
  br     x16
  ;; code emitted for brevity

lJTI0_0:
  .long LBB0_2-Ltmp1
  .long LBB0_6-Ltmp1
  .long LBB0_11-Ltmp1
  .long LBB0_10-Ltmp1
  .long LBB0_9-Ltmp1
```

##### 2.6.2.1.4 保护（向后边界的）返回

当一个函数被调用时，调用指令之后的指令地址会被存储在一个寄存器或栈上。这个指向下一个指令的地址被称为“返回地址”。当被调用的函数返回时，它将使用一条指令跳转到返回地址。这是一个间接的控制流，因为分支的目标不是硬编码在指令中，而是来自一个寄存器或内存位置。如果攻击者可以改变返回地址的值，他们可以重定向控制流。

```asm
  ...
  ;; the bl instruction jumps to function f and
  ;; stores the return address, i.e. the address of
  ;; the 'add' instruction, in register x30
  bl f
  add x0, x0, x1
  ...

f:
  ;; stores x29 and x30 on the stack. After executing this
  ;; instruction the return address is in memory, on the stack.
  stp x29, x30, [sp, #16]!
  ...
  ;; load the x29 and x30 registers from the stack. Under the usual
  ;; threat model, an attacker with a write primitive may have overwritten
  ;; the value in memory and may control the value in registers x29 and x30
  ldp x29, x30, [sp], #16
  ;; The return instruction jumps to the address stored in register x30.
  ret x30
```

大多数向后边界的 CFI 方案在执行返回指令之前添加检查，以验证返回地址没有被篡改。

[影子栈]]{.index entry=“影子栈”}方法将返回地址存储在第二个栈上。一些影子栈方法也将返回地址存储在正常栈的原始位置。在这些方法中，在执行返回之前，它会验证常规栈和影子栈上的返回值是否相等。所有影子栈方法都有机制使攻击者难以或无法覆盖影子栈上的返回地址。

仅软件实现的例子是 clang 影子栈，这在第 @sec:clang-shadow-stack 节中有更详细的解释。硬件支持的影子栈包括[Arm 的 Guarded control stack (GCS)](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-a-profile-architecture-2022)，以及[Intel 的 CET Shadow Stack](https://www.intel.com/content/www/us/en/content-details/785687/complex-shadow-stack-updates-intel-control-flow-enforcement-technology.html)。

##### 2.6.2.1.5 可能需要保护的其它代码指针

任何代码指针存储在内存中时，都可能被具有写入原语攻击者修改。前几节给出了代码指针可能源自各种源代码结构的示例，例如函数指针、虚函数表、返回地址等。这个列表并不全面，还有更多源代码结构可能导致代码指针存储在内存中，例如：

+   C++ 协程通常使用包含代码指针的结构来实现。最近，对这些结构的滥用被命名为[协程帧导向编程（CFOP）](https://www.usenix.org/conference/usenixsecurity25/presentation/bajo)。

+   程序链接表（PLT）和[全局偏移表（GOT）](https://en.wikipedia.org/wiki/Global_Offset_Table)通常包含代码指针。一种常见的保护这些表不被攻击者覆盖的方法是在程序启动期间使这些表[只读](https://www.redhat.com/en/blog/hardening-elf-binaries-using-relocation-read-only-relro)。

+   信号处理程序和信号处理程序帧包含代码指针，更多详情请见 @sec:sigreturn-oriented-programming。我们是否应该列出来自其他语言的间接控制流示例？

#### 2.6.2.2 几种 CFI 方案的详细描述

下面，我们将更详细地探讨几种 CFI 方案：

+   Clang CFI 在章节 @sec:clang-cfi

+   Clang 阴影栈在章节 @sec:clang-shadow-stack

+   基于指针认证的 CFI 方案：

    +   pac-ret 在章节 @sec:pac-ret

    +   arm64e 和 pauthabi 在章节 @sec:arm64e-pauthabi

+   分支目标识别（BTI）在章节 @sec:bti

##### 2.6.2.2.1 Clang CFI

[Clang 的 CFI](https://clang.llvm.org/docs/ControlFlowIntegrity.html) 包含各种前向边缘控制流完整性检查。这包括检查间接函数调用的目标是否是正确类型的地址捕获函数。

当使用`-fsanitize=cfi -flto -fvisibility=hidden` 5 编译时，`call_foo`的代码可能看起来像这样：

```asm
00000000004006b4 <call_foo(A*)>:
  4006b4:       a9bf7bfd        stp     x29, x30, [sp, #-16]!
  4006b8:       910003fd        mov     x29, sp
  4006bc:       f9400008        ldr     x8, [x0]
  4006c0:       90000009        adrp    x9, 400000 <_init-0x558>
  4006c4:       91216129        add     x9, x9, #0x858
  4006c8:       cb090109        sub     x9, x8, x9
  4006cc:       d1004129        sub     x9, x9, #0x10
  4006d0:       93c91529        ror     x9, x9, #5
  4006d4:       f100093f        cmp     x9, #0x2
  4006d8:       540000a2        b.cs    4006ec <call_foo(A*)+0x38>
  4006dc:       f9400108        ldr     x8, [x8]
  4006e0:       d63f0100        blr     x8
  4006e4:       a8c17bfd        ldp     x29, x30, [sp], #16
  4006e8:       d65f03c0        ret
  4006ec:       d4200020        brk     #0x1
```

这段代码看起来很复杂，但它所做的就是检查参数的虚表指针（vptr）指向`A`或`B`的虚函数表，这些表是连续存储的，并且是唯一允许的可能性。为不同类型的控制流转移生成的检查是相似的。

##### 2.6.2.2.2 Clang 阴影栈

Clang 还实现了一种称为[阴影栈](https://clang.llvm.org/docs/ShadowCallStack.html)的向后边缘 CFI 方案。在 Clang 的实现中，使用一个单独的栈来存储返回地址，这意味着基于栈的缓冲区溢出不能用来覆盖返回地址。阴影栈的地址是随机化的，并保存在一个专用寄存器中，注意确保它永远不会泄露，这意味着除非通过其他方式发现其位置，否则不能使用任意的写入原语来攻击阴影栈。

例如，当使用`-fsanitize=shadow-call-stack -ffixed-x18` 6 编译时，从之前的栈缓冲区溢出示例生成的`main`函数代码将类似于：

```asm
main:
    cmp w0, #2
    b.lt    .LBB1_2
    str x30, [x18], #8
    stp x29, x30, [sp, #-16]!
    mov x29, sp
    ldr x0, [x1, #8]
    bl  copy_and_print
    ldp x29, x30, [sp], #16
    ldr x30, [x18, #-8]!
.LBB1_2:
    mov w0, wzr
    ret
```

你可以看到，影子栈地址被保存在`x18`寄存器中。返回地址也保存在“正常”栈上，以与 unwinders 兼容，但实际上并不用于函数返回。

##### 2.6.2.2.3 指针认证

除了软件实现之外，还有许多基于硬件的 CFI（控制流完整性）实现。基于硬件的实现有可能提供比仅软件的 CFI 方案更好的保护和性能。

例如，指针认证[@Rutland2017]，是 Armv8.3 的一个特性，仅在 AArch64 状态下受支持，可以用来减轻代码重用攻击。

指针认证为给定地址计算一个指针*签名*，称为指针认证码（PAC），见图@fig:pauth-sign-auth。PAC 代码存储在指针的较高位，这些位通常未被使用。

在较高位包含 PAC 代码的指针被称为*签名指针*。未签名的指针称为*原始指针*。

指针认证背后的基本思想是，攻击者会尝试利用内存漏洞覆盖内存中的指针。指针认证旨在检测攻击者是否覆盖了内存中的指针。它是通过确保指针：

1.  在内存中始终进行签名，并且

1.  在将指针加载到寄存器并使用它之间，指针会被进行认证。

如果认证失败，程序将产生故障。

根据要签名的指针类型，可以使用指针认证实现不同的加固方案，例如仅对返回地址进行签名、对所有函数指针进行签名，或更多。

指针认证有效的一个关键方面是使攻击者难以构造出正确的 PAC（指针认证码），以便通过认证。为了实现这一点，除了地址之外，还使用了另外两个输入来计算 PAC：一个所谓的密钥和一个修改器：

+   密钥是一个软件无法直接访问的秘密值，因此攻击者无法检索密钥值。这使得攻击者难以离线计算给定地址的 PAC 值。密钥可以被视为一种[盐](https://en.wikipedia.org/wiki/Pepper_(cryptography))

+   我们还希望避免攻击者从程序的一个上下文中获取一个签名指针，并在不同的上下文中使用它。修饰符是特定于指针使用上下文的值。不同的强化方案将使用不同的修饰符。两种基于指针认证的不同强化方案的示例在章节 @sec:pac-ret（pac-ret 强化方案）和 @sec:arm64e-pauthabi（arm64e/pauthabi 强化方案）中描述。修饰符可以被视为一种 [盐](https://en.wikipedia.org/wiki/Salt_(cryptography))。

    当攻击者成功从一个上下文中获取一个签名指针，并用它覆盖另一个上下文中的另一个指针时，这被称为指针替换攻击。为不同上下文使用不同的修饰符使得指针替换攻击更加困难。

![AArch64 签名和认证操作以将原始指针转换为签名指针以及相反操作](img/file4.svg)

AArch64 签名和认证操作以将原始指针转换为签名指针以及相反操作

如上所述的指针认证指令可以用来实现各种强化方案。在这本书中，我们只详细介绍了两种在生产中用于数十亿设备的方案：pac-ret 和 arm64e/pauthabi。

我们没有进一步介绍的基于指针认证的其他强化方案包括：PACStack [@Liljestrand2021]、Camouflage [@DenisCourmont2021]、PAL [@Yoo2021]、PTAuth [@farkhani2021]、PAC it up [@Liljestrand2019]、FIPAC [@Schilling2022]、[结构保护](https://discourse.llvm.org/t/rfc-structure-protection-a-family-of-uaf-mitigation-techniques/85555) 以及更多。其中一些强化方案除了保护控制流外，还能以其他方式保护二进制文件免受攻击。

###### 2.6.2.2.3.1 pac-ret：向后边界的 CFI

[Clang](https://clang.llvm.org/docs/ClangCommandLineReference.html#aarch64) 和 [GCC](https://gcc.gnu.org/onlinedocs/gcc/AArch64-Options.html) 在使用 `-mbranch-protection=pac-ret` 标志编译时，都使用指针认证进行返回地址签名。通过示例来说明其工作原理最为直观：

当使用 `pac-ret` 功能编译示例 @ex:stack-buffer-overflow 中的 `main` 函数时，编译器将生成：

```asm
main:
    cmp     w0, #2
    b.lt    .LBB1_2
    paciasp
    stp     x29, x30, [sp, #-16]!
    ldr     x0, [x1, #8]
    mov     x29, sp
    bl      copy_and_print
    ldp     x29, x30, [sp], #16
    autiasp
.LBB1_2:
    mov     w0, wzr
    ret
```

注意 `paciasp` 和 `autiasp` 指令。在进入此函数时，返回地址，即函数在执行 `ret` 指令时将跳转回的地址，被存储在寄存器 `x30` 中。

指令 `paciasp` 计算寄存器 `x30` 中返回地址的 PAC，并将其存储在 `x30` 的高位。PAC 是从以下“输入”计算得出的：

1.  存储在 `x30` 寄存器中的地址，即返回地址，

1.  秘密密钥 `IA`，如指令 `paciasp` 中的 `ia` 所示。该密钥对程序不可访问。

1.  作为修饰符，当前栈指针 (`sp`) 的值，如指令 `paciasp` 中的 `sp` 所指示。

在执行`paciasp`指令后，`x30`中的值是一个有符号指针。`stp`指令将这个有符号指针存储到内存中。在通常的威胁模型下，一个具有写原始权限的攻击者可以在指针在内存中时修改其值。因此，在通过`ldp`指令再次将值加载到`x30`之后，应该认为它可能已被篡改。

因此，编译器在从内存中加载有符号指针并在`ret`指令中使用它之间插入`autiasp`指令。`autiasp`指令验证`x30`高位的 PAC，考虑到秘密密钥`IA`和修改器`sp`。如果 PAC 是正确的，这在正常执行中是常见的情况，地址的扩展位将被恢复，以便可以在`ret`指令中使用该地址。然而，如果 PAC 是不正确的，高位将被损坏，因此后续使用该地址（例如在`ret`指令中）将导致错误。

通过确保我们不存储任何没有 PAC 的返回地址，我们可以显著降低 ROP 攻击的有效性：由于秘密密钥无法被攻击者检索，攻击者无法计算给定地址和修改器的正确 PAC，并且被限制于猜测它。

在给定系统配置中，猜测 PAC 的成功概率取决于可用的 PAC 比特数的精确数量。

认证的指针容易受到指针替换攻击，其中使用给定修改器签名的指针被替换为另一个也使用相同修改器签名的不同指针。在`pac-ret`方案中，这通过使用栈指针作为修改器来缓解，这限制了已签名返回地址指针的重用，仅限于具有相同栈指针值的函数帧。

###### 2.6.2.2.3.2 arm64e 和 pauthabi：前向边 CFI

在 arm64e 或 pauthabi 中使用 pauth 应该更详细地解释，包括签名和认证预言机或 Oracle 的概念 [#259](https://github.com/llsoftsec/llsoftsecbook/issues/259)

指针认证也可以更广泛地使用，例如实现前向边 CFI 方案，就像在 arm64e ABI 中那样[@McCall2019]。然而，指针认证指令足够通用，也可以用于实现更通用的内存安全措施，而不仅仅是 CFI。

##### 2.6.2.2.4 BTI 和其他粗粒度 CFI 方案

[分支目标识别（BTI）](https://developer.arm.com/documentation/102433/0100/Jump-oriented-programming?lang=en)，在 Armv8.5 中引入，提供了粗粒度前向边保护。使用 BTI，间接分支的目标位置必须用新的指令`BTI`标记。有四种不同的 BTI 指令允许不同类型的间接分支（间接跳转、间接调用、两者或都不允许）。对非 BTI 指令或错误类型的 BTI 指令的间接分支将引发分支目标异常。

Clang 和 GCC 都支持生成 BTI 指令，使用`-mbranch-protection=bti`标志，或者，为了启用 BTI 和指针认证的返回地址签名，使用`-mbranch-protection=standard`。

BTI 的两个方面可以简化其部署：可以标记单个页面为受保护或不受保护，上述描述的 BTI 检查仅适用于针对受保护页面的间接分支。此外，BTI 指令已被分配到提示空间，因此在不支持 BTI 的核心中，它将作为无操作执行，有助于其采用。

粗粒度前向边 CFI 的另一种实现是 Windows [控制流保护](https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard)，它只允许调用标记为有效间接控制流目标的函数。

#### 2.6.2.3 CFI 实现陷阱

当实施如上所述的 CFI 措施时，重要的是要意识到影响类似方案已知弱点。[@Conti2015] 描述了当某些寄存器在栈上溢出时，CFI 实现可能会受到影响，这些寄存器可能被攻击者控制。例如，如果一个包含刚刚验证过的函数指针的寄存器被溢出，检查可以通过覆盖溢出的指针来有效地绕过。

讨论了针对代码重用攻击的各种缓解措施后，现在是时候将我们的注意力转向不同类型的攻击，这些攻击不试图覆盖代码指针：针对非控制数据的攻击，这将是下一节的主题。

## 2.7 非控制数据攻击

在前面的章节中，我们关注了通过覆盖控制数据来颠覆控制流，这些数据用于改变程序计数器的值，例如返回地址和函数指针。由于这些类型的攻击非常突出，许多缓解措施都是为了保持控制流完整性而设计的。非控制数据攻击（entry=“non-control data attacks”），也称为仅数据攻击，可以完全绕过这些缓解措施，因为它们修改的数据不是这些缓解措施所保护的控制数据。

非控制数据攻击的范围可以从非常简单的针对单个数据点的攻击到非常复杂的具有非常高表达性的攻击 [@Beer2021]。一个非常简单的例子可能看起来像这样：

```asm
// Returns zero for failure, non-zero for success.
int authenticate() {
  int authenticated = 0;
  char passphrase[10];
  if (fgets(passphrase, 20, stdin)) {     // buffer overflow
    if (!strcmp(passphrase, "secret\n")) {
      authenticated = 1;
    }
  }
  return authenticated;
}
```

示例展示了一个简化的 7 函数，该函数从用户那里读取密码，将其与已知值进行比较，并将一个整型堆栈变量设置为指示“认证”是否成功。该函数包含一个非常明显的缓冲区溢出，因为传递给 `fgets` 的字符串长度限制与缓冲区大小不匹配。

图 @fig:non-control-data-attack 展示了当使用 Clang 10.08 编译 AArch64 代码时此函数的堆栈帧布局。如图所示，`passphrase` 的溢出将覆盖 `authenticated`，将其设置为非零值，即使密码输入不正确。随后，`authenticate` 函数将返回非零值，错误地指示认证成功。

![`authenticate` 的堆栈帧](img/file5.svg)

`authenticate` 的堆栈帧

对于更多在真实应用中可能发生的简单数据攻击示例，请参阅 [@Chen2005]。尽管这清楚地表明数据攻击是一个真实的问题，但它留下了一个非常重要的问题：此类攻击的极限是什么？虽然人们可能会倾向于假设数据攻击在本质上是有局限性的，但[@Hu2016] 中已经证明，实际上它们可以非常灵活。[@Hu2016] 描述了面向数据编程（DOP），这是一种针对易受攻击程序构建数据攻击的通用方法，从程序中的已知内存错误开始 9。

[@Hu2016] 的作者描述了一种名为 MINDOP 的小型语言，它具有虚拟指令集和虚拟寄存器。MINDOP 的虚拟寄存器对应于内存位置。MINDOP 指令对应于这些虚拟寄存器上的操作，例如将值加载到虚拟寄存器中，从虚拟寄存器中存储值，算术运算，甚至条件和无条件跳转。作者展示了如何识别代码中实现各种 MINDOP 指令并从内存错误可达的小工具，以及如何借助调度器小工具将这些小工具组合在一起，调度器小工具的作用是专门用于将小工具串联起来。

将小工具组合在一起对于交互式攻击来说更简单，攻击者可以不断提供恶意输入以触发初始内存错误和一系列小工具，所需次数不限。对于非交互式攻击，还需要使用 MINDOP 跳转操作，这些操作与一个提供虚拟程序计数器的内存位置一起使用。

创建 DOP 攻击的过程并不简单，也不是完全自动化的。相关文献[@Ispoglou2018] 关注于自动化数据攻击。

当阅读关于最近安全问题的文章时，你更可能遇到“原始”这个词，而不是与数据导向设备相关的术语，这个词在前面的部分中已有描述。这些概念是相关的：例如，一个任意的读取原始操作可以通过链式连接（可能很大）数量的 DOP 设备来产生。讨论原始操作提供了一个更高级别的抽象层次，因为它通常更容易用高级操作来推理，而不是需要拼接成许多小段代码以执行操作。

总结来说，数据仅攻击是一个重大的关注点。因为我们看到的大多数缓解技术都是控制流导向的，它们在设计上不足以保护这种不同类型的攻击。在下一节中，我们将探讨我们如何从源头解决这些问题：内存错误。

## 2.8 防止和检测内存错误

到目前为止，我们已经讨论了像 C 和 C++这样的非内存安全语言容易受到内存错误的影响，因此容易受到利用。在本节中，我们将讨论 C/C++程序员可用的工具，以帮助他们检测可能导致内存错误的漏洞。

### 2.8.1 清理器

清理器是在程序执行期间检测错误的工具。清理器通常有两个组件：一个编译器工具集部分，它引入了新的检查，以及一个运行时库部分。它们通常在生产模式下运行成本太高，因为它们往往会增加执行时间和内存使用。它们通常在应用程序测试期间使用，经常与模糊器 10 结合使用。

一个非常流行的清理器是[地址清理器（ASan）](https://clang.llvm.org/docs/AddressSanitizer.html)。它的目标是检测各种内存错误。这包括越界访问、使用后释放、双重释放和无效释放 11。GCC 和 Clang 都有地址清理器的实现，但我们将重点放在 Clang 实现上。

ASan 使用影子内存来跟踪应用程序内存的状态。影子内存中的每个字节记录了应用程序内存中 8 个字节的信息。它表示这 8 个字节中有多少是可寻址的。当没有字节是可寻址的，它将编码额外的细节（例如，这 8 个字节是否超出栈边界、超出堆边界、已释放的内存等）。对于每 8 个应用程序内存字节需要 1 个影子内存字节意味着 ASan 需要预留应用程序虚拟地址空间的一八分之一 [@Serebryany2012]。影子内存是在一个连续的块中分配的，这使得将应用程序内存映射到影子内存变得简单。

ASan 的运行时库用其专用的版本替换了内存分配函数，如`malloc`和`free`。`malloc`在每个分配前后引入红区，这些红区被标记为不可寻址。`free`将整个分配标记为不可寻址，并将其置于隔离区，以便在一段时间内（基于 FIFO 原则）不会重新分配。这允许检测使用后释放。运行时库还处理影子内存的管理。

ASan 在编译器中对代码进行代码插桩时，在每一个栈数组分配和全局变量周围引入了红区。然后，它对加载和存储进行插桩，根据存储在影子内存中的信息检查访问的内存是否可寻址，如果访问不可寻址的内存，则报告错误。

ASan 不会产生误报，并且易于使用。它需要使用`-fsanitize=address`选项编译和链接程序。在实践中，它用于测试[大型项目](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/asan.md)。Linux 内核中也有类似的动态内存错误检测工具，[KASAN](https://www.kernel.org/doc/html/v5.0/dev-tools/kasan.html)。

ASan 最大的缺点是其高运行时开销和内存使用，这归因于隔离区、红区和影子内存。[硬件辅助地址 Sanitizer (HWASAN)](https://clang.llvm.org/docs/HardwareAssistedAddressSanitizerDesign.html)与 ASan 类似，但通过部分硬件辅助可以降低内存开销，但牺牲了可移植性。

在 AArch64 上，HWASAN 使用顶部字节忽略（TBI）。当 TBI 启用时，在执行内存访问时忽略指针的顶部字节，允许软件使用该顶部字节存储元数据，而不影响执行。每个分配都对齐到 16 字节，每个 16 字节的内存块（称为“粒度”）随机分配一个 8 位标签。标签存储在影子内存中，并放置在指向对象的指针的顶部字节。然后对内存加载和存储进行插桩，以检查指针中存储的标签是否与内存中存储的标签匹配，并在发生不匹配时报告错误。添加图表以演示 HWASAN 的工作原理 [#168](https://github.com/llsoftsec/llsoftsecbook/issues/168)

对于小于 16 字节的粒度，影子内存中存储的值不是实际的标签，而是粒度的长度。实际的标签存储在粒度本身的最后一个字节。对于影子内存中值为 1 到 15 的标签，HWASAN 检查访问是否在粒度范围内，并且指针标签与粒度最后一个字节中存储的标签匹配。

HWASAN 也易于使用，只需使用`-fsanitize=hwaddress`标志编译和链接应用程序即可。

[MemTagSanitizer](https://llvm.org/docs/MemTagSanitizer.html)更进一步，并使用 Armv8.5-A [内存标记扩展 (MTE)](https://developer.arm.com/documentation/102925/0100)。使用 MTE，标签检查由硬件自动完成，并在不匹配时引发异常。MTE 的粒度大小为 16 位，而标签为 4 位。考虑添加一个关于 MTE 及其应用的整个章节 [#169](https://github.com/llsoftsec/llsoftsecbook/issues/169)

[UndefinedBehaviorSanitizer (UBSan)](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html#ubsan-checks)在程序执行期间检测未定义的行为，例如静态确定的数组边界之外的数组访问、空指针解引用、有符号整数溢出以及各种导致数据丢失的整数转换。尽管其中一些检查与内存错误没有直接关系，但这些类型的错误可能导致错误的指针运算、错误的分配大小以及其他导致内存错误的问题，因此检测和解决这些问题非常重要。

UBSan 的文档描述了所有可用的检查。其中大部分检查通过`-fsanitize=undefined`标志启用，但也有其他有用的检查分组，例如与整数转换和算术相关的`-fsanitize=integer`。

还有许多其他清理器，超出了本节合理涵盖的范围。对于感兴趣的读者，我们列出了一些更多：

+   [MemorySanitizer](https://clang.llvm.org/docs/MemorySanitizer.html)：检测未初始化的读取。

+   [ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html)：检测数据竞争。

+   [GWP-ASan](https://llvm.org/docs/GwpAsan.html)：检测使用后释放和堆缓冲区溢出，具有低开销，使其适用于生产环境。它仅在分配样本上执行检查。

描述检测内存错误的其他机制，包括基于软件的（静态分析、库和缓冲区硬化）和基于硬件的，例如基于 PAuth 的指针完整性方案、MTE 等 [#170](https://github.com/llsoftsec/llsoftsecbook/issues/170)

### 2.8.2 边界检查

确保内存访问发生在每个对象分配的边界内是内存安全的重要组成部分。这通常用“空间内存安全”这个术语来描述。越界访问会导致受限的读写原语 12。攻击者通常可以轻松地将这些转换为任意的读写原语。例如，这可以通过覆盖目标内存访问问题的对象之后的分配中的指针字段来实现。

C 和 C++ 内存语言通常不执行边界检查 13。这是 C/C++ 程序中内存错误的一个来源。然而，编译器有引入边界检查的历史，尽管语言没有要求这样做，但这是为了提高现有 C/C++ 代码库的安全性。

最简单的编译器选项之一是 `-Warray-bounds`，它在数组访问始终超出边界时发出警告。因此，此选项仅限于具有静态已知大小的数组。GCC 和 Clang 都支持此选项。

两个编译器都支持的另一个选项是 `-fsanitize=bounds`，包含在 UBSan 中，它在运行时检查对静态大小数组的访问边界。这处理了比 `-Warray-bounds` 更多的案例，因为它还可以检查对动态索引的访问。然而，它仍然有限制，因为它不能对动态大小数组执行边界检查，并且它仍然仅限于数组边界检查。一个更全面的解决方案还应涵盖指针，特别是如果执行指针运算。

你可能会注意到 `-fsanitize=bounds` 引入的边界检查与 Address Sanitizer 之间有一些重叠。尽管 `-fsanitize=bounds` 的作用域仅限于静态大小数组，但值得注意的是，它仍然可以捕获 Address Sanitizer 无法捕获的对象成员访问中的内部溢出，因为访问仍然在分配内。例如，给定以下代码：

```asm
struct foo {
  int a[6];
  int b;
};

int get(struct foo *x, int i) {
  return x->a[i];
}
```

对 `get(f, 6)` 的调用将使用 `-fsanitize=bounds` 发生错误，但不会使用 `-fsanitize=address`。

Clang 和 GCC 还支持两个内置函数，可以返回关于变量大小的信息。`__builtin_object_size` 可以用于具有静态已知大小的对象，并且始终在编译时评估，而 `__builtin_dynamic_object_size` 也可以从标记为具有 `alloc_size` 函数属性的分配函数中传播动态信息[`gcc.gnu.org/onlinedocs/gcc-4.7.2/gcc/Function-Attributes.html`](https://gcc.gnu.org/onlinedocs/gcc-4.7.2/gcc/Function-Attributes.html)。这两个内置函数可以用于在用户或库代码中引入边界检查。例如，`_FORTIFY_SOURCE` 宏[`man7.org/linux/man-pages/man7/feature_test_macros.7.html`](https://man7.org/linux/man-pages/man7/feature_test_macros.7.html) 指示 `glibc` 在各种字符串和内存操作函数中引入边界检查，例如 `memcpy`。随着宏值的增加（当前使用的值是 1-3），检查的数量也会增加。例如，较低的两个级别不会使用 `__builtin_dynamic_object_size` 内置函数，因为它有运行时开销，这超出了检查本身的额外开销。

为了支持动态大小数组的边界检查，针对 [GCC](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=108896) 和 [Clang](https://reviews.llvm.org/D148381) 的最新提议建议添加一个结构体成员属性，`element_count`。此属性将应用于结构体中的 [灵活数组成员](https://en.wikipedia.org/wiki/Flexible_array_member)，表示结构体中另一个表示数组长度的成员。

`[-fbounds-safety](https://discourse.llvm.org/t/rfc-enforcing-bounds-safety-in-c-fbounds-safety/70854)` 提案更进一步，引入了一种可以更广泛地应用于指针的类似注解。该提案还旨在通过仅在 [应用程序二进制接口 (ABI)](https://en.wikipedia.org/wiki/Application_binary_interface) 边界处要求注解来减轻程序员所承担的注解负担 14。不跨越 ABI 边界的局部变量将隐式转换为使用宽指针。这些宽指针与原始指针一起存储边界信息。

此外，还有一些针对 C++ 代码库的加固努力。例如，[libc++ 加固模式](https://libcxx.llvm.org/Hardening.html) 启用了一些旨在捕获库中未定义行为的断言。[C++ 缓冲区加固提案](https://discourse.llvm.org/t/rfc-c-buffer-hardening/65734) 旨在扩展这种库加固。该提案还将引入一种编程模型，其中所有指针运算都被视为不安全。指针运算必须用 C++ 库中的替代方案替换，例如 `std::array`。加固库中这些替代方案的实现将包括边界检查。

成功使用边界检查编译器功能对大型代码库进行操作需要大量的努力。例如，将 Linux 内核重构以使用灵活数组的边界检查，正如在 [@Cook2023] 中所述。

此外，还有基于硬件的空间内存安全违规的缓解措施。例如，[CHERI](https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/) 向传统的指令集架构引入了 *能力*。能力结合了一个虚拟地址及其描述其对应边界和权限的元数据。能力不能伪造，因此可以提供非常强的保证。Arm 开发了一种原型架构，该架构适应了 CHERI，以及一个原型 SoC 和开发板，作为 [Arm Morello 项目](https://www.arm.com/architecture/cpu/morello) 的一部分。

当然，减轻空间内存安全漏洞的另一种方法是使用专门为空间内存安全性设计的语言。这类语言确保所有内存访问都经过检查，无论是在编译时还是在运行时。例如，[Rust 编程语言](https://www.rust-lang.org/)在编译器无法证明访问在范围内时引入边界检查 15。还有许多其他内存安全的语言，具有不同的特性。一个例子是 JavaScript，这是一种动态类型、通常 JIT 编译的语言。我们将在下一节讨论在实现此类语言支持时出现的一些问题。

## 2.9 JIT 编译器漏洞

编译器正确性显然非常重要，因为即使源代码没有错误，错误编译也会创建有缺陷的程序。可能不那么明显的是，这些错误可能具有安全影响。例如，它们可以在其他情况下内存安全的语言中引入内存安全错误。在某些情况下，一个错误可能不会影响大多数程序，并且在检测和修复之前不会在实际中引起安全问题。当然，这是假设该错误没有被故意注入到编译器中。

编译器错误是[即时编译器（JIT）](https://en.wikipedia.org/wiki/Just-in-time_compilation)中一个有趣的来源，这些错误可能导致安全问题 16。JIT 编译通常用于在程序执行期间接收源代码作为输入的程序中，例如在网页浏览器中，用于执行网页中包含的 JavaScript 代码。在这种情况下，JIT 编译器的输入来自任意网站，因此是不可信的。这种 JIT 编译器中的错误可能导致整个程序（在此处为浏览器）被破坏，如果恶意输入（例如来自恶意网站）故意触发错误编译，以破坏正在实现的语言的内存安全性。

对于本节，我们专注于 JavaScript，这是一种动态类型、内存安全的语言，但我们讨论的担忧也适用于其他动态编译的语言。

没有静态已知类型的情况下，为了优化 JavaScript 代码，JavaScript 引擎会求助于类型分析[@Pizlo2020]，记录执行代码时遇到的类型。然后，在优化过程中使用这些类型，它推测在代码的后续运行中会遇到相同的类型，并插入检查以验证这些关于类型的假设仍然成立。当检查失败时，优化后的代码会被替换为可以处理所有类型的未优化代码，这个过程称为去优化或栈上替换（OSR）。去优化确保去优化函数的状态在类型检查失败执行的点被正确地重新创建。

例如，一个函数如下：

```asm
function foo(x, y) {
  return x + y;
}
```

当`x`和`y`都是数字时将返回一个数字，但当任一为字符串时将返回一个字符串。优化编译器可以使用分析结果生成优化代码。例如，当在分析期间两个参数都是整数时，它可以生成如下伪代码的代码：

```asm
foo:
  if x not integer, deoptimize
  if y not integer, deoptimize
  result = x + y
  if overflowed, deoptimize
  return result
```

你可能想知道类型检查是如何实现的，这与 JavaScript 引擎中值的表示密切相关[@Wingo2011]。简而言之，JavaScript 引擎使用特定的位模式来指示一个值是否应该被解释为指针，或者作为整数或浮点值。例如，[V8 JavaScript 引擎](https://v8.dev/)使用最低有效位来表示一个[值是指针](https://v8.dev/blog/pointer-compression#value-tagging-in-v8)，否则它是一个小整数（需要将其右移以访问其值）。指针随后指向包含一个[隐藏类](https://v8.dev/docs/hidden-classes)成员的对象，该成员用于类型检查。

除了在分析期间收集的类型信息之外，优化 JavaScript 编译器还将分析到的类型传播到相关值。例如，如果预期值`x`是一个字符串，并且我们检查这个假设，那么`x + 1`也将是一个字符串（在这种情况下不需要额外的检查）。除了简单的类型传播之外，它们通常执行范围分析，以尽可能精确地确定值的范围，这对于边界检查消除很有用。

边界检查消除（Bounds Check Elimination，BCE）是执行数组访问边界检查的语言中的一种常见优化，以确保每个访问的索引都在数组的边界内。当边界检查被证明是冗余时，BCE 会去除边界检查，例如，当数组访问使用一个已知小于数组长度的常量索引时。有关 JavaScript 中越界数组访问行为的详细信息，请参阅[这里](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length)。

范围分析是一个很好的例子，说明即时编译器（JIT）的 bug 可以引入漏洞。不正确的范围分析结果可以被边界检查消除（bounds check elimination）用来错误地消除实际上应该保留在优化代码中的边界检查。例如，对于以下函数：

```asm
function foo(x) {
  y = bar(x);
  var a = [0, 1, 2];
  return a[y];
}
```

如果范围分析决定`y`的值在范围`[0, 2]`内，但实际上值在范围`[0, 3]`内，那么对于访问`a[y]`的边界检查可能会被错误地消除，假设访问是在边界内的。[@Glazunov2021]列出了一些类似的假设漏洞的例子，以及影响了广泛使用的 JavaScript 引擎的此类漏洞的例子。

上文描述的错误类型为攻击者提供了一个有限的读写原语，因为数组分配发生了线性溢出。攻击者可以在此基础上构建以获得任意的读写原语。由于 JIT 编译器在运行时生成可执行代码，它们通常使用可读写和可执行的内存。这种内存对攻击者非常有用，攻击者可以使用任意的写入原语将他们的有效载荷复制到这段代码内存中，然后跳转到它。因此，可读写和可执行内存使 JIT 成为攻击者的诱人目标。

与范围分析相关的错误只是 JavaScript 引擎中遇到的一种常见错误类型。[@Groß2022] 列出了其他一些常见的错误类型，这些错误会导致 JavaScript 引擎中时间内存和空间内存安全以及类型安全的违规。

我们如何防御此类漏洞？有几种互补的方法，例如：

1.  使用模糊测试来发现编译器错误。对于 JavaScript，一个有用的模糊测试工具是 [Fuzzilli](https://github.com/googleprojectzero/fuzzilli)。

1.  在处理易出错的编译器优化，如边界检查消除时，要更加谨慎。例如，[V8 JavaScript 引擎](https://v8.dev/) 引入了[针对类型错误的边界检查加固](https://bugs.chromium.org/p/v8/issues/detail?id=8806) 17。

1.  而不是试图防止编译器（和其他）错误，假设它们将存在，并引入缓解措施，以防止攻击者基于错误提供的初始有限原语构建任意的读写原语。例如，对于 64 位架构，V8 实现了一个基于[指针压缩](https://v8.dev/blog/pointer-compression)的[沙盒](https://docs.google.com/document/d/1FM4fQmIhEqPG8uGp5o9A-mnPB5BOeScZYpkHjo0KKA8/edit)。通过指针压缩，指针由基指针的 32 位索引表示，而不是完整的 64 位值。通过确保沙盒（JavaScript 堆所在的位置）内的所有指针都进行了压缩，并且压缩指针始终指向沙盒内部，一个允许在沙盒内覆盖内存的有限原语就不能用来通过覆盖指针值构建任意的读写原语。

1.  防止代码内存同时可执行和可写也是理想的选择。这被称为[W^X](https://en.wikipedia.org/wiki/W%5EX)。W^X 的一个简单实现是仅根据页表切换内存权限，但这种做法在涉及多个线程时，不足以防止攻击者向代码内存写入[@Song2015]。一个更有效的解决方案是使用单独的编译过程，这是唯一具有写入 JIT 代码内存权限的过程。或者，一些架构提供了特殊功能，可以限制用户空间基于页面的内存权限，从而有效地允许不同线程有不同的权限。这些功能在实现 W^X 时也可能很有用。对于 AArch64，这个功能被称为[权限覆盖](https://developer.arm.com/documentation/102376/0200/Permission-indirection-and-permission-overlay-extensions)。

在本节中，我们讨论了即时编译器（JIT）的安全性，并描述了导致漏洞的 JavaScript 编译器错误。尽管我们没有专注于 JavaScript 利用的细节，但感兴趣的读者可以参考[@saelo2021a]和[@saelo2021b]。

* * *
