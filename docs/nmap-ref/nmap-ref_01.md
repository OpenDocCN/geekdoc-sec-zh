# 一、 描述

## 一、 描述

Nmap (“Network Mapper(网络映射器)”) 是一款开放源代码的 网络探测和安全审核的工具。 它的设计目标是快速地扫描大型网络，当然用它扫描单个 主机也没有问题。Nmap 以新颖的方式 使用原始 IP 报文来发现网络上有哪些主机，那些 主机提供什么服务(应用程序名和版本)，那 些服务运行在什么操作系统(包括版本信息)， 它们使用什么类型的报文过滤器/防火墙，以及一堆其它功能。虽然 Nmap 通常用于安全审核， 许多系统管理员和网络管理员也用它来做一些 日常的工作，比如查看整个网络的信息， 管理服务升级计划，以及监视主机和服务的运行。

Nmap 输出的是扫描目标的列表，以及每个目标的补充信息，至于是哪些信息则依赖于所使用的 选项。 “所感兴趣的端口表格”是其中的关键。那张表列出端口号，协议，服务名称和状态。 状态可能是 open(开放的)，filtered(被过滤的)， closed(关闭的)，或者 unfiltered(未被过 滤的)。 Open(开放的)意味着目标机器上的应用程序正在该端口监听连接/报文。 filtered(被 过滤的) 意味着防火墙，过滤器或者其它网络障碍阻止了该端口被访问，Nmap 无法得知 它是 open(开放的) 还是 closed(关闭的)。 closed(关闭的) 端口没有应用程序在它上面监听，但 是他们随时可能开放。 当端口对 Nmap 的探测做出响应，但是 Nmap 无法确定它们是关闭还是开 放时，这些端口就被认为是 unfiltered(未被过滤的) 如果 Nmap 报告状态组合 open|filtered 和 closed|filtered 时，那说明 Nmap 无法确定该端口处于两个状态中的哪一个状态。 当要求 进行版本探测时，端口表也可以包含软件的版本信息。当要求进行 IP 协议扫描时 (-sO)，Nmap 提供关于所支持的 IP 协议而不是正在监听的端口的信息。

除了所感兴趣的端口表，Nmap 还能提供关于目标机的进一步信息，包括反向域名，操作系统猜 测，设备类型，和 MAC 地址。

一个典型的 Nmap 扫描如 Example 1, “一个典型的 Nmap 扫描”所示。在这个例子中，唯一的选 项是-A， 用来进行操作系统及其版本的探测，-T4 可以加快执行速度，接着是两个目标主机名

### Example 1\. 一个典型的 Nmap 扫描

```
# nmap -A -T4 scanme.nmap.org playground

[Starting nmap ( http://www.insecure.org/nmap/](http://www.insecure.org/nmap/) )
Interesting ports on scanme.nmap.org (205.217.153.62):
(The 1663 ports scanned but not shown below are in state: filtered)
port STATE SERVICE VERSION
22/tcp open ssh OpenSSH 3.9p1 (protocol 1.99)
53/tcp open domain
70/tcp closed gopher
80/tcp open http Apache httpd 2.0.52 ((Fedora))
113/tcp closed auth
Device type: general purpose
Running: Linux 2.4.X|2.5.X|2.6.X
OS details: Linux 2.4.7 - 2.6.11，Linux 2.6.0 - 2.6.11
Uptime 33。908 days (since Thu Jul 21 03:38:03 2005)

Interesting ports on playground。nmap。或者 g (192.168.0.40):
(The 1659 ports scanned but not shown below are in state: closed)
port STATE SERVICE VERSION
135/tcp open msrpc Microsoft Windows RPC
139/tcp open netbios-ssn
389/tcp open ldap?
![image](img/Image_003.png)
445/tcp open microsoft-ds Microsoft Windows XP microsoft-ds
1002/tcp open windows-icfw?
1025/tcp open msrpc Microsoft Windows RPC
1720/tcp open H.323/Q.931 CompTek AquaGateKeeper
5800/tcp open vnc-http RealVNC 4.0 (Resolution 400x250; VNC TCP port: 5900)
5900/tcp open vnc VNC (protocol 3.8)
MAC Address: 00:A0:CC:63:85:4B (Lite-on Communications)
Device type: general purpose
Running: Microsoft Windows NT/2K/XP
OS details: Microsoft Windows XP Pro RC1+ through final release
Service Info: OSs: Windows，Windows XP

Nmap finished: 2 IP addresses (2 hosts up) scanned in 88.392 seconds 
```