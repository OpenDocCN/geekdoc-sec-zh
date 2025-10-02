# 附录

## 嵌入式 HTML 标签

```
<IMG SRC=”http://server/path/”> 
```

浏览器发送的 HTTP GET 请求类似如下：

```
GET http://server/path/ HTTP/1.1
Host: host
User-Agent: Firefox/1.5.0.1
Content-length: 0 
```

表单也可用于：

```
<FORM ACTION=”http://server/path/” NAME=”myform” METHOD=”POST”>
<INPUT TYPE=”HIDDEN” NAME=”Username” VALUE=”Foo”>
<INPUT TYPE=”HIDDEN” NAME=”Password” VALUE=”Bar”>
</FORM> 
```

然后使用 JavaScript，我们可以自动提交表单：

```
<SCRIPT language=”JavaScript”> document.myform.submit();
</SCRIPT> 
```

浏览器发送的 HTTP POST 请求类似如下：

```
POST http://server/path/ HTTP/1.1
Host: server
User-Agent: Firefox/1.5.0.1
Content-length: 25

Username=Foo&Password=Bar 
```

一个 JavaScript 发起的表单提交可能导致的网络浏览器发出一个警告对话框，用户可能会进行点击。其他的 html

标签包括 APPLET，BASE，BODY，EMBED，LAYER，META，OBJECT，LINK，SCRIPT 和 STYLE 也可以达到同样的效果。

## JavaScript DOM 对象

```
var img = new Image();
img.src = “http://server/path/”; 
```

浏览器发送一个 HTTP GET 请求类似如下：

```
GET http://server/path/
HTTP/1.1
Host: server
User-Agent: Firefox/1.5.0.1
Content-length: 0 
```

使用 JavaScript DOM 对象创建一个 HTML 表单：

```
var form = document.createElement(‘form’);
form.setAttribute(“action”, “http://server/path/”);
form.setAttribute(“method”, “POST”);
form.setAttribute(“name”, “myform”);
var input 1 = document.createElement(‘input’);
input1.setAttribute(“type”, “hidden”);
input1.setAttribute(“name”, “Username”);
input1.setAttribute(“value”, “Foo”);
var input 2 = document.createElement(‘input’);
input2.setAttribute(“type”, “hidden”);
input2.setAttribute(“name”, “Password”);
input2.setAttribute(“value”, “Bar”);
document.body.appendChild(form);
form.appendChild(input1);
form.appendChild(input2);
form.myform.submit(); 
```

JavaScript 会自动提交表单，导致 Web 浏览器发送一个 HTTP POST 请求类似如下：

```
POST http://server/path/ HTTP/1.1
Host: server
User-Agent: Firefox/1.5.0.1
Content-length: 25

Username=Foo&Password=Bar 
```

## XmlHttpRequest (XHR)

```
var req = new XMLHttpRequest();
req.open(‘GET’, ‘http://server/path)/’, true);
req.onreadystatechange = function () {
    if (req.readyState == 4) {
        alert(req.responseText);
    }
};
req.send(null); 
```

浏览器发送的 HTTP GET 请求类似如下：

```
GET http://server/path/ HTTP/1.1
Host: server
User-Agent: Firefox/1.5.0.1
Content-length: 0 
```

使用 XHR 发送一个 POST 请求：

```
var post_data = “Username=Foo&Password=Bar”;
var req = new XMLHttpRequest();
req.open(POST, ‘ http://host/path/’, true);
req.onreadystatechange = function () {
    if (req.readyState == 4) {
        alert(req.responseText);
    }
};
req.send(post_data); 
```

发送的 POST 请求类似如下：

```
POST http://server/path/ HTTP/1.1
Host: server
User-Agent: Firefox/1.5.0.1
Content-length: 25

Username=Foo&Password=Bar 
```