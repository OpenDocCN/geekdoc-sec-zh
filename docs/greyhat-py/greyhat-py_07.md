# 六、HOOKING

# 6 HOOKING

Hooking 是一种强大的进程监控 (process-observation)技 术,通过改变进程的流程，以监视进程中数据的访问和改变。 Hooking 常用于隐藏 rootkits，窃取按键信息，还有调试工作。在逆向调试中，通过构建简单的 hook 检索我们需要的信息，能够节省很多手工操作的时间。hook，简单而强大。

在 Windows 系统中，有非常多的方法实现 hook。我们主要介绍两种：soft hook 和 hard hook。soft hook 就是在要附加的目标进程中，插入 INT3 中断，接管进程的执行流程。这和 58 夜的“扩展断点处理”很像。hard hook 则是在目标进程中硬编码( hard-coding)一个跳转 到 hook 代码(用汇编代码编写)。Soft hook 在频繁的函数调用中很有用。然而，为了对目标 进 程 产 生 最 小 的 影 响 就 必 须 用 到 hard hook 。 有 两 种 主 要 的 hard hook ， 分 别 是 heap-management routines 和 intensive file I/O operations。

我们在前面介绍的工具实现 hook。用 PyDbg 实现 soft hook 用于嗅探加密的网络传输。 用 Immunity 实现 hard hook 做一些高效的 heap instrumentation。

# 6.1 用 PyDbg 实现 Soft Hooking

## 6.1 用 PyDbg 实现 Soft Hooking

第一个例子就是在应用层嗅探加密的网络传输。平时为了明白客户端和服务器之间的工 作流程，我们都会使用一个网络分析器列如 Wireshark。很不幸的是，Wireshark 获得的数据 经常都是加密过的，使得协议分析变得模糊。用 soft hooking 你能够在数据加密前或者接受 并解密后捕获它们。

实验目标就是最流行的开源浏览器 Mozilla Firefox。为了这次实验，我们假设 Firefox 是闭源的（否则会相当没趣）。我们的任务就是在 firefox.exe 进程加密数据前嗅探出数据。 现在最通用的网络加密协议就是 SSL，这次的主要目标就是解决她。

为了跟踪函数的调用（未加密数据的传递），需要使用记录模块间调用的技巧 [`forum.immunityinc.com/index.php?topic=35.0`](http://forum.immunityinc.com/index.php?topic=35.0) )。现在首要解决的问题就是在什么地方 设置 hook。我们先假定将 hook 设置在 PR_Write 函数上（由 nspr4.dll.导出）。当这个函数被 执行的时候，堆栈[ ESP + 8 ]指向 ASCII 字符串（包含我们提交的但未加密的数据）。 ESP + 8 说明它是 PR_Write 的第二个函数，也是我们需要的，记录它，恢复程序。

首先打开 Firefox，输入网址 [`www.openrce.org/`](http://www.openrce.org/)。一旦你接收了 SSl 证书，页面就 加载成功。接着 Immunity 附加到 firefox.exe 进程在 nspr4.PR_Write 设置断点。在 OpenRCE 网站右上角有一个登录窗口，设置用户名为 test 和密码 test，点击 Login 按钮。设置的断点 立刻被触发；再按 F9，断点再次触发。最后，你将在栈看到如下的内容：

```py
[ESP + 8] => ASCII "username=test&password=test&remember_me=on" 
```

很好，我们很清晰的看到了用户名和密码。但是如果从网络层看传输的数据，将是一堆 经过 SSL 加密的无意义的数据。这种方法不仅对 OpenRCE 有效。当你浏览任何一个需要传 输敏感数据的网站的时候，这些数据都将很容易的被捕捉到。现在再也不用手工操作调试器 去捕捉了，自动化才是王道。

在用 PyDbg 定义 soft hook 之前，需要先定义一个包含说有 hook 目标的容器。如下初始 化容器：

```py
hooks = utils.hook_container() 
```

使用 hook_container 类的 add()方法将我们定义的 hook 加进去。函数原型：

```py
add( pydbg, address, num_arguments, func_entry_hook, func_exit_hook ) 
```

第一个参数设置成一个有效的 pydbg 目标，address 参数设置成要安装 hook 的地址， num_arguments 设置成传递给 hook 的参数。func_entry_hook 和 func_exit_hook 都是回调函数。 func_entry_hook 是 hook 被触发后立刻调用的，func_exit_hook 是被 hook 的函数将要退出之 前执行的。entry hook 用于得到函数的参数，exit hook 用于捕捉函数的返回值。

```py
def entry_hook( dbg, args ):
    # Hook code here
    return DBG_CONTINUE 
```

dbg 参数设置成有效的 pydbg 目标，args 接收一个列表，包含 hook 触发时接收到的参 数。

exit hook 回调函数有一点不同就是多了个 ret 参数，包含了函数的返回值(EAX 的值): def exit_hook( dbg, args, ret ):

```py
# Hook code here
return DBG_CONTINUE 
```

接下用实例看看如何用 entry hook 嗅探加密前的数据。

```py
#firefox_hook.py from pydbg import *
from pydbg.defines import * 
import utils
import sys
dbg = pydbg()
found_firefox = False
# Let's set a global pattern that we can make the hook
# search for
pattern = "password"
# This is our entry hook callback function
# the argument we are interested in is args[1] 
def ssl_sniff( dbg, args ):
    # Now we read out the memory pointed to by the second argument
    # it is stored as an ASCII string, so we'll loop on a read until
    # we reach a NULL byte buffer = ""
    offset = 0
    while 1:
        byte = dbg.read_process_memory( args[1] + offset, 1 ) 
        if byte != "\x00":
            buffer += byte offset += 1 continue
        else:
            break
    if pattern in buffer:
        print "Pre-Encrypted: %s" % buffer 
    return DBG_CONTINUE
# Quick and dirty process enumeration to find firefox.exe 
for (pid, name) in dbg.enumerate_processes():
    if name.lower() == "firefox.exe": 
        found_firefox = True
        hooks = utils.hook_container() dbg.attach(pid)
        print "[*] Attaching to firefox.exe with PID: %d" % pid
        # Resolve the function address
        hook_address = dbg.func_resolve_debuggee("nspr4.dll","PR_Wri 
        if hook_address:
            # Add the hook to the container. We aren't interested
            # in using an exit callback, so we set it to None. 
            hooks.add( dbg, hook_address, 2, ssl_sniff, None )
            print "[*] nspr4.PR_Write hooked at: 0x%08x" % hook_address 
            break
        else:
            print "[*] Error: Couldn't resolve hook address." 
            sys.exit(-1)
if found_firefox:
    print "[*] Hooks set, continuing process." 
    dbg.run()
else:
    print "[*] Error: Couldn't find the firefox.exe process." 
    sys.exit(-1) 
```

代码简洁明了:在 PR_Write 上设置 hook，当 hook 被触发的时候，我们尝试读出第二个 参数指向的字符串。如果有符合的数据就打印在命令行。启动一个新的 Firefox，接着运行 firefox_hook.py 脚本。重复之前的步骤，登录 [`www.openrce.org/`](http://www.openrce.org/)，将看到输出如下：

```py
[*] Attaching to firefox.exe with PID: 1344 
[*] nspr4.PR_Write hooked at: 0x601a2760 
[*] Hooks set, continuing process.
Pre-Encrypted: username=test&password=test&remember_me=on 
Pre-Encrypted: username=test&password=test&remember_me=on
Pre-Encrypted: username=jms&password=yeahright!&remember_me=on 
```

Listing 6-1: How cool is that! 我们能看到未加密前的用户名密码

我们已经看到了 soft hook 的轻量级和强大能力。这种方法能被用于所有类型的调试和 逆向过程。在上面的例子中 soft hook 的工作还算正常，如果遇到有性能限制的函数调用时， 进程马上就会变得缓慢，行为异常，还可能崩溃。只是因为，当 INT3 被触发的时候，会将 执行权限交给我们的 hook 代码之后返回。这回花费非常多的事件，如果函数每秒钟执行数 千次。接下来让我们看看如何通过设置 hard hook 和 instrument low-level heap routines 以解决这个问题。

# 6.2 Hard Hooking

## 6.2 Hard Hooking

现在轮到有趣的地方了，hard hooking。这种 hook 很高级，对进程的影响也很小，因为 hook 代码字节写成了 x86 汇编代码。在使用 soft hook 的时候在断点触发的时候有很多事件 发生，接着执行 hook 代码，最后恢复进程。使用 hard hook 的时候，只要在进程内部扩展一 块区域，存放 hook 代码，跳转到此区域执行完成后，返回正常的程序执行流程。优点就是， hard hook 目标进程的时候，进程没有暂停，不像 soft hook。

Immunity 调试器提供了一个简单的对象 FastLogHook 用来创建 hard hook。FastLogHook 在需要 hook 的函数里写入跳转代码，跳到 FastLogHook 申请的一块代码区域，函数内被跳 转代码覆盖的代码就存放在这块新创建的区域。当你构造 fast log hooks 的时候，需要先定 一个 hook 指针，然后定义想要记录的数据指针。程序框架如下：

```py
imm = immlib.Debugger()
fast = immlib.FastLogHook( imm ) 
fast.logFunction( address, num_arguments ) 
fast.logRegister( register ) 
fast.logDirectMemory( address ) 
fast.logBaseDisplacement( register, offset ) 
```

logFunction 接受两个参数，address 就是在希望 hook 的函数内部的某个地址（这个地 址会被跳转指令覆盖）。如果在函数的头部 hook，num_arguments 则设置成想要捕捉到的参 数 的 数 量 ， 如 果 在 函 数 的 结 束 hook ， 则 设 置 成 0 。 数 据 的 记 录 由 logRegister(),logBaseDisplacement(), and logDirectMemory()三个方法完成。

```py
logRegister( register ) 
logBaseDisplacement( register, offset ) logDirectMemory( address ) 
```

logRegister()方法用于跟踪指定的寄存器，比如跟踪函数的返回值（存储在 EAX 中）。 logBaseDisplacement()方法接收 2 个参数，一个寄存器，和一个偏移量；用于从栈中提取参 数或者根据寄存器和偏移量获取。最后一个 logDirectMemory()用于从指定的内存地址获取 数据。

当 hook 触发，log 函数执行之后，他们就将数据存储在一个 FastLogHook 申请的地址。 为了检索 hook 的结果，你必须使用 getAllLog()函数，它会返回一个 Python 列表：

```py
[( hook_address, ( arg1, arg2, argN )), ... ] 
```

所以每次 hook 被触发的时候，触发地址就存在 hook_address 里，所有需要的信息包含 在 第 二 项 中 。 还 有 另 外 一 个 重 要 的 FastLogHook 就 是 STDCALLFastLogHook( 用 于 STDCALL 调用约定)。cdecl 调用约定使用 FastLogHook。

Nicolas Waisman(顶级堆溢出专家)开发了 hippie(利用 hard hook)，可以在 Immunity 中通 过 PyCommand 进行调用。Nico 的解说：

创造 Hippie 的目的是为了创建一个好笑的 log hook，使得处理海量的堆函数调用变成可 能。举个例子：如果你用 Notepad 打开一个文 件 对 话 框 ， 它 需 要 调 用 大 约 4500 次 RtlAllocateHeap 和 RtlFreeHeap 。 如 果 是 Internet Explorer，堆相关的函数调用会有 10 倍甚至更多。

通过 hippie 学习堆的操作，对于将来写基于堆利用的 exploit 相当重要。出于简洁的原 因，我们只使用 hippie 的核心功能创建一个简单的脚本 hippie_easy.py。

在我们开始前，先了解下 RtlAllocateHeap 和 RtlFreeHeap。

```py
BOOLEAN RtlFreeHeap(
    IN PVOID HeapHandle, IN ULONG Flags,
    IN PVOID HeapBase
);
PVOID RtlAllocateHeap(
    IN PVOID HeapHandle, IN ULONG Flags,
    IN SIZE_T Size
); 
```

RtlFreeHeap 和 RtlAllocateHeap 的所有参数都是必须捕捉的，不过 RtlAllocateHeap 返回的新堆的地址也是需要捕捉的。

```py
#hippie_easy.py 
import immlib 
import immutils
# This is Nico's function that looks for the correct
# basic block that has our desired ret instruction
# this is used to find the proper hook point for RtlAllocateHeap 
def getRet(imm, allocaddr, max_opcodes = 300):
    addr = allocaddr
    for a in range(0, max_opcodes):
        op = imm.disasmForward( addr ) 
        if op.isRet():
            if op.getImmConst() == 0xC:
                op = imm.disasmBackward( addr, 3 ) 
                return op.getAddress()
        addr = op.getAddress() 
    return 0x0
# A simple wrapper to just print out the hook
# results in a friendly manner, it simply checks the hook
# address against the stored addresses for RtlAllocateHeap, RtlFreeHeap def showresult(imm, a, rtlallocate):
if a[0] == rtlallocate:
    imm.Log( "RtlAllocateHeap(0x%08x, 0x%08x, 0x%08x) <- 0x%08x %s" % (a[1][0], a[1][1], a[1][2], a[1][3], extra), address = a[1][3] )
    return "done"
else:
    imm.Log( "RtlFreeHeap(0x%08x, 0x%08x, 0x%08x)" % (a[1][0], a[1][1], a[1][2]) )
def main(args):
    imm = immlib.Debugger()
    Name = "hippie"
    fast = imm.getKnowledge( Name ) 
    if fast:
        # We have previously set hooks, so we must want
        # to print the results 
        hook_list = fast.getAllLog()
        rtlallocate, rtlfree = imm.getKnowledge("FuncNames") 
        for a in hook_list:
            ret = showresult( imm, a, rtlallocate )
        return "Logged: %d hook hits." % len(hook_list)
    # We want to stop the debugger before monkeying around 
    imm.Pause()
    rtlfree = imm.getAddress("ntdll.RtlFreeHeap") 
    rtlallocate = imm.getAddress("ntdll.RtlAllocateHeap") 
    module = imm.getModule("ntdll.dll")
    if not module.isAnalysed():
        imm.analyseCode( module.getCodebase() )
    # We search for the correct function exit point 
    rtlallocate = getRet( imm, rtlallocate, 1000 )
    imm.Log("RtlAllocateHeap hook: 0x%08x" % rtlallocate)
    # Store the hook points
    imm.addKnowledge( "FuncNames", ( rtlallocate, rtlfree ) )
    # Now we start building the hook
    fast = immlib.STDCALLFastLogHook( imm )
    # We are trapping RtlAllocateHeap at the end of the function 
    imm.Log("Logging on Alloc 0x%08x" % rtlallocate) 
    fast.logFunction( rtlallocate )
    fast.logBaseDisplacement( "EBP", 8 ) 
    fast.logBaseDisplacement( "EBP", 0xC ) 
    fast.logBaseDisplacement( "EBP", 0x10 ) 
    fast.logRegister( "EAX" )
    # We are trapping RtlFreeHeap at the head of the function 
    imm.Log("Logging on RtlFreeHeap 0x%08x" % rtlfree) 
    fast.logFunction( rtlfree, 3 )
    # Set the hook fast.Hook()
    # Store the hook object so we can retrieve results later 
    imm.addKnowledge(Name, fast, force_add = 1)
    return "Hooks set, press F9 to continue the process." 
```

第一个函数使用 Nico 内建的代码块找到可以在 RtlAllocateHeap 内部设置 hook 的地址。 让我们反汇编 RtlAllocateHeap 函数看看最后几行的指令是怎么样的：

```py
0x7C9106D7 F605 F002FE7F TEST BYTE PTR DS:[7FFE02F0],2
0x7C9106DE 0F85 1FB20200 JNZ ntdll.7C93B903
0x7C9106E4 8BC6 MOV EAX,ESI
0x7C9106E6 E8 17E7FFFF CALL ntdll.7C90EE02
0x7C9106EB C2 0C00 RETN 0C 
```

Python 代码从函数的头部看似反汇编，直到在 0x7C9106EB 找到 RET 指令然后确认整 行指令包含 0x0C。然后往后反汇编 3 行指令到达 0x7C9106D7。这样做只不过是为了确保 有足够的空间写入 5 个字节的 JMP 指令。如果我们在 RET 这行写入 5 个字节的 JMP 指令， 数据就会覆盖出函数的代码范围。那接下来很可能发生恐怖的事情，破坏了代码对齐，进程 会崩溃。这些小函数能帮你解决很多可怕的事情，在二进制面前，任何的差错都会导致灾难。

下一行代码就是简单的判断 hook 是否设置了，如果设置了就从 knowledge base 中获取 必要的目标，然后打印出 hook 信息。脚本第一次运行的时候设置 hook，第二次运行的时候 监视 hook 到的结果，每次运行都获取新的 hook 数据。如果想查询任何存储在 knowledge base 里的目标，重要从调试器的 shell 里访问就行了。

最后一块代码就是构造 hook 和监视点。对于 RtlAllocateHeap 调用获取所有的三个参数 还有返回值，RtlFreeHeap 只要获取三个参数就可以了。只用了不超过 100 行的代码，我们 就成功使用了强大的 hard hook，没用使用任何的编辑器和多余的工具。Very cool！

让用 notepad.exe 做测试，看看是否如 Nico 所说打开一个对话框就会有将近 4500 个堆 调用。在 Immunity 下打开 C:\WINDOWS\System32\notepad.exe 运行!hippie_easy 命令(如果 不懂看 第五章)。恢复进程，在 Notepad 里选择 File-->Open。

现在确认结果。重复运行!hippie_easy，你将会看到调试器日志窗口(ALT-L)的输出。

```py
RtlFreeHeap(0x000a0000, 0x00000000, 0x000ca0b0) 
RtlFreeHeap(0x000a0000, 0x00000000, 0x000ca058) 
RtlFreeHeap(0x000a0000, 0x00000000, 0x000ca020) 
RtlFreeHeap(0x001a0000, 0x00000000, 0x001a3ae8) 
RtlFreeHeap(0x00030000, 0x00000000, 0x00037798)
RtlFreeHeap(0x000a0000, 0x00000000, 0x000c9fe8) 
```

Listing 6-2 由!hippie_easy PyCommand 产生的输出

非常好！我们有了一些结果，如果你看到 Immunity 调试器的状态栏，会看到总共有 4674 次触发。所以 Nico 是对的。你能在任何时候重新运行脚本以便看到新的触发结果和统计数 值。最 cool 的地方是成千上万次的调用都不会降低到进程的执行效率。

hook 将会在你的逆向调试中一次又一次的使用。在这里我们不仅学会了运用强大的 hook 技能，还让这一切自动的进行，这是美好的，这是幸福的，这是伟大的。接下来让我 们学习如何控制一个进程，那会更有趣。