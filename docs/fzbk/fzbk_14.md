# 使用语法进行模糊测试

> 原文：[`www.fuzzingbook.org/html/Grammars.html`](http://www.fuzzingbook.org/html/Grammars.html)

在"基于变异的模糊测试"章节中，我们已经看到了如何使用额外的提示——例如示例输入文件——来加速测试生成。在本章中，我们进一步发展了这个想法，通过提供程序合法输入的**规范**。通过语法指定输入允许非常系统且高效地生成测试，特别是对于复杂的输入格式。语法还作为配置模糊测试、API 模糊测试、GUI 模糊测试以及更多的基础。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('Jc8Whz0W41o') 
```

**先决条件**

+   你应该了解基本的模糊测试是如何工作的，例如从介绍模糊测试的章节中了解。

+   对于基于变异的模糊测试和覆盖率的知识目前**不需要**，但仍然推荐。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import List, Dict, Union, Any, Tuple, Optional 
```

```py
import Fuzzer 
```

## 概述

要使用本章提供的代码，请编写

```py
>>> from fuzzingbook.Grammars import <identifier> 
```

然后利用以下功能。

本章介绍了**语法**作为指定输入语言的一种简单方法，以及如何使用它们对具有语法有效输入的程序进行测试。语法被定义为非终结符号到一系列替代扩展的映射，如下例所示：

```py
>>> US_PHONE_GRAMMAR: Grammar = {
>>>     "<start>": ["<phone-number>"],
>>>     "<phone-number>": ["(<area>)<exchange>-<line>"],
>>>     "<area>": ["<lead-digit><digit><digit>"],
>>>     "<exchange>": ["<lead-digit><digit><digit>"],
>>>     "<line>": ["<digit><digit><digit><digit>"],
>>>     "<lead-digit>": ["2", "3", "4", "5", "6", "7", "8", "9"],
>>>     "<digit>": ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
>>> }
>>> 
>>> assert is_valid_grammar(US_PHONE_GRAMMAR) 
```

非终结符号用尖括号括起来（例如，`<digit>`）。要从语法生成一个输入字符串，一个**生成器**从起始符号（`<start>`）开始，并随机选择这个符号的随机扩展。它继续这个过程，直到所有非终结符号都被扩展。`simple_grammar_fuzzer()`函数正是这样做的：

```py
>>> [simple_grammar_fuzzer(US_PHONE_GRAMMAR) for i in range(5)]
['(692)449-5179',
 '(519)230-7422',
 '(613)761-0853',
 '(979)881-3858',
 '(810)914-5475'] 
```

实际上，虽然你应该使用 GrammarFuzzer 类或其基于覆盖率的、基于概率的或基于生成器的衍生类之一，而不是`simple_grammar_fuzzer()`；这些更高效，可以防止无限增长，并提供几个额外的功能。

本章还介绍了一个语法工具箱，其中包含几个辅助函数，这些函数可以简化语法的编写，例如使用字符类和重复的快捷符号，或扩展语法。

## 输入语言

程序的所有可能行为都可以通过其输入触发。"输入"在这里可以是一系列可能的来源：我们谈论的是从文件、环境或网络中读取的数据，用户输入的数据，或从与其他资源的交互中获得的数据。所有这些输入的集合决定了程序将如何表现——包括其失败。在测试时，考虑可能的输入来源、如何控制它们以及**如何系统地测试它们**是非常有帮助的。

为了简化起见，我们目前假设程序只有一个输入源；这也是我们在前几章中使用的相同假设。程序的有效输入集合被称为*语言*。语言的范围从简单到复杂：CSV 语言表示有效的逗号分隔输入集合，而 Python 语言表示有效的 Python 程序集合。我们通常将数据语言和编程语言分开，尽管任何程序也可以被视为输入数据（例如，用于编译器）。[维基百科上的文件格式页面](https://en.wikipedia.org/wiki/List_of_file_formats)列出了超过 1,000 种不同的文件格式，每种格式都是其自己的语言。

为了正式描述语言，*形式语言*领域已经设计了一系列*语言规范*，这些规范描述了一种语言。*正则表达式*代表了这些语言中最简单的一类，用于表示字符串集合：例如，正则表达式`[a-z]*`表示一个（可能为空）的小写字母序列。*自动机理论*将这些语言与接受这些输入的自动机联系起来；例如，*有限状态机*可以用来指定正则表达式的语言。

正则表达式非常适合不太复杂的输入格式，与之相关的有限状态机也具有许多使它们非常适合推理的性质。然而，为了指定更复杂的输入，它们很快就会遇到限制。在语言谱的另一端，我们有*通用语法*，它表示由*图灵机*接受的语言。图灵机可以计算任何可以计算的东西；由于 Python 是图灵完备的，这意味着我们也可以使用 Python 程序$p$来指定或甚至枚举合法的输入。但是，计算机科学理论也告诉我们，每个这样的测试程序都必须为要测试的程序专门编写，这不是我们想要的自动化水平。

## 语法

正则表达式和图灵机之间的中间地带由*语法*覆盖。语法是正式指定输入语言中最受欢迎（且理解最好）的形式化方法之一。使用语法，可以表达输入语言的各种属性。语法特别适合表达输入的*句法结构*，并且是表达嵌套或递归输入的首选形式化方法。我们使用的语法是所谓的*上下文无关语法*，这是一种最容易且最受欢迎的语法形式化方法。

### 规则和扩展

语法由一个*起始符号*和一组*扩展规则*（或简单地称为*规则*）组成，这些规则表明起始符号（和其他符号）如何进行扩展。例如，考虑以下语法，表示两个数字的序列：

```py
<start> ::= <digit><digit>
<digit> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

读取这样的语法时，从起始符号（`<start>`）开始。一个展开规则`<A> ::= <B>`意味着左侧的符号（`<A>`）可以被右侧的字符串（`<B>`）替换。在上面的语法中，`<start>`将被替换为`<digit><digit>`。

在这个字符串中，`<digit>`将被替换为`<digit>`规则右侧的字符串。特殊运算符`|`表示*展开替代项*（或简称为*替代项*），意味着可以选择任何数字进行展开。因此，每个`<digit>`将被展开为给定的数字之一，最终得到一个介于`00`和`99`之间的字符串。对于`0`到`9`没有进一步的展开，所以我们已经准备好了。

关于语法的有趣之处在于它们可以是*递归的*。也就是说，展开可以使用之前展开的符号——然后这些符号将被再次展开。作为一个例子，考虑一个描述整数的语法：

```py
<start>  ::= <integer>
<integer> ::= <digit> | <digit><integer>
<digit>   ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

在这里，一个`<integer>`要么是一个单独的数字，要么是一个数字后面跟着另一个整数。因此，数字`1234`将被表示为一个单独的数字`1`，后面跟着整数`234`，这个整数又是一个数字`2`，后面跟着整数`34`。

如果我们想要表达一个整数可以由一个符号（`+`或`-`） precede，我们就会把语法写成

```py
<start>   ::= <number>
<number>  ::= <integer> | +<integer> | -<integer>
<integer> ::= <digit> | <digit><integer>
<digit>   ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

这些规则正式定义了语言：可以从起始符号推导出的任何内容都是语言的一部分；不能推导出的则不是。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import quiz 
```

### 习题

哪些字符串不能由上述`<start>`符号产生？

### 算术表达式

让我们扩展我们的语法以涵盖完整的*算术表达式*——这是一个语法的典型示例。我们看到一个表达式（`<expr>`）要么是一个和，要么是一个差，要么是一个项；一个项要么是一个乘法，要么是一个除法，要么是一个因子；一个因子要么是一个数字，要么是一个括号表达式。几乎所有的规则都可以有递归性，从而允许任意复杂的表达式，例如`(1 + 2) * (3.4 / 5.6 - 789)`。

```py
<start>   ::= <expr>
<expr>    ::= <term> + <expr> | <term> - <expr> | <term>
<term>    ::= <term> * <factor> | <term> / <factor> | <factor>
<factor>  ::= +<factor> | -<factor> | (<expr>) | <integer> | <integer>.<integer>
<integer> ::= <digit><integer> | <digit>
<digit>   ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

在这样的语法中，如果我们从`<start>`开始，然后逐个展开符号，随机选择替代项，我们可以快速产生一个有效的算术表达式，然后是另一个。这种*语法模糊化*在产生复杂输入时非常有效，这正是我们将在本章中实现的。

### 习题

哪些字符串不能由上述`<start>`符号产生？

## 在 Python 中表示语法

构建语法模糊器时的第一步是找到一个合适的语法格式。为了使语法的编写尽可能简单，我们使用基于字符串和列表的格式。我们的 Python 语法采用符号名称和展开之间的*映射*格式，其中展开是*列表*形式的替代项。因此，一个用于数字的单规则语法具有以下形式

```py
DIGIT_GRAMMAR = {
    "<start>":
        ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
} 
```

<details id="Excursion:-A-Grammar-Type"><summary>一个`Grammar`类型</summary>

让我们定义一个语法类型，以便我们可以静态地检查语法类型。

语法类型的第一次尝试可能是这样的：每个符号（字符串）映射到一个展开列表（字符串）：

```py
SimpleGrammar = Dict[str, List[str]] 
```

然而，我们的 `opts()` 功能，用于添加可选属性，我们将在本章后面介绍，也允许展开成为由字符串和选项组成的配对，其中选项是将字符串映射到值的映射：

```py
Option = Dict[str, Any] 
```

因此，一个展开要么是一个字符串——要么是一个字符串和选项的配对。

```py
Expansion = Union[str, Tuple[str, Option]] 
```

使用这个，我们现在可以定义一个 `Grammar`，它将字符串映射到 `Expansion` 列表。</details>

我们可以在 *`Grammar`* 类型中捕获语法结构，其中每个符号（字符串）映射到一个展开列表（字符串）：

```py
Grammar = Dict[str, List[Expansion]] 
```

使用这种 `Grammar` 类型，算术表达式的完整语法看起来是这样的：

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

在语法中，每个符号只能定义一次。我们可以通过其符号访问任何规则...

```py
EXPR_GRAMMAR["<digit>"] 
```

```py
['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']

```

...并且我们可以检查一个符号是否在语法中：

```py
"<identifier>" in EXPR_GRAMMAR 
```

```py
False

```

注意，我们假设规则左侧（即映射中的键）始终是单个符号。这是赋予我们的语法以 *上下文无关* 特性的属性。

## 一些定义

我们假设规范起始符号是 `<start>`：

```py
START_SYMBOL = "<start>" 
```

方便的 `nonterminals()` 函数可以从一个展开中提取非终结符符号列表（即，`<` 和 `>` 之间的任何内容，除了空格）。

```py
import [re](https://docs.python.org/3/library/re.html) 
```

```py
RE_NONTERMINAL = re.compile(r'(<[^<> ]*>)') 
```

```py
def nonterminals(expansion):
    # In later chapters, we allow expansions to be tuples,
    # with the expansion being the first element
    if isinstance(expansion, tuple):
        expansion = expansion[0]

    return RE_NONTERMINAL.findall(expansion) 
```

```py
assert nonterminals("<term> * <factor>") == ["<term>", "<factor>"]
assert nonterminals("<digit><integer>") == ["<digit>", "<integer>"]
assert nonterminals("1 < 3 > 2") == []
assert nonterminals("1 <3> 2") == ["<3>"]
assert nonterminals("1 + 2") == []
assert nonterminals(("<1>", {'option': 'value'})) == ["<1>"] 
```

同样，`is_nonterminal()` 检查某个符号是否是非终结符：

```py
def is_nonterminal(s):
    return RE_NONTERMINAL.match(s) 
```

```py
assert is_nonterminal("<abc>")
assert is_nonterminal("<symbol-1>")
assert not is_nonterminal("+") 
```

## 简单语法模糊器

让我们现在使用上述语法。我们将构建一个非常简单的语法模糊器，它从起始符号（`<start>`）开始，然后不断展开它。为了避免无限输入的展开，我们对非终结符的数量（`max_nonterminals`）设置了一个限制。此外，为了避免陷入无法进一步减少符号数量的情况，我们还限制了总的展开步骤数。

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
class ExpansionError(Exception):
    pass 
```

```py
def simple_grammar_fuzzer(grammar: Grammar, 
                          start_symbol: str = START_SYMBOL,
                          max_nonterminals: int = 10,
                          max_expansion_trials: int = 100,
                          log: bool = False) -> str:
  """Produce a string from `grammar`.
 `start_symbol`: use a start symbol other than `<start>` (default).
 `max_nonterminals`: the maximum number of nonterminals 
 still left for expansion
 `max_expansion_trials`: maximum # of attempts to produce a string
 `log`: print expansion progress if True"""

    term = start_symbol
    expansion_trials = 0

    while len(nonterminals(term)) > 0:
        symbol_to_expand = random.choice(nonterminals(term))
        expansions = grammar[symbol_to_expand]
        expansion = random.choice(expansions)
        # In later chapters, we allow expansions to be tuples,
        # with the expansion being the first element
        if isinstance(expansion, tuple):
            expansion = expansion[0]

        new_term = term.replace(symbol_to_expand, expansion, 1)

        if len(nonterminals(new_term)) < max_nonterminals:
            term = new_term
            if log:
                print("%-40s" % (symbol_to_expand + " -> " + expansion), term)
            expansion_trials = 0
        else:
            expansion_trials += 1
            if expansion_trials >= max_expansion_trials:
                raise ExpansionError("Cannot expand " + repr(term))

    return term 
```

让我们看看这个简单的语法模糊器是如何从起始符号获得算术表达式的：

```py
simple_grammar_fuzzer(grammar=EXPR_GRAMMAR, max_nonterminals=3, log=True) 
```

```py
<start> -> <expr>                        <expr>
<expr> -> <term> + <expr>                <term> + <expr>
<term> -> <factor>                       <factor> + <expr>
<factor> -> <integer>                    <integer> + <expr>
<integer> -> <digit>                     <digit> + <expr>
<digit> -> 6                             6 + <expr>
<expr> -> <term> - <expr>                6 + <term> - <expr>
<expr> -> <term>                         6 + <term> - <term>
<term> -> <factor>                       6 + <factor> - <term>
<factor> -> -<factor>                    6 + -<factor> - <term>
<term> -> <factor>                       6 + -<factor> - <factor>
<factor> -> (<expr>)                     6 + -(<expr>) - <factor>
<factor> -> (<expr>)                     6 + -(<expr>) - (<expr>)
<expr> -> <term>                         6 + -(<term>) - (<expr>)
<expr> -> <term>                         6 + -(<term>) - (<term>)
<term> -> <factor>                       6 + -(<factor>) - (<term>)
<factor> -> +<factor>                    6 + -(+<factor>) - (<term>)
<factor> -> +<factor>                    6 + -(++<factor>) - (<term>)
<term> -> <factor>                       6 + -(++<factor>) - (<factor>)
<factor> -> (<expr>)                     6 + -(++(<expr>)) - (<factor>)
<factor> -> <integer>                    6 + -(++(<expr>)) - (<integer>)
<expr> -> <term>                         6 + -(++(<term>)) - (<integer>)
<integer> -> <digit>                     6 + -(++(<term>)) - (<digit>)
<digit> -> 9                             6 + -(++(<term>)) - (9)
<term> -> <factor> * <term>              6 + -(++(<factor> * <term>)) - (9)
<term> -> <factor>                       6 + -(++(<factor> * <factor>)) - (9)
<factor> -> <integer>                    6 + -(++(<integer> * <factor>)) - (9)
<integer> -> <digit>                     6 + -(++(<digit> * <factor>)) - (9)
<digit> -> 2                             6 + -(++(2 * <factor>)) - (9)
<factor> -> +<factor>                    6 + -(++(2 * +<factor>)) - (9)
<factor> -> -<factor>                    6 + -(++(2 * +-<factor>)) - (9)
<factor> -> -<factor>                    6 + -(++(2 * +--<factor>)) - (9)
<factor> -> -<factor>                    6 + -(++(2 * +---<factor>)) - (9)
<factor> -> -<factor>                    6 + -(++(2 * +----<factor>)) - (9)
<factor> -> <integer>.<integer>          6 + -(++(2 * +----<integer>.<integer>)) - (9)
<integer> -> <digit>                     6 + -(++(2 * +----<digit>.<integer>)) - (9)
<integer> -> <digit>                     6 + -(++(2 * +----<digit>.<digit>)) - (9)
<digit> -> 1                             6 + -(++(2 * +----1.<digit>)) - (9)
<digit> -> 7                             6 + -(++(2 * +----1.7)) - (9)

```

```py
'6 + -(++(2 * +----1.7)) - (9)'

```

通过增加非终结符的限制，我们可以快速得到更长的产生式：

```py
for i in range(10):
    print(simple_grammar_fuzzer(grammar=EXPR_GRAMMAR, max_nonterminals=5)) 
```

```py
7 / +48.5
-5.9 / 9 - 4 * +-(-+++((1 + (+7 - (-1 * (++-+7.7 - -+-4.0))))) * +--4 - -(6) + 64)
8.2 - 27 - -9 / +((+9 * --2 + --+-+-((-1 * +(8 - 5 - 6)) * (-((-+(((+(4))))) - ++4) / +(-+---((5.6 - --(3 * -1.8 * +(6 * +-(((-(-6) * ---+6)) / +--(+-+-7 * (-0 * (+(((((2)) + 8 - 3 - ++9.0 + ---(--+7 / (1 / +++6.37) + (1) / 482) / +++-+0)))) * -+5 + 7.513)))) - (+1 / ++((-84)))))))) * ++5 / +-(--2 - -++-9.0)))) / 5 * --++090
1 - -3 * 7 - 28 / 9
(+9) * +-5 * ++-926.2 - (+9.03 / -+(-(-6) / 2 * +(-+--(8) / -(+1.0) - 5 + 4)) * 3.5)
8 + -(9.6 - 3 - -+-4 * +77)
-(((((++((((+((++++-((+-37))))))))))))) / ++(-(+++(+6)) * -++-(+(++(---6 * (((7)) * (1) / (-7.6 * 535338) + +256) * 0) * 0))) - 4 + +1
5.43
(9 / -405 / -23 - +-((+-(2 * (13))))) + +6 - +8 - 934
-++2 - (--+715769550) / 8 / (1)

```

注意，虽然我们的模糊器在大多数情况下都能完成任务，但它有一些缺点。

### 问答

`simple_grammar_fuzzer()` 有哪些缺点？

事实上，`simple_grammar_fuzzer()` 由于搜索和替换操作的数量庞大，效率相当低，甚至可能无法生成字符串。另一方面，实现简单，在大多数情况下都能完成任务。对于本章，我们将坚持使用它；在下一章中，我们将展示如何构建一个更高效的版本。

## 将语法可视化成铁路图

使用语法，我们可以轻松地指定我们之前讨论的几个示例的格式。例如，上述算术表达式可以直接发送到 `bc`（或任何接受算术表达式的程序）。在我们介绍一些额外的语法之前，让我们提供一种 *可视化* 语法的方法，以提供另一种辅助理解的观点。

*铁路图*，也称为 *语法图*，是上下文无关语法的图形表示。它们从左到右读取，遵循可能的“铁路”轨道；轨道上遇到的符号序列定义了语言。为了生成铁路图，我们实现了一个名为 `syntax_diagram()` 的函数。

<details id="Excursion:-Implementing-syntax_diagram()"><summary>实现 `syntax_diagram()`</summary>

我们使用 RailroadDiagrams，一个用于可视化的外部库。

```py
from RailroadDiagrams import NonTerminal, Terminal, Choice, HorizontalChoice, Sequence
from RailroadDiagrams import show_diagram 
```

```py
from [IPython.display](https://ipython.readthedocs.io/en/stable/api/generated/IPython.display.html) import SVG 
```

我们首先定义了一个名为 `syntax_diagram_symbol()` 的方法来可视化给定的符号。终结符号用椭圆形表示，而非终结符号（如 `<term>`）用矩形表示。

```py
def syntax_diagram_symbol(symbol: str) -> Any:
    if is_nonterminal(symbol):
        return NonTerminal(symbol[1:-1])
    else:
        return Terminal(symbol) 
```

```py
SVG(show_diagram(syntax_diagram_symbol('<term>'))) 
```

<svg class="railroad-diagram" height="62" viewBox="0 0 154.0 62" width="154.0"><g transform="translate(.5 .5)"><g class="non-terminal"><text x="77.0" y="35">term</text></g></g></svg>

我们定义 `syntax_diagram_expr()` 来可视化扩展替代方案。

```py
def syntax_diagram_expr(expansion: Expansion) -> Any:
    # In later chapters, we allow expansions to be tuples,
    # with the expansion being the first element
    if isinstance(expansion, tuple):
        expansion = expansion[0]

    symbols = [sym for sym in re.split(RE_NONTERMINAL, expansion) if sym != ""]
    if len(symbols) == 0:
        symbols = [""]  # special case: empty expansion

    return Sequence(*[syntax_diagram_symbol(sym) for sym in symbols]) 
```

```py
SVG(show_diagram(syntax_diagram_expr(EXPR_GRAMMAR['<term>'][0]))) 
```

<svg class="railroad-diagram" height="62" viewBox="0 0 310.5 62" width="310.5"><g transform="translate(.5 .5)"><g><g class="non-terminal"><text x="85.5" y="35">factor</text></g> <g class="terminal"><text x="163.75" y="35">*</text></g> <g class="non-terminal"><text x="233.5" y="35">term</text></g></g></g></svg>

这是 `<term>` 的第一个替代方案——一个 `<factor>` 后跟 `*` 和一个 `<term>`。

接下来，我们定义 `syntax_diagram_alt()` 来显示替代表达式。

```py
from [itertools](https://docs.python.org/3/library/itertools.html) import zip_longest 
```

```py
def syntax_diagram_alt(alt: List[Expansion]) -> Any:
    max_len = 5
    alt_len = len(alt)
    if alt_len > max_len:
        iter_len = alt_len // max_len
        alts = list(zip_longest(*[alt[i::iter_len] for i in range(iter_len)]))
        exprs = [[syntax_diagram_expr(expr) for expr in alt
                  if expr is not None] for alt in alts]
        choices = [Choice(len(expr) // 2, *expr) for expr in exprs]
        return HorizontalChoice(*choices)
    else:
        return Choice(alt_len // 2, *[syntax_diagram_expr(expr) for expr in alt]) 
```

```py
SVG(show_diagram(syntax_diagram_alt(EXPR_GRAMMAR['<digit>']))) 
```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

我们可以看到 `<digit>` 可以是 `0` 到 `9` 之间的任何单个数字。

最后，我们定义了 `syntax_diagram()`，它接受一个语法，并显示其规则的语法图。

```py
def syntax_diagram(grammar: Grammar) -> None:
    from [IPython.display](https://ipython.readthedocs.io/en/stable/api/generated/IPython.display.html) import SVG, display

    for key in grammar:
        print("%s" % key[1:-1])
        display(SVG(show_diagram(syntax_diagram_alt(grammar[key])))) 
```</details>

让我们使用 `syntax_diagram()` 来生成我们的表达式语法的铁路图：

```py
syntax_diagram(EXPR_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">expr</text></g></g></g></g></svg>

```py
expr

```

<svg class="railroad-diagram" height="122" viewBox="0 0 313.5 122" width="313.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">term</text></g> <g class="terminal"><text x="156.75" y="35">+</text></g> <g class="non-terminal"><text x="226.5" y="35">expr</text></g></g> <g><g class="non-terminal"><text x="87.0" y="65">term</text></g> <g class="terminal"><text x="156.75" y="65">-</text></g> <g class="non-terminal"><text x="226.5" y="65">expr</text></g></g> <g><g class="non-terminal"><text x="156.75" y="95">term</text></g></g></g></g></svg>

```py
term

```

<svg class="railroad-diagram" height="122" viewBox="0 0 330.5 122" width="330.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="95.5" y="35">factor</text></g> <g class="terminal"><text x="173.75" y="35">*</text></g> <g class="non-terminal"><text x="243.5" y="35">term</text></g></g> <g><g class="non-terminal"><text x="95.5" y="65">factor</text></g> <g class="terminal"><text x="173.75" y="65">/</text></g> <g class="non-terminal"><text x="243.5" y="65">term</text></g></g> <g><g class="non-terminal"><text x="165.25" y="95">factor</text></g></g></g></g></svg>

```py
factor

```

<svg class="railroad-diagram" height="182" viewBox="0 0 347.5 182" width="347.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="128.25" y="65">-</text></g> <g class="non-terminal"><text x="198.0" y="65">factor</text></g></g> <g><g class="terminal"><text x="128.25" y="35">+</text></g> <g class="non-terminal"><text x="198.0" y="35">factor</text></g></g> <g><g class="terminal"><text x="112.5" y="95">(</text></g> <g class="non-terminal"><text x="173.75" y="95">expr</text></g> <g class="terminal"><text x="235.0" y="95">)</text></g></g> <g><g class="non-terminal"><text x="99.75" y="125">integer</text></g> <g class="terminal"><text x="173.75" y="125">.</text></g> <g class="non-terminal"><text x="247.75" y="125">integer</text></g></g> <g><g class="non-terminal"><text x="173.75" y="155">integer</text></g></g></g></g></svg>

```py
integer

```

<svg class="railroad-diagram" height="92" viewBox="0 0 282.0 92" width="282.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">digit</text></g> <g class="non-terminal"><text x="182.25" y="35">integer</text></g></g> <g><g class="non-terminal"><text x="141.0" y="65">digit</text></g></g></g></g></svg>

```py
digit

```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

这种铁路图表示法在可视化语法结构时非常有用，尤其是对于更复杂的语法。

## 一些语法

让我们创建（并可视化）一些更多的语法，并使用它们进行模糊测试。

### CGI 语法

这里是 覆盖章节 中引入的 `cgi_decode()` 的语法。

```py
CGI_GRAMMAR: Grammar = {
    "<start>":
        ["<string>"],

    "<string>":
        ["<letter>", "<letter><string>"],

    "<letter>":
        ["<plus>", "<percent>", "<other>"],

    "<plus>":
        ["+"],

    "<percent>":
        ["%<hexdigit><hexdigit>"],

    "<hexdigit>":
        ["0", "1", "2", "3", "4", "5", "6", "7",
            "8", "9", "a", "b", "c", "d", "e", "f"],

    "<other>":  # Actually, could be _all_ letters
        ["0", "1", "2", "3", "4", "5", "a", "b", "c", "d", "e", "-", "_"],
} 
```

```py
syntax_diagram(CGI_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 191.0 62" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="95.5" y="35">string</text></g></g></g></g></svg>

```py
string

```

<svg class="railroad-diagram" height="92" viewBox="0 0 282.0 92" width="282.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="141.0" y="35">letter</text></g></g> <g><g class="non-terminal"><text x="95.5" y="65">letter</text></g> <g class="non-terminal"><text x="186.5" y="65">string</text></g></g></g></g></svg>

```py
letter

```

<svg class="railroad-diagram" height="122" viewBox="0 0 199.5 122" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">plus</text></g></g> <g><g class="non-terminal"><text x="99.75" y="65">percent</text></g></g> <g><g class="non-terminal"><text x="99.75" y="95">other</text></g></g></g></g></svg>

```py
plus

```

<svg class="railroad-diagram" height="62" viewBox="0 0 148.5 62" width="148.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">+</text></g></g></g></g></svg>

```py
percent

```

<svg class="railroad-diagram" height="62" viewBox="0 0 364.5 62" width="364.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="35">%</text></g> <g class="non-terminal"><text x="152.5" y="35">hexdigit</text></g> <g class="non-terminal"><text x="260.5" y="35">hexdigit</text></g></g></g></g></svg>

```py
hexdigit

```

<svg class="railroad-diagram" height="138" viewBox="0 0 611.0 138" width="611.0"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g> <g><g class="terminal"><text x="84.25" y="103">2</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">3</text></g></g> <g><g class="terminal"><text x="172.75" y="73">4</text></g></g> <g><g class="terminal"><text x="172.75" y="103">5</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">6</text></g></g> <g><g class="terminal"><text x="261.25" y="73">7</text></g></g> <g><g class="terminal"><text x="261.25" y="103">8</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">9</text></g></g> <g><g class="terminal"><text x="349.75" y="73">a</text></g></g> <g><g class="terminal"><text x="349.75" y="103">b</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">c</text></g></g> <g><g class="terminal"><text x="438.25" y="73">d</text></g></g> <g><g class="terminal"><text x="438.25" y="103">e</text></g></g></g> <g><g><g class="terminal"><text x="526.75" y="73">f</text></g></g></g></g></g></svg>

```py
other

```

<svg class="railroad-diagram" height="109" viewBox="0 0 699.5 109" width="699.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">a</text></g></g> <g><g class="terminal"><text x="349.75" y="73">b</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">c</text></g></g> <g><g class="terminal"><text x="438.25" y="73">d</text></g></g></g> <g><g><g class="terminal"><text x="526.75" y="43">e</text></g></g> <g><g class="terminal"><text x="526.75" y="73">-</text></g></g></g> <g><g><g class="terminal"><text x="615.25" y="73">_</text></g></g></g></g></g></svg>

与 基本模糊测试 或 基于变异的模糊测试 相比，语法可以快速生成各种组合：

```py
for i in range(10):
    print(simple_grammar_fuzzer(grammar=CGI_GRAMMAR, max_nonterminals=10)) 
```

```py
+%9a
+++%ce+
+_
+%c6c
++
+%cd+5
1%ee
%b9%d5
%96
%57d%42

```

### URL 语法

我们在 CGI 输入中看到的相同属性也适用于更复杂的输入。让我们使用语法生成大量的有效 URL：

```py
URL_GRAMMAR: Grammar = {
    "<start>":
        ["<url>"],
    "<url>":
        ["<scheme>://<authority><path><query>"],
    "<scheme>":
        ["http", "https", "ftp", "ftps"],
    "<authority>":
        ["<host>", "<host>:<port>", "<userinfo>@<host>", "<userinfo>@<host>:<port>"],
    "<host>":  # Just a few
        ["cispa.saarland", "www.google.com", "fuzzingbook.com"],
    "<port>":
        ["80", "8080", "<nat>"],
    "<nat>":
        ["<digit>", "<digit><digit>"],
    "<digit>":
        ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"],
    "<userinfo>":  # Just one
        ["user:password"],
    "<path>":  # Just a few
        ["", "/", "/<id>"],
    "<id>":  # Just a few
        ["abc", "def", "x<digit><digit>"],
    "<query>":
        ["", "?<params>"],
    "<params>":
        ["<param>", "<param>&<params>"],
    "<param>":  # Just a few
        ["<id>=<id>", "<id>=<nat>"],
} 
```

```py
syntax_diagram(URL_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 165.5 62" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="82.75" y="35">url</text></g></g></g></g></svg>

```py
url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 529.5 62" width="529.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="95.5" y="35">scheme</text></g> <g class="terminal"><text x="173.75" y="35">://</text></g> <g class="non-terminal"><text x="264.75" y="35">authority</text></g> <g class="non-terminal"><text x="360.0" y="35">path</text></g> <g class="non-terminal"><text x="438.25" y="35">query</text></g></g></g></g></svg>

```py
scheme

```

<svg class="railroad-diagram" height="152" viewBox="0 0 182.5 152" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="91.25" y="65">https</text></g></g> <g><g class="terminal"><text x="91.25" y="35">http</text></g></g> <g><g class="terminal"><text x="91.25" y="95">ftp</text></g></g> <g><g class="terminal"><text x="91.25" y="125">ftps</text></g></g></g></g></svg>

```py
authority

```

<svg class="railroad-diagram" height="152" viewBox="0 0 453.0 152" width="453.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="165.25" y="65">host</text></g> <g class="terminal"><text x="226.5" y="65">:</text></g> <g class="non-terminal"><text x="287.75" y="65">port</text></g></g> <g><g class="non-terminal"><text x="226.5" y="35">host</text></g></g> <g><g class="non-terminal"><text x="165.25" y="95">userinfo</text></g> <g class="terminal"><text x="243.5" y="95">@</text></g> <g class="non-terminal"><text x="304.75" y="95">host</text></g></g> <g><g class="non-terminal"><text x="104.0" y="125">userinfo</text></g> <g class="terminal"><text x="182.25" y="125">@</text></g> <g class="non-terminal"><text x="243.5" y="125">host</text></g> <g class="terminal"><text x="304.75" y="125">:</text></g> <g class="non-terminal"><text x="366.0" y="125">port</text></g></g></g></g></svg>

```py
host

```

<svg class="railroad-diagram" height="122" viewBox="0 0 267.5 122" width="267.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="133.75" y="35">cispa.saarland</text></g></g> <g><g class="terminal"><text x="133.75" y="65">www.google.com</text></g></g> <g><g class="terminal"><text x="133.75" y="95">fuzzingbook.com</text></g></g></g></g></svg>

```py
port

```

<svg class="railroad-diagram" height="122" viewBox="0 0 174.0 122" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">80</text></g></g> <g><g class="terminal"><text x="87.0" y="65">8080</text></g></g> <g><g class="non-terminal"><text x="87.0" y="95">nat</text></g></g></g></g></svg>

```py
nat

```

<svg class="railroad-diagram" height="92" viewBox="0 0 265.0 92" width="265.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="132.5" y="35">digit</text></g></g> <g><g class="non-terminal"><text x="91.25" y="65">digit</text></g> <g class="non-terminal"><text x="173.75" y="65">digit</text></g></g></g></g></svg>

```py
digit

```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

```py
userinfo

```

<svg class="railroad-diagram" height="62" viewBox="0 0 250.5 62" width="250.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="125.25" y="35">user:password</text></g></g></g></g></svg>

```py
path

```

<svg class="railroad-diagram" height="122" viewBox="0 0 205.5 122" width="205.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="102.75" y="65">/</text></g></g> <g><g class="terminal"><text x="74.25" y="95">/</text></g> <g class="non-terminal"><text x="127.0" y="95">id</text></g></g></g></g></svg>

```py
id

```

<svg class="railroad-diagram" height="122" viewBox="0 0 313.5 122" width="313.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="156.75" y="35">abc</text></g></g> <g><g class="terminal"><text x="156.75" y="65">def</text></g></g> <g><g class="terminal"><text x="74.25" y="95">x</text></g> <g class="non-terminal"><text x="139.75" y="95">digit</text></g> <g class="non-terminal"><text x="222.25" y="95">digit</text></g></g></g></g></svg>

```py
query

```

<svg class="railroad-diagram" height="92" viewBox="0 0 239.5 92" width="239.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="74.25" y="65">?</text></g> <g class="non-terminal"><text x="144.0" y="65">params</text></g></g></g></g></svg>

```py
params

```

<svg class="railroad-diagram" height="92" viewBox="0 0 322.0 92" width="322.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="161.0" y="35">param</text></g></g> <g><g class="non-terminal"><text x="91.25" y="65">param</text></g> <g class="terminal"><text x="156.75" y="65">&</text></g> <g class="non-terminal"><text x="226.5" y="65">params</text></g></g></g></g></svg>

```py
param

```

<svg class="railroad-diagram" height="92" viewBox="0 0 271.0 92" width="271.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="82.75" y="35">id</text></g> <g class="terminal"><text x="135.5" y="35">=</text></g> <g class="non-terminal"><text x="188.25" y="35">id</text></g></g> <g><g class="non-terminal"><text x="78.5" y="65">id</text></g> <g class="terminal"><text x="131.25" y="65">=</text></g> <g class="non-terminal"><text x="188.25" y="65">nat</text></g></g></g></g></svg>

再次，在毫秒之内，我们可以生成大量的有效输入。

```py
for i in range(10):
    print(simple_grammar_fuzzer(grammar=URL_GRAMMAR, max_nonterminals=10)) 
```

```py
https://user:password@cispa.saarland:80/
http://fuzzingbook.com?def=56&x89=3&x46=48&def=def
ftp://cispa.saarland/?x71=5&x35=90&def=abc
https://cispa.saarland:80/def?def=7&x23=abc
https://fuzzingbook.com:80/
https://fuzzingbook.com:80/abc?def=abc&abc=x14&def=abc&abc=2&def=38
ftps://fuzzingbook.com/x87
https://user:password@fuzzingbook.com:6?def=54&x44=abc
http://fuzzingbook.com:80?x33=25&def=8
http://fuzzingbook.com:8080/def

```

### 自然语言语法

最后，语法不仅限于 *形式语言*，如计算机输入，还可以用于生成 *自然语言*。这是我们用来选择这本书标题的语法：

```py
TITLE_GRAMMAR: Grammar = {
    "<start>": ["<title>"],
    "<title>": ["<topic>: <subtopic>"],
    "<topic>": ["Generating Software Tests", "<fuzzing-prefix>Fuzzing", "The Fuzzing Book"],
    "<fuzzing-prefix>": ["", "The Art of ", "The Joy of "],
    "<subtopic>": ["<subtopic-main>",
                   "<subtopic-prefix><subtopic-main>",
                   "<subtopic-main><subtopic-suffix>"],
    "<subtopic-main>": ["Breaking Software",
                        "Generating Software Tests",
                        "Principles, Techniques and Tools"],
    "<subtopic-prefix>": ["", "Tools and Techniques for "],
    "<subtopic-suffix>": [" for <reader-property> and <reader-property>",
                          " for <software-property> and <software-property>"],
    "<reader-property>": ["Fun", "Profit"],
    "<software-property>": ["Robustness", "Reliability", "Security"],
} 
```

```py
syntax_diagram(TITLE_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 182.5 62" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">title</text></g></g></g></g></svg>

```py
title

```

<svg class="railroad-diagram" height="62" viewBox="0 0 347.5 62" width="347.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">topic</text></g> <g class="terminal"><text x="161.0" y="35">:</text></g> <g class="non-terminal"><text x="243.5" y="35">subtopic</text></g></g></g></g></svg>

```py
topic

```

<svg class="railroad-diagram" height="122" viewBox="0 0 358.5 122" width="358.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="179.25" y="35">Generating Software Tests</text></g></g> <g><g class="non-terminal"><text x="129.5" y="65">fuzzing-prefix</text></g> <g class="terminal"><text x="258.75" y="65">Fuzzing</text></g></g> <g><g class="terminal"><text x="179.25" y="95">The Fuzzing Book</text></g></g></g></g></svg>

```py
fuzzing-prefix

```

<svg class="railroad-diagram" height="122" viewBox="0 0 233.5 122" width="233.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="116.75" y="65">The Art of</text></g></g> <g><g class="terminal"><text x="116.75" y="95">The Joy of</text></g></g></g></g></svg>

```py
subtopic

```

<svg class="railroad-diagram" height="122" viewBox="0 0 418.0 122" width="418.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="209.0" y="35">subtopic-main</text></g></g> <g><g class="non-terminal"><text x="133.75" y="65">subtopic-prefix</text></g> <g class="non-terminal"><text x="292.75" y="65">subtopic-main</text></g></g> <g><g class="non-terminal"><text x="125.25" y="95">subtopic-main</text></g> <g class="non-terminal"><text x="284.25" y="95">subtopic-suffix</text></g></g></g></g></svg>

```py
subtopic-main

```

<svg class="railroad-diagram" height="122" viewBox="0 0 412.0 122" width="412.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="206.0" y="35">Breaking Software</text></g></g> <g><g class="terminal"><text x="206.0" y="65">Generating Software Tests</text></g></g> <g><g class="terminal"><text x="206.0" y="95">Principles, Techniques and Tools</text></g></g></g></g></svg>

```py
subtopic-prefix

```

<svg class="railroad-diagram" height="92" viewBox="0 0 352.5 92" width="352.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="176.25" y="65">Tools and Techniques for</text></g></g></g></g></svg>

```py
subtopic-suffix

```

<svg class="railroad-diagram" height="92" viewBox="0 0 634.0 92" width="634.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="108.25" y="35">for</text></g> <g class="non-terminal"><text x="233.25" y="35">reader-property</text></g> <g class="terminal"><text x="358.25" y="35">and</text></g> <g class="non-terminal"><text x="483.25" y="35">reader-property</text></g></g> <g><g class="terminal"><text x="91.25" y="65">for</text></g> <g class="non-terminal"><text x="224.75" y="65">software-property</text></g> <g class="terminal"><text x="358.25" y="65">and</text></g> <g class="non-terminal"><text x="491.75" y="65">software-property</text></g></g></g></g></svg>

```py
reader-property

```

<svg class="railroad-diagram" height="92" viewBox="0 0 191.0 92" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="95.5" y="35">Fun</text></g></g> <g><g class="terminal"><text x="95.5" y="65">Profit</text></g></g></g></g></svg>

```py
software-property

```

<svg class="railroad-diagram" height="122" viewBox="0 0 233.5 122" width="233.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="116.75" y="35">Robustness</text></g></g> <g><g class="terminal"><text x="116.75" y="65">Reliability</text></g></g> <g><g class="terminal"><text x="116.75" y="95">Security</text></g></g></g></g></svg>

```py
from [typing](https://docs.python.org/3/library/typing.html) import Set 
```

```py
titles: Set[str] = set()
while len(titles) < 10:
    titles.add(simple_grammar_fuzzer(
        grammar=TITLE_GRAMMAR, max_nonterminals=10))
titles 
```

```py
{'Fuzzing: Generating Software Tests',
 'Fuzzing: Principles, Techniques and Tools',
 'Generating Software Tests: Breaking Software',
 'Generating Software Tests: Breaking Software for Robustness and Robustness',
 'Generating Software Tests: Principles, Techniques and Tools',
 'Generating Software Tests: Principles, Techniques and Tools for Profit and Fun',
 'Generating Software Tests: Tools and Techniques for Principles, Techniques and Tools',
 'The Fuzzing Book: Breaking Software',
 'The Fuzzing Book: Generating Software Tests for Profit and Profit',
 'The Fuzzing Book: Generating Software Tests for Robustness and Robustness'}

```

（如果你在这里发现存在冗余（“鲁棒性和鲁棒性”）：在我们的 基于覆盖的模糊测试章节 中，我们将展示如何只覆盖每个扩展一次。如果你更喜欢某些替代方案，概率语法模糊测试 将为你提供。）

## 语法作为变异种子

语法的一个非常有用的特性是它们产生的大多数输入都是有效的。从句法的角度来看，输入实际上是*始终*有效的，因为它们满足给定语法的约束。（当然，首先需要一个有效的语法。）然而，还有一些*语义*特性在语法中难以表达。例如，对于一个 URL，端口号的范围应该在 1024 到 2048 之间，这在语法中很难写出来。如果必须满足更复杂的约束，很快就会达到语法的表达能力极限。

一种解决方法是给语法附加约束，正如我们将在本书后面讨论的。另一种可能性是将基于语法的模糊测试和基于变异的模糊测试的优点结合起来。想法是使用语法生成的输入作为进一步基于变异的模糊测试的*种子*。这样，我们不仅可以探索*有效*的输入，还可以检查有效和无效输入之间的*边界*。这尤其有趣，因为略微无效的输入可以找到解析错误（这些错误通常很多）。与一般的模糊测试一样，意外的是揭示程序中错误的关键。

要使用我们生成的输入作为种子，我们可以直接将它们输入到之前介绍的变异模糊测试器中：

```py
from MutationFuzzer import MutationFuzzer  # minor dependency 
```

```py
number_of_seeds = 10
seeds = [
    simple_grammar_fuzzer(
        grammar=URL_GRAMMAR,
        max_nonterminals=10) for i in range(number_of_seeds)]
seeds 
```

```py
['ftps://user:password@www.google.com:80',
 'http://cispa.saarland/',
 'ftp://www.google.com:42/',
 'ftps://user:password@fuzzingbook.com:39?abc=abc',
 'https://www.google.com?x33=1&x06=1',
 'http://www.google.com:02/',
 'https://user:password@www.google.com/',
 'ftp://cispa.saarland:8080/?abc=abc&def=def&abc=5',
 'http://www.google.com:80/def?def=abc',
 'http://user:password@cispa.saarland/']

```

```py
m = MutationFuzzer(seeds) 
```

```py
[m.fuzz() for i in range(20)] 
```

```py
['ftps://user:password@www.google.com:80',
 'http://cispa.saarland/',
 'ftp://www.google.com:42/',
 'ftps://user:password@fuzzingbook.com:39?abc=abc',
 'https://www.google.com?x33=1&x06=1',
 'http://www.google.com:02/',
 'https://user:password@www.google.com/',
 'ftp://cispa.saarland:8080/?abc=abc&def=def&abc=5',
 'http://www.google.com:80/def?def=abc',
 'http://user:password@cispa.saarland/',
 'Eh4tp:www.coogle.com:80/def?d%f=abc',
 'ftps://}ser:passwod@fuzzingbook.com:9?abc=abc',
 'uftp//cispa.sRaarland:808&0?abc=abc&def=defabc=5',
 'http://user:paswor9d@cispar.saarland/v',
 'ftp://Www.g\x7fogle.cAom:42/',
 'hht://userC:qassMword@cispy.csaarland/',
 'httx://ww.googlecom:80defde`f=ac',
 'htt://cispq.waarlnd/',
 'htFtp\t://cmspa./saarna(md/',
 'ft:/www.google.com:42\x0f']

```

虽然前 10 次`fuzz()`调用返回的是种子输入（按设计），但后续的调用又创建了任意的变异。使用`MutationCoverageFuzzer`而不是`MutationFuzzer`，我们可以再次通过覆盖率来引导搜索——从而将多个世界的优点结合起来。

## 语法工具箱

现在我们介绍一些有助于我们编写语法的技巧。

### 转义

在我们的语法中使用`<`和`>`来界定非终结符，我们如何实际上表达某些输入应该包含`<`和`>`呢？答案是简单的：只需为它们引入一个符号。

```py
simple_nonterminal_grammar: Grammar = {
    "<start>": ["<nonterminal>"],
    "<nonterminal>": ["<left-angle><identifier><right-angle>"],
    "<left-angle>": ["<"],
    "<right-angle>": [">"],
    "<identifier>": ["id"]  # for now
} 
```

在`simple_nonterminal_grammar`中，`<left-angle>`的展开和`<right-angle>`的展开都不可能被误认为是非终结符。因此，我们可以生成尽可能多的。

（注意，这并不适用于`simple_grammar_fuzzer()`，而是适用于我们在下一章中将要介绍的`GrammarFuzzer`类。）

### 扩展语法

在本书的进程中，我们经常遇到通过*扩展*现有语法以添加新功能来创建语法的问题。这种扩展在面向对象编程中非常类似于子类化。

要从一个现有语法$g$创建一个新的语法$g'$，我们首先将$g$复制到$g'$中，然后添加新的选择和/或添加新符号来扩展现有规则。以下是一个示例，扩展上述`nonterminal`语法以包含一个更好的标识符规则：

```py
import [copy](https://docs.python.org/3/library/copy.html) 
```

```py
nonterminal_grammar = copy.deepcopy(simple_nonterminal_grammar)
nonterminal_grammar["<identifier>"] = ["<idchar>", "<identifier><idchar>"]
nonterminal_grammar["<idchar>"] = ['a', 'b', 'c', 'd']  # for now 
```

```py
nonterminal_grammar 
```

```py
{'<start>': ['<nonterminal>'],
 '<nonterminal>': ['<left-angle><identifier><right-angle>'],
 '<left-angle>': ['<'],
 '<right-angle>': ['>'],
 '<identifier>': ['<idchar>', '<identifier><idchar>'],
 '<idchar>': ['a', 'b', 'c', 'd']}

```

由于这种语法的扩展是一个常见的操作，我们引入了一个自定义函数`extend_grammar()`，它首先复制给定的语法，然后使用 Python 字典的`update()`方法从字典中更新它：

```py
def extend_grammar(grammar: Grammar, extension: Grammar = {}) -> Grammar:
  """Create a copy of `grammar`, updated with `extension`."""
    new_grammar = copy.deepcopy(grammar)
    new_grammar.update(extension)
    return new_grammar 
```

这个对 `extend_grammar()` 的调用将 `simple_nonterminal_grammar` 扩展到 `nonterminal_grammar`，就像上面的“手动”示例一样：

```py
nonterminal_grammar = extend_grammar(simple_nonterminal_grammar,
                                     {
                                         "<identifier>": ["<idchar>", "<identifier><idchar>"],
                                         # for now
                                         "<idchar>": ['a', 'b', 'c', 'd']
                                     }
                                     ) 
```

### 字符类别

在上述 `nonterminal_grammar` 中，我们只列出了前几个字母；实际上，手动枚举语法中的所有字母或数字，如 `<idchar> ::= 'a' | 'b' | 'c' ...` 是一件痛苦的事情。

然而，请记住，语法是程序的一部分，因此也可以通过编程方式构建。我们引入了一个名为 `srange()` 的函数，该函数构建一个字符串中的字符列表：

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
def srange(characters: str) -> List[Expansion]:
  """Construct a list with all characters in the string"""
    return [c for c in characters] 
```

如果我们传递它常量 `string.ascii_letters`，它包含所有 ASCII 字母，`srange()` 返回所有 ASCII 字母的列表：

```py
string.ascii_letters 
```

```py
'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'

```

```py
srange(string.ascii_letters)[:10] 
```

```py
['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j']

```

我们可以在我们的语法中使用这样的常量来快速定义标识符：

```py
nonterminal_grammar = extend_grammar(nonterminal_grammar,
                                     {
                                         "<idchar>": (srange(string.ascii_letters) + 
                                                      srange(string.digits) + 
                                                      srange("-_"))
                                     }
                                     ) 
```

```py
[simple_grammar_fuzzer(nonterminal_grammar, "<identifier>") for i in range(10)] 
```

```py
['b', 'd', 'V9', 'x4c', 'YdiEWj', 'c', 'xd', '7', 'vIU', 'QhKD']

```

短路 `crange(start, end)` 返回从 `start` 到（包括）`end` 的 ASCII 范围内的所有字符列表：

```py
def crange(character_start: str, character_end: str) -> List[Expansion]:
    return [chr(i)
            for i in range(ord(character_start), ord(character_end) + 1)] 
```

我们可以使用这一点来表示字符范围：

```py
crange('0', '9') 
```

```py
['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']

```

```py
assert crange('a', 'z') == srange(string.ascii_lowercase) 
```

### 语法快捷方式

在上述 `nonterminal_grammar` 中，就像在其他语法中一样，我们必须使用**递归**来表示字符的重复，即通过引用原始定义：

```py
nonterminal_grammar["<identifier>"] 
```

```py
['<idchar>', '<identifier><idchar>']

```

如果我们能够简单地声明一个非终结符应该是一个非空字母序列——例如，如下所示

```py
<identifier> = <idchar>+
```

其中 `+` 表示其后符号的非空重复。

在语法中，像 `+` 这样的运算符经常被用作方便的**快捷方式**。正式来说，我们的语法采用所谓的[巴科斯-诺尔范式](https://en.wikipedia.org/wiki/Backus-Naur_form)，或简称**BNF**。运算符**扩展**了所谓的**扩展 BNF**，或简称**EBNF**：

+   `<symbol>?` 的形式表示 `<symbol>` 是可选的——也就是说，它可以出现 0 次或 1 次。

+   `<symbol>+` 的形式表示 `<symbol>` 可以出现 1 次或多次重复。

+   `<symbol>*` 的形式表示 `<symbol>` 可以出现 0 次或多次。（换句话说，它是一个可选的重复。）

为了使事情更有趣，我们希望使用**括号**与上述快捷方式一起使用。因此，`(<foo><bar>)?` 表示 `<foo>` 和 `<bar>` 的序列是可选的。

使用这样的运算符，我们可以以更简单的方式定义标识符规则。为此，让我们创建原始语法的副本并修改 `<identifier>` 规则：

```py
nonterminal_ebnf_grammar = extend_grammar(nonterminal_grammar,
                                          {
                                              "<identifier>": ["<idchar>+"]
                                          }
                                          ) 
```

同样，我们也可以简化表达式语法。考虑符号是可选的，以及整数可以表示为数字序列。

```py
EXPR_EBNF_GRAMMAR: Grammar = {
    "<start>":
        ["<expr>"],

    "<expr>":
        ["<term> + <expr>", "<term> - <expr>", "<term>"],

    "<term>":
        ["<factor> * <term>", "<factor> / <term>", "<factor>"],

    "<factor>":
        ["<sign>?<factor>", "(<expr>)", "<integer>(.<integer>)?"],

    "<sign>":
        ["+", "-"],

    "<integer>":
        ["<digit>+"],

    "<digit>":
        srange(string.digits)
} 
```

让我们实现一个名为 `convert_ebnf_grammar()` 的函数，它接受这样的 EBNF 语法并将其自动转换为 BNF 语法。

<details id="Excursion:-Implementing-convert_ebnf_grammar()"><summary>实现 `convert_ebnf_grammar()`</summary>

我们的目的是将像上面那样的 EBNF 语法转换为常规的 BNF 语法。这是通过四个规则完成的：

1.  表达式 `(content)op`，其中 `op` 是 `?`、`+`、`*` 之一，变为 `<new-symbol>op`，并添加一个新规则 `<new-symbol> ::= content`。

1.  表达式 `<symbol>?` 变为 `<new-symbol>`，其中 `<new-symbol> ::= <empty> | <symbol>`。

1.  表达式 `<symbol>+` 变为 `<new-symbol>`，其中 `<new-symbol> ::= <symbol> | <symbol><new-symbol>`。

1.  表达式 `<symbol>*` 变为 `<new-symbol>`，其中 `<new-symbol> ::= <empty> | <symbol><new-symbol>`。

在这里，`<empty>` 扩展为空字符串，正如 `<empty> ::=`。 (这也可以称为 *epsilon 扩展*。)

如果这些运算符让你想起了 *正则表达式*，这并非偶然：实际上，任何基本正则表达式都可以使用上述规则（以及上面定义的 `crange()` 中的字符类）转换为语法。

在上述示例上应用这些规则会产生以下结果：

+   `<idchar>+` 变为 `<idchar><new-symbol>`，其中 `<new-symbol> ::= <idchar> | <idchar><new-symbol>`。

+   `<integer>(.<integer>)?` 变为 `<integer><new-symbol>`，其中 `<new-symbol> ::= <empty> | .<integer>`。

让我们在三个步骤中实现这些规则。

##### 创建新符号

首先，我们需要一种创建新符号的机制。这相当直接。

```py
def new_symbol(grammar: Grammar, symbol_name: str = "<symbol>") -> str:
  """Return a new symbol for `grammar` based on `symbol_name`"""
    if symbol_name not in grammar:
        return symbol_name

    count = 1
    while True:
        tentative_symbol_name = symbol_name[:-1] + "-" + repr(count) + ">"
        if tentative_symbol_name not in grammar:
            return tentative_symbol_name
        count += 1 
```

```py
assert new_symbol(EXPR_EBNF_GRAMMAR, '<expr>') == '<expr-1>' 
```

##### 扩展括号表达式

接下来，我们需要一种方法从我们的扩展中提取括号表达式，并根据上述规则进行扩展。让我们从提取表达式开始：

```py
RE_PARENTHESIZED_EXPR = re.compile(r'\([^()]*\)[?+*]') 
```

```py
def parenthesized_expressions(expansion: Expansion) -> List[str]:
    # In later chapters, we allow expansions to be tuples,
    # with the expansion being the first element
    if isinstance(expansion, tuple):
        expansion = expansion[0]

    return re.findall(RE_PARENTHESIZED_EXPR, expansion) 
```

```py
assert parenthesized_expressions("(<foo>)* (<foo><bar>)+ (+<foo>)? <integer>(.<integer>)?") == [
    '(<foo>)*', '(<foo><bar>)+', '(+<foo>)?', '(.<integer>)?'] 
```

我们现在可以使用这些来应用上述第 1 条规则，为括号中的表达式引入新符号。

```py
def convert_ebnf_parentheses(ebnf_grammar: Grammar) -> Grammar:
  """Convert a grammar in extended BNF to BNF"""
    grammar = extend_grammar(ebnf_grammar)
    for nonterminal in ebnf_grammar:
        expansions = ebnf_grammar[nonterminal]

        for i in range(len(expansions)):
            expansion = expansions[i]
            if not isinstance(expansion, str):
                expansion = expansion[0]

            while True:
                parenthesized_exprs = parenthesized_expressions(expansion)
                if len(parenthesized_exprs) == 0:
                    break

                for expr in parenthesized_exprs:
                    operator = expr[-1:]
                    contents = expr[1:-2]

                    new_sym = new_symbol(grammar)

                    exp = grammar[nonterminal][i]
                    opts = None
                    if isinstance(exp, tuple):
                        (exp, opts) = exp
                    assert isinstance(exp, str)

                    expansion = exp.replace(expr, new_sym + operator, 1)
                    if opts:
                        grammar[nonterminal][i] = (expansion, opts)
                    else:
                        grammar[nonterminal][i] = expansion

                    grammar[new_sym] = [contents]

    return grammar 
```

这将按照上述草图进行转换：

```py
convert_ebnf_parentheses({"<number>": ["<integer>(.<integer>)?"]}) 
```

```py
{'<number>': ['<integer><symbol>?'], '<symbol>': ['.<integer>']}

```

即使对于嵌套的括号表达式也有效：

```py
convert_ebnf_parentheses({"<foo>": ["((<foo>)?)+"]}) 
```

```py
{'<foo>': ['<symbol-1>+'], '<symbol>': ['<foo>'], '<symbol-1>': ['<symbol>?']}

```

##### 扩展运算符

在扩展括号表达式之后，我们现在需要处理跟在运算符后面的符号（`?`，`*`，`+`）。与上面的 `convert_ebnf_parentheses()` 类似，我们首先提取所有跟在运算符后面的符号。

```py
RE_EXTENDED_NONTERMINAL = re.compile(r'(<[^<> ]*>[?+*])') 
```

```py
def extended_nonterminals(expansion: Expansion) -> List[str]:
    # In later chapters, we allow expansions to be tuples,
    # with the expansion being the first element
    if isinstance(expansion, tuple):
        expansion = expansion[0]

    return re.findall(RE_EXTENDED_NONTERMINAL, expansion) 
```

```py
assert extended_nonterminals(
    "<foo>* <bar>+ <elem>? <none>") == ['<foo>*', '<bar>+', '<elem>?'] 
```

我们的转换器提取符号和运算符，并根据上述规则添加新符号。

```py
def convert_ebnf_operators(ebnf_grammar: Grammar) -> Grammar:
  """Convert a grammar in extended BNF to BNF"""
    grammar = extend_grammar(ebnf_grammar)
    for nonterminal in ebnf_grammar:
        expansions = ebnf_grammar[nonterminal]

        for i in range(len(expansions)):
            expansion = expansions[i]
            extended_symbols = extended_nonterminals(expansion)

            for extended_symbol in extended_symbols:
                operator = extended_symbol[-1:]
                original_symbol = extended_symbol[:-1]
                assert original_symbol in ebnf_grammar, \
                    f"{original_symbol} is not defined in grammar"

                new_sym = new_symbol(grammar, original_symbol)

                exp = grammar[nonterminal][i]
                opts = None
                if isinstance(exp, tuple):
                    (exp, opts) = exp
                assert isinstance(exp, str)

                new_exp = exp.replace(extended_symbol, new_sym, 1)
                if opts:
                    grammar[nonterminal][i] = (new_exp, opts)
                else:
                    grammar[nonterminal][i] = new_exp

                if operator == '?':
                    grammar[new_sym] = ["", original_symbol]
                elif operator == '*':
                    grammar[new_sym] = ["", original_symbol + new_sym]
                elif operator == '+':
                    grammar[new_sym] = [
                        original_symbol, original_symbol + new_sym]

    return grammar 
```

```py
convert_ebnf_operators({"<integer>": ["<digit>+"], "<digit>": ["0"]}) 
```

```py
{'<integer>': ['<digit-1>'],
 '<digit>': ['0'],
 '<digit-1>': ['<digit>', '<digit><digit-1>']}

```

##### 全部一起

我们可以将这两个结合起来，首先扩展括号，然后是运算符：

```py
def convert_ebnf_grammar(ebnf_grammar: Grammar) -> Grammar:
    return convert_ebnf_operators(convert_ebnf_parentheses(ebnf_grammar)) 
```</details>

这里是一个使用 `convert_ebnf_grammar()` 的例子：

```py
convert_ebnf_grammar({"<authority>": ["(<userinfo>@)?<host>(:<port>)?"]}) 
```

```py
{'<authority>': ['<symbol-2><host><symbol-1-1>'],
 '<symbol>': ['<userinfo>@'],
 '<symbol-1>': [':<port>'],
 '<symbol-2>': ['', '<symbol>'],
 '<symbol-1-1>': ['', '<symbol-1>']}

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

成功！我们已经将 EBNF 语法优雅地转换为 BNF。

使用字符类和 EBNF 语法转换，我们有两个强大的工具，使编写语法变得更容易。我们将反复使用这些工具，当我们处理语法时。

### 语法扩展

在本书的整个过程中，我们经常需要为语法指定 *附加信息*，例如 *概率* 或 *约束*。为了支持这些扩展，以及可能的其他扩展，我们定义了一种 *注释* 机制。

我们对注释语法的概念是向单个扩展添加 *注释*。为此，我们允许扩展不仅可以是一个字符串，还可以是一个字符串和一组属性的 *对*，如下所示

```py
"<expr>":
        [("<term> + <expr>", opts(min_depth=10)),
         ("<term> - <expr>", opts(max_depth=2)),
         "<term>"] 
```

在这里，`opts()`函数允许我们表达适用于单个扩展的注释；在这种情况下，加法将被注释为具有 10 的`min_depth`值，减法将被注释为具有 2 的`max_depth`值。这些注释的含义留给处理语法的各个算法来决定；然而，一般而言，它们可以被忽略。

<details id="Excursion:-Implementing-opts()"><summary>实现`opts()`</summary>

我们的`opts()`辅助函数返回其参数到值的映射：

```py
def opts(**kwargs: Any) -> Dict[str, Any]:
    return kwargs 
```

```py
opts(min_depth=10) 
```

```py
{'min_depth': 10}

```

为了处理扩展字符串以及扩展和注释的成对出现，我们通过指定的辅助函数`exp_string()`和`exp_opts()`来访问扩展字符串和相关注释：

```py
def exp_string(expansion: Expansion) -> str:
  """Return the string to be expanded"""
    if isinstance(expansion, str):
        return expansion
    return expansion[0] 
```

```py
exp_string(("<term> + <expr>", opts(min_depth=10))) 
```

```py
'<term> + <expr>'

```

```py
def exp_opts(expansion: Expansion) -> Dict[str, Any]:
  """Return the options of an expansion.  If options are not defined, return {}"""
    if isinstance(expansion, str):
        return {}
    return expansion[1] 
```

```py
def exp_opt(expansion: Expansion, attribute: str) -> Any:
  """Return the given attribution of an expansion.
 If attribute is not defined, return None"""
    return exp_opts(expansion).get(attribute, None) 
```

```py
exp_opts(("<term> + <expr>", opts(min_depth=10))) 
```

```py
{'min_depth': 10}

```

```py
exp_opt(("<term> - <expr>", opts(max_depth=2)), 'max_depth') 
```

```py
2

```

最后，我们定义一个设置特定选项的辅助函数：

```py
def set_opts(grammar: Grammar, symbol: str, expansion: Expansion, 
             opts: Option = {}) -> None:
  """Set the options of the given expansion of grammar[symbol] to opts"""
    expansions = grammar[symbol]
    for i, exp in enumerate(expansions):
        if exp_string(exp) != exp_string(expansion):
            continue

        new_opts = exp_opts(exp)
        if opts == {} or new_opts == {}:
            new_opts = opts
        else:
            for key in opts:
                new_opts[key] = opts[key]

        if new_opts == {}:
            grammar[symbol][i] = exp_string(exp)
        else:
            grammar[symbol][i] = (exp_string(exp), new_opts)

        return

    raise KeyError(
        "no expansion " +
        repr(symbol) +
        " -> " +
        repr(
            exp_string(expansion))) 
```</details>

## 检查语法

由于语法表示为字符串，因此引入错误相对容易。因此，让我们引入一个检查语法一致性的辅助函数。

辅助函数`is_valid_grammar()`遍历语法以检查是否所有使用的符号都已定义，反之亦然，这对于调试非常有用；它还检查是否所有符号都从起始符号可达。你不必深入了解细节，但像往常一样，在使用输入数据之前确保输入数据正确是非常重要的。

<details id="Excursion:-Implementing-is_valid_grammar()"><summary>实现`is_valid_grammar()`</summary>

```py
import [sys](https://docs.python.org/3/library/sys.html) 
```

```py
def def_used_nonterminals(grammar: Grammar, start_symbol: 
                          str = START_SYMBOL) -> Tuple[Optional[Set[str]], 
                                                       Optional[Set[str]]]:
  """Return a pair (`defined_nonterminals`, `used_nonterminals`) in `grammar`.
 In case of error, return (`None`, `None`)."""

    defined_nonterminals = set()
    used_nonterminals = {start_symbol}

    for defined_nonterminal in grammar:
        defined_nonterminals.add(defined_nonterminal)
        expansions = grammar[defined_nonterminal]
        if not isinstance(expansions, list):
            print(repr(defined_nonterminal) + ": expansion is not a list",
                  file=sys.stderr)
            return None, None

        if len(expansions) == 0:
            print(repr(defined_nonterminal) + ": expansion list empty",
                  file=sys.stderr)
            return None, None

        for expansion in expansions:
            if isinstance(expansion, tuple):
                expansion = expansion[0]
            if not isinstance(expansion, str):
                print(repr(defined_nonterminal) + ": "
                      + repr(expansion) + ": not a string",
                      file=sys.stderr)
                return None, None

            for used_nonterminal in nonterminals(expansion):
                used_nonterminals.add(used_nonterminal)

    return defined_nonterminals, used_nonterminals 
```

```py
def reachable_nonterminals(grammar: Grammar,
                           start_symbol: str = START_SYMBOL) -> Set[str]:
    reachable = set()

    def _find_reachable_nonterminals(grammar, symbol):
        nonlocal reachable
        reachable.add(symbol)
        for expansion in grammar.get(symbol, []):
            for nonterminal in nonterminals(expansion):
                if nonterminal not in reachable:
                    _find_reachable_nonterminals(grammar, nonterminal)

    _find_reachable_nonterminals(grammar, start_symbol)
    return reachable 
```

```py
def unreachable_nonterminals(grammar: Grammar,
                             start_symbol=START_SYMBOL) -> Set[str]:
    return grammar.keys() - reachable_nonterminals(grammar, start_symbol) 
```

```py
def opts_used(grammar: Grammar) -> Set[str]:
    used_opts = set()
    for symbol in grammar:
        for expansion in grammar[symbol]:
            used_opts |= set(exp_opts(expansion).keys())
    return used_opts 
```

```py
def is_valid_grammar(grammar: Grammar,
                     start_symbol: str = START_SYMBOL, 
                     supported_opts: Set[str] = set()) -> bool:
  """Check if the given `grammar` is valid.
 `start_symbol`: optional start symbol (default: `<start>`)
 `supported_opts`: options supported (default: none)"""

    defined_nonterminals, used_nonterminals = \
        def_used_nonterminals(grammar, start_symbol)
    if defined_nonterminals is None or used_nonterminals is None:
        return False

    # Do not complain about '<start>' being not used,
    # even if start_symbol is different
    if START_SYMBOL in grammar:
        used_nonterminals.add(START_SYMBOL)

    for unused_nonterminal in defined_nonterminals - used_nonterminals:
        print(repr(unused_nonterminal) + ": defined, but not used. Consider applying trim_grammar() on the grammar",
              file=sys.stderr)
    for undefined_nonterminal in used_nonterminals - defined_nonterminals:
        print(repr(undefined_nonterminal) + ": used, but not defined",
              file=sys.stderr)

    # Symbols must be reachable either from <start> or given start symbol
    unreachable = unreachable_nonterminals(grammar, start_symbol)
    msg_start_symbol = start_symbol

    if START_SYMBOL in grammar:
        unreachable = unreachable - \
            reachable_nonterminals(grammar, START_SYMBOL)
        if start_symbol != START_SYMBOL:
            msg_start_symbol += " or " + START_SYMBOL

    for unreachable_nonterminal in unreachable:
        print(repr(unreachable_nonterminal) + ": unreachable from " + msg_start_symbol + ". Consider applying trim_grammar() on the grammar",
              file=sys.stderr)

    used_but_not_supported_opts = set()
    if len(supported_opts) > 0:
        used_but_not_supported_opts = opts_used(
            grammar).difference(supported_opts)
        for opt in used_but_not_supported_opts:
            print(
                "warning: option " +
                repr(opt) +
                " is not supported",
                file=sys.stderr)

    return used_nonterminals == defined_nonterminals and len(unreachable) == 0 
```

为了使语法适合`is_valid_grammar()`，以下函数可能很有用。函数`trim_grammar()`自动*删除*对于不再需要的非终端的规则。如果你添加了新的规则，这些规则*删除*了一些扩展，使得一些非终端变得过时，这很有用。

```py
def trim_grammar(grammar: Grammar, start_symbol=START_SYMBOL) -> Grammar:
  """Create a copy of `grammar` where all unused and unreachable nonterminals are removed."""
    new_grammar = extend_grammar(grammar)
    defined_nonterminals, used_nonterminals = \
        def_used_nonterminals(grammar, start_symbol)
    if defined_nonterminals is None or used_nonterminals is None:
        return new_grammar

    unused = defined_nonterminals - used_nonterminals
    unreachable = unreachable_nonterminals(grammar, start_symbol)
    for nonterminal in unused | unreachable:
        del new_grammar[nonterminal]

    return new_grammar 
```</details>

让我们利用`is_valid_grammar()`。我们定义的语法通过了测试：

```py
assert is_valid_grammar(EXPR_GRAMMAR)
assert is_valid_grammar(CGI_GRAMMAR)
assert is_valid_grammar(URL_GRAMMAR) 
```

检查也可以应用于 EBNF 语法：

```py
assert is_valid_grammar(EXPR_EBNF_GRAMMAR) 
```

尽管如此，这些测试并没有通过：

```py
assert not is_valid_grammar({"<start>": ["<x>"], "<y>": ["1"]}) 
```

```py
'<y>': defined, but not used. Consider applying trim_grammar() on the grammar
'<x>': used, but not defined
'<y>': unreachable from <start>. Consider applying trim_grammar() on the grammar

```

```py
assert not is_valid_grammar({"<start>": "123"}) 
```

```py
'<start>': expansion is not a list

```

```py
assert not is_valid_grammar({"<start>": []}) 
```

```py
'<start>': expansion list empty

```

```py
assert not is_valid_grammar({"<start>": [1, 2, 3]}) 
```

```py
'<start>': 1: not a string

```

（`#type: ignore`注释避免了静态检查器将上述内容标记为错误）。

从现在开始，在定义语法时，我们总会使用`is_valid_grammar()`。

## 经验教训

+   语法是表达和生成语法上有效输入的有力工具。

+   从语法生成的输入可以直接使用，或者用作基于变异的模糊测试的种子。

+   语法可以通过字符类和运算符进行扩展，以简化编写过程。

## 下一步

由于它们为生成软件测试提供了一个很好的基础，因此在这本书中我们反复使用语法。作为预览，我们可以使用语法来 fuzz 配置：

```py
<options> ::= <option>*
<option> ::= -h | --version | -v | -d | -i | --global-config <filename>
```

我们可以使用语法来进行模糊测试函数和 API 以及模糊测试图形用户界面：

```py
<call-sequence> ::= <call>*
<call> ::= urlparse(<url>) | urlsplit(<url>)
```

我们可以将概率和约束分配给单个扩展：

```py
<term>: 50% <factor> * <term> |  30% <factor> / <term> | 20% <factor>
<integer>: <digit>+ { <integer> >= 100 }
```

所有这些额外功能都特别有价值，因为我们能够

1.  *自动推断语法*，无需手动指定，并且

1.  *引导它们向特定目标前进*，例如覆盖率或关键功能；

我们也讨论了本书中所有技术。

然而，要达到这个目标，我们仍然有一些作业要做。特别是，我们首先必须学会如何

+   创建一个高效的语法模糊器

## 背景

作为人类语言的基础之一，语法与人类语言一样历史悠久。生成语法的首次*形式化*是由公元前 350 年的 Dakṣiputra Pāṇini 完成的 [[Dakṣiputra Pāṇini, 350 BCE](https://en.wikipedia.org/wiki/P%C4%81%E1%B9%87ini%23A%E1%B9%A3%E1%B9%AD%C4%81dhy%C4%81y%C4%AB)]。作为表达数据和程序形式语言的一般手段，它们在计算机科学中的作用不容小觑。Chomsky [[Chomsky *et al*, 1956](https://chomsky.info/wp-content/uploads/195609-.pdf)] 的开创性工作引入了正则语言、上下文无关语法、上下文相关语法和通用语法的核心模型，这些模型自计算机科学中作为指定输入和编程语言的方法以来一直被使用（和教授）。

使用语法来*生成*测试输入可以追溯到 Burkhardt [[Burkhardt *et al*, 1967](https://doi.org/10.1007/BF02235512)]，后来被 Hanford [[Hanford *et al*, 1970](https://doi.org/10.1147/sj.94.0242)] 和 Purdom [[Purdom *et al*, 1972](https://doi.org/10.1007/BF01932308)] 重新发现并应用。从那时起，语法测试最重要的用途一直是*编译器测试*。实际上，基于语法的测试是编译器和 Web 浏览器能够正常工作的一个重要原因：

+   [CSmith](https://embed.cs.utah.edu/csmith/)工具 [[Yang *et al*, 2011](https://doi.org/10.1145/1993498.1993532)]专门针对 C 程序，从 C 语法开始，然后应用额外的步骤，例如引用之前定义的变量和函数或确保整数和类型安全。其作者们已经用它“找到并报告了 400 多个以前未知的编译器错误”。

+   与本书共享两位作者的[LangFuzz](http://issta2016.cispa.saarland/interview-with-christian-holler/)工作 [[Holler *et al*, 2012](https://www.usenix.org/system/files/conference/usenixsecurity12/sec12-final73.pdf)]，使用通用语法生成输出，并日夜不停地生成 JavaScript 程序并测试它们的解释器；截至今天，它已在 Mozilla Firefox、Google Chrome 和 Microsoft Edge 等浏览器中发现了超过 2,600 个错误。

+   [EMI 项目](http://web.cs.ucdavis.edu/~su/emi-project/) [[Le *et al*, 2014](https://doi.org/10.1145/2594291.2594334)] 使用语法来对 C 编译器进行压力测试，将已知的测试转换为在所有输入上语义等效的替代程序。这又导致了 C 编译器中超过 100 个错误的修复。

+   [Grammarinator](https://github.com/renatahodovan/grammarinator) [[Hodován 等人，2018](https://www.researchgate.net/publication/328510752_Grammarinator_a_grammar-based_open_source_fuzzer)] 是一个开源的语法模糊器（用 Python 编写！），使用流行的 ANTLR 格式作为语法规范。像 LangFuzz 一样，它使用语法进行解析和生成，并在 *JerryScript* 轻量级 JavaScript 引擎及其相关平台上发现了 100 多个问题。

+   [Domato](https://github.com/googleprojectzero/domato) 是一个通用的语法生成引擎，专门用于模糊 DOM 输入。它已经在流行的网络浏览器中揭示了许多安全问题。

编译器和网络浏览器，当然，不仅是需要语法进行测试的领域，也是语法广为人知的领域。本书中的主张是，语法可以用来生成几乎 *任何* 输入，我们的目标是赋予你做到这一点的能力。

## 练习

### 练习 1：JSON 语法

查看一下 [JSON 规范](http://www.json.org)，并从中推导出一个语法：

+   使用 *字符类* 来表达有效字符

+   使用 EBNF 表达重复和可选部分

+   假设

    +   一个字符串是一系列数字、ASCII 字母、标点符号和空格字符的序列，没有引号或转义符。

    +   空白只是一个空格。

+   使用 `is_valid_grammar()` 确保语法有效。

将语法输入到 `simple_grammar_fuzzer()`。你是否遇到任何错误，为什么？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Grammars.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 2：寻找错误

名称 `simple_grammar_fuzzer()` 并非偶然：它扩展语法的几种方式都有限制。如果你将 `simple_grammar_fuzzer()` 应用于上面定义的 `nonterminal_grammar` 和 `expr_grammar`，会发生什么？为什么？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Grammars.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 3：带有正则表达式的语法

在扩展了正则表达式的 *语法* 中，我们可以使用特殊形式

```py
/regex/
```

以包括正则表达式在扩展中。例如，我们可以有一个规则

```py
<integer> ::= /[+-]?[0-9]+/
```

以快速表达一个整数是可选的符号，后跟一系列数字。

#### 第一部分：将正则表达式转换为

编写一个转换器 `convert_regex(r)`，它接受一个正则表达式 `r` 并创建一个等效的语法。支持以下正则表达式构造：

+   `*`、`+`、`?`、`()` 应在 EBNF 中正常工作。

+   `a|b` 应转换为选择列表 `[a, b]`。

+   `.` 应匹配任何字符（除了换行符）。

+   `[abc]` 应转换为 `srange("abc")`

+   `[^abc]` 应转换为 ASCII 字符集 *除了* `srange("abc")`。

+   `[a-b]` 应转换为 `crange(a, b)`。

+   `[^a-b]` 应该转换为除了 `crange(a, b)` 的 ASCII 字符集。

示例：`convert_regex(r"[0-9]+")` 应该生成一个如下的文法

```py
{
    "<start>": ["<s1>"],
    "<s1>": [ "<s2>", "<s1><s2>" ],
    "<s2>": crange('0', '9')
} 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Grammars.ipynb#Exercises) 来完成练习并查看解决方案。

#### 第二部分：识别和扩展正则表达式

编写一个转换器 `convert_regex_grammar(g)`，它接收一个包含正则表达式形式的 EBNF 文法 `g`，并创建一个等效的 BNF 文法。支持上述正则表达式构造。

示例：`convert_regex_grammar({ "<integer>" : "/[+-]?[0-9]+/" })` 应该生成一个如下的文法

```py
{
    "<integer>": ["<s1><s3>"],
    "<s1>": [ "", "<s2>" ],
    "<s2>": srange("+-"),
    "<s3>": [ "<s4>", "<s4><s3>" ],
    "<s4>": crange('0', '9')
} 
```

可选：在正则表达式中支持转义：`\c` 转换为字面字符 `c`；`\/` 转换为 `/`（因此不会结束正则表达式）；`\\` 转换为 `\`。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Grammars.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 4：将文法定义为函数（高级）

为了获得更简洁的语法指定语法，可以使用 Python 构造，然后由一个额外的函数进行解析。例如，我们可以想象一个使用 `|` 作为分隔备选方案的语法定义：

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Grammars.ipynb#Exercises) 来完成练习并查看解决方案。

```py
def expression_grammar_fn():
    start = "<expr>"
    expr = "<term> + <expr>" | "<term> - <expr>"
    term = "<factor> * <term>" | "<factor> / <term>" | "<factor>"
    factor = "+<factor>" | "-<factor>" | "(<expr>)" | "<integer>.<integer>" | "<integer>"
    integer = "<digit><integer>" | "<digit>"
    digit = '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' 
```

如果我们执行 `expression_grammar_fn()`，这将产生一个错误。然而，`expression_grammar_fn()` 的目的不是被执行，而是作为 *数据* 使用，从这些数据中构建文法。

```py
with ExpectError():
    expression_grammar_fn() 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_8783/1271268731.py", line 2, in <module>
    expression_grammar_fn()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_8783/3029408019.py", line 3, in expression_grammar_fn
    expr = "<term> + <expr>" | "<term> - <expr>"
           ~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~
TypeError: unsupported operand type(s) for |: 'str' and 'str' (expected)

```

为了此目的，我们使用了 `ast`（抽象语法树）和 `inspect`（代码检查）模块。

```py
import [ast](https://docs.python.org/3/library/ast.html)
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

首先，我们获取 `expression_grammar_fn()` 的源代码...

```py
source = inspect.getsource(expression_grammar_fn)
source 
```

```py
'def expression_grammar_fn():\n    start = "<expr>"\n    expr = "<term> + <expr>" | "<term> - <expr>"\n    term = "<factor> * <term>" | "<factor> / <term>" | "<factor>"\n    factor = "+<factor>" | "-<factor>" | "(<expr>)" | "<integer>.<integer>" | "<integer>"\n    integer = "<digit><integer>" | "<digit>"\n    digit = \'0\' | \'1\' | \'2\' | \'3\' | \'4\' | \'5\' | \'6\' | \'7\' | \'8\' | \'9\'\n'

```

...然后将其解析为抽象语法树：

```py
tree = ast.parse(source) 
```

我们现在可以解析树以查找运算符和备选方案。`get_alternatives()` 遍历树中的所有节点 `op`；如果节点看起来像二进制 *或* (`|`) 操作，我们将进一步深入并递归。如果不是，我们就到达了一个单一的产生式，并尝试从产生式中获取表达式。我们根据我们想要如何表示产生式来定义 `to_expr` 参数。在这种情况下，我们用一个单独的字符串来表示一个单一的产生式。

```py
def get_alternatives(op, to_expr=lambda o: o.s):
    if isinstance(op, ast.BinOp) and isinstance(op.op, ast.BitOr):
        return get_alternatives(op.left, to_expr) + [to_expr(op.right)]
    return [to_expr(op)] 
```

`funct_parser()` 接收一个函数的抽象语法树（例如，`expression_grammar_fn()`），并遍历所有赋值：

```py
def funct_parser(tree, to_expr=lambda o: o.s):
    return {assign.targets[0].id: get_alternatives(assign.value, to_expr)
            for assign in tree.body[0].body} 
```

结果是一个我们正则格式的文法：

```py
grammar = funct_parser(tree)
for symbol in grammar:
    print(symbol, "::=", grammar[symbol]) 
```

```py
start ::= ['<expr>']
expr ::= ['<term> + <expr>', '<term> - <expr>']
term ::= ['<factor> * <term>', '<factor> / <term>', '<factor>']
factor ::= ['+<factor>', '-<factor>', '(<expr>)', '<integer>.<integer>', '<integer>']
integer ::= ['<digit><integer>', '<digit>']
digit ::= ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']

```

```py
/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_8783/1939255217.py:1: DeprecationWarning: Attribute s is deprecated and will be removed in Python 3.14; use value instead
  def funct_parser(tree, to_expr=lambda o: o.s):

```

#### 第一部分 (a)：一个单一函数

编写一个单独的函数 `define_grammar(fn)`，它接收一个定义为函数的文法（例如 `expression_grammar_fn()`），并返回一个正则文法。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Grammars.ipynb#Exercises) 来完成练习并查看解决方案。

#### 第一部分 (b)：替代表示法

我们注意到，我们之前设计的语法表示不允许简单地生成如 `srange()` 和 `crange()` 这样的备选方案。此外，人们可能会发现表达式的字符串表示有限制。实际上，扩展我们的语法定义以支持如下语法是简单的：

```py
def define_name(o):
    return o.id if isinstance(o, ast.Name) else o.s 
```

```py
def define_expr(op):
    if isinstance(op, ast.BinOp) and isinstance(op.op, ast.Add):
        return (*define_expr(op.left), define_name(op.right))
    return (define_name(op),) 
```

```py
def define_ex_grammar(fn):
    return define_grammar(fn, define_expr) 
```

语法：

```py
@define_ex_grammar
def expression_grammar():
    start   = expr
    expr    = (term + '+' + expr
            |  term + '-' + expr)
    term    = (factor + '*' + term
            |  factor + '/' + term
            |  factor)
    factor  = ('+' + factor
            |  '-' + factor
            |  '(' + expr + ')'
            |  integer + '.' + integer
            |  integer)
    integer = (digit + integer
            |  digit)
    digit   = '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'

for symbol in expression_grammar:
    print(symbol, "::=", expression_grammar[symbol]) 
```

**注意。** 因此获得的语法数据结构比标准数据结构更详细。它将每个产生式表示为一个元组。

我们注意到，我们尚未在上面的语法中启用 `srange()` 或 `crange()`。你将如何添加这些？ (*提示：将 `define_expr()` 包装起来以查找 `ast.Call`*)

#### 第二部分：扩展语法

引入一个操作符 `*`，它接受一个 `(min, max)` 对，其中 `min` 和 `max` 分别代表最小和最大重复次数。缺失的 `min` 值表示零；缺失的 `max` 值表示无穷大。

```py
def identifier_grammar_fn():
    identifier = idchar * (1,) 
```

使用 `*` 操作符，我们可以泛化 EBNF 操作符——`?` 变为 (0,1)，`*` 变为 (0,), 和 `+` 变为 (1,)。编写一个转换器，它接受使用 `*` 定义的扩展语法，解析它，并将其转换为 BNF。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Grammars.ipynb#Exercises) 来完成练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容根据 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可。内容的一部分源代码，以及用于格式化和显示该内容的源代码，根据 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 许可。 [最后更改：2024-06-30 18:31:28+02:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/Grammars.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[使用语法进行模糊测试](https://www.fuzzingbook.org/html/Grammars.html)". 在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 的 "[模糊测试书籍](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/Grammars.html`](https://www.fuzzingbook.org/html/Grammars.html). Retrieved 2024-06-30 18:31:28+02:00.

```py
@incollection{fuzzingbook2024:Grammars,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Fuzzing with Grammars},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/Grammars.html}},
    note = {Retrieved 2024-06-30 18:31:28+02:00},
    url = {https://www.fuzzingbook.org/html/Grammars.html},
    urldate = {2024-06-30 18:31:28+02:00}
}

```
