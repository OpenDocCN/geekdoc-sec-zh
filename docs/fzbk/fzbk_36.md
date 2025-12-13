# 第六部分：管理模糊测试

> 原文：[`www.fuzzingbook.org/html/06_Managing_Fuzzing.html`](http://www.fuzzingbook.org/html/06_Managing_Fuzzing.html)

本部分讨论了如何管理大规模的模糊测试。

+   大规模模糊测试 讨论了如何创建用于模糊测试的大规模基础设施，运行数百万次测试并管理其结果。

+   何时停止模糊测试 详细说明了如何估计何时足够的模糊测试已经足够——以及何时可以让你的计算机处理其他任务。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容受 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 的许可。作为内容一部分的源代码，以及用于格式化和显示该内容的源代码，受 [MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license) 的许可。 [最后修改：2020-10-13 15:12:25+02:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/06_Managing_Fuzzing.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用这篇作品

Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler: "[第六部分：管理模糊测试](https://www.fuzzingbook.org/html/06_Managing_Fuzzing.html)". In Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler, "[模糊测试书](https://www.fuzzingbook.org/)", [`www.fuzzingbook.org/html/06_Managing_Fuzzing.html`](https://www.fuzzingbook.org/html/06_Managing_Fuzzing.html). Retrieved 2020-10-13 15:12:25+02:00.

```py
@incollection{fuzzingbook2020:06_Managing_Fuzzing,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    booktitle = {The Fuzzing Book},
    title = {Part VI: Managing Fuzzing},
    year = {2020},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/html/06_Managing_Fuzzing.html}},
    note = {Retrieved 2020-10-13 15:12:25+02:00},
    url = {https://www.fuzzingbook.org/html/06_Managing_Fuzzing.html},
    urldate = {2020-10-13 15:12:25+02:00}
}

```
