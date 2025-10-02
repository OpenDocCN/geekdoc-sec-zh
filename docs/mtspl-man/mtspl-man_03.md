# 2\. 安装和初始化设置

本指南将在 Ubuntu Linux 10.04 x64 操作系统上安装 Metasploit Professional 版 本。 此外，将构建一个测试实验室来运行 POC。

2.1 前往 [`www.rapid7.com/products/metasploit/system-requirements.jsp`](http://www.rapid7.com/products/metasploit/system-requirements.jsp) 并检 查更新过的系统要求。

硬件要求

*   2 GHz+ 处理器

*   2 GB 内存（推荐 4 GB，根据相同设备上的虚拟机目标可能需要更高配置）

*   500MB+ 可用磁盘空间

*   10/100 Mbps 网卡

操作系统

*   Windows XP、2003、Vista、2008 Server 和 Windows 7

*   Red Hat Enterprise Linux 5.x、6.x - x86 和 x86_64

*   Ubuntu Linux 8.04、10.04 - x86 和 x86_64

浏览器版本

*   Mozilla Firefox 4.0+

*   Microsoft Internet Explorer 9

*   Google Chrome 10+

2.2 要下载 Metasploit 安装程序，请前往 [`metasploit.com/download/`](http://metasploit.com/download/) 并下载正 确的安装程序。 可以在此处找到 Professional、Express、Framework 和 Community 版本。 应下载 Professional 版本来运行 POC。

![](img/metasploit 中文操作手册 3205.png)

2.3 要获取文档（例如安装指南和用户指南等），请访问

[`community.rapid7.com/community/solutions/metasploit?view=documents`](https://community.rapid7.com/community/solutions/metasploit?view=documents)

2.4 （下方是针对 Ubuntu linux 的配置说明，如果需要安装在其他操作系统上，请参阅 安装指南。 安装程序文件名在您的 POC 中可能有所不同。） 打开命令提示框并前往 Metasploit 安装程序所位于的文件夹，使用`sudo -i`命令并输入密码来获得超级 用户特权。 使用`chmod +x metasploit-latest-linux-x64-installer.run`命令， 将执行文件属性添加到此安装程序。

![](img/metasploit 中文操作手册 3632.png)

2.5 使用`./metasploit-latest-linux-x64-installer.run`命令来运行此安装程序。 阅读并接受许可协议。 点击`Forward`（下一步）继续。

![](img/metasploit 中文操作手册 3761.png)

2.6 选择安装文件夹。 如果安装了主机反病毒程序，应将此文件夹添加到 AV 白名单中。

![](img/metasploit 中文操作手册 3811.png)

2.7 决定是否要将 Metasploit 安装为自动启动的服务。 这一步将添加一个初始化脚本， 以便在启动时调用 $INSTALLERBASE/ctlscript.sh。

![](img/metasploit 中文操作手册 3914.png)

2.8 接下来两步是接受 3790 端口作为默认的 Web GUI 端口或输入其他端口，并为 Web GUI 的 HTTPS 连接生成 SSL 证书。

![](img/metasploit 中文操作手册 4026.png)

2.9 其后两步是启用/禁用自动更新并开始运行安装程序。

![](img/metasploit 中文操作手册 4057.png)

2.10 要管理 Metasploit 服务，请前往安装文件夹并使用以下命令：

*   使用`./ctlscript.shstart`命令来启动服务

*   使用`./ctlscript.sh status`命令来检查服务状态

*   使用`./ctlscript.sh stop`命令来停止服务

![](img/metasploit 中文操作手册 4245.png)

2.11 要运行初始化设置，请打开本地浏览器并访问 [`127.0.0.1:3790。`](https://127.0.0.1:3790。) 在初始 化完成前不允许连接远程浏览器。 输入登录信息。

![](img/metasploit 中文操作手册 4332.png)

2.12 在另一个屏幕上，输入激活许可证。 请确保可以访问 `updates.metasploits.com`，因为该站点或 IP 可能被归入`黑客`类别而受到 web 过滤解决方案的阻止。 如果您没有密钥，您可以点击`Register your Metasploit license here!`（在此处注册您的 Metasploit 许可证！）来获取密 钥。

![](img/metasploit 中文操作手册 4530.png)

![](img/metasploit 中文操作手册 4536.png)

2.13 前往`Administration &gt; Software Updates`（管理 > 软件更新），并确保所有 信息（产品密钥、产品版本、注册到、许可证过期日期）都准确无误。 点击`Check for Updates`（检查更新）。 必要时请设置 HTTP 代理服务器。

![](img/metasploit 中文操作手册 4707.png)

![](img/metasploit 中文操作手册 4710.png)

2.14 要构建测试实验室，请前往 [`www.metasploit.com/help/test-lab.jsp，您`](http://www.metasploit.com/help/test-lab.jsp，您) 将找到 Metasploit 的测试实验室网络结构。 这些系统可以是虚拟机镜像或在工作站/服务器硬件上运行的实际操作系统。

*   Metasploit 控制台： 安装托管 Metasploit 的操作系统。

*   Metasploitable： 具有大量漏洞的 Linux 主机

*   Ultimate LAMP： 具有大量漏洞的 web 应用程序服务器

*   XP SP3： 其他 XP 服务包级别或 Windows 7 尤佳

*   2003 Server： 具有服务包级别的 2008 Server 尤佳

![](img/metasploit 中文操作手册 5095.png)

2.15 要下载 Metasploitable，请在上述网页上搜索`download the Metasploitable machine using BitTorent`（使用 BitTorent 下载 Metasploitable 机器）。 下载 种子，然后下载 Metasploitable 虚拟机镜像。 在 VM Workstation 或 VM Player* 中运行该文件。登录 ID 和密码分别是`msfadmin`和`msfadmin`。

*VM Player 是免费软件，但是只允许一个镜像。 VMWare 网站上提供 VM Workstation 试用版。

2.16 要下载 Ultimate LAMP： 请在上述网页上搜索`You can download UltimateLAMP here`（您可以从此处下载 UltimateLAMP）。 下载 Ultimate LAMP VM 镜像。 在 VM Workstation 或 VM Player 上运行该文件。 登录 ID 和密码分别是`vmware`和 `vmware`。

2.17 至于 Windows 域环境，您需要自行准备一台 Windows 服务器和一台 Windows 工 作站。