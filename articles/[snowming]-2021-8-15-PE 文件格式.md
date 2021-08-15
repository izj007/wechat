
- 文件中使用偏移（`offset`）指示位置
- 内存中使用 VA（`Virtual Address`）指示位置


![title](https://leanote.com/api/file/getImage?fileId=5eff0058ab64415e130008f5)




# 0x01：DOS 头：IMAGE_DOS_HEADER

大小64字节，偏移区域为 00000000 - 00000003F

![title](https://leanote.com/api/file/getImage?fileId=5efd8b29ab64411e26001660)

- PE 文件的 DOS 头是为了兼容 DOS 文件；
- 位于 PE 文件的最前面；





|值|结构体成员|大小|注释|
|:--|:--|:--|:--|
|4d5a|**e_magic**|WORD 2字节|DOS 签名=> ASCII 值 `MZ`|
|0090|e_cblp|WORD 2字节||
|0003|e_cp|WORD 2字节||
|0000|e_crlc|WORD 2字节||
|0004|e_cparhdr|WORD 2字节||
|0000|e_minalloc|WORD 2字节||
|ffff|e_maxalloc|WORD 2字节||
|0000|e_ss|WORD 2字节||
|00b8|e_sp|WORD 2字节||
|0000|e_csum|WORD 2字节||
|0000|e_ip|WORD 2字节||
|0000|e_cs|WORD 2字节||
|0040|e_lfarlc|WORD 2字节||
|0000|e_ovno|WORD 2字节||
|0000000000000000|e_res[4]|WORD数组，4个元素，8字节||
|0000|e_oemid|WORD 2字节||
|0000|e_oeminfo|WORD 2字节||
|0000000000000000000000000000000000000000|e_res2[10]|WORD数组，10个元素，20字节||
|000000f0|**e_lfanew**|LONG 4字节|指示NT头的偏移|

>注：小端序标识法


# 0x02：DOS stub

- DOS stub 位于 DOS 头下方，是个可选项，大小不固定。
- DOS stub 由代码与数据混合而成。

![title](https://leanote.com/api/file/getImage?fileId=5efd97f7ab64411e2600172e)

这里 00000040 - 000000ef 区域为 DOS stub，因为 DOS stub 位于 DOS 头下方，NT 头之上。在 `IMAGE_DOS_HEADER` 结构体中的 `e_lfanew` 成员指定了 NT 头的偏移为 000000f0。这两个范围中间就是 DOS stub 的偏移区域。

文件偏移 40~4D 区域为16位的汇编指令。32位的 Windows OS 中不会运行该命令（由于被识别为 PE 文件，所以完全忽视该代码）。

```
0e              PUSH  CS
1f              POP   DS
ba0e00          MOV   DX,0E     ; DX = 0E:"This program cannot be run in DOS mode"
b409            MOV   AH,09
cd21            INT   21        ; AH = 09:WriteString()
b8014c          MOV   AX,4C01   
cd21            INT   21        ; AH = 4C01:Exit() 
```



# 0x03：NT 头：IMAGE_NT_HEADERS 结构体


IMAGE_NT_HEADERS 结构体的大小为 F8。


![title](https://leanote.com/api/file/getImage?fileId=5efda7ceab64411e26001810)


```
typedef struct _IMAGE_NT_HEADERS{
    DWORD Signature;     //PE Signature : 50450000("PE"00)
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32
```

- 第一个成员为签名结构体，其值为 50450000h（`PE00`）。
- 第二个成员为文件头 `FileHeader`；
- 第三个成员为可选头 `OptionalHeader`。


## 3.1 NT 头：文件头 IMAGE_FILE_HEADER 结构体


![title](https://leanote.com/api/file/getImage?fileId=5efdaba1ab64411e2600185d)




|值|结构体成员|大小|注释|
|:--|:--|:--|:--|
|014c|**Machine**|WORD 2字节|每个CPU都拥有唯一的Machine码，兼容32位Intel x86芯片的 Machine 码为 14C|
|0006|**NumberOfSections**|WORD 2字节|文件中存在的节区数量|
|93b4e8fa|TimeDateStamp|DWORD 4字节|此值不影响文件的运行，只用来记录编译器创建此文件的时间|
|00000000|PointerToSymbolTable|DWORD 4字节||
|00000000|NumberOfSymbols|DWORD 4字节||
|00e0|**SizeOfOptionalHeader**|WORD 2字节 |指出 IMAGE_NT_HEADERS 结构体中可选头结构体的大小。E0 是 IMAGE_OPTIONAL_HEADER32 结构体的大小|
|0102|**Characteristics**|WORD 2字节|用于标识文件的属性，文件是否是可运行的状态、是否为DLL文件等信息；<br/>0102h由0100h和0002h bit OR组合而成；<br/>0002h `IMAGE_FILE_EXECUTABLE_IMAGE` 代表文件可执行；<br/>0100h `IMAGE_FILE_32BIT_MACHINE` 代表32 bit word machine。|
>注：小端序标识法



## 3.2 NT 头：可选头：IMAGE_OPTIONAL_HEADER32 结构体


IMAGE_OPTIONAL_HEADER32 是 PE 头结构体中最大的。



![title](https://leanote.com/api/file/getImage?fileId=5efdbce7ab64411c280019b1)





|值|结构体成员|大小|注释|
|:--|:--|:--|:--|
|010b|**Magic**|WORD 2字节|为 IMAGE_OPTIONAL_HEADER32 结构体是，Magic 码为10b；为 IMAGE_OPTIONAL_HEADER64 结构体时，Magic 码为 20B|MajorLinkerVersion|BYTE 1字节||
|0f|MinorLinkerVersion|BYTE 1字节||
|0001fc00|SizeOfCode|DWORD 4字节||
|00007400|SizeOfInitializedDate|DWORD 4字节||
|00000000|SizeOfUninitializedData|DWORD 4字节||
|0001f8d0|**AddressOfEntryPoint**|DWORD 4字节|持有 EP 的 RVA 值。该值指出程序最先执行的代码起始地址，非常重要。|
|00001000|BaseOfCode|DWORD 4字节||
|00021000|BaseOfData|DWORD 4字节||
|00400000|**ImageBase**|DWORD 4字节|执行 PE 文件时，PE 装载器先创建进程，再将文件载入内存，然后把 EIP 寄存器的值设置为 ImageBase+AddressOfEntryPoint|
|00001000|**SectionAlignment**|DWORD 4字节|指定节区在内存中的最小单位。内存的节区大小是 SectionAlignment 值的整数倍|
|00000200|**FileAlignment**|DWORD 4字节|指定节区在磁盘文件中的最小单位。磁盘文件的节区大小是 FileAlignment 值的整数倍|
|000a|MajorOperatingSystemVersion|WORD 2字节||
|0000|MinorOperatingSystemVersion|WORD 2字节||
|000a|MajorImageVersion|WORD 2字节||
|0000|MinorImageVersion|WORD 2字节||
|000a|MajorSubsystemVersion|WORD 2字节||
|0000|MinorSubsystemVersion|WORD 2字节||
|00000000|Win32VersionValue|DWORD 4字节||
|0002b000|**SizeOfImage**|DWORD 4字节|加载 PE 文件到内存时，SizeOfImage 指定了 PE Image 在虚拟内存中所占空间的大小|
|00000400|**SizeOfHeaders**|DWORD 4字节|此值用于指出整个 PE 头的大小。这个值也必须是 FileAlignment 的整数倍。第一节区所在位置与 SizeOfHeaders 距文件开始偏移的量相同|
|00032822|CheckSum|DWORD 4字节||
|0002|**Subsystem**|WORD 2字节|用来区分系统驱动文件（*.sys）与普通的可执行文件。在这里 notepad.exe 属于窗口应用程序所以值为2|
|c140|DllCharacteristics|WORD 2字节||
|00040000|SizeOfStackReserve|DWORD 4字节||
|00011000|SizeOfStackCommit|DWORD 4字节||
|00100000|SizeOfHeapReserve|DWORD 4字节||
|00001000|SizeOfHeapCommit|DWORD 4 字节||
|00000000|LoderFlags|DWORD 4字节||
|00000010|**NumberOfRvaAndizes**|DOWRD 4字节|用于指定 DataDirectory(IMAGE_OPTIONAL_HEADER32 结构体的最后一个成员）数组的个数|
|00000000|**RVA of EXPORT Directory**|||
|00000000|**size of EXPORT Directory**|||
|000234b8|**RVA of IMPORT Directory**|||
|00000370|**size of IMPORT Directory**|||
|00027000|RVA of RESOURCE Directory|||
|00000be0|size of RESOURCE Directory|||
|00000000|RVA of EXCEPTION Directory|||
|00000000|size of EXCEPTION Directory|||
|00000000|RVA of SECURITY Directory|||
|00000000|size of SECURITY Directory|||
|00028000|RVA of BASERELOC Directory|||
|000021a8|size of BASERELOC Directory|||
|00004a80|RVA of DEBUG Directory|||
|00000054|size of DEBUG Directory|||
|00000000|RVA of COPYRIGHT Directory|||
|00000000|size of COPYRIGHT Directory|||
|00000000|RVA of GLOBALPTR Directory|||
|00000000|size of GLOBALPTR Directory|||
|000013d4|**RVA of TLS Directory**|||
|00000018|**size of TLS Directory**|||
|00001330|RVA of LOAD_CONFIG Directory|||
|000000a4|size of LOAD_CONFIG Directory|||
|00000000|RVA of BOUND_IMPORT Directory|||
|00000000|size of BOUND_IMPORT Directory|||
|00023000|RVA of IAT Directory|||
|000004b4|size of IAT Directory|||
|000207a4|RVA of DELAY_IMPORT Directory|||
|000000e0|size of DELAY_IMPORT Directory|||
|00000000|RVA of COM_DESCRIPTOR Directory|||
|00000000|size of COM_DESCRIPTOR Directory|||
|00000000|RVA of Reserved Directory|||
|00000000|size of Reserved Directory|||
>注：小端序标识法





# 0x04：节区头

- 节区头中有各节区（code\date\resource）的文件/内存的起始位置、大小、访问权限 等。

节区头是由 `IMAGE_SECTION_HEADER` 结构体组成的数组，每个结构体对应一个节区。

IMAGE_SECTION_HEADER 结构体：

```
#define IMAGE_SIZEOF_SHORT_NAME

typedef struct _IMAGE_SECTION_HEADER {
    BYTE Name[IMAGE_SIZEOF_SHORT_NAME];  //[]为数组，这是一个BYTE数组
    union {  //这是一个DWORD联合体，按中间最大的算。可解释为 PA，也可解释为 VS
        DWORD PhysicalAddress; 
        DWORD VirtualSize;
    } Misc; //联合体名字
    DWORD VirtualAddress;
    DWORD SizeOfRawData;
    DWORD PointerToRawData;
    DWORD PointerToRelocations;
    DWORD PointerToLinenumbers;
    WORD NumberOfRelocations;
    WORD NumberOfLinenumbers;
    DWORD Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```


notepad.exe 中的节区头数组（共6个节区）：


![title](https://leanote.com/api/file/getImage?fileId=5f0128e8ab64411f37000304)

`IMAGE_SECTION_HEADER` 结构体数组的实际值：



|值|结构体成员|大小|注释|
|:--|:--|:--|:--|
|2e74657874000000|Name|BYTE 字节数组8元素|**.text**|
|0001fb50|**VirtualSize**|DWORD 4字节|内存中节区所占大小|
|00001000|**VirtualAddress**|DWORD 4字节|内存中节区起始地址（RVA）|
|0001fc00|**SizeOfRawData**|DWORD 4字节|磁盘文件中节区所占大小|
|00000400|**PointerToRawData**|DWORD 4字节|磁盘文件中节区起始位置|
|00000000|PointerToRelocations|DWORD 4字节||
|00000000|PointerToLinenumbers |DWORD 4字节||
|0000 |NumberOfRelocations |WORD 2字节||
|0000    | NumberOfLinenumbers|WORD 2字节||
|60000020   | Characteristics|DWORD 4字节|`IMAGE_SCN_CNT_CODE`//节区包含代码<br/>`IMAGE_SCN_MEM_EXECUTE`//节区可执行<br/>`IMAGE_SCN_MEM_READ`//节区可读<br/>bit OR 而成，符合 code 内存属性的访问权限|
|2e64617461000000|Name|Byte 字节数组8元素|**.data**|
|00001db0|**VirtualSize** | DWORD 4字节|内存中节区所占大小 |
|00021000|**VirtualAddress**| DWORD 4字节|内存中节区起始地址（RVA）|
|00000800|**SizeOfRawData**|DWORD 4字节|磁盘文件中节区所占大小|
|00020000| **PointerToRawData**|DWORD 4字节|磁盘文件中节区起始位置|
|00000000|PointerToRelocations |DWORD 4字节||
|00000000|PointerToLinenumbers|DWORD 4 字节|| 
|0000|NumberOfRelocations|WORD 2字节||
|0000|NumberOfLinenumbers|WORD 2字节||
|c0000040|Characteristics|    DWORD 4字节|`IMAGE_SCN_CNT_INITIALIZED_DATA`//节区包含初始化的数据<br/>`IMAGE_SCN_MEM_READ`//节区可读<br/>`IMAGE_SCN_MEM_WRITE`//节区可写<br/>bit OR 而成，符合 data 内存属性的访问权限|
|2e69646174610000|Name|Byte 字节数组8元素|**.idata**|
|00002472|**VirtualSize** | DWORD 4字节|内存中节区所占大小 |
|00023000|**VirtualAddress**| DWORD 4字节|内存中节区起始地址（RVA）|
|00002600|**SizeOfRawData**|DWORD 4字节|磁盘文件中节区所占大小|
|00020800| **PointerToRawData**|DWORD 4字节|磁盘文件中节区起始位置|
|00000000|PointerToRelocations |DWORD 4字节||
|00000000|PointerToLinenumbers|DWORD 4字节|| 
|0000|NumberOfRelocations|WORD 2字节||
|0000|NumberOfLinenumbers|WORD 2字节||
|40000040|Characteristics|    DWORD 4字节|`IMAGE_SCN_CNT_INITIALIZED_DATA`//节区包含初始化的数据<br/>`IMAGE_SCN_MEM_READ`//节区可读<br/>bit OR 而成|
|2e64696461740000|Name|Byte 字节数组8元素|**.didat**|
|00000078|**VirtualSize** | DWORD 4字节|内存中节区所占大小 |
|00026000|**VirtualAddress**| DWORD 4字节|内存中节区起始地址（RVA）|
|00000200|**SizeOfRawData**|DWORD 4字节|磁盘文件中节区所占大小|
|00022e00| **PointerToRawData**|DWORD 4字节|磁盘文件中节区起始位置|
|00000000|PointerToRelocations |DWORD 4字节||
|00000000|PointerToLinenumbers|DWORD 4字节|| 
|0000|NumberOfRelocations|WORD 2字节||
|0000|NumberOfLinenumbers|WORD 2字节||
|c0000040|Characteristics|    DWORD 4字节|`IMAGE_SCN_CNT_INITIALIZED_DATA`//节区包含初始化的数据<br/>`IMAGE_SCN_MEM_READ`//节区可读<br/>`IMAGE_SCN_MEM_WRITE`//节区可写<br/>bit OR 而成|
|2e72737263000000|Name|Byte 字节数组8元素|**.rsrc**|
|00000be0|**VirtualSize** | DWORD 4字节|内存中节区所占大小 |
|00027000|**VirtualAddress**| DWORD 4字节|内存中节区起始地址（RVA）|
|00000c00|**SizeOfRawData**|DWORD 4字节|磁盘文件中节区所占大小|
|00023000| **PointerToRawData**|DWORD 4字节|磁盘文件中节区起始位置|
|00000000|PointerToRelocations |DWORD 4字节||
|00000000|PointerToLinenumbers|DWORD 4字节|| 
|0000|NumberOfRelocations|WORD 2字节||
|0000|NumberOfLinenumbers|WORD 2字节||
|40000040|Characteristics|    DWORD 4字节|`IMAGE_SCN_CNT_INITIALIZED_DATA`//节区包含初始化的数据<br/>`IMAGE_SCN_MEM_READ`//节区可读<br/>bit OR 而成，符合 rsrc 资源内存属性的访问权限|
|2e72656c6f630000|Name|Byte 字节数组8元素|**.reloc**|
|000021a8|**VirtualSize** | DWORD 4字节|内存中节区所占大小 |
|00028000|**VirtualAddress**| DWORD 4字节|内存中节区起始地址（RVA）|
|00002200|**SizeOfRawData**|DWORD 4字节|磁盘文件中节区所占大小|
|00023c00| **PointerToRawData**|DWORD 4字节|磁盘文件中节区起始位置|
|00000000|PointerToRelocations |DWORD 4字节||
|00000000|PointerToLinenumbers|DWORD 4字节|| 
|0000|NumberOfRelocations|WORD 2字节||
|0000|NumberOfLinenumbers|WORD 2字节||
|42000040|Characteristics|    DWORD 4字节|`INITIALIZED_DATA`<br/>`DISCARDABLE`<br/>`READ`<br/>ALIGN_DEFAULT(16)|


>注：小端序标识法



Image 映像：

>注：讲解 PE 文件时经常出现「映像」（`Image`）这一术语。PE 文件加载到内存时，文件不会原封不动的加载，而是要根据节区头中定义的节区起始地址、节区大小等加载。因此，磁盘文件中的 PE 与内存中的 PE 具有不同形态。将装载到内存中的形态成为 Image（映像）以示区别。


综上：

notepad.exe 中 PE 头的偏移区域为 `00000000` - `000002D7`。

![title](https://leanote.com/api/file/getImage?fileId=5f01297dab64411d380002f1)



# 0x05：IAT：IMAGE_IMPORT_DESCRIPTOR 结构体

 1. IAT，Import Address Table 导出地址表。IAT 是一种表格，用来记录程序正在使用哪些库中的哪些函数。
 2. `IMAGE_IMPORT_DESCRIPTOR` 结构体中记录着 PE 文件要导入哪些库文件。导入多少库就存在多少个 IMAGE_IMPORT_DESCRIPTOR 结构体。

**PE 转载器把导入函数输入至 IAT 的顺序**

 1. 读取 IID 的 Name 成员，获取库名称字符串（`kernel32.dll`）。
 2. 装载相应库。 → LoadLibrary("kernel32.dll")
 3. 读取 IID 的 OriginalFirstThunk 成员，获取 INT 地址。
 4. 逐一读取 INT 中数组的值，获取相应 IMAGE_IMPORT_BY_NAME 地址（RVA）。
 5. 使用 IMAGE_IMPORT_BY_NAME 的 Hint（Ordinal） 或 Name 项，获取相应函数的起始地址。
 6. 读取 IID 的 FirstThunk（IAT）成员，获得 IAT 地址。
 7. 将上面获得的函数地址输入相应 IAT 数组值。
 8. 重复以上步骤 4~7，直至 INT 结束（遇到 NULL 时）。



`IMAGE_IMPORT_DESCRIPTOR` 结构体数组不在 PE 头而在 PE 体中，但其查找位置在 PE 头中。

在 NT 头、可选头 `IMAGE_OPTIONAL_HEADER32` 结构体中，有以下两个结构体成员：

|值|结构体成员|大小|备注|
|:--:|:--:|:--:|:--:|:--:|
|000234b8|**RVA of IMPORT Directory**|||
|00000370|**size of IMPORT Directory**|||
 
 
 

IMAGE_OPTIONAL_HEADER32.DataDirectory[1].VirtualAddress （DataDirectory 数组的第 0 个元素是 EXPORT...）的值即是 IMAGE_IMPORT_DESCRIPTOR 结构体数组（也被称为 IMPORT Directory Table）的起始位置（RVA值）。




RVA 是234b8，参考 `IMAGE_SECTION_HEADER` 结构体数组，位于第 3 个节区，即 `.idata`。RAW=RVA-VirtualAddress+PointerToRawData=234b8-23000+20800=20cb8


![title](https://leanote.com/api/file/getImage?fileId=5f01b618ab64411d38000885)



notepad.exe 文件的第一个 IMAGE_IMPORT_DESCRIPTOR 结构体


|成员偏移|成员|RVA|RAW|
|:--:|:--:|:--:|:--:|
|20cb8|OriginalFirstThunk(INT)|00023834|21034|
|20cbc|TimeDateStamp|00000000|-|
|20cc0|ForwarderChain|00000000|-|
|20cc4|Name|00023e20|**21620**|
|20cc8|FirstThunk(IAT)|0002300c|2080c|

**库名称（Name）**

![title](https://leanote.com/api/file/getImage?fileId=5f01b9e5ab64411d380008a2)

Name 是一个字符串指针，它指向导入函数所属的库文件名称。在文件偏移 21620（RVA:23E20→RAW:21620）处可以看到字符串 `GDI32.dll`。


**OrignalFirstThunk-INT**

INT 是一个包含导入函数信息（Ordinal，Name）的结构体指针数组。只有获得了这些信息才能在加载到进程内存的库中准确求得相应函数的起始位置。

跟踪 OriginalFirstThunk 成员（RVA：23838→Raw：21034）。

![title](https://leanote.com/api/file/getImage?fileId=5f01bc31ab64411d380008b4)

如图是 INT，由地址数组形式组成（数组尾部以 NULL 结束）。每个地址值分别指向 `IMAGE_IMPORT_BY_NAME` 结构体。跟踪数组的第一个值 00023d12（RVA），进入该地址，可以看到导入的 API 函数的名称字符串。



**IMAGE_IMPORT_BY_NAME**

RVA:23d12 即为 RAW:21512。


![title](https://leanote.com/api/file/getImage?fileId=5f029010ab64411d3800108b)

根据 `IMAGE_IMPORT_BY_NAME` 结构体成员，文件偏移 21512 最初的 WORD 2个字节值（O35C）为Ordinal/Hint，是库中函数的固有编号。Ordinal的后面为函数名称字符串（BYTE 数组） `SelectObject`，字符串末尾以 Terminating NULL（00）结束。


INT 是 IMAGE_IMPORT_BY_NAME 结构体指针数组，数组的第1个元素指向函数的 Ordinal 值`02d0`。函数的名称为 `GetTextFaceW`。

注：在两个数组元素之间的00没什么含义。


**FirstThunk-IAT**

根据下面这个表：

|成员偏移|成员|RVA|RAW|
|:--:|:--:|:--:|:--:|
|20cb8|OriginalFirstThunk(INT)|00023834|21034|
|20cbc|TimeDateStamp|00000000|-|
|20cc0|ForwarderChain|00000000|-|
|20cc4|Name|00023e20|**21620**|
|20cc8|FirstThunk(IAT)|0002300c|2080c|


IAT 的 RVA:2300C 即为 RAW:2080c。

![title](https://leanote.com/api/file/getImage?fileId=5f0297ebab64411f37001187)

IAT 第一个元素值被硬编码为 00023d12，该值无实际意义，notepad.exe 文件加载到内存时，准确的地址值会取代该值。

01000000
0102300c


# 0x06：EAT：IMAGE_EXPORT_DIRECTORY 结构体


为 IMAGE_OPTIONAL_HEADER32.DataDirectory[0].VirtualAddress。


