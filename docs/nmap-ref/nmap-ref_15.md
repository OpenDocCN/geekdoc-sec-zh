# 十五、 运行时的交互

## 十五、 运行时的交互

**Nmap 目前还不具有这个功能，本节内容可能会增加或删除。**

在执行 Nmap 时，所有的键盘敲击都被记录。这使得用户可以与 程序交互而不需要终止或重启 特定的键可改变选项，其它键会输出 一个有关扫描的状态消息。约定如下，小写字母增加 打 印量，大写字母减少打印量。

`v / V`

增加 / 减少细节

`d / D`

提高 / 降低调试级别

`p / P`

打开/ 养老报文跟踪

其它

打印的信息类似于：

```
Stats: 0:00:08 elapsed; 111 hosts completed (5 up)， 5 undergoing Service Scan Service scan Timing: About 28.00% done; ETC: 16:18 (0:00:15 remaining) 
```