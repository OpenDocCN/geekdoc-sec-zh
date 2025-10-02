# 他们是怎么做的:传播的方法

对于一个病毒或蠕虫想要成功，它需要一个执行和传播的方法。电子邮件病毒通常在鼠标点击后执行，然后通过 您的联系人列表来发送带有恶意软件的的邮件进行传播。网络蠕虫利用远程利用漏洞危害机器和并且通过连接到其他 存在漏洞的主机进行传播。除传播之外，蠕虫病毒还是高度多样化，包括创造 DDoS 僵尸网络，垃圾邮件僵尸，或远 程键盘监控的能力。XSS 蠕虫与其他形式的恶意软件相似，但以自己独特的方式执行和传播。使用一个网站来存放恶 意代码，XSS 蠕虫和病毒通过控制 Web 浏览器，使得它复制恶意软件到网络上的其他地方去感染别人来进行传播。例 如，一个含有恶意软件的博客评论，可以使用访问者的浏览器发布额外的感染性的博客评论。XSS 蠕虫病毒可能会使 得浏览器进行发送电子邮件，转账，删除/修改数据，入侵其他网站，下载非法内容，以及许多其他形式的恶意活动。 用最简单的方式去理解，就是如果没有适当的防御，在网站上的任何功能都可以在未经用户许可的情况下运行。

在最后一节中，我们将重点放在 XSS 漏洞本身，以及用户可以怎么样被利用。现在，我们来看 XSS 恶意软件是如 何可以进行远程通信。XSS 漏洞利用代码，通常是 HTML / JavaScript，使用三种方式使得浏览器发送远程 HTTP 请求的 浏览器：嵌入式的 HTML 标签，JavaScript DOM 的对象，XMLHTTPRequest（XHR）。

另外，请记住，如果你碰巧登录到远程网站，您的浏览器是被迫作出的身份验证的请求的。关于 XSS 恶意软件的 传播方法与传统互联网病毒的明显差异将会进行简要说明。

## 嵌入式 HTML 标签

一些 HTML 标签具有在页面加载时会自动发起 Web 浏览器的 HTTP 请求的属性。有一个例子是 IMG（图像）的 SRC 属性。SRC 属性用于指定在 Web 页面中显示的图像文件的 URL 地址。当你的浏览器载入带有 IMG 标签的网页， 图像会自动被请求，并在浏览器中显示。但是，SRC 属性也可以被用于任何 Web 服务器的其他 URL，不仅仅是包含图 像的。

例如，如果我们进行了谷歌搜索“WhiteHat Security”我们最终得到了下面的 URL：

```
http://www.google.com/search?hl=en&q=whitehat+security&btnG=Google+Search 
```

这个 URL 可以很容易地取代 IMG 内的 SRC 属性的值，从而迫使您的 Web 浏览器中执行相同的谷歌搜索。

```
<img src=”http://www.google.com/search?hl=en&q=whitehat+security&btnG=Google+Search”> 
```

显然使得 Web 浏览器发送一个谷歌搜索请求是没有什么危害性的。然而，URL 构建的同样的过程可以用来使得 Web 浏览器自动进行银行账户资金的汇款，发表煽动性言论，甚至入侵网站。这一点可以说明，这是一个迫使一个 Web 浏览器连接到其他网站，使的 XSS 蠕虫病毒传播的机制。

额外的源码例子可见“嵌入式 HTML 标签”的附录部分。

## JavaScript 和文档对象模型

JavaScript 被用于给网站访问者一个丰富的、交互式的体验。这些网页更接近于一个软件应用程序，而不是一个 静态的 HTML 文档。我们常会看到 JavaScript 被用于进行图像的翻转，动态表单输入检查，弹出对话框，下拉菜单， 拖动和拖放等。JavaScript 有接近完全的对网站上的每一个对象，包括图像，cookies，窗口，框架和文本内容的访问。 这些对象中的每一个都是文档对象模型（DOM）的一部分。

DOM 提供了一系列 JavaScript 读取和操作的应用程序编程接口（API）。类似嵌入式 HTML 标签的功能，JavaScript 可以操纵 DOM 对象自动发起 Web 浏览器的 HTTP 请求。图像和窗口的源 URL 可以被重新指定为其他 URL 的。

在上一节中，我们可以使用 JavaScript 来改变图像的 DOM 对象 SRC 进行谷歌搜索“WhiteHat Security”。

```
img[0].src = http://www.google.com/search?hl=en&q=whitehat+security&btnG=Google+Search; 
```

正如上一节中，迫使 Web 浏览器连接到其他网站发送一个谷歌搜索请求是一种无害的例子。但这也说明了 XSS 恶意软件传播的另一种方法。

额外的源码例子可见“JavaScript DOM 对象”的附录部分。

## XmlHttpRequest (XHR)

2005 年 2 月，Jesse James Garrett 创造一个被称为“异步 JavaScript 和 XML”或“AJAX”的 Web 编程术语 10。AJAX 定义了使网站内容进行更新而无需重新加载的技术的集合。今天，许多流行的网站，包括 Gmail 和谷歌地图使用 AJAX 的丰富的功能。中央的底层技术是一种称为 XMLHttpRequest1（1 XHR）的 JavaScript API，支持 IE 浏览器，Mozilla，Firefox，Safari，Camin，Opera 和许多其他浏览器的。XHR 提供了一个灵活的机制来发送 HTTP 请求。有 XHR 后，使用 HTML 技 巧或操作 DOM 对象是没有必要的。任意请求可以在后台发送。

源码例子可见“XmlHTTPRequest”的附录部分。