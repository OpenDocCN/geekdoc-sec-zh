# Web 请求

## Web 请求

这篇文章将会演示如何使用 python 进行 web 请求，这里需要几个 python 的模块来使得我们能够更容易创建和解析 web 请求与响应(httplib,Mechanize,Beautiful Soup 和 urllib/urllib2),安装这些模块并且检查这些功能函数.

**创建一个 Web 请求**

下面有个简短的例子，展示了使用 python 的 SimpleHTTPServer 创建一个本地 web 服务器，并且建立一个请求:

![建立 web 请求](img/0x501.png)

**解析 HTML**

现在我们已经使用 Python 建立了一个 web 请求，现在我们要找一个模块来解析 HTML 文件。而前面我们提到了 BeautifulSoup 模块能够帮助我们基于 HTML 标签解析 HTML。下面有一个例子，可以帮助你理解如何去解析 HTML 文件:

![BeautifulSoup](img/0x502.png)

BeautifulSoup 对于帮助我们解析 HTML 非常强大，例如你可以使用 BeautifulSoup 内部的函数"find_all"去查找你想要解析的内容。例如:"iframes = parsed.find_all(‘iframe’)".

**实战写一个应用**

大家都知道，我们可以使用大量的查询去获取更多的 web 资源，在这里，Python 脚本能够自动帮你完成你的查询并且获取到你想要的资源.我常常使用 iplist.net 去反查域名，看看到底有多少个域名指向了一个 IP.

当你开始写脚本的时候，你首先得先考虑两件事情:

1、请求 URL 的连接结构 ２、你想要什么信息？你可以通过 HTML 标签定位到你想要的数据部分，当然为更加准确，你也可以使用正则式去匹配.

iplist.net 的结构相对简单"[`iplist.net/`](http://iplist.net/)<ip class="calibre11">/",因为我们能够相对比较容易的从一个文件里面使用循环把所有的 IP 都读取出来，下一步就是查看源代码，看看你最想要的是那个部分的内容，在这个例子中我们可以看到 HTML 标签 header 里面有一行`<h2>domain_name</h2>`.</ip>

那么我们就使用 BeautifulSoup 去分离这个页面的源码，下面是执行脚本的过程，我们这里只提取域名并且打印到 STDOUT:

![IPlist](img/0x503.png)

FireBug 是一个分析源代码的工具，很强加，下面你就可以看到高亮的代码就是我们需要的信息;

![Firebug](img/0x504.png)

说到这里，这篇文章就已经就已经完成了，对于 web 请求你可以去分析 python 究竟是如何去请求的，并且如何去提取自己有用的信息并且打印到 STDOUT.这里有一个解析 iplist.net 比较复杂的[脚本](https://github.com/primalsecn/python_code/blob/master/iplist.py)，里面有非常完整的解析原理。大家可以看看