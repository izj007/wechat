##  干货｜Windows下进程操作的一些C++代码

原创 11ccaab  [ HACK学习呀 ](javascript:void\(0\);)

**HACK学习呀** ![]()

微信号 Hacker1961X

功能介绍
HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

____

__

收录于话题

#c++ 1

#代码 1

#windows 1

#免杀 1

#进程操作 1

# 0x01 进程遍历

因为进程是在随时进行变动的所以我们需要获取一张快照

## 1.1 CreateToolhelp32Snapshot

    
    
    HANDLE CreateToolhelp32Snapshot(  DWORD dwFlags,  DWORD th32ProcessID);

因为要获取进程第一个参数选择TH32CS_SNAPPROCESS来获取系统中所有的进程，具体可以参考[CreateToolhelp32Snapshot]：https://docs.microsoft.com/zh-
cn/windows/win32/api/tlhelp32/nf-
tlhelp32-createtoolhelp32snapshot?f1url=%3FappId%3DDev16IDEF1%26l%3DZH-
CN%26k%3Dk(TLHELP32%252FCreateToolhelp32Snapshot);k(CreateToolhelp32Snapshot);k(DevLang-C%252B%252B);k(TargetOS-
Windows)%26rd%3Dtrue

## 1.2 Process32First

    
    
    BOOL Process32First(  HANDLE           hSnapshot,  LPPROCESSENTRY32 lppe);

第一个参数使用上面CreateToolhelp32Snapshot函数返回的句柄。第二个参数执行了PROCESSENTRY32结构的指针，它包含了进程信息。检索进程里的第一个进程信息。

### 1.2.1 PROCESSENTRY32

    
    
    typedef struct tagPROCESSENTRY32 {  DWORD     dwSize;  DWORD     cntUsage;  DWORD     th32ProcessID;  ULONG_PTR th32DefaultHeapID;  DWORD     th32ModuleID;  DWORD     cntThreads;  DWORD     th32ParentProcessID;  LONG      pcPriClassBase;  DWORD     dwFlags;  CHAR      szExeFile[MAX_PATH];} PROCESSENTRY32;

使用时候要把结构体清零。szExeFile为进程名称，其他都根据名称一样。

## 1.3 Process32Next

    
    
    BOOL Process32Next(  HANDLE           hSnapshot,  LPPROCESSENTRY32 lppe);

检索快照中的下一个进程信息。

    
    
    void ListProcess() {    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);    PROCESSENTRY32 pe32 = { 0 };    pe32.dwSize = sizeof(PROCESSENTRY32);    BOOL bRet = Process32First(hSnap, &pe32);    while (bRet)    {        bRet = Process32Next(hSnap, &pe32);        printf(pe32.szExeFile);        pid = pe32.th32ProcessID;        wprintf(L"\r\n");        printf("pid:%d", pe32.th32ProcessID);        printf("\r\n-----------------------------------------");        wprintf(L"\r\n");    }    ::CloseHandle(hSnap);}

![](https://gitee.com/fuli009/images/raw/master/public/20210810104555.png)

# 0x02 模块遍历

同理只需要将CreateToolhelp32Snapshot的dwFlags修改为TH32CS_SNAPMODULE，th32ProcessID参数为进程的pid，这里要先获取进程pid。

## 2.1 获取进程pid

    
    
    typedef struct tagPROCESSENTRY32 {  DWORD     dwSize;  DWORD     cntUsage;  DWORD     th32ProcessID;  ULONG_PTR th32DefaultHeapID;  DWORD     th32ModuleID;  DWORD     cntThreads;  DWORD     th32ParentProcessID;  LONG      pcPriClassBase;  DWORD     dwFlags;  CHAR      szExeFile[MAX_PATH];} PROCESSENTRY32;

通过PROCESSENTRY32结构体的th32ProcessID来获取进程pid。遍历进程通过strcmp匹配到我们的进程名就返回the32ProcessID。

    
    
    int GetProcessPid(char* ProcessName) {    Flag = CheckPorcess(ProcessName);    if (Flag) {        HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);        PROCESSENTRY32 pe32 = { 0 };        pe32.dwSize = sizeof(PROCESSENTRY32);        BOOL bRet = Process32First(hSnap, &pe32);        while (bRet)        {            bRet = Process32Next(hSnap, &pe32);            if (strcmp(pe32.szExeFile, ProcessName) == 0) {                pid = pe32.th32ProcessID;            }        }        CloseHandle(hSnap);        return pid;    }    printf("[-]Process not found");    return 0;}

## 2.2 模块遍历

获取到了进程pid，放入CreateToolhelp32Snapshot第二个参数，但是因为考虑到进程可能根本不存在所以写一个CheckPorcess方法来判断是否存在该进程。

    
    
    BOOL CheckPorcess(char* ParanetProcessName) {    DWORD i = 0;    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);    PROCESSENTRY32 pe32 = { 0 };    pe32.dwSize = sizeof(PROCESSENTRY32);    BOOL bRet = Process32First(hSnap, &pe32);    while (bRet)    {        bRet = Process32Next(hSnap, &pe32);        if (strcmp(pe32.szExeFile, ParanetProcessName) == 0) {            CloseHandle(hSnap);            return TRUE;        }    }    return FALSE;}

然后遍历模块

    
    
    BOOL ListModule(char* ProcessName) {    Flag = CheckPorcess(ProcessName);    if (Flag) {        pid = GetProcessPid(ProcessName);        HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid);        MODULEENTRY32 me32 = { 0 };        me32.dwSize = sizeof(MODULEENTRY32);        BOOL bRet = Module32First(hSnap, &me32);        while (bRet)        {            bRet = Module32Next(hSnap, &me32);            printf(me32.szExePath);            printf("\r\n");        }        CloseHandle(hSnap);        return TRUE;    }        printf("[-]Process not found");    return FALSE;}

![](https://gitee.com/fuli009/images/raw/master/public/20210810104603.png)

# 0x03 遍历线程

使用TH32CS_SNAPTHREAD参数来获取，这里都大同小异了。

    
    
    BOOL ListThread(char* ProcessName) {    Flag = CheckPorcess(ProcessName);    if (Flag) {        pid = GetProcessPid(ProcessName);        THREADENTRY32 te32 = { 0 };        te32.dwSize = sizeof(THREADENTRY32);        HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, pid);        BOOL bRet = Thread32First(hSnap, &te32);        while (bRet)        {            Thread32Next(hSnap, &te32);            if (te32.th32OwnerProcessID == pid) {                printf(TEXT("\n     THREAD ID      = 0x%08X"), te32.th32ThreadID);                printf(TEXT("\n     base priority  = %d"), te32.tpBasePri);                printf(TEXT("\n     delta priority = %d"), te32.tpDeltaPri);                break;            }        }        CloseHandle(hSnap);        return TRUE;    }    printf("[-]Process not found");    return FALSE;}

通过te32.th32OwnerProcessID与GetProcessPid方法获取到的pid来对比进而确定当前进程。

# 0x04 干进程

## 4.1 TerminateProcess

    
    
    BOOL TerminateProcess(  HANDLE hProcess,  UINT   uExitCode);

第一个参数为要结束进程的进程句柄，第二个参数为终止代码。

## 4.2 OpenProcess

    
    
    HANDLE OpenProcess(  DWORD dwDesiredAccess,  BOOL  bInheritHandle,  DWORD dwProcessId);

第一个参数为进程访问权限这里设置为拥有全部权限PROCESS_ALL_ACCESS，具体查看[进程访问权限]：https://docs.microsoft.com/en-
us/windows/win32/procthread/process-security-and-access-rights
第二个参数为是否要继承句柄。第三个参数为进程pid。

    
    
    BOOL KillProcess(char* ProcessName) {    Flag = CheckPorcess(ProcessName);    if (Flag) {        HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);        PROCESSENTRY32 pe32 = { 0 };        pe32.dwSize = sizeof(PROCESSENTRY32);        BOOL bRet = Process32First(hSnap, &pe32);        while (bRet)        {            bRet = Process32Next(hSnap, &pe32);            if (strcmp(pe32.szExeFile, ProcessName) == 0) {                pid = pe32.th32ProcessID;            }        }        HANDLE hProcessName = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);        bRet = TerminateProcess(hProcessName, 0);        if (bRet) {            printf("Kill %s success!", pe32.szExeFile);        }        else        {            printf("Kill %s fail!", pe32.szExeFile);        }        CloseHandle(hSnap);        return TRUE;    }    printf("[-]Process not found");    return FALSE;}

# 0x05 dll注入

dll加载：1.静态调用：通过在我们的程序中添加头文件，以及lib文件来完成调用，前提就是获取dll然后还有头文件
2.动态调用：仅仅只需要一个dll即可完成调用

这里写一个Test方法

    
    
    #include <Windows.h>__declspec(dllexport) void Test(){    MessageBox(NULL, NULL, NULL, NULL);}

![](https://gitee.com/fuli009/images/raw/master/public/20210810104605.png)

可以看到有一些脏数据。这里可以协商约定来解决 1.__stdcall 标准 栈传参，函数内部（被调用者）平栈 2. __cdecl c
栈传参，函数外部（调用者）平栈 3. __fastcall 快速 寄存器传参 4. __thiscall
类的thiscall调用约定，使用ecx寄存器来传递this指针

    
    
    extern "C"{    __declspec(dllexport) void __stdcall Test(){        MessageBox(NULL, NULL, NULL, NULL);    }}

__stdcall是函数内部平参可以举个例子

    
    
    void __stdcall test(int n1, int n2){        return;}int main(){    test(1, 2);    return 0;}

两个返回8一个返回4

    
    
    void __stdcall test(int n1, int n2){002013C0  push        ebp  002013C1  mov         ebp,esp  002013C3  sub         esp,0C0h  002013C9  push        ebx  002013CA  push        esi  002013CB  push        edi  002013CC  lea         edi,[ebp-0C0h]  002013D2  mov         ecx,30h  002013D7  mov         eax,0CCCCCCCCh  002013DC  rep stos    dword ptr es:[edi]          return;}002013DE  pop         edi  002013DF  pop         esi  002013E0  pop         ebx  002013E1  mov         esp,ebp  002013E3  pop         ebp  002013E4  ret         8

## 5.1 动态调用

这里使用动态调用

流程大概就是 1.在目标进程中申请内存 2.向目标进程内存中写入shellcode(没有特征，编码比较麻烦) 3.创建远线程执行shellcode

首先打开进程获取到进程句柄

    
    
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, rProcessId);if (hProcess == INVALID_HANDLE_VALUE) {    return FALSE;}

然后再要注入的进程中申请内存

VirtualAllocEx

    
    
    LPVOID VirtualAllocEx(  HANDLE hProcess,  LPVOID lpAddress,  SIZE_T dwSize,  DWORD  flAllocationType,  DWORD  flProtect);

详情查看[VirtualAllocEx]：https://docs.microsoft.com/zh-
cn/windows/win32/api/memoryapi/nf-memoryapi-
virtualallocex?f1url=%3FappId%3DDev16IDEF1%26l%3DZH-
CN%26k%3Dk(MEMORYAPI%252FVirtualAllocEx);k(VirtualAllocEx);k(DevLang-C%252B%252B);k(TargetOS-
Windows)%26rd%3Dtrue 这里要用到可读可写权限PAGE_READWRITE。

    
    
    pDllAddr = VirtualAllocEx(hProcess, NULL, strlen(szDllPath) + 1, MEM_COMMIT, PAGE_READWRITE);

然后再要注入的进程里面写入数据

WriteProcessMemory

    
    
    BOOL WriteProcessMemory(  HANDLE  hProcess,  LPVOID  lpBaseAddress,  LPCVOID lpBuffer,  SIZE_T  nSize,  SIZE_T  *lpNumberOfBytesWritten);
    
    
    WriteProcessMemory(hProcess, pDllAddr, szDllPath, strlen(szDllPath) + 1, NULL);

然后获取loadlibrary的地址后就通过CreateRemoteThread加载dll地址和函数地址来调用

    
    
    pfnStartAddr = GetProcAddress(::GetModuleHandle("Kernel32"), "LoadLibraryA");

最后再使用CreateRemoteThread创建远线程，注入DLL

    
    
    if ((hRemoteThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)pfnStartAddr, pDllAddr, 0, NULL)) == NULL)    {        std::cout << "Injecting thread failed!" << std::endl;        return FALSE;    }

InjectDll函数

    
    
    BOOL InjectDll(int rProcessId, const char* szDllPath) {    HANDLE hProcess = NULL;    LPVOID pDllAddr = NULL;    FARPROC pfnStartAddr = NULL;    HANDLE hRemoteThread = NULL;    //打开进程    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, rProcessId);    if (hProcess == INVALID_HANDLE_VALUE) {        return FALSE;    }    //在要注入的进程中申请内存    pDllAddr = VirtualAllocEx(hProcess, NULL, strlen(szDllPath) + 1, MEM_COMMIT, PAGE_READWRITE);    if (!pDllAddr)    {        return FALSE;    }    //给要注入的进程中写入数据    WriteProcessMemory(hProcess, pDllAddr, szDllPath, strlen(szDllPath) + 1, NULL);    //获取LoadLibraryA函数的地址    pfnStartAddr = GetProcAddress(::GetModuleHandle("Kernel32"), "LoadLibraryA");    //使用CreateRemoteThread创建远线程，注入DLL    if ((hRemoteThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)pfnStartAddr, pDllAddr, 0, NULL)) == NULL)    {        std::cout << "Injecting thread failed!" << std::endl;        return FALSE;    }    /*CloseHandle(hProcess);    CloseHandle(hRemoteThread);*/    return TRUE;}

这里我们写一个程序加一个dll来验证

DemoDll.dll

    
    
    // dllmain.cpp : 定义 DLL 应用程序的入口点。#include "pch.h"#include <windows.h>#include <process.h>#include <processthreadsapi.h>#include <atlstr.h>int pid = 0;CString str;int GetCurrentID() {    pid = _getpid();    str.Format("%d", pid);    return pid;}BOOL APIENTRY DllMain( HMODULE hModule,                       DWORD  ul_reason_for_call,                       LPVOID lpReserved                     ){    switch (ul_reason_for_call)    {    case DLL_PROCESS_ATTACH:        pid = GetCurrentID();        MessageBox(NULL, str, "title", MB_OK);    case DLL_THREAD_ATTACH:    case DLL_THREAD_DETACH:    case DLL_PROCESS_DETACH:        break;    }    return TRUE;}

Demo.exe

    
    
    #include <windows.h>#include <stdio.h>void main() {    printf("test\r\n");    system("pause");}

运行Demo.exe，然后先遍历模块

![](https://gitee.com/fuli009/images/raw/master/public/20210810104606.png)

dll注入

![](https://gitee.com/fuli009/images/raw/master/public/20210810104608.png)![](https://gitee.com/fuli009/images/raw/master/public/20210810104609.png)![](https://gitee.com/fuli009/images/raw/master/public/20210810104611.png)

# 0x06 PPID欺骗

PPID欺骗允许使用任意进程启动程序

同理先判断父进程是否存在

    
    
    DWORD GetParanetProcessID(char* ParanetProcessName) {    BOOL Check = CheckPorcess(ParanetProcessName);    if (Check) {        HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);        PROCESSENTRY32 pe32 = { 0 };        pe32.dwSize = sizeof(PROCESSENTRY32);        BOOL bRet = Process32First(hSnap, &pe32);        while (bRet)        {            Process32Next(hSnap, &pe32);            if (strcmp(pe32.szExeFile, ParanetProcessName) == 0) {                ppid = pe32.th32ProcessID;                break;            }        }        CloseHandle(hSnap);    }    return ppid;}
    
    
    void PpidSpoofing(char* ProcessAddr,DWORD ParaentPid) {    STARTUPINFOEXA si = { 0 };    si.StartupInfo.cb = sizeof(STARTUPINFOEXA);    PROCESS_INFORMATION pi = { 0 };    SIZE_T SizeBuff;    HANDLE parentProcessHandle = OpenProcess(PROCESS_ALL_ACCESS, false, ParaentPid);    ZeroMemory(&si, sizeof(STARTUPINFOEXA));    InitializeProcThreadAttributeList(NULL, 1, 0, &SizeBuff);    si.lpAttributeList = (LPPROC_THREAD_ATTRIBUTE_LIST)HeapAlloc(GetProcessHeap(), 0, SizeBuff);    InitializeProcThreadAttributeList(si.lpAttributeList, 1, 0, &SizeBuff);    UpdateProcThreadAttribute(si.lpAttributeList, 0, PROC_THREAD_ATTRIBUTE_PARENT_PROCESS, &parentProcessHandle, sizeof(HANDLE), NULL, NULL);    BOOL bRet = CreateProcessA(ProcessAddr, NULL, NULL, NULL, TRUE, CREATE_NEW_CONSOLE | EXTENDED_STARTUPINFO_PRESENT, NULL, NULL, reinterpret_cast<LPSTARTUPINFOA>(&si), &pi);    if (!bRet) {        printf("ProcessAddr error!");    }    CloseHandle(parentProcessHandle);}

首先我们要获取父进程的进程句柄然后为进程和线程创建初始化指定的属性列表使用InitializeProcThreadAttributeList。

    
    
    BOOL InitializeProcThreadAttributeList(  LPPROC_THREAD_ATTRIBUTE_LIST lpAttributeList,  DWORD                        dwAttributeCount,  DWORD                        dwFlags,  PSIZE_T                      lpSize);

最后一个参数指定输入时lpAttributeList缓冲区的大小。si.lpAttributeList在堆中分配一块内存，分配的大小为前面的SizeBuff。然后再使用InitializeProcThreadAttributeList初始化进程和线程的属性列表最后使用UpdateProcThreadAttribute函数来更新进程和线程的指定属性，最后创建我们的进程。

![](https://gitee.com/fuli009/images/raw/master/public/20210810104612.png)![]()

**![](https://gitee.com/fuli009/images/raw/master/public/20210810104613.png)**

  

 **推荐阅读：**

  

[Office如何快速进行宏免杀](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247492416&idx=1&sn=c444b28f7aa67e9ee15d42bc1aeef10d&chksm=ec1cb67fdb6b3f69d33753fd68cad86f401c07f5fddb3157c81468cab144978a5fb3840da037&scene=21#wechat_redirect)  

  

[远程线程注入Dll，突破Session
0](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496299&idx=1&sn=a3c3ac02810b31b728648cb52f8601d5&chksm=ec1ca754db6b2e423418323353e2e00a85c8cb2c261dd407e6fa2587c6f657680c0b5c909901&scene=21#wechat_redirect)  

  

[干货 |
如何快速完成DLL劫持，实现权限维持，重启上线](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247493305&idx=1&sn=5b292ca23204c7c6adabaff3bc6d7edf&chksm=ec1cb386db6b3a90e08b077116175120e078e74697832f9455eb2dc7020334955b8642af8288&scene=21#wechat_redirect)  

  

[科普 |
DLL劫持原理与实践](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247487178&idx=1&sn=41916ea90c1503452ab68dcf435df470&chksm=ec1f5bf5db68d2e3ee81d150ac24fdf1145d33c700e1c49f1619e64050d8b104dc736b8d0c0d&scene=21#wechat_redirect)

  

本月报名可以参加抽奖送BADUSB的优惠活动  

  
[![](https://gitee.com/fuli009/images/raw/master/public/20210810104614.png)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247498688&idx=1&sn=d81921a3873e254b0a135d9ffaa00468&chksm=ec1caeffdb6b27e9d129e1b00e92e01d49ccca43bb18f2388c733143557bfaaf62d0efd7f22f&scene=21#wechat_redirect)

  

 **点赞，转发，在看**

  

原创投稿作者：11ccaab

![](https://gitee.com/fuli009/images/raw/master/public/20210810104615.png)

  

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

干货｜Windows下进程操作的一些C++代码

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

