# 使用生成器进行模糊测试

> 原文：[`www.fuzzingbook.org/html/GeneratorGrammarFuzzer.html`](http://www.fuzzingbook.org/html/GeneratorGrammarFuzzer.html)

在本章中，我们展示了如何通过 *函数* 扩展语法——这些函数在语法扩展期间执行，可以生成、检查或更改生成的元素。向语法中添加函数允许进行非常灵活的测试生成，将语法生成和编程的最佳之处结合起来。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('6Z35ChunpLY') 
```

**先决条件**

+   由于本章深入探讨了 高效语法模糊测试章节 中讨论的技术，因此建议对技术有良好的理解。

## 概述

要使用本章提供的代码（Importing.html），请编写

```py
>>> from fuzzingbook.GeneratorGrammarFuzzer import <identifier> 
```

然后利用以下功能。

本章介绍了将 *函数* 附接到单个生成规则的能力：

+   一个 `pre` 函数在扩展之前执行。其结果（通常是字符串）可以 *替换* 实际的扩展。

+   一个 `post` 函数在扩展之后执行。如果它返回一个字符串，则该字符串替换扩展；如果它返回 `False`，则触发新的扩展。

两个函数都可以返回 `None` 以完全不干扰语法的生成。

要将函数 `F` 附接到语法中单个扩展 `S` 上，将 `S` 替换为

```py
(S, opts(pre=F))   # Set a function to be executed before expansion 
```

或

```py
(S, opts(post=F))  # Set a function to be executed after expansion 
```

这里有一个例子，要从程序给出的列表中获取一个区号，我们可以编写：

```py
>>> from Grammars import US_PHONE_GRAMMAR, extend_grammar, opts
>>> def pick_area_code():
>>>     return random.choice(['555', '554', '553'])
>>> PICKED_US_PHONE_GRAMMAR = extend_grammar(US_PHONE_GRAMMAR,
>>> {
>>>     "<area>": [("<lead-digit><digit><digit>", opts(pre=pick_area_code))]
>>> }) 
```

`GeneratorGrammarFuzzer` 将提取并解释这些选项。以下是一个示例：

```py
>>> picked_us_phone_fuzzer = GeneratorGrammarFuzzer(PICKED_US_PHONE_GRAMMAR)
>>> [picked_us_phone_fuzzer.fuzz() for i in range(5)]
['(554)732-6097',
 '(555)469-0662',
 '(553)671-5358',
 '(555)686-8011',
 '(554)453-4067'] 
```

如您所见，现在所有的区号都来自 `pick_area_code()`。这样的定义允许将程序代码（如 `pick_area_code()`）与语法紧密关联。

`PGGCFuzzer` 类整合了来自 `GrammarFuzzer` 类 及其 基于覆盖、基于概率 和 基于生成器 的所有特性。

<svg width="566pt" height="877pt" viewBox="0.00 0.00 565.75 876.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 872.75)"><g id="node1" class="node"><title>PGGCFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class PGGCFuzzer:

唯一支持所有 fuzzingbook 功能的基于语法的模糊测试器是"><text text-anchor="start" x="162.12" y="-21.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">PGGCFuzzer</text></a></g></g> <g id="node2" class="node"><title>ProbabilisticGeneratorGrammarCoverageFuzzer</title> <g id="a_node2"><a xlink:href="#" xlink:title="class ProbabilisticGeneratorGrammarCoverageFuzzer:

结合 `GeneratorGrammarFuzzer` 的特性

以及 `ProbabilisticGrammarCoverageFuzzer`。《ProbabilisticGeneratorGrammarCoverageFuzzer》</text></a></g> <g id="a_node2_0"><a xlink:href="#" xlink:title="ProbabilisticGeneratorGrammarCoverageFuzzer"><g id="a_node2_1"><a xlink:href="#" xlink:title="__init__(self, grammar: Dict[str, List[Expansion]], *, replacement_attempts: int = 10, **kwargs) -> None:

构造函数。

`replacement_attempts` - see `GeneratorGrammarFuzzer` constructor.

所有其他关键字参数都传递给 `ProbabilisticGrammarFuzzer`。《__init__()`</text></a></g> <g id="a_node2_2"><a xlink:href="#" xlink:title="fuzz_tree(self) -> DerivationTree:

从语法中生成一个推导树。《fuzz_tree()`</text></a></g> <g id="a_node2_3"><a xlink:href="#" xlink:title="add_tree_coverage(self, tree: DerivationTree) -> None"><text text-anchor="start" x="143" y="-118.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">添加树覆盖范围()</text></a></g> <g id="a_node2_4"><a xlink:href="#" xlink:title="restart_expansion(self) -> None"><text text-anchor="start" x="143" y="-106.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">重新启动扩展()</text></a></g> <g id="a_node2_5"><a xlink:href="#" xlink:title="supported_opts(self) -> Set[str]:

支持的选项集合。应在子类中重载。《supported_opts()`</text></a></g></a></g></g></g> <g id="edge1" class="edge"><title>PGGCFuzzer->ProbabilisticGeneratorGrammarCoverageFuzzer</title></g> <g id="node3" class="node"><title>GeneratorGrammarFuzzer</title> <g id="a_node3"><a xlink:href="#" xlink:title="class GeneratorGrammarFuzzer:

高效地从语法中生成字符串，使用推导树。《GeneratorGrammarFuzzer》<text text-anchor="start" x="11.38" y="-562.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">生成语法模糊器</text> <g id="a_node3_6"><a xlink:href="#" xlink:title="GeneratorGrammarFuzzer"><g id="a_node3_7"><a xlink:href="#" xlink:title="__init__(self, grammar: Dict[str, List[Expansion]], replacement_attempts: int = 10, **kwargs) -> None:

从 `grammar` 中生成字符串，从 `start_symbol` 开始。《Produce strings from `grammar`, starting with `start_symbol`.</text></a></g></g></g></g></g>

如果提供了 `min_nonterminals` 或 `max_nonterminals`，则使用它们作为限制

用于生成非终结符的数量。

如果设置了 `disp`，则显示中间推导树。

如果设置了`log`，则将中间步骤作为文本显示在标准输出上。《`__init__()`</a></g> <g id="a_node3_8"><a xlink:href="#" xlink:title="fuzz_tree(self) -> DerivationTree:

从语法生成推导树。"><`fuzz_tree()`</a></g> <g id="a_node3_9"><a xlink:href="#" xlink:title="apply_result(self, result: Any, children: List[DerivationTree]) -> List[DerivationTree]"><`apply_result()`</a></g> <g id="a_node3_10"><a xlink:href="#" xlink:title="choose_tree_expansion(self, tree: DerivationTree, expandable_children: List[DerivationTree]) -> int:

返回`expandable_children`中子树的索引

要选择的用于展开的树。默认为随机。"><`choose_tree_expansion()`</a></g> <g id="a_node3_11"><a xlink:href="#" xlink:title="eval_function(self, tree, function)"><`eval_function()`</a></g> <g id="a_node3_12"><a xlink:href="#" xlink:title="expand_tree_once(self, tree: DerivationTree) -> DerivationTree:

在树中选择一个未展开的符号；展开它。

可在子类中重载。"><`expand_tree_once()`</a></g> <g id="a_node3_13"><a xlink:href="#" xlink:title="find_expansion(self, tree)"><`find_expansion()`</a></g> <g id="a_node3_14"><a xlink:href="#" xlink:title="process_chosen_children(self, children: List[DerivationTree], expansion: Expansion) -> List[DerivationTree]:

在选择后处理儿童。默认情况下，不执行任何操作。&nbsp;`process_chosen_children()`</a></g> <g id="a_node3_15"><a xlink:href="#" xlink:title="reset_generators(self) -> None"><text text-anchor="start" x="8" y="-437.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">reset_generators()</text></a></g> <g id="a_node3_16"><a xlink:href="#" xlink:title="restart_expansion(self) -> None"><text text-anchor="start" x="8" y="-425.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">restart_expansion()</text></a></g> <g id="a_node3_17"><a xlink:href="#" xlink:title="run_generator(self, expansion: Expansion, function: Callable) -> Iterator"><text text-anchor="start" x="8" y="-411.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">run_generator()</text></a></g> <g id="a_node3_18"><a xlink:href="#" xlink:title="run_post_functions(self, tree: DerivationTree, depth: Union[int, float] = inf) -> Tuple[bool, Optional[List[DerivationTree]]]"><text text-anchor="start" x="8" y="-399" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">run_post_functions()</text></a></g> <g id="a_node3_19"><a xlink:href="#" xlink:title="run_post_functions_locally(self, new_tree: DerivationTree) -> DerivationTree"><text text-anchor="start" x="8" y="-386.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">run_post_functions_locally()</text></a></g> <g id="a_node3_20"><a xlink:href="#" xlink:title="supported_opts(self) -> Set[str]:

支持的选项集合。应在子类中重载。"><text text-anchor="start" x="8" y="-374.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">supported_opts()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>ProbabilisticGeneratorGrammarCoverageFuzzer->GeneratorGrammarFuzzer</title></g> <g id="node6" class="node"><title>ProbabilisticGrammarCoverageFuzzer</title> <g id="a_node6"><a xlink:href="ProbabilisticGrammarFuzzer.html" xlink:title="class ProbabilisticGrammarCoverageFuzzer:

从语法生成，旨在覆盖所有展开。"><text text-anchor="start" x="192.38" y="-234.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">ProbabilisticGrammarCoverageFuzzer</text></a></g></g> <g id="edge5" class="edge"><title>ProbabilisticGeneratorGrammarCoverageFuzzer->ProbabilisticGrammarCoverageFuzzer</title></g> <g id="node4" class="node"><title>GrammarFuzzer</title> <g id="a_node4"><a xlink:href="GrammarFuzzer.html" xlink:title="class GrammarFuzzer:

从语法中高效地生成字符串，使用推导树。"><text text-anchor="start" x="184.12" y="-755.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarFuzzer</text> <g id="a_node4_21"><a xlink:href="#" xlink:title="GrammarFuzzer"><g id="a_node4_22"><a xlink:href="GrammarFuzzer.html" xlink:title="__init__(self, grammar: Dict[str, List[Expansion]], start_symbol: str = '<start>', min_nonterminals: int = 0, max_nonterminals: int = 10, disp: bool = False, log: Union[bool, int] = False) -> None:

从`grammar`中生成字符串，从`start_symbol`开始。

如果提供了`min_nonterminals`或`max_nonterminals`，则使用它们作为限制

生成非终结符的数量。

如果`disp`被设置，显示中间的推导树。

如果`log`被设置，将中间步骤作为文本输出到标准输出。"><text text-anchor="start" x="201" y="-733.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node4_23"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz(self) -> str:

从语法中生成一个字符串。"><text text-anchor="start" x="201" y="-720.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node4_24"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz_tree(self) -> DerivationTree:

从语法中生成一个推导树。"><text text-anchor="start" x="201" y="-707.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz_tree()</text></a></g></a></g></a></g></g> <g id="edge3" class="edge"><title>GeneratorGrammarFuzzer->GrammarFuzzer</title></g> <g id="node5" class="node"><title>Fuzzer</title> <g id="a_node5"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊测试器的基类。"><text text-anchor="start" x="213.38" y="-851.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Fuzzer</text> <g id="a_node5_25"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node5_26"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

使用模糊输入运行`runner`"><text text-anchor="start" x="216" y="-829.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">run()</text></a></g> <g id="a_node5_27"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

使用模糊输入运行`runner`，`trials`次。《runs()</text></a></g></a></g></a></g></g> <g id="edge4" class="edge"><title>GrammarFuzzer->Fuzzer</title></g> <g id="node7" class="node"><title>GrammarCoverageFuzzer</title> <g id="a_node7"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="class GrammarCoverageFuzzer:

从语法中生成，旨在覆盖所有扩展。《GrammarCoverageFuzzer</text></a></g></g> <g id="edge6" class="edge"><title>ProbabilisticGrammarCoverageFuzzer->GrammarCoverageFuzzer</title></g> <g id="node10" class="node"><title>ProbabilisticGrammarFuzzer</title> <g id="a_node10"><a xlink:href="ProbabilisticGrammarFuzzer.html" xlink:title="class ProbabilisticGrammarFuzzer:

基于语法的模糊测试器，尊重语法中的概率。《ProbabilisticGrammarFuzzer</text></a></g></g> <g id="edge10" class="edge"><title>ProbabilisticGrammarCoverageFuzzer->ProbabilisticGrammarFuzzer</title></g> <g id="node8" class="node"><title>SimpleGrammarCoverageFuzzer</title> <g id="a_node8"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="class SimpleGrammarCoverageFuzzer:

在选择扩展时，优先选择未被覆盖的扩展。《SimpleGrammarCoverageFuzzer</text></a></g></g> <g id="edge7" class="edge"><title>GrammarCoverageFuzzer->SimpleGrammarCoverageFuzzer</title></g> <g id="node9" class="node"><title>TrackingGrammarCoverageFuzzer</title> <g id="a_node9"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="class TrackingGrammarCoverageFuzzer:

在生成过程中跟踪语法覆盖。《TrackingGrammarCoverageFuzzer</text> <g id="a_node9_28"><a xlink:href="#" xlink:title="TrackingGrammarCoverageFuzzer"><g id="a_node9_29"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="__init__(self, *args, **kwargs) -> None:

从`grammar`生成字符串，从`start_symbol`开始。《译文：》

如果提供了`min_nonterminals`或`max_nonterminals`，则使用它们作为限制。

对于生成的非终结符数量。

如果设置了`disp`，则显示中间推导树。

如果设置了`log`，则将中间步骤作为文本显示在标准输出上。《__init__()`</a></g></a></g></a></g></g> <g id="edge8" class="edge"><title>SimpleGrammarCoverageFuzzer->TrackingGrammarCoverageFuzzer</title></g> <g id="edge9" class="edge"><title>TrackingGrammarCoverageFuzzer->GrammarFuzzer</title></g> <g id="edge11" class="edge"><title>ProbabilisticGrammarFuzzer->GrammarFuzzer</title></g> <g id="node11" class="node"><title>图例</title> <text text-anchor="start" x="264.38" y="-40.5" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="264.38" y="-30.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="270.38" y="-30.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="264.38" y="-20.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="270.38" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="264.38" y="-10.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="270.38" y="-10.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="264.38" y="-1.45" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 示例：测试信用卡系统

假设你在一个购物系统中工作，该系统除了其他几个功能外，还允许客户使用信用卡支付。你的任务是测试支付功能。

为了简化问题，我们将假设我们只需要两份数据——一个 16 位的信用卡号和要收费的金额。这两份数据都可以通过语法轻松生成，如下所示：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import Callable, Set, List, Dict, Optional, Iterator, Any, Union, Tuple, cast 
```

```py
from Fuzzer import Fuzzer 
```

```py
from Grammars import EXPR_GRAMMAR, is_valid_grammar, is_nonterminal, extend_grammar
from Grammars import opts, exp_opt, exp_string, crange, Grammar, Expansion 
```

```py
from GrammarFuzzer import DerivationTree 
```

```py
CHARGE_GRAMMAR: Grammar = {
    "<start>": ["Charge <amount> to my credit card <credit-card-number>"],
    "<amount>": ["$<float>"],
    "<float>": ["<integer>.<digit><digit>"],
    "<integer>": ["<digit>", "<integer><digit>"],
    "<digit>": crange('0', '9'),

    "<credit-card-number>": ["<digits>"],
    "<digits>": ["<digit-block><digit-block><digit-block><digit-block>"],
    "<digit-block>": ["<digit><digit><digit><digit>"],
} 
```

```py
assert is_valid_grammar(CHARGE_GRAMMAR) 
```

所有这些都工作得很好——我们可以生成任意数量的信用卡号：

```py
from GrammarFuzzer import GrammarFuzzer, all_terminals 
```

```py
g = GrammarFuzzer(CHARGE_GRAMMAR)
[g.fuzz() for i in range(5)] 
```

```py
['Charge $9.40 to my credit card 7166898575638313',
 'Charge $8.79 to my credit card 6845418694643271',
 'Charge $5.64 to my credit card 6655894657077388',
 'Charge $0.60 to my credit card 2596728464872261',
 'Charge $8.90 to my credit card 2363769342732142']

```

然而，当实际上用这些数据测试我们的系统时，我们发现存在两个问题：

1.  我们想测试被收费的*特定*金额——例如，超过信用卡额度的金额。

1.  我们发现，10 个信用卡号中有 9 个因为校验和不正确而被拒绝。如果我们想测试信用卡号的拒绝，这没问题——但如果我们想测试处理收费的实际功能，我们需要*有效*的号码。

我们可以忽略这些问题；毕竟，最终只是一件时间问题，直到生成大量有效数字。至于第一个问题，我们也可以通过适当地更改语法来解决它——比如说，只生成至少有六个前导数字的充电。然而，将此推广到任意范围的值将很麻烦。

然而，第二个问题，信用卡号码的校验和，却更为复杂——至少在语法方面，一个复杂的算术运算，如校验和，不能仅用语法表示——至少不能在我们这里使用的*上下文无关语法*中。原则上，*可以*在一个*上下文相关语法*中这样做，但这将毫无乐趣。我们想要的是一个机制，允许我们将*程序性计算*附加到我们的语法中，将两者的优点结合起来。

## 将函数附加到扩展

本章的关键思想是*扩展*语法，以便可以将*Python 函数*附加到单个扩展。这些函数可以执行

1.  *在扩展之前*，*替换*要扩展的元素为计算值；或者

1.  *扩展后*，*检查*生成的元素，并可能替换它们。

在这两种情况下，函数都是使用在语法章节中引入的`opts()`扩展机制指定的。因此，它们与符号`s`的特定扩展$e$相关联。

### 在扩展之前调用的函数

使用`pre`选项定义的函数在将`s`扩展为$e$之前被调用。它的值*替换*要生成的扩展$e$。为了生成上面信用卡示例的值，我们可以定义一个*预扩展*生成函数

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
def high_charge() -> float:
    return random.randint(10000000, 90000000) / 100.0 
```

使用`opts()`，我们可以将此函数附加到语法：

```py
CHARGE_GRAMMAR.update({
    "<float>": [("<integer>.<digit><digit>", opts(pre=high_charge))],
}) 
```

目的是，每当`<float>`被扩展时，函数`high_charge`就会被调用以生成`<float>`的值。（在语法中，实际的扩展仍然存在，对于忽略函数的模糊器，如`GrammarFuzzer`）。

由于与语法相关的函数通常非常简单，我们还可以使用*lambda*表达式*内联*它们。*lambda 表达式*用于*匿名*函数，这些函数的范围和功能有限。以下是一个示例：

```py
def apply_twice(function, x):
    return function(function(x)) 
```

```py
apply_twice(lambda x: x * x, 2) 
```

```py
16

```

在这里，我们不必给要应用的`function`两次命名（比如，`square()`）；相反，我们在调用时内联应用它。

使用`lambda`，我们的语法看起来是这样的：

```py
CHARGE_GRAMMAR.update({
    "<float>": [("<integer>.<digit><digit>",
                 opts(pre=lambda: random.randint(10000000, 90000000) / 100.0))]
}) 
```

### 在扩展之后调用的函数

使用`post`选项定义的函数在将`s`扩展为$e$之后被调用，并将$e$中符号的扩展值作为参数传递。扩展后的函数可以以两种方式服务：

1.  它可以作为扩展值的*约束*或*过滤器*，如果扩展有效则返回`True`，否则返回`False`；如果返回`False`，则尝试另一个扩展。

1.  它还可以作为*修复*，返回一个字符串值；就像预扩展函数一样，返回的值替换了扩展。

对于我们的信用卡示例，我们可以选择两种方式。如果我们有一个函数`check_credit_card(s)`，它对于有效的数字 `s` 返回 `True`，对于无效的数字返回 `False`，我们将选择第一种选项：

```py
CHARGE_GRAMMAR.update({
    "<credit-card-number>": [("<digits>", opts(post=lambda digits: check_credit_card(digits)))]
}) 
```

使用这样的过滤器，只能生成有效的信用卡。平均而言，每次`check_credit_card()`满足条件时，仍然需要 10 次尝试。

如果我们有一个函数`fix_credit_card(s)`，它改变数字以使校验和有效并返回“修复”后的数字，我们可以使用这个函数代替：

```py
CHARGE_GRAMMAR.update({
    "<credit-card-number>": [("<digits>", opts(post=lambda digits: fix_credit_card(digits)))]
}) 
```

在这里，每个数字只生成一次然后修复。这非常高效。

用于信用卡的校验和函数是[Luhn 算法](https://en.wikipedia.org/wiki/Luhn_algorithm)，这是一个简单而有效的公式。

```py
def luhn_checksum(s: str) -> int:
  """Compute Luhn's check digit over a string of digits"""
    LUHN_ODD_LOOKUP = (0, 2, 4, 6, 8, 1, 3, 5, 7,
                       9)  # sum_of_digits (index * 2)

    evens = sum(int(p) for p in s[-1::-2])
    odds = sum(LUHN_ODD_LOOKUP[int(p)] for p in s[-2::-2])
    return (evens + odds) % 10 
```

```py
def valid_luhn_checksum(s: str) -> bool:
  """Check whether the last digit is Luhn's checksum over the earlier digits"""
    return luhn_checksum(s[:-1]) == int(s[-1]) 
```

```py
def fix_luhn_checksum(s: str) -> str:
  """Return the given string of digits, with a fixed check digit"""
    return s[:-1] + repr(luhn_checksum(s[:-1])) 
```

```py
luhn_checksum("123") 
```

```py
8

```

```py
fix_luhn_checksum("123x") 
```

```py
'1238'

```

我们可以在我们的信用卡语法中使用这些函数：

```py
check_credit_card: Callable[[str], bool] = valid_luhn_checksum
fix_credit_card: Callable[[str], str] = fix_luhn_checksum 
```

```py
fix_credit_card("1234567890123456") 
```

```py
'1234567890123458'

```

## 用于整合约束的类

虽然指定函数很容易，但我们的语法 fuzzer 将简单地忽略它们，就像它忽略所有扩展一样。尽管如此，它将发出警告：

```py
g = GrammarFuzzer(CHARGE_GRAMMAR)
g.fuzz() 
```

```py
'Charge $4.05 to my credit card 0637034038177393'

```

我们需要定义一个特殊的 fuzzer，它实际上调用给定的`pre`和`post`函数并根据其行为。我们将其命名为`GeneratorGrammarFuzzer`：

```py
class GeneratorGrammarFuzzer(GrammarFuzzer):
    def supported_opts(self) -> Set[str]:
        return super().supported_opts() | {"pre", "post", "order"} 
```

我们定义自定义函数来访问`pre`和`post`选项：

```py
def exp_pre_expansion_function(expansion: Expansion) -> Optional[Callable]:
  """Return the specified pre-expansion function, or None if unspecified"""
    return exp_opt(expansion, 'pre') 
```

```py
def exp_post_expansion_function(expansion: Expansion) -> Optional[Callable]:
  """Return the specified post-expansion function, or None if unspecified"""
    return exp_opt(expansion, 'post') 
```

`order`属性将在本章后面使用。

## 在扩展之前生成元素

我们的首要任务是实现预扩展函数——即在扩展之前调用的函数，用于替换要扩展的值。为此，我们挂钩到`process_chosen_children()`方法，该方法在扩展之前获取选定的子项。我们将其设置为调用给定的`pre`函数并将结果应用于子项，可能替换它们。

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def process_chosen_children(self, children: List[DerivationTree],
                                expansion: Expansion) -> List[DerivationTree]:
        function = exp_pre_expansion_function(expansion)
        if function is None:
            return children

        assert callable(function)
        if inspect.isgeneratorfunction(function):
            # See "generators", below
            result = self.run_generator(expansion, function)
        else:
            result = function()

        if self.log:
            print(repr(function) + "()", "=", repr(result))
        return self.apply_result(result, children)

    def run_generator(self, expansion: Expansion, function: Callable):
        ... 
```

`apply_result()`方法从预扩展函数中获取结果并将其应用于子项。确切的效果取决于结果类型：

+   一个*字符串* $s$ 将整个扩展替换为 $s$。

+   一个*列表* $[x_1, x_2, \dots, x_n]$ 对于每个不是`None`的 $x_i$，将第 $i$ 个符号替换为 $x_i$。将`None`指定为列表元素 $x_i$ 有助于保持该元素不变。如果 $x_i$ 不是一个字符串，它将被转换为字符串。

+   `None`的值将被忽略。如果只想在扩展时调用函数而不影响扩展的字符串，这很有用。

+   *布尔值*将被忽略。这对于下面讨论的后续扩展函数很有用。

+   所有*其他类型*都转换为字符串，替换整个扩展。

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def apply_result(self, result: Any,
                     children: List[DerivationTree]) -> List[DerivationTree]:
        if isinstance(result, str):
            children = [(result, [])]
        elif isinstance(result, list):
            symbol_indexes = [i for i, c in enumerate(children)
                              if is_nonterminal(c[0])]

            for index, value in enumerate(result):
                if value is not None:
                    child_index = symbol_indexes[index]
                    if not isinstance(value, str):
                        value = repr(value)
                    if self.log:
                        print(
                            "Replacing", all_terminals(
                                children[child_index]), "by", value)

                    # children[child_index] = (value, [])
                    child_symbol, _ = children[child_index]
                    children[child_index] = (child_symbol, [(value, [])])
        elif result is None:
            pass
        elif isinstance(result, bool):
            pass
        else:
            if self.log:
                print("Replacing", "".join(
                    [all_terminals(c) for c in children]), "by", result)

            children = [(repr(result), [])]

        return children 
```

### 示例：数值范围

在上述扩展之后，我们完全支持预扩展函数。使用增强的`CHARGE_GRAMMAR`，我们发现实际上使用了预扩展`lambda`函数：

```py
charge_fuzzer = GeneratorGrammarFuzzer(CHARGE_GRAMMAR)
charge_fuzzer.fuzz() 
```

```py
'Charge $439383.87 to my credit card 2433506594138520'

```

日志显示，当调用预扩展函数时发生了更多细节。我们看到扩展 `<integer>.<digit><digit>` 被直接替换为计算值：

```py
amount_fuzzer = GeneratorGrammarFuzzer(
    CHARGE_GRAMMAR, start_symbol="<amount>", log=True)
amount_fuzzer.fuzz() 
```

```py
Tree: <amount>
Expanding <amount> randomly
Tree: $<float>
Expanding <float> randomly
<function <lambda> at 0x1109f2ac0>() = 382087.72
Replacing <integer>.<digit><digit> by 382087.72
Tree: $382087.72
'$382087.72'

```

```py
'$382087.72'

```

### 示例：更多数值范围

我们还可以在其他上下文中使用这样的预扩展函数。假设我们想要生成每个数字都在 100 到 200 之间的算术表达式。我们可以相应地扩展 `EXPR_GRAMMAR`：

```py
expr_100_200_grammar = extend_grammar(EXPR_GRAMMAR,
                                      {
                                          "<factor>": [
                                              "+<factor>", "-<factor>", "(<expr>)",

                                              # Generate only the integer part with a function;
                                              # the fractional part comes from
                                              # the grammar
                                              ("<integer>.<integer>", opts(
                                                  pre=lambda: [random.randint(100, 200), None])),

                                              # Generate the entire integer
                                              # from the function
                                              ("<integer>", opts(
                                                  pre=lambda: random.randint(100, 200))),
                                          ],
                                      }
                                      ) 
```

```py
expr_100_200_fuzzer = GeneratorGrammarFuzzer(expr_100_200_grammar)
expr_100_200_fuzzer.fuzz() 
```

```py
'(108.6 / 155 + 177) / 118 * 120 * 107 + 151 + 195 / -200 - 150 * 188 / 147 + 112'

```

### 支持 Python 生成器

Python 语言有自己的生成器函数概念，我们当然也希望支持它。Python 中的 *生成器函数* 是一个返回所谓的 *迭代器对象* 的函数，我们可以逐个迭代它。

在 Python 中创建生成器函数时，定义一个普通函数，使用 `yield` 语句而不是 `return` 语句。虽然 `return` 语句会终止函数，但 `yield` 语句会暂停其执行，保存所有状态，以便稍后在下一次连续调用中恢复。

这里是一个生成器函数的示例。当第一次调用时，`iterate()` 产生值 1，然后是 2、3，依此类推：

```py
def iterate():
    t = 0
    while True:
        t = t + 1
        yield t 
```

我们可以在循环中使用 `iterate`，就像 `range()` 函数（它也是一个生成器函数）一样：

```py
for i in iterate():
    if i > 10:
        break
    print(i, end=" ") 
```

```py
1 2 3 4 5 6 7 8 9 10 

```

我们还可以将 `iterate()` 作为预扩展生成器函数使用，确保它将创建一个接一个的连续整数：

```py
iterate_grammar = extend_grammar(EXPR_GRAMMAR,
                                 {
                                     "<factor>": [
                                         "+<factor>", "-<factor>", "(<expr>)",
                                         # "<integer>.<integer>",

                                         # Generate one integer after another
                                         # from the function
                                         ("<integer>", opts(pre=iterate)),
                                     ],
                                 }) 
```

为了支持生成器，我们上面的 `process_chosen_children()` 方法检查一个函数是否是生成器；如果是，它调用 `run_generator()` 方法。当 `run_generator()` 在 `fuzz_tree()`（或 `fuzz()`）调用期间第一次看到该函数时，它调用该函数以创建一个生成器对象；这被保存在 `generators` 属性中，然后调用。后续调用直接转到生成器，保留状态。

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def fuzz_tree(self) -> DerivationTree:
        self.reset_generators()
        return super().fuzz_tree()

    def reset_generators(self) -> None:
        self.generators: Dict[str, Iterator] = {}

    def run_generator(self, expansion: Expansion,
                      function: Callable) -> Iterator:
        key = repr((expansion, function))
        if key not in self.generators:
            self.generators[key] = function()
        generator = self.generators[key]
        return next(generator) 
```

这是否可行？让我们在我们的语法上运行我们的 fuzzer，使用 `iterator()`：

```py
iterate_fuzzer = GeneratorGrammarFuzzer(iterate_grammar)
iterate_fuzzer.fuzz() 
```

```py
'1 * ++++3 / ---+4 - 2 * +--6 / 7 * 10 - (9 - 11) - 5 + (13) * 14 + 8 + 12'

```

我们看到该表达式包含所有以 1 开头的整数。

除了指定我们自己的 Python 生成器函数，如 `iterate()`，我们还可以使用内置的 Python 生成器之一，如 `range()`。这也会生成以 1 开头的整数：

```py
iterate_grammar = extend_grammar(EXPR_GRAMMAR,
                                 {
                                     "<factor>": [
                                         "+<factor>", "-<factor>", "(<expr>)",
                                         ("<integer>", opts(pre=range(1, 1000))),
                                     ],
                                 }) 
```

还可以使用 Python 列表推导式，通过在括号中添加它们的生成器函数：

```py
iterate_grammar = extend_grammar(EXPR_GRAMMAR,
                                 {
                                     "<factor>": [
                                         "+<factor>", "-<factor>", "(<expr>)",
                                         ("<integer>", opts(
                                             pre=(x for x in range(1, 1000)))),
                                     ],
                                 }) 
```

注意，上述两种语法实际上会导致当创建超过 1,000 个整数时，fuzzer 会引发异常，但您会发现修复这个问题非常容易。

最后，`yield` 实际上是一个表达式，而不是一个语句，因此也可以有一个 `lambda` 表达式 `yield` 一个值。如果您发现这个用法有合理的用途，请告诉我们。

## 扩展后的元素检查和修复

现在，让我们转向我们要支持的第二个函数集——即，后扩展函数。使用它们的最简单方法是，在生成整个树之后运行它们，就像 `pre` 函数一样处理替换。然而，如果其中一个返回 `False`，我们将重新开始。

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def fuzz_tree(self) -> DerivationTree:
        while True:
            tree = super().fuzz_tree()
            (symbol, children) = tree
            result, new_children = self.run_post_functions(tree)
            if not isinstance(result, bool) or result:
                return (symbol, new_children)
            self.restart_expansion()

    def restart_expansion(self) -> None:
        # To be overloaded in subclasses
        self.reset_generators() 
```

方法`run_post_functions()`递归地应用于推导树的所有节点。对于每个节点，它确定应用的扩展，然后运行与该扩展关联的函数。

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    # Return True iff all constraints of grammar are satisfied in TREE
    def run_post_functions(self, tree: DerivationTree,
                           depth: Union[int, float] = float("inf")) \
                               -> Tuple[bool, Optional[List[DerivationTree]]]:
        symbol: str = tree[0]
        children: List[DerivationTree] = cast(List[DerivationTree], tree[1])

        if children == []:
            return True, children  # Terminal symbol

        try:
            expansion = self.find_expansion(tree)
        except KeyError:
            # Expansion (no longer) found - ignore
            return True, children

        result = True
        function = exp_post_expansion_function(expansion)
        if function is not None:
            result = self.eval_function(tree, function)
            if isinstance(result, bool) and not result:
                if self.log:
                    print(
                        all_terminals(tree),
                        "did not satisfy",
                        symbol,
                        "constraint")
                return False, children

            children = self.apply_result(result, children)

        if depth > 0:
            for c in children:
                result, _ = self.run_post_functions(c, depth - 1)
                if isinstance(result, bool) and not result:
                    return False, children

        return result, children 
```

辅助方法`find_expansion()`接受一个子树`tree`，并确定应用于创建`tree`中子节点的语法。

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def find_expansion(self, tree):
        symbol, children = tree

        applied_expansion = \
            "".join([child_symbol for child_symbol, _ in children])

        for expansion in self.grammar[symbol]:
            if exp_string(expansion) == applied_expansion:
                return expansion

        raise KeyError(
            symbol +
            ": did not find expansion " +
            repr(applied_expansion)) 
```

方法`eval_function()`是负责实际调用表达式后函数的方法。它创建一个包含所有非终结子节点扩展的参数列表——也就是说，语法扩展中的每个符号都有一个参数。然后调用给定的函数。

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def eval_function(self, tree, function):
        symbol, children = tree

        assert callable(function)

        args = []
        for (symbol, exp) in children:
            if exp != [] and exp is not None:
                symbol_value = all_terminals((symbol, exp))
                args.append(symbol_value)

        result = function(*args)
        if self.log:
            print(repr(function) + repr(tuple(args)), "=", repr(result))

        return result 
```

注意，与预扩展函数不同，表达式后的函数通常处理已经产生的值，所以我们在这里不支持 Python 生成器。

### 示例：负表达式

让我们在一个示例上尝试这些表达式后的函数。假设我们只想产生评估结果为负数的算术表达式——例如，将这些生成的表达式输入到编译器或其他外部系统中。使用`pre`函数来构造性地完成这项任务将非常困难。相反，我们可以定义一个约束条件来检查这个特定的属性，使用 Python 的`eval()`函数。

Python 的`eval()`函数接受一个字符串，并按照 Python 规则对其进行评估。由于我们生成的表达式语法略不同于 Python，并且 Python 在评估过程中可能会引发算术异常，我们需要一种优雅地处理这些错误的方法。函数`eval_with_exception()`封装了`eval()`；如果在评估过程中发生异常，它将返回 False——这会导致生产算法产生另一个值。

```py
from ExpectError import ExpectError 
```

```py
def eval_with_exception(s):
    # Use "mute=True" to suppress all messages
    with ExpectError(print_traceback=False):
        return eval(s)
    return False 
```

```py
negative_expr_grammar = extend_grammar(EXPR_GRAMMAR,
                                       {
                                           "<start>": [("<expr>", opts(post=lambda s: eval_with_exception(s) < 0))]
                                       }
                                       )

assert is_valid_grammar(negative_expr_grammar) 
```

```py
negative_expr_fuzzer = GeneratorGrammarFuzzer(negative_expr_grammar)
expr = negative_expr_fuzzer.fuzz()
expr 
```

```py
ZeroDivisionError: division by zero (expected)

```

```py
'(8.9 / 6 * 4 - 0.2 + -7 - 7 - 8 * 6) * 7 * 15.55 - -945.9'

```

结果确实是负数：

```py
eval(expr) 
```

```py
-5178.726666666667

```

### 示例：匹配 XML 标签

表达式后的函数不仅可以用来*检查*扩展，还可以用来修复它们。为此，我们可以让它们返回一个字符串或字符串列表；就像预扩展函数一样，这些字符串将替换整个扩展或单个符号。

作为示例，考虑*XML 文档*，它们由匹配的*XML 标签*内的文本组成。例如，考虑以下 HTML 片段，它是 XML 的一个子集：

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import HTML 
```

```py
HTML("<strong>A bold text</strong>") 
```

**粗体文本**

这个片段由两个包围文本的 HTML（XML）标签组成；标签名（`strong`）在打开标签（`<strong>`）和关闭标签（`</strong>`）中都存在。

对于一个*有限*的标签集合（例如，HTML 标签`<strong>`、`<head>`、`<body>`、`<form>`等等），我们可以定义一个上下文无关文法来解析它；每对标签将组成语法中的一个单独规则。然而，如果标签集合是*无限*的，比如在通用 XML 中，我们就不能定义一个合适的文法；这是因为约束条件要求关闭标签必须与打开标签匹配是上下文相关的，因此不适合上下文无关文法。

（顺便提一下，如果关闭标签具有标识符 *reversed* (`</gnorts>`), 那么一个上下文无关文法可以描述它。将其作为一个编程练习。）

我们可以通过引入适当的后扩展函数来解决此问题，这些函数可以自动使关闭标签与打开标签匹配。让我们从一个简单的语法开始，用于生成 XML 树：

```py
XML_GRAMMAR: Grammar = {
    "<start>": ["<xml-tree>"],
    "<xml-tree>": ["<<id>><xml-content></<id>>"],
    "<xml-content>": ["Text", "<xml-tree>"],
    "<id>": ["<letter>", "<id><letter>"],
    "<letter>": crange('a', 'z')
} 
```

```py
assert is_valid_grammar(XML_GRAMMAR) 
```

如果我们使用这个语法进行模糊测试，我们会得到非匹配的 XML 标签，这是预期的：

```py
xml_fuzzer = GrammarFuzzer(XML_GRAMMAR)
xml_fuzzer.fuzz() 
```

```py
'<t><qju>Text</m></q>'

```

设置一个后扩展函数，将第二个标识符设置为在第一个中找到的字符串，可以解决这个问题：

```py
XML_GRAMMAR.update({
    "<xml-tree>": [("<<id>><xml-content></<id>>",
                    opts(post=lambda id1, content, id2: [None, None, id1])
                    )]
}) 
```

```py
assert is_valid_grammar(XML_GRAMMAR) 
```

```py
xml_fuzzer = GeneratorGrammarFuzzer(XML_GRAMMAR)
xml_fuzzer.fuzz() 
```

```py
'<u>Text</u>'

```

### 示例：校验和

作为最后一个例子，让我们考虑引言中的校验和问题。有了我们新定义的修复机制，我们现在可以生成有效的信用卡号码：

```py
credit_card_fuzzer = GeneratorGrammarFuzzer(
    CHARGE_GRAMMAR, start_symbol="<credit-card-number>")
credit_card_number = credit_card_fuzzer.fuzz()
credit_card_number 
```

```py
'2967308746680770'

```

```py
assert valid_luhn_checksum(credit_card_number) 
```

有效性扩展到整个语法：

```py
charge_fuzzer = GeneratorGrammarFuzzer(CHARGE_GRAMMAR)
charge_fuzzer.fuzz() 
```

```py
'Charge $818819.97 to my credit card 2817984968014288'

```

## 本地检查和修复

到目前为止，我们总是首先生成整个表达式树，然后在之后检查其有效性。这可能会变得昂贵：如果首先生成多个元素，然后发现其中之一无效，我们会在尝试（随机）重新生成匹配输入上花费大量时间。

为了展示这个问题，让我们创建一个表达式语法，其中所有数字都由零和一组成。然而，我们不是通过构造性方法来做这件事，而是在事后使用 `post` 约束过滤掉所有不符合的表达式：

```py
binary_expr_grammar = extend_grammar(EXPR_GRAMMAR,
                                     {
                                         "<integer>": [("<digit><integer>", opts(post=lambda digit, _: digit in ["0", "1"])),
                                                       ("<digit>", opts(post=lambda digit: digit in ["0", "1"]))]
                                     }
                                     ) 
```

```py
assert is_valid_grammar(binary_expr_grammar) 
```

这可以工作，但非常慢；找到匹配表达式可能需要几秒钟。

```py
binary_expr_fuzzer = GeneratorGrammarFuzzer(binary_expr_grammar)
binary_expr_fuzzer.fuzz() 
```

```py
'(-+0)'

```

我们可以通过检查约束来解决此问题，不仅针对最终子树，而且在子树一旦完成就进行检查。为此，我们扩展了 `expand_tree_once()` 方法，使其在子树中的所有符号都展开后立即调用后扩展函数。

```py
class RestartExpansionException(Exception):
    pass 
```

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def expand_tree_once(self, tree: DerivationTree) -> DerivationTree:
        # Apply inherited method.  This also calls `expand_tree_once()` on all
        # subtrees.
        new_tree: DerivationTree = super().expand_tree_once(tree)

        (symbol, children) = new_tree
        if all([exp_post_expansion_function(expansion)
                is None for expansion in self.grammar[symbol]]):
            # No constraints for this symbol
            return new_tree

        if self.any_possible_expansions(tree):
            # Still expanding
            return new_tree

        return self.run_post_functions_locally(new_tree) 
```

主要工作发生在辅助方法 `run_post_functions_locally()` 中。它通过将 `depth` 设置为零，仅在当前节点上运行 `run_post_functions()` 函数 $f$，因为任何完成的子树已经运行了它们的后扩展函数。如果 $f$ 返回 `False`，则 `run_post_functions_locally()` 返回一个未展开的符号，这样主驱动程序就可以尝试另一种扩展。它最多尝试 10 次（在构建期间通过 `replacement_attempts` 参数配置）；之后，它引发一个 `RestartExpansionException` 来从头开始重新创建树。

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def run_post_functions_locally(self, new_tree: DerivationTree) -> DerivationTree:
        symbol, _ = new_tree

        result, children = self.run_post_functions(new_tree, depth=0)
        if not isinstance(result, bool) or result:
            # No constraints, or constraint satisfied
            # children = self.apply_result(result, children)
            new_tree = (symbol, children)
            return new_tree

        # Replace tree by unexpanded symbol and try again
        if self.log:
            print(
                all_terminals(new_tree),
                "did not satisfy",
                symbol,
                "constraint")

        if self.replacement_attempts_counter > 0:
            if self.log:
                print("Trying another expansion")
            self.replacement_attempts_counter -= 1
            return (symbol, None)

        if self.log:
            print("Starting from scratch")
        raise RestartExpansionException 
```

类构造方法以及 `fuzz_tree()` 被设置为处理额外的功能：

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def __init__(self, grammar: Grammar, replacement_attempts: int = 10,
                 **kwargs) -> None:
        super().__init__(grammar, **kwargs)
        self.replacement_attempts = replacement_attempts

    def restart_expansion(self) -> None:
        super().restart_expansion()
        self.replacement_attempts_counter = self.replacement_attempts

    def fuzz_tree(self) -> DerivationTree:
        self.replacement_attempts_counter = self.replacement_attempts
        while True:
            try:
                # This is fuzz_tree() as defined above
                tree = super().fuzz_tree()
                return tree
            except RestartExpansionException:
                self.restart_expansion() 
```

```py
binary_expr_fuzzer = GeneratorGrammarFuzzer(
    binary_expr_grammar, replacement_attempts=100)
binary_expr_fuzzer.fuzz() 
```

```py
'+0 / +-1 - 1 / +0 * -+0 * 0 * 1 / 1'

```

## 定义和用途

在上述生成器和约束的基础上，我们也可以处理复杂示例。来自 解析器章节 的 `VAR_GRAMMAR` 语法定义了多个变量为算术表达式（这些表达式本身也可以包含变量）。在语法上应用简单的 `GrammarFuzzer` 产生大量的标识符，但每个标识符都有一个独特的名称。

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
VAR_GRAMMAR: Grammar = {
    '<start>': ['<statements>'],
    '<statements>': ['<statement>;<statements>', '<statement>'],
    '<statement>': ['<assignment>'],
    '<assignment>': ['<identifier>=<expr>'],
    '<identifier>': ['<word>'],
    '<word>': ['<alpha><word>', '<alpha>'],
    '<alpha>': list(string.ascii_letters),
    '<expr>': ['<term>+<expr>', '<term>-<expr>', '<term>'],
    '<term>': ['<factor>*<term>', '<factor>/<term>', '<factor>'],
    '<factor>':
    ['+<factor>', '-<factor>', '(<expr>)', '<identifier>', '<number>'],
    '<number>': ['<integer>.<integer>', '<integer>'],
    '<integer>': ['<digit><integer>', '<digit>'],
    '<digit>': crange('0', '9')
} 
```

```py
assert is_valid_grammar(VAR_GRAMMAR) 
```

```py
g = GrammarFuzzer(VAR_GRAMMAR)
for i in range(10):
    print(g.fuzz()) 
```

```py
Gc=F/1*Y+M-D-9;N=n/(m)/m*7
a=79.0;W=o-9;v=2;K=u;D=9
o=y-z+y+4;q=5+W;X=T
M=-98.032*5/o
H=IA-5-1;n=3-t;QQ=5-5
Y=-80;d=D-M+M;Z=4.3+1*r-5+b
ZDGSS=(1*Y-4)*54/0*pcO/4;RI=r*5.0
Q=6+z-6;J=6/t/9/i-3-5+k
x=-GT*+-x*6++-93*5
q=da*T/e--v;x=3+g;bk=u

```

我们希望的是，在表达式中，只使用之前定义过的标识符。为此，我们在符号表周围引入了一组函数，该符号表跟踪所有已定义的变量。

```py
SYMBOL_TABLE: Set[str] = set() 
```

```py
def define_id(id: str) -> None:
    SYMBOL_TABLE.add(id) 
```

```py
def use_id() -> Union[bool, str]:
    if len(SYMBOL_TABLE) == 0:
        return False

    id = random.choice(list(SYMBOL_TABLE))
    return id 
```

```py
def clear_symbol_table() -> None:
    global SYMBOL_TABLE
    SYMBOL_TABLE = set() 
```

为了使用符号表，我们在`VAR_GRAMMAR`上附加了预展开和后展开函数，这些函数定义和查找符号表中的标识符。我们称我们的扩展语法为`CONSTRAINED_VAR_GRAMMAR`：

```py
CONSTRAINED_VAR_GRAMMAR = extend_grammar(VAR_GRAMMAR) 
```

首先，我们设置语法，使得每次定义一个标识符后，我们将其名称存储在符号表中：

```py
CONSTRAINED_VAR_GRAMMAR = extend_grammar(CONSTRAINED_VAR_GRAMMAR, {
    "<assignment>": [("<identifier>=<expr>",
                      opts(post=lambda id, expr: define_id(id)))]
}) 
```

其次，我们确保在生成标识符时，也从符号表中获取它。（在这里我们使用`post`，这样如果还没有可用的标识符，我们可以返回`False`，从而导致另一个展开的产生。）

```py
CONSTRAINED_VAR_GRAMMAR = extend_grammar(CONSTRAINED_VAR_GRAMMAR, {
    "<factor>": ['+<factor>', '-<factor>', '(<expr>)',
                 ("<identifier>", opts(post=lambda _: use_id())),
                 '<number>']
}) 
```

最后，每次我们（重新）启动展开时，我们都会清除符号表。这很有用，因为我们可能偶尔需要重新启动展开。

```py
CONSTRAINED_VAR_GRAMMAR = extend_grammar(CONSTRAINED_VAR_GRAMMAR, {
    "<start>": [("<statements>", opts(pre=clear_symbol_table))]
}) 
```

```py
assert is_valid_grammar(CONSTRAINED_VAR_GRAMMAR) 
```

使用这种语法进行模糊测试确保每个使用的标识符实际上已经定义：

```py
var_grammar_fuzzer = GeneratorGrammarFuzzer(CONSTRAINED_VAR_GRAMMAR)
for i in range(10):
    print(var_grammar_fuzzer.fuzz()) 
```

```py
DB=+(8/4/7-9+3+3)/2178/+-9
lNIqc=+(1+9-8)/2.9*8/5*0
Sg=(+9/8/6)*++1/(1+7)*8*4
r=+---552
iz=5/7/7;K=1+6*iz*1
q=3-2;MPy=q;p=2*5
zj=+5*-+35.2-+1.5727978+(-(-0/6-7+3))*--+44*1
Tl=((0*9+4-3)-6)/(-3-7*8*8/7)+9
aXZ=-5/-+3*9/3/1-8-+0*0/3+7+4
NA=-(8+a-1)*1.6;g=++7;a=++g*g*g

```

## 展开顺序

虽然我们之前的定义/使用示例确保每个使用的变量也是一个已定义的变量，但它并不关心这些定义的顺序。事实上，可能首先展开分号右侧的项，在符号表中创建一个条目，然后稍后用于左侧的表达式。我们可以通过在 Python 中实际评估产生的变量赋值来演示这一点，使用`exec()`来执行赋值序列。（鲜为人知的事实：Python *确实*支持`;`作为语句分隔符。）

```py
var_grammar_fuzzer = GeneratorGrammarFuzzer(CONSTRAINED_VAR_GRAMMAR)
with ExpectError():
    for i in range(100):
        s = var_grammar_fuzzer.fuzz()
        try:
            exec(s, {}, {})
        except SyntaxError:
            continue
        except ZeroDivisionError:
            continue
print(s) 
```

```py
f=(9)*kOj*kOj-6/7;kOj=(9-8)*7*1

```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11764/3970000697.py", line 6, in <module>
    exec(s, {}, {})
  File "<string>", line 1, in <module>
NameError: name 'kOj' is not defined (expected)

```

为了解决这个问题，我们允许显式指定展开的顺序。对于我们的先前模糊器，这种顺序无关紧要，因为最终所有符号都会被展开；如果我们有具有副作用展开函数，那么控制展开的顺序（以及相关函数调用的顺序）可能很重要。

为了指定顺序，我们为单个展开分配一个特殊的属性`order`。这是一个列表，其中包含每个符号的编号，表示展开的顺序，从最小的开始。例如，以下规则指定了分号分隔符左侧应首先展开：

```py
CONSTRAINED_VAR_GRAMMAR = extend_grammar(CONSTRAINED_VAR_GRAMMAR, {
    "<statements>": [("<statement>;<statements>", opts(order=[1, 2])),
                     "<statement>"]
}) 
```

同样，我们希望在表达式展开后才产生变量的定义，因为否则，表达式可能已经引用了已定义的变量：

```py
CONSTRAINED_VAR_GRAMMAR = extend_grammar(CONSTRAINED_VAR_GRAMMAR, {
    "<assignment>": [("<identifier>=<expr>", opts(post=lambda id, expr: define_id(id),
                                                  order=[2, 1]))],
}) 
```

辅助函数`exp_order()`允许我们检索顺序：

```py
def exp_order(expansion):
  """Return the specified expansion ordering, or None if unspecified"""
    return exp_opt(expansion, 'order') 
```

为了控制符号扩展的顺序，我们钩入`choose_tree_expansion()`方法，该方法专门设置为在子类中扩展。它通过`expandable_children`列表中的可扩展子项进行选择，并将它们与扩展的非终端子项匹配以确定它们的顺序号。具有最低顺序号的可扩展子项的索引`min_given_order`随后返回，选择这个子项进行扩展。

```py
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def choose_tree_expansion(self, tree: DerivationTree,
                              expandable_children: List[DerivationTree]) \
                              -> int:
  """Return index of subtree in `expandable_children`
 to be selected for expansion. Defaults to random."""
        (symbol, tree_children) = tree
        assert isinstance(tree_children, list)

        if len(expandable_children) == 1:
            # No choice
            return super().choose_tree_expansion(tree, expandable_children)

        expansion = self.find_expansion(tree)
        given_order = exp_order(expansion)
        if given_order is None:
            # No order specified
            return super().choose_tree_expansion(tree, expandable_children)

        nonterminal_children = [c for c in tree_children if c[1] != []]
        assert len(nonterminal_children) == len(given_order), \
            "Order must have one element for each nonterminal"

        # Find expandable child with lowest ordering
        min_given_order = None
        j = 0
        for k, expandable_child in enumerate(expandable_children):
            while j < len(
                    nonterminal_children) and expandable_child != nonterminal_children[j]:
                j += 1
            assert j < len(nonterminal_children), "Expandable child not found"
            if self.log:
                print("Expandable child #%d  %s has order %d" %
                      (k, expandable_child[0], given_order[j]))

            if min_given_order is None or given_order[j] < given_order[min_given_order]:
                min_given_order = k

        assert min_given_order is not None

        if self.log:
            print("Returning expandable child #%d  %s" %
                  (min_given_order, expandable_children[min_given_order][0]))

        return min_given_order 
```

这样，我们的模糊测试器现在可以尊重顺序，并且所有变量都得到了适当的定义：

```py
var_grammar_fuzzer = GeneratorGrammarFuzzer(CONSTRAINED_VAR_GRAMMAR)
for i in range(100):
    s = var_grammar_fuzzer.fuzz()
    if i < 10:
        print(s)
    try:
        exec(s, {}, {})
    except SyntaxError:
        continue
    except ZeroDivisionError:
        continue 
```

```py
a=(1)*0*3/8+0/8-4/8-0
r=+0*+8/-4+((9)*2-1-8+6/9)
D=+(2*3+6*0)-(5)/9*0/2;Q=D
C=9*(2-1)*9*0-1.2/6-3*5
G=-25.1
H=+4*4/8.5*4-8*4+(5);D=6
PIF=4841/++(460.1---626)*51755;E=(8)/-PIF+6.8*(7-PIF)*9*PIF;k=8
X=((0)*2/0*6+7*3)/(0-7-9)
x=94.2+25;x=++x/(7)+-9/8/2/x+-1/x;I=x
cBM=51.15;f=81*-+--((2++cBM/cBM*+1*0/0-5+cBM))

```

实际的编程语言不仅有一个全局作用域，还有多个局部作用域，通常是嵌套的。通过仔细组织全局和局部符号表，我们可以设置一个语法来处理所有这些。然而，在模糊测试编译器和解释器时，我们通常只关注单个函数，对于这些函数来说，一个单一的作用域就足够使大多数输入有效。

## 全部整合

让我们通过将我们的生成器功能与其他之前引入的语法功能集成来结束这一章，特别是覆盖率驱动的模糊测试和概率语法模糊测试。

集成单个特性的通用思路是通过*多重继承*，这我们在`ProbabilisticGrammarCoverageFuzzer`中已经使用过，如概率模糊测试练习中介绍的那样。

### 生成器和概率模糊测试

概率模糊测试很容易与生成器集成，因为它们都以不同的方式扩展了`GrammarFuzzer`。

```py
from ProbabilisticGrammarFuzzer import ProbabilisticGrammarFuzzer  # minor dependency 
```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import inheritance_conflicts 
```

```py
inheritance_conflicts(ProbabilisticGrammarFuzzer, GeneratorGrammarFuzzer) 
```

```py
['supported_opts']

```

我们必须实现`supported_opts()`作为两个超类的合并。同时，我们也设置了构造函数，使其调用这两个超类。

```py
class ProbabilisticGeneratorGrammarFuzzer(GeneratorGrammarFuzzer,
                                          ProbabilisticGrammarFuzzer):
  """Join the features of `GeneratorGrammarFuzzer` 
 and `ProbabilisticGrammarFuzzer`"""

    def supported_opts(self) -> Set[str]:
        return (super(GeneratorGrammarFuzzer, self).supported_opts() |
                super(ProbabilisticGrammarFuzzer, self).supported_opts())

    def __init__(self, grammar: Grammar, *, replacement_attempts: int = 10,
                 **kwargs):
  """Constructor.
 `replacement_attempts` - see `GeneratorGrammarFuzzer` constructor.
 All other keywords go into `ProbabilisticGrammarFuzzer`.
 """
        super(GeneratorGrammarFuzzer, self).__init__(
                grammar,
                replacement_attempts=replacement_attempts)
        super(ProbabilisticGrammarFuzzer, self).__init__(grammar, **kwargs) 
```

让我们给我们的联合类做一个简单的测试，使用概率来优先考虑长标识符：

```py
CONSTRAINED_VAR_GRAMMAR.update({
    '<word>': [('<alpha><word>', opts(prob=0.9)),
               '<alpha>'],
}) 
```

```py
pgg_fuzzer = ProbabilisticGeneratorGrammarFuzzer(CONSTRAINED_VAR_GRAMMAR)
pgg_fuzzer.supported_opts() 
```

```py
{'order', 'post', 'pre', 'prob'}

```

```py
pgg_fuzzer.fuzz() 
```

```py
'a=5+3/8/8+-1/6/9/8;E=6'

```

# 生成器和语法覆盖率

基于语法覆盖的模糊测试是一个更大的挑战。不仅仅是因为在两个方法中都有重载；我们可以像上面一样解决这些问题。

```py
from ProbabilisticGrammarFuzzer import ProbabilisticGrammarCoverageFuzzer  # minor dependency 
```

```py
from GrammarCoverageFuzzer import GrammarCoverageFuzzer  # minor dependency 
```

```py
inheritance_conflicts(ProbabilisticGrammarCoverageFuzzer,
                      GeneratorGrammarFuzzer) 
```

```py
['__init__', 'supported_opts']

```

```py
import [copy](https://docs.python.org/3/library/copy.html) 
```

```py
class ProbabilisticGeneratorGrammarCoverageFuzzer(GeneratorGrammarFuzzer,
                                                  ProbabilisticGrammarCoverageFuzzer):
  """Join the features of `GeneratorGrammarFuzzer` 
 and `ProbabilisticGrammarCoverageFuzzer`"""

    def supported_opts(self) -> Set[str]:
        return (super(GeneratorGrammarFuzzer, self).supported_opts() |
                super(ProbabilisticGrammarCoverageFuzzer, self).supported_opts())

    def __init__(self, grammar: Grammar, *,
                 replacement_attempts: int = 10, **kwargs) -> None:
  """Constructor.
 `replacement_attempts` - see `GeneratorGrammarFuzzer` constructor.
 All other keywords go into `ProbabilisticGrammarFuzzer`.
 """
        super(GeneratorGrammarFuzzer, self).__init__(
                grammar,
                replacement_attempts)
        super(ProbabilisticGrammarCoverageFuzzer, self).__init__(
                grammar,
                **kwargs) 
```

问题在于在扩展过程中，我们可能会生成（并覆盖）后来丢弃的扩展（例如，因为`post`函数返回`False`）。因此，我们必须*删除*这种不再存在于最终生产中的覆盖率。

我们通过在生成最终树之后*重建覆盖率*来解决这个问题。为此，我们钩入`fuzz_tree()`方法。我们在创建树之前保存原始覆盖率，然后在之后恢复它。然后我们遍历生成的树，再次添加其覆盖率（`add_tree_coverage()`）。

```py
class ProbabilisticGeneratorGrammarCoverageFuzzer(
        ProbabilisticGeneratorGrammarCoverageFuzzer):

    def fuzz_tree(self) -> DerivationTree:
        self.orig_covered_expansions = copy.deepcopy(self.covered_expansions)
        tree = super().fuzz_tree()
        self.covered_expansions = self.orig_covered_expansions
        self.add_tree_coverage(tree)
        return tree

    def add_tree_coverage(self, tree: DerivationTree) -> None:
        (symbol, children) = tree
        assert isinstance(children, list)
        if len(children) > 0:
            flat_children: List[DerivationTree] = [
                (child_symbol, None)
                for (child_symbol, _) in children
            ]
            self.add_coverage(symbol, flat_children)
            for c in children:
                self.add_tree_coverage(c) 
```

作为最后一步，我们确保如果必须从头开始重新启动扩展，我们也恢复之前的覆盖率，这样我们就可以完全重新开始：

```py
class ProbabilisticGeneratorGrammarCoverageFuzzer(
        ProbabilisticGeneratorGrammarCoverageFuzzer):

    def restart_expansion(self) -> None:
        super().restart_expansion()
        self.covered_expansions = self.orig_covered_expansions 
```

让我们试试这个。在我们生成一个字符串之后，我们应该在`expansion_coverage()`中看到其覆盖率：

```py
pggc_fuzzer = ProbabilisticGeneratorGrammarCoverageFuzzer(
    CONSTRAINED_VAR_GRAMMAR)
pggc_fuzzer.fuzz() 
```

```py
'H=+-2+7.4*(9)/0-6*8;T=5'

```

```py
pggc_fuzzer.expansion_coverage() 
```

```py
{'<alpha> -> H',
 '<alpha> -> T',
 '<assignment> -> <identifier>=<expr>',
 '<digit> -> 0',
 '<digit> -> 2',
 '<digit> -> 4',
 '<digit> -> 5',
 '<digit> -> 6',
 '<digit> -> 7',
 '<digit> -> 8',
 '<digit> -> 9',
 '<expr> -> <term>',
 '<expr> -> <term>+<expr>',
 '<expr> -> <term>-<expr>',
 '<factor> -> (<expr>)',
 '<factor> -> +<factor>',
 '<factor> -> -<factor>',
 '<factor> -> <number>',
 '<identifier> -> <word>',
 '<integer> -> <digit>',
 '<number> -> <integer>',
 '<number> -> <integer>.<integer>',
 '<start> -> <statements>',
 '<statement> -> <assignment>',
 '<statements> -> <statement>',
 '<statements> -> <statement>;<statements>',
 '<term> -> <factor>',
 '<term> -> <factor>*<term>',
 '<term> -> <factor>/<term>',
 '<word> -> <alpha>'}

```

再次模糊测试最终会覆盖所有标识符中的字母：

```py
[pggc_fuzzer.fuzz() for i in range(10)] 
```

```py
['llcyzc=3.0*02.3*1',
 'RfMgRYmd=---2.9',
 'p=+(7+3/4)*+-4-3.2*((2)-4)/2',
 'z=1-2/4-3*9+3+5',
 'v=(2/3)/1/2*8+(3)-7*2-1',
 'L=9.5/9-(7)/8/1+2-2;c=L',
 'U=+-91535-1-9-(9)/1;i=U',
 'g=-8.3*7*5+1*5*9-5;k=1',
 'J=+-8-(5/6-1)/7-6+7',
 'p=053/-(8*0*3*2/1);t=p']

```

通过 `ProbabilisticGeneratorGrammarCoverageFuzzer`，我们现在有一个语法模糊测试器，它结合了高效的语法模糊测试、覆盖率、概率和生成器函数。唯一缺少的是更短的名字。"PGGCFuzzer"，也许？

```py
class PGGCFuzzer(ProbabilisticGeneratorGrammarCoverageFuzzer):
  """The one grammar-based fuzzer that supports all fuzzingbook features"""
    pass 
```

## 经验教训

附属于语法扩展的函数可以服务

+   作为*生成器*，高效地从函数中生成符号展开；

+   作为*约束*来检查生成的字符串是否满足（复杂的）有效性条件；并且

+   作为*修复*，对生成的字符串应用更改，例如校验和和标识符。

## 下一步

通过本章，我们拥有了强大的语法，我们可以在多个领域中使用它们：

+   在关于模糊测试 API 的章节中，我们展示了如何使用 `GeneratorGrammarFuzzer` 功能来生成用于测试的复杂数据结构，结合语法和生成器函数。

+   在关于模糊测试用户界面的章节中，我们使用 `GeneratorGrammarFuzzer` 来生成复杂的用户界面输入。

## 背景

对于模糊测试 API，生成器函数非常常见。在关于 API 模糊测试的章节中，我们展示了如何将它们与语法结合，以生成更丰富的测试。

生成器函数和语法的组合主要因为我们在全 Python 环境中定义并使用语法。我们不知道有其他基于语法的模糊测试系统具有类似的功能。

## 练习

### 练习 1：树处理

到目前为止，我们的 `pre` 和 `post` 处理函数都接受和生成字符串。然而，在某些情况下，直接访问*推导树*可能很有用——例如，访问和检查某些子元素。

你的任务是扩展 `GeneratorGrammarFuzzer`，使其预处理和后处理函数可以接受和返回推导树。为此，请按照以下步骤操作：

1.  扩展 `GeneratorGrammarFuzzer`，使得一个函数可以返回一个推导树（一个元组）或推导树的列表，然后以与字符串相同的方式替换子树。

1.  扩展 `GeneratorGrammarFuzzer`，添加一个 `post_tree` 属性，它接受一个函数就像 `post` 一样，除了它的参数会是推导树。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GeneratorGrammarFuzzer.ipynb#Exercises)来练习并查看解决方案。

### 练习 2：属性语法

设置一个机制，通过该机制可以给推导树中的单个元素附加任意*属性*。扩展函数可以将这样的属性附加到单个符号上（比如，通过返回 `opts()`），并在后续调用中访问符号的属性。以下是一个示例：

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GeneratorGrammarFuzzer.ipynb#Exercises)来练习并查看解决方案。

```py
ATTR_GRAMMAR = {
    "<clause>": [("<xml-open>Text<xml-close>", opts(post=lambda x1, x2: [None, x1.name]))],
    "<xml-open>": [("<<tag>>", opts(post=lambda tag: opts(name=...)))],
    "<xml-close>": ["</<tag>>"]
} 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GeneratorGrammarFuzzer.ipynb#Exercises) 进行练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后更改：2024-01-10 15:45:59+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/GeneratorGrammarFuzzer.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[使用生成器进行模糊测试](https://www.fuzzingbook.org/html/GeneratorGrammarFuzzer.html)". 在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 的 "[模糊测试书籍](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/GeneratorGrammarFuzzer.html`](https://www.fuzzingbook.org/html/GeneratorGrammarFuzzer.html). Retrieved 2024-01-10 15:45:59+01:00.

```py
@incollection{fuzzingbook2024:GeneratorGrammarFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Fuzzing with Generators},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/GeneratorGrammarFuzzer.html}},
    note = {Retrieved 2024-01-10 15:45:59+01:00},
    url = {https://www.fuzzingbook.org/html/GeneratorGrammarFuzzer.html},
    urldate = {2024-01-10 15:45:59+01:00}
}

```
