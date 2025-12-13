# 软件测试简介

> [`www.fuzzingbook.org/html/Intro_Testing.html`](http://www.fuzzingbook.org/html/Intro_Testing.html)

在我们进入本书的核心部分之前，让我们先介绍软件测试的基本概念。为什么需要测试软件呢？一个人该如何测试软件？一个人如何判断一个测试是否成功？一个人如何知道是否测试得足够了？在这一章中，让我们回顾最重要的概念，同时熟悉 Python 和交互式笔记本。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('C8_pjdl7pK0') 
```

本章（以及本书）并不打算取代测试方面的教科书；请参阅结尾处的背景，以获取推荐的阅读材料。

## 简单测试

让我们从简单的例子开始。你的同事被要求实现一个平方根函数 $\sqrt{x}$。（让我们暂时假设环境中还没有这样的函数。）在研究了[牛顿-拉夫森方法](https://en.wikipedia.org/wiki/Newton%27s_method)之后，她提出了以下 Python 代码，声称实际上这个`my_sqrt()`函数可以计算平方根。

```py
def my_sqrt(x):
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx 
```

你的任务是找出这个函数是否真的做了它声称要做的事情。

### 理解 Python 程序

如果你刚开始接触 Python，你可能首先需要理解上述代码的功能。我们非常推荐阅读[Python 教程](https://docs.python.org/3/tutorial/)，以了解 Python 是如何工作的。理解上述代码最重要的三件事是这些：

1.  Python 通过*缩进*来构建程序结构，因此函数和`while`循环体是通过缩进来定义的；

1.  Python 是*动态类型化的*，这意味着变量如`x`、`approx`或`guess`的类型是在运行时确定的。

1.  Python 的大多数语法特性都受到了其他常见语言的影响，例如控制结构（`while`、`if`）、赋值（`=`）或比较（`==`、`!=`、`<`）。

有了这些，你就可以理解上述代码的功能了：从一个`guess`值`x / 2`开始，它在`approx`中计算越来越好的近似值，直到`approx`的值不再改变。这就是最终返回的值。

### 运行一个函数

为了找出`my_sqrt()`函数是否工作正确，我们可以用几个值来*测试*它。例如，对于`x = 4`，它会产生正确的值：

```py
my_sqrt(4) 
```

```py
2.0

```

上面的`my_sqrt(4)`部分（所谓的*单元*）是 Python 解释器的输入，默认情况下会*评估*它。下面的部分（`2.0`）是它的输出。我们可以看到`my_sqrt(4)`产生了正确的值。

对于`x = 2.0`，情况似乎也是一样：

```py
my_sqrt(2) 
```

```py
1.414213562373095

```

### 与笔记本交互

如果你在这个交互式笔记本中阅读，你可以尝试用其他值来测试`my_sqrt()`。点击上面带有`my_sqrt()`调用的其中一个单元格，并更改其值——比如说，改为`my_sqrt(1)`。按`Shift+Enter`（或点击播放符号）来执行它并查看结果。如果你得到一个错误信息，请转到上面定义`my_sqrt()`的单元格并首先执行这个操作。你也可以一次性运行所有单元格；有关详细信息，请查看笔记本菜单。（实际上，你也可以通过点击来更改文本，并纠正如这句话中的错误。）

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import quiz 
```

### 习题

`my_sqrt(16)`会产生什么结果？

通过取消以下行的注释并执行它来亲自尝试：

```py
# my_sqrt(16) 
```

执行单个单元格不会执行其他单元格，所以如果你的单元格依赖于另一个单元格中的定义，而你尚未执行该定义，你会得到一个错误。你可以从菜单中选择`运行所有单元格以上`来确保所有定义都已设置。

还要注意，除非被覆盖，否则所有定义都会在执行之间保留。有时，因此重启内核（即从头开始启动 Python 解释器）以消除旧的、多余的定义是有帮助的。

### 调试一个函数

要了解`my_sqrt()`是如何工作的，一个简单的策略是在关键位置插入`print()`语句。例如，你可以记录`approx`的值，以查看每次循环迭代如何逐渐接近实际值：

```py
def my_sqrt_with_log(x):
  """Computes the square root of x, using the Newton–Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        print("approx =", approx)  # <-- New
        approx = guess
        guess = (approx + x / approx) / 2
    return approx 
```

```py
my_sqrt_with_log(9) 
```

```py
approx = None
approx = 4.5
approx = 3.25
approx = 3.0096153846153846
approx = 3.000015360039322
approx = 3.0000000000393214

```

```py
3.0

```

交互式笔记本还允许启动一个交互式调试器——在单元格顶部插入一个“魔法行”`%%debug`并查看会发生什么。不幸的是，交互式调试器会干扰我们的动态分析技术，所以我们主要使用日志和断言进行调试。

### 检查一个函数

让我们回到测试。我们可以读取并运行代码，但上述`my_sqrt(2)`的值实际上是否正确？我们可以通过利用$\sqrt{x}$平方再次必须是$x$来轻松验证，换句话说，$\sqrt{x} \times \sqrt{x} = x$。让我们看看：

```py
my_sqrt(2) * my_sqrt(2) 
```

```py
1.9999999999999996

```

好吧，我们确实有一些舍入误差，但除此之外，这似乎很正常。

我们现在所做的是测试上述程序：我们在给定的输入上执行它，并检查其结果是否正确。这种测试是在程序投入生产前的质量保证的最基本要求。

## 自动化测试执行

到目前为止，我们都是手动测试上述程序，即手动运行它并手动检查其结果。这是一种非常灵活的测试方式，但从长远来看，它相当低效：

1.  手动地，你只能检查非常有限数量的执行及其结果

1.  在对程序进行任何更改后，你必须重复测试过程

这就是为什么自动化测试非常有用。一个简单的方法是让计算机首先进行计算，然后让它检查结果。

例如，这段代码会自动测试$\sqrt{4} = 2$是否成立：

```py
result = my_sqrt(4)
expected_result = 2.0
if result == expected_result:
    print("Test passed")
else:
    print("Test failed") 
```

```py
Test passed

```

这个测试的好处是我们可以反复运行它，从而确保至少 4 的平方根被正确计算。但仍然有一些问题：

1.  我们需要一个测试的单行代码

1.  我们并不关心舍入误差

1.  我们只检查单个输入（和单个结果）

让我们逐一解决这些问题。首先，让我们使测试更加紧凑。几乎所有的编程语言都有一种方法来自动检查条件是否成立，如果不成立则停止执行。这被称为 *断言*，它在测试中非常有用。

在 Python 中，`assert` 语句接受一个条件，如果条件为真，则不发生任何操作。（如果一切按预期进行，你就不应该被打扰。）但是，如果条件评估为假，则 `assert` 会引发异常，表明测试刚刚失败。

在我们的例子中，我们可以使用 `assert` 来轻松检查 `my_sqrt()` 是否产生如上所述的预期结果：

```py
assert my_sqrt(4) == 2 
```

当你执行这一行代码时，没有任何操作：我们只是展示了（或断言）我们的实现确实产生了 $\sqrt{4} = 2$。

但是，记住，浮点数计算可能会引入舍入误差。因此，我们不能简单地比较两个浮点数的相等性；相反，我们需要确保它们之间的绝对差值保持在某个特定的阈值以下，通常表示为 $\epsilon$ 或 `epsilon`。这就是我们如何做到这一点：

```py
EPSILON = 1e-8 
```

```py
assert abs(my_sqrt(4) - 2) < EPSILON 
```

我们还可以为此引入一个特殊函数，并现在对具体值进行更多测试：

```py
def assertEquals(x, y, epsilon=1e-8):
    assert abs(x - y) < epsilon 
```

```py
assertEquals(my_sqrt(4), 2)
assertEquals(my_sqrt(9), 3)
assertEquals(my_sqrt(100), 10) 
```

看起来是有效的，对吧？如果我们知道计算的预期结果，我们可以反复使用这样的断言来确保我们的程序正确工作。

（提示：真正的 Python 程序员会使用函数 `math.isclose()` 来代替。）

## 生成测试

记住，性质 $\sqrt{x} \times \sqrt{x} = x$ 在普遍情况下是成立的？我们也可以用几个值显式地测试这一点：

```py
assertEquals(my_sqrt(2) * my_sqrt(2), 2)
assertEquals(my_sqrt(3) * my_sqrt(3), 3)
assertEquals(my_sqrt(42.11) * my_sqrt(42.11), 42.11) 
```

仍然看起来是有效的，对吧？最重要的是，$\sqrt{x} \times \sqrt{x} = x$ 是我们可以很容易地测试成千上万个值的东西：

```py
for n in range(1, 1000):
    assertEquals(my_sqrt(n) * my_sqrt(n), n) 
```

使用 100 个值测试 `my_sqrt()` 需要多少时间？让我们看看。

我们使用自己的 `Timer` 模块来测量经过的时间。为了能够使用 `Timer`，我们首先导入我们的实用模块，这允许我们导入其他笔记本。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from Timer import Timer 
```

```py
with Timer() as t:
    for n in range(1, 10000):
        assertEquals(my_sqrt(n) * my_sqrt(n), n)
print(t.elapsed_time()) 
```

```py
0.013280000013764948

```

10,000 个值大约需要百分之一秒，所以 `my_sqrt()` 的单次执行需要 1/1000000 秒，或者说大约 1 微秒。

让我们用随机选择的 10,000 个值重复这个过程。Python 的 `random.random()` 函数返回一个介于 0.0 和 1.0 之间的随机值：

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
with Timer() as t:
    for i in range(10000):
        x = 1 + random.random() * 1000000
        assertEquals(my_sqrt(x) * my_sqrt(x), x)
print(t.elapsed_time()) 
```

```py
0.015275916957762092

```

在一秒钟内，我们已经测试了 10,000 个随机值，每次计算平方根都是正确的。我们可以对`my_sqrt()`的每一次更改重复这个测试，每次都增强我们对`my_sqrt()`按预期工作的信心。不过，请注意，虽然随机函数在产生随机值时是*无偏的*，但它不太可能生成会剧烈改变程序行为的特殊值。我们将在下面进一步讨论这个问题。

## 运行时验证

我们不仅可以为`my_sqrt()`编写和运行测试，还可以将检查直接*集成到实现中*。这样，*每次*调用`my_sqrt()`都将自动进行检查。

这样的*自动运行时检查*非常容易实现：

```py
def my_sqrt_checked(x):
    root = my_sqrt(x)
    assertEquals(root * root, x)
    return root 
```

现在，每次我们用`my_sqrt_checked()`计算根时$\dots$

```py
my_sqrt_checked(2.0) 
```

```py
1.414213562373095

```

...我们已经知道结果是正确的，并且对于每一次新的成功计算也将如此。

自动运行时检查，如上所述，假设了两件事：

+   必须能够*制定*这样的运行时检查。总是应该有具体的值来检查，但以抽象的方式制定所需属性可能非常复杂。在实践中，你需要决定哪些属性最为关键，并为它们设计适当的检查。此外，运行时检查可能不仅取决于局部属性，还取决于程序状态的多个属性，所有这些属性都必须被识别。

+   必须能够*承担*这样的运行时检查。在`my_sqrt()`的情况下，检查并不昂贵；但如果我们必须检查，比如说，在简单操作之后的大型数据结构，检查的成本可能会很快变得难以承受。在实践中，运行时检查通常在生产过程中被禁用，以效率换取可靠性。另一方面，一套全面的运行时检查是发现错误并快速调试它们的极好方式；你需要决定在生产过程中你仍然需要多少这样的功能。

### 问答

运行时检查能保证总是会有正确的结果吗？

运行时检查的一个重要限制是，它们只能确保*如果存在结果*需要检查时的正确性——也就是说，它们不能保证总是会有一个结果。与*符号验证技术*和程序证明相比，这是一个重要的限制，后者也可以保证存在结果——尽管需要付出更高的（通常是手动）努力。

## 系统输入与函数输入

在这一点上，我们可以将`my_sqrt()`提供给其他程序员，他们可以将它嵌入到他们的代码中。在某个时候，它将不得不处理来自*第三方*的输入，即程序员无法控制的输入。

让我们通过假设一个*程序* `sqrt_program()` 来模拟这个*系统输入*，其输入是由第三方控制的字符串：

```py
def sqrt_program(arg: str) -> None:
    x = int(arg)
    print('The root of', x, 'is', my_sqrt(x)) 
```

我们假设`sqrt_program`是一个程序，它从命令行接受系统输入，如下所示：

```py
$  sqrt_program  4
2 
```

我们可以很容易地用一些系统输入调用`sqrt_program()`：

```py
sqrt_program("4") 
```

```py
The root of 4 is 2.0

```

问题是什么？问题是我们没有检查外部输入的有效性。尝试调用`sqrt_program(-1)`，例如。会发生什么？

事实上，如果你用一个负数调用`my_sqrt()`，它会进入一个无限循环。由于技术原因，我们在这个章节中不能有无限循环（除非我们想让代码永远运行）；所以我们使用一个特殊的`with ExpectTimeOut(1)`结构在一秒后中断执行。

```py
from ExpectError import ExpectTimeout 
```

```py
with ExpectTimeout(1):
    sqrt_program("-1") 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/1288144681.py", line 2, in <module>
    sqrt_program("-1")
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/449782637.py", line 3, in sqrt_program
    print('The root of', x, 'is', my_sqrt(x))
                                  ^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/2661069967.py", line 5, in my_sqrt
    while approx != guess:
          ^^^^^^^^^^^^^^^
  File "Timeout.ipynb", line 43, in timeout_handler
    raise TimeoutError()
TimeoutError (expected)

```

上述信息是一个*错误信息*，表示出现了问题。它列出了在错误发生时活跃的函数和行号的*调用栈*。最底部的行是最后执行的行；上面的行代表函数调用——在我们的例子中，直到`my_sqrt(x)`。

我们不希望我们的代码因为异常而终止。因此，在接收外部输入时，我们必须确保它得到了适当的验证。例如，我们可以这样写：

```py
def sqrt_program(arg: str) -> None:
    x = int(arg)
    if x < 0:
        print("Illegal Input")
    else:
        print('The root of', x, 'is', my_sqrt(x)) 
```

然后我们可以确信`my_sqrt()`是按照其规范调用的。

```py
sqrt_program("-1") 
```

```py
Illegal Input

```

但等等！如果`sqrt_program()`没有用数字调用会发生什么？

### 习题

`sqrt_program('xyzzy')`的结果是什么？

让我们试试看！当我们尝试转换一个非数字字符串时，这也会导致运行时错误：

```py
from ExpectError import ExpectError 
```

```py
with ExpectError():
    sqrt_program("xyzzy") 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/1336991207.py", line 2, in <module>
    sqrt_program("xyzzy")
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/3211514011.py", line 2, in sqrt_program
    x = int(arg)
        ^^^^^^^^
ValueError: invalid literal for int() with base 10: 'xyzzy' (expected)

```

这里有一个版本，它也检查了不良输入：

```py
def sqrt_program(arg: str) -> None:
    try:
        x = float(arg)
    except ValueError:
        print("Illegal Input")
    else:
        if x < 0:
            print("Illegal Number")
        else:
            print('The root of', x, 'is', my_sqrt(x)) 
```

```py
sqrt_program("4") 
```

```py
The root of 4.0 is 2.0

```

```py
sqrt_program("-1") 
```

```py
Illegal Number

```

```py
sqrt_program("xyzzy") 
```

```py
Illegal Input

```

我们现在已经看到，在系统层面，程序必须能够优雅地处理任何类型的输入，而不会进入不受控制的状态。这当然给程序员带来了负担，他们必须努力使他们的程序在各种情况下都健壮。然而，当生成软件测试时，这种负担却变成了*好处*：如果一个程序可以处理任何类型的输入（可能带有定义良好的错误信息），我们也可以*发送任何类型的输入*。但是，在用生成的值调用函数时，我们必须*知道*其精确的先决条件。

## 测试的局限性

尽管我们在测试中尽了最大努力，但请记住，你总是在检查一个*有限*的输入集的功能。因此，可能始终存在*未测试*的输入，对于这些输入，函数可能仍然会失败。

在`my_sqrt()`的情况下，例如，计算$\sqrt{0}$会导致除以零：

```py
with ExpectError():
    root = my_sqrt(0) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/820411145.py", line 2, in <module>
    root = my_sqrt(0)
           ^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/2661069967.py", line 7, in my_sqrt
    guess = (approx + x / approx) / 2
                      ~~^~~~~~~~
ZeroDivisionError: float division by zero (expected)

```

在我们迄今为止的测试中，我们没有检查这个条件，这意味着基于$\sqrt{0} = 0$的程序会意外地失败。但即使我们将随机生成器的输入范围设置为 0–1000000 而不是 1–1000000，随机产生零值的概率仍然是一百万分之一。如果一个函数对少数几个个别值的行为有根本性的不同，普通的随机测试很少有机会产生这些值。

当然，我们可以相应地修复函数，记录`x`接受的值，并处理特殊情况`x = 0`：

```py
def my_sqrt_fixed(x):
    assert 0 <= x
    if x == 0:
        return 0
    return my_sqrt(x) 
```

这样，我们现在可以正确地计算$\sqrt{0} = 0$：

```py
assert my_sqrt_fixed(0) == 0 
```

非法值现在会导致异常：

```py
with ExpectError():
    root = my_sqrt_fixed(-1) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/305965227.py", line 2, in <module>
    root = my_sqrt_fixed(-1)
           ^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_1231/3001478627.py", line 2, in my_sqrt_fixed
    assert 0 <= x
           ^^^^^^
AssertionError (expected)

```

然而，我们必须记住，尽管广泛的测试可能会让我们对程序的正确性有很高的信心，但它并不能保证所有未来的执行都将正确。即使是运行时验证，它检查每一个结果，也只能保证如果它产生了结果，那么结果将是正确的；但无法保证未来的执行不会导致失败的检查。当我写这篇文章的时候，我*相信*`my_sqrt_fixed(x)`是所有有限数字$x$的$\sqrt{x}$的正确实现，但我不能确定。

使用牛顿-拉夫森方法，我们仍然有很好的机会实际上*证明*实现是正确的：实现简单，数学理解得很好。遗憾的是，这种情况只适用于少数领域。如果我们不想进行完整的正确性证明，我们在测试中最好的机会是

1.  在几个精心选择的输入上测试程序；并且

1.  详尽且自动地检查结果。

这就是我们在这门课程的剩余部分要做的：设计帮助我们彻底测试程序的技术，以及帮助我们检查其状态是否正确的技术。享受吧！

## 经验教训

+   测试的目标是执行一个程序，以便我们发现错误。

+   测试执行、测试生成和检查测试结果可以自动化。

+   测试是不完整的；它不能提供 100%的保证代码没有错误。

## 下一步

从这里，你可以继续学习如何

+   使用*fuzzing*测试具有随机输入的程序

享受阅读！

## 背景

关于软件测试与分析有许多研究。

+   一本全新的现代、全面和在线的测试教科书是["有效的软件测试：开发者指南"](https://www.effective-software-testing.com) [[Maurício Aniche，2022](https://www.effective-software-testing.com)]。强烈推荐！

+   对于这本书，我们也很乐意推荐“软件测试与分析”[[Pezzè *et al*，2008](http://ix.cs.uoregon.edu/~michal/book/)]作为该领域的入门书籍；其强大的技术重点非常适合我们的方法。

+   其他一些重要的必读之作，包括心理学和组织，对软件测试有全面的方法，包括“软件测试的艺术”[[Myers *et al*，2004](https://dl.acm.org/citation.cfm?id=983238)]以及“软件测试技术”[[Beizer *et al*，1990](https://dl.acm.org/citation.cfm?id=79060)]。

## 练习

### 练习 1：熟悉笔记本和 Python

你在这本书中的第一个练习是熟悉笔记本和 Python，这样你就可以运行书中的代码示例——并尝试你自己的。以下是一些帮助你开始的任务。

#### 初级水平：在浏览器中运行笔记本

获取代码的最简单方法是在浏览器中运行它们。

1.  从[网页](https://www.fuzzingbook.org/html/Intro_Testing.html)中，查看顶部的菜单。选择`资源` $\rightarrow$ `作为笔记本编辑`。

1.  短暂等待后，这将直接在你的浏览器中打开一个 Jupyter Notebook，其中包含当前章节作为笔记本。

1.  你可以再次滚动浏览材料，但你可以点击任何代码示例来编辑并运行其代码（通过输入`Shift` + `Return`）。你可以随意编辑示例。

1.  注意，代码示例通常依赖于早期代码，所以请确保先运行前面的代码。

1.  你所做的任何更改都不会被保存（除非你将笔记本保存到磁盘）。

关于 Jupyter Notebook 的帮助，从[网页](https://www.fuzzingbook.org/html/Intro_Testing.html)中，查看`帮助`菜单。

#### 高级水平：在你的机器上运行 Python 代码

如果你想要做出更大的更改，但不想使用 Jupyter，这将很有用。

1.  从[网页](https://www.fuzzingbook.org/html/Intro_Testing.html)中，查看顶部的菜单。选择`资源` $\rightarrow$ `下载代码`。

1.  这将下载该章节的 Python 代码作为一个单独的 Python .py 文件，你可以将其保存到你的电脑上。

1.  然后，你可以打开文件，在你的首选 Python 环境中编辑并运行它以重新运行示例。

1.  最重要的是，你可以导入它到你的代码中并重用函数、类和其他资源。

关于 Python 的帮助，从[网页](https://www.fuzzingbook.org/html/Intro_Testing.html)中，查看`帮助`菜单。

#### 高级水平：在你的机器上运行笔记本

如果你想在你的机器上使用 Jupyter，这将很有用。这将允许你运行更复杂的示例，例如带有图形输出的示例。

1.  从[网页](https://www.fuzzingbook.org/html/Intro_Testing.html)中，查看顶部的菜单。选择`资源` $\rightarrow$ `所有笔记本`。

1.  这将下载所有 Jupyter Notebook 作为一个`.ipynb`文件的集合，你可以将其保存到你的电脑上。

1.  然后，你可以在 Jupyter Notebook 或 Jupyter Lab 中打开这些笔记本，编辑并运行它们。要导航到其他笔记本，请打开笔记本`00_ 目录.ipynb`。

1.  你也可以使用`选择` `资源` $\rightarrow$ `下载笔记本`来下载单个笔记本。但是，运行这些笔记本需要你已经下载了其他笔记本。

关于 Jupyter Notebook 的帮助，从[网页](https://www.fuzzingbook.org/html/Intro_Testing.html)中，查看`帮助`菜单。

#### 老板级别：做出贡献！

如果你想要通过补丁或其他材料为本书做出贡献，这将很有用。它还让你可以访问本书的最新版本。

1.  从[网页](https://www.fuzzingbook.org/html/Intro_Testing.html)中，查看顶部的菜单。选择`资源` $\rightarrow$ `GitHub 仓库`。

1.  这将带你去包含本书所有源代码的 GitHub 仓库，包括最新的笔记本。

1.  然后，你可以将此仓库克隆到你的磁盘上，这样你就能获得最新和最好的版本。

1.  你可以在 GitHub 页面上报告问题并提出拉取请求。

1.  使用 `git pull` 更新仓库将使您获得更新。

如果您想贡献代码或文本，请查看作者指南。

### 练习 2：测试 Shellsort

考虑以下 [Shellsort](https://en.wikipedia.org/wiki/Shellsort) 函数的实现，它接受一个元素列表并（可能）对其进行排序。

```py
def shellsort(elems):
    sorted_elems = elems.copy()
    gaps = [701, 301, 132, 57, 23, 10, 4, 1]
    for gap in gaps:
        for i in range(gap, len(sorted_elems)):
            temp = sorted_elems[i]
            j = i
            while j >= gap and sorted_elems[j - gap] > temp:
                sorted_elems[j] = sorted_elems[j - gap]
                j -= gap
            sorted_elems[j] = temp

    return sorted_elems 
```

第一次测试表明 `shellsort()` 实际上可能工作：

```py
shellsort([3, 2, 1]) 
```

```py
[1, 2, 3]

```

实现使用一个 *列表* 作为参数 `elems`（它将其复制到 `sorted_elems`）以及用于固定列表 `gaps`。列表在其他语言中像 *数组* 一样工作：

```py
a = [5, 6, 99, 7]
print("First element:", a[0], "length:", len(a)) 
```

```py
First element: 5 length: 4

```

`range()` 函数返回一个包含元素的可迭代列表。它通常与 `for` 循环一起使用，如上述实现所示。

```py
for x in range(1, 5):
    print(x) 
```

```py
1
2
3
4

```

#### 第一部分：手动测试用例

您现在的任务是彻底测试 `shellsort()` 的各种输入。

首先，设置带有多个手动编写的测试用例的 `assert` 语句。选择测试用例以确保覆盖极端情况。使用 `==` 比较两个列表。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Intro_Testing.ipynb#Exercises)来练习习题并查看解决方案。

#### 第二部分：随机输入

第二次，创建随机列表作为 `shellsort()` 的参数。利用以下辅助谓词来检查结果是否（a）已排序，以及（b）是否是原始列表的排列。

```py
def is_sorted(elems):
    return all(elems[i] <= elems[i + 1] for i in range(len(elems) - 1)) 
```

```py
is_sorted([3, 5, 9]) 
```

```py
True

```

```py
def is_permutation(a, b):
    return len(a) == len(b) and all(a.count(elem) == b.count(elem) for elem in a) 
```

```py
is_permutation([3, 2, 1], [1, 3, 2]) 
```

```py
True

```

从一个随机列表生成器开始，使用 `[]` 作为空列表，并使用 `elems.append(x)` 将元素 `x` 添加到列表 `elems` 中。使用上述辅助函数来评估结果。生成并测试 1,000 个列表。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Intro_Testing.ipynb#Exercises)来练习习题并查看解决方案。

### 练习 3：二次方程求解器

给定一个方程 $ax² + bx + c = 0$，我们希望找到 $x$ 的解，给定 $a$、$b$ 和 $c$ 的值。以下代码应该完成这个任务，使用方程 $$ x = \frac{-b \pm \sqrt{b² - 4ac}}{2a} $$

```py
def quadratic_solver(a, b, c):
    q = b * b - 4 * a * c
    solution_1 = (-b + my_sqrt_fixed(q)) / (2 * a)
    solution_2 = (-b - my_sqrt_fixed(q)) / (2 * a)
    return (solution_1, solution_2) 
```

```py
quadratic_solver(3, 4, 1) 
```

```py
(-0.3333333333333333, -1.0)

```

上述实现是不完整的。您可以触发

1.  一个除以零的错误；并且

1.  违反 `my_sqrt_fixed()` 的先验条件。

如何做到这一点，以及如何防止这种情况发生？

#### 第一部分：寻找触发错误的输入

对于上述两种情况中的每一种，确定 `a`、`b`、`c` 的值以触发错误。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Intro_Testing.ipynb#Exercises)来练习习题并查看解决方案。

#### 第二部分：修复问题

适当地扩展代码以处理这些情况。对于不存在的值返回 `None`。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Intro_Testing.ipynb#Exercises)来练习习题并查看解决方案。

#### 第三部分：其他事项

随机输入下发现这些条件的机会有多大？假设每秒可以进行十亿次测试，平均需要等待多长时间才能触发一个错误？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Intro_Testing.ipynb#Exercises)进行练习并查看解决方案。

### 练习 4：无限与更远

当我们说`my_sqrt_fixed(x)`对所有*有限*数字$x$都有效时：如果你将$x$设置为$\infty$（无穷大），会发生什么？试一试！

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Intro_Testing.ipynb#Exercises)进行练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。内容的一部分源代码，以及用于格式化和显示该内容的源代码受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。最后更改日期：2023-11-11 18:18:06+01:00。引用 [版权信息](https://cispa.de/en/impressum)

## 如何引用此作品

安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒："[软件测试入门](https://www.fuzzingbook.org/html/Intro_Testing.html)"。在安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒的《[模糊测试书](https://www.fuzzingbook.org/)[`www.fuzzingbook.org/html/Intro_Testing.html`](https://www.fuzzingbook.org/html/Intro_Testing.html)]中。检索日期：2023-11-11 18:18:06+01:00。

```py
@incollection{fuzzingbook2023:Intro_Testing,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Introduction to Software Testing},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/Intro_Testing.html}},
    note = {Retrieved 2023-11-11 18:18:06+01:00},
    url = {https://www.fuzzingbook.org/html/Intro_Testing.html},
    urldate = {2023-11-11 18:18:06+01:00}
}

```
