# 11\. 社会工程

在 Metasploit Pro 中使用`Campaigns`（宣传活动）来启动`Social Engineering`

（社会工程）。 `Campaign`（宣传活动）包括 Web、电子邮件和 USB 模式。 您可以 自定义网页和电子邮件内容来模拟面向客户端的常规钓鱼攻击。 一旦目标客户端连接到

Metasploit Pro 服务器，就可以自动运行客户端入侵模块。

11.1 点击![](img/metasploit 中文操作手册 19190.png)选项卡进入`Campaigns`（宣传活动）页面。

![](img/metasploit 中文操作手册 19221.png)

11.2 点击 ![](img/metasploit 中文操作手册 19227.png) 来新建一个宣传活动。 输入`Campaign Name`（宣传活动名 称）和`Listener Bind IP`（监听程序绑定 IP）。 要启动一个 Web Campaign

（Web 宣传活动），勾选`Start a web server`（启动 web 服务器），输入 URI

（可选）、Web 服务器 IP 和端口。

![](img/metasploit 中文操作手册 19409.png)

11.3 一旦保存完毕，您将看到`Create a web template for this campaign`（为此 宣传活动创建一个 web 模板）页面，因为已在上一步中选择`Start a web server`（启动 web 服务器）。 您可以编辑 HTML 模板。 要克隆一个 URL，请输入 URL 并点击![](img/metasploit 中文操作手册 19569.png)，将显示已克隆 URL 的 HTML 代码。

![](img/metasploit 中文操作手册 19601.png)

11.4 在`Exploit Settings`（入侵设置）部分下，`Start Browser Autopwn`（启 动浏览器 Autopwn）会自动发送相关客户端浏览器和操作系统指纹的入侵。 您可以选 择`Start a specific browser exploit`（启动特定的浏览器入侵）来测试特定的 客户端模块（例如 Aurora）。 或者您可以选择`Don’t start any exploit`（不 启动任何入侵），不执行任何操作。

![](img/metasploit 中文操作手册 19835.png)

11.5 ![](img/metasploit 中文操作手册 19841.png)选择`Start Browser Autopwn`（启动浏览器 Autopwn）。 并点击 ![](img/metasploit 中文操作手册 19932.png)。![](img/metasploit 中文操作手册 19939.png) 可以启动、停止和再次运行宣传活动。点击![](img/metasploit 中文操作手册 19942.png)。

![](img/metasploit 中文操作手册 19936.png)

11.6 点击![](img/metasploit 中文操作手册 19950.png)和`OK`（确定）来启动宣传活动。

![](img/metasploit 中文操作手册 19973.png)

11.7 该宣传活动在`Tasks`（任务）选项卡下运行。

![](img/metasploit 中文操作手册 20012.png)

11.8 尝试从目标客户端的浏览器访问 Web Campaign URL（Web 宣传活动 URL）。 任务 日志将显示入侵信息。

![](img/metasploit 中文操作手册 20084.png)

11.9 如果入侵成功，将构建新的会话。

![](img/metasploit 中文操作手册 20112.png)