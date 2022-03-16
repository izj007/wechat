#  从外网 Weblogic 打进内网接管域控

[ 鸿鹄实验室 ](javascript:void\(0\);)

**鸿鹄实验室** ![]()

微信号 gh_a2210090ba3f

功能介绍 鸿鹄实验室，欢迎关注

____

__

收录于话题

以下文章来源于Tide安全团队 ，作者Komorebi

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM7f2r4cHKV0tKOK4v87qvAYZazjibgqC5HkYyaQeibGau4g/0)
**Tide安全团队** .

Tide安全团队以信安技术研究为目标，致力于分享高质量原创文章、开源安全工具、交流安全技术，研究方向覆盖网络攻防、Web安全、移动终端、安全开发、物联网/工控安全/AI安全等多个领域，对安全感兴趣的小伙伴可以关注我们。

![](https://gitee.com/fuli009/images/raw/master/public/20220316101228.png)  
逛公众号的时候看到了大佬提供的靶场，闲来无事正好拿来练练手学习一下，
感谢大佬的无私分享。感兴趣的同学可以去原文章中找一下环境地址，这里就不外放了，尊重作者版权。

## 目标设定

一台出网主机，三台域内机器，获取域控桌面下的flag即为完成任务  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101240.png)

## weblogic反序列化，powershell上线CS

  

  * 已知服务器地址为192.168.92.137，开放7001端口，扫描发现存在weblogic反序列化漏洞CVE_2020_2551  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101241.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20220316101242.png)  

  * tasklist无杀软，administrator权限  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101243.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20220316101244.png)

  

  * powershell上线CS  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101245.png)

## 内网信息收集

  * 内网信息收集，发现为双网卡机器  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101246.png)  

  * 搭frp代理进内网  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101247.png)  

  * fscan扫内网10段，可以看到有一台ms17010以及mssql弱口令sa&sa  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101248.png)  

  * 工具连接mssql，服务器低权限，不出网  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101250.png)  

![](https://gitee.com/fuli009/images/raw/master/public/20220316101251.png)  
![]()  
ipconfig /all可以看到是域内机器，ping该域名可获得域控IP地址 10.10.10.8  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101252.png)  

## 利用GPP获取域控administrator密码

### 什么是 GPP

  

GPP是指组策略首选项（Group Policy Preference），GPP通过操作组策略对象GPO（Group Policy
Object）对域中的资源进行管理。  
Freebuf的这篇文章http://www.freebuf.com/vuls/92016.html讲了GPP的应用场景和与之对应的安全问题。简单来说就是，出于想更新每台主机上本地账户密码的目的，利用GPP可以指定某个域账户为所有计算机的本地计算机管理账户。  
而这个账号信息存储在

  * 

    
    
    \[Domain Controller]\SYSVOL[Domain]\Policies中的某个Grouop.xml中

其中的cpassword为AES加密值。但在AD中的所有用户都可以读取Group.xml，对于AES的对称加密，在微软的MSDN上可以查到cpassword使用的固定秘钥  
  

### 利用过程

根据作者描述，靶场存在GPP漏洞，尝试利用一下  
使用  

    
    
    dir /s /a \\域控IP\SYSVOL\*.xml

  
构造如下语句

    
    
    dir /s /a \\redteam.red\SYSVOL\redteam.red\Groups.xml

![](https://gitee.com/fuli009/images/raw/master/public/20220316101254.png)  
获取到groups.xml文件，打开获取到cpassword字段

    
    
    type \\redteam.red\SYSVOL\redteam.red\Policies\{B6805F5A-614E-4D32-9C2B-7AC2B6798080}\Machine\Preferences\Groups\Groups.xml

  
![]()  
利用gpprefdecrypt.py获取到域控密码“cpassword”字段，得到

    
    
    administrator/Admin12345

![](https://gitee.com/fuli009/images/raw/master/public/20220316101255.png)

## IPC横向访问域控获取flag

  * IPC横向IPC是专用管道，可以实现对远程计算机的访问，需要使用目标系统用户的账号密码，使用139、445端口
  * 回到weblogic被控主机cs执行以下命令

    
    
    shell net use \\10.10.10.8\ipc$ "Admin12345" /user:redteam.red\administrator

  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101256.png)

  

  * 至此可以接管域控，以administrator权限执行命令。

  * 访问域控上的资源，获取 flag.txt  
![](https://gitee.com/fuli009/images/raw/master/public/20220316101257.png)

## 总结

域内提供了三台主机，还有一台ms17010的主机没有用上，当然目前的环境跟实战环境也是有一些出入，比如按个人习惯cs上线后肯定是先想办法看能不能开3389以及如何规避杀软，这里知道环境网络结构直接奔着目标去了，所以一切比较顺利。根据靶场描述，使用其他思路比如约束委派或者非约束委派都可接管域控,感兴趣的可以尝试下。另外再次感谢作者提供的靶场。

## 原文链接

从外网 Weblogic 打进内网，再到约束委派接管域控  
[https://mp.weixin.qq.com/s/dcYbIfLwN-
Aw0Z9XxQSGkQ](https://mp.weixin.qq.com/s?__biz=MzkxNDEwMDA4Mw==&mid=2247488950&idx=1&sn=48d93f1fac38eae99cc4e78474eb557c&scene=21#wechat_redirect)  
  

E

N

D

  

  

 **关**

 **于**

 **我**

 **们**

Tide安全团队正式成立于2019年1月，是新潮信息旗下以互联网攻防技术研究为目标的安全团队，团队致力于分享高质量原创文章、开源安全工具、交流安全技术，研究方向覆盖网络攻防、系统安全、Web安全、移动终端、安全开发、物联网/工控安全/AI安全等多个领域。

团队作为“省级等保关键技术实验室”先后与哈工大、齐鲁银行、聊城大学、交通学院等多个高校名企建立联合技术实验室。团队公众号自创建以来，共发布原创文章370余篇，自研平台达到26个，目有15个平台已开源。此外积极参加各类线上、线下CTF比赛并取得了优异的成绩。如有对安全行业感兴趣的小伙伴可以踊跃加入或关注我们。

![](https://gitee.com/fuli009/images/raw/master/public/20220316101259.png)

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

从外网 Weblogic 打进内网接管域控

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

