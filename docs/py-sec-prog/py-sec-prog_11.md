# 系统命令调用

## 自动化命令

这一章将会介绍使用 python 自动执行系统命令,我们将使用 python 展示两个执行命令的方式(os,subprocess).

当你开始创建一个脚本的时候,你会发现 os.system 和 subprocess.Popen 都是执行系统命令,它们不是一样的吗?其实它们两个根本不一样,subprocess 允许你执行命令直接通过 stdout 赋值给一个变量,这样你就可以在结果输出之前做一些操作,譬如:输出内容的格式化等.这些东西在你以后的会很有帮助.

Ok,说了这么多,让我们来看看代码:

```py
>>>
>>> import os
>>> os.system('uname -a')
Linux cell 3.11.0-20-generic #35~precise1-Ubuntu SMP Fri May 2 21:32:55 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
0
>>> os.system('id')
uid=1000(cell) gid=1000(cell) groups=1000(cell),0(root)
0
>>> os.system('ping -c 1 127.0.0.1')
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_req=1 ttl=64 time=0.043 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.043/0.043/0.043/0.000 ms
0
>>> 
```

上面这段代码并没有完全的演示完 os 模块所有的功能,不过你可以使用"dir(os)"命令来查看其他的函数(译者注:　如果不会使用可以使用 help()命令).

下面我们使用 subprocess 模块运行相同的命令:

```py
>>> import subprocess
>>>
>>> com_str = 'uname -a'
>>> command = subprocess.Popen([com_str], stdout=subprocess.PIPE, shell=True)
>>> (output, error) = command.communicate()
>>> print output
Linux cell 3.11.0-20-generic #35~precise1-Ubuntu SMP Fri May 2 21:32:55 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

>>> com_str = 'id'
>>> command = subprocess.Popen([com_str], stdout=subprocess.PIPE, shell=True)
>>> (output, error) = command.communicate()
>>> print output
uid=1000(cell) gid=1000(cell) groups=1000(cell),0(root)
>>> 
```

和第一段代码对比你会发现语法比较复杂,但是你可以把内容存储到一个变量里面并且你也可以把返回的内容写入到一个文件里面去;

```py
>>> com_str = 'id'
>>> command = subprocess.Popen([com_str], stdout=subprocess.PIPE, shell=True)
>>> (output, error) = command.communicate()
>>> output
'uid=1000(cell) gid=1000(cell) groups=1000(cell),0(root)\n'
>>> f = open('file.txt', 'w')
>>> f.write(output)
>>> f.close()
>>> for line in open('file.txt', 'r'):
...   print line
...
uid=1000(cell) gid=1000(cell) groups=1000(cell),0(root)

>>> 
```

这一章我们讲解了如何自动化执行系统命令,记住当你以后遇到 CLI 的时候可以把它丢到 python 脚本里面;

最后自己尝试一下写一个脚本,把输出的内容写入到一个文件里面或者是只输出部分信息.