# 类图

> 原文：[`www.fuzzingbook.org/html/ClassDiagram.html`](http://www.fuzzingbook.org/html/ClassDiagram.html)

这是一个简单的类图查看器。针对书籍进行了定制。

**先决条件**

+   *在此处参考早期章节作为笔记本，如下所示：* 早期章节。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

## 概述

要使用本章提供的代码（Importing.html），请编写

```py
>>> from fuzzingbook.ClassDiagram import <identifier> 
```

然后利用以下功能。

函数 `display_class_hierarchy()` 显示给定类（或类列表）的类层次结构。

+   关键字参数 `public_methods`，如果提供，是一个要由客户端使用的“公共”方法列表（默认：所有带有文档字符串的方法）。

+   关键字参数 `abstract_classes`，如果提供，是一个要显示为“抽象”的类列表（即带有草书类名）。

```py
>>> display_class_hierarchy(D_Class, abstract_classes=[A_Class]) 
```

<svg width="251pt" height="284pt" viewBox="0.00 0.00 251.38 283.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 279.5)"><g id="node1" class="node"><title>D_Class</title> <g id="a_node1"><a xlink:href="#" xlink:title="class D_Class:

从多个超类继承的子类。

附带相当长但无意义的文档。"><text text-anchor="start" x="49" y="-31.57" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">D_Class</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="D_Class"><g id="a_node1_1"><a xlink:href="#" xlink:title="foo(self) -> None:

一架二战时期的 foo 战斗机。"><text text-anchor="start" x="58.75" y="-9.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>B_Class</title> <g id="a_node2"><a xlink:href="#" xlink:title="class B_Class:

继承了一些方法的子类。"><text text-anchor="start" x="8" y="-149.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">B_Class</text> <g id="a_node2_2"><a xlink:href="#" xlink:title="B_Class"><g id="a_node2_3"><a xlink:href="#" xlink:title="VAR = 'A variable'"><text text-anchor="start" x="23.75" y="-126.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">VAR</text></a></g></a></g> <g id="a_node2_4"><a xlink:href="#" xlink:title="B_Class"><g id="a_node2_5"><a xlink:href="#" xlink:title="bar(self, qux: Any = None, bartender: int = 42) -> None:

一个 qux 走进了一家酒吧。

`bartender` 是一个可选属性。"><text text-anchor="start" x="17.75" y="-106.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">bar()</text></a></g> <g id="a_node2_6"><a xlink:href="#" xlink:title="foo(self) -> None:

一架二战时期的 foo 战斗机。"><text text-anchor="start" x="17.75" y="-93.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>D_Class->B_Class</title></g> <g id="node4" class="node"><title>C_Class</title> <g id="a_node4"><a xlink:href="#" xlink:title="class C_Class:

一个注入某些方法的类"><text text-anchor="start" x="91.38" y="-132.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">C_Class</text> <g id="a_node4_7"><a xlink:href="#" xlink:title="C_Class"><g id="a_node4_8"><a xlink:href="#" xlink:title="qux(self, arg: List[Union[int, str, NoneType]]) -> List[Union[int, str, NoneType]]"><text text-anchor="start" x="100.75" y="-109.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">qux()</text></a></g></a></g></a></g></g> <g id="edge3" class="edge"><title>D_Class->C_Class</title></g> <g id="node3" class="node"><title>A_Class</title> <g id="a_node3"><a xlink:href="#" xlink:title="class A_Class:

一个正确完成 A 任务的类。

附带较长的文档字符串。"><text text-anchor="start" x="8" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-style="italic" font-size="14.00" fill="#b03a2e">A_Class</text> <g id="a_node3_9"><a xlink:href="#" xlink:title="A_Class"><g id="a_node3_10"><a xlink:href="#" xlink:title="foo(self) -> None:

《辉煌的 Foo 的冒险》"><text text-anchor="start" x="8.75" y="-236.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g> <g id="a_node3_11"><a xlink:href="#" xlink:title="quux(self) -> None:

一个未被使用的。"><text text-anchor="start" x="8.75" y="-223.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">quux()</text></a></g> <g id="a_node3_12"><a xlink:href="#" xlink:title="second(self) -> None"><text text-anchor="start" x="8.75" y="-210" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">second()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>B_Class->A_Class</title></g> <g id="node5" class="node"><title>图例</title> <text text-anchor="start" x="124.12" y="-40.5" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="124.12" y="-30.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.12" y="-30.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="124.12" y="-20.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.12" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="124.12" y="-10.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.12" y="-10.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="124.12" y="-1.45" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 获取类层次结构

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

使用 `mro()`，我们可以访问类层次结构。我们确保避免由 `class X(X)` 创建的重复项。

```py
def class_hierarchy(cls: Type) -> List[Type]:
    superclasses = cls.mro()
    hierarchy = []
    last_superclass_name = ""

    for superclass in superclasses:
        if superclass.__name__ != last_superclass_name:
            hierarchy.append(superclass)
            last_superclass_name = superclass.__name__

    return hierarchy 
```

这里有一个例子：

```py
class A_Class:
  """A Class which does A thing right.
 Comes with a longer docstring."""

    def foo(self) -> None:
  """The Adventures of the glorious Foo"""
        pass

    def quux(self) -> None:
  """A method that is not used."""
        pass 
```

```py
class A_Class(A_Class):
    # We define another function in a separate cell.

    def second(self) -> None:
        pass 
```

```py
class B_Class(A_Class):
  """A subclass inheriting some methods."""

    VAR = "A variable"

    def foo(self) -> None:
  """A WW2 foo fighter."""
        pass

    def bar(self, qux: Any = None, bartender: int = 42) -> None:
  """A qux walks into a bar.
 `bartender` is an optional attribute."""
        pass 
```

```py
SomeType = List[Optional[Union[str, int]]] 
```

```py
class C_Class:
  """A class injecting some method"""

    def qux(self, arg: SomeType) -> SomeType:
        return arg 
```

```py
class D_Class(B_Class, C_Class):
  """A subclass inheriting from multiple superclasses.
 Comes with a fairly long, but meaningless documentation."""

    def foo(self) -> None:
        B_Class.foo(self) 
```

```py
class D_Class(D_Class):
    pass  # An incremental addiiton that should not impact D's semantics 
```

```py
class_hierarchy(D_Class) 
```

```py
[__main__.D_Class,
 __main__.B_Class,
 __main__.A_Class,
 __main__.C_Class,
 object]

```

## 获取类树

我们可以使用 `__bases__` 来获取直接基类。

```py
D_Class.__bases__ 
```

```py
(__main__.D_Class,)

```

`class_tree()` 返回一个类树，使用具有相同名称的“最低”（最专用）的类。

```py
def class_tree(cls: Type, lowest: Optional[Type] = None) -> List[Tuple[Type, List]]:
    ret = []
    for base in cls.__bases__:
        if base.__name__ == cls.__name__:
            if not lowest:
                lowest = cls
            ret += class_tree(base, lowest)
        else:
            if lowest:
                cls = lowest
            ret.append((cls, class_tree(base)))

    return ret 
```

```py
class_tree(D_Class) 
```

```py
[(__main__.D_Class, [(__main__.B_Class, [(__main__.A_Class, [])])]),
 (__main__.D_Class, [(__main__.C_Class, [])])]

```

```py
class_tree(D_Class)[0][0] 
```

```py
__main__.D_Class

```

```py
assert class_tree(D_Class)[0][0] == D_Class 
```

`class_set()` 将树扁平化为集合：

```py
def class_set(classes: Union[Type, List[Type]]) -> Set[Type]:
    if not isinstance(classes, list):
        classes = [classes]

    ret = set()

    def traverse_tree(tree: List[Tuple[Type, List]]) -> None:
        for (cls, subtrees) in tree:
            ret.add(cls)
            for subtree in subtrees:
                traverse_tree(subtrees)

    for cls in classes:
        traverse_tree(class_tree(cls))

    return ret 
```

```py
class_set(D_Class) 
```

```py
{__main__.A_Class, __main__.B_Class, __main__.C_Class, __main__.D_Class}

```

```py
assert A_Class in class_set(D_Class) 
```

```py
assert B_Class in class_set(D_Class) 
```

```py
assert C_Class in class_set(D_Class) 
```

```py
assert D_Class in class_set(D_Class) 
```

```py
class_set([B_Class, C_Class]) 
```

```py
{__main__.A_Class, __main__.B_Class, __main__.C_Class}

```

### 获取文档

```py
A_Class.__doc__ 
```

```py
A_Class.__bases__[0].__doc__ 
```

```py
'A Class which does A thing right.\n    Comes with a longer docstring.'

```

```py
A_Class.__bases__[0].__name__ 
```

```py
'A_Class'

```

```py
D_Class.foo 
```

```py
<function __main__.D_Class.foo(self) -> None>

```

```py
D_Class.foo.__doc__ 
```

```py
A_Class.foo.__doc__ 
```

```py
'The Adventures of the glorious Foo'

```

```py
def docstring(obj: Any) -> str:
    doc = inspect.getdoc(obj)
    return doc if doc else "" 
```

```py
docstring(A_Class) 
```

```py
'A Class which does A thing right.\nComes with a longer docstring.'

```

```py
docstring(D_Class.foo) 
```

```py
'A WW2 foo fighter.'

```

```py
def unknown() -> None:
    pass 
```

```py
docstring(unknown) 
```

```py
''

```

```py
import [html](https://docs.python.org/3/library/html.html) 
```

```py
import [re](https://docs.python.org/3/library/re.html) 
```

```py
def escape(text: str) -> str:
    text = html.escape(text)
    assert '<' not in text
    assert '>' not in text
    text = text.replace('{', '&#x7b;')
    text = text.replace('|', '&#x7c;')
    text = text.replace('}', '&#x7d;')
    return text 
```

```py
escape("f(foo={})") 
```

```py
'f(foo=&#x7b;&#x7d;)'

```

```py
def escape_doc(docstring: str) -> str:
    DOC_INDENT = 0
    docstring = "&#x0a;".join(
        ' ' * DOC_INDENT + escape(line).strip()
        for line in docstring.split('\n')
    )
    return docstring 
```

```py
print(escape_doc("'Hello\n {You|Me}'")) 
```

```py
&#x27;Hello&#x0a;&#x7b;You&#x7c;Me&#x7d;&#x27;

```

## 获取方法和变量

```py
inspect.getmembers(D_Class) 
```

```py
[('VAR', 'A variable'),
 ('__class__', type),
 ('__delattr__', <slot wrapper '__delattr__' of 'object' objects>),
 ('__dict__', mappingproxy({'__module__': '__main__', '__doc__': None})),
 ('__dir__', <method '__dir__' of 'object' objects>),
 ('__doc__', None),
 ('__eq__', <slot wrapper '__eq__' of 'object' objects>),
 ('__format__', <method '__format__' of 'object' objects>),
 ('__ge__', <slot wrapper '__ge__' of 'object' objects>),
 ('__getattribute__', <slot wrapper '__getattribute__' of 'object' objects>),
 ('__getstate__', <method '__getstate__' of 'object' objects>),
 ('__gt__', <slot wrapper '__gt__' of 'object' objects>),
 ('__hash__', <slot wrapper '__hash__' of 'object' objects>),
 ('__init__', <slot wrapper '__init__' of 'object' objects>),
 ('__init_subclass__', <function D_Class.__init_subclass__>),
 ('__le__', <slot wrapper '__le__' of 'object' objects>),
 ('__lt__', <slot wrapper '__lt__' of 'object' objects>),
 ('__module__', '__main__'),
 ('__ne__', <slot wrapper '__ne__' of 'object' objects>),
 ('__new__', <function object.__new__(*args, **kwargs)>),
 ('__reduce__', <method '__reduce__' of 'object' objects>),
 ('__reduce_ex__', <method '__reduce_ex__' of 'object' objects>),
 ('__repr__', <slot wrapper '__repr__' of 'object' objects>),
 ('__setattr__', <slot wrapper '__setattr__' of 'object' objects>),
 ('__sizeof__', <method '__sizeof__' of 'object' objects>),
 ('__str__', <slot wrapper '__str__' of 'object' objects>),
 ('__subclasshook__', <function D_Class.__subclasshook__>),
 ('__weakref__', <attribute '__weakref__' of 'A_Class' objects>),
 ('bar',
  <function __main__.B_Class.bar(self, qux: Any = None, bartender: int = 42) -> None>),
 ('foo', <function __main__.D_Class.foo(self) -> None>),
 ('quux', <function __main__.A_Class.quux(self) -> None>),
 ('qux',
  <function __main__.C_Class.qux(self, arg: List[Union[int, str, NoneType]]) -> List[Union[int, str, NoneType]]>),
 ('second', <function __main__.A_Class.second(self) -> None>)]

```

```py
def class_items(cls: Type, pred: Callable) -> List[Tuple[str, Any]]:
    def _class_items(cls: Type) -> List:
        all_items = inspect.getmembers(cls, pred)
        for base in cls.__bases__:
            all_items += _class_items(base)

        return all_items

    unique_items = []
    items_seen = set()
    for (name, item) in _class_items(cls):
        if name not in items_seen:
            unique_items.append((name, item))
            items_seen.add(name)

    return unique_items 
```

```py
def class_methods(cls: Type) -> List[Tuple[str, Callable]]:
    return class_items(cls, inspect.isfunction) 
```

```py
def defined_in(name: str, cls: Type) -> bool:
    if not hasattr(cls, name):
        return False

    defining_classes = []

    def search_superclasses(name: str, cls: Type) -> None:
        if not hasattr(cls, name):
            return

        for base in cls.__bases__:
            if hasattr(base, name):
                defining_classes.append(base)
                search_superclasses(name, base)

    search_superclasses(name, cls)

    if any(cls.__name__ != c.__name__ for c in defining_classes):
        return False  # Already defined in superclass

    return True 
```

```py
assert not defined_in('VAR', A_Class) 
```

```py
assert defined_in('VAR', B_Class) 
```

```py
assert not defined_in('VAR', C_Class) 
```

```py
assert not defined_in('VAR', D_Class) 
```

```py
def class_vars(cls: Type) -> List[Any]:
    def is_var(item: Any) -> bool:
        return not callable(item)

    return [item for item in class_items(cls, is_var) 
            if not item[0].startswith('__') and defined_in(item[0], cls)] 
```

```py
class_methods(D_Class) 
```

```py
[('bar',
  <function __main__.B_Class.bar(self, qux: Any = None, bartender: int = 42) -> None>),
 ('foo', <function __main__.D_Class.foo(self) -> None>),
 ('quux', <function __main__.A_Class.quux(self) -> None>),
 ('qux',
  <function __main__.C_Class.qux(self, arg: List[Union[int, str, NoneType]]) -> List[Union[int, str, NoneType]]>),
 ('second', <function __main__.A_Class.second(self) -> None>)]

```

```py
class_vars(B_Class) 
```

```py
[('VAR', 'A variable')]

```

我们只对

+   在该类中*定义*的函数

+   带有文档字符串的函数

```py
def public_class_methods(cls: Type) -> List[Tuple[str, Callable]]:
    return [(name, method) for (name, method) in class_methods(cls) 
            if method.__qualname__.startswith(cls.__name__)] 
```

```py
def doc_class_methods(cls: Type) -> List[Tuple[str, Callable]]:
    return [(name, method) for (name, method) in public_class_methods(cls) 
            if docstring(method) is not None] 
```

```py
public_class_methods(D_Class) 
```

```py
[('foo', <function __main__.D_Class.foo(self) -> None>)]

```

```py
doc_class_methods(D_Class) 
```

```py
[('foo', <function __main__.D_Class.foo(self) -> None>)]

```

```py
def overloaded_class_methods(classes: Union[Type, List[Type]]) -> Set[str]:
    all_methods: Dict[str, Set[Callable]] = {}
    for cls in class_set(classes):
        for (name, method) in class_methods(cls):
            if method.__qualname__.startswith(cls.__name__):
                all_methods.setdefault(name, set())
                all_methods[name].add(cls)

    return set(name for name in all_methods if len(all_methods[name]) >= 2) 
```

```py
overloaded_class_methods(D_Class) 
```

```py
{'foo'}

```

## 使用方法名称绘制类层次结构

```py
from [inspect](https://docs.python.org/3/library/inspect.html) import signature 
```

```py
import [warnings](https://docs.python.org/3/library/warnings.html) 
```

```py
import [os](https://docs.python.org/3/library/os.html) 
```

```py
def display_class_hierarchy(classes: Union[Type, List[Type]], *,
                            public_methods: Optional[List] = None,
                            abstract_classes: Optional[List] = None,
                            include_methods: bool = True,
                            include_class_vars: bool = True,
                            include_legend: bool = True,
                            local_defs_only: bool = True,
                            types: Dict[str, Any] = {},
                            project: str = 'fuzzingbook',
                            log: bool = False) -> Any:
  """Visualize a class hierarchy.
`classes` is a Python class (or a list of classes) to be visualized.
`public_methods`, if given, is a list of methods to be shown as "public" (bold).
 (Default: all methods with a docstring)
`abstract_classes`, if given, is a list of classes to be shown as "abstract" (cursive).
 (Default: all classes with an abstract method)
`include_methods`: if set (default), include all methods
`include_legend`: if set (default), include a legend
`local_defs_only`: if set (default), hide details of imported classes
`types`: type names with definitions, to be used in docs
 """
    from [graphviz](https://graphviz.readthedocs.io/) import Digraph

    if project == 'debuggingbook':
        CLASS_FONT = 'Raleway, Helvetica, Arial, sans-serif'
        CLASS_COLOR = '#6A0DAD'  # HTML 'purple'
    else:
        CLASS_FONT = 'Patua One, Helvetica, sans-serif'
        CLASS_COLOR = '#B03A2E'

    METHOD_FONT = "'Fira Mono', 'Source Code Pro', 'Courier', monospace"
    METHOD_COLOR = 'black'

    if isinstance(classes, list):
        starting_class = classes[0]
    else:
        starting_class = classes
        classes = [starting_class]

    title = starting_class.__name__ + " class hierarchy"

    dot = Digraph(comment=title)
    dot.attr('node', shape='record', fontname=CLASS_FONT)
    dot.attr('graph', rankdir='BT', tooltip=title)
    dot.attr('edge', arrowhead='empty')

    # Hack to force rendering as HTML, allowing hovers and links in Jupyter
    dot._repr_html_ = dot._repr_image_svg_xml

    edges = set()
    overloaded_methods: Set[str] = set()

    drawn_classes = set()

    def method_string(method_name: str, public: bool, overloaded: bool,
                      fontsize: float = 10.0) -> str:
        method_string = f'<font face="{METHOD_FONT}" point-size="{str(fontsize)}">'

        if overloaded:
            name = f'<i>{method_name}()</i>'
        else:
            name = f'{method_name}()'

        if public:
            method_string += f'<b>{name}</b>'
        else:
            method_string += f'<font color="{METHOD_COLOR}">' \
                             f'{name}</font>'

        method_string += '</font>'
        return method_string

    def var_string(var_name: str, fontsize: int = 10) -> str:
        var_string = f'<font face="{METHOD_FONT}" point-size="{str(fontsize)}">'
        var_string += f'{var_name}'
        var_string += '</font>'
        return var_string

    def is_overloaded(method_name: str, f: Any) -> bool:
        return (method_name in overloaded_methods or
                (docstring(f) is not None and "in subclasses" in docstring(f)))

    def is_abstract(cls: Type) -> bool:
        if not abstract_classes:
            return inspect.isabstract(cls)

        return (cls in abstract_classes or
                any(c.__name__ == cls.__name__ for c in abstract_classes))

    def is_public(method_name: str, f: Any) -> bool:
        if public_methods:
            return (method_name in public_methods or
                    f in public_methods or
                    any(f.__qualname__ == m.__qualname__
                        for m in public_methods))

        return bool(docstring(f))

    def frame_module(frameinfo: Any) -> str:
        return os.path.splitext(os.path.basename(frameinfo.frame.f_code.co_filename))[0]

    def callers() -> List[str]:
        frames = inspect.getouterframes(inspect.currentframe())
        return [frame_module(frameinfo) for frameinfo in frames]

    def is_local_class(cls: Type) -> bool:
        return cls.__module__ == '__main__' or cls.__module__ in callers()

    def class_vars_string(cls: Type, url: str) -> str:
        cls_vars = class_vars(cls)
        if len(cls_vars) == 0:
            return ""

        vars_string = f'<table border="0" cellpadding="0" ' \
                      f'cellspacing="0" ' \
                      f'align="left" tooltip="{cls.__name__}" href="#">'

        for (name, var) in cls_vars:
            if log:
                print(f"    Drawing {name}")

            var_doc = escape(f"{name} = {repr(var)}")
            tooltip = f' tooltip="{var_doc}"'
            href = f' href="{url}"'
            vars_string += f'<tr><td align="left" border="0"' \
                           f'{tooltip}{href}>'

            vars_string += var_string(name)
            vars_string += '</td></tr>'

        vars_string += '</table>'
        return vars_string

    def class_methods_string(cls: Type, url: str) -> str:
        methods = public_class_methods(cls)
        # return "<br/>".join([name + "()" for (name, f) in methods])
        methods_string = f'<table border="0" cellpadding="0" ' \
                         f'cellspacing="0" ' \
                         f'align="left" tooltip="{cls.__name__}" href="#">'

        public_methods_only = local_defs_only and not is_local_class(cls)

        methods_seen = False
        for public in [True, False]:
            for (name, f) in methods:
                if public != is_public(name, f):
                    continue

                if public_methods_only and not public:
                    continue

                if log:
                    print(f"    Drawing {name}()")

                if is_public(name, f) and not docstring(f):
                    warnings.warn(f"{f.__qualname__}() is listed as public,"
                                  f" but has no docstring")

                overloaded = is_overloaded(name, f)

                sig = str(inspect.signature(f))
                # replace 'List[Union[...]]' by the actual type def
                for tp in types:
                    tp_def = str(types[tp]).replace('typing.', '')
                    sig = sig.replace(tp_def, tp)
                sig = sig.replace('__main__.', '')

                method_doc = escape(name + sig)
                if docstring(f):
                    method_doc += ":&#x0a;" + escape_doc(docstring(f))

                if log:
                    print(f"    Method doc: {method_doc}")

                # Tooltips are only shown if a href is present, too
                tooltip = f' tooltip="{method_doc}"'
                href = f' href="{url}"'
                methods_string += f'<tr><td align="left" border="0"' \
                                  f'{tooltip}{href}>'

                methods_string += method_string(name, public, overloaded)

                methods_string += '</td></tr>'
                methods_seen = True

        if not methods_seen:
            return ""

        methods_string += '</table>'
        return methods_string

    def display_class_node(cls: Type) -> None:
        name = cls.__name__

        if name in drawn_classes:
            return
        drawn_classes.add(name)

        if log:
            print(f"Drawing class {name}")

        if cls.__module__ == '__main__':
            url = '#'
        else:
            url = cls.__module__ + '.ipynb'

        if is_abstract(cls):
            formatted_class_name = f'<i>{cls.__name__}</i>'
        else:
            formatted_class_name = cls.__name__

        if include_methods or include_class_vars:
            vars = class_vars_string(cls, url)
            methods = class_methods_string(cls, url)
            spec = '<{<b><font color="' + CLASS_COLOR + '">' + \
                formatted_class_name + '</font></b>'
            if include_class_vars and vars:
                spec += '|' + vars
            if include_methods and methods:
                spec += '|' + methods
            spec += '}>'
        else:
            spec = '<' + formatted_class_name + '>'

        class_doc = escape('class ' + cls.__name__)
        if docstring(cls):
            class_doc += ':&#x0a;' + escape_doc(docstring(cls))
        else:
            warnings.warn(f"Class {cls.__name__} has no docstring")

        dot.node(name, spec, tooltip=class_doc, href=url)

    def display_class_trees(trees: List[Tuple[Type, List]]) -> None:
        for tree in trees:
            (cls, subtrees) = tree
            display_class_node(cls)

            for subtree in subtrees:
                (subcls, _) = subtree

                if (cls.__name__, subcls.__name__) not in edges:
                    dot.edge(cls.__name__, subcls.__name__)
                    edges.add((cls.__name__, subcls.__name__))

            display_class_trees(subtrees)

    def display_legend() -> None:
        fontsize = 8.0

        label = f'<b><font color="{CLASS_COLOR}">Legend</font></b><br align="left"/>' 

        for item in [
            method_string("public_method",
                          public=True, overloaded=False, fontsize=fontsize),
            method_string("private_method",
                          public=False, overloaded=False, fontsize=fontsize),
            method_string("overloaded_method",
                          public=False, overloaded=True, fontsize=fontsize)
        ]:
            label += '&bull;&nbsp;' + item + '<br align="left"/>'

        label += f'<font face="Helvetica" point-size="{str(fontsize  +  1)}">' \
                 'Hover over names to see doc' \
                 '</font><br align="left"/>'

        dot.node('Legend', label=f'<{label}>', shape='plain', fontsize=str(fontsize + 2))

    for cls in classes:
        tree = class_tree(cls)
        overloaded_methods = overloaded_class_methods(cls)
        display_class_trees(tree)

    if include_legend:
        display_legend()

    return dot 
```

```py
display_class_hierarchy(D_Class, types={'SomeType': SomeType},
                        project='debuggingbook', log=True) 
```

```py
Drawing class D_Class
    Drawing foo()
    Method doc: foo(self) -&gt; None:&#x0a;A WW2 foo fighter.
Drawing class B_Class
    Drawing VAR
    Drawing bar()
    Method doc: bar(self, qux: Any = None, bartender: int = 42) -&gt; None:&#x0a;A qux walks into a bar.&#x0a;`bartender` is an optional attribute.
    Drawing foo()
    Method doc: foo(self) -&gt; None:&#x0a;A WW2 foo fighter.
Drawing class A_Class
    Drawing foo()
    Method doc: foo(self) -&gt; None:&#x0a;The Adventures of the glorious Foo
    Drawing quux()
    Method doc: quux(self) -&gt; None:&#x0a;A method that is not used.
    Drawing second()
    Method doc: second(self) -&gt; None
Drawing class C_Class
    Drawing qux()
    Method doc: qux(self, arg: SomeType) -&gt; SomeType

```

<svg width="254pt" height="282pt" viewBox="0.00 0.00 254.12 282.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 278)"><g id="node1" class="node"><title>D_Class</title> <g id="a_node1"><a xlink:href="#" xlink:title="class D_Class:

从多个超类继承的子类。

伴随相当长但无意义的文档。"><text text-anchor="start" x="50" y="-31.2" font-family="Raleway, Helvetica, Arial, sans-serif" font-weight="bold" font-size="14.00" fill="#6a0dad">D_Class</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="D_Class"><g id="a_node1_1"><a xlink:href="#" xlink:title="foo(self) -> None:

一架二战时期的 foo 战斗机。"><text text-anchor="start" x="60.5" y="-9.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>B_Class</title> <g id="a_node2"><a xlink:href="#" xlink:title="class B_Class:

继承了一些方法的子类。《B_Class》 <g id="a_node2_2"><a xlink:href="#" xlink:title="B_Class"><g id="a_node2_3"><a xlink:href="#" xlink:title="VAR = 'A variable'"><text text-anchor="start" x="24.5" y="-126.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">VAR</text></a></g></a></g> <g id="a_node2_4"><a xlink:href="#" xlink:title="B_Class"><g id="a_node2_5"><a xlink:href="#" xlink:title="bar(self, qux: Any = None, bartender: int = 42) -> None:

一个 qux 走进酒吧。

`bartender`是一个可选属性。"><text text-anchor="start" x="18.5" y="-106.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">bar()</text></a></g> <g id="a_node2_6"><a xlink:href="#" xlink:title="foo(self) -> None:

一架二战时期的 foo 战斗机。"><text text-anchor="start" x="18.5" y="-93.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>D_Class->B_Class</title></g> <g id="node4" class="node"><title>C_Class</title> <g id="a_node4"><a xlink:href="#" xlink:title="class C_Class:

注入了一些方法的类。《C_Class》 <g id="a_node4_7"><a xlink:href="#" xlink:title="C_Class"><g id="a_node4_8"><a xlink:href="#" xlink:title="qux(self, arg: SomeType) -> SomeType"><text text-anchor="start" x="103.5" y="-109.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">qux()</text></a></g></a></g></a></g></g> <g id="edge3" class="edge"><title>D_Class->C_Class</title></g> <g id="node3" class="node"><title>A_Class</title> <g id="a_node3"><a xlink:href="#" xlink:title="class A_Class:

一个正确完成 A 任务的类。

带有更长的文档字符串。"><text text-anchor="start" x="8" y="-257.2" font-family="Raleway, Helvetica, Arial, sans-serif" font-weight="bold" font-size="14.00" fill="#6a0dad">A_Class</text> <g id="a_node3_9"><a xlink:href="#" xlink:title="A_Class"><g id="a_node3_10"><a xlink:href="#" xlink:title="foo(self) -> None:

The Adventures of the glorious Foo"><text text-anchor="start" x="9.5" y="-235.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g> <g id="a_node3_11"><a xlink:href="#" xlink:title="quux(self) -> None:

未使用的方法。"><text text-anchor="start" x="9.5" y="-223" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">quux()</text></a></g> <g id="a_node3_12"><a xlink:href="#" xlink:title="second(self) -> None"><text text-anchor="start" x="9.5" y="-209.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">second()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>B_Class->A_Class</title></g> <g id="node5" class="node"><title>图例</title> <text text-anchor="start" x="126.88" y="-40.5" font-family="Raleway, Helvetica, Arial, sans-serif" font-weight="bold" font-size="10.00" fill="#6a0dad">图例</text> <text text-anchor="start" x="126.88" y="-30.5" font-family="Raleway, Helvetica, Arial, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="132.88" y="-30.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="126.88" y="-20.5" font-family="Raleway, Helvetica, Arial, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="132.88" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="126.88" y="-10.5" font-family="Raleway, Helvetica, Arial, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="132.88" y="-10.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="126.88" y="-1.45" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

```py
display_class_hierarchy(D_Class, types={'SomeType': SomeType},
                        project='fuzzingbook') 
```

<svg width="251pt" height="284pt" viewBox="0.00 0.00 251.38 283.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 279.5)"><g id="node1" class="node"><title>D_Class</title> <g id="a_node1"><a xlink:href="#" xlink:title="class D_Class:

A subclass inheriting from multiple superclasses.

Comes with a fairly long, but meaningless documentation."><text text-anchor="start" x="49" y="-31.57" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">D_Class</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="D_Class"><g id="a_node1_1"><a xlink:href="#" xlink:title="foo(self) -> None:

一架二战时期的 foo 战斗机。"><text text-anchor="start" x="58.75" y="-9.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>B_Class</title> <g id="a_node2"><a xlink:href="#" xlink:title="class B_Class:

继承一些方法的子类。《B_Class</text> <g id="a_node2_2"><a xlink:href="#" xlink:title="B_Class"><g id="a_node2_3"><a xlink:href="#" xlink:title="VAR = 'A variable'"><text text-anchor="start" x="23.75" y="-126.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">VAR</text></a></g></a></g> <g id="a_node2_4"><a xlink:href="#" xlink:title="B_Class"><g id="a_node2_5"><a xlink:href="#" xlink:title="bar(self, qux: Any = None, bartender: int = 42) -> None:

一只 qux 走进酒吧。《bar()`</text></a></g></a></g></g>

`bartender` 是一个可选属性。"><text text-anchor="start" x="17.75" y="-106.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">bar()</text></a></g> <g id="a_node2_6"><a xlink:href="#" xlink:title="foo(self) -> None:

一架二战时期的 foo 战斗机。"><text text-anchor="start" x="17.75" y="-93.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>D_Class->B_Class</title></g> <g id="node4" class="node"><title>C_Class</title> <g id="a_node4"><a xlink:href="#" xlink:title="class C_Class:

向类中注入一些方法。《C_Class</text> <g id="a_node4_7"><a xlink:href="#" xlink:title="C_Class"><g id="a_node4_8"><a xlink:href="#" xlink:title="qux(self, arg: SomeType) -> SomeType"><text text-anchor="start" x="100.75" y="-109.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">qux()</text></a></g></a></g></a></g></g> <g id="edge3" class="edge"><title>D_Class->C_Class</title></g> <g id="node3" class="node"><title>A_Class</title> <g id="a_node3"><a xlink:href="#" xlink:title="class A_Class:

正确完成 A 任务的类。

带有更长的文档字符串。"><text text-anchor="start" x="8" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">A_Class</text> <g id="a_node3_9"><a xlink:href="#" xlink:title="A_Class"><g id="a_node3_10"><a xlink:href="#" xlink:title="foo(self) -> None:

荣耀的 Foo 的冒险故事"><text text-anchor="start" x="8.75" y="-236.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">foo()</text></a></g> <g id="a_node3_11"><a xlink:href="#" xlink:title="quux(self) -> None:

未使用的方 法。"><text text-anchor="start" x="8.75" y="-223.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">quux()</text></a></g> <g id="a_node3_12"><a xlink:href="#" xlink:title="second(self) -> None"><text text-anchor="start" x="8.75" y="-210" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">second()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>B_Class->A_Class</title></g> <g id="node5" class="node"><title>图例</title> <text text-anchor="start" x="124.12" y="-40.5" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="124.12" y="-30.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.12" y="-30.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">公共方法()</text> <text text-anchor="start" x="124.12" y="-20.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.12" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">私有方法()</text> <text text-anchor="start" x="124.12" y="-10.5" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="130.12" y="-10.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">重载方法()</text> <text text-anchor="start" x="124.12" y="-1.45" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

这里是一个带有抽象类和日志记录的变体：

```py
display_class_hierarchy([A_Class, B_Class],
                        abstract_classes=[A_Class],
                        public_methods=[
                            A_Class.quux,
                        ],
                        log=True) 
```

```py
Drawing class A_Class
    Drawing quux()
    Method doc: quux(self) -&gt; None:&#x0a;A method that is not used.
    Drawing foo()
    Method doc: foo(self) -&gt; None:&#x0a;The Adventures of the glorious Foo
    Drawing second()
    Method doc: second(self) -&gt; None
Drawing class B_Class
    Drawing VAR
    Drawing bar()
    Method doc: bar(self, qux: Any = None, bartender: int = 42) -&gt; None:&#x0a;A qux walks into a bar.&#x0a;`bartender` is an optional attribute.
    Drawing foo()
    Method doc: foo(self) -&gt; None:&#x0a;A WW2 foo fighter.

```

<svg width="210pt" height="199pt" viewBox="0.00 0.00 210.38 198.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 194.5)"><g id="node1" class="node"><title>A_Class</title> <g id="a_node1"><a xlink:href="#" xlink:title="class A_Class:

正确执行某事的类。

附带更长的文档字符串。"><text text-anchor="start" x="8" y="-173.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-style="italic" font-size="14.00" fill="#b03a2e">A_Class</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="A_Class"><g id="a_node1_1"><a xlink:href="#" xlink:title="quux(self) -> None:

一个未使用的函数。"><text text-anchor="start" x="8.75" y="-151.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">quux()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="foo(self) -> None:

光荣的 Foo 的冒险故事"><text text-anchor="start" x="8.75" y="-137.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">foo()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="second(self) -> None"><text text-anchor="start" x="8.75" y="-125" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">second()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>B 类</title> <g id="a_node2"><a xlink:href="#" xlink:title="class B 类:

继承了一些方法的子类。"><text text-anchor="start" x="8" y="-64.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">B 类</text> <g id="a_node2_4"><a xlink:href="#" xlink:title="B 类"><g id="a_node2_5"><a xlink:href="#" xlink:title="VAR = 'A variable'"><text text-anchor="start" x="23.75" y="-41.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">VAR</text></a></g></a></g> <g id="a_node2_6"><a xlink:href="#" xlink:title="B 类"><g id="a_node2_7"><a xlink:href="#" xlink:title="bar(self, qux: Any = None, bartender: int = 42) -> None:

一个 qux 走进了一家酒吧。

`bartender`是一个可选属性。"><text text-anchor="start" x="17.75" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">bar()</text></a></g> <g id="a_node2_8"><a xlink:href="#" xlink:title="foo(self) -> None:

一架二战时期的 foo 战斗机。"><text text-anchor="start" x="17.75" y="-8.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="10.00">foo()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>B_Class->A_Class</title></g> <g id="node3" class="node"><title>图例</title> <text text-anchor="start" x="83.12" y="-56.62" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="83.12" y="-46.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="89.12" y="-46.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">public_method()</text> <text text-anchor="start" x="83.12" y="-36.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="89.12" y="-36.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">private_method()</text> <text text-anchor="start" x="83.12" y="-26.62" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="89.12" y="-26.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">overloaded_method()</text> <text text-anchor="start" x="83.12" y="-17.57" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 练习

享受阅读！

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后更改：2024-06-30 18:45:02+02:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/ClassDiagram.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用这篇作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[类图](https://www.fuzzingbook.org/html/ClassDiagram.html)". 在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 编著的 "[模糊测试书籍](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/ClassDiagram.html`](https://www.fuzzingbook.org/html/ClassDiagram.html). Retrieved 2024-06-30 18:45:02+02:00.

```py
@incollection{fuzzingbook2024:ClassDiagram,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Class Diagrams},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/ClassDiagram.html}},
    note = {Retrieved 2024-06-30 18:45:02+02:00},
    url = {https://www.fuzzingbook.org/html/ClassDiagram.html},
    urldate = {2024-06-30 18:45:02+02:00}
}

```
