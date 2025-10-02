# 7\. 入侵

本章将通过 Metasploit 利用服务入侵主机来说明服务器端入侵。

7.1 要入侵主机，请前往主机选项卡`Analysis &gt; Hosts`（分析 > 主机）。 选择主机 然后点击![](img/metasploit 中文操作手册 11595.png)按钮。

![](img/metasploit 中文操作手册 11604.png)

7.2 在`Automated Exploit Settings`（自动化入侵设置）部分下方，确认目标 IP 地 址是否正确。 选择`Reliability`（可靠性）级别来实现成功入侵几率和损坏目标 几率之间的平衡。

*   Low（低）： 损坏和入侵的几率最高

*   Excellent（极好）： 不会使目标崩溃，而且避免大量高风险的操作

![](img/metasploit 中文操作手册 11785.png)

7.3 点击 ![](img/metasploit 中文操作手册 11791.png) 来访问高级选项，例如`Targeting`（目标）、 `Payload Settings`（有效载荷设置）、`Exploit Selection`（入侵选择）和 `Advanced Settings`（高级设置）。

7.4 `Targeting`（目标）部分允许您排除地址并忽略已知易受入侵的设备。 `Payload Settings`（有效载荷设置）与`Bruceforce 6.6`（强力攻击 6.6）相同。

![](img/metasploit 中文操作手册 12082.png)

7.5 在`Exploit Selection`（入侵选择）部分下，您可以为入侵设置端口并删除不太相 关的模块。 可以利用主机操作系统、开放的端口和漏洞参考来匹配将要使用的入侵。

![](img/metasploit 中文操作手册 12183.png)

7.6 在`Advanced Settings`（高级设置）部分下，您可以设置入侵行为，例如并发入侵 和超时。 启用`Only obtain one session per target`（一个目标仅获取一个会 话）会将会话数限制为一个，即使主机具有多个漏洞允许创建多个会话也不例外。

至于`Transport Evasion`（传输闪避）级别，`Low`（低）会在 TCP 数据包之间 插入延迟，`Medium`（中）将发送小型的 TCP 数据包，`High`（高）将应用这两 种闪避技术。 选择的级别越高，所需时间就越长，不过被检测到的几率也越低（例如 IDS）。

`Application Evasion`（应用程序闪避）这个闪避选项用于基于 DCERPC、SMB 和 HTTP 的入侵。 闪避选项越高级，闪避级别就越高。 `Dry run`（预检）将在任务 和日志中显示入侵信息。 不会进行实际的入侵。

![](img/metasploit 中文操作手册 12636.png)

7.7 点击右下角的![](img/metasploit 中文操作手册 12666.png)来启动入侵任务。

![](img/metasploit 中文操作手册 12680.png)

如果入侵成功，就会在`Session`（会话）选项卡上显示连接的会话和会话编号。

![](img/metasploit 中文操作手册 12726.png)