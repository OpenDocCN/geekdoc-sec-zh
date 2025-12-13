# 测试编译器

> 原文：[`www.fuzzingbook.org/html/PythonFuzzer.html`](http://www.fuzzingbook.org/html/PythonFuzzer.html)

在本章中，我们将利用 grammars and grammar-based testing 系统地生成 *程序代码* - 例如，测试编译器或解释器。不出所料，我们使用 *Python* 和 *Python 解释器* 作为我们的领域。

我们选择 Python 不仅是因为本书的其余部分也是基于 Python。最重要的是，Python 带来了许多我们可以利用的内置基础设施，特别是

+   *parsers*，它将 Python 代码转换为抽象语法树（AST）表示。

+   *unparsers*，它接受一个 AST 并将其转换回 Python 代码。

这使我们能够利用在 AST 上操作的语法，而不是具体语法，从而大大降低复杂性。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('Nr1xbKj_WRQ') 
```

**先决条件**

+   您必须阅读关于 Fuzzing with Grammars 的章节，以了解语法和基于语法的测试是如何工作的。

## 概述

要使用本章提供的代码[Importing.html]，请编写

```py
>>> from fuzzingbook.PythonFuzzer import <identifier> 
```

然后利用以下功能。

本章提供了一个 `PythonFuzzer` 类，允许生成任意的 Python 代码元素：

```py
>>> fuzzer = PythonFuzzer()
>>> print(fuzzer.fuzz())
def R():
    break 
```

默认情况下，`PythonFuzzer` 生成一个 *函数定义* - 即，如上所示的一组语句。您可以通过传递 `start_symbol` 参数来指定您想要的 Python 元素：

```py
>>> fuzzer = PythonFuzzer('<While>')
>>> print(fuzzer.fuzz())
while {set()[set():set():set()]}:
    C = set()
    D @= set()
    break
else:
    return 
```

这里是一个所有可能的起始符号列表。它们的名称反映了 [Python `ast` 模块文档](https://docs.python.org/3/library/ast.html)中的非终结符。

```py
>>> sorted(list(PYTHON_AST_GRAMMAR.keys()))
['<Assert>',
 '<Assign>',
 '<Attribute>',
 '<AugAssign>',
 '<BinOp>',
 '<BoolOp>',
 '<Break>',
 '<Call>',
 '<Compare>',
 '<Constant>',
 '<Continue>',
 '<Delete>',
 '<Dict>',
 '<EmptySet>',
 '<Expr>',
 '<For>',
 '<FunctionDef>',
 '<If>',
 '<List>',
 '<Module>',
 '<Name>',
 '<Pass>',
 '<Return>',
 '<Set>',
 '<Slice>',
 '<Starred>',
 '<Subscript>',
 '<Tuple>',
 '<UnaryOp>',
 '<While>',
 '<With>',
 '<arg>',
 '<arg_list>',
 '<args>',
 '<args_param>',
 '<arguments>',
 '<bool>',
 '<boolop>',
 '<cmpop>',
 '<cmpop_list>',
 '<cmpops>',
 '<decorator_list_param>',
 '<defaults_param>',
 '<digit>',
 '<digits>',
 '<expr>',
 '<expr_list>',
 '<exprs>',
 '<float>',
 '<func>',
 '<id>',
 '<id_continue>',
 '<id_start>',
 '<identifier>',
 '<integer>',
 '<keyword>',
 '<keyword_list>',
 '<keywords>',
 '<keywords_param>',
 '<kw_defaults_param>',
 '<kwarg>',
 '<kwonlyargs_param>',
 '<lhs_Attribute>',
 '<lhs_List>',
 '<lhs_Name>',
 '<lhs_Starred>',
 '<lhs_Subscript>',
 '<lhs_Tuple>',
 '<lhs_expr>',
 '<lhs_exprs>',
 '<literal>',
 '<mod>',
 '<none>',
 '<nonempty_expr_list>',
 '<nonempty_lhs_expr_list>',
 '<nonempty_stmt_list>',
 '<nonzerodigit>',
 '<not_double_quotes>',
 '<not_single_quotes>',
 '<operator>',
 '<orelse_param>',
 '<posonlyargs_param>',
 '<returns>',
 '<start>',
 '<stmt>',
 '<stmt_list>',
 '<stmts>',
 '<string>',
 '<type_comment>',
 '<type_ignore>',
 '<type_ignore_list>',
 '<type_ignore_param>',
 '<type_ignores>',
 '<unaryop>',
 '<vararg>',
 '<withitem>',
 '<withitem_list>',
 '<withitems>'] 
```

如果您想对 Python 代码生成有更多控制，以下是幕后发生的事情。EBNF 语法 `PYTHON_AST_GRAMMAR` 可以解析并生成 Python 的 *抽象语法树*。要生成没有 `PythonFuzzer` 的 Python 模块，您需要采取以下步骤：

**步骤 1:** 创建一个适合 `ISLaSolver`（或任何其他语法模糊器）的非 EBNF 语法：

```py
>>> python_ast_grammar = convert_ebnf_grammar(PYTHON_AST_GRAMMAR) 
```

**步骤 2:** 将生成的语法输入到语法模糊器（如 ISLa）中：

```py
>>> solver = ISLaSolver(python_ast_grammar, start_symbol='<FunctionDef>') 
```

**步骤 3:** 让语法模糊器生成一个字符串。这个字符串代表一个 AST。

```py
>>> ast_string = str(solver.solve())
>>> ast_string
'FunctionDef(name=\'y\', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])' 
```

**步骤 4:** 将 AST 转换为实际的 Python AST 数据结构。

```py
>>> from [ast](https://docs.python.org/3/library/ast.html) import *
>>> abstract_syntax_tree = eval(ast_string) 
```

**步骤 5:** 最后，将 AST 结构转换回可读的 Python 代码：

```py
>>> ast.fix_missing_locations(abstract_syntax_tree)
>>> print(ast.unparse(abstract_syntax_tree))
@set()
def y():
    return 
```

本章有许多其他应用，包括解析和修改 Python 代码、进化模糊测试等。

这里是 `PythonFuzzer` 构造函数的详细信息：

`PythonFuzzer(self, start_symbol: Optional[str] = None, *, grammar: Optional[Dict[str, List[Union[str, Tuple[str, Dict[str, Any]]]]]] = None, constraint: Optional[str] = None, **kw_params) -> None`

生成 Python 代码。参数包括：

+   `start_symbol`：要生成的语法实体（默认：`<FunctionDef>`）

+   `grammar`：要使用的 EBNF 语法（默认：`PYTHON__AST_GRAMMAR`）；并且

+   `constraint` 一个 ISLa 约束（如果有）。

额外的关键字参数传递给 `ISLaSolver` 超类。

<svg width="248pt" height="152pt" viewBox="0.00 0.00 248.12 152.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 148.25)"><g id="node1" class="node"><title>PythonFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class PythonFuzzer:

生成 Python 代码。"><text text-anchor="start" x="8" y="-43.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">PythonFuzzer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="PythonFuzzer"><g id="a_node1_1"><a xlink:href="#" xlink:title="__init__(self, start_symbol: Optional[str] = None, *, grammar: Optional[Dict[str, List[Union[str, Tuple[str, Dict[str, Any]]]]]] = None, constraint: Optional[str] = None, **kw_params) -> None:

生成 Python 代码。参数包括：

* `start_symbol`: 要生成的语法实体（默认: `<FunctionDef>`）

* `grammar`: 要使用的 EBNF 语法（默认: `PYTHON__AST_GRAMMAR`）；以及

* `constraint` 一个 ISLa 约束（如果有）。

额外的关键字参数传递给 `ISLaSolver` 超类。"><text text-anchor="start" x="21.5" y="-21.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="fuzz(self) -> str:

生成 Python 代码字符串。"><text text-anchor="start" x="21.5" y="-8.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">fuzz()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>ISLaSolver</title> <g id="a_node2"><a xlink:href="isla.solver.ipynb" xlink:title="class ISLaSolver:

ISLa 公式/约束的求解器类。其顶级方法包括

:meth:`~isla.solver.ISLaSolver.solve`

用于为 ISLa 约束生成解决方案。

:meth:`~isla.solver.ISLaSolver.check`

用于检查给定输入是否满足 ISLa 约束。

:meth:`~isla.solver.ISLaSolver.parse`

用于解析和验证输入。

:meth:`~isla.solver.ISLaSolver.repair`

用于修复输入，使其满足约束。

:meth:`~isla.solver.ISLaSolver.mutate`

使用它来变异输入，使得结果满足一个约束。《ISLaSolver》<text text-anchor="start" x="18.88" y="-127.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">ISLa 求解器</text> <g id="a_node2_3"><a xlink:href="#" xlink:title="ISLa 求解器"><g id="a_node2_4"><a xlink:href="isla.solver.ipynb" xlink:title="__init__(self, grammar: Union[Dict[str, List[str]], str], formula: Union[isla.language.Formula, str, NoneType] = None, structural_predicates: Set[isla.language.StructuralPredicate] = frozenset({StructuralPredicate(name='nth', arity=3, eval_fun=<function is_nth>), StructuralPredicate(name='inside', arity=2, eval_fun=<function in_tree>), StructuralPredicate(name='same_position', arity=2, eval_fun=<function is_same_position>), StructuralPredicate(name='consecutive', arity=2, eval_fun=<function consecutive>), StructuralPredicate(name='different_position', arity=2, eval_fun=<function is_different_position>), StructuralPredicate(name='before', arity=2, eval_fun=<function is_before>), StructuralPredicate(name='level', arity=4, eval_fun=<function level_check>), StructuralPredicate(name='direct_child', arity=2, eval_fun=<function is_direct_child>), StructuralPredicate(name='after', arity=2, eval_fun=<function is_after>)}), semantic_predicates: Set[isla.language.SemanticPredicate] = frozenset({SemanticPredicate(count, 3)}), max_number_free_instantiations: int = 10, max_number_smt_instantiations: int = 10, max_number_tree_insertion_results: int = 5, enforce_unique_trees_in_queue: bool = False, debug: bool = False, cost_computer: Optional[ForwardRef('CostComputer')] = None, timeout_seconds: Optional[int] = None, global_fuzzer: bool = False, predicates_unique_in_int_arg: Tuple[isla.language.SemanticPredicate, ...] = (SemanticPredicate(count, 3),), fuzzer_factory: Callable[[Dict[str, List[str]]], isla.fuzzer.GrammarFuzzer] = <function SolverDefaults.<lambda>>, tree_insertion_methods: Optional[int] = None, activate_unsat_support: bool = False, grammar_unwinding_threshold: int = 4, initial_tree: isla.helpers.Maybe[isla.derivation_tree.DerivationTree] = Maybe(a=None), enable_optimized_z3_queries: bool = True, start_symbol: Optional[str] = None):>

:class:`~isla.solver.ISLaSolver`的构造函数接受大量的

参数。然而，除了第一个参数：code:`grammar`之外，其他都是可选的。

构建 ISLa 求解器的最简单方法就是只向它提供一个

仅语法；然后它就像一个语法模糊器。

>>> import random

>>> random.seed(1)

>>> import string

>>> LANG_GRAMMAR = {

... &nbsp;&nbsp;&nbsp;&nbsp;“<start>”:

... &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&quot;<stmt>&quot;],

... &nbsp;&nbsp;&nbsp;&nbsp;“<stmt>”:

... &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&quot;<assgn> ; <stmt>&quot;, &quot;<assgn>&quot;],

... &nbsp;&nbsp;&nbsp;&nbsp;“<assgn>”:

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

:param grammar: 基础语法；可以是 &quot;Fuzzing Book&quot; 字典

或在 BNF 语法中。

:param formula: 要解决的公式；可以是字符串或易于解析的

公式。如果没有给出公式，则假定默认的 `true` 约束，并且

当解算器回退到语法 fuzzer 时。产生的解决方案的数量

然后将由 `max_number_free_instantiations` 绑定。

:param structural_predicates: 解析公式时使用的结构谓词

公式。

:param semantic_predicates: 解析公式时使用的语义谓词

:param max_number_free_instantiations: 非终结符实例化的次数

应该由基于覆盖率的 fuzzer 展开，不受任何公式约束。

:param max_number_smt_instantiations: SMT 公式的解的数量

应该被产生。

:param max_number_tree_insertion_results: 使用树插入方法时的最大结果数

通过树插入解决存在量词。

:param enforce_unique_trees_in_queue: 如果为真，则队列中与同一树相同的

已经存在的树在队列中被丢弃，无论其

约束。

:param debug: 如果为真，则关于状态演变的调试信息

收集，特别是在状态树字段。树的根在

字段 state_tree_root。字段 costs 存储计算的成本值

所有新节点。

:param cost_computer: 用于计算相关成本的 `CostComputer` 类

到放置状态在 ISLa 的队列中。

:param timeout_seconds: 解算器将在多少秒后终止。

:param global_fuzzer: 如果设置为 True，则仅使用一个覆盖率引导的语法 fuzzer

对象用于完成无约束的开放推导树。

整个生成时间。这可能对某些目标有益；例如，我们

经验表明，使用 CSV 可以显著提高速度。然而，实现的 k-path

覆盖率可能会降低。

:param predicates_unique_in_int_arg: 在某些情况下需要此参数

实例化全称整数量词。提供的谓词应该

恰好有一个整数参数，并且对于恰好一个整数值

一次所有其他参数都固定。

:param fuzzer_factory: 用于实例化的 fuzzer 构造函数

&quot;free&quot; 非终结符。

:param tree_insertion_methods: 要使用的存在量词树插入方法的组合

通过树插入消除量词。全选：`DIRECT_EMBEDDING &amp;

SELF_EMBEDDING &amp; CONTEXT_ADDITION`.

:param activate_unsat_support: 如果假设公式可能

触发针对不可满足性的附加测试。这

减少输入生成性能，但可能确保终止（带有

对于不可满足的问题，如果求解器可以

否则可能会发散。

:param grammar_unwinding_threshold: 当查询 SMT 求解器时，ISLa 传递一个

定义的正规表达式用于涉及的非终结符的语法。如果这个

语法不是正则的，我们在参考语法中展开相应的部分

深度达到`grammar_unwinding_threshold`。如果这个深度太浅，它可能会

发生方程等无法解决的情况；如果它太深，它可能会

对性能产生负面影响（并且非常巨大）。

:param initial_tree: 如果求解器应该使用队列的初始输入树

不会从树`(<start>, None)`开始。

:param enable_optimized_z3_queries: 启用 Z3 查询的预处理（主要是

与长度等事物相关的数值问题。这可以提高性能

显著；然而，可能存在某些问题无法解决

不再需要。在这种情况下，这个选项可以被/应该被禁用。

:param start_symbol: 这是`initial_tree`的替代方案，用于以

一个不同于`<start>`的起始符号。如果提供了`start_symbol`，则树

由一个具有`start_symbol`值的单个根节点组成的树被选择为

初始树。"><text text-anchor="start" x="21.5" y="-105.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>PythonFuzzer->ISLaSolver</title></g> <g id="node3" class="node"><title>图例</title> <text text-anchor="start" x="120.88" y="-46.25" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="120.88" y="-36.25" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="126.88" y="-36.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="120.88" y="-26.25" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="126.88" y="-26.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="120.88" y="-16.25" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="126.88" y="-16.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="120.88" y="-7.2" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 具体代码的语法

要*生成*代码，用*具体*语法编写语法规则相当容易。如果我们想生成，比如说，算术表达式，我们可以轻松地创建一个具体的语法规则，它正好能完成这个任务。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from Grammars import Grammar
from Grammars import is_valid_grammar, convert_ebnf_grammar, extend_grammar, trim_grammar 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import Optional 
```

我们使用[Fuzzingbook 格式定义语法](https://www.fuzzingbook.org/html/Grammars.html)，其中语法被表示为从符号到展开备选列表的字典。

```py
EXPR_GRAMMAR: Grammar = {
    "<start>":
        ["<expr>"],

    "<expr>":
        ["<term> + <expr>", "<term> - <expr>", "<term>"],

    "<term>":
        ["<factor> * <term>", "<factor> / <term>", "<factor>"],

    "<factor>":
        ["+<factor>",
         "-<factor>",
         "(<expr>)",
         "<integer>.<integer>",
         "<integer>"],

    "<integer>":
        ["<digit><integer>", "<digit>"],

    "<digit>":
        ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
} 
```

```py
assert is_valid_grammar(EXPR_GRAMMAR) 
```

我们可以使用这个语法生成语法上有效的算术表达式。我们使用 ISLa 求解器作为我们的生成器，因为它是功能最强大的；但在这个阶段，我们也可以使用任何其他我们的语法模糊器，比如 GrammarFuzzer。

```py
from [isla.solver](https://rindphi.github.io/isla/) import ISLaSolver 
```

这里是从语法中产生的具体输入：

```py
expr_solver = ISLaSolver(EXPR_GRAMMAR)
for _ in range(10):
    print(expr_solver.solve()) 
```

```py
4.3 + 512 / -(7 / 6 - 0 / 9 * 1 * 1) * +8.3 / 7 * 4 / 6
(4 / 7 + 1) / (4) / 9 / 8 + 4 / (3 + 6 - 7)
+--(--(-9) * (4 * 7 + (4) + 4) + --(+(3)) - 6 + 0 / 7 + 7)
(2 * 6 + 0 - 5) * 4 - +1 * (2 - 2) / 8 / 6
(+-(0 - (1) * 7 / 3)) / ((1 * 3 + 8) + 9 - +1 / --0) - 5 * (-+939.491)
+2.9 * 0 / 501.19814 / --+--(6.05002)
+-8.8 / (1) * -+1 + -8 + 9 - 3 / 8 * 6 + 4 * 3 * 5
(+(8 / 9 - 1 - 7)) + ---06.30 / +4.39
8786.82 - +01.170 / 9.2 - +(7) + 1 * 9 - 0
+-6 * 0 / 5 * (-(1.7 * +(-1 / +4.9 * 5 * 1 * 2) + -4.2 + (6 + -5) / (4 * 3 + 4)))

```

我们可以将语法进一步扩展，使其也能生成赋值和其他语句，逐步覆盖编程语言的整个语法。然而，这并不是一个好主意。为什么？

问题在于，当测试*编译器*时，你不仅想要能够*生成*代码，还想要能够*解析*代码，这样你就可以随意对其进行变异和操作。这正是我们的“具体”语法会给我们带来问题的地方。虽然我们可以轻松解析严格遵循语法的代码（或表达式）...

```py
expr_solver.check('2 + 2') 
```

```py
True

```

...一个空格就足以让它失败...

```py
expr_solver.check('2 +  2') 
```

```py
Error parsing "2 +  2" starting with "<start>"

```

```py
False

```

...以及空格的缺失：

```py
expr_solver.check('2+2') 
```

```py
Error parsing "2+2" starting with "<start>"

```

```py
False

```

事实上，在大多数编程语言中，空格是可选的。我们可以更新我们的语法规则，使其能够始终处理可选的空格（引入一个`<space>`非终结符）。但这样，还有其他特性，比如*注释*...

```py
expr_solver.check('2 + 2    # should be 4') 
```

```py
Error parsing "2 + 2    # should be 4" starting with "<start>"

```

```py
False

```

...或者*续行*...

```py
expr_solver.check('2 + \\\n2')  # An expression split over two lines 
```

```py
Error parsing "2 + \
2" starting with "<start>"

```

```py
False

```

我们语法需要覆盖的内容。

此外，还有一些语言特性甚至无法在上下文无关语法中正确表示：

+   例如，在 C 编程语言中，解析器需要知道一个标识符是否被定义为*类型*

+   在 Python 中，*缩进*级别不能用上下文无关语法表示。

因此，通常一个好的做法是使用专门的*解析器*（或*预处理器*）将输入转换为更*抽象*的表示——通常是*树*结构。在编程语言中，这样的树被称为*抽象语法树*（AST）；它是编译器操作的数据结构。

## 抽象语法树

表示程序代码的抽象语法树（ASTs）是世界上（如果不是*最复杂*的数据结构）最复杂的数据结构之一——这主要是因为它们反映了编程语言及其特性的所有复杂性。好消息是，在 Python 中，处理 AST 特别容易——可以使用标准语言特性来处理它们。

让我们用一个例子来说明 AST。这是我们想要处理的代码片段：

```py
def main():
    print("Hello, world!")  # A simple example 
```

```py
main() 
```

```py
Hello, world!

```

让我们获取这个函数的源代码：

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
main_source = inspect.getsource(main)
print(main_source) 
```

```py
def main():
    print("Hello, world!")  # A simple example

```

我们使用[Python AST 模块](https://docs.python.org/3/library/ast.html)将这段代码字符串转换为 AST 并返回。

```py
import [ast](https://docs.python.org/3/library/ast.html) 
```

使用 `ast.parse()`，我们可以将 `main()` 源代码解析成 AST：

```py
main_tree = ast.parse(main_source) 
```

这就是这个树看起来像什么：

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import show_ast 
```

```py
show_ast(main_tree) 
```

<svg xmlns:xlink="http://www.w3.org/1999/xlink" width="369pt" height="332pt" viewBox="0.00 0.00 368.62 332.00"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 328)"><g id="node1" class="node"><title>0</title> <text text-anchor="start" x="83.38" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">FunctionDef</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="32.75" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"main"</text></g> <g id="edge1" class="edge"><title>0--1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="128.75" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">arguments</text></g> <g id="edge2" class="edge"><title>0--2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="start" x="202.25" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Expr</text></g> <g id="edge3" class="edge"><title>0--3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="start" x="202.25" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Call</text></g> <g id="edge4" class="edge"><title>3--4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="start" x="159.25" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge5" class="edge"><title>4--5</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="start" x="242.75" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge8" class="edge"><title>4--8</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="93.75" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"print"</text></g> <g id="edge6" class="edge"><title>5--6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="175.75" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge7" class="edge"><title>5--7</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="290.75" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"Hello, world!"</text></g> <g id="edge9" class="edge"><title>8--9</title></g></g></svg>

我们看到函数定义已经变成了 `FunctionDef` 节点，其第三个子节点是一个 `Expr` 节点，它又变成了一个 `Call` ——调用 `"print"` 函数，参数为 `"Hello, world!"`。

这些 AST 节点每个都是一个 *构造函数* ——也就是说，我们可以调用 `FunctionDef()` 来获取一个函数定义节点，或者调用 `Call()` 来获取一个调用节点。这些构造函数将 AST *子节点* 作为参数，但也可以接受很多 *可选* 参数（我们迄今为止还没有使用）。将 AST *转储* 到字符串中会显示每个构造函数的所有参数：

```py
print(ast.dump(main_tree, indent=4)) 
```

```py
Module(
    body=[
        FunctionDef(
            name='main',
            args=arguments(
                posonlyargs=[],
                args=[],
                kwonlyargs=[],
                kw_defaults=[],
                defaults=[]),
            body=[
                Expr(
                    value=Call(
                        func=Name(id='print', ctx=Load()),
                        args=[
                            Constant(value='Hello, world!')],
                        keywords=[]))],
            decorator_list=[])],
    type_ignores=[])

```

[Python ast 文档](https://docs.python.org/3/library/ast.html) 列出了所有这些构造函数，它们构成了抽象语法。有超过 100 个单独的构造函数！（我们说过 AST 是复杂的，对吧？）

上述字符串表示的不错之处在于我们可以 *直接* 使用它并将其转换成树：

```py
from [ast](https://docs.python.org/3/library/ast.html) import * 
```

```py
my_main_tree = Module(
    body=[
        FunctionDef(
            name='main',
            args=arguments(
                posonlyargs=[],
                args=[],
                kwonlyargs=[],
                kw_defaults=[],
                defaults=[]),
            body=[
                Expr(
                    value=Call(
                        func=Name(id='print', ctx=Load()),
                        args=[
                            Constant(value='Hello, world!')],
                        keywords=[]))],
            decorator_list=[])],
    type_ignores=[]) 
```

我们可以将这个树编译成可执行代码：

```py
my_main_tree = fix_missing_locations(my_main_tree)  # required for trees built from constructors
my_main_code = compile(my_main_tree, filename='<unknown>', mode='exec') 
```

```py
del main  # This deletes the definition of main() 
```

```py
exec(my_main_code)  # This defines main() again from `code` 
```

```py
main() 
```

```py
Hello, world!

```

我们还可以 *反解析* 树（即再次将其转换为源代码）。（注意在解析过程中注释是如何丢失的。）

```py
print(ast.unparse(my_main_tree)) 
```

```py
def main():
    print('Hello, world!')

```

因此，我们可以

1.  *解析* 具体代码到 AST（使用 `ast.parse()`）

1.  *生成* 新的 AST 和 *修改* 现有的 AST

1.  *反解析* AST 以获得具体的代码（使用 `ast.unparse()`）

为了 *生成* 和 *修改* AST（如上所述的第二步），我们需要产生 *正确* 的 AST 的手段，调用所有构造函数并使用正确的参数。因此，我们有一个 *AST 语法*，它可以（并解析）生成我们想要的 AST。

## ASTs 的语法

程序设计语言的语法是围绕中最复杂的形式语法之一，AST 反映了其中很多复杂性。我们将使用 Python 文档中指定的 [抽象 AST 语法](https://docs.python.org/3/library/ast.html) 作为基础，并逐步构建形式上下文无关语法。

### 常量

我们将从一个简单的常量开始——字符串和整数。同样，我们使用 `fuzzingbook` 语法，因为它允许更容易的扩展。

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
ANYTHING_BUT_DOUBLE_QUOTES_AND_BACKSLASH = (string.digits + string.ascii_letters + string.punctuation + ' ').replace('"', '').replace('\\', '')
ANYTHING_BUT_SINGLE_QUOTES_AND_BACKSLASH = (string.digits + string.ascii_letters + string.punctuation + ' ').replace("'", '').replace('\\', '') 
```

```py
ANYTHING_BUT_DOUBLE_QUOTES_AND_BACKSLASH 
```

```py
"0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!#$%&'()*+,-./:;<=>?@[]^_`{|}~ "

```

```py
ANYTHING_BUT_SINGLE_QUOTES_AND_BACKSLASH 
```

```py
'0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&()*+,-./:;<=>?@[]^_`{|}~ '

```

```py
PYTHON_AST_CONSTANTS_GRAMMAR: Grammar = {
    '<start>': [ '<expr>' ],

    # Expressions
    '<expr>': [ '<Constant>', '<Expr>' ],
    '<Expr>': [ 'Expr(value=<expr>)' ],

    # Constants
    '<Constant>': [ 'Constant(value=<literal>)' ],
    '<literal>': [ '<string>', '<integer>', '<float>', '<bool>', '<none>' ],

    # Strings
    '<string>': [ '"<not_double_quotes>*"', "'<not_single_quotes>*'" ],
    '<not_double_quotes>': list(ANYTHING_BUT_DOUBLE_QUOTES_AND_BACKSLASH),
    '<not_single_quotes>': list(ANYTHING_BUT_SINGLE_QUOTES_AND_BACKSLASH),
    # FIXME: The actual rules for Python strings are also more complex:
    # https://docs.python.org/3/reference/lexical_analysis.html#numeric-literals

    # Numbers
    '<integer>': [ '<digit>', '<nonzerodigit><digits>' ],
    '<float>': [ '<integer>.<integer>' ],
    '<nonzerodigit>': ['1', '2', '3', '4', '5', '6', '7', '8', '9'],
    '<digits>': [ '<digit><digits>', '<digit>' ],
    '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
    # FIXME: There are _many_ more ways to express numbers in Python; see
    # https://docs.python.org/3/reference/lexical_analysis.html#numeric-literals

    # More
    '<bool>': [ 'True', 'False' ],
    '<none>': [ 'None' ],

    # FIXME: Not supported: bytes, format strings, regex strings...
} 
```

注意，我们在语法中使用 *扩展的巴科斯-诺尔范式*（这里：`<string>`）：

+   `<elem>+` 表示 `<elem>` 的一个或多个实例；

+   `<elem>*` 表示 `<elem>` 的零个或多个实例；

+   `<elem>?` 表示 `<elem>` 的一个或零个实例。

调用 `is_valid_grammar()` 确保我们的语法没有常见的错误。不要在没有它的情况下编写语法！

```py
assert is_valid_grammar(PYTHON_AST_CONSTANTS_GRAMMAR) 
```

```py
constants_grammar = convert_ebnf_grammar(PYTHON_AST_CONSTANTS_GRAMMAR)
constants_solver = ISLaSolver(constants_grammar)
constants_tree_str = str(constants_solver.solve())
print(constants_tree_str) 
```

```py
Expr(value=Constant(value=None))

```

我们可以从这个表达式创建一个 AST，并将其转换为 Python 代码（好吧，一个字面量）：

```py
constants_tree = eval(constants_tree_str)
ast.unparse(constants_tree) 
```

```py
'None'

```

让我们做几次：

```py
def test_samples(grammar: Grammar, iterations: int = 10, start_symbol = None, log: bool = True):
    g = convert_ebnf_grammar(grammar)
    solver = ISLaSolver(g, start_symbol=start_symbol, max_number_free_instantiations=iterations)
    for i in range(iterations):
        tree_str = str(solver.solve())
        tree = eval(tree_str)
        ast.fix_missing_locations(tree)
        if log:
            code = ast.unparse(tree)
            print(f'{code:40} # {tree_str}') 
```

```py
test_samples(PYTHON_AST_CONSTANTS_GRAMMAR) 
```

```py
False                                    # Expr(value=Constant(value=False))
2                                        # Constant(value=2)
None                                     # Constant(value=None)
'#'                                      # Constant(value="#")
550.81                                   # Constant(value=550.81)
True                                     # Constant(value=True)
'.'                                      # Constant(value='.')
467                                      # Constant(value=467)
7894                                     # Constant(value=7894)
263                                      # Constant(value=263)

```

我们的语法也可以 *解析* 从具体代码中获得的 AST。

```py
sample_constant_code = "4711"
sample_constant_ast = ast.parse(sample_constant_code).body[0]  # get the `Expr` node
sample_constant_ast_str = ast.dump(sample_constant_ast)
print(sample_constant_ast_str) 
```

```py
Expr(value=Constant(value=4711))

```

```py
constant_solver = ISLaSolver(constants_grammar)
constant_solver.check(sample_constant_ast_str) 
```

```py
True

```

让我们提出一个测验问题：*我们的语法支持负数吗？* 为了这个，我们首先找出 `Constant()` 构造函数是否也接受一个 *负数* 作为参数？结果是它可以：

```py
ast.unparse(Constant(value=-1)) 
```

```py
'-1'

```

但如果我们解析一个负数，比如 `-1`，会发生什么？有人可能会假设这仅仅会得到一个 `Constant(-1)`，对吧？自己试试看！

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import quiz 
```

### 习题

如果我们解析一个负数，我们会得到

答案是解析 `-1` 会得到一个一元减号 `USub()` 应用到一个正数上：

```py
print(ast.dump(ast.parse('-1'))) 
```

```py
Module(body=[Expr(value=UnaryOp(op=USub(), operand=Constant(value=1)))], type_ignores=[])

```

由于一元运算符（目前）不是我们语法的组成部分，它无法处理负数：

```py
sample_constant_code = "-1"
sample_constant_ast = ast.parse(sample_constant_code).body[0]  # get the `Expr` node
sample_constant_ast_str = ast.dump(sample_constant_ast)
constant_solver = ISLaSolver(constants_grammar)
constant_solver.check(sample_constant_ast_str) 
```

```py
Error parsing "Expr(value=UnaryOp(op=USub(), operand=Constant(value=1)))" starting with "<start>"

```

```py
False

```

在接下来的几节中，我们将逐步扩展我们的语法，加入越来越多的 Python 特性，最终覆盖（几乎）整个语言。

<details id="Excursion:-Composites"><summary>复合结构</summary>

让我们添加复合常量——列表、字典、元组等。这些在 AST 中的表示如下：

```py
print(ast.dump(ast.parse("{ 'a': set() }"), indent=4)) 
```

```py
Module(
    body=[
        Expr(
            value=Dict(
                keys=[
                    Constant(value='a')],
                values=[
                    Call(
                        func=Name(id='set', ctx=Load()),
                        args=[],
                        keywords=[])]))],
    type_ignores=[])

```

让我们将这些编码到语法中，再次使用来自[抽象 AST 语法](https://docs.python.org/3/library/ast.html)的定义。所有这些结构也接受*上下文*，其中标识符被使用——`Load()` 如果它们用于评估，`Store()` 如果它们出现在赋值的左侧（是的，在 Python 中，你可以在赋值的左侧有一个元组，比如 `(x, y) = (1, 2)`），以及 `Del()` 如果它们用作 `del` 语句的操作数。目前，我们只交替使用 `Load()` 和 `Del()`。

```py
PYTHON_AST_COMPOSITES_GRAMMAR: Grammar = extend_grammar(
    PYTHON_AST_CONSTANTS_GRAMMAR, {
    '<expr>': PYTHON_AST_CONSTANTS_GRAMMAR['<expr>'] + [
        '<Dict>', '<Set>', '<List>', '<Tuple>'
    ],

    '<Dict>': [ 'Dict(keys=<expr_list>, values=<expr_list>)' ],
    '<Set>': [ 'Set(elts=<nonempty_expr_list>)', '<EmptySet>' ],
    '<EmptySet>': [ 'Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])' ],
    '<List>': [
        'List(elts=<expr_list>, ctx=Load())',
        'List(elts=<expr_list>, ctx=Del())',
    ],
    '<Tuple>': [
        'Tuple(elts=<expr_list>, ctx=Load())',
        'Tuple(elts=<expr_list>, ctx=Del())',
    ],

    # Lists of expressions
    '<expr_list>': [ '[<exprs>?]' ],
    '<nonempty_expr_list>': [ '[<exprs>]' ],
    '<exprs>': [ '<expr>', '<exprs>, <expr>' ],
}) 
```

```py
assert is_valid_grammar(PYTHON_AST_COMPOSITES_GRAMMAR) 
```

```py
for elt in [ '<Constant>', '<Dict>', '<Set>', '<List>', '<Tuple>' ]:
    print(elt)
    test_samples(PYTHON_AST_COMPOSITES_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<Constant>
'c'                                      # Constant(value='c')
96.7                                     # Constant(value=96.7)
None                                     # Constant(value=None)
False                                    # Constant(value=False)
505                                      # Constant(value=505)
'U'                                      # Constant(value="U")
True                                     # Constant(value=True)
41398                                    # Constant(value=41398)
24                                       # Constant(value=24)
72                                       # Constant(value=72)

<Dict>
{}                                       # Dict(keys=[], values=[List(elts=[Dict(keys=[List(elts=[Constant(value=9.63)], ctx=Load())], values=[Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Set(elts=[Constant(value=True), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])], ctx=Load())]), Constant(value=2), Tuple(elts=[Constant(value=''), Constant(value=False), Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), Expr(value=List(elts=[Constant(value=None)], ctx=Load())), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del())], ctx=Del())])
{577: ''}                                # Dict(keys=[Constant(value=577), Constant(value=34), Constant(value=286), Constant(value=7051)], values=[Constant(value="")])
{90: 14}                                 # Dict(keys=[Constant(value=90)], values=[Constant(value=14), Constant(value=88), Constant(value=435)])
{"nF}[ (^{bXBrwzf-P@geW'.]~G>;O2i&/t7Cc5:QU1jR4q_8VJ)Hsxd#o*aT3Sv!$ku?IhMpmA,EL0ZN=`9yK|<Y6lD+%I": 'Gym]A&K;70{jJLC"DV)/Y S.eNMEQq^%?i+-b!hz|gcUBvW485O#pPu~d:(F>_<a}kI2norf9H[T,lXt=w6@Z*1$xs`"R3'} # Dict(keys=[Constant(value="nF}[ (^{bXBrwzf-P@geW'.]~G>;O2i&/t7Cc5:QU1jR4q_8VJ)Hsxd#o*aT3Sv!$ku?IhMpmA,EL0ZN=`9yK|<Y6lD+%I")], values=[Constant(value='Gym]A&K;70{jJLC"DV)/Y S.eNMEQq^%?i+-b!hz|gcUBvW485O#pPu~d:(F>_<a}kI2norf9H[T,lXt=w6@Z*1$xs`"R3')])
{}                                       # Dict(keys=[], values=[])
{}                                       # Dict(keys=[], values=[])
{}                                       # Dict(keys=[], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Constant(value=True), Constant(value=687596.53), Dict(keys=[Set(elts=[Expr(value=Set(elts=[Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), Constant(value=34.676)]))])], values=[Set(elts=[Set(elts=[List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load())], ctx=Del())])])]), List(elts=[], ctx=Load())])
{}                                       # Dict(keys=[], values=[])
{}                                       # Dict(keys=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], values=[])
{}                                       # Dict(keys=[Tuple(elts=[], ctx=Del())], values=[])

<Set>
{
[], 79.2}                              # Set(elts=[Expr(value=List(elts=[], ctx=Del())), Constant(value=79.2)])
set()                                    # Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])
{{{False: [set()], None: []}, (({20: set()},),)}} # Set(elts=[Set(elts=[Dict(keys=[Constant(value=False), Constant(value=None)], values=[List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load()), List(elts=[], ctx=Del())]), Tuple(elts=[Tuple(elts=[Dict(keys=[Constant(value=20)], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Constant(value=True), List(elts=[], ctx=Del())])], ctx=Load())], ctx=Del())])])
{'Z'}                                    # Set(elts=[Constant(value='Z')])
{3763, ''}                               # Set(elts=[Constant(value=3763), Constant(value="")])
{475, 136, 95, 841, 58}                  # Set(elts=[Constant(value=475), Constant(value=136), Constant(value=95), Constant(value=841), Constant(value=58)])
{"F3Ye]1UZz&sPrG:D-R`k?5d+SM,/4b!uE fW;L$)@oQ'h^qI[(lXgN0wmt=~Jav86|Vp%72CcOBj_nHK<9A*#i}yTx>{."} # Set(elts=[Constant(value="F3Ye]1UZz&sPrG:D-R`k?5d+SM,/4b!uE fW;L$)@oQ'h^qI[(lXgN0wmt=~Jav86|Vp%72CcOBj_nHK<9A*#i}yTx>{.")])
{66, 7}                                  # Set(elts=[Constant(value=66), Constant(value=7)])
{set(), '', None, '_P[', 'L}w,6'}        # Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Constant(value=''), Constant(value=None), Constant(value='_P['), Constant(value='L}w,6')])
{'51I{Ef&u;kThXbRo]cV/8)Q@W>4|=J7lHge"+^y%(rv<q.DM:najxi9OUG?!KS zsd2t-Fm3NApB#0$~C`*PY'} # Set(elts=[Constant(value='51I{Ef&u;kThXbRo]cV/8)Q@W>4|=J7lHge"+^y%(rv<q.DM:najxi9OUG?!KS zsd2t-Fm3NApB#0$~C`*PY')])

<List>
[[], {831.3: (7, set(), {('1',)})}]      # List(elts=[List(elts=[], ctx=Load()), Dict(keys=[Constant(value=831.30), Constant(value=None), Expr(value=Tuple(elts=[Constant(value=""), Constant(value=True), Constant(value=False)], ctx=Del()))], values=[Tuple(elts=[Constant(value=7), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Set(elts=[Tuple(elts=[Constant(value='1')], ctx=Load())])], ctx=Load())])], ctx=Del())
[22]                                     # List(elts=[Constant(value=22)], ctx=Load())
[64]                                     # List(elts=[Constant(value=64)], ctx=Load())
[56]                                     # List(elts=[Constant(value=56)], ctx=Del())
[9589]                                   # List(elts=[Constant(value=9589)], ctx=Load())
[780]                                    # List(elts=[Constant(value=780)], ctx=Del())
[164, 47]                                # List(elts=[Constant(value=164), Constant(value=47)], ctx=Load())
["^dG@0 N26zE73qSfX,>xhPlW#j.1cQO4bF+A:LZR'CT=$i_", 'tJI`]gD_M/8yu!%<n~&H|9w*)Ur5sk(e}[vap?V-oK{BYm;eccmO'] # List(elts=[Constant(value="^dG@0 N26zE73qSfX,>xhPlW#j.1cQO4bF+A:LZR'CT=$i_"), Constant(value="tJI`]gD_M/8yu!%<n~&H|9w*)Ur5sk(e}[vap?V-oK{BYm;eccmO")], ctx=Load())
['e]@JX9LBnA:0Ha³KVf OWuFT%*8ZGtp/x`Cw"li|Mq?_UI45$)zNh#gDcs;!-d[,(~{>bYrE<.RQ27}&moSk+vjP=6y9'] # List(elts=[Constant(value='e]@JX9LBnA:0Ha³KVf OWuFT%*8ZGtp/x`Cw"li|Mq?_UI45$)zNh#gDcs;!-d[,(~{>bYrE<.RQ27}&moSk+vjP=6y9')], ctx=Load())
[set(), set()]                           # List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load())

<Tuple>
()                                       # Tuple(elts=[], ctx=Load())
(set(),)                                 # Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del())
(set(), [], 
1.4, [[None], True], {set(): (False, {set()})}) # Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), List(elts=[], ctx=Del()), Expr(value=Constant(value=1.4)), List(elts=[List(elts=[Constant(value=None)], ctx=Load()), Constant(value=True)], ctx=Load()), Dict(keys=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), Expr(value=Constant(value=False))], values=[Tuple(elts=[Constant(value=False), Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])], ctx=Load())])], ctx=Del())
('',)                                    # Tuple(elts=[Constant(value="")], ctx=Load())
(93,)                                    # Tuple(elts=[Constant(value=93)], ctx=Load())
(28371613, 51, 892, 45, 10678, '')       # Tuple(elts=[Constant(value=28371613), Constant(value=51), Constant(value=892), Constant(value=45), Constant(value=10678), Constant(value='')], ctx=Del())
(72, 632)                                # Tuple(elts=[Constant(value=72), Constant(value=632)], ctx=Load())
('p[R#U', '5JRh~3', 'aAI>V+LBk60Ogp')    # Tuple(elts=[Constant(value='p[R#U'), Constant(value="5JRh~3"), Constant(value="aAI>V+LBk60Ogp")], ctx=Load())
(363,)                                   # Tuple(elts=[Constant(value=363)], ctx=Del())
('a*wyz!$CcJ.TDj?<8Q`o}|fG~3%FX/O:r@YW5dK,MqLt^l&B(PbH1_ZInkimvSV4x> u{+2gs)h"e9NA;76]=E-0;',) # Tuple(elts=[Constant(value='a*wyz!$CcJ.TDj?<8Q`o}|fG~3%FX/O:r@YW5dK,MqLt^l&B(PbH1_ZInkimvSV4x> u{+2gs)h"e9NA;76]=E-0;')], ctx=Load())

```

你可能会遇到一些不常见的表达式。例如：

1.  `()` 是一个空元组。

1.  `(1,)` 是一个只有一个元素的元组。

1.  `{}` 是一个空字典；`{1}` 是一个只有一个元素的集合。

1.  空集合用 `set()` 表示。

我们使用 `set()` 来表示空集的事实实际上是我们的 `PYTHON_AST_COMPOSITES_GRAMMAR` 语法的一个特性。如果我们不提供任何元素就调用 `Set()` AST 构造函数，我们就能得到这个漂亮的表达式...

```py
print(ast.unparse(Set(elts=[]))) 
```

```py
{*()}

```

...这确实会评估为一个空集。

```py
{*()} 
```

```py
set()

```

从技术上来说，这些都是正确的，但我们希望坚持（某种程度上）更易读的代码。如果你想让你程序员的伙伴们感到困惑，总是使用 `{*()}` 而不是 `set()`。</details> <details id="Excursion:-Expressions"><summary>表达式</summary>

让我们通过添加*表达式*来扩展我们的语法。Python 解析器已经处理了优先级规则，因此我们可以以类似的方式处理所有一元和二元运算符。

```py
print(ast.dump(ast.parse("2 + 2 is not False"), indent=4)) 
```

```py
Module(
    body=[
        Expr(
            value=Compare(
                left=BinOp(
                    left=Constant(value=2),
                    op=Add(),
                    right=Constant(value=2)),
                ops=[
                    IsNot()],
                comparators=[
                    Constant(value=False)]))],
    type_ignores=[])

```

```py
PYTHON_AST_EXPRS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_COMPOSITES_GRAMMAR, {
    '<expr>': PYTHON_AST_COMPOSITES_GRAMMAR['<expr>'] + [
        '<BoolOp>', '<BinOp>', '<UnaryOp>', '<Compare>',
    ],

    # Booleans: and or
    '<BoolOp>': [ 'BoolOp(op=<boolop>, values=<expr_list>)' ],
    '<boolop>': [ 'And()', 'Or()' ],

    # Binary operators: + - * ...
    '<BinOp>': [ 'BinOp(left=<expr>, op=<operator>, right=<expr>)' ],
    '<operator>': [ 'Add()', 'Sub()', 'Mult()', 'MatMult()',
                   'Div()', 'Mod()', 'Pow()',
                   'LShift()', 'RShift()', 'BitOr()', 'BitXor()', 'BitAnd()',
                   'FloorDiv()' ],

    # Unary operators: not + - ...
    '<UnaryOp>': [ 'UnaryOp(op=<unaryop>, operand=<expr>)'],
    '<unaryop>': [ 'Invert()', 'Not()', 'UAdd()', 'USub()' ],

    # Comparisons: == != < <= > >= is in ...
    '<Compare>': [ 'Compare(left=<expr>, ops=<cmpop_list>, comparators=<expr_list>)'],
    '<cmpop_list>': [ '[<cmpops>?]' ],
    '<cmpops>': [ '<cmpop>', '<cmpop>, <cmpops>' ],
    '<cmpop>': [ 'Eq()', 'NotEq()', 'Lt()', 'LtE()', 'Gt()', 'GtE()',
                 'Is()', 'IsNot()', 'In()', 'NotIn()' ],

    # FIXME: There's a few more expressions: GeneratorExp, Await, YieldFrom, ...
}) 
```

```py
assert is_valid_grammar(PYTHON_AST_EXPRS_GRAMMAR) 
```

```py
for elt in [ '<BoolOp>', '<BinOp>', '<UnaryOp>', '<Compare>' ]:
    print(elt)
    test_samples(PYTHON_AST_EXPRS_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<BoolOp>
() and {-([]) / (set(), set()), {
True: set()}} # BoolOp(op=And(), values=[BoolOp(op=Or(), values=[]), Set(elts=[BinOp(left=UnaryOp(op=USub(), operand=Compare(left=List(elts=[], ctx=Del()), ops=[], comparators=[])), op=Div(), right=Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load())), Dict(keys=[Expr(value=Constant(value=True))], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), List(elts=[], ctx=Load())])])])
(set(), set(), set() @ set() | set() + set()) and set() ** (set() ^ set()) * set() # BoolOp(op=And(), values=[Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=BitOr(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))], ctx=Del()), BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
set() % (set() >> set()) - (set() << set()) or set() & set() # BoolOp(op=Or(), values=[BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=RShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=Sub(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
'8' or 6                                 # BoolOp(op=Or(), values=[Constant(value='8'), Constant(value=6)])
~+123.95                                 # BoolOp(op=Or(), values=[UnaryOp(op=Invert(), operand=UnaryOp(op=UAdd(), operand=Constant(value=123.95)))])
not False // None                        # BoolOp(op=Or(), values=[UnaryOp(op=Not(), operand=BinOp(left=Constant(value=False), op=FloorDiv(), right=Constant(value=None)))])
'S' and 6180 in 397494                   # BoolOp(op=And(), values=[Constant(value="S"), Compare(left=Constant(value=6180), ops=[In()], comparators=[Constant(value=397494)])])
41                                       # BoolOp(op=And(), values=[Constant(value=41)])
214                                      # BoolOp(op=Or(), values=[Constant(value=214)])
5818 and "N1qoR6ak 2UJTWyh>!B)/#YKe0]=w{E.-Q`F[5'&⁹cA~<V+M$bnLu%H8I3;g*D?rz7Xj:}pPvif_GOtx4,(ZCdmls|@YiT" and 70 and 884 # BoolOp(op=And(), values=[Constant(value=5818), Constant(value="N1qoR6ak 2UJTWyh>!B)/#YKe0]=w{E.-Q`F[5'&⁹cA~<V+M$bnLu%H8I3;g*D?rz7Xj:}pPvif_GOtx4,(ZCdmls|@YiT"), Constant(value=70), Constant(value=884)])

<BinOp>
{} - 33                                  # BinOp(left=Expr(value=Dict(keys=[], values=[Tuple(elts=[UnaryOp(op=Invert(), operand=List(elts=[List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load()), Tuple(elts=[], ctx=Del()), BoolOp(op=And(), values=[Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], ctx=Load())])], ctx=Del())), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Compare(left=Tuple(elts=[], ctx=Load()), ops=[], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]))], ctx=Del())])), op=Sub(), right=Constant(value=33))
set() / (set() << set()) * (set() >> set()) // (set() @ set() & set()) # BinOp(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Div(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=Mult(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=RShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=FloorDiv(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))
None ^ False                             # BinOp(left=Constant(value=None), op=BitXor(), right=Constant(value=False))
-'' + +7719.5                            # BinOp(left=UnaryOp(op=USub(), operand=Constant(value="")), op=Add(), right=UnaryOp(op=UAdd(), operand=Constant(value=7719.5)))
(set() or 906) >> ('F') | (not (True)) % ((set() and set()) << False) # BinOp(left=BinOp(left=BoolOp(op=Or(), values=[Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), Constant(value=906)]), op=RShift(), right=BoolOp(op=Or(), values=[Constant(value='F')])), op=BitOr(), right=BinOp(left=UnaryOp(op=Not(), operand=BoolOp(op=Or(), values=[Constant(value=True)])), op=Mod(), right=BinOp(left=BoolOp(op=And(), values=[Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[])]), op=LShift(), right=Constant(value=False))))
((set()) > None != set()) | ((set()))    # BinOp(left=Compare(left=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), ops=[Gt(), NotEq()], comparators=[Constant(value=None), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), op=BitOr(), right=Compare(left=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), ops=[], comparators=[Constant(value=True)]))
524 - 188                                # BinOp(left=Constant(value=524), op=Sub(), right=Constant(value=188))
6214 / 81                                # BinOp(left=Constant(value=6214), op=Div(), right=Constant(value=81))
26 / 43                                  # BinOp(left=Constant(value=26), op=Div(), right=Constant(value=43))
"s85;3Rw?ST!NI]_-eJ(x7'kG|z}C^&fWLnY[Z,rV*Qj.`Ed%:4<t" ^ '/$ao6 U{2cim@hHtF>b+vX)KBg1l=qyMDp~O0#A9uPa+l' # BinOp(left=Constant(value="s85;3Rw?ST!NI]_-eJ(x7'kG|z}C^&fWLnY[Z,rV*Qj.`Ed%:4<t"), op=BitXor(), right=Constant(value="/$ao6 U{2cim@hHtF>b+vX)KBg1l=qyMDp~O0#A9uPa+l"))

<UnaryOp>
+(set(), 
[])                            # UnaryOp(op=UAdd(), operand=Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Expr(value=List(elts=[], ctx=Del()))], ctx=Del()))
~(None)                                  # UnaryOp(op=Invert(), operand=BoolOp(op=Or(), values=[Constant(value=None)]))
-(((not {set(), set()})) & {set(): set(), (): set()}) # UnaryOp(op=USub(), operand=BinOp(left=Compare(left=UnaryOp(op=Not(), operand=Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])), ops=[], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), op=BitAnd(), right=Dict(keys=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Tuple(elts=[], ctx=Load())], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load())])))
-((set() + set()) % ((set() << set()) / (set() ^ set()))) ** (set() // set() >> set() * set()) # UnaryOp(op=USub(), operand=BinOp(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Mod(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Div(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))), op=Pow(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=FloorDiv(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=RShift(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))))
-True                                    # UnaryOp(op=USub(), operand=Constant(value=True))
+(4.3 @ (823 - '&') | '')                # UnaryOp(op=UAdd(), operand=BinOp(left=BinOp(left=Constant(value=4.30), op=MatMult(), right=BinOp(left=Constant(value=823), op=Sub(), right=Constant(value='&'))), op=BitOr(), right=Constant(value="")))
~(False <= 51 not in 959)                # UnaryOp(op=Invert(), operand=BoolOp(op=And(), values=[Compare(left=Constant(value=False), ops=[LtE(), NotIn()], comparators=[Constant(value=51), Constant(value=959)])]))
~17                                      # UnaryOp(op=Invert(), operand=Constant(value=17))
not 26                                   # UnaryOp(op=Not(), operand=Constant(value=26))
-68                                      # UnaryOp(op=USub(), operand=Constant(value=68))

<Compare>
()                                       # Compare(left=BoolOp(op=Or(), values=[]), ops=[], comparators=[Expr(value=Constant(value=8)), Tuple(elts=[List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del()), Dict(keys=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], values=[]), Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Tuple(elts=[], ctx=Del())]), UnaryOp(op=UAdd(), operand=BinOp(left=Compare(left=Tuple(elts=[], ctx=Load()), ops=[], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), op=Add(), right=List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load())))], ctx=Del())])
(set() & set()) / (set() @ (set() - set())) not in set() ^ set() # Compare(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Div(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Sub(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))), ops=[NotIn()], comparators=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
set() % set() // None << (set() - set() >> set() ** set()) <= set() | set() > set() + set() # Compare(left=BinOp(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=FloorDiv(), right=Constant(value=None)), op=LShift(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Sub(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=RShift(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))), ops=[LtE(), Gt()], comparators=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Constant(value=True)])
-632.86 != (not ~(not ('')))             # Compare(left=UnaryOp(op=USub(), operand=Constant(value=632.860)), ops=[NotEq()], comparators=[UnaryOp(op=Not(), operand=UnaryOp(op=Invert(), operand=UnaryOp(op=Not(), operand=BoolOp(op=And(), values=[Constant(value='')]))))])
'' >= 717 is not False                   # Compare(left=Constant(value=""), ops=[GtE(), IsNot(), Eq()], comparators=[Constant(value=717), Constant(value=False)])
15 < 39                                  # Compare(left=Constant(value=15), ops=[Lt(), Is(), Gt(), In()], comparators=[Constant(value=39)])
548 != 934688                            # Compare(left=Constant(value=548), ops=[NotEq(), LtE()], comparators=[Constant(value=934688)])
"w-xSGA8TI{%pRcq6e!_E:}P]9LM/&b1+7*lBDnvu)[o`3dY|Oj~JU<#Z'rH;g,f>@Q0tKk4N$iVaFhzW52y=(C.? sXm^{ " in 425 # Compare(left=Constant(value="w-xSGA8TI{%pRcq6e!_E:}P]9LM/&b1+7*lBDnvu)[o`3dY|Oj~JU<#Z'rH;g,f>@Q0tKk4N$iVaFhzW52y=(C.? sXm^{ "), ops=[In()], comparators=[Constant(value=425), Constant(value=21270)])
'H]3Ky.2p-:#6F%9V{X⁸)lMD[;7Otk/hgImvcJf& E`uG}w?PY:' >= 'nCALds|1zjq4BZ$"ab*@_(e<!rT=iUW~05+,Q>oNSxRVpF' # Compare(left=Constant(value='H]3Ky.2p-:#6F%9V{X⁸)lMD[;7Otk/hgImvcJf& E`uG}w?PY:'), ops=[GtE(), Gt(), NotIn(), IsNot(), GtE()], comparators=[Constant(value='nCALds|1zjq4BZ$"ab*@_(e<!rT=iUW~05+,Q>oNSxRVpF')])
6.3                                      # Compare(left=Constant(value=6.3), ops=[], comparators=[])

```

并非所有这些表达式都是*类型正确的*。例如，`set() * set()` 在运行时会引发类型错误。尽管如此，它们*可以被正确解析*。

到目前为止，我们的语法有多好？让我们创建 20 个表达式并检查其中有多少是*类型正确的*。

1.  解析时没有`SyntaxError`。

1.  评估时没有 `TypeError`。

```py
expr_iterations = 20
bad_syntax = 0
bad_type = 0
ast_exprs_grammar = convert_ebnf_grammar(PYTHON_AST_EXPRS_GRAMMAR)
expr_solver = ISLaSolver(ast_exprs_grammar, max_number_free_instantiations=expr_iterations)
for i in range(expr_iterations):
    expr_tree = eval(str(expr_solver.solve()))
    expr_tree = fix_missing_locations(expr_tree)
    expr_str = ast.unparse(expr_tree)
    print(i, expr_str)
    try:
        ...  # insert parsing code here
    except SyntaxError:
        bad_syntax += 1
    except TypeError:
        bad_type += 1

    try:
        ...  # <-- insert evaluation code here
    except TypeError:
        bad_type += 1
    except SyntaxError:
        bad_syntax += 1

print(f"Bad syntax: {bad_syntax}/{expr_iterations}")
print(f"Bad type: {bad_type}/{expr_iterations}") 
```

```py
0 set()
1 
2 [
~(False,) >> {635: (set() @ set() & set(),), 99.1 not in set() ** set(): {[set() ^ set()]}}]
3 (set() * set() - (set() + set())) / ((set() ^ set()) % set() | set() << set() % set())
4 not None
5 -+(True and '#' and 'x')
6 (8876 > 46 in 36 != 50)
7 24
8 "LfDW-kSM|tpB&+V*RgQ7U]3xq)zh~n^`wTdie5jvPN: A2K?$ZGJ(X;%@sr9mcIu!}OC/1><b=y'0H8o_.4lFYa{6[,>E?"
9 'o,awXihgeM[581Bln"RA60^k2N_L=d$C`7U~f)(&ZG]#m+DqF|PjpIQ<.4ur@ T!-W}Vs:Y{*zOEJb3StHK>?y%c/;iv9'
10 ((set()) < set() is set()) >= ((set()))
11 ((set())) == ((set()) <= set())
12 
13 
14 set()
15 set()
16 []
17 
18 ((() or 'k5'))
19 () | []
Bad syntax: 0/20
Bad type: 0/20

```

我们在这里做得还不错。在原则上，可以使用 ISLa 约束，使得生成的代码能够正确地类型化——但这可能需要数百到数千条规则。我们将把这个练习留给读者。

注意，一旦*标识符*出现，你不应该重复这个实验。有很小的可能性，fuzzer 会合成一个像`os.remove("/")`这样的调用——然后你的文件系统就消失了！</details> <details id="Excursion:-Names-and-Function-Calls"><summary>名称和函数调用</summary>

让我们添加一些*标识符*，这样我们就可以调用*函数*。

```py
ID_START = string.ascii_letters + '_'
ID_CONTINUE = ID_START + string.digits 
```

```py
ID_CONTINUE 
```

```py
'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_0123456789'

```

```py
print(ast.dump(ast.parse("xyzzy(a, b=c)"), indent=4)) 
```

```py
Module(
    body=[
        Expr(
            value=Call(
                func=Name(id='xyzzy', ctx=Load()),
                args=[
                    Name(id='a', ctx=Load())],
                keywords=[
                    keyword(
                        arg='b',
                        value=Name(id='c', ctx=Load()))]))],
    type_ignores=[])

```

```py
PYTHON_AST_IDS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_EXPRS_GRAMMAR, {
    '<expr>': PYTHON_AST_EXPRS_GRAMMAR['<expr>'] + [
        '<Name>', '<Call>'
    ],

    # Identifiers
    '<Name>': [
        'Name(id=<identifier>, ctx=Load())',
        'Name(id=<identifier>, ctx=Del())'
    ],
    '<identifier>': [ "'<id>'" ],
    '<id>': [ '<id_start><id_continue>*' ],
    '<id_start>': list(ID_START),
    '<id_continue>': list(ID_CONTINUE),
    # FIXME: Actual rules are a bit more complex; see
    # https://docs.python.org/3/reference/lexical_analysis.html#identifiers

    # Function Calls
    '<Call>': [ 'Call(func=<func><args_param><keywords_param>)' ],
    '<args_param>': [ ', args=<expr_list>' ],
    '<keywords_param>': [ ', keywords=<keyword_list>' ],
    '<func>': [ '<expr>' ],  # Actually <Expr>, but this is more readable and parses 90%
    '<keyword_list>': [ '[<keywords>?]' ],
    '<keywords>': [ '<keyword>', '<keyword>, <keywords>' ],
    '<keyword>': [ 'keyword(arg=<identifier>, value=<expr>)' ]
}) 
```

```py
# do import this unconditionally
if sys.version_info >= (3, 13):
    PYTHON_AST_IDS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_IDS_GRAMMAR, {
        # As of 3.13, args and keywords parameters are optional
        '<Call>': [ 'Call(func=<func><args_param>?<keywords_param>?)' ],
    }) 
```

```py
assert is_valid_grammar(PYTHON_AST_IDS_GRAMMAR) 
```

```py
for elt in [ '<Name>', '<Call>' ]:
    print(elt)
    test_samples(PYTHON_AST_IDS_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<Name>
n                                        # Name(id='n', ctx=Load())
vmGtKyT3Oq1gBC_srAIRaeQw6Dh8V5oLdj9FcvHfb4MpPZiNuEJ27WYU0lnkSxX9Lz # Name(id='vmGtKyT3Oq1gBC_srAIRaeQw6Dh8V5oLdj9FcvHfb4MpPZiNuEJ27WYU0lnkSxX9Lz', ctx=Del())
h                                        # Name(id='h', ctx=Load())
L                                        # Name(id='L', ctx=Del())
M                                        # Name(id='M', ctx=Load())
g                                        # Name(id='g', ctx=Del())
P                                        # Name(id='P', ctx=Del())
It                                       # Name(id='It', ctx=Del())
jGn7g                                    # Name(id='jGn7g', ctx=Load())
psj                                      # Name(id='psj', ctx=Del())

<Call>
{{set(): set()}(+set())}(m7K, (), u=[set() // set()]) # Call(func=Set(elts=[Call(func=Dict(keys=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), args=[UnaryOp(op=UAdd(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], keywords=[])]), args=[Name(id='m7K', ctx=Del()), Tuple(elts=[], ctx=Load())], keywords=[keyword(arg='u', value=List(elts=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=FloorDiv(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], ctx=Load()))])
(())(set(), None, 
U, j=False, i=set())  # Call(func=Compare(left=BoolOp(op=Or(), values=[]), ops=[], comparators=[BoolOp(op=And(), values=[])]), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Constant(value=None), Expr(value=Name(id='U', ctx=Load()))], keywords=[keyword(arg='j', value=Constant(value=False)), keyword(arg='i', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
set(), set(), set(), (set(),), T=set(), L=set(), y=set()) # Call(func=List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del()), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del())], keywords=[keyword(arg='T', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), keyword(arg='L', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), keyword(arg='y', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
(set() - set() ** set() % (set() @ set()))(set() * set(), set() << set(), W=set() / set()) # Call(func=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Sub(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Mod(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))), args=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], keywords=[keyword(arg='W', value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))])
(set() >> set())((set() | set()) ^ set(), g=set() & set(), B=set() + set()) # Call(func=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=RShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), args=[BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], keywords=[keyword(arg='g', value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), keyword(arg='B', value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))])
''(-(not 48.9), Q=~70, FmD=True, h=set()) # Call(func=Constant(value=''), args=[UnaryOp(op=USub(), operand=UnaryOp(op=Not(), operand=Constant(value=48.9)))], keywords=[keyword(arg='Q', value=UnaryOp(op=Invert(), operand=Constant(value=70))), keyword(arg='FmD', value=Constant(value=True)), keyword(arg='h', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
((set() in set()) > set())(None, v=set()) # Call(func=Compare(left=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[In()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), ops=[Gt()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), args=[Compare(left=Constant(value=None), ops=[], comparators=[])], keywords=[keyword(arg='v', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
''(set(), V, l, t, _, zM=H)              # Call(func=Constant(value=""), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Name(id='V', ctx=Load()), Name(id='l', ctx=Load()), Name(id='t', ctx=Del()), Name(id='_', ctx=Load())], keywords=[keyword(arg='zM', value=Name(id='H', ctx=Load()))])
xTzqJe5gQ(n80d, qkw=b)                   # Call(func=Name(id='xTzqJe5gQ', ctx=Del()), args=[Name(id='n80d', ctx=Load())], keywords=[keyword(arg='qkw', value=Name(id='b', ctx=Del()))])
k(set(), set(), set(), E, o=c)           # Call(func=Name(id='k', ctx=Load()), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Name(id='E', ctx=Load())], keywords=[keyword(arg='o', value=Name(id='c', ctx=Load()))])

```

```py
ast_ids_grammar = convert_ebnf_grammar(PYTHON_AST_IDS_GRAMMAR) 
```

```py
id_solver = ISLaSolver(ast_ids_grammar, start_symbol='<id>')
assert id_solver.check('open') 
```

```py
name_solver = ISLaSolver(ast_ids_grammar)
assert name_solver.check("Name(id='open', ctx=Load())") 
```

```py
call_solver = ISLaSolver(ast_ids_grammar, start_symbol='<keyword_list>')
assert call_solver.check('[]') 
```

```py
call_str = ast.dump(ast.parse('open("foo.txt", "r")').body[0].value)
print(call_str)
call_solver = ISLaSolver(ast_ids_grammar)
assert call_solver.check(call_str) 
```

```py
Call(func=Name(id='open', ctx=Load()), args=[Constant(value='foo.txt'), Constant(value='r')], keywords=[])

```</details> <details id="Excursion:-Attributes-and-Subscripts"><summary>属性和下标</summary>

让我们添加属性和下标。

```py
print(ast.dump(ast.parse("a[b].c"), indent=4)) 
```

```py
Module(
    body=[
        Expr(
            value=Attribute(
                value=Subscript(
                    value=Name(id='a', ctx=Load()),
                    slice=Name(id='b', ctx=Load()),
                    ctx=Load()),
                attr='c',
                ctx=Load()))],
    type_ignores=[])

```

```py
PYTHON_AST_ATTRS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_IDS_GRAMMAR, {
    '<expr>': PYTHON_AST_IDS_GRAMMAR['<expr>'] + [
        '<Attribute>', '<Subscript>', '<Starred>',
    ],

    # Attributes
    '<Attribute>': [
        'Attribute(value=<expr>, attr=<identifier>, ctx=Load())',
        'Attribute(value=<expr>, attr=<identifier>, ctx=Del())',
    ],

    # Subscripts
    '<Subscript>': [
        'Subscript(value=<expr>, slice=<Slice>, ctx=Load())',
        'Subscript(value=<expr>, slice=<Slice>, ctx=Del())',
    ],
    '<Slice>': [
        'Slice()',
        'Slice(<expr>)',
        'Slice(<expr>, <expr>)',
        'Slice(<expr>, <expr>, <expr>)',
    ],

    # Starred
    '<Starred>': [
        'Starred(value=<expr>, ctx=Load())',
        'Starred(value=<expr>, ctx=Del())',
    ],

    # We're extending the set of callers a bit
    '<func>': [ '<Name>', '<Attribute>', '<Subscript>' ],
}) 
```

```py
assert is_valid_grammar(PYTHON_AST_ATTRS_GRAMMAR) 
```

```py
for elt in [ '<Attribute>', '<Subscript>', '<Starred>' ]:
    print(elt)
    test_samples(PYTHON_AST_ATTRS_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<Attribute>
{}.zZ                                    # Attribute(value=Dict(keys=[BoolOp(op=Or(), values=[Expr(value=UnaryOp(op=UAdd(), operand=Call(func=Name(id='e', ctx=Del()), args=[], keywords=[]))), BinOp(left=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), op=Sub(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))]), Starred(value=Attribute(value=Tuple(elts=[Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del())])], ctx=Load()), attr='HV', ctx=Load()), ctx=Del())], values=[]), attr='zZ', ctx=Del())
OON6Q9X8m1yqSkYJtGPI_bADfjMTaIhp._Rr5dHs2n7UwzFoLulcei3KCgW4EvxB60jmPP # Attribute(value=Name(id='OON6Q9X8m1yqSkYJtGPI_bADfjMTaIhp', ctx=Load()), attr='_Rr5dHs2n7UwzFoLulcei3KCgW4EvxB60jmPP', ctx=Del())
175 .M                                   # Attribute(value=Constant(value=175), attr='M', ctx=Del())
*[set() * set() + set() / set()][(set() << set(), set() % set(), set() ** (set() & set())):].Wn # Attribute(value=Starred(value=Subscript(value=List(elts=[BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Add(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))], ctx=Load()), slice=Slice(Tuple(elts=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))], ctx=Del())), ctx=Load()), ctx=Load()), attr='Wn', ctx=Del())
((-+set()[:]()[set():set():set()] | (not ~set().E())) @ '' // (None ^ False)).B # Attribute(value=BinOp(left=BinOp(left=BinOp(left=UnaryOp(op=USub(), operand=UnaryOp(op=UAdd(), operand=Subscript(value=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), args=[], keywords=[]), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Del()))), op=BitOr(), right=UnaryOp(op=Not(), operand=UnaryOp(op=Invert(), operand=Call(func=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='E', ctx=Del()), args=[], keywords=[])))), op=MatMult(), right=Constant(value='')), op=FloorDiv(), right=BinOp(left=Constant(value=None), op=BitXor(), right=Constant(value=False))), attr='B', ctx=Load())
((99.8) >> (True)['HAVsYE|,]@bXz!hguQimRwL0)2=W-8PteTK<{c~*3}f$OandqF1%&4IJ"MjZ>^k`pv;/U_?B.7[+y#(G 9S5CDoNrlx:6Z':'S']).yM # Attribute(value=BinOp(left=BoolOp(op=And(), values=[Constant(value=99.8)]), op=RShift(), right=Subscript(value=BoolOp(op=And(), values=[Constant(value=True)]), slice=Slice(Constant(value='HAVsYE|,]@bXz!hguQimRwL0)2=W-8PteTK<{c~*3}f$OandqF1%&4IJ"MjZ>^k`pv;/U_?B.7[+y#(G 9S5CDoNrlx:6Z'), Constant(value="S")), ctx=Del())), attr='yM', ctx=Del())
(((set()) < set() == set()) not in set()).l # Attribute(value=Compare(left=Compare(left=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), ops=[Lt(), Eq()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), ops=[NotIn()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), Name(id='c', ctx=Del())]), attr='l', ctx=Del())
vlL.CZ                                   # Attribute(value=Name(id='vlL', ctx=Load()), attr='CZ', ctx=Del())
w.nyuCk                                  # Attribute(value=Name(id='w', ctx=Del()), attr='nyuCk', ctx=Load())
Js.Za                                    # Attribute(value=Name(id='Js', ctx=Load()), attr='Za', ctx=Load())

<Subscript>
{279.0 >> [], -*set()[:]:, set(), ())}[{}:] # Subscript(value=Set(elts=[BinOp(left=Constant(value=279.0), op=RShift(), right=List(elts=[BoolOp(op=And(), values=[])], ctx=Del())), UnaryOp(op=USub(), operand=Call(func=Subscript(value=Subscript(value=Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Load()), slice=Slice(), ctx=Del()), slice=Slice(), ctx=Load()), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Tuple(elts=[], ctx=Load())], keywords=[]))]), slice=Slice(Dict(keys=[], values=[Name(id='U', ctx=Load())])), ctx=Del())
(set()).y[():b:
set()]                   # Subscript(value=Attribute(value=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), attr='y', ctx=Load()), slice=Slice(Tuple(elts=[], ctx=Del()), Name(id='b', ctx=Del()), Expr(value=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]))), ctx=Del())
(set() << set() - set()).c[[set() @ set() // set()]:*(set() & set()).z] # Subscript(value=Attribute(value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Sub(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), attr='c', ctx=Load()), slice=Slice(List(elts=[BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=FloorDiv(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], ctx=Load()), Starred(value=Attribute(value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), attr='z', ctx=Del()), ctx=Del())), ctx=Load())
((set() | set()) ^ set() ** set())[set() * set():(set() + set()) / set()] # Subscript(value=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=BitXor(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), slice=Slice(BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), ctx=Load())
None['t':]                               # Subscript(value=Constant(value=None), slice=Slice(Constant(value="t")), ctx=Load())
(not set().H(~set(), N=set()))[M():n():set()] # Subscript(value=UnaryOp(op=Not(), operand=Call(func=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='H', ctx=Del()), args=[UnaryOp(op=Invert(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], keywords=[keyword(arg='N', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])), slice=Slice(Call(func=Name(id='M', ctx=Load()), args=[], keywords=[]), Call(func=Name(id='n', ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Del())
False[1632:]                             # Subscript(value=Constant(value=False), slice=Slice(Constant(value=1632)), ctx=Del())
(('') % +(94 or True))[True or ((t)) is set() <= Q:] # Subscript(value=BinOp(left=BoolOp(op=Or(), values=[Constant(value='')]), op=Mod(), right=UnaryOp(op=UAdd(), operand=BoolOp(op=Or(), values=[Constant(value=94), Constant(value=True)]))), slice=Slice(BoolOp(op=Or(), values=[Constant(value=True), Compare(left=Compare(left=Compare(left=Name(id='t', ctx=Load()), ops=[], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), ops=[], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), ops=[Is(), LtE()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Name(id='Q', ctx=Load()), Name(id='r', ctx=Del())])])), ctx=Load())
l7oUAETh5yCvxmRcgJ8[vtk3XeH:midn6Wa4]    # Subscript(value=Name(id='l7oUAETh5yCvxmRcgJ8', ctx=Load()), slice=Slice(Name(id='vtk3XeH', ctx=Load()), Name(id='midn6Wa4', ctx=Load())), ctx=Load())
JN0GQSzfYw1MLI2up6[gD9VZbsK_lqjrPOFB:]   # Subscript(value=Name(id='JN0GQSzfYw1MLI2up6', ctx=Load()), slice=Slice(Name(id='gD9VZbsK_lqjrPOFB', ctx=Del())), ctx=Load())

<Starred>
*[]                                      # Starred(value=List(elts=[], ctx=Del()), ctx=Load())
*(
{{set().j(K.J, Q=set()): (+*(set())[set():set():set()],)}, 440.7}) >> i # Starred(value=BinOp(left=BoolOp(op=And(), values=[Expr(value=Set(elts=[Dict(keys=[Call(func=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='j', ctx=Del()), args=[Attribute(value=Name(id='K', ctx=Del()), attr='J', ctx=Load())], keywords=[keyword(arg='Q', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])], values=[Tuple(elts=[UnaryOp(op=UAdd(), operand=Starred(value=Subscript(value=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Load()), ctx=Load()))], ctx=Del())]), Constant(value=440.7)]))]), op=RShift(), right=Name(id='i', ctx=Load())), ctx=Del())
*[set(), set(), set() @ set()][(set(), set() - set(), set() % set() / set() ** set()):] # Starred(value=Subscript(value=List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], ctx=Load()), slice=Slice(Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Sub(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Div(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))], ctx=Load())), ctx=Del()), ctx=Load())
*(set() ^ set()) & (set() << set()) + set() | set() * set() # Starred(value=BinOp(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=BitAnd(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=BitOr(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), ctx=Del())
*'g'                                     # Starred(value=Constant(value='g'), ctx=Load())
*-None                                   # Starred(value=UnaryOp(op=USub(), operand=Constant(value=None)), ctx=Del())
*9523:, -set(), not ~set(), (not not set())[-(set() // set()):]) # Starred(value=Call(func=Subscript(value=Constant(value=9523), slice=Slice(), ctx=Del()), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), UnaryOp(op=USub(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), UnaryOp(op=Not(), operand=UnaryOp(op=Invert(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), Subscript(value=UnaryOp(op=Not(), operand=UnaryOp(op=Not(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), slice=Slice(UnaryOp(op=USub(), operand=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=FloorDiv(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), BoolOp(op=Or(), values=[])), ctx=Load())], keywords=[]), ctx=Del())
*False                                   # Starred(value=Constant(value=False), ctx=Load())
*X(Y(q=set()), I=U(), D=set())           # Starred(value=Call(func=Name(id='X', ctx=Load()), args=[Call(func=Name(id='Y', ctx=Load()), args=[], keywords=[keyword(arg='q', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])], keywords=[keyword(arg='I', value=Call(func=Name(id='U', ctx=Load()), args=[], keywords=[])), keyword(arg='D', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))]), ctx=Del())
*'#'                                     # Starred(value=Constant(value="#"), ctx=Del())

```</details> <details id="Excursion:-Variable-Assignments"><summary>变量赋值</summary>

现在是变量赋值的时候了。这些使事情变得更加复杂，因为我们有一个限制性的表达式集合在赋值的左侧。

```py
PYTHON_AST_ASSIGNMENTS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_ATTRS_GRAMMAR, {
    '<start>': [ '<stmt>' ],

    '<stmt>': [
        '<Assign>', '<AugAssign>',
        '<Expr>'
    ],

    # Assignments
    '<Assign>': [
        'Assign(targets=<nonempty_lhs_expr_list>, value=<expr><type_comment>?)',
    ],
    '<type_comment>': [ ', type_comment=<string>' ],
    '<AugAssign>': [
        'AugAssign(target=<lhs_expr>, op=<operator>, value=<expr>)',
    ],

    # Lists of left-hand side expressions
    # '<lhs_expr_list>': [ '[<lhs_exprs>?]' ],
    '<nonempty_lhs_expr_list>': [ '[<lhs_exprs>]' ],
    '<lhs_exprs>': [ '<lhs_expr>', '<lhs_exprs>, <lhs_expr>' ],

    # On the left-hand side of assignments, we allow a number of structures
    '<lhs_expr>': [
        '<lhs_Name>',  # Most common
        '<lhs_List>', '<lhs_Tuple>',
        '<lhs_Attribute>',
        '<lhs_Subscript>',
        '<lhs_Starred>',
    ],

    '<lhs_Name>': [ 'Name(id=<identifier>, ctx=Store())', ],

    '<lhs_List>': [
        'List(elts=<nonempty_lhs_expr_list>, ctx=Store())',
    ],
    '<lhs_Tuple>': [
        'Tuple(elts=<nonempty_lhs_expr_list>, ctx=Store())',
    ],
    '<lhs_Attribute>': [
        'Attribute(value=<lhs_expr>, attr=<identifier>, ctx=Store())',
    ],
    '<lhs_Subscript>': [
        'Subscript(value=<lhs_expr>, slice=<Slice>, ctx=Store())',
    ],
    '<lhs_Starred>': [
        'Starred(value=<lhs_expr>, ctx=Store())',
    ],
}) 
```

```py
assert is_valid_grammar(PYTHON_AST_ASSIGNMENTS_GRAMMAR) 
```

```py
for elt in ['<Assign>', '<AugAssign>']:
    print(elt)
    test_samples(PYTHON_AST_ASSIGNMENTS_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<Assign>
*[(r,), (V, C[set():set()]), Z[set():].WDY3i] = () # type: * # Assign(targets=[Starred(value=List(elts=[Tuple(elts=[Name(id='r', ctx=Store())], ctx=Store()), Tuple(elts=[Name(id='V', ctx=Store()), Subscript(value=Name(id='C', ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store())], ctx=Store()), Attribute(value=Subscript(value=Name(id='Z', ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), attr='WDY3i', ctx=Store())], ctx=Store()), ctx=Store())], value=Tuple(elts=[], ctx=Load()), type_comment='*')
h[set():set():set()][set():*set():set()[:]()][:] = [set()].Yzt # Assign(targets=[Subscript(value=Subscript(value=Subscript(value=Name(id='h', ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Load()), Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load()), args=[], keywords=[])), ctx=Store()), slice=Slice(), ctx=Store())], value=Attribute(value=List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del()), attr='Yzt', ctx=Load()))
N[:][:][set():set():set()][{}:][:] = 
ExcXjv1h # type: R # Assign(targets=[Subscript(value=Subscript(value=Subscript(value=Subscript(value=Subscript(value=Name(id='N', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), slice=Slice(Dict(keys=[], values=[])), ctx=Store()), slice=Slice(BoolOp(op=Or(), values=[])), ctx=Store())], value=Expr(value=Name(id='ExcXjv1h', ctx=Del())), type_comment="R")
H[:][:][set():set():set()] = -set() # type: y{ # Assign(targets=[Subscript(value=Subscript(value=Subscript(value=Name(id='H', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store())], value=UnaryOp(op=USub(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), type_comment='y{')
K[:][:] = a[:][set():set()] = False # type: USsF # Assign(targets=[Subscript(value=Subscript(value=Name(id='K', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(), ctx=Store()), Subscript(value=Subscript(value=Name(id='a', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store())], value=Constant(value=False), type_comment="USsF")
B[set():set()] = set()[:] << (set()[:]) # type: K # Assign(targets=[Subscript(value=Name(id='B', ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store())], value=BinOp(left=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), op=LShift(), right=Compare(left=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), ops=[], comparators=[])), type_comment='K')
sKC = fm = (*set().y, *{set()}) # type: L^}3QF # Assign(targets=[Name(id='sKC', ctx=Store()), Name(id='fm', ctx=Store())], value=Tuple(elts=[Starred(value=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='y', ctx=Del()), ctx=Del()), Starred(value=Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), ctx=Load())], ctx=Del()), type_comment='L^}3QF')
S = n = I = [set(), set(), F] # type: 8-h # Assign(targets=[Name(id='S', ctx=Store()), Name(id='n', ctx=Store()), Name(id='I', ctx=Store())], value=List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Name(id='F', ctx=Load())], ctx=Load()), type_comment="8-h")
gy = set() % set() @ set() - (set() & set()) # type: .~ # Assign(targets=[Name(id='gy', ctx=Store())], value=BinOp(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Sub(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), type_comment=".~")
cnoOWRu = set() * (set() >> set() ^ set() + set()) # type: ['Ox# # Assign(targets=[Name(id='cnoOWRu', ctx=Store())], value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=RShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=BitXor(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))), type_comment="['Ox#")

<AugAssign>
K <<= set()                              # AugAssign(target=Name(id='K', ctx=Store()), op=LShift(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))
(_, [A]).H ^= (*{}.a[set() | set():set():], False) # AugAssign(target=Attribute(value=Tuple(elts=[Name(id='_', ctx=Store()), List(elts=[Name(id='A', ctx=Store())], ctx=Store())], ctx=Store()), attr='H', ctx=Store()), op=BitXor(), value=Tuple(elts=[Subscript(value=Attribute(value=Starred(value=Dict(keys=[], values=[]), ctx=Del()), attr='a', ctx=Del()), slice=Slice(BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BoolOp(op=Or(), values=[])), ctx=Load()), Constant(value=False)], ctx=Load()))
*i[:][:][y():set()] -= [~(
set())]       # AugAssign(target=Subscript(value=Starred(value=Subscript(value=Subscript(value=Name(id='i', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(), ctx=Store()), ctx=Store()), slice=Slice(Call(func=Name(id='y', ctx=Del()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), op=Sub(), value=List(elts=[UnaryOp(op=Invert(), operand=Compare(left=Expr(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ops=[], comparators=[]))], ctx=Load()))
t3lmH[(set(), set()):] //= oxNerA8       # AugAssign(target=Subscript(value=Name(id='t3lmH', ctx=Store()), slice=Slice(Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del())), ctx=Store()), op=FloorDiv(), value=Name(id='oxNerA8', ctx=Load()))
pdnk2WaQFLs @= {*[set()].Qc[set().x:]}   # AugAssign(target=Name(id='pdnk2WaQFLs', ctx=Store()), op=MatMult(), value=Set(elts=[Subscript(value=Starred(value=Attribute(value=List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del()), attr='Qc', ctx=Load()), ctx=Load()), slice=Slice(Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='x', ctx=Load())), ctx=Del())]))
YMy **= (set() + (set() & set())) / (None % (set() >> set())) # AugAssign(target=Name(id='YMy', ctx=Store()), op=Pow(), value=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=Div(), right=BinOp(left=Constant(value=None), op=Mod(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=RShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))))
rXvE0_VP7puUYSIJwg4qDZt9z6RjiChKGTofbBO15 *= +'h' # AugAssign(target=Name(id='rXvE0_VP7puUYSIJwg4qDZt9z6RjiChKGTofbBO15', ctx=Store()), op=Mult(), value=UnaryOp(op=UAdd(), operand=Constant(value="h")))
PFUN += not Trueset(): # AugAssign(target=Name(id='PFUN', ctx=Store()), op=Add(), value=UnaryOp(op=Not(), operand=Call(func=Subscript(value=Constant(value=True), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Del()), args=[Constant(value=991.2)], keywords=[keyword(arg='J', value=Constant(value=None)), keyword(arg='k', value=Constant(value=False))])))
g ^= (-set()).m(set(), , u=-set(), h=set()) # AugAssign(target=Name(id='g', ctx=Store()), op=BitXor(), value=Call(func=Attribute(value=UnaryOp(op=USub(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), attr='m', ctx=Load()), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BoolOp(op=And(), values=[])], keywords=[keyword(arg='u', value=UnaryOp(op=USub(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), keyword(arg='h', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))]))
Ce |= 448                                # AugAssign(target=Name(id='Ce', ctx=Store()), op=BitOr(), value=Constant(value=448))

```</details> <details id="Excursion:-Statements"><summary>语句</summary>

现在是语句。这里有很多。

```py
PYTHON_AST_STMTS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_ASSIGNMENTS_GRAMMAR, {
    '<start>': [ '<stmt>' ],

    '<stmt>': PYTHON_AST_ASSIGNMENTS_GRAMMAR['<stmt>'] + [
        '<For>', '<While>', '<If>',
        '<Return>', '<Delete>', '<Assert>',
        '<Pass>', '<Break>', '<Continue>',
        '<With>'
    ],

    # Control structures
    '<For>': [
        'For(target=<lhs_expr>, iter=<expr>, body=<nonempty_stmt_list>, orelse=<stmt_list><type_comment>)'
    ],
    '<stmt_list>': [ '[<stmts>?]' ],
    '<nonempty_stmt_list>': [ '[<stmts>]' ],
    '<stmts>': [ '<stmt>', '<stmt>, <stmts>' ],

    '<While>': [
        'While(test=<expr>, body=<nonempty_stmt_list>, orelse=<stmt_list>)'
    ],

    '<If>': [
        'If(test=<expr>, body=<nonempty_stmt_list><orelse_param>)'
    ],
    '<orelse_param>': [
        ', orelse=<stmt_list>'
    ],

    '<With>': [
        'With(items=<withitem_list>, body=<nonempty_stmt_list><type_comment>?)'
    ],
    '<withitem_list>': [ '[<withitems>?]' ],
    '<withitems>': [ '<withitem>', '<withitems>, <withitem>' ],
    '<withitem>': [
        'withitem(context_expr=<expr>)',
        'withitem(context_expr=<expr>, optional_vars=<lhs_expr>)',
    ],

    # Other statements
    '<Return>': [
        'Return()',
        'Return(value=<expr>)'
    ],
    '<Delete>': [
        'Delete(targets=<expr_list>)'
    ],
    '<Assert>': [
        'Assert(test=<expr>)',
        'Assert(test=<expr>, msg=<expr>)'
    ],
    '<Pass>': [ 'Pass()'],
    '<Break>': [ 'Break()' ],
    '<Continue>': [ 'Continue()']

    # FIXME: A few more: AsyncFor, AsyncWith, Match, Try, TryStar
    # Import, ImportFrom, Global, Nonlocal...
}) 
```

```py
# do import this unconditionally
if sys.version_info >= (3, 13):
    PYTHON_AST_STMTS_GRAMMAR: Grammar = \
        extend_grammar(PYTHON_AST_STMTS_GRAMMAR, {
        # As of 3.13, orelse is optional
        '<If>': [
            'If(test=<expr>, body=<nonempty_stmt_list><orelse_param>?)'
        ],
    }) 
```

```py
assert is_valid_grammar(PYTHON_AST_STMTS_GRAMMAR) 
```

```py
for elt in PYTHON_AST_STMTS_GRAMMAR['<stmt>']:
    print(elt)
    test_samples(PYTHON_AST_STMTS_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<Assign>
*[v[:][:][:]][{}:+*set()[:]()] = (XDBoW_Av,).L4 = (32.6,) # type:  # Assign(targets=[Starred(value=Subscript(value=List(elts=[Subscript(value=Subscript(value=Subscript(value=Name(id='v', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(), ctx=Store())], ctx=Store()), slice=Slice(Dict(keys=[], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), UnaryOp(op=UAdd(), operand=Starred(value=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load()), args=[], keywords=[]), ctx=Load()))), ctx=Store()), ctx=Store()), Attribute(value=Tuple(elts=[Name(id='XDBoW_Av', ctx=Store())], ctx=Store()), attr='L4', ctx=Store())], value=Tuple(elts=[Constant(value=32.6)], ctx=Load()), type_comment="")
g[:][set():][[]::set()] = set()[:]       # Assign(targets=[Subscript(value=Subscript(value=Subscript(value=Name(id='g', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), slice=Slice(List(elts=[], ctx=Del()), BoolOp(op=And(), values=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store())], value=Compare(left=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), ops=[], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]))
W[:] = y[:][set():set():set()] = 
K18E # type: N # Assign(targets=[Subscript(value=Name(id='W', ctx=Store()), slice=Slice(), ctx=Store()), Subscript(value=Subscript(value=Name(id='y', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store())], value=Expr(value=Name(id='K18E', ctx=Load())), type_comment='N')
V = _ = (set() | set()).E # type: i0     # Assign(targets=[Name(id='V', ctx=Store()), Name(id='_', ctx=Store())], value=Attribute(value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), attr='E', ctx=Del()), type_comment="i0")
cZIujm3gC = eMePLrNVy9z2 # type: Wd~OC6+v02ey # Assign(targets=[Name(id='cZIujm3gC', ctx=Store())], value=Name(id='eMePLrNVy9z2', ctx=Del()), type_comment='Wd~OC6+v02ey')
Yf0lcOSaT = *[{set()}.b, (set(), set())] # type: *H<u&~|  # Assign(targets=[Name(id='Yf0lcOSaT', ctx=Store())], value=Starred(value=List(elts=[Attribute(value=Set(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), attr='b', ctx=Load()), Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del())], ctx=Load()), ctx=Del()), type_comment="*H<u&~| ")
N = i = ((set() ^ set()) & set()) * (set() + set()) # type: +ps # Assign(targets=[Name(id='N', ctx=Store()), Name(id='i', ctx=Store())], value=BinOp(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Mult(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), type_comment="+ps")
m = P = set() @ set() << set() // set() # type: ]J # Assign(targets=[Name(id='m', ctx=Store()), Name(id='P', ctx=Store())], value=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=LShift(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=FloorDiv(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), type_comment=']J')
j = O = 18 % (set() / set()) # type: R?$6 # Assign(targets=[Name(id='j', ctx=Store()), Name(id='O', ctx=Store())], value=BinOp(left=Constant(value=18), op=Mod(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), type_comment="R?$6")
TGFJsqKdXkwb65xYnpHRh7UtQi4u = False # type: q(Q>GHPBsa!|bUV9&$w`Su.8-hAi3}7)#=LDx@5"?Kgjkz,pt_r%XT1m/f{c*;^ZlIE: YRnoM4[F< # Assign(targets=[Name(id='TGFJsqKdXkwb65xYnpHRh7UtQi4u', ctx=Store())], value=Constant(value=False), type_comment='q(Q>GHPBsa!|bUV9&$w`Su.8-hAi3}7)#=LDx@5"?Kgjkz,pt_r%XT1m/f{c*;^ZlIE: YRnoM4[F<')

<AugAssign>
(*krT_.qL2x,)[~[
set(), None, {}[:]].L:] //= (,) # AugAssign(target=Subscript(value=Tuple(elts=[Attribute(value=Starred(value=Name(id='krT_', ctx=Store()), ctx=Store()), attr='qL2x', ctx=Store())], ctx=Store()), slice=Slice(UnaryOp(op=Invert(), operand=Attribute(value=List(elts=[Expr(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Constant(value=None), Subscript(value=Dict(keys=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], values=[]), slice=Slice(), ctx=Del())], ctx=Load()), attr='L', ctx=Load()))), ctx=Store()), op=FloorDiv(), value=Tuple(elts=[BoolOp(op=And(), values=[])], ctx=Del()))
[h[:], F[:], l[:][set():Z]] -= U(*set() | set()) # AugAssign(target=List(elts=[Subscript(value=Name(id='h', ctx=Store()), slice=Slice(), ctx=Store()), Subscript(value=Name(id='F', ctx=Store()), slice=Slice(), ctx=Store()), Subscript(value=Subscript(value=Name(id='l', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), Name(id='Z', ctx=Load())), ctx=Store())], ctx=Store()), op=Sub(), value=Call(func=Name(id='U', ctx=Del()), args=[BinOp(left=Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Del()), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], keywords=[]))
Q[[]:set()][[]:set()[:]:set()[:]] &= *(set(),).b # AugAssign(target=Subscript(value=Subscript(value=Name(id='Q', ctx=Store()), slice=Slice(List(elts=[], ctx=Del()), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), slice=Slice(List(elts=[], ctx=Del()), Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load()), Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load())), ctx=Store()), op=BitAnd(), value=Starred(value=Attribute(value=Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load()), attr='b', ctx=Del()), ctx=Load()))
wnBzQMG <<= {set() @ set() ^ set() ** set() / set()} # AugAssign(target=Name(id='wnBzQMG', ctx=Store()), op=LShift(), value=Set(elts=[BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=BitXor(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))]))
Psdhpk1YVICRcN0J4wDPjqZmE856iFUbKf9oASWlXgvtyH7Oeua3Lyt6 *= 48.5 # AugAssign(target=Name(id='Psdhpk1YVICRcN0J4wDPjqZmE856iFUbKf9oASWlXgvtyH7Oeua3Lyt6', ctx=Store()), op=Mult(), value=Constant(value=48.5))
a %= ''                                  # AugAssign(target=Name(id='a', ctx=Store()), op=Mod(), value=Constant(value=""))
J += -True                               # AugAssign(target=Name(id='J', ctx=Store()), op=Add(), value=UnaryOp(op=USub(), operand=Constant(value=True)))
oU >>= set()set():set():set(), H=set()) # AugAssign(target=Name(id='oU', ctx=Store()), op=RShift(), value=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Del()), args=[], keywords=[keyword(arg='E', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), keyword(arg='H', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))]))
N //= not +(not 7927416330)              # AugAssign(target=Name(id='N', ctx=Store()), op=FloorDiv(), value=UnaryOp(op=Not(), operand=UnaryOp(op=UAdd(), operand=UnaryOp(op=Not(), operand=Constant(value=7927416330)))))
s += '' or False or 8888 .W((set()), v=set(), Y=set()) # AugAssign(target=Name(id='s', ctx=Store()), op=Add(), value=BoolOp(op=Or(), values=[Constant(value=''), Constant(value=False), Call(func=Attribute(value=Constant(value=8888), attr='W', ctx=Load()), args=[Compare(left=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), ops=[], comparators=[])], keywords=[keyword(arg='v', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), keyword(arg='Y', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])]))

<Expr>

*{(None, n, [] << +R()[{}.r::set().I])} # Expr(value=Expr(value=Starred(value=Set(elts=[Tuple(elts=[Constant(value=None), Name(id='n', ctx=Load()), BinOp(left=List(elts=[], ctx=Del()), op=LShift(), right=UnaryOp(op=UAdd(), operand=Subscript(value=Call(func=Name(id='R', ctx=Del()), args=[], keywords=[]), slice=Slice(Attribute(value=Dict(keys=[], values=[]), attr='r', ctx=Del()), BoolOp(op=Or(), values=[]), Compare(left=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='I', ctx=Load()), ops=[], comparators=[])), ctx=Del())))], ctx=Load())]), ctx=Del())))
(*((set() ^ set()) - (set() | set())) / (set() % set()), [set(), set() >> set(), set() // set(), set() + set()])[:] # Expr(value=Subscript(value=Tuple(elts=[Starred(value=BinOp(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Sub(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=Div(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), ctx=Load()), List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=RShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=FloorDiv(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], ctx=Load())], ctx=Del()), slice=Slice(), ctx=Load()))
851648.62 * True & 0                     # Expr(value=BinOp(left=BinOp(left=Constant(value=851648.62), op=Mult(), right=Constant(value=True)), op=BitAnd(), right=Constant(value=0)))
not -((set() @ set()) ** set()[:]())[set().z(_=set()):] # Expr(value=UnaryOp(op=Not(), operand=UnaryOp(op=USub(), operand=Subscript(value=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Pow(), right=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load()), args=[], keywords=[])), slice=Slice(Call(func=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='z', ctx=Del()), args=[], keywords=[keyword(arg='_', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])), ctx=Load()))))
'N'                                      # Expr(value=BoolOp(op=And(), values=[Constant(value="N")]))
(not ~'f')[~False:+(17 == ZAEPSYo_lKJHf6my8xTR2wg9b3d71qBeC5Mj6)] # Expr(value=Subscript(value=UnaryOp(op=Not(), operand=UnaryOp(op=Invert(), operand=Constant(value='f'))), slice=Slice(UnaryOp(op=Invert(), operand=Constant(value=False)), UnaryOp(op=UAdd(), operand=Compare(left=Constant(value=17), ops=[Eq()], comparators=[Name(id='ZAEPSYo_lKJHf6my8xTR2wg9b3d71qBeC5Mj6', ctx=Load()), Name(id='FcVkWZ0hQsONnpzGLrXut4vFIDiUBa', ctx=Load())]))), ctx=Del()))
i                                        # Expr(value=Name(id='i', ctx=Load()))
o2                                       # Expr(value=Name(id='o2', ctx=Del()))
AR                                       # Expr(value=Name(id='AR', ctx=Load()))
Y                                        # Expr(value=Name(id='Y', ctx=Del()))

<For>
for U, [D, I] in []: # type: j
    set()
    m /= set() # For(target=Tuple(elts=[Name(id='U', ctx=Store()), List(elts=[Name(id='D', ctx=Store()), Name(id='I', ctx=Store())], ctx=Store())], ctx=Store()), iter=Compare(left=List(elts=[], ctx=Load()), ops=[Eq()], comparators=[]), body=[Expr(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), AugAssign(target=Name(id='m', ctx=Store()), op=Div(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], orelse=[], type_comment="j")
for *O.s in {}: # type: }
    with :
        break
    assert set()
else:
    pass
    return # For(target=Starred(value=Attribute(value=Name(id='O', ctx=Store()), attr='s', ctx=Store()), ctx=Store()), iter=Dict(keys=[], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), body=[With(items=[], body=[Break()]), Assert(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], orelse=[Pass(), Return()], type_comment='}')
for q[:][set():set():set()] in *set(): # type: 
    return
    return
else:
    continue
    continue # For(target=Subscript(value=Subscript(value=Name(id='q', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), iter=Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Load()), body=[Return(), Return()], orelse=[Continue(), Continue()], type_comment="")
for g[:][set():] in 
set(): # type: 
    return
else:
    l = set()
    return # For(target=Subscript(value=Subscript(value=Name(id='g', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), iter=Expr(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), body=[Return()], orelse=[Assign(targets=[Name(id='l', ctx=Store())], value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return()], type_comment="")
for v in set().F(): # type: 
    if set():
        return
    return # For(target=Name(id='v', ctx=Store()), iter=Call(func=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='F', ctx=Load()), args=[], keywords=[]), body=[If(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[]), Return()], orelse=[], type_comment="")
for Z[:] in +set(): # type: 
    del 
    return
else:
    while set():
        return # For(target=Subscript(value=Name(id='Z', ctx=Store()), slice=Slice(), ctx=Store()), iter=UnaryOp(op=UAdd(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), body=[Delete(targets=[]), Return()], orelse=[While(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[])], type_comment="")
for z[set():set()] in (): # type: L
    for o in set(): # type: 
        return
else:
    return
    return # For(target=Subscript(value=Name(id='z', ctx=Store()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Store()), iter=Tuple(elts=[], ctx=Load()), body=[For(target=Name(id='o', ctx=Store()), iter=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[], type_comment='')], orelse=[Return(), Return()], type_comment="L")
for G[:] in True .KA: # type: 
    assert set(), set()
else:
    return # For(target=Subscript(value=Name(id='G', ctx=Store()), slice=Slice(), ctx=Store()), iter=Attribute(value=Constant(value=True), attr='KA', ctx=Del()), body=[Assert(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), msg=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], orelse=[Return()], type_comment='')
for b[:][set() ^ set():] in e[set():]: # type: #
    return
else:
    return # For(target=Subscript(value=Subscript(value=Name(id='b', ctx=Store()), slice=Slice(), ctx=Store()), slice=Slice(BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BoolOp(op=And(), values=[])), ctx=Store()), iter=Subscript(value=Name(id='e', ctx=Load()), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Load()), body=[Return()], orelse=[Return()], type_comment='#')
for LNx in *wu: # type: ckM<v
    return k
else:
    return # For(target=Name(id='LNx', ctx=Store()), iter=Starred(value=Name(id='wu', ctx=Del()), ctx=Del()), body=[Return(value=Name(id='k', ctx=Del()))], orelse=[Return()], type_comment="ckM<v")

<While>
while 
k:
    pass                       # While(test=BoolOp(op=Or(), values=[Expr(value=Name(id='k', ctx=Load()))]), body=[Pass()], orelse=[])
while *set()[set().e:]:
    del 
    with :
        return
    return
    continue
else:
    break
    return # While(test=Subscript(value=Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Load()), slice=Slice(Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='e', ctx=Load())), ctx=Load()), body=[Delete(targets=[]), With(items=[], body=[Return()]), Return(), Continue()], orelse=[Break(), Return()])
while {}:
    for H[:] in set(): # type: 
        return
    else:
        return
else:
    l |= set()
    while set():
        return # While(test=Dict(keys=[], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), body=[For(target=Subscript(value=Name(id='H', ctx=Store()), slice=Slice(), ctx=Store()), iter=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[Return()], type_comment='')], orelse=[AugAssign(target=Name(id='l', ctx=Store()), op=BitOr(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), While(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[])])
while 'C':
    set()
    return
else:
    t = set()
    if set():
        return
    return
    return # While(test=Constant(value="C"), body=[Expr(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return()], orelse=[Assign(targets=[Name(id='t', ctx=Store())], value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), If(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[]), Return(), Return()])
while (not set()) == set():
    assert set()
    return
else:
    assert set(), set()
    return # While(test=Compare(left=UnaryOp(op=Not(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ops=[Eq()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), body=[Assert(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return()], orelse=[Assert(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), msg=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return()])
while () @ set():
    return [set(), set()]
    return
    return
    return
else:
    return set()[:]() # While(test=BinOp(left=Tuple(elts=[], ctx=Del()), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), body=[Return(value=List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del())), Return(), Return(), Return()], orelse=[Return(value=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), args=[], keywords=[]))])
while *(set(),):
    (h,) //= X
    E <<= set()
else:
    P *= set().W # While(test=Starred(value=Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load()), ctx=Del()), body=[AugAssign(target=Tuple(elts=[Name(id='h', ctx=Store())], ctx=Store()), op=FloorDiv(), value=Name(id='X', ctx=Del())), AugAssign(target=Name(id='E', ctx=Store()), op=LShift(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], orelse=[AugAssign(target=Name(id='P', ctx=Store()), op=Mult(), value=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='W', ctx=Del()))])
while [{set() + set()}]:
    *[u] ^= {set() & set() >> set(), set() / set()}
else:
    s.N %= set()
    m **= set() # While(test=List(elts=[Set(elts=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])], ctx=Load()), body=[AugAssign(target=Starred(value=List(elts=[Name(id='u', ctx=Store())], ctx=Store()), ctx=Store()), op=BitXor(), value=Set(elts=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=RShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))]))], orelse=[AugAssign(target=Attribute(value=Name(id='s', ctx=Store()), attr='N', ctx=Store()), op=Mod(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), AugAssign(target=Name(id='m', ctx=Store()), op=Pow(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
while False:
    with set(), set(): # type: ^B
        x -= set()
else:
    v = +5 # type: % # While(test=Constant(value=False), body=[With(items=[withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], body=[AugAssign(target=Name(id='x', ctx=Store()), op=Sub(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], type_comment='^B')], orelse=[Assign(targets=[Name(id='v', ctx=Store())], value=UnaryOp(op=UAdd(), operand=Constant(value=5)), type_comment='%')])
while ~Y():
    T = set()
    return
    return
else:
    p = set()[:]() # While(test=UnaryOp(op=Invert(), operand=Call(func=Name(id='Y', ctx=Del()), args=[], keywords=[])), body=[Assign(targets=[Name(id='T', ctx=Store())], value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return(), Return()], orelse=[Assign(targets=[Name(id='p', ctx=Store())], value=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load()), args=[], keywords=[]))])

<If>
if :
    return
    for [a] in set(): # type: 
        break
    pass
    continue
    return
else:
    del set()[:] # If(test=BoolOp(op=Or(), values=[]), body=[Return(), For(target=List(elts=[Name(id='a', ctx=Store())], ctx=Store()), iter=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Break()], orelse=[], type_comment=""), Pass(), Continue(), Return()], orelse=[Delete(targets=[Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load())])])
if set()[:]():
    set()
    u %= set()
    return
else:
    return
    return # If(test=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), args=[], keywords=[]), body=[Expr(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), AugAssign(target=Name(id='u', ctx=Store()), op=Mod(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return()], orelse=[Return(), Return()])
if None >= set():
    assert set(), set().q
    return
    return
else:
    return # If(test=Compare(left=Constant(value=None), ops=[GtE()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), body=[Assert(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), msg=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='q', ctx=Del())), Return(), Return()], orelse=[Return()])
if +va:
    while set():
        return
    if set():
        return
else:
    Z = *set() # If(test=UnaryOp(op=UAdd(), operand=Name(id='va', ctx=Load())), body=[While(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[]), If(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[])], orelse=[Assign(targets=[Name(id='Z', ctx=Store())], value=Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Del()))])
if 
set() << []:
    with :
        return
    assert ()
else:
    j &= set()
    return set() # If(test=Expr(value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=List(elts=[], ctx=Load()))), body=[With(items=[], body=[Return()]), Assert(test=Tuple(elts=[], ctx=Del()))], orelse=[AugAssign(target=Name(id='j', ctx=Store()), op=BitAnd(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
if {set(): set()}:
    G[:] **= set()
else:
    (Q,) += set() # If(test=Dict(keys=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), body=[AugAssign(target=Subscript(value=Name(id='G', ctx=Store()), slice=Slice(), ctx=Store()), op=Pow(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], orelse=[AugAssign(target=Tuple(elts=[Name(id='Q', ctx=Store())], ctx=Store()), op=Add(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
if *(set(),):
    h |= set()
    D >>= set()
else:
    *W /= r # If(test=Starred(value=Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load()), ctx=Load()), body=[AugAssign(target=Name(id='h', ctx=Store()), op=BitOr(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), AugAssign(target=Name(id='D', ctx=Store()), op=RShift(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], orelse=[AugAssign(target=Starred(value=Name(id='W', ctx=Store()), ctx=Store()), op=Div(), value=Name(id='r', ctx=Del()))])
if {[set(), set(), set()]}:
    w[:].N ^= set().F
else:
    L[:].z *= set().C # If(test=Set(elts=[List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del())]), body=[AugAssign(target=Attribute(value=Subscript(value=Name(id='w', ctx=Store()), slice=Slice(), ctx=Store()), attr='N', ctx=Store()), op=BitXor(), value=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='F', ctx=Load()))], orelse=[AugAssign(target=Attribute(value=Subscript(value=Name(id='L', ctx=Store()), slice=Slice(), ctx=Store()), attr='z', ctx=Store()), op=Mult(), value=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='C', ctx=Load()))])
if 192:
    return
    k //= set()
else:
    y -= set()
    d @= set() # If(test=Constant(value=192), body=[Return(), AugAssign(target=Name(id='k', ctx=Store()), op=FloorDiv(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], orelse=[AugAssign(target=Name(id='y', ctx=Store()), op=Sub(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), AugAssign(target=Name(id='d', ctx=Store()), op=MatMult(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
if not True[set():set()]:
    S = False
else:
    E = J = set() # If(test=UnaryOp(op=Not(), operand=Subscript(value=Constant(value=True), slice=Slice(Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Load())), body=[Assign(targets=[Name(id='S', ctx=Store())], value=Constant(value=False))], orelse=[Assign(targets=[Name(id='E', ctx=Store()), Name(id='J', ctx=Store())], value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])

<Return>
return ()                                # Return(value=Tuple(elts=[], ctx=Load()))
return                                   # Return()
return *[{set(): g(), set().k: set()[:], set(): False}, set()] # Return(value=Starred(value=List(elts=[Dict(keys=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='k', ctx=Load()), Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[])], values=[Call(func=Name(id='g', ctx=Load()), args=[], keywords=[]), Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), Constant(value=False)]), BoolOp(op=Or(), values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])], ctx=Del()), ctx=Load()))
return O2sIF9wuGDe5hBzM10X7a >> (not 
{[*(set(), set())[set() ^ set():set() % set()]].idboHj}) # Return(value=BinOp(left=Name(id='O2sIF9wuGDe5hBzM10X7a', ctx=Del()), op=RShift(), right=UnaryOp(op=Not(), operand=Expr(value=Set(elts=[Attribute(value=List(elts=[Starred(value=Subscript(value=Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del()), slice=Slice(BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), ctx=Load()), ctx=Del())], ctx=Load()), attr='idboHj', ctx=Del())])))))
return ((set() | set()) << set() - set()) @ (set() ** set() * (set() / set())) # Return(value=BinOp(left=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=LShift(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Sub(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=MatMult(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Mult(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))))
return 7.33                              # Return(value=Constant(value=7.33))
return 'G'                               # Return(value=Constant(value="G"))
return ~-None & set():, X=set()) // +set()[:].c(set()[:], set()[:], l=set(), Q=set()) # Return(value=BinOp(left=UnaryOp(op=Invert(), operand=UnaryOp(op=USub(), operand=Constant(value=None))), op=BitAnd(), right=BinOp(left=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load()), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], keywords=[keyword(arg='X', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))]), op=FloorDiv(), right=UnaryOp(op=UAdd(), operand=Call(func=Attribute(value=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), attr='c', ctx=Del()), args=[Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load())], keywords=[keyword(arg='l', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), keyword(arg='Q', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])))))
return (set() + set() and 24)[''[set() | set():set() % set():set()]:] # Return(value=Subscript(value=BoolOp(op=And(), values=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Constant(value=24)]), slice=Slice(Subscript(value=Constant(value=''), slice=Slice(BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Load())), ctx=Load()))
return (oiLk < set() != set()) > 496 <= True # Return(value=Compare(left=Compare(left=Name(id='oiLk', ctx=Del()), ops=[Lt(), NotEq()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Name(id='f8', ctx=Del())]), ops=[Gt(), LtE()], comparators=[Constant(value=496), Constant(value=True)]))

<Delete>
del ((not {(set()[:]().PTA2)[*set():
{}]})) // True, [] # Delete(targets=[BinOp(left=Compare(left=UnaryOp(op=Not(), operand=Set(elts=[Subscript(value=BoolOp(op=And(), values=[Attribute(value=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), args=[], keywords=[]), attr='PTA2', ctx=Load())]), slice=Slice(Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Del()), Expr(value=Dict(keys=[], values=[]))), ctx=Load())])), ops=[], comparators=[Tuple(elts=[Name(id='G', ctx=Load())], ctx=Load())]), op=FloorDiv(), right=Constant(value=True)), List(elts=[], ctx=Del())])
del [set(), set(), set() / set()], *().QD, y4iFkwX # Delete(targets=[List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], ctx=Load()), Attribute(value=Starred(value=Tuple(elts=[], ctx=Del()), ctx=Load()), attr='QD', ctx=Del()), Name(id='y4iFkwX', ctx=Del())])
del set(), set(), set() ^ set(), set() % set() >> set() - set() @ set() # Delete(targets=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=RShift(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Sub(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=MatMult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))))])
del set() << set(), set() | set(), set() ** set(), set() * (set() + set()) # Delete(targets=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))])
del 6                                    # Delete(targets=[Constant(value=6)])
del '_'                                  # Delete(targets=[Constant(value='_')])
del ~50.413, +-set()[:][set() & set():].F_(set() + set(), set()[:], L=set(), Z=set()) # Delete(targets=[UnaryOp(op=Invert(), operand=Constant(value=50.413)), UnaryOp(op=UAdd(), operand=UnaryOp(op=USub(), operand=Call(func=Attribute(value=Subscript(value=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load()), slice=Slice(BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), ctx=Load()), attr='F_', ctx=Del()), args=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Add(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del())], keywords=[keyword(arg='L', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), keyword(arg='Z', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])))])
del set() or None or z(), (set() and set())[False:c():T()] # Delete(targets=[BoolOp(op=Or(), values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Constant(value=None), Call(func=Name(id='z', ctx=Del()), args=[], keywords=[])]), Subscript(value=BoolOp(op=And(), values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), slice=Slice(Constant(value=False), Call(func=Name(id='c', ctx=Del()), args=[], keywords=[]), Call(func=Name(id='T', ctx=Load()), args=[], keywords=[])), ctx=Del())])
del ''                                   # Delete(targets=[Constant(value="")])
del k5vofh3xGZH == R1rc                  # Delete(targets=[Compare(left=Name(id='k5vofh3xGZH', ctx=Load()), ops=[Eq()], comparators=[Name(id='R1rc', ctx=Del()), Name(id='HPJup', ctx=Del())])])

<Assert>
assert {}                                # Assert(test=Dict(keys=[], values=[List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Constant(value=6.7), Attribute(value=BoolOp(op=Or(), values=[]), attr='Q', ctx=Del()), Compare(left=Expr(value=UnaryOp(op=Invert(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), ops=[NotIn()], comparators=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])], ctx=Del())]))
assert Z4mcX(set(), o=set()), ICkz[*(set() ^ set(),):] # Assert(test=Call(func=Name(id='Z4mcX', ctx=Del()), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], keywords=[keyword(arg='o', value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))]), msg=Subscript(value=Name(id='ICkz', ctx=Load()), slice=Slice(Starred(value=Tuple(elts=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitXor(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], ctx=Load()), ctx=Load())), ctx=Load()))
assert [().H, {().h}, *(set(),)[*set() / set():set() // set()]] # Assert(test=List(elts=[Attribute(value=Tuple(elts=[], ctx=Del()), attr='H', ctx=Del()), Set(elts=[Attribute(value=Tuple(elts=[], ctx=Del()), attr='h', ctx=Load())]), Subscript(value=Starred(value=Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load()), ctx=Del()), slice=Slice(Starred(value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Div(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Load()), BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=FloorDiv(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), ctx=Del())], ctx=Load()))
assert set() ** set() % (set() - set()), (set() >> (set() & set())) + (set() | set()) * (set() << set()) # Assert(test=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Mod(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Sub(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), msg=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=RShift(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), op=Add(), right=BinOp(left=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitOr(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), op=Mult(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=LShift(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))))
assert None                              # Assert(test=Constant(value=None))
assert True                              # Assert(test=Constant(value=True))
assert not 331                           # Assert(test=UnaryOp(op=Not(), operand=Constant(value=331)))
assert -set()[:].x(set(), set()), +(not (set()[:]())[:]) # Assert(test=UnaryOp(op=USub(), operand=Call(func=Attribute(value=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), attr='x', ctx=Load()), args=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], keywords=[])), msg=UnaryOp(op=UAdd(), operand=UnaryOp(op=Not(), operand=Subscript(value=BoolOp(op=And(), values=[Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Load()), args=[], keywords=[])]), slice=Slice(), ctx=Load()))))
assert 'X' @ (set())[False:set()]['':][False:][9:'Rbw':'m'] # Assert(test=BinOp(left=Constant(value='X'), op=MatMult(), right=Subscript(value=Subscript(value=Subscript(value=Subscript(value=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), slice=Slice(Constant(value=False), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), ctx=Del()), slice=Slice(Constant(value="")), ctx=Load()), slice=Slice(Constant(value=False)), ctx=Load()), slice=Slice(Constant(value=9), Constant(value="Rbw"), Constant(value="m")), ctx=Del())))
assert bwxOpNKPEWF6yVnaubG5BIrJ2lt3AiD97QMsvf_LjYeSZHqohR0g81TUd # Assert(test=Name(id='bwxOpNKPEWF6yVnaubG5BIrJ2lt3AiD97QMsvf_LjYeSZHqohR0g81TUd', ctx=Del()))

<Pass>
pass                                     # Pass()
pass                                     # Pass()
pass                                     # Pass()
pass                                     # Pass()
pass                                     # Pass()
pass                                     # Pass()
pass                                     # Pass()
pass                                     # Pass()
pass                                     # Pass()
pass                                     # Pass()

<Break>
break                                    # Break()
break                                    # Break()
break                                    # Break()
break                                    # Break()
break                                    # Break()
break                                    # Break()
break                                    # Break()
break                                    # Break()
break                                    # Break()
break                                    # Break()

<Continue>
continue                                 # Continue()
continue                                 # Continue()
continue                                 # Continue()
continue                                 # Continue()
continue                                 # Continue()
continue                                 # Continue()
continue                                 # Continue()
continue                                 # Continue()
continue                                 # Continue()
continue                                 # Continue()

<With>
with :
    [c, (y,)] //= {}
    with set(), set(): # type: t
        return # With(items=[], body=[AugAssign(target=List(elts=[Name(id='c', ctx=Store()), Tuple(elts=[Name(id='y', ctx=Store())], ctx=Store())], ctx=Store()), op=FloorDiv(), value=Dict(keys=[], values=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='e', ctx=Load())])), With(items=[withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], body=[Return()], type_comment='t')])
with set() as C, *set() as *P: # type: 
    while (set())[:]:
        break # With(items=[withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), optional_vars=Name(id='C', ctx=Store())), withitem(context_expr=Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Del()), optional_vars=Starred(value=Name(id='P', ctx=Store()), ctx=Store()))], body=[While(test=Subscript(value=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[]), slice=Slice(), ctx=Load()), body=[Break()], orelse=[])], type_comment="")
with '' as G[:]._: # type: H!
    del set(), set(), set()
    set()
    pass
    continue # With(items=[withitem(context_expr=Constant(value=''), optional_vars=Attribute(value=Subscript(value=Name(id='G', ctx=Store()), slice=Slice(), ctx=Store()), attr='_', ctx=Store()))], body=[Delete(targets=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])]), Expr(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Pass(), Continue()], type_comment="H!")
with : # type: |S9vg
    for B in set(): # type: 
        return # With(items=[withitem(context_expr=BoolOp(op=And(), values=[]))], body=[For(target=Name(id='B', ctx=Store()), iter=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[], type_comment="")], type_comment='|S9vg')
with () as Y[:]: # type: t>A
    b = E = set() # With(items=[withitem(context_expr=Tuple(elts=[], ctx=Load()), optional_vars=Subscript(value=Name(id='Y', ctx=Store()), slice=Slice(), ctx=Store()))], body=[Assign(targets=[Name(id='b', ctx=Store()), Name(id='E', ctx=Store())], value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], type_comment="t>A")
with set(), set() as K[:]: # type: f
    if set():
        return
    return
    return # With(items=[withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), optional_vars=Subscript(value=Name(id='K', ctx=Store()), slice=Slice(), ctx=Store()))], body=[If(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[]), Return(), Return()], type_comment="f")
with set(), set(), [], set() as r[:]: # type: n
    assert set()
    return
    return # With(items=[withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), withitem(context_expr=List(elts=[], ctx=Del())), withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), optional_vars=Subscript(value=Name(id='r', ctx=Store()), slice=Slice(), ctx=Store()))], body=[Assert(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return(), Return()], type_comment='n')
with set() as v: # type: $5a?@c
    assert set(), set()
    return
    return
    return # With(items=[withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), optional_vars=Name(id='v', ctx=Store()))], body=[Assert(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), msg=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return(), Return(), Return()], type_comment="$5a?@c")
with set() as j: # type:  j
    return set()[:]()
    return
    return
    return # With(items=[withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), optional_vars=Name(id='j', ctx=Store()))], body=[Return(value=Call(func=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), args=[], keywords=[])), Return(), Return(), Return()], type_comment=' j')
with set(): # type: 
    J[:] &= set() * set()
    h /= I
    return # With(items=[withitem(context_expr=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], body=[AugAssign(target=Subscript(value=Name(id='J', ctx=Store()), slice=Slice(), ctx=Store()), op=BitAnd(), value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), AugAssign(target=Name(id='h', ctx=Store()), op=Div(), value=Name(id='I', ctx=Load())), Return()], type_comment='')

```

让我们看看我们是否也能正确地*解析*代码。以下是一个示例：

```py
with_tree = ast.parse("""
with open('foo.txt') as myfile:
 content = myfile.readlines()
 if content is not None:
 print(content)
""") 
```

```py
python_ast_stmts_grammar = convert_ebnf_grammar(PYTHON_AST_STMTS_GRAMMAR)
with_tree_str = ast.dump(with_tree.body[0])  # get the `With(...)` subtree
print(with_tree_str)
with_solver = ISLaSolver(python_ast_stmts_grammar)
assert with_solver.check(with_tree_str) 
```

```py
With(items=[withitem(context_expr=Call(func=Name(id='open', ctx=Load()), args=[Constant(value='foo.txt')], keywords=[]), optional_vars=Name(id='myfile', ctx=Store()))], body=[Assign(targets=[Name(id='content', ctx=Store())], value=Call(func=Attribute(value=Name(id='myfile', ctx=Load()), attr='readlines', ctx=Load()), args=[], keywords=[])), If(test=Compare(left=Name(id='content', ctx=Load()), ops=[IsNot()], comparators=[Constant(value=None)]), body=[Expr(value=Call(func=Name(id='print', ctx=Load()), args=[Name(id='content', ctx=Load())], keywords=[]))], orelse=[])])

```

看起来我们的语法也能正确地解析非平凡代码。我们做得很好！</details> <details id="Excursion:-Function-Definitions"><summary>函数定义</summary>

现在是函数定义。这里没有太多惊喜。

```py
print(ast.dump(ast.parse("""
def f(a, b=1):
 pass
"""
), indent=4)) 
```

```py
Module(
    body=[
        FunctionDef(
            name='f',
            args=arguments(
                posonlyargs=[],
                args=[
                    arg(arg='a'),
                    arg(arg='b')],
                kwonlyargs=[],
                kw_defaults=[],
                defaults=[
                    Constant(value=1)]),
            body=[
                Pass()],
            decorator_list=[])],
    type_ignores=[])

```

```py
PYTHON_AST_DEFS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_STMTS_GRAMMAR, {
    '<stmt>': PYTHON_AST_STMTS_GRAMMAR['<stmt>'] + [ '<FunctionDef>' ],

    '<FunctionDef>': [
        'FunctionDef(name=<identifier>, args=<arguments>, body=<nonempty_stmt_list><decorator_list_param><returns>?<type_comment>?)'
    ],
    '<decorator_list_param>': [
        ', decorator_list=<expr_list>'
    ],

    '<arguments>': [
        'arguments(<posonlyargs_param>args=<arg_list><vararg>?<kwonlyargs_param><kw_defaults_param><kwarg>?<defaults_param>)'
    ],
    '<posonlyargs_param>': [
        'posonlyargs=<arg_list>, '
    ],
    '<kwonlyargs_param>': [
        ', kwonlyargs=<arg_list>'
    ],
    '<kw_defaults_param>': [
        ', kw_defaults=<expr_list>'
    ],
    '<defaults_param>': [
        ', defaults=<expr_list>'
    ],

    '<arg_list>': [ '[<args>?]' ],
    '<args>': [ '<arg>', '<arg>, <arg>' ],
    '<arg>': [ 'arg(arg=<identifier>)' ],

    '<vararg>': [ ', vararg=<arg>' ],
    '<kwarg>': [ ', kwarg=<arg>' ],
    '<returns>': [ ', returns=<expr>' ],

    # FIXME: Not handled: AsyncFunctionDef, ClassDef
}) 
```

在 Python 3.12 及以后的版本中，函数定义也有一个`type_param`字段：

```py
# do import this unconditionally
if sys.version_info >= (3, 12):
    PYTHON_AST_DEFS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_DEFS_GRAMMAR, {
    '<FunctionDef>': [
        'FunctionDef(name=<identifier>, args=<arguments>, body=<nonempty_stmt_list><decorator_list_param><returns>?<type_comment>?<type_params>?)'
    ],
    '<type_params>': [
        ', type_params=<type_param_list>',
    ],
    '<type_param_list>': [ '[<type_param>?]' ],
    '<type_param>': [ '<TypeVar>', '<ParamSpec>', '<TypeVarTuple>' ],
    '<TypeVar>': [
        'TypeVar(name=<identifier>(, bound=<expr>)?)'
    ],
    '<ParamSpec>': [
        'ParamSpec(name=<identifier>)'
    ],
    '<TypeVarTuple>': [
        'TypeVarTuple(name=<identifier>)'
    ]
    }) 
```

在 Python 3.13 及以后的版本中，几个`<FunctionDef>`和`<arguments>`属性是可选的：

```py
# do import this unconditionally
if sys.version_info >= (3, 13):
    PYTHON_AST_DEFS_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_DEFS_GRAMMAR, {
    '<FunctionDef>': [
        'FunctionDef(name=<identifier>, args=<arguments>, body=<nonempty_stmt_list><decorator_list_param>?<returns>?<type_comment>?<type_params>?)'
    ],
    '<arguments>': [
        'arguments(<posonlyargs_param>?args=<arg_list><vararg>?<kwonlyargs_param>?<kw_defaults_param>?<kwarg>?<defaults_param>?)'
    ],
    }) 
```

```py
assert is_valid_grammar(PYTHON_AST_DEFS_GRAMMAR) 
```

```py
for elt in [ '<arguments>', '<FunctionDef>' ]:
    print(elt)
    test_samples(PYTHON_AST_DEFS_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<arguments>
i, /, Wr, x                              # arguments(posonlyargs=[arg(arg='i')], args=[arg(arg='Wr'), arg(arg='x')], kwonlyargs=[], kw_defaults=[List(elts=[UnaryOp(op=UAdd(), operand=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Dict(keys=[], values=[]), Name(id='P', ctx=Load())], ctx=Load())], defaults=[])
G, /, h=, *e, u=set(), **R3              # arguments(posonlyargs=[arg(arg='G')], args=[arg(arg='h')], vararg=arg(arg='e'), kwonlyargs=[arg(arg='u')], kw_defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Starred(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ctx=Del())], kwarg=arg(arg='R3'), defaults=[BoolOp(op=Or(), values=[])])
n, C, /, s, T=set(), *S, L=set(), **j    # arguments(posonlyargs=[arg(arg='n'), arg(arg='C')], args=[arg(arg='s'), arg(arg='T')], vararg=arg(arg='S'), kwonlyargs=[arg(arg='L')], kw_defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], kwarg=arg(arg='j'), defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])
F, z, /, Q, N, *Y, X=set(), **g          # arguments(posonlyargs=[arg(arg='F'), arg(arg='z')], args=[arg(arg='Q'), arg(arg='N')], vararg=arg(arg='Y'), kwonlyargs=[arg(arg='X'), arg(arg='I')], kw_defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], kwarg=arg(arg='g'), defaults=[])
A=set(), /, B=set(), f=set(), *O6, K=set(), **Z # arguments(posonlyargs=[arg(arg='A')], args=[arg(arg='B'), arg(arg='f')], vararg=arg(arg='O6'), kwonlyargs=[arg(arg='K')], kw_defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], kwarg=arg(arg='Z'), defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])
p, y=set(), /, H=
set(), *l, Jo=set(), **V # arguments(posonlyargs=[arg(arg='p'), arg(arg='y')], args=[arg(arg='H')], vararg=arg(arg='l'), kwonlyargs=[arg(arg='Jo')], kw_defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], kwarg=arg(arg='V'), defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Expr(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
m, /, U, c, *w, **o                      # arguments(posonlyargs=[arg(arg='m')], args=[arg(arg='U'), arg(arg='c')], vararg=arg(arg='w'), kwonlyargs=[arg(arg='b'), arg(arg='q')], kw_defaults=[], kwarg=arg(arg='o'), defaults=[])
k, v, /, E, t=set() % set(), *_, **rR    # arguments(posonlyargs=[arg(arg='k'), arg(arg='v')], args=[arg(arg='E'), arg(arg='t')], vararg=arg(arg='_'), kwonlyargs=[], kw_defaults=[], kwarg=arg(arg='rR'), defaults=[BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))])
M, /, D, *d, a=set(), **Z                # arguments(posonlyargs=[arg(arg='M')], args=[arg(arg='D')], vararg=arg(arg='d'), kwonlyargs=[arg(arg='a')], kw_defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='F', ctx=Del())], kwarg=arg(arg='Z'), defaults=[])
n, Y, /, g, y=set(), *z, **U             # arguments(posonlyargs=[arg(arg='n'), arg(arg='Y')], args=[arg(arg='g'), arg(arg='y')], vararg=arg(arg='z'), kwonlyargs=[], kw_defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], kwarg=arg(arg='U'), defaults=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])])

<FunctionDef>
def U():
    return                      # FunctionDef(name='U', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[])
def F():
    pass                        # FunctionDef(name='F', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Pass()], decorator_list=[])
def u() -> set(): # type: 
    continue  # FunctionDef(name='u', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Continue()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), type_comment="")
def D() -> set(): # type: 
    break     # FunctionDef(name='D', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Break()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), type_comment='')
def w(): # type: 
    return             # FunctionDef(name='w', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], type_comment='')
def g() -> set(): # type: 
    return    # FunctionDef(name='g', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), type_comment='')
def q() -> set(): # type: 
    return    # FunctionDef(name='q', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), type_comment='')
def W() -> set():
    return             # FunctionDef(name='W', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))
def I() -> set():
    return             # FunctionDef(name='I', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))
def n() -> set(): # type: 
    return    # FunctionDef(name='n', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), type_comment="")

```</details> <details id="Excursion:-Modules"><summary>模块</summary>

我们以*模块*结束——一系列的定义。在所有其他定义之后，这现在相当直接。

```py
PYTHON_AST_MODULE_GRAMMAR: Grammar = extend_grammar(PYTHON_AST_DEFS_GRAMMAR, {
    '<start>': [ '<mod>' ],
    '<mod>': [ '<Module>' ],
    '<Module>': [ 'Module(body=<nonempty_stmt_list><type_ignore_param>)'],

    '<type_ignore_param>': [ ', type_ignores=<type_ignore_list>' ],
    '<type_ignore_list>': [ '[<type_ignores>?]' ],
    '<type_ignores>': [ '<type_ignore>', '<type_ignore>, <type_ignore>' ],
    '<type_ignore>': [ 'TypeIgnore(lineno=<integer>, tag=<string>)' ],
}) 
```

```py
# do import this unconditionally
if sys.version_info >= (3, 13):
    PYTHON_AST_MODULE_GRAMMAR: Grammar = \
        extend_grammar(PYTHON_AST_MODULE_GRAMMAR, {
        # As of 3.13, the type_ignore parameter is optional
        '<Module>': [ 'Module(body=<nonempty_stmt_list><type_ignore_param>?)'],
    }) 
```

```py
assert is_valid_grammar(PYTHON_AST_MODULE_GRAMMAR) 
```

```py
for elt in [ '<Module>' ]:
    print(elt)
    test_samples(PYTHON_AST_MODULE_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<Module>
EESc9e.w @ {
[*(not set())[y():set():{}]], } # Module(body=[Expr(value=BinOp(left=Attribute(value=Name(id='EESc9e', ctx=Del()), attr='w', ctx=Load()), op=MatMult(), right=Set(elts=[Expr(value=List(elts=[Starred(value=Subscript(value=UnaryOp(op=Not(), operand=Compare(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), ops=[], comparators=[])), slice=Slice(Call(func=Name(id='y', ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Dict(keys=[], values=[])), ctx=Load()), ctx=Load())], ctx=Del())), BoolOp(op=And(), values=[])])))], type_ignores=[])
while (None, ''):
    m = set()
    del 
    return
else:
    break
    with :
        return
    pass
return
continue # Module(body=[While(test=Tuple(elts=[Constant(value=None), Constant(value='')], ctx=Load()), body=[Assign(targets=[Name(id='m', ctx=Store())], value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Delete(targets=[]), Return()], orelse=[Break(), With(items=[], body=[Return()]), Pass()]), Return(), Continue()], type_ignores=[TypeIgnore(lineno=27, tag=''), TypeIgnore(lineno=2, tag="h")])
for I.V in set()[:]: # type: 
    return # Module(body=[For(target=Attribute(value=Name(id='I', ctx=Store()), attr='V', ctx=Store()), iter=Subscript(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), slice=Slice(), ctx=Del()), body=[Return()], orelse=[], type_comment="")], type_ignores=[TypeIgnore(lineno=131, tag='[bm')])
def Q():
    return
assert set().a
return # Module(body=[FunctionDef(name='Q', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[]), Assert(test=Attribute(value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), attr='a', ctx=Del())), Return()], type_ignores=[TypeIgnore(lineno=56, tag="M")])
if set():
    return
*h <<= set()        # Module(body=[If(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), body=[Return()], orelse=[]), AugAssign(target=Starred(value=Name(id='h', ctx=Store()), ctx=Store()), op=LShift(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], type_ignores=[TypeIgnore(lineno=8, tag=""), TypeIgnore(lineno=5, tag="")])
return [set()]
assert (set(), set()), [] # Module(body=[Return(value=List(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Load())), Assert(test=Tuple(elts=[Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])], ctx=Del()), msg=List(elts=[], ctx=Del()))], type_ignores=[TypeIgnore(lineno=89, tag=""), TypeIgnore(lineno=0, tag="Q")])
D |= set()                               # Module(body=[AugAssign(target=Name(id='D', ctx=Store()), op=BitOr(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))], type_ignores=[TypeIgnore(lineno=74, tag='1'), TypeIgnore(lineno=90, tag="")])
[o, j] += *set() % set() ** set()        # Module(body=[AugAssign(target=List(elts=[Name(id='o', ctx=Store()), Name(id='j', ctx=Store())], ctx=Store()), op=Add(), value=Starred(value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mod(), right=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Pow(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))), ctx=Del()))], type_ignores=[TypeIgnore(lineno=3980, tag="7'Z")])
x[:] /= set()
i -= set() & set()         # Module(body=[AugAssign(target=Subscript(value=Name(id='x', ctx=Store()), slice=Slice(), ctx=Store()), op=Div(), value=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), AugAssign(target=Name(id='i', ctx=Store()), op=Sub(), value=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=BitAnd(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])))], type_ignores=[TypeIgnore(lineno=40, tag='W2j')])
s //= -(set() * set())                   # Module(body=[AugAssign(target=Name(id='s', ctx=Store()), op=FloorDiv(), value=UnaryOp(op=USub(), operand=BinOp(left=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), op=Mult(), right=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))))], type_ignores=[TypeIgnore(lineno=665, tag=""), TypeIgnore(lineno=5, tag="")])

```</details>

到目前为止，我们已经涵盖了（几乎）Python 的所有 AST 元素。还有一些 Python 元素需要考虑（标记为`FIXME`，见上），但我们将把这些留给读者。让我们定义`PYTHON_AST_GRAMMAR`为这一章中出现的官方语法。

```py
PYTHON_AST_GRAMMAR = PYTHON_AST_MODULE_GRAMMAR
python_ast_grammar = convert_ebnf_grammar(PYTHON_AST_GRAMMAR) 
```

这里有一些（非常奇怪）的 Python 函数示例，我们可以生成。所有这些都是有效的，但只有*语法上*有效——通过这种方式生成的代码样本中，真正有意义的非常少。

```py
for elt in [ '<FunctionDef>' ]:
    print(elt)
    test_samples(PYTHON_AST_GRAMMAR, start_symbol=elt)
    print() 
```

```py
<FunctionDef>
def w():
    pass                        # FunctionDef(name='w', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Pass()], decorator_list=[])
def a():
    break                       # FunctionDef(name='a', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Break()], decorator_list=[])
def o():
    return                      # FunctionDef(name='o', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[])
def v(): # type: 
    continue           # FunctionDef(name='v', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Continue()], decorator_list=[], type_comment='')
def j(): # type: 
    return             # FunctionDef(name='j', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], type_comment="")
def k():
    return
    return           # FunctionDef(name='k', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return(), Return()], decorator_list=[])
def Q() -> set(): # type: 
    return    # FunctionDef(name='Q', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), type_comment='')
def d() -> None:
    return
    assert set(), set()
    return # FunctionDef(name='d', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return(), Assert(test=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]), msg=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[])), Return()], decorator_list=[], returns=Constant(value=None))
def K() -> set():
    return             # FunctionDef(name='K', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], returns=Call(func=Name(id="set", ctx=Load()), args=[], keywords=[]))
def y(): # type: 
    return             # FunctionDef(name='y', args=arguments(posonlyargs=[], args=[], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Return()], decorator_list=[], type_comment='')

```

## 用于模糊测试 Python 的类

为了方便起见，让我们引入一个名为`PythonFuzzer`的类，它利用上述语法来生成 Python 代码。这将相当容易使用。

```py
class PythonFuzzer(ISLaSolver):
  """Produce Python code."""

    def __init__(self,
                 start_symbol: Optional[str] = None, *,
                 grammar: Optional[Grammar] = None,
                 constraint: Optional[str] =None,
                 **kw_params) -> None:
  """Produce Python code. Parameters are:

 * `start_symbol`: The grammatical entity to be generated (default: `<FunctionDef>`)
 * `grammar`: The EBNF grammar to be used (default: `PYTHON__AST_GRAMMAR`); and
 * `constraint` an ISLa constraint (if any).

 Additional keyword parameters are passed to the `ISLaSolver` superclass.
 """
        if start_symbol is None:
            start_symbol = '<FunctionDef>'
        if grammar is None:
            grammar = PYTHON_AST_GRAMMAR
        assert start_symbol in grammar

        g = convert_ebnf_grammar(grammar)
        if constraint is None:
            super().__init__(g, start_symbol=start_symbol, **kw_params)
        else:
            super().__init__(g, constraint, start_symbol=start_symbol, **kw_params)

    def fuzz(self) -> str:
  """Produce a Python code string."""
        abstract_syntax_tree = eval(str(self.solve()))
        ast.fix_missing_locations(abstract_syntax_tree)
        return ast.unparse(abstract_syntax_tree) 
```

默认情况下，`PythonFuzzer`将生成一个*函数定义*——即函数头和主体。

```py
fuzzer = PythonFuzzer()
print(fuzzer.fuzz()) 
```

```py
def L():
    continue

```

通过传递一个起始符号作为参数，你可以让`PythonFuzzer`生成任意的 Python 元素：

```py
fuzzer = PythonFuzzer('<While>')
print(fuzzer.fuzz()) 
```

```py
while (set()[set():set()], *(set())):
    if {}:
        while set():
            continue
        break
    else:
        del 
        return

```

这里是一个所有可能的起始符号列表：

```py
sorted(list(PYTHON_AST_GRAMMAR.keys())) 
```

```py
['<Assert>',
 '<Assign>',
 '<Attribute>',
 '<AugAssign>',
 '<BinOp>',
 '<BoolOp>',
 '<Break>',
 '<Call>',
 '<Compare>',
 '<Constant>',
 '<Continue>',
 '<Delete>',
 '<Dict>',
 '<EmptySet>',
 '<Expr>',
 '<For>',
 '<FunctionDef>',
 '<If>',
 '<List>',
 '<Module>',
 '<Name>',
 '<Pass>',
 '<Return>',
 '<Set>',
 '<Slice>',
 '<Starred>',
 '<Subscript>',
 '<Tuple>',
 '<UnaryOp>',
 '<While>',
 '<With>',
 '<arg>',
 '<arg_list>',
 '<args>',
 '<args_param>',
 '<arguments>',
 '<bool>',
 '<boolop>',
 '<cmpop>',
 '<cmpop_list>',
 '<cmpops>',
 '<decorator_list_param>',
 '<defaults_param>',
 '<digit>',
 '<digits>',
 '<expr>',
 '<expr_list>',
 '<exprs>',
 '<float>',
 '<func>',
 '<id>',
 '<id_continue>',
 '<id_start>',
 '<identifier>',
 '<integer>',
 '<keyword>',
 '<keyword_list>',
 '<keywords>',
 '<keywords_param>',
 '<kw_defaults_param>',
 '<kwarg>',
 '<kwonlyargs_param>',
 '<lhs_Attribute>',
 '<lhs_List>',
 '<lhs_Name>',
 '<lhs_Starred>',
 '<lhs_Subscript>',
 '<lhs_Tuple>',
 '<lhs_expr>',
 '<lhs_exprs>',
 '<literal>',
 '<mod>',
 '<none>',
 '<nonempty_expr_list>',
 '<nonempty_lhs_expr_list>',
 '<nonempty_stmt_list>',
 '<nonzerodigit>',
 '<not_double_quotes>',
 '<not_single_quotes>',
 '<operator>',
 '<orelse_param>',
 '<posonlyargs_param>',
 '<returns>',
 '<start>',
 '<stmt>',
 '<stmt_list>',
 '<stmts>',
 '<string>',
 '<type_comment>',
 '<type_ignore>',
 '<type_ignore_list>',
 '<type_ignore_param>',
 '<type_ignores>',
 '<unaryop>',
 '<vararg>',
 '<withitem>',
 '<withitem_list>',
 '<withitems>']

```

## 定制 Python Fuzzer

在模糊测试时，你可能对生成的输出的*特定*属性感兴趣。我们如何影响`PythonFuzzer`生成的代码？我们探索两种方法：

+   通过调整*语法*以满足我们的需求

+   通过添加*约束*来自定义输出。

### 调整语法

调整输出生成的一个简单方法是*调整语法*。

假设你想要没有装饰器的函数定义。为了实现这一点，你可以*修改产生函数定义的规则*：

```py
PYTHON_AST_GRAMMAR['<FunctionDef>'] 
```

```py
['FunctionDef(name=<identifier>, args=<arguments>, body=<nonempty_stmt_list><decorator_list_param><returns>?<type_comment>?)']

```

作为一个 AST 规则，它以*抽象语法*的形式出现，因此我们首先必须确定我们想要调整的元素。在我们的例子中，这是 `decorator_list`。

由于 decorator_list 是一个列表，我们可以修改规则以只产生空列表。为了创建一个新的适应语法，我们不会修改现有的 `PYTHON_AST_GRAMMAR`。相反，我们使用 `extend_grammar()` 函数创建一个新的语法，其中包含一个新的、适应的 `<FunctionDef>` 规则：

```py
python_ast_grammar_without_decorators: Grammar = extend_grammar(PYTHON_AST_GRAMMAR,
{
    '<FunctionDef>' :
        ['FunctionDef(name=<identifier>, args=<arguments>, body=<nonempty_stmt_list>, decorator_list=[])']
}) 
```

然而，我们还没有完成。我们还需要确保我们的语法是*有效的*，因为任何拼写错误的非终端标识符都会在生产过程中导致问题。为此，我们使用 `is_valid_grammar()` 函数：

```py
from ExpectError import ExpectError 
```

```py
with ExpectError():
    assert is_valid_grammar(python_ast_grammar_without_decorators) 
```

```py
'<decorator_list_param>': defined, but not used. Consider applying trim_grammar() on the grammar
'<returns>': defined, but not used. Consider applying trim_grammar() on the grammar
'<decorator_list_param>': unreachable from <start>. Consider applying trim_grammar() on the grammar
'<returns>': unreachable from <start>. Consider applying trim_grammar() on the grammar
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_17604/3611578183.py", line 2, in <module>
    assert is_valid_grammar(python_ast_grammar_without_decorators)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError (expected)

```

我们可以看到，随着我们的更改，我们的语法有一个*孤立的规则*：`<returns>` 规则不再被使用。这是因为 `<returns>` 是 `<type_annotation>` 的一部分，我们刚刚已经删除了。（`<type_annotation>` 在为变量定义类型时仍然被使用。）

为了解决这个问题，我们需要从我们的语法中删除 `<returns>` 规则。幸运的是，我们有一个名为 `trim_grammar()` 的函数，它可以删除所有孤立的规则：

```py
python_ast_grammar_without_decorators = trim_grammar(python_ast_grammar_without_decorators) 
```

这样，我们的语法就变得有效了...

```py
assert is_valid_grammar(python_ast_grammar_without_decorators) 
```

...并且我们可以用它来进行模糊测试——现在不再需要装饰器：

```py
fuzzer = PythonFuzzer(grammar=python_ast_grammar_without_decorators)
print(fuzzer.fuzz()) 
```

```py
def X():
    break

```

一旦你理解了语法结构，调整语法就很简单了；但是，AST 语法很复杂；此外，你的更改和扩展会紧密地与语法结构相关联。仔细研究上面定义的各个规则。

### 使用约束进行定制

使用约束进行定制的一个更优雅的替代方案是利用*约束*来调整语法以满足你的需求。由于 `PythonFuzzer` 是从 `ISLaSolver` 派生的，我们可以传递一个 `constraint` 参数来约束语法，正如在使用约束进行模糊测试章节中讨论的那样。

如果我们想要每个标识符有 10 个字符的函数定义，我们可以使用一个 ISLa 约束：

```py
fuzzer = PythonFuzzer(constraint='str.len(<id>) = 10')
print(fuzzer.fuzz()) 
```

```py
def yWOOLwypwp(): # type: 
    return

```

我们还可以约束单个子项——比如说，函数的实际标识符。

```py
# Also works (the <identifier> has quotes)
fuzzer = PythonFuzzer(constraint='<FunctionDef>.<identifier> = "\'my_favorite_function\'"')
print(fuzzer.fuzz()) 
```

```py
@[set(), set()]
@set() | {}
@(-*set())[set():():
set()[:]()]
def my_favorite_function(dlFf=Qr, l1M=set(), *) -> 942.5:
    return

```

假设我们想要测试编译器如何处理大数字。让我们定义一个约束，使得函数体（`<nonempty_stmt_list>`）至少包含一个值至少为 1000 的整数（`<integer>`）：

```py
fuzzer = PythonFuzzer(constraint=
"""
 exists <integer> x:
 (inside(x, <nonempty_stmt_list>) and str.to.int(x) > 1000)
""")
print(fuzzer.fuzz()) 
```

```py
@[set(), +set(), 
set()]
@{set(): set(), set(): set()}
@(set(), *set() & set())
def l(r, a, /, *uXLV, _=set()[:], **Z) -> sdTYWE9b or {set(), set().R}.Vy != z1vw([]):
    del 1007

```

假设我们想要测试带有非平凡函数的编译器。以下是定义一个约束的方法，使得函数体恰好有*三个*语句（`<stmt>`）。请注意，这可能需要超过一分钟才能解决，但结果肯定是一个非平凡函数。

```py
# This will not work with ISLa 2
fuzzer = PythonFuzzer(constraint="""
 forall <FunctionDef> def: count(def, "<stmt>", "3")
""")
print(fuzzer.fuzz()) 
```

```py
@3.91
def V8(w, /, *, t=set(), C5D=set(), **foT6):
    if *{}.S[:] - ((set()) not in set() in set()):
        pass
    else:
        return

```

最后，如果我们想要装饰器列表为空，就像我们在语法修改示例中做的那样，我们可以约束装饰器列表为空：

```py
fuzzer = PythonFuzzer(constraint='<FunctionDef>..<expr_list> = "[]"')
print(fuzzer.fuzz()) 
```

```py
def l(Jws4IzSPx_O2ajk687obQB3mflULCTJWnAv9GHg0YRtVNycueKFDMihZ5rXd1pqEo, /, *, **g):
    return

```

## 修改代码

当为编译器生成代码（或者实际上，生成一般输入时），通常一个好的做法不是从头开始创建*一切*，而是*修改*现有的输入。这样，可以在*常见输入*（需要修改的输入）和*不常见输入*（通过修改添加的新部分）之间达到更好的平衡。

### 解析输入

要*修改*输入，我们首先需要能够*解析*它们。这正是语法真正发挥作用的地方——它真的能够解析所有可能的代码吗？这就是为什么依赖于一个*现有*的、经过验证的解析器（在我们的例子中是 Python 解析器）并在一个*抽象*（在我们的例子中是 AST）上操作是非常方便的。

我们已经看到如何使用`ast.parse()`将代码解析成 AST：

```py
def sum(a, b):    # A simple example
    the_sum = a + b
    return the_sum 
```

```py
sum_source = inspect.getsource(sum)
sum_tree = ast.parse(sum_source)
print(ast.unparse(sum_tree)) 
```

```py
def sum(a, b):
    the_sum = a + b
    return the_sum

```

```py
sum_str = ast.dump(sum_tree)
sum_str 
```

```py
"Module(body=[FunctionDef(name='sum', args=arguments(posonlyargs=[], args=[arg(arg='a'), arg(arg='b')], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Assign(targets=[Name(id='the_sum', ctx=Store())], value=BinOp(left=Name(id='a', ctx=Load()), op=Add(), right=Name(id='b', ctx=Load()))), Return(value=Name(id='the_sum', ctx=Load()))], decorator_list=[])], type_ignores=[])"

```

我们的语法能够解析这个（非平凡）字符串：

```py
solver = ISLaSolver(python_ast_grammar)
assert solver.check(sum_str) 
```

要修改输入，我们首先必须将其解析成*推导树*结构。这是（再次）代码的树表示，但这次，使用的是*我们*语法的元素。

```py
sum_tree = solver.parse(sum_str) 
```

让我们检查一下推导树的样子。唉，字符串表示非常长，并不那么有用：

```py
len(repr(sum_tree)) 
```

```py
8737

```

```py
repr(sum_tree)[:200] 
```

```py
"DerivationTree('<start>', (DerivationTree('<mod>', (DerivationTree('<Module>', (DerivationTree('Module(body=', (), id=495073), DerivationTree('<nonempty_stmt_list>', (DerivationTree('', (), id=495071"

```

然而，我们可以*可视化*推导树：

```py
from [GrammarFuzzer import display_tree 
```

```py
display_tree(sum_tree) 
```

<svg width="2150pt" height="1228pt" viewBox="0.00 0.00 2150.25 1228.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 1224.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="1188.5" y="-1206.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="1188.5" y="-1156.7" font-family="Times,serif" font-size="14.00"><mod></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="1188.5" y="-1106.45" font-family="Times,serif" font-size="14.00"><Module></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="995.5" y="-1056.2" font-family="Times,serif" font-size="14.00">Module(body=</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="1117.5" y="-1056.2" font-family="Times,serif" font-size="14.00"><nonempty_stmt_list></text></g> <g id="edge4" class="edge"><title>2->4</title></g> <g id="node201" class="node"><title>200</title> <text text-anchor="middle" x="1259.5" y="-1056.2" font-family="Times,serif" font-size="14.00"><type_ignore_param></text></g> <g id="edge200" class="edge"><title>2->200</title></g> <g id="node207" class="node"><title>206</title> <text text-anchor="middle" x="1354.5" y="-1056.2" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge206" class="edge"><title>2->206</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="1061.5" y="-1005.95" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="1117.5" y="-1005.95" font-family="Times,serif" font-size="14.00"><stmts></text></g> <g id="edge6" class="edge"><title>4->6</title></g> <g id="node200" class="node"><title>199</title> <text text-anchor="middle" x="1173.5" y="-1005.95" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge199" class="edge"><title>4->199</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="1117.5" y="-955.7" font-family="Times,serif" font-size="14.00"><stmt></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="1117.5" y="-905.45" font-family="Times,serif" font-size="14.00"><FunctionDef></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="61.5" y="-855.2" font-family="Times,serif" font-size="14.00">FunctionDef(name=</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="167.5" y="-855.2" font-family="Times,serif" font-size="14.00"><identifier></text></g> <g id="edge10" class="edge"><title>8->10</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="681.5" y="-855.2" font-family="Times,serif" font-size="14.00">, args=</text></g> <g id="edge23" class="edge"><title>8->23</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="755.5" y="-855.2" font-family="Times,serif" font-size="14.00"><arguments></text></g> <g id="edge24" class="edge"><title>8->24</title></g> <g id="node82" class="node"><title>81</title> <text text-anchor="middle" x="1066.5" y="-855.2" font-family="Times,serif" font-size="14.00">, body=</text></g> <g id="edge81" class="edge"><title>8->81</title></g> <g id="node83" class="node"><title>82</title> <text text-anchor="middle" x="1168.5" y="-855.2" font-family="Times,serif" font-size="14.00"><nonempty_stmt_list></text></g> <g id="edge82" class="edge"><title>8->82</title></g> <g id="node191" class="node"><title>190</title> <text text-anchor="middle" x="1315.5" y="-855.2" font-family="Times,serif" font-size="14.00"><decorator_list_param></text></g> <g id="edge190" class="edge"><title>8->190</title></g> <g id="node197" class="node"><title>196</title> <text text-anchor="middle" x="1432.5" y="-855.2" font-family="Times,serif" font-size="14.00"><returns-1></text></g> <g id="edge196" class="edge"><title>8->196</title></g> <g id="node198" class="node"><title>197</title> <text text-anchor="middle" x="1537.5" y="-855.2" font-family="Times,serif" font-size="14.00"><type_comment-3></text></g> <g id="edge197" class="edge"><title>8->197</title></g> <g id="node199" class="node"><title>198</title> <text text-anchor="middle" x="1625.5" y="-855.2" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge198" class="edge"><title>8->198</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="75.5" y="-804.95" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="121.5" y="-804.95" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge12" class="edge"><title>10->12</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="167.5" y="-804.95" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge22" class="edge"><title>10->22</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="28.5" y="-754.7" font-family="Times,serif" font-size="14.00"><id_start></text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="121.5" y="-754.7" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge15" class="edge"><title>12->15</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="26.5" y="-704.45" font-family="Times,serif" font-size="14.00">s (115)</text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="103.5" y="-704.45" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="207.5" y="-704.45" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge18" class="edge"><title>15->18</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="103.5" y="-654.2" font-family="Times,serif" font-size="14.00">u (117)</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="189.5" y="-654.2" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge19" class="edge"><title>18->19</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="293.5" y="-654.2" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge21" class="edge"><title>18->21</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="189.5" y="-603.95" font-family="Times,serif" font-size="14.00">m (109)</text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="230.5" y="-804.95" font-family="Times,serif" font-size="14.00">arguments(</text></g> <g id="edge25" class="edge"><title>24->25</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="340.5" y="-804.95" font-family="Times,serif" font-size="14.00"><posonlyargs_param></text></g> <g id="edge26" class="edge"><title>24->26</title></g> <g id="node34" class="node"><title>33</title> <text text-anchor="middle" x="435.5" y="-804.95" font-family="Times,serif" font-size="14.00">args=</text></g> <g id="edge33" class="edge"><title>24->33</title></g> <g id="node35" class="node"><title>34</title> <text text-anchor="middle" x="497.5" y="-804.95" font-family="Times,serif" font-size="14.00"><arg_list></text></g> <g id="edge34" class="edge"><title>24->34</title></g> <g id="node61" class="node"><title>60</title> <text text-anchor="middle" x="575.5" y="-804.95" font-family="Times,serif" font-size="14.00"><vararg-1></text></g> <g id="edge60" class="edge"><title>24->60</title></g> <g id="node62" class="node"><title>61</title> <text text-anchor="middle" x="685.5" y="-804.95" font-family="Times,serif" font-size="14.00"><kwonlyargs_param></text></g> <g id="edge61" class="edge"><title>24->61</title></g> <g id="node68" class="node"><title>67</title> <text text-anchor="middle" x="825.5" y="-804.95" font-family="Times,serif" font-size="14.00"><kw_defaults_param></text></g> <g id="edge67" class="edge"><title>24->67</title></g> <g id="node74" class="node"><title>73</title> <text text-anchor="middle" x="936.5" y="-804.95" font-family="Times,serif" font-size="14.00"><kwarg-1></text></g> <g id="edge73" class="edge"><title>24->73</title></g> <g id="node75" class="node"><title>74</title> <text text-anchor="middle" x="1035.5" y="-804.95" font-family="Times,serif" font-size="14.00"><defaults_param></text></g> <g id="edge74" class="edge"><title>24->74</title></g> <g id="node81" class="node"><title>80</title> <text text-anchor="middle" x="1119.5" y="-804.95" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge80" class="edge"><title>24->80</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="258.5" y="-754.7" font-family="Times,serif" font-size="14.00">posonlyargs=</text></g> <g id="edge27" class="edge"><title>26->27</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="341.5" y="-754.7" font-family="Times,serif" font-size="14.00"><arg_list></text></g> <g id="edge28" class="edge"><title>26->28</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="391.5" y="-754.7" font-family="Times,serif" font-size="14.00">,</text></g> <g id="edge32" class="edge"><title>26->32</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="286.5" y="-704.45" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="345.5" y="-704.45" font-family="Times,serif" font-size="14.00"><args-1></text></g> <g id="edge30" class="edge"><title>28->30</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="404.5" y="-704.45" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge31" class="edge"><title>28->31</title></g> <g id="node36" class="node"><title>35</title> <text text-anchor="middle" x="435.5" y="-754.7" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge35" class="edge"><title>34->35</title></g> <g id="node37" class="node"><title>36</title> <text text-anchor="middle" x="494.5" y="-754.7" font-family="Times,serif" font-size="14.00"><args-1></text></g> <g id="edge36" class="edge"><title>34->36</title></g> <g id="node60" class="node"><title>59</title> <text text-anchor="middle" x="553.5" y="-754.7" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge59" class="edge"><title>34->59</title></g> <g id="node38" class="node"><title>37</title> <text text-anchor="middle" x="494.5" y="-704.45" font-family="Times,serif" font-size="14.00"><args></text></g> <g id="edge37" class="edge"><title>36->37</title></g> <g id="node39" class="node"><title>38</title> <text text-anchor="middle" x="408.5" y="-654.2" font-family="Times,serif" font-size="14.00"><arg></text></g> <g id="edge38" class="edge"><title>37->38</title></g> <g id="node49" class="node"><title>48</title> <text text-anchor="middle" x="494.5" y="-654.2" font-family="Times,serif" font-size="14.00">,</text></g> <g id="edge48" class="edge"><title>37->48</title></g> <g id="node50" class="node"><title>49</title> <text text-anchor="middle" x="559.5" y="-654.2" font-family="Times,serif" font-size="14.00"><arg></text></g> <g id="edge49" class="edge"><title>37->49</title></g> <g id="node40" class="node"><title>39</title> <text text-anchor="middle" x="301.5" y="-603.95" font-family="Times,serif" font-size="14.00">arg(arg=</text></g> <g id="edge39" class="edge"><title>38->39</title></g> <g id="node41" class="node"><title>40</title> <text text-anchor="middle" x="375.5" y="-603.95" font-family="Times,serif" font-size="14.00"><identifier></text></g> <g id="edge40" class="edge"><title>38->40</title></g> <g id="node48" class="node"><title>47</title> <text text-anchor="middle" x="441.5" y="-603.95" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge47" class="edge"><title>38->47</title></g> <g id="node42" class="node"><title>41</title> <text text-anchor="middle" x="329.5" y="-553.7" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge41" class="edge"><title>40->41</title></g> <g id="node43" class="node"><title>42</title> <text text-anchor="middle" x="375.5" y="-553.7" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge42" class="edge"><title>40->42</title></g> <g id="node47" class="node"><title>46</title> <text text-anchor="middle" x="421.5" y="-553.7" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge46" class="edge"><title>40->46</title></g> <g id="node44" class="node"><title>43</title> <text text-anchor="middle" x="329.5" y="-503.45" font-family="Times,serif" font-size="14.00"><id_start></text></g> <g id="edge43" class="edge"><title>42->43</title></g> <g id="node46" class="node"><title>45</title> <text text-anchor="middle" x="421.5" y="-503.45" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge45" class="edge"><title>42->45</title></g> <g id="node45" class="node"><title>44</title> <text text-anchor="middle" x="329.5" y="-453.2" font-family="Times,serif" font-size="14.00">a (97)</text></g> <g id="edge44" class="edge"><title>43->44</title></g> <g id="node51" class="node"><title>50</title> <text text-anchor="middle" x="498.5" y="-603.95" font-family="Times,serif" font-size="14.00">arg(arg=</text></g> <g id="edge50" class="edge"><title>49->50</title></g> <g id="node52" class="node"><title>51</title> <text text-anchor="middle" x="572.5" y="-603.95" font-family="Times,serif" font-size="14.00"><identifier></text></g> <g id="edge51" class="edge"><title>49->51</title></g> <g id="node59" class="node"><title>58</title> <text text-anchor="middle" x="638.5" y="-603.95" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge58" class="edge"><title>49->58</title></g> <g id="node53" class="node"><title>52</title> <text text-anchor="middle" x="526.5" y="-553.7" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge52" class="edge"><title>51->52</title></g> <g id="node54" class="node"><title>53</title> <text text-anchor="middle" x="572.5" y="-553.7" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge53" class="edge"><title>51->53</title></g> <g id="node58" class="node"><title>57</title> <text text-anchor="middle" x="618.5" y="-553.7" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge57" class="edge"><title>51->57</title></g> <g id="node55" class="node"><title>54</title> <text text-anchor="middle" x="543.5" y="-503.45" font-family="Times,serif" font-size="14.00"><id_start></text></g> <g id="edge54" class="edge"><title>53->54</title></g> <g id="node57" class="node"><title>56</title> <text text-anchor="middle" x="635.5" y="-503.45" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge56" class="edge"><title>53->56</title></g> <g id="node56" class="node"><title>55</title> <text text-anchor="middle" x="543.5" y="-453.2" font-family="Times,serif" font-size="14.00">b (98)</text></g> <g id="edge55" class="edge"><title>54->55</title></g> <g id="node63" class="node"><title>62</title> <text text-anchor="middle" x="626.5" y="-754.7" font-family="Times,serif" font-size="14.00">, kwonlyargs=</text></g> <g id="edge62" class="edge"><title>61->62</title></g> <g id="node64" class="node"><title>63</title> <text text-anchor="middle" x="712.5" y="-754.7" font-family="Times,serif" font-size="14.00"><arg_list></text></g> <g id="edge63" class="edge"><title>61->63</title></g> <g id="node65" class="node"><title>64</title> <text text-anchor="middle" x="653.5" y="-704.45" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge64" class="edge"><title>63->64</title></g> <g id="node66" class="node"><title>65</title> <text text-anchor="middle" x="712.5" y="-704.45" font-family="Times,serif" font-size="14.00"><args-1></text></g> <g id="edge65" class="edge"><title>63->65</title></g> <g id="node67" class="node"><title>66</title> <text text-anchor="middle" x="771.5" y="-704.45" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge66" class="edge"><title>63->66</title></g> <g id="node69" class="node"><title>68</title> <text text-anchor="middle" x="799.5" y="-754.7" font-family="Times,serif" font-size="14.00">, kw_defaults=</text></g> <g id="edge68" class="edge"><title>67->68</title></g> <g id="node70" class="node"><title>69</title> <text text-anchor="middle" x="890.5" y="-754.7" font-family="Times,serif" font-size="14.00"><expr_list></text></g> <g id="edge69" class="edge"><title>67->69</title></g> <g id="node71" class="node"><title>70</title> <text text-anchor="middle" x="826.5" y="-704.45" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge70" class="edge"><title>69->70</title></g> <g id="node72" class="node"><title>71</title> <text text-anchor="middle" x="888.5" y="-704.45" font-family="Times,serif" font-size="14.00"><exprs-1></text></g> <g id="edge71" class="edge"><title>69->71</title></g> <g id="node73" class="node"><title>72</title> <text text-anchor="middle" x="950.5" y="-704.45" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge72" class="edge"><title>69->72</title></g> <g id="node76" class="node"><title>75</title> <text text-anchor="middle" x="982.5" y="-754.7" font-family="Times,serif" font-size="14.00">, defaults=</text></g> <g id="edge75" class="edge"><title>74->75</title></g> <g id="node77" class="node"><title>76</title> <text text-anchor="middle" x="1061.5" y="-754.7" font-family="Times,serif" font-size="14.00"><expr_list></text></g> <g id="edge76" class="edge"><title>74->76</title></g> <g id="node78" class="node"><title>77</title> <text text-anchor="middle" x="999.5" y="-704.45" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge77" class="edge"><title>76->77</title></g> <g id="node79" class="node"><title>78</title> <text text-anchor="middle" x="1061.5" y="-704.45" font-family="Times,serif" font-size="14.00"><exprs-1></text></g> <g id="edge78" class="edge"><title>76->78</title></g> <g id="node80" class="node"><title>79</title> <text text-anchor="middle" x="1123.5" y="-704.45" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge79" class="edge"><title>76->79</title></g> <g id="node84" class="node"><title>83</title> <text text-anchor="middle" x="1168.5" y="-804.95" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge83" class="edge"><title>82->83</title></g> <g id="node85" class="node"><title>84</title> <text text-anchor="middle" x="1224.5" y="-804.95" font-family="Times,serif" font-size="14.00"><stmts></text></g> <g id="edge84" class="edge"><title>82->84</title></g> <g id="node190" class="node"><title>189</title> <text text-anchor="middle" x="1280.5" y="-804.95" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge189" class="edge"><title>82->189</title></g> <g id="node86" class="node"><title>85</title> <text text-anchor="middle" x="1183.5" y="-754.7" font-family="Times,serif" font-size="14.00"><stmt></text></g> <g id="edge85" class="edge"><title>84->85</title></g> <g id="node155" class="node"><title>154</title> <text text-anchor="middle" x="1225.5" y="-754.7" font-family="Times,serif" font-size="14.00">,</text></g> <g id="edge154" class="edge"><title>84->154</title></g> <g id="node156" class="node"><title>155</title> <text text-anchor="middle" x="1302.5" y="-754.7" font-family="Times,serif" font-size="14.00"><stmts></text></g> <g id="edge155" class="edge"><title>84->155</title></g> <g id="node87" class="node"><title>86</title> <text text-anchor="middle" x="1183.5" y="-704.45" font-family="Times,serif" font-size="14.00"><Assign></text></g> <g id="edge86" class="edge"><title>85->86</title></g> <g id="node88" class="node"><title>87</title> <text text-anchor="middle" x="873.5" y="-654.2" font-family="Times,serif" font-size="14.00">Assign(targets=</text></g> <g id="edge87" class="edge"><title>86->87</title></g> <g id="node89" class="node"><title>88</title> <text text-anchor="middle" x="1008.5" y="-654.2" font-family="Times,serif" font-size="14.00"><nonempty_lhs_expr_list></text></g> <g id="edge88" class="edge"><title>86->88</title></g> <g id="node122" class="node"><title>121</title> <text text-anchor="middle" x="1122.5" y="-654.2" font-family="Times,serif" font-size="14.00">, value=</text></g> <g id="edge121" class="edge"><title>86->121</title></g> <g id="node123" class="node"><title>122</title> <text text-anchor="middle" x="1183.5" y="-654.2" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge122" class="edge"><title>86->122</title></g> <g id="node153" class="node"><title>152</title> <text text-anchor="middle" x="1276.5" y="-654.2" font-family="Times,serif" font-size="14.00"><type_comment-1></text></g> <g id="edge152" class="edge"><title>86->152</title></g> <g id="node154" class="node"><title>153</title> <text text-anchor="middle" x="1364.5" y="-654.2" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge153" class="edge"><title>86->153</title></g> <g id="node90" class="node"><title>89</title> <text text-anchor="middle" x="906.5" y="-603.95" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge89" class="edge"><title>88->89</title></g> <g id="node91" class="node"><title>90</title> <text text-anchor="middle" x="974.5" y="-603.95" font-family="Times,serif" font-size="14.00"><lhs_exprs></text></g> <g id="edge90" class="edge"><title>88->90</title></g> <g id="node121" class="node"><title>120</title> <text text-anchor="middle" x="1042.5" y="-603.95" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge120" class="edge"><title>88->120</title></g> <g id="node92" class="node"><title>91</title> <text text-anchor="middle" x="864.5" y="-553.7" font-family="Times,serif" font-size="14.00"><lhs_expr></text></g> <g id="edge91" class="edge"><title>90->91</title></g> <g id="node93" class="node"><title>92</title> <text text-anchor="middle" x="859.5" y="-503.45" font-family="Times,serif" font-size="14.00"><lhs_Name></text></g> <g id="edge92" class="edge"><title>91->92</title></g> <g id="node94" class="node"><title>93</title> <text text-anchor="middle" x="736.5" y="-453.2" font-family="Times,serif" font-size="14.00">Name(id=</text></g> <g id="edge93" class="edge"><title>92->93</title></g> <g id="node95" class="node"><title>94</title> <text text-anchor="middle" x="815.5" y="-453.2" font-family="Times,serif" font-size="14.00"><identifier></text></g> <g id="edge94" class="edge"><title>92->94</title></g> <g id="node120" class="node"><title>119</title> <text text-anchor="middle" x="903.5" y="-453.2" font-family="Times,serif" font-size="14.00">, ctx=Store())</text></g> <g id="edge119" class="edge"><title>92->119</title></g> <g id="node96" class="node"><title>95</title> <text text-anchor="middle" x="769.5" y="-402.95" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge95" class="edge"><title>94->95</title></g> <g id="node97" class="node"><title>96</title> <text text-anchor="middle" x="815.5" y="-402.95" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge96" class="edge"><title>94->96</title></g> <g id="node119" class="node"><title>118</title> <text text-anchor="middle" x="861.5" y="-402.95" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge118" class="edge"><title>94->118</title></g> <g id="node98" class="node"><title>97</title> <text text-anchor="middle" x="742.5" y="-352.7" font-family="Times,serif" font-size="14.00"><id_start></text></g> <g id="edge97" class="edge"><title>96->97</title></g> <g id="node100" class="node"><title>99</title> <text text-anchor="middle" x="834.5" y="-352.7" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge99" class="edge"><title>96->99</title></g> <g id="node99" class="node"><title>98</title> <text text-anchor="middle" x="742.5" y="-302.45" font-family="Times,serif" font-size="14.00">t (116)</text></g> <g id="edge98" class="edge"><title>97->98</title></g> <g id="node101" class="node"><title>100</title> <text text-anchor="middle" x="826.5" y="-302.45" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge100" class="edge"><title>99->100</title></g> <g id="node103" class="node"><title>102</title> <text text-anchor="middle" x="930.5" y="-302.45" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge102" class="edge"><title>99->102</title></g> <g id="node102" class="node"><title>101</title> <text text-anchor="middle" x="826.5" y="-252.2" font-family="Times,serif" font-size="14.00">h (104)</text></g> <g id="edge101" class="edge"><title>100->101</title></g> <g id="node104" class="node"><title>103</title> <text text-anchor="middle" x="917.5" y="-252.2" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge103" class="edge"><title>102->103</title></g> <g id="node106" class="node"><title>105</title> <text text-anchor="middle" x="1021.5" y="-252.2" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge105" class="edge"><title>102->105</title></g> <g id="node105" class="node"><title>104</title> <text text-anchor="middle" x="917.5" y="-201.95" font-family="Times,serif" font-size="14.00">e (101)</text></g> <g id="edge104" class="edge"><title>103->104</title></g> <g id="node107" class="node"><title>106</title> <text text-anchor="middle" x="1008.5" y="-201.95" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge106" class="edge"><title>105->106</title></g> <g id="node109" class="node"><title>108</title> <text text-anchor="middle" x="1112.5" y="-201.95" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge108" class="edge"><title>105->108</title></g> <g id="node108" class="node"><title>107</title> <text text-anchor="middle" x="1008.5" y="-151.7" font-family="Times,serif" font-size="14.00">_ (95)</text></g> <g id="edge107" class="edge"><title>106->107</title></g> <g id="node110" class="node"><title>109</title> <text text-anchor="middle" x="1098.5" y="-151.7" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge109" class="edge"><title>108->109</title></g> <g id="node112" class="node"><title>111</title> <text text-anchor="middle" x="1202.5" y="-151.7" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge111" class="edge"><title>108->111</title></g> <g id="node111" class="node"><title>110</title> <text text-anchor="middle" x="1098.5" y="-101.45" font-family="Times,serif" font-size="14.00">s (115)</text></g> <g id="edge110" class="edge"><title>109->110</title></g> <g id="node113" class="node"><title>112</title> <text text-anchor="middle" x="1188.5" y="-101.45" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge112" class="edge"><title>111->112</title></g> <g id="node115" class="node"><title>114</title> <text text-anchor="middle" x="1292.5" y="-101.45" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge114" class="edge"><title>111->114</title></g> <g id="node114" class="node"><title>113</title> <text text-anchor="middle" x="1188.5" y="-51.2" font-family="Times,serif" font-size="14.00">u (117)</text></g> <g id="edge113" class="edge"><title>112->113</title></g> <g id="node116" class="node"><title>115</title> <text text-anchor="middle" x="1279.5" y="-51.2" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge115" class="edge"><title>114->115</title></g> <g id="node118" class="node"><title>117</title> <text text-anchor="middle" x="1383.5" y="-51.2" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge117" class="edge"><title>114->117</title></g> <g id="node117" class="node"><title>116</title> <text text-anchor="middle" x="1279.5" y="-0.95" font-family="Times,serif" font-size="14.00">m (109)</text></g> <g id="edge116" class="edge"><title>115->116</title></g> <g id="node124" class="node"><title>123</title> <text text-anchor="middle" x="1183.5" y="-603.95" font-family="Times,serif" font-size="14.00"><BinOp></text></g> <g id="edge123" class="edge"><title>122->123</title></g> <g id="node125" class="node"><title>124</title> <text text-anchor="middle" x="995.5" y="-553.7" font-family="Times,serif" font-size="14.00">BinOp(left=</text></g> <g id="edge124" class="edge"><title>123->124</title></g> <g id="node126" class="node"><title>125</title> <text text-anchor="middle" x="1067.5" y="-553.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge125" class="edge"><title>123->125</title></g> <g id="node137" class="node"><title>136</title> <text text-anchor="middle" x="1120.5" y="-553.7" font-family="Times,serif" font-size="14.00">, op=</text></g> <g id="edge136" class="edge"><title>123->136</title></g> <g id="node138" class="node"><title>137</title> <text text-anchor="middle" x="1183.5" y="-553.7" font-family="Times,serif" font-size="14.00"><operator></text></g> <g id="edge137" class="edge"><title>123->137</title></g> <g id="node140" class="node"><title>139</title> <text text-anchor="middle" x="1252.5" y="-553.7" font-family="Times,serif" font-size="14.00">, right=</text></g> <g id="edge139" class="edge"><title>123->139</title></g> <g id="node141" class="node"><title>140</title> <text text-anchor="middle" x="1311.5" y="-553.7" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge140" class="edge"><title>123->140</title></g> <g id="node152" class="node"><title>151</title> <text text-anchor="middle" x="1365.5" y="-553.7" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge151" class="edge"><title>123->151</title></g> <g id="node127" class="node"><title>126</title> <text text-anchor="middle" x="1066.5" y="-503.45" font-family="Times,serif" font-size="14.00"><Name></text></g> <g id="edge126" class="edge"><title>125->126</title></g> <g id="node128" class="node"><title>127</title> <text text-anchor="middle" x="986.5" y="-453.2" font-family="Times,serif" font-size="14.00">Name(id=</text></g> <g id="edge127" class="edge"><title>126->127</title></g> <g id="node129" class="node"><title>128</title> <text text-anchor="middle" x="1065.5" y="-453.2" font-family="Times,serif" font-size="14.00"><identifier></text></g> <g id="edge128" class="edge"><title>126->128</title></g> <g id="node136" class="node"><title>135</title> <text text-anchor="middle" x="1152.5" y="-453.2" font-family="Times,serif" font-size="14.00">, ctx=Load())</text></g> <g id="edge135" class="edge"><title>126->135</title></g> <g id="node130" class="node"><title>129</title> <text text-anchor="middle" x="1019.5" y="-402.95" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge129" class="edge"><title>128->129</title></g> <g id="node131" class="node"><title>130</title> <text text-anchor="middle" x="1065.5" y="-402.95" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge130" class="edge"><title>128->130</title></g> <g id="node135" class="node"><title>134</title> <text text-anchor="middle" x="1111.5" y="-402.95" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge134" class="edge"><title>128->134</title></g> <g id="node132" class="node"><title>131</title> <text text-anchor="middle" x="1047.5" y="-352.7" font-family="Times,serif" font-size="14.00"><id_start></text></g> <g id="edge131" class="edge"><title>130->131</title></g> <g id="node134" class="node"><title>133</title> <text text-anchor="middle" x="1139.5" y="-352.7" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge133" class="edge"><title>130->133</title></g> <g id="node133" class="node"><title>132</title> <text text-anchor="middle" x="1047.5" y="-302.45" font-family="Times,serif" font-size="14.00">a (97)</text></g> <g id="edge132" class="edge"><title>131->132</title></g> <g id="node139" class="node"><title>138</title> <text text-anchor="middle" x="1183.5" y="-503.45" font-family="Times,serif" font-size="14.00">Add()</text></g> <g id="edge138" class="edge"><title>137->138</title></g> <g id="node142" class="node"><title>141</title> <text text-anchor="middle" x="1312.5" y="-503.45" font-family="Times,serif" font-size="14.00"><Name></text></g> <g id="edge141" class="edge"><title>140->141</title></g> <g id="node143" class="node"><title>142</title> <text text-anchor="middle" x="1235.5" y="-453.2" font-family="Times,serif" font-size="14.00">Name(id=</text></g> <g id="edge142" class="edge"><title>141->142</title></g> <g id="node144" class="node"><title>143</title> <text text-anchor="middle" x="1314.5" y="-453.2" font-family="Times,serif" font-size="14.00"><identifier></text></g> <g id="edge143" class="edge"><title>141->143</title></g> <g id="node151" class="node"><title>150</title> <text text-anchor="middle" x="1401.5" y="-453.2" font-family="Times,serif" font-size="14.00">, ctx=Load())</text></g> <g id="edge150" class="edge"><title>141->150</title></g> <g id="node145" class="node"><title>144</title> <text text-anchor="middle" x="1268.5" y="-402.95" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge144" class="edge"><title>143->144</title></g> <g id="node146" class="node"><title>145</title> <text text-anchor="middle" x="1314.5" y="-402.95" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge145" class="edge"><title>143->145</title></g> <g id="node150" class="node"><title>149</title> <text text-anchor="middle" x="1360.5" y="-402.95" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge149" class="edge"><title>143->149</title></g> <g id="node147" class="node"><title>146</title> <text text-anchor="middle" x="1282.5" y="-352.7" font-family="Times,serif" font-size="14.00"><id_start></text></g> <g id="edge146" class="edge"><title>145->146</title></g> <g id="node149" class="node"><title>148</title> <text text-anchor="middle" x="1374.5" y="-352.7" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge148" class="edge"><title>145->148</title></g> <g id="node148" class="node"><title>147</title> <text text-anchor="middle" x="1282.5" y="-302.45" font-family="Times,serif" font-size="14.00">b (98)</text></g> <g id="edge147" class="edge"><title>146->147</title></g> <g id="node157" class="node"><title>156</title> <text text-anchor="middle" x="1379.5" y="-704.45" font-family="Times,serif" font-size="14.00"><stmt></text></g> <g id="edge156" class="edge"><title>155->156</title></g> <g id="node158" class="node"><title>157</title> <text text-anchor="middle" x="1455.5" y="-654.2" font-family="Times,serif" font-size="14.00"><Return></text></g> <g id="edge157" class="edge"><title>156->157</title></g> <g id="node159" class="node"><title>158</title> <text text-anchor="middle" x="1385.5" y="-603.95" font-family="Times,serif" font-size="14.00">Return(value=</text></g> <g id="edge158" class="edge"><title>157->158</title></g> <g id="node160" class="node"><title>159</title> <text text-anchor="middle" x="1463.5" y="-603.95" font-family="Times,serif" font-size="14.00"><expr></text></g> <g id="edge159" class="edge"><title>157->159</title></g> <g id="node189" class="node"><title>188</title> <text text-anchor="middle" x="1517.5" y="-603.95" font-family="Times,serif" font-size="14.00">) (41)</text></g> <g id="edge188" class="edge"><title>157->188</title></g> <g id="node161" class="node"><title>160</title> <text text-anchor="middle" x="1478.5" y="-553.7" font-family="Times,serif" font-size="14.00"><Name></text></g> <g id="edge160" class="edge"><title>159->160</title></g> <g id="node162" class="node"><title>161</title> <text text-anchor="middle" x="1408.5" y="-503.45" font-family="Times,serif" font-size="14.00">Name(id=</text></g> <g id="edge161" class="edge"><title>160->161</title></g> <g id="node163" class="node"><title>162</title> <text text-anchor="middle" x="1487.5" y="-503.45" font-family="Times,serif" font-size="14.00"><identifier></text></g> <g id="edge162" class="edge"><title>160->162</title></g> <g id="node188" class="node"><title>187</title> <text text-anchor="middle" x="1574.5" y="-503.45" font-family="Times,serif" font-size="14.00">, ctx=Load())</text></g> <g id="edge187" class="edge"><title>160->187</title></g> <g id="node164" class="node"><title>163</title> <text text-anchor="middle" x="1470.5" y="-453.2" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge163" class="edge"><title>162->163</title></g> <g id="node165" class="node"><title>164</title> <text text-anchor="middle" x="1516.5" y="-453.2" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge164" class="edge"><title>162->164</title></g> <g id="node187" class="node"><title>186</title> <text text-anchor="middle" x="1562.5" y="-453.2" font-family="Times,serif" font-size="14.00">' (39)</text></g> <g id="edge186" class="edge"><title>162->186</title></g> <g id="node166" class="node"><title>165</title> <text text-anchor="middle" x="1456.5" y="-402.95" font-family="Times,serif" font-size="14.00"><id_start></text></g> <g id="edge165" class="edge"><title>164->165</title></g> <g id="node168" class="node"><title>167</title> <text text-anchor="middle" x="1548.5" y="-402.95" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge167" class="edge"><title>164->167</title></g> <g id="node167" class="node"><title>166</title> <text text-anchor="middle" x="1456.5" y="-352.7" font-family="Times,serif" font-size="14.00">t (116)</text></g> <g id="edge166" class="edge"><title>165->166</title></g> <g id="node169" class="node"><title>168</title> <text text-anchor="middle" x="1540.5" y="-352.7" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge168" class="edge"><title>167->168</title></g> <g id="node171" class="node"><title>170</title> <text text-anchor="middle" x="1644.5" y="-352.7" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge170" class="edge"><title>167->170</title></g> <g id="node170" class="node"><title>169</title> <text text-anchor="middle" x="1540.5" y="-302.45" font-family="Times,serif" font-size="14.00">h (104)</text></g> <g id="edge169" class="edge"><title>168->169</title></g> <g id="node172" class="node"><title>171</title> <text text-anchor="middle" x="1631.5" y="-302.45" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge171" class="edge"><title>170->171</title></g> <g id="node174" class="node"><title>173</title> <text text-anchor="middle" x="1735.5" y="-302.45" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge173" class="edge"><title>170->173</title></g> <g id="node173" class="node"><title>172</title> <text text-anchor="middle" x="1631.5" y="-252.2" font-family="Times,serif" font-size="14.00">e (101)</text></g> <g id="edge172" class="edge"><title>171->172</title></g> <g id="node175" class="node"><title>174</title> <text text-anchor="middle" x="1722.5" y="-252.2" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge174" class="edge"><title>173->174</title></g> <g id="node177" class="node"><title>176</title> <text text-anchor="middle" x="1826.5" y="-252.2" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge176" class="edge"><title>173->176</title></g> <g id="node176" class="node"><title>175</title> <text text-anchor="middle" x="1722.5" y="-201.95" font-family="Times,serif" font-size="14.00">_ (95)</text></g> <g id="edge175" class="edge"><title>174->175</title></g> <g id="node178" class="node"><title>177</title> <text text-anchor="middle" x="1811.5" y="-201.95" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge177" class="edge"><title>176->177</title></g> <g id="node180" class="node"><title>179</title> <text text-anchor="middle" x="1915.5" y="-201.95" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge179" class="edge"><title>176->179</title></g> <g id="node179" class="node"><title>178</title> <text text-anchor="middle" x="1811.5" y="-151.7" font-family="Times,serif" font-size="14.00">s (115)</text></g> <g id="edge178" class="edge"><title>177->178</title></g> <g id="node181" class="node"><title>180</title> <text text-anchor="middle" x="1901.5" y="-151.7" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge180" class="edge"><title>179->180</title></g> <g id="node183" class="node"><title>182</title> <text text-anchor="middle" x="2005.5" y="-151.7" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge182" class="edge"><title>179->182</title></g> <g id="node182" class="node"><title>181</title> <text text-anchor="middle" x="1901.5" y="-101.45" font-family="Times,serif" font-size="14.00">u (117)</text></g> <g id="edge181" class="edge"><title>180->181</title></g> <g id="node184" class="node"><title>183</title> <text text-anchor="middle" x="1992.5" y="-101.45" font-family="Times,serif" font-size="14.00"><id_continue></text></g> <g id="edge183" class="edge"><title>182->183</title></g> <g id="node186" class="node"><title>185</title> <text text-anchor="middle" x="2096.5" y="-101.45" font-family="Times,serif" font-size="14.00"><id_continue-1></text></g> <g id="edge185" class="edge"><title>182->185</title></g> <g id="node185" class="node"><title>184</title> <text text-anchor="middle" x="1992.5" y="-51.2" font-family="Times,serif" font-size="14.00">m (109)</text></g> <g id="edge184" class="edge"><title>183->184</title></g> <g id="node192" class="node"><title>191</title> <text text-anchor="middle" x="1358.5" y="-804.95" font-family="Times,serif" font-size="14.00">, decorator_list=</text></g> <g id="edge191" class="edge"><title>190->191</title></g> <g id="node193" class="node"><title>192</title> <text text-anchor="middle" x="1453.5" y="-804.95" font-family="Times,serif" font-size="14.00"><expr_list></text></g> <g id="edge192" class="edge"><title>190->192</title></g> <g id="node194" class="node"><title>193</title> <text text-anchor="middle" x="1391.5" y="-754.7" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge193" class="edge"><title>192->193</title></g> <g id="node195" class="node"><title>194</title> <text text-anchor="middle" x="1453.5" y="-754.7" font-family="Times,serif" font-size="14.00"><exprs-1></text></g> <g id="edge194" class="edge"><title>192->194</title></g> <g id="node196" class="node"><title>195</title> <text text-anchor="middle" x="1515.5" y="-754.7" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge195" class="edge"><title>192->195</title></g> <g id="node202" class="node"><title>201</title> <text text-anchor="middle" x="1254.5" y="-1005.95" font-family="Times,serif" font-size="14.00">, type_ignores=</text></g> <g id="edge201" class="edge"><title>200->201</title></g> <g id="node203" class="node"><title>202</title> <text text-anchor="middle" x="1367.5" y="-1005.95" font-family="Times,serif" font-size="14.00"><type_ignore_list></text></g> <g id="edge202" class="edge"><title>200->202</title></g> <g id="node204" class="node"><title>203</title> <text text-anchor="middle" x="1285.5" y="-955.7" font-family="Times,serif" font-size="14.00">[ (91)</text></g> <g id="edge203" class="edge"><title>202->203</title></g> <g id="node205" class="node"><title>204</title> <text text-anchor="middle" x="1367.5" y="-955.7" font-family="Times,serif" font-size="14.00"><type_ignores-1></text></g> <g id="edge204" class="edge"><title>202->204</title></g> <g id="node206" class="node"><title>205</title> <text text-anchor="middle" x="1449.5" y="-955.7" font-family="Times,serif" font-size="14.00">] (93)</text></g> <g id="edge205" class="edge"><title>202->205</title></g></g></svg>

我们可以看到，推导树由*非终端*节点组成，其子节点构成了从语法中来的*扩展*。例如，在最顶层，我们看到一个`<start>`非终端扩展成一个`<mod>`非终端，而`<mod>`非终端又扩展成一个`<Module>`非终端。这完全来自语法规则

```py
python_ast_grammar['<start>'] 
```

```py
['<mod>']

```

和

```py
python_ast_grammar['<mod>'] 
```

```py
['<Module>']

```

`<mod>`的子节点是一个`<Module>`，它扩展成以下节点

+   `(body=`

+   `<nonempty_stmt_list>`

+   `, type_ignores=`

+   `<type_ignore_list>`

+   `)`

在这里，像`(body=`或`, type_ignores=`这样的节点被称为*终端*节点（因为它们没有更多的元素可以扩展）。非终端如`<nonempty_stmt_list>`将在下面进一步扩展——特别是，`<nonempty_stmt_list>`扩展成一个`<FunctionDef>`节点，代表`sum()`的定义。

再次，结构严格遵循我们语法中的`<Module>`定义：

```py
python_ast_grammar['<Module>'] 
```

```py
['Module(body=<nonempty_stmt_list><type_ignore_param>)']

```

如果我们以深度优先、从左到右的顺序遍历树，并且只收集终端符号，我们就能获得我们解析的原字符串。将`str()`函数应用于推导树会得到 exactly that string：

```py
str(sum_tree) 
```

```py
"Module(body=[FunctionDef(name='sum', args=arguments(posonlyargs=[], args=[arg(arg='a'), arg(arg='b')], kwonlyargs=[], kw_defaults=[], defaults=[]), body=[Assign(targets=[Name(id='the_sum', ctx=Store())], value=BinOp(left=Name(id='a', ctx=Load()), op=Add(), right=Name(id='b', ctx=Load()))), Return(value=Name(id='the_sum', ctx=Load()))], decorator_list=[])], type_ignores=[])"

```

再次，我们可以将这个字符串转换成 AST（抽象语法树），从而获得我们的原始函数：

```py
sum_ast = ast.fix_missing_locations(eval(str(sum_tree)))
print(ast.unparse(sum_ast)) 
```

```py
def sum(a, b):
    the_sum = a + b
    return the_sum

```

### 修改输入

使用推导树，我们可以对我们的输入有一个*结构化*的表示。在我们的例子中，我们已经有 AST 了，为什么还要引入一个新的呢？答案是简单的：推导树还允许我们*合成*新的输入，因为我们有一个*语法*描述了它们的结构。

最值得注意的是，我们可以按照以下方式修改输入：

1.  将输入解析成如上所示的推导树。

1.  在推导树中随机选择一些节点`<symbol>`进行修改。

1.  使用语法为`<symbol>`生成一个新的扩展。

1.  将`<symbol>`的子节点替换为刚刚生成的扩展。

1.  如有必要，重复此过程。

这是一个不错的编程任务，如果你想看看蓝图，可以看看这篇关于[使用语法进行灰盒模糊测试](https://www.fuzzingbook.org/html/GreyboxGrammarFuzzer.html)的教程中的`FragmentMutator`。

幸运的是，ISLa 已经为我们提供了执行此操作的功能。`ISLaSolver.mutate()`方法接受一个输入并根据语法规则对其进行变异。要变异的输入可以以推导树的形式给出，也可以以字符串的形式给出；其输出是一个推导树（这又可以转换成字符串）。

让我们在`sum()`函数上应用`mutate()`。`min_mutations`和`max_mutations`参数定义了应该执行多少次变异步骤；我们将两者都设置为 1，以便正好进行一次变异。

```py
sum_mutated_tree = solver.mutate(sum_str, min_mutations=1, max_mutations=1) 
```

```py
sum_mutated_ast = ast.fix_missing_locations(eval(str(sum_mutated_tree)))
print(ast.unparse(sum_mutated_ast)) 
```

```py
def sum(a, b):
    the_sum = a + b
    return the_sum

```

玩弄上面的内容，看看变异的效果。注意，如果顶级节点（如`<FunctionDef>`或`<Module>`）被选中进行变异，那么`sum()`将被替换为完全不同的内容。否则，代码仍然与原始的`sum()`代码非常相似。

当然，我们增加变异的数量越多，代码看起来就越不同：

```py
sum_mutated_tree = solver.mutate(sum_str, min_mutations=10, max_mutations=20) 
```

```py
sum_mutated_ast = ast.fix_missing_locations(eval(str(sum_mutated_tree)))
print(ast.unparse(sum_mutated_ast)) 
```

```py
def sum(a, b):
    the_9GuWCvL4cpgyi37K5I_ = a + b
    return the_jXHPe1oqMG

```

通过调整`mutate()`参数，我们可以控制我们的输入应该是多么*常见*和多么*不常见*。

### 变异的有效性如何？

变异现有代码能帮助我们找到错误吗？让我们假设我们有一个有错误的编译器，它为形式为`<elem> * (<elem> + <elem>)`的表达式生成糟糕的代码。`has_distributive_law()`中的代码检查 AST 是否存在此错误：

```py
def has_distributive_law(tree) -> bool:
    for node in walk(tree):  # iterate over all nodes in `tree`
        # print(node)
        if isinstance(node, ast.BinOp):
            if isinstance(node.op, ast.Mult):
                if isinstance(node.right, ast.BinOp):
                    if isinstance(node.right.op, ast.Add):
                        return True

                if isinstance(node.left, ast.BinOp):
                    if isinstance(node.left.op, ast.Add):
                        return True

    return False 
```

为了理解这是如何工作的，AST 的可视化非常有帮助：

```py
show_ast(ast.parse("1 + (2 * 3)")) 
```

<svg xmlns:xlink="http://www.w3.org/1999/xlink" width="342pt" height="332pt" viewBox="0.00 0.00 342.00 332.00"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 328)"><g id="node1" class="node"><title>0</title> <text text-anchor="start" x="113.5" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Expr</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="start" x="109.38" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">BinOp</text></g> <g id="edge1" class="edge"><title>0--1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="start" x="8" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge2" class="edge"><title>1--2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="130" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Add</text></g> <g id="edge4" class="edge"><title>1--4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="start" x="184.38" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">BinOp</text></g> <g id="edge5" class="edge"><title>1--5</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="35" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">1</text></g> <g id="edge3" class="edge"><title>2--3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="start" x="88" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge6" class="edge"><title>5--6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="207" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Mult</text></g> <g id="edge8" class="edge"><title>5--8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="start" x="260" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge9" class="edge"><title>5--9</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="121" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">2</text></g> <g id="edge7" class="edge"><title>6--7</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="293" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">3</text></g> <g id="edge10" class="edge"><title>9--10</title></g></g></svg>

```py
has_distributive_law(ast.parse("1 * (2 + 3)")) 
```

```py
True

```

```py
has_distributive_law(ast.parse("(1 + 2) * 3")) 
```

```py
True

```

```py
has_distributive_law(ast.parse("1 + (2 * 3)")) 
```

```py
False

```

```py
has_distributive_law(ast.parse("def f(a, b):\n return a * (b + 10)")) 
```

```py
True

```

我们需要多少次尝试才能找到触发`has_distributive_law()`函数错误的变异？让我们编写一个函数来计算这个数字。

```py
def how_many_mutations(code: str) -> int:
    solver = ISLaSolver(python_ast_grammar)

    code_ast = ast.parse(code)
    code_ast = ast.fix_missing_locations(code_ast)
    code_ast_str = ast.dump(code_ast)
    code_derivation_tree = solver.parse(code_ast_str)
    mutations = 0
    mutated_code_ast = code_ast

    while not has_distributive_law(mutated_code_ast):
        mutations += 1
        if mutations % 100 == 0:
            print(f'{mutations}...', end='')

        mutated_code_str = str(solver.mutate(code_derivation_tree))
        mutated_code_ast = eval(mutated_code_str)
        # mutated_code_ast = ast.fix_missing_locations(mutated_code_ast)
        # print(ast.dump(mutated_code_ast))
        # print(ast.unparse(mutated_code_ast))

    return mutations 
```

如果我们传递一个已经表现出错误的输入，我们不需要任何变异：

```py
assert how_many_mutations('1 * (2 + 3)') == 0 
```

然而，我们离错误越远，找到它所需的变异（和时间）就越多。值得注意的是，将`2 + 2`变异到具有分配律仍然比变异`2`要快得多。

```py
how_many_mutations('2 + 2')    # <-- Note: this can take a minute 
```

```py
54

```

```py
how_many_mutations('2')  # <-- Note: this can take several minutes 
```

```py
100...200...300...400...500...600...700...800...900...1000...1100...1200...1300...1400...1500...1600...1700...1800...1900...2000...2100...2200...2300...2400...2500...

```

```py
2500

```

我们得出结论，变异现有代码确实是有帮助的，尤其是如果它在语法上*接近触发错误的输入*。如果你想有很好的机会找到错误，请专注于*之前已经触发错误的输入*——有时对这些输入进行简单的变异就已经有助于找到新的错误。

## 进化模糊测试

变异输入的一个有趣应用是使用变异进行*进化模糊测试*。想法是拥有一个输入种群，对它们应用*变异*，并检查它们是否在特定目标上有所改进（通常是代码覆盖率）。那些确实有所改进的输入被保留下来（“适者生存”）作为下一代，并进一步进化。通过重复这个过程足够多次，我们可能会获得覆盖大量代码的输入，从而提高发现错误的机会。

让我们假设我们有一个有缺陷的编译器，它为形式为 `<elem> * (<elem> + <elem>)` 的表达式生成糟糕的代码。上面的 `has_distributive_law()` 函数检查 AST 中是否存在这个错误。

我们的目的是通过模糊测试来检测这个错误。但如果我们简单地从头开始生成随机输入，可能需要很长时间才能生成触发错误的精确操作符组合。

### 获取覆盖率

要让我们的模糊测试器由覆盖率引导，我们首先需要 *测量* 代码覆盖率。我们使用了来自《模糊测试手册》的 [覆盖率模块](https://www.fuzzingbook.org/html/Coverage.html)，它特别易于使用。它简单地使用一个 `with` 子句来从 `with` 子句中的代码获取覆盖率。以下是如何获取我们上面 `has_distributive_law()` 代码的覆盖率的方法：

```py
from Coverage import Coverage 
```

```py
mult_ast = ast.parse("1 * 2")
with Coverage() as cov:
    has_distributive_law(mult_ast) 
```

`coverage()` 方法告诉我们代码中哪些行实际上已经被执行到。这包括来自 `has_distributive_law()` 的行，也包括其他被调用的函数中的行。

```py
cov.coverage() 
```

```py
{('_handle_fromlist', 1217),
 ('_handle_fromlist', 1218),
 ('_handle_fromlist', 1225),
 ('_handle_fromlist', 1229),
 ('_handle_fromlist', 1241),
 ('has_distributive_law', 2),
 ('has_distributive_law', 4),
 ('has_distributive_law', 5),
 ('has_distributive_law', 6),
 ('has_distributive_law', 10),
 ('has_distributive_law', 14),
 ('iter_child_nodes', 272),
 ('iter_child_nodes', 273),
 ('iter_child_nodes', 274),
 ('iter_child_nodes', 275),
 ('iter_child_nodes', 276),
 ('iter_child_nodes', 277),
 ('iter_child_nodes', 278),
 ('iter_fields', 260),
 ('iter_fields', 261),
 ('iter_fields', 262),
 ('walk', 386),
 ('walk', 387),
 ('walk', 388),
 ('walk', 389),
 ('walk', 390),
 ('walk', 391)}

```

哪些行被执行了？通过一点代码检查，我们可以轻松地可视化已覆盖的行：

```py
def show_coverage(cov, fun):
    fun_lines, fun_start = inspect.getsourcelines(fun)
    fun_name = fun.__name__
    coverage = cov.coverage()
    for line in range(len(fun_lines)):
        if (fun_name, line + fun_start) in coverage:
            print('# ', end='')  # covered lines
        else:
            print('  ', end='')  # uncovered lines
        print(line + fun_start, fun_lines[line], end='') 
```

```py
show_coverage(cov, has_distributive_law) 
```

```py
  1 def has_distributive_law(tree) -> bool:
# 2     for node in walk(tree):  # iterate over all nodes in `tree`
  3         # print(node)
# 4         if isinstance(node, ast.BinOp):
# 5             if isinstance(node.op, ast.Mult):
# 6                 if isinstance(node.right, ast.BinOp):
  7                     if isinstance(node.right.op, ast.Add):
  8                         return True
  9 
# 10                 if isinstance(node.left, ast.BinOp):
  11                     if isinstance(node.left.op, ast.Add):
  12                         return True
  13 
# 14     return False

```

在这个列表中，一个 `#` 表示代码已被执行（已覆盖）。我们看到我们的输入 "1 * 2" 满足第 4 行和第 5 行的条件，但不满足后续行的条件。

### 适应度

让我们现在使用覆盖率作为 *适应度函数* 来引导进化。适应度（覆盖率）越高，输入被保留用于进一步进化的可能性就越高。我们的 `ast_fitness()` 函数简单地计算 `has_distributive_law()` 中覆盖的行数。

```py
def ast_fitness(code_ast) -> int:
    with Coverage() as cov:
        has_distributive_law(code_ast)
    lines = set()
    for (name, line) in cov.coverage():
        if name == has_distributive_law.__name__:
            lines.add(line)
    return len(lines) 
```

下面是一些给定输入的适应度：

```py
ast_fitness(ast.parse("1")) 
```

```py
3

```

```py
ast_fitness(ast.parse("1 + 1")) 
```

```py
4

```

```py
ast_fitness(ast.parse("1 * 2")) 
```

```py
6

```

```py
ast_fitness(ast.parse("1 * (2 + 3)")) 
```

```py
6

```

现在，让我们设置一个适应度函数，它接受推导树。本质上，我们的 `tree_fitness()` 函数基于上面的 `ast_fitness()` 函数；然而，我们还添加了一个小的组件 `1 / len(code_str)`，以给较短的输入额外的适应度。否则，我们的输入可能会不断增长，使突变变得低效。

```py
def tree_fitness(tree) -> float:
    code_str = str(tree)
    code_ast = ast.fix_missing_locations(eval(code_str))
    fitness = ast_fitness(code_ast)
    # print(ast.unparse(code_ast), f"\n=> Fitness = {fitness}\n")
    return fitness + 1 / len(code_str) 
```

```py
tree_fitness(sum_tree) 
```

```py
4.002666666666666

```

### 进化输入

让我们现在利用我们的适应度函数来实现一个简单的进化模糊测试算法。我们开始于 *进化* —— 也就是说，通过突变来从一个种群中添加后代。我们的初始种群由一个候选者组成——在我们的例子中，`sum_tree` 反映了上面的 `sum()` 函数。

```py
def initial_population(tree):
    return [ (tree, tree_fitness(tree)) ] 
```

```py
sum_population = initial_population(sum_tree) 
```

```py
len(sum_population) 
```

```py
1

```

我们的 `evolve()` 函数为每个种群成员添加两个新的子代。

```py
OFFSPRING = 2 
```

```py
def evolve(population, min_fitness=-1):
    solver = ISLaSolver(python_ast_grammar)

    for (candidate, _) in list(population):
        for i in range(OFFSPRING):
            child = solver.mutate(candidate, min_mutations=1, max_mutations=1)
            child_fitness = tree_fitness(child)
            if child_fitness > min_fitness:
                population.append((child, child_fitness))
    return population 
```

```py
sum_population = evolve(sum_population)
len(sum_population) 
```

```py
3

```

由于我们可以进化所有这些，我们得到了指数级增长。

```py
sum_population = evolve(sum_population)
len(sum_population) 
```

```py
9

```

```py
sum_population = evolve(sum_population)
len(sum_population) 
```

```py
27

```

```py
sum_population = evolve(sum_population)
len(sum_population) 
```

```py
81

```

```py
sum_population = evolve(sum_population)
len(sum_population) 
```

```py
243

```

### 适者生存

没有个体可以无限扩张并仍然存活。因此，让我们将种群限制在一定的规模内。

```py
POPULATION_SIZE = 100 
```

`select()` 函数实现了适者生存：它将种群限制在最多 `POPULATION_SIZE` 个元素，并按它们的适应度（从高到低）进行排序。适应度低于 `POPULATION_SIZE` 的成员无法存活。

```py
def get_fitness(elem):
    (candidate, fitness) = elem
    return fitness

def select(population):
    population = sorted(population, key=get_fitness, reverse=True)
    population = population[:POPULATION_SIZE]
    return population 
```

我们可以使用以下调用来修剪我们的 `sum_population` 到最适应的成员：

```py
sum_population = select(sum_population)
len(sum_population) 
```

```py
100

```

### 进化

我们现在已经准备好了所有东西：

+   我们有一个 *种群*（比如说，`sum_population`）

+   我们可以通过`evolve()`函数进化种群

+   我们只能让最适应的生存下来（使用`select()`）

让我们在几代中重复这个过程。我们跟踪每次找到新的“最佳”候选者并记录它们。如果我们找到一个触发错误的候选者，我们就停止。请注意，这可能会花费很长时间，并且不一定能产生完美的结果。

在基于搜索的方法中很常见，如果我们经过几代之后还没有找到足够的解决方案，我们就停止并重新开始搜索（这里：`GENERATIONS`）。除此之外，我们会继续搜索，直到我们找到一个解决方案。

```py
GENERATIONS = 100  # Upper bound 
```

```py
trial = 1
found = False

while not found:
    sum_population = initial_population(sum_tree)
    prev_best_fitness = -1

    for generation in range(GENERATIONS):
        sum_population = evolve(sum_population, min_fitness=prev_best_fitness)
        sum_population = select(sum_population)
        best_candidate, best_fitness = sum_population[0]
        if best_fitness > prev_best_fitness:
            print(f"Generation {generation}: found new best candidate (fitness={best_fitness}):")
            best_ast = ast.fix_missing_locations(eval(str(best_candidate)))
            print(ast.unparse(best_ast))
            prev_best_fitness = best_fitness

            if has_distributive_law(best_ast):
                print("Done!")
                found = True
                break

    trial = trial + 1
    print(f"\n\nRestarting; trial #{trial}") 
```

```py
Generation 0: found new best candidate (fitness=4.002666666666666):
def sum(a, b):
    the_sum = a + b
    return the_sum
Generation 1: found new best candidate (fitness=4.0027027027027025):
def sum(a, b):
    the_sum = a + b
    return FE
Generation 4: found new best candidate (fitness=4.002865329512894):
def sum():
    the_sum = a + b
    return the_sum
Generation 5: found new best candidate (fitness=6.00094696969697):
if set()[:] * *set():

    def sum(a, b):
        mc = a + b
        return FE
else:
    M = set()
continue

set().f[set():set()]()
Generation 7: found new best candidate (fitness=7.002364066193853):
def sum(a, b):
    mc = (a + b) * ()
    return FE
Done!

Restarting; trial #2

```

成功！我们找到了一段触发错误的代码。检查分配律的出现。

```py
print(ast.unparse(best_ast)) 
```

```py
def sum(a, b):
    mc = (a + b) * ()
    return FE

```

```py
assert has_distributive_law(best_ast) 
```

你可能会注意到并非所有代码都是触发错误的必要条件。我们可以让我们的进化模糊器运行得更久一些，看看它是否可以被进一步减少，或者使用专门的输入减少技术，例如[Delta Debugging](https://www.fuzzingbook.org/html/Reducer.html)。

### 进化模糊测试的可能性

`distributive_law()`中的错误在没有进化指导的情况下能否被发现 - 也就是说，仅仅通过将一个变异应用到`sum()`上？

当产生一个表达式（`<expr>`）时，我们计算触发概率有多大

+   产生一个二元运算符，并且

+   产生一个`*`，并且

+   产生另一个二元运算符作为子节点，并且

+   产生一个`+`

让我们在我们的语法上进行一些查询来计算概率。

```py
assert '<BinOp>' in python_ast_grammar['<expr>'] 
```

```py
len(python_ast_grammar['<expr>']) 
```

```py
15

```

```py
assert 'Add()' in python_ast_grammar['<operator>']
assert 'Mult()' in python_ast_grammar['<operator>'] 
```

```py
len(python_ast_grammar['<operator>']) 
```

```py
13

```

```py
(len(python_ast_grammar['<expr>'])       # chances of choosing a `BinOp`
* len(python_ast_grammar['<operator>'])  # chances of choosing a `*`
* len(python_ast_grammar['<expr>'])      # chances of choosing a `BinOp` as a child
* len(python_ast_grammar['<operator>'])  # chances of choosing a `+`
/ 2)   # two chances - one for the left child, one for the right 
```

```py
19012.5

```

平均来说，我们需要大约 19000 次（非进化）运行，直到我们得到一个触发分配律的表达式。所以，利用额外的信息（比如覆盖率）来引导变异向目标进化是绝对更好的。

## 经验教训

+   当创建和处理复杂输入，例如程序代码时，

    +   尝试依赖现有的基础设施将输入*解析*成某种*抽象语法*，然后

    +   让你的语法*处理那个抽象语法*而不是具体语法。

+   特别是，程序代码通常在编译或解释之前被转换成*抽象语法树*，你可以（并且应该）利用这种转换。

+   一旦程序代码被转换成抽象语法树（AST），尽管其复杂，但生成、变异和进化都相对容易。

## 背景

编译器测试领域的开创性工作是*Csmith* [[杨等人，2011](https://doi.org/10.1145/1993498.1993532)]，一个 C 程序生成器。Csmith 已被用于彻底测试编译器，如 Clang 或 GCC；除了产生语法正确的代码外，它还旨在实现*语义*正确性以及避免未定义和未指定行为。这是编译器测试领域的人必读的。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。 [最后修改：2024-11-24 21:37:28+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/PythonFuzzer.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[测试编译器](https://www.fuzzingbook.org/html/PythonFuzzer.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[模糊测试书籍](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/PythonFuzzer.html`](https://www.fuzzingbook.org/html/PythonFuzzer.html). Retrieved 2024-11-24 21:37:28+01:00.

```py
@incollection{fuzzingbook2024:PythonFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Testing Compilers},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/PythonFuzzer.html}},
    note = {Retrieved 2024-11-24 21:37:28+01:00},
    url = {https://www.fuzzingbook.org/html/PythonFuzzer.html},
    urldate = {2024-11-24 21:37:28+01:00}
}

```
