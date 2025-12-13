# 错误处理

> 原文：[`www.fuzzingbook.org/html/ExpectError.html`](http://www.fuzzingbook.org/html/ExpectError.html)

这个笔记本中的代码有助于处理错误。通常，笔记本中的错误会导致代码执行停止；而笔记本中的无限循环会导致笔记本无限运行。这个笔记本提供了两个类来帮助解决这些问题。

**先决条件**

+   这个笔记本需要对 Python 的高级概念有所了解，特别是

    +   类

    +   Python 的 `with` 语句

    +   跟踪

    +   测量时间

    +   异常

## 概述

要使用本章提供的代码（Importing.html），请编写

```py
>>> from fuzzingbook.ExpectError import <identifier> 
```

然后利用以下功能。

`ExpectError` 类允许你捕获并报告异常，同时继续执行。这在笔记本中很有用，因为它们通常会一遇到异常就中断执行。它的典型用法是与 `with` 语句结合：

```py
>>> with ExpectError():
>>>     x = 1 / 0
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/2664980466.py", line 2, in <module>
    x = 1 / 0
        ~~^~~
ZeroDivisionError: division by zero (expected) 
```

`ExpectTimeout` 类允许你在指定的时间后中断执行。这对于中断可能无限运行的代码很有用。

```py
>>> with ExpectTimeout(5):
>>>     long_running_test()
Start
0 seconds have passed
1 seconds have passed
2 seconds have passed
3 seconds have passed

Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/1223755941.py", line 2, in <module>
    long_running_test()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/3930412460.py", line 4, in long_running_test
    time.sleep(1)
  File "Timeout.ipynb", line 43, in timeout_handler
    raise TimeoutError()
TimeoutError (expected) 
```

异常及其相关的堆栈跟踪会作为错误消息打印。如果你不希望这样，可以使用以下关键字选项：

+   `print_traceback`（默认为 True）可以设置为 `False` 以避免打印堆栈跟踪

+   `mute`（默认为 False）可以设置为 `True` 以完全避免任何输出。

## 捕获错误

`ExpectError` 类允许表达某些代码会产生异常。典型的用法如下：

```py
from ExpectError import ExpectError

with ExpectError():
    function_that_is_supposed_to_fail() 
```

如果发生异常，它会在标准错误上打印；然而，执行继续。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
import [traceback](https://docs.python.org/3/library/traceback.html)
import [sys](https://docs.python.org/3/library/sys.html) 
```

```py
from [types](https://docs.python.org/3/library/types.html) import FrameType, TracebackType 
```

```py
class ExpectError:
  """Execute a code block expecting (and catching) an error."""

    def __init__(self, exc_type: Optional[type] = None, 
                 print_traceback: bool = True, mute: bool = False):
  """
 Constructor. Expect an exception of type `exc_type` (`None`: any exception).
 If `print_traceback` is set (default), print a traceback to stderr.
 If `mute` is set (default: False), do not print anything.
 """
        self.print_traceback = print_traceback
        self.mute = mute
        self.expected_exc_type = exc_type

    def __enter__(self) -> Any:
  """Begin of `with` block"""
        return self

    def __exit__(self, exc_type: type, 
                 exc_value: BaseException, tb: TracebackType) -> Optional[bool]:
  """End of `with` block"""
        if exc_type is None:
            # No exception
            return

        if (self.expected_exc_type is not None
            and exc_type != self.expected_exc_type):
            raise  # Unexpected exception

        # An exception occurred
        if self.print_traceback:
            lines = ''.join(
                traceback.format_exception(
                    exc_type,
                    exc_value,
                    tb)).strip()
        else:
            lines = traceback.format_exception_only(
                exc_type, exc_value)[-1].strip()

        if not self.mute:
            print(lines, "(expected)", file=sys.stderr)
        return True  # Ignore it 
```

这里有一个例子：

```py
def fail_test() -> None:
    # Trigger an exception
    x = 1 / 0 
```

```py
with ExpectError():
    fail_test() 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/1235320646.py", line 2, in <module>
    fail_test()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/278441162.py", line 3, in fail_test
    x = 1 / 0
        ~~^~~
ZeroDivisionError: division by zero (expected)

```

```py
with ExpectError(print_traceback=False):
    fail_test() 
```

```py
ZeroDivisionError: division by zero (expected)

```

我们可以指定期望的异常类型。这样，如果发生其他情况，我们会收到通知。

```py
with ExpectError(ZeroDivisionError):
    fail_test() 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/1259188418.py", line 2, in <module>
    fail_test()
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/278441162.py", line 3, in fail_test
    x = 1 / 0
        ~~^~~
ZeroDivisionError: division by zero (expected)

```

```py
with ExpectError():
    with ExpectError(ZeroDivisionError):
        some_nonexisting_function() 
```

```py
Traceback (most recent call last):
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/2242794116.py", line 2, in <module>
    with ExpectError(ZeroDivisionError):
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71827/2242794116.py", line 3, in <module>
    some_nonexisting_function()  # type: ignore
    ^^^^^^^^^^^^^^^^^^^^^^^^^
NameError: name 'some_nonexisting_function' is not defined (expected)

```

## 捕获超时

`ExpectTimeout(seconds)` 类允许表达某些代码可能运行很长时间或无限时间；因此，在 `seconds` 秒后中断执行。典型的用法如下：

```py
from ExpectError import ExpectTimeout

with ExpectTimeout(2) as t:
    function_that_is_supposed_to_hang() 
```

如果发生异常，它会在标准错误上打印（与 `ExpectError` 类似）；然而，执行继续。

如果需要在 `with` 块内取消超时，`t.cancel()` 将会起作用。

实现使用 `sys.settrace()`，因为这似乎是实现超时最可移植的方法。然而，它并不高效。此外，它只适用于单个 Python 代码行，并且不会中断长时间运行的系统函数。

```py
import [sys](https://docs.python.org/3/library/sys.html)
import [time](https://docs.python.org/3/library/time.html) 
```

```py
from Timeout import Timeout 
```

```py
class ExpectTimeout(Timeout):
  """Execute a code block expecting (and catching) a timeout."""

    def __init__(self, timeout: Union[int, float],
                 print_traceback: bool = True, mute: bool = False):
  """
 Constructor. Interrupt execution after `seconds` seconds.
 If `print_traceback` is set (default), print a traceback to stderr.
 If `mute` is set (default: False), do not print anything.
 """
        super().__init__(timeout)

        self.print_traceback = print_traceback
        self.mute = mute

    def __exit__(self, exc_type: type,
                 exc_value: BaseException, tb: TracebackType) -> Optional[bool]:
  """End of `with` block"""

        super().__exit__(exc_type, exc_value, tb)

        if exc_type is None:
            return

        # An exception occurred
        if self.print_traceback:
            lines = ''.join(
                traceback.format_exception(
                    exc_type,
                    exc_value,
                    tb)).strip()
        else:
            lines = traceback.format_exception_only(
                exc_type, exc_value)[-1].strip()

        if not self.mute:
            print(lines, "(expected)", file=sys.stderr)

        return True  # Ignore exception 
```

这里有一个例子：

```py
def long_running_test() -> None:
    print("Start")
    for i in range(10):
        time.sleep(1)
        print(i, "seconds have passed")
    print("End") 
```

```py
with ExpectTimeout(5, print_traceback=False):
    long_running_test() 
```

```py
Start
0 seconds have passed
1 seconds have passed
2 seconds have passed
3 seconds have passed

```

```py
TimeoutError (expected)

```

注意，可以嵌套多个超时。

```py
with ExpectTimeout(5, print_traceback=False):
    with ExpectTimeout(3, print_traceback=False):
        long_running_test()
    long_running_test() 
```

```py
Start
0 seconds have passed
1 seconds have passed

```

```py
TimeoutError (expected)

```

```py
Start
0 seconds have passed
1 seconds have passed
2 seconds have passed
3 seconds have passed

```

```py
TimeoutError (expected)

```

就这样，朋友们——享受吧！

## 经验教训

+   使用 `ExpectError` 类，可以非常容易地处理错误，而不会中断笔记本执行。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内 容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)的许可。 [最后修改：2023-11-11 18:25:46+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/ExpectError.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用这篇作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[错误处理](https://www.fuzzingbook.org/html/ExpectError.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler, "[模糊测试书](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/ExpectError.html`](https://www.fuzzingbook.org/html/ExpectError.html). Retrieved 2023-11-11 18:25:46+01:00.

```py
@incollection{fuzzingbook2023:ExpectError,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Error Handling},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/ExpectError.html}},
    note = {Retrieved 2023-11-11 18:25:46+01:00},
    url = {https://www.fuzzingbook.org/html/ExpectError.html},
    urldate = {2023-11-11 18:25:46+01:00}
}

```
