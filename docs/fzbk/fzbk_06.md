# 第二部分：词汇模糊测试

> 原文：[`www.fuzzingbook.org/html/02_Lexical_Fuzzing.html`](http://www.fuzzingbook.org/html/02_Lexical_Fuzzing.html)

这一部分介绍了在**词汇**级别的测试生成，即组成字符序列。

+   模糊测试：使用随机输入破坏事物 从最简单的测试生成技术之一开始：模糊测试将一串随机字符输入到程序中，希望揭示失败。

+   在获取覆盖率中，我们通过评估这些测试的**代码覆盖率**来衡量这些测试的有效性——也就是说，在测试运行期间测量程序的实际执行部分。对于试图覆盖尽可能多代码的测试生成器来说，测量这种覆盖率也是至关重要的。

+   基于变异的模糊测试 展示了如何**变异**现有输入以测试新的行为。我们展示了如何创建这样的变异，以及如何引导它们向尚未发现的代码，应用了流行的 AFL 模糊测试器的核心概念。

+   灰盒模糊测试进一步扩展了输入变异的概念，使用统计估计器来引导测试生成向可能存在的错误方向。

+   基于搜索的模糊测试将引导的概念进一步发展，引入基于搜索的算法来系统地生成程序的测试数据。

+   变异分析 将合成的缺陷（变异）种入程序代码中，以检查测试是否能够找到它们。如果测试没有找到变异，它们很可能也不会找到真正的错误。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)许可。 [最后更改：2023-01-07 15:27:27+01:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/02_Lexical_Fuzzing.ipynb) • 引用 • [ imprint](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler: "[第二部分：词汇模糊测试](https://www.fuzzingbook.org/html/02_Lexical_Fuzzing.html)"。在 Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, 和 Christian Holler 编著的 "[模糊测试书](https://www.fuzzingbook.org/)" 中，[`www.fuzzingbook.org/html/02_Lexical_Fuzzing.html`](https://www.fuzzingbook.org/html/02_Lexical_Fuzzing.html)。检索日期：2023-01-07 15:27:27+01:00。

```py
@incollection{fuzzingbook2023:02_Lexical_Fuzzing,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Part II: Lexical Fuzzing},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/02_Lexical_Fuzzing.html}},
    note = {Retrieved 2023-01-07 15:27:27+01:00},
    url = {https://www.fuzzingbook.org/html/02_Lexical_Fuzzing.html},
    urldate = {2023-01-07 15:27:27+01:00}
}

```
