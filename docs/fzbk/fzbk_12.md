# 变异分析

> 原文：[`www.fuzzingbook.org/html/MutationAnalysis.html`](http://www.fuzzingbook.org/html/MutationAnalysis.html)

在关于覆盖的章节中，我们展示了如何确定程序中哪些部分被程序执行，从而获得一组测试用例在覆盖程序结构方面的有效性感。然而，覆盖率本身可能不是衡量测试有效性的最佳指标，因为即使从未检查结果是否正确，也可能有很高的覆盖率。在本章中，我们介绍了一种评估测试套件有效性的另一种方法：在代码中注入*变异*（*人工故障*）后，我们检查测试套件是否可以检测这些人工故障。其思想是，如果它未能检测到这种变异，它也会错过真实错误。

**先决条件**

+   您需要了解程序是如何执行的。

+   您应该已经阅读了关于覆盖的章节。

## 摘要

要使用本章提供的代码，请编写

```py
>>> from fuzzingbook.MutationAnalysis import <identifier> 
```

然后利用以下功能。

本章介绍了两种在主题程序上运行*变异分析*的方法。第一种类`MuFunctionAnalyzer`针对单个函数。给定一个函数`gcd`和两个测试用例评估，可以对测试用例进行如下变异分析：

```py
>>> for mutant in MuFunctionAnalyzer(gcd, log=True):
>>>     with mutant:
>>>         assert gcd(1, 0) == 1, "Minimal"
>>>         assert gcd(0, 1) == 1, "Mirror"
>>> mutant.pm.score()
->  gcd_1
<-  gcd_1
Detected gcd_1 <class 'UnboundLocalError'> cannot access local variable 'c' where it is not associated with a value

->  gcd_2
<-  gcd_2
Detected gcd_2 <class 'AssertionError'> Mirror

->  gcd_3
<-  gcd_3

->  gcd_4
<-  gcd_4

->  gcd_5
<-  gcd_5

->  gcd_6
<-  gcd_6

->  gcd_7
<-  gcd_7
Detected gcd_7 <class 'AssertionError'> Minimal

0.42857142857142855 
```

第二种类`MuProgramAnalyzer`针对具有测试套件的独立程序。给定一个程序`gcd`，其源代码在`gcd_src`中提供，测试套件由`TestGCD`提供，可以如下评估`TestGCD`的变异分数：

```py
>>> class TestGCD(unittest.TestCase):
>>>     def test_simple(self):
>>>         assert cfg.gcd(1, 0) == 1
>>> 
>>>     def test_mirror(self):
>>>         assert cfg.gcd(0, 1) == 1
>>> for mutant in MuProgramAnalyzer('gcd', gcd_src):
>>>     mutant[test_module].runTest('TestGCD')
>>> mutant.pm.score()
======================================================================
FAIL: test_simple (__main__.TestGCD.test_simple)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 3, in test_simple
    assert cfg.gcd(1, 0) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_simple (__main__.TestGCD.test_simple)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 3, in test_simple
    assert cfg.gcd(1, 0) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_simple (__main__.TestGCD.test_simple)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 3, in test_simple
    assert cfg.gcd(1, 0) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_simple (__main__.TestGCD.test_simple)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 3, in test_simple
    assert cfg.gcd(1, 0) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_simple (__main__.TestGCD.test_simple)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 3, in test_simple
    assert cfg.gcd(1, 0) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_simple (__main__.TestGCD.test_simple)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 3, in test_simple
    assert cfg.gcd(1, 0) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_simple (__main__.TestGCD.test_simple)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 3, in test_simple
    assert cfg.gcd(1, 0) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)

1.0 
```

因此获得的变异分数是衡量给定测试套件质量比纯覆盖率更好的指标。

## 为什么结构覆盖率不足以

结构覆盖率测量的一个问题是它未能检查测试套件生成的程序执行是否实际上是*正确的*。也就是说，一个产生错误输出但测试套件未注意到的执行与产生正确输出的执行在覆盖率中被计为完全相同。事实上，如果删除典型测试用例中的断言，新测试套件的覆盖率不会改变，但新测试套件比原始测试套件要少得多。作为一个例子，考虑这个“测试”：

```py
def ineffective_test_1():
    execute_the_program_as_a_whole()
    assert True 
```

这里的最终断言将始终通过，无论`execute_the_program_as_a_whole()`做什么。好吧，如果`execute_the_program_as_a_whole()`引发异常，测试将失败，但我们也可以绕过这一点：

```py
def ineffective_test_2():
    try:
        execute_the_program_as_a_whole()
    except:
        pass
    assert True 
```

然而，这些“测试”的问题在于`execute_the_program_as_a_whole()`可能达到 100%的代码覆盖率（或 100%的任何其他结构覆盖率指标）。然而，这个 100%的数字并不能反映测试发现错误的能力，实际上这个能力是 0%。

这确实不是一种理想的状态。我们如何验证我们的测试实际上是有用的呢？一个替代方案（在第关于覆盖率的章节中暗示）是将错误注入程序中，并评估测试套件捕捉这些注入错误的有效性。然而，这又引入了另一个问题。我们最初如何产生这些错误呢？任何手动工作都可能受到开发者对错误可能发生位置及其影响的先入为主的偏见。此外，编写好的错误可能需要花费大量时间，而收益却非常间接。因此，这种解决方案是不够的。

## 使用变异分析植入人工故障

变异分析提供了一种评估测试套件有效性的替代方案。变异分析的想法是在程序代码中植入*人工故障*，称为*变异*，并检查测试套件是否能发现它们。例如，这种变异可能是在`execute_the_program_as_a_whole()`函数中的某个地方将`+`替换为`-`。当然，上述无效的测试不会检测到这一点，因为它们没有检查任何结果。一个有效的测试将会这样做；并且假设测试在发现*人工*故障方面越有效，它在发现*真实*故障方面就越有效。

变异分析带来的洞察是从程序员的视角考虑引入错误的概率。如果假设程序中每个程序元素所获得的关注程度足够相似，那么可以进一步假设程序中的每个标记都有相似的概率被错误地转录。当然，程序员会纠正编译器（或其他静态分析工具）检测到的任何错误。因此，那些不同于原始版本并通过编译阶段的合法标记集合被认为是其可能的*变异*集合，这些变异代表了程序中的*可能故障*。然后，测试套件的判断标准是它检测（从而防止）此类变异的能力。检测到的此类变异与所有*有效*变异产生的比例被用作变异分数。在本章中，我们将看到如何在 Python 程序中实现变异分析。获得的变异分数代表了任何程序分析工具防止错误的能力，并且可以用来评估静态测试套件、测试生成器（如模糊器）、以及静态和符号执行框架。

可能会直观地考虑一个稍微不同的视角。测试套件是一个程序，可以将其视为接受要测试的程序作为输入。评估这样一个程序（测试套件）的最佳方法是什么？我们可以通过对输入程序应用小的突变来模糊测试套件，并验证所讨论的测试套件不会产生意外的行为。测试套件应该只允许原始程序通过；因此，任何未被检测为错误的突变都代表测试套件中的错误。

## 通过示例展示的结构覆盖率充分性

让我们引入一个更详细的例子来说明覆盖率的问题以及突变分析是如何工作的。下面的`triangle()`程序根据边长$a$、$b$和$c$将三角形分类到正确的三角形类别。我们希望验证程序是否正确工作。

```py
def triangle(a, b, c):
    if a == b:
        if b == c:
            return 'Equilateral'
        else:
            return 'Isosceles'
    else:
        if b == c:
            return "Isosceles"
        else:
            if a == c:
                return "Isosceles"
            else:
                return "Scalene" 
```

这里有一些测试用例来确保程序能正常工作。

```py
def strong_oracle(fn):
    assert fn(1, 1, 1) == 'Equilateral'

    assert fn(1, 2, 1) == 'Isosceles'
    assert fn(2, 2, 1) == 'Isosceles'
    assert fn(1, 2, 2) == 'Isosceles'

    assert fn(1, 2, 3) == 'Scalene' 
```

运行它们实际上会导致所有测试通过。

```py
strong_oracle(triangle) 
```

然而，“所有测试通过”的陈述只有在我们知道我们的测试是有效的时才有价值。我们的测试套件的有效性是什么？正如我们在覆盖率章节中看到的，可以使用结构覆盖率技术，如语句覆盖率，来获得测试用例有效性的度量。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from Coverage import Coverage 
```

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

我们添加了一个`show_coverage()`函数来可视化获得的覆盖率。

```py
class VisualCoverage(Coverage):
    def show_coverage(self, fn):
        src = inspect.getsource(fn)
        name = fn.__name__
        covered = set([lineno for method,
                       lineno in self._trace if method == name])
        for i, s in enumerate(src.split('\n')):
            print('%s  %2d: %s' % ('#' if i + 1 in covered else ' ', i + 1, s)) 
```

```py
with VisualCoverage() as cov:
    strong_oracle(triangle) 
```

```py
cov.show_coverage(triangle) 
```

```py
   1: def triangle(a, b, c):
#  2:     if a == b:
#  3:         if b == c:
#  4:             return 'Equilateral'
   5:         else:
#  6:             return 'Isosceles'
   7:     else:
#  8:         if b == c:
#  9:             return "Isosceles"
  10:         else:
# 11:             if a == c:
# 12:                 return "Isosceles"
  13:             else:
# 14:                 return "Scalene"
  15: 

```

我们的`strong_oracle()`似乎已经充分覆盖了所有可能的情况。也就是说，根据结构覆盖率，我们的测试用例集是相当好的。然而，获得的覆盖率是否就是全部故事？考虑这个测试套件：

```py
def weak_oracle(fn):
    assert fn(1, 1, 1) == 'Equilateral'

    assert fn(1, 2, 1) != 'Equilateral'
    assert fn(2, 2, 1) != 'Equilateral'
    assert fn(1, 2, 2) != 'Equilateral'

    assert fn(1, 2, 3) != 'Equilateral' 
```

我们在这里检查的只是具有不等边的三角形不是等边三角形。我们获得了什么样的覆盖率？

```py
with VisualCoverage() as cov:
    weak_oracle(triangle) 
```

```py
cov.show_coverage(triangle) 
```

```py
   1: def triangle(a, b, c):
#  2:     if a == b:
#  3:         if b == c:
#  4:             return 'Equilateral'
   5:         else:
#  6:             return 'Isosceles'
   7:     else:
#  8:         if b == c:
#  9:             return "Isosceles"
  10:         else:
# 11:             if a == c:
# 12:                 return "Isosceles"
  13:             else:
# 14:                 return "Scalene"
  15: 

```

事实上，似乎在覆盖率上没有*任何*差异。`weak_oracle()`获得的覆盖率与`strong_oracle()`完全相同。然而，稍加思考应该会让人相信`weak_oracle()`并不像`strong_oracle()`那样有效。然而，*覆盖率*无法区分这两个测试套件。我们在覆盖率上遗漏了什么？这里的问题是覆盖率无法评估我们断言的质量。事实上，覆盖率根本不在乎断言。然而，正如我们上面所看到的，断言是测试套件有效性的极其重要的一部分。因此，我们需要一种方法来评估断言的质量。

## 注入人工故障

注意，在 覆盖率章节 中，覆盖率被提出作为测试套件发现错误的可能性的 *代理*。如果我们实际上尝试评估测试套件发现错误的可能性怎么办？我们只需要将错误注入到程序中，一次一个，并计算我们的测试套件检测到的这种错误数量。检测频率将为我们提供测试套件发现错误的实际可能性。这种技术被称为 *故障注入*。以下是一个 *故障注入* 的例子。

```py
def triangle_m1(a, b, c):
    if a == b:
        if b == c:
            return 'Equilateral'
        else:
            # return 'Isosceles'
            return None  # <-- injected fault
    else:
        if b == c:
            return "Isosceles"
        else:
            if a == c:
                return "Isosceles"
            else:
                return "Scalene" 
```

让我们看看我们的测试套件是否足够好，能够捕捉到这个错误。我们首先检查 `weak_oracle()` 是否能够检测到这个变化。

```py
from ExpectError import ExpectError 
```

```py
with ExpectError():
    weak_oracle(triangle_m1) 
```

`weak_oracle()` 无法检测到任何变化。那么我们的 `strong_oracle()` 呢？

```py
with ExpectError():
    strong_oracle(triangle_m1) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/3191755624.py", line 2, in <module>
    strong_oracle(triangle_m1)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/566939880.py", line 5, in strong_oracle
    assert fn(2, 2, 1) == 'Isosceles'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError (expected)

```

我们的 `strong_oracle()` 能够检测到这个错误，这是 `strong_oracle()` 可能是一个更好的测试套件的证据。

*故障注入* 可以提供一个测试套件有效性的良好度量，前提是我们有一个可能的错误列表。问题是收集这样一组 *无偏* 错误相当昂贵。创建合理难以检测的良好错误很困难，而且是一个手动过程。鉴于这是一个手动过程，生成的错误将受到创建它的开发者的先入为主的偏见。即使这样的精心挑选的错误可用，它们也不太可能是详尽的，可能会错过重要的错误类别和程序的某些部分。因此，*故障注入* 不能替代覆盖率。我们能做得更好吗？

变异分析为精心挑选的错误集提供了一种替代方案。关键洞察是，如果假设程序员理解了相关的程序，那么犯的大多数错误很可能都是小的转录错误（几个标记）。编译器可能会捕获这些错误中的大多数。因此，程序中剩余的大多数错误很可能是由程序结构中某些点的小（单个标记）变化引起的（这个特定的假设被称为 *合格程序员假设* 或 *有限邻域假设*）。

那么由多个较小错误组成的较大错误呢？关键洞察在于，对于大多数这样的复杂错误，单独检测一个较小错误的测试用例很可能检测到包含它的较大复杂错误。（这个假设被称为 *耦合效应*。）

我们如何将这些假设用于实践呢？想法是简单地生成所有可能的、与原始程序不同但只经过微小变化（如单个标记变化）的 *有效* 程序变体（这些变体被称为 *突变体*）。接下来，将给定的测试套件应用于生成的每个变体。任何被测试套件检测到的突变体都被说成是被测试套件 *杀死* 的。测试套件的有效性由被杀死的突变体与生成的有效突变体的比例给出。

我们接下来实现一个简单的变异分析框架，并使用它来评估我们的测试套件。

## 变异 Python 代码

为了操作 Python 程序，我们在*抽象语法树*（AST）表示形式上工作——这是编译器和解释器在读取程序文本后工作的内部表示形式。

简而言之，我们将程序转换成树，然后*改变树的某些部分*——例如，将 `+` 运算符更改为 `-` 或相反，或将实际语句更改为不执行任何操作的 `pass` 语句。然后，可以进一步处理生成的变异树；它可以传递给 Python 解释器执行，或者我们可以*将其反解析*回文本形式。

我们首先导入 AST 操作模块。

```py
import [ast](https://docs.python.org/3/library/ast.html)
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

我们可以使用 `inspect.getsource()` 获取 Python 函数的源代码。（注意，这不适用于在其他笔记本中定义的函数。）

```py
triangle_source = inspect.getsource(triangle)
triangle_source 
```

```py
'def triangle(a, b, c):\n    if a == b:\n        if b == c:\n            return \'Equilateral\'\n        else:\n            return \'Isosceles\'\n    else:\n        if b == c:\n            return "Isosceles"\n        else:\n            if a == c:\n                return "Isosceles"\n            else:\n                return "Scalene"\n'

```

为了以视觉上令人愉悦的形式查看这些内容，我们的函数 `print_content(s, suffix)` 格式化和突出显示字符串 `s`，就像它是一个以 `suffix` 结尾的文件。因此，我们可以像查看（并突出显示）Python 文件一样查看（并突出显示）源代码：

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import print_content 
```

```py
print_content(triangle_source, '.py') 
```

```py
def triangle(a, b, c):
    if a == b:
        if b == c:
            return 'Equilateral'
        else:
            return 'Isosceles'
    else:
        if b == c:
            return "Isosceles"
        else:
            if a == c:
                return "Isosceles"
            else:
                return "Scalene"

```

解析这些语句会给我们一个抽象语法树（AST）——这是程序以树形表示的形式。

```py
triangle_ast = ast.parse(triangle_source) 
```

这个 AST 看起来是什么样子？辅助函数 `ast.dump()`（文本输出）和 `showast.show_ast()`（带有 [showast](https://github.com/hchasestevens/show_ast) 的图形输出）允许我们检查树的结构。我们看到函数以具有名称和参数的 `FunctionDef` 开始，后面是一个体，即语句列表；在这种情况下，体只包含一个 `If`，它本身包含其他类型的节点，如 `If`、`Compare`、`Name`、`Str` 和 `Return`。

```py
print(ast.dump(triangle_ast, indent=4)) 
```

```py
Module(
    body=[
        FunctionDef(
            name='triangle',
            args=arguments(
                posonlyargs=[],
                args=[
                    arg(arg='a'),
                    arg(arg='b'),
                    arg(arg='c')],
                kwonlyargs=[],
                kw_defaults=[],
                defaults=[]),
            body=[
                If(
                    test=Compare(
                        left=Name(id='a', ctx=Load()),
                        ops=[
                            Eq()],
                        comparators=[
                            Name(id='b', ctx=Load())]),
                    body=[
                        If(
                            test=Compare(
                                left=Name(id='b', ctx=Load()),
                                ops=[
                                    Eq()],
                                comparators=[
                                    Name(id='c', ctx=Load())]),
                            body=[
                                Return(
                                    value=Constant(value='Equilateral'))],
                            orelse=[
                                Return(
                                    value=Constant(value='Isosceles'))])],
                    orelse=[
                        If(
                            test=Compare(
                                left=Name(id='b', ctx=Load()),
                                ops=[
                                    Eq()],
                                comparators=[
                                    Name(id='c', ctx=Load())]),
                            body=[
                                Return(
                                    value=Constant(value='Isosceles'))],
                            orelse=[
                                If(
                                    test=Compare(
                                        left=Name(id='a', ctx=Load()),
                                        ops=[
                                            Eq()],
                                        comparators=[
                                            Name(id='c', ctx=Load())]),
                                    body=[
                                        Return(
                                            value=Constant(value='Isosceles'))],
                                    orelse=[
                                        Return(
                                            value=Constant(value='Scalene'))])])])],
            decorator_list=[],
            type_params=[])],
    type_ignores=[])

```

文字太多？这个图形表示可能使事情更简单。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import rich_output 
```

```py
if rich_output():
    import [showast](https://pypi.org/project/showast/)
    showast.show_ast(triangle_ast) 
```

<svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1791pt" height="476pt" viewBox="0.00 0.00 1791.38 476.00"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 472)"><g id="node1" class="node"><title>0</title> <text text-anchor="start" x="115.88" y="-445.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">FunctionDef</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="49.25" y="-372.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"triangle"</text></g> <g id="edge1" class="edge"><title>0--1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="start" x="124.12" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arguments</text></g> <g id="edge2" class="edge"><title>0--2</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="start" x="495" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">If</text></g> <g id="edge9" class="edge"><title>0--9</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="start" x="40.88" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arg</text></g> <g id="edge3" class="edge"><title>2--3</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="start" x="112.88" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arg</text></g> <g id="edge5" class="edge"><title>2--5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="start" x="184.88" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arg</text></g> <g id="edge7" class="edge"><title>2--7</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="53.25" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"a"</text></g> <g id="edge4" class="edge"><title>3--4</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="125.25" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"b"</text></g> <g id="edge6" class="edge"><title>5--6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="197.25" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"c"</text></g> <g id="edge8" class="edge"><title>7--8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="start" x="348.38" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge10" class="edge"><title>9--10</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="start" x="656" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">If</text></g> <g id="edge18" class="edge"><title>9--18</title></g> <g id="node34" class="node"><title>33</title> <text text-anchor="start" x="1187" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">If</text></g> <g id="edge33" class="edge"><title>9--33</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="start" x="252.75" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge11" class="edge"><title>10--11</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="341.25" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Eq</text></g> <g id="edge14" class="edge"><title>10--14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="start" x="396.75" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge15" class="edge"><title>10--15</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="197.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"a"</text></g> <g id="edge12" class="edge"><title>11--12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="269.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge13" class="edge"><title>11--13</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="341.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"b"</text></g> <g id="edge16" class="edge"><title>15--16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="413.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge17" class="edge"><title>15--17</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="start" x="564.38" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge19" class="edge"><title>18--19</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="start" x="674.5" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge27" class="edge"><title>18--27</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="start" x="799.5" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge30" class="edge"><title>18--30</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="start" x="468.75" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge20" class="edge"><title>19--20</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="557.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Eq</text></g> <g id="edge23" class="edge"><title>19--23</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="start" x="612.75" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge24" class="edge"><title>19--24</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="413.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"b"</text></g> <g id="edge21" class="edge"><title>20--21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="485.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge22" class="edge"><title>20--22</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="557.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"c"</text></g> <g id="edge25" class="edge"><title>24--25</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="629.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge26" class="edge"><title>24--26</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="start" x="693.25" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge28" class="edge"><title>27--28</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="736.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"Equilateral"</text></g> <g id="edge29" class="edge"><title>28--29</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="start" x="815.25" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge31" class="edge"><title>30--31</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="869.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"Isosceles"</text></g> <g id="edge32" class="edge"><title>31--32</title></g> <g id="node35" class="node"><title>34</title> <text text-anchor="start" x="1102.38" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge34" class="edge"><title>33--34</title></g> <g id="node43" class="node"><title>42</title> <text text-anchor="start" x="1209.5" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge42" class="edge"><title>33--42</title></g> <g id="node46" class="node"><title>45</title> <text text-anchor="start" x="1521" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">If</text></g> <g id="edge45" class="edge"><title>33--45</title></g> <g id="node36" class="node"><title>35</title> <text text-anchor="start" x="1022.75" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge35" class="edge"><title>34--35</title></g> <g id="node39" class="node"><title>38</title> <text text-anchor="middle" x="1111.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Eq</text></g> <g id="edge38" class="edge"><title>34--38</title></g> <g id="node40" class="node"><title>39</title> <text text-anchor="start" x="1166.75" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge39" class="edge"><title>34--39</title></g> <g id="node37" class="node"><title>36</title> <text text-anchor="middle" x="967.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"b"</text></g> <g id="edge36" class="edge"><title>35--36</title></g> <g id="node38" class="node"><title>37</title> <text text-anchor="middle" x="1039.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge37" class="edge"><title>35--37</title></g> <g id="node41" class="node"><title>40</title> <text text-anchor="middle" x="1111.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"c"</text></g> <g id="edge40" class="edge"><title>39--40</title></g> <g id="node42" class="node"><title>41</title> <text text-anchor="middle" x="1183.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge41" class="edge"><title>39--41</title></g> <g id="node44" class="node"><title>43</title> <text text-anchor="start" x="1242.25" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge43" class="edge"><title>42--43</title></g> <g id="node45" class="node"><title>44</title> <text text-anchor="middle" x="1281.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"Isosceles"</text></g> <g id="edge44" class="edge"><title>43--44</title></g> <g id="node47" class="node"><title>46</title> <text text-anchor="start" x="1436.38" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge46" class="edge"><title>45--46</title></g> <g id="node55" class="node"><title>54</title> <text text-anchor="start" x="1569.5" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge54" class="edge"><title>45--54</title></g> <g id="node58" class="node"><title>57</title> <text text-anchor="start" x="1690.5" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge57" class="edge"><title>45--57</title></g> <g id="node48" class="node"><title>47</title> <text text-anchor="start" x="1362.75" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge47" class="edge"><title>46--47</title></g> <g id="node51" class="node"><title>50</title> <text text-anchor="middle" x="1451.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Eq</text></g> <g id="edge50" class="edge"><title>46--50</title></g> <g id="node52" class="node"><title>51</title> <text text-anchor="start" x="1506.75" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge51" class="edge"><title>46--51</title></g> <g id="node49" class="node"><title>48</title> <text text-anchor="middle" x="1307.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"a"</text></g> <g id="edge48" class="edge"><title>47--48</title></g> <g id="node50" class="node"><title>49</title> <text text-anchor="middle" x="1379.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge49" class="edge"><title>47--49</title></g> <g id="node53" class="node"><title>52</title> <text text-anchor="middle" x="1451.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"c"</text></g> <g id="edge52" class="edge"><title>51--52</title></g> <g id="node54" class="node"><title>53</title> <text text-anchor="middle" x="1523.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge53" class="edge"><title>51--53</title></g> <g id="node56" class="node"><title>55</title> <text text-anchor="start" x="1582.25" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge55" class="edge"><title>54--55</title></g> <g id="node57" class="node"><title>56</title> <text text-anchor="middle" x="1621.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"Isosceles"</text></g> <g id="edge56" class="edge"><title>55--56</title></g> <g id="node59" class="node"><title>58</title> <text text-anchor="start" x="1697.25" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge58" class="edge"><title>57--58</title></g> <g id="node60" class="node"><title>59</title> <text text-anchor="middle" x="1738.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"Scalene"</text></g> <g id="edge59" class="edge"><title>58--59</title></g></g></svg>

函数 `ast.unparse()` 将这样的树转换回更熟悉的文本 Python 代码表示形式。

```py
print_content(ast.unparse(triangle_ast), '.py') 
```

```py
def triangle(a, b, c):
    if a == b:
        if b == c:
            return 'Equilateral'
        else:
            return 'Isosceles'
    elif b == c:
        return 'Isosceles'
    elif a == c:
        return 'Isosceles'
    else:
        return 'Scalene'

```

## 函数的简单变异器

让我们现在去修改 `triangle()` 程序。产生这个程序有效变异版本的一个简单方法是将其中的一些语句替换为 `pass`。

`MuFunctionAnalyzer` 是负责测试套件变异分析的主要类。它接受要测试的函数。它通过解析和反解析一次提供的源代码来规范化源代码。这是确保后续的原始代码和变异代码之间的 `diff` 不受空白、注释等差异的影响所必需的。

```py
class MuFunctionAnalyzer:
    def __init__(self, fn, log=False):
        self.fn = fn
        self.name = fn.__name__
        src = inspect.getsource(fn)
        self.ast = ast.parse(src)
        self.src = ast.unparse(self.ast)  # normalize
        self.mutator = self.mutator_object()
        self.nmutations = self.get_mutation_count()
        self.un_detected = set()
        self.mutants = []
        self.log = log

    def mutator_object(self, locations=None):
        return StmtDeletionMutator(locations)

    def register(self, m):
        self.mutants.append(m)

    def finish(self):
        pass 
```

`get_mutation_count()` 获取可用的变异数量。我们稍后会看到如何实现这一点。

```py
class MuFunctionAnalyzer(MuFunctionAnalyzer):
    def get_mutation_count(self):
        self.mutator.visit(self.ast)
        return self.mutator.count 
```

`Mutator` 为实现单个突变提供了基类。它接受一个要突变的地点列表。它假设子类确定的所有感兴趣节点的 `mutable_visit()` 方法被调用。当 `Mutator` 在没有要突变的地点列表的情况下被调用时，它简单地遍历所有可能的突变点，并在 `self.count` 中保留计数。如果它被调用并带有特定的突变地点列表，则 `mutable_visit()` 方法调用 `mutation_visit()`，该方法在节点上执行突变。请注意，单个位置可以产生多个突变。（因此有哈希表）。

```py
class Mutator(ast.NodeTransformer):
    def __init__(self, mutate_location=-1):
        self.count = 0
        self.mutate_location = mutate_location

    def mutable_visit(self, node):
        self.count += 1  # statements start at line no 1
        if self.count == self.mutate_location:
            return self.mutation_visit(node)
        return self.generic_visit(node) 
```

`StmtDeletionMutator` 简单地钩入所有语句处理访问者。它通过将给定的语句替换为 `pass` 来执行突变。正如你所见，它访问了所有类型的语句。

```py
class StmtDeletionMutator(Mutator):
    def visit_Return(self, node): return self.mutable_visit(node)
    def visit_Delete(self, node): return self.mutable_visit(node)

    def visit_Assign(self, node): return self.mutable_visit(node)
    def visit_AnnAssign(self, node): return self.mutable_visit(node)
    def visit_AugAssign(self, node): return self.mutable_visit(node)

    def visit_Raise(self, node): return self.mutable_visit(node)
    def visit_Assert(self, node): return self.mutable_visit(node)

    def visit_Global(self, node): return self.mutable_visit(node)
    def visit_Nonlocal(self, node): return self.mutable_visit(node)

    def visit_Expr(self, node): return self.mutable_visit(node)

    def visit_Pass(self, node): return self.mutable_visit(node)
    def visit_Break(self, node): return self.mutable_visit(node)
    def visit_Continue(self, node): return self.mutable_visit(node) 
```

实际的突变包括用 `pass` 语句替换节点：

```py
class StmtDeletionMutator(StmtDeletionMutator):
    def mutation_visit(self, node): return ast.Pass() 
```

对于 `triangle()`，这个访问者产生了五个突变——即用 `pass` 替换五个 `return` 语句：

```py
MuFunctionAnalyzer(triangle).nmutations 
```

```py
5

```

我们需要一种方法来获取单个突变体。为此，我们将我们的 `MuFunctionAnalyzer` 转换为一个 *可迭代对象*。

```py
class MuFunctionAnalyzer(MuFunctionAnalyzer):
    def __iter__(self):
        return PMIterator(self) 
```

`PMIterator` 是 `MuFunctionAnalyzer` 的 *迭代器* 类，定义如下。

```py
class PMIterator:
    def __init__(self, pm):
        self.pm = pm
        self.idx = 0 
```

`next()` 方法返回相应的 `Mutant`：

```py
class PMIterator(PMIterator):
    def __next__(self):
        i = self.idx
        if i >= self.pm.nmutations:
            self.pm.finish()
            raise StopIteration()
        self.idx += 1
        mutant = Mutant(self.pm, self.idx, log=self.pm.log)
        self.pm.register(mutant)
        return mutant 
```

`Mutant` 类包含在给定突变位置时生成突变体的逻辑。

```py
class Mutant:
    def __init__(self, pm, location, log=False):
        self.pm = pm
        self.i = location
        self.name = "%s_%s" % (self.pm.name, self.i)
        self._src = None
        self.tests = []
        self.detected = False
        self.log = log 
```

这里是如何使用它的：

```py
for m in MuFunctionAnalyzer(triangle):
    print(m.name) 
```

```py
triangle_1
triangle_2
triangle_3
triangle_4
triangle_5

```

这些名称有点通用。让我们看看我们是否能对产生的突变有更多的了解。

`generate_mutant()` 简单地调用 `mutator()` 方法，并将 AST 的副本传递给突变器。

```py
class Mutant(Mutant):
    def generate_mutant(self, location):
        mutant_ast = self.pm.mutator_object(
            location).visit(ast.parse(self.pm.src))  # copy
        return ast.unparse(mutant_ast) 
```

`src()` 方法返回突变后的源代码。

```py
class Mutant(Mutant):
    def src(self):
        if self._src is None:
            self._src = self.generate_mutant(self.i)
        return self._src 
```

这里是如何获取突变体以及可视化与原始代码的差异：

```py
import [difflib](https://docs.python.org/3/library/difflib.html) 
```

```py
for mutant in MuFunctionAnalyzer(triangle):
    shape_src = mutant.pm.src
    for line in difflib.unified_diff(mutant.pm.src.split('\n'),
                                     mutant.src().split('\n'),
                                     fromfile=mutant.pm.name,
                                     tofile=mutant.name, n=3):
        print(line) 
```

```py
--- triangle

+++ triangle_1

@@ -1,7 +1,7 @@

 def triangle(a, b, c):
     if a == b:
         if b == c:
-            return 'Equilateral'
+            pass
         else:
             return 'Isosceles'
     elif b == c:
--- triangle

+++ triangle_2

@@ -3,7 +3,7 @@

         if b == c:
             return 'Equilateral'
         else:
-            return 'Isosceles'
+            pass
     elif b == c:
         return 'Isosceles'
     elif a == c:
--- triangle

+++ triangle_3

@@ -5,7 +5,7 @@

         else:
             return 'Isosceles'
     elif b == c:
-        return 'Isosceles'
+        pass
     elif a == c:
         return 'Isosceles'
     else:
--- triangle

+++ triangle_4

@@ -7,6 +7,6 @@

     elif b == c:
         return 'Isosceles'
     elif a == c:
-        return 'Isosceles'
+        pass
     else:
         return 'Scalene'
--- triangle

+++ triangle_5

@@ -9,4 +9,4 @@

     elif a == c:
         return 'Isosceles'
     else:
-        return 'Scalene'
+        pass

```

在这个 `diff` 输出中，以 `+` 前缀的行是添加的，而以 `-` 前缀的行是删除的。我们看到五个突变体确实用 `pass` 语句替换了每个返回语句。

我们向 `Mutant` 添加了 `diff()` 方法，以便可以直接调用它。

```py
class Mutant(Mutant):
    def diff(self):
        return '\n'.join(difflib.unified_diff(self.pm.src.split('\n'),
                                              self.src().split('\n'),
                                              fromfile='original',
                                              tofile='mutant',
                                              n=3)) 
```

## 评估突变

我们现在准备好实现实际的评估。我们定义我们的突变体作为一个 *上下文管理器*，以验证所有给出的断言是否成功。想法是我们可以编写如下代码

```py
for mutant in MuFunctionAnalyzer(function):
    with mutant:
        assert function(x) == y 
```

当 `mutant` 激活时（即 `with:` 下的代码块），原始函数被替换为突变函数。

`__enter__()` 函数在进入 `with` 块时被调用。它创建突变体作为一个 Python 函数，并将其放置在全局命名空间中，这样 `assert` 语句就执行突变函数而不是原始函数。

```py
class Mutant(Mutant):
    def __enter__(self):
        if self.log:
            print('->\t%s' % self.name)
        c = compile(self.src(), '<mutant>', 'exec')
        eval(c, globals()) 
```

`__exit__()` 函数检查是否发生了异常（即断言失败，或引发了其他错误）；如果是这样，它将突变标记为 `detected`。最后，它恢复原始函数定义。

```py
class Mutant(Mutant):
    def __exit__(self, exc_type, exc_value, traceback):
        if self.log:
            print('<-\t%s' % self.name)
        if exc_type is not None:
            self.detected = True
            if self.log:
                print("Detected %s" % self.name, exc_type, exc_value)
        globals()[self.pm.name] = self.pm.fn
        if self.log:
            print()
        return True 
```

`finish()` 方法简单地调用突变体上的方法，检查突变体是否被发现，并返回结果。

```py
from ExpectError import ExpectTimeout 
```

```py
class MuFunctionAnalyzer(MuFunctionAnalyzer):
    def finish(self):
        self.un_detected = {
            mutant for mutant in self.mutants if not mutant.detected} 
```

突变分数——测试套件检测到的突变体的比率——通过 `score()` 计算。1.0 的分数意味着所有突变体都被发现；0.1 的分数意味着只有 10% 的突变体被检测到。

```py
class MuFunctionAnalyzer(MuFunctionAnalyzer):
    def score(self):
        return (self.nmutations - len(self.un_detected)) / self.nmutations 
```

这是我们的框架的使用方法。

```py
import [sys](https://docs.python.org/3/library/sys.html) 
```

```py
for mutant in MuFunctionAnalyzer(triangle, log=True):
    with mutant:
        assert triangle(1, 1, 1) == 'Equilateral', "Equal Check1"
        assert triangle(1, 0, 1) != 'Equilateral', "Equal Check2"
        assert triangle(1, 0, 2) != 'Equilateral', "Equal Check3"
mutant.pm.score() 
```

```py
->	triangle_1
<-	triangle_1
Detected triangle_1 <class 'AssertionError'> Equal Check1

->	triangle_2
<-	triangle_2

->	triangle_3
<-	triangle_3

->	triangle_4
<-	triangle_4

->	triangle_5
<-	triangle_5

```

```py
0.2

```

五个突变中只有一个导致了失败的断言。因此，`weak_oracle()` 测试套件的突变分数为 20%。

```py
for mutant in MuFunctionAnalyzer(triangle):
    with mutant:
        weak_oracle(triangle)
mutant.pm.score() 
```

```py
0.2

```

由于我们正在修改全局命名空间，我们不需要在突变体的 for 循环中直接引用该函数。

```py
def oracle():
    strong_oracle(triangle) 
```

```py
for mutant in MuFunctionAnalyzer(triangle, log=True):
    with mutant:
        oracle()
mutant.pm.score() 
```

```py
->	triangle_1
<-	triangle_1
Detected triangle_1 <class 'AssertionError'> 

->	triangle_2
<-	triangle_2
Detected triangle_2 <class 'AssertionError'> 

->	triangle_3
<-	triangle_3
Detected triangle_3 <class 'AssertionError'> 

->	triangle_4
<-	triangle_4
Detected triangle_4 <class 'AssertionError'> 

->	triangle_5
<-	triangle_5
Detected triangle_5 <class 'AssertionError'> 

```

```py
1.0

```

即，我们能够通过 `strong_oracle()` 测试套件实现 `100%` 的突变分数。

这里是另一个例子。`gcd()` 计算两个数的最大公约数。

```py
def gcd(a, b):
    if a < b:
        c = a
        a = b
        b = c

    while b != 0:
        c = a
        a = b
        b = c % b

    return a 
```

这里是对它的一个测试。它有多有效？

```py
for mutant in MuFunctionAnalyzer(gcd, log=True):
    with mutant:
        assert gcd(1, 0) == 1, "Minimal"
        assert gcd(0, 1) == 1, "Mirror" 
```

```py
->	gcd_1
<-	gcd_1
Detected gcd_1 <class 'UnboundLocalError'> cannot access local variable 'c' where it is not associated with a value

->	gcd_2
<-	gcd_2
Detected gcd_2 <class 'AssertionError'> Mirror

->	gcd_3
<-	gcd_3

->	gcd_4
<-	gcd_4

->	gcd_5
<-	gcd_5

->	gcd_6
<-	gcd_6

->	gcd_7
<-	gcd_7
Detected gcd_7 <class 'AssertionError'> Minimal

```

```py
mutant.pm.score() 
```

```py
0.42857142857142855

```

我们看到我们的 `TestGCD` 测试套件能够获得 42% 的突变分数。

## 模块和测试套件的突变器

考虑我们之前讨论的 `triangle()` 程序。正如我们讨论的那样，产生这个程序有效突变版本的一个简单方法是将其中的一些语句替换为 `pass`。

为了演示目的，我们希望像程序在另一个文件中一样进行操作。我们可以通过在 Python 中创建一个 `Module` 对象并将函数附加到它上来实现这一点。

```py
import [types](https://docs.python.org/3/library/types.html) 
```

```py
def import_code(code, name):
    module = types.ModuleType(name)
    exec(code, module.__dict__)
    return module 
```

我们将 `triangle()` 函数附加到 `shape` 模块。

```py
shape = import_code(shape_src, 'shape') 
```

我们现在可以通过 `shape` 模块调用三角形。

```py
shape.triangle(1, 1, 1) 
```

```py
'Equilateral'

```

我们想测试 `triangle()` 函数。为此，我们定义了一个如下的 `StrongShapeTest` 类。

```py
import [unittest](https://docs.python.org/3/library/unittest.html) 
```

```py
class StrongShapeTest(unittest.TestCase):

    def test_equilateral(self):
        assert shape.triangle(1, 1, 1) == 'Equilateral'

    def test_isosceles(self):
        assert shape.triangle(1, 2, 1) == 'Isosceles'
        assert shape.triangle(2, 2, 1) == 'Isosceles'
        assert shape.triangle(1, 2, 2) == 'Isosceles'

    def test_scalene(self):
        assert shape.triangle(1, 2, 3) == 'Scalene' 
```

我们定义了一个辅助函数 `suite()`，该函数遍历给定的类并识别测试函数。

```py
def suite(test_class):
    suite = unittest.TestSuite()
    for f in test_class.__dict__:
        if f.startswith('test_'):
            suite.addTest(test_class(f))
    return suite 
```

`TestTriangle` 类的测试可以通过不同的测试运行器调用。最简单的方法是直接调用 `TestCase` 的 `run()` 方法。

```py
suite(StrongShapeTest).run(unittest.TestResult()) 
```

```py
<unittest.result.TestResult run=3 errors=0 failures=0>

```

`TextTestRunner` 类提供了控制执行详细程度的能力。它还允许在 *第一次* 失败时返回。

```py
runner = unittest.TextTestRunner(verbosity=0, failfast=True)
runner.run(suite(StrongShapeTest)) 
```

```py
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK

```

```py
<unittest.runner.TextTestResult run=3 errors=0 failures=0>

```

在覆盖率下运行程序如下：

```py
with VisualCoverage() as cov:
    suite(StrongShapeTest).run(unittest.TestResult()) 
```

获得的覆盖率如下：

```py
cov.show_coverage(triangle) 
```

```py
   1: def triangle(a, b, c):
#  2:     if a == b:
#  3:         if b == c:
#  4:             return 'Equilateral'
   5:         else:
#  6:             return 'Isosceles'
#  7:     else:
#  8:         if b == c:
#  9:             return "Isosceles"
# 10:         else:
  11:             if a == c:
# 12:                 return "Isosceles"
  13:             else:
  14:                 return "Scalene"
  15: 

```

```py
class WeakShapeTest(unittest.TestCase):
    def test_equilateral(self):
        assert shape.triangle(1, 1, 1) == 'Equilateral'

    def test_isosceles(self):
        assert shape.triangle(1, 2, 1) != 'Equilateral'
        assert shape.triangle(2, 2, 1) != 'Equilateral'
        assert shape.triangle(1, 2, 2) != 'Equilateral'

    def test_scalene(self):
        assert shape.triangle(1, 2, 3) != 'Equilateral' 
```

它获得了多少覆盖率？

```py
with VisualCoverage() as cov:
    suite(WeakShapeTest).run(unittest.TestResult()) 
```

```py
cov.show_coverage(triangle) 
```

```py
   1: def triangle(a, b, c):
#  2:     if a == b:
#  3:         if b == c:
#  4:             return 'Equilateral'
   5:         else:
#  6:             return 'Isosceles'
#  7:     else:
#  8:         if b == c:
#  9:             return "Isosceles"
# 10:         else:
  11:             if a == c:
# 12:                 return "Isosceles"
  13:             else:
  14:                 return "Scalene"
  15: 

```

`MuProgramAnalyzer` 是负责测试套件突变分析的主要类。它接受要测试的模块名称及其源代码。它通过解析和重新解析一次给定的源代码来规范化源代码。这是确保后续的原始和突变体之间的 `diff` 不受空白、注释等差异的影响所必需的。

```py
class MuProgramAnalyzer(MuFunctionAnalyzer):
    def __init__(self, name, src):
        self.name = name
        self.ast = ast.parse(src)
        self.src = ast.unparse(self.ast)
        self.changes = []
        self.mutator = self.mutator_object()
        self.nmutations = self.get_mutation_count()
        self.un_detected = set()

    def mutator_object(self, locations=None):
        return AdvStmtDeletionMutator(self, locations) 
```

我们现在扩展 `Mutator` 类。

```py
class AdvMutator(Mutator):
    def __init__(self, analyzer, mutate_locations=None):
        self.count = 0
        self.mutate_locations = [] if mutate_locations is None else mutate_locations
        self.pm = analyzer

    def mutable_visit(self, node):
        self.count += 1  # statements start at line no 1
        return self.mutation_visit(node) 
```

`AdvStmtDeletionMutator` 简单地钩入所有语句处理访问者。它通过用 `pass` 替换给定的语句来进行突变。

```py
class AdvStmtDeletionMutator(AdvMutator, StmtDeletionMutator):
    def __init__(self, analyzer, mutate_locations=None):
        AdvMutator.__init__(self, analyzer, mutate_locations)

    def mutation_visit(self, node):
        index = 0  # there is only one way to delete a statement -- replace it by pass
        if not self.mutate_locations:  # counting pass
            self.pm.changes.append((self.count, index))
            return self.generic_visit(node)
        else:
            # get matching changes for this pass
            mutating_lines = set((count, idx)
                                 for (count, idx) in self.mutate_locations)
            if (self.count, index) in mutating_lines:
                return ast.Pass()
            else:
                return self.generic_visit(node) 
```

再次，我们可以获得 `triangle()` 产生的突变数量如下。

```py
MuProgramAnalyzer('shape', shape_src).nmutations 
```

```py
5

```

我们需要一种方法来获取单个突变体。为此，我们将我们的 `MuProgramAnalyzer` 转换为 *可迭代*。

```py
class MuProgramAnalyzer(MuProgramAnalyzer):
    def __iter__(self):
        return AdvPMIterator(self) 
```

`AdvPMIterator` 是 `MuProgramAnalyzer` 的 *迭代器* 类，定义如下。

```py
class AdvPMIterator:
    def __init__(self, pm):
        self.pm = pm
        self.idx = 0 
```

`next()` 方法返回相应的 `Mutant`。

```py
class AdvPMIterator(AdvPMIterator):
    def __next__(self):
        i = self.idx
        if i >= len(self.pm.changes):
            raise StopIteration()
        self.idx += 1
        # there could be multiple changes in one mutant
        return AdvMutant(self.pm, [self.pm.changes[i]]) 
```

`Mutant` 类包含在给定突变位置时生成突变体的逻辑。

```py
class AdvMutant(Mutant):
    def __init__(self, pm, locations):
        self.pm = pm
        self.i = locations
        self.name = "%s_%s" % (self.pm.name,
                               '_'.join([str(i) for i in self.i]))
        self._src = None 
```

这是它的用法：

```py
shape_src = inspect.getsource(triangle) 
```

```py
for m in MuProgramAnalyzer('shape', shape_src):
    print(m.name) 
```

```py
shape_(1, 0)
shape_(2, 0)
shape_(3, 0)
shape_(4, 0)
shape_(5, 0)

```

`generate_mutant()`函数简单地调用`mutator()`方法，并将 AST 的副本传递给突变器。

```py
class AdvMutant(AdvMutant):
    def generate_mutant(self, locations):
        mutant_ast = self.pm.mutator_object(
            locations).visit(ast.parse(self.pm.src))  # copy
        return ast.unparse(mutant_ast) 
```

`src()`方法返回突变后的源代码。

```py
class AdvMutant(AdvMutant):
    def src(self):
        if self._src is None:
            self._src = self.generate_mutant(self.i)
        return self._src 
```

再次，我们将突变体表示为与原始版本的差异：

```py
import [difflib](https://docs.python.org/3/library/difflib.html) 
```

我们向`Mutant`类添加了`diff()`方法，以便可以直接调用。

```py
class AdvMutant(AdvMutant):
    def diff(self):
        return '\n'.join(difflib.unified_diff(self.pm.src.split('\n'),
                                              self.src().split('\n'),
                                              fromfile='original',
                                              tofile='mutant',
                                              n=3)) 
```

```py
for mutant in MuProgramAnalyzer('shape', shape_src):
    print(mutant.name)
    print(mutant.diff())
    break 
```

```py
shape_(1, 0)
--- original

+++ mutant

@@ -1,7 +1,7 @@

 def triangle(a, b, c):
     if a == b:
         if b == c:
-            return 'Equilateral'
+            pass
         else:
             return 'Isosceles'
     elif b == c:

```

我们现在准备实施实际的评估。为此，我们需要能够接受定义测试套件的模块，并在其上调用测试方法。`getitem`方法接受测试模块，将测试模块上的导入条目固定为正确指向突变体模块，并将其传递给测试运行器`MutantTestRunner`。

```py
class AdvMutant(AdvMutant):
    def __getitem__(self, test_module):
        test_module.__dict__[
            self.pm.name] = import_code(
            self.src(), self.pm.name)
        return MutantTestRunner(self, test_module) 
```

`MutantTestRunner`简单地调用测试模块上的所有`test_`方法，检查突变体是否被发现，并返回结果。

```py
from ExpectError import ExpectTimeout 
```

```py
class MutantTestRunner:
    def __init__(self, mutant, test_module):
        self.mutant = mutant
        self.tm = test_module

    def runTest(self, tc):
        suite = unittest.TestSuite()
        test_class = self.tm.__dict__[tc]
        for f in test_class.__dict__:
            if f.startswith('test_'):
                suite.addTest(test_class(f))
        runner = unittest.TextTestRunner(verbosity=0, failfast=True)
        try:
            with ExpectTimeout(1):
                res = runner.run(suite)
                if res.wasSuccessful():
                    self.mutant.pm.un_detected.add(self)
                return res
        except SyntaxError:
            print('Syntax Error (%s)' % self.mutant.name)
            return None
        raise Exception('Unhandled exception during test execution') 
```

突变分数是通过`score()`函数计算的。

```py
class MuProgramAnalyzer(MuProgramAnalyzer):
    def score(self):
        return (self.nmutations - len(self.un_detected)) / self.nmutations 
```

下面是如何使用我们的框架。

```py
import [sys](https://docs.python.org/3/library/sys.html) 
```

```py
test_module = sys.modules[__name__]
for mutant in MuProgramAnalyzer('shape', shape_src):
    mutant[test_module].runTest('WeakShapeTest')
mutant.pm.score() 
```

```py
======================================================================
FAIL: test_equilateral (__main__.WeakShapeTest.test_equilateral)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/511514204.py", line 3, in test_equilateral
    assert shape.triangle(1, 1, 1) == 'Equilateral'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK

```

```py
0.2

```

`WeakShape`测试套件只产生了`20%`的突变分数。

```py
for mutant in MuProgramAnalyzer('shape', shape_src):
    mutant[test_module].runTest('StrongShapeTest')
mutant.pm.score() 
```

```py
======================================================================
FAIL: test_equilateral (__main__.StrongShapeTest.test_equilateral)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2670057129.py", line 4, in test_equilateral
    assert shape.triangle(1, 1, 1) == 'Equilateral'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_isosceles (__main__.StrongShapeTest.test_isosceles)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2670057129.py", line 8, in test_isosceles
    assert shape.triangle(2, 2, 1) == 'Isosceles'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_isosceles (__main__.StrongShapeTest.test_isosceles)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2670057129.py", line 9, in test_isosceles
    assert shape.triangle(1, 2, 2) == 'Isosceles'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_isosceles (__main__.StrongShapeTest.test_isosceles)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2670057129.py", line 7, in test_isosceles
    assert shape.triangle(1, 2, 1) == 'Isosceles'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (failures=1)
======================================================================
FAIL: test_scalene (__main__.StrongShapeTest.test_scalene)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2670057129.py", line 12, in test_scalene
    assert shape.triangle(1, 2, 3) == 'Scalene'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 3 tests in 0.000s

FAILED (failures=1)

```

```py
1.0

```

另一方面，我们能够使用`StrongShapeTest`测试套件实现`100%`的突变分数。

这里是另一个例子，`gcd()`。

```py
gcd_src = inspect.getsource(gcd) 
```

```py
class TestGCD(unittest.TestCase):
    def test_simple(self):
        assert cfg.gcd(1, 0) == 1

    def test_mirror(self):
        assert cfg.gcd(0, 1) == 1 
```

```py
for mutant in MuProgramAnalyzer('cfg', gcd_src):
    mutant[test_module].runTest('TestGCD')
mutant.pm.score() 
```

```py
======================================================================
ERROR: test_mirror (__main__.TestGCD.test_mirror)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 6, in test_mirror
    assert cfg.gcd(0, 1) == 1
           ^^^^^^^^^^^^^
  File "<string>", line 5, in gcd
UnboundLocalError: cannot access local variable 'c' where it is not associated with a value

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (errors=1)
======================================================================
FAIL: test_mirror (__main__.TestGCD.test_mirror)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 6, in test_mirror
    assert cfg.gcd(0, 1) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (failures=1)
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
======================================================================
FAIL: test_simple (__main__.TestGCD.test_simple)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_7987/2565918356.py", line 3, in test_simple
    assert cfg.gcd(1, 0) == 1
           ^^^^^^^^^^^^^^^^^^
AssertionError

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)

```

```py
0.42857142857142855

```

我们看到我们的`TestGCD`测试套件能够获得`42%`的突变分数。

## 等价突变体的问题

突变分析的一个问题在于，并非所有生成的突变体都需要是错误的。例如，考虑下面的`new_gcd()`程序。

```py
def new_gcd(a, b):
    if a < b:
        a, b = b, a
    else:
        a, b = a, b

    while b != 0:
        a, b = b, a % b
    return a 
```

这个程序可以被突变以产生以下突变体。

```py
def mutated_gcd(a, b):
    if a < b:
        a, b = b, a
    else:
        pass

    while b != 0:
        a, b = b, a % b
    return a 
```

```py
for i, mutant in enumerate(MuFunctionAnalyzer(new_gcd)):
    print(i,mutant.src()) 
```

```py
0 def new_gcd(a, b):
    if a < b:
        pass
    else:
        a, b = (a, b)
    while b != 0:
        a, b = (b, a % b)
    return a
1 def new_gcd(a, b):
    if a < b:
        a, b = (b, a)
    else:
        pass
    while b != 0:
        a, b = (b, a % b)
    return a
2 def new_gcd(a, b):
    if a < b:
        a, b = (b, a)
    else:
        a, b = (a, b)
    while b != 0:
        pass
    return a
3 def new_gcd(a, b):
    if a < b:
        a, b = (b, a)
    else:
        a, b = (a, b)
    while b != 0:
        a, b = (b, a % b)
    pass

```

与原始版本相比，其他突变体可能是错误的，但`mutant 1`在语义上与原始版本不可区分，因为它删除了一个无关紧要的赋值。这意味着`mutant 1`不代表错误。这类不代表错误的突变体被称为*等价突变体*。等价突变体的问题在于，在存在等价突变体的情况下，判断突变分数变得非常困难。例如，在 70%的突变分数下，从 0 到 30%的突变体可能是等价的。因此，不知道实际等价突变体的数量，就无法判断测试可以改进多少。我们讨论了两种处理等价突变体的方法。

### 等价突变体数量的统计估计

如果存活的突变体数量足够少，人们可能可以简单地手动检查它们。然而，如果突变体的数量足够大（比如说 > 1000），人们可以从存活的突变体中随机选择较少的数量并手动评估它们，以查看它们是否代表错误。样本大小的确定由以下二项分布公式（通过正态分布近似）控制：

$$ n \ge \hat{p}(1-\hat{p})\bigg(\frac{Z_{\frac{\alpha}{2}}}{\Delta}\bigg)² $$

其中 $n$ 是样本数量，$p$ 是概率分布的参数，$\alpha$ 是所需的精度，$\Delta$ 是精度。对于 95% 的精度，$Z_{0.95}=1.96$。我们有以下值（$\hat{p}(1-\hat{p})$ 的最大值为 0.25）和 $Z$ 是正态分布的临界值：

$$ n \ge 0.25\bigg(\frac{1.96}{\Delta}\bigg)² $$

对于 $\Delta = 0.01$（即最大误差为 1%），我们需要评估 $9604$ 个突变体以确定等效性。如果将约束放宽到 $\Delta = 0.1$（即误差为 10%），那么只需要评估 $96$ 个突变体以确定等效性。

### Chao 估计器对不死突变体数量的统计估计

虽然仅采样有限数量的突变体的想法很有吸引力，但它仍然有限，因为需要手动分析。如果计算能力便宜，另一种通过 Chao 估计器估计真实突变体数量（因此是等效突变体数量）的方法是通过 Chao 估计器。正如我们将在关于 何时停止模糊测试 的章节中看到的那样，公式如下：

$$ \hat S_\text{Chao1} = \begin{cases} S(n) + \frac{f_1²}{2f_2} & \text{if } f_2>0\\ S(n) + \frac{f_1(f_1-1)}{2} & \text{otherwise} \end{cases} $$

基本思想是计算每个测试对每个突变体的完整测试矩阵 $T \times M$ 的结果。变量 $f_1$ 表示恰好被杀死一次的突变体数量，而变量 $f_2$ 表示恰好被杀死两次的变量数量。$S(n)$ 是被杀死的突变体总数。在这里，$\hat{S}_{Chao1}$ 提供了真实突变体数量的估计。如果 $M$ 是生成的总突变体数量，那么 $M - \hat{S}_{Chao1}$ 表示 **不死** 突变体的数量。请注意，这些 **不死** 突变体与传统等效突变体有些不同，因为 **死亡率** 取决于用于区分变异行为的预言者。也就是说，如果使用依赖于抛出错误来检测杀死的模糊器，它将无法检测出产生不同输出但不抛出错误的突变体。因此，*Chao1* 估计将基本上是模糊器在给定无限时间的情况下可以检测到的突变体的渐近值。当使用的预言者足够强大时，**不死** 突变体估计将接近真实的 **等效** 突变体估计。更多细节请参阅关于 何时停止模糊测试 的章节。测试中物种发现的全面指南是 Boehme 等人于 2018 年发表的论文 [[Böhme 等人，2018](https://doi.org/10.1145/3210309)]。

## 经验教训

+   我们已经了解到为什么结构覆盖率不足以评估测试套件的质量。

+   我们已经了解到如何使用突变分析来评估测试套件的质量。

+   我们已经了解到突变分析的局限性——等效和冗余突变体，以及如何估计它们。

## 下一步

+   虽然简单的模糊测试生成质量较差的预言机，但诸如符号和条件等技术可以提高模糊测试中使用的预言机的质量。

+   动态不变量也可以在提高预言机质量方面大有帮助。

+   关于何时停止模糊测试的章节提供了 Chao 估计器的详细概述。

## 背景

突变分析的想法最初由 Lipton 等人提出[Lipton *et al*, 1971]。Jia 等人发表了一篇关于突变分析研究的优秀调查[Jia *et al*, 2011]。Papadakis 等人关于突变分析的章节[Papadakis *et al*, 2019]也是对当前突变分析趋势的另一个优秀概述。

## 练习

### 练习 1：算术表达式突变器

我们简单的语句删除突变只是程序可能突变的一种方式。另一类突变体是*表达式突变*，其中算术运算符（如`{+,-,*,/}`等）相互替换。例如，给定一个表达式如下

```py
x = x + 1
```

可以将其突变成

```py
x = x - 1
```

和

```py
x = x * 1
```

和

```py
x = x / 1
```

首先，我们需要找出我们想要突变的节点类型。我们通过 ast 函数获取这些信息，并发现节点类型被命名为 BinOp

```py
print(ast.dump(ast.parse("1 + 2 - 3 * 4 / 5"), indent=4)) 
```

```py
Module(
    body=[
        Expr(
            value=BinOp(
                left=BinOp(
                    left=Constant(value=1),
                    op=Add(),
                    right=Constant(value=2)),
                op=Sub(),
                right=BinOp(
                    left=BinOp(
                        left=Constant(value=3),
                        op=Mult(),
                        right=Constant(value=4)),
                    op=Div(),
                    right=Constant(value=5))))],
    type_ignores=[])

```

要突变树，因此你需要更改`op`属性（它具有`Add`、`Sub`、`Mult`和`Div`等值之一）

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/MutationAnalysis.ipynb#Exercises)来练习并查看解决方案。

要突变树，我们需要更改`op`属性（它具有`Add`、`Sub`、`Mult`和`Div`等值之一）。编写一个`BinOpMutator`类来完成必要的突变，然后创建一个`MuBinOpAnalyzer`类，它是`MuFunctionAnalyzer`的子类，并利用`BinOpMutator`。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/MutationAnalysis.ipynb#Exercises)来练习并查看解决方案。

### 练习 2：优化突变分析

我们进行突变分析的技巧在效率上有些低，因为我们在测试那些在测试用例未覆盖的代码中的突变体时也会运行测试。测试用例没有检测它们未覆盖的代码部分错误的可能性。因此，最简单的优化之一是首先从给定的测试用例中恢复覆盖率信息，并且只对那些突变位于测试用例覆盖的代码中的突变体运行测试用例。你能修改`MuFunctionAnalyzer`以将恢复覆盖率作为第一步吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/MutationAnalysis.ipynb#Exercises)来练习并查看解决方案。

### 练习 3：字节码突变器

我们已经看到了如何根据源代码进行突变。这种方法的一个缺点是 Python 字节码也被其他语言针对。在这种情况下，源代码可能无法直接转换为 Python AST，而突变字节码是更可取的。你能实现一个针对 Python 函数的字节码突变器，该突变器直接突变字节码而不是先获取源代码然后进行突变吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/MutationAnalysis.ipynb#Exercises)来处理练习并查看解决方案。

### 练习 4：估计残留缺陷密度

程序的缺陷密度是指程序在发布前检测到的缺陷数量除以程序大小。残留缺陷密度是逃逸检测的缺陷百分比。虽然估计实际的残留缺陷密度很困难，但突变分析可以提供一个上限。未检测到的突变数量是程序内剩余缺陷数量的一个合理的上限。然而，这个上限可能太宽。原因是某些剩余的错误可能相互影响，如果同时存在，可能被现有的测试套件检测到。因此，一个更紧的上限是在给定程序中可以存在而不会被给定测试套件检测到的突变数量。这可以通过从可能的完整突变集开始，并应用来自减少章节的 delta-debugging 来确定需要移除的最小突变数量来实现，以使突变通过测试套件不被检测到。你能通过扩展`MuFunctionAnalyzer`来生成一个新的`RDDEstimator`，该`RDDEstimator`使用这种技术估计残留缺陷密度的上限吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/MutationAnalysis.ipynb#Exercises)来处理练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)的许可。 [最后更改：2023-11-11 18:18:06+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/MutationAnalysis.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒："[突变分析](https://www.fuzzingbook.org/html/MutationAnalysis.html)"。收录于安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒所著的"[模糊测试书](https://www.fuzzingbook.org/)"中。[`www.fuzzingbook.org/html/MutationAnalysis.html`](https://www.fuzzingbook.org/html/MutationAnalysis.html)。检索日期：2023-11-11 18:18:06+01:00.

```py
@incollection{fuzzingbook2023:MutationAnalysis,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Mutation Analysis},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/MutationAnalysis.html}},
    note = {Retrieved 2023-11-11 18:18:06+01:00},
    url = {https://www.fuzzingbook.org/html/MutationAnalysis.html},
    urldate = {2023-11-11 18:18:06+01:00}
}

```
