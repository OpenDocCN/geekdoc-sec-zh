# 减少导致失败的输入

> 原文：[`www.fuzzingbook.org/html/Reducer.html`](http://www.fuzzingbook.org/html/Reducer.html)

通过构造，模糊测试器生成的输入可能难以阅读。这会在*调试*期间引起问题，当人类需要分析失败的确切原因时。在本章中，我们介绍了将导致失败的输入*自动减少并简化到最小*的技术，以简化调试过程。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('JOv1xGVdXAU') 
```

**先决条件**

+   简单的"delta debugging"减少技术没有特定的先决条件。

+   由于减少通常与模糊测试一起使用，阅读关于基本模糊测试的章节是个好主意。

+   后续基于语法的技巧需要了解推导树和解析。

## 概述

要使用本章提供的代码，请编写

```py
>>> from fuzzingbook.Reducer import <identifier> 
```

然后利用以下功能。

一个*减少器*接受一个导致失败的输入，并将其减少到仍然可以重现失败的最小值。本章提供了实现此类减少器的`Reducer`类。

这里有一个简单的例子：一个算术表达式在 Python 解释器中引起错误：

```py
>>> !python -c 'x = 1 + 2 * 3 / 0'
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ZeroDivisionError: division by zero 
```

我们能否将这个输入减少到最小？要使用`Reducer`，首先需要构建一个`Runner`，其结果在精确错误发生时为`FAIL`。因此，我们构建了一个`ZeroDivisionRunner`，其`run()`方法在发生`ZeroDivisionError`时将特别返回`FAIL`结果。

```py
>>> from Fuzzer import ProgramRunner
>>> import [subprocess](https://docs.python.org/3/library/subprocess.html)
>>> class ZeroDivisionRunner(ProgramRunner):
>>>     """Make outcome 'FAIL' if ZeroDivisionError occurs"""
>>> 
>>>     def run(self, inp: str = "") -> Tuple[subprocess.CompletedProcess, Outcome]:
>>>         process, outcome = super().run(inp)
>>>         if process.stderr.find('ZeroDivisionError') >= 0:
>>>             outcome = 'FAIL'
>>>         return process, outcome 
```

如果我们将这个表达式输入到`ZeroDivisionRunner`中，它将产生一个`FAIL`的结果，正如设计的那样。

```py
>>> python_input = "x = 1 + 2 * 3 / 0"
>>> python_runner = ZeroDivisionRunner("python")
>>> process, outcome = python_runner.run(python_input)
>>> outcome
'FAIL' 
```

Delta Debugging 是一种简单且稳健的减少算法。我们可以将`DeltaDebuggingReducer`与这个运行器绑定，并让它确定导致`python`程序失败的子串：

```py
>>> dd = DeltaDebuggingReducer(python_runner)
>>> dd.reduce(python_input)
'3/0' 
```

输入被减少到最小：我们得到了除以零的本质。

<svg width="476pt" height="364pt" viewBox="0.00 0.00 475.50 363.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 359.5)"><g id="node1" class="node"><title>DeltaDebuggingReducer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class DeltaDebuggingReducer:

使用 delta debugging 减少输入。"><text text-anchor="start" x="8" y="-75.58" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">DeltaDebuggingReducer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="DeltaDebuggingReducer"><g id="a_node1_1"><a xlink:href="#" xlink:title="reduce(self, inp: str) -> str:

使用 delta debugging 减少输入`inp`。返回减少后的输入。"><text text-anchor="start" x="57.88" y="-53.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">reduce()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>CachingReducer</title> <g id="a_node2"><a xlink:href="#" xlink:title="class CachingReducer:

一个同时缓存测试结果的减少器。"><text text-anchor="start" x="119.38" y="-216.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">CachingReducer</text> <g id="a_node2_2"><a xlink:href="#" xlink:title="CachingReducer"><g id="a_node2_3"><a xlink:href="#" xlink:title="reset(self):

将测试计数器重置为零。在子类中扩展。"><text text-anchor="start" x="147.88" y="-194.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">reset()</text></a></g> <g id="a_node2_4"><a xlink:href="#" xlink:title="test(self, inp):

使用输入`inp`进行测试。返回结果。

在子类中扩展。"><text text-anchor="start" x="147.88" y="-181.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">test()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>DeltaDebuggingReducer->CachingReducer</title></g> <g id="node3" class="node"><title>Reducer</title> <g id="a_node3"><a xlink:href="#" xlink:title="class Reducer:

减少器的基础类。"><text text-anchor="start" x="144.12" y="-338.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Reducer</text> <g id="a_node3_5"><a xlink:href="#" xlink:title="Reducer"><g id="a_node3_6"><a xlink:href="#" xlink:title="__init__(self, runner: Fuzzer.Runner, log_test: bool = False) -> None:

将减少器附加到给定的`runner`"><text text-anchor="start" x="138.88" y="-316.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__init__()</text></a></g> <g id="a_node3_7"><a xlink:href="#" xlink:title="reduce(self, inp: str) -> str:

减少输入`inp`。返回减少后的输入。

在子类中定义。"><text text-anchor="start" x="138.88" y="-303.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">reduce()</text></a></g> <g id="a_node3_8"><a xlink:href="#" xlink:title="reset(self) -> None:

将测试计数器重置为零。在子类中扩展。"><text text-anchor="start" x="138.88" y="-291" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">reset()</text></a></g> <g id="a_node3_9"><a xlink:href="#" xlink:title="test(self, inp: str) -> str:

使用输入`inp`进行测试。返回结果。

在子类中定义。"><text text-anchor="start" x="138.88" y="-278.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">test()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>CachingReducer->Reducer</title></g> <g id="node4" class="node"><title>GrammarReducer</title> <g id="a_node4"><a xlink:href="#" xlink:title="class GrammarReducer:

使用语法减少输入。"><text text-anchor="start" x="201.88" y="-120.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarReducer</text> <g id="a_node4_10"><a xlink:href="#" xlink:title="GrammarReducer"><g id="a_node4_11"><a xlink:href="#" xlink:title="__init__(self, runner: Fuzzer.Runner, parser: Parser.Parser, *, log_test: bool = False, log_reduce: bool = False):

构造函数。

`runner`是将要使用的运行器。

`parser`是将要使用的解析器。

`log_test` - 如果设置，显示测试和结果。

`log_reduce` - 如果设置，显示减少步骤。"><text text-anchor="start" x="189.88" y="-98" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node4_12"><a xlink:href="#" xlink:title="reduce(self, inp):

减少输入`inp`。返回减少后的输入。

在子类中定义。"><text text-anchor="start" x="189.88" y="-85.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">reduce()</text></a></g> <g id="a_node4_13"><a xlink:href="#" xlink:title="alternate_reductions(self, tree: DerivationTree, symbol: str, depth: int = -1)"><text text-anchor="start" x="189.88" y="-71.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">alternate_reductions()</text></a></g> <g id="a_node4_14"><a xlink:href="#" xlink:title="parse(self, inp)"><text text-anchor="start" x="189.88" y="-58.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">parse()</text></a></g> <g id="a_node4_15"><a xlink:href="#" xlink:title="reduce_subtree(self, tree: DerivationTree, subtree: DerivationTree, depth: int = -1)"><text text-anchor="start" x="189.88" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">reduce_subtree()</text></a></g> <g id="a_node4_16"><a xlink:href="#" xlink:title="reduce_tree(self, tree)"><text text-anchor="start" x="189.88" y="-33.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">reduce_tree()</text></a></g> <g id="a_node4_17"><a xlink:href="#" xlink:title="subtrees_with_symbol(self, tree: DerivationTree, symbol: str, depth: int = -1, ignore_root: bool = True) -> List[DerivationTree]:

在`tree`中查找所有根节点为`symbol`的子树。

如果`ignore_root`为真，忽略`tree`的根节点。"><text text-anchor="start" x="189.88" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">subtrees_with_symbol()</text></a></g> <g id="a_node4_18"><a xlink:href="#" xlink:title="symbol_reductions(self, tree: DerivationTree, symbol: str, depth: int = -1):

找到给定符号"><text text-anchor="start" x="189.88" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">symbol_reductions()</text></a></g></a></g></a></g></g> <g id="edge3" class="edge"><title>GrammarReducer->CachingReducer</title></g> <g id="node5" class="node"><title>图例</title> <text text-anchor="start" x="348.25" y="-84.5" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="348.25" y="-74.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="354.25" y="-74.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="348.25" y="-64.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="354.25" y="-64.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="348.25" y="-54.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="354.25" y="-54.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="348.25" y="-45.45" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 为什么缩减？

在这一点上，我们已经看到了许多测试生成技术，它们都以某种形式产生输入以触发故障。如果它们成功了——也就是说，程序实际上失败了——我们必须找出故障发生的原因以及如何修复它。

这里有一个这样的例子。我们有一个名为`MysteryRunner`的类，它有一个`run()`方法，根据其代码，它偶尔会失败。但这种情况实际上在什么情况下会发生？我们故意隐藏了确切的条件，以便使其不明显。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import quiz 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import Tuple, List, Sequence, Any, Optional 
```

```py
from ExpectError import ExpectError 
```

```py
from Fuzzer import RandomFuzzer, Runner, Outcome 
```

```py
import [re](https://docs.python.org/3/library/re.html) 
```

```py
class MysteryRunner(Runner):
    def run(self, inp: str) -> Tuple[str, Outcome]:
        x = inp.find(chr(0o17 + 0o31))
        y = inp.find(chr(0o27 + 0o22))
        if x >= 0 and y >= 0 and x < y:
            return (inp, Runner.FAIL)
        else:
            return (inp, Runner.PASS) 
```

让我们对函数进行模糊测试，直到找到失败的输入。

```py
mystery = MysteryRunner()
random_fuzzer = RandomFuzzer()
while True:
    inp = random_fuzzer.fuzz()
    result, outcome = mystery.run(inp)
    if outcome == mystery.FAIL:
        break 
```

```py
failing_input = result
failing_input 
```

```py
' 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#'

```

这里的输入中有些东西导致`MysteryRunner`失败。但究竟是什么？

## 手动输入缩减

调试过程中的一个重要步骤是*缩减*——也就是说，识别那些导致故障发生的相关情况，并（如果可能）省略那些不相关的部分。正如 Kernighan 和 Pike [Kernighan *et al*, 1999]所说：

> 对于问题的每一个情况，检查它是否与问题发生相关。如果不相关，请将其从问题报告中或相关测试用例中移除。

特别对于输入，他们建议一个*分而治之*的过程：

> 通过二分搜索进行操作。丢弃一半的输入并查看输出是否仍然错误；如果不是，返回到上一个状态并丢弃另一半的输入。

这是我们很容易尝试的事情，使用我们最后生成的输入：

```py
failing_input 
```

```py
' 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#'

```

例如，如果我们只输入前半部分，我们可以看到错误是否仍然发生：

```py
half_length = len(failing_input) // 2   # // is integer division
first_half = failing_input[:half_length]
mystery.run(first_half) 
```

```py
(" 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69':", 'PASS')

```

不行——只有前半部分是不够的。也许后半部分可以？

```py
second_half = failing_input[half_length:]
mystery.run(second_half) 
```

```py
('\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#', 'PASS')

```

这也没有好到哪里去。我们可能仍然可以通过移除更小的块来继续进行——比如说，一个字符接一个字符。如果我们的测试是确定性的并且容易重复，那么很明显，这个过程最终将产生一个简化的输入。但是，这仍然是一个相当低效的过程，尤其是对于长输入。我们需要的是一个 *策略*，这个策略可以有效地最小化导致失败的输入——一个可以自动化的策略。

## Delta Debugging

一种有效减少导致失败的输入的策略是 *delta debugging* [[Zeller *et al*, 2002](https://doi.org/10.1109/32.988498)]。Delta Debugging 实现了上面列出的 "二分搜索" 策略，但有一个转折：如果两个部分都没有失败（如上所述），它将继续从输入中移除越来越小的块，直到消除单个字符。因此，在移除前半部分之后，我们移除四分之一，然后是二分之一，以此类推。

让我们在我们的例子中说明这一点，看看如果我们移除四分之一会发生什么。

```py
quarter_length = len(failing_input) // 4
input_without_first_quarter = failing_input[quarter_length:]
mystery.run(input_without_first_quarter) 
```

```py
('50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#',
 'FAIL')

```

啊！这失败了，并且将我们的失败输入减少了 25%。让我们再移除一个四分之一。

```py
input_without_first_and_second_quarter = failing_input[quarter_length * 2:]
mystery.run(input_without_first_and_second_quarter) 
```

```py
('\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#', 'PASS')

```

这并不太令人惊讶，因为我们之前已经有过这样的例子：

```py
second_half 
```

```py
'\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#'

```

```py
input_without_first_and_second_quarter 
```

```py
'\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#'

```

那么，移除三分之一怎么样？

```py
input_without_first_and_third_quarter = failing_input[quarter_length:
                                                      quarter_length * 2] + failing_input[quarter_length * 3:]
mystery.run(input_without_first_and_third_quarter) 
```

```py
("50#7*8=$&&=$9!%6(4=&69':<7+1<2!4$>92+$1<(3%&5''>#", 'PASS')

```

好的。让我们移除四分之一。

```py
input_without_first_and_fourth_quarter = failing_input[quarter_length:quarter_length * 3]
mystery.run(input_without_first_and_fourth_quarter) 
```

```py
('50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+', 'FAIL')

```

是的！这成功了。我们的输入现在缩小了 50%。

我们现在已经尝试移除组成原始失败字符串 $\frac{1}{2}$ 和 $\frac{1}{4}$ 的部分。在下一轮迭代中，我们将移除更小的部分——$\frac{1}{8}$，$\frac{1}{16}$，以此类推。我们继续进行，直到我们只剩下 $\frac{1}{97}$——即单个字符。

然而，这是我们可以愉快地让计算机为我们做的事情。我们首先引入一个 `Reducer` 类，作为所有类型减法器的抽象超类。`test()` 方法运行单个测试（如果需要，则进行日志记录）；`reduce()` 方法最终将输入简化到最小。

```py
class Reducer:
  """Base class for reducers."""

    def __init__(self, runner: Runner, log_test: bool = False) -> None:
  """Attach reducer to the given `runner`"""
        self.runner = runner
        self.log_test = log_test
        self.reset()

    def reset(self) -> None:
  """Reset the test counter to zero. To be extended in subclasses."""
        self.tests = 0

    def test(self, inp: str) -> Outcome:
  """Test with input `inp`. Return outcome.
 To be extended in subclasses."""

        result, outcome = self.runner.run(inp)
        self.tests += 1
        if self.log_test:
            print("Test #%d" % self.tests, repr(inp), repr(len(inp)), outcome)
        return outcome

    def reduce(self, inp: str) -> str:
  """Reduce input `inp`. Return reduced input.
 To be defined in subclasses."""

        self.reset()
        # Default: Don't reduce
        return inp 
```

`CachingReducer` 变体保存测试结果，这样我们就不必反复运行相同的测试：

```py
class CachingReducer(Reducer):
  """A reducer that also caches test outcomes"""

    def reset(self):
        super().reset()
        self.cache = {}

    def test(self, inp):
        if inp in self.cache:
            return self.cache[inp]

        outcome = super().test(inp)
        self.cache[inp] = outcome
        return outcome 
```

接下来是 *Delta Debugging* 算法。Delta Debugging 实现了上述策略：它首先移除大小为 $\frac{1}{2}$ 的大块；如果这样做不失败，然后我们继续移除大小为 $\frac{1}{4}$ 的小块，然后是 $\frac{1}{8}$，以此类推。

我们的实现几乎与 Zeller 在 [[Zeller *et al*, 2002](https://doi.org/10.1109/32.988498)] 中的 Python 代码相同；唯一的区别是它已经被调整以在 Python 3 和我们的 `Runner` 框架上工作。变量 `n`（最初为 2）表示粒度——在每一步中，移除大小为 $\frac{1}{n}$ 的块。如果没有测试失败（`some_complement_is_failing` 为 False），则 `n` 被加倍——直到它达到输入的长度。

```py
class DeltaDebuggingReducer(CachingReducer):
  """Reduce inputs using delta debugging."""

    def reduce(self, inp: str) -> str:
  """Reduce input `inp` using delta debugging. Return reduced input."""

        self.reset()
        assert self.test(inp) != Runner.PASS

        n = 2     # Initial granularity
        while len(inp) >= 2:
            start = 0.0
            subset_length = len(inp) / n
            some_complement_is_failing = False

            while start < len(inp):
                complement = inp[:int(start)] + \
                    inp[int(start + subset_length):]

                if self.test(complement) == Runner.FAIL:
                    inp = complement
                    n = max(n - 1, 2)
                    some_complement_is_failing = True
                    break

                start += subset_length

            if not some_complement_is_failing:
                if n == len(inp):
                    break
                n = min(n * 2, len(inp))

        return inp 
```

为了了解`DeltaDebuggingReducer`是如何工作的，让我们将其运行在我们的失败输入上。随着每一步的进行，我们看到剩余的输入如何越来越小，直到只剩下两个字符：

```py
dd_reducer = DeltaDebuggingReducer(mystery, log_test=True)
dd_reducer.reduce(failing_input) 
```

```py
Test #1 ' 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#' 97 FAIL
Test #2 '\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#' 49 PASS
Test #3 " 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69':" 48 PASS
Test #4 '50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#' 73 FAIL
Test #5 "50#7*8=$&&=$9!%6(4=&69':<7+1<2!4$>92+$1<(3%&5''>#" 49 PASS
Test #6 '50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+' 48 FAIL
Test #7 '\'<3+0-3.24#7=!&60)2/+";+' 24 PASS
Test #8 "50#7*8=$&&=$9!%6(4=&69':" 24 PASS
Test #9 '9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+' 36 FAIL
Test #10 '9!%6(4=&69\':=!&60)2/+";+' 24 FAIL
Test #11 '=!&60)2/+";+' 12 PASS
Test #12 "9!%6(4=&69':" 12 PASS
Test #13 '=&69\':=!&60)2/+";+' 18 PASS
Test #14 '9!%6(4=!&60)2/+";+' 18 FAIL
Test #15 '9!%6(42/+";+' 12 PASS
Test #16 '9!%6(4=!&60)' 12 FAIL
Test #17 '=!&60)' 6 PASS
Test #18 '9!%6(4' 6 PASS
Test #19 '6(4=!&60)' 9 FAIL
Test #20 '6(460)' 6 FAIL
Test #21 '60)' 3 PASS
Test #22 '6(4' 3 PASS
Test #23 '(460)' 5 FAIL
Test #24 '460)' 4 PASS
Test #25 '(0)' 3 FAIL
Test #26 '0)' 2 PASS
Test #27 '(' 1 PASS
Test #28 '()' 2 FAIL
Test #29 ')' 1 PASS

```

```py
'()'

```

现在我们知道为什么`MysteryRunner`失败了——只要输入包含两个匹配的括号就足够了。Delta Debugging 在 29 步中确定了这一点。其结果是*1-minimal*，这意味着包含的每个字符都是产生错误所必需的；删除任何（如上所述的测试`#27`和`#29`所示）都不会使测试失败。这个特性由 delta debugging 算法保证，在其最后阶段，它总是尝试逐个删除字符。

如上所述的简化测试用例具有许多优点：

+   一个简化的测试用例**减少了程序员的认知负荷**。测试用例更短且更专注，因此不会让程序员负担无关的细节。简化的输入通常会导致更短的执行时间和更小的程序状态，这两者都会在理解错误时减少搜索空间。在我们的案例中，我们消除了大量的无关输入——只有简化输入中包含的两个字符是相关的。

+   一个简化的测试用例**更容易沟通**。这里只需要总结：`MysteryRunner 在"()"上失败`，这比`MysteryRunner 在 4100 字符的输入（附件）上失败`要好得多。

+   一个简化的测试用例有助于**识别重复项**。如果已经报告了类似的错误，并且它们都被简化到相同的原因（即输入包含匹配的括号），那么很明显，所有这些错误都是同一根本原因的不同症状——并且可以通过一个代码修复一次性解决。

Delta Debugging 有多有效？在最佳情况下（当左半部分或右半部分失败时），测试的数量与输入长度$n$的对数成比例（即$O(\log_2 n)$）；这与二分搜索的复杂度相同。然而，在最坏的情况下，delta debugging 可能需要与$n²$成比例的测试次数（即$O(n²)$）——这种情况发生在我们下降到字符粒度时，我们必须反复尝试删除所有字符，结果发现删除最后一个字符会导致失败 [[Zeller *et al*, 2002](https://doi.org/10.1109/32.988498)]。（这是一个相当病态的情况，尽管如此。）

通常，delta debugging 是一个易于实现、易于部署和易于使用的稳健算法——前提是底层测试用例是确定性的，并且运行速度足够快，可以保证进行多次实验。由于这些是使模糊测试有效的相同先决条件，delta debugging 是模糊测试的一个优秀伴侣。

### 测验

如果被测试的函数没有失败会发生什么？

事实上，`DeltaDebugger`检查其假设是否成立。如果不成立，则断言失败。

```py
with ExpectError():
    dd_reducer.reduce("I am a passing input") 
```

```py
Test #1 'I am a passing input' 20 PASS

```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_29923/3080932113.py", line 2, in <module>
    dd_reducer.reduce("I am a passing input")
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_29923/2870451724.py", line 8, in reduce
    assert self.test(inp) != Runner.PASS
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError (expected)

```

## 基于语法的输入简化

如果输入语言在句法上很复杂，delta 调试可能需要进行多次尝试来简化，甚至可能无法简化输入。因此，在本章的后半部分，我们介绍了一个名为*基于语法的简化算法*（或简称为 GRABR）的算法，该算法利用*语法*来简化句法复杂的输入。

### 词汇简化与句法规则

尽管 delta 调试在一般情况下很稳健，但在某些情况下它可能效率低下，甚至完全失败。作为一个例子，考虑一些*表达式输入*，例如`1 + (2 * 3)`。Delta 调试需要多次测试来简化导致失败的输入，但最终它会返回一个最小输入

```py
expr_input = "1 + (2 * 3)"
dd_reducer = DeltaDebuggingReducer(mystery, log_test=True)
dd_reducer.reduce(expr_input) 
```

```py
Test #1 '1 + (2 * 3)' 11 FAIL
Test #2 '2 * 3)' 6 PASS
Test #3 '1 + (' 5 PASS
Test #4 '+ (2 * 3)' 9 FAIL
Test #5 '+ ( 3)' 6 FAIL
Test #6 ' 3)' 3 PASS
Test #7 '+ (' 3 PASS
Test #8 ' ( 3)' 5 FAIL
Test #9 '( 3)' 4 FAIL
Test #10 '3)' 2 PASS
Test #11 '( ' 2 PASS
Test #12 '(3)' 3 FAIL
Test #13 '()' 2 FAIL
Test #14 ')' 1 PASS
Test #15 '(' 1 PASS

```

```py
'()'

```

然而，在上面的测试中，只有少数几个实际上代表了语法上有效的算术表达式。在实际环境中，我们可能想要测试一个实际上能够*解析*此类表达式并*拒绝*所有无效输入的程序。我们定义了一个名为`EvalMysteryRunner`的类，它首先根据我们表达式语法的规则*解析*给定的输入，并且只有当它符合条件时，才会传递给我们的原始`MysteryRunner`。这模拟了测试表达式解释器的情况，在这种情况下，只有有效的输入才能触发错误。

```py
from Grammars import EXPR_GRAMMAR 
```

```py
from Parser import EarleyParser, Parser  # minor dependency 
```

```py
class EvalMysteryRunner(MysteryRunner):
    def __init__(self) -> None:
        self.parser = EarleyParser(EXPR_GRAMMAR)

    def run(self, inp: str) -> Tuple[str, Outcome]:
        try:
            tree, *_ = self.parser.parse(inp)
        except SyntaxError:
            return (inp, Runner.UNRESOLVED)

        return super().run(inp) 
```

```py
eval_mystery = EvalMysteryRunner() 
```

在这种情况下，结果证明 delta 调试完全失败。它应用的任何简化都不会产生语法上有效的输入，因此输入的整体复杂性与之前一样。

```py
dd_reducer = DeltaDebuggingReducer(eval_mystery, log_test=True)
dd_reducer.reduce(expr_input) 
```

```py
Test #1 '1 + (2 * 3)' 11 FAIL
Test #2 '2 * 3)' 6 UNRESOLVED
Test #3 '1 + (' 5 UNRESOLVED
Test #4 '+ (2 * 3)' 9 UNRESOLVED
Test #5 '1 2 * 3)' 8 UNRESOLVED
Test #6 '1 + ( 3)' 8 UNRESOLVED
Test #7 '1 + (2 *' 8 UNRESOLVED
Test #8 ' + (2 * 3)' 10 UNRESOLVED
Test #9 '1+ (2 * 3)' 10 UNRESOLVED
Test #10 '1 (2 * 3)' 9 UNRESOLVED
Test #11 '1 + 2 * 3)' 10 UNRESOLVED
Test #12 '1 + ( * 3)' 10 UNRESOLVED
Test #13 '1 + (2 3)' 9 UNRESOLVED
Test #14 '1 + (2 *3)' 10 UNRESOLVED
Test #15 '1 + (2 * ' 9 UNRESOLVED
Test #16 '1  (2 * 3)' 10 UNRESOLVED
Test #17 '1 +(2 * 3)' 10 UNRESOLVED
Test #18 '1 + (2* 3)' 10 UNRESOLVED
Test #19 '1 + (2  3)' 10 UNRESOLVED
Test #20 '1 + (2 * )' 10 UNRESOLVED
Test #21 '1 + (2 * 3' 10 UNRESOLVED

```

```py
'1 + (2 * 3)'

```

如果被测试的程序对输入的有效性有多个约束，这种行为是可能的。Delta 调试并不了解这些约束（也不了解输入结构的一般情况），因此它可能会一次又一次地违反这些约束。

### 基于语法的简化方法

为了简化具有高句法复杂性的输入，我们使用另一种方法：而不是简化输入字符串，我们简化表示其结构的*树*。一般想法是，从一个*推导树*开始，这个树是通过解析输入得到的，然后*用相同类型的较小子树替换子树*。这些替代子树可以来自

1.  从树本身，或者

1.  通过使用树中的元素应用替代语法扩展。

让我们用一个例子来展示这两种策略。我们从一个算术表达式的推导树开始：

```py
from Grammars import Grammar
from GrammarFuzzer import all_terminals, expansion_to_children, display_tree 
```

```py
derivation_tree, *_ = EarleyParser(EXPR_GRAMMAR).parse(expr_input)
display_tree(derivation_tree) 
```

<svg width="210pt" height="575pt" viewBox="0.00 0.00 209.62 575.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 571)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="81.62" y="-553.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="81.62" y="-503.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="33.62" y="-453.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="81.62" y="-453.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="127.62" y="-453.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="31.62" y="-402.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27.62" y="-352.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="20.62" y="-302.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="20.62" y="-252.2" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="127.62" y="-402.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="127.62" y="-352.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="74.62" y="-302.45" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="128.62" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge12" class="edge"><title>10->12</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="182.62" y="-302.45" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge24" class="edge"><title>10->24</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="128.62" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="79.62" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="128.62" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge18" class="edge"><title>13->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="174.62" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge19" class="edge"><title>13->19</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="79.62" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="79.62" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="79.62" y="-51.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="174.62" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="174.62" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="174.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge22" class="edge"><title>21->22</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="174.62" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge23" class="edge"><title>22->23</title></g></g></svg>

### 通过替换子树进行简化

为了简化这个树，我们可以用树中较低位置的某个`<expr>`子树替换树中较高位置的任何`<expr>`符号。例如，我们可以用最上面的`<expr>`替换它的右`<expr>`子树，得到字符串`(2 + 3)`：

```py
import [copy](https://docs.python.org/3/library/copy.html) 
```

```py
new_derivation_tree = copy.deepcopy(derivation_tree)
# We really should have some query language
sub_expr_tree = new_derivation_tree[1][0][1][2]
display_tree(sub_expr_tree) 
```

<svg width="157pt" height="475pt" viewBox="0.00 0.00 157.00 474.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 470.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="76" y="-453.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="76" y="-402.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="76" y="-352.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="22" y="-302.45" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="76" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge4" class="edge"><title>2->4</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="130" y="-302.45" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge16" class="edge"><title>2->16</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="76" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="27" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="76" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge10" class="edge"><title>5->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="122" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge11" class="edge"><title>5->11</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="122" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="122" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="122" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="122" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge15" class="edge"><title>14->15</title></g></g></svg>

```py
new_derivation_tree[1][0] = sub_expr_tree
display_tree(new_derivation_tree) 
```

<svg width="157pt" height="525pt" viewBox="0.00 0.00 157.00 524.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 520.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="76" y="-503.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="76" y="-453.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="76" y="-402.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="76" y="-352.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="22" y="-302.45" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="76" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge5" class="edge"><title>3->5</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="130" y="-302.45" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge17" class="edge"><title>3->17</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="76" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="27" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="76" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge11" class="edge"><title>6->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="122" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge12" class="edge"><title>6->12</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="122" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="122" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="122" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="122" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge16" class="edge"><title>15->16</title></g></g></svg>

```py
all_terminals(new_derivation_tree) 
```

```py
'(2 * 3)'

```

用一个子树替换另一个子树只有在单个元素如`<expr>`在我们的树中多次出现时才有效。在上面的简化`new_derivation_tree`中，我们只能再次替换一个`<expr>`树。

### 通过替代扩展进行简化

简化此树的第二种方法是应用 *替代展开*。也就是说，对于符号，我们检查是否存在具有较少子节点的替代展开。然后，我们用替代展开替换符号，并从树中填充所需的符号。

例如，考虑上面的 `new_derivation_tree`。对 `<term>` 应用的展开是

```py
<term> ::= <term> * <factor> 
```

让我们用以下替代展开来替换它：

```py
<term> ::= <factor>
```

```py
term_tree = new_derivation_tree[1][0][1][0][1][0][1][1][1][0]
display_tree(term_tree) 
```

<svg width="157pt" height="274pt" viewBox="0.00 0.00 157.00 273.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 269.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="76" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="27" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="76" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge5" class="edge"><title>0->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="122" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge6" class="edge"><title>0->6</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="122" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="122" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="122" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="122" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge10" class="edge"><title>9->10</title></g></g></svg>

```py
shorter_term_tree = term_tree[1][2]
display_tree(shorter_term_tree) 
```

<svg width="62pt" height="223pt" viewBox="0.00 0.00 62.00 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="27" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge4" class="edge"><title>3->4</title></g></g></svg>

```py
new_derivation_tree[1][0][1][0][1][0][1][1][1][0] = shorter_term_tree
display_tree(new_derivation_tree) 
```

<svg width="147pt" height="475pt" viewBox="0.00 0.00 146.75 474.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 470.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="69.38" y="-453.2" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="69.38" y="-402.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="69.38" y="-352.7" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="69.38" y="-302.45" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="15.38" y="-252.2" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="69.38" y="-252.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge5" class="edge"><title>3->5</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="123.38" y="-252.2" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge11" class="edge"><title>3->11</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="69.38" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="69.38" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="69.38" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="69.38" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="69.38" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge10" class="edge"><title>9->10</title></g></g></svg>

```py
all_terminals(new_derivation_tree) 
```

```py
'(3)'

```

如果我们用（较小的）子树替换推导子树，并且如果我们寻找再次产生较小子树的替代展开，我们可以系统地简化输入。这可能会比 delta 调试快得多，因为我们的输入始终是语法有效的。然而，我们需要一个策略来确定何时应用哪个简化规则。这就是我们在本节剩余部分要开发的。

<details id="Excursion:-A-Class-for-Reducing-with-Grammars"><summary>用于语法减少的类</summary>

我们引入 `GrammarReducer` 类，它再次是一个 `Reducer`。请注意，我们派生自 `CachingReducer`，因为该策略会产生多个重复。

```py
class GrammarReducer(CachingReducer):
  """Reduce inputs using grammars"""

    def __init__(self, runner: Runner, parser: Parser, *,
                 log_test: bool = False, log_reduce: bool = False):
  """Constructor.
 `runner` is the runner to be used.
 `parser` is the parser to be used.
 `log_test` - if set, show tests and results.
 `log_reduce` - if set, show reduction steps.
 """

        super().__init__(runner, log_test=log_test)
        self.parser = parser
        self.grammar = parser.grammar()
        self.start_symbol = parser.start_symbol()
        self.log_reduce = log_reduce
        self.try_all_combinations = False 
```

#### 一些辅助工具

我们定义了一些辅助函数，我们将在我们的策略中需要它们。`tree_list_to_string()` 做名字暗示的事情，从一个推导树的列表创建一个字符串：

```py
from GrammarFuzzer import DerivationTree 
```

```py
def tree_list_to_string(q: List[DerivationTree]) -> str:
    return "[" + ", ".join([all_terminals(tree) for tree in q]) + "]" 
```

```py
tree_list_to_string([derivation_tree, derivation_tree]) 
```

```py
'[1 + (2 * 3), 1 + (2 * 3)]'

```

函数 `possible_combinations()` 接收一个列表的列表 $[[x_1, x_2], [y_1, y_2], \dots]$ 并创建一个组合列表 $[[x_1, y_1], [x_1, y_2], [x_2, y_1], [x_2, y_2], \dots]$.

```py
def possible_combinations(list_of_lists: List[List[Any]]) -> List[List[Any]]:
    if len(list_of_lists) == 0:
        return []

    ret = []
    for e in list_of_lists[0]:
        if len(list_of_lists) == 1:
            ret.append([e])
        else:
            for c in possible_combinations(list_of_lists[1:]):
                new_combo = [e] + c
                ret.append(new_combo)

    return ret 
```

```py
possible_combinations([[1, 2], ['a', 'b']]) 
```

```py
[[1, 'a'], [1, 'b'], [2, 'a'], [2, 'b']]

```

函数 `number_of_nodes()` 和 `max_height()` 分别返回给定树的节点数和最大高度。

```py
def number_of_nodes(tree: DerivationTree) -> int:
    (symbol, children) = tree
    if children is None:
        return 1

    return 1 + sum([number_of_nodes(c) for c in children]) 
```

```py
number_of_nodes(derivation_tree) 
```

```py
25

```

```py
def max_height(tree: DerivationTree) -> int:
    (symbol, children) = tree
    if children is None or len(children) == 0:
        return 1

    return 1 + max([max_height(c) for c in children]) 
```

```py
max_height(derivation_tree) 
```

```py
12

```

#### 简化策略

现在我们来实现我们的两种简化策略——替换子树和替代展开。

##### 寻找子树

方法 `subtrees_with_symbol()` 返回给定树中所有根节点等于给定符号的子树。如果设置了 `ignore_root`（默认值），则不比较 `tree` 的根节点。（下面将讨论 `depth` 参数。）

```py
class GrammarReducer(GrammarReducer):
    def subtrees_with_symbol(self, tree: DerivationTree,
                             symbol: str, depth: int = -1,
                             ignore_root: bool = True) -> List[DerivationTree]:
  """Find all subtrees in `tree` whose root is `symbol`.
 If `ignore_root` is true, ignore the root note of `tree`."""

        ret = []
        (child_symbol, children) = tree
        if depth <= 0 and not ignore_root and child_symbol == symbol:
            ret.append(tree)

        # Search across all children
        if depth != 0 and children is not None:
            for c in children:
                ret += self.subtrees_with_symbol(c,
                                                 symbol,
                                                 depth=depth - 1,
                                                 ignore_root=False)

        return ret 
```

这里有一个例子：这些都是我们推导树 `derivation_tree` 中包含 `<term>` 的所有子树。

```py
grammar_reducer = GrammarReducer(
    mystery,
    EarleyParser(EXPR_GRAMMAR),
    log_reduce=True) 
```

```py
all_terminals(derivation_tree) 
```

```py
'1 + (2 * 3)'

```

```py
[all_terminals(t) for t in grammar_reducer.subtrees_with_symbol(
    derivation_tree, "<term>")] 
```

```py
['1', '(2 * 3)', '2 * 3', '3']

```

如果我们想要替换 `<term>` 子树以简化树，这些是我们可以用它们替换的子树。

##### 替代展开

我们的第二种策略，通过替代展开简化，要复杂一些。我们首先获取给定符号的可能展开（从具有最少子节点的那些开始）。对于每个展开，我们使用 `subtrees_with_symbols()`（上面）从子树中填充符号的值。然后我们选择第一个可能的组合（或者如果设置了属性 `try_all_combinations`，则选择所有组合）。

```py
class GrammarReducer(GrammarReducer):
    def alternate_reductions(self, tree: DerivationTree, symbol: str, 
                             depth: int = -1):
        reductions = []

        expansions = self.grammar.get(symbol, [])
        expansions.sort(
            key=lambda expansion: len(
                expansion_to_children(expansion)))

        for expansion in expansions:
            expansion_children = expansion_to_children(expansion)

            match = True
            new_children_reductions = []
            for (alt_symbol, _) in expansion_children:
                child_reductions = self.subtrees_with_symbol(
                    tree, alt_symbol, depth=depth)
                if len(child_reductions) == 0:
                    match = False   # Child not found; cannot apply rule
                    break

                new_children_reductions.append(child_reductions)

            if not match:
                continue  # Try next alternative

            # Use the first suitable combination
            for new_children in possible_combinations(new_children_reductions):
                new_tree = (symbol, new_children)
                if number_of_nodes(new_tree) < number_of_nodes(tree):
                    reductions.append(new_tree)
                    if not self.try_all_combinations:
                        break

        # Sort by number of nodes
        reductions.sort(key=number_of_nodes)

        return reductions 
```

```py
grammar_reducer = GrammarReducer(
    mystery,
    EarleyParser(EXPR_GRAMMAR),
    log_reduce=True) 
```

```py
all_terminals(derivation_tree) 
```

```py
'1 + (2 * 3)'

```

这里是 `<term>` 的所有组合：

```py
grammar_reducer.try_all_combinations = True
print([all_terminals(t)
       for t in grammar_reducer.alternate_reductions(derivation_tree, "<term>")]) 
```

```py
['1', '2', '3', '1 * 1', '1 * 3', '2 * 1', '2 * 3', '3 * 1', '3 * 3', '(2 * 3)', '1 * 2 * 3', '2 * 2 * 3', '3 * 2 * 3', '1 * (2 * 3)', '(2 * 3) * 1', '(2 * 3) * 3', '2 * (2 * 3)', '3 * (2 * 3)']

```

然而，默认情况下，只是简单地返回这些中的第一个：

```py
grammar_reducer.try_all_combinations = False
[all_terminals(t) for t in grammar_reducer.alternate_reductions(
    derivation_tree, "<term>")] 
```

```py
['1', '1 * 1']

```

##### 两种策略结合

现在，让我们合并两种策略。为了用给定的符号替换子树，我们首先搜索已经存在的子树（使用 `subtrees_with_symbol()`）；然后进行备选扩展（使用 `alternate_expansions()`）。

```py
class GrammarReducer(GrammarReducer):
    def symbol_reductions(self, tree: DerivationTree, symbol: str, 
                          depth: int = -1):
  """Find all expansion alternatives for the given symbol"""
        reductions = (self.subtrees_with_symbol(tree, symbol, depth=depth)
                      + self.alternate_reductions(tree, symbol, depth=depth))

        # Filter duplicates
        unique_reductions = []
        for r in reductions:
            if r not in unique_reductions:
                unique_reductions.append(r)

        return unique_reductions 
```

```py
grammar_reducer = GrammarReducer(
    mystery,
    EarleyParser(EXPR_GRAMMAR),
    log_reduce=True) 
```

```py
all_terminals(derivation_tree) 
```

```py
'1 + (2 * 3)'

```

这些是 `<expr>` 节点的可能简化方式。注意我们首先返回子树（`1 + (2 * 3)`，`(2 * 3)`，`2 * 3`），然后再进行 `<expr>` 的备选扩展（`1`）。

```py
reductions = grammar_reducer.symbol_reductions(derivation_tree, "<expr>")
tree_list_to_string([r for r in reductions]) 
```

```py
'[1 + (2 * 3), (2 * 3), 2 * 3, 1]'

```

这些是 `<term>` 节点的可能简化方式。再次强调，我们首先有推导树的子树，然后是备选扩展 `1 * 1`。

```py
reductions = grammar_reducer.symbol_reductions(derivation_tree, "<term>")
tree_list_to_string([r for r in reductions]) 
```

```py
'[1, (2 * 3), 2 * 3, 3, 1 * 1]'

```

#### 简化策略

现在，我们能够为树中的每个符号返回多个备选方案。这就是我们在简化策略的核心函数 `reduce_subtree()` 中应用的内容。从 `subtree` 开始，对于每个子节点，我们找到可能的简化。对于每个简化，我们用简化替换子节点并测试结果（完整）树。如果它失败，我们的简化是成功的；否则，我们将子节点放回原位并尝试下一个简化。最终，我们将 `reduce_subtree()` 应用到所有子节点上，也将它们简化。

```py
class GrammarReducer(GrammarReducer):
    def reduce_subtree(self, tree: DerivationTree,
                       subtree: DerivationTree, depth: int = -1):
        symbol, children = subtree
        if children is None or len(children) == 0:
            return False

        if self.log_reduce:
            print("Reducing", all_terminals(subtree), "with depth", depth)

        reduced = False
        while True:
            reduced_child = False
            for i, child in enumerate(children):
                if child is None:
                    continue

                (child_symbol, _) = child
                for reduction in self.symbol_reductions(
                        child, child_symbol, depth):
                    if number_of_nodes(reduction) >= number_of_nodes(child):
                        continue

                    # Try this reduction
                    if self.log_reduce:
                        print(
                            "Replacing",
                            all_terminals(
                                children[i]),
                            "by",
                            all_terminals(reduction))
                    children[i] = reduction
                    if self.test(all_terminals(tree)) == Runner.FAIL:
                        # Success
                        if self.log_reduce:
                            print("New tree:", all_terminals(tree))
                        reduced = reduced_child = True
                        break
                    else:
                        # Didn't work out - restore
                        children[i] = child

            if not reduced_child:
                if self.log_reduce:
                    print("Tried all alternatives for", all_terminals(subtree))
                break

        # Run recursively
        for c in children:
            if self.reduce_subtree(tree, c, depth):
                reduced = True

        return reduced 
```

我们现在只需要几个驱动器。`reduce_tree()` 方法是进入 `reduce_subtree()` 的主要入口点：

```py
class GrammarReducer(GrammarReducer):
    def reduce_tree(self, tree):
        return self.reduce_subtree(tree, tree) 
```

自定义方法 `parse()` 将给定输入转换为推导树：

```py
class GrammarReducer(GrammarReducer):
    def parse(self, inp):
        tree, *_ = self.parser.parse(inp)
        if self.log_reduce:
            print(all_terminals(tree))
        return tree 
```

`reduce()` 方法是唯一的入口点，解析输入然后简化它。

```py
class GrammarReducer(GrammarReducer):
    def reduce(self, inp):
        tree = self.parse(inp)
        self.reduce_tree(tree)
        return all_terminals(tree) 
```</details>

让我们在实际中尝试我们的 `GrammarReducer` 类，针对输入 `expr_input` 和 `mystery()` 函数。我们能够多快将其简化？

```py
expr_input 
```

```py
'1 + (2 * 3)'

```

```py
grammar_reducer = GrammarReducer(
    eval_mystery,
    EarleyParser(EXPR_GRAMMAR),
    log_test=True)
grammar_reducer.reduce(expr_input) 
```

```py
Test #1 '(2 * 3)' 7 FAIL
Test #2 '2 * 3' 5 PASS
Test #3 '3' 1 PASS
Test #4 '2' 1 PASS
Test #5 '(3)' 3 FAIL

```

```py
'(3)'

```

成功！仅用五步，我们的 `GrammarReducer` 就将输入简化到导致失败的最小值。注意，所有测试都是通过构造语法有效的，避免了导致 delta 调试停滞的 `UNRESOLVED` 结果。

### 深度优先策略

即使五步已经很好，我们仍然可以做得更好。如果我们查看上面的日志，我们会看到在测试 `#2` 中，输入（树）简化为 `2 * 3` 后，我们的 `GrammarReducer` 首先尝试用 `2` 和 `3` 替换树，它们是备选的 `<term>` 子树。这当然可能有效；但如果有很多可能的子树，我们的策略将花费相当多的时间逐一尝试。

如上所述，Delta 调试遵循尝试将输入大约减半的想法，因此快速向最小输入推进。通过用更小的子树替换树，我们*可能*显著简化树，但可能需要多次尝试才能做到。更好的策略是首先只考虑*大的*子树——既用于子树替换也用于备选扩展。为了找到这样的*大的*子树，我们限制在子树中搜索可能替换的*深度*——首先查看直接后代，然后查看更低的后代。

这是`subtrees_with_symbol()`中使用的`depth`参数的作用，并通过调用函数传递。如果设置，则只返回给定深度的符号。以下是一个例子，再次从我们的推导树`derivation_tree`开始：

```py
grammar_reducer = GrammarReducer(
    mystery,
    EarleyParser(EXPR_GRAMMAR),
    log_reduce=True) 
```

```py
all_terminals(derivation_tree) 
```

```py
'1 + (2 * 3)'

```

```py
display_tree(derivation_tree) 
```

<svg width="210pt" height="575pt" viewBox="0.00 0.00 209.62 575.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 571)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="81.62" y="-553.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="81.62" y="-503.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="33.62" y="-453.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="81.62" y="-453.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="127.62" y="-453.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="31.62" y="-402.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27.62" y="-352.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="20.62" y="-302.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="20.62" y="-252.2" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="127.62" y="-402.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="127.62" y="-352.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="74.62" y="-302.45" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="128.62" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge12" class="edge"><title>10->12</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="182.62" y="-302.45" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge24" class="edge"><title>10->24</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="128.62" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="79.62" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="128.62" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge18" class="edge"><title>13->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="174.62" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge19" class="edge"><title>13->19</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="79.62" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="79.62" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="79.62" y="-51.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="174.62" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="174.62" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="174.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge22" class="edge"><title>21->22</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="174.62" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge23" class="edge"><title>22->23</title></g></g></svg>

在深度为 1 时，没有`<term>`符号：

```py
[all_terminals(t) for t in grammar_reducer.subtrees_with_symbol(
    derivation_tree, "<term>", depth=1)] 
```

```py
[]

```

在深度为 2 时，我们在左侧有`<term>`子树：

```py
[all_terminals(t) for t in grammar_reducer.subtrees_with_symbol(
    derivation_tree, "<term>", depth=2)] 
```

```py
['1']

```

在深度为 3 时，我们在右侧有`<term>`子树：

```py
[all_terminals(t) for t in grammar_reducer.subtrees_with_symbol(
    derivation_tree, "<term>", depth=3)] 
```

```py
['(2 * 3)']

```

现在的想法是从深度 0 开始，随着我们的进行逐步增加：

```py
class GrammarReducer(GrammarReducer):
    def reduce_tree(self, tree):
        depth = 0
        while depth < max_height(tree):
            reduced = self.reduce_subtree(tree, tree, depth)
            if reduced:
                depth = 0    # Start with new tree
            else:
                depth += 1   # Extend search for subtrees
        return tree 
```

```py
grammar_reducer = GrammarReducer(
    mystery,
    EarleyParser(EXPR_GRAMMAR),
    log_test=True)
grammar_reducer.reduce(expr_input) 
```

```py
Test #1 '(2 * 3)' 7 FAIL
Test #2 '(3)' 3 FAIL
Test #3 '3' 1 PASS

```

```py
'(3)'

```

我们看到，在我们的设置中，深度优先策略甚至需要更少的步骤。

### 比较策略

我们通过构建一个非常长的表达式来结束，以展示基于文本的 delta 调试和我们的基于语法的减少之间的差异：

```py
from GrammarFuzzer import GrammarFuzzer 
```

```py
long_expr_input = GrammarFuzzer(EXPR_GRAMMAR, min_nonterminals=100).fuzz()
long_expr_input 
```

```py
'++---((-2 / 3 / 3 - -+1 / 5 - 2) * ++6 / +8 * 4 / 9 / 2 * 8 + ++(5) * 3 / 8 * 0 + 3 * 3 + 4 / 0 / 6 + 9) * ++++(+--9 * -3 * 7 / 4 + --(4) / 3 - 0 / 3 + 5 + 0) * (1 * 6 - 1 / 9 * 5 - 9 / 0 + 7) * ++(8 - 1) * +1 * 7 * 0 + ((1 + 4) / 4 * 8 * 9 * 4 + 4 / (4) * 1 - (4) * 8 * 5 + 1 + 4) / (+(2 - 1 - 9) * 5 + 3 + 6 - 2) * +3 * (3 - 7 + 8) / 4 - -(9 * 4 - 1 * 0 + 5) / (5 / 9 * 5 + 2) * 7 + ((7 - 5 + 3) / 1 * 8 - 8 - 9) * --+1 * 4 / 4 - 4 / 7 * 4 - 3 / 6 * 1 - 2 - 7 - 8'

```

使用语法，我们只需要进行少量测试就能找到导致失败的输入：

```py
from Timer import Timer 
```

```py
grammar_reducer = GrammarReducer(eval_mystery, EarleyParser(EXPR_GRAMMAR))
with Timer() as grammar_time:
    print(grammar_reducer.reduce(long_expr_input)) 
```

```py
(9)

```

```py
grammar_reducer.tests 
```

```py
10

```

```py
grammar_time.elapsed_time() 
```

```py
0.0751019999734126

```

与此相反，delta 调试需要数量级更多的测试（以及相应的时间）。再次强调，减少的效果并不像基于语法的减少器那样完美。

```py
dd_reducer = DeltaDebuggingReducer(eval_mystery)
with Timer() as dd_time:
    print(dd_reducer.reduce(long_expr_input)) 
```

```py
((2 - 1 - 2) * 8 + (5) - (4)) / ((2) * 3) * (9) / 3 / 1 - 8

```

```py
dd_reducer.tests 
```

```py
900

```

```py
dd_time.elapsed_time() 
```

```py
1.6051183340023272

```

我们看到，如果一个输入在语法上复杂，使用语法来减少输入是最佳选择。

## 经验教训

+   将导致失败的输入减少到最小有助于测试和调试。

+   *Delta 调试*是一个简单且健壮的算法，可以轻松减少测试用例。

+   对于语法复杂的输入，*基于语法的减少*要快得多，并且结果更好。

## 下一步

我们下一章将重点介绍 Web GUI Fuzzing，这是另一个生成和减少测试用例至关重要的领域。

## 背景

这里讨论的“词法”delta 调试算法源于[[Zeller 等人，2002](https://doi.org/10.1109/32.988498)]；实际上，这正是 Zeller 在 2002 年所使用的 Python 实现。系统地减少输入的想法已经被发现多次，尽管不如 delta 调试那样自动和通用。[Slutz 等人，1998](https://www.microsoft.com/en-us/research/publication/massive-stochastic-testing-of-sql/)，例如，讨论了 SQL 数据库中 SQL 语句的系统化减少；这个过程作为人工工作被[ Kernighan 等人，1999](https://www.cs.princeton.edu/courses/archive/s99/cos217/lectures/lecture10.pdf)很好地描述了。

关于 delta debugging 在处理语法复杂输入时的不足，最初是在 *编译器测试* 中讨论的，并且很快发现 *减少树输入* 而不是字符串输入是一个替代方案。*分层 delta debugging* (*HDD*) [[Misherghi 等人，2006](https://doi.org/10.1145/1134285.1134307)] 在解析树的子树上应用 delta debugging，系统地减少解析树到最小。*广义树减少* [[Herfert 等人，2017](http://dl.acm.org/citation.cfm?id=3155562.3155669)] 将这一想法推广到应用任意 *模式*，例如在子树中将一个项替换为兼容项，就像 `subtrees_with_symbol()` 所做的那样。使用 *语法* 来减少输入最初是在 *Perses* 工具 [[Sun 等人，2018](https://doi.org/10.1145/3180155.3180236)] 中实现的；我们的算法实现了非常相似的战略。寻找替代扩展（如 `alternate_reductions()`）是本章的一个贡献。

虽然将 delta debugging 应用于代码行可以做得相当不错，但 *语法* 和特别是 *语言特定* 的方法可以为当前编程语言做更好的工作：

+   *C-Reduce* [[Regehr 等人，2012](https://doi.org/10.1145/2254064.2254104)] 是一个专门针对编程语言减少的减少器。除了 delta debugging 或树变换风格的减少外，C-Reduce 还提供了超过 30 种源到源转换，这些转换将聚合替换为标量，删除定义和所有调用位置上的函数参数，将函数更改为返回 `void` 并删除所有 `return` 语句，等等。虽然这些原则专门针对 C 语言（并用于测试 C 编译器），但它们适用于遵循 ALGOL 类语法的任意编程语言。

+   Kalhauge 和 Palsberg [[Kalhauge 等人，2019](https://doi.org/10.1145/3338906.3338956)] 介绍了 *依赖图二进制减少*，这是一个减少任意具有依赖关系的输入的通用解决方案。他们的 *J-Reduce* 工具专门针对 Java 程序，并且比 delta debugging 快得多，并且达到了更高的减少率。

减少输入在 *基于属性的测试* 的上下文中也效果良好；也就是说，为单个函数生成数据结构，然后可以在失败时对其进行减少（“缩小”）。[Hypothesis](https://hypothesis.works) fuzzer 有许多特定类型的缩小策略；这篇[博客文章](https://hypothesis.works/articles/integrated-shrinking/)讨论了其一些特性。

在调试书籍中关于 ["减少故障诱导输入" 的章节](https://www.debuggingbook.org/html/DeltaDebugger.html) 提供了一个更易于部署的 delta debugging 的替代实现 `DeltaDebugger`；在这里，只需简单地

```py
with DeltaDebugger() as dd:
    fun(args...)
dd 
```

减少 `args` 中用于失败（抛出异常）的函数 `fun()` 的输入。该章节还讨论了进一步的用法示例，包括将 *代码* 简化到最小。

David McIver 的这篇 [博客文章](https://www.drmaciver.com/2019/01/notes-on-test-case-reduction/) 包含了大量关于如何在实践中应用缩减的见解，特别是不同抽象级别的多次运行。

## 练习

最佳缩减输入的方法仍然是一个研究不充分的研究领域，有很多机会。

### 练习 1：基于变异的模糊测试与缩减

当使用种群进行模糊测试时，偶尔*缩减*每个元素的长度可能很有用，这样未来的后代也会更短，这通常可以加快它们的测试速度。

考虑 基于变异的模糊测试章节 中的 `MutationFuzzer` 类。扩展它，以便每当向种群添加新输入时，它首先使用 delta 调试进行缩减。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Reducer.ipynb#Exercises) 进行练习并查看解决方案。

### 练习 2：通过生产进行缩减

如上所述的基于语法的输入缩减可能是一个好的算法，但绝不是唯一的替代方案。一个有趣的问题是“缩减”是否应该仅限于已经存在的元素，或者是否允许创建*新*元素。这些元素可能不在原始输入中，但仍然可以生成一个更小的输入，从而仍然能够重现原始的失败。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Reducer.ipynb#Exercises) 进行练习并查看解决方案。

例如，考虑以下语法：

```py
<number> ::= <float> | <integer> | <not-a-number>
<float> ::= <digits>.<digits>
<integer> ::= <digits>
<not-a-number> ::= NaN
<digits> ::= [0-9]+
```

假设输入 `100.99` 失败。我们可能能够将其缩减到最小值，例如 `1.9`。然而，我们不能将其缩减到 `<integer>` 或 `<not-a-number>`，因为这些符号在原始输入中不存在。通过允许为这些符号创建*替代*，我们也可以测试如 `1` 或 `NaN` 这样的输入，并进一步泛化程序失败输入的类别。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Reducer.ipynb#Exercises) 进行练习并查看解决方案。

创建一个 `GenerativeGrammarReducer` 类，作为 `GrammarReducer` 的子类；相应地扩展 `reduce_subtree()` 方法。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Reducer.ipynb#Exercises) 进行练习并查看解决方案。

### 练习 3：大规模缩减竞赛

为之前定义的语法创建一个*基准*，包括：

1.  一组*输入*，使用 `GrammarFuzzer` 和其衍生工具从这些语法生成；

1.  一组*测试*，用于检查单个符号以及这些符号的成对和三重组合的出现：

    +   如果输入不是语法有效的，测试应该*未解决*；

    +   如果符号（或其成对或三重组合）出现，测试应该*失败*；

    +   在所有其他情况下，测试应该*通过*。

在基准测试中比较 delta 调试和基于语法的调试。实现 HDD [[Misherghi 等人，2006](https://doi.org/10.1145/1134285.1134307)] 和 *广义树缩减* [[Herfert 等人，2017](http://dl.acm.org/citation.cfm?id=3155562.3155669)] 并将它们添加到比较中。哪种方法表现最好，以及在什么情况下？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Reducer.ipynb#Exercises) 来完成练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。 [最后修改：2023-11-11 18:18:06+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/Reducer.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[减少故障诱导输入](https://www.fuzzingbook.org/html/Reducer.html)". 在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler, "[模糊测试书](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/Reducer.html`](https://www.fuzzingbook.org/html/Reducer.html). Retrieved 2023-11-11 18:18:06+01:00.

```py
@incollection{fuzzingbook2023:Reducer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Reducing Failure-Inducing Inputs},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/Reducer.html}},
    note = {Retrieved 2023-11-11 18:18:06+01:00},
    url = {https://www.fuzzingbook.org/html/Reducer.html},
    urldate = {2023-11-11 18:18:06+01:00}
}

```
