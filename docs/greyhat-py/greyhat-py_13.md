# 十二、PyEmu

# 12 PyEmu

PyEmu 由 Cody Pierce(TippingPoint DVLabs team) 于 2007 在黑帽大会上首次公布。PyEmu 是一个存 Python 实现的 IA32 仿真器，用于仿真 CPU 的各种行为以完成不同的任务。仿 真器非常有用，比如在调试病毒的时候，我们就不用真正的运行 它，而是通过仿真器欺骗它在我们的模拟环境中运行。 PyEmu 里 有 三 个 类 :IDAPyEmu, PyDbgPyEmu 和 PEPyEmu 。 IDAPyEmu 用于在 IDA Pro 内完成各种仿真任务（由 DAPython 调用， 详看第十一章），PyDbgPyEmu 类用于动态分析，同时它允许使用我们真正的内存和寄存器。

PEPyEmu 类是一个独立的静态分析库，不需要 IDA 就能完成反汇编任务。我们主要介 绍

IDAPyEmu 和 PEPyEm，剩下的 PyDbgPyEmu 留给大家自己去试验。下面先从 PyEmu 的安 装开始，接着深入介绍仿真器的架构，为实际应用做好准备。

# 12.1 安装 PyEmu

## 12.1 安装 PyEmu

从 [`www.nostarch.com/ghpython.htm`](http://www.nostarch.com/ghpython.htm) 下载作者打包好的文件，如果没有的同学去 google code 上下。

文件下载好后，解压到 C:\PyEmu。每次创建 PyEmu 脚本的时候，都要加入以下两行 Python 代码：

```py
sys.path.append("C:\PyEmu\") 
sys.path.append("C:\PyEmu\lib") 
```

接下来让我们输入了解下 PyEmu 的系统架构，方便后面的脚本编写。

# 12.2 PyEmu 一览

## 12.2 PyEmu 一览

PyEmu 被划分成三个重要的系统：PyCPU，PyMemory 和 PyEmu。与我们交互最多的 就是 PyEmu 类，它再和 PyCPU 和 PyMemoey 交互完成底层的仿真工作。当我们测试驱动 PyEmu 执行一个指令的时候，它就调用 PyCPU 完成真正的指令操作。PyCPU 在进行指令操 作的时候，把需要的内存操作告诉 PyEmu，由 PyEmu 继续调用 PyMemory 辅助完成整个指 令的操作，最后由 PyEmu 将指令的结果返回给调用者。

接下来，让我们简短的了解下各个子系统和他们的使用方法，以便更好的明白伟大的 PyEmu 替我们完成了什么，同时大家也能对实际应用有个初略的了解。

### 12.2.1 PyCPU

PyCPU 类是 PyEmu 的核心，它模拟成和真实的 CPU 一样。在仿真的过程中，它负责 执行指令。当 PyCPU 处理一个指令的时候，会先检索指令指针 ( 由负责静态分析的 IDA Pro/PEPyEmu 或者负责动态调试的 PyDbg 获取)，然后将指令传递给 pydasm，由后者解码成 操作码和操作对象。PyCPU 提供的独立解码指令的能力使得 PyEmu 的跨平台变成了可能。 每个 PyEmu 接收到的指令，都有一个相对应内部函数。举个例子，如果将指令 CMP EAX ,1 传给 PyCPU，接着 PyCPU 就会调用 PyCPU CMP()函数执行真正的操作,并从内存中 检索必要的值，之后设置 CPU 的标志位，告诉程序这次比较的结果。有兴趣的各位都可以 看看 PyCPU.py，所有的 PyEmu 支持的指令处理函数都在这里，通过研究它们可以明白 CPU 是如何完成那些神秘的底层操作的。别担心代码的可读性， Cody 在这上面可没少花功夫。

### 12.2.2 PyMemory

PyMemor 负责加载和储存执行指令的必要数据。同时也可以对可执行程序的代码和数 据块进行映射，以便在仿真器中访问。在将借完两个主要类之后，让我们看看核心类 PyEmu，以及相关的类方法。

### 12.2.3 PyEmu

PyEmu 负责驱动整个仿真器的运作。PyEmu 类本身被设计的非常轻便和灵活，使得开 发者能够很块的开发出强大的仿真器脚本，而不用关心底层操作。这一切都由 PyEmu 提供的帮助函数实现，使用它们能让我们的这个逆向工作变得更简单，无论是操作执行流程，改变寄存器值还是更新内存等等。下面就来卓一介绍它们。

### 12.2.4 执行操作

PyEmu 的执行过程由一个函数控制，execute()。原型如下：

```py
execute( steps=1, start=0x0, end=0x0 ) 
```

总共三个参数，如果一个都没有提供，就从 PyEmu 当前的地址开始执行。这个地址也 许是 PyDbg 的 EIP 寄存器指向的位置，也许是 PEPyEmu 加载的可执行程序的入口地址，也 许是 IDA Pro 光标所处的位置。start 为开始执行的地址，steps 为执行的指令数量，end 为结 束的地址。

### 12.2.5 内存和寄存器操作

修改和检索寄存器与内存的值在逆向的过程中特别重要。 PyEmu 将它们分成了 4 类： 内存，栈变量(stack variables)，栈参数(stack arguments)，寄存器。内存操作由 get_memory() 和 set_memory()完成。

```py
get_memory( address, size ) set_memory( address, value, size=0 ) 
```

get_memory()函数接收 2 个参数:address 为要查询的地址，size 为要获得数据的大小。 set_memoey()负责写入数据，address 为写入的地址，value 为写入的值，size 为写入数据的 大小。

另外两类基于栈操作的函数也差不多，主要负责栈框架中函数参数和本地变量的检索和 修改。

```py
set_stack_argument( offset, value, name="" ) 
get_stack_argument( offset=0x0, name="" ) 
set_stack_variable( offset, value, name="" ) 
get_stack_variable( offset=0x0, name="" ) 
```

set_stack_argument()的 offset 相对与 ESP，用于对传入函数的参数进行改变。在操作的过程中可以提供可以可选的名字。get_stack_argument()通过 offset 指定的相对于 ESP 的位移 获得参数值，或者通过指定的 name(前提是在 set_stack_argument 中提供了)获得。使用方式 如下：

```py
set_stack_argument( 0x8, 0x12345678, name="arg_0" ) get_stack_argument( 0x8 )
get_stack_argument( "arg_0" ) 
```

set_stack_variable()和 get_stack_variable()的操作也类似除了 offset 是相对于 EBP(如果允 许的话)以外，因为它们负责操作函数的局部变量。

### 12.2.6 处理函数

处理函数提供了一种非常强大且灵活的回调结构，用于 观察，设置或者修改程序的特定部分。PyEmu 中有 8 个主要处理函数： register 处理函数, library 处理函数, exception 处理函数, instruction 处理函数, opcode 处理函数, memory 处理 函数, high-level memory 处理函数还有 program counter 处理函数。让我们快速的了解下每一 个函数，之后我们马上要在用到它们。

12.2.6.1 Register 处理函数

Register Handlers 寄存器处理函数，用于监视任何寄存器的改变。只要有寄存器的遭到 修改就将触发 Register Handlers。安装方式如下：

```py
set_register_handler( register, register_handler_function ) 
set_register_handler( "eax ", eax_register_handler ) 
```

安装好之后，就需要定义处理函数了，原型如下：

```py
def register_handler_function( emu, register, value, type ): 
```

当处理函数被调用的时候，所有的参数都又 PyEmu 传入，第一个参数就是 PyEmu 实例 首，接着是寄存器名，以及寄存器的值，type 告诉我们这次操作是读还是写。时间久了你就 会发现用这种方式观察寄存器是有多么强大且方便，如果需要你还能在处理函数里改变它 们。

12.2.6.2 ### Library 处理函数

Library handle 库处理函数，能让我们捕捉所有的外部库调用，在它们被调用进程序之前就截获它们，这样就能很方便的修改外部库函数的调用方式以及返回值。安装方式如下：

```py
set_library_handler( function, library_handler_function ) 
set_library_handler( "CreateProcessA", create_process_handler ) 
set_library_handler("LoadLibraryA", loadlibrary) 
```

库处理函数的原型如下：

```py
def library_handler_function( emu, library, address ): 
```

第一个参数就是 PyEmu 的实例。library 为我们想要监视的函数，或者库，第三个是函 数被映射在内存中的地址。

12.2.6.3 Exception 处理函数

Exception Handlers 异常处理函数和第二章介绍的"处理函数相似"。PyEmu 仿真器中的 异常会触发 Exception Handlers 的调用。当前 PyEmu 支持通用保护错误，也就是说我们能够 处理在模拟器中的任何内存访问违例。安装方式如下：

```py
set_exception_handler( "GP", gp_exception_handler ) 
```

Exception 处理函数原型如下：

```py
def gp_exception_handler( emu, exception, address ): 
```

同样，第一个参数是 PyEmu 实例，exception 为异常代码，address 为异常发生的地址。

12.2.6.4 Instruction 处理函数

Instruction Handlers 指令处理函数，很强大，因为它能捕捉任何特定的指令。就像 Cody 在 BlackHat 说展示的那样，你能够通过安装一个 CMP 指令的处理函数，来监视整个程序流 程的分支判断，并控制它们。

```py
set_instruction_handler( instruction, instruction_handler ) 
set_instruction_handler( "cmp", cmp_instruction_handler ) 
```

Instruction 处理函数原型如下：

```py
def cmp_instruction_handler( emu, instruction, op1, op2, op3 ): 
```

第一个参数照旧是 PyEmu 实例，instruction 则为被执行的指令，另外三个都是可能的运 算对象。

12.2.6.5 Opcode 处理函数

Opcode handlers 操作码处理函数和指令处理函数非常相似，任何一个特定的操作码被执 行的时候，都会调用 Opcode handlers。这样我们对代码的控制就变得更精确了。每一个指令 都有可能有不同的操作码这依赖于它们的运算对象，例如，PUSH EAX 时操作码是 0x50， 而 PUSH 0x70 时操作码是 0x6A，合起来整个指令的操作码就是 0x6A70,如下所示：

```py
50 PUSH EAX
6A 70 PUSH 0x70 
```

它们的安装方法很简单：

```py
set_opcode_handler( opcode, opcode_handler ) 
set_opcode_handler( 0x50, my_push_eax_handler ) 
set_opcode_handler( 0x6A70, my_push_70_handler ) 
```

第一个参数只要简单的设置成我们需要捕捉的操作码，第二个参数就是处理函数了。捕捉的范围不限于单个字节，而可以是多这个字节，就想第二个例子一样。处理函数原型如下：

```py
def opcode_handler( emu, opcode, op1, op2, op3 ): 
```

第一个 PyEmu 实例，后面不再累赘。opcode 是捕捉到的操作码，剩下的三个就是指令 可能使用到的计算对象。

12.2.6.6 Memory 处理函数

Memory handlers 内存处理函数用于跟踪特定地址的数据访问。它能让我们很方便的跟 踪缓冲区中感兴趣的数据以及全局变量的改变过程。安装过程如下：

```py
set_memory_handler( address, memory_handler ) 
set_memory_handler( 0x12345678, my_memory_handler ) 
```

address 简单传入我们想要观察的内存地址， my_memory_handler 就是我们的处理函 数。函数原型如下：

```py
def memory_handler( emu, address, value, size, type ) 
```

第二个参数 address 为发生内存访问的地址，value 是被读取或者写入的数据，size 是数 据的大小，type 告诉我们这次操作读还是写。

12.2.6.7 High-Level Memory 处理函数

High-Level Memory Handlers 高级内存处理函数，很高级很强大。通过安装它们，我们 就能监视这个内存快（包括栈和堆）的读写。这样就能全面的控制内存的访问，是不是很邪恶。安装方式如下：

```py
set_memory_write_handler( memory_write_handler ) 
set_memory_read_handler( memory_read_handler ) 
set_memory_access_handler( memory_access_handler )
set_stack_write_handler( stack_write_handler ) 
set_stack_read_handler( stack_read_handler ) 
set_stack_access_handler( stack_access_handler )
set_heap_write_handler( heap_write_handler ) 
set_heap_read_handler( heap_read_handler ) 
set_heap_access_handler( heap_access_handler ) 
```

所有的这些安装函数只要简单的提供一个处理函数就可以了，任何内存的变动都会通知 我们。处理函数的原型如下：

```py
def memory_write_handler( emu, address ): 
def memory_read_handler( emu, address ):
def memory_access_handler( emu, address, type ): 
```

memory_write_handler 和 memory_read_handler 只是简单的接收 PyEmu 实例和发生读写 的地址。第三个 access handler 多了一个 type 用于说明这次不做到的是读数据还是些数据。 栈和堆的处理函数和上面的一样，不做解说。

12.2.6.8 Program Counter 处理函数

The program counter handler 程序计数器处理函数，将在程序执行到特定地址的时候触 发。安装过程如下：

```py
set_pc_handler( address, pc_handler ) 
set_pc_handler( 0x12345678, 12345678_pc_handler ) 
```

address 为我们将要监视的地址，一旦 CPU 执行到这就会触发我们的处理函数。处理 函数的原型如下：

```py
def pc_handler( emu, address ): 
```

第二个参数 address 为被捕捉到的地址。

现在我们已经讲解完了，PyEmu 的基础知识。是时候将它们用于实际工作中了。接下 来会进行两个实验。第一个使用 IDAPyEmu 在 IDA Pro 模拟一个简单的函数调用。第二个 实验使用 PEPyEmu 解压一个被 UPX 压缩过的（伟大的开源压缩程序）二进制文件。

# 12.3 IDAPyEmu

## 12.3 IDAPyEmu

我们的第一个例子就是在 IDA Pro 分析程序的时候，使用 PyEmu 仿真一次简单的函数 调用。这次实验的程序就是 addnum.exe，主要功能就是从命令行中接收两个参数，然后相 加，再输出结果，代码使用 C++ 编写，可从 [`www.nostarch.com/ghpython.htm`](http://www.nostarch.com/ghpython.htm) 下载。

```py
/*addnum.cpp*/
#include <stdlib.h>
#include <stdio.h>
#include <windows.h>
int add_number( int num1, int num2 )
{
    int sum;
    sum = num1 + num2; 
    return sum;
}
int main(int argc, char* argv[])
{
    int num1, num2; 
    int return_value; 
    if( argc < 2 )
    {
        printf("You need to enter two numbers to add.\n"); printf("addnum.exe num1 num2\n");
        return 0;
    } 
    num1 = atoi(argv[1]); 
    num2 = atoi(argv[2]);
    return_value = add_number( num1, num2 );
    printf("Sum of %d + %d = %d",num1, num2, return_value ); 
    return 0;
} 
```

程序将命令行传入的参数转换成整数，然后调用 add_number 函数相加。我们将 add_number 函数作为我们的仿真对象，因为它够简单而且结果也很容易验证，作为我们使 用 PyEmu 的起点是个不二选择。

在深入 PyEmu 使用之前，让我们看看 add_number 的反汇编代码。

```py
var_4= dword ptr -4 
# sum variable arg_0= dword ptr 8 
# int num1 arg_4= dword ptr 0Ch 
# int num2 push ebp
mov ebp, esp
push ecx
mov eax, [ebp+arg_0]
add eax, [ebp+arg_4]
mov [ebp+var_4], eax
mov eax, [ebp+var_4]
mov esp, ebp
pop ebp retn 
```

Listing 12-1: add_number 的反汇编代码

var_4，arg_0，arg_4 分别是参数在栈中的位置，从 C++的反汇编代码中可以清楚的看 出，整个函数的执行流程，和参数的调用关系。我们将使用 PyEmu 仿真整个函数，也就是 上面列出的汇编代码，同时设置 arg_0 和 arg_4 为我们需要的任何数，最后 retn 返回的时候， 捕获 EAX 的值，也就是函数的返回值。虽然仿真的函数似乎过于简单，不过整个仿真过程 就是一切函数仿真的基础，一通百通。

### 12.3.1 函数仿真

开始脚本编写，第一步确认 PyEmu 的路径设置正确。

```py
#addnum_function_call.py 
import sys 
sys.path.append("C:\\PyEmu") 
sys.path.append("C:\\PyEmu\\lib") 
from PyEmu import * 
```

设置好库路径之后，就要开始函数仿真部分的编写了。首先将我们逆向的程序的，代码 块和数据块映射到仿真器中，以便仿真器仿真运行。因为我们会使用 IDAPython 加载这些块，对相关函数不熟悉的同学，请翻到第十一章，认真阅读。

```py
#addnum_function_call.py
...
emu = IDAPyEmu()
# Load the binary's code segment 
code_start = SegByName(".text") 
code_end = SegEnd( code_start )
while code_start <= code_end:
    emu.set_memory( code_start, GetOriginalByte(code_start), size=1 ) 
    code_start += 1
print "[*] Finished loading code section into memory."
# Load the binary's data segment 
data_start = SegByName(".data") 
data_end = SegEnd( data_start )
while data_start <= data_end:
    emu.set_memory( data_start, GetOriginalByte(data_start), size=1 ) 
    data_start += 1
print "[*] Finished loading data section into memory." 
```

使用任何仿真器方法之前都必须实例化一个 IDAPyEmu 对象。接着将代码块和数据块 加载进 PyEmu 的内存，名副其实的依葫芦画瓢喔。使用 IDAPython 的 SegByName()函数找 出块首，SegEnd()找出块尾。然后一个一个字节的将这些块中的数据拷贝到 PyEmu 的内存 中。代码和数据块都加载完成后，就要设置栈参数了，这些参数可以任意设置，最后再安装 一个 retn 指令处理函数。

```py
#addnum_function_call.py
...
# Set EIP to start executing at the function head 
emu.set_register("EIP", 0x00401000)
# Set up the ret handler 
emu.set_mnemonic_handler("ret", ret_handler)
# Set the function parameters for the call
emu.set_stack_argument(0x8, 0x00000001, name="arg_0")
emu.set_stack_argument(0xc, 0x00000002, name="arg_4")
# There are 10 instructions in this function 
emu.execute( steps = 10 ) 
print "[*] Finished function emulation run." 
```

首先将 EIP 指向到函数头，0x00401000，PyEmu 仿真器将从这里开始执行指令。接着， 在函数的 retn 指令上设置 助记符(mnemonic)或者指令处理函数(set_instruction_handler)。第 三步，设置栈参数以供函数调用。在这里设置成 0x00000001 和 0x00000002。最后让 PyEmu 执行完成整个函数 10 行代码。完整的代码如下。

```py
#addnum_function_call.py
import sys
sys.path.append("C:\\PyEmu") 
sys.path.append("C:\\PyEmu\\lib") 
from PyEmu import *
def ret_handler(emu, address): 
    num1 = emu.get_stack_argument("arg_0") 
    num2 = emu.get_stack_argument("arg_4")
    sum = emu.get_register("EAX")
    print "[*] Function took: %d, %d and the result is %d." % (num1, n
    return True emu = IDAPyEmu()
    # Load the binary's code segment 
    code_start = SegByName(".text") 
    code_end = SegEnd( code_start ) 
    while code_start <= code_end:
        emu.set_memory( code_start, GetOriginalByte(code_start), size=1 ) 
        code_start += 1
    print "[*] Finished loading code section into memory."
    # Load the binary's data segment 
    data_start = SegByName(".data") 
    data_end = SegEnd( data_start ) 
    while data_start <= data_end:
        emu.set_memory( data_start, GetOriginalByte(data_start), size=1 ) 
        data_start += 1
    print "[*] Finished loading data section into memory."
    # Set EIP to start executing at the function head 
    emu.set_register("EIP", 0x00401000)
    # Set up the ret handler 
    emu.set_mnemonic_handler("ret", ret_handler)
    # Set the function parameters for the call 
    emu.set_stack_argument(0x8, 0x00000001, name="arg_0") 
    emu.set_stack_argument(0xc, 0x00000002, name="arg_4")
    # There are 10 instructions in this function 
    emu.execute( steps = 10 )
    print "[*] Finished function emulation run." 
```

ret 指令处理函数简单的设置成检索出栈参数和 EAX 的值，最后再将它们打印出来。 用 IDA 加载 addnum.exe，然后将 PyEmu 脚本当作 IDAPython 文件调用。输出结果将如下：

```py
[*] Finished loading code section into memory. 
[*] Finished loading data section into memory. 
[*] Function took 1, 2 and the result is 3.
[*] Finished function emulation run. 
```

Listing 12-2: IDAPyEmu 仿真函数的输出

很好很简单！整个过程很成功，栈参数和返回值都从捕获，说明函数仿真成功了。作为进一步的练习，各位可以加载不同的文件，随机的选择一个函数进行仿真，然后监视相关数 据的调用或者任何感兴趣的东西。某一天，当你遇到一个上千行的函数的时候，相信这种方 法能帮你从无数的分支，循环还有可怕的指针中拯救出来，它们节省的不仅仅是事件，更是 你的信心。接下来让我们用 PEPyEmu 库解压一个被压缩文件。

### 12.3.2 PEPyEmu

PEPyEmu 类用于可执行文件的静态分析（不需要 IDA Pro）。整个处理过程就是将磁盘 上的可执行文件映射到内存中，然后使用 pydasm 进行指令解码。下面的试验中，我们将通 过仿真器运行一个压缩过的可执行文件，然后把解压出来的原始文件转存到硬盘上。这次使 用的压缩软件就是 UPX(Ultimate Packer for Executables)，一款伟大的开源压缩软件，同时也 是使用最广的压缩软件，用于最大程度的压缩可执行文件，同样也能被病毒软件用来迷惑分 析者。在使用自定义 PyEmu 脚本( Cody Pierce 提供 )对程序进行解压之前，让我们看看压缩 程序是怎么工作的。

### 12.3.3 压缩程序

压缩程序由来已久。最早在我们使用 1.44 软盘的时候，压缩程序就用来尽可能的减少 程序大小(想当初我们的软盘上可是有上千号文件)，随着事件的流逝，这项技术也渐渐成为 病毒开发中的一个主要部分，用来迷惑分析者。一个典型的压缩程序会将目标程序的代码段 和数据段进行压缩，然后将入口点替换成解压的代码。当程序执行的时候，解压代码就会将 原始代码加压进内存，然后跳到原始入口点 OEP(original entry point )，开始正常运行程序。 在我们分析调试任何压缩过的程序之前，也都必须解压它们。这时候你会想到用调试器完成 这项任务（因为各种丰富的脚本），不过现在的病毒一般都缴入反调试代码，用调试器进行 解压变得越来越困难。那怎么办呢？用仿真器。因为我们并没有附加到正在执行的程序，而 是将压缩过的代码拷贝到仿真器中运行，然后等待它自动解压完成，接着再把解压出来的原 始程序，转储到硬盘上。以后就能够正常的分析调试它们了。

这次我们选择 UPX 压缩 calc.exe。然后用 PyEmu 解压它，最后 dump 出来。记得这种 方法同样适用于别的压缩程序，万变不离其宗。

1.  ### UPX

UPX 是自由的，是开源的，是跨平台的(Linux Windows....)。提供不同的压缩级别，和 许多附加的选项，用于完成各种不同的压缩任务。我们使用默认的压缩方案，都让你可随意 的测试。

从 [`upx.sourceforge.net`](http://upx.sourceforge.net) 下载 UPX。

解压到 C 盘，官方没有提供图形界面，所以我们必须从命令行操作。打开 CMD，改 变当前目录到 C:\upx303w(也就是 UPX 解压的目录)，输入以下命令：

```py
C:\upx303w>upx -o c:\calc_upx.exe C:\Windows\system32\calc.exe
Ultimate Packer for eXecutables 
Copyright (C) 1996 - 2008
UPX 3.03w Markus Oberhumer, Laszlo Molnar & John Reiser Apr 27th 2008
File size Ratio Format Name
-------------------- ------ ----------- -----------
114688 -> 56832 49.55% win32/pe calc_upx.exe
Packed 1 file. C:\upx303w> 
```

成功的压缩了 Windows 的计算器，并且转储到了 C 盘下。

-o 为输出标志，指定输出文件名。接下来，终于到了 PEPyEmu 出马了。

1.  使用 PEPyEmu 解压 UPX

UPX 压缩可执行程序的方法很简单明了：重写程序的入口点，指向解压代码，同时添 加两个而外的块，UPX0 和 UPX1。使用 Immunity 加载压缩程序,检查内存布局(ALT-M),将会看到如下相似的输出：

```py
Address Size Owner Section Contains Access Initial Access 00100000 00001000 calc_upx PE Header R RWE 
01001000 00019000 calc_upx UPX0 RWE RWE
0101A000 00007000 calc_upx UPX1 code RWE RWE 
01021000 00007000 calc_upx .rsrc data,imports RW RWE
resources 
```

Listing 12-3: UPX 压缩之后的程序的内存布局.

UPX1 显示为代码块，其中包含了主要的解压代码。代码经过 UPX1 的解压之后，就跳 出 UPX1 块，到达真正的可执行代码块，开始执行程序。我们要做的就是让仿真器运行解压 代码，同时不断的检测 EIP 和 JMP，当发现有 JMP 指令使得 EIP 的范围超出 UPX1 段的时 候，说明将到跳转到原始代码段了。

接下来开始代码的编写，这次我们只使用独立的 PEPyEmu 模块。

```py
#upx_unpacker.py
from ctypes import *
# You must set your path to pyemu 
sys.path.append("C:\\PyEmu") 
sys.path.append("C:\\PyEmu\\lib")
from PyEmu import PEPyEmu
# Commandline arguments 
exename = sys.argv[1] 
outputfile = sys.argv[2]
# Instantiate our emulator object 
emu = PEPyEmu()
if exename:
    # Load the binary into PyEmu 
    if not emu.load(exename): 
        print "[!] Problem loading %s" % exename 
        sys.exit(2)
else:
    print "[!] Blank filename specified" 
    sys.exit(3) # Set our library handlers
emu.set_library_handler("LoadLibraryA", loadlibrary) 
emu.set_library_handler("GetProcAddress", getprocaddress) 
emu.set_library_handler("VirtualProtect", virtualprotect)
# Set a breakpoint at the real entry point to dump binary 
emu.set_mnemonic_handler( "jmp", jmp_handler )
# Execute starting from the header entry point 
emu.execute( start=emu.entry_point ) 
```

第 一 步 将 压 缩 文 件 加 载 进 PyEmu 。 第 二 部 ， 在 LoadLibraryA, GetProcAddress, VirtualProtect 三个函数上设置库处理函数。这些函数都将在解压代码中调用，这些操作必须 我们自己在仿真器中完成。第三步，在解压程序执行完成准备跳到 OEP 的时候，我们将进 行相关的操作，这个任务就有 JMP 指令处理函数完成。最后告诉仿真器，从压缩程序头部 开始执行代码。

```py
#upx_unpacker.py
from ctypes import *
# You must set your path to pyemu 
sys.path.append("C:\\PyEmu") 
sys.path.append("C:\\PyEmu\\lib") 
from PyEmu import PEPyEmu
'''
HMODULE WINAPI LoadLibrary(
    in LPCTSTR lpFileName
); 
'''
def loadlibrary(name, address):
# Retrieve the DLL name
dllname = emu.get_memory_string(emu.get_memory(emu.get_register("ESP")))
# Make a real call to LoadLibrary and return the handle 
dllhandle = windll.kernel32.LoadLibraryA(dllname)
emu.set_register("EAX", dllhandle)
# Reset the stack and return from the handler
return_address = emu.get_memory(emu.get_register("ESP")) 
emu.set_register("ESP", emu.get_register("ESP") + 8) 
emu.set_register("EIP", return_address)
return True
'''
FARPROC WINAPI GetProcAddress(
    in HMODULE hModule,
    in LPCSTR lpProcName
);
''' 
def getprocaddress(name, address):
    # Get both arguments, which are a handle and the procedure name 
    handle = emu.get_memory(emu.get_register("ESP") + 4) 
    proc_name = emu.get_memory(emu.get_register("ESP") + 8)
    # lpProcName can be a name or ordinal, if top word is null it's an ordinal
    # lpProcName 的高 16 位是 null 的时候,它就是序列号(也就是个地址),否者就是名字
    if (proc_name >> 16):
        procname = emu.get_memory_string(emu.get_memory(emu.get_register("ESP") + 8)) 
    else:
        procname = arg2
    #这 arg2 不知道从何而来,应该是 procname = proc_name
    # Add the procedure to the emulator 
    emu.os.add_library(handle, procname)
    import_address = emu.os.get_library_address(procname)
    # Return the import address 
    emu.set_register("EAX", import_address)
    # Reset the stack and return from our handler
    return_address = emu.get_memory(emu.get_register("ESP")) 
    emu.set_register("ESP", emu.get_register("ESP") + 8)
    #这里应该是 r("ESP") + 8，因为有两个参数需要平衡
    emu.set_register("EIP", return_address) 
    return True
'''
BOOL WINAPI VirtualProtect(
    in LPVOID lpAddress,
    in SIZE_T dwSize,
    in DWORD flNewProtect,
    out PDWORD lpflOldProtect
);
''' 
def virtualprotect(name, address):
    # Just return TRUE
    emu.set_register("EAX", 1)
    # Reset the stack and return from our handler
    return_address = emu.get_memory(emu.get_register("ESP")) 
    emu.set_register("ESP", emu.get_register("ESP") + 16) 
    emu.set_register("EIP", return_address)
    return True
# When the unpacking routine is finished, handle the JMP to the OEP 
def jmp_handler(emu, mnemonic, eip, op1, op2, op3):
    # The UPX1 section
    if eip < emu.sections["UPX1"]["base"]:
        print "[*] We are jumping out of the unpacking routine." 
        print "[*] OEP = 0x%08x" % eip
        # Dump the unpacked binary to disk 
        dump_unpacked(emu)
        # We can stop emulating now 
        emu.emulating = False
        return True 
```

LoadLibrary 处理函数从栈中捕捉到调用的 DLL 的名字，然后使用 ctypes 库函数进 行真正的 LoadLibraryA 调用，这个函数由 kernel32.dll 导出。调用成功返回后，将句柄传递 给 EAX 寄存器，重新调整仿真器栈，最后重处理函数返回。同样， GetProcAddress 处理函 数 从 栈 中 接 收 两 个 参 数(arg2) ， 然 后 在 仿 真 器 中 进 行 真 实 的 调 用(emu.os.add_library 和 emu.os.get_library_address) ， 这 个 函 数 也 由 kernel32.dll 导 出 ( 当 然 也 可 以 使 用 windll.kernel32.GetProcAddress) 。 之 后 把 地 址 存 储 到 EAX ， 调 整 栈 ( 这 里 原 作 者 使 用 emu.set_register("ESP", emu.get_register("ESP") + 8)，不过由于是两个参数，应该是+12)，返 回。第三个 VirtualProtect 处理函数，只是简单的返回一个 True 值，接着就是一样的栈处理 和从函数中返回。之所以这样做，是因为我们不需要真正的保护内存中的某个页面；我们值 需要确保在仿真器中的 VirtualProtect 调用都返回真。最后的 JMP 指令处理函数做了一个简 单的确认，看是否要跳出解压代码段，如果跳出，就调用 dump_unpacked 将代码转储到硬 盘上。之后告诉仿真器停止工作，解压工作完成了。

下面就是 dump_unpacked 代码。

```py
#upx_unpacker.py
...
def dump_unpacked(emu): 
    global outputfile
    fh = open(outputfile, 'wb')
    print "[*] Dumping UPX0 Section" 
    base = emu.sections["UPX0"]["base"]
    length = emu.sections["UPX0"]["vsize"]
    print "[*] Base: 0x%08x Vsize: %08x"% (base, length) 
    for x in range(length):
        fh.write("%c" % emu.get_memory(base + x, 1)) 
    print "[*] Dumping UPX1 Section"
    base = emu.sections["UPX1"]["base"] 
    length = emu.sections["UPX1"]["vsize"]
    print "[*] Base: 0x%08x Vsize: %08x" % (base, length) 
    for x in range(length):
        fh.write("%c" % emu.get_memory(base + x, 1)) 
    print "[*] Finished." 
```

我们只需要简单的将 UPX0 和 UPX1 两个段的代码写入文件。一旦文件 dump 成功，就能够想正常程序一样分析调试它们了。在命令行中使用我们的解压脚本看看：

```py
C:\>C:\Python25\python.exe upx_unpacker.py C:\calc_upx.exe calc_clean.exe 
[*] We are jumping out of the unpacking routine.
[*] OEP = 0x01012475
[*] Dumping UPX0 Section
[*] Base: 0x01001000 Vsize: 00019000
[*] Dumping UPX1 Section
[*] Base: 0x0101a000 Vsize: 00007000 [*] Finished.
C:\> 
```

Listing 12-4:upx_unpacker.py 的命令行输出

现在我们有了一个和未加密的 calc.exe 一样的 calc_clean.exe。大功告成，各位不妨测试 着写写不同壳的解压代码，相信不久之后你会学到更多。