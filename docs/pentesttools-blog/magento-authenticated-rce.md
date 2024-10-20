# 通过反序列化的 PHAR 文件认证的 Pentest-Tools.com RCE

> 原文：<https://pentest-tools.com/blog/magento-authenticated-rce>

回到 2019 年 8 月，我使用 [HackerOne bug bounty 平台](https://hackerone.com/magento)报告了 Magento 中影响 2.3.2、2.3.3 和 2.3.4 版本的安全漏洞。

该错误影响了 Magento 的一些安装，并允许我们基于 PHAR 文件的反序列化方式和滥用 Magento 的协议指令来获得远程代码执行。

Adobe 于 2020 年 4 月 28 日修补了该漏洞。在撰写本文时，漏洞报告尚未公开。

鉴于此漏洞可被任何权限级别的用户利用，它对未打补丁的部署有很高的业务风险影响。

## 1.弱点分析

来深究一下细节吧！下面我们将回顾受影响的 Magento 版本和实例，导致漏洞的原因，以及如何利用它来实现远程代码执行。

### TL；速度三角形定位法(dead reckoning)

为了更好地执行后续步骤，这里总结了攻击顺序:

1.并非所有 Magento 2.3.2、2.3.3 和 2.3.4 实例都会受到影响。从 2.3.1 开始，默认情况下禁用 PHAR 支持。在默认的 Magento 安装中，你可以在 **app/bootstrap.php** 下检查 PHAR 支持是否被禁用。如果你找到下面的代码片段，你的 Magento 安装是**而不是**受影响的:

```
if (in_array('phar', \stream_get_wrappers())) {
    stream_wrapper_unregister('phar');
}
```

2.该漏洞存在于管理界面中，您可以通过向页面添加新图像来触发它。PHAR 档案可以嵌入到 JPEGs 文件中——继续阅读了解如何嵌入。该图像将被传递给一个 **getimagesize()** ，如果文件名以**“phar://”**开头，它将触发[反序列化](https://pentest-tools.com/blog/exploit-phar-deserialization-vulnerability)

3.在这种情况下，它不会，我们需要找到一种方法来绕过这一点。在前端，图像 src 被设置为{{media url= **filename** }}。这将决定 Magento 的**媒体指令**要处理的图像。我们发现的错误允许我们改变 src 来访问*另一个*指令。所以我们使用了**协议指令**，它允许我们完全控制文件名。

4.剩下的唯一事情就是找到可以在 POP 链中使用的类来实现 RCE。我们使用了**guzzle http/Stream/FnStream**和 **phpseclib\Crypt\Hash** 。

### 背景:在 JPEGs 中嵌入 PHAR

这种攻击需要能够在服务器上放置 PHAR 档案，因为只有当包装器引用的资源是本地资源时，才会考虑包装器。

您可能知道，文件上传受到各种限制。当试图利用这种漏洞时，能够创建既作为 PHAR 存档有效又作为另一种文件类型(如 JPEG)有效的文件是很有用的。

您可以用三种文件格式存储 PHARs:phar、tar 和 zip。对于这个例子，让我们研究一下 TAR 格式，因为它提供了一些您可以使用的灵活性。

让我们使用 phar、tar 和 jpeg 的以下属性来创建有效的 TAR/JPEG 多语种文件:

*   **jpeg** :可以在元数据中插入任意长度的注释

*   **tar** :档案的前 100 个字节包含档案的第一个文件名；存档的结尾由 1024 个连续的 0 字节标记

*   **phar** :任何数量的数据都可以放在存根的开头。

然后，您可以在 JPEG 文件中嵌入 PHAR 档案:

1.  以 JPEG 开始标记 **0xFFD8** 开始归档，后面是注释开始 **0xFFFE** 和注释长度。

2.  然后跟随实际的注释，这将是我们的 phar 存档，加上 1024 个零来标记存档结束。因此，注释的长度等于存档内容的长度+ 1024。

3.  之后是图像数据。

一个假设的 JPG/PHAR 多语言者将具有以下结构:

**0x ffd 8 |****0x fffe**|**评论 _ 长度|**|**PHAR _ 数据** | **存档 _ 结束** | **图像 _ 数据**

### 实用分析

该漏洞存在于 Magento 的 WYSIWYG 编辑器中负责渲染图像的组件中。

编辑器有两种状态，**显示**和**隐藏**。每当您从**隐藏**切换到**显示**时，页面上的每个图像都会向一个端点发出 GET 请求。看起来是这样的:

magento.url/admin/cms/wysiwyg/directive/___directive/**base64(图片。src)** /...

在后端，这到达**供应商/magento/模块-CMS/控制器/**

**Adminhtml/WYSIWYG/directive。PHP**

```
public function execute()

{

      $directive = $this->getRequest()->getParam('___directive');

      $directive = $this->urlDecoder->decode($directive);

[...snip...]

try   {

                      $filter = $this->_objectManager->create(Filter::class);

                      $imagePath = $filter->filter($directive);

                      $image = …

                       [...snip...]

                       $image->open($imagePath);

                       [...snip...]

      }

}
```

前面提到的 base64 编码图像 src 被看作是 **$directive** 变量的值。

默认情况下，该 src 的格式为**{ { media URL = image-name } }**。在这里，**媒体**就是我们所说的**指令**。

这很重要，因为**Directive.php**中的下一步是对**_ _ 指令**:**$ image path = $ filter->filter($ directive)应用一些过滤；**

该滤波的结果将是**一个图像路径**，其值取决于 src 中使用的指令。我们一会儿将回到这一点。

获得图像路径后，创建一个新的图像对象。同时，图像被打开。处理打开的函数调用 **getimagesize(imagePath)** ，该函数将 [**反序列化 PHAR 档案**](https://pentest-tools.com/blog/exploit-phar-deserialization-vulnerability) 。

让我们通过一个例子来看看这是如何发生的:

1.  上传一个名为 phar.jpg 的新图片到我们的 Magento 网页。

2.  添加图像为:`<img src="{{media url=”phar.jpg”}}" alt="" >`

3.  按按钮显示所见即所得编辑器。在后端，**Directive.php**会收到 **{{media url="phar.jpg"}}** 。

4.  这将被传递给过滤函数，产生**图像路径**。

5.  使用在调用**$ image->open($ image path)**中得到的图像路径；在这种情况下， **$imagePath** 将是 **pub/media/phar.jpg** ，其中 **pub/media** 是 Magento 中的默认媒体位置。

为了反序列化 PHAR，您需要控制传递给 **getimagesize()** 的名称的开头。正如我们关于 PHAR 反序列化的技术指南中所述，只有当文件名看起来像“ **phar://…** ”时，反序列化才有效。

所以你需要找到一种方法来完全控制产生的**图像路径**。

### 协议指令

回到**Directive.php**，我们知道图像路径是调用**$ image path = $ filter->filter($ directive)**的结果。

$filter 对象是 Magento \ \ Cms \ \ Model \ \ Template \ \ Filter 类的一个实例，它继承了 Magento \ \ Email \ \ Model \ \ Template \ \ Filter 类。保留这些细节，因为它们以后会有用。

filter 方法将尝试将**$指令**匹配到一个特定的正则表达式。基于匹配，它将对基于 **$directive** 中的指令的方法进行回调。

在默认场景中，使用**{ { media URL = " phar . jpg " } }**作为输入，将调用 **mediaDirective** 。这个方法属于**Magento \ \ Cms \ \ Model \ \ Template \ \ Filter**。如前所述，这个类继承了**Magento \ \ Email \ \ Model \ \ Template \ \ Filter**，幸运的是它包含了更多的指令方法。

这意味着您可以更改输入来访问另一个指令。对于这个深入研究，我们使用了 **protocolDirective** ，您可以使用它来控制图像的最终名称。要访问该指令，您必须提供以下格式的输入**{ {协议...}}**

让我们看一下这个方法的代码片段:

```
public function protocolDirective($construction)

{

        $params = $this->getParameters($construction[2]);

        $isSecure = 

$this->_storeManager->getStore($store)->isCurrentlySecure();

      $protocol = $isSecure ? 'https' : 'http';

      if  (isset($params['url']))   {

               return $protocol . '://' . $params['url'];

      } elseif (isset($params['http']) && isset($params['https'])) {

            if ($isSecure)    {

                return $params['https'];

            }

             return $params['http'];

      }

      return $protocol;

}
```

从第一行开始， **$params** 将是一个关联数组。如果你看下面，你可以看到它包括 3 个相关的键: **url** 、 **http** 和 **https** 。

如果未设置**$ params[' URL ']****并且设置了 **$params['http']** 和 **$params['https']** ，则该方法将根据商店是否使用 https 返回作为 **http** 或 **https** 的值传递的任何内容。**

那么您得到的有效负载是:**{ { protocol http = " phar://phar . jpg " https = " phar://phar . jpg " } }**。

用这个作为前端图片的 **src** 会导致**$ image path = " phar://phar . jpg "**触发反序列化。

### 利润

我们现在需要一个 POP 链来包含在我们的 PHAR 中，以利用该应用程序。

使用的类是**guzzle http/Stream/FnStream**和 **phpseclib\Crypt\Hash** 。

**guzzle http/Stream/FnStream**的析构函数用于启动漏洞链:

```
public function __destruct()

{

      if (isset($this->_fn_close)) {

          call_user_func($this->_fn_close);

      }

}
```

您可以使用它来回调当前作用域中任何类的任何方法。这方面的一个候选方法是再次使用 call_ **user_func()** ，但是带有多个参数。这允许您使用像 **passthru()** 或 **exec()** 这样的函数在服务器上执行命令。

**phpseclib\Crypt\Hash** 的方法 **_computeKey** 进行这样的调用:

```
function _computeKey()

{

          if ($this->key === false) {

               $this->computedKey = false;

               return;

      }

           if (strlen($this->key) <= $this->b) {

$this->computedKey = $this->key;

                return;

      }

           switch ($this->engine) {

              //modified the cases to ease understanding

              case 2:

                     $this->computedKey = mhash($this->hash, $this->key);

                     break;

             case 3:

                     $this->computedKey = hash($this->hash, $this->key, true);

                     break;

             case 1:

                     $this->computedKey = call_user_func($this->hash, $this->key);

      }

}
```

要使用该函数执行代码，您必须满足三个条件:

1.  将**$键**设置为比 **$b** 长的值。假设 **$key** 将是我们的服务器命令，您可以将$b 设置为一个小数字，比如 0

2.  将 **$engine** 设置为 1，这样开关将转到最后一种情况。

3.  将 **$hash** 设置为类似于**“passthru”**的值。

## 2.影响

通过滥用 Magento 的协议指令，任何经过验证的用户都可以上传精心制作的图像文件，该文件能够在 web 服务器用户的上下文中执行远程代码执行。

## 3.概念证明

<template x-if="showVideo"></template>

负责任披露时间表

*   **2019 年 8 月 29 日** -提交给 HackerOne 的漏洞

*   **2019 年 9 月 12 日**——黑客龙(HackerOne)排查出的 Bug

*   **2019 年 10 月 8 日** - Magento (Adobe)安全团队回应

*   **2019 年 11 月 27 日**-hacker one 颁发的赏金

*   2020 年 4 月 28 日-Adobe 发布的修复程序