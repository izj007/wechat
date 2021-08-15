# 0x01 代码


使用 `CreateProcessW` 函数，写一段简单的调用代码，想通过调试这个文件把握与进程创建相关的 API 调用流程。

下面的代码中，创建的是一个 `notepad.exe` 进程。要注意此 API 的坑点在于第一个参数需要为 NULL，写起来更简单一点。


```
//createprocess.cpp

#include <stdio.h>
#include <windows.h>
#include "corecrt_wstring.h"

void main()
{
    //结构体
    STARTUPINFO si;
    PROCESS_INFORMATION pi;

    ZeroMemory(&si, sizeof(si));
    ZeroMemory(&pi, sizeof(pi));

    //结构的大小
    si.cb = sizeof(STARTUPINFO);

    //设置lpCommandLine
    // #define max_path 260 filetype:h
    TCHAR szCmd[MAX_PATH] = { 0, };
    wcscpy_s(szCmd, L"notepad.exe");

    //返回值为 BOOL，TRUE 表示进程创建成功。!TRUE 表示创建失败
    if (!CreateProcessW(
        NULL,                   //lpApplicationName
        szCmd,                  //lpCommandLine
        NULL,                   //lpProcessAttributes
        NULL,                   //lpThreadAttributes
        FALSE,                  //bInheritHandles
        NORMAL_PRIORITY_CLASS,  //dwCreationFlags
        NULL,                   //lpEnvironment
        NULL,                   //lpCurrentDirectory
        &si,                    //lpStartupInfo
        &pi                    //lpProcessInformation
        )) {
        return;
    }
        
    //返回结果在 lpProcessInformation 结构体的 hProcess 成员中，是一个进程句柄
    if (pi.hProcess != NULL)
    {
        CloseHandle(pi.hProcess);
        //还需要关闭主线程句柄。因为同时生成进程及主线程
        CloseHandle(pi.hThread);
    }
}
```


关于 `szCmd` 参数路径的说明：


![title](https://leanote.com/api/file/getImage?fileId=5f1fda8eab64411a9f001807)




编译得到可执行文件。

# 0x02 调试

地址 `009D1842` 处是调用 `kernel32!CreateProcessW` 的指令。

![title](https://leanote.com/api/file/getImage?fileId=5f1fdb95ab64411a9f001822)

WinDBG 调试：

发现 `kernel32!CreateProcessW` 函数内部又调用了 `KERNELBASE!CreateProcessInternalW`。


```

Microsoft (R) Windows Debugger Version 10.0.20153.1000 X86
Copyright (c) Microsoft Corporation. All rights reserved.

CommandLine: D:\snow_code\全局钩取\Debug\全局钩取.exe
Symbol search path is: srv*
Executable search path is: 
ModLoad: 009c0000 009e0000   全局钩取.exe
ModLoad: 77340000 774da000   ntdll.dll
ModLoad: 74e00000 74ee0000   C:\Windows\SysWOW64\KERNEL32.DLL
ModLoad: 77110000 7730e000   C:\Windows\SysWOW64\KERNELBASE.dll
ModLoad: 6c870000 6c87c000   C:\Windows\SysWOW64\ghijt32.dll
ModLoad: 76b20000 76b99000   C:\Windows\SysWOW64\ADVAPI32.dll
ModLoad: 764a0000 7655f000   C:\Windows\SysWOW64\msvcrt.dll
ModLoad: 74ee0000 74f56000   C:\Windows\SysWOW64\sechost.dll
ModLoad: 74fd0000 7508b000   C:\Windows\SysWOW64\RPCRT4.dll
ModLoad: 74b10000 74b30000   C:\Windows\SysWOW64\SspiCli.dll
ModLoad: 74b00000 74b0a000   C:\Windows\SysWOW64\CRYPTBASE.dll
ModLoad: 76420000 7647f000   C:\Windows\SysWOW64\bcryptPrimitives.dll
ModLoad: 755b0000 756cf000   C:\Windows\SysWOW64\ucrtbase.dll
ModLoad: 74210000 74224000   C:\Windows\SysWOW64\VCRUNTIME140.dll
ModLoad: 7b210000 7b22d000   C:\Windows\SysWOW64\VCRUNTIME140D.dll
ModLoad: 7b230000 7b3a5000   C:\Windows\SysWOW64\ucrtbased.dll
(6690.2a00): Break instruction exception - code 80000003 (first chance)
eax=00000000 ebx=00b87000 ecx=e05a0000 edx=00000000 esi=01061fb0 edi=7734688c
eip=773ee9e2 esp=00cff6e8 ebp=00cff714 iopl=0         nv up ei pl zr na pe nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
ntdll!LdrpDoDebuggerBreak+0x2b:
773ee9e2 cc              int     3


//u命令
//u命令的作用就是反编译指定地址参数之后的代码，如果不指定地址参数，即只输入u命令执行，那么默认就是反编译当前线程的当前指令。
0:000> u
ntdll!LdrpDoDebuggerBreak+0x2b:
773ee9e2 cc              int     3
773ee9e3 eb07            jmp     ntdll!LdrpDoDebuggerBreak+0x35 (773ee9ec)
773ee9e5 33c0            xor     eax,eax
773ee9e7 40              inc     eax
773ee9e8 c3              ret
773ee9e9 8b65e8          mov     esp,dword ptr [ebp-18h]
773ee9ec c745fcfeffffff  mov     dword ptr [ebp-4],0FFFFFFFEh
773ee9f3 8b4df0          mov     ecx,dword ptr [ebp-10h]


0:000> u kernel32!CreateProcessW
KERNEL32!CreateProcessW:
74e19ef0 8bff            mov     edi,edi
74e19ef2 55              push    ebp
74e19ef3 8bec            mov     ebp,esp
74e19ef5 5d              pop     ebp
74e19ef6 ff25d814e874    jmp     dword ptr [KERNEL32!WerpLaunchAeDebug+0x1dde8 (74e814d8)]
74e19efc cc              int     3
74e19efd cc              int     3
74e19efe cc              int     3


0:000> dd 74e814d8
74e814d8  772183d0 772219e0 7721d610 77222860
74e814e8  7723ee70 771fecf0 772b75b0 772251f0
74e814f8  7722bd30 772b5770 77225aa0 772256a0
74e81508  77226020 7721d6a0 00000000 772b5c60
74e81518  772b7710 772b5810 772b59f0 77221240
74e81528  77220950 77200df0 772206d0 772b5b60
74e81538  77230730 77224270 00000000 772b5960
74e81548  772b5cb0 772b5a40 772b7a10 772b7900


0:000> u 772183d0
KERNELBASE!CreateProcessW:
772183d0 8bff            mov     edi,edi
772183d2 55              push    ebp
772183d3 8bec            mov     ebp,esp
772183d5 6a00            push    0
772183d7 ff752c          push    dword ptr [ebp+2Ch]
772183da ff7528          push    dword ptr [ebp+28h]
772183dd ff7524          push    dword ptr [ebp+24h]
772183e0 ff7520          push    dword ptr [ebp+20h]


//uf命令
//uf命令是用来反编译整个函数的汇编代码，
0:000> uf KERNELBASE!CreateProcessW
KERNELBASE!CreateProcessW:
772183d0 8bff            mov     edi,edi
772183d2 55              push    ebp
772183d3 8bec            mov     ebp,esp
772183d5 6a00            push    0
772183d7 ff752c          push    dword ptr [ebp+2Ch]
772183da ff7528          push    dword ptr [ebp+28h]
772183dd ff7524          push    dword ptr [ebp+24h]
772183e0 ff7520          push    dword ptr [ebp+20h]
772183e3 ff751c          push    dword ptr [ebp+1Ch]
772183e6 ff7518          push    dword ptr [ebp+18h]
772183e9 ff7514          push    dword ptr [ebp+14h]
772183ec ff7510          push    dword ptr [ebp+10h]
772183ef ff750c          push    dword ptr [ebp+0Ch]
772183f2 ff7508          push    dword ptr [ebp+8]
772183f5 6a00            push    0
772183f7 e874000000      call    KERNELBASE!CreateProcessInternalW (77218470)
772183fc 5d              pop     ebp
772183fd c22800          ret     28h
```


继续跟踪进入 `KERNELBASE!CreateProcessInternalW`，因为此函数很大，比较复杂，所以用 IDA 继续看其实现。


# 0x03 CreateProcessW 函数所属模块问题的探讨

在继续跟下去的时候，我发现一个奇怪的现象：

```

Microsoft (R) Windows Debugger Version 10.0.20153.1000 X86
Copyright (c) Microsoft Corporation. All rights reserved.

CommandLine: D:\snow_code\createprocess\Debug\createprocess.exe

************* Path validation summary **************
Response                         Time (ms)     Location
Deferred                                       srv*
Deferred                                       cache*D:\Symbols
Deferred                                       srv*http://msdl.microsoft.com/download/symbols
Symbol search path is: srv*;cache*D:\Symbols;srv*http://msdl.microsoft.com/download/symbols
Executable search path is: 
ModLoad: 00490000 004b0000   createprocess.exe
ModLoad: 77dc0000 77f5a000   ntdll.dll
ModLoad: 776c0000 777a0000   C:\Windows\SysWOW64\KERNEL32.DLL
ModLoad: 76aa0000 76c9e000   C:\Windows\SysWOW64\KERNELBASE.dll
ModLoad: 6c8e0000 6c8ec000   C:\Windows\SysWOW64\ghijt32.dll
ModLoad: 755b0000 75629000   C:\Windows\SysWOW64\ADVAPI32.dll
ModLoad: 76880000 7693f000   C:\Windows\SysWOW64\msvcrt.dll
ModLoad: 75840000 758b6000   C:\Windows\SysWOW64\sechost.dll
ModLoad: 76e20000 76edb000   C:\Windows\SysWOW64\RPCRT4.dll
ModLoad: 75590000 755b0000   C:\Windows\SysWOW64\SspiCli.dll
ModLoad: 75580000 7558a000   C:\Windows\SysWOW64\CRYPTBASE.dll
ModLoad: 758c0000 7591f000   C:\Windows\SysWOW64\bcryptPrimitives.dll
ModLoad: 77c80000 77d9f000   C:\Windows\SysWOW64\ucrtbase.dll
ModLoad: 745d0000 745e4000   C:\Windows\SysWOW64\VCRUNTIME140.dll
ModLoad: 79140000 792b5000   C:\Windows\SysWOW64\ucrtbased.dll
ModLoad: 78620000 7863d000   C:\Windows\SysWOW64\VCRUNTIME140D.dll
(1430.3d08): Break instruction exception - code 80000003 (first chance)
eax=00000000 ebx=003f5000 ecx=c6520000 edx=00000000 esi=007b1fa0 edi=77dc688c
eip=77e6e9e2 esp=005af6f4 ebp=005af720 iopl=0         nv up ei pl zr na pe nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
ntdll!LdrpDoDebuggerBreak+0x2b:
77e6e9e2 cc              int     3
0:000> u 
ntdll!LdrpDoDebuggerBreak+0x2b:
77e6e9e2 cc              int     3
77e6e9e3 eb07            jmp     ntdll!LdrpDoDebuggerBreak+0x35 (77e6e9ec)
77e6e9e5 33c0            xor     eax,eax
77e6e9e7 40              inc     eax
77e6e9e8 c3              ret
77e6e9e9 8b65e8          mov     esp,dword ptr [ebp-18h]
77e6e9ec c745fcfeffffff  mov     dword ptr [ebp-4],0FFFFFFFEh
77e6e9f3 8b4df0          mov     ecx,dword ptr [ebp-10h]
0:000> u kernel32!CreateProcessW
Couldn't resolve error at 'kernel32!CreateProcessW'
0:000> .reload /user
.reload /user is kernel-only
0:000> u kernel32!CreateProcessW
Couldn't resolve error at 'kernel32!CreateProcessW'
0:000> x kernel32!CreateProcess*
776d9ef0          KERNEL32!CreateProcessWStub (_CreateProcessWStub@40)
776f40c0          KERNEL32!CreateProcessInternalAStub (_CreateProcessInternalAStub@48)
776f40e0          KERNEL32!CreateProcessInternalWStub (_CreateProcessInternalWStub@48)
776f4060          KERNEL32!CreateProcessAStub (_CreateProcessAStub@40)
776f40a0          KERNEL32!CreateProcessAsUserWStub (_CreateProcessAsUserWStub@44)
776f4080          KERNEL32!CreateProcessAsUserAStub (_CreateProcessAsUserAStub@44)
0:000> x *!CreateProcess*
776d9ef0          KERNEL32!CreateProcessWStub (_CreateProcessWStub@40)
776f40c0          KERNEL32!CreateProcessInternalAStub (_CreateProcessInternalAStub@48)
776f40e0          KERNEL32!CreateProcessInternalWStub (_CreateProcessInternalWStub@48)
776f4060          KERNEL32!CreateProcessAStub (_CreateProcessAStub@40)
776f40a0          KERNEL32!CreateProcessAsUserWStub (_CreateProcessAsUserWStub@44)
776f4080          KERNEL32!CreateProcessAsUserAStub (_CreateProcessAsUserAStub@44)
*** WARNING: Unable to verify checksum for createprocess.exe
755cfd30          ADVAPI32!CreateProcessAsUserW (<no parameter info>)
755e27c0          ADVAPI32!CreateProcessAsUserA (<no parameter info>)
755f6c50          ADVAPI32!CreateProcessWithLogonW (<no parameter info>)
755f6c90          ADVAPI32!CreateProcessWithTokenW (<no parameter info>)
76ba83d0          KERNELBASE!CreateProcessW (<no parameter info>)
76ba8470          KERNELBASE!CreateProcessInternalW (<no parameter info>)
76c47110          KERNELBASE!CreateProcessA (<no parameter info>)
76c47150          KERNELBASE!CreateProcessAsUserA (<no parameter info>)
76c47190          KERNELBASE!CreateProcessAsUserW (<no parameter info>)
76c471d0          KERNELBASE!CreateProcessInternalA (<no parameter info>)
```

使用 WinDBG 调试，会发现 `CreateProcessW` 其实在 `ADVAPI32.dll` 模块，而 `kernel32.dll` 模块里面只有 `CreateProcessWStub` 方法。

但是用 IDA 查看其实 kernel.dll 里面导出函数表是有 `CreateProcessW` 函数的（`CreateProcessA` 其实也是调用了 `CreateProcessW`）。



这个原理就类似于在 VS 2019 里面去写 DLL 的时候，用 `external c` 或者 `.def` 方法去导出函数时候，源代码里面的一些函数，导出函数时候必须变名字，也就是导出函数表里面的函数名不能跟源码里面的方法同名。`CreateProcessW` 方法和 `CreateProcessWStub` 方法也是这样，`CreateProcessWStub` 是源码里面的函数名，而 `CreateProcessW` 是导出函数表里面的重命名，其实是一个函数。

![title](https://leanote.com/api/file/getImage?fileId=5f203ff5ab64411a9f001d14)


**证明：**

以 `kernel32!CreateProcess` 为例，其 RVA = 0x6B834060 - 0x6B800000 = 0x34060


![title](https://leanote.com/api/file/getImage?fileId=5f2042beab64411a9f001d45)

WinDbg 里面：



```
0:000> u kernel32+34060
KERNEL32!CreateProcessAStub:
776f4060 8bff            mov     edi,edi
776f4062 55              push    ebp
776f4063 8bec            mov     ebp,esp
776f4065 5d              pop     ebp
776f4066 ff25d0147477    jmp     dword ptr [KERNEL32!_imp__CreateProcessA (777414d0)]
776f406c cc              int     3
776f406d cc              int     3
776f406e cc              int     3
0:000> dd 777414d0
777414d0  76c47110 76c459b0 76ba83d0 76bb19e0
777414e0  76bad610 76bb2860 76bcee70 76b8ecf0
777414f0  76c475b0 76bb51f0 76bbbd30 76c45770
77741500  76bb5aa0 76bb56a0 76bb6020 76bad6a0
77741510  00000000 76c45c60 76c47710 76c45810
77741520  76c459f0 76bb1240 76bb0950 76b90df0
77741530  76bb06d0 76c45b60 76bc0730 76bb4270
77741540  00000000 76c45960 76c45cb0 76c45a40
0:000> u 76c47110
KERNELBASE!CreateProcessA:
76c47110 8bff            mov     edi,edi
76c47112 55              push    ebp
76c47113 8bec            mov     ebp,esp
76c47115 6a00            push    0
76c47117 ff752c          push    dword ptr [ebp+2Ch]
76c4711a ff7528          push    dword ptr [ebp+28h]
76c4711d ff7524          push    dword ptr [ebp+24h]
76c47120 ff7520          push    dword ptr [ebp+20h]
```

足以证明： `CreateProcessAStub` 和 `CreateProcessA` 是同一个函数。

同理：`CreateProcessWStub` 和 `CreateProcessW` 是同一个函数。



# 0x04 继续调试

继续回来，IDA 里面看 `KERNELBASE!CreateProcessInternalW` 函数的具体实现。用 IDA 分析 `KERNELBASE.dll`，查看 `CreateProcessInternalW` 的伪码实现：


![title](https://leanote.com/api/file/getImage?fileId=5f2044baab64411c9a001d73)

一路寻找两种函数：

1. `NT` 或 `Zw` 开头的函数（进了内核）
2. `ndrclientcall`（RPC CALL）

![title](https://leanote.com/api/file/getImage?fileId=5f204619ab64411a9f001d7f)

找到了 `NtCreateUserProcess` 函数，这是一个导入函数：

![title](https://leanote.com/api/file/getImage?fileId=5f204661ab64411c9a001d8a)

在导入表中查看发现属于 `ntdll.dll`。


继续在 IDA 中加载 `ntdll.dll` 继续跟：

![title](https://leanote.com/api/file/getImage?fileId=5f2046bfab64411c9a001d8e)


发现其 `systemcall` 了，系统调用。



至此进了内核，此 `CreateProcessW` 函数实现结束。

