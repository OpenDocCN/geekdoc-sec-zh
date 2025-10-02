# 5\. 入侵基本原理

本章将说明管理入侵和强力攻击的基本组件。 这些组件包括有效载荷、客户端/服务器端 攻击和模块。

5.1 有效载荷： 提供两个针对入侵和强力攻击的有效载荷选项。 如果选择 Meterpreter 不起作用，则使用 Command Shell。

5.2 Meterpreter 是 Metasploit 有效载荷，为攻击者提供交互式的外壳（shell）。例 如：利用 VNC 控制设备的屏幕并浏览、上传和下载文件。 该选项能够完成大量入侵 后任务并进行定制，例如在网络内运行脚本自动化入侵。 将在以下情况创建 Meterpreter 会话：

*   成功入侵 Windows

*   对 Windows 进行 SSH 强力攻击

*   对 Windows 进行 Telnet 强力攻击

*   对 Windows 进行 SMB 强力攻击

*   对 Windows 进行 Tomcat 强力攻击

5.3 Command Shell 帮助用户对主机运行收集脚本或运行任意命令。 将在以下情况创建 Command Shell 会话：

*   成功入侵 *nix

*   对 *nix 进行 SSH 强力攻击

*   对 *nix 进行 Telnet 强力攻击

*   对 *nix 进行 Tomcat 强力攻击

5.4 模块： 任何人都可以开发模块，为社区做出贡献。

*   充分利用一处的所有 Metasploit 模块

    *   通过关键字轻松简便地使用搜索界面

    *   可以扩展所有标准入侵来瞄准一定范围

    *   使用任何您已知的首选模块

*   细粒度控制模块选项

    *   指定并覆盖任何标准选项

    *   使用 Advanced（高级）和 Evasion（闪避）选项

*   基本自动化有效载荷的选择

    *   选择 Meterpreter vs Shell

    *   选择 Reverse（反向）vs Bind（绑定）

    *   选择端口范围

5.5 Click the ![](img/metasploit 中文操作手册 9235.png)在菜单栏上将显示模块统计和搜索关键字格式

![](img/metasploit 中文操作手册 9277.png)·

5.6 要搜索模块，请输入`name: vulnerability_keyword`或`cve-xxxx-xxxx`。 例 如：`name:scanner ftp`。

![](img/metasploit 中文操作手册 9372.png)

5.7 服务器端入侵： Metasploit 将作为客户端连接到服务器并进行入侵。 例如： Metasploit 连接到 HTTP 服务并入侵 web 应用程序。 要搜索用来入侵服务器的模 块，请搜索`app:server`。

![](img/metasploit 中文操作手册 9494.png)

5.8 客户端入侵： 可以设置 Metasploit 监听服务并让客户端进行访问。 一旦建立了连 接，Metasploit 将尝试入侵客户端。 在本例中，Metasploit 将作为服务器并入侵客 户端。 要搜索用来入侵客户端的最新模块，请搜索`app:client`。

5.9 会话： 一旦 Metasploit 入侵了主机，就会构建一个会话。 会话是与主机建立的连 接，并让您在主机上`执行某些操作`，例如复制文件。

![](img/metasploit 中文操作手册 9726.png)