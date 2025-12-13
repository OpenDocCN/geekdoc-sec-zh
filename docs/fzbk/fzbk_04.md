# 书籍之旅

> [模糊测试书](http://www.fuzzingbook.org/html/Tours.html)

这本书非常庞大。拥有超过 20,000 行代码和 150,000 字的文本，印刷版将覆盖超过 1,200 页的文本。显然，我们不假设每个人都想阅读所有内容。

虽然这本书的章节可以依次阅读，但有许多可能的阅读路径。在这个图中，箭头 $A \rightarrow B$ 表示章节 $A$ 是章节 $B$ 的先决条件。你可以选择任意路径来获取你最感兴趣的主题：

<svg width="1482pt" height="622pt" viewBox="0.00 0.00 1481.88 622.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 618)"><g id="node1" class="node"><title>Fuzzer</title> <g id="a_node1"><a xlink:href="Fuzzer.html" xlink:title="Fuzzing: Breaking Things with Random Inputs (Fuzzer)

在本章中，我们将从最简单的测试生成技术开始。随机文本生成，也称为模糊测试，的关键思想是将一串随机字符输入到程序中，希望揭露故障。《模糊测试：破坏事物的方法》<text text-anchor="middle" x="910.62" y="-516.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Fuzzing: Breaking</text> <text text-anchor="middle" x="910.62" y="-498.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Things</text> <text text-anchor="middle" x="910.62" y="-480.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">with Random Inputs</text></a></g></g> <g id="node2" class="node"><title>Coverage</title> <g id="a_node2"><a xlink:href="Coverage.html" xlink:title="Code Coverage (Coverage)

在上一章中，我们介绍了基本的模糊测试——即生成随机输入来测试程序。如何衡量这些测试的有效性？一种方法就是检查找到的（数量和严重性）错误；但如果错误很少，我们需要一个代理来衡量测试揭露错误的概率。在本章中，我们引入了代码覆盖的概念，测量在测试运行期间程序的实际执行部分。测量这种覆盖率对于试图覆盖尽可能多代码的测试生成器来说也非常关键。《代码覆盖率》<text text-anchor="middle" x="551.62" y="-329.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Code Coverage</text></a></g></g> <g id="edge1" class="edge"><title>Fuzzer->Coverage</title></g> <g id="node3" class="node"><title>SearchBasedFuzzer</title> <g id="a_node3"><a xlink:href="SearchBasedFuzzer.html" xlink:title="Search-Based Fuzzing (SearchBasedFuzzer)

有时我们不仅对模糊测试尽可能多的不同程序输入感兴趣，还希望推导出特定的测试输入，以实现某些目标，例如到达程序中的特定语句。当我们对我们要找的东西有一个想法时，我们就可以去寻找它。搜索算法是计算机科学的核心，但将经典搜索算法如广度优先搜索或深度优先搜索应用于测试搜索是不切实际的，因为这些算法可能需要我们查看所有可能的输入。然而，领域知识可以用来克服这个问题。例如，如果我们能估计几个程序输入中哪一个更接近我们正在寻找的，那么这些信息可以引导我们更快地达到目标——这种信息被称为启发式。启发式系统地应用的方式被元启发式搜索算法所捕捉。这里的“元”表示这些算法是通用的，并且可以根据不同的问题实例化。元启发式通常从自然界观察到的过程中获得灵感。例如，有一些算法模仿进化过程、群体智能或化学反应。总的来说，它们比穷举搜索方法更有效率，因此可以应用于广阔的搜索空间——对于它们来说，程序输入领域的广阔搜索空间不是问题。《基于搜索的模糊测试》（<text text-anchor="middle" x="768.62" y="-409.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Search-Based Fuzzing</text>）</a></g></g> <g id="edge2" class="edge"><title>Fuzzer->SearchBasedFuzzer</title></g> <g id="node4" class="node"><title>Grammars</title> <g id="a_node4"><a xlink:href="Grammars.html" xlink:title="Fuzzing with Grammars (Grammars)

在“基于变异的模糊测试”章节中，我们看到了如何使用额外的提示——例如样本输入文件——来加速测试生成。在本章中，我们将这一想法进一步发展，通过提供程序合法输入的规范。通过语法指定输入可以实现非常系统和高效的测试生成，尤其是在复杂的输入格式中。语法还作为配置模糊测试、API 模糊测试、GUI 模糊测试以及更多测试的基础。《使用语法进行模糊测试》（<text text-anchor="middle" x="910.62" y="-418.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Fuzzing with</text> <text text-anchor="middle" x="910.62" y="-400.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Grammars</text>）</a></g></g> <g id="edge3" class="edge"><title>Fuzzer->Grammars</title></g> <g id="node5" class="node"><title>SymbolicFuzzer</title> <g id="a_node5"><a xlink:href="SymbolicFuzzer.html" xlink:title="Symbolic Fuzzing (SymbolicFuzzer)

传统模糊测试方法的一个问题是，它们无法测试系统可能具有的所有可能行为，尤其是在输入空间很大时。很多时候，特定执行分支的执行可能只发生在非常特定的输入下，这可能只占输入空间的一小部分。传统的模糊测试方法依赖于偶然来产生所需的输入。然而，当要探索的空间很大时，依赖于随机生成我们想要的值是一个坏主意。例如，一个接受字符串的函数，即使只考虑前 10 个字符，也有$2^{80}$种可能的输入。如果寻找特定的字符串，即使在超级计算机上，随机生成值也需要几千年。《符号模糊测试</a></g></g> <g id="edge4" class="edge"><title>Fuzzer->SymbolicFuzzer</title></g> <g id="node6" class="node"><title>大规模模糊测试</title> <g id="a_node6"><a xlink:href="FuzzingInTheLarge.html" xlink:title="大规模模糊测试（FuzzingInTheLarge）

在前面的章节中，我们总是只关注几秒钟内在一台机器上进行的模糊测试。然而，在现实世界中，模糊测试通常在数十台甚至数千台机器上运行；持续数小时、数天甚至数周；针对一个程序或数十个程序。在这种情况下，需要一个基础设施来收集单个模糊测试运行中的失败数据，并将这些数据汇总到一个中央存储库中。在本章中，我们将探讨这样一个基础设施，即来自 Mozilla 的 FuzzManager 框架。《大规模模糊测试</a></g></g> <g id="edge5" class="edge"><title>Fuzzer->FuzzingInTheLarge</title></g> <g id="node8" class="node"><title>变异模糊测试器</title> <g id="a_node8"><a xlink:href="MutationFuzzer.html" xlink:title="基于变异的模糊测试（变异模糊测试器）

大多数随机生成的输入在语法上是无效的，因此很快就会被处理程序拒绝。为了在输入处理之外测试功能，我们必须增加获得有效输入的机会。其中一种方法就是所谓的突变模糊测试——即对现有输入进行微小修改，这些修改可能仍然保持输入的有效性，但会测试新的行为。我们展示了如何创建这样的突变，以及如何引导它们向尚未发现的代码，应用了流行的 AFL 模糊器中的核心概念。《基于突变的模糊测试》（Mutation-Based Fuzzing）</a></g></g> <g id="edge7" class="edge"><title>Coverage->MutationFuzzer</title></g> <g id="node9" class="node"><title>MutationAnalysis</title> <g id="a_node9"><a xlink:href="MutationAnalysis.html" xlink:title="突变分析（MutationAnalysis）

在关于覆盖率的一章中，我们展示了如何识别程序中哪些部分被执行，从而对一组测试用例覆盖程序结构的有效性有一个感性的认识。然而，覆盖率本身可能并不是衡量测试有效性的最佳指标，因为即使没有检查结果是否正确，也可能有很高的覆盖率。在这一章中，我们介绍了一种评估测试套件有效性的另一种方法：在代码中注入突变——人工错误后，我们检查测试套件是否能够检测到这些人工错误。其理念是，如果它未能检测到这样的突变，它也可能错过真实的错误。《突变分析》（Mutation Analysis）</a></g></g> <g id="edge8" class="edge"><title>Coverage->MutationAnalysis</title></g> <g id="node10" class="node"><title>GrammarCoverageFuzzer</title> <g id="a_node10"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="语法覆盖率（GrammarCoverageFuzzer）

从语法中生成输入使得一个规则的每个可能的扩展都有相同的可能性。然而，为了生成一个全面的测试套件，最大化多样性更有意义——例如，通过避免重复相同的扩展。在本章中，我们探讨了如何系统地覆盖语法的元素，以最大化多样性并确保不遗漏任何单个元素。《语法覆盖率》（Grammar Coverage）</text> <a xlink:href="GrammarCoverageFuzzer.html" xlink:title="Grammar Coverage Fuzzer"><text text-anchor="middle" x="1039.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">语法覆盖率</text></a></g></g> <g id="edge9" class="edge"><title>覆盖率->语法覆盖率模糊测试器</title></g> <g id="node11" class="node"><title>ProbabilisticGrammarFuzzer</title> <g id="a_node11"><a xlink:href="ProbabilisticGrammarFuzzer.html" xlink:title="概率语法模糊测试 (ProbabilisticGrammarFuzzer)

让我们通过为单个扩展分配概率来赋予语法更多的能力。这使我们能够控制每种元素应该产生多少，从而允许我们将生成的测试针对特定的功能。我们还展示了如何从给定的样本输入中学习这样的概率，并特别将我们的测试针对这些样本中不常见的输入特征。《概率语法模糊测试》（Probabilistic Grammar Fuzzing）</text> <text text-anchor="middle" x="409.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">概率语法模糊测试</text></a></g></g> <g id="edge10" class="edge"><title>覆盖率->概率语法模糊测试器</title></g> <g id="node12" class="node"><title>ConcolicFuzzer</title> <g id="a_node12"><a xlink:href="ConcolicFuzzer.html" xlink:title="Concolic Fuzzing (ConcolicFuzzer)

在信息流章节中，我们看到了如何使用动态污点来生成比仅仅寻找程序崩溃更智能的测试用例。我们还看到了如何使用污点来更新语法，从而更专注于危险的方法。《动态不变量》（DynamicInvariants）</text> <a xlink:href="DynamicInvariants.html" xlink:title="挖掘函数规范 (DynamicInvariants)"><text text-anchor="middle" x="892.62" y="-89.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">动态不变量</text></a></g></g> <g id="edge11" class="edge"><title>覆盖率->ConcolicFuzzer</title></g> <g id="node13" class="node"><title>DynamicInvariants</title> <g id="a_node13"><a xlink:href="DynamicInvariants.html" xlink:title="挖掘函数规范 (DynamicInvariants)"><text text-anchor="middle" x="1039.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">动态不变量</text></a></g></g>

在测试一个程序时，不仅需要覆盖其多种行为；还需要检查结果是否符合预期。&nbsp;在本章中，我们介绍了一种技术，使我们能够从一组给定的执行中挖掘函数规范，从而得到函数期望和提供的内容的抽象和形式描述。"><text text-anchor="middle" x="415.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">挖掘函数</text> <text text-anchor="middle" x="415.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">规范</text></a></g></g> <g id="edge12" class="edge"><title>覆盖率->动态不变量</title></g> <g id="node14" class="node"><title>PythonFuzzer</title> <g id="a_node14"><a xlink:href="PythonFuzzer.html" xlink:title="测试编译器 (PythonFuzzer)

在本章中，我们将利用语法和基于语法的测试来系统地生成程序代码——&nbsp;例如，测试编译器或解释器。不出所料，我们使用 Python 和 Python 解释器作为我们的领域。"><text text-anchor="middle" x="729.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">测试编译器</text></a></g></g> <g id="edge13" class="edge"><title>覆盖率->PythonFuzzer</title></g> <g id="node15" class="node"><title>WhenToStopFuzzing</title> <g id="a_node15"><a xlink:href="WhenToStopFuzzing.html" xlink:title="何时停止模糊测试 (WhenToStopFuzzing)

在过去的章节中，我们讨论了几种模糊测试技术。&nbsp;知道该做什么很重要，但知道何时停止做某事也同样重要。&nbsp;在本章中，我们将学习何时停止模糊测试——&nbsp;并使用一个突出的例子来说明这一点：第二次世界大战期间纳粹德国海军使用的恩尼格玛机来加密通信，以及艾伦·图灵和 I.J.古德如何使用模糊测试技术破解海军恩尼格玛机的密码。"><text text-anchor="middle" x="76.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">何时停止模糊测试</text></a></g></g> <g id="edge14" class="edge"><title>覆盖率->何时停止模糊测试</title></g> <g id="node18" class="node"><title>GrammarFuzzer</title> <g id="a_node18"><a xlink:href="GrammarFuzzer.html" xlink:title="高效的语法模糊测试 (GrammarFuzzer)

在关于语法的章节中，我们看到了如何使用语法进行非常有效和高效的测试。&nbsp;在本章中，我们将之前基于字符串的算法精炼为基于树的算法，这要快得多，并且允许对模糊输入的生产有更多的控制。《高效语法》模糊测试</a></g></g> <g id="edge17" class="edge"><title>Grammars->GrammarFuzzer</title></g> <g id="node7" class="node"><title>Intro_Testing</title> <g id="a_node7"><a xlink:href="Intro_Testing.html" xlink:title="软件测试入门 (Intro_Testing)

在我们进入本书的核心部分之前，让我们介绍软件测试的基本概念。&nbsp;为什么需要测试软件？&nbsp;一个人如何测试软件？&nbsp;一个人如何判断测试是否成功？&nbsp;一个人如何知道是否测试得足够？&nbsp;在本章中，让我们回顾最重要的概念，同时熟悉 Python 和交互式笔记本。《软件测试入门》</a></g></g> <g id="edge6" class="edge"><title>Intro_Testing->Fuzzer</title></g> <g id="node16" class="node"><title>GreyboxFuzzer</title> <g id="a_node16"><a xlink:href="GreyboxFuzzer.html" xlink:title="灰盒模糊测试 (GreyboxFuzzer)

在上一章中，我们介绍了基于变异的模糊测试，这是一种通过在给定输入上应用小变异来生成模糊输入的技术。在本章中，我们展示了如何引导这些变异以实现特定的目标，如覆盖率。本章中的算法源自流行的美国模糊跳蚤（AFL）模糊器，特别是其 AFLFast 和 AFLGo 版本。我们将探索 AFL 背后的灰盒模糊测试算法，以及我们如何利用它来解决自动化漏洞检测的各种问题。《灰盒模糊测试》</a></g></g> <g id="edge15" class="edge"><title>MutationFuzzer->GreyboxFuzzer</title></g> <g id="node24" class="node"><title>GrammarMiner</title> <g id="a_node24"><a xlink:href="GrammarMiner.html" xlink:title="挖掘输入语法 (GrammarMiner)

到目前为止，我们看到的语法大多是手动指定的——也就是说，你必须（或者知道输入格式的人）首先设计和编写一个语法。 &nbsp;虽然我们迄今为止看到的语法相对简单，但为复杂输入创建语法可能需要相当多的努力。 &nbsp;因此，在本章中，我们介绍了从程序中自动挖掘语法的技巧——通过执行程序并观察它们如何处理输入的哪些部分。 &nbsp;结合语法模糊器，这使我们能够

1. 选择一个程序，

2. 提取其输入语法，

使用本书中的概念以高效率和效果进行模糊处理。"><text text-anchor="middle" x="1050.62" y="-98.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">挖掘输入</text> <text text-anchor="middle" x="1050.62" y="-80.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">语法</text></a></g></g> <g id="edge25" class="edge"><title>GrammarCoverageFuzzer->GrammarMiner</title></g> <g id="node25" class="node"><title>ConfigurationFuzzer</title> <g id="a_node25"><a xlink:href="ConfigurationFuzzer.html" xlink:title="测试配置（ConfigurationFuzzer）

程序的行为不仅受其数据控制。 &nbsp;程序的配置——即通过选项或配置文件设置的，控制程序在其（常规）输入数据上执行设置的设置——同样会影响行为，因此可以也应该进行测试。 &nbsp;在本章中，我们探讨如何系统地测试和覆盖软件配置。 &nbsp;通过自动推断配置选项，我们可以直接应用这些技术，无需编写语法。 &nbsp;最后，我们展示如何系统地覆盖配置选项的组合，快速检测不希望出现的干扰。"><text text-anchor="middle" x="1039.62" y="-178.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">测试</text> <text text-anchor="middle" x="1039.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">配置</text></a></g></g> <g id="edge26" class="edge"><title>GrammarCoverageFuzzer->ConfigurationFuzzer</title></g> <g id="node26" class="node"><title>Carver</title> <g id="a_node26"><a xlink:href="Carver.html" xlink:title="切割单元测试（Carver）

到目前为止，我们总是生成系统输入，即程序整体通过其输入通道获得的数据。如果我们只对测试一小组函数感兴趣，必须通过系统进行测试可能会非常低效。本章介绍了一种称为雕刻的技术，它给定一个系统测试，自动提取一组单元测试，这些测试复制了系统测试期间看到的调用。关键思想是记录这些调用，以便我们可以在以后重新播放它们——整体或选择性地。此外，我们还探讨了如何从雕刻的单元测试中合成 API 语法；这意味着我们可以合成 API 测试而无需编写任何语法。《<text text-anchor="middle" x="878.62" y="-13.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Carving Unit Tests</text></a></g></g> <g id="edge27" class="edge"><title>GrammarCoverageFuzzer->Carver</title></g> <g id="node27" class="node"><title>GUIFuzzer</title> <g id="a_node27"><a xlink:href="GUIFuzzer.html" xlink:title="Testing Graphical User Interfaces (GUIFuzzer)

在本章中，我们探讨如何为图形用户界面（GUI）生成测试，从我们之前关于 Web 测试的示例中抽象出来。基于提取用户界面元素和激活它们的一般方法，我们的技术可以推广到任意图形用户界面，从富 Web 应用到移动应用，并通过表单和导航元素系统地探索用户界面。《<text text-anchor="middle" x="1338.62" y="-178.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Testing Graphical</text> <text text-anchor="middle" x="1338.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">User Interfaces</text></a></g></g> <g id="edge28" class="edge"><title>GrammarCoverageFuzzer->GUIFuzzer</title></g> <g id="node29" class="node"><title>APIFuzzer</title> <g id="a_node29"><a xlink:href="APIFuzzer.html" xlink:title="Fuzzing APIs (APIFuzzer)

到目前为止，我们总是生成系统输入，即程序整体通过其输入通道获得的数据。然而，我们也可以生成直接进入单个函数的输入，在这个过程中获得灵活性和速度。在本章中，我们探讨使用语法来合成函数调用代码的使用，这使得您可以生成非常高效地直接调用函数的程序代码。《<text text-anchor="middle" x="598.62" y="-89.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Fuzzing APIs</text></a></g></g> <g id="edge32" class="edge"><title>ProbabilisticGrammarFuzzer->APIFuzzer</title></g> <g id="node17" class="node"><title>GreyboxGrammarFuzzer</title> <g id="a_node17"><a xlink:href="GreyboxGrammarFuzzer.html" xlink:title="Greybox Fuzzing with Grammars (GreyboxGrammarFuzzer)

在本章中，我们介绍了对我们句法模糊测试技术的重要扩展，所有这些扩展都利用了现有输入的句法部分。《使用语法进行灰盒模糊测试》<text text-anchor="middle" x="739.62" y="-98.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Greybox Fuzzing with</text> <text text-anchor="middle" x="739.62" y="-80.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Grammars</text></a></g></g> <g id="edge16" class="edge"><title>GreyboxFuzzer->GreyboxGrammarFuzzer</title></g> <g id="edge18" class="edge"><title>GrammarFuzzer->GrammarCoverageFuzzer</title></g> <g id="edge23" class="edge"><title>GrammarFuzzer->PythonFuzzer</title></g> <g id="node19" class="node"><title>Parser</title> <g id="a_node19"><a xlink:href="Parser.html" xlink:title="Parsing Inputs (Parser)

在关于语法的章节中，我们讨论了语法如何

用于表示各种语言。我们还看到了语法如何

生成对应语言的字符串。语法还可以执行

反向。也就是说，给定一个字符串，可以将字符串分解为其

与生成它的语法中使用的部分相对应的组成部分

– 该字符串的推导树。这些部分（以及来自其他类似

字符串的部分）可以稍后使用相同的语法重新组合，以生成新的字符串。"><text text-anchor="middle" x="901.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Parsing Inputs</text></a></g></g> <g id="edge19" class="edge"><title>GrammarFuzzer->Parser</title></g> <g id="node20" class="node"><title>GeneratorGrammarFuzzer</title> <g id="a_node20"><a xlink:href="GeneratorGrammarFuzzer.html" xlink:title="Fuzzing with Generators (GeneratorGrammarFuzzer)

在本章中，我们展示了如何通过函数扩展语法——这些代码片段在语法扩展期间执行，可以生成、检查或更改生成的元素。向语法添加函数允许进行非常灵活的测试生成，将语法生成和编程的最佳之处结合起来。《使用生成器模糊测试》<text text-anchor="middle" x="714.62" y="-178.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Fuzzing with</text> <text text-anchor="middle" x="714.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Generators</text></a></g></g> <g id="edge20" class="edge"><title>GrammarFuzzer->GeneratorGrammarFuzzer</title></g> <g id="node21" class="node"><title>Reducer</title> <g id="a_node21"><a xlink:href="Reducer.html" xlink:title="Reducing Failure-Inducing Inputs (Reducer)

通过构建，模糊器生成的输入可能难以阅读。这会在调试期间引起问题，当人类需要分析失败的确切原因时。在本章中，我们介绍了自动将导致失败的输入简化到最小以简化调试的技术。《减少失败-诱导输入》<text text-anchor="middle" x="1301.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Reducing Failure-</text> <text text-anchor="middle" x="1301.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Inducing Inputs</text></a></g></g> <g id="edge21" class="edge"><title>GrammarFuzzer->Reducer</title></g> <g id="node22" class="node"><title>FuzzingWithConstraints</title> <g id="a_node22"><a xlink:href="FuzzingWithConstraints.html" xlink:title="Fuzzing with Constraints (FuzzingWithConstraints)

在前面的章节中，我们看到了基于语法的模糊测试如何使我们能够高效地生成大量的语法有效输入。

然而，有一些语义输入特征无法在上下文无关语法中表达，例如《使用约束进行模糊测试》<text text-anchor="middle" x="1173.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Fuzzing with</text> <text text-anchor="middle" x="1173.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Constraints</text></a></g></g> <g id="edge22" class="edge"><title>GrammarFuzzer->FuzzingWithConstraints</title></g> <g id="node23" class="node"><title>WebFuzzer</title> <g id="a_node23"><a xlink:href="WebFuzzer.html" xlink:title="Testing Web Applications (WebFuzzer)

在本章中，我们探讨了如何为图形用户界面（GUIs），特别是 Web 界面生成测试。我们设置了一个（有漏洞的）Web 服务器，并展示了如何系统地探索其行为——首先使用手写的语法，然后使用从用户界面自动推断出的语法。我们还展示了如何对这些服务器进行系统性的攻击，特别是使用代码和 SQL 注入。《测试 Web》<text text-anchor="middle" x="1427.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Testing Web</text> <text text-anchor="middle" x="1427.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Applications</text></a></g></g> <g id="edge24" class="edge"><title>GrammarFuzzer->WebFuzzer</title></g> <g id="edge29" class="edge"><title>Parser->ProbabilisticGrammarFuzzer</title></g> <g id="edge30" class="edge"><title>Parser->GreyboxGrammarFuzzer</title></g> <g id="node28" class="node"><title>InformationFlow</title> <g id="a_node28"><a xlink:href="InformationFlow.html" xlink:title="Tracking Information Flow (InformationFlow)

我们已经探讨了如何生成更好的输入，这些输入可以深入到要测试的程序中。在这样做的时候，我们依赖于程序崩溃来告诉我们我们已经成功地在程序中找到了问题。然而，这相当简单。如果程序的行为只是不正确，但不会导致崩溃呢？能否做得更好？"><text text-anchor="middle" x="892.62" y="-178.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">跟踪信息</text> <text text-anchor="middle" x="892.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">流程</text></a></g></g> <g id="edge31" class="edge"><title>Parser->InformationFlow</title></g> <g id="edge33" class="edge"><title>GeneratorGrammarFuzzer->APIFuzzer</title></g> <g id="edge37" class="edge"><title>WebFuzzer->GUIFuzzer</title></g> <g id="edge35" class="edge"><title>InformationFlow->ConcolicFuzzer</title></g> <g id="edge34" class="edge"><title>InformationFlow->GrammarMiner</title></g> <g id="edge36" class="edge"><title>APIFuzzer->Carver</title></g></g></svg>

但是，由于即使是这张地图也可能让人感到不知所措，这里有一些*导览*来帮助你开始。这些导览允许你根据你是程序员、学生还是研究人员，专注于特定的视图。

## 实用主义程序员导览

你有一个要测试的程序。你希望尽可能快地生成尽可能彻底的测试。你不太关心*如何*实现，但应该能完成任务。你想要学习如何*使用*事物。

1.  **从测试简介开始**以获取基本概念。（你本来就会知道这些，但快速提醒一下也无妨）。

1.  **使用模糊器章节中的简单模糊器**来测试你的程序与第一个随机输入。

1.  **从你的程序中获取覆盖率**并使用覆盖率信息来引导测试生成以实现代码覆盖率。

1.  **为你的程序定义一个输入语法**并使用这个语法彻底模糊测试你的程序，使用语法正确的输入。作为模糊器，我们推荐使用语法覆盖率模糊器，因为这确保了输入元素的覆盖率。

1.  如果你想要**对生成的输入有更多控制**，考虑概率模糊测试和使用生成函数的模糊测试。

1.  如果你想要**部署大量模糊器**，学习如何管理大量模糊器。

在每一章中，从“概述”部分开始；这些将快速介绍如何使用事物，并指向相关的使用示例。就这样吧。回去工作并享受吧！

## 按页导览

这些之旅是本书的组织方式。在阅读了测试简介以获得基本概念之后，你可以通过这些部分进行阅读：

1.  **词法之旅**专注于*词法*测试生成技术，即逐字符和逐字节组合输入的技术。非常快速且健壮的技术，具有最小的偏差。

1.  **语法之旅**专注于*语法*作为指定输入语法的手段。生成的测试生成器产生语法正确的输入，使测试更快，并为测试者提供大量控制机制。

1.  **语义之旅**利用*代码语义*来塑造和引导测试生成。高级技术包括提取输入语法、挖掘函数规范和符号约束求解，以覆盖尽可能多的代码路径。

1.  **应用之旅**将早期部分定义的技术应用于 Web 服务器、用户界面、API 或配置等领域。

1.  **管理之旅**最终关注如何处理和组织大量测试生成器，以及何时停止模糊测试。

大多数这些章节都从“概要”部分开始，解释如何使用最重要的概念。你可以选择是否想要一个“使用”视角（那么只需阅读概要）或一个“理解”视角（那么继续阅读）。

## 本科生之旅

你是计算机科学和/或软件工程的学生。你想要了解测试和相关领域的基础知识。除了仅仅*使用*技术，你还想深入挖掘算法和实现。以下是我们为你推荐的：

1.  从测试简介和覆盖率开始，以获得**基本概念**。（你可能已经了解其中的一些，但嘿，你是个学生，对吧？）

1.  **从模糊器章节学习简单的模糊器是如何工作的**。这已经为你提供了在 90 年代摧毁了 30%的 UNIX 工具的工具。如果你测试一个从未被模糊测试过的工具会发生什么？

1.  **基于变异的模糊测试**是当今模糊测试的标准：取一组种子，并对其进行变异，直到找到错误。

1.  **学习如何使用语法生成语法正确的输入**。这使得测试生成更加高效，但首先你必须编写（或挖掘）一个语法。

1.  **学习如何 fuzz API 和图形用户界面**。这两个都是软件测试生成的重要领域。

1.  **学习如何自动将导致失败的输入减少到最小**。这对于调试来说是一个巨大的节省时间，尤其是在与自动化测试结合使用时。

对于所有这些章节，请尝试实验实现，以理解其概念。请随意进行实验。

如果你是一名教师，上述章节可以在编程和/或软件工程课程中派上用场。利用幻灯片和/或现场编程，让学生们完成练习。

## 研究生之旅

在“本科生”之旅的基础上，你希望更深入地了解测试生成技术，包括更复杂的技术。

1.  **基于搜索的测试** 允许你引导测试生成向特定目标发展，例如代码覆盖率。稳健且高效。

1.  了解**配置测试**的介绍。如何测试和覆盖带有多个配置选项的系统？

1.  **变异分析** 将合成缺陷（变异）种入程序代码，以检查测试是否能够找到它们。如果测试没有找到变异，它们可能也不会找到真正的错误。

1.  **学习如何解析输入** 使用语法。如果你想分析、分解、变异现有输入，你需要一个解析器。

1.  **Concolic 和 符号 模糊测试** 通过解决程序路径上的约束来达到难以测试的代码。在可靠性至关重要的地方使用；也是一个热门的研究课题。

1.  **学习如何估计何时停止模糊测试**。总得在某处停下来，对吧？

对于所有这些章节，请尝试实验代码；自由地创建你自己的变体和扩展。这就是我们如何进行研究的！

如果你是一名教师，上述章节可以在软件工程和测试的高级课程中派上用场。再次强调，你可以利用幻灯片和/或现场编程，让学生们完成练习。

## 黑盒之旅

这次之旅专注于 *黑盒模糊测试* ——也就是说，不需要来自被测试程序的反馈的技术。看看

1.  **基本模糊测试**。这已经为你提供了在 90 年代摧毁了 30%的 UNIX 工具的工具。如果你测试一个以前从未进行过模糊测试的工具会发生什么呢？

1.  **语法模糊测试** 专注于 *语法* 作为指定输入语法的手段。生成的测试生成器产生语法正确的输入，使测试更快，并为测试者提供大量的控制机制。

1.  **语义模糊测试** 将 *约束* 附加到语法上，使得输入不仅语法有效，而且 *语义* 也有效——并赋予你塑造测试输入的能力，就像你希望的那样，

1.  **特定领域模糊测试** 展示了这些技术的多种应用，从配置到图形用户界面。

1.  如果你想**部署大量模糊测试器**，学习如何管理大量模糊测试器。

## 白盒之旅

这次游览专注于*白盒模糊测试*——即利用被测试程序反馈的技术。看看

1.  **覆盖率** 了解覆盖率的基本概念以及如何为 Python 测量它。

1.  **基于变异的模糊测试** 在今天的模糊测试中几乎是标准：取一组种子，并变异它们，直到找到错误。

1.  **灰盒模糊测试** 使用来自流行的美国模糊跳蚤（AFL）模糊测试器的算法。

1.  **信息流** 和 **约束性模糊测试** 展示了如何在 Python 程序中捕获信息流以及如何利用它来生成更智能的测试用例。

1.  **符号模糊测试**，在不执行程序的情况下推理程序的行为。

## 研究者游览

在“研究生”游览之上，你正在寻找介于实验室阶段和广泛应用之间的技术——特别是，仍有大量改进空间的技术。如果你在寻找研究想法，就选择这些主题。

1.  **挖掘函数规范** 是研究中的一个热门话题：给定一个函数，我们如何推断一个描述其行为的抽象模型？与测试生成相结合在这里提供了几个机会，特别是对于动态规范挖掘。

1.  **挖掘输入语法** 承诺将词汇模糊测试的鲁棒性和易用性与语法模糊测试的效率和速度相结合。想法是从程序中自动挖掘输入语法，然后作为语法模糊测试的基础。仍处于早期阶段，但潜力巨大。

1.  **概率语法模糊测试** 为程序员提供了对哪些元素应该生成的很多控制。在本章概述的概率模糊测试和从给定测试中挖掘数据交叉处有许多研究可能性。

1.  **使用生成器的模糊测试** 和 **使用约束的模糊测试** 为程序员提供了对输入生成的最终控制权，即通过允许他们定义自己的生成器函数或定义自己的输入约束。最大的挑战是：如何在最小的上下文约束下最大限度地利用语法描述的威力？

1.  **切割单元测试** 通过从仅重放单个函数调用（可能带有新生成的参数）的程序执行中提取单元测试，承诺可以显著加快测试执行（和生成）。在 Python 中，切割简单易行；这里有很多可以玩的空间。

1.  **测试 Web 服务器和图形用户界面**是一个热门的研究领域，由实践者测试和保障其接口的需求（以及其他实践者突破这些接口的需求）推动。同样，这里还有许多未探索的潜力。

1.  **灰盒模糊测试和基于语法的灰盒模糊测试** 引入了*统计估计器*，以引导测试生成向最有可能发现新漏洞的输入和输入属性。测试、程序分析和统计学的交汇处为未来的研究提供了许多可能性。

对于所有这些主题，拥有实现并展示这些概念的 Python 源代码是一个重要的资产。你可以轻松地用你自己的想法扩展实现，并在笔记本中直接运行评估。一旦你的方法稳定，考虑将其移植到具有更广泛可用主题的语言（例如 C 语言）。

## 作者之旅

这是一次终极之旅——你已经学到了所有内容，并希望为这本书做出贡献。那么，你应该再读两章：

1.  **作者指南**介绍了如何为这本书做出贡献（编码风格、写作风格、约定等）。

1.  **模板章节**是您章节的蓝图。

如果你想做出贡献，请随时联系我们——最好在写作之前，但写作之后也可以。我们将很高兴将你的材料纳入其中。

## 经验教训

+   您可以从头到尾阅读这本书...

+   ...但根据你的需求和资源，遵循一个特定的路线可能更合适。

+   现在去探索生成软件测试!

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容根据[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)授权。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，根据[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)授权。[最后修改：2024-06-30 18:32:41+02:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/Tours.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[书中的之旅](https://www.fuzzingbook.org/html/Tours.html)". 在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 的 "[模糊测试书](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/Tours.html`](https://www.fuzzingbook.org/html/Tours.html). Retrieved 2024-06-30 18:32:41+02:00.

```py
@incollection{fuzzingbook2024:Tours,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Tours through the Book},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/Tours.html}},
    note = {Retrieved 2024-06-30 18:32:41+02:00},
    url = {https://www.fuzzingbook.org/html/Tours.html},
    urldate = {2024-06-30 18:32:41+02:00}
}

```
