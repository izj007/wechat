本文希望通过使用 Windbg 调试 kernel32.dll，查看其导出表、 探索 `IMAGE_EXPORT_DIRECTORY` 数据结构，理解以下三个表之间的相互关系：

- AddressOfFunctions
- AddressOfNames
- AddressOfNameOrdinals

便于更好的理解后续通过函数名称在内存中获取函数指针的 shellcode 编程。

---------------


# ① kernel32.dll 的 Export Directory






将 WinDbg attach 到 `C:\Windows\System32\notepad.exe`（x64） 程序上。
 


```
0:000> !peb
PEB at 00000093ec76a000
    InheritedAddressSpace:    No
    ReadImageFileExecOptions: No
    BeingDebugged:            Yes
    ImageBaseAddress:         00007ff626d20000
    NtGlobalFlag:             70
    NtGlobalFlag2:            0
    Ldr                       00007fff777053c0
    Ldr.Initialized:          Yes
    Ldr.InInitializationOrderModuleList: 000001986c942ef0 . 000001986c943770
    Ldr.InLoadOrderModuleList:           000001986c9430a0 . 000001986c94a760
    Ldr.InMemoryOrderModuleList:         000001986c9430b0 . 000001986c94a770
                    Base TimeStamp                     Module
            7ff626d20000 5e82e461 Mar 31 14:34:09 2020 C:\Windows\System32\notepad.exe
            7fff775a0000 443b1261 Apr 11 10:20:17 2006 C:\Windows\SYSTEM32\ntdll.dll
            7fff77400000 33adb7d2 Jun 23 07:40:02 1997 C:\Windows\System32\KERNEL32.DLL
            7fff74660000 eb8644a5 Mar 20 15:39:17 2095 C:\Windows\System32\KERNELBASE.dll
```

获取到 `KERNEL32.DLL` 的内存地址为 7fff77400000。


查看此 DLL 的头信息：

```
0:000> !dh 7fff77400000 -f

File Type: DLL
FILE HEADER VALUES
    8664 machine (X64)
       6 number of sections
33ADB7D2 time date stamp Mon Jun 23 07:40:02 1997

       0 file pointer to symbol table
       0 number of symbols
      F0 size of optional header
    2022 characteristics
            Executable
            App can handle >2gb addresses
            DLL

OPTIONAL HEADER VALUES
     20B magic #
   14.15 linker version
   74800 size of code
   38A00 size of initialized data
       0 size of uninitialized data
   17CC0 address of entry point
    1000 base of code
         ----- new -----
00007fff77400000 image base
    1000 section alignment
     200 file alignment
       3 subsystem (Windows CUI)
   10.00 operating system version
   10.00 image version
   10.00 subsystem version
   B2000 size of image
     400 size of headers
   BA886 checksum
0000000000040000 size of stack reserve
0000000000001000 size of stack commit
0000000000100000 size of heap reserve
0000000000001000 size of heap commit
    4160  DLL characteristics
            High entropy VA supported
            Dynamic base
            NX compatible
            Guard
   8EC80 [    DD40] address [size] of Export Directory
   9C9C0 [     744] address [size] of Import Directory
   B0000 [     520] address [size] of Resource Directory
   AA000 [    5424] address [size] of Exception Directory
   ACA00 [    3AB0] address [size] of Security Directory
   B1000 [     24C] address [size] of Base Relocation Directory
   7CFF0 [      54] address [size] of Debug Directory
       0 [       0] address [size] of Description Directory
       0 [       0] address [size] of Special Directory
       0 [       0] address [size] of Thread Storage Directory
   76780 [     108] address [size] of Load Configuration Directory
       0 [       0] address [size] of Bound Import Directory
   77160 [    2A10] address [size] of Import Address Table Directory
       0 [       0] address [size] of Delay Import Directory
       0 [       0] address [size] of COR20 Header Directory
       0 [       0] address [size] of Reserved Directory
```


获取到 Export Directory 的 RVA 为 0x8EC80。

-------------

# ② IMAGE_EXPORT_DIRECTORY 结构体分析

每个加载模块都有其自己的 Export Directory，`Export Directory` 是一个 `IMAGE_EXPORT_DIRECTORY` 结构体：

```
[StructLayout(LayoutKind.Sequential)]
public struct IMAGE_EXPORT_DIRECTORY
{
	public UInt32 Characteristics;
	public UInt32 TimeDateStamp;
	public UInt16 MajorVersion;
	public UInt16 MinorVersion;
	public UInt32 Name;
	public UInt32 Base;
	public UInt32 NumberOfFunctions;
	public UInt32 NumberOfNames;
	public UInt32 AddressOfFunctions;     // RVA from base of image
	public UInt32 AddressOfNames;     // RVA from base of image
	public UInt32 AddressOfNameOrdinals;  // RVA from base of image
} 
```


Winnt.h：
```
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD   Characteristics;
    DWORD   TimeDateStamp;
    WORD    MajorVersion;
    WORD    MinorVersion;
    DWORD   Name;
    DWORD   Base;
    DWORD   NumberOfFunctions;
    DWORD   NumberOfNames;
    DWORD   AddressOfFunctions;     // RVA from base of image
    DWORD   AddressOfNames;         // RVA from base of image
    DWORD   AddressOfNameOrdinals;  // RVA from base of image
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;

```


如上，此结构包含3个指向数组的指针，指针的值是这三个数组基于基地址的 RVA：

- AddressOfFunctions：这个数组用于存储所有导出函数的 RVA 地址。
- AddressOfNames：这是一个 RVA 数组，每个 RVA 对应一个导出函数名称。
- AddressOfNameOrdinals：此数组与 AddressOfNames 数组存在一对一的对应关系。


![title](https://leanote.com/api/file/getImage?fileId=60658be3ab64417533000361)


查看 `Export Directory` 处的内存：

```
0:000> dd 7fff77400000+8EC80
00007fff`7748ec80  00000000 33adb7d2 00000000 00092c4a
00007fff`7748ec90  00000001 0000065d 0000065d 0008eca8
00007fff`7748eca0  0009061c 00091f90 00092c6f 00092ca5
00007fff`7748ecb0  0001e690 0001a9a0 000216a0 00010890
00007fff`7748ecc0  00022bc0 00022bd0 00092d2b 00036c50
00007fff`7748ecd0  00052be0 00052c40 0001fe90 0001cfc0
00007fff`7748ece0  000355e0 0001ed40 000355f0 00033ca0
00007fff`7748ecf0  00092e64 00092ea4 00003a60 00022810
```

根据 `IMAGE_EXPORT_DIRECTORY` 结构体的定义可得：

- Characteristics = 00000000
- TimeDateStamp = 33adb7d2
- MajorVersion = 0000
- MinorVersion = 0000
- Name = 00092c4a
- Base = 00000001
- NumberOfFunctions = 0000065d
- NumberOfNames = 0000065d
- AddressOfFunctions = 0008eca8
- AddressOfNames = 0009061c 
- AddressOfNameOrdinals = 00091f90


先来查看 AddressOfNames 数组的内存（如上， RVA 0009061c）：

这一块内存是一个 RVA 数组，里面每项是导出函数名字符串的 RVA。如：

```
0:000> dd 7fff77400000+0009061c
00007fff`7749061c  00092c57 00092c90 00092cc3 00092cd2
00007fff`7749062c  00092ce7 00092cf0 00092cf9 00092d0a
00007fff`7749063c  00092d1b 00092d60 00092d86 00092da5
00007fff`7749064c  00092dc4 00092dd1 00092de4 00092dfc
00007fff`7749065c  00092e17 00092e2c 00092e49 00092e88
00007fff`7749066c  00092ec9 00092edc 00092ee9 00092f03
00007fff`7749067c  00092f21 00092f58 00092f9d 00092fe8
00007fff`7749068c  00093043 00093098 000930eb 00093140

0:000> da 7fff77400000+00092c57
00007fff`77492c57  "AcquireSRWLockExclusive"

0:000> da 7fff77400000+00092c90
00007fff`77492c90  "AcquireSRWLockShared"

0:000> da 7fff77400000+00092cc3
00007fff`77492cc3  "ActivateActCtx"

0:000> da 7fff77400000+00092cd2
00007fff`77492cd2  "ActivateActCtxWorker"

0:000> da 7fff77400000+00092ce7
00007fff`77492ce7  "AddAtomA"
```

在这里  `NumberOfFunctions` 与 `NumberOfNames` 的值指向同一个内存地址，说明是一样的。但其实这两个值不一定相等，因为并不是全部导出函数都一定有名称，如下图有的导出函数只有序号：


![title](https://leanote.com/api/file/getImage?fileId=606308e2ab64414ea100074f)



再来查看 AddressOfFunctions 数组中的元素。前面已经获知 NumberOfFunctions 的 RVA 为 0008eca8。在 WINDBG 中查看此数组的元素：


```
0:000> dd 7fff77400000+0008eca8
00007fff`7748eca8  00092c6f 00092ca5 0001e690 0001a9a0
00007fff`7748ecb8  000216a0 00010890 00022bc0 00022bd0
00007fff`7748ecc8  00092d2b 00036c50 00052be0 00052c40
00007fff`7748ecd8  0001fe90 0001cfc0 000355e0 0001ed40
00007fff`7748ece8  000355f0 00033ca0 00092e64 00092ea4
00007fff`7748ecf8  00003a60 00022810 00035610 00035600
00007fff`7748ed08  00092f37 00092f75 00092fbd 00093010
00007fff`7748ed18  00093068 000930bc 00093110 0009315b

0:000> u 7fff77400000+00092c6f
KERNEL32!_xmmc09e3889374bc6a8405f39999999999a+0xaa6f:
00007fff`77492c6f 4e54            push    rsp
00007fff`77492c71 44              ???
00007fff`77492c72 4c              ???
00007fff`77492c73 4c              ???
00007fff`77492c74 2e52            push    rdx
00007fff`77492c76 746c            je      KERNEL32!_xmmc09e3889374bc6a8405f39999999999a+0xaae4 (00007fff`77492ce4)
00007fff`77492c78 41637175        movsxd  esi,dword ptr [r9+75h]
00007fff`77492c7c 6972655352574c  imul    esi,dword ptr [rdx+65h],4C575253h

0:000> u 7fff77400000+00092ca5
KERNEL32!_xmmc09e3889374bc6a8405f39999999999a+0xaaa5:
00007fff`77492ca5 4e54            push    rsp
00007fff`77492ca7 44              ???
00007fff`77492ca8 4c              ???
00007fff`77492ca9 4c              ???
00007fff`77492caa 2e52            push    rdx
00007fff`77492cac 746c            je      KERNEL32!_xmmc09e3889374bc6a8405f39999999999a+0xab1a (00007fff`77492d1a)
00007fff`77492cae 41637175        movsxd  esi,dword ptr [r9+75h]
00007fff`77492cb2 6972655352574c  imul    esi,dword ptr [rdx+65h],4C575253h

0:000> u 7fff77400000+0001e690
KERNEL32!ActivateActCtxStub:
00007fff`7741e690 48ff2591a30500  jmp     qword ptr [KERNEL32!_imp_ActivateActCtx (00007fff`77478a28)]
00007fff`7741e697 cc              int     3
00007fff`7741e698 cc              int     3
00007fff`7741e699 cc              int     3
00007fff`7741e69a cc              int     3
00007fff`7741e69b cc              int     3
00007fff`7741e69c cc              int     3
00007fff`7741e69d cc              int     3
```

通过查看反汇编代码可以看出每个元素的确是导出函数地址的 RVA。


通过上面两个数组的查询，我们可以拼凑出这样一张具有函数地址和函数名称的表：

|函数地址   |   函数名称 | 
| :-------- |: --------| 
|7fff77400000+00092c6f |AcquireSRWLockExclusive | 
| 7fff77400000+00092ca5  |  AcquireSRWLockShared | 
|  7fff77400000+0001e690 |    ActivateActCtx | 


但现在其实存在问题：

实际上，我们目前尚不知道这些地址是否实际上都与给定名称匹配，因为由 AddressOfNameOrdinals 表负责确定哪个函数名称属于哪个地址。


接下来需要分析 AddressOfNameOrdinals 数组。到目前为止，我们已经提到 AddressOfNameOrdinals 数组的第一个元素应包含索引0，因为 AddressOfNames 数组中的第一个元素指向 AddressOfFunctions 数组中的第一个元素。AddressOfNames 中的第二个元素指向 AddressOfFunctions 中的第二个元素，以此类推。为验证这一点，可在内存中查看 AddressOfNameOrdinals 数组。根据我们刚刚查询的数组 RVA：

```
AddressOfNameOrdinals = 00091f90
```

因为 AddressOfNameOridinals 数组中的值是 16位（字而非双字）的序号，所以使用 `dw` 命令（`WORD`）而不是 `dd` 命令：

可以看到里面都是导出函数的序号：





```
0:000> dw 7fff77400000+00091f90
00007fff`77491f90  0000 0001 0002 0003 0004 0005 0006 0007
00007fff`77491fa0  0008 0009 000a 000b 000c 000d 000e 000f
00007fff`77491fb0  0010 0011 0012 0013 0014 0015 0016 0017
00007fff`77491fc0  0018 0019 001a 001b 001c 001d 001e 001f
00007fff`77491fd0  0020 0021 0022 0023 0024 0025 0026 0027
00007fff`77491fe0  0028 0029 002a 002b 002c 002d 002e 002f
00007fff`77491ff0  0030 0031 0032 0033 0034 0035 0036 0037
00007fff`77492000  0038 0039 003a 003b 003c 003d 003e 003f
```

可以看出这个 dll 中，导出函数是按顺序排列的，所以这个 dll 里面函数名和函数名称也是一一对应的。

-------------

# ③ 根据函数名获取函数地址

思考一下，如何获取内存中导出函数的地址呢？


![title](https://leanote.com/api/file/getImage?fileId=606308e2ab64414ea100074f)


- 第一种情况，已知导出函数序号（`Oridinal`）：

    1. 获取 AddressofFunctions 的内存地址。并获取第一个 addr0 的地址。
    2. 根据函数序号计算距离 addr0 偏移了多少个元素。得出函数地址。
    
    
- 第二种情况，已知函数名称：
    1. 在 AddressOfNames 数组中遍历，找到对应的函数名称的数组下标。
    2. 在 AddressOfNameOrdinals 数组中查看对应下标的元素的值，即为数组偏移。
    3. 在 AddressOfFunctions 数组中根据偏移计算元素位置。得出函数地址。

    

```
Export Names Table > Export Ordinals Table -> Export Address Table = Function Address (VA)
```

在本文的例子中，假设我们想寻找 `AddAtomA` 函数的函数地址，流程如下：

① 在 `AddressOfNames` 数组中遍历和比对函数名，找到数组下标。
 
AddressOfNames：

![title](https://leanote.com/api/file/getImage?fileId=60653c18ab64417737000131)


```
0:000> dd 7fff77400000+0009061c
00007fff`7749061c  00092c57 00092c90 00092cc3 00092cd2
00007fff`7749062c  00092ce7 00092cf0 00092cf9 00092d0a
00007fff`7749063c  00092d1b 00092d60 00092d86 00092da5
00007fff`7749064c  00092dc4 00092dd1 00092de4 00092dfc
00007fff`7749065c  00092e17 00092e2c 00092e49 00092e88
00007fff`7749066c  00092ec9 00092edc 00092ee9 00092f03
00007fff`7749067c  00092f21 00092f58 00092f9d 00092fe8
00007fff`7749068c  00093043 00093098 000930eb 00093140
0:000> da 7fff77400000+00092c57
00007fff`77492c57  "AcquireSRWLockExclusive"
0:000> da 7fff77400000+00092c90
00007fff`77492c90  "AcquireSRWLockShared"
0:000> da 7fff77400000+00092cc3
00007fff`77492cc3  "ActivateActCtx"
0:000> da 7fff77400000+00092cd2
00007fff`77492cd2  "ActivateActCtxWorker"
0:000> da 7fff77400000+00092ce7
00007fff`77492ce7  "AddAtomA"
```

可以看到数组下标4。这也对应了在 `AddressOfNameOrdinals` 中下标为4。




② 在 `AddressOfNameOrdinals` 查询对应的值。

AddressOfNameOrdinals：

![title](https://leanote.com/api/file/getImage?fileId=60653d74ab64417533000230)

```
0:000> dw 7fff77400000+00091f90
00007fff`77491f90  0000 0001 0002 0003 0004 0005 0006 0007
00007fff`77491fa0  0008 0009 000a 000b 000c 000d 000e 000f
00007fff`77491fb0  0010 0011 0012 0013 0014 0015 0016 0017
00007fff`77491fc0  0018 0019 001a 001b 001c 001d 001e 001f
00007fff`77491fd0  0020 0021 0022 0023 0024 0025 0026 0027
00007fff`77491fe0  0028 0029 002a 002b 002c 002d 002e 002f
00007fff`77491ff0  0030 0031 0032 0033 0034 0035 0036 0037
00007fff`77492000  0038 0039 003a 003b 003c 003d 003e 003f
```
![title](https://leanote.com/api/file/getImage?fileId=60653ef4ab64417533000233)


 
下标为4的元素对应的序号为 0004。



③ 在 `AddressOfFunctions` 中查询函数地址。

AddressOfFunctions：

![title](https://leanote.com/api/file/getImage?fileId=60654040ab64417533000240)


```
0:000> dd 7fff77400000+0008eca8
00007fff`7748eca8  00092c6f 00092ca5 0001e690 0001a9a0
00007fff`7748ecb8  000216a0 00010890 00022bc0 00022bd0
00007fff`7748ecc8  00092d2b 00036c50 00052be0 00052c40
00007fff`7748ecd8  0001fe90 0001cfc0 000355e0 0001ed40
00007fff`7748ece8  000355f0 00033ca0 00092e64 00092ea4
00007fff`7748ecf8  00003a60 00022810 00035610 00035600
00007fff`7748ed08  00092f37 00092f75 00092fbd 00093010
00007fff`7748ed18  00093068 000930bc 00093110 0009315b
```

查看下标4的元素， RVA 为 000216a0：

```
0:000> u 7fff77400000+000216a0
KERNEL32!AddAtomA:
00007fff`774216a0 4c8bc1          mov     r8,rcx
00007fff`774216a3 4533c9          xor     r9d,r9d
00007fff`774216a6 b101            mov     cl,1
00007fff`774216a8 33d2            xor     edx,edx
00007fff`774216aa e919f2feff      jmp     KERNEL32!InternalAddAtom (00007fff`774108c8)
00007fff`774216af cc              int     3
00007fff`774216b0 cc              int     3
00007fff`774216b1 cc              int     3
```

验证的确是 `AddAtomA` 函数的地址。


-------------


# ④ 参考文档


- [Deep Dive into OS Internals with Windbg](https://www.exploit-db.com/docs/english/18576-deep-dive-into-os-internals-with-windbg.pdf)
- [The Export Directory](https://resources.infosecinstitute.com/topic/the-export-directory/)