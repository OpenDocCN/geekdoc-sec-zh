# 挖掘函数规范

> 原文：[`www.fuzzingbook.org/html/DynamicInvariants.html`](http://www.fuzzingbook.org/html/DynamicInvariants.html)

当测试一个程序时，不仅需要覆盖其多种行为；还需要*检查*结果是否符合预期。在本章中，我们介绍了一种技术，使我们能够从一组给定的执行中*挖掘*函数规范，从而得到函数期望和提供的形式化*描述*。

这些所谓的*动态不变性*从一组执行中生成函数参数和变量的前置和后置条件。它们在多种环境中都很有用：

+   动态不变性为符号模糊测试提供了重要信息，例如函数参数的类型和范围。

+   动态不变性为形式化程序证明和验证提供了前置和后置条件。

+   动态不变性提供了许多断言，可以检查函数行为是否已改变

+   动态不变性提供的检查可以作为*预言机*，用于检查生成的测试的效果

传统上，动态不变性依赖于它们从中派生的执行。然而，当与全面的测试生成器配对时，它们很快就会变得非常精确，正如本章所示。

**先决条件**

+   你应该熟悉跟踪程序执行，如关于覆盖的章节中所述。

+   在本节的后面部分，我们访问 Python 程序的内部*抽象语法树*表示，并对其进行转换，如关于信息流的章节中所述。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
import Coverage
import Intro_Testing 
```

## 摘要

要使用本章提供的代码[Importing.html]，请编写

```py
>>> from fuzzingbook.DynamicInvariants import <identifier> 
```

然后利用以下功能。

本章提供了两个类，可以自动从函数和一组输入中提取规范：

+   `TypeAnnotator`用于*类型*，

+   `InvariantAnnotator`用于*前置*和*后置条件*。

它们都是通过*观察*函数及其在`with`子句中的调用来工作的。以下是一个类型注释器的示例：

```py
>>> def sum(a, b):
>>>     return a + b
>>> with TypeAnnotator() as type_annotator:
>>>     sum(1, 2)
>>>     sum(-4, -5)
>>>     sum(0, 0) 
```

`typed_functions()`方法将返回一个表示`sum2()`的带有执行期间观察到的类型的注释。

```py
>>> print(type_annotator.typed_functions())
def sum(a: int, b: int) -> int:
    return a + b 
```

不变注释器的工作方式类似：

```py
>>> with InvariantAnnotator() as inv_annotator:
>>>     sum(1, 2)
>>>     sum(-4, -5)
>>>     sum(0, 0) 
```

`functions_with_invariants()`方法将返回一个表示`sum2()`的注释，其中包含推断的前置和后置条件，这些条件对所有观察到的值都成立。

```py
>>> print(inv_annotator.functions_with_invariants())
@precondition(lambda b, a: isinstance(a, int))
@precondition(lambda b, a: isinstance(b, int))
@postcondition(lambda return_value, b, a: a == return_value - b)
@postcondition(lambda return_value, b, a: b == return_value - a)
@postcondition(lambda return_value, b, a: isinstance(return_value, int))
@postcondition(lambda return_value, b, a: return_value == a + b)
@postcondition(lambda return_value, b, a: return_value == b + a)
def sum(a, b):
    return a + b 
```

这种类型规范和不变性可以作为*预言机*（用于检测偏离给定运行集的偏差）以及所有类型的*符号代码分析*很有帮助。本章详细介绍了如何自定义要检查的属性。

## 规范和断言

当实现一个函数或程序时，通常针对*规范*（即代码需要满足的一系列文档化要求）进行工作。这些规范可以是自然语言。然而，形式化规范允许计算机检查规范是否得到满足。

在 测试简介 中，我们看到了 *前置条件* 和 *后置条件* 如何描述函数的行为。考虑以下（简单的）平方根函数：

```py
def any_sqrt(x):
    assert x >= 0  # Precondition

    ...

    assert result * result == x  # Postcondition
    return result 
```

断言 `assert p` 检查条件 `p`；如果条件不成立，则执行被终止。在这里，实际的主体尚未编写；我们使用断言作为对 `any_sqrt()` *期望* 和 *提供* 的规范。

最顶部的断言是 *前置条件*，说明了函数参数的要求。结尾的断言是 *后置条件*，说明了函数结果（包括其与原始参数的关系）的性质。使用这些前置和后置条件作为规范，我们现在可以编写一个满足这些条件的平方根函数。一旦实现，我们就可以让断言在运行时检查 `any_sqrt()` 是否按预期工作；一个 符号测试生成器 或 冲突测试生成器 甚至会尝试找到断言 *不成立* 的特定输入。（断言可以看作是一个条件分支，用于终止执行，任何试图覆盖所有代码分支的技术也会尝试尽可能多地使断言无效。）

然而，并非所有代码都是首先带有明确规范的；更不用说大多数代码带有正式的前置和后置条件。（只需看看这本书的章节。）这是令人遗憾的：正如肯·汤普森著名地说，“没有规范，就没有错误——只有惊喜”。这对测试也是一个问题，因为当然，测试需要一些规范来测试。这提出了一个有趣的问题：我们能否以某种方式 *改造* 现有代码，使其带有“规范”，这些规范可以正确地描述其行为，使开发者只需简单地 *检查* 而不必从头编写？这就是我们在本章所做的事情。

## 为什么泛型错误检查不够

在我们开始 *挖掘* 规范之前，让我们首先讨论为什么拥有它们可能是有用的。作为一个激励的例子，考虑 测试简介 中的平方根函数的完整实现：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
def my_sqrt(x):
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx 
```

`my_sqrt()` 没有任何检查类型或值的函数。因此，调用者调用 `my_sqrt()` 时很容易出错：

```py
from ExpectError import ExpectError, ExpectTimeout 
```

```py
with ExpectError():
    my_sqrt("foo") 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/829521914.py", line 2, in <module>
    my_sqrt("foo")
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2661069967.py", line 4, in my_sqrt
    guess = x / 2
            ~~^~~
TypeError: unsupported operand type(s) for /: 'str' and 'int' (expected)

```

```py
with ExpectError():
    x = my_sqrt(0.0) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/1975547953.py", line 2, in <module>
    x = my_sqrt(0.0)
        ^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2661069967.py", line 7, in my_sqrt
    guess = (approx + x / approx) / 2
                      ~~^~~~~~~~
ZeroDivisionError: float division by zero (expected)

```

至少，Python 系统会在运行时捕获这些错误。然而，以下调用却简单地让函数进入无限循环：

```py
with ExpectTimeout(1):
    x = my_sqrt(-1.0) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/1349814288.py", line 2, in <module>
    x = my_sqrt(-1.0)
        ^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2661069967.py", line 5, in my_sqrt
    while approx != guess:
          ^^^^^^^^^^^^^^^
  File "Timeout.ipynb", line 43, in timeout_handler
    raise TimeoutError()
TimeoutError (expected)

```

我们的目标是通过 *注解* 函数来避免上述错误，这些注解提供了防止错误的信息。想法是提供一个 *规范*，该规范可以随后在运行时或静态时进行检查。

\todo{介绍 *合同* 的概念。}

## 指定和检查数据类型

对于我们的 Python 代码，我们最需要的“规范”之一是**类型**。Python 是一种“动态”类型语言，这意味着所有数据类型都是在运行时确定的；代码本身并没有明确声明变量是整数、字符串、数组、字典——或任何其他类型。

作为 Python 代码的**作者**，省略显式的类型声明可能会节省时间（并且允许一些有趣的技巧）。是否缺少类型有助于人类**阅读**和**理解**代码还不确定。对于一个试图分析代码的**计算机**来说，缺少显式类型是有害的。比如说，如果约束求解器看到 `if x:` 而无法知道 `x` 是否应该是一个数字或一个字符串，这会引入**歧义**。这样的歧义可能会在整个分析中成倍增加——或者是在产生过于不准确结果的分析中。

Python 3.6 及以后的版本允许将数据类型作为**注解**用于函数参数（实际上，用于所有变量）和返回值。例如，我们可以声明 `my_sqrt()` 是一个接受浮点值并返回一个值的函数：

```py
def my_sqrt_with_type_annotations(x: float) -> float:
  """Computes the square root of x, using the Newton-Raphson method"""
    return my_sqrt(x) 
```

默认情况下，Python 解释器会忽略这样的注解。因此，你仍然可以用字符串作为参数调用 `my_sqrt_typed()` 并得到与上面完全相同的结果。然而，你可以利用特殊的**类型检查**模块来检查类型——**动态地**在运行时检查，或者**静态地**通过分析代码而不必执行它。

### 离题：运行时类型检查

（注释掉，因为 `enforce` 不受 Python 3.9 支持）

Python 的 `enforce` 包提供了一个函数装饰器，它会在运行时自动插入类型检查代码。以下是它的用法：

```py
# import enforce 
```

```py
# @enforce.runtime_validation
# def my_sqrt_with_checked_type_annotations(x: float) -> float:
#     """Computes the square root of x, using the Newton-Raphson method"""
#     return my_sqrt(x) 
```

现在，当用与声明的类型不同的类型调用 `my_sqrt_with_checked_type_annotations()` 时，会引发异常：

```py
# with ExpectError():
#     my_sqrt_with_checked_type_annotations(True) 
```

注意，这个错误没有被“无类型”变体捕获，其中传递布尔值会愉快地返回 $\sqrt{1}$ 作为结果。

```py
# my_sqrt(True) 
```

在 Python（和其他语言）中，布尔值 `True` 和 `False` 可以隐式转换为整数 1 和 0；然而，很难想象一个调用 `sqrt()` 不会出错的情况。

### 静态类型检查

类型注解也可以**静态地**进行检查——也就是说，甚至不需要运行代码。让我们创建一个简单的 Python 文件，其中包含上述 `my_sqrt_typed()` 定义和一个错误的调用。

```py
import [inspect](https://docs.python.org/3/library/inspect.html)
import [tempfile](https://docs.python.org/3/library/tempfile.html) 
```

```py
f = tempfile.NamedTemporaryFile(mode='w', suffix='.py')
f.name 
```

```py
'/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpe7k1dgu9.py'

```

```py
f.write(inspect.getsource(my_sqrt))
f.write('\n')
f.write(inspect.getsource(my_sqrt_with_type_annotations))
f.write('\n')
f.write("print(my_sqrt_with_type_annotations('123'))\n")
f.flush() 
```

这些是我们新创建的 Python 文件的内容：

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import print_file 
```

```py
print_file(f.name) 
```

```py
def my_sqrt(x):
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

def my_sqrt_with_type_annotations(x: float) -> float:
  """Computes the square root of x, using the Newton-Raphson method"""
    return my_sqrt(x)

print(my_sqrt_with_type_annotations('123'))

```

[Mypy](http://mypy-lang.org) 是一个用于 Python 程序的类型检查器。因为它进行静态类型检查，所以类型不会在运行时产生开销；此外，静态检查可能比启用运行时类型检查的长时间测试序列更快。让我们看看 `mypy` 在上述文件上会产生什么结果：

```py
import [subprocess](https://docs.python.org/3/library/subprocess.html) 
```

```py
result = subprocess.run(["mypy", "--strict", f.name], universal_newlines=True, stdout=subprocess.PIPE)
del f  # Delete temporary file 
```

```py
print(result.stdout) 
```

```py
/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpe7k1dgu9.py:1: error: Function is missing a type annotation  [no-untyped-def]
/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpe7k1dgu9.py:12: error: Returning Any from function declared to return "float"  [no-any-return]
/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpe7k1dgu9.py:12: error: Call to untyped function "my_sqrt" in typed context  [no-untyped-call]
/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpe7k1dgu9.py:14: error: Argument 1 to "my_sqrt_with_type_annotations" has incompatible type "str"; expected "float"  [arg-type]
Found 4 errors in 1 file (checked 1 source file)

```

我们可以看到 `mypy` 对无类型的函数定义（如 `my_sqrt()`）表示不满；然而，最重要的是，它发现最后一行中调用 `my_sqrt_with_type_annotations()` 的类型是错误的。

使用`mypy`，我们可以像在静态类型语言中一样在 Python 中实现相同的安全类型，前提是我们作为程序员也产生了必要的类型注解。有没有简单的方法可以获得这些注解？

## 矿化类型规范

我们的首要任务将是从运行时观察到的*值*中挖掘类型注解（作为代码的一部分）。这些类型注解将从实际函数执行中挖掘出来，通过（正常）运行学习预期的参数和返回类型应该是什么。通过观察一系列这样的调用，我们可以推断出`x`和返回值都是`float`类型：

```py
y = my_sqrt(25.0)
y 
```

```py
5.0

```

```py
y = my_sqrt(2.0)
y 
```

```py
1.414213562373095

```

我们如何从执行中挖掘类型？答案是简单的：

1.  我们在执行过程中*观察*一个函数

1.  我们跟踪其参数的类型

1.  我们将这些类型作为*注解*包含到代码中。

为了做到这一点，我们可以利用我们已经在覆盖率章节中观察到的 Python 的跟踪功能。每当调用一个函数时，我们都会检索其参数、它们的值和它们的类型。

### 跟踪调用

为了在运行时观察参数类型，我们定义了一个*跟踪函数*，它跟踪`my_sqrt()`的执行，检查其参数和返回值。`Tracker`类被设置为在`with`块中跟踪函数，如下所示：

```py
with Tracker() as tracker:
    function_to_be_tracked(...)
info = tracker.collected_information() 
```

就像在覆盖率章节中一样，我们使用`sys.settrace()`函数在执行期间跟踪单个函数。我们在`with`块开始时打开跟踪；此时调用`__enter__()`方法。当`with`块的执行结束时，调用`__exit__()`。

```py
import [sys](https://docs.python.org/3/library/sys.html) 
```

```py
class Tracker:
    def __init__(self, log=False):
        self._log = log
        self.reset()

    def reset(self):
        self._calls = {}
        self._stack = []

    def traceit(self):
  """Placeholder to be overloaded in subclasses"""
        pass

    # Start of `with` block
    def __enter__(self):
        self.original_trace_function = sys.gettrace()
        sys.settrace(self.traceit)
        return self

    # End of `with` block
    def __exit__(self, exc_type, exc_value, tb):
        sys.settrace(self.original_trace_function) 
```

`traceit()`方法目前什么也不做；这是在专门的子类中完成的。`CallTracker`类实现了一个`traceit()`函数，用于检查函数调用和返回：

```py
class CallTracker(Tracker):
    def traceit(self, frame, event, arg):
  """Tracking function: Record all calls and all args"""
        if event == "call":
            self.trace_call(frame, event, arg)
        elif event == "return":
            self.trace_return(frame, event, arg)

        return self.traceit 
```

当函数被调用时，会调用`trace_call()`；它检索函数名称和当前参数，并将它们保存在栈上。

```py
class CallTracker(CallTracker):
    def trace_call(self, frame, event, arg):
  """Save current function name and args on the stack"""
        code = frame.f_code
        function_name = code.co_name
        arguments = get_arguments(frame)
        self._stack.append((function_name, arguments))

        if self._log:
            print(simple_call_string(function_name, arguments)) 
```

```py
def get_arguments(frame):
  """Return call arguments in the given frame"""
    # When called, all arguments are local variables
    local_variables = dict(frame.f_locals)  # explicit copy
    arguments = [(var, frame.f_locals[var]) for var in local_variables]
    arguments.reverse()  # Want same order as call
    return arguments 
```

当函数返回时，会调用`trace_return()`。现在我们也有了返回值。我们可以记录整个调用及其参数和返回值（如果需要的话），并将其保存在我们的调用列表中。

```py
class CallTracker(CallTracker):
    def trace_return(self, frame, event, arg):
  """Get return value and store complete call with arguments and return value"""
        code = frame.f_code
        function_name = code.co_name
        return_value = arg
        # TODO: Could call get_arguments() here to also retrieve _final_ values of argument variables

        called_function_name, called_arguments = self._stack.pop()
        assert function_name == called_function_name

        if self._log:
            print(simple_call_string(function_name, called_arguments), "returns", return_value)

        self.add_call(function_name, called_arguments, return_value) 
```

`simple_call_string()`是一个用于日志记录的辅助函数，以用户友好的方式打印调用。

```py
def simple_call_string(function_name, argument_list, return_value=None):
  """Return function_name(arg[0], arg[1], ...) as a string"""
    call = function_name + "(" + \
        ", ".join([var + "=" + repr(value)
                   for (var, value) in argument_list]) + ")"

    if return_value is not None:
        call += " = " + repr(return_value)

    return call 
```

`add_call()`将调用保存在列表中；每个函数名称都有自己的列表。

```py
class CallTracker(CallTracker):
    def add_call(self, function_name, arguments, return_value=None):
  """Add given call to list of calls"""
        if function_name not in self._calls:
            self._calls[function_name] = []
        self._calls[function_name].append((arguments, return_value)) 
```

使用`calls()`，我们可以检索调用列表，无论是针对特定函数还是所有函数。

```py
class CallTracker(CallTracker):
    def calls(self, function_name=None):
  """Return list of calls for function_name, 
 or a mapping function_name -> calls for all functions tracked"""
        if function_name is None:
            return self._calls

        return self._calls[function_name] 
```

让我们将其付诸实践。我们打开日志记录以跟踪单个调用及其返回值：

```py
with CallTracker(log=True) as tracker:
    y = my_sqrt(25)
    y = my_sqrt(2.0) 
```

```py
my_sqrt(x=25)
my_sqrt(x=25) returns 5.0
my_sqrt(x=2.0)
my_sqrt(x=2.0) returns 1.414213562373095
__exit__(tb=None, exc_value=None, exc_type=None, self=<__main__.CallTracker object at 0x10b64c920>)

```

执行后，我们可以检索单个调用：

```py
calls = tracker.calls('my_sqrt')
calls 
```

```py
[([('x', 25)], 5.0), ([('x', 2.0)], 1.414213562373095)]

```

每个调用都是一个对（`argument_list`, `return_value`），其中`argument_list`是一个包含对（`parameter_name`, `value`）的列表。

```py
my_sqrt_argument_list, my_sqrt_return_value = calls[0]
simple_call_string('my_sqrt', my_sqrt_argument_list, my_sqrt_return_value) 
```

```py
'my_sqrt(x=25) = 5.0'

```

如果函数不返回值，则`return_value`为`None`。

```py
def hello(name):
    print("Hello,", name) 
```

```py
with CallTracker() as tracker:
    hello("world") 
```

```py
Hello, world

```

```py
hello_calls = tracker.calls('hello')
hello_calls 
```

```py
[([('name', 'world')], None)]

```

```py
hello_argument_list, hello_return_value = hello_calls[0]
simple_call_string('hello', hello_argument_list, hello_return_value) 
```

```py
"hello(name='world')"

```

### 获取类型

尽管你可能已经阅读或听到过，Python 实际上*是*一种类型语言。只是它是*动态类型化的*——类型仅在运行时使用和检查（而不是在代码中声明，它们可以在编译时*静态检查*）。因此，我们可以检索 Python 中所有值的类型：

```py
type(4) 
```

```py
int

```

```py
type(2.0) 
```

```py
float

```

```py
type([4]) 
```

```py
list

```

我们可以检索`my_sqrt()`的第一个参数的类型：

```py
parameter, value = my_sqrt_argument_list[0]
parameter, type(value) 
```

```py
('x', int)

```

以及返回值的类型：

```py
type(my_sqrt_return_value) 
```

```py
float

```

因此，我们看到（到目前为止），`my_sqrt()`是一个接受（包括）整数和浮点数并返回浮点数的函数。我们可以将`my_sqrt()`声明为：

```py
def my_sqrt_annotated(x: float) -> float:
    return my_sqrt(x) 
```

这是一种可以放入静态类型检查器中的表示，允许检查对`my_sqrt()`的调用是否实际上传递了一个数字。动态类型检查器可以在运行时执行此类检查。当然，任何符号解释都将从额外的注释中受益良多。

默认情况下，Python 不会对这样的注释做任何事情。然而，工具可以访问函数和其他对象的注释：

```py
my_sqrt_annotated.__annotations__ 
```

```py
{'x': float, 'return': float}

```

这是运行时检查器如何访问注释以进行检查的方式。

### 访问函数结构

我们的计划是根据我们看到的类型自动注解函数。为此，我们需要一些模块，允许我们将函数转换为树表示（称为*抽象语法树*，或 AST）并将其转换回来；我们已经在 concolic 和 symbolic 测试的章节中看到了这些。

```py
import [ast](https://docs.python.org/3/library/ast.html)
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

我们可以使用`inspect.getsource()`获取 Python 函数的源代码。（注意，这不适用于在其他笔记本中定义的函数。）

```py
my_sqrt_source = inspect.getsource(my_sqrt)
my_sqrt_source 
```

```py
'def my_sqrt(x):\n    """Computes the square root of x, using the Newton-Raphson method"""\n    approx = None\n    guess = x / 2\n    while approx != guess:\n        approx = guess\n        guess = (approx + x / approx) / 2\n    return approx\n'

```

为了以视觉上令人愉悦的形式查看这些内容，我们的函数`print_content(s, suffix)`格式化和突出显示字符串`s`，就像它是一个以`suffix`结尾的文件一样。因此，我们可以将源代码视为（并突出显示）一个 Python 文件：

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import print_content 
```

```py
print_content(my_sqrt_source, '.py') 
```

```py
def my_sqrt(x):
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

解析此内容为我们提供了一个抽象语法树（AST）——以树形表示的程序。

```py
my_sqrt_ast = ast.parse(my_sqrt_source) 
```

这个 AST 看起来是什么样子？辅助函数`ast.dump()`（文本输出）和`showast.show_ast()`（带有[showast](https://github.com/hchasestevens/show_ast)的图形输出）允许我们检查树的结构。我们看到函数以具有名称和参数的`FunctionDef`开始，然后是一个体，它是一个类型为`Expr`（文档字符串）、类型`Assign`（赋值）、`While`（具有自己体的 while 循环）和最后的`Return`的语句列表。

```py
print(ast.dump(my_sqrt_ast, indent=4)) 
```

```py
Module(
    body=[
        FunctionDef(
            name='my_sqrt',
            args=arguments(
                posonlyargs=[],
                args=[
                    arg(arg='x')],
                kwonlyargs=[],
                kw_defaults=[],
                defaults=[]),
            body=[
                Expr(
                    value=Constant(value='Computes the square root of x, using the Newton-Raphson method')),
                Assign(
                    targets=[
                        Name(id='approx', ctx=Store())],
                    value=Constant(value=None)),
                Assign(
                    targets=[
                        Name(id='guess', ctx=Store())],
                    value=BinOp(
                        left=Name(id='x', ctx=Load()),
                        op=Div(),
                        right=Constant(value=2))),
                While(
                    test=Compare(
                        left=Name(id='approx', ctx=Load()),
                        ops=[
                            NotEq()],
                        comparators=[
                            Name(id='guess', ctx=Load())]),
                    body=[
                        Assign(
                            targets=[
                                Name(id='approx', ctx=Store())],
                            value=Name(id='guess', ctx=Load())),
                        Assign(
                            targets=[
                                Name(id='guess', ctx=Store())],
                            value=BinOp(
                                left=BinOp(
                                    left=Name(id='approx', ctx=Load()),
                                    op=Add(),
                                    right=BinOp(
                                        left=Name(id='x', ctx=Load()),
                                        op=Div(),
                                        right=Name(id='approx', ctx=Load()))),
                                op=Div(),
                                right=Constant(value=2)))],
                    orelse=[]),
                Return(
                    value=Name(id='approx', ctx=Load()))],
            decorator_list=[],
            type_params=[])],
    type_ignores=[])

```

文字太多？这个图形表示可能使事情更简单。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import rich_output 
```

```py
if rich_output():
    import [showast](https://pypi.org/project/showast/)
    showast.show_ast(my_sqrt_ast) 
```

<svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1684pt" height="548pt" viewBox="0.00 0.00 1684.00 548.00"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 544)"><g id="node1" class="node"><title>0</title> <text text-anchor="start" x="282.62" y="-517.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">FunctionDef</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="82" y="-444.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"my_sqrt"</text></g> <g id="edge1" class="edge"><title>0--1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="start" x="152.88" y="-445.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arguments</text></g> <g id="edge2" class="edge"><title>0--2</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="start" x="261.25" y="-445.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Assign</text></g> <g id="edge5" class="edge"><title>0--5</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="start" x="346.25" y="-445.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Assign</text></g> <g id="edge10" class="edge"><title>0--10</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="start" x="909.38" y="-445.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">While</text></g> <g id="edge21" class="edge"><title>0--21</title></g> <g id="node59" class="node"><title>58</title> <text text-anchor="start" x="1316.25" y="-445.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge58" class="edge"><title>0--58</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="start" x="63.62" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arg</text></g> <g id="edge3" class="edge"><title>2--3</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="27" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"x"</text></g> <g id="edge4" class="edge"><title>3--4</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="start" x="182.5" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge6" class="edge"><title>5--6</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="285" y="-372.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Constant</text></g> <g id="edge9" class="edge"><title>5--9</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="113" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"approx"</text></g> <g id="edge7" class="edge"><title>6--7</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="201" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Store</text></g> <g id="edge8" class="edge"><title>6--8</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="start" x="354.5" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge11" class="edge"><title>10--11</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="start" x="458.38" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">BinOp</text></g> <g id="edge14" class="edge"><title>10--14</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="285" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"guess"</text></g> <g id="edge12" class="edge"><title>11--12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="369" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Store</text></g> <g id="edge13" class="edge"><title>11--13</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="start" x="426.5" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge15" class="edge"><title>14--15</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="middle" x="515" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Div</text></g> <g id="edge18" class="edge"><title>14--18</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="start" x="568" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge19" class="edge"><title>14--19</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="371" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"x"</text></g> <g id="edge16" class="edge"><title>15--16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="443" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge17" class="edge"><title>15--17</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="middle" x="515" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">2</text></g> <g id="edge20" class="edge"><title>19--20</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="start" x="777.12" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge22" class="edge"><title>21--22</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="start" x="956.25" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Assign</text></g> <g id="edge30" class="edge"><title>21--30</title></g> <g id="node38" class="node"><title>37</title> <text text-anchor="start" x="1290.25" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Assign</text></g> <g id="edge37" class="edge"><title>21--37</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="start" x="670.5" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge23" class="edge"><title>22--23</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="769" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">NotEq</text></g> <g id="edge26" class="edge"><title>22--26</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="start" x="826.5" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge27" class="edge"><title>22--27</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="middle" x="601" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"approx"</text></g> <g id="edge24" class="edge"><title>23--24</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="687" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge25" class="edge"><title>23--25</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="middle" x="769" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"guess"</text></g> <g id="edge28" class="edge"><title>27--28</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="851" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge29" class="edge"><title>27--29</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="start" x="964.5" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge31" class="edge"><title>30--31</title></g> <g id="node35" class="node"><title>34</title> <text text-anchor="start" x="1092.5" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge34" class="edge"><title>30--34</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="937" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"approx"</text></g> <g id="edge32" class="edge"><title>31--32</title></g> <g id="node34" class="node"><title>33</title> <text text-anchor="middle" x="1025" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Store</text></g> <g id="edge33" class="edge"><title>31--33</title></g> <g id="node36" class="node"><title>35</title> <text text-anchor="middle" x="1109" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"guess"</text></g> <g id="edge35" class="edge"><title>34--35</title></g> <g id="node37" class="node"><title>36</title> <text text-anchor="middle" x="1191" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge36" class="edge"><title>34--36</title></g> <g id="node39" class="node"><title>38</title> <text text-anchor="start" x="1298.5" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge38" class="edge"><title>37--38</title></g> <g id="node42" class="node"><title>41</title> <text text-anchor="start" x="1411.38" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">BinOp</text></g> <g id="edge41" class="edge"><title>37--41</title></g> <g id="node40" class="node"><title>39</title> <text text-anchor="middle" x="1273" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"guess"</text></g> <g id="edge39" class="edge"><title>38--39</title></g> <g id="node41" class="node"><title>40</title> <text text-anchor="middle" x="1357" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Store</text></g> <g id="edge40" class="edge"><title>38--40</title></g> <g id="node43" class="node"><title>42</title> <text text-anchor="start" x="1411.38" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">BinOp</text></g> <g id="edge42" class="edge"><title>41--42</title></g> <g id="node56" class="node"><title>55</title> <text text-anchor="middle" x="1506" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Div</text></g> <g id="edge55" class="edge"><title>41--55</title></g> <g id="node57" class="node"><title>56</title> <text text-anchor="start" x="1559" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge56" class="edge"><title>41--56</title></g> <g id="node44" class="node"><title>43</title> <text text-anchor="start" x="1343.5" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge43" class="edge"><title>42--43</title></g> <g id="node47" class="node"><title>46</title> <text text-anchor="middle" x="1432" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Add</text></g> <g id="edge46" class="edge"><title>42--46</title></g> <g id="node48" class="node"><title>47</title> <text text-anchor="start" x="1485.38" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">BinOp</text></g> <g id="edge47" class="edge"><title>42--47</title></g> <g id="node45" class="node"><title>44</title> <text text-anchor="middle" x="1275" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"approx"</text></g> <g id="edge44" class="edge"><title>43--44</title></g> <g id="node46" class="node"><title>45</title> <text text-anchor="middle" x="1361" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge45" class="edge"><title>43--45</title></g> <g id="node49" class="node"><title>48</title> <text text-anchor="start" x="1417.5" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge48" class="edge"><title>47--48</title></g> <g id="node52" class="node"><title>51</title> <text text-anchor="middle" x="1506" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Div</text></g> <g id="edge51" class="edge"><title>47--51</title></g> <g id="node53" class="node"><title>52</title> <text text-anchor="start" x="1561.5" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge52" class="edge"><title>47--52</title></g> <g id="node50" class="node"><title>49</title> <text text-anchor="middle" x="1376" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"x"</text></g> <g id="edge49" class="edge"><title>48--49</title></g> <g id="node51" class="node"><title>50</title> <text text-anchor="middle" x="1448" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge50" class="edge"><title>48--50</title></g> <g id="node54" class="node"><title>53</title> <text text-anchor="middle" x="1563" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"approx"</text></g> <g id="edge53" class="edge"><title>52--53</title></g> <g id="node55" class="node"><title>54</title> <text text-anchor="middle" x="1649" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge54" class="edge"><title>52--54</title></g> <g id="node58" class="node"><title>57</title> <text text-anchor="middle" x="1592" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">2</text></g> <g id="edge57" class="edge"><title>56--57</title></g> <g id="node60" class="node"><title>59</title> <text text-anchor="start" x="1503.5" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge59" class="edge"><title>58--59</title></g> <g id="node61" class="node"><title>60</title> <text text-anchor="middle" x="1520" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"approx"</text></g> <g id="edge60" class="edge"><title>59--60</title></g> <g id="node62" class="node"><title>61</title> <text text-anchor="middle" x="1606" y="-300.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge61" class="edge"><title>59--61</title></g></g></svg>

函数`ast.unparse()`将这样的树转换回更熟悉的文本 Python 代码表示。注释已删除，并且可能比之前有更多的括号，但结果具有相同的语义：

```py
print_content(ast.unparse(my_sqrt_ast), '.py') 
```

```py
def my_sqrt(x):
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

### 使用给定类型注解函数

让我们现在转换这些树以添加类型注释。我们从一个辅助函数`parse_type(name)`开始，它将类型名称解析为 AST。

```py
def parse_type(name):
    class ValueVisitor(ast.NodeVisitor):
        def visit_Expr(self, node):
            self.value_node = node.value

    tree = ast.parse(name)
    name_visitor = ValueVisitor()
    name_visitor.visit(tree)
    return name_visitor.value_node 
```

```py
print(ast.dump(parse_type('int'))) 
```

```py
Name(id='int', ctx=Load())

```

```py
print(ast.dump(parse_type('[object]'))) 
```

```py
List(elts=[Name(id='object', ctx=Load())], ctx=Load())

```

我们现在定义一个辅助函数，该函数实际上向函数 AST 添加类型注释。`TypeTransformer` 类建立在 Python 标准库 `ast.NodeTransformer` 基础之上。它将被调用为

```py
TypeTransformer({'x': 'int'}, 'float').visit(ast) 
```

要注释 `my_sqrt()` 的参数：`x` 为 `int`，返回类型为 `float`。然后可以反解析、编译或分析返回的 AST。

```py
class TypeTransformer(ast.NodeTransformer):
    def __init__(self, argument_types, return_type=None):
        self.argument_types = argument_types
        self.return_type = return_type
        super().__init__() 
```

`TypeTransformer` 的核心方法是 `visit_FunctionDef()`，它在 AST 中的每个函数定义上都会被调用。它的参数 `node` 是要转换的函数定义的子树。我们的实现访问单个参数并在它们上调用 `annotate_args()`；它还在节点的 `returns` 属性中设置返回类型。

```py
class TypeTransformer(TypeTransformer):
    def visit_FunctionDef(self, node):
  """Add annotation to function"""
        # Set argument types
        new_args = []
        for arg in node.args.args:
            new_args.append(self.annotate_arg(arg))

        new_arguments = ast.arguments(
            node.args.posonlyargs,
            new_args,
            node.args.vararg,
            node.args.kwonlyargs,
            node.args.kw_defaults,
            node.args.kwarg,
            node.args.defaults
        )

        # Set return type
        if self.return_type is not None:
            node.returns = parse_type(self.return_type)

        return ast.copy_location(
            ast.FunctionDef(node.name, new_arguments, 
                            node.body, node.decorator_list,
                            node.returns), node) 
```

每个参数都获得自己的注释，这些注释来自最初传递给类的类型：

```py
class TypeTransformer(TypeTransformer):
    def annotate_arg(self, arg):
  """Add annotation to single function argument"""
        arg_name = arg.arg
        if arg_name in self.argument_types:
            arg.annotation = parse_type(self.argument_types[arg_name])
        return arg 
```

这可行吗？让我们用类型注释 `my_sqrt()` 的参数和返回类型来注释 AST：

```py
new_ast = TypeTransformer({'x': 'int'}, 'float').visit(my_sqrt_ast) 
```

当我们反解析新的 AST 时，我们看到注释实际上确实存在：

```py
print_content(ast.unparse(new_ast), '.py') 
```

```py
def my_sqrt(x: int) -> float:
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

同样，我们可以注释上面提到的 `hello()` 函数：

```py
hello_source = inspect.getsource(hello) 
```

```py
hello_ast = ast.parse(hello_source) 
```

```py
new_ast = TypeTransformer({'name': 'str'}, 'None').visit(hello_ast) 
```

```py
print_content(ast.unparse(new_ast), '.py') 
```

```py
def hello(name: str) -> None:
    print('Hello,', name)

```

### 使用挖掘的类型注释函数

现在让我们用在运行时挖掘的类型注释函数。我们从简单的函数 `type_string()` 开始，该函数确定给定值（作为字符串）的适当类型：

```py
def type_string(value):
    return type(value).__name__ 
```

```py
type_string(4) 
```

```py
'int'

```

```py
type_string([]) 
```

```py
'list'

```

对于复合结构，`type_string()` 不检查元素类型；因此，`[3]` 的类型只是 `list`，而不是例如 `list[int]`。目前，`list` 就足够了。

```py
type_string([3]) 
```

```py
'list'

```

`type_string()` 将用于推断在运行时通过 `CallTracker.calls()` 返回的参数值的类型：

```py
with CallTracker() as tracker:
    y = my_sqrt(25.0)
    y = my_sqrt(2.0) 
```

```py
tracker.calls() 
```

```py
{'my_sqrt': [([('x', 25.0)], 5.0), ([('x', 2.0)], 1.414213562373095)]}

```

`annotate_types()` 函数接受这样的调用列表，并对列出的每个函数进行注释：

```py
def annotate_types(calls):
    annotated_functions = {}

    for function_name in calls:
        try:
            annotated_functions[function_name] = annotate_function_with_types(function_name, calls[function_name])
        except KeyError:
            continue

    return annotated_functions 
```

对于每个函数，我们获取其源代码及其 AST，然后进入 `annotate_function_ast_with_types()` 中的实际注释：

```py
def annotate_function_with_types(function_name, function_calls):
    function = globals()[function_name]  # May raise KeyError for internal functions
    function_code = inspect.getsource(function)
    function_ast = ast.parse(function_code)
    return annotate_function_ast_with_types(function_ast, function_calls) 
```

`annotate_function_ast_with_types()` 函数使用看到的调用调用 `TypeTransformer`，并对每个调用迭代参数，确定它们的类型，并用这些类型注释 AST。当我们遇到类型冲突时，使用通用类型 `Any`，我们将在下面讨论。

```py
from [typing](https://docs.python.org/3/library/typing.html) import Any 
```

```py
def annotate_function_ast_with_types(function_ast, function_calls):
    parameter_types = {}
    return_type = None

    for calls_seen in function_calls:
        args, return_value = calls_seen
        if return_value is not None:
            if return_type is not None and return_type != type_string(return_value):
                return_type = 'Any'
            else:
                return_type = type_string(return_value)

        for parameter, value in args:
            try:
                different_type = parameter_types[parameter] != type_string(value)
            except KeyError:
                different_type = False

            if different_type:
                parameter_types[parameter] = 'Any'
            else:
                parameter_types[parameter] = type_string(value)

    annotated_function_ast = TypeTransformer(parameter_types, return_type).visit(function_ast)
    return annotated_function_ast 
```

这是使用跟踪器记录的类型注释的 `my_sqrt()`：

```py
print_content(ast.unparse(annotate_types(tracker.calls())['my_sqrt']), '.py') 
```

```py
def my_sqrt(x: float) -> float:
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

### 一站式注释

让我们在单个类 `TypeAnnotator` 中将这些内容整合起来，该类首先跟踪函数调用，然后允许访问带有类型注释的跟踪函数的 AST（以及源代码形式）。`typed_functions()` 方法返回注释函数的字符串；`typed_functions_ast()` 返回它们的 AST。

```py
class TypeTracker(CallTracker):
    pass 
```

```py
class TypeAnnotator(TypeTracker):
    def typed_functions_ast(self, function_name=None):
        if function_name is None:
            return annotate_types(self.calls())

        return annotate_function_with_types(function_name, 
                                            self.calls(function_name))

    def typed_functions(self, function_name=None):
        if function_name is None:
            functions = ''
            for f_name in self.calls():
                try:
                    f_text = ast.unparse(self.typed_functions_ast(f_name))
                except KeyError:
                    f_text = ''
                functions += f_text
            return functions

        return ast.unparse(self.typed_functions_ast(function_name)) 
```

这是如何使用 `TypeAnnotator` 的。我们首先跟踪一系列调用：

```py
with TypeAnnotator() as annotator:
    y = my_sqrt(25.0)
    y = my_sqrt(2.0) 
```

跟踪后，我们可以立即检索跟踪函数的注释版本：

```py
print_content(annotator.typed_functions(), '.py') 
```

```py
def my_sqrt(x: float) -> float:
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

这也适用于多个和多样化的函数。有人可以基于执行期间看到的类型实现一个自动类型注释器，用于 Python 文件。

```py
with TypeAnnotator() as annotator:
    hello('type annotations')
    y = my_sqrt(1.0) 
```

```py
Hello, type annotations

```

```py
print_content(annotator.typed_functions(), '.py') 
```

```py
def hello(name: str):
    print('Hello,', name)def my_sqrt(x: float) -> float:
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

现在可以将上述内容发送给类型检查器，它将检测调用者和被调用者之间是否存在任何类型不一致。同样，如上所述的类型注解对于符号代码分析（如 符号模糊测试 章节中所述）非常有用，因为它们有效地限制了参数和变量可以取的值的集合。

### 多种类型

让我们现在来解析一下在 `annotate_function_ast_with_types()` 中魔法 `Any` 类型的角色。如果我们看到同一个参数有多个类型，我们将它的类型设置为 `Any`。对于 `my_sqrt()` 来说，这是有意义的，因为它的参数可以是整数也可以是浮点数：

```py
with CallTracker() as tracker:
    y = my_sqrt(25.0)
    y = my_sqrt(4) 
```

```py
print_content(ast.unparse(annotate_types(tracker.calls())['my_sqrt']), '.py') 
```

```py
def my_sqrt(x: Any) -> float:
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

以下函数 `sum3()` 可以用浮点数作为参数调用，这将导致参数获得 `float` 类型：

```py
def sum3(a, b, c):
    return a + b + c 
```

```py
with TypeAnnotator() as annotator:
    y = sum3(1.0, 2.0, 3.0)
y 
```

```py
6.0

```

```py
print_content(annotator.typed_functions(), '.py') 
```

```py
def sum3(a: float, b: float, c: float) -> float:
    return a + b + c

```

如果我们用整数调用 `sum3()`，那么参数将获得 `int` 类型：

```py
with TypeAnnotator() as annotator:
    y = sum3(1, 2, 3)
y 
```

```py
6

```

```py
print_content(annotator.typed_functions(), '.py') 
```

```py
def sum3(a: int, b: int, c: int) -> int:
    return a + b + c

```

我们也可以用字符串调用 `sum3()`，这将给参数赋予 `str` 类型：

```py
with TypeAnnotator() as annotator:
    y = sum3("one", "two", "three")
y 
```

```py
'onetwothree'

```

```py
print_content(annotator.typed_functions(), '.py') 
```

```py
def sum3(a: str, b: str, c: str) -> str:
    return a + b + c

```

如果我们有多个调用，但参数类型不同，`TypeAnnotator()` 将会为这两个参数和返回值分配一个 `Any` 类型：

```py
with TypeAnnotator() as annotator:
    y = sum3(1, 2, 3)
    y = sum3("one", "two", "three") 
```

```py
typed_sum3_def = annotator.typed_functions('sum3') 
```

```py
print_content(typed_sum3_def, '.py') 
```

```py
def sum3(a: Any, b: Any, c: Any) -> Any:
    return a + b + c

```

类型 `Any` 明确表示一个对象确实可以具有任何类型；它既不会在运行时也不会在静态时进行类型检查。在某种程度上，这削弱了类型检查的力量；但它也保留了一些许多 Python 程序员所享受的类型灵活性。除了 `Any` 之外，`typing` 模块还支持定义模糊类型的几种额外方式；我们将记住这些内容以备后续练习。

## 指定和检查不变量

除了基本数据类型之外，我们还可以从参数检查几个其他属性。例如，我们可以检查一个参数是否可以是负数、零或正数；或者一个参数应该比第二个参数小；或者结果应该是两个参数的和——这些属性在（Python）类型中无法表达。

这样的属性被称为 *不变量*，因为它们在函数的所有调用中都保持不变。具体来说，不变量以 *前置* 和 *后置条件* 的形式出现——这些条件在函数的开始和结束时始终成立。（还有 *数据* 和 *对象* 不变量，它们表达数据或对象状态上始终成立的属性，但在这本书中我们不考虑这些。） 

### 使用前置和后置条件注释函数

指定前置和后置条件的经典方法是通过 *断言*，这在我们在 测试章节 中已经介绍过了。前置条件检查函数的参数是否满足预期的属性；后置条件对结果做同样的检查。我们可以使用断言如下表达和检查：

```py
def my_sqrt_with_invariants(x):
    assert x >= 0  # Precondition

    ...

    assert result * result == x  # Postcondition
    return result 
```

然而，一种更优雅的方法是在语法上将不变量与当前函数分开。使用适当的装饰器，我们可以如下指定前置和后置条件：

```py
@precondition lambda x: x >= 0
@postcondition lambda return_value, x: return_value * return_value == x
def my_sqrt_with_invariants(x):
    # normal code without assertions
    ... 
```

装饰器 `@precondition` 和 `@postcondition` 分别在装饰函数之前和之后运行给定的函数（指定为匿名 `lambda` 函数）。如果函数返回 `False`，则条件被违反。`@precondition` 获取函数参数作为参数；`@postcondition` 还额外获取返回值作为第一个参数。

实现这样的装饰器并不困难。我们的实现基于一个来自 StackOverflow 的 [代码片段](https://stackoverflow.com/questions/12151182/python-precondition-postcondition-for-member-function-how)：

```py
import [functools](https://docs.python.org/3/library/functools.html) 
```

```py
def condition(precondition=None, postcondition=None):
    def decorator(func):
        @functools.wraps(func) # preserves name, docstring, etc
        def wrapper(*args, **kwargs):
            if precondition is not None:
               assert precondition(*args, **kwargs), "Precondition violated"

            retval = func(*args, **kwargs) # call original function or method
            if postcondition is not None:
               assert postcondition(retval, *args, **kwargs), "Postcondition violated"

            return retval
        return wrapper
    return decorator

def precondition(check):
    return condition(precondition=check)

def postcondition(check):
    return condition(postcondition=check) 
```

使用这些，我们现在可以开始装饰 `my_sqrt()`：

```py
@precondition(lambda x: x > 0)
def my_sqrt_with_precondition(x):
    return my_sqrt(x) 
```

这会捕获违反前置条件的参数：

```py
with ExpectError():
    my_sqrt_with_precondition(-1.0) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2353876897.py", line 2, in <module>
    my_sqrt_with_precondition(-1.0)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 6, in wrapper
    assert precondition(*args, **kwargs), "Precondition violated"
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: Precondition violated (expected)

```

同样，我们可以提供一个后置条件：

```py
EPSILON = 1e-5 
```

```py
@postcondition(lambda ret, x: ret * ret - x < EPSILON)
def my_sqrt_with_postcondition(x):
    return my_sqrt(x) 
```

```py
y = my_sqrt_with_postcondition(2.0)
y 
```

```py
1.414213562373095

```

如果我们有一个有错误的 $\sqrt{x}$ 实现，这将很快被发现：

```py
@postcondition(lambda ret, x: ret * ret - x < EPSILON)
def buggy_my_sqrt_with_postcondition(x):
    return my_sqrt(x) + 0.1 
```

```py
with ExpectError():
    y = buggy_my_sqrt_with_postcondition(2.0) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/1985029262.py", line 2, in <module>
    y = buggy_my_sqrt_with_postcondition(2.0)
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 10, in wrapper
    assert postcondition(retval, *args, **kwargs), "Postcondition violated"
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: Postcondition violated (expected)

```

虽然检查前置条件和后置条件是捕获错误的好方法，但指定它们可能会很繁琐。让我们看看我们是否可以（再次）*挖掘*其中的一些。

## 矿山不变量

要 *挖掘* 不变量，我们可以使用之前相同的跟踪功能；不过，我们不是保存单个变量的值，而是现在检查值是否满足特定的 *属性*。例如，如果我们看到的所有 `x` 的值都满足条件 `x > 0`，那么我们将 `x > 0` 作为函数的不变量。如果我们看到 `x` 的正值、零值和负值，那么关于 `x` 就没有属性可以讨论了。

因此，一般思路是：

1.  将观察到的所有变量值与一组预定义的属性进行比较；并且

1.  只保留对所有观察到的运行都成立的属性。

### 定义属性

我们所说的属性究竟是什么意思？这里有一小部分经常用于不变量的值属性。所有这些属性都将使用 *元变量* `X`、`Y` 和 `Z`（实际上，任何大写标识符）替换为函数参数的名称：

```py
INVARIANT_PROPERTIES = [
    "X < 0",
    "X <= 0",
    "X > 0",
    "X >= 0",
    "X == 0",
    "X != 0",
] 
```

当 `my_sqrt(x)` 被调用，比如 `my_sqrt(5.0)` 时，我们看到 `x = 5.0` 成立。然后，上述属性将全部针对 `x` 进行检查。只有 `X > 0`、`X >= 0` 和 `X != 0` 这三个属性适用于观察到的调用；因此 `x > 0`、`x >= 0` 和 `x != 0` 将成为 `my_sqrt(x)` 的潜在前置条件。

我们可以检查许多其他属性，例如两个参数之间的关系：

```py
INVARIANT_PROPERTIES += [
    "X == Y",
    "X > Y",
    "X < Y",
    "X >= Y",
    "X <= Y",
] 
```

类型也可以通过属性进行检查。对于任何函数参数 `X`，以下其中之一将成立：

```py
INVARIANT_PROPERTIES += [
    "isinstance(X, bool)",
    "isinstance(X, int)",
    "isinstance(X, float)",
    "isinstance(X, list)",
    "isinstance(X, dict)",
] 
```

我们可以检查算术属性：

```py
INVARIANT_PROPERTIES += [
    "X == Y + Z",
    "X == Y * Z",
    "X == Y - Z",
    "X == Y / Z",
] 
```

这里是三个值之间的关系，一个 Python 特殊情况：

```py
INVARIANT_PROPERTIES += [
    "X < Y < Z",
    "X <= Y <= Z",
    "X > Y > Z",
    "X >= Y >= Z",
] 
```

最后，我们还可以检查列表或字符串属性。这只是一个很小的选择。

```py
INVARIANT_PROPERTIES += [
    "X == len(Y)",
    "X == sum(Y)",
    "X.startswith(Y)",
] 
```

### 提取元变量

在我们进行实际挖掘之前，让我们先介绍一些 *辅助函数*。`metavars()` 从属性中提取元变量集（`X`、`Y`、`Z` 等）。为此，我们将属性解析为 Python 表达式，然后访问标识符。

```py
def metavars(prop):
    metavar_list = []

    class ArgVisitor(ast.NodeVisitor):
        def visit_Name(self, node):
            if node.id.isupper():
                metavar_list.append(node.id)

    ArgVisitor().visit(ast.parse(prop))
    return metavar_list 
```

```py
assert metavars("X < 0") == ['X'] 
```

```py
assert metavars("X.startswith(Y)") == ['X', 'Y'] 
```

```py
assert metavars("isinstance(X, str)") == ['X'] 
```

### 实例化属性

要将属性作为不变量产生，我们需要能够用变量名 *实例化* 它。例如，将 `X > 0` 实例化为 `X` 为 `a` 的情况，得到 `a > 0`。为此，函数 `instantiate_prop()` 接受一个属性和一组变量名，并将元变量从左到右与集合中的相应变量名实例化。

```py
def instantiate_prop_ast(prop, var_names):
    class NameTransformer(ast.NodeTransformer):
        def visit_Name(self, node):
            if node.id not in mapping:
                return node
            return ast.Name(id=mapping[node.id], ctx=ast.Load())

    meta_variables = metavars(prop)
    assert len(meta_variables) == len(var_names)

    mapping = {}
    for i in range(0, len(meta_variables)):
        mapping[meta_variables[i]] = var_names[i]

    prop_ast = ast.parse(prop, mode='eval')
    new_ast = NameTransformer().visit(prop_ast)

    return new_ast 
```

```py
def instantiate_prop(prop, var_names):
    prop_ast = instantiate_prop_ast(prop, var_names)
    prop_text = ast.unparse(prop_ast).strip()
    while prop_text.startswith('(') and prop_text.endswith(')'):
        prop_text = prop_text[1:-1]
    return prop_text 
```

```py
assert instantiate_prop("X > Y", ['a', 'b']) == 'a > b' 
```

```py
assert instantiate_prop("X.startswith(Y)", ['x', 'y']) == 'x.startswith(y)' 
```

### 评估属性

要实际 *评估* 属性，我们不需要实例化它们。相反，我们只需将它们转换为布尔函数，使用 `lambda`：

```py
def prop_function_text(prop):
    return "lambda " + ", ".join(metavars(prop)) + ": " + prop

def prop_function(prop):
    return eval(prop_function_text(prop)) 
```

这里有一个简单的例子：

```py
prop_function_text("X > Y") 
```

```py
'lambda X, Y: X > Y'

```

```py
p = prop_function("X > Y")
p(100, 1) 
```

```py
True

```

```py
p(1, 100) 
```

```py
False

```

### 检查不变量

要从执行中提取不变量，我们需要检查所有可能的参数实例。如果要检查的函数有两个参数 `a` 和 `b`，我们既将属性 `X < Y` 实例化为 `a < b`，也实例化为 `b < a`，并检查每一个。

要获取所有组合，我们使用 Python 的 `permutations()` 函数：

```py
import [itertools](https://docs.python.org/3/library/itertools.html) 
```

```py
for combination in itertools.permutations([1.0, 2.0, 3.0], 2):
    print(combination) 
```

```py
(1.0, 2.0)
(1.0, 3.0)
(2.0, 1.0)
(2.0, 3.0)
(3.0, 1.0)
(3.0, 2.0)

```

函数 `true_property_instantiations()` 接受一个属性和一个元组列表 (`var_name`, `value`)。然后，它产生所有给定值的属性实例，并返回那些评估为 True 的实例。

```py
def true_property_instantiations(prop, vars_and_values, log=False):
    instantiations = set()
    p = prop_function(prop)

    len_metavars = len(metavars(prop))
    for combination in itertools.permutations(vars_and_values, len_metavars):
        args = [value for var_name, value in combination]
        var_names = [var_name for var_name, value in combination]

        try:
            result = p(*args)
        except:
            result = None

        if log:
            print(prop, combination, result)
        if result:
            instantiations.add((prop, tuple(var_names)))

    return instantiations 
```

这里有一个例子。如果 `x == -1` 和 `y == 1`，属性 `X < Y` 对 `x < y` 成立，但对 `y < x` 不成立：

```py
invs = true_property_instantiations("X < Y", [('x', -1), ('y', 1)], log=True)
invs 
```

```py
X < Y (('x', -1), ('y', 1)) True
X < Y (('y', 1), ('x', -1)) False

```

```py
{('X < Y', ('x', 'y'))}

```

实例化检索到简短形式：

```py
for prop, var_names in invs:
    print(instantiate_prop(prop, var_names)) 
```

```py
x < y

```

同样，对于 `x` 和 `y` 的值如上所述，属性 `X < 0` 仅对 `x` 成立，但对 `y` 不成立：

```py
invs = true_property_instantiations("X < 0", [('x', -1), ('y', 1)], log=True) 
```

```py
X < 0 (('x', -1),) True
X < 0 (('y', 1),) False

```

```py
for prop, var_names in invs:
    print(instantiate_prop(prop, var_names)) 
```

```py
x < 0

```

### 提取不变量

现在我们将上述不变量提取应用于函数执行期间观察到的函数参数和返回值。为此，我们将 `CallTracker` 类扩展为 `InvariantTracker` 类，该类自动计算所有函数和跟踪期间观察到的所有调用的不变量。

默认情况下，`InvariantTracker` 使用上面定义的属性；然而，可以指定不同的属性集。

```py
class InvariantTracker(CallTracker):
    def __init__(self, props=None, **kwargs):
        if props is None:
            props = INVARIANT_PROPERTIES

        self.props = props
        super().__init__(**kwargs) 
```

`InvariantTracker` 的关键方法是 `invariants()` 方法。该方法遍历观察到的调用并检查哪些属性成立。仅保留属性的交集——即对所有调用都成立的属性集合——并最终返回。特殊变量 `return_value` 被设置为保留返回值。

```py
RETURN_VALUE = 'return_value' 
```

```py
class InvariantTracker(InvariantTracker):
    def invariants(self, function_name=None):
        if function_name is None:
            return {function_name: self.invariants(function_name) for function_name in self.calls()}

        invariants = None
        for variables, return_value in self.calls(function_name):
            vars_and_values = variables + [(RETURN_VALUE, return_value)]

            s = set()
            for prop in self.props:
                s |= true_property_instantiations(prop, vars_and_values, self._log)
            if invariants is None:
                invariants = s
            else:
                invariants &= s

        return invariants 
```

这里是使用 `invariants()` 的一个示例。我们在一组小的调用上运行跟踪器。

```py
with InvariantTracker() as tracker:
    y = my_sqrt(25.0)
    y = my_sqrt(10.0)

tracker.calls() 
```

```py
{'my_sqrt': [([('x', 25.0)], 5.0), ([('x', 10.0)], 3.162277660168379)]}

```

`invariants()` 方法生成一组属性，这些属性对于观察到的运行成立，以及它们在函数参数上的实例化。

```py
invs = tracker.invariants('my_sqrt')
invs 
```

```py
{('X != 0', ('return_value',)),
 ('X != 0', ('x',)),
 ('X < Y', ('return_value', 'x')),
 ('X <= Y', ('return_value', 'x')),
 ('X > 0', ('return_value',)),
 ('X > 0', ('x',)),
 ('X > Y', ('x', 'return_value')),
 ('X >= 0', ('return_value',)),
 ('X >= 0', ('x',)),
 ('X >= Y', ('x', 'return_value')),
 ('isinstance(X, float)', ('return_value',)),
 ('isinstance(X, float)', ('x',))}

```

如前所述，实际的实例化更容易阅读：

```py
def pretty_invariants(invariants):
    props = []
    for (prop, var_names) in invariants:
        props.append(instantiate_prop(prop, var_names))
    return sorted(props) 
```

```py
pretty_invariants(invs) 
```

```py
['isinstance(return_value, float)',
 'isinstance(x, float)',
 'return_value != 0',
 'return_value < x',
 'return_value <= x',
 'return_value > 0',
 'return_value >= 0',
 'x != 0',
 'x > 0',
 'x > return_value',
 'x >= 0',
 'x >= return_value']

```

我们可以看到，`x` 和返回值都具有 `float` 类型。我们还可以看到，它们总是大于零。这些可能是有用的前置和后置条件，特别是对于符号分析。

然而，也存在一个不普遍成立的不变量，即 `return_value <= x`，如下例所示：

```py
my_sqrt(0.01) 
```

```py
0.1

```

显然，0.1 > 0.01 成立。这是一个我们没有从足够多样化的输入中学习的例子。但是，一旦我们有一个包含 `x = 0.1` 的调用，不变量 `return_value <= x` 就被消除了：

```py
with InvariantTracker() as tracker:
    y = my_sqrt(25.0)
    y = my_sqrt(10.0)
    y = my_sqrt(0.01)

pretty_invariants(tracker.invariants('my_sqrt')) 
```

```py
['isinstance(return_value, float)',
 'isinstance(x, float)',
 'return_value != 0',
 'return_value > 0',
 'return_value >= 0',
 'x != 0',
 'x > 0',
 'x >= 0']

```

我们将在稍后讨论如何确保输入的多样性足够。（提示：这涉及到测试生成。）

让我们尝试使用我们的不变量跟踪器对 `sum3()` 进行测试。我们看到所有类型都定义良好；然而，所有参数都是非零的属性是针对观察到的调用特定的。

```py
with InvariantTracker() as tracker:
    y = sum3(1, 2, 3)
    y = sum3(-4, -5, -6)

pretty_invariants(tracker.invariants('sum3')) 
```

```py
['a != 0',
 'b != 0',
 'c != 0',
 'isinstance(a, int)',
 'isinstance(b, int)',
 'isinstance(c, int)',
 'isinstance(return_value, int)',
 'return_value != 0']

```

如果我们用字符串而不是数字调用 `sum3()`，我们会得到不同的不变量。值得注意的是，我们获得了返回值以 `a` 的值为起始值的后条件——如果使用字符串，这是一个普遍的后条件。

```py
with InvariantTracker() as tracker:
    y = sum3('a', 'b', 'c')
    y = sum3('f', 'e', 'd')

pretty_invariants(tracker.invariants('sum3')) 
```

```py
['a != 0',
 'a < return_value',
 'a <= return_value',
 'b != 0',
 'c != 0',
 'return_value != 0',
 'return_value > a',
 'return_value >= a',
 'return_value.startswith(a)']

```

如果我们用字符串和数字（以及零）调用 `sum3()`，就没有留下任何在所有调用中都成立的属性。这就是灵活性的代价。

```py
with InvariantTracker() as tracker:
    y = sum3('a', 'b', 'c')
    y = sum3('c', 'b', 'a')
    y = sum3(-4, -5, -6)
    y = sum3(0, 0, 0)

pretty_invariants(tracker.invariants('sum3')) 
```

```py
[]

```

### 将挖掘的不变量转换为注释

与类型一样，我们希望有一些功能，我们可以将挖掘的不变量作为注释添加到现有的函数中。为此，我们引入了 `InvariantAnnotator` 类，它扩展了 `InvariantTracker`。

我们从一个辅助方法开始。`params()` 返回在调用过程中观察到的参数名称的逗号分隔列表。

```py
class InvariantAnnotator(InvariantTracker):
    def params(self, function_name):
        arguments, return_value = self.calls(function_name)[0]
        return ", ".join(arg_name for (arg_name, arg_value) in arguments) 
```

```py
with InvariantAnnotator() as annotator:
    y = my_sqrt(25.0)
    y = sum3(1, 2, 3) 
```

```py
annotator.params('my_sqrt') 
```

```py
'x'

```

```py
annotator.params('sum3') 
```

```py
'c, b, a'

```

现在是实际的注释。`preconditions()` 返回从挖掘的不变量中获取的前条件（即，那些不依赖于返回值的属性），作为一个带有注释的字符串：

```py
class InvariantAnnotator(InvariantAnnotator):
    def preconditions(self, function_name):
        conditions = []

        for inv in pretty_invariants(self.invariants(function_name)):
            if inv.find(RETURN_VALUE) >= 0:
                continue  # Postcondition

            cond = "@precondition(lambda " + self.params(function_name) + ": " + inv + ")"
            conditions.append(cond)

        return conditions 
```

```py
with InvariantAnnotator() as annotator:
    y = my_sqrt(25.0)
    y = my_sqrt(0.01)
    y = sum3(1, 2, 3) 
```

```py
annotator.preconditions('my_sqrt') 
```

```py
['@precondition(lambda x: isinstance(x, float))',
 '@precondition(lambda x: x != 0)',
 '@precondition(lambda x: x > 0)',
 '@precondition(lambda x: x >= 0)']

```

`postconditions()` 对后条件做同样的处理：

```py
class InvariantAnnotator(InvariantAnnotator):
    def postconditions(self, function_name):
        conditions = []

        for inv in pretty_invariants(self.invariants(function_name)):
            if inv.find(RETURN_VALUE) < 0:
                continue  # Precondition

            cond = ("@postcondition(lambda " + 
                RETURN_VALUE + ", " + self.params(function_name) + ": " + inv + ")")
            conditions.append(cond)

        return conditions 
```

```py
with InvariantAnnotator() as annotator:
    y = my_sqrt(25.0)
    y = my_sqrt(0.01)
    y = sum3(1, 2, 3) 
```

```py
annotator.postconditions('my_sqrt') 
```

```py
['@postcondition(lambda return_value, x: isinstance(return_value, float))',
 '@postcondition(lambda return_value, x: return_value != 0)',
 '@postcondition(lambda return_value, x: return_value > 0)',
 '@postcondition(lambda return_value, x: return_value >= 0)']

```

使用这些，我们可以将一个函数作为注释添加前条件和后条件：

```py
class InvariantAnnotator(InvariantAnnotator):
    def functions_with_invariants(self):
        functions = ""
        for function_name in self.invariants():
            try:
                function = self.function_with_invariants(function_name)
            except KeyError:
                continue
            functions += function
        return functions

    def function_with_invariants(self, function_name):
        function = globals()[function_name]  # Can throw KeyError
        source = inspect.getsource(function)
        return "\n".join(self.preconditions(function_name) + 
                         self.postconditions(function_name)) + '\n' + source 
```

下面是 `function_with_invariants()` 在其所有荣耀中的样子：

```py
with InvariantAnnotator() as annotator:
    y = my_sqrt(25.0)
    y = my_sqrt(0.01)
    y = sum3(1, 2, 3) 
```

```py
print_content(annotator.function_with_invariants('my_sqrt'), '.py') 
```

```py
@precondition(lambda x: isinstance(x, float))
@precondition(lambda x: x != 0)
@precondition(lambda x: x > 0)
@precondition(lambda x: x >= 0)
@postcondition(lambda return_value, x: isinstance(return_value, float))
@postcondition(lambda return_value, x: return_value != 0)
@postcondition(lambda return_value, x: return_value > 0)
@postcondition(lambda return_value, x: return_value >= 0)
def my_sqrt(x):
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

很多的不变量，是吗？在下面（以及在练习中），我们将讨论如何关注最相关的属性。

### 一些例子

这里还有一个例子。`list_length()` 递归地计算一个 Python 函数的长度。让我们看看我们是否可以挖掘其不变量：

```py
def list_length(L):
    if L == []:
        length = 0
    else:
        length = 1 + list_length(L[1:])
    return length 
```

```py
with InvariantAnnotator() as annotator:
    length = list_length([1, 2, 3])

print_content(annotator.functions_with_invariants(), '.py') 
```

```py
@precondition(lambda L: L != 0)
@precondition(lambda L: isinstance(L, list))
@postcondition(lambda return_value, L: isinstance(return_value, int))
@postcondition(lambda return_value, L: return_value == len(L))
@postcondition(lambda return_value, L: return_value >= 0)
def list_length(L):
    if L == []:
        length = 0
    else:
        length = 1 + list_length(L[1:])
    return length

```

几乎所有这些属性（除了第一个）都是相关的。当然，不变量之所以如此整洁，是因为返回值等于 `len(L)`，以及 `X == len(Y)` 是要检查的属性列表的一部分。

下一个例子是一个非常简单的函数：

```py
def sum2(a, b):
    return a + b 
```

```py
with InvariantAnnotator() as annotator:
    sum2(31, 45)
    sum2(0, 0)
    sum2(-1, -5) 
```

所有不变量都捕捉了 `a`、`b` 和返回值之间的关系，即 `return_value == a + b` 在所有变化中。

```py
print_content(annotator.functions_with_invariants(), '.py') 
```

```py
@precondition(lambda b, a: isinstance(a, int))
@precondition(lambda b, a: isinstance(b, int))
@postcondition(lambda return_value, b, a: a == return_value - b)
@postcondition(lambda return_value, b, a: b == return_value - a)
@postcondition(lambda return_value, b, a: isinstance(return_value, int))
@postcondition(lambda return_value, b, a: return_value == a + b)
@postcondition(lambda return_value, b, a: return_value == b + a)
def sum2(a, b):
    return a + b

```

如果我们有一个没有返回值的函数，返回值是 `None`，我们只能挖掘前条件。（好吧，我们得到一个“后条件”，即返回值是非零的，这对于 `None` 是成立的。）

```py
def print_sum(a, b):
    print(a + b) 
```

```py
with InvariantAnnotator() as annotator:
    print_sum(31, 45)
    print_sum(0, 0)
    print_sum(-1, -5) 
```

```py
76
0
-6

```

```py
print_content(annotator.functions_with_invariants(), '.py') 
```

```py
@precondition(lambda b, a: isinstance(a, int))
@precondition(lambda b, a: isinstance(b, int))
@postcondition(lambda return_value, b, a: return_value != 0)
def print_sum(a, b):
    print(a + b)

```

### 检查规范

如上所示，具有不变量的函数可以输入到 Python 解释器中，以便检查所有前条件和后条件。我们创建了一个函数 `my_sqrt_annotated()`，它包括上面挖掘的所有不变量。

```py
with InvariantAnnotator() as annotator:
    y = my_sqrt(25.0)
    y = my_sqrt(0.01) 
```

```py
my_sqrt_def = annotator.functions_with_invariants()
my_sqrt_def = my_sqrt_def.replace('my_sqrt', 'my_sqrt_annotated') 
```

```py
print_content(my_sqrt_def, '.py') 
```

```py
@precondition(lambda x: isinstance(x, float))
@precondition(lambda x: x != 0)
@precondition(lambda x: x > 0)
@precondition(lambda x: x >= 0)
@postcondition(lambda return_value, x: isinstance(return_value, float))
@postcondition(lambda return_value, x: return_value != 0)
@postcondition(lambda return_value, x: return_value > 0)
@postcondition(lambda return_value, x: return_value >= 0)
def my_sqrt_annotated(x):
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

```

```py
exec(my_sqrt_def) 
```

“注释”版本检查无效的参数——或者更精确地说，检查尚未观察到的属性的参数：

```py
with ExpectError():
    my_sqrt_annotated(-1.0) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/3390953352.py", line 2, in <module>
    my_sqrt_annotated(-1.0)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 8, in wrapper
    retval = func(*args, **kwargs) # call original function or method
             ^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 8, in wrapper
    retval = func(*args, **kwargs) # call original function or method
             ^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 6, in wrapper
    assert precondition(*args, **kwargs), "Precondition violated"
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: Precondition violated (expected)

```

这与原始版本形成对比，原始版本只是对负值挂起：

```py
with ExpectTimeout(1):
    my_sqrt(-1.0) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2605599394.py", line 2, in <module>
    my_sqrt(-1.0)
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2661069967.py", line 5, in my_sqrt
    while approx != guess:
          ^^^^^^^^^^^^^^^
  File "Timeout.ipynb", line 43, in timeout_handler
    raise TimeoutError()
TimeoutError (expected)

```

如果我们对函数定义进行修改，使得返回值的属性发生变化，那么这种*回归*会被视为违反了后置条件。让我们通过简单地反转结果，将$-2$作为 4 的平方根来举例说明。

```py
my_sqrt_def = my_sqrt_def.replace('my_sqrt_annotated', 'my_sqrt_negative')
my_sqrt_def = my_sqrt_def.replace('return approx', 'return -approx') 
```

```py
print_content(my_sqrt_def, '.py') 
```

```py
@precondition(lambda x: isinstance(x, float))
@precondition(lambda x: x != 0)
@precondition(lambda x: x > 0)
@precondition(lambda x: x >= 0)
@postcondition(lambda return_value, x: isinstance(return_value, float))
@postcondition(lambda return_value, x: return_value != 0)
@postcondition(lambda return_value, x: return_value > 0)
@postcondition(lambda return_value, x: return_value >= 0)
def my_sqrt_negative(x):
  """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return -approx

```

```py
exec(my_sqrt_def) 
```

从技术上来说，$-2$确实是 4 的平方根，因为$(-2)² = 4$成立。然而，这种更改可能对`my_sqrt()`的调用者来说是出乎意料的，因此，这将在第一次调用时被发现：

```py
with ExpectError():
    my_sqrt_negative(2.0) 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2428286732.py", line 2, in <module>
    my_sqrt_negative(2.0)  # type: ignore
    ^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 8, in wrapper
    retval = func(*args, **kwargs) # call original function or method
             ^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 8, in wrapper
    retval = func(*args, **kwargs) # call original function or method
             ^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 8, in wrapper
    retval = func(*args, **kwargs) # call original function or method
             ^^^^^^^^^^^^^^^^^^^^^
  [Previous line repeated 4 more times]
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 10, in wrapper
    assert postcondition(retval, *args, **kwargs), "Postcondition violated"
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: Postcondition violated (expected)

```

我们看到前置和后置条件，以及类型，如何在测试期间充当*占卜者*。特别是，一旦我们为一系列函数挖掘了它们，我们就可以用测试生成器反复检查它们——尤其是在代码更改之后。检查越多，越具体，我们检测到变化带来的不期望效果的可能性就越大。

## 从生成的测试中挖掘规范

从挖掘出的执行中，挖掘出的规范只能达到那样的水平。如果我们只看到像上面定义的`sum2()`这样的单个调用，我们将面临几个挖掘出的前置和后置条件，这些条件*过度专业化*于观察到的值：

```py
with InvariantAnnotator() as annotator:
    y = sum2(2, 2)
print_content(annotator.functions_with_invariants(), '.py') 
```

```py
@precondition(lambda b, a: a != 0)
@precondition(lambda b, a: a <= b)
@precondition(lambda b, a: a == b)
@precondition(lambda b, a: a > 0)
@precondition(lambda b, a: a >= 0)
@precondition(lambda b, a: a >= b)
@precondition(lambda b, a: b != 0)
@precondition(lambda b, a: b <= a)
@precondition(lambda b, a: b == a)
@precondition(lambda b, a: b > 0)
@precondition(lambda b, a: b >= 0)
@precondition(lambda b, a: b >= a)
@precondition(lambda b, a: isinstance(a, int))
@precondition(lambda b, a: isinstance(b, int))
@postcondition(lambda return_value, b, a: a < return_value)
@postcondition(lambda return_value, b, a: a <= b <= return_value)
@postcondition(lambda return_value, b, a: a <= return_value)
@postcondition(lambda return_value, b, a: a == return_value - b)
@postcondition(lambda return_value, b, a: a == return_value / b)
@postcondition(lambda return_value, b, a: b < return_value)
@postcondition(lambda return_value, b, a: b <= a <= return_value)
@postcondition(lambda return_value, b, a: b <= return_value)
@postcondition(lambda return_value, b, a: b == return_value - a)
@postcondition(lambda return_value, b, a: b == return_value / a)
@postcondition(lambda return_value, b, a: isinstance(return_value, int))
@postcondition(lambda return_value, b, a: return_value != 0)
@postcondition(lambda return_value, b, a: return_value == a * b)
@postcondition(lambda return_value, b, a: return_value == a + b)
@postcondition(lambda return_value, b, a: return_value == b * a)
@postcondition(lambda return_value, b, a: return_value == b + a)
@postcondition(lambda return_value, b, a: return_value > 0)
@postcondition(lambda return_value, b, a: return_value > a)
@postcondition(lambda return_value, b, a: return_value > b)
@postcondition(lambda return_value, b, a: return_value >= 0)
@postcondition(lambda return_value, b, a: return_value >= a)
@postcondition(lambda return_value, b, a: return_value >= a >= b)
@postcondition(lambda return_value, b, a: return_value >= b)
@postcondition(lambda return_value, b, a: return_value >= b >= a)
def sum2(a, b):
    return a + b

```

例如，挖掘出的前置条件`a == b`仅适用于观察到的单个调用；同样的，挖掘出的后置条件`return_value == a * b`也是如此。然而，`sum2()`显然可以用其他不满足这些条件的值成功调用。

为了摆脱这个陷阱，我们必须从越来越多样化的运行中学习。如果我们有更多`sum2()`的调用，我们会看到不变量的集合如何迅速缩小：

```py
with InvariantAnnotator() as annotator:
    length = sum2(1, 2)
    length = sum2(-1, -2)
    length = sum2(0, 0)

print_content(annotator.functions_with_invariants(), '.py') 
```

```py
@precondition(lambda b, a: isinstance(a, int))
@precondition(lambda b, a: isinstance(b, int))
@postcondition(lambda return_value, b, a: a == return_value - b)
@postcondition(lambda return_value, b, a: b == return_value - a)
@postcondition(lambda return_value, b, a: isinstance(return_value, int))
@postcondition(lambda return_value, b, a: return_value == a + b)
@postcondition(lambda return_value, b, a: return_value == b + a)
def sum2(a, b):
    return a + b

```

但我们从哪里得到这样多样化的运行呢？这是生成软件测试的工作。一个简单的`sum2()`调用语法将很容易解决这个问题。

```py
from GrammarFuzzer import GrammarFuzzer  # minor dependency
from Grammars import is_valid_grammar, crange  # minor dependency
from Grammars import convert_ebnf_grammar, Grammar  # minor dependency 
```

```py
SUM2_EBNF_GRAMMAR: Grammar = {
    "<start>": ["<sum2>"],
    "<sum2>": ["sum2(<int>, <int>)"],
    "<int>": ["<_int>"],
    "<_int>": ["(-)?<leaddigit><digit>*", "0"],
    "<leaddigit>": crange('1', '9'),
    "<digit>": crange('0', '9')
} 
```

```py
assert is_valid_grammar(SUM2_EBNF_GRAMMAR) 
```

```py
sum2_grammar =  convert_ebnf_grammar(SUM2_EBNF_GRAMMAR) 
```

```py
sum2_fuzzer = GrammarFuzzer(sum2_grammar)
[sum2_fuzzer.fuzz() for i in range(10)] 
```

```py
['sum2(60, 3)',
 'sum2(-4, 0)',
 'sum2(-579, 34)',
 'sum2(3, 0)',
 'sum2(-8, 0)',
 'sum2(0, 8)',
 'sum2(3, -9)',
 'sum2(0, 0)',
 'sum2(0, 5)',
 'sum2(-3181, 0)']

```

```py
with InvariantAnnotator() as annotator:
    for i in range(10):
        eval(sum2_fuzzer.fuzz())

print_content(annotator.function_with_invariants('sum2'), '.py') 
```

```py
@precondition(lambda b, a: a != 0)
@precondition(lambda b, a: isinstance(a, int))
@precondition(lambda b, a: isinstance(b, int))
@postcondition(lambda return_value, b, a: a == return_value - b)
@postcondition(lambda return_value, b, a: b == return_value - a)
@postcondition(lambda return_value, b, a: isinstance(return_value, int))
@postcondition(lambda return_value, b, a: return_value != 0)
@postcondition(lambda return_value, b, a: return_value == a + b)
@postcondition(lambda return_value, b, a: return_value == b + a)
def sum2(a, b):
    return a + b

```

但然后，仅仅为了推导出一组前置和后置条件而编写测试（或测试驱动程序）可能太过费力——特别是，因为测试可以很容易地从给定的前置和后置条件中推导出来。因此，首先指定不变量，然后让测试生成器或程序证明者来完成这项工作会更明智。

此外，API 语法，如上所述，必须设置得能够真正尊重前置条件——在我们的例子中，我们只使用正数调用`sqrt()`，已经假设了它的前置条件。在某种程度上，因此需要一种规范（一个模型，一个语法）来挖掘另一种规范——这是一个鸡生蛋的问题。

然而，有一种方法可以解决这个问题：如果能够在系统级别自动生成测试，那么就有一个*无限的执行来源*来学习恒等式。在这些执行中的每一个，所有函数都会使用满足（隐式）先决条件的值被调用，这使我们能够为这些函数挖掘恒等式。这是因为在系统级别，无效的输入首先必须被系统拒绝。系统级别的有意义的先决条件，确保只有有效的输入通过，因此被分解为许多有意义的先决条件（以及随后的后置条件）在函数级别。

然而，对于这一点来说，一个重要的要求是，需要在系统级别有好的测试生成器。在下一部分中，我们将讨论如何自动为各种领域生成测试，从配置到图形用户界面。

## 经验教训

+   类型注解和显式恒等式允许检查参数和结果以期望的数据类型和其他属性。

+   可以通过观察运行时的参数和结果自动挖掘数据类型和恒等式。

+   矿掘出的恒等式的质量取决于执行过程中观察到的值的多样性；这种多样性可以通过生成测试来增加。

## 下一步

本章总结了关于语义模糊测试技术的部分。在下一部分，我们将探讨从配置和 API 到图形用户界面的特定领域模糊测试技术。

## 背景

[DAIKON 动态不变检测器](https://plse.cs.washington.edu/daikon/)可以被认为是函数规范挖掘器的鼻祖。经过 20 多年的持续维护和扩展，它以本章所述的风格为包括 C、C++、C#、Eiffel、F#、Java、Perl 和 Visual Basic 在内的多种语言挖掘可能的恒等式。除了上述功能外，它还拥有大量可能的恒等式模式库，支持数据恒等式，可以消除由其他恒等式暗示的恒等式，并确定统计置信度以忽略不太可能的恒等式。相应的论文[[Ernst 等人，2001](https://doi.org/10.1109/32.908957)]是软件工程领域的开创性且被引用次数最多的论文之一。基于 DAIKON 和检测恒等式，已经发表了大量作品；请参阅此[精选列表](http://plse.cs.washington.edu/daikon/pubs/)以获取详细信息。

测试生成器与不变性检测之间的交互已在 [[Ernst 等人，2001](https://doi.org/10.1109/32.908957)] 中讨论（顺便提一下，也使用了语法）。Eclat 工具 [[Pacheco 等人，2005](https://doi.org/10.1007/11531142_22)] 是单元级测试生成器与 DAIKON 风格的不变性挖掘之间紧密交互的一个模型示例，其中挖掘的不变性被用来生成或 acles 并系统地引导测试生成器向故障揭示输入。

挖掘规范不仅限于前置条件和后置条件。论文《挖掘规范》[[Ammons 等人，2002](https://doi.org/10.1145/503272.503275)]是该领域的另一篇经典之作，它从执行中学习状态协议。正如我们在同名章节中描述的语法挖掘也可以被视为一种规范挖掘方法，这次是学习输入格式的规范。

当涉及到向现有代码添加类型注解时，博客文章["Python 中类型提示的状态"](https://www.bernat.tech/the-state-of-type-hints-in-python/)提供了关于如何使用和检查 Python 类型提示的极佳概述。要添加类型注解，有两个重要的工具可用，它们也实现了我们上述的方法：

+   [MonkeyType](https://instagram-engineering.com/let-your-code-type-hint-itself-introducing-open-source-monkeytype-a855c7284881) 实现了上述跟踪执行并使用类型提示注解 Python 3 参数、返回值和变量的方法。

+   [PyAnnotate](https://github.com/dropbox/pyannotate) 执行类似的工作，专注于 Python 2 中的代码。它不会生成 Python 3 风格的注解，而是生成可以作为注释处理的注解，这些注释可以被静态类型检查器处理。

这些工具是由 Facebook 和 Dropbox 的工程师创建的，分别帮助他们检查数百万行代码中的类型问题。

## 练习

我们挖掘类型和不变量的代码远非完整。有许多方法可以扩展我们的实现，其中一些我们在练习中进行了讨论。

### 练习 1：联合类型

Python 的 `typing` 模块允许表达一个参数可以有多种类型。对于 `my_sqrt(x)`，这允许表达 `x` 可以是 `int` 或 `float`：

```py
from [typing](https://docs.python.org/3/library/typing.html) import Union, Optional 
```

```py
def my_sqrt_with_union_type(x: Union[int, float]) -> float: 
    ... 
```

扩展 `TypeAnnotator` 以支持参数和返回值的联合类型。使用 `Optional[X]` 作为 `Union[X, None]` 的简写。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises)来完成练习并查看解决方案。

### 练习 2：局部变量的类型

在 Python 中，不仅可以使用类型注解参数，还可以注解局部和全局变量——例如，在我们的 `my_sqrt()` 实现中的 `approx` 和 `guess`：

```py
def my_sqrt_with_local_types(x: Union[int, float]) -> float:
  """Computes the square root of x, using the Newton-Raphson method"""
    approx: Optional[float] = None
    guess: float = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx 
```

扩展 `TypeAnnotator` 以便它还注释局部变量的类型。在函数 AST 中搜索赋值语句，确定赋值的类型，并将其作为左侧的注释。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 3：冗长的不变性检查器

我们的不变性检查器实现没有清楚地告诉用户哪个前置/后置条件失败。

```py
@precondition(lambda s: len(s) > 0)
def remove_first_char(s):
    return s[1:] 
```

```py
with ExpectError():
    remove_first_char('') 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2212034949.py", line 2, in <module>
    remove_first_char('')
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/906718213.py", line 6, in wrapper
    assert precondition(*args, **kwargs), "Precondition violated"
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: Precondition violated (expected)

```

以下实现添加了一个可选的 `doc` 关键字参数，如果违反了不变性，则会打印出来：

```py
def verbose_condition(precondition=None, postcondition=None, doc='Unknown'):
    def decorator(func):
        @functools.wraps(func) # preserves name, docstring, etc
        def wrapper(*args, **kwargs):
            if precondition is not None:
                assert precondition(*args, **kwargs), "Precondition violated: " + doc

            retval = func(*args, **kwargs) # call original function or method
            if postcondition is not None:
                assert postcondition(retval, *args, **kwargs), "Postcondition violated: " + doc

            return retval
        return wrapper
    return decorator 
```

```py
def verbose_precondition(check, **kwargs):
    return verbose_condition(precondition=check, doc=kwargs.get('doc', 'Unknown')) 
```

```py
def verbose_postcondition(check, **kwargs):
    return verbose_condition(postcondition=check, doc=kwargs.get('doc', 'Unknown')) 
```

```py
@verbose_precondition(lambda s: len(s) > 0, doc="len(s) > 0")
def remove_first_char(s):
    return s[1:]

remove_first_char('abc') 
```

```py
'bc'

```

```py
with ExpectError():
    remove_first_char('') 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/2212034949.py", line 2, in <module>
    remove_first_char('')
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_35443/860932556.py", line 6, in wrapper
    assert precondition(*args, **kwargs), "Precondition violated: " + doc
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: Precondition violated: len(s) > 0 (expected)

```

扩展 `InvariantAnnotator` 以便它包括生成的预/后置条件中的条件。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 4：保存初始值

如果在函数执行过程中参数的值发生变化，这很容易使我们的实现困惑：值在函数开始时被跟踪，但只在返回时进行检查。扩展 `InvariantAnnotator` 及其使用的基础设施，以便

+   它在函数调用的开始和结束时都保存了参数值；

+   后置条件可以表达为参数的 *初始* 值以及参数的 *最终* 值；

+   挖掘出的后置条件也引用了这两个值。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 5：隐含关系

几个挖掘出的不变性实际上是由其他不变性隐含的：如果 `x > 0` 成立，那么这隐含了 `x >= 0` 和 `x != 0`。扩展 `InvariantAnnotator` 以便显式地编码属性之间的隐含关系，并且隐含的属性不再作为不变性列出。参见 [[Ernst 等人，2001](https://doi.org/10.1109/32.908957)] 获取想法。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 6：局部变量

后置条件也可能引用局部变量的值。考虑扩展 `InvariantAnnotator` 及其基础设施，以便记录执行结束时的局部变量值，并将它们作为不变性推理机制的一部分。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 7：探索不变性替代方案

在挖掘出一组不变性之后，让一个 concolic fuzzer 生成测试，这些测试系统地尝试使预/后置条件无效。你能推广到什么程度？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 8：语法生成的属性

要检查的属性集越大，可以发现的潜在不变量就越多。创建一个能够系统地生成大量属性的 *语法*。参见 [[Ernst 等人，2001](https://doi.org/10.1109/32.908957)] 中可能的模式。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 9：将不变量嵌入为断言

而不是将不变量作为前置条件和后置条件的注释，将它们作为 `assert` 语句插入到函数代码中，如下所示：

```py
def my_sqrt(x):
    'Computes the square root of x, using the Newton-Raphson method'
    assert isinstance(x, int), 'violated precondition'
    assert (x > 0), 'violated precondition'
    approx = None
    guess = (x / 2)
    while (approx != guess):
        approx = guess
        guess = ((approx + (x / approx)) / 2)
    return_value = approx
    assert (return_value < x), 'violated postcondition'
    assert isinstance(return_value, float), 'violated postcondition'
    return approx 
```

这样的表述可能使测试生成器和符号分析更容易访问和解释前置条件和后置条件。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/DynamicInvariants.ipynb#Exercises) 来完成练习并查看解决方案。

![Creative Commons 许可证](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容根据 [Creative Commons 知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，根据 [MIT 许可证](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 许可。 [最后更改：2024-11-09 17:07:29+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/DynamicInvariants.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[挖掘函数规范](https://www.fuzzingbook.org/html/DynamicInvariants.html)"。在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 的 "[模糊测试书](https://www.fuzzingbook.org/)" 中，[`www.fuzzingbook.org/html/DynamicInvariants.html`](https://www.fuzzingbook.org/html/DynamicInvariants.html)。检索日期：2024-11-09 17:07:29+01:00。

```py
@incollection{fuzzingbook2024:DynamicInvariants,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Mining Function Specifications},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/DynamicInvariants.html}},
    note = {Retrieved 2024-11-09 17:07:29+01:00},
    url = {https://www.fuzzingbook.org/html/DynamicInvariants.html},
    urldate = {2024-11-09 17:07:29+01:00}
}

```
