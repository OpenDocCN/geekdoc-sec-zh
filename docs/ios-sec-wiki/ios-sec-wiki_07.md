# 七、iOS 越狱程序编写原理

# iOS 越狱程序编写原理

# 7.1 Mobile Substrate 简介

### Cydia Substrate

Cydia Substrate(以前叫做 MobileSubstrate)是一个框架，允许第三方的开发者在系统的方法里打一些运行时补丁，扩展一些方法，类似 OS X 上的 Application Enhancer。

saurik 为 Substrate 写了非常全的[介绍文档](http://www.cydiasubstrate.com/id/264d6581-a762-4343-9605-729ef12ff0af/)。

Cydia Substrate 有 3 部分组成：

> *   MobileHooker
> *   MobileLoader
> *   safe mode

### MobileHooker

MobileHooker 用来替换系统函数，这个过程也叫 Hooking。有如下的 API 可以使用：

```
IMP MSHookMessage(Class class, SEL selector, IMP replacement, const char* prefix); // prefix should be NULL.

void MSHookMessageEx(Class class, SEL selector, IMP replacement, IMP *result);

void MSHookFunction(void* function, void* replacement, void** p_original); 
```

MSHookMessageEx 用来替换 Objective-C 的函数，MSHookFunction 用来替换 C/C++函数。具体用法参见[这里](http://iphonedevwiki.net/index.php/MobileSubstrate) 和 [这里](http://www.cydiasubstrate.com/api/c/MSHookFunction/)

### MobileLoader

MobileLoader 把第 3 方补丁程序加载进入运行的程序中。 MobileLoader 首先会通过 DYLD_INSERT_LIBRARIES 把自己加载进入目标程序，然后它会在/Library/MobileSubstrate/DynamicLibraries/中找到需要加载的动态链接库并加载它们，控制是否加载到目标程序，是通过一个 plist 文件来控制的。如果需要被加载的动态库的名称叫做 foo.dylib，那么这个 plist 文件就叫做 foo.plist，这个里面有一个字段叫做 filter，里面写明需要 hook 进的目标程序的 bundle id。

比如，如果只想要 foo.dylib 加载进入 SpringBoard，那么对应的 plist 文件中的 filter 就应该这样写：

```
Filter = {
  Bundles = (com.apple.springboard);
}; 
```

关于使用 DYLD_INSERT_LIBRARIES 来注入代码的例子可以参见[这里](http://blog.timac.org/?p=761)

### Safe mode

当编写的扩展导致 SpringBoard crash 的时候，MobileLoader 会捕获这个异常，然后让设备进入安全模式。在安全模式中，所有的第 3 方扩展都会被禁用。

下面这些 signal 会触发安全模式： SIGTRAP SIGABRT SIGILL SIGBUS SIGSEGV SIGSYS

### 小结

本文简要介绍了 Cydia Substarte。后面要介绍的越狱程序 Tweak，就是利用 Cydia Substrate 中的 MobileLoader 来加载的。

* * *

[# iOS 越狱程序编写下的更多文章](http://security.ios-wiki.com/issue-7/)

# 7.2 Tweak 编写简介

开发越狱程序和日常开发的 iOS 程序很相似，不过，越狱程序能做更强大的事情。你的设备越狱之后，你就能够 hook 进 Apple 提供的几乎所有的 class，来控制 iPhone/iPad 的功能。

在[3.6 Theos：越狱程序开发框架](http://security.ios-wiki.com/issue-3-6/) 这一节，我们详细介绍了如何安装 Theos 以及各种工具，头文件下载地址，以及编译出错的各种情况的解决方法。

也介绍了如何创建 Tweak 和把 Tweak 程序部署到 iOS 设备上。

之前我在 blog 上也写过几篇文章：

[iOS 越狱程序开发（1）－ 工具篇](http://wufawei.com/2013/08/iOS-jailbroken-programming-1/)
[iOS 越狱程序开发（2）－ 构建和部署](http://wufawei.com/2013/08/iOS-jailbroken-programming-2/)
[iOS 越狱程序开发（3）－ Your First Tweak](http://wufawei.com/2013/08/iOS-jailbroken-programming-3/)
[iOS 越狱程序开发（4）－ 总结](http://wufawei.com/2013/08/iOS-jailbroken-programming-4/)

[@拓词 Joey](http://weibo.com/2js3) 在其 blog 上也分享了一篇文章[使用 Theos 做一个简单的 Mobile Substrate Tweak](http://joeyio.com/ios/2014/01/01/make-a-mobile-substrate-tweak-using-theos/)，介绍了如何在锁屏界面增加一个 UILabel 显示一行文字，欢迎前往阅读。

### 小结

[3.6 Theos：越狱程序开发框架](http://security.ios-wiki.com/issue-3-6/) 这里介绍得非常详细，从第一步开始，一步步教你编写 Tweak，建议边阅读边实践。

如果遇到任何问题，请给我微博或者微信公众账号：**iOS 技术分享** 留言。

* * *

[#7 iOS 越狱程序编写下的更多文章](http://security.ios-wiki.com/issue-7/)

# 7.3 Tweak 程序编写一般步骤

#### 确定目标

第一步是确定目标，即你要分析的 App，你需要在这个 App 上编写 Tweak 完成的功能，比如挂钩 SpringBoard 使得桌面启动的时候弹框，或者拦截某个具体的应用的特定 API 调用，获得关键信息。

#### 导出头文件

确定目标之后，就可以利用 Clutch 先破解 App，然后利用 class-dump-z 导出头文件，找到你感兴趣的类，对它进行分析。

#### 获得类的方法

有时候，头文件没有所有方法调用的信息，这个时候你可以利用 cycript，使用之前介绍的[trick](http://security.ios-wiki.com/issue-4-5/)，比如 printMethod 打印出所有的方法。

#### 编写 Tweak

这一步你应该拿到需要 Hook 的类以及对应的方法，利用[Tweak 编写简介]（[`security.ios-wiki.com/issue-7-2/）中介绍的技术编写 Tweak。`](http://security.ios-wiki.com/issue-7-2/）中介绍的技术编写 Tweak。)

#### 安装与测试

把上一步的 Tweak 安装到你的设备上，验证你的 Tweak 是否工作正常。

#### 小结

这里简要介绍了 Tweak 编写的一般步骤，后面的章节会有更具体的例子来介绍这个步骤。

* * *

[#7 iOS 越狱程序编写下的更多文章](http://security.ios-wiki.com/issue-7/)

# 7.4 小结

本章简要介绍了 Cyida Substrate，Tweak 程序编写的一般步骤，下一章，我们将应用之前学到的内容进行实战。

* * *

[#7 iOS 越狱程序编写下的更多文章](http://security.ios-wiki.com/issue-7/)