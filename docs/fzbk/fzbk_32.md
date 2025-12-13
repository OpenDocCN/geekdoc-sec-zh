# 切割单元测试

> 原文：[`www.fuzzingbook.org/html/Carver.html`](http://www.fuzzingbook.org/html/Carver.html)

到目前为止，我们总是生成*系统输入*，即程序通过其输入通道整体获得的数据。如果我们只对测试一小组功能感兴趣，必须通过系统进行测试可能会非常低效。本章介绍了一种称为*切割*的技术，它给定一个系统测试，自动提取一组*单元测试*，这些测试复制了系统测试期间看到的调用。关键思想是*记录*这样的调用，以便我们可以在以后*回放*它们——整体或选择性地。此外，我们还探讨了如何从切割单元测试中合成 API 语法；这意味着我们可以*合成 API 测试而无需编写任何语法*。

**先决条件**

+   切割技术利用了函数调用和变量的动态跟踪，如配置模糊测试章节中所述。

+   使用语法测试单元在 API 模糊测试章节中介绍。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
import APIFuzzer 
```

## 概述

要使用本章提供的代码导入，请编写

```py
>>> from fuzzingbook.Carver import <identifier> 
```

然后利用以下功能。

本章提供了在系统测试期间记录和回放函数调用的方法。由于单个函数调用比整个系统运行快得多，这种“切割”机制有潜力使测试运行得更快。

### 记录调用

`CallCarver`类在活动期间记录所有发生的调用。它与`with`子句一起使用：

```py
>>> with CallCarver() as carver:
>>>     y = my_sqrt(2)
>>>     y = my_sqrt(4) 
```

执行后，`called_functions()`列出遇到的功能名称：

```py
>>> carver.called_functions()
['my_sqrt', '__exit__'] 
```

`arguments()`方法列出为函数记录的参数。这是一个将功能名称映射到参数列表的列表的映射；每个参数是一个参数名和值的对。

```py
>>> carver.arguments('my_sqrt')
[[('x', 2)], [('x', 4)]] 
```

复杂的参数被正确地序列化，这样它们可以很容易地恢复。

### 合成调用

虽然这样的记录参数已经可以转换为参数和调用，但一个更好的选择是创建一个*语法*来记录调用。这允许合成任意*组合*的参数，同时也为调用进一步定制提供了一个基础。

`CallGrammarMiner`类将切割执行的列表转换成一个语法。

```py
>>> my_sqrt_miner = CallGrammarMiner(carver)
>>> my_sqrt_grammar = my_sqrt_miner.mine_call_grammar()
>>> my_sqrt_grammar
{'<start>': ['<call>'],
 '<call>': ['<my_sqrt>'],
 '<my_sqrt-x>': ['2', '4'],
 '<my_sqrt>': ['my_sqrt(<my_sqrt-x>)']} 
```

这个语法可以用来合成调用。

```py
>>> fuzzer = GrammarCoverageFuzzer(my_sqrt_grammar)
>>> fuzzer.fuzz()
'my_sqrt(4)' 
```

这些调用可以单独执行，有效地从系统测试中提取单元测试：

```py
>>> eval(fuzzer.fuzz())
1.414213562373095 
```

## 系统测试与单元测试

记得为语法模糊测试引入的 URL 语法吗？有了这样的语法，我们可以愉快地再次测试 Web 浏览器，检查它对任意页面请求的反应。

让我们定义一个非常简单的“网络浏览器”，它会根据 URL 下载内容。

```py
import [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) 
```

```py
def webbrowser(url):
  """Download the http/https resource given by the URL"""
    import [requests](http://docs.python-requests.org/en/master/)  # Only import if needed

    r = requests.get(url)
    return r.text 
```

让我们在[fuzzingbook.org](https://www.fuzzingbook.org/)上应用这个方法并测量时间，使用计时器类：

```py
from Timer import Timer 
```

```py
with Timer() as webbrowser_timer:
    fuzzingbook_contents = webbrowser(
        "http://www.fuzzingbook.org/html/Fuzzer.html")

print("Downloaded %d bytes in %.2f seconds" %
      (len(fuzzingbook_contents), webbrowser_timer.elapsed_time())) 
```

```py
Downloaded 474839 bytes in 0.48 seconds

```

```py
fuzzingbook_contents[:100] 
```

```py
'\n<!-- A html document -->\n<!-- \nwith standard nbconvert css layout\nwith standard nbconvert input/out'

```

当然，一个完整的网络浏览器也会渲染 HTML 内容。我们可以使用这些命令（但我们不这样做，因为我们不想在这里复制整个网页）：

```py
from [IPython.display](https://ipython.readthedocs.io/en/stable/api/generated/IPython.display.html) import HTML, display
HTML(fuzzingbook_contents) 
```

不得不一次又一次地启动整个浏览器（或让它渲染一个网页）意味着有很多开销，尤其是如果我们只想测试其功能的一个子集。特别是，在代码更改后，我们更愿意只测试受更改影响的函数子集，而不是反复运行经过良好测试的函数。

让我们假设我们更改了处理解析给定 URL 并将其分解为各个元素（方案“http”）、网络位置（`"www.fuzzingbook.com"`）或路径（`"/html/Fuzzer.html"`）的函数——这个函数名为`urlparse()`：

```py
from [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) import urlparse 
```

```py
urlparse('https://www.fuzzingbook.com/html/Carver.html') 
```

```py
ParseResult(scheme='https', netloc='www.fuzzingbook.com', path='/html/Carver.html', params='', query='', fragment='')

```

您可以看到 URL 的各个组成部分——*方案*（`"http"`）、*网络位置*（`"www.fuzzingbook.com"`）或路径（`"//html/Carver.html"`）都被正确地识别。其他元素（如`params`、`query`或`fragment`）为空，因为它们不是我们输入的一部分。

有趣的是，仅执行`urlparse()`比运行整个`webbrowser()`快得多。让我们测量这个因子：

```py
runs = 1000
with Timer() as urlparse_timer:
    for i in range(runs):
        urlparse('https://www.fuzzingbook.com/html/Carver.html')

avg_urlparse_time = urlparse_timer.elapsed_time() / 1000
avg_urlparse_time 
```

```py
1.4512089546769858e-06

```

将其与网络浏览器所需的时间进行比较

```py
webbrowser_timer.elapsed_time() 
```

```py
0.48406379198422655

```

时间上的差异巨大：

```py
webbrowser_timer.elapsed_time() / avg_urlparse_time 
```

```py
333558.98916153726

```

因此，在运行`webbrowser()`一次所需的时间内，我们可以执行`urlparse()`——数十万次——而且这还不包括浏览器渲染下载的 HTML、运行包含的脚本以及网页加载时发生的其他事情所需的时间。因此，允许我们在*单元*级别进行测试的策略非常有前景，因为它们可以节省大量开销。

## 切割单元测试

在单元级别测试方法和函数需要非常了解要测试的各个单元以及它们与其他单元的交互。因此，设置适当的基础设施并手动编写单元测试既具有挑战性，又具有回报。然而，手动编写单元测试有一个有趣的替代方案。通过记录和回放函数调用的*切割*技术自动将系统测试转换为单元测试：

1.  在系统测试（给定或生成的）过程中，我们记录所有对函数的调用，包括函数读取的所有参数和其他变量。

1.  从这些中，我们合成一个自包含的*单元测试*，该测试重建了包含所有参数的函数调用。

1.  这个单元测试可以随时以高效率执行（回放）。

在本章剩余部分，让我们探索这些步骤。

## 记录调用

我们的首要挑战是记录函数调用及其参数。（为了简单起见，我们限制自己只记录参数，忽略函数读取的任何全局变量或其他非参数。）为了记录调用和参数，我们使用我们为覆盖率引入的机制：通过设置跟踪函数，我们跟踪所有进入单个函数的调用，同时也保存它们的参数。就像 `Coverage` 对象一样，我们希望使用 `Carver` 对象能够与 `with` 语句一起使用，这样我们就可以跟踪特定的代码块：

```py
with Carver() as carver:
    function_to_be_traced()
c = carver.calls() 
```

初始定义支持这种结构：

\todo{从 动态不变性 获取跟踪器}

```py
import [sys](https://docs.python.org/3/library/sys.html) 
```

```py
class Carver:
    def __init__(self, log=False):
        self._log = log
        self.reset()

    def reset(self):
        self._calls = {}

    # Start of `with` block
    def __enter__(self):
        self.original_trace_function = sys.gettrace()
        sys.settrace(self.traceit)
        return self

    # End of `with` block
    def __exit__(self, exc_type, exc_value, tb):
        sys.settrace(self.original_trace_function) 
```

实际工作发生在 `traceit()` 方法中，它记录了所有调用到 `_calls` 属性中。首先，我们定义两个辅助函数：

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
def get_qualified_name(code):
  """Return the fully qualified name of the current function"""
    name = code.co_name
    module = inspect.getmodule(code)
    if module is not None:
        name = module.__name__ + "." + name
    return name 
```

```py
def get_arguments(frame):
  """Return call arguments in the given frame"""
    # When called, all arguments are local variables
    local_variables = frame.f_locals.copy()
    arguments = [(var, frame.f_locals[var])
                 for var in local_variables]
    arguments.reverse()  # Want same order as call
    return arguments 
```

```py
class CallCarver(Carver):
    def add_call(self, function_name, arguments):
  """Add given call to list of calls"""
        if function_name not in self._calls:
            self._calls[function_name] = []
        self._calls[function_name].append(arguments)

    # Tracking function: Record all calls and all args
    def traceit(self, frame, event, arg):
        if event != "call":
            return None

        code = frame.f_code
        function_name = code.co_name
        qualified_name = get_qualified_name(code)
        arguments = get_arguments(frame)

        self.add_call(function_name, arguments)
        if qualified_name != function_name:
            self.add_call(qualified_name, arguments)

        if self._log:
            print(simple_call_string(function_name, arguments))

        return None 
```

最后，我们需要一些便利函数来访问调用：

```py
class CallCarver(CallCarver):
    def calls(self):
  """Return a dictionary of all calls traced."""
        return self._calls

    def arguments(self, function_name):
  """Return a list of all arguments of the given function
 as (VAR, VALUE) pairs.
 Raises an exception if the function was not traced."""
        return self._calls[function_name]

    def called_functions(self, qualified=False):
  """Return all functions called."""
        if qualified:
            return [function_name for function_name in self._calls.keys()
                    if function_name.find('.') >= 0]
        else:
            return [function_name for function_name in self._calls.keys()
                    if function_name.find('.') < 0] 
```

### 记录 my_sqrt()

让我们尝试我们的新 `Carver` 类——首先是一个非常简单的函数：

```py
from Intro_Testing import my_sqrt 
```

```py
with CallCarver() as sqrt_carver:
    my_sqrt(2)
    my_sqrt(4) 
```

我们可以检索所有看到的调用...

```py
sqrt_carver.calls() 
```

```py
{'my_sqrt': [[('x', 2)], [('x', 4)]],
 'Intro_Testing.my_sqrt': [[('x', 2)], [('x', 4)]],
 '__exit__': [[('tb', None),
   ('exc_value', None),
   ('exc_type', None),
   ('self', <__main__.CallCarver at 0x164d98e20>)]]}

```

```py
sqrt_carver.called_functions() 
```

```py
['my_sqrt', '__exit__']

```

...以及特定函数的参数：

```py
sqrt_carver.arguments("my_sqrt") 
```

```py
[[('x', 2)], [('x', 4)]]

```

我们定义了一个便利函数，以便更好地打印这些列表：

```py
def simple_call_string(function_name, argument_list):
  """Return function_name(arg[0], arg[1], ...) as a string"""
    return function_name + "(" + \
        ", ".join([var + "=" + repr(value)
                   for (var, value) in argument_list]) + ")" 
```

```py
for function_name in sqrt_carver.called_functions():
    for argument_list in sqrt_carver.arguments(function_name):
        print(simple_call_string(function_name, argument_list)) 
```

```py
my_sqrt(x=2)
my_sqrt(x=4)
__exit__(tb=None, exc_value=None, exc_type=None, self=<__main__.CallCarver object at 0x164d98e20>)

```

这是一个可以直接使用的语法来再次调用 `my_sqrt()`：

```py
eval("my_sqrt(x=2)") 
```

```py
1.414213562373095

```

### 雕刻 urlparse()

如果我们将此应用于 `webbrowser()` 会发生什么？

```py
with CallCarver() as webbrowser_carver:
    webbrowser("https://www.fuzzingbook.org") 
```

我们看到从网络检索 URL 需要相当多的功能：

```py
function_list = webbrowser_carver.called_functions(qualified=True)
len(function_list) 
```

```py
361

```

```py
print(function_list[:50]) 
```

```py
['requests.api.get', 'requests.api.request', 'requests.sessions.__init__', 'requests.utils.default_headers', 'requests.utils.default_user_agent', 'requests.structures.__init__', 'collections.abc.update', 'abc.__instancecheck__', 'requests.structures.__setitem__', 'requests.hooks.default_hooks', 'requests.hooks.<dictcomp>', 'requests.cookies.cookiejar_from_dict', 'http.cookiejar.__init__', 'threading.RLock', 'http.cookiejar.__iter__', 'requests.cookies.<listcomp>', 'http.cookiejar.deepvalues', 'http.cookiejar.vals_sorted_by_key', 'requests.adapters.__init__', 'urllib3.util.retry.__init__', 'urllib3.util.retry.<listcomp>', 'requests.adapters.init_poolmanager', 'urllib3.poolmanager.__init__', 'urllib3.request.__init__', 'urllib3._collections.__init__', 'requests.sessions.mount', 'requests.sessions.<listcomp>', 'requests.sessions.__enter__', 'requests.sessions.request', 'requests.models.__init__', 'requests.sessions.prepare_request', 'requests.cookies.merge_cookies', 'requests.cookies.update', 'requests.utils.get_netrc_auth', 'collections.abc.get', 'os.__getitem__', 'os.encode', 'requests.utils.<genexpr>', 'posixpath.expanduser', 'posixpath._get_sep', 'collections.abc.__contains__', 'os.decode', 'genericpath.exists', 'urllib.parse.urlparse', 'urllib.parse._coerce_args', 'urllib.parse.urlsplit', 'urllib.parse._splitnetloc', 'urllib.parse._checknetloc', 'urllib.parse._noop', 'netrc.__init__']

```

在许多其他函数中，我们还有一个对 `urlparse()` 的调用：

```py
urlparse_argument_list = webbrowser_carver.arguments("urllib.parse.urlparse")
urlparse_argument_list 
```

```py
[[('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')],
 [('allow_fragments', True),
  ('scheme', ''),
  ('url', 'https://www.fuzzingbook.org/')]]

```

再次，我们可以将其转换为格式良好的调用：

```py
urlparse_call = simple_call_string("urlparse", urlparse_argument_list[0])
urlparse_call 
```

```py
"urlparse(allow_fragments=True, scheme='', url='https://www.fuzzingbook.org')"

```

再次，我们可以重新执行这个调用：

```py
eval(urlparse_call) 
```

```py
ParseResult(scheme='https', netloc='www.fuzzingbook.org', path='', params='', query='', fragment='')

```

现在，我们已经成功地从 `webbrowser()` 执行中雕刻出了对 `urlparse()` 的调用。

## 回放调用

完整且普遍地回放调用很棘手，因为有几个挑战需要解决。这包括：

1.  我们需要能够 *访问* 单个函数。如果我们通过名称访问一个函数，该名称必须在作用域内。如果名称不可见（例如，因为它是一个模块内的名称），我们必须使其可见。

1.  任何在参数外部访问的 *资源* 必须被记录并重建以供回放。如果变量引用外部资源（如文件或网络资源），这可能很困难。

1.  *复杂对象* 也必须重建。

这些约束使得在函数与它的环境有大量交互时，雕刻变得困难甚至不可能。为了说明这些问题，考虑在 `webbrowser()` 中调用的 `email.parser.parse()` 方法：

```py
email_parse_argument_list = webbrowser_carver.arguments("email.parser.parse") 
```

对该方法的调用看起来像这样：

```py
email_parse_call = simple_call_string(
    "email.parser.Parser.parse",
    email_parse_argument_list[0])
email_parse_call 
```

```py
'email.parser.Parser.parse(headersonly=False, fp=<_io.StringIO object at 0x165160040>, self=<email.parser.Parser object at 0x164d9a3b0>)'

```

我们看到 `email.parser.Parser.parse()` 是 `email.parser.Parser` 对象（`self`）的一部分，并且它接收一个 `StringIO` 对象（`fp`）。这两个都是非原始值。我们如何可能重建它们？

### 对象序列化

复杂对象问题的答案在于创建一个*持久*的表示，可以在以后的某个时间点重建。这个过程被称为*序列化*；在 Python 中，它也被称为*pickle*。`pickle`模块提供了创建对象序列化表示的方法。让我们将此应用于我们刚刚找到的`email.parser.Parser`对象：

```py
import [pickle](https://docs.python.org/3/library/pickle.html) 
```

```py
email_parse_argument_list 
```

```py
[[('headersonly', False),
  ('fp', <_io.StringIO at 0x165160040>),
  ('self', <email.parser.Parser at 0x164d9a3b0>)]]

```

```py
parser_object = email_parse_argument_list[0][2][1]
parser_object 
```

```py
<email.parser.Parser at 0x164d9a3b0>

```

```py
pickled = pickle.dumps(parser_object)
pickled 
```

```py
b'\x80\x04\x95w\x00\x00\x00\x00\x00\x00\x00\x8c\x0cemail.parser\x94\x8c\x06Parser\x94\x93\x94)\x81\x94}\x94(\x8c\x06_class\x94\x8c\x0bhttp.client\x94\x8c\x0bHTTPMessage\x94\x93\x94\x8c\x06policy\x94\x8c\x11email._policybase\x94\x8c\x08Compat32\x94\x93\x94)\x81\x94ub.'

```

从表示序列化的`email.parser.Parser`对象的字符串中，我们可以在任何时间重新创建 Parser 对象：

```py
unpickled_parser_object = pickle.loads(pickled)
unpickled_parser_object 
```

```py
<email.parser.Parser at 0x1653cc430>

```

序列化机制使我们能够为所有作为参数传递的对象（假设它们可以被 pickle，即）生成表示。现在我们可以扩展`simple_call_string()`函数，使其自动序列化对象。此外，我们将其设置为，如果第一个参数名为`self`（即，它是一个类方法），我们将其作为`self`对象的方法。

```py
def call_value(value):
    value_as_string = repr(value)
    if value_as_string.find('<') >= 0:
        # Complex object
        value_as_string = "pickle.loads(" + repr(pickle.dumps(value)) + ")"
    return value_as_string 
```

```py
def call_string(function_name, argument_list):
  """Return function_name(arg[0], arg[1], ...) as a string, pickling complex objects"""
    if len(argument_list) > 0:
        (first_var, first_value) = argument_list[0]
        if first_var == "self":
            # Make this a method call
            method_name = function_name.split(".")[-1]
            function_name = call_value(first_value) + "." + method_name
            argument_list = argument_list[1:]

    return function_name + "(" + \
        ", ".join([var + "=" + call_value(value)
                   for (var, value) in argument_list]) + ")" 
```

让我们应用扩展的`call_string()`方法来创建对`email.parser.parse()`的调用，包括序列化的对象：

```py
call = call_string("email.parser.Parser.parse", email_parse_argument_list[0])
print(call) 
```

```py
email.parser.Parser.parse(headersonly=False, fp=pickle.loads(b'\x80\x04\x95\xc4\x02\x00\x00\x00\x00\x00\x00\x8c\x03_io\x94\x8c\x08StringIO\x94\x93\x94)\x81\x94(X\x9b\x02\x00\x00Connection: keep-alive\r\nContent-Length: 51336\r\nServer: GitHub.com\r\nContent-Type: text/html; charset=utf-8\r\nLast-Modified: Sat, 09 Nov 2024 16:09:36 GMT\r\nAccess-Control-Allow-Origin: *\r\nETag: W/"672f8940-4620a"\r\nexpires: Sat, 09 Nov 2024 17:02:19 GMT\r\nCache-Control: max-age=600\r\nContent-Encoding: gzip\r\nx-proxy-cache: MISS\r\nX-GitHub-Request-Id: 4FED:361A70:4094950:424934E:672F9343\r\nAccept-Ranges: bytes\r\nAge: 0\r\nDate: Sat, 09 Nov 2024 16:52:20 GMT\r\nVia: 1.1 varnish\r\nX-Served-By: cache-fra-eddf8230152-FRA\r\nX-Cache: MISS\r\nX-Cache-Hits: 0\r\nX-Timer: S1731171140.907105,VS0,VE105\r\nVary: Accept-Encoding\r\nX-Fastly-Request-ID: ca9f40b3c3e14ac63fadb8002a5b3b2d5be59d1b\r\n\r\n\x94\x8c\x01\n\x94M\x9b\x02Nt\x94b.'), self=pickle.loads(b'\x80\x04\x95w\x00\x00\x00\x00\x00\x00\x00\x8c\x0cemail.parser\x94\x8c\x06Parser\x94\x93\x94)\x81\x94}\x94(\x8c\x06_class\x94\x8c\x0bhttp.client\x94\x8c\x0bHTTPMessage\x94\x93\x94\x8c\x06policy\x94\x8c\x11email._policybase\x94\x8c\x08Compat32\x94\x93\x94)\x81\x94ub.'))

```

使用涉及序列化对象的这个调用，我们现在可以重新运行原始调用并获得有效结果：

```py
import [email](https://docs.python.org/3/library/email.html) 
```

```py
eval(call) 
```

```py
<http.client.HTTPMessage at 0x1653cd720>

```

### 所有调用

到目前为止，我们只看到了一次`webbrowser()`的调用。在`webbrowser()`内部，我们实际上可以雕刻和重放多少次调用？让我们尝试一下并计算数字。

```py
import [traceback](https://docs.python.org/3/library/traceback.html) 
```

```py
import [enum](https://docs.python.org/3/library/enum.html)
import [socket](https://docs.python.org/3/library/socket.html) 
```

```py
all_functions = set(webbrowser_carver.called_functions(qualified=True))
call_success = set()
run_success = set() 
```

```py
exceptions_seen = set()

for function_name in webbrowser_carver.called_functions(qualified=True):
    for argument_list in webbrowser_carver.arguments(function_name):
        try:
            call = call_string(function_name, argument_list)
            call_success.add(function_name)

            result = eval(call)
            run_success.add(function_name)

        except Exception as exc:
            exceptions_seen.add(repr(exc))
            # print("->", call, file=sys.stderr)
            # traceback.print_exc()
            # print("", file=sys.stderr)
            continue 
```

```py
print("%d/%d calls (%.2f%%) successfully created and %d/%d calls (%.2f%%) successfully ran" % (
    len(call_success), len(all_functions), len(
        call_success) * 100 / len(all_functions),
    len(run_success), len(all_functions), len(run_success) * 100 / len(all_functions))) 
```

```py
240/361 calls (66.48%) successfully created and 49/361 calls (13.57%) successfully ran

```

大约四分之一的调用成功。让我们看看我们得到的一些错误信息：

```py
for i in range(10):
    print(list(exceptions_seen)[i]) 
```

```py
NameError("name 'logging' is not defined")
TypeError("cannot pickle 'SSLSocket' object")
AttributeError("module 'enum' has no attribute '__call__'")
AttributeError("'NoneType' object has no attribute 'readline'")
NameError("name 'codecs' is not defined")
SyntaxError('invalid syntax', ('<string>', 1, 17, "requests.models.<genexpr>(.0=pickle.loads(b'\\x80\\x04\\x95\\x1b\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x8c\\x08builtins\\x94\\x8c\\x04iter\\x94\\x93\\x94]\\x94\\x85\\x94R\\x94.'))", 1, 18))
SyntaxError('invalid syntax', ('<string>', 1, 18, "urllib3.util.url.<genexpr>(.0=pickle.loads(b'\\x80\\x04\\x95\\x1c\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x8c\\x08builtins\\x94\\x8c\\x04iter\\x94\\x93\\x94\\x8c\\x00\\x94\\x85\\x94R\\x94.'))", 1, 19))
AttributeError("module 'email.parser' has no attribute 'parsestr'")
SyntaxError('invalid syntax', ('<string>', 1, 16, "requests.utils.<genexpr>(f='.netrc', .0=pickle.loads(b'\\x80\\x04\\x950\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x8c\\x08builtins\\x94\\x8c\\x04iter\\x94\\x93\\x94\\x8c\\x06.netrc\\x94\\x8c\\x06_netrc\\x94\\x86\\x94\\x85\\x94R\\x94K\\x01b.'))", 1, 17))
AttributeError("module 'email.message' has no attribute 'get'")

```

我们看到：

+   **大多数调用都可以转换为调用字符串**。如果不是这种情况，这主要是因为传递了非序列化对象。

+   **大约四分之一的调用可以执行**。失败运行的错误信息各不相同；最常见的是调用了一个不在作用域内的内部名称。

我们的雕刻机制应该谨慎对待：我们仍然没有涵盖访问外部变量和值（例如全局变量）的情况，序列化机制无法重新创建外部资源。尽管如此，如果感兴趣的函数属于那些*可以*雕刻和重放的那些函数，我们可以非常有效地使用它们的原始参数重新运行其调用。

## 从雕刻调用中挖掘 API 语法

到目前为止，我们使用雕刻调用来重放最初遇到的完全相同的调用。然而，我们也可以*变异*雕刻调用，以有效地使用先前记录的参数模糊 API。

一般思路如下：

1.  首先，我们记录程序给定执行中特定函数的所有调用。

1.  第二，我们创建一个语法，它包含所有这些调用，为每个参数提供单独的规则，并为每个找到的值提供替代方案；这允许我们产生任意*重新组合*这些参数的调用。

让我们在以下章节中探索这些步骤。

### 从调用到语法

让我们从例子开始。`power(x, y)` 函数返回 $x^y$；它只是 `math.pow()` 函数的包装器。（由于 `power()` 在 Python 中定义，我们可以跟踪它——与在 C 中实现的 `math.pow()` 相比。）

```py
import [math](https://docs.python.org/3/library/math.html) 
```

```py
def power(x, y):
    return math.pow(x, y) 
```

让我们调用 `power()` 并记录其参数：

```py
with CallCarver() as power_carver:
    z = power(1, 2)
    z = power(3, 4) 
```

```py
power_carver.arguments("power") 
```

```py
[[('y', 2), ('x', 1)], [('y', 4), ('x', 3)]]

```

从这个记录的参数列表中，我们现在可以创建一个用于 `power()` 调用的语法，其中 `x` 和 `y` 扩展为看到的值：

```py
from Grammars import START_SYMBOL, is_valid_grammar, new_symbol
from Grammars import extend_grammar, Grammar 
```

```py
POWER_GRAMMAR: Grammar = {
    "<start>": ["power(<x>, <y>)"],
    "<x>": ["1", "3"],
    "<y>": ["2", "4"]
}

assert is_valid_grammar(POWER_GRAMMAR) 
```

当使用此语法进行模糊测试时，我们得到 `x` 和 `y` 的任意组合；目标是确保所有值至少被测试一次：

```py
from GrammarCoverageFuzzer import GrammarCoverageFuzzer 
```

```py
power_fuzzer = GrammarCoverageFuzzer(POWER_GRAMMAR)
[power_fuzzer.fuzz() for i in range(5)] 
```

```py
['power(1, 2)', 'power(3, 4)', 'power(1, 2)', 'power(3, 4)', 'power(3, 4)']

```

我们需要一种方法，将 `power_carver` 中看到的参数自动转换为 `POWER_GRAMMAR` 中看到的语法。这就是我们在下一节中定义的内容。

### 调用语法挖掘器

我们引入了一个名为 `CallGrammarMiner` 的类，它接受一个 `Carver` 对象，并自动从看到的调用中生成语法。为了初始化，我们传递 carver 对象：

```py
class CallGrammarMiner:
    def __init__(self, carver, log=False):
        self.carver = carver
        self.log = log 
```

#### 初始语法

初始语法产生一个单一的调用。可能的 `<call>` 扩展将在以后构建：

```py
import [copy](https://docs.python.org/3/library/copy.html) 
```

```py
class CallGrammarMiner(CallGrammarMiner):
    CALL_SYMBOL = "<call>"

    def initial_grammar(self):
        return extend_grammar(
            {START_SYMBOL: [self.CALL_SYMBOL],
                self.CALL_SYMBOL: []
             }) 
```

```py
m = CallGrammarMiner(power_carver)
initial_grammar = m.initial_grammar()
initial_grammar 
```

```py
{'<start>': ['<call>'], '<call>': []}

```

#### 参数语法

让我们先从一个参数列表中创建一个语法。`mine_arguments_grammar()` 方法为 carving 过程中看到的参数创建一个语法，例如这些：

```py
arguments = power_carver.arguments("power")
arguments 
```

```py
[[('y', 2), ('x', 1)], [('y', 4), ('x', 3)]]

```

`mine_arguments_grammar()` 方法遍历看到的变量，并为每个变量名创建一个映射 `variables`，将变量名映射到一组看到的值（作为字符串，通过 `call_value()`）。在第二步中，它然后为每个变量名创建一个规则，扩展为看到的值。

```py
class CallGrammarMiner(CallGrammarMiner):
    def var_symbol(self, function_name, var, grammar):
        return new_symbol(grammar, "<" + function_name + "-" + var + ">")

    def mine_arguments_grammar(self, function_name, arguments, grammar):
        var_grammar = {}

        variables = {}
        for argument_list in arguments:
            for (var, value) in argument_list:
                value_string = call_value(value)
                if self.log:
                    print(var, "=", value_string)

                if value_string.find("<") >= 0:
                    var_grammar["<langle>"] = ["<"]
                    value_string = value_string.replace("<", "<langle>")

                if var not in variables:
                    variables[var] = set()
                variables[var].add(value_string)

        var_symbols = []
        for var in variables:
            var_symbol = self.var_symbol(function_name, var, grammar)
            var_symbols.append(var_symbol)
            var_grammar[var_symbol] = list(variables[var])

        return var_grammar, var_symbols 
```

```py
m = CallGrammarMiner(power_carver)
var_grammar, var_symbols = m.mine_arguments_grammar(
    "power", arguments, initial_grammar) 
```

```py
var_grammar 
```

```py
{'<power-y>': ['2', '4'], '<power-x>': ['3', '1']}

```

额外返回的 `var_symbols` 是调用中参数符号的列表：

```py
var_symbols 
```

```py
['<power-y>', '<power-x>']

```

#### 调用语法

要获取单个函数的语法（`mine_function_grammar()`），我们向函数添加一个调用：

```py
class CallGrammarMiner(CallGrammarMiner):
    def function_symbol(self, function_name, grammar):
        return new_symbol(grammar, "<" + function_name + ">")

    def mine_function_grammar(self, function_name, grammar):
        arguments = self.carver.arguments(function_name)

        if self.log:
            print(function_name, arguments)

        var_grammar, var_symbols = self.mine_arguments_grammar(
            function_name, arguments, grammar)

        function_grammar = var_grammar
        function_symbol = self.function_symbol(function_name, grammar)

        if len(var_symbols) > 0 and var_symbols[0].find("-self") >= 0:
            # Method call
            function_grammar[function_symbol] = [
                var_symbols[0] + "." + function_name + "(" + ", ".join(var_symbols[1:]) + ")"]
        else:
            function_grammar[function_symbol] = [
                function_name + "(" + ", ".join(var_symbols) + ")"]

        if self.log:
            print(function_symbol, "::=", function_grammar[function_symbol])

        return function_grammar, function_symbol 
```

```py
m = CallGrammarMiner(power_carver)
function_grammar, function_symbol = m.mine_function_grammar(
    "power", initial_grammar)
function_grammar 
```

```py
{'<power-y>': ['2', '4'],
 '<power-x>': ['3', '1'],
 '<power>': ['power(<power-y>, <power-x>)']}

```

额外返回的 `function_symbol` 包含刚刚添加的函数调用的名称：

```py
function_symbol 
```

```py
'<power>'

```

#### 所有调用的语法

现在我们重复上述步骤，以所有在 carving 过程中看到的函数调用。为此，我们只需遍历所有看到的函数调用：

```py
power_carver.called_functions() 
```

```py
['power', '__exit__']

```

```py
class CallGrammarMiner(CallGrammarMiner):
    def mine_call_grammar(self, function_list=None, qualified=False):
        grammar = self.initial_grammar()
        fn_list = function_list
        if function_list is None:
            fn_list = self.carver.called_functions(qualified=qualified)

        for function_name in fn_list:
            if function_list is None and (function_name.startswith("_") or function_name.startswith("<")):
                continue  # Internal function

            # Ignore errors with mined functions
            try:
                function_grammar, function_symbol = self.mine_function_grammar(
                    function_name, grammar)
            except:
                if function_list is not None:
                    raise

            if function_symbol not in grammar[self.CALL_SYMBOL]:
                grammar[self.CALL_SYMBOL].append(function_symbol)
            grammar.update(function_grammar)

        assert is_valid_grammar(grammar)
        return grammar 
```

`mine_call_grammar()` 方法是客户端可以且应该使用的方法——首先用于挖掘...

```py
m = CallGrammarMiner(power_carver)
power_grammar = m.mine_call_grammar()
power_grammar 
```

```py
{'<start>': ['<call>'],
 '<call>': ['<power>'],
 '<power-y>': ['2', '4'],
 '<power-x>': ['3', '1'],
 '<power>': ['power(<power-y>, <power-x>)']}

```

...然后进行模糊测试：

```py
power_fuzzer = GrammarCoverageFuzzer(power_grammar)
[power_fuzzer.fuzz() for i in range(5)] 
```

```py
['power(4, 3)', 'power(2, 1)', 'power(4, 3)', 'power(4, 3)', 'power(2, 3)']

```

通过这种方式，我们已经成功地从一个记录的执行中提取了一个语法；与“简单”的 carving 相比，我们的语法允许我们 *重新组合* 参数，从而在 API 层面上进行模糊测试。

## 模糊测试 Web 函数

现在我们将我们的语法挖掘器应用于更大的 API——我们在 carving 过程中已经遇到的 `urlparse()` 函数。

```py
with CallCarver() as webbrowser_carver:
    webbrowser("https://www.fuzzingbook.org") 
```

我们可以从遇到的调用中挖掘一个语法：

```py
m = CallGrammarMiner(webbrowser_carver)
webbrowser_grammar = m.mine_call_grammar() 
```

这是一个相当大的语法：

```py
call_list = webbrowser_grammar['<call>']
len(call_list) 
```

```py
136

```

```py
print(call_list[:20]) 
```

```py
['<webbrowser>', '<default_headers>', '<default_user_agent>', '<update>', '<default_hooks>', '<cookiejar_from_dict>', '<RLock>', '<deepvalues>', '<vals_sorted_by_key>', '<init_poolmanager>', '<mount>', '<prepare_request>', '<merge_cookies>', '<get_netrc_auth>', '<encode>', '<expanduser>', '<decode>', '<exists>', '<urlparse>', '<urlsplit>']

```

这是 `urlparse()` 函数的规则：

```py
webbrowser_grammar["<urlparse>"] 
```

```py
['urlparse(<urlparse-allow_fragments>, <urlparse-scheme>, <urlparse-url>)']

```

这里是参数。

```py
webbrowser_grammar["<urlparse-url>"] 
```

```py
["'https://www.fuzzingbook.org'", "'https://www.fuzzingbook.org/'"]

```

如果我们现在对这些规则应用模糊器，我们将系统地覆盖所有看到的参数变体，包括当然在 carving 过程中没有看到的组合。再次强调，我们在这里在 API 层面上进行模糊测试。

```py
urlparse_fuzzer = GrammarCoverageFuzzer(
    webbrowser_grammar, start_symbol="<urlparse>")
for i in range(5):
    print(urlparse_fuzzer.fuzz()) 
```

```py
urlparse(True, '', 'https://www.fuzzingbook.org')
urlparse(True, '', 'https://www.fuzzingbook.org/')
urlparse(True, '', 'https://www.fuzzingbook.org')
urlparse(True, '', 'https://www.fuzzingbook.org')
urlparse(True, '', 'https://www.fuzzingbook.org')

```

正如 carving 所看到的，在 API 级别运行测试比执行系统测试快得多。因此，这需要方法级别的模糊测试手段：

```py
from [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) import urlsplit 
```

```py
from Timer import Timer 
```

```py
with Timer() as urlsplit_timer:
    urlsplit('http://www.fuzzingbook.org/', 'http', True)
urlsplit_timer.elapsed_time() 
```

```py
1.2375006917864084e-05

```

```py
with Timer() as webbrowser_timer:
    webbrowser("http://www.fuzzingbook.org")
webbrowser_timer.elapsed_time() 
```

```py
0.31702329200925305

```

```py
webbrowser_timer.elapsed_time() / urlsplit_timer.elapsed_time() 
```

```py
25618.029477754102

```

但另一方面，在 carving 过程中遇到的问题也适用，特别是需要重新创建原始函数环境的要求。如果我们还更改或重新组合参数，我们还会面临 *违反隐含先决条件* 的额外风险——即调用一个从未为这些参数设计过的函数。这种由于调用错误而不是实现错误而产生的 *误报* 必须被识别（通常是手动）并排除（例如，通过更改或限制语法）。然而，在 API 级别的巨大速度提升可能很好地证明这种额外投资的合理性。

## 经验教训

+   *Carving* 允许在系统测试期间记录的功能调用进行有效的回放。

+   函数调用可以比系统调用快 *几个数量级*。

+   *序列化* 允许创建复杂对象的持久表示。

+   与其环境高度交互或访问外部资源的函数难以进行 carving。

+   从 carved 调用中，可以生成任意组合 carved 参数的 API 语法。

## 下一步

在下一章中，我们将讨论 如何减少导致失败的输入。

## 背景

Carving 被 Elbaum 等人发明 [[Elbaum *et al*, 2006](https://doi.org/10.1145/1181775.1181806)]，最初是为 Java 实现的。在本章中，我们遵循了他们的一些设计选择（包括仅记录和序列化方法参数）。

Carving 和 API 级别的模糊测试的组合在 [[Kampmann *et al*, 2018](https://arxiv.org/abs/1812.07932)] 中进行了描述。

## 练习

### 练习 1：用于回归测试的 Carving

到目前为止，在 carving 过程中，我们只关注了重现 *调用*，但并未检查这些调用的 *结果*。这对于 *回归测试* 非常重要——即检查代码的更改是否不会妨碍现有功能。我们可以通过记录不仅 *调用*，还包括 *返回值* 来实现这一点——然后稍后比较相同的调用是否产生相同的结果。这可能在所有情况下都不适用；依赖于时间、随机性或其他外部因素的价值可能不同。然而，对于抽象这些细节的功能，检查没有任何变化是测试的重要部分。

我们的目标是设计一个 `ResultCarver` 类，它通过记录调用和返回值来扩展 `CallCarver`。

在第一步中，创建一个 `traceit()` 方法，通过扩展 `traceit()` 方法来跟踪返回值。`traceit()` 事件类型是 `"return"`，`arg` 参数是返回值。以下是一个仅打印返回值的原型：

使用笔记本（[Use the notebook](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Carver.ipynb#Exercises)）来练习题目并查看解决方案。

```py
class ResultCarver(CallCarver):
    def traceit(self, frame, event, arg):
        if event == "return":
            if self._log:
                print("Result:", arg)

        super().traceit(frame, event, arg)
        # Need to return traceit function such that it is invoked for return
        # events
        return self.traceit 
```

```py
with ResultCarver(log=True) as result_carver:
    my_sqrt(2) 
```

```py
my_sqrt(x=2)
Result: 1.414213562373095
__exit__(tb=None, exc_value=None, exc_type=None, self=<__main__.ResultCarver object at 0x1653ccf10>)

```

#### 第一部分：存储函数结果

扩展上述代码，以便以将结果与当前返回的函数（或方法）关联的方式存储。为此，您需要跟踪当前调用的函数的 *调用栈*。

使用笔记本（[Use the notebook](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Carver.ipynb#Exercises)）来练习题目并查看解决方案。

#### 第二部分：访问结果

为它提供一个 `result()` 方法，该方法返回特定函数名称和结果的记录值：

```py
class ResultCarver(CallCarver):
    def result(self, function_name, argument):
  """Returns the result recorded for function_name(argument""" 
```

使用笔记本（[Use the notebook](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Carver.ipynb#Exercises)）来练习题目并查看解决方案。

#### 第三部分：生成断言

对于在 `webbrowser()` 执行期间调用的函数，创建一组 *断言* 来检查返回的结果是否仍然相同。为此测试 `urllib.parse.urlparse()`。

使用笔记本（[Use the notebook](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Carver.ipynb#Exercises)）来练习题目并查看解决方案。

### 练习 2：抽象参数

当从执行中挖掘 API 语法时，设置一个抽象方案以扩大测试期间使用的参数范围。如果一个参数的所有值都符合某种类型 `T`，则将其抽象为 `<T>`。例如，如果已经看到了对 `foo(1)`、`foo(2)`、`foo(3)` 的调用，则语法应将其调用抽象为 `foo(<int>)`，其中 `<int>` 被适当地定义。

对多种常见类型执行此操作：整数、正数、浮点数、主机名、URL、电子邮件地址等。

使用笔记本（[Use the notebook](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/Carver.ipynb#Exercises)）来练习题目并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后更改：2023-11-11 18:18:05+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/Carver.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser 和 Christian Holler: "[切割单元测试](https://www.fuzzingbook.org/html/Carver.html)". 在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser 和 Christian Holler 的 "[模糊测试书籍](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/Carver.html`](https://www.fuzzingbook.org/html/Carver.html). 获取时间：2023-11-11 18:18:05+01:00.

```py
@incollection{fuzzingbook2023:Carver,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Carving Unit Tests},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/Carver.html}},
    note = {Retrieved 2023-11-11 18:18:05+01:00},
    url = {https://www.fuzzingbook.org/html/Carver.html},
    urldate = {2023-11-11 18:18:05+01:00}
}

```
