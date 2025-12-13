# 带约束的模糊测试

> 原文：[`www.fuzzingbook.org/html/FuzzingWithConstraints.html`](http://www.fuzzingbook.org/html/FuzzingWithConstraints.html)

在前面的章节中，我们看到了基于语法的模糊测试如何使我们能够高效地生成大量的语法有效输入。然而，有一些*语义*输入特征无法在上下文无关语法中表示，例如

+   "$X$ 是 $Y$ 的长度"；

+   "$X$ 是之前声明的标识符"；或者

+   "$X$ 应该大于 4,096 字节"。

在本章中，我们展示了 [ISLa](https://rindphi.github.io/isla/) 框架如何使我们能够将此类特征作为添加到语法中的*约束*来表示。通过让 ISLa 自动解决这些约束，我们产生的输入不仅*语法*有效，而且实际上*语义*有效。此外，此类约束允许我们非常精确地*塑造*我们想要用于测试的输入。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo("dgaGuwn-1OU") 
```

**先决条件**

+   你应该已经阅读了关于语法的章节。

+   关于生成器和过滤器的章节解决了一个类似的问题，但使用程序代码而不是约束。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

## 概述

要使用本章节提供的代码，请编写

```py
>>> from fuzzingbook.FuzzingWithConstraints import <identifier> 
```

然后利用以下功能。

本章介绍了 [ISLa](https://rindphi.github.io/isla/) 框架，该框架包括

+   *ISLa 规范语言*，允许向语法添加*约束*

+   *ISLa 求解器*，解决这些约束以生成语义（和语法）上有效的输入

+   *ISLa 检查器*，检查给定输入是否满足这些约束。

ISLa 求解器的一个典型用法如下。首先，使用以下命令安装 ISLa：

```py
$  pip  install  isla-solver 
```

然后，你可以导入求解器作为

```py
>>> from [isla.solver](https://rindphi.github.io/isla/) import ISLaSolver 
```

ISLa 求解器需要两个东西。首先，一个*语法*——比如说，美国电话号码。

```py
>>> from Grammars import US_PHONE_GRAMMAR 
```

其次，你需要*约束*——一个表示一个或多个语法元素条件的字符串。常见的函数包括

+   `str.len()`，返回字符串的长度

+   `str.to.int()`，将字符串转换为整数

在这里，我们使用一个约束实例化 ISLa 求解器，该约束指出区号应大于 900：

```py
>>> solver = ISLaSolver(US_PHONE_GRAMMAR, 
>>>             """
>>>             str.to.int(<area>) > 900
>>>             """) 
```

使用它，调用 `solver.solve()` 返回约束的*解决方案*。

```py
>>> str(solver.solve())
'(904)657-8423' 
```

`solve()` 返回一个推导树，通常使用 `str()` 转换为字符串，如上所述。`print()` 函数隐式地执行此操作。

后续调用 `solve()` 返回更多解决方案：

```py
>>> for _ in range(10):
>>>     print(solver.solve())
(904)596-1030
(904)436-4047
(904)723-4207
(904)323-0627
(904)249-7959
(904)879-4209
(904)910-7560
(904)900-4775
(904)565-2710
(910)223-7794 
```

我们看到求解器产生了一系列满足约束的输入——区号总是大于 900。

`ISLaSolver()` 构造函数提供了几个额外的参数来配置求解器，如下文所述。额外的 `ISLaSolver` 方法允许检查输入是否满足约束，并提供额外的功能。

<svg width="226pt" height="94pt" viewBox="0.00 0.00 226.25 94.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 90)"><g id="node1" class="node"><title>ISLaSolver</title> <g id="a_node1"><a xlink:href="isla.solver.ipynb" xlink:title="类 ISLaSolver:

ISLa 公式的约束求解器类。其顶级方法包括

:meth:`~isla.solver.ISLaSolver.solve`

用于生成 ISLa 约束的解决方案。

:meth:`~isla.solver.ISLaSolver.check`

用于检查给定输入是否满足 ISLa 约束。

:meth:`~isla.solver.ISLaSolver.parse`

用于解析和验证输入。

:meth:`~isla.solver.ISLaSolver.repair`

用于修复输入，使其满足约束。

:meth:`~isla.solver.ISLaSolver.mutate`

用于变异输入，使得结果满足约束。"><text text-anchor="start" x="8" y="-69.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">ISLaSolver</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="ISLaSolver"><g id="a_node1_1"><a xlink:href="isla.solver.ipynb" xlink:title="__init__(self, grammar: Union[Mapping[str, Sequence[str]], str], formula: Union[isla.language.Formula, str, NoneType] = None, structural_predicates: Set[isla.language.StructuralPredicate] = frozenset({StructuralPredicate(name='inside', arity=2, eval_fun=<function in_tree>), StructuralPredicate(name='level', arity=4, eval_fun=<function level_check>), StructuralPredicate(name='consecutive', arity=2, eval_fun=<function consecutive>), StructuralPredicate(name='before', arity=2, eval_fun=<function is_before>), StructuralPredicate(name='nth', arity=3, eval_fun=<function is_nth>), StructuralPredicate(name='same_position', arity=2, eval_fun=<function is_same_position>), StructuralPredicate(name='after', arity=2, eval_fun=<function is_after>), StructuralPredicate(name='different_position', arity=2, eval_fun=<function is_different_position>), StructuralPredicate(name='direct_child', arity=2, eval_fun=<function is_direct_child>)}), semantic_predicates: Set[isla.language.SemanticPredicate] = frozenset({SemanticPredicate(count, 3)}), max_number_free_instantiations: int = 10, max_number_smt_instantiations: int = 10, max_number_tree_insertion_results: int = 5, enforce_unique_trees_in_queue: bool = False, debug: bool = False, cost_computer: Optional[ForwardRef('CostComputer')] = None, timeout_seconds: Optional[int] = None, global_fuzzer: bool = False, predicates_unique_in_int_arg: Tuple[isla.language.SemanticPredicate, ...] = (SemanticPredicate(count, 3),), fuzzer_factory: Callable[[Mapping[str, Sequence[str]]], isla.fuzzer.GrammarFuzzer] = <function SolverDefaults.<lambda>>, tree_insertion_methods: Optional[int] = None, activate_unsat_support: bool = False, grammar_unwinding_threshold: int = 4, initial_tree: returns.maybe.Maybe[isla.derivation_tree.DerivationTree] = <Nothing>, enable_optimized_z3_queries: bool = True, start_symbol: Optional[str] = None):

:class:`~isla.solver.ISLaSolver` 构造函数接受大量参数。然而，除了第一个参数 :code:`grammar` 之外，其余的都是可选的。

参数。但是，除了第一个参数 :code:`grammar` 之外，其余的都是可选的。

构建 ISLa 求解器的最简单方法就是只向它提供一个

仅语法；它就像一个语法模糊器。

>>> import random

>>> random.seed(1)

>>> import string

>>> LANG_GRAMMAR = {

... &nbsp;&nbsp;&nbsp;&nbsp;&quot;<start>&quot;:

... &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&quot;<stmt>&quot;],

... &nbsp;&nbsp;&nbsp;&nbsp;&quot;<stmt>&quot;:

... &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&quot;<assgn> ; <stmt>&quot;, &quot;<assgn>&quot;],

... &nbsp;&nbsp;&nbsp;&nbsp;&quot;<assgn>&quot;:

... &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&quot;<var> := <rhs>&quot;],

... &nbsp;&nbsp;&nbsp;&nbsp;&quot;<rhs>&quot;:

... &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&quot;<var>&quot;, &quot;<digit>&quot;],

... &nbsp;&nbsp;&nbsp;&nbsp;&quot;<var>&quot;: list(string.ascii_lowercase),

... &nbsp;&nbsp;&nbsp;&nbsp;&quot;<digit>&quot;: list(string.digits)

... }

>>>

>>> from isla.solver import ISLaSolver

>>> solver = ISLaSolver(LANG_GRAMMAR)

>>>

>>> str(solver.solve())

'd := 9'

>>> str(solver.solve())

'v := n ; s := r'

:param grammar: 基础语法；可以是“Fuzzing Book”字典

或在 BNF 语法中。

:param formula: 要解决的公式；可以是字符串或易于解析的

公式。如果没有给出公式，则假定默认的`true`约束，并且

如果解算器回退到语法 fuzzer。产生的解决方案数量

然后将绑定到`max_number_free_instantiations`。

:param structural_predicates: 解析公式时使用的结构谓词

公式。

:param semantic_predicates: 解析公式时使用的语义谓词

:param max_number_free_instantiations: 非终结符实例化的次数

都不受任何公式约束，应由基于覆盖率的 fuzzer 扩展。

:param max_number_smt_instantiations: 实例化通用整数量词的解决方案数量

应该被产生。

:param max_number_tree_insertion_results: 当

通过树插入解决存在量词。

:param enforce_unique_trees_in_queue: 如果为 true，则具有与队列中

已经存在的树在队列中被丢弃，不考虑

约束。

:param debug: 如果为 true，则关于状态演化的调试信息

收集，特别是在字段 state_tree 中。树的根在

field state_tree_root. 字段 costs 存储了计算的成本值

所有新节点。

:param cost_computer: 用于计算相关成本的`CostComputer`类

将状态放入 ISLa 的队列中。

:param timeout_seconds: 解算器将在多少秒后终止。

:param global_fuzzer: 如果设置为 true，则仅使用一个基于覆盖率的

对象用于完成无约束的开放派生树

整个生成时间。这可能对某些目标有利；例如，我们

经验表明 CSV 运行速度明显更快。然而，达到的 k 路径

覆盖率可能会降低。

:param predicates_unique_in_int_arg: 在某些情况下需要

instantiating universal integer quantifiers. 提供的谓词应该

恰好有一个整数参数，并且对于恰好一个整数值成立

一旦所有其他参数都固定。

:param fuzzer_factory: 要使用的 fuzzer 的构造函数，用于实例化

&quot;free&quot;非终结符。

:param tree_insertion_methods: 要使用的存在量词插入方法的组合

通过树插入进行量词消除。全选择：`DIRECT_EMBEDDING & CONTEXT_ADDITION`

SELF_EMBEDDING & CONTEXT_ADDITION`.

:param activate_unsat_support: 如果假设公式可能

可能不可满足。这会触发对不可满足性的额外测试

降低输入生成性能，但可能确保终止（带有

求解器结果（对于不可满足的问题，求解器可以

否则将发散。

:param grammar_unwinding_threshold: 当查询 SMT 求解器时，ISLa 传递一个

涉及的非终端的语法正则表达式。如果该

如果涉及的语法不是正则的，我们在参考语法中展开相应的部分

深度达到`grammar_unwinding_threshold`。如果这个深度太浅，它可能会

发生方程等无法解决的情况；如果太深，它可能会

会严重影响性能（并且非常严重）。

:param initial_tree: 如果求解器应该从队列开始，则对应的初始输入树

不从树`(<start>, None)`开始。

:param enable_optimized_z3_queries: 启用 Z3 查询的预处理（主要是

与长度等事物相关的数值问题）。这可以提高性能

显著；然而，可能会发生某些问题无法解决的情况

不再需要。在这种情况下，此选项可以/应该被禁用。

:param start_symbol: 这是`initial_tree`的替代方案，用于从

一个不同的起始符号`<start>`。如果提供了`start_symbol`，则树

由单个根节点组成，其值为`start_symbol`，作为

初始树。"><text text-anchor="start" x="10.62" y="-47" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__init__()</text></a></g> <g id="a_node1_2"><a xlink:href="isla.solver.ipynb" xlink:title="check(self, inp: isla.derivation_tree.DerivationTree | str) -> bool:

评估给定的推导树是否满足传递给

的求解器。如果无法评估，则引发`UnknownResultError`。

（例如，由于求解器超时或无法解决的语义谓词），则引发

评估）。

:param inp: 要评估的输入，可以是已解析的或字符串形式。

:return: 一个布尔值。"><text text-anchor="start" x="10.62" y="-34.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">check()</text></a></g> <g id="a_node1_3"><a xlink:href="isla.solver.ipynb" xlink:title="parse(self, inp: str, nonterminal: str = '<start>', skip_check: bool = False, silent: bool = False) -> isla.derivation_tree.DerivationTree:

解析给定的输入`inp`。如果输入不满足语法，则引发`SyntaxError`，

如果它满足语法，则返回`SemanticError`，如果不满足约束

（仅在`nonterminal`是`<start>`时进行检查），并返回解析

`DerivationTree`否则。

:param inp: 要解析的输入。

:param nonterminal: 如果提供了字符串，则从该非终端开始解析

对应于子语法的树。我们不检查语义

在那种情况下可能不正确。

:param skip_check: 如果为 True，则省略语义检查。

:param silent: 如果为 True，则在发生错误时不会将错误发送到日志流。

解析失败。

:return: 解析后的 `DerivationTree`。《<text text-anchor="start" x="10.62" y="-21.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">parse()</text></a></g> <g id="a_node1_4"><a xlink:href="isla.solver.ipynb" xlink:title="solve(self) -> isla.derivation_tree.DerivationTree:

尝试计算给定 ISLa 公式的解决方案。返回该解决方案，

如果有任何。此函数可以重复调用以获取更多解决方案，直到

会引发两种异常类型之一：一个 :class:`StopIteration` 表示

没有更多解决方案可找到；如果超时，将引发一个 :class:`TimeoutError`

发生。之后，每次都会引发异常。

超时可以通过 :code:`timeout_seconds`

:meth:`构造函数 <isla.solver.ISLaSolver.__init__>` 参数。

:return: 传递给 ISLa 公式的解决方案

:class:`isla.solver.ISLaSolver`。《<text text-anchor="start" x="10.62" y="-8.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">solve()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>图例</title> <text text-anchor="start" x="99" y="-59" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="99" y="-49" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="105" y="-49" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="99" y="-39" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="105" y="-39" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="99" y="-29" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="105" y="-29" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="99" y="-19.95" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

ISLa 功能性也适用于命令行：

```py
>>> !isla --help
usage: isla [-h] [-v]
            {solve,fuzz,check,find,parse,repair,mutate,create,config} ...

The ISLa command line interface.

options:
  -h, --help            show this help message and exit
  -v, --version         Print the ISLa version number

Commands:
  {solve,fuzz,check,find,parse,repair,mutate,create,config}
    solve               create solutions to ISLa constraints or check their
                        unsatisfiability
    fuzz                pass solutions to an ISLa constraint to a test subject
    check               check whether an input satisfies an ISLa constraint
    find                filter files satisfying syntactic & semantic
                        constraints
    parse               parse an input into a derivation tree if it satisfies
                        an ISLa constraint
    repair              try to repair an existing input such that it satisfies
                        an ISLa constraint
    mutate              mutate an input such that the result satisfies an ISLa
                        constraint
    create              create grammar and constraint stubs
    config              dumps the default configuration file 
```

## 语义输入属性

在这本书中，我们经常使用语法来系统地生成输入，以覆盖输入结构等。但是，虽然使用语法表达输入的*语法*相对容易，但有些输入属性是*无法*使用语法表达的。这些输入属性被称为*语义*属性。

让我们用一个简单的例子来说明语义属性。我们想要测试一个由两个设置配置的系统，一个是*页面大小*，另一个是*缓冲区大小*。这两个设置都作为整数数字作为人类可读配置文件的一部分。该文件的*语法*由以下语法给出：

```py
from Grammars import Grammar, is_valid_grammar, syntax_diagram, crange 
```

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
CONFIG_GRAMMAR: Grammar = {
    "<start>": ["<config>"],
    "<config>": [
        "pagesize=<pagesize>\n"
        "bufsize=<bufsize>"
    ],
    "<pagesize>": ["<int>"],
    "<bufsize>": ["<int>"],
    "<int>": ["<leaddigit><digits>"],
    "<digits>": ["", "<digit><digits>"],
    "<digit>": list("0123456789"),
    "<leaddigit>": list("123456789")
} 
```

```py
assert is_valid_grammar(CONFIG_GRAMMAR) 
```

这是一个将此语法作为铁路图可视化的例子，显示了其结构：

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 191.0 62" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="95.5" y="35">config</text></g></g></g></g></svg>

```py
config

```

<svg class="railroad-diagram" height="62" viewBox="0 0 540.5 62" width="540.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="108.25" y="35">pagesize=</text></g> <g class="non-terminal"><text x="220.5" y="35">pagesize</text></g> <g class="terminal"><text x="332.75" y="35">bufsize=</text></g> <g class="non-terminal"><text x="440.75" y="35">bufsize</text></g></g></g></g></svg>

```py
pagesize

```

<svg class="railroad-diagram" height="62" viewBox="0 0 165.5 62" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="82.75" y="35">int</text></g></g></g></g></svg>

```py
bufsize

```

<svg class="railroad-diagram" height="62" viewBox="0 0 165.5 62" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="82.75" y="35">int</text></g></g></g></g></svg>

```py
int

```

<svg class="railroad-diagram" height="62" viewBox="0 0 307.5 62" width="307.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="108.25" y="35">leaddigit</text></g> <g class="non-terminal"><text x="212.0" y="35">digits</text></g></g></g></g></svg>

```py
digits

```

<svg class="railroad-diagram" height="92" viewBox="0 0 273.5 92" width="273.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="65">digit</text></g> <g class="non-terminal"><text x="178.0" y="65">digits</text></g></g></g></g></svg>

```py
digit

```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

```py
leaddigit

```

<svg class="railroad-diagram" height="80" viewBox="0 0 876.5 80" width="876.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="44">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="44">2</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="44">3</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="44">4</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="44">5</text></g></g></g> <g><g><g class="terminal"><text x="526.75" y="44">6</text></g></g></g> <g><g><g class="terminal"><text x="615.25" y="44">7</text></g></g></g> <g><g><g class="terminal"><text x="703.75" y="44">8</text></g></g></g> <g><g><g class="terminal"><text x="792.25" y="44">9</text></g></g></g></g></g></svg>

使用这个语法，我们现在可以使用我们的任何基于语法的模糊测试器来生成有效输入。例如：

```py
from GrammarFuzzer import GrammarFuzzer, DerivationTree 
```

```py
fuzzer = GrammarFuzzer(CONFIG_GRAMMAR) 
```

```py
for i in range(10):
    print(i)
    print(fuzzer.fuzz()) 
```

```py
0
pagesize=4057
bufsize=817
1
pagesize=9
bufsize=8
2
pagesize=5
bufsize=25
3
pagesize=1
bufsize=2
4
pagesize=62
bufsize=57
5
pagesize=2
bufsize=893
6
pagesize=1
bufsize=33
7
pagesize=7537
bufsize=3
8
pagesize=97
bufsize=983
9
pagesize=2
bufsize=2

```

到目前为止，一切顺利——确实，这些随机值将帮助我们测试我们的（假设的）系统。但如果我们想进一步*控制*这些值，对系统进行测试呢？

语法给我们带来了一定的控制。例如，如果我们想确保页面大小至少为 100,000，那么可以设置如下规则：

```py
"<bufsize>": ["<leaddigit><digit><digit><digit><digit><digit>"] 
```

就可以完成这项工作。我们也可以通过使其以奇数位结束来表达页面大小应该是奇数。但如果我们想声明页面大小应该是 8 的倍数，或者更大或小于缓冲区大小，我们就无能为力了。

在关于使用生成器的模糊测试章节中，我们看到了如何将*程序代码*附加到单个规则上——程序代码可以立即*生成*单个元素，或者只*过滤*满足特定条件的元素。

附加代码使事情变得非常灵活，但也存在几个缺点：

+   首先，同时满足多个约束条件是非常困难的。本质上，你必须编写自己的*策略*来生成输入，这在某种程度上抵消了拥有像语法这样的抽象表示的优势。

+   第二，你的代码是不可移植的。虽然语法可以很容易地适应任何基于语法的模糊测试器，但添加，比如说，Python 代码，将你永远绑定在 Python 环境中。

+   第三，程序代码只能用于*生成*输入或*检查*输入，但不能两者兼得。这又与纯语法表示相比是一个缺点。

因此，我们正在寻找一种更*通用*的方式来表达语义属性——以及一种更*声明式*的方式来表达语义属性。

<details id="Excursion:-Unrestricted-Grammars"><summary>无限制语法</summary>

解决这个问题的一个非常通用的方法就是使用*无限制*语法，而不是我们迄今为止使用的*上下文无关*语法。在一个无限制语法中，一个可以在扩展规则的左侧有多个符号，这使得它们非常灵活。事实上，无限制语法是*图灵通用*的，这意味着它们可以表达任何在程序代码中也可以表达的特征；因此，它们可以检查和生成具有任意特征的任意字符串。（如果它们完成了，那就是说——无限制语法也受到停机问题的困扰。）缺点是实际上没有对无限制语法的编程支持——我们不得不从头开始在一个语法中实现所有算术、字符串和其他功能，这——嗯——不是很有趣。</details>

## 指定约束

在最近的研究中，*多米尼克·施泰因霍费尔*和*安德烈亚斯·策勒*（本书的作者之一）提出了一种基础设施，允许生成具有*任意属性*的输入，但无需麻烦地实现生成器或检查器。相反，他们建议一种专门用于指定输入的*语言*，称为[ISLa](https://rindphi.github.io/isla/)（输入指定语言）。*ISLa*结合了一个标准的上下文无关*语法*和表达输入及其元素*语义属性*的*约束*。ISLa 可以用作*模糊器*（生成满足约束的输入）以及*检查器*（检查输入是否满足给定的约束）。

让我们通过例子来说明 ISLa。ISLa 是一个名为`isla-solver`的 Python 包，可以使用`pip`轻松安装：

```py
$  pip  install  isla-solver 
```

这也将安装所有依赖包。

ISLa 的核心是*ISLa 求解器*——实际*解决*约束以生成满足输入的组件。

```py
import [isla](https://rindphi.github.io/isla/) 
```

```py
from [isla.solver](https://rindphi.github.io/isla/) import ISLaSolver 
```

`ISLaSolver`的构造函数接受两个必选参数。

+   *语法*是求解器应该从中生成输入的语法。

+   *约束*是指产生的输入应该满足的约束。

要表达一个约束，我们有各种*函数*和*谓词*可供选择。这些可以应用于语法的单个元素，特别是它们的非终结符。例如，函数`str.len()`返回字符串的长度。如果我们想要有页面大小至少有 6 位数的输入，我们可以这样写：

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 'str.len(<pagesize>) >= 6') 
```

`solve()`方法返回 ISLa 求解器产生的下一个字符串，作为一个*推导树*（见关于使用语法进行模糊测试的章节）。要将这些转换为字符串，我们可以使用`str()`转换器：

```py
str(solver.solve()) 
```

```py
'pagesize=111534185\nbufsize=3'

```

`print()`方法会隐式地将其参数转换为字符串。要获取，比如说，下一个 10 个解决方案，我们可以这样写：

```py
for _ in range(10):
    print(solver.solve()) 
```

```py
pagesize=111534185
bufsize=402493567181
pagesize=111534185
bufsize=1
pagesize=111534185
bufsize=96
pagesize=111534185
bufsize=81
pagesize=111534185
bufsize=514
pagesize=111534185
bufsize=2
pagesize=111534185
bufsize=635
pagesize=111534185
bufsize=7
pagesize=111534185
bufsize=3
pagesize=25395746815885
bufsize=44

```

...我们看到，确实，每个页面大小正好有六位数字。

<details id="Excursion:-Derivation-Trees"><summary>推导树</summary>

如果你直接检查`solve()`返回的推导树，你会得到一个非常复杂的结构：

```py
solution = solver.solve()
solution 
```

```py
DerivationTree('<start>', (DerivationTree('<config>', (DerivationTree('pagesize=', (), id=2), DerivationTree('<pagesize>', (DerivationTree('<int>', (DerivationTree('<leaddigit>', (DerivationTree('2', (), id=824),), id=825), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('5', (), id=834),), id=835), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('3', (), id=844),), id=845), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('9', (), id=854),), id=855), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('5', (), id=864),), id=865), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('7', (), id=874),), id=875), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('4', (), id=884),), id=885), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('6', (), id=894),), id=895), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('8', (), id=904),), id=905), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('1', (), id=914),), id=915), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('5', (), id=924),), id=925), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('8', (), id=934),), id=935), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('8', (), id=944),), id=945), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('5', (), id=954),), id=955), DerivationTree('<digits>', (), id=959)), id=948)), id=938)), id=928)), id=918)), id=908)), id=898)), id=888)), id=878)), id=868)), id=858)), id=848)), id=838)), id=828)), id=819),), id=816), DerivationTree('\nbufsize=', (), id=4), DerivationTree('<bufsize>', (DerivationTree('<int>', (DerivationTree('<leaddigit>', (DerivationTree('3', (), id=1614),), id=1621), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('0', (), id=1630),), id=1640), DerivationTree('<digits>', (DerivationTree('<digit>', (DerivationTree('1', (), id=1646),), id=1655), DerivationTree('<digits>', (DerivationTree('', (), id=1641),), id=1644)), id=1629)), id=1625)), id=1611),), id=1608)), id=1),), id=0)

```

我们可以轻松地可视化树，揭示其结构：

```py
display_tree(solution) 
```

<svg width="898pt" height="927pt" viewBox="0.00 0.00 897.62 926.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 922.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="143.38" y="-905.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="143.38" y="-855.2" font-family="Times,serif" font-size="14.00"><config></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="27.38" y="-804.95" font-family="Times,serif" font-size="14.00">pagesize=</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="104.38" y="-804.95" font-family="Times,serif" font-size="14.00"><pagesize></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node48" class="node"><title>47</title> <text text-anchor="middle" x="182.38" y="-804.95" font-family="Times,serif" font-size="14.00">\nbufsize=</text></g> <g id="edge47" class="edge"><title>1->47</title></g> <g id="node49" class="node"><title>48</title> <text text-anchor="middle" x="257.38" y="-804.95" font-family="Times,serif" font-size="14.00"><bufsize></text></g> <g id="edge48" class="edge"><title>1->48</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="104.38" y="-754.7" font-family="Times,serif" font-size="14.00"><int></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="42.38" y="-704.45" font-family="Times,serif" font-size="14.00"><leaddigit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="115.38" y="-704.45" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge7" class="edge"><title>4->7</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="42.38" y="-654.2" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="106.38" y="-654.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="168.38" y="-654.2" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge10" class="edge"><title>7->10</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="106.38" y="-603.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="165.38" y="-603.95" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="227.38" y="-603.95" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge13" class="edge"><title>10->13</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="165.38" y="-553.7" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="224.38" y="-553.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="286.38" y="-553.7" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge16" class="edge"><title>13->16</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="224.38" y="-503.45" font-family="Times,serif" font-size="14.00">9 (57)</text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="282.38" y="-503.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="344.38" y="-503.45" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge19" class="edge"><title>16->19</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="282.38" y="-453.2" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge18" class="edge"><title>17->18</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="340.38" y="-453.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="402.38" y="-453.2" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge22" class="edge"><title>19->22</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="340.38" y="-402.95" font-family="Times,serif" font-size="14.00">7 (55)</text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="398.38" y="-402.95" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="460.38" y="-402.95" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge25" class="edge"><title>22->25</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="398.38" y="-352.7" font-family="Times,serif" font-size="14.00">4 (52)</text></g> <g id="edge24" class="edge"><title>23->24</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="456.38" y="-352.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge26" class="edge"><title>25->26</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="518.38" y="-352.7" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge28" class="edge"><title>25->28</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="456.38" y="-302.45" font-family="Times,serif" font-size="14.00">6 (54)</text></g> <g id="edge27" class="edge"><title>26->27</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="514.38" y="-302.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="576.38" y="-302.45" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge31" class="edge"><title>28->31</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="514.38" y="-252.2" font-family="Times,serif" font-size="14.00">8 (56)</text></g> <g id="edge30" class="edge"><title>29->30</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="572.38" y="-252.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge32" class="edge"><title>31->32</title></g> <g id="node35" class="node"><title>34</title> <text text-anchor="middle" x="634.38" y="-252.2" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge34" class="edge"><title>31->34</title></g> <g id="node34" class="node"><title>33</title> <text text-anchor="middle" x="572.38" y="-201.95" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge33" class="edge"><title>32->33</title></g> <g id="node36" class="node"><title>35</title> <text text-anchor="middle" x="630.38" y="-201.95" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge35" class="edge"><title>34->35</title></g> <g id="node38" class="node"><title>37</title> <text text-anchor="middle" x="692.38" y="-201.95" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge37" class="edge"><title>34->37</title></g> <g id="node37" class="node"><title>36</title> <text text-anchor="middle" x="630.38" y="-151.7" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge36" class="edge"><title>35->36</title></g> <g id="node39" class="node"><title>38</title> <text text-anchor="middle" x="688.38" y="-151.7" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge38" class="edge"><title>37->38</title></g> <g id="node41" class="node"><title>40</title> <text text-anchor="middle" x="750.38" y="-151.7" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge40" class="edge"><title>37->40</title></g> <g id="node40" class="node"><title>39</title> <text text-anchor="middle" x="688.38" y="-101.45" font-family="Times,serif" font-size="14.00">8 (56)</text></g> <g id="edge39" class="edge"><title>38->39</title></g> <g id="node42" class="node"><title>41</title> <text text-anchor="middle" x="746.38" y="-101.45" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge41" class="edge"><title>40->41</title></g> <g id="node44" class="node"><title>43</title> <text text-anchor="middle" x="808.38" y="-101.45" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge43" class="edge"><title>40->43</title></g> <g id="node43" class="node"><title>42</title> <text text-anchor="middle" x="746.38" y="-51.2" font-family="Times,serif" font-size="14.00">8 (56)</text></g> <g id="edge42" class="edge"><title>41->42</title></g> <g id="node45" class="node"><title>44</title> <text text-anchor="middle" x="804.38" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge44" class="edge"><title>43->44</title></g> <g id="node47" class="node"><title>46</title> <text text-anchor="middle" x="866.38" y="-51.2" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge46" class="edge"><title>43->46</title></g> <g id="node46" class="node"><title>45</title> <text text-anchor="middle" x="804.38" y="-0.95" font-family="Times,serif" font-size="14.00">5 (53)</text></g> <g id="edge45" class="edge"><title>44->45</title></g> <g id="node50" class="node"><title>49</title> <text text-anchor="middle" x="257.38" y="-754.7" font-family="Times,serif" font-size="14.00"><int></text></g> <g id="edge49" class="edge"><title>48->49</title></g> <g id="node51" class="node"><title>50</title> <text text-anchor="middle" x="247.38" y="-704.45" font-family="Times,serif" font-size="14.00"><leaddigit></text></g> <g id="edge50" class="edge"><title>49->50</title></g> <g id="node53" class="node"><title>52</title> <text text-anchor="middle" x="320.38" y="-704.45" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge52" class="edge"><title>49->52</title></g> <g id="node52" class="node"><title>51</title> <text text-anchor="middle" x="247.38" y="-654.2" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge51" class="edge"><title>50->51</title></g> <g id="node54" class="node"><title>53</title> <text text-anchor="middle" x="311.38" y="-654.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge53" class="edge"><title>52->53</title></g> <g id="node56" class="node"><title>55</title> <text text-anchor="middle" x="373.38" y="-654.2" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge55" class="edge"><title>52->55</title></g> <g id="node55" class="node"><title>54</title> <text text-anchor="middle" x="311.38" y="-603.95" font-family="Times,serif" font-size="14.00">0 (48)</text></g> <g id="edge54" class="edge"><title>53->54</title></g> <g id="node57" class="node"><title>56</title> <text text-anchor="middle" x="369.38" y="-603.95" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge56" class="edge"><title>55->56</title></g> <g id="node59" class="node"><title>58</title> <text text-anchor="middle" x="431.38" y="-603.95" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge58" class="edge"><title>55->58</title></g> <g id="node58" class="node"><title>57</title> <text text-anchor="middle" x="369.38" y="-553.7" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge57" class="edge"><title>56->57</title></g> <g id="node60" class="node"><title>59</title></g> <g id="edge59" class="edge"><title>58->59</title></g></g></svg>

通过将推导树转换为字符串，我们得到表示的字符串：

```py
str(solution) 
```

```py
'pagesize=25395746815885\nbufsize=301'

```

`print()` 隐式地执行此操作，因此打印解决方案会给我们字符串：

```py
print(solution) 
```

```py
pagesize=25395746815885
bufsize=301

```

除非你想检查推导树或访问其元素，将其转换为字符串会使它更容易管理。</details>

要表达最小数值，我们可以使用更优雅的方式。例如，函数 `str.to.int()` 将字符串转换为整数。为了获得至少 100000 的页面大小，我们也可以这样写

```py
solver = ISLaSolver(CONFIG_GRAMMAR,
                    'str.to.int(<pagesize>) >= 100000') 
```

```py
print(solver.solve()) 
```

```py
pagesize=100005
bufsize=25

```

如果我们想使页面大小在 100 到 200 之间，我们可以将其表述为一个逻辑合取（使用 `and`）

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 str.to.int(<pagesize>) >= 100 and 
 str.to.int(<pagesize>) <= 200
 ''')
print(solver.solve()) 
```

```py
pagesize=168
bufsize=9

```

如果我们想使页面大小是七的倍数，我们可以这样写

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 str.to.int(<pagesize>) mod 7 = 0
 ''')
print(solver.solve()) 
```

```py
pagesize=7
bufsize=7

```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import quiz 
```

### 习题

以下哪个约束表示页面大小和缓冲区大小必须相等？试一试！

事实上，ISLa 约束也可以涉及多个元素。表达两个元素之间的等式很简单，使用单个等号。在 ISLa 中没有赋值操作（`=` 可能会引起混淆）。

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 <pagesize> = <bufsize>
 ''')
print(solver.solve()) 
```

```py
pagesize=8
bufsize=8

```

我们还可以使用数值约束，声明缓冲区大小应该总是比页面大小多一个：

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 str.to.int(<pagesize>) > 1024 and
 str.to.int(<bufsize>) = str.to.int(<pagesize>) + 1
 ''')
print(solver.solve()) 
```

```py
pagesize=1025
bufsize=1026

```

所有上述功能（如 `str.to.int()`）实际上都源于 *SMT-LIB* 库，用于 *可满足性模理论*（SMT），这是表达约束求解器（如 ISLa）约束的标准。[SMT-LIB 中定义的所有理论列表](https://smtlib.cs.uiowa.edu/theories.shtml) 列出了数十个可以在 ISLa 约束中使用的函数和谓词。

### 习题

以下哪些约束是确保所有数字都在 1 到 3 之间的必要条件？ <details id="Excursion:-Using-SMT-LIB-Syntax"><summary>使用 SMT-LIB 语法</summary>

除了上述对程序员熟悉的“中置”语法之外，ISLa 还支持完整的 SMT-LIB 语法。SMT-LIB 使用类似于 LISP 的“前缀”语法来表示函数和运算符，其中所有函数和运算符都写作 `(f x_1 x_2 x_3 ...)`。因此，上述谓词将被写成

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 (and
 (> (str.to.int <pagesize>) 1024)
 (= (str.to.int <bufsize>)
 (+ (str.to.int <pagesize>) 1)))
 ''')
print(solver.solve()) 
```

```py
pagesize=1025
bufsize=1026

```

注意，对于布尔运算符，如 `and`，我们仍然使用 ISLa 中置语法；让 ISLa 处理这些运算符比将它们传递给约束求解器更有效率。</details>

## ISLa 命令行

当你安装 `isla-solver` 时，你也会得到一个 `isla` 命令行工具。这允许你从命令行或 shell 脚本创建输入。

让我们首先创建一个适合 `isla` 的 *语法文件*。`isla` 接受 Fuzzingbook 格式的语法；它们需要定义一个名为 `grammar` 的变量。

```py
with open('grammar.py', 'w') as f:
    f.write('grammar = ')
    f.write(str(CONFIG_GRAMMAR)) 
```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import print_file 
```

```py
print_file('grammar.py') 
```

```py
grammar = {'<start>': ['<config>'], '<config>': ['pagesize=<pagesize>\nbufsize=<bufsize>'], '<pagesize>': ['<int>'], '<bufsize>': ['<int>'], '<int>': ['<leaddigit><digits>'], '<digits>': ['', '<digit><digits>'], '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'], '<leaddigit>': ['1', '2', '3', '4', '5', '6', '7', '8', '9']}

```

使用这种方式，我们可以简单地将 `isla` 作为语法模糊器使用。默认情况下，`isla solve` 产生单个匹配输出：

```py
!isla  solve  grammar.py 
```

```py
pagesize=5
bufsize=81452639708

```

然而，`isla` 的真正威力在于我们（再次）添加要解决的 *约束* - 要么在单独的 *约束文件* 中，要么（更简单）直接在命令行上：

```py
!isla  solve  grammar.py  --constraint  '<pagesize> = <bufsize>' 
```

```py
pagesize=8
bufsize=8

```

```py
!isla  solve  grammar.py  \
    --constraint '<pagesize> = <bufsize> and str.to.int(<pagesize>) > 10000' 
```

```py
pagesize=99290248
bufsize=99290248

```

`isla` 可执行文件提供了几个选项和命令，并且在命令行上是一个很好的替代品。

```py
!isla  --help 
```

```py
usage: isla [-h] [-v]
            {solve,fuzz,check,find,parse,repair,mutate,create,config} ...

The ISLa command line interface.

options:
  -h, --help            show this help message and exit
  -v, --version         Print the ISLa version number

Commands:
  {solve,fuzz,check,find,parse,repair,mutate,create,config}
    solve               create solutions to ISLa constraints or check their
                        unsatisfiability
    fuzz                pass solutions to an ISLa constraint to a test subject
    check               check whether an input satisfies an ISLa constraint
    find                filter files satisfying syntactic & semantic
                        constraints
    parse               parse an input into a derivation tree if it satisfies
                        an ISLa constraint
    repair              try to repair an existing input such that it satisfies
                        an ISLa constraint
    mutate              mutate an input such that the result satisfies an ISLa
                        constraint
    create              create grammar and constraint stubs
    config              dumps the default configuration file

```

## 访问元素

到目前为止，我们只是通过引用它们的名称来访问非终结符，例如 `<bufsize>` 或 `<pagesize>`。然而，在某些情况下，这种方法并不足够。例如，在我们的配置语法中，我们可能想要访问（或约束）`<int>` 元素。但是，我们不想一次约束所有的整数，而只想约束特定**上下文**中的那些——比如说，那些作为 `<pagesize>` 元素一部分出现的，或者只那些作为 `<config>` 元素一部分出现的。

为了达到这个目的，ISLa 允许使用两个特殊操作符来引用给定元素的部分。

表达式 `<a>.<b>` 指的是某个元素 `<a>` 的**直接**子部分 `<b>`。也就是说，`<b>` 必须出现在 `<a>` 的某个展开规则中。例如，`<pagesize>.<int>` 指的是页面大小的 `<int>` 元素。以下是一个使用点操作符的示例：

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 <pagesize>.<int> = <bufsize>.<int>
 ''')
print(solver.solve()) 
```

```py
pagesize=8
bufsize=8

```

然而，表达式 `<a>..<b>` 指的是某个元素 `<a>` 的**任何**子部分 `<b>`。也就是说，`<b>` 可以出现在 `<a>` 的展开中，也可以出现在任何子元素（及其子元素）的展开中。以下是一个使用双点操作符的示例，强制 `<config>` 元素中的每个数字都是 `7`：

```py
from ExpectError import ExpectError 
```

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 <config>..<digit> = "7" and <config>..<leaddigit> = "7"
 ''')
print(solver.solve()) 
```

```py
pagesize=77
bufsize=7

```

要理解点和双点，将问题字符串可视化为**推导树**有助于理解，该推导树在基于语法的模糊测试章节中讨论过。输入的推导树如下所示：

```py
pagesize=12
bufsize=34
```

例如，看起来是这样的：

<svg width="405pt" height="324pt" viewBox="0.00 0.00 404.62 323.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 319.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="143.38" y="-302.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="143.38" y="-252.2" font-family="Times,serif" font-size="14.00"><config></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="27.38" y="-201.95" font-family="Times,serif" font-size="14.00">pagesize=</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="104.38" y="-201.95" font-family="Times,serif" font-size="14.00"><pagesize></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="182.38" y="-201.95" font-family="Times,serif" font-size="14.00">\nbufsize=</text></g> <g id="edge11" class="edge"><title>1->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="257.38" y="-201.95" font-family="Times,serif" font-size="14.00"><bufsize></text></g> <g id="edge12" class="edge"><title>1->12</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="104.38" y="-151.7" font-family="Times,serif" font-size="14.00"><int></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="42.38" y="-101.45" font-family="Times,serif" font-size="14.00"><leaddigit></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="115.38" y="-101.45" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge7" class="edge"><title>4->7</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="42.38" y="-51.2" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="106.38" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="168.38" y="-51.2" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge10" class="edge"><title>7->10</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="106.38" y="-0.95" font-family="Times,serif" font-size="14.00">2 (50)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="257.38" y="-151.7" font-family="Times,serif" font-size="14.00"><int></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="247.38" y="-101.45" font-family="Times,serif" font-size="14.00"><leaddigit></text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="320.38" y="-101.45" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge16" class="edge"><title>13->16</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="247.38" y="-51.2" font-family="Times,serif" font-size="14.00">3 (51)</text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="311.38" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="373.38" y="-51.2" font-family="Times,serif" font-size="14.00"><digits></text></g> <g id="edge19" class="edge"><title>16->19</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="311.38" y="-0.95" font-family="Times,serif" font-size="14.00">4 (52)</text></g> <g id="edge18" class="edge"><title>17->18</title></g></g></svg>

在这个树中，`.` 语法指的是**直接**子元素。`<bufsize>.<int>` 是一个 `<int>` 节点，它是 `<bufsize>` 的直接后裔（但不是任何其他 `<int>` 节点）。相比之下，`<config>..<digit>` 指的是 `<config>` 节点的所有 `<digit>` 后裔。

如果一个元素有多个相同类型的**直接**子元素，可以使用特殊 `<a>[$n$]` 语法来访问类型为 `<a>` 的第 $n$ 个子元素。要访问第一个子元素，$n$ 等于一，而不是零，就像在 [XPath 简写语法](https://www.w3.org/TR/1999/REC-xpath-19991116/#path-abbrev) 中一样。在我们的配置语法中，没有包含相同非终结符多次展开的情况，因此我们不需要这个功能。

为了演示索引点，考虑以下语法，它生成由三个 "a" 或 "b" 字符组成的行：

```py
LINES_OF_THREE_AS_OR_BS_GRAMMAR: Grammar = {
    '<start>': ['<A>'],
    '<A>': ['<B><B><B>', '<B><B><B>\n<A>'],
    '<B>': ['a', 'b']
} 
```

```py
fuzzer = GrammarFuzzer(LINES_OF_THREE_AS_OR_BS_GRAMMAR)
for i in range(5):
    print(i)
    print(fuzzer.fuzz()) 
```

```py
0
aab
bab
1
aaa
aba
2
aba
baa
bba
3
bbb
abb
4
baa

```

我们可以强制，比如说，一行中的第二个字符始终是 "b:"

```py
solver = ISLaSolver(LINES_OF_THREE_AS_OR_BS_GRAMMAR, 
  '''
 <A>.<B>[2] = "b"
 ''')

for i in range(5):
    print(i)
    print(solver.solve()) 
```

```py
0
abb
1
bbb
2
abb
3
bbb
4
abb

```

## 量词

默认情况下，ISLa 约束中的所有非终结符都是**普遍**量化的——也就是说，任何应用于，比如说，某些 `<int>` 元素的约束都应用于结果字符串中的所有 `<int>` 元素。但是，如果你只想约束**一个**元素，你必须在 ISLa 中指定这一点（并且可以这样做），使用**存在量词**。

要在 ISLa 中使用存在量词，使用以下构造

```py
exists TYPE VARIABLE in CONTEXT:
    (CONSTRAINT)
```

其中`VARIABLE`是某个标识符，`TYPE`是其类型（作为一个非终端），`CONTEXT`是约束应该成立的上下文（再次是一个非终端）。`CONSTRAINT`再次是一个约束表达式，你现在可以使用`VARIABLE`作为你假设存在的元素。

让我们再次用一个简单的例子来说明存在量词。我们想要确保在我们生成的字符串中至少有一个整数的值大于 1000。因此我们写

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 exists <int> i in <start>:
 str.to.int(i) > 1000
 ''')

for _ in range(10):
    print(solver.solve()) 
```

```py
pagesize=1007
bufsize=5
pagesize=1007
bufsize=2280617349521
pagesize=1007
bufsize=8
pagesize=1007
bufsize=7
pagesize=1007
bufsize=3
pagesize=1007
bufsize=93
pagesize=1007
bufsize=630
pagesize=1007
bufsize=4
pagesize=1007
bufsize=14
pagesize=1007
bufsize=5

```

我们注意到所有生成的输入都满足至少有一个整数满足约束的要求。

指定变量名是可选的；如果你省略它，你可以使用量词非终端。因此，上述约束也可以用更紧凑的方式表达：

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 exists <int> in <start>:
 str.to.int(<int>) > 1000
 ''')

print(solver.solve()) 
```

```py
pagesize=1003
bufsize=64

```

除了存在量词之外，还有全称量词，使用`forall`关键字代替`exists`。如果我们想要某个上下文中的所有元素都满足一个约束，这将很有用。

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 forall <int> in <start>:
 str.to.int(<int>) > 1000
 ''')

for _ in range(10):
    print(solver.solve()) 
```

```py
pagesize=1001
bufsize=1001
pagesize=1001
bufsize=1002
pagesize=1001
bufsize=1003
pagesize=1001
bufsize=1004
pagesize=1002
bufsize=1001
pagesize=1002
bufsize=1002
pagesize=1002
bufsize=1003
pagesize=1002
bufsize=1004
pagesize=1003
bufsize=1001
pagesize=1003
bufsize=1002

```

我们可以看到所有`<int>`元素都满足约束。

默认情况下，所有直接在约束中重用的非终端在`<start>`符号内都是全称量词，所以上述实际上可以简化为

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 str.to.int(<int>) > 1000
 ''')

for _ in range(10):
    print(solver.solve()) 
```

```py
pagesize=1001
bufsize=1001
pagesize=1001
bufsize=1002
pagesize=1001
bufsize=1003
pagesize=1001
bufsize=1004
pagesize=1002
bufsize=1001
pagesize=1002
bufsize=1002
pagesize=1002
bufsize=1003
pagesize=1002
bufsize=1004
pagesize=1003
bufsize=1001
pagesize=1003
bufsize=1002

```

...然后你意识到在我们所有的初始约束中，我们总是有一个隐含的全称量词。

## 选择扩展

有时，我们希望量词只适用于非终端的一个特定扩展。形式

```py
forall TYPE VARIABLE=PATTERN in CONTEXT:
    (CONSTRAINT)
```

表示约束只适用于与模式中给出的扩展匹配的实际变量。 （再次，我们可以用`exists`替换`forall`，使其成为一个存在量词而不是全称量词。）

这里是使用`forall`的一个例子：

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 forall <int>="<leaddigit><digits>" in <start>:
 (<leaddigit> = "7")
 ''') 
```

这确保了当`<int>`扩展为一个前导数字后跟更多数字时，前导数字变为`7`。结果是所有`<int>`值现在都以一个`7`位数字开头：

```py
str(solver.solve()) 
```

```py
'pagesize=71\nbufsize=770835426929713'

```

同样，我们可以约束整个`<int>`，从而确保所有数字都大于 100：

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 forall <int> in <start>:
 (str.to.int(<int>) > 100)
 ''')

str(solver.solve()) 
```

```py
'pagesize=101\nbufsize=101'

```

默认情况下，所有变量在`<start>`中都是全称量词，所以上述也可以表示为

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 str.to.int(<int>) > 100
 ''')

str(solver.solve()) 
```

```py
'pagesize=101\nbufsize=101'

```

## 匹配扩展元素

在一个量词模式中，我们也可以为单个非终端元素命名并在约束中使用它们。这是通过将非终端`<ID>`替换为特殊形式`{<ID> VARIABLE}`（用大括号括起来）来完成的，这使得变量`VARIABLE`成为由`ID`匹配的值的占位符；`VARIABLE`然后可以在约束中使用。

这里有一个例子。在扩展`<leaddigit><int>`中，我们想要确保`<leaddigit>`始终是`9`。使用特殊的括号形式，我们将`lead`设为一个变量，其值为`<leaddigit>`，然后可以在约束中使用它：

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 forall <int> i="{<leaddigit> lead}<digits>" in <start>:
 (lead = "9")
 ''') 
```

这（再次）确保了所有前导数字应该是`9`：

```py
for _ in range(10):
    print(solver.solve()) 
```

```py
pagesize=92
bufsize=9168435097796
pagesize=9
bufsize=9188
pagesize=981
bufsize=999
pagesize=9
bufsize=9
pagesize=9
bufsize=9
pagesize=9
bufsize=9
pagesize=91
bufsize=9
pagesize=9
bufsize=9
pagesize=90
bufsize=9
pagesize=9
bufsize=9242

```

我们能否用更简单的方式表达上述内容？是的！首先，我们可以直接引用`<leaddigit>`而不是引入像`i`和`lead`这样的变量：

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 forall <int>="<leaddigit><digits>" in <start>:
 (<leaddigit> = "9")
 ''')
print(solver.solve()) 
```

```py
pagesize=9
bufsize=941890257631

```

此外，使用隐含的全称量词和之前引入的点符号，我们可以写出，例如

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 <int>.<leaddigit> = "9"
 ''')
print(solver.solve()) 
```

```py
pagesize=99
bufsize=9501387624

```

或者，也可以

```py
solver = ISLaSolver(CONFIG_GRAMMAR, 
  '''
 <leaddigit> = "9"
 ''')
print(solver.solve()) 
```

```py
pagesize=98
bufsize=93762401955

```

并获得相同的结果（尽管由于随机性，结果可能不是完全相同的值）：

但是，尽管全称量词和点符号对于许多情况来说是足够的，模式匹配符号更加通用和灵活——即使它可能更难阅读。

## 检查字符串

使用`ISLaSolver`，我们还可以检查一个字符串是否满足约束。这可以应用于输入，也可以应用于*输出*；因此，ISLa 约束可以作为*预言机*——即检查测试结果的*谓词*。

让我们检查在给定的字符串中，`<pagesize>`和`<bufsize>`是否相同。

```py
constraint = '<pagesize> = <bufsize>'
solver = ISLaSolver(CONFIG_GRAMMAR, constraint) 
```

要检查树，我们可以将其传递给`solver`对象的`evaluate()`方法——并发现给定的输入*不*满足我们的约束。

```py
solver.check('pagesize=12\nbufsize=34') 
```

```py
False

```

然而，如果我们用满足约束的输入重复上述操作，我们将获得一个`True`结果。

```py
solver.check('pagesize=27\nbufsize=27') 
```

```py
True

```

检查约束比解决约束更有效，因为 ISLa 不需要搜索可能的解决方案。

## 案例研究

让我们进一步通过几个案例研究来阐述 ISLa。

### 在 XML 中匹配标识符

可扩展标记语言（XML）是输入语言无法完全用上下文无关文法表达的典型例子。问题并不在于表达 XML 的*语法*——基础相当简单：

```py
XML_GRAMMAR: Grammar = {
    "<start>": ["<xml-tree>"],
    "<xml-tree>": ["<open-tag><xml-content><close-tag>"],
    "<open-tag>": ["<<id>>"],
    "<close-tag>": ["</<id>>"],
    "<xml-content>": ["Text", "<xml-tree>"],
    "<id>": ["<letter>", "<id><letter>"],
    "<letter>": crange('a', 'z')
} 
```

```py
assert is_valid_grammar(XML_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 208.0 62" width="208.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="104.0" y="35">xml-tree</text></g></g></g></g></svg>

```py
xml-tree

```

<svg class="railroad-diagram" height="62" viewBox="0 0 458.0 62" width="458.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="104.0" y="35">open-tag</text></g> <g class="non-terminal"><text x="224.75" y="35">xml-content</text></g> <g class="non-terminal"><text x="349.75" y="35">close-tag</text></g></g></g></g></svg>

```py
open-tag

```

<svg class="railroad-diagram" height="62" viewBox="0 0 254.0 62" width="254.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35"><</text></g> <g class="non-terminal"><text x="127.0" y="35">id</text></g> <g class="terminal"><text x="179.75" y="35">></text></g></g></g></g></svg>

```py
close-tag

```

<svg class="railroad-diagram" height="62" viewBox="0 0 262.5 62" width="262.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="78.5" y="35"></</text></g> <g class="non-terminal"><text x="135.5" y="35">id</text></g> <g class="terminal"><text x="188.25" y="35">></text></g></g></g></g></svg>

```py
xml-content

```

<svg class="railroad-diagram" height="92" viewBox="0 0 208.0 92" width="208.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="104.0" y="35">Text</text></g></g> <g><g class="non-terminal"><text x="104.0" y="65">xml-tree</text></g></g></g></g></svg>

```py
id

```

<svg class="railroad-diagram" height="92" viewBox="0 0 248.0 92" width="248.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="124.0" y="35">letter</text></g></g> <g><g class="non-terminal"><text x="78.5" y="65">id</text></g> <g class="non-terminal"><text x="152.5" y="65">letter</text></g></g></g></g></svg>

```py
letter

```

<svg class="railroad-diagram" height="198" viewBox="0 0 611.0 198" width="611.0"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="73">b</text></g></g> <g><g class="terminal"><text x="84.25" y="43">a</text></g></g> <g><g class="terminal"><text x="84.25" y="103">c</text></g></g> <g><g class="terminal"><text x="84.25" y="133">d</text></g></g> <g><g class="terminal"><text x="84.25" y="163">e</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="73">g</text></g></g> <g><g class="terminal"><text x="172.75" y="43">f</text></g></g> <g><g class="terminal"><text x="172.75" y="103">h</text></g></g> <g><g class="terminal"><text x="172.75" y="133">i</text></g></g> <g><g class="terminal"><text x="172.75" y="163">j</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="73">l</text></g></g> <g><g class="terminal"><text x="261.25" y="43">k</text></g></g> <g><g class="terminal"><text x="261.25" y="103">m</text></g></g> <g><g class="terminal"><text x="261.25" y="133">n</text></g></g> <g><g class="terminal"><text x="261.25" y="163">o</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="73">q</text></g></g> <g><g class="terminal"><text x="349.75" y="43">p</text></g></g> <g><g class="terminal"><text x="349.75" y="103">r</text></g></g> <g><g class="terminal"><text x="349.75" y="133">s</text></g></g> <g><g class="terminal"><text x="349.75" y="163">t</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="73">v</text></g></g> <g><g class="terminal"><text x="438.25" y="43">u</text></g></g> <g><g class="terminal"><text x="438.25" y="103">w</text></g></g> <g><g class="terminal"><text x="438.25" y="133">x</text></g></g> <g><g class="terminal"><text x="438.25" y="163">y</text></g></g></g> <g><g><g class="terminal"><text x="526.75" y="103">z</text></g></g></g></g></g></svg>

当我们从文法生成输入时，问题变得明显：`<open-tag>`和`<close-tag>`中的`<id>`元素不匹配。

```py
fuzzer = GrammarFuzzer(XML_GRAMMAR)
fuzzer.fuzz() 
```

```py
'<xdps><s><x><f>Text</g></ka></k></hk>'

```

如果我们想要标签 ID 匹配，我们需要提出一个*有限*的标签集（比如，HTML 中的标签）；然后我们可以为每个标签扩展一个规则——`<body>...</body>`，`<p>...</p>`，`<strong>...</strong>`等等。但对于一个*无限*的标签集，就像我们的语法一样，在上下文无关文法中表达两个标签 ID 必须匹配是不可能的。

然而，使用 ISLa，约束文法是很容易的。我们需要的只是约束`<xml-tree>`的规则：

```py
solver = ISLaSolver(XML_GRAMMAR, 
  '''
 <xml-tree>.<open-tag>.<id> = <xml-tree>.<close-tag>.<id>
 ''', max_number_smt_instantiations=1)

for _ in range(3):
    print(solver.solve()) 
```

```py
<p>Text</p>
<p><p>Text</p></p>
<p><p><p>Text</p></p></p>

```

我们看到，`<id>`标签现在确实相互匹配。

`<details id="Excursion:-Solver-Configuration-Parameters"><summary>Solver Configuration Parameters</summary>`

我们传递给`ISLaSolver`对象的配置参数`max_number_smt_instantiations`限制了 ISLa 底层 SMT 求解器的调用次数。一般来说，更高的数字会导致每次生成更多的输入。尽管许多输入看起来结构上相似。如果我们旨在生成结构多样化的输入，并且不关心，例如，标签的名称，那么为这个参数选择一个较低的值是有意义的。这就是`max_number_smt_instantiations=10`发生的情况，这是当前的默认值：

```py
solver = ISLaSolver(XML_GRAMMAR, 
  '''
 <xml-tree>.<open-tag>.<id> = <xml-tree>.<close-tag>.<id>
 ''', max_number_smt_instantiations=10)

for _ in range(3):
    print(solver.solve()) 
```

```py
<p>Text</p>
<h>Text</h>
<k>Text</k>

```

参数 `max_number_free_instantiations` 具有类似的作用：ISLa 随机实例化非终结符号，其值不受约束的限制。它选择——惊喜！——最多 `max_number_free_instantiations` 这样随机的实例化。

其他有趣的配置参数是 `structural_predicates` 和 `semantic_predicates`，它们允许你通过向求解器传递自定义的结构和语义谓词来扩展 ISLa 语言。你可以在 ISLa 约束中使用这些集合中的所有谓词来解决。默认情况下，语义谓词 `count(in_tree, NEEDLE, NUM)` 和以下结构谓词是可用的：

+   `after(node_1, node_2)`

+   `before(node_1, node_2)`

+   `consecutive(node_1, node_2)`

+   `count(in_tree, NEEDLE, NUM)`

+   `different_position(node_1, node_2)`

+   `direct_child(node_1, node_2)`

+   `inside(node_1, node_2)`

+   `level(PRED, NONTERMINAL, node_1, node_2)`

+   `nth(N, node_1, node_2)`

+   `same_position(node_1, node_2)`

与 生成器章节 中的“输入生成器”解决方案相比，我们的基于约束的解决方案是纯声明性的——也可以用于解析和检查输入。当然，我们还可以轻松地添加更多约束：

```py
solver = ISLaSolver(XML_GRAMMAR, 
  '''
 <xml-tree>.<open-tag>.<id> = <xml-tree>.<close-tag>.<id>
 and
 str.len(<id>) > 10
 ''', max_number_smt_instantiations=1)

for _ in range(3):
    print(solver.solve()) 
```

```py
<ppkphklftkp>Text</ppkphklftkp>
<ppkphklftkp><ppnpxcqktdh>Text</ppnpxcqktdh></ppkphklftkp>
<ppkphklftkp><ppnpxcqktdh><ppkxsixmahk>Text</ppkxsixmahk></ppnpxcqktdh></ppkphklftkp>

```

### 编程语言中的定义和用法

在使用生成的程序代码测试编译器时，人们经常遇到一个问题，即在 *使用* 一个标识符之前，必须先 *声明* 它——指定其类型、一些初始值等等。

在以下语法中，这个问题很容易说明，该语法产生 *赋值序列*。变量名由单个小写字母组成；值只能是数字；赋值由分号分隔。

```py
LANG_GRAMMAR: Grammar = {
    "<start>":
        ["<stmt>"],
    "<stmt>":
        ["<assgn>", "<assgn>; <stmt>"],
    "<assgn>":
        ["<lhs> := <rhs>"],
    "<lhs>":
        ["<var>"],
    "<rhs>":
        ["<var>", "<digit>"],
    "<var>": list(string.ascii_lowercase),
    "<digit>": list(string.digits)
} 
```

```py
assert is_valid_grammar(LANG_GRAMMAR) 
```

```py
syntax_diagram(LANG_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">stmt</text></g></g></g></g></svg>

```py
stmt

```

<svg class="railroad-diagram" height="92" viewBox="0 0 313.5 92" width="313.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="156.75" y="35">assgn</text></g></g> <g><g class="non-terminal"><text x="91.25" y="65">assgn</text></g> <g class="terminal"><text x="161.0" y="65">;</text></g> <g class="non-terminal"><text x="226.5" y="65">stmt</text></g></g></g></g></svg>

```py
assgn

```

<svg class="railroad-diagram" height="62" viewBox="0 0 305.0 62" width="305.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="82.75" y="35">lhs</text></g> <g class="terminal"><text x="152.5" y="35">:=</text></g> <g class="non-terminal"><text x="222.25" y="35">rhs</text></g></g></g></g></svg>

```py
lhs

```

<svg class="railroad-diagram" height="62" viewBox="0 0 165.5 62" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="82.75" y="35">var</text></g></g></g></g></svg>

```py
rhs

```

<svg class="railroad-diagram" height="92" viewBox="0 0 182.5 92" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">var</text></g></g> <g><g class="non-terminal"><text x="91.25" y="65">digit</text></g></g></g></g></svg>

```py
var

```

<svg class="railroad-diagram" height="198" viewBox="0 0 611.0 198" width="611.0"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="73">b</text></g></g> <g><g class="terminal"><text x="84.25" y="43">a</text></g></g> <g><g class="terminal"><text x="84.25" y="103">c</text></g></g> <g><g class="terminal"><text x="84.25" y="133">d</text></g></g> <g><g class="terminal"><text x="84.25" y="163">e</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="73">g</text></g></g> <g><g class="terminal"><text x="172.75" y="43">f</text></g></g> <g><g class="terminal"><text x="172.75" y="103">h</text></g></g> <g><g class="terminal"><text x="172.75" y="133">i</text></g></g> <g><g class="terminal"><text x="172.75" y="163">j</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="73">l</text></g></g> <g><g class="terminal"><text x="261.25" y="43">k</text></g></g> <g><g class="terminal"><text x="261.25" y="103">m</text></g></g> <g><g class="terminal"><text x="261.25" y="133">n</text></g></g> <g><g class="terminal"><text x="261.25" y="163">o</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="73">q</text></g></g> <g><g class="terminal"><text x="349.75" y="43">p</text></g></g> <g><g class="terminal"><text x="349.75" y="103">r</text></g></g> <g><g class="terminal"><text x="349.75" y="133">s</text></g></g> <g><g class="terminal"><text x="349.75" y="163">t</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="73">v</text></g></g> <g><g class="terminal"><text x="438.25" y="43">u</text></g></g> <g><g class="terminal"><text x="438.25" y="103">w</text></g></g> <g><g class="terminal"><text x="438.25" y="133">x</text></g></g> <g><g class="terminal"><text x="438.25" y="163">y</text></g></g></g> <g><g><g class="terminal"><text x="526.75" y="103">z</text></g></g></g></g></g></svg>

```py
digit

```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

这里是语法产生的某些赋值序列：

```py
fuzzer = GrammarFuzzer(LANG_GRAMMAR) 
```

```py
for _ in range(10):
    print(fuzzer.fuzz()) 
```

```py
w := 7; g := 2
f := m; x := 3
p := 3
h := f; h := u
n := k; k := z; z := 6
m := x; g := h
d := 3
h := 6
s := m
k := q

```

我们可以看到，赋值 *语法* 与我们在常见编程语言中看到的是相似的。然而，*语义* 呢，嗯，值得怀疑，因为我们通常访问尚未定义的值的变量。再次强调，这是一个 *语义* 属性，仅凭上下文无关文法是无法表达的。

我们需要在这里指定一个约束，即赋值的右侧只能有出现在左侧的变量名。在 ISLa 中，我们通过以下约束来实现这一点：

```py
solver = ISLaSolver(LANG_GRAMMAR, 
  '''
 forall <rhs> in <assgn>:
 exists <assgn> declaration:
 <rhs>.<var> = declaration.<lhs>.<var>
 ''',
            max_number_smt_instantiations=1,
            max_number_free_instantiations=1) 
```

```py
for _ in range(10):
    print(solver.solve()) 
```

```py
y := 1
a := 0; t := 5
h := h
u := 2; l := 4; p := 7
p := 3; r := 8; v := p
x := 9; b := 6; a := a
s := 4; h := 4; f := h
c := 5; p := p; k := 5
d := 5; q := d; n := 2
e := 6; m := p; p := 9

```

这已经好多了，但还不是完美的——我们可能仍然有像 `a := a` 或 `a := b; b := 5` 这样的赋值。这是因为我们的约束还没有考虑到 *顺序* —— 在 `<rhs>` 元素中，我们只能使用之前定义的变量。

为了这个目的，ISLa 提供了一个 `before()` 谓词：`before(A, B)` 表示元素 `A` 必须出现在元素 `B` 之前。使用 `before()`，我们可以将我们的约束重写为

```py
solver = ISLaSolver(LANG_GRAMMAR, 
  '''
 forall <rhs> in <assgn>:
 exists <assgn> declaration:
 (before(declaration, <assgn>) and
 <rhs>.<var> = declaration.<lhs>.<var>)
 ''',
            max_number_free_instantiations=1,
            max_number_smt_instantiations=1) 
```

... 因此确保在赋值的右侧，我们只使用之前定义的标识符。

```py
for _ in range(10):
    print(solver.solve()) 
```

```py
p := 1
w := 4; o := 6
b := 3; p := b; b := p; f := b
p := 0; p := p; z := p; g := 2
a := 9; a := a; r := 8; i := a
a := 5; a := a; a := 7; t := a
h := 0; j := h; h := 0; s := h
h := 2; h := h; h := 0; u := h
b := 9; b := 5; p := b; b := p; e := b
b := 9; b := b; p := b; b := p; c := b

```

如果您发现分配序列太短，可以使用 ISLa 的 `count()` 谓词。`count(VARIABLE, NONTERMINAL, N)` 确保 VARIABLE 中的 NONTERMINAL 数量正好是 N。要编写具有恰好 5 个赋值的语句，请写

```py
solver = ISLaSolver(LANG_GRAMMAR, 
  '''
 forall <rhs> in <assgn>:
 exists <assgn> declaration:
 (before(declaration, <assgn>) and
 <rhs>.<var> = declaration.<lhs>.<var>)
 and
 count(start, "<assgn>", "5")
 ''', 
            max_number_smt_instantiations=1,
            max_number_free_instantiations=1) 
```

```py
for _ in range(10):
    print(solver.solve()) 
```

```py
h := 8; y := h; c := 3; f := 9; a := 0
p := 2; k := p; p := 4; e := 7; o := p
h := 1; r := h; m := 6; g := 5; x := h
p := 5; h := p; w := 0; d := 2; s := h
p := 4; q := p; n := 0; p := 4; u := p
a := 5; i := a; b := a; z := 9; t := 6
p := 3; b := p; l := b; j := 0; v := 0
b := 3; d := b; q := d; p := 1; b := p
d := 4; l := d; l := d; p := 5; i := p
h := 5; d := h; o := d; b := 5; n := h

```

## 经验教训

+   使用 ISLa，我们可以向语法添加并解决 *约束*，从而表达我们的测试输入的 *语义属性*

+   声明约束（并让求解器解决它们）比添加生成器代码更灵活，而且也是语言无关的

+   使用 ISLa 很有趣 :-)

## 下一步

在接下来的章节中，我们将继续关注语义。其中之一，我们将学习如何

+   从现有输入中提取语法

+   使用 symbolic fuzzing - 即，使用约束求解器达到特定位置

+   使用 concolic fuzzing - 即，将符号模糊测试与具体运行相结合以提高效率

## 背景

+   ISLa 在 ESEC/FSE 2022 的论文 ["输入不变量"](https://publications.cispa.saarland/3596/) 中被介绍。

+   [ISLa 项目](https://github.com/rindPHI/isla) 包含完整的源代码和完整的参考。

## 练习

### 练习 1：字符串编码

表示字符串的一种常见方式是 *长度前缀字符串*，这种表示方法由 *PASCAL* 编程语言普及。长度前缀字符串以几个字节开始，这些字节编码了字符串的长度 $L$，然后是 $L$ 个实际字符。例如，假设使用两个字节来编码长度，字符串 `"Hello"` 可以表示为以下序列

```py
0x00 0x05 'H' 'e' 'l' 'l' 'o'
```

#### 第一部分：语法

编写一个定义长度前缀字符串语法的语法。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/FuzzingWithConstraints.ipynb#Exercises) 来完成练习并查看解决方案。

#### 第二部分：语义

使用 ISLa 生成有效的长度前缀字符串。利用 [SMT-LIB 字符串库](https://smtlib.cs.uiowa.edu/theories-UnicodeStrings.shtml) 找到适当的转换函数。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/FuzzingWithConstraints.ipynb#Exercises) 来完成练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。内容的一部分源代码，以及用于格式化和显示该内容的源代码，受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后更改：2024-11-09 17:07:29+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/FuzzingWithConstraints.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒："[带有约束的模糊测试](https://www.fuzzingbook.org/html/FuzzingWithConstraints.html)"。收录于安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒所著的"[模糊测试书籍](https://www.fuzzingbook.org/)"中。[`www.fuzzingbook.org/html/FuzzingWithConstraints.html`](https://www.fuzzingbook.org/html/FuzzingWithConstraints.html)。检索日期：2024-11-09 17:07:29+01:00.

```py
@incollection{fuzzingbook2024:FuzzingWithConstraints,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Fuzzing with Constraints},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/FuzzingWithConstraints.html}},
    note = {Retrieved 2024-11-09 17:07:29+01:00},
    url = {https://www.fuzzingbook.org/html/FuzzingWithConstraints.html},
    urldate = {2024-11-09 17:07:29+01:00}
}

```
