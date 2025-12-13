# 高效的语法模糊测试

> 原文：[`www.fuzzingbook.org/html/GrammarFuzzer.html`](http://www.fuzzingbook.org/html/GrammarFuzzer.html)

在关于语法的章节中，我们看到了如何使用*语法*进行非常有效和高效的测试。在本章中，我们将之前的*基于字符串*算法改进为*基于树*的算法，这要快得多，并且可以更精确地控制模糊输入的生成。

本章中介绍的算法是其他几种技术的基石；因此，本章在书中起到了“枢纽”的作用。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('Ohl8TLcLl3A') 
```

**先决条件**

+   您应该了解基于语法的模糊测试是如何工作的，例如，可以参考关于语法的章节。

## 概述

要使用本章提供的代码（导入代码），请编写

```py
>>> from fuzzingbook.GrammarFuzzer import <identifier> 
```

然后利用以下功能。

### 高效的语法模糊测试

本章介绍了`GrammarFuzzer`，这是一个高效的语法模糊测试工具，它接受一个语法来生成符合语法的输入字符串。以下是一个典型的用法示例：

```py
>>> from Grammars import US_PHONE_GRAMMAR
>>> phone_fuzzer = GrammarFuzzer(US_PHONE_GRAMMAR)
>>> phone_fuzzer.fuzz()
'(771)306-0659' 
```

`GrammarFuzzer`构造函数接受多个关键字参数来控制其行为。例如，`start_symbol`允许设置开始扩展的符号（而不是`<start>`）：

```py
>>> area_fuzzer = GrammarFuzzer(US_PHONE_GRAMMAR, start_symbol='<area>')
>>> area_fuzzer.fuzz()
'409' 
```

这是如何参数化`GrammarFuzzer`构造函数的示例：

```py
Produce strings from `grammar`, starting with `start_symbol`.
If `min_nonterminals` or `max_nonterminals` is given, use them as limits 
for the number of nonterminals produced.  
If `disp` is set, display the intermediate derivation trees.
If `log` is set, show intermediate steps as text on standard output. 
```

<svg width="324pt" height="458pt" viewBox="0.00 0.00 323.62 458.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 454.25)"><g id="node1" class="node"><title>GrammarFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class GrammarFuzzer:

使用推导树高效地从语法中生成字符串。"><text text-anchor="start" x="39.12" y="-311.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarFuzzer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="GrammarFuzzer"><g id="a_node1_1"><a xlink:href="#" xlink:title="__init__(self, grammar: Dict[str, List[Expansion]], start_symbol: str = '<start>', min_nonterminals: int = 0, max_nonterminals: int = 10, disp: bool = False, log: Union[bool, int] = False) -> None:

从`grammar`生成字符串，以`start_symbol`开始。

如果提供了`min_nonterminals`或`max_nonterminals`，则使用它们作为限制

对于生成的非终结符数量。

如果设置`disp`，则显示中间推导树。

如果设置`log`，则将中间步骤以文本形式显示在标准输出上。"><text text-anchor="start" x="8" y="-289.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="fuzz(self) -> str:

从语法生成一个字符串。"><text text-anchor="start" x="8" y="-276.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="fuzz_tree(self) -> DerivationTree:

从语法中生成推导树。"><text text-anchor="start" x="8" y="-263.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">fuzz_tree()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="any_possible_expansions(self, node: DerivationTree) -> bool"><text text-anchor="start" x="8" y="-250" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">any_possible_expansions()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="check_grammar(self) -> None:

检查传入的语法。"><text text-anchor="start" x="8" y="-237.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">check_grammar()</text></a></g> <g id="a_node1_6"><a xlink:href="#" xlink:title="choose_node_expansion(self, node: DerivationTree, children_alternatives: List[List[DerivationTree]]) -> int:

返回在`children_alternatives`中要选择的扩展的索引。

`children_alternatives`：`node`的可能子节点列表。

默认为随机。在子类中可被覆盖。"><text text-anchor="start" x="8" y="-225.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">choose_node_expansion()</text></a></g> <g id="a_node1_7"><a xlink:href="#" xlink:title="choose_tree_expansion(self, tree: DerivationTree, children: List[DerivationTree]) -> int:

返回要选择的子树在`children`中的索引。"><text text-anchor="start" x="8" y="-213" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">choose_tree_expansion()</text></a></g>

默认为随机。"><text text-anchor="start" x="8" y="-211.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">choose_tree_expansion()</text></a></g> <g id="a_node1_8"><a xlink:href="#" xlink:title="expand_node(self, node: DerivationTree) -> DerivationTree"><text text-anchor="start" x="8" y="-199" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expand_node()</text></a></g> <g id="a_node1_9"><a xlink:href="#" xlink:title="expand_node_by_cost(self, node: DerivationTree, choose: Callable = <built-in function min>) -> DerivationTree"><text text-anchor="start" x="8" y="-186.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expand_node_by_cost()</text></a></g> <g id="a_node1_10"><a xlink:href="#" xlink:title="expand_node_max_cost(self, node: DerivationTree) -> DerivationTree"><text text-anchor="start" x="8" y="-173.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expand_node_max_cost()</text></a></g> <g id="a_node1_11"><a xlink:href="#" xlink:title="expand_node_min_cost(self, node: DerivationTree) -> DerivationTree"><text text-anchor="start" x="8" y="-160.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expand_node_min_cost()</text></a></g> <g id="a_node1_12"><a xlink:href="#" xlink:title="expand_node_randomly(self, node: DerivationTree) -> DerivationTree:

为`node`选择一个随机扩展并返回它"><text text-anchor="start" x="8" y="-148" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expand_node_randomly()</text></a></g> <g id="a_node1_13"><a xlink:href="#" xlink:title="expand_tree(self, tree: DerivationTree) -> DerivationTree:

使用三阶段策略扩展`tree`，直到所有扩展完成。"><text text-anchor="start" x="8" y="-135.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expand_tree()</text></a></g> <g id="a_node1_14"><a xlink:href="#" xlink:title="expand_tree_once(self, tree: DerivationTree) -> DerivationTree:

在树中选择一个未扩展的符号并扩展它

可在子类中重载。"><text text-anchor="start" x="8" y="-123.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">expand_tree_once()</text></a></g> <g id="a_node1_15"><a xlink:href="#" xlink:title="expand_tree_with_strategy(self, tree: DerivationTree, expand_node_method: Callable, limit: Optional[int] = None):

使用`expand_node_method`作为节点扩展函数来扩展树

直到可能的扩展数量达到`limit`。"><text text-anchor="start" x="8" y="-109.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expand_tree_with_strategy()</text></a></g> <g id="a_node1_16"><a xlink:href="#" xlink:title="expansion_cost(self, expansion: Expansion, seen: Set[str] = set()) -> Union[int, float]"><text text-anchor="start" x="8" y="-97" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expansion_cost()</text></a></g> <g id="a_node1_17"><a xlink:href="#" xlink:title="expansion_to_children(self, expansion: Expansion) -> List[DerivationTree]"><text text-anchor="start" x="8" y="-84.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">expansion_to_children()</text></a></g> <g id="a_node1_18"><a xlink:href="#" xlink:title="init_tree(self) -> DerivationTree"><text text-anchor="start" x="8" y="-71.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">init_tree()</text></a></g> <g id="a_node1_19"><a xlink:href="#" xlink:title="log_tree(self, tree: DerivationTree) -> None:

如果 self.log 被设置，则输出一个树形结构；如果 self.display 也被设置，则显示树结构"><text text-anchor="start" x="8" y="-58.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">log_tree()</text></a></g> <g id="a_node1_20"><a xlink:href="#" xlink:title="possible_expansions(self, node: DerivationTree) -> int"><text text-anchor="start" x="8" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">possible_expansions()</text></a></g> <g id="a_node1_21"><a xlink:href="#" xlink:title="process_chosen_children(self, chosen_children: List[DerivationTree], expansion: Expansion) -> List[DerivationTree]:

在选择后处理子节点。默认情况下，不执行任何操作。《process_chosen_children()`</a></g> <g id="a_node1_22"><a xlink:href="#" xlink:title="supported_opts(self) -> Set[str]:

支持的选项集。应在子类中重载。《supported_opts()`</a></g> <g id="a_node1_23"><a xlink:href="#" xlink:title="symbol_cost(self, symbol: str, seen: Set[str] = set()) -> Union[int, float]"><text text-anchor="start" x="8" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">symbol_cost()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>Fuzzer</title> <g id="a_node2"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊测试器的基类。《Fuzzer`</text> <g id="a_node2_24"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node2_25"><a xlink:href="Fuzzer.html" xlink:title="__init__(self) -> None:

构造函数</a></g> <g id="a_node2_26"><a xlink:href="Fuzzer.html" xlink:title="fuzz(self) -> str:

返回模糊输入</a></g> <g id="a_node2_27"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

使用模糊输入运行 `runner` 的 `run()` 方法</a></g> <g id="a_node2_28"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

使用模糊输入运行`runner`，`trials`次"><text text-anchor="start" x="59" y="-373" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">runs()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>GrammarFuzzer->Fuzzer</title></g> <g id="node3" class="node"><title>图例</title> <text text-anchor="start" x="196.38" y="-180.12" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="196.38" y="-170.12" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="202.38" y="-170.12" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="196.38" y="-160.12" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="202.38" y="-160.12" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="196.38" y="-150.12" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="202.38" y="-150.12" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="196.38" y="-141.07" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

### 推导树

在内部，`GrammarFuzzer`利用推导树逐步展开。在生成字符串后，生成的树可以通过`derivation_tree`属性访问。

```py
>>> display_tree(phone_fuzzer.derivation_tree) 
```

<svg width="636pt" height="223pt" viewBox="0.00 0.00 635.75 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="265.12" y="-201.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="265.12" y="-151.7" font-family="Times,serif" font-size="14.00"><phone-number></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="113.12" y="-101.45" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="166.12" y="-101.45" font-family="Times,serif" font-size="14.00"><area></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="232.12" y="-101.45" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge10" class="edge"><title>1->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="299.12" y="-101.45" font-family="Times,serif" font-size="14.00"><exchange></text></g> <g id="edge11" class="edge"><title>1->11</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="366.12" y="-101.45" font-family="Times,serif" font-size="14.00">- (45)</text></g> <g id="edge18" class="edge"><title>1->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="489.12" y="-101.45" font-family="Times,serif" font-size="14.00"><line></text></g> <g id="edge19" class="edge"><title>1->19</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="34.12" y="-51.2" font-family="Times,serif" font-size="14.00"><lead-digit></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="107.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge6" class="edge"><title>3->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="166.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge8" class="edge"><title>3->8</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="34.12" y="-0.95" font-family="Times,serif" font-size="14.00">7 (55)</text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="107.12" y="-0.95" font-family="Times,serif" font-size="14.00">7 (55)</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="166.12" y="-0.95" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="239.12" y="-51.2" font-family="Times,serif" font-size="14.00"><lead-digit></text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="312.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge14" class="edge"><title>11->14</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="371.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge16" class="edge"><title>11->16</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="239.12" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="312.12" y="-0.95" font-family="Times,serif" font-size="14.00">0 (48)</text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="371.12" y="-0.95" font-family="Times,serif" font-size="14.00">6 (54)</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="430.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="489.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge22" class="edge"><title>19->22</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="548.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge24" class="edge"><title>19->24</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="607.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge26" class="edge"><title>19->26</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="430.12" y="-0.95" font-family="Times,serif" font-size="14.00">0 (48)</text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="489.12" y="-0.95" font-family="Times,serif" font-size="14.00">6 (54)</text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="548.12" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge25" class="edge"><title>24->25</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="607.12" y="-0.95" font-family="Times,serif" font-size="14.00">9 (57)</text></g> <g id="edge27" class="edge"><title>26->27</title></g></g></svg>

在推导树的内部表示中，一个*节点*是一个对（`symbol`, `children`）。对于非终结符，`symbol`是被展开的符号，而`children`是进一步节点的列表。对于终结符，`symbol`是终结符字符串，而`children`为空。

```py
>>> phone_fuzzer.derivation_tree
('<start>',
 [('<phone-number>',
   [('(', []),
    ('<area>',
     [('<lead-digit>', [('7', [])]),
      ('<digit>', [('7', [])]),
      ('<digit>', [('1', [])])]),
    (')', []),
    ('<exchange>',
     [('<lead-digit>', [('3', [])]),
      ('<digit>', [('0', [])]),
      ('<digit>', [('6', [])])]),
    ('-', []),
    ('<line>',
     [('<digit>', [('0', [])]),
      ('<digit>', [('6', [])]),
      ('<digit>', [('5', [])]),
      ('<digit>', [('9', [])])])])]) 
```

本章包含各种辅助工具，用于处理推导树，包括可视化工具——特别是上面的`display_tree()`函数。

## 一个不充分的算法

在上一章中，我们介绍了`simple_grammar_fuzzer()`函数，该函数接受一个语法并自动从中生成一个语法上有效的字符串。然而，`simple_grammar_fuzzer()`正如其名所暗示的那样——简单。为了说明问题，让我们回到在语法章节中从`EXPR_GRAMMAR_BNF`创建的`expr_grammar`：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import quiz 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import Tuple, List, Optional, Any, Union, Set, Callable, Dict 
```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import unicode_escape 
```

```py
from Grammars import EXPR_EBNF_GRAMMAR, convert_ebnf_grammar, Grammar, Expansion
from Grammars import simple_grammar_fuzzer, is_valid_grammar, exp_string 
```

```py
expr_grammar = convert_ebnf_grammar(EXPR_EBNF_GRAMMAR)
expr_grammar 
```

```py
{'<start>': ['<expr>'],
 '<expr>': ['<term> + <expr>', '<term> - <expr>', '<term>'],
 '<term>': ['<factor> * <term>', '<factor> / <term>', '<factor>'],
 '<factor>': ['<sign-1><factor>', '(<expr>)', '<integer><symbol-1>'],
 '<sign>': ['+', '-'],
 '<integer>': ['<digit-1>'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<symbol>': ['.<integer>'],
 '<sign-1>': ['', '<sign>'],
 '<symbol-1>': ['', '<symbol>'],
 '<digit-1>': ['<digit>', '<digit><digit-1>']}

```

`expr_grammar`有一个有趣的特性。如果我们将其输入到`simple_grammar_fuzzer()`中，该函数会陷入停滞：

```py
from ExpectError import ExpectTimeout 
```

```py
with ExpectTimeout(1):
    simple_grammar_fuzzer(grammar=expr_grammar, max_nonterminals=3) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_9136/3259437052.py", line 2, in <module>
    simple_grammar_fuzzer(grammar=expr_grammar, max_nonterminals=3)
  File "Grammars.ipynb", line 87, in simple_grammar_fuzzer
    symbol_to_expand = random.choice(nonterminals(term))
                                     ^^^^^^^^^^^^^^^^^^
  File "Grammars.ipynb", line 61, in nonterminals
    return RE_NONTERMINAL.findall(expansion)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "Timeout.ipynb", line 43, in timeout_handler
    raise TimeoutError()
TimeoutError (expected)

```

为什么会这样？看看语法；记住你关于`simple_grammar_fuzzer()`的知识；并以`log=true`参数运行`simple_grammar_fuzzer()`以查看展开情况。

### 问答

为什么`simple_grammar_fuzzer()`会挂起？

确实！问题就在这个规则中：

```py
expr_grammar['<factor>'] 
```

```py
['<sign-1><factor>', '(<expr>)', '<integer><symbol-1>']

```

在这里，除了 `(expr)` 以外的任何选择都会增加符号的数量，即使只是临时的。由于我们对要扩展的符号数量设置了硬限制，因此扩展 `<factor>` 的唯一选择是 `(<expr>)`，这会导致 *无限添加括号*。

无限扩展的问题只是 `simple_grammar_fuzzer()` 存在的问题之一。更多问题包括：

1.  *这是低效的*。随着每次迭代，这个模糊器会搜索到目前为止产生的字符串以寻找要扩展的符号。随着生成字符串的增长，这变得低效。

1.  *这是难以控制的*。即使在限制符号数量的同时，仍然可能获得非常长的字符串——甚至无限长的字符串，如上所述。

让我们通过绘制不同长度字符串所需的时间来阐述这两个问题。

```py
from Grammars import simple_grammar_fuzzer 
```

```py
from Grammars import START_SYMBOL, EXPR_GRAMMAR, URL_GRAMMAR, CGI_GRAMMAR 
```

```py
from Grammars import RE_NONTERMINAL, nonterminals, is_nonterminal 
```

```py
from Timer import Timer 
```

```py
trials = 50
xs = []
ys = []
for i in range(trials):
    with Timer() as t:
        s = simple_grammar_fuzzer(EXPR_GRAMMAR, max_nonterminals=15)
    xs.append(len(s))
    ys.append(t.elapsed_time())
    print(i, end=" ")
print() 
```

```py
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 

```

```py
average_time = sum(ys) / trials
print("Average time:", average_time) 
```

```py
Average time: 0.10001757324207575

```

```py
%matplotlib inline

import [matplotlib.pyplot](https://matplotlib.org/) as plt
plt.scatter(xs, ys)
plt.title('Time required for generating an output'); 
```

![](img/2584c049202651a68ebeb102d6b4f1c4.png)

我们看到，(1) 生成输出所需的时间与输出长度成二次方增长，以及 (2) 产生的输出中有很大一部分长度达到数万个字符。

为了解决这些问题，我们需要一个 *更智能的算法*——一个更高效、更好地控制扩展并能够在 `expr_grammar` 中预见 `(expr)` 选项会导致潜在无限扩展，而与其他两个选项不同的算法。

## 推导树

为了获得更高效的算法并更好地控制扩展，我们将使用语法产生的字符串的特殊表示。一般想法是使用一个将随后扩展的 *树* 结构——所谓的 *推导树*。这种表示允许我们始终跟踪我们的扩展状态——回答诸如哪些元素被扩展成了哪些其他元素，以及哪些符号还需要扩展等问题。此外，向树中添加新元素比一次又一次替换字符串要高效得多。

与编程中使用的其他树一样，推导树（也称为 *解析树* 或 *具体语法树*）由 *节点* 组成，这些节点有其他节点（称为 *子节点*）作为它们的 *子节点*。树从没有父节点的单个节点开始；这被称为 *根节点*；没有子节点的节点称为 *叶节点*。

推导树语法扩展过程如下所示，使用的是来自语法章节的算术语法 [Grammars.html]。我们从一个节点作为树的根节点开始，代表 *起始符号*——在我们的情况下是 `<start>`。

<svg width="48pt" height="22pt" viewBox="0.00 0.00 47.75 22.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 18.25)"><title>root</title> <g id="node1" class="node"><title>\<start\></title> <text text-anchor="middle" x="19.88" y="-0.95" font-family="Times,serif" font-size="14.00"><start></text></g></g></svg>

为了扩展树，我们需要遍历它，寻找没有子节点的非终结符号 $S$。因此，$S$ 是一个仍然需要扩展的符号。然后，我们从语法中选择一个扩展来应用于 $S$。接着，我们将扩展作为 $S$ 的新子节点添加。对于我们的起始符号 `<start>`，唯一的扩展是 `<expr>`，所以我们将其添加为子节点。

<svg width="49pt" height="73pt" viewBox="0.00 0.00 48.50 72.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 68.5)"><title>root</title> <g id="node1" class="node"><title>\<start\></title> <text text-anchor="middle" x="20.25" y="-51.2" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>\<expr\></title> <text text-anchor="middle" x="20.25" y="-0.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>\<start\>->\<expr\></title></g></g></svg>

从推导树构建生成的字符串，我们按顺序遍历树并收集树叶处的符号。在上面的例子中，我们得到字符串 `"<expr>"`。

为了进一步扩展树，我们选择另一个符号进行扩展，并将其扩展作为新的子节点添加。这将得到 `<expr>` 符号，它被扩展为 `<expr> + <term>`，添加了三个子节点。

<svg width="138pt" height="123pt" viewBox="0.00 0.00 138.12 122.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 118.75)"><title>root</title> <g id="node1" class="node"><title>\<start\></title> <text text-anchor="middle" x="66.12" y="-101.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>\<expr\></title> <text text-anchor="middle" x="66.12" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>\<start\>->\<expr\></title></g> <g id="node3" class="node"><title>\<expr\></title> <text text-anchor="middle" x="22.12" y="-0.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>\<expr\>->\<expr\></title></g> <g id="node4" class="node"><title>+</title> <text text-anchor="middle" x="66.12" y="-0.95" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge3" class="edge"><title>\<expr\>->+</title></g> <g id="node5" class="node"><title>\<term\></title> <text text-anchor="middle" x="109.12" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>\<expr\>->\<term\></title></g></g></svg>

我们重复扩展，直到没有符号可以扩展：

<svg width="151pt" height="374pt" viewBox="0.00 0.00 150.88 374.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 370)"><title>root</title> <g id="node1" class="node"><title>\<start\></title> <text text-anchor="middle" x="72.88" y="-352.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>\<expr\></title> <text text-anchor="middle" x="72.88" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>\<start\>->\<expr\></title></g> <g id="node3" class="node"><title>\<expr\></title> <text text-anchor="middle" x="28.88" y="-252.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>\<expr\>->\<expr\></title></g> <g id="node4" class="node"><title>+</title> <text text-anchor="middle" x="72.88" y="-252.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge3" class="edge"><title>\<expr\>->+</title></g> <g id="node5" class="node"><title>\<term\></title> <text text-anchor="middle" x="115.88" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>\<expr\>->\<term\></title></g> <g id="node6" class="node"><title>\<term\></title> <text text-anchor="middle" x="28.88" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge5" class="edge"><title>\<expr\> ->\<term\></title></g> <g id="node11" class="node"><title>\<factor\></title> <text text-anchor="middle" x="115.88" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge10" class="edge"><title>\<term\>->\<factor\></title></g> <g id="node7" class="node"><title>\<factor\></title> <text text-anchor="middle" x="28.88" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge6" class="edge"><title>\<term\> ->\<factor\></title></g> <g id="node8" class="node"><title>\<integer\></title> <text text-anchor="middle" x="28.88" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge7" class="edge"><title>\<factor\> ->\<integer\></title></g> <g id="node9" class="node"><title>\<digit\></title> <text text-anchor="middle" x="28.88" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge8" class="edge"><title>\<integer\> ->\<digit\></title></g> <g id="node10" class="node"><title>2</title> <text text-anchor="middle" x="28.88" y="-0.95" font-family="Times,serif" font-size="14.00">2</text></g> <g id="edge9" class="edge"><title>\<digit\> ->2</title></g> <g id="node12" class="node"><title>\<integer\></title> <text text-anchor="middle" x="115.88" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge11" class="edge"><title>\<factor\>->\<integer\></title></g> <g id="node13" class="node"><title>\<digit\></title> <text text-anchor="middle" x="115.88" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge12" class="edge"><title>\<integer\>->\<digit\></title></g> <g id="node14" class="node"><title>2</title> <text text-anchor="middle" x="115.88" y="-51.2" font-family="Times,serif" font-size="14.00">2</text></g> <g id="edge13" class="edge"><title>\<digit\>->2</title></g></g></svg>

现在我们有了字符串 `2 + 2` 的表示。然而，与字符串本身相比，推导树记录了生成的字符串的 *整个结构*（以及生产历史，或 *推导* 历史）。它还允许进行简单的比较和操作——比如，用一个子树（子结构）替换另一个子树。

## 表示推导树

在 Python 中表示推导树时，我们使用以下格式。一个节点是一个对

```py
(SYMBOL_NAME, CHILDREN) 
```

其中 `SYMBOL_NAME` 是表示节点的字符串（即 `"<start>"` 或 `"+"`），而 `CHILDREN` 是子节点列表。

`CHILDREN` 可以取一些特殊值：

1.  `None` 作为未来扩展的占位符。这意味着该节点是一个应该进一步扩展的 *非终端符号*。

1.  `[]`（即空列表）表示没有子节点。这意味着该节点是一个不能再扩展的 *终端符号*。

类型 `DerivationTree` 捕获了这种结构。(`Any` 实际上应该读作 `DerivationTree`，但 Python 静态类型检查器无法很好地处理递归类型。)

```py
DerivationTree = Tuple[str, Optional[List[Any]]] 
```

让我们以一个非常简单的推导树为例，表示上面提到的 `<expr> + <term>` 的中间步骤。

```py
derivation_tree: DerivationTree = ("<start>",
                   [("<expr>",
                     [("<expr>", None),
                      (" + ", []),
                         ("<term>", None)]
                     )]) 
```

为了更好地理解这个树的结构，让我们引入一个函数 `display_tree()`，它可视化这个树。

<details id="Excursion:-Implementing-display_tree()"><summary>实现 `display_tree()`</summary>

我们使用 `graphviz` 包中的 `dot` 绘图程序算法性地遍历上述结构。（除非你对树可视化有浓厚的兴趣，否则你可以直接跳到下面的示例。）

```py
from [graphviz](https://graphviz.readthedocs.io/) import Digraph 
```

```py
from [IPython.display](https://ipython.readthedocs.io/en/stable/api/generated/IPython.display.html) import display 
```

```py
import [re](https://docs.python.org/3/library/re.html)
import [string](https://docs.python.org/3/library/string.html) 
```

```py
def dot_escape(s: str, show_ascii=None) -> str:
  """Return s in a form suitable for dot.
 If `show_ascii` is True or length of `s` is 1, also append ascii value."""
    escaped_s = ''
    if show_ascii is None:
        show_ascii = (len(s) == 1)  # Default: Single chars only

    if show_ascii and s == '\n':
        return '\\\\n (10)'

    s = s.replace('\n', '\\n')
    for c in s:
        if re.match('[,<>\\\\"]', c):
            escaped_s += '\\' + c
        elif c in string.printable and 31 < ord(c) < 127:
            escaped_s += c
        else:
            escaped_s += '\\\\x' + format(ord(c), '02x')

        if show_ascii:
            escaped_s += f' ({ord(c)})'

    return escaped_s 
```

```py
assert dot_escape("hello") == "hello" 
```

```py
assert dot_escape("<hello>, world") == "\\<hello\\>\\, world" 
```

```py
assert dot_escape("\\n") == "\\\\n" 
```

```py
assert dot_escape("\n", show_ascii=False) == "\\\\n" 
```

```py
assert dot_escape("\n", show_ascii=True) == "\\\\n (10)" 
```

```py
assert dot_escape("\n", show_ascii=True) == "\\\\n (10)" 
```

```py
assert dot_escape('\x01', show_ascii=False) == "\\\\x01" 
```

```py
assert dot_escape('\x01') == "\\\\x01 (1)" 
```

虽然我们现在对可视化 `derivation_tree` 感兴趣，但我们希望泛化可视化过程。特别是，如果我们的方法 `display_tree()` 可以显示任何类似的数据结构，那将是有帮助的。为此，我们定义了一个辅助方法 `extract_node()`，它从给定的数据结构中提取当前符号和子节点。默认实现简单地从任何 `derivation_tree` 节点中提取符号、子节点和注释。

```py
def extract_node(node, id):
    symbol, children, *annotation = node
    return symbol, children, ''.join(str(a) for a in annotation) 
```

在可视化树时，有时显示某些节点的方式不同是有用的。例如，有时区分未处理的节点和处理过的节点是有用的。我们定义了一个辅助过程 `default_node_attr()`，它提供基本的显示，可以被用户自定义。

```py
def default_node_attr(dot, nid, symbol, ann):
    dot.node(repr(nid), dot_escape(symbol)) 
```

与节点类似，边也可能需要修改。我们定义 `default_edge_attr()` 作为用户可以定制的辅助过程。

```py
def default_edge_attr(dot, start_node, stop_node):
    dot.edge(repr(start_node), repr(stop_node)) 
```

在可视化树时，有时可能希望改变树的外观。例如，如果树是从左到右而不是从上到下排列的，有时更容易查看。为此，我们定义了另一个辅助过程 `default_graph_attr()`。

```py
def default_graph_attr(dot):
    dot.attr('node', shape='plain') 
```

最后，我们定义了一个方法 `display_tree()`，它接受这四个函数 `extract_node()`、`default_edge_attr()`、`default_node_attr()` 和 `default_graph_attr()`，并使用它们来显示树。

```py
def display_tree(derivation_tree: DerivationTree,
                 log: bool = False,
                 extract_node: Callable = extract_node,
                 node_attr: Callable = default_node_attr,
                 edge_attr: Callable = default_edge_attr,
                 graph_attr: Callable = default_graph_attr) -> Any:

    # If we import display_tree, we also have to import its functions
    from [graphviz](https://graphviz.readthedocs.io/) import Digraph

    counter = 0

    def traverse_tree(dot, tree, id=0):
        (symbol, children, annotation) = extract_node(tree, id)
        node_attr(dot, id, symbol, annotation)

        if children:
            for child in children:
                nonlocal counter
                counter += 1
                child_id = counter
                edge_attr(dot, id, child_id)
                traverse_tree(dot, child, child_id)

    dot = Digraph(comment="Derivation Tree")
    graph_attr(dot)
    traverse_tree(dot, derivation_tree)
    if log:
        print(dot)
    return dot 
```</details>

这就是我们的树所可视化的内容：

```py
display_tree(derivation_tree) 
```

<svg width="142pt" height="123pt" viewBox="0.00 0.00 142.25 122.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 118.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="66.25" y="-101.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="66.25" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="20.25" y="-0.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="66.25" y="-0.95" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="113.25" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>1->4</title></g></g></svg>

### 习题

那么这些中的哪一个是 `derivation_tree` 的内部表示？

您可以亲自检查：

```py
derivation_tree 
```

```py
('<start>', [('<expr>', [('<expr>', None), (' + ', []), ('<term>', None)])])

```

在本书中，我们偶尔也会使用一个函数 `display_annotated_tree()`，它允许向单个节点添加注释。

<details id="Excursion:-Source-code-and-example-for-display_annotated_tree()"><summary>为 `display_annotated_tree()` 的源代码和示例</summary>

`display_annotated_tree()` 显示带注释的树结构，并按从左到右的顺序排列图。

```py
def display_annotated_tree(tree: DerivationTree,
                           a_nodes: Dict[int, str],
                           a_edges: Dict[Tuple[int, int], str],
                           log: bool = False):
    def graph_attr(dot):
        dot.attr('node', shape='plain')
        dot.graph_attr['rankdir'] = 'LR'

    def annotate_node(dot, nid, symbol, ann):
        if nid in a_nodes:
            dot.node(repr(nid), 
                     "%s (%s)" % (dot_escape(unicode_escape(symbol)),
                                  a_nodes[nid]))
        else:
            dot.node(repr(nid), dot_escape(unicode_escape(symbol)))

    def annotate_edge(dot, start_node, stop_node):
        if (start_node, stop_node) in a_edges:
            dot.edge(repr(start_node), repr(stop_node),
                     a_edges[(start_node, stop_node)])
        else:
            dot.edge(repr(start_node), repr(stop_node))

    return display_tree(tree, log=log,
                        node_attr=annotate_node,
                        edge_attr=annotate_edge,
                        graph_attr=graph_attr) 
```

```py
display_annotated_tree(derivation_tree, {3: 'plus'}, {(1, 3): 'op'}, log=False) 
```

<svg width="226pt" height="87pt" viewBox="0.00 0.00 225.75 87.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 83.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-33.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="97" y="-33.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="192.25" y="-65.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="192.25" y="-33.95" font-family="Times,serif" font-size="14.00">+  (plus)</text></g> <g id="edge3" class="edge"><title>1->3</title> <text text-anchor="middle" x="142" y="-41.08" font-family="Times,serif" font-size="14.00">op</text></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="192.25" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>1->4</title></g></g></svg></details>

如果我们想将树中的所有叶节点作为字符串查看，下面的 `all_terminals()` 函数就派上用场了：

```py
def all_terminals(tree: DerivationTree) -> str:
    (symbol, children) = tree
    if children is None:
        # This is a nonterminal symbol not expanded yet
        return symbol

    if len(children) == 0:
        # This is a terminal symbol
        return symbol

    # This is an expanded symbol:
    # Concatenate all terminal symbols from all children
    return ''.join([all_terminals(c) for c in children]) 
```

```py
all_terminals(derivation_tree) 
```

```py
'<expr> + <term>'

```

另一个替代的 `tree_to_string()` 函数也将树转换为字符串；然而，它将非终结符号替换为空字符串。

```py
def tree_to_string(tree: DerivationTree) -> str:
    symbol, children, *_ = tree
    if children:
        return ''.join(tree_to_string(c) for c in children)
    else:
        return '' if is_nonterminal(symbol) else symbol 
```

```py
tree_to_string(derivation_tree) 
```

```py
' + '

```

## 展开节点

现在我们开发一个算法，该算法接受一个包含未扩展符号的树（例如，上面的 `derivation_tree`），并依次扩展这些符号。与早期的模糊器一样，我们创建一个特殊的 `Fuzzer` 子类——在这种情况下，`GrammarFuzzer`。`GrammarFuzzer` 获取一个语法和一个起始符号；其他参数将稍后用于进一步控制创建并支持调试。

```py
from Fuzzer import Fuzzer 
```

```py
class GrammarFuzzer(Fuzzer):
  """Produce strings from grammars efficiently, using derivation trees."""

    def __init__(self,
                 grammar: Grammar,
                 start_symbol: str = START_SYMBOL,
                 min_nonterminals: int = 0,
                 max_nonterminals: int = 10,
                 disp: bool = False,
                 log: Union[bool, int] = False) -> None:
  """Produce strings from `grammar`, starting with `start_symbol`.
 If `min_nonterminals` or `max_nonterminals` is given, use them as limits 
 for the number of nonterminals produced. 
 If `disp` is set, display the intermediate derivation trees.
 If `log` is set, show intermediate steps as text on standard output."""

        self.grammar = grammar
        self.start_symbol = start_symbol
        self.min_nonterminals = min_nonterminals
        self.max_nonterminals = max_nonterminals
        self.disp = disp
        self.log = log
        self.check_grammar()  # Invokes is_valid_grammar() 
```

要向 `GrammarFuzzer` 添加更多方法，我们使用之前已经介绍过的针对 `MutationFuzzer` 类 的黑客技巧。该结构

```py
class GrammarFuzzer(GrammarFuzzer):
    def new_method(self, args):
        pass 
```

允许我们向 `GrammarFuzzer` 类添加一个新方法 `new_method()`。（实际上，我们得到一个新的 `GrammarFuzzer` 类，它扩展了旧的一个，但对我们所有的目的来说，这并不重要。）

<details id="Excursion:-check_grammar()-implementation"><summary>`check_grammar()` 实现</summary>

我们可以使用上面的技巧来定义辅助方法 `check_grammar()`，该方法检查给定的语法是否一致：

```py
class GrammarFuzzer(GrammarFuzzer):
    def check_grammar(self) -> None:
  """Check the grammar passed"""
        assert self.start_symbol in self.grammar
        assert is_valid_grammar(
            self.grammar,
            start_symbol=self.start_symbol,
            supported_opts=self.supported_opts())

    def supported_opts(self) -> Set[str]:
  """Set of supported options. To be overloaded in subclasses."""
        return set()  # We don't support specific options 
```</details>

现在我们定义一个辅助方法 `init_tree()`，它构建一个只包含起始符号的树：

```py
class GrammarFuzzer(GrammarFuzzer):
    def init_tree(self) -> DerivationTree:
        return (self.start_symbol, None) 
```

```py
f = GrammarFuzzer(EXPR_GRAMMAR)
display_tree(f.init_tree()) 
```

<svg width="48pt" height="22pt" viewBox="0.00 0.00 47.75 22.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 18.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-0.95" font-family="Times,serif" font-size="14.00"><start></text></g></g></svg>

这是我们想要扩展的树。

### 选择一个要扩展的子节点

其中一个中心方法在 `GrammarFuzzer` 中是 `choose_node_expansion()`。此方法获取一个节点（例如，`<start>` 节点）和要扩展的可能子节点列表（每个可能的语法扩展对应一个），从中选择一个，并返回其在可能子节点列表中的索引。

通过重载此方法（特别是在后面的章节中），我们可以实现不同的策略——目前，它只是随机选择给定列表中的一个子节点（这些子节点本身也是推导树的列表）。

```py
class GrammarFuzzer(GrammarFuzzer):
    def choose_node_expansion(self, node: DerivationTree,
                              children_alternatives: List[List[DerivationTree]]) -> int:
  """Return index of expansion in `children_alternatives` to be selected.
 'children_alternatives`: a list of possible children for `node`.
 Defaults to random. To be overloaded in subclasses."""
        return random.randrange(0, len(children_alternatives)) 
```

### 获取可能的扩展列表

要实际获取可能的子节点列表，我们需要一个辅助函数 `expansion_to_children()`，它接受一个扩展字符串并将其分解成一个推导树列表——每个符号（终结符或非终结符）对应一个字符串。

<details id="Excursion:-Implementing-expansion_to_children()"><summary>实现 `expansion_to_children()`</summary>

函数 `expansion_to_children()` 使用 `re.split()` 方法将扩展字符串分割成一个子节点列表：

```py
def expansion_to_children(expansion: Expansion) -> List[DerivationTree]:
    # print("Converting " + repr(expansion))
    # strings contains all substrings -- both terminals and nonterminals such
    # that ''.join(strings) == expansion

    expansion = exp_string(expansion)
    assert isinstance(expansion, str)

    if expansion == "":  # Special case: epsilon expansion
        return [("", [])]

    strings = re.split(RE_NONTERMINAL, expansion)
    return [(s, None) if is_nonterminal(s) else (s, [])
            for s in strings if len(s) > 0] 
```</details>

```py
expansion_to_children("<term> + <expr>") 
```

```py
[('<term>', None), (' + ', []), ('<expr>', None)]

```

对于 *epsilon 扩展* 的情况，即扩展到空字符串，如 `<symbol> ::=`，需要特殊处理：

```py
expansion_to_children("") 
```

```py
[('', [])]

```

就像 语法章节 中的 `nonterminals()` 一样，我们为未来的扩展提供了支持，允许扩展是一个包含额外数据的元组（这些数据将被忽略）。

```py
expansion_to_children(("+<term>", {"extra_data": 1234})) 
```

```py
[('+', []), ('<term>', None)]

```

我们将这个辅助函数实现为 `GrammarFuzzer` 中的一个方法，以便子类可以重载它：

```py
class GrammarFuzzer(GrammarFuzzer):
    def expansion_to_children(self, expansion: Expansion) -> List[DerivationTree]:
        return expansion_to_children(expansion) 
```

### 将事物组合起来

使用这个，我们现在可以

1.  树中的某个未扩展的节点，

1.  选择一个随机的扩展，并且

1.  返回新的树。

这就是方法 `expand_node_randomly()` 所做的。

<details id="Excursion:-expand_node_randomly()-implementation"><summary>`expand_node_randomly()` 实现</summary>

`expand_node_randomly()` 函数使用辅助函数 `choose_node_expansion()` 从可能的子节点数组中随机选择一个索引。（`choose_node_expansion()` 可以在子类中重载。）

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
class GrammarFuzzer(GrammarFuzzer):
    def expand_node_randomly(self, node: DerivationTree) -> DerivationTree:
  """Choose a random expansion for `node` and return it"""
        (symbol, children) = node
        assert children is None

        if self.log:
            print("Expanding", all_terminals(node), "randomly")

        # Fetch the possible expansions from grammar...
        expansions = self.grammar[symbol]
        children_alternatives: List[List[DerivationTree]] = [
            self.expansion_to_children(expansion) for expansion in expansions
        ]

        # ... and select a random expansion
        index = self.choose_node_expansion(node, children_alternatives)
        chosen_children = children_alternatives[index]

        # Process children (for subclasses)
        chosen_children = self.process_chosen_children(chosen_children,
                                                       expansions[index])

        # Return with new children
        return (symbol, chosen_children) 
```

通用 `expand_node()` 方法可以后来用于选择不同的展开策略；到目前为止，它只使用 `expand_node_randomly()`。

```py
class GrammarFuzzer(GrammarFuzzer):
    def expand_node(self, node: DerivationTree) -> DerivationTree:
        return self.expand_node_randomly(node) 
```

辅助函数 `process_chosen_children()` 什么都不做；它可以被子类重载以在一旦选择子节点后处理它们。

```py
class GrammarFuzzer(GrammarFuzzer):
    def process_chosen_children(self,
                                chosen_children: List[DerivationTree],
                                expansion: Expansion) -> List[DerivationTree]:
  """Process children after selection.  By default, does nothing."""
        return chosen_children 
```</details>

这是 `expand_node_randomly()` 的工作方式：

```py
f = GrammarFuzzer(EXPR_GRAMMAR, log=True)

print("Before expand_node_randomly():")
expr_tree = ("<integer>", None)
display_tree(expr_tree) 
```

```py
Before expand_node_randomly():

```

<svg width="62pt" height="22pt" viewBox="0.00 0.00 62.00 22.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 18.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="27" y="-0.95" font-family="Times,serif" font-size="14.00"><integer></text></g></g></svg>

```py
print("After expand_node_randomly():")
expr_tree = f.expand_node_randomly(expr_tree)
display_tree(expr_tree) 
```

```py
After expand_node_randomly():
Expanding <integer> randomly

```

<svg width="62pt" height="73pt" viewBox="0.00 0.00 62.00 72.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 68.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="27" y="-0.95" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge1" class="edge"><title>0->1</title></g></g></svg>

### 习题

如果我们展开 `<digit>` 子树，我们会得到什么树？

我们当然可以对其进行测试，对吧？下面我们开始：

```py
digit_subtree = expr_tree[1][0]
display_tree(digit_subtree) 
```

<svg width="49pt" height="22pt" viewBox="0.00 0.00 49.25 22.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 18.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="20.62" y="-0.95" font-family="Times,serif" font-size="14.00"><digit></text></g></g></svg>

```py
print("After expanding the <digit> subtree:")
digit_subtree = f.expand_node_randomly(digit_subtree)
display_tree(digit_subtree) 
```

```py
After expanding the <digit> subtree:
Expanding <digit> randomly

```

<svg width="49pt" height="73pt" viewBox="0.00 0.00 49.25 72.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 68.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="20.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="20.62" y="-0.95" font-family="Times,serif" font-size="14.00">7 (55)</text></g> <g id="edge1" class="edge"><title>0->1</title></g></g></svg>

我们看到 `<digit>` 根据语法规则再次展开——即展开成一个单独的数字。

### 习题

原始的 `expr_tree` 是否会受到这个更改的影响？

尽管我们已经更改了一个子树，但原始的 `expr_tree` 没有受到影响：

```py
display_tree(expr_tree) 
```

<svg width="62pt" height="73pt" viewBox="0.00 0.00 62.00 72.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 68.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="27" y="-0.95" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge1" class="edge"><title>0->1</title></g></g></svg>

这是因为 `expand_node_randomly()` 返回一个新的（展开的）树，并且不会改变作为参数传递的树。

## 展开树

现在我们将应用我们的单节点展开函数到树中的某个节点。为此，我们首先需要 *搜索树中的未展开节点*。`possible_expansions()` 函数计算树中未展开符号的数量：

```py
class GrammarFuzzer(GrammarFuzzer):
    def possible_expansions(self, node: DerivationTree) -> int:
        (symbol, children) = node
        if children is None:
            return 1

        return sum(self.possible_expansions(c) for c in children) 
```

```py
f = GrammarFuzzer(EXPR_GRAMMAR)
print(f.possible_expansions(derivation_tree)) 
```

```py
2

```

如果树有任何未展开的节点，`any_possible_expansions()` 方法返回 True。

```py
class GrammarFuzzer(GrammarFuzzer):
    def any_possible_expansions(self, node: DerivationTree) -> bool:
        (symbol, children) = node
        if children is None:
            return True

        return any(self.any_possible_expansions(c) for c in children) 
```

```py
f = GrammarFuzzer(EXPR_GRAMMAR)
f.any_possible_expansions(derivation_tree) 
```

```py
True

```

下面是 `expand_tree_once()`，这是我们树展开算法的核心方法。它首先检查当前是否正在一个未展开的非终结符号上应用；如果是，它将调用上面讨论的 `expand_node()`。

如果节点已经展开（即有子节点），它会检查仍有未展开符号的子节点子集，随机选择其中一个，并递归地应用自身于该子节点。

<details id="Excursion:-expand_tree_once()-implementation"><summary>`expand_tree_once()` 实现</summary>

`expand_tree_once()` 方法在原地替换子节点，这意味着它实际上是在修改作为参数传递的树，而不是返回一个新的树。这种原地修改使得这个函数特别高效。再次强调，我们使用辅助方法 (`choose_tree_expansion()`) 从可以展开的子节点列表中返回选择的索引。

```py
class GrammarFuzzer(GrammarFuzzer):
    def choose_tree_expansion(self,
                              tree: DerivationTree,
                              children: List[DerivationTree]) -> int:
  """Return index of subtree in `children` to be selected for expansion.
 Defaults to random."""
        return random.randrange(0, len(children))

    def expand_tree_once(self, tree: DerivationTree) -> DerivationTree:
  """Choose an unexpanded symbol in tree; expand it.
 Can be overloaded in subclasses."""
        (symbol, children) = tree
        if children is None:
            # Expand this node
            return self.expand_node(tree)

        # Find all children with possible expansions
        expandable_children = [
            c for c in children if self.any_possible_expansions(c)]

        # `index_map` translates an index in `expandable_children`
        # back into the original index in `children`
        index_map = [i for (i, c) in enumerate(children)
                     if c in expandable_children]

        # Select a random child
        child_to_be_expanded = \
            self.choose_tree_expansion(tree, expandable_children)

        # Expand in place
        children[index_map[child_to_be_expanded]] = \
            self.expand_tree_once(expandable_children[child_to_be_expanded])

        return tree 
```</details>

让我们说明 `expand_tree_once()` 的工作原理。我们以上面的推导树开始...

```py
derivation_tree = ("<start>",
                   [("<expr>",
                     [("<expr>", None),
                      (" + ", []),
                         ("<term>", None)]
                     )])
display_tree(derivation_tree) 
```

<svg width="142pt" height="123pt" viewBox="0.00 0.00 142.25 122.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 118.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="66.25" y="-101.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="66.25" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="20.25" y="-0.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="66.25" y="-0.95" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="113.25" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>1->4</title></g></g></svg>

...现在将其展开两次：

```py
f = GrammarFuzzer(EXPR_GRAMMAR, log=True)
derivation_tree = f.expand_tree_once(derivation_tree)
display_tree(derivation_tree) 
```

```py
Expanding <expr> randomly

```

<svg width="143pt" height="173pt" viewBox="0.00 0.00 143.00 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="67" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="67" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="21" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="67" y="-51.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="114" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="21" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g></g></svg>

```py
derivation_tree = f.expand_tree_once(derivation_tree)
display_tree(derivation_tree) 
```

```py
Expanding <term> randomly

```

<svg width="208pt" height="173pt" viewBox="0.00 0.00 208.00 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="81" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="81" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="30" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="81" y="-51.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="130" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="21" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="84" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="133" y="-0.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge7" class="edge"><title>5->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="179" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge8" class="edge"><title>5->8</title></g></g></svg>

我们看到，随着每一步，会有一个符号被展开。现在只需要重复应用这个过程，进一步展开树。

## 关闭展开

使用 `expand_tree_once()`，我们可以继续扩展树 - 但我们实际上如何停止呢？这里的关键思想，由 Luke 在 [[Luke *et al*, 2000](https://doi.org/10.1109/4235.873237)] 中提出，是在将推导树膨胀到某个最大大小之后，我们*只想应用那些至少增加树大小的扩展*。例如，对于 `<factor>`，我们更倾向于扩展到 `<integer>`，因为这不会引入进一步的递归（和潜在的大小膨胀）；对于 `<integer>`，同样，扩展到 `<digit>` 更受青睐，因为它比 `<digit><integer>` 更少增加树的大小。

为了识别扩展一个符号的成本，我们引入了两个相互依赖的函数：

+   `symbol_cost()` 返回使用 `expansion_cost()` 计算每个扩展的成本后，一个符号所有扩展中的最小成本。

+   `expansion_cost()` 返回 `expansions` 中所有扩展的总和。如果在遍历过程中再次遇到非终结符，扩展的成本是 $\infty$，表示（可能是无限的）递归。

<details id="Excursion:-Implementing-Cost-Functions"><summary>实现成本函数</summary>

```py
class GrammarFuzzer(GrammarFuzzer):
    def symbol_cost(self, symbol: str, seen: Set[str] = set()) \
            -> Union[int, float]:
        expansions = self.grammar[symbol]
        return min(self.expansion_cost(e, seen | {symbol}) for e in expansions)

    def expansion_cost(self, expansion: Expansion,
                       seen: Set[str] = set()) -> Union[int, float]:
        symbols = nonterminals(expansion)
        if len(symbols) == 0:
            return 1  # no symbol

        if any(s in seen for s in symbols):
            return float('inf')

        # the value of a expansion is the sum of all expandable variables
        # inside + 1
        return sum(self.symbol_cost(s, seen) for s in symbols) + 1 
```</details>

这里有两个例子：扩展一个数字的最小成本是 1，因为我们必须从其扩展中选择一个。

```py
f = GrammarFuzzer(EXPR_GRAMMAR)
assert f.symbol_cost("<digit>") == 1 
```

然而，扩展 `<expr>` 的最小成本是五，因为这是最小数量的扩展所需。(`<expr>` $\rightarrow$ `<term>` $\rightarrow$ `<factor>` $\rightarrow$ `<integer>` $\rightarrow$ `<digit>` $\rightarrow$ 1)

```py
assert f.symbol_cost("<expr>") == 5 
```

我们定义 `expand_node_by_cost(self, node, choose)`，这是 `expand_node()` 的一个变体，它考虑了上述成本。它确定所有子节点中的最小成本 `cost`，然后使用 `choose` 函数从列表中选择一个子节点，默认情况下是成本最小。如果有多个子节点具有相同的最小成本，它将在这些子节点之间随机选择。

<details id="Excursion:-expand_node_by_cost()-implementation"><summary>`expand_node_by_cost()` 实现</summary>

```py
class GrammarFuzzer(GrammarFuzzer):
    def expand_node_by_cost(self, node: DerivationTree, 
                            choose: Callable = min) -> DerivationTree:
        (symbol, children) = node
        assert children is None

        # Fetch the possible expansions from grammar...
        expansions = self.grammar[symbol]

        children_alternatives_with_cost = [(self.expansion_to_children(expansion),
                                            self.expansion_cost(expansion, {symbol}),
                                            expansion)
                                           for expansion in expansions]

        costs = [cost for (child, cost, expansion)
                 in children_alternatives_with_cost]
        chosen_cost = choose(costs)
        children_with_chosen_cost = [child for (child, child_cost, _) 
                                     in children_alternatives_with_cost
                                     if child_cost == chosen_cost]
        expansion_with_chosen_cost = [expansion for (_, child_cost, expansion)
                                      in children_alternatives_with_cost
                                      if child_cost == chosen_cost]

        index = self.choose_node_expansion(node, children_with_chosen_cost)

        chosen_children = children_with_chosen_cost[index]
        chosen_expansion = expansion_with_chosen_cost[index]
        chosen_children = self.process_chosen_children(
            chosen_children, chosen_expansion)

        # Return with a new list
        return (symbol, chosen_children) 
```</details>

简略的 `expand_node_min_cost()` 将 `min()` 作为 `choose` 函数传递，这使得它以最小成本扩展节点。

```py
class GrammarFuzzer(GrammarFuzzer):
    def expand_node_min_cost(self, node: DerivationTree) -> DerivationTree:
        if self.log:
            print("Expanding", all_terminals(node), "at minimum cost")

        return self.expand_node_by_cost(node, min) 
```

我们现在可以使用这个函数通过 `expand_tree_once()` 和上面的 `expand_node_min_cost()` 作为扩展函数来关闭我们的推导树的扩展。

```py
class GrammarFuzzer(GrammarFuzzer):
    def expand_node(self, node: DerivationTree) -> DerivationTree:
        return self.expand_node_min_cost(node) 
```

```py
f = GrammarFuzzer(EXPR_GRAMMAR, log=True)
display_tree(derivation_tree) 
```

<svg width="208pt" height="173pt" viewBox="0.00 0.00 208.00 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="81" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="81" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="30" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="81" y="-51.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="130" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="21" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="84" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="133" y="-0.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge7" class="edge"><title>5->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="179" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge8" class="edge"><title>5->8</title></g></g></svg>

```py
if f.any_possible_expansions(derivation_tree):
    derivation_tree = f.expand_tree_once(derivation_tree)
display_tree(derivation_tree) 
```

```py
Expanding <term> at minimum cost

```

<svg width="211pt" height="223pt" viewBox="0.00 0.00 211.00 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="84" y="-201.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="84" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="33" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="84" y="-101.45" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="133" y="-101.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="24" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="24" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="87" y="-51.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="136" y="-51.2" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge8" class="edge"><title>6->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="182" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge9" class="edge"><title>6->9</title></g></g></svg>

```py
if f.any_possible_expansions(derivation_tree):
    derivation_tree = f.expand_tree_once(derivation_tree)
display_tree(derivation_tree) 
```

```py
Expanding <factor> at minimum cost

```

<svg width="215pt" height="223pt" viewBox="0.00 0.00 215.00 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="87" y="-201.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="87" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="36" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="87" y="-101.45" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="136" y="-101.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="24" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="91" y="-51.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="140" y="-51.2" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge9" class="edge"><title>6->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="186" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge10" class="edge"><title>6->10</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="93" y="-0.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge8" class="edge"><title>7->8</title></g></g></svg>

```py
if f.any_possible_expansions(derivation_tree):
    derivation_tree = f.expand_tree_once(derivation_tree)
display_tree(derivation_tree) 
```

```py
Expanding <term> at minimum cost

```

<svg width="218pt" height="223pt" viewBox="0.00 0.00 218.00 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="87" y="-201.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="87" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="36" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="87" y="-101.45" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="136" y="-101.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="24" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="91" y="-51.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="140" y="-51.2" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge9" class="edge"><title>6->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="186" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge10" class="edge"><title>6->10</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="93" y="-0.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="186" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge11" class="edge"><title>10->11</title></g></g></svg>

我们会一直扩展，直到所有非终结符都被扩展。

```py
while f.any_possible_expansions(derivation_tree):
    derivation_tree = f.expand_tree_once(derivation_tree) 
```

```py
Expanding <integer> at minimum cost
Expanding <digit> at minimum cost
Expanding <factor> at minimum cost
Expanding <integer> at minimum cost
Expanding <factor> at minimum cost
Expanding <integer> at minimum cost
Expanding <digit> at minimum cost
Expanding <digit> at minimum cost

```

这里是最终的树：

```py
display_tree(derivation_tree) 
```

<svg width="224pt" height="374pt" viewBox="0.00 0.00 224.00 374.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 370)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="90" y="-352.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="90" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="39" y="-252.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="90" y="-252.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="139" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge9" class="edge"><title>1->9</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="30" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="27" y="-0.95" font-family="Times,serif" font-size="14.00">7 (55)</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="94" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="143" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge14" class="edge"><title>9->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="189" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge15" class="edge"><title>9->15</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="96" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="96" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="96" y="-51.2" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="189" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="189" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="189" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge18" class="edge"><title>17->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="189" y="-0.95" font-family="Times,serif" font-size="14.00">8 (56)</text></g> <g id="edge19" class="edge"><title>18->19</title></g></g></svg>

我们看到在每一步中，`expand_node_min_cost()` 选择一个不会增加符号数量的扩展，最终关闭所有打开的扩展。

## 节点膨胀

尤其是在扩展的开始阶段，我们可能对获取尽可能多的节点感兴趣——也就是说，我们希望优先选择给我们提供更多可扩展非终端的扩展。这实际上与 `expand_node_min_cost()` 给我们的完全相反，我们可以实现一个 `expand_node_max_cost()` 方法，它将始终在具有最高成本的节点之间进行选择：

```py
class GrammarFuzzer(GrammarFuzzer):
    def expand_node_max_cost(self, node: DerivationTree) -> DerivationTree:
        if self.log:
            print("Expanding", all_terminals(node), "at maximum cost")

        return self.expand_node_by_cost(node, max) 
```

为了说明 `expand_node_max_cost()`，我们再次重新定义 `expand_node()` 以使用它，然后使用 `expand_tree_once()` 来展示几个扩展步骤：

```py
class GrammarFuzzer(GrammarFuzzer):
    def expand_node(self, node: DerivationTree) -> DerivationTree:
        return self.expand_node_max_cost(node) 
```

```py
derivation_tree = ("<start>",
                   [("<expr>",
                     [("<expr>", None),
                      (" + ", []),
                         ("<term>", None)]
                     )]) 
```

```py
f = GrammarFuzzer(EXPR_GRAMMAR, log=True)
display_tree(derivation_tree) 
```

<svg width="142pt" height="123pt" viewBox="0.00 0.00 142.25 122.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 118.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="66.25" y="-101.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="66.25" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="20.25" y="-0.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="66.25" y="-0.95" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="113.25" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>1->4</title></g></g></svg>

```py
if f.any_possible_expansions(derivation_tree):
    derivation_tree = f.expand_tree_once(derivation_tree)
display_tree(derivation_tree) 
```

```py
Expanding <term> at maximum cost

```

<svg width="188pt" height="173pt" viewBox="0.00 0.00 188.25 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="66.25" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="66.25" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="20.25" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="66.25" y="-51.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="113.25" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="64.25" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="113.25" y="-0.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge6" class="edge"><title>4->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="159.25" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge7" class="edge"><title>4->7</title></g></g></svg>

```py
if f.any_possible_expansions(derivation_tree):
    derivation_tree = f.expand_tree_once(derivation_tree)
display_tree(derivation_tree) 
```

```py
Expanding <factor> at maximum cost

```

<svg width="188pt" height="223pt" viewBox="0.00 0.00 188.25 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="66.25" y="-201.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="66.25" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="20.25" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="66.25" y="-101.45" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="113.25" y="-101.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="64.25" y="-51.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="113.25" y="-51.2" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge8" class="edge"><title>4->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="159.25" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge9" class="edge"><title>4->9</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="36.25" y="-0.95" font-family="Times,serif" font-size="14.00">- (45)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="93.25" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge7" class="edge"><title>5->7</title></g></g></svg>

```py
if f.any_possible_expansions(derivation_tree):
    derivation_tree = f.expand_tree_once(derivation_tree)
display_tree(derivation_tree) 
```

```py
Expanding <expr> at maximum cost

```

<svg width="300pt" height="223pt" viewBox="0.00 0.00 300.00 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="146" y="-201.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="146" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="93" y="-101.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="146" y="-101.45" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="197" y="-101.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="21" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="68" y="-51.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge4" class="edge"><title>2->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="114" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge5" class="edge"><title>2->5</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="176" y="-51.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="225" y="-51.2" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge11" class="edge"><title>7->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="271" y="-51.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge12" class="edge"><title>7->12</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="147" y="-0.95" font-family="Times,serif" font-size="14.00">- (45)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="204" y="-0.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge10" class="edge"><title>8->10</title></g></g></svg>

我们可以看到，在每一步中，非终端的数量都在增加。显然，我们必须对这个数量进行限制。

## 三种扩展阶段

我们现在可以将这三个阶段合并到一个单独的函数 `expand_tree()` 中，该函数将按以下方式工作：

1.  **最大成本扩展。**使用最大成本的扩展来扩展树，直到我们至少有 `min_nonterminals` 个非终端。这个阶段可以通过将 `min_nonterminals` 设置为零来轻松跳过。

1.  **随机扩展。**继续随机扩展树，直到我们达到 `max_nonterminals` 个非终端。

1.  **最小成本扩展。**以最小成本关闭扩展。

我们通过让 `expand_node` 引用扩展方法来实施这三个阶段。这是通过设置 `expand_node`（方法引用）首先为 `expand_node_max_cost`（即调用 `expand_node()` 将调用 `expand_node_max_cost()`），然后 `expand_node_randomly`，最后 `expand_node_min_cost` 来控制的。在前两个阶段，我们还分别设置了 `min_nonterminals` 和 `max_nonterminals` 的最大限制。

<details id="Excursion:-Implementation-of-three-phase-expand_tree()"><summary>三个阶段 `expand_tree()` 的实现</summary>

```py
class GrammarFuzzer(GrammarFuzzer):
    def log_tree(self, tree: DerivationTree) -> None:
  """Output a tree if self.log is set; if self.display is also set, show the tree structure"""
        if self.log:
            print("Tree:", all_terminals(tree))
            if self.disp:
                display(display_tree(tree))
            # print(self.possible_expansions(tree), "possible expansion(s) left")

    def expand_tree_with_strategy(self, tree: DerivationTree,
                                  expand_node_method: Callable,
                                  limit: Optional[int] = None):
  """Expand tree using `expand_node_method` as node expansion function
 until the number of possible expansions reaches `limit`."""
        self.expand_node = expand_node_method
        while ((limit is None
                or self.possible_expansions(tree) < limit)
               and self.any_possible_expansions(tree)):
            tree = self.expand_tree_once(tree)
            self.log_tree(tree)
        return tree

    def expand_tree(self, tree: DerivationTree) -> DerivationTree:
  """Expand `tree` in a three-phase strategy until all expansions are complete."""
        self.log_tree(tree)
        tree = self.expand_tree_with_strategy(
            tree, self.expand_node_max_cost, self.min_nonterminals)
        tree = self.expand_tree_with_strategy(
            tree, self.expand_node_randomly, self.max_nonterminals)
        tree = self.expand_tree_with_strategy(
            tree, self.expand_node_min_cost)

        assert self.possible_expansions(tree) == 0

        return tree 
```

让我们在我们的例子中尝试一下。我们从一个半展开的推导树开始：

```py
initial_derivation_tree: DerivationTree = ("<start>",
                   [("<expr>",
                     [("<expr>", None),
                      (" + ", []),
                         ("<term>", None)]
                     )]) 
```

```py
display_tree(initial_derivation_tree) 
```

<svg width="142pt" height="123pt" viewBox="0.00 0.00 142.25 122.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 118.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="66.25" y="-101.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="66.25" y="-51.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="20.25" y="-0.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="66.25" y="-0.95" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="113.25" y="-0.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge4" class="edge"><title>1->4</title></g></g></svg>

我们现在将我们的扩展策略应用于这棵树。我们看到，最初节点以最大成本扩展，然后随机扩展，最后以最小成本关闭扩展。

```py
f = GrammarFuzzer(
    EXPR_GRAMMAR,
    min_nonterminals=3,
    max_nonterminals=5,
    log=True)
derivation_tree = f.expand_tree(initial_derivation_tree) 
```

```py
Tree: <expr> + <term>
Expanding <expr> at maximum cost
Tree: <term> + <expr> + <term>
Expanding <expr> randomly
Tree: <term> + <term> + <term>
Expanding <term> randomly
Tree: <factor> / <term> + <term> + <term>
Expanding <term> randomly
Tree: <factor> / <factor> + <term> + <term>
Expanding <factor> randomly
Tree: <integer> / <factor> + <term> + <term>
Expanding <term> randomly
Tree: <integer> / <factor> + <factor> * <term> + <term>
Expanding <factor> at minimum cost
Tree: <integer> / <integer> + <factor> * <term> + <term>
Expanding <integer> at minimum cost
Tree: <integer> / <digit> + <factor> * <term> + <term>
Expanding <factor> at minimum cost
Tree: <integer> / <digit> + <integer> * <term> + <term>
Expanding <integer> at minimum cost
Tree: <digit> / <digit> + <integer> * <term> + <term>
Expanding <term> at minimum cost
Tree: <digit> / <digit> + <integer> * <term> + <factor>
Expanding <digit> at minimum cost
Tree: <digit> / 5 + <integer> * <term> + <factor>
Expanding <factor> at minimum cost
Tree: <digit> / 5 + <integer> * <term> + <integer>
Expanding <integer> at minimum cost
Tree: <digit> / 5 + <integer> * <term> + <digit>
Expanding <integer> at minimum cost
Tree: <digit> / 5 + <digit> * <term> + <digit>
Expanding <digit> at minimum cost
Tree: 7 / 5 + <digit> * <term> + <digit>
Expanding <digit> at minimum cost
Tree: 7 / 5 + <digit> * <term> + 0
Expanding <term> at minimum cost
Tree: 7 / 5 + <digit> * <factor> + 0
Expanding <factor> at minimum cost
Tree: 7 / 5 + <digit> * <integer> + 0
Expanding <integer> at minimum cost
Tree: 7 / 5 + <digit> * <digit> + 0
Expanding <digit> at minimum cost
Tree: 7 / 5 + 4 * <digit> + 0
Expanding <digit> at minimum cost
Tree: 7 / 5 + 4 * 2 + 0

```

这是最终的推导树：

```py
display_tree(derivation_tree) 
```

<svg width="360pt" height="475pt" viewBox="0.00 0.00 359.62 474.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 470.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="236" y="-453.2" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="236" y="-402.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="186" y="-352.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="236" y="-352.7" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge27" class="edge"><title>1->27</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="285" y="-352.7" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge28" class="edge"><title>1->28</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="111" y="-302.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="179" y="-302.45" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge14" class="edge"><title>2->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="225" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge15" class="edge"><title>2->15</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-252.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="75" y="-252.2" font-family="Times,serif" font-size="14.00">/</text></g> <g id="edge8" class="edge"><title>3->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="120" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge9" class="edge"><title>3->9</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="27" y="-201.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00">7 (55)</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="107" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="106" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="106" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="106" y="-51.2" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="225" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="176" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="225" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge21" class="edge"><title>16->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="271" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge22" class="edge"><title>16->22</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="178" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge18" class="edge"><title>17->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="178" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge19" class="edge"><title>18->19</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="178" y="-51.2" font-family="Times,serif" font-size="14.00">4 (52)</text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="271" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="271" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge24" class="edge"><title>23->24</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="271" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge25" class="edge"><title>24->25</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="271" y="-0.95" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge26" class="edge"><title>25->26</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="289" y="-302.45" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="311" y="-252.2" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge30" class="edge"><title>29->30</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="331" y="-201.95" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge31" class="edge"><title>30->31</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="331" y="-151.7" font-family="Times,serif" font-size="14.00">0 (48)</text></g> <g id="edge32" class="edge"><title>31->32</title></g></g></svg>

这就是生成的字符串：

```py
all_terminals(derivation_tree) 
```

```py
'7 / 5 + 4 * 2 + 0'

```

## 将所有这些放在一起

基于此，我们现在可以定义一个函数 `fuzz()`，它就像 `simple_grammar_fuzzer()` 一样，简单地从一个语法中生成一个字符串。因此，它不再暴露推导树的复杂性。

```py
class GrammarFuzzer(GrammarFuzzer):
    def fuzz_tree(self) -> DerivationTree:
  """Produce a derivation tree from the grammar."""
        tree = self.init_tree()
        # print(tree)

        # Expand all nonterminals
        tree = self.expand_tree(tree)
        if self.log:
            print(repr(all_terminals(tree)))
        if self.disp:
            display(display_tree(tree))
        return tree

    def fuzz(self) -> str:
  """Produce a string from the grammar."""
        self.derivation_tree = self.fuzz_tree()
        return all_terminals(self.derivation_tree) 
```

我们现在可以在所有定义的语法上应用这个策略（并可视化推导树）。

```py
f = GrammarFuzzer(EXPR_GRAMMAR)
f.fuzz() 
```

```py
'18.3 * 21.95 / 0'

```

在调用 `fuzz()` 之后，生成的推导树可以在 `derivation_tree` 属性中访问：

```py
display_tree(f.derivation_tree) 
```

<svg width="518pt" height="424pt" viewBox="0.00 0.00 517.62 424.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 420.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="274.62" y="-402.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="274.62" y="-352.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="274.62" y="-302.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="160.62" y="-252.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="274.62" y="-252.2" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge14" class="edge"><title>2->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="335.62" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge15" class="edge"><title>2->15</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="70.62" y="-201.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="130.62" y="-201.95" font-family="Times,serif" font-size="14.00">. (46)</text></g> <g id="edge10" class="edge"><title>3->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="190.62" y="-201.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge11" class="edge"><title>3->11</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="20.62" y="-151.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="86.62" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge7" class="edge"><title>4->7</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="20.62" y="-101.45" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="86.62" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="86.62" y="-51.2" font-family="Times,serif" font-size="14.00">8 (56)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="174.62" y="-151.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="157.62" y="-101.45" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="311.62" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="359.62" y="-201.95" font-family="Times,serif" font-size="14.00">/</text></g> <g id="edge30" class="edge"><title>15->30</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="422.62" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge31" class="edge"><title>15->31</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="251.62" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="311.62" y="-151.7" font-family="Times,serif" font-size="14.00">. (46)</text></g> <g id="edge23" class="edge"><title>16->23</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="371.62" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge24" class="edge"><title>16->24</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="212.62" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge18" class="edge"><title>17->18</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="278.62" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge20" class="edge"><title>17->20</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="212.62" y="-51.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge19" class="edge"><title>18->19</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="278.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="278.62" y="-0.95" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge22" class="edge"><title>21->22</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="344.62" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge25" class="edge"><title>24->25</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="410.62" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge27" class="edge"><title>24->27</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="344.62" y="-51.2" font-family="Times,serif" font-size="14.00">9 (57)</text></g> <g id="edge26" class="edge"><title>25->26</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="410.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge28" class="edge"><title>27->28</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="410.62" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="453.62" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge32" class="edge"><title>31->32</title></g> <g id="node34" class="node"><title>33</title> <text text-anchor="middle" x="482.62" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge33" class="edge"><title>32->33</title></g> <g id="node35" class="node"><title>34</title> <text text-anchor="middle" x="482.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge34" class="edge"><title>33->34</title></g> <g id="node36" class="node"><title>35</title> <text text-anchor="middle" x="482.62" y="-0.95" font-family="Times,serif" font-size="14.00">0 (48)</text></g> <g id="edge35" class="edge"><title>34->35</title></g></g></svg>

让我们在其他语法格式上尝试语法模糊器（及其树）。

```py
f = GrammarFuzzer(URL_GRAMMAR)
f.fuzz() 
```

```py
'ftp://user:password@www.google.com:53/def?abc=def'

```

```py
display_tree(f.derivation_tree) 
```

<svg width="573pt" height="324pt" viewBox="0.00 0.00 573.00 323.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 319.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="214.5" y="-302.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="214.5" y="-252.2" font-family="Times,serif" font-size="14.00"><url></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="69.5" y="-201.95" font-family="Times,serif" font-size="14.00"><scheme></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="158.5" y="-201.95" font-family="Times,serif" font-size="14.00">://</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="214.5" y="-201.95" font-family="Times,serif" font-size="14.00"><authority></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="345.5" y="-201.95" font-family="Times,serif" font-size="14.00"><path></text></g> <g id="edge18" class="edge"><title>1->18</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="440.5" y="-201.95" font-family="Times,serif" font-size="14.00"><query></text></g> <g id="edge22" class="edge"><title>1->22</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="7.5" y="-151.7" font-family="Times,serif" font-size="14.00">ftp</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="63.5" y="-151.7" font-family="Times,serif" font-size="14.00"><userinfo></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="131.5" y="-151.7" font-family="Times,serif" font-size="14.00">@ (64)</text></g> <g id="edge8" class="edge"><title>5->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="188.5" y="-151.7" font-family="Times,serif" font-size="14.00"><host></text></g> <g id="edge9" class="edge"><title>5->9</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="241.5" y="-151.7" font-family="Times,serif" font-size="14.00">: (58)</text></g> <g id="edge11" class="edge"><title>5->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="293.5" y="-151.7" font-family="Times,serif" font-size="14.00"><port></text></g> <g id="edge12" class="edge"><title>5->12</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="63.5" y="-101.45" font-family="Times,serif" font-size="14.00">user:password</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="188.5" y="-101.45" font-family="Times,serif" font-size="14.00">www.google.com</text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="293.5" y="-101.45" font-family="Times,serif" font-size="14.00"><nat></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="263.5" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="322.5" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge16" class="edge"><title>13->16</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="263.5" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="322.5" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="345.5" y="-151.7" font-family="Times,serif" font-size="14.00">/ (47)</text></g> <g id="edge19" class="edge"><title>18->19</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="392.5" y="-151.7" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge20" class="edge"><title>18->20</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="392.5" y="-101.45" font-family="Times,serif" font-size="14.00">def</text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="440.5" y="-151.7" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="502.5" y="-151.7" font-family="Times,serif" font-size="14.00"><params></text></g> <g id="edge24" class="edge"><title>22->24</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="502.5" y="-101.45" font-family="Times,serif" font-size="14.00"><param></text></g> <g id="edge25" class="edge"><title>24->25</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="453.5" y="-51.2" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge26" class="edge"><title>25->26</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="502.5" y="-51.2" font-family="Times,serif" font-size="14.00">= (61)</text></g> <g id="edge28" class="edge"><title>25->28</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="551.5" y="-51.2" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge29" class="edge"><title>25->29</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="453.5" y="-0.95" font-family="Times,serif" font-size="14.00">abc</text></g> <g id="edge27" class="edge"><title>26->27</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="551.5" y="-0.95" font-family="Times,serif" font-size="14.00">def</text></g> <g id="edge30" class="edge"><title>29->30</title></g></g></svg>

```py
f = GrammarFuzzer(CGI_GRAMMAR, min_nonterminals=3, max_nonterminals=5)
f.fuzz() 
```

```py
'e+d+'

```

```py
display_tree(f.derivation_tree) 
```

<svg width="243pt" height="374pt" viewBox="0.00 0.00 242.75 374.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 370)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="54.12" y="-352.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="54.12" y="-302.45" font-family="Times,serif" font-size="14.00"><string></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="22.12" y="-252.2" font-family="Times,serif" font-size="14.00"><letter></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="86.12" y="-252.2" font-family="Times,serif" font-size="14.00"><string></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="22.12" y="-201.95" font-family="Times,serif" font-size="14.00"><other></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="22.12" y="-151.7" font-family="Times,serif" font-size="14.00">e (101)</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="85.12" y="-201.95" font-family="Times,serif" font-size="14.00"><letter></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="149.12" y="-201.95" font-family="Times,serif" font-size="14.00"><string></text></g> <g id="edge9" class="edge"><title>5->9</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="85.12" y="-151.7" font-family="Times,serif" font-size="14.00"><plus></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="85.12" y="-101.45" font-family="Times,serif" font-size="14.00">+ (43)</text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="147.12" y="-151.7" font-family="Times,serif" font-size="14.00"><letter></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="211.12" y="-151.7" font-family="Times,serif" font-size="14.00"><string></text></g> <g id="edge13" class="edge"><title>9->13</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="147.12" y="-101.45" font-family="Times,serif" font-size="14.00"><other></text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="147.12" y="-51.2" font-family="Times,serif" font-size="14.00">d (100)</text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="211.12" y="-101.45" font-family="Times,serif" font-size="14.00"><letter></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="211.12" y="-51.2" font-family="Times,serif" font-size="14.00"><plus></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="211.12" y="-0.95" font-family="Times,serif" font-size="14.00">+ (43)</text></g> <g id="edge16" class="edge"><title>15->16</title></g></g></svg>

我们与 `simple_grammar_fuzzer()` 的表现如何？

```py
trials = 50
xs = []
ys = []
f = GrammarFuzzer(EXPR_GRAMMAR, max_nonterminals=20)
for i in range(trials):
    with Timer() as t:
        s = f.fuzz()
    xs.append(len(s))
    ys.append(t.elapsed_time())
    print(i, end=" ")
print() 
```

```py
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 

```

```py
average_time = sum(ys) / trials
print("Average time:", average_time) 
```

```py
Average time: 0.0263434316823259

```

```py
%matplotlib inline

import [matplotlib.pyplot](https://matplotlib.org/) as plt
plt.scatter(xs, ys)
plt.title('Time required for generating an output'); 
```

![](img/7a67155bf919078dff92eb49265dfe42.png)

我们生成的测试速度更快，但我们的输入也更小。我们看到，通过推导树，我们可以更好地控制语法的生成。

最后，当 `simple_grammar_fuzzer()` 失败时，`GrammarFuzzer` 如何与 `expr_grammar` 一起工作？它没有任何问题：

```py
f = GrammarFuzzer(expr_grammar, max_nonterminals=10)
f.fuzz() 
```

```py
'(9 + 7) * 7 * 2 * 5 + 7 * 2 / 6'

```

使用`GrammarFuzzer`，我们现在有一个坚实的基础来构建更多的模糊器，并展示生成软件测试领域的更多激动人心的概念。其中许多甚至不需要编写语法——相反，它们从当前领域**推断**出一个语法，因此即使不编写语法也可以使用基于语法的模糊测试。请保持关注！

## 经验教训

+   **推导树**对于表达输入结构很重要

+   *基于推导树的语法模糊测试*

    1.  比基于字符串的语法模糊测试更有效率，

    1.  对输入生成提供了更好的控制，

    1.  有效避免了无限扩展的问题。

## 下一步

恭喜！你已经到达了本书的核心“枢纽”之一。从这里，有一系列基于语法模糊的技术。

### 扩展语法

首先，我们有一些技术，它们都以某种形式**扩展**语法：

+   解析和重新组合输入允许使用现有输入，再次使用推导树

+   覆盖语法扩展允许**组合**覆盖

+   将**概率**分配给单个扩展提供了对扩展的额外控制

+   将**约束**分配给单个扩展允许对单个规则表达**语义约束**。

### 应用语法

其次，我们可以在各种涉及某种形式自动学习的上下文中**应用**语法：

+   模糊 API，从 API 学习语法

+   模糊图形用户界面，从用户界面学习语法以进行后续模糊测试

+   挖掘语法，学习任意输入格式的语法

继续扩展！

## 背景

推导树（通常称为**解析树**）是一种标准的数据结构，解析器将输入分解到其中。在《龙书》（也称为《编译原理、技术和工具》）[[Aho 等人，2006](https://www.pearson.com/us/higher-education/program/Aho-Compilers-Principles-Techniques-and-Tools-2nd-Edition/PGM167067.html)]中讨论了将解析作为编译程序的一部分到推导树中。我们也在解析和重新组合输入时使用推导树。

本章的关键思想，即扩展直到达到符号的限制，然后总是选择最短路径，源自 Luke [[Luke 等人，2000](https://doi.org/10.1109/4235.873237)]。

## 练习

### 练习 1：缓存方法结果

跟踪`GrammarFuzzer`显示某些方法被反复调用，总是使用相同的值。

设置一个具有**缓存**的类`FasterGrammarFuzzer`，该缓存检查方法是否之前已被调用，如果是，则返回之前计算的“缓存”值。对`expansion_to_children()`做这件事。比较优化前后的调用次数。

**重要**：对于 `expansion_to_children()`，确保返回的每个列表都是单独的副本。如果你返回相同的（缓存的）列表，这将干扰 `GrammarFuzzer` 的就地修改。为此，请使用 Python 的 `copy.deepcopy()` 函数。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GrammarFuzzer.ipynb#Exercises)来练习题目并查看解决方案。

### 练习 2：语法预编译

一些方法，如 `symbol_cost()` 或 `expansion_cost()`，返回一个仅依赖于语法的值。设置一个类 `EvenFasterGrammarFuzzer()`，在初始化时预先计算这些值，这样后续调用 `symbol_cost()` 或 `expansion_cost()` 只需查找这些值。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GrammarFuzzer.ipynb#Exercises)来练习题目并查看解决方案。

### 练习 3：维护要扩展的树

在 `expand_tree_once()` 中，算法反复遍历树以找到仍然可以扩展的非终结符。通过保留树中仍然可以扩展的非终结符符号列表来加快这个过程。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GrammarFuzzer.ipynb#Exercises)来练习题目并查看解决方案。

### 练习 4：交替随机扩展

我们可以定义 `expand_node_randomly()`，使其简单地调用 `expand_node_by_cost(node, random.choice)`：

```py
class ExerciseGrammarFuzzer(GrammarFuzzer):
    def expand_node_randomly(self, node: DerivationTree) -> DerivationTree:
        if self.log:
            print("Expanding", all_terminals(node), "randomly by cost")

        return self.expand_node_by_cost(node, random.choice) 
```

原始实现和这个替代方案有什么区别？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GrammarFuzzer.ipynb#Exercises)来练习题目并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后更改：2023-11-11 18:18:06+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/GrammarFuzzer.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用这篇作品

安德烈亚斯·泽勒（Andreas Zeller）、拉胡尔·戈皮纳特（Rahul Gopinath）、马塞尔·博姆（Marcel Böhme）、戈登·弗朗西斯（Gordon Fraser）和克里斯蒂安·霍勒（Christian Holler）: "[高效的语法模糊测试](https://www.fuzzingbook.org/html/GrammarFuzzer.html)". 在安德烈亚斯·泽勒、拉胡尔·戈皮纳特、马塞尔·博姆、戈登·弗朗西斯和克里斯蒂安·霍勒的 "[模糊测试书](https://www.fuzzingbook.org/)" 中，[`www.fuzzingbook.org/html/GrammarFuzzer.html`](https://www.fuzzingbook.org/html/GrammarFuzzer.html). 2023-11-11 18:18:06+01:00 获取。

```py
@incollection{fuzzingbook2023:GrammarFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Efficient Grammar Fuzzing},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/GrammarFuzzer.html}},
    note = {Retrieved 2023-11-11 18:18:06+01:00},
    url = {https://www.fuzzingbook.org/html/GrammarFuzzer.html},
    urldate = {2023-11-11 18:18:06+01:00}
}

```
