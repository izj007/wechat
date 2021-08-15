>Author：Snowming@A-Team



**如何判断一个恶意攻击载荷会不会被 AMSI 机制拦截?**


已知 Windows 10 的 office VBA macros 组件中集成了AMSI 功能，所以在本文中，我尝试使用 Windbg 调试器分析 AMSI 机制对一个我自制的 VBA 恶意样本的检测过程，最终、通过 AMSI 机制核心检测 API 的返回结果进行分析，我得出结论：此样本绕过了 AMSI 机制。

# 0x01 概念

[Antimalware Scan Interface (AMSI)](https://docs.microsoft.com/en-us/windows/win32/amsi/antimalware-scan-interface-portal)


从2012年 Powershell  攻击利用工具成型化以来，互联网中的通过无文件与 Powershell 的攻击事件数量逐年递增，无文件攻击可以绕过那些基于签名和文件检测的传统安全软件。微软在2015年中旬开始提出了针对无文件攻击和 Powershell 脚本攻击的检测缓解方案- AMSI。AMSI  是一个与供应商无关的安全接口，用于为 Microsoft Defender 等杀毒软件提供 PowerShell 脚本检测接口。脚本在运行时，AMSI 将对脚本的代码进行检测，并通过接口传递给系统内的杀毒软件。如杀毒软件发现代码存在恶意特征，将中止（`block`）脚本的执行，并将结果反馈给用户。

AMSI 与基于杀软虚拟沙盒的动态行为分析（行为沙盒）一定程度类似。行为沙盒对 API 链以及传入参数通过启发式算法对程序的恶意性进行评估，以 `TrojanDownloader/Upatre` 家族的样本 （SHA1 `a3c20b864669b19407d56e240281d88ca10c1c7f`）为例，参数层面分析了复制的文件名、InternetOpen 的 Agent 名称、试图访问的网址等关键数据：

![title](https://leanote.com/api/file/getImage?fileId=5f202af2ab64411a9f001bf5)


AMSI 也是在应用启动的时候检测可执行文件，并扫描启动后可能会打开的后续资源文件。AMSI 的两个核心扫描 function：

- AmsiScanBuffer
- AmsiScanString

扫描内容缓冲区和字符串以查找恶意软件，也是对 API 及其传入参数进行分析，根据结果作评分以判断是否应该 block。

AMSI 主要针对 Windows 系统上的一些脚本攻击，比如 powershell 无文件攻击、VBScript脚本，对脚本进行扫描。AMSI一些可能失效的地方比如：从WMI名字空间、注册表、事件日志等非常规位置加载的脚本、不用 powershell.exe 执行(可用网络策略服务器之类的工具)的 PowerShell 脚本等。



### 最低支持 OS 版本

- PC：Windows 10 [desktop apps only]
- 服务器：Windows Server 2016 [desktop apps only]



### 扫描范围


- 落地的文件
- 内存（fileless malware）
- 流
- content source URL/IP reputation checks


### 与 AMSI 集成的 Windows 组件

AMSI 功能已集成到 Windows 10 的这些组件中：

- User Account Control, or UAC (elevation of EXE, COM, MSI, or ActiveX installation)
- PowerShell (scripts, interactive use, and dynamic code evaluation)
- Windows Script Host (wscript.exe and cscript.exe)
- JavaScript and VBScript
- Office VBA macros

既然 `Office VBA macros` 集成了 AMSI，这提醒我们在制作 VBA 宏样本的过程中，机器最好断网，避免上传。


### AMSI 实现范例（伪代码）

```
#include <amsi.h>

const S_OK = 0x00000000 

typedef enum AMSI_ATTRIBUTE {
  AMSI_ATTRIBUTE_APP_NAME,
  AMSI_ATTRIBUTE_CONTENT_NAME,
  AMSI_ATTRIBUTE_CONTENT_SIZE,
  AMSI_ATTRIBUTE_CONTENT_ADDRESS,
  AMSI_ATTRIBUTE_SESSION,
  AMSI_ATTRIBUTE_REDIRECT_CHAIN_SIZE,
  AMSI_ATTRIBUTE_REDIRECT_CHAIN_ADDRESS,
  AMSI_ATTRIBUTE_ALL_SIZE,
  AMSI_ATTRIBUTE_ALL_ADDRESS,
  AMSI_ATTRIBUTE_QUIET
} ;
typedef enum AMSI_RESULT {
  AMSI_RESULT_CLEAN,
  AMSI_RESULT_NOT_DETECTED,
  AMSI_RESULT_BLOCKED_BY_ADMIN_START,
  AMSI_RESULT_BLOCKED_BY_ADMIN_END,
  AMSI_RESULT_DETECTED
} ;

long result, result1, string_result, buffer_result;
Handle *scanned_string;
Handle *scanned_buffer;
long buffer_length;

//AMSI 初始化功能
Handle *amsiContext = HAMSICONTEXT amsiContext
Handle *amsiSession = HAMSICONTEXT *amsiSession
result = AmsiInitialize("homemade_AV", amsiContext);
if(result != S_OK){
    exit(HRESULT);   //error code
}                  

//打开一个会话，可以在其中关联多个扫描请求。
result1 = AmsiOpenSession(amsiContext,amsiSession );
if(result != S_OK){
    exit(HRESULT);   //error code
}    

//扫描字符串以查找恶意软件
string_result = AmsiScanString(amsiContext, scanned_string, "C:\Users\LuckyDog\Desktop\love.docm", amsiSession, AMSI_RESULT)
if(result != S_OK){
    exit(HRESULT);   //error code
}    

// 通过会话关联多种扫描
// Scans a buffer-full of content for malware. 适用于无文件的情况
buffer_result = AmsiScanBuffer(amsiContext, scanned_buffer, buffer_length, "C:\Users\LuckyDog\Desktop\love.docm", amsiSession, AMSI_RESULT);
if(result != S_OK){
    exit(HRESULT);   //error code
}    

//确定扫描结果是否指示应阻止内容
//AMSI_RESULT是返回的枚举，一个常量
AmsiResultIsMalware(AMSI_RESULT);

//调用打开会话 API 之后一定要关闭会话
AmsiCloseSession(amsiContext, amisiSession);

//调用初始化 API 之后一定要调用 Uninitialize API
AmsiUninitialize(amsiContext);
```

>杀软固定测试文件：
[eicar](https://www.eicar.org/?page_id=3950)


# 0x02 Windbg 调试 AMSI 对 VBA 宏的检测

`WINWORD.exe` 是多个文档共享的进程，打开含有 VBA 宏的 `.docm` 文档，发现的确载入了 `amsi.dll`。

![title](https://leanote.com/api/file/getImage?fileId=5f1ebee0ab64411a9f000b36)

但是不清楚 amsi.dll 到底对 VBA 宏做了何种检测。从 API 的层面，是仅仅 `AmsiInitialize` 了？还是做了扫描？又扫描了哪些内容？所以需要使用调试器对此过程进行调试。


需要注意的是，打开 `WORD` 进程之后，很快就 load 了 `amsi.dll`，所以需要 `Ctrl`+`E` 打开 WORD 进程附加到 Windbg 中，这样的方式来调试 docm 文档。


打开 word 进程之后，快速使用以下命令对 amsi.dll 下断点，以使得在加载 amsi.dll 之前下断：
```
sxe ld:amsi
```

使用 CFF Explorer 查看 `amsi.dll` 的导出函数：

![title](https://leanote.com/api/file/getImage?fileId=5f1ec108ab64411c9a000b4e)

批量给导出函数下断点：

```
0:010> x amsi!Amsi*
00007ff8`efb921e0 amsi!AmsiInitialize (<no parameter info>)
00007ff8`efb92460 amsi!AmsiUninitialize (<no parameter info>)
00007ff8`efb924c0 amsi!AmsiOpenSession (<no parameter info>)
00007ff8`efb92520 amsi!AmsiCloseSession (<no parameter info>)
00007ff8`efb92540 amsi!AmsiScanBuffer (<no parameter info>)
00007ff8`efb92640 amsi!AmsiScanString (<no parameter info>)
00007ff8`efb926a0 amsi!AmsiUacInitialize (<no parameter info>)
00007ff8`efb928c0 amsi!AmsiUacUninitialize (<no parameter info>)
00007ff8`efb92920 amsi!AmsiUacScan (<no parameter info>)

0:010> bm amsi!Amsi*
breakpoint 1 redefined
  1: 00007ff8`efb921e0 @!"amsi!AmsiInitialize"
breakpoint 2 redefined
  2: 00007ff8`efb92460 @!"amsi!AmsiUninitialize"
breakpoint 3 redefined
  3: 00007ff8`efb924c0 @!"amsi!AmsiOpenSession"
breakpoint 0 redefined
  0: 00007ff8`efb92520 @!"amsi!AmsiCloseSession"
breakpoint 4 redefined
  4: 00007ff8`efb92540 @!"amsi!AmsiScanBuffer"
 12: 00007ff8`efb92640 @!"amsi!AmsiScanString"
 13: 00007ff8`efb926a0 @!"amsi!AmsiUacInitialize"
 14: 00007ff8`efb928c0 @!"amsi!AmsiUacUninitialize"
 15: 00007ff8`efb92920 @!"amsi!AmsiUacScan"
 
0:010> bl
     0 e Disable Clear  00007ff8`efb92520     0001 (0001)  0:**** amsi!AmsiCloseSession
     1 e Disable Clear  00007ff8`efb921e0     0001 (0001)  0:**** amsi!AmsiInitialize
     2 e Disable Clear  00007ff8`efb92460     0001 (0001)  0:**** amsi!AmsiUninitialize
     3 e Disable Clear  00007ff8`efb924c0     0001 (0001)  0:**** amsi!AmsiOpenSession
     4 e Disable Clear  00007ff8`efb92540     0001 (0001)  0:**** amsi!AmsiScanBuffer
    12 e Disable Clear  00007ff8`efb92640     0001 (0001)  0:**** amsi!AmsiScanString
    13 e Disable Clear  00007ff8`efb926a0     0001 (0001)  0:**** amsi!AmsiUacInitialize
    14 e Disable Clear  00007ff8`efb928c0     0001 (0001)  0:**** amsi!AmsiUacUninitialize
    15 e Disable Clear  00007ff8`efb92920     0001 (0001)  0:**** amsi!AmsiUacScan

```



```
0:010> bm amsi!Dll*
 10: 00007ff8`efb91860 @!"amsi!DllCanUnloadNow"
 11: 00007ff8`efb91890 @!"amsi!DllGetClassObject"
 12: 00007ff8`efb919c0 @!"amsi!DllRegisterServer"
breakpoint 12 redefined
 12: 00007ff8`efb919c0 @!"amsi!DllUnregisterServer"
0:010> bl
     0 e Disable Clear  00007ff8`efb92520     0001 (0001)  0:**** amsi!AmsiCloseSession
     1 e Disable Clear  00007ff8`efb921e0     0001 (0001)  0:**** amsi!AmsiInitialize
     2 e Disable Clear  00007ff8`efb92460     0001 (0001)  0:**** amsi!AmsiUninitialize
     3 e Disable Clear  00007ff8`efb924c0     0001 (0001)  0:**** amsi!AmsiOpenSession
     4 e Disable Clear  00007ff8`efb92540     0001 (0001)  0:**** amsi!AmsiScanBuffer
     6 e Disable Clear  00007ff8`efb92640     0001 (0001)  0:**** amsi!AmsiScanString
     7 e Disable Clear  00007ff8`efb926a0     0001 (0001)  0:**** amsi!AmsiUacInitialize
     8 e Disable Clear  00007ff8`efb928c0     0001 (0001)  0:**** amsi!AmsiUacUninitialize
     9 e Disable Clear  00007ff8`efb92920     0001 (0001)  0:**** amsi!AmsiUacScan
    10 e Disable Clear  00007ff8`efb91860     0001 (0001)  0:**** amsi!DllCanUnloadNow
    11 e Disable Clear  00007ff8`efb91890     0001 (0001)  0:**** amsi!DllGetClassObject
    12 e Disable Clear  00007ff8`efb919c0     0001 (0001)  0:**** amsi!DllUnregisterServer
```

其中，此两个函数的 `Function RVA` 是一样的，所以只断在一个函数上。
```
0:010> u amsi!DllRegisterServer
amsi!DllUnregisterServer:
00007ff8`efb919c0 b832000780      mov     eax,80070032h
00007ff8`efb919c5 c3              ret
00007ff8`efb919c6 cc              int     3
00007ff8`efb919c7 cc              int     3
00007ff8`efb919c8 cc              int     3
00007ff8`efb919c9 cc              int     3
00007ff8`efb919ca cc              int     3
00007ff8`efb919cb cc              int     3
```

如此，就给 amsi.dll 的所有导出函数下了断点：

![title](https://leanote.com/api/file/getImage?fileId=5f1ec4d6ab64411a9f000b78)







>注：
`bp`、`bu`、`bm` 中，只有 `bm` 能加通配符（此处用了模糊匹配通配符 `*`）



然后开始运行，发现命中了初始化函数，也就是触发了 AMSI 初始化方法：

```
0:010> g
Breakpoint 1 hit
amsi!AmsiInitialize:
00007ff8`efb921e0 48895c2410      mov     qword ptr [rsp+10h],rbx ss:0000003f`f77fdb68=0000000000000000
```

`amsi!DllGetClassObject` 和 `amsi!DllRegisterServer` 在 MSDN 中没有文档化，但是这两个函数都是 COM 的入口点，用于方便的实例化一个 COM 对象。AMSI 的扫描功能似乎是通过自己的 COM 服务器来实现的，当 COM 服务器被实例化时会被暴露出来。当 AMSI 加载时，它实例化其自有的 COM 组件，并暴露诸如 `amsi!AmsiOpenSession`、`amsi!AmsiScanBuffer`、`amsi!AmsiScanString` 和 `amsi!AmsiCloseSession` 之类的方法。


```
0:000> g
Breakpoint 11 hit
amsi!DllGetClassObject:
00007ff8`efb91890 488bc4          mov     rax,rsp
0:000> g
ModLoad: 00007ff8`eb590000 00007ff8`eb623000   C:\Windows\System32\appresolver.dll
ModLoad: 00007ff9`03060000 00007ff9`03087000   C:\Windows\System32\SLC.dll
ModLoad: 00007ff8`e8160000 00007ff8`e816d000   C:\Windows\SYSTEM32\LINKINFO.dll
ModLoad: 00007ff9`005c0000 00007ff9`00646000   C:\Windows\SYSTEM32\policymanager.dll
ModLoad: 00007ff9`00d20000 00007ff9`00daa000   C:\Windows\SYSTEM32\msvcp110_win.dll
ModLoad: 00007ff8`e8470000 00007ff8`e84ef000   C:\Windows\SYSTEM32\ntshrui.dll
ModLoad: 00007ff8`f5ac0000 00007ff8`f5ae6000   C:\Windows\SYSTEM32\srvcli.dll
ModLoad: 00007ff8`f0040000 00007ff8`f0052000   C:\Windows\SYSTEM32\cscapi.dll
(a8c.1ef4): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
KERNEL32!IsBadStringPtrA+0x1f:
00007ff9`05c6519f 8a01            mov     al,byte ptr [rcx] ds:00000000`00000002=??
0:000> k
 # Child-SP          RetAddr               Call Site
00 0000003f`f6b9a620 00007ff8`be1a42d3     KERNEL32!IsBadStringPtrA+0x1f
01 0000003f`f6b9a640 00007ff8`be0fd2ad     VBE7!rtcSetDatabaseLcid+0x6d6f
02 0000003f`f6b9a730 000001c3`eda7543f     VBE7!rtcBeep+0x736f1
03 0000003f`f6b9a7c0 00007ff8`bdf30e3b     0x000001c3`eda7543f
04 0000003f`f6b9a7f0 00007ff8`bdf30de7     VBE7!rtcGetCurrentCalendar+0x1b5b
05 0000003f`f6b9a950 00007ff9`071af3af     VBE7!rtcGetCurrentCalendar+0x1b07
06 0000003f`f6b9aab0 00007ff9`071990d6     OLEAUT32!SetErrorInfo+0xc6f
07 0000003f`f6b9aaf0 00007ff8`bdf8bcfe     OLEAUT32!DispCallFunc+0x226
08 0000003f`f6b9abb0 00007ff8`bdf8d75e     VBE7!VarPtr+0x54492
09 0000003f`f6b9ac00 00007ff8`be01f4a3     VBE7!VarPtr+0x55ef2
0a 0000003f`f6b9bd90 00007ff8`be01f711     VBE7!DllVbeInit+0x130f3
0b 0000003f`f6b9be00 00007ff8`b9e73137     VBE7!DllVbeInit+0x13361
0c 0000003f`f6b9bee0 00007ff8`b9e72220     wwlib!DllCanUnloadNow+0x1ca9b7
0d 0000003f`f6b9bf30 00007ff8`b9d82d30     wwlib!DllCanUnloadNow+0x1c9aa0
0e 0000003f`f6b9bf80 00007ff8`b8f621aa     wwlib!DllCanUnloadNow+0xda5b0
0f 0000003f`f6b9c1d0 00007ff8`b8f5810a     wwlib!FMain+0x1efa
10 0000003f`f6b9c3e0 00007ff8`b902cfe0     wwlib!DllMain+0x11aba
11 0000003f`f6b9c500 00007ff8`b932bd3b     wwlib!DllGetClassObject+0x91b70
12 0000003f`f6b9c540 00007ff8`b932b46c     wwlib!DllGetClassObject+0x3908cb
13 0000003f`f6b9d700 00007ff8`b645679f     wwlib!DllGetClassObject+0x38fffc
14 0000003f`f6b9e980 00007ff8`b6456589     mso98win32client!Ordinal3038+0x16bf
15 0000003f`f6b9eac0 00007ff8`b6456470     mso98win32client!Ordinal3038+0x14a9
16 0000003f`f6b9ed50 00007ff8`bce1cd40     mso98win32client!Ordinal3038+0x1390
17 0000003f`f6b9ee80 00007ff8`bceb47d9     mso20win32client!Ordinal1756+0x90
18 0000003f`f6b9ef20 00007ff8`bce1cd40     mso20win32client!Ordinal519+0xb9
19 0000003f`f6b9ef80 00007ff8`bce1c5bc     mso20win32client!Ordinal1756+0x90
1a 0000003f`f6b9f020 00007ff8`bcef4799     mso20win32client!Ordinal1482+0xcfc
1b 0000003f`f6b9f0a0 00007ff8`b62f6e6a     mso20win32client!Ordinal1380+0x2b9
1c 0000003f`f6b9f0d0 00007ff8`b82bcfca     mso98win32client!Ordinal454+0x1da
1d 0000003f`f6b9f130 00007ff8`b82bcd86     mso30win32client!Ordinal1025+0x42a
1e 0000003f`f6b9f200 00007ff8`b82bcb62     mso30win32client!Ordinal1025+0x1e6
1f 0000003f`f6b9f2a0 00007ff8`b634079e     mso30win32client!Ordinal1156+0x32
20 0000003f`f6b9f2d0 00007ff8`b8d8520b     mso98win32client!Ordinal891+0x13e
21 0000003f`f6b9f310 00007ff8`b8d3a4be     wwlib+0x6520b
22 0000003f`f6b9f360 00007ff8`b8d3a9ad     wwlib+0x1a4be
23 0000003f`f6b9f3b0 00007ff8`b8dc20b4     wwlib+0x1a9ad
24 0000003f`f6b9f650 00007ff8`b8d39f4f     wwlib!WordMailReact::StringCommand::~StringCommand+0xa0f4
25 0000003f`f6b9f6e0 00007ff8`b8f6031d     wwlib+0x19f4f
26 0000003f`f6b9f770 00007ff7`47f61143     wwlib!FMain+0x6d
27 0000003f`f6b9f7a0 00007ff7`47f61412     winword+0x1143
28 0000003f`f6b9fa40 00007ff9`05c47bd4     winword+0x1412
29 0000003f`f6b9fa80 00007ff9`0730ce51     KERNEL32!BaseThreadInitThunk+0x14
2a 0000003f`f6b9fab0 00000000`00000000     ntdll!RtlUserThreadStart+0x21
0:000> g
(a8c.1ef4): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
KERNEL32!IsBadStringPtrA+0x1f:
00007ff9`05c6519f 8a01            mov     al,byte ptr [rcx] ds:00000000`00001124=??
0:000> g
(a8c.1ef4): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
KERNEL32!IsBadStringPtrA+0x1f:
00007ff9`05c6519f 8a01            mov     al,byte ptr [rcx] ds:00000000`00001124=??
0:000> g
(a8c.1ef4): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
KERNEL32!IsBadStringPtrA+0x1f:
00007ff9`05c6519f 8a01            mov     al,byte ptr [rcx] ds:00000000`00001124=??
0:000> g
(a8c.1ef4): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
KERNEL32!IsBadStringPtrA+0x1f:
00007ff9`05c6519f 8a01            mov     al,byte ptr [rcx] ds:00000000`00001124=??
0:000> g
(a8c.1ef4): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
KERNEL32!IsBadStringPtrA+0x1f:
00007ff9`05c6519f 8a01            mov     al,byte ptr [rcx] ds:00000000`00001124=??
0:000> g
(a8c.1ef4): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
KERNEL32!IsBadStringPtrA+0x1f:
00007ff9`05c6519f 8a01            mov     al,byte ptr [rcx] ds:00000000`00001124=??
```


反复报错误码为 `c0000005` 的错误，在 Windbg 中忽略这个错误：
```
0:000> sxd c0000005
```

然后继续运行：

```
0:000> g
(a8c.1ef4): Access violation - code c0000005 (first chance)
(a8c.1ef4): Access violation - code c0000005 (first chance)
...
(a8c.1ef4): Access violation - code c0000005 (first chance)
(a8c.1ef4): Access violation - code c0000005 (first chance)
```

发现命中了扫描方法，这个扫描方法是针对字符串的：

```
Breakpoint 6 hit
amsi!AmsiScanString:
00007ff8`efb92640 4883ec38        sub     rsp,38h
```

>注：
[k、kb、kc、kd、kp、kP、kv（显示堆栈回溯）](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/debugger/k--kb--kc--kd--kp--kp--kv--display-stack-backtrace-)

使用 `kb` 命令查看传递到堆栈跟踪中的每个函数的前三个参数：

```
0:000> kb
 # RetAddr               : Args to Child                                                           : Call Site
00 00007ff8`be1a3a21     : 000001c3`e8b539f6 0000003f`f6b9a598 0000003f`f6b9a5b0 00007ff9`072dfb91 : amsi!AmsiScanString
01 00007ff8`be1a3cac     : 0000003f`f6b9a610 00007ff8`00000012 00007ff8`be2e8540 000001c3`00000065 : VBE7!rtcSetDatabaseLcid+0x64bd
02 00007ff8`be1a447c     : 000001c3`eda742f0 000001c3`00000012 00007ff8`be2e8540 00000000`00000065 : VBE7!rtcSetDatabaseLcid+0x6748
03 00007ff8`be0fd2ad     : 000001c3`eda74310 000001c3`e87c0005 000001c3`ed9be348 0000003f`f6b9a760 : VBE7!rtcSetDatabaseLcid+0x6f18
04 000001c3`eda75683     : 000001c3`eda74310 000001c3`ed9be408 00000000`00000000 000001c3`ed9be420 : VBE7!rtcBeep+0x736f1
05 00007ff8`bdf2d69c     : 00007ff8`bdf20000 00007ff8`bdf29ac0 0000003f`f6b9a940 000001c3`e8761372 : 0x000001c3`eda75683
06 00007ff8`bdf2f17f     : 00007ff8`be3001f0 00000000`00000000 00000000`00000000 00000000`00000400 : VBE7!Ordinal984+0xd69c
07 00007ff9`071af3af     : 00007ff8`be3001f0 00000000`00000000 00000000`00000008 00000000`00000400 : VBE7!rtcSendKeys+0x177
08 00007ff9`071990d6     : 0000003f`f6b90001 000001c3`e8a93680 00000000`00000001 00000000`00000000 : OLEAUT32!SetErrorInfo+0xc6f
09 00007ff8`bdf8bcfe     : 000001c3`e8a929a4 000001c3`e8bf1218 00000000`00000000 000001c3`dbb30b70 : OLEAUT32!DispCallFunc+0x226
0a 00007ff8`bdf8d75e     : 0000003f`f6b9ae20 000001c3`eda74af8 000001c3`e6090001 0000003f`f6b9be50 : VBE7!VarPtr+0x54492
0b 00007ff8`be01f4a3     : 00000000`00000000 000001c3`eda74af8 00000000`60000003 00000000`00000001 : VBE7!VarPtr+0x55ef2
0c 00007ff8`be01f711     : 000001c3`e8b37ec0 0000003f`60000003 00007ff8`be2bfa80 000001c3`00000800 : VBE7!DllVbeInit+0x130f3
0d 00007ff8`b9e73137     : 00000000`60000003 ffffffff`00000001 00007ff8`bdfd6102 000001c3`e8b37ec0 : VBE7!DllVbeInit+0x13361
0e 00007ff8`b9e72220     : 00000000`00000000 000001c3`e8a12f60 000001c3`e8a12f60 00007ff8`b93783e3 : wwlib!DllCanUnloadNow+0x1ca9b7
0f 00007ff8`b9d82d30     : 000001c3`e609d2f8 00000000`00000038 0000003f`f6b9c2e0 00000000`00000000 : wwlib!DllCanUnloadNow+0x1c9aa0
10 00007ff8`b8f621aa     : 00000000`00000000 00000000`00000001 00000000`00000000 00000000`00000000 : wwlib!DllCanUnloadNow+0xda5b0
11 00007ff8`b8f5810a     : 000001c3`000001c2 000001c3`e8d5b400 000001c3`e8a100ff 00000000`000000ff : wwlib!FMain+0x1efa
12 00007ff8`b902cfe0     : 00004f79`b6ae1557 00007ff8`b8ea5cd2 00000000`00000000 0000003f`f6b9c640 : wwlib!DllMain+0x11aba
13 00007ff8`b932bd3b     : 00000000`00000000 0000003f`f6b9d748 00000000`ffffffff 00000000`00000000 : wwlib!DllGetClassObject+0x91b70
14 00007ff8`b932b46c     : 00000000`00000824 0000003f`f6b9d800 00000000`00000000 00007ff8`ba992b48 : wwlib!DllGetClassObject+0x3908cb
15 00007ff8`b645679f     : 00000000`00000000 0000003f`f6b9ebc0 000001c3`e8826890 000001c3`e8826890 : wwlib!DllGetClassObject+0x38fffc
16 00007ff8`b6456589     : 000001c3`000004fb 00000000`00000001 00000000`00000001 0000003f`f6b9eb20 : mso98win32client!Ordinal3038+0x16bf
17 00007ff8`b6456470     : 0000003f`00001ef4 000001c3`ed8e9600 00000000`00000001 000001c3`db908810 : mso98win32client!Ordinal3038+0x14a9
18 00007ff8`bce1cd40     : 00000000`00000000 000001c3`ed37b0c0 00000000`00000000 000001c3`db908810 : mso98win32client!Ordinal3038+0x1390
19 00007ff8`bceb47d9     : 000001c3`ed37b0a0 00007ff8`bce1c8ea 000001c3`ed37b0a0 00007ff8`bcddfd3e : mso20win32client!Ordinal1756+0x90
1a 00007ff8`bce1cd40     : 00000000`00000000 00007ff8`bce1c7f9 00000000`00000000 0000003f`f6b9f000 : mso20win32client!Ordinal519+0xb9
1b 00007ff8`bce1c5bc     : 00000000`00000014 00000000`00000000 00000000`00000014 000001c3`db90acc8 : mso20win32client!Ordinal1756+0x90
1c 00007ff8`bcef4799     : 000001c3`e89d37f0 000001c3`db908638 00000000`00000001 000001c3`db908810 : mso20win32client!Ordinal1482+0xcfc
1d 00007ff8`b62f6e6a     : 000001c3`db939c10 00000000`00000000 ffffffff`fffffffe 00000000`00000000 : mso20win32client!Ordinal1380+0x2b9
1e 00007ff8`b82bcfca     : 000001c3`db90aad0 00000000`00000000 00000000`00000000 00000000`00000001 : mso98win32client!Ordinal454+0x1da
1f 00007ff8`b82bcd86     : 000001c3`db90aab0 000001c3`db90aad0 000001c3`e879ec30 000001c3`e879ec30 : mso30win32client!Ordinal1025+0x42a
20 00007ff8`b82bcb62     : 00000000`00001ef4 000001c3`db90aab0 00000000`00000000 00000000`00000001 : mso30win32client!Ordinal1025+0x1e6
21 00007ff8`b634079e     : ffffffff`00001ef4 00007ff8`b8e7a589 ffffffff`fffffffe 00000301`005c1256 : mso30win32client!Ordinal1156+0x32
22 00007ff8`b8d8520b     : 00007ff8`ba7f0401 00007ff8`b8d85187 00000000`00000001 00007ff8`ba7f0400 : mso98win32client!Ordinal891+0x13e
23 00007ff8`b8d3a4be     : 00007ff8`ba7f0478 00007ff8`ba7f0478 00007ff8`ba7f0400 00007ff8`ba7f0400 : wwlib+0x6520b
24 00007ff8`b8d3a9ad     : 00000000`00000ab0 0000003f`f6b9f4b0 00007ff8`ba7f0478 0000003f`f6b9f484 : wwlib+0x1a4be
25 00007ff8`b8dc20b4     : 00000000`00000100 00000000`00000000 00000000`00000000 00000000`00000000 : wwlib+0x1a9ad
26 00007ff8`b8d39f4f     : 00000000`00000100 00000000`00000000 00000000`00000100 00000000`00000000 : wwlib!WordMailReact::StringCommand::~StringCommand+0xa0f4
27 00007ff8`b8f6031d     : 00007ff8`b8d20000 00000000`00000000 00007ff7`47f60000 00000000`00000000 : wwlib+0x19f4f
28 00007ff7`47f61143     : 00007ff8`b8d20000 00007ff8`b8f602b0 000001c3`d9a335cd 00000000`00000000 : wwlib!FMain+0x6d
29 00007ff7`47f61412     : 00000000`0000000a 00000000`00000000 00000000`00000000 00000000`00000000 : winword+0x1143
2a 00007ff9`05c47bd4     : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : winword+0x1412
2b 00007ff9`0730ce51     : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : KERNEL32!BaseThreadInitThunk+0x14
2c 00000000`00000000     : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!RtlUserThreadStart+0x21
```

![title](https://leanote.com/api/file/getImage?fileId=5f1ecc5dab64411c9a000bdf)


查看此函数，我们主要想看 `string` 参数，也就是扫描的字符串。所以查看第二个参数 `rdx` 寄存器：

>注：
[x64 calling convention](https://docs.microsoft.com/en-us/cpp/build/x64-calling-convention?view=vs-2019)



```
0:000> r rdx
rdx=000001c3ed2ce0f0
0:000> dq rdx
000001c3`ed2ce0f0  006b000a`003b0029 0065006e`00720065
000001c3`ed2ce100  002e0032`0033006c 0063006f`00720050
000001c3`ed2ce110  00330073`00730065 00780065`004e0032
000001c3`ed2ce120  00300028`00570074 00300030`00300030
000001c3`ed2ce130  00300030`00300030 00310030`00300030
000001c3`ed2ce140  002c0034`00320031 00300030`00300030
000001c3`ed2ce150  00330063`00310030 00620039`00640065
000001c3`ed2ce160  00300030`00320065 006b000a`003b0029
0:000> db rdx
000001c3`ed2ce0f0  29 00 3b 00 0a 00 6b 00-65 00 72 00 6e 00 65 00  ).;...k.e.r.n.e.
000001c3`ed2ce100  6c 00 33 00 32 00 2e 00-50 00 72 00 6f 00 63 00  l.3.2...P.r.o.c.
000001c3`ed2ce110  65 00 73 00 73 00 33 00-32 00 4e 00 65 00 78 00  e.s.s.3.2.N.e.x.
000001c3`ed2ce120  74 00 57 00 28 00 30 00-30 00 30 00 30 00 30 00  t.W.(.0.0.0.0.0.
000001c3`ed2ce130  30 00 30 00 30 00 30 00-30 00 30 00 30 00 31 00  0.0.0.0.0.0.0.1.
000001c3`ed2ce140  31 00 32 00 34 00 2c 00-30 00 30 00 30 00 30 00  1.2.4.,.0.0.0.0.
000001c3`ed2ce150  30 00 31 00 63 00 33 00-65 00 64 00 39 00 62 00  0.1.c.3.e.d.9.b.
000001c3`ed2ce160  65 00 32 00 30 00 30 00-29 00 3b 00 0a 00 6b 00  e.2.0.0.).;...k.
0:000> du rdx
000001c3`ed2ce0f0  ");.kernel32.Process32NextW(00000"
000001c3`ed2ce130  "00000001124,000001c3ed9be200);.k"
000001c3`ed2ce170  "ernel32.Process32NextW(000000000"
000001c3`ed2ce1b0  "0001124,000001c3ed9be200);.kerne"
000001c3`ed2ce1f0  "l32.Process32NextW(0000000000001"
000001c3`ed2ce230  "124,000001c3ed9be200);.kernel32."
000001c3`ed2ce270  "Process32NextW(0000000000001124,"
000001c3`ed2ce2b0  "000001c3ed9be200);.kernel32.Proc"
000001c3`ed2ce2f0  "ess32NextW(0000000000001124,0000"
000001c3`ed2ce330  "01c3ed9be200);.kernel32.Process3"
000001c3`ed2ce370  "2NextW(0000000000001124,000001c3"
000001c3`ed2ce3b0  "ed9be200);.kernel32.Process32Nex"
```

字符串格式查看内存具体内容：

>注：
[.printf](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-printf)
Windbg 中的 `.printf` 跟 C 中差不多，用于格式化输出字符串：

```
0:000> .writemem c:\Users\LuckyDog\Desktop\yyy.dmp 0x000001c3ed2ce0f0 0x000001c3ed9be348
Writing 6f0259 bytes...............................................................
Unable to read memory at 000001c3`ed2ed8f0, file is incomplete
windbg> .hh .printf

)0:000> .printf /D /od "%mu",rdx
);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.OpenProcess(00000000001fffff,0000000000000000,00000000000009e4);
kernel32.VirtualAllocEx(0000000000001260,0000000000000000,000000000000037d,0000000000001000,0000000000000040);
kernel32.WriteProcessMemory(0000000000001260,0000000002c40000,000001c3ed6427a0,000000000000037d,0000000000000000);
```

可以看到，这是一种`日志式`的扫描方式，也就是先执行多条，然后把日志发给杀软（AMSI 始终只是一个接口而已）。

[Office VBA + AMSI：揭开恶意宏的面纱](https://www.microsoft.com/security/blog/2018/09/12/office-vba-amsi-parting-the-veil-on-malicious-macros/)

![title](https://leanote.com/api/file/getImage?fileId=5f1ecff1ab64411c9a000c06)

在上面的记录中，AMSI 的 scan 函数扫描了我的 VBA 中的调用的 API，至于传入参数是地址，具体杀软对于这些传入参数的处理方式尚未可知。

继续运行，还是调用了 `amsi!AmsiScanString` 方法：

```
0:000> g
ModLoad: 00007ff8`f5fa0000 00007ff8`f60a6000   C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2006.10-0\MPCLIENT.DLL
ModLoad: 00007ff9`02d70000 00007ff9`02d92000   C:\Windows\SYSTEM32\gpapi.dll
ModLoad: 00007ff8`e7d20000 00007ff8`e7eb6000   C:\Windows\System32\TaskFlowDataEngine.dll
ModLoad: 00007ff8`f6730000 00007ff8`f6c75000   C:\Windows\System32\cdp.dll
ModLoad: 00007ff9`00db0000 00007ff9`00e7f000   C:\Windows\System32\dsreg.dll
ModLoad: 00007ff8`eb410000 00007ff8`eb46b000   C:\Program Files\Microsoft Office\root\Office16\IEAWSDC.DLL
ModLoad: 00007ff9`057b0000 00007ff9`05c20000   C:\Windows\System32\SETUPAPI.dll
(a8c.1ef4): Access violation - code c0000005 (first chance)
(a8c.1ef4): Access violation - code c0000005 (first chance)
Breakpoint 6 hit
amsi!AmsiScanString:
00007ff8`efb92640 4883ec38        sub     rsp,38h
0:000> kb
 # RetAddr               : Args to Child                                                           : Call Site
00 00007ff8`be1a3a21     : 000001c3`e8b539f6 0000003f`f6b9a588 0000003f`f6b9a5a0 00007ff9`072dfb91 : amsi!AmsiScanString
01 00007ff8`be1a3cac     : 0000003f`f6b9a600 00007ff8`00000012 00007ff8`be2e8540 000001c3`00000065 : VBE7!rtcSetDatabaseLcid+0x64bd
02 00007ff8`be1a447c     : 000001c3`eda757a0 000001c3`00000012 00007ff8`be2e8540 00000000`00000065 : VBE7!rtcSetDatabaseLcid+0x6748
03 00007ff8`be0fd2ad     : 000001c3`eda757c0 000001c3`e87c0007 000001c3`ed9be338 0000003f`f6b9a750 : VBE7!rtcSetDatabaseLcid+0x6f18
04 000001c3`eda758c3     : 000001c3`eda757c0 000001c3`eda756e6 00007ff8`bdf2a0f8 00000000`02c40000 : VBE7!rtcBeep+0x736f1
05 00007ff8`bdf30e3b     : 00000000`00000004 000001c3`e87613c4 0000003f`f6b9a940 000001c3`e8761372 : 0x000001c3`eda758c3
06 00007ff8`bdf2f17f     : 00007ff8`be3001f0 00000000`00000000 00000000`00000000 00000000`00000400 : VBE7!rtcGetCurrentCalendar+0x1b5b
07 00007ff9`071af3af     : 00007ff8`be3001f0 00000000`00000000 00000000`00000008 00000000`00000400 : VBE7!rtcSendKeys+0x177
08 00007ff9`071990d6     : 0000003f`f6b90001 000001c3`e8a93680 00000000`00000001 00000000`00000000 : OLEAUT32!SetErrorInfo+0xc6f
09 00007ff8`bdf8bcfe     : 000001c3`e8a929a4 000001c3`e8bf1218 00000000`00000000 000001c3`dbb30b70 : OLEAUT32!DispCallFunc+0x226
0a 00007ff8`bdf8d75e     : 0000003f`f6b9ae20 000001c3`eda74af8 000001c3`e6090001 0000003f`f6b9be50 : VBE7!VarPtr+0x54492
0b 00007ff8`be01f4a3     : 00000000`00000000 000001c3`eda74af8 00000000`60000003 00000000`00000001 : VBE7!VarPtr+0x55ef2
0c 00007ff8`be01f711     : 000001c3`e8b37ec0 0000003f`60000003 00007ff8`be2bfa80 000001c3`00000800 : VBE7!DllVbeInit+0x130f3
0d 00007ff8`b9e73137     : 00000000`60000003 ffffffff`00000001 00007ff8`bdfd6102 000001c3`e8b37ec0 : VBE7!DllVbeInit+0x13361
0e 00007ff8`b9e72220     : 00000000`00000000 000001c3`e8a12f60 000001c3`e8a12f60 00007ff8`b93783e3 : wwlib!DllCanUnloadNow+0x1ca9b7
0f 00007ff8`b9d82d30     : 000001c3`e609d2f8 00000000`00000038 0000003f`f6b9c2e0 00000000`00000000 : wwlib!DllCanUnloadNow+0x1c9aa0
10 00007ff8`b8f621aa     : 00000000`00000000 00000000`00000001 00000000`00000000 00000000`00000000 : wwlib!DllCanUnloadNow+0xda5b0
11 00007ff8`b8f5810a     : 000001c3`000001c2 000001c3`e8d5b400 000001c3`e8a100ff 00000000`000000ff : wwlib!FMain+0x1efa
12 00007ff8`b902cfe0     : 00004f79`b6ae1557 00007ff8`b8ea5cd2 00000000`00000000 0000003f`f6b9c640 : wwlib!DllMain+0x11aba
13 00007ff8`b932bd3b     : 00000000`00000000 0000003f`f6b9d748 00000000`ffffffff 00000000`00000000 : wwlib!DllGetClassObject+0x91b70
14 00007ff8`b932b46c     : 00000000`00000824 0000003f`f6b9d800 00000000`00000000 00007ff8`ba992b48 : wwlib!DllGetClassObject+0x3908cb
15 00007ff8`b645679f     : 00000000`00000000 0000003f`f6b9ebc0 000001c3`e8826890 000001c3`e8826890 : wwlib!DllGetClassObject+0x38fffc
16 00007ff8`b6456589     : 000001c3`000004fb 00000000`00000001 00000000`00000001 0000003f`f6b9eb20 : mso98win32client!Ordinal3038+0x16bf
17 00007ff8`b6456470     : 0000003f`00001ef4 000001c3`ed8e9600 00000000`00000001 000001c3`db908810 : mso98win32client!Ordinal3038+0x14a9
18 00007ff8`bce1cd40     : 00000000`00000000 000001c3`ed37b0c0 00000000`00000000 000001c3`db908810 : mso98win32client!Ordinal3038+0x1390
19 00007ff8`bceb47d9     : 000001c3`ed37b0a0 00007ff8`bce1c8ea 000001c3`ed37b0a0 00007ff8`bcddfd3e : mso20win32client!Ordinal1756+0x90
1a 00007ff8`bce1cd40     : 00000000`00000000 00007ff8`bce1c7f9 00000000`00000000 0000003f`f6b9f000 : mso20win32client!Ordinal519+0xb9
1b 00007ff8`bce1c5bc     : 00000000`00000014 00000000`00000000 00000000`00000014 000001c3`db90acc8 : mso20win32client!Ordinal1756+0x90
1c 00007ff8`bcef4799     : 000001c3`e89d37f0 000001c3`db908638 00000000`00000001 000001c3`db908810 : mso20win32client!Ordinal1482+0xcfc
1d 00007ff8`b62f6e6a     : 000001c3`db939c10 00000000`00000000 ffffffff`fffffffe 00000000`00000000 : mso20win32client!Ordinal1380+0x2b9
1e 00007ff8`b82bcfca     : 000001c3`db90aad0 00000000`00000000 00000000`00000000 00000000`00000001 : mso98win32client!Ordinal454+0x1da
1f 00007ff8`b82bcd86     : 000001c3`db90aab0 000001c3`db90aad0 000001c3`e879ec30 000001c3`e879ec30 : mso30win32client!Ordinal1025+0x42a
20 00007ff8`b82bcb62     : 00000000`00001ef4 000001c3`db90aab0 00000000`00000000 00000000`00000001 : mso30win32client!Ordinal1025+0x1e6
21 00007ff8`b634079e     : ffffffff`00001ef4 00007ff8`b8e7a589 ffffffff`fffffffe 00000301`005c1256 : mso30win32client!Ordinal1156+0x32
22 00007ff8`b8d8520b     : 00007ff8`ba7f0401 00007ff8`b8d85187 00000000`00000001 00007ff8`ba7f0400 : mso98win32client!Ordinal891+0x13e
23 00007ff8`b8d3a4be     : 00007ff8`ba7f0478 00007ff8`ba7f0478 00007ff8`ba7f0400 00007ff8`ba7f0400 : wwlib+0x6520b
24 00007ff8`b8d3a9ad     : 00000000`00000ab0 0000003f`f6b9f4b0 00007ff8`ba7f0478 0000003f`f6b9f484 : wwlib+0x1a4be
25 00007ff8`b8dc20b4     : 00000000`00000100 00000000`00000000 00000000`00000000 00000000`00000000 : wwlib+0x1a9ad
26 00007ff8`b8d39f4f     : 00000000`00000100 00000000`00000000 00000000`00000100 00000000`00000000 : wwlib!WordMailReact::StringCommand::~StringCommand+0xa0f4
27 00007ff8`b8f6031d     : 00007ff8`b8d20000 00000000`00000000 00007ff7`47f60000 00000000`00000000 : wwlib+0x19f4f
28 00007ff7`47f61143     : 00007ff8`b8d20000 00007ff8`b8f602b0 000001c3`d9a335cd 00000000`00000000 : wwlib!FMain+0x6d
29 00007ff7`47f61412     : 00000000`0000000a 00000000`00000000 00000000`00000000 00000000`00000000 : winword+0x1143
2a 00007ff9`05c47bd4     : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : winword+0x1412
2b 00007ff9`0730ce51     : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : KERNEL32!BaseThreadInitThunk+0x14
2c 00000000`00000000     : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!RtlUserThreadStart+0x21
0:000> r rdx
rdx=000001c3ed2dc160
0:000> .printf /D /od "%mu",rdx
);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.OpenProcess(00000000001fffff,0000000000000000,00000000000009e4);
kernel32.VirtualAllocEx(0000000000001260,0000000000000000,000000000000037d,0000000000001000,0000000000000040);
kernel32.WriteProcessMemory(0000000000001260,0000000002c40000,000001c3ed6427a0,000000000000037d,0000000000000000);
kernel32.CreateRemoteThread(0000000000001260,0000000000000000,0000000000000000,0000000002c40000,,0000000000000000,);
```

这次的内存内容比上次显示多扫描了一个 `kernel32.CreateRemoteThread` API，这是我的 VBA 宏中最后调用的一个执行 shellcode 的 API。

继续运行，发现调用了 `amsi!AmsiScanBuffer` 扫描方法，同样查看其内存内容：


![title](https://leanote.com/api/file/getImage?fileId=5f1ed155ab64411a9f000c04)


注：第二个参数是 `buffer`，这是扫描的缓冲区内容。

```
0:000> g
Breakpoint 4 hit
amsi!AmsiScanBuffer:
00007ff8`efb92540 4c8bdc          mov     r11,rsp
0:000> .printf /D /od "%mu",rdx
);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.Process32NextW(0000000000001124,000001c3ed9be200);
kernel32.OpenProcess(00000000001fffff,0000000000000000,00000000000009e4);
kernel32.VirtualAllocEx(0000000000001260,0000000000000000,000000000000037d,0000000000001000,0000000000000040);
kernel32.WriteProcessMemory(0000000000001260,0000000002c40000,000001c3ed6427a0,000000000000037d,0000000000000000);
kernel32.CreateRemoteThread(0000000000001260,0000000000000000,0000000000000000,0000000002c40000,,0000000000000000,);
```


对于此扫描方法，我还想查看其第4个参数：

![title](https://leanote.com/api/file/getImage?fileId=5f1ed1aaab64411a9f000c08)


也就是寄存器 `r9`，这个参数是文件名：



```
0:000> .printf /D /od "%mu",r9
C:\Users\LuckyDog\Desktop\love.docm
```
看上去没毛病。

但是如果我想查看其第6个参数，也就是返回值，却不正确：

>注：
x64程序参数对应的寄存器：
参数1： `rcx`
参数2： `rdx`
参数3： `r8`
参数4： `r9`
参数5： `rsp+20`
参数6： `rsp+28`
具体参考：[x64 calling convention](https://docs.microsoft.com/en-us/cpp/build/x64-calling-convention?view=vs-2019)

```
0000003f`f6b9a5300:000> db rsp+28
0000003f`f6b9a530  00 00 00 00 00 00 00 00-00 a6 b9 f6 3f 00 00 00  ............?...
0000003f`f6b9a540  d0 a5 b9 f6 3f 00 00 00-21 3a 1a be f8 7f 00 00  ....?...!:......
0000003f`f6b9a550  f6 39 b5 e8 c3 01 00 00-88 a5 b9 f6 3f 00 00 00  .9..........?...
0000003f`f6b9a560  a0 a5 b9 f6 3f 00 00 00-91 fb 2d 07 f9 7f 00 00  ....?.....-.....
0000003f`f6b9a570  00 a6 b9 f6 3f 00 00 00-00 00 a3 d9 c3 01 00 00  ....?...........
0000003f`f6b9a580  d0 a5 b9 f6 3f 00 00 00-20 1c 00 00 f8 7f 00 00  ....?... .......
0000003f`f6b9a590  f6 39 b5 e8 c3 01 00 00-a0 57 a7 ed c3 01 00 00  .9.......W......
0000003f`f6b9a5a0  60 c1 2d ed c3 01 00 00-0d 18 f2 bd f8 7f 00 00  `.-.............
0:000> r rsp+28
           ^ Syntax error in 'r rsp+28'
0:000> db rsp+28
0000003f`f6b9a530  00 00 00 00 00 00 00 00-00 a6 b9 f6 3f 00 00 00  ............?...
0000003f`f6b9a540  d0 a5 b9 f6 3f 00 00 00-21 3a 1a be f8 7f 00 00  ....?...!:......
0000003f`f6b9a550  f6 39 b5 e8 c3 01 00 00-88 a5 b9 f6 3f 00 00 00  .9..........?...
0000003f`f6b9a560  a0 a5 b9 f6 3f 00 00 00-91 fb 2d 07 f9 7f 00 00  ....?.....-.....
0000003f`f6b9a570  00 a6 b9 f6 3f 00 00 00-00 00 a3 d9 c3 01 00 00  ....?...........
0000003f`f6b9a580  d0 a5 b9 f6 3f 00 00 00-20 1c 00 00 f8 7f 00 00  ....?... .......
0000003f`f6b9a590  f6 39 b5 e8 c3 01 00 00-a0 57 a7 ed c3 01 00 00  .9.......W......
0000003f`f6b9a5a0  60 c1 2d ed c3 01 00 00-0d 18 f2 bd f8 7f 00 00  `.-.............
0:000> kb rsp+0x28
Requested number of stack frames (0x3ff6b9a530) is too large! The maximum number is 0xffff.
                 ^ Range error in 'kb rsp+0x28'
0:000> kb rsp+28
Requested number of stack frames (0x3ff6b9a530) is too large! The maximum number is 0xffff.
               ^ Range error in 'kb rsp+28'
0:000> db rsp+28
0000003f`f6b9a530  00 00 00 00 00 00 00 00-00 a6 b9 f6 3f 00 00 00  ............?...
0000003f`f6b9a540  d0 a5 b9 f6 3f 00 00 00-21 3a 1a be f8 7f 00 00  ....?...!:......
0000003f`f6b9a550  f6 39 b5 e8 c3 01 00 00-88 a5 b9 f6 3f 00 00 00  .9..........?...
0000003f`f6b9a560  a0 a5 b9 f6 3f 00 00 00-91 fb 2d 07 f9 7f 00 00  ....?.....-.....
0000003f`f6b9a570  00 a6 b9 f6 3f 00 00 00-00 00 a3 d9 c3 01 00 00  ....?...........
0000003f`f6b9a580  d0 a5 b9 f6 3f 00 00 00-20 1c 00 00 f8 7f 00 00  ....?... .......
0000003f`f6b9a590  f6 39 b5 e8 c3 01 00 00-a0 57 a7 ed c3 01 00 00  .9.......W......
0000003f`f6b9a5a0  60 c1 2d ed c3 01 00 00-0d 18 f2 bd f8 7f 00 00  `.-.............
0:000> db rsp+0x28
0000003f`f6b9a530  00 00 00 00 00 00 00 00-00 a6 b9 f6 3f 00 00 00  ............?...
0000003f`f6b9a540  d0 a5 b9 f6 3f 00 00 00-21 3a 1a be f8 7f 00 00  ....?...!:......
0000003f`f6b9a550  f6 39 b5 e8 c3 01 00 00-88 a5 b9 f6 3f 00 00 00  .9..........?...
0000003f`f6b9a560  a0 a5 b9 f6 3f 00 00 00-91 fb 2d 07 f9 7f 00 00  ....?.....-.....
0000003f`f6b9a570  00 a6 b9 f6 3f 00 00 00-00 00 a3 d9 c3 01 00 00  ....?...........
0000003f`f6b9a580  d0 a5 b9 f6 3f 00 00 00-20 1c 00 00 f8 7f 00 00  ....?... .......
0000003f`f6b9a590  f6 39 b5 e8 c3 01 00 00-a0 57 a7 ed c3 01 00 00  .9.......W......
0000003f`f6b9a5a0  60 c1 2d ed c3 01 00 00-0d 18 f2 bd f8 7f 00 00  `.-.............
```


这是因为，第6个参数是方法的返回结果，具体格式是一个枚举，所以需要先 `step out` 此函数。

>注：
`step over` 仅仅往前走1步，获取函数返回结果需要 `step out` 跳出方法之后才能获取返回结果。注意区分函数的返回结果和返回值。

![title](https://leanote.com/api/file/getImage?fileId=5f1ed2a1ab64411c9a000c23)

![title](https://leanote.com/api/file/getImage?fileId=5f1ed2bbab64411c9a000c24)


注意在使用 `gu` 命令前，要先将地址（这里是 0000003f\`f6b9a530）保存到一个变量：

```
0:000> p
amsi!AmsiScanBuffer+0x3:
00007ff8`efb92543 49895b08        mov     qword ptr [r11+8],rbx ds:0000003f`f6b9a510=08a6913c00000020
0:000> db rsp+28
0000003f`f6b9a530  00 00 00 00 00 00 00 00-00 a6 b9 f6 3f 00 00 00  ............?...
0000003f`f6b9a540  d0 a5 b9 f6 3f 00 00 00-21 3a 1a be f8 7f 00 00  ....?...!:......
0000003f`f6b9a550  86 8e 9d ed c3 01 00 00-88 a5 b9 f6 3f 00 00 00  ............?...
0000003f`f6b9a560  a0 a5 b9 f6 3f 00 00 00-91 fb 2d 07 f9 7f 00 00  ....?.....-.....
0000003f`f6b9a570  00 a6 b9 f6 3f 00 00 00-00 00 a3 d9 c3 01 00 00  ....?...........
0000003f`f6b9a580  d0 a5 b9 f6 3f 00 00 00-20 1c 00 00 f8 7f 00 00  ....?... .......
0000003f`f6b9a590  86 8e 9d ed c3 01 00 00-70 08 b5 ed c3 01 00 00  ........p.......
0000003f`f6b9a5a0  40 49 fb f5 c3 01 00 00-0d 18 f2 bd f8 7f 00 00  @I..............

0:000> r @$t1=@rsp+28
0:000> ?@$t1
Evaluate expression: 274722301232 = 0000003f`f6b9a530
0:000> gu
amsi!AmsiScanString+0x47:
00007ff8`efb92687 eb05            jmp     amsi!AmsiScanString+0x4e (00007ff8`efb9268e)
0:000> db @$t1
0000003f`f6b9a530  00 00 00 00 00 00 00 00-00 a6 b9 f6 3f 00 00 00  ............?...
0000003f`f6b9a540  d0 a5 b9 f6 3f 00 00 00-21 3a 1a be f8 7f 00 00  ....?...!:......
0000003f`f6b9a550  86 8e 9d ed c3 01 00 00-88 a5 b9 f6 3f 00 00 00  ............?...
0000003f`f6b9a560  a0 a5 b9 f6 3f 00 00 00-91 fb 2d 07 f9 7f 00 00  ....?.....-.....
0000003f`f6b9a570  00 a6 b9 f6 3f 00 00 00-00 00 a3 d9 c3 01 00 00  ....?...........
0000003f`f6b9a580  d0 a5 b9 f6 3f 00 00 00-20 1c 00 00 f8 7f 00 00  ....?... .......
0000003f`f6b9a590  86 8e 9d ed c3 01 00 00-70 08 b5 ed c3 01 00 00  ........p.......
0000003f`f6b9a5a0  40 49 fb f5 c3 01 00 00-0d 18 f2 bd f8 7f 00 00  @I..............
0:000> dd @$t1
0000003f`f6b9a530  00000000 00000000 f6b9a600 0000003f
0000003f`f6b9a540  f6b9a5d0 0000003f be1a3a21 00007ff8
0000003f`f6b9a550  ed9d8e86 000001c3 f6b9a588 0000003f
0000003f`f6b9a560  f6b9a5a0 0000003f 072dfb91 00007ff9
0000003f`f6b9a570  f6b9a600 0000003f d9a30000 000001c3
0000003f`f6b9a580  f6b9a5d0 0000003f 00001c20 00007ff8
0000003f`f6b9a590  ed9d8e86 000001c3 edb50870 000001c3
0000003f`f6b9a5a0  f5fb4940 000001c3 bdf2180d 00007ff8
```

然后这个结果：地址 0000003f\`f6b9a530 处的 Long（8字节）`00000000 00000000` 说明结果为 `AMSI_RESULT_CLEAN`。


![title](https://leanote.com/api/file/getImage?fileId=5f1ed496ab64411c9a000cda)

![title](https://leanote.com/api/file/getImage?fileId=5f1ed54fab64411a9f000ccd)

说明此样本通过了 `AMSI` 的检测！


继续执行，发现执行完毕，检测完成。

> 注：其实后面应该有 调用 `AmsiUninitialize(amsiContext)` 方法的，在一次调试中看到了。

```
0:000> g
(a8c.1ef4): Access violation - code c0000005 (first chance)
ModLoad: 00007ff8`fe860000 00007ff8`fefce000   C:\Windows\System32\OneCoreUAPCommonProxyStub.dll
ModLoad: 00007ff8`eb590000 00007ff8`eb623000   C:\Windows\System32\appresolver.dll
ModLoad: 00007ff9`03060000 00007ff9`03087000   C:\Windows\System32\SLC.dll
ModLoad: 00007ff8`edad0000 00007ff8`edb49000   C:\Windows\System32\OneCoreCommonProxyStub.dll
ModLoad: 00007ff8`e7d20000 00007ff8`e7eb6000   C:\Windows\System32\TaskFlowDataEngine.dll
ModLoad: 00007ff8`f6730000 00007ff8`f6c75000   C:\Windows\System32\cdp.dll
ModLoad: 00007ff9`00db0000 00007ff9`00e7f000   C:\Windows\System32\dsreg.dll
ModLoad: 00007ff8`f6f00000 00007ff8`f6f89000   C:\Windows\SYSTEM32\WINSPOOL.DRV
(a8c.1ef4): C++ EH exception - code e06d7363 (first chance)
(a8c.1ef4): C++ EH exception - code e06d7363 (first chance)

```

自此调试完毕。


>更进一步：
可以尝试在 Windbg 中修改此 `AMSI_RESULT enumeration` 的结果，看 Windows Defender 是否报查杀。
[AMSI_RESULT enumeration](https://docs.microsoft.com/en-us/windows/win32/api/amsi/ne-amsi-amsi_result)

#  0x03 总结

到目前为止，我对 AMSI 的理解是：

AMSI 只是一个接口，在不同的杀软中对其有具体的实现，此机制对样本进行扫描，主要是以下两种函数：

- AmsiScanBuffer
- AmsiScanString

然后将多条扫描结果以日志的形式传给杀软。那么之后的处理，还是不同的杀软有不同的实现机制。其本身会有一个是否 block 的 result。


AMSI 在 Win10 中就是一个 dll。

Windows Defender 的 AMSI 在物理机和虚拟机中，扫描大多没有差别。但是 360 等杀软可能针对物理机和虚拟机有不同的实现。这种情况下，与其说绕过 AMSI，最终还是落到绕过具体的杀软。




---------


参考文档：


1. [Office VBA + AMSI：揭开恶意宏的面纱](https://www.microsoft.com/security/blog/2018/09/12/office-vba-amsi-parting-the-veil-on-malicious-macros/)