# 4\. 发现设备

运行渗透测试的第一步是发现一些目标。 Metasploit 可以使用内置的工具和模块来进行 扫描并发现设备，从其他工具导入结果，对目标网络使用 Nexpose 来发现漏洞。

4.1 前往该项目并点击![](img/metasploit 中文操作手册 6308.png)按钮。 输入要执行发现操作的地址（范围）。

![](img/metasploit 中文操作手册 6335.png)

4.2 点击![](img/metasploit 中文操作手册 6379.png)来自定义扫描。 可以找到 Nmap 参数和端口设置。 将鼠 标移至![](img/metasploit 中文操作手册 6413.png)来获得帮助。 在`Advanced Target Settings`（高级目标设置）下方，

您可以自定义 NMAP 参数、源端口和目标端口。

![](img/metasploit 中文操作手册 6494.png)

4.3 在`Discovery Setttings`（发现设置）下方，可以设置扫描速度和超时来控制网络 使用。 在此处启用的选项越多，扫描就越慢。 如果选择`Dry run`（预检）， Metasploit 不会扫描网络，但是任务日志将显示相关信息。

![](img/metasploit 中文操作手册 6630.png)

4.4 在`Discovery Credentials`（发现凭证）下方，可以设置 SMB 凭证来进行共享并 发现 Windows 网络中的用户名。 在`Automatic Tagging`（自动添加标签）下方， 可以自动添加操作系统标签，这样您便能基于操作系统（例如 Linux）更简便地管理 主机了。

![](img/metasploit 中文操作手册 6792.png)

4.5 点击右下角的`Launch Scan`（启动扫描）。![](img/metasploit 中文操作手册 6848.png) 一旦开始扫描，就会在

`Tasks`（任务）选项卡![](img/metasploit 中文操作手册 6876.png)上显示`Discovering`（发现）任务。

![](img/metasploit 中文操作手册 6902.png)

4.6 除了 Metasploit 发现的设备以外，还可以导入设备列表。 点击项目主页上的

![](img/metasploit 中文操作手册 6953.png)。 将鼠标移至![](img/metasploit 中文操作手册 6966.png)来查看支持的所有文件类型。

![](img/metasploit 中文操作手册 6985.png)

4.7 选择要排除的文件、站点和地址。 点击![](img/metasploit 中文操作手册 7038.png)。

![](img/metasploit 中文操作手册 7045.png)

4.8 Metasploit 可以启动 Nexpose 来执行设备发现操作。 前往`Administration`（管 理）选项卡并点击`Global Settings`（全局设置）。 向下滚动鼠标至`Nexpose Consoles`（Nexpose 控制台）部分并点击![](img/metasploit 中文操作手册 7228.png)。 输入信息来 添加 Nexpose 控制台。

![](img/metasploit 中文操作手册 7268.png)

![](img/metasploit 中文操作手册 7271.png)

4.9 要启动 Nexpose 扫描，在项目主页上点击![](img/metasploit 中文操作手册 7312.png)。 选择新添加的 Nexpose 控制 台，指定 IP/IP 范围和`Scan template`（扫描模板）。 点击右下角的 ![](img/metasploit 中文操作手册 7377.png)来启动扫描。

![](img/metasploit 中文操作手册 7389.png)

4.10 一旦启动 Nexpose 进行扫描，就会在`Tasks`（任务）选项卡上显示相关信息。

![](img/metasploit 中文操作手册 7446.png)

4.11 完成扫描后，主机选项卡`Analysis &gt; Hosts`（分析 > 主机）将显示检测出易 受入侵的主机。

![](img/metasploit 中文操作手册 7522.png)

4.12 最后一栏是主机状态。 这些状态如下所示：

1\. Scanned（已扫描）– 已发现设备。

2\. Cracked（已破解）– 已成功地强力破解凭证，但尚未获取会话。

3\. Shelled（已攻陷）– 已获取设备上打开的会话。

4\. Looted（已掠取）– 已从设备收集证据。

![](img/metasploit 中文操作手册 7675.png)

4.13 如果启动 Nexpose 来发现设备，您可以查看易于受到入侵的漏洞列表。 前往 `Analysis &gt; Vulnerabilities`（分析 > 漏洞）。 将显示找到的所有主机的漏 洞。 点击`reference`（参考）图标来检查来自网络的漏洞信息，这样便能选择特 定的模块来入侵主机了。

![](img/metasploit 中文操作手册 7842.png)

4.14 标签有助于对主机进行分组。 例如：您可以利用相同的标签（例如 `os_windows`）为所有主机生成一份报告。 如果已启用 4.4 中提到的`Automatic Tagging`（自动添加标签）功能，主机表格将显示检测出的操作系统标签。

![](img/metasploit 中文操作手册 8003.png)

4.15 要添加您的标签，请选择主机然后点击 ![](img/metasploit 中文操作手册 8009.png) 。 输入标签名称然后点击 ![](img/metasploit 中文操作手册 8010.png) 。

![](img/metasploit 中文操作手册 8054.png)

![](img/metasploit 中文操作手册 8060.png)

4.16 某些主机可能拥有多个标签。 将鼠标移至`Tags`（标签）栏下方的![](img/metasploit 中文操作手册 8103.png)。 将显 示所有标签。

![](img/metasploit 中文操作手册 8120.png)

4.17 点击 ![](img/metasploit 中文操作手册 8124.png) 以便进入`Tags`（标签）页面。 要删除标签，请将其选定并点击 ![](img/metasploit 中文操作手册 8166.png)。 您可以同时选择和删除多个标签。

![](img/metasploit 中文操作手册 8189.png)

![](img/metasploit 中文操作手册 8195.png)

4.18 要管理标签，请选择一个标签并点击 ![](img/metasploit 中文操作手册 8196.png) 。 您可以更改名称、添加描述并设 置`Report summary`（报告摘要）、`Report detail`（报告详细信息）和 `Critical`（关键）。 这三个标签表示以报告摘要、报告详细信息和关键的发现 结果来描述主机。

![](img/metasploit 中文操作手册 8343.png)

下方是标签`Lab_Machines`的`Report summary`（报告摘要）示例。

![](img/metasploit 中文操作手册 8398.png)