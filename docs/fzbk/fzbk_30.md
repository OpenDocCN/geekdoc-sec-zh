# 测试配置

> 原文：[`www.fuzzingbook.org/html/ConfigurationFuzzer.html`](http://www.fuzzingbook.org/html/ConfigurationFuzzer.html)

程序的行为不仅受其数据控制。程序的 *配置* – 即通过选项或配置文件设置的，控制程序在其（常规）输入数据上执行设置的设置 – 同样会影响行为，因此可以也应该进行测试。在本章中，我们探讨了如何系统地 *测试* 和 *覆盖* 软件配置。通过 *自动推断配置选项*，我们可以直接应用这些技术，无需编写语法。最后，我们展示了如何系统地覆盖配置选项的 *组合*，快速检测不希望出现的干扰。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo('L0ztoXVru2U') 
```

**先决条件**

+   您应该已经阅读了关于语法的章节。

+   您应该已经阅读了关于语法覆盖的章节。

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [typing](https://docs.python.org/3/library/typing.html) import List, Union, Optional, Callable, Type 
```

## 概述

要使用本章提供的代码（Importing.html），请编写

```py
>>> from fuzzingbook.ConfigurationFuzzer import <identifier> 
```

然后利用以下功能。

本章提供了两个类：

+   `OptionRunner` 自动从 Python 程序中提取命令行选项；

+   `OptionFuzzer` 使用这些功能自动测试具有大量选项的 Python 程序。

`OptionRunner` 运行程序直到它解析其参数，然后提取一个描述其调用的语法：

```py
>>> autopep8_runner = OptionRunner("autopep8", "foo.py") 
```

语法可以通过 `ebnf_grammar()` 方法提取：

```py
>>> option_ebnf_grammar = autopep8_runner.ebnf_grammar()
>>> option_ebnf_grammar
{'<start>': ['(<option>)*<arguments>'],
 '<option>': [' -h',
  ' --help',
  ' --version',
  ' -v',
  ' --verbose',
  ' -d',
  ' --diff',
  ' -i',
  ' --in-place',
  ' --global-config <filename>',
  ' --ignore-local-config',
  ' -r',
  ' --recursive',
  ' -j <n>',
  ' --jobs <n>',
  ' -p <n>',
  ' --pep8-passes <n>',
  ' -a',
  ' --aggressive',
  ' --experimental',
  ' --exclude <globs>',
  ' --list-fixes',
  ' --ignore <errors>',
  ' --select <errors>',
  ' --max-line-length <n>',
  ' --line-range <line> <line>',
  ' --range <line> <line>',
  ' --indent-size <int>',
  ' --hang-closing',
  ' --exit-code'],
 '<arguments>': [' foo.py'],
 '<str>': ['<char>+'],
 '<char>': ['0',
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
  'f',
  'g',
  'h',
  'i',
  'j',
  'k',
  'l',
  'm',
  'n',
  'o',
  'p',
  'q',
  'r',
  's',
  't',
  'u',
  'v',
  'w',
  'x',
  'y',
  'z',
  'A',
  'B',
  'C',
  'D',
  'E',
  'F',
  'G',
  'H',
  'I',
  'J',
  'K',
  'L',
  'M',
  'N',
  'O',
  'P',
  'Q',
  'R',
  'S',
  'T',
  'U',
  'V',
  'W',
  'X',
  'Y',
  'Z',
  '!',
  '"',
  '#',
  '$',
  '%',
  '&',
  "'",
  '(',
  ')',
  '*',
  '+',
  ',',
  '-',
  '.',
  '/',
  ':',
  ';',
  '<',
  '=',
  '>',
  '?',
  '@',
  '[',
  '\\',
  ']',
  '^',
  '_',
  '`',
  '{',
  '|',
  '}',
  '~'],
 '<filename>': ['<str>'],
 '<int>': ['(-)?<digit>+'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<n>': ['<int>'],
 '<globs>': ['<str>'],
 '<errors>': ['<str>'],
 '<line>': ['<int>']} 
```

语法可以立即用于模糊测试。`GrammarCoverageFuzzer` 将确保所有选项都被覆盖：

```py
>>> from Grammars import convert_ebnf_grammar
>>> fuzzer = GrammarCoverageFuzzer(convert_ebnf_grammar(option_ebnf_grammar))
>>> [fuzzer.fuzz() for i in range(3)]
[' --max-line-length -64 foo.py',
 " -a --version --select u --diff --list-fixes -r --range -50 3 --ignore iq --hang-closing --ignore-local-config --aggressive --in-place --indent-size 9 -d --global-config wQ --help --line-range -8226 7 -j 1 -p 2 -h --experimental -i -v --jobs -81 --exclude c --exit-code --verbose --recursive --exclude j --pep8-passes 587 --global-config 5ty --global-config W]96{< --ignore M --exclude . --select > --global-config T --ignore z'C --select %EIL -r -a -v foo.py",
 ' --exclude VKSl --exclude 3[ --global-config ⁰x --ignore 4 --exclude _ --global-config 7 --select mh --global-config e --global-config R --global-config JH\\ --exclude OP --ignore = --global-config @ --select ?N --global-config s --select *}v --ignore - --select Do,GA --exclude bkn --ignore U --exclude | --global-config 2 --exclude 8 --select " --exclude pZ --select / --exclude f(aYdg) --select : --global-config ~ --global-config ! --global-config 1B`$X --exclude ; --select F --select & --ignore r --exclude #+ --exit-code --aggressive --select ?/ -p -6 --version --ignore-local-config -r -v foo.py'] 
```

`OptionFuzzer` 类总结了这些步骤。其构造函数接受一个 `OptionRunner` 以自动提取语法；它执行必要的步骤来提取语法并对其进行模糊测试。

```py
>>> autopep8_runner = OptionRunner("autopep8", "foo.py")
>>> autopep8_fuzzer = OptionFuzzer(autopep8_runner)
>>> [autopep8_fuzzer.fuzz() for i in range(3)]
[' --version foo.py',
 ' --ignore Rj --jobs -6 --line-range -04 7953 --help --aggressive -v --ignore-local-config -h --experimental -r --global-config 3 --indent-size -8 --exclude ;&M"! -a -p 1 --hang-closing -j 263 --verbose --recursive --in-place --list-fixes --select t@fs --range -0 7 -d --pep8-passes 7 --diff -i --exit-code --max-line-length -3 --global-config L --select u| --global-config \' --global-config r --exclude Ik) --ignore D- --ignore 4 --select XS --global-config 7 --ignore ~. --ignore e9 --exclude ph --ignore U --global-config dz --global-config Q$o --exclude 6( --ignore x --select 5J:N --exclude < --ignore O --global-config ,c --global-config = --exclude W\\ --global-config ? --select l --select ^ --select V --exclude P --select Z --select >0 --select TH --exclude * --exclude G --select 8 --global-config bn --global-config C{a1m --exclude F --ignore _ --ignore g --ignore ]q --exclude wy --ignore % --global-config v --ignore + --global-config EK/ --ignore [#}Y --global-config `i --global-config 2 --ignore BE --global-config A --indent-size -22 --recursive --hang-closing --exclude ` --indent-size -61 --diff -a foo.py',
 ' foo.py'] 
```

测试的最后一步现在是将这些参数调用程序。

注意，`OptionRunner` 是实验性的：它假设相关的 Python 程序使用了 `argparse` 模块；并且并非所有 `argparse` 功能都得到支持。尽管如此，它即使在非平凡程序上也能做得相当不错。

`OptionRunner` 构造函数接受一个额外的 `miner` 关键字参数，该参数接受要使用的参数语法挖掘器的类。默认情况下，这是 `OptionGrammarMiner` – 一个辅助类，可以用来（并扩展）创建自己的选项语法挖掘器。

<svg width="586pt" height="771pt" viewBox="0.00 0.00 586.12 771.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 767)"><g id="node1" class="node"><title>OptionRunner</title> <g id="a_node1"><a xlink:href="#" xlink:title="class OptionRunner:

在确定其选项语法的同时运行程序"><text text-anchor="start" x="14" y="-168.82" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">OptionRunner</text> <g id="a_node1_0"><a xlink:href="#" xlink:title="OptionRunner"><g id="a_node1_1"><a xlink:href="#" xlink:title="__init__(self, program: Union[str, List[str]], arguments: Optional[str] = None, *, log: bool = False, miner_class: Optional[Type[OptionGrammarMiner]] = None):

构造函数。

`program` - 要执行的（Python）程序

`arguments` - 一个可选的字符串，包含用于`program`的参数

`log` - 如果为 True，在挖掘器中启用日志记录

`miner_class` - 要使用的`OptionGrammarMiner`类

(默认: `OptionGrammarMiner`)"><text text-anchor="start" x="9.5" y="-146.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node1_2"><a xlink:href="#" xlink:title="ebnf_grammar(self):

返回以 EBNF 形式提取的语法"><text text-anchor="start" x="9.5" y="-133.88" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">ebnf_grammar()</text></a></g> <g id="a_node1_3"><a xlink:href="#" xlink:title="grammar(self):

返回以 BNF 形式提取的语法"><text text-anchor="start" x="9.5" y="-121.12" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">grammar()</text></a></g> <g id="a_node1_4"><a xlink:href="#" xlink:title="executable(self)"><text text-anchor="start" x="9.5" y="-107.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">executable()</text></a></g> <g id="a_node1_5"><a xlink:href="#" xlink:title="find_contents(self)"><text text-anchor="start" x="9.5" y="-94.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">find_contents()</text></a></g> <g id="a_node1_6"><a xlink:href="#" xlink:title="find_grammar(self)"><text text-anchor="start" x="9.5" y="-81.88" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">find_grammar()</text></a></g> <g id="a_node1_7"><a xlink:href="#" xlink:title="invoker(self)"><text text-anchor="start" x="9.5" y="-69.12" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">invoker()</text></a></g> <g id="a_node1_8"><a xlink:href="#" xlink:title="set_arguments(self, args)"><text text-anchor="start" x="9.5" y="-56.38" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">set_arguments()</text></a></g> <g id="a_node1_9"><a xlink:href="#" xlink:title="set_invocation(self, program)"><text text-anchor="start" x="9.5" y="-43.62" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">set_invocation()</text></a></g></a></g></a></g></g> <g id="node2" class="node"><title>ProgramRunner</title> <g id="a_node2"><a xlink:href="Fuzzer.html" xlink:title="class ProgramRunner:

使用输入测试程序。<text text-anchor="start" x="8" y="-288.45" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">ProgramRunner</text> <g id="a_node2_10"><a xlink:href="#" xlink:title="ProgramRunner"><g id="a_node2_11"><a xlink:href="Fuzzer.html" xlink:title="__init__(self, program: Union[str, List[str]]) -> None:

初始化。

`program` 是传递给 `subprocess.run()` 的程序规范 `<text text-anchor="start" x="27.5" y="-266.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g></a></g></a></g></g> <g id="edge1" class="edge"><title>OptionRunner->ProgramRunner</title></g> <g id="node3" class="node"><title>Runner</title> <g id="a_node3"><a xlink:href="Fuzzer.html" xlink:title="class Runner:

测试输入的基本类。"><text text-anchor="start" x="34.62" y="-431.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Runner</text> <g id="a_node3_12"><a xlink:href="#" xlink:title="Runner"><g id="a_node3_13"><a xlink:href="Fuzzer.html" xlink:title="FAIL = 'FAIL'"><text text-anchor="start" x="27.5" y="-408" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">FAIL</text></a></g> <g id="a_node3_14"><a xlink:href="Fuzzer.html" xlink:title="PASS = 'PASS'"><text text-anchor="start" x="27.5" y="-395.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">PASS</text></a></g> <g id="a_node3_15"><a xlink:href="Fuzzer.html" xlink:title="UNRESOLVED = 'UNRESOLVED'"><text text-anchor="start" x="27.5" y="-382.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">UNRESOLVED</text></a></g></a></g> <g id="a_node3_16"><a xlink:href="#" xlink:title="Runner"><g id="a_node3_17"><a xlink:href="Fuzzer.html" xlink:title="__init__(self) -> None:

初始化"><text text-anchor="start" x="27.5" y="-362.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node3_18"><a xlink:href="Fuzzer.html" xlink:title="run(self, inp: str) -> Any:

使用给定的输入运行 runner"><text text-anchor="start" x="27.5" y="-350" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">run()</text></a></g></a></g></a></g></g> <g id="edge2" class="edge"><title>ProgramRunner->Runner</title></g> <g id="node4" class="node"><title>OptionFuzzer</title> <g id="a_node4"><a xlink:href="#" xlink:title="class OptionFuzzer:

使用程序的参数对 (Python) 程序进行模糊测试"><text text-anchor="start" x="179.25" y="-124.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">OptionFuzzer</text> <g id="a_node4_19"><a xlink:href="#" xlink:title="OptionFuzzer"><g id="a_node4_20"><a xlink:href="#" xlink:title="__init__(self, runner: OptionRunner, *args, **kwargs):

构造函数。`runner` 是一个 OptionRunner。"><text text-anchor="start" x="190.5" y="-102" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node4_21"><a xlink:href="#" xlink:title="run(self, runner=None, inp=''):

运行 `runner` 并使用模糊输入"><text text-anchor="start" x="190.5" y="-89.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">run()</text></a></g></a></g></a></g></g> <g id="node5" class="node"><title>GrammarCoverageFuzzer</title> <g id="a_node5"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="class GrammarCoverageFuzzer:

从语法生成字符串，旨在覆盖所有扩展。"><text text-anchor="start" x="141.75" y="-278.07" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarCoverageFuzzer</text></a></g></g> <g id="edge3" class="edge"><title>OptionFuzzer->GrammarCoverageFuzzer</title></g> <g id="node6" class="node"><title>SimpleGrammarCoverageFuzzer</title> <g id="a_node6"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="class SimpleGrammarCoverageFuzzer:

在选择扩展时，优先选择未被覆盖的扩展。"><text text-anchor="start" x="121.12" y="-391.32" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">SimpleGrammarCoverageFuzzer</text></a></g></g> <g id="edge4" class="edge"><title>GrammarCoverageFuzzer->SimpleGrammarCoverageFuzzer</title></g> <g id="node7" class="node"><title>TrackingGrammarCoverageFuzzer</title> <g id="a_node7"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="class TrackingGrammarCoverageFuzzer:

从语法生成字符串，旨在覆盖所有扩展。"><text text-anchor="start" x="114.38" y="-514.95" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">TrackingGrammarCoverageFuzzer</text> <g id="a_node7_22"><a xlink:href="#" xlink:title="TrackingGrammarCoverageFuzzer"><g id="a_node7_23"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="__init__(self, *args, **kwargs) -> None:

从 `grammar` 生成字符串，以 `start_symbol` 为起点。

如果设置了 `disp`，则显示中间推导树。

在生成过程中跟踪语法覆盖率。"><text text-anchor="start" x="170.62" y="-624.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">GrammarFuzzer</text> <g id="a_node8_24"><a xlink:href="#" xlink:title="GrammarFuzzer"><g id="a_node8_25"><a xlink:href="GrammarFuzzer.html" xlink:title="__init__(self, grammar: Dict[str, List[Expansion]], start_symbol: str = '<start>', min_nonterminals: int = 0, max_nonterminals: int = 10, disp: bool = False, log: Union[bool, int] = False) -> None:

生成从 `grammar` 开始的字符串，以 `start_symbol` 为起点。

如果设置了 `log`，则将中间步骤作为文本输出到标准输出。"><text text-anchor="start" x="190.5" y="-492.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g></a></g></a></g></g> <g id="edge5" class="edge"><title>SimpleGrammarCoverageFuzzer->TrackingGrammarCoverageFuzzer</title></g> <g id="node8" class="node"><title>GrammarFuzzer</title> <g id="a_node8"><a xlink:href="GrammarFuzzer.html" xlink:title="class GrammarFuzzer:

生成从 `grammar` 开始的字符串，以 `start_symbol` 为起点。

从 `grammar` 生成字符串，以 `start_symbol` 为起点。

如果提供了 `min_nonterminals` 或 `max_nonterminals`，则使用它们作为限制。

对于产生的非终结符数量。

如果`disp`被设置，显示中间推导树。

如果`log`被设置，将中间步骤以文本形式显示在标准输出上。"><text text-anchor="start" x="187.5" y="-602" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node8_26"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz(self) -> str:

从语法中生成一个字符串。"><text text-anchor="start" x="187.5" y="-589.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node8_27"><a xlink:href="GrammarFuzzer.html" xlink:title="fuzz_tree(self) -> DerivationTree:

从语法中生成一个推导树。"><text text-anchor="start" x="187.5" y="-576.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">fuzz_tree()</text></a></g></a></g></a></g></g> <g id="edge6" class="edge"><title>TrackingGrammarCoverageFuzzer->GrammarFuzzer</title></g> <g id="node9" class="node"><title>Fuzzer</title> <g id="a_node9"><a xlink:href="Fuzzer.html" xlink:title="class Fuzzer:

模糊器的基类。"><text text-anchor="start" x="199.88" y="-746.2" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">Fuzzer</text> <g id="a_node9_28"><a xlink:href="#" xlink:title="Fuzzer"><g id="a_node9_29"><a xlink:href="Fuzzer.html" xlink:title="__init__(self) -> None:

构造函数"><text text-anchor="start" x="190.5" y="-724" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">__init__()</text></a></g> <g id="a_node9_30"><a xlink:href="Fuzzer.html" xlink:title="fuzz(self) -> str:

返回模糊输入"><text text-anchor="start" x="190.5" y="-711.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">fuzz()</text></a></g> <g id="a_node9_31"><a xlink:href="Fuzzer.html" xlink:title="run(self, runner: Fuzzer.Runner = <Fuzzer.Runner object>) -> Tuple[subprocess.CompletedProcess, str]:

使用模糊输入运行`runner`"><text text-anchor="start" x="190.5" y="-698.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-style="italic" font-size="10.00">run()</text></a></g> <g id="a_node9_32"><a xlink:href="Fuzzer.html" xlink:title="runs(self, runner: Fuzzer.Runner = <Fuzzer.PrintRunner object>, trials: int = 10) -> List[Tuple[subprocess.CompletedProcess, str]]:

运行 `runner`，使用模糊输入，`trials` 次数"><text text-anchor="start" x="190.5" y="-685.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">runs()</text></a></g></a></g></a></g></g> <g id="edge7" class="edge"><title>GrammarFuzzer->Fuzzer</title></g> <g id="node10" class="node"><title>OptionGrammarMiner</title> <g id="a_node10"><a xlink:href="#" xlink:title="class OptionGrammarMiner:

用于提取选项语法的辅助类"><text text-anchor="start" x="296.25" y="-204.7" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="14.00" fill="#b03a2e">OptionGrammarMiner</text> <g id="a_node10_33"><a xlink:href="#" xlink:title="OptionGrammarMiner"><g id="a_node10_34"><a xlink:href="#" xlink:title="ARGUMENTS_SYMBOL = '<arguments>'"><text text-anchor="start" x="316.5" y="-181.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">ARGUMENTS_SYMBOL</text></a></g> <g id="a_node10_35"><a xlink:href="#" xlink:title="OPTION_SYMBOL = '<option>'"><text text-anchor="start" x="316.5" y="-168.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">OPTION_SYMBOL</text></a></g></a></g> <g id="a_node10_36"><a xlink:href="#" xlink:title="OptionGrammarMiner"><g id="a_node10_37"><a xlink:href="#" xlink:title="__init__(self, function: Callable, log: bool = False):

构造函数。

`function` - 使用 argparse() 处理参数的函数

`log` - 如果为 True，则输出诊断信息"><text text-anchor="start" x="307.5" y="-149" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">__init__()</text></a></g> <g id="a_node10_38"><a xlink:href="#" xlink:title="mine_ebnf_grammar(self):

提取 EBNF 选项语法"><text text-anchor="start" x="307.5" y="-136.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">mine_ebnf_grammar()</text></a></g> <g id="a_node10_39"><a xlink:href="#" xlink:title="mine_grammar(self):

提取 BNF 选项语法"><text text-anchor="start" x="307.5" y="-123.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="10.00">mine_grammar()</text></a></g> <g id="a_node10_40"><a xlink:href="#" xlink:title="add_group(self, locals, exclusive)"><text text-anchor="start" x="307.5" y="-109.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">add_group()</text></a></g> <g id="a_node10_41"><a xlink:href="#" xlink:title="add_int_rule(self)"><text text-anchor="start" x="307.5" y="-97" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">add_int_rule()</text></a></g> <g id="a_node10_42"><a xlink:href="#" xlink:title="add_metavar_rule(self, metavar, type_)"><text text-anchor="start" x="307.5" y="-84.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">add_metavar_rule()</text></a></g> <g id="a_node10_43"><a xlink:href="#" xlink:title="add_parameter(self, kwargs, metavar)"><text text-anchor="start" x="307.5" y="-71.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">add_parameter()</text></a></g> <g id="a_node10_44"><a xlink:href="#" xlink:title="add_str_rule(self)"><text text-anchor="start" x="307.5" y="-58.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">add_str_rule()</text></a></g> <g id="a_node10_45"><a xlink:href="#" xlink:title="add_type_rule(self, type_)"><text text-anchor="start" x="307.5" y="-46" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">add_type_rule()</text></a></g> <g id="a_node10_46"><a xlink:href="#" xlink:title="process_arg(self, arg, in_group, kwargs)"><text text-anchor="start" x="307.5" y="-33.25" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">process_arg()</text></a></g> <g id="a_node10_47"><a xlink:href="#" xlink:title="process_argument(self, locals, in_group)"><text text-anchor="start" x="307.5" y="-20.5" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">process_argument()</text></a></g> <g id="a_node10_48"><a xlink:href="#" xlink:title="traceit(self, frame, event, arg)"><text text-anchor="start" x="307.5" y="-7.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="10.00">traceit()</text></a></g></a></g></a></g></g> <g id="node11" class="node"><title>图例</title> <text text-anchor="start" x="458.88" y="-126.75" font-family="Patua One, Helvetica, sans-serif" font-weight="bold" font-size="10.00" fill="#b03a2e">图例</text> <text text-anchor="start" x="458.88" y="-116.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="464.88" y="-116.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-weight="bold" font-size="8.00">公共方法()</text> <text text-anchor="start" x="458.88" y="-106.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="464.88" y="-106.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-size="8.00">私有方法()</text> <text text-anchor="start" x="458.88" y="-96.75" font-family="Patua One, Helvetica, sans-serif" font-size="10.00">• </text> <text text-anchor="start" x="464.88" y="-96.75" font-family="'Fira Mono', 'Source Code Pro', 'Courier', monospace" font-style="italic" font-size="8.00">重载方法()</text> <text text-anchor="start" x="458.88" y="-87.7" font-family="Helvetica,sans-Serif" font-size="9.00">将鼠标悬停在名称上以查看文档</text></g></g></svg>

## 配置选项

当我们谈论程序的输入时，我们通常想到的是它处理的*数据*。这也是我们在过去章节中模糊测试的内容——无论是使用随机输入、基于变异的模糊测试还是基于语法的模糊测试。然而，程序通常有几个输入来源，所有这些都可以也应该被测试——并包含在测试生成中。

一个重要的输入来源是程序的*配置*——也就是说，一组输入，通常在开始处理数据时设置一次，然后在程序运行时保持不变，甚至在程序部署时保持不变。此类配置通常在*配置文件*中设置（例如，作为键/值对）；然而，对于命令行工具来说，最普遍的方法是在命令行上的*配置选项*。

例如，考虑`grep`实用程序在文件中查找文本模式。`grep`工作的确切模式由大量选项控制，可以通过提供`--help`选项来列出：

```py
!grep  --help 
```

```py
usage: grep [-abcdDEFGHhIiJLlMmnOopqRSsUVvwXxZz] [-A num] [-B num] [-C[num]]
	[-e pattern] [-f file] [--binary-files=value] [--color=when]
	[--context[=num]] [--directories=action] [--label] [--line-buffered]
	[--null] [pattern] [file ...]

```

所有这些选项都需要测试它们是否正确操作。在安全测试中，任何此类选项也可能触发尚未知的漏洞。因此，此类选项可以成为自己的*模糊目标*。在本章中，我们分析如何系统地测试此类选项——而且更好的是，如何从给定的程序文件中提取可能的配置，这样我们就不需要指定任何内容。

## Python 中的选项

让我们继续使用我们共同的编程语言，并检查在 Python 中如何处理选项。`argparse` 模块提供了一个具有强大功能——以及复杂性的命令行参数（和选项）解析器。你首先定义一个解析器（`argparse.ArgumentParser()`），然后逐个添加具有各种特性的单个参数。每个参数的附加参数可以指定参数的类型（`type`）（例如，整数或字符串），或参数的数量（`nargs`）。

默认情况下，参数存储在`parse_args()`返回的`args`对象中，其名称下——因此，`args.integers`包含之前添加的`integer`参数。特殊操作（`actions`）允许在给定的变量中存储特定值；`store_const`操作将给定的`const`存储在由`dest`命名的属性中。以下示例接受多个整数参数（`integers`）以及要应用于这些整数的操作符（`--sum`、`--min`或`--max`）。所有操作符都将函数引用存储在`accumulate`属性中，最终在解析的整数上调用：

```py
import [argparse](https://docs.python.org/3/library/argparse.html) 
```

```py
def process_numbers(args=[]):
    parser = argparse.ArgumentParser(description='Process some integers.')
    parser.add_argument('integers', metavar='N', type=int, nargs='+',
                        help='an integer for the accumulator')
    group = parser.add_mutually_exclusive_group(required=True)
    group.add_argument('--sum', dest='accumulate', action='store_const',
                       const=sum,
                       help='sum the integers')
    group.add_argument('--min', dest='accumulate', action='store_const',
                       const=min,
                       help='compute the minimum')
    group.add_argument('--max', dest='accumulate', action='store_const',
                       const=max,
                       help='compute the maximum')

    args = parser.parse_args(args)
    print(args.accumulate(args.integers)) 
```

下面是如何`process_numbers()`工作的。例如，我们可以对给定的参数调用`--min`选项来计算最小值：

```py
process_numbers(["--min", "100", "200", "300"]) 
```

```py
100

```

或者计算三个数字的总和：

```py
process_numbers(["--sum", "1", "2", "3"]) 
```

```py
6

```

当通过`add_mutually_exclusive_group()`（如上所示）定义时，选项是互斥的。因此，我们只能有一个操作符：

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from ExpectError import ExpectError 
```

```py
with ExpectError(SystemExit, print_traceback=False):
    process_numbers(["--sum", "--max", "1", "2", "3"]) 
```

```py
usage: ipykernel_launcher.py [-h] (--sum | --min | --max) N [N ...]
ipykernel_launcher.py: error: argument --max: not allowed with argument --sum
SystemExit: 2 (expected)

```

## 配置的语法

如何测试具有多个选项的系统？最简单的答案是为它编写一个语法。`PROCESS_NUMBERS_EBNF_GRAMMAR`语法反映了选项和参数的可能组合：

```py
from Grammars import crange, srange, convert_ebnf_grammar, extend_grammar, is_valid_grammar
from Grammars import START_SYMBOL, new_symbol, Grammar 
```

```py
PROCESS_NUMBERS_EBNF_GRAMMAR: Grammar = {
    "<start>": ["<operator> <integers>"],
    "<operator>": ["--sum", "--min", "--max"],
    "<integers>": ["<integer>", "<integers> <integer>"],
    "<integer>": ["<digit>+"],
    "<digit>": crange('0', '9')
}

assert is_valid_grammar(PROCESS_NUMBERS_EBNF_GRAMMAR) 
```

```py
PROCESS_NUMBERS_GRAMMAR = convert_ebnf_grammar(PROCESS_NUMBERS_EBNF_GRAMMAR) 
```

我们可以将这个语法输入到我们的语法覆盖率模糊测试工具中，并让它逐个覆盖选项：

```py
from GrammarCoverageFuzzer import GrammarCoverageFuzzer 
```

```py
f = GrammarCoverageFuzzer(PROCESS_NUMBERS_GRAMMAR, min_nonterminals=10)
for i in range(3):
    print(f.fuzz()) 
```

```py
--max 9 5 8 210 80 9756431
--sum 9 4 99 1245 612370
--min 2 3 0 46 15798 7570926

```

当然，我们也可以使用这些参数调用`process_numbers()`。为此，我们需要使用`split()`将语法产生的字符串转换回一个包含单个参数的列表：

```py
f = GrammarCoverageFuzzer(PROCESS_NUMBERS_GRAMMAR, min_nonterminals=10)
for i in range(3):
    args = f.fuzz().split()
    print(args)
    process_numbers(args) 
```

```py
['--max', '8', '9', '3067', '44', '13852967057']
13852967057
['--sum', '9', '8', '63', '9278111', '59206197798']
59215475989
['--min', '4', '1', '4864', '33342', '7827970808951']
1

```

同样，我们可以为任何要测试的程序定义语法；以及为配置文件定义语法。然而，每当程序发生变化时，都需要更新语法，这会带来维护负担。鉴于语法所需的信息已经全部编码在程序中，问题随之而来：*我们为什么不能一开始就从程序中提取配置选项呢？*

## 挖掘配置选项

在本节中，我们尝试直接从程序中提取选项和参数信息，这样我们就不需要指定配置语法。目标是拥有一个能够处理任意程序选项和参数的配置模糊测试工具，只要它遵循特定的参数处理约定。对于 Python 程序来说，这意味着使用`argparse`模块。

我们的思路如下：我们执行给定的程序，直到实际解析参数为止——即调用`argparse.parse_args()`。在此之前的所有调用，特别是定义参数和选项的调用（`add_argument()`），我们都会进行跟踪。从这些调用中，我们构建语法。

### 跟踪参数

让我们用一个简单的实验来说明这种方法：我们定义一个跟踪函数（有关详细信息，请参阅我们的覆盖率章节），在调用`process_numbers`时处于活动状态。如果我们有一个调用`add_argument`的方法，我们将访问并打印出局部变量（此时是方法的参数）。

```py
import [sys](https://docs.python.org/3/library/sys.html) 
```

```py
import [string](https://docs.python.org/3/library/string.html) 
```

```py
def trace_locals(frame, event, arg):
    if event != "call":
        return
    method_name = frame.f_code.co_name
    if method_name != "add_argument":
        return
    locals = frame.f_locals
    print(method_name, locals) 
```

我们得到的是所有`add_argument()`调用的列表，以及传递给方法的参数：

```py
sys.settrace(trace_locals)
process_numbers(["--sum", "1", "2", "3"])
sys.settrace(None) 
```

```py
add_argument {'self': ArgumentParser(prog='ipykernel_launcher.py', usage=None, description='Process some integers.', formatter_class=<class 'argparse.HelpFormatter'>, conflict_handler='error', add_help=True), 'args': ('-h', '--help'), 'kwargs': {'action': 'help', 'default': '==SUPPRESS==', 'help': 'show this help message and exit'}}
add_argument {'self': ArgumentParser(prog='ipykernel_launcher.py', usage=None, description='Process some integers.', formatter_class=<class 'argparse.HelpFormatter'>, conflict_handler='error', add_help=True), 'args': ('integers',), 'kwargs': {'metavar': 'N', 'type': <class 'int'>, 'nargs': '+', 'help': 'an integer for the accumulator'}}
add_argument {'self': <argparse._MutuallyExclusiveGroup object at 0x11142f260>, 'args': ('--sum',), 'kwargs': {'dest': 'accumulate', 'action': 'store_const', 'const': <built-in function sum>, 'help': 'sum the integers'}}
add_argument {'self': <argparse._MutuallyExclusiveGroup object at 0x11142f260>, 'args': ('--min',), 'kwargs': {'dest': 'accumulate', 'action': 'store_const', 'const': <built-in function min>, 'help': 'compute the minimum'}}
add_argument {'self': <argparse._MutuallyExclusiveGroup object at 0x11142f260>, 'args': ('--max',), 'kwargs': {'dest': 'accumulate', 'action': 'store_const', 'const': <built-in function max>, 'help': 'compute the maximum'}}
6

```

从`args`参数中，我们可以访问要定义的各个选项和参数：

```py
def trace_options(frame, event, arg):
    if event != "call":
        return
    method_name = frame.f_code.co_name
    if method_name != "add_argument":
        return
    locals = frame.f_locals
    print(locals['args']) 
```

```py
sys.settrace(trace_options)
process_numbers(["--sum", "1", "2", "3"])
sys.settrace(None) 
```

```py
('-h', '--help')
('integers',)
('--sum',)
('--min',)
('--max',)
6

```

我们可以看到，每个参数都是一个包含一个成员（例如，`integers`或`--sum`）或两个成员（`-h`和`--help`）的元组，这表示同一选项的替代形式。我们的任务将是遍历`add_arguments()`的参数，不仅检测选项和参数的名称，还要检测它们是否接受额外的参数，以及参数的类型。

### 选项和参数的语法挖掘器

让我们现在构建一个类，它收集所有这些信息以创建一个语法。

我们使用`ParseInterrupt`异常在收集所有参数和选项后中断程序执行：

```py
class ParseInterrupt(Exception):
    pass 
```

`OptionGrammarMiner` 类接受一个可执行函数，用于从中挖掘选项和参数的语法：

```py
class OptionGrammarMiner:
  """Helper class for extracting option grammars"""

    def __init__(self, function: Callable, log: bool = False):
  """Constructor.
 `function` - a function processing arguments using argparse()
 `log` - output diagnostics if True
 """
        self.function = function
        self.log = log 
```

`mine_ebnf_grammar()` 方法是所有事情发生的地方。它创建一个形式的语法

```py
<start> ::= <option>* <arguments>
<option> ::= <empty>
<arguments> ::= <empty>
```

在其中将收集选项和参数。然后它设置一个跟踪函数（有关详细信息，请参阅我们的覆盖率章节），在之前定义的 `function` 被调用时处于活动状态。当调用 `parse_args()` 时引发 `ParseInterrupt` 将结束执行。

```py
class OptionGrammarMiner(OptionGrammarMiner):
    OPTION_SYMBOL = "<option>"
    ARGUMENTS_SYMBOL = "<arguments>"

    def mine_ebnf_grammar(self):
  """Extract EBNF option grammar"""
        self.grammar = {
            START_SYMBOL: ["(" + self.OPTION_SYMBOL + ")*" + self.ARGUMENTS_SYMBOL],
            self.OPTION_SYMBOL: [],
            self.ARGUMENTS_SYMBOL: []
        }
        self.current_group = self.OPTION_SYMBOL

        old_trace = sys.gettrace()
        sys.settrace(self.traceit)
        try:
            self.function()
        except ParseInterrupt:
            pass
        sys.settrace(old_trace)

        return self.grammar

    def mine_grammar(self):
  """Extract BNF option grammar"""
        return convert_ebnf_grammar(self.mine_ebnf_grammar()) 
```

跟踪函数检查四个方法：`add_argument()` 是最重要的函数，它会导致处理参数；`frame.f_locals` 再次是局部变量的集合，此时主要是 `add_argument()` 的参数。由于互斥组也有 `add_argument()` 方法，我们设置 `in_group` 标志以区分。

注意，我们没有做出任何具体努力来区分多个解析器或组；我们只是假设存在一个解析器，并且在任何时刻最多只有一个互斥组。

```py
class OptionGrammarMiner(OptionGrammarMiner):
    def traceit(self, frame, event, arg):
        if event != "call":
            return

        if "self" not in frame.f_locals:
            return

        self_var = frame.f_locals["self"]
        method_name = frame.f_code.co_name

        if method_name == "add_argument":
            in_group = repr(type(self_var)).find("Group") >= 0
            self.process_argument(frame.f_locals, in_group)
        elif method_name == "add_mutually_exclusive_group":
            self.add_group(frame.f_locals, exclusive=True)
        elif method_name == "add_argument_group":
            # self.add_group(frame.f_locals, exclusive=False)
            pass
        elif method_name == "parse_args":
            raise ParseInterrupt

        return self.traceit 
```

`process_arguments()` 现在分析传递的参数并将它们添加到语法中：

+   如果参数以 `-` 开头，它会被添加到 `<option>` 列表中的可选元素。

+   否则，它会被添加到 `<argument>` 列表中。

可选的 `nargs` 参数指定可以跟随多少个参数。如果它是一个数字，我们就在语法中添加相应数量的元素；如果它是一个抽象指定符（例如，`+` 或 `*`），我们就直接将其用作 EBNF 操作符。

由于参数数量众多且具有可选行为，这是一个相当混乱的函数，但它确实完成了工作。

```py
class OptionGrammarMiner(OptionGrammarMiner):
    def process_argument(self, locals, in_group):
        args = locals["args"]
        kwargs = locals["kwargs"]

        if self.log:
            print(args)
            print(kwargs)
            print()

        for arg in args:
            self.process_arg(arg, in_group, kwargs) 
```

```py
class OptionGrammarMiner(OptionGrammarMiner):
    def process_arg(self, arg, in_group, kwargs):
        if arg.startswith('-'):
            if not in_group:
                target = self.OPTION_SYMBOL
            else:
                target = self.current_group
            metavar = None
            arg = " " + arg
        else:
            target = self.ARGUMENTS_SYMBOL
            metavar = arg
            arg = ""

        if "nargs" in kwargs:
            nargs = kwargs["nargs"]
        else:
            nargs = 1

        param = self.add_parameter(kwargs, metavar)
        if param == "":
            nargs = 0

        if isinstance(nargs, int):
            for i in range(nargs):
                arg += param
        else:
            assert nargs in "?+*"
            arg += '(' + param + ')' + nargs

        if target == self.OPTION_SYMBOL:
            self.grammar[target].append(arg)
        else:
            self.grammar[target].append(arg) 
```

`add_parameter()` 方法处理选项的可能参数。如果参数定义了 `action`，则它不取任何参数。否则，我们识别参数的类型（如 `int` 或 `str`）并使用适当的规则增强语法。

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
class OptionGrammarMiner(OptionGrammarMiner):
    def add_parameter(self, kwargs, metavar):
        if "action" in kwargs:
            # No parameter
            return ""

        type_ = "str"
        if "type" in kwargs:
            given_type = kwargs["type"]
            # int types come as '<class int>'
            if inspect.isclass(given_type) and issubclass(given_type, int):
                type_ = "int"

        if metavar is None:
            if "metavar" in kwargs:
                metavar = kwargs["metavar"]
            else:
                metavar = type_

        self.add_type_rule(type_)
        if metavar != type_:
            self.add_metavar_rule(metavar, type_)

        param = " <" + metavar + ">"

        return param 
```

`add_type_rule()` 方法将参数类型的规则添加到语法中。如果参数由元变量（例如，`N`）标识，我们也会添加一个规则以提高可读性。

```py
class OptionGrammarMiner(OptionGrammarMiner):
    def add_type_rule(self, type_):
        if type_ == "int":
            self.add_int_rule()
        else:
            self.add_str_rule()

    def add_int_rule(self):
        self.grammar["<int>"] = ["(-)?<digit>+"]
        self.grammar["<digit>"] = crange('0', '9')

    def add_str_rule(self):
        self.grammar["<str>"] = ["<char>+"]
        self.grammar["<char>"] = srange(
            string.digits
            + string.ascii_letters
            + string.punctuation)

    def add_metavar_rule(self, metavar, type_):
        self.grammar["<" + metavar + ">"] = ["<" + type_ + ">"] 
```

`add_group()` 方法将一个新的互斥组添加到语法中。我们为添加到组中的选项定义了一个新符号（例如，`<group>`），并使用 `required` 和 `exclusive` 标志定义适当的扩展操作符。然后，该组被作为前缀添加到语法中，如下所示：

```py
<start> ::= <group><option>* <arguments>
<group> ::= <empty>
```

并通过在组内调用 `add_argument()` 的下一个调用填充。

```py
class OptionGrammarMiner(OptionGrammarMiner):
    def add_group(self, locals, exclusive):
        kwargs = locals["kwargs"]
        if self.log:
            print(kwargs)

        required = kwargs.get("required", False)
        group = new_symbol(self.grammar, "<group>")

        if required and exclusive:
            group_expansion = group
        if required and not exclusive:
            group_expansion = group + "+"
        if not required and exclusive:
            group_expansion = group + "?"
        if not required and not exclusive:
            group_expansion = group + "*"

        self.grammar[START_SYMBOL][0] = group_expansion + \
            self.grammar[START_SYMBOL][0]
        self.grammar[group] = []
        self.current_group = group 
```

就这样！有了这个，我们现在可以从我们的 `process_numbers()` 程序中提取语法。再次打开日志可以揭示我们使用的变量。

```py
miner = OptionGrammarMiner(process_numbers, log=True)
process_numbers_grammar = miner.mine_ebnf_grammar() 
```

```py
('-h', '--help')
{'action': 'help', 'default': '==SUPPRESS==', 'help': 'show this help message and exit'}

('integers',)
{'metavar': 'N', 'type': <class 'int'>, 'nargs': '+', 'help': 'an integer for the accumulator'}

{'required': True}
('--sum',)
{'dest': 'accumulate', 'action': 'store_const', 'const': <built-in function sum>, 'help': 'sum the integers'}

('--min',)
{'dest': 'accumulate', 'action': 'store_const', 'const': <built-in function min>, 'help': 'compute the minimum'}

('--max',)
{'dest': 'accumulate', 'action': 'store_const', 'const': <built-in function max>, 'help': 'compute the maximum'}

```

这里是提取的语法：

```py
process_numbers_grammar 
```

```py
{'<start>': ['<group>(<option>)*<arguments>'],
 '<option>': [' -h', ' --help'],
 '<arguments>': ['( <integers>)+'],
 '<int>': ['(-)?<digit>+'],
 '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
 '<integers>': ['<int>'],
 '<group>': [' --sum', ' --min', ' --max']}

```

语法正确地识别了找到的组：

```py
process_numbers_grammar["<start>"] 
```

```py
['<group>(<option>)*<arguments>']

```

```py
process_numbers_grammar["<group>"] 
```

```py
[' --sum', ' --min', ' --max']

```

它还标识了一个由我们提供的 `argparse` 模块提供的 `--help` 选项：

```py
process_numbers_grammar["<option>"] 
```

```py
[' -h', ' --help']

```

语法也正确地识别了参数的类型：

```py
process_numbers_grammar["<arguments>"] 
```

```py
['( <integers>)+']

```

```py
process_numbers_grammar["<integers>"] 
```

```py
['<int>']

```

`int` 的规则设置为 `add_int_rule()` 所定义的。

```py
process_numbers_grammar["<int>"] 
```

```py
['(-)?<digit>+']

```

我们可以将这个语法转换为 BNF，这样我们就可以立即使用它进行模糊测试：

```py
assert is_valid_grammar(process_numbers_grammar) 
```

```py
grammar = convert_ebnf_grammar(process_numbers_grammar)
assert is_valid_grammar(grammar) 
```

```py
f = GrammarCoverageFuzzer(grammar)
for i in range(10):
    print(f.fuzz()) 
```

```py
 --sum 9
 --max -h --help --help -16 -0
 --min --help 2745341 8
 --min 1 27
 --sum --help --help -2
 --sum --help 0 3 -77
 --sum -3
 --sum --help 429 8 10 0295 -694 1
 --max -h 91 -1425 99
 --sum -795 -94 8 -44

```

每次调用都遵循在 `argparse` 调用中设定的规则。通过从现有程序中挖掘选项和参数，我们现在可以立即模糊这些选项——而无需指定语法。

## 测试 Autopep8

让我们尝试在现实世界的 Python 程序上使用选项语法挖掘器。`autopep8` 是一个将 Python 代码自动转换为 [Python 代码 PEP 8 风格指南](https://www.python.org/dev/peps/pep-0008/)的工具。（实际上，本书中的所有 Python 代码在生产过程中都通过 `autopep8` 运行。）`autopep8` 提供了广泛的功能，可以通过使用 `--help` 来查看：

```py
!autopep8  --help 
```

```py
usage: autopep8 [-h] [--version] [-v] [-d] [-i] [--global-config filename]
                [--ignore-local-config] [-r] [-j n] [-p n] [-a]
                [--experimental] [--exclude globs] [--list-fixes]
                [--ignore errors] [--select errors] [--max-line-length n]
                [--line-range line line] [--hang-closing] [--exit-code]
                [files ...]

Automatically formats Python code to conform to the PEP 8 style guide.

positional arguments:
  files                 files to format or '-' for standard in

options:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -v, --verbose         print verbose messages; multiple -v result in more
                        verbose messages
  -d, --diff            print the diff for the fixed source
  -i, --in-place        make changes to files in place
  --global-config filename
                        path to a global pep8 config file; if this file does
                        not exist then this is ignored (default:
                        /Users/zeller/.config/pep8)
  --ignore-local-config
                        don't look for and apply local config files; if not
                        passed, defaults are updated with any config files in
                        the project's root directory
  -r, --recursive       run recursively over directories; must be used with
                        --in-place or --diff
  -j n, --jobs n        number of parallel jobs; match CPU count if value is
                        less than 1
  -p n, --pep8-passes n
                        maximum number of additional pep8 passes (default:
                        infinite)
  -a, --aggressive      enable non-whitespace changes; multiple -a result in
                        more aggressive changes
  --experimental        enable experimental fixes
  --exclude globs       exclude file/directory names that match these comma-
                        separated globs
  --list-fixes          list codes for fixes; used by --ignore and --select
  --ignore errors       do not fix these errors/warnings (default:
                        E226,E24,W50,W690)
  --select errors       fix only these errors/warnings (e.g. E4,W)
  --max-line-length n   set maximum allowed line length (default: 79)
  --line-range line line, --range line line
                        only fix errors found within this inclusive range of
                        line numbers (e.g. 1 99); line numbers are indexed at
                        1
  --hang-closing        hang-closing option passed to pycodestyle
  --exit-code           change to behavior of exit code. default behavior of
                        return value, 0 is no differences, 1 is error exit.
                        return 2 when add this option. 2 is exists
                        differences.

```

### Autopep8 设置

我们想系统地测试这些选项。为了部署我们的配置语法挖掘器，我们需要找到可执行文件源代码：

```py
import [os](https://docs.python.org/3/library/os.html) 
```

```py
def find_executable(name):
    for path in os.get_exec_path():
        qualified_name = os.path.join(path, name)
        if os.path.exists(qualified_name):
            return qualified_name
    return None 
```

```py
autopep8_executable = find_executable("autopep8")
assert autopep8_executable is not None
autopep8_executable 
```

```py
'/Users/zeller/.pyenv/versions/3.10.2/bin/autopep8'

```

接下来，我们构建一个函数，该函数读取文件内容并执行它。

```py
def autopep8():
    executable = find_executable("autopep8")

    # First line has to contain "/usr/bin/env python" or like
    first_line = open(executable).readline()
    assert first_line.find("python") >= 0

    contents = open(executable).read()
    exec(contents) 
```

### 挖掘 Autopep8 语法

我们可以在我们的语法挖掘器中使用 `autopep8()` 函数：

```py
autopep8_miner = OptionGrammarMiner(autopep8) 
```

并从中提取语法：

```py
autopep8_ebnf_grammar = autopep8_miner.mine_ebnf_grammar() 
```

这之所以有效，是因为在这里，`autopep8` 不是一个单独的过程（也不是一个单独的 Python 解释器），我们是在当前的 Python 解释器中运行 `autopep8()` 函数（以及 `autopep8` 代码）——直到调用 `parse_args()`，在那里我们再次中断执行。此时，`autopep8` 代码除了设置参数解析器之外没有做任何事情——这正是我们所感兴趣的。

挖掘出的语法选项精确地反映了提供 `--help` 时看到的选项：

```py
print(autopep8_ebnf_grammar["<option>"]) 
```

```py
[' -h', ' --help', ' --version', ' -v', ' --verbose', ' -d', ' --diff', ' -i', ' --in-place', ' --global-config <filename>', ' --ignore-local-config', ' -r', ' --recursive', ' -j <n>', ' --jobs <n>', ' -p <n>', ' --pep8-passes <n>', ' -a', ' --aggressive', ' --experimental', ' --exclude <globs>', ' --list-fixes', ' --ignore <errors>', ' --select <errors>', ' --max-line-length <n>', ' --line-range <line> <line>', ' --range <line> <line>', ' --indent-size <int>', ' --hang-closing', ' --exit-code']

```

元变量如 `<n>` 或 `<line>` 是整数的占位符。我们假设所有同名元变量具有相同的类型：

```py
autopep8_ebnf_grammar["<line>"] 
```

```py
['<int>']

```

语法挖掘器推断出 `autopep8` 的参数是一系列文件：

```py
autopep8_ebnf_grammar["<arguments>"] 
```

```py
['( <files>)*']

```

这些选项反过来都是字符串：

```py
autopep8_ebnf_grammar["<files>"] 
```

```py
['<str>']

```

由于我们只对测试选项感兴趣，而不是参数，我们将参数固定为一个必填输入。 （否则，我们会生成大量的随机文件名。）

```py
autopep8_ebnf_grammar["<arguments>"] = [" <files>"]
autopep8_ebnf_grammar["<files>"] = ["foo.py"]
assert is_valid_grammar(autopep8_ebnf_grammar) 
```

### 创建 Autopep8 选项

现在，让我们使用推断出的语法进行模糊测试。同样，我们将 EBNF 语法转换为正则 BNF 语法：

```py
autopep8_grammar = convert_ebnf_grammar(autopep8_ebnf_grammar)
assert is_valid_grammar(autopep8_grammar) 
```

我们可以使用语法来模糊所有选项：

```py
f = GrammarCoverageFuzzer(autopep8_grammar, max_nonterminals=4)
for i in range(20):
    print(f.fuzz()) 
```

```py
 -r foo.py
 -h --experimental --hang-closing foo.py
 --list-fixes -v foo.py
 --aggressive -d foo.py
 --indent-size 9 --help foo.py
 --exit-code --recursive foo.py
 --diff --version -i foo.py
 --max-line-length 0 --in-place --verbose foo.py
 --ignore-local-config -a foo.py
 --select x -i --exit-code foo.py
 -j 8 --diff foo.py
 -d -v -d foo.py
 -p 6 -i foo.py
 -v --diff foo.py
 --ignore uA --recursive foo.py
 --jobs 5 -r foo.py
 --range 4 1 foo.py
 --ignore-local-config -i foo.py
 -r --exit-code foo.py
 -v -r foo.py

```

让我们将这些选项应用到实际程序上。我们需要一个名为 `foo.py` 的文件作为输入： （注意，以下命令将覆盖当前工作目录中已存在的 `foo.py` 文件。如果您下载了笔记本并在本地工作，请注意这一点。）

```py
def create_foo_py():
    open("foo.py", "w").write("""
def twice(x = 2):
 return  x  +  x
""") 
```

```py
create_foo_py() 
```

```py
print(open("foo.py").read(), end="") 
```

```py
def twice(x = 2):
    return  x  +  x

```

我们看到 `autopep8` 如何修复间距：

```py
!autopep8  foo.py 
```

```py
def twice(x=2):
    return x + x

```

现在，让我们将这些事情组合起来。我们定义一个 `ProgramRunner`，它将使用从挖掘的 `autopep8` 语法中获得的参数来运行 `autopep8` 可执行文件。

```py
from Fuzzer import ProgramRunner 
```

使用挖掘的选项运行 `autopep8` 显示了令人惊讶的高通过率。 （我们注意到一些选项相互依赖或互斥，但这由程序逻辑处理，而不是参数解析器，因此超出了我们的范围。）`GrammarCoverageFuzzer` 确保每个选项至少测试一次。（顺便说一句，数字和字母也是如此。）

```py
f = GrammarCoverageFuzzer(autopep8_grammar, max_nonterminals=5)
for i in range(20):
    invocation = "autopep8" + f.fuzz()
    print("$ " + invocation)
    args = invocation.split()
    autopep8_runner = ProgramRunner(args)
    result, outcome = autopep8_runner.run()
    if result.stderr != "":
        print(result.stderr, end="") 
```

```py
$ autopep8 foo.py
$ autopep8 --diff --max-line-length 4 --exit-code --range 5 8 -p 2 foo.py
$ autopep8 --ignore z --verbose -r --list-fixes foo.py
--recursive must be used with --in-place or --diff$ autopep8 --exclude 5 -h -i --aggressive --in-place foo.py
$ autopep8 --select a --help --experimental foo.py
$ autopep8 --indent-size -30 --recursive foo.py
--recursive must be used with --in-place or --diff$ autopep8 --global-config < -j 9 -v -a foo.py
parallel jobs requires --in-place$ autopep8 --line-range 7 1 --hang-closing -d foo.py
First value of --range should be less than or equal to the second$ autopep8 --pep8-passes 6 --hang-closing --version --ignore-local-config foo.py
$ autopep8 --jobs -2 --experimental --version foo.py
$ autopep8 --ignore Y: --select ! --global-config e foo.py
$ autopep8 --select 1 -a --recursive --aggressive foo.py
--recursive must be used with --in-place or --diff$ autopep8 --ignore * --ignore `0 --global-config _ --verbose foo.py
[file:foo.py]
--->  Applying global fix for E265
--->  5 issue(s) to fix {'E251': {2}, 'E271': {3}, 'E221': {3}, 'E222': {3}}
--->  3 issue(s) to fix {'E251': {2}, 'E221': {3}, 'E222': {3}}
--->  1 issue(s) to fix {'E222': {3}}
--->  0 issue(s) to fix {}
$ autopep8 --global-config ,\ --exclude r -v foo.py
[file:foo.py]
--->  Applying global fix for E265
--->  5 issue(s) to fix {'E251': {2}, 'E271': {3}, 'E221': {3}, 'E222': {3}}
--->  3 issue(s) to fix {'E251': {2}, 'E221': {3}, 'E222': {3}}
--->  1 issue(s) to fix {'E222': {3}}
--->  0 issue(s) to fix {}
$ autopep8 --global-config xd6M --recursive foo.py
--recursive must be used with --in-place or --diff$ autopep8 --select R --exclude L --version --ignore-local-config foo.py
$ autopep8 --select " --verbose -h -d foo.py
$ autopep8 --diff -i -h foo.py
$ autopep8 --in-place --select w --version -i foo.py
$ autopep8 --ignore 49 --exclude lI -i foo.py

```

我们的 `foo.py` 文件现在已经多次进行了格式化：

```py
print(open("foo.py").read(), end="") 
```

```py
def twice(x=2):
    return x + x

```

我们不再需要它了，所以我们需要清理一下：

```py
import [os](https://docs.python.org/3/library/os.html) 
```

```py
os.remove("foo.py") 
```

## 用于模糊配置选项的类

让我们现在创建可重用的类，我们可以用于测试任意程序。（好吧，让我们说“任意用 Python 编写并使用 `argparse` 模块处理命令行参数的程序。”）

类 `OptionRunner` 是 `ProgramRunner` 的子类，负责自动确定语法，使用与上面 `autopep8` 相同的步骤。

```py
from Grammars import unreachable_nonterminals 
```

```py
class OptionRunner(ProgramRunner):
  """Run a program while determining its option grammar"""

    def __init__(self, program: Union[str, List[str]],
                 arguments: Optional[str] = None, *,
                 log: bool = False,
                 miner_class: Optional[Type[OptionGrammarMiner]] = None):
  """Constructor.
 `program` - the (Python) program to be executed
 `arguments` - an (optional) string with arguments for `program`
 `log` - if True, enable logging in miner
 `miner_class` - the `OptionGrammarMiner` class to be used
 (default: `OptionGrammarMiner`)
 """
        if isinstance(program, str):
            self.base_executable = program
        else:
            self.base_executable = program[0]

        if miner_class is None:
            miner_class = OptionGrammarMiner
        self.miner_class = miner_class
        self.log = log

        self.find_contents()
        self.find_grammar()
        if arguments is not None:
            self.set_arguments(arguments)
        super().__init__(program) 
```

首先，我们找到 Python 可执行文件的内容：

```py
class OptionRunner(OptionRunner):
    def find_contents(self):
        self._executable = find_executable(self.base_executable)
        if self._executable is None:
            raise IOError(self.base_executable + ": not found")

        first_line = open(self._executable).readline()
        if first_line.find("python") < 0:
            raise IOError(self.base_executable + ": not a Python executable")

        self.contents = open(self._executable).read()

    def invoker(self):
        # We are passing the local variables as is, such that we can access `self`
        # We set __name__ to '__main__' to invoke the script as an executable
        exec(self.contents, {'__name__': '__main__'})

    def executable(self):
        return self._executable 
```

接下来，我们使用 `OptionGrammarMiner` 类确定语法：

```py
class OptionRunner(OptionRunner):
    def find_grammar(self):
        miner = self.miner_class(self.invoker, log=self.log)
        self._ebnf_grammar = miner.mine_ebnf_grammar()

    def ebnf_grammar(self):
  """Return extracted grammar in EBNF form"""
        return self._ebnf_grammar

    def grammar(self):
  """Return extracted grammar in BNF form"""
        return convert_ebnf_grammar(self._ebnf_grammar) 
```

两个服务方法 `set_arguments()` 和 `set_invocation()` 帮助我们更改参数和程序，分别。

```py
class OptionRunner(OptionRunner):
    def set_arguments(self, args):
        self._ebnf_grammar["<arguments>"] = [" " + args]
        # Delete rules for previous arguments
        for nonterminal in unreachable_nonterminals(self._ebnf_grammar):
            del self._ebnf_grammar[nonterminal]

    def set_invocation(self, program):
        self.program = program 
```

我们可以在 `autopep8` 上实例化该类并立即获取语法：

```py
autopep8_runner = OptionRunner("autopep8", "foo.py") 
```

```py
print(autopep8_runner.ebnf_grammar()["<option>"]) 
```

```py
[' -h', ' --help', ' --version', ' -v', ' --verbose', ' -d', ' --diff', ' -i', ' --in-place', ' --global-config <filename>', ' --ignore-local-config', ' -r', ' --recursive', ' -j <n>', ' --jobs <n>', ' -p <n>', ' --pep8-passes <n>', ' -a', ' --aggressive', ' --experimental', ' --exclude <globs>', ' --list-fixes', ' --ignore <errors>', ' --select <errors>', ' --max-line-length <n>', ' --line-range <line> <line>', ' --range <line> <line>', ' --indent-size <int>', ' --hang-closing', ' --exit-code']

```

`OptionFuzzer` 与给定的 `OptionRunner` 交互以获取其语法，然后将其传递给其 `GrammarCoverageFuzzer` 超类。

```py
class OptionFuzzer(GrammarCoverageFuzzer):
  """Fuzz a (Python) program using its arguments"""

    def __init__(self, runner: OptionRunner, *args, **kwargs):
  """Constructor. `runner` is an OptionRunner."""
        assert issubclass(type(runner), OptionRunner)
        self.runner = runner
        grammar = runner.grammar()
        super().__init__(grammar, *args, **kwargs) 
```

当调用 `run()` 时，`OptionFuzzer` 使用其语法的 `fuzz()` 创建一个新的调用，并使用语法中的参数运行现在给定（或之前设置的）运行器。请注意，`run()` 中指定的运行器可以与初始化期间设置的运行器不同；这允许从程序中挖掘选项并在另一个上下文中应用。

```py
class OptionFuzzer(OptionFuzzer):
    def run(self, runner=None, inp=""):
        if runner is None:
            runner = self.runner
        assert issubclass(type(runner), OptionRunner)
        invocation = runner.executable() + " " + self.fuzz()
        runner.set_invocation(invocation.split())
        return runner.run(inp) 
```

### 示例：Autopep8

让我们将新定义的类应用于 `autopep8` 运行器：

```py
autopep8_fuzzer = OptionFuzzer(autopep8_runner, max_nonterminals=5) 
```

```py
for i in range(3):
    print(autopep8_fuzzer.fuzz()) 
```

```py
 foo.py
 --in-place --ignore-local-config --jobs 6 --recursive -i foo.py
 --help -a --indent-size -95 --pep8-passes 3 --exclude = -r foo.py

```

我们现在可以系统地使用这些类测试 `autopep8`：

```py
autopep8_fuzzer.run(autopep8_runner) 
```

```py
(CompletedProcess(args=['/Users/zeller/.pyenv/versions/3.10.2/bin/autopep8', '--hang-closing', '--exit-code', '-d', '--version', 'foo.py'], returncode=0, stdout='autopep8 1.6.0 (pycodestyle: 2.12.1)\n', stderr=''),
 'PASS')

```

### 示例：MyPy

我们可以提取用于 Python 的 `mypy` 静态类型检查器的选项：

```py
assert find_executable("mypy") is not None 
```

```py
mypy_runner = OptionRunner("mypy", "foo.py")
print(mypy_runner.ebnf_grammar()["<option>"]) 
```

```py
[' -h', ' --help', ' -v', ' --verbose', ' -V', ' --version', ' -O <FORMAT>', ' --output <FORMAT>', ' --config-file <str>', ' --warn-unused-configs', ' --no-warn-unused-configs', ' --no-namespace-packages', ' --namespace-packages', ' --ignore-missing-imports', ' --follow-untyped-imports', ' --follow-imports <str>', ' --python-executable', ' --no-site-packages', ' --no-silence-site-packages', ' --python-version <x.y>', ' --platform', ' --always-true', ' --always-false', ' --disallow-any-expr', ' --disallow-any-decorated', ' --disallow-any-explicit', ' --disallow-any-generics', ' --allow-any-generics', ' --disallow-any-unimported', ' --allow-any-unimported', ' --disallow-subclassing-any', ' --allow-subclassing-any', ' --disallow-untyped-calls', ' --allow-untyped-calls', ' --untyped-calls-exclude', ' --disallow-untyped-defs', ' --allow-untyped-defs', ' --disallow-incomplete-defs', ' --allow-incomplete-defs', ' --check-untyped-defs', ' --no-check-untyped-defs', ' --disallow-untyped-decorators', ' --allow-untyped-decorators', ' --implicit-optional', ' --no-implicit-optional', ' --strict-optional', ' --no-strict-optional', ' --force-uppercase-builtins', ' --no-force-uppercase-builtins', ' --force-union-syntax', ' --no-force-union-syntax', ' --warn-redundant-casts', ' --no-warn-redundant-casts', ' --warn-unused-ignores', ' --no-warn-unused-ignores', ' --no-warn-no-return', ' --warn-no-return', ' --warn-return-any', ' --no-warn-return-any', ' --warn-unreachable', ' --no-warn-unreachable', ' --report-deprecated-as-note', ' --no-report-deprecated-as-note', ' --allow-untyped-globals', ' --disallow-untyped-globals', ' --allow-redefinition', ' --disallow-redefinition', ' --no-implicit-reexport', ' --implicit-reexport', ' --strict-equality', ' --no-strict-equality', ' --extra-checks', ' --no-extra-checks', ' --strict', ' --disable-error-code', ' --enable-error-code', ' --show-error-context', ' --hide-error-context', ' --show-column-numbers', ' --hide-column-numbers', ' --show-error-end', ' --hide-error-end', ' --hide-error-codes', ' --show-error-codes', ' --show-error-code-links', ' --hide-error-code-links', ' --pretty', ' --no-pretty', ' --no-color-output', ' --color-output', ' --no-error-summary', ' --error-summary', ' --show-absolute-path', ' --hide-absolute-path', ' --soft-error-limit <int>', ' -i', ' --incremental', ' --no-incremental', ' --cache-dir', ' --sqlite-cache', ' --no-sqlite-cache', ' --cache-fine-grained', ' --skip-version-check', ' --skip-cache-mtime-checks', ' --pdb', ' --show-traceback', ' --tb', ' --raise-exceptions', ' --custom-typing-module <MODULE>', ' --old-type-inference', ' --new-type-inference', ' --enable-incomplete-feature', ' --custom-typeshed-dir <DIR>', ' --warn-incomplete-stub', ' --no-warn-incomplete-stub', ' --shadow-file', ' --fast-exit', ' --no-fast-exit', ' --allow-empty-bodies', ' --disallow-empty-bodies', ' --export-ref-info', ' --any-exprs-report <DIR>', ' --cobertura-xml-report <DIR>', ' --html-report <DIR>', ' --linecount-report <DIR>', ' --linecoverage-report <DIR>', ' --lineprecision-report <DIR>', ' --txt-report <DIR>', ' --xml-report <DIR>', ' --xslt-html-report <DIR>', ' --xslt-txt-report <DIR>', ' --quickstart-file <str>', ' --junit-xml <str>', ' --junit-format <str>', ' --find-occurrences <CLASS.MEMBER>', ' --scripts-are-modules', ' --install-types', ' --no-install-types', ' --non-interactive', ' --interactive', ' --stats', ' --inferstats', ' --dump-build-stats', ' --timing-stats <str>', ' --line-checking-stats <str>', ' --debug-cache', ' --dump-deps', ' --dump-graph', ' --semantic-analysis-only', ' --test-env', ' --local-partial-types', ' --logical-deps', ' --bazel', ' --package-root', ' --cache-map( <str>)+', ' --debug-serialize', ' --disable-bytearray-promotion', ' --disable-memoryview-promotion', ' --strict-concatenate', ' --explicit-package-bases', ' --no-explicit-package-bases', ' --fast-module-lookup', ' --no-fast-module-lookup', ' --exclude', ' -m', ' --module', ' -p', ' --package', ' -c', ' --command']

```

```py
mypy_fuzzer = OptionFuzzer(mypy_runner, max_nonterminals=5)
for i in range(10):
    print(mypy_fuzzer.fuzz()) 
```

```py
 --namespace-packages foo.py
 --no-warn-no-return --enable-incomplete-feature -m --disallow-empty-bodies foo.py
 --dump-deps --disallow-subclassing-any --disallow-any-decorated --show-error-end --no-install-types --cache-fine-grained --sqlite-cache --no-warn-unreachable --no-warn-return-any --force-uppercase-builtins --debug-serialize --disable-bytearray-promotion --implicit-reexport --linecount-report l --fast-exit --allow-untyped-globals --no-color-output --color-output --show-error-code-links --inferstats --cache-map @ S --allow-any-generics --error-summary foo.py
 --allow-subclassing-any --no-incremental --fast-module-lookup --no-site-packages --old-type-inference --allow-redefinition --no-force-uppercase-builtins --incremental --new-type-inference foo.py
 --disallow-any-explicit --no-implicit-optional --no-warn-incomplete-stub foo.py
 --disallow-any-unimported --warn-unused-configs --hide-error-codes --allow-untyped-calls --follow-untyped-imports foo.py
 -h --no-report-deprecated-as-note --xslt-txt-report ze --command --dump-graph foo.py
 --find-occurrences 1 --strict-optional --hide-column-numbers --export-ref-info foo.py
 --soft-error-limit 06 --semantic-analysis-only foo.py
 --no-pretty --no-strict-equality --no-namespace-packages --tb --pdb --warn-redundant-casts --strict --install-types --disable-memoryview-promotion foo.py

```

### 示例：Notedown

这里是 `notedown` 笔记本到 Markdown 转换器的配置选项：

```py
assert find_executable("notedown") is not None 
```

```py
import [warnings](https://docs.python.org/3/library/warnings.html) 
```

```py
with warnings.catch_warnings():
    # Workaround: `notedown` can issue a `DeprecationWarning`
    warnings.filterwarnings("ignore", category=DeprecationWarning)
    notedown_runner = OptionRunner("notedown") 
```

```py
print(notedown_runner.ebnf_grammar()["<option>"]) 
```

```py
[' -h', ' --help', ' -o( <str>)?', ' --output( <str>)?', ' --from <str>', ' --to <str>', ' --run', ' --execute', ' --timeout <int>', ' --strip', ' --precode( <str>)+', ' --knit( <str>)?', ' --rmagic', ' --nomagic', ' --render', ' --template <str>', ' --match <str>', ' --examples', ' --version', ' --debug']

```

```py
notedown_fuzzer = OptionFuzzer(notedown_runner, max_nonterminals=5)
for i in range(10):
    print(notedown_fuzzer.fuzz()) 
```

```py
 --version O[
 --help --from tF --match C --execute
 --timeout -50 --template ! --debug --examples --rmagic --output M4kI --strip --render mQ
 --run -h --nomagic c
 --to eW --render '
 -o --knit ^%J --precode } --render --execute --render $
 --precode . s -h --run \
 --nomagic -h --execute #
 --execute --run ai
 --output --knit --run --version --execute A

```

## 组合测试

我们的 `CoverageGrammarFuzzer` 在至少覆盖每个选项一次方面做得很好，这对于系统测试来说很棒。然而，正如我们上面的例子所看到的，一些选项需要彼此，而其他选项则相互干扰。作为优秀的测试人员，我们不仅要单独覆盖每个选项，还要覆盖选项的组合。

Python 的 `itertools` 模块为我们提供了从列表中创建组合的方法。例如，我们可以获取 `notedown` 选项并创建所有对的列表。

```py
from [itertools](https://docs.python.org/3/library/itertools.html) import combinations 
```

```py
option_list = notedown_runner.ebnf_grammar()["<option>"]
pairs = list(combinations(option_list, 2)) 
```

有很多对：

```py
len(pairs) 
```

```py
190

```

```py
print(pairs[:20]) 
```

```py
[(' -h', ' --help'), (' -h', ' -o( <str>)?'), (' -h', ' --output( <str>)?'), (' -h', ' --from <str>'), (' -h', ' --to <str>'), (' -h', ' --run'), (' -h', ' --execute'), (' -h', ' --timeout <int>'), (' -h', ' --strip'), (' -h', ' --precode( <str>)+'), (' -h', ' --knit( <str>)?'), (' -h', ' --rmagic'), (' -h', ' --nomagic'), (' -h', ' --render'), (' -h', ' --template <str>'), (' -h', ' --match <str>'), (' -h', ' --examples'), (' -h', ' --version'), (' -h', ' --debug'), (' --help', ' -o( <str>)?')]

```

经常测试每一对这样的选项就足以覆盖所有选项之间的干扰。（程序很少涉及三个或更多配置设置的条件。）为此，我们将语法从具有选项列表更改为具有选项对列表，这样覆盖这些选项将自动覆盖所有对。

我们创建了一个名为 `pairwise()` 的函数，它接受语法中出现的选项列表，并返回一个包含成对选项的列表——也就是说，我们的原始选项，但已连接。

```py
def pairwise(option_list):
    return [option_1 + option_2
            for (option_1, option_2) in combinations(option_list, 2)] 
```

这里是前 20 对：

```py
print(pairwise(option_list)[:20]) 
```

```py
[' -h --help', ' -h -o( <str>)?', ' -h --output( <str>)?', ' -h --from <str>', ' -h --to <str>', ' -h --run', ' -h --execute', ' -h --timeout <int>', ' -h --strip', ' -h --precode( <str>)+', ' -h --knit( <str>)?', ' -h --rmagic', ' -h --nomagic', ' -h --render', ' -h --template <str>', ' -h --match <str>', ' -h --examples', ' -h --version', ' -h --debug', ' --help -o( <str>)?']

```

新的语法`pairwise_notedown_grammar`是`notedown`语法的副本，但将选项列表替换为上述成对选项列表。

```py
notedown_grammar = notedown_runner.grammar()
pairwise_notedown_grammar = extend_grammar(notedown_grammar)
pairwise_notedown_grammar["<option>"] = pairwise(notedown_grammar["<option>"])
assert is_valid_grammar(pairwise_notedown_grammar) 
```

使用“成对”语法进行模糊测试现在可以一对一对地覆盖：

```py
notedown_pairwise_fuzzer = GrammarCoverageFuzzer(
    pairwise_notedown_grammar, max_nonterminals=4) 
```

```py
for i in range(10):
    print(notedown_pairwise_fuzzer.fuzz()) 
```

```py
 --nomagic --debug
 -h --help --examples --version l
 --execute --examples --run --execute
 --execute --debug --render --debug
 --help --render --strip --version
 -h --debug _U
 --execute --render --render --version Q
 --execute --rmagic -h --run
 --rmagic --version --help --execute d
 --strip --render --examples --debug ?

```

我们实际上能测试所有选项的组合吗？实际上不行，因为随着长度的增加，组合的数量会迅速增长。当选项数量达到最大值时（有 20 个选项时，只有 1 个涉及*所有*选项的组合），它会再次减少，但绝对数字仍然令人震惊：

```py
for combination_length in range(1, 20):
    tuples = list(combinations(option_list, combination_length))
    print(combination_length, len(tuples)) 
```

```py
1 20
2 190
3 1140
4 4845
5 15504
6 38760
7 77520
8 125970
9 167960
10 184756
11 167960
12 125970
13 77520
14 38760
15 15504
16 4845
17 1140
18 190
19 20

```

形式上，在长度为$n$的选项集中，长度为$k$的组合数是二项式系数 $$ {n \choose k} = \frac{n!}{k!(n - k)!} $$

对于$k = 2$（所有配对）给出的是

$$ {n \choose 2} = \frac{n!}{2(n - 2)!} = \frac{n (n - 1)}{2} $$

对于具有 30 个选项的`autopep8`...

```py
len(autopep8_runner.ebnf_grammar()["<option>"]) 
```

```py
30

```

...因此，我们需要 870 个测试来覆盖所有配对：

```py
len(autopep8_runner.ebnf_grammar()["<option>"]) * \
    (len(autopep8_runner.ebnf_grammar()["<option>"]) - 1) 
```

```py
870

```

对于具有 140 多个选项的`mypy`，我们最终需要进行 20,000 多个测试：

```py
len(mypy_runner.ebnf_grammar()["<option>"]) 
```

```py
170

```

```py
len(mypy_runner.ebnf_grammar()["<option>"]) * \
    (len(mypy_runner.ebnf_grammar()["<option>"]) - 1) 
```

```py
28730

```

即使每一对运行需要一秒钟，我们测试完成也需要三个小时。

如果你的程序有更多选项，你希望它们在组合中都被覆盖，建议进一步限制配置的数量——例如，通过将组合测试限制到可能相互交互的组合；并单独覆盖所有其他（可能是正交的）选项。

通过扩展语法创建配置的这种机制可以很容易地扩展到其他配置目标。可能想要探索更多的配置，或在特定上下文中的扩展。下面的练习为你准备了一些选项。

## 经验教训

+   除了常规输入数据外，程序*配置*也是一个重要的测试目标。

+   对于使用标准库解析命令行选项和参数的给定程序，可以自动提取这些并将其转换为语法。

+   要覆盖不仅单个选项，还包括选项的组合，可以扩展语法以覆盖所有配对，或者提出更加雄心勃勃的目标。

## 下一步

如果你喜欢从程序中挖掘语法的想法，不要错过：

+   如何从输入数据中挖掘语法

我们在书中的下一步将专注于：

+   如何解析和重新组合输入

+   如何为特定生成项分配权重和概率

+   如何简化导致失败的输入

## 背景

尽管配置数据与其他输入数据一样可能导致故障，但在测试生成中它却得到了相对较少的关注——可能是因为，与“常规”输入数据不同，配置数据并不那么受外部方的控制，而且，与常规数据不同，配置的变异性很小。为软件配置创建模型并使用这些模型进行测试是常见的，对偶测试的想法也是如此。有关概述，请参阅[[Pezzè 等人，2008](http://ix.cs.uoregon.edu/~michal/book/)]；有关对最先进技术的讨论和比较，请参阅[[J. Petke 等人，2015](https://doi.org/10.1109/TSE.2015.2421279)]。

更具体地说，[[Sutton 等人，2007](http://www.fuzzing.org/)]还讨论了系统性地覆盖命令行选项的技术。Dai 等人[[Dai 等人，2010](https://doi.org/10.4018/jsse.2010070103)]通过更改与配置文件相关的变量来应用配置模糊测试。

## 练习

### 练习 1：#ifdef 配置模糊测试

在 C 程序中，可以使用*C 预处理器*来选择哪些代码部分应该被编译，哪些不应该被编译。例如，在 C 代码

```py
#ifdef LONG_FOO
long  foo()  {  ...  }
#else
int  foo()  {  ...  }
#endif 
```

如果定义了预处理器变量`LONG_FOO`，编译器将以`long`类型编译函数`foo()`，如果没有定义，则以`int`类型编译。此类预处理器变量要么在源文件中设置（使用`#define`，如`#define LONG_FOO`），要么在 C 编译器命令行上设置（使用`-D<variable>`或`-D<variable>=<value>`，如`-DLONG_FOO`）。

这种*条件编译*用于将 C 程序配置到其环境中。特定于系统的代码可以包含大量的条件编译。例如，考虑`xmlparse.c`的摘录，它是 Python 运行时库的一部分的 XML 解析器：

```py
#if defined(_WIN32) && !defined(LOAD_LIBRARY_SEARCH_SYSTEM32)
# define LOAD_LIBRARY_SEARCH_SYSTEM32  0x00000800
#endif

#if !defined(HAVE_GETRANDOM) && !defined(HAVE_SYSCALL_GETRANDOM) \
 && !defined(HAVE_ARC4RANDOM_BUF) && !defined(HAVE_ARC4RANDOM) \
 && !defined(XML_DEV_URANDOM) \
 && !defined(_WIN32) \
 && !defined(XML_POOR_ENTROPY)
# error
#endif

#if !defined(TIOCSWINSZ) || defined(__SCO__) || defined(__UNIXWARE__)
#define USE_SYSV_ENVVARS /* COLUMNS/LINES vs. TERMCAP */
#endif

#ifdef XML_UNICODE_WCHAR_T
#define XML_T(x) (const wchar_t)x
#define XML_L(x) L ## x
#else
#define XML_T(x) (const unsigned short)x
#define XML_L(x) x
#endif

int  fun(int  x)  {  return  XML_T(x);  } 
```

对于上述代码中的 C 预处理器的一个典型配置可能是`cc -c -D_WIN32 -DXML_POOR_ENTROPY -DXML_UNICODE_WCHAR_T xmlparse.c`，定义了给定的预处理器变量并选择了适当的代码片段。

由于编译器一次只能编译一个配置（这意味着我们一次也只能测试一个生成的可执行文件），因此你的任务是找出这些配置中哪些实际上可以编译。为此，分三步进行。

#### 第一部分：提取预处理器变量

编写一个`cpp_identifiers()`函数，该函数给定一组行（例如，从`open(filename).readlines()`读取），提取在`#if`或`#ifdef`预处理器指令中引用的所有预处理器变量。在上述示例 C 输入上应用`ifdef_identifiers()`，以便

```py
cpp_identifiers(open("xmlparse.c").readlines()) 
```

返回集合

```py
{'_WIN32', 'LOAD_LIBRARY_SEARCH_SYSTEM32', 'HAVE_GETRANDOM', 'HAVE_SYSCALL_GETRANDOM', 'HAVE_ARC4RANDOM_BUF', ...} 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises)进行练习并查看解决方案。

#### 第二部分：推导选项语法

通过`cpp_identifiers()`的帮助，创建一个语法，该语法具有带有选项列表的 C 编译器调用，其中每个选项的形式为`-D<变量>`，用于预处理器变量`<变量>`。使用此语法`cpp_grammar`，一个模糊测试器

```py
g = GrammarCoverageFuzzer(cpp_grammar) 
```

将创建 C 编译器调用，例如

```py
[g.fuzz() for i in range(10)]
['cc -DHAVE_SYSCALL_GETRANDOM xmlparse.c',
 'cc -D__SCO__ -DRANDOM_BUF -DXML_UNICODE_WCHAR_T -D__UNIXWARE__ xmlparse.c',
 'cc -DXML_POOR_ENTROPY xmlparse.c',
 'cc -DRANDOM xmlparse.c',
 'cc -D_WIN xmlparse.c',
 'cc -DHAVE_ARC xmlparse.c', ...] 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises)进行练习并查看解决方案。

#### 第三部分：C 预处理器配置模糊测试

使用刚刚生成的语法，使用`GrammarCoverageFuzzer`进行模糊测试

1.  分别测试每个处理器变量

1.  使用`pairwise()`分别测试处理器变量的每一对。

如果您实际运行这些调用会发生什么？

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises)进行练习并查看解决方案。

```py
os.remove("xmlparse.c") 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises)进行练习并查看解决方案。

```py
if os.path.exists("xmlparse.o"):
    os.remove("xmlparse.o") 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises)进行练习并查看解决方案。

### 练习 2：.ini 配置模糊测试

除了命令行选项之外，*配置文件*也是配置的重要来源之一。在这个练习中，我们将考虑 Python `ConfigParser`模块提供的非常简单的配置语言，它与 Microsoft Windows *.ini*文件中找到的内容非常相似。

以下`ConfigParser`输入文件的示例直接来自[ConfigParser 文档](https://docs.python.org/3/library/configparser.html)：

```py
[DEFAULT]
ServerAliveInterval = 45
Compression = yes
CompressionLevel = 9
ForwardX11 = yes

[bitbucket.org]
User = hg

[topsecret.server.com]
Port = 50022
ForwardX11 = no
```

上面的`ConfigParser`文件可以编程创建：

```py
import [configparser](https://docs.python.org/3/library/configparser.html) 
```

```py
config = configparser.ConfigParser()
config['DEFAULT'] = {'ServerAliveInterval': '45',
                     'Compression': 'yes',
                     'CompressionLevel': '9'}
config['bitbucket.org'] = {}
config['bitbucket.org']['User'] = 'hg'
config['topsecret.server.com'] = {}
topsecret = config['topsecret.server.com']
topsecret['Port'] = '50022'     # mutates the parser
topsecret['ForwardX11'] = 'no'  # same here
config['DEFAULT']['ForwardX11'] = 'yes'
with open('example.ini', 'w') as configfile:
    config.write(configfile)

with open('example.ini') as configfile:
    print(configfile.read(), end="") 
```

```py
[DEFAULT]
serveraliveinterval = 45
compression = yes
compressionlevel = 9
forwardx11 = yes

[bitbucket.org]
user = hg

[topsecret.server.com]
port = 50022
forwardx11 = no

```

并再次读取：

```py
config = configparser.ConfigParser()
config.read('example.ini')
topsecret = config['topsecret.server.com']
topsecret['Port'] 
```

```py
'50022'

```

#### 第一部分：读取配置

使用`configparser`创建一个程序，读取上述配置文件并访问单个元素。

#### 第二部分：创建配置语法

设计一个语法，该语法将自动创建适合您上述程序的配置文件。用它来模糊测试您的程序。

#### 第三部分：挖掘配置语法

通过动态跟踪对配置元素的单独访问，您可以从执行中再次提取一个基本语法。为此，创建一个具有特殊方法`__getitem__`的`ConfigParser`子类：

```py
class TrackingConfigParser(configparser.ConfigParser):
    def __getitem__(self, key):
        print("Accessing", repr(key))
        return super().__getitem__(key) 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises)进行练习并查看解决方案。

对于`TrackingConfigParser`对象`p`，`p.__getitem__(key)`将在访问`p[key]`时被调用：

```py
tracking_config_parser = TrackingConfigParser()
tracking_config_parser.read('example.ini')
section = tracking_config_parser['topsecret.server.com'] 
```

```py
Accessing 'topsecret.server.com'

```

使用`__getitem__()`，如上所述，实现一个跟踪机制，在您的程序访问读取的配置时，自动保存访问的选项和读取的值。从这些值创建一个原型语法；用于模糊测试。

最后，别忘了清理：

```py
import [os](https://docs.python.org/3/library/os.html) 
```

```py
os.remove("example.ini") 
```

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 3：提取和模糊测试 C 命令行选项

在 C 程序中，`getopt()` 函数经常用于处理配置选项。一个调用

```py
getopt(argc, argv, "bf:")
```

表示程序接受两个选项 `-b` 和 `-f`，其中 `-f` 接受一个参数（如下面的冒号所示）。

#### 第一部分：Getopt 模糊测试

编写一个框架，该框架对于给定的 C 程序，可以自动提取 `getopt()` 的参数并为其推导出一个模糊测试语法。实现这一目标有多种方法：

1.  在程序源代码中扫描 `getopt()` 的出现，并返回传递的字符串。（粗略，但应该经常有效。）

1.  将自己的 `getopt()` 实现插入到源代码中（实际上替换了运行时库中的 `getopt()`），该实现输出 `getopt()` 参数并退出程序。重新编译并运行。

1.  （高级。）与上述方法相同，但不是更改源代码，而是挂钩到 *动态链接器*，该链接器在运行时将程序与 C 运行时库链接。设置库加载路径（在 Linux 和 Unix 上，这是 `LD_LIBRARY_PATH` 环境变量），以便首先链接自己的 `getopt()` 版本，然后是常规库。执行程序（无需重新编译）应产生所需的结果。

将此应用于 `grep` 和 `ls`；报告生成的语法和结果。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

#### 第二部分：模糊测试 C 中的长选项

与第一部分相同，但还挂钩到 GNU 变体 `getopt_long()`，它接受带有双短横线的“长”参数，例如 `--help`。请注意，上述方法 1 在这里将不起作用，因为“长”选项是在单独定义的结构中定义的。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises) 来完成练习并查看解决方案。

### 练习 4：上下文中的扩展

在我们上述选项配置中，我们有许多符号都扩展到相同的整数。例如，`autopep8` 的 `--line-range` 选项接受两个 `<line>` 参数，这两个参数都扩展到相同的 `<int>` 符号：

```py
<option> ::= ... | --line-range <line> <line> | ...
<line> ::= <int>
<int> ::= (-)?<digit>+
<digit> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

```py
autopep8_runner.ebnf_grammar()["<line>"] 
```

```py
['<int>']

```

```py
autopep8_runner.ebnf_grammar()["<int>"] 
```

```py
['(-)?<digit>+']

```

```py
autopep8_runner.ebnf_grammar()["<digit>"] 
```

```py
['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']

```

然而，一旦 `GrammarCoverageFuzzer` 覆盖了 `<int>` 的所有变体（特别是通过覆盖所有数字）对于 *一个* 选项，它将不再努力为下一个选项实现此类覆盖。尽管如此，可能希望为每个选项单独实现此类覆盖。

使用我们现有的 `GrammarCoverageFuzzer` 实现这一点的另一种方法是相应地更改语法。想法是*复制*扩展——也就是说，用一个新符号 $s'$ 的定义来替换符号 $s$ 的扩展，这个定义是从 $s$ 复制的。这样，从覆盖的角度来看，$s'$ 和 $s$ 是独立的符号，并且将独立覆盖。

例如，再次考虑上述 `--line-range` 选项。如果我们希望我们的测试独立覆盖两个 `<line>` 参数的所有元素，我们可以将第二个 `<line>` 扩展复制到一个新的符号 `<line'>` 中，并随后的复制扩展：

```py
<option> ::= ... | --line-range <line> <line'> | ...
<line> ::= <int>
<line'> ::= <int'>
<int> ::= (-)?<digit>+
<int'> ::= (-)?<digit'>+
<digit> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
<digit'> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

设计一个函数 `inline(grammar, symbol)`，它返回一个 `grammar` 的副本，其中 `<symbol>` 及其扩展的每个出现都成为单独的副本。上述语法可能是 `inline(autopep8_runner.ebnf_grammar(), "<line>")` 的结果。

在复制时，复制中的扩展也应参考复制中的符号。因此，当在

`<int> ::= <int><digit>`

make that

```py <int>::=</int>

<int'> ::= <int'><digit'> ```

(而不是 `<int'> ::= <int><digit'>` 或 `<int'> ::= <int><digit>`).

确保为原始语法中的每个出现精确地添加一组新符号，并且不要在递归存在的情况下进一步扩展。

[使用笔记本](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs%2Fnotebooks/ConfigurationFuzzer.ipynb#Exercises) 中展开 `<int>` 来进行练习并查看解决方案。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后更改：2023-11-11 18:18:05+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/ConfigurationFuzzer.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[测试配置](https://www.fuzzingbook.org/html/ConfigurationFuzzer.html)"。在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 的 "[模糊测试书籍](https://www.fuzzingbook.org/)" 中，[`www.fuzzingbook.org/html/ConfigurationFuzzer.html`](https://www.fuzzingbook.org/html/ConfigurationFuzzer.html)。检索日期：2023-11-11 18:18:05+01:00。

```py
@incollection{fuzzingbook2023:ConfigurationFuzzer,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Testing Configurations},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/ConfigurationFuzzer.html}},
    note = {Retrieved 2023-11-11 18:18:05+01:00},
    url = {https://www.fuzzingbook.org/html/ConfigurationFuzzer.html},
    urldate = {2023-11-11 18:18:05+01:00}
}

```
