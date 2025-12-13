# 第四部分：语义模糊测试

> 原文：[`www.fuzzingbook.org/html/04_Semantical_Fuzzing.html`](http://www.fuzzingbook.org/html/04_Semantical_Fuzzing.html)

本部分介绍了考虑输入的*语义*的测试生成技术，特别是处理输入的程序的行为。

+   带有约束的模糊测试向语法中添加*语义约束*。通过自动解决这些约束，我们可以生成既在语法上又在语义上有效的输入。

+   语法挖掘展示了如何通过分析输入的各个部分是如何被处理的来从程序中提取输入语法。这些生成的语法可以直接用于模糊测试。

+   跟踪信息流展示了如何在整个程序中跟踪输入，以便发现信息泄露并进一步改进分析技术。

+   并发模糊测试分析程序代码以解决程序中的*路径约束*，以覆盖难以到达的分支和行为。

+   符号模糊测试的工作原理类似于并发模糊测试，但根本不需要任何执行。

+   挖掘函数规范从程序执行中提取类型信息以及前置条件和后置条件——这些信息对于程序分析、测试和验证非常有用。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT License](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。最后修改时间：2023-01-07 15:48:35+01:00。引用 [版权信息](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[第四部分：语义模糊测试](https://www.fuzzingbook.org/html/04_Semantical_Fuzzing.html)"。在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 的"[模糊测试书](https://www.fuzzingbook.org/)"中。[`www.fuzzingbook.org/html/04_Semantical_Fuzzing.html`](https://www.fuzzingbook.org/html/04_Semantical_Fuzzing.html)。检索时间：2023-01-07 15:48:35+01:00。

```py
@incollection{fuzzingbook2023:04_Semantical_Fuzzing,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Part IV: Semantic Fuzzing},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/04_Semantical_Fuzzing.html}},
    note = {Retrieved 2023-01-07 15:48:35+01:00},
    url = {https://www.fuzzingbook.org/html/04_Semantical_Fuzzing.html},
    urldate = {2023-01-07 15:48:35+01:00}
}

```
