# 四、PyDBG---纯 PYTHON 调试器

# 4 PyDBG---纯 PYTHON 调试器

话说上回我们讲到如何在 windows 下构造一个用户模式的 调试器，最后在大家的不懈努力下，终于历史性的完成了这一伟 大工程。这回，咱们该去取取经了，看看传说中的 PyDbg。传说又是传说，别担心，这 个传说是真的，我用人格担保。PyDbg 出生于 2006 年，出生地 Montreal, Quebec，父亲 Pedram Amini，担当角色：逆向工程框架 PaiMei 的核心组件。现在 PyDbg 已经用于各种各样的工具之中了，其中包括 Taof （非常流行的 fuzzer 代理）ioctlizer（作者开发的一个针对 windwos 驱动的 fuzzer）。如此强大的东西，不用就太可惜了（Python 的好处就是别人有的你也会有）。 首先用它来扩展下断点处理功能。接着干些高级的活：处理程序崩溃，进程快照还有将来 Fuzz 需要用的东西。现在就开工，开工，速度开工！

# 4.1 扩展断点处理

## 4.1 扩展断点处理

在前面的章节中我们讲解了用事件处理函数处理调试事件的方法。用 PyDbg 可以很容 易的扩展这种功能，只需要构建一个用户模式的回调函数。当收到一个调试事件的时候，回 调函数执行我们定义的操作。比如读取特定地址的数据，设置更更多的断点，操作内存。操 作完成后，再将权限交还给调试器，恢复被调试的进程。

PyDbg 设置函数的断点原型如下：

```py
bp_set(address, description="",restore=True,handler=None) 
```

address 是要设置的断点的地址，description 参数可选，用来给每个断点设置唯一的名字。 restore 决定了是否要在断点被触发以后重新设置， handler 指向断点触发时候调用的回调函 数。断点回调函数只接收一个参数，就是 pydbg()类的实例化对象。所有的上下文数据，线 程，进程信息都在回调函数被调用的时候，装填在这个类中。

以 printf_loop.py 为测试目标，让我们实现一个自定义的回调函数。这次我们在 printf() 函数上下断点，以便读取 printf()输出时用到的参数 counter 变量，之后用一个 1 到 100 的随 机数替换这个变量的值，最后再打印出来。记住，我们是在目标进程内处理，拷贝，操作这 些实时的断点信息。这非常的强大！新建一个 printf_random.py 文件，键入下面的代码。

```py
#printf_random.py
from pydbg import *
from pydbg.defines import * 
import struct
import random
# This is our user defined callback function 
def printf_randomizer(dbg):
    # Read in the value of the counter at ESP + 0x8 as a DWORD 
    parameter_addr = dbg.context.Esp + 0x8
    counter = dbg.read_process_memory(parameter_addr,4)
    # When we use read_process_memory, it returns a packed binary
    # string. We must first unpack it before we can use it further. 
    counter = struct.unpack("L",counter)[0]
    print "Counter: %d" % int(counter)
    # Generate a random number and pack it into binary format
    # so that it is written correctly back into the process 
    random_counter = random.randint(1,100) 
    random_counter = struct.pack("L",random_counter)[0]
    # Now swap in our random number and resume the process 
    dbg.write_process_memory(parameter_addr,random_counter)
    return DBG_CONTINUE
# Instantiate the pydbg class 
dbg = pydbg()
# Now enter the PID of the printf_loop.py process 
pid = raw_input("Enter the printf_loop.py PID: ")
# Attach the debugger to that process 
dbg.attach(int(pid))
# Set the breakpoint with the printf_randomizer function
# defined as a callback
printf_address = dbg.func_resolve("msvcrt","printf") 
dbg.bp_set(printf_address,description="printf_address",handler=printf_randomizer)
# Resume the process 
dbg.run() 
```

现在运行 printf_loop.py 和 printf_random.py 两个文件。输出结果将和表 4-1 相似。

Table 4-1:调试器和进程的输出

| Output from Debugger | Output from Debugged Process |
| --- | --- |
| Enter the printf_loop.py PID: 3466 | Loop iteration 0! |
| … | Loop iteration 1! |
| … | Loop iteration 2! |
| … | Loop iteration 3! |
| Counter: 4 | Loop iteration 32! |
| Counter: 5 | Loop iteration 39! |
| Counter: 6 | Loop iteration 86! |
| Counter: 7 | Loop iteration 22! |
| Counter: 8 | Loop iteration 70! |
| Counter: 9 | Loop iteration 95! |
| Counter: 10 | Loop iteration 60! |

为了不把你搞混，让我们看看 printf_loop.py 代码。

```py
from ctypes import * 
import time
msvcrt = cdll.msvcrt 
counter = 0
while 1:
    msvcrt.printf("Loop iteration %d!\n" % counter) 
    time.sleep(2)
    counter += 1 
```

先搞明白一点，printf()接受的这个 counter 是主函数里 counter 的拷贝，就是说在 printf 函数内部,无论怎么修改都不会影响到外面的这个 counter(C 语言所说的只有传递指针才能真 正的改变值)。

你应该看到，调试器在 printf 循环到第 counter 变量为 4 的时候才设置了断点。这是 因 为被 counter 被捕捉到的时候已经为 4 了（这是为了让大家看到对比结果，不要认为调试器 傻了）。同样你会看到 printf_loop.py 的输出结果一直到 3 都是正常的。到 4 的时候，printf() 被中断，内部的 counter 被随即修改为 32!这个例子很简单且强大，它告诉了你在调试事件 发生的时候如何构建回调函数完成自定义的操作。现在让我们看一看 PyDbg 是如何处理应 用程序崩溃的。

# 4.2 处理访问违例

## 4.2 处理访问违例

当程序尝试访问它们没有权限访问的页面的时候或者以一种不合法的方式访问内存的 时候，就会产生访问违例。导致违例错误的范围很广，从内存溢出到不恰当的处理空指针都 有可能。从安全角度考虑，每一个访问违例都应该仔细的审查，因为它们有可能被利用。

当调试器处理访问违例的时候，需要搜集所有和违例相关的信息，栈框架，寄存器，以及引起违例的指令。接着我们就能够用这些信息写一个利用程序或者创建一个二进制的补丁 文件。

PyDbg 能够很方便的实现一个违例访问处理函数，并输出相关的奔溃信息。这次的测试 目标就是危险的 C 函数 strcpy() ，我们用它创建一个会被溢出的程序。接下来我们再写一个 简短的 PyDbg 脚本附加到进程并处理违例。溢出的脚本 buffer_overflow.py，代码如下：

```py
# buffer_overflow.py 
from ctypes import * 
msvcrt = cdll.msvcrt
# Give the debugger time to attach, then hit a button 
raw_input("Once the debugger is attached, press any key.")
# Create the 5-byte destination buffer 
buffer = c_char_p("AAAAA")
# The overflow string 
overflow = "A" * 100
# Run the overflow 
msvcrt.strcpy(buffer, overflow) 
```

问题出在这句 msvcrt.strcpy(buffer, overflow)，接受的应该是一个指针，而传递给函数的是一个变量，函数就会把 overflow 当作指针使用，把里头的值当作地址用（0x41414141414.....）。可惜这个地址是很可能是不能用的。现在我们已经构造了测试案例， 接下来是处理程序了。

```py
# access_violation_handler.py
from pydbg import *
from pydbg.defines import *
# Utility libraries included with PyDbg 
import utils
# This is our access violation handler 
def check_accessv(dbg):
    # We skip first-chance exceptions
    if dbg.dbg.u.Exception.dwFirstChance:
        return DBG_EXCEPTION_NOT_HANDLED
    crash_bin = utils.crash_binning.crash_binning() 
    crash_bin.record_crash(dbg)
    print crash_bin.crash_synopsis() 
    dbg.terminate_process()
    return DBG_EXCEPTION_NOT_HANDLED
pid = raw_input("Enter the Process ID: ") 
dbg = pydbg()
dbg.attach(int(pid)) dbg.set_callback(EXCEPTION_ACCESS_VIOLATION,check_accessv) 
dbg.run() 
```

现在运行 buffer_overflow.py，并记下它的进程号，我们先暂停它等处理完以后再运行。

执行 access_violation_handler.py 文件，输入测试套件的 PID.当调试器附加到进程以后，在测 试套件的终端里按任何键，接下来你应该看到和表 4-1 相似的输出。

```py
python25.dll:1e071cd8 mov ecx,[eax+0x54] from thread 3376 caused access violation when attempting to read from 0x41414195 CONTEXT DUMP
    EIP: 1e071cd8 mov ecx,[eax+0x54] 
    EAX: 41414141 (1094795585) -> N/A
    EBX: 00b055d0 ( 11556304) -> @U`" B`Ox,`O )Xb@|V`"L{O+H]$6 (heap) 
    ECX: 0021fe90 ( 2227856) -> !$4|7|4|@%,\!$H8|!OGGBG)00S\o (stack) 
    EDX: 00a1dc60 ( 10607712) -> V0`w`W (heap)
    EDI: 1e071cd0 ( 503782608) -> N/A
    ESI: 00a84220 ( 11026976) -> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA (heap)
    EBP: 1e1cf448 ( 505214024) -> enable() -> NoneEnable automa (stack) 
    ESP: 0021fe74 ( 2227828) -> 2? BUH` 7|4|@%,\!$H8|!OGGBG) (stack)
    +00: 00000000 ( 0) -> N/A
    +04: 1e063f32 ( 503725874) -> N/A
    +08: 00a84220 ( 11026976) -> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA (heap)
    +0c: 00000000 ( 0) -> N/A
    +10: 00000000 ( 0) -> N/A
    +14: 00b055c0 ( 11556288) -> @F@U`" B`Ox,`O )Xb@|V`"L{O+H]$ (heap)
disasm around:
    0x1e071cc9 int3 
    0x1e071cca int3 
    0x1e071ccb int3 
    0x1e071ccc int3 
    0x1e071ccd int3 
    0x1e071cce int3 
    0x1e071ccf int3 
    0x1e071cd0 push esi
    0x1e071cd1 mov esi,[esp+0x8] 
    0x1e071cd5 mov eax,[esi+0x4] 
    0x1e071cd8 mov ecx,[eax+0x54] 
    0x1e071cdb test ch,0x40 
    0x1e071cde jz 0x1e071cff 
    0x1e071ce0 mov eax,[eax+0xa4] 
    0x1e071ce6 test eax,eax 
    0x1e071ce8 jz 0x1e071cf4 
    0x1e071cea push esi
    0x1e071ceb call eax 
    0x1e071ced add esp,0x4 
    0x1e071cf0 test eax,eax 
    0x1e071cf2 jz 0x1e071cff
SEH unwind:
    0021ffe0 -> python.exe:1d00136c jmp [0x1d002040] 
    ffffffff -> kernel32.dll:7c839aa8 push ebp 
```

Listing 4-1：PyDbg 捕捉到的奔溃信息

输出了很多有用的信息片断。第一个部分指出了那个指令引发了访问异常以及指令在哪 个块里。这个信息可以帮助 你写出漏洞利用程序或者用静态分析工具分析问题出在哪里。 第二部分转储出了所有寄存器的值，特别有趣的是，我们将 EAX 覆盖成了 0x41414141（0x41 是大写 A 的的十六进制表示）。同样，我们看到 ESI 指向了一个由 A 组成的字符串。和 ESP+08 指向同一个地方。第三部分是在故障指令附近代码的反汇编指令。最后一块是奔溃发生时候 注册的结构化异常处理程序的列表。

用 PyDbg 构建一个奔溃处理程序就是这么简单。不仅能够自动化的处理崩溃，还能在 在事后剖析进程发生的一切。下节，我们用 PyDbg 的进程内部快照功能创建一个进 程 rewinder。

# 4.3 进程快照

## 4.3 进程快照

PyDbg 提供了一个非常酷的功能，进程快照。使用进程快照的时候，我们就能够冰冻进 程，获取进程的内存数据。以后我们想要让进程回到这个时刻的状态，只要使用这个时刻的 快照就行了。

### 4.3.1 获得进程快照

第一步,在一个准确的时间获得一份目标进程的精确快照。为了使得快照足够精确，需 要得到所有线程以及 CPU 上下文，还有进程的整个内存。将这些数据存储起来，下次我们 需要恢复快照的时候就能用的到。

为了防止在获取快照的时候，进程的数据或者状态被修改，需要将进程挂起来，这个任 务由 suspend_all_threads()完成。挂起进程之后，可以用 process_snapshot()获取快照。快照完 成之后，用 resume_all_threads()恢复挂起的进程，让程序继续执行。当某个时刻我们需要将 进程恢复到从前的状态，简单的 process_restore()就行了。这看起来是不是太简单了？

现在新建个 snapshot.py 试验下，代码的功能就是我们输入"snap"的时候创建一个快照， 输入"restore"的时候将进程恢复到快照时的状态。

```py
#snapshot.py
from pydbg import *
from pydbg.defines import * 
import threading
import time 
import sys
class snapshotter(object):
    def init (self,exe_path):
        self.exe_path = exe_path
        self.pid = None
        self.dbg = None
        self.running = True
        # Start the debugger thread, and loop until it sets the PID
        # of our target process
        pydbg_thread = threading.Thread(target=self.start_debugger) pydbg_thread.setDaemon(0)
        pydbg_thread.start()
        while self.pid == None: 
            time.sleep(1)
        # We now have a PID and the target is running; let's get a
        # second thread running to do the snapshots
        monitor_thread = threading.Thread(target=self.monitor_debugger) monitor_thread.setDaemon(0)
        monitor_thread.start() 
    def monitor_debugger(self):
        while self.running == True:
            input = raw_input("Enter: 'snap','restore' or 'quit'") 
            input = input.lower().strip()
            if input == "quit":
                print "[*] Exiting the snapshotter." 
                self.running = False self.dbg.terminate_process()
            elif input == "snap":
                print "[*] Suspending all threads." self.dbg.suspend_all_threads()
                print "[*] Obtaining snapshot." 
                self.dbg.process_snapshot()
                print "[*] Resuming operation." 
                self.dbg.resume_all_threads()
            elif input == "restore":
                print "[*] Suspending all threads." self.dbg.suspend_all_threads()
                print "[*] Restoring snapshot." 
                self.dbg.process_restore()
                print "[*] Resuming operation." 
                self.dbg.resume_all_threads()
    def start_debugger(self): self.dbg = pydbg()
        pid = self.dbg.load(self.exe_path) 
        self.pid = self.dbg.pid
        self.dbg.run() 
        exe_path = "C:\\WINDOWS\\System32\\calc.exe" 
snapshotter(exe_path) 
```

那么第一步就是在调试器内部创建一个新线程，并用此启动目标进程。通过使用分开的线程，就能将被调试的进程和调试器的操作分开，这样我们输入不同的快照命令进行操作的 时候，就不用强迫被调试进程暂停。当创建新线程的代码返回了有效的 PID，我们就创建另 一个线程，接受我们输入的调试命令。之后这个线程根据我们输入的命令决定不同的操作（快 照，恢复快照，结束程序）。

我们之所以选择计算器作为例子，是因为通过操作图形界面 ，可以更清晰的看到，快 照的作用。先在计算器里输入一些数据，然后在终端里输入"snap"进行快照,之后再在计算器 里进行别的操作。最后就当的输入"restore"，你将看到，计算器回到了最初时快照的状态。 使用这种方法我们能够将进程恢复到任意我们希望的状态。

现在让我们将所有的新学的 PyDbg 知识，创建一个 fuzz 辅助工具，帮助我们找到软件 的漏洞，并自动处理奔溃事件。

### 4.3.2 组合代码

我们已经介绍了一些 PyDbg 非常有用的功能，接下来要构建一个工具用来根除应用程 序中出现的可利用的漏洞。在我们平常的开发过程中，有些函数是非常危险的，很容易造成 缓冲区溢出，字符串问题，以及内存出错，对这些函数需要重点关注。

工具将定位于危险函数，并跟踪它们的调用。当我们认为函数被危险调用了，就将 4 堆栈中的 4 个参数接触引用，弹出栈，并且在函数产生溢出之前对进程快照。如果这次访问 违例了，我们的脚本将把进程恢复到，函数被调用之前的快照。并从这开始，单步执行，同 时 反 汇 编 每 个 执 行 的 代 码 ， 直 到 我 们 也 抛 出 了 访 问 违 例 ， 或 者 执 行 完 了 MAX_INSTRUCTIONS（我们要监视的代码数量）。无论什么时候当你看到一个危险的函数 在处理你输入的数据的时候，尝试操作数据 crash 数据都似乎值得。这是创造出我们的漏洞 利用程序的第一步。

开动代码，建立 danger_track.py，输入下面的代码。

```py
#danger_track.py
from pydbg import *
from pydbg.defines import * 
import utils
# This is the maximum number of instructions we will log
# after an access violation MAX_INSTRUCTIONS = 10
# This is far from an exhaustive list; add more for bonus points dangerous_functions = {
    "strcpy" : "msvcrt.dll",
    "strncpy" : "msvcrt.dll",
    "sprintf" : "msvcrt.dll", "vsprintf": "msvcrt.dll"
}
dangerous_functions_resolved = {} 
crash_encountered = False
instruction_count = 0 
def danger_handler(dbg):
    # We want to print out the contents of the stack; that's about it
    # Generally there are only going to be a few parameters, so we will
    # take everything from ESP to ESP+20, which should give us enough
    # information to determine if we own any of the data esp_offset = 0
    print "[*] Hit %s" % dangerous_functions_resolved[dbg.context.Eip]
    print "================================================================="
    while esp_offset <= 20:
        parameter = dbg.smart_dereference(dbg.context.Esp + esp_offset)
        print "[ESP + %d] => %s" % (esp_offset, parameter)
        esp_offset += 4
    print "=================================================================\n
    dbg.suspend_all_threads() 
    dbg.process_snapshot() 
    dbg.resume_all_threads()
    return DBG_CONTINUE
def access_violation_handler(dbg): 
    global crash_encountered
    # Something bad happened, which means something good happened :)
    # Let's handle the access violation and then restore the process
    # back to the last dangerous function that was called 
    if dbg.dbg.u.Exception.dwFirstChance:
        return DBG_EXCEPTION_NOT_HANDLED
    crash_bin = utils.crash_binning.crash_binning() 
    crash_bin.record_crash(dbg)
    print crash_bin.crash_synopsis()
    if crash_encountered == False: 
        dbg.suspend_all_threads() 
        dbg.process_restore() 
        crash_encountered = True
    # We flag each thread to single step
        for thread_id in dbg.enumerate_threads():
            print "[*] Setting single step for thread: 0x%08x" % thread_id 
        h_thread = dbg.open_thread(thread_id)
        dbg.single_step(True, h_thread) 
        dbg.close_handle(h_thread)
    # Now resume execution, which will pass control to our
    # single step handler 
    dbg.resume_all_threads() 
    return DBG_CONTINUE
    else:
        dbg.terminate_process()
    return DBG_EXCEPTION_NOT_HANDLED
def single_step_handler(dbg): 
    global instruction_count 
    global crash_encountered
    if crash_encountered:
        if instruction_count == MAX_INSTRUCTIONS: 
            dbg.single_step(False)
            return DBG_CONTINUE
        else:
            # Disassemble this instruction
            instruction = dbg.disasm(dbg.context.Eip)
            print "#%d\t0x%08x : %s" % (instruction_count,dbg.context.Eip, instruction)
            instruction_count += 1 
            dbg.single_step(True)
            return DBG_CONTINUE
dbg = pydbg()
pid = int(raw_input("Enter the PID you wish to monitor: "))
dbg.attach(pid)
# Track down all of the dangerous functions and set breakpoints 
for func in dangerous_functions.keys():
    func_address = dbg.func_resolve( dangerous_functions[func],func )
    print "[*] Resolved breakpoint: %s -> 0x%08x" % ( func, func_address ) 
    dbg.bp_set( func_address, handler = danger_handler ) 
    dangerous_functions_resolved[func_address] = func
dbg.set_callback( EXCEPTION_ACCESS_VIOLATION, access_violation_handler ) 
dbg.set_callback( EXCEPTION_SINGLE_STEP, single_step_handler )
dbg.run() 
```

通过之前对 PyDbg 的诸多讲解，这段代码应该看起来不那么难了吧。测试这个脚本的 最好方法，就是运行一个有漏洞价格的程序，然后让脚本附加到进程，和程序交互，尝试 crash 程序。

我们已经对 PyDbg 有了一定的了解，不过这只是它强大功能的一部分，还有更多的东 西，需要你自己去挖掘。再好的东西也满足不了那些"懒惰"的 hacker。PyDbg 固然强大，方 便的扩展，自动化调试。不过每次要完成任务的时候，都要自己动手编写代码。接下来介绍 的 Immunity Debugger 弥补了这点，完美的结合了图形化调试和脚本调试。它能让你更懒， 哈。让我们继续。