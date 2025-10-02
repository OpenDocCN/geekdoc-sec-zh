# 8\. 控制会话

当您在目标系统上发现了有效的主机并获取了该系统上会话的访问权限，您就可以控制这 些打开的会话。 有两类会话：

Meterpreter 会话 – 这些会话的功能更强大。 不仅允许您利用 VNC 获取设备的访问权 限，还帮助您利用内置的文件浏览器上传/下载敏感信息。

Command shell 会话 – 这些会话允许您对主机运行收集脚本，并通过外壳（shell）来 运行任意命令。

8.1 可以通过`Exploit`（7.8 入侵章节）或`Bruteforce`（6.10 强力攻击章节）来 构建会话。

8.2 要重新打开会话，请前往`Sessions`（会话）选项卡。 如果存在任何`Active Sessions`（活动会话），则关闭所有这些会话。 点击其中一个已关闭会话的 `Attack Module`（攻击模块）。

![](img/metasploit 中文操作手册 13180.png)

8.3 将显示模块详细信息、地址和有效载荷选项。 点击`Run Module`（运行模块）再次 进行入侵。

![](img/metasploit 中文操作手册 13240.png)

8.4 Meterpreter 会话示例。 前往一个处于活动状态的会话。 为 Windows 使用 Meterpreter 有效载荷。 您可以点击`Terminate Session`（终止会话）来关闭此 会话（不过现在不要执行该操作）。

![](img/metasploit 中文操作手册 13367.png)

8.5 收集证据： 获得访问权限后可以从目标自动收集证据。 可以使用这些证据执行进一步分析和渗透测试。 点击![](img/metasploit 中文操作手册 13457.png)开始收集证据。 您可以选择收集 `System Information`（系统信息，即操作系统信息）、`Passwords`（密码）、 `Screenshots`（屏幕截图）和`SSH Keys`（SSH 密钥）。 您也可以下载匹配模 式的文件。

![](img/metasploit 中文操作手册 13586.png)

8.6 要检查从主机收集而来的证据，请前往`Analysis &gt; Hosts`（分析 > 主机）并从表 格中选择主机。 可以在`Captured Evidence`（已捕获的证据）选项卡上找到这些 证据（例如：凭证和屏幕截图）。

![](img/metasploit 中文操作手册 13723.png)

8.7 Meterpreter 会话将显示![](img/metasploit 中文操作手册 13774.png)，允许您通过 VNC 会话连接到目标。 点 击`OK`（确定）来访问虚拟桌面。

![](img/metasploit 中文操作手册 13816.png)

8.8 Metasploit Pro 拥有一个 Java 小程序形式的 VNC 客户端。 请为您的平台安装最新 的 Java。 此外还可以选择使用外部客户端（例如 VNC Viewer）。 点击蓝色的 `Java Applet`（Java 小程序）继续。

![](img/metasploit 中文操作手册 13953.png)

8.9 现在您就有了 VNC 会话。

![](img/metasploit 中文操作手册 13982.png)

8.10 点击![](img/metasploit 中文操作手册 13992.png)来访问目标的文件系统。 Metasploit 将显示所有已映射的 驱动器。 您可以浏览目录，下载、上传并删除作为证据的文件。

![](img/metasploit 中文操作手册 14058.png)

8.11 点击![](img/metasploit 中文操作手册 14071.png)，为特定的模式搜索远程文件系统。 这是寻找敏感文件的 有用工具。

![](img/metasploit 中文操作手册 14106.png)

8.12 点击![](img/metasploit 中文操作手册 14114.png)与目标上的 Command Shell 进行交互。 使用`help`（帮 助）来显示可用命令。它可以显示进程、浏览文件系统、使用 Webcam 等。

![](img/metasploit 中文操作手册 14197.png)

8.13 下方是`getuid`和`getsystem`命令的示例。

![](img/metasploit 中文操作手册 14234.png)

![](img/metasploit 中文操作手册 14240.png)

8.14 点击![](img/metasploit 中文操作手册 14248.png)按钮来进行 Pivot Attack，将已入侵的主机（例如主机 A）作为网关（TCP/UDP）来入侵另一个主机（例如主机 B）。如果该攻击引起警报， 将看到远程主机（主机 A），而不是 Metasploit 主机。 点击`OK`（确定）来继续 创建 Proxy Pivot。 如果创建成功，您将看见`Route via over Session 61 created`（通过创建的会话 61 来进行路由）这条消息。

例如：Metasploit Pro（192.168.152.10）通过 WinXP 的 Proxy Pivot

（192.168.152.133）入侵目标 Metasploitable（192.168.152.129）。 从 Metasploitable 捕获的数据包来看，这些数据包只与 WinXP（而不是 Metasploit）

进行交换。

![](img/metasploit 中文操作手册 14647.png)

8.15 ![](img/metasploit 中文操作手册 14658.png)用来通过远程主机（Ethernet/IP）对通信量进行 Pivot Attack。通常，该功能用来入侵公共系统并交付 Meterpreter。 VPN Pivot 将利用与 远端受损主机建立的连接在攻击机器上创建接口。 要测试此功能，您需要准备一个具 有两个接口的 Windows 主机，一个接口连接到 Metasploit，另一个接口连接到另一 个目标主机。

![](img/metasploit 中文操作手册 14844.png)

8.16 `Session History`（会话历史记录）选项卡显示已在运行哪些命令（例如 VNC）和脚本（例如 VPN.rb）以及输出信息（例如：浏览文件系统）。

![](img/metasploit 中文操作手册 14940.png)

8.17 `Post-Exploitation Modules`（入侵后模块）选项卡列出可以在此会话上运行 的所有可用模块。 可以从以下链接找到脚本库。

[`www.metasploit.com/redmine/projects/framework/repository/show/scripts/meter`](http://www.metasploit.com/redmine/projects/framework/repository/show/scripts/meter) preter

8.18 要运行一个模块，请找到`Windows Gather Product Key`（Windows 收集产品密

钥）并点击![](img/metasploit 中文操作手册 15499.png)。 若成功，将在`Stored Data & Files`（已存储数据和 文件）选项卡上显示密钥。

Home Project A Modules Windows Gather Product Key

![](img/metasploit 中文操作手册 15609.png)