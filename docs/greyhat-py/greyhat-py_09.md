# 八、FUZZING

# 8 FUZZING

Fuzzing 一直以来都是个热点话题，因为使用它能非常高 效的寻找出软件的漏洞。简单的说，Fuzzing 就是向目标程序发 送畸形或者半畸形的数据以引发错误。这一章，让我们先了解几 个不同类型的 fuzzer 还有 bug，之后我们还要自己动手写实现一 个 file fuzzer。下一章，会详细的介绍 Sulley fuzzing 框架和如何设计一个针对 Windows 驱动的 fuzzer。

fuzzers 基本上分成 2 大类：generation（产生） 和 mutation（变异）。Generation fuzzers 创建数据，然后发送到到目标程序， mutation fuzzers 并不创建数据，而是截获程序接收的 数据，然后修改数据。 举个例子，当我们要 fuzz 一个 web 服务器的时候，generation fuzzer 会生成一套变形的 Http 请求然后发送给 web 服务器，而 mutation fuzzer 会捕获 Http 请求， 在请求传递给 web 服务器前修改它们。

为了将来我们创建创建一个高效的 fuzzer，我们需要先对不同类型的 bug 做一个简单的 了解，并且看看 fuzzer 如何触发它们。如果要更详细的了解软件安全检测，可以看下面的书。

# 8.1 Bug 的分类

## 8.1 Bug 的分类

当 hacker 或者逆向工程师分析软件漏洞的时候，都会设法找到能控制程序执行的 bug。 Fuzzer 就提供了一种自动化的方式帮助我们找出 bug，然后获得系统的控制权，提升权限， 并且偷取程序访问的信息，不论目标程序是在系统独立运行的进程，还是只运行脚本的网络 程序。在这里我们关注的是独立运行的进程，以及其泄漏的信息。

### 8.1.1 缓冲区溢出

缓冲区溢出是最常见的软件漏洞。所有的正常的内存管理函数，字符处理代码甚至编程语言本身都可能产生缓冲区溢出漏洞，致使软件出错。

简而言之，缓冲区溢出就是由于，把个过多的数据存储在一个过小的内存空间里，所引发的软件错误。对于其原理的可以用个很形象的比喻来说明。如果一个水桶，只能装一加仑 的水，我们只导入几滴水或者半加仑的水，甚至是一加仑，把水桶填满了，都不会出问题， 一切都正常如初。如果我们导入两加仑的水，那水就会溢出到地板上，到时候，你就得收拾 这堆烂摊子了。用在软件上也是一样的，如果太多的水（数据），倒入到一个桶（buffer）内， 水就会溢出到作为的地表（memory）上。当一个攻击者能够控制多余的数据对内存区域的 覆盖，就拿那个得到代码的执行权限，进一步获取到系统信息或者做别的事。有两种主要的 缓冲区溢出：基于栈的和基于堆的。两种溢出的表现不同，但产生的结果相同：攻击者最终 控制代码的执行。

栈溢出的特点就是通过溢出覆盖栈，来控制程序的执行流程：比如改变函数的指针，变量的值，异常处理程序，或者覆盖函数的返回地址，可以得到代码的执行权限。栈溢出的时 候，会抛出一个访问违例；这样我们就能够在 fuzzing 的时候非常方便的跟踪到它们。

应用程序在运行的时候会动态的申请一块内存区域，这些区域就是堆。堆是一块一块连在一起的，负责存储元数据。当攻击者将数据覆盖到自己申请的堆以外的别的堆的时候，堆 溢出就发生了。接着攻击者能通过覆盖数据改变任何存储在堆上的数据：变量，函数指针， 安全令牌，以及各种重要的数据。堆溢出很难被立即的跟踪到，因为被影响的内存快，一般 不会被程序立即的访问，需要等到程序结束之前的某个时间内才有可能访问到，当然也有可 能一直不访问。在 fuzzing 的时候我们就必须一直等到一个访问违例产生了才会知道，这个 堆溢出是否在成功了。

MICROSOFT GLOBAL FLAGS

这项技术是专门为软件开发者(or exploit writer)专门设计的。Gflags(Global flags 全局标 志)是一系列的诊断和调试设置，能够让你非常精确的跟踪，记录，和调试软件。在 2000， xp 和 2003 上都能够使用这项技术。

这项技术最有趣的地方在于堆页面的校对。当我们在一个进程上打开这个选项的时候，校对器会动态的更重内存的操作，包括内存的申请和释放。不过真正令人高兴的特点是，它 能够在堆溢出发生的时候立刻产生一个调试中断，这样调试器就会停在产生错误的指令上。 这样在下面的调试中遇到了堆相关的 bug 的时候我们就能方便的查找到源头在哪。

我们能够使用 gflags.exe 来编辑 Gflags 标志帮助我们跟踪堆溢出。 [`www.microsoft.com/downloads/details.aspx?FamilyId=49AE8576-9BB9-4126-9761-BA80`](http://www.microsoft.com/downloads/details.aspx?FamilyId=49AE8576-9BB9-4126-9761-BA80) 这个软件是 Microsoft "免费"提供的，请"放心"安装。

Immunity 也提供了一个 Gflags 库，并且将它和 PyCommand 结合在了一起。更多的信 息访问 [`debugger.immunityinc.com/`](http://debugger.immunityinc.com/).

为了通过 fuzzing 溢出目标程序，我们会简单的传递给程序大量的数据，然后跟踪程序， 找出利用了我们传入数据的代码，然后祈祷它没有合格验证数据长度的,my god!

接下来看看另一个常见的应用程序漏洞，Integer Overflows 整数溢出。

### 8.1.2 Integer Overflows

整数溢出是一种非常有趣的漏洞，包括程序如何处理（编译器标准的）有符号整数和 exploit 如何利用这个整数。一个有符号整数，由 2 个字节组成，表示的范围从 -32767 到 32767。当我们从尝试向存储一个整数的地方写入超过其大小的数字的时候，整数溢出就触 发了。因为存如的数字太大，处理器会自动的将高位多出来的字节丢弃。初看起来，好像没 什么值得利用的。下面让我们看一个设计好的例子：

```py
MOV EAX, [ESP + 0x8] 
LEA EDI, [EAX + 0x24] 
PUSH EDI
CALL msvcrt.malloc 
```

第一条指令将栈内的数据[esp+0x8]传给 EAX，第二条指令，将 EAX 加上 0x24 这个地 址存储在 EDI 中。之后我们将这个唯一的参数(申请的内存的大小)传入函数 malloc。一切看 起来都很正常，真的是这样吗？假设在栈中的数据是一个有符号整数，而且非常大，几乎接 近了有符号整数的最大值（ 32767），然后传递给 EAX，EAX 加上 0x24,整数溢出，最后我 们得到一个非常小的值。看一看表 8-1，看看这一切是如何发生的，假定在堆上的参数是我 们能够控制的，我们给它设置成一个非常大的值 0xFFFFFFF5。

```py
Stack Parameter => 0xFFFFFFF5 
Arithmetic Operation => 0xFFFFFFF5 + 0x24
Arithmetic Result => 0x100000019 (larger than 32 bits) 
Processor Truncates => 0x00000019 
```

Listing 8-1:在控制下的整数操作

如何一切顺利，malloc 将只申请 0x19 个字节大小的空间，这块内存比程序本身要申请 的空间小很多。如果程序将一大块的数据写入这块区域，缓冲区溢出就发生了。在 fuzzing 的时候，我们得从整形最大值和最小值两个方面入手，测试执行溢出，接下来就是设法进一 步控制溢出，使溢出变得更完美。

下面让我们快速的看一看另一种常见漏洞，格式化字符串漏洞 Format String Attacks。

### 8.1.3 Format String Attacks

格式化字符串攻击，顾名思义，攻击者通过将设计好的字符串传入特定字符串格式化函 数，使其产生溢出，列如 C 语言的 printf。让我们先看看 printf 的原型:

```py
int printf( const char * format, ... ); 
```

第一个参数是一个完整需要被格式化的字符串，我们可以附加额外的参数，表示数据将 以什么形式被输出。举个例子：

```py
int test = 10000;
printf("We have written %d lines of code so far.", test); 
Output:
We have written 10000 lines of code so far. 
```

%d 是一个模式说明符,格式指定符指定了特定的输出格式(变量 test 以数字的形式输 出)。如果一个程序员不小心睡着了（压榨 严重的压榨），写出了下面的代码：

```py
char* test = "%x"; 
printf(test); 
Output: 5a88c3188 
```

这看起来和上面的很不同。我们传递了一个模式说明符给 printf 函数，但是没有传递需 要打印的变量。printf 会分析我们传递给它的参数，并且假设栈中的下一个参数就是需要打 印的参数，但是其实这个是毫无效果的一个数据。在这个例子中是 0x5a88c3188，也许是存 在栈上的数据，也有可能跟是一个指向内存的指针。有两个指示符很有趣，一个是 %s，另 一个是 %n。 %s 指示符告诉字符串函数，把内存当作字符串来扫描，直到遇到一个 NULL 字符，代表字符串结束了。这对于读取一大块连续的数据或者读取特定地址的数据都十分有 用，当然你也可以用它来 crash 程序。%n 指示符（惟一一个）允许向内存写入内存，而不 仅仅是格式化字符串。这就允许，攻击者覆盖函数的返回地址，或者改写一个以存在的函数 指针，以获得代码的执行权限。在 fuzzing 的似乎后，我们只要在测试用例中加入这些特定 格式说明符，然后传递给一个被错误使用了的字符串处理函数。

现在我们已经对不同的 bug 类型有了个个大概的了解，是时候开始创造第一个 fuzzer 了。接下来我们会简单的实现一个 file fuzzer，它先将正常的文件变形之后拿去给程序处理。 这次继续使用我们久违的老朋友 PyDbg。Come on!!!

# 8.2 File Fuzzer

## 8.2 File Fuzzer

File format vulnerabilitie 文件格式化漏洞已经渐渐的成为了客户端攻击的流行方式，而 我们最感兴趣的就是找出文件格式化分析时出现的漏洞。无论面对的目标是杀毒软件还是文 档阅读器，我们都希望测试库尽可能的全，最好是包含说有的文件格式。同时还要确保，我 们的 fuzzer 能准确的捕捉到崩溃信息，然后自动化的决策出是否是可利用的漏洞。最后还要 加入 emailing 的功能，在我们有成千上万的测试案例的时候，你不会想傻傻的做在机器前看 数据流吧！

现在开始写代码，第一步，构造创建一个类框架，用于简单的文件选择。

```py
#file_fuzzer.py
from pydbg import *
from pydbg.defines 
import * import utils
import random 
import sys 
import struct 
import threading 
import os
import shutil 
import time 
import getopt 
class file_fuzzer:
    def init (self, exe_path, ext, notify): 
        self.exe_path = exe_path
        self.ext = ext
        self.notify_crash = notify 
        self.orig_file = None 
        self.mutated_file = None 
        self.iteration = 0
        self.exe_path = exe_path
        self.orig_file = None 
        self.mutated_file = None
        self.iteration = 0
        self.crash = None 
        self.send_notify = False 
        self.pid = None
        self.in_accessv_handler = False 
        self.dbg = None
        self.running = False
        self.ready = False
        # Optional
        self.smtpserver = 'mail.nostarch.com' 
        self.recipients = ['jms@bughunter.ca',] 
        self.sender = 'jms@bughunter.ca'
        self.test_cases = [ "%s%n%s%n%s%n", "\xff", "\x00", "A" ] 
    def file_picker( self ):
        file_list = os.listdir("examples/") 
        list_length = len(file_list)
        file = file_list[random.randint(0, list_length-1)] shutil.copy("examples\\%s" % file,"test.%s" % self.ext) 
        return file 
```

类框架定义了一些全局变量，用于跟踪记录文件的基础信息，这些文件将会在变形后加 入测试例。file_picker 函数使用内建的 Python 函数列出目录内的所有文件，然后随机选取一 个进行变形。

接下来我们要做一些线程方面的工作：加载 目标程序，跟踪崩溃信息，在文档分析完成之后终止目标程序。第一步，将目标程序加载进 一个调试线程，并且安装自定义的访问违例处理代码。第二步，创建第二个线程，用于监视 调试的线程，并且负责在一段长度的时间之后杀死调试线程。最后还得附加一段 email 提醒 的代码。

```py
#file_fuzzer.py
...
def fuzz( self ):
    while 1:
        if not self.running: #(1)
            # We first snag a file for mutation 
            self.test_file = self.file_picker() 
            self.mutate_file()
            # Start up the debugger thread 
            pydbg_thread = threading.Thread(target=self.start_debugger)
            pydbg_thread.setDaemon(0)
            pydbg_thread.start() 
            while self.pid == None:
                time.sleep(1)
            # Start up the monitoring thread 
            monitor_thread = threading.Thread (target=self.monitor_debugger) 
            monitor_thread.setDaemon(0) 
            monitor_thread.start()
        else:
            self.iteration += 1
            time.sleep(1)
# Our primary debugger thread that the application
# runs under
def start_debugger(self):
    print "[*] Starting debugger for iteration: %d" % self.iteration 
    self.running = True
    self.dbg = pydbg() 
    self.dbg.set_callback(EXCEPTION_ACCESS_VIOLATION,self.check_accessv) 
    pid = self.dbg.load(self.exe_path,"test.%s" % self.ext)
    self.pid = self.dbg.pid 
    self.dbg.run()
# Our access violation handler that traps the crash
# information and stores it 
def check_accessv(self,dbg):
    if dbg.dbg.u.Exception.dwFirstChance: 
        return DBG_CONTINUE
    print "[*] Woot! Handling an access violation!" 
    self.in_accessv_handler = True
    crash_bin = utils.crash_binning.crash_binning() 
    crash_bin.record_crash(dbg)
    self.crash = crash_bin.crash_synopsis()
    # Write out the crash informations
    crash_fd = open("crashes\\crash-%d" % self.iteration,"w") 
    crash_fd.write(self.crash)
    # Now back up the files
    shutil.copy("test.%s" % self.ext,"crashes\\%d.%s" % (self.iteration,self.ext))
    shutil.copy("examples\\%s" % self.test_file,"crashes\\%d_orig.%s" % (self.iteration,self.ext))
    self.dbg.terminate_process() self.in_accessv_handler = False 
    self.running = False
    return DBG_EXCEPTION_NOT_HANDLED
# This is our monitoring function that allows the application
# to run for a few seconds and then it terminates it 
def monitor_debugger(self):
    counter = 0
    print "[*] Monitor thread for pid: %d waiting." % self.pid, 
    while counter < 3:
        time.sleep(1) 
        print counter, 
        counter += 1
    if self.in_accessv_handler != True: 
        time.sleep(1) 
        self.dbg.terminate_process() 
        self.pid = None
        self.running = False
    else:
        print "[*] The access violation handler is doing its business. Waiting."
        while self.running: 
            time.sleep(1)
# Our emailing routine to ship out crash information 
def notify(self):
    crash_message = "From:%s\r\n\r\nTo:\r\n\r\nIteration: %d\n\nOutput:\n\n %s" % (self.sender, self.iteration, self.crash)
    session = smtplib.SMTP(smtpserver) 
    session.sendmail(sender, recipients, crash_message) 
    session.quit()
    return 
```

我们已经有了个比较完整的流程，能够顺利的完成 fuzz 了，让我们简单的看看各个函 数的作用。第一步，通过 self.running 确保当前只有一个调试线程在执行或者访问违例的处 理程序没有在搜集崩溃数据。第二步，我们把随即选择到文件，传入变形函数，这个函数会 在稍后实现。

一旦文件变形完成，第三步，我们就创建一个调试线程，启动目标程序，并将上面随即选中的文件的路径名字，作为命令行参数传入。接着一个条件循环，等待目标进程的创建。 当程序创建成功的时候，得到新的 PID，第四步，创建一个监视进程，确保在一段事件以后 杀死调试的程序。监视线程创建成功以后，我们就增加统计标志，然后加入主循环，等待一 次 fuzz 的完成，继续下一次 fuzz。现在让我们增加一个简单的变形函数。

```py
#file_fuzzer.py
...
def mutate_file( self ):
    # Pull the contents of the file into a buffer 
    fd = open("test.%s" % self.ext, "rb") 
    stream = fd.read()
    fd.close()
    # The fuzzing meat and potatoes, really simple
    # Take a random test case and apply it to a random position
    # in the file
    test_case = self.test_cases[random.randint(0,len(self.test_cases)-1)] 
    stream_length = len(stream)
    rand_offset = random.randint(0, stream_length - 1 ) 
    rand_len = random.randint(1, 1000)
    # Now take the test case and repeat it 
    test_case = test_case * rand_len
    # Apply it to the buffer, we are just
    # splicing in our fuzz data 
    fuzz_file = stream[0:rand_offset]
    fuzz_file += str(test_case) 
    fuzz_file += stream[rand_offset:]
    # Write out the file
    fd = open("test.%s" % self.ext, "wb") 
    fd.write( fuzz_file )
    fd.close() 
    return 
```

这是一个基础的变形函数。我们从全部测试用例中随即的选取一个；然后同样随即的获取一个文件位移和需要附加的 fuzz 数据的长度。用位移和长度信息生成附加的 fuzz 数据， 最后将原始数据分片，在其中加入 fuzz 数据。一切完成后，把新生成的文件覆盖原来的文 件。紧接着就是调试线程开始新一轮的测试了。现在让我们实现命令行处理部分。

```py
#file_fuzzer.py
...
def print_usage(): 
    print "[*]"
    print "[*] file_fuzzer.py -e <Executable Path> -x <File Extension>" 
    print "[*]"
    sys.exit(0)
if name == " main ":
print "[*] Generic File Fuzzer."
# This is the path to the document parser
# and the filename extension to use 
try:
    opts, argo = getopt.getopt(sys.argv[1:],"e:x:n") 
except getopt.GetoptError:
    print_usage() 
exe_path = None 
ext = None
notify = False
for o,a in opts:
if o == "-e":
    exe_path = a 
elif o == "-x":
    ext = a 
elif o == "-n":
    notify = True
if exe_path is not None and ext is not None: 
    fuzzer = file_fuzzer( exe_path, ext, notify ) 
    fuzzer.fuzz()
else:
    print_usage() 
```

现在我们的 file_fuzzer.py 脚本已经能够接收到命令行参数了。-e 标志指示需要 fuzz 的 目标程序的路径。-x 选项是我们需要用于测试的文件的扩展名；举个例子.txt 就说明我们要 用文本文件作为测试数据。-n 选项告诉 fuzzer 是否要接收通知。

最好的测试 fuzzer 的方法，就是在测试目标程序的时候观察数据的变形结果 。在 fuzz 文本文件的时候，用 Windows 记事本是再好不过的了。因为你能够直接的看到每一次的数 据的变化，比用十六进制编辑器和二进制对比工具方便很多。在启动 file_fuzzer.py 脚本之 前，需要在脚本当前目录下新建两个目录 examples 和 crashes 。然后在 examples 目录下存 放几个以.txt 结尾的文件，接着使用如下命令启动脚本。

```py
python file_fuzzer.py -e C:\\WINDOWS\\system32\\notepad.exe -x .txt 
```

随着记事本的启动，你能看到被变形过的文件。在对变形之后的数据满意以后，你就可以使用这个 file fuzzer 测试别的程序了。

# 8.3 改进你的 Fuzzer

## 8.3 改进你的 Fuzzer

虽然我们已经创建了一个 fuzzer，而且只要能够给它提供足够多的时间，它就能找出一 些 bug。但是在通往强大的路还很长很长。

### 8.3.1 Code Coverage

Code coverage 是一个度量，通过统计测试目标程序的过程中，执行了函数。Fuzzing 专家 Charlie Miller 通过经验证明，寻找到的 bug 数量和 Code coverage 的增长成正比。那 我们怎么证明呢！最简单的方法就是，在你 fuzz 目标程序的时候，使用调试器在目标进程 上的说有函数上设置断点，然后使用不同的测试案例去 fuzz 目标进程，根据找到 bug 和击 中的函数数量，你就会知道自己的 fuzz 的效率。还有更多的使用 Code coverage 的复杂的例 子，你可以将它们的技术加入你的 file fuzzer。

### 8.3.2 Automated Static Analysis

通 过 对 二 进 制 文 件 进 行 Automated Static Analysis( 自 动 化 的 静 态 分 析 ) ， 能 够 帮 助 bughunter 更高效的找出目标代码的弱点。跟踪容易出错的函数（例如 strcpy），并且监视函 数的执行过程，会有很好的效果。还有很多别的优点，比如跟踪内部的内存拷贝操作，忽略 不必要的错误处理代码，等等。对目标将程序了解的越多，找出 bug 的机会就越大。

将这些功能加入我们创建的 fuzzer，会很大的提高我们今后的工作效率。在我们设计 fuzzer 的时候扩展性是非常重要的，在以后不断的功能扩张中，你会感谢今天花在前端设计 上的时间，是多么的值得。接下来让我们看看一个基于 Python 的 fuzzing 框架（Pedram Amini, Aaron Portnoy of TippingPoint）。之后我们会深入介绍我的一个 fuzzer 作品 ioctlizer，用于 查找使用了 I/O 控制代码的 Windows 驱动中的漏洞。