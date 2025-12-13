# 超时

> 原文：[`www.fuzzingbook.org/html/Timeout.html`](http://www.fuzzingbook.org/html/Timeout.html)

本笔记本中的代码有助于在给定时间后中断执行。

**先决条件**

+   这个笔记本需要对 Python 的高级概念有所了解，特别是

    +   类

    +   Python 的 `with` 语句

    +   Python 的 `signal` 函数

    +   测量时间

## 概述

要 使用本章节提供的代码，请编写

```py
>>> from fuzzingbook.Timeout import <identifier> 
```

然后利用以下功能。

`Timeout` 类在给定超时时间到期后抛出 `TimeoutError` 异常。它的典型用法是与 `with` 语句结合使用：

```py
>>> try:
>>>     with Timeout(0.2):
>>>         some_long_running_function()
>>>     print("complete!")
>>> except TimeoutError:
>>>     print("Timeout!")
Timeout! 
```

注意：在 Unix/Linux 系统上，`Timeout` 类使用 [SIGALRM 信号](https://docs.python.org/3.10/library/signal.html)（中断）来实现超时；这对跟踪代码的性能没有影响。在其他系统（特别是 Windows）上，`Timeout` 使用 [sys.settrace()](https://docs.python.org/3.10/library/sys.html?highlight=settrace#sys.settrace) 函数在每行代码后检查计时器，这会影响跟踪代码的性能。

## 测量时间

`Timeout` 类允许在给定时间间隔后中断某些代码执行。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
import [time](https://docs.python.org/3/library/time.html) 
```

```py
from [types](https://docs.python.org/3/library/types.html) import FrameType, TracebackType 
```

## 变体 1：Unix（使用信号，高效）

```py
import [signal](https://docs.python.org/3/library/signal.html) 
```

```py
class SignalTimeout:
  """Execute a code block raising a timeout."""

    def __init__(self, timeout: Union[int, float]) -> None:
  """
 Constructor. Interrupt execution after `timeout` seconds.
 """
        self.timeout = timeout
        self.old_handler: Any = signal.SIG_DFL
        self.old_timeout = 0.0

    def __enter__(self) -> Any:
  """Begin of `with` block"""
        # Register timeout() as handler for signal 'SIGALRM'"
        self.old_handler = signal.signal(signal.SIGALRM, self.timeout_handler)
        self.old_timeout, _ = signal.setitimer(signal.ITIMER_REAL, self.timeout)
        return self

    def __exit__(self, exc_type: Type, exc_value: BaseException,
                 tb: TracebackType) -> None:
  """End of `with` block"""
        self.cancel()
        return  # re-raise exception, if any

    def cancel(self) -> None:
  """Cancel timeout"""
        signal.signal(signal.SIGALRM, self.old_handler)
        signal.setitimer(signal.ITIMER_REAL, self.old_timeout)

    def timeout_handler(self, signum: int, frame: Optional[FrameType]) -> None:
  """Handle timeout (SIGALRM) signal"""
        raise TimeoutError() 
```

这里是一个例子：

```py
def some_long_running_function() -> None:
    i = 10000000
    while i > 0:
        i -= 1 
```

```py
try:
    with SignalTimeout(0.2):
        some_long_running_function()
        print("Complete!")
except TimeoutError:
    print("Timeout!") 
```

```py
Timeout!

```

## 变体 2：通用/Windows（使用跟踪，效率不高）

```py
import [sys](https://docs.python.org/3/library/sys.html) 
```

```py
class GenericTimeout:
  """Execute a code block raising a timeout."""

    def __init__(self, timeout: Union[int, float]) -> None:
  """
 Constructor. Interrupt execution after `timeout` seconds.
 """

        self.seconds_before_timeout = timeout
        self.original_trace_function: Optional[Callable] = None
        self.end_time: Optional[float] = None

    def check_time(self, frame: FrameType, event: str, arg: Any) -> Callable:
  """Tracing function"""
        if self.original_trace_function is not None:
            self.original_trace_function(frame, event, arg)

        current_time = time.time()
        if self.end_time and current_time >= self.end_time:
            raise TimeoutError

        return self.check_time

    def __enter__(self) -> Any:
  """Begin of `with` block"""
        start_time = time.time()
        self.end_time = start_time + self.seconds_before_timeout

        self.original_trace_function = sys.gettrace()
        sys.settrace(self.check_time)
        return self

    def __exit__(self, exc_type: type, 
                 exc_value: BaseException, tb: TracebackType) -> Optional[bool]:
  """End of `with` block"""
        self.cancel()
        return None  # re-raise exception, if any

    def cancel(self) -> None:
  """Cancel timeout"""
        sys.settrace(self.original_trace_function) 
```

再次，我们的例子：

```py
try:
    with GenericTimeout(0.2):
        some_long_running_function()
        print("Complete!")
except TimeoutError:
    print("Timeout!") 
```

```py
Timeout!

```

## 选择正确的变体

```py
Timeout: Type[SignalTimeout] = SignalTimeout if hasattr(signal, 'SIGALRM') else GenericTimeout 
```

## 练习

创建一个在 Windows 上高效运行的 `Timeout` 变体。请注意，如何在编程论坛上这是一个长期争论的问题。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后更改：2023-11-11 18:25:46+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/Timeout.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[Timeout](https://www.fuzzingbook.org/html/Timeout.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[The Fuzzing Book](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/Timeout.html`](https://www.fuzzingbook.org/html/Timeout.html). Retrieved 2023-11-11 18:25:46+01:00.

```py
@incollection{fuzzingbook2023:Timeout,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Timeout},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/Timeout.html}},
    note = {Retrieved 2023-11-11 18:25:46+01:00},
    url = {https://www.fuzzingbook.org/html/Timeout.html},
    urldate = {2023-11-11 18:25:46+01:00}
}

```
