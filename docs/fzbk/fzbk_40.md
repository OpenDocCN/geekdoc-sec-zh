# 学术原型设计

> [`www.fuzzingbook.org/html/AcademicPrototyping.html`](http://www.fuzzingbook.org/html/AcademicPrototyping.html)

*这是安德烈亚斯·策勒尔（Andreas Zeller）在 ESEC/FSE 2022 会议上的教程 "Academic Prototyping" 的手稿。*

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo("Z7Z0cdwPS0U") 
```

## 关于本教程

在我们的 Fuzzing Book 中，我们使用 Python 来实现自动化测试技术，并将其作为大多数测试主题的语言。为什么是 Python？简短的答案是

> Python 让我们惊人地 **高效**。本书中的大多数技术只需 **2-3 天** 就可以实施。这比 "经典" 语言如 C 或 Java 快 **10-20 倍**。

生产力的提升达到 10-20 倍是巨大的，几乎是荒谬的。为什么会这样，这对研究和教学有什么影响？

### 我将讨论什么

在这个教程中，我将通过从头开始原型设计一个 *符号测试生成器* 来演示这一点。这通常被认为是一个非常困难的任务，需要数月时间来构建。然而，本章中开发代码的时间不到两小时——解释它的时间不到 20 分钟。

我们将探讨这种生产力的原因，将 *Python* 和 *Jupyter notebooks* 作为主要驱动力。我们还将讨论这对学术软件开发的影响。

### 你将学到什么

+   使用 *Python* 和其库作为原型设计的语言

+   使用 *Jupyter Notebooks* 作为编码和运行评估的工具

+   理解生产力和可重复性的提升来自哪里

## 分析编译语言的困难

在我的职业生涯中，我构建（并允许构建）了几个在编译语言（如 C 或 Java）上操作的工具——特别是

+   自动化调试工具，这些工具将提取和分析程序执行中的动态信息，

+   自动化测试工具，这些工具将使用（生成的）输入对单个应用程序或函数进行测试。

从这次经历中学到的东西可以总结如下：

1.  配置编译代码是 **困难的**。你需要连接到现有的基础设施，如 Java 或 C 编译器，找出要添加哪些代码，找出如何适应相应的编译器。

1.  从编译代码中捕获信息也很 **困难**。同样，你需要对代码进行 **配置**（见上文），但还需要确保信息被存储（高效地？），稍后可以检索和解码。

所有这些都充满了错误。成功的测试生成器之所以假设基础设施绝对最小化（例如，使用现有工具获取覆盖率，而无需更多），是有充分理由的。

如果你想要进行静态分析甚至符号分析，情况会更糟。自己解析 C 和 Java 代码是困难的；对于大规模代码，几乎是不可能的。由于现有基础设施（如 Soot）的力量，在某种中间表示上工作是可能的。但如果你想要扩展这些，你将面临数周甚至数月的工作。

符号分析特别困难，因为你通常希望在一个至少与源代码一样高的抽象级别上操作，这要求你在符号级别、源代码级别和实际机器代码之间建立联系。所有这些都是可能的，但很困难。

现在，如果你的重点是真正分析大规模代码，例如，因为

+   领域非常重要，问题具有挑战性

+   你在工业界工作，并且从事安全关键系统

+   你想在编译语言的大量样本上评估某种方法

那么最终将你的方法应用于大规模代码可能是有意义的。但在那之前，你可能想检查你的方法是否真的有效。

这就是*原型设计*发挥作用的地方。原型设计意味着开发一个快速解决方案来探索某种方法的*可行性*：

+   从实践中收集反馈

+   快速*进化*一种方法

+   以证明某事可能有效

换句话说，这是我们通常需要在学术领域展示的概念验证。

## Python 很简单

Python 是一种高级语言，它允许人们专注于实际的*算法*，而不是单个位和字节如何在内存中传递。对于这本书来说，这一点很重要：我们希望关注个别技术是如何工作的，而不是它们的优化。关注算法允许你玩弄和调整它们，并快速开发自己的。一旦你找到了做事的方法，你仍然可以将你的方法移植到其他语言或专门的设置中。

例如，考虑一下（不）著名的*三角形*程序，它将长度为$a$、$b$、$c$的三角形分类为三个类别之一。它看起来像伪代码；然而，我们可以轻松地执行它。

```py
def triangle(a, b, c):
    if a == b:
        if b == c:
            return 'equilateral'
        else:
            return 'isosceles #1'
    else:
        if b == c:
            return 'isosceles #2'
        else:
            if a == c:
                return 'isosceles #3'
            else:
                return 'scalene' 
```

这里是执行`triangle()`函数的一个示例：

```py
triangle(2, 3, 4) 
```

```py
'scalene'

```

在本章剩余部分，我们将使用`triangle()`函数作为测试程序的持续示例。当然，`triangle()`函数的复杂性远远低于大型系统，本章所展示的内容也不适用于，比如说，由数千个相互交织的微服务组成的生态系统。然而，其目的是展示某些技术如果拥有合适的语言和环境，可以多么容易实现。

## 构建一个最小测试器

如果你想用随机值测试`triangle()`，这相当简单。只需携带一个 Python 随机数生成器，并将它们投入`triangle()`中。

```py
from [random](https://docs.python.org/3/library/random.html) import randrange 
```

```py
for i in range(10):
    a = randrange(1, 10)
    b = randrange(1, 10)
    c = randrange(1, 10)

    t = triangle(a, b, c)
    print(f"triangle({a}, {b}, {c}) = {repr(t)}") 
```

```py
triangle(4, 1, 7) = 'scalene'
triangle(4, 5, 3) = 'scalene'
triangle(7, 2, 1) = 'scalene'
triangle(7, 6, 9) = 'scalene'
triangle(5, 5, 8) = 'isosceles #1'
triangle(1, 7, 1) = 'isosceles #3'
triangle(6, 4, 8) = 'scalene'
triangle(8, 9, 5) = 'scalene'
triangle(4, 8, 3) = 'scalene'
triangle(6, 1, 1) = 'isosceles #2'

```

到目前为止，一切顺利——但这在几乎任何编程语言中都可以做到。是什么让 Python 变得特别？

## Python 中的动态分析：如此简单以至于令人痛苦

动态分析是跟踪程序执行期间发生情况的能力。Python 的`settrace()`机制允许你在程序执行时跟踪所有代码行、所有变量、所有值——所有这些都在几行代码中完成。我们来自覆盖率章节的`Coverage`类展示了如何用五行代码捕获所有执行行的跟踪；这样的跟踪可以轻松地转换为执行行或分支的集合。再加上两行，你可以轻松地跟踪所有函数、参数、变量值——例如，查看我们的动态不变性章节。你甚至可以访问单个函数的源代码（并且可以打印出来！）所有这些只需要 10 分钟，也许 20 分钟就可以实现。

这里有一段 Python 代码可以完成所有这些。我们跟踪执行的行，并为每一行打印其源代码和所有局部变量的当前值：

```py
import [sys](https://docs.python.org/3/library/sys.html)
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
def traceit(frame, event, arg):
    function_code = frame.f_code
    function_name = function_code.co_name
    lineno = frame.f_lineno
    vars = frame.f_locals

    source_lines, starting_line_no = inspect.getsourcelines(frame.f_code)
    loc = f"{function_name}:{lineno}  {source_lines[lineno  -  starting_line_no].rstrip()}"
    vars = ", ".join(f"{name} = {vars[name]}" for name in vars)

    print(f"{loc:50} ({vars})")

    return traceit 
```

函数`sys.settrace()`将`traceit()`注册为跟踪函数；然后它会跟踪`triangle()`的给定调用：

```py
def triangle_traced():
    sys.settrace(traceit)
    triangle(2, 2, 1)
    sys.settrace(None) 
```

```py
triangle_traced() 
```

```py
triangle:1 def triangle(a, b, c):                  (a = 2, b = 2, c = 1)
triangle:2     if a == b:                          (a = 2, b = 2, c = 1)
triangle:3         if b == c:                      (a = 2, b = 2, c = 1)
triangle:6             return 'isosceles #1'       (a = 2, b = 2, c = 1)
triangle:6             return 'isosceles #1'       (a = 2, b = 2, c = 1)

```

相比之下，尝试为 C 语言构建这样的动态分析。你可以选择*仪器化*代码以跟踪所有执行的行并记录变量值，将结果信息存储在某个数据库中。这可能需要*几周*，甚至*几个月*才能实现。你也可以通过调试器（逐步打印-打印-逐步打印-打印）运行你的代码；但同样，编程交互可能需要几天。一旦你得到初步结果，你可能会意识到你需要其他或更好的东西，所以你回到起点。这不是一件有趣的事情。

与上述动态分析一起，你可以使模糊测试更加智能。例如，基于搜索的测试会进化一个输入种群，以实现特定目标，如覆盖率。有了良好的动态分析，你可以快速实现针对任意目标的基于搜索的策略。

## Python 中的静态分析：仍然简单

静态分析指的是在不实际执行程序代码的情况下分析程序代码的能力。对 Python 代码进行静态分析以推断任何属性可能是一个噩梦，因为这种语言非常动态。（更多内容见下文。）

如果你的静态分析不需要是*可靠的*，例如，因为你只使用它来*支持*和*指导*其他技术，如测试，那么 Python 中的静态分析可以非常简单。`ast`模块允许你将任何 Python 函数转换为抽象语法树（AST），然后你可以随意遍历它。这是我们的`triangle()`函数的 AST：

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import rich_output 
```

```py
import [ast](https://docs.python.org/3/library/ast.html) 
```

```py
if rich_output():
    # Normally, this will do
    from [showast](https://pypi.org/project/showast/) import show_ast
else:
    def show_ast(tree):
        ast.dump(tree, indent=4) 
```

```py
triangle_source = inspect.getsource(triangle)
triangle_ast = ast.parse(triangle_source)
show_ast(triangle_ast) 
```

<svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1867pt" height="476pt" viewBox="0.00 0.00 1867.38 476.00"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 472)"><g id="node1" class="node"><title>0</title> <text text-anchor="start" x="115.88" y="-445.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">FunctionDef</text></g> <g id="node2" class="node"><title>1</title> <text text-anchor="middle" x="49.25" y="-372.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"triangle"</text></g> <g id="edge1" class="edge"><title>0--1</title></g> <g id="node3" class="node"><title>2</title> <text text-anchor="start" x="124.12" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arguments</text></g> <g id="edge2" class="edge"><title>0--2</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="start" x="495" y="-373.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">If</text></g> <g id="edge9" class="edge"><title>0--9</title></g> <g id="node4" class="node"><title>3</title> <text text-anchor="start" x="40.88" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arg</text></g> <g id="edge3" class="edge"><title>2--3</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="start" x="112.88" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arg</text></g> <g id="edge5" class="edge"><title>2--5</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="start" x="184.88" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">arg</text></g> <g id="edge7" class="edge"><title>2--7</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="53.25" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"a"</text></g> <g id="edge4" class="edge"><title>3--4</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="125.25" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"b"</text></g> <g id="edge6" class="edge"><title>5--6</title></g> <g id="node9" class="node"><title>8</title> <text text-anchor="middle" x="197.25" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"c"</text></g> <g id="edge8" class="edge"><title>7--8</title></g> <g id="node11" class="node"><title>10</title> <text text-anchor="start" x="348.38" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge10" class="edge"><title>9--10</title></g> <g id="node19" class="node"><title>18</title> <text text-anchor="start" x="656" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">If</text></g> <g id="edge18" class="edge"><title>9--18</title></g> <g id="node34" class="node"><title>33</title> <text text-anchor="start" x="1220" y="-301.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">If</text></g> <g id="edge33" class="edge"><title>9--33</title></g> <g id="node12" class="node"><title>11</title> <text text-anchor="start" x="252.75" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge11" class="edge"><title>10--11</title></g> <g id="node15" class="node"><title>14</title> <text text-anchor="middle" x="341.25" y="-228.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Eq</text></g> <g id="edge14" class="edge"><title>10--14</title></g> <g id="node16" class="node"><title>15</title> <text text-anchor="start" x="396.75" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge15" class="edge"><title>10--15</title></g> <g id="node13" class="node"><title>12</title> <text text-anchor="middle" x="197.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"a"</text></g> <g id="edge12" class="edge"><title>11--12</title></g> <g id="node14" class="node"><title>13</title> <text text-anchor="middle" x="269.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge13" class="edge"><title>11--13</title></g> <g id="node17" class="node"><title>16</title> <text text-anchor="middle" x="341.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"b"</text></g> <g id="edge16" class="edge"><title>15--16</title></g> <g id="node18" class="node"><title>17</title> <text text-anchor="middle" x="413.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge17" class="edge"><title>15--17</title></g> <g id="node20" class="node"><title>19</title> <text text-anchor="start" x="564.38" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge19" class="edge"><title>18--19</title></g> <g id="node28" class="node"><title>27</title> <text text-anchor="start" x="674.5" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge27" class="edge"><title>18--27</title></g> <g id="node31" class="node"><title>30</title> <text text-anchor="start" x="799.5" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge30" class="edge"><title>18--30</title></g> <g id="node21" class="node"><title>20</title> <text text-anchor="start" x="468.75" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge20" class="edge"><title>19--20</title></g> <g id="node24" class="node"><title>23</title> <text text-anchor="middle" x="557.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Eq</text></g> <g id="edge23" class="edge"><title>19--23</title></g> <g id="node25" class="node"><title>24</title> <text text-anchor="start" x="612.75" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge24" class="edge"><title>19--24</title></g> <g id="node22" class="node"><title>21</title> <text text-anchor="middle" x="413.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"b"</text></g> <g id="edge21" class="edge"><title>20--21</title></g> <g id="node23" class="node"><title>22</title> <text text-anchor="middle" x="485.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge22" class="edge"><title>20--22</title></g> <g id="node26" class="node"><title>25</title> <text text-anchor="middle" x="557.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"c"</text></g> <g id="edge25" class="edge"><title>24--25</title></g> <g id="node27" class="node"><title>26</title> <text text-anchor="middle" x="629.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge26" class="edge"><title>24--26</title></g> <g id="node29" class="node"><title>28</title> <text text-anchor="start" x="693.25" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge28" class="edge"><title>27--28</title></g> <g id="node30" class="node"><title>29</title> <text text-anchor="middle" x="736.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"equilateral"</text></g> <g id="edge29" class="edge"><title>28--29</title></g> <g id="node32" class="node"><title>31</title> <text text-anchor="start" x="821.25" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge31" class="edge"><title>30--31</title></g> <g id="node33" class="node"><title>32</title> <text text-anchor="middle" x="881.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"isosceles #1"</text></g> <g id="edge32" class="edge"><title>31--32</title></g> <g id="node35" class="node"><title>34</title> <text text-anchor="start" x="1131.38" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge34" class="edge"><title>33--34</title></g> <g id="node43" class="node"><title>42</title> <text text-anchor="start" x="1235.5" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge42" class="edge"><title>33--42</title></g> <g id="node46" class="node"><title>45</title> <text text-anchor="start" x="1577" y="-229.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">If</text></g> <g id="edge45" class="edge"><title>33--45</title></g> <g id="node36" class="node"><title>35</title> <text text-anchor="start" x="1047.75" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge35" class="edge"><title>34--35</title></g> <g id="node39" class="node"><title>38</title> <text text-anchor="middle" x="1136.25" y="-156.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Eq</text></g> <g id="edge38" class="edge"><title>34--38</title></g> <g id="node40" class="node"><title>39</title> <text text-anchor="start" x="1191.75" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge39" class="edge"><title>34--39</title></g> <g id="node37" class="node"><title>36</title> <text text-anchor="middle" x="992.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"b"</text></g> <g id="edge36" class="edge"><title>35--36</title></g> <g id="node38" class="node"><title>37</title> <text text-anchor="middle" x="1064.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge37" class="edge"><title>35--37</title></g> <g id="node41" class="node"><title>40</title> <text text-anchor="middle" x="1136.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"c"</text></g> <g id="edge40" class="edge"><title>39--40</title></g> <g id="node42" class="node"><title>41</title> <text text-anchor="middle" x="1208.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge41" class="edge"><title>39--41</title></g> <g id="node44" class="node"><title>43</title> <text text-anchor="start" x="1274.25" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge43" class="edge"><title>42--43</title></g> <g id="node45" class="node"><title>44</title> <text text-anchor="middle" x="1319.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"isosceles #2"</text></g> <g id="edge44" class="edge"><title>43--44</title></g> <g id="node47" class="node"><title>46</title> <text text-anchor="start" x="1489.38" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Compare</text></g> <g id="edge46" class="edge"><title>45--46</title></g> <g id="node55" class="node"><title>54</title> <text text-anchor="start" x="1628.5" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge54" class="edge"><title>45--54</title></g> <g id="node58" class="node"><title>57</title> <text text-anchor="start" x="1758.5" y="-157.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Return</text></g> <g id="edge57" class="edge"><title>45--57</title></g> <g id="node48" class="node"><title>47</title> <text text-anchor="start" x="1413.75" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge47" class="edge"><title>46--47</title></g> <g id="node51" class="node"><title>50</title> <text text-anchor="middle" x="1502.25" y="-84.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Eq</text></g> <g id="edge50" class="edge"><title>46--50</title></g> <g id="node52" class="node"><title>51</title> <text text-anchor="start" x="1557.75" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Name</text></g> <g id="edge51" class="edge"><title>46--51</title></g> <g id="node49" class="node"><title>48</title> <text text-anchor="middle" x="1358.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"a"</text></g> <g id="edge48" class="edge"><title>47--48</title></g> <g id="node50" class="node"><title>49</title> <text text-anchor="middle" x="1430.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge49" class="edge"><title>47--49</title></g> <g id="node53" class="node"><title>52</title> <text text-anchor="middle" x="1502.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"c"</text></g> <g id="edge52" class="edge"><title>51--52</title></g> <g id="node54" class="node"><title>53</title> <text text-anchor="middle" x="1574.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">Load</text></g> <g id="edge53" class="edge"><title>51--53</title></g> <g id="node56" class="node"><title>55</title> <text text-anchor="start" x="1640.25" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge55" class="edge"><title>54--55</title></g> <g id="node57" class="node"><title>56</title> <text text-anchor="middle" x="1685.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"isosceles #3"</text></g> <g id="edge56" class="edge"><title>55--56</title></g> <g id="node59" class="node"><title>58</title> <text text-anchor="start" x="1767.25" y="-85.95" font-family="Courier,monospace" font-weight="bold" font-size="14.00" fill="#004080">Constant</text></g> <g id="edge58" class="edge"><title>57--58</title></g> <g id="node60" class="node"><title>59</title> <text text-anchor="middle" x="1814.25" y="-12.95" font-family="Courier,monospace" font-size="14.00" fill="#008040">"scalene"</text></g> <g id="edge59" class="edge"><title>58--59</title></g></g></svg>

现在假设有人想使用静态分析来识别所有`triangle`分支及其条件。你会遍历 AST，寻找`If`节点，并取它们的第一个子节点（条件）。这也很简单：

```py
def collect_conditions(tree):
    conditions = []

    def traverse(node):
        if isinstance(node, ast.If):
            cond = ast.unparse(node.test).strip()
            conditions.append(cond)

        for child in ast.iter_child_nodes(node):
            traverse(child)

    traverse(tree)
    return conditions 
```

这里是`triangle()`代码中出现的四个`if`条件：

```py
collect_conditions(triangle_ast) 
```

```py
['a == b', 'b == c', 'b == c', 'a == c']

```

我们不仅可以从程序中提取单个元素，还可以随意更改它们，并将树转换回源代码。程序转换（例如，用于仪器或突变分析）变得轻而易举。上述代码仅用五分钟就写好了。再次尝试用 Java 或 C 来完成。

## Python 中的符号推理：有相应的包

让我们回到测试。我们已经展示了如何从代码中提取条件。要到达`triangle()`函数的特定位置，需要找到导致该分支的*路径条件*的解决方案。要到达`triangle()`的最后一行（`'scalene'`分支），我们必须找到满足$$a \ne b \land b \ne c \land a \ne c$$的解决方案

我们可以使用一个*约束*求解器来完成这项工作，例如微软的[*Z3*求解器](https://github.com/Z3Prover/z3)：

```py
import [z3](https://github.com/Z3Prover/z3#readme) 
```

让我们使用 Z3 来找到`'scalene'`分支条件的解决方案：

```py
a = z3.Int('a')
b = z3.Int('b')
c = z3.Int('c') 
```

```py
s = z3.Solver()
s.add(z3.And(a > 0, b > 0, c > 0))  # Triangle edges are positive
s.add(z3.And(a != b, b != c, a != c))  # Our condition
s.check() 
```

**sat**

Z3 已经向我们表明存在一个解决方案（“sat”=“可满足”）。让我们找到一个：

```py
m = s.model()
m 
```

[a = 1, c = 3, b = 2]

我们可以直接使用这个解决方案来测试`triangle()`函数，并发现它确实覆盖了`'scalene'`分支。`as_long()`方法将 Z3 结果转换为数值。

```py
triangle(m[a].as_long(), m[b].as_long(), m[c].as_long()) 
```

```py
'scalene'

```

## 符号测试生成器

通过我们所看到的，我们现在可以构建一个*符号测试生成器*——一个试图系统地创建覆盖所有路径的测试输入的工具。让我们通过探索树中的所有路径来找到我们需要解决的所有条件。我们立即将这些路径转换为 Z3 格式：

```py
def collect_path_conditions(tree):
    paths = []

    def traverse_if_children(children, context, cond):
        old_paths = len(paths)
        for child in children:
            traverse(child, context + [cond])
        if len(paths) == old_paths:
            paths.append(context + [cond])

    def traverse(node, context):
        if isinstance(node, ast.If):
            cond = ast.unparse(node.test).strip()
            not_cond = "z3.Not(" + cond + ")"

            traverse_if_children(node.body, context, cond)
            traverse_if_children(node.orelse, context, not_cond)

        else:
            for child in ast.iter_child_nodes(node):
                traverse(child, context)

    traverse(tree, [])

    return ["z3.And(" + ", ".join(path) + ")" for path in paths] 
```

```py
path_conditions = collect_path_conditions(triangle_ast)
path_conditions 
```

```py
['z3.And(a == b, b == c)',
 'z3.And(a == b, z3.Not(b == c))',
 'z3.And(z3.Not(a == b), b == c)',
 'z3.And(z3.Not(a == b), z3.Not(b == c), a == c)',
 'z3.And(z3.Not(a == b), z3.Not(b == c), z3.Not(a == c))']

```

现在我们需要做的就是将这些约束输入到 Z3 中。我们看到我们很容易覆盖所有分支：

```py
for path_condition in path_conditions:
    s = z3.Solver()
    s.add(a > 0, b > 0, c > 0)
    eval(f"s.check({path_condition})")
    m = s.model()
    print(m, triangle(m[a].as_long(), m[b].as_long(), m[c].as_long())) 
```

```py
[a = 1, c = 1, b = 1] equilateral
[c = 2, a = 1, b = 1] isosceles #1
[c = 2, a = 1, b = 2] isosceles #2
[c = 1, a = 1, b = 2] isosceles #3
[c = 3, a = 1, b = 2] scalene

```

成功！我们已经覆盖了三角形程序的 所有分支！

现在，上述内容仍然非常有限——并且针对`triangle()`代码的能力进行了定制。完整的实现实际上

+   将整个 Python 条件翻译成 Z3 语法（如果可能），

+   处理更多的控制流结构，如返回、断言、异常

+   以及更多（循环、调用，等等）

其中一些可能不受 Z3 理论的支持。

为了让约束求解器更容易找到解决方案，你也可以提供从早期执行中观察到的*具体值*，这些值已知能够到达程序中的特定路径。这些具体值将从上述跟踪机制中收集，然后：你将拥有一个相当强大且可扩展的符号化（具体-符号）测试生成器。

现在，上述可能需要你一两天的时间，并且随着你将测试生成器扩展到`triangle()`之外，你将添加越来越多的功能。好的部分是，你将发明出的每一个功能实际上可能是一项研究贡献——一些以前没有人想过的事情。无论你有什么想法：你都可以快速实现它，并在原型中尝试。而且，这会比传统语言快得多。

## 可复现的实验

仅使用像 Python 这样的语言可能会让你成为一个（且富有创意的）代码研究员。然而，作为一个好的科学家，你还需要在运行实验时保持*纪律*。*你到底做了什么来达到论文中所述的结果*？这就是我的第二个不那么秘密的武器发挥作用的地方：笔记本。

Jupyter 笔记本可以存储*所有*你的代码（尽管许多人更喜欢使用 IDE 来做这件事）；但至少，它可以*记录*你在实验中进行的所有步骤，包括解释设置和你的理由的*富文本*，以及（当然！）丰富的可视化图表，这些图表可以直观地展示你的结果。这正是 Jupyter 笔记本擅长的，如果你做得正确，你可以从原始数据记录到图表和数字，就像你在论文中找到的那样，记录和复制你所有的实验。

这里有一些我们在我（基于 Jupyter）的书中使用图表的例子。首先是一些“标准”的数据可视化：

+   [图表](https://www.fuzzingbook.org/html/GreyboxFuzzer.html#Greybox-Mutation-based-Fuzzing)

+   [树状图](https://www.fuzzingbook.org/html/ChangeCounter.html#Putting-it-all-Together)

然而，我们也创建了我们的*自己的可视化*——例如

+   [类图](https://www.fuzzingbook.org/html/GreyboxFuzzer.html#Greybox-Fuzzing)

+   [状态机](https://www.debuggingbook.org/html/Intro_Debugging.html#Visualizing-Code)

+   [程序频谱](https://www.fuzzingbook.org/html/StatisticalDebugger.html#Continuous-Spectrum)

+   [依赖关系](https://www.debuggingbook.org/html/Slicer.html#Dependency-Graphs)

+   [铁路图](https://www.fuzzingbook.org/html/Grammars.html#Visualizing-Grammars-as-Railroad-Diagrams)

所有这些都会随着程序代码及其结果自动更新——因此始终是最新的。吃掉它，LaTeX——还有 TikZ！

但不仅如此——你还可以*检查*结果是否与你已经在论文中写下的内容相符。如果你在论文中写道你发现$p \le 0.05$，那么一个如下的计算

```py
p = ...   # insert lengthy computation here
assert p <= 0.05 
```

将确保你的论文中的陈述（a）得到验证，并且（b）可以被其他人验证，包括整个计算路径。（并且连续测试将自动检测是否出现任何差异。）

## 原型设计流程

关于原型设计（使用 Python 或其他语言），有一件很棒的事情是它允许你完全专注于你的*方法*，而不是基础设施。很明显，这对*教学*很有用——你可以在讲座中使用上述例子来快速传达程序分析、测试生成、调试等基本技术。

但原型设计有更多的优势。一个 Jupyter Notebook（就像这个一样）记录了你如何开发你的方法，包括示例、实验和理由——同时仍然关注本质。如果你以“经典”的方式编写工具，你最终会交付数千行代码，这些代码可以做任何事情，但只有在你实现了所有这些之后，你才会知道这些事情实际上是否可行。这是一个巨大的风险，而且如果你还需要更改某些东西，你将不得不一次又一次地重构代码。此外，对于任何将来会处理这段代码的人来说，如果它被埋藏在大量的基础设施和重构之下，可能需要几天甚至几周的时间才能重新提取出方法的基本思想。

在这个阶段，我们的结果是现在我们**两次**实施新想法：

+   首先，我们将事物作为笔记本（就像这个一样）来实现，尝试各种方法和参数，直到我们找到正确的方法。

+   只有当我们确保方法正确，并且我们有信心它能够工作后，我们才在可以处理大规模程序的工具中重新实现它。这仍然可能需要几周到几个月的时间，但至少我们知道我们走在正确的道路上。

顺便说一下，原始笔记本可能寿命更长，因为它们更简单、文档更好，并捕捉到了我们新颖想法的精髓。这就是这本书中几个笔记本的由来。

## 保持事物更新

Python 是一个动态环境。你使用的包越多，将来某天有人可能引入一个改变，使得你的代码无法运行的可能性就越高。因此，我建议你设置一个**持续测试**方案，其中所有笔记本都会在固定的时间间隔自动运行，运行并重新运行你的实验。

对于模糊测试书籍和调试书籍，我每周需要花费大约 30 分钟的时间来更新内容并修复错误。有了持续测试，其他人实际使用你的代码的可能性要高得多（因为它们将在他们的机器上**工作**）。即使作为博士导师，我也可以轻松地花费这些时间，这让我有一种良好的感觉，即其他人实际上可以利用我们的工作。

## 不会工作的事情

Python 以其静态分析困难而闻名，这是事实；其动态特性使得传统的静态分析难以排除特定的行为。

我们认为 Python 是原型设计自动化测试和动态分析技术的优秀语言，也是展示**轻量级**静态和符号分析技术的良好语言，这些技术将被用来**指导**和**支持**其他技术（例如，生成软件测试）。

但如果你只想通过代码的静态分析来**证明**特定的属性（或其不存在），那么 Python 至少是一个挑战；对于某些领域，我们肯定会**警告**不要使用它。

### （无）类型检查

使用 Python 来演示**静态类型检查**将不会是最优的，因为 Python 程序通常不包含类型注解。在编码时，我通常在没有类型注解的情况下开始，但一旦代码稳定并进行了文档化，就会**回溯**添加它们；这是因为我认为有类型签名使得其他人更容易使用代码：

```py
def typed_triangle(a: int, b: int, c: int) -> str:
    return triangle(a, b, c) 
```

对于 Python，有可用的**静态类型检查器**，特别是 *mypy*；这些工具在揭示代码中的类型错误方面做得足够好。尽管如此，如果你的代码经过良好的测试（而且应该是这样），静态类型检查器发现新错误的机会很小。不过，如果你想展示类型检查的优点（或者自己构建新的静态类型检查器或类型系统），Python 可能是一个不错的游乐场。

### （无）程序证明

Python 是一种高度动态的语言，你可以在运行时更改**任何东西**。将不同类型分配给变量没有问题，就像

```py
x = 42
x = "a string" 
```

或者根据某些运行时条件改变变量的存在（和作用域）：

```py
p1, p2 = True, False

if p1:
    x = 42
if p2:
    del x

# Does x exist at this point? 
```

这些特性使得对代码进行符号推理（包括静态分析和类型检查）变得非常困难，如果不是完全不可能的话。如果你需要轻量级的静态和符号分析技术来**指导**其他技术（比如测试生成），那么不精确可能不会造成太大的伤害。但如果你想从你的代码中推导出**保证**，请不要使用 Python 作为测试对象；再次强调，像 Java/ML/Haskell（或一些非常受限的玩具语言）这样的强静态类型语言是实验的更好基础。

这并不意味着像 Python 这样的语言不应该进行静态检查。相反，Python 的广泛应用强烈呼吁更好的静态检查工具。但如果你想教授或研究静态和符号技术，我们绝对不会选择 Python 作为我们的首选语言。

## 尝试一下！

上面所有的代码示例你都可以运行——并且按你的意愿进行修改！从网页上，最简单的方法是转到“资源 $\rightarrow$ 作为笔记本编辑”，你可以在浏览器中直接实验原始的 Jupyter Notebook。（使用 `Shift` + `Return` 来执行代码。）

从“资源”菜单中，你还可以下载 Python 代码（`.py`）以在 Python 环境中运行，或者下载笔记本（`.ipynb`）以在 Jupyter 中运行——同样，按你的意愿进行修改。如果你想在你的机器上运行此代码，你需要以下包：

```py
pip install showast
pip install z3-solver
```

享受吧！

## 经验教训

**Python** 是一个用于原型设计、测试和调试工具的伟大语言：

+   在 Python 中，动态分析和静态分析的实施非常容易。

+   Python 提供了大量的基础设施，用于解析、将程序作为树处理以及约束求解。

+   这些可以在**数小时**内而不是数周内帮助你开发新技术。

尽管如此，Python 并不推荐作为纯符号代码分析领域。

+   静态类型很少，甚至没有

+   这种语言高度动态，几乎没有静态保证。

然而，即使是潜在的*不稳定的*符号分析也能指导测试生成——而且这又非常容易实现。

**Jupyter Notebooks**（使用 Python 或其他语言）非常适合*原型设计*：

+   笔记本记录了你的方法要点，包括示例和实验。

+   这对于教学、沟通甚至文档都是非常好的。

+   在原型上早期进行实验可以降低后续大规模实施的风险。

## 下一步

如果你想看到更多我们使用 Python 进行原型设计的例子——看看这本书这里！特别是，

+   看看我们如何逐步开发 fuzzers；

+   看看我们如何使用动态分析来检查覆盖率；或者

+   看看我们如何分析 Python 代码进行 concolic 和 symbolic 以及模糊测试。

有很多东西要学——享受阅读吧！

## 背景

*三角形问题*是从 Myers 和 Sandler 的《软件测试的艺术》中改编的[[Myers *et al*，2004](https://dl.acm.org/citation.cfm?id=983238)]。这是一个据说很简单的问题，但当你考虑所有可能出错的事情时，它揭示了令人惊讶的深度。

本章中使用的*Z3 求解器*是在微软研究院 Leonardo de Moura 和 Nikolaj Bjørner 的领导下开发的[[De Moura *et al*，2008](https://link.springer.com/chapter/10.1007/978-3-540-78800-3_24)]。它是功能最强大、最受欢迎的求解器之一。

## 练习

### 练习 1：特性！特性！

我们的路径收集器仍然非常有限。不工作的事情包括

+   复杂条件，例如布尔运算符。Python 运算符`a and b`需要翻译成 Z3 语法`z3.And(a, b)`。

+   早期返回。在`if A: return`之后，后续语句的条件`not A`必须成立。

+   作业。

+   循环。

+   函数调用。

实现这些功能越多，你将越接近一个完整的 Python 符号测试生成器。但到了某个阶段，*你的原型可能就不再是原型了*，那时，Python 可能就不再是最佳的语言选择。找到一个合适的时机，从原型工具切换到生产工具。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容根据[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)授权。内容中的源代码，以及用于格式化和显示该内容的源代码，根据[MIT License](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)授权。[最后更改：2023-01-07 13:43:14+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/AcademicPrototyping.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用这篇作品

安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒："[学术原型设计](https://www.fuzzingbook.org/html/AcademicPrototyping.html)"。收录于安德烈亚斯·策勒，拉胡尔·戈皮纳特，马塞尔·博 hme，戈登·弗莱泽，以及克里斯蒂安·霍勒所著的"[模糊测试书籍](https://www.fuzzingbook.org/)"中。[`www.fuzzingbook.org/html/AcademicPrototyping.html`](https://www.fuzzingbook.org/html/AcademicPrototyping.html)。检索日期：2023-01-07 13:43:14+01:00.

```py
@incollection{fuzzingbook2023:AcademicPrototyping,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Academic Prototyping},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/AcademicPrototyping.html}},
    note = {Retrieved 2023-01-07 13:43:14+01:00},
    url = {https://www.fuzzingbook.org/html/AcademicPrototyping.html},
    urldate = {2023-01-07 13:43:14+01:00}
}

```
