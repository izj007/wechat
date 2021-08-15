请将本文与 [Bypass DACL 注入进程（二）](http://blog.leanote.com/post/snowming/c8fe5abeaf65) 一文结合阅读。

>- 因为蚂蚁笔记官方图片服务器极其不稳定，如果本文出现图片加载不出的问题，请转到此链接查看： https://note.roger101.com/blog/post/snowming/9a21028cdb2e

# 0x01 MIC & DACL

在进程注入的时候，需要使用 OpenProcess 打开进程句柄，同时 dwDesiredAccess 参数一定会包括 PROCESS_VM_WRITE 特权。但是要使用写权限去打开另一个进程会有一些限制和保护机制，这些限制和保护机制包括：

- Mandatory Integrity Control (MIC)
- Protected Process(PP) & Protected Process Light(PPL)

本文中主要讨论 MIC。

>MIC is a protection method to control access to objects based on their "Integrity level".
There are 4 integrity levels:

>- Low Level for process which are restricted to access most of the system (for example Internet explorer)
- Medium Level is the default for any process started by unprivileged users and also administrator users if UAC is enabled.
- High level is for process running with administrator privileges
- System level are ran by SYSTEM users, generally the level of system services and process requiring the highest protection.

>For our concern that means the injector process will only be able to inject into a process running with inferior or equal integrity level. For example, if UAC is activated, even if user account is administrator a process will run at "Medium" integrity level. In addition to that, AppContainer process are also sandboxed and have restricted behaviors.

在 Windows 系统中，进程也是一个 Object。对进程的访问受到 `Discretionary access control list(DACL)` 的限制。

![title](https://leanote.com/api/file/getImage?fileId=5f675b98ab64412015001b07)

MSDN links:

- [DACLs and ACEs](https://docs.microsoft.com/en-us/windows/win32/secauthz/dacls-and-aces)
- [Creating a DACL](https://docs.microsoft.com/en-us/windows/win32/secbp/creating-a-dacl)


# 0x02 Win API & Win CRT 分析

![title](https://leanote.com/api/file/getImage?fileId=5f675c11ab64412015001b0b)


> [Enabling and Disabling Privileges in C++](https://docs.microsoft.com/en-us/windows/win32/secauthz/enabling-and-disabling-privileges-in-c--)

所谓绕过 MIC & DACL，实质上就是要提升当前进程的权限，使之有特权去更改我们需要操作的进程的状态。在枚举/结束系统进程或操作系统服务时，会出现自己权限不足而失败的情况，这时就需要提升自己的进程权限。

在进程注入的场景中，我们想要独立于 MIC & DACL 去对任何进程以 all access 调用 OpenProcess 打开进程句柄，那么我们需要启用 SeDebugPrivilege 特权。Windows以字符串的形式表示系统特权，如 SeCreatePagefilePrivilege 表示该特权用于创建页面文件，`SeDebugPrivilege` 表示该特权可用于调试及更改其它进程的内存，为了便于在代码中引用这些字符串，微软在 winnt.h 中定义了一组宏，如 #define SE_DEBUG_NAME TEXT("SeDebugPrivilege")。完整的特权列表可以查阅 msdn 的 security 一章。


![title](https://leanote.com/api/file/getImage?fileId=5f675caeab64411e11001b56)

>[Privilege Constants (Authorization)](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants)

要对一个任意进程（包括系统安全进程和服务进程）进行指定了写相关的访问权的 OpenProcess 操作，只要当前进程具有 SeDeDebug 权限就可以了。要是一个用户是 Administrator 或是被给予了相应的权限，就可以具有该权限。可是，就算我们用 Administrator 帐号对一个系统安全进程执行 OpenProcess(PROCESS_ALL_ACCESS,FALSE, dwProcessID) 还是会遇到「访问拒绝」的错误。什么原因呢？原来在默认的情况下进程的一些访问权限是没有被 Enable 的，所以我们要做的首先是 Enable 这些权限。与此相关的一些API函数有：

1. OpenProcessToken
2. LookupPrivilegevalue
3. AdjustTokenPrivileges

具体的 Win API 调用链为：



- 1、 **使用 OpenProcessToken 打开与本进程相关联的 `access token`**。windows的每个用户登录系统后，系统会产生一个访问令牌（access token） ，其中关联了当前用户的权限信息，用户登录后创建的每一个进程都含有用户access token的拷贝，当进程试图执行某些需要特殊权限的操作或是访问受保护的内核对象时，系统会检查其acess token中的权限信息以决定是否授权操作。Administrator组成员的access token中会含有一些可以执行系统级操作的特权（privileges） ，如终止任意进程、关闭/重启系统、加载设备驱动和更改系统时间等，不过这些特权默认是被禁用的，当Administrator组成员创建的进程中包含一些需要特权的操作时，进程必须首先打开这些禁用的特权以提升自己的权限，否则系统将拒绝进程的操作。注意，非Administrator组成员创建的进程无法提升自身的权限，因此下面提到的进程均指Administrator组成员创建的进程。

![title](https://leanote.com/api/file/getImage?fileId=5f675d70ab64411e11001b58)


- 2、 **使用 LookupPrivilegeValue 获取指定特权的 LUID，在这里指定特权是 TEXT("SeDebugPrivilege")。**因为虽然 Windows 使用字符串表示特权，但查询或更改特权的 API 需要 LUID 来引用相应的特权，LUID 表示 local unique identifier，它是一个64位值，在当前系统中是唯一的。为了提升进程权限到指定的特权，我们必须先找到该特权对应的 LUID，这时要调用 LookupPrivilegeValue 函数。
获得特权对应的 LUID 之后，我们要打开该特权。此时要用到 LUID_AND_ATTRIBUTES 结构，其定义如下：

```
typedef struct _LUID_AND_ATTRIBUTES {
    LUID Luid;
    DWORD Attributes;
} LUID_AND_ATTRIBUTES, * PLUID_AND_ATTRIBUTES;
```
Attributes 设为 SE_PRIVILEGE_ENABLED 时将打开 Luid 对应的特权。

- 3、 AdjustTokenPrivileges 对指定的 access token 启用特权，启用  SeDebugPrivilege 特权。设置完 LUID_AND_ATTRIBUTES 结构体之后，我们需要调用 AdjustTokenPrivileges 函数通知操作系统将指定的 access token 权限中的特权置为打开状态，前面我们说过，进程执行需要特列权限的操作时系统将检查其 access token，因此更改了进程的 access token 特权设置，也就是更改了所属进程的特权设置。

# 0x03 代码实现


MSDN 启用特权示例：

```
#include <windows.h>
#include <stdio.h>
#pragma comment(lib, "cmcfg32.lib")
 
BOOL SetPrivilege(
    HANDLE hToken,          // access token handle
    LPCTSTR lpszPrivilege,  // name of privilege to enable/disable
    BOOL bEnablePrivilege   // to enable or disable privilege
    ) 
{
    TOKEN_PRIVILEGES tp;
    LUID luid;
 
    if ( !LookupPrivilegeValue( 
            NULL,            // lookup privilege on local system
            lpszPrivilege,   // privilege to lookup 
            &luid ) )        // receives LUID of privilege
    {
        printf("LookupPrivilegeValue error: %u\n", GetLastError() ); 
        return FALSE; 
    }
 
    tp.PrivilegeCount = 1;
    tp.Privileges[0].Luid = luid;
    if (bEnablePrivilege)
        tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
    else
        tp.Privileges[0].Attributes = 0;
 
    // Enable the privilege or disable all privileges.
 
    if ( !AdjustTokenPrivileges(
           hToken, 
           FALSE, 
           &tp, 
           sizeof(TOKEN_PRIVILEGES), 
           (PTOKEN_PRIVILEGES) NULL, 
           (PDWORD) NULL) )
    { 
          printf("AdjustTokenPrivileges error: %u\n", GetLastError() ); 
          return FALSE; 
    } 
 
    if (GetLastError() == ERROR_NOT_ALL_ASSIGNED)
 
    {
          printf("The token does not have the specified privilege. \n");
          return FALSE;
    } 
 
    return TRUE;
}
```

示例二：

```
/*
Enable a privilege for the current process
*/
BOOL MagicSecurity::EnableWindowsPrivilege(TCHAR* Privilege)
{
     HANDLE token;
     TOKEN_PRIVILEGES priv;
     BOOL ret = FALSE;
     my_dbgprint(" [+] Enable %s privilege\n", Privilege);
     if (OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &token)) {
     priv.PrivilegeCount = 1;
     priv.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
         if (LookupPrivilegeValue(NULL, Privilege, &priv.Privileges[0].Luid) != FALSE &&
             AdjustTokenPrivileges(token, FALSE, &priv, 0, NULL, NULL) != FALSE) {
             ret = TRUE;
         }
         if (GetLastError() == ERROR_NOT_ALL_ASSIGNED) // In case privilege is not part of token (ex run as non admin)
         {
             ret = FALSE;
         }
         CloseHandle(token);
     }
     
     if (ret == TRUE)
        my_dbgprint(" [-] Sucess\n");
     else
        my_dbgprint(" [!] Failure\n");
     return ret;
}
 
Call it with:
MagicSecurity::EnableWindowsPrivilege((TCHAR *)TEXT("SeDebugPrivilege"));
```

**示例三：**

**这段示例是我自己封的方法，是为了启用当前进程的 SeDebugPrivilege 特权，已测试通过。**

```
#pragma comment(lib, "advapi32.lib")

//为当前进程赋予 SeDeDebug 特权
BOOL EnableSeDebug(void)
{
	//初始化变量
	HANDLE hToken = 0;
	TOKEN_PRIVILEGES tokenPrivileges = { 0 };
	LUID luid = { 0 };

	wchar_t* lpszPrivilege = _wcsdup(TEXT("SeDebugPrivilege"));
	
	//获取当前进程的进程句柄
	OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &hToken);   //通过&hToken来使用PHANDLE TokenHandle
	if (!LookupPrivilegeValue(
		NULL,             //在本地系统上查找特权名称
		lpszPrivilege,    //要提升至的权限
		&luid))           //接收权限的 LUID
	{
		printf("LookupPrivilege failed with System Error Code: %d.\n", GetLastError());
		return (FALSE);
	}
		
	
	tokenPrivileges.PrivilegeCount = 1;
	tokenPrivileges.Privileges[0].Luid = luid;
	tokenPrivileges.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
	
	if (!AdjustTokenPrivileges(hToken, FALSE, &tokenPrivileges, 0, NULL, NULL)) 
	{
		printf("AdjustTokenPrivileges failed with SYSTEM ERROR CODE: %u\n", GetLastError());
		return (FALSE);
	}
	
	return (TRUE);
}


int main(void)
{
	int result;
    result = EnableSeDebug();
	printf("result = %d\n", result);
	return 0;
}
```


# 0x04 参考文档

- [Code injection series part 1](https://blog.sevagas.com/IMG/pdf/code_injection_series_part1.pdf)
- [提升进程权限-OpenProcessToken等函数的用法](https://blog.csdn.net/stonesharp/article/details/7709674)