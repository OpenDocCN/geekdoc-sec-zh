# 计时器

> 原文：[`www.fuzzingbook.org/html/Timer.html`](http://www.fuzzingbook.org/html/Timer.html)

本笔记本中的代码有助于测量时间。

**先决条件**

+   本笔记本需要一些对 Python 高级概念的理解，特别是

    +   课程

    +   Python 的 `with` 语句

    +   测量时间

## 概述

要使用本章提供的代码，请编写

```py
>>> from fuzzingbook.Timer import <identifier> 
```

然后利用以下功能。

`Timer`类允许您测量经过的实际时间（以分数秒为单位）。它的典型用法是与`with`子句结合：

```py
>>> with Timer() as t:
>>>     some_long_running_function()
>>> t.elapsed_time()
0.020704666967503726 
```

## 测量时间

`Timer`类允许在代码执行期间测量经过的时间。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
import [time](https://docs.python.org/3/library/time.html) 
```

```py
def clock() -> float:
  """
 Return the number of fractional seconds elapsed since some point of reference.
 """
    return time.perf_counter() 
```

```py
from [types](https://docs.python.org/3/library/types.html) import TracebackType 
```

```py
class Timer:
    def __init__(self) -> None:
  """Constructor"""
        self.start_time = clock()
        self.end_time = None

    def __enter__(self) -> Any:
  """Begin of `with` block"""
        self.start_time = clock()
        self.end_time = None
        return self

    def __exit__(self, exc_type: Type, exc_value: BaseException,
                 tb: TracebackType) -> None:
  """End of `with` block"""
        self.end_time = clock()

    def elapsed_time(self) -> float:
  """Return elapsed time in seconds"""
        if self.end_time is None:
            # still running
            return clock() - self.start_time
        else:
            return self.end_time - self.start_time 
```

这里有一个例子：

```py
def some_long_running_function() -> None:
    i = 1000000
    while i > 0:
        i -= 1 
```

```py
print("Stopping total time:")
with Timer() as t:
    some_long_running_function()
print(t.elapsed_time()) 
```

```py
Stopping total time:
0.020959541958291084

```

```py
print("Stopping time in between:")
with Timer() as t:
    for i in range(10):
        some_long_running_function()
        print(t.elapsed_time()) 
```

```py
Stopping time in between:
0.01922783302143216
0.039810792019125074
0.06000966700958088
0.08149537502322346
0.10375195799861103
0.12619870802154765
0.14785908302292228
0.16952008299995214
0.19183695799438283
0.21232758299447596

```

好了，各位——享受吧！

## 经验教训

+   使用`Timer`类，测量经过的时间非常容易。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。 [最后更改：2023-11-11 18:25:47+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/Timer.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[Timer](https://www.fuzzingbook.org/html/Timer.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[The Fuzzing Book](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/Timer.html`](https://www.fuzzingbook.org/html/Timer.html). Retrieved 2023-11-11 18:25:47+01:00.

```py
@incollection{fuzzingbook2023:Timer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Timer},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/Timer.html}},
    note = {Retrieved 2023-11-11 18:25:47+01:00},
    url = {https://www.fuzzingbook.org/html/Timer.html},
    urldate = {2023-11-11 18:25:47+01:00}
}

```
