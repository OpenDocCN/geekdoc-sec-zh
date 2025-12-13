# 跟踪信息流

> 原文：[`www.fuzzingbook.org/html/InformationFlow.html`](http://www.fuzzingbook.org/html/InformationFlow.html)

我们已经探讨了如何生成更好的输入，这些输入可以深入到所讨论的程序中。在这样做的时候，我们依赖于程序崩溃来告诉我们我们已经成功地在程序中找到了问题。然而，这相当简单。如果程序的行为只是不正确，但不会导致崩溃呢？能否做得更好？

在本章中，我们深入探讨了如何在 Python 中跟踪信息流，以及这些流如何被用来确定程序是否按预期行为。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('WZi0dTvJ2Ug') 
```

**先决条件**

+   您应该已经阅读了关于覆盖的章节（Coverage.html）。

+   您应该已经阅读了关于概率模糊测试的章节（ProbabilisticGrammarFuzzer.html）。

我们首先建立我们的基础设施，以便我们可以利用之前定义的函数。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import List, Any, Optional, Union 
```

## 概述

要使用本章提供的代码（Importing.html），请编写

```py
>>> from fuzzingbook.InformationFlow import <identifier> 
```

然后利用以下功能。

本章提供了两个 Python *字符串* 包装器，允许跟踪各种属性。这包括有关输入安全属性的信息和有关输入字符串起源索引的信息。

### 跟踪字符串污染

`tstr` 对象是 Python 字符串的替代品，允许跟踪和检查 *污染* —— 即有关字符串来源的信息。例如，可以给来自第三方输入的字符串标记“LOW”的污染，这意味着它们具有低安全级别。污染信息传递给 `tstr` 对象的构造函数：

```py
>>> thello = tstr('hello', taint='LOW') 
```

`tstr` 对象与原始 Python 字符串完全兼容。例如，我们可以对其进行索引并访问子字符串：

```py
>>> thello[:4]
'hell' 
```

然而，`tstr` 对象还存储了污染信息，可以使用 `taint` 属性访问：

```py
>>> thello.taint
'LOW' 
```

污染的一个好处是它们会传播到从原始污染字符串派生出的所有字符串。确实，任何从 `tstr` 字符串操作产生的字符串片段都会产生另一个包含原始污染的 `tstr` 对象。例如：

```py
>>> thello[1:2].taint
'LOW' 
```

`tstr` 对象复制了大多数 `str` 方法，如类图所示：

<svg width="270pt" height="562pt" viewBox="0.00 0.00 269.62 562.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 558.25)"><g id="node1" class="node"><title>tstr</title> <g id="a_node1"><a xlink:href="#" xlink:title="class tstr:

字符串包装器，保存污染信息"><text text-anchor="start" x="51.12" y="-464.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">tstr</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="tstr"><g id="a_node1_1"><a xlink:href="#" xlink:title="__add__(self, *args, **kwargs):

返回 self+value。"><text text-anchor="start" x="8" y="-442.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__add__()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="__format__(self, *args, **kwargs):

返回一个格式化的字符串表示形式，如 format_spec 所描述。"><text text-anchor="start" x="8" y="-429.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__format__()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="__getitem__(self, *args, **kwargs):

返回 self[key]。"><text text-anchor="start" x="8" y="-416.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__getitem__()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="__init__(self, value: Any, taint: Any = None, **kwargs) -> None:

构造函数。

`value` 是 `tstr` 对象要从中构建的字符串值。

`taint` 是要传播到派生字符串的可选污点。"><text text-anchor="start" x="8" y="-404" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__init__()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="__mod__(self, *args, **kwargs):

返回 self%value。"><text text-anchor="start" x="8" y="-391.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__mod__()</text></a></g> <g id="a_node1_6"><a xlink:href="#" xlink:title="__mul__(self, *args, **kwargs):

返回 self*value。"><text text-anchor="start" x="8" y="-378.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__mul__()</text></a></g> <g id="a_node1_7"><a xlink:href="#" xlink:title="__new__(cls, value, *args, **kw):

创建一个 tstr() 实例。内部使用。《__new__()`</text></a></g> <g id="a_node1_8"><a xlink:href="#" xlink:title="__radd__(self, value):

返回 value + self，作为一个 `tstr` 对象"><text text-anchor="start" x="8" y="-353" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__radd__()</text></a></g> <g id="a_node1_9"><a xlink:href="#" xlink:title="__repr__(self) -> tstr:

返回一个表示形式。"><text text-anchor="start" x="8" y="-340.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__repr__()</text></a></g> <g id="a_node1_10"><a xlink:href="#" xlink:title="__rmod__(self, *args, **kwargs):

返回值%self."><text text-anchor="start" x="8" y="-327.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__rmod__()</text></a></g> <g id="a_node1_11"><a xlink:href="#" xlink:title="__rmul__(self, *args, **kwargs):

返回值*self."><text text-anchor="start" x="8" y="-314.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__rmul__()</text></a></g> <g id="a_node1_12"><a xlink:href="#" xlink:title="__str__(self) -> str:

将字符串转换为字符串"><text text-anchor="start" x="8" y="-302" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__str__()</text></a></g> <g id="a_node1_13"><a xlink:href="#" xlink:title="capitalize(self, *args, **kwargs):

返回一个首字母大写的字符串版本。

更具体地说，使第一个字符为大写，其余为小写

case."><text text-anchor="start" x="8" y="-289.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">capitalize()</text></a></g> <g id="a_node1_14"><a xlink:href="#" xlink:title="casefold(self, *args, **kwargs):

返回一个适合不区分大小写的字符串版本。"><text text-anchor="start" x="8" y="-276.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">casefold()</text></a></g> <g id="a_node1_15"><a xlink:href="#" xlink:title="center(self, *args, **kwargs):

返回一个宽度为 width 的居中字符串。

使用指定的填充字符进行填充（默认为空格）。"><text text-anchor="start" x="8" y="-263.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">center()</text></a></g> <g id="a_node1_16"><a xlink:href="#" xlink:title="clear_taint(self):

移除污点"><text text-anchor="start" x="8" y="-251" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">clear_taint()</text></a></g> <g id="a_node1_17"><a xlink:href="#" xlink:title="encode(self, *args, **kwargs):

使用为编码注册的编解码器编码字符串。

编码

要编码的编码。

errors

编码错误处理方案。

默认为 'strict'，表示编码错误会引发异常。

UnicodeEncodeError。 &nbsp;其他可能的值是 'ignore'，'replace' 和

'xmlcharrefreplace' 以及任何其他已注册的

可以处理 UnicodeEncodeErrors 的 codecs.register_error。"><text text-anchor="start" x="8" y="-238.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">encode()</text></a></g> <g id="a_node1_18"><a xlink:href="#" xlink:title="expandtabs(self, *args, **kwargs):

返回一个副本，其中所有制表符字符都使用空格展开。

如果未提供 tabsize，则假设为 8 个字符的制表符大小。"><text text-anchor="start" x="8" y="-225.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">expandtabs()</text></a></g> <g id="a_node1_19"><a xlink:href="#" xlink:title="format(self, *args, **kwargs):

S.format(*args, **kwargs) -> str

返回使用 args 和 kwargs 中的替换的格式化版本 S。

替换通过花括号('{'和'}')标识。"><text text-anchor="start" x="8" y="-212.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">format()</text></a></g> <g id="a_node1_20"><a xlink:href="#" xlink:title="format_map(self, *args, **kwargs):

S.format_map(mapping) -> str

返回使用映射中的替换的格式化版本 S。

替换通过花括号('{'和'}')标识。"><text text-anchor="start" x="8" y="-200" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">format_map()</text></a></g> <g id="a_node1_21"><a xlink:href="#" xlink:title="has_taint(self):

检查是否存在污染"><text text-anchor="start" x="8" y="-187.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">has_taint()</text></a></g> <g id="a_node1_22"><a xlink:href="#" xlink:title="join(self, *args, **kwargs):

连接任意数量的字符串。

被调用的字符串方法插入到每个给定字符串之间。

结果作为新的字符串返回。

示例: '.'.join(['ab', 'pq', 'rs']) -> 'ab.pq.rs'"><text text-anchor="start" x="8" y="-174.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">join()</text></a></g> <g id="a_node1_23"><a xlink:href="#" xlink:title="ljust(self, *args, **kwargs):

返回长度为 width 的左对齐字符串。

使用指定的填充字符进行填充（默认为空格）。"><text text-anchor="start" x="8" y="-161.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">ljust()</text></a></g> <g id="a_node1_24"><a xlink:href="#" xlink:title="lower(self, *args, **kwargs):

返回转换为小写的字符串副本。"><text text-anchor="start" x="8" y="-149" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">lower()</text></a></g> <g id="a_node1_25"><a xlink:href="#" xlink:title="lstrip(self, *args, **kwargs):

返回移除前导空白的字符串副本。

如果提供了 chars 并且不为 None，则移除 chars 中的字符。"><text text-anchor="start" x="8" y="-136.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">lstrip()</text></a></g> <g id="a_node1_26"><a xlink:href="#" xlink:title="make_str_wrapper(fun):

将 `fun`（一个 `str` 方法）作为 `tstr`"><text text-anchor="start" x="8" y="-123.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">make_str_wrapper()</text></a></g> <g id="a_node1_27"><a xlink:href="#" xlink:title="replace(self, *args, **kwargs):

返回所有子字符串 old 被新替换的副本。

计数

最大替换次数。

-1（默认值）表示替换所有出现。

如果提供了可选参数 count，则只替换前 count 次出现。

替换。"><text text-anchor="start" x="8" y="-110.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">replace()</text></a></g> <g id="a_node1_28"><a xlink:href="#" xlink:title="rjust(self, *args, **kwargs):

返回长度为宽度的右对齐字符串。

使用指定的填充字符进行填充（默认为空格）。"><text text-anchor="start" x="8" y="-98" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">rjust()</text></a></g> <g id="a_node1_29"><a xlink:href="#" xlink:title="rstrip(self, *args, **kwargs):

返回移除尾部空白的字符串副本。

如果提供了 chars 并且不是 None，则移除 chars 中的字符。"><text text-anchor="start" x="8" y="-85.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">rstrip()</text></a></g> <g id="a_node1_30"><a xlink:href="#" xlink:title="strip(self, *args, **kwargs):

返回移除前后空白的字符串副本。

如果提供了 chars 并且不是 None，则移除 chars 中的字符。"><text text-anchor="start" x="8" y="-72.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">strip()</text></a></g> <g id="a_node1_31"><a xlink:href="#" xlink:title="swapcase(self, *args, **kwargs):

将大写字母转换为小写字母，将小写字母转换为大写字母。"><text text-anchor="start" x="8" y="-59.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">swapcase()</text></a></g> <g id="a_node1_32"><a xlink:href="#" xlink:title="title(self, *args, **kwargs):

返回一个版本，其中每个单词都是标题化的大小写。

更具体地说，单词以大写字母开头，其余的

大小写字符有下划线。"><text text-anchor="start" x="8" y="-47" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">title()</text></a></g> <g id="a_node1_33"><a xlink:href="#" xlink:title="translate(self, *args, **kwargs):

使用给定的翻译表替换字符串中的每个字符。

表

翻译表，它必须是一个将 Unicode 序列号映射到

Unicode 序列号、字符串或 None。

表必须通过 __getitem__ 实现查找/索引，例如一个

字典或列表。&nbsp;如果此操作引发 LookupError，则字符将被

映射到 None 的字符将被删除。"><text text-anchor="start" x="8" y="-34.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">translate()</text></a></g> <g id="a_node1_34"><a xlink:href="#" xlink:title="upper(self, *args, **kwargs):

返回将字符串转换为上档形式的副本。"><text text-anchor="start" x="8" y="-21.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">upper()</text></a></g> <g id="a_node1_35"><a xlink:href="#" xlink:title="create(self, s)"><text text-anchor="start" x="8" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">create()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>str</title> <g id="a_node2"><a xlink:href="builtins.html" xlink:title="class str:

str(object='') -> str

str(bytes_or_buffer[, encoding[, errors]]) -> str

从给定的对象创建一个新的字符串对象。如果编码或

如果指定了错误，则对象必须暴露一个未修改的数据缓冲区

将使用给定的编码和错误处理程序进行解码。

否则，返回对象.__str__() 的结果（如果已定义）

或 repr(object)。

编码默认为 sys.getdefaultencoding()。

errors 默认为 'strict'。"><text text-anchor="start" x="53.5" y="-532.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">str</text></a></g></g> <g id="edge1" class="edge"><title>tstr->str</title></g> <g id="node3" class="node"><title>图例</title> <text text-anchor="start" x="142.38" y="-256.62" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="142.38" y="-246.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="148.38" y="-246.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="142.38" y="-236.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="148.38" y="-236.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="142.38" y="-226.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="148.38" y="-226.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="142.38" y="-217.57" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

### 跟踪角色起源

`ostr` 对象通过不仅跟踪污点，还跟踪来自输入字符串的*索引*来扩展 `tstr` 对象，这允许您精确地跟踪单个字符的来源。假设您有一个长字符串，在索引 100 处包含密码 `"joshua1234"`。然后您可以使用以下方式使用 `ostr` 保存此起源信息：

```py
>>> secret = ostr("joshua1234", origin=100, taint='SECRET') 
```

`ostr` 的 `origin` 属性提供了对索引列表的访问：

```py
>>> secret.origin
[100, 101, 102, 103, 104, 105, 106, 107, 108, 109]
>>> secret.taint
'SECRET' 
```

`ostr` 对象与 Python 字符串兼容，除了字符串操作返回 `ostr` 对象（包括保存的起源和索引信息）。索引 `-1` 表示相应的字符没有提供给 `ostr()` 构造函数的起源：

```py
>>> secret_substr = (secret[0:4] + "-" + secret[6:])
>>> secret_substr.taint
'SECRET'
>>> secret_substr.origin
[100, 101, 102, 103, -1, 106, 107, 108, 109] 
```

`ostr` 对象复制了大多数 `str` 方法，如类图所示：

<svg width="252pt" height="596pt" viewBox="0.00 0.00 251.62 595.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 591.75)"><g id="node1" class="node"><title>ostr</title> <g id="a_node1"><a xlink:href="#" xlink:title="class ostr:

字符串包装器，保存污点和起源信息"><text text-anchor="start" x="41" y="-497.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">ostr</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="ostr"><g id="a_node1_1"><a xlink:href="#" xlink:title="DEFAULT_ORIGIN = 0"><text text-anchor="start" x="11" y="-474.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">DEFAULT_ORIGIN</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="UNKNOWN_ORIGIN = -1"><text text-anchor="start" x="11" y="-462" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">UNKNOWN_ORIGIN</text></a></g></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="ostr"><g id="a_node1_4"><a xlink:href="#" xlink:title="__add__(self, other):

返回 self+value."><text text-anchor="start" x="8" y="-442.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__add__()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="__getitem__(self, key):

返回 self[key]."><text text-anchor="start" x="8" y="-429.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__getitem__()</text></a></g> <g id="a_node1_6"><a xlink:href="#" xlink:title="__init__(self, value: Any, taint: Any = None, origin: Union[int, List[int], NoneType] = None, **kwargs) -> None:

构造函数。

`value` 是 `ostr` 对象要从中构造的字符串值。

`taint` 是一个（可选的）要传播到派生字符串的污点。

`origin`（可选）可以是

- 表示 `value` 中第一个字符索引的整数，或者

- 表示 `value` 中字符起源的整数列表，"><text text-anchor="start" x="8" y="-416.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__init__()</text></a></g> <g id="a_node1_7"><a xlink:href="#" xlink:title="__iter__(self):

实现 iter(self)."><text text-anchor="start" x="8" y="-404" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__iter__()</text></a></g> <g id="a_node1_8"><a xlink:href="#" xlink:title="__mod__(self, s):

返回 self%value."><text text-anchor="start" x="8" y="-391.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__mod__()</text></a></g> <g id="a_node1_9"><a xlink:href="#" xlink:title="__new__(cls, value, *args, **kw):

创建一个 ostr() 实例。内部使用。<text text-anchor="start" x="8" y="-378.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__new__()</text></a></g> <g id="a_node1_10"><a xlink:href="#" xlink:title="__repr__(self):

返回 repr(self)."><text text-anchor="start" x="8" y="-365.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__repr__()</text></a></g> <g id="a_node1_11"><a xlink:href="#" xlink:title="__rmod__(self, s):

返回 value%self."><text text-anchor="start" x="8" y="-353" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__rmod__()</text></a></g> <g id="a_node1_12"><a xlink:href="#" xlink:title="__str__(self):

返回 str(self)."><text text-anchor="start" x="8" y="-340.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__str__()</text></a></g> <g id="a_node1_13"><a xlink:href="#" xlink:title="capitalize(self):

返回字符串的大写版本。

更具体地说，使第一个字符大写，其余字符小写。

case."><text text-anchor="start" x="8" y="-327.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">capitalize()</text></a></g> <g id="a_node1_14"><a xlink:href="#" xlink:title="expandtabs(self, n=8):

返回使用空格展开所有制表符的副本。

如果未提供 tabsize，则假定制表符大小为 8 个字符。<text text-anchor="start" x="8" y="-314.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">expandtabs()</text></a></g> <g id="a_node1_15"><a xlink:href="#" xlink:title="join(self, iterable):

连接任意数量的字符串。

被调用的字符串方法插入到每个给定字符串之间。

结果以新的字符串形式返回。

示例：'.join(['ab', 'pq', 'rs']) -> 'ab.pq.rs'"><text text-anchor="start" x="8" y="-302" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">join()</text></a></g> <g id="a_node1_16"><a xlink:href="#" xlink:title="ljust(self, width, fillchar=' '):

返回一个长度为 width 的左对齐字符串。

使用指定的填充字符进行填充（默认为空格）。"><text text-anchor="start" x="8" y="-289.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">ljust()</text></a></g> <g id="a_node1_17"><a xlink:href="#" xlink:title="lower(self):

返回一个转换为小写的字符串副本。"><text text-anchor="start" x="8" y="-276.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">lower()</text></a></g> <g id="a_node1_18"><a xlink:href="#" xlink:title="lstrip(self, cl=None):

返回一个移除了前导空白的字符串副本。

如果 chars 给出且不为 None，则移除 chars 中的字符。"><text text-anchor="start" x="8" y="-263.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">lstrip()</text></a></g> <g id="a_node1_19"><a xlink:href="#" xlink:title="partition(self, sep):

使用给定的分隔符将字符串分为三部分。

这将在字符串中搜索分隔符。 &nbsp;如果找到分隔符，

返回一个包含分隔符之前部分、分隔符本身以及分隔符之后部分的 3 元组。

本身以及它之后的部分。

如果没有找到分隔符，则返回一个包含原始字符串

和两个空字符串。"><text text-anchor="start" x="8" y="-251" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">partition()</text></a></g> <g id="a_node1_20"><a xlink:href="#" xlink:title="replace(self, a, b, n=None):

返回一个副本，其中所有子字符串 old 都被 new 替换。

计数

最大替换出现次数。

-1（默认值）表示替换所有出现。

如果提供了可选参数 count，则只替换前 count 次出现。

被替换。"><text text-anchor="start" x="8" y="-238.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">replace()</text></a></g> <g id="a_node1_21"><a xlink:href="#" xlink:title="rjust(self, width, fillchar=' '):

返回一个长度为 width 的右对齐字符串。

使用指定的填充字符进行填充（默认为空格）。"><text text-anchor="start" x="8" y="-225.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">rjust()</text></a></g> <g id="a_node1_22"><a xlink:href="#" xlink:title="rpartition(self, sep):

使用给定的分隔符将字符串分为三部分。

这将在字符串的末尾开始搜索分隔符。如果

当找到分隔符时，返回一个包含分隔符之前的部分、分隔符本身以及分隔符之后的部分的 3 元组。

分隔符，分隔符本身以及它后面的部分。

如果未找到分隔符，则返回一个包含两个空字符串的 3 元组

以及原始字符串。"><text text-anchor="start" x="8" y="-212.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">rpartition()</text></a></g> <g id="a_node1_23"><a xlink:href="#" xlink:title="rsplit(self, sep=None, maxsplit=-1):

使用 sep 作为分隔符字符串返回字符串中的子字符串列表。

sep

用于分割字符串的分隔符。

当设置为 None（默认值）时，将在任何空白处分割

字符（包括 \n \r \t \f 和空格）并将丢弃

从结果中删除空字符串。

maxsplit

最大分割次数。

-1（默认值）表示无限制。

分割从字符串的末尾开始，向前进行。"><text text-anchor="start" x="8" y="-200" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">rsplit()</text></a></g> <g id="a_node1_24"><a xlink:href="#" xlink:title="rstrip(self, cl=None):

返回一个移除尾部空白的字符串副本。

如果 chars 给出且不为 None，则移除 chars 中的字符。"><text text-anchor="start" x="8" y="-187.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">rstrip()</text></a></g> <g id="a_node1_25"><a xlink:href="#" xlink:title="split(self, sep=None, maxsplit=-1):

使用 sep 作为分隔符字符串返回字符串中的子字符串列表。"><text text-anchor="start" x="8" y="-174.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">split()</text></a></g>

sep

用于分割字符串的分隔符。

当设置为 None（默认值）时，将在任何空白处分割

字符（包括 \n \r \t \f 和空格）并将丢弃

从结果中删除空字符串。

maxsplit

最大分割次数。

-1（默认值）表示无限制。

分割从字符串的前端开始，向末端进行。

注意，str.split() 主要用于有意分隔的数据。

对于包含标点的自然文本，请考虑使用

正则表达式模块。"><text text-anchor="start" x="8" y="-174.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">split()</text></a></g> <g id="a_node1_26"><a xlink:href="#" xlink:title="strip(self, cl=None):

返回一个移除前后空白的字符串副本。

如果 chars 给出且不为 None，则移除 chars 中的字符。"><text text-anchor="start" x="8" y="-161.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">strip()</text></a></g> <g id="a_node1_27"><a xlink:href="#" xlink:title="swapcase(self):

将大写字母转换为小写字母，将小写字母转换为大写字母。"><text text-anchor="start" x="8" y="-149" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">swapcase()</text></a></g> <g id="a_node1_28"><a xlink:href="#" xlink:title="title(self):

返回一个版本，其中每个单词都是首字母大写。

更具体地说，单词以大写字母开头，其余部分

大写字母有小写字母。"><text text-anchor="start" x="8" y="-136.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">title()</text></a></g> <g id="a_node1_29"><a xlink:href="#" xlink:title="upper(self):

返回将字符串转换为大写的副本。"><text text-anchor="start" x="8" y="-123.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">upper()</text></a></g> <g id="a_node1_30"><a xlink:href="#" xlink:title="x(self, i=0):

从给定的对象中提取索引/切片处的子字符串 `x()`"><text text-anchor="start" x="8" y="-110.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">x()</text></a></g> <g id="a_node1_31"><a xlink:href="#" xlink:title="__radd__(self, other)"><text text-anchor="start" x="8" y="-97" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">__radd__()</text></a></g> <g id="a_node1_32"><a xlink:href="#" xlink:title="_split_helper(self, sep, splitted)"><text text-anchor="start" x="8" y="-84.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">_split_helper()</text></a></g> <g id="a_node1_33"><a xlink:href="#" xlink:title="_split_space(self, splitted)"><text text-anchor="start" x="8" y="-71.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">_split_space()</text></a></g> <g id="a_node1_34"><a xlink:href="#" xlink:title="clear_origin(self)"><text text-anchor="start" x="8" y="-58.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">clear_origin()</text></a></g> <g id="a_node1_35"><a xlink:href="#" xlink:title="clear_taint(self)"><text text-anchor="start" x="8" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">clear_taint()</text></a></g> <g id="a_node1_36"><a xlink:href="#" xlink:title="create(self, res, origin=None)"><text text-anchor="start" x="8" y="-33.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">create()</text></a></g> <g id="a_node1_37"><a xlink:href="#" xlink:title="has_origin(self)"><text text-anchor="start" x="8" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">has_origin()</text></a></g> <g id="a_node1_38"><a xlink:href="#" xlink:title="has_taint(self)"><text text-anchor="start" x="8" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">has_taint()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>str</title> <g id="a_node2"><a xlink:href="builtins.html" xlink:title="class str:

str(object='') -> str

str(bytes_or_buffer[, encoding[, errors]]) -> str

从给定的对象创建一个新的字符串对象。如果编码或

如果指定了 errors，则对象必须公开一个数据缓冲区

将使用给定的编码和错误处理程序进行解码。

否则，返回对象.__str__()（如果已定义）的结果

或 repr(object)。

编码默认为 sys.getdefaultencoding()。

errors defaults to 'strict'."><text text-anchor="start" x="44.5" y="-565.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">str</text></a></g></g> <g id="edge1" class="edge"><title>ostr->str</title></g> <g id="node3" class="node"><title>图例</title> <text text-anchor="start" x="124.38" y="-273.38" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="124.38" y="-263.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.38" y="-263.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="124.38" y="-253.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.38" y="-253.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="124.38" y="-243.38" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.38" y="-243.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="124.38" y="-234.32" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 一个易受攻击的数据库

假设我们想在 Python 中实现一个*内存数据库*服务。这里有一个相当薄弱的尝试。我们使用以下数据集。

```py
INVENTORY = """\
1997,van,Ford,E350
2000,car,Mercury,Cougar
1999,car,Chevy,Venture\
""" 
```

```py
VEHICLES = INVENTORY.split('\n') 
```

我们的 DB 是一个 Python 类，它解析其参数并抛出下面定义的`SQLException`。

```py
class SQLException(Exception):
    pass 
```

该数据库只是一个仅通过 SQL 查询公开的 Python `dict`。

```py
class DB:
    def __init__(self, db={}):
        self.db = dict(db) 
```

### 表的表示

数据库包含表，这些表是通过`create_table()`方法调创建的。每个表数据结构是一对值。第一个值是包含列名和类型的元数据。第二个值是表中的值列表。

```py
class DB(DB):
    def create_table(self, table, defs):
        self.db[table] = (defs, []) 
```

可以使用`table()`方法调用通过名称检索表。

```py
class DB(DB):
    def table(self, t_name):
        if t_name in self.db:
            return self.db[t_name]
        raise SQLException('Table (%s) was not found' % repr(t_name)) 
```

这里是如何使用这两个示例的示例。我们用一个包含四个列：`year`、`kind`、`company`和`model`的表`inventory`填充。最初，我们的表是空的。

```py
def sample_db():
    db = DB()
    inventory_def = {'year': int, 'kind': str, 'company': str, 'model': str}
    db.create_table('inventory', inventory_def)
    return db 
```

使用`table()`，我们可以检索表定义以及其内容。

```py
db = sample_db()
db.table('inventory') 
```

```py
({'year': int, 'kind': str, 'company': str, 'model': str}, [])

```

我们还定义了`column()`函数，用于从表声明中检索列定义。

```py
class DB(DB):
    def column(self, table_decl, c_name):
        if c_name in table_decl: 
            return table_decl[c_name]
        raise SQLException('Column (%s) was not found' % repr(c_name)) 
```

```py
db = sample_db()
decl, rows = db.table('inventory')
db.column(decl, 'year') 
```

```py
int

```

### 执行 SQL 语句

`DB`的`sql()`方法执行 SQL 语句。它检查其参数，并根据要执行的 SQL 语句类型分发查询。

```py
class DB(DB):
    def do_select(self, query):
        ...

    def do_update(self, query):
        ...

    def do_insert(self, query):
        ...

    def do_delete(self, query):
        ...

    def sql(self, query):
        methods = [('select ', self.do_select),
                   ('update ', self.do_update),
                   ('insert into ', self.do_insert),
                   ('delete from', self.do_delete)]
        for key, method in methods:
            if query.startswith(key):
                return method(query[len(key):])
        raise SQLException('Unknown SQL (%s)' % query) 
```

这里是使用`DB`类的一个示例：

```py
some_db = DB()
some_db.sql('select year from inventory') 
```

然而，到目前为止，处理 SQL 语句的各个方法尚未定义。让我们在下一步中完成这项工作。

<details id="Excursion:-Implementing-SQL-Statements"><summary>实现 SQL 语句</summary>

#### 选择数据

`do_select()`方法处理 SQL `select`语句以从表中检索数据。

```py
class DB(DB):
    def do_select(self, query):
        FROM, WHERE = ' from ', ' where '
        table_start = query.find(FROM)
        if table_start < 0:
            raise SQLException('no table specified')

        where_start = query.find(WHERE)
        select = query[:table_start]

        if where_start >= 0:
            t_name = query[table_start + len(FROM):where_start]
            where = query[where_start + len(WHERE):]
        else:
            t_name = query[table_start + len(FROM):]
            where = ''
        _, table = self.table(t_name)

        if where:
            selected = self.expression_clause(table, "(%s)" % where)
            selected_rows = [hm for i, data, hm in selected if data]
        else:
            selected_rows = table

        rows = self.expression_clause(selected_rows, "(%s)" % select)
        return [data for i, data, hm in rows] 
```

`expression_clause()`方法用于两个目的：

1.  在形式`select` $x$，$y$，$z` `from` $t`中，它*评估*（并返回）在所选行上下文中的表达式$x$，$y$，$z$。

1.  如果给出了`where`子句`p`，它也会在行的上下文中评估`p`，并且只有当`p`成立时才将行包含在选择中。

为了评估像$x$，$y$，$z$或$p$这样的表达式，`expression_clause()`方法使用了 Python 的`eval()`评估函数。

```py
class DB(DB):
    def expression_clause(self, table, statement):
        selected = []
        for i, hm in enumerate(table):
            selected.append((i, self.my_eval(statement, {}, hm), hm))

        return selected 
```

如果`eval()`由于任何原因失败，我们将引发异常：

```py
class DB(DB):
    def my_eval(self, statement, g, l):
        try:
            return eval(statement, g, l)
        except Exception:
            raise SQLException('Invalid WHERE (%s)' % repr(statement)) 
```

**注意：**在这里使用`eval()`引入了一些重要的安全问题，我们将在本章后面讨论。

这是我们可以如何使用`sql()`来发出查询的方法。注意，表仍然是空的。

```py
db = sample_db()
db.sql('select year from inventory') 
```

```py
[]

```

```py
db = sample_db()
db.sql('select year from inventory where year == 2018') 
```

```py
[]

```

#### 插入数据

`do_insert()`方法处理 SQL `insert`语句。

```py
class DB(DB):
    def do_insert(self, query):
        VALUES = ' values '
        table_end = query.find('(')
        t_name = query[:table_end].strip()
        names_end = query.find(')')
        decls, table = self.table(t_name)
        names = [i.strip() for i in query[table_end + 1:names_end].split(',')]

        # verify columns exist
        for k in names:
            self.column(decls, k)

        values_start = query.find(VALUES)

        if values_start < 0:
            raise SQLException('Invalid INSERT (%s)' % repr(query))

        values = [
            i.strip() for i in query[values_start + len(VALUES) + 1:-1].split(',')
        ]

        if len(names) != len(values):
            raise SQLException(
                'names(%s) != values(%s)' % (repr(names), repr(values)))

        # dict lookups happen in C code, so we can't use that
        kvs = {}
        for k,v in zip(names, values):
            for key,kval in decls.items():
                if k == key:
                    kvs[key] = self.convert(kval, v)
        table.append(kvs) 
```

在 SQL 中，列可以以任何支持的数据类型出现。为了确保它使用最初声明的类型进行存储，我们需要将值转换为特定类型的转换能力，这由`convert()`提供。

```py
import [ast](https://docs.python.org/3/library/ast.html) 
```

```py
class DB(DB):
    def convert(self, cast, value):
        try:
            return cast(ast.literal_eval(value))
        except:
            raise SQLException('Invalid Conversion %s(%s)' % (cast, value)) 
```

这里是使用 SQL `insert`命令的一个示例：

```py
db = sample_db()
db.sql('insert into inventory (year, kind, company, model) values (1997, "van", "Ford", "E350")')
db.table('inventory') 
```

```py
({'year': int, 'kind': str, 'company': str, 'model': str},
 [{'year': 1997, 'kind': 'van', 'company': 'Ford', 'model': 'E350'}])

```

当数据库已填充时，我们还可以运行更复杂的查询：

```py
db.sql('select year + 1, kind from inventory') 
```

```py
[(1998, 'van')]

```

```py
db.sql('select year, kind from inventory where year == 1997') 
```

```py
[(1997, 'van')]

```

#### 更新数据

类似地，`do_update()`处理 SQL `update`语句。

```py
class DB(DB):
    def do_update(self, query):
        SET, WHERE = ' set ', ' where '
        table_end = query.find(SET)

        if table_end < 0:
            raise SQLException('Invalid UPDATE (%s)' % repr(query))

        set_end = table_end + 5
        t_name = query[:table_end]
        decls, table = self.table(t_name)
        names_end = query.find(WHERE)

        if names_end >= 0:
            names = query[set_end:names_end]
            where = query[names_end + len(WHERE):]
        else:
            names = query[set_end:]
            where = ''

        sets = [[i.strip() for i in name.split('=')]
                for name in names.split(',')]

        # verify columns exist
        for k, v in sets:
            self.column(decls, k)

        if where:
            selected = self.expression_clause(table, "(%s)" % where)
            updated = [hm for i, d, hm in selected if d]
        else:
            updated = table

        for hm in updated:
            for k, v in sets:
                # we can not do dict lookups because it is implemented in C.
                for key, kval in decls.items():
                    if key == k:
                        hm[key] = self.convert(kval, v)

        return "%d records were updated" % len(updated) 
```

这里是一个示例。让我们首先用值再次填充数据库：

```py
db = sample_db()
db.sql('insert into inventory (year, kind, company, model) values (1997, "van", "Ford", "E350")')
db.sql('select year from inventory') 
```

```py
[1997]

```

现在，我们可以更新内容：

```py
db.sql('update inventory set year = 1998 where year == 1997')
db.sql('select year from inventory') 
```

```py
[1998]

```

```py
db.table('inventory') 
```

```py
({'year': int, 'kind': str, 'company': str, 'model': str},
 [{'year': 1998, 'kind': 'van', 'company': 'Ford', 'model': 'E350'}])

```

#### 删除数据

最后，SQL `delete`语句由`do_delete()`处理。

```py
class DB(DB):
    def do_delete(self, query):
        WHERE = ' where '
        table_end = query.find(WHERE)
        if table_end < 0:
            raise SQLException('Invalid DELETE (%s)' % query)

        t_name = query[:table_end].strip()
        _, table = self.table(t_name)
        where = query[table_end + len(WHERE):]
        selected = self.expression_clause(table, "%s" % where)
        deleted = [i for i, d, hm in selected if d]
        for i in sorted(deleted, reverse=True):
            del table[i]

        return "%d records were deleted" % len(deleted) 
```

这里是一个示例。让我们首先用值再次填充数据库：

```py
db = sample_db()
db.sql('insert into inventory (year, kind, company, model) values (1997, "van", "Ford", "E350")')
db.sql('select year from inventory') 
```

```py
[1997]

```

现在，我们可以删除数据：

```py
db.sql('delete from inventory where company == "Ford"') 
```

```py
'1 records were deleted'

```

我们的数据库名现在为空：

```py
db.sql('select year from inventory') 
```

```py
[]

```</details>

这是我们的数据库如何被使用的方法。

```py
db = DB() 
```

我们首先在我们的数据库中创建一个包含正确数据类型的表。

```py
inventory_def = {'year': int, 'kind': str, 'company': str, 'model': str}
db.create_table('inventory', inventory_def) 
```

这里是一个简单的便利函数，用于使用我们的数据集更新表。

```py
def update_inventory(sqldb, vehicle):
    inventory_def = sqldb.db['inventory'][0]
    k, v = zip(*inventory_def.items())
    val = [repr(cast(val)) for cast, val in zip(v, vehicle.split(','))]
    sqldb.sql('insert into inventory (%s) values (%s)' % (','.join(k),
                                                          ','.join(val))) 
```

```py
for V in VEHICLES:
    update_inventory(db, V) 
```

我们的数据库名下`INVENTORY`表现在包含与`VEHICLES`相同的数据集。

```py
db.db 
```

```py
{'inventory': ({'year': int, 'kind': str, 'company': str, 'model': str},
  [{'year': 1997, 'kind': 'van', 'company': 'Ford', 'model': 'E350'},
   {'year': 2000, 'kind': 'car', 'company': 'Mercury', 'model': 'Cougar'},
   {'year': 1999, 'kind': 'car', 'company': 'Chevy', 'model': 'Venture'}])}

```

这里是一个示例选择语句。

```py
db.sql('select year,kind from inventory') 
```

```py
[(1997, 'van'), (2000, 'car'), (1999, 'car')]

```

```py
db.sql("select company,model from inventory where kind == 'car'") 
```

```py
[('Mercury', 'Cougar'), ('Chevy', 'Venture')]

```

我们可以在其上运行更新操作。

```py
db.sql("update inventory set year = 1998, company = 'Suzuki' where kind == 'van'") 
```

```py
'1 records were updated'

```

```py
db.db 
```

```py
{'inventory': ({'year': int, 'kind': str, 'company': str, 'model': str},
  [{'year': 1998, 'kind': 'van', 'company': 'Suzuki', 'model': 'E350'},
   {'year': 2000, 'kind': 'car', 'company': 'Mercury', 'model': 'Cougar'},
   {'year': 1999, 'kind': 'car', 'company': 'Chevy', 'model': 'Venture'}])}

```

它甚至可以即时进行数学运算！

```py
db.sql('select int(year)+10 from inventory') 
```

```py
[2008, 2010, 2009]

```

向我们的表中添加新行。

```py
db.sql("insert into inventory (year, kind, company, model) values (1, 'charriot', 'Rome', 'Quadriga')") 
```

```py
db.db 
```

```py
{'inventory': ({'year': int, 'kind': str, 'company': str, 'model': str},
  [{'year': 1998, 'kind': 'van', 'company': 'Suzuki', 'model': 'E350'},
   {'year': 2000, 'kind': 'car', 'company': 'Mercury', 'model': 'Cougar'},
   {'year': 1999, 'kind': 'car', 'company': 'Chevy', 'model': 'Venture'},
   {'year': 1, 'kind': 'charriot', 'company': 'Rome', 'model': 'Quadriga'}])}

```

我们随后将其删除。

```py
db.sql("delete from inventory where year < 1900") 
```

```py
'1 records were deleted'

```

### 模糊测试 SQL

为了验证一切是否正常，让我们进行模糊测试。首先，我们定义我们的语法。

<details id="Excursion:-Defining-a-SQL-grammar"><summary>定义 SQL 语法</summary>

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
from Grammars import START_SYMBOL, Grammar, Expansion, \
    is_valid_grammar, extend_grammar 
```

```py
EXPR_GRAMMAR: Grammar = {
    "<start>": ["<expr>"],
    "<expr>": ["<bexpr>", "<aexpr>", "(<expr>)", "<term>"],
    "<bexpr>": [
        "<aexpr><lt><aexpr>",
        "<aexpr><gt><aexpr>",
        "<expr>==<expr>",
        "<expr>!=<expr>",
    ],
    "<aexpr>": [
        "<aexpr>+<aexpr>", "<aexpr>-<aexpr>", "<aexpr>*<aexpr>",
        "<aexpr>/<aexpr>", "<word>(<exprs>)", "<expr>"
    ],
    "<exprs>": ["<expr>,<exprs>", "<expr>"],
    "<lt>": ["<"],
    "<gt>": [">"],
    "<term>": ["<number>", "<word>"],
    "<number>": ["<integer>.<integer>", "<integer>", "-<number>"],
    "<integer>": ["<digit><integer>", "<digit>"],
    "<word>": ["<word><letter>", "<word><digit>", "<letter>"],
    "<digit>":
    list(string.digits),
    "<letter>":
    list(string.ascii_letters + '_:.')
}

assert is_valid_grammar(EXPR_GRAMMAR) 
```

```py
PRINTABLE_CHARS: List[str] = [i for i in string.printable 
                                    if i not in "<>'\"\t\n\r\x0b\x0c\x00"] + ['<lt>', '<gt>'] 
```

```py
INVENTORY_GRAMMAR = extend_grammar(EXPR_GRAMMAR,
    {
        '<start>': ['<query>'],
        '<query>': [
            'select <exprs> from <table>',
            'select <exprs> from <table> where <bexpr>',
            'insert into <table> (<names>) values (<literals>)',
            'update <table> set <assignments> where <bexpr>',
            'delete from <table> where <bexpr>',
        ],
        '<table>': ['<word>'],
        '<names>': ['<column>,<names>', '<column>'],
        '<column>': ['<word>'],
        '<literals>': ['<literal>', '<literal>,<literals>'],
        '<literal>': ['<number>', "'<chars>'"],
        '<assignments>': ['<kvp>,<assignments>', '<kvp>'],
        '<kvp>': ['<column>=<value>'],
        '<value>': ['<word>'],
        '<chars>': ['<char>', '<char><chars>'],
        '<char>': PRINTABLE_CHARS,
    })

assert is_valid_grammar(INVENTORY_GRAMMAR) 
```

如从我们数据库的源代码中可以看到，函数总是检查表名是否正确。因此，我们修改语法以选择我们的特定表，这样它就有更好的机会深入。我们将在后面的章节中看到这是如何自动完成的。

```py
INVENTORY_GRAMMAR_F = extend_grammar(INVENTORY_GRAMMAR, 
                                     {'<table>': ['inventory']}) 
```</details>

```py
from GrammarFuzzer import GrammarFuzzer 
```

```py
gf = GrammarFuzzer(INVENTORY_GRAMMAR_F)
for _ in range(10):
    query = gf.fuzz()
    print(repr(query))
    try:
        res = db.sql(query)
        print(repr(res))
    except SQLException as e:
        print("> ", e)
        pass
    except:
        traceback.print_exc()
        break
    print() 
```

```py
'select O6fo,-977091.1,-36.46 from inventory'
>  Invalid WHERE ('(O6fo,-977091.1,-36.46)')

'select g3 from inventory where -3.0!=V/g/b+Q*M*G'
>  Invalid WHERE ('(-3.0!=V/g/b+Q*M*G)')

'update inventory set z=a,x=F_,Q=K where p(M)<_*S'
>  Column ('z') was not found

'update inventory set R=L5pk where e*l*y-u>K+U(:)'
>  Column ('R') was not found

'select _/d*Q+H/d(k)<t+M-A+P from inventory'
>  Invalid WHERE ('(_/d*Q+H/d(k)<t+M-A+P)')

'select F5 from inventory'
>  Invalid WHERE ('(F5)')

'update inventory set jWh.=a6 where wcY(M)>IB7(i)'
>  Column ('jWh.') was not found

'update inventory set U=y where L(W<c,(U!=W))<V(((q)==m<F),O,l)'
>  Column ('U') was not found

'delete from inventory where M/b-O*h*E<H-W>e(Y)-P'
>  Invalid WHERE ('M/b-O*h*E<H-W>e(Y)-P')

'select ((kP(86)+b*S+J/Z/U+i(U))) from inventory'
>  Invalid WHERE ('(((kP(86)+b*S+J/Z/U+i(U))))')

```

模糊测试似乎没有触发任何崩溃。然而，崩溃是我们唯一应该担心的问题吗？

### 评估的邪恶

在我们的数据库实现中——特别是在`expression_clause()`方法中——我们使用了`eval()`来使用 Python 解释器评估表达式。这允许我们在 SQL 语句中释放 Python 表达式的全部力量。

```py
db.sql('select year from inventory where year < 2000') 
```

```py
[1998, 1999]

```

在上述查询中，子句 `year < 2000` 是在每行的 Python 上下文中使用 `expression_clause()` 评估的；因此，`year < 2000` 评估为 `True` 或 `False`。

对于正在 `select` 的表达式也是如此：

```py
db.sql('select year - 1900 if year < 2000 else year - 2000 from inventory') 
```

```py
[98, 0, 99]

```

这之所以有效，是因为 `year - 1900 if year < 2000 else year - 2000` 是一个有效的 Python 表达式。（尽管它不是一个有效的 SQL 表达式。）

上面的问题是，Python 表达式没有 *限制*。如果用户尝试以下操作会怎样？

```py
db.sql('select __import__("os").popen("pwd").read() from inventory') 
```

```py
['/Users/zeller/Projects/fuzzingbook/notebooks\n',
 '/Users/zeller/Projects/fuzzingbook/notebooks\n',
 '/Users/zeller/Projects/fuzzingbook/notebooks\n']

```

上述语句实际上是从用户的文件系统中读取的。而不是 `os.popen("pwd").read()`，它可以执行任意的 Python 命令——访问数据、安装软件、运行后台进程。这就是“Python 表达式的全部力量”反过来对我们产生作用的地方。

我们希望的是让我们的 *程序* 充分利用其功能；然而，*用户*（或任何第三方）不应被委托去做同样的事情。因此，我们需要区分（受信任的）*来自程序的输入* 和（不受信任的）*来自用户的输入*。

一种允许这种区分的方法是 *动态污染分析*。其想法是识别接受用户输入作为 *源* 的函数，这些函数 *污染* 通过它们进入的任何字符串，以及执行危险操作的函数作为 *汇*。最后，我们将某些函数祝福为 *污染净化器*。其想法是，来自源头的输入在未经净化之前不应到达汇点。这允许我们使用比简单地检查崩溃更强的预言者。

## 跟踪字符串污染

可以执行各种级别的污染跟踪。最简单的是跟踪一个字符串片段起源于特定的环境，并且没有经过污染去除过程。为此，我们只需使用 `tstr` 将原始字符串包装在环境标识符（即 *污染*）中，并在每次操作产生另一个字符串片段时生成 `tstr` 实例。属性 `taint` 持有一个标识该实例是从哪个环境中派生的标签。

### 污染字符串类

为了捕获信息流，我们需要一个新的字符串类。这个想法是使用新的受污染字符串类 `tstr` 作为原始 `str` 类的包装器。然而，`str` 是一个 *不可变* 类。因此，在构造后不会调用它的 `__init__()` 方法。这意味着 `str` 的任何子类也不会调用 `__init__()` 方法。如果我们想调用我们的初始化例程，我们需要 [挂钩到 `__new__()`](https://docs.python.org/3/reference/datamodel.html#basic-customization) 并返回我们自己的类的实例。我们将此与我们的初始化代码结合在 `__init__()` 中。

```py
class tstr(str):
  """Wrapper for strings, saving taint information"""

    def __new__(cls, value, *args, **kw):
  """Create a tstr() instance. Used internally."""
        return str.__new__(cls, value)

    def __init__(self, value: Any, taint: Any = None, **kwargs) -> None:
  """Constructor.
 `value` is the string value the `tstr` object is to be constructed from.
 `taint` is an (optional) taint to be propagated to derived strings."""
        self.taint: Any = taint 
```

```py
class tstr(tstr):
    def __repr__(self) -> tstr:
  """Return a representation."""
        return tstr(str.__repr__(self), taint=self.taint) 
```

```py
class tstr(tstr):
    def __str__(self) -> str:
  """Convert to string"""
        return str.__str__(self) 
```

例如，如果我们用 `tstr` 包装 `"hello"`，那么我们应该能够访问它的污染：

```py
thello: tstr = tstr('hello', taint='LOW') 
```

```py
thello.taint 
```

```py
'LOW'

```

```py
repr(thello).taint 
```

```py
'LOW'

```

默认情况下，当我们包装一个字符串时，它是被污点化的。因此，我们还需要一种清除字符串中污点的方法。一种方法是在上面简单地返回一个`str`实例。然而，有时人们可能希望从现有的实例中移除污点。这是通过`clear_taint()`完成的。在`clear_taint()`期间，我们只需将污点设置为`None`。这个方法附带一个配对的方法`has_taint()`，它检查一个`tstr`实例是否有污点。

```py
class tstr(tstr):
    def clear_taint(self):
  """Remove taint"""
        self.taint = None
        return self

    def has_taint(self):
  """Check if taint is present"""
        return self.taint is not None 
```

### 字符串操作符

为了传播污点，我们必须扩展字符串函数，例如操作符。我们可以通过一个单一的步骤来完成，重载所有字符串方法和操作符。

当我们从现有的污点字符串创建一个新的字符串时，我们将传播其污点。

```py
class tstr(tstr):
    def create(self, s):
        return tstr(s, taint=self.taint) 
```

`make_str_wrapper()`函数创建了一个现有字符串方法的包装器，将污点附加到方法的结果上：

```py
class tstr(tstr):
    @staticmethod
    def make_str_wrapper(fun):
  """Make `fun` (a `str` method) a method in `tstr`"""
        def proxy(self, *args, **kwargs):
            res = fun(self, *args, **kwargs)
            return self.create(res)

        if hasattr(fun, '__doc__'):
            # Copy docstring
            proxy.__doc__ = fun.__doc__

        return proxy 
```

我们为所有返回字符串的字符串方法都这样做：

```py
def informationflow_init_1():
    for name in ['__format__', '__mod__', '__rmod__', '__getitem__',
                 '__add__', '__mul__', '__rmul__',
                 'capitalize', 'casefold', 'center', 'encode',
                 'expandtabs', 'format', 'format_map', 'join',
                 'ljust', 'lower', 'lstrip', 'replace',
                 'rjust', 'rstrip', 'strip', 'swapcase', 'title', 'translate', 'upper']:
        fun = getattr(str, name)
        setattr(tstr, name, tstr.make_str_wrapper(fun)) 
```

```py
informationflow_init_1() 
```

```py
INITIALIZER_LIST = [informationflow_init_1] 
```

```py
def initialize():
    for fn in INITIALIZER_LIST:
        fn() 
```

唯一缺少的操作符是左侧为常规字符串，右侧为污点字符串的`+`。Python 支持一个`__radd__()`方法，当关联的对象在加法运算的右侧使用时会被调用。

```py
class tstr(tstr):
    def __radd__(self, value):
  """Return value + self, as a `tstr` object"""
        return self.create(value + str(self)) 
```

这样，我们就已经完成了。让我们创建一个带有污点`LOW`的字符串`thello`。

```py
thello = tstr('hello', taint='LOW') 
```

现在，任何子字符串也将被污点化：

```py
thello[0].taint 
```

```py
'LOW'

```

```py
thello[1:3].taint 
```

```py
'LOW'

```

字符串加法将返回一个带有污点的`tstr`对象：

```py
(tstr('foo', taint='HIGH') + 'bar').taint 
```

```py
'HIGH'

```

我们的`__radd__()`方法确保如果`tstr`出现在字符串加法的右侧，它也能正常工作：

```py
('foo' + tstr('bar', taint='HIGH')).taint 
```

```py
'HIGH'

```

```py
thello += ', world' 
```

```py
thello.taint 
```

```py
'LOW'

```

其他操作符，如乘法，也适用：

```py
(thello * 5).taint 
```

```py
'LOW'

```

```py
('hw %s' % thello).taint 
```

```py
'LOW'

```

```py
(tstr('hello %s', taint='HIGH') % 'world').taint 
```

```py
'HIGH'

```

## 跟踪不受信任的输入

那么，人们可以用污点字符串做什么呢？我们重新考虑`DB`示例。我们定义一个“更好”的`TrustedDB`，它只接受带有污点`"TRUSTED"`的字符串。

```py
class TrustedDB(DB):
    def sql(self, s):
        assert isinstance(s, tstr), "Need a tainted string"
        assert s.taint == 'TRUSTED', "Need a string with trusted taint"
        return super().sql(s) 
```

如果向字符串提供一个“未知”（即不存在的）信任级别，将导致`TrustedDB`失败：

```py
bdb = TrustedDB(db.db) 
```

```py
from ExpectError import ExpectError 
```

```py
with ExpectError():
    bdb.sql("select year from INVENTORY") 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/3935989889.py", line 2, in <module>
    bdb.sql("select year from INVENTORY")
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/995123203.py", line 3, in sql
    assert isinstance(s, tstr), "Need a tainted string"
           ^^^^^^^^^^^^^^^^^^^
AssertionError: Need a tainted string (expected)

```

此外，任何用户输入最初都会被标记为带有污点`"UNTRUSTED"`。如果我们将一个不受信任的字符串放入我们的更好计算器中，它也会失败：

```py
bad_user_input = tstr('__import__("os").popen("ls").read()', taint='UNTRUSTED')
with ExpectError():
    bdb.sql(bad_user_input) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/3307042773.py", line 3, in <module>
    bdb.sql(bad_user_input)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/995123203.py", line 4, in sql
    assert s.taint == 'TRUSTED', "Need a string with trusted taint"
           ^^^^^^^^^^^^^^^^^^^^
AssertionError: Need a string with trusted taint (expected)

```

因此，在计算过程中，我们必须将“不受信任”的输入转换为“受信任”的字符串。这个过程称为*净化*。为了我们的目的，一个简单的净化函数可以确保输入只包含少量允许的字符（不包括字母或引号）；如果是这种情况，则输入获得一个新的`"TRUSTED"`污点。如果不是，我们将字符串转换为（不受信任的）空字符串；其他替代方案可以是引发错误或转义或删除“不受信任”的字符。

```py
import [re](https://docs.python.org/3/library/re.html) 
```

```py
def sanitize(user_input):
    assert isinstance(user_input, tstr)
    if re.match(
            r'^select +[-a-zA-Z0-9_, ()]+ from +[-a-zA-Z0-9_, ()]+$', user_input):
        return tstr(user_input, taint='TRUSTED')
    else:
        return tstr('', taint='UNTRUSTED') 
```

```py
good_user_input = tstr("select year,model from inventory", taint='UNTRUSTED')
sanitized_input = sanitize(good_user_input)
sanitized_input 
```

```py
'select year,model from inventory'

```

```py
sanitized_input.taint 
```

```py
'TRUSTED'

```

```py
bdb.sql(sanitized_input) 
```

```py
[(1998, 'E350'), (2000, 'Cougar'), (1999, 'Venture')]

```

让我们现在尝试我们的不受信任输入：

```py
sanitized_input = sanitize(bad_user_input)
sanitized_input 
```

```py
''

```

```py
sanitized_input.taint 
```

```py
'UNTRUSTED'

```

```py
with ExpectError():
    bdb.sql(sanitized_input) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/249000876.py", line 2, in <module>
    bdb.sql(sanitized_input)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/995123203.py", line 4, in sql
    assert s.taint == 'TRUSTED', "Need a string with trusted taint"
           ^^^^^^^^^^^^^^^^^^^^
AssertionError: Need a string with trusted taint (expected)

```

类似地，我们可以防止网络模糊测试章节中讨论的 SQL 和代码注入。

## 污点感知模糊测试

我们还可以使用污点来*将模糊测试直接导向那些可能生成危险输入的语法规则*。这里的想法是识别出由我们的模糊测试器生成的导致不受信任执行的输入。首先，我们定义当污点值达到危险操作时要抛出的异常。

```py
class Tainted(Exception):
    def __init__(self, v):
        self.v = v

    def __str__(self):
        return 'Tainted[%s]' % self.v 
```

### 污点 DB

接下来，由于`my_eval()`是`DB`类中最危险的操作，我们定义了一个新的类`TaintedDB`，它覆盖了`my_eval()`，以便在不受信任的字符串到达这个部分时抛出异常。

```py
class TaintedDB(DB):
    def my_eval(self, statement, g, l):
        if statement.taint != 'TRUSTED':
            raise Tainted(statement)
        try:
            return eval(statement, g, l)
        except:
            raise SQLException('Invalid SQL (%s)' % repr(statement)) 
```

我们初始化一个`TaintedDB`实例

```py
tdb = TaintedDB() 
```

```py
tdb.db = db.db 
```

然后我们开始模糊测试。

```py
import [traceback](https://docs.python.org/3/library/traceback.html) 
```

```py
for _ in range(10):
    query = gf.fuzz()
    print(repr(query))
    try:
        res = tdb.sql(tstr(query, taint='UNTRUSTED'))
        print(repr(res))
    except SQLException as e:
        pass
    except Tainted as e:
        print("> ", e)
    except:
        traceback.print_exc()
        break
    print() 
```

```py
'delete from inventory where y/u-l+f/y<Y(c)/A-H*q'
>  Tainted[y/u-l+f/y<Y(c)/A-H*q]

"insert into inventory (G,Wmp,sl3hku3) values ('<','?')"

"insert into inventory (d0) values (',_G')"

'select P*Q-w/x from inventory where X<j==:==j*r-f'
>  Tainted[(X<j==:==j*r-f)]

'select a>F*i from inventory where Q/I-_+P*j>.'
>  Tainted[(Q/I-_+P*j>.)]

'select (V-i<T/g) from inventory where T/r/G<FK(m)/(i)'
>  Tainted[(T/r/G<FK(m)/(i))]

'select (((i))),_(S,_)/L-k<H(Sv,R,n,W,Y) from inventory'
>  Tainted[((((i))),_(S,_)/L-k<H(Sv,R,n,W,Y))]

'select (N==c*U/P/y),i-e/n*y,T!=w,u from inventory'
>  Tainted[((N==c*U/P/y),i-e/n*y,T!=w,u)]

'update inventory set _=B,n=v where o-p*k-J>T'

'select s from inventory where w4g4<.m(_)/_>t'
>  Tainted[(w4g4<.m(_)/_>t)]

```

可以看到，对现有表上的`insert`、`update`、`select`和`delete`语句会导致污染异常。我们现在可以专注于这些特定类型的输入。然而，这并不是我们能做的唯一事情。在后面的章节中，我们将看到如何使用字符起源来识别输入中特定部分的污染执行。但在那之前，我们探索其他污染的使用。

## 防止隐私泄露

使用污染，我们还可以确保秘密信息不会泄露。我们可以将一个特殊的污染`"SECRET"`分配给那些信息绝对不能泄露的字符串：

```py
secrets = tstr('<Plenty of secret keys>', taint='SECRET') 
```

访问`secrets`的任何子字符串都会传播污染：

```py
secrets[1:3].taint 
```

```py
'SECRET'

```

考虑模糊测试章节中的*心跳*安全漏洞，其中服务器会意外地回复不仅包括用户发送给它的输入，还包括秘密内存。如果回复只包含用户输入，则与之相关的没有污染：

```py
user_input = "hello"
reply = user_input 
```

```py
isinstance(reply, tstr) 
```

```py
False

```

然而，如果回复包含**任何**秘密部分，回复将会被污染：

```py
reply = user_input + secrets[0:5] 
```

```py
reply 
```

```py
'hello<Plen'

```

```py
reply.taint 
```

```py
'SECRET'

```

我们服务器的输出函数现在将确保返回的数据不包含任何秘密信息：

```py
def send_back(s):
    assert not isinstance(s, tstr) and not s.taint == 'SECRET'
    ... 
```

```py
with ExpectError():
    send_back(reply) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/3747050841.py", line 2, in <module>
    send_back(reply)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/3158733057.py", line 2, in send_back
    assert not isinstance(s, tstr) and not s.taint == 'SECRET'  # type: ignore
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError (expected)

```

我们的`tstr`解决方案可以帮助识别信息泄露——但远非完美。如果我们实际上从模糊测试章节中取出`heartbeat()`的实现，我们会看到**任何**回复都被标记为`SECRET`——即使那些甚至没有访问秘密内存的回复：

```py
from Fuzzer import heartbeat 
```

```py
reply = heartbeat('hello', 5, memory=secrets) 
```

```py
reply.taint 
```

```py
'SECRET'

```

为什么会这样？如果我们查看`heartbeat()`的实现，我们会看到它首先从（非秘密）回复和（秘密）内存中构建一个长字符串`memory`，然后返回`memory`中的第一个字符。

```py
# Store reply in memory
    memory = reply + memory[len(reply):] 
```

到目前为止，整个内存仍然被污染为`SECRET`，**包括**来自`reply`的非秘密部分。

我们可能可以通过将`reply`标记为`PUBLIC`来绕过这个问题——但这样，这个污染就会与`memory`的`SECRET`标签冲突。如果我们从两个不同污染的字符串中组合一个字符串会发生什么呢？

```py
thilo = tstr("High", taint='HIGH') + tstr("Low", taint='LOW') 
```

结果表明，在这种情况下，`__add__()`方法优先于`__radd__()`方法，这意味着右边的`"Low"`字符串被视为一个常规（非污染）字符串。

```py
thilo 
```

```py
'HighLow'

```

```py
thilo.taint 
```

```py
'HIGH'

```

我们可以设置`__add__()`和其他方法，以特殊处理冲突的污染。然而，这种冲突应该如何解决将高度**依赖于应用程序**：

+   如果我们使用污染来表示**隐私级别**，`SECRET`隐私应该优先于`PUBLIC`隐私。因此，任何`SECRET`污染字符串和`PUBLIC`污染字符串的组合都应该有`SECRET`污染。

+   如果我们用“污染”来表示信息的**来源**，那么一个**未信任**的来源应该比一个**信任**的来源优先。因此，任何由**未信任**污染的字符串和**信任**污染的字符串的组合都应该有一个**未信任**的污染。

当然，这样的冲突解决可以实施。但即便如此，它们也不会帮助我们区分`heartbeat()`示例中的秘密和非秘密输出数据。

## 跟踪单个字符

幸运的是，有一个更好、更通用的方法来解决上述问题。不同污染字符串组合的关键是不仅给字符串分配污染，实际上给每一点信息——在我们的案例中，是字符——分配污染。如果每个字符都有它自己的污染，新的字符组合将简单地继承这个污染的每个字符。为此，我们引入了第二点信息，称为**来源**。

通过将每个实例作为单独的实例（在动态来源研究中称为**颜色**）来区分各种未信任的来源是可能的。你将在语法挖掘章节中看到这种技术的实例。

在本节中，我们跟踪**字符级别**的来源。也就是说，给定一个由原始来源字符串的一部分生成的片段，一个人将能够知道这个片段是从输入字符串的哪个部分取出的。本质上，每个来自来源的输入字符索引都得到它自己的颜色。

更复杂的来源，如**位图来源**也是可能的，其中单个字符可能由多个来源的字符索引生成（例如，对字符串的**校验和**操作）。在本章中我们不考虑这些。

### 跟踪字符来源的类

让我们引入一个名为`ostr`的类，它像`tstr`一样为每个字符串携带一个污染，并且还携带每个字符的**来源**，这表明了它的来源。它是一个特定范围内的连续数字（默认情况下，从零开始），表示它在特定来源中的**位置**。

```py
class ostr(str):
  """Wrapper for strings, saving taint and origin information"""
    DEFAULT_ORIGIN = 0

    def __new__(cls, value, *args, **kw):
  """Create an ostr() instance. Used internally."""
        return str.__new__(cls, value)

    def __init__(self, value: Any, taint: Any = None,
                 origin: Optional[Union[int, List[int]]] = None, **kwargs) -> None:
  """Constructor.
 `value` is the string value the `ostr` object is to be constructed from.
 `taint` is an (optional) taint to be propagated to derived strings.
 `origin` (optional) is either
 - an integer denoting the index of the first character in `value`, or
 - a list of integers denoting the origins of the characters in `value`,
 """
        self.taint = taint

        if origin is None:
            origin = ostr.DEFAULT_ORIGIN
        if isinstance(origin, int):
            self.origin = list(range(origin, origin + len(self)))
        else:
            self.origin = origin
        assert len(self.origin) == len(self) 
```

正如上面的`tstr`一样，我们实现了将它们转换为（常规）Python 字符串的方法：

```py
class ostr(ostr):
    def create(self, s):
        return ostr(s, taint=self.taint, origin=self.origin) 
```

```py
class ostr(ostr):
    UNKNOWN_ORIGIN = -1

    def __repr__(self):
        # handle escaped chars
        origin = [ostr.UNKNOWN_ORIGIN]
        for s, o in zip(str(self), self.origin):
            origin.extend([o] * (len(repr(s)) - 2))

        origin.append(ostr.UNKNOWN_ORIGIN)
        return ostr(str.__repr__(self), taint=self.taint, origin=origin) 
```

```py
class ostr(ostr):
    def __str__(self):
        return str.__str__(self) 
```

默认情况下，字符来源从`0`开始：

```py
othello = ostr('hello')
assert othello.origin == [0, 1, 2, 3, 4] 
```

我们也可以指定起始来源，如下所示 -- `6..10`

```py
tworld = ostr('world', origin=6)
assert tworld.origin == [6, 7, 8, 9, 10] 
```

```py
a = ostr("hello\tworld") 
```

```py
repr(a).origin 
```

```py
[-1, 0, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9, 10, -1]

```

`str()` 返回一个没有来源或污染信息的 `str` 实例：

```py
assert type(str(othello)) == str 
```

然而，`repr()` 会保留原始字符串的来源信息：

```py
repr(othello) 
```

```py
"'hello'"

```

```py
repr(othello).origin 
```

```py
[-1, 0, 1, 2, 3, 4, -1]

```

就像污染一样，我们可以清除来源并检查一个来源是否存在：

```py
class ostr(ostr):
    def clear_taint(self):
        self.taint = None
        return self

    def has_taint(self):
        return self.taint is not None 
```

```py
class ostr(ostr):
    def clear_origin(self):
        self.origin = [self.UNKNOWN_ORIGIN] * len(self)
        return self

    def has_origin(self):
        return any(origin != self.UNKNOWN_ORIGIN for origin in self.origin) 
```

```py
othello = ostr('Hello')
assert othello.has_origin() 
```

```py
othello.clear_origin()
assert not othello.has_origin() 
```

在本节的剩余部分，我们重新实现了各种字符串方法，以便它们也能跟踪来源。如果你觉得这太麻烦，可以直接跳转到下一节，那里提供了许多使用示例。

<details id="Excursion:-Implementing-String-Methods"><summary>实现字符串方法</summary>

#### 创建

我们需要创建新的子字符串，这些子字符串被 `ostr` 对象包装。然而，我们还想允许我们的子类创建它们自己的实例。因此，我们再次提供了一个 `create()` 方法，该方法生成一个新的 `ostr` 实例。

```py
class ostr(ostr):
    def create(self, res, origin=None):
        return ostr(res, taint=self.taint, origin=origin) 
```

```py
othello = ostr('hello', taint='HIGH')
otworld = othello.create('world', origin=6) 
```

```py
otworld.origin 
```

```py
[6, 7, 8, 9, 10]

```

```py
otworld.taint 
```

```py
'HIGH'

```

```py
assert (othello.origin, otworld.origin) == (
    [0, 1, 2, 3, 4], [6, 7, 8, 9, 10]) 
```

#### 索引

在 Python 中，索引是通过 `__getitem__()` 提供的。正整数的索引很简单。然而，它有两个额外的细节。第一个是，如果索引是负数，则从字符串的末尾开始计算这么多字符，即最后一个字符之后的字符串。也就是说，最后一个字符的负索引是 `-1`

```py
class ostr(ostr):
    def __getitem__(self, key):
        res = super().__getitem__(key)
        if isinstance(key, int):
            key = len(self) + key if key < 0 else key
            return self.create(res, [self.origin[key]])
        elif isinstance(key, slice):
            return self.create(res, self.origin[key])
        else:
            assert False 
```

```py
ohello = ostr('hello', taint='HIGH')
assert (ohello[0], ohello[-1]) == ('h', 'o')
ohello[0].taint 
```

```py
'HIGH'

```

另一个细节是 `__getitem__()` 可以接受一个切片。我们将在下一节讨论这个问题。

#### 切片

Python 的 `slice` 操作符 `[n:m]` 依赖于对象是一个 `iterator`。因此，我们定义了 `__iter__()` 方法，它返回一个自定义的 `iterator`。

```py
class ostr(ostr):
    def __iter__(self):
        return ostr_iterator(self) 
```

`__iter__()` 方法需要一个支持 `iterator` 对象。`iterator` 用于保存当前迭代的状态，它通过保持对原始 `ostr` 的引用和当前迭代的索引 `_str_idx` 来实现。

```py
class ostr_iterator():
    def __init__(self, ostr):
        self._ostr = ostr
        self._str_idx = 0

    def __next__(self):
        if self._str_idx == len(self._ostr):
            raise StopIteration
        # calls ostr getitem should be ostr
        c = self._ostr[self._str_idx]
        assert isinstance(c, ostr)
        self._str_idx += 1
        return c 
```

将所有这些放在一起：

```py
thw = ostr('hello world', taint='HIGH')
thw[0:5] 
```

```py
'hello'

```

```py
assert thw[0:5].has_taint()
assert thw[0:5].has_origin() 
```

```py
thw[0:5].taint 
```

```py
'HIGH'

```

```py
thw[0:5].origin 
```

```py
[0, 1, 2, 3, 4]

```

#### 分割

```py
def make_split_wrapper(fun):
    def proxy(self, *args, **kwargs):
        lst = fun(self, *args, **kwargs)
        return [self.create(elem) for elem in lst]
    return proxy 
```

```py
for name in ['split', 'rsplit', 'splitlines']:
    fun = getattr(str, name)
    setattr(ostr, name, make_split_wrapper(fun)) 
```

```py
othello = ostr('hello world', taint='LOW')
othello == 'hello world' 
```

```py
True

```

```py
othello.split()[0].taint 
```

```py
'LOW'

```

（读者练习：处理 *分区*，即通过子字符串分割字符串）

#### 连接

如果将两个原始字符串连接在一起，可能希望将每个字符串的来源转移到结果字符串的相应部分。字符串的连接是通过重写 `__add__()` 方法来实现的。

```py
class ostr(ostr):
    def __add__(self, other):
        if isinstance(other, ostr):
            return self.create(str.__add__(self, other),
                               (self.origin + other.origin))
        else:
            return self.create(str.__add__(self, other),
                               (self.origin + [self.UNKNOWN_ORIGIN for i in other])) 
```

```py
othello = ostr("hello")
otworld = ostr("world", origin=6)
othw = othello + otworld
assert othw.origin == [0, 1, 2, 3, 4, 6, 7, 8, 9, 10] 
```

如果将 `ostr` 与 `str` 连接起来会怎样？

```py
space = "  "
th_w = othello + space + otworld
assert th_w.origin == [
    0,
    1,
    2,
    3,
    4,
    ostr.UNKNOWN_ORIGIN,
    ostr.UNKNOWN_ORIGIN,
    6,
    7,
    8,
    9,
    10] 
```

一个细节是，当添加 `ostr` 和 `str` 时，用户可能首先放置 `str`，在这种情况下，`__add__()` 方法将在 `str` 实例上调用，而不是在 `ostr` 实例上。然而，Python 提供了一个解决方案。如果在一个 `ostr` 实例上定义了 `__radd__()`，那么将调用该方法而不是 `str.__add__()`。

```py
class ostr(ostr):
    def __radd__(self, other):
        origin = other.origin if isinstance(other, ostr) else [
            self.UNKNOWN_ORIGIN for i in other]
        return self.create(str.__add__(other, self), (origin + self.origin)) 
```

我们来测试一下：

```py
shello = "hello"
otworld = ostr("world")
thw = shello + otworld
assert thw.origin == [ostr.UNKNOWN_ORIGIN] * len(shello) + [0, 1, 2, 3, 4] 
```

这些方法：`切片` 和 `连接` 足以实现其他产生字符串的方法，并且不会改变字符本身（即不改变大小写）。因此，我们接下来看看辅助方法。

#### 提取原始字符串

给定一个特定的输入索引，方法 `x()` 从 `ostr` 中提取相应的原始部分。作为一个便利，它支持 `slices` 和 `ints`。

```py
class ostr(ostr):
    class TaintException(Exception):
        pass

    def x(self, i=0):
  """Extract substring at index/slice `i`"""
        if not self.origin:
            raise origin.TaintException('Invalid request idx')
        if isinstance(i, int):
            return [self[p]
                    for p in [k for k, j in enumerate(self.origin) if j == i]]
        elif isinstance(i, slice):
            r = range(i.start or 0, i.stop or len(self), i.step or 1)
            return [self[p]
                    for p in [k for k, j in enumerate(self.origin) if j in r]] 
```

```py
thw = ostr('hello world', origin=100) 
```

```py
assert thw.x(101) == ['e'] 
```

```py
assert thw.x(slice(101, 105)) == ['e', 'l', 'l', 'o'] 
```

#### 替换

`replace()` 方法用于将字符串的一部分替换为另一个字符串。

```py
class ostr(ostr):
    def replace(self, a, b, n=None):
        old_origin = self.origin
        b_origin = b.origin if isinstance(
            b, ostr) else [self.UNKNOWN_ORIGIN] * len(b)
        mystr = str(self)
        i = 0

        while True:
            if n and i >= n:
                break
            idx = mystr.find(a)
            if idx == -1:
                break
            last = idx + len(a)
            mystr = mystr.replace(a, b, 1)
            partA, partB = old_origin[0:idx], old_origin[last:]
            old_origin = partA + b_origin + partB
            i += 1

        return self.create(mystr, old_origin) 
```

```py
my_str = ostr("aa cde aa")
res = my_str.replace('aa', 'bb')
assert res, res.origin == ('bb', 'cde', 'bb',
                           [ostr.UNKNOWN_ORIGIN, ostr.UNKNOWN_ORIGIN,
                            2, 3, 4, 5, 6,
                            ostr.UNKNOWN_ORIGIN, ostr.UNKNOWN_ORIGIN]) 
```

```py
my_str = ostr("aa cde aa")
res = my_str.replace('aa', ostr('bb', origin=100))
assert (
    res, res.origin) == (
        ('bb cde bb'), [
            100, 101, 2, 3, 4, 5, 6, 100, 101]) 
```

#### 分割

我们实际上必须重新实现分割操作，并且按空格分割与其他分割略有不同。

```py
class ostr(ostr):
    def _split_helper(self, sep, splitted):
        result_list = []
        last_idx = 0
        first_idx = 0
        sep_len = len(sep)

        for s in splitted:
            last_idx = first_idx + len(s)
            item = self[first_idx:last_idx]
            result_list.append(item)
            first_idx = last_idx + sep_len
        return result_list

    def _split_space(self, splitted):
        result_list = []
        last_idx = 0
        first_idx = 0
        sep_len = 0
        for s in splitted:
            last_idx = first_idx + len(s)
            item = self[first_idx:last_idx]
            result_list.append(item)
            v = str(self[last_idx:])
            sep_len = len(v) - len(v.lstrip(' '))
            first_idx = last_idx + sep_len
        return result_list

    def rsplit(self, sep=None, maxsplit=-1):
        splitted = super().rsplit(sep, maxsplit)
        if not sep:
            return self._split_space(splitted)
        return self._split_helper(sep, splitted)

    def split(self, sep=None, maxsplit=-1):
        splitted = super().split(sep, maxsplit)
        if not sep:
            return self._split_space(splitted)
        return self._split_helper(sep, splitted) 
```

```py
my_str = ostr('ab cdef ghij kl')
ab, cdef, ghij, kl = my_str.rsplit(sep=' ')
assert (ab.origin, cdef.origin, ghij.origin,
        kl.origin) == ([0, 1], [3, 4, 5, 6], [8, 9, 10, 11], [13, 14])

my_str = ostr('ab cdef ghij kl', origin=list(range(0, 15)))
ab, cdef, ghij, kl = my_str.rsplit(sep=' ')
assert(ab.origin, cdef.origin, kl.origin) == ([0, 1], [3, 4, 5, 6], [13, 14]) 
```

```py
my_str = ostr('ab   cdef ghij    kl', origin=100, taint='HIGH')
ab, cdef, ghij, kl = my_str.rsplit()
assert (ab.origin, cdef.origin, ghij.origin,
        kl.origin) == ([100, 101], [105, 106, 107, 108], [110, 111, 112, 113],
                       [118, 119])

my_str = ostr('ab   cdef ghij    kl', origin=list(range(0, 20)), taint='HIGH')
ab, cdef, ghij, kl = my_str.split()
assert (ab.origin, cdef.origin, kl.origin) == ([0, 1], [5, 6, 7, 8], [18, 19])
assert ab.taint == 'HIGH' 
```

#### 去除空格

```py
class ostr(ostr):
    def strip(self, cl=None):
        return self.lstrip(cl).rstrip(cl)

    def lstrip(self, cl=None):
        res = super().lstrip(cl)
        i = self.find(res)
        return self[i:]

    def rstrip(self, cl=None):
        res = super().rstrip(cl)
        return self[0:len(res)] 
```

```py
my_str1 = ostr("  abc  ")
v = my_str1.strip()
assert v, v.origin == ('abc', [2, 3, 4]) 
```

```py
my_str1 = ostr("  abc  ")
v = my_str1.lstrip()
assert (v, v.origin) == ('abc  ', [2, 3, 4, 5, 6]) 
```

```py
my_str1 = ostr("  abc  ")
v = my_str1.rstrip()
assert (v, v.origin) == ('  abc', [0, 1, 2, 3, 4]) 
```

#### 展开制表符

```py
class ostr(ostr):
    def expandtabs(self, n=8):
        parts = self.split('\t')
        res = super().expandtabs(n)
        all_parts = []
        for i, p in enumerate(parts):
            all_parts.extend(p.origin)
            if i < len(parts) - 1:
                l = len(all_parts) % n
                all_parts.extend([p.origin[-1]] * l)
        return self.create(res, all_parts) 
```

```py
my_s = str("ab\tcd")
my_ostr = ostr("ab\tcd")
v1 = my_s.expandtabs(4)
v2 = my_ostr.expandtabs(4) 
```

```py
assert str(v1) == str(v2)
assert (len(v1), repr(v2), v2.origin) == (6, "'ab  cd'", [0, 1, 1, 1, 3, 4]) 
```

```py
class ostr(ostr):
    def join(self, iterable):
        mystr = ''
        myorigin = []
        sep_origin = self.origin
        lst = list(iterable)
        for i, s in enumerate(lst):
            sorigin = s.origin if isinstance(s, ostr) else [
                self.UNKNOWN_ORIGIN] * len(s)
            myorigin.extend(sorigin)
            mystr += str(s)
            if i < len(lst) - 1:
                myorigin.extend(sep_origin)
                mystr += str(self)
        res = super().join(iterable)
        assert len(res) == len(mystr)
        return self.create(res, myorigin) 
```

```py
my_str = ostr("ab cd", origin=100)
(v1, v2), v3 = my_str.split(), 'ef'
assert (v1.origin, v2.origin) == ([100, 101], [103, 104]) 
```

```py
v4 = ostr('').join([v2, v3, v1])
assert (
    v4, v4.origin) == (
        'cdefab', [
            103, 104, ostr.UNKNOWN_ORIGIN, ostr.UNKNOWN_ORIGIN, 100, 101]) 
```

```py
my_str = ostr("ab cd", origin=100)
(v1, v2), v3 = my_str.split(), 'ef'
assert (v1.origin, v2.origin) == ([100, 101], [103, 104]) 
```

```py
v4 = ostr(',').join([v2, v3, v1])
assert (v4, v4.origin) == ('cd,ef,ab',
                           [103, 104, 0, ostr.UNKNOWN_ORIGIN, ostr.UNKNOWN_ORIGIN, 0, 100, 101]) 
```

#### 分区

```py
class ostr(ostr):
    def partition(self, sep):
        partA, sep, partB = super().partition(sep)
        return (self.create(partA, self.origin[0:len(partA)]),
                self.create(sep,
                            self.origin[len(partA):len(partA) + len(sep)]),
                self.create(partB, self.origin[len(partA) + len(sep):]))

    def rpartition(self, sep):
        partA, sep, partB = super().rpartition(sep)
        return (self.create(partA, self.origin[0:len(partA)]),
                self.create(sep,
                            self.origin[len(partA):len(partA) + len(sep)]),
                self.create(partB, self.origin[len(partA) + len(sep):])) 
```

#### 居中

```py
class ostr(ostr):
    def ljust(self, width, fillchar=' '):
        res = super().ljust(width, fillchar)
        initial = len(res) - len(self)
        if isinstance(fillchar, tstr):
            t = fillchar.x()
        else:
            t = self.UNKNOWN_ORIGIN
        return self.create(res, [t] * initial + self.origin) 
```

```py
class ostr(ostr):
    def rjust(self, width, fillchar=' '):
        res = super().rjust(width, fillchar)
        final = len(res) - len(self)
        if isinstance(fillchar, tstr):
            t = fillchar.x()
        else:
            t = self.UNKNOWN_ORIGIN
        return self.create(res, self.origin + [t] * final) 
```

#### 模块

```py
class ostr(ostr):
    def __mod__(self, s):
        # nothing else implemented for the time being
        assert isinstance(s, str)
        s_origin = s.origin if isinstance(
            s, ostr) else [self.UNKNOWN_ORIGIN] * len(s)
        i = self.find('%s')
        assert i >= 0
        res = super().__mod__(s)
        r_origin = self.origin[:]
        r_origin[i:i + 2] = s_origin
        return self.create(res, origin=r_origin) 
```

```py
class ostr(ostr):
    def __rmod__(self, s):
        # nothing else implemented for the time being
        assert isinstance(s, str)
        r_origin = s.origin if isinstance(
            s, ostr) else [self.UNKNOWN_ORIGIN] * len(s)
        i = s.find('%s')
        assert i >= 0
        res = super().__rmod__(s)
        s_origin = self.origin[:]
        r_origin[i:i + 2] = s_origin
        return self.create(res, origin=r_origin) 
```

```py
a = ostr('hello %s world', origin=100)
a 
```

```py
'hello %s world'

```

```py
(a % 'good').origin 
```

```py
[100, 101, 102, 103, 104, 105, -1, -1, -1, -1, 108, 109, 110, 111, 112, 113]

```

```py
b = 'hello %s world'
c = ostr('bad', origin=10)
(b % c).origin 
```

```py
[-1, -1, -1, -1, -1, -1, 10, 11, 12, -1, -1, -1, -1, -1, -1]

```

#### 不改变来源的字符串方法

```py
class ostr(ostr):
    def swapcase(self):
        return self.create(str(self).swapcase(), self.origin)

    def upper(self):
        return self.create(str(self).upper(), self.origin)

    def lower(self):
        return self.create(str(self).lower(), self.origin)

    def capitalize(self):
        return self.create(str(self).capitalize(), self.origin)

    def title(self):
        return self.create(str(self).title(), self.origin) 
```

```py
a = ostr('aa', origin=100).upper()
a, a.origin 
```

```py
('AA', [100, 101])

```

#### 通用包装器

这些操作不是严格必要的，但可能对调试有用。

```py
def make_basic_str_wrapper(fun):
    def proxy(*args, **kwargs):
        res = fun(*args, **kwargs)
        return res
    return proxy 
```

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
import [types](https://docs.python.org/3/library/types.html) 
```

```py
def informationflow_init_2():
    ostr_members = [name for name, fn in inspect.getmembers(ostr, callable)
                    if isinstance(fn, types.FunctionType) and fn.__qualname__.startswith('ostr')]

    for name, fn in inspect.getmembers(str, callable):
        if name not in set(['__class__', '__new__', '__str__', '__init__',
                            '__repr__', '__getattribute__']) | set(ostr_members):
            setattr(ostr, name, make_basic_str_wrapper(fn)) 
```

```py
informationflow_init_2() 
```

```py
INITIALIZER_LIST.append(informationflow_init_2) 
```

#### 尚未翻译的方法

这些方法从其他字符串生成字符串。然而，我们还没有为这些方法提供正确的实现。因此，这些方法被标记为危险，直到我们可以生成正确的翻译。

```py
def make_str_abort_wrapper(fun):
    def proxy(*args, **kwargs):
        raise ostr.TaintException(
            '%s Not implemented in `ostr`' %
            fun.__name__)
    return proxy 
```

```py
def informationflow_init_3():
    for name, fn in inspect.getmembers(str, callable):
        # Omitted 'splitlines' as this is needed for formatting output in
        # IPython/Jupyter
        if name in ['__format__', 'format_map', 'format',
                    '__mul__', '__rmul__', 'center', 'zfill', 'decode', 'encode']:
            setattr(ostr, name, make_str_abort_wrapper(fn)) 
```

```py
informationflow_init_3() 
```

```py
INITIALIZER_LIST.append(informationflow_init_3) 
```

当生成字符串操作的代理包装器可以处理大多数常见的信息流传输情况时，一些涉及字符串的操作无法被覆盖。例如，考虑以下。</details>

### 检查来源

所有这些实现后，我们现在有了完整的`ostr`字符串，我们可以轻松检查每个字符的来源。

要检查一个字符串是否来自另一个字符串，我们可以将来源转换为集合，并使用标准集合操作：

```py
s = ostr("hello", origin=100)
s[1] 
```

```py
'e'

```

```py
s[1].origin 
```

```py
[101]

```

```py
set(s[1].origin) <= set(s.origin) 
```

```py
True

```

```py
t = ostr("world", origin=200) 
```

```py
set(s.origin) <= set(t.origin) 
```

```py
False

```

```py
u = s + t + "!" 
```

```py
u.origin 
```

```py
[100, 101, 102, 103, 104, 200, 201, 202, 203, 204, -1]

```

```py
ostr.UNKNOWN_ORIGIN in u.origin 
```

```py
True

```

### 重新审视隐私泄露

让我们应用它来看看我们是否可以为检查`heartbeat()`函数与信息泄露问题找到一个令人满意的解决方案。

```py
SECRET_ORIGIN = 1000 
```

我们定义了一个“秘密”，它不能泄露：

```py
secret = ostr('<again, some super-secret input>', origin=SECRET_ORIGIN) 
```

`secret`中的每个字符的来源都以`SECRET_ORIGIN`开头：

```py
print(secret.origin) 
```

```py
[1000, 1001, 1002, 1003, 1004, 1005, 1006, 1007, 1008, 1009, 1010, 1011, 1012, 1013, 1014, 1015, 1016, 1017, 1018, 1019, 1020, 1021, 1022, 1023, 1024, 1025, 1026, 1027, 1028, 1029, 1030, 1031]

```

如果我们现在用一个给定的字符串调用`heartbeat()`，回复的来源应该全部是`UNKNOWN_ORIGIN`（来自输入），并且没有任何字符应该有`SECRET_ORIGIN`。

```py
hello_s = heartbeat('hello', 5, memory=secret)
hello_s 
```

```py
'hello'

```

```py
assert isinstance(hello_s, ostr) 
```

```py
print(hello_s.origin) 
```

```py
[-1, -1, -1, -1, -1]

```

我们可以通过制定适当的断言来验证秘密没有泄露：

```py
assert hello_s.origin == [ostr.UNKNOWN_ORIGIN] * len(hello_s) 
```

```py
assert all(origin == ostr.UNKNOWN_ORIGIN for origin in hello_s.origin) 
```

```py
assert not any(origin >= SECRET_ORIGIN for origin in hello_s.origin) 
```

所有断言都通过，再次确认没有秘密泄露。

让我们现在去利用`heartbeat()`来揭示它的秘密。由于`heartbeat()`没有改变，它仍然像以前一样脆弱：

```py
hello_s = heartbeat('hello', 32, memory=secret)
hello_s 
```

```py
'hellon, some super-secret input>'

```

现在，然而，回复*确实*包含秘密信息：

```py
assert isinstance(hello_s, ostr) 
```

```py
print(hello_s.origin) 
```

```py
[-1, -1, -1, -1, -1, 1005, 1006, 1007, 1008, 1009, 1010, 1011, 1012, 1013, 1014, 1015, 1016, 1017, 1018, 1019, 1020, 1021, 1022, 1023, 1024, 1025, 1026, 1027, 1028, 1029, 1030, 1031]

```

```py
with ExpectError():
    assert hello_s.origin == [ostr.UNKNOWN_ORIGIN] * len(hello_s) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/2698516187.py", line 2, in <module>
    assert hello_s.origin == [ostr.UNKNOWN_ORIGIN] * len(hello_s)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError (expected)

```

```py
with ExpectError():
    assert all(origin == ostr.UNKNOWN_ORIGIN for origin in hello_s.origin) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/1358366226.py", line 2, in <module>
    assert all(origin == ostr.UNKNOWN_ORIGIN for origin in hello_s.origin)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError (expected)

```

```py
with ExpectError():
    assert not any(origin >= SECRET_ORIGIN for origin in hello_s.origin) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/1577803914.py", line 2, in <module>
    assert not any(origin >= SECRET_ORIGIN for origin in hello_s.origin)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError (expected)

```

我们现在可以将这些断言集成到`heartbeat()`函数中，使其在泄露信息之前失败。此外（或者作为替代），我们也可以重新编写我们的输出函数，以便不泄露任何秘密信息。我们将这两项练习留给读者。

## 污染导向模糊测试

之前的*污点感知模糊测试*有点令人不满意，因为我们无法专注于导致危险操作的语法特定部分。我们使用`TrackingDB`的*污点导向模糊测试*来解决这个问题。

这里的想法是跟踪每个到达`eval`的字符的来源，然后跟踪回生成它的语法节点，并增加再次使用这些节点的概率。

### TrackingDB

`TrackingDB`与`TaintedDB`类似。区别在于，如果我们发现执行已经达到`my_eval`，我们只需简单地引发`Tainted`。

```py
class TrackingDB(TaintedDB):
    def my_eval(self, statement, g, l):
        if statement.origin:
            raise Tainted(statement)
        try:
            return eval(statement, g, l)
        except:
            raise SQLException('Invalid SQL (%s)' % repr(statement)) 
```

接下来，我们需要一个特别定制的模糊测试器，它保留污点。

### TaintedGrammarFuzzer

我们定义了一个`TaintedGrammarFuzzer`类，确保污点传播到推导树。这与语法模糊测试章节中的`GrammarFuzzer`类似，除了来源和污点被保留。

```py
import [random](https://docs.python.org/3/library/random.html) 
```

```py
from GrammarFuzzer import GrammarFuzzer 
```

```py
from Parser import canonical 
```

```py
class TaintedGrammarFuzzer(GrammarFuzzer):
    def __init__(self,
                 grammar,
                 start_symbol=START_SYMBOL,
                 expansion_switch=1,
                 log=False):
        self.tainted_start_symbol = ostr(
            start_symbol, origin=[1] * len(start_symbol))
        self.expansion_switch = expansion_switch
        self.log = log
        self.grammar = grammar
        self.c_grammar = canonical(grammar)
        self.init_tainted_grammar()

    def expansion_cost(self, expansion, seen=set()):
        symbols = [e for e in expansion if e in self.c_grammar]
        if len(symbols) == 0:
            return 1

        if any(s in seen for s in symbols):
            return float('inf')

        return sum(self.symbol_cost(s, seen) for s in symbols) + 1

    def fuzz_tree(self):
        tree = (self.tainted_start_symbol, [])
        nt_leaves = [tree]
        expansion_trials = 0
        while nt_leaves:
            idx = random.randint(0, len(nt_leaves) - 1)
            key, children = nt_leaves[idx]
            expansions = self.ct_grammar[key]
            if expansion_trials < self.expansion_switch:
                expansion = random.choice(expansions)
            else:
                costs = [self.expansion_cost(e) for e in expansions]
                m = min(costs)
                all_min = [i for i, c in enumerate(costs) if c == m]
                expansion = expansions[random.choice(all_min)]

            new_leaves = [(token, []) for token in expansion]
            new_nt_leaves = [e for e in new_leaves if e[0] in self.ct_grammar]
            children[:] = new_leaves
            nt_leaves[idx:idx + 1] = new_nt_leaves
            if self.log:
                print("%-40s" % (key + " -> " + str(expansion)))
            expansion_trials += 1
        return tree

    def fuzz(self):
        self.derivation_tree = self.fuzz_tree()
        return self.tree_to_string(self.derivation_tree) 
```

我们使用专门准备的污染语法进行模糊测试。我们为每个单独的定义、每个单独的规则和每个单独的标记标记一个单独的来源（我们在检查语法后选择了 10 个标记边界）。这允许我们精确跟踪语法中哪些部分参与了我们所关心的操作。

```py
class TaintedGrammarFuzzer(TaintedGrammarFuzzer):
    def init_tainted_grammar(self):
        key_increment, alt_increment, token_increment = 1000, 100, 10
        key_origin = key_increment
        self.ct_grammar = {}
        for key, val in self.c_grammar.items():
            key_origin += key_increment
            os = []
            for v in val:
                ts = []
                key_origin += alt_increment
                for t in v:
                    nt = ostr(t, origin=key_origin)
                    key_origin += token_increment
                    ts.append(nt)
                os.append(ts)
            self.ct_grammar[key] = os

        # a use tracking grammar
        self.ctp_grammar = {}
        for key, val in self.ct_grammar.items():
            self.ctp_grammar[key] = [(v, dict(use=0)) for v in val] 
```

与以前一样，我们初始化`TrackingDB`

```py
trdb = TrackingDB(db.db) 
```

最后，我们需要确保在将树转换回字符串时保留污点。为此，我们定义了`tainted_tree_to_string()`

```py
class TaintedGrammarFuzzer(TaintedGrammarFuzzer):
    def tree_to_string(self, tree):
        symbol, children, *_ = tree
        e = ostr('')
        if children:
            return e.join([self.tree_to_string(c) for c in children])
        else:
            return e if symbol in self.c_grammar else symbol 
```

我们定义了`update_grammar()`，它接受一组达到危险操作的来源和用于模糊测试的原始字符串的推导树，以更新增强语法。

```py
class TaintedGrammarFuzzer(TaintedGrammarFuzzer):
    def update_grammar(self, origin, dtree):
        def update_tree(dtree, origin):
            key, children = dtree
            if children:
                updated_children = [update_tree(c, origin) for c in children]
                corigin = set.union(
                    *[o for (key, children, o) in updated_children])
                corigin = corigin.union(set(key.origin))
                return (key, children, corigin)
            else:
                my_origin = set(key.origin).intersection(origin)
                return (key, [], my_origin)

        key, children, oset = update_tree(dtree, set(origin))
        for key, alts in self.ctp_grammar.items():
            for alt, o in alts:
                alt_origins = set([i for token in alt for i in token.origin])
                if alt_origins.intersection(oset):
                    o['use'] += 1 
```

现在，我们已经准备好进行模糊测试。

```py
def tree_type(tree):
    key, children = tree
    return (type(key), key, [tree_type(c) for c in children]) 
```

```py
tgf = TaintedGrammarFuzzer(INVENTORY_GRAMMAR_F)
x = None
for _ in range(10):
    qtree = tgf.fuzz_tree()
    query = tgf.tree_to_string(qtree)
    assert isinstance(query, ostr)
    try:
        print(repr(query))
        res = trdb.sql(query)
        print(repr(res))
    except SQLException as e:
        print(e)
    except Tainted as e:
        print(e)
        origin = e.args[0].origin
        tgf.update_grammar(origin, qtree)
    except:
        traceback.print_exc()
        break
    print() 
```

```py
'select (g!=(9)!=((:)==2==9)!=J)==-7 from inventory'
Tainted[((g!=(9)!=((:)==2==9)!=J)==-7)]

'delete from inventory where ((c)==T)!=5==(8!=Y)!=-5'
Tainted[((c)==T)!=5==(8!=Y)!=-5]

'select (((w==(((X!=------8)))))) from inventory'
Tainted[((((w==(((X!=------8)))))))]

'delete from inventory where ((.==(-3)!=(((-3))))!=(S==(((n))==Y))!=--2!=N==-----0==--0)!=(((((R))))==((v)))!=((((((------2==Q==-8!=(q)!=(((.!=2))==J)!=(1)!=(((-4!=--5==J!=(((A==.)))))!=(((((0==(P!=((R))!=(((j)))!=7))))==O==K))==(q))==--1==((H)==(t)==s!=-6==((y))==R)!=((H))!=W==--4==(P==(u)==-0)!=O==((-5==-------2!=4!=U))!=-1==((((((R!=-6))))))!=1!=Z)))==(((I)!=((S))!=(-4==s)==(7!=(A))==(s)==p==((_)!=(C))==((w)))))))'
Tainted[((.==(-3)!=(((-3))))!=(S==(((n))==Y))!=--2!=N==-----0==--0)!=(((((R))))==((v)))!=((((((------2==Q==-8!=(q)!=(((.!=2))==J)!=(1)!=(((-4!=--5==J!=(((A==.)))))!=(((((0==(P!=((R))!=(((j)))!=7))))==O==K))==(q))==--1==((H)==(t)==s!=-6==((y))==R)!=((H))!=W==--4==(P==(u)==-0)!=O==((-5==-------2!=4!=U))!=-1==((((((R!=-6))))))!=1!=Z)))==(((I)!=((S))!=(-4==s)==(7!=(A))==(s)==p==((_)!=(C))==((w)))))))]

'delete from inventory where ((2)==T!=-1)==N==(P)==((((((6==a)))))!=8)==(3)!=((---7))'
Tainted[((2)==T!=-1)==N==(P)==((((((6==a)))))!=8)==(3)!=((---7))]

'delete from inventory where o!=2==---5==3!=t'
Tainted[o!=2==---5==3!=t]

'select (2) from inventory'
Tainted[((2))]

'select _ from inventory'
Tainted[(_)]

'select L!=(((1!=(Z)==C)!=C))==(((-0==-5==Q!=((--2!=(-0)==((0))==M)==(A))!=(X)!=e==(K==((b)))!=b==9==((((l)!=-7!=4)!=s==G))!=6==((((5==(((v==(((((((a!=d))==0!=4!=(4)==--1==(h)==-8!=(9)==-4)))))!=I!=-4))==v!=(Y==b)))==(a))!=((7)))))))==((4)) from inventory'
Tainted[(L!=(((1!=(Z)==C)!=C))==(((-0==-5==Q!=((--2!=(-0)==((0))==M)==(A))!=(X)!=e==(K==((b)))!=b==9==((((l)!=-7!=4)!=s==G))!=6==((((5==(((v==(((((((a!=d))==0!=4!=(4)==--1==(h)==-8!=(9)==-4)))))!=I!=-4))==v!=(Y==b)))==(a))!=((7)))))))==((4)))]

'delete from inventory where _==(7==(9)!=(---5)==1)==-8'
Tainted[_==(7==(9)!=(---5)==1)==-8]

```

现在，我们可以检查我们的增强语法，看看每个规则被使用了多少次。

```py
tgf.ctp_grammar 
```

```py
{'<start>': [(['<query>'], {'use': 10})],
 '<expr>': [(['<bexpr>'], {'use': 8}),
  (['<aexpr>'], {'use': 8}),
  (['(', '<expr>', ')'], {'use': 8}),
  (['<term>'], {'use': 10})],
 '<bexpr>': [(['<aexpr>', '<lt>', '<aexpr>'], {'use': 0}),
  (['<aexpr>', '<gt>', '<aexpr>'], {'use': 0}),
  (['<expr>', '==', '<expr>'], {'use': 8}),
  (['<expr>', '!=', '<expr>'], {'use': 8})],
 '<aexpr>': [(['<aexpr>', '+', '<aexpr>'], {'use': 0}),
  (['<aexpr>', '-', '<aexpr>'], {'use': 0}),
  (['<aexpr>', '*', '<aexpr>'], {'use': 0}),
  (['<aexpr>', '/', '<aexpr>'], {'use': 0}),
  (['<word>', '(', '<exprs>', ')'], {'use': 0}),
  (['<expr>'], {'use': 8})],
 '<exprs>': [(['<expr>', ',', '<exprs>'], {'use': 0}),
  (['<expr>'], {'use': 5})],
 '<lt>': [(['<'], {'use': 0})],
 '<gt>': [(['>'], {'use': 0})],
 '<term>': [(['<number>'], {'use': 9}), (['<word>'], {'use': 9})],
 '<number>': [(['<integer>', '.', '<integer>'], {'use': 0}),
  (['<integer>'], {'use': 9}),
  (['-', '<number>'], {'use': 8})],
 '<integer>': [(['<digit>', '<integer>'], {'use': 0}),
  (['<digit>'], {'use': 9})],
 '<word>': [(['<word>', '<letter>'], {'use': 0}),
  (['<word>', '<digit>'], {'use': 0}),
  (['<letter>'], {'use': 9})],
 '<digit>': [(['0'], {'use': 2}),
  (['1'], {'use': 4}),
  (['2'], {'use': 6}),
  (['3'], {'use': 3}),
  (['4'], {'use': 2}),
  (['5'], {'use': 5}),
  (['6'], {'use': 3}),
  (['7'], {'use': 5}),
  (['8'], {'use': 6}),
  (['9'], {'use': 3})],
 '<letter>': [(['a'], {'use': 2}),
  (['b'], {'use': 1}),
  (['c'], {'use': 1}),
  (['d'], {'use': 1}),
  (['e'], {'use': 1}),
  (['f'], {'use': 0}),
  (['g'], {'use': 1}),
  (['h'], {'use': 1}),
  (['i'], {'use': 0}),
  (['j'], {'use': 1}),
  (['k'], {'use': 0}),
  (['l'], {'use': 1}),
  (['m'], {'use': 0}),
  (['n'], {'use': 1}),
  (['o'], {'use': 1}),
  (['p'], {'use': 1}),
  (['q'], {'use': 1}),
  (['r'], {'use': 0}),
  (['s'], {'use': 2}),
  (['t'], {'use': 2}),
  (['u'], {'use': 1}),
  (['v'], {'use': 2}),
  (['w'], {'use': 2}),
  (['x'], {'use': 0}),
  (['y'], {'use': 1}),
  (['z'], {'use': 0}),
  (['A'], {'use': 2}),
  (['B'], {'use': 0}),
  (['C'], {'use': 2}),
  (['D'], {'use': 0}),
  (['E'], {'use': 0}),
  (['F'], {'use': 0}),
  (['G'], {'use': 1}),
  (['H'], {'use': 1}),
  (['I'], {'use': 2}),
  (['J'], {'use': 2}),
  (['K'], {'use': 2}),
  (['L'], {'use': 1}),
  (['M'], {'use': 1}),
  (['N'], {'use': 2}),
  (['O'], {'use': 1}),
  (['P'], {'use': 2}),
  (['Q'], {'use': 2}),
  (['R'], {'use': 1}),
  (['S'], {'use': 1}),
  (['T'], {'use': 2}),
  (['U'], {'use': 1}),
  (['V'], {'use': 0}),
  (['W'], {'use': 1}),
  (['X'], {'use': 2}),
  (['Y'], {'use': 3}),
  (['Z'], {'use': 2}),
  (['_'], {'use': 3}),
  ([':'], {'use': 1}),
  (['.'], {'use': 1})],
 '<query>': [(['select ', '<exprs>', ' from ', '<table>'], {'use': 5}),
  (['select ', '<exprs>', ' from ', '<table>', ' where ', '<bexpr>'],
   {'use': 0}),
  (['insert into ',
    '<table>',
    ' (',
    '<names>',
    ') values (',
    '<literals>',
    ')'],
   {'use': 0}),
  (['update ', '<table>', ' set ', '<assignments>', ' where ', '<bexpr>'],
   {'use': 0}),
  (['delete from ', '<table>', ' where ', '<bexpr>'], {'use': 5})],
 '<table>': [(['inventory'], {'use': 0})],
 '<names>': [(['<column>', ',', '<names>'], {'use': 0}),
  (['<column>'], {'use': 0})],
 '<column>': [(['<word>'], {'use': 0})],
 '<literals>': [(['<literal>'], {'use': 0}),
  (['<literal>', ',', '<literals>'], {'use': 0})],
 '<literal>': [(['<number>'], {'use': 0}),
  (["'", '<chars>', "'"], {'use': 0})],
 '<assignments>': [(['<kvp>', ',', '<assignments>'], {'use': 0}),
  (['<kvp>'], {'use': 0})],
 '<kvp>': [(['<column>', '=', '<value>'], {'use': 0})],
 '<value>': [(['<word>'], {'use': 0})],
 '<chars>': [(['<char>'], {'use': 0}), (['<char>', '<chars>'], {'use': 0})],
 '<char>': [(['0'], {'use': 0}),
  (['1'], {'use': 0}),
  (['2'], {'use': 0}),
  (['3'], {'use': 0}),
  (['4'], {'use': 0}),
  (['5'], {'use': 0}),
  (['6'], {'use': 0}),
  (['7'], {'use': 0}),
  (['8'], {'use': 0}),
  (['9'], {'use': 0}),
  (['a'], {'use': 0}),
  (['b'], {'use': 0}),
  (['c'], {'use': 0}),
  (['d'], {'use': 0}),
  (['e'], {'use': 0}),
  (['f'], {'use': 0}),
  (['g'], {'use': 0}),
  (['h'], {'use': 0}),
  (['i'], {'use': 0}),
  (['j'], {'use': 0}),
  (['k'], {'use': 0}),
  (['l'], {'use': 0}),
  (['m'], {'use': 0}),
  (['n'], {'use': 0}),
  (['o'], {'use': 0}),
  (['p'], {'use': 0}),
  (['q'], {'use': 0}),
  (['r'], {'use': 0}),
  (['s'], {'use': 0}),
  (['t'], {'use': 0}),
  (['u'], {'use': 0}),
  (['v'], {'use': 0}),
  (['w'], {'use': 0}),
  (['x'], {'use': 0}),
  (['y'], {'use': 0}),
  (['z'], {'use': 0}),
  (['A'], {'use': 0}),
  (['B'], {'use': 0}),
  (['C'], {'use': 0}),
  (['D'], {'use': 0}),
  (['E'], {'use': 0}),
  (['F'], {'use': 0}),
  (['G'], {'use': 0}),
  (['H'], {'use': 0}),
  (['I'], {'use': 0}),
  (['J'], {'use': 0}),
  (['K'], {'use': 0}),
  (['L'], {'use': 0}),
  (['M'], {'use': 0}),
  (['N'], {'use': 0}),
  (['O'], {'use': 0}),
  (['P'], {'use': 0}),
  (['Q'], {'use': 0}),
  (['R'], {'use': 0}),
  (['S'], {'use': 0}),
  (['T'], {'use': 0}),
  (['U'], {'use': 0}),
  (['V'], {'use': 0}),
  (['W'], {'use': 0}),
  (['X'], {'use': 0}),
  (['Y'], {'use': 0}),
  (['Z'], {'use': 0}),
  (['!'], {'use': 0}),
  (['#'], {'use': 0}),
  (['$'], {'use': 0}),
  (['%'], {'use': 0}),
  (['&'], {'use': 0}),
  (['('], {'use': 0}),
  ([')'], {'use': 0}),
  (['*'], {'use': 0}),
  (['+'], {'use': 0}),
  ([','], {'use': 0}),
  (['-'], {'use': 0}),
  (['.'], {'use': 0}),
  (['/'], {'use': 0}),
  ([':'], {'use': 0}),
  ([';'], {'use': 0}),
  (['='], {'use': 0}),
  (['?'], {'use': 0}),
  (['@'], {'use': 0}),
  (['['], {'use': 0}),
  (['\\'], {'use': 0}),
  ([']'], {'use': 0}),
  (['^'], {'use': 0}),
  (['_'], {'use': 0}),
  (['`'], {'use': 0}),
  (['{'], {'use': 0}),
  (['|'], {'use': 0}),
  (['}'], {'use': 0}),
  (['~'], {'use': 0}),
  ([' '], {'use': 0}),
  (['<lt>'], {'use': 0}),
  (['<gt>'], {'use': 0})]}

```

从这里，我们的想法是专注于达到危险操作的规则更频繁的规则，并增加这种类型值的概率。

### 污点跟踪的局限性

虽然我们的框架可以检测信息泄露，但它绝不是完美的。有几种方式会导致污点丢失，因此信息可能仍然会泄露。

#### 转换

我们只通过*字符串*和*字符*跟踪污点和来源。如果我们将这些转换为数字（或其他数据），信息就会丢失。

以此为例，考虑这个函数，将单个字符转换为数字再转换回来：

```py
def strip_all_info(s):
    t = ""
    for c in s:
        t += chr(ord(c))
    return t 
```

```py
othello = ostr("Secret")
othello 
```

```py
'Secret'

```

```py
othello.origin 
```

```py
[0, 1, 2, 3, 4, 5]

```

污点来源不会通过数字转换传播：

```py
thello_stripped = strip_all_info(thello)
thello_stripped 
```

```py
'hello, world'

```

```py
with ExpectError():
    thello_stripped.origin 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/588526133.py", line 2, in <module>
    thello_stripped.origin
AttributeError: 'str' object has no attribute 'origin' (expected)

```

通过扩展数字以包含污点和来源，就像我们对字符串所做的那样，可以解决这个问题。然而，在某个时候，这仍然会失败，因为一旦达到 Python 库中的内部 C 函数，污点就不会传播到 C 函数中。（除非开始为这些函数实现动态污点，那就是了。）

#### 内部 C 库

如前所述，对*内部* C 库的调用不会传播污点。例如，以下代码保留了污点，

```py
hello = ostr('hello', origin=100)
world = ostr('world', origin=200)
(hello + ' ' + world).origin 
```

```py
[100, 101, 102, 103, 104, -1, 200, 201, 202, 203, 204]

```

而对`join`的调用，即使应该是等效的，也会失败。

```py
with ExpectError():
    ''.join([hello, ' ', world]).origin 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_34365/2341342688.py", line 2, in <module>
    ''.join([hello, ' ', world]).origin  # type: ignore
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AttributeError: 'str' object has no attribute 'origin' (expected)

```

#### 隐式信息流

即使一个人可以污点程序中的所有数据，仍然有方法可以破坏信息流——特别是通过将显式流转换为*隐式*流，或将数据流转换为*控制流*。以下是一个例子：

```py
def strip_all_info_again(s):
    t = ""
    for c in s:
        if c == 'a':
            t += 'a'
        elif c == 'b':
            t += 'b'
        elif c == 'c':
            t += 'c'
    ... 
```

使用这样的函数，`s`中的字符和`t`中的字符之间没有显式的数据流；然而，字符串将是相同的。这个问题在处理和操作外部输入的程序中经常发生。

#### 强制污点

转换和隐式信息流是污点信息和来源信息丢失的几种可能性之一。为了解决这个问题，最好的解决方案是*始终假设未受污染的字符串最坏的情况*：

+   当谈到信任时，一个未受污染的字符串应被视为*可能不受信任*，因此除非经过清理，否则不应依赖。

+   当谈到隐私时，一个未受污染的字符串应被视为*可能机密*，因此不应泄露。

因此，你的程序应该始终有两种类型的污点：一种用于显式信任（或秘密）的，另一种用于显式未信任（或非秘密）的。如果在过程中污点丢失，你可能需要从其来源恢复它——这与上面讨论的字符串方法类似。好处是创建一个可信的应用程序，其中每个信息流都可以在运行时进行检查，违规行为可以通过自动化测试迅速发现。

## 经验教训

+   基于字符串和字符的污点允许动态跟踪从输入到系统内部以及返回输出的信息流。

+   检查污点可以揭示运行时未授权的输入和信息泄露。

+   数据转换和隐式数据流可能会去除污点信息；结果的无污点字符串应被视为具有最糟糕的污点。

+   污点可以与模糊测试结合使用，提供比仅依赖程序崩溃更健壮的不正确行为指示。

## 下一步

相比于我们的污点导向模糊测试，一个更好的替代方案是利用*符号*技术，这些技术会考虑到被测试程序的语义。关于流模糊测试的章节介绍了这些符号技术，目的是为了探索信息流；随后的章节关于符号模糊测试展示了如何充分利用符号执行来覆盖代码。同样，基于搜索的模糊测试通常可以提供一个更经济的探索策略。

## 背景

在本章中，我们使用库方法对 Python 进行污点分析，这是 Conti 等人讨论的[[Conti 等人，2012](https://doi.org/10.1007/978-3-642-27937-9_15)]。

## 练习

### 练习 1：污点数字

引入一个名为`tint`（污点整数）的类，它像`tstr`一样有一个污点属性，该属性从`tint`传递到`tint`。

#### 第一部分：创建

实现`tint`类，使污点设置：

```py
x = tint(42, taint='SECRET')
assert x.taint == 'SECRET' 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/InformationFlow.ipynb#Exercises)来练习并查看解决方案。

#### 第二部分：算术表达式

确保污点在算术表达式中传递；支持加、减、乘、除运算符。

```py
y = x + 1
assert y.taint == 'SECRET' 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/InformationFlow.ipynb#Exercises)来练习并查看解决方案。

#### 第三部分：从整数传递污点到字符串

将污点整数转换为字符串（使用`repr()`）应产生污点字符串：

```py
x_s = repr(x)
assert x_s.taint == 'SECRET' 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/InformationFlow.ipynb#Exercises)来练习并查看解决方案。

#### 第四部分：从字符串传递污点到整数

将带有`taint`属性的污点对象转换为整数时，应传递该污点：

```py
password = tstr('1234', taint='NOT_EXACTLY_SECRET')
x = tint(password)
assert x == 1234
assert x.taint == 'NOT_EXACTLY_SECRET' 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/InformationFlow.ipynb#Exercises)进行练习并查看解决方案。

### 练习 2：信息流测试

生成测试以确保信息流的最大化，尽可能多地传播特定的污点。为基于搜索的测试实现一个适当的适应度函数，并让基于搜索的模糊器搜索解决方案。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/InformationFlow.ipynb#Exercises)进行练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。 [最后更改：2024-11-09 17:07:29+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/InformationFlow.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用此作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[Tracking Information Flow](https://www.fuzzingbook.org/html/InformationFlow.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[The Fuzzing Book](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/InformationFlow.html`](https://www.fuzzingbook.org/html/InformationFlow.html). Retrieved 2024-11-09 17:07:29+01:00.

```py
@incollection{fuzzingbook2024:InformationFlow,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Tracking Information Flow},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/InformationFlow.html}},
    note = {Retrieved 2024-11-09 17:07:29+01:00},
    url = {https://www.fuzzingbook.org/html/InformationFlow.html},
    urldate = {2024-11-09 17:07:29+01:00}
}

```
