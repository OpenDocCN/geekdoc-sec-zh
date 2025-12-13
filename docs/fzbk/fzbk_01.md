# 模糊测试之书

> 原文：[`www.fuzzingbook.org/`](https://www.fuzzingbook.org/)

### 生成软件测试的工具和技术

**由安德烈亚斯·泽勒、拉胡尔·戈皮纳特、马塞尔·博 hme、戈登·弗莱泽和克里斯蒂安·霍勒编写**

# 关于本书

**欢迎来到《模糊测试书》！** 软件存在缺陷，捕捉缺陷可能需要大量努力。本书通过**自动化**软件测试来解决这个问题，特别是通过**自动生成测试**。近年来，新型技术的开发导致了测试生成和软件测试的显著改进。这些技术现在已经足够成熟，可以汇编成书——甚至包含可执行代码。

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import YouTubeVideo
YouTubeVideo("w4u5gCgPlmg") 
```

## 适用于纸张、屏幕和键盘的教科书

你可以用四种方式使用这本书：

+   你可以在**浏览器中阅读章节**。查看菜单上方的章节列表，或立即开始阅读测试介绍或模糊测试介绍。所有代码均可下载。

+   你可以**作为 Jupyter 笔记本与章节进行交互**（测试版）。这允许你在浏览器中编辑和扩展代码，**实时实验**。只需在每个章节的顶部选择“资源→作为笔记本编辑”。[尝试与模糊测试的介绍进行交互。](https://mybinder.org/v2/gh/uds-se/fuzzingbook/HEAD?labpath=docs/notebooks/Fuzzer.ipynb)

+   你可以在**自己的项目中使用代码**。你可以将代码作为 Python 程序下载；只需选择“资源→下载代码”以获取某一章节或“资源→所有代码”以获取所有章节。这些代码文件可以执行，产生（希望）与笔记本相同的结果。甚至更简单：安装 fuzzingbook Python 包。

+   你可以将**章节作为幻灯片进行展示**。这允许你在讲座中展示材料。只需在每个章节的顶部选择“资源→查看幻灯片”。[尝试查看模糊测试的介绍幻灯片。](https://www.fuzzingbook.org/slides/Fuzzer.slides.html)

## 本书面向的对象

这本书被设计为一本软件测试或安全测试课程的**教科书**；作为软件测试、安全测试或软件工程课程的**补充材料**；以及作为软件开发者的**资源**。我们涵盖了随机模糊测试、基于变异的模糊测试、基于语法的测试生成、符号测试等，所有技术都用你可以亲自尝试的代码示例进行了说明。

## 新闻

本书是**正在进行的作品**。所有计划中的章节都已发布，但我们仍在通过[小版本和大版本发布]对文本和代码进行改进。[关注我们在 Mastodon 上的更新](https://mastodon.social/@TheFuzzingBook)。

<link rel="stylesheet" href="/html/mastodon-timeline.css">

## 关于作者

本书由 *Andreas Zeller, Rahul Gopinath, Marcel Böhme, Gordon Fraser 和 Christian Holler* 撰写。我们都是软件测试和测试生成领域的长期专家；我们编写或为地球上一些最重要的测试生成器和模糊器做出了贡献。例如，如果您正在 Firefox、Chrome 或 Edge 网络浏览器中阅读此内容，您可以在我们的帮助下安全地这样做，因为 *本书中列出的技术已经在他们的 JavaScript 解释器中发现了超过 2,600 个错误*。我们很高兴分享我们的专业知识，并使其对公众可访问。

## 常见问题

### 故障排除

#### 为什么启动交互式笔记本需要这么长时间？

交互式笔记本使用 [mybinder.org](https://mybinder.org) 服务，该服务在其自己的服务器上运行笔记本。通过 mybinder.org 启动 Jupyter 通常需要大约 30 秒，具体取决于您的互联网连接。然而，如果您是在书籍更新后首次调用 binder，binder 将重新创建其环境，这可能需要几分钟。偶尔重新加载页面。

#### 交互式笔记本不工作！

mybinder.org 对存储库施加了 [100 个并发用户的限制](https://mybinder.readthedocs.io/en/latest/user-guidelines.html)。此外，如 [mybinder.org 状态和可靠性页面](https://mybinder.readthedocs.io/en/latest/reliability.html)中所述，

> 由于 mybinder.org 是一个研究试点项目，该项目的主要目标是了解使用模式和负载，以便未来项目发展。虽然我们努力实现网站可靠性和可用性，但我们希望用户了解此服务的目的是研究，我们不保证其在关键任务使用中的性能。

有 mybinder.org 的替代品；见下文。

#### 我有交互式笔记本的替代品吗？

如果 mybinder.org 无法工作或不符合您的需求，您有许多替代方案：

1.  **下载 Python 代码**（使用顶部菜单）并在您喜欢的环境中编辑和运行它。这很容易做到，并且不需要很多资源。

1.  **下载 Jupyter 笔记本**（使用顶部菜单）并在 Jupyter 中打开。以下是 [如何在您的机器上安装 jupyter notebook](https://www.dataquest.io/blog/jupyter-notebook-tutorial/) 的方法。

有关详细信息，请参阅我们关于 在您的程序中使用 Debuggingbook 代码 的文章。

作为另一种替代方案，您还可以 **使用我们的 Docker 镜像**（实验性）。[安装 Docker](https://docs.docker.com/get-docker/) 然后运行

```py
 $ docker pull zeller24/fuzzingbook
    $ docker run -it --rm -p 8888:8888 zeller24/fuzzingbook
```

然后在您的网络浏览器中，打开控制台输出中给出的 URL (`http://127.0.0.1/...` 或 `http://localhost/...`)。这应该会为您提供与 mybinder.org 相同的环境。

如果您想创建自己的 Docker 镜像，请以我们的 [Dockerfile](https://github.com/uds-se/fuzzingbook/blob/master/binder/Dockerfile) 作为起点。

#### 我能在 Windows 机器上运行代码吗？

我们尽量使代码尽可能通用，但偶尔，当我们与操作系统交互时，我们假设是一个类 Unix 环境（因为那是 Binder 提供的）。要在您的 Windows 机器上运行这些示例，您可以安装 Linux 子系统或 Linux 虚拟机。

#### 您不能运行自己的专用云服务吗？

技术上，是的；但这将花费金钱和精力，而我们宁愿把这些精力花在本书上。如果您想为公众托管[JupyterHub](http://jupyter.org/hub)或[BinderHub](https://github.com/jupyterhub/binderhub)实例，请*这样做*并让我们知道。

### 内容

#### 我可以在自己的程序中使用您的代码吗？

是的！有关详细信息，请参阅安装说明。

#### 哪些内容已经出现？

请参阅发布说明以获取详细信息。

#### 我如何引用您的工作？

感谢您引用我们的工作！一旦本书完成，您将能够以传统方式引用它。在此期间，只需点击每个章节网页底部的“引用”按钮即可获取引用条目。

#### 您可以引用我的论文吗？并且可能写一个关于它的章节？

我们总是很高兴收到建议！如果我们遗漏了重要的参考文献，我们当然会添加。如果您想涵盖特定的材料，最好的方法是*自己编写笔记本*；请参阅我们的作者指南以获取编码和写作的说明。然后我们可以引用它，甚至托管它。

### 教学和课程作业

#### 我可以在我的课程中使用您的材料吗？

当然可以！只需尊重[许可证](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md)（包括归属和共享）。如果您想将材料用于商业目的，请联系我们。

#### 我可以扩展或改编您的材料吗？

是的！再次，请参阅[许可证](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md)以获取详细信息。

#### 我如何基于本书运行课程？

我们已经在各种课程中成功使用了这些材料。

+   最初，我们在讲座中使用幻灯片和代码，并进行*现场编码*来展示技术是如何工作的。

+   现在，本书的目标是做到完全自包含；也就是说，它应该在没有额外支持的情况下也能工作。因此，我们现在在*翻转课堂*环境中向学生提供完整的章节，让学生在空闲时间完成笔记本。我们会在教室里讨论过去笔记本的经验，并讨论未来的笔记本。

+   我们让学生从书中做练习或进行更大的（模糊）项目。我们还有使用本书作为其研究基础的学生；实际上，用 Python 原型化 Python 非常容易。

在运行课程时，不要依赖 mybinder.org – 它不会为更大的学生群体提供足够多的资源。相反，安装并运行您自己的中心。

#### 我可以专注于特定的子集吗？

我们为不同的受众编译了多本书的导览。我们的网站地图列出了各个章节之间的依赖关系。

#### 我如何扩展或修改您的幻灯片？

下载 Jupyter Notebooks（使用顶部菜单），并在您方便的时候修改笔记本（见上文），包括“幻灯片类型”设置。然后，

1.  从 Jupyter Notebook 下载幻灯片；或者

1.  使用 RISE 扩展程序([说明](http://www.blog.pythonlibrary.org/2018/09/25/creating-presentations-with-jupyter-notebook/))直接从 Jupyter notebook 中展示您的幻灯片。

#### 您提供材料的 PDF 版本吗？

到目前为止，我们不提供 PDF 版本的支援。在完成书籍后，我们将制作 PDF 和印刷版本。

### 其他问题

#### 我有一个问题、评论或建议。我该怎么做？

您可以在 Mastodon 上的[@TheFuzzingBook@mastodon.social](https://mastodon.social/@TheFuzzingBook)上发布，让读者社区参与讨论。如果您希望修复的错误，请在[开发页面](https://github.com/uds-se/fuzzingbook/issues)上报告问题。

#### 我两周前报告了一个问题。何时会得到解决？

我们按照以下顺序优先处理问题：

1.  在 fuzzingbook.org 上发布的代码中的错误

1.  在 fuzzingbook.org 上发布的文本中的错误

1.  编写缺失的章节

1.  尚未发布的代码或文本中的问题

1.  与开发或构建相关的问题

1.  标记为“beta”的事项

1.  其他所有事项

#### 我如何自己解决问题？

我们很高兴您提出这个问题。[开发页面](https://github.com/uds-se/fuzzingbook/)有所有源代码和一些补充材料。欢迎提交修复问题的拉取请求。

#### 我如何贡献？

再次，我们很高兴您在这里！我们很高兴接受

+   **代码修复和改进**。请将任何代码放在 MIT 许可下，这样我们就可以轻松地将其包含在内。

+   **关于特定主题的额外文本、章节和笔记本**。我们计划为第三方贡献设置一个专门的文件夹。

请参阅我们的作者指南，了解编码和写作的说明。

![Creative Commons License](img/2f3faa36146c6fb38bbab67add09aa5f.png) 本项目的内容根据[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)授权。内容的一部分源代码，以及用于格式化和显示该内容的源代码，根据[MIT 许可协议](https://github.com/uds-se/fuzzingbook/blob/master/LICENSE.md#mit-license)授权。[最后更改：2024-07-01 16:50:18+02:00](https://github.com/uds-se/fuzzingbook/commits/master/notebooks/index.ipynb) • 引用 • [版权信息](https://cispa.de/en/impressum)

## 如何引用这篇作品？

安德烈亚斯·泽勒、拉胡尔·戈皮纳特、马塞尔·博 hme、戈登·弗拉瑟和克里斯蒂安·霍勒："[模糊测试书](https://www.fuzzingbook.org/)"。检索日期：2024-07-01 16:50:18+02:00。

```py
@book{fuzzingbook2024,
    author = {Andreas Zeller and Rahul Gopinath and Marcel B{\"o}hme and Gordon Fraser and Christian Holler},
    title = {The Fuzzing Book},
    year = {2024},
    publisher = {CISPA Helmholtz Center for Information Security},
    howpublished = {\url{https://www.fuzzingbook.org/}},
    note = {Retrieved 2024-07-01 16:50:18+02:00},
    url = {https://www.fuzzingbook.org/},
    urldate = {2024-07-01 16:50:18+02:00}
}

```
