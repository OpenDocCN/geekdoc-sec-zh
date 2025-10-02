# 三、自己动手写一个 windows 调试器

# 3 自己动手写一个 windows 调试器

现在我们已经讲解完了基础知识，是时候实现一个真正的的 调试器的时候了。当微软开发 windows 的时候，他们增加了一 大堆的令人惊喜的调试函数以帮助开发者们保证产品的质量。我 们将大量的使用这些函数创建你自己的纯 python 调试器。有一点很重要，我们本质上是在深入的学习 PyDbg(Pedram Amini’s )的使用，这是目前能找到的最简洁的 Windows 平台下的 Python 调试器 。拜 Pedram 所赐，我尽可能用 PyDbg 完成了我的代码（包括函数名，变 量，等等），同时你也可以更容易的用 PyDbg 实现你的调试器。

为了对一个进程进行调试，你首先必须用一些方法把调试器和进程连接起来。所以，我 们的调试器要不然就是装载一个可执行程序然后运行它，要不然就是动态的附加到一个运行 的进程。Windows 的调试接口（Windows debugging API）提供了一个非常简单的方法完成 这两点。

运行一个程序和附加到一个程序有细微的差别。打开一个程序的优点在于他能在程序运 行任何代码之前完全的控制程序。这在分析病毒或者恶意代码的时候非常有用。附加到一个 进程，仅仅是强行的进入一个已经运行了的进程内部，它允许你跳过启动部分的代码，分析 你感兴趣的代码。你正在分析的地方也就是程序目前正在执行的地方。

第一种方法，其实就是从调试器本身调用这个程序（调试器就是父进程，对被调试进程 的控制权限更大）。在 Windows 上创建一个进程用 CreateProcessA()函数。将特定的标志传 进这个函数，使得目标进程能够被调试。一个 CreateProcessA()调用看起来像这样：

```py
BOOL WINAPI CreateProcessA( 
    LPCSTR lpApplicationName, 
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

初看这个调用相当恐怖，不过，在逆向工程中我们必须把大的部分分解成小的部分以便理解。这里我们只关心在调试器中创建一个进程需要注意的参数。这些参数是 lpApplicationName,lpCommandLine,dwCreationFlags,lpStartupInfo, 和 lpProcessInformation。 剩余的参数可以设置成空值（NULL）。关于这个函数的详细解释可以查看 MSDN(微软之葵 花宝典)。最前面的两个参数用于设置，需要执行的程序的路径和我们希望传递给程序的参 数。dwCreationFlags （创建标记）参数接受一个特定值，表示我们希望程序以被调试的状 态 启 动 。 最 后 两 个 参 数 分 别 分 别 指 向 2 个 结 构 (STARTUPINFO and PROCESS_INFORMATION) ，不仅包含了进程如何启动，以及启动后的许多重要信息 。（ lpStartupInfo ： STARTUPINFO 结 构 ， 用 于 在 创 建 子 进 程 时 设 置 各 种 属 性 ， lpProcessInformation：PROCESS_INFORMATION 结构，用来在进程创建后接收相关信息 ， 该结构由系统填写。）

创建两个 Python 文件 my_debugger.py 和 my_debugger_defines.py。我们将创建一个父类 debugger() 接 着 逐 渐 的 增 加 各 种 调 试 函 数 。 另 外 ， 把 所 有 的 结 构 ， 联 合 ， 常 量 放 到 my_debugger_defines.py 方便以后维护。

```py
# my_debugger_defines.py
from ctypes import *
# Let's map the Microsoft types to ctypes for clarity 
WORD = c_ushort
DWORD = c_ulong
LPBYTE = POINTER(c_ubyte) LPTSTR = POINTER(c_char)
HANDLE = c_void_p
# Constants
DEBUG_PROCESS = 0x00000001 
CREATE_NEW_CONSOLE = 0x00000010
# Structures for CreateProcessA() function 
class STARTUPINFO(Structure):
    _fields_ = [
        ("cb", DWORD),
        ("lpReserved", LPTSTR), 
        ("lpDesktop", LPTSTR),
        ("lpTitle", LPTSTR),
        ("dwX", DWORD),
        ("dwY", DWORD),
        ("dwXSize", DWORD),
        ("dwYSize", DWORD),
        ("dwXCountChars", DWORD), 
        ("dwYCountChars", DWORD), 
        ("dwFillAttribute",DWORD), 
        ("dwFlags", DWORD), 
        ("wShowWindow", WORD), 
        ("cbReserved2", WORD), 
        ("lpReserved2", LPBYTE), 
        ("hStdInput", HANDLE), 
        ("hStdOutput", HANDLE), 
        ("hStdError", HANDLE),
    ]
class PROCESS_INFORMATION(Structure):
    _fields_ = [
        ("hProcess", HANDLE), 
        ("hThread", HANDLE), 
        ("dwProcessId", DWORD), 
        ("dwThreadId", DWORD),
    ]
# my_debugger.py
from ctypes import *
from my_debugger_defines import * 
kernel32 = windll.kernel32
class debugger():
    def init (self): 
        pass
    def load(self,path_to_exe):
        # dwCreation flag determines how to create the process
        # set creation_flags = CREATE_NEW_CONSOLE if you want
        # to see the calculator GUI 
        creation_flags = DEBUG_PROCESS
        # instantiate the structs
        startupinfo = STARTUPINFO() 
        process_information = PROCESS_INFORMATION()
        # The following two options allow the started process
        # to be shown as a separate window. This also illustrates
        # how different settings in the STARTUPINFO struct can affect
        # the debuggee. 
        startupinfo.dwFlags = 0x1 
        startupinfo.wShowWindow = 0x0
        # We then initialize the cb variable in the STARTUPINFO struct
        # which is just the size of the struct itself 
        startupinfo.cb = sizeof(startupinfo)
        if kernel32.CreateProcessA(path_to_exe,
                                    None, 
                                    None, 
                                    None, 
                                    None,
                                    creation_flags, 
                                    None,
                                    None, 
                                    byref(startupinfo),
                                    byref(process_information)): 
            print "[*] We have successfully launched the process!" print "[*] PID: %d" % process_information.dwProcessId
        else:
            print "[*] Error: 0x%08x." % kernel32.GetLastError() 
```

现在我们将构造一个简短的测试模块确定一下一切都能正常工作。调用 my_test.py，保 证前面的文件都在同一个目录下。

```py
#my_test.py
import my_debugger
debugger = my_debugger.debugger() debugger.load("C:\\WINDOWS\\system32\\calc.exe") 
```

如果你是通过命令行或者 IDE 手动输入上面的代码，将会新产生一个进程也就是你键 入程序名，然后返回进程 ID（PID），最后结束。如果你用上面的例子 calc.exe，你将看不到 计算器的图形界面出现。因为进程没有把界面绘画到屏幕上，它在等待调试器继续执行的命 令。很快我们就能让他继续执行下去了。不过在这之前，我们已经找到了如何产生一个进程 用于调试，现在让我们实现另一个功能，附加到一个正在运行的进程。

为了附加到指定的进程，就必须先得到它的句柄。许多后面将用到的函数都需要句柄做 参数，同时我们也能在调试之前确认是否有权限调试它（如果附加都不行，就别提调试了）。 这个任务由 OpenProcess()完成，此函数由 kernel32.dll 库倒出，原型如下：

```py
HANDLE WINAPI OpenProcess( 
    DWORD dwDesiredAccess, 
    BOOL bInheritHandle 
    DWORD dwProcessId
); 
```

dwDesiredAccess 参数决定了我们希望对将要打开的进程拥有什么样的权限（当然是 越 大 越 好 root is hack ）。 因 为 要 执 行 调 试 ， 我 们 设 置 成 PROCESS_ALL_ACCESS 。 bInheritHandle 参数设置成 False，dwProcessId 参数设置成我们希望获得句柄的进程 ID ，也就是前面获得的 PID。如果函数成功执行，将返回一个目标进程的句柄。

接下来用 DebugActiveProcess()函数附加到目标进程：

```py
BOOL WINAPI DebugActiveProcess(
    DWORD dwProcessId
); 
```

把需要 a 附加的 PID 传入。一旦系统认为我们有权限访问目标进程，目标进程就假定 我们的调试器已经准备好处理调试事件，然后把进程的控制权转移给调试器。调试器接着循 环调用 WaitForDebugEvent()以便俘获调试事件。函数原型如下：

```py
BOOL WINAPI WaitForDebugEvent(
    LPDEBUG_EVENT lpDebugEvent, 
    DWORD dwMilliseconds
); 
```

第一个参数指向 DEBUG_EVENT 结构，这个结构描述了一个调试事件。第二个参数设 置成 INFINITE（无限等待），这样 WaitForDebugEvent() 就不用返回，一直等待直到一个事 件产生。

调试器捕捉的每一个事件都有相关联的事件处理函数，在程序继续执行前可以完成不同 的 操 作 。 当 处 理 函 数 完 成 了 操 作 ， 我 们 希 望 进 程 继 续 执 行 用 ， 这 时 候 再 调 用 ContinueDebugEvent()。原型如下：

```py
BOOL WINAPI ContinueDebugEvent( 
    DWORD dwProcessId,
    DWORD dwThreadId, 
    DWORD dwContinueStatus
); 
```

dwProcessId 和 dwThreadId 参数由 DEBUG_EVENT 结构里的数据填充，当调试器捕捉 到调试事件的时候，也就是 WaitForDebugEvent()成功执行的时候，进程 ID 和线程 ID 就以 及初始化好了。dwContinueStatus 参数告诉进程是继续执行(DBG_CONTINUE)，还是产生异 常(DBG_EXCEPTION_NOT_HANDLED)。

还剩下一件事没做，从进程分离出来：把进程 ID 传递给 DebugActiveProcessStop()。 现在我们把这些全合在一起，扩展我们的 my_debugger 类，让他拥有附加和分离一个进程的功能。同时加上打开一个进程和获得进程句柄的能力。最后在我们的主循环里完成事 件处理函数。打开 my_debugger.py 键入以下代码。

提示：所有需要的结构,联合和常量都定义在了 debugger_defines.py 文件里，完整的代码可 以从 [`www.nostarch.com/ghpython.htm`](http://www.nostarch.com/ghpython.htm) 下载。

```py
#my_debugger.py
from ctypes import *
from my_debugger_defines import * 
kernel32 = windll.kernel32
class debugger():
    def init (self):
        self.h_process = None
        self.pid = None 
        self.debugger_active = False
    def load(self,path_to_exe):
        ...
        print "[*] We have successfully launched the process!" 
        print "[*] PID: %d" % process_information.dwProcessId
        # Obtain a valid handle to the newly created process
        # and store it for future access
        self.h_process = self.open_process(process_information.dwProcessId)
        ...
    def open_process(self,pid):
        h_process = kernel32.OpenProcess(PROCESS_ALL_ACCESS,pid,False) 
        return h_process
    def attach(self,pid):
        self.h_process = self.open_process(pid)
        # We attempt to attach to the process
        # if this fails we exit the call
        if kernel32.DebugActiveProcess(pid): 
            self.debugger_active = True 
            self.pid = int(pid) 
            self.run()
        else:
            print "[*] Unable to attach to the process."
    def run(self):
        # Now we have to poll the debuggee for
        # debugging events
        while self.debugger_active == True: 
            self.get_debug_event()
    def get_debug_event(self):
        debug_event = DEBUG_EVENT() 
        continue_status= DBG_CONTINUE
        if kernel32.WaitForDebugEvent(byref(debug_event),INFINITE):
            # We aren't going to build any event handlers
            # just yet. Let's just resume the process for now. 
            raw_input("Press a key to continue...") 
            self.debugger_active = False
            kernel32.ContinueDebugEvent( \ 
                debug_event.dwProcessId, \ 
                debug_event.dwThreadId, \ 
                continue_status )
    def detach(self):
        if kernel32.DebugActiveProcessStop(self.pid): 
            print "[*] Finished debugging. Exiting..." 
            return True
        else:
            print "There was an error" return False 
```

现在让我们修改下测试套件以便使用新创建的函数。

```py
#my_test.py
import my_debugger
debugger = my_debugger.debugger()
pid = raw_input("Enter the PID of the process to attach to: ") 
debugger.attach(int(pid))
debugger.detach() 
```

按以下的步骤进行测试（windows 下）：

1.  选择 开始->运行->所有程序->附件->计算器

2.  右击桌面低端的任务栏，从退出的菜单中选择任务管理器。

3.  选择进程面板.

4.  如果你没看到 PID 栏，选择 查看->选择列

5.  确保进程标识符(PID)前面的确认框是选中的，然后单击 OK。

6.  找到 calc.exe 相关联的 PID

7.  执行 my_test.py 同时前面找到的 PID 传递给它。

8.  当 Press a key to continue...打印在屏幕上的时候，试着操作计算器的界面。你应该什么键都 按不了。这是因为进程被调试器挂起来了，等待进一步的指示。

9.  在你的 Python 控制台里按任何的键，脚本将输出别的信息，热爱后结束。

10.  现在你能够操作计算器了。

如果一切都如描绘的一样正常工作，把下面两行从 my_debugger.py 中注释掉：

```py
# raw_input("Press any key to continue...")
# self.debugger_active = False 
```

现在我们已经讲解了获取进程句柄的基础知识，以及如何创建一个进程，附加一个运行 的进程，接下来让我们给调试器加入更多高级的功能。

# 3.2 获得 CPU 寄存器状态

## 3.2 获得 CPU 寄存器状态

一个调试器必须能够在任何时候都搜集到 CPU 的各个寄存器的状态。当异常发生的时 候这能让我们确定栈的状态，目前正在执行的指令是什么，以及其他一些非常有用的信息。 要实现这个目的，首先要获取被调试目标内部的线程句柄，这个功能由 OpenThread()实现. 函数原型如下:

```py
HANDLE WINAPI OpenThread( 
    DWORD dwDesiredAccess, 
    BOOL bInheritHandle, 
    DWORD dwThreadId
); 
```

这看起来非常像 OpenProcess()的姐妹函数，除了这次是用线程标识符（thread identifier TID） 提到了进程标识符（PID）。

我们必须先获得一个执行着的程序内部所有线程的一个列表，然后选择我们想要的，再 用 OpenThread() 获 取 它 的 句 柄 。 让 我 研 究 下 如 何 在 一 个 系 统 里 枚 举 线 程 （ enumerate threads）。

### 3.2.1 枚举线程

为了得到一个进程里寄存器的状态，我们必须枚举进程内部所有正在运行的线程。线程 是进程中真正的执行体（大部分活都是线程干的），即使一个程序不是多线程的，它也至少 有一个线程，主线程。实现这一功能的是一个强大的函数 CreateToolhelp32Snapshot()，它由 kernel32.dll 导出。这个函数能枚举出一个进程内部所有线程的列表，以加载的模块（DLLs） 的列表，以及进程所拥有的堆的列表。函数原型如下：

```py
HANDLE WINAPI CreateToolhelp32Snapshot( 
    DWORD dwFlags,
    DWORD th32ProcessID
); 
```

dwFlags 参数标志了我们需要收集的数据类型（线程，进程，模块，或者堆 ）。这里我 们把它设置成 TH32CS_SNAPTHREAD，也就是 0x00000004，表示我们要搜集快照 snapshot 中 所 有 已 经 注 册 了 的 线 程 。 th32ProcessID 传 入 我 们 要 快 照 的 进 程 ， 不 过 它 只 对 TH32CS_SNAPMODULE, TH32CS_SNAPMODULE32, TH32CS_SNAPHEAPLIST, and TH32CS_SNAPALL 这几个模块有用,对 TH32CS_SNAPTHREAD 可是没什么用的哦（后面 有说明）。当 CreateToolhelp32Snapshot()调用成功，就会返回一个快照对象的句柄，被接下 来的函数调以便搜集更多的数据。

一旦我们从快照中获得了线程的列表，我们就能用 Thread32First()枚举它们了。函数原型如下：

```py
BOOL WINAPI Thread32First( 
    HANDLE hSnapshot, 
    LPTHREADENTRY32 lpte
); 
```

hSnapshot 就 是 上 面 通 过 CreateToolhelp32Snapshot() 获 得 镜 像 句 柄 ， lpte 指 向 一 个 THREADENTRY32 结构（必须初始化过）。这个结构在 Thread32First()在调用成功后自动填 充，其中包含了被发现的第一个线程的相关信息。结构定义如下：

```py
typedef struct THREADENTRY32{ 
    DWORD dwSize;
    DWORD cntUsage; 
    DWORD th32ThreadID;
    DWORD th32OwnerProcessID; 
    LONG tpBasePri;
    LONG tpDeltaPri; 
    DWORD dwFlags;
}; 
```

在这个结构中我们感兴趣的是 dwSize, th32ThreadID, 和 th32OwnerProcessID 3 个参数。 dwSize 必须在 Thread32First()调用之前初始化，只要把值设置成 THREADENTRY32 结构的 大小就可以了。th32ThreadID 是我们当前发现的这个线程的 TID，这个参数可以被前面说过 的 OpenThread() 函数调用以打开此线程，进行别的操作。 th32OwnerProcessID 填充了当前 线 程 所 属 进 程 的 PID 。 为 了 确 定 线 程 是 否 属 于 我 们 调 试 的 目 标 进 程 ， 需 要 将 th32OwnerProcessID 的值和目标进程对比，相等则说明这个线程是我们正在调试的。一旦我 们获得了第一个线程的信息，我们就能通过调用 Thread32Next()获取快照中的下一个线程条 目。它的参数和 Thread32First()一样。循环调用 Thread32Next()直到列表的末端。

### 3.2.2 把所有的组合起来

现在我们已经获得了一个线程的有效句柄，最后一步就是获取所有寄存器的值。这就需 要通过 GetThreadContext()来实现。同样我们也能用 SetThreadContext()改变它们。

```py
BOOL WINAPI GetThreadContext( 
    HANDLE hThread, 
    LPCONTEXT lpContext
);

BOOL WINAPI SetThreadContext( 
    HANDLE hThread, 
    LPCONTEXT lpContext
); 
```

hThread 参数是从 OpenThread() 返回的线程句柄，lpContext 指向一个 CONTEXT 结构， 其中存储了所有寄存器的值。CONTEXT 非常重要，定义如下：

```py
typedef struct CONTEXT { 
    DWORD ContextFlags; 
    DWORD Dr0;
    DWORD Dr1;
    DWORD Dr2;
    DWORD Dr3;
    DWORD Dr6;
    DWORD Dr7;
    FLOATING_SAVE_AREA FloatSave;
    DWORD  SegGs; 
    DWORD  SegFs; 
    DWORD  SegEs; 
    DWORD  SegDs; 
    DWORD  Edi; 
    DWORD  Esi; 
    DWORD  Ebx; 
    DWORD  Edx; 
    DWORD  Ecx; 
    DWORD  Eax; 
    DWORD  Ebp; 
    DWORD  Eip; 
    DWORD  SegCs; 
    DWORD  EFlags; 
    DWORD  Esp; 
    DWORD  SegSs; 
    BYTE  ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION]; 
}; 
```

如你说见所有的寄存器都在这个列表中了，包括调试寄存器和段寄存器。在我们剩下的 工作中，将大量的使用到这个结构，所以尽快的实习起来。

让我们回来看看我们的老朋友 my_debugger.py 继续扩展它，增加枚举线程和获取寄存 器的功能。

```py
#my_debugger.py
class debugger():
    ...
    def open_thread (self, thread_id):
        h_thread = kernel32.OpenThread(THREAD_ALL_ACCESS, None,
        thread_id)
        if h_thread is not None:
            return h_thread
        else:
            print "[*] Could not obtain a valid thread handle." 
            return False
    def enumerate_threads(self):
        thread_entry = THREADENTRY32()
        36 Chapter 3
        thread_list = []
        snapshot = kernel32.CreateToolhelp32Snapshot(TH32CS
        _SNAPTHREAD, self.pid)
        if snapshot is not None:
            # You have to set the size of the struct
            # or the call will fail
            thread_entry.dwSize = sizeof(thread_entry) success = kernel32.Thread32First(snapshot,
            byref(thread_entry))
            byref(thread_entry))
            while success:
                if thread_entry.th32OwnerProcessID == self.pid: 
                    thread_list.append(thread_entry.th32ThreadID)
                success = kernel32.Thread32Next(snapshot,
                kernel32.CloseHandle(snapshot) return thread_list
        else:
            return False
    def get_thread_context (self, thread_id): 
        context = CONTEXT()
        context.ContextFlags = CONTEXT_FULL | CONTEXT_DEBUG_REGISTERS
        # Obtain a handle to the thread 
        h_thread = self.open_thread(thread_id)
        if kernel32.GetThreadContext(h_thread, byref(context)): 
            kernel32.CloseHandle(h_thread)
            return context
        else:
            return False 
```

调试器已经扩展成功，让我们更新测试模块试验下新功能。

```py
#my_test.py
import my_debugger
debugger = my_debugger.debugger()
pid = raw_input("Enter the PID of the process to attach to: ") debugger.attach(int(pid))
list = debugger.enumerate_threads()
# For each thread in the list we want to
# grab the value of each of the registers Building a Windows Debugger 37
for thread in list:
    thread_context = debugger.get_thread_context(thread)
    # Now let's output the contents of some of the registers
    print "[*] Dumping registers for thread ID: 0x%08x" % thread 
    print "[**] EIP: 0x%08x" % thread_context.Eip
    print "[**] ESP: 0x%08x" % thread_context.Esp 
    print "[**] EBP: 0x%08x" % thread_context.Ebp 
    print "[**] EAX: 0x%08x" % thread_context.Eax 
    print "[**] EBX: 0x%08x" % thread_context.Ebx 
    print "[**] ECX: 0x%08x" % thread_context.Ecx 
    print "[**] EDX: 0x%08x" % thread_context.Edx 
    print "[*] END DUMP"
debugger.detach() 
```

当你运行测试代码，你将看到如清单 3-1 显示的数据。

```py
Enter the PID of the process to attach to: 4028
[*] Dumping registers for thread ID: 0x00000550 
[**] EIP: 0x7c90eb94
[**] ESP: 0x0007fde0 
[**] EBP: 0x0007fdfc 
[**] EAX: 0x006ee208 
[**] EBX: 0x00000000
[**] ECX: 0x0007fdd8 
[**] EDX: 0x7c90eb94 
[*] END DUMP
[*] Dumping registers for thread ID: 0x000005c0 
[**] EIP: 0x7c95077b
[**] ESP: 0x0094fff8 
[**] EBP: 0x00000000 
[**] EAX: 0x00000000 
[**] EBX: 0x00000001 
[**] ECX: 0x00000002 
[**] EDX: 0x00000003 
[*] END DUMP
[*] Finished debugging. Exiting... 
```

Listing 3-1:每个线程的 CPU 寄存器值

太酷了 ! 我们现在能够在任何时候查询所有寄存器的状态了。试验下不同的进程 ,看看能得到什么结果。到此为止我们已经完成了我们调试器的核心部分，是时间实现一些基础 调试事件的处理函数了。

# 3.3 实现调试事件处理

## 3.3 实现调试事件处理

为了让我们的调试器能够针对特定的事件采取相应的行动，我们必须给所有调试器能够 捕捉到的调试事件，编写处理函数。回去看看 WaitForDebugEvent() 函数，每当它捕捉到一 个调试事件的时候，就返回一个填充好了的 DEBUG_EVENT 结构。之前我们都忽略掉这个 结构，直接让进程继续执行下去，现在我们要用存储在结构里的信息决定如何处理调试事件。 DEBUG_EVENT 定义如下:

```py
typedef struct DEBUG_EVENT { 
    DWORD dwDebugEventCode; 
    DWORD dwProcessId; 
    DWORD dwThreadId;
    union {
        EXCEPTION_DEBUG_INFO Exception; 
        CREATE_THREAD_DEBUG_INFO CreateThread; 
        CREATE_PROCESS_DEBUG_INFO CreateProcessInfo; 
        EXIT_THREAD_DEBUG_INFO ExitThread; 
        EXIT_PROCESS_DEBUG_INFO ExitProcess; 
        LOAD_DLL_DEBUG_INFO LoadDll;
        UNLOAD_DLL_DEBUG_INFO UnloadDll; 
        OUTPUT_DEBUG_STRING_INFO DebugString; 
        RIP_INFO RipInfo;
    }u;
}; 
```

在这个结构中有很多有用的信息。dwDebugEventCode 是最重要的，它表明了是什么事 件被 WaitForDebugEvent() 捕捉到了。同时也决定了，在联合(union )u 里存储的是什么类型 的值。u 里的变量由 dwDebugEventCode 决定，一一对应如下：

| Event Code | Event Code Value | Union u Value |
| --- | --- | --- |
| 0x1 | EXCEPTION_DEBUG_EVENT | u.Exception |
| 0x2 | CREATE_THREAD_DEBUG_EVENT | u.CreateThread |
| 0x3 | CREATE_PROCESS_DEBUG_EVENT | u.CreateProcessInfo |
| 0x4 | EXIT_THREAD_DEBUG_EVENT | u.ExitThread |
| 0x5 | EXIT_PROCESS_DEBUG_EVENT | u.ExitProcess |
| 0x6 | LOAD_DLL_DEBUG_EVENT | u.LoadDll |
| 0x7 | UNLOAD_DLL_DEBUG_EVENT | u.UnloadDll |
| 0x8 | OUPUT_DEBUG_STRING_EVENT | u.DebugString |
| 0x9 | RIP_EVENT | u.RipInfo |

Table 3-1:调试事件

通过观察 dwDebugEventCode 的值，再通过上面的表就能找到与之相对应的存储在 u 里 的变量。让我们修改调试循环，通过获得的事件代码的值，显示当前发生的事件信息。用这些信息，我们能够了解到调试器启动或者附加一个线程后的整个流程。继续更新 my_debugger.py 和 our my_test.py 脚本。

```py
#my_debugger.py
...
class debugger():
    def init (self):
        self.h_process = None
        self.pid = None 
        self.debugger_active = False 
        self.h_thread = None
        self.context = None
        ...
    def get_debug_event(self):
        debug_event = DEBUG_EVENT() 
        continue_status= DBG_CONTINUE
        if kernel32.WaitForDebugEvent(byref(debug_event),INFINITE):
            # Let's obtain the thread and context information 
            self.h_thread = self.open_thread(debug_event.dwThread) 
            self.context = self.get_thread_context(self.h_thread)
            print "Event Code: %d Thread ID: %d" % (debug_event.dwDebugEventCode, debug_event.dwThre
            kernel32.ContinueDebugEvent( 
                debug_event.dwProcessId, 
                debug_event.dwThreadId, 
                continue_status )
#my_test.py
import my_debugger
debugger = my_debugger.debugger()
pid = raw_input("Enter the PID of the process to attach to: ") 
debugger.attach(int(pid))
debugger.run() 
debugger.detach() 
```

如果你用的是 calc.exe，输出将如下所示：

```py
Enter the PID of the process to attach to: 2700 
Event Code: 3 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 6 Thread ID: 3976
Event Code: 2 Thread ID: 3912
Event Code: 1 Thread ID: 3912
Event Code: 4 Thread ID: 3912 
```

Listing 3-2: 当附加到 cacl.exe 时的事件代码

基于脚本的输出，我们能看到 CREATE_PROCESS_EVENT (0x3)事件是第一个发生的， 接 下 来 的 是 一 堆 的 LOAD_DLL_DEBUG_EVENT (0x6) 事 件 ， 然 后 CREATE_THREAD_DEBUG_EVENT (0x2) 创 建 一 个 新 线 程 。 接 着 就 是 一 个 EXCEPTION_DEBUG_EVENT (0x1)例外事件，它由 windows 设置的断点所引发的，允许在 进程启动前观察进程的状态。最后一个事件是 EXIT_THREAD_DEBUG_EVENT (0x4)，它 由进程 3912 结束只身产生。

例外事件是非常重要，例外可能包括断点，访问异常，或者内存访问错误（例如尝试写 到一个只读的内存区）。所有这些都很重要，但是让我们捕捉先捕捉第一个 windows 设置的 断点。打开 my_debugger.py 加入以下代码：

```py
#my_debugger.py
...
class debugger():
    def init (self):
        self.h_process = None
        self.pid = None 
        self.debugger_active = False 
        self.h_thread = None
        self.context = None
        self.exception = None 
        self.exception_address = None
        ...
    def get_debug_event(self):
        debug_event = DEBUG_EVENT()
        continue_status= DBG_CONTINUE
        if kernel32.WaitForDebugEvent(byref(debug_event),INFINITE):
            # Let's obtain the thread and context information 
            self.h_thread = self.open_thread(debug_event.dwThreadId) 
            self.context = self.get_thread_context(self.h_thread)
            print "Event Code: %d Thread ID: %d" % (debug_event.dwDebugEventCode, debug_event.dwThreadId)
            # If the event code is an exception, we want to
            # examine it further.
            if debug_event.dwDebugEventCode == EXCEPTION_DEBUG_EVENT:
                # Obtain the exception code 
                exception = debug_event.u.Exception.ExceptionRecord.ExceptionCod
                self.exception_address = debug_event.u.Exception.ExceptionRecord.ExceptionAdd
            if exception == EXCEPTION_ACCESS_VIOLATION: 
                print "Access Violation Detected."
                # If a breakpoint is detected, we call an internal
                # handler.
            elif exception == EXCEPTION_BREAKPOINT: 
                continue_status = self.exception_handler_breakpoint()
            elif ec == EXCEPTION_GUARD_PAGE:
                print "Guard Page Access Detected." 
            elif ec == EXCEPTION_SINGLE_STEP:
                print "Single Stepping." 
            kernel32.ContinueDebugEvent( 
                debug_event.dwProcessId,
                debug_event.dwThreadId, 
                continue_status )
            ...
    def exception_handler_breakpoint():
        print "[*] Inside the breakpoint handler." 
        print "Exception Address: 0x%08x" % self.exception_address
        return DBG_CONTINUE 
```

如果你重新运行这个脚本，将看到由软件断点的异常处理函数打印的输出结果。我们已经创建了硬件断点和内存断点的处理模型。接下来我们要详细的实现这三种不同类型断点的 处理函数。

# 3.4 全能的断点

## 3.4 全能的断点

现在我们已经有了一个能够正常运行的调试器核心，是时候加入断点功能了。用我们在第二章学到的，实现设置软件，硬件，内存三种断点的功能。接着实现与之对应的断点处理 函数，最后在断点被击中之后干净的恢复进程。

### 3.4.1 软件断点

为了设置软件断点，我们必须能够将数据写入目标进程的内存。这需要通过 ReadProcessMemory() 和 WriteProcessMemory()实现。它们非常相似：

```py
BOOL WINAPI ReadProcessMemory(
    HANDLE hProcess,
    LPCVOID lpBaseAddress, 
    LPVOID lpBuffer,
    SIZE_T nSize,
    SIZE_T* lpNumberOfBytesRead
);

BOOL WINAPI WriteProcessMemory( 
    HANDLE hProcess,
    LPCVOID lpBaseAddress, 
    LPCVOID lpBuffer, 
    SIZE_T nSize,
    SIZE_T* lpNumberOfBytesWritten
); 
```

这两个函数都允许调试器观察和更新被调试的进程的内存。参数也都很简单。 lpBaseAddress 是要开始读或者些的目标地址， lpBuffer 指向一块缓冲区，用来接收 lpBaseAddress 读出的数据或者写入 lpBaseAddress 。 nSize 是 想 要 读 写 的 数 据 大 小 ， lpNumberOfBytesWritten 由函数填写，通过它我们就能够知道一次操作过后实际读写了的数 据。

现在让我们的调试器实现软件断点就相当容易了。修改调试器的核心类，以支持设置和 处理软件断点。

```py
#my_debugger.py
...
class debugger():
    def init (self):
        self.h_process = None
        self.pid = None 
        self.debugger_active = False
        ...
        self.h_thread = None
        self.context = None
        self.breakpoints = {}
    def read_process_memory(self,address,length): data = ""
        read_buf = create_string_buffer(length) 
        count = c_ulong(0)
        if not kernel32.ReadProcessMemory(self.h_process,
                                            address, 
                                            read_buf, 
                                            length, 
                                            byref(count)):
            return False
        else:
            data += read_buf.raw return data
    def write_process_memory(self,address,data): count = c_ulong(0)
        length = len(data)
        c_data = c_char_p(data[count.value:])
        if not kernel32.WriteProcessMemory(self.h_process,
                                            address, 
                                            c_data, 
                                            length, 
                                            byref(count)):

            return False
        else:
            return True
    def bp_set(self,address):
        if not self.breakpoints.has_key(address): 
            try:
                # store the original byte
                original_byte = self.read_process_memory(address, 1)
                # write the INT3 opcode 
                self.write_process_memory(address, "\xCC")
                # register the breakpoint in our internal list self.breakpoints[address] = (address, original_byte)
            except:
                return False 
        return True 
```

现在调试器已经支持软件断点了，我们需要找个地址设置一个试试看。一般断点设置在 函数调用的地方，为了这次实验，我们就用老朋友 printf()作为将要捕获的目标函数。WIndows 调试 API 提供了简洁的 方法以确定一个函数的虚拟地址， GetProcAddress()，同样也是从 kernel32.dll 导出的。这个 函数需要的主要参数就是一个模块（一个 dll 或者一个.exe 文件）的句柄。模块中一般都包含 了我们感兴趣的函数; 可以通过 GetModuleHandle()获得模块的句柄。原型如下：

```py
FARPROC WINAPI GetProcAddress( 
    HMODULE hModule,
    LPCSTR lpProcName
);
HMODULE WINAPI GetModuleHandle( 
    LPCSTR lpModuleName
); 
```

这是一个很清晰的事件链：获得一个模块的句柄，然后查找从中导出感兴趣的函数的地 址。让我们增加一个调试函数，完成刚才做的。回到 my_debugger.py.。

```py
my_debugger.py
...
class debugger():
    ...
    def func_resolve(self,dll,function):
        handle = kernel32.GetModuleHandleA(dll)
        address = kernel32.GetProcAddress(handle, function) 
        kernel32.CloseHandle(handle)
        return address 
```

现在创建第二个测试套件，循环的调用 printf()。我们将解析出函数的地址， 然后在这个地址上设置一个断点。之后断点被触发，就能看见输出结果，最后被测试的进程 继续执行循环。创建一个新的 Python 脚本 printf_loop.py，输入下面代码。

```py
#printf_loop.py from ctypes 
import * import time
msvcrt = cdll.msvcrt 
counter = 0
while 1:
    msvcrt.printf("Loop iteration %d!\n" % counter) 
    time.sleep(2)
    counter += 1 
```

现在更新测试套件，附加到进程，在 printf()上设置断点。

```py
#my_test.py
import my_debugger
debugger = my_debugger.debugger()
pid = raw_input("Enter the PID of the process to attach to: ") 
debugger.attach(int(pid))
printf_address = debugger.func_resolve("msvcrt.dll","printf") 
print "[*] Address of printf: 0x%08x" % printf_address 
debugger.bp_set(printf_address)
debugger.run() 
```

现在开始测试，在命令行里运行 printf_loop.py。从 Windows 任务管理器里获得 python.exe 的 PID。然后运行 my_test.py ，键入 PID。你将看到如下的输出：

```py
Enter the PID of the process to attach to: 4048 
[*] Address of printf: 0x77c4186a
[*] Setting breakpoint at: 0x77c4186a 
Event Code: 3 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 6 Thread ID: 3148
Event Code: 2 Thread ID: 3620
Event Code: 1 Thread ID: 3620
[*] Exception address: 0x7c901230 
[*] Hit the first breakpoint.
Event Code: 4 Thread ID: 3620
Event Code: 1 Thread ID: 3148
[*] Exception address: 0x77c4186a 
[*] Hit user defined breakpoint. 
```

Listing 3-3: 处理软件断点事件的事件顺序

我们首先看到 printf()的函数地址在 0x77c4186a，然后在这里设置断点。第一个捕捉到 的异常是由 Windows 设置的断点触发的。第二个异常发生的地址在 0x77c4186a,也就是 printf() 函数的地址。断点处理之后，进程将恢复循环。现在我们的调试器已经支持软件断点，接下 来轮到硬件断点了。

### 3.4.2 硬件断点

第二种类型的断点是硬件断点，通过设置相对应的 CPU 调试寄存器来实现。我们在之 前的章节已经详细的讲解了过程，现在来具体的实现它们。有一件很重要的事情要记住，当 我们使用硬件断点的时候要跟踪四个可用的调试寄存器哪个是可用的哪个已经被使用了。必 须确保我们使用的那个寄存器是空的，否则硬件断点就不能在我们希望的地方触发。

让我们开始枚举进程里的所有线程，然后获取它们的 CPU 内容拷贝。通过得到内容拷 贝，我们能够定义 DR0 到 DR3 寄存器的其中一个，让它包含目标断点地址。之后我们在 DR7 寄存器的相应的位上设置断 点的属性和长度。

设置断点的代码之前我们已经完成了，剩下的就是修改处理调试事件的主函数，让它能 够处理由硬件断点引发的异常。我们知道硬件断点由 INT1 (或者说是步进事件),所以我们就 只要就当的添加另一个异常处理函数到调试循环里。让我们设置断点。

```py
#my_debugger.py
...
class debugger():
    def init (self):
        self.h_process = None
        self.pid = None 
        self.debugger_active = False 
        self.h_thread = None
        self.context = None 
        self.breakpoints = {} 
        self.first_breakpoint= True 
        self.hardware_breakpoints = {}
        ...
    def bp_set_hw(self, address, length, condition):
        # Check for a valid length value 
        if length not in (1, 2, 4):
            return False
        else:
            length -= 1
        # Check for a valid condition
        if condition not in (HW_ACCESS, HW_EXECUTE, HW_WRITE): return False
        # Check for available slots
        if not self.hardware_breakpoints.has_key(0): 
            available = 0
        elif not self.hardware_breakpoints.has_key(1): 
            available = 1
        elif not self.hardware_breakpoints.has_key(2): 
            available = 2
        elif not self.hardware_breakpoints.has_key(3): 
            available = 3
        else:
            return False
        # We want to set the debug register in every thread 
        for thread_id in self.enumerate_threads():
            context = self.get_thread_context(thread_id=thread_id)
            # Enable the appropriate flag in the DR7
            # register to set the breakpoint 
            context.Dr7 |= 1 << (available * 2)
        # Save the address of the breakpoint in the
        # free register that we found 
        if available == 0:
            context.Dr0 = address 
        elif available == 1:
            context.Dr1 = address 
        elif available == 2:
            context.Dr2 = address 
        elif available == 3:
            context.Dr3 = address
        # Set the breakpoint condition
        context.Dr7 |= condition << ((available * 4) + 16)
        # Set the length
        context.Dr7 |= length << ((available * 4) + 18)
        # Set thread context with the break set 
        h_thread = self.open_thread(thread_id)
        kernel32.SetThreadContext(h_thread,byref(context))
        # update the internal hardware breakpoint array at the used
        # slot index.
        self.hardware_breakpoints[available] = (address,length,condition)
        return True 
```

通过确认全局的硬件断点字典，我们选择了一个空的调试寄存器存储硬件断点。一 旦我们得到空位，接下来做的就是将硬件断点的地址填入调试寄存器，然后对 DR7 的标志 位进行更新适当的更新，启动断点。现在我们已经能够处理硬件断点了，让我们更新事件处 理函数添加一个 INT1 中断的异常处理。

```py
#my_debugger.py
...
class debugger():
    ...
    ...
    def get_debug_event(self):
        if self.exception == EXCEPTION_ACCESS_VIOLATION: 
            print "Access Violation Detected."
        elif self.exception == EXCEPTION_BREAKPOINT: 
            continue_status = self.exception_handler_breakpoint()
        elif self.exception == EXCEPTION_GUARD_PAGE: 
            print "Guard Page Access Detected."
        elif self.exception == EXCEPTION_SINGLE_STEP: 
            self.exception_handler_single_step()
    def exception_handler_single_step(self):
        # Comment from PyDbg:
        # determine if this single step event occurred in reaction to a
        # hardware breakpoint and grab the hit breakpoint.
        # according to the Intel docs, we should be able to check for
        # the BS flag in Dr6\. but it appears that Windows
        # isn't properly propagating that flag down to us.
        if self.context.Dr6 & 0x1 and self.hardware_breakpoints.has_key(0): 
            slot = 0
        elif self.context.Dr6 & 0x2 and self.hardware_breakpoints.has_key(1): 
            slot = 1
        elif self.context.Dr6 & 0x4 and self.hardware_breakpoints.has_key(2): 
            slot = 2
        elif self.context.Dr6 & 0x8 and self.hardware_breakpoints.has_key(3): 
            slot = 3
        else:
            # This wasn't an INT1 generated by a hw breakpoint
            continue_status = DBG_EXCEPTION_NOT_HANDLED
        # Now let's remove the breakpoint from the list
        if self.bp_del_hw(slot):
            continue_status = DBG_CONTINUE 
        print "[*] Hardware breakpoint removed." 
        return continue_status
    def bp_del_hw(self,slot):
        # Disable the breakpoint for all active threads 
        for thread_id in self.enumerate_threads():
            context = self.get_thread_context(thread_id=thread_id)
            # Reset the flags to remove the breakpoint 
            context.Dr7 &= ~(1 << (slot * 2))
            # Zero out the address 
            if slot == 0:
                context.Dr0 = 0x00000000 
            elif slot == 1:
                context.Dr1 = 0x00000000 
            elif slot == 2:
                context.Dr2 = 0x00000000 
            elif slot == 3:
                context.Dr3 = 0x00000000
            # Remove the condition flag
            context.Dr7 &= ~(3 << ((slot * 4) + 16))
            # Remove the length flag
            context.Dr7 &= ~(3 << ((slot * 4) + 18))
            # Reset the thread's context with the breakpoint removed 
            h_thread = self.open_thread(thread_id) 
            kernel32.SetThreadContext(h_thread,byref(context))
        # remove the breakpoint from the internal list. 
        del self.hardware_breakpoints[slot]
        return True 
```

代码很容易理解；当 INT1 被击中（触发）的时候，查看是否有调试寄存器能够设置硬 件断点（通过检测 DR6）。如果有能够使用的就继续。接着如果在发生异常的地址发现一个 硬件断点，就将 DR7 的标志位置零，在其中的一个寄存器中填入断点的地址。让我们修改 my_test.py 并在 printf()上设置硬件断点看看。

```py
#my_test.py
import my_debugger
from my_debugger_defines import * 
debugger = my_debugger.debugger()
pid = raw_input("Enter the PID of the process to attach to: ") debugger.attach(int(pid))
printf = debugger.func_resolve("msvcrt.dll","printf") 
print "[*] Address of printf: 0x%08x" % printf
debugger.bp_set_hw(printf,1,HW_EXECUTE) debugger.run() 
```

这个测试模块在 printf()上设置了一个断点，只要调用函数，就会触发调试事件。断点 的长度是一个字节。你应该注意到在这个模块中我们导入了 my_debugger_defines.py 文件； 为的是访问 HW_EXECUTE 变量，这样书写能使代码更清晰。

运行后输出结果如下：

```py
Enter the PID of the process to attach to: 2504 
[*] Address of printf: 0x77c4186a
Event Code: 3 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 6 Thread ID: 3704
Event Code: 2 Thread ID: 2228
Event Code: 1 Thread ID: 2228
[*] Exception address: 0x7c901230 
[*] Hit the first breakpoint.
Event Code: 4 Thread ID: 2228
Event Code: 1 Thread ID: 3704
[*] Hardware breakpoint removed. 
```

Listing 3-4: 处理一个硬件断点事件的顺序

一切都在预料中，程序抛出异常，处理程序移除断点。事件处理完之后，程序继续 循环执行代码。现在我们的轻量级调试器已经支持硬件和软件断点了，最后来实现内存断点 吧。

### 3.4.3 内存断点

最后一个要实现的功能是内存断点。大概流程如下;首先查询一个内存块以并找到基地 址（页面在虚拟内存中的起始地址）。一旦确定了页面大小，接着就设置页面权限，使其成 为保护（guard）页。当 CPU 尝试访问这块内存时，就会抛出一个 GUARD_PAGE_EXCEPTION 异常。我们用对应的异常处理函数，将页面权限恢复到以前，最后让程序继续执行。

为了能准确的计算出页面的大小，就要向系统查询信息获得一个内存页的默认大小。这 由 GetSystemInfo()函数完成，函数会装填一个 SYSTEM_INFO 结构，这个结构包含 wPageSize 成员，这就是操作系统内存页默认大小。

```py
#my_debugger.py
...
class debugger():
    def init (self):
        self.h_process = None
        self.pid = None 
        self.debugger_active = False 
        self.h_thread = None
        self.context = None 
        self.breakpoints = {} 
        self.first_breakpoint= True self.hardware_breakpoints = {}
        # Here let's determine and store
        # the default page size for the system 
        system_info = SYSTEM_INFO() 
        kernel32.GetSystemInfo(byref(system_info)) 
        self.page_size = system_info.dwPageSize
        ... 
```

已经获得默认页大小，那剩下的就是查询和控制页面的权限。第一步让我们查询出内存断点存在于内存里的哪一个页面。调用 VirtualQueryEx() 函数，将会填充一个 MEMORY_BASIC_INFORMATION 结构，这个结构中包含了页的信息。函数和结构定义如 下:

```py
SIZE_T WINAPI VirtualQuery( 
    HANDLE hProcess, 
    LPCVOID lpAddress,
    PMEMORY_BASIC_INFORMATION lpBuffer,
    SIZE_T dwLength
);
typedef struct MEMORY_BASIC_INFORMATION{ 
    PVOID BaseAddress;
    PVOID AllocationBase; 
    DWORD AllocationProtect; 
    SIZE_T RegionSize; 
    DWORD State;
    DWORD Protect;
    DWORD Type;
} 
```

上面的结构中 BaseAddress 的值就是我们要设置权限的页面的开始地址。接下来用 VirtualProtectEx()设置权限，函数原型如下：

```py
BOOL WINAPI VirtualProtectEx( 
    HANDLE hProcess,
    LPVOID lpAddress, 
    SIZE_T dwSize, 
    DWORD flNewProtect, 
    PDWORD lpflOldProtect
); 
```

让我们着手写代码。我们将创建 2 个全局列表，其中一个包含所有已经设置了好了 的保护页，另一个包含了所有的内存断点，在处理 GUARD_PAGE_EXCEPTION 异常的时 候将用得着。之后我们将在断点地址上，以及周围的区域设置权限。（因为断点地址有可能 横跨 2 个页面）。

```py
#my_debugger.py
...
class debugger():
    def init (self):
        ...
        self.guarded_pages = [] 
        self.memory_breakpoints = {}
        ...
    def bp_set_mem (self, address, size):
        mbi = MEMORY_BASIC_INFORMATION()
        # If our VirtualQueryEx() call doesn’t return
        # a full-sized MEMORY_BASIC_INFORMATION
        # then return False
        if kernel32.VirtualQueryEx(self.h_process,
                                    address, 
                                    byref(mbi),
                                    sizeof(mbi)) < sizeof(mbi):
            return False
        current_page = mbi.BaseAddress
        # We will set the permissions on all pages that are
        # affected by our memory breakpoint.
        while current_page <= address + size:
            # Add the page to the list; this will
            # differentiate our guarded pages from those
            # that were set by the OS or the debuggee process self.guarded_pages.append(current_page)
            old_protection = c_ulong(0)
            if not kernel32.VirtualProtectEx(self.h_process, 
                                            current_page, 
                                            size,
                                            mbi.Protect | PAGE_GUARD, 
                                            byref(old_protection)): 
                return False
            # Increase our range by the size of the
            # default system memory page size 
            current_page += self.page_size
        # Add the memory breakpoint to our global list self.memory_breakpoints[address] = (address, size, mbi)
        return True 
```

现在我们已经能够设置内存断点了。如果用以前的 printf() 循环作为测试对象，你将看 到测试模块只是简单的输出 Guard Page Access Detected。不过有一件好事，就是系统替我们 完成了扫尾工作，一旦保护页被访问，就会抛出一个异常，这时候系统会移除页面的保护属 性，然后允许程序继续执行。不过你能做些别的，在调试的循环代码里，加入特定的处理过 程，在断点触发的时候，重设断点，读取断点处的内存，喝瓶‘蚁力神’（这个不强求，哈）， 或者干点别的。

总结

目前为止我们已经开发了一个基于 Windows 的轻量级调试器。不仅对创建调试器有了 深刻的领会，也学会了很多重要的技术，无论将来做不做调试都非常有用。至少在用别的调 试器的时候你能够明白底层做了些什么，也能够修改调试器，让它更好用。这些能让你更强！ 更强！

下一步是展示下调试器的高级用法，分别是 PyDbg 和 Immunity Debugger，它们成熟稳 定而且都有基于 Windows 的版本。揭开 PyDbg 工作的方式，你将得到更多的有用的东西， 也将更容易的深入了解它。Immunity 调试器结构有轻微的不同，却提供了非常多不同的优 点。明白这它们实现特定调试任务的方法对于我们实现自动化调试非常重要。接下来轮到 PyDbg 上产。好戏开场。我先睡觉 ing。