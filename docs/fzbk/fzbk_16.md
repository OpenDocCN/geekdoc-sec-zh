# 语法覆盖率

> 原文：[`www.fuzzingbook.org/html/GrammarCoverageFuzzer.html`](http://www.fuzzingbook.org/html/GrammarCoverageFuzzer.html)

从语法生成输入 给出规则的所有可能的扩展相同的可能性。然而，为了生成一个全面的测试套件，最大化 *多样性* 更有意义——例如，通过不重复相同的扩展。在本章中，我们探讨了如何系统地 *覆盖* 语法元素，以最大化多样性，并且不遗漏个别元素。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('GGb3e5p0HC8') 
```

**先决条件**

+   您应该已经阅读了 关于语法的章节。

+   您应该已经阅读了 关于高效语法模糊的章节。

## 概述

要 使用本章提供的代码，请编写

```py
>>> from fuzzingbook.GrammarCoverageFuzzer import <identifier> 
```

然后利用以下功能。

本章介绍了 `GrammarCoverageFuzzer`，这是一个高效的语法模糊器，它扩展了来自关于高效语法模糊的章节的 `GrammarFuzzer`。它力求至少覆盖所有扩展一次，从而确保功能的覆盖率。

在以下示例中，例如，我们使用 `GrammarCoverageFuzzer` 生成一个表达式。我们看到生成的表达式覆盖了单个表达式中的所有数字和所有运算符。

```py
>>> from Grammars import EXPR_GRAMMAR
>>> expr_fuzzer = GrammarCoverageFuzzer(EXPR_GRAMMAR)
>>> expr_fuzzer.fuzz()
'-(2 + 3) * 4.5 / 6 - 2.0 / +8 + 7 + 3' 
```

模糊后，`expansion_coverage()` 方法返回一个语法扩展覆盖的映射。

```py
>>> expr_fuzzer.expansion_coverage()
{'<digit> -> 0',
 '<digit> -> 1',
 '<digit> -> 2',
 '<digit> -> 3',
 '<digit> -> 4',
 '<digit> -> 5',
 '<digit> -> 6',
 '<digit> -> 7',
 '<digit> -> 8',
 '<digit> -> 9',
 '<expr> -> <term>',
 '<expr> -> <term> + <expr>',
 '<expr> -> <term> - <expr>',
 '<factor> -> (<expr>)',
 '<factor> -> +<factor>',
 '<factor> -> -<factor>',
 '<factor> -> <integer>',
 '<factor> -> <integer>.<integer>',
 '<integer> -> <digit>',
 '<integer> -> <digit><integer>',
 '<start> -> <expr>',
 '<term> -> <factor>',
 '<term> -> <factor> * <term>',
 '<term> -> <factor> / <term>'} 
```

后续对 `fuzz()` 的调用将寻求进一步的覆盖（即，覆盖其他区域代码数字，例如）；对 `reset()` 的调用清除记录的覆盖，重新开始。

由于这种覆盖率在输入中也产生了更高的代码覆盖率，因此 `GrammarCoverageFuzzer` 是 `GrammarFuzzer` 的一个推荐扩展。

<svg width="346pt" height="582pt" viewBox="0.00 0.00 345.75 582.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 578)"><g id="node1" class="node"><title>GrammarCoverageFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class GrammarCoverageFuzzer:

从语法生成，旨在覆盖所有扩展。《GrammarCoverageFuzzer》<text text-anchor="start" x="35.38" y="-69.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarCoverageFuzzer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="GrammarCoverageFuzzer"><g id="a_node1_1"><a xlink:href="#" xlink:title="_new_child_coverage(self, children: List[DerivationTree], max_depth: Union[int, float]) -> Set[str]"><text text-anchor="start" x="45.12" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">_new_child_coverage()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="choose_node_expansion(self, node: DerivationTree, children_alternatives: List[List[DerivationTree]]) -> int:

在 `children_alternatives` 中选择 `node` 的一个扩展。

返回 `n`，使得扩展 `children_alternatives[n]`

产生最高的额外覆盖范围。"><text text-anchor="start" x="45.12" y="-34.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">choose_node_expansion()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="new_child_coverage(self, symbol: str, children: List[DerivationTree], max_depth: Union[int, float] = inf) -> Set[str]:

返回将获得的新覆盖范围

通过扩展 (`symbol`, `children`)"><text text-anchor="start" x="45.12" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">new_child_coverage()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="new_coverages(self, node: DerivationTree, children_alternatives: List[List[DerivationTree]]) -> Optional[List[Set[str]]]:

返回每个子节点在最小深度下的覆盖范围。《text text-anchor="start" x="45.12" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">new_coverages()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>SimpleGrammarCoverageFuzzer</title> <g id="a_node2"><a xlink:href="#" xlink:title="class SimpleGrammarCoverageFuzzer:

在选择扩展时，优先选择未被覆盖的扩展。"><text text-anchor="start" x="14.75" y="-178.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">SimpleGrammarCoverageFuzzer</text> <g id="a_node2_5"><a xlink:href="#" xlink:title="SimpleGrammarCoverageFuzzer"><g id="a_node2_6"><a xlink:href="#" xlink:title="choose_covered_node_expansion(self, node: DerivationTree, children_alternatives: List[List[DerivationTree]]) -> int:

返回在 `_covered_` `children_alternatives` 中的扩展索引

需要被选择。

在子类中需要重载。"><text text-anchor="start" x="15.12" y="-156.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">choose_covered_node_expansion()</text></a></g> <g id="a_node2_7"><a xlink:href="#" xlink:title="choose_node_expansion(self, node: DerivationTree, children_alternatives: List[List[DerivationTree]]) -> int:

返回 `children_alternatives` 中要选择的扩展索引。

如果有，选择未被覆盖的扩展。"><text text-anchor="start" x="15.12" y="-143.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">choose_node_expansion()</text></a></g> <g id="a_node2_8"><a xlink:href="#" xlink:title="choose_uncovered_node_expansion(self, node: DerivationTree, children_alternatives: List[List[DerivationTree]]) -> int:

返回在 `_uncovered_` `children_alternatives` 中的扩展索引

需要被选择。

在子类中重载。"><text text-anchor="start" x="15.12" y="-130.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">choose_uncovered_node_expansion()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>GrammarCoverageFuzzer->SimpleGrammarCoverageFuzzer</title></g> <g id="node3" class="node"><title>TrackingGrammarCoverageFuzzer</title> <g id="a_node3"><a xlink:href="#" xlink:title="class TrackingGrammarCoverageFuzzer:

在生产过程中跟踪语法覆盖率"><text text-anchor="start" x="8" y="-351.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">TrackingGrammarCoverageFuzzer</text> <g id="a_node3_9"><a xlink:href="#" xlink:title="TrackingGrammarCoverageFuzzer"><g id="a_node3_10"><a xlink:href="#" xlink:title="__init__(self, *args, **kwargs) -> None:

从`grammar`生成字符串，从`start_symbol`开始。

如果提供了`min_nonterminals`或`max_nonterminals`，则使用它们作为限制

计算产生的非终结符数量。

如果设置了`disp`，则显示中间推导树。

如果设置了`log`，则将中间步骤以文本形式显示在标准输出上。"><text text-anchor="start" x="30.12" y="-329.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node3_11"><a xlink:href="#" xlink:title="expansion_coverage(self) -> Set[str]:

返回作为字符串的已覆盖扩展集 SYMBOL -> EXPANSION"><text text-anchor="start" x="30.12" y="-316.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">expansion_coverage()</text></a></g> <g id="a_node3_12"><a xlink:href="#" xlink:title="max_expansion_coverage(self, symbol: Optional[str] = None, max_depth: Union[int, float] = inf) -> Set[str]:

返回语法中所有扩展的集合

从`symbol`开始（默认：起始符号）。

如果提供了`max_depth`，则仅扩展到该深度。"><text text-anchor="start" x="30.12" y="-303.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">max_expansion_coverage()</text></a></g> <g id="a_node3_13"><a xlink:href="#" xlink:title="missing_expansion_coverage(self) -> Set[str]:

返回尚未覆盖的扩展"><text text-anchor="start" x="30.12" y="-291" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">missing_expansion_coverage()</text></a></g> <g id="a_node3_14"><a xlink:href="#" xlink:title="reset_coverage(self) -> None:

清除迄今为止跟踪的覆盖信息"><text text-anchor="start" x="30.12" y="-278.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">reset_coverage()</text></a></g> <g id="a_node3_15"><a xlink:href="#" xlink:title="_max_expansion_coverage(self, symbol: str, max_depth: Union[int, float]) -> Set[str]"><text text-anchor="start" x="30.12" y="-264.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">_max_expansion_coverage()</text></a></g> <g id="a_node3_16"><a xlink:href="#" xlink:title="add_coverage(self, symbol: str, new_child: Union[str, Tuple[str, Dict[str, Any]], List[DerivationTree]]) -> None"><text text-anchor="start" x="30.12" y="-251.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">add_coverage()</text></a></g> <g id="a_node3_17"><a xlink:href="#" xlink:title="choose_node_expansion(self, node: DerivationTree, children_alternatives: List[List[DerivationTree]]) -> int:

返回在`children_alternatives`中要选择的扩展的索引。

'children_alternatives'：`node`的可能子节点列表。

默认为随机。在子类中重载。"><text text-anchor="start" x="30.12" y="-240" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">choose_node_expansion()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>SimpleGrammarCoverageFuzzer->TrackingGrammarCoverageFuzzer</title></g> <g id="node4" class="node"><title>GrammarFuzzer</title> <g id="a_node4"><a xlink:href="GrammarFuzzer.html" xlink:title="class GrammarFuzzer:

使用推导树高效地生成语法字符串。"><text text-anchor="start" x="64.25" y="-460.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarFuzzer</text> <g id="a_node4_18"><a xlink:href="#" xlink:title="GrammarFuzzer"><g id="a_node4_19"><a xlink:href="GrammarFuzzer.html" xlink:title="__init__(self, grammar: Dict[str, List[Expansion]], start_symbol: str = '<start>', min_nonterminals: int = 0, max_nonterminals: int = 10, disp: bool = False, log: Union[bool, int] = False) -> None:

从`grammar`生成字符串，从`start_symbol`开始。

如果提供了`min_nonterminals`或`max_nonterminals`，则使用它们作为限制。

对于产生的非终结符数量。

如果`disp`被设置，则显示中间的推导树。

如果`log`被设置，则将中间步骤作为文本显示在标准输出上。"><text text-anchor="start" x="81.12" y="-438.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node4_20"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz(self) -> str:

从语法生成字符串。</text></a></g> <g id="a_node4_21"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz_tree(self) -> DerivationTree:

从语法生成推导树。</text></a></g></a></g></a></g></g> <g id="edge3" class="edge"><title>TrackingGrammarCoverageFuzzer->GrammarFuzzer</title></g> <g id="node5" class="node"><title>Fuzzer</title> <g id="a_node5"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊测试器的基类。</text> <g id="a_node5_22"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node5_23"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

使用模糊输入运行 `runner` 的 `run()` 方法：</text></a></g> <g id="a_node5_24"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

使用模糊输入运行`runner`，`trials`次"><text text-anchor="start" x="96.12" y="-522.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">runs()</text></a></g></a></g></a></g></g> <g id="edge4" class="edge"><title>GrammarFuzzer->Fuzzer</title></g> <g id="node6" class="node"><title>图例</title> <text text-anchor="start" x="218.5" y="-59" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="218.5" y="-49" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="224.5" y="-49" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="218.5" y="-39" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="224.5" y="-39" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="218.5" y="-29" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="224.5" y="-29" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="218.5" y="-19.95" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 覆盖语法元素

测试生成的目的是覆盖程序的所有功能——当然，希望包括失败的功能。然而，这种功能与*输入结构*相关：如果我们无法生成某些输入元素，那么相关的代码和功能也不会被触发，从而消除了我们在那里找到错误的机会。

例如，考虑我们表达式语法`EXPR_GRAMMAR`来自关于语法的章节：

+   如果我们不生成负数，则不会测试负数。

+   如果我们不生成浮点数，则不会测试浮点数。

因此，我们的目标必须是*覆盖所有可能的扩展*——而且不仅仅是偶然的，而是有计划的。

要最大化这种多样性的一种方法是通过*跟踪*语法生成过程中发生的扩展：如果我们已经看到了一些扩展，我们就可以从可能的扩展集中优先考虑其他可能的扩展候选者。考虑我们表达式语法中的以下规则：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import quiz 
```

```py
from Fuzzer import Fuzzer 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import Dict, List, Set, Union, Optional 
```

```py
from Grammars import EXPR_GRAMMAR, CGI_GRAMMAR, URL_GRAMMAR, START_SYMBOL
from Grammars import is_valid_grammar, extend_grammar, Grammar 
```

```py
EXPR_GRAMMAR["<factor>"] 
```

```py
['+<factor>', '-<factor>', '(<expr>)', '<integer>.<integer>', '<integer>']

```

假设我们已经在前一个 `<factor>` 的扩展中产生了一个 `<integer>`。当扩展下一个因子时，我们会标记 `<integer>` 扩展为已覆盖，并选择尚未覆盖的替代方案之一，例如 `-<factor>`（一个负数）或 `<integer>.<integer>`（一个浮点数）。只有当我们覆盖了所有替代方案后，我们才会回过头来重新考虑之前覆盖的扩展。

### 问答

表达式 `1 + 2` 覆盖了 `EXPR_GRAMMAR` 的哪些扩展？

的确！该表达式从 `<start>` 扩展到单个数字。

### 跟踪语法覆盖率

这种 *语法覆盖率* 的概念很容易实现。我们引入一个类 `TrackingGrammarCoverageFuzzer`，它跟踪当前实现的语法覆盖率：

```py
from Grammars import Grammar, Expansion
from GrammarFuzzer import GrammarFuzzer, all_terminals, nonterminals, \
    display_tree, DerivationTree 
```

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
class TrackingGrammarCoverageFuzzer(GrammarFuzzer):
  """Track grammar coverage during production"""

    def __init__(self, *args, **kwargs) -> None:
        # invoke superclass __init__(), passing all arguments
        super().__init__(*args, **kwargs)
        self.reset_coverage() 
```

#### 跟踪扩展

在集合 `covered_expansions` 中，我们存储看到的单个扩展。

```py
class TrackingGrammarCoverageFuzzer(TrackingGrammarCoverageFuzzer):
    def expansion_coverage(self) -> Set[str]:
  """Return the set of covered expansions as strings SYMBOL -> EXPANSION"""
        return self.covered_expansions

    def reset_coverage(self) -> None:
  """Clear coverage info tracked so far"""
        self.covered_expansions: Set[str] = set() 
```

我们将扩展保存为字符串 "*符号* -> *扩展*"，使用函数 `expansion_key()` 为 (*符号*, *扩展*) 对生成字符串表示。

```py
def expansion_key(symbol: str, 
                  expansion: Union[Expansion,
                                   DerivationTree, 
                                   List[DerivationTree]]) -> str:
  """Convert (symbol, `expansion`) into a key "SYMBOL -> EXPRESSION". 
 `expansion` can be an expansion string, a derivation tree,
 or a list of derivation trees."""

    if isinstance(expansion, tuple):
        # Expansion or single derivation tree
        expansion, _ = expansion

    if not isinstance(expansion, str):
        # Derivation tree
        children = expansion
        expansion = all_terminals((symbol, children))

    assert isinstance(expansion, str)

    return symbol + " -> " + expansion 
```

这里有一个例子：

```py
expansion_key(START_SYMBOL, EXPR_GRAMMAR[START_SYMBOL][0]) 
```

```py
'<start> -> <expr>'

```

除了 *扩展*，我们还可以传递一个子节点列表作为参数，然后它将自动转换为字符串。

```py
children: List[DerivationTree] = [("<expr>", None), (" + ", []), ("<term>", None)]
expansion_key("<expr>", children) 
```

```py
'<expr> -> <expr> + <term>'

```

#### 计算可能的扩展

我们可以通过枚举所有扩展来计算语法中的可能扩展集。方法 `max_expansion_coverage()` 从给定的符号（默认：语法起始符号）递归遍历语法，并将所有扩展累积到集合 `expansions` 中。通过 `max_depth` 参数（默认：$\infty$），我们可以控制语法探索的深度；我们将在本章后面需要它。

```py
class TrackingGrammarCoverageFuzzer(TrackingGrammarCoverageFuzzer):
    def _max_expansion_coverage(self, symbol: str, 
                                max_depth: Union[int, float]) -> Set[str]:
        if max_depth <= 0:
            return set()

        self._symbols_seen.add(symbol)

        expansions = set()
        for expansion in self.grammar[symbol]:
            expansions.add(expansion_key(symbol, expansion))
            for nonterminal in nonterminals(expansion):
                if nonterminal not in self._symbols_seen:
                    expansions |= self._max_expansion_coverage(
                        nonterminal, max_depth - 1)

        return expansions

    def max_expansion_coverage(self, symbol: Optional[str] = None,
                               max_depth: Union[int, float] = float('inf')) \
            -> Set[str]:
  """Return set of all expansions in a grammar 
 starting with `symbol` (default: start symbol).
 If `max_depth` is given, expand only to that depth."""
        if symbol is None:
            symbol = self.start_symbol

        self._symbols_seen: Set[str] = set()
        cov = self._max_expansion_coverage(symbol, max_depth)

        if symbol == START_SYMBOL:
            assert len(self._symbols_seen) == len(self.grammar)

        return cov 
```

我们可以使用 `max_expansion_coverage()` 来计算表达式语法中的所有扩展：

```py
expr_fuzzer = TrackingGrammarCoverageFuzzer(EXPR_GRAMMAR)
expr_fuzzer.max_expansion_coverage() 
```

```py
{'<digit> -> 0',
 '<digit> -> 1',
 '<digit> -> 2',
 '<digit> -> 3',
 '<digit> -> 4',
 '<digit> -> 5',
 '<digit> -> 6',
 '<digit> -> 7',
 '<digit> -> 8',
 '<digit> -> 9',
 '<expr> -> <term>',
 '<expr> -> <term> + <expr>',
 '<expr> -> <term> - <expr>',
 '<factor> -> (<expr>)',
 '<factor> -> +<factor>',
 '<factor> -> -<factor>',
 '<factor> -> <integer>',
 '<factor> -> <integer>.<integer>',
 '<integer> -> <digit>',
 '<integer> -> <digit><integer>',
 '<start> -> <expr>',
 '<term> -> <factor>',
 '<term> -> <factor> * <term>',
 '<term> -> <factor> / <term>'}

```

#### 在模糊过程中跟踪扩展

在扩展过程中，我们可以跟踪已看到的扩展。为此，我们钩入方法 `choose_node_expansion()`，在语法模糊器中扩展单个节点。

```py
class TrackingGrammarCoverageFuzzer(TrackingGrammarCoverageFuzzer):
    def add_coverage(self, symbol: str,
                     new_child: Union[Expansion, List[DerivationTree]]) -> None:
        key = expansion_key(symbol, new_child)

        if self.log and key not in self.covered_expansions:
            print("Now covered:", key)
        self.covered_expansions.add(key)

    def choose_node_expansion(self, node: DerivationTree,
                              children_alternatives: 
                              List[List[DerivationTree]]) -> int:
        (symbol, children) = node
        index = super().choose_node_expansion(node, children_alternatives)
        self.add_coverage(symbol, children_alternatives[index])
        return index 
```

方法 `missing_expansion_coverage()` 是一个辅助方法，它返回仍需覆盖的扩展：

```py
class TrackingGrammarCoverageFuzzer(TrackingGrammarCoverageFuzzer):
    def missing_expansion_coverage(self) -> Set[str]:
  """Return expansions not covered yet"""
        return self.max_expansion_coverage() - self.expansion_coverage() 
```

#### 将事物组合在一起

让我们展示跟踪是如何工作的。为了使事情简单，让我们只关注 `<digit>` 扩展。

```py
digit_fuzzer = TrackingGrammarCoverageFuzzer(
    EXPR_GRAMMAR, start_symbol="<digit>", log=True)
digit_fuzzer.fuzz() 
```

```py
Tree: <digit>
Expanding <digit> randomly
Now covered: <digit> -> 9
Tree: 9
'9'

```

```py
'9'

```

```py
digit_fuzzer.fuzz() 
```

```py
Tree: <digit>
Expanding <digit> randomly
Now covered: <digit> -> 0
Tree: 0
'0'

```

```py
'0'

```

```py
digit_fuzzer.fuzz() 
```

```py
Tree: <digit>
Expanding <digit> randomly
Now covered: <digit> -> 5
Tree: 5
'5'

```

```py
'5'

```

这里是迄今为止覆盖的扩展集合：

```py
digit_fuzzer.expansion_coverage() 
```

```py
{'<digit> -> 0', '<digit> -> 5', '<digit> -> 9'}

```

这是我们可以覆盖的所有扩展集合：

```py
digit_fuzzer.max_expansion_coverage() 
```

```py
{'<digit> -> 0',
 '<digit> -> 1',
 '<digit> -> 2',
 '<digit> -> 3',
 '<digit> -> 4',
 '<digit> -> 5',
 '<digit> -> 6',
 '<digit> -> 7',
 '<digit> -> 8',
 '<digit> -> 9'}

```

这是缺失的覆盖率：

```py
digit_fuzzer.missing_expansion_coverage() 
```

```py
{'<digit> -> 1',
 '<digit> -> 2',
 '<digit> -> 3',
 '<digit> -> 4',
 '<digit> -> 6',
 '<digit> -> 7',
 '<digit> -> 8'}

```

平均来说，我们需要产生多少个字符才能覆盖所有扩展？

```py
def average_length_until_full_coverage(fuzzer: TrackingGrammarCoverageFuzzer) -> float:
    trials = 50

    sum = 0
    for trial in range(trials):
        # print(trial, end=" ")
        fuzzer.reset_coverage()
        while len(fuzzer.missing_expansion_coverage()) > 0:
            s = fuzzer.fuzz()
            sum += len(s)

    return sum / trials 
```

```py
digit_fuzzer.log = False
average_length_until_full_coverage(digit_fuzzer) 
```

```py
28.4

```

对于完整表达式，这需要更长的时间：

```py
expr_fuzzer = TrackingGrammarCoverageFuzzer(EXPR_GRAMMAR)
average_length_until_full_coverage(expr_fuzzer) 
```

```py
138.12

```

### 覆盖语法扩展

现在，让我们不仅跟踪覆盖率，而且实际 *产生* 覆盖率。思路如下：

1.  我们确定尚未覆盖的子节点（在 `uncovered_children` 中）

1.  如果所有子节点都已覆盖，我们将回退到原始方法（即随机选择一个扩展）

1.  否则，我们从未覆盖的子节点中选择一个子节点并将其标记为已覆盖。

为了这个目的，我们引入了一个新的模糊器 `SimpleGrammarCoverageFuzzer`，它在 `choose_node_expansion()` 方法中实现了这个策略——这是`GrammarFuzzer`超类用来选择要扩展的子类的方法。

```py
class SimpleGrammarCoverageFuzzer(TrackingGrammarCoverageFuzzer):
  """When choosing expansions, prefer expansions not covered."""

    def choose_node_expansion(self,
                              node: DerivationTree,
                              children_alternatives: List[List[DerivationTree]]) -> int:
  """Return index of expansion in `children_alternatives` to be selected.
 Picks uncovered expansions, if any."""

        # Prefer uncovered expansions
        (symbol, children) = node
        uncovered_children = [c for (i, c) in enumerate(children_alternatives)
                              if expansion_key(symbol, c)
                              not in self.covered_expansions]
        index_map = [i for (i, c) in enumerate(children_alternatives)
                     if c in uncovered_children]

        if len(uncovered_children) == 0:
            # All expansions covered - use superclass method
            return self.choose_covered_node_expansion(node, children_alternatives)

        # Select from uncovered nodes
        index = self.choose_uncovered_node_expansion(node, uncovered_children)

        return index_map[index] 
```

提供了两种方法 `choose_covered_node_expansion()` 和 `choose_uncovered_node_expansion()`，供子类钩子使用：

```py
class SimpleGrammarCoverageFuzzer(SimpleGrammarCoverageFuzzer):
    def choose_uncovered_node_expansion(self,
                                        node: DerivationTree,
                                        children_alternatives: List[List[DerivationTree]]) \
            -> int:
  """Return index of expansion in _uncovered_ `children_alternatives`
 to be selected.
 To be overloaded in subclasses."""
        return TrackingGrammarCoverageFuzzer.choose_node_expansion(
            self, node, children_alternatives)

    def choose_covered_node_expansion(self,
                                      node: DerivationTree,
                                      children_alternatives: List[List[DerivationTree]]) \
            -> int:
  """Return index of expansion in _covered_ `children_alternatives`
 to be selected.
 To be overloaded in subclasses."""
        return TrackingGrammarCoverageFuzzer.choose_node_expansion(
            self, node, children_alternatives) 
```

通过返回迄今为止覆盖的扩展集，我们可以多次调用模糊器，每次都增加语法覆盖率。例如，使用 `EXPR_GRAMMAR` 语法生成数字，模糊器会一个接一个地生成数字：

```py
f = SimpleGrammarCoverageFuzzer(EXPR_GRAMMAR, start_symbol="<digit>")
f.fuzz() 
```

```py
'5'

```

```py
f.fuzz() 
```

```py
'2'

```

```py
f.fuzz() 
```

```py
'1'

```

到目前为止，这是已覆盖的扩展集：

```py
f.expansion_coverage() 
```

```py
{'<digit> -> 1', '<digit> -> 2', '<digit> -> 5'}

```

让我们再模糊一些。我们看到，随着每次迭代，我们都会覆盖另一个扩展：

```py
for i in range(7):
    print(f.fuzz(), end=" ") 
```

```py
0 9 7 4 8 3 6 

```

最后，所有扩展都已覆盖：

```py
f.missing_expansion_coverage() 
```

```py
set()

```

让我们在一个更复杂的语法上应用这个——例如，完整的表达式语法。我们看到，经过几次迭代，我们覆盖了每个数字、运算符和扩展：

```py
f = SimpleGrammarCoverageFuzzer(EXPR_GRAMMAR)
for i in range(10):
    print(f.fuzz()) 
```

```py
+(0.31 / (5) / 9 + 4 * 6 / 3 - 8 - 7) * -2
+++2 / 87360
((4) * 0 - 1) / -9.6 + 7 / 6 + 1 * 8 + 7 * 8
++++26 / -64.45
(8 / 1 / 6 + 9 + 7 + 8) * 1.1 / 0 * 1
7.7
++(3.5 / 3) - (-4 + 3) / (8 / 0) / -4 * 2 / 1
+(90 / --(28 * 8 / 5 + 5 / (5 / 8))) - +9.36 / 2.5 * (5 * (7 * 6 * 5) / 8)
9.11 / 7.28
1 / (9 - 5 * 6) / 6 / 7 / 7 + 1 + 1 - 7 * -3

```

再次，所有扩展都已覆盖：

```py
f.missing_expansion_coverage() 
```

```py
set()

```

我们看到，我们的策略在实现覆盖率方面比随机方法更有效：

```py
average_length_until_full_coverage(SimpleGrammarCoverageFuzzer(EXPR_GRAMMAR)) 
```

```py
52.28

```

## 深度预见

为单个规则选择扩展是一个好的开始；然而，正如以下示例所示，这还不够。我们对来自语法章节的 CGI 语法应用我们的覆盖率模糊器：

```py
CGI_GRAMMAR 
```

```py
{'<start>': ['<string>'],
 '<string>': ['<letter>', '<letter><string>'],
 '<letter>': ['<plus>', '<percent>', '<other>'],
 '<plus>': ['+'],
 '<percent>': ['%<hexdigit><hexdigit>'],
 '<hexdigit>': ['0',
  '1',
  '2',
  '3',
  '4',
  '5',
  '6',
  '7',
  '8',
  '9',
  'a',
  'b',
  'c',
  'd',
  'e',
  'f'],
 '<other>': ['0', '1', '2', '3', '4', '5', 'a', 'b', 'c', 'd', 'e', '-', '_']}

```

```py
f = SimpleGrammarCoverageFuzzer(CGI_GRAMMAR)
for i in range(10):
    print(f.fuzz()) 
```

```py
c
+%a6++
+-
+
++
%18%b7
+e
_
d2+%e3
%d0

```

经过 10 次迭代后，我们仍然有一些扩展未被覆盖：

```py
f.missing_expansion_coverage() 
```

```py
{'<hexdigit> -> 2',
 '<hexdigit> -> 4',
 '<hexdigit> -> 5',
 '<hexdigit> -> 9',
 '<hexdigit> -> c',
 '<hexdigit> -> f',
 '<other> -> 0',
 '<other> -> 1',
 '<other> -> 3',
 '<other> -> 4',
 '<other> -> 5',
 '<other> -> a',
 '<other> -> b'}

```

为什么会这样？问题是，在 CGI 语法中，要覆盖的最大变化数发生在 `hexdigit` 规则中。然而，我们首先需要*到达*这个扩展。当扩展 `<letter>` 符号时，我们有三种可能的扩展选择：

```py
CGI_GRAMMAR["<letter>"] 
```

```py
['<plus>', '<percent>', '<other>']

```

如果所有三个扩展都已经覆盖，那么上面的 `choose_node_expansion()` 将随机选择一个——即使在选择 `<percent>` 时可能还有更多扩展需要覆盖。

我们需要的是一个更好的策略，如果之后还有更多未覆盖的扩展，它会选择 `<percent>`——即使 `<percent>` 已经被覆盖。这种策略最初由 W. Burkhardt [[Burkhardt *et al*, 1967](https://doi.org/10.1007/BF02235512)] 以“最短路径选择”的名义讨论。

> 这个版本从几个开发选择中选择了，在其中有未使用的单位可用，从最短路径开始。

这是我们将在下一步中实现的内容。

### 确定最大符号覆盖率

为了解决这个问题，我们引入了一个新的类 `GrammarCoverageFuzzer`，它基于 `SimpleGrammarCoverageFuzzer`，但具有*更好的策略*。首先，我们需要计算从特定符号可以到达的*最大扩展集*，正如我们在 `max_expansion_coverage()` 中已经实现的。想法是稍后计算这个集和已覆盖扩展的*交集*，这样我们就可以优先考虑那些具有非空交集的扩展。

第一步——计算从一个符号可以到达的最大展开集——已经实现。通过传递一个 `symbol` 参数给 `max_expansion_coverage()`，我们可以计算每个符号的可能展开：

```py
f = SimpleGrammarCoverageFuzzer(EXPR_GRAMMAR)
f.max_expansion_coverage('<integer>') 
```

```py
{'<digit> -> 0',
 '<digit> -> 1',
 '<digit> -> 2',
 '<digit> -> 3',
 '<digit> -> 4',
 '<digit> -> 5',
 '<digit> -> 6',
 '<digit> -> 7',
 '<digit> -> 8',
 '<digit> -> 9',
 '<integer> -> <digit>',
 '<integer> -> <digit><integer>'}

```

我们看到，通过展开 `<integer>`，我们可以覆盖总共 12 个生成式。

### 问答

`f.max_expansion_coverage('<digit>')` 会返回多少个生成式？

的确。以下是 `<digit>` 所有可能的展开：

```py
f.max_expansion_coverage('<digit>') 
```

```py
{'<digit> -> 0',
 '<digit> -> 1',
 '<digit> -> 2',
 '<digit> -> 3',
 '<digit> -> 4',
 '<digit> -> 5',
 '<digit> -> 6',
 '<digit> -> 7',
 '<digit> -> 8',
 '<digit> -> 9'}

```

### 确定尚未覆盖的子节点

我们现在可以开始实现 `GrammarCoverageFuzzer`。我们的想法是确定每个子节点的 *缺失覆盖率*。

给定一个子节点列表，我们可以使用 `max_expansion_coverage()` 来计算每个子节点的最大覆盖率。从这些中，我们 *减去* 已经看到的覆盖率（`expansion_coverage()`）。这结果是我们可以获得的覆盖率。

```py
class GrammarCoverageFuzzer(SimpleGrammarCoverageFuzzer):
  """Produce from grammars, aiming for coverage of all expansions."""

    def new_child_coverage(self,
                           symbol: str,
                           children: List[DerivationTree],
                           max_depth: Union[int, float] = float('inf')) -> Set[str]:
  """Return new coverage that would be obtained 
 by expanding (`symbol`, `children`)"""

        new_cov = self._new_child_coverage(children, max_depth)
        new_cov.add(expansion_key(symbol, children))
        new_cov -= self.expansion_coverage()   # -= is set subtraction
        return new_cov

    def _new_child_coverage(self, children: List[DerivationTree],
                            max_depth: Union[int, float]) -> Set[str]:
        new_cov: Set[str] = set()
        for (c_symbol, _) in children:
            if c_symbol in self.grammar:
                new_cov |= self.max_expansion_coverage(c_symbol, max_depth)

        return new_cov 
```

让我们说明 `new_child_coverage()`。我们再次开始模糊测试，随机选择展开。

```py
f = GrammarCoverageFuzzer(EXPR_GRAMMAR, start_symbol="<digit>", log=True)
f.fuzz() 
```

```py
Tree: <digit>
Expanding <digit> randomly
Now covered: <digit> -> 2
Tree: 2
'2'

```

```py
'2'

```

这是我们的当前覆盖率：

```py
f.expansion_coverage() 
```

```py
{'<digit> -> 2'}

```

如果我们想将 `<digit>` 展开为 `0`，这将给我们新的覆盖率：

```py
f.new_child_coverage("<digit>", [('0', [])]) 
```

```py
{'<digit> -> 0'}

```

如果我们想再次将 `<digit>` 展开为 `2`，这将给我们 *没有* 新的覆盖率：

```py
f.new_child_coverage("<digit>", [('2', [])]) 
```

```py
set()

```

当我们遍历 `<digit>` 的单个展开可能性时，我们看到所有展开都提供了额外的覆盖率，*除了* 我们已经覆盖的 `2`。

```py
for expansion in EXPR_GRAMMAR["<digit>"]:
    children = f.expansion_to_children(expansion)
    print(expansion, f.new_child_coverage("<digit>", children)) 
```

```py
0 {'<digit> -> 0'}
1 {'<digit> -> 1'}
2 set()
3 {'<digit> -> 3'}
4 {'<digit> -> 4'}
5 {'<digit> -> 5'}
6 {'<digit> -> 6'}
7 {'<digit> -> 7'}
8 {'<digit> -> 8'}
9 {'<digit> -> 9'}

```

这意味着在每次选择展开时，我们可以利用 `new_child_coverage()` 并在提供最大新（未见）覆盖率的展开中选择。

### 自适应前瞻

在选择子节点时，我们并不寻求获得的最大总体覆盖率，因为这会使具有许多未覆盖可能性的展开完全主导其他展开。相反，我们追求一种 *广度优先* 策略，首先覆盖所有给定深度的展开，然后才寻找更深的深度。

`new_coverages()` 方法是这个策略的核心：从最大深度（`max_depth`）为零开始，它增加深度直到找到至少一个未覆盖的展开。

<details id="Excursion:-Implementing-new_coverage()"><summary>实现 `new_coverage()`</summary>

```py
class GrammarCoverageFuzzer(GrammarCoverageFuzzer):
    def new_coverages(self, node: DerivationTree,
                      children_alternatives: List[List[DerivationTree]]) \
            -> Optional[List[Set[str]]]:
  """Return coverage to be obtained for each child at minimum depth"""

        (symbol, children) = node
        for max_depth in range(len(self.grammar)):
            new_coverages = [
                self.new_child_coverage(
                    symbol, c, max_depth) for c in children_alternatives]
            max_new_coverage = max(len(new_coverage)
                                   for new_coverage in new_coverages)
            if max_new_coverage > 0:
                # Uncovered node found
                return new_coverages

        # All covered
        return None 
```</details>

### 全部一起

我们现在可以定义 `choose_node_expansion()` 来利用这种策略：

1.  我们确定要获得的可能覆盖率（使用 `new_coverages()`）。

1.  我们（随机地）在具有最大覆盖率的子节点之间进行选择（使用 `choose_uncovered_node_expansion()`）。

<details id="Excursion:-Implementing-choose_node_expansion()"><summary>实现 `choose_node_expansion()`</summary>

```py
class GrammarCoverageFuzzer(GrammarCoverageFuzzer):
    def choose_node_expansion(self, node: DerivationTree,
                              children_alternatives: List[List[DerivationTree]]) -> int:
  """Choose an expansion of `node` among `children_alternatives`.
 Return `n` such that expanding `children_alternatives[n]`
 yields the highest additional coverage."""

        (symbol, children) = node
        new_coverages = self.new_coverages(node, children_alternatives)

        if new_coverages is None:
            # All expansions covered - use superclass method
            return self.choose_covered_node_expansion(node, children_alternatives)

        max_new_coverage = max(len(cov) for cov in new_coverages)

        children_with_max_new_coverage = [c for (i, c) in enumerate(children_alternatives)
                                          if len(new_coverages[i]) == max_new_coverage]
        index_map = [i for (i, c) in enumerate(children_alternatives)
                     if len(new_coverages[i]) == max_new_coverage]

        # Select a random expansion
        new_children_index = self.choose_uncovered_node_expansion(
            node, children_with_max_new_coverage)
        new_children = children_with_max_new_coverage[new_children_index]

        # Save the expansion as covered
        key = expansion_key(symbol, new_children)

        if self.log:
            print("Now covered:", key)
        self.covered_expansions.add(key)

        return index_map[new_children_index] 
```</details>

现在，我们的 `GrammarCoverageFuzzer` 已经完成！让我们在一系列示例中应用它。在表达式上，它很快就能覆盖所有数字和运算符：

```py
f = GrammarCoverageFuzzer(EXPR_GRAMMAR, min_nonterminals=3)
f.fuzz() 
```

```py
'-4.02 / (1) * +3 + 5.9 / 7 * 8 - 6'

```

```py
f.max_expansion_coverage() - f.expansion_coverage() 
```

```py
set()

```

平均来说，这比简单策略要快：

```py
average_length_until_full_coverage(GrammarCoverageFuzzer(EXPR_GRAMMAR)) 
```

```py
50.74

```

在 CGI 语法上，只需几次迭代就能覆盖所有字母和数字：

```py
f = GrammarCoverageFuzzer(CGI_GRAMMAR, min_nonterminals=5)
while len(f.max_expansion_coverage() - f.expansion_coverage()) > 0:
    print(f.fuzz()) 
```

```py
%18%d03
%c3%94%7f+cd
%a6%b5%e2%5e%4c-54e01a2
%5eb%7cb_ec%a0+

```

这种改进也可以在比较随机、仅展开和深度前瞻策略在 CGI 语法上的效果时看到：

```py
average_length_until_full_coverage(TrackingGrammarCoverageFuzzer(CGI_GRAMMAR)) 
```

```py
211.34

```

```py
average_length_until_full_coverage(SimpleGrammarCoverageFuzzer(CGI_GRAMMAR)) 
```

```py
68.64

```

```py
average_length_until_full_coverage(GrammarCoverageFuzzer(CGI_GRAMMAR)) 
```

```py
40.38

```

## 覆盖率在上下文中

有时候，语法元素不仅仅在一个地方使用。例如，在我们的表达式语法中，`<integer>` 符号既用于整数数字，也用于浮点数字：

```py
EXPR_GRAMMAR["<factor>"] 
```

```py
['+<factor>', '-<factor>', '(<expr>)', '<integer>.<integer>', '<integer>']

```

如上所述，我们的覆盖生产将确保所有 `<integer>` 扩展（即所有 `<digit>` 扩展）都被覆盖。然而，单个数字将分布在语法中 `<integer>` 的所有出现中。如果我们基于覆盖的 fuzzer 产生，例如，`1234.56` 和 `7890`，我们将对所有数字扩展实现全面覆盖。然而，上面的 `<integer>.<integer>` 和 `<factor>` 扩展中的 `<integer>` 将分别只覆盖一部分数字。如果浮点数和整数有不同的函数来读取它们，我们希望每个这样的函数都使用所有数字进行测试；也许我们还想用所有数字测试浮点数的整数部分和小数部分。

忽略符号使用的上下文（在我们的例子中，是 `<factor>` 上下文中 `<integer>` 和 `<digit>` 的各种使用）可能是有用的，如果我们假设这个符号的所有出现都被同等对待的话。如果不这样，确保符号的出现系统地被独立于其他出现覆盖的一种方法是将出现分配给一个新的符号，这个新符号是旧符号的副本。我们首先将展示如何手动创建这样的副本，然后是一个自动执行此操作的专用函数。

### 通过手动扩展语法以实现上下文覆盖

如上所述，通过复制符号以及它们引用的规则，这是一种简单的方法来实现上下文覆盖。例如，我们可以将 `<integer>.<integer>` 替换为 `<integer-1>.<integer-2>`，并给 `<integer-1>` 和 `<integer-2>` 赋予与原始 `<integer>` 相同的定义。这意味着不仅 `<integer>` 的所有扩展，还包括 `<integer-1>` 和 `<integer-2>` 的所有扩展都将被覆盖。

让我们用实际的代码来说明这一点：

```py
dup_expr_grammar = extend_grammar(EXPR_GRAMMAR,
                                  {
                                      "<factor>": ["+<factor>", "-<factor>", "(<expr>)", "<integer-1>.<integer-2>", "<integer>"],
                                      "<integer-1>": ["<digit-1><integer-1>", "<digit-1>"],
                                      "<integer-2>": ["<digit-2><integer-2>", "<digit-2>"],
                                      "<digit-1>":
                                      ["0", "1", "2", "3", "4",
                                          "5", "6", "7", "8", "9"],
                                      "<digit-2>":
                                      ["0", "1", "2", "3", "4",
                                          "5", "6", "7", "8", "9"]
                                  }
                                  ) 
```

```py
assert is_valid_grammar(dup_expr_grammar) 
```

如果我们现在在扩展的语法上运行基于覆盖的 fuzzer，我们将覆盖所有数字，包括常规整数中的所有数字，以及浮点数的整数部分和小数部分中的所有数字：

```py
f = GrammarCoverageFuzzer(dup_expr_grammar, start_symbol="<factor>")
for i in range(10):
    print(f.fuzz()) 
```

```py
-(43.76 / 8.0 * 5.5 / 6.9 * 6 / 4 + +03)
(90.1 - 1 * 7.3 * 9 + 5 / 8 / 7)
2.8
1.2
10.4
2
4386
7
0
08929.4302

```

我们看到我们的“预见性”覆盖 fuzzer 如何专门生成覆盖整数部分和小数部分所有数字的浮点数。

### 通过程序扩展语法以实现上下文覆盖

如果我们想在上下文中增强覆盖范围，手动调整我们的语法可能不是最佳选择，因为任何对语法的更改都必须在所有副本中重复。相反，我们引入了一个函数，它会为我们执行复制。

函数 `duplicate_context()` 接收一个语法、语法中的一个符号以及该符号的扩展（`None` 或未提供：符号的所有扩展），并将扩展更改为引用所有原始引用的规则的副本。我们的想法是调用它如下：

```py
dup_expr_grammar = extend_grammar(EXPR_GRAMMAR)
duplicate_context(dup_expr_grammar, "<factor>", "<integer>.<integer>") 
```

并得到与上面手动更改相似的结果。

下面是代码：

```py
from Grammars import new_symbol, unreachable_nonterminals
from GrammarFuzzer import expansion_to_children 
```

```py
def duplicate_context(grammar: Grammar, 
                      symbol: str,
                      expansion: Optional[Expansion] = None, 
                      depth: Union[float, int] = float('inf')):
  """Duplicate an expansion within a grammar.

 In the given grammar, take the given expansion of the given `symbol`
 (if `expansion` is omitted: all symbols), and replace it with a
 new expansion referring to a duplicate of all originally referenced rules.

 If `depth` is given, limit duplication to `depth` references
 (default: unlimited)
 """
    orig_grammar = extend_grammar(grammar)
    _duplicate_context(grammar, orig_grammar, symbol,
                       expansion, depth, seen={})

    # After duplication, we may have unreachable rules; delete them
    for nonterminal in unreachable_nonterminals(grammar):
        del grammar[nonterminal] 
```

<details id="Excursion:-Implementing-_duplicate_context()"><summary>实现 `_duplicate_context()`</summary>

大部分工作都发生在这个辅助函数中。额外的参数 `seen` 跟踪已经展开的符号，以避免无限递归。

```py
import [copy](https://docs.python.org/3/library/copy.html) 
```

```py
def _duplicate_context(grammar: Grammar,
                       orig_grammar: Grammar,
                       symbol: str,
                       expansion: Optional[Expansion],
                       depth: Union[float, int],
                       seen: Dict[str, str]) -> None:
  """Helper function for `duplicate_context()`"""

    for i in range(len(grammar[symbol])):
        if expansion is None or grammar[symbol][i] == expansion:
            new_expansion = ""
            for (s, c) in expansion_to_children(grammar[symbol][i]):
                if s in seen:                 # Duplicated already
                    new_expansion += seen[s]
                elif c == [] or depth == 0:   # Terminal symbol or end of recursion
                    new_expansion += s
                else:                         # Nonterminal symbol - duplicate
                    # Add new symbol with copy of rule
                    new_s = new_symbol(grammar, s)
                    grammar[new_s] = copy.deepcopy(orig_grammar[s])

                    # Duplicate its expansions recursively
                    # {**seen, **{s: new_s}} is seen + {s: new_s}
                    _duplicate_context(grammar, orig_grammar, new_s, expansion=None,
                                       depth=depth - 1, seen={**seen, **{s: new_s}})
                    new_expansion += new_s

            grammar[symbol][i] = new_expansion 
```</details>

下面是我们的上述示例，展示了 `duplicate_context()` 的工作原理，现在有了结果。我们让它重复我们的表达式语法中的 `<integer>.<integer>` 展开式，并得到一个新的语法，其中 `<integer-1>` 和 `<integer-2>` 都指的是原始规则的副本：

```py
dup_expr_grammar = extend_grammar(EXPR_GRAMMAR)
duplicate_context(dup_expr_grammar, "<factor>", "<integer>.<integer>")
dup_expr_grammar 
```

```py
{'<start>': ['<expr>'],
 '<expr>': ['<term> + <expr>', '<term> - <expr>', '<term>'],
 '<term>': ['<factor> * <term>', '<factor> / <term>', '<factor>'],
 '<factor>': ['+<factor>',
  '-<factor>',
  '(<expr>)',
  '<integer-1>.<integer-2>',
  '<integer>'],
 '<integer>': ['<digit><integer>', '<digit>'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<integer-1>': ['<digit-1><integer-1>', '<digit-2>'],
 '<digit-1>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<digit-2>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<integer-2>': ['<digit-3><integer-2>', '<digit-4>'],
 '<digit-3>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<digit-4>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']}

```

就像上面一样，使用这样的语法进行覆盖模糊测试现在将覆盖数字在多个上下文中的情况。更准确地说，有五个上下文：常规整数，以及单个和多个数字的浮点数的整数部分和小数部分。

```py
f = GrammarCoverageFuzzer(dup_expr_grammar, start_symbol="<factor>")
for i in range(10):
    print(f.fuzz()) 
```

```py
(57.5)
2
+-(1 / 3 + 6 / 0 - 7 * 59 * 3 + 8 * 4)
374.88
5.709
0.93
01.1
892.27
219.50
6.636

```

`depth` 参数控制重复应该深入到什么程度。将 `depth` 设置为 1 将只会重复下一个规则：

```py
dup_expr_grammar = extend_grammar(EXPR_GRAMMAR)
duplicate_context(dup_expr_grammar, "<factor>", "<integer>.<integer>", depth=1)
dup_expr_grammar 
```

```py
{'<start>': ['<expr>'],
 '<expr>': ['<term> + <expr>', '<term> - <expr>', '<term>'],
 '<term>': ['<factor> * <term>', '<factor> / <term>', '<factor>'],
 '<factor>': ['+<factor>',
  '-<factor>',
  '(<expr>)',
  '<integer-1>.<integer-2>',
  '<integer>'],
 '<integer>': ['<digit><integer>', '<digit>'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<integer-1>': ['<digit><integer-1>', '<digit>'],
 '<integer-2>': ['<digit><integer-2>', '<digit>']}

```

```py
assert is_valid_grammar(dup_expr_grammar) 
```

默认情况下，`depth` 被设置为 $\infty$，表示无限制的重复。真正的无限制重复可能会对像 `EXPR_GRAMMAR` 这样的递归语法造成问题，因此 `duplicate_context()` 被设置为一旦重复就不再重复符号。尽管如此，如果我们将其应用于重复 *所有* `<expr>` 展开式，我们将得到一个包含至少 292 条规则的语法：

```py
dup_expr_grammar = extend_grammar(EXPR_GRAMMAR)
duplicate_context(dup_expr_grammar, "<expr>") 
```

```py
assert is_valid_grammar(dup_expr_grammar)
len(dup_expr_grammar) 
```

```py
292

```

这为我们提供了近 2000 个展开式来覆盖：

```py
f = GrammarCoverageFuzzer(dup_expr_grammar)
len(f.max_expansion_coverage()) 
```

```py
1981

```

再次重复一次会继续增加语法和覆盖要求：

```py
dup_expr_grammar = extend_grammar(EXPR_GRAMMAR)
duplicate_context(dup_expr_grammar, "<expr>")
duplicate_context(dup_expr_grammar, "<expr-1>")
len(dup_expr_grammar) 
```

```py
594

```

```py
f = GrammarCoverageFuzzer(dup_expr_grammar)
len(f.max_expansion_coverage()) 
```

```py
3994

```

在这一点上，可以单独覆盖许多上下文——例如，加法中元素的乘法：

```py
dup_expr_grammar["<expr>"] 
```

```py
['<term-1> + <expr-4>', '<term-5> - <expr-8>', '<term-9>']

```

```py
dup_expr_grammar["<term-1-1>"] 
```

```py
['<factor-1-1> * <term-1-1>', '<factor-2-1> / <term-1-1>', '<factor-3-1>']

```

```py
dup_expr_grammar["<factor-1-1>"] 
```

```py
['+<factor-1-1>',
 '-<factor-1-1>',
 '(<expr-1-1>)',
 '<integer-1-1>.<integer-2-1>',
 '<integer-3-1>']

```

结果语法可能不再适用于人类维护；但运行覆盖驱动的模糊器，如 `GrammarCoverageFuzzer()`，将覆盖所有这些展开式在所有上下文中的情况。如果您想覆盖许多上下文中的元素，那么 `duplicate_context()` 后跟覆盖驱动的模糊器就是您的朋友。

## 通过覆盖语法来覆盖代码

有无上下文：通过系统地覆盖所有输入元素，我们在输入中得到了更大的多样性——但这是否意味着程序行为的多样性也更大？毕竟，我们想要覆盖的行为，包括意外行为。

在语法中，有一些元素直接对应于程序功能。处理算术表达式的程序将具有由单个元素直接触发的功能——比如说，由 `+` 触发的加法功能，由 `-` 触发的减法，以及由输入中浮点数触发的浮点运算。

这种输入结构与功能之间的联系导致了语法覆盖与代码覆盖之间的强 *相关性*。换句话说：如果我们能够实现高语法覆盖，这也将导致高代码覆盖。

### CGI 语法

让我们在我们的语法之一上探索这种关系——比如说，从 覆盖率章节 中的 CGI 解码器。

<details id="Excursion:-Creating-the-Plot"><summary>创建图表</summary>

我们计算一个映射 `coverages`，其中在 `coverages[x]` = `{y_1, y_2, ...}`，`x` 是获得的语法覆盖率，而 `y_n` 是第 `n` 次运行获得的代码覆盖率。

我们首先计算最大覆盖率，就像在 覆盖率章节 中所做的那样：

```py
from Coverage import Coverage, cgi_decode 
```

```py
with Coverage() as cov_max:
    cgi_decode('+')
    cgi_decode('%20')
    cgi_decode('abc')
    try:
        cgi_decode('%?a')
    except:
        pass 
```

现在，我们运行我们的实验：

```py
f = GrammarCoverageFuzzer(CGI_GRAMMAR, max_nonterminals=2)
coverages: Dict[float, List[float]] = {}

trials = 100
for trial in range(trials):
    f.reset_coverage()
    overall_cov = set()
    max_cov = 30

    for i in range(10):
        s = f.fuzz()
        with Coverage() as cov:
            cgi_decode(s)
        overall_cov |= cov.coverage()

        x = len(f.expansion_coverage()) * 100 / len(f.max_expansion_coverage())
        y = len(overall_cov) * 100 / len(cov_max.coverage())
        if x not in coverages:
            coverages[x] = []
        coverages[x].append(y) 
```

我们计算 `y` 值的平均值：

```py
xs = list(coverages.keys())
ys = [sum(coverages[x]) / len(coverages[x]) for x in coverages] 
```

并创建一个散点图：

```py
%matplotlib inline 
```

```py
import [matplotlib.pyplot](https://matplotlib.org/) as plt 
```

```py
import [matplotlib.ticker](https://matplotlib.org/) as mtick 
```

```py
ax = plt.axes(label="CGI coverage")
ax.yaxis.set_major_formatter(mtick.PercentFormatter())
ax.xaxis.set_major_formatter(mtick.PercentFormatter())

plt.xlim(0, max(xs))
plt.ylim(0, max(ys))

plt.title('Coverage of cgi_decode() vs. grammar coverage')
plt.xlabel('grammar coverage (expansions)')
plt.ylabel('code coverage (lines)') 
```

```py
Text(0, 0.5, 'code coverage (lines)')

```

![](img/27485e5e210ed4f8e8261a349b297050.png)</details>

这个散点图显示了语法覆盖率（X 轴）和代码覆盖率（Y 轴）之间的关系。

![](img/ff88d70b414fa2198050ee38b503cbff.png)

我们可以看到，语法覆盖率越高，代码覆盖率也越高。

这也转化为约 0.9 的相关系数，表明有很强的相关性：

```py
import [numpy](https://numpy.org/) as np 
```

```py
np.corrcoef(xs, ys) 
```

```py
array([[1\.        , 0.81663071],
       [0.81663071, 1\.        ]])

```

这也由 Spearman 排名相关系数所证实：

```py
from [scipy.stats](https://docs.scipy.org/doc/scipy/reference/) import spearmanr 
```

```py
spearmanr(xs, ys) 
```

```py
SignificanceResult(statistic=np.float64(0.9477544699285101), pvalue=np.float64(2.2771918715723359e-10))

```

### URL 语法

让我们在 URL 语法上重复这个实验。我们使用上面的相同代码，除了交换语法和函数：

```py
from [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) import urlparse 
```

<details id="Excursion:-Creating-the-Plot"><summary>创建图表</summary>

再次，我们首先计算最大覆盖率，就像在 覆盖率章节 中所做的那样，进行有根据的猜测：

```py
with Coverage() as cov_max:
    urlparse("http://foo.bar/path")
    urlparse("https://foo.bar#fragment")
    urlparse("ftp://user:password@foo.bar?query=value")
    urlparse("ftps://127.0.0.1/?x=1&y=2") 
```

接下来是实际的实验：

```py
f = GrammarCoverageFuzzer(URL_GRAMMAR, max_nonterminals=2)
coverages: Dict[float, List[float]] = {}

trials = 100
for trial in range(trials):
    f.reset_coverage()
    overall_cov = set()

    for i in range(20):
        s = f.fuzz()
        with Coverage() as cov:
            urlparse(s)
        overall_cov |= cov.coverage()

        x = len(f.expansion_coverage()) * 100 / len(f.max_expansion_coverage())
        y = len(overall_cov) * 100 / len(cov_max.coverage())
        if x not in coverages:
            coverages[x] = []
        coverages[x].append(y) 
```

```py
xs = list(coverages.keys())
ys = [sum(coverages[x]) / len(coverages[x]) for x in coverages] 
```

```py
ax = plt.axes(label="URL coverage")
ax.yaxis.set_major_formatter(mtick.PercentFormatter())
ax.xaxis.set_major_formatter(mtick.PercentFormatter())

plt.xlim(0, max(xs))
plt.ylim(0, max(ys))

plt.title('Coverage of urlparse() vs. grammar coverage')
plt.xlabel('grammar coverage (expansions)')
plt.ylabel('code coverage (lines)') 
```

```py
Text(0, 0.5, 'code coverage (lines)')

```

![](img/06a0d73f00bef31eac2cfa3d2996f9fa.png)</details>

这个散点图显示了语法覆盖率（X 轴）和代码覆盖率（Y 轴）之间的关系。

```py
plt.scatter(xs, ys); 
```

![](img/e76fe0d9672244c397b5b62d43de9b28.png)

在这里，我们有一个更强的相关性，超过 .95：

```py
np.corrcoef(xs, ys) 
```

```py
array([[1\.       , 0.8819171],
       [0.8819171, 1\.       ]])

```

这也由 Spearman 排名相关系数所证实：

```py
spearmanr(xs, ys) 
```

```py
SignificanceResult(statistic=np.float64(0.9486832980505139), pvalue=np.float64(0.05131670194948612))

```

我们得出结论：如果想要获得高代码覆盖率，首先努力实现高语法覆盖率是一个好主意。

### 这是否总是有效？

对于 CGI 和 URL 示例观察到的相关性，并不适用于每个程序和每个结构。

#### 等价元素

首先，一些语法元素被程序统一处理，即使语法将它们视为不同的符号。例如，在 URL 的主机名中，我们可以有多个字符，尽管 URL 处理程序将它们都视为相同。同样，一旦组成数字，单个数字与数字本身的价值相比，影响较小。因此，在数字或字符上实现多样性并不一定会导致功能上的大差异。

通过区分依赖于上下文的元素，并为每个上下文覆盖替代方案，可以解决这个问题，正如上面所讨论的。关键是确定需要多样性的上下文，以及不需要多样性的上下文。

#### 深度数据处理

其次，数据处理的方式可以产生很大的差异。考虑一个*媒体播放器*的输入，它由压缩的媒体数据组成。在处理媒体数据时，媒体播放器将表现出行为差异（特别是在其输出中），但这些差异不能通过媒体数据的单个元素直接触发。同样，在一个大型输入集上训练的*机器学习器*通常不会由输入的单一句法元素控制。（好吧，它可以，但那样的话，我们就不需要机器学习器了。）在这些“深度”数据处理的情况下，在语法中实现结构覆盖率并不一定会引起代码覆盖率。

解决这个问题的方法之一是不仅要实现*句法*多样性，实际上还要实现*语义*多样性。在关于有约束的模糊测试的章节中，我们将看到如何具体生成和过滤输入值，特别是数值。这样的生成器也可以在上下文中应用，以便可以单独控制输入的每个方面。此外，在上面的例子中，*一些*输入部分在结构上仍然可以覆盖：*元数据*（例如作者名称或媒体播放器的作曲家）或*配置数据*（例如机器学习器的设置）可以并且应该系统地覆盖；我们将在“配置模糊测试”章节中看到这是如何实现的[ConfigurationFuzzer.html]。

## 经验教训

+   快速实现*语法覆盖率*会导致大量不同输入的产生。

+   复制语法规则允许在特定的*上下文*中覆盖元素。

+   实现语法覆盖率有助于获得*代码覆盖率*。

## 下一步

从这里，你可以学习如何

+   使用语法覆盖率系统地测试配置。

## 背景

确保语法中的每个扩展至少使用一次的想法可以追溯到 Burkhardt [[Burkhardt *et al*, 1967](https://doi.org/10.1007/BF02235512)]，后来由 Paul Purdom [[Purdom *et al*, 1972](https://doi.org/10.1007/BF01932308)]重新发现。语法覆盖率和代码覆盖率之间的关系是由 Nikolas Havrikov 发现的，他在他的博士论文中探讨了这一点。

## 练习

### 练习 1：测试 ls

考虑 Unix 的`ls`程序，它用于列出目录的内容。为调用`ls`创建一个语法：

```py
LS_EBNF_GRAMMAR: Grammar = {
    '<start>': ['-<options>'],
    '<options>': ['<option>*'],
    '<option>': ['1', 'A', '@',
                 # many more
                 ]
} 
```

```py
assert is_valid_grammar(LS_EBNF_GRAMMAR) 
```

使用`GrammarCoverageFuzzer`测试所有选项。确保使用每个选项集调用`ls`。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GrammarCoverageFuzzer.ipynb#Exercises) 进行练习并查看解决方案。

### 练习 2：缓存

`max_expansion_coverage()`函数的值仅取决于语法。修改实现方式，使得每个符号和深度的值在初始化（`__init__()`）时预先计算；这样，`max_expansion_coverage()`函数就可以简单地从表中查找值。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GrammarCoverageFuzzer.ipynb#Exercises) 进行练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，均受[MIT License](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。最后修改时间：2023-11-11 18:18:06+01:00。引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[语法覆盖率](https://www.fuzzingbook.org/html/GrammarCoverageFuzzer.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[Fuzzing Book](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/GrammarCoverageFuzzer.html`](https://www.fuzzingbook.org/html/GrammarCoverageFuzzer.html). Retrieved 2023-11-11 18:18:06+01:00.

```py
@incollection{fuzzingbook2023:GrammarCoverageFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Grammar Coverage},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/GrammarCoverageFuzzer.html}},
    note = {Retrieved 2023-11-11 18:18:06+01:00},
    url = {https://www.fuzzingbook.org/html/GrammarCoverageFuzzer.html},
    urldate = {2023-11-11 18:18:06+01:00}
}

```
