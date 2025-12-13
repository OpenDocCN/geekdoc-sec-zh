# 解析输入

> 原文：[`www.fuzzingbook.org/html/Parser.html`](http://www.fuzzingbook.org/html/Parser.html)

在语法章节中，我们讨论了语法如何用来表示各种语言。我们还看到了语法如何用来生成对应语言的字符串。语法还可以执行相反的操作。也就是说，给定一个字符串，可以将该字符串分解为其构成部分，这些部分对应于生成它的语法部分——该字符串的*推导树*。这些部分（以及来自其他类似字符串的部分）可以稍后使用相同的语法重新组合，以生成新的字符串。

在本章中，我们使用语法来解析和分解给定的一组有效种子输入到它们对应的推导树。这种结构表示允许我们突变、交叉和重新组合它们的部分，以生成新的有效、略有变化的输入（即模糊测试）。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('k39i9de0L54') 
```

**先决条件**

+   您应该已经阅读了关于语法的章节。

+   还需要理解语法模糊测试章节中的推导树。

## 概述

要使用本章提供的代码（Importing.html），请编写

```py
>>> from fuzzingbook.Parser import <identifier> 
```

然后利用以下功能。

本章介绍了 `Parser` 类，将字符串解析为*推导树*，如关于高效语法模糊测试的章节中所述。提供了两个重要的解析器类：

+   解析表达式语法解析器 (`PEGParser`)。这些非常高效，但限于特定的语法结构。值得注意的是，备选方案代表*有序选择*。也就是说，我们不会选择所有可能匹配的规则，而是在第一个成功匹配的地方停止。

+   Earley 解析器 (`EarleyParser`)。这些解析器接受任何类型的上下文无关语法，并探索所有解析备选方案（如果有的话）。

使用这些中的任何一个都相当简单。首先，用一种语法实例化它们：

```py
>>> from Grammars import US_PHONE_GRAMMAR
>>> us_phone_parser = EarleyParser(US_PHONE_GRAMMAR) 
```

然后，使用 `parse()` 方法检索可能的推导树列表：

```py
>>> trees = us_phone_parser.parse("(555)987-6543")
>>> tree = list(trees)[0]
>>> display_tree(tree) 
```

<svg width="636pt" height="223pt" viewBox="0.00 0.00 635.75 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="265.12" y="-201.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="265.12" y="-151.7" font-family="Times,serif" font-size="14.00"><phone-number></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="113.12" y="-101.45" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="166.12" y="-101.45" font-family="Times,serif" font-size="14.00"><area></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="232.12" y="-101.45" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge10" class="edge"><title>1->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="299.12" y="-101.45" font-family="Times,serif" font-size="14.00"><exchange></text></g> <g id="edge11" class="edge"><title>1->11</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="366.12" y="-101.45" font-family="Times,serif" font-size="14.00">- (45)</text></g> <g id="edge18" class="edge"><title>1->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="489.12" y="-101.45" font-family="Times,serif" font-size="14.00"><line></text></g> <g id="edge19" class="edge"><title>1->19</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="34.12" y="-51.2" font-family="Times,serif" font-size="14.00"><lead-digit></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="107.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge6" class="edge"><title>3->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="166.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge8" class="edge"><title>3->8</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="34.12" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="107.12" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="166.12" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="239.12" y="-51.2" font-family="Times,serif" font-size="14.00"><lead-digit></text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="312.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge14" class="edge"><title>11->14</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="371.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge16" class="edge"><title>11->16</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="239.12" y="-0.95" font-family="Times,serif" font-size="14.00">9 (57)</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="312.12" y="-0.95" font-family="Times,serif" font-size="14.00">8 (56)</text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="371.12" y="-0.95" font-family="Times,serif" font-size="14.00">7 (55)</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="430.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="489.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge22" class="edge"><title>19->22</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="548.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge24" class="edge"><title>19->24</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="607.12" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge26" class="edge"><title>19->26</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="430.12" y="-0.95" font-family="Times,serif" font-size="14.00">6 (54)</text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="489.12" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="548.12" y="-0.95" font-family="Times,serif" font-size="14.00">4 (52)</text></g> <g id="edge25" class="edge"><title>24->25</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="607.12" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge27" class="edge"><title>26->27</title></g></g></svg>

这些推导树可以用于测试生成，特别是用于突变和重新组合现有输入。

<svg width="382pt" height="395pt" viewBox="0.00 0.00 381.62 394.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 390.5)"><g id="node1" class="node"><title>PEGParser</title> <g id="a_node1"><a xlink:href="#" xlink:title="class PEGParser:

用于解析表达式语法（PEGs）的 Packrat 解析器。《PEGParser》"><text text-anchor="start" x="18.12" y="-126.58" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">PEGParser</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="PEGParser"><g id="a_node1_1"><a xlink:href="#" xlink:title="parse_prefix(self, text):

返回文本的最长前缀的（光标，森林）对。

在子类中定义。"><text text-anchor="start" x="8" y="-104.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">parse_prefix()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="unify_rule(self, rule, text, at)"><text text-anchor="start" x="8" y="-90.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">unify_rule()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="unify_key(self, key, text, at=0)"><text text-anchor="start" x="8" y="-77.88" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">unify_key()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>解析器</title> <g id="a_node2"><a xlink:href="#" xlink:title="class Parser:

解析的基类。"><text text-anchor="start" x="93.5" y="-369.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Parser</text> <g id="a_node2_4"><a xlink:href="#" xlink:title="Parser"><g id="a_node2_5"><a xlink:href="#" xlink:title="__init__(self, grammar, **kwargs):

构造函数。

`grammar` 是用于解析的语法。

关键字参数：

`start_symbol` 是起始符号（默认：'<start>'）。

`log` 启用日志记录（默认：False）。

`coalesce` 定义了是否应该合并标记（默认：True）。

`tokens`，如果设置，是用于的标记集合。"><text text-anchor="start" x="71" y="-347.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__init__()</text></a></g> <g id="a_node2_6"><a xlink:href="#" xlink:title="grammar(self) -> Grammar:

返回此解析器的语法。"><text text-anchor="start" x="71" y="-334.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">grammar()</text></a></g> <g id="a_node2_7"><a xlink:href="#" xlink:title="parse(self, text: str) -> Iterable[DerivationTree]:

使用语法解析 `text`。

返回一个解析树的迭代器。"><text text-anchor="start" x="71" y="-322" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">parse()</text></a></g> <g id="a_node2_8"><a xlink:href="#" xlink:title="start_symbol(self) -> str:

返回此解析器的起始符号。"><text text-anchor="start" x="71" y="-309.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">start_symbol()</text></a></g> <g id="a_node2_9"><a xlink:href="#" xlink:title="coalesce(self, children: List[DerivationTree]) -> List[DerivationTree]"><text text-anchor="start" x="71" y="-295.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">coalesce()</text></a></g> <g id="a_node2_10"><a xlink:href="#" xlink:title="parse_on(self, text: str, start_symbol: str) -> Generator"><text text-anchor="start" x="71" y="-282.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">parse_on()</text></a></g> <g id="a_node2_11"><a xlink:href="#" xlink:title="parse_prefix(self, text: str) -> Tuple[int, Iterable[DerivationTree]]:

返回文本的最长前缀的（光标，森林）对。

在子类中定义。《text text-anchor="start" x="71" y="-271" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">parse_prefix()</text></a></g> <g id="a_node2_12"><a xlink:href="#" xlink:title="prune_tree(self, tree)"><text text-anchor="start" x="71" y="-257.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">prune_tree()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>PEGParser->Parser</title></g> <g id="node3" class="node"><title>EarleyParser</title> <g id="a_node3"><a xlink:href="#" xlink:title="class EarleyParser:

Earley 解析器。此解析器可以解析任何上下文无关文法。"><text text-anchor="start" x="138" y="-196.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">EarleyParser</text> <g id="a_node3_13"><a xlink:href="#" xlink:title="EarleyParser"><g id="a_node3_14"><a xlink:href="#" xlink:title="__init__(self, grammar, **kwargs):

构造函数。

`grammar` 是用于解析的文法。

关键字参数：

`start_symbol` 是起始符号（默认: '<start>'）。

`log` 启用日志记录（默认: False）。

`coalesce` 定义是否应该合并标记（默认: True）。

`tokens`, 如果设置，是一个将要使用的标记集合。"><text text-anchor="start" x="126" y="-174.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node3_15"><a xlink:href="#" xlink:title="chart_parse(self, words, start)"><text text-anchor="start" x="126" y="-160.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">chart_parse()</text></a></g> <g id="a_node3_16"><a xlink:href="#" xlink:title="complete(self, col, state)"><text text-anchor="start" x="126" y="-148" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">complete()</text></a></g> <g id="a_node3_17"><a xlink:href="#" xlink:title="earley_complete(self, col, state)"><text text-anchor="start" x="126" y="-135.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">earley_complete()</text></a></g> <g id="a_node3_18"><a xlink:href="#" xlink:title="extract_a_tree(self, forest_node)"><text text-anchor="start" x="126" y="-122.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">extract_a_tree()</text></a></g> <g id="a_node3_19"><a xlink:href="#" xlink:title="extract_trees(self, forest_node)"><text text-anchor="start" x="126" y="-109.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">extract_trees()</text></a></g> <g id="a_node3_20"><a xlink:href="#" xlink:title="fill_chart(self, chart)"><text text-anchor="start" x="126" y="-97" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">fill_chart()</text></a></g> <g id="a_node3_21"><a xlink:href="#" xlink:title="forest(self, s, kind, chart)"><text text-anchor="start" x="126" y="-84.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">forest()</text></a></g> <g id="a_node3_22"><a xlink:href="#" xlink:title="parse(self, text):

使用语法解析`text`。

返回一个解析树的可迭代对象。"><text text-anchor="start" x="126" y="-72.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">parse()</text></a></g> <g id="a_node3_23"><a xlink:href="#" xlink:title="parse_forest(self, chart, state)"><text text-anchor="start" x="126" y="-58.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">parse_forest()</text></a></g> <g id="a_node3_24"><a xlink:href="#" xlink:title="parse_paths(self, named_expr, chart, frm, til)"><text text-anchor="start" x="126" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">parse_paths()</text></a></g> <g id="a_node3_25"><a xlink:href="#" xlink:title="parse_prefix(self, text):

返回文本的最长前缀的（光标，森林）对。"><text text-anchor="start" x="126" y="-34" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">parse_prefix()</text></a></g>

在子类中定义。"><text text-anchor="start" x="126" y="-34.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">parse_prefix()</text></a></g> <g id="a_node3_26"><a xlink:href="#" xlink:title="predict(self, col, sym, state)"><text text-anchor="start" x="126" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">predict()</text></a></g> <g id="a_node3_27"><a xlink:href="#" xlink:title="scan(self, col, state, letter)"><text text-anchor="start" x="126" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">scan()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>EarleyParser->Parser</title></g> <g id="node4" class="node"><title>图例</title> <text text-anchor="start" x="254.38" y="-122.75" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="254.38" y="-112.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="260.38" y="-112.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">公共方法()</text> <text text-anchor="start" x="254.38" y="-102.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="260.38" y="-102.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">私有方法()</text> <text text-anchor="start" x="254.38" y="-92.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="260.38" y="-92.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">重载方法()</text> <text text-anchor="start" x="254.38" y="-83.7" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import Dict, List, Tuple, Collection, Set, Iterable, Generator, cast 
```

```py
from Fuzzer import Fuzzer  # minor dependency 
```

```py
from Grammars import EXPR_GRAMMAR, START_SYMBOL, RE_NONTERMINAL
from Grammars import is_valid_grammar, syntax_diagram, Grammar 
```

```py
from GrammarFuzzer import GrammarFuzzer, display_tree, tree_to_string, dot_escape
from GrammarFuzzer import DerivationTree 
```

```py
from ExpectError import ExpectError 
```

```py
from [IPython.display](https://ipython.readthedocs.io/en/stable/api/generated/IPython.display.html) import display 
```

```py
from Timer import Timer 
```

## 为什么进行模糊测试的解析？

为什么想要解析现有的输入来进行模糊测试？让我们用一个例子来说明这个问题。这是一个简单的程序，它接受一个包含车辆详情的 CSV 文件并处理这些信息。

```py
def process_inventory(inventory):
    res = []
    for vehicle in inventory.split('\n'):
        ret = process_vehicle(vehicle)
        res.extend(ret)
    return '\n'.join(res) 
```

CSV 文件每行包含一辆车的详细信息。每一行都在`process_vehicle()`中处理。

```py
def process_vehicle(vehicle):
    year, kind, company, model, *_ = vehicle.split(',')
    if kind == 'van':
        return process_van(year, company, model)

    elif kind == 'car':
        return process_car(year, company, model)

    else:
        raise Exception('Invalid entry') 
```

根据车辆类型的不同，处理方式也会变化。

```py
def process_van(year, company, model):
    res = ["We have a %s  %s van from %s vintage." % (company, model, year)]
    iyear = int(year)
    if iyear > 2010:
        res.append("It is a recent model!")
    else:
        res.append("It is an old but reliable model!")
    return res 
```

```py
def process_car(year, company, model):
    res = ["We have a %s  %s car from %s vintage." % (company, model, year)]
    iyear = int(year)
    if iyear > 2016:
        res.append("It is a recent model!")
    else:
        res.append("It is an old but reliable model!")
    return res 
```

这里是`process_inventory()`接受的输入样本。

```py
mystring = """\
1997,van,Ford,E350
2000,car,Mercury,Cougar\
"""
print(process_inventory(mystring)) 
```

```py
We have a Ford E350 van from 1997 vintage.
It is an old but reliable model!
We have a Mercury Cougar car from 2000 vintage.
It is an old but reliable model!

```

让我们尝试对这个程序进行模糊测试。鉴于`process_inventory()`接受 CSV 文件，我们可以编写一个简单的语法来生成逗号分隔的值，并生成所需的 CSV 行。为了方便起见，我们直接模糊`process_vehicle()`。

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
CSV_GRAMMAR: Grammar = {
    '<start>': ['<csvline>'],
    '<csvline>': ['<items>'],
    '<items>': ['<item>,<items>', '<item>'],
    '<item>': ['<letters>'],
    '<letters>': ['<letter><letters>', '<letter>'],
    '<letter>': list(string.ascii_letters + string.digits + string.punctuation + ' \t\n')
} 
```

我们首先需要一些基础设施来查看语法。

```py
syntax_diagram(CSV_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 199.5 62" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">csvline</text></g></g></g></g></svg>

```py
csvline

```

<svg class="railroad-diagram" height="62" viewBox="0 0 182.5 62" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">items</text></g></g></g></g></svg>

```py
items

```

<svg class="railroad-diagram" height="92" viewBox="0 0 305.0 92" width="305.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">item</text></g> <g class="terminal"><text x="148.25" y="35">,</text></g> <g class="non-terminal"><text x="213.75" y="35">items</text></g></g> <g><g class="non-terminal"><text x="152.5" y="65">item</text></g></g></g></g></svg>

```py
item

```

<svg class="railroad-diagram" height="62" viewBox="0 0 199.5 62" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">letters</text></g></g></g></g></svg>

```py
letters

```

<svg class="railroad-diagram" height="92" viewBox="0 0 290.5 92" width="290.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="95.5" y="35">letter</text></g> <g class="non-terminal"><text x="190.75" y="35">letters</text></g></g> <g><g class="non-terminal"><text x="145.25" y="65">letter</text></g></g></g></g></svg>

```py
letter

```

<svg class="railroad-diagram" height="618" viewBox="0 0 611.0 618" width="611.0"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="283">i</text></g></g> <g><g class="terminal"><text x="84.25" y="253">h</text></g></g> <g><g class="terminal"><text x="84.25" y="223">g</text></g></g> <g><g class="terminal"><text x="84.25" y="193">f</text></g></g> <g><g class="terminal"><text x="84.25" y="163">e</text></g></g> <g><g class="terminal"><text x="84.25" y="133">d</text></g></g> <g><g class="terminal"><text x="84.25" y="103">c</text></g></g> <g><g class="terminal"><text x="84.25" y="73">b</text></g></g> <g><g class="terminal"><text x="84.25" y="43">a</text></g></g> <g><g class="terminal"><text x="84.25" y="313">j</text></g></g> <g><g class="terminal"><text x="84.25" y="343">k</text></g></g> <g><g class="terminal"><text x="84.25" y="373">l</text></g></g> <g><g class="terminal"><text x="84.25" y="403">m</text></g></g> <g><g class="terminal"><text x="84.25" y="433">n</text></g></g> <g><g class="terminal"><text x="84.25" y="463">o</text></g></g> <g><g class="terminal"><text x="84.25" y="493">p</text></g></g> <g><g class="terminal"><text x="84.25" y="523">q</text></g></g> <g><g class="terminal"><text x="84.25" y="553">r</text></g></g> <g><g class="terminal"><text x="84.25" y="583">s</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="283">B</text></g></g> <g><g class="terminal"><text x="172.75" y="253">A</text></g></g> <g><g class="terminal"><text x="172.75" y="223">z</text></g></g> <g><g class="terminal"><text x="172.75" y="193">y</text></g></g> <g><g class="terminal"><text x="172.75" y="163">x</text></g></g> <g><g class="terminal"><text x="172.75" y="133">w</text></g></g> <g><g class="terminal"><text x="172.75" y="103">v</text></g></g> <g><g class="terminal"><text x="172.75" y="73">u</text></g></g> <g><g class="terminal"><text x="172.75" y="43">t</text></g></g> <g><g class="terminal"><text x="172.75" y="313">C</text></g></g> <g><g class="terminal"><text x="172.75" y="343">D</text></g></g> <g><g class="terminal"><text x="172.75" y="373">E</text></g></g> <g><g class="terminal"><text x="172.75" y="403">F</text></g></g> <g><g class="terminal"><text x="172.75" y="433">G</text></g></g> <g><g class="terminal"><text x="172.75" y="463">H</text></g></g> <g><g class="terminal"><text x="172.75" y="493">I</text></g></g> <g><g class="terminal"><text x="172.75" y="523">J</text></g></g> <g><g class="terminal"><text x="172.75" y="553">K</text></g></g> <g><g class="terminal"><text x="172.75" y="583">L</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="283">U</text></g></g> <g><g class="terminal"><text x="261.25" y="253">T</text></g></g> <g><g class="terminal"><text x="261.25" y="223">S</text></g></g> <g><g class="terminal"><text x="261.25" y="193">R</text></g></g> <g><g class="terminal"><text x="261.25" y="163">Q</text></g></g> <g><g class="terminal"><text x="261.25" y="133">P</text></g></g> <g><g class="terminal"><text x="261.25" y="103">O</text></g></g> <g><g class="terminal"><text x="261.25" y="73">N</text></g></g> <g><g class="terminal"><text x="261.25" y="43">M</text></g></g> <g><g class="terminal"><text x="261.25" y="313">V</text></g></g> <g><g class="terminal"><text x="261.25" y="343">W</text></g></g> <g><g class="terminal"><text x="261.25" y="373">X</text></g></g> <g><g class="terminal"><text x="261.25" y="403">Y</text></g></g> <g><g class="terminal"><text x="261.25" y="433">Z</text></g></g> <g><g class="terminal"><text x="261.25" y="463">0</text></g></g> <g><g class="terminal"><text x="261.25" y="493">1</text></g></g> <g><g class="terminal"><text x="261.25" y="523">2</text></g></g> <g><g class="terminal"><text x="261.25" y="553">3</text></g></g> <g><g class="terminal"><text x="261.25" y="583">4</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="283">$</text></g></g> <g><g class="terminal"><text x="349.75" y="253">#</text></g></g> <g><g class="terminal"><text x="349.75" y="223">"</text></g></g> <g><g class="terminal"><text x="349.75" y="193">!</text></g></g> <g><g class="terminal"><text x="349.75" y="163">9</text></g></g> <g><g class="terminal"><text x="349.75" y="133">8</text></g></g> <g><g class="terminal"><text x="349.75" y="103">7</text></g></g> <g><g class="terminal"><text x="349.75" y="73">6</text></g></g> <g><g class="terminal"><text x="349.75" y="43">5</text></g></g> <g><g class="terminal"><text x="349.75" y="313">%</text></g></g> <g><g class="terminal"><text x="349.75" y="343">&</text></g></g> <g><g class="terminal"><text x="349.75" y="373">'</text></g></g> <g><g class="terminal"><text x="349.75" y="403">(</text></g></g> <g><g class="terminal"><text x="349.75" y="433">)</text></g></g> <g><g class="terminal"><text x="349.75" y="463">*</text></g></g> <g><g class="terminal"><text x="349.75" y="493">+</text></g></g> <g><g class="terminal"><text x="349.75" y="523">,</text></g></g> <g><g class="terminal"><text x="349.75" y="553">-</text></g></g> <g><g class="terminal"><text x="349.75" y="583">.</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="283">[</text></g></g> <g><g class="terminal"><text x="438.25" y="253">@</text></g></g> <g><g class="terminal"><text x="438.25" y="223">?</text></g></g> <g><g class="terminal"><text x="438.25" y="193">></text></g></g> <g><g class="terminal"><text x="438.25" y="163">=</text></g></g> <g><g class="terminal"><text x="438.25" y="133"><</text></g></g> <g><g class="terminal"><text x="438.25" y="103">;</text></g></g> <g><g class="terminal"><text x="438.25" y="73">:</text></g></g> <g><g class="terminal"><text x="438.25" y="43">/</text></g></g> <g><g class="terminal"><text x="438.25" y="313">\</text></g></g> <g><g class="terminal"><text x="438.25" y="343">]</text></g></g> <g><g class="terminal"><text x="438.25" y="373">^</text></g></g> <g><g class="terminal"><text x="438.25" y="403">_</text></g></g> <g><g class="terminal"><text x="438.25" y="433">`</text></g></g> <g><g class="terminal"><text x="438.25" y="463">{</text></g></g> <g><g class="terminal"><text x="438.25" y="493">|</text></g></g> <g><g class="terminal"><text x="438.25" y="523">}</text></g></g> <g><g class="terminal"><text x="438.25" y="553">~</text></g></g></g></g></g></svg>

我们生成了`1000`个值，并用每个值评估`process_vehicle()`。

```py
gf = GrammarFuzzer(CSV_GRAMMAR, min_nonterminals=4)
trials = 1000
valid: List[str] = []
time = 0
for i in range(trials):
    with Timer() as t:
        vehicle_info = gf.fuzz()
        try:
            process_vehicle(vehicle_info)
            valid.append(vehicle_info)
        except:
            pass
        time += t.elapsed_time()
print("%d valid strings, that is GrammarFuzzer generated %f%% valid entries from %d inputs" %
      (len(valid), len(valid) * 100.0 / trials, trials))
print("Total time of %f seconds" % time) 
```

```py
0 valid strings, that is GrammarFuzzer generated 0.000000% valid entries from 1000 inputs
Total time of 2.478398 seconds

```

这显然是不起作用的。但为什么呢？

```py
gf = GrammarFuzzer(CSV_GRAMMAR, min_nonterminals=4)
trials = 10
time = 0
for i in range(trials):
    vehicle_info = gf.fuzz()
    try:
        print(repr(vehicle_info), end="")
        process_vehicle(vehicle_info)
    except Exception as e:
        print("\t", e)
    else:
        print() 
```

```py
'9w9J\'/,LU<"l,|,Y,Zv)Amvx,c\n'	 Invalid entry
'(n8].H7,qolS'	 not enough values to unpack (expected at least 4, got 2)
'\nQoLWQ,jSa'	 not enough values to unpack (expected at least 4, got 2)
'K1,\n,RE,fq,%,,sT+aAb'	 Invalid entry
"m,d,,8j4'),-yQ,B7"	 Invalid entry
'g4,s1\t[}{.,M,<,\nzd,.am'	 Invalid entry
',Z[,z,c,#x1,gc.F'	 Invalid entry
'pWs,rT`,R'	 not enough values to unpack (expected at least 4, got 3)
'iN,br%,Q,R'	 Invalid entry
'ol,\nH<\tn,^#,=A'	 Invalid entry

```

除非 fuzzer 能够生成`van`或`car`，否则没有任何条目能够通过。实际上，原因是语法本身并没有捕获关于格式的完整信息。所以这里有一个想法。我们修改`GrammarFuzzer`以了解我们的格式。

```py
import [copy](https://docs.python.org/3/library/copy.html) 
```

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
class PooledGrammarFuzzer(GrammarFuzzer):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._node_cache = {}

    def update_cache(self, key, values):
        self._node_cache[key] = values

    def expand_node_randomly(self, node):
        (symbol, children) = node
        assert children is None
        if symbol in self._node_cache:
            if random.randint(0, 1) == 1:
                return super().expand_node_randomly(node)
            return copy.deepcopy(random.choice(self._node_cache[symbol]))
        return super().expand_node_randomly(node) 
```

让我们再试一次！

```py
gf = PooledGrammarFuzzer(CSV_GRAMMAR, min_nonterminals=4)
gf.update_cache('<item>', [
    ('<item>', [('car', [])]),
    ('<item>', [('van', [])]),
])
trials = 10
time = 0
for i in range(trials):
    vehicle_info = gf.fuzz()
    try:
        print(repr(vehicle_info), end="")
        process_vehicle(vehicle_info)
    except Exception as e:
        print("\t", e)
    else:
        print() 
```

```py
',h,van,|'	 Invalid entry
'M,w:K,car,car,van'	 Invalid entry
'J,?Y,van,van,car,J,~D+'	 Invalid entry
'S4,car,car,o'	 invalid literal for int() with base 10: 'S4'
'2*-,van'	 not enough values to unpack (expected at least 4, got 2)
'van,%,5,]'	 Invalid entry
'van,G3{y,j,h:'	 Invalid entry
'$0;o,M,car,car'	 Invalid entry
'2d,f,e'	 not enough values to unpack (expected at least 4, got 3)
'/~NE,car,car'	 not enough values to unpack (expected at least 4, got 3)

```

至少我们正在取得进展！如果*我们能够在我们的 fuzzer 中结合我们对样本数据的了解*，那就太好了。实际上，如果我们可以从样本中*提取*模板和有效值，并在我们的模糊测试中使用它们，那就更好了。我们如何做到这一点？对这个问题的快速回答是：使用*解析器*。

## 使用解析器

一般而言，*解析器*是程序处理（结构化）输入的部分。我们本章讨论的解析器将输入字符串转换成*推导树*（在关于高效语法模糊测试的章节中讨论）。从用户的角度来看，解析输入只需要两个步骤：

1.  使用语法初始化解析器，如下所示

    ```py
    parser = Parser(grammar)
    ```

1.  使用解析器检索推导树列表：

```py
trees = parser.parse(input) 
```

一旦我们解析了一个树，我们就可以像从语法模糊测试中产生的推导树一样使用它。

我们讨论了许多这样的解析器，特别是

+   解析表达式语法解析器 (`PEGParser`)，它们非常高效，但仅限于特定的语法结构；

+   Earley 解析器 (`EarleyParser`)，它接受任何类型的上下文无关语法。

如果你只是想*使用*解析器（比如说，因为你的主要重点是测试），你只需在这里停下来，继续下一章，在那里我们将学习如何利用解析输入来变异和重新组合它们。不过，如果你想*理解*解析器是如何工作的，那么这一章就是为你准备的。

## 临时解析器

正如我们在上一节中看到的，程序员经常必须提取遵循某些规则的数据部分。例如，对于*CSV*文件，每一行的元素之间由*逗号*分隔，并且使用多行来存储数据。

为了提取信息，我们编写了一个临时的解析器`simple_parse_csv()`。

```py
def simple_parse_csv(mystring: str) -> DerivationTree:
    children: List[DerivationTree] = []
    tree = (START_SYMBOL, children)
    for i, line in enumerate(mystring.split('\n')):
        children.append(("record %d" % i, [(cell, [])
                                           for cell in line.split(',')]))
    return tree 
```

我们还更改了图表的默认方向，将其从*从上到下*改为*从左到右*，以便使用`lr_graph()`更容易查看。

```py
def lr_graph(dot):
    dot.attr('node', shape='plain')
    dot.graph_attr['rankdir'] = 'LR' 
```

`display_tree()`显示了我们的 CSV 文件在解析后的结构。

```py
tree = simple_parse_csv(mystring)
display_tree(tree, graph_attr=lr_graph) 
```

<svg width="212pt" height="246pt" viewBox="0.00 0.00 212.00 246.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 242.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-112.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="98.25" y="-160.95" font-family="Times,serif" font-size="14.00">record 0</text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="98.25" y="-64.95" font-family="Times,serif" font-size="14.00">record 1</text></g> <g id="edge6" class="edge"><title>0->6</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="180.38" y="-224.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="180.38" y="-192.95" font-family="Times,serif" font-size="14.00">van</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="180.38" y="-160.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="180.38" y="-128.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="180.38" y="-96.95" font-family="Times,serif" font-size="14.00">2000</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="180.38" y="-64.95" font-family="Times,serif" font-size="14.00">car</text></g> <g id="edge8" class="edge"><title>6->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="180.38" y="-32.95" font-family="Times,serif" font-size="14.00">Mercury</text></g> <g id="edge9" class="edge"><title>6->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="180.38" y="-0.95" font-family="Times,serif" font-size="14.00">Cougar</text></g> <g id="edge10" class="edge"><title>6->10</title></g></g></svg>

这当然是简单的。如果我们遇到稍微复杂一些的情况呢？再次，另一个来自维基百科的例子。

```py
mystring = '''\
1997,Ford,E350,"ac, abs, moon",3000.00\
'''
print(mystring) 
```

```py
1997,Ford,E350,"ac, abs, moon",3000.00

```

我们定义了一种新的注释方法`highlight_node()`来标记有趣的节点。

```py
def highlight_node(predicate):
    def hl_node(dot, nid, symbol, ann):
        if predicate(dot, nid, symbol, ann):
            dot.node(repr(nid), dot_escape(symbol), fontcolor='red')
        else:
            dot.node(repr(nid), dot_escape(symbol))
    return hl_node 
```

使用`highlight_node()`我们可以突出显示那些被错误解析的特定节点。

```py
tree = simple_parse_csv(mystring)
bad_nodes = {5, 6, 7, 12, 13, 20, 22, 23, 24, 25} 
```

```py
def hl_predicate(_d, nid, _s, _a): return nid in bad_nodes 
```

```py
highlight_err_node = highlight_node(hl_predicate)
display_tree(tree, log=False, node_attr=highlight_err_node,
             graph_attr=lr_graph) 
```

<svg width="209pt" height="214pt" viewBox="0.00 0.00 209.00 214.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 210.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-96.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="98.25" y="-96.95" font-family="Times,serif" font-size="14.00">record 0</text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="178.88" y="-192.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="178.88" y="-160.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="178.88" y="-128.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="178.88" y="-96.95" font-family="Times,serif" font-size="14.00" fill="red">"ac</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="178.88" y="-64.95" font-family="Times,serif" font-size="14.00" fill="red">abs</text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="178.88" y="-32.95" font-family="Times,serif" font-size="14.00" fill="red">moon"</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="178.88" y="-0.95" font-family="Times,serif" font-size="14.00">3000.00</text></g> <g id="edge8" class="edge"><title>1->8</title></g></g></svg>

标记的节点表示我们的解析出错的地方。我们当然可以扩展我们的解析器以理解引号。首先我们定义一些辅助函数`parse_quote()`、`find_comma()`和`comma_split()`

```py
def parse_quote(string, i):
    v = string[i + 1:].find('"')
    return v + i + 1 if v >= 0 else -1 
```

```py
def find_comma(string, i):
    slen = len(string)
    while i < slen:
        if string[i] == '"':
            i = parse_quote(string, i)
            if i == -1:
                return -1
        if string[i] == ',':
            return i
        i += 1
    return -1 
```

```py
def comma_split(string):
    slen = len(string)
    i = 0
    while i < slen:
        c = find_comma(string, i)
        if c == -1:
            yield string[i:]
            return
        else:
            yield string[i:c]
        i = c + 1 
```

我们可以将我们的`parse_csv()`过程更新为使用我们的高级引号解析器。

```py
def parse_csv(mystring):
    children = []
    tree = (START_SYMBOL, children)
    for i, line in enumerate(mystring.split('\n')):
        children.append(("record %d" % i, [(cell, [])
                                           for cell in comma_split(line)]))
    return tree 
```

我们新的`parse_csv()`现在可以正确处理引号。

```py
tree = parse_csv(mystring)
display_tree(tree, graph_attr=lr_graph) 
```

<svg width="253pt" height="150pt" viewBox="0.00 0.00 253.25 150.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 146.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-64.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="98.25" y="-64.95" font-family="Times,serif" font-size="14.00">record 0</text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="201" y="-128.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="201" y="-96.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="201" y="-64.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="201" y="-32.95" font-family="Times,serif" font-size="14.00">"ac, abs, moon"</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="201" y="-0.95" font-family="Times,serif" font-size="14.00">3000.00</text></g> <g id="edge6" class="edge"><title>1->6</title></g></g></svg>

当然，这并不能长久地存活：

```py
mystring = '''\
1999,Chevy,"Venture \\"Extended Edition, Very Large\\"",,5000.00\
'''
print(mystring) 
```

```py
1999,Chevy,"Venture \"Extended Edition, Very Large\"",,5000.00

```

几个嵌入的引号就足以再次让我们的解析器困惑。

```py
tree = parse_csv(mystring)
bad_nodes = {4, 5}
display_tree(tree, node_attr=highlight_err_node, graph_attr=lr_graph) 
```

<svg width="321pt" height="168pt" viewBox="0.00 0.00 320.75 168.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 164.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-66.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="98.25" y="-66.95" font-family="Times,serif" font-size="14.00">record 0</text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="234.75" y="-146.95" font-family="Times,serif" font-size="14.00">1999</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="234.75" y="-114.95" font-family="Times,serif" font-size="14.00">Chevy</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="234.75" y="-82.95" font-family="Times,serif" font-size="14.00" fill="red">"Venture \"Extended Edition</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="234.75" y="-50.95" font-family="Times,serif" font-size="14.00" fill="red">Very Large\""</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="234.75" y="-0.95" font-family="Times,serif" font-size="14.00">5000.00</text></g> <g id="edge7" class="edge"><title>1->7</title></g></g></svg>

这里是 CSV 文件中的另一个记录：

```py
mystring = '''\
1996,Jeep,Grand Cherokee,"MUST SELL!
air, moon roof, loaded",4799.00
'''
print(mystring) 
```

```py
1996,Jeep,Grand Cherokee,"MUST SELL!
air, moon roof, loaded",4799.00

```

```py
tree = parse_csv(mystring)
bad_nodes = {5, 6, 7, 8, 9, 10}
display_tree(tree, node_attr=highlight_err_node, graph_attr=lr_graph) 
```

<svg width="259pt" height="214pt" viewBox="0.00 0.00 258.50 214.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 210.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-48.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="98.25" y="-128.95" font-family="Times,serif" font-size="14.00">record 0</text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="98.25" y="-48.95" font-family="Times,serif" font-size="14.00" fill="red">record 1</text></g> <g id="edge6" class="edge"><title>0->6</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="98.25" y="-16.95" font-family="Times,serif" font-size="14.00" fill="red">record 2</text></g> <g id="edge10" class="edge"><title>0->10</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="203.62" y="-192.95" font-family="Times,serif" font-size="14.00">1996</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="203.62" y="-160.95" font-family="Times,serif" font-size="14.00">Jeep</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="203.62" y="-128.95" font-family="Times,serif" font-size="14.00">Grand Cherokee</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="203.62" y="-96.95" font-family="Times,serif" font-size="14.00" fill="red">"MUST SELL!</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="203.62" y="-64.95" font-family="Times,serif" font-size="14.00" fill="red">air</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="203.62" y="-32.95" font-family="Times,serif" font-size="14.00" fill="red">moon roof</text></g> <g id="edge8" class="edge"><title>6->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="203.62" y="-0.95" font-family="Times,serif" font-size="14.00" fill="red">loaded",4799.00</text></g> <g id="edge9" class="edge"><title>6->9</title></g></g></svg>

修复这个问题需要修改内层的`parse_quote()`和外层的`parse_csv()`过程。我们注意到，这些特性实际上在 CSV [RFC 4180](https://tools.ietf.org/html/rfc4180)中都有记录。

事实上，任何额外的改进都会因为一点额外的复杂性而崩溃。当遇到递归表达式时，问题变得严重。例如，JSON 是 CSV 文件的常见替代品，用于保存数据。同样，如果数据来自网络，可能不得不从 HTML 表格而不是 CSV 文件中解析数据。

人们可能会倾向于用一点更具体的解析方法来修复它，加入一些*正则表达式*。然而，这却是[通向疯狂的道路](https://stackoverflow.com/a/1732454)。

正式解析器在这里大放异彩。主要思想是，任何给定的一组字符串属于一种语言，这些语言可以通过它们的语法来指定（正如我们在语法章节中看到的）。语法的好处在于它们可以被*组合*。也就是说，可以在不影响外部结构的情况下，将更细致的细节引入内部结构，同样，也可以在不严重影响内部结构的情况下改变外部结构。

## 解析中的语法

我们简要描述了在解析上下文中的语法。

<details id="Excursion:-Grammars-and-Derivation-Trees"><summary>语法和推导树</summary>

正如您在语法章节中读到的，语法是一组*规则*，它解释了起始符号如何被扩展。每条规则都有一个名称，也称为*非终结符*，以及一组非终结符可以如何扩展的*选择方案*。

```py
A1_GRAMMAR: Grammar = {
    "<start>": ["<expr>"],
    "<expr>": ["<expr>+<expr>", "<expr>-<expr>", "<integer>"],
    "<integer>": ["<digit><integer>", "<digit>"],
    "<digit>": ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
} 
```

```py
syntax_diagram(A1_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">expr</text></g></g></g></g></svg>

```py
expr

```

<svg class="railroad-diagram" height="122" viewBox="0 0 296.5 122" width="296.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">expr</text></g> <g class="terminal"><text x="148.25" y="35">+</text></g> <g class="non-terminal"><text x="209.5" y="35">expr</text></g></g> <g><g class="non-terminal"><text x="87.0" y="65">expr</text></g> <g class="terminal"><text x="148.25" y="65">-</text></g> <g class="non-terminal"><text x="209.5" y="65">expr</text></g></g> <g><g class="non-terminal"><text x="148.25" y="95">integer</text></g></g></g></g></svg>

```py
integer

```

<svg class="railroad-diagram" height="92" viewBox="0 0 282.0 92" width="282.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">digit</text></g> <g class="non-terminal"><text x="182.25" y="35">integer</text></g></g> <g><g class="non-terminal"><text x="141.0" y="65">digit</text></g></g></g></g></svg>

```py
digit

```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

在上述表达式中，规则`<expr> : [<expr>+<expr>,<expr>-<expr>,<integer>]`对应于非终结符`<expr>`可能如何扩展。表达式`<expr>+<expr>`对应于一种选择方案。我们称这种扩展为非终结符`<expr>`的*选择*扩展。最后，在表达式`<expr>+<expr>`中，`<expr>`、`+`和`<expr>`都是该扩展中的*符号*。根据其扩展是否在语法中可用，符号可以是非终结符或终结符。

这里是一个表示我们想要解析的算术表达式的字符串，该表达式由上面的语法指定：

```py
mystring = '1+2' 
```

由这个语法得出的我们表达式的*推导树*如下所示：

```py
tree = ('<start>', [('<expr>',
                     [('<expr>', [('<integer>', [('<digit>', [('1', [])])])]),
                      ('+', []),
                      ('<expr>', [('<integer>', [('<digit>', [('2',
                                                               [])])])])])])
assert mystring == tree_to_string(tree)
display_tree(tree) 
```

<svg width="174pt" height="274pt" viewBox="0.00 0.00 174.00 273.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 269.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="83" y="-252.2" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="83" y="-201.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="83" y="-151.7" font-family="Times,serif" font-size="14.00">+ (43)</text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="139" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="27" y="-0.95" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="139" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="139" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="139" y="-0.95" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge10" class="edge"><title>9->10</title></g></g></svg>

虽然语法可以用来指定一种给定的语言，但可能存在多个语法对应于同一种语言。例如，这里还有一个语法来描述相同的加法表达式。

```py
A2_GRAMMAR: Grammar = {
    "<start>": ["<expr>"],
    "<expr>": ["<integer><expr_>"],
    "<expr_>": ["+<expr>", "-<expr>", ""],
    "<integer>": ["<digit><integer_>"],
    "<integer_>": ["<integer>", ""],
    "<digit>": ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
} 
```

```py
syntax_diagram(A2_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">expr</text></g></g></g></g></svg>

```py
expr

```

<svg class="railroad-diagram" height="62" viewBox="0 0 282.0 62" width="282.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">integer</text></g> <g class="non-terminal"><text x="190.75" y="35">expr_</text></g></g></g></g></svg>

```py
expr_

```

<svg class="railroad-diagram" height="122" viewBox="0 0 222.5 122" width="222.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">+</text></g> <g class="non-terminal"><text x="135.5" y="35">expr</text></g></g> <g><g class="terminal"><text x="74.25" y="65">-</text></g> <g class="non-terminal"><text x="135.5" y="65">expr</text></g></g></g></g></svg>

```py
integer

```

<svg class="railroad-diagram" height="62" viewBox="0 0 290.5 62" width="290.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">digit</text></g> <g class="non-terminal"><text x="186.5" y="35">integer_</text></g></g></g></g></svg>

```py
integer_

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">integer</text></g></g></g></g></svg>

```py
digit

```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

相应的推导树如下所示：

```py
tree = ('<start>', [('<expr>', [('<integer>', [('<digit>', [('1', [])]),
                                               ('<integer_>', [])]),
                                ('<expr_>', [('+', []),
                                             ('<expr>',
                                              [('<integer>',
                                                [('<digit>', [('2', [])]),
                                                 ('<integer_>', [])]),
                                               ('<expr_>', [])])])])])
assert mystring == tree_to_string(tree)
display_tree(tree) 
```

<svg width="278pt" height="324pt" viewBox="0.00 0.00 278.25 323.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 319.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="122.62" y="-302.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="122.62" y="-252.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="88.62" y="-201.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="157.62" y="-201.95" font-family="Times,serif" font-size="14.00"><expr_></text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="20.62" y="-151.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="89.62" y="-151.7" font-family="Times,serif" font-size="14.00"><integer_></text></g> <g id="edge5" class="edge"><title>2->5</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="20.62" y="-101.45" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="156.62" y="-151.7" font-family="Times,serif" font-size="14.00">+ (43)</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="212.62" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge8" class="edge"><title>6->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="177.62" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="246.62" y="-101.45" font-family="Times,serif" font-size="14.00"><expr_></text></g> <g id="edge13" class="edge"><title>8->13</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="142.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="211.62" y="-51.2" font-family="Times,serif" font-size="14.00"><integer_></text></g> <g id="edge12" class="edge"><title>9->12</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="142.62" y="-0.95" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge11" class="edge"><title>10->11</title></g></g></svg>

事实上，可能存在不同类别的语法来描述同一种语言。例如，第一个语法`A1_GRAMMAR`是一种同时具有*右递归*和*左递归*的语法，而第二个语法`A2_GRAMMAR`在其任何产生式中的非终结符中都没有左递归，但包含*ε产生式*。（ε产生式是指其右侧为空字符串的产生式。）</details> <details id="Excursion:-Recursion"><summary>递归</summary>

你可能已经注意到我们在自己的定义中重复使用了术语`<expr>`。在自己的定义中使用相同的非终结符称为*递归*。在解析中，应该注意两种特定的递归类型，正如我们在下一节中看到的。

#### 递归

如果一个语法的任何非终结符都是左递归的，则该语法是*左递归的*，如果一个非终结符的任何产生式的最左边符号本身，则该非终结符是*直接左递归的*。

```py
LR_GRAMMAR: Grammar = {
    '<start>': ['<A>'],
    '<A>': ['<A>a', ''],
} 
```

```py
syntax_diagram(LR_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="74.25" y="35">A</text></g></g></g></g></svg>

```py
A

```

<svg class="railroad-diagram" height="92" viewBox="0 0 197.0 92" width="197.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="74.25" y="35">A</text></g> <g class="terminal"><text x="122.75" y="35">a</text></g></g></g></g></svg>

```py
mystring = 'aaaaaa'
display_tree(
    ('<start>', [('<A>', [('<A>', [('<A>', []), ('a', [])]), ('a', [])]),
                 ('a', [])])) 
```

<svg width="130pt" height="173pt" viewBox="0.00 0.00 130.25 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="82.12" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="59.12" y="-101.45" font-family="Times,serif" font-size="14.00"><A></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="106.12" y="-101.45" font-family="Times,serif" font-size="14.00">a (97)</text></g> <g id="edge6" class="edge"><title>0->6</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="36.12" y="-51.2" font-family="Times,serif" font-size="14.00"><A></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="83.12" y="-51.2" font-family="Times,serif" font-size="14.00">a (97)</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="13.12" y="-0.95" font-family="Times,serif" font-size="14.00"><A></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="60.12" y="-0.95" font-family="Times,serif" font-size="14.00">a (97)</text></g> <g id="edge4" class="edge"><title>2->4</title></g></g></svg>

如果任何最左边的符号可以通过其定义扩展为产生非终结符作为扩展的最左边符号，则语法是间接左递归的。如果在非终结符的连续扩展过程中，达到一个规则，该规则在其它符号的前缀之后包含相同的非终结符，并且这些符号可以推导出空字符串，则左递归被称为*隐藏左递归*。例如，在`A1_GRAMMAR`中，如果`<digit>`可以推导出空字符串，则`<integer>`将被认为是隐藏左递归的。

右递归语法定义方式类似。下面是表示与`LR_GRAMMAR`相同语言的右递归语法的推导树。

```py
RR_GRAMMAR: Grammar = {
    '<start>': ['<A>'],
    '<A>': ['a<A>', ''],
} 
```

```py
syntax_diagram(RR_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="74.25" y="35">A</text></g></g></g></g></svg>

```py
A

```

<svg class="railroad-diagram" height="92" viewBox="0 0 197.0 92" width="197.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">a</text></g> <g class="non-terminal"><text x="122.75" y="35">A</text></g></g></g></g></svg>

```py
display_tree(('<start>', [('<A>', [
                  ('a', []), ('<A>', [('a', []), ('<A>', [('a', []), ('<A>', [])])])])]
             )) 
```

<svg width="130pt" height="223pt" viewBox="0.00 0.00 130.25 223.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 219.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="39.12" y="-201.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="39.12" y="-151.7" font-family="Times,serif" font-size="14.00"><A></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="16.12" y="-101.45" font-family="Times,serif" font-size="14.00">a (97)</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="63.12" y="-101.45" font-family="Times,serif" font-size="14.00"><A></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="39.12" y="-51.2" font-family="Times,serif" font-size="14.00">a (97)</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="86.12" y="-51.2" font-family="Times,serif" font-size="14.00"><A></text></g> <g id="edge5" class="edge"><title>3->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="62.12" y="-0.95" font-family="Times,serif" font-size="14.00">a (97)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="109.12" y="-0.95" font-family="Times,serif" font-size="14.00"><A></text></g> <g id="edge7" class="edge"><title>5->7</title></g></g></svg>

#### 二义性

为了使问题更加复杂，可能存在多个推导树——也称为*解析*——对应于同一语法中的同一字符串。例如，字符串`1+2+3`可以使用`A1_GRAMMAR`以两种方式解析，如下所示：

```py
mystring = '1+2+3'
tree = ('<start>',
        [('<expr>',
          [('<expr>', [('<expr>', [('<integer>', [('<digit>', [('1', [])])])]),
                       ('+', []),
                       ('<expr>', [('<integer>',
                                    [('<digit>', [('2', [])])])])]), ('+', []),
           ('<expr>', [('<integer>', [('<digit>', [('3', [])])])])])])
assert mystring == tree_to_string(tree)
display_tree(tree) 
```

<svg width="239pt" height="324pt" viewBox="0.00 0.00 239.00 323.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 319.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="145" y="-302.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="145" y="-252.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="87" y="-201.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="145" y="-201.95" font-family="Times,serif" font-size="14.00">+ (43)</text></g> <g id="edge12" class="edge"><title>1->12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="202" y="-201.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge13" class="edge"><title>1->13</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="83" y="-151.7" font-family="Times,serif" font-size="14.00">+ (43)</text></g> <g id="edge7" class="edge"><title>2->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="139" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge8" class="edge"><title>2->8</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="27" y="-0.95" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="138" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="138" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="138" y="-0.95" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="204" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="204" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="204" y="-51.2" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge16" class="edge"><title>15->16</title></g></g></svg>

```py
tree = ('<start>',
        [('<expr>', [('<expr>', [('<integer>', [('<digit>', [('1', [])])])]),
                     ('+', []),
                     ('<expr>',
                      [('<expr>', [('<integer>', [('<digit>', [('2', [])])])]),
                       ('+', []),
                       ('<expr>', [('<integer>', [('<digit>', [('3',
                                                                [])])])])])])])
assert tree_to_string(tree) == mystring
display_tree(tree) 
```

<svg width="239pt" height="324pt" viewBox="0.00 0.00 239.00 323.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 319.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="89" y="-302.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="89" y="-252.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="31" y="-201.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="89" y="-201.95" font-family="Times,serif" font-size="14.00">+ (43)</text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="146" y="-201.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="27" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="92" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="148" y="-151.7" font-family="Times,serif" font-size="14.00">+ (43)</text></g> <g id="edge12" class="edge"><title>7->12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="204" y="-151.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge13" class="edge"><title>7->13</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="93" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="93" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="93" y="-0.95" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="204" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="204" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="204" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge16" class="edge"><title>15->16</title></g></g></svg>

有许多方法可以解决二义性。在下一节中解释的*解析表达式语法*采取的一种方法是指定特定的解决顺序，并选择第一个。另一种方法是简单地返回所有可能的生产树，这是我们稍后开发的*Earley 解析器*采取的方法。</details>

## 解析器类

接下来，我们开发不同的解析器。为了做到这一点，我们定义了一个所有解析器都必须遵守的最小解析接口。使用语法解析字符串有两种方法。

1.  传统的方法是使用一个*词法分析器*（也称为*标记器*或*扫描器*）首先对输入的字符串进行标记化，然后一次喂给语法一个标记。词法分析器通常是一个较小的解析器，它接受*正则语言*。这种方法的优势在于，解析器使用的语法可以避免标记化的细节。此外，在解析结束时，可以得到一个浅层推导树，可以直接用于生成*抽象语法树*。

1.  第二种方法是使用完整的解析后的树剪枝器。使用这种方法，一个人使用包含语法完整细节的语法。接下来，与标记对应的节点被剪枝，并用它们对应的字符串作为叶节点替换。这种方法的优势在于解析器更强大，并且没有人工区分*词法分析*和*解析*。

在本章中，我们使用第二种方法。这种方法在`prune_tree`方法中实现。

我们下面定义的`*Parser*`类提供了最小的接口。实现此接口的类需要实现的主要方法是`parse_prefix`和`parse`。`parse_prefix`返回一个元组，其中包含成功解析到的索引，以及直到该索引的解析森林。如果解析成功，`parse`方法返回一个推导树列表。

```py
class Parser:
  """Base class for parsing."""

    def __init__(self, grammar: Grammar, *,
                 start_symbol: str = START_SYMBOL,
                 log: bool = False,
                 coalesce: bool = True,
                 tokens: Set[str] = set()) -> None:
  """Constructor.
 `grammar` is the grammar to be used for parsing.
 Keyword arguments:
 `start_symbol` is the start symbol (default: '<start>').
 `log` enables logging (default: False).
 `coalesce` defines if tokens should be coalesced (default: True).
 `tokens`, if set, is a set of tokens to be used."""
        self._grammar = grammar
        self._start_symbol = start_symbol
        self.log = log
        self.coalesce_tokens = coalesce
        self.tokens = tokens

    def grammar(self) -> Grammar:
  """Return the grammar of this parser."""
        return self._grammar

    def start_symbol(self) -> str:
  """Return the start symbol of this parser."""
        return self._start_symbol

    def parse_prefix(self, text: str) -> Tuple[int, Iterable[DerivationTree]]:
  """Return pair (cursor, forest) for longest prefix of text. 
 To be defined in subclasses."""
        raise NotImplementedError

    def parse(self, text: str) -> Iterable[DerivationTree]:
  """Parse `text` using the grammar. 
 Return an iterable of parse trees."""
        cursor, forest = self.parse_prefix(text)
        if cursor < len(text):
            raise SyntaxError("at " + repr(text[cursor:]))
        return [self.prune_tree(tree) for tree in forest]

    def parse_on(self, text: str, start_symbol: str) -> Generator:
        old_start = self._start_symbol
        try:
            self._start_symbol = start_symbol
            yield from self.parse(text)
        finally:
            self._start_symbol = old_start

    def coalesce(self, children: List[DerivationTree]) -> List[DerivationTree]:
        last = ''
        new_lst: List[DerivationTree] = []
        for cn, cc in children:
            if cn not in self._grammar:
                last += cn
            else:
                if last:
                    new_lst.append((last, []))
                    last = ''
                new_lst.append((cn, cc))
        if last:
            new_lst.append((last, []))
        return new_lst

    def prune_tree(self, tree: DerivationTree) -> DerivationTree:
        name, children = tree
        assert isinstance(children, list)

        if self.coalesce_tokens:
            children = self.coalesce(cast(List[DerivationTree], children))
        if name in self.tokens:
            return (name, [(tree_to_string(tree), [])])
        else:
            return (name, [self.prune_tree(c) for c in children]) 
```

<details id="Excursion:-Canonical-Grammars"><summary>规范语法</summary>

我们从语法章节导入的`EXPR_GRAMMAR`面向生成。特别是，生成规则存储为字符串。我们需要稍微调整这种表示法，以符合*规范表示法*，其中每个规则中的每个标记都单独表示。`canonical`格式使用单独的标记来表示扩展中的每个符号。

```py
CanonicalGrammar = Dict[str, List[List[str]]] 
```

```py
import [re](https://docs.python.org/3/library/re.html) 
```

```py
def single_char_tokens(grammar: Grammar) -> Dict[str, List[List[Collection[str]]]]:
    g_ = {}
    for key in grammar:
        rules_ = []
        for rule in grammar[key]:
            rule_ = []
            for token in rule:
                if token in grammar:
                    rule_.append(token)
                else:
                    rule_.extend(token)
            rules_.append(rule_)
        g_[key] = rules_
    return g_ 
```

```py
def canonical(grammar: Grammar) -> CanonicalGrammar:
    def split(expansion):
        if isinstance(expansion, tuple):
            expansion = expansion[0]

        return [token for token in re.split(
            RE_NONTERMINAL, expansion) if token]

    return {
        k: [split(expression) for expression in alternatives]
        for k, alternatives in grammar.items()
    } 
```

```py
CE_GRAMMAR: CanonicalGrammar = canonical(EXPR_GRAMMAR)
CE_GRAMMAR 
```

```py
{'<start>': [['<expr>']],
 '<expr>': [['<term>', ' + ', '<expr>'],
  ['<term>', ' - ', '<expr>'],
  ['<term>']],
 '<term>': [['<factor>', ' * ', '<term>'],
  ['<factor>', ' / ', '<term>'],
  ['<factor>']],
 '<factor>': [['+', '<factor>'],
  ['-', '<factor>'],
  ['(', '<expr>', ')'],
  ['<integer>', '.', '<integer>'],
  ['<integer>']],
 '<integer>': [['<digit>', '<integer>'], ['<digit>']],
 '<digit>': [['0'],
  ['1'],
  ['2'],
  ['3'],
  ['4'],
  ['5'],
  ['6'],
  ['7'],
  ['8'],
  ['9']]}

```

我们还提供了一种方便的方法，以便更容易地显示规范语法。

```py
def recurse_grammar(grammar, key, order):
    rules = sorted(grammar[key])
    old_len = len(order)
    for rule in rules:
        for token in rule:
            if token not in grammar: continue
            if token not in order:
                order.append(token)
    new = order[old_len:]
    for ckey in new:
        recurse_grammar(grammar, ckey, order) 
```

```py
def show_grammar(grammar, start_symbol=START_SYMBOL):
    order = [start_symbol]
    recurse_grammar(grammar, start_symbol, order)
    return {k: sorted(grammar[k]) for k in order} 
```

```py
show_grammar(CE_GRAMMAR) 
```

```py
{'<start>': [['<expr>']],
 '<expr>': [['<term>'],
  ['<term>', ' + ', '<expr>'],
  ['<term>', ' - ', '<expr>']],
 '<term>': [['<factor>'],
  ['<factor>', ' * ', '<term>'],
  ['<factor>', ' / ', '<term>']],
 '<factor>': [['(', '<expr>', ')'],
  ['+', '<factor>'],
  ['-', '<factor>'],
  ['<integer>'],
  ['<integer>', '.', '<integer>']],
 '<integer>': [['<digit>'], ['<digit>', '<integer>']],
 '<digit>': [['0'],
  ['1'],
  ['2'],
  ['3'],
  ['4'],
  ['5'],
  ['6'],
  ['7'],
  ['8'],
  ['9']]}

```

我们提供了一种回滚规范表达式的方法。

```py
def non_canonical(grammar):
    new_grammar = {}
    for k in grammar:
        rules = grammar[k]
        new_rules = []
        for rule in rules:
            new_rules.append(''.join(rule))
        new_grammar[k] = new_rules
    return new_grammar 
```

```py
non_canonical(CE_GRAMMAR) 
```

```py
{'<start>': ['<expr>'],
 '<expr>': ['<term> + <expr>', '<term> - <expr>', '<term>'],
 '<term>': ['<factor> * <term>', '<factor> / <term>', '<factor>'],
 '<factor>': ['+<factor>',
  '-<factor>',
  '(<expr>)',
  '<integer>.<integer>',
  '<integer>'],
 '<integer>': ['<digit><integer>', '<digit>'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']}

```

在解析过程中使用`canonical`表示法更容易。因此，我们更新了我们的解析器类，以存储`canonical`表示法。

```py
class Parser(Parser):
    def __init__(self, grammar, **kwargs):
        self._start_symbol = kwargs.get('start_symbol', START_SYMBOL)
        self.log = kwargs.get('log', False)
        self.tokens = kwargs.get('tokens', set())
        self.coalesce_tokens = kwargs.get('coalesce', True)
        canonical_grammar = kwargs.get('canonical', False)
        if canonical_grammar:
            self.cgrammar = single_char_tokens(grammar)
            self._grammar = non_canonical(grammar)
        else:
            self._grammar = dict(grammar)
            self.cgrammar = single_char_tokens(canonical(grammar))
        # we do not require a single rule for the start symbol
        if len(grammar.get(self._start_symbol, [])) != 1:
            self.cgrammar['<>'] = [[self._start_symbol]] 
```

我们更新了`prune_tree()`以考虑如果插入了一个假起始符号。

```py
class Parser(Parser):
    def prune_tree(self, tree):
        name, children = tree
        if name == '<>':
            assert len(children) == 1
            return self.prune_tree(children[0])
        if self.coalesce_tokens:
            children = self.coalesce(children)
        if name in self.tokens:
            return (name, [(tree_to_string(tree), [])])
        else:
            return (name, [self.prune_tree(c) for c in children]) 
```

## 解析表达式语法

一种*[解析表达式语法](http://bford.info/pub/lang/peg)* (*PEG*) [[Ford *et al*, 2004](https://doi.org/10.1145/982962.964011)] 是一种基于*识别*的形式语法，它指定了解析给定字符串所需的步骤序列。解析表达式语法与我们在语法章节中看到的*上下文无关语法* (*CFG*)非常相似。与 CFG 一样，解析表达式语法由一组非终结符和相应的备选方案表示，这些备选方案表示如何匹配每个符号。例如，这里是一个匹配`a`或`b`的 PEG。

```py
PEG1 = {
    '<start>': ['a', 'b']
} 
```

然而，与*CFG*不同，备选方案代表*有序选择*。也就是说，我们不会选择所有可能匹配的规则，而是在第一个成功匹配的地方停止。例如，下面的*PEG*可以匹配`ab`但不能匹配`abc`，而*CFG*会匹配两者。（我们称有序选择表达式的序列为*选择表达式*，而不是备选方案，以区分*CFG*。）

```py
PEG2 = {
    '<start>': ['ab', 'abc']
} 
```

*选择表达式* 中的每个选择代表满足该特定选择的方法规则。选择是一系列符号（终结符和非终结符），它们与给定的文本匹配，就像在 *CFG* 中一样。

除了我们迄今为止看到的语法定义的语法之外，*PEG* 还可以包含一些额外的元素。有关更多信息，请参阅本章末尾的练习。

PEGs 模拟了手写递归下降解析器中的典型实践，因此它可能被认为更容易理解。

### 用于谓词表达式语法的 Packrat 解析器

在不手动编写解析器的情况下，*Packrat* 解析是一种最简单的解析技术之一，也是解析 PEGs 的技术之一。*Packrat* 解析器之所以得名，是因为它试图缓存所有简单问题的结果，希望这些解决方案可以在以后避免重新计算。我们接下来开发一个最小的 *Packrat* 解析器。

我们首先从 `Parser` 基类派生，并在 `parse()` 方法中接受要解析的文本，该方法反过来调用 `unify_key()` 并使用 `start_symbol`。

**注意。**虽然我们的 PEG 解析器只能生成单个无歧义的解析树，但其他解析器可以为歧义语法生成多个解析。因此，我们返回一个树列表（在这种情况下只有一个元素）。

```py
class PEGParser(Parser):
    def parse_prefix(self, text):
        cursor, tree = self.unify_key(self.start_symbol(), text, 0)
        return cursor, [tree] 
```

<details id="Excursion:-Implementing-PEGParser"><summary>实现 `PEGParser`</summary>

#### 统一键

`unify_key()` 算法很简单。如果给定一个终结符，它尝试将符号与文本中的当前位置匹配。如果符号和文本匹配，它成功返回新的解析索引 `at`。

如果另一方面，它被给了一个非终结符，它检索与键对应的选项表达式，并尝试使用 `unify_rule()` 按顺序匹配每个选择。如果 **任何** 规则成功与给定的文本统一化，则解析被认为是成功的，我们返回 `unify_rule()` 返回的新解析索引。

```py
class PEGParser(PEGParser):
  """Packrat parser for Parsing Expression Grammars (PEGs)."""

    def unify_key(self, key, text, at=0):
        if self.log:
            print("unify_key: %s with %s" % (repr(key), repr(text[at:])))
        if key not in self.cgrammar:
            if text[at:].startswith(key):
                return at + len(key), (key, [])
            else:
                return at, None
        for rule in self.cgrammar[key]:
            to, res = self.unify_rule(rule, text, at)
            if res is not None:
                return (to, (key, res))
        return 0, None 
```

```py
mystring = "1"
peg = PEGParser(EXPR_GRAMMAR, log=True)
peg.unify_key('1', mystring) 
```

```py
unify_key: '1' with '1'

```

```py
(1, ('1', []))

```

```py
mystring = "2"
peg.unify_key('1', mystring) 
```

```py
unify_key: '1' with '2'

```

```py
(0, None)

```

#### 统一规则

`unify_rule()` 方法类似。它检索需要与文本统一化的规则对应的标记，并按顺序对它们调用 `unify_key()`。如果 **所有** 标记都成功地与文本统一化，则解析成功。

```py
class PEGParser(PEGParser):
    def unify_rule(self, rule, text, at):
        if self.log:
            print('unify_rule: %s with %s' % (repr(rule), repr(text[at:])))
        results = []
        for token in rule:
            at, res = self.unify_key(token, text, at)
            if res is None:
                return at, None
            results.append(res)
        return at, results 
```

```py
mystring = "0"
peg = PEGParser(EXPR_GRAMMAR, log=True)
peg.unify_rule(peg.cgrammar['<digit>'][0], mystring, 0) 
```

```py
unify_rule: ['0'] with '0'
unify_key: '0' with '0'

```

```py
(1, [('0', [])])

```

```py
mystring = "12"
peg.unify_rule(peg.cgrammar['<integer>'][0], mystring, 0) 
```

```py
unify_rule: ['<digit>', '<integer>'] with '12'
unify_key: '<digit>' with '12'
unify_rule: ['0'] with '12'
unify_key: '0' with '12'
unify_rule: ['1'] with '12'
unify_key: '1' with '12'
unify_key: '<integer>' with '2'
unify_rule: ['<digit>', '<integer>'] with '2'
unify_key: '<digit>' with '2'
unify_rule: ['0'] with '2'
unify_key: '0' with '2'
unify_rule: ['1'] with '2'
unify_key: '1' with '2'
unify_rule: ['2'] with '2'
unify_key: '2' with '2'
unify_key: '<integer>' with ''
unify_rule: ['<digit>', '<integer>'] with ''
unify_key: '<digit>' with ''
unify_rule: ['0'] with ''
unify_key: '0' with ''
unify_rule: ['1'] with ''
unify_key: '1' with ''
unify_rule: ['2'] with ''
unify_key: '2' with ''
unify_rule: ['3'] with ''
unify_key: '3' with ''
unify_rule: ['4'] with ''
unify_key: '4' with ''
unify_rule: ['5'] with ''
unify_key: '5' with ''
unify_rule: ['6'] with ''
unify_key: '6' with ''
unify_rule: ['7'] with ''
unify_key: '7' with ''
unify_rule: ['8'] with ''
unify_key: '8' with ''
unify_rule: ['9'] with ''
unify_key: '9' with ''
unify_rule: ['<digit>'] with ''
unify_key: '<digit>' with ''
unify_rule: ['0'] with ''
unify_key: '0' with ''
unify_rule: ['1'] with ''
unify_key: '1' with ''
unify_rule: ['2'] with ''
unify_key: '2' with ''
unify_rule: ['3'] with ''
unify_key: '3' with ''
unify_rule: ['4'] with ''
unify_key: '4' with ''
unify_rule: ['5'] with ''
unify_key: '5' with ''
unify_rule: ['6'] with ''
unify_key: '6' with ''
unify_rule: ['7'] with ''
unify_key: '7' with ''
unify_rule: ['8'] with ''
unify_key: '8' with ''
unify_rule: ['9'] with ''
unify_key: '9' with ''
unify_rule: ['<digit>'] with '2'
unify_key: '<digit>' with '2'
unify_rule: ['0'] with '2'
unify_key: '0' with '2'
unify_rule: ['1'] with '2'
unify_key: '1' with '2'
unify_rule: ['2'] with '2'
unify_key: '2' with '2'

```

```py
(2, [('<digit>', [('1', [])]), ('<integer>', [('<digit>', [('2', [])])])])

```

```py
mystring = "1 + 2"
peg = PEGParser(EXPR_GRAMMAR, log=False)
peg.parse(mystring) 
```

```py
[('<start>',
  [('<expr>',
    [('<term>', [('<factor>', [('<integer>', [('<digit>', [('1', [])])])])]),
     (' + ', []),
     ('<expr>',
      [('<term>',
        [('<factor>', [('<integer>', [('<digit>', [('2', [])])])])])])])])]

```

这两种方法是相互递归的，鉴于 `unify_key()` 尝试每个替代方案直到成功，`unify_key` 可以多次用相同的参数调用。因此，记忆化 `unify_key` 的结果非常重要。Python 提供了一个简单的装饰器 `lru_cache`，用于记忆化任何具有可哈希参数的函数调用。我们将它添加到我们的实现中，以便重复调用 `unify_key()` 与相同的参数时获取缓存的结果。

这种记忆化赋予了算法其名称 – *Packrat*。

```py
from [functools](https://docs.python.org/3/library/functools.html) import lru_cache 
```

```py
class PEGParser(PEGParser):
    @lru_cache(maxsize=None)
    def unify_key(self, key, text, at=0):
        if key not in self.cgrammar:
            if text[at:].startswith(key):
                return at + len(key), (key, [])
            else:
                return at, None
        for rule in self.cgrammar[key]:
            to, res = self.unify_rule(rule, text, at)
            if res is not None:
                return (to, (key, res))
        return 0, None 
```

我们将 `PEGParser` 的初始化和调用封装在 `Parser` 基类中已经实现的 `parse()` 方法中，该方法接受要解析的文本以及语法。</details>

这里有一些我们解析器在行动中的例子。

```py
mystring = "1 + (2 * 3)"
peg = PEGParser(EXPR_GRAMMAR)
for tree in peg.parse(mystring):
    assert tree_to_string(tree) == mystring
    display(display_tree(tree)) 
```

<svg width="210pt" height="575pt" viewBox="0.00 0.00 209.62 575.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 571)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="81.62" y="-553.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="81.62" y="-503.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="33.62" y="-453.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="81.62" y="-453.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="127.62" y="-453.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="31.62" y="-402.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27.62" y="-352.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="20.62" y="-302.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="20.62" y="-252.2" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="127.62" y="-402.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="127.62" y="-352.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="74.62" y="-302.45" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="128.62" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge12" class="edge"><title>10->12</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="182.62" y="-302.45" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge24" class="edge"><title>10->24</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="128.62" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="79.62" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="128.62" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge18" class="edge"><title>13->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="174.62" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge19" class="edge"><title>13->19</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="79.62" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="79.62" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="79.62" y="-51.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="174.62" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="174.62" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="174.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge22" class="edge"><title>21->22</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="174.62" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge23" class="edge"><title>22->23</title></g></g></svg>

```py
mystring = "1 * (2 + 3.35)"
for tree in peg.parse(mystring):
    assert tree_to_string(tree) == mystring
    display(display_tree(tree)) 
```

<svg width="312pt" height="625pt" viewBox="0.00 0.00 312.00 625.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 621.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="81" y="-603.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="81" y="-553.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="81" y="-503.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="30" y="-453.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="81" y="-453.2" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge7" class="edge"><title>2->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="128" y="-453.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge8" class="edge"><title>2->8</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-402.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="21" y="-352.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="21" y="-302.45" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="128" y="-402.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="75" y="-352.7" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="129" y="-352.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge11" class="edge"><title>9->11</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="183" y="-352.7" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge31" class="edge"><title>9->31</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="80" y="-302.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="129" y="-302.45" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge17" class="edge"><title>11->17</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="176" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge18" class="edge"><title>11->18</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="80" y="-252.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="72" y="-201.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="56" y="-151.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="56" y="-101.45" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="177" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge19" class="edge"><title>18->19</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="181" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="122" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="182" y="-151.7" font-family="Times,serif" font-size="14.00">. (46)</text></g> <g id="edge24" class="edge"><title>20->24</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="242" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge25" class="edge"><title>20->25</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="122" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge22" class="edge"><title>21->22</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="122" y="-51.2" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="211" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge26" class="edge"><title>25->26</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="277" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge28" class="edge"><title>25->28</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="211" y="-51.2" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge27" class="edge"><title>26->27</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="277" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="277" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge30" class="edge"><title>29->30</title></g></g></svg>

应该意识到，虽然语法看起来像是一个 *CFG*，但由 *PEG* 描述的语言可能不同。实际上，只有 *LL(1)* 文法才能保证 PEG 和其他解析器表示相同的语言。对于其他类别的文法，PEG 的行为可能会令人惊讶 [[Redziejowski 等人，2008](http://dl.acm.org/citation.cfm?id=2365896.2365924)]。

## 解析上下文无关文法

### PEG 的问题

虽然 *PEG* 在表面上看起来很简单，但它们在某些情况下的行为可能有点不直观。例如，这里有一个例子 [[Redziejowski 等人，2008](http://dl.acm.org/citation.cfm?id=2365896.2365924)]：

```py
PEG_SURPRISE: Grammar = {
    "<A>": ["a<A>a", "aa"]
} 
```

当将其解释为 *CFG* 并用作字符串生成器时，它将产生形式为 `aa, aaaa, aaaaaa` 的字符串，即它产生 `a` 的数量为 $ 2*n $（其中 $ n > 0 $）的字符串。

```py
strings = []
for nn in range(4):
    f = GrammarFuzzer(PEG_SURPRISE, start_symbol='<A>')
    tree = ('<A>', None)
    for _ in range(nn):
        tree = f.expand_tree_once(tree)
    tree = f.expand_tree_with_strategy(tree, f.expand_node_min_cost)
    strings.append(tree_to_string(tree))
    display_tree(tree)
strings 
```

```py
['aa', 'aaaa', 'aaaaaa', 'aaaaaaaa']

```

然而，*PEG* 解析器只能识别形式为 $2^n$ 的字符串

```py
peg = PEGParser(PEG_SURPRISE, start_symbol='<A>')
for s in strings:
    with ExpectError():
        for tree in peg.parse(s):
            display_tree(tree)
        print(s) 
```

```py
aa
aaaa
aaaaaaaa

```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_10601/3226632005.py", line 4, in <module>
    for tree in peg.parse(s):
                ^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_10601/2022555909.py", line 40, in parse
    raise SyntaxError("at " + repr(text[cursor:]))
SyntaxError: at 'aa' (expected)

```

这不是 *解析表达式文法* 的唯一问题。虽然 *PEG* 表达能力强，并且用于解析它们的 *packrat* 解析器简单直观，但 *PEG* 在我们的目的上存在一个主要的缺陷。*PEG* 旨在进行语言识别，并且不清楚如何将任意的 *PEG* 转换为 *CFG*。正如我们之前提到的，将 *PEG* 作为 *CFG* 的天真重新解释并不奏效。此外，不清楚由 *PEG* 表示的语言类与由 *CFG* 表示的语言类之间的确切关系。由于我们的主要重点是 *fuzzing*（即字符串的生成），我们接下来将查看 *可以接受上下文无关文法的解析器*。

*CFG* 解析器的一般思想如下：查看输入文本中允许的字符数，并使用这些字符以及我们的解析器状态来确定哪些规则可以应用于完成解析。接下来，我们将查看一个典型的 *CFG* 解析算法，即 Earley 解析器。

### 耳语法分析器

Earley 解析器是一种通用解析器，能够解析任何任意的 *CFG*。它是由 Jay Earley [[Earley 等人，1970](https://doi.org/10.1145/362007.362035)] 发明的，用于计算语言学。虽然对于具有任意文法的字符串解析，其计算复杂度为 $O(n³)$，但它可以在 $O(n²)$ 时间内解析具有无歧义文法的字符串，并且可以在线性时间（$O(n)$）内解析所有 *[LR(k)](https://en.wikipedia.org/wiki/LR_parser)* 文法 [[Joop M.I.M. Leo，1991](https://doi.org/https://doi.org/10.1016/0304-3975(91)90180-A)]。进一步的改进——特别是处理 ε 规则——是由 Aycock 等人发明的 [John Aycock 等人，2002]。

注意，我们实现的一个限制是，起始符号在其备选表达式中只能有一个备选方案。这在实践中并不是一个限制，因为任何具有多个起始符号备选方案的语法都可以通过添加一个新的起始符号来扩展，该起始符号只选择原始起始符号。也就是说，给定以下语法，

```py
grammar = {
    '<start>': ['<A>', '<B>'],
    ...
}
```

可以将其重写如下以符合 *单备选方案* 规则。

```py
grammar = {
    '<start>': ['<start_>'],
    '<start_>': ['<A>', '<B>'],
    ...
}
```

让我们实现一个名为 `EarleyParser` 的类，它再次从 `Parser` 类派生，而 `Parser` 类实现了 Earley 解析器。

<details id="Excursion:-Implementing-EarleyParser"><summary>实现 `EarleyParser`</summary>

我们首先实现一个更简单的解析器，它几乎适用于所有 *CFGs*，但并不完全。特别是，我们的解析器不理解 *epsilon rules* – 产生空字符串的规则。我们稍后展示如何扩展解析器来处理这些规则。

我们在下面的示例中使用以下语法。

```py
SAMPLE_GRAMMAR: Grammar = {
    '<start>': ['<A><B>'],
    '<A>': ['a<B>c', 'a<A>'],
    '<B>': ['b<C>', '<D>'],
    '<C>': ['c'],
    '<D>': ['d']
}
C_SAMPLE_GRAMMAR = canonical(SAMPLE_GRAMMAR) 
```

```py
syntax_diagram(SAMPLE_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 197.0 62" width="197.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="74.25" y="35">A</text></g> <g class="non-terminal"><text x="122.75" y="35">B</text></g></g></g></g></svg>

```py
A

```

<svg class="railroad-diagram" height="92" viewBox="0 0 245.5 92" width="245.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">a</text></g> <g class="non-terminal"><text x="122.75" y="35">B</text></g> <g class="terminal"><text x="171.25" y="35">c</text></g></g> <g><g class="terminal"><text x="98.5" y="65">a</text></g> <g class="non-terminal"><text x="147.0" y="65">A</text></g></g></g></g></svg>

```py
B

```

<svg class="railroad-diagram" height="92" viewBox="0 0 197.0 92" width="197.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">b</text></g> <g class="non-terminal"><text x="122.75" y="35">C</text></g></g> <g><g class="non-terminal"><text x="98.5" y="65">D</text></g></g></g></g></svg>

```py
C

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">c</text></g></g></g></g></svg>

```py
D

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">d</text></g></g></g></g></svg>

Earley 解析的基本思想如下：

+   从与 START_SYMBOL 对应的备选表达式开始。这些代表从高层次解析字符串的可能方式。本质上，每个表达式代表一个解析路径。将我们字符串的可能解析集的每个表达式排队。一个表达式的解析索引是已经识别的表达式部分。在解析的开始，所有表达式的解析索引都在开始处。此外，每个字母都得到一个表达式队列，该队列识别我们在解析中的该点的字母。

+   检查我们可能的解析队列，看是否有任何以非终结符开始的。如果是这样，那么在解析给定的规则之前，需要从输入中识别出该非终结符。因此，将对应于非终结符的备选表达式添加到队列中。递归地这样做。

+   到这一点，我们已准备好前进。检查输入中的当前字母，并选择所有在解析索引处具有该特定字母的表达式。这些表达式现在可以前进一步。通过增加它们的解析索引并将它们添加到准备识别下一个输入字母的表达式队列中来实现这些选择的表达式。

+   如果在执行这些操作时，我们发现任何表达式已经完成解析，我们就获取其相应的非终结符，并前进所有在其解析索引处具有该非终结符的表达式。

+   递归地继续此过程，直到我们为当前字母排队的所有表达式都已被处理。然后开始处理下一个字母的队列。

我们将在接下来的章节中通过示例详细解释每个步骤。

解析器使用动态规划在每个字母索引处生成一个包含可能解析的 *森林* 的表 – 表中有与输入中字母数量一样多的列，并且每列包含解析的各个阶段的不同的解析规则。

例如，给定输入`adcd`，第 0 列将包含以下内容：

```py
<start> : ● <A> <B>
```

这是起始规则，表示我们目前正在解析规则`<start>`，解析状态是在识别符号`<A>`之前。它还会包含以下内容，这是它可能采取的两个完成解析的路径。

```py
<A> : ● a <B> c
<A> : ● a <A>
```

第 1 列将包含以下内容，这代表了读取`a`后的可能完成情况。

```py
<A> : a ● <B> c
<A> : a ● <A>
<B> : ● b <C>
<B> : ● <D>
<A> : ● a <B> c
<A> : ● a <A>
<D> : ● d
```

在读取`d`之后，第 2 列将包含以下内容

```py
<D> : d ●
<B> : <D> ●
<A> : a <B> ● c
```

类似地，在读取`c`之后，第 3 列将包含以下内容

```py
<A> : a <B> c ●
<start> : <A> ● <B>
<B> : ● b <C>
<B> : ● <D>
<D> : ● d
```

最后，在读取`d`之后，第 4 列将包含以下内容，其中`<start>`规则末尾的`●`表示解析成功。

```py
<D> : d ●
<B> : <D> ●
<start> : <A> <B> ●
```

如上所示，我们实际上是在根据我们读取的每个字母和可以应用的语言规则填充一个表（这个表也称为**图表**）的条目。这个图表给了解析器它的另一个名字——图表解析。

#### 列表

我们首先定义`Column`。`Column`通过其在输入字符串中的`index`和该索引处的`letter`进行初始化。内部上，我们也会随着解析的进行跟踪添加到列中的状态。

```py
class Column:
    def __init__(self, index, letter):
        self.index, self.letter = index, letter
        self.states, self._unique = [], {}

    def __str__(self):
        return "%s chart[%d]\n%s" % (self.letter, self.index, "\n".join(
            str(state) for state in self.states if state.finished())) 
```

`Column`只存储唯一的`states`。因此，当一个新的`state`被添加到我们的`Column`中时，我们会检查它是否已知。

```py
class Column(Column):
    def add(self, state):
        if state in self._unique:
            return self._unique[state]
        self._unique[state] = state
        self.states.append(state)
        state.e_col = self
        return self._unique[state] 
```

#### 项

项表示对特定规则的*解析过程中的解析*。因此，项包含非终端名称和相应的替代表达式（`expr`），它们共同形成规则，以及在这个表达式中的当前解析位置——`dot`。

**注意。**如果你熟悉[LR 解析](https://en.wikipedia.org/wiki/LR_parser)，你会注意到一个项只是一个`LR0`项。

```py
class Item:
    def __init__(self, name, expr, dot):
        self.name, self.expr, self.dot = name, expr, dot 
```

我们还提供了一些便利方法。`finished()`方法检查`dot`是否已经移动到`expr`的最后一个元素之外。`advance()`方法产生一个新的`Item`，其中`dot`向前移动一个标记，表示解析的推进。`at_dot()`方法返回当前正在解析的符号。

```py
class Item(Item):
    def finished(self):
        return self.dot >= len(self.expr)

    def advance(self):
        return Item(self.name, self.expr, self.dot + 1)

    def at_dot(self):
        return self.expr[self.dot] if self.dot < len(self.expr) else None 
```

下面是如何使用项的一个例子。我们首先定义我们的项

```py
item_name = '<B>'
item_expr = C_SAMPLE_GRAMMAR[item_name][1]
an_item = Item(item_name, tuple(item_expr), 0) 
```

为了确定解析的状态，我们使用`at_dot()`

```py
an_item.at_dot() 
```

```py
'<D>'

```

那就是下一个要解析的符号是`<D>`

如果我们推进项，我们会得到另一个表示完成解析规则`<B>`的项。

```py
another_item = an_item.advance() 
```

```py
another_item.finished() 
```

```py
True

```

#### 状态

对于`Earley`解析，解析状态仅仅是与每个状态的起始`s_col`和结束列`e_col`等一些元信息相关的`Item`。因此，我们通过继承`Item`来创建`State`。由于我们感兴趣的是比较状态，我们定义了`hash()`和`eq()`方法。

```py
class State(Item):
    def __init__(self, name, expr, dot, s_col, e_col=None):
        super().__init__(name, expr, dot)
        self.s_col, self.e_col = s_col, e_col

    def __str__(self):
        def idx(var):
            return var.index if var else -1

        return self.name + ':= ' + ' '.join([
            str(p)
            for p in [*self.expr[:self.dot], '|', *self.expr[self.dot:]]
        ]) + "(%d,%d)" % (idx(self.s_col), idx(self.e_col))

    def copy(self):
        return State(self.name, self.expr, self.dot, self.s_col, self.e_col)

    def _t(self):
        return (self.name, self.expr, self.dot, self.s_col.index)

    def __hash__(self):
        return hash(self._t())

    def __eq__(self, other):
        return self._t() == other._t()

    def advance(self):
        return State(self.name, self.expr, self.dot + 1, self.s_col) 
```

`State`的使用与`Item`类似。唯一的区别是它用于与`Column`一起跟踪解析状态。例如，我们如下初始化第一列：

```py
col_0 = Column(0, None)
item_tuple = tuple(*C_SAMPLE_GRAMMAR[START_SYMBOL])
start_state = State(START_SYMBOL, item_tuple, 0, col_0)
col_0.add(start_state)
start_state.at_dot() 
```

```py
'<A>'

```

然后通过使用`Column`的`add()`方法更新第一列。

```py
sym = start_state.at_dot()
for alt in C_SAMPLE_GRAMMAR[sym]:
    col_0.add(State(sym, tuple(alt), 0, col_0))
for s in col_0.states:
    print(s) 
```

```py
<start>:= | <A> <B>(0,0)
<A>:= | a <B> c(0,0)
<A>:= | a <A>(0,0)

```

#### 解析算法

*Earley*算法首先通过将列（与输入中的字母数量一样多）初始化到图表中开始。我们还用表示对应于起始符号的表达式的状态初始化第一列。在我们的例子中，状态对应于起始符号，其中`点`位于`0`，表示如下。`●`符号代表解析状态。在这种情况下，我们还没有解析任何内容。

```py
<start>: ● <A> <B>
```

我们将这个部分图表传递给一个用于填充解析图表其余部分的方法。

在开始解析之前，我们用表示起始符号的当前解析状态初始化图表。

```py
class EarleyParser(Parser):
  """Earley Parser. This parser can parse any context-free grammar."""

    def __init__(self, grammar: Grammar, **kwargs) -> None:
        super().__init__(grammar, **kwargs)
        self.chart: List = []  # for type checking

    def chart_parse(self, words, start):
        alt = tuple(*self.cgrammar[start])
        chart = [Column(i, tok) for i, tok in enumerate([None, *words])]
        chart[0].add(State(start, alt, 0, chart[0]))
        return self.fill_chart(chart) 
```

`fill_chart()`中的主要解析循环有三个基本操作：`predict()`、`scan()`和`complete()`。我们接下来讨论`predict`。

#### 预测状态

我们已经用状态`[<A>,<B>]`和`点`在`0`的位置初始化了`chart[0]`。接下来，鉴于`<A>`是一个非终结符，我们`predict`这个状态的可能的解析延续。也就是说，它可以是`a <B> c`或`A <A>`。

`predict()`的一般思想如下：假设你有一个名为`<A>`的状态，来自上面的语法，包含表达式`[a,<B>,c]`。想象一下你已经看到了`a`，这意味着`点`将位于`<B>`上。下面是我们解析状态的一个表示。●的左侧代表已经解析的部分（`a`），右侧代表尚未解析的部分（`<B> c`）。

```py
<A>: a  ●  <B> c
```

要识别`<B>`，我们查看`<B>`的定义，它有不同的可选表达式。`predict()`步骤将每个这些可选表达式添加到状态的集合中，`点`位于`0`。

```py
<A>: a ● <B> c
<B>: ● b c
<B>: ● <D>
```

从本质上讲，当用当前非终结符调用`predict()`方法时，它会获取与这个非终结符对应的可选表达式，并将这些作为预测的*子状态*添加到*当前*列。

```py
class EarleyParser(EarleyParser):
    def predict(self, col, sym, state):
        for alt in self.cgrammar[sym]:
            col.add(State(sym, tuple(alt), 0, col)) 
```

要了解如何使用`predict`，我们首先像之前一样构建第 0 列，并将构建的列分配给一个`EarleyParser`的实例。

```py
col_0 = Column(0, None)
col_0.add(start_state)
ep = EarleyParser(SAMPLE_GRAMMAR)
ep.chart = [col_0] 
```

它应该包含一个单一的状态——`<start> at 0`。

```py
for s in ep.chart[0].states:
    print(s) 
```

```py
<start>:= | <A> <B>(0,0)

```

我们应用`predict()`来填充第 0 列，该列应包含可能的解析路径。

```py
ep.predict(col_0, '<A>', s)
for s in ep.chart[0].states:
    print(s) 
```

```py
<start>:= | <A> <B>(0,0)
<A>:= | a <B> c(0,0)
<A>:= | a <A>(0,0)

```

#### 扫描标记

如果状态中包含的不是非终结符，而是一个终结符，比如一个字母呢？在这种情况下，我们就可以取得一些进展了。例如，考虑第二个状态：

```py
<B>: ● b c
```

我们扫描下一列的字母。假设下一个标记是`b`。如果字母与我们已有的匹配，则通过将当前状态向前移动一个字母来创建一个新的状态。

```py
<B>: b ● c
```

这个新状态被添加到下一列（即匹配字母所在的列）。

```py
class EarleyParser(EarleyParser):
    def scan(self, col, state, letter):
        if letter == col.letter:
            col.add(state.advance()) 
```

如前所述，我们首先构建部分解析，这次添加一个新列，这样我们就可以观察`scan()`的效果。

```py
ep = EarleyParser(SAMPLE_GRAMMAR)
col_1 = Column(1, 'a')
ep.chart = [col_0, col_1] 
```

```py
new_state = ep.chart[0].states[1]
print(new_state) 
```

```py
<A>:= | a <B> c(0,0)

```

```py
ep.scan(col_1, new_state, 'a')
for s in ep.chart[1].states:
    print(s) 
```

```py
<A>:= a | <B> c(0,1)

```

#### 完成处理

当我们前进时，如果我们实际上`complete()`当前规则的解析处理怎么办？如果是这样，我们不仅想更新这个状态，还想更新所有*父状态*，这些状态是从这个状态派生出来的。例如，假设我们有以下状态。

```py
<A>: a ● <B> c
<B>: b c ●
```

状态`<B>: b c ●`现在已完成。因此，我们需要将`<A>: a ● <B> c`向前推进一步。

我们如何确定父状态？注意从`predict`中，我们将预测的子状态添加到与检查状态相同的列。因此，我们查看当前状态的起始列，具有与完成状态名称相同的符号`at_dot`。

对于找到的每个此类父状态，我们前进该父状态（因为我们刚刚完成了它们的`at_dot`的非终结符解析）并将新状态添加到当前列。

```py
class EarleyParser(EarleyParser):
    def complete(self, col, state):
        return self.earley_complete(col, state)

    def earley_complete(self, col, state):
        parent_states = [
            st for st in state.s_col.states if st.at_dot() == state.name
        ]
        for st in parent_states:
            col.add(st.advance()) 
```

下面是一个完成处理的例子。首先我们完成第 0 列。

```py
ep = EarleyParser(SAMPLE_GRAMMAR)
col_1 = Column(1, 'a')
col_2 = Column(2, 'd')
ep.chart = [col_0, col_1, col_2]
ep.predict(col_0, '<A>', s)
for s in ep.chart[0].states:
    print(s) 
```

```py
<start>:= | <A> <B>(0,0)
<A>:= | a <B> c(0,0)
<A>:= | a <A>(0,0)

```

然后我们使用`scan()`来填充第 1 列。

```py
for state in ep.chart[0].states:
    if state.at_dot() not in SAMPLE_GRAMMAR:
        ep.scan(col_1, state, 'a')
for s in ep.chart[1].states:
    print(s) 
```

```py
<A>:= a | <B> c(0,1)
<A>:= a | <A>(0,1)

```

```py
for state in ep.chart[1].states:
    if state.at_dot() in SAMPLE_GRAMMAR:
        ep.predict(col_1, state.at_dot(), state)
for s in ep.chart[1].states:
    print(s) 
```

```py
<A>:= a | <B> c(0,1)
<A>:= a | <A>(0,1)
<B>:= | b <C>(1,1)
<B>:= | <D>(1,1)
<A>:= | a <B> c(1,1)
<A>:= | a <A>(1,1)
<D>:= | d(1,1)

```

然后我们再次使用`scan()`来填充第 2 列。

```py
for state in ep.chart[1].states:
    if state.at_dot() not in SAMPLE_GRAMMAR:
        ep.scan(col_2, state, state.at_dot())

for s in ep.chart[2].states:
    print(s) 
```

```py
<D>:= d |(1,2)

```

现在，我们可以使用`complete()`：

```py
for state in ep.chart[2].states:
    if state.finished():
        ep.complete(col_2, state)

for s in ep.chart[2].states:
    print(s) 
```

```py
<D>:= d |(1,2)
<B>:= <D> |(1,2)
<A>:= a <B> | c(0,2)

```

#### 填充图表

`fill_chart()`中的主要驱动循环基本上按顺序调用这些操作。我们按顺序遍历每一列。

+   对于每一列，一次获取列中的一个状态，并检查该状态是否为`finished`。

    +   如果是，那么我们`complete()`依赖于此状态的所有父状态。

+   如果状态未完成，我们检查该状态当前符号`at_dot`是否为非终结符。

    +   如果它是非终结符，我们`predict()`可能的延续，并使用这些状态更新当前列。

    +   如果不是，我们`scan()`下一个列，并在它匹配下一个字母时前进当前状态。

```py
class EarleyParser(EarleyParser):
    def fill_chart(self, chart):
        for i, col in enumerate(chart):
            for state in col.states:
                if state.finished():
                    self.complete(col, state)
                else:
                    sym = state.at_dot()
                    if sym in self.cgrammar:
                        self.predict(col, sym, state)
                    else:
                        if i + 1 >= len(chart):
                            continue
                        self.scan(chart[i + 1], state, sym)
            if self.log:
                print(col, '\n')
        return chart 
```

现在我们可以识别一个给定的字符串属于由语法表示的语言。

```py
ep = EarleyParser(SAMPLE_GRAMMAR, log=True)
columns = ep.chart_parse('adcd', START_SYMBOL) 
```

```py
None chart[0]

a chart[1]

d chart[2]
<D>:= d |(1,2)
<B>:= <D> |(1,2) 

c chart[3]
<A>:= a <B> c |(0,3) 

d chart[4]
<D>:= d |(3,4)
<B>:= <D> |(3,4)
<start>:= <A> <B> |(0,4) 

```

我们打印的图表只显示了每个索引处的完成条目。括号内的表达式表示识别第一个字符之前的列，以及结束列。

注意`<start>`非终结符如何显示完全解析的状态。

```py
last_col = columns[-1]
for state in last_col.states:
    if state.name == '<start>':
        print(state) 
```

```py
<start>:= <A> <B> |(0,4)

```

由于`chart_parse()`返回完成表，我们现在需要提取推导树。

#### 解析方法

为了确定我们已经解析多远，我们只需查找`chart_parse()`中`start_symbol`被找到的最后一个索引。

```py
class EarleyParser(EarleyParser):
    def parse_prefix(self, text):
        self.table = self.chart_parse(text, self.start_symbol())
        for col in reversed(self.table):
            states = [
                st for st in col.states if st.name == self.start_symbol()
            ]
            if states:
                return col.index, states
        return -1, [] 
```

下面是`parse_prefix()`的实际应用。

```py
ep = EarleyParser(SAMPLE_GRAMMAR)
cursor, last_states = ep.parse_prefix('adcd')
print(cursor, [str(s) for s in last_states]) 
```

```py
4 ['<start>:= <A> <B> |(0,4)']

```

下面的内容改编自关于 Earley 解析的优秀参考资料[Loup Vaillant](http://loup-vaillant.fr/tutorials/earley-parsing/)。

我们的`parse()`方法如下。它依赖于两个将在下面定义的方法`parse_forest()`和`extract_trees()`。

```py
class EarleyParser(EarleyParser):
    def parse(self, text):
        cursor, states = self.parse_prefix(text)
        start = next((s for s in states if s.finished()), None)

        if cursor < len(text) or not start:
            raise SyntaxError("at " + repr(text[cursor:]))

        forest = self.parse_forest(self.table, start)
        for tree in self.extract_trees(forest):
            yield self.prune_tree(tree) 
```

#### 解析路径

`parse_paths()`方法尝试将`named_expr`中给出的表达式与解析的字符串统一。为此，它提取`named_expr`中的最后一个符号并检查它是否是终结符号。如果是，那么它检查`til`处的图表以查看对应位置的字母是否与终结符号匹配。如果匹配，则将我们的起始索引扩展到符号的长度。

如果符号是非终结符，那么我们从当前末尾列索引（`til`）检索与该非终结符对应的已解析状态，并收集起始索引。这些是剩余表达式的末尾列索引。

给定我们的起始索引列表，我们从剩余的表达式中获得解析路径。如果我们能获得任何，则返回解析路径。如果不能，则返回一个空列表。

```py
class EarleyParser(EarleyParser):
    def parse_paths(self, named_expr, chart, frm, til):
        def paths(state, start, k, e):
            if not e:
                return [[(state, k)]] if start == frm else []
            else:
                return [[(state, k)] + r
                        for r in self.parse_paths(e, chart, frm, start)]

        *expr, var = named_expr
        starts = None
        if var not in self.cgrammar:
            starts = ([(var, til - len(var),
                        't')] if til > 0 and chart[til].letter == var else [])
        else:
            starts = [(s, s.s_col.index, 'n') for s in chart[til].states
                      if s.finished() and s.name == var]

        return [p for s, start, k in starts for p in paths(s, start, k, expr)] 
```

下面是`parse_paths()`的实际应用

```py
print(SAMPLE_GRAMMAR['<start>'])
ep = EarleyParser(SAMPLE_GRAMMAR)
completed_start = last_states[0]
paths = ep.parse_paths(completed_start.expr, columns, 0, 4)
for path in paths:
    print([list(str(s_) for s_ in s) for s in path]) 
```

```py
['<A><B>']
[['<B>:= <D> |(3,4)', 'n'], ['<A>:= a <B> c |(0,3)', 'n']]

```

也就是说，给定输入`adcd`，对于`<start>`的解析路径包括识别表达式`<A><B>`。这通过两个状态识别：从输入(0)到输入(2)的`<A>`状态，它进一步涉及到识别规则`a<B>c`，以及下一个状态`<B>`，它涉及到识别规则`<D>`。

#### 解析森林

`parse_forest()`方法接受表示完成解析的状态，并确定其表达式对应解析表达式的可能方式。例如，假设我们正在解析`1+2+3`，状态在`expr`中有`[<expr>,+,<expr>]`。它可以解析为`[{<expr>:1+2},+,{<expr>:3}]`或`[{<expr>:1},+,{<expr>:2+3}]`。

```py
class EarleyParser(EarleyParser):
    def forest(self, s, kind, chart):
        return self.parse_forest(chart, s) if kind == 'n' else (s, [])

    def parse_forest(self, chart, state):
        pathexprs = self.parse_paths(state.expr, chart, state.s_col.index,
                                     state.e_col.index) if state.expr else []
        return state.name, [[(v, k, chart) for v, k in reversed(pathexpr)]
                            for pathexpr in pathexprs] 
```

```py
ep = EarleyParser(SAMPLE_GRAMMAR)
result = ep.parse_forest(columns, last_states[0])
result 
```

```py
('<start>',
 [[(<__main__.State at 0x1071c37d0>,
    'n',
    [<__main__.Column at 0x1071c2810>,
     <__main__.Column at 0x1071c27e0>,
     <__main__.Column at 0x1071c29f0>,
     <__main__.Column at 0x1071c1be0>,
     <__main__.Column at 0x1071c30e0>]),
   (<__main__.State at 0x1071c3980>,
    'n',
    [<__main__.Column at 0x1071c2810>,
     <__main__.Column at 0x1071c27e0>,
     <__main__.Column at 0x1071c29f0>,
     <__main__.Column at 0x1071c1be0>,
     <__main__.Column at 0x1071c30e0>])]])

```

#### 提取树

从`parse_forest()`我们得到一棵树森林。我们需要从那个森林中提取一棵树。这是通过以下方式完成的。

（目前，我们返回第一个可用的推导树。为了做到这一点，我们需要从对应于`start`的状态中提取解析森林。）

```py
class EarleyParser(EarleyParser):
    def extract_a_tree(self, forest_node):
        name, paths = forest_node
        if not paths:
            return (name, [])
        return (name, [self.extract_a_tree(self.forest(*p)) for p in paths[0]])

    def extract_trees(self, forest):
        yield self.extract_a_tree(forest) 
```

我们现在验证我们的解析器是否可以解析给定的表达式。

```py
A3_GRAMMAR: Grammar = {
    "<start>": ["<bexpr>"],
    "<bexpr>": [
        "<aexpr><gt><aexpr>", "<aexpr><lt><aexpr>", "<aexpr>=<aexpr>",
        "<bexpr>=<bexpr>", "<bexpr>&<bexpr>", "<bexpr>|<bexpr>", "(<bexrp>)"
    ],
    "<aexpr>":
    ["<aexpr>+<aexpr>", "<aexpr>-<aexpr>", "(<aexpr>)", "<integer>"],
    "<integer>": ["<digit><integer>", "<digit>"],
    "<digit>": ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"],
    "<lt>": ['<'],
    "<gt>": ['>']
} 
```

```py
syntax_diagram(A3_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 182.5 62" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">bexpr</text></g></g></g></g></svg>

```py
bexpr

```

<svg class="railroad-diagram" height="80" viewBox="0 0 1837.5 80" width="1837.5"><g transform="translate(.5 .5)"><g><g><g><g class="non-terminal"><text x="101.25" y="44">aexpr</text></g> <g class="non-terminal"><text x="171.0" y="44">gt</text></g> <g class="non-terminal"><text x="240.75" y="44">aexpr</text></g></g></g> <g><g><g class="non-terminal"><text x="363.25" y="44">aexpr</text></g> <g class="non-terminal"><text x="433.0" y="44">lt</text></g> <g class="non-terminal"><text x="502.75" y="44">aexpr</text></g></g></g> <g><g><g class="non-terminal"><text x="625.25" y="44">aexpr</text></g> <g class="terminal"><text x="690.75" y="44">=</text></g> <g class="non-terminal"><text x="756.25" y="44">aexpr</text></g></g></g> <g><g><g class="non-terminal"><text x="878.75" y="44">bexpr</text></g> <g class="terminal"><text x="944.25" y="44">=</text></g> <g class="non-terminal"><text x="1009.75" y="44">bexpr</text></g></g></g> <g><g><g class="non-terminal"><text x="1132.25" y="44">bexpr</text></g> <g class="terminal"><text x="1197.75" y="44">&</text></g> <g class="non-terminal"><text x="1263.25" y="44">bexpr</text></g></g></g> <g><g><g class="non-terminal"><text x="1385.75" y="44">bexpr</text></g> <g class="terminal"><text x="1451.25" y="44">|</text></g> <g class="non-terminal"><text x="1516.75" y="44">bexpr</text></g></g></g> <g><g><g class="terminal"><text x="1622.25" y="44">(</text></g> <g class="non-terminal"><text x="1687.75" y="44">bexrp</text></g> <g class="terminal"><text x="1753.25" y="44">)</text></g></g></g></g></g></svg>

```py
aexpr

```

<svg class="railroad-diagram" height="152" viewBox="0 0 313.5 152" width="313.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="65">aexpr</text></g> <g class="terminal"><text x="156.75" y="65">-</text></g> <g class="non-terminal"><text x="222.25" y="65">aexpr</text></g></g> <g><g class="non-terminal"><text x="91.25" y="35">aexpr</text></g> <g class="terminal"><text x="156.75" y="35">+</text></g> <g class="non-terminal"><text x="222.25" y="35">aexpr</text></g></g> <g><g class="terminal"><text x="91.25" y="95">(</text></g> <g class="non-terminal"><text x="156.75" y="95">aexpr</text></g> <g class="terminal"><text x="222.25" y="95">)</text></g></g> <g><g class="non-terminal"><text x="156.75" y="125">integer</text></g></g></g></g></svg>

```py
integer

```

<svg class="railroad-diagram" height="92" viewBox="0 0 282.0 92" width="282.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">digit</text></g> <g class="non-terminal"><text x="182.25" y="35">integer</text></g></g> <g><g class="non-terminal"><text x="141.0" y="65">digit</text></g></g></g></g></svg>

```py
digit

```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

```py
lt

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35"><</text></g></g></g></g></svg>

```py
gt

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">></text></g></g></g></g></svg>

```py
mystring = '(1+24)=33'
parser = EarleyParser(A3_GRAMMAR)
for tree in parser.parse(mystring):
    assert tree_to_string(tree) == mystring
    display_tree(tree) 
```

现在我们有一个完整的解析器，可以解析几乎任意的*CFG*。剩下一个小问题需要修复——即后面我们将看到的 epsilon 规则的情况。

#### 含糊解析

含糊的语法是可以为某些给定字符串产生多个推导树的语法。例如，`A3_GRAMMAR`可以以两种不同的方式解析`1+2+3`——`[1+2]+3`和`1+[2+3]`。

对于无歧义解析，提取一棵树可能是合理的。但是，如果给定的语法在给定字符串时产生歧义怎么办？在这种情况下，我们需要提取所有推导树。我们增强`extract_trees()`方法以提取多个推导树。

```py
import [itertools](https://docs.python.org/3/library/itertools.html) as I 
```

```py
class EarleyParser(EarleyParser):
    def extract_trees(self, forest_node):
        name, paths = forest_node
        if not paths:
            yield (name, [])

        for path in paths:
            ptrees = [self.extract_trees(self.forest(*p)) for p in path]
            for p in I.product(*ptrees):
                yield (name, p) 
```

如前所述，我们验证一切是否正常工作。

```py
mystring = '1+2'
parser = EarleyParser(A1_GRAMMAR)
for tree in parser.parse(mystring):
    assert mystring == tree_to_string(tree)
    display_tree(tree) 
```

也可以使用`GrammarFuzzer`来验证一切是否正常工作。

```py
gf = GrammarFuzzer(A1_GRAMMAR)
for i in range(5):
    s = gf.fuzz()
    print(i, s)
    for tree in parser.parse(s):
        assert tree_to_string(tree) == s 
```

```py
0 045+3+2-9+7-7-5-1-449
1 0+9+5-2+1-8+4-3+7+2
2 76413
3 9339
4 62

```

#### Aycock Epsilon 修复

在解析过程中，人们经常需要知道一个给定的非终结符是否可以推导出空字符串。例如，在下面的语法中，A 可以推导出空字符串，而 B 则不能。可以推导出空字符串的非终结符被称为*可空*非终结符。例如，在下面的语法`E_GRAMMAR_1`中，`<A>`是*可空*的，由于`<A>`是`<start>`的其中一个选择，因此`<start>`也是*可空*的。但`<B>`不是*可空*的。

```py
E_GRAMMAR_1: Grammar = {
    '<start>': ['<A>', '<B>'],
    '<A>': ['a', ''],
    '<B>': ['b']
} 
```

原始 Earley 实现的一个问题是它处理可以推导出空字符串的规则不好。例如，给定的语法应该匹配`a`

```py
EPSILON = ''
E_GRAMMAR: Grammar = {
    '<start>': ['<S>'],
    '<S>': ['<A><A><A><A>'],
    '<A>': ['a', '<E>'],
    '<E>': [EPSILON]
} 
```

```py
syntax_diagram(E_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="74.25" y="35">S</text></g></g></g></g></svg>

```py
S

```

<svg class="railroad-diagram" height="62" viewBox="0 0 294.0 62" width="294.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="74.25" y="35">A</text></g> <g class="non-terminal"><text x="122.75" y="35">A</text></g> <g class="non-terminal"><text x="171.25" y="35">A</text></g> <g class="non-terminal"><text x="219.75" y="35">A</text></g></g></g></g></svg>

```py
A

```

<svg class="railroad-diagram" height="92" viewBox="0 0 148.5 92" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">a</text></g></g> <g><g class="non-terminal"><text x="74.25" y="65">E</text></g></g></g></g></svg>

```py
E

```

```py
mystring = 'a'
parser = EarleyParser(E_GRAMMAR)
with ExpectError():
    trees = parser.parse(mystring) 
```

Aycock 等人[John Aycock *et al*, 2002]提出了一种简单的修复方法。他们的想法是预先计算`nullable`集合，并使用它来推进`nullable`状态。然而，在我们这样做之前，我们需要计算`nullable`集合。`nullable`集合包括所有可以推导出空字符串的非终结符。

计算可空集需要迭代语法中的每个产生式规则，并检查给定的规则是否可以推导出空字符串。每次迭代都需要考虑新发现的 `nullable` 终端。当获得稳定的结果时，该过程停止。这个过程可以抽象为一个更通用的方法 `fixpoint`。

##### Fixpoint

函数的 `fixpoint` 是函数域中的一个元素，该元素映射到自身。例如，1 是平方根的 `fixpoint`，因为 `squareroot(1) == 1`。

（我们使用 `str` 而不是 `hash` 来检查 `fixpoint` 中的等价性，因为作为参数我们希望使用的 `set` 数据结构有一个良好的字符串表示，但不是可哈希的）。

```py
def fixpoint(f):
    def helper(arg):
        while True:
            sarg = str(arg)
            arg_ = f(arg)
            if str(arg_) == sarg:
                return arg
            arg = arg_

    return helper 
```

记得 第一章 中的 `my_sqrt()` 吗？我们可以使用 `fixpoint` 来定义 `my_sqrt()`。

```py
def my_sqrt(x):
    @fixpoint
    def _my_sqrt(approx):
        return (approx + x / approx) / 2

    return _my_sqrt(1) 
```

```py
my_sqrt(2) 
```

```py
1.414213562373095

```

##### Nullable

同样，我们可以使用 `fixpoint` 来定义 `nullable`。我们本质上提供了单个中间步骤的定义。也就是说，假设 `nullables` 包含当前的 `nullable` 非终结符，我们遍历语法，寻找 `nullable` 的产生式——即，整个序列在某些扩展中可以产生空字符串的产生式。

我们需要遍历不同的替代表达式及其对应非终结符。因此，我们定义了一个 `rules()` 方法，将我们的字典表示转换为这种对格式。

```py
def rules(grammar):
    return [(key, choice)
            for key, choices in grammar.items()
            for choice in choices] 
```

`terminals()` 方法从 `canonical` 语法表示中提取所有终端符号。

```py
def terminals(grammar):
    return set(token
               for key, choice in rules(grammar)
               for token in choice if token not in grammar) 
```

```py
def nullable_expr(expr, nullables):
    return all(token in nullables for token in expr) 
```

```py
def nullable(grammar):
    productions = rules(grammar)

    @fixpoint
    def nullable_(nullables):
        for A, expr in productions:
            if nullable_expr(expr, nullables):
                nullables |= {A}
        return (nullables)

    return nullable_({EPSILON}) 
```

```py
for key, grammar in {
        'E_GRAMMAR': E_GRAMMAR,
        'E_GRAMMAR_1': E_GRAMMAR_1
}.items():
    print(key, nullable(canonical(grammar))) 
```

```py
E_GRAMMAR {'', '<start>', '<S>', '<E>', '<A>'}
E_GRAMMAR_1 {'', '<start>', '<A>'}

```

因此，一旦我们有了 `nullable` 集合，我们只需要在调用与一个非终结符对应的状态的 `predict` 之后检查它是否是 `nullable`，如果是，就前进并将状态添加到当前列。

```py
class EarleyParser(EarleyParser):
    def __init__(self, grammar, **kwargs):
        super().__init__(grammar, **kwargs)
        self.epsilon = nullable(self.cgrammar)

    def predict(self, col, sym, state):
        for alt in self.cgrammar[sym]:
            col.add(State(sym, tuple(alt), 0, col))
        if sym in self.epsilon:
            col.add(state.advance()) 
```

```py
mystring = 'a'
parser = EarleyParser(E_GRAMMAR)
for tree in parser.parse(mystring):
    display_tree(tree) 
```

为了确保我们的解析器可以解析所有类型的语法，让我们尝试两个更多的测试用例。

```py
DIRECTLY_SELF_REFERRING: Grammar = {
    '<start>': ['<query>'],
    '<query>': ['select <expr> from a'],
    "<expr>": ["<expr>", "a"],
}
INDIRECTLY_SELF_REFERRING: Grammar = {
    '<start>': ['<query>'],
    '<query>': ['select <expr> from a'],
    "<expr>": ["<aexpr>", "a"],
    "<aexpr>": ["<expr>"],
} 
```

```py
mystring = 'select a from a'
for grammar in [DIRECTLY_SELF_REFERRING, INDIRECTLY_SELF_REFERRING]:
    forest = EarleyParser(grammar).parse(mystring)
    print('recognized', mystring)
    try:
        for tree in forest:
            print(tree_to_string(tree))
    except RecursionError as e:
        print("Recursion error", e) 
```

```py
recognized select a from a
Recursion error maximum recursion depth exceeded
recognized select a from a
Recursion error maximum recursion depth exceeded

```

为什么这里会出现递归错误？原因是，我们的 `extract_trees()` 实现是急切的。也就是说，在它能够构建外层解析树之前，它会尝试提取 *所有* 内部解析树。当存在自引用时，这会导致递归。这里有一个简单的提取器可以避免这个问题。这里的想法是随机和懒惰地选择一个节点来扩展，这样可以避免无限递归。

#### Tree Extractor

正如你上面看到的，尝试提取所有树的其中一个问题是解析森林可以由无限数量的树组成。因此，我们通过一次提取一棵树来解决这个问题。

```py
class SimpleExtractor:
    def __init__(self, parser, text):
        self.parser = parser
        cursor, states = parser.parse_prefix(text)
        start = next((s for s in states if s.finished()), None)
        if cursor < len(text) or not start:
            raise SyntaxError("at " + repr(cursor))
        self.my_forest = parser.parse_forest(parser.table, start)

    def extract_a_node(self, forest_node):
        name, paths = forest_node
        if not paths:
            return ((name, 0, 1), []), (name, [])
        cur_path, i, length = self.choose_path(paths)
        child_nodes = []
        pos_nodes = []
        for s, kind, chart in cur_path:
            f = self.parser.forest(s, kind, chart)
            postree, ntree = self.extract_a_node(f)
            child_nodes.append(ntree)
            pos_nodes.append(postree)

        return ((name, i, length), pos_nodes), (name, child_nodes)

    def choose_path(self, arr):
        length = len(arr)
        i = random.randrange(length)
        return arr[i], i, length

    def extract_a_tree(self):
        pos_tree, parse_tree = self.extract_a_node(self.my_forest)
        return self.parser.prune_tree(parse_tree) 
```

使用方法如下：

```py
de = SimpleExtractor(EarleyParser(DIRECTLY_SELF_REFERRING), mystring) 
```

```py
for i in range(5):
    tree = de.extract_a_tree()
    print(tree_to_string(tree)) 
```

```py
select a from a
select a from a
select a from a
select a from a
select a from a

```

关于间接引用：

```py
ie = SimpleExtractor(EarleyParser(INDIRECTLY_SELF_REFERRING), mystring) 
```

```py
for i in range(5):
    tree = ie.extract_a_tree()
    print(tree_to_string(tree)) 
```

```py
select a from a
select a from a
select a from a
select a from a
select a from a

```

注意，`SimpleExtractor` 并不保证返回的树的唯一性。然而，可以通过跟踪从 `pos_tree` 变量扩展出的特定节点来解决这个问题，从而避免探索相同的路径。

为了实现这一点，我们提取传递到 `SimpleExtractor` 的随机流，并使用它来控制哪些节点被探索。不同的探索路径可以形成一个节点树。

我们从单个选择节点的定义开始。`self._chosen` 表示当前做出的选择，`self.next` 存储使用 `self._chosen` 完成的下一个选择。`self.total` 存储在这个节点中可以拥有的总选择数。

```py
class ChoiceNode:
    def __init__(self, parent, total):
        self._p, self._chosen = parent, 0
        self._total, self.next = total, None

    def chosen(self):
        assert not self.finished()
        return self._chosen

    def __str__(self):
        return '%d(%s/%s  %s)' % (self._i, str(self._chosen),
                                 str(self._total), str(self.next))

    def __repr__(self):
        return repr((self._i, self._chosen, self._total))

    def increment(self):
        # as soon as we increment, next becomes invalid
        self.next = None
        self._chosen += 1
        if self.finished():
            if self._p is None:
                return None
            return self._p.increment()
        return self

    def finished(self):
        return self._chosen >= self._total 
```

现在我们来到增强的 `EnhancedExtractor()`。

```py
class EnhancedExtractor(SimpleExtractor):
    def __init__(self, parser, text):
        super().__init__(parser, text)
        self.choices = ChoiceNode(None, 1) 
```

首先，我们定义 `choose_path()`，它给定一个数组和选择节点，如果存在，返回对应于下一个选择节点的数组元素，或者产生一个新的选择节点，并返回该元素。

```py
class EnhancedExtractor(EnhancedExtractor):
    def choose_path(self, arr, choices):
        arr_len = len(arr)
        if choices.next is not None:
            if choices.next.finished():
                return None, None, None, choices.next
        else:
            choices.next = ChoiceNode(choices, arr_len)
        next_choice = choices.next.chosen()
        choices = choices.next
        return arr[next_choice], next_choice, arr_len, choices 
```

我们在这里定义 `extract_a_node()`。在提取过程中，我们有一个选择：是允许无限森林，还是应该有一个没有直接递归的有限树的数量？直接递归是指存在一个具有相同非终端的父节点，它解析了相同的范围。在这里，我们选择不提取这样的树。它们可以在解析后添加回来。

这是一个递归过程，它检查一个节点，提取完成该节点所需的路径。单个路径（对应于非终端）可能再次由一系列较小的路径组成。这些路径再次使用对 `extract_a_node()` 的递归调用提取。

当我们遇到我们想要避免的节点递归之一时会发生什么？在这种情况下，我们返回当前的选择节点，它会冒泡到 `extract_a_tree()`。该过程增加最后一个选择，然后依次增加父节点，直到我们达到一个仍有探索选项的选择节点。

如果我们遇到特定选择节点的选择结束（即，我们已经从节点中耗尽了可以采取的路径）怎么办？在这种情况下，我们也返回当前的选择节点，它会冒泡到 `extract_a_tree()`。该过程增加最后一个选择，然后冒泡到下一个具有未探索路径的选择。

```py
class EnhancedExtractor(EnhancedExtractor):
    def extract_a_node(self, forest_node, seen, choices):
        name, paths = forest_node
        if not paths:
            return (name, []), choices

        cur_path, _i, _l, new_choices = self.choose_path(paths, choices)
        if cur_path is None:
            return None, new_choices
        child_nodes = []
        for s, kind, chart in cur_path:
            if kind == 't':
                child_nodes.append((s, []))
                continue
            nid = (s.name, s.s_col.index, s.e_col.index)
            if nid in seen:
                return None, new_choices
            f = self.parser.forest(s, kind, chart)
            ntree, newer_choices = self.extract_a_node(f, seen | {nid}, new_choices)
            if ntree is None:
                return None, newer_choices
            child_nodes.append(ntree)
            new_choices = newer_choices
        return (name, child_nodes), new_choices 
```

`extract_a_tree()` 是单个树的深度优先提取器。它试图提取一个树，如果提取返回 `None`，则表示特定的选择已经耗尽，或者我们遇到了递归。在这种情况下，我们增加选择，并探索新的路径。

```py
class EnhancedExtractor(EnhancedExtractor):
    def extract_a_tree(self):
        while not self.choices.finished():
            parse_tree, choices = self.extract_a_node(self.my_forest, set(), self.choices)
            choices.increment()
            if parse_tree is not None:
                return self.parser.prune_tree(parse_tree)
        return None 
```

注意，`EnhancedExtractor` 只提取非直接递归的节点。也就是说，如果它找到一个具有与具有相同非终端的父节点覆盖相同范围的非终端的节点，它就会跳过该节点。

```py
ee = EnhancedExtractor(EarleyParser(INDIRECTLY_SELF_REFERRING), mystring) 
```

```py
i = 0
while True:
    i += 1
    t = ee.extract_a_tree()
    if t is None: break
    print(i, t)
    s = tree_to_string(t)
    assert s == mystring 
```

```py
1 ('<start>', [('<query>', [('select ', []), ('<expr>', [('a', [])]), (' from a', [])])])

```

```py
istring = '1+2+3+4'
ee = EnhancedExtractor(EarleyParser(A1_GRAMMAR), istring) 
```

```py
i = 0
while True:
    i += 1
    t = ee.extract_a_tree()
    if t is None: break
    print(i, t)
    s = tree_to_string(t)
    assert s == istring 
```

```py
1 ('<start>', [('<expr>', [('<expr>', [('<expr>', [('<expr>', [('<integer>', [('<digit>', [('1', [])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('2', [])])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('3', [])])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('4', [])])])])])])
2 ('<start>', [('<expr>', [('<expr>', [('<expr>', [('<integer>', [('<digit>', [('1', [])])])]), ('+', []), ('<expr>', [('<expr>', [('<integer>', [('<digit>', [('2', [])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('3', [])])])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('4', [])])])])])])
3 ('<start>', [('<expr>', [('<expr>', [('<expr>', [('<integer>', [('<digit>', [('1', [])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('2', [])])])])]), ('+', []), ('<expr>', [('<expr>', [('<integer>', [('<digit>', [('3', [])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('4', [])])])])])])])
4 ('<start>', [('<expr>', [('<expr>', [('<integer>', [('<digit>', [('1', [])])])]), ('+', []), ('<expr>', [('<expr>', [('<expr>', [('<integer>', [('<digit>', [('2', [])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('3', [])])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('4', [])])])])])])])
5 ('<start>', [('<expr>', [('<expr>', [('<integer>', [('<digit>', [('1', [])])])]), ('+', []), ('<expr>', [('<expr>', [('<integer>', [('<digit>', [('2', [])])])]), ('+', []), ('<expr>', [('<expr>', [('<integer>', [('<digit>', [('3', [])])])]), ('+', []), ('<expr>', [('<integer>', [('<digit>', [('4', [])])])])])])])])

```

#### 更多 Earley 解析

对于 Earley 解析器，存在许多其他优化。一个快速的工业强度 Earley 解析器实现是 [Marpa 解析器](https://jeffreykegler.github.io/Marpa-web-site/)。此外，Earley 解析不必限于字符数据。也可以使用广义 Earley 解析器解析流（音频和视频流）[Qi 等人，2018]。</details>

这里有一些 Earley 解析器在行动中的示例。

```py
mystring = "1 + (2 * 3)"
earley = EarleyParser(EXPR_GRAMMAR)
for tree in earley.parse(mystring):
    assert tree_to_string(tree) == mystring
    display(display_tree(tree)) 
```

<svg width="210pt" height="575pt" viewBox="0.00 0.00 209.62 575.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 571)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="81.62" y="-553.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="81.62" y="-503.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="33.62" y="-453.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="81.62" y="-453.2" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="127.62" y="-453.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="31.62" y="-402.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27.62" y="-352.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="20.62" y="-302.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="20.62" y="-252.2" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="127.62" y="-402.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="127.62" y="-352.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="74.62" y="-302.45" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="128.62" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge12" class="edge"><title>10->12</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="182.62" y="-302.45" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge24" class="edge"><title>10->24</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="128.62" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="79.62" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="128.62" y="-201.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge18" class="edge"><title>13->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="174.62" y="-201.95" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge19" class="edge"><title>13->19</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="79.62" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="79.62" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="79.62" y="-51.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="174.62" y="-151.7" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="174.62" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="174.62" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge22" class="edge"><title>21->22</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="174.62" y="-0.95" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge23" class="edge"><title>22->23</title></g></g></svg>

```py
mystring = "1 * (2 + 3.35)"
for tree in earley.parse(mystring):
    assert tree_to_string(tree) == mystring
    display(display_tree(tree)) 
```

<svg width="312pt" height="625pt" viewBox="0.00 0.00 312.00 625.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 621.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="81" y="-603.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="81" y="-553.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="81" y="-503.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="30" y="-453.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="81" y="-453.2" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge7" class="edge"><title>2->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="128" y="-453.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge8" class="edge"><title>2->8</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-402.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="21" y="-352.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="21" y="-302.45" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="128" y="-402.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="75" y="-352.7" font-family="Times,serif" font-size="14.00">( (40)</text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="129" y="-352.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge11" class="edge"><title>9->11</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="183" y="-352.7" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge31" class="edge"><title>9->31</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="80" y="-302.45" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="129" y="-302.45" font-family="Times,serif" font-size="14.00">+</text></g> <g id="edge17" class="edge"><title>11->17</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="176" y="-302.45" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge18" class="edge"><title>11->18</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="80" y="-252.2" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="72" y="-201.95" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="56" y="-151.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="56" y="-101.45" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="177" y="-252.2" font-family="Times,serif" font-size="14.00"><term></text></g> <g id="edge19" class="edge"><title>18->19</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="181" y="-201.95" font-family="Times,serif" font-size="14.00"><factor></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="122" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="182" y="-151.7" font-family="Times,serif" font-size="14.00">. (46)</text></g> <g id="edge24" class="edge"><title>20->24</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="242" y="-151.7" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge25" class="edge"><title>20->25</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="122" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge22" class="edge"><title>21->22</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="122" y="-51.2" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="211" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge26" class="edge"><title>25->26</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="277" y="-101.45" font-family="Times,serif" font-size="14.00"><integer></text></g> <g id="edge28" class="edge"><title>25->28</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="211" y="-51.2" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge27" class="edge"><title>26->27</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="277" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="277" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge30" class="edge"><title>29->30</title></g></g></svg>

与上面的 `PEGParser` 相比，`EarleyParser` 可以处理任意上下文无关文法。

### 探索：测试解析器

虽然我们已经定义了两种解析器变体，但得到一些确认我们的解析工作良好的信息会很好。虽然可以形式化证明它们是有效的，但生成随机的文法、它们对应的字符串，并使用相同的文法进行解析，会更有满足感。

```py
def prod_line_grammar(nonterminals, terminals):
    g = {
        '<start>': ['<symbols>'],
        '<symbols>': ['<symbol><symbols>', '<symbol>'],
        '<symbol>': ['<nonterminals>', '<terminals>'],
        '<nonterminals>': ['<lt><alpha><gt>'],
        '<lt>': ['<'],
        '<gt>': ['>'],
        '<alpha>': nonterminals,
        '<terminals>': terminals
    }

    if not nonterminals:
        g['<nonterminals>'] = ['']
        del g['<lt>']
        del g['<alpha>']
        del g['<gt>']

    return g 
```

```py
syntax_diagram(prod_line_grammar(["A", "B", "C"], ["1", "2", "3"])) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 199.5 62" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">symbols</text></g></g></g></g></svg>

```py
symbols

```

<svg class="railroad-diagram" height="92" viewBox="0 0 290.5 92" width="290.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="95.5" y="35">symbol</text></g> <g class="non-terminal"><text x="190.75" y="35">symbols</text></g></g> <g><g class="non-terminal"><text x="145.25" y="65">symbol</text></g></g></g></g></svg>

```py
symbol

```

<svg class="railroad-diagram" height="92" viewBox="0 0 242.0 92" width="242.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="121.0" y="35">nonterminals</text></g></g> <g><g class="non-terminal"><text x="121.0" y="65">terminals</text></g></g></g></g></svg>

```py
nonterminals

```

<svg class="railroad-diagram" height="62" viewBox="0 0 296.5 62" width="296.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="78.5" y="35">lt</text></g> <g class="non-terminal"><text x="148.25" y="35">alpha</text></g> <g class="non-terminal"><text x="218.0" y="35">gt</text></g></g></g></g></svg>

```py
lt

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35"><</text></g></g></g></g></svg>

```py
gt

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">></text></g></g></g></g></svg>

```py
alpha

```

<svg class="railroad-diagram" height="122" viewBox="0 0 148.5 122" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">A</text></g></g> <g><g class="terminal"><text x="74.25" y="65">B</text></g></g> <g><g class="terminal"><text x="74.25" y="95">C</text></g></g></g></g></svg>

```py
terminals

```

<svg class="railroad-diagram" height="122" viewBox="0 0 148.5 122" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">1</text></g></g> <g><g class="terminal"><text x="74.25" y="65">2</text></g></g> <g><g class="terminal"><text x="74.25" y="95">3</text></g></g></g></g></svg>

```py
def make_rule(nonterminals, terminals, num_alts):
    prod_grammar = prod_line_grammar(nonterminals, terminals)

    gf = GrammarFuzzer(prod_grammar, min_nonterminals=3, max_nonterminals=5)
    name = "<%s>" % ''.join(random.choices(string.ascii_uppercase, k=3))

    return (name, [gf.fuzz() for _ in range(num_alts)]) 
```

```py
make_rule(["A", "B", "C"], ["1", "2", "3"], 3) 
```

```py
('<FYU>', ['<C>23', '<C><A>', '<B><C>3'])

```

```py
from Grammars import unreachable_nonterminals 
```

```py
def make_grammar(num_symbols=3, num_alts=3):
    terminals = list(string.ascii_lowercase)
    grammar = {}
    name = None
    for _ in range(num_symbols):
        nonterminals = [k[1:-1] for k in grammar.keys()]
        name, expansions = \
            make_rule(nonterminals, terminals, num_alts)
        grammar[name] = expansions

    grammar[START_SYMBOL] = [name]

    # Remove unused parts
    for nonterminal in unreachable_nonterminals(grammar):
        del grammar[nonterminal]

    assert is_valid_grammar(grammar)

    return grammar 
```

```py
make_grammar() 
```

```py
{'<ILY>': ['lhp', 'gta', 'sm'],
 '<FZD>': ['qn<ILY>', 'e<ILY><ILY>g', '<ILY>f<ILY>m'],
 '<ITK>': ['<ILY>fyy', '<ILY><ILY>t', '<FZD>l<ILY>ao'],
 '<start>': ['<ITK>']}

```

现在我们验证我们的任意文法是否可以被 Earley 解析器使用。

```py
for i in range(5):
    my_grammar = make_grammar()
    print(my_grammar)
    parser = EarleyParser(my_grammar)
    mygf = GrammarFuzzer(my_grammar)
    s = mygf.fuzz()
    print(s)
    for tree in parser.parse(s):
        assert tree_to_string(tree) == s
        display_tree(tree) 
```

```py
{'<SCS>': ['ts', 'f', 'ng'], '<BQN>': ['wm<SCS>', '<SCS>wi', '<SCS>hw'], '<UZC>': ['gyk<BQN>br', '<SCS>iqp', '<BQN>vb'], '<start>': ['<UZC>']}
fhwvb
{'<CRN>': ['meze', 'de', 'cpcv'], '<AIS>': ['<CRN>hb', 'dc<CRN>', 'pa<CRN>x'], '<MAO>': ['<CRN>su', '<CRN>hj', '<CRN><AIS>g'], '<start>': ['<MAO>']}
dehj
{'<MFY>': ['y', 'w', ''], '<ZOY>': ['oe<MFY>', 'h<MFY>u', 'lowr'], '<HFT>': ['<ZOY>ro', '<ZOY>w', '<ZOY><ZOY>w'], '<start>': ['<HFT>']}
lowrro
{'<CYC>': ['cg', 'enl', 'ovd'], '<TUV>': ['<CYC>hf', '<CYC>nl', 'fhg'], '<MOQ>': ['g<TUV>g', '<CYC>ix', '<CYC><TUV><CYC>'], '<start>': ['<MOQ>']}
cgix
{'<WJJ>': ['dszdlh', 'j', 'fd'], '<RQM>': ['<WJJ>wx', 'xs<WJJ><WJJ>', '<WJJ>x'], '<JNY>': ['<WJJ>oa', '<WJJ><WJJ>cx', 'xd<RQM>'], '<start>': ['<JNY>']}
joa

```

通过这种方式，我们已经完成了对 *任意* CFG 的实现和测试，现在它可以与 `LangFuzzer` 一起使用，以生成更好的模糊测试输入。

## 背景

存在许多解析技术，可以使用给定的文法解析给定的字符串，并产生相应的推导树或树。然而，其中一些技术仅在特定的文法类别上工作。这些文法类别以可以接受该类别文法的特定类型的解析器命名。也就是说，解析器能力的上限定义了以该解析器命名的文法类别。

*LL* 和 *LR* 解析是解析的主要传统。在这里，*LL* 表示从左到右、最左推导，它代表自顶向下的方法。另一方面，LR（从左到右、最右推导）代表自底向上的方法。另一种看待它的方式是，LL 解析器按 *前序* 递增地计算推导树，而 LR 解析器按 *后序* 计算推导树 [Pingali 等人，2015])。

不同的文法类别在用户编写该类别文法时可以使用的特性上有所不同。也就是说，相应的解析器将无法解析使用比允许的更多特性的文法。例如，`A2_GRAMMAR` 是一个 *LL* 文法，因为它没有左递归，而 `A1_GRAMMAR` 不是一个 *LL* 文法。这是因为 *LL* 解析器从左到右解析其输入，并通过展开它遇到的非终结符来构建其输入的最左推导。如果这些规则之一有左递归，*LL* 解析器将进入无限循环。

同样，一个文法如果是 LL(k) 的，那么它可以被一个具有 k 个前瞻标记的 LL 解析器解析，而 LR(k) 文法只能用至少 k 个前瞻标记的 LR 解析器解析。这些文法很有趣，因为 LL(k) 和 LR(k) 文法都有 $O(n)$ 解析器，并且与其他文法相比，可以在相对有限的计算预算下使用。

可以提供*LL(k)*语法的语言被称为*LL(k)*语言（其中 k 是所需的最小前瞻）。类似地，*LR(k)*定义为具有*LR(k)*语法的语言集合。就语言而言，LL(k) $\subset$ LL(k+1) 和 LL(k) $\subset$ LR(k)，且 *LR(k)* $=$ *LR(1)*。所有确定性的 *CFLs* 都有一个 *LR(1)* 语法。然而，存在本质上模糊的 *CFLs* [Ogden 等人，1968]，对于这些语言，无法提供 *LR(1)* 语法。

对于*CFGs*的其它主要解析算法是 GLL [Scott 等人，2010]，GLR [Tomita 等人，1987, Tomita 等人，2012]，和 CYK [Grune 等人，2008]。另一方面，ALL(*)（由 ANTLR 使用）是一种使用类似正则表达式谓词的语法表示（类似于高级 PEGs – 见 练习），而不是固定的前瞻。因此，ALL(*)可以接受比 CFGs 更广泛的语法类别。

在解析的计算限制方面，主要的 CFG 解析器对于任意语法具有$O(n³)$的复杂度。然而，使用任意*CFG*进行解析可以简化为布尔矩阵乘法 [[Valiant 等人，1975](https://doi.org/10.1016/S0022-0000(75)80046-8)]（以及反向操作 [[Lee 等人，2002](https://doi.org/10.1145/505241.505242)]）。目前，这被限制在$O(2^{23728639})$ [[Le Gall 等人，2014](https://doi.org/10.1145/2608628.2608664)]。因此，解析任意 CFG 的最坏情况复杂度可能仍然接近立方。

关于 PEGs，目前尚不清楚可以用*PEG*表达的语言的实际类别。特别是，我们知道*PEGs*可以表达某些语言，例如$a^n b^n c^n$。然而，我们不知道是否存在不能用*PEGs*表达的*CFLs*。在 2.3 节中，我们提供了一个反直觉的 PEG 语法的例子。虽然这对于我们的目的很重要（我们使用语法来生成输入），但这并不是对使用 PEG 进行解析的批评。PEG 专注于编写用于识别给定语言的语法，而不一定是解释任意 PEG 可能产生的语言。给定一个要解析的上下文无关语言，几乎总是可以为其编写一个 PEG 语法。鉴于 1）一个 PEG 可以在$O(n)$时间内解析任何字符串，2）目前我们知道没有 CFL 不能表示为 PEG，以及 3）与*LR*语法相比，PEG 通常更直观，因为它允许自顶向下的解释，在编写语言解析器时，应该认真考虑使用 PEG。

## 经验教训

+   语法可以用来为给定的字符串生成推导树。

+   解析表达式语法直观且易于实现，但需要小心编写。

+   Earley 解析器可以解析任意的上下文无关语法。

## 下一步

+   使用解析后的输入重新组合现有输入

## 练习

### 练习 1：另一种 Packrat

在 *Packrat* 解析器中，我们展示了如何实现一个简单的 *PEG* 解析器。该解析器使用索引跟踪文本中的当前位置。你能修改解析器，使其仅使用当前子串而不是跟踪索引吗？也就是说，它不再需要 `at` 参数。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来练习习题并查看解决方案。

**解答**。以下是一个可能的解决方案：

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来练习习题并查看解决方案。

### 习题 2：更多 PEG 语法

*PEG* 语法提供了一些类似于正则表达式的记法便利。例如，它支持以下运算符（字母 `T` 和 `A` 代表可以是终结符或非终结符的标记。`ε` 是空字符串，而 `/` 是有序选择运算符，类似于非有序选择运算符 `|`）：

+   `T?` 表示 `T` 的可选贪婪匹配，`A := T?` 等价于 `A := T/ε`。

+   `T*` 表示 `T` 的零个或多个贪婪匹配，`A := T*` 等价于 `A := T A/ε`。

+   `T+` 表示一个或多个贪婪匹配——等价于 `TT*`

如果你查看上述三种记法，每种都可以在语法中以基本语法的形式表示。记得 语法章节 中的练习，它开发了 `define_ex_grammar()`，可以将语法表示为 Python 代码？将 `define_ex_grammar()` 扩展为 `define_peg()` 以支持上述记法便利。装饰器应将包含这些记法的给定语法重写为等价的基本语法语法。

### 习题 3：PEG 谓词

除了这些记法便利之外，它还支持两个谓词，可以提供不消耗任何输入的强大前瞻功能。

+   `T&A` 表示一个 *And-谓词*，如果 `T` 匹配，并且它立即后面跟着 `A`

+   `T!A` 表示一个 *Not-谓词*，如果 `T` 匹配，并且它不是立即后面跟着 `A`

在我们的 *PEG* 解析器中实现这些谓词。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来练习习题并查看解决方案。

### 习题 4：Earley 填充图表

在 `Earley Parser` 的 `Column` 类中，我们既以 `list` 的形式也以 `dict` 的形式保存状态，尽管 `dict` 是有序的。你能解释一下为什么吗？

**提示**：查看 `fill_chart` 方法。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来练习习题并查看解决方案。

### 习题 5：Leo 解析器

原始 Earley 解析器的一个问题是，虽然它可以使用任意的 *上下文无关文法* 解析字符串，但其对右递归文法的性能是二次的。也就是说，使用右递归文法解析需要 $O(n²)$ 的时间和空间。例如，考虑以下字符串通过两个不同的文法 `LR_GRAMMAR` 和 `RR_GRAMMAR` 的解析。

```py
mystring = 'aaaaaa' 
```

要看到这个问题，我们需要启用日志记录。以下是使用 `LR_GRAMMAR` 进行解析的日志版本。

```py
result = EarleyParser(LR_GRAMMAR, log=True).parse(mystring)
for _ in result: pass # consume the generator so that we can see the logs 
```

```py
None chart[0]
<A>:= |(0,0)
<start>:= <A> |(0,0) 

a chart[1]
<A>:= <A> a |(0,1)
<start>:= <A> |(0,1) 

a chart[2]
<A>:= <A> a |(0,2)
<start>:= <A> |(0,2) 

a chart[3]
<A>:= <A> a |(0,3)
<start>:= <A> |(0,3) 

a chart[4]
<A>:= <A> a |(0,4)
<start>:= <A> |(0,4) 

a chart[5]
<A>:= <A> a |(0,5)
<start>:= <A> |(0,5) 

a chart[6]
<A>:= <A> a |(0,6)
<start>:= <A> |(0,6) 

```

将其与以下 `RR_GRAMMAR` 的解析进行比较：

```py
result = EarleyParser(RR_GRAMMAR, log=True).parse(mystring)
for _ in result: pass 
```

```py
None chart[0]
<A>:= |(0,0)
<start>:= <A> |(0,0) 

a chart[1]
<A>:= |(1,1)
<A>:= a <A> |(0,1)
<start>:= <A> |(0,1) 

a chart[2]
<A>:= |(2,2)
<A>:= a <A> |(1,2)
<A>:= a <A> |(0,2)
<start>:= <A> |(0,2) 

a chart[3]
<A>:= |(3,3)
<A>:= a <A> |(2,3)
<A>:= a <A> |(1,3)
<A>:= a <A> |(0,3)
<start>:= <A> |(0,3) 

a chart[4]
<A>:= |(4,4)
<A>:= a <A> |(3,4)
<A>:= a <A> |(2,4)
<A>:= a <A> |(1,4)
<A>:= a <A> |(0,4)
<start>:= <A> |(0,4) 

a chart[5]
<A>:= |(5,5)
<A>:= a <A> |(4,5)
<A>:= a <A> |(3,5)
<A>:= a <A> |(2,5)
<A>:= a <A> |(1,5)
<A>:= a <A> |(0,5)
<start>:= <A> |(0,5) 

a chart[6]
<A>:= |(6,6)
<A>:= a <A> |(5,6)
<A>:= a <A> |(4,6)
<A>:= a <A> |(3,6)
<A>:= a <A> |(2,6)
<A>:= a <A> |(1,6)
<A>:= a <A> |(0,6)
<start>:= <A> |(0,6) 

```

从每个字母的解析日志中可以看出，具有表示 `<A>: a <A> ● (i, j)` 的状态数量在每一阶段都会增加，这些状态仅仅是前一个字母的遗留。它们除了简单地完成这些条目外，对解析没有做出任何更多贡献。然而，它们占用空间，并需要资源进行检查，在分析中贡献了一个 `n` 的因子。

Joop Leo [[Joop M.I.M. Leo, 1991](https://doi.org/10.1016/0304-3975(91)90180-A)] 发现，通过检测右递归可以避免这种低效。想法是在开始 `completion` 步骤之前，检查当前项是否有 *确定性归约路径*。如果存在这样的路径，将 *确定性归约路径* 的最顶层元素的一个副本添加到当前列，并返回。如果没有，执行原始的 `completion` 步骤。

**定义 2.1**：如果一个项位于 $[A \rightarrow \gamma., i]$ 上方的确定性归约路径上，如果它是 $[B \rightarrow \alpha A ., k]$，其中 $[B \rightarrow \alpha . A, k]$ 是 $ I_i $ 中唯一带有 A 前点的项，或者如果它位于 $[B \rightarrow \alpha A ., k]$ 上方的确定性归约路径上。如果一个项位于这样的路径上，并且没有项位于其上方的确定性归约路径上，则称为 *最顶层* 项[[Joop M.I.M. Leo, 1991](https://doi.org/10.1016/0304-3975(91)90180-A)]。

寻找一个 *确定性归约路径* 的方法如下：

给定一个完整的状态，表示为 `<A> : seq_1 ● (s, e)`，其中 `s` 是此规则的起始列，`e` 是当前列，如果满足两个约束条件，则在其上方存在一个 *确定性归约路径*。

1.  在列 `s` 中存在一个形式为 `<B> : seq_2 ● <A> (k, s)` 的 *单个* 项。

1.  这应该是 `s` 中带有 `<A>` 前点的 *单个* 项

结果项的形式为 `<B> : seq_2 <A> ● (k, e)`，这仅仅是 (1) 中的项的扩展，并且在确定性归约路径中被视为高于 `<A>:.. (s, e)`。`seq_1` 和 `seq_2` 是任意的符号序列。

这形成以下链接链，其中 `<A>:.. (s_1, e)` 是 `<B>:.. (s_2, e)` 的子项等。

这里有一种可视化链的方法：

```py
<C> : seq_3 <B> ● (s_3, e)  
             |  constraints satisfied by <C> : seq_3 ● <B> (s_3, s_2)
            <B> : seq_2 <A> ● (s_2, e)  
                         | constraints satisfied by <B> : seq_2 ● <A> (s_2, s_1)
                        <A> : seq_1 ● (s_1, e)
```

实质上，我们想要做的是识别潜在的确定性右递归候选者，对它们进行完善，然后*丢弃结果*。我们这样做，直到达到最顶层。参见 Grune 等人~[Grune *et al*, 2008]以获取更多信息。

注意，完善是在同一列（`e`）中进行的，每个候选者满足的约束在越来越早的列中（如下所示）：

```py
<C> : seq_3 ● <B> (s_3, s_2)  -->              <C> : seq_3 <B> ● (s_3, e)
               |
              <B> : seq_2 ● <A> (s_2, s_1) --> <B> : seq_2 <A> ● (s_2, e)  
                             |
                            <A> : seq_1 ●                        (s_1, e)
```

沿着这个链，最顶层的项是 `<C>:.. (s_3, e)`，它没有父项。需要保存的最顶层项被 Leo 称为*传递*项，它与开始查找的非终结符号相关联。传递项需要添加到我们检查的每一列中。

这是解析器`LeoParser`的框架。

```py
class LeoParser(EarleyParser):
    def complete(self, col, state):
        return self.leo_complete(col, state)

    def leo_complete(self, col, state):
        detred = self.deterministic_reduction(state)
        if detred:
            col.add(detred.copy())
        else:
            self.earley_complete(col, state)

    def deterministic_reduction(self, state):
        raise NotImplementedError 
```

你能实现`deterministic_reduction()`方法以获取最顶层元素吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来处理练习并查看解决方案。

**解决方案。**以下是一个可能的解决方案：

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来处理练习并查看解决方案。

**高级:** 我们已经固定了复杂度界限。然而，因为我们只保存了右递归的最顶层项，所以我们需要调整我们的解析器，使其在提取解析树时意识到这一调整。你能修复它吗？

**提示:** Leo 建议简单地将 Leo 项集转换为正常的 Earley 集，并将确定性归约的结果扩展到其原始形式。为此，请记住我们之前绘制的约束链图。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来处理练习并查看解决方案。

**解决方案。**以下是一个可能的解决方案。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来处理练习并查看解决方案。

### 练习 6：过滤后的 Earley 解析器

我们 Earley 和 Leo 解析器的一个问题是，当使用包含替代项中标记重复的语法进行解析时，它可能会陷入无限循环。例如，考虑以下语法。

```py
RECURSION_GRAMMAR: Grammar = {
    "<start>": ["<A>"],
    "<A>": ["<A>", "<A>aa", "AA", "<B>"],
    "<B>": ["<C>", "<C>cc", "CC"],
    "<C>": ["<B>", "<B>bb", "BB"]
} 
```

使用这种语法，可以产生无限链的推导 `<A>`（直接递归）或无限链的推导 `<B> -> <C> -> <B> ...`（间接递归）。问题是，我们的实现可能会陷入试图推导这些无限链的困境。一种可能性是使用`LazyExtractor`。另一种可能性是简单地避免生成这样的链。

```py
from ExpectError import ExpectTimeout 
```

```py
with ExpectTimeout(1, print_traceback=False):
    mystring = 'AA'
    parser = LeoParser(RECURSION_GRAMMAR)
    tree, *_ = parser.parse(mystring)
    assert tree_to_string(tree) == mystring
    display_tree(tree) 
```

```py
RecursionError: maximum recursion depth exceeded (expected)

```

你能实现一个解决方案，使得任何包含这种链的树都被丢弃吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来处理练习并查看解决方案。

**解决方案。**以下是一个可能的解决方案。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 7：迭代 Earley 解析器

在某些情况下，递归算法非常方便，但有时由于内存或速度问题，我们可能希望使用迭代而不是递归。

你能实现一个 `EarleyParser` 的迭代版本吗？

**提示**：通常，你可以使用栈将递归算法替换为迭代算法。一种简单的方法是将参数推入栈中，而不是传递给递归函数。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises) 来完成练习并查看解决方案。

**解决方案**。这里是一个可能的解决方案。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 8：非终止符号的第一个集合

我们之前提供了一种提取 `nullable`（epsilon）集的方法，这通常用于解析。除了 `nullable`，解析算法通常还会使用另外两个集合 `first` 和 `follow`（[`en.wikipedia.org/wiki/Canonical_LR_parser#FIRST_and_FOLLOW_sets`](https://en.wikipedia.org/wiki/Canonical_LR_parser#FIRST_and_FOLLOW_sets)）。终止符号的第一个集合是其自身，非终止符号的第一个集合是由可以出现在该非终止符号任何推导开头的终止符号组成的。任何可以推导空字符串的非终止符号的第一个集合应包含 `EPSILON`。例如，使用我们的 `A1_GRAMMAR`，`<expr>` 和 `<start>` 的第一个集合是 `{0,1,2,3,4,5,6,7,8,9}`。对于任何自递归非终止符号的第一个集合的提取是足够简单的。只需递归地计算其选择表达式中第一个元素的第一个集合即可。对于自递归非终止符号的第一个集合的计算是复杂的。必须递归地计算第一个集合，直到确定不再可以向第一个集合中添加更多的终止符号。

你能使用我们的 `fixpoint()` 装饰器实现 `first` 集合吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 9：非终止符号的 follow 集合

follow 集合的定义与 first 集合类似。非终止符号的 follow 集合是在任何推导中使用该非终止符号之后可能出现的终止符号的集合。起始符号的 follow 集合是 `EOF`，任何非终止符号的 follow 集合是任何选择表达式中跟在其后的所有符号的第一个集合的超集。

例如，`A1_GRAMMAR` 中 `<expr>` 的 follow 集合是集合 `{EOF, +, -}`。

与上一个练习类似，使用 `fixpoint()` 装饰器实现 `followset()`。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来完成练习并查看解决方案。

### 练习 10：一个 LL(1) 解析器

如我们之前提到的，存在其他类型的解析器，它们以从左到右的方式操作，使用最右推导（*LR(k)*）或最左推导（*LL(k)*），其中 *k* 表示解析器被允许使用的先行符的数量。

那么应该怎么做呢？这个先行符可以用来确定应用哪个规则。在 *LL(1)* 解析器的情况下，应用哪个规则是通过查看不同规则的 *first* 集合来确定的。我们之前实现了 `first_expr()`，它接受一个表达式、`nullables` 集合，并计算该规则的第一个集合。

如果一个规则可以推导出空集，那么如果看到相应非终结符的 `follow()` 集合，该规则也可能适用。

#### 第一部分：一个 LL(1) 解析表

本练习的第一部分是实现描述在看到先行符上的终结符时 *LL(1)* 解析器应采取什么操作的 *解析表*。该表应以 *字典* 的形式呈现，其中键代表非终结符，值应包含另一个字典，其键为终结符，值为继续解析的特定规则。

让我们用一个例子来说明这个表格。`parse_table()` 方法填充一个 `self.table` 数据结构，该结构应符合以下要求：

```py
class LL1Parser(Parser):
    def parse_table(self):
        self.my_rules = rules(self.cgrammar)
        self.table = ...          # fill in here to produce

    def rules(self):
        for i, rule in enumerate(self.my_rules):
            print(i, rule)

    def show_table(self):
        ts = list(sorted(terminals(self.cgrammar)))
        print('Rule Name\t| %s' % ' | '.join(t for t in ts))
        for k in self.table:
            pr = self.table[k]
            actions = list(str(pr[t]) if t in pr else ' ' for t in ts)
            print('%s  \t| %s' % (k, ' | '.join(actions))) 
```

当调用 `LL1Parser(A2_GRAMMAR).show_table()` 时，应该得到以下表格：

```py
for i, r in enumerate(rules(canonical(A2_GRAMMAR))):
    print("%d\t  %s := %s" % (i, r[0], r[1])) 
```

```py
0	 <start> := ['<expr>']
1	 <expr> := ['<integer>', '<expr_>']
2	 <expr_> := ['+', '<expr>']
3	 <expr_> := ['-', '<expr>']
4	 <expr_> := []
5	 <integer> := ['<digit>', '<integer_>']
6	 <integer_> := ['<integer>']
7	 <integer_> := []
8	 <digit> := ['0']
9	 <digit> := ['1']
10	 <digit> := ['2']
11	 <digit> := ['3']
12	 <digit> := ['4']
13	 <digit> := ['5']
14	 <digit> := ['6']
15	 <digit> := ['7']
16	 <digit> := ['8']
17	 <digit> := ['9']

```

| Rule Name |  | + | - | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| start |  |  |  | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| expr |  |  |  | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| expr_ |  | 2 | 3 |  |  |  |  |  |  |  |  |  |  |
| integer |  |  |  | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 5 |
| integer_ |  | 7 | 7 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 |
| digit |  |  |  | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 |

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)来完成练习并查看解决方案。

#### 第二部分：解析器

一旦我们有了解析表，实现解析器的步骤如下：考虑要解析的标记序列中的第一个标记，并用起始符号初始化栈。

当栈不为空时，从栈中提取第一个符号，如果该符号是终结符，则验证该符号是否与输入流中的项匹配。如果符号是非终结符，则使用该符号和输入项从解析表中查找下一个规则。将找到的规则插入到栈顶。跟踪正在解析的表达式以构建解析表。

使用之前定义的解析表来实现完整的 LL(1) 解析器。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)进行练习并查看解决方案。

**解决方案**。以下是完整的解析器：

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Parser.ipynb#Exercises)进行练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，均受[MIT License](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。最后修改时间：2024-11-09 17:07:29+01:00。引用 · [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

安德烈亚斯·泽勒（Andreas Zeller）、拉胡尔·戈皮纳特（Rahul Gopinath）、马塞尔·博姆（Marcel Böhme）、戈登·弗朗西斯（Gordon Fraser）和克里斯蒂安·霍勒（Christian Holler）："[解析输入](https://www.fuzzingbook.org/html/Parser.html)"。收录于安德烈亚斯·泽勒、拉胡尔·戈皮纳特、马塞尔·博姆、戈登·弗朗西斯和克里斯蒂安·霍勒的《[模糊测试书](https://www.fuzzingbook.org/)[`www.fuzzingbook.org/html/Parser.html`](https://www.fuzzingbook.org/html/Parser.html)》中。检索时间：2024-11-09 17:07:29+01:00。

```py
@incollection{fuzzingbook2024:Parser,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Parsing Inputs},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/Parser.html}},
    note = {Retrieved 2024-11-09 17:07:29+01:00},
    url = {https://www.fuzzingbook.org/html/Parser.html},
    urldate = {2024-11-09 17:07:29+01:00}
}

```
