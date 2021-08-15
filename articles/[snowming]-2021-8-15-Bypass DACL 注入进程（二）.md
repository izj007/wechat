>- 因为蚂蚁笔记官方图片服务器极其不稳定，如果本文出现图片加载不出的问题，请转到此链接查看： 
https://shimo.im/docs/rKXY8V9rrYPt3xKK/ 《Bypass DACL 注入进程（二）》


在 [Bypass MIC & DACL 注入进程](http://blog.leanote.com/post/snowming/39b634f54089) 一文中，有几个需要注意的地方。


调用链是：
1. OpenProcessToken
2. LookupPrivilegevalue
3. AdjustTokenPrivileges

目的是使用 OpenProcess 使用写权限去打开远程进程句柄，通过对当前进程启用 `SeDebugPrivilege` 特权绕过远程进程内存保护的限制。


![](https://images-cdn.shimo.im/KuWBT2kVpv6i0poj__thumbnail.png)


但是启用此特权的前提是，当前进程具有此特权。


普通用户特权：

![Title](https://images-cdn.shimo.im/xFBnJkM8tbCIrOZb__thumbnail.png)

Administrator 组用户特权：


![Title](https://images-cdn.shimo.im/rYO0KY9GS3TE1p5L__thumbnail.png)

可以看到，Administrator 组的用户才具备此特权，只是被禁用了。所以我们需要通过调用 `AdjustTokenPrivileges` API 来启用此特权。

>AdjustTokenPrivileges 函数（securitybaseapi.h）
该 AdjustTokenPrivileges 功能启用或禁用特权在指定的访问令牌。启用或禁用访问令牌中的特权需要 TOKEN_ADJUST_PRIVILEGES 访问。

在**以普通用户运行的** Visual Studio 中执行以下测试代码，发现不启用 SeDebugPrivilege 时，无法打开 pid 为 5192 的进程句柄。

![](https://images-cdn.shimo.im/k1kzxJp1LHGJ8Uzj__thumbnail.png)


此进程为 Session 0 中的宿迁进程 `svchost.exe`。


![](https://images-cdn.shimo.im/5Xy8P5GjIcE1f0HP__thumbnail.png)


尝试启用 SeDebugPrivilege，发现依然无法打开 pid 为 5192 的进程句柄。

![](https://images-cdn.shimo.im/CIWnVKD2K1zFhZGq__thumbnail.png)


这是符合以上我们所说的 token 特权问题的。也就是普通用户根本就无 `SeDebugPrivilege` 这一特权，所以启用此特权也是无效的。

所以结论是：

>Administrator 组成员的 access token 中会含有一些可以执行系统级操作的特权（privileges） ，如终止任意进程、关闭/重启系统、加载设备驱动和更改系统时间等，不过这些特权默认是被禁用的，当 Administrator 组成员创建的进程中包含一些需要特权的操作时，进程必须首先打开这些禁用的特权以提升自己的权限，否则系统将拒绝进程的操作。
而非 Administrator 组成员创建的进程无法提升自身的权限。

另一个奇怪的现象是，在**以管理员身份运行的** Visual Studio 中执行以下测试代码，发现不启用 SeDebugPrivilege 时，也可以打开 pid 为 5192 的进程句柄。


![](https://images-cdn.shimo.im/aJFnfLxENZANZZEj__thumbnail.png)


这是为什么呢？


以普通用户身份运行的 Visual Studio 进程，其特权为：

![](https://images-cdn.shimo.im/MZqZ0nmKDp5yjBjI__thumbnail.png)

不包括  SeDebugPrivilege 特权。


但以管理员用户身份运行的 Visual Studio，其特权为：

![title](https://images-cdn.shimo.im/G1H28J21JmvOkSKL__thumbnail.png)

所以是因为 Visual Studio 已经启用了此特权。


直接运行 EXE 进行测试，当注释掉启用特权的代码时，打开进程句柄失败：

![](https://images-cdn.shimo.im/fFqRYT22s6zCkr7L__thumbnail.png)


当运行了启用特权的代码时，打开此 Session 0 中以 `NT AUTHORITY\SYSTEM` 用户创建的进程时，打开进程句柄成功：

![](https://images-cdn.shimo.im/3ienSYhdT4hh1SbH__thumbnail.png)



注：

文中对 Administrator 组用户启用 `SeDebugPrivilege` 特权的代码实现：

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

注2：

也可以给非 Administrator 用户组的用户加上此 SeDebugPrivilege 特权，但是需要特权来给用户加上此特权，又回到了先有鸡先有蛋的问题，故不再延伸讨论。

