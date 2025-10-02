# 12\. 高级技术

本章节将说明`Post-Exploitation Macros`（入侵后宏）和`Persistent Agent`（持 续代理程序）。 `Post-Exploitation Macros`（入侵后宏）帮助您在构建完会话后自 动执行操作。 例如：通过 XP 工作站构建完一个会话后，无需管理员的介入即可自动执 行密码收集和屏幕捕获操作。

`Persistent Agent`（持续代理程序）帮助您在重启和登录主机后，让主机自动构建针 对 Metasploit 的会话。无需再次入侵/强力攻击主机。 换言之，在 Metasploit 的控制 下，部署了代理程序的主机就像是`僵尸`。

12.1 将鼠标移至顶部菜单栏上的![](img/metasploit 中文操作手册 20468.png)选项卡并点击![](img/metasploit 中文操作手册 20475.png)。

![](img/metasploit 中文操作手册 20482.png)

12.2 在`Global Settings`（全局设置）页面上的`Post-Exploitation Macros`

（入侵后宏）部分中，点击![](img/metasploit 中文操作手册 20559.png)来添加一个新的宏。

![](img/metasploit 中文操作手册 20574.png)

12.3 输入名称和描述并点击![](img/metasploit 中文操作手册 20595.png)，然后就会显示供选择的模块。

![](img/metasploit 中文操作手册 20612.png)

12.4 您可以按关键字（例如 windows key）来搜索模块。 将鼠标移至选定模块（例 如：Windows Gather Product Key）的最后一栏，将显示![](img/metasploit 中文操作手册 20702.png)，点击该图标为此新宏添 加一个操作。

![](img/metasploit 中文操作手册 20726.png)

![](img/metasploit 中文操作手册 20732.png)

12.5 点击 ![](img/metasploit 中文操作手册 20765.png) 进行确认，以便将选定的操作添加到此宏。

12.6 将在上方表格中显示已添加的操作。 点击![](img/metasploit 中文操作手册 20819.png)来保存设置。

![](img/metasploit 中文操作手册 20828.png)

12.7 在构建完会话时可以自动启动宏。 您可以在`Exploit`（入侵，7.4 章节）、`Bruteforce`（强力攻击，6.6 章节）（![](img/metasploit 中文操作手册 20941.png)）、`Campaigns > General Settings`（宣传活动 > 常规设置， 11.2 章节）中的`Advanced Section > Payload Settings` （高级部分 > 有效载荷设置）下选择`Macro`（宏），也可 以在各个模块的`Payload Options`（有效载荷选项）中选择这些宏。

![](img/metasploit 中文操作手册 21016.png)

![](img/metasploit 中文操作手册 21122.png)

![](img/metasploit 中文操作手册 21128.png)

12.8 在`Persistent Listeners`（持续监听器）部分下，您可以添加监听器来让客户端自动回连。 点击![](img/metasploit 中文操作手册 21214.png)。

![](img/metasploit 中文操作手册 21221.png)

12.9 选择一个项目来与此监听器相关联。 选择`Listener Payload`（监听器有效载 荷）、`Address`（地址）和`Port`（端口）。 一旦远程主机回连，即可自动加 载宏。

![](img/metasploit 中文操作手册 21328.png)

12.10 点击![](img/metasploit 中文操作手册 21358.png)，即可在选定的项目上找到`Listening`（监听）任务。

![](img/metasploit 中文操作手册 21394.png)

![](img/metasploit 中文操作手册 21399.png)

12.11 可以通过点击`Inactive/Active`（闲置/活动）状态来启动/停止监听器。

![](img/metasploit 中文操作手册 21455.png)

12.12 现在`母舰`已准备就绪，下一步是部署持续代理程序，以便让客户端回连。 在 针对 WinXP 的活动会话中，运行入侵后模块`Metasploit Pro Persistent Agent`（Metasploit Pro 持续代理程序）。

![](img/metasploit 中文操作手册 21591.png)

![](img/metasploit 中文操作手册 21594.png)

12.13 选择会话并点击![](img/metasploit 中文操作手册 21629.png)。 您将在任务日志上看到目标对象中的代理程序位 置（例如：c:.....\ajlkfljdsaf.exe）。

![](img/metasploit 中文操作手册 21690.png)

12.14 前往目标文件系统，并检查是否存在这个代理程序。

![](img/metasploit 中文操作手册 21730.png)

12.15 重启 WinXP 目标并再次登录。 您将看到会话的自动构建。

![](img/metasploit 中文操作手册 21777.png)

12.16 要删除代理程序，您可以运行入侵后模块`Metasplit Pro Persistent Agent Cleaner`（Metasplit Pro 持续代理程序清除器）。