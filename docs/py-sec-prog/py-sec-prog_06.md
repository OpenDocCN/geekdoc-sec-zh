# Python 转 exe

## Python 转 exe

**使用 PyInstaller 生成可以执行程序**

这一章是教大家如何把自己的 python 脚本编译成 windows 下可执行文件，它可以让你的 python 脚本跨平台去运行，并且不需要去安装 python 解释器。首先我们需要下载依赖包,cygwin(或者其他的工具也可以，这里我们使用 Pywin).

Linux: sudo apt-get install python2.7 build-essential python-dev zlib1g-dev upx Windows: [`www.activestate.com/activepython`](http://www.activestate.com/activepython) (fully packaged installer file)

安装 [Pywin32](http://sourceforge.net/projects/pywin32/), [Setuptools](https://pypi.python.org/pypi/setuptools#downloads), [PyInstaller](http://www.pyinstaller.org/)

**安装完成之后**

下一步我们就运行 python 命令生成可执行文件:

```py
python pyinstaller.py -onefile <scriptName> 
```

执行上面的命令之后，导入依赖文件并且生成一个新的文件，这个文件里面包含了三个文件<scriptname class="calibre11">.txt,<scriptname class="calibre11">.spec 和<scriptname class="calibre11">.exe 文件，其中.txt 与.spec 可以删除掉，而.exe 的文件就是你需要的执行程序.</scriptname></scriptname></scriptname>

**完整的封装执行程序**

Python 脚本现在已经被编译成了 windows PE 文件，并且不需要 Python 解释器就能够在 windows 下面独立运行，这可以让你更轻松的把脚本迁移到 windows 上面而且不用担心依赖包缺失的问题.

一个简单的脚本:

```py
#!/usr/bin/python

import os

os.system("echo Hello World!") 
```

现在我们把上面这个脚本编译成为一个可以执行的文件:

```py
c:\PathToPython\python.exe pyinstaller.py --onefile helloWorld.py

> helloWorld.exe
Hello World! 
```

如果你想更详细的了解这个过程，可以参考[BACK TO THE SOURCE CODE – Forward/Reverse Engineering Python Malware](http://www.primalsecurity.net/back-to-the-source-code-forwardingreverse-engineering-python-malware/)

把你的 python 脚本编译成一个可以在 windows 上面可以执行的可执行程序是很有用的，因为它不需要你安装 python 解释器还有依赖包

大家可以尝试一下[0x2](https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x2.md)中的例子，把那个脚本编译成可执行程序。