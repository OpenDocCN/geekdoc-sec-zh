# 模糊化 API

> 原文：[`www.fuzzingbook.org/html/APIFuzzer.html`](http://www.fuzzingbook.org/html/APIFuzzer.html)

到目前为止，我们总是生成*系统输入*，即程序通过其输入通道整体获取的数据。然而，我们也可以生成直接进入单个函数的输入，从而在过程中获得灵活性和速度。在本章中，我们探讨了使用语法来合成函数调用代码的使用，这允许您生成*非常高效地直接调用函数的程序代码*。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('CC1VvOGkzm8') 
```

**先决条件**

+   您必须了解语法模糊化是如何工作的，例如从语法章节中了解。

+   我们使用*生成器函数*，如生成器模糊化章节中讨论的那样。

+   我们使用概率，如概率模糊化章节中讨论的那样。

## 概述

要使用本章提供的代码（导入），请编写

```py
>>> from fuzzingbook.APIFuzzer import <identifier> 
```

然后利用以下功能。

本章提供了有用的*语法构造函数*，用于生成*函数调用*。

语法是概率性的，并使用生成器，因此使用`ProbabilisticGeneratorGrammarFuzzer`作为生产者。

```py
>>> from GeneratorGrammarFuzzer import ProbabilisticGeneratorGrammarFuzzer 
```

`INT_GRAMMAR`、`FLOAT_GRAMMAR`、`ASCII_STRING_GRAMMAR`分别产生整数、浮点数和字符串：

```py
>>> fuzzer = ProbabilisticGeneratorGrammarFuzzer(INT_GRAMMAR)
>>> [fuzzer.fuzz() for i in range(10)]
['-51', '9', '0', '0', '0', '0', '32', '0', '0', '0']
>>> fuzzer = ProbabilisticGeneratorGrammarFuzzer(FLOAT_GRAMMAR)
>>> [fuzzer.fuzz() for i in range(10)]
['0e0',
 '-9.43e34',
 '-7.3282e0',
 '-9.5e-9',
 '0',
 '-30.840386e-5',
 '3',
 '-4.1e0',
 '-9.7',
 '413']
>>> fuzzer = ProbabilisticGeneratorGrammarFuzzer(ASCII_STRING_GRAMMAR)
>>> [fuzzer.fuzz() for i in range(10)]
['"#vYV*t@I%KNTT[q~}&-v+[zAzj[X-z|RzC$(g$Br]1tC\':5<F-"',
 '""',
 '"^S/"',
 '"y)QDs_9"',
 '")dY~?WYqMh,bwn3\\"A!02Pk`gx"',
 '"01n|(dd$-d.sx\\"83\\"h/]qx)d9LPNdrk$}$4t3zhC.%3VY@AZZ0wCs2 N"',
 '"D\\6\\xgw#TQ}$\'3"',
 '"LaM{"',
 '"\\"ux\'1H!=%;2T$.=l"',
 '"=vkiV~w.Ypt,?JwcEr}Moc>!5<U+DdYAup\\"N 0V?h3x~jFN3"'] 
```

`int_grammar_with_range(start, end)`生成一个整数语法，其值`N`满足`start <= N <= end`：

```py
>>> int_grammar = int_grammar_with_range(100, 200)
>>> fuzzer = ProbabilisticGeneratorGrammarFuzzer(int_grammar)
>>> [fuzzer.fuzz() for i in range(10)]
['154', '149', '185', '117', '182', '154', '131', '194', '147', '192'] 
```

`float_grammar_with_range(start, end)`生成一个浮点数语法，其值`N`满足`start <= N <= end`。

```py
>>> float_grammar = float_grammar_with_range(100, 200)
>>> fuzzer = ProbabilisticGeneratorGrammarFuzzer(float_grammar)
>>> [fuzzer.fuzz() for i in range(10)]
['121.8092479227325',
 '187.18037169119634',
 '127.9576486784452',
 '125.47768739781723',
 '151.8091820472274',
 '117.864410860742',
 '187.50918008379483',
 '119.29335112884749',
 '149.2637029583114',
 '126.61818995939146'] 
```

所有这些值都可以立即用于测试函数调用：

```py
>>> from [math](https://docs.python.org/3/library/math.html) import sqrt
>>> fuzzer = ProbabilisticGeneratorGrammarFuzzer(int_grammar)
>>> call = "sqrt(" + fuzzer.fuzz() + ")"
>>> call
'sqrt(143)'
>>> eval(call)
11.958260743101398 
```

这些语法也可以组合成更复杂的语法。`list_grammar(object_grammar)`返回一个生成由`object_grammar`定义的对象列表的语法。

```py
>>> int_list_grammar = list_grammar(int_grammar)
>>> fuzzer = ProbabilisticGeneratorGrammarFuzzer(int_list_grammar)
>>> [fuzzer.fuzz() for i in range(5)]
['[118, 111, 188, 137, 129]',
 '[170, 172]',
 '[171, 161, 117, 191, 175, 183, 164]',
 '[189]',
 '[129, 110, 178]']
>>> some_list = eval(fuzzer.fuzz())
>>> some_list
[172, 120, 106, 192, 124, 191, 161, 100, 117]
>>> len(some_list)
9 
```

类似地，我们可以程序化地构造任意进一步的数据类型以测试单个函数。

## 模糊化函数

让我们从我们的第一个问题开始：我们如何模糊化一个给定的函数？对于像 Python 这样的解释型语言来说，这很简单。我们只需要生成我们想要测试的函数的调用。我们可以通过语法轻松地做到这一点。

以 Python 库中的`urlparse()`函数为例。`urlparse()`接受一个 URL 并将其分解为其各个组成部分。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) import urlparse 
```

```py
urlparse('https://www.fuzzingbook.com/html/APIFuzzer.html') 
```

```py
ParseResult(scheme='https', netloc='www.fuzzingbook.com', path='/html/APIFuzzer.html', params='', query='', fragment='')

```

您可以看到 URL 的各个组成部分——*方案*（`"http"`）、*网络位置*（`"www.fuzzingbook.com"`）或路径（`"//html/APIFuzzer.html"`）都被正确识别。其他元素（如`params`、`query`或`fragment`）为空，因为它们不是我们输入的一部分。

要测试`urlparse()`，我们希望给它提供大量不同的 URL。我们可以从在"语法章节中定义的 URL 语法中获取这些 URL。

```py
from Grammars import URL_GRAMMAR, is_valid_grammar, START_SYMBOL
from Grammars import opts, extend_grammar, Grammar
from GrammarFuzzer import GrammarFuzzer 
```

```py
url_fuzzer = GrammarFuzzer(URL_GRAMMAR) 
```

```py
for i in range(10):
    url = url_fuzzer.fuzz()
    print(urlparse(url)) 
```

```py
ParseResult(scheme='https', netloc='user:password@cispa.saarland:8080', path='/', params='', query='', fragment='')
ParseResult(scheme='http', netloc='cispa.saarland:1', path='/', params='', query='', fragment='')
ParseResult(scheme='https', netloc='fuzzingbook.com:7', path='', params='', query='', fragment='')
ParseResult(scheme='https', netloc='user:password@cispa.saarland:80', path='', params='', query='', fragment='')
ParseResult(scheme='ftps', netloc='user:password@fuzzingbook.com', path='', params='', query='', fragment='')
ParseResult(scheme='ftp', netloc='fuzzingbook.com', path='/abc', params='', query='abc=x31&def=x20', fragment='')
ParseResult(scheme='ftp', netloc='user:password@fuzzingbook.com', path='', params='', query='', fragment='')
ParseResult(scheme='https', netloc='www.google.com:80', path='/', params='', query='', fragment='')
ParseResult(scheme='http', netloc='fuzzingbook.com:52', path='/', params='', query='', fragment='')
ParseResult(scheme='ftps', netloc='user:password@cispa.saarland', path='', params='', query='', fragment='')

```

这样，我们可以轻松地测试任何 Python 函数——通过设置一个运行它的脚手架。但是，如果我们想要一个可以反复运行的测试，每次都不需要生成新的调用，我们应该如何进行呢？

## 合成代码

如上所述的“脚手架”方法有一个重要的缺点：它将测试生成和测试执行耦合成一个单一单元，不允许在不同时间或不同语言下运行。为了解耦这两个过程，我们采取另一种方法：而不是生成输入并立即将此输入馈入函数，我们*合成代码*来调用具有给定输入的函数。

例如，如果我们生成字符串

```py
call = "urlparse('http://www.cispa.de/')" 
```

我们可以在任何时间执行整个字符串（从而运行测试）：

```py
eval(call) 
```

```py
ParseResult(scheme='http', netloc='www.cispa.de', path='/', params='', query='', fragment='')

```

为了系统地生成这样的调用，我们再次可以使用一个语法：

```py
URLPARSE_GRAMMAR: Grammar = {
    "<call>":
        ['urlparse("<url>")']
}

# Import definitions from URL_GRAMMAR
URLPARSE_GRAMMAR.update(URL_GRAMMAR)
URLPARSE_GRAMMAR["<start>"] = ["<call>"]

assert is_valid_grammar(URLPARSE_GRAMMAR) 
```

这个语法创建形式为`urlparse(<url>)`的调用，其中`<url>`来自“导入”的 URL 语法。想法是创建许多这样的调用并将它们喂入 Python 解释器。

```py
URLPARSE_GRAMMAR 
```

```py
{'<call>': ['urlparse("<url>")'],
 '<start>': ['<call>'],
 '<url>': ['<scheme>://<authority><path><query>'],
 '<scheme>': ['http', 'https', 'ftp', 'ftps'],
 '<authority>': ['<host>',
  '<host>:<port>',
  '<userinfo>@<host>',
  '<userinfo>@<host>:<port>'],
 '<host>': ['cispa.saarland', 'www.google.com', 'fuzzingbook.com'],
 '<port>': ['80', '8080', '<nat>'],
 '<nat>': ['<digit>', '<digit><digit>'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<userinfo>': ['user:password'],
 '<path>': ['', '/', '/<id>'],
 '<id>': ['abc', 'def', 'x<digit><digit>'],
 '<query>': ['', '?<params>'],
 '<params>': ['<param>', '<param>&<params>'],
 '<param>': ['<id>=<id>', '<id>=<nat>']}

```

我们现在可以使用这个语法进行模糊测试和合成对`urlparse()`的调用：

```py
urlparse_fuzzer = GrammarFuzzer(URLPARSE_GRAMMAR)
urlparse_fuzzer.fuzz() 
```

```py
'urlparse("http://user:password@fuzzingbook.com:8080?abc=x29")'

```

正如上面一样，我们可以立即执行这些调用。为了更好地了解正在发生的事情，我们定义了一个小的辅助函数：

```py
# Call function_name(arg[0], arg[1], ...) as a string
def do_call(call_string):
    print(call_string)
    result = eval(call_string)
    print("\t= " + repr(result))
    return result 
```

```py
call = urlparse_fuzzer.fuzz()
do_call(call) 
```

```py
urlparse("http://www.google.com?abc=def")
	= ParseResult(scheme='http', netloc='www.google.com', path='', params='', query='abc=def', fragment='')

```

```py
ParseResult(scheme='http', netloc='www.google.com', path='', params='', query='abc=def', fragment='')

```

例如，如果`urlparse()`是一个 C 函数，我们可以将其调用嵌入到一些（也是生成的）C 函数中：

```py
URLPARSE_C_GRAMMAR: Grammar = {
    "<cfile>": ["<cheader><cfunction>"],
    "<cheader>": ['#include "urlparse.h"\n\n'],
    "<cfunction>": ["void test() {\n<calls>}\n"],
    "<calls>": ["<call>", "<calls><call>"],
    "<call>": ['    urlparse("<url>");\n']
} 
```

```py
URLPARSE_C_GRAMMAR.update(URL_GRAMMAR) 
```

```py
URLPARSE_C_GRAMMAR["<start>"] = ["<cfile>"] 
```

```py
assert is_valid_grammar(URLPARSE_C_GRAMMAR) 
```

```py
urlparse_fuzzer = GrammarFuzzer(URLPARSE_C_GRAMMAR)
print(urlparse_fuzzer.fuzz()) 
```

```py
#include "urlparse.h"

void test() {
    urlparse("http://user:password@cispa.saarland:99/x69?x57=abc");
}

```

## 合成 Oracle

在我们的`urlparse()`示例中，Python 和 C 变体都只检查`urlparse()`中的*通用*错误；也就是说，它们只检测致命错误和异常。为了进行全面测试，我们还需要设置一个特定的*Oracle*来检查结果是否有效。

我们的计划是检查 URL 的特定部分是否在结果中再次出现——也就是说，如果方案是`http:`，那么返回的`ParseResult`也应该包含一个`http:`方案。如关于使用生成器进行模糊测试的章节中讨论的那样，两个符号之间的字符串相等性（如`http:`）不能在上下文无关语法中表示。然而，我们可以使用一个*生成器函数*（也在关于使用生成器进行模糊测试的章节中介绍）来自动强制执行这样的相等性。

这里有一个例子。在`urlparse()`的结果上调用`geturl()`应该返回原始传递给`urlparse()`的 URL。

```py
from GeneratorGrammarFuzzer import GeneratorGrammarFuzzer, ProbabilisticGeneratorGrammarFuzzer 
```

```py
URLPARSE_ORACLE_GRAMMAR: Grammar = extend_grammar(URLPARSE_GRAMMAR,
{
     "<call>": [("assert urlparse('<url>').geturl() == '<url>'",
                 opts(post=lambda url_1, url_2: [None, url_1]))]
}) 
```

```py
urlparse_oracle_fuzzer = GeneratorGrammarFuzzer(URLPARSE_ORACLE_GRAMMAR)
test = urlparse_oracle_fuzzer.fuzz()
print(test) 
```

```py
assert urlparse('https://user:password@cispa.saarland/abc?abc=abc').geturl() == 'https://user:password@cispa.saarland/abc?abc=abc'

```

```py
exec(test) 
```

以类似的方式，我们也可以检查结果的单个组件：

```py
URLPARSE_ORACLE_GRAMMAR: Grammar = extend_grammar(URLPARSE_GRAMMAR,
{
     "<call>": [("result = urlparse('<scheme>://<host><path>?<params>')\n"
                 # + "print(result)\n"
                 + "assert result.scheme == '<scheme>'\n"
                 + "assert result.netloc == '<host>'\n"
                 + "assert result.path == '<path>'\n"
                 + "assert result.query == '<params>'",
                 opts(post=lambda scheme_1, authority_1, path_1, params_1,
                      scheme_2, authority_2, path_2, params_2:
                      [None, None, None, None,
                       scheme_1, authority_1, path_1, params_1]))]
})

# Get rid of unused symbols
del URLPARSE_ORACLE_GRAMMAR["<url>"]
del URLPARSE_ORACLE_GRAMMAR["<query>"]
del URLPARSE_ORACLE_GRAMMAR["<authority>"]
del URLPARSE_ORACLE_GRAMMAR["<userinfo>"]
del URLPARSE_ORACLE_GRAMMAR["<port>"] 
```

```py
urlparse_oracle_fuzzer = GeneratorGrammarFuzzer(URLPARSE_ORACLE_GRAMMAR)
test = urlparse_oracle_fuzzer.fuzz()
print(test) 
```

```py
result = urlparse('https://www.google.com/?def=18&abc=abc')
assert result.scheme == 'https'
assert result.netloc == 'www.google.com'
assert result.path == '/'
assert result.query == 'def=18&abc=abc'

```

```py
exec(test) 
```

使用生成器函数可能感觉有点繁琐。确实，如果我们只坚持使用 Python，我们也可以创建一个*单元测试*，该测试直接调用 fuzzer 来生成单个部分：

```py
def fuzzed_url_element(symbol):
    return GrammarFuzzer(URLPARSE_GRAMMAR, start_symbol=symbol).fuzz() 
```

```py
scheme = fuzzed_url_element("<scheme>")
authority = fuzzed_url_element("<authority>")
path = fuzzed_url_element("<path>")
query = fuzzed_url_element("<params>")
url = "%s://%s%s?%s" % (scheme, authority, path, query)
result = urlparse(url)
# print(result)
assert result.geturl() == url
assert result.scheme == scheme
assert result.path == path
assert result.query == query 
```

使用这样的单元测试可以更容易地表达预言机。然而，我们失去了像`GrammarCoverageFuzzer`那样系统地覆盖单个 URL 元素和替代方案的能力，以及像`ProbabilisticGrammarFuzzer`那样引导生成特定元素的能力。此外，语法允许我们为任意编程语言和 API 生成测试。

## 数据合成

对于`urlparse()`，我们使用了一个非常具体的语法来创建一个非常具体的参数。尽管许多函数将基本数据类型作为（某些）参数，但我们因此定义了生成精确参数的语法。更好的是，我们可以定义专门针对我们特定需求的生成语法函数，例如返回特定范围内的值。

### 整数

我们引入一个简单的语法来生成整数。

```py
from Grammars import convert_ebnf_grammar, crange 
```

```py
from ProbabilisticGrammarFuzzer import ProbabilisticGrammarFuzzer 
```

```py
INT_EBNF_GRAMMAR: Grammar = {
    "<start>": ["<int>"],
    "<int>": ["<_int>"],
    "<_int>": ["(-)?<leaddigit><digit>*", "0"],
    "<leaddigit>": crange('1', '9'),
    "<digit>": crange('0', '9')
}

assert is_valid_grammar(INT_EBNF_GRAMMAR) 
```

```py
INT_GRAMMAR = convert_ebnf_grammar(INT_EBNF_GRAMMAR)
INT_GRAMMAR 
```

```py
{'<start>': ['<int>'],
 '<int>': ['<_int>'],
 '<_int>': ['<symbol-1><leaddigit><digit-1>', '0'],
 '<leaddigit>': ['1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<symbol>': ['-'],
 '<symbol-1>': ['', '<symbol>'],
 '<digit-1>': ['', '<digit><digit-1>']}

```

```py
int_fuzzer = GrammarFuzzer(INT_GRAMMAR)
print([int_fuzzer.fuzz() for i in range(10)]) 
```

```py
['699', '-44', '321', '-7', '-6', '67', '0', '0', '57', '0']

```

如果我们需要特定范围内的整数，我们可以添加一个生成器函数来做到这一点：

```py
from Grammars import set_opts 
```

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
def int_grammar_with_range(start, end):
    int_grammar = extend_grammar(INT_GRAMMAR)
    set_opts(int_grammar, "<int>", "<_int>",
        opts(pre=lambda: random.randint(start, end)))
    return int_grammar 
```

```py
int_fuzzer = GeneratorGrammarFuzzer(int_grammar_with_range(900, 1000))
[int_fuzzer.fuzz() for i in range(10)] 
```

```py
['942', '955', '997', '967', '939', '923', '984', '914', '991', '982']

```

### 浮点数

浮点数的语法与整数语法非常相似。

```py
FLOAT_EBNF_GRAMMAR: Grammar = {
    "<start>": ["<float>"],
    "<float>": [("<_float>", opts(prob=0.9)), "inf", "NaN"],
    "<_float>": ["<int>(.<digit>+)?<exp>?"],
    "<exp>": ["e<int>"]
}
FLOAT_EBNF_GRAMMAR.update(INT_EBNF_GRAMMAR)
FLOAT_EBNF_GRAMMAR["<start>"] = ["<float>"]

assert is_valid_grammar(FLOAT_EBNF_GRAMMAR) 
```

```py
FLOAT_GRAMMAR = convert_ebnf_grammar(FLOAT_EBNF_GRAMMAR)
FLOAT_GRAMMAR 
```

```py
{'<start>': ['<float>'],
 '<float>': [('<_float>', {'prob': 0.9}), 'inf', 'NaN'],
 '<_float>': ['<int><symbol-2><exp-1>'],
 '<exp>': ['e<int>'],
 '<int>': ['<_int>'],
 '<_int>': ['<symbol-1-1><leaddigit><digit-1>', '0'],
 '<leaddigit>': ['1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<symbol>': ['.<digit-2>'],
 '<symbol-1>': ['-'],
 '<symbol-2>': ['', '<symbol>'],
 '<exp-1>': ['', '<exp>'],
 '<symbol-1-1>': ['', '<symbol-1>'],
 '<digit-1>': ['', '<digit><digit-1>'],
 '<digit-2>': ['<digit>', '<digit><digit-2>']}

```

```py
float_fuzzer = ProbabilisticGrammarFuzzer(FLOAT_GRAMMAR)
print([float_fuzzer.fuzz() for i in range(10)]) 
```

```py
['0', '-4e0', '-3.3', '0.55e0', '0e2', '0.2', '-48.6e0', '0.216', '-4.844', '-6.100']

```

```py
def float_grammar_with_range(start, end):
    float_grammar = extend_grammar(FLOAT_GRAMMAR)
    set_opts(float_grammar, "<float>", "<_float>", opts(
        pre=lambda: start + random.random() * (end - start)))
    return float_grammar 
```

```py
float_fuzzer = ProbabilisticGeneratorGrammarFuzzer(
    float_grammar_with_range(900.0, 900.9))
[float_fuzzer.fuzz() for i in range(10)] 
```

```py
['900.1695968039919',
 '900.3273891873373',
 '900.225192820568',
 '900.3231805358258',
 '900.4963527393471',
 'inf',
 'inf',
 '900.6037658059212',
 '900.6212350658716',
 '900.3877831415683']

```

### 字符串

最后，我们引入一个用于生成字符串的语法。

```py
ASCII_STRING_EBNF_GRAMMAR: Grammar = {
    "<start>": ["<ascii-string>"],
    "<ascii-string>": ['"<ascii-chars>"'],
    "<ascii-chars>": [
        ("", opts(prob=0.05)),
        "<ascii-chars><ascii-char>"
    ],
    "<ascii-char>": crange(" ", "!") + [r'\"'] + crange("#", "~")
}

assert is_valid_grammar(ASCII_STRING_EBNF_GRAMMAR) 
```

```py
ASCII_STRING_GRAMMAR = convert_ebnf_grammar(ASCII_STRING_EBNF_GRAMMAR) 
```

```py
string_fuzzer = ProbabilisticGrammarFuzzer(ASCII_STRING_GRAMMAR)
print([string_fuzzer.fuzz() for i in range(10)]) 
```

```py
['"BgY)"', '"j[-64Big65wso(f:wg|}w&*D9JthLX}0@PT^]mr[`69Cq8H713ITYx<#jpml)\\""', '"{);XWZJ@d`\'[h#F{1)C9M?%C`="', '"Y"', '"C4gh`?uzJzD~$\\\\"=|j)jj=SrBLIJ@0IbYiwIvNf5#pT4QUR}[g,35?Wg4i?3TdIsR0|eq3r;ZKuyI\'<\\"[p/x$<$B!\\"_"', '"J0HG33+E(p8JQtKW.;G7 ^?."', '"7r^B:Jf*J.@sqfED|M)3,eJ&OD"', '"c3Hcx^&*~3\\"Jvac}cX"', '"\'IHBQ:N+U:w(OAFn0pHLzX"', '"x4agH>H-2{Q|\\kpYF"']

```

## 合成复杂数据

如上所述，我们还可以从基本数据生成数据结构（如集合或列表）中的*复杂数据*。我们以列表为例说明这种生成。

### 列表

```py
LIST_EBNF_GRAMMAR: Grammar = {
    "<start>": ["<list>"],
    "<list>": [
        ("[]", opts(prob=0.05)),
        "[<list-objects>]"
    ],
    "<list-objects>": [
        ("<list-object>", opts(prob=0.2)),
        "<list-object>, <list-objects>"
    ],
    "<list-object>": ["0"],
}

assert is_valid_grammar(LIST_EBNF_GRAMMAR) 
```

```py
LIST_GRAMMAR = convert_ebnf_grammar(LIST_EBNF_GRAMMAR) 
```

我们列表生成器接受一个生成对象的语法；然后它使用这些语法中的对象实例化一个列表语法。

```py
def list_grammar(object_grammar, list_object_symbol=None):
    obj_list_grammar = extend_grammar(LIST_GRAMMAR)
    if list_object_symbol is None:
        # Default: Use the first expansion of <start> as list symbol
        list_object_symbol = object_grammar[START_SYMBOL][0]

    obj_list_grammar.update(object_grammar)
    obj_list_grammar[START_SYMBOL] = ["<list>"]
    obj_list_grammar["<list-object>"] = [list_object_symbol]

    assert is_valid_grammar(obj_list_grammar)

    return obj_list_grammar 
```

```py
int_list_fuzzer = ProbabilisticGrammarFuzzer(list_grammar(INT_GRAMMAR))
[int_list_fuzzer.fuzz() for i in range(10)] 
```

```py
['[0, -4, 23, 0, 0, 9, 0, -6067681]',
 '[-1, -1, 0, -7]',
 '[-5, 0]',
 '[1, 0, -628088, -6, -811, 0, 99, 0]',
 '[-35, -10, 0, 67]',
 '[-3, 0, -2, 0, 0]',
 '[0, -267, -78, -733, 0, 0, 0, 0]',
 '[0, -6, 71, -9]',
 '[-72, 76, 0, 2]',
 '[0, 9, 0, 0, -572, 29, 8, 8, 0]']

```

```py
string_list_fuzzer = ProbabilisticGrammarFuzzer(
    list_grammar(ASCII_STRING_GRAMMAR))
[string_list_fuzzer.fuzz() for i in range(10)] 
```

```py
['["gn-A$j>", "SPX;", "", "", ""]',
 '["_", "Qp"]',
 '["M", "5\\"`X744", "b+5fyM!", "gR`"]',
 '["^h", "8$u", "", "", ""]',
 '["6X;", "", "T1wp%\'t"]',
 '["-?Kk", "@B", "}", "", ""]',
 '["FD<mqK", ")Y4NI3M.&@1/2.p", "]C#c1}z#+5{7ERA[|", "EOFM])BEMFcGM.~k&RMj*,:m8^!5*:vv%ci"]',
 '["", "*B.pKI\\"L", "O)#<Y", "\\", "", "", ""]',
 '["g"]',
 '["", "\\JS;~t", "h)", "k", "", ""]']

```

```py
float_list_fuzzer = ProbabilisticGeneratorGrammarFuzzer(list_grammar(
    float_grammar_with_range(900.0, 900.9)))
[float_list_fuzzer.fuzz() for i in range(10)] 
```

```py
['[900.558064701869, 900.6079527708223, 900.1985188111297, 900.5159940886509, 900.1881413629061, 900.4074809145482, 900.8279453113845, 900.1531931708976, 900.2651056125504, inf, 900.828295978669]',
 '[900.4956935906264, 900.8166792417645, 900.2044872129637]',
 '[900.6177668624133, 900.793129850367, 900.5024769009476, 900.5874531663001, inf, 900.3476216137291, 900.5680329060473, 900.1524624203945, 900.1157565249836, 900.0943774301732, 900.1589468212459, 900.8563415304703, 900.2871041191156, 900.2469765832253, 900.408183791468]',
 '[NaN, 900.1152482126347, 900.1139109179966, NaN, 900.0634308730662, 900.1918596242257]',
 '[900.49418992478]',
 '[900.6566851795975, NaN, 900.5585085641878, 900.8678799526169, 900.5580757140183]',
 '[900.6265067760952]',
 '[900.5271187218734, 900.3413004135587, 900.0362652510535, 900.2938223153569, 900.6584186055829, 900.5394909707123, 900.5119630230411, 900.2024669591465]',
 '[900.5068304562362, 900.5173419618334, 900.5268996804168, 900.5247314889621, 900.1082421801126, 900.761200730868, 900.100950598924, 900.1424140649187, inf, inf, 900.4546924838603, 900.7025508468811, 900.5147250716594, 900.4943696257178, 900.814107878577, 900.3540228715348, 900.6165673939341, 900.121833279104, 900.8337503512706, 900.0607374037857, 900.2746253938637, 900.2491844866619, 900.7325728031923]',
 '[900.6962790125643, 900.6055198052603, 900.0950691946015, 900.6283670716376, NaN, 900.112869956762]']

```

字典、集合等生成器可以以类似的方式定义。通过连接语法生成器，我们可以生成具有任意元素的复杂数据结构。

## 经验教训

+   要模糊单个函数，可以轻松设置生成函数调用的语法。

+   在 API 级别进行模糊测试可能比在系统级别进行模糊测试要快得多，但会因违反隐含的先决条件而带来误报的风险。

## 下一步

本章全部关于手动编写测试和控制哪些数据被生成。在下一章中，我们将介绍一个更高层次的自动化：

+   *Carving*自动记录程序执行中的函数调用和参数。

+   我们可以将这些转换为*语法*，从而允许使用记录值的各种组合来测试这些函数。

使用这些技术，我们自动获得已经调用应用程序上下文中函数的语法，这使得我们指定它们的工作变得容易得多。

## 背景

使用生成器函数生成输入结构的想法最初在 QuickCheck [[Claessen *et al*, 2000](https://doi.org/10.1145/351240.351266)] 中被探索。Python 的一个非常好的实现是[hypothesis 包](https://hypothesis.readthedocs.io/en/latest/)，它允许编写和组合用于测试 API 的数据结构生成器。

## 练习

本章的练习结合了上述技术与之前介绍的模糊测试技术。

### 练习 1：深度参数

在为 `urlparse()` 生成或 acles 的示例中，重要的元素如 `authority` 或 `port` 并未检查。通过后扩展函数丰富 `URLPARSE_ORACLE_GRAMMAR`，这些函数将生成的元素存储在符号表中，以便在生成断言时可以访问。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/APIFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 2：覆盖论证组合

在关于 配置测试 的章节中，我们也讨论了 *组合测试* – 即对配置元素 *集合* 的系统覆盖。实现一个方案，通过改变语法，允许覆盖所有 *配对* 的参数值。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/APIFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 3：突变参数

为了扩大测试中使用的论证范围，应用在 突变模糊测试 中引入的 *突变方案* – 例如，翻转单个字节或从字符串中删除字符。在语法推理期间或作为调用函数时的单独步骤应用此方案。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/APIFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。内容的一部分源代码，以及用于格式化和显示该内容的源代码受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后更改：2023-11-11 18:18:05+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/APIFuzzer.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[模糊测试 API](https://www.fuzzingbook.org/html/APIFuzzer.html)". 在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 的 "[模糊测试书](https://www.fuzzingbook.org/)" 中，[`www.fuzzingbook.org/html/APIFuzzer.html`](https://www.fuzzingbook.org/html/APIFuzzer.html). Retrieved 2023-11-11 18:18:05+01:00.

```py
@incollection{fuzzingbook2023:APIFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Fuzzing APIs},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/APIFuzzer.html}},
    note = {Retrieved 2023-11-11 18:18:05+01:00},
    url = {https://www.fuzzingbook.org/html/APIFuzzer.html},
    urldate = {2023-11-11 18:18:05+01:00}
}

```
