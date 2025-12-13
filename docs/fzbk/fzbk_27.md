# 符号模糊测试

> 原文：[`www.fuzzingbook.org/html/SymbolicFuzzer.html`](http://www.fuzzingbook.org/html/SymbolicFuzzer.html)

模糊测试的传统方法中存在的问题之一是它们未能锻炼系统可能具有的所有可能行为，尤其是在输入空间很大时。很多时候，特定执行分支的执行可能只发生在非常特定的输入上，这可能只占输入空间的一小部分。传统的模糊测试方法依赖于偶然来产生它们需要的输入。然而，当探索的空间很大时，依赖于随机生成我们想要的值是一个坏主意。例如，一个接受字符串的函数，即使只考虑前 10 个字符，也有$2^{80}$种可能的输入。如果寻找特定的字符串，即使在超级计算机上随机生成值也需要几千年。

在关于 concolic 测试的章节中，我们看到了*符号跟踪*如何提供一种出路。我们看到了如何使用 Python 解释器通过直接信息流实现符号跟踪。然而，这种方法有两个问题。

+   第一，符号跟踪依赖于样本输入的存在。如果没有样本输入怎么办？

+   其次，如果程序有基于控制流等间接信息流，直接信息流可能不可靠。

在这两种情况下，*静态代码分析*可以弥合差距。然而，这引发了一个问题：我们能否通过静态检查程序来确定其完整的行为，并检查它在某些（未知）输入下是否表现异常，或者导致意外的输出？

*符号执行*是我们可以在不执行程序的情况下推理程序行为的一种方式。程序是一种计算，可以被视为一个方程组，从给定的输入中获得输出值。以符号方式执行程序——即数学上解决这些方程——以及任何指定的目标，如覆盖特定分支或获得特定输出，将为我们提供完成此任务的输入。

在本章中，我们研究如何实现符号执行，以及如何将其用于获取模糊测试中的有趣值。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('AJfRBF8NEWU') 
```

**先决条件**

+   您应该了解如何在 Python 中使用[类型注解](https://docs.python.org/3/library/typing.html)。

+   熟悉[SMT 求解器](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories)，特别是[Z3](https://github.com/Z3Prover/z3)是有用的。

+   您应该已经阅读了关于覆盖的章节。

+   熟悉关于 concolic fuzzing 的章节会有所帮助。

## 概述

要使用本章节提供的代码，请编写

```py
>>> from fuzzingbook.SymbolicFuzzer import <identifier> 
```

然后利用以下功能。

本章提供了一个符号模糊引擎 `SymbolicFuzzer` 的实现。该模糊器使用符号执行来穷举探索程序中的路径到有限的深度，并生成将到达这些路径的输入。

例如，考虑函数 `gcd()`，它计算 `a` 和 `b` 的最大公约数：

```py
def gcd(a: int, b: int) -> int:
    if a < b:
        c: int = a
        a = b
        b = c

    while b != 0:
        c: int = a
        a = b
        b = c % b

    return a 
```

要探索 `gcd()`，可以使用模糊器如下，生成覆盖 `gcd()` 中不同路径（包括多次循环迭代）的参数值：

```py
>>> gcd_fuzzer = SymbolicFuzzer(gcd, max_tries=10, max_iter=10, max_depth=10)
>>> for i in range(10):
>>>     args = gcd_fuzzer.fuzz()
>>>     print(args)
{'a': 8, 'b': 3}
{'a': 1, 'b': 2}
{'a': 2, 'b': 5}
{'a': 7, 'b': 6}
{'a': 9, 'b': 10}
{'a': 4, 'b': 4}
{'a': 10, 'b': 9}
{'a': 2, 'b': 10}
{'a': 14, 'b': 7}
{'a': 3, 'b': 2} 
```

注意，`fuzz()` 返回的变量值是 Z3 的 *符号* 值；要将它们转换为 Python 数字，请使用它们的 `as_long()` 方法：

```py
>>> for i in range(10):
>>>     args = gcd_fuzzer.fuzz()
>>>     a = args['a'].as_long()
>>>     b = args['b'].as_long()
>>>     d = gcd(a, b)
>>>     print(f"gcd({a}, {b}) = {d}")
gcd(0, 8) = 8
gcd(-1, 10) = -1
gcd(13, 2) = 1
gcd(0, 10) = 10
gcd(6, 7) = 1
gcd(14, 2) = 2
gcd(-1, 11) = -1
gcd(15, 0) = 15
gcd(0, -1) = -1
gcd(-3, -2) = -1 
```

符号模糊器受到一些约束。首先，它要求要模糊化的函数具有正确的类型注解，包括所有局部变量。其次，它通过展开循环来解决循环，但仅限于固定的数量。

对于没有循环和变量重新分配的程序，`SimpleSymbolicFuzzer` 是一个更快但功能更有限的替代方案。

`<svg width="300pt" height="402pt" viewBox="0.00 0.00 300.00 401.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 397.75)"><g id="node1" class="node"><title>SymbolicFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class SymbolicFuzzer:`

带有重新分配和循环展开的符号模糊测试"><text text-anchor="start" x="28.62" y="-81.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">SymbolicFuzzer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="SymbolicFuzzer"><g id="a_node1_1"><a xlink:href="#" xlink:title="extract_constraints(self, path)"><text text-anchor="start" x="8.38" y="-59.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">extract_constraints()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="get_all_paths(self, fenter)"><text text-anchor="start" x="8.38" y="-47" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">get_all_paths()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="get_next_path(self)"><text text-anchor="start" x="8.38" y="-34.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">get_next_path()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="options(self, kwargs)"><text text-anchor="start" x="8.38" y="-21.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">options()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="solve_path_constraint(self, path)"><text text-anchor="start" x="8.38" y="-8.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">solve_path_constraint()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>SimpleSymbolicFuzzer</title> <g id="a_node2"><a xlink:href="#" xlink:title="class SimpleSymbolicFuzzer:

简单符号模糊器"><text text-anchor="start" x="8" y="-254.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">SimpleSymbolicFuzzer</text> <g id="a_node2_6"><a xlink:href="#" xlink:title="SimpleSymbolicFuzzer"><g id="a_node2_7"><a xlink:href="#" xlink:title="__init__(self, fn, **kwargs):">

构造函数。

`fn` 是要模糊测试的函数。

可能的关键字参数：

* `max_depth` - 应尝试的深度

跟踪执行（默认 100）

* `max_tries` - 尝试的最大次数

在放弃之前，我们将尝试生成一个值（默认 100）

* `max_iter` - 我们将尝试的迭代次数（默认 100）。"><text text-anchor="start" x="8.38" y="-232.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node2_8"><a xlink:href="#" xlink:title="fuzz(self):">

为每条路径生成一个解决方案。

返回变量名到（符号）Z3 值的映射。"><text text-anchor="start" x="8.38" y="-220" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node2_9"><a xlink:href="#" xlink:title="extract_constraints(self, path)"><text text-anchor="start" x="8.38" y="-207.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">extract_constraints()</text></a></g> <g id="a_node2_10"><a xlink:href="#" xlink:title="get_all_paths(self, fenter, depth=0)"><text text-anchor="start" x="8.38" y="-194.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">get_all_paths()</text></a></g> <g id="a_node2_11"><a xlink:href="#" xlink:title="get_next_path(self)"><text text-anchor="start" x="8.38" y="-181.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">get_next_path()</text></a></g> <g id="a_node2_12"><a xlink:href="#" xlink:title="options(self, kwargs)"><text text-anchor="start" x="8.38" y="-169" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">options()</text></a></g> <g id="a_node2_13"><a xlink:href="#" xlink:title="process(self)"><text text-anchor="start" x="8.38" y="-155.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">process()</text></a></g> <g id="a_node2_14"><a xlink:href="#" xlink:title="solve_path_constraint(self, path)"><text text-anchor="start" x="8.38" y="-143.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">solve_path_constraint()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>SymbolicFuzzer->SimpleSymbolicFuzzer</title></g> <g id="node3" class="node"><title>Fuzzer</title> <g id="a_node3"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊测试器的基类。"><text text-anchor="start" x="56.75" y="-376.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Fuzzer</text> <g id="a_node3_15"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node3_16"><a xlink:href="Fuzzer.html" xlink:title="__init__(self) -> None:

构造函数`__init__()`</text></a></g> <g id="a_node3_17"><a xlink:href="Fuzzer.html" xlink:title="fuzz(self) -> str:

返回模糊输入</a></g> <g id="a_node3_18"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

使用模糊输入运行`runner`，执行`run()`操作</a></g> <g id="a_node3_19"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

使用模糊输入，`trials`次运行`runner`，执行`runs()`操作</a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>SimpleSymbolicFuzzer->Fuzzer</title></g> <g id="node4" class="node"><title>图例</title> <text text-anchor="start" x="172.75" y="-65.38" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="172.75" y="-55.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="178.75" y="-55.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="172.75" y="-45.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="178.75" y="-45.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="172.75" y="-35.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="178.75" y="-35.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="172.75" y="-26.32" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 获取覆盖的路径条件

在关于解析和重新组合输入的章节中，我们看到了为`process_vehicle()`生成输入是多么困难——这是一个接受字符串的简单函数。那里给出的解决方案是依赖于现有的样本输入。然而，这个解决方案是不充分的，因为它假设样本输入的存在。如果手头没有样本输入怎么办呢？

对于一个更简单的例子，让我们考虑以下三角形函数（我们已经在关于符号化模糊测试的章节中见过）。我们能否生成输入来覆盖所有路径？

*注意。* 我们使用类型注解来表示程序的参数类型。《发现动态不变量》这一章将讨论这些类型如何被自动推断。

```py
def check_triangle(a: int, b: int, c: int) -> str:
    if a == b:
        if a == c:
            if b == c:
                return "Equilateral"
            else:
                return "Isosceles"
        else:
            return "Isosceles"
    else:
        if b != c:
            if a == c:
                return "Isosceles"
            else:
                return "Scalene"
        else:
            return "Isosceles" 
```

### 控制流图

这个函数的控制流图可以表示如下：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
from ControlFlow import PyCFG, to_graph, gen_cfg 
```

```py
def show_cfg(fn, **kwargs):
    return to_graph(gen_cfg(inspect.getsource(fn)), **kwargs) 
```

```py
show_cfg(check_triangle) 
```

<svg width="676pt" height="487pt" viewBox="0.00 0.00 675.50 486.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 482.5)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="259.75" y="-450.32" font-family="Times,serif" font-size="14.00">1: enter: check_triangle(a, b, c)</text></g> <g id="node9" class="node"><title>3</title> <text text-anchor="middle" x="259.75" y="-373.32" font-family="Times,serif" font-size="14.00">2: if: a == b</text></g> <g id="edge7" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="353.75" y="-15.82" font-family="Times,serif" font-size="14.00">1: exit: check_triangle(a, b, c)</text></g> <g id="node3" class="node"><title>6</title> <text text-anchor="middle" x="144.75" y="-92.83" font-family="Times,serif" font-size="14.00">5: return 'Equilateral'</text></g> <g id="edge1" class="edge"><title>6->2</title></g> <g id="node4" class="node"><title>7</title> <text text-anchor="middle" x="287.75" y="-92.83" font-family="Times,serif" font-size="14.00">7: return 'Isosceles'</text></g> <g id="edge2" class="edge"><title>7->2</title></g> <g id="node5" class="node"><title>8</title> <text text-anchor="middle" x="59.75" y="-146.82" font-family="Times,serif" font-size="14.00">9: return 'Isosceles'</text></g> <g id="edge3" class="edge"><title>8->2</title></g> <g id="node6" class="node"><title>11</title> <text text-anchor="middle" x="466.75" y="-92.83" font-family="Times,serif" font-size="14.00">13: return 'Isosceles'</text></g> <g id="edge4" class="edge"><title>11->2</title></g> <g id="node7" class="node"><title>12</title> <text text-anchor="middle" x="607.75" y="-92.83" font-family="Times,serif" font-size="14.00">15: return 'Scalene'</text></g> <g id="edge5" class="edge"><title>12->2</title></g> <g id="node8" class="node"><title>13</title> <text text-anchor="middle" x="369.75" y="-146.82" font-family="Times,serif" font-size="14.00">17: return 'Isosceles'</text></g> <g id="edge6" class="edge"><title>13->2</title></g> <g id="node10" class="node"><title>4</title> <text text-anchor="middle" x="175.75" y="-287.07" font-family="Times,serif" font-size="14.00">3: if: a == c</text></g> <g id="edge8" class="edge"><title>3->4</title> <text text-anchor="middle" x="226.9" y="-330.2" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node12" class="node"><title>9</title> <text text-anchor="middle" x="368.75" y="-287.07" font-family="Times,serif" font-size="14.00">11: if: b != c</text></g> <g id="edge13" class="edge"><title>3->9</title> <text text-anchor="middle" x="324.53" y="-330.2" font-family="Times,serif" font-size="14.00">F</text></g> <g id="edge12" class="edge"><title>4->8</title> <text text-anchor="middle" x="127.31" y="-243.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node11" class="node"><title>5</title> <text text-anchor="middle" x="175.75" y="-200.82" font-family="Times,serif" font-size="14.00">4: if: b == c</text></g> <g id="edge9" class="edge"><title>4->5</title> <text text-anchor="middle" x="179.88" y="-243.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge10" class="edge"><title>5->6</title> <text text-anchor="middle" x="168.96" y="-146.82" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge11" class="edge"><title>5->7</title> <text text-anchor="middle" x="252.08" y="-146.82" font-family="Times,serif" font-size="14.00">F</text></g> <g id="edge17" class="edge"><title>9->13</title> <text text-anchor="middle" x="372.85" y="-243.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node13" class="node"><title>10</title> <text text-anchor="middle" x="476.75" y="-200.82" font-family="Times,serif" font-size="14.00">12: if: a == c</text></g> <g id="edge14" class="edge"><title>9->10</title> <text text-anchor="middle" x="433.34" y="-243.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge15" class="edge"><title>10->11</title> <text text-anchor="middle" x="477.36" y="-146.82" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge16" class="edge"><title>10->12</title> <text text-anchor="middle" x="565.39" y="-146.82" font-family="Times,serif" font-size="14.00">F</text></g></g></svg>

程序可能执行的路径可以表示如下，数字表示执行的具体行号。

```py
paths = {
    '<path 1>': ([1, 2, 3, 4, 5], 'Equilateral'),
    '<path 2>': ([1, 2, 3, 4, 7], 'Isosceles'),
    '<path 3>': ([1, 2, 3, 9], 'Isosceles'),
    '<path 4>': ([1, 2, 11, 12, 13], 'Isosceles'),
    '<path 5>': ([1, 2, 11, 12, 15], 'Scalene'),
    '<path 6>': ([1, 2, 11, 17], 'Isosceles'),
} 
```

考虑 `<path 1>`。要追踪此路径，我们需要按顺序执行以下语句。

```py
1: check_triangle(a, b, c)
2: if (a == b) -> True
3: if (a == c) -> True
4: if (b == c) -> True
5: return 'Equilateral' 
```

那就是，任何追踪此路径的执行都必须从满足行号 `2: (a == b)` 评估为 `True`，`3: (a == c)` 评估为 `True`，以及 `4: (b == c)` 评估为 `True` 的 `a`、`b` 和 `c` 的值开始。我们能生成满足这些约束的输入吗？

我们已经在《关于 concolic 模糊测试》这一章中看到，如何使用 Z3 这样的 SMT 求解器来获得解决方案。

```py
import [z3](https://github.com/Z3Prover/z3#readme) 
```

```py
z3_ver = z3.get_version()
print(z3_ver) 
```

```py
(4, 11, 2, 0)

```

```py
assert z3_ver >= (4, 8, 6, 0), "Please check z3 version" 
```

我们需要什么样的符号变量？我们可以从函数的类型注解中获取这些信息。

```py
def get_annotations(fn):
    sig = inspect.signature(fn)
    return ([(i.name, i.annotation)
             for i in sig.parameters.values()], sig.return_annotation) 
```

```py
params, ret = get_annotations(check_triangle)
params, ret 
```

```py
([('a', int), ('b', int), ('c', int)], str)

```

我们创建符号变量来表示每个参数

```py
SYM_VARS = {
    int: (
        z3.Int, z3.IntVal), float: (
            z3.Real, z3.RealVal), str: (
                z3.String, z3.StringVal)} 
```

```py
def get_symbolicparams(fn):
    params, ret = get_annotations(fn)
    return [SYM_VARS[typ]0
            for name, typ in params], SYM_VARS[ret]0 
```

```py
(a, b, c), r = get_symbolicparams(check_triangle)
a, b, c, r 
```

```py
(a, b, c, __return__)

```

我们现在可以要求 *z3* 为我们求解以下方程组。

```py
z3.solve(a == b, a == c, b == c) 
```

```py
[a = 0, b = 0, c = 0]

```

在这里，我们发现了程序中的第一个问题。我们的程序似乎没有检查边长是否大于零。（现实世界中的三角形所有边长都是正数。）暂时假设我们没有这个限制。我们的程序是否正确地遵循了描述的路径？

我们可以使用《关于 concolic 模糊测试》这一章中的 `ArcCoverage` 作为跟踪器来可视化以下信息。

```py
from ConcolicFuzzer import ArcCoverage  # minor dependency 
```

首先，我们恢复跟踪。

```py
with ArcCoverage() as cov:
    assert check_triangle(0, 0, 0) == 'Equilateral'
cov._trace, cov.arcs() 
```

```py
([('check_triangle', 1),
  ('check_triangle', 2),
  ('check_triangle', 3),
  ('check_triangle', 4),
  ('check_triangle', 5),
  ('__exit__', 102),
  ('__exit__', 105)],
 [(1, 2), (2, 3), (3, 4), (4, 5), (5, 102), (102, 105)])

```

我们现在可以确定所采取的路径。

### 已采取路径的 CFG

```py
show_cfg(check_triangle, arcs=cov.arcs()) 
```

<svg width="683pt" height="420pt" viewBox="0.00 0.00 682.50 420.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 416)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="295.75" y="-383.82" font-family="Times,serif" font-size="14.00">1: enter: check_triangle(a, b, c)</text></g> <g id="node9" class="node"><title>3</title> <text text-anchor="middle" x="295.75" y="-307.82" font-family="Times,serif" font-size="14.00">2: if: a == b</text></g> <g id="edge7" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="338.75" y="-15.82" font-family="Times,serif" font-size="14.00">1: exit: check_triangle(a, b, c)</text></g> <g id="node3" class="node"><title>6</title> <text text-anchor="middle" x="151.75" y="-91.83" font-family="Times,serif" font-size="14.00">5: return 'Equilateral'</text></g> <g id="edge1" class="edge"><title>6->2</title></g> <g id="node4" class="node"><title>7</title> <text text-anchor="middle" x="294.75" y="-91.83" font-family="Times,serif" font-size="14.00">7: return 'Isosceles'</text></g> <g id="edge2" class="edge"><title>7->2</title></g> <g id="node5" class="node"><title>8</title> <text text-anchor="middle" x="59.75" y="-163.82" font-family="Times,serif" font-size="14.00">9: return 'Isosceles'</text></g> <g id="edge3" class="edge"><title>8->2</title></g> <g id="node6" class="node"><title>11</title> <text text-anchor="middle" x="473.75" y="-91.83" font-family="Times,serif" font-size="14.00">13: return 'Isosceles'</text></g> <g id="edge4" class="edge"><title>11->2</title></g> <g id="node7" class="node"><title>12</title> <text text-anchor="middle" x="614.75" y="-91.83" font-family="Times,serif" font-size="14.00">15: return 'Scalene'</text></g> <g id="edge5" class="edge"><title>12->2</title></g> <g id="node8" class="node"><title>13</title> <text text-anchor="middle" x="381.75" y="-163.82" font-family="Times,serif" font-size="14.00">17: return 'Isosceles'</text></g> <g id="edge6" class="edge"><title>13->2</title></g> <g id="node10" class="node"><title>4</title> <text text-anchor="middle" x="211.75" y="-235.82" font-family="Times,serif" font-size="14.00">3: if: a == c</text></g> <g id="edge8" class="edge"><title>3->4</title></g> <g id="node12" class="node"><title>9</title> <text text-anchor="middle" x="381.75" y="-235.82" font-family="Times,serif" font-size="14.00">11: if: b != c</text></g> <g id="edge13" class="edge"><title>3->9</title></g> <g id="edge12" class="edge"><title>4->8</title></g> <g id="node11" class="node"><title>5</title> <text text-anchor="middle" x="211.75" y="-163.82" font-family="Times,serif" font-size="14.00">4: if: b == c</text></g> <g id="edge9" class="edge"><title>4->5</title></g> <g id="edge10" class="edge"><title>5->6</title></g> <g id="edge11" class="edge"><title>5->7</title></g> <g id="edge17" class="edge"><title>9->13</title></g> <g id="node13" class="node"><title>10</title> <text text-anchor="middle" x="542.75" y="-163.82" font-family="Times,serif" font-size="14.00">12: if: a == c</text></g> <g id="edge14" class="edge"><title>9->10</title></g> <g id="edge15" class="edge"><title>10->11</title></g> <g id="edge16" class="edge"><title>10->12</title></g></g></svg>

如您所见，所采取的路径是 `<path 1>`。

类似地，为了解决 `<path 2>`，我们只需要在 <line 2> 处反转条件：

```py
z3.solve(a == b, a == c, z3.Not(b == c)) 
```

```py
no solution

```

符号执行表明没有解决方案。稍加思考就会让我们相信这确实是正确的。让我们继续其他路径。《<path 3>`可以通过在 `<line 4>` 处反转条件来获得。

```py
z3.solve(a == b, z3.Not(a == c)) 
```

```py
[b = 1, c = 0, a = 1]

```

```py
with ArcCoverage() as cov:
    assert check_triangle(1, 1, 0) == 'Isosceles'
[i for fn, i in cov._trace if fn == 'check_triangle'] 
```

```py
[1, 2, 3, 9]

```

```py
paths['<path 3>'] 
```

```py
([1, 2, 3, 9], 'Isosceles')

```

那么 `<path 4>` 呢？

```py
z3.solve(z3.Not(a == b), b != c, a == c) 
```

```py
[b = 0, c = 1, a = 1]

```

正如我们之前提到的，我们的程序没有考虑到零或负长度的边。我们可以修改我们的程序来检查零和负输入。然而，我们是否总是需要确保每个函数都要考虑到所有可能的输入？可能 `check_triangle` 并没有直接暴露给用户，而是从一个已经保证输入为正的另一个函数中调用。在《关于动态不变量》这一章中，我们将展示如何发现这样的前置条件和后置条件。

我们可以很容易地在这里添加这样的前置条件。

```py
pre_condition = z3.And(a > 0, b > 0, c > 0) 
```

```py
z3.solve(pre_condition, z3.Not(a == b), b != c, a == c) 
```

```py
[c = 2, b = 1, a = 2]

```

```py
with ArcCoverage() as cov:
    assert check_triangle(1, 2, 1) == 'Isosceles'
[i for fn, i in cov._trace if fn == 'check_triangle'] 
```

```py
[1, 2, 11, 12, 13]

```

```py
paths['<path 4>'] 
```

```py
([1, 2, 11, 12, 13], 'Isosceles')

```

继续到路径 <5>：

```py
z3.solve(pre_condition, z3.Not(a == b), b != c, z3.Not(a == c)) 
```

```py
[a = 1, c = 3, b = 2]

```

确实是一个 *不等边三角形*。

```py
with ArcCoverage() as cov:
    assert check_triangle(3, 1, 2) == 'Scalene' 
```

```py
paths['<path 5>'] 
```

```py
([1, 2, 11, 12, 15], 'Scalene')

```

最后，对于 `<path 6>`，过程是相似的。

```py
z3.solve(pre_condition, z3.Not(a == b), z3.Not(b != c)) 
```

```py
[c = 2, a = 1, b = 2]

```

```py
with ArcCoverage() as cov:
    assert check_triangle(2, 1, 1) == 'Isosceles'
[i for fn, i in cov._trace if fn == 'check_triangle'] 
```

```py
[1, 2, 11, 17]

```

```py
paths['<path 6>'] 
```

```py
([1, 2, 11, 17], 'Isosceles')

```

如果我们想要另一个解决方案呢？我们可以简单地要求求解器再次求解，而不给出相同的值。

```py
seen = [z3.And(a == 2, b == 1, c == 1)] 
```

```py
z3.solve(pre_condition, z3.Not(z3.Or(seen)), z3.Not(a == b), z3.Not(b != c)) 
```

```py
[c = 2, a = 1, b = 2]

```

```py
seen.append(z3.And(a == 1, b == 2, c == 2)) 
```

```py
z3.solve(pre_condition, z3.Not(z3.Or(seen)), z3.Not(a == b), z3.Not(b != c)) 
```

```py
[c = 1, a = 3, b = 1]

```

也就是说，通过简单的符号计算，我们能够轻松地看到（1）一些路径是不可达的，并且（2）一些条件是不充分的——我们需要先决条件。那么，我们得到的总覆盖率如何？

### 可视化覆盖率

可以通过以下方式可视化语句覆盖率。

```py
class VisualizedArcCoverage(ArcCoverage):
    def show_coverage(self, fn):
        src = fn if isinstance(fn, str) else inspect.getsource(fn)
        covered = set([lineno for method, lineno in self._trace])
        for i, s in enumerate(src.split('\n')):
            print('%s  %2d: %s' % ('#' if i + 1 in covered else ' ', i + 1, s)) 
```

我们运行在覆盖率跟踪器下获得的所有输入。

```py
with VisualizedArcCoverage() as cov:
    assert check_triangle(0, 0, 0) == 'Equilateral'
    assert check_triangle(1, 1, 0) == 'Isosceles'
    assert check_triangle(1, 2, 1) == 'Isosceles'
    assert check_triangle(3, 1, 2) == 'Scalene'
    assert check_triangle(2, 1, 1) == 'Isosceles' 
```

```py
cov.show_coverage(check_triangle) 
```

```py
#  1: def check_triangle(a: int, b: int, c: int) -> str:
#  2:     if a == b:
#  3:         if a == c:
#  4:             if b == c:
#  5:                 return "Equilateral"
   6:             else:
   7:                 return "Isosceles"
   8:         else:
#  9:             return "Isosceles"
  10:     else:
# 11:         if b != c:
# 12:             if a == c:
# 13:                 return "Isosceles"
  14:             else:
# 15:                 return "Scalene"
  16:         else:
# 17:             return "Isosceles"
  18: 

```

覆盖率正如预期的那样。生成的值似乎覆盖了所有可覆盖的代码。

我们已经看到了如何对程序中的每条路径进行推理。我们能否将它们组合起来，生成一个表示程序行为的单个表达式？这是我们接下来要讨论的。

### 函数摘要

考虑这个方程来确定绝对值。

```py
def abs_value(x: float) -> float:
    if x < 0:
        v: float = -x
    else:
        v: float = x
    return v 
```

```py
show_cfg(abs_value) 
```

<svg width="264pt" height="365pt" viewBox="0.00 0.00 263.62 365.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 361.25)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="128.74" y="-329.07" font-family="Times,serif" font-size="14.00">1: enter: abs_value(x)</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="128.74" y="-252.07" font-family="Times,serif" font-size="14.00">2: if: x < 0</text></g> <g id="edge2" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="128.74" y="-15.82" font-family="Times,serif" font-size="14.00">1: exit: abs_value(x)</text></g> <g id="node3" class="node"><title>6</title> <text text-anchor="middle" x="128.74" y="-92.83" font-family="Times,serif" font-size="14.00">6: return v</text></g> <g id="edge1" class="edge"><title>6->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="60.74" y="-165.82" font-family="Times,serif" font-size="14.00">3: v: float = -x</text></g> <g id="edge3" class="edge"><title>3->4</title> <text text-anchor="middle" x="102.94" y="-208.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="197.74" y="-165.82" font-family="Times,serif" font-size="14.00">5: v: float = x</text></g> <g id="edge4" class="edge"><title>3->5</title> <text text-anchor="middle" x="171.12" y="-208.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="edge5" class="edge"><title>4->6</title></g> <g id="edge6" class="edge"><title>5->6</title></g></g></svg>

我们能说关于 `line: 5` 处 `v` 的值有什么吗？让我们追踪一下。首先，我们在 `line: 1` 有变量 `x`。

```py
(x,), r = get_symbolicparams(abs_value) 
```

在 `line: 2`，我们面临可能的路径的分岔。因此，我们产生了两条具有相应约束的路径。

```py
l2_T = x < 0
l2_F = z3.Not(x < 0) 
```

对于 `line: 3`，我们只需要考虑 `If` 路径。然而，我们有一个赋值。因此，我们在这里使用一个新的变量。源代码中指示的类型是 *float*，其等价的 *z3* 类型是 *Real*。

```py
v_0 = z3.Real('v_0')
l3 = z3.And(l2_T, v_0 == -x) 
```

同样，对于 `line: 5`，我们有一个赋值。（我们能否重用之前的变量 `v_0`？）

```py
v_1 = z3.Real('v_1')
l5 = z3.And(l2_F, v_1 == x) 
```

当我们来到 `line: 6` 时，我们看到我们有两个输入流。我们有一个选择。我们可以像之前那样保持每条路径分开。

```py
v = z3.Real('v')
for s in [z3.And(l3, v == v_0), z3.And(l5, v == v_1)]:
    z3.solve(x != 0, s) 
```

```py
[x = -1/2, v_0 = 1/2, v = 1/2]
[v_1 = 1, x = 1, v = 1]

```

或者，我们可以将它们组合起来，在 `line: 6` 产生一个单个谓词。

```py
v = z3.Real('v')
l6 = z3.Or(z3.And(l3, v == v_0), z3.And(l5, v == v_1))
z3.solve(l6) 
```

```py
[v_1 = 1/2, x = -1/4, v_0 = 1/4, v = 1/4]

```

**注意。** 合并两个执行流可能并不简单，尤其是在执行路径被多次遍历时（例如循环和递归）。对那些感兴趣的人来说，可以查阅[推断循环不变式](https://www.st.cs.uni-saarland.de/publications/details/galeotti-hvc-2014/)。

我们可以使这产生任意数量的 `abs()` 的解，如下所示。

```py
s = z3.Solver()
s.add(l6)
for i in range(5):
    if s.check() == z3.sat:
        m = s.model()
        x_val = m[x]
        print(m)
    else:
        print('no solution')
        break
    s.add(z3.Not(x == x_val))
s 
```

```py
[v_1 = 1/2, x = 1/2, v_0 = 0, v = 1/2]
[v_1 = 0, x = 0, v = 0]
[v_1 = 1/4, x = 1/4, v = 1/4]
[v_1 = 1/8, x = 1/8, v = 1/8]
[v_1 = 1/16, x = 1/16, v = 1/16]

```

[x < 0 ∧ v_0 = -x ∧ v = v_0 ∨ ¬(x < 0) ∧ v_1 = x ∧ v = v_1, ¬(1/2 = x), ¬(0 = x), ¬(1/4 = x), ¬(1/8 = x), ¬(1/16 = x)]

求解器并不特别随机。因此，我们需要稍微帮助它一下，以便在负数范围内产生值。

```py
s.add(x < 0)
for i in range(5):
    if s.check() == z3.sat:
        m = s.model()
        x_val = m[x]
        print(m)
    else:
        print('no solution')
        break
    s.add(z3.Not(x == x_val)) 
```

```py
[x = -1/32, v_0 = 1/32, v = 1/32]
[x = -33/32, v_0 = 33/32, v = 33/32]
[x = -65/32, v_0 = 65/32, v = 65/32]
[x = -97/32, v_0 = 97/32, v = 97/32]
[x = -129/32, v_0 = 129/32, v = 129/32]

```

```py
s 
```

[x < 0 ∧ v_0 = -x ∧ v = v_0 ∨ ¬(x < 0) ∧ v_1 = x ∧ v = v_1, ¬(1/2 = x), ¬(0 = x), ¬(1/4 = x), ¬(1/8 = x), ¬(1/16 = x), x < 0, ¬(-1/32 = x), ¬(-33/32 = x), ¬(-65/32 = x), ¬(-97/32 = x), ¬(-129/32 = x)]

注意，在 `line: 6` 产生的单个表达式本质上是对 `abs_value()` 的摘要。

```py
abs_value_summary = l6
abs_value_summary 
```

x < 0 ∧ v_0 = -x ∧ v = v_0 ∨ ¬(x < 0) ∧ v_1 = x ∧ v = v_1

可以使用 *z3* 求解器在可能的情况下简化谓词。

```py
z3.simplify(l6) 
```

¬(0 ≤ x) ∧ v_0 = -1·x ∧ v = v_0 ∨ 0 ≤ x ∧ v_1 = x ∧ v = v_1

可以使用这个摘要而不是追踪到 `abs_value()`，当 `abs_value()` 在其他地方被使用时。然而，这给我们带来了一个问题。同一个函数可能会被多次调用。在这种情况下，使用相同的变量会导致冲突。避免这种情况的一种方法是将一些调用特定的值作为前缀添加到变量中。

**注意：** SMT 2.0 标准允许直接定义函数（在 SMT 术语中称为 *宏*）。例如，`abs-value` 将如下定义：

```py
(define-fun  abs-value  ((x  Int))  Int
  (if  (>  x  0)
  x
  (*  -1  x))) 
```

或者等价地，（特别是如果 `abs-value` 是递归定义的）

```py
(declare-fun  abs-value  (Int)  Int)
(assert  (forall  ((x  Int))
  (=  (abs-value  x)
  (if  (>  x  0)
  x
  (*  -1  x))))) 
```

然后，可以说

```py
(> (abs-value x) (abs-value y))
```

不幸的是，z3py 项目没有在 Python 中公开这个功能。因此，我们必须使用 `prefix_vars()` 诡计。

```py
import [ast](https://docs.python.org/3/library/ast.html) 
```

`prefix_vars()` 方法修改表达式中的变量，使得变量带有给定的前缀。

```py
def prefix_vars(astnode, prefix):
    if isinstance(astnode, ast.BoolOp):
        return ast.BoolOp(astnode.op,
                          [prefix_vars(i, prefix) for i in astnode.values], [])
    elif isinstance(astnode, ast.BinOp):
        return ast.BinOp(
            prefix_vars(astnode.left, prefix), astnode.op,
            prefix_vars(astnode.right, prefix))
    elif isinstance(astnode, ast.UnaryOp):
        return ast.UnaryOp(astnode.op, prefix_vars(astnode.operand, prefix))
    elif isinstance(astnode, ast.Call):
        return ast.Call(prefix_vars(astnode.func, prefix),
                        [prefix_vars(i, prefix) for i in astnode.args],
                        astnode.keywords)
    elif isinstance(astnode, ast.Compare):
        return ast.Compare(
            prefix_vars(astnode.left, prefix), astnode.ops,
            [prefix_vars(i, prefix) for i in astnode.comparators])
    elif isinstance(astnode, ast.Name):
        if astnode.id in {'And', 'Or', 'Not'}:
            return ast.Name('z3.%s' % (astnode.id), astnode.ctx)
        else:
            return ast.Name('%s%s' % (prefix, astnode.id), astnode.ctx)
    elif isinstance(astnode, ast.Return):
        return ast.Return(prefix_vars(astnode.value, env))
    else:
        return astnode 
```

为了应用 `prefix_vars()`，需要 Python 表达式的 *抽象语法树*（AST）。我们通过调用 `ast.parse()` 获得它：

```py
xy_ast = ast.parse('x+y') 
```

我们可以将生成的树可视化如下：

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import rich_output 
```

```py
if rich_output():
    # Normally, this will do
    from [showast](https://pypi.org/project/showast/) import show_ast
else:
    def show_ast(tree):
        ast.dump(tree, indent=4) 
```

```py
show_ast(xy_ast) 
```

<svg xmlns:xlink="http://www.w3.org/1999/xlink" width="296pt" height="260pt" viewBox="0.00 0.00 296.00 260.00"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 256)"><g id="node1" class="node"><title>0</title> <text text-anchor="start" x="118.5" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Expr</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="start" x="114.38" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">BinOp</text></g> <g id="edge1" class="edge"><title>0--1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="start" x="46.5" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge2" class="edge"><title>1--2</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="135" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Add</text></g> <g id="edge5" class="edge"><title>1--5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="start" x="190.5" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge6" class="edge"><title>1--6</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="27" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"x"</text></g> <g id="edge3" class="edge"><title>2--3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="99" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge4" class="edge"><title>2--4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="189" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"y"</text></g> <g id="edge7" class="edge"><title>6--7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="261" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge8" class="edge"><title>6--8</title></g></g></svg>

然而，可视化并没有显示的是，在解析 Python 源代码时，生成的 AST 默认被一个 `Module` 包装：

```py
xy_ast 
```

```py
<ast.Module at 0x116dfa6e0>

```

为了访问表达式（`Expr`），我们需要访问该“模块”的第一个子节点：

```py
xy_ast.body[0] 
```

```py
<ast.Expr at 0x116dfa4a0>

```

实际的表达式位于那个 `Expr` 对象中：

```py
xy_ast.body[0].value 
```

```py
<ast.BinOp at 0x116dfa2c0>

```

因此，为了更容易地操作表达式 AST，我们定义了一个函数 `get_expression()`，它展开它并返回表达式内部的 AST 表示。

```py
def get_expression(src):
    return ast.parse(src).body[0].value 
```

它的使用方法如下：

```py
e = get_expression('x+y')
e 
```

```py
<ast.BinOp at 0x116df8a00>

```

`to_src()` 函数允许我们将表达式 *反解析*。

```py
def to_src(astnode):
    return ast.unparse(astnode).strip() 
```

它的使用方法如下：

```py
to_src(e) 
```

```py
'x + y'

```

我们可以将这两部分结合起来生成一个带前缀的表达式。让我们将所有变量前缀设置为 `x1_`：

```py
abs_value_summary_ast = get_expression(str(abs_value_summary))
print(to_src(prefix_vars(abs_value_summary_ast, 'x1_'))) 
```

```py
z3.Or(z3.And(z3.And(x1_x < 0, x1_v_0 == -x1_x), x1_v == x1_v_0), z3.And(z3.And(z3.Not(x1_x < 0), x1_v_1 == x1_x), x1_v == x1_v_1))

```

#### 获取变量的名称和类型

关于使用的声明呢？鉴于我们拥有所有的方程在 *Z3* 中，我们可以直接检索这些信息。我们定义了 `z3_names_and_types()`，它接受一个 *Z3* 表达式，并提取所需的变量定义。

```py
def z3_names_and_types(z3_ast):
    hm = {}
    children = z3_ast.children()
    if children:
        for c in children:
            hm.update(z3_names_and_types(c))
    else:
        # HACK.. How else to distinguish literals and vars?
        if (str(z3_ast.decl()) != str(z3_ast.sort())):
            hm["%s" % str(z3_ast.decl())] = 'z3.%s' % str(z3_ast.sort())
        else:
            pass
    return hm 
```

```py
abs_value_declarations = z3_names_and_types(abs_value_summary)
abs_value_declarations 
```

```py
{'x': 'z3.Real', 'v_0': 'z3.Real', 'v': 'z3.Real', 'v_1': 'z3.Real'}

```

然而，`z3_names_and_types()` 的局限性在于它需要 *Z3* AST 来操作。因此，我们还定义了 `used_identifiers()`，它可以直接从任何 Python 表达式的字符串表示中提取标识符（包括 *Z3* 约束）。在这里的一个权衡是我们失去了类型信息。但我们将看到如何恢复它。

```py
def used_identifiers(src):
    def names(astnode):
        lst = []
        if isinstance(astnode, ast.BoolOp):
            for i in astnode.values:
                lst.extend(names(i))
        elif isinstance(astnode, ast.BinOp):
            lst.extend(names(astnode.left))
            lst.extend(names(astnode.right))
        elif isinstance(astnode, ast.UnaryOp):
            lst.extend(names(astnode.operand))
        elif isinstance(astnode, ast.Call):
            for i in astnode.args:
                lst.extend(names(i))
        elif isinstance(astnode, ast.Compare):
            lst.extend(names(astnode.left))
            for i in astnode.comparators:
                lst.extend(names(i))
        elif isinstance(astnode, ast.Name):
            lst.append(astnode.id)
        elif isinstance(astnode, ast.Expr):
            lst.extend(names(astnode.value))
        elif isinstance(astnode, (ast.Num, ast.Str, ast.Tuple, ast.NameConstant)):
            pass
        elif isinstance(astnode, ast.Assign):
            for t in astnode.targets:
                lst.extend(names(t))
            lst.extend(names(astnode.value))
        elif isinstance(astnode, ast.Module):
            for b in astnode.body:
                lst.extend(names(b))
        else:
            raise Exception(str(astnode))
        return list(set(lst))
    return names(ast.parse(src)) 
```

```py
used_identifiers(str(abs_value_summary)) 
```

```py
['v_0', 'x', 'v', 'v_1']

```

我们现在可以注册函数摘要 `abs_value` 以供以后使用。

```py
function_summaries = {}
function_summaries['abs_value'] = {
    'predicate': str(abs_value_summary),
    'vars': abs_value_declarations} 
```

如我们之前提到的，我们不希望依赖于 *Z3* 来提取类型信息。更好的替代方案是让用户指定类型信息作为注释，并从程序中提取这些信息。我们将在下一节中看到如何实现这一点。

首先，我们将 *Python 类型到 Z3 类型* 映射转换为它的字符串等效形式。

```py
SYM_VARS_STR = {
    k.__name__: ("z3.%s" % v1.__name__, "z3.%s" % v2.__name__)
    for k, (v1, v2) in SYM_VARS.items()
}
SYM_VARS_STR 
```

```py
{'int': ('z3.Int', 'z3.IntVal'),
 'float': ('z3.Real', 'z3.RealVal'),
 'str': ('z3.String', 'z3.StringVal')}

```

我们还定义了一个方便的方法 `translate_to_z3_name()`，用于访问符号变量的 *Z3* 类型。

```py
def translate_to_z3_name(v):
    return SYM_VARS_STR[v][0] 
```

我们现在定义了 `declarations()` 方法，该方法提取 Python *语句* 中使用的变量。其思路是寻找包含注释类型信息的扩展赋值。这些信息被收集并返回。

如果有 `call` 节点，它们代表函数调用。这些函数调用中使用的变量从相应的函数摘要中恢复。

```py
def declarations(astnode, hm=None):
    if hm is None:
        hm = {}
    if isinstance(astnode, ast.Module):
        for b in astnode.body:
            declarations(b, hm)
    elif isinstance(astnode, ast.FunctionDef):
        # hm[astnode.name + '__return__'] = \
        # translate_to_z3_name(astnode.returns.id)
        for a in astnode.args.args:
            hm[a.arg] = translate_to_z3_name(a.annotation.id)
        for b in astnode.body:
            declarations(b, hm)
    elif isinstance(astnode, ast.Call):
        # get declarations from the function summary.
        n = astnode.function
        assert isinstance(n, ast.Name)  # for now.
        name = n.id
        hm.update(dict(function_summaries[name]['vars']))
    elif isinstance(astnode, ast.AnnAssign):
        assert isinstance(astnode.target, ast.Name)
        hm[astnode.target.id] = translate_to_z3_name(astnode.annotation.id)
    elif isinstance(astnode, ast.Assign):
        # verify it is already defined
        for t in astnode.targets:
            assert isinstance(t, ast.Name)
            assert t.id in hm
    elif isinstance(astnode, ast.AugAssign):
        assert isinstance(astnode.target, ast.Name)
        assert astnode.target.id in hm
    elif isinstance(astnode, (ast.If, ast.For, ast.While)):
        for b in astnode.body:
            declarations(b, hm)
        for b in astnode.orelse:
            declarations(b, hm)
    elif isinstance(astnode, ast.Return):
        pass
    else:
        raise Exception(str(astnode))
    return hm 
```

因此，我们现在可以提取表达式中所使用的变量。

```py
declarations(ast.parse('s: int = 3\np: float = 4.0\ns += 1')) 
```

```py
{'s': 'z3.Int', 'p': 'z3.Real'}

```

我们将 `declarations()` 包裹在直接操作函数对象的 `used_vars()` 方法中。

```py
def used_vars(fn):
    return declarations(ast.parse(inspect.getsource(fn))) 
```

这里是如何使用它的：

```py
used_vars(check_triangle) 
```

```py
{'a': 'z3.Int', 'b': 'z3.Int', 'c': 'z3.Int'}

```

```py
used_vars(abs_value) 
```

```py
{'x': 'z3.Real', 'v': 'z3.Real'}

```

给定提取的变量及其 *Z3* 类型，我们需要一种方法在需要时重新实例化它们。我们定义了 `define_symbolic_vars()` 函数，该函数将这些描述转换为可以直接 `exec()` 的形式。

```py
def define_symbolic_vars(fn_vars, prefix):
    sym_var_dec = ', '.join([prefix + n for n in fn_vars])
    sym_var_def = ', '.join(["%s('%s%s')" % (t, prefix, n)
                             for n, t in fn_vars.items()])
    return "%s = %s" % (sym_var_dec, sym_var_def) 
```

这里是如何使用它的：

```py
define_symbolic_vars(abs_value_declarations, '') 
```

```py
"x, v_0, v, v_1 = z3.Real('x'), z3.Real('v_0'), z3.Real('v'), z3.Real('v_1')"

```

接下来，我们定义 `gen_fn_summary()`，它使用 *Z3* 返回可实例化的函数摘要。

```py
def gen_fn_summary(prefix, fn):
    summary = function_summaries[fn.__name__]['predicate']
    fn_vars = function_summaries[fn.__name__]['vars']
    decl = define_symbolic_vars(fn_vars, prefix)
    summary_ast = get_expression(summary)
    return decl, to_src(prefix_vars(summary_ast, prefix)) 
```

这里是如何使用它的：

```py
gen_fn_summary('a_', abs_value) 
```

```py
("a_x, a_v_0, a_v, a_v_1 = z3.Real('a_x'), z3.Real('a_v_0'), z3.Real('a_v'), z3.Real('a_v_1')",
 'z3.Or(z3.And(z3.And(a_x < 0, a_v_0 == -a_x), a_v == a_v_0), z3.And(z3.And(z3.Not(a_x < 0), a_v_1 == a_x), a_v == a_v_1))')

```

```py
gen_fn_summary('b_', abs_value) 
```

```py
("b_x, b_v_0, b_v, b_v_1 = z3.Real('b_x'), z3.Real('b_v_0'), z3.Real('b_v'), z3.Real('b_v_1')",
 'z3.Or(z3.And(z3.And(b_x < 0, b_v_0 == -b_x), b_v == b_v_0), z3.And(z3.And(z3.Not(b_x < 0), b_v_1 == b_x), b_v == b_v_1))')

```

我们如何使用我们的函数摘要？这里有一个使用 `abs_value()` 的函数 `abs_max()`。

```py
def abs_max(a: float, b: float):
    a1: float = abs_value(a)
    b1: float = abs_value(b)
    if a1 > b1:
        c: float = a1
    else:
        c: float = b1
    return c 
```

要符号化跟踪此函数，我们首先定义两个变量 `a` 和 `b`。

```py
a = z3.Real('a')
b = z3.Real('b') 
```

`line: 2` 行包含对 `a1` 的定义，我们将其定义为符号变量。

```py
a1 = z3.Real('a1') 
```

我们还需要调用 `abs_value()`，这可以通过以下方式完成。由于这是对 `abs_value()` 的第一次调用，我们使用 `abs1` 作为前缀。

```py
d, v = gen_fn_summary('abs1_', abs_value)
d, v 
```

```py
("abs1_x, abs1_v_0, abs1_v, abs1_v_1 = z3.Real('abs1_x'), z3.Real('abs1_v_0'), z3.Real('abs1_v'), z3.Real('abs1_v_1')",
 'z3.Or(z3.And(z3.And(abs1_x < 0, abs1_v_0 == -abs1_x), abs1_v == abs1_v_0), z3.And(z3.And(z3.Not(abs1_x < 0), abs1_v_1 == abs1_x), abs1_v == abs1_v_1))')

```

我们还需要将得到的结果值 (`<prefix>_v`) 等同于我们之前定义的符号变量 `a1`。

```py
l2_src = "l2 = z3.And(a == abs1_x, a1 == abs1_v, %s)" % v
l2_src 
```

```py
'l2 = z3.And(a == abs1_x, a1 == abs1_v, z3.Or(z3.And(z3.And(abs1_x < 0, abs1_v_0 == -abs1_x), abs1_v == abs1_v_0), z3.And(z3.And(z3.Not(abs1_x < 0), abs1_v_1 == abs1_x), abs1_v == abs1_v_1)))'

```

应用声明和赋值。

```py
exec(d)
exec(l2_src) 
```

```py
l2 
```

a = abs1_x ∧ a1 = abs1_v ∧ (abs1_x < 0 ∧ abs1_v_0 = -abs1_x ∧ abs1_v = abs1_v_0 ∨ ¬(abs1_x < 0) ∧ abs1_v_1 = abs1_x ∧ abs1_v = abs1_v_1)

我们还需要对 `line: 3` 进行相同的操作，但以 `abs2` 作为前缀。

```py
b1 = z3.Real('b1')
d, v = gen_fn_summary('abs2_', abs_value)
l3_src = "l3_ = z3.And(b == abs2_x, b1 == abs2_v, %s)" % v
exec(d)
exec(l3_src) 
```

```py
l3_ 
```

b = abs2_x ∧ b1 = abs2_v ∧ (abs2_x < 0 ∧ abs2_v_0 = -abs2_x ∧ abs2_v = abs2_v_0 ∨ ¬(abs2_x < 0) ∧ abs2_v_1 = abs2_x ∧ abs2_v = abs2_v_1)

要获取 `line: 3` 的真实谓词集，我们需要添加来自 `line: 2` 的谓词。

```py
l3 = z3.And(l2, l3_) 
```

```py
l3 
```

a = abs1_x ∧ a1 = abs1_v ∧ (abs1_x < 0 ∧ abs1_v_0 = -abs1_x ∧ abs1_v = abs1_v_0 ∨ ¬(abs1_x < 0) ∧ abs1_v_1 = abs1_x ∧ abs1_v = abs1_v_1) ∧ b = abs2_x ∧ b1 = abs2_v ∧ (abs2_x < 0 ∧ abs2_v_0 = -abs2_x ∧ abs2_v = abs2_v_0 ∨ ¬(abs2_x < 0) ∧ abs2_v_1 = abs2_x ∧ abs2_v = abs2_v_1)

这个方程可以用 z3 简化一点。

```py
z3.simplify(l3) 
```

a = abs1_x ∧ a1 = abs1_v ∧ (¬(0 ≤ abs1_x) ∧ abs1_v_0 = -1·abs1_x ∧ abs1_v = abs1_v_0 ∨ 0 ≤ abs1_x ∧ abs1_v_1 = abs1_x ∧ abs1_v = abs1_v_1) ∧ b = abs2_x ∧ b1 = abs2_v ∧ (¬(0 ≤ abs2_x) ∧ abs2_v_0 = -1·abs2_x ∧ abs2_v = abs2_v_0 ∨ 0 ≤ abs2_x ∧ abs2_v_1 = abs2_x ∧ abs2_v = abs2_v_1)

到 `line: 4` 行，我们有一个条件。

```py
l4_cond = a1 > b1
l4 = z3.And(l3, l4_cond) 
```

对于 `line: 5`，我们定义了符号变量 `c_0`，假设我们采取了 *IF* 分支。

```py
c_0 = z3.Real('c_0')
l5 = z3.And(l4, c_0 == a1) 
```

对于 `line: 6`，采取了 *ELSE* 分支。因此，我们反转该条件。

```py
l6 = z3.And(l3, z3.Not(l4_cond)) 
```

对于 `line: 7`，我们定义 `c_1`。

```py
c_1 = z3.Real('c_1')
l7 = z3.And(l6, c_1 == b1) 
```

```py
s1 = z3.Solver()
s1.add(l5)
s1.check() 
```

**sat**

```py
m1 = s1.model()
sorted([(d, m1[d]) for d in m1.decls() if not d.name(
).startswith('abs')], key=lambda x: x[0].name()) 
```

```py
[(a, 1/2), (a1, 1/2), (b, -1/4), (b1, 1/4), (c_0, 1/2)]

```

```py
s2 = z3.Solver()
s2.add(l7)
s2.check() 
```

**sat**

```py
m2 = s2.model()
sorted([(d, m2[d]) for d in m2.decls() if not d.name(
).startswith('abs')], key=lambda x: x[0].name()) 
```

```py
[(a, -1/2), (a1, 1/2), (b, 1/2), (b1, 1/2), (c_1, 1/2)]

```

我们真正想要做的是自动化这个过程，因为手动做既麻烦又容易出错。本质上，我们希望有能力提取程序中的 *所有路径*，并对每条路径进行符号执行，这将生成覆盖程序所有可到达部分的输入。

## 简单符号模糊测试

我们定义了一个简单的 *符号模糊测试器*，它可以生成以下假设下的输入值 *符号化*：

+   程序中没有循环。

+   函数是自包含的。

+   没有递归。

+   变量没有重新赋值。

关键思想如下：我们从入口点遍历控制流图，并生成到给定深度的所有可能路径。然后我们收集路径上遇到的约束，并生成输入，以便遍历程序到该点。

我们基于类 `Fuzzer` 构建我们的模糊测试器。

```py
from Fuzzer import Fuzzer 
```

我们首先提取传递给函数的控制流图。我们还提供了一个钩子，供子类进行其处理。

```py
class SimpleSymbolicFuzzer(Fuzzer):
  """Simple symbolic fuzzer"""

    def __init__(self, fn, **kwargs):
  """Constructor.
 `fn` is the function to be fuzzed.
 Possible keyword parameters:
 * `max_depth` - the depth to which one should attempt
 to trace the execution (default 100) 
 * `max_tries` - the maximum number of attempts
 we will try to produce a value before giving up (default 100)
 * `max_iter` - the number of iterations we will attempt (default 100).
 """
        self.fn_name = fn.__name__
        py_cfg = PyCFG()
        py_cfg.gen_cfg(inspect.getsource(fn))
        self.fnenter, self.fnexit = py_cfg.functions[self.fn_name]
        self.used_variables = used_vars(fn)
        self.fn_args = list(inspect.signature(fn).parameters)
        self.z3 = z3.Solver()

        self.paths = None
        self.last_path = None

        self.options(kwargs)
        self.process()

    def process(self):
        ...  # to be defined later 
```

我们需要一些变量来控制我们愿意遍历多少。

`MAX_DEPTH` 是尝试跟踪执行深度的深度。

```py
MAX_DEPTH = 100 
```

`MAX_TRIES` 是我们在放弃之前尝试生成值的最大尝试次数。

```py
MAX_TRIES = 100 
```

`MAX_ITER` 是我们将尝试的迭代次数。

```py
MAX_ITER = 100 
```

`options()` 方法在模糊测试类中设置这些参数。

```py
class SimpleSymbolicFuzzer(SimpleSymbolicFuzzer):
    def options(self, kwargs):
        self.max_depth = kwargs.get('max_depth', MAX_DEPTH)
        self.max_tries = kwargs.get('max_tries', MAX_TRIES)
        self.max_iter = kwargs.get('max_iter', MAX_ITER)
        self._options = kwargs 
```

初始化生成控制流图并将其钩子连接到 `fnenter` 和 `fnexit`。

```py
symfz_ct = SimpleSymbolicFuzzer(check_triangle) 
```

```py
symfz_ct.fnenter, symfz_ct.fnexit 
```

```py
(id:9 line[1] parents: [] : enter: check_triangle(a, b, c),
 id:10 line[1] parents: [14, 15, 16, 19, 20, 21] : exit: check_triangle(a, b, c))

```

### 生成所有可能的路径

我们可以从 `fnenter` 开始使用 `get_all_paths()` 程序递归地检索函数中的所有路径。

策略如下：从函数入口点 `fnenter` 开始，并使用 CFG 递归地跟随子节点。在任何节点处都有分支，会有多个子节点。在其他节点处只有一个子节点。假设一个节点有 $n$ 个子节点，这样的节点将产生 $n$ 条路径。我们将当前节点附加到每条路径的头部，并返回由此产生的所有路径。

```py
class SimpleSymbolicFuzzer(SimpleSymbolicFuzzer):
    def get_all_paths(self, fenter, depth=0):
        if depth > self.max_depth:
            raise Exception('Maximum depth exceeded')
        if not fenter.children:
            return [[(0, fenter)]]

        fnpaths = []
        for idx, child in enumerate(fenter.children):
            child_paths = self.get_all_paths(child, depth + 1)
            for path in child_paths:
                # In a conditional branch, idx is 0 for IF, and 1 for Else
                fnpaths.append([(idx, fenter)] + path)
        return fnpaths 
```

这可以这样使用。

```py
symfz_ct = SimpleSymbolicFuzzer(check_triangle)
all_paths = symfz_ct.get_all_paths(symfz_ct.fnenter) 
```

```py
len(all_paths) 
```

```py
6

```

```py
all_paths[1] 
```

```py
[(0, id:24 line[1] parents: [] : enter: check_triangle(a, b, c)),
 (0, id:26 line[2] parents: [24] : _if: a == b),
 (0, id:27 line[3] parents: [26] : _if: a == c),
 (1, id:28 line[4] parents: [27] : _if: b == c),
 (0, id:30 line[7] parents: [28] : return 'Isosceles'),
 (0,
  id:25 line[1] parents: [29, 30, 31, 34, 35, 36] : exit: check_triangle(a, b, c))]

```

我们将 `get_all_paths()` 钩子初始化如下。

```py
class SimpleSymbolicFuzzer(SimpleSymbolicFuzzer):
    def process(self):
        self.paths = self.get_all_paths(self.fnenter)
        self.last_path = len(self.paths) 
```

### 提取所有约束

对于任何给定的路径，我们定义一个函数 `extract_constraints()` 来提取约束，以便它们可以直接用 *Z3* 执行。`idx` 代表所采取的特定分支。因此，如果在条件语句中采取了 `False` 分支，我们将附加条件的否定。

```py
class SimpleSymbolicFuzzer(SimpleSymbolicFuzzer):
    def extract_constraints(self, path):
        predicates = []
        for (idx, elt) in path:
            if isinstance(elt.ast_node, ast.AnnAssign):
                if elt.ast_node.target.id in {'_if', '_while'}:
                    s = to_src(elt.ast_node.annotation)
                    predicates.append(("%s" if idx == 0 else "z3.Not(%s)") % s)
                elif isinstance(elt.ast_node.annotation, ast.Call):
                    assert elt.ast_node.annotation.func.id == self.fn_name
                else:
                    node = elt.ast_node
                    t = ast.Compare(node.target, [ast.Eq()], [node.value])
                    predicates.append(to_src(t))
            elif isinstance(elt.ast_node, ast.Assign):
                node = elt.ast_node
                t = ast.Compare(node.targets[0], [ast.Eq()], [node.value])
                predicates.append(to_src(t))
            else:
                pass
        return predicates 
```

```py
symfz_ct = SimpleSymbolicFuzzer(check_triangle)
all_paths = symfz_ct.get_all_paths(symfz_ct.fnenter)
symfz_ct.extract_constraints(all_paths[0]) 
```

```py
['a == b', 'a == c', 'b == c']

```

```py
constraints = symfz_ct.extract_constraints(all_paths[1])
constraints 
```

```py
['a == b', 'a == c', 'z3.Not(b == c)']

```

### 使用简单符号模糊测试器进行模糊测试

为了实际生成解决方案，我们定义 `fuzz()`。为此，我们首先需要提取所有路径。然后选择特定的路径，并提取该路径中的约束，然后使用 *z3* 解决。

```py
from [contextlib](https://docs.python.org/3/library/contextlib.html) import contextmanager 
```

首先，我们为我们的当前求解器创建一个检查点，以便我们可以检查一个谓词，并在必要时回滚。

```py
@contextmanager
def checkpoint(z3solver):
    z3solver.push()
    yield z3solver
    z3solver.pop() 
```

`use_path()` 函数提取单个函数的约束，将其应用于我们的当前求解器（在检查点下），并在找到一些解决方案时返回结果。如果找到了解决方案，我们还会确保我们永远不会重用这些解决方案。

```py
class SimpleSymbolicFuzzer(SimpleSymbolicFuzzer):
    def solve_path_constraint(self, path):
        # re-initializing does not seem problematic.
        # a = z3.Int('a').get_id() remains the same.
        constraints = self.extract_constraints(path)
        decl = define_symbolic_vars(self.used_variables, '')
        exec(decl)

        solutions = {}
        with checkpoint(self.z3):
            st = 'self.z3.add(%s)' % ', '.join(constraints)
            eval(st)
            if self.z3.check() != z3.sat:
                return {}
            m = self.z3.model()
            solutions = {d.name(): m[d] for d in m.decls()}
            my_args = {k: solutions.get(k, None) for k in self.fn_args}
        predicate = 'z3.And(%s)' % ','.join(
            ["%s == %s" % (k, v) for k, v in my_args.items()])
        eval('self.z3.add(z3.Not(%s))' % predicate)
        return my_args 
```

我们定义 `get_path()` 来检索当前路径并更新使用的路径。

```py
class SimpleSymbolicFuzzer(SimpleSymbolicFuzzer):
    def get_next_path(self):
        self.last_path -= 1
        if self.last_path == -1:
            self.last_path = len(self.paths) - 1
        return self.paths[self.last_path] 
```

`fuzz()` 方法简单地按顺序解决每条路径。

```py
class SimpleSymbolicFuzzer(SimpleSymbolicFuzzer):
    def fuzz(self):
  """Produce one solution for each path.
 Returns a mapping of variable names to (symbolic) Z3 values."""
        for i in range(self.max_tries):
            res = self.solve_path_constraint(self.get_next_path())
            if res:
                return res

        return {} 
```

模糊测试器可以使用如下方式。请注意，我们需要使用 `as_long()` 将返回的符号变量转换为 Python 数字：

```py
a, b, c = None, None, None
symfz_ct = SimpleSymbolicFuzzer(check_triangle)
for i in range(1, 10):
    args = symfz_ct.fuzz()
    res = check_triangle(args['a'].as_long(),
                         args['b'].as_long(),
                         args['c'].as_long())
    print(args, "result:", res) 
```

```py
{'a': 2, 'b': 3, 'c': 3} result: Isosceles
{'a': 6, 'b': 4, 'c': 5} result: Scalene
{'a': 8, 'b': 7, 'c': 8} result: Isosceles
{'a': 9, 'b': 9, 'c': 10} result: Isosceles
{'a': 11, 'b': 11, 'c': 11} result: Equilateral
{'a': 13, 'b': 12, 'c': 12} result: Isosceles
{'a': 16, 'b': 14, 'c': 15} result: Scalene
{'a': 18, 'b': 17, 'c': 18} result: Isosceles
{'a': 19, 'b': 19, 'c': 20} result: Isosceles

```

对于符号分数，我们访问它们的分子和分母：

```py
symfz_av = SimpleSymbolicFuzzer(abs_value)
for i in range(1, 10):
    args = symfz_av.fuzz()
    abs_res = abs_value(args['x'].numerator_as_long() /
                        args['x'].denominator_as_long())
    print(args, "result:", abs_res) 
```

```py
{'x': 0} result: 0.0
{'x': -1/2} result: 0.5
{'x': 1/2} result: 0.5
{'x': -1/4} result: 0.25
{'x': 1/4} result: 0.25
{'x': -3/8} result: 0.375
{'x': 1/8} result: 0.125
{'x': -3/2} result: 1.5
{'x': 1/16} result: 0.0625

```

*SimpleSymbolicFuzzer* 对于我们检查的 *简单* 程序似乎工作得很好。

### 简单模糊测试器的问题

正如我们之前提到的，`SimpleSymbolicFuzzer` 还不能处理变量重新赋值。此外，它也无法考虑任何循环。例如，考虑以下程序。

```py
def gcd(a: int, b: int) -> int:
    if a < b:
        c: int = a
        a = b
        b = c

    while b != 0:
        c: int = a
        a = b
        b = c % b

    return a 
```

```py
show_cfg(gcd) 
```

<svg width="282pt" height="671pt" viewBox="0.00 0.00 281.77 670.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 666.5)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="169.9" y="-634.33" font-family="Times,serif" font-size="14.00">1: enter: gcd(a, b)</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="169.9" y="-557.33" font-family="Times,serif" font-size="14.00">2: if: a < b</text></g> <g id="edge2" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="71.9" y="-88.83" font-family="Times,serif" font-size="14.00">1: exit: gcd(a, b)</text></g> <g id="node3" class="node"><title>11</title> <text text-anchor="middle" x="71.9" y="-165.82" font-family="Times,serif" font-size="14.00">12: return a</text></g> <g id="edge1" class="edge"><title>11->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="130.9" y="-471.07" font-family="Times,serif" font-size="14.00">3: c: int = a</text></g> <g id="edge3" class="edge"><title>3->4</title> <text text-anchor="middle" x="156.86" y="-514.2" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="179.9" y="-252.07" font-family="Times,serif" font-size="14.00">7: while: b != 0</text></g> <g id="edge7" class="edge"><title>3->7</title> <text text-anchor="middle" x="213.65" y="-398.07" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="141.9" y="-398.07" font-family="Times,serif" font-size="14.00">4: a = b</text></g> <g id="edge4" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="149.9" y="-325.07" font-family="Times,serif" font-size="14.00">5: b = c</text></g> <g id="edge5" class="edge"><title>5->6</title></g> <g id="edge6" class="edge"><title>6->7</title></g> <g id="edge12" class="edge"><title>7->11</title> <text text-anchor="middle" x="136.11" y="-208.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node10" class="node"><title>8</title> <text text-anchor="middle" x="179.9" y="-165.82" font-family="Times,serif" font-size="14.00">8: c: int = a</text></g> <g id="edge9" class="edge"><title>7->8</title> <text text-anchor="middle" x="184.02" y="-208.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node9" class="node"><title>10</title> <text text-anchor="middle" x="227.9" y="-11.82" font-family="Times,serif" font-size="14.00">10: b = c % b</text></g> <g id="edge8" class="edge"><title>10->7</title></g> <g id="node11" class="node"><title>9</title> <text text-anchor="middle" x="196.9" y="-88.83" font-family="Times,serif" font-size="14.00">9: a = b</text></g> <g id="edge10" class="edge"><title>8->9</title></g> <g id="edge11" class="edge"><title>9->10</title></g></g></svg>

```py
from ExpectError import ExpectError 
```

```py
with ExpectError():
    symfz_gcd = SimpleSymbolicFuzzer(gcd, max_depth=1000, max_iter=10)
    for i in range(1, 100):
        r = symfz_gcd.fuzz()
        v = gcd(r['a'].as_long(), r['b'].as_long())
        print(r, v) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/3731434224.py", line 2, in <module>
    symfz_gcd = SimpleSymbolicFuzzer(gcd, max_depth=1000, max_iter=10)
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/2089833100.py", line 26, in __init__
    self.process()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/4234366425.py", line 3, in process
    self.paths = self.get_all_paths(self.fnenter)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/228300930.py", line 10, in get_all_paths
    child_paths = self.get_all_paths(child, depth + 1)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/228300930.py", line 10, in get_all_paths
    child_paths = self.get_all_paths(child, depth + 1)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/228300930.py", line 10, in get_all_paths
    child_paths = self.get_all_paths(child, depth + 1)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  [Previous line repeated 998 more times]
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/228300930.py", line 4, in get_all_paths
    raise Exception('Maximum depth exceeded')
Exception: Maximum depth exceeded (expected)

```

这里的问题是我们的 *SimpleSymbolicFuzzer* 没有循环和变量重新赋值的概念。我们将在下一节中看到如何解决这个问题。

## 高级符号模糊测试

我们接下来定义了一个名为 `SymbolicFuzzer` 的工具，它可以处理重新赋值和*循环展开*。

```py
class SymbolicFuzzer(SimpleSymbolicFuzzer):
  """Symbolic fuzzing with reassignments and loop unrolling"""

    def options(self, kwargs):
        super().options(kwargs) 
```

一旦允许重新赋值和循环展开，我们就必须处理新生成变量的命名问题。这正是我们将要解决的问题。

### 处理重新赋值

我们希望重命名表达式中的所有变量，使得变量带有其使用计数注释。这使得确定变量重新赋值成为可能。为此，我们定义了 `rename_variables()` 函数，当给定一个包含不同变量当前使用索引的 `env` 时，使用注释重命名传递的 AST 节点中的变量，并返回带有修改的副本。请注意，我们在这里不能使用 [NodeTransformer](https://docs.python.org/3/library/ast.html#ast.NodeTransformer)，因为它会修改 AST。

即，如果表达式是 `env[v] == 1`，则将 `v` 重命名为 `_v_1`

```py
def rename_variables(astnode, env):
    if isinstance(astnode, ast.BoolOp):
        fn = 'z3.And' if isinstance(astnode.op, ast.And) else 'z3.Or'
        return ast.Call(
            ast.Name(fn, None),
            [rename_variables(i, env) for i in astnode.values], [])
    elif isinstance(astnode, ast.BinOp):
        return ast.BinOp(
            rename_variables(astnode.left, env), astnode.op,
            rename_variables(astnode.right, env))
    elif isinstance(astnode, ast.UnaryOp):
        if isinstance(astnode.op, ast.Not):
            return ast.Call(
                ast.Name('z3.Not', None),
                [rename_variables(astnode.operand, env)], [])
        else:
            return ast.UnaryOp(astnode.op,
                               rename_variables(astnode.operand, env))
    elif isinstance(astnode, ast.Call):
        return ast.Call(astnode.func,
                        [rename_variables(i, env) for i in astnode.args],
                        astnode.keywords)
    elif isinstance(astnode, ast.Compare):
        return ast.Compare(
            rename_variables(astnode.left, env), astnode.ops,
            [rename_variables(i, env) for i in astnode.comparators])
    elif isinstance(astnode, ast.Name):
        if astnode.id not in env:
            env[astnode.id] = 0
        num = env[astnode.id]
        return ast.Name('_%s_%d' % (astnode.id, num), astnode.ctx)
    elif isinstance(astnode, ast.Return):
        return ast.Return(rename_variables(astnode.value, env))
    else:
        return astnode 
```

为了验证它按预期工作，我们从一个环境开始。

```py
env = {'x': 1} 
```

```py
ba = get_expression('x == 1 and y == 2')
type(ba) 
```

```py
ast.BoolOp

```

```py
assert to_src(rename_variables(ba, env)) == 'z3.And(_x_1 == 1, _y_0 == 2)' 
```

```py
bo = get_expression('x == 1 or y == 2')
type(bo.op) 
```

```py
ast.Or

```

```py
assert to_src(rename_variables(bo, env)) == 'z3.Or(_x_1 == 1, _y_0 == 2)' 
```

```py
b = get_expression('x + y')
type(b) 
```

```py
ast.BinOp

```

```py
assert to_src(rename_variables(b, env)) == '_x_1 + _y_0' 
```

```py
u = get_expression('-y')
type(u) 
```

```py
ast.UnaryOp

```

```py
assert to_src(rename_variables(u, env)) == '-_y_0' 
```

```py
un = get_expression('not y')
type(un.op) 
```

```py
ast.Not

```

```py
assert to_src(rename_variables(un, env)) == 'z3.Not(_y_0)' 
```

```py
c = get_expression('x == y')
type(c) 
```

```py
ast.Compare

```

```py
assert to_src(rename_variables(c, env)) == '_x_1 == _y_0' 
```

```py
f = get_expression('fn(x,y)')
type(f) 
```

```py
ast.Call

```

```py
assert to_src(rename_variables(f, env)) == 'fn(_x_1, _y_0)' 
```

```py
env 
```

```py
{'x': 1, 'y': 0}

```

接下来，我们想要处理控制流图（CFG），并正确转换路径。

### 跟踪赋值

为了在 CFG 中跟踪赋值，我们定义了一个名为 `PNode` 的数据结构，用于存储当前的 CFG 节点。

```py
class PNode:
    def __init__(self, idx, cfgnode, parent=None, order=0, seen=None):
        self.seen = {} if seen is None else seen
        self.max_iter = MAX_ITER
        self.idx, self.cfgnode, self.parent, self.order = idx, cfgnode, parent, order

    def __repr__(self):
        return "PNode:%d[%s order:%d]" % (self.idx, str(self.cfgnode),
                                          self.order) 
```

定义新的 `PNode` 如下所示。

```py
cfg = PyCFG()
cfg.gen_cfg(inspect.getsource(gcd))
gcd_fnenter, _ = cfg.functions['gcd'] 
```

```py
PNode(0, gcd_fnenter) 
```

```py
PNode:0[id:27 line[1] parents: [] : enter: gcd(a, b) order:0]

```

`copy()` 方法为子节点生成一个副本，指示所采取的路径（使用子节点的 `order`）。

```py
class PNode(PNode):
    def copy(self, order):
        p = PNode(self.idx, self.cfgnode, self.parent, order, self.seen)
        assert p.order == order
        return p 
```

使用复制操作。

```py
PNode(0, gcd_fnenter).copy(1) 
```

```py
PNode:0[id:27 line[1] parents: [] : enter: gcd(a, b) order:1]

```

#### 路径的逐步探索

我们 `SimpleSymbolicFuzzer` 的问题之一是在尝试另一条路径之前会探索一条路径到完成。然而，这是非最优的。有人可能希望以更逐步的方式探索图，每次只扩展一个可能的执行。

因此，我们定义了 `explore()` 函数，该函数会逐个探索节点的子节点（如果有的话）。如果彻底执行，这将生成从起始节点到没有更多子节点为止的所有路径。我们将 `PNode` 定义为一个容器类，以便可以从外部驱动此迭代，并在达到最大迭代次数或需要优先考虑某些路径时停止。

```py
class PNode(PNode):
    def explore(self):
        ret = []
        for (i, n) in enumerate(self.cfgnode.children):
            key = "[%d]%s" % (self.idx + 1, n)
            ccount = self.seen.get(key, 0)
            if ccount > self.max_iter:
                continue  # drop this child
            self.seen[key] = ccount + 1
            pn = PNode(self.idx + 1, n, self.copy(i), seen=self.seen)
            ret.append(pn)
        return ret 
```

我们可以使用 `explore()` 如下。

```py
PNode(0, gcd_fnenter).explore() 
```

```py
[PNode:1[id:29 line[2] parents: [27] : _if: a < b order:0]]

```

```py
PNode(0, gcd_fnenter).explore()[0].explore() 
```

```py
[PNode:2[id:30 line[3] parents: [29] : c: int = a order:0],
 PNode:2[id:33 line[7] parents: [32, 29, 36] : _while: b != 0 order:0]]

```

方法 `get_path_to_root()` 递归地通过子节点到父节点的链向上检索，直到最顶层的父节点。

```py
class PNode(PNode):
    def get_path_to_root(self):
        path = []
        n = self
        while n:
            path.append(n)
            n = n.parent
        return list(reversed(path)) 
```

```py
p = PNode(0, gcd_fnenter)
[s.get_path_to_root() for s in p.explore()[0].explore()[0].explore()[0].explore()] 
```

```py
[[PNode:0[id:27 line[1] parents: [] : enter: gcd(a, b) order:0],
  PNode:1[id:29 line[2] parents: [27] : _if: a < b order:0],
  PNode:2[id:30 line[3] parents: [29] : c: int = a order:0],
  PNode:3[id:31 line[4] parents: [30] : a = b order:0],
  PNode:4[id:32 line[5] parents: [31] : b = c order:0]]]

```

节点的字符串表示形式是 `z3` 可解形式。

```py
class PNode(PNode):
    def __str__(self):
        path = self.get_path_to_root()
        ssa_path = to_single_assignment_predicates(path)
        return ', '.join([to_src(p) for p in ssa_path]) 
```

然而，在使用它之前，我们需要注意变量重命名，以便重新赋值可以工作。

#### 重命名使用过的变量

我们需要重命名使用的变量。任何变量`v = xxx`都应该重命名为`_v_0`，任何后续的赋值，如`v = v + 1`，都应该转换为`_v_1 = _v_0 + 1`，以及后续的条件，如`v == x`，应该转换为`(_v_1 == _x_0)`。`to_single_assignment_predicates()`方法为给定路径执行此操作。

```py
def to_single_assignment_predicates(path):
    env = {}
    new_path = []
    for i, node in enumerate(path):
        ast_node = node.cfgnode.ast_node
        new_node = None
        if isinstance(ast_node, ast.AnnAssign) and ast_node.target.id in {
                'exit'}:
            new_node = None
        elif isinstance(ast_node, ast.AnnAssign) and ast_node.target.id in {'enter'}:
            args = [
                ast.parse(
                    "%s == _%s_0" %
                    (a.id, a.id)).body[0].value for a in ast_node.annotation.args]
            new_node = ast.Call(ast.Name('z3.And', None), args, [])
        elif isinstance(ast_node, ast.AnnAssign) and ast_node.target.id in {'_if', '_while'}:
            new_node = rename_variables(ast_node.annotation, env)
            if node.order != 0:
                assert node.order == 1
                new_node = ast.Call(ast.Name('z3.Not', None), [new_node], [])
        elif isinstance(ast_node, ast.AnnAssign):
            assigned = ast_node.target.id
            val = [rename_variables(ast_node.value, env)]
            env[assigned] = 0 if assigned not in env else env[assigned] + 1
            target = ast.Name('_%s_%d' %
                              (ast_node.target.id, env[assigned]), None)
            new_node = ast.Expr(ast.Compare(target, [ast.Eq()], val))
        elif isinstance(ast_node, ast.Assign):
            assigned = ast_node.targets[0].id
            val = [rename_variables(ast_node.value, env)]
            env[assigned] = 0 if assigned not in env else env[assigned] + 1
            target = ast.Name('_%s_%d' %
                              (ast_node.targets[0].id, env[assigned]), None)
            new_node = ast.Expr(ast.Compare(target, [ast.Eq()], val))
        elif isinstance(ast_node, (ast.Return, ast.Pass)):
            new_node = None
        else:
            s = "NI %s  %s" % (type(ast_node), ast_node.target.id)
            raise Exception(s)
        new_path.append(new_node)
    return new_path 
```

这里是如何使用它的：

```py
p = PNode(0, gcd_fnenter)
path = p.explore()[0].explore()[0].explore()[0].get_path_to_root()
spath = to_single_assignment_predicates(path) 
```

```py
[to_src(s) for s in spath] 
```

```py
['z3.And(a == _a_0, b == _b_0)', '_a_0 < _b_0', '_c_0 == _a_0', '_a_1 == _b_0']

```

#### 在循环前检查

*concolic*执行简化*symbolic*执行的一种方式是在循环的处理上。我们不是试图确定循环的不变量，而是简单地*展开*循环多次，直到达到`MAX_DEPTH`限制。然而，并不是所有的循环都需要展开到`MAX_DEPTH`才停止。其中一些可能在之前就退出了。因此，在继续进一步探索之前，检查给定的约束集是否可以满足是必要的。

```py
def identifiers_with_types(identifiers, defined):
    with_types = dict(defined)
    for i in identifiers:
        if i[0] == '_':
            nxt = i[1:].find('_', 1)
            name = i[1:nxt + 1]
            assert name in defined
            typ = defined[name]
            with_types[i] = typ
    return with_types 
```

`extract_constraints()`从路径中生成`z3`约束。主要工作由`to_single_assignment_predicates()`完成。然后`extract_constraints()`将 AST 转换为源代码。

```py
class SymbolicFuzzer(SymbolicFuzzer):
    def extract_constraints(self, path):
        return [to_src(p) for p in to_single_assignment_predicates(path) if p] 
```

### 解决路径约束

现在，我们更新我们的`solve_path_constraint()`方法，以考虑在重新赋值过程中创建的新标识符。

```py
class SymbolicFuzzer(SymbolicFuzzer):
    def solve_path_constraint(self, path):
        # re-initializing does not seem problematic.
        # a = z3.Int('a').get_id() remains the same.
        constraints = self.extract_constraints(path)
        identifiers = [
            c for i in constraints for c in used_identifiers(i)]  # <- changes
        with_types = identifiers_with_types(
            identifiers, self.used_variables)  # <- changes
        decl = define_symbolic_vars(with_types, '')
        exec(decl)

        solutions = {}
        with checkpoint(self.z3):
            st = 'self.z3.add(%s)' % ', '.join(constraints)
            eval(st)
            if self.z3.check() != z3.sat:
                return {}
            m = self.z3.model()
            solutions = {d.name(): m[d] for d in m.decls()}
            my_args = {k: solutions.get(k, None) for k in self.fn_args}

        predicate = 'z3.And(%s)' % ','.join(
            ["%s == %s" % (k, v) for k, v in my_args.items()])
        eval('self.z3.add(z3.Not(%s))' % predicate)

        return my_args 
```

### 生成所有路径

`get_all_paths()`现在也进行了类似的更新，以便只展开到指定的层数。它还被转换为迭代探索风格，以便以广度优先的方式探索 CFG。

```py
class SymbolicFuzzer(SymbolicFuzzer):
    def get_all_paths(self, fenter):
        path_lst = [PNode(0, fenter)]
        completed = []
        for i in range(self.max_iter):
            new_paths = [PNode(0, fenter)]
            for path in path_lst:
                # explore each path once
                if path.cfgnode.children:
                    np = path.explore()
                    for p in np:
                        if path.idx > self.max_depth:
                            break
                        new_paths.append(p)
                else:
                    completed.append(path)
            path_lst = new_paths
        return completed + path_lst 
```

我们现在可以使用我们的高级 SymbolicFuzzer 如下获得所有路径。

```py
asymfz_gcd = SymbolicFuzzer(
    gcd, max_iter=10, max_tries=10, max_depth=10)
all_paths = asymfz_gcd.get_all_paths(asymfz_gcd.fnenter) 
```

```py
len(all_paths) 
```

```py
38

```

```py
all_paths[37].get_path_to_root() 
```

```py
[PNode:0[id:40 line[1] parents: [] : enter: gcd(a, b) order:0],
 PNode:1[id:42 line[2] parents: [40] : _if: a < b order:1],
 PNode:2[id:46 line[7] parents: [45, 42, 49] : _while: b != 0 order:0],
 PNode:3[id:47 line[8] parents: [46] : c: int = a order:0],
 PNode:4[id:48 line[9] parents: [47] : a = b order:0],
 PNode:5[id:49 line[10] parents: [48] : b = c % b order:0],
 PNode:6[id:46 line[7] parents: [45, 42, 49] : _while: b != 0 order:0],
 PNode:7[id:47 line[8] parents: [46] : c: int = a order:0],
 PNode:8[id:48 line[9] parents: [47] : a = b order:0],
 PNode:9[id:49 line[10] parents: [48] : b = c % b order:0],
 PNode:10[id:46 line[7] parents: [45, 42, 49] : _while: b != 0 order:0]]

```

我们还可以列出每个路径中的谓词。

```py
for s in to_single_assignment_predicates(all_paths[37].get_path_to_root()):
    if s is not None:
        print(to_src(s)) 
```

```py
z3.And(a == _a_0, b == _b_0)
z3.Not(_a_0 < _b_0)
_b_0 != 0
_c_0 == _a_0
_a_1 == _b_0
_b_1 == _c_0 % _b_0
_b_1 != 0
_c_1 == _a_1
_a_2 == _b_1
_b_2 == _c_1 % _b_1
_b_2 != 0

```

```py
constraints = asymfz_gcd.extract_constraints(all_paths[37].get_path_to_root()) 
```

```py
constraints 
```

```py
['z3.And(a == _a_0, b == _b_0)',
 'z3.Not(_a_0 < _b_0)',
 '_b_0 != 0',
 '_c_0 == _a_0',
 '_a_1 == _b_0',
 '_b_1 == _c_0 % _b_0',
 '_b_1 != 0',
 '_c_1 == _a_1',
 '_a_2 == _b_1',
 '_b_2 == _c_1 % _b_1',
 '_b_2 != 0']

```

打印出的约束表明我们的变量重命名方法成功。我们只需要一个额外的部分来完成拼图。我们的路径仍然是一个`PNode`。我们需要修改`get_next_path()`，以便返回相应的谓词链。

```py
class SymbolicFuzzer(SymbolicFuzzer):
    def get_next_path(self):
        self.last_path -= 1
        if self.last_path == -1:
            self.last_path = len(self.paths) - 1
        return self.paths[self.last_path].get_path_to_root() 
```

我们将在下一节中看到如何使用我们的 fuzzer 进行模糊测试。

### 使用高级 SymbolicFuzzer 进行模糊测试

我们使用我们的高级 SymbolicFuzzer 对*gcd*进行模糊测试以生成合理的输入。

```py
asymfz_gcd = SymbolicFuzzer(
    gcd, max_tries=10, max_iter=10, max_depth=10)
data = []
for i in range(10):
    r = asymfz_gcd.fuzz()
    data.append((r['a'].as_long(), r['b'].as_long()))
    v = gcd(*data[-1])
    print(r, "result:", repr(v)) 
```

```py
{'a': 8, 'b': 3} result: 1
{'a': 1, 'b': 2} result: 1
{'a': 2, 'b': 5} result: 1
{'a': 5, 'b': 2} result: 1
{'a': 3, 'b': 4} result: 1
{'a': 9, 'b': 9} result: 9
{'a': 7, 'b': 6} result: 1
{'a': 5, 'b': 10} result: 5
{'a': 3, 'b': 1} result: 1
{'a': 10, 'b': 7} result: 1

```

输出看起来是合理的。然而，我们获得了多少覆盖率？

```py
with VisualizedArcCoverage() as cov:
    for a, b in data:
        gcd(a, b) 
```

```py
cov.show_coverage(gcd) 
```

```py
#  1: def gcd(a: int, b: int) -> int:
#  2:     if a < b:
#  3:         c: int = a  # type: ignore
#  4:         a = b
#  5:         b = c
   6: 
#  7:     while b != 0:
#  8:         c: int = a  # type: ignore
#  9:         a = b
# 10:         b = c % b
  11: 
# 12:     return a
  13: 

```

```py
show_cfg(gcd, arcs=cov.arcs()) 
```

<svg width="282pt" height="636pt" viewBox="0.00 0.00 281.77 636.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 632)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="173.9" y="-599.83" font-family="Times,serif" font-size="14.00">1: enter: gcd(a, b)</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="173.9" y="-523.83" font-family="Times,serif" font-size="14.00">2: if: a < b</text></g> <g id="edge2" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="71.9" y="-87.83" font-family="Times,serif" font-size="14.00">1: exit: gcd(a, b)</text></g> <g id="node3" class="node"><title>11</title> <text text-anchor="middle" x="71.9" y="-163.82" font-family="Times,serif" font-size="14.00">12: return a</text></g> <g id="edge1" class="edge"><title>11->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="134.9" y="-451.82" font-family="Times,serif" font-size="14.00">3: c: int = a</text></g> <g id="edge3" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="179.9" y="-235.82" font-family="Times,serif" font-size="14.00">7: while: b != 0</text></g> <g id="edge7" class="edge"><title>3->7</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="145.9" y="-379.82" font-family="Times,serif" font-size="14.00">4: a = b</text></g> <g id="edge4" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="151.9" y="-307.82" font-family="Times,serif" font-size="14.00">5: b = c</text></g> <g id="edge5" class="edge"><title>5->6</title></g> <g id="edge6" class="edge"><title>6->7</title></g> <g id="edge12" class="edge"><title>7->11</title></g> <g id="node10" class="node"><title>8</title> <text text-anchor="middle" x="179.9" y="-163.82" font-family="Times,serif" font-size="14.00">8: c: int = a</text></g> <g id="edge9" class="edge"><title>7->8</title></g> <g id="node9" class="node"><title>10</title> <text text-anchor="middle" x="227.9" y="-11.82" font-family="Times,serif" font-size="14.00">10: b = c % b</text></g> <g id="edge8" class="edge"><title>10->7</title></g> <g id="node11" class="node"><title>9</title> <text text-anchor="middle" x="196.9" y="-87.83" font-family="Times,serif" font-size="14.00">9: a = b</text></g> <g id="edge10" class="edge"><title>8->9</title></g> <g id="edge11" class="edge"><title>9->10</title></g></g></svg>

事实上，分支和语句覆盖率可视化似乎都表明我们实现了完整的覆盖率。我们如何在实践中使用我们的 fuzzer 呢？我们探索了一个解决二次方程根的程序的小案例研究。

#### 示例：二次方程的根

这里是寻找二次方程根的著名方程。

```py
from [typing](https://docs.python.org/3/library/typing.html) import Tuple 
```

```py
def roots(a: float, b: float, c: float) -> Tuple[float, float]:
    d: float = b * b - 4 * a * c
    ax: float = 0.5 * d
    bx: float = 0
    while (ax - bx) > 0.1:
        bx = 0.5 * (ax + d / ax)
        ax = bx

    s: float = bx
    a2: float = 2 * a
    ba2: float = b / a2

    return -ba2 + s / a2, -ba2 - s / a2 
```

程序看起来正确吗？让我们调查程序是否合理。但在那之前，我们需要一个辅助函数`sym_to_float()`来将符号值转换为浮点数。

```py
def sym_to_float(v):
    if v is None:
        return math.inf
    elif isinstance(v, z3.IntNumRef):
        return v.as_long()
    return v.numerator_as_long() / v.denominator_as_long() 
```

现在，我们准备开始模糊测试。

```py
asymfz_roots = SymbolicFuzzer(
    roots,
    max_tries=10,
    max_iter=10,
    max_depth=10) 
```

```py
with ExpectError():
    for i in range(100):
        r = asymfz_roots.fuzz()
        print(r)
        d = [sym_to_float(r[i]) for i in ['a', 'b', 'c']]
        v = roots(*d)
        print(d, v) 
```

```py
{'a': 11/10, 'b': 0, 'c': -1/2}
[1.1, 0.0, -0.5] (0.7045454545454545, -0.7045454545454545)
{'a': 0, 'b': 58617/131072, 'c': -1/2}

```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/94617992.py", line 6, in <module>
    v = roots(*d)
        ^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/2299859991.py", line 11, in roots
    ba2: float = b / a2
                 ~~^~~~
ZeroDivisionError: float division by zero (expected)

```

我们有一个`ZeroDivisionError`。我们能消除它吗？

##### 根 - 在除法前检查

```py
def roots2(a: float, b: float, c: float) -> Tuple[float, float]:
    d: float = b * b - 4 * a * c

    xa: float = 0.5 * d
    xb: float = 0
    while (xa - xb) > 0.1:
        xb = 0.5 * (xa + d / xa)
        xa = xb

    s: float = xb

    if a == 0:
        return -c / b, -c / b  # only one solution

    a2: float = 2 * a
    ba2: float = b / a2
    return -ba2 + s / a2, -ba2 - s / a2 
```

```py
asymfz_roots = SymbolicFuzzer(
    roots2,
    max_tries=10,
    max_iter=10,
    max_depth=10) 
```

```py
with ExpectError():
    for i in range(1000):
        r = asymfz_roots.fuzz()
        d = [sym_to_float(r[i]) for i in ['a', 'b', 'c']]
        v = roots2(*d)
        #print(d, v) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/567876003.py", line 5, in <module>
    v = roots2(*d)
        ^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_16239/714795544.py", line 13, in roots2
    return -c / b, -c / b  # only one solution
           ~~~^~~
ZeroDivisionError: float division by zero (expected)

```

显然，我们的修复是不完整的。让我们再试一次。

##### 根 - 消除零除错误

```py
import [math](https://docs.python.org/3/library/math.html) 
```

```py
def roots3(a: float, b: float, c: float) -> Tuple[float, float]:
    d: float = b * b - 4 * a * c

    xa: float = 0.5 * d
    xb: float = 0
    while (xa - xb) > 0.1:
        xb = 0.5 * (xa + d / xa)
        xa = xb
    s: float = xb

    if a == 0:
        if b == 0:
            return math.inf, math.inf
        return -c / b, -c / b  # only one solution

    a2: float = 2 * a
    ba2: float = b / a2
    return -ba2 + s / a2, -ba2 - s / a2 
```

```py
asymfz_roots = SymbolicFuzzer(
    roots3,
    max_tries=10,
    max_iter=10,
    max_depth=10) 
```

```py
for i in range(10):
    r = asymfz_roots.fuzz()
    print(r)
    d = [sym_to_float(r[i]) for i in ['a', 'b', 'c']]
    v = roots3(*d)
    print(d, v) 
```

```py
{'a': -1, 'b': 0, 'c': 0}
[-1.0, 0.0, 0.0] (0.0, 0.0)
{'a': -11/20, 'b': 0, 'c': 1}
[-0.55, 0.0, 1.0] (-1.409090909090909, 1.409090909090909)
{'a': -1/40, 'b': 0, 'c': 2}
[-0.025, 0.0, 2.0] (0.0, 0.0)
{'a': 0, 'b': -1/4, 'c': 3}
[0.0, -0.25, 3.0] (12.0, 12.0)
{'a': 0, 'b': 0, 'c': 4}
[0.0, 0.0, 4.0] (inf, inf)
{'a': 0, 'b': -777645/524288, 'c': 5}
[0.0, -1.4832401275634766, 5.0] (3.370998334715712, 3.370998334715712)
{'a': -2/91, 'b': 0, 'c': 91/40}
[-0.02197802197802198, 0.0, 2.275] (0.0, 0.0)
{'a': 0, 'b': 4/9, 'c': 6}
[0.0, 0.4444444444444444, 6.0] (-13.5, -13.5)
{'a': 0, 'b': 0, 'c': 7}
[0.0, 0.0, 7.0] (inf, inf)
{'a': -7/6, 'b': -1, 'c': 3/2}
[-1.1666666666666667, -1.0, 1.5] (-1.7142857142857142, 0.857142857142857)

```

通过这种方式，我们已经证明了我们可以使用我们的*SymbolicFuzzer*来模糊测试程序，并且它可以帮助识别代码中的问题。

## 局限性

在`roots3()`函数中存在一个明显的错误。我们没有检查负根。然而，符号执行似乎没有检测到它。我们为什么不能检测到负根的问题？因为我们停止执行并在预定深度处没有抛出错误。也就是说，我们的符号执行是宽泛但浅层的。克服这种限制的一种方法是通过依赖并发符号执行，这允许一个人比纯符号执行更深入。

第二个问题是符号执行必然是计算密集型的。这意味着基于规范的模糊测试通常能够生成更大的一组输入，并且对不检查魔数字节程序提供更多的覆盖率，从而提供合理的探索梯度。

## 经验教训

+   可以使用符号执行来增强探索程序所有特性的输入。

+   符号执行可以广泛但浅层。

+   符号执行非常适合依赖于特定值存在于输入中的程序，然而，当这些值不存在时，其效用会降低，并且输入空间在覆盖率方面代表一个梯度。

## 下一步

+   基于搜索的模糊测试在随机模糊测试无法提供足够结果，但符号模糊测试又过于沉重时，通常可以是一个可接受的折中方案。

## 背景

程序的符号执行最初由 King 在 1976 年描述[[King 等人，1976](https://doi.org/10.1145/360248.360252)]。它在软件漏洞分析中得到了广泛的应用，特别是二进制程序。一些著名的符号执行工具包括*KLEE* [Cadar 等人，2008]，*angr* [Wang 等人，2017]，*Driller* [Stephens 等人，2016]，和*SAGE* [Godefroid 等人，2012]。最著名的 Python 符号执行环境是 CHEF [[Bucur 等人，2014](https://doi.org/10.1145/2541940.2541977)]，它通过修改解释器来进行符号执行。

本章中使用的 Z3 求解器是在微软研究院由 Leonardo de Moura 和 Nikolaj Bjørner 领导开发的[[De Moura 等人，2008](https://link.springer.com/chapter/10.1007/978-3-540-78800-3_24)]。它是最受欢迎的求解器之一。

## 练习

### 练习 1：扩展符号模糊测试以使用函数摘要

我们在第一部分展示了如何生成函数摘要。你能扩展`SymbolicFuzzer`以在需要时使用函数摘要吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/SymbolicFuzzer.ipynb#Exercises)来练习并查看解决方案。

### 练习 2：静态检查循环是否应该进一步展开

我们研究了在固定深度探索过程中循环展开的情况。然而，并非所有循环都需要完全展开。一些循环可能只包含固定数量的迭代次数。例如，考虑下面的循环。

```py
i = 0
while i < 10:
    i += 1 
```

这个循环需要正好展开 10 次。对于这种情况，你能实现一个名为`can_be_satisfied()`的方法，如下所示，只有当路径条件可以满足时才进一步展开。

```py
class SymbolicFuzzer(SymbolicFuzzer):
    def get_all_paths(self, fenter):
        path_lst = [PNode(0, fenter)]
        completed = []
        for i in range(self.max_iter):
            new_paths = [PNode(0, fenter)]
            for path in path_lst:
                # explore each path once
                if path.cfgnode.children:
                    np = path.explore()
                    for p in np:
                        if path.idx > self.max_depth:
                            break
                        if self.can_be_satisfied(p):
                            new_paths.append(p)
                        else:
                            break
                else:
                    completed.append(path)
            path_lst = new_paths
        return completed + path_lst 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/SymbolicFuzzer.ipynb#Exercises)来处理练习并查看解决方案。

**解决方案**。这是一个解决方案。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/SymbolicFuzzer.ipynb#Exercises)来处理练习并查看解决方案。

### 练习 3：实现一个 Concolic Fuzzer

我们在关于 concolic fuzzing 的章节中看到了如何使用信息流来追踪函数的 concolic。然而，这并不十分理想，因为当信息流是间接的（如在基于控制流的信息流中）时，约束可能会丢失。你能使用我们为符号执行构建的基础设施来实现 concolic 追踪吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/SymbolicFuzzer.ipynb#Exercises)来处理练习并查看解决方案。

**解决方案**。这是一个可能的解决方案。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/SymbolicFuzzer.ipynb#Exercises)来处理练习并查看解决方案。

的确，追踪的路径现在不同了。可以通过重复此过程必要的次数来探索所有附近的路径。

你能将这种探索纳入 concolic fuzzer 中吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/SymbolicFuzzer.ipynb#Exercises)来处理练习并查看解决方案。

## 兼容性

本章的早期版本使用`AdvancedSymbolicFuzzer`作为`SymbolicFuzzer`的名称。

```py
AdvancedSymbolicFuzzer = SymbolicFuzzer 
```

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT License](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。 [最后更改：2025-01-22 09:37:42+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/SymbolicFuzzer.ipynb) • 引用 • [ imprint](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser 和 Christian Holler: "[符号模糊测试](https://www.fuzzingbook.org/html/SymbolicFuzzer.html)". 在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser 和 Christian Holler 的著作 "[模糊测试书籍](https://www.fuzzingbook.org/)" 中，[`www.fuzzingbook.org/html/SymbolicFuzzer.html`](https://www.fuzzingbook.org/html/SymbolicFuzzer.html). 2025-01-22 09:37:42+01:00 获取。

```py
@incollection{fuzzingbook2025:SymbolicFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Symbolic Fuzzing},
    year = {2025},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/SymbolicFuzzer.html}},
    note = {Retrieved 2025-01-22 09:37:42+01:00},
    url = {https://www.fuzzingbook.org/html/SymbolicFuzzer.html},
    urldate = {2025-01-22 09:37:42+01:00}
}

```
