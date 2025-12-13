# 第三部分：句法模糊测试

> 原文：[`www.fuzzingbook.org/html/03_Syntactical_Fuzzing.html`](http://www.fuzzingbook.org/html/03_Syntactical_Fuzzing.html)

本部分介绍了在*句法*级别的测试生成，即从语言结构中组合输入。

+   语法提供了程序合法输入的*规范*。通过语法指定输入可以实现非常系统和高效的测试生成，特别是对于复杂的输入格式。

+   高效语法模糊测试介绍了基于树的语法模糊测试算法，这些算法速度更快，允许对模糊输入的生产有更多的控制。

+   语法覆盖率允许系统地覆盖语法的元素，从而最大化多样性，不会遗漏任何单个元素。

+   解析输入展示了如何使用语法解析和分解一组有效的种子输入到它们对应的推导树。

+   概率语法模糊测试通过为单个扩展分配*概率*，使语法拥有更大的能力。

+   使用生成器进行模糊测试展示了如何通过*函数*扩展语法——这些代码片段在语法扩展期间执行，可以生成、检查或更改生成的元素。

+   灰盒语法模糊测试利用结构表示，使我们能够突变、交叉和重组它们的部分，以生成新的有效、略微改变的输入。

+   减少导致失败的输入介绍了自动将导致失败的输入减少和简化到最小，以减轻调试难度的技术。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容根据[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)授权。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，根据[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)授权。 [最后更改：2023-10-16 19:18:09+02:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/03_Syntactical_Fuzzing.ipynb) • 引用 • [印记](https://cispa.de/en/impressum)

## 如何引用本作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[第三部分：句法模糊测试](https://www.fuzzingbook.org/html/03_Syntactical_Fuzzing.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[The Fuzzing Book](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/03_Syntactical_Fuzzing.html`](https://www.fuzzingbook.org/html/03_Syntactical_Fuzzing.html). Retrieved 2023-10-16 19:18:09+02:00.

```py
@incollection{fuzzingbook2023:03_Syntactical_Fuzzing,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Part III: Syntactic Fuzzing},
    year = {2023},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/03_Syntactical_Fuzzing.html}},
    note = {Retrieved 2023-10-16 19:18:09+02:00},
    url = {https://www.fuzzingbook.org/html/03_Syntactical_Fuzzing.html},
    urldate = {2023-10-16 19:18:09+02:00}
}

```
