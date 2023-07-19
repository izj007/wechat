#  怎么在无EXECUTE权限的内存上执行shellcode

原创 wonderkun  [ 安全的矛与盾 ](javascript:void\(0\);)

**安全的矛与盾** ![]()

微信号 gh_b4c853063b88

功能介绍 知攻、知守、知进退；有矛、有盾、有安全。

____

___发表于_

收录于合集

## 前言

各种shellcode loader总是避免不掉一个问题，申请具备EXECUTE权限的内存来执行shellcode。这是shellcode
loader永远的痛，也是检测的重灾区，怎么避免这个内存区域被内存扫描盯上，也是内存免杀的一部分重要工作。

如果你问我，在windows最新版本的系统上，没有EXECUTE权限的内存页上是否可以执行代码？我可以很明确的告诉你不可以的。但是如果你再问我一遍，我会再回答你：事情可能也没那么绝对，其实也不是完全不行。

我在知识星球《攻防智库》中就发了一个引导性问题，不过没有人参与讨论，只能我在此自问自答了。

内存执行保护的历史

没有执行权限的内存页上的代码不可执行，这个事情并不是生来就理所当然的。而是在`Microsoft Windows (XP SP2, 2003, and
Vista)`之后新增的特性，其目的是为了防止堆或者栈上的恶意代码执行，其真实的名字叫 DEP( Data Execution Prevention
)。如果在受保护的页面上执行代码，就会抛出 `STATUS_ACCESS_VIOLATION` 异常。

相关的详细介绍可以参考如下文章：

  * • https://blog.csdn.net/qq_55202378/article/details/127685185

  * • https://www.pythonstudio.us/reverse-engineering/bypassing-dep-on-windows.html

  * • https://fahrishih.medium.com/bypassing-windows-dep-data-execution-prevention-d1a276a973f1

操作系统的DEP策略可以使用api `GetSystemDEPPolicy()` 来获取，可以取如下几种值:

![]()

在windows server上是比较严格的 `OptOut` ，在其他客户端版本上是`OptIn`

使用`bcdedit` 命令就可以列出当前的DEP策略：

![]()

知道DEP是怎么回事后，我们看如下问题：

    
    
    int main()  
    {  
         u_char shellcode[] = {  
              0x90,0x90,0x90,0xcc,  
              0x90,0x90,0x90,0xcc,  
              0x90,0x90,0x90,0xcc,  
              0x90,0x90,0x90,0xcc,  
              0x90,0x90,0x90,0xcc,  
          };  
          LPVOID memory = VirtualAlloc(NULL, sizeof(shellcode), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);  
              memcpy(memory, shellcode, sizeof(shellcode));  
          ((void(*)())memory)();  
    }

怎么能让这段代码正常执行呢？在普通的windows客户端版本上，(dep策略是OptIn)的机器上,编译为32位，然后去除NX标志位，就可以直接执行了。

![]()

但是在windows server上(DEP策略是OptOut)却并不行，下面就开始解决这个问题。

## 32位程序在OptOut策略上关闭DEP的n种方法

### 方法一:

调用ntdll一个没有公开的api `ZwSetInformationProcess`,这也是之前x86上漏洞利用绕过DEP的常用方法，具体代码如下：

    
    
    typedef  NTSTATUS  (NTAPI *  PZwSetInformationProcess)(  
        _In_ HANDLE  ProcessHandle,  
        _In_ PROCESSINFOCLASS  ProcessInformationClass,  
        _In_ PVOID  ProcessInformation,  
        _In_ ULONG  ProcessInformationLength  
    );  
      
    //BOOL status = SetProcessDEPPolicy(0);  
    PZwSetInformationProcess ZwSetInformationProcess = (PZwSetInformationProcess)GetProcAddress(GetModuleHandleA("NTDLL.DLL"), "ZwSetInformationProcess");  
    INT ProcessInformation = 0x2;  
    NTSTATUS status = ZwSetInformationProcess(  
        GetCurrentProcess(),  
        ProcessExecuteFlags, // 0x22  
        &ProcessInformation,  
        0x4  
    );

注意一定是`NTAPI` 的调用约定，要不然会崩溃。

### 方法二:

调用windows提供的一个api， `SetProcessDEPPolicy`:

![]()

    
    
    BOOL status = SetProcessDEPPolicy(0);

### 方法三:

方法三就要讨论到我们提供的那个样本了，样本执行起来之后发现它的DEP是disable的。

![]()

怎么做到的呢？其实只需要两步：

  1. 1. 去掉.textsection的execute权限

![]()

  2. 2. 去掉NX保护

那问题来了，操作系统是怎么关闭了 `DEP`的呢？查看ntdll的，可以发现如下代码：

![]()![]()

调用`ZwSetInformationProcess`的条件有如下三个任意一个满足就可以,感觉ida反编译的结果应该是错误的，就直接看汇编了：

  1. 1. _LdrpCheckForSecuROMImage() & 0xFF != 0

  2. 2\. _LdrpEntrySectionValid() & 0xFF == 0代码如下：

![]()

entry_section没有执行权限就返回0了，跟预期一样

  3. 3\. _LdrpCheckForSafeDiscImage() & 0xFF !=0 代码如下：

![]()

大概意思是匹配到字符串 `BoG_ *90.0&!! Yy>`
就关闭DEP，或者在前9个section中出现名字为`stxt371`、`.txt`或者`.txt2`的section。剩下的方法四、五、六留给大家自己去验证吧。

## 64位程序关闭DEP的方法

想啥呢？咋可能有。

## 结尾

其实32位程序关闭dep也是一种假关闭，看函数`SetProcessDEPPolicy`也知道仅仅是在模拟器层面上关闭了限制，真实的内存依然是无法执行代码的，这是CPU硬件支持的特性。

虽然64位的不可执行内存不能执行代码，但是有的时候内存被关注的太严格的时候，临时用一下32位也是一种选择。

# 安全的矛与盾

我们的知识星球"安全的矛与盾"是一个既讲攻击也讲防御，开放的、前沿的安全技术分享社区。在这里你不仅可以学习到最新的攻击方法与逃避检测的技术，也可以学到最全面的安全防御体系，了解入侵检测、攻击防护系统的原理与实践。站在攻与防不同的视角看问题，提高自己对安全的理解和深度，做到:
**知攻、知守、知进退；有矛、有盾、有安全。** ****

 **更多的干货内容，更深入的技术交流，尽在知识星球“安全的矛与盾”，欢迎大家扫码** **加入！** **有问题请咨询微信
**Manliness_man![]()

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

