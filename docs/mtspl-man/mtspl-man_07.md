# 6\. 强力攻击

强力攻击这种技术通过尝试不同的凭证来获取访问权限。 一旦使用了正确的用户 ID 和 密码，Metasploit 就能自动登录。

6.1 选择一个项目或主机，然后点击![](img/metasploit 中文操作手册 9832.png)来进入`Bruteforce`（强力攻击）页面。 必要时请编辑目标地址并选择要对其进行强力攻击的服务。

![](img/metasploit 中文操作手册 9889.png)

6.2 `Depth`（深度）级别定义密码的位数和复杂性。

*   Quick（快速）： 简单的默认用户名和密码，小型静态列表

*   默认： 由制造商/ISP 设置的常见密码，已知后门

*   常规： 从扫描数据自动生成凭证，极少视协议而定的用户名和大量常见密码

*   深入： 额外的常见密码词表，不适用于较慢的服务，例如 Telnet 和 SSH。

![](img/metasploit 中文操作手册 10073.png)

6.3 ![](img/metasploit 中文操作手册 10076.png)按钮将允许您自定义`Target`（目标）、`Credential Selection`（凭证选择）、`Playload`（有效载荷）、`Bruteforce Limiter`（强力攻击限制器）、`Credential Generation`（凭证生成）和`Credential Mutation`（凭证变化）。

6.4 在`Targeting`（目标）部分下方，您可以排除目标、设置速度、启用`Dry run`（预检）和详细日志。

*   选择适用于您网络带宽的`Speed`（速度）。

*   `Dry run`（预检）仅生成凭证，但不会强力攻击目标主机。

![](img/metasploit 中文操作手册 10374.png)

6.5 在`Credential Selection`（凭证选择）部分下方，您可以使用以下格式（仅接受 密码或用户名）添加额外的凭证。 您还可以指定 SMB 域。

*   domain/username pass1 pass2 pass3

*   username pass1 pass2 pass3

*   <blank> pass1 pass2 pass3

*   username

![](img/metasploit 中文操作手册 10573.png)

6.6 在`Payload Settings`（有效载荷设置）下方，您可以自定义有效载荷。 通常使用 默认值。 请参阅 5.1 有效载荷获得更多信息。

![](img/metasploit 中文操作手册 10693.png)

![](img/metasploit 中文操作手册 10696.png)

6.7 在`Bruteforce Limiters`（强力攻击限制器）部分下，设置一些参数的限制，例如 每个用户、服务的密码猜测最多次数。 `Automatically open sessions with guessed credentials`（使用猜测的凭证自动打开会话）将利用成功的验证（例如 SSH）构建会话。

![](img/metasploit 中文操作手册 10874.png)

6.8 `Credential Generation`（凭证生成）和`Credential Mutation`（凭证变化） 允许您微调 Metasploit 生成凭证的方式。

![](img/metasploit 中文操作手册 10980.png)

![](img/metasploit 中文操作手册 10986.png)

6.9 点击![](img/metasploit 中文操作手册 11018.png)来启动强力攻击任务。 该 GUI 显示处于强力攻击下的服务（例 如 SMB）、使用的凭证和结果。

![](img/metasploit 中文操作手册 11073.png)

6.10 如果对目标进行了成功的强力攻击，`Sessions`（会话）选项卡将显示连接的会 话。

![](img/metasploit 中文操作手册 11129.png)

6.11 要使用您自己的词典，请点击 ![](img/metasploit 中文操作手册 11135.png)进入`Bruteforce`（强力攻击）页面。

点击左上角的![](img/metasploit 中文操作手册 11186.png)。 您可以在新页面中上传您的凭证文件。 将鼠标 移至![](img/metasploit 中文操作手册 11213.png)来获取有关文件格式的更多信息。

![](img/metasploit 中文操作手册 11234.png)

6.12 在`Bruteforce`（强力攻击）页面上，为`Depth`（深度）选择`imported only`（仅导入）。 必要时在发动强力攻击之前进行其他高级配置。

![](img/metasploit 中文操作手册 11326.png)

6.13 要检查从强力攻击收集而来的凭证，请前往`Analysis &gt; Hosts`（分析 > 主 机）并从表格中选择主机。 `Credentials`（凭证）选项卡将显示强力攻击找到的 凭证，以及猜测情况。

![](img/metasploit 中文操作手册 11459.png)