# 第二章 编译/安装 Wireshark

**目录**

*   2.1\. 须知
*   2.2\. 获得源
*   2.3\. 在 UNIX 下安装之前
*   2.4\. 在 UNIX 下编译 Wireshark
*   2.5\. 在 UNIX 下安装二进制包
    *   2.5.1\. 在 Linux 或类似环境下安装 RPM 包
    *   2.5.2\. 在 Debian 环境下安装 Deb 包
    *   2.5.3\. 在 Gentoo Linux 环境下安装 Portage
    *   2.5.4\. 在 FreeBSD 环境下安装包
*   2.6\. 解决 UNIX 下安装过程中的问题
*   2.7\. 在 Windows 下编译源
*   2.8\. 在 Windows 下安装 Wireshark
    *   2.8.1\. 安装 Wireshark
    *   2.8.2\. 手动安装 WinPcap
    *   2.8.3\. 更新 Wireshark
    *   2.8.4\. 更新 WinPcap
    *   2.8.5\. 卸载 Wireshark
    *   2.8.6\. 卸载 WinPcap

## 2.1\. 须知

万事皆有开头，Wireshark 也同样如此。要想使用 Wireshark，你必须：

*   获得一个适合您操作系统的二进制包，或者

*   获得源文件为您的操作系统编译。

目前，只有两到三种 Linux 发行版可以传送 Wireshark，而且通常传输的都是过时的版本。至今尚未有 UNIX 版本可以传输 Wireshark . Windows 的任何版本都不能传输 Wireshark.基于以上原因，你需要知道从哪能得到最新版本的 Wireshark 以及如何安装它。

本章节向您展示如何获得源文件和二进制包，如何根据你的需要编译 Wireshark 源文件。

以下是通常的步骤：

1.  下载需要的相关包，例如：源文件或者二进制发行版。

2.  将源文件编译成二进制包(如果您下载的是源文件的话)。这样做做可以整合编译和/或安装其他需要的包。

3.  安装二进制包到最终目标位置。

## 2.2\. 获得源

你可以从 Wireshark 网站[`www.wireshark.org`](http://www.wireshark.org/).同时获取源文件和二进制发行版。选择您需要下载的链接，然后选择源文件或二进制发行包所在的镜像站点（尽可能离你近一点的站点）。

> ![](img/000066.png)
> 
> 下载所有需要的文件 !
> 
> 一般来说，除非您已经下载 Wireshark,如果您想编译 Wireshark 源文件，您可能需要下载多个包。这些在后面章节会提到。
> 
> ![](img/000066.png)
> 
> 注意
> 
> 当你发现在网站上有多个二进制发行版可用，您应该选择适合您平台的版本，他们同时通常会有多个版本紧跟在当前版本后面，那些通常时拥有那些平台的用户编译的。

基于以上原因，您可能想自己下载源文件自己编译，因为这样相对方便一点。

## 2.3\. 在 UNIX 下安装之前

在编译或者安装二进制发行版之前，您必须确定已经安装如下包：

1.  GTK+, The GIMP Tool Kit.

    您将会同样需要 Glib.它们都可以从 www.gtk.org 获得。

2.  Libpcap , Wireshark 用来捕捉包的工具

    您可以从 www.tcpdump.org 获得。

根据您操作系统的不同，您或许能够安装二进制包，如 RPMs.或许您需要获得源文件并编译它。

如果您已经下载了 GTK+源文件，例 2.1 “从源文件编译 GTK+”提供的指令对您编译有所帮助。

**例 2.1\. 从源文件编译 GTK+**

```
gzip -dc gtk+-1.2.10.tar.gz | tar xvf - 
```

```
<much output removed> 
```

```
./configure 
```

```
<much output removed> 
```

```
make install 
```

```
<much output remove> 
```

```
test-------------- 
```

> ![](img/000066.png)
> 
> 注意
> 
> 您可能需要修改例 2.1 “从源文件编译 GTK+”中提供的版本号成对应您下载的 GTK+版本。如果 GTK 的目录发生变更，您同样需要修改它。，tar xvf 显示您需要修改的目录。
> 
> ![](img/000066.png)
> 
> 注意
> 
> 如果您使用 Linux,或者安装了 GUN **tar**，您可以使用**tar zxvfgtk+-1.2.10.tar.gz**命令。同样也可能使用**gunzip –c**或者**gzcat**而不是许多 UNIX 中的**gzip –dc**
> 
> ![](img/000066.png)
> 
> 注意
> 
> 如果您在 windows 中下载了 gtk+ 或者其他文件。您的文件可能名称为：`gtk+-1_2_8_tar.gz`

如果在执行例 2.1 “从源文件编译 GTK+”中的指令时有错误发生的话，你可以咨询 GTK+网站。

如果您已经下载了 libpcap 源，一般指令如例 2.2 “编译、安装 libpcap” 显示的那样会帮您完成编译。同样，如果您的操作系统不支持**tcpdump**,您可以从[tcpdump](http://www.tcpdump.org/)网站下载安装它。

**例 2.2\. 编译、安装 libpcap**

```
gzip -dc libpcap-0.9.4.tar.Z | tar xvf - 
```

```
<much output removed> 
```

```
cd libpcap-0.9.4 
```

```
./configure 
```

```
<much output removed> 
```

```
make 
```

```
<much output removed> 
```

```
make install 
```

```
<much output removed> 
```

> ![](img/000066.png)
> 
> 注意
> 
> Libpcap 的目录需要根据您的版本进行修改。**tar xvf**命令显示您解压缩的目录。

RedHat 6.x 及其以上版本环境下（包括基于它的发行版，如 Mandrake）,您可以直接运行 RPM 安装所有的包。大多数情况下的 Linux 需要安装 GTK+和 Glib.反过来说，你可能需要安装所有包的定制版。安装命令可以参考例 2.3 “在 RedHat Linux 6.2 或者基于该版本得发行版下安装需要的 RPM 包”。如果您还没有安装，您可能需要安装需要的 RPMs。

**例 2.3\. 在 RedHat Linux 6.2 或者基于该版本得发行版下安装需要的 RPM 包**

```
cd /mnt/cdrom/RedHat/RPMS 
```

```
rpm -ivh glib-1.2.6-3.i386.rpm 
```

```
rpm -ivh glib-devel-1.2.6-3.i386.rpm 
```

```
rpm -ivh gtk+-1.2.6-7.i386.rpm 
```

```
rpm -ivh gtk+-devel-1.2.6-7.i386.rpm 
```

```
rpm -ivh libpcap-0.4-19.i386.rpm 
```

> ![](img/000066.png)
> 
> 注意
> 
> 如果您使用 RedHat 6.2 之后的版本，需要的 RMPs 包可能已经变化。您需要使用正确的 RMPs 包。

在 Debian 下您可以使用 apt-ge 命令。apt-get 将会为您完成所有的操作。参见例 2.4 “在 Deban 下安装 Deb”

**例 2.4\. 在 Deban 下安装 Deb**

```
apt-get install wireshark-dev 
```

## 2.4\. 在 UNIX 下编译 Wireshark

如果在 Unix 操作系统下可以用如下步骤编译 Wireshark 源代码：

1.  如果使用 Linux 则解压**gzip'd tar**文件,如果您使用 UNIX，则解压 GUN **tar**文件。对于 Linux 命令如下：

    ```
    tar zxvf wireshark-0.99.5-tar.gz 
    ```

    对于 UNIX 版本，命令如下

    ```
    gzip -d wireshark-0.99.5-tar.gz 
    ```

    ```
    tar xvf wireshark-0.99.5-tar 
    ```

    | ![](img/000066.png) | 注意 | |

    使用管道命令行 gzip –dc Wireshark-0.99.5-tar.gz|tar xvf 同样可以[9]

    |

    | ![](img/000066.png) | 注意 | |

    如果您在 Windows 下下载了 Wireshark,你会发现文件名中的那些点变成了下划线。

    |

2.  将当前目录设置成源文件的目录。

3.  配置您的源文件以编译成适合您的 Unix 的版本。命令如下：

    ```
    ./configure 
    ```

    如果找个步骤提示错误，您需要修正错误，然后重新 configure.解决编译错误可以参考第 2.6 节 “解决 UNIX 下安装过程中的问题 ”

4.  使用 make 命令将源文件编译成二进制包，例如：

    ```
    make 
    ```

5.  安装您编译好的二进制包到最终目标，使用如下命令：

    ```
    make install 
    ```

    一旦您使用 make install 安装了 Wireshark,您就可以通过输入 Wireshark 命令来运行它了。

[9] 译者注：看到别人翻译 Pipelin 之类的，似乎就是叫管道，不知道是否准确

## 2.5\. 在 UNIX 下安装二进制包

一般来说，在您的 UNIX 下安装二进制发行包使用的方式根据您的 UNIX 的版本类型而各有不同。例如 AIX 下，您可以使用 smit 安装，Tru64 UNIX 您可以使用 setld 命令。

### 2.5.1\. 在 Linux 或类似环境下安装 RPM 包

使用如下命令安装 Wireshark RPM 包

```
rpm -ivh wireshark-0.99.5.i386.rpm 
```

如果因为缺少 Wireshark 依赖的软件而导致安装错误，请先安装依赖的软件，然后再尝试安装。REDHAT 下依赖的软件请参考例 2.3 “在 RedHat Linux 6.2 或者基于该版本得发行版下安装需要的 RPM 包”

### 2.5.2\. 在 Debian 环境下安装 Deb 包

使用下列命令在 Debian 下安装 Wireshark

```
apt-get install Wireshark 
```

apt-get 会为您完成所有的相关操作

### 2.5.3\. 在 Gentoo Linux 环境下安装 Portage

使用如下命令在 Gentoo Linux 下安装 wireshark 以及所有的需要的附加文件

```
USE="adns gtk ipv6 portaudio snmp ssl kerberos threads selinux" emerge wireshark 
```

### 2.5.4\. 在 FreeBSD 环境下安装包

使用如下命令在 FreeBSD 下安装 Wireshark

```
pkg_add -r wireshark 
```

pkg_add 会为您完成所有的相关操作

## 2.6\. 解决 UNIX 下安装过程中的问题 [10]

安装过程中可能会遇到一些错误信息。这里给出一些错误的解决办法：

如果**configure**那一步发生错误。你需要找出错误的原因，您可以检查日志文件 config.log(在源文件目录下)，看看都发生了哪些错误。有价值的信息通常在最后几行。

一般原因是因为您缺少 GTK+环境，或者您的 GTK+版本过低。configure 错误的另一个原因是因为因为缺少 libpcap(这就是前面提到的捕捉包的工具)。

另外一个常见问题是很多用户抱怨最后编译、链接过程需要等待太长时间。这通常是因为使用老式的**sed**命令（比如 solaris 下传输）。自从 libtool 脚本使用 sed 命令建立最终链接命令，常常会导致不可知的错误。您可以通过下载最新版本的 sed 解决该问题[`directory.fsf.org/GNU/sed.html`](http://directory.fsf.org/GNU/sed.html).

如果您无法检测出错误原因。发送邮件到 wireshark-dev 说明您的问题。当然，邮件里要附上 config.log 以及其他您认为对解决问题有帮助的东西，例如 make 过程的追踪。

[10] 译者注：本人不熟悉 UNIX/LINUX，这一段翻译的有点云里雾里，可能大家通过这部分想安装 Wireshark 会适得其反，那就对不住了。下面个人说一下 UNIX/LINUX 下安装方法。 UNIX/LINUX 下安装时，有两种安装方式，1 是下载源码包自己编译，这种方式的好处是因为下载源码包是单一的，可以自行加以修改，编译就是适合自己平台的了。 2、是利用已经做好的发行包直接安装，这种方法的好处是只要下载到跟自己平台对应的就可以，但缺点也在这里，不是每个平台都能找到合适的。不管是编译安装，还是使用发行包安装，都需要有一些有些基本基本支持。比如 Linux 下的 GTK+支持，捕捉包时需要用的 libpcap. 这一点可以参考第 2.3 节 “在 UNIX 下安装之前 ”。编译的一般步骤是解压，编译，安装（**tar zxvf Wireshark-0.99.5-tar.gz;make;make install**）.直接安装则是根据各自平台安装的特点。

## 2.7\. 在 Windows 下编译源

在 Windows 平台下，我们建议最好是使用二进制包直接安装，除非您是从事 Wireshark 开发的。 如果想了解关于 Windows 下编译安装 Wireshark，请查看我们的开发 WIKI 网站[`wiki.wireshark.org/Development`](http://wiki.wireshark.org/Development)来了解最新的开发方面的文档。

## 2.8\. 在 Windows 下安装 Wireshark

本节将探讨在 Windows 下安装 Wireshark 二进制包。

### 2.8.1\. 安装 Wireshark

您获得的 Wireshark 二进制安装包可能名称类似`Wireshark-setup-x.y.z.exe.` Wireshark 安装包包含 WinPcap,所以您不需要单独下载安装它。

您只需要在[`www.wireshark.org/download.html#releases`](http://www.wireshark.org/download.html#releases)下载 Wireshark 安装包并执行它即可。除了普通的安装之外，还有几个组件供挑选安装。

> ![](img/000068.png)
> 
> 提示：尽量保持默认设置
> 
> 如果您不了解设置的作用的话。

#### <a name="2.8.1.1"></a>选择组件[11]

Wireshark(包括 GTK1 和 GTK2 接口无法同时安装):

如果您使用 GTK2 的 GUI 界面遇到问题可以尝试 GTK1，在 Windows 下 256 色（8bit）显示模式无法运行 GTK2.但是某些高级分析统计功能在 GTK1 下可能无法实现。

*   **Wireshark GTK1**-Wireshark 是一个 GUI 网络分析工具

*   **Wireshark GTK2**-Wireshark 是一个 GUI 网络分析工具（建议使用 GTK2 GUI 模组工具）

*   **GTK-Wimp**-GTKWimp 是诗歌 GTK2 窗口模拟(看起来感觉像原生 windows32 程序，推荐使用)

*   **TSshark**-TShark 是一个命令行的网络分析工具

插件/扩展(Wireshark,TShark 分析引擎):

*   **Dissector Plugins**-分析插件：带有扩展分析的插件

*   **Tree Statistics Plugins**-树状统计插件：统计工具扩展

*   **Mate - Meta Analysis and Tracing Engine (experimental)**:可配置的显示过滤引擎，参考[`wiki.wireshark.org/Mate`](http://wiki.wireshark.org/Mate).

*   **SNMP MIBs**: SNMP，MIBS 的详细分析。

Tools/工具(处理捕捉文件的附加命令行工具

**User’s Guide**-用户手册-本地安装的用户手册。如果不安装用户手册，帮助菜单的大部分按钮的结果可能就是访问 internet.

*   **Editcap** - Editcap is a program that reads a capture file and writes some or all of the packets into another capture file. /Editcap 是一个读取捕捉文件的程序，还可以将一个捕捉文件力的部分或所有信息写入另一个捕捉文件。（文件合并 or 插入？）
*   **Text2Pcap** - Text2pcap is a program that reads in an ASCII hex dump and writes the data into a libpcap-style capture file./Tex2pcap 是一个读取 ASCII hex，写入数据到 libpcap 个文件的程序。

*   **Mergecap** - Mergecap is a program that combines multiple saved capture files into a single output file. / Mergecap 是一个可以将多个播捉文件合并为一个的程序。

*   **Capinfos** - Capinfos is a program that provides information on capture files. /Capinfos 是一个显示捕捉文件信息的程序。

#### <a name="c2.8.1.2"></a>“Additional Tasks”页

*   **Start Menu Shortcuts**-开始菜单快捷方式-增加一些快捷方式到开始菜单

*   **Desktop Icon**-桌面图标-增加 Wireshark 图标到桌面

*   **Quick Launch Icon**-快速启动图标-增加一个 Wireshark 图标到快速启动工具栏

*   **Associate file extensions to Wireshark**-Wireshark 文件关联-将捕捉包默认打开方式关联到 Wireshark

#### <a name="c2.8.1.3"></a>Install WinPcap?”页

Wireshark 安装包里包含了最新版的 WinPcap 安装包。

如果您没有安装 WinPcap 。您将无法捕捉网络流量。但是您还是可以打开以保存的捕捉包文件。

*   **Currently installed WinPcap version**-当前安装的 WinPcap 版本

*   **Install WinPcap x.x** -如果当前安装的版本低于 Wireshark 自带的，该选项将会是默认值。

*   **Start WinPcap service "NPF" at startup** -将 WinPcap 的服务 NPF 在启动时运行-这样其它非管理员用户就同样可以捕捉包了。

更多关于 WinPcap 的信息：

*   Wireshark 相关[`wiki.wireshark.org/WinPcap`](http://wiki.wireshark.org/WinPcap)

*   WinPcap 官方网站：[`www.winpcap.org`](http://www.winpcap.org/)

#### <a name="c2.8.1.4"></a>安装命令选项

您可以直接在命令行运行安装包，不加任何参数，这样会显示常用的参数以供交互安装。 在个别应用中，可以选择一些参数定制安装：

*   **/NCRC** 禁止 CRC 校检

*   **/S** 静默模式安装或卸载 Wireshark.注意：静默模式安装时不会安装 WinPcap!

*   **/desktopicon** 安装桌面图标，/desktopicon=yes 表示安装图标，反之则不是，适合静默模式。

*   **/quicklaunchicon** 将图标安装到快速启动工具栏，=yes-安装到工具栏，=no-不安装，不填按默认设置。

*   **/D** 设置默认安装目录(`$INSTDIR`),首选安装目录和安装目录注册表键值，该选项必须设置到最后。即使路径包含空格

**例 2.5\.**

```
wireshark-setup-0.99.5.exe /NCRC /S /desktopicon=yes /quicklaunchicon=no /D=C:\Program Files\Foo 
```

### 2.8.2\. 手动安装 WinPcap

> ![](img/000066.png)
> 
> 注意
> 
> 事先声明，Wireshark 安装时会谨慎对待 WinPcap 的安装，所以您通常不必担心 WinPcap。

下面的 WinPcap 仅适合您需要尝试未包括在 Wireshark 内的不同版本 WinPcap。例如一个新版本的 WinPcap 发布了，您需要安装它。

单独的 WinPcap 版本（包括 alpha or beta 版）可以在下面地址下载到

*   WinPcap 官方网站：[`www.winpcap.org`](http://www.winpcap.org/)

*   Wiretapped.net 镜像站点: [`www.mirrors.wiretapped.net/security/packet-capture/winpcap`](http://www.mirrors.wiretapped.net/security/packet-capture/winpcap)

在下载页面您将会发现 WinPcap 的安装包名称通常类似于”auto-installer”。它们可以在 NT4.0/2000/XP/vista 下安装。

### 2.8.3\. 更新 Wireshark

有时候您可能想将您的 WinPcap 更新到最新版本，如果您订阅了 Wireshark 通知邮件，您将会获得 Wireshark 新版本发布的通知，见第 1.6.4 节 “邮件列表”

。

新版诞生通常需要 8-12 周。更新 Wireshark 就是安装一下新版本。下载并安装它就可以。更新通常不需要重新启动，也不会更改过去的默认设置

### 2.8.4\. 更新 WinPcap

WinPcap 的更新不是十分频繁，通常一年左右。新版本出现的时候您会收到 WinPcap 的通知。更新 WinPcap 后需要重新启动。

> ![](img/000057.png)
> 
> 警告
> 
> 在安装新版 WinPcap 之前，如果您已经安装了旧版 WinPcap,您必须先卸载它。最近版本的 WinPcap 安装时会自己卸载旧版。

### 2.8.5\. 卸载 Wireshark

你可以用常见方式卸载 Wireshark,使用添加/删除程序，选择”Wireshark”选项开始卸载即可。

Wireshark 卸载过程中会提供一些选项供您选择卸载哪些部分，默认是卸载核心组件，但保留个人设置和 WinPcap.

WinPcap 默认不会被卸载，因为其他类似 Wireshark 的程序有可能同样适用 WinPcap

### 2.8.6\. 卸载 WinPcap

你可以单独卸载 WinPcap,在添加/删除程序选择”WinPcap”卸载它。

> ![](img/000066.png)
> 
> 注意
> 
> 卸载 WinPcap 之后您将不能使用 Wireshark 捕捉包。

在卸载完成之后最好重新启动计算机。

[11] 涉及到过多的名次，软件又没有中文版，这里及以后尽量不翻译名称