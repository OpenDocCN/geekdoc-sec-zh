# 符号模糊测试

> 原文：[`www.fuzzingbook.org/html/ConcolicFuzzer.html`](http://www.fuzzingbook.org/html/ConcolicFuzzer.html)

在信息流章节中，我们看到了如何使用动态污点来生成比仅仅寻找程序崩溃更智能的测试用例。我们也看到了如何使用污点来更新语法，从而更专注于危险的方法。

虽然污点很有帮助，但未解释的字符串只是攻击向量之一。我们能否对执行过程中任何点的变量属性说些什么？例如，我们能否肯定地说一个函数将始终接收到正确长度的缓冲区？

*符号执行*在这里提供了一个解决方案。对于函数的*符号执行*的想法如下：我们从函数的一个样本输入开始，在跟踪下执行函数。在执行通过每个条件点时，我们以*符号变量之间的关系*的形式*保存遇到的条件*。在这里，一个*符号变量*可以被视为一种真实变量的占位符，类似于在代数中求解 x 时使用的 x。符号变量可以用来指定关系，而无需实际解决它们。

通过符号执行，我们可以收集执行路径遇到的约束，并使用它来回答关于程序在任何我们喜欢的程序执行路径点的行为的问题。我们可以进一步使用符号执行来增强模糊测试。

在本章中，我们深入探讨了如何执行 Python 函数的符号执行，以及如何利用符号执行来增强模糊测试。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('KDcMjWX5ulU') 
```

**先决条件**

+   你应该已经阅读了覆盖率章节。

+   你应该已经阅读了信息流章节。

+   熟悉[SMT 求解器](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories)的基本概念会有所帮助。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import List, Callable, Dict, Tuple 
```

## 概述

要使用本章提供的代码，请编写

```py
>>> from fuzzingbook.ConcolicFuzzer import <identifier> 
```

然后利用以下功能。

本章定义了两个主要类：`SimpleConcolicFuzzer`和`ConcolicGrammarFuzzer`。`SimpleConcolicFuzzer`首先使用样本输入收集遇到的谓词。模糊器随后否定随机谓词以生成新的输入约束。这些约束在解决后产生探索接近原始路径的路径的输入。

### ConcolicTracer

两个模糊器的核心概念是一个*符号跟踪器*，它捕获程序执行过程中的符号变量和路径条件。

`ConcolicTracer`在`with`块中使用；`tracer[function]`的语法在`tracer`中执行`function`，同时捕获条件。以下是对`cgi_decode()`函数的示例：

```py
>>> with ConcolicTracer() as _:
>>>     _cgi_decode 
```

执行后，我们可以在`decls`属性中检索符号变量。这是一个符号变量到类型的映射。

```py
>>> _.decls
{'cgi_decode_s_str_1': 'String'} 
```

提取的路径条件可以在`path`属性中找到：

```py
>>> _.path
[0 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 0, 1) == "+"),
 Not(str.substr(cgi_decode_s_str_1, 0, 1) == "%"),
 1 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 1, 1) == "+"),
 str.substr(cgi_decode_s_str_1, 1, 1) == "%",
 Not(str.substr(cgi_decode_s_str_1, 2, 1) == "0"),
 Not(str.substr(cgi_decode_s_str_1, 2, 1) == "1"),
 str.substr(cgi_decode_s_str_1, 2, 1) == "2",
 str.substr(cgi_decode_s_str_1, 3, 1) == "0",
 4 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 4, 1) == "+"),
 Not(str.substr(cgi_decode_s_str_1, 4, 1) == "%"),
 Not(5 < Length(cgi_decode_s_str_1))] 
```

`context` 属性包含一对 `decls` 和 `path` 属性；这对于将其传递给 `ConcolicTracer` 构造函数很有用。

```py
>>> assert _.context == (_.decls, _.path) 
```

我们可以解决这些约束，以获得一个与原始（跟踪）调用相同路径的函数参数值：

```py
>>> _.zeval()
('sat', {'s': ('A%20B', 'String')}) 
```

`zeval()` 函数还允许传递 *替代* 或 *否定* 约束。请参阅相关章节中的示例。

<svg width="250pt" height="145pt" viewBox="0.00 0.00 250.25 145.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 141)"><g id="node1" class="node"><title>ConcolicTracer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class ConcolicTracer:

跟踪函数执行，跟踪变量和路径条件"><text text-anchor="start" x="8" y="-120.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">ConcolicTracer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="ConcolicTracer"><g id="a_node1_1"><a xlink:href="#" xlink:title="__call__(self, *args):

将 self 作为函数调用。"><text text-anchor="start" x="13.62" y="-98" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__call__()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="__init__(self, context=None):

构造函数。"><text text-anchor="start" x="13.62" y="-85.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__init__()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="zeval(self, predicates=None, *, python=False, log=False):

在当前上下文中评估 `predicates`。

- 如果设置了 `python`，则使用 z3 Python API；否则使用 z3 独立版本。

如果设置了 `log`，则显示 z3 的输入。

返回一个对 (`result`, `solution`)，其中

- `result` 是 `'sat'`（可满足）时；

`solution` 是变量到（值，类型）对的映射；或者

- `result`不是`'sat'`，表示错误；然后`solution`是`None`"><text text-anchor="start" x="13.62" y="-72.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">zeval()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="__enter__(self)"><text text-anchor="start" x="13.62" y="-58.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">__enter__()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="__exit__(self, exc_type, exc_value, tb)"><text text-anchor="start" x="13.62" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">__exit__()</text></a></g> <g id="a_node1_6"><a xlink:href="#" xlink:title="__getitem__(self, fn)"><text text-anchor="start" x="13.62" y="-33.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">__getitem__()</text></a></g> <g id="a_node1_7"><a xlink:href="#" xlink:title="concolic(self, args)"><text text-anchor="start" x="13.62" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">concolic()</text></a></g> <g id="a_node1_8"><a xlink:href="#" xlink:title="smt_expr(self, show_decl=False, simplify=False, path=[])"><text text-anchor="start" x="13.62" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">smt_expr()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>图例</title> <text text-anchor="start" x="123" y="-84.5" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="123" y="-74.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="129" y="-74.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="123" y="-64.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="129" y="-64.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="123" y="-54.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="129" y="-54.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="123" y="-45.45" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

### SimpleConcolicFuzzer

从`ConcolicTracer`获得的约束被添加到 Concolic fuzzer 中，如下所示：

```py
>>> scf = SimpleConcolicFuzzer()
>>> scf.add_trace(_, 'a%20d') 
```

Concolic fuzzer 随后使用添加的约束来指导其模糊测试，如下所示：

```py
>>> scf = SimpleConcolicFuzzer()
>>> for i in range(20):
>>>     v = scf.fuzz()
>>>     if v is None:
>>>         break
>>>     print(repr(v))
>>>     with ExpectError(print_traceback=False):
>>>         with ConcolicTracer() as _:
>>>             _cgi_decode
>>>     scf.add_trace(_, v)
' '
'%'
''
'AB'
'A+'
'%'
'A'
'AB'
'+'

IndexError: string index out of range (expected)
IndexError: string index out of range (expected)
IndexError: string index out of range (expected)

'%'
'A%'
'%'
'A+'
'A'
'AB'
'A'
'A+B'
'ABC'

IndexError: string index out of range (expected)
IndexError: string index out of range (expected)
IndexError: string index out of range (expected)

'AB%'
'A%'

IndexError: string index out of range (expected) 
```

我们看到额外的输入是如何探索额外路径的。

<svg width="294pt" height="229pt" viewBox="0.00 0.00 294.38 228.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 224.75)"><g id="node1" class="node"><title>SimpleConcolicFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class SimpleConcolicFuzzer:

模糊器的基类。"><text text-anchor="start" x="8" y="-81.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">SimpleConcolicFuzzer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="SimpleConcolicFuzzer"><g id="a_node1_1"><a xlink:href="#" xlink:title="__init__(self):

构造函数"><text text-anchor="start" x="35.75" y="-59.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="fuzz(self):

返回模糊输入"><text text-anchor="start" x="35.75" y="-47" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="add_trace(self, trace, s)"><text text-anchor="start" x="35.75" y="-33.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">add_trace()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="get_newpath(self)"><text text-anchor="start" x="35.75" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">get_newpath()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="next_choice(self)"><text text-anchor="start" x="35.75" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">next_choice()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>Fuzzer</title> <g id="a_node2"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊器的基类。"><text text-anchor="start" x="54.12" y="-203.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Fuzzer</text> <g id="a_node2_6"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node2_7"><a xlink:href="Fuzzer.html" xlink:title="__init__(self) -> None:

构造函数"><text text-anchor="start" x="44.75" y="-181.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node2_8"><a xlink:href="Fuzzer.html" xlink:title="fuzz(self) -> str:

返回模糊输入"><text text-anchor="start" x="44.75" y="-169" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node2_9"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

运行 `runner` 并使用模糊输入，`trials` 次数"><text text-anchor="start" x="44.75" y="-156.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">run()</text></a></g> <g id="a_node2_10"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

运行 `runner` 并使用模糊输入，`trials` 次数"><text text-anchor="start" x="44.75" y="-143.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">runs()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>SimpleConcolicFuzzer->Fuzzer</title></g> <g id="node3" class="node"><title>图例</title> <text text-anchor="start" x="167.12" y="-65.38" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="167.12" y="-55.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="173.12" y="-55.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">公共方法()</text> <text text-anchor="start" x="167.12" y="-45.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="173.12" y="-45.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">私有方法()</text> <text text-anchor="start" x="167.12" y="-35.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="173.12" y="-35.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">重载方法()</text> <text text-anchor="start" x="167.12" y="-26.32" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

### ConcolicGrammarFuzzer

The `SimpleConcolicFuzzer` simply explores all paths near the original path traversed by the sample input. It uses a simple mechanism to explore the paths that are near the paths that it knows about, and other than code paths, knows nothing about the input.

另一方面，`ConcolicGrammarFuzzer` 了解输入语法，并且可以在模糊测试过程中收集来自受试者的反馈。它可以将遇到的一些约束提升到语法中，从而实现更深入的模糊测试。其使用方法如下：

```py
>>> from InformationFlow import INVENTORY_GRAMMAR, SQLException
>>> cgf = ConcolicGrammarFuzzer(INVENTORY_GRAMMAR)
>>> cgf.prune_tokens(prune_tokens)
>>> for i in range(10):
>>>     query = cgf.fuzz()
>>>     print(query)
>>>     with ConcolicTracer() as _:
>>>         with ExpectError(print_traceback=False):
>>>             try:
>>>                 res = _db_select
>>>                 print(repr(res))
>>>             except SQLException as e:
>>>                 print(e)
>>>         cgf.update_grammar(_)
>>>         print()
insert into H (DZxQ) values (60366,'QR',-21.2981,6,38.7)
Table ('H') was not found

select 340.0 from i8g4
Table ('i8g4') was not found

delete from months where -16.98==Q000
Invalid WHERE ('-16.98==Q000')

update uKt set D=d,:=m,c=R,R=C where A==Q==M
Table ('uKt') was not found

insert into months (q491) values ('Ib^|}',2,'8/','k')
Column ('q491') was not found

select w from lU where 445==M(v/n*J!=a)>W-e/k-r(n)*G
Table ('lU') was not found

select (r),k-Q*f>Z,((s)),i>N,f!=t from FK9
Table ('FK9') was not found

update m5 set name=U,name=W where (d*D>k)==(d)
Table ('m5') was not found

select H2==h!=R,j-e+F*t,(L),p,W from p_
Table ('p_') was not found

delete from vehicles where _(w)+q/D/x>(e+H>u*b)
Invalid WHERE ('_(w)+q/D/x>(e+H>u*b)') 
```

<svg width="318pt" height="478pt" viewBox="0.00 0.00 317.62 478.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 474.25)"><g id="node1" class="node"><title>ConcolicGrammarFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class ConcolicGrammarFuzzer:

使用推导树高效地生成语法字符串。"><text text-anchor="start" x="13.62" y="-94.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">ConcolicGrammarFuzzer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="ConcolicGrammarFuzzer"><g id="a_node1_1"><a xlink:href="#" xlink:title="fuzz(self):

从语法中生成字符串。"><text text-anchor="start" x="41" y="-72.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="coalesce(self, children)"><text text-anchor="start" x="41" y="-58.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">coalesce()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="prune_tokens(self, tokens)"><text text-anchor="start" x="41" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">prune_tokens()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="prune_tree(self, tree, tokens)"><text text-anchor="start" x="41" y="-33.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">prune_tree()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="tree_to_string(self, tree)"><text text-anchor="start" x="41" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">tree_to_string()</text></a></g> <g id="a_node1_6"><a xlink:href="#" xlink:title="update_grammar(self, trace)"><text text-anchor="start" x="41" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">update_grammar()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>GrammarFuzzer</title> <g id="a_node2"><a xlink:href="GrammarFuzzer.html" xlink:title="class GrammarFuzzer:

使用推导树高效地生成语法字符串。"><text text-anchor="start" x="39.12" y="-331.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarFuzzer</text> <g id="a_node2_7"><a xlink:href="#" xlink:title="GrammarFuzzer"><g id="a_node2_8"><a xlink:href="GrammarFuzzer.html" xlink:title="__init__(self, grammar: Dict[str, List[Union[str, Tuple[str, Dict[str, Any]]]]], start_symbol: str = '<start>', min_nonterminals: int = 0, max_nonterminals: int = 10, disp: bool = False, log: Union[bool, int] = False) -> None:

从`grammar`中生成字符串，以`start_symbol`开始。

如果提供了`min_nonterminals`或`max_nonterminals`，则使用它们作为限制

生成非终结符的数量。

如果`disp`被设置，显示中间的推导树。"><text text-anchor="start" x="41" y="-72.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="coalesce(self, children)"><text text-anchor="start" x="41" y="-58.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">coalesce()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="prune_tokens(self, tokens)"><text text-anchor="start" x="41" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">prune_tokens()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="prune_tree(self, tree, tokens)"><text text-anchor="start" x="41" y="-33.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">prune_tree()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="tree_to_string(self, tree)"><text text-anchor="start" x="41" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">tree_to_string()</text></a></g> <g id="a_node1_6"><a xlink:href="#" xlink:title="update_grammar(self, trace)"><text text-anchor="start" x="41" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">update_grammar()</text></a></g></a></g></a></g></g>

如果设置了`log`，则将中间步骤作为文本显示在标准输出上。《初始化()`</text></a></g> <g id="a_node2_9"><a xlink:href="GrammarFuzzer.html" xlink:title="check_grammar(self) -> None:

检查传递的语法。《检查语法()`</text></a></g> <g id="a_node2_10"><a xlink:href="GrammarFuzzer.html" xlink:title="choose_node_expansion(self, node: Tuple[str, Optional[List[Any]]], children_alternatives: List[List[Tuple[str, Optional[List[Any]]]]]) -> int:

返回`children_alternatives`中要选择的扩展索引。

`'children_alternatives'`: `node`的可能子节点列表。

默认为随机。在子类中重载。《选择节点扩展()`</text></a></g> <g id="a_node2_11"><a xlink:href="GrammarFuzzer.html" xlink:title="choose_tree_expansion(self, tree: Tuple[str, Optional[List[Any]]], children: List[Tuple[str, Optional[List[Any]]]]) -> int:

返回要扩展的子树在`children`中的索引。

默认为随机。《选择树扩展()`</text></a></g> <g id="a_node2_12"><a xlink:href="GrammarFuzzer.html" xlink:title="expand_node_randomly(self, node: Tuple[str, Optional[List[Any]]]) -> Tuple[str, Optional[List[Any]]]:

为`node`选择一个随机扩展并返回它。《随机扩展节点()`</text></a></g> <g id="a_node2_13"><a xlink:href="GrammarFuzzer.html" xlink:title="expand_tree(self, tree: Tuple[str, Optional[List[Any]]]) -> Tuple[str, Optional[List[Any]]]:

以三阶段策略扩展`tree`，直到所有扩展都完成。《扩展树()`</text></a></g> <g id="a_node2_14"><a xlink:href="GrammarFuzzer.html" xlink:title="expand_tree_once(self, tree: Tuple[str, Optional[List[Any]]]) -> Tuple[str, Optional[List[Any]]]:

在树中选择一个未扩展的符号；扩展它。

可在子类中重载。"><text text-anchor="start" x="8" y="-232.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">expand_tree_once()</text></a></g> <g id="a_node2_15"><a xlink:href="GrammarFuzzer.html" xlink:title="expand_tree_with_strategy(self, tree: Tuple[str, Optional[List[Any]]], expand_node_method: Callable, limit: Optional[int] = None):

使用`expand_node_method`作为节点扩展函数展开树

直到可能的扩展数量达到`limit`。"><text text-anchor="start" x="8" y="-220" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">expand_tree_with_strategy()</text></a></g> <g id="a_node2_16"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz(self) -> str:

从语法中生成一个字符串。"><text text-anchor="start" x="8" y="-207.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node2_17"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz_tree(self) -> Tuple[str, Optional[List[Any]]]:

从语法中生成一个推导树。"><text text-anchor="start" x="8" y="-194.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">fuzz_tree()</text></a></g> <g id="a_node2_18"><a xlink:href="GrammarFuzzer.html" xlink:title="log_tree(self, tree: Tuple[str, Optional[List[Any]]]) -> None:

如果 self.log 被设置，则输出一个树；如果 self.display 也被设置，则显示树结构"><text text-anchor="start" x="8" y="-181.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">log_tree()</text></a></g> <g id="a_node2_19"><a xlink:href="GrammarFuzzer.html" xlink:title="process_chosen_children(self, chosen_children: List[Tuple[str, Optional[List[Any]]]], expansion: Union[str, Tuple[str, Dict[str, Any]]]) -> List[Tuple[str, Optional[List[Any]]]]:

选择后处理子节点。默认情况下，不执行任何操作。"><text text-anchor="start" x="8" y="-169" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">process_chosen_children()</text></a></g> <g id="a_node2_20"><a xlink:href="GrammarFuzzer.html" xlink:title="supported_opts(self) -> Set[str]:

支持的选项集合。在子类中可重载。"><text text-anchor="start" x="8" y="-156.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">supported_opts()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>ConcolicGrammarFuzzer->GrammarFuzzer</title></g> <g id="node3" class="node"><title>Fuzzer</title> <g id="a_node3"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊器的基类。"><text text-anchor="start" x="68.38" y="-453.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Fuzzer</text> <g id="a_node3_21"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node3_22"><a xlink:href="Fuzzer.html" xlink:title="__init__(self) -> None:

构造函数"><text text-anchor="start" x="59" y="-431.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node3_23"><a xlink:href="Fuzzer.html" xlink:title="fuzz(self) -> str:

返回模糊输入"><text text-anchor="start" x="59" y="-418.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node3_24"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

使用模糊输入运行 `runner`"><text text-anchor="start" x="59" y="-405.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">run()</text></a></g> <g id="a_node3_25"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

使用模糊输入运行 `runner`，`trials` 次数"><text text-anchor="start" x="59" y="-393" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">runs()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>GrammarFuzzer->Fuzzer</title></g> <g id="node4" class="node"><title>图例</title> <text text-anchor="start" x="190.38" y="-71.75" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="190.38" y="-61.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="196.38" y="-61.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="190.38" y="-51.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="196.38" y="-51.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="190.38" y="-41.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="196.38" y="-41.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="190.38" y="-32.7" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 追踪约束

在关于信息流的章节中，我们看到了如何使用动态污点来通过指示输入的哪一部分到达了有趣的地方来指导模糊测试。然而，动态污点跟踪在它能够传播的信息方面是有限的。例如，我们可能想要探索当输入的某些属性发生变化时会发生什么。

例如，假设我们有一个返回输入的*阶乘值*的函数`factorial()`。

```py
def factorial(n):
    if n < 0:
        return None

    if n == 0:
        return 1

    if n == 1:
        return 1

    v = 1
    while n != 0:
        v = v * n
        n = n - 1

    return v 
```

我们用`5`的值来测试这个函数。

```py
factorial(5) 
```

```py
120

```

这是否足以探索函数的所有特性？我们如何知道？验证我们已经探索了所有特性的方法之一是查看*获得的覆盖范围*。首先，我们需要将`Coverage`类从关于覆盖的章节扩展，以提供覆盖弧。

```py
from Coverage import Coverage 
```

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
class ArcCoverage(Coverage):
    def traceit(self, frame, event, args):
        if event != 'return':
            f = inspect.getframeinfo(frame)
            self._trace.append((f.function, f.lineno))
        return self.traceit

    def arcs(self):
        t = [i for f, i in self._trace]
        return list(zip(t, t[1:])) 
```

接下来，我们使用`Tracer`来获取覆盖弧。

```py
with ArcCoverage() as cov:
    factorial(5) 
```

我们现在可以使用覆盖弧来可视化所获得的覆盖范围。

```py
from ControlFlow import to_graph, gen_cfg 
```

```py
to_graph(gen_cfg(inspect.getsource(factorial)), arcs=cov.arcs()) 
```

<svg width="418pt" height="564pt" viewBox="0.00 0.00 418.25 564.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 560)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="98.75" y="-527.83" font-family="Times,serif" font-size="14.00">1: enter: factorial(n)</text></g> <g id="node7" class="node"><title>3</title> <text text-anchor="middle" x="98.75" y="-451.82" font-family="Times,serif" font-size="14.00">2: if: n < 0</text></g> <g id="edge5" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="142.75" y="-15.82" font-family="Times,serif" font-size="14.00">1: exit: factorial(n)</text></g> <g id="node3" class="node"><title>4</title> <text text-anchor="middle" x="47.75" y="-163.82" font-family="Times,serif" font-size="14.00">3: return None</text></g> <g id="edge1" class="edge"><title>4->2</title></g> <g id="node4" class="node"><title>6</title> <text text-anchor="middle" x="123.75" y="-307.82" font-family="Times,serif" font-size="14.00">6: return 1</text></g> <g id="edge2" class="edge"><title>6->2</title></g> <g id="node5" class="node"><title>8</title> <text text-anchor="middle" x="193.75" y="-235.82" font-family="Times,serif" font-size="14.00">9: return 1</text></g> <g id="edge3" class="edge"><title>8->2</title></g> <g id="node6" class="node"><title>13</title> <text text-anchor="middle" x="229.75" y="-91.83" font-family="Times,serif" font-size="14.00">16: return v</text></g> <g id="edge4" class="edge"><title>13->2</title></g> <g id="edge6" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>5</title> <text text-anchor="middle" x="150.75" y="-379.82" font-family="Times,serif" font-size="14.00">5: if: n == 0</text></g> <g id="edge7" class="edge"><title>3->5</title></g> <g id="edge8" class="edge"><title>5->6</title></g> <g id="node9" class="node"><title>7</title> <text text-anchor="middle" x="253.75" y="-307.82" font-family="Times,serif" font-size="14.00">8: if: n == 1</text></g> <g id="edge9" class="edge"><title>5->7</title></g> <g id="edge10" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="283.75" y="-235.82" font-family="Times,serif" font-size="14.00">11: v = 1</text></g> <g id="edge11" class="edge"><title>7->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="308.75" y="-163.82" font-family="Times,serif" font-size="14.00">12: while: n != 0</text></g> <g id="edge12" class="edge"><title>9->10</title></g> <g id="edge16" class="edge"><title>10->13</title></g> <g id="node13" class="node"><title>11</title> <text text-anchor="middle" x="331.75" y="-91.83" font-family="Times,serif" font-size="14.00">13: v = v * n</text></g> <g id="edge14" class="edge"><title>10->11</title></g> <g id="node12" class="node"><title>12</title> <text text-anchor="middle" x="367.75" y="-15.82" font-family="Times,serif" font-size="14.00">14: n = n - 1</text></g> <g id="edge13" class="edge"><title>12->10</title></g> <g id="edge15" class="edge"><title>11->12</title></g></g></svg>

我们看到路径`[1, 2, 5, 8, 11, 12, 13, 14]`被覆盖（绿色），但子路径如`[2, 3]`、`[5, 6]`和`[8, 9]`未被探索（红色）。我们需要的是生成输入，使得在`2`处采纳`True`分支的能力。我们如何做到这一点？

## 符号执行

覆盖额外分支的一种方法是通过查看正在执行的路径，并收集路径遇到的*条件约束*。然后我们可以尝试生成引导我们采取未遍历路径的输入。

首先，让我们逐步分析这个函数。

```py
lines = [i[1] for i in cov._trace if i[0] == 'factorial']
src = {i + 1: s for i, s in enumerate(
    inspect.getsource(factorial).split('\n'))} 
```

+   行（1）仅仅是函数的入口点。我们知道输入是`n`，它是一个整数。

```py
src[1] 
```

```py
'def factorial(n):'

```

+   行（2）是一个断言`n < 0`。由于接下来采取的是行（5），我们知道在执行路径的这个点上，断言是`false`。

```py
src[2], src[3], src[4], src[5] 
```

```py
('    if n < 0:', '        return None', '', '    if n == 0:')

```

我们注意到，这是其中之一，`true`分支没有被采纳的断言。我们如何生成一个能够采纳`true`分支的值呢？一种方法是用符号变量来表示输入，编码约束，并使用*SMT 求解器*来解决约束的否定。

正如我们在章节引言中提到的，符号变量可以被视为真实变量的某种占位符，类似于在代数中求解`x`时的`x`。这些变量可以用来编码程序中变量的约束。我们确定变量应该遵守的约束，并最终产生一个遵守所有施加约束的值。

## 解决约束

要解决这些约束，可以使用 *理论可满足性 (SMT)* 求解器。SMT 求解器建立在 *可满足性 (SAT)* 求解器之上。SAT 求解器用于检查一阶逻辑中的布尔公式（例如 `(a | b ) & (~a | ~b)`）是否可以通过任何变量的赋值（例如 `a = true, b = false`）来满足。SMT 求解器将这些 SAT 求解器扩展到特定的背景理论 -- 例如，*整数理论* 或 *字符串理论*。也就是说，给定一个以字符串变量表示的公式（例如 `h + t == 'hello,world'`）作为字符串约束，一个理解 *字符串理论* 的 SMT 求解器可以用来检查该约束是否可以满足，如果可以满足，则提供公式中使用的变量的具体值实例（例如 `h = 'hello,', t = 'world'`）。

在本章中，我们使用 SMT 求解器 Z3。

```py
import [z3](https://github.com/Z3Prover/z3#readme) 
```

```py
z3_ver = z3.get_version() 
```

```py
print(z3_ver) 
```

```py
(4, 11, 2, 0)

```

```py
assert z3_ver >= (4, 8, 13, 0), \
    f"Please install z3-solver 4.8.13.0 or later - you have {z3_ver}" 
```

让我们先设置 Z3。为了确保本章中使用的字符串约束能够成功评估，我们需要指定 `z3str3` 求解器。此外，我们将 Z3 计算的超时时间设置为 30 秒。

```py
# z3.set_option('smt.string_solver', 'z3str3')
z3.set_option('timeout', 30 * 1000)  # milliseconds 
```

要编码约束，我们需要符号变量。在这里，我们将 `zn` 作为 Z3 符号整数变量 `n` 的占位符。

```py
zn = z3.Int('n') 
```

记得 `factorial()` 第 2 行的约束 `(n < 0)` 吗？我们现在可以将这个约束编码如下。

```py
zn < 0 
```

n < 0

我们之前追踪了 `factorial(5)` 的执行过程。我们注意到，当输入为 `5` 时，执行流程进入了谓词 `n < 0` 的 `else` 分支。我们可以这样表达这个观察结果。

```py
z3.Not(zn < 0) 
```

¬(n < 0)

现在我们来解决约束。`z3.solve()` 方法检查约束是否可满足；如果是，它还提供变量的值，使得约束得到满足。例如，我们可以要求 Z3 提供一个输入，使其执行时进入 `else` 分支，如下所示：

```py
z3.solve(z3.Not(zn < 0)) 
```

```py
[n = 0]

```

这是一个 *解决方案*（尽管是一个平凡的解决方案）。SMT 求解器可以用来解决更难的问题。例如，下面是如何解一个二次方程的。

```py
x = z3.Real('x')
eqn = (2 * x**2 - 11 * x + 5 == 0)
z3.solve(eqn) 
```

```py
[x = 5]

```

再次，这是一个 *解决方案*。我们可以要求 z3 给我们另一个解决方案，如下所示。

```py
z3.solve(x != 5, eqn) 
```

```py
[x = 1/2]

```

事实上，`x = 5` 和 `x = 1/2` 都是二次方程 $2x² -11x + 5 = 0$ 的解

同样，我们可以要求 *Z3* 提供一个输入，使其满足 `factorial()` 第 2 行中编码的约束，从而进入 `if` 分支。

```py
z3.solve(zn < 0) 
```

```py
[n = -1]

```

那就是如果将 `-1` 作为 `factorial()` 的输入，它保证在执行过程中进入第 2 行的 `if` 分支。

让我们尝试使用这个方法来提高我们的覆盖率。在这里，`-1` 是上面的解决方案。

```py
with cov as cov:
    factorial(-1) 
```

```py
to_graph(gen_cfg(inspect.getsource(factorial)), arcs=cov.arcs()) 
```

<svg width="418pt" height="564pt" viewBox="0.00 0.00 418.25 564.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 560)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="98.75" y="-527.83" font-family="Times,serif" font-size="14.00">1: enter: factorial(n)</text></g> <g id="node7" class="node"><title>3</title> <text text-anchor="middle" x="98.75" y="-451.82" font-family="Times,serif" font-size="14.00">2: if: n < 0</text></g> <g id="edge5" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="142.75" y="-15.82" font-family="Times,serif" font-size="14.00">1: exit: factorial(n)</text></g> <g id="node3" class="node"><title>4</title> <text text-anchor="middle" x="47.75" y="-163.82" font-family="Times,serif" font-size="14.00">3: return None</text></g> <g id="edge1" class="edge"><title>4->2</title></g> <g id="node4" class="node"><title>6</title> <text text-anchor="middle" x="123.75" y="-307.82" font-family="Times,serif" font-size="14.00">6: return 1</text></g> <g id="edge2" class="edge"><title>6->2</title></g> <g id="node5" class="node"><title>8</title> <text text-anchor="middle" x="193.75" y="-235.82" font-family="Times,serif" font-size="14.00">9: return 1</text></g> <g id="edge3" class="edge"><title>8->2</title></g> <g id="node6" class="node"><title>13</title> <text text-anchor="middle" x="229.75" y="-91.83" font-family="Times,serif" font-size="14.00">16: return v</text></g> <g id="edge4" class="edge"><title>13->2</title></g> <g id="edge6" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>5</title> <text text-anchor="middle" x="150.75" y="-379.82" font-family="Times,serif" font-size="14.00">5: if: n == 0</text></g> <g id="edge7" class="edge"><title>3->5</title></g> <g id="edge8" class="edge"><title>5->6</title></g> <g id="node9" class="node"><title>7</title> <text text-anchor="middle" x="253.75" y="-307.82" font-family="Times,serif" font-size="14.00">8: if: n == 1</text></g> <g id="edge9" class="edge"><title>5->7</title></g> <g id="edge10" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="283.75" y="-235.82" font-family="Times,serif" font-size="14.00">11: v = 1</text></g> <g id="edge11" class="edge"><title>7->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="308.75" y="-163.82" font-family="Times,serif" font-size="14.00">12: while: n != 0</text></g> <g id="edge12" class="edge"><title>9->10</title></g> <g id="edge16" class="edge"><title>10->13</title></g> <g id="node13" class="node"><title>11</title> <text text-anchor="middle" x="331.75" y="-91.83" font-family="Times,serif" font-size="14.00">13: v = v * n</text></g> <g id="edge14" class="edge"><title>10->11</title></g> <g id="node12" class="node"><title>12</title> <text text-anchor="middle" x="367.75" y="-15.82" font-family="Times,serif" font-size="14.00">14: n = n - 1</text></g> <g id="edge13" class="edge"><title>12->10</title></g> <g id="edge15" class="edge"><title>11->12</title></g></g></svg>

好的，所以我们已经覆盖了图中的更多部分。让我们继续使用原始输入 `factorial(5)`：

+   在第 (5) 行，我们遇到了一个新的谓词 `n == 0`，对于这个谓词，我们又选择了 `false` 分支。

```py
src[5] 
```

```py
'    if n == 0:'

```

需要的谓词，以跟随路径到这一点如下。

```py
predicates = [z3.Not(zn < 0), z3.Not(zn == 0)] 
```

+   如果我们继续到第 (8) 行，我们会遇到另一个谓词，对于这个谓词，我们又选择了 `false` 分支

```py
src[8] 
```

```py
'    if n == 1:'

```

到目前为止遇到的谓词如下

```py
predicates = [z3.Not(zn < 0), z3.Not(zn == 0), z3.Not(zn == 1)] 
```

要选择（6）处的分支，我们本质上必须遵守直到那个点的谓词，但反转最后一个谓词。

```py
last = len(predicates) - 1
z3.solve(predicates[0:-1] + [z3.Not(predicates[-1])]) 
```

```py
[n = 1]

```

我们在这里所做的是跟踪对应于特定输入`factorial(5)`的执行，使用具体值，并与之一起，保持*符号阴影变量*，使我们能够捕获约束。正如我们在引言中提到的，这种通过使用符号变量跟踪具体执行的方法被称为*符号化执行*。

我们如何自动化这个过程呢？一种方法是使用与信息流章节中类似的基础设施，并使用 Python 继承来创建可以跟踪具体执行的*符号代理对象*。

## 符号化跟踪器

现在我们定义一个类来在执行过程中*收集*符号变量和路径条件。想法是有一个在`with`块中调用的`ConcolicTracer`类。为了在跟踪其路径条件的同时执行一个函数，我们需要*转换*其参数，我们通过通过`[]`项访问调用函数来实现。

这是一个典型的`ConcolicTracer`用法：

```py
with ConcolicTracer as _:
    _.function 
```

执行后，我们可以通过`decls`属性访问符号变量：

```py
_.decls 
```

而`path`属性列出了遇到的先决条件路径：

```py
_.path 
```

`context`属性包含一对声明和路径：

```py
_.context 
```

如果你第一次阅读这篇文章，请跳过实现部分，直接查看示例。

<details id="Excursion:-Implementing-ConcolicTracer"><summary>实现 ConcolicTracer</summary>

现在我们来实现`ConcolicTracer`。它的构造函数接受一个`context`参数，该参数包含迄今为止看到的符号变量的声明和路径条件。我们只需要在嵌套上下文中使用它。

```py
class ConcolicTracer:
  """Trace function execution, tracking variables and path conditions"""

    def __init__(self, context=None):
  """Constructor."""
        self.context = context if context is not None else ({}, [])
        self.decls, self.path = self.context 
```

我们为`with`块添加了进入和退出方法。

```py
class ConcolicTracer(ConcolicTracer):
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, tb):
        return 
```

我们使用内省来确定函数的参数，该参数被钩入`getitem()`方法。

```py
class ConcolicTracer(ConcolicTracer):
    def __getitem__(self, fn):
        self.fn = fn
        self.fn_args = {i: None for i in inspect.signature(fn).parameters}
        return self 
```

最后，使用`call`方法调用函数本身。

```py
class ConcolicTracer(ConcolicTracer):
    def __call__(self, *args):
        self.result = self.fn(*self.concolic(args))
        return self.result 
```

现在，我们将`concolic()`定义为透明的函数。稍后它将被修改以产生符号变量。

```py
class ConcolicTracer(ConcolicTracer):
    def concolic(self, args):
        return args 
```

我们现在已经为*跟踪*函数准备好了：

```py
with ConcolicTracer() as _:
    _factorial 
```

以及用于检索结果（但实际上并不*计算*它们）：

```py
_.decls 
```

```py
{}

```

```py
_.path 
```

```py
[]

```

`decls`和`path`属性将由我们接下来定义的符号化代理对象设置。

#### 符号化代理对象

现在我们定义可以用于符号化跟踪的符号化代理对象。首先，我们定义`zproxy_create()`方法，它给定一个类名，正确地创建该类的实例和相应的符号变量，并在上下文信息`context`中注册符号变量。

```py
def zproxy_create(cls, z_type, z3var, context, z_name, v=None):
    z_value = cls(context, z3var(z_name), v)
    context[0][z_name] = z_type  # add to decls
    return z_value 
```

#### 布尔代理类

首先，我们定义用于跟踪遇到的谓词的 `zbool` 类。它是一个包装类，包含符号（`z`）和具体（`v`）值。具体值用于确定要采取的路径，符号值用于收集遇到的谓词。

初始化分为两部分进行。第一部分是使用 `zproxy_create()` 正确初始化和注册与传递的参数对应的阴影符号变量。这仅在符号变量需要首先初始化时使用。在其他所有情况下，使用现有的符号值调用构造函数。

```py
class zbool:
    @classmethod
    def create(cls, context, z_name, v):
        return zproxy_create(cls, 'Bool', z3.Bool, context, z_name, v)

    def __init__(self, context, z, v=None):
        self.context = context
        self.z = z
        self.v = v
        self.decl, self.path = self.context 
```

这里是如何使用的。我们在 concolic tracer 的当前上下文中创建一个具有 `True` 值的符号变量 `my_bool_arg`：

```py
with ConcolicTracer() as _:
    val = zbool.create(_.context, 'my_bool_arg', True) 
```

我们现在可以访问 `z` 属性中的符号名称：

```py
val.z 
```

my_bool_arg

值位于 `v` 属性中：

```py
val.v 
```

```py
True

```

注意，封装的 `ConcolicTracer()` 的上下文会自动更新（通过 `zproxy_create()`），以包含变量声明和类型：

```py
_.context 
```

```py
({'my_bool_arg': 'Bool'}, [])

```

上下文也可以通过 `context` 属性访问；两者指向相同的数据结构。

```py
val.context 
```

```py
({'my_bool_arg': 'Bool'}, [])

```

##### 编码公式的否定

`zbool` 类允许对其具体和符号值进行否定。

```py
class zbool(zbool):
    def __not__(self):
        return zbool(self.context, z3.Not(self.z), not self.v) 
```

这里是如何使用它的。

```py
with ConcolicTracer() as _:
    val = zbool.create(_.context, 'my_bool_arg', True).__not__() 
```

```py
val.z 
```

¬my_bool_arg

```py
val.v 
```

```py
False

```

```py
_.context 
```

```py
({'my_bool_arg': 'Bool'}, [])

```

##### 在条件上注册谓词

`zbool` 类正在被用来跟踪程序执行过程中出现的布尔条件。它通过在评估时立即在上下文中注册相应的符号表达式来跟踪这些条件。在评估时，会调用 `__bool__()` 方法；因此，我们可以挂钩到这个方法：

```py
class zbool(zbool):
    def __bool__(self):
        r, pred = (True, self.z) if self.v else (False, z3.Not(self.z))
        self.path.append(pred)
        return r 
```

`zbool` 类可以用来跟踪执行过程中遇到的布尔值和条件。例如，我们可以将 `factorial()` 中第 6 行遇到的条件编码如下：

首先，我们定义具体的值（`ca`）及其阴影符号变量（`za`）。

```py
ca = 5
za = z3.Int('a') 
```

然后，我们将其包装在 `zbool` 中，并在条件中使用它，强制条件在上下文中注册。

```py
with ConcolicTracer() as _:
    if zbool(_.context, za == z3.IntVal(5), ca == 5):
        print('success') 
```

```py
success

```

我们可以按照以下方式检索已注册的条件。

```py
_.path 
```

```py
[5 == a]

```

#### 整数的代理类

接下来，我们定义一个用于 `int` 的符号包装器 `zint`。这个类跟踪在 `context` 中使用的 `int` 变量和遇到的谓词。最后，它还保留具体值，以便可以用来确定要采取的路径。由于 `zint` 扩展了原始的 `int` 类，我们必须定义一个 *新* 方法来允许其扩展。

```py
class zint(int):
    def __new__(cls, context, zn, v, *args, **kw):
        return int.__new__(cls, v, *args, **kw) 
```

与 `zbool` 的情况一样，初始化也分为两部分进行。第一部分是如果正在注册新的符号参数，则使用 `create()`，然后进行常规初始化。

```py
class zint(zint):
    @classmethod
    def create(cls, context, zn, v=None):
        return zproxy_create(cls, 'Int', z3.Int, context, zn, v)

    def __init__(self, context, z, v=None):
        self.z, self.v = z, v
        self.context = context 
```

`zint` 对象的 `int` 值是其具体值。

```py
class zint(zint):
    def __int__(self):
        return self.v

    def __pos__(self):
        return self.v 
```

使用这些代理的方式如下。

```py
with ConcolicTracer() as _:
    val = zint.create(_.context, 'int_arg', 0) 
```

```py
val.z 
```

int_arg

```py
val.v 
```

```py
0

```

```py
_.context 
```

```py
({'int_arg': 'Int'}, [])

```

`zint` 类通常用于与其他 `int` 进行算术运算或比较。这些 `int` 可以是一个变量或一个常量值。我们定义了一个辅助方法 `_zv()`，用于检查给定值是哪种 `int`，并生成正确的符号等价物。

```py
class zint(zint):
    def _zv(self, o):
        return (o.z, o.v) if isinstance(o, zint) else (z3.IntVal(o), o) 
```

它可以用以下方式使用

```py
with ConcolicTracer() as _:
    val = zint.create(_.context, 'int_arg', 0) 
```

```py
val._zv(0) 
```

```py
(0, 0)

```

```py
val._zv(val) 
```

```py
(int_arg, 0)

```

##### 整数之间的等价

两个整数可以使用 `ne` 和 `eq` 进行等价比较。

```py
class zint(zint):
    def __ne__(self, other):
        z, v = self._zv(other)
        return zbool(self.context, self.z != z, self.v != v)

    def __eq__(self, other):
        z, v = self._zv(other)
        return zbool(self.context, self.z == z, self.v == v) 
```

我们还使用 `eq` 定义了 `*req*`，以防比较的整数在左侧。

```py
class zint(zint):
    def __req__(self, other):
        return self.__eq__(other) 
```

它可以用以下方式使用。

```py
with ConcolicTracer() as _:
    ia = zint.create(_.context, 'int_a', 0)
    ib = zint.create(_.context, 'int_b', 0)
    v1 = ia == ib
    v2 = ia != ib
    v3 = 0 != ib
    print(v1.z, v2.z, v3.z) 
```

```py
int_a == int_b int_a != int_b 0 != int_b

```

##### 整数之间的比较

整数也可以用于比较顺序，相关的方法定义如下。

```py
class zint(zint):
    def __lt__(self, other):
        z, v = self._zv(other)
        return zbool(self.context, self.z < z, self.v < v)

    def __gt__(self, other):
        z, v = self._zv(other)
        return zbool(self.context, self.z > z, self.v > v) 
```

我们使用比较和等价运算符来提供其他缺失的运算符。

```py
class zint(zint):
    def __le__(self, other):
        z, v = self._zv(other)
        return zbool(self.context, z3.Or(self.z < z, self.z == z),
                     self.v < v or self.v == v)

    def __ge__(self, other):
        z, v = self._zv(other)
        return zbool(self.context, z3.Or(self.z > z, self.z == z),
                     self.v > v or self.v == v) 
```

这些函数可以用以下方式使用。

```py
with ConcolicTracer() as _:
    ia = zint.create(_.context, 'int_a', 0)
    ib = zint.create(_.context, 'int_b', 1)
    v1 = ia > ib
    v2 = ia < ib
    print(v1.z, v2.z)
    v3 = ia >= ib
    v4 = ia <= ib
    print(v3.z, v4.z) 
```

```py
int_a > int_b int_a < int_b
Or(int_a > int_b, int_a == int_b) Or(int_a < int_b, int_a == int_b)

```

##### 整数的二元运算符

我们实现了整数的相关算术运算符，如 [Python 文档](https://docs.python.org/3/reference/datamodel.html#object.__add__) 中所述。（注释掉的运算符对于 `z3.ArithRef` 不是直接可用的。如果需要，它们需要单独实现。参见练习，了解如何实现。）

```py
INT_BINARY_OPS = [
    '__add__',
    '__sub__',
    '__mul__',
    '__truediv__',
    # '__div__',
    '__mod__',
    # '__divmod__',
    '__pow__',
    # '__lshift__',
    # '__rshift__',
    # '__and__',
    # '__xor__',
    # '__or__',
    '__radd__',
    '__rsub__',
    '__rmul__',
    '__rtruediv__',
    # '__rdiv__',
    '__rmod__',
    # '__rdivmod__',
    '__rpow__',
    # '__rlshift__',
    # '__rrshift__',
    # '__rand__',
    # '__rxor__',
    # '__ror__',
] 
```

```py
def make_int_binary_wrapper(fname, fun, zfun):
    def proxy(self, other):
        z, v = self._zv(other)
        z_ = zfun(self.z, z)
        v_ = fun(self.v, v)
        if isinstance(v_, float):
            # we do not implement float results yet.
            assert round(v_) == v_
            v_ = round(v_)
        return zint(self.context, z_, v_)

    return proxy 
```

```py
INITIALIZER_LIST: List[Callable] = [] 
```

```py
def initialize():
    for fn in INITIALIZER_LIST:
        fn() 
```

```py
def init_concolic_1():
    for fname in INT_BINARY_OPS:
        fun = getattr(int, fname)
        zfun = getattr(z3.ArithRef, fname)
        setattr(zint, fname, make_int_binary_wrapper(fname, fun, zfun)) 
```

```py
INITIALIZER_LIST.append(init_concolic_1) 
```

```py
init_concolic_1() 
```

```py
with ConcolicTracer() as _:
    ia = zint.create(_.context, 'int_a', 0)
    ib = zint.create(_.context, 'int_b', 1)
    print((ia + ib).z)
    print((ia + 10).z)
    print((11 + ib).z)
    print((ia - ib).z)
    print((ia * ib).z)
    print((ia / ib).z)
    print((ia ** ib).z) 
```

```py
int_a + int_b
int_a + 10
11 + int_b
int_a - int_b
int_a*int_b
int_a/int_b
int_a**int_b

```

##### 整数一元运算符

我们还实现了以下相关的一元运算符。

```py
INT_UNARY_OPS = [
    '__neg__',
    '__pos__',
    # '__abs__',
    # '__invert__',
    # '__round__',
    # '__ceil__',
    # '__floor__',
    # '__trunc__',
] 
```

```py
def make_int_unary_wrapper(fname, fun, zfun):
    def proxy(self):
        return zint(self.context, zfun(self.z), fun(self.v))

    return proxy 
```

```py
def init_concolic_2():
    for fname in INT_UNARY_OPS:
        fun = getattr(int, fname)
        zfun = getattr(z3.ArithRef, fname)
        setattr(zint, fname, make_int_unary_wrapper(fname, fun, zfun)) 
```

```py
INITIALIZER_LIST.append(init_concolic_2) 
```

```py
init_concolic_2() 
```

我们可以使用上面定义的一元运算符如下：

```py
with ConcolicTracer() as _:
    ia = zint.create(_.context, 'int_a', 0)
    print((-ia).z)
    print((+ia).z) 
```

```py
-int_a
int_a

```

##### 在布尔上下文中使用整数

整数可以在条件语句或作为布尔谓词（如 `or`、`and` 和 `not`）的一部分转换为布尔上下文。在这些情况下，会调用 `__bool__()` 方法。不幸的是，此方法需要一个原始布尔值。因此，我们将当前整数公式强制转换为布尔谓词，并在当前上下文中注册它。

```py
class zint(zint):
    def __bool__(self):
        # return zbool(self.context, self.z, self.v) <-- not allowed
        # force registering boolean condition
        if self != 0:
            return True
        return False 
```

它的使用如下

```py
with ConcolicTracer() as _:
    za = zint.create(_.context, 'int_a', 1)
    zb = zint.create(_.context, 'int_b', 0)
    if za and zb:
        print(1) 
```

```py
_.context 
```

```py
({'int_a': 'Int', 'int_b': 'Int'}, [0 != int_a, Not(0 != int_b)])

```

#### ConcolicTracer 的剩余方法

我们现在完成 `ConcolicTracer` 的某些方法。

##### 转换为 SMT 表达式格式

由于我们使用的是 SMT 求解器 z3，检索符号表达式的对应 SMT 表达式通常很有用。这可以用作 `z3` 或其他 SMT 求解器的参数。

SMT 表达式的格式（[SMT-LIB](http://smtlib.github.io/jSMTLIB/SMTLIBTutorial.pdf)）如下：

+   [S-EXP](https://en.wikipedia.org/wiki/S-expression) 格式的变量声明。例如，以下声明了一个符号整数变量 `x`

    ```py
    (declare-const x Int)
    ```

    这声明了一个长度为 `8` 的 `bit vector` `b`

    ```py
    (declare-const b (_ BitVec 8))
    ```

    这声明了一个符号实变量 `r`

    ```py
    (declare-const x Real)
    ```

    这声明了一个符号字符串变量 `s`

    ```py
    (declare-const s String)
    ```

声明的变量可以用于编码在 *S-EXP* 格式的逻辑公式中。例如，这里是一个逻辑公式。

```py
(assert
    (and
        (= a b)
        (= a c)
        (! b c)))
```

这里是另一个例子，使用字符串变量。

```py
(or (< 0 (str.indexof (str.substr my_str1 7 19) " where " 0))
    (= (str.indexof (str.substr my_str1 7 19) " where " 0) 0))
```

```py
class ConcolicTracer(ConcolicTracer):
    def smt_expr(self, show_decl=False, simplify=False, path=[]):
        r = []
        if show_decl:
            for decl in self.decls:
                v = self.decls[decl]
                v = '(_ BitVec 8)' if v == 'BitVec' else v
                r.append("(declare-const %s  %s)" % (decl, v))
        path = path if path else self.path
        if path:
            path = z3.And(path)
            if show_decl:
                if simplify:
                    return '\n'.join([
                        *r,
                        "(assert %s)" % z3.simplify(path).sexpr()
                    ])
                else:
                    return '\n'.join(
                        [*r, "(assert %s)" % path.sexpr()])
            else:
                return z3.simplify(path).sexpr()
        else:
            return '' 
```

要了解如何使用 `smt_expr()`，让我们考虑一个例子。`triangle()` 函数用于确定给定的三角形边长是否构成一个 `equilateral` 三角形、一个 `isosceles` 三角形或一个 `scalene` 三角形。它的实现如下。

```py
def triangle(a, b, c):
    if a == b:
        if b == c:
            return 'equilateral'
        else:
            return 'isosceles'
    else:
        if b == c:
            return 'isosceles'
        else:
            if a == c:
                return 'isosceles'
            else:
                return 'scalene' 
```

```py
triangle(1, 2, 1) 
```

```py
'isosceles'

```

要使 `triangle()` 在 `ConcolicTracer` 下运行，我们首先定义（符号）参数。定义的三角形边长为 `1, 1, 1`，即它是一个 `equilateral` 三角形。

```py
with ConcolicTracer() as _:
    za = zint.create(_.context, 'int_a', 1)
    zb = zint.create(_.context, 'int_b', 1)
    zc = zint.create(_.context, 'int_c', 1)
    triangle(za, zb, zc)
print(_.context) 
```

```py
({'int_a': 'Int', 'int_b': 'Int', 'int_c': 'Int'}, [int_a == int_b, int_b == int_c])

```

现在我们可以调用 `smt_expr()` 来检索以下 SMT 表达式。

```py
print(_.smt_expr(show_decl=True)) 
```

```py
(declare-const int_a Int)
(declare-const int_b Int)
(declare-const int_c Int)
(assert (and (= int_a int_b) (= int_b int_c)))

```

收集到的谓词也可以直接使用 Python z3 API 解决。

```py
z3.solve(_.path) 
```

```py
[int_c = 0, int_a = 0, int_b = 0]

```

##### 生成新名称

在使用代理类时，我们通常需要生成新的符号变量，这些变量的名称之前未被使用过。为此，我们定义了 `fresh_name()`，它总是为名称生成唯一的整数。

```py
COUNTER = 0 
```

```py
def fresh_name():
    global COUNTER
    COUNTER += 1
    return COUNTER 
```

它可以这样使用：

```py
fresh_name() 
```

```py
1

```

```py
def reset_counter():
    global COUNTER
    COUNTER = 0 
```

```py
class ConcolicTracer(ConcolicTracer):
    def __enter__(self):
        reset_counter()
        return self

    def __exit__(self, exc_type, exc_value, tb):
        return 
```

##### 将参数翻译为 Concolic 代理

我们之前已将 `concolic()` 定义为一个透明函数。现在我们提供这个函数的完整实现。它检查给定函数的参数，并从传递的具体参数中推断参数类型。然后它使用这些信息为每个参数实例化正确的代理类。

```py
class ConcolicTracer(ConcolicTracer):
    def concolic(self, args):
        my_args = []
        for name, arg in zip(self.fn_args, args):
            t = type(arg).__name__
            zwrap = globals()['z' + t]
            vname = "%s_%s_%s_%s" % (self.fn.__name__, name, t, fresh_name())
            my_args.append(zwrap.create(self.context, vname, arg))
            self.fn_args[name] = vname
        return my_args 
```

这是它的使用方法：

```py
with ConcolicTracer() as _:
    _factorial 
```

使用新的 `concolic()` 方法，阶乘的参数被正确地与符号变量关联，这允许我们检索遇到的谓词。

```py
_.context 
```

```py
({'factorial_n_int_1': 'Int'},
 [Not(0 > factorial_n_int_1),
  Not(0 == factorial_n_int_1),
  Not(1 == factorial_n_int_1),
  0 != factorial_n_int_1,
  0 != factorial_n_int_1 - 1,
  0 != factorial_n_int_1 - 1 - 1,
  0 != factorial_n_int_1 - 1 - 1 - 1,
  0 != factorial_n_int_1 - 1 - 1 - 1 - 1,
  Not(0 != factorial_n_int_1 - 1 - 1 - 1 - 1 - 1)])

```

如前所述，我们还可以打印出可以直接传递给命令行 SMT 求解器的 SMT 表达式。

```py
print(_.smt_expr(show_decl=True)) 
```

```py
(declare-const factorial_n_int_1 Int)
(assert (let ((a!1 (distinct 0 (- (- (- factorial_n_int_1 1) 1) 1)))
      (a!2 (- (- (- (- factorial_n_int_1 1) 1) 1) 1)))
  (and (not (> 0 factorial_n_int_1))
       (not (= 0 factorial_n_int_1))
       (not (= 1 factorial_n_int_1))
       (distinct 0 factorial_n_int_1)
       (distinct 0 (- factorial_n_int_1 1))
       (distinct 0 (- (- factorial_n_int_1 1) 1))
       a!1
       (distinct 0 a!2)
       (not (distinct 0 (- a!2 1))))))

```

接下来，我们定义了在 Python 和命令行中评估 SMT 表达式的方法。

##### 评估 Concolic 表达式

我们定义 `zeval()` 来在上下文中解决谓词，并返回结果。它有两种模式。`python` 模式使用 `z3` Python API 解决并返回结果。如果 `python` 模式为假，则将 SMT 表达式写入文件，并调用命令行 `z3` 进行求解。

```py
class ConcolicTracer(ConcolicTracer):
    def zeval(self, predicates=None, *,python=False, log=False):
  """Evaluate `predicates` in current context.
 - If `python` is set, use the z3 Python API; otherwise use z3 standalone.
 - If `log` is set, show input to z3.
 Return a pair (`result`, `solution`) where
 - `result` is either `'sat'` (satisfiable); then 
 solution` is a mapping of variables to (value, type) pairs; or
 - `result` is not `'sat'`, indicating an error; then `solution` is `None`
 """
        if predicates is None:
            path = self.path
        else:
            path = list(self.path)
            for i in sorted(predicates):
                if len(path) > i:
                    path[i] = predicates[i]
                else:
                    path.append(predicates[i])
        if log:
            print('Predicates in path:')
            for i, p in enumerate(path):
                print(i, p)
            print()

        r, sol = (zeval_py if python else zeval_smt)(path, self, log)
        if r == 'sat':
            return r, {k: sol.get(self.fn_args[k], None) for k in self.fn_args}
        else:
            return r, None 
```

##### 使用 Python API

给定函数遇到的谓词集以及函数执行的跟踪器，`zeval_py()` 函数首先声明相关的符号变量，并使用 `z3.Solver()` 提供一组输入，这些输入会在函数中跟踪相同的路径。

```py
def zeval_py(path, cc, log):
    for decl in cc.decls:
        if cc.decls[decl] == 'BitVec':
            v = "z3.%s('%s', 8)" % (cc.decls[decl], decl)
        else:
            v = "z3.%s('%s')" % (cc.decls[decl], decl)
        exec(v)
    s = z3.Solver()
    s.add(z3.And(path))
    if s.check() == z3.unsat:
        return 'No Solutions', {}
    elif s.check() == z3.unknown:
        return 'Gave up', None
    assert s.check() == z3.sat
    m = s.model()
    return 'sat', {d.name(): m[d] for d in m.decls()} 
```

它可以这样使用：

```py
with ConcolicTracer() as _:
    _factorial 
```

```py
_.zeval(python=True) 
```

```py
('sat', {'n': 5})

```

即，给定约束集，赋值 `n == 5` 符合所有约束。

##### 使用命令行

`zeval_smt()` 函数将 SMT 表达式写入文件系统，并调用 `z3` SMT 求解器的命令行来解决问题。SMT 表达式的结果是另一个 `sexpr`。因此，我们首先定义 `parse_sexp()` 来解析并返回正确的值。

```py
import [re](https://docs.python.org/3/library/re.html) 
```

```py
import [subprocess](https://docs.python.org/3/library/subprocess.html) 
```

```py
SEXPR_TOKEN = r'''(?mx)
 \s*(?:
 (?P<bra>\()|
 (?P<ket>\))|
 (?P<token>[^"()\s]+)|
 (?P<string>"[^"]*")
 )''' 
```

```py
def parse_sexp(sexp):
    stack, res = [], []
    for elements in re.finditer(SEXPR_TOKEN, sexp):
        kind, value = [(t, v) for t, v in elements.groupdict().items() if v][0]
        if kind == 'bra':
            stack.append(res)
            res = []
        elif kind == 'ket':
            last, res = res, stack.pop(-1)
            res.append(last)
        elif kind == 'token':
            res.append(value)
        elif kind == 'string':
            res.append(value[1:-1])
        else:
            assert False
    return res 
```

可以这样使用 `parse_sexp()` 函数

```py
parse_sexp('abcd (hello 123 (world "hello world"))') 
```

```py
['abcd', ['hello', '123', ['world', 'hello world']]]

```

现在我们定义 `zeval_smt()`，它直接使用 `z3` 命令行，并使用 `parse_sexp()` 解析并返回函数参数的解决方案（如果有的话）。

```py
import [tempfile](https://docs.python.org/3/library/tempfile.html)
import [os](https://docs.python.org/3/library/os.html) 
```

```py
Z3_BINARY = 'z3'  # Z3 binary to invoke 
```

```py
Z3_OPTIONS = '-t:6000'  # Z3 options - a soft timeout of 6000 milliseconds 
```

```py
def zeval_smt(path, cc, log):
    s = cc.smt_expr(True, True, path)

    with tempfile.NamedTemporaryFile(mode='w', suffix='.smt',
                                     delete=False) as f:
        f.write(s + "\n")
        f.write("(check-sat)\n")
        f.write("(get-model)\n")

    if log:
        print(open(f.name).read())

    cmd = f"{Z3_BINARY}  {Z3_OPTIONS}  {f.name}"
    if log:
        print(cmd)

    output = subprocess.getoutput(cmd)

    os.remove(f.name)

    if log:
        print(output)

    o = parse_sexp(output)
    if not o:
        return 'Gave up', None

    kind = o[0]
    if kind == 'unknown':
        return 'Gave up', None
    elif kind == 'timeout':
        return 'Timeout', None
    elif kind == 'unsat':
        return 'No Solutions', {}

    assert kind == 'sat', kind
    if o[1][0] == 'model': # up to 4.8.8.0
        return 'sat', {i[1]: (i[-1], i[-2]) for i in o[1][1:]}
    else:
        return 'sat', {i[1]: (i[-1], i[-2]) for i in o[1][0:]} 
```

现在我们可以这样使用 `zeval()`。

```py
with ConcolicTracer() as _:
    _factorial 
```

```py
_.zeval(log=True) 
```

```py
Predicates in path:
0 Not(0 > factorial_n_int_1)
1 Not(0 == factorial_n_int_1)
2 Not(1 == factorial_n_int_1)
3 0 != factorial_n_int_1
4 0 != factorial_n_int_1 - 1
5 0 != factorial_n_int_1 - 1 - 1
6 0 != factorial_n_int_1 - 1 - 1 - 1
7 0 != factorial_n_int_1 - 1 - 1 - 1 - 1
8 Not(0 != factorial_n_int_1 - 1 - 1 - 1 - 1 - 1)

(declare-const factorial_n_int_1 Int)
(assert (and (<= 0 factorial_n_int_1)
     (not (= 0 factorial_n_int_1))
     (not (= 1 factorial_n_int_1))
     (not (= 2 factorial_n_int_1))
     (not (= 3 factorial_n_int_1))
     (not (= 4 factorial_n_int_1))
     (= 5 factorial_n_int_1)))
(check-sat)
(get-model)

z3 -t:6000 /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmp9aqp4ftz.smt
sat
(
  (define-fun factorial_n_int_1 () Int
    5)
)

```

```py
('sat', {'n': ('5', 'Int')})

```

事实上，无论是使用命令行还是使用 Python API，我们都得到了类似的结果 (`n == 5`)。

#### 字符串代理类

这里，我们定义了代理字符串类 `zstr`。首先我们定义我们的初始化例程。由于 `str` 是一个原始类型，我们定义 `new` 来扩展它。

```py
class zstr(str):
    def __new__(cls, context, zn, v):
        return str.__new__(cls, v) 
```

如前所述，初始化通过 `create()` 和构造函数进行。

```py
class zstr(zstr):
    @classmethod
    def create(cls, context, zn, v=None):
        return zproxy_create(cls, 'String', z3.String, context, zn, v)

    def __init__(self, context, z, v=None):
        self.context, self.z, self.v = context, z, v
        self._len = zint(context, z3.Length(z), len(v))
        #self.context[1].append(z3.Length(z) == z3.IntVal(len(v))) 
```

我们还定义了 `_zv()` 辅助函数，以帮助我们在接受另一个字符串的方法中使用。

```py
class zstr(zstr):
    def _zv(self, o):
        return (o.z, o.v) if isinstance(o, zstr) else (z3.StringVal(o), o) 
```

### 使用字符的 ASCII 值的技巧。

**注意：** 暂时解决方案；这个块应该在 [这个提交](https://github.com/Z3Prover/z3/issues/5764) 发布后消失，这允许我们直接使用 Python API。

```py
from [typing](https://docs.python.org/3/library/typing.html) import Union, Optional, Dict, Generator, Set

def visit_z3_expr(
        e: Union[z3.ExprRef, z3.QuantifierRef],
        seen: Optional[Dict[Union[z3.ExprRef, z3.QuantifierRef], bool]] = None) -> \
        Generator[Union[z3.ExprRef, z3.QuantifierRef], None, None]:
    if seen is None:
        seen = {}
    elif e in seen:
        return

    seen[e] = True
    yield e

    if z3.is_app(e):
        for ch in e.children():
            for e in visit_z3_expr(ch, seen):
                yield e
        return

    if z3.is_quantifier(e):
        for e in visit_z3_expr(e.body(), seen):
            yield e
        return

def is_z3_var(e: z3.ExprRef) -> bool:
    return z3.is_const(e) and e.decl().kind() == z3.Z3_OP_UNINTERPRETED

def get_all_vars(e: z3.ExprRef) -> Set[z3.ExprRef]:
    return {sub for sub in visit_z3_expr(e) if is_z3_var(sub)}

def z3_ord(str_expr: z3.SeqRef) -> z3.ArithRef:
    return z3.parse_smt2_string(
        f"(assert (= 42 (str.to_code {str_expr.sexpr()})))",
        decls={str(c): c for c in get_all_vars(str_expr)}
    )[0].children()[1]

def z3_chr(int_expr: z3.ArithRef) -> z3.SeqRef:
    return z3.parse_smt2_string(
        f"(assert (= \"4\" (str.from_code {int_expr.sexpr()})))",
        decls={str(c): c for c in get_all_vars(int_expr)}
    )[0].children()[1] 
```

##### 获取序数值

我们定义 `zord`，它给定一个符号化的单字符长字符串，获取该字符串的 `ord()` 值。

```py
def zord(context, c):
    return z3_ord(c) 
```

我们可以这样使用它

```py
zc = z3.String('arg_%d' % fresh_name()) 
```

```py
with ConcolicTracer() as _:
    zi = zord(_.context, zc) 
```

没有定义新的变量。

```py
_.context 
```

```py
({}, [])

```

这里是 smtlib 的表示形式。

```py
zi.sexpr() 
```

```py
'(str.to_code arg_2)'

```

我们可以指定 `ord()` 的结果，并调用 `z3.solve()` 以提供所需的解决方案。

```py
(zi == 65).sexpr() 
```

```py
'(= (str.to_code arg_2) 65)'

```

```py
z3.solve([zi == 65]) 
```

```py
[arg_2 = "A"]

```

##### 将序数值转换为 ASCII

类似地，我们可以使用 `zchr()` 将 ASCII 值转换回单个字符字符串。

```py
def zchr(context, i):
    return z3_chr(i) 
```

使用它之前，我们首先定义一个长度为 8 位的位向量。

```py
i = z3.Int('ival_%d' % fresh_name()) 
```

我们现在可以按以下方式检索 `chr()` 表示形式。

```py
with ConcolicTracer() as _:
    zc = zchr(_.context, i) 
```

没有定义新的变量。

```py
_.context 
```

```py
({}, [])

```

```py
(zc== z3.StringVal('a')).sexpr() 
```

```py
'(= (str.from_code ival_1) "a")'

```

和之前一样，我们可以指定调用 `chr()` 后的最终结果，以获取原始参数。

```py
z3.solve([zc == z3.StringVal('a')]) 
```

```py
[ival_1 = 97]

```

##### 字符串之间的等价性

`zstr` 的等价性定义与 `zint` 类似

```py
class zstr(zstr):
    def __eq__(self, other):
        z, v = self._zv(other)
        return zbool(self.context, self.z == z, self.v == v)

    def __req__(self, other):
        return self.__eq__(other) 
```

`zstr` 类的使用方法如下。

```py
def tstr1(s):
    if s == 'h':
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr1 
```

```py
_.zeval() 
```

```py
('sat', {'s': ('h', 'String')})

```

即使有多个字符，它也能正常工作。

```py
def tstr1(s):
    if s == 'hello world':
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr1 
```

```py
_.context 
```

```py
({'tstr1_s_str_1': 'String'}, [tstr1_s_str_1 == "hello world"])

```

```py
_.zeval() 
```

```py
('sat', {'s': ('hello world', 'String')})

```

##### 字符串长度

不幸的是，在 Python 中，我们无法覆盖 `len()` 以返回新的数据类型。因此，我们绕过这个问题。

```py
class zstr(zstr):
    def __len__(self):
        raise NotImplemented() 
```

```py
class zstr(zstr):
    def length(self):
        return self._len 
```

```py
with ConcolicTracer() as _:
    za = zstr.create(_.context, 'str_a', "s")
    if za.length() > 0:
        print(1) 
```

```py
1

```

```py
_.context 
```

```py
({'str_a': 'String'}, [0 < Length(str_a)])

```

```py
def tstr2(s):
    if s.length() > 1:
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr2 
```

```py
_.context 
```

```py
({'tstr2_s_str_1': 'String'}, [1 < Length(tstr2_s_str_1)])

```

```py
_.zeval(log=True) 
```

```py
Predicates in path:
0 1 < Length(tstr2_s_str_1)

(declare-const tstr2_s_str_1 String)
(assert (not (<= (str.len tstr2_s_str_1) 1)))
(check-sat)
(get-model)

z3 -t:6000 /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpwlyj0q93.smt
sat
(
  (define-fun tstr2_s_str_1 () String
    "AB")
)

```

```py
('sat', {'s': ('AB', 'String')})

```

##### 字符串连接

如果我们需要连接两个字符串怎么办？我们需要额外的辅助函数来完成这个操作。

```py
class zstr(zstr):
    def __add__(self, other):
        z, v = self._zv(other)
        return zstr(self.context, self.z + z, self.v + v)

    def __radd__(self, other):
        return self.__add__(other) 
```

这里是如何使用它的。首先，我们创建包装后的参数

```py
with ConcolicTracer() as _:
    v1, v2 = [zstr.create(_.context, 'arg_%d' % fresh_name(), s)
              for s in ['hello', 'world']]
    if (v1 + ' ' + v2) == 'hello world':
        print('hello world') 
```

```py
hello world

```

符号变量的添加在 `context` 中被保留

```py
_.context 
```

```py
({'arg_1': 'String', 'arg_2': 'String'},
 [Concat(Concat(arg_1, " "), arg_2) == "hello world"])

```

##### 生成子字符串

同样，访问子字符串也需要额外的帮助。

```py
class zstr(zstr):
    def __getitem__(self, idx):
        if isinstance(idx, slice):
            start, stop, step = idx.indices(len(self.v))
            assert step == 1  # for now
            assert stop >= start  # for now
            rz = z3.SubString(self.z, start, stop - start)
            rv = self.v[idx]
        elif isinstance(idx, int):
            rz = z3.SubString(self.z, idx, 1)
            rv = self.v[idx]
        else:
            assert False  # for now
        return zstr(self.context, rz, rv)

    def __iter__(self):
        return zstr_iterator(self.context, self) 
```

##### 字符串的迭代器类

我们定义迭代器如下。

```py
class zstr_iterator():
    def __init__(self, context, zstr):
        self.context = context
        self._zstr = zstr
        self._str_idx = 0
        self._str_max = zstr._len  # intz is not an _int_

    def __next__(self):
        if self._str_idx == self._str_max:  # intz#eq
            raise StopIteration
        c = self._zstr[self._str_idx]
        self._str_idx += 1
        return c

    def __len__(self):
        return self._len 
```

这里是如何使用它的。

```py
def tstr3(s):
    if s[0] == 'h' and s[1] == 'e' and s[3] == 'l':
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr3 
```

再次，上下文显示了遇到的谓词。

```py
_.context 
```

```py
({'tstr3_s_str_1': 'String'},
 [str.substr(tstr3_s_str_1, 0, 1) == "h",
  str.substr(tstr3_s_str_1, 1, 1) == "e",
  str.substr(tstr3_s_str_1, 3, 1) == "l"])

```

函数 `zeval()` 返回谓词的解。请注意，返回的值并不是我们传入的参数。这是由于我们有的谓词。也就是说，我们没有对 `s[2]` 上的字符值施加约束。

```py
_.zeval() 
```

```py
('sat', {'s': ('heAl', 'String')})

```

##### 转换为大小写等效

一个主要复杂点是支持 `upper()` 和 `lower()` 方法。我们使用之前定义的 `zchr()` 和 `zord()` 函数来完成此操作。

```py
class zstr(zstr):
    def upper(self):
        empty = ''
        ne = 'empty_%d' % fresh_name()
        result = zstr.create(self.context, ne, empty)
        self.context[1].append(z3.StringVal(empty) == result.z)
        cdiff = (ord('a') - ord('A'))
        for i in self:
            oz = zord(self.context, i.z)
            uz = zchr(self.context, oz - cdiff)
            rz = z3.And([oz >= ord('a'), oz <= ord('z')])
            ov = ord(i.v)
            uv = chr(ov - cdiff)
            rv = ov >= ord('a') and ov <= ord('z')
            if zbool(self.context, rz, rv):
                i = zstr(self.context, uz, uv)
            else:
                i = zstr(self.context, i.z, i.v)
            result += i
        return result 
```

`lower()` 函数与 `upper()` 类似，除了字符范围相反，小写字母位于大写字母之上。因此，我们将差值加到序数上，以将字符转换为小写。

```py
class zstr(zstr):
    def lower(self):
        empty = ''
        ne = 'empty_%d' % fresh_name()
        result = zstr.create(self.context, ne, empty)
        self.context[1].append(z3.StringVal(empty) == result.z)
        cdiff = (ord('a') - ord('A'))
        for i in self:
            oz = zord(self.context, i.z)
            uz = zchr(self.context, oz + cdiff)
            rz = z3.And([oz >= ord('A'), oz <= ord('Z')])
            ov = ord(i.v)
            uv = chr(ov + cdiff)
            rv = ov >= ord('A') and ov <= ord('Z')
            if zbool(self.context, rz, rv):
                i = zstr(self.context, uz, uv)
            else:
                i = zstr(self.context, i.z, i.v)
            result += i
        return result 
```

这里是如何使用 `upper()` 的。

```py
def tstr4(s):
    if s.upper() == 'H':
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr4 
```

再次，我们使用 `zeval()` 解决收集到的约束，并验证我们的约束是否正确。

```py
_.zeval() 
```

```py
('sat', {'s': ('h', 'String')})

```

这里是一个使用 `lower()` 的更大例子：

```py
def tstr5(s):
    if s.lower() == 'hello world':
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr5 
```

```py
_.zeval() 
```

```py
('sat', {'s': ('Hello World', 'String')})

```

再次，我们获得了正确的输入值。

##### 检查字符串前缀

我们定义了 `startswith()`。

```py
class zstr(zstr):
    def startswith(self, other, beg=0, end=None):
        assert end is None  # for now
        assert isinstance(beg, int)  # for now
        zb = z3.IntVal(beg)

        others = other if isinstance(other, tuple) else (other, )

        last = False
        for o in others:
            z, v = self._zv(o)
            r = z3.IndexOf(self.z, z, zb)
            last = zbool(self.context, r == zb, self.v.startswith(v))
            if last:
                return last
        return last 
```

一个例子。

```py
def tstr6(s):
    if s.startswith('hello'):
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr6 
```

```py
_.zeval() 
```

```py
('sat', {'s': ('helloAhello', 'String')})

```

```py
with ConcolicTracer() as _:
    r = _tstr6 
```

```py
_.zeval() 
```

```py
('sat', {'s': ('', 'String')})

```

和之前一样，谓词只确保 `startswith()` 返回了真值。因此，我们的解决方案反映了这一点。

##### 查找子字符串

我们还定义了 `find()`。

```py
class zstr(zstr):
    def find(self, other, beg=0, end=None):
        assert end is None  # for now
        assert isinstance(beg, int)  # for now
        zb = z3.IntVal(beg)
        z, v = self._zv(other)
        zi = z3.IndexOf(self.z, z, zb)
        vi = self.v.find(v, beg, end)
        return zint(self.context, zi, vi) 
```

一个例子。

```py
def tstr7(s):
    if s.find('world') != -1:
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr7 
```

```py
_.zeval() 
```

```py
('sat', {'s': ('worldAworld', 'String')})

```

和之前一样，谓词只确保 `find()` 返回的值大于 -1。因此，我们的解决方案反映了这一点。

##### 从末尾移除空格

接下来，我们实现 `strip()`。

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
class zstr(zstr):
    def rstrip(self, chars=None):
        if chars is None:
            chars = string.whitespace
        if self._len == 0:
            return self
        else:
            last_idx = self._len - 1
            cz = z3.SubString(self.z, last_idx.z, 1)
            cv = self.v[-1]
            zcheck_space = z3.Or([cz == z3.StringVal(char) for char in chars])
            vcheck_space = any(cv == char for char in chars)
            if zbool(self.context, zcheck_space, vcheck_space):
                return zstr(self.context, z3.SubString(self.z, 0, last_idx.z),
                            self.v[0:-1]).rstrip(chars)
            else:
                return self 
```

```py
def tstr8(s):
    if s.rstrip(' ') == 'a b':
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr8
    print(r) 
```

```py
True

```

```py
_.zeval() 
```

```py
('sat', {'s': ('a b   ', 'String')})

```

```py
class zstr(zstr):
    def lstrip(self, chars=None):
        if chars is None:
            chars = string.whitespace
        if self._len == 0:
            return self
        else:
            first_idx = 0
            cz = z3.SubString(self.z, 0, 1)
            cv = self.v[0]
            zcheck_space = z3.Or([cz == z3.StringVal(char) for char in chars])
            vcheck_space = any(cv == char for char in chars)
            if zbool(self.context, zcheck_space, vcheck_space):
                return zstr(self.context, z3.SubString(
                    self.z, 1, self._len.z), self.v[1:]).lstrip(chars)
            else:
                return self 
```

```py
def tstr9(s):
    if s.lstrip(' ') == 'a b':
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr9
    print(r) 
```

```py
True

```

```py
_.zeval() 
```

```py
('sat', {'s': ('   a b', 'String')})

```

```py
class zstr(zstr):
    def strip(self, chars=None):
        return self.lstrip(chars).rstrip(chars) 
```

示例用法。

```py
def tstr10(s):
    if s.strip() == 'a b':
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr10
    print(r) 
```

```py
True

```

```py
_.zeval() 
```

```py
('sat', {'s': ('\\u{c}\\u{a}\\u{9}\\u{a}a b\\u{d}\\u{d}', 'String')})

```

`strip()`生成了正确的约束。

##### 分割字符串

我们按照以下方式实现字符串`split()`。

```py
class zstr(zstr):
    def split(self, sep=None, maxsplit=-1):
        assert sep is not None  # default space based split is complicated
        assert maxsplit == -1  # for now.
        zsep = z3.StringVal(sep)
        zl = z3.Length(zsep)
        # zi would be the length of prefix
        zi = z3.IndexOf(self.z, zsep, z3.IntVal(0))
        # Z3Bug: There is a bug in the `z3.IndexOf` method which returns
        # `z3.SeqRef` instead of `z3.ArithRef`. So we need to fix it.
        zi = z3.ArithRef(zi.ast, zi.ctx)

        vi = self.v.find(sep)
        if zbool(self.context, zi >= z3.IntVal(0), vi >= 0):
            zprefix = z3.SubString(self.z, z3.IntVal(0), zi)
            zmid = z3.SubString(self.z, zi, zl)
            zsuffix = z3.SubString(self.z, zi + zl,
                                   z3.Length(self.z))
            return [zstr(self.context, zprefix, self.v[0:vi])] + zstr(
                self.context, zsuffix, self.v[vi + len(sep):]).split(
                    sep, maxsplit)
        else:
            return [self] 
```

```py
def tstr11(s):
    if s.split(',') == ['a', 'b', 'c']:
        return True
    else:
        return False 
```

```py
with ConcolicTracer() as _:
    r = _tstr11
    print(r) 
```

```py
True

```

```py
_.zeval() 
```

```py
('sat', {'s': ('a,b,c', 'String')})

```

##### 陷阱线

为了便于调试，我们终止对`str`中未由`zstr`覆盖的方法的任何调用。

```py
def make_str_abort_wrapper(fun):
    def proxy(*args, **kwargs):
        raise Exception('%s Not implemented in `zstr`' % fun.__name__)
    return proxy 
```

```py
def init_concolic_3():
    strmembers = inspect.getmembers(zstr, callable)
    zstrmembers = {m[0] for m in strmembers if len(
        m) == 2 and 'zstr' in m[1].__qualname__}
    for name, fn in inspect.getmembers(str, callable):
        # Omitted 'splitlines' as this is needed for formatting output in
        # IPython/Jupyter
        if name not in zstrmembers and name not in [
            'splitlines',
            '__class__',
            '__contains__',
            '__delattr__',
            '__dir__',
            '__format__',
            '__ge__',
            '__getattribute__',
            '__getnewargs__',
            '__gt__',
            '__hash__',
            '__le__',
            '__len__',
            '__lt__',
            '__mod__',
            '__mul__',
            '__ne__',
            '__reduce__',
            '__reduce_ex__',
            '__repr__',
            '__rmod__',
            '__rmul__',
            '__setattr__',
            '__sizeof__',
                '__str__']:
            setattr(zstr, name, make_str_abort_wrapper(fn)) 
```

```py
INITIALIZER_LIST.append(init_concolic_3) 
```

```py
init_concolic_3() 
```</details>

### 示例：三角形

我们之前展示了如何在`ConcolicTracer`下运行`triangle()`。

```py
with ConcolicTracer() as _:
    print(_triangle) 
```

```py
scalene

```

符号变量如下：

```py
_.decls 
```

```py
{'triangle_a_int_1': 'Int',
 'triangle_b_int_2': 'Int',
 'triangle_c_int_3': 'Int'}

```

谓词如下：

```py
_.path 
```

```py
[Not(triangle_a_int_1 == triangle_b_int_2),
 Not(triangle_b_int_2 == triangle_c_int_3),
 Not(triangle_a_int_1 == triangle_c_int_3)]

```

使用`zeval()`解决这些路径条件并获得解决方案。我们发现 Z3 给了我们三个不同的整数值：

```py
_.zeval() 
```

```py
('sat',
 {'a': ('0', 'Int'), 'b': (['-', '2'], 'Int'), 'c': (['-', '1'], 'Int')})

```

（注意，某些值可能是负数。实际上，`triangle()`也使用负长度值，即使真实三角形只有正长度。）

如果我们使用这些值调用`triangle()`，我们将采取与原始输入*完全相同的路径*：

```py
triangle(0, -2, -1) 
```

```py
'scalene'

```

我们可以让 z3 *否定*个别条件——从而采取不同的路径。首先，我们检索符号变量。

```py
za, zb, zc = [z3.Int(s) for s in _.decls.keys()] 
```

```py
za, zb, zc 
```

```py
(triangle_a_int_1, triangle_b_int_2, triangle_c_int_3)

```

然后，我们将一个否定谓词传递给`zeval()`。键（这里：`1`）确定新谓词将替换哪个谓词。

```py
_.zeval({1: zb == zc}) 
```

```py
('sat', {'a': ('1', 'Int'), 'b': ('0', 'Int'), 'c': ('0', 'Int')})

```

```py
triangle(1, 0, 1) 
```

```py
'isosceles'

```

更新的谓词如预期返回`isosceles`。通过否定进一步的条件，我们可以系统地探索`triangle()`中的所有分支。

### 示例：解码 CGI 字符串

让我们在覆盖率章节中提到的示例程序`cgi_decode()`上应用`ConcolicTracer`。请注意，我们需要稍微重写其代码，因为`hex_values`中的哈希查找还不能用于传递约束。

```py
def cgi_decode(s):
  """Decode the CGI-encoded string `s`:
 * replace "+" by " "
 * replace "%xx" by the character with hex number xx.
 Return the decoded string.  Raise `ValueError` for invalid inputs."""

    # Mapping of hex digits to their integer values
    hex_values = {
        '0': 0, '1': 1, '2': 2, '3': 3, '4': 4,
        '5': 5, '6': 6, '7': 7, '8': 8, '9': 9,
        'a': 10, 'b': 11, 'c': 12, 'd': 13, 'e': 14, 'f': 15,
        'A': 10, 'B': 11, 'C': 12, 'D': 13, 'E': 14, 'F': 15,
    }

    t = ''
    i = 0
    while i < s.length():
        c = s[i]
        if c == '+':
            t += ' '
        elif c == '%':
            digit_high, digit_low = s[i + 1], s[i + 2]
            i = i + 2
            found = 0
            v = 0
            for key in hex_values:
                if key == digit_high:
                    found = found + 1
                    v = hex_values[key] * 16
                    break
            for key in hex_values:
                if key == digit_low:
                    found = found + 1
                    v = v + hex_values[key]
                    break
            if found == 2:
                if v >= 128:
                    # z3.StringVal(urllib.parse.unquote('%80')) <-- bug in z3
                    raise ValueError("Invalid encoding")
                t = t + chr(v)
            else:
                raise ValueError("Invalid encoding")
        else:
            t = t + c
        i = i + 1
    return t 
```

```py
with ConcolicTracer() as _:
    _cgi_decode 
```

```py
_.context 
```

```py
({'cgi_decode_s_str_1': 'String'}, [Not(0 < Length(cgi_decode_s_str_1))])

```

```py
with ConcolicTracer() as _:
    _cgi_decode 
```

一旦执行，我们就可以在`decls`属性中检索到符号变量。这是一个符号变量到类型的映射。

```py
_.decls 
```

```py
{'cgi_decode_s_str_1': 'String'}

```

提取的路径条件可以在`path`属性中找到：

```py
_.path 
```

```py
[0 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 0, 1) == "+"),
 Not(str.substr(cgi_decode_s_str_1, 0, 1) == "%"),
 1 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 1, 1) == "+"),
 str.substr(cgi_decode_s_str_1, 1, 1) == "%",
 Not(str.substr(cgi_decode_s_str_1, 2, 1) == "0"),
 Not(str.substr(cgi_decode_s_str_1, 2, 1) == "1"),
 str.substr(cgi_decode_s_str_1, 2, 1) == "2",
 str.substr(cgi_decode_s_str_1, 3, 1) == "0",
 4 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 4, 1) == "+"),
 Not(str.substr(cgi_decode_s_str_1, 4, 1) == "%"),
 Not(5 < Length(cgi_decode_s_str_1))]

```

`context`属性包含一对`decls`和`path`属性；这对于将其传递给`ConcolicTracer`构造函数很有用。

```py
assert _.context == (_.decls, _.path) 
```

我们可以解决这些约束以获得与原始（跟踪）调用相同路径的函数参数的值：

```py
_.zeval() 
```

```py
('sat', {'s': ('A%20B', 'String')})

```

否定其中一些约束将产生不同的路径，从而提高代码覆盖率。这正是我们的 concolic 模糊器（见后）所做的。让我们去否定第一个约束，即第一个字符*不应*是`+`字符：

```py
_.path[0] 
```

0 < Length(cgi_decode_s_str_1)

为了计算否定字符串，我们必须通过 z3 原语来构建它：

```py
zs = z3.String('cgi_decode_s_str_1') 
```

```py
z3.SubString(zs, 0, 1) == z3.StringVal('a') 
```

str.substr(cgi_decode_s_str_1, 0, 1) = "a"

使用要更改的路径条件调用`zeval()`获得满足否定谓词的新输入：

```py
(result, new_vars) = _.zeval({1: z3.SubString(zs, 0, 1) == z3.StringVal('+')}) 
```

```py
new_vars 
```

```py
{'s': ('+%20A', 'String')}

```

```py
(new_s, new_s_type) = new_vars['s'] 
```

```py
new_s 
```

```py
'+%20A'

```

我们可以通过重新运行带有`new_s`作为输入的 tracer 来验证`new_s`确实采取了新的路径：

```py
with ConcolicTracer() as _:
    _cgi_decode 
```

```py
_.path 
```

```py
[0 < Length(cgi_decode_s_str_1),
 str.substr(cgi_decode_s_str_1, 0, 1) == "+",
 1 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 1, 1) == "+"),
 str.substr(cgi_decode_s_str_1, 1, 1) == "%",
 Not(str.substr(cgi_decode_s_str_1, 2, 1) == "0"),
 Not(str.substr(cgi_decode_s_str_1, 2, 1) == "1"),
 str.substr(cgi_decode_s_str_1, 2, 1) == "2",
 str.substr(cgi_decode_s_str_1, 3, 1) == "0",
 4 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 4, 1) == "+"),
 Not(str.substr(cgi_decode_s_str_1, 4, 1) == "%"),
 Not(5 < Length(cgi_decode_s_str_1))]

```

通过否定进一步的条件，我们可以探索更多的代码。

### 示例：四舍五入

这里是一个给你最近的十倍乘数的函数

```py
def round10(r):
    while r % 10 != 0:
        r += 1
    return r 
```

如前所述，我们在`ConcolicTracer`上下文中执行函数。

```py
with ConcolicTracer() as _:
    r = _round10 
```

我们验证了我们能够捕获所有谓词：

```py
_.context 
```

```py
({'round10_r_int_1': 'Int'},
 [0 != round10_r_int_1%10,
  0 != (round10_r_int_1 + 1)%10,
  0 != (round10_r_int_1 + 1 + 1)%10,
  0 != (round10_r_int_1 + 1 + 1 + 1)%10,
  0 != (round10_r_int_1 + 1 + 1 + 1 + 1)%10,
  0 != (round10_r_int_1 + 1 + 1 + 1 + 1 + 1)%10,
  0 != (round10_r_int_1 + 1 + 1 + 1 + 1 + 1 + 1)%10,
  0 != (round10_r_int_1 + 1 + 1 + 1 + 1 + 1 + 1 + 1)%10,
  0 != (round10_r_int_1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1)%10,
  Not(0 !=
      (round10_r_int_1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1)%10)])

```

我们使用`zeval()`来获取更多采取相同路径的输入。

```py
_.zeval() 
```

```py
('sat', {'r': (['-', '9'], 'Int')})

```

### 示例：绝对最大值

我们的 concolic 代理是否在函数之间工作？比如说我们有一个函数 `max_value()` 如下。

```py
def abs_value(a):
    if a > 0:
        return a
    else:
        return -a 
```

它由另一个函数 `abs_max()` 调用。

```py
def abs_max(a, b):
    a1 = abs_value(a)
    b1 = abs_value(b)
    if a1 > b1:
        c = a1
    else:
        c = b1
    return c 
```

在 `abs_max()` 上使用 `Concolic()` 上下文。

```py
with ConcolicTracer() as _:
    _abs_max 
```

如预期的那样，我们在函数之间有谓词。

```py
_.context 
```

```py
({'abs_max_a_int_1': 'Int', 'abs_max_b_int_2': 'Int'},
 [0 < abs_max_a_int_1, 0 < abs_max_b_int_2, abs_max_a_int_1 > abs_max_b_int_2])

```

```py
_.zeval() 
```

```py
('sat', {'a': ('2', 'Int'), 'b': ('1', 'Int')})

```

解决谓词按预期工作。

使用负数作为参数，以便在 `abs_value()` 中采取不同的分支。

```py
with ConcolicTracer() as _:
    _abs_max 
```

```py
_.context 
```

```py
({'abs_max_a_int_1': 'Int', 'abs_max_b_int_2': 'Int'},
 [Not(0 < abs_max_a_int_1),
  Not(0 < abs_max_b_int_2),
  -abs_max_a_int_1 > -abs_max_b_int_2])

```

```py
_.zeval() 
```

```py
('sat', {'a': (['-', '1'], 'Int'), 'b': ('0', 'Int')})

```

该解决方案反映了我们的谓词。（我们在 `abs_value()` 中使用了 `a > 0`）。

### 示例：二项式系数

对于一个使用不同类型变量的更大示例，比如说我们想通过以下公式计算二项式系数

$$ ^nP_k=\frac{n!}{(n-k)!} $$

$$ \binom nk=\,^nC_k=\frac{^nP_k}{k!} $$

我们定义函数如下。

```py
def factorial(n):
    v = 1
    while n != 0:
        v *= n
        n -= 1

    return v 
```

```py
def permutation(n, k):
    return factorial(n) / factorial(n - k) 
```

```py
def combination(n, k):
    return permutation(n, k) / factorial(k) 
```

```py
def binomial(n, k):
    if n < 0 or k < 0 or n < k:
        raise Exception('Invalid values')
    return combination(n, k) 
```

如前所述，我们在 `ConcolicTracer` 下运行该函数。

```py
with ConcolicTracer() as _:
    v = _binomial 
```

然后调用 `zeval()` 进行评估。

```py
_.zeval() 
```

```py
('sat', {'n': ('4', 'Int'), 'k': ('2', 'Int')})

```

### 示例：数据库

对于一个使用 Concolic 字符串类 `zstr` 的更大示例，我们使用来自 信息流章节 的 DB 类。

```py
if __name__ == '__main__':
    if z3.get_version() > (4, 8, 7, 0):
        print("""Note: The following example may not work with your Z3 version;
see https://github.com/Z3Prover/z3/issues/5763 for details.
Consider `pip install z3-solver==4.8.7.0` as a workaround.""") 
```

```py
Note: The following example may not work with your Z3 version;
see https://github.com/Z3Prover/z3/issues/5763 for details.
Consider `pip install z3-solver==4.8.7.0` as a workaround.

```

```py
from InformationFlow import DB, sample_db, update_inventory 
```

我们首先填充我们的数据库。

```py
from GrammarMiner import VEHICLES  # minor dependency 
```

```py
db = sample_db()
for V in VEHICLES:
    update_inventory(db, V) 
```

```py
db.db 
```

```py
{'inventory': ({'year': int, 'kind': str, 'company': str, 'model': str},
  [{'year': 1997, 'kind': 'van', 'company': 'Ford', 'model': 'E350'},
   {'year': 2000, 'kind': 'car', 'company': 'Mercury', 'model': 'Cougar'},
   {'year': 1999, 'kind': 'car', 'company': 'Chevy', 'model': 'Venture'}])}

```

我们现在准备模糊测试我们的 `DB` 类。散列函数难以直接处理（因为它们依赖于内部 C 函数）。因此，我们稍微修改了 `table()`。

```py
class ConcolicDB(DB):
    def table(self, t_name):
        for k, v in self.db:
            if t_name == k:
                return v
        raise SQLException('Table (%s) was not found' % repr(t_name))

    def column(self, decl, c_name):
        for k in decl:
            if c_name == k:
                return decl[k]
        raise SQLException('Column (%s) was not found' % repr(c_name)) 
```

为了简化，我们定义了一个单独的函数 `db_select()`，它直接调用 `db.sql()`。

```py
def db_select(s):
    my_db = ConcolicDB()
    my_db.db = [(k, v) for (k, v) in db.db.items()]
    r = my_db.sql(s)
    return r 
```

现在我们想在 `ConcolicTracer` 下运行 SQL 语句，并收集得到的谓词。

```py
with ConcolicTracer() as _:
    _db_select 
```

执行过程中遇到的谓词如下：

```py
_.path 
```

```py
[0 == IndexOf(db_select_s_str_1, "select ", 0),
 0 == IndexOf(db_select_s_str_1, "select ", 0),
 Not(0 >
     IndexOf(str.substr(db_select_s_str_1, 7, 19),
             " from ",
             0)),
 Not(Or(0 <
        IndexOf(str.substr(db_select_s_str_1, 7, 19),
                " where ",
                0),
        0 ==
        IndexOf(str.substr(db_select_s_str_1, 7, 19),
                " where ",
                0))),
 str.substr(str.substr(db_select_s_str_1, 7, 19), 10, 9) ==
 "inventory"]

```

我们可以像之前一样使用 `zeval()` 来解决约束。

```py
_.zeval() 
```

```py
('Gave up', None)

```

## 带约束的模糊测试

`SimpleConcolicFuzzer` 类从由其他模糊测试器生成的样本输入开始。然后它在 `ConcolicTracer` 下运行正在测试的函数，并收集路径谓词。然后它对路径内的随机谓词取反，并用 Z3 解决它，以产生一个新的输出，该输出保证采取与原始输出不同的路径。

与上面的 `ConcolicTracer` 一样，请首先查看示例，然后再深入研究实现。

<details id="Excursion:-Implementing-SimpleConcolicFuzzer"><summary>实现 SimpleConcolicFuzzer</summary>

首先，我们导入 `Fuzzer` 接口，并编写示例程序 `hang_if_no_space()`

```py
from Fuzzer import Fuzzer 
```

```py
def hang_if_no_space(s):
    i = 0
    while True:
        if i < s.length():
            if s[i] == ' ':
                break
        i += 1 
```

```py
from ExpectError import ExpectTimeout, ExpectError 
```

```py
import [random](https://docs.python.org/3/library/random.html) 
```

#### 决策表示

为了使模糊测试器工作，我们需要一种表示在跟踪期间做出的决策的方法。我们将其保存在一个 *二叉树* 中，其中每个节点代表一个做出的决策，每个叶子节点代表一个完整的路径。二叉树中的节点由 `TraceNode` 类表示。

当添加新节点时，它代表父节点在某个谓词上做出的决策。这个谓词作为 `smt_val` 提供，对于这个子节点来说，它是 `True` 以到达。由于谓词实际上存在于父节点中，我们还携带一个成员 `smt`，它将由第一个添加的子节点更新。

```py
class TraceNode:
    def __init__(self, smt_val, parent, info):
        # This is the smt that lead to this node
        self._smt_val = z3.simplify(smt_val) if smt_val is not None else None

        # This is the predicate that this node might perform at a future point
        self.smt = None
        self.info = info
        self.parent = parent
        self.children = {}
        self.path = None
        self.tree = None
        self._pattern = None
        self.log = True

    def no(self): return self.children.get(self.tree.no_bit)

    def yes(self): return self.children.get(self.tree.yes_bit)

    def get_children(self): return (self.no(), self.yes())

    def __str__(self):
        return 'TraceNode[%s]' % ','.join(self.children.keys()) 
```

我们添加一个 `PlausibleChild` 类来跟踪叶子节点。

```py
class PlausibleChild:
    def __init__(self, parent, cond, tree):
        self.parent = parent
        self.cond = cond
        self.tree = tree
        self._smt_val = None

    def __repr__(self):
        return 'PlausibleChild[%s]' % (self.parent.pattern() + ':' + self.cond) 
```

当使用叶子节点生成新的路径时，我们期望其兄弟 `TraceNode` 已经被探索。因此，我们使用兄弟的值作为上下文 `cc` 和父节点的 `smt_val`。

```py
class PlausibleChild(PlausibleChild):
    def smt_val(self):
        if self._smt_val is not None:
            return self._smt_val
        # if the parent has other children, then that child would have updatd the parent's smt
        # Hence, we can use that child's smt_value's opposite as our value.
        assert self.parent.smt is not None
        if self.cond == self.tree.no_bit:
            self._smt_val = z3.Not(self.parent.smt)
        else:
            self._smt_val = self.parent.smt
        return self._smt_val

    def cc(self):
        if self.parent.info.get('cc') is not None:
            return self.parent.info['cc']
        # if there is a plausible child node, it means that there can
        # be at most one child.
        siblings = list(self.parent.children.values())
        assert len(siblings) == 1
        # We expect at the other child to have cc
        return siblings[0].info['cc'] 
```

`PlausibleChild` 实例用于使用 `path_expression()` 生成新的探索路径。

```py
class PlausibleChild(PlausibleChild):
    def path_expression(self):
        path_to_root = self.parent.get_path_to_root()
        assert path_to_root[0]._smt_val is None
        return [i._smt_val for i in path_to_root[1:]] + [self.smt_val()] 
```

`TraceTree` 类帮助我们跟踪二叉树。一开始，根节点是一个哨兵 `TraceNode` 实例，并且简单地有两个可能的子节点作为叶子节点。一旦添加了第一个跟踪，其中一个可能的子节点将变成一个真正的子节点。

```py
class TraceTree:
    def __init__(self):
        self.root = TraceNode(smt_val=None, parent=None, info={'num': 0})
        self.root.tree = self
        self.leaves = {}
        self.no_bit, self.yes_bit = '0', '1'

        pprefix = ':'
        for bit in [self.no_bit, self.yes_bit]:
            self.leaves[pprefix + bit] = PlausibleChild(self.root, bit, self)
        self.completed_paths = {} 
```

`TraceTree` 的 `add_trace()` 方法提供了一种添加新跟踪的方式。它被单独保留在初始化之外，因为我们可能希望从同一个函数添加多个跟踪。

```py
class TraceTree(TraceTree):
    def add_trace(self, tracer, string):
        last = self.root
        i = 0
        for i, elt in enumerate(tracer.path):
            last = last.add_child(elt=elt, i=i + 1, cc=tracer, string=string)
        last.add_child(elt=z3.BoolVal(True), i=i + 1, cc=tracer, string=string) 
```

为了使 `add_trace()` 方法工作，我们需要更多的基础设施，我们将在下面定义。

`bit()` 方法将谓词转换为一个与每个谓词所做决策相对应的位。如果选择 `if` 分支，结果是 `1`，而 `else` 分支由 `0` 表示。模式表示从根到叶子所需的决定的位模式。

```py
class TraceNode(TraceNode):
    def bit(self):
        if self._smt_val is None:
            return None
        return self.tree.no_bit if self._smt_val.decl(
        ).name() == 'not' else self.tree.yes_bit

    def pattern(self):
        if self._pattern is not None:
            return self._pattern
        path = self.get_path_to_root()
        assert path[0]._smt_val is None
        assert path[0].parent is None

        self._pattern = ''.join([p.bit() for p in path[1:]])
        return self._pattern 
```

每个节点都知道如何添加一个新的子节点，并获取到根的路径，该路径被缓存。

当我们将子节点添加到根节点时，这意味着当前节点有一个决策，子节点是决策的结果。因此，为了获取正在进行的决策，我们简化 `smt` 表达式，并检查它是否以 `not` 开头。如果它不以 `not` 开头，我们将其解释为节点中的当前决策。如果它以 `not` 开头，那么我们将其解释为当前节点正在评估的表达式 `not(smt)`。

我们只知道在至少遍历程序一次之后做出的第一个决策。一旦程序被遍历，我们就更新父节点，以反映导致当前子节点的决策。

```py
class TraceNode(TraceNode):
    def add_child(self, elt, i, cc, string):
        if elt == z3.BoolVal(True):
            # No more exploration here. Simply unregister the leaves of *this*
            # node and possibly register them in completed nodes, and exit
            for bit in [self.tree.no_bit, self.tree.yes_bit]:
                child_leaf = self.pattern() + ':' + bit
                if child_leaf in self.tree.leaves:
                    del self.tree.leaves[child_leaf]
            self.tree.completed_paths[self.pattern()] = self
            return None

        child_node = TraceNode(smt_val=elt,
                               parent=self,
                               info={'num': i, 'cc': cc, 'string': string})
        child_node.tree = self.tree

        # bit represents the path that child took from this node.
        bit = child_node.bit()

        # first we update our smt decision
        if bit == self.tree.yes_bit:  # yes, which means the smt can be used as is
            if self.smt is not None:
                assert self.smt == child_node._smt_val
            else:
                self.smt = child_node._smt_val
        # no, which means we have to negate it to get the decision.
        elif bit == self.tree.no_bit:
            smt_ = z3.simplify(z3.Not(child_node._smt_val))
            if self.smt is not None:
                assert smt_ == self.smt
            else:
                self.smt = smt_
        else:
            assert False

        if bit in self.children:
            #    if self.log:
            #print(elt, child_node.bit(), i, string)
            #print(i,'overwriting', bit,'=>',self.children[bit],'with',child_node)
            child_node = self.children[bit]
            #self.children[bit] = child_node
            #child_node.children = old.children
        else:
            self.children[bit] = child_node

        # At this point, we have to unregister any leaves that correspond to this child from tree,
        # and add the plausible children of this child as leaves to be explored. Note that
        # if it is the end (z3.True), we do not have any more children.
        child_leaf = self.pattern() + ':' + bit
        if child_leaf in self.tree.leaves:
            del self.tree.leaves[child_leaf]

        pprefix = child_node.pattern() + ':'

        # Plausible children.
        for bit in [self.tree.no_bit, self.tree.yes_bit]:
            self.tree.leaves[pprefix +
                             bit] = PlausibleChild(child_node, bit, self.tree)
        return child_node 
```

从任何节点到根的路径只计算一次并缓存。

```py
class TraceNode(TraceNode):
    def get_path_to_root(self):
        if self.path is not None:
            return self.path
        parent_path = []
        if self.parent is not None:
            parent_path = self.parent.get_path_to_root()
        self.path = parent_path + [self]
        return self.path 
```

#### `SimpleConcolicFuzzer` 类

`SimpleConcolicFuzzer` 使用 `Fuzzer` 接口定义。

```py
class SimpleConcolicFuzzer(Fuzzer):
    def __init__(self):
        self.ct = TraceTree()
        self.max_tries = 1000
        self.last = None
        self.last_idx = None 
```

我们定义的 `add_trace()` 方法如下使用。首先，我们使用一个随机字符串生成 concolic 跟踪。

```py
with ExpectTimeout(2):
    with ConcolicTracer() as _:
        _hang_if_no_space 
```

接下来，我们初始化并将此跟踪添加到模糊测试器中。

```py
_.path 
```

```py
[0 < Length(hang_if_no_space_s_str_1),
 Not(str.substr(hang_if_no_space_s_str_1, 0, 1) == " "),
 1 < Length(hang_if_no_space_s_str_1),
 Not(str.substr(hang_if_no_space_s_str_1, 1, 1) == " "),
 2 < Length(hang_if_no_space_s_str_1),
 str.substr(hang_if_no_space_s_str_1, 2, 1) == " "]

```

```py
scf = SimpleConcolicFuzzer()
scf.ct.add_trace(_, 'ab d') 
```

我们添加的路径可以从以下 `TraceTree` 中获得。

```py
[i._smt_val for i in scf.ct.root.get_children(
)[0].get_children(
)[0].get_children(
)[0].get_children(
)[0].get_children(
)[0].get_children(
)[1].get_path_to_root()] 
```

```py
[None,
 Not(Length(hang_if_no_space_s_str_1) <= 0),
 Not(str.substr(hang_if_no_space_s_str_1, 0, 1) == " "),
 Not(Length(hang_if_no_space_s_str_1) <= 1),
 Not(str.substr(hang_if_no_space_s_str_1, 1, 1) == " "),
 Not(Length(hang_if_no_space_s_str_1) <= 2),
 str.substr(hang_if_no_space_s_str_1, 2, 1) == " "]

```

下面是我们现在可以探索的已注册的叶子节点。

```py
for key in scf.ct.leaves:
    print(key, '\t', scf.ct.leaves[key]) 
```

```py
:1 	 PlausibleChild[:1]
0:1 	 PlausibleChild[0:1]
00:1 	 PlausibleChild[00:1]
000:1 	 PlausibleChild[000:1]
0000:1 	 PlausibleChild[0000:1]
00000:0 	 PlausibleChild[00000:0]

```

接下来，我们需要一种方法来可视化构建的树。

```py
from GrammarFuzzer import display_tree 
```

```py
TREE_NODES = {} 
```

```py
def my_extract_node(tnode, id):
    key, node, parent = tnode
    if node is None:
        # return '? (%s:%s)' % (parent.pattern(), key) , [], ''
        return '?', [], ''
    if node.smt is None:
        return '* %s' % node.info.get('string', ''), [], ''

    no, yes = node.get_children()
    num = str(node.info.get('num'))
    children = [('0', no, node), ('1', yes, node)]
    TREE_NODES[id] = 0
    return "(%s) %s" % (num, str(node.smt)), children, '' 
```

```py
def my_edge_attr(dot, start_node, stop_node):
    # the edges are always drawn '0:NO' first.
    if TREE_NODES[start_node] == 0:
        color, label = 'red', '0'
        TREE_NODES[start_node] = 1
    else:
        color, label = 'blue', '1'
        TREE_NODES[start_node] = 2
    dot.edge(repr(start_node), repr(stop_node), color=color, label=label) 
```

```py
def display_trace_tree(root):
    TREE_NODES.clear()
    return display_tree(
        ('', root, None), extract_node=my_extract_node, edge_attr=my_edge_attr) 
```

```py
display_trace_tree(scf.ct.root) 
```

<svg width="700pt" height="409pt" viewBox="0.00 0.00 699.50 409.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 405.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="570.75" y="-387.95" font-family="Times,serif" font-size="14.00">(0) Length(hang_if_no_space_s_str_1) <= 0</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="480.75" y="-323.45" font-family="Times,serif" font-size="14.00">(1) str.substr(hang_if_no_space_s_str_1, 0, 1) == " "</text></g> <g id="edge1" class="edge"><title>0->1</title> <text text-anchor="middle" x="537.19" y="-355.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="659.75" y="-323.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge12" class="edge"><title>0->12</title> <text text-anchor="middle" x="626.6" y="-355.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="402.75" y="-258.95" font-family="Times,serif" font-size="14.00">(2) Length(hang_if_no_space_s_str_1) <= 1</text></g> <g id="edge2" class="edge"><title>1->2</title> <text text-anchor="middle" x="452.11" y="-291.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="557.75" y="-258.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge11" class="edge"><title>1->11</title> <text text-anchor="middle" x="529.52" y="-291.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="312.75" y="-194.45" font-family="Times,serif" font-size="14.00">(3) str.substr(hang_if_no_space_s_str_1, 1, 1) == " "</text></g> <g id="edge3" class="edge"><title>2->3</title> <text text-anchor="middle" x="369.19" y="-226.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="491.75" y="-194.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge10" class="edge"><title>2->10</title> <text text-anchor="middle" x="458.6" y="-226.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="234.75" y="-129.95" font-family="Times,serif" font-size="14.00">(4) Length(hang_if_no_space_s_str_1) <= 2</text></g> <g id="edge4" class="edge"><title>3->4</title> <text text-anchor="middle" x="284.11" y="-162.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="389.75" y="-129.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge9" class="edge"><title>3->9</title> <text text-anchor="middle" x="361.52" y="-162.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="144.75" y="-65.45" font-family="Times,serif" font-size="14.00">(5) str.substr(hang_if_no_space_s_str_1, 2, 1) == " "</text></g> <g id="edge5" class="edge"><title>4->5</title> <text text-anchor="middle" x="201.19" y="-97.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="323.75" y="-65.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge8" class="edge"><title>4->8</title> <text text-anchor="middle" x="290.6" y="-97.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="119.75" y="-0.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge6" class="edge"><title>5->6</title> <text text-anchor="middle" x="137.86" y="-33.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="170.75" y="-0.95" font-family="Times,serif" font-size="14.00">* ab d</text></g> <g id="edge7" class="edge"><title>5->7</title> <text text-anchor="middle" x="163.45" y="-33.2" font-family="Times,serif" font-size="14.00">1</text></g></g></svg>

例如，模式 `00000:0` 对应以下谓词。

```py
scf.ct.leaves['00000:0'] 
```

```py
PlausibleChild[00000:0]

```

```py
scf.ct.leaves['00000:0'].path_expression() 
```

```py
[Not(Length(hang_if_no_space_s_str_1) <= 0),
 Not(str.substr(hang_if_no_space_s_str_1, 0, 1) == " "),
 Not(Length(hang_if_no_space_s_str_1) <= 1),
 Not(str.substr(hang_if_no_space_s_str_1, 1, 1) == " "),
 Not(Length(hang_if_no_space_s_str_1) <= 2),
 Not(str.substr(hang_if_no_space_s_str_1, 2, 1) == " ")]

```

类似地，模式 `:1` 对应以下谓词。

```py
scf.ct.leaves[':1'] 
```

```py
PlausibleChild[:1]

```

```py
scf.ct.leaves[':1'].path_expression() 
```

```py
[Length(hang_if_no_space_s_str_1) <= 0]

```

我们现在可以通过寻找一个不完全探索的叶子节点来生成下一个要生成的输入。想法是收集所有叶子节点，并随机选择一个。

```py
class SimpleConcolicFuzzer(SimpleConcolicFuzzer):
    def add_trace(self, trace, s):
        self.ct.add_trace(trace, s)

    def next_choice(self):
        #lst = sorted(list(self.ct.leaves.keys()), key=len)
        c = random.choice(list(self.ct.leaves.keys()))
        #c = lst[0]
        return self.ct.leaves[c] 
```

我们如下使用 `next_choice()`。

```py
scf = SimpleConcolicFuzzer()
scf.add_trace(_, 'ab d')
node = scf.next_choice() 
```

```py
node 
```

```py
PlausibleChild[0000:1]

```

```py
node.path_expression() 
```

```py
[Not(Length(hang_if_no_space_s_str_1) <= 0),
 Not(str.substr(hang_if_no_space_s_str_1, 0, 1) == " "),
 Not(Length(hang_if_no_space_s_str_1) <= 1),
 Not(str.substr(hang_if_no_space_s_str_1, 1, 1) == " "),
 Length(hang_if_no_space_s_str_1) <= 2]

```

我们获取下一个探索的选择，并扩展路径表达式，然后使用 `get_newpath()` 方法返回它以及一个上下文。

```py
class SimpleConcolicFuzzer(SimpleConcolicFuzzer):
    def get_newpath(self):
        node = self.next_choice()
        path = node.path_expression()
        return path, node.cc() 
```

```py
scf = SimpleConcolicFuzzer()
scf.add_trace(_, 'abcd')
path, cc = scf.get_newpath()
path 
```

```py
[Length(hang_if_no_space_s_str_1) <= 0]

```

#### 模糊测试方法

`fuzz()`方法简单地生成新的谓词列表，并求解它们以产生新的输入。

```py
class SimpleConcolicFuzzer(SimpleConcolicFuzzer):
    def fuzz(self):
        if self.ct.root.children == {}:
            # a random value to generate comparisons. This would be
            # the initial value around which we explore with concolic
            # fuzzing.
            # str_len = random.randint(1,100)
            # return ' '*str_len
            return ' '
        for i in range(self.max_tries):
            path, last = self.get_newpath()
            s, v = zeval_smt(path, last, log=False)
            if s != 'sat':
                # raise Exception("Unexpected UNSAT")
                continue

            val = list(v.values())[0]
            elt, typ = val

            # make sure that we do not retry the tried paths
            # The tracer we add here is incomplete. This gets updated when
            # the add_trace is called from the concolic fuzzer context.
            # self.add_trace(ConcolicTracer((last.decls, path)), elt)
            if typ == 'Int':
                if len(elt) == 2 and elt[0] == '-':  # negative numbers are [-, x]
                    return -1*int(elt[1])
                return int(elt)
            elif typ == 'String':
                return elt
            return elt
        return None 
```</details>

为了说明`SimpleConcolicFuzzer`，让我们将其应用于`Coverage`章节中的示例程序`cgi_decode()`。请注意，我们不能直接使用它，因为`hex_values`中的哈希查找还不能用于传递约束。

```py
with ConcolicTracer() as _:
    _cgi_decode 
```

```py
_.path 
```

```py
[0 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 0, 1) == "+"),
 Not(str.substr(cgi_decode_s_str_1, 0, 1) == "%"),
 1 < Length(cgi_decode_s_str_1),
 str.substr(cgi_decode_s_str_1, 1, 1) == "+",
 2 < Length(cgi_decode_s_str_1),
 Not(str.substr(cgi_decode_s_str_1, 2, 1) == "+"),
 Not(str.substr(cgi_decode_s_str_1, 2, 1) == "%"),
 Not(3 < Length(cgi_decode_s_str_1))]

```

```py
scf = SimpleConcolicFuzzer()
scf.add_trace(_, 'a+c') 
```

*跟踪树*显示了迄今为止遇到的路径条件。任何指向"?"的蓝色边都意味着有一条尚未走过的路径。

```py
display_trace_tree(scf.ct.root) 
```

<svg width="674pt" height="603pt" viewBox="0.00 0.00 673.75 602.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 598.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="563.38" y="-581.45" font-family="Times,serif" font-size="14.00">(0) Length(cgi_decode_s_str_1) <= 0</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="481.38" y="-516.95" font-family="Times,serif" font-size="14.00">(1) str.substr(cgi_decode_s_str_1, 0, 1) == "+"</text></g> <g id="edge1" class="edge"><title>0->1</title> <text text-anchor="middle" x="533.09" y="-549.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="644.38" y="-516.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge18" class="edge"><title>0->18</title> <text text-anchor="middle" x="614.51" y="-549.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="398.38" y="-452.45" font-family="Times,serif" font-size="14.00">(2) str.substr(cgi_decode_s_str_1, 0, 1) == "%"</text></g> <g id="edge2" class="edge"><title>1->2</title> <text text-anchor="middle" x="450.68" y="-484.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="563.38" y="-452.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge17" class="edge"><title>1->17</title> <text text-anchor="middle" x="533.09" y="-484.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="329.38" y="-387.95" font-family="Times,serif" font-size="14.00">(3) Length(cgi_decode_s_str_1) <= 1</text></g> <g id="edge3" class="edge"><title>2->3</title> <text text-anchor="middle" x="373.43" y="-420.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="466.38" y="-387.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge16" class="edge"><title>2->16</title> <text text-anchor="middle" x="441.84" y="-420.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="247.38" y="-323.45" font-family="Times,serif" font-size="14.00">(4) str.substr(cgi_decode_s_str_1, 1, 1) == "+"</text></g> <g id="edge4" class="edge"><title>3->4</title> <text text-anchor="middle" x="299.09" y="-355.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="410.38" y="-323.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge15" class="edge"><title>3->15</title> <text text-anchor="middle" x="380.51" y="-355.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="232.38" y="-258.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge5" class="edge"><title>4->5</title> <text text-anchor="middle" x="244.59" y="-291.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="369.38" y="-258.95" font-family="Times,serif" font-size="14.00">(5) Length(cgi_decode_s_str_1) <= 2</text></g> <g id="edge6" class="edge"><title>4->6</title> <text text-anchor="middle" x="322.68" y="-291.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="254.38" y="-194.45" font-family="Times,serif" font-size="14.00">(6) str.substr(cgi_decode_s_str_1, 2, 1) == "+"</text></g> <g id="edge7" class="edge"><title>6->7</title> <text text-anchor="middle" x="325.55" y="-226.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="417.38" y="-194.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge14" class="edge"><title>6->14</title> <text text-anchor="middle" x="401.05" y="-226.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="171.38" y="-129.95" font-family="Times,serif" font-size="14.00">(7) str.substr(cgi_decode_s_str_1, 2, 1) == "%"</text></g> <g id="edge8" class="edge"><title>7->8</title> <text text-anchor="middle" x="223.68" y="-162.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="336.38" y="-129.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge13" class="edge"><title>7->13</title> <text text-anchor="middle" x="306.09" y="-162.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="102.38" y="-65.45" font-family="Times,serif" font-size="14.00">(8) Length(cgi_decode_s_str_1) <= 3</text></g> <g id="edge9" class="edge"><title>8->9</title> <text text-anchor="middle" x="146.43" y="-97.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="239.38" y="-65.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge12" class="edge"><title>8->12</title> <text text-anchor="middle" x="214.84" y="-97.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="77.38" y="-0.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge10" class="edge"><title>9->10</title> <text text-anchor="middle" x="95.49" y="-33.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="127.38" y="-0.95" font-family="Times,serif" font-size="14.00">* a+c</text></g> <g id="edge11" class="edge"><title>9->11</title> <text text-anchor="middle" x="120.49" y="-33.2" font-family="Times,serif" font-size="14.00">1</text></g></g></svg>

因此，我们进行模糊测试以获取一个非空的新路径。

```py
v = scf.fuzz()
print(v) 
```

```py
A+

```

我们现在可以像之前一样获得新的跟踪信息。

```py
with ExpectError():
    with ConcolicTracer() as _:
        _cgi_decode 
```

使用`add_trace()`将新的跟踪信息添加到我们的模糊测试器中。

```py
scf.add_trace(_, v) 
```

更新的二叉树如下。注意`Root`节点的子节点之间的差异。

```py
display_trace_tree(scf.ct.root) 
```

<svg width="673pt" height="603pt" viewBox="0.00 0.00 672.75 602.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 598.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="562.38" y="-581.45" font-family="Times,serif" font-size="14.00">(0) Length(cgi_decode_s_str_1) <= 0</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="480.38" y="-516.95" font-family="Times,serif" font-size="14.00">(1) str.substr(cgi_decode_s_str_1, 0, 1) == "+"</text></g> <g id="edge1" class="edge"><title>0->1</title> <text text-anchor="middle" x="532.09" y="-549.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="643.38" y="-516.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge18" class="edge"><title>0->18</title> <text text-anchor="middle" x="613.51" y="-549.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="397.38" y="-452.45" font-family="Times,serif" font-size="14.00">(2) str.substr(cgi_decode_s_str_1, 0, 1) == "%"</text></g> <g id="edge2" class="edge"><title>1->2</title> <text text-anchor="middle" x="449.68" y="-484.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="562.38" y="-452.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge17" class="edge"><title>1->17</title> <text text-anchor="middle" x="532.09" y="-484.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="328.38" y="-387.95" font-family="Times,serif" font-size="14.00">(3) Length(cgi_decode_s_str_1) <= 1</text></g> <g id="edge3" class="edge"><title>2->3</title> <text text-anchor="middle" x="372.43" y="-420.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="465.38" y="-387.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge16" class="edge"><title>2->16</title> <text text-anchor="middle" x="440.84" y="-420.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="246.38" y="-323.45" font-family="Times,serif" font-size="14.00">(4) str.substr(cgi_decode_s_str_1, 1, 1) == "+"</text></g> <g id="edge4" class="edge"><title>3->4</title> <text text-anchor="middle" x="298.09" y="-355.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="409.38" y="-323.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge15" class="edge"><title>3->15</title> <text text-anchor="middle" x="379.51" y="-355.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="231.38" y="-258.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge5" class="edge"><title>4->5</title> <text text-anchor="middle" x="243.59" y="-291.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="368.38" y="-258.95" font-family="Times,serif" font-size="14.00">(5) Length(cgi_decode_s_str_1) <= 2</text></g> <g id="edge6" class="edge"><title>4->6</title> <text text-anchor="middle" x="321.68" y="-291.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="254.38" y="-194.45" font-family="Times,serif" font-size="14.00">(6) str.substr(cgi_decode_s_str_1, 2, 1) == "+"</text></g> <g id="edge7" class="edge"><title>6->7</title> <text text-anchor="middle" x="324.96" y="-226.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="415.38" y="-194.45" font-family="Times,serif" font-size="14.00">* A+</text></g> <g id="edge14" class="edge"><title>6->14</title> <text text-anchor="middle" x="399.46" y="-226.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="171.38" y="-129.95" font-family="Times,serif" font-size="14.00">(7) str.substr(cgi_decode_s_str_1, 2, 1) == "%"</text></g> <g id="edge8" class="edge"><title>7->8</title> <text text-anchor="middle" x="223.68" y="-162.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="336.38" y="-129.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge13" class="edge"><title>7->13</title> <text text-anchor="middle" x="306.09" y="-162.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="102.38" y="-65.45" font-family="Times,serif" font-size="14.00">(8) Length(cgi_decode_s_str_1) <= 3</text></g> <g id="edge9" class="edge"><title>8->9</title> <text text-anchor="middle" x="146.43" y="-97.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="239.38" y="-65.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge12" class="edge"><title>8->12</title> <text text-anchor="middle" x="214.84" y="-97.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="77.38" y="-0.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge10" class="edge"><title>9->10</title> <text text-anchor="middle" x="95.49" y="-33.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="127.38" y="-0.95" font-family="Times,serif" font-size="14.00">* a+c</text></g> <g id="edge11" class="edge"><title>9->11</title> <text text-anchor="middle" x="120.49" y="-33.2" font-family="Times,serif" font-size="14.00">1</text></g></g></svg>

完整的模糊测试运行如下：

```py
scf = SimpleConcolicFuzzer()
for i in range(10):
    v = scf.fuzz()
    print(repr(v))
    if v is None:
        continue
    with ConcolicTracer() as _:
        with ExpectError(print_traceback=False):
            # z3.StringVal(urllib.parse.unquote('%80')) <-- bug in z3
            _cgi_decode
    scf.add_trace(_, v) 
```

```py
' '
''
'+'
'%'
'+A'
'++'
'AB'
'++A'
'A%'
'+AB'

```

```py
IndexError: string index out of range (expected)
IndexError: string index out of range (expected)

```

```py
display_trace_tree(scf.ct.root) 
```

<svg width="981pt" height="603pt" viewBox="0.00 0.00 980.88 602.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 598.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="638.38" y="-581.45" font-family="Times,serif" font-size="14.00">(0) Length(cgi_decode_s_str_1) <= 0</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="562.38" y="-516.95" font-family="Times,serif" font-size="14.00">(1) str.substr(cgi_decode_s_str_1, 0, 1) == "+"</text></g> <g id="edge1" class="edge"><title>0->1</title> <text text-anchor="middle" x="610.56" y="-549.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node37" class="node"><title>36</title> <text text-anchor="middle" x="714.38" y="-516.95" font-family="Times,serif" font-size="14.00">*</text></g> <g id="edge36" class="edge"><title>0->36</title> <text text-anchor="middle" x="686.56" y="-549.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="437.38" y="-452.45" font-family="Times,serif" font-size="14.00">(2) str.substr(cgi_decode_s_str_1, 0, 1) == "%"</text></g> <g id="edge2" class="edge"><title>1->2</title> <text text-anchor="middle" x="514.45" y="-484.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="688.38" y="-452.45" font-family="Times,serif" font-size="14.00">(2) Length(cgi_decode_s_str_1) <= 1</text></g> <g id="edge13" class="edge"><title>1->13</title> <text text-anchor="middle" x="640.04" y="-484.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="328.38" y="-387.95" font-family="Times,serif" font-size="14.00">(3) Length(cgi_decode_s_str_1) <= 1</text></g> <g id="edge3" class="edge"><title>2->3</title> <text text-anchor="middle" x="396.01" y="-420.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="460.38" y="-387.95" font-family="Times,serif" font-size="14.00">* %</text></g> <g id="edge12" class="edge"><title>2->12</title> <text text-anchor="middle" x="454.31" y="-420.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="291.38" y="-323.45" font-family="Times,serif" font-size="14.00">(4) str.substr(cgi_decode_s_str_1, 1, 1) == "+"</text></g> <g id="edge4" class="edge"><title>3->4</title> <text text-anchor="middle" x="316.56" y="-355.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="445.38" y="-323.45" font-family="Times,serif" font-size="14.00">*  </text></g> <g id="edge11" class="edge"><title>3->11</title> <text text-anchor="middle" x="400.73" y="-355.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="185.38" y="-258.95" font-family="Times,serif" font-size="14.00">(5) str.substr(cgi_decode_s_str_1, 1, 1) == "%"</text></g> <g id="edge5" class="edge"><title>4->5</title> <text text-anchor="middle" x="251.24" y="-291.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="350.38" y="-258.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge10" class="edge"><title>4->10</title> <text text-anchor="middle" x="329.53" y="-291.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="102.38" y="-194.45" font-family="Times,serif" font-size="14.00">(6) Length(cgi_decode_s_str_1) <= 2</text></g> <g id="edge6" class="edge"><title>5->6</title> <text text-anchor="middle" x="154.68" y="-226.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="238.38" y="-194.45" font-family="Times,serif" font-size="14.00">* A%</text></g> <g id="edge9" class="edge"><title>5->9</title> <text text-anchor="middle" x="220" y="-226.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="88.38" y="-129.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge7" class="edge"><title>6->7</title> <text text-anchor="middle" x="100" y="-162.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="136.38" y="-129.95" font-family="Times,serif" font-size="14.00">* AB</text></g> <g id="edge8" class="edge"><title>6->8</title> <text text-anchor="middle" x="125.8" y="-162.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="667.38" y="-387.95" font-family="Times,serif" font-size="14.00">(3) str.substr(cgi_decode_s_str_1, 1, 1) == "+"</text></g> <g id="edge14" class="edge"><title>13->14</title> <text text-anchor="middle" x="683.13" y="-420.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node36" class="node"><title>35</title> <text text-anchor="middle" x="823.38" y="-387.95" font-family="Times,serif" font-size="14.00">* +</text></g> <g id="edge35" class="edge"><title>13->35</title> <text text-anchor="middle" x="771.34" y="-420.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="611.38" y="-323.45" font-family="Times,serif" font-size="14.00">(4) str.substr(cgi_decode_s_str_1, 1, 1) == "%"</text></g> <g id="edge15" class="edge"><title>14->15</title> <text text-anchor="middle" x="647.77" y="-355.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="862.38" y="-323.45" font-family="Times,serif" font-size="14.00">(4) Length(cgi_decode_s_str_1) <= 2</text></g> <g id="edge26" class="edge"><title>14->26</title> <text text-anchor="middle" x="785.72" y="-355.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="495.38" y="-258.95" font-family="Times,serif" font-size="14.00">(5) Length(cgi_decode_s_str_1) <= 2</text></g> <g id="edge16" class="edge"><title>15->16</title> <text text-anchor="middle" x="567.14" y="-291.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="632.38" y="-258.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge25" class="edge"><title>15->25</title> <text text-anchor="middle" x="627.13" y="-291.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="449.38" y="-194.45" font-family="Times,serif" font-size="14.00">(6) str.substr(cgi_decode_s_str_1, 2, 1) == "+"</text></g> <g id="edge17" class="edge"><title>16->17</title> <text text-anchor="middle" x="479.87" y="-226.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="610.38" y="-194.45" font-family="Times,serif" font-size="14.00">* +A</text></g> <g id="edge24" class="edge"><title>16->24</title> <text text-anchor="middle" x="566.55" y="-226.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="349.38" y="-129.95" font-family="Times,serif" font-size="14.00">(7) str.substr(cgi_decode_s_str_1, 2, 1) == "%"</text></g> <g id="edge18" class="edge"><title>17->18</title> <text text-anchor="middle" x="411.71" y="-162.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="514.38" y="-129.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge23" class="edge"><title>17->23</title> <text text-anchor="middle" x="491.07" y="-162.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="254.38" y="-65.45" font-family="Times,serif" font-size="14.00">(8) Length(cgi_decode_s_str_1) <= 3</text></g> <g id="edge19" class="edge"><title>18->19</title> <text text-anchor="middle" x="313.76" y="-97.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="391.38" y="-65.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge22" class="edge"><title>18->22</title> <text text-anchor="middle" x="377.51" y="-97.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="228.38" y="-0.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge20" class="edge"><title>19->20</title> <text text-anchor="middle" x="247.08" y="-33.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="281.38" y="-0.95" font-family="Times,serif" font-size="14.00">* +AB</text></g> <g id="edge21" class="edge"><title>19->21</title> <text text-anchor="middle" x="273.67" y="-33.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="799.38" y="-258.95" font-family="Times,serif" font-size="14.00">(5) str.substr(cgi_decode_s_str_1, 2, 1) == "+"</text></g> <g id="edge27" class="edge"><title>26->27</title> <text text-anchor="middle" x="839.89" y="-291.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node35" class="node"><title>34</title> <text text-anchor="middle" x="959.38" y="-258.95" font-family="Times,serif" font-size="14.00">* ++</text></g> <g id="edge34" class="edge"><title>26->34</title> <text text-anchor="middle" x="922.94" y="-291.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="780.38" y="-194.45" font-family="Times,serif" font-size="14.00">(6) str.substr(cgi_decode_s_str_1, 2, 1) == "%"</text></g> <g id="edge28" class="edge"><title>27->28</title> <text text-anchor="middle" x="794.95" y="-226.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node34" class="node"><title>33</title> <text text-anchor="middle" x="945.38" y="-194.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge33" class="edge"><title>27->33</title> <text text-anchor="middle" x="888.83" y="-226.7" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="676.38" y="-129.95" font-family="Times,serif" font-size="14.00">(7) Length(cgi_decode_s_str_1) <= 3</text></g> <g id="edge29" class="edge"><title>28->29</title> <text text-anchor="middle" x="741.07" y="-162.2" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="813.38" y="-129.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge32" class="edge"><title>28->32</title> <text text-anchor="middle" x="803.21" y="-162.2" font-family="Times,serif" font-size="14.00">1</text></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="649.38" y="-65.45" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge30" class="edge"><title>29->30</title> <text text-anchor="middle" x="668.67" y="-97.7" font-family="Times,serif" font-size="14.00">0</text></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="702.38" y="-65.45" font-family="Times,serif" font-size="14.00">* ++A</text></g> <g id="edge31" class="edge"><title>29->31</title> <text text-anchor="middle" x="695.08" y="-97.7" font-family="Times,serif" font-size="14.00">1</text></g></g></svg>

**注意。**我们的 concolic 跟踪器有限制，因为它不跟踪字符串长度的变化。这导致它将具有相同前缀的每个字符串视为相同的字符串。

`SimpleConcolicFuzzer`在探索给定样本输入路径附近的路径方面相当高效。然而，在选择要遵循的路径时，它并不非常智能。我们来看看另一个将获得的谓词提升到语法的模糊测试器，并实现更好的模糊测试。

## Concolic 语法模糊测试

Concolic 框架可以直接用于基于语法的模糊测试。我们实现了一个名为`ConcolicGrammarFuzzer`的类来完成这项工作。

<details id="Excursion:-Implementing-ConcolicGrammarFuzzer"><summary>实现 ConcolicGrammarFuzzer</summary>

首先，我们扩展我们的`GrammarFuzzer`，添加一个辅助方法`tree_to_string()`，以便我们可以检索模糊输出的推导树。我们还定义了`prune_tree()`和`coalesce()`方法来减少子树的深度。这些方法接受一个标记类型列表，使得属于标记类型的节点通过调用`tree_to_string()`从树转换为叶节点。

```py
from InformationFlow import INVENTORY_GRAMMAR, SQLException 
```

```py
from GrammarFuzzer import GrammarFuzzer 
```

```py
class ConcolicGrammarFuzzer(GrammarFuzzer):
    def tree_to_string(self, tree):
        symbol, children, *_ = tree
        e = ''
        if children:
            return e.join([self.tree_to_string(c) for c in children])
        else:
            return e if symbol in self.grammar else symbol

    def prune_tree(self, tree, tokens):
        name, children = tree
        children = self.coalesce(children)
        if name in tokens:
            return (name, [(self.tree_to_string(tree), [])])
        else:
            return (name, [self.prune_tree(c, tokens) for c in children])

    def coalesce(self, children):
        last = ''
        new_lst = []
        for cn, cc in children:
            if cn not in self.grammar:
                last += cn
            else:
                if last:
                    new_lst.append((last, []))
                    last = ''
                new_lst.append((cn, cc))
        if last:
            new_lst.append((last, []))
        return new_lst 
```

我们现在可以使用模糊测试器为我们的数据库生成输入。

```py
tgf = ConcolicGrammarFuzzer(INVENTORY_GRAMMAR)
while True:
    qtree = tgf.fuzz_tree()
    query = str(tgf.tree_to_string(qtree))
    if query.startswith('select'):
        break 
```

```py
from ExpectError import ExpectError 
```

```py
with ExpectError():
    print(repr(query))
    with ConcolicTracer() as _:
        res = _db_select)
    print(repr(res)) 
```

```py
'select t4(I,N)!=b(k)/O!=(K4(:/Z)) from I7'

```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34717/2536269233.py", line 4, in <module>
    res = _db_select)
          ^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34717/2687284210.py", line 3, in __call__
    self.result = self.fn(*self.concolic(args))
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34717/1994573112.py", line 4, in db_select
    r = my_db.sql(s)
        ^^^^^^^^^^^^
  File "InformationFlow.ipynb", line 65, in sql
    return method(query[len(key):])
           ^^^^^^^^^^^^^^^^^^^^^^^^
  File "InformationFlow.ipynb", line 84, in do_select
    _, table = self.table(t_name)
               ^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34717/2474817571.py", line 6, in table
    raise SQLException('Table (%s) was not found' % repr(t_name))
InformationFlow.SQLException: Table ('I7') was not found (expected)

```

我们的模糊测试器返回时抛出异常。它无法找到指定的表。让我们检查它遇到的谓词。

```py
for i, p in enumerate(_.path):
    print(i, p) 
```

```py
0 0 == IndexOf(db_select_s_str_1, "select ", 0)
1 0 == IndexOf(db_select_s_str_1, "select ", 0)
2 Not(0 >
    IndexOf(str.substr(db_select_s_str_1, 7, 34),
            " from ",
            0))
3 Not(Or(0 <
       IndexOf(str.substr(db_select_s_str_1, 7, 34),
               " where ",
               0),
       0 ==
       IndexOf(str.substr(db_select_s_str_1, 7, 34),
               " where ",
               0)))
4 Not(str.substr(str.substr(db_select_s_str_1, 7, 34), 32, 2) ==
    "inventory")

```

注意，我们可以通过使用`ConcolicTracer`获得语法中不存在的约束。特别是，看看我们如何能够获得表需要是`inventory`（谓词 11）的条件，以便模糊测试成功。

我们如何将这些内容提升到语法中？特别是如何自动完成？我们有一个选项是简单地切换最后获得的谓词。在我们的例子中，最后的谓词是（11）。我们能否简单地反转谓词并再次求解？

```py
new_path = _.path[0:-1] + [z3.Not(_.path[-1])] 
```

```py
new_ = ConcolicTracer((_.decls, new_path))
new_.fn = _.fn
new_.fn_args = _.fn_args 
```

```py
new_.zeval() 
```

```py
('No Solutions', None)

```

事实上，这不会起作用，因为正在比较的字符串长度不同。

```py
print(_.path[-1])
z3.solve(z3.Not(_.path[-1])) 
```

```py
Not(str.substr(str.substr(db_select_s_str_1, 7, 34), 32, 2) ==
    "inventory")
no solution

```

一个更好的想法是调查正在进行的什么*字符串*比较，并将其与语法中的相应节点关联起来。让我们检查我们的推导树（修剪以避免递归结构，并专注于重要部分）。

```py
from GrammarFuzzer import display_tree 
```

```py
prune_tokens = [
    '<value>', '<table>', '<column>', '<literals>', '<exprs>', '<bexpr>'
]
dt = tgf.prune_tree(qtree, prune_tokens)
display_tree(dt) 
```

<svg width="220pt" height="173pt" viewBox="0.00 0.00 219.62 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="104.25" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="104.25" y="-101.45" font-family="Times,serif" font-size="14.00"><query></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="17.25" y="-51.2" font-family="Times,serif" font-size="14.00">select</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="75.25" y="-51.2" font-family="Times,serif" font-size="14.00"><exprs></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="133.25" y="-51.2" font-family="Times,serif" font-size="14.00">from</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="190.25" y="-51.2" font-family="Times,serif" font-size="14.00"><table></text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="75.25" y="-0.95" font-family="Times,serif" font-size="14.00">t4(I,N)!=b(k)/O!=(K4(:/Z))</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="190.25" y="-0.95" font-family="Times,serif" font-size="14.00">I7</text></g> <g id="edge7" class="edge"><title>6->7</title></g></g></svg>

我们能否识别输入的哪一部分是由语法的哪一部分提供的？我们定义了`span()`，可以从推导树中恢复此信息。对于给定的节点，让我们假设起点是已知的。然后，在处理子节点时，我们按以下方式进行：我们一次选择一个子节点，从左到右进行，并计算子节点的长度。当前子节点之前的子节点长度加上我们的起点给出了当前子节点的起点。每个节点的终点简单地是其最后一个子节点的终点（或其节点长度，如果它是叶子节点）。

```py
from GrammarFuzzer import START_SYMBOL 
```

```py
def span(node, g, node_start=0):
    hm = {}
    k, cs = node
    end_i = node_start
    new_cs = []
    for c in cs:
        chm, (ck, child_start, child_end, gcs) = span(c, g, end_i)
        new_cs.append((ck, child_start, child_end, gcs))
        end_i = child_end
        hm.update(chm)
    node_end = end_i if cs else node_start + len(k)
    if k in g and k != START_SYMBOL:
        hm[k] = (node_start, node_end - node_start)
    return hm, (k, node_start, node_end, new_cs) 
```

我们可以这样使用它：

```py
span_hm, _n = span(dt, INVENTORY_GRAMMAR) 
```

```py
span_hm 
```

```py
{'<exprs>': (7, 26), '<table>': (39, 2), '<query>': (0, 41)}

```

我们可以通过以下方式检查我们是否得到了正确的值。

```py
print("query:", query)
for k in span_hm:
    start, l = span_hm[k]
    print(k, query[start:start + l]) 
```

```py
query: select t4(I,N)!=b(k)/O!=(K4(:/Z)) from I7
<exprs> t4(I,N)!=b(k)/O!=(K4(:/Z))
<table> I7
<query> select t4(I,N)!=b(k)/O!=(K4(:/Z)) from I7

```

接下来，我们需要获取每个谓词中做出的所有比较。为此，我们定义了两个辅助函数。第一个是`unwrap_substrings()`，它将多个对`z3.SubString`的调用转换为给定的 z3 字符串表达式的起始位置和长度。

```py
def unwrap_substrings(s):
    assert s.decl().name() == 'str.substr'
    cs, frm, l = s.children()
    fl = frm.as_long()
    ll = l.as_long()
    if cs.decl().name() == 'str.substr':
        newfrm, _l = unwrap_substrings(cs)
        return (fl + newfrm, ll)
    else:
        return (fl, ll) 
```

我们定义了`traverse_z3()`，它遍历给定的 z3 字符串表达式，并收集所有直接字符串比较到原始参数的子串。

```py
def traverse_z3(p, hm):
    def z3_as_string(v):
        return v.as_string()

    n = p.decl().name()
    if n == 'not':
        return traverse_z3(p.children()[0], hm)
    elif n == '=':
        i, j = p.children()
        if isinstance(i, (int, z3.IntNumRef)):
            return traverse_z3(j, hm)
        elif isinstance(j, (int, z3.IntNumRef)):
            return traverse_z3(i, hm)
        else:
            if i.is_string() and j.is_string():
                if i.is_string_value():
                    cs, frm, l = j.children()
                    if (isinstance(frm, z3.IntNumRef)
                            and isinstance(l, z3.IntNumRef)):
                        hm[z3_as_string(i)] = unwrap_substrings(j)
                elif j.is_string_value():
                    cs, frm, l = i.children()
                    if (isinstance(frm, z3.IntNumRef)
                            and isinstance(l, z3.IntNumRef)):
                        hm[z3_as_string(j)] = unwrap_substrings(i)
            else:
                assert False  # for now
    elif n == '<' or n == '>':
        i, j = p.children()
        if isinstance(i, (int, z3.IntNumRef)):
            return traverse_z3(j, hm)
        elif isinstance(j, (int, z3.IntNumRef)):
            return traverse_z3(i, hm)
        else:
            assert False
    return p 
```

```py
comparisons: Dict[str, Tuple] = {}
for p in _.path:
    traverse_z3(p, comparisons)
comparisons 
```

```py
{'inventory': (39, 2)}

```

我们现在需要声明与`comparisons`中的子串匹配的字符串变量，并为路径中的每个项目求解它们。为此，我们定义了`find_alternatives()`。

```py
def find_alternatives(spans, cmp):
    alts = {}
    for key in spans:
        start, l = spans[key]
        rset = set(range(start, start + l))
        for ckey in cmp:
            cstart, cl = cmp[ckey]
            cset = set(range(cstart, cstart + cl))
            # if rset.issubset(cset): <- ignoring subsets for now.
            if rset == cset:
                if key not in alts:
                    alts[key] = set()
                alts[key].add(ckey)
    return alts 
```

我们可以这样使用它。

```py
alternatives = find_alternatives(span_hm, comparisons)
alternatives 
```

```py
{'<table>': {'inventory'}}

```

因此，我们为语法中的每个键有了我们的备选方案。我们现在可以按以下方式更新我们的语法。

```py
INVENTORY_GRAMMAR_NEW = dict(INVENTORY_GRAMMAR) 
```

```py
for k in alternatives:
    INVENTORY_GRAMMAR_NEW[k] = INVENTORY_GRAMMAR_NEW[k] + list(alternatives[k]) 
```

在这里，我们做出了一个选择。我们本可以完全覆盖`<table>`的定义。相反，我们添加了我们的新备选方案到现有定义中。这样，我们的模糊测试器也会偶尔尝试`<table>`的其他值。

```py
INVENTORY_GRAMMAR_NEW['<table>'] 
```

```py
['<word>', 'inventory']

```

让我们尝试用我们新的语法进行模糊测试。

```py
cgf = ConcolicGrammarFuzzer(INVENTORY_GRAMMAR_NEW) 
```

```py
for i in range(10):
    qtree = cgf.fuzz_tree()
    query = cgf.tree_to_string(qtree)
    print(query)
    with ExpectError(print_traceback=False):
        try:
            with ConcolicTracer() as _:
                res = _db_select
            print(repr(res))
        except SQLException as e:
            print(e)
        print() 
```

```py
insert into inventory (i9Oam41gsP2,h97,q8J:.70J) values ('.q')
Column ('i9Oam41gsP2') was not found

select C from wy where R/s/y>_-X-.+C/u==(((---6.5)))
Table ('wy') was not found

update T set I=gj5 where (-8.6/O*.-W)==s-O<Z((R),((N==(:))))
Table ('T') was not found

update inventory set f=o,V=Q6 where l0!=(((((-5)))==(rGJ)))
Column ('f') was not found

delete from j where T==5.58
Table ('j') was not found

insert into inventory (py75) values ('R','/','fd8g',3883.0)
Column ('py75') was not found

update inventory set GY5=X where G-g/z(w)<2.5
Column ('GY5') was not found

update nb0 set i=K,b=R,u=: where D>A
Table ('nb0') was not found

insert into inventory (P,wmE,U,F) values (50,'/',--6.2)
Column ('P') was not found

delete from GTV3_ where :-M!=t>n+R/x+r*a/t-r-V
Table ('GTV3_') was not found

```

即，我们能够到达危险的方法`my_eval()`。实际上，我们所做的是将谓词的部分提升到语法中。新的语法可以生成比以前更深入程序的输入。请注意，我们只处理了相等谓词。如果需要，也可以将`<`和`>`比较运算符提升到语法中。

将我们的模糊测试器的输出与下面的原始`GrammarFuzzer`进行比较。

```py
gf = GrammarFuzzer(INVENTORY_GRAMMAR)
for i in range(10):
    query = gf.fuzz()
    print(query)
    with ExpectError(print_traceback=False):
        try:
            res = db_select(query)
            print(repr(res))
        except SQLException as e:
            print(e)
        print() 
```

```py
insert into UCu4 (E,xM:lOq6,u38p,W54G3b0) values (':',1.835)
Table ('UCu4') was not found

insert into B81 (Np) values ('h')
Table ('B81') was not found

delete from w where Xn((T))>a(8.8,g)/h+t-P-j+L
Table ('w') was not found

update q75 set L=z4 where ((QUy+N))==A/P-L*ao(R)/I
Table ('q75') was not found

update q3 set x=F where l(N)-P-S+t==e
Table ('q3') was not found

update Dy06rr set h=F where (z!=Q)==(((a<q)))
Table ('Dy06rr') was not found

delete from Z where V(7)>a==O(mA,j(g)*:,s,B)-5-eD(c,F!=n)==eO41Xy
Table ('Z') was not found

select 1.8 from U3X8p
Table ('U3X8p') was not found

update N set w=X9,A=w,M=Z where ((b!=c))==U/N<I
Table ('N') was not found

insert into I1 (g,_Q,y8e0) values (5.7,'.&',05.2)
Table ('I1') was not found

```

如所示，原始语法模糊测试器无法超越表验证。

#### 修剪和更新

我们在`ConcolicGrammarFuzzer`中实现了这些方法。`update_grammar()`方法允许`ConcolicGrammarFuzzer`从 concolic 模糊测试中收集反馈，并相应地更新用于模糊测试的语法。

```py
class ConcolicGrammarFuzzer(ConcolicGrammarFuzzer):
    def prune_tokens(self, tokens):
        self.prune_tokens = tokens

    def update_grammar(self, trace):
        self.comparisons = {}
        for p in trace.path:
            traverse_z3(p, self.comparisons)
        alternatives = find_alternatives(self.span_range, self.comparisons)
        if self.log:
            print('Alternatives:', alternatives, 'Span:', self.span_range)
        new_grammar = dict(self.grammar)
        for k in alternatives:
            new_grammar[k] = list(set(new_grammar[k] + list(alternatives[k])))
        self.grammar = new_grammar 
```

`fuzz()`方法简单地生成推导树，计算跨度范围，并返回从推导树生成的字符串。

```py
class ConcolicGrammarFuzzer(ConcolicGrammarFuzzer):
    def fuzz(self):
        qtree = self.fuzz_tree()
        self.pruned_tree = self.prune_tree(qtree, self.prune_tokens)
        query = self.tree_to_string(qtree)
        self.span_range, _n = span(self.pruned_tree, self.grammar)
        return query 
```

为了确保我们的方法有效，让我们稍微更新我们的表。

```py
inventory = db.db.pop('inventory', None) 
```

```py
db.db['vehicles'] = inventory
db.db['months'] = ({
    'month': int,
    'name': str
}, [{
    'month': i + 1,
    'name': m
} for i, m in enumerate([
    'jan', 'feb', 'mar', 'apr', 'may', 'jun', 'jul', 'aug', 'sep', 'oct',
    'nov', 'dec'
])])
db.db 
```

```py
{'vehicles': ({'year': int, 'kind': str, 'company': str, 'model': str},
  [{'year': 1997, 'kind': 'van', 'company': 'Ford', 'model': 'E350'},
   {'year': 2000, 'kind': 'car', 'company': 'Mercury', 'model': 'Cougar'},
   {'year': 1999, 'kind': 'car', 'company': 'Chevy', 'model': 'Venture'}]),
 'months': ({'month': int, 'name': str},
  [{'month': 1, 'name': 'jan'},
   {'month': 2, 'name': 'feb'},
   {'month': 3, 'name': 'mar'},
   {'month': 4, 'name': 'apr'},
   {'month': 5, 'name': 'may'},
   {'month': 6, 'name': 'jun'},
   {'month': 7, 'name': 'jul'},
   {'month': 8, 'name': 'aug'},
   {'month': 9, 'name': 'sep'},
   {'month': 10, 'name': 'oct'},
   {'month': 11, 'name': 'nov'},
   {'month': 12, 'name': 'dec'}])}

```</details>

`ConcolicGrammarFuzzer`的使用如下。

```py
cgf = ConcolicGrammarFuzzer(INVENTORY_GRAMMAR)
cgf.prune_tokens(prune_tokens)
for i in range(10):
    query = cgf.fuzz()
    print(query)
    with ConcolicTracer() as _:
        with ExpectError(print_traceback=False):
            try:
                res = _db_select
                print(repr(res))
            except SQLException as e:
                print(e)
        cgf.update_grammar(_)
        print() 
```

```py
select Qq6L,(X) from LYg0 where ((x<w))!=(A)
Table ('LYg0') was not found

update vehicles set l=b,E=u,v=E,h=I where (N)==W*i*_-x
Column ('l') was not found

update Xw set h=w,x=w,F=l,U=g where R==j<o
Table ('Xw') was not found

insert into months (q) values ('*','u',43.6)
Column ('q') was not found

select y-A-F+x>Q/b+i==j==w!=r,j,d from months
Invalid WHERE ('(y-A-F+x>Q/b+i==j==w!=r,j,d)')

select ((J/F-K-M+w*n)),(:<O==(f)) from vehicles
Invalid WHERE ('(((J/F-K-M+w*n)),(:<O==(f)))')

insert into months (Ui) values (82)
Column ('Ui') was not found

insert into Zn1 (month) values (5)
Table ('Zn1') was not found

update b set month=t,month=cY,name=p,name=T where S/O==-2
Table ('b') was not found

delete from vehicles where (Q)+s(t)-B(n,E)>T/i-E(u)
Invalid WHERE ('(Q)+s(t)-B(n,E)>T/i-E(u)')

```

如所示，模糊测试器开始时对`vehicles`、`months`和`years`表没有任何了解，但它从 concolic 执行中识别出来，并将其提升到语法中。这使我们能够提高模糊测试的有效性。

## 局限性

就像动态污点分析一样，隐式控制流可能会在冲突执行中掩盖遇到的谓词。然而，这种限制可以通过将源中的任何常量与其相应的代理对象包装起来在一定程度上克服。同样，调用内部 C 函数可能会导致符号信息丢失，并且只能获得部分信息。

## 经验教训

+   冲突执行在程序行为方面可以提供比污点分析更多的信息。然而，这需要更大的运行时成本。因此，与污点分析不同，实时分析通常是不可能的。

+   与污点分析类似，冲突执行也受到诸如间接控制流和内部函数调用等限制。

+   冲突执行的谓词可以与模糊测试结合使用，以提供比污点更稳健的不正确行为指示，并且可以用于创建更擅长生成有效输入的语法。

## 下一步

相比于冲突模糊测试，符号模糊测试是一个成本更高但更强的替代方案。同样，基于搜索的模糊测试通常比依赖于 SMT 求解器提供与当前路径略有不同的输入的探索策略更便宜。

## 背景

冲突执行技术最初用于告知并扩展符号执行*的*范围 [[King *et al*, 1976](https://doi.org/10.1145/360248.360252)]，这是一种用于程序分析的静态分析技术。Laron 等人引用{Larson2003}是第一个使用冲突执行技术的人。

使用代理对象收集约束的想法是由 Cadar 等人开创的[Cadar *et al*, 2005]。本章中使用的 Python 程序冲突执行技术是由 PeerCheck [[A. Bruni *et al*, 2011](https://hoheinzollern.files.wordpress.com/2008/04/seer1.pdf)]和 Python Error Finder [[Damián Barsotti *et al*, 2018](https://doi.org/https://doi.org/10.1016/j.entcs.2018.06.003)]开创的。

## 练习

### 练习 1：实现冲突浮点代理类

在实现`zint`二进制运算符时，我们断言结果是`int`。然而，情况并不一定如此。例如，除法可能得到`float`。因此，我们需要为`float`创建代理对象。你能实现一个类似的`float`代理对象并修复`zint`二进制运算符的定义吗？

**解决方案**。解决方案如下。

就像`zint`的情况一样，我们首先为`zfloat`扩展打开。

```py
class zfloat(float):
    def __new__(cls, context, zn, v, *args, **kw):
        return float.__new__(cls, v, *args, **kw) 
```

然后我们实现初始化方法。

```py
class zfloat(zfloat):
    @classmethod
    def create(cls, context, zn, v=None):
        return zproxy_create(cls, 'Real', z3.Real, context, zn, v)

    def __init__(self, context, z, v=None):
        self.z, self.v = z, v
        self.context = context 
```

当二进制操作中的一个参数不是`float`时的辅助方法。

```py
class zfloat(zfloat):
    def _zv(self, o):
        return (o.z, o.v) if isinstance(o, zfloat) else (z3.RealVal(o), o) 
```

将`float`强制转换为布尔值以用于条件语句。

```py
class zfloat(zfloat):
    def __bool__(self):
        # force registering boolean condition
        if self != 0.0:
            return True
        return False 
```

定义比较方法的通用代理方法

```py
def make_float_bool_wrapper(fname, fun, zfun):
    def proxy(self, other):
        z, v = self._zv(other)
        z_ = zfun(self.z, z)
        v_ = fun(self.v, v)
        return zbool(self.context, z_, v_)

    return proxy 
```

我们在定义的`zfloat`类上应用比较方法。

```py
FLOAT_BOOL_OPS = [
    '__eq__',
    # '__req__',
    '__ne__',
    # '__rne__',
    '__gt__',
    '__lt__',
    '__le__',
    '__ge__',
] 
```

```py
for fname in FLOAT_BOOL_OPS:
    fun = getattr(float, fname)
    zfun = getattr(z3.ArithRef, fname)
    setattr(zfloat, fname, make_float_bool_wrapper(fname, fun, zfun)) 
```

类似地，我们定义了二进制运算符的通用代理方法。

```py
def make_float_binary_wrapper(fname, fun, zfun):
    def proxy(self, other):
        z, v = self._zv(other)
        z_ = zfun(self.z, z)
        v_ = fun(self.v, v)
        return zfloat(self.context, z_, v_)

    return proxy 
```

在`zfloat`上应用它们

```py
FLOAT_BINARY_OPS = [
    '__add__',
    '__sub__',
    '__mul__',
    '__truediv__',
    # '__div__',
    '__mod__',
    # '__divmod__',
    '__pow__',
    # '__lshift__',
    # '__rshift__',
    # '__and__',
    # '__xor__',
    # '__or__',
    '__radd__',
    '__rsub__',
    '__rmul__',
    '__rtruediv__',
    # '__rdiv__',
    '__rmod__',
    # '__rdivmod__',
    '__rpow__',
    # '__rlshift__',
    # '__rrshift__',
    # '__rand__',
    # '__rxor__',
    # '__ror__',
] 
```

```py
for fname in FLOAT_BINARY_OPS:
    fun = getattr(float, fname)
    zfun = getattr(z3.ArithRef, fname)
    setattr(zfloat, fname, make_float_binary_wrapper(fname, fun, zfun)) 
```

这些用法如下。

```py
with ConcolicTracer() as _:
    za = zfloat.create(_.context, 'float_a', 1.0)
    zb = zfloat.create(_.context, 'float_b', 0.0)
    if za * zb:
        print(1) 
```

```py
_.context 
```

```py
({'float_a': 'Real', 'float_b': 'Real'}, [Not(float_a*float_b != 0)])

```

最后，我们修复 `zint` 二进制包装器，以便在需要时正确创建 `zfloat`。

```py
def make_int_binary_wrapper(fname, fun, zfun):
    def proxy(self, other):
        z, v = self._zv(other)
        z_ = zfun(self.z, z)
        v_ = fun(self.v, v)
        if isinstance(v_, float):
            return zfloat(self.context, z_, v_)
        elif isinstance(v_, int):
            return zint(self.context, z_, v_)
        else:
            assert False

    return proxy 
```

```py
for fname in INT_BINARY_OPS:
    fun = getattr(int, fname)
    zfun = getattr(z3.ArithRef, fname)
    setattr(zint, fname, make_int_binary_wrapper(fname, fun, zfun)) 
```

检查它是否按预期工作。

```py
with ConcolicTracer() as _:
    v = _binomial 
```

```py
_.zeval() 
```

```py
('sat', {'n': ('4', 'Int'), 'k': ('2', 'Int')})

```

### 练习 2：位操作

与浮点数类似，实现位操作函数如 `xor` 需要将 `int` 转换为其位向量等价物，对它们进行操作，然后再将其转换回原始类型。你能实现 `zint` 的位操作吗？

**解决方案**。解决方案如下。

我们首先定义代理方法，就像之前一样。

```py
def make_int_bit_wrapper(fname, fun, zfun):
    def proxy(self, other):
        z, v = self._zv(other)
        z_ = z3.BV2Int(
            zfun(
                z3.Int2BV(
                    self.z, num_bits=64), z3.Int2BV(
                    z, num_bits=64)))
        v_ = fun(self.v, v)
        return zint(self.context, z_, v_)

    return proxy 
```

然后将它应用于 `zint` 类。

```py
BIT_OPS = [
    '__lshift__',
    '__rshift__',
    '__and__',
    '__xor__',
    '__or__',
    '__rlshift__',
    '__rrshift__',
    '__rand__',
    '__rxor__',
    '__ror__',
] 
```

```py
def init_concolic_4():
    for fname in BIT_OPS:
        fun = getattr(int, fname)
        zfun = getattr(z3.BitVecRef, fname)
        setattr(zint, fname, make_int_bit_wrapper(fname, fun, zfun)) 
```

```py
INITIALIZER_LIST.append(init_concolic_4) 
```

```py
init_concolic_4() 
```

Invert 是唯一的单目位操作方法。

```py
class zint(zint):
    def __invert__(self):
        return zint(self.context, z3.BV2Int(
            ~z3.Int2BV(self.z, num_bits=64)), ~self.v) 
```

`my_fn()` 函数计算 `xor` 并在结果为非零值时返回 `True`。

```py
def my_fn(a, b):
    o_ = (a | b)
    a_ = (a & b)
    if o_ & ~a_:
        return True
    else:
        return False 
```

在 `ConcolicTracer` 下使用

```py
with ConcolicTracer() as _:
    print(_my_fn) 
```

```py
True

```

我们记录计算出的 SMT 表达式以验证一切是否顺利。

```py
_.zeval(log=True) 
```

```py
Predicates in path:
0 0 !=
BV2Int(int2bv(BV2Int(int2bv(my_fn_a_int_1) |
                     int2bv(my_fn_b_int_2))) &
       int2bv(BV2Int(~int2bv(BV2Int(int2bv(my_fn_a_int_1) &
                                    int2bv(my_fn_b_int_2))))))

(declare-const my_fn_a_int_1 Int)
(declare-const my_fn_b_int_2 Int)
(assert (let ((a!1 (bvnot (bvor (bvnot ((_ int2bv 64) my_fn_a_int_1))
                        (bvnot ((_ int2bv 64) my_fn_b_int_2))))))
(let ((a!2 (bvor (bvnot (bvor ((_ int2bv 64) my_fn_a_int_1)
                              ((_ int2bv 64) my_fn_b_int_2)))
                 a!1)))
  (not (= 0 (bv2int (bvnot a!2)))))))
(check-sat)
(get-model)

z3 -t:6000 /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpi5jlhyxm.smt
sat
(
  (define-fun my_fn_a_int_1 () Int
    (- 1))
  (define-fun my_fn_b_int_2 () Int
    (- 9223372036854775809))
)

```

```py
('sat', {'a': (['-', '1'], 'Int'), 'b': (['-', '9223372036854775809'], 'Int')})

```

我们可以从生成的公式中确认位操作函数是否正确工作。

### 练习 3：字符串转换函数

我们已经看到了如何定义 `upper()` 和 `lower()`。你能定义 `capitalize()`、`title()` 和 `swapcase()` 方法吗？

**解决方案**。解决方案尚未提供。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容根据 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可。内容的一部分源代码，以及用于格式化和显示该内容的源代码，根据 [MIT License](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 许可。最后更改日期：2024-11-09 17:07:29+01:00。引用 • [印记](https://cispa.de/en/impressum)

## 如何引用此作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[Concolic Fuzzing](https://www.fuzzingbook.org/html/ConcolicFuzzer.html)"。在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 的 "[The Fuzzing Book](https://www.fuzzingbook.org/)" 中，[`www.fuzzingbook.org/html/ConcolicFuzzer.html`](https://www.fuzzingbook.org/html/ConcolicFuzzer.html)。检索日期：2024-11-09 17:07:29+01:00。

```py
@incollection{fuzzingbook2024:ConcolicFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Concolic Fuzzing},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/ConcolicFuzzer.html}},
    note = {Retrieved 2024-11-09 17:07:29+01:00},
    url = {https://www.fuzzingbook.org/html/ConcolicFuzzer.html},
    urldate = {2024-11-09 17:07:29+01:00}
}

```
