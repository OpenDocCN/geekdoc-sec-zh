# 入门

## 入门

这将是第一个一系列关于 python 编程的博客文章。python 是一门非常强大的语言，因为它有信息安全社区的支撑。这意味着很多工具都是由 python 编写并且可以在脚本中调用很多模块。使用模块的好处就是只需要少量的代码就能够完成所需的任务。

这篇文章假定你的系统是 Linux，python 版本是 2.*。在写代码的时候你也可以直接的写在解释器里面(linux 里面输入 python 即可进入)，也可以把代码放到一个文件里面。很多人会发现把代码存放到文件里面要比直接写在解释器上面要好很多。值得注意的是 python 中强制缩进。大家在写函数声明，循环，if/else 语句等等的时候就会发现。

**python 解释器**

在终端里面输入 python:

```py
~$ python
Python 2.7.6 (default, Mar 22 2014, 22:59:56) 
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

输入之后你就可以直接在解释器里面写你的代码了。下面我们将声明两个变量，并且使用 type()函数查看变量的类型。假设我们声明了一个字符串和整型：

```py
>>>
>>> ip = '8.8.8.8'
>>> port = 53
>>>
>>> type(ip)
<type 'str'>
>>>
>>> type(port)
<type 'int'>
>>> 
```

你可以使用内置的 help()函数去了解一个函数的详细。记住这一点，它可以帮助你在学习语言的时候学习到更多的详细内容.

```py
>>>
>>> help(type)
>>> 
```

有时你会想把一些变量和字符串连接起来然后通过脚本显示出来。那么你就需要使用 str()函数把整型转换成字符串类型

```py
>>> ip='1.1.1.1'
>>> port=55
>>> print 'the ip is:'+ip+'and the port is:'+str(port)
the ip is:1.1.1.1and the port is:55 
```

前面声明变量的时候"IP"就是一个字符串就不需要转换，而"port"就需要。现在你就已经知道了两个基本的数据类型(string 和 integer)。现在你可以试试使用内置函数与这两个数据类型写出其他的代码。

Python 字符串允许你通过偏移值来获取你想需要的字符串,并且可以通过 len()函数来获取字符串的长度，它可以帮助你更方便的操作字符串。

```py
>>>
>>> domain='primalsecurity.net'
>>> domain
'primalsecurity.net'
>>> domain[0]
'p'
>>> domain[0:3]
'pri'
>>> domain[1:]
'rimalsecurity.net'

>>> len(domain)
18 
```

你可以使用内建的 dir()函数来列出模块定义的标识符。标识符有函数、类和变量。

```py
>>> dir(ip)
['__add__', '__class__', '__contains__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__getslice__', '__gt__', '__hash__', '__init__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '_formatter_field_name_split', '_formatter_parser', 'capitalize', 'center', 'count', 'decode', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'index', 'isalnum', 'isalpha', 'isdigit', 'islower', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill'] 
```

现在你可以使用上面列举出来的内建字符串函数，如果想知道这个函数的更多描述可以参考前面提到的 help()函数:

```py
>>>
>>> help(ip.split)
>>>
>>> string = ip+':'+str(port)
>>> string
'8.8.8.8:53'
>>>
>>> string.split(':')
['8.8.8.8', '53'] 
```

这 split 函数把一个字符串通过":"切割生成一个新的列表。这是一个非常有用的字符串函数因为你能够把这个字符串里面的有用信息提出出来。例如，你获取到了一个 ip 列表，你想在这个列表里面添加一个索引值。你也可以删除和添加新的值到这个列表里面通过.append()和.remove()函数

```py
>>>
>>> list = string.split(':')
>>>
>>> list
['8.8.8.8', '53']
>>>
>>> list[0]
'8.8.8.8'
>>>
>>> list.append('google')
>>> list
['8.8.8.8', '53', 'google']
>>> list.remove('google')
>>> list
['8.8.8.8', '53']
>>> 
```

**Python 模块**

在上面提到过，Python 模块能够让你用少量的代码就能够完成你的任务,Python 有许多有用的内建模块(os,subprocess,socket,urllib,httplib,re,sys 等等)和第三方模块(cymruwhois,scapy,dpkt,spider 等等).使用 Python 模块很简单"import <modulenmae class="calibre11">". OS 模块是非常重要的因为你需要在你的 Python 代码里面调用系统命令:</modulenmae>

```py
>>>
>>> import os
>>>
>>> dir(os)
['EX_CANTCREAT', 'EX_CONFIG', 'EX_DATAERR', 'EX_IOERR', 'EX_NOHOST', 'EX_NOINPUT', 'EX_NOPERM', 'EX_NOUSER', 'EX_OK', 'EX_OSERR', 'EX_OSFILE', 'EX_PROTOCOL', 'EX_SOFTWARE', 'EX_TEMPFAIL', 'EX_UNAVAILABLE', 'EX_USAGE', 'F_OK', 'NGROUPS_MAX', 'O_APPEND', 'O_ASYNC', 'O_CREAT', 'O_DIRECT', 'O_DIRECTORY', 'O_DSYNC', 'O_EXCL', 'O_LARGEFILE', 'O_NDELAY', 'O_NOATIME', 'O_NOCTTY', 'O_NOFOLLOW', 'O_NONBLOCK', 'O_RDONLY', 'O_RDWR', 'O_RSYNC', 'O_SYNC', 'O_TRUNC', 'O_WRONLY', 'P_NOWAIT', 'P_NOWAITO', 'P_WAIT', 'R_OK', 'SEEK_CUR', 'SEEK_END', 'SEEK_SET', 'ST_APPEND', 'ST_MANDLOCK', 'ST_NOATIME', 'ST_NODEV', 'ST_NODIRATIME', 'ST_NOEXEC', 'ST_NOSUID', 'ST_RDONLY', 'ST_RELATIME', 'ST_SYNCHRONOUS', 'ST_WRITE', 'TMP_MAX', 'UserDict', 'WCONTINUED', 'WCOREDUMP', 'WEXITSTATUS', 'WIFCONTINUED', 'WIFEXITED', 'WIFSIGNALED', 'WIFSTOPPED', 'WNOHANG', 'WSTOPSIG', 'WTERMSIG', 'WUNTRACED', 'W_OK', 'X_OK', '_Environ', '__all__', '__builtins__', '__doc__', '__file__', '__name__', '__package__', '_copy_reg', '_execvpe', '_exists', '_exit', '_get_exports_list', '_make_stat_result', '_make_statvfs_result', '_pickle_stat_result', '_pickle_statvfs_result', '_spawnvef', 'abort', 'access', 'altsep', 'chdir', 'chmod', 'chown', 'chroot', 'close', 'closerange', 'confstr', 'confstr_names', 'ctermid', 'curdir', 'defpath', 'devnull', 'dup', 'dup2', 'environ', 'errno', 'error', 'execl', 'execle', 'execlp', 'execlpe', 'execv', 'execve', 'execvp', 'execvpe', 'extsep', 'fchdir', 'fchmod', 'fchown', 'fdatasync', 'fdopen', 'fork', 'forkpty', 'fpathconf', 'fstat', 'fstatvfs', 'fsync', 'ftruncate', 'getcwd', 'getcwdu', 'getegid', 'getenv', 'geteuid', 'getgid', 'getgroups', 'getloadavg', 'getlogin', 'getpgid', 'getpgrp', 'getpid', 'getppid', 'getresgid', 'getresuid', 'getsid', 'getuid', 'initgroups', 'isatty', 'kill', 'killpg', 'lchown', 'linesep', 'link', 'listdir', 'lseek', 'lstat', 'major', 'makedev', 'makedirs', 'minor', 'mkdir', 'mkfifo', 'mknod', 'name', 'nice', 'open', 'openpty', 'pardir', 'path', 'pathconf', 'pathconf_names', 'pathsep', 'pipe', 'popen', 'popen2', 'popen3', 'popen4', 'putenv', 'read', 'readlink', 'remove', 'removedirs', 'rename', 'renames', 'rmdir', 'sep', 'setegid', 'seteuid', 'setgid', 'setgroups', 'setpgid', 'setpgrp', 'setregid', 'setresgid', 'setresuid', 'setreuid', 'setsid', 'setuid', 'spawnl', 'spawnle', 'spawnlp', 'spawnlpe', 'spawnv', 'spawnve', 'spawnvp', 'spawnvpe', 'stat', 'stat_float_times', 'stat_result', 'statvfs', 'statvfs_result', 'strerror', 'symlink', 'sys', 'sysconf', 'sysconf_names', 'system', 'tcgetpgrp', 'tcsetpgrp', 'tempnam', 'times', 'tmpfile', 'tmpnam', 'ttyname', 'umask', 'uname', 'unlink', 'unsetenv', 'urandom', 'utime', 'wait', 'wait3', 'wait4', 'waitpid', 'walk', 'write']
>>> 
```

你可以看到上面 os 模块给你提供了很多可以使用的功能函数，其中我发现我经常使用"os.system"，我可给它传递一个命令，然后通过它去在系统底层执行我们传递的命令.下面我们将会执行一个命令"echo ‘UHJpbWFsIFNlY3VyaXR5Cg==’ | base64 -d":

```py
>>>
>>> os.system("echo 'UHJpbWFsIFNlY3VyaXR5Cg==' | base64 -d")
Primal Security
>>> 
```

**创建一个文件对象**

现在我们将演示一些例子,如何在 Python 里面从一个文件里面读取数据和创建一个文件。下面的这个例子演示了如何创建一个文件对象，并且读取/写入数据到这个对象里面，通常你自己读取一个文件的数据，并且做一些逻辑处理然后把输出的写到文件里面:

```py
>>>
>>> file = open('test.txt', 'w')
>>> file.write('Hello World')
>>> file.close()    
>>> file = open('test.txt', 'r')
>>> file.readlines()
['Hello World']
>>> 
```

在 Python 解释器里面练习上面的内容并且多加巩固，因为这些内容在后面的章节里面会经常使用，当我写代码的时候，我喜欢打开两个终端，一个用于执行 python 解释器，还有一个用来把逻辑写入到脚本里面。下一章将会写一个真实的 Python 脚本， 声明定义，类和 sys 模块。