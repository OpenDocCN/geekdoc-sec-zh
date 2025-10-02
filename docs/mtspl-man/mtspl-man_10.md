# 9\. Web 应用程序测试

Metasploit Pro 在`Web Apps`（Web 应用程序）选项卡下提供 Web 扫描、Web 审 计和 Web 入侵功能。 这些功能帮助您在处于活动状态的 Web 内容和表单中搜索漏洞 并进行入侵。

*   Web 应用程序扫描： 对网页启用爬虫、搜索表单和活动内容

*   Web 应用程序审计： 搜索这些表单中的漏洞

*   Web 应用程序入侵： 入侵找到的漏洞

9.1 点击![](img/metasploit 中文操作手册 16532.png)选项卡前往`Web Apps`（Web 应用程序）页面。

![](img/metasploit 中文操作手册 16566.png)

9.2 要运行 web 扫描，请前往`Web Apps &gt; WebScan`或`Project &gt; WebScan`（项目 > WebScan）。

![](img/metasploit 中文操作手册 16653.png)

9.3 输入条件，包括`Seed URLs`（种子 URL）、`Max # pages requested`（请求的 最大页数）、`Max amount of time`（最长时间）、`# of concurrent requests`、`allowed per website`（每个网站允许的并发请求数量）。 从过去的扫描中选择已 识别的 web 服务。

![](img/metasploit 中文操作手册 16658.png)

![](img/metasploit 中文操作手册 16841.png)

9.4 要为 HTTP 基本验证和 cookie 输入凭证，请点击![](img/metasploit 中文操作手册 16914.png)按钮。

![](img/metasploit 中文操作手册 16920.png)

9.5 点击![](img/metasploit 中文操作手册 16955.png)按钮来启动 web 扫描。 扫描后，将在`Web Apps`（Web 应用程 序）选项卡上显示找到的 Web 应用程序。 这些信息包括 IP 地址、网站 URL、服务 名称、页数和表单数量。

![](img/metasploit 中文操作手册 17056.png)

9.6 要运行 Web 审计来搜索已扫描表单中的漏洞，请点击 ![](img/metasploit 中文操作手册 17060.png) 按钮。 输入您 的条件，包括`Max # requests sent to target application form`（发送至目标 应用程序表单的最大请求数量）、`Max amount of time per form`（每个表单的最 长时间）、`Max # of unique form instances`（唯一表单实例的最大数量）、 `Credential information`（凭证信息）和`User agent`（用户代理程序）。

![](img/metasploit 中文操作手册 17350.png)

9.7 您也可以选择`Target Web App`（目标 Web 应用程序）。

![](img/metasploit 中文操作手册 17398.png)

9.8 点击 ![](img/metasploit 中文操作手册 17404.png) 按钮来启动 web 审计。 将在`Web Apps`（Web 应用程序）选项 卡上更新 Web 审计结果，例如漏洞数量。 点击`Risk`（风险）图标，`Vulns`

（漏洞）数量或`Web Site`（网站）URL 来检查 web 应用程序的漏洞列表。

![](img/metasploit 中文操作手册 17545.png)

9.9 将通过以下信息显示具有漏洞的 web 应用程序，包括漏洞类型（例如 SQL 注入）、 路径、机密性级别、所用方式、参数和证据。

![](img/metasploit 中文操作手册 17626.png)

9.10 要通过漏洞入侵 web 应用程序，请点击漏洞详细信息的`Path`（路径）。 例 如：选择一个属于 XSS 类别的漏洞。 利用`VULNERABLE`这个单词定位登录字段， 并通过在`VULNERABLE`后添加一些信息来编辑该字段。

![](img/metasploit 中文操作手册 17761.png)

9.11 点击 ![](img/metasploit 中文操作手册 17770.png) 。该浏览器将显示已被注入的网页。

![](img/metasploit 中文操作手册 17800.png)