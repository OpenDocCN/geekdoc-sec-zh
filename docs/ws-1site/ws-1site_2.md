# （二）：应用 Wireshark 观察基本网络协议

**TCP:**

TCP/IP 通过三次握手建立一个连接。这一过程中的三种报文是：SYN，SYN/ACK，ACK。

第一步是找到 PC 发送到网络服务器的第一个 SYN 报文，这标识了 TCP 三次握手的开始。

如果你找不到第一个 SYN 报文，选择**Edit -> Find Packet**菜单选项。选择 Display Filter，输入过滤条件：tcp.flags，这时会看到一个 flag 列表用于选择。选择合适的 flag，tcp.flags.syn 并且加上==1。点击 Find，之后 trace 中的第一个 SYN 报文就会高亮出来了。

注意：Find Packet 也可以用于搜索十六进制字符，比如恶意软件信号，或搜索字符串，比如抓包文件中的协议命令。

一个快速过滤 TCP 报文流的方式是在**Packet List Panel**中右键报文，并且选择**Follow TCP Stream**。这就创建了一个只显示 TCP 会话报文的自动过滤条件。

这一步骤会弹出一个会话显示窗口，默认情况下包含 TCP 会话的 ASCII 代码，客户端报文用红色表示服务器报文则为蓝色。

窗口类似下图所示，对于读取协议有效载荷非常有帮助，比如 HTTP，SMTP，FTP。

更改为十六进制 Dump 模式查看载荷的十六进制代码，如下图所示：

关闭弹出窗口，Wireshark 就只显示所选 TCP 报文流。现在可以轻松分辨出 3 次握手信号。

注意：这里 Wireshark 自动为此 TCP 会话创建了一个显示过滤。本例中：(ip.addr eq 192.168.1.2 and ip.addr eq 209.85.227.19) and (tcp.port eq 80 and tcp.port eq 52336)

**SYN**报文：

图中显示的 5 号报文是从客户端发送至服务器端的 SYN 报文，此报文用于与服务器建立同步，确保客户端和服务器端的通信按次序传输。SYN 报文的头部有一个 32 bit 序列号。底端对话框显示了报文一些有用信息如报文类型，序列号。

**SYN/AC**K 报文：

7 号报文是服务器的响应。一旦服务器接收到客户端的 SYN 报文，就读取报文的序列号并且使用此编号作为响应，也就是说它告知客户机，服务器接收到了 SYN 报文，通过对原 SYN 报文序列号加一并且作为响应编号来实现，之后客户端就知道服务器能够接收通信。

**ACK**报文：

8 号报文是客户端对服务器发送的确认报文，告诉服务器客户端接收到了 SYN/ACK 报文，并且与前一步一样客户端也将序列号加一，此包发送完毕，客户端和服务器进入[ESTABLISHED](http://baike.baidu.com/view/1137549.htm)状态，完成三次握手。

**ARP & ICMP：**

开启 Wireshark 抓包。打开 Windows 控制台窗口，使用 ping 命令行工具查看与相邻机器的连接状况。

停止抓包之后，Wireshark 如下图所示。

ARP 和 ICMP 报文相对较难辨认，创建只显示 ARP 或 ICMP 的过滤条件。

**ARP**报文：

地址解析协议，即 ARP（Address Resolution Protocol），是根据[IP 地址](http://baike.baidu.com/view/3930.htm)获取[物理地址](http://baike.baidu.com/view/883168.htm)的一个[TCP/IP 协议](http://baike.baidu.com/view/7649.htm)。其功能是：[主机](http://baike.baidu.com/view/23880.htm)将 ARP 请求[广播](http://baike.baidu.com/view/35385.htm)到网络上的所有主机，并接收返回消息，确定目标[IP 地址](http://baike.baidu.com/view/3930.htm)的物理地址，同时将 IP 地址和硬件地址存入本机 ARP 缓存中，下次请求时直接查询 ARP 缓存。

最初从 PC 发出的 ARP 请求确定 IP 地址 192.168.1.1 的 MAC 地址，并从相邻系统收到 ARP 回复。ARP 请求之后，会看到 ICMP 报文。

**ICMP**报文：

网络控制消息协定（Internet Control Message Protocol，ICMP）用于[TCP/IP](http://zh.wikipedia.org/wiki/TCP/IP)网络中发送控制消息，提供可能发生在通信环境中的各种问题反馈，通过这些信息，令管理者可以对所发生的问题作出诊断，然后采取适当的措施解决。

PC 发送 echo 请求，收到 echo 回复如上图所示。ping 报文被 mark 成 Type 8，回复报文 mark 成 Type 0。

如果多次 ping 同一系统，在 PC 上删除 ARP cache，使用如下 ARP 命令之后，会产生一个新的 ARP 请求。

C:> ping 192.168.1.1

… ping output …

C:> arp –d *

**HTTP：**

HTTP 协议是目前使用最广泛的一种基础协议，这得益于目前很多应用都基于 WEB 方式，实现容易，软件开发部署也简单，无需额外的客户端，使用浏览器即可使用。这一过程开始于请求服务器传送网络文件。

从上图可见报文中包括一个 GET 命令，当 HTTP 发送初始 GET 命令之后，TCP 继续数据传输过程，接下来的链接过程中 HTTP 会从服务器请求数据并使用 TCP 将数据传回客户端。传送数据之前，服务器通过发送 HTTP OK 消息告知客户端请求有效。如果服务器没有将目标发送给客户端的许可，将会返回 403 Forbidden。如果服务器找不到客户端所请求的目标，会返回 404。

如果没有更多数据，连接可被终止，类似于 TCP 三次握手信号的 SYN 和 ACK 报文，这里发送的是 FIN 和 ACK 报文。当服务器结束传送数据，就发送 FIN/ACK 给客户端，此报文表示结束连接。接下来客户端返回 ACK 报文并且对 FIN/ACK 中的序列号加 1。这就从服务器端终止了通信。要结束这一过程客户端必须重新对服务器端发起这一过程。必须在客户端和服务器端都发起并确认 FIN/ACK 过程。