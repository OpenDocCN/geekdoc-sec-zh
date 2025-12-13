# 概率语法模糊测试

> [`www.fuzzingbook.org/html/ProbabilisticGrammarFuzzer.html`](http://www.fuzzingbook.org/html/ProbabilisticGrammarFuzzer.html)

让我们通过为单个扩展分配 *概率* 来赋予语法更多的能力。这允许我们控制每个元素应该产生多少，从而允许我们 *针对* 生成测试以特定功能为目标。我们还展示了如何从给定的样本输入中学习这样的概率，并具体将测试直接针对这些样本中不常见的输入特征。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('9htOliNwopc') 
```

**先决条件**

+   您应该已经阅读了 关于语法的章节。

+   我们的实现与在 "Efficient Grammar Fuzzing" 中引入的基于语法的 fuzzer 挂钩。

+   为了从样本中学习概率，我们使用了 parsers。

## 概述

要 使用本章提供的代码，请编写

```py
>>> from fuzzingbook.ProbabilisticGrammarFuzzer import <identifier> 
```

然后利用以下功能。

一个 *概率* 语法允许将单个 *概率* 附着到产生规则上。要将单个扩展 `S` 的概率设置为值 `X`（介于 0 和 1 之间），将其替换为

```py
(S, opts(prob=X)) 
```

如果我们想确保生成的 90% 电话号码的区号以 `9` 开头，我们可以编写：

```py
>>> from Grammars import US_PHONE_GRAMMAR, extend_grammar, opts
>>> PROBABILISTIC_US_PHONE_GRAMMAR: Grammar = extend_grammar(US_PHONE_GRAMMAR,
>>> {
>>>       "<lead-digit>": [
>>>                           "2", "3", "4", "5", "6", "7", "8",
>>>                           ("9", opts(prob=0.9))
>>>                       ],
>>> }) 
```

`ProbabilisticGrammarFuzzer` 将提取并解释这些选项。以下是一个示例：

```py
>>> probabilistic_us_phone_fuzzer = ProbabilisticGrammarFuzzer(PROBABILISTIC_US_PHONE_GRAMMAR)
>>> [probabilistic_us_phone_fuzzer.fuzz() for i in range(5)]
['(918)925-2501',
 '(981)925-0792',
 '(934)995-5029',
 '(955)999-7801',
 '(964)927-0877'] 
```

如您所见，现在大多数区号都以 `9` 开头。

<svg width="336pt" height="287pt" viewBox="0.00 0.00 336.38 287.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 283)"><g id="node1" class="node"><title>ProbabilisticGrammarFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class ProbabilisticGrammarFuzzer:

尊重语法中概率的基于语法的 fuzzer。"><text text-anchor="start" x="8" y="-56.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">ProbabilisticGrammarFuzzer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="ProbabilisticGrammarFuzzer"><g id="a_node1_1"><a xlink:href="#" xlink:title="check_grammar(self) -> None:

检查传递的语法"><text text-anchor="start" x="26.75" y="-34.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">check_grammar()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="choose_node_expansion(self, node: DerivationTree, children_alternatives: List[Any]) -> int:

返回 `children_alternatives` 中要选择的扩展的索引。

'children_alternatives'：`node` 的可能子列表。

默认为随机。在子类中重载。"><text text-anchor="start" x="26.75" y="-21.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">choose_node_expansion()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="supported_opts(self) -> Set[str]:

支持的选项集合。应在子类中重载。"><text text-anchor="start" x="26.75" y="-8.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">supported_opts()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>GrammarFuzzer</title> <g id="a_node2"><a xlink:href="GrammarFuzzer.html" xlink:title="class GrammarFuzzer:

使用推导树高效地从语法中生成字符串。"><text text-anchor="start" x="45.88" y="-165.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarFuzzer</text> <g id="a_node2_4"><a xlink:href="#" xlink:title="GrammarFuzzer"><g id="a_node2_5"><a xlink:href="GrammarFuzzer.html" xlink:title="__init__(self, grammar: Dict[str, List[Expansion]], start_symbol: str = '<start>', min_nonterminals: int = 0, max_nonterminals: int = 10, disp: bool = False, log: Union[bool, int] = False) -> None:

从`grammar`生成字符串，以`start_symbol`开始。

如果提供了`min_nonterminals`或`max_nonterminals`，则使用它们作为限制。

对于生成的非终结符数量。

如果设置了`disp`，则显示中间推导树。

如果设置了`log`，则以文本形式在标准输出中显示中间步骤。"><text text-anchor="start" x="62.75" y="-143.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node2_6"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz(self) -> str:

从语法中生成一个字符串。"><text text-anchor="start" x="62.75" y="-130.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node2_7"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz_tree(self) -> DerivationTree:

从语法中生成一个推导树。"><text text-anchor="start" x="62.75" y="-118" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">fuzz_tree()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>ProbabilisticGrammarFuzzer->GrammarFuzzer</title></g> <g id="node3" class="node"><title>Fuzzer</title> <g id="a_node3"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊器的基类。"><text text-anchor="start" x="75.12" y="-262.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Fuzzer</text> <g id="a_node3_8"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node3_9"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

使用模糊输入运行 `runner`，如下所示 `<text text-anchor="start" x="77.75" y="-240" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">run()</text></a></g> <g id="a_node3_10"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

使用模糊输入，`trials` 次运行 `runner`，如下所示 `<text text-anchor="start" x="77.75" y="-227.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">runs()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>GrammarFuzzer->Fuzzer</title></g> <g id="node4" class="node"><title>图例</title> <text text-anchor="start" x="209.12" y="-52.62" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="209.12" y="-42.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="215.12" y="-42.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="209.12" y="-32.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="215.12" y="-32.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="209.12" y="-22.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="215.12" y="-22.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="209.12" y="-13.57" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 首位数字定律

在我们迄今为止的所有例子中，你可能已经注意到程序生成的输入与现实生活中出现的“自然”输入有很大不同。即使是像数字这样的无害元素也是如此——是的，我们迄今为止生成的数字实际上与现实世界中的数字是不同的。这是因为现实生活中的数值数据集中，*首位有效数字*很可能是小的：实际上，平均而言，首位数字 `1` 出现的频率比首位数字 `8` 或 `9` 高出 *六倍*。已经证明，这一结果适用于各种数据集，包括电费账单、街道地址、股票价格、房价、人口数量、死亡率、河流长度、物理和数学常数（维基百科）。

这个首位数字定律最初是由 Newcomb 观察到的 [[Simon Newcomb, 1881](http://www.jstor.org/stable/2369148)]，后来由 Benford 在 [[Frank Benford, 1938](http://links.jstor.org/sici?sici=0003-049X%2819380331%2978%3A4%3C551%3ATLOAN%3E2.0.CO%3B2-G)] 中形式化。让我们看看决定一个数字首位数字的条件。我们可以通过将数字转换为字符串并取第一个字符来轻松地计算首位数字：

```py
def first_digit_via_string(x: int) -> int:
    return ord(repr(x)[0]) - ord('0') 
```

```py
first_digit_via_string(2001) 
```

```py
2

```

要进行数学处理，我们必须取它们对数的分数部分，或者正式地

$$ d = 10^{\{\log_{10}(x)\}} $$

$\{x\}$ 是 $x$ 的分数部分（即 $\{1.234\} = 0.234$）。

```py
import [math](https://docs.python.org/3/library/math.html) 
```

```py
def first_digit_via_log(x: int) -> int:
    frac, whole = math.modf(math.log10(x))
    return int(10 ** frac) 
```

```py
first_digit_via_log(2001) 
```

```py
2

```

大多数“自然”出现的数字集合在其对数分数部分中不应有任何偏差，因此，分数部分 $\{\log_{10}(x)\}$ 通常均匀分布。然而，个别数字的分数部分*并不*均匀分布。

对于一个数字以数字 $d$ 开头，必须满足条件 $d < 10^{\{\log_{10}(x)\}} < d + 1$。因此，要使数字以数字 1 开头，分数部分 $\{\log_{10}(x)\}$ 必须在以下范围内

```py
(math.log10(1), math.log10(2)) 
```

```py
(0.0, 0.3010299956639812)

```

然而，要开始计算数字 2 的概率，它必须在以下范围内

```py
(math.log10(2), math.log10(3)) 
```

```py
(0.3010299956639812, 0.47712125471966244)

```

这个范围要小得多。形式上，首位数字 $d$ 的概率 $P(d)$（再次假设分数部分均匀分布）被称为本福特定律：$$ P(d) = \log_{10}(d + 1) - \log_{10}(d) $$ 这给我们：

```py
def prob_leading_digit(d: int) -> float:
    return math.log10(d + 1) - math.log10(d) 
```

让我们计算所有数字的概率：

```py
digit_probs = [prob_leading_digit(d) for d in range(1, 10)]
[(d, "%.2f" % digit_probs[d - 1]) for d in range(1, 10)] 
```

```py
[(1, '0.30'),
 (2, '0.18'),
 (3, '0.12'),
 (4, '0.10'),
 (5, '0.08'),
 (6, '0.07'),
 (7, '0.06'),
 (8, '0.05'),
 (9, '0.05')]

```

![图片](img/209633c7fc0e1c06b36760d52f9e7433.png)

我们看到，首位数字 1 的概率确实是首位数字 9 的六倍。

本福特定律有许多应用。最值得注意的是，它可以用来检测“非自然”数字，即那些看起来是随机创建而不是来自“自然”来源的数字。如果你写一篇科学论文并伪造数据，通过插入随机数（例如，使用我们的语法模糊器[GrammarFuzzer.html]对整数进行操作），你可能会违反本福特定律，这确实可以被察觉。另一方面，如果我们*想要*创建符合本福特定律的数字，我们该如何进行？为此，我们需要能够在我们的语法中*编码*上述概率，以确保在所有情况下，首位数字确实是 `1` 的 30%。

## 指定概率

本章的目标是为语法中的单个扩展分配*概率*，这样我们就可以表达某些扩展选项应该比其他选项更受青睐。这不仅对生成“自然”外观的数字有用，而且更有助于*指导*测试生成向特定目标。如果你最近更改了程序中的某些代码，你可能会希望生成测试这些更改代码的输入。通过提高与更改代码相关的输入元素的概率，你将得到更多测试这些更改代码的测试。

我们表达概率的概念是使用注释机制（在关于语法的章节中介绍）注释单个展开式，例如概率等属性。为此，我们允许展开式不仅可以是一个字符串，也可以是一个字符串和一组属性的 *对*，如下所示

```py
"<expr>":
        [("<term> + <expr>", opts(prob=0.1)),
         ("<term> - <expr>", opts(prob=0.2)),
         "<term>"] 
```

在这里，`opts()` 函数将允许我们表达选择单个展开式的概率。加法有 10% 的概率，减法有 20%。剩余的概率（在这种情况下为 70%）将平均分配给所有未指定的展开式（在这种情况下是最后一个）。

现在我们可以使用带有 `opts()` 的对来为我们的表达式语法分配概率：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from Fuzzer import Fuzzer 
```

```py
from GrammarFuzzer import GrammarFuzzer, all_terminals, display_tree, DerivationTree 
```

```py
from Grammars import is_valid_grammar, EXPR_GRAMMAR, START_SYMBOL, crange
from Grammars import opts, exp_string, exp_opt, set_opts
from Grammars import Grammar, Expansion 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import List, Dict, Set, Optional, cast, Any, Tuple 
```

```py
PROBABILISTIC_EXPR_GRAMMAR: Grammar = {
    "<start>":
        ["<expr>"],

    "<expr>":
        [("<term> + <expr>", opts(prob=0.1)),
         ("<term> - <expr>", opts(prob=0.2)),
         "<term>"],

    "<term>":
        [("<factor> * <term>", opts(prob=0.1)),
         ("<factor> / <term>", opts(prob=0.1)),
         "<factor>"
         ],

    "<factor>":
        ["+<factor>", "-<factor>", "(<expr>)",
            "<leadinteger>", "<leadinteger>.<integer>"],

    "<leadinteger>":
        ["<leaddigit><integer>", "<leaddigit>"],

    # Benford's law: frequency distribution of leading digits
    "<leaddigit>":
        [("1", opts(prob=0.301)),
         ("2", opts(prob=0.176)),
         ("3", opts(prob=0.125)),
         ("4", opts(prob=0.097)),
         ("5", opts(prob=0.079)),
         ("6", opts(prob=0.067)),
         ("7", opts(prob=0.058)),
         ("8", opts(prob=0.051)),
         ("9", opts(prob=0.046)),
         ],

    # Remaining digits are equally distributed
    "<integer>":
        ["<digit><integer>", "<digit>"],

    "<digit>":
        ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"],
} 
```

```py
assert is_valid_grammar(PROBABILISTIC_EXPR_GRAMMAR, supported_opts={'prob'}) 
```

这是语法展开式在内部表示的方式：

```py
leaddigits: List[Expansion] = PROBABILISTIC_EXPR_GRAMMAR["<leaddigit>"]
leaddigits 
```

```py
[('1', {'prob': 0.301}),
 ('2', {'prob': 0.176}),
 ('3', {'prob': 0.125}),
 ('4', {'prob': 0.097}),
 ('5', {'prob': 0.079}),
 ('6', {'prob': 0.067}),
 ('7', {'prob': 0.058}),
 ('8', {'prob': 0.051}),
 ('9', {'prob': 0.046})]

```

然而，我们通常通过指定的辅助函数访问展开字符串和相关的概率，即 `exp_string()`（来自关于语法的章节）和 `exp_prob()`：

```py
leaddigit_expansion = leaddigits[0]
leaddigit_expansion 
```

```py
('1', {'prob': 0.301})

```

```py
exp_string(leaddigit_expansion) 
```

```py
'1'

```

```py
def exp_prob(expansion: Expansion) -> float:
  """Return the options of an expansion"""
    return exp_opt(expansion, 'prob') 
```

```py
exp_prob(leaddigit_expansion) 
```

```py
0.301

```

我们现有的模糊器都设置为使用这种方式注释的语法。它们简单地忽略所有注释。

```py
f = GrammarFuzzer(PROBABILISTIC_EXPR_GRAMMAR)
f.fuzz() 
```

```py
'4 + ++--7.0 - -7 - +++7.3 * (1 * 3 + 5 / 3 / 5 + 2) * 38 * (2 + 8)'

```

```py
from GrammarCoverageFuzzer import GrammarCoverageFuzzer  # minor dependency 
```

```py
f = GrammarCoverageFuzzer(PROBABILISTIC_EXPR_GRAMMAR)
f.fuzz() 
```

```py
'1.30'

```

## 计算概率

让我们定义访问给定展开式概率的函数。在这样做的同时，它们也会检查不一致性。

### 分布概率

这是如何为没有指定概率的展开式分配概率的。给定一个展开规则

$$S ::= a_1\:|\: a_2 \:|\: \dots \:|\: a_n \:|\: u_1 \:|\: u_2 \:|\: \dots u_m$$

对于 $n \ge 0$ 个具有指定概率 $p(a_i)$ 的备选方案 $a_i$ 和 $m \ge 0$ 个具有未指定概率 $p(u_j)$ 的备选方案 $u_j$，"剩余"的概率将平均分配给所有 $u_j$；换句话说，

$$p(u_j) = \frac{1 - \sum_{i = 1}^{n}p(a_i)}{m}$$

如果没有指定概率（$n = 0$），则所有展开式具有相同的概率。

概率的总体和必须为 1：

$$\sum_{i = 1}^{n} p(a_i) + \sum_{j = 1}^{m} p(u_i) = 1$$

在分配概率的同时，我们检查这些属性。

函数 `exp_probabilities()` 返回一个映射，将规则中的所有展开式映射到它们各自的概率。

```py
def exp_probabilities(expansions: List[Expansion],
                      nonterminal: str ="<symbol>") \
        -> Dict[Expansion, float]:
    probabilities = [exp_prob(expansion) for expansion in expansions]
    prob_dist = prob_distribution(probabilities, nonterminal)

    prob_mapping: Dict[Expansion, float] = {}
    for i in range(len(expansions)):
        expansion = exp_string(expansions[i])
        prob_mapping[expansion] = prob_dist[i]

    return prob_mapping 
```

`exp_probabilities()` 的核心处理在 `prob_distribution()` 中完成，它执行实际的检查和计算。

```py
def prob_distribution(probabilities: List[Optional[float]],
                      nonterminal: str = "<symbol>"):
    epsilon = 0.00001

    number_of_unspecified_probabilities = probabilities.count(None)
    if number_of_unspecified_probabilities == 0:
        sum_probabilities = cast(float, sum(probabilities))
        assert abs(sum_probabilities - 1.0) < epsilon, \
            nonterminal + ": sum of probabilities must be 1.0"
        return probabilities

    sum_of_specified_probabilities = 0.0
    for p in probabilities:
        if p is not None:
            sum_of_specified_probabilities += p
    assert 0 <= sum_of_specified_probabilities <= 1.0, \
        nonterminal + ": sum of specified probabilities must be between 0.0 and 1.0"

    default_probability = ((1.0 - sum_of_specified_probabilities)
                           / number_of_unspecified_probabilities)
    all_probabilities = []
    for p in probabilities:
        if p is None:
            p = default_probability
        all_probabilities.append(p)

    assert abs(sum(all_probabilities) - 1.0) < epsilon
    return all_probabilities 
```

这是 `exp_probabilities()` 为注释的 `<leaddigit>` 元素返回的映射：

```py
print(exp_probabilities(PROBABILISTIC_EXPR_GRAMMAR["<leaddigit>"])) 
```

```py
{'1': 0.301, '2': 0.176, '3': 0.125, '4': 0.097, '5': 0.079, '6': 0.067, '7': 0.058, '8': 0.051, '9': 0.046}

```

如果没有展开式被注释，所有展开式被选中的可能性相同，就像我们之前的语法模糊器一样。

```py
print(exp_probabilities(PROBABILISTIC_EXPR_GRAMMAR["<digit>"])) 
```

```py
{'0': 0.1, '1': 0.1, '2': 0.1, '3': 0.1, '4': 0.1, '5': 0.1, '6': 0.1, '7': 0.1, '8': 0.1, '9': 0.1}

```

这是 `exp_probabilities()` 如何将任何剩余的概率分配给未注释的展开式：

```py
exp_probabilities(PROBABILISTIC_EXPR_GRAMMAR["<expr>"]) 
```

```py
{'<term> + <expr>': 0.1, '<term> - <expr>': 0.2, '<term>': 0.7}

```

### 检查概率

我们可以使用 `exp_probabilities()` 的检查功能来检查概率语法的一致性：

```py
def is_valid_probabilistic_grammar(grammar: Grammar,
                                   start_symbol: str = START_SYMBOL) -> bool:
    if not is_valid_grammar(grammar, start_symbol):
        return False

    for nonterminal in grammar:
        expansions = grammar[nonterminal]
        _ = exp_probabilities(expansions, nonterminal)

    return True 
```

```py
assert is_valid_probabilistic_grammar(PROBABILISTIC_EXPR_GRAMMAR) 
```

```py
assert is_valid_probabilistic_grammar(EXPR_GRAMMAR) 
```

```py
from ExpectError import ExpectError 
```

```py
with ExpectError():
    assert is_valid_probabilistic_grammar({"<start>": [("1", opts(prob=0.5))]}) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11325/1729613243.py", line 2, in <module>
    assert is_valid_probabilistic_grammar({"<start>": [("1", opts(prob=0.5))]})
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11325/3185228479.py", line 8, in is_valid_probabilistic_grammar
    _ = exp_probabilities(expansions, nonterminal)
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11325/4013279880.py", line 5, in exp_probabilities
    prob_dist = prob_distribution(probabilities, nonterminal)  # type: ignore
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11325/1431748745.py", line 8, in prob_distribution
    assert abs(sum_probabilities - 1.0) < epsilon, \
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: <start>: sum of probabilities must be 1.0 (expected)

```

```py
with ExpectError():
    assert is_valid_probabilistic_grammar(
        {"<start>": [("1", opts(prob=1.5)), "2"]}) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11325/2913569331.py", line 2, in <module>
    assert is_valid_probabilistic_grammar(
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11325/3185228479.py", line 8, in is_valid_probabilistic_grammar
    _ = exp_probabilities(expansions, nonterminal)
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11325/4013279880.py", line 5, in exp_probabilities
    prob_dist = prob_distribution(probabilities, nonterminal)  # type: ignore
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_11325/1431748745.py", line 16, in prob_distribution
    assert 0 <= sum_of_specified_probabilities <= 1.0, \
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: <start>: sum of specified probabilities must be between 0.0 and 1.0 (expected)

```

## 按概率展开

现在我们已经看到了如何为语法指定概率，我们实际上可以实施概率扩展。在我们的 `ProbabilisticGrammarFuzzer` 中，只需要重载一个方法，即 `choose_node_expansion()`。对于我们可以选择的每个子节点（通常是符号的所有扩展），我们确定它们的概率（使用上面定义的 `exp_probabilities()`），并使用带有 `weight` 参数的 `random.choices()` 进行加权选择。

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
class ProbabilisticGrammarFuzzer(GrammarFuzzer):
  """A grammar-based fuzzer respecting probabilities in grammars."""

    def check_grammar(self) -> None:
        super().check_grammar()
        assert is_valid_probabilistic_grammar(self.grammar, self.start_symbol)

    def supported_opts(self) -> Set[str]:
        return super().supported_opts() | {'prob'} 
```

```py
class ProbabilisticGrammarFuzzer(ProbabilisticGrammarFuzzer):
    def choose_node_expansion(self, node: DerivationTree,
                              children_alternatives: List[Any]) -> int:
        (symbol, tree) = node
        expansions = self.grammar[symbol]
        probabilities = exp_probabilities(expansions)

        weights: List[float] = []
        for children in children_alternatives:
            expansion = all_terminals((symbol, children))
            children_weight = probabilities[expansion]
            if self.log:
                print(repr(expansion), "p =", children_weight)
            weights.append(children_weight)

        if sum(weights) == 0:
            # No alternative (probably expanding at minimum cost)
            return random.choices(
                range(len(children_alternatives)))[0]
        else:
            return random.choices(
                range(len(children_alternatives)), weights=weights)[0] 
```

我们的概率语法模糊测试器与非概率语法模糊测试器的工作方式相同，只不过它实际上尊重概率注释。让我们生成一些遵循本福特定律的“自然”数字：

```py
natural_fuzzer = ProbabilisticGrammarFuzzer(
    PROBABILISTIC_EXPR_GRAMMAR, start_symbol="<leadinteger>")
print([natural_fuzzer.fuzz() for i in range(20)]) 
```

```py
['2588', '106', '10', '2', '7', '1', '95', '4', '192', '8', '2', '1', '1', '2', '2', '208', '1036', '5592', '157', '1442']

```

相比之下，这些数字是纯随机的：

```py
integer_fuzzer = GrammarFuzzer(
    PROBABILISTIC_EXPR_GRAMMAR, start_symbol="<leadinteger>")
print([integer_fuzzer.fuzz() for i in range(20)]) 
```

```py
['3', '1', '56', '5', '408251024', '7', '27', '2', '9', '6456', '7', '32', '1', '4', '7', '19', '2', '6', '2', '5']

```

“自然”数字真的比随机的数字更“自然”吗？为了证明 `ProbabilisticGrammarFuzzer` 确实尊重概率注释，让我们为首位数字创建一个特定的模糊测试器：

```py
leaddigit_fuzzer = ProbabilisticGrammarFuzzer(
    PROBABILISTIC_EXPR_GRAMMAR, start_symbol="<leaddigit>")
leaddigit_fuzzer.fuzz() 
```

```py
'8'

```

如果我们生成数千个首位数字，它们的分布应该再次遵循本福特定律：

```py
trials = 10000

count = {}
for c in crange('0', '9'):
    count[c] = 0

for i in range(trials):
    count[leaddigit_fuzzer.fuzz()] += 1

print([(digit, count[digit] / trials) for digit in count]) 
```

```py
[('0', 0.0), ('1', 0.3003), ('2', 0.1756), ('3', 0.1222), ('4', 0.0938), ('5', 0.0816), ('6', 0.0651), ('7', 0.06), ('8', 0.0537), ('9', 0.0477)]

```

命题得证！分布几乎完全符合最初指定的。我们现在有一个可以通过指定概率来控制的模糊测试器。

## 定向模糊测试

为单个扩展分配概率使我们能够很好地控制应该生成哪些输入。通过明智地选择概率，我们可以*引导*模糊测试针对特定的功能和特性——例如，针对特别关键、容易出错或最近更改的功能。

例如，考虑来自语法章节的 URL 语法。让我们假设我们刚刚对我们的安全 FTP 协议的实现进行了修改。通过提高 `ftps` 方案的概率，我们可以生成更多专门测试此功能的 URL。

首先，让我们定义一个设置特定选项的辅助函数：

这里有一个专门用于概率的特化：

```py
def set_prob(grammar: Grammar, symbol: str, 
             expansion: Expansion, prob: Optional[float]) -> None:
  """Set the probability of the given expansion of grammar[symbol]"""
    set_opts(grammar, symbol, expansion, opts(prob=prob)) 
```

让我们使用 `set_prob()` 给 `ftps` 扩展分配 80%的概率：

```py
from Grammars import URL_GRAMMAR, extend_grammar 
```

```py
probabilistic_url_grammar = extend_grammar(URL_GRAMMAR)
set_prob(probabilistic_url_grammar, "<scheme>", "ftps", 0.8)
assert is_valid_probabilistic_grammar(probabilistic_url_grammar) 
```

```py
probabilistic_url_grammar["<scheme>"] 
```

```py
['http', 'https', 'ftp', ('ftps', {'prob': 0.8})]

```

如果我们用这种语法进行模糊测试，我们将得到大量的 `ftps:` 前缀：

```py
prob_url_fuzzer = ProbabilisticGrammarFuzzer(probabilistic_url_grammar)
for i in range(10):
    print(prob_url_fuzzer.fuzz()) 
```

```py
ftps://cispa.saarland:80
ftps://user:password@cispa.saarland/
ftps://fuzzingbook.com/abc
ftps://fuzzingbook.com/x48
ftps://user:password@fuzzingbook.com/
ftps://www.google.com:2?x18=8
ftps://user:password@www.google.com:6
ftps://user:password@cispa.saarland/def
ftps://user:password@cispa.saarland/def?def=52
ftps://user:password@cispa.saarland/

```

类似地，我们可以将 URL 生成定向到特定的主机或端口；我们可以优先选择带有查询、片段或登录信息的 URL，或者不包含这些信息的 URL。只需设置适当的概率即可。

通过将扩展的概率设置为零，我们可以有效地禁用特定的扩展：

```py
set_prob(probabilistic_url_grammar, "<scheme>", "ftps", 0.0)
assert is_valid_probabilistic_grammar(probabilistic_url_grammar) 
```

```py
prob_url_fuzzer = ProbabilisticGrammarFuzzer(probabilistic_url_grammar)
for i in range(10):
    print(prob_url_fuzzer.fuzz()) 
```

```py
ftp://user:password@cispa.saarland/x00
https://user:password@www.google.com/?def=6&def=x18&def=def
https://user:password@fuzzingbook.com:7/?def=abc
https://user:password@www.google.com:8080/
ftp://www.google.com/?abc=36&x34=5
http://user:password@cispa.saarland/
https://www.google.com/
https://user:password@www.google.com:85/?def=18
http://user:password@www.google.com:80/
http://fuzzingbook.com:80/?abc=def

```

注意，即使我们将扩展的概率设置为零，我们仍然可能看到扩展被采用。这可能在我们的语法模糊测试器的“关闭”阶段发生，此时扩展以最小成本关闭。在这个阶段，即使扩展的概率为零，如果这是关闭扩展所必需的，也会被采用。

让我们使用我们表达式语法中的 `<expr>` 规则来说明这个特性：

```py
from Grammars import EXPR_GRAMMAR 
```

```py
probabilistic_expr_grammar = extend_grammar(EXPR_GRAMMAR)
probabilistic_expr_grammar["<expr>"] 
```

```py
['<term> + <expr>', '<term> - <expr>', '<term>']

```

如果我们将 `<term>` 扩展的概率设置为零，字符串应该会不断扩展。

```py
set_prob(probabilistic_expr_grammar, "<expr>", "<term>", 0.0)
assert is_valid_probabilistic_grammar(probabilistic_expr_grammar) 
```

然而，在“关闭”阶段，子表达式最终会展开为 `<term>`，因为这是关闭展开的唯一方式。跟踪 `choose_node_expansion()` 显示它只使用了一个可能展开 `<term>`，即使其指定的概率为零，也必须采取它。

```py
prob_expr_fuzzer = ProbabilisticGrammarFuzzer(probabilistic_expr_grammar)
prob_expr_fuzzer.fuzz() 
```

```py
'44 / 7 / 1 * 3 / 6 - +1.63 + 3 * 7 + 1 - 2'

```

## 上下文中的概率

虽然指定的概率为我们提供了控制哪些展开以及它们被采取多频繁的手段，但这种控制本身可能不足以。例如，考虑以下用于 IPv4 地址的语法：

```py
def decrange(start: int, end: int) -> List[Expansion]:
  """Return a list with string representations of numbers in the range [start, end)"""
    return [repr(n) for n in range(start, end)] 
```

```py
IP_ADDRESS_GRAMMAR: Grammar = {
    "<start>": ["<address>"],
    "<address>": ["<octet>.<octet>.<octet>.<octet>"],
    # ["0", "1", "2", ..., "255"]
    "<octet>": decrange(0, 256)
} 
```

```py
print(IP_ADDRESS_GRAMMAR["<octet>"][:20]) 
```

```py
['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19']

```

```py
assert is_valid_grammar(IP_ADDRESS_GRAMMAR) 
```

我们可以轻松地使用这种语法来创建 IP 地址：

```py
ip_fuzzer = ProbabilisticGrammarFuzzer(IP_ADDRESS_GRAMMAR)
ip_fuzzer.fuzz() 
```

```py
'34.155.77.136'

```

然而，如果我们想要为四个八位字节中的任何一个分配一个特定的概率，我们就无能为力了。我们所能做的就是为所有四个八位字节分配相同的概率分布：

```py
probabilistic_ip_address_grammar = extend_grammar(IP_ADDRESS_GRAMMAR)
set_prob(probabilistic_ip_address_grammar, "<octet>", "127", 0.8) 
```

```py
probabilistic_ip_fuzzer = ProbabilisticGrammarFuzzer(
    probabilistic_ip_address_grammar)
probabilistic_ip_fuzzer.fuzz() 
```

```py
'127.127.127.127'

```

如果我们想要为四个八位字节中的每一个分配 *不同的* 概率，我们该怎么办？

答案在于 *上下文* 的概念，我们在讨论 语法覆盖模糊器 时已经见过。与基于覆盖的模糊测试一样，想法是 *复制* 我们想要根据其上下文设置概率的元素。在我们的例子中，这意味着复制 `<octet>` 元素为四个单独的元素，每个元素都可以获得一个单独的概率分布。我们可以通过 `duplicate_context()` 方法程序化地做到这一点：

```py
from GrammarCoverageFuzzer import duplicate_context  # minor dependency 
```

```py
probabilistic_ip_address_grammar = extend_grammar(IP_ADDRESS_GRAMMAR)
duplicate_context(probabilistic_ip_address_grammar, "<address>") 
```

```py
probabilistic_ip_address_grammar["<address>"] 
```

```py
['<octet-1>.<octet-2>.<octet-3>.<octet-4>']

```

我们现在可以为每个 `<octet>` 符号分配不同的概率。例如，我们可以通过将它们的概率设置为 100% 来强制特定的展开：

```py
set_prob(probabilistic_ip_address_grammar, "<octet-1>", "127", 1.0)
set_prob(probabilistic_ip_address_grammar, "<octet-2>", "0", 1.0) 
```

```py
assert is_valid_probabilistic_grammar(probabilistic_ip_address_grammar) 
```

剩下的两个八位字节 `<octet-3>` 和 `<octet-4>` 没有设置特定的概率。在模糊测试期间，因此它们的所有展开（所有八位字节）仍然可用：

```py
probabilistic_ip_fuzzer = ProbabilisticGrammarFuzzer(
    probabilistic_ip_address_grammar)
[probabilistic_ip_fuzzer.fuzz() for i in range(5)] 
```

```py
['127.0.201.77',
 '127.0.98.36',
 '127.0.12.232',
 '127.0.146.161',
 '127.0.245.151']

```

就像覆盖一样，我们可以任意多次复制语法规则以获得更多更细粒度的概率控制。然而，这种更细粒度的控制也带来了必须维护这些概率的代价。因此，在下一节中，我们将讨论自动分配和调整此类概率的方法。

## 从样本中学习概率

概率不一定总是需要手动设置。它们也可以从其他来源 *学习*，特别是通过计算 *在给定输入集中单个展开发生的频率*。这在许多情况下都很有用，包括：

1.  测试 *常见* 的特性。想法是在测试过程中，人们可能首先想要关注频繁发生（或频繁使用）的特性，以确保最常见的用法正确无误。

1.  测试 *不常见* 的特性。在这里，想法是让测试生成器专注于在输入中很少见（或根本不见）的特性。这与 语法覆盖 的动机相同，但从概率的角度来看。

1.  关注特定的*切片*。可能有一组输入特别有趣（例如，因为它们锻炼了关键功能，或者最近发现了错误）。使用这种学习分布进行模糊测试使我们能够*关注*这些感兴趣的功能。

让我们先介绍计数扩展和学习概率，然后详细说明这些场景。

### 计数扩展

我们首先实现一种方法来取一组输入并确定该集合中的扩展数量。为此，我们需要在上一章中引入的*解析器*将字符串输入转换为推导树。对于我们的 IP 地址语法，这是如何工作的：

```py
from Parser import Parser, EarleyParser 
```

```py
IP_ADDRESS_TOKENS = {"<octet>"}  # EarleyParser needs explicit tokens 
```

```py
parser = EarleyParser(IP_ADDRESS_GRAMMAR) 
```

```py
tree, *_ = parser.parse("127.0.0.1")
display_tree(tree) 
```

<svg width="375pt" height="173pt" viewBox="0.00 0.00 374.75 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="183.38" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="183.38" y="-101.45" font-family="Times,serif" font-size="14.00"><address></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="21.38" y="-51.2" font-family="Times,serif" font-size="14.00"><octet></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="75.38" y="-51.2" font-family="Times,serif" font-size="14.00">. (46)</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="129.38" y="-51.2" font-family="Times,serif" font-size="14.00"><octet></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="183.38" y="-51.2" font-family="Times,serif" font-size="14.00">. (46)</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="237.38" y="-51.2" font-family="Times,serif" font-size="14.00"><octet></text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="291.38" y="-51.2" font-family="Times,serif" font-size="14.00">. (46)</text></g> <g id="edge10" class="edge"><title>1->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="345.38" y="-51.2" font-family="Times,serif" font-size="14.00"><octet></text></g> <g id="edge11" class="edge"><title>1->11</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="21.38" y="-0.95" font-family="Times,serif" font-size="14.00">127</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="129.38" y="-0.95" font-family="Times,serif" font-size="14.00">0 (48)</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="237.38" y="-0.95" font-family="Times,serif" font-size="14.00">0 (48)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="345.38" y="-0.95" font-family="Times,serif" font-size="14.00">1 (49)</text></g> <g id="edge12" class="edge"><title>11->12</title></g></g></svg>

在这样的树中，我们现在可以*计数*单个扩展。例如，在上面的树中，我们有`<octet>`扩展到`0`的两个，一个扩展到`1`，一个扩展到`127`。`<octet>`扩展到`0`占所有看到的扩展的 50%；扩展到`127`和`1`各占 25%，其他占 0%。这是我们想要分配给我们的“学习”语法的概率。

我们引入了一个类`ExpansionCountMiner`，它允许我们计数单个扩展发生的频率。它的初始化方法接受一个解析器（例如，一个`EarleyParser`），该解析器将使用适当的语法进行初始化。

```py
from GrammarCoverageFuzzer import expansion_key  # minor dependency 
```

```py
from Grammars import is_nonterminal 
```

```py
class ExpansionCountMiner:
    def __init__(self, parser: Parser, log: bool = False) -> None:
        assert isinstance(parser, Parser)
        self.grammar = extend_grammar(parser.grammar())
        self.parser = parser
        self.log = log
        self.reset() 
```

属性`expansion_counts`保存了看到的扩展；使用`add_tree()`添加一个树遍历给定的树并添加所有看到的扩展。

```py
class ExpansionCountMiner(ExpansionCountMiner):
    def reset(self) -> None:
        self.expansion_counts: Dict[str, int] = {}

    def add_coverage(self, symbol: str, children: List[DerivationTree]) -> None:
        key = expansion_key(symbol, children)

        if self.log:
            print("Found", key)

        if key not in self.expansion_counts:
            self.expansion_counts[key] = 0
        self.expansion_counts[key] += 1

    def add_tree(self, tree: DerivationTree) -> None:
        (symbol, children) = tree
        if not is_nonterminal(symbol):
            return
        assert children is not None

        direct_children: List[DerivationTree] = [
            (symbol, None) if is_nonterminal(symbol) 
            else (symbol, []) for symbol, c in children]
        self.add_coverage(symbol, direct_children)

        for c in children:
            self.add_tree(c) 
```

方法`count_expansions()`是面向公众的；它接受一个输入列表，解析它们，并处理生成的树。方法`counts()`返回找到的计数。

```py
class ExpansionCountMiner(ExpansionCountMiner):
    def count_expansions(self, inputs: List[str]) -> None:
        for inp in inputs:
            tree, *_ = self.parser.parse(inp)
            self.add_tree(tree)

    def counts(self) -> Dict[str, int]:
        return self.expansion_counts 
```

让我们在我们的 IP 地址语法上试一试。我们为我们的 IP 地址语法创建一个`ExpansionCountMiner`：

```py
expansion_count_miner = ExpansionCountMiner(EarleyParser(IP_ADDRESS_GRAMMAR)) 
```

我们解析一组（小的）IP 地址并计数发生的扩展：

```py
expansion_count_miner.count_expansions(["127.0.0.1", "1.2.3.4"])
expansion_count_miner.counts() 
```

```py
{'<start> -> <address>': 2,
 '<address> -> <octet>.<octet>.<octet>.<octet>': 2,
 '<octet> -> 127': 1,
 '<octet> -> 0': 2,
 '<octet> -> 1': 2,
 '<octet> -> 2': 1,
 '<octet> -> 3': 1,
 '<octet> -> 4': 1}

```

你可以看到我们有一个扩展到`127`，两个扩展到`0`。这些是我们可以用来自定义概率的计数。

### 分配概率

由`ExpansionCountMiner`确定的计数分布是我们可以用来自定义我们语法的概率。为此，我们引入了一个子类`ProbabilisticGrammarMiner`，其方法`set_expansion_probabilities()`处理给定符号的所有扩展，检查它是否出现在给定的计数分布中，并使用以下公式分配概率。

给定一个从样本中挖掘出的推导树集合$T$，我们确定符号$S \rightarrow a_1 | \dots | a_n$的每个替代$a_i$的概率$p_i$。

$$p_i = \frac{\text{在$T$中$S \rightarrow a_i$的扩展}}{\text{在$T$中$S$的扩展}}$$

如果$S$在$T$中根本不出现，那么$p_i$是*未指定的*。

下面是`set_expansion_probabilities()`的实现，实现了上述公式：

```py
class ProbabilisticGrammarMiner(ExpansionCountMiner):
    def set_probabilities(self, counts: Dict[str, int]):
        for symbol in self.grammar:
            self.set_expansion_probabilities(symbol, counts)

    def set_expansion_probabilities(self, symbol: str, counts: Dict[str, int]):
        expansions = self.grammar[symbol]
        if len(expansions) == 1:
            set_prob(self.grammar, symbol, expansions[0], None)
            return

        expansion_counts = [
            counts.get(
                expansion_key(
                    symbol,
                    expansion),
                0) for expansion in expansions]
        total = sum(expansion_counts)
        for i, expansion in enumerate(expansions):
            p = expansion_counts[i] / total if total > 0 else None
            # if self.log:
            #     print("Setting", expansion_key(symbol, expansion), p)
            set_prob(self.grammar, symbol, expansion, p) 
```

`ProbabilisticGrammarMiner`的典型用法是通过`mine_probabilistic_grammar()`函数，它首先从一组输入中确定一个分布，然后相应地设置概率。

```py
class ProbabilisticGrammarMiner(ProbabilisticGrammarMiner):
    def mine_probabilistic_grammar(self, inputs: List[str]) -> Grammar:
        self.count_expansions(inputs)
        self.set_probabilities(self.counts())
        return self.grammar 
```

让我们将其付诸实践。我们为 IP 地址创建一个语法挖掘器：

```py
probabilistic_grammar_miner = ProbabilisticGrammarMiner(
    EarleyParser(IP_ADDRESS_GRAMMAR)) 
```

我们现在使用`mine_probabilistic_grammar()`来挖掘语法：

```py
probabilistic_ip_address_grammar = probabilistic_grammar_miner.mine_probabilistic_grammar([
                                                                                          "127.0.0.1", "1.2.3.4"]) 
```

```py
assert is_valid_probabilistic_grammar(probabilistic_ip_address_grammar) 
```

这是我们的语法中八位字节的结果分布：

```py
[expansion for expansion in probabilistic_ip_address_grammar['<octet>']
    if exp_prob(expansion) > 0] 
```

```py
[('0', {'prob': 0.25}),
 ('1', {'prob': 0.25}),
 ('2', {'prob': 0.125}),
 ('3', {'prob': 0.125}),
 ('4', {'prob': 0.125}),
 ('127', {'prob': 0.125})]

```

如果我们使用这些概率进行模糊测试，我们将获得与样本中相同的八位字节分布：

```py
probabilistic_ip_fuzzer = ProbabilisticGrammarFuzzer(
    probabilistic_ip_address_grammar)
[probabilistic_ip_fuzzer.fuzz() for i in range(10)] 
```

```py
['4.2.2.0',
 '2.1.4.0',
 '0.1.3.127',
 '0.3.0.127',
 '4.0.2.1',
 '3.127.0.0',
 '2.2.1.1',
 '4.0.1.0',
 '2.4.0.1',
 '0.0.3.127']

```

通过从样本中学习，我们可以调整我们的模糊测试，以适应这个样本的（句法）特性。

### 测试常见特性

现在我们来探讨我们的三个使用场景。第一个场景是从样本中直接创建概率分布，并在测试生成期间使用这些分布。这有助于将测试生成集中在那些*最常用*的特性上，从而最大限度地减少客户遇到失败的风险。

为了说明常见特性的测试，我们选择 URL 域名。让我们假设我们正在运行一些与 Web 相关的服务，这是我们客户访问最多的 URL 样本：

```py
URL_SAMPLE: List[str] = [
    "https://user:password@cispa.saarland:80/",
    "https://fuzzingbook.com?def=56&x89=3&x46=48&def=def",
    "https://cispa.saarland:80/def?def=7&x23=abc",
    "https://fuzzingbook.com:80/",
    "https://fuzzingbook.com:80/abc?def=abc&abc=x14&def=abc&abc=2&def=38",
    "ftps://fuzzingbook.com/x87",
    "https://user:password@fuzzingbook.com:6?def=54&x44=abc",
    "http://fuzzingbook.com:80?x33=25&def=8",
    "http://fuzzingbook.com:8080/def",
] 
```

使用关于解析器的章节中的 Earley 解析器，我们可以将这些输入解析成解析树；不过，我们必须要指定一个标记集。

```py
URL_TOKENS: Set[str] = {"<scheme>", "<userinfo>", "<host>", "<port>", "<id>"} 
```

```py
url_parser = EarleyParser(URL_GRAMMAR, tokens=URL_TOKENS)
url_input = URL_SAMPLE[2]
print(url_input)
tree, *_ = url_parser.parse(url_input)
display_tree(tree) 
```

```py
https://cispa.saarland:80/def?def=7&x23=abc

```

<svg width="536pt" height="374pt" viewBox="0.00 0.00 536.38 374.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 370)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="151.88" y="-352.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="151.88" y="-302.45" font-family="Times,serif" font-size="14.00"><url></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="28.88" y="-252.2" font-family="Times,serif" font-size="14.00"><scheme></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="95.88" y="-252.2" font-family="Times,serif" font-size="14.00">://</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="151.88" y="-252.2" font-family="Times,serif" font-size="14.00"><authority></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="229.88" y="-252.2" font-family="Times,serif" font-size="14.00"><path></text></g> <g id="edge11" class="edge"><title>1->11</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="326.88" y="-252.2" font-family="Times,serif" font-size="14.00"><query></text></g> <g id="edge15" class="edge"><title>1->15</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="21.88" y="-201.95" font-family="Times,serif" font-size="14.00">https</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="72.88" y="-201.95" font-family="Times,serif" font-size="14.00"><host></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="125.88" y="-201.95" font-family="Times,serif" font-size="14.00">: (58)</text></g> <g id="edge8" class="edge"><title>5->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="177.88" y="-201.95" font-family="Times,serif" font-size="14.00"><port></text></g> <g id="edge9" class="edge"><title>5->9</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="72.88" y="-151.7" font-family="Times,serif" font-size="14.00">cispa.saarland</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="177.88" y="-151.7" font-family="Times,serif" font-size="14.00">80</text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="229.88" y="-201.95" font-family="Times,serif" font-size="14.00">/ (47)</text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="276.88" y="-201.95" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge13" class="edge"><title>11->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="275.88" y="-151.7" font-family="Times,serif" font-size="14.00">def</text></g> <g id="edge14" class="edge"><title>13->14</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="326.88" y="-201.95" font-family="Times,serif" font-size="14.00">? (63)</text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="389.88" y="-201.95" font-family="Times,serif" font-size="14.00"><params></text></g> <g id="edge17" class="edge"><title>15->17</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="327.88" y="-151.7" font-family="Times,serif" font-size="14.00"><param></text></g> <g id="edge18" class="edge"><title>17->18</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="389.88" y="-151.7" font-family="Times,serif" font-size="14.00">& (38)</text></g> <g id="edge25" class="edge"><title>17->25</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="460.88" y="-151.7" font-family="Times,serif" font-size="14.00"><params></text></g> <g id="edge26" class="edge"><title>17->26</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="276.88" y="-101.45" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge19" class="edge"><title>18->19</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="325.88" y="-101.45" font-family="Times,serif" font-size="14.00">= (61)</text></g> <g id="edge21" class="edge"><title>18->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="377.88" y="-101.45" font-family="Times,serif" font-size="14.00"><nat></text></g> <g id="edge22" class="edge"><title>18->22</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="276.88" y="-51.2" font-family="Times,serif" font-size="14.00">def</text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="364.88" y="-51.2" font-family="Times,serif" font-size="14.00"><digit></text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="364.88" y="-0.95" font-family="Times,serif" font-size="14.00">7 (55)</text></g> <g id="edge24" class="edge"><title>23->24</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="462.88" y="-101.45" font-family="Times,serif" font-size="14.00"><param></text></g> <g id="edge27" class="edge"><title>26->27</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="416.88" y="-51.2" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge28" class="edge"><title>27->28</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="465.88" y="-51.2" font-family="Times,serif" font-size="14.00">= (61)</text></g> <g id="edge30" class="edge"><title>27->30</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="514.88" y="-51.2" font-family="Times,serif" font-size="14.00"><id></text></g> <g id="edge31" class="edge"><title>27->31</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="416.88" y="-0.95" font-family="Times,serif" font-size="14.00">x23</text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="514.88" y="-0.95" font-family="Times,serif" font-size="14.00">abc</text></g> <g id="edge32" class="edge"><title>31->32</title></g></g></svg>

让我们将我们的`ProbabilisticGrammarMiner`类应用于这些输入，使用上述`url_parser`解析器，并获取一个概率 URL 语法：

```py
probabilistic_grammar_miner = ProbabilisticGrammarMiner(url_parser)
probabilistic_url_grammar = probabilistic_grammar_miner.mine_probabilistic_grammar(
    URL_SAMPLE) 
```

这些是我们解析过程中获得的数据：

```py
print(probabilistic_grammar_miner.counts()) 
```

```py
{'<start> -> <url>': 9, '<url> -> <scheme>://<authority><path><query>': 9, '<scheme> -> https': 6, '<authority> -> <userinfo>@<host>:<port>': 2, '<userinfo> -> user:password': 2, '<host> -> cispa.saarland': 2, '<port> -> 80': 5, '<path> -> /': 2, '<query> -> <query>': 4, '<authority> -> <host>': 2, '<host> -> fuzzingbook.com': 7, '<path> -> <path>': 3, '<query> -> ?<params>': 5, '<params> -> <param>&<params>': 10, '<param> -> <id>=<nat>': 9, '<id> -> def': 11, '<nat> -> <digit><digit>': 5, '<digit> -> 5': 3, '<digit> -> 6': 1, '<id> -> x89': 1, '<nat> -> <digit>': 4, '<digit> -> 3': 2, '<id> -> x46': 1, '<digit> -> 4': 2, '<digit> -> 8': 3, '<params> -> <param>': 5, '<param> -> <id>=<id>': 6, '<authority> -> <host>:<port>': 5, '<path> -> /<id>': 4, '<digit> -> 7': 1, '<id> -> x23': 1, '<id> -> abc': 7, '<id> -> x14': 1, '<digit> -> 2': 2, '<scheme> -> ftps': 1, '<id> -> x87': 1, '<port> -> 6': 1, '<id> -> x44': 1, '<scheme> -> http': 2, '<id> -> x33': 1, '<port> -> 8080': 1}

```

这些计数转换成个别概率。我们看到在我们的样本中，大多数 URL 使用`https:`方案，而没有输入使用`ftp:`方案。

```py
probabilistic_url_grammar['<scheme>'] 
```

```py
[('http', {'prob': 0.2222222222222222}),
 ('https', {'prob': 0.6666666666666666}),
 ('ftp', {'prob': 0.0}),
 ('ftps', {'prob': 0.1111111111111111})]

```

同样，我们看到大多数给定的 URL 都有多个参数：

```py
probabilistic_url_grammar['<params>'] 
```

```py
[('<param>', {'prob': 0.3333333333333333}),
 ('<param>&<params>', {'prob': 0.6666666666666666})]

```

当我们使用这种概率语法进行模糊测试时，这些分布反映在我们生成的输入中——没有`ftp:`方案，并且大多数输入都有多个参数。

```py
g = ProbabilisticGrammarFuzzer(probabilistic_url_grammar)
[g.fuzz() for i in range(10)] 
```

```py
['https://fuzzingbook.com/def?def=abc&def=abc&def=def&def=abc&abc=def&def=def&abc=def',
 'http://fuzzingbook.com:80/def?def=7&def=abc&abc=88',
 'https://cispa.saarland/def?def=2',
 'http://user:password@fuzzingbook.com:80/def?abc=abc&def=78',
 'http://cispa.saarland:80/?def=54&def=abc',
 'https://fuzzingbook.com:80/def?def=def',
 'http://fuzzingbook.com:80/abc?abc=abc&abc=def&def=85&abc=7&def=6&abc=2&def=abc',
 'https://fuzzingbook.com/abc?def=32&def=3&abc=4',
 'http://fuzzingbook.com/def?abc=24&def=def&def=48',
 'https://cispa.saarland:80/?abc=abc']

```

能够复制从样本中学习到的概率分布不仅对于关注常用特性很重要。它还可以帮助实现*有效输入*，特别是如果一个人学习概率*在上下文中*，如上所述：如果在给定上下文中，某些元素比其他元素更有可能（因为它们相互依赖），学习到的概率分布将反映这一点；因此，从这个学习到的概率分布生成的输入将有更高的可能性是有效的。我们将在下面的练习中进一步探讨这一点。

### 测试不常见特性

到目前为止，我们一直关注*常见*特性；但从测试的角度来看，也可以测试*不常见*特性——也就是说，这些特性在我们的使用样本中很少出现，因此在实践中练习得较少。这在安全测试中是一个常见的场景，其中人们关注不常见（可能还不太为人所知）的特性，因为用户较少意味着报告的漏洞较少，因此还有更多的漏洞被发现和利用。

为了让我们的概率语法模糊器专注于*不常见*特征，我们*改变学习到的概率*，使得常见出现的特征（即那些具有高学习概率的特征）得到低概率，反之亦然：最后的成为第一，第一成为最后。实现这种概率*逆运算*的一个特别简单的方法是*交换*它们：概率最高和最低的备选方案交换它们的概率，以此类推，概率第二高和第二低的备选方案，概率第三高和最低的备选方案，等等。

函数`invert_expansion()`从一个语法中接受一个扩展（备选方案的列表），并返回一个新的逆扩展，其中概率根据上述规则进行了交换。它创建一个索引列表，按增加的概率排序，然后为每个第$n$个元素分配索引中第$n$个最后一个元素的概率。

```py
import [copy](https://docs.python.org/3/library/copy.html) 
```

```py
def invert_expansion(expansion: List[Expansion]) -> List[Expansion]:
    def sort_by_prob(x: Tuple[int, float]) -> float:
        index, prob = x
        return prob if prob is not None else 0.0

    inverted_expansion: List[Expansion] = copy.deepcopy(expansion)
    indexes_and_probs = [(index, exp_prob(alternative))
                         for index, alternative in enumerate(expansion)]
    indexes_and_probs.sort(key=sort_by_prob)
    indexes = [i for (i, _) in indexes_and_probs]

    for j in range(len(indexes)):
        k = len(indexes) - 1 - j
        # print(indexes[j], "gets", indexes[k])
        inverted_expansion[indexes[j]][1]['prob'] = expansion[indexes[k]][1]['prob']

    return inverted_expansion 
```

下面是`invert_expansion()`函数的作用示例。这是我们原始的 URL 方案的概率分布：

```py
probabilistic_url_grammar['<scheme>'] 
```

```py
[('http', {'prob': 0.2222222222222222}),
 ('https', {'prob': 0.6666666666666666}),
 ('ftp', {'prob': 0.0}),
 ('ftps', {'prob': 0.1111111111111111})]

```

这是"逆"分布。我们看到，之前概率为零的`ftp:`方案现在具有最高的概率，而最常用的方案`https:`现在具有之前`ftp:`方案的零概率。

```py
invert_expansion(probabilistic_url_grammar['<scheme>']) 
```

```py
[('http', {'prob': 0.1111111111111111}),
 ('https', {'prob': 0.0}),
 ('ftp', {'prob': 0.6666666666666666}),
 ('ftps', {'prob': 0.2222222222222222})]

```

这种概率交换的一个优点是概率的总和保持不变；不需要归一化。另一个优点是，逆运算的逆运算会返回原始分布：

```py
invert_expansion(invert_expansion(probabilistic_url_grammar['<scheme>'])) 
```

```py
[('http', {'prob': 0.2222222222222222}),
 ('https', {'prob': 0.6666666666666666}),
 ('ftp', {'prob': 0.0}),
 ('ftps', {'prob': 0.1111111111111111})]

```

注意，我们的实现并不普遍满足这个属性：如果扩展中的两个备选方案$a_1$和$a_2$具有相同的概率，那么第二次逆运算可能会将不同的概率分配给$a_1$和$a_2$。

我们可以在整个语法中应用这种扩展的逆运算：

```py
def invert_probs(grammar: Grammar) -> Grammar:
    inverted_grammar = extend_grammar(grammar)
    for symbol in grammar:
        inverted_grammar[symbol] = invert_expansion(grammar[symbol])
    return inverted_grammar 
```

这意味着每个扩展都会交换概率：

```py
probabilistic_url_grammar["<digit>"] 
```

```py
[('0', {'prob': 0.0}),
 ('1', {'prob': 0.0}),
 ('2', {'prob': 0.14285714285714285}),
 ('3', {'prob': 0.14285714285714285}),
 ('4', {'prob': 0.14285714285714285}),
 ('5', {'prob': 0.21428571428571427}),
 ('6', {'prob': 0.07142857142857142}),
 ('7', {'prob': 0.07142857142857142}),
 ('8', {'prob': 0.21428571428571427}),
 ('9', {'prob': 0.0})]

```

```py
inverted_probabilistic_url_grammar = invert_probs(probabilistic_url_grammar)
inverted_probabilistic_url_grammar["<digit>"] 
```

```py
[('0', {'prob': 0.21428571428571427}),
 ('1', {'prob': 0.21428571428571427}),
 ('2', {'prob': 0.07142857142857142}),
 ('3', {'prob': 0.07142857142857142}),
 ('4', {'prob': 0.0}),
 ('5', {'prob': 0.0}),
 ('6', {'prob': 0.14285714285714285}),
 ('7', {'prob': 0.14285714285714285}),
 ('8', {'prob': 0.0}),
 ('9', {'prob': 0.14285714285714285})]

```

如果我们现在使用这种"逆"语法进行模糊测试，生成的输入将专注于*输入样本的补集*。我们将得到大量关于用户/密码特征的测试，以及`ftp:`方案——本质上，我们语言中所有存在的特征，但在我们的输入样本中（如果有的话）很少使用。

```py
g = ProbabilisticGrammarFuzzer(inverted_probabilistic_url_grammar)
[g.fuzz() for i in range(10)] 
```

```py
['ftp://www.google.com',
 'ftp://www.google.com/',
 'ftp://www.google.com/',
 'ftp://user:password@cispa.saarland',
 'ftps://www.google.com/',
 'ftp://user:password@www.google.com/',
 'ftp://user:password@www.google.com',
 'ftp://www.google.com/',
 'ftp://user:password@www.google.com',
 'ftp://user:password@www.google.com/']

```

除了只有常见或只有不常见特征之外，还可以创建混合形式——例如，在常见环境中测试不常见特征。这在安全测试中可能很有用，其中可能需要一个无害的（常见）"信封"与一个（不常见）"有效载荷"相结合。这完全取决于我们在哪里以及如何调整概率。

### 从输入片段中学习概率

在我们之前的例子中，我们已经从所有输入中学习以生成常见或不常见的输入。然而，我们也可以从输入的子集中学习，以关注该子集中存在的特征（或者相反，避免其特征）。例如，如果我们知道某些输入子集涵盖了某个感兴趣的功能（比如，因为它特别关键或因为最近进行了更改），我们可以从这个子集学习，并将测试生成集中在其特征上。

为了说明这种方法，让我们使用在覆盖率章节中引入的 CGI 语法。在我们的 CGI 解码器中，我们对第 25 行特别感兴趣——即处理一个`%`字符后跟两个有效十六进制数字的行：

```py
...
        elif c == '%':
            digit_high, digit_low = s[i + 1], s[i + 2]
            i += 2
            if digit_high in hex_values and digit_low in hex_values:
                v = hex_values[digit_high] * 16 + hex_values[digit_low] ### Line 25
                t += chr(v)
        ... 
```

假设我们并不确切知道第 25 行在什么条件下被执行——但即便如此，我们仍然希望对其进行彻底测试。然而，借助我们的概率学习工具，我们可以学习这些条件。我们从一组随机输入开始，并考虑涵盖第 25 行的子集。

```py
from Coverage import Coverage, cgi_decode
from Grammars import CGI_GRAMMAR 
```

```py
cgi_fuzzer = GrammarFuzzer(CGI_GRAMMAR) 
```

```py
trials = 100
coverage = {}

for i in range(trials):
    cgi_input = cgi_fuzzer.fuzz()
    with Coverage() as cov:
        cgi_decode(cgi_input)
    coverage[cgi_input] = cov.coverage() 
```

这些都是涵盖第 25 行的随机输入：

```py
coverage_slice = [cgi_input for cgi_input in coverage
                  if ('cgi_decode', 25) in coverage[cgi_input]] 
```

```py
print(coverage_slice) 
```

```py
['%36%c1%d2c2%4f++e', '%2c%90+', 'c++%8b', 'a+%95', '%76%00', '+5', '4', '+', 'a', '%71', '%a2', '%39%51%db%7c%66', '++', '%2a', 'd', '%b9225', '++c', '%13+b', '%32', '2', '+2+', '-1%b11%d8', '%08', 'd+4', '%a3', '%fe', 'e', '1++', '+%82%ed%42', '%d5', '%5bc', '51', '%b0', '%47', 'b+%20', '%d7', '0+%17', '%a5', '%84', 'e+4%fc', 'd%6f+++1a', '3', 'd+%95+', '%1e', '%244', '%3c', '5%75+%99%3c', '+-', 'b', '%80%74+a', '%a7', '21', 'ae', '%c1%da', '%c5+', 'b%44', '%70%c4_3', '1', 'dd+ad', '4%63', '%364+', '%79%ab', '%8a%f6', '%53%43', '+++%55+b5', '%51+++', '+%28', '1%1c+', '+%41%9b', '%0d%20', '+%3d+%c2']

```

实际上，大约一半的输入涵盖了第 25 行：

```py
len(coverage_slice) / trials 
```

```py
0.71

```

让我们从这些输入片段中学习一个概率语法：

```py
probabilistic_grammar_miner = ProbabilisticGrammarMiner(
    EarleyParser(CGI_GRAMMAR))
probabilistic_cgi_grammar = probabilistic_grammar_miner.mine_probabilistic_grammar(
    coverage_slice) 
```

```py
assert is_valid_probabilistic_grammar(probabilistic_cgi_grammar) 
```

我们可以看到，百分号出现的可能性非常高：

```py
probabilistic_cgi_grammar['<letter>'] 
```

```py
[('<plus>', {'prob': 0.2556818181818182}),
 ('<percent>', {'prob': 0.42045454545454547}),
 ('<other>', {'prob': 0.32386363636363635})]

```

使用这个语法，我们现在可以生成专门针对第 25 行的测试：

```py
probabilistic_cgi_fuzzer = ProbabilisticGrammarFuzzer(
    probabilistic_cgi_grammar)
print([probabilistic_cgi_fuzzer.fuzz() for i in range(20)]) 
```

```py
['5', '%3e', '+4%e6', '%19+', '%da+', '%5c', '%28%5e', '%b5+', '2d4', '455%8c', '5%cb', '%4c+%5c4e5+e+%aa%db%1d', 'a%1c%13', 'e+', '%08%cc', 'b3', '3', '%c0%25', '+', '++c+%54d']

```

```py
trials = 100
coverage = {}

for i in range(trials):
    cgi_input = probabilistic_cgi_fuzzer.fuzz()
    with Coverage() as cov:
        cgi_decode(cgi_input)
    coverage[cgi_input] = cov.coverage() 
```

我们可以看到，涵盖第 25 行的输入比例已经很高，这表明我们的聚焦是有效的：

```py
coverage_slice: List[str] = [cgi_input for cgi_input in coverage
                             if ('cgi_decode', 25) in coverage[cgi_input]] 
```

```py
len(coverage_slice) / trials 
```

```py
0.88

```

再重复一次，可以进一步提高聚焦度：

```py
for run in range(3):
    probabilistic_cgi_grammar = probabilistic_grammar_miner.mine_probabilistic_grammar(
        coverage_slice)
    probabilistic_cgi_fuzzer = ProbabilisticGrammarFuzzer(
        probabilistic_cgi_grammar)

    trials = 100
    coverage = {}

    for i in range(trials):
        cgi_input = probabilistic_cgi_fuzzer.fuzz()
        with Coverage() as cov:
            cgi_decode(cgi_input)
        coverage[cgi_input] = cov.coverage()

    coverage_slice = [cgi_input for cgi_input in coverage
                      if ('cgi_decode', 25) in coverage[cgi_input]] 
```

```py
len(coverage_slice) / trials 
```

```py
0.83

```

通过从样本输入的子集中学习（和重新学习）概率，我们可以将模糊器**专门化**到该子集的特性——在我们的案例中，包含百分号和有效十六进制字母的输入。我们可以专门化事物的程度是由我们可以控制的变量数量决定的——在我们的案例中，是单个规则的概率。正如上面所讨论的，添加更多上下文到语法中，将增加变量的数量，从而增加专门化的程度。

然而，高度的专业化限制了我们在选定范围之外探索组合的可能性，并限制了发现由这些组合引起的错误的可能性。这种权衡在机器学习中被称为“探索与利用”——是尝试尽可能探索（可能浅显的）多种组合，还是专注于（利用）特定区域？最终，一切都取决于错误在哪里，以及我们最有可能在哪里找到它们。分配和学习概率使我们能够控制搜索策略——从常见到不常见再到特定的子集。

## 检测不自然数字

让我们通过回到我们的入门示例来结束这一章。我们说过，本福特定律不仅允许我们生成，还允许我们检测“不自然”的首位数字分布，例如简单随机选择产生的那些。

如果我们使用忽略概率的常规 `GrammarFuzzer` 类（随机）生成首位数字，这将是我们为每个首位数字得到的分布：

```py
sample_size = 1000
random_integer_fuzzer = GrammarFuzzer(
    PROBABILISTIC_EXPR_GRAMMAR,
    start_symbol="<leaddigit>")
random_integers = [random_integer_fuzzer.fuzz() for i in range(sample_size)] 
```

```py
random_counts = [random_integers.count(str(c)) for c in crange('1', '9')]
random_counts 
```

```py
[112, 104, 106, 103, 133, 97, 126, 120, 99]

```

（为了简单起见，我们在这里使用简单的列表 `count()` 方法，而不是部署完整的 `ProbabilisticGrammarMiner`。）

如果我们有一个自然分布的首位数字，这是我们预期的：

```py
expected_prob_counts = [
    exp_prob(
        PROBABILISTIC_EXPR_GRAMMAR["<leaddigit>"][i]) *
    sample_size for i in range(9)]
print(expected_prob_counts) 
```

```py
[301.0, 176.0, 125.0, 97.0, 79.0, 67.0, 58.0, 51.0, 46.0]

```

如果我们有一个随机分布，我们预期会有一个均匀分布：

```py
expected_random_counts = [sample_size / 9 for i in range(9)]
print(expected_random_counts) 
```

```py
[111.11111111111111, 111.11111111111111, 111.11111111111111, 111.11111111111111, 111.11111111111111, 111.11111111111111, 111.11111111111111, 111.11111111111111, 111.11111111111111]

```

哪个分布更好地匹配我们的 `random_counts` 首位数字？为此，我们运行 $\chi²$-测试来比较我们找到的分布（`random_counts`）与“自然”首位数字分布 `expected_prob_counts` 和随机分布 `expected_random_counts`。

```py
from [scipy.stats](https://docs.scipy.org/doc/scipy/reference/) import chisquare 
```

结果表明，观察到的分布遵循“自然”分布的概率为零（`pvalue` = 0.0）：

```py
chisquare(random_counts, expected_prob_counts) 
```

```py
Power_divergenceResult(statistic=np.float64(435.87462280458345), pvalue=np.float64(3.925216460200427e-89))

```

然而，有 97%的概率，观察到的行为遵循随机分布：

```py
chisquare(random_counts, expected_random_counts) 
```

```py
Power_divergenceResult(statistic=np.float64(11.42), pvalue=np.float64(0.17901776899017763))

```

因此，如果你发现一些发布的数据并对其有效性表示怀疑，你可以运行上述测试来检查它们是否可能是自然的。更好的是，坚持要求作者使用 Jupyter 笔记本来生成他们的结果，这样你就可以检查计算的每一步 :-)

## 经验教训

+   通过指定概率，可以将模糊测试引导到感兴趣的输入特征。

+   从样本中学习概率允许人们专注于在输入样本中常见或不常见的特征。

+   从样本的子集中学习概率允许人们产生更相似的输入。

## 下一步

现在我们已经将概率和语法（以及回顾了解析器和语法）结合起来，我们为许多应用奠定了基础。我们接下来的章节将专注于

+   如何[最小化]失败输入（Reducer.html）

+   如何在函数级别上切割和生成测试

+   如何自动测试（Web）用户界面

享受吧！

## 背景

通过解析数据语料库来挖掘概率的想法首次在“学习模糊测试：使用输入数据的概率生成模型进行独立的应用程序模糊测试”[[Patra 等人，2016](http://mp.binaervarianz.de/TreeFuzz_TR_Nov2016.pdf)]中被提出，该研究还学习了应用概率规则进行派生树。首先在“来自地狱的输入：从常见样本生成不常见输入”[[Esteban Pavese 等人，2018](http://arxiv.org/abs/1812.07525)]中应用了这个想法，并反转概率或从切片中学习。

我们对贝纳德定律的阐述遵循[这篇文章](https://brilliant.org/wiki/benfords-law/)。

## 练习

### 练习 1：具有覆盖率的概率模糊测试

创建一个扩展 `GrammarCoverageFuzzer` 的 `ProbabilisticGrammarCoverageFuzzer` 类，它具有概率能力。想法是首先覆盖所有未覆盖的扩展（如 `GrammarCoverageFuzzer`），一旦所有扩展都被覆盖，就按概率（如 `ProbabilisticGrammarFuzzer`）进行。

为了达到这个目的，定义新的 `choose_covered_node_expansion()` 和 `choose_uncovered_node_expansion()` 方法实例，这些方法基于给定的权重选择一个扩展。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ProbabilisticGrammarFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

如果你是一个高级程序员，通过从 `GrammarCoverageFuzzer` 和 `ProbabilisticGrammarFuzzer` 的 *多重继承* 来实现这个类以实现这一点。

多重继承是一个棘手的问题。如果你有两个类 $A'$ 和 $A''$，它们都从 $A$ 继承，$A$ 的相同方法 $m()` 可能会在 $A'$ 和 $A''$ 中被重载。如果你现在从 *两个* $A'$ 和 $A''$ 继承，并调用 $m()`，应该调用哪个 $m()` 实现呢？Python 通过简单地调用继承的第一个类中的 $m()` 方法来“解决”这个冲突。

为了避免此类冲突，可以检查继承的顺序是否会影响结果。`inheritance_conflicts()` 方法会相互比较属性；如果它们指向不同的代码，你必须解决冲突。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ProbabilisticGrammarFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import inheritance_conflicts 
```

```py
inheritance_conflicts(GrammarCoverageFuzzer, ProbabilisticGrammarFuzzer) 
```

```py
['choose_node_expansion']

```

除了 `choose_covered_node_expansion()` 和 `choose_uncovered_node_expansion()` 之外，你还需要实现这个方法以支持多重继承。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ProbabilisticGrammarFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 2：从过去的错误中学习

如果从已知之前导致失败的输入中学习，从一组输入中学习可以非常有价值。在这个练习中，你将学习从过去的安全漏洞中学习分布。

1.  下载 [js-vuln-db](https://github.com/tunz/js-vuln-db)，一组 JavaScript 引擎漏洞。每个漏洞都附带用于测试它的代码。

1.  使用 `re.findall()` 和适当的正则表达式从代码中提取所有 *数字字面量*。

1.  将这些字面量转换为（十进制）*数值*并计算它们各自的 occurrence。

1.  创建一个语法 `RISKY_NUMBERS`，以反映上述计数的概率来生成这些数字。

当然，漏洞不仅仅是特定的数字，但有些数字比其他数字更有可能引起错误。下次你对系统进行模糊测试时，不要随机生成数字；相反，从 `RISKY_NUMBERS` 中选择一个 :-)

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ProbabilisticGrammarFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)的许可。 [最后修改：2024-11-09 17:07:29+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/ProbabilisticGrammarFuzzer.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用这篇作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[概率语法模糊测试](https://www.fuzzingbook.org/html/ProbabilisticGrammarFuzzer.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[模糊测试书籍](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/ProbabilisticGrammarFuzzer.html`](https://www.fuzzingbook.org/html/ProbabilisticGrammarFuzzer.html). Retrieved 2024-11-09 17:07:29+01:00.

```py
@incollection{fuzzingbook2024:ProbabilisticGrammarFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Probabilistic Grammar Fuzzing},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/ProbabilisticGrammarFuzzer.html}},
    note = {Retrieved 2024-11-09 17:07:29+01:00},
    url = {https://www.fuzzingbook.org/html/ProbabilisticGrammarFuzzer.html},
    urldate = {2024-11-09 17:07:29+01:00}
}

```
