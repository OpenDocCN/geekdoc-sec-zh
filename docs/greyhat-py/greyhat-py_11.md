# 十、Fuzzing Windows 驱动

# 10 Fuzzing Windows 驱动

对于 hacker 来说，攻击 Windows 驱动程序已经不再神秘。

从前，驱动程序常被远程溢出，而如今驱动漏洞越来越多的用于本地提权。在前面我们使用 Sulley 找出了 WarFTPD 的溢出漏洞。

WarFTPD 在远程的机器上由一个受限的用户启动，我们在远程溢出它之后，就会获得一个 受限的权限，这个权限一般是很小的，如果似乎，很多信息都无法获取，很多服务都访问不 了。如果这时候我们拥有一个本地驱动的 exploit，那就能够将权限提升到系统级别，you are god now!

驱动在内核模式下运行，而我们的程序在用户模式下运行，为了在两种模式之间进行交 互，就要使用 IOCTLs（input/output controls ）。当 IOCTLs 处理代码有问题的时候，我们就 能利用它获取系统权限。

接下来，我们首先要介绍下如何通过实现 IOCTLs 来和本地的设备进行联系，并且尝试 使用 Immunity 变形 IOCTLs 数据。然后，学会使用 Immunity 提供的 driverlib 库获取驱动信 息，以及从一个编译好的驱动文件中解码出重要的控制流程，设备名，和 IOCTL 代码。最 后用从 drivelib 获得的数据构建测试数据，使用 ioctlizer（我写的一个驱动 fuzzer）进行一次 driver fuzz。

# 10.1 驱动通信

## 10.1 驱动通信

几乎每个在 Windows 上注册了的驱动程序都有一个设备名和一个符号链接。用户模式 的程序能够通过符号链接获得驱动的句柄，然后使用这个句柄和驱动进行联系。具体函数如 下：

```py
HANDLE WINAPI CreateFileW( 
    LPCTSTR lpFileName, 
    DWORD dwDesiredAccess, 
    DWORD dwShareMode,
    LPSECURITY_ATTRIBUTES lpSecurityAttribute 
    DWORD dwCreationDisposition,
    DWORD dwFlagsAndAttributes, 
    HANDLE hTemplateFile
); 
```

第一个参数，填写文件名或者设备名，这里填写目标驱动的符号连接。 dwDesiredAccess 表示访问方式，读或者写（可以既读又写，也可以不读不写），GENERIC_READ (0x80000000) 读，GENERIC_WRITE (0x40000000)写。dwShareMode 这里设置成 0，表示在 CreateFileW 返回并且安全关闭了句柄之后，才能访问设备。 lpSecurityAttributes 设置成 NULL，表示使 用 默 认 的 安 全 描 述 符 ， 并 且 不 能 被 子 进 程 继 承 。 dwCreationDisposition 参 数 设 置 成 OPEN_EXISTING (0x3)，表示如果设备存在就打开，其余情况返回错误。最后两个参数简 单的设置成 NULL。

当 CreateFileW 成功返回一个有效的句柄之后，我们就能使用 DeviceIoControl（由 kernel32.dll 导出）传递一个 IOCTL 给设备。

```py
BOOL WINAPI DeviceIoControl( 
    HANDLE hDevice,
    DWORD dwIoControlCode, 
    LPVOID lpInBuffer,
    DWORD nInBufferSize, 
    LPVOID lpOutBuffer, 
    DWORD nOutBufferSize, 
    LPDWORD lpBytesReturned,
    LPOVERLAPPED lpOverlapped
); 
```

第 一个 参数 由 CreateFileW 返 回的 句柄 。 dwIoControlCode 是 要传 递给 设备 启动 的 IOCTL 代码。这个代码决定了调用驱动中的什么功能。参数 lpInBuffer 指向一个缓冲区，包 含了将要传递给驱动的数据。这个缓冲区是我们后面要重点操作的地方，fuzz 数据将存在这。 nInBufferSize 为传递给驱动的缓冲区的大小。 lpOutBuffer 和 lpOutBufferSize，和前两个参 数一样，不过是用于接收驱动返回的数据。lpBytesReturned 为驱动实际返回的数据的长度。 最后一个参数简单的设置成 NULL。

现在对于驱动的交互，大家应该不陌生了，接下来就祭出我们的 Immunity，用它 Hook 住 DeviceIoControl 然后变形输入缓冲区内的数据，最后 fuzzing every driver。

# 10.2 用 Immunity fuzzing 驱动

## 10.2 用 Immunity fuzzing 驱动

我们需要使用 Immunity 强大的调试功能，挂钩住 DeviceIoControl 函数，在数据到达目 标驱动之前，截获它们，这就是我们 Driver Fuzzing 的基础。如果一切顺利，最后可以将一 些列工作写出自动化的 PyCommand，我们只要喝着茶看着 Immunity 完成一切工作：截获 DeviceIoControl，变形缓冲区数据，记录相关信息，将控制权交还给目标程序。之所以要对 数据进行记录，是因为每次成功的 fuzzing 都会引起系统奔溃，而记录可以更好的还原崩溃 时发送的数据。

提示

确保不要在自己的机器上进行实验。除非你想见到无数次的蓝屏，重启，最后就是 硬盘报销的声音，哈哈！老天保佑，我们还可以使用虚拟机，虽然它的模拟在某些底层细节 上不是很好，不过这可比硬盘便宜。

开动代码。新建一个 Python 脚本 ioctl_fuzzer.py。

```py
#ioctl_fuzzer.py 
import struct 
import random
from immlib import *
class ioctl_hook( LogBpHook ):
    def init ( self ):
        self.imm = Debugger() 
        self.logfile = "C:\ioctl_log.txt" 
        LogBpHook. init ( self )
    def run( self, regs ): 
        """
        We use the following offsets from the ESP register to trap the arguments to DeviceIoControl:
        ESP+4 -> hDevice 
        ESP+8 -> IoControlCode
        ESP+C -> InBuffer 
        ESP+10 -> InBufferSize 
        ESP+14 -> OutBuffer 
        ESP+18 -> OutBufferSize 
        ESP+1C -> pBytesReturned 
        ESP+20 -> pOverlapped
        """
        in_buf = ""
        # read the IOCTL code 
        ioctl_code = self.imm.readLong( regs['ESP'] + 8 )
        # read out the InBufferSize 
        inbuffer_size = self.imm.readLong( regs['ESP'] + 0x10 )
        # now we find the buffer in memory to mutate 
        inbuffer_ptr = self.imm.readLong( regs['ESP'] + 0xC )
        # grab the original buffer
        in_buffer = self.imm.readMemory( inbuffer_ptr, inbuffer_size ) 
        mutated_buffer = self.mutate( inbuffer_size )
        # write the mutated buffer into memory 
        self.imm.writeMemory( inbuffer_ptr, mutated_buffer )
        # save the test case to file
        self.save_test_case( ioctl_code, inbuffer_size, in_buffer, mutated_buffer )
    def mutate( self, inbuffer_size ):
        counter = 0 
        mutated_buffer = ""
        # We are simply going to mutate the buffer with random bytes 
        while counter < inbuffer_size:
            mutated_buffer += struct.pack( "H", random.randint(0, 255) )[0] 
            counter += 1
        return mutated_buffer
    def save_test_case( self, ioctl_code,inbuffer_size, in_buffer, mutated_buffer ):
        message = "***** | |\n"
        message += "IOCTL Code: 0x%08x\n" % ioctl_code 
        message += "Buffer Size: %d\n" % inbuffer_size 
        message += "Original Buffer: %s\n" % in_buffer
        message += "Mutated Buffer: %s\n" % mutated_buffer.encode("HEX") 
        message += "***** | |\n\n"
        fd = open( self.logfile, "a" ) 
        fd.write( message ) 
        fd.close()
def main(args):
    imm = Debugger()
    deviceiocontrol = imm.getAddress( "kernel32.DeviceIoControl" ) 
    ioctl_hooker = ioctl_hook()
    ioctl_hooker.add( "%08x" % deviceiocontrol, deviceiocontrol ) 
    return "[*] IOCTL Fuzzer Ready for Action!" 
```

这里没有用到任何新的 Immunity 知识，只是继承了 LogBpHook 类，做了很小的扩展， 这一切都在第五章做了详细的介绍。代码非常清晰明了，显示获得传递给驱动的 IOCT 代 码，输入缓冲区长度，输入缓冲区位置。接着通过对输入数据的变形，创建一个包含了随即 字符的新缓冲区，长度和输入缓冲区一样。之后将新缓冲区的数据写入原缓冲区，保存测试 样例。最后把控制权交还给用户程序。

记得把 ioctl_fuzzer.py 放到 PyCommands 目录下。这样我们就能使用 ioctl_fuzzer 命令 fuzz 任何使用 IOCTLs 了的程序（嗅探器，防火墙，或者杀毒软件）。表 10-1 是 Wireshark 的 fuzz 结果。

```py
*****
IOCTL Code: 0x00120003
Buffer Size: 36
Original Buffer: 0000000000000000000100000001000000000000000000000000000000000000
Mutated Buffer: a4100338ff334753457078100f78bde62cdc872747482a51375db5aa2255c46e
*****
*****
IOCTL Code: 0x00001ef0 
Buffer Size: 4
Original Buffer: 28010000 
Mutated Buffer: ab12d7e6
***** 
```

Listing 10-1: Wireshark 的 fuzzing 输出

在 我 们 将 一 大 堆 的 垃 圾 扔 给 驱 动 器 之 后 ， 在 于 发 现 了 两 个 可 用 的 IOCTL 代 码 0x00001ef0 和 0x0012003 。如果要继续测试，就必须不断的和用户模式下的 Wireshark 进行 交互，这样 Wireshark 就会调用不同 IOCTL 代码，最后祈祷上帝让其中一个 IOCTL 处理代 码发生崩溃。

虽然这样做很简单，也确实很够找出漏洞。不过还是不够聪明。举个例子，我们并不知 道正在 fuzzing 的设备名，（不过可以通过 hook CreateFileW，然后观察被 DeviceIoControl 使用了的句柄，从而逆推得到设备名 ），而且 fuzz 的 IOCTL 代码并不全，我们在用户模式 下对程序进行的操作是有限的，这样程序对驱动功能的调用也是有限的。这就像碰运气。我 们期待的是一个更加聪明的 fuzzer，它能对所有的 IOCTL 不间断的 fuzzing，直到你的硬盘 报销，或者在这之前发现一个漏洞。

这可能吗，可能，先从我们伟大的 Immunity 携带的 driverlib 库开始。使用 driverlib 我 们能枚举出驱动程序所有的设备名和 IOCTL 代码。把这些结合起来就能够实现一个高效， 独立，全自动化的 fuzzer 了，这是一个伟大的进步，解放双手，不做野蛮人。Let’s get cracking。

### 10.3.1 找出设备名

用 Immunity 内建的 driverlib 库找出设备名很就当。让我们看看 driverlib 是怎么实现这 个功能的。

```py
def getDeviceNames( self ):
    string_list = self.imm.getReferencedStrings( self.module.getCodebase() )
    for entry in string_list:
        if "\\Device\\" in entry[2]:
            self.imm.log( "Possible match at address: 0x%08x" % entry[0], address =
            entry[0] )
        self.deviceNames.append( entry[2].split("\"")[1] )
    self.imm.log("Possible device names: %s" % self.deviceNames)
    return self.deviceNames 
```

Listing 10-2: driverlib 库找出设备名的方法

代码通过检索驱动中所有被引用了的字符串，找出其中包含了 "\Device\"的项。这项就可能是驱动程序注册了的符号链接，用来让用户模式下的程序调用的。我们就使用 C:\WINDOWS\System32\beep.sys 测试以下看看。以下操作都在 Immunity 中进行。

```py
*** Immunity Debugger Python Shell v0.1 
*** Immlib instanciated as 'imm' PyObject READY.
>>> import driverlib
>>> driver = driverlib.Driver()
>>> driver.getDeviceNames() ['\\Device\\Beep']
>>> 
```

我们很简单的使用三行代码就找到了一个可用的设备名 \Device\Beep,这省去了我们通 过反汇编一行行查找代码的时间。Simple is Beautiful！下面看看 driverlib 是如何查找 IOCTL dispatch function（IOCTL 调度函数）和 IO IOCTL codes（ IOCTL 代码）的。

任何驱动要实现 IOCTL 接口，都必须有一个 IOCTL dispatch 负责处理各种 IOCTL 请求。 当驱动被加载的似乎后，第一个访问的函数就是 DriverEntry。DriverEntry 的主要框架如下：

```py
NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath)
{
    UNICODE_STRING uDeviceName; 
    UNICODE_STRING uDeviceSymlink; P
    DEVICE_OBJECT gDeviceObject;
    RtlInitUnicodeString( &uDeviceName, L"\\Device\\GrayHat" ); 
    RtlInitUnicodeString( &uDeviceSymlink, L"\\DosDevices\\GrayHat" )
    // Register the device
    IoCreateDevice( DriverObject, 0, &uDeviceName, FILE_DEVICE_NETWORK, 0, FALSE, &gDeviceObject );
    // We access the driver through its symlink 
    IoCreateSymbolicLink(&uDeviceSymlink, &uDeviceName);
    // Setup function pointers
    DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = IOCTLDispatch; 
    DriverObject->DriverUnload = DriverUnloadCallbac 
    DriverObject->MajorFunction[IRP_MJ_CREATE] = DriverCreateCloseCa 
    DriverObject->MajorFunction[IRP_MJ_CLOSE] = DriverCreateCloseCa 
    return STATUS_SUCCESS;
} 
```

Listing 10-3: DriverEntry 的 C 源码实现

这是一个非常基础的 DriverEntry 代码框架，但是很直观的说明了设备是如何初始化的。 要注意的是这行:

```py
DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = IOCTLDispatch 
```

这行告示驱动器 IOCTLDispatch 负责所有 IOCTL 请求。当一个驱动器编译完成后，这 行程序的汇编伪代码如下：

```py
mov dword ptr [REG+70h], CONSTANT 
```

这指令集看起来有些特殊，REG 和 CONSTANT 都是汇编代码，IOCTLDispatch 指针将 被 存 储 在 (REG) 位 移 0x70 的 地 方 上 。 使 用 这 些 指 令 ， 我 们 就 能 找 出 IOCTL 处 理 代 码 CONSTANT，也就是 IOCTLDispatch，接着顺藤摸瓜找出 IOCTL 代码。driverlib 的具体实 现如下：

```py
def getIOCTLDispatch( self ):
    search_pattern = "MOV DWORD PTR [R32+70],CONST"
    dispatch_address = self.imm.searchCommandsOnModule( self.module
    .getCodebase(), search_pattern )
    # We have to weed out some possible bad matches 
    for address in dispatch_address:
        instruction = self.imm.disasm( address[0] )
        if "MOV DWORD PTR" in instruction.getResult(): 
            if "+70" in instruction.getResult():
            self.IOCTLDispatchFunctionAddress = instruction.getImmConst() self.IOCTLDispatchFunction =
            self.imm.getFunction( self.IOCTLDispatchFunctio
            break
    # return a Function object if successful 
    return self.IOCTLDispatchFunction 
```

Listing 10-4: 找出 IOCTL dispatch function 的方法

最新的 Immunity 中还有另一种列举函数搜索的方法，不过原理都一样。一旦我们找到 了合适的函数，就将这个函数对象返回，在后面 IOCTL 代码查找中将会用它。

下面来看看 IOCTL dispatch 的函数是如何实现的，以及如何查找出所有的 IOCTL 代码。

### 10.3.3 找出 IOCTL 代码

IOCTL dispatch 根据传入的值(也就是 IOCTL 代码)执行相应的操作。这也是我们千方 百计要找出所有 IOCTL 的原因，因为 IOCTL 就相当于用户模式下你调用的"函数"。让我们 先看一段用 C 实现的 IOCTL dispatch，之后我们反汇编它们，并从中找出 IOCTL 代码。

```py
NTSTATUS IOCTLDispatch( IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp )
{
    ULONG FunctionCode; 
    PIO_STACK_LOCATION IrpSp;
    // Setup code to get the request initialized
    IrpSp = IoGetCurrentIrpStackLocation(Irp); 
    FunctionCode = IrpSp->Parameters.DeviceIoControl.IoControlCode;
    // Once the IOCTL code has been determined, perform a
    // specific action
    switch(FunctionCode)
    {
        case 0x1337:
            // ... Perform action A case 0x1338:
            // ... Perform action B case 0x1339:
            // ... Perform action C
    }
    Irp->IoStatus.Status = STATUS_SUCCESS; 
    IoCompleteRequest( Irp, IO_NO_INCREMENT ); 
    return STATUS_SUCCESS;
} 
```

Listing 10-5: 一 段 简 单 的 IOCTL dispatch 代 码 支 持 三 种 IOCTL 代 码 (0x13370x1338, 0x1339)

当函数从 IOCTL 请求中检索到 IOCTL 代码的时候，就将代码传递个 switch{}语句，然 后根据 IOCTL 代码执行相应的操作。switch 语句在汇编之后有可能是以下两种形式。

```py
// Series of CMP statements against a constant
CMP DWORD PTR SS:[EBP-48], 1339 # Test for 0x1339
JE 0xSOMEADDRESS # Jump to 0x1339 action
CMP DWORD PTR SS:[EBP-48], 1338 # Test for 0x1338 JE 0xSOMEADDRESS
CMP DWORD PTR SS:[EBP-48], 1337 # Test for 0x1337 JE 0xSOMEADDRESS
// Series of SUB instructions decrementing the IOCTL code
MOV ESI, DWORD PTR DS:[ESI + C] # Store the IOCTL code in ESI
SUB ESI, 1337 # Test for 0x1337
JE 0xSOMEADDRESS # Jump to 0x1337 action SUB ESI, 1 # Test for 0x1338
JE 0xSOMEADDRESS # Jump to 0x1338 action
SUB ESI, 1 # Test for 0x1339
JE 0xSOMEADDRESS # Jump to 0x1339 action 
```

Listing 10-6: 两种不同的 switch{}反汇编指令

switch{} 的反汇编指令有很多种，不过最常见的就是上面两种。在第一种情况下，我 们可以通过一些列的 CMP 指令，找到进行比较的常量，这些就是 IOCTL 代码。第二种情 况，稍微复杂点，它由一系列的 SUB 指令接条件跳转实现。关键的一行如下：

```py
SUB ESI, 1337 
```

这一行告诉了我们，最小的 IOCTL 代码就是 0x1337。从这里开始，0x1337 作为第一个 常量，每行 SUB 指令减去多少，我能就加上多少，每次加出来的新的值作为一个新的 IOCTL 代码。不断累加，直到 switch 结束。具体实现可以看 Immunity 目录下的 Libs\driverlib.py。 代码自动化的找出了 IOCTL dispatch 和所有的 IOCTL codes。

现在 driverlib 为我们完成了最脏最累的活。接下来让我们做些高雅的事！用 driverlib 捕捉驱动程序中所有的设备名和 IOCTL 代码，并且将结果保存到 Python pickle 中。接着用 它们构建 IOCTL fuzzer。Let’s get fuzzy！

# 10.4 构建 Driver Fuzzer

## 10.4 构建 Driver Fuzzer

第一步在完成 PyCommand:IOCTL-dump。

```py
#ioctl_dump.py 
import pickle 
import driverlib
from immlib import * 
def main( args ):
    ioctl_list = [] device_list = []
    imm = Debugger() 
    driver = driverlib.Driver()
    # Grab the list of IOCTL codes and device names 
    ioctl_list = driver.getIOCTLCodes()
    if not len(ioctl_list):
        return "[*] ERROR! Couldn't find any IOCTL codes." 
    device_list = driver.getDeviceNames()
    if not len(device_list):
        return "[*] ERROR! Couldn't find any device names."
    # Now create a keyed dictionary and pickle it to a file 
    master_list = {}
    master_list["ioctl_list"] = ioctl_list 
    master_list["device_list"] = device_list
    filename = "%s.fuzz" % imm.getDebuggedName() 
    fd = open( filename, "wb" )
    pickle.dump( master_list, fd ) 
    fd.close()
    return "[*] SUCCESS! Saved IOCTL codes and device names to %s" % filename 
```

这个 PyCommand 相当简单：检索 IOCTL 代码列表，检索设备名列表，将他们存到字 典中，然后保存到文件里。下次我们只要在 Immunity 的命令行中简单的输入 !ioctl_dump， pickle 文件就会保存到 Immunity 目录下。

万事俱备只欠 fuzzer。接下来就是 coding and coding，我们实现的这个 fuzzer 检测范围 限制在内存错误和缓冲区溢出，不过扩展也是很容易的。

```py
#my_ioctl_fuzzer.py
import pickle 
import sys 
import random
from ctypes import * 
kernel32 = windll.kernel32
# Defines for Win32 API Calls 
GENERIC_READ = 0x80000000 
GENERIC_WRITE = 0x40000000 
OPEN_EXISTING = 0x3
# Open the pickle and retrieve the dictionary 
fd = open(sys.argv[1], "rb") 
master_list = pickle.load(fd)
ioctl_list = master_list["ioctl_list"] 
device_list = master_list["device_list"] 
fd.close()
# Now test that we can retrieve valid handles to all
# device names, any that don't pass we remove from our test cases 
valid_devices = [] 
for device_name in device_list:
    # Make sure the device is accessed properly
    device_file = u"\\\\.\\%s" % device_name.split("\\")[::-1][0] 
    print "[*] Testing for device: %s" % device_file
    driver_handle = kernel32.CreateFileW(device_file,GENERIC_READGENERIC_WRITE,0,None,OPEN_EXISTI NG,0,None)
    if driver_handle:
        print "[*] Success! %s is a valid device!" 
        if device_file not in valid_devices:
            valid_devices.append( device_file )
        kernel32.CloseHandle( driver_handle )
    else:
        print "[*] Failed! %s NOT a valid device."
if not len(valid_devices):
    print "[*] No valid devices found. Exiting..." 
    sys.exit(0)
# Now let's begin feeding the driver test cases until we can't be
# it anymore! CTRL-C to exit the loop and stop fuzzing 
while 1:
    # Open the log file first
    fd = open("my_ioctl_fuzzer.log","a")
    # Pick a random device name 
    current_device = valid_devices[random.randint(0, len(valid_devices)-1 )]
    fd.write("[*] Fuzzing: %s\n" % current_device)
    # Pick a random IOCTL code 
    current_ioctl = ioctl_list[random.randint(0, len(ioctl_list)-1)]
    fd.write("[*] With IOCTL: 0x%08x\n" % current_ioctl)
    # Choose a random length current_length = random.randint(0, 10000) fd.write("[*] Buffer length: %d\n" % current_length)
    # Let's test with a buffer of repeating As
    # Feel free to create your own test cases here 
    in_buffer = "A" * current_length
    # Give the IOCTL run an out_buffer
    out_buf = (c_char * current_length)() 
    bytes_returned = c_ulong(current_length)
    # Obtain a handle
    driver_handle = kernel32.CreateFileW(device_file, GENERIC_READ|GENERIC_WRITE,0,None,OPEN_EXISTING,0,None)
    fd.write("!!FUZZ!!\n")
    # Run the test case
    kernel32.DeviceIoControl( driver_handle, current_ioctl, in_buffer, current_length, byref(out_buf), current_length, byref(bytes_returned), None )
    fd.write( "[*] Test case finished. %d bytes returned.\n\n" % bytes_returned.value )
    # Close the handle and carry on! 
    kernel32.CloseHandle( driver_handle ) 
    fd.close() 
```

先从 pickle 文件中取出包含 IOCTL 代码和设备名的字典。从列表中找出能够获得句柄 的设备名。如果无法获取，就从列表中移除。接着随机选取一个设备名和 IOCTL 代码，创 建一个随机长度的缓冲区。最后将 IOCTL 发送给驱动。

使用如下命令进行 fuzzing。

```py
C:\>python.exe my_ioctl_fuzzer.py i2omgmt.sys.fuzz 
```

如果 fuzzer crash 了机器，我们能够很准确的获得发送的 IOCTL 代码。接着就是调试驱 动了。表 10-7 显示的就是一个未知驱动的 fuzzing 过程。

```py
[*] Fuzzing: \\.\unnamed
[*] With IOCTL: 0x84002019
[*] Buffer length: 3277
!!FUZZ!!
[*] Test case finished. 3277 bytes returned. 
[*] Fuzzing: \\.\unnamed
[*] With IOCTL: 0x84002020
[*] Buffer length: 2137
!!FUZZ!!
[*] Test case finished. 1 bytes returned. 
[*] Fuzzing: \\.\unnamed
[*] With IOCTL: 0x84002016
[*] Buffer length: 1097
!!FUZZ!!
[*] Test case finished. 1097 bytes returned. 
[*] Fuzzing: \\.\unnamed
[*] With IOCTL: 0x8400201c
[*] Buffer length: 9366
!!FUZZ!! 
```

Listing 10-7: 一次成功的 fuzzing 记录

能够很清楚的看到，上一个 IOCTL，0x8400201c 引发了系统崩溃，因为这是最后一条 记录。目前为止我们的 fuzzer 很简单，但是很漂亮，可以通过不断的扩展功能，使它更强大。 其中一个可能的方法就是，将 InBufferLength 或者 OutBufferLength 参数设置成和实际传入的数据长度不一样。开始毁灭之路吧 ，哈哈！！ destroy all drivers in your path!