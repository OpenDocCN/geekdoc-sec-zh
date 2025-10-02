## python 安全编程教程

> 原文：[Python Tutorials](http://www.primalsecurity.net/tutorials/python-tutorials/)
> 
> 译者：[smartFlash](https://github.com/smartFlash)
> 
> 来源：[pySecurity](https://github.com/smartFlash/pySecurity)
> 
> 协议：[MIT License](https://github.com/smartFlash/pySecurity/blob/master/LICENSE)

```py
~# python
>>> import urllib
>>> from bs4 import BeautifulSoup
>>> url = urllib.urlopen("http://www.primalsecurity.net")
>>> output = BeautifulSoup(url.read(), 'lxml')
>>> output.title
<title>Primal Security Podcast</title>
>>> 
```

这是一套 python 系列教程，学习本套教程不需要你有任何编程背景。教程由最简单的 hello world 到信息安全应用实例。逐个难点击破:

0x0 – [入门](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x0.md)

0x0 – [入门 Pt.2](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x02.md)

0×1 – [端口扫描](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x1.md)

0x2 – [反向 shell](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x2.md)

0x3 – [模糊测试](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x3.md)

0x4 – [Python 转 exe](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x4.md)

0x5 – [Web 请求](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x5.md)

0x6 – [爬虫](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x6.md)

0x7 – [Web 扫描和利用](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x7.md)

0x8 – [Whois 查询](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x8.md)

0x9 – [系统命令调用](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x9.md)

0xA – [Python 版的 Metasploit](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x10.md)

0xB – [伪终端](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x11.md)

0xC – [exp 编写](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0xc.md)

用例 1: [CVE-2014-6271](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x13.md)

用例 2: [CVE-2012-1823](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x14.md)

用例 3: [CVE-2012-3152](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x15.md)

用例 4: [CVE-2014-3704](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x16.md)