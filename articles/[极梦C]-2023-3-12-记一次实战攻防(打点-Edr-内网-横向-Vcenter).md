#  记一次实战攻防(打点-Edr-内网-横向-Vcenter)

[ 极梦C ](javascript:void\(0\);)

**极梦C** ![]()

微信号 gh_2353880ae4d9

功能介绍 只专注于实战的实战派。

____

___发表于_

收录于合集

以下文章来源于Tide安全团队 ，作者tale

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM7f2r4cHKV0tKOK4v87qvAYZazjibgqC5HkYyaQeibGau4g/0)
**Tide安全团队** .

Tide安全团队以信安技术研究为目标，致力于分享高质量原创文章、开源安全工具、交流安全技术，研究方向覆盖网络攻防、Web安全、移动终端、安全开发、物联网/工控安全/AI安全等多个领域，对安全感兴趣的小伙伴可以关注我们。

  

## 前言

前不久参加了一场攻防演练，过程既简单也曲折，最后通过横向渗透获取到了vcenter管理控制台权限，成功拿下本次演练目标。

## 寻找目标

目标分配后，面对大范围的目标，首先要做的就是寻找一些容易获取权限的站点，比如shiro、weblogic以及各类反序列化漏洞。之后再将目标锁定到管理后台，使用爆破等手段寻找一些能进后台的账号密码，然后再去找上传点拿shell。  
本次渗透在某站注册页面发现了上传身份证的地方，但是上传后发现无返回路径。  
![](https://gitee.com/fuli009/images/raw/master/public/20230312224825.png)后通过目录扫描，发现了存在目录遍历漏洞，可以看到不同日期上传的文件

    
    
    http://IP/upload/Attachment/

![](https://gitee.com/fuli009/images/raw/master/public/20230312224847.png)然后在对上传点测试时发现存在waf，拦截了非法后缀名、文件内容等。对此进行一些常规的修改和测试例如修改Content-
Type、修改boundary前加减空格、多个Content-Disposition字段、分块传输、脏字符最终上传成功。

## 获取webshell

找到上传后的木马获取webshell![](https://gitee.com/fuli009/images/raw/master/public/20230312224849.png)拿到webshell发现是IIS权限，通过查看进程发现装有杀软和edr![](https://gitee.com/fuli009/images/raw/master/public/20230312224850.png)对木马进行免杀上线CS，然后利用CS插件进行提权时发现失败。  

![](https://gitee.com/fuli009/images/raw/master/public/20230312224852.png)

然后通过搜集本机的服务发现开放了1433端口，通过查找配置文件发现了数据库连接地址  
![](https://gitee.com/fuli009/images/raw/master/public/20230312224853.png)

## 提权system

利用CS开启代理后，登录连接MSSQL数据库，执行命令为system权限![](https://gitee.com/fuli009/images/raw/master/public/20230312224854.png)最终利用数据库权限执行CS木马上线获取system权限。  

![](https://gitee.com/fuli009/images/raw/master/public/20230312224855.png)

## 绕过edr

通过query
user查看管理员用户不在线，利用mimikatz获取管理员密码和hash，搭建socks隧道进入目标机远程桌面，发现存在某edr二次验证![](https://gitee.com/fuli009/images/raw/master/public/20230312224856.png)查询网上资料后发现将该文件删除或重命名即可绕过，成功登录服务器  

    
    
    C:Program Files/Sangfor/EDR/agent/bin/sfrdpverify.exe

![](https://gitee.com/fuli009/images/raw/master/public/20230312224857.png)登录该管理员主机后，发现阿里云盘，通过搜索阿里云盘文件发现大部分为备份文件，在某一最近更新的项目中发现服务器密码本txt，其中记录的账号密码一台服务器为本机服务器，一台为其它服务器![](https://gitee.com/fuli009/images/raw/master/public/20230312224859.png)连接密码本中记录的第二台服务器，同时上线CS，有了第二台跳板机后，上传内网扫描工具使用第一台跳板机进行扫描（主要是怕在只有一台跳板机情况下直接扫描容易触发设备告警，造成权限丢失）。同时继续搜集信息，扩大攻击面。  
![](https://gitee.com/fuli009/images/raw/master/public/20230312224900.png)

## 突破隔离内网横向

最后发现查看本机记录的mstsc发现可连接新网段机器，同时测试发现只有该服务器可访问![](https://gitee.com/fuli009/images/raw/master/public/20230312224901.png)![](https://gitee.com/fuli009/images/raw/master/public/20230312224902.png)连接该服务器后，继续发现存在套娃服务器xx.xx.xx.7![](https://gitee.com/fuli009/images/raw/master/public/20230312224904.png)测试发现均不出网，使用CS的中转功能进行中转跳板机上线![](https://gitee.com/fuli009/images/raw/master/public/20230312224905.png)并上传扫描工具对该网段进行扫描，扫描发现该网段存在vcenter管理控制台![](https://gitee.com/fuli009/images/raw/master/public/20230312224906.png)

## 获取vcenter管理控制权

使用常用的漏洞如CVE-2021-21972、CVE-2021-21985等漏洞进行测试无果后，将目标锁定在该服务器其它端口上，通过全端口扫描发现该服务器还开放41433端口存在MSSQL服务。使用top100弱口令爆破失败后，打算通过搜集拿到的所有服务器、数据库、浏览器等密码进行碰撞。最终运气好碰撞成功，拿到该vcenter服务器权限。![](https://gitee.com/fuli009/images/raw/master/public/20230312224907.png)![](https://gitee.com/fuli009/images/raw/master/public/20230312224908.png)成功登录该vcenter服务器![](https://gitee.com/fuli009/images/raw/master/public/20230312224909.png)最终拿下该目标单位约70余台服务器，20台数据存储等。至此，目标单位出局。  
![](https://gitee.com/fuli009/images/raw/master/public/20230312224910.png)

## 总结

此次渗透表面看起来丝滑顺畅，实际当中走了不少弯路，尤其是安全防护、安全设备以及蓝队防守人员存在，让打点变得更为困难。同时大量的时间花费在了内网信息搜集上，包括在内网中扫描出不少资产和漏洞，越做越大搜集的信息也越来越多，很容易迷失方向从而会使人变得烦躁，所以在攻防演练中的有限时间内需要根据具体的评分规则进行目的性渗透(刷分)。  
  
  
  
  
  
  
 **星球内容**

正式运营星球:

1.src真实漏洞挖掘案例分享(永久不定时更新),过程详细一看就会哦。  
2.自研/二开等工具/平台的分享。  
3.漏洞分析/资料共享等。  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230312224911.png)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230312224912.png)

  

  
  

![](https://gitee.com/fuli009/images/raw/master/public/20230312224913.png)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230312224914.png)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230312224915.png)

  
  
  
  
  
  

 **ps:漏洞都是加班加点挖到的,最近白嫖的人日益增多越来越多。  
白嫖党过多,增加了新人阅读权限,3天就解除。**

  
  
  
  

  

  

  

  

  

  
  
  
  
  

  

  

关于文章/Tools获取方式:请关注交流群或者知识星球。

  

  

  

  

  
  
  
  
  
  

关于交流群：因为某些原因，更改一下交流群的获取方式:

1.请点击联系我们->联系官方->客服小助手添加二维码拉群 。  

![]()

  

  

  

  

  
  

关于知识星球的获取方式:

1.后台回复发送 "知识星球"，即可获取知识星球二维码。

2如若上述方式不行，请点击联系我们->联系官方->客服小助手添加二维码进入星球 。  

![]()

  

  

  

  

  

  

  

免责声明

  

          

  

本公众号文章以技术分享学习为目的。

由于传播、利用本公众号发布文章而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号及作者不为此承担任何责任。

一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！

  

  

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230312224917.png)

  

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

