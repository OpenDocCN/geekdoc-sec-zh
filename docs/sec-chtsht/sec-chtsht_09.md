# REST 安全备忘单

> 原文：[Cryptographic Storage Cheat Sheet](https://www.owasp.org/index.php/Cryptographic_Storage_Cheat_Sheet)
> 
> 来源：[REST 安全备忘单](http://cheatsheets.hackdig.com/?6.htm)

百度 dor 加入翻译项目，并翻译此章节

## 简介

REST 或 REpresentational State Transfer 是通过 URL 元素 系统的表达特定的实体 ，REST 是一个以架构风格来构建 Web 顶层服务而不是一个架构。REST 是通过使用简单的 URL 用基于 web 系统的交互而不是通过复杂的 http 请求实体 或者 post 参数 从系统中请求特定的条目。本文是帮助基于 REST 服务最佳实践的向导（而不是详尽的手册）

## 认证和会话管理

RSET 风格的 Web 服务应该（should）使用基于会话的认证，使用通过 POST 请求 或使用作为 POST 请求体中的参数的 API key 或 cookie 来建立的会话 。 用户名（usernames）和密码(passwords) , 会话标识符（session tokens） 和 API keys 都不应该（should not）出现在 URL 中，因为这样做的话 web 服务器的日志系统能够捕获这些值并且他们具有内在的价值。

如下面的两个 url 是合理的

*   [`example.com/resourceCollection//action`](https://example.com/resourceCollection//action)

*   [`twitter.com/vanderaj/lists`](https://twitter.com/vanderaj/lists)

下面的就是不合理的

*   [`example.com/controller//action?apiKey=a53f435643de32`](https://example.com/controller//action?apiKey=a53f435643de32) (API Key 在 URL 中)

*   [`example.com/controller//action?apiKey=a53f435643de32`](http://example.com/controller//action?apiKey=a53f435643de32) ( API Key 在 URL 中 而且没有使用 https 加密传输 )

## 保护会话状态

很多 web services 都被尽可能的写成无状态的。This usually ends up with a state blob being sent as part of the transaction.

*   考虑到仅仅使用 session token 或者 api key 在服务端的缓存中维系客户端的状态 。这是很多 web app 的做法，这也是比较安全的原因。

*   反重放(Anti-replay ) 攻击者能够通过剪贴和复制报文伪装成某个用户。考虑到使用有时间限制的加密秘钥 加密 session token 或者 api key 日期 时间 和 来源 IP ,总的来说 对本地客户端存储的认证 token(authentication token)做的保护 能够 避免一些 重放攻击的

*   不要让它容易解密和改变内部状态比它应该做得更好

简短的说，即使你有个一个 brochureware web 站点 ， 你不要放入到 [`example.com/users/2313/edit?isAdmin=false&debug=false&allowCSRPanel=false`](https://example.com/users/2313/edit?isAdmin=false&debug=false&allowCSRPanel=false) 这个 url 中，这样你将很快以有很多 、 管理员(admins ) 桌面使用帮助用户(help desk helpers ) 以及 开发者（developers） 结束

## Authorization(授权)

### Anti-farming

就像比价站点又或像一些聚合站点兴起一样 ， 很多 REST 风格的的 web 服务兴起 并且提供服务 。由于没有技术手段阻止它的使用,所以强烈考 通过提供高速服务（farming）作为一种商业模式来激励使收费成为可能 或者通过合约限制服务的使用的条目和条件。CAPTHAs 和 一些相似的方法能够减少一些简单的攻击行为，当是这个不能阻止一些完备机构或者有很强技术能力的攻击 。使用双向认证的客户端 TLS 也许是一个限制授信组织访问的一种途径，但是这并不能保证万无一失 尤其是当凭证被人为的重放或者由于互联网事故造成的重放。

### Protect HTTP methods

REST 风格的 API 一般使用 GET(读) POST（创建） PUT(替换/更新) 和 DELETE(删除一条记录) 这几种 http 请求. 对于每一个单独的资源集合、用户或者动作 并不是所有的这些方法都是可用的。确保对会话（session）的 token/API key 和相关联的资源集合、动作和记录对于请求的 HTTP 方法是可用的.例如 你有一个关于图书馆 REST 风格的 API ，允许一个匿名的用户删除书的目录实体的做法是不合适的,但是不管是图书管理员还是匿名用户 允许他们获得图书的目录实体是没有不妥的地方的。

### Whitelist Allowable Methods (白名单方法)

在同一个实体上对给定的一个 URL 允许多种方法进行不同的操作在 REST 风格的服务是很常见的。 例如，一个 GET 的请求也许是读一个实体 然而 当请求为 PUT 方法是就是更新一个已经存在的实体, 当请求是 POST 方法时 将创建一个实体 , 请求为 DELETE 方法时将删除一个已经存在的实体. 适当的限制可允许的动作以便只有被允许的动作才能正常运行而其他动作都将返回一个适当的状态码（如 403 禁止访问 ） 这点对服务(service)来说很重要 。

尤其在 Java EE 中 要实现这点比较困难。可以参看 Bypassing Web Authentication and Authorization with HTTP Verb Tampering 对常见配置的说明

### Protect privileged actions and sensitive resource collections( 保护私有的方法和敏感的资源集合)

并不是所有的用户都能访问所有的 web service . 不想让一个管理 web 的 services 被滥用 这是至关重要的:

*   [`example.com/admin/exportAllData`](https://example.com/admin/exportAllData)

会话 token 或 API　key 应该被单独的作为 cookie 或 请求体的参数（body parameter）来发送 用来确保 私有集合或者私有方法对没有授权的使用是被适当的保护的

### Protect against cross-site request forgery( 保护伪造的跨域请求)

通过 REST 风格的 web services 暴露的资源要确保任何 PUT POST　DELETE 方法的请求对伪造的跨站点请求是被保护的 这点很重要。通过基于 token 的访问 是一个典型的案例。

你可以通过查看 [Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%40CSRF%41_Prevention_Cheat_Sheet) 来获得更多的关于如何防范 CSRF 的信息。

如果在里的应用中存在任何的 XSS 即使使用随机的 tokens CSRF 还是很容易实现的。所以请确保你已经知道如何防止 XSS 。

### Insecure Direct object references ( 不安全的直接对象引用 )

这个也许看起来很明显，但是如果你有一个银行账户的 REST 的 WEB 服务 ，你必须确保有对主键和外键有足够的认证 ：

*   [`example.com/account/325365436/transfer?amount=$100.00&toAccount=473846376`](https://example.com/account/325365436/transfer?amount=$100.00&toAccount=473846376)

在这个例子中 ， 将钱从任何一个账户中转账到另外一个账户中是可能的，这明显是很愚蠢的。这个例子中甚至没有一个随机的 token 来确保安全

*   [`example.com/invoice/2362365`](https://example.com/invoice/2362365)

在这个例子中，获取所有单据的拷贝时有可能的。

请确保你懂得怎么防护在 OWASP (2010 年公布的 )中前 10 的 [insecure direct object references]（[`www.owasp.org/index.php/Top_10_2010-A4-Insecure_Direct_Object_References`](https://www.owasp.org/index.php/Top_10_2010-A4-Insecure_Direct_Object_References) ( 不安全的直接对象引用 )

## Input Validation ( 输入验证 )

### Input validation 101 (输入验证 101 )

除了将你知道所有到 R 的输入验证应用 ESTful 的 web 服务中去 还有添加额外 10%验证 ，因为自动化的工具能够在数小时内很简单的模糊你的接口 高速的结束。所以：

*   Assist the user > Reject input > Sanitize (filtering) > No input validation

协助用户是最有意义的, 在很多场景中的情况是 问题存在于键盘和电脑两者之间(PRBACK) 。帮助用户输入高质量的数据到你的 web 服务上，例如 一个有效的邮政编码对一个被提供的地址是有意义的，或者一个有意义的日期。 如果不是 拒绝这个输入。如果他们继续 或者 文本框 又或其他难以验证的领域 ，过滤输入也许是不合算的但是对防止 XSS 和 SQL 注入 攻击是有帮助的。如果你已经减少了对输入的过滤或者对输入不做验证了 ，确保输入编码对你的应用是非常强壮的。

要记录输入验证失败的条目，尤其如果你假设你写的客户端的编码将调用你的 web 服务。事实上是任何人都能调用你的 web 服务 ， 假设有个用户每秒执行上百次的输入验证的失败不是一个好的现象。考虑到在一定的数字每小时或每天速率限制 API 请求 用来防止 API 的滥用。

### Secure parsing

用安全解析器来分析请求消息。如果你使用的是 XML , 确保使用的解析器是不容易被 XXE attacks(XXE 攻击的)。

### Strong typing( 强类型 )

如果仅仅允许的指是 true 或者 false 或者 是一个 数字 或者 小的可接受的数字 ，那执行大部分的攻击是困难的 。尽快的使用强类型的输入数据。

### Validate Incoming Content-Types ( 验证输入内容类型 )

当 POST 或者 PUT　新数据时，客户端将指定　Content-Type( 例如：application/xml or application/json ) 。客户端应当不假设 Content-Type , 但是需要检测头部设定的 Content-Type 和 实际内容是否一致。 一个没有指定 Content-Type 的头部 或者一个 非预期 Content-Type 的头部 将导致服务器返回一个 406 不被接受 内容拒绝的响应。

### Validate Response Types ( 验证响应类型 )

对于 REST 服务允许多个响应类型是很常见的。（例如：application/xml 或者 application/json 和 客户端通过请求头部指定的响应类型的优先顺序 ）不要简单的拷贝接受头部到响应头部的 Content-type 。如果接受头部和指定的允许的类型不一致将拒绝请求（ 返回 406 不被可接受的 响应）。

因为对于典型的响应类型有很多 MIME 类型，对于客户端的文档指定哪些 MIME 的类型应当被使用是很重要的。

### XML Input Validation(XML 输入验证 )

基于 XML 的服务必须确保对通过使用安全的 XML 解析器对常见的基于 XML 攻击有防护能力。这通常以为着需要防 XML External Entity 攻击，XML-signature wrapping 等。

对于这些攻击可以参看[`ws-attacks.org`](http://ws-attacks.org)

## Output Encoding (输入编码)

### Send security headers( 发送安全的头部)

为了确保给定资源的内容能够被浏览器正确的解析，服务器应当总是发送 正确的 Content-Type 的头部 而且正确的 Content-Type 头部应当包含字符设置 。服务器还应当发送 X-Content-Type-Options：无探测确保了浏览器不能尝试决定不同的 Content-Type 而是实际发送的（能导致 XSS）。

此外 客户端应当发送 X-Frame-Options：防止在较老的浏览器中拖拽“点击劫持”攻击

### JSON encoding ( json 编码 )

阻止任意的远端 javascript 代码在浏览器中执行或者如果你在使用 node.js 阻止其在服务器上执行 是 json 编码者一个主要的关心的地方。使用一个合适的 JSON 序列化器去编码用户提供的数据以阻止用户提供的输入在浏览器中执行 是很重要的。

当向浏览器的ＤＯＭ树种插入一个值时，强烈建议使用 .value/.innerText/.textContent 而不是使用 .innerHTML ,这样能避免简单的 DOM XSS 攻击。

### XML encoding( XML 编码)

XML 应当不使用连接的字符串来建立。应当总是使用 XML 序列化器来构造 。这能保证发送到浏览器的 XML 是可解析的 而且不包含 XML 注入。如果需要查看更多的信息可以参看 Web Service Security Cheat Sheet

## Cryptography ( 加密 )

### Data in transit( 数据传输 )

除了完全是对公众公开的信息 其他都应该使用 TLS , 尤其是有证书 更新 删除 和 其他任何传输的内容 都应该使用 TLS 来传输。在现代的硬件下 TLS 的花费是微不足道的 ，这点微不足道成本远远低于因为安全对终端用户的赔偿。

对高度隐私的 web 服务 可以考虑使用相互认证的客户端凭证来提供额外的保护。

### Data in storage( 存储数据 )

当谈到正确的存储敏感或受管制的数据时，任何一个 web 应用程序都应该建议使用领先的做法。关于更多信息可以参看 OWASP Top 10 2010 - A7 Insecure Cryptographic Storage.

## Related Articles

OWASP Cheat Sheets Project Homepage

*   [OWASP Cheat Sheet Series](https://www.owasp.org/index.php/OWASP_Cheat_Sheet_Series "OWASP Cheat Sheet Series")

Developer Cheat Sheets (Builder)

*   [Authentication Cheat Sheet](https://www.owasp.org/index.php/Authentication_Cheat_Sheet "Authentication Cheat Sheet")
*   [Choosing and Using Security Questions Cheat Sheet](https://www.owasp.org/index.php/Choosing_and_Using_Security_Questions_Cheat_Sheet "Choosing and Using Security Questions Cheat Sheet")
*   [Clickjacking Defense Cheat Sheet](https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet "Clickjacking Defense Cheat Sheet")
*   [C-Based Toolchain Hardening Cheat Sheet](https://www.owasp.org/index.php/C-Based_Toolchain_Hardening_Cheat_Sheet "C-Based Toolchain Hardening Cheat Sheet")
*   [Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%40CSRF%42_Prevention_Cheat_Sheet)
*   [Cryptographic Storage Cheat Sheet](https://www.owasp.org/index.php/Cryptographic_Storage_Cheat_Sheet "Cryptographic Storage Cheat Sheet")
*   [DOM based XSS Prevention Cheat Sheet](https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet "DOM based XSS Prevention Cheat Sheet")
*   [Forgot Password Cheat Sheet](https://www.owasp.org/index.php/Forgot_Password_Cheat_Sheet "Forgot Password Cheat Sheet")
*   [HTML5 Security Cheat Sheet](https://www.owasp.org/index.php/HTML5_Security_Cheat_Sheet "HTML5 Security Cheat Sheet")
*   [Input Validation Cheat Sheet](https://www.owasp.org/index.php/Input_Validation_Cheat_Sheet "Input Validation Cheat Sheet")
*   [JAAS Cheat Sheet](https://www.owasp.org/index.php/JAAS_Cheat_Sheet "JAAS Cheat Sheet")
*   [Logging Cheat Sheet](https://www.owasp.org/index.php/Logging_Cheat_Sheet "Logging Cheat Sheet")
*   [.NET Security Cheat Sheet](https://www.owasp.org/index.php/.NET_Security_Cheat_Sheet ".NET Security Cheat Sheet")
*   [OWASP Top Ten Cheat Sheet](https://www.owasp.org/index.php/OWASP_Top_Ten_Cheat_Sheet "OWASP Top Ten Cheat Sheet")
*   [Password Storage Cheat Sheet](https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet "Password Storage Cheat Sheet")
*   [Pinning Cheat Sheet](https://www.owasp.org/index.php/Pinning_Cheat_Sheet "Pinning Cheat Sheet")
*   [Query Parameterization Cheat Sheet](https://www.owasp.org/index.php/Query_Parameterization_Cheat_Sheet "Query Parameterization Cheat Sheet")
*   [Ruby on Rails Cheatsheet](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet "Ruby on Rails Cheatsheet")
*   REST Security Cheat Sheet
*   [Session Management Cheat Sheet](https://www.owasp.org/index.php/Session_Management_Cheat_Sheet "Session Management Cheat Sheet")
*   [SQL Injection Prevention Cheat Sheet](https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet "SQL Injection Prevention Cheat Sheet")
*   [Transport Layer Protection Cheat Sheet](https://www.owasp.org/index.php/Transport_Layer_Protection_Cheat_Sheet "Transport Layer Protection Cheat Sheet")
*   [Unvalidated Redirects and Forwards Cheat Sheet](https://www.owasp.org/index.php/Unvalidated_Redirects_and_Forwards_Cheat_Sheet "Unvalidated Redirects and Forwards Cheat Sheet")
*   [User Privacy Protection Cheat Sheet](https://www.owasp.org/index.php/User_Privacy_Protection_Cheat_Sheet "User Privacy Protection Cheat Sheet")
*   [Web Service Security Cheat Sheet](https://www.owasp.org/index.php/Web_Service_Security_Cheat_Sheet "Web Service Security Cheat Sheet")
*   [XSS (Cross Site Scripting) Prevention Cheat Sheet](https://www.owasp.org/index.php/XSS_%40Cross_Site_Scripting%42_Prevention_Cheat_Sheet)

Assessment Cheat Sheets (Breaker)

*   [Attack Surface Analysis Cheat Sheet](https://www.owasp.org/index.php/Attack_Surface_Analysis_Cheat_Sheet "Attack Surface Analysis Cheat Sheet")
*   [XSS Filter Evasion Cheat Sheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet "XSS Filter Evasion Cheat Sheet")
*   [REST Assessment Cheat Sheet](https://www.owasp.org/index.php/REST_Assessment_Cheat_Sheet "REST Assessment Cheat Sheet")

Mobile Cheat Sheets

*   [IOS Developer Cheat Sheet](https://www.owasp.org/index.php/IOS_Developer_Cheat_Sheet "IOS Developer Cheat Sheet")
*   [Mobile Jailbreaking Cheat Sheet](https://www.owasp.org/index.php/Mobile_Jailbreaking_Cheat_Sheet "Mobile Jailbreaking Cheat Sheet")

OpSec Cheat Sheets (Defender)

*   [Virtual Patching Cheat Sheet](https://www.owasp.org/index.php/Virtual_Patching_Cheat_Sheet "Virtual Patching Cheat Sheet")

Draft Cheat Sheets

*   [Access Control Cheat Sheet](https://www.owasp.org/index.php/Access_Control_Cheat_Sheet "Access Control Cheat Sheet")
*   [Application Security Architecture Cheat Sheet](https://www.owasp.org/index.php/Application_Security_Architecture_Cheat_Sheet "Application Security Architecture Cheat Sheet")
*   [Business Logic Security Cheat Sheet](https://www.owasp.org/index.php/Business_Logic_Security_Cheat_Sheet "Business Logic Security Cheat Sheet")
*   [PHP Security Cheat Sheet](https://www.owasp.org/index.php/PHP_Security_Cheat_Sheet "PHP Security Cheat Sheet")
*   [Secure Coding Cheat Sheet](https://www.owasp.org/index.php/Secure_Coding_Cheat_Sheet "Secure Coding Cheat Sheet")
*   [Secure SDLC Cheat Sheet](https://www.owasp.org/index.php/Secure_SDLC_Cheat_Sheet "Secure SDLC Cheat Sheet")
*   [Threat Modeling Cheat Sheet](https://www.owasp.org/index.php/Threat_Modeling_Cheat_Sheet "Threat Modeling Cheat Sheet")
*   [Web Application Security Testing Cheat Sheet](https://www.owasp.org/index.php/Web_Application_Security_Testing_Cheat_Sheet "Web Application Security Testing Cheat Sheet")
*   [Grails Secure Code Review Cheat Sheet](https://www.owasp.org/index.php/Grails_Secure_Code_Review_Cheat_Sheet "Grails Secure Code Review Cheat Sheet")
*   [IOS Application Security Testing Cheat Sheet](https://www.owasp.org/index.php/IOS_Application_Security_Testing_Cheat_Sheet "IOS Application Security Testing Cheat Sheet")
*   [Key Management Cheat Sheet](https://www.owasp.org/index.php/Key_Management_Cheat_Sheet "Key Management Cheat Sheet")
*   [Insecure Direct Object Reference Prevention Cheat Sheet](https://www.owasp.org/index.php/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet "Insecure Direct Object Reference Prevention Cheat Sheet")
*   [Content Security Policy Cheat Sheet](https://www.owasp.org/index.php/Content_Security_Policy_Cheat_Sheet "Content Security Policy Cheat Sheet")

## Authors and Primary Editors

Erlend Oftedal - erlend.ofted<wbr class="calibre28">al@owasp.org

Andrew van der Stock - vanderaj@owa<wbr class="calibre28">sp.org