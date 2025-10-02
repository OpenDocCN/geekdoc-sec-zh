# 一、搭建开发环境

# 1 搭建开发环境

在即将开始令人兴奋的 Python Hack 之前，让我们先花一 点点事件准备好自己的工具。相信我这样做是值得的，它会让你 玩的更快乐。

这章我们会简单的讲解，Python2.5 的安装，Eclipse 配置，以及如何编写 C 兼容的 Python 代码。

# 1.1 操作系统准备

## 1.1 操作系统准备

就逆向的趣味性而言，Windows 是最好的目标。无数的工具和广泛的使用人群，使得代 码开发和 Crack 都变得更容易，所以本书的大部分代码都基于 Windows(任何你能搞的到的 Windows 版本)。

少部分例子也能运行在 32 位的 Linux 上。无论是安装在 VMware(VMware 提供免费版 本,不同为版权担心)上还是实机上，都行。Linux 版本众多，本书推荐基于 Red Hat 的发布平 台:Fedora Core 7 or Centos 5。

免费的 VMWARE 镜像

VMware 在网站上提供了免费的版本。这些虚拟机用于逆工程，漏洞分析，或者任何 程序的调试，同时和主机完全独立开来。 主程序下载链接:[`www.vmware.com/appliances/`](http://www.vmware.com/appliances/)， Pyayer 程序下载链接:[`www.vmware.com/products/player/`](http://www.vmware.com/products/player/)。

# 1.2 获取和安装 Python2.5

## 1.2 获取和安装 Python2.5

Linuxer 可以跳过这个步骤，大部分 Linux 都内置了 Python。Windows 下可以通过独立 的安装包进行安装。

### 1.2.1 在 Windows 上安装 Python

Windows 的安装版本可以从 Python 主页上下载 http:// python.org/ftp/python/2.5.1/python-2.5.1.msi。双击，一步一步的按指示安装就行。在默认的 主目录 C:/Python25/下，安装了 python.exe 和默认的库。

提示 建议大家安装 Immunity 调试器，其包含了很多必须的附加程序，其中就有 Python 2.5 。 在 后 面 的 章 节 中 ， 我 们 也 会 使 用 到 Immunity 。 下 载 页 面 [`debugger.immunityinc.com`](http://debugger.immunityinc.com)要用代理还要填写些资料。

### 1.2.2 在 Linux 上安装 Python

如果需要在 Linux 上手工安装 Python 的话，可以按如下的步骤进行。这里使用 Red Hat 的衍生版，并且这个过程使用 root 权限。 第一步，下载 Python 2.5 源码并解压:

```py
# cd /usr/local/
# wget http://python.org/ftp/python/2.5.1/Python-2.5.1.tgz
# tar –zxvf Python-2.5.1.tgz
# mv Python-2.5.1 Python25
# cd Python25 
```

代码解压到/usr/local/Python25 之后，就要编译安装了:

```py
# ./configure –-prefix=/usr/local/Python25
# make && make install
# pwd
/usr/local/Python25
# python
Python 2.5.1 (r251:54863, Mar 14 2012, 07:39:18)
[GCC 3.4.6 20060404 (Red Hat 3.4.6-8)] on Linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

现在我们就拥有了一个交互式的 Python Shell，能够自由的操作 Python 和 Python 库了。 输入个语句测试下:

```py
>>> print "Hello World!" Hello World!
>>> exit()
# 
```

很好！一切工作正常。为了让系统能够找到 Python 计时器的路径，需要编辑/root/.bashrc 文件(/用户名/.bashrc)。我个人比较喜欢 nano,不过你可以使用你喜欢编辑器(个人推荐 vim 嘿 嘿)。打开/root/.bashrc，在文件底部加入以下代码。

```py
export PATH=/usr/local/Python25/:$PATH 
```

这样每次执行 python 命令的时候，就不用输入完整的 python 路径了。下次用 root 登录 的时候，就在任何 shell 下输入 python 就能得到一个交互式的 Python Shell 了。

为了方便的开发代码，下面让我们配置自己 IDE(ntegrated development environment )。 (我的开发环境如下:ActivePython,UliPad 或者 Script.NET，ipython 或者 bpython。调试，自动 提示，参数说明全都有了。)

# 1.3 配置 Eclipse 和 PyDev

## 1.3 配置 Eclipse 和 PyDev

为了快速的的开发调试 Python 程序，就必须要使用一个稳定的 IDE 平台。这里作者推 荐的时候 Eclipse(跨平台的 IDE)和 PyDev。Eclipse 以其强大的可定制性而出名。下面让我们 看看和安装和配置它们：

1.  从 [`www.eclipse.org/downloads/`](http://www.eclipse.org/downloads/)下载压缩包

2.  解压到 C:\Eclipse

3.  运行 C:\Eclipse\eclipse.exe

4.  第一次运行，会询问在哪里设置工作区的主目录；使用默认的就行,将 Use this as default and do not ask again 勾上，点击 OK。

5.  Eclipse 安装好以后，选择 Help Software Updates Find and Install

6.  选择 Search for new features to install 然后点击 Next。

7.  点击 New Remote Site。

8.  在 Name 后面填上 PyDev Update，在 URl 后面填上 [`pydev.sourceforge.net/updates/`](http://pydev.sourceforge.net/updates/)， 点击 OK 确认，接着点击 Finish，Eclipse 会自动升级 PyDev。

9.  过一会儿，更新窗口就会出现，找到顶端的 PyDev Update，选上 PyDev，单击 Next 继 续下一步。

10.  阅读 PyDev 协议，如果同意，在 I accept the terms in the licens agreement 选上。

11.  单击 Next，和 Finish。Eclipse 开始安装 PyDe 扩展，全部完成后，单击 Install All。 12 最后一步，在 PyDev 安装好之后，单击 Yes，Eclipse 会重新启动并加载 PyDev。

使用如下步骤配置 Eclipse，以确保 PyDev 能正确的调用 Python 解释器执行脚本。

1.  Eclipese 驱动后，选择 Window Preferences

2.  扩展 PyDev，选择 Interpreter – Python。

3.  在对话框顶端的 Python Interpreters 中点击 New。

4.  浏览到 C:\Python25\python.exe，然后点击 Open。

5.  下一个对话框将会列出 Python 中已经安装了的库。

6.  再次点击 OK 完成安装。

在开始编码前，需要创建一个 PyDev 工程。本书的所有代码都可以在这个工程中打开。

1.  依次选择 File-->New-->Project。

2.  展开 PyDev 选择 PyDev Project，点击 Next 继续。

3.  将工程命名为 Gray Hat Python. 点击 Finish。

Eclipse 窗口自动更新之后，会看到 Gray Hat Python 工程出现在屏幕左上角。现在右击 sec 文件夹，选择 New-->PyDev Module。在 Name 字段输入 chapter1-test，点击 Finish。就会 看到，工程面板被更新了，chapter1-test.py 被加到列表中。

在 Eclipse 中运行 Python 脚本，重要单击工具栏上的 Run As(由绿圈包围的白色箭头)按 钮就行了。要运行以前的脚本，可以使用快捷键 CTRL-F11。脚本的输出会显示在 Eclipse 底端的 Console 面板。现在万事俱备只欠代码。

### 1.3.1 hacker 们的朋友:ctypes

ctypes 是强大的，强大到本书以后介绍的几乎所有库都要基于此。使用它我们就能够调 用动态链接库中函数，同时创建各种复杂的 C 数据类型和底层操作函数。毫无疑问，ctypes 就是本书的基础。

### 1.3.2 使用动态链接库

使用 ctypes 的第一步就是明白如何解析和访问动态链接库中的函数。一个 dynamically linked library(被动态连接的库)其实就是一个二进制文件，不过一般自己不运行，而是由别 的程序调用执行。在 Windows 上叫做 dynamic link libraries (DLL)动态链接库,在 Linux 上叫 做 shared objects (SO)共享库。无论什么平台，这些库中的函数都必须通过导出的名字调用， 之后再在内存中找出真正的地址。所以正常情况下，要调用函数，都必须先解析出函数地址， 不过 ctypes 替我们完成了这一步。

ctypes 提供了三种方法调用动态链接库:cdll(), windll(), 和 oledll()。它们的不同之处就在 于，函数的调用方法和返回值。cdll() 加载的库，其导出的函数必须使用标准的 cdecl 调用约定。windll()方法加载的库，其导出的函数必须使用 stdcall 调用约定(Win32 API 的原生约 定)。oledll()方法和 windll()类似，不过如果函数返回一个 HRESULT 错误代码，可以使用 COM 函数得到具体的错误信息。

调用约定

调用约定专指函数的调用方法。其中包括，函数参数的传递方法，顺序（压入栈或 者传给寄存器），以及函数返回时，栈的平衡处理。下面这两种约定是我们最常用到的： cdecl and stdcall。cdecl 调用约定，函数的参数从右往左依次压入栈内，函数的调用者， 在函数执行完成后，负责函数的平衡。这种约定常用于 x86 架构的 C 语言里。

In C

```py
int python_rocks(reason_one, reason_two, reason_three); 
```

In x86 Assembly

```py
push reason_three 
push reason_two 
push reason_one 
call python_rocks 
add esp, 12 
```

从上面的汇编代码中，可以清晰的看出参数的传递顺序，最后一行，栈指针增加了 12 个字节(三个参数传递个函数，每个被压入栈的指针都占 4 个字节，共 12 个)， 使得 函数调用之后的栈指针恢复到调用前的位置。

下面是个 stdcall 调用约定的了例子，用于 Win32 API。

In C

```py
int my_socks(color_one color_two, color_three); 
```

In x86 Assembly

```py
push color_three 
push color_two 
push color_one
call my_socks 
```

这个例子里，参数传递的顺序也是从右到左，不过栈的平衡处理由函数 my_socks 自己完成，而不是调用者。 最后一点，这两种调用方式的返回值都存储在 EAX 中。

下面做一个简单的试验，直接从 C 库中调用 printf()函数打印一条消息，Windows 中的 C 库 位于 C:\WINDOWS\system32\msvcrt.dll，Linux 中的 C 库位于/lib/libc.so.6。

chapter1-printf.py Code on Windows

```py
from ctypes import * 

msvcrt = cdll.msvcrt
message_string = "Hello world!\n" 
msvcrt.printf("Testing: %s", message_string) 
```

输出结果见如下：

```py
C:\Python25> python chapter1-printf.py 
Testing: Hello world!
C:\Python25> 
```

Linux 下会有略微不同：

chapter1-printf.py Code on Linux

```py
from ctypes import *

libc = CDLL("libc.so.6")
message_string = "Hello world!\n" 
libc.printf("Testing: %s", message_string) 
```

输出结果如下:

```py
# python /root/chapter1-printf.py 
Testing: Hello world!
# 
```

可以看到 ctypes 调用动态链接库中的函数有多简单。

### 1.3.3 构造 C 数据类型

使用 Python 创建一个 C 数据类型很简单，你可以很容易的使用由 C 或者 C++些的组件。 Listing 1-1 显示三者之间的对于关系。

| C Type | Python Type | ctypes Type |
| --- | --- | --- |
| char | 1-character string | c_char |
| wchar_t | 1-character Unicode string | c_wchar |
| char | int/long | c_byte |
| char | int/long | c_ubyte |
| short | int/long | c_short |
| unsigned short | int/long | c_ushort |
| int | int/long | C_int |
| unsigned int | int/long | c_uint |
| long | int/long | c_long |
| unsigned long | int/long | c_ulong |
| long long | int/long | c_longlong |
| unsigned long long | int/long | c_ulonglong |
| float | float | c_float |
| double | float | c_double |
| char * (NULL terminated) | string or none | c_char_p |
| wchar_t * (NULL terminated) | unicode or none | c_wchar_p |
| void * | int/long or none | c_void_p |

Listing 1-1:Python 与 C 数据类型映射

请把这章表放到随时很拿到的地方。ctypes 类型初始化的值，大小和类型必须符合定义 的要求。看下面的例子。

```py
C:\Python25> python.exe
Python 2.5 (r25:51908, Sep 19 2006, 09:52:17) [MSC v.1310 32 bit (Intel)] on win32 Type "help", "copyright", "credits" or "license" for more information.
>>> from ctypes import *
>>> c_int() c_long(0)
>>> c_char_p("Hello world!") c_char_p('Hello world!')
>>> c_ushort(-5) c_ushort(65531)
>>> c_short(-5) c_short(-5)
>>> seitz = c_char_p("loves the python")
>>> print seitz c_char_p('loves the python')
>>> print seitz.value loves the python
>>> exit() 
```

最 后 一 个 例 子 将 包 含 了 "loves the python" 的 字 符 串 指 针 赋 值 给 变 量 seitz ， 并 通 过 seitz.value 方法间接引用了指针的内容，

### 1.3.5 定义结构和联合

结构和联合是非常重要的数据类型，被大量的适用于 WIN32 的 API 和 Linux 的 libc 中。 一个结构变量就是一组简单变量的集合 (所有变量都占用空间)些结构内的变量在类型上没 有限制，可以通过点加变量名来访问。比如 beer_recipe.amt_barley，就是访问 beer_recipe 结 构中的 amt_barley 变量。

In C

```py
struct beer_recipe
{
    int amt_barley; 
    int amt_water;
}; 
```

In Python

```py
class beer_recipe(Structure):
    _fields_ = [ 
        ("amt_barley", c_int), 
        ("amt_water", c_int),
    ] 
```

如你所见，ctypes 很简单的就创建了一个 C 兼容的结构。 联合和结构很像。但是联合中所有变量同处一个内存地址，只占用一个变量的内存空间，这个空间的大小就是最大的那个变量的大小。这样就能够将联合作为不同类型的变量操作访 问了。

In C

```py
union {
    long barley_long; int barley_int;
    char barley_char[8];
}barley_amount; 
```

In Python

```py
class barley_amount(Union):
    _fields_ = [ 
        ("barley_long", c_long), 
        ("barley_int", c_int),
        ("barley_char", c_char * 8),
    ] 
```

如果我们将一个整数赋值给联合中的 barley_int，接着我们就能够调用 barley_char，用 字符的形式显示刚才输入的 66。

chapter1-unions.py

```py
from ctypes import *
class barley_amount(Union):
    _fields_ = [
        ("barley_long", c_long), ("barley_int", c_int), ("barley_char", c_char * 8),
    ]
value = raw_input("Enter the amount of barley to put into the beer vat: 
my_barley = barley_amount(int(value))
print "Barley amount as a long: %ld" % my_barley.barley_long 
print "Barley amount as an int: %d" % my_barley.barley_long 
print "Barley amount as a char: %s" % my_barley.barley_char 
```

输出如下:

```py
C:\Python25> python chapter1-unions.py
Enter the amount of barley to put into the beer vat: 66 
Barley amount as a long: 66
Barley amount as an int: 66 
Barley amount as a char: B C:\Python25> 
```

给联合赋一个值就能得到三种不同的表现方式。最后一个 barley_char 输出的结果是 B， 因为 66 刚好是 B 的 ASCII 码。

barley_char 成员同时也是个数组，一个八个字符大小的数组。在 ctypes 中申请一个数组， 只要简单的将变量类型乘以想要申请的数量就可以了。

一切就绪，开始我们的旅程吧！