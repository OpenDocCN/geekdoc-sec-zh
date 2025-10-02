# 点击劫持防御备忘单

> 原文：[Clickjacking Defense Cheat Sheet](https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet)
> 
> 来源：[点击劫持防御备忘单](http://cheatsheets.hackdig.com/?1.htm)

此备忘单的主要目的是为开发者提供点击劫持/UI 纠正攻击的防御指导。

抵御“点击劫持”攻击的最普遍的方法是通过各种形式的“嵌入阻断”功能，防止攻击者通过 iframe 将你的站点嵌入他们的页面。本备忘单将讨论实现嵌入阻断的两种方式：第一种是 X-Frame-Options 头信息（可能有些浏览器不支持）；第二种方式是使用 javascript 编写嵌入阻断代码。

## 使用 X-Frame-Options 响应头信息来防御

X-Frame-Options HTTP 响应头能用来向浏览器指明是否允许渲染在或标签内的页面。网站可以用它来确保网站内容不被嵌入到其他站点，从而避免遭受点击劫持攻击。

## X-Frame-Options 响应头类型

X-Frame-Options 头信息有三种可用的值

*   **DENY**，阻止所有的域名嵌入此页面。.

*   **SAMEORIGIN**，只允许当前站点嵌入此页面。

*   **ALLOW-FROM uri**，允许指定的“URI”嵌入此页面（如 ALLOW-FROM [`www.example.com`](http://www.example.com) ，则允许 www.example.com 嵌入此页面）。ALLOW-FROM 选项是 2012 年左右添加的, 所以一些旧浏览器可能不支持这个参数。**不要依赖 ALLOW-FROM 参数。**如果你使用这个选项，但是浏览器不支持它，那相当于你没有做任何点击劫持防御。

## 浏览器支持

下面的浏览器支持 X-Frame-Options 头信息。

| **浏览器** | **引入 DENY/SAMEORIGIN 支持** | **引入 ALLOW-FROM 支持** |
| --- | --- | --- |
| Chrome | 4.1.249.1042 |  |
| Firefox (Gecko) | 3.6.9 (1.9.2.9) | 18.0 |
| Internet Explorer | 8.0 | 9.0 |
| Opera | 10.50 |  |
| Safari | 4.0 | Not supported/Bug reported |

引用:

Mozilla Developer Network

IETF Draft

X-Frame-Options Compatibility Test - 从这个页面可以获得浏览器 X-Frame-Options 头信息的支持情况

## 实现

要实现这种保护，你需要在想要保护（免受点击劫持攻击）的页面上加上头信息。一种方法是在每个页面上都加上这种头信息。另一种更简单的方法是实现一个过滤器，自动的为每一个页面添加头信息。

OWASP 有篇文章和一些代码，提供了在 Java EE 环境中，有关防护方法的详细实现。SDL 博客发表了一篇文章，设计了在.NET 环境下实现防护的方式。

## 局限

## 每个页面的政策规范 Per-page policy specification

如果每个页面都需要指定政策，那么部署会变得非常复杂。为整个网站提供约束能力（如在登陆的时候）能够简化实现的过程。The policy needs to be specified for every page, which can complicate deployment. Providing the ability to enforce it for the entire site, at login time for instance, could simplify adoption.

## 多域名网站的问题 Problems with multi-domain sites

当前的实现不允许网站管理员提供允许嵌入本页面的域名白名单。虽然白名单列表是危险的，在一些特殊情况下，网站管理员可能除了使用多个主机名外，没有其他选择。The current implementation does not allow the webmaster to provide a whitelist of domains that are allowed to frame the page. While whitelisting can be dangerous, in some cases a webmaster might have no choice but to use more than one hostname.

## 代理

Web 代理会添加或去掉头信息。如果一个 web 代理去掉 X-Frame-Options 头信息，网站将失去这种嵌入保护。

脚本是支持遗留浏览器防止被嵌入的最好解决方案

一种抵御点击劫持的方法是在每个不允许被嵌入的页面包含“frame-breaker”脚本。下面的方法能防止一个网页被嵌入（甚至在不支持 X-Frame-Options-Header 的遗留浏览器上）。

在文档的头部添加下面的内容：

首先在样式元素上添加一个 ID：

```
<style id="antiClickjack">body{display:none !important;}</style> 
```

然后在后面的脚本中根据这个 ID 删除这个样式：

```
<script type="text/javascript">
   if (self === top) {
       var antiClickjack = document.getElementById("antiClickjack");
       antiClickjack.parentNode.removeChild(antiClickjack);
   } else {
       top.location = self.location;
   }
</script> 
```

这样，一切都可以放在文档的头部，而你的 API 只需要一个 method/taglib。

引用：[`www.codemagi.com/blog/post/194`](https://www.codemagi.com/blog/post/194)

## window.confirm() 保护

使用 x-frame-options 或 frame-brreaking 脚本是更万无一失的保护方法。然而，在内容比如允许嵌入的情况下，可以使用 window.confirm()，在用户执行的操作时，通知用户，从而减轻点击劫持的危害。

调用 window.confirm()会显示一个不能被其他页面嵌入的提示框。如果 window.confirm()的发起页面与父页面不同，提示框将显示在 window.confirm()的源页面。在这种情况下，浏览器显示提示框的发起地址，从而避免用户遭受点击劫持攻击。应该指出的是，IE 浏览器是唯一不显示提示框发起地址的浏览器，为了解决这个问题，要确保 IE 消息对话框内的信息包含操作类型的上下文信息。例如：

```
<script type="text/javascript">
   var action_confirm = window.confirm("Are you sure you want to delete your youtube account?")
   if (action_confirm) {
       //... perform action
   } else {
       //...  The user does not want to perform the requested action.
   }
</script> 
```

## 脚本失效

请看下面的代码片段，用它来防御点击劫持是**不推荐的**：

```
<script>if (top!=self) top.location.href=self.location.href</script> 
```

这个简单的框架打断脚本试图阻止页面被框架包含，或者强制父窗口加载当前页面的 URL 地址。不幸的是，许多反制这种类型的防御方法的脚本已经公开。我们在这里介绍一下。

## 双层嵌入

一些框架防御技术通过在 parent.location 上分配值，使页面跳转到正确站点。如果受攻击页面只是被一个页面嵌套，这种方法是可行的。然而，如果攻击者将受攻击页面嵌入到一个被另一个页面嵌入的页面（一个双层嵌入页面），访问 parent.location 就会违反现有流行的浏览器的安全规则（子框架导航规则）。这个安全规则会使这种跳转失效：

**受攻击页面的防御代码：**

```
if(top.location!=self.locaton) {
  parent.location = self.location;
} 
```

**攻击者顶层嵌套页面代码:**

```
<iframe src="attacker2.html"> 
```

**攻击者子嵌套页面代码:**

```
<iframe src="http://www.victim.com"> 
```

## onBeforeUnload 事件

用户可以手动取消任何框架页面的跳转请求。嵌套页面利用这一点，注册一个当框架页面即将跳转时执行的 onBeforeUnload 事件。这个事件函数会给用户弹出一个提示框。比如攻击者想嵌入 PayPal 的页面，他注册了一个事件，提示“你想退出 PayPal 吗”。当这个提示框显示给用户时，用户如果点击取消，PayPal 的页面防御就会失效。

攻击者使用下面的代码来在父层页面注册一个 unload 事件：

```
<script>
window.onbeforeunload = function()
{
  return "Asking the user nicely";
}
</script>
<iframe src="http://www.paypal.com"> 
```

PayPal 的框架保护代码将会生成一个 BeforeUnload 事件，从而激活我们的函数，提示用户取消这次跳转事件。

## 无内容冲洗

上一个攻击需要用户“参与”，同样的攻击也可以在不提示用户的情况下发生。大多数浏览器（IE7, IE8, Google Chrome, and Firefox）让攻击者在 onBeforeUnload 事件中通过重复提交跳转请求到一个返回空内容的页面(HTTP/1.0 204 No Content)。请求一个无内容页面是一个 NOP，会刷新请求管道，从而取消原来的跳转。下面是示例代码(**经测试最新的 chrome、firefox、safari 已经不支持这个特性**)：

```
var preventbust = 0
window.onbeforeunload = function() { killbust++ }
setInterval( function() {
  if(killbust > 0){
  killbust = 2;
  window.top.location = 'http://nocontent204.com'
  }
}, 1);
<iframe src="http://www.victim.com"> 
```

## 利用 XSS 过滤器

IE8 和 Google Chrome 引入反射性 XSS 过滤器，这有助于保护 web 页面免遭某些类型的 XSS 攻击。Nava 和 Lindsay（供职于 Blackhat 公司）观察到这些过滤器能被用来绕过框架保护代码。IE8 的 XSS 过滤器使用一组正则表达式来比较请求的参数，以此来找到特定的 XSS 攻击。使用“假性诱导技术”，过滤器能被用来禁止某些的脚本。通过匹配请求参数中的所有脚本标签的开头，XSS 过滤器会禁用页面内所有内联脚本，包括框架保护脚本。对于外部脚本也可以使用同样的手法来禁用所有外部脚本。因为加载的 javascript 还可以调用、cookie 也可用，这种攻击对点击劫持十分有效。

**被攻击页面的防护代码**

```
<script>
if(top != self) {
  top.location = self.location;
}
</script> 
```

**攻击页面代码:**

```
<iframe src="http://www.victim.com/?v=<script>if''> 
```

XSS 过滤器将匹配受攻击页面的参数`<script>if`，从而禁用受害者页面所有的内联脚本，包括框架保护代码。

## Clobbering top.location

一些现代浏览器将 location 变量当成一个特殊的不可变上下文属性。然而，在 IE7 和 Safari4.0.4 中，location 变量能被重新定义。在 IE7 中，一旦框架页面重新定义 location 变量，所有的子页面的尝试读取 top.location 的框架保护代码都会触发一个安全规则（在另一个领域读取局部变量），同样，任何通过 top.location 改变父页面的地址的操作都会失败。

**被攻击页面的防护代码:**

```
if(top.location != self.location) {
  top.location = self.location;
} 
```

**攻击页面代码:**

```
<script> var location = "clobbered";
</script>
<iframe src="http://www.victim.com">
</iframe> 
```

**Safari 4.0.4**

我们观察到：虽然 location 在多数情况下是不变的，但当使用 defineSetter 设置一个自定义的 location 设置时，location 会变为未定义变量。代码如下：

```
<script>
  window.defineSetter("location" , function(){});
</script> 
```

这样一来，任何尝试读取或改变父页面的位置的操作都会失败。

## 限制区域

大多数框架防护依赖被嵌套页面内的 Javascript，由它进行检测和保护。如果 javascript 在子框架内被禁用，框架保护代码将不会执行。不幸的是，有一些方法可以在子框架内限制 javascript 的运行：

**在 IE 8:**

```
<iframe src="http://www.victim.com" security="restricted"></iframe> 
```

**在 Chrome:**

```
<iframe src="http://www.victim.com" sandbox></iframe> 
```

**在 Firefox 和 IE：** 在父页面激活编辑模式。