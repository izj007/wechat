# 0x01 Upack 壳介绍

Upack 壳是一款运行时压缩壳，以对 PE 头的独特变形技法而闻名。基本上难以用于免杀，免杀的话一般可以用加密壳。

>注：UPack 压缩器本身不是恶意程序，但是许多恶意代码制作者用 UPack 来压缩自己的恶意代码，使文件变得畸形，是许多杀毒软件将 UPack 压缩的文件全部识别为病毒文件并删除。达到了一种白文件加了 UPack 壳也被识别为病毒文件的效果 :（

在 Windows 10 上，Upack 程序本身无法打开，其加壳后的软件也无法打开。所以本文的实验环境都是在 Windows XP professional SP3 上。


# 0x02 使用 UPack 压缩 notepad.exe

![title](https://leanote.com/api/file/getImage?fileId=5f06c5b3ab644124cb000720)


对 C:\Windows\system32\notepad.exe 压缩：

![title](https://leanote.com/api/file/getImage?fileId=5f06c676ab644124cb000727)



Upack 会直接压缩源文件本身。压缩完还是可以正常打开程序。


对压缩后的文件使用 CFF Explorer 查看，PE 文件头区域比较完整，但是 NT 头 - 可选头 - Data Directories[10] 数组显示有很多无效的 section。


![title](https://leanote.com/api/file/getImage?fileId=5f06c7e3ab644124cb000738)



# 0x03 比较 PE 文件头

先使用 Hex Editor 打开两个文件，比较其 PE 头部分。

**原 notepad.exe 的 PE 文件头：**

![title](https://leanote.com/api/file/getImage?fileId=5f06cb40ab644124cb000756)

可以看出是非常标准的 PE 文件头，其中数据按照 `IMAGE_DOS_HEADER`、`DOS Stub`、`IMAGE_NT_HEADERS`、`IMAGE_SECTION_HEADER` 顺序排列。



**notepad.exe Upack 压缩文件的 PE 文件头**


![title](https://leanote.com/api/file/getImage?fileId=5f06cc22ab644124cb00075f)

PE 头看起来比较奇怪。因为：

- `MZ` 与 `PE` 贴得太近了，中间还夹着一个库名；
- 没有 DOS stub；
- 没有找到明确的节区头......

下面就来分析 Upack 中使用的这种独特的 PE 文件头结构。


----------------------------

# 0x04 分析 UPack 的 PE 文件头

## 一、重叠文件头


`重叠文件头`是一种压缩器经常使用的技法，可以把 MZ 文件头（`IMAGE_DOS_HEADER`）与 PE 文件头（`IMAGE_NT_HEADERS`） 巧妙重叠在一起，可以有效节约文件头空间，增加文件头的复杂性，给分析带来很大困难（一些 PE 相关工具难以分析）。

使用 Stud_PE 查看一下 IMAGE_DOS_HEADER 部分：

>注：Stud_PE 2.4.0.1 版本及之后已加入对 UPack 壳的支持修复。

![title](https://leanote.com/api/file/getImage?fileId=5f06cef0ab644126bd0007a3)


关注 MZ 文件头（`IMAGE_DOS_HEADER`）中的以下2个重要成员：

```
(offset   0)  e_magic：Magic number = AD5A('MZ')
(offset 03C) e_lfanew：指示 NT 头的偏移
```


根据 PE 文件格式规范，`IMAGE_NT_HEADERS` 的起始位置是「可变的」。也就是说：`IMAGE_NT_HEADERS` 的起始位置由 `e_lfanew` 的值决定。

而一般在一个正常程序中，`e_lfanew` 的值如下：

```
e_lfanew = IMAGE_DOS_HEADER 文件头大小(40) + DOS stub 大小（可变，在这里为 A0）= E0
```


应该为 E0，实际上 UPack 中 `e_lfanew` 的值为10，这并不违反 PE 规范，只是钻了规范本身的空子罢了。通过把 `e_lfanew` 设为一个小于`IMAGE_DOS_HEADER 文件头大小(40) + DOS stub 大小`的值，就可以把 MZ 文件头（`IAMGE_DOS_HEADER`）和 PE 文件头（`IMAGE_NT_HEADERS`）重叠在一起。


## 二、IMAGE_FILE_HEADER.SizeOfOptionalHeader


NT 头 - 文件头的 `IMAGE_FILE_HEADER` 结构体：

```
typedef struct _IMAGE_FILE_HEADER {
    WORD   Machine;
    WORD   NumberOfSections;
    DWORD  TimeDateStamp;
    DWORD  PointerToSymbolTable;
    DWORD  NumberOfSymbols;
    WORD   SizeOfOptionalHeader;
    WORD   Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

这中间的成员 `SizeOfOptionHeader` 表示 PE 文件头中紧接在 IMAGE_FILE_HEADER 下的 IMAGE_OPTIONAL_HEADER 结构体的长度（E0）。

![title](https://leanote.com/api/file/getImage?fileId=5f06d921ab644126bd0007f7)


UPack 将该值更改为 148。


在这里有一个知识： IMAGE_OPTIONAL_HEADER 是结构体，PE32 文件格式中其大小已经被确定为 E0。那么既然如此，为什么还需要在 `IMAGE_FILE_HEADER` 结构体中输入 `SizeOfOptionalHeader` 成员呢？其实这里的设计意图是：由于 IMAGE_OPTIONAL_HEADER 的种类很多，所以需要另外输入结构体的大小（比如，64位 PE32+ 的 IMAGE_OPTIONAL_HEADER 结构体的大小为 F0）。

`SizeOfOptionalHeader` 的另一层含义是确定节区头（`IMAGE_SECTION_HEADER`）的起始偏移。因为可选头后面就是节区头。

仅从 PE 文件头来看，紧接着 IMAGE_OPTIONAL_HEADER 的好像是 IMAGE_SECTION_HEADER。但更准确的说： `IMAGE_OPTIONAL_HEADER` 的起始偏移 + `SizeOfOptionalHeader` 值后的位置开始才是 `IMAGE_SECTION_HEADER`。


UPack 把 `SizeOfOptionalHeader` 的值从 E0 改为 148，这样会导致 IMAGE_SECTION_HEADER 的实际偏移是从偏移 170 开始的。

```
10h(`e_lfanew`)+4h(Signature)+14h(IMAGE_FILE_HEADER 2+2+4+4+4+2+2=14h)=28h
```


![title](https://leanote.com/api/file/getImage?fileId=5f06e219ab644126bd000861)



Upack 的意图是什么？为什么要把 `SizeOfOptionalHeader` 这个值改大呢？

UPack 的基本特征就是把 PE 文件头变形，像扭曲的麻花一样，像文件头适当插入解码需要的代码。增大 `SizeOfOptionalHeader` 的值后，就在 `IMAGE_OPTIONAL_HEADER` 和 `IMAGE_SECTION_HEADER` 之间增加了额外空间。UPack 就向这个区域添加解码代码，这是一种超越 PE 文件头常规理解的巧妙方法。



这里有一个坑点：

STud_PE 显示 `IMAGE_OPTIONAL_HEADER` 结束的位置为 107；

![title](https://leanote.com/api/file/getImage?fileId=5f06e88eab644124cb0008bf)


而 010 Editor 显示 `IMAGE_OPTIONAL_HEADER` 结束的位置为 D7：


![title](https://leanote.com/api/file/getImage?fileId=5f06e9a0ab644126bd0008ab)


这个差别主要是因为 Upack 把 `NumberOfRvaAndSize` 的值由 10 改为了 A 个。这样就导致 DATA_DIRECTORY[] 数组少占用了 6*8(3*16)个字节的空间。


而 IMAGE_SECTION_HEADER 的起始位置为170。
所以综上总 D7 到 170 之间的中间区域都可以被 UPack 添加解压代码。

使用 010 Editor 查看中间的区域：


![title](https://leanote.com/api/file/getImage?fileId=5f07ac1fab644124cb000f56)

使用 x32dbg 查看反汇编代码，可以看到从 010010D8 到 010016F 之间并不是 PE 文件头中的信息，而是 UPack 中使用的代码。



![title](https://leanote.com/api/file/getImage?fileId=5f07ad9bab644124cb000f62)


若 PE 相关实用工具将其识别为 PE 文件头信息，就会引发错误，导致程序无法正常运行。


## 三、IMAGE_OPTIONAL_HEADER.NumberOfRvaAndSizes


NT 可选头的结构体 `IMAGE_OPTIONAL_HEADER32`：

```
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD VirtualAddress;
    DWORD Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;

#define IMAGE_NUMBEROF_DIRECTORY_ENTRIRD 16

typedef struct _IMAGE_OPTIONAL_HEADER {
    WORD Magic;
    BYTE MajorLinkerVersion;
    BYTE MinorLinkerVersion;
    DWORD SizeOfCode;
    DWORD SizeOfInitializedData;
    DWORD SizeOfUninitializedData;
    DWORD AddressOfEntryPoint;
    DWORD BaseOfCode;
    DWORD BaseOfData;
    DWORD ImageBase;
    DWORD SectionAlignment;
    DWORD FileAlignment;
    DWORD MajorOperatingSystemVersion;
    DWORD MinorOperatingSystemVersion;
    DWORD MajorImageVersion;
    DWORD MinorImageVersion;
    DWORD MajorSubsystemVersion;
    DWORD MinorSubsystemVersion;
    DWORD Win32VersionValue;
    DWORD SizeOfImage;
    DWORD SizeOfHeaders;
    DWORD CheckSum;
    WORD Subsystem;
    WORD DllCharacteristics;
    DWORD SizeOfStackReserve;
    DWORD SizeOfStackCommit;
    DWORD SizeOfHeapReserve;
    DWORD SizeOfHeapCommit;
    DWORD LoaderFlags;
    DWORD NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```


结构体中的 `NumberOfRvaAndSizes` 成员用来指定 DataDirectory 数组的个数。**虽然结构体定义中明确的指出了数组个数为 IMAGE_NUMBBEROF_DIRECTORY_ENTRIES(16)。但是 PE 装载器通过查看 NumberOfRvaAndSizes 值来识别数组大小。换言之，数组大小也可能不是16。**


UPack 正是利用了这一点。UPack中将 `NumberOfRvaAndSizes` 的值改为了 A 个，而正常文件中值为 10h。


![title](https://leanote.com/api/file/getImage?fileId=5f07bb18ab644124cb000fd3)


IMAGE_DATA_DIRECTORY 结构体数组元素的个数已经被确定为 10，但 PE 规范将 NumberOfRvaAndSizes 值作为数组元素的个数。所以 UPack 中 IMAGE_DATA_DIRECTORY 结构体数组的后 6 个元素被忽略。


IMAGE_DATA_DIRECTORY 结构体数组的各项如下：



|索引|内容|
|:--|:--|
|0|EXPORT Directory|
|1|IMPORT Directory|
|2|RESOURCE Directory|
|3|EXCEPTION Directory|
|4|SECURITY Directory|
|5|BASEERELOC Directory|
|6|DEBUG Directory|
|7|COPYRIGHT Directory|
|8|GLOBALPTR Directory|
|9|TLS Directory|
|A|LOAD_CONFIG Directory|
|B|BOUND_IMPORT Directory|
|C|IAT Directory|
|D|DELAY_IMPORT Directory|
|E|COM_DESCRIPTOR Directory|
|F|Reserved Directory|


UPack 将 IMAGE_OPTIONAL_HEADER.NumberOfRvaAndSizes 的值更改为 A，从 LOAD_CONDIG Directory 项（文件偏移 D8 以后）开始不再使用。


![title](https://leanote.com/api/file/getImage?fileId=5f07bea5ab644126bd000fc0)

如图的蓝色高亮区域就是 UPack 忽视的部分（D8~107区域=LOAD_CONFIG Directory之后）。

使用调试器查看被忽视的这部分区域：

![title](https://leanote.com/api/file/getImage?fileId=5f07bff9ab644124cb000ff9)

这部分是 UPack 自身的解码代码。

>注：一些调试器（比如 OllyDbg）检查 PE 文件时会检查 NumberOfRvaAndSizes 的值是否为 10h，也会弹出错误消息框。但是这个错误信息并不重要，可以忽略。使用其他插件也可以完全删除，仅供参考。





## 四、IMAGE_SECTION_HEADER


IMAGE_SECTION_HEADER 结构体中，UPack 会把自身数据记录到程序运行不需要的项目中。这与 UPack 向 PE 文件头中不使用的区域覆写自身代码与数据的方式是一样的（PE 文件头中未使用的区域比想象的要多）。

可以看出节区数为3个，`IMAGE_SECTION_HEADER` 结构体数组的起始偏移为 170。截止到 000001E7。即为偏移 170 ~ 1E7 的区域。


![title](https://leanote.com/api/file/getImage?fileId=5f07c430ab644124cb00106f)


下图中框选的结构体成员对程序运行没有任何意义。

![title](https://leanote.com/api/file/getImage?fileId=5f07c6bfab644124cb00108e)



## 五、重叠节区


UPack 的主要特征之一就是可以随意重叠 PE 节区（注：不是节区头，就是节区）和文件头。

![title](https://leanote.com/api/file/getImage?fileId=5f07da75ab644126bd001152)

通过 Stud_PE 查看 UPack 的 IMAGE_SECTION_HEADER。可以看到：

- 第1个节区和第3个节区的起始偏移（RawOffset）值都为 10。偏移10原本属于PE头区域，但是 UPack 中这个位置起已经是节区部分了。
- 第1个节区与第3个节区在文件中的大小（RawSize）是完全一样的，都是 IF0。
- 但是第1个节区和第3个节区的 VirtualOffset 和 VirtualSize 是不相等的。


PE 规范并未明确指出这样做是不行的。综上，UPack 会对 PE 文件头、第1个节区和第3个街区进行重叠。但是当映射到内存时，又会映射到三个不同的内存位置（文件头、第一个节区、第三个节区）。也就是说，用相同的文件映像可以分别创建出处于不同位置的、大小不同的内存映像。

文件的头（第一/第三个节区）区域的大小为200（**本身是 01F，但是 UPack 的 FileAlignment 为200，对齐200就是200**），其实这是非常小的。相反，第二个节区尺寸非常大（AE28），占据了文件的大部分区域，原文件（notepad.exe）即压缩于此。



另外一个需要注意的部分是内存中的第一个节区区域，它的内存尺寸为 13000，与原文件（压缩前的 notepad.exe）的 SizeOfImage 具有相同的值。这说明：**压缩在第二个街区中的文件映像会被原样解压缩到第一个节区（notepad的内存映像）。**


![title](https://leanote.com/api/file/getImage?fileId=5f07df89ab644124cb0011c7)

另外，原 notepad.exe 拥有3个节区（`.text`、`.data`、`.rsrc`），它们被解压到一个节区。


总结一下：压缩的 notepad.exe 在内存的第二个节区，解压缩的同时被记录到第一个节区。重要的是，notepad.exe（原文件）的内存映像会被整体解压，所以程序能够正常运行（地址变得准确而一致）。


## 六、RVA to RAW


```
RAW = RVA - VirtualAddress + PointerToRawData
```

计算一下 EP 的文件偏移量（RAW）。UPack 的 EP 是 RVA 00001018。

![title](https://leanote.com/api/file/getImage?fileId=5f07facbab644126bd00129f)

RVA 1018 位于第一个 section：

![title](https://leanote.com/api/file/getImage?fileId=5f07fb43ab644126bd0012a4)

将其代入公式换算如下：


RAW = 1018 - 1000(`VirtualOffset`) + 10(`RawOffset`) = 28


但是使用 010 Editor 查看 RAW 28：

![title](https://leanote.com/api/file/getImage?fileId=5f07fc3dab644124cb0012e4)


发现 RAW 28 不是代码区域，而是（ordinal:010B）`LoadLibraryA` 字符串区域。所以很显然 EP 的 RAW 算的不对，原因就在于第一个节区的 `PointerToRawData` 值 10。


一般而言，**指向节区开始的文件偏移的** `PointerToRawData` 值应该是 FileAlignment 的整数倍。UPack 的 FileAlignment 为 200：


![title](https://leanote.com/api/file/getImage?fileId=5f07fd98ab644126bd0012b9)


所以 PointerToRawData 的值应该为 200 的整数倍。PE 装载器发现第一个节区的 PointerToRawData(10) 不是 FileAlignment(200) 的整数倍时，它会强制将其识别为整数倍。在这里会被识别为0（因为如果识别为 200，那么 EP 将不会被包括在内）。这使得 UPack 文件能够正常运行，但是许多 PE 相关实用程序都会发生错误。


所以最终 RVA → RAW 变换如下：
```
RAW=1018-1000+0=18
```


使用调试器查看相应区域的代码：


![title](https://leanote.com/api/file/getImage?fileId=5f07fefeab644126bd0012ca)


的确就是 EP。


## 七、导入表（IMAGE_IMPORT_DESCRIPTOR array）


导入表是一个数组，导入多少个库就有多少个 `IMAGE_IMPORT_DESCRIPTOR` 结构体。

UPack 的导入表结构组织相当独特。

![title](https://leanote.com/api/file/getImage?fileId=5f0800adab644124cb001312)


首先从 Directory Table 中获取 IDT（`IMAGE_IMPORT_DESCRIPTOR` 结构体数组）的位置。

如图：

- RVA of Import Table 为 000261EE
- size of Import Table 为 00000014

计算 Import Table 的 RAW。首先要确定该 RVA(261EE) 属于哪个节区：


![title](https://leanote.com/api/file/getImage?fileId=5f08019aab644124cb00131c)



如图在内存中属于第三个节区。


```
RAW=261EE(RVA)-26000(VirtualOffset)+0(RawOffset)=1EE
```
>注：第三节区的 RawOffset 本身为 10，为了跟 FileAlignment(200) 对齐，被强制变换为0。


使用 010 Editor 查看文件偏移 1EE 中的数据：


![title](https://leanote.com/api/file/getImage?fileId=5f0804a6ab644124cb001337)

从 1EE 开始的 14H 字节，此部分就是使用 UPack 节区隐藏玄机的地方。


IMAGE_IMPORT_DESCRIPTOR 结构体的定义为：


```
typedef struct_IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD Characteristics;
        DWORD OriginalFirstThunk;  //INT
    };
    DWORD TimeDateStamp;
    DWORD ForwarderChain;
    DWORD Name;
    DWORD FirstThunk;    //IAT
}IMAGE_IMPORT_DESCRIPTOR;
```


根据 PE 规范，导入表是由一系列 `IMAGE_IMPORT_DESCRIPTOR` 结构体组成的数组，最后以一个内容为 NULL 的结构体结束。


下图中的蓝色高亮区域就是 `IMAGE_IMPORT_DESCRIPTOR` 结构体数组（导入表）。

![title](https://leanote.com/api/file/getImage?fileId=5f0804a6ab644124cb001337)

偏移 1EE~201 为第一个结构体，其后既不是第二个结构体，也不是表示导入表结束的 NULL 结构体。

乍一看这种做法分明是违反 PE 规范的。但是值得注意的一个点是：

![title](https://leanote.com/api/file/getImage?fileId=5f0806e5ab644124cb001362)



第三个节区文件偏移为 00（强制对齐）~200（1F0+10）。内存映像区域为 26000（对齐的）~27000。


那么偏移200以下的区域，就不会映射到第三个节区的内存了。


![title](https://leanote.com/api/file/getImage?fileId=5f080ab9ab644126bd00135c)


第三个节区加载到内存时，文件偏移 0~1FF 的区域映射到内存的 26000~261FF 区域，而第三个节区剩余的内存区域 26200~27000 为了对齐 SectionAlignment 全部填充为 NULL。

使用调试器查看：

![title](https://leanote.com/api/file/getImage?fileId=5f080c7fab644124cb0013a1)


可以看到只映射到 010261FF，从 01026200 开始全部填充为 NULL 值。


这刚好符合了 PE 规范的导入表条件，01026202 地址以后出现 NULL 结构体。而这正是 UPack 使用节区的玄机。从文件看导入表好像是损坏了，但是在内存中却是正确的。在内存中 `IMAGE_IMPORT_DESCRIPTOR` 结构体后紧跟 NULL 结构体。

大部分 PE 实用程序从文件中读导入表时都会被「偏移 1EE~201 为第一个结构体，其后既不是第二个结构体，也不是表示导入表结束的 NULL 结构体」这一玄机所迷惑，查找错误的地址，继而引起内存引用错误，导致程序非正常终止。


## 八、导入地址表


下面通过分析 IAT 查看 UPack 都输入了那些 DLL 中的哪些 API。

结合以下两部分，可以得到：


① 蓝色高亮区域就是 `IMAGE_IMPORT_DESCRIPTOR` 结构体数组（导入表）。

![title](https://leanote.com/api/file/getImage?fileId=5f0804a6ab644124cb001337)



② IMAGE_IMPORT_DESCRIPTOR 结构体的定义为：


```
typedef struct_IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD Characteristics;
        DWORD OriginalFirstThunk;  //INT
    };
    DWORD TimeDateStamp;
    DWORD ForwarderChain;
    DWORD Name;
    DWORD FirstThunk;    //IAT
}IMAGE_IMPORT_DESCRIPTOR;
```



|偏移|成员|RVA|
|:--|:--|:--|
|1EE|OriginalFirstThunk(INT)|0|
|1FA|Name|2|
|1FE|FirstThunk(IAT)|11E8|


首先 Name 的 RVA 值为2，它属于 PE 头区域，因为第一个节区是从 RVA 1000 开始的：

![title](https://leanote.com/api/file/getImage?fileId=5f0811e7ab644124cb0013f6)


Header 区域中 RVA 和 RAW 值是一样的（头一般很小），故使用 010 Editor 查看文件中的偏移（RAW）为2的区域：


![title](https://leanote.com/api/file/getImage?fileId=5f0814b8ab644124cb001414)

可以看到字符串 `KERNEL32.ALL`，此位置原本是 DOS 头 `IMAGE_DOS_HEADER` 的区域，属于不使用的区域（IMAGE_DOS_HEADER 中只有 `e_magic` 和 `e_lfanew` 比较重要），UPack 将 ImportDLL 名称写入该处。

>注：
IMAGE_IMPORT_DESCRIPTOR 结构体中：
>
- Name 成员如果是 ASCII 码就以1个`00`结束，Unicode 编码就以2个`00`结束。
- OriginalFirstThunk(INT) 成员以 `00000000`（4个00）结束。
- FirstThunk 成员也以 `00000000`（4个00）结束。
- PE 程序可能用 IAT 指示 API 名称字符串，也可能用 INT 指示 API 名称字符串。二者里面其中一个有 API 名称字符串即可。




得到 DLL 名称后，再看一下从中导入了哪些 API 函数。一般而言，跟踪 OriginalFirstThunk(INT) 能够发现 API 名称字符串。但是像 UPack 这样 OriginalFirstThunk(INT) 为0时，跟踪 FirstThunk(IAT) 似乎是更好的选择（只要 INT、IAT 其中一个有 API 名称字符串即可）。


因为 IAT 的值为 11E8，属于第1个节区：

```
RAW = 11E8-1000+0=1E8
```

>注：第一节区的 RawOFFSet 值本来是10，为了跟 FileAlignment 对齐被强制转换为 0。




在 010 Editor 中查看 1E8 的文件偏移：

![title](https://leanote.com/api/file/getImage?fileId=5f081ea2ab644126bd001447)


蓝色高亮的部分就是 IAT 域，同时也作为 INT 来使用。也就是说，该处是 Name Pointer（RVA）数组，其结束是 NULL。此外还可以看到导入了2个 API，分别为 RVA 28 和 RVA BE。

由于这两个 RVA 都属于 header 区域，所以 RVA 与 RAW 值是一样的。

如图可以看到导入的 `LoadLibraryA` 函数，其中 RVA 位置上存在着此导入函数的 [ordinal+名称字符串]，其中 ordinal 为 `0B01`，是库中函数的固有编号。ordinal 的后面为函数名称字符串 `LoadLibraryA`。

![title](https://leanote.com/api/file/getImage?fileId=5f081fbaab644124cb00149b)


同理，另一个导入的 API 为在 RAW BE 处：

![title](https://leanote.com/api/file/getImage?fileId=5f08210eab644126bd001471)


- ordinal 为 `0000`；
- 函数名称字符串为 `GetProcAddress`；
- IAT 以 `00` 结束。


所以导入的2个 API 函数分别为 `LoadLibraryA` 与 `GetProcAddress`，它们在形成原文件的 IAT 时非常方便，所以普通压缩器也常常导入使用。


