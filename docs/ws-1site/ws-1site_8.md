# （八）：应用 Wireshark 过滤条件抓取特定数据流

应用抓包过滤，选择 Capture | Options，扩展窗口查看到**Capture Filter**栏。双击选定的接口，如下图所示，弹出**Edit Interface Settints**窗口。

下图显示了**Edit Interface Settings**窗口，这里可以设置抓包过滤条件。如果你确知抓包过滤条件的语法，直接在 Capture Filter 区域输入。在输入错误时，Wireshark 通过红色背景区域表明无法处理过滤条件。最有可能的情况是，过滤条件中含有输入错误，或是使用了 display filter 的语法。

点击**Capture Filter**按钮查看并选择已保存的抓包过滤条件。

# 更多信息

**抓取指定****IP 地址的数据流:**

如果你的抓包环境下有很多主机正在通讯，可以考虑使用所观察主机的 IP 地址来进行过滤。以下为 IP 地址抓包过滤示例：

*   host 10.3.1.1：抓取发到/来自 10.3.1.1 的数据流
*   host 2406:da00:ff00::6b16:f02d：抓取发到/来自 IPv6 地址 2406:da00:ff00::6b16:f02d 的数据流
*   not host 10.3.1.1：抓取除了发到/来自 10.3.1.1 以外的所有数据流
*   src host 10.3.1.1：抓取来自 10.3.1.1 的数据流
*   dst host 10.3.1.1：抓取发到 10.3.1.1 的数据流
*   host 10.3.1.1 or 10.3.1.2：抓取发到/来自 10.3.1.1，以及与之通讯的所有数据流，与 10.3.1.2，以及与之通讯的所有数据流
*   host [www.espn.com](http://www.espn.com/)：抓取发到/来自所有解析为[www.espn.com 的 IP](http://www.espn.xn--comip-k81m/)地址的数据流

**抓取指定****IP 地址范围的数据流:**

当你需要抓取来自/发到一组地址的数据流，可以采用 CIDR(无类别域间路由，Classless Interdomain Routing)格式或使用 mask 参数。

*   net 10.3.0.0/16：抓取网络 10.3.0.0 上发到/来自所有主机的数据流(16 表示长度)
*   net 10.3.0.0 mask 255.255.0.0：与之前的过滤结果相同
*   ip6 net 2406:da00:ff00::/64：抓取网络 2406:da00:ff00:0000(IPv6)上发到/来自所有主机的数据流
*   not dst net 10.3.0.0/16：抓取除了发到以 10.3 开头的 IP 地址以外的所有数据流
*   not src net 10.3.0.0/16：抓取除了来自以 10.3 开头的 IP 地址以外的所有数据流
*   ip proto ：抓取 ip 协议字段等于值的报文。如 TCP(code 6), UDP(code 17), ICMP(code 1)。
*   ip[2:2]==：ip 报文大小
*   ip[8]==：TTL(Time to Live)值
*   ip[9]==：协议值
*   icmp[icmptype]==: 抓取 ICMP 代码等于 identifier 的 ICMP 报文, 如 icmp-echo 以及 icmp-request。

方括号中第一个数字表示从协议头开始的偏移量，第二个数字表示需要观察多少位。

**抓取发到广播或多播地址的数据流****:**

只需侦听广播或多播数据流，就可以掌握网络上主机的许多信息。

*   ip broadcast：抓取广播报文
*   ip multicast：抓取多播报文
*   dst host ff02::1：抓取到 IPv6 多播地址所有主机的数据流
*   dst host ff02::2：抓取到 IPv6 多播地址所有路由器的数据流

小贴士：

Wireshark 包含了一些默认的抓包过滤条件。点击主工具栏的**Edit Capture Filters**，跳转到已保存抓包过滤列表。你会发现一些常见抓包过滤的示例。

**抓取基于****MAC 地址的数据流:**

当你需要抓取发到/来自某一主机的 IPv4 或 IPv6 数据流，可创建基于主机 MAC 地址的抓包过滤条件。

应用 MAC 地址时，需确保与目标主机处于同一网段。

*   ether host 00:08:15:00:08:15：抓取发到/来自 00:08:15:00:08:15 的数据流
*   ether src 02:0A:42:23:41:AC：抓取来自 02:0A:42:23:41:AC 的数据流
*   ether dst 02:0A:42:23:41:AC：抓取发到 02:0A:42:23:41:AC 的数据流
*   not ether host 00:08:15:00:08:15：抓取除了发到/来自 00:08:15:00:08:15 以外的所有数据流
*   ether broadcast 或 ether dst ff:ff:ff:ff:ff:ff：抓取广播报文
*   ether multicast：多播报文
*   抓取指定以太网类型的报文：ether proto 0800
*   抓取指定 VLAN：vlan
*   抓取指定几个 VLAN：vlan and vlan

**抓取基于指定应用的数据流****:**

你可能需要查看基于一个或几个应用的数据流。抓包过滤器语法无法识别应用名，因此需要根据端口号来定义应用。通过目标应用的 TCP 或 UDP 端口号，将不相关的报文过滤掉。

*   port 53：抓取发到/来自端口 53 的 UDP/TCP 数据流（典型是 DNS 数据流）
*   not port 53：抓取除了发到/来自端口 53 以外的 UDP/TCP 数据流
*   port 80：抓取发到/来自端口 80 的 UDP/TCP 数据流（典型是 HTTP 数据流）
*   udp port 67：抓取发到/来自端口 67 的 UDP 数据流（典型是 DHCP 据流）
*   tcp port 21：抓取发到/来自端口 21 的 TCP 数据流（典型是 FTP 命令通道）
*   portrange 1-80：抓取发到/来自端口 1-80 的所有 UDP/TCP 数据流
*   tcp portrange 1-80：抓取发到/来自端口 1-80 的所有 TCP 数据流

**抓取结合端口的数据流****:**

当你需要抓取多个不连续端口号的数据流，将它们通过逻辑符号连接起来，如下图所示：

*   port 20 or port 21：抓取发到/来自端口 20 或 21 的 UDP/TCP 数据流（典型是 FTP 数据和命令端口）
*   host 10.3.1.1 and port 80：抓取发到/来自 10.3.1.1 端口 80 的数据流
*   host 10.3.1.1 and not port 80：抓取发到/来自 10.3.1.1 除了端口 80 以外的数据流
*   udp src port 68 and udp dst port 67：抓取从端口 68 到端口 67 的所有 UDP 数据流（典型是从 DHCP 客户端到 DHCP 服务器）
*   udp src port 67 and udp dst port 68：抓取从端口 67 到端口 68 的所有 UDP 数据流（典型是从 DHCP 服务器到 DHCP 客户端）
*   抓取 TCP 连接的开始（SYN）和结束（FIN）报文，配置 tcp[tcpflags] & (tcp-syn|tcp-fin)!=0
*   抓取所有 RST(Reset)标志位为 1 的 TCP 报文，配置 tcp[tcpflags] & (tcp-rst)!=0
*   less ：抓取小于等于某一长度的报文，等同于 len
*   greater ：抓取大于等于某一长度的报文，等同于 len >=

SYN: 简历连接的信号

FIN: 关闭连接的信号

ACK: 确认接收数据的信号

RST: 立即关闭连接的信号

PSH: 推信号，尽快将数据转由应用处理

*   tcp[13] & 0×00 = 0: No flags set (null scan)
*   tcp[13] & 0×01 = 1: FIN set and ACK not set
*   tcp[13] & 0×03 = 3: SYN set and FIN set
*   tcp[13] & 0×05 = 5: RST set and FIN set
*   tcp[13] & 0×06 = 6: SYN set and RST set
*   tcp[13] & 0×08 = 8: PSH set and ACK not set

tcp[13]是从协议头开始的偏移量，0,1,3,5,6,8 是标识位

**尽量避免使用抓包过滤。即便多看几个报文，也比漏看一个报文要好。**当你抓取了大量报文的时候，用显示过滤（过滤选项也更多）来重点查看某一数据流。

小贴士：

如果你需要查看 TCP 帧中的某一 ASCII 字符串，用 Wireshark String-Matching Capture Filter Generator([`www.wireshark.org/tools/string-cf.html`](http://www.wireshark.org/tools/string-cf.html))。例如，想要抓取 HTTP GET 报文，输入 GET 并将 TCP 偏移量设置为 0。