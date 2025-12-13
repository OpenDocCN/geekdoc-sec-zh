# 第五部分：特定领域模糊测试

> 原文：[`www.fuzzingbook.org/html/05_Domain-Specific_Fuzzing.html`](http://www.fuzzingbook.org/html/05_Domain-Specific_Fuzzing.html)

本部分讨论了多个特定领域的测试生成。对于所有这些领域，我们介绍了生成输入的 *模糊器* 以及分析输入结构的 *挖掘器*。

+   测试配置 系统性地 *测试* 和 *覆盖* 软件配置。通过 *自动推断配置选项*，我们可以直接应用这些技术，无需编写语法。

+   测试 API 展示了如何生成直接输入到单个函数的输入，从而在过程中获得灵活性和速度。

+   Carving 对系统测试进行测试，并自动提取一组 *单元测试*，这些测试复制了单元测试期间看到的调用。关键思想是 *记录* 这些调用，以便我们稍后可以 *回放* 它们——可以是全部或选择性地。

+   测试编译器 展示了如何使用 *语法* 和 *基于语法的测试* 系统地探索编译器或解释器的行为，以系统地生成程序代码。

+   测试 Web 应用程序 展示了如何系统地探索 Web 应用程序的行为——首先使用手写的语法，然后使用从用户界面自动推断出的语法。我们还展示了如何对这些服务器进行系统性的攻击，特别是使用代码和 SQL 注入。

+   测试图形用户界面 探索了如何为图形用户界面 (GUI) 生成测试，从丰富的 Web 应用程序推广到移动应用程序，并通过表单和导航元素系统地探索用户界面。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容根据 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可。内容的一部分源代码，以及用于格式化和显示该内容的源代码，根据 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 许可。最后更改日期：2024-11-09 17:07:29+01:00 [`github.com/uds-se/fuzzingbook/commits/master/notebooks/05_Domain-Specific_Fuzzing.ipynb`](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/05_Domain-Specific_Fuzzing.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

安德烈亚斯·泽勒，拉胡尔·戈皮纳特，马塞尔·博姆，戈登·弗朗西斯，克里斯蒂安·霍勒："[第五部分：特定领域模糊测试](https://www.fuzzingbook.org/html/05_Domain-Specific_Fuzzing.html)"。在安德烈亚斯·泽勒，拉胡尔·戈皮纳特，马塞尔·博姆，戈登·弗朗西斯和克里斯蒂安·霍勒的 "[模糊测试书](https://www.fuzzingbook.org/)" 中，[`www.fuzzingbook.org/html/05_Domain-Specific_Fuzzing.html`](https://www.fuzzingbook.org/html/05_Domain-Specific_Fuzzing.html)。检索日期：2024-11-09 17:07:29+01:00。

```py
@incollection{fuzzingbook2024:05_Domain-Specific_Fuzzing,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Part V: Domain-Specific Fuzzing},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/05_Domain-Specific_Fuzzing.html}},
    note = {Retrieved 2024-11-09 17:07:29+01:00},
    url = {https://www.fuzzingbook.org/html/05_Domain-Specific_Fuzzing.html},
    urldate = {2024-11-09 17:07:29+01:00}
}

```
