--------

注：本文中提到的 `wcscpy()` API，应该替换为 `wcscpy_s()` API。出于安全考虑。

因为本文是在取消了安全检查的编译器上编译通过的，故犯此错误。


------


# 0x01 无法创建进程

事情是这样的：下面这段是我写的代码，大概功能就是判断操作系统位数，然后选择 32/64-bit 机器上32位 notepad.exe 程序的绝对路径，传入 `CreateProcessW` API，然后创建一个挂起状态的 notepad.exe 进程。


```
#include <stdio.h>
#include <windows.h>
#include <tchar.h>
#include <conio.h>

using namespace std;

/* length: 799 bytes */
/* 32-bit shellcode*/
unsigned char buf[] = "\xfc\xe8\32";


/* 安全的取得真实系统信息*/
VOID SafeGetNativeSystemInfo(__out LPSYSTEM_INFO lpSystemInfo)
{
    if (NULL == lpSystemInfo)    return;
    typedef VOID(WINAPI* LPFN_GetNativeSystemInfo)(LPSYSTEM_INFO lpSystemInfo);
    LPFN_GetNativeSystemInfo fnGetNativeSystemInfo = (LPFN_GetNativeSystemInfo)GetProcAddress(GetModuleHandle(_T("kernel32")), "GetNativeSystemInfo");;
    if (NULL != fnGetNativeSystemInfo)
    {
        fnGetNativeSystemInfo(lpSystemInfo);
    }
    else
    {
        GetSystemInfo(lpSystemInfo);
    }
}

/* 获取操作系统位数 */
int GetSystemBits()
{
    SYSTEM_INFO si = {0};
    SafeGetNativeSystemInfo(&si);
    if (si.wProcessorArchitecture == PROCESSOR_ARCHITECTURE_AMD64 ||
        si.wProcessorArchitecture == PROCESSOR_ARCHITECTURE_IA64)
    {
        return 64;
    }
    return 32;
}

void _tmain(int argc, _TCHAR* argv[])
{
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    BOOL result;
    wchar_t* CommandLine;
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    /*step 0:判断机器是 32 还是 64 位，以此确定 32-bit 程序 notepad.exe 的路径*/
    const int nBitSys = GetSystemBits();
    //_tprintf(_T("This is a %d-bit System."), nBitSys);
    if (nBitSys == 32)
    {
        CommandLine = const_cast<wchar_t*>(L"C:\\Windows\\System32\\notepad.exe");     
    }
    if (nBitSys == 64)
    {
        CommandLine = const_cast<wchar_t*>(L"C:\\Windows\\SysWOW64\\notepad.exe");
    }
    _tprintf(_T("Commandline:\n%s"), CommandLine);
    /*step 1: 调用 CreateProcess 以挂起的方式（CREATE_SUSPENDED）创建进程*/
    result = CreateProcessW(NULL, CommandLine, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL,NULL,&si,&pi);
    if (!result)
    {
        _tprintf(_T("CreateProcess failed (%d).\n"), GetLastError());
        return;
    }
    // Wait until child process exits.
    WaitForSingleObject(pi.hProcess, INFINITE);
    //Close process and thread handles.
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);
    
    return;
}
```


此 API 我已经用过很多次了，但是这次我遇到一个问题，就是：


![title](https://leanote.com/api/file/getImage?fileId=5f31408cab64413c97001b72)


更诡异的问题是：

- 用 `Debug`、`x86` 无法创建挂起进程；
- 用 `Release`、`x86` 可以正常创建挂起进程；
- 暂不考虑 `x64`，因为最终此代码将跟 32 位 shellcode 结合使用。


# 0x02 问题分析

通过打印字符串 Debug 方法，发现程序运行到此行代码崩溃：

```
result = CreateProcessW(NULL, CommandLine, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL,NULL,&si,&pi);
```

甚至无法 `GetLastError()`，因为在这句代码执行中就崩溃了，不会获取返回值。

观察一下我在 `_tmain()` 主函数模块中定义的 CommandLine 变量，相关代码有以下几行：


```
    wchar_t* CommandLine;
    if (nBitSys == 32)
    {
        CommandLine = const_cast<wchar_t*>(L"C:\\Windows\\System32\\notepad.exe");     
    }
    if (nBitSys == 64)
    {
        CommandLine = const_cast<wchar_t*>(L"C:\\Windows\\SysWOW64\\notepad.exe");
    }
    _tprintf(_T("Commandline:\n%s"), CommandLine);
    /*step 1: 调用 CreateProcess 以挂起的方式（CREATE_SUSPENDED）创建进程*/
    result = CreateProcessW(NULL, CommandLine, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL,NULL,&si,&pi);
```


- 首先我把 `CommandLine` 定义为一个 `wchar_t` 类型的指针，并未对其初始化；
- 然后根据位数，对其赋值，强行去掉 `const wchar_t*` 的 const 属性，赋值给 CommandLine；
- 把 CommandLine 作为第二个参数传给 CreateProcessW API。


看上去语义正确，且能成功生成解决方案，但是为什么一运行就崩，无法正常生成挂起进程呢？


既然我崩的是 `CreateProcessW` API，那么就从传参查起。

![title](https://leanote.com/api/file/getImage?fileId=5f314359ab64413a91001c17)



```
    wchar_t* CommandLine;
    if (nBitSys == 32)
    {
        CommandLine = const_cast<wchar_t*>(L"C:\\Windows\\System32\\notepad.exe");     
    }
    if (nBitSys == 64)
    {
        CommandLine = const_cast<wchar_t*>(L"C:\\Windows\\SysWOW64\\notepad.exe");
    }
```


第一我没有对 CommandLine 初始化，其次，既然实际的传入内容是 `L"C:\\Windows\\System32\\notepad.exe"`，哪怕我对它使用 `const_cast<wchar_t*>` 强制去除了 const 属性，但是对于这样一个 `wchar_t*` 指针，编译出来的 PE 还是会把参数内容存入 `.data` 节区，属性就是可读不可写。但是此 API 要求传入的第二个参数必须可写。这样权限不一致就导致了访问冲突。

所以根本原因在于：我所谓的强制去除 const 属性，只是语义层面的。在编译时、编译器还是会把这个参数内容包含在文件映像的只读部分，就会引起访问违规。


# 0x03 代码优化

## 方法1:

方法1 是把 `CommandLine` 的类型定义为数组。你可能会疑问：数组不就是指针，有什么区别吗？

虽然数组的确是指针，但是参考下文：自动分配内存的数组是在栈中的。


[请问C中的数组是存在栈中，还是堆中？](https://bbs.csdn.net/topics/390536150)


所以以数组方式来定义 CommandLine，在内存中必定是把此字符串放在可读/写内存中的，就不会违规了。


实现代码：



```
#include <stdio.h>
#include <windows.h>
#include <tchar.h>
#include <conio.h>

using namespace std;

/* length: 799 bytes */
/* 32-bit shellcode*/
unsigned char buf[] = "\xfc\xe8\x89";


/* 安全的取得真实系统信息*/
VOID SafeGetNativeSystemInfo(__out LPSYSTEM_INFO lpSystemInfo)
{
    if (NULL == lpSystemInfo)    return;
    typedef VOID(WINAPI* LPFN_GetNativeSystemInfo)(LPSYSTEM_INFO lpSystemInfo);
    LPFN_GetNativeSystemInfo fnGetNativeSystemInfo = (LPFN_GetNativeSystemInfo)GetProcAddress(GetModuleHandle(_T("kernel32")), "GetNativeSystemInfo");;
    if (NULL != fnGetNativeSystemInfo)
    {
        ZeroMemory(lpSystemInfo, sizeof(SYSTEM_INFO));
        //printf("%p", fnGetNativeSystemInfo);
        //MessageBoxW(NULL,NULL,NULL,MB_OK);
        fnGetNativeSystemInfo(lpSystemInfo);
    }
    else
    {
        GetSystemInfo(lpSystemInfo);
    }
}

/* 获取操作系统位数 */
int GetSystemBits()
{
    SYSTEM_INFO si = { 0 };
    SafeGetNativeSystemInfo(&si);
    if (si.wProcessorArchitecture == PROCESSOR_ARCHITECTURE_AMD64 ||
        si.wProcessorArchitecture == PROCESSOR_ARCHITECTURE_IA64)
    {
        return 64;
    }
    return 32;
}

void _tmain(int argc, _TCHAR* argv[])
{
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    BOOL result;
    wchar_t CommandLine[MAX_PATH] = { 0 };
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    /*step 0:判断机器是 32 还是 64 位，以此确定 32-bit 程序 notepad.exe 的路径*/
    const int nBitSys = GetSystemBits();
    //_tprintf(_T("This is a %d-bit System."), nBitSys);
    //wcscpy 第一个参数一定得清零，要么赋值为0，要么 ZeroMemory，否则可能复制字符串最后的/0
    if (nBitSys == 32)
    {
        wcscpy(CommandLine, L"C:\\Windows\\System32\\notepad.exe");
    }
    if (nBitSys == 64)
    { 
        wcscpy(CommandLine, L"C:\\Windows\\SysWOW64\\notepad.exe");
    }
    _tprintf(_T("Commandline:\n%s"), CommandLine);
    /*step 1: 调用 CreateProcess 以挂起的方式（CREATE_SUSPENDED）创建进程*/
    result = CreateProcessW(NULL, CommandLine, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi);
    if (!result)
    {
        _tprintf(_T("CreateProcess failed (%d).\n"), GetLastError());
        return;
    }
    // Wait until child process exits.
    WaitForSingleObject(pi.hProcess, INFINITE);
    //Close process and thread handles.
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return;
}
```

`Debug` `x86` 配置下编译执行，发现挂起进程创建成功：

![title](https://leanote.com/api/file/getImage?fileId=5f314d4cab64413c97001c1c)



关键代码：


```
wchar_t CommandLine[MAX_PATH] = { 0 };
//wcscpy 第一个参数一定得清零，要么赋值为0，要么 ZeroMemory，否则可能复制字符串最后的/0
if (nBitSys == 32)
{
    wcscpy(CommandLine, L"C:\\Windows\\System32\\notepad.exe");
}
if (nBitSys == 64)
{ 
    wcscpy(CommandLine, L"C:\\Windows\\SysWOW64\\notepad.exe");
}

result = CreateProcessW(NULL, CommandLine, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi);
```


- 通过 `wsccpy()` API 宽字节把字符串复制进 `wchat_t` 类型数组。
- CommandLine 数组一开始初始化为0。要注意 `wcscpy()` 第一个参数一定得清零，要么赋值为0，要么 ZeroMemory，否则可能复制字符串最后的`\0`。
- [MSDN - strcpy_s，wcscpy_s，_mbscpy_s，_mbscpy_s_l](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/strcpy-s-wcscpy-s-mbscpy-s?view=vs-2019)


## 方法2：


使用 `_wcsdup()` API。 


`_wcsdup()` 是 `_strdup()` 的宽字符版本。 `_wcsdup()` 的参数和返回值是宽字符字符串。



```
wchar_t *_wcsdup(  
   const wchar_t *strSource   
); 
```

`_strdup` 函数调用 malloc 来为 strSource 的副本分配存储空间，然后将 strSource 复制到分配的空间。所以，最后要调用 free 释放堆内存。

参考：

- [CSDN -_strdup、_wcsdup、_mbsdup 浅析](https://blog.csdn.net/hellokandy/article/details/78360988)
- [msdn - _strdup，_wcsdup，_mbsdup](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/strdup-wcsdup-mbsdup?view=vs-2019)


实现代码：

```
#include <stdio.h>
#include <windows.h>
#include <tchar.h>
#include <conio.h>

using namespace std;

/* length: 799 bytes */
/* 32-bit shellcode*/
unsigned char buf[] = "\xfc\xe8\x89";


/* 安全的取得真实系统信息*/
VOID SafeGetNativeSystemInfo(__out LPSYSTEM_INFO lpSystemInfo)
{
    if (NULL == lpSystemInfo)    return;
    typedef VOID(WINAPI* LPFN_GetNativeSystemInfo)(LPSYSTEM_INFO lpSystemInfo);
    LPFN_GetNativeSystemInfo fnGetNativeSystemInfo = (LPFN_GetNativeSystemInfo)GetProcAddress(GetModuleHandle(_T("kernel32")), "GetNativeSystemInfo");;
    if (NULL != fnGetNativeSystemInfo)
    {
        ZeroMemory(lpSystemInfo, sizeof(SYSTEM_INFO));
        fnGetNativeSystemInfo(lpSystemInfo);
    }
    else
    {
        GetSystemInfo(lpSystemInfo);
    }
}

/* 获取操作系统位数 */
int GetSystemBits()
{
    SYSTEM_INFO si = { 0 };
    SafeGetNativeSystemInfo(&si);
    if (si.wProcessorArchitecture == PROCESSOR_ARCHITECTURE_AMD64 ||
        si.wProcessorArchitecture == PROCESSOR_ARCHITECTURE_IA64)
    {
        return 64;
    }
    return 32;
}

void _tmain(int argc, _TCHAR* argv[])
{
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    BOOL result;
    const wchar_t* CommandLine = 0;
    //或者
    //wchar_t CommandLine[MAX_PATH] = { 0 };
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    /*step 0:判断机器是 32 还是 64 位，以此确定 32-bit 程序 notepad.exe 的路径*/
    const int nBitSys = GetSystemBits();
    if (nBitSys == 32)
    {
        CommandLine = L"C:\\Windows\\System32\\notepad.exe";

    }
    if (nBitSys == 64)
    {
        CommandLine = L"C:\\Windows\\SysWOW64\\notepad.exe";
    }
    _tprintf(_T("Commandline:\n%s"), CommandLine);
    /*step 1: 调用 CreateProcess 以挂起的方式（CREATE_SUSPENDED）创建进程*/
    wchar_t* CommandString = _wcsdup(CommandLine);
    result = CreateProcessW(NULL, CommandString , NULL, NULL, FALSE, CREATE_SUSPENDED, NULL,NULL,&si,&pi);
    //printf("result: %d", result);
    if (!result)
    {
        _tprintf(_T("CreateProcess failed (%d).\n"), GetLastError());
        return;
    }
    //CommandLine 只是个临时变量，及时释放缓冲区
    free(CommandString);
    // Wait until child process exits.
    WaitForSingleObject(pi.hProcess, INFINITE);
    //Close process and thread handles.
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return;
}
```


`Debug` `x86` 配置下编译执行，发现挂起进程创建成功：


![title](https://leanote.com/api/file/getImage?fileId=5f314ce3ab64413c97001c16)



关键代码：

```
const wchar_t* CommandLine = 0;

if (nBitSys == 32)
{
    CommandLine = L"C:\\Windows\\System32\\notepad.exe";

}
if (nBitSys == 64)
{
    CommandLine = L"C:\\Windows\\SysWOW64\\notepad.exe";
}

//这一步是为了获取新分配的缓冲区地址，方便一会儿 free() 释放内存空间
wchar_t* CommandString = _wcsdup(CommandLine);
result = CreateProcessW(NULL, CommandString , NULL, NULL, FALSE, CREATE_SUSPENDED, NULL,NULL,&si,&pi);

//CommandLine 只是个临时变量，及时释放缓冲区
free(CommandString);
```


- 注：`_wcsdup()` 也可以用来把数组复制到指针中。参考此文中的示例：[CSDN -_strdup、_wcsdup、_mbsdup 浅析](https://blog.csdn.net/hellokandy/article/details/78360988)

# 0x04 总结

`CreateProcess()` 函数的第二个参数 `lpCommandLine` 用于指定要传给新进程的命令行字符串。


在函数原型中，`lpCommandLine` 参数的类型为 `LPWSTR`，这意味着 `CreateProcess` 期望我们传入的是一个非“常量字符串”的地址。在内部，CreateProcess 实际上会修改我们传给它的命令行字符串。但是 CreateProcess 返回之前，它会将这个字符串还原为原来的形式。


这一看上去微不足道的细节其实很重要，因为如果命令行字符串包含在文件映像的只读部分（如 `.data` 节区），就会引起访问违规，例如，以下代码就会导致访问违规，因为 Microsoft 的 C/C++ 编译器把 `NOTEPAD` 字符串放在只读内存中：

```
STARTUPINFO si = { sizeof(si) };
PROCESS_INFORMATION pi;
CreateProcess(NULL, TEXT("NOTEPAD"), NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi);
```


`CreateProcess()` 试图修改字符串时，会引起一个访问违规。如：

![title](https://leanote.com/api/file/getImage?fileId=5f31408cab64413c97001b72)



解决这个问题的最佳方式是：

**在调用 CreateProcess() 之前，把常量字符串复制到一个临时缓冲区。**如下所示：

```
STARTUPINFO si = { sizeof(si) };
PROCESS_INFOMATION pi;
TCHAR szCommandLine[] = TEXT("NOTEPAD");
CreateProcess(NULL, szCommandLine, NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi);
```


也可以使用 `_wcsdup()`、`wcscpy` 等 API 把常量字符串复制到临时缓冲区中。但是要注意：被复制的字符串/数组一定要清零，可以直接赋值为0，也可以使用 `ZeroMemory()` API。初始化是一种良好的编程习惯！

至于为什么 Release 配置下可以正常执行功能，可能是因为此配置下做了一些编译时优化，把字符串放在可读/写内存中，所以对 `CreateProcess()` 的调用不会引起访问违规。



----------------


## 参考文档：

- Windows 核心编程（第五版），P86
- [Linux X86架构参数传递规则](https://blog.csdn.net/u010039418/article/details/85275211)：32-bit 程序参数入栈，64-bit 程序寄存器传参

