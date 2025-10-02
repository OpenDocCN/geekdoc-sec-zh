# WEB 应用安全测试备忘单

> 原文：[Web Application Security Testing Cheat Sheet](https://www.owasp.org/index.php/Web_Application_Security_Testing_Cheat_Sheet)
> 
> 来源：[WEB 应用安全测试备忘单](http://cheatsheets.hackdig.com/?8.htm)

## 介绍

这个备忘单是一个对 WEB 应用程序执行黑盒测试的任务清单。

## 目的

这个清单可以当成有经验的测试老手的备忘录、结合 OWASP 测试指南一起使用。清单将与测试指南 v4 一起更新（[`www.owasp.org/index.php/OWASP_Testing_Guide_v4_Table_of_Contents`](https://www.owasp.org/index.php/OWASP_Testing_Guide_v4_Table_of_Contents)）。

我们希望把这个备忘单做成 XML 文档，这样就可以使用脚本来将其转换成各种格式，如 pdf、Media Wki、HTML 等，这样同样使得将文档转换为某种打印格式变得容易。

感谢所有给予反馈和帮助的人，如果你有任何意见或建议，欢迎提出并加入到编辑队伍中来。

## 检查列表

### 收集信息

手动访问站点

使用爬虫来抓取（手工）无法访问或隐藏的内容

检查泄露信息的文件，如 robots.txt, sitemap.xml, .DS_Store

检查主要的搜索引擎索引的此站点的公开内容

检查不同的浏览器 UA 获取的内容的差异（如使用爬虫的 UA 访问手机站点）

检查 WEB 应用程序的指纹（Fingerprinting）

确认使用的技术

确认用户角色

确认应用程序的入口地址

确认客户端代码

确认不同的版本的差异（如 web, mobile web, mobile app, web services）

确认位于同一主机或业务相关的应用程序

确认所有的主机名和端口

确认第三方的托管内容

### 配置管理

检查常用的应用程序和管理 URL

检查旧文件、备份文件和未引用文件是否存在

检查支持的 HTTP 方法和 XST 漏洞（[`www.hackdig.com/?01/hack-11.htm`](http://www.hackdig.com/?01/hack-11.htm)）

检查对文件后缀的处理

检查安全 HTTP 头（如 CSP, X-Frame-Options, HSTS，见[`www.hackdig.com/?07/hack-4958.htm`](http://www.hackdig.com/?07/hack-4958.htm)）

测试安全策略（如 Flash, Silverlight, robots）

在线上环境测试非生产数据或做相反的操作

检查客户端代码中的敏感信息（如 API keys,凭据等）

### 安全传输

检查 SSL 版本、算法和密钥长度

检查数字证书有效性

检查凭据是否只通过 HTTPS 传输数据

检查登陆表单是否只通过 HTTPS 传输数据

检查会话令牌是否只通过 HTTPS 传输

检查是否使用了 HSTS

### 认证

测试枚举用户

测试认证绕过

测试暴力破解保护

测试密码规则的质量

测试记住密码功能

测试密码表单的自动完成的功能

测试密码重置和找回

测试密码修改流程

测试验证码

测试多因子认证

测试注销功能

测试 HTTP 的缓存管理（如 Pragma, Expires, Max-age）

测试默认登陆账号

测试用户认证历史

测试账号锁定和密码修改成功的通知渠道

测试跨应用程序共享模式/SSO 的一致性

### 会话管理

确定应用程序管理会话的方式（如将 cookie tokens、url 中的 token）

检查会话 cookie 的标示(httpOnly 和 secure)

检查会话 cookie 的返回（path 和 domain）

检查会话 cookie 的有效期（expires 和 max-age）

检查会话 cookie 的过期失效

检查会话 cookie 的相对超时失效

检查会话 cookie 退出后失效

测试用户是否可以同时拥有多个会话

测试会话 cookie 的随机性

确认会话令牌在登陆、角色变化和退出时的更新

测试跨应用共享 session 会话的一致性

测试会话过载（未限制会话应用范围，见：[`www.owasp.org/index.php/Testing_for_Session_puzzling_(OTG-SESS-010)`](https://www.owasp.org/index.php/Testing_for_Session_puzzling_(OTG-SESS-010))）

测试是否存在 CSRF 和点击劫持漏洞

### 授权

测试路径遍历

测试绕过授权

测试垂直访问控制问题

测试水平访问控制问题

测试授权检查缺失

### 数据验证

测试反射型 XSS

测试存储型 XSS

测试 DOM 型 XSS

测试 CSF（flash XSS）

测试 HTML 注入

测试 SQL 注入

测试 LDAP 注入

测试 ORM 注入

测试 XML 注入（[`www.hackdig.com/?03/hack-8921.htm`](http://www.hackdig.com/?03/hack-8921.htm)）

测试 XXE 注入

测试 SSI 注入([`www.hackdig.com/?01/hack-7955.htm`](http://www.hackdig.com/?01/hack-7955.htm))

测试 XPath 注入

测试 XQuery 注入

测试 IMAP/SMTP 注入

测试 Code 注入

测试 EL 注入（[`www.owasp.org/index.php/Expression_Language_Injection`](https://www.owasp.org/index.php/Expression_Language_Injection)）

测试 Command 注入

测试 Overflow （堆, 栈和整形溢出）

测试 Format String（错误的字符串格式化）

测试 incubated vulnerabilities（缺陷孵化）

测试 HTTP Splitting/Smuggling（协议层）

测试 HTTP Verb Tampering（权限干涉）

测试 Open Redirection

测试本地文件包含

测试远程文件包含

比较客户端与服务端的验证规则

测试 NoSQL 注入

测试 HTTP 参数污染

测试自动绑定（auto-binding:[`click.apache.org/docs/user-guide/html/ch02s03.html`](https://click.apache.org/docs/user-guide/html/ch02s03.html)）

测试 Mass Assignment（见 ror 经典漏洞，[`blog.xdite.net/posts/2012/03/05/github-hacked-rails-security/`](http://blog.xdite.net/posts/2012/03/05/github-hacked-rails-security/)）

测试 NULL/Invalid Session Cookie

### 拒绝服务

测试反自动化/机器请求

测试账号锁定

测试 HTTP 协议 DoS

测试 SQL 通配符 DoS/sleep Dos

### 业务逻辑

测试功能滥用

测试缺乏不可否认性（非对称加密作用）

测试信任关系

测试数据完整性

测试指责分离

### 密码学

检查应加密数据是否加密

根据上下文检查是否使用了错误的算法

检查使用弱算法

检查是否合理使用盐

检查随机函数（的随机性）

### 风险功能—文件上传

检查可接受的文件类型是否在白名单内

检查文件尺寸限制、上传频率和总文件数的阈值与限制情况

检查文件内容是否与定义的文件类型相符

检查所有上传的文件都经过杀毒软件扫描

检查不安全的文件名是否经过处理

检查不能在 web 根目录下直接访问上传文件

检查上传的文件是否存储在相同的主机名和端口

检查文件和其他媒体继承了身份验证和授权功能

### 风险功能—支付信息

测试 WEB 服务器或应用程序是否存在已知漏洞和配置问题

测试默认或易被猜到的密码

测试生产环境的非生产数据或做相反的测试

测试注入漏洞

测试缓冲区溢出

测试不安全的加密存储

测试传输层保护不足

测试不适当的错误处理

测试 CVSS v2 评分> 4.0 的全部漏洞

测试身份验证和授权的问题

测试 CSRF

### HTML 5

测试 WEB 消息传递

测试 WEB 本地存储 SQL 注入

检查 CORS 的实现

检查离线的 WEB 应用程序

## 其他格式

DradisPro 模板格式 [on github](https://github.com/raesene/OWASP_Web_App_Testing_Cheatsheet_Converter/blob/master/OWASP_Web_Application_Testing_Cheat_Sheet.xml)

Asana 在[Templana](http://templana.com/templates/owasp-website-security-checklist/)的格式 （感谢 Bastien Siebman）

## 作者与主编

[Simon Bennetts](https://www.owasp.org/index.php/User:Simon_Bennetts)

[Rory McCune](https://www.owasp.org/index.php/User:Raesene)

Colin Watson

Simone Onofri

包括 Testing Guide v3 的全部作者

## 其他贡献者

[Ryan Dewhurst](https://www.owasp.org/index.php/User:Ryan_Dewhurst)

[Amro AlOlaqi](https://www.owasp.org/index.php/User:Amro_Ahmed)

翻译 TaoGOGO