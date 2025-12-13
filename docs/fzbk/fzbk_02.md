# 模糊测试书籍

> 原文：[`www.fuzzingbook.org/html/00_Table_of_Contents.html`](https://www.fuzzingbook.org/html/00_Table_of_Contents.html)

## 网站地图

虽然这本书的章节可以依次阅读，但书中有许多可能的路径。在这个图中，箭头 *A* → *B* 表示章节 *A* 是章节 *B* 的先决条件。你可以选择任意路径来获取你最感兴趣的主题：

<svg width="1482pt" height="622pt" viewBox="0.00 0.00 1481.88 622.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 618)"><g id="node1" class="node"><title>模糊器</title> <g id="a_node1"><a xlink:href="Fuzzer.html" xlink:title="模糊测试：使用随机输入破坏事物 (模糊器)

在本章中，我们将从最简单的测试生成技术开始。随机文本生成，也称为模糊测试，的关键思想是将一串随机字符输入到程序中，希望揭露故障。《模糊测试：破坏》<text text-anchor="middle" x="910.62" y="-516.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">模糊测试：破坏</text> <text text-anchor="middle" x="910.62" y="-498.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">事物</text> <text text-anchor="middle" x="910.62" y="-480.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">使用随机输入</text></a></g></g> <g id="node2" class="node"><title>覆盖率</title> <g id="a_node2"><a xlink:href="Coverage.html" xlink:title="代码覆盖率 (Coverage)

在上一章中，我们介绍了基本的模糊测试——即生成随机输入来测试程序。&nbsp;我们如何衡量这些测试的有效性？&nbsp;一种方法就是检查找到的（数量和严重性）错误；但如果错误很少，我们需要一个代理来衡量测试发现错误的概率。&nbsp;在本章中，我们引入了代码覆盖的概念，测量在测试运行期间程序的实际执行部分。&nbsp;对于试图覆盖尽可能多代码的测试生成器来说，测量这种覆盖率也是至关重要的。《代码覆盖率</text></a></g></g> <g id="edge1" class="edge"><title>模糊器->覆盖率</title></g> <g id="node3" class="node"><title>基于搜索的模糊器</title> <g id="a_node3"><a xlink:href="SearchBasedFuzzer.html" xlink:title="基于搜索的模糊测试 (SearchBasedFuzzer)

有时我们不仅对模糊测试尽可能多的不同程序输入感兴趣，还希望推导出能够实现某些目标的特定测试输入，例如到达程序中的特定语句。当我们有了我们想要寻找的东西的概念时，我们就可以去寻找它。搜索算法是计算机科学的核心，但将经典搜索算法如广度优先搜索或深度优先搜索应用于测试搜索是不切实际的，因为这些算法可能需要我们查看所有可能的输入。然而，领域知识可以用来克服这个问题。例如，如果我们能够估计几个程序输入中哪一个更接近我们正在寻找的，那么这些信息可以引导我们更快地达到目标——这种信息被称为启发式。启发式系统地应用的方式被元启发式搜索算法所捕捉。"元"表示这些算法是通用的，并且可以根据不同的问题实例化。元启发式通常从自然界观察到的过程中获得灵感。例如，有一些算法模仿进化过程、群体智能或化学反应。总的来说，它们比穷举搜索方法更有效率，因此可以应用于广阔的搜索空间——对于它们来说，程序输入领域的广阔搜索空间不是问题。《基于搜索的模糊测试》</text></a></g></g> <g id="edge2" class="edge"><title>Fuzzer->SearchBasedFuzzer</title></g> <g id="node4" class="node"><title>语法</title> <g id="a_node4"><a xlink:href="Grammars.html" xlink:title="使用语法进行模糊测试 (Grammars)

在“基于变异的模糊测试”章节中，我们看到了如何使用额外的提示——例如样本输入文件——来加速测试生成。在本章中，我们将这一想法进一步发展，通过提供程序合法输入的规范。通过语法指定输入可以实现非常系统和高效的测试生成，特别是在复杂的输入格式中。语法还作为配置模糊测试、API 模糊测试、GUI 模糊测试等的基础。《使用语法进行模糊测试》</text></a></g></g> <g id="edge3" class="edge"><title>Fuzzer->Grammars</title></g> <g id="node5" class="node"><title>SymbolicFuzzer</title> <g id="a_node5"><a xlink:href="SymbolicFuzzer.html" xlink:title="符号模糊测试 (SymbolicFuzzer)

传统模糊测试方法的一个问题是，它们无法测试系统可能具有的所有可能行为，尤其是在输入空间很大时。很多时候，特定执行分支的执行可能只发生在非常特定的输入下，这可能只占输入空间的一小部分。传统的模糊测试方法依赖于偶然来产生所需的输入。然而，当要探索的空间很大时，依赖于随机生成我们想要的值是一个坏主意。例如，一个接受字符串的函数，即使只考虑前 10 个字符，也有$2^{80}$种可能的输入。如果寻找特定的字符串，即使在超级计算机上，随机生成值也需要几千年。《符号模糊测试</a></g></g> <g id="edge4" class="edge"><title>Fuzzer->SymbolicFuzzer</title></g> <g id="node6" class="node"><title>大规模模糊测试</title> <g id="a_node6"><a xlink:href="FuzzingInTheLarge.html" xlink:title="大规模模糊测试（FuzzingInTheLarge）

在前面的章节中，我们总是只关注几秒钟内在一台机器上进行的模糊测试。然而，在现实世界中，模糊测试通常在数十甚至数千台机器上运行；持续数小时、数天甚至数周；针对一个或数十个程序。在这种情况下，需要一个基础设施来收集单个模糊测试运行中的故障数据，并将这些数据汇总到中央存储库中。在本章中，我们将探讨这样一个基础设施，即来自 Mozilla 的 FuzzManager 框架。《大规模模糊测试</a></g></g> <g id="edge5" class="edge"><title>Fuzzer->FuzzingInTheLarge</title></g> <g id="node8" class="node"><title>变异模糊测试器</title> <g id="a_node8"><a xlink:href="MutationFuzzer.html" xlink:title="基于变异的模糊测试（变异模糊测试器）

大多数随机生成的输入在语法上都是无效的，因此很快就会被处理程序拒绝。为了在输入处理之外锻炼功能，我们必须增加获得有效输入的机会。其中一种方法就是所谓的突变模糊测试——也就是说，对现有输入进行微小修改，这些修改可能仍然保持输入的有效性，同时测试新的行为。我们展示了如何创建这样的突变，以及如何引导它们指向尚未发现的代码，应用了流行的 AFL 模糊器中的核心概念。《基于突变的模糊测试》<text text-anchor="middle" x="551.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Mutation-Based</text> <text text-anchor="middle" x="551.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Fuzzing</text></a></g></g> <g id="edge7" class="edge"><title>Coverage->MutationFuzzer</title></g> <g id="node9" class="node"><title>MutationAnalysis</title> <g id="a_node9"><a xlink:href="MutationAnalysis.html" xlink:title="Mutation Analysis (MutationAnalysis)

在关于覆盖率的章节中，我们展示了如何确定程序中哪些部分被执行，从而了解一组测试用例在覆盖程序结构方面的有效性。然而，覆盖率本身可能不是衡量测试有效性的最佳指标，因为即使没有检查结果是否正确，也可能有很高的覆盖率。在本章中，我们介绍了一种评估测试套件有效性的另一种方法：在代码中注入突变——人工错误后，我们检查测试套件是否能够检测到这些人工错误。其理念是，如果它未能检测到这样的突变，它也会错过真实的错误。《突变分析》<text text-anchor="middle" x="235.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">Mutation Analysis</text></a></g></g> <g id="edge8" class="edge"><title>Coverage->MutationAnalysis</title></g> <g id="node10" class="node"><title>GrammarCoverageFuzzer</title> <g id="a_node10"><a xlink:href="GrammarCoverageFuzzer.html" xlink:title="Grammar Coverage (GrammarCoverageFuzzer)

从语法中生成输入使得一个规则的每个可能的扩展都有相同的可能性。然而，为了生成一个全面的测试套件，最大化多样性更有意义——例如，通过避免重复相同的扩展。在本章中，我们探讨了如何系统地覆盖语法的元素，以最大化多样性并确保不遗漏任何单个元素。《语法覆盖率》（Grammar Coverage）</text> <a xlink:href="GrammarCoverageFuzzer.html" xlink:title="Grammar Coverage Fuzzer"><text text-anchor="middle" x="1039.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">语法覆盖率</text></a></g></g> <g id="edge9" class="edge"><title>覆盖率->语法覆盖率模糊测试器</title></g> <g id="node11" class="node"><title>ProbabilisticGrammarFuzzer</title> <g id="a_node11"><a xlink:href="ProbabilisticGrammarFuzzer.html" xlink:title="概率语法模糊测试 (ProbabilisticGrammarFuzzer)

让我们通过为单个扩展分配概率来赋予语法更多的能力。这使我们能够控制每种元素应该产生多少，从而允许我们将生成的测试针对特定的功能。我们还展示了如何从给定的样本输入中学习这样的概率，并特别将我们的测试针对这些样本中不常见的输入特征。《概率语法模糊测试》（Probabilistic Grammar Fuzzing）</text> <text text-anchor="middle" x="409.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">概率语法模糊测试</text></a></g></g> <g id="edge10" class="edge"><title>覆盖率->概率语法模糊测试器</title></g> <g id="node12" class="node"><title>ConcolicFuzzer</title> <g id="a_node12"><a xlink:href="ConcolicFuzzer.html" xlink:title="Concolic Fuzzing (ConcolicFuzzer)

在信息流章节中，我们看到了如何使用动态污点来生成比仅仅寻找程序崩溃更智能的测试用例。我们还看到了如何使用污点来更新语法，从而更专注于危险的方法。《动态不变量》（DynamicInvariants）</text> <a xlink:href="DynamicInvariants.html" xlink:title="挖掘函数规范 (DynamicInvariants)"><text text-anchor="middle" x="892.62" y="-89.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">动态不变量</text></a></g></g> <g id="edge11" class="edge"><title>覆盖率->ConcolicFuzzer</title></g> <g id="node13" class="node"><title>DynamicInvariants</title> <g id="a_node13"><a xlink:href="DynamicInvariants.html" xlink:title="挖掘函数规范 (DynamicInvariants)"><text text-anchor="middle" x="1039.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">动态不变量</text></a></g></g>

在测试一个程序时，不仅需要覆盖其多种行为；还需要检查结果是否符合预期。在本章中，我们介绍了一种技术，使我们能够从一组给定的执行中挖掘函数规范，从而得到函数期望和提供的形式化和抽象描述。"><text text-anchor="middle" x="415.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">挖掘函数</text> <text text-anchor="middle" x="415.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">规范</text></a></g></g> <g id="edge12" class="edge"><title>Coverage->DynamicInvariants</title></g> <g id="node14" class="node"><title>PythonFuzzer</title> <g id="a_node14"><a xlink:href="PythonFuzzer.html" xlink:title="测试编译器 (PythonFuzzer)

在本章中，我们将利用语法和基于语法的测试来系统地生成程序代码——例如，用于测试编译器或解释器。不出所料，我们使用 Python 和 Python 解释器作为我们的领域。"><text text-anchor="middle" x="729.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">测试编译器</text></a></g></g> <g id="edge13" class="edge"><title>Coverage->PythonFuzzer</title></g> <g id="node15" class="node"><title>WhenToStopFuzzing</title> <g id="a_node15"><a xlink:href="WhenToStopFuzzing.html" xlink:title="何时停止模糊测试 (WhenToStopFuzzing)

在前面的章节中，我们讨论了几种模糊测试技术。知道该做什么很重要，但知道何时停止做某事也同样重要。在本章中，我们将学习何时停止模糊测试——并为此目的使用一个突出的例子：第二次世界大战中纳粹德国海军使用的恩尼格玛机来加密通信，以及艾伦·图灵和 I.J.古德如何使用模糊测试技术破解海军恩尼格玛机的密码。"><text text-anchor="middle" x="76.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">何时停止模糊测试</text></a></g></g> <g id="edge14" class="edge"><title>Coverage->WhenToStopFuzzing</title></g> <g id="node18" class="node"><title>GrammarFuzzer</title> <g id="a_node18"><a xlink:href="GrammarFuzzer.html" xlink:title="高效的语法模糊测试 (GrammarFuzzer)

在关于语法的章节中，我们看到了如何使用语法进行非常有效和高效的测试。在本章中，我们将之前基于字符串的算法精炼为基于树的算法，这要快得多，并且允许对模糊输入的生产有更多的控制。"><text text-anchor="middle" x="970.62" y="-338.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">高效语法</text> <text text-anchor="middle" x="970.62" y="-320.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">模糊测试</text></a></g></g> <g id="edge17" class="edge"><title>Grammars->GrammarFuzzer</title></g> <g id="node7" class="node"><title>Intro_Testing</title> <g id="a_node7"><a xlink:href="Intro_Testing.html" xlink:title="软件测试入门（Intro_Testing）

在我们进入本书的核心部分之前，让我们介绍软件测试的基本概念。为什么需要测试软件？如何测试软件？如何判断测试是否成功？如何知道是否测试得足够？在本章中，让我们回顾最重要的概念，同时熟悉 Python 和交互式笔记本。"><text text-anchor="middle" x="910.62" y="-596.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">软件测试入门</text> <text text-anchor="middle" x="910.62" y="-578.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">软件测试</text></a></g></g> <g id="edge6" class="edge"><title>Intro_Testing->Fuzzer</title></g> <g id="node16" class="node"><title>GreyboxFuzzer</title> <g id="a_node16"><a xlink:href="GreyboxFuzzer.html" xlink:title="灰盒模糊测试（GreyboxFuzzer）

在上一章中，我们介绍了基于变异的模糊测试技术，这是一种通过在给定输入上应用小变异来生成模糊输入的技术。在本章中，我们展示了如何引导这些变异以实现特定目标，例如覆盖率。本章中的算法源自流行的美国模糊跳蚤（AFL）模糊器，特别是其 AFLFast 和 AFLGo 版本。我们将探索 AFL 背后的灰盒模糊测试算法，以及我们如何利用它来解决自动化漏洞检测的各种问题。《灰盒模糊测试》（Greybox Fuzzing）</a></g></g> <g id="edge15" class="edge"><title>MutationFuzzer->GreyboxFuzzer</title></g> <g id="node24" class="node"><title>GrammarMiner</title> <g id="a_node24"><a xlink:href="GrammarMiner.html" xlink:title="挖掘输入语法（GrammarMiner）

到目前为止，我们所看到的语法大多是手动指定的——也就是说，你必须（或者知道输入格式的人）首先设计和编写一个语法。虽然我们迄今为止看到的语法相对简单，但为复杂输入创建语法可能需要相当多的努力。因此，在本章中，我们介绍了从程序中自动挖掘语法的技巧——通过执行程序并观察它们如何处理输入的哪些部分。与语法模糊器结合使用，这使我们能够

1. 选择一个程序，

2. 提取其输入语法，

使用本书中的概念以高效率和效果进行模糊处理。"><text text-anchor="middle" x="1050.62" y="-98.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">挖掘输入</text> <text text-anchor="middle" x="1050.62" y="-80.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">语法</text></a></g></g> <g id="edge25" class="edge"><title>GrammarCoverageFuzzer->GrammarMiner</title></g> <g id="node25" class="node"><title>ConfigurationFuzzer</title> <g id="a_node25"><a xlink:href="ConfigurationFuzzer.html" xlink:title="测试配置（ConfigurationFuzzer）

程序的行为不仅受其数据控制。程序配置——即控制程序在其（常规）输入数据上执行设置的选项或配置文件——同样会影响行为，因此可以也应该进行测试。在本章中，我们探讨了如何系统地测试和覆盖软件配置。通过自动推断配置选项，我们可以直接应用这些技术，无需编写语法。最后，我们展示了如何系统地覆盖配置选项的组合，快速检测不希望出现的干扰。"><text text-anchor="middle" x="1039.62" y="-178.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">测试</text> <text text-anchor="middle" x="1039.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">配置</text></a></g></g> <g id="edge26" class="edge"><title>GrammarCoverageFuzzer->ConfigurationFuzzer</title></g> <g id="node26" class="node"><title>Carver</title> <g id="a_node26"><a xlink:href="Carver.html" xlink:title="切割单元测试（Carver）

到目前为止，我们一直生成系统输入，即程序整体通过其输入通道获得的数据。如果我们只对测试一小组函数感兴趣，必须通过系统进行测试可能会非常低效。本章介绍了一种称为雕刻的技术，给定一个系统测试，自动提取一组单元测试，这些测试复制了系统测试期间看到的调用。关键思想是记录这些调用，以便我们可以在以后重新播放它们——整体或选择性地。此外，我们还探讨了如何从雕刻的单元测试中合成 API 语法；这意味着我们可以合成 API 测试，而无需编写任何语法。"><text text-anchor="middle" x="878.62" y="-13.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">雕刻单元测试</text></a></g></g> <g id="edge27" class="edge"><title>语法覆盖率模糊器->雕刻器</title></g> <g id="node27" class="node"><title>GUI 模糊器</title> <g id="a_node27"><a xlink:href="GUIFuzzer.html" xlink:title="测试图形用户界面（GUIFuzzer）

在本章中，我们探讨如何为图形用户界面（GUI）生成测试，从我们之前的 Web 测试示例中抽象出来。基于提取用户界面元素和激活它们的一般方法，我们的技术可以推广到任意图形用户界面，从富 Web 应用到移动应用，并通过表单和导航元素系统地探索用户界面。"><text text-anchor="middle" x="1338.62" y="-178.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">测试图形用户界面</text> <text text-anchor="middle" x="1338.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">（Testing Graphical User Interfaces）</text></a></g></g> <g id="edge28" class="edge"><title>语法覆盖率模糊器->GUI 模糊器</title></g> <g id="node29" class="node"><title>API 模糊器</title> <g id="a_node29"><a xlink:href="APIFuzzer.html" xlink:title="模糊化 API（APIFuzzer）

到目前为止，我们一直生成系统输入，即程序整体通过其输入通道获得的数据。然而，我们也可以生成直接进入单个函数的输入，在这个过程中获得灵活性和速度。在本章中，我们探讨使用语法来合成函数调用代码，这使得你可以生成非常高效地直接调用函数的程序代码。"><text text-anchor="middle" x="598.62" y="-89.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">模糊化 API</text></a></g></g> <g id="edge32" class="edge"><title>概率语法模糊器->API 模糊器</title></g> <g id="node17" class="node"><title>灰盒语法模糊器</title> <g id="a_node17"><a xlink:href="GreyboxGrammarFuzzer.html" xlink:title="使用语法进行灰盒模糊化（灰盒语法模糊器）

在本章中，我们介绍了对我们句法模糊技术的重要扩展，所有这些扩展都利用了现有输入的句法部分。"><text text-anchor="middle" x="739.62" y="-98.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">使用语法进行灰盒模糊测试</text> <text text-anchor="middle" x="739.62" y="-80.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">语法</text></a></g></g> <g id="edge16" class="edge"><title>GreyboxFuzzer->GreyboxGrammarFuzzer</title></g> <g id="edge18" class="edge"><title>GrammarFuzzer->GrammarCoverageFuzzer</title></g> <g id="edge23" class="edge"><title>GrammarFuzzer->PythonFuzzer</title></g> <g id="node19" class="node"><title>Parser</title> <g id="a_node19"><a xlink:href="Parser.html" xlink:title="解析输入 (Parser)

在关于语法的章节中，我们讨论了语法如何被

用于表示各种语言。我们还看到了语法如何被用来

生成相应语言的字符串。语法还可以执行

反向操作。也就是说，给定一个字符串，可以将字符串分解为其组成部分

与生成它的语法中使用的语法部分相对应的组成部分

– 该字符串的推导树。这些部分（以及来自其他类似

字符串）可以在以后使用相同的语法重新组合以生成新的字符串。"><text text-anchor="middle" x="901.62" y="-249.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">解析输入</text></a></g></g> <g id="edge19" class="edge"><title>GrammarFuzzer->Parser</title></g> <g id="node20" class="node"><title>GeneratorGrammarFuzzer</title> <g id="a_node20"><a xlink:href="GeneratorGrammarFuzzer.html" xlink:title="使用生成器进行模糊测试 (GeneratorGrammarFuzzer)

在本章中，我们展示了如何通过函数扩展语法 –&nbsp;在语法扩展期间执行的代码片段，可以生成、检查或更改生成的元素。 &nbsp;向语法中添加函数允许进行非常灵活的测试生成，将语法生成和编程的最佳之处结合起来。"><text text-anchor="middle" x="714.62" y="-178.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">使用</text> <text text-anchor="middle" x="714.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">生成器进行模糊测试</text></a></g></g> <g id="edge20" class="edge"><title>GrammarFuzzer->GeneratorGrammarFuzzer</title></g> <g id="node21" class="node"><title>Reducer</title> <g id="a_node21"><a xlink:href="Reducer.html" xlink:title="减少导致失败的输入 (Reducer)

通过构造，模糊器生成的输入可能难以阅读。 &nbsp;这会在调试期间造成问题，当人类需要分析失败的确切原因时。 &nbsp;在本章中，我们介绍了自动将导致失败的输入减少和简化的技术，以便于调试。"><text text-anchor="middle" x="1301.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">减少失败-</text> <text text-anchor="middle" x="1301.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">诱导输入</text></a></g></g> <g id="edge21" class="edge"><title>GrammarFuzzer->Reducer</title></g> <g id="node22" class="node"><title>FuzzingWithConstraints</title> <g id="a_node22"><a xlink:href="FuzzingWithConstraints.html" xlink:title="使用约束进行模糊测试 (FuzzingWithConstraints)

在前面的章节中，我们看到了基于语法的模糊测试如何使我们能够高效地生成大量的语法有效输入。

然而，有一些语义输入特征无法用上下文无关文法表达，例如"><text text-anchor="middle" x="1173.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">使用</text> <text text-anchor="middle" x="1173.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">约束</text></a></g></g> <g id="edge22" class="edge"><title>GrammarFuzzer->FuzzingWithConstraints</title></g> <g id="node23" class="node"><title>WebFuzzer</title> <g id="a_node23"><a xlink:href="WebFuzzer.html" xlink:title="测试网络应用程序 (WebFuzzer)

在本章中，我们探讨了如何为图形用户界面（GUIs），特别是网络界面生成测试。 &nbsp;我们设置了一个（有漏洞的）网络服务器，并展示了如何系统地探索其行为——&nbsp;首先使用手写的语法，然后使用从用户界面自动推断出的语法。 &nbsp;我们还展示了如何对这些服务器进行系统性的攻击，特别是使用代码和 SQL 注入。"><text text-anchor="middle" x="1427.62" y="-258.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">测试网络</text> <text text-anchor="middle" x="1427.62" y="-240.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">应用程序</text></a></g></g> <g id="edge24" class="edge"><title>GrammarFuzzer->WebFuzzer</title></g> <g id="edge29" class="edge"><title>解析器->概率语法模糊器</title></g> <g id="edge30" class="edge"><title>解析器->灰盒语法模糊器</title></g> <g id="node28" class="node"><title>信息流</title> <g id="a_node28"><a xlink:href="InformationFlow.html" xlink:title="跟踪信息流 (InformationFlow)

我们已经探讨了如何生成更好的输入，这些输入可以深入到所讨论的程序中。在这样做的时候，我们依赖于程序崩溃来告诉我们我们已经成功地在程序中找到了问题。然而，这相当简单。如果程序的行为只是不正确，但不会导致崩溃呢？能否做得更好？"><text text-anchor="middle" x="892.62" y="-178.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">跟踪信息</text> <text text-anchor="middle" x="892.62" y="-160.7" font-family="Patua One, Helvetica, sans-serif" font-size="14.00" fill="#b03a2e">流程</text></a></g></g> <g id="edge31" class="edge"><title>解析器->信息流</title></g> <g id="edge33" class="edge"><title>生成语法模糊测试器->API 模糊测试器</title></g> <g id="edge37" class="edge"><title>Web 模糊测试器->GUI 模糊测试器</title></g> <g id="edge35" class="edge"><title>信息流->Concolic 模糊测试器</title></g> <g id="edge34" class="edge"><title>信息流->语法挖掘器</title></g> <g id="edge36" class="edge"><title>API 模糊测试器->Carver</title></g></g></svg>

## 目录

### 第一部分：激发你的兴趣

在这部分，我们介绍了本书的主题。

#### 本书之旅

这本书非常庞大。拥有超过 20,000 行代码和 150,000 字的文本，印刷版将覆盖超过 1,200 页的文本。显然，我们并不假设每个人都想阅读所有内容。

#### 软件测试简介

在我们进入本书的核心部分之前，让我们介绍软件测试的基本概念。为什么需要测试软件？一个人如何测试软件？一个人如何知道测试是否成功？一个人如何知道是否测试得足够？在本章中，让我们回顾最重要的概念，同时熟悉 Python 和交互式笔记本。

### 第二部分：词汇模糊测试

这一部分介绍了*词汇级别*的测试生成，即字符序列的组成。

#### 模糊测试：使用随机输入破坏事物

在本章中，我们将从最简单的测试生成技术开始。随机文本生成的关键思想，也称为*模糊测试*，是将一串*随机字符*输入到程序中，希望揭示出故障。

#### 代码覆盖率

在上一章中，我们介绍了*基本模糊测试*——即生成随机输入以测试程序。我们如何衡量这些测试的有效性呢？一种方法就是检查找到的（数量和严重性）错误；但如果错误很少，我们需要一个*测试发现错误的可能性的代理*。在本章中，我们引入了*代码覆盖率*的概念，测量在测试运行期间程序的实际执行部分。测量这种覆盖率对于试图覆盖尽可能多代码的测试生成器来说也是至关重要的。

#### 基于突变的模糊测试

大多数随机生成的输入在语法上是*无效的*，因此很快就会被处理程序拒绝。为了测试输入处理之外的功能，我们必须增加获得有效输入的机会。其中一种方法就是所谓的*突变模糊测试*——即对现有输入进行微小更改，这些更改可能仍然保持输入有效，但可以测试新的行为。我们展示了如何创建这样的突变，以及如何引导它们指向尚未发现的代码，应用来自流行的 AFL 模糊测试器的核心概念。

#### 灰盒模糊测试

在上一章中，我们介绍了*基于突变的模糊测试*，这是一种通过在给定输入上应用小突变来生成模糊输入的技术。在本章中，我们展示了如何*引导*这些突变指向特定的目标，如覆盖率。本章中的算法源自流行的[American Fuzzy Lop](http://lcamtuf.coredump.cx/afl/)（AFL）模糊测试器，特别是其[AFLFast](https://github.com/mboehme/aflfast)和[AFLGo](https://github.com/aflgo/aflgo)版本。我们将探索 AFL 背后的灰盒模糊测试算法以及我们如何利用它来解决自动化漏洞检测的各种问题。

#### 基于搜索的模糊测试

有时候，我们不仅对模糊测试尽可能多的不同程序输入感兴趣，还希望推导出*特定*的测试输入，以达到某些目标，例如到达程序中的特定语句。当我们有了我们想要寻找的东西的概念时，我们就可以*搜索*它了。搜索算法是计算机科学的核心，但将经典搜索算法如广度优先搜索或深度优先搜索应用于测试搜索是不切实际的，因为这些算法可能需要我们查看所有可能的输入。然而，领域知识可以用来克服这个问题。例如，如果我们能估计几个程序输入中哪一个更接近我们想要找的，那么这个信息可以引导我们更快地达到目标——这个信息被称为*启发式*。启发式系统地应用的方式被捕获在*元启发式*搜索算法中。"Meta"表示这些算法是通用的，并且可以根据不同的问题实例化。元启发式通常从自然界观察到的过程中获得灵感。例如，有一些算法模仿进化过程、群体智能或化学反应。总的来说，它们比穷举搜索方法更有效率，因此可以应用于广阔的搜索空间——对于它们来说，程序输入领域的广阔搜索空间不是问题。

#### 突变分析

在关于覆盖率（Coverage）的章节中，我们展示了如何确定程序中哪些部分被执行，从而对一组测试用例在覆盖程序结构方面的有效性有一个概念。然而，覆盖率本身可能不是衡量测试有效性的最佳指标，因为一个人可以有很高的覆盖率，但从未检查结果是否正确。在这一章中，我们介绍了一种评估测试套件有效性的另一种方法：在代码中注入*突变*——*人工故障*后，我们检查测试套件是否可以检测到这些人工故障。这个想法是，如果它未能检测到这样的突变，它也会错过真实的错误。

### 第三部分：语法模糊测试

这一部分介绍了*语法*级别的测试生成，即从语言结构中组合输入。

#### 使用语法进行模糊测试

在"基于突变的模糊测试"（Mutation-Based Fuzzing）这一章中，我们看到了如何使用额外的提示——例如样本输入文件——来加速测试生成。在这一章中，我们更进一步，通过提供程序合法输入的*规范*。通过*语法*指定输入允许非常系统和高效的测试生成，特别是对于复杂的输入格式。语法还作为配置模糊测试、API 模糊测试、GUI 模糊测试等的基础。

#### 高效的语法模糊测试

在语法章节中，我们看到了如何使用*语法*进行非常有效和高效的测试。在本章中，我们将之前的基于*字符串*的算法精炼为基于*树*的算法，这要快得多，并允许对模糊输入的生产有更多的控制。

#### 语法覆盖率

从语法生成输入为规则的每个可能扩展赋予相同的可能性。然而，为了生成全面的测试套件，最大化*多样性*更有意义——例如，不要反复重复相同的扩展。在本章中，我们探讨了如何系统地*覆盖*语法的元素，以最大化多样性，并确保不遗漏单个元素。

#### 解析输入

在语法章节中，我们讨论了如何使用语法来表示各种语言。我们还看到了如何使用语法来生成对应语言的字符串。语法还可以执行相反的操作。也就是说，给定一个字符串，可以将该字符串分解为其组成部分，这些组成部分对应于用于生成它的语法的部分——该字符串的*推导树*。这些部分（以及来自其他类似字符串的部分）可以稍后使用相同的语法重新组合，以生成新的字符串。

#### 概率语法模糊

让我们通过为单个扩展分配*概率*来赋予语法更多的能力。这允许我们控制应该生成多少个每个元素，从而允许我们将生成的测试针对特定的功能。我们还展示了如何从给定的样本输入中学习这样的概率，并具体地将测试指向这些样本中不常见的输入特征。

#### 使用生成器进行模糊测试

在本章中，我们展示了如何通过*函数*扩展语法——这些代码在语法扩展期间执行，可以生成、检查或更改生成的元素。向语法添加函数允许进行非常灵活的测试生成，结合了语法生成和编程的最佳之处。

#### 使用语法进行灰盒模糊测试

在本章中，我们介绍了对我们语法模糊技术的重要扩展，所有这些扩展都利用了*现有输入*的*语法*部分。

#### 减少导致失败的输入

通过构造，模糊器创建的输入可能难以阅读。这会在*调试*期间造成问题，当人类需要分析失败的确切原因时。在本章中，我们介绍了将导致失败输入自动减少和简化的技术，以简化调试过程。

### 第四部分：语义模糊测试

本部分介绍了考虑输入*语义*的测试生成技术，特别是处理输入的程序的行为。

#### 约束模糊测试

在前面的章节中，我们看到了基于语法的模糊测试如何使我们能够高效地生成大量的语法有效输入。然而，有一些*语义*输入特征无法在上下文无关语法中表达，例如

#### 挖掘输入语法

到目前为止，我们所看到的语法大多是手动指定的——也就是说，你必须（或知道输入格式的人）首先设计和编写一个语法。虽然我们迄今为止看到的语法相当简单，但为复杂输入创建语法可能需要相当多的努力。因此，在这一章中，我们介绍了从程序中*自动挖掘语法*的技术——通过执行程序并观察它们如何处理输入的哪些部分。与语法模糊测试器结合使用，这使我们能够

1.  取一个程序，

1.  提取其输入语法，并且

1.  使用本书中的概念，以高效率和有效性进行模糊测试。#### 跟踪信息流

我们已经探讨了如何生成更好的输入，这些输入可以深入到所讨论的程序中。在这样做的时候，我们依赖于程序崩溃来告诉我们我们已经成功地在程序中找到了问题。然而，这相当简单。如果程序的行为只是不正确，但不会导致崩溃呢？能否做得更好？

#### 约束模糊测试

在信息流章节中，我们看到了如何使用动态污点来生成比仅仅寻找程序崩溃更智能的测试用例。我们也看到了如何使用污点来更新语法，从而更专注于危险的方法。

#### 符号模糊测试

模糊测试的传统方法中存在的问题是，它们无法锻炼系统可能具有的所有可能行为，尤其是在输入空间很大时。很多时候，特定执行分支的执行可能只发生在非常特定的输入上，这可能只占输入空间的一小部分。传统的模糊测试方法依赖于机会来生成它们需要的输入。然而，当要探索的空间很大时，依赖于随机性来生成我们想要的值是一个坏主意。例如，一个接受字符串的函数，即使只考虑前 10 个字符，也有$2^{80}$种可能的输入。如果寻找特定的字符串，即使在超级计算机上，随机生成值也需要几千年。

#### 挖掘函数规范

在测试程序时，不仅需要覆盖其多种行为；还需要*检查*结果是否符合预期。在本章中，我们介绍了一种技术，允许我们从一组给定的执行中挖掘函数规范，从而得到函数期望和提供的抽象和正式*描述*。

### 第五部分：特定领域模糊测试

这一部分讨论了针对多个特定领域的测试生成。对于所有这些领域，我们引入了*模糊器*，用于生成输入，以及*挖掘器*，用于分析输入结构。

#### 测试配置

程序的行为不仅受其数据控制。程序*配置*——即通过选项或配置文件设置的，控制程序在其（常规）输入数据上执行设置的设置——同样会影响行为，因此可以也应该进行测试。在本章中，我们探讨了如何系统地*测试*和*覆盖*软件配置。通过*自动推断配置选项*，我们可以直接应用这些技术，无需编写语法。最后，我们展示了如何系统地覆盖配置选项的*组合*，快速检测不希望出现的干扰。

#### 模糊测试 API

到目前为止，我们始终生成*系统输入*，即程序整体通过其输入通道获得的数据。然而，我们也可以生成直接进入单个函数的输入，从而在过程中获得灵活性和速度。在本章中，我们探讨了使用语法合成函数调用代码的使用，这允许你生成*非常高效地直接调用函数的程序代码*。

#### 雕刻单元测试

到目前为止，我们始终生成*系统输入*，即程序整体通过其输入通道获得的数据。如果我们只对测试一小组函数感兴趣，通过系统进行测试可能非常低效。本章介绍了一种称为*雕刻*的技术，它给定一个系统测试，自动提取一组*单元测试*，这些测试复制了系统测试期间看到的调用。关键思想是*记录*这些调用，以便我们可以在以后*回放*它们——整体或选择性地。此外，我们还探讨了如何从雕刻的单元测试中合成 API 语法；这意味着我们可以*合成 API 测试，而无需编写任何语法*。

#### 测试编译器

在本章中，我们将利用语法和基于语法的测试系统地生成*程序代码*——例如，测试编译器或解释器。不出所料，我们使用*Python*和*Python 解释器*作为我们的领域。

#### 测试 Web 应用程序

在本章中，我们探讨如何为图形用户界面（GUI）生成测试，特别是在 Web 界面。我们设置了一个（有漏洞的）Web 服务器，并演示了如何系统地探索其行为——首先使用手写的语法，然后使用从用户界面自动推断出的语法。我们还展示了如何对这些服务器进行系统性的攻击，特别是使用代码和 SQL 注入。

#### 测试图形用户界面

在本章中，我们探讨如何为图形用户界面（GUI）生成测试，从我们之前关于 Web 测试的示例中抽象出来。基于提取用户界面元素和激活它们的一般方法，我们的技术可以推广到任意图形用户界面，从富 Web 应用到移动应用，并通过表单和导航元素系统地探索用户界面。

### 第六部分：管理模糊测试

这一部分讨论了如何管理大规模的模糊测试。

#### 大规模模糊测试

在过去的章节中，我们总是关注仅在一台机器上持续几秒钟的模糊测试。然而，在现实世界中，模糊测试是在数十台甚至数千台机器上运行的；持续数小时、数天甚至数周；针对一个程序或数十个程序。在这种情况下，需要一个*基础设施*来*收集*单个模糊测试运行中的失败数据，并将这些数据*聚合*到一个中央存储库中。在本章中，我们将检查这样一个基础设施，即 Mozilla 的*FuzzManager*框架。

#### 何时停止模糊测试

在过去的章节中，我们讨论了几种模糊测试技术。知道*做什么*很重要，但知道何时*停止*做事情也同样重要。在本章中，我们将学习何时*停止模糊测试*——并使用一个突出的例子来说明这一点：在第二次世界大战中，纳粹德国海军使用的用于加密通信的*恩尼格玛*机器，以及 Alan Turing 和 I.J. Good 如何使用*模糊测试技术*来破解海军恩尼格玛机的密码。

### 附录

这一部分包含支持其他笔记本的笔记本和模块。

#### 学术原型设计

*这是 Andreas Zeller 在 ESEC/FSE 2022 会议上发表的“学术原型设计”教程的手稿。*

#### 使用 Python 进行原型设计

*这是 Andreas Zeller 在 TAIC PART 2020 会议上发表的“几分钟内编写有效的测试工具”主题演讲的手稿。*

#### 错误处理

这个笔记本中的代码有助于处理错误。通常，笔记本代码中的错误会导致代码执行停止；而笔记本代码中的无限循环会导致笔记本无限运行。这个笔记本提供了两个类来帮助解决这些问题。

#### 计时器

这个笔记本中的代码有助于测量时间。

#### 超时

本笔记本中的代码有助于在给定时间后中断执行。

#### 类图

这是一个简单的类图查看器。针对本书进行了定制。

#### 铁路图

本笔记本中的代码有助于绘制语法图。它是[Tab Atkins Jr.的优秀库](https://github.com/tabatkins/railroad-diagrams)的一个（略有定制）副本，不幸的是，它不是一个 Python 包。

#### 控制流图

本笔记本中的代码有助于获取 Python 函数的控制流图。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内文内容根据[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)授权。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，根据[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)授权。[最后更改：2024-07-01 12:05:22+02:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/00_Table_of_Contents.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用这篇作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[Fuzzing Book](https://www.fuzzingbook.org/html/00_Table_of_Contents.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[Fuzzing Book](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/00_Table_of_Contents.html`](https://www.fuzzingbook.org/html/00_Table_of_Contents.html). Retrieved 2024-07-01 12:05:22+02:00.

```py
@incollection{fuzzingbook2024:00_Table_of_Contents,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {The Fuzzing Book},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/00_Table_of_Contents.html}},
    note = {Retrieved 2024-07-01 12:05:22+02:00},
    url = {https://www.fuzzingbook.org/html/00_Table_of_Contents.html},
    urldate = {2024-07-01 12:05:22+02:00}
}

```
