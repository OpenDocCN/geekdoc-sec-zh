# 矿输入语法

> 原文：[`www.fuzzingbook.org/html/GrammarMiner.html`](http://www.fuzzingbook.org/html/GrammarMiner.html)

到目前为止，我们看到的语法大多是手动指定的——也就是说，您（或了解输入格式的人）最初必须设计和编写一个语法。虽然我们迄今为止看到的语法相对简单，但为复杂输入创建语法可能需要相当多的努力。因此，在本章中，我们介绍了从程序中*自动挖掘语法*的技术——通过执行程序并观察它们如何处理输入的哪些部分。结合语法模糊器，这使我们能够

1.  取一个程序，

1.  提取其输入语法，并且

1.  使用本书中的概念以高效率和效果进行模糊测试，

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo("ddM1oL2LYDI") 
```

**先决条件**

+   您应该已经阅读了语法章节。

+   配置模糊化章节介绍了配置选项的语法挖掘，以及执行过程中的变量和值观察。

+   我们使用覆盖率章节中的跟踪器。

+   从解析器章节中关于解析的概念也是很有用的。

## 概述

要使用本章节提供的代码（导入），请编写

```py
>>> from fuzzingbook.GrammarMiner import <identifier> 
```

然后利用以下功能。

本章提供了一些类，可以从现有程序中挖掘输入语法。函数`recover_grammar()`可能是最容易使用的。它接受一个函数和一组输入，并返回一个描述其输入语言的语法。

我们在`url_parse()`函数上应用`recover_grammar()`，该函数接收并分解 URL：

```py
>>> url_parse('https://www.fuzzingbook.org/')
>>> URLS
['http://user:pass@www.google.com:80/?q=path#ref',
 'https://www.cispa.saarland:80/',
 'http://www.fuzzingbook.org/#News'] 
```

我们使用`recover_grammar()`从`url_parse()`中提取输入语法：

```py
>>> grammar = recover_grammar(url_parse, URLS, files=['urllib/parse.py'])
>>> grammar
{'<start>': ['<urlsplit@452:url>'],
 '<urlsplit@452:url>': ['<urlparse@396:scheme>:<_splitnetloc@413:url>'],
 '<urlparse@396:scheme>': ['http', 'https'],
 '<_splitnetloc@413:url>': ['//<urlparse@396:netloc>/',
  '//<urlparse@396:netloc><urlsplit@494:url>'],
 '<urlparse@396:netloc>': ['www.cispa.saarland:80',
  'www.fuzzingbook.org',
  'user:pass@www.google.com:80'],
 '<urlsplit@494:url>': ['<urlsplit@502:url>#<urlparse@396:fragment>',
  '/#<urlparse@396:fragment>'],
 '<urlsplit@502:url>': ['/?<urlparse@396:query>'],
 '<urlparse@396:query>': ['q=path'],
 '<urlparse@396:fragment>': ['News', 'ref']} 
```

非终结符的名称有点技术性；但语法很好地代表了输入的结构；例如，不同的方案（`"http"`，`"https"`）都被识别出来：

```py
>>> syntax_diagram(grammar)
start 
```

<svg class="railroad-diagram" height="62" viewBox="0 0 276.0 62" width="276.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="138.0" y="35">urlsplit@452:url</text></g></g></g></g></svg>

```py
urlsplit@452:url
```

<svg class="railroad-diagram" height="62" viewBox="0 0 560.0 62" width="560.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">urlparse@396:scheme</text></g> <g class="terminal"><text x="275.75" y="35">:</text></g> <g class="non-terminal"><text x="405.0" y="35">_splitnetloc@413:url</text></g></g></g></g></svg>

```py
urlparse@396:scheme
```

<svg class="railroad-diagram" height="92" viewBox="0 0 182.5 92" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="91.25" y="35">http</text></g></g> <g><g class="terminal"><text x="91.25" y="65">https</text></g></g></g></g></svg>

```py
_splitnetloc@413:url
```

<svg class="railroad-diagram" height="92" viewBox="0 0 534.5 92" width="534.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="142.25" y="35">//</text></g> <g class="non-terminal"><text x="271.5" y="35">urlparse@396:netloc</text></g> <g class="terminal"><text x="396.5" y="35">/</text></g></g> <g><g class="terminal"><text x="78.5" y="65">//</text></g> <g class="non-terminal"><text x="207.75" y="65">urlparse@396:netloc</text></g> <g class="non-terminal"><text x="396.5" y="65">urlsplit@494:url</text></g></g></g></g></svg>

```py
urlparse@396:netloc
```

<svg class="railroad-diagram" height="122" viewBox="0 0 369.5 122" width="369.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="184.75" y="35">www.cispa.saarland:80</text></g></g> <g><g class="terminal"><text x="184.75" y="65">www.fuzzingbook.org</text></g></g> <g><g class="terminal"><text x="184.75" y="95">user:pass@www.google.com:80</text></g></g></g></g></svg>

```py
urlsplit@494:url
```

<svg class="railroad-diagram" height="92" viewBox="0 0 543.0 92" width="543.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="138.0" y="35">urlsplit@502:url</text></g> <g class="terminal"><text x="250.25" y="35">#</text></g> <g class="non-terminal"><text x="383.75" y="35">urlparse@396:fragment</text></g></g> <g><g class="terminal"><text x="162.25" y="65">/#</text></g> <g class="non-terminal"><text x="300.0" y="65">urlparse@396:fragment</text></g></g></g></g></svg>

```py
urlsplit@502:url
```

<svg class="railroad-diagram" height="62" viewBox="0 0 350.0 62" width="350.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="78.5" y="35">/?</text></g> <g class="non-terminal"><text x="203.5" y="35">urlparse@396:query</text></g></g></g></g></svg>

```py
urlparse@396:query
```

<svg class="railroad-diagram" height="62" viewBox="0 0 191.0 62" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="95.5" y="35">q=path</text></g></g></g></g></svg>

```py
urlparse@396:fragment
```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">News</text></g></g> <g><g class="terminal"><text x="87.0" y="65">ref</text></g></g></g></g></svg>

该语法可以立即用于模糊测试，生成任意组合的输入元素，这些元素都是语法上有效的。

```py
>>> from GrammarCoverageFuzzer import GrammarCoverageFuzzer
>>> fuzzer = GrammarCoverageFuzzer(grammar)
>>> [fuzzer.fuzz() for i in range(5)]
['https://www.cispa.saarland:80/#ref',
 'http://www.fuzzingbook.org/',
 'http://user:pass@www.google.com:80/?q=path#News',
 'https://www.fuzzingbook.org/?q=path#ref',
 'http://www.cispa.saarland:80/#News'] 
```

能够自动提取语法并使用该语法进行模糊测试，可以以最少的手动工作生成非常有效的测试。

## 语法挑战

考虑解析器章节中的`process_inventory()`方法：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import List, Tuple, Callable, Any
from [collections.abc](https://docs.python.org/3/library/collections.abc.html) import Iterable 
```

```py
from Parser import process_inventory, process_vehicle, process_car, process_van, lr_graph  # minor dependency 
```

它接受以下形式的输入。

```py
INVENTORY = """\
1997,van,Ford,E350
2000,car,Mercury,Cougar
1999,car,Chevy,Venture\
""" 
```

```py
print(process_inventory(INVENTORY)) 
```

```py
We have a Ford E350 van from 1997 vintage.
It is an old but reliable model!
We have a Mercury Cougar car from 2000 vintage.
It is an old but reliable model!
We have a Chevy Venture car from 1999 vintage.
It is an old but reliable model!

```

从解析器章节中我们发现，当输入格式包括仅在代码中表达的具体细节时，粗略的语法在模糊测试中效果不佳。也就是说，尽管我们有 CSV 文件的正式规范([RFC 4180](https://tools.ietf.org/html/rfc4180))，库存系统还包括针对 CSV 文件每个索引处期望内容的进一步规则。简单地重新组合现有输入虽然实用，但并不完整。特别是，它依赖于最初就存在正式的输入规范。然而，我们无法保证程序遵守所提供的输入规范。

解决这一困境的一种方法是对正在测试的程序进行询问，以了解其输入规范是什么。也就是说，如果正在测试的程序是以特定方法负责处理输入特定部分的方式编写的，那么可以通过观察解析过程来恢复解析树。此外，可以通过从多个输入树中抽象出合理的语法近似。

*我们首先假设（1）程序是以这样的方式编写的，即特定方法负责解析程序的特定片段--这几乎包括所有临时解析器*。

理念如下：

+   将钩子连接到 Python 执行中，并观察输入字符串片段在不同方法中被产生和命名。

+   将输入片段以树状结构拼接起来以检索**解析树**。

+   从多个解析树中提取公共元素以生成输入的**上下文无关语法**。

## 简单语法挖掘器

假设我们想要获取`process_vehicle()`函数的输入语法。我们首先收集这个函数的样本输入。

```py
VEHICLES = INVENTORY.split('\n') 
```

负责处理库存的方法集合如下。

```py
INVENTORY_METHODS = {
    'process_inventory',
    'process_vehicle',
    'process_van',
    'process_car'} 
```

我们从配置模糊测试章节中看到，可以连接到 Python 运行时来观察函数的参数和创建的任何局部变量。我们还看到，可以通过检查`frame`参数来获得执行上下文。这里有一个简单的跟踪器，可以返回跟踪函数中的局部变量和其他上下文信息。我们重用了`Coverage`跟踪类。

### 跟踪器

```py
from Coverage import Coverage 
```

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
class Tracer(Coverage):
    def traceit(self, frame, event, arg):
        method_name = inspect.getframeinfo(frame).function
        if method_name not in INVENTORY_METHODS:
            return
        file_name = inspect.getframeinfo(frame).filename

        param_names = inspect.getargvalues(frame).args
        lineno = inspect.getframeinfo(frame).lineno
        local_vars = inspect.getargvalues(frame).locals
        print(event, file_name, lineno, method_name, param_names, local_vars)
        return self.traceit 
```

我们在跟踪上下文中运行代码。

```py
with Tracer() as tracer:
    process_vehicle(VEHICLES[0]) 
```

```py
call Parser.ipynb 29 process_vehicle ['vehicle'] {'vehicle': '1997,van,Ford,E350'}
line Parser.ipynb 30 process_vehicle ['vehicle'] {'vehicle': '1997,van,Ford,E350'}
line Parser.ipynb 31 process_vehicle ['vehicle'] {'vehicle': '1997,van,Ford,E350', 'year': '1997', 'kind': 'van', 'company': 'Ford', 'model': 'E350', '_': []}
line Parser.ipynb 32 process_vehicle ['vehicle'] {'vehicle': '1997,van,Ford,E350', 'year': '1997', 'kind': 'van', 'company': 'Ford', 'model': 'E350', '_': []}
call Parser.ipynb 40 process_van ['year', 'company', 'model'] {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line Parser.ipynb 40 process_van ['year', 'company', 'model'] {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line Parser.ipynb 41 process_van ['year', 'company', 'model'] {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line Parser.ipynb 42 process_van ['year', 'company', 'model'] {'year': '1997', 'company': 'Ford', 'model': 'E350', 'res': ['We have a Ford E350 van from 1997 vintage.']}
line Parser.ipynb 43 process_van ['year', 'company', 'model'] {'year': '1997', 'company': 'Ford', 'model': 'E350', 'res': ['We have a Ford E350 van from 1997 vintage.'], 'iyear': 1997}
line Parser.ipynb 46 process_van ['year', 'company', 'model'] {'year': '1997', 'company': 'Ford', 'model': 'E350', 'res': ['We have a Ford E350 van from 1997 vintage.'], 'iyear': 1997}
line Parser.ipynb 47 process_van ['year', 'company', 'model'] {'year': '1997', 'company': 'Ford', 'model': 'E350', 'res': ['We have a Ford E350 van from 1997 vintage.', 'It is an old but reliable model!'], 'iyear': 1997}
return Parser.ipynb 47 process_van ['year', 'company', 'model'] {'year': '1997', 'company': 'Ford', 'model': 'E350', 'res': ['We have a Ford E350 van from 1997 vintage.', 'It is an old but reliable model!'], 'iyear': 1997}
return Parser.ipynb 32 process_vehicle ['vehicle'] {'vehicle': '1997,van,Ford,E350', 'year': '1997', 'kind': 'van', 'company': 'Ford', 'model': 'E350', '_': []}

```

我们从跟踪中想要得到的主要是输入片段分配给不同变量的列表。我们可以使用跟踪设施`settrace()`来获取，就像上面所展示的那样。

然而，`settrace()`函数连接到 Python 调试设施。当它在运行时，没有任何调试器可以连接到程序。也就是说，如果我们的语法挖掘器存在问题，我们将无法将其附加到调试器上来理解发生了什么。这并不理想。因此，我们将跟踪器限制在可能的最简单实现中，并在后续阶段实现语法挖掘的核心。

`traceit()`函数依赖于`frame`变量提供的信息，该变量暴露了 Python 的内部信息。我们定义了一个`context`类，用于封装我们从`frame`中需要的信息。

### 上下文

`Context`类提供了对当前模块和参数名称等信息轻松访问的接口。

```py
class Context:
    def __init__(self, frame, track_caller=True):
        self.method = inspect.getframeinfo(frame).function
        self.parameter_names = inspect.getargvalues(frame).args
        self.file_name = inspect.getframeinfo(frame).filename
        self.line_no = inspect.getframeinfo(frame).lineno

    def _t(self):
        return (self.file_name, self.line_no, self.method,
                ','.join(self.parameter_names))

    def __repr__(self):
        return "%s:%d:%s(%s)" % self._t() 
```

在这里，我们添加了一些方便的方法，这些方法在`frame`上操作以转换为`Context`。

```py
class Context(Context):
    def extract_vars(self, frame):
        return inspect.getargvalues(frame).locals

    def parameters(self, all_vars):
        return {k: v for k, v in all_vars.items() if k in self.parameter_names}

    def qualified(self, all_vars):
        return {"%s:%s" % (self.method, k): v for k, v in all_vars.items()} 
```

我们将打印上下文挂钩到`traceit()`，以观察其作用。首先，我们定义一个`log_event()`来显示事件。

```py
def log_event(event, var):
    print({'call': '->', 'return': '<-'}.get(event, '  '), var) 
```

并在`traceit()`函数中使用`log_event()`。

```py
class Tracer(Tracer):
    def traceit(self, frame, event, arg):
        log_event(event, Context(frame))
        return self.traceit 
```

在跟踪模式下运行`process_vehicle()`会打印遇到的上下文。

```py
with Tracer() as tracer:
    process_vehicle(VEHICLES[0]) 
```

```py
-> Parser.ipynb:29:process_vehicle(vehicle)
   Parser.ipynb:30:process_vehicle(vehicle)
   Parser.ipynb:31:process_vehicle(vehicle)
   Parser.ipynb:32:process_vehicle(vehicle)
-> Parser.ipynb:40:process_van(year,company,model)
   Parser.ipynb:40:process_van(year,company,model)
   Parser.ipynb:41:process_van(year,company,model)
   Parser.ipynb:42:process_van(year,company,model)
   Parser.ipynb:43:process_van(year,company,model)
   Parser.ipynb:46:process_van(year,company,model)
   Parser.ipynb:47:process_van(year,company,model)
<- Parser.ipynb:47:process_van(year,company,model)
<- Parser.ipynb:32:process_vehicle(vehicle)
-> Coverage.ipynb:102:__exit__(self,exc_type,exc_value,tb)
   Coverage.ipynb:105:__exit__(self,exc_type,exc_value,tb)

```

执行任何函数产生的跟踪信息可能会非常大。因此，我们需要将注意力限制在特定的模块上。此外，我们还专门将注意力限制在`str`变量上，因为这些变量更有可能包含输入片段。（我们将在练习中展示如何处理复杂对象。）

我们之前开发的`Context`类用于决定要监控哪些模块以及要跟踪哪些变量。

我们存储当前的*输入字符串*，以便可以用来确定是否有任何特定的字符串片段来自当前输入字符串。任何可选参数都单独处理。

```py
class Tracer(Tracer):
    def __init__(self, my_input, **kwargs):
        self.options(kwargs)
        self.my_input, self.trace = my_input, [] 
```

我们使用可选参数`files`来指示我们感兴趣的特定源文件，使用`methods`来指示感兴趣的特定方法。此外，我们还使用`log`来指定在跟踪期间是否启用详细日志记录。我们使用之前定义的`log_event()`方法进行日志记录。

选项处理如下。

```py
class Tracer(Tracer):
    def options(self, kwargs):
        self.files = kwargs.get('files', [])
        self.methods = kwargs.get('methods', [])
        self.log = log_event if kwargs.get('log') else lambda _evt, _var: None 
```

检查`files`和`methods`以确定特定事件是否应该被跟踪。

```py
class Tracer(Tracer):
    def tracing_context(self, cxt, event, arg):
        fres = not self.files or any(
            cxt.file_name.endswith(f) for f in self.files)
        mres = not self.methods or any(cxt.method == m for m in self.methods)
        return fres and mres 
```

与事件上下文类似，我们还想将注意力限制在特定的变量上。目前，我们只想关注字符串。（请参阅本章末尾的练习，了解如何将其扩展到其他类型的对象。）

```py
class Tracer(Tracer):
    def tracing_var(self, k, v):
        return isinstance(v, str) 
```

我们修改了`traceit()`，使其仅在特定事件上调用带有上下文信息的`on_event()`函数。

```py
class Tracer(Tracer):
    def on_event(self, event, arg, cxt, my_vars):
        self.trace.append((event, arg, cxt, my_vars))

    def create_context(self, frame):
        return Context(frame)

    def traceit(self, frame, event, arg):
        cxt = self.create_context(frame)
        if not self.tracing_context(cxt, event, arg):
            return self.traceit
        self.log(event, cxt)

        my_vars = {
            k: v
            for k, v in cxt.extract_vars(frame).items()
            if self.tracing_var(k, v)
        }
        self.on_event(event, arg, cxt, my_vars)
        return self.traceit 
```

`Tracer`类现在可以专注于特定文件上的特定类型的事件。此外，它为我们感兴趣的变量提供了一个一级过滤器。例如，我们只想关注包含输入片段的`process_*`方法中的变量。以下是我们的更新后的`Tracer`如何使用。

```py
with Tracer(VEHICLES[0], methods=INVENTORY_METHODS, log=True) as tracer:
    process_vehicle(VEHICLES[0]) 
```

```py
-> Parser.ipynb:29:process_vehicle(vehicle)
   Parser.ipynb:30:process_vehicle(vehicle)
   Parser.ipynb:31:process_vehicle(vehicle)
   Parser.ipynb:32:process_vehicle(vehicle)
-> Parser.ipynb:40:process_van(year,company,model)
   Parser.ipynb:40:process_van(year,company,model)
   Parser.ipynb:41:process_van(year,company,model)
   Parser.ipynb:42:process_van(year,company,model)
   Parser.ipynb:43:process_van(year,company,model)
   Parser.ipynb:46:process_van(year,company,model)
   Parser.ipynb:47:process_van(year,company,model)
<- Parser.ipynb:47:process_van(year,company,model)
<- Parser.ipynb:32:process_vehicle(vehicle)

```

执行产生了以下跟踪信息。

```py
for t in tracer.trace:
    print(t[0], t[2].method, dict(t[3])) 
```

```py
call process_vehicle {'vehicle': '1997,van,Ford,E350'}
line process_vehicle {'vehicle': '1997,van,Ford,E350'}
line process_vehicle {'vehicle': '1997,van,Ford,E350', 'year': '1997', 'kind': 'van', 'company': 'Ford', 'model': 'E350'}
line process_vehicle {'vehicle': '1997,van,Ford,E350', 'year': '1997', 'kind': 'van', 'company': 'Ford', 'model': 'E350'}
call process_van {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line process_van {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line process_van {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line process_van {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line process_van {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line process_van {'year': '1997', 'company': 'Ford', 'model': 'E350'}
line process_van {'year': '1997', 'company': 'Ford', 'model': 'E350'}
return process_van {'year': '1997', 'company': 'Ford', 'model': 'E350'}
return process_vehicle {'vehicle': '1997,van,Ford,E350', 'year': '1997', 'kind': 'van', 'company': 'Ford', 'model': 'E350'}

```

由于我们已经在`Tracer`中保存了输入，因此再次将输入作为参数指定是多余的。

```py
with Tracer(VEHICLES[0], methods=INVENTORY_METHODS, log=True) as tracer:
    process_vehicle(tracer.my_input) 
```

```py
-> Parser.ipynb:29:process_vehicle(vehicle)
   Parser.ipynb:30:process_vehicle(vehicle)
   Parser.ipynb:31:process_vehicle(vehicle)
   Parser.ipynb:32:process_vehicle(vehicle)
-> Parser.ipynb:40:process_van(year,company,model)
   Parser.ipynb:40:process_van(year,company,model)
   Parser.ipynb:41:process_van(year,company,model)
   Parser.ipynb:42:process_van(year,company,model)
   Parser.ipynb:43:process_van(year,company,model)
   Parser.ipynb:46:process_van(year,company,model)
   Parser.ipynb:47:process_van(year,company,model)
<- Parser.ipynb:47:process_van(year,company,model)
<- Parser.ipynb:32:process_vehicle(vehicle)

```

### DefineTracker

我们定义了一个`DefineTracker`类，用于处理来自`Tracer`的跟踪信息。其思路是存储不同的变量定义，这些定义是输入片段。

跟踪器识别出输入字符串中的字符串片段，并将它们存储在字典 `my_assignments` 中。它保存了跟踪记录，以及相应的输入以进行处理。最后，它调用 `process()` 来处理它所接收的 `trace`。我们将从一个依赖于某些假设的简单跟踪器开始，稍后我们将看到这些假设如何被放宽。

```py
class DefineTracker:
    def __init__(self, my_input, trace, **kwargs):
        self.options(kwargs)
        self.my_input = my_input
        self.trace = trace
        self.my_assignments = {}
        self.process() 
```

使用子串搜索的一个问题是，短字符串序列往往包含在其他字符串序列中，即使它们可能不是来自原始字符串。也就是说，如果输入片段是 `v`，它同样可能来自 `van` 或 `chevy`。我们依赖于能够预测给定片段在输入中确切位置的能力。因此，我们定义了一个常量 `FRAGMENT_LEN`，这样我们忽略长度不超过该长度的字符串。我们同样也加入了日志记录功能。

```py
FRAGMENT_LEN = 3 
```

```py
class DefineTracker(DefineTracker):
    def options(self, kwargs):
        self.log = log_event if kwargs.get('log') else lambda _evt, _var: None
        self.fragment_len = kwargs.get('fragment_len', FRAGMENT_LEN) 
```

我们的跟踪器简单地记录变量值，正如它们出现时。接下来，我们需要检查变量是否包含来自 **输入字符串** 的值。常见的做法是依赖于符号执行或至少动态污染，这些方法强大但复杂。然而，通过简单地依赖子串搜索可以获得一个合理的近似。也就是说，我们考虑任何产生的子串，如果它是原始输入字符串的子串，我们就认为它来自原始输入。

我们定义了一个 `is_input_fragment()` 方法，该方法依赖于字符串包含来检测字符串是否来自输入。

```py
class DefineTracker(DefineTracker):
    def is_input_fragment(self, var, value):
        return len(value) >= self.fragment_len and value in self.my_input 
```

我们可以使用 `is_input_fragment()` 来选择仅定义的变量子集，如下所示在 `fragments()` 中实现。

```py
class DefineTracker(DefineTracker):
    def fragments(self, variables):
        return {k: v for k, v in variables.items(
        ) if self.is_input_fragment(k, v)} 
```

跟踪器处理每个事件，并在每个事件中，它将包含输入字符串部分的当前局部变量更新到字典 `my_assignments` 中。请注意，这里有一个关于重新赋值期间发生什么的选择。我们可以丢弃所有重新赋值，或者只保留最后的赋值。在这里，我们选择后者。如果您想要前者行为，在存储片段之前检查值是否存在于 `my_assignments` 中。

```py
class DefineTracker(DefineTracker):
    def track_event(self, event, arg, cxt, my_vars):
        self.log(event, (cxt.method, my_vars))
        self.my_assignments.update(self.fragments(my_vars))

    def process(self):
        for event, arg, cxt, my_vars in self.trace:
            self.track_event(event, arg, cxt, my_vars) 
```

使用跟踪器，我们可以获取输入片段。例如，假设我们只对至少 `5` 个字符长的字符串感兴趣。

```py
tracker = DefineTracker(tracer.my_input, tracer.trace, fragment_len=5)
for k, v in tracker.my_assignments.items():
    print(k, '=', repr(v)) 
```

```py
vehicle = '1997,van,Ford,E350'

```

或者长度为 `2` 个字符的字符串（默认）。

```py
tracker = DefineTracker(tracer.my_input, tracer.trace)
for k, v in tracker.my_assignments.items():
    print(k, '=', repr(v)) 
```

```py
vehicle = '1997,van,Ford,E350'
year = '1997'
kind = 'van'
company = 'Ford'
model = 'E350'

```

```py
class DefineTracker(DefineTracker):
    def assignments(self):
        return self.my_assignments.items() 
```

### 组装推导树

```py
from Grammars import START_SYMBOL, syntax_diagram, \
    is_nonterminal, Grammar 
```

```py
from GrammarFuzzer import GrammarFuzzer, display_tree, \
    DerivationTree 
```

来自 `DefineTracker` 的输入片段只讲述了故事的一半。片段可能在解析的不同阶段被创建。因此，我们需要将片段组装成输入的推导树。基本思路如下：

我们上一步的输入是：

```py
"1997,van,Ford,E350" 
```

我们开始一个推导树，并将其与文法中的起始符号关联。

```py
derivation_tree: DerivationTree = (START_SYMBOL, [("1997,van,Ford,E350", [])]) 
```

```py
display_tree(derivation_tree) 
```

<svg width="120pt" height="73pt" viewBox="0.00 0.00 119.75 72.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 68.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="55.88" y="-51.2" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="55.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997,van,Ford,E350</text></g> <g id="edge1" class="edge"><title>0->1</title></g></g></svg>

下一个输入是：

```py
vehicle = "1997,van,Ford,E350" 
```

由于车辆节点完全覆盖了 `<start>` 节点的值，我们用车辆节点替换了该值。

```py
derivation_tree: DerivationTree = (START_SYMBOL, 
                                   [('<vehicle>', [("1997,van,Ford,E350", [])],
                                                   [])]) 
```

```py
display_tree(derivation_tree) 
```

<svg width="120pt" height="123pt" viewBox="0.00 0.00 119.75 122.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 118.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="55.88" y="-101.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="55.88" y="-51.2" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="55.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997,van,Ford,E350</text></g> <g id="edge2" class="edge"><title>1->2</title></g></g></svg>

下一个输入是：

```py
year = '1997' 
```

从 `<start>` 遍历推导树，我们看到它替换了 `<vehicle>` 节点值的一部分。因此，我们将 `<vehicle>` 节点的值分割成两个子节点，其中一个对应于值 `"1997"`，另一个对应于 `",van,Ford,E350"`，并将第一个替换为 `<year>` 节点。

```py
derivation_tree: DerivationTree = (START_SYMBOL, 
                                   [('<vehicle>', [('<year>', [('1997', [])]),
                                                   (",van,Ford,E350", [])], [])]) 
```

```py
display_tree(derivation_tree) 
```

<svg width="150pt" height="173pt" viewBox="0.00 0.00 150.25 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="59.88" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="59.88" y="-101.45" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="19.88" y="-51.2" font-family="Times,serif" font-size="14.00"><year></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="99.88" y="-51.2" font-family="Times,serif" font-size="14.00">,van,Ford,E350</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="19.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge3" class="edge"><title>2->3</title></g></g></svg>

我们对以下进行类似操作：

```py
company = 'Ford' 
```

```py
derivation_tree: DerivationTree = (START_SYMBOL, 
                                   [('<vehicle>', [('<year>', [('1997', [])]),
                                                   (",van,", []),
                                                   ('<company>', [('Ford', [])]),
                                                   (",E350", [])], [])]) 
```

```py
display_tree(derivation_tree) 
```

<svg width="228pt" height="173pt" viewBox="0.00 0.00 228.00 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="102.88" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="102.88" y="-101.45" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="19.88" y="-51.2" font-family="Times,serif" font-size="14.00"><year></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="70.88" y="-51.2" font-family="Times,serif" font-size="14.00">,van,</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="135.88" y="-51.2" font-family="Times,serif" font-size="14.00"><company></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="203.88" y="-51.2" font-family="Times,serif" font-size="14.00">,E350</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="19.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="135.88" y="-0.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge6" class="edge"><title>5->6</title></g></g></svg>

类似地，对于

```py
kind = 'van' 
```

和

```py
model = 'E350' 
```

```py
derivation_tree: DerivationTree = (START_SYMBOL, 
                                   [('<vehicle>', [('<year>', [('1997', [])]),
                                                   (",", []),
                                                   ("<kind>", [('van', [])]),
                                                   (",", []),
                                                   ('<company>', [('Ford', [])]),
                                                   (",", []),
                                                   ("<model>", [('E350', [])])
                                                   ], [])]) 
```

```py
display_tree(derivation_tree) 
```

<svg width="403pt" height="173pt" viewBox="0.00 0.00 403.38 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="178.88" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="178.88" y="-101.45" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="19.88" y="-51.2" font-family="Times,serif" font-size="14.00"><year></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="72.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="125.88" y="-51.2" font-family="Times,serif" font-size="14.00"><kind></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="178.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="244.88" y="-51.2" font-family="Times,serif" font-size="14.00"><company></text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="310.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge10" class="edge"><title>1->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="369.88" y="-51.2" font-family="Times,serif" font-size="14.00"><model></text></g> <g id="edge11" class="edge"><title>1->11</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="19.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="125.88" y="-0.95" font-family="Times,serif" font-size="14.00">van</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="244.88" y="-0.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="369.88" y="-0.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge12" class="edge"><title>11->12</title></g></g></svg>

我们现在根据上述描述的步骤开发完整的算法。`TreeMiner` 初始化为输入字符串、变量分配，并将其转换为相应的推导树。

```py
class TreeMiner:
    def __init__(self, my_input, my_assignments, **kwargs):
        self.options(kwargs)
        self.my_input = my_input
        self.my_assignments = my_assignments
        self.tree = self.get_derivation_tree()

    def options(self, kwargs):
        self.log = log_call if kwargs.get('log') else lambda _i, _v: None

    def get_derivation_tree(self):
        return (START_SYMBOL, []) 
```

`log_call()` 如下。

```py
def log_call(indent, var):
    print('\t' * indent, var) 
```

基本思想如下：

+   **目前，我们假设分配给变量的值是稳定的。也就是说，它不会被重新分配。特别是，没有递归调用，或者从不同部分对同一函数的多次调用。**（我们将在稍后展示如何克服这个限制）。

+   对于在 `my_assignments` 中找到的每个 *var*，*value* 对：

    1.  我们在推导树中递归地搜索 *value* `val` 的出现。

    1.  如果在节点 `P1` 的值 `V1` 中找到一个出现，我们将节点 `P1` 的值分成三部分，中间部分匹配 *value* `val`，第一部分和最后一部分是 `V1` 中的对应的前缀和后缀。

    1.  重新构建具有三个子节点的节点 `P1`，其中前面提到的前缀和后缀是字符串值，匹配的值 `val` 被替换为一个具有单个值 `val` 的节点 `var`。

首先，我们定义一个包装器来从变量名生成一个非终结符。

```py
def to_nonterminal(var):
    return "<" + var.lower() + ">" 
```

`string_part_of_value()` 方法检查给定的 `part` 值是否是整体的一部分。

```py
class TreeMiner(TreeMiner):
    def string_part_of_value(self, part, value):
        return (part in value) 
```

`partition_by_part()` 方法如果匹配，则按给定的部分分割 `value`，并返回一个包含第一个部分、被替换的部分和最后一个部分的列表。这是一个可以作为子列表一部分的格式。

```py
class TreeMiner(TreeMiner):
    def partition(self, part, value):
        return value.partition(part) 
```

```py
class TreeMiner(TreeMiner):
    def partition_by_part(self, pair, value):
        k, part = pair
        prefix_k_suffix = [
                    (k, [[part, []]]) if i == 1 else (e, [])
                    for i, e in enumerate(self.partition(part, value))
                    if e]
        return prefix_k_suffix 
```

`insert_into_tree()` 方法接受一个给定的树 `tree` 和一个 `(k,v)` 对。它递归地检查给定的对是否可以应用。如果对可以应用，它应用该对并返回 `True`。

```py
class TreeMiner(TreeMiner):
    def insert_into_tree(self, my_tree, pair):
        var, values = my_tree
        k, v = pair
        self.log(1, "- Node: %s\t\t? (%s:%s)" % (var, k, repr(v)))
        applied = False
        for i, value_ in enumerate(values):
            value, arr = value_
            self.log(2, "-> [%d] %s" % (i, repr(value)))
            if is_nonterminal(value):
                applied = self.insert_into_tree(value_, pair)
                if applied:
                    break
            elif self.string_part_of_value(v, value):
                prefix_k_suffix = self.partition_by_part(pair, value)
                del values[i]
                for j, rep in enumerate(prefix_k_suffix):
                    values.insert(j + i, rep)
                applied = True

                self.log(2, " > %s" % (repr([i[0] for i in prefix_k_suffix])))
                break
            else:
                continue
        return applied 
```

这是 `insert_into_tree()` 的用法。

```py
tree: DerivationTree = (START_SYMBOL, [("1997,van,Ford,E350", [])])
m = TreeMiner('', {}, log=True) 
```

首先，我们的输入字符串作为唯一的节点。

```py
display_tree(tree) 
```

<svg width="120pt" height="73pt" viewBox="0.00 0.00 119.75 72.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 68.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="55.88" y="-51.2" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="55.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997,van,Ford,E350</text></g> <g id="edge1" class="edge"><title>0->1</title></g></g></svg>

插入 `<vehicle>` 节点。

```py
v = m.insert_into_tree(tree, ('<vehicle>', "1997,van,Ford,E350")) 
```

```py
	 - Node: <start>		? (<vehicle>:'1997,van,Ford,E350')
		 -> [0] '1997,van,Ford,E350'
		  > ['<vehicle>']

```

```py
display_tree(tree) 
```

<svg width="120pt" height="123pt" viewBox="0.00 0.00 119.75 122.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 118.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="55.88" y="-101.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="55.88" y="-51.2" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="55.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997,van,Ford,E350</text></g> <g id="edge2" class="edge"><title>1->2</title></g></g></svg>

插入 `<model>` 节点。

```py
v = m.insert_into_tree(tree, ('<model>', 'E350')) 
```

```py
	 - Node: <start>		? (<model>:'E350')
		 -> [0] '<vehicle>'
	 - Node: <vehicle>		? (<model>:'E350')
		 -> [0] '1997,van,Ford,E350'
		  > ['1997,van,Ford,', '<model>']

```

```py
display_tree((tree)) 
```

<svg width="160pt" height="173pt" viewBox="0.00 0.00 160.12 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="83.62" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="83.62" y="-101.45" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="41.62" y="-51.2" font-family="Times,serif" font-size="14.00">1997,van,Ford,</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="126.62" y="-51.2" font-family="Times,serif" font-size="14.00"><model></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="126.62" y="-0.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge4" class="edge"><title>3->4</title></g></g></svg>

插入 `<company>`。

```py
v = m.insert_into_tree(tree, ('<company>', 'Ford')) 
```

```py
	 - Node: <start>		? (<company>:'Ford')
		 -> [0] '<vehicle>'
	 - Node: <vehicle>		? (<company>:'Ford')
		 -> [0] '1997,van,Ford,'
		  > ['1997,van,', '<company>', ',']

```

```py
display_tree(tree) 
```

<svg width="264pt" height="173pt" viewBox="0.00 0.00 263.50 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="138" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="138" y="-101.45" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="27" y="-51.2" font-family="Times,serif" font-size="14.00">1997,van,</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="105" y="-51.2" font-family="Times,serif" font-size="14.00"><company></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="171" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="230" y="-51.2" font-family="Times,serif" font-size="14.00"><model></text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="105" y="-0.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="230" y="-0.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge7" class="edge"><title>6->7</title></g></g></svg>

插入 `<kind>`。

```py
v = m.insert_into_tree(tree, ('<kind>', 'van')) 
```

```py
	 - Node: <start>		? (<kind>:'van')
		 -> [0] '<vehicle>'
	 - Node: <vehicle>		? (<kind>:'van')
		 -> [0] '1997,van,'
		  > ['1997,', '<kind>', ',']

```

```py
display_tree(tree) 
```

<svg width="347pt" height="173pt" viewBox="0.00 0.00 346.88 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="155.38" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="155.38" y="-101.45" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="15.38" y="-51.2" font-family="Times,serif" font-size="14.00">1997,</text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="69.38" y="-51.2" font-family="Times,serif" font-size="14.00"><kind></text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="122.38" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="188.38" y="-51.2" font-family="Times,serif" font-size="14.00"><company></text></g> <g id="edge6" class="edge"><title>1->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="254.38" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="313.38" y="-51.2" font-family="Times,serif" font-size="14.00"><model></text></g> <g id="edge9" class="edge"><title>1->9</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="69.38" y="-0.95" font-family="Times,serif" font-size="14.00">van</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="188.38" y="-0.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="313.38" y="-0.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge10" class="edge"><title>9->10</title></g></g></svg>

插入 `<year>`。

```py
v = m.insert_into_tree(tree, ('<year>', '1997')) 
```

```py
	 - Node: <start>		? (<year>:'1997')
		 -> [0] '<vehicle>'
	 - Node: <vehicle>		? (<year>:'1997')
		 -> [0] '1997,'
		  > ['<year>', ',']

```

```py
display_tree(tree) 
```

<svg width="403pt" height="173pt" viewBox="0.00 0.00 403.38 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="178.88" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="178.88" y="-101.45" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="19.88" y="-51.2" font-family="Times,serif" font-size="14.00"><year></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="72.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="125.88" y="-51.2" font-family="Times,serif" font-size="14.00"><kind></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="178.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="244.88" y="-51.2" font-family="Times,serif" font-size="14.00"><company></text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="310.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge10" class="edge"><title>1->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="369.88" y="-51.2" font-family="Times,serif" font-size="14.00"><model></text></g> <g id="edge11" class="edge"><title>1->11</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="19.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="125.88" y="-0.95" font-family="Times,serif" font-size="14.00">van</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="244.88" y="-0.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="369.88" y="-0.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge12" class="edge"><title>11->12</title></g></g></svg>

为了简化生活，我们定义了一个包装函数 `nt_var()`，它将把一个标记转换为相应的非终结符符号。

```py
class TreeMiner(TreeMiner):
    def nt_var(self, var):
        return var if is_nonterminal(var) else to_nonterminal(var) 
```

现在，我们需要将一个新的定义应用到整个语法中。

```py
class TreeMiner(TreeMiner):
    def apply_new_definition(self, tree, var, value):
        nt_var = self.nt_var(var)
        return self.insert_into_tree(tree, (nt_var, value)) 
```

此算法实现为 `get_derivation_tree()`。

```py
class TreeMiner(TreeMiner):
    def get_derivation_tree(self):
        tree = (START_SYMBOL, [(self.my_input, [])])

        for var, value in self.my_assignments:
            self.log(0, "%s=%s" % (var, repr(value)))
            self.apply_new_definition(tree, var, value)
        return tree 
```

`TreeMiner` 的用法如下：

```py
with Tracer(VEHICLES[0]) as tracer:
    process_vehicle(tracer.my_input)
assignments = DefineTracker(tracer.my_input, tracer.trace).assignments()
dt = TreeMiner(tracer.my_input, assignments, log=True)
dt.tree 
```

```py
 vehicle='1997,van,Ford,E350'
	 - Node: <start>		? (<vehicle>:'1997,van,Ford,E350')
		 -> [0] '1997,van,Ford,E350'
		  > ['<vehicle>']
 year='1997'
	 - Node: <start>		? (<year>:'1997')
		 -> [0] '<vehicle>'
	 - Node: <vehicle>		? (<year>:'1997')
		 -> [0] '1997,van,Ford,E350'
		  > ['<year>', ',van,Ford,E350']
 kind='van'
	 - Node: <start>		? (<kind>:'van')
		 -> [0] '<vehicle>'
	 - Node: <vehicle>		? (<kind>:'van')
		 -> [0] '<year>'
	 - Node: <year>		? (<kind>:'van')
		 -> [0] '1997'
		 -> [1] ',van,Ford,E350'
		  > [',', '<kind>', ',Ford,E350']
 company='Ford'
	 - Node: <start>		? (<company>:'Ford')
		 -> [0] '<vehicle>'
	 - Node: <vehicle>		? (<company>:'Ford')
		 -> [0] '<year>'
	 - Node: <year>		? (<company>:'Ford')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind>'
	 - Node: <kind>		? (<company>:'Ford')
		 -> [0] 'van'
		 -> [3] ',Ford,E350'
		  > [',', '<company>', ',E350']
 model='E350'
	 - Node: <start>		? (<model>:'E350')
		 -> [0] '<vehicle>'
	 - Node: <vehicle>		? (<model>:'E350')
		 -> [0] '<year>'
	 - Node: <year>		? (<model>:'E350')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind>'
	 - Node: <kind>		? (<model>:'E350')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company>'
	 - Node: <company>		? (<model>:'E350')
		 -> [0] 'Ford'
		 -> [5] ',E350'
		  > [',', '<model>']

```

```py
('<start>',
 [('<vehicle>',
   [('<year>', [['1997', []]]),
    (',', []),
    ('<kind>', [['van', []]]),
    (',', []),
    ('<company>', [['Ford', []]]),
    (',', []),
    ('<model>', [['E350', []]])])])

```

获得的推导树如下。

```py
display_tree(TreeMiner(tracer.my_input, assignments).tree) 
```

<svg width="403pt" height="173pt" viewBox="0.00 0.00 403.38 173.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 169)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="178.88" y="-151.7" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="178.88" y="-101.45" font-family="Times,serif" font-size="14.00"><vehicle></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="19.88" y="-51.2" font-family="Times,serif" font-size="14.00"><year></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="72.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="125.88" y="-51.2" font-family="Times,serif" font-size="14.00"><kind></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="178.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge7" class="edge"><title>1->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="244.88" y="-51.2" font-family="Times,serif" font-size="14.00"><company></text></g> <g id="edge8" class="edge"><title>1->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="310.88" y="-51.2" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge10" class="edge"><title>1->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="369.88" y="-51.2" font-family="Times,serif" font-size="14.00"><model></text></g> <g id="edge11" class="edge"><title>1->11</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="19.88" y="-0.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="125.88" y="-0.95" font-family="Times,serif" font-size="14.00">van</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="244.88" y="-0.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="369.88" y="-0.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge12" class="edge"><title>11->12</title></g></g></svg>

结合所有片段：

```py
trees = []
for vehicle in VEHICLES:
    print(vehicle)
    with Tracer(vehicle) as tracer:
        process_vehicle(tracer.my_input)
    assignments = DefineTracker(tracer.my_input, tracer.trace).assignments()
    trees.append((tracer.my_input, assignments))
    for var, val in assignments:
        print(var + " = " + repr(val))
    print() 
```

```py
1997,van,Ford,E350
vehicle = '1997,van,Ford,E350'
year = '1997'
kind = 'van'
company = 'Ford'
model = 'E350'

2000,car,Mercury,Cougar
vehicle = '2000,car,Mercury,Cougar'
year = '2000'
kind = 'car'
company = 'Mercury'
model = 'Cougar'

1999,car,Chevy,Venture
vehicle = '1999,car,Chevy,Venture'
year = '1999'
kind = 'car'
company = 'Chevy'
model = 'Venture'

```

对应的推导树如下。

```py
csv_dt = []
for inputstr, assignments in trees:
    print(inputstr)
    dt = TreeMiner(inputstr, assignments)
    csv_dt.append(dt)
    display_tree(dt.tree) 
```

```py
1997,van,Ford,E350
2000,car,Mercury,Cougar
1999,car,Chevy,Venture

```

### 从推导树恢复语法

我们定义了一个名为 `Miner` 的类，它可以组合多个推导树以生成语法。初始语法为空。

```py
class GrammarMiner:
    def __init__(self):
        self.grammar = {} 
```

`tree_to_grammar()` 方法通过一次选择一个节点，并将其添加到语法中，将我们的推导树转换为语法。节点名称成为键，它拥有的任何子列表都成为该键的另一个备选方案。

```py
class GrammarMiner(GrammarMiner):
    def tree_to_grammar(self, tree):
        node, children = tree
        one_alt = [ck for ck, gc in children]
        hsh = {node: [one_alt] if one_alt else []}
        for child in children:
            if not is_nonterminal(child[0]):
                continue
            chsh = self.tree_to_grammar(child)
            for k in chsh:
                if k not in hsh:
                    hsh[k] = chsh[k]
                else:
                    hsh[k].extend(chsh[k])
        return hsh 
```

```py
gm = GrammarMiner()
gm.tree_to_grammar(csv_dt[0].tree) 
```

```py
{'<start>': [['<vehicle>']],
 '<vehicle>': [['<year>', ',', '<kind>', ',', '<company>', ',', '<model>']],
 '<year>': [['1997']],
 '<kind>': [['van']],
 '<company>': [['Ford']],
 '<model>': [['E350']]}

```

这里生成的语法是 `规范化的`。我们定义了一个函数 `readable()`，它接受一个规范化的语法，并以可读的形式返回它。

```py
def readable(grammar):
    def readable_rule(rule):
        return ''.join(rule)

    return {k: list(set(readable_rule(a) for a in grammar[k]))
            for k in grammar} 
```

```py
syntax_diagram(readable(gm.tree_to_grammar(csv_dt[0].tree))) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 199.5 62" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">vehicle</text></g></g></g></g></svg>

```py
vehicle

```

<svg class="railroad-diagram" height="62" viewBox="0 0 575.5 62" width="575.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">year</text></g> <g class="terminal"><text x="148.25" y="35">,</text></g> <g class="non-terminal"><text x="209.5" y="35">kind</text></g> <g class="terminal"><text x="270.75" y="35">,</text></g> <g class="non-terminal"><text x="344.75" y="35">company</text></g> <g class="terminal"><text x="418.75" y="35">,</text></g> <g class="non-terminal"><text x="484.25" y="35">model</text></g></g></g></g></svg>

```py
year

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1997</text></g></g></g></g></svg>

```py
kind

```

<svg class="railroad-diagram" height="62" viewBox="0 0 165.5 62" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="82.75" y="35">van</text></g></g></g></g></svg>

```py
company

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">Ford</text></g></g></g></g></svg>

```py
model

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">E350</text></g></g></g></g></svg>

`add_tree()` 方法从当前语法中获取非终结符的组合列表，以及要添加到语法中的树，并更新每个非终结符的定义。

```py
import [itertools](https://docs.python.org/3/library/itertools.html) 
```

```py
class GrammarMiner(GrammarMiner):
    def add_tree(self, t):
        t_grammar = self.tree_to_grammar(t.tree)
        self.grammar = {
            key: self.grammar.get(key, []) + t_grammar.get(key, [])
            for key in itertools.chain(self.grammar.keys(), t_grammar.keys())
        } 
```

`add_tree()` 的使用如下：

```py
inventory_grammar_miner = GrammarMiner()
for dt in csv_dt:
    inventory_grammar_miner.add_tree(dt) 
```

```py
syntax_diagram(readable(inventory_grammar_miner.grammar)) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 199.5 62" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">vehicle</text></g></g></g></g></svg>

```py
vehicle

```

<svg class="railroad-diagram" height="62" viewBox="0 0 575.5 62" width="575.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="87.0" y="35">year</text></g> <g class="terminal"><text x="148.25" y="35">,</text></g> <g class="non-terminal"><text x="209.5" y="35">kind</text></g> <g class="terminal"><text x="270.75" y="35">,</text></g> <g class="non-terminal"><text x="344.75" y="35">company</text></g> <g class="terminal"><text x="418.75" y="35">,</text></g> <g class="non-terminal"><text x="484.25" y="35">model</text></g></g></g></g></svg>

```py
year

```

<svg class="railroad-diagram" height="122" viewBox="0 0 174.0 122" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1999</text></g></g> <g><g class="terminal"><text x="87.0" y="65">2000</text></g></g> <g><g class="terminal"><text x="87.0" y="95">1997</text></g></g></g></g></svg>

```py
kind

```

<svg class="railroad-diagram" height="92" viewBox="0 0 165.5 92" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="82.75" y="35">car</text></g></g> <g><g class="terminal"><text x="82.75" y="65">van</text></g></g></g></g></svg>

```py
company

```

<svg class="railroad-diagram" height="122" viewBox="0 0 199.5 122" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Mercury</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Chevy</text></g></g> <g><g class="terminal"><text x="99.75" y="95">Ford</text></g></g></g></g></svg>

```py
model

```

<svg class="railroad-diagram" height="122" viewBox="0 0 199.5 122" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">E350</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Cougar</text></g></g> <g><g class="terminal"><text x="99.75" y="95">Venture</text></g></g></g></g></svg>

给定来自各种输入的执行跟踪，可以定义 `update_grammar()` 以从跟踪中获取完整的语法。

```py
class GrammarMiner(GrammarMiner):
    def update_grammar(self, inputstr, trace):
        at = self.create_tracker(inputstr, trace)
        dt = self.create_tree_miner(inputstr, at.assignments())
        self.add_tree(dt)
        return self.grammar

    def create_tracker(self, *args):
        return DefineTracker(*args)

    def create_tree_miner(self, *args):
        return TreeMiner(*args) 
```

完整的语法恢复在 `recover_grammar()` 中实现。

```py
def recover_grammar(fn: Callable, inputs: Iterable[str], 
                    **kwargs: Any) -> Grammar:
    miner = GrammarMiner()

    for inputstr in inputs:
        with Tracer(inputstr, **kwargs) as tracer:
            fn(tracer.my_input)
        miner.update_grammar(tracer.my_input, tracer.trace)

    return readable(miner.grammar) 
```

注意，语法可以直接从跟踪器中检索，而不需要中间的推导树阶段。然而，通过推导树可以检查正在分段的输入，并验证它是否正确发生。

#### 示例 1. 恢复库存语法

```py
inventory_grammar = recover_grammar(process_vehicle, VEHICLES) 
```

```py
inventory_grammar 
```

```py
{'<start>': ['<vehicle>'],
 '<vehicle>': ['<year>,<kind>,<company>,<model>'],
 '<year>': ['1999', '2000', '1997'],
 '<kind>': ['car', 'van'],
 '<company>': ['Mercury', 'Chevy', 'Ford'],
 '<model>': ['E350', 'Cougar', 'Venture']}

```

#### 示例 2. 恢复 URL 语法

我们的算法足够鲁棒，可以从现实世界的程序中恢复语法。例如，Python `urlib` 模块中的 `urlparse` 函数接受以下示例 URL。

```py
URLS = [
    'http://user:pass@www.google.com:80/?q=path#ref',
    'https://www.cispa.saarland:80/',
    'http://www.fuzzingbook.org/#News',
] 
```

`urllib` 缓存其中间结果以实现快速访问。因此，我们需要在每次调用后使用 `clear_cache()` 禁用它。

```py
from [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) import urlparse, clear_cache 
```

我们使用样本 URL 如下恢复语法。`urlparse` 函数倾向于缓存其之前的解析结果。因此，我们定义了一个新的方法 `url_parse()`，在每次调用之前清除缓存。

```py
def url_parse(url):
    clear_cache()
    urlparse(url) 
```

```py
trees = []
for url in URLS:
    print(url)
    with Tracer(url) as tracer:
        url_parse(tracer.my_input)
    assignments = DefineTracker(tracer.my_input, tracer.trace).assignments()
    trees.append((tracer.my_input, assignments))
    for var, val in assignments:
        print(var + " = " + repr(val))
    print()

url_dt = []
for inputstr, assignments in trees:
    print(inputstr)
    dt = TreeMiner(inputstr, assignments)
    url_dt.append(dt)
    display_tree(dt.tree) 
```

```py
http://user:pass@www.google.com:80/?q=path#ref
url = 'http://user:pass@www.google.com:80/?q=path#ref'
scheme = 'http'
netloc = 'user:pass@www.google.com:80'
fragment = 'ref'
query = 'q=path'

https://www.cispa.saarland:80/
url = 'https://www.cispa.saarland:80/'
scheme = 'https'
netloc = 'www.cispa.saarland:80'

http://www.fuzzingbook.org/#News
url = 'http://www.fuzzingbook.org/#News'
scheme = 'http'
netloc = 'www.fuzzingbook.org'
fragment = 'News'

http://user:pass@www.google.com:80/?q=path#ref
https://www.cispa.saarland:80/
http://www.fuzzingbook.org/#News

```

让我们使用 `url_parse()` 来恢复语法：

```py
url_grammar = recover_grammar(url_parse, URLS, files=['urllib/parse.py']) 
```

```py
syntax_diagram(url_grammar) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 165.5 62" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="82.75" y="35">url</text></g></g></g></g></svg>

```py
url

```

<svg class="railroad-diagram" height="122" viewBox="0 0 643.5 122" width="643.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="95.5" y="35">scheme</text></g> <g class="terminal"><text x="173.75" y="35">://</text></g> <g class="non-terminal"><text x="252.0" y="35">netloc</text></g> <g class="terminal"><text x="326.0" y="35">/?</text></g> <g class="non-terminal"><text x="395.75" y="35">query</text></g> <g class="terminal"><text x="461.25" y="35">#</text></g> <g class="non-terminal"><text x="539.5" y="35">fragment</text></g></g> <g><g class="non-terminal"><text x="219.25" y="65">scheme</text></g> <g class="terminal"><text x="297.5" y="65">://</text></g> <g class="non-terminal"><text x="375.75" y="65">netloc</text></g> <g class="terminal"><text x="445.5" y="65">/</text></g></g> <g><g class="non-terminal"><text x="161.0" y="95">scheme</text></g> <g class="terminal"><text x="239.25" y="95">://</text></g> <g class="non-terminal"><text x="317.5" y="95">netloc</text></g> <g class="terminal"><text x="391.5" y="95">/#</text></g> <g class="non-terminal"><text x="474.0" y="95">fragment</text></g></g></g></g></svg>

```py
scheme

```

<svg class="railroad-diagram" height="92" viewBox="0 0 182.5 92" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="91.25" y="35">http</text></g></g> <g><g class="terminal"><text x="91.25" y="65">https</text></g></g></g></g></svg>

```py
netloc

```

<svg class="railroad-diagram" height="122" viewBox="0 0 369.5 122" width="369.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="184.75" y="35">www.cispa.saarland:80</text></g></g> <g><g class="terminal"><text x="184.75" y="65">www.fuzzingbook.org</text></g></g> <g><g class="terminal"><text x="184.75" y="95">user:pass@www.google.com:80</text></g></g></g></g></svg>

```py
query

```

<svg class="railroad-diagram" height="62" viewBox="0 0 191.0 62" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="95.5" y="35">q=path</text></g></g></g></g></svg>

```py
fragment

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">News</text></g></g> <g><g class="terminal"><text x="87.0" y="65">ref</text></g></g></g></g></svg>

恢复的语法对 URL 格式的描述相当合理。

### 模糊测试

我们现在可以使用恢复的语法进行如下模糊测试。

首先，库存语法。

```py
f = GrammarFuzzer(inventory_grammar)
for _ in range(10):
    print(f.fuzz()) 
```

```py
1997,car,Mercury,E350
2000,car,Chevy,Cougar
1997,van,Mercury,Venture
1999,car,Ford,Venture
2000,car,Mercury,E350
2000,car,Mercury,Cougar
1997,car,Chevy,E350
1997,car,Chevy,E350
1997,car,Mercury,Cougar
1999,car,Chevy,E350

```

接下来，是 URL 语法。

```py
f = GrammarFuzzer(url_grammar)
for _ in range(10):
    print(f.fuzz()) 
```

```py
https://user:pass@www.google.com:80/
http://www.cispa.saarland:80/?q=path#News
https://user:pass@www.google.com:80/
https://user:pass@www.google.com:80/#ref
http://user:pass@www.google.com:80/
http://user:pass@www.google.com:80/#ref
http://user:pass@www.google.com:80/?q=path#News
http://www.fuzzingbook.org/?q=path#News
http://www.fuzzingbook.org/?q=path#ref
http://www.cispa.saarland:80/

```

这意味着我们现在可以取一个程序和一些样本，提取其语法，然后使用这个语法进行模糊测试。这真是个好机会！

### 简单挖掘器的问题

我们简单语法挖掘器的问题之一是假设变量分配的值是稳定的。不幸的是，这可能在所有情况下都不成立。例如，这里有一个格式略有不同的 URL。

```py
URLS_X = URLS + ['ftp://freebsd.org/releases/5.8'] 
```

从这组样本生成的语法不如我们之前得到的那么好

```py
url_grammar = recover_grammar(url_parse, URLS_X, files=['urllib/parse.py']) 
```

```py
syntax_diagram(url_grammar) 
```

```py
start

```

<svg class="railroad-diagram" height="92" viewBox="0 0 413.0 92" width="413.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="206.5" y="35">url</text></g></g> <g><g class="non-terminal"><text x="95.5" y="65">scheme</text></g> <g class="terminal"><text x="173.75" y="65">://</text></g> <g class="non-terminal"><text x="252.0" y="65">netloc</text></g> <g class="non-terminal"><text x="330.25" y="65">url</text></g></g></g></g></svg>

```py
url

```

<svg class="railroad-diagram" height="152" viewBox="0 0 643.5 152" width="643.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="95.5" y="65">scheme</text></g> <g class="terminal"><text x="173.75" y="65">://</text></g> <g class="non-terminal"><text x="252.0" y="65">netloc</text></g> <g class="terminal"><text x="326.0" y="65">/?</text></g> <g class="non-terminal"><text x="395.75" y="65">query</text></g> <g class="terminal"><text x="461.25" y="65">#</text></g> <g class="non-terminal"><text x="539.5" y="65">fragment</text></g></g> <g><g class="terminal"><text x="321.75" y="35">/releases/5.8</text></g></g> <g><g class="non-terminal"><text x="219.25" y="95">scheme</text></g> <g class="terminal"><text x="297.5" y="95">://</text></g> <g class="non-terminal"><text x="375.75" y="95">netloc</text></g> <g class="terminal"><text x="445.5" y="95">/</text></g></g> <g><g class="non-terminal"><text x="161.0" y="125">scheme</text></g> <g class="terminal"><text x="239.25" y="125">://</text></g> <g class="non-terminal"><text x="317.5" y="125">netloc</text></g> <g class="terminal"><text x="391.5" y="125">/#</text></g> <g class="non-terminal"><text x="474.0" y="125">fragment</text></g></g></g></g></svg>

```py
scheme

```

<svg class="railroad-diagram" height="122" viewBox="0 0 182.5 122" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="91.25" y="35">http</text></g></g> <g><g class="terminal"><text x="91.25" y="65">ftp</text></g></g> <g><g class="terminal"><text x="91.25" y="95">https</text></g></g></g></g></svg>

```py
netloc

```

<svg class="railroad-diagram" height="152" viewBox="0 0 369.5 152" width="369.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="184.75" y="65">www.fuzzingbook.org</text></g></g> <g><g class="terminal"><text x="184.75" y="35">www.cispa.saarland:80</text></g></g> <g><g class="terminal"><text x="184.75" y="95">freebsd.org</text></g></g> <g><g class="terminal"><text x="184.75" y="125">user:pass@www.google.com:80</text></g></g></g></g></svg>

```py
query

```

<svg class="railroad-diagram" height="62" viewBox="0 0 191.0 62" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="95.5" y="35">q=path</text></g></g></g></g></svg>

```py
fragment

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">News</text></g></g> <g><g class="terminal"><text x="87.0" y="65">ref</text></g></g></g></g></svg>

显然，出了些问题。

为了调查 `url` 定义出错的原因，让我们检查 URL 的跟踪。

```py
clear_cache()
with Tracer(URLS_X[0]) as tracer:
    urlparse(tracer.my_input)
for i, t in enumerate(tracer.trace):
    if t[0] in {'call', 'line'} and 'parse.py' in str(t[2]) and t[3]:
        print(i, t[2]._t()[1], t[3:]) 
```

```py
0 374 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
1 394 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
5 129 ({'arg': ''},)
6 126 ({'arg': ''},)
7 131 ({'arg': ''},)
8 132 ({'arg': ''},)
10 395 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
11 452 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
12 474 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
16 129 ({'arg': ''},)
17 126 ({'arg': ''},)
18 131 ({'arg': ''},)
19 132 ({'arg': ''},)
21 477 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
22 478 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
23 480 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
24 481 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\t'},)
25 482 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\t'},)
26 480 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\t'},)
27 481 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\r'},)
28 482 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\r'},)
29 480 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\r'},)
30 481 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n'},)
31 482 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n'},)
32 480 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n'},)
33 484 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n'},)
34 485 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n'},)
35 486 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': ''},)
36 487 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': ''},)
37 488 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': ''},)
38 489 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 'h'},)
39 488 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 'h'},)
40 489 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 't'},)
41 488 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 't'},)
42 489 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 't'},)
43 488 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 't'},)
44 489 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 'p'},)
45 488 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 'p'},)
46 492 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': '', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 'p'},)
47 493 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'scheme': 'http', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 'p'},)
48 494 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'scheme': 'http', 'b': '\n', 'netloc': '', 'query': '', 'fragment': '', 'c': 'p'},)
49 413 ({'url': '//user:pass@www.google.com:80/?q=path#ref'},)
50 414 ({'url': '//user:pass@www.google.com:80/?q=path#ref'},)
51 415 ({'url': '//user:pass@www.google.com:80/?q=path#ref'},)
52 416 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '/'},)
53 417 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '/'},)
54 418 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '/'},)
55 415 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '/'},)
56 416 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '?'},)
57 417 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '?'},)
58 418 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '?'},)
59 415 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '?'},)
60 416 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '#'},)
61 417 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '#'},)
62 418 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '#'},)
63 415 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '#'},)
64 419 ({'url': '//user:pass@www.google.com:80/?q=path#ref', 'c': '#'},)
66 495 ({'url': '/?q=path#ref', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': '', 'fragment': '', 'c': 'p'},)
67 496 ({'url': '/?q=path#ref', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': '', 'fragment': '', 'c': 'p'},)
68 498 ({'url': '/?q=path#ref', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': '', 'fragment': '', 'c': 'p'},)
69 501 ({'url': '/?q=path#ref', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': '', 'fragment': '', 'c': 'p'},)
70 502 ({'url': '/?q=path#ref', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': '', 'fragment': '', 'c': 'p'},)
71 503 ({'url': '/?q=path', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': '', 'fragment': 'ref', 'c': 'p'},)
72 504 ({'url': '/?q=path', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': '', 'fragment': 'ref', 'c': 'p'},)
73 505 ({'url': '/', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': 'q=path', 'fragment': 'ref', 'c': 'p'},)
74 421 ({'netloc': 'user:pass@www.google.com:80'},)
75 422 ({'netloc': 'user:pass@www.google.com:80'},)
76 423 ({'netloc': 'user:pass@www.google.com:80'},)
78 506 ({'url': '/', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': 'q=path', 'fragment': 'ref', 'c': 'p'},)
82 507 ({'url': '/', 'scheme': 'http', 'b': '\n', 'netloc': 'user:pass@www.google.com:80', 'query': 'q=path', 'fragment': 'ref', 'c': 'p'},)
87 396 ({'url': 'http://user:pass@www.google.com:80/?q=path#ref', 'scheme': ''},)
88 397 ({'url': '/', 'scheme': 'http', 'netloc': 'user:pass@www.google.com:80', 'query': 'q=path', 'fragment': 'ref'},)
89 400 ({'url': '/', 'scheme': 'http', 'netloc': 'user:pass@www.google.com:80', 'query': 'q=path', 'fragment': 'ref'},)
90 401 ({'url': '/', 'scheme': 'http', 'netloc': 'user:pass@www.google.com:80', 'query': 'q=path', 'fragment': 'ref', 'params': ''},)
94 402 ({'url': '/', 'scheme': 'http', 'netloc': 'user:pass@www.google.com:80', 'query': 'q=path', 'fragment': 'ref', 'params': ''},)

```

注意，随着解析的进行，`url` 的值是如何变化的？这违反了我们对变量赋值值稳定的假设。接下来，我们将探讨如何消除这种限制。

## 带有重新分配的语法挖掘器

唯一识别不同变量的方法之一是在它们定义和值变化时都使用 *行号* 进行注释。考虑下面的代码片段

### 跟踪变量赋值位置

```py
def C(cp_1):
    c_2 = cp_1 + '@2'
    c_3 = c_2 + '@3'
    return c_3 
```

```py
def B(bp_7):
    b_8 = bp_7 + '@8'
    return C(b_8) 
```

```py
def A(ap_12):
    a_13 = ap_12 + '@13'
    a_14 = B(a_13) + '@14'
    a_14 = a_14 + '@15'
    a_13 = a_14 + '@16'
    a_14 = B(a_13) + '@17'
    a_14 = B(a_13) + '@18' 
```

注意，所有变量要么命名与它们定义的位置相对应，要么值被注释以表明它已被更改。

让我们运行这个带有跟踪的代码。

```py
with Tracer('____') as tracer:
    A(tracer.my_input)

for t in tracer.trace:
    print(t[0], "%d:%s" % (t[2].line_no, t[2].method), t[3]) 
```

```py
call 1:A {'ap_12': '____'}
line 2:A {'ap_12': '____'}
line 3:A {'ap_12': '____', 'a_13': '____@13'}
call 1:B {'bp_7': '____@13'}
line 2:B {'bp_7': '____@13'}
line 3:B {'bp_7': '____@13', 'b_8': '____@13@8'}
call 1:C {'cp_1': '____@13@8'}
line 2:C {'cp_1': '____@13@8'}
line 3:C {'cp_1': '____@13@8', 'c_2': '____@13@8@2'}
line 4:C {'cp_1': '____@13@8', 'c_2': '____@13@8@2', 'c_3': '____@13@8@2@3'}
return 4:C {'cp_1': '____@13@8', 'c_2': '____@13@8@2', 'c_3': '____@13@8@2@3'}
return 3:B {'bp_7': '____@13', 'b_8': '____@13@8'}
line 4:A {'ap_12': '____', 'a_13': '____@13', 'a_14': '____@13@8@2@3@14'}
line 5:A {'ap_12': '____', 'a_13': '____@13', 'a_14': '____@13@8@2@3@14@15'}
line 6:A {'ap_12': '____', 'a_13': '____@13@8@2@3@14@15@16', 'a_14': '____@13@8@2@3@14@15'}
call 1:B {'bp_7': '____@13@8@2@3@14@15@16'}
line 2:B {'bp_7': '____@13@8@2@3@14@15@16'}
line 3:B {'bp_7': '____@13@8@2@3@14@15@16', 'b_8': '____@13@8@2@3@14@15@16@8'}
call 1:C {'cp_1': '____@13@8@2@3@14@15@16@8'}
line 2:C {'cp_1': '____@13@8@2@3@14@15@16@8'}
line 3:C {'cp_1': '____@13@8@2@3@14@15@16@8', 'c_2': '____@13@8@2@3@14@15@16@8@2'}
line 4:C {'cp_1': '____@13@8@2@3@14@15@16@8', 'c_2': '____@13@8@2@3@14@15@16@8@2', 'c_3': '____@13@8@2@3@14@15@16@8@2@3'}
return 4:C {'cp_1': '____@13@8@2@3@14@15@16@8', 'c_2': '____@13@8@2@3@14@15@16@8@2', 'c_3': '____@13@8@2@3@14@15@16@8@2@3'}
return 3:B {'bp_7': '____@13@8@2@3@14@15@16', 'b_8': '____@13@8@2@3@14@15@16@8'}
line 7:A {'ap_12': '____', 'a_13': '____@13@8@2@3@14@15@16', 'a_14': '____@13@8@2@3@14@15@16@8@2@3@17'}
call 1:B {'bp_7': '____@13@8@2@3@14@15@16'}
line 2:B {'bp_7': '____@13@8@2@3@14@15@16'}
line 3:B {'bp_7': '____@13@8@2@3@14@15@16', 'b_8': '____@13@8@2@3@14@15@16@8'}
call 1:C {'cp_1': '____@13@8@2@3@14@15@16@8'}
line 2:C {'cp_1': '____@13@8@2@3@14@15@16@8'}
line 3:C {'cp_1': '____@13@8@2@3@14@15@16@8', 'c_2': '____@13@8@2@3@14@15@16@8@2'}
line 4:C {'cp_1': '____@13@8@2@3@14@15@16@8', 'c_2': '____@13@8@2@3@14@15@16@8@2', 'c_3': '____@13@8@2@3@14@15@16@8@2@3'}
return 4:C {'cp_1': '____@13@8@2@3@14@15@16@8', 'c_2': '____@13@8@2@3@14@15@16@8@2', 'c_3': '____@13@8@2@3@14@15@16@8@2@3'}
return 3:B {'bp_7': '____@13@8@2@3@14@15@16', 'b_8': '____@13@8@2@3@14@15@16@8'}
return 7:A {'ap_12': '____', 'a_13': '____@13@8@2@3@14@15@16', 'a_14': '____@13@8@2@3@14@15@16@8@2@3@18'}
call 102:__exit__ {}
line 105:__exit__ {}

```

每个变量首先被引用如下：

+   `cp_1` -- *调用* `1:C`

+   `c_2` -- *行* `3:C`（但上一个事件是*行* `2:C`）

+   `c_3` -- *行* `4:C`（但上一个事件是*行* `3:C`）

+   `bp_7` -- *调用* `7:B`

+   `b_8` -- *行* `9:B`（但上一个事件是*行* `8:B`）

+   `ap_12` -- *调用* `12:A`

+   `a_13` -- *行* `14:A`（但上一个事件是*行* `13:A`）

+   `a_14` -- *行* `15:A`（上一个事件是*返回* `9:B`。然而，`A()`中的上一个事件是*行* `14:A`）

+   在第 15 行重新分配`a_14` -- *行* `16:A`（上一个事件是*行* `15:A`）

+   在第 16 行重新分配`a_13` -- *行* `17:A`（上一个事件是*行* `16:A`）

+   在第 17 行重新分配`a_14` -- *返回* `17:A`（`A()`中的上一个事件是*行* `17:A`）

+   在第 18 行重新分配`a_14` -- *返回* `18:A`（`A()`中的上一个事件是*行* `18:A`）

因此，我们的观察结果是，如果是一个调用，当前位置是定义任何新变量的正确位置。另一方面，如果首次引用变量（或重新分配了新值），那么考虑的正确位置是*同一方法调用中的*上一个位置。接下来，让我们看看我们如何将此信息纳入变量命名。

接下来，我们需要一种方法来跟踪正在进行的单个方法调用。为此，我们定义了类`CallStack`。每次方法调用都会获得一个单独的标识符，当方法调用结束时，标识符会被重置。

### CallStack

```py
class CallStack:
    def __init__(self, **kwargs):
        self.options(kwargs)
        self.method_id = (START_SYMBOL, 0)
        self.method_register = 0
        self.mstack = [self.method_id]

    def enter(self, method):
        self.method_register += 1
        self.method_id = (method, self.method_register)
        self.log('call', "%s%s" % (self.indent(), str(self)))
        self.mstack.append(self.method_id)

    def leave(self):
        self.mstack.pop()
        self.log('return', "%s%s" % (self.indent(), str(self)))
        self.method_id = self.mstack[-1] 
```

几个额外的函数以简化生活。

```py
class CallStack(CallStack):
    def options(self, kwargs):
        self.log = log_event if kwargs.get('log') else lambda _evt, _var: None

    def indent(self):
        return len(self.mstack) * "\t"

    def at(self, n):
        return self.mstack[n]

    def __len__(self):
        return len(mstack) - 1

    def __str__(self):
        return "%s:%d" % self.method_id

    def __repr__(self):
        return repr(self.method_id) 
```

我们还定义了一个方便的方法来显示给定的堆栈。

```py
def display_stack(istack):
    def stack_to_tree(stack):
        current, *rest = stack
        if not rest:
            return (repr(current), [])
        return (repr(current), [stack_to_tree(rest)])
    display_tree(stack_to_tree(istack.mstack), graph_attr=lr_graph) 
```

这是我们可以使用`CallStack`的方法。

```py
cs = CallStack()
display_stack(cs)
cs 
```

```py
('<start>', 0)

```

```py
cs.enter('hello')
display_stack(cs)
cs 
```

```py
('hello', 1)

```

```py
cs.enter('world')
display_stack(cs)
cs 
```

```py
('world', 2)

```

```py
cs.leave()
display_stack(cs)
cs 
```

```py
('hello', 1)

```

```py
cs.enter('world')
display_stack(cs)
cs 
```

```py
('world', 3)

```

```py
cs.leave()
display_stack(cs)
cs 
```

```py
('hello', 1)

```

为了处理变量重新分配，我们需要一个比字典更智能的数据结构来存储变量。我们首先定义了一个简单的接口`Vars`。它作为变量的容器，并在`my_assignments`处实例化。

### Vars

`Vars`在其内部字典`defs`中存储变量引用，该字典在解析过程中出现。我们用原始字符串初始化字典。

```py
class Vars:
    def __init__(self, original):
        self.defs = {}
        self.my_input = original 
```

字典需要两个方法：`update()`，它接受一组键值对以更新自身，以及`_set_kv()`，它更新特定的键值对。

```py
class Vars(Vars):
    def _set_kv(self, k, v):
        self.defs[k] = v

    def __setitem__(self, k, v):
        self._set_kv(k, v)

    def update(self, v):
        for k, v in v.items():
            self._set_kv(k, v) 
```

`Vars`是内部字典的代理。例如，以下是使用它的方法。

```py
v = Vars('')
v.defs 
```

```py
{}

```

```py
v['x'] = 'X'
v.defs 
```

```py
{'x': 'X'}

```

```py
v.update({'x': 'x', 'y': 'y'})
v.defs 
```

```py
{'x': 'x', 'y': 'y'}

```

### AssignmentVars

我们现在扩展简单的`Vars`以处理变量重新分配。为此，我们定义了`AssignmentVars`。

检测重新分配和重命名变量的想法如下：我们使用`accessed_seq_var`跟踪特定变量的上一个重新分配。它包含任何特定变量的最后重命名作为其对应值。`new_vars`包含在本迭代中添加的所有新变量的列表。

```py
class AssignmentVars(Vars):
    def __init__(self, original):
        super().__init__(original)
        self.accessed_seq_var = {}
        self.var_def_lines = {}
        self.current_event = None
        self.new_vars = set()
        self.method_init() 
```

`method_init()`方法负责使用`call_stack`中保存的记录来跟踪方法调用。`event_locations`用于跟踪此方法内访问的位置。这是用于变量定义行号跟踪的。

```py
class AssignmentVars(AssignmentVars):
    def method_init(self):
        self.call_stack = CallStack()
        self.event_locations = {self.call_stack.method_id: []} 
```

`update()`现在被修改为使用`var_location_register()`跟踪任何更改的行号。我们使用后重新初始化`new_vars`以供下一个事件使用。

```py
class AssignmentVars(AssignmentVars):
    def update(self, v):
        for k, v in v.items():
            self._set_kv(k, v)
        self.var_location_register(self.new_vars)
        self.new_vars = set() 
```

变量名现在包含一个索引，表示它经历了多少次重新赋值，从而使得每次重新赋值都是一个唯一的变量。

```py
class AssignmentVars(AssignmentVars):
    def var_name(self, var):
        return (var, self.accessed_seq_var[var]) 
```

在存储变量时，我们首先需要检查它之前是否已知。如果它不是，我们需要初始化重命名计数。这是通过`var_access`完成的。

```py
class AssignmentVars(AssignmentVars):
    def var_access(self, var):
        if var not in self.accessed_seq_var:
            self.accessed_seq_var[var] = 0
        return self.var_name(var) 
```

在变量重新赋值期间，我们更新`accessed_seq_var`以反映新的计数。

```py
class AssignmentVars(AssignmentVars):
    def var_assign(self, var):
        self.accessed_seq_var[var] += 1
        self.new_vars.add(self.var_name(var))
        return self.var_name(var) 
```

这些方法可以这样使用

```py
sav = AssignmentVars('')
sav.defs 
```

```py
{}

```

```py
sav.var_access('v1') 
```

```py
('v1', 0)

```

```py
sav.var_assign('v1') 
```

```py
('v1', 1)

```

再次赋值给它会增加计数器。

```py
sav.var_assign('v1') 
```

```py
('v1', 2)

```

逻辑的核心在于`_set_kv()`。当一个变量被赋值时，我们获取序列化变量名`s_var`。如果序列化变量名在`defs`中之前是未知的，那么我们就没有进一步的担忧。我们将序列化变量添加到`defs`中。

如果变量之前已知，那么这是一个可能重新赋值的指示。在这种情况下，我们查看变量所持有的值。我们检查值是否已更改。如果没有，那么它就不是。

如果值已经改变，它就是一个重新赋值。我们首先使用`var_assign`增加变量使用序列，检索新名称，并在`defs`中更新新名称。

```py
class AssignmentVars(AssignmentVars):
    def _set_kv(self, var, val):
        s_var = self.var_access(var)
        if s_var in self.defs and self.defs[s_var] == val:
            return
        self.defs[self.var_assign(var)] = val 
```

这里是如何使用它的。第一次将变量赋值初始化其计数器。

```py
sav = AssignmentVars('')
sav['x'] = 'X'
sav.defs 
```

```py
{('x', 1): 'X'}

```

如果变量再次以相同的值赋值，那么它可能不是重新赋值。

```py
sav['x'] = 'X'
sav.defs 
```

```py
{('x', 1): 'X'}

```

然而，如果值发生了变化，它就是一个重新赋值。

```py
sav['x'] = 'Y'
sav.defs 
```

```py
{('x', 1): 'X', ('x', 2): 'Y'}

```

这里有一个微妙之处。子方法可以从父方法的中部被调用，并且两者可以使用不同的值具有相同的变量名。在这种情况下，当子方法返回时，父方法将具有上下文中的旧变量和旧值。在我们的实现中，我们将其视为重新赋值。然而，这是可以的，因为添加新的重新赋值是无害的，但遗漏一个则不行。此外，我们稍后还会讨论如何避免这种情况。

我们还定义了`register_event()`、`method_enter()`和`method_exit()`的记账代码，这些是负责跟踪方法栈的方法。基本思想是，每个`method_enter()`代表一个新的方法调用。因此，它值得一个新的方法 ID，该 ID 由`method_register`生成，并保存在`method_id`中。由于这是一个新的方法，方法栈通过一个具有此 ID 的元素进行扩展。在`method_exit()`的情况下，我们弹出方法栈，并将当前`method_id`重置为当前一个以下的值。

```py
class AssignmentVars(AssignmentVars):
    def method_enter(self, cxt, my_vars):
        self.current_event = 'call'
        self.call_stack.enter(cxt.method)
        self.event_locations[self.call_stack.method_id] = []
        self.register_event(cxt)
        self.update(my_vars)

    def method_exit(self, cxt, my_vars):
        self.current_event = 'return'
        self.register_event(cxt)
        self.update(my_vars)
        self.call_stack.leave()

    def method_statement(self, cxt, my_vars):
        self.current_event = 'line'
        self.register_event(cxt)
        self.update(my_vars) 
```

对于每个方法事件，我们还会使用 `register_event()` 注册事件，该事件跟踪 *此* 方法中引用的行号。

```py
class AssignmentVars(AssignmentVars):
    def register_event(self, cxt):
        self.event_locations[self.call_stack.method_id].append(cxt.line_no) 
```

`var_location_register()` 保存新添加变量的位置。在 `call` 中的变量定义位置是 *当前* 位置。然而，对于 `line`，它将是当前方法中的前一个事件。

```py
class AssignmentVars(AssignmentVars):
    def var_location_register(self, my_vars):
        def loc(mid):
            if self.current_event == 'call':
                return self.event_locations[mid][-1]
            elif self.current_event == 'line':
                return self.event_locations[mid][-2]
            elif self.current_event == 'return':
                return self.event_locations[mid][-2]
            else:
                assert False

        my_loc = loc(self.call_stack.method_id)
        for var in my_vars:
            self.var_def_lines[var] = my_loc 
```

我们定义 `defined_vars()`，它返回以下标注行号的变量的名称。

```py
class AssignmentVars(AssignmentVars):
    def defined_vars(self, formatted=True):
        def fmt(k):
            v = (k[0], self.var_def_lines[k])
            return "%s@%s" % v if formatted else v

        return [(fmt(k), v) for k, v in self.defs.items()] 
```

与 `defined_vars()` 类似，我们定义了 `seq_vars()`，它用使用次数标注不同的变量。

```py
class AssignmentVars(AssignmentVars):
    def seq_vars(self, formatted=True):
        def fmt(k):
            v = (k[0], self.var_def_lines[k], k[1])
            return "%s@%s:%s" % v if formatted else v

        return {fmt(k): v for k, v in self.defs.items()} 
```

### AssignmentTracker

`AssignmentTracker` 使用我们之前定义的 `AssignmentVars` 保存赋值定义。

```py
class AssignmentTracker(DefineTracker):
    def __init__(self, my_input, trace, **kwargs):
        self.options(kwargs)
        self.my_input = my_input

        self.my_assignments = self.create_assignments(my_input)

        self.trace = trace
        self.process()

    def create_assignments(self, *args):
        return AssignmentVars(*args) 
```

为了微调过程，我们定义了一个可选参数，称为 `track_return`。在跟踪方法返回时，Python 会产生一个包含返回值结果的虚拟变量。如果设置 `track_return`，我们将捕获此值作为变量。

+   `track_return` -- 如果为真，则向 Vars 添加一个 *虚拟变量* 来表示返回值

```py
class AssignmentTracker(AssignmentTracker):
    def options(self, kwargs):
        self.track_return = kwargs.get('track_return', False)
        super().options(kwargs) 
```

在跟踪过程中，可能会有不同类型的事件，包括 `call`（当函数进入时），`return`（当函数返回时），`exception`（当抛出异常时）和 `line`（当执行语句时）。

之前的 `Tracker` 过于简单，因为它没有区分不同的事件。我们纠正了这一点，并分别定义了 `on_call()`、`on_return()` 和 `on_line()`，它们将在对应的事件上被调用。

注意，`on_line()` 也会被调用于 `on_return()`。原因是，Python 在执行相应的行之前会调用跟踪函数。因此，实际上，`on_return()` 是在环境执行前一个语句产生的绑定下被调用的。我们的处理实际上是在前一个语句绑定的值上进行的。因此，在这里调用 `on_line()` 是合适的，因为它给事件处理程序提供了一个机会来处理前一个绑定。

```py
class AssignmentTracker(AssignmentTracker):
    def on_call(self, arg, cxt, my_vars):
        my_vars = cxt.parameters(my_vars)
        self.my_assignments.method_enter(cxt, self.fragments(my_vars))

    def on_line(self, arg, cxt, my_vars):
        self.my_assignments.method_statement(cxt, self.fragments(my_vars))

    def on_return(self, arg, cxt, my_vars):
        self.on_line(arg, cxt, my_vars)
        my_vars = {'<-%s' % cxt.method: arg} if self.track_return else {}
        self.my_assignments.method_exit(cxt, my_vars)

    def on_exception(self, arg, cxt, my_vara):
        return

    def track_event(self, event, arg, cxt, my_vars):
        self.current_event = event
        dispatch = {
            'call': self.on_call,
            'return': self.on_return,
            'line': self.on_line,
            'exception': self.on_exception
        }
        dispatchevent 
```

我们现在可以使用 `AssignmentTracker` 跟踪不同的变量。为了验证我们的变量行号推断是否有效，我们从函数 `A()`、`B()` 和 `C()`（移除了数据注释，以便正确识别输入片段）中恢复定义。

```py
def C(cp_1):
    c_2 = cp_1
    c_3 = c_2
    return c_3 
```

```py
def B(bp_7):
    b_8 = bp_7
    return C(b_8) 
```

```py
def A(ap_12):
    a_13 = ap_12
    a_14 = B(a_13)
    a_14 = a_14
    a_13 = a_14
    a_14 = B(a_13)
    a_14 = B(a_14)[3:] 
```

用足够的输入运行 `A()`。

```py
with Tracer('---xxx') as tracer:
    A(tracer.my_input)
tracker = AssignmentTracker(tracer.my_input, tracer.trace, log=True)
for k, v in tracker.my_assignments.seq_vars().items():
    print(k, '=', repr(v))
print()
for k, v in tracker.my_assignments.defined_vars(formatted=True):
    print(k, '=', repr(v)) 
```

```py
ap_12@1:1 = '---xxx'
a_13@2:1 = '---xxx'
bp_7@1:1 = '---xxx'
b_8@2:1 = '---xxx'
cp_1@1:1 = '---xxx'
c_2@2:1 = '---xxx'
c_3@3:1 = '---xxx'
a_14@3:1 = '---xxx'
a_14@7:2 = 'xxx'

ap_12@1 = '---xxx'
a_13@2 = '---xxx'
bp_7@1 = '---xxx'
b_8@2 = '---xxx'
cp_1@1 = '---xxx'
c_2@2 = '---xxx'
c_3@3 = '---xxx'
a_14@3 = '---xxx'
a_14@7 = 'xxx'

```

如所示，现在每个变量的行号都已正确识别。

让我们尝试检索一个真实世界示例的赋值。

```py
traces = []
for inputstr in URLS_X:
    clear_cache()
    with Tracer(inputstr, files=['urllib/parse.py']) as tracer:
        urlparse(tracer.my_input)
    traces.append((tracer.my_input, tracer.trace))

    tracker = AssignmentTracker(tracer.my_input, tracer.trace, log=True)
    for k, v in tracker.my_assignments.defined_vars():
        print(k, '=', repr(v))
    print() 
```

```py
url@374 = 'http://user:pass@www.google.com:80/?q=path#ref'
url@492 = '//user:pass@www.google.com:80/?q=path#ref'
scheme@492 = 'http'
url@494 = '/?q=path#ref'
netloc@494 = 'user:pass@www.google.com:80'
url@502 = '/?q=path'
fragment@502 = 'ref'
query@504 = 'q=path'
url@395 = 'http://user:pass@www.google.com:80/?q=path#ref'

url@374 = 'https://www.cispa.saarland:80/'
url@492 = '//www.cispa.saarland:80/'
scheme@492 = 'https'
netloc@494 = 'www.cispa.saarland:80'
url@395 = 'https://www.cispa.saarland:80/'

url@374 = 'http://www.fuzzingbook.org/#News'
url@492 = '//www.fuzzingbook.org/#News'
scheme@492 = 'http'
url@494 = '/#News'
netloc@494 = 'www.fuzzingbook.org'
fragment@502 = 'News'
url@395 = 'http://www.fuzzingbook.org/#News'

url@374 = 'ftp://freebsd.org/releases/5.8'
url@492 = '//freebsd.org/releases/5.8'
scheme@492 = 'ftp'
url@494 = '/releases/5.8'
netloc@494 = 'freebsd.org'
url@395 = 'ftp://freebsd.org/releases/5.8'
url@396 = '/releases/5.8'

```

变量的行号可以从 [urllib/parse.py](https://github.com/python/cpython/blob/3.6/Lib/urllib/parse.py) 的源代码中进行验证。

### 恢复推导树

处理变量重新赋值是否有助于我们的 URL 示例？我们接下来看看这些。

```py
class TreeMiner(TreeMiner):
    def get_derivation_tree(self):
        tree = (START_SYMBOL, [(self.my_input, [])])
        for var, value in self.my_assignments:
            self.log(0, "%s=%s" % (var, repr(value)))
            self.apply_new_definition(tree, var, value)
        return tree 
```

#### 示例 1：恢复 URL 推导树

首先，我们获取 URL 1 的推导树

##### URL 1 推导树

```py
clear_cache()
with Tracer(URLS_X[0], files=['urllib/parse.py']) as tracer:
    urlparse(tracer.my_input)
sm = AssignmentTracker(tracer.my_input, tracer.trace)
dt = TreeMiner(tracer.my_input, sm.my_assignments.defined_vars())
display_tree(dt.tree) 
```

<svg width="506pt" height="324pt" viewBox="0.00 0.00 505.88 323.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 319.75)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="123.38" y="-302.45" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="123.38" y="-252.2" font-family="Times,serif" font-size="14.00"><url@374></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="45.38" y="-201.95" font-family="Times,serif" font-size="14.00"><scheme@492></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="123.38" y="-201.95" font-family="Times,serif" font-size="14.00">: (58)</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="188.38" y="-201.95" font-family="Times,serif" font-size="14.00"><url@492></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="45.38" y="-151.7" font-family="Times,serif" font-size="14.00">http</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="125.38" y="-151.7" font-family="Times,serif" font-size="14.00">//</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="188.38" y="-151.7" font-family="Times,serif" font-size="14.00"><netloc@494></text></g> <g id="edge7" class="edge"><title>5->7</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="336.38" y="-151.7" font-family="Times,serif" font-size="14.00"><url@494></text></g> <g id="edge9" class="edge"><title>5->9</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="159.38" y="-101.45" font-family="Times,serif" font-size="14.00">user:pass@www.google.com:80</text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="297.38" y="-101.45" font-family="Times,serif" font-size="14.00"><url@502></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="364.38" y="-101.45" font-family="Times,serif" font-size="14.00"># (35)</text></g> <g id="edge14" class="edge"><title>9->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="448.38" y="-101.45" font-family="Times,serif" font-size="14.00"><fragment@502></text></g> <g id="edge15" class="edge"><title>9->15</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="265.38" y="-51.2" font-family="Times,serif" font-size="14.00">/?</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="328.38" y="-51.2" font-family="Times,serif" font-size="14.00"><query@504></text></g> <g id="edge12" class="edge"><title>10->12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="328.38" y="-0.95" font-family="Times,serif" font-size="14.00">q=path</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="448.38" y="-51.2" font-family="Times,serif" font-size="14.00">ref</text></g> <g id="edge16" class="edge"><title>15->16</title></g></g></svg>

接下来，我们获取 URL 4 的推导树

##### URL 4 推导树

```py
clear_cache()
with Tracer(URLS_X[-1], files=['urllib/parse.py']) as tracer:
    urlparse(tracer.my_input)
sm = AssignmentTracker(tracer.my_input, tracer.trace)
dt = TreeMiner(tracer.my_input, sm.my_assignments.defined_vars())
display_tree(dt.tree) 
```

<svg width="322pt" height="274pt" viewBox="0.00 0.00 322.12 273.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 269.5)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="123.38" y="-252.2" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="123.38" y="-201.95" font-family="Times,serif" font-size="14.00"><url@374></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="45.38" y="-151.7" font-family="Times,serif" font-size="14.00"><scheme@492></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="123.38" y="-151.7" font-family="Times,serif" font-size="14.00">: (58)</text></g> <g id="edge4" class="edge"><title>1->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="188.38" y="-151.7" font-family="Times,serif" font-size="14.00"><url@492></text></g> <g id="edge5" class="edge"><title>1->5</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="45.38" y="-101.45" font-family="Times,serif" font-size="14.00">ftp</text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="125.38" y="-101.45" font-family="Times,serif" font-size="14.00">//</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="188.38" y="-101.45" font-family="Times,serif" font-size="14.00"><netloc@494></text></g> <g id="edge7" class="edge"><title>5->7</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="280.38" y="-101.45" font-family="Times,serif" font-size="14.00"><url@494></text></g> <g id="edge9" class="edge"><title>5->9</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="188.38" y="-51.2" font-family="Times,serif" font-size="14.00">freebsd.org</text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="280.38" y="-51.2" font-family="Times,serif" font-size="14.00"><url@396></text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="280.38" y="-0.95" font-family="Times,serif" font-size="14.00">/releases/5.8</text></g> <g id="edge11" class="edge"><title>10->11</title></g></g></svg>

分析树似乎属于同一个语法。因此，我们获得了整个集合的语法。首先，我们更新`recover_grammar()`以使用`AssignTracker`。

### 恢复语法

```py
class GrammarMiner(GrammarMiner):
    def update_grammar(self, inputstr, trace):
        at = self.create_tracker(inputstr, trace)
        dt = self.create_tree_miner(inputstr, at.my_assignments.defined_vars())
        self.add_tree(dt)
        return self.grammar

    def create_tracker(self, *args):
        return AssignmentTracker(*args)

    def create_tree_miner(self, *args):
        return TreeMiner(*args) 
```

接下来，我们使用修改后的`recover_grammar()`对从 URL 获得的推导树进行处理。

```py
url_grammar = recover_grammar(url_parse, URLS_X, files=['urllib/parse.py']) 
```

恢复的语法如下。

```py
syntax_diagram(url_grammar) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 199.5 62" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">url@374</text></g></g></g></g></svg>

```py
url@374

```

<svg class="railroad-diagram" height="62" viewBox="0 0 373.0 62" width="373.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="112.5" y="35">scheme@492</text></g> <g class="terminal"><text x="199.25" y="35">:</text></g> <g class="non-terminal"><text x="273.25" y="35">url@492</text></g></g></g></g></svg>

```py
scheme@492

```

<svg class="railroad-diagram" height="122" viewBox="0 0 182.5 122" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="91.25" y="35">http</text></g></g> <g><g class="terminal"><text x="91.25" y="65">ftp</text></g></g> <g><g class="terminal"><text x="91.25" y="95">https</text></g></g></g></g></svg>

```py
url@492

```

<svg class="railroad-diagram" height="92" viewBox="0 0 381.5 92" width="381.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="78.5" y="35">//</text></g> <g class="non-terminal"><text x="169.5" y="35">netloc@494</text></g> <g class="non-terminal"><text x="281.75" y="35">url@494</text></g></g> <g><g class="terminal"><text x="104.0" y="65">//</text></g> <g class="non-terminal"><text x="195.0" y="65">netloc@494</text></g> <g class="terminal"><text x="281.75" y="65">/</text></g></g></g></g></svg>

```py
netloc@494

```

<svg class="railroad-diagram" height="152" viewBox="0 0 369.5 152" width="369.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="184.75" y="65">www.fuzzingbook.org</text></g></g> <g><g class="terminal"><text x="184.75" y="35">www.cispa.saarland:80</text></g></g> <g><g class="terminal"><text x="184.75" y="95">freebsd.org</text></g></g> <g><g class="terminal"><text x="184.75" y="125">user:pass@www.google.com:80</text></g></g></g></g></svg>

```py
url@494

```

<svg class="railroad-diagram" height="122" viewBox="0 0 390.0 122" width="390.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="195.0" y="35">url@396</text></g></g> <g><g class="terminal"><text x="124.0" y="65">/#</text></g> <g class="non-terminal"><text x="223.5" y="65">fragment@502</text></g></g> <g><g class="non-terminal"><text x="99.75" y="95">url@502</text></g> <g class="terminal"><text x="173.75" y="95">#</text></g> <g class="non-terminal"><text x="269.0" y="95">fragment@502</text></g></g></g></g></svg>

```py
url@502

```

<svg class="railroad-diagram" height="62" viewBox="0 0 273.5 62" width="273.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="78.5" y="35">/?</text></g> <g class="non-terminal"><text x="165.25" y="35">query@504</text></g></g></g></g></svg>

```py
query@504

```

<svg class="railroad-diagram" height="62" viewBox="0 0 191.0 62" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="95.5" y="35">q=path</text></g></g></g></g></svg>

```py
fragment@502

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">News</text></g></g> <g><g class="terminal"><text x="87.0" y="65">ref</text></g></g></g></g></svg>

```py
url@396

```

<svg class="railroad-diagram" height="62" viewBox="0 0 250.5 62" width="250.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="125.25" y="35">/releases/5.8</text></g></g></g></g></svg>

让我们稍微模糊一下，看看产生的值是否合理。

```py
f = GrammarFuzzer(url_grammar)
for _ in range(10):
    print(f.fuzz()) 
```

```py
http://freebsd.org/
ftp://freebsd.org/releases/5.8
http://www.cispa.saarland:80/
ftp://freebsd.org/releases/5.8
https://user:pass@www.google.com:80/releases/5.8
https://freebsd.org/
ftp://www.cispa.saarland:80/?q=path#News
http://www.fuzzingbook.org/
https://www.cispa.saarland:80/
ftp://user:pass@www.google.com:80/

```

我们的修改似乎有所帮助。接下来，我们检查是否还能检索到库存的语法。

#### 示例 2：恢复库存语法

```py
inventory_grammar = recover_grammar(process_vehicle, VEHICLES) 
```

```py
syntax_diagram(inventory_grammar) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 225.0 62" width="225.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="112.5" y="35">vehicle@29</text></g></g></g></g></svg>

```py
vehicle@29

```

<svg class="railroad-diagram" height="62" viewBox="0 0 677.5 62" width="677.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="99.75" y="35">year@30</text></g> <g class="terminal"><text x="173.75" y="35">,</text></g> <g class="non-terminal"><text x="247.75" y="35">kind@30</text></g> <g class="terminal"><text x="321.75" y="35">,</text></g> <g class="non-terminal"><text x="408.5" y="35">company@30</text></g> <g class="terminal"><text x="495.25" y="35">,</text></g> <g class="non-terminal"><text x="573.5" y="35">model@30</text></g></g></g></g></svg>

```py
year@30

```

<svg class="railroad-diagram" height="122" viewBox="0 0 174.0 122" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1999</text></g></g> <g><g class="terminal"><text x="87.0" y="65">2000</text></g></g> <g><g class="terminal"><text x="87.0" y="95">1997</text></g></g></g></g></svg>

```py
kind@30

```

<svg class="railroad-diagram" height="92" viewBox="0 0 165.5 92" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="82.75" y="35">car</text></g></g> <g><g class="terminal"><text x="82.75" y="65">van</text></g></g></g></g></svg>

```py
company@30

```

<svg class="railroad-diagram" height="122" viewBox="0 0 199.5 122" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Mercury</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Chevy</text></g></g> <g><g class="terminal"><text x="99.75" y="95">Ford</text></g></g></g></g></svg>

```py
model@30

```

<svg class="railroad-diagram" height="122" viewBox="0 0 199.5 122" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">E350</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Cougar</text></g></g> <g><g class="terminal"><text x="99.75" y="95">Venture</text></g></g></g></g></svg>

使用模糊化从语法中产生值。

```py
f = GrammarFuzzer(inventory_grammar)
for _ in range(10):
    print(f.fuzz()) 
```

```py
1997,van,Chevy,E350
1999,van,Mercury,E350
2000,van,Chevy,Venture
2000,van,Ford,E350
1997,van,Mercury,Cougar
1997,car,Ford,E350
1997,car,Mercury,Venture
1997,car,Mercury,E350
2000,van,Mercury,Cougar
1997,car,Chevy,Venture

```

### 语法挖掘器重新赋值的问题

我们语法挖掘器的一个问题是它还没有考虑到当前上下文。也就是说，在替换时，一个变量可以替换它无法访问的标记（因此，它不是一个片段）。考虑以下例子。

```py
with Tracer(INVENTORY) as tracer:
    process_inventory(tracer.my_input)
sm = AssignmentTracker(tracer.my_input, tracer.trace)
dt = TreeMiner(tracer.my_input, sm.my_assignments.defined_vars())
display_tree(dt.tree, graph_attr=lr_graph) 
```

<svg width="508pt" height="598pt" viewBox="0.00 0.00 508.25 598.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 594.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-256.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="123" y="-256.95" font-family="Times,serif" font-size="14.00"><inventory@22></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="247.12" y="-464.95" font-family="Times,serif" font-size="14.00"><vehicle@24></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="247.12" y="-288.95" font-family="Times,serif" font-size="14.00">\n (10)</text></g> <g id="edge14" class="edge"><title>1->14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="247.12" y="-256.95" font-family="Times,serif" font-size="14.00"><vehicle@24></text></g> <g id="edge15" class="edge"><title>1->15</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="247.12" y="-224.95" font-family="Times,serif" font-size="14.00">\n (10)</text></g> <g id="edge27" class="edge"><title>1->27</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="247.12" y="-80.95" font-family="Times,serif" font-size="14.00"><vehicle@24></text></g> <g id="edge28" class="edge"><title>1->28</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="370.5" y="-576.95" font-family="Times,serif" font-size="14.00"><year@30></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="370.5" y="-544.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge5" class="edge"><title>2->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="370.5" y="-512.95" font-family="Times,serif" font-size="14.00"><kind@30></text></g> <g id="edge6" class="edge"><title>2->6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="370.5" y="-480.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge8" class="edge"><title>2->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="370.5" y="-448.95" font-family="Times,serif" font-size="14.00"><company@30></text></g> <g id="edge9" class="edge"><title>2->9</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="370.5" y="-416.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge11" class="edge"><title>2->11</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="370.5" y="-384.95" font-family="Times,serif" font-size="14.00"><model@30></text></g> <g id="edge12" class="edge"><title>2->12</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="476.62" y="-576.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="476.62" y="-512.95" font-family="Times,serif" font-size="14.00">van</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="476.62" y="-448.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge10" class="edge"><title>9->10</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="476.62" y="-384.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="370.5" y="-352.95" font-family="Times,serif" font-size="14.00"><year@30></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="370.5" y="-320.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge18" class="edge"><title>15->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="370.5" y="-288.95" font-family="Times,serif" font-size="14.00"><kind@30></text></g> <g id="edge19" class="edge"><title>15->19</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="370.5" y="-256.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge21" class="edge"><title>15->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="370.5" y="-224.95" font-family="Times,serif" font-size="14.00"><company@30></text></g> <g id="edge22" class="edge"><title>15->22</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="370.5" y="-192.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge24" class="edge"><title>15->24</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="370.5" y="-160.95" font-family="Times,serif" font-size="14.00"><model@30></text></g> <g id="edge25" class="edge"><title>15->25</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="476.62" y="-352.95" font-family="Times,serif" font-size="14.00">2000</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="476.62" y="-288.95" font-family="Times,serif" font-size="14.00">car</text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="476.62" y="-224.95" font-family="Times,serif" font-size="14.00">Mercury</text></g> <g id="edge23" class="edge"><title>22->23</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="476.62" y="-160.95" font-family="Times,serif" font-size="14.00">Cougar</text></g> <g id="edge26" class="edge"><title>25->26</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="370.5" y="-128.95" font-family="Times,serif" font-size="14.00"><year@30></text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="370.5" y="-96.95" font-family="Times,serif" font-size="14.00">,car,</text></g> <g id="edge31" class="edge"><title>28->31</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="370.5" y="-64.95" font-family="Times,serif" font-size="14.00"><company@30></text></g> <g id="edge32" class="edge"><title>28->32</title></g> <g id="node35" class="node"><title>34</title> <text text-anchor="middle" x="370.5" y="-32.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge34" class="edge"><title>28->34</title></g> <g id="node36" class="node"><title>35</title> <text text-anchor="middle" x="370.5" y="-0.95" font-family="Times,serif" font-size="14.00"><model@30></text></g> <g id="edge35" class="edge"><title>28->35</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="476.62" y="-128.95" font-family="Times,serif" font-size="14.00">1999</text></g> <g id="edge30" class="edge"><title>29->30</title></g> <g id="node34" class="node"><title>33</title> <text text-anchor="middle" x="476.62" y="-64.95" font-family="Times,serif" font-size="14.00">Chevy</text></g> <g id="edge33" class="edge"><title>32->33</title></g> <g id="node37" class="node"><title>36</title> <text text-anchor="middle" x="476.62" y="-0.95" font-family="Times,serif" font-size="14.00">Venture</text></g> <g id="edge36" class="edge"><title>35->36</title></g></g></svg>

如所示，我们获得的分析树并不完全符合我们的预期。如果我们启用`TreeMiner`的日志记录，问题就很容易被发现。

```py
dt = TreeMiner(tracer.my_input, sm.my_assignments.defined_vars(), log=True) 
```

```py
 inventory@22='1997,van,Ford,E350\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture'
	 - Node: <start>		? (<inventory@22>:'1997,van,Ford,E350\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture')
		 -> [0] '1997,van,Ford,E350\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture'
		  > ['<inventory@22>']
 vehicle@24='1997,van,Ford,E350'
	 - Node: <start>		? (<vehicle@24>:'1997,van,Ford,E350')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<vehicle@24>:'1997,van,Ford,E350')
		 -> [0] '1997,van,Ford,E350\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture'
		  > ['<vehicle@24>', '\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture']
 year@30='1997'
	 - Node: <start>		? (<year@30>:'1997')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<year@30>:'1997')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<year@30>:'1997')
		 -> [0] '1997,van,Ford,E350'
		  > ['<year@30>', ',van,Ford,E350']
 kind@30='van'
	 - Node: <start>		? (<kind@30>:'van')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<kind@30>:'van')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<kind@30>:'van')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<kind@30>:'van')
		 -> [0] '1997'
		 -> [1] ',van,Ford,E350'
		  > [',', '<kind@30>', ',Ford,E350']
 company@30='Ford'
	 - Node: <start>		? (<company@30>:'Ford')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<company@30>:'Ford')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<company@30>:'Ford')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<company@30>:'Ford')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<company@30>:'Ford')
		 -> [0] 'van'
		 -> [3] ',Ford,E350'
		  > [',', '<company@30>', ',E350']
 model@30='E350'
	 - Node: <start>		? (<model@30>:'E350')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<model@30>:'E350')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<model@30>:'E350')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<model@30>:'E350')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<model@30>:'E350')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<model@30>:'E350')
		 -> [0] 'Ford'
		 -> [5] ',E350'
		  > [',', '<model@30>']
 vehicle@24='2000,car,Mercury,Cougar'
	 - Node: <start>		? (<vehicle@24>:'2000,car,Mercury,Cougar')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<vehicle@24>:'2000,car,Mercury,Cougar')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<vehicle@24>:'2000,car,Mercury,Cougar')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<vehicle@24>:'2000,car,Mercury,Cougar')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<vehicle@24>:'2000,car,Mercury,Cougar')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<vehicle@24>:'2000,car,Mercury,Cougar')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<vehicle@24>:'2000,car,Mercury,Cougar')
		 -> [0] 'E350'
		 -> [1] '\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture'
		  > ['\n', '<vehicle@24>', '\n1999,car,Chevy,Venture']
 year@30='2000'
	 - Node: <start>		? (<year@30>:'2000')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<year@30>:'2000')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<year@30>:'2000')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<year@30>:'2000')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<year@30>:'2000')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<year@30>:'2000')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<year@30>:'2000')
		 -> [0] 'E350'
		 -> [1] '\n'
		 -> [2] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<year@30>:'2000')
		 -> [0] '2000,car,Mercury,Cougar'
		  > ['<year@30>', ',car,Mercury,Cougar']
 kind@30='car'
	 - Node: <start>		? (<kind@30>:'car')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<kind@30>:'car')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<kind@30>:'car')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<kind@30>:'car')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<kind@30>:'car')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<kind@30>:'car')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<kind@30>:'car')
		 -> [0] 'E350'
		 -> [1] '\n'
		 -> [2] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<kind@30>:'car')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<kind@30>:'car')
		 -> [0] '2000'
		 -> [1] ',car,Mercury,Cougar'
		  > [',', '<kind@30>', ',Mercury,Cougar']
 company@30='Mercury'
	 - Node: <start>		? (<company@30>:'Mercury')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<company@30>:'Mercury')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<company@30>:'Mercury')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<company@30>:'Mercury')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<company@30>:'Mercury')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<company@30>:'Mercury')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<company@30>:'Mercury')
		 -> [0] 'E350'
		 -> [1] '\n'
		 -> [2] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<company@30>:'Mercury')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<company@30>:'Mercury')
		 -> [0] '2000'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<company@30>:'Mercury')
		 -> [0] 'car'
		 -> [3] ',Mercury,Cougar'
		  > [',', '<company@30>', ',Cougar']
 model@30='Cougar'
	 - Node: <start>		? (<model@30>:'Cougar')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<model@30>:'Cougar')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<model@30>:'Cougar')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<model@30>:'Cougar')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<model@30>:'Cougar')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<model@30>:'Cougar')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<model@30>:'Cougar')
		 -> [0] 'E350'
		 -> [1] '\n'
		 -> [2] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<model@30>:'Cougar')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<model@30>:'Cougar')
		 -> [0] '2000'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<model@30>:'Cougar')
		 -> [0] 'car'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<model@30>:'Cougar')
		 -> [0] 'Mercury'
		 -> [5] ',Cougar'
		  > [',', '<model@30>']
 vehicle@24='1999,car,Chevy,Venture'
	 - Node: <start>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] 'E350'
		 -> [1] '\n'
		 -> [2] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] '2000'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] 'car'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] 'Mercury'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<vehicle@24>:'1999,car,Chevy,Venture')
		 -> [0] 'Cougar'
		 -> [3] '\n1999,car,Chevy,Venture'
		  > ['\n', '<vehicle@24>']
 year@30='1999'
	 - Node: <start>		? (<year@30>:'1999')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<year@30>:'1999')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<year@30>:'1999')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<year@30>:'1999')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<year@30>:'1999')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<year@30>:'1999')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<year@30>:'1999')
		 -> [0] 'E350'
		 -> [1] '\n'
		 -> [2] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<year@30>:'1999')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<year@30>:'1999')
		 -> [0] '2000'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<year@30>:'1999')
		 -> [0] 'car'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<year@30>:'1999')
		 -> [0] 'Mercury'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<year@30>:'1999')
		 -> [0] 'Cougar'
		 -> [3] '\n'
		 -> [4] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<year@30>:'1999')
		 -> [0] '1999,car,Chevy,Venture'
		  > ['<year@30>', ',car,Chevy,Venture']
 company@30='Chevy'
	 - Node: <start>		? (<company@30>:'Chevy')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<company@30>:'Chevy')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<company@30>:'Chevy')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<company@30>:'Chevy')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<company@30>:'Chevy')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<company@30>:'Chevy')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<company@30>:'Chevy')
		 -> [0] 'E350'
		 -> [1] '\n'
		 -> [2] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<company@30>:'Chevy')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<company@30>:'Chevy')
		 -> [0] '2000'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<company@30>:'Chevy')
		 -> [0] 'car'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<company@30>:'Chevy')
		 -> [0] 'Mercury'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<company@30>:'Chevy')
		 -> [0] 'Cougar'
		 -> [3] '\n'
		 -> [4] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<company@30>:'Chevy')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<company@30>:'Chevy')
		 -> [0] '1999'
		 -> [1] ',car,Chevy,Venture'
		  > [',car,', '<company@30>', ',Venture']
 model@30='Venture'
	 - Node: <start>		? (<model@30>:'Venture')
		 -> [0] '<inventory@22>'
	 - Node: <inventory@22>		? (<model@30>:'Venture')
		 -> [0] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<model@30>:'Venture')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<model@30>:'Venture')
		 -> [0] '1997'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<model@30>:'Venture')
		 -> [0] 'van'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<model@30>:'Venture')
		 -> [0] 'Ford'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<model@30>:'Venture')
		 -> [0] 'E350'
		 -> [1] '\n'
		 -> [2] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<model@30>:'Venture')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<model@30>:'Venture')
		 -> [0] '2000'
		 -> [1] ','
		 -> [2] '<kind@30>'
	 - Node: <kind@30>		? (<model@30>:'Venture')
		 -> [0] 'car'
		 -> [3] ','
		 -> [4] '<company@30>'
	 - Node: <company@30>		? (<model@30>:'Venture')
		 -> [0] 'Mercury'
		 -> [5] ','
		 -> [6] '<model@30>'
	 - Node: <model@30>		? (<model@30>:'Venture')
		 -> [0] 'Cougar'
		 -> [3] '\n'
		 -> [4] '<vehicle@24>'
	 - Node: <vehicle@24>		? (<model@30>:'Venture')
		 -> [0] '<year@30>'
	 - Node: <year@30>		? (<model@30>:'Venture')
		 -> [0] '1999'
		 -> [1] ',car,'
		 -> [2] '<company@30>'
	 - Node: <company@30>		? (<model@30>:'Venture')
		 -> [0] 'Chevy'
		 -> [3] ',Venture'
		  > [',', '<model@30>']

```

看看最后一条语句。我们有一个值`1999,car,`，其中只有`year`被替换了。我们不再有`'car'`变量来继续这里的替换。这是因为`'1999,car,Chevy,Venture'`中的`'car'`值没有被当作新值处理，因为`'car'`值在另一个方法调用（`'2000,car,Mercury,Cougar'`）的相同位置已经出现过。

## 带作用域的语法挖掘器

我们需要将当前上下文中的变量检查纳入其中。我们已经有了一个方法调用栈，这样我们可以在任何时刻获取当前方法。我们需要对变量做同样的处理。

为了做到这一点，我们将`CallStack`扩展到新的类`InputStack`，该类保存了调用的方法以及观察到的参数。它本质上是对方法激活的记录。我们从栈底的原始输入开始，对于每个新的方法调用，我们将该调用的参数作为新记录推入栈中。

### 输入栈

```py
class InputStack(CallStack):
    def __init__(self, i, fragment_len=FRAGMENT_LEN):
        self.inputs = [{START_SYMBOL: i}]
        self.fragment_len = fragment_len
        super().__init__() 
```

为了检查特定变量是否需要保存，我们定义了`in_current_record()`，它只检查当前作用域中的变量是否包含（而不是原始输入字符串）。

```py
class InputStack(InputStack):
    def in_current_record(self, val):
        return any(val in var for var in self.inputs[-1].values()) 
```

```py
my_istack = InputStack('hello my world') 
```

```py
my_istack.in_current_record('hello') 
```

```py
True

```

```py
my_istack.in_current_record('bye') 
```

```py
False

```

```py
my_istack.inputs.append({'greeting': 'hello', 'location': 'world'}) 
```

```py
my_istack.in_current_record('hello') 
```

```py
True

```

```py
my_istack.in_current_record('my') 
```

```py
False

```

我们定义了一个`ignored()`方法，如果变量不是字符串，或者变量长度小于定义的`fragment_len`，则返回 true。

```py
class InputStack(InputStack):
    def ignored(self, val):
        return not (isinstance(val, str) and len(val) >= self.fragment_len) 
```

```py
my_istack = InputStack('hello world')
my_istack.ignored(1) 
```

```py
True

```

```py
my_istack.ignored('a') 
```

```py
True

```

```py
my_istack.ignored('help') 
```

```py
False

```

我们现在可以定义一个`in_scope()`方法，该方法检查变量是否需要被忽略，如果不忽略，则检查变量值是否存在于当前作用域中。

```py
class InputStack(InputStack):
    def in_scope(self, k, val):
        if self.ignored(val):
            return False
        return self.in_current_record(val) 
```

最后，我们更新`enter()`，将当前上下文中的相关变量推送到栈中。

```py
class InputStack(InputStack):
    def enter(self, method, inputs):
        my_inputs = {k: v for k, v in inputs.items() if self.in_scope(k, v)}
        self.inputs.append(my_inputs)
        super().enter(method) 
```

当一个方法返回时，我们还需要一个相应的`leave()`来弹出输入并展开栈。

```py
class InputStack(InputStack):
    def leave(self):
        self.inputs.pop()
        super().leave() 
```

### ScopedVars

我们需要更新我们的`AssignmentVars`，包括变量定义的作用域信息。我们首先从更新`method_init()`开始。

```py
class ScopedVars(AssignmentVars):
    def method_init(self):
        self.call_stack = self.create_call_stack(self.my_input)
        self.event_locations = {self.call_stack.method_id: []}

    def create_call_stack(self, i):
        return InputStack(i) 
```

同样，`method_enter()`现在初始化当前方法调用的`accessed_seq_var`。

```py
class ScopedVars(ScopedVars):
    def method_enter(self, cxt, my_vars):
        self.current_event = 'call'
        self.call_stack.enter(cxt.method, my_vars)
        self.accessed_seq_var[self.call_stack.method_id] = {}
        self.event_locations[self.call_stack.method_id] = []
        self.register_event(cxt)
        self.update(my_vars) 
```

`update()`方法现在保存定义值的上下文。在函数参数的情况下，上下文应该是函数被调用的上下文。另一方面，在语句执行期间定义的值将具有当前上下文。

此外，我们在值上而不是在键上做注释，因为我们不希望在下一行参数上下文中重复变量。它们将具有相同的值，但不同的上下文，因为它们存在于语句执行中。

```py
class ScopedVars(ScopedVars):
    def update(self, v):
        if self.current_event == 'call':
            context = -2
        elif self.current_event == 'line':
            context = -1
        else:
            context = -1
        for k, v in v.items():
            self._set_kv(k, (v, self.call_stack.at(context)))
        self.var_location_register(self.new_vars)
        self.new_vars = set() 
```

我们还需要保存当前方法调用以确定哪些变量在作用域中。现在，这个信息被纳入变量名作为`accessed_seq_var[method_id][var]`。

```py
class ScopedVars(ScopedVars):
    def var_name(self, var):
        return (var, self.call_stack.method_id,
                self.accessed_seq_var[self.call_stack.method_id][var]) 
```

如前所述，`var_access`简单地初始化相应的计数器，这次是在`method_id`的上下文中。

```py
class ScopedVars(ScopedVars):
    def var_access(self, var):
        if var not in self.accessed_seq_var[self.call_stack.method_id]:
            self.accessed_seq_var[self.call_stack.method_id][var] = 0
        return self.var_name(var) 
```

在变量重新赋值期间，我们更新`accessed_seq_var`以反映新的计数。

```py
class ScopedVars(ScopedVars):
    def var_assign(self, var):
        self.accessed_seq_var[self.call_stack.method_id][var] += 1
        self.new_vars.add(self.var_name(var))
        return self.var_name(var) 
```

我们现在更新`defined_vars()`以考虑新的信息。

```py
class ScopedVars(ScopedVars):
    def defined_vars(self, formatted=True):
        def fmt(k):
            method, i = k[1]
            v = (method, i, k[0], self.var_def_lines[k])
            return "%s[%d]:%s@%s" % v if formatted else v

        return [(fmt(k), v) for k, v in self.defs.items()] 
```

更新`seq_vars()`以考虑新的信息。

```py
class ScopedVars(ScopedVars):
    def seq_vars(self, formatted=True):
        def fmt(k):
            method, i = k[1]
            v = (method, i, k[0], self.var_def_lines[k], k[2])
            return "%s[%d]:%s@%s:%s" % v if formatted else v

        return {fmt(k): v for k, v in self.defs.items()} 
```

### 作用域跟踪器

在定义了`InputStack`和`Vars`之后，我们现在可以定义`ScopeTracker`。`ScopeTracker`只保存当前作用域中存在的变量。

```py
class ScopeTracker(AssignmentTracker):
    def __init__(self, my_input, trace, **kwargs):
        self.current_event = None
        super().__init__(my_input, trace, **kwargs)

    def create_assignments(self, *args):
        return ScopedVars(*args) 
```

我们定义了一个检查变量是否存在于作用域中的包装器。

```py
class ScopeTracker(ScopeTracker):
    def is_input_fragment(self, var, value):
        return self.my_assignments.call_stack.in_scope(var, value) 
```

我们可以这样使用`ScopeTracker`。

```py
vehicle_traces = []
with Tracer(INVENTORY) as tracer:
    process_inventory(tracer.my_input)
sm = ScopeTracker(tracer.my_input, tracer.trace)
vehicle_traces.append((tracer.my_input, sm))
for k, v in sm.my_assignments.seq_vars().items():
    print(k, '=', repr(v)) 
```

```py
process_inventory[1]:inventory@22:1 = ('1997,van,Ford,E350\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture', ('<start>', 0))
process_inventory[1]:inventory@22:2 = ('1997,van,Ford,E350\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture', ('process_inventory', 1))
process_inventory[1]:vehicle@24:1 = ('1997,van,Ford,E350', ('process_inventory', 1))
process_vehicle[2]:vehicle@29:1 = ('1997,van,Ford,E350', ('process_inventory', 1))
process_vehicle[2]:vehicle@29:2 = ('1997,van,Ford,E350', ('process_vehicle', 2))
process_vehicle[2]:year@30:1 = ('1997', ('process_vehicle', 2))
process_vehicle[2]:kind@30:1 = ('van', ('process_vehicle', 2))
process_vehicle[2]:company@30:1 = ('Ford', ('process_vehicle', 2))
process_vehicle[2]:model@30:1 = ('E350', ('process_vehicle', 2))
process_van[3]:year@40:1 = ('1997', ('process_vehicle', 2))
process_van[3]:company@40:1 = ('Ford', ('process_vehicle', 2))
process_van[3]:model@40:1 = ('E350', ('process_vehicle', 2))
process_van[3]:year@40:2 = ('1997', ('process_van', 3))
process_van[3]:company@40:2 = ('Ford', ('process_van', 3))
process_van[3]:model@40:2 = ('E350', ('process_van', 3))
process_inventory[1]:vehicle@24:2 = ('2000,car,Mercury,Cougar', ('process_inventory', 1))
process_vehicle[4]:vehicle@29:1 = ('2000,car,Mercury,Cougar', ('process_inventory', 1))
process_vehicle[4]:vehicle@29:2 = ('2000,car,Mercury,Cougar', ('process_vehicle', 4))
process_vehicle[4]:year@30:1 = ('2000', ('process_vehicle', 4))
process_vehicle[4]:kind@30:1 = ('car', ('process_vehicle', 4))
process_vehicle[4]:company@30:1 = ('Mercury', ('process_vehicle', 4))
process_vehicle[4]:model@30:1 = ('Cougar', ('process_vehicle', 4))
process_car[5]:year@49:1 = ('2000', ('process_vehicle', 4))
process_car[5]:company@49:1 = ('Mercury', ('process_vehicle', 4))
process_car[5]:model@49:1 = ('Cougar', ('process_vehicle', 4))
process_car[5]:year@49:2 = ('2000', ('process_car', 5))
process_car[5]:company@49:2 = ('Mercury', ('process_car', 5))
process_car[5]:model@49:2 = ('Cougar', ('process_car', 5))
process_inventory[1]:vehicle@24:3 = ('1999,car,Chevy,Venture', ('process_inventory', 1))
process_vehicle[6]:vehicle@29:1 = ('1999,car,Chevy,Venture', ('process_inventory', 1))
process_vehicle[6]:vehicle@29:2 = ('1999,car,Chevy,Venture', ('process_vehicle', 6))
process_vehicle[6]:year@30:1 = ('1999', ('process_vehicle', 6))
process_vehicle[6]:kind@30:1 = ('car', ('process_vehicle', 6))
process_vehicle[6]:company@30:1 = ('Chevy', ('process_vehicle', 6))
process_vehicle[6]:model@30:1 = ('Venture', ('process_vehicle', 6))
process_car[7]:year@49:1 = ('1999', ('process_vehicle', 6))
process_car[7]:company@49:1 = ('Chevy', ('process_vehicle', 6))
process_car[7]:model@49:1 = ('Venture', ('process_vehicle', 6))
process_car[7]:year@49:2 = ('1999', ('process_car', 7))
process_car[7]:company@49:2 = ('Chevy', ('process_car', 7))
process_car[7]:model@49:2 = ('Venture', ('process_car', 7))

```

### 恢复推导树

在`apply_new_definition()`中的主要区别是，我们添加了一个第二个条件来检查作用域。特别是，变量只能替换作用域中存在的字符串片段的部分。变量作用域由`scope`指示。然而，仅仅考虑作用域是不够的。例如，考虑下面的片段。

```py
def my_fn(stringval):
    partA, partB = stringval.split('/')
    return partA, partB

svalue = ...
v1, v2 = my_fn(svalue) 
```

在这里，`v1`和`v2`从先前的函数调用中获取它们的值，而不是从它们当前上下文中获取。也就是说，我们必须为内部子方法调用可能生成的大片段的情况提供异常。为了考虑这一点，我们定义了`mseq()`，它检索方法调用序列。在上面的情况下，内部子方法调用的`mseq()`将大于当前的`mseq()`。如果是这样，我们允许替换进行。

```py
class ScopeTreeMiner(TreeMiner):
    def mseq(self, key):
        method, seq, var, lno = key
        return seq 
```

`nt_var()`方法需要接受元组并从中生成一个非终结符号。我们跳过方法序列，因为它与语法无关。

```py
class ScopeTreeMiner(ScopeTreeMiner):
    def nt_var(self, key):
        method, seq, var, lno = key
        return to_nonterminal("%s@%d:%s" % (method, lno, var)) 
```

我们现在重新定义`apply_new_definition()`以考虑上下文和作用域。特别是，只有当变量在*作用域*内时，变量才能替换值的一部分 -- 即，它的作用域（方法序列号）与值的序列号相同，无论是作为参数的调用上下文还是当前上下文。当值的序列号大于变量的序列号时，会做出例外。在这种情况下，值可能来自内部调用。在这种情况下，我们允许替换继续进行。

```py
class ScopeTreeMiner(ScopeTreeMiner):
    def partition(self, part, value):
        return value.partition(part)
    def partition_by_part(self, pair, value):
        (nt_var, nt_seq), (v, v_scope) = pair
        prefix_k_suffix = [
                    (nt_var, [(v, [], nt_seq)]) if i == 1 else (e, [])
                    for i, e in enumerate(self.partition(v, value))
                    if e]
        return prefix_k_suffix

    def insert_into_tree(self, my_tree, pair):
        var, values, my_scope = my_tree
        (nt_var, nt_seq), (v, v_scope) = pair
        applied = False
        for i, value_ in enumerate(values):
            key, arr, scope = value_
            self.log(2, "-> [%d] %s" % (i, repr(value_)))
            if is_nonterminal(key):
                applied = self.insert_into_tree(value_, pair)
                if applied:
                    break
            else:
                if v_scope != scope:
                    if nt_seq > scope:
                        continue
                if not v or not self.string_part_of_value(v, key):
                    continue
                prefix_k_suffix = [(k, children, scope) for k, children
                                   in self.partition_by_part(pair, key)]
                del values[i]
                for j, rep in enumerate(prefix_k_suffix):
                    values.insert(j + i, rep)

                applied = True
                self.log(2, " > %s" % (repr([i[0] for i in prefix_k_suffix])))
                break
        return applied 
```

`apply_new_definition()`现在被修改为携带额外的上下文信息`mseq`。

```py
class ScopeTreeMiner(ScopeTreeMiner):
    def apply_new_definition(self, tree, var, value_):
        nt_var = self.nt_var(var)
        seq = self.mseq(var)
        val, (smethod, mseq) = value_
        return self.insert_into_tree(tree, ((nt_var, seq), (val, mseq))) 
```

我们也修改了`get_derivation_tree()`，使得初始节点携带上下文。

```py
class ScopeTreeMiner(ScopeTreeMiner):
    def get_derivation_tree(self):
        tree = (START_SYMBOL, [(self.my_input, [], 0)], 0)
        for var, value in self.my_assignments:
            self.log(0, "%s=%s" % (var, repr(value)))
            self.apply_new_definition(tree, var, value)
        return tree 
```

#### 示例 1：恢复 URL 解析树

我们验证我们的 URL 解析树恢复仍然按预期工作。

```py
url_dts = []
for inputstr in URLS_X:
    clear_cache()
    with Tracer(inputstr, files=['urllib/parse.py']) as tracer:
        urlparse(tracer.my_input)
    sm = ScopeTracker(tracer.my_input, tracer.trace)
    for k, v in sm.my_assignments.defined_vars(formatted=False):
        print(k, '=', repr(v))
    dt = ScopeTreeMiner(
        tracer.my_input,
        sm.my_assignments.defined_vars(
            formatted=False))
    display_tree(dt.tree, graph_attr=lr_graph)
    url_dts.append(dt) 
```

```py
('urlparse', 1, 'url', 374) = ('http://user:pass@www.google.com:80/?q=path#ref', ('<start>', 0))
('urlparse', 1, 'url', 374) = ('http://user:pass@www.google.com:80/?q=path#ref', ('urlparse', 1))
('urlsplit', 3, 'url', 452) = ('http://user:pass@www.google.com:80/?q=path#ref', ('urlparse', 1))
('urlsplit', 3, 'url', 452) = ('http://user:pass@www.google.com:80/?q=path#ref', ('urlsplit', 3))
('urlsplit', 3, 'url', 492) = ('//user:pass@www.google.com:80/?q=path#ref', ('urlsplit', 3))
('urlsplit', 3, 'scheme', 492) = ('http', ('urlsplit', 3))
('_splitnetloc', 5, 'url', 413) = ('//user:pass@www.google.com:80/?q=path#ref', ('urlsplit', 3))
('_splitnetloc', 5, 'url', 413) = ('//user:pass@www.google.com:80/?q=path#ref', ('_splitnetloc', 5))
('urlsplit', 3, 'url', 494) = ('/?q=path#ref', ('urlsplit', 3))
('urlsplit', 3, 'netloc', 494) = ('user:pass@www.google.com:80', ('urlsplit', 3))
('urlsplit', 3, 'url', 502) = ('/?q=path', ('urlsplit', 3))
('urlsplit', 3, 'fragment', 502) = ('ref', ('urlsplit', 3))
('urlsplit', 3, 'query', 504) = ('q=path', ('urlsplit', 3))
('_checknetloc', 6, 'netloc', 421) = ('user:pass@www.google.com:80', ('urlsplit', 3))
('_checknetloc', 6, 'netloc', 421) = ('user:pass@www.google.com:80', ('_checknetloc', 6))
('urlparse', 1, 'scheme', 396) = ('http', ('urlparse', 1))
('urlparse', 1, 'netloc', 396) = ('user:pass@www.google.com:80', ('urlparse', 1))
('urlparse', 1, 'query', 396) = ('q=path', ('urlparse', 1))
('urlparse', 1, 'fragment', 396) = ('ref', ('urlparse', 1))
('urlparse', 1, 'url', 374) = ('https://www.cispa.saarland:80/', ('<start>', 0))
('urlparse', 1, 'url', 374) = ('https://www.cispa.saarland:80/', ('urlparse', 1))
('urlsplit', 3, 'url', 452) = ('https://www.cispa.saarland:80/', ('urlparse', 1))
('urlsplit', 3, 'url', 452) = ('https://www.cispa.saarland:80/', ('urlsplit', 3))
('urlsplit', 3, 'url', 492) = ('//www.cispa.saarland:80/', ('urlsplit', 3))
('urlsplit', 3, 'scheme', 492) = ('https', ('urlsplit', 3))
('_splitnetloc', 5, 'url', 413) = ('//www.cispa.saarland:80/', ('urlsplit', 3))
('_splitnetloc', 5, 'url', 413) = ('//www.cispa.saarland:80/', ('_splitnetloc', 5))
('urlsplit', 3, 'netloc', 494) = ('www.cispa.saarland:80', ('urlsplit', 3))
('_checknetloc', 6, 'netloc', 421) = ('www.cispa.saarland:80', ('urlsplit', 3))
('_checknetloc', 6, 'netloc', 421) = ('www.cispa.saarland:80', ('_checknetloc', 6))
('urlparse', 1, 'scheme', 396) = ('https', ('urlparse', 1))
('urlparse', 1, 'netloc', 396) = ('www.cispa.saarland:80', ('urlparse', 1))
('urlparse', 1, 'url', 374) = ('http://www.fuzzingbook.org/#News', ('<start>', 0))
('urlparse', 1, 'url', 374) = ('http://www.fuzzingbook.org/#News', ('urlparse', 1))
('urlsplit', 3, 'url', 452) = ('http://www.fuzzingbook.org/#News', ('urlparse', 1))
('urlsplit', 3, 'url', 452) = ('http://www.fuzzingbook.org/#News', ('urlsplit', 3))
('urlsplit', 3, 'url', 492) = ('//www.fuzzingbook.org/#News', ('urlsplit', 3))
('urlsplit', 3, 'scheme', 492) = ('http', ('urlsplit', 3))
('_splitnetloc', 5, 'url', 413) = ('//www.fuzzingbook.org/#News', ('urlsplit', 3))
('_splitnetloc', 5, 'url', 413) = ('//www.fuzzingbook.org/#News', ('_splitnetloc', 5))
('urlsplit', 3, 'url', 494) = ('/#News', ('urlsplit', 3))
('urlsplit', 3, 'netloc', 494) = ('www.fuzzingbook.org', ('urlsplit', 3))
('urlsplit', 3, 'fragment', 502) = ('News', ('urlsplit', 3))
('_checknetloc', 6, 'netloc', 421) = ('www.fuzzingbook.org', ('urlsplit', 3))
('_checknetloc', 6, 'netloc', 421) = ('www.fuzzingbook.org', ('_checknetloc', 6))
('urlparse', 1, 'scheme', 396) = ('http', ('urlparse', 1))
('urlparse', 1, 'netloc', 396) = ('www.fuzzingbook.org', ('urlparse', 1))
('urlparse', 1, 'fragment', 396) = ('News', ('urlparse', 1))
('urlparse', 1, 'url', 374) = ('ftp://freebsd.org/releases/5.8', ('<start>', 0))
('urlparse', 1, 'url', 374) = ('ftp://freebsd.org/releases/5.8', ('urlparse', 1))
('urlsplit', 3, 'url', 452) = ('ftp://freebsd.org/releases/5.8', ('urlparse', 1))
('urlsplit', 3, 'url', 452) = ('ftp://freebsd.org/releases/5.8', ('urlsplit', 3))
('urlsplit', 3, 'url', 492) = ('//freebsd.org/releases/5.8', ('urlsplit', 3))
('urlsplit', 3, 'scheme', 492) = ('ftp', ('urlsplit', 3))
('_splitnetloc', 5, 'url', 413) = ('//freebsd.org/releases/5.8', ('urlsplit', 3))
('_splitnetloc', 5, 'url', 413) = ('//freebsd.org/releases/5.8', ('_splitnetloc', 5))
('urlsplit', 3, 'url', 494) = ('/releases/5.8', ('urlsplit', 3))
('urlsplit', 3, 'netloc', 494) = ('freebsd.org', ('urlsplit', 3))
('_checknetloc', 6, 'netloc', 421) = ('freebsd.org', ('urlsplit', 3))
('_checknetloc', 6, 'netloc', 421) = ('freebsd.org', ('_checknetloc', 6))
('urlparse', 1, 'url', 396) = ('/releases/5.8', ('urlparse', 1))
('urlparse', 1, 'scheme', 396) = ('ftp', ('urlparse', 1))
('urlparse', 1, 'netloc', 396) = ('freebsd.org', ('urlparse', 1))

```

#### 示例 2：恢复库存解析树

接下来，我们看看如何从上次失败的`process_inventory()`中恢复解析树。

```py
with Tracer(INVENTORY) as tracer:
    process_inventory(tracer.my_input)

sm = ScopeTracker(tracer.my_input, tracer.trace)
for k, v in sm.my_assignments.defined_vars():
    print(k, '=', repr(v))
inventory_dt = ScopeTreeMiner(
    tracer.my_input,
    sm.my_assignments.defined_vars(
        formatted=False))
display_tree(inventory_dt.tree, graph_attr=lr_graph) 
```

```py
process_inventory[1]:inventory@22 = ('1997,van,Ford,E350\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture', ('<start>', 0))
process_inventory[1]:inventory@22 = ('1997,van,Ford,E350\n2000,car,Mercury,Cougar\n1999,car,Chevy,Venture', ('process_inventory', 1))
process_inventory[1]:vehicle@24 = ('1997,van,Ford,E350', ('process_inventory', 1))
process_vehicle[2]:vehicle@29 = ('1997,van,Ford,E350', ('process_inventory', 1))
process_vehicle[2]:vehicle@29 = ('1997,van,Ford,E350', ('process_vehicle', 2))
process_vehicle[2]:year@30 = ('1997', ('process_vehicle', 2))
process_vehicle[2]:kind@30 = ('van', ('process_vehicle', 2))
process_vehicle[2]:company@30 = ('Ford', ('process_vehicle', 2))
process_vehicle[2]:model@30 = ('E350', ('process_vehicle', 2))
process_van[3]:year@40 = ('1997', ('process_vehicle', 2))
process_van[3]:company@40 = ('Ford', ('process_vehicle', 2))
process_van[3]:model@40 = ('E350', ('process_vehicle', 2))
process_van[3]:year@40 = ('1997', ('process_van', 3))
process_van[3]:company@40 = ('Ford', ('process_van', 3))
process_van[3]:model@40 = ('E350', ('process_van', 3))
process_inventory[1]:vehicle@24 = ('2000,car,Mercury,Cougar', ('process_inventory', 1))
process_vehicle[4]:vehicle@29 = ('2000,car,Mercury,Cougar', ('process_inventory', 1))
process_vehicle[4]:vehicle@29 = ('2000,car,Mercury,Cougar', ('process_vehicle', 4))
process_vehicle[4]:year@30 = ('2000', ('process_vehicle', 4))
process_vehicle[4]:kind@30 = ('car', ('process_vehicle', 4))
process_vehicle[4]:company@30 = ('Mercury', ('process_vehicle', 4))
process_vehicle[4]:model@30 = ('Cougar', ('process_vehicle', 4))
process_car[5]:year@49 = ('2000', ('process_vehicle', 4))
process_car[5]:company@49 = ('Mercury', ('process_vehicle', 4))
process_car[5]:model@49 = ('Cougar', ('process_vehicle', 4))
process_car[5]:year@49 = ('2000', ('process_car', 5))
process_car[5]:company@49 = ('Mercury', ('process_car', 5))
process_car[5]:model@49 = ('Cougar', ('process_car', 5))
process_inventory[1]:vehicle@24 = ('1999,car,Chevy,Venture', ('process_inventory', 1))
process_vehicle[6]:vehicle@29 = ('1999,car,Chevy,Venture', ('process_inventory', 1))
process_vehicle[6]:vehicle@29 = ('1999,car,Chevy,Venture', ('process_vehicle', 6))
process_vehicle[6]:year@30 = ('1999', ('process_vehicle', 6))
process_vehicle[6]:kind@30 = ('car', ('process_vehicle', 6))
process_vehicle[6]:company@30 = ('Chevy', ('process_vehicle', 6))
process_vehicle[6]:model@30 = ('Venture', ('process_vehicle', 6))
process_car[7]:year@49 = ('1999', ('process_vehicle', 6))
process_car[7]:company@49 = ('Chevy', ('process_vehicle', 6))
process_car[7]:model@49 = ('Venture', ('process_vehicle', 6))
process_car[7]:year@49 = ('1999', ('process_car', 7))
process_car[7]:company@49 = ('Chevy', ('process_car', 7))
process_car[7]:model@49 = ('Venture', ('process_car', 7))

```

<svg width="1852pt" height="662pt" viewBox="0.00 0.00 1851.50 662.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 658.25)"><g id="node1" class="node"><title>0</title> <text text-anchor="middle" x="19.88" y="-320.95" font-family="Times,serif" font-size="14.00"><start></text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="174.38" y="-320.95" font-family="Times,serif" font-size="14.00"><process_inventory@22:inventory></text></g> <g id="edge1" class="edge"><title>0->1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="middle" x="407.62" y="-320.95" font-family="Times,serif" font-size="14.00"><process_inventory@22:inventory></text></g> <g id="edge2" class="edge"><title>1->2</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="634.5" y="-432.95" font-family="Times,serif" font-size="14.00"><process_inventory@24:vehicle></text></g> <g id="edge3" class="edge"><title>2->3</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="634.5" y="-352.95" font-family="Times,serif" font-size="14.00">\n (10)</text></g> <g id="edge23" class="edge"><title>2->23</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="634.5" y="-320.95" font-family="Times,serif" font-size="14.00"><process_inventory@24:vehicle></text></g> <g id="edge24" class="edge"><title>2->24</title></g> <g id="node45" class="node"><title>44</title> <text text-anchor="middle" x="634.5" y="-288.95" font-family="Times,serif" font-size="14.00">\n (10)</text></g> <g id="edge44" class="edge"><title>2->44</title></g> <g id="node46" class="node"><title>45</title> <text text-anchor="middle" x="634.5" y="-216.95" font-family="Times,serif" font-size="14.00"><process_inventory@24:vehicle></text></g> <g id="edge45" class="edge"><title>2->45</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="848.62" y="-464.95" font-family="Times,serif" font-size="14.00"><process_vehicle@29:vehicle></text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="1056.38" y="-528.95" font-family="Times,serif" font-size="14.00"><process_vehicle@29:vehicle></text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="1269.75" y="-640.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:year></text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="middle" x="1269.75" y="-608.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge10" class="edge"><title>5->10</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="middle" x="1269.75" y="-576.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:kind></text></g> <g id="edge11" class="edge"><title>5->11</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="1269.75" y="-544.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge13" class="edge"><title>5->13</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="1269.75" y="-512.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:company></text></g> <g id="edge14" class="edge"><title>5->14</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="1269.75" y="-480.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge18" class="edge"><title>5->18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="middle" x="1269.75" y="-448.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:model></text></g> <g id="edge19" class="edge"><title>5->19</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="1479" y="-640.95" font-family="Times,serif" font-size="14.00"><process_van@40:year></text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="1678.5" y="-640.95" font-family="Times,serif" font-size="14.00"><process_van@40:year></text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="1819.88" y="-640.95" font-family="Times,serif" font-size="14.00">1997</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="1479" y="-576.95" font-family="Times,serif" font-size="14.00">van</text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="middle" x="1479" y="-512.95" font-family="Times,serif" font-size="14.00"><process_van@40:company></text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="1678.5" y="-512.95" font-family="Times,serif" font-size="14.00"><process_van@40:company></text></g> <g id="edge16" class="edge"><title>15->16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="1819.88" y="-512.95" font-family="Times,serif" font-size="14.00">Ford</text></g> <g id="edge17" class="edge"><title>16->17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="1479" y="-448.95" font-family="Times,serif" font-size="14.00"><process_van@40:model></text></g> <g id="edge20" class="edge"><title>19->20</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="1678.5" y="-448.95" font-family="Times,serif" font-size="14.00"><process_van@40:model></text></g> <g id="edge21" class="edge"><title>20->21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="1819.88" y="-448.95" font-family="Times,serif" font-size="14.00">E350</text></g> <g id="edge22" class="edge"><title>21->22</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="848.62" y="-320.95" font-family="Times,serif" font-size="14.00"><process_vehicle@29:vehicle></text></g> <g id="edge25" class="edge"><title>24->25</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="1056.38" y="-320.95" font-family="Times,serif" font-size="14.00"><process_vehicle@29:vehicle></text></g> <g id="edge26" class="edge"><title>25->26</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="middle" x="1269.75" y="-416.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:year></text></g> <g id="edge27" class="edge"><title>26->27</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="middle" x="1269.75" y="-384.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge31" class="edge"><title>26->31</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="1269.75" y="-352.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:kind></text></g> <g id="edge32" class="edge"><title>26->32</title></g> <g id="node35" class="node"><title>34</title> <text text-anchor="middle" x="1269.75" y="-320.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge34" class="edge"><title>26->34</title></g> <g id="node36" class="node"><title>35</title> <text text-anchor="middle" x="1269.75" y="-288.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:company></text></g> <g id="edge35" class="edge"><title>26->35</title></g> <g id="node40" class="node"><title>39</title> <text text-anchor="middle" x="1269.75" y="-256.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge39" class="edge"><title>26->39</title></g> <g id="node41" class="node"><title>40</title> <text text-anchor="middle" x="1269.75" y="-224.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:model></text></g> <g id="edge40" class="edge"><title>26->40</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="1479" y="-416.95" font-family="Times,serif" font-size="14.00"><process_car@49:year></text></g> <g id="edge28" class="edge"><title>27->28</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="1678.5" y="-416.95" font-family="Times,serif" font-size="14.00"><process_car@49:year></text></g> <g id="edge29" class="edge"><title>28->29</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="middle" x="1819.88" y="-416.95" font-family="Times,serif" font-size="14.00">2000</text></g> <g id="edge30" class="edge"><title>29->30</title></g> <g id="node34" class="node"><title>33</title> <text text-anchor="middle" x="1479" y="-352.95" font-family="Times,serif" font-size="14.00">car</text></g> <g id="edge33" class="edge"><title>32->33</title></g> <g id="node37" class="node"><title>36</title> <text text-anchor="middle" x="1479" y="-288.95" font-family="Times,serif" font-size="14.00"><process_car@49:company></text></g> <g id="edge36" class="edge"><title>35->36</title></g> <g id="node38" class="node"><title>37</title> <text text-anchor="middle" x="1678.5" y="-288.95" font-family="Times,serif" font-size="14.00"><process_car@49:company></text></g> <g id="edge37" class="edge"><title>36->37</title></g> <g id="node39" class="node"><title>38</title> <text text-anchor="middle" x="1819.88" y="-288.95" font-family="Times,serif" font-size="14.00">Mercury</text></g> <g id="edge38" class="edge"><title>37->38</title></g> <g id="node42" class="node"><title>41</title> <text text-anchor="middle" x="1479" y="-224.95" font-family="Times,serif" font-size="14.00"><process_car@49:model></text></g> <g id="edge41" class="edge"><title>40->41</title></g> <g id="node43" class="node"><title>42</title> <text text-anchor="middle" x="1678.5" y="-224.95" font-family="Times,serif" font-size="14.00"><process_car@49:model></text></g> <g id="edge42" class="edge"><title>41->42</title></g> <g id="node44" class="node"><title>43</title> <text text-anchor="middle" x="1819.88" y="-224.95" font-family="Times,serif" font-size="14.00">Cougar</text></g> <g id="edge43" class="edge"><title>42->43</title></g> <g id="node47" class="node"><title>46</title> <text text-anchor="middle" x="848.62" y="-136.95" font-family="Times,serif" font-size="14.00"><process_vehicle@29:vehicle></text></g> <g id="edge46" class="edge"><title>45->46</title></g> <g id="node48" class="node"><title>47</title> <text text-anchor="middle" x="1056.38" y="-112.95" font-family="Times,serif" font-size="14.00"><process_vehicle@29:vehicle></text></g> <g id="edge47" class="edge"><title>46->47</title></g> <g id="node49" class="node"><title>48</title> <text text-anchor="middle" x="1269.75" y="-192.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:year></text></g> <g id="edge48" class="edge"><title>47->48</title></g> <g id="node53" class="node"><title>52</title> <text text-anchor="middle" x="1269.75" y="-160.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge52" class="edge"><title>47->52</title></g> <g id="node54" class="node"><title>53</title> <text text-anchor="middle" x="1269.75" y="-128.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:kind></text></g> <g id="edge53" class="edge"><title>47->53</title></g> <g id="node56" class="node"><title>55</title> <text text-anchor="middle" x="1269.75" y="-96.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge55" class="edge"><title>47->55</title></g> <g id="node57" class="node"><title>56</title> <text text-anchor="middle" x="1269.75" y="-64.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:company></text></g> <g id="edge56" class="edge"><title>47->56</title></g> <g id="node61" class="node"><title>60</title> <text text-anchor="middle" x="1269.75" y="-32.95" font-family="Times,serif" font-size="14.00">, (44)</text></g> <g id="edge60" class="edge"><title>47->60</title></g> <g id="node62" class="node"><title>61</title> <text text-anchor="middle" x="1269.75" y="-0.95" font-family="Times,serif" font-size="14.00"><process_vehicle@30:model></text></g> <g id="edge61" class="edge"><title>47->61</title></g> <g id="node50" class="node"><title>49</title> <text text-anchor="middle" x="1479" y="-192.95" font-family="Times,serif" font-size="14.00"><process_car@49:year></text></g> <g id="edge49" class="edge"><title>48->49</title></g> <g id="node51" class="node"><title>50</title> <text text-anchor="middle" x="1678.5" y="-192.95" font-family="Times,serif" font-size="14.00"><process_car@49:year></text></g> <g id="edge50" class="edge"><title>49->50</title></g> <g id="node52" class="node"><title>51</title> <text text-anchor="middle" x="1819.88" y="-192.95" font-family="Times,serif" font-size="14.00">1999</text></g> <g id="edge51" class="edge"><title>50->51</title></g> <g id="node55" class="node"><title>54</title> <text text-anchor="middle" x="1479" y="-128.95" font-family="Times,serif" font-size="14.00">car</text></g> <g id="edge54" class="edge"><title>53->54</title></g> <g id="node58" class="node"><title>57</title> <text text-anchor="middle" x="1479" y="-64.95" font-family="Times,serif" font-size="14.00"><process_car@49:company></text></g> <g id="edge57" class="edge"><title>56->57</title></g> <g id="node59" class="node"><title>58</title> <text text-anchor="middle" x="1678.5" y="-64.95" font-family="Times,serif" font-size="14.00"><process_car@49:company></text></g> <g id="edge58" class="edge"><title>57->58</title></g> <g id="node60" class="node"><title>59</title> <text text-anchor="middle" x="1819.88" y="-64.95" font-family="Times,serif" font-size="14.00">Chevy</text></g> <g id="edge59" class="edge"><title>58->59</title></g> <g id="node63" class="node"><title>62</title> <text text-anchor="middle" x="1479" y="-0.95" font-family="Times,serif" font-size="14.00"><process_car@49:model></text></g> <g id="edge62" class="edge"><title>61->62</title></g> <g id="node64" class="node"><title>63</title> <text text-anchor="middle" x="1678.5" y="-0.95" font-family="Times,serif" font-size="14.00"><process_car@49:model></text></g> <g id="edge63" class="edge"><title>62->63</title></g> <g id="node65" class="node"><title>64</title> <text text-anchor="middle" x="1819.88" y="-0.95" font-family="Times,serif" font-size="14.00">Venture</text></g> <g id="edge64" class="edge"><title>63->64</title></g></g></svg>

恢复的解析树看起来是合理的。

从我们的示例（2）中，人们可能会注意到三个子树 -- `vehicle[2:1]`、`vehicle[4:1]`和`vehicle[6:1]`非常相似。我们将探讨如何利用这一点直接生成语法。

### 语法挖掘

`tree_to_grammar()`现在重新定义为如下，以考虑节点中的额外作用域。

```py
class ScopedGrammarMiner(GrammarMiner):
    def tree_to_grammar(self, tree):
        key, children, scope = tree
        one_alt = [ckey for ckey, gchildren, cscope in children if ckey != key]
        hsh = {key: [one_alt] if one_alt else []}
        for child in children:
            (ckey, _gc, _cscope) = child
            if not is_nonterminal(ckey):
                continue
            chsh = self.tree_to_grammar(child)
            for k in chsh:
                if k not in hsh:
                    hsh[k] = chsh[k]
                else:
                    hsh[k].extend(chsh[k])
        return hsh 
```

语法是规范形式，需要调整以显示。首先，恢复的库存语法。

```py
si = ScopedGrammarMiner()
si.add_tree(inventory_dt)
syntax_diagram(readable(si.grammar)) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 395.0 62" width="395.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="197.5" y="35">process_inventory@22:inventory</text></g></g></g></g></svg>

```py
process_inventory@22:inventory

```

<svg class="railroad-diagram" height="62" viewBox="0 0 1031.0 62" width="1031.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="189.0" y="35">process_inventory@24:vehicle</text></g><g class="non-terminal"><text x="515.5" y="35">process_inventory@24:vehicle</text></g><g class="non-terminal"><text x="842.0" y="35">process_inventory@24:vehicle</text></g></g></g></g></svg>

```py
process_inventory@24:vehicle

```

<svg class="railroad-diagram" height="62" viewBox="0 0 361.0 62" width="361.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="180.5" y="35">process_vehicle@29:vehicle</text></g></g></g></g></svg>

```py
process_vehicle@29:vehicle

```

<svg class="railroad-diagram" height="62" viewBox="0 0 1221.5 62" width="1221.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="167.75" y="35">process_vehicle@30:year</text></g> <g class="terminal"><text x="309.75" y="35">,</text></g> <g class="non-terminal"><text x="451.75" y="35">process_vehicle@30:kind</text></g> <g class="terminal"><text x="593.75" y="35">,</text></g> <g class="non-terminal"><text x="748.5" y="35">process_vehicle@30:company</text></g> <g class="terminal"><text x="903.25" y="35">,</text></g> <g class="non-terminal"><text x="1049.5" y="35">process_vehicle@30:model</text></g></g></g></g></svg>

```py
process_vehicle@30:year

```

<svg class="railroad-diagram" height="92" viewBox="0 0 301.5 92" width="301.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">process_car@49:year</text></g></g> <g><g class="non-terminal"><text x="150.75" y="65">process_van@40:year</text></g></g></g></g></svg>

```py
process_van@40:year

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1997</text></g></g></g></g></svg>

```py
process_vehicle@30:kind

```

<svg class="railroad-diagram" height="92" viewBox="0 0 165.5 92" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="82.75" y="35">car</text></g></g> <g><g class="terminal"><text x="82.75" y="65">van</text></g></g></g></g></svg>

```py
process_vehicle@30:company

```

<svg class="railroad-diagram" height="92" viewBox="0 0 327.0 92" width="327.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="163.5" y="35">process_van@40:company</text></g></g> <g><g class="non-terminal"><text x="163.5" y="65">process_car@49:company</text></g></g></g></g></svg>

```py
process_van@40:company

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">Ford</text></g></g></g></g></svg>

```py
process_vehicle@30:model

```

<svg class="railroad-diagram" height="92" viewBox="0 0 310.0 92" width="310.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="155.0" y="35">process_van@40:model</text></g></g> <g><g class="non-terminal"><text x="155.0" y="65">process_car@49:model</text></g></g></g></g></svg>

```py
process_van@40:model

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">E350</text></g></g></g></g></svg>

```py
process_car@49:year

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1999</text></g></g> <g><g class="terminal"><text x="87.0" y="65">2000</text></g></g></g></g></svg>

```py
process_car@49:company

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Mercury</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Chevy</text></g></g></g></g></svg>

```py
process_car@49:model

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Cougar</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Venture</text></g></g></g></g></svg>

恢复的 URL 语法。

```py
su = ScopedGrammarMiner()
for t in url_dts:
    su.add_tree(t)
syntax_diagram(readable(su.grammar)) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 276.0 62" width="276.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="138.0" y="35">urlparse@374:url</text></g></g></g></g></svg>

```py
urlparse@374:url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 276.0 62" width="276.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="138.0" y="35">urlsplit@452:url</text></g></g></g></g></svg>

```py
urlsplit@452:url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 526.0 62" width="526.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">urlsplit@492:scheme</text></g> <g class="terminal"><text x="275.75" y="35">:</text></g> <g class="non-terminal"><text x="388.0" y="35">urlsplit@492:url</text></g></g></g></g></svg>

```py
urlsplit@492:scheme

```

<svg class="railroad-diagram" height="62" viewBox="0 0 301.5 62" width="301.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">urlparse@396:scheme</text></g></g></g></g></svg>

```py
urlparse@396:scheme

```

<svg class="railroad-diagram" height="122" viewBox="0 0 182.5 122" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="91.25" y="35">http</text></g></g> <g><g class="terminal"><text x="91.25" y="65">ftp</text></g></g> <g><g class="terminal"><text x="91.25" y="95">https</text></g></g></g></g></svg>

```py
urlsplit@492:url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 310.0 62" width="310.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="155.0" y="35">_splitnetloc@413:url</text></g></g></g></g></svg>

```py
_splitnetloc@413:url

```

<svg class="railroad-diagram" height="92" viewBox="0 0 534.5 92" width="534.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="78.5" y="35">//</text></g> <g class="non-terminal"><text x="207.75" y="35">urlsplit@494:netloc</text></g> <g class="non-terminal"><text x="396.5" y="35">urlsplit@494:url</text></g></g> <g><g class="terminal"><text x="142.25" y="65">//</text></g> <g class="non-terminal"><text x="271.5" y="65">urlsplit@494:netloc</text></g> <g class="terminal"><text x="396.5" y="65">/</text></g></g></g></g></svg>

```py
urlsplit@494:netloc

```

<svg class="railroad-diagram" height="62" viewBox="0 0 335.5 62" width="335.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="167.75" y="35">_checknetloc@421:netloc</text></g></g></g></g></svg>

```py
_checknetloc@421:netloc

```

<svg class="railroad-diagram" height="62" viewBox="0 0 301.5 62" width="301.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">urlparse@396:netloc</text></g></g></g></g></svg>

```py
urlparse@396:netloc

```

<svg class="railroad-diagram" height="152" viewBox="0 0 369.5 152" width="369.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="184.75" y="65">www.fuzzingbook.org</text></g></g> <g><g class="terminal"><text x="184.75" y="35">www.cispa.saarland:80</text></g></g> <g><g class="terminal"><text x="184.75" y="95">freebsd.org</text></g></g> <g><g class="terminal"><text x="184.75" y="125">user:pass@www.google.com:80</text></g></g></g></g></svg>

```py
urlsplit@494:url

```

<svg class="railroad-diagram" height="122" viewBox="0 0 543.0 122" width="543.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="271.5" y="35">urlparse@396:url</text></g></g> <g><g class="non-terminal"><text x="138.0" y="65">urlsplit@502:url</text></g> <g class="terminal"><text x="250.25" y="65">#</text></g> <g class="non-terminal"><text x="383.75" y="65">urlsplit@502:fragment</text></g></g> <g><g class="terminal"><text x="162.25" y="95">/#</text></g> <g class="non-terminal"><text x="300.0" y="95">urlsplit@502:fragment</text></g></g></g></g></svg>

```py
urlsplit@502:url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 350.0 62" width="350.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="78.5" y="35">/?</text></g> <g class="non-terminal"><text x="203.5" y="35">urlsplit@504:query</text></g></g></g></g></svg>

```py
urlsplit@504:query

```

<svg class="railroad-diagram" height="62" viewBox="0 0 293.0 62" width="293.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="146.5" y="35">urlparse@396:query</text></g></g></g></g></svg>

```py
urlparse@396:query

```

<svg class="railroad-diagram" height="62" viewBox="0 0 191.0 62" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="95.5" y="35">q=path</text></g></g></g></g></svg>

```py
urlsplit@502:fragment

```

<svg class="railroad-diagram" height="62" viewBox="0 0 318.5 62" width="318.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="159.25" y="35">urlparse@396:fragment</text></g></g></g></g></svg>

```py
urlparse@396:fragment

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">News</text></g></g> <g><g class="terminal"><text x="87.0" y="65">ref</text></g></g></g></g></svg>

```py
urlparse@396:url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 250.5 62" width="250.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="125.25" y="35">/releases/5.8</text></g></g></g></g></svg>

人们可能会注意到语法并非完全可读，有许多单标记定义。

因此，最后一部分拼图是清理方法`clean_grammar()`，它清理了这样的定义。想法是寻找单标记定义，其中键正好由另一个键（单选方案、单标记、非终结符）定义。

```py
class ScopedGrammarMiner(ScopedGrammarMiner):
    def get_replacements(self, grammar):
        replacements = {}
        for k in grammar:
            if k == START_SYMBOL:
                continue
            alts = grammar[k]
            if len(set([str(i) for i in alts])) != 1:
                continue
            rule = alts[0]
            if len(rule) != 1:
                continue
            tok = rule[0]
            if not is_nonterminal(tok):
                continue
            replacements[k] = tok
        return replacements 
```

一旦我们有了这样一个列表，就迭代地用我们之前找到的标记替换原始键的任何位置。重复此操作，直到没有剩余的键。

```py
class ScopedGrammarMiner(ScopedGrammarMiner):
    def clean_grammar(self):
        replacements = self.get_replacements(self.grammar)

        while True:
            changed = set()
            for k in self.grammar:
                if k in replacements:
                    continue
                new_alts = []
                for alt in self.grammar[k]:
                    new_alt = []
                    for t in alt:
                        if t in replacements:
                            new_alt.append(replacements[t])
                            changed.add(t)
                        else:
                            new_alt.append(t)
                    new_alts.append(new_alt)
                self.grammar[k] = new_alts
            if not changed:
                break
            for k in changed:
                self.grammar.pop(k, None)
        return readable(self.grammar) 
```

`clean_grammar()`的使用如下：

```py
si = ScopedGrammarMiner()
si.add_tree(inventory_dt)
syntax_diagram(readable(si.clean_grammar())) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 395.0 62" width="395.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="197.5" y="35">process_inventory@22:inventory</text></g></g></g></g></svg>

```py
process_inventory@22:inventory

```

<svg class="railroad-diagram" height="62" viewBox="0 0 980.0 62" width="980.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="180.5" y="35">process_vehicle@29:vehicle</text></g><g class="non-terminal"><text x="490.0" y="35">process_vehicle@29:vehicle</text></g><g class="non-terminal"><text x="799.5" y="35">process_vehicle@29:vehicle</text></g></g></g></g></svg>

```py
process_vehicle@29:vehicle

```

<svg class="railroad-diagram" height="62" viewBox="0 0 1221.5 62" width="1221.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="167.75" y="35">process_vehicle@30:year</text></g> <g class="terminal"><text x="309.75" y="35">,</text></g> <g class="non-terminal"><text x="451.75" y="35">process_vehicle@30:kind</text></g> <g class="terminal"><text x="593.75" y="35">,</text></g> <g class="non-terminal"><text x="748.5" y="35">process_vehicle@30:company</text></g> <g class="terminal"><text x="903.25" y="35">,</text></g> <g class="non-terminal"><text x="1049.5" y="35">process_vehicle@30:model</text></g></g></g></g></svg>

```py
process_vehicle@30:year

```

<svg class="railroad-diagram" height="92" viewBox="0 0 301.5 92" width="301.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">process_car@49:year</text></g></g> <g><g class="non-terminal"><text x="150.75" y="65">process_van@40:year</text></g></g></g></g></svg>

```py
process_van@40:year

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1997</text></g></g></g></g></svg>

```py
process_vehicle@30:kind

```

<svg class="railroad-diagram" height="92" viewBox="0 0 165.5 92" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="82.75" y="35">car</text></g></g> <g><g class="terminal"><text x="82.75" y="65">van</text></g></g></g></g></svg>

```py
process_vehicle@30:company

```

<svg class="railroad-diagram" height="92" viewBox="0 0 327.0 92" width="327.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="163.5" y="35">process_van@40:company</text></g></g> <g><g class="non-terminal"><text x="163.5" y="65">process_car@49:company</text></g></g></g></g></svg>

```py
process_van@40:company

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">Ford</text></g></g></g></g></svg>

```py
process_vehicle@30:model

```

<svg class="railroad-diagram" height="92" viewBox="0 0 310.0 92" width="310.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="155.0" y="35">process_van@40:model</text></g></g> <g><g class="non-terminal"><text x="155.0" y="65">process_car@49:model</text></g></g></g></g></svg>

```py
process_van@40:model

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">E350</text></g></g></g></g></svg>

```py
process_car@49:year

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1999</text></g></g> <g><g class="terminal"><text x="87.0" y="65">2000</text></g></g></g></g></svg>

```py
process_car@49:company

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Mercury</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Chevy</text></g></g></g></g></svg>

```py
process_car@49:model

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Cougar</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Venture</text></g></g></g></g></svg>

我们更新了`update_grammar()`以使用正确的跟踪器和挖掘器。

```py
class ScopedGrammarMiner(ScopedGrammarMiner):
    def update_grammar(self, inputstr, trace):
        at = self.create_tracker(inputstr, trace)
        dt = self.create_tree_miner(
            inputstr, at.my_assignments.defined_vars(
                formatted=False))
        self.add_tree(dt)
        return self.grammar

    def create_tracker(self, *args):
        return ScopeTracker(*args)

    def create_tree_miner(self, *args):
        return ScopeTreeMiner(*args) 
```

`recover_grammar()`使用正确的挖掘器，并返回一个清理过的语法。

```py
def recover_grammar(fn, inputs, **kwargs):
    miner = ScopedGrammarMiner()
    for inputstr in inputs:
        with Tracer(inputstr, **kwargs) as tracer:
            fn(tracer.my_input)
        miner.update_grammar(tracer.my_input, tracer.trace)
    return readable(miner.clean_grammar()) 
```

```py
url_grammar = recover_grammar(url_parse, URLS_X, files=['urllib/parse.py']) 
```

```py
syntax_diagram(url_grammar) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 276.0 62" width="276.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="138.0" y="35">urlsplit@452:url</text></g></g></g></g></svg>

```py
urlsplit@452:url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 560.0 62" width="560.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">urlparse@396:scheme</text></g> <g class="terminal"><text x="275.75" y="35">:</text></g> <g class="non-terminal"><text x="405.0" y="35">_splitnetloc@413:url</text></g></g></g></g></svg>

```py
urlparse@396:scheme

```

<svg class="railroad-diagram" height="122" viewBox="0 0 182.5 122" width="182.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="91.25" y="35">http</text></g></g> <g><g class="terminal"><text x="91.25" y="65">ftp</text></g></g> <g><g class="terminal"><text x="91.25" y="95">https</text></g></g></g></g></svg>

```py
_splitnetloc@413:url

```

<svg class="railroad-diagram" height="92" viewBox="0 0 534.5 92" width="534.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="142.25" y="35">//</text></g> <g class="non-terminal"><text x="271.5" y="35">urlparse@396:netloc</text></g> <g class="terminal"><text x="396.5" y="35">/</text></g></g> <g><g class="terminal"><text x="78.5" y="65">//</text></g> <g class="non-terminal"><text x="207.75" y="65">urlparse@396:netloc</text></g> <g class="non-terminal"><text x="396.5" y="65">urlsplit@494:url</text></g></g></g></g></svg>

```py
urlparse@396:netloc

```

<svg class="railroad-diagram" height="152" viewBox="0 0 369.5 152" width="369.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="184.75" y="65">www.fuzzingbook.org</text></g></g> <g><g class="terminal"><text x="184.75" y="35">www.cispa.saarland:80</text></g></g> <g><g class="terminal"><text x="184.75" y="95">freebsd.org</text></g></g> <g><g class="terminal"><text x="184.75" y="125">user:pass@www.google.com:80</text></g></g></g></g></svg>

```py
urlsplit@494:url

```

<svg class="railroad-diagram" height="122" viewBox="0 0 543.0 122" width="543.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="271.5" y="35">urlparse@396:url</text></g></g> <g><g class="non-terminal"><text x="138.0" y="65">urlsplit@502:url</text></g> <g class="terminal"><text x="250.25" y="65">#</text></g> <g class="non-terminal"><text x="383.75" y="65">urlparse@396:fragment</text></g></g> <g><g class="terminal"><text x="162.25" y="95">/#</text></g> <g class="non-terminal"><text x="300.0" y="95">urlparse@396:fragment</text></g></g></g></g></svg>

```py
urlsplit@502:url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 350.0 62" width="350.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="78.5" y="35">/?</text></g> <g class="non-terminal"><text x="203.5" y="35">urlparse@396:query</text></g></g></g></g></svg>

```py
urlparse@396:query

```

<svg class="railroad-diagram" height="62" viewBox="0 0 191.0 62" width="191.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="95.5" y="35">q=path</text></g></g></g></g></svg>

```py
urlparse@396:fragment

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">News</text></g></g> <g><g class="terminal"><text x="87.0" y="65">ref</text></g></g></g></g></svg>

```py
urlparse@396:url

```

<svg class="railroad-diagram" height="62" viewBox="0 0 250.5 62" width="250.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="125.25" y="35">/releases/5.8</text></g></g></g></g></svg>

```py
f = GrammarFuzzer(url_grammar)
for _ in range(10):
    print(f.fuzz()) 
```

```py
ftp://user:pass@www.google.com:80/
ftp://freebsd.org/?q=path#News
ftp://freebsd.org/
ftp://www.fuzzingbook.org/?q=path#News
https://www.fuzzingbook.org/
ftp://www.fuzzingbook.org/#News
ftp://www.fuzzingbook.org/releases/5.8
http://www.fuzzingbook.org/?q=path#ref
http://freebsd.org/
ftp://user:pass@www.google.com:80/

```

```py
inventory_grammar = recover_grammar(process_inventory, [INVENTORY]) 
```

```py
syntax_diagram(inventory_grammar) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 395.0 62" width="395.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="197.5" y="35">process_inventory@22:inventory</text></g></g></g></g></svg>

```py
process_inventory@22:inventory

```

<svg class="railroad-diagram" height="62" viewBox="0 0 980.0 62" width="980.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="180.5" y="35">process_vehicle@29:vehicle</text></g><g class="non-terminal"><text x="490.0" y="35">process_vehicle@29:vehicle</text></g><g class="non-terminal"><text x="799.5" y="35">process_vehicle@29:vehicle</text></g></g></g></g></svg>

```py
process_vehicle@29:vehicle

```

<svg class="railroad-diagram" height="62" viewBox="0 0 1221.5 62" width="1221.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="167.75" y="35">process_vehicle@30:year</text></g> <g class="terminal"><text x="309.75" y="35">,</text></g> <g class="non-terminal"><text x="451.75" y="35">process_vehicle@30:kind</text></g> <g class="terminal"><text x="593.75" y="35">,</text></g> <g class="non-terminal"><text x="748.5" y="35">process_vehicle@30:company</text></g> <g class="terminal"><text x="903.25" y="35">,</text></g> <g class="non-terminal"><text x="1049.5" y="35">process_vehicle@30:model</text></g></g></g></g></svg>

```py
process_vehicle@30:year

```

<svg class="railroad-diagram" height="92" viewBox="0 0 301.5 92" width="301.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">process_car@49:year</text></g></g> <g><g class="non-terminal"><text x="150.75" y="65">process_van@40:year</text></g></g></g></g></svg>

```py
process_van@40:year

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1997</text></g></g></g></g></svg>

```py
process_vehicle@30:kind

```

<svg class="railroad-diagram" height="92" viewBox="0 0 165.5 92" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="82.75" y="35">car</text></g></g> <g><g class="terminal"><text x="82.75" y="65">van</text></g></g></g></g></svg>

```py
process_vehicle@30:company

```

<svg class="railroad-diagram" height="92" viewBox="0 0 327.0 92" width="327.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="163.5" y="35">process_van@40:company</text></g></g> <g><g class="non-terminal"><text x="163.5" y="65">process_car@49:company</text></g></g></g></g></svg>

```py
process_van@40:company

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">Ford</text></g></g></g></g></svg>

```py
process_vehicle@30:model

```

<svg class="railroad-diagram" height="92" viewBox="0 0 310.0 92" width="310.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="155.0" y="35">process_van@40:model</text></g></g> <g><g class="non-terminal"><text x="155.0" y="65">process_car@49:model</text></g></g></g></g></svg>

```py
process_van@40:model

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">E350</text></g></g></g></g></svg>

```py
process_car@49:year

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1999</text></g></g> <g><g class="terminal"><text x="87.0" y="65">2000</text></g></g></g></g></svg>

```py
process_car@49:company

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Mercury</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Chevy</text></g></g></g></g></svg>

```py
process_car@49:model

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Cougar</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Venture</text></g></g></g></g></svg>

```py
f = GrammarFuzzer(inventory_grammar)
for _ in range(10):
    print(f.fuzz()) 
```

```py
1997,van,Mercury,E350
1999,car,Ford,Venture
2000,car,Ford,Cougar
2000,van,Chevy,Venture
1999,car,Mercury,E350
1997,van,Ford,Venture
1997,car,Chevy,Cougar
1999,car,Ford,E350
1999,car,Chevy,Cougar
1997,car,Chevy,Venture
1997,car,Ford,E350
2000,car,Mercury,E350
1999,van,Chevy,E350
1997,van,Ford,Cougar
1999,van,Chevy,Venture
1999,car,Ford,E350
1999,van,Mercury,Venture
1997,car,Ford,Cougar
1999,car,Mercury,Venture
1997,van,Mercury,E350
1999,car,Chevy,Cougar
2000,van,Chevy,Venture
2000,car,Ford,Venture
1997,car,Mercury,E350
1997,van,Chevy,E350
1997,van,Ford,E350
2000,car,Ford,E350
1997,car,Chevy,Venture
1997,van,Ford,E350
2000,van,Chevy,Cougar

```

我们看到跟踪作用域如何帮助我们提取更精确的语法。

注意，我们使用 *字符串* 包含测试作为确定特定字符串片段是否来自原始输入字符串的一种方式。虽然与动态污染相比，这可能看起来相当容易出错，但我们注意到，许多跟踪工具，如 `dtrace()` 和 `ptrace()`，允许在不同的平台上直接从二进制执行中获取我们所需的信息。然而，获取动态污染的方法几乎总是涉及在它们可以使用之前对二进制文件进行仪器化。因此，这种字符串包含的方法可以比动态污染方法更普遍地应用。此外，动态污染通常由于隐式传输或在 *Python* 和 *C* 代码之间的边界而丢失。字符串包含没有这样的问题。因此，我们的方法通常可以获得比依赖于动态污染更好的结果。

## 经验教训

+   给定一组程序的样本输入，如果程序依赖于手写解析器，我们可以通过检查执行过程中的变量值来学习输入语法。

+   简单的字符串包含检查足以从现实世界程序中获得相当准确的语法。

+   结果语法可以直接用于模糊测试，并且可以对您拥有的任何样本产生乘数效应。

## 下一步

+   学习如何使用 信息流 来进一步提高映射输入到状态。

## 背景

从一组样本中恢复语言（即不考虑可能处理它们的程序）是一个经过充分研究的话题。Higuera 的优秀参考文献 [De la Higuera 等人，2010] 涵盖了所有经典方法。黑盒语法挖掘的最新状态由 Clark 等人 [Clark 等人，2013] 描述。

从一个程序中学习输入语言，无论是否有样本，尽管它具有模糊测试的潜力，但仍然是一个新兴话题。在这个领域，Lin 等人进行了开创性的工作 [[Lin 等人，2008](https://doi.org/10.1145/1453101.1453114)]，他们发明了一种从自顶向下和自底向上解析器中检索解析树的方法。本章中描述的方法直接基于 Hoschele 等人的 AUTOGRAM 工作 [[Höschele 等人，2017](https://doi.org/10.1109/ICSE-C.2017.14)]。

## 练习

### 练习 1：简化复杂对象

我们的语法挖掘器只检查字符串片段。然而，程序可能经常传递包含输入片段的容器或自定义对象。例如，考虑我们对库存处理器的合理修改，我们使用自定义对象 `Vehicle` 来携带片段。

```py
class Vehicle:
    def __init__(self, vehicle: str):
        year, kind, company, model, *_ = vehicle.split(',')
        self.year, self.kind, self.company, self.model = year, kind, company, model 
```

```py
def process_inventory_with_obj(inventory: str) -> str:
    res = []
    for vehicle in inventory.split('\n'):
        ret = process_vehicle(vehicle)
        res.extend(ret)

    return '\n'.join(res) 
```

```py
def process_vehicle_with_obj(vehicle: str) -> List[str]:
    v = Vehicle(vehicle)
    if v.kind == 'van':
        return process_van_with_obj(v)

    elif v.kind == 'car':
        return process_car_with_obj(v)

    else:
        raise Exception('Invalid entry') 
```

```py
def process_van_with_obj(vehicle: Vehicle) -> List[str]:
    res = [
        "We have a %s  %s van from %s vintage." % (vehicle.company,
                                                  vehicle.model, vehicle.year)
    ]
    iyear = int(vehicle.year)
    if iyear > 2010:
        res.append("It is a recent model!")
    else:
        res.append("It is an old but reliable model!")
    return res 
```

```py
def process_car_with_obj(vehicle: Vehicle) -> List[str]:
    res = [
        "We have a %s  %s car from %s vintage." % (vehicle.company,
                                                  vehicle.model, vehicle.year)
    ]
    iyear = int(vehicle.year)
    if iyear > 2016:
        res.append("It is a recent model!")
    else:
        res.append("It is an old but reliable model!")
    return res 
```

我们像以前一样恢复语法。

```py
vehicle_grammar = recover_grammar(
    process_inventory_with_obj,
    [INVENTORY],
    methods=INVENTORY_METHODS) 
```

新的车辆语法在细节上缺失，特别是关于货车和汽车的不同的模型和公司。

```py
syntax_diagram(vehicle_grammar) 
```

```py
start

```

<svg class="railroad-diagram" height="62" viewBox="0 0 980.0 62" width="980.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="180.5" y="35">process_vehicle@29:vehicle</text></g><g class="non-terminal"><text x="490.0" y="35">process_vehicle@29:vehicle</text></g><g class="non-terminal"><text x="799.5" y="35">process_vehicle@29:vehicle</text></g></g></g></g></svg>

```py
process_vehicle@29:vehicle

```

<svg class="railroad-diagram" height="62" viewBox="0 0 1221.5 62" width="1221.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="167.75" y="35">process_vehicle@30:year</text></g> <g class="terminal"><text x="309.75" y="35">,</text></g> <g class="non-terminal"><text x="451.75" y="35">process_vehicle@30:kind</text></g> <g class="terminal"><text x="593.75" y="35">,</text></g> <g class="non-terminal"><text x="748.5" y="35">process_vehicle@30:company</text></g> <g class="terminal"><text x="903.25" y="35">,</text></g> <g class="non-terminal"><text x="1049.5" y="35">process_vehicle@30:model</text></g></g></g></g></svg>

```py
process_vehicle@30:year

```

<svg class="railroad-diagram" height="92" viewBox="0 0 301.5 92" width="301.5"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="150.75" y="35">process_car@49:year</text></g></g> <g><g class="non-terminal"><text x="150.75" y="65">process_van@40:year</text></g></g></g></g></svg>

```py
process_van@40:year

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1997</text></g></g></g></g></svg>

```py
process_vehicle@30:kind

```

<svg class="railroad-diagram" height="92" viewBox="0 0 165.5 92" width="165.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="82.75" y="35">car</text></g></g> <g><g class="terminal"><text x="82.75" y="65">van</text></g></g></g></g></svg>

```py
process_vehicle@30:company

```

<svg class="railroad-diagram" height="92" viewBox="0 0 327.0 92" width="327.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="163.5" y="35">process_van@40:company</text></g></g> <g><g class="non-terminal"><text x="163.5" y="65">process_car@49:company</text></g></g></g></g></svg>

```py
process_van@40:company

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">Ford</text></g></g></g></g></svg>

```py
process_vehicle@30:model

```

<svg class="railroad-diagram" height="92" viewBox="0 0 310.0 92" width="310.0"><g transform="translate(.5 .5)"><g><g><g class="non-terminal"><text x="155.0" y="35">process_van@40:model</text></g></g> <g><g class="non-terminal"><text x="155.0" y="65">process_car@49:model</text></g></g></g></g></svg>

```py
process_van@40:model

```

<svg class="railroad-diagram" height="62" viewBox="0 0 174.0 62" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">E350</text></g></g></g></g></svg>

```py
process_car@49:year

```

<svg class="railroad-diagram" height="92" viewBox="0 0 174.0 92" width="174.0"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="87.0" y="35">1999</text></g></g> <g><g class="terminal"><text x="87.0" y="65">2000</text></g></g></g></g></svg>

```py
process_car@49:company

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Mercury</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Chevy</text></g></g></g></g></svg>

```py
process_car@49:model

```

<svg class="railroad-diagram" height="92" viewBox="0 0 199.5 92" width="199.5"><g transform="translate(.5 .5)"><g><g><g class="terminal"><text x="99.75" y="35">Cougar</text></g></g> <g><g class="terminal"><text x="99.75" y="65">Venture</text></g></g></g></g></svg>

问题在于，我们在追踪过程中特别寻找包含输入字符串片段的字符串对象。你能修改我们的语法挖掘器，以便正确地考虑复杂的对象吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GrammarMiner.ipynb#Exercises)来处理练习并查看解决方案。

### 练习 2：从 InformationFlow 中引入污点

我们一直使用*字符串包含*来检查特定的片段是否来自输入字符串。这并不令人满意，因为它要求我们在跟踪字符串的大小上做出妥协，这限制在大于`FRAGMENT_LEN`的字符串。此外，可能存在一种方法可以处理一个字符串，其中片段重复，但属于不同的标记。例如，CSV 文件中的嵌套逗号会导致我们的解析器失败。避免这种情况的一种方法是通过*动态污点*，并检查污点包含而不是字符串包含。

关于信息流的章节详细说明了如何引入动态污点。你能根据作用域更新我们的语法挖掘器，使用*动态污点*吗？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/GrammarMiner.ipynb#Exercises)来处理练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png)本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码受[MIT License](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。 [最后更改：2023-11-11 18:18:06+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/GrammarMiner.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用这篇作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[Mining Input Grammars](https://www.fuzzingbook.org/html/GrammarMiner.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[The Fuzzing Book](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/GrammarMiner.html`](https://www.fuzzingbook.org/html/GrammarMiner.html). Retrieved 2023-11-11 18:18:06+01:00.

```py
@incollection{fuzzingbook2023:GrammarMiner,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Mining Input Grammars},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/GrammarMiner.html}},
    note = {Retrieved 2023-11-11 18:18:06+01:00},
    url = {https://www.fuzzingbook.org/html/GrammarMiner.html},
    urldate = {2023-11-11 18:18:06+01:00}
}

```
