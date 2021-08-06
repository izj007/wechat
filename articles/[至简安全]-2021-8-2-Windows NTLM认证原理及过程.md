##  Windows NTLM认证原理及过程

Shincehor  [ 至简安全 ](javascript:void\(0\);)

**至简安全** ![]()

微信号 gh_5d5b4e19e73f

功能介绍
至简安全是由一群志同道合和的小伙伴组成的团队，专注于Web安全、内网渗透、红蓝对抗技术。成立的初心是为了共同提升安全技术，帮助学习者从简单的角度去学习网络安全。

____

__

收录于话题

Windows认证协议主要有以下两种：

  

1、NTLM认证，Windows早期密码管理方式。认证过程比较简单。  

  

2、Kerberos认证用于域环境中，他是一种基于票据(Ticket)的认证方式。  

  

本篇文章主要介绍NTLM认证  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210806182651.png)

  

一共是有两种认证场景：  

  * 交互式（用户登录到一台计算机）

  * 非交互式（用户访问服务器资源

    **  1、**（交互式登录到某客户机）用户使用： **域名、用户名、密码** ，登陆到某台客户端。然后客户端计算并存储用户密码的Hash值，然后将密码丢掉

  

![](https://gitee.com/fuli009/images/raw/master/public/20210806182652.png)

  

     **2、** 客户端将自己的用户名发送给服务器

  

![](https://gitee.com/fuli009/images/raw/master/public/20210806182653.png)

  

    3、服务器产生一个长度为16字节的随机数，这个被称作 **challenge** ，然后将其发给客户端  

      

![](https://gitee.com/fuli009/images/raw/master/public/20210806182654.png)

  

    4、客户端使用用户密码的散列值加密服务器发送过来的 Challenge，并将结果发回给服务器， 该步骤通常称为：应答 **Response**

 **  
**

![]()

  

  

     5、然后服务器会将以下三个东西发给域控制器

  * 用户名

  * 发送给客户端的 Challenge

  * 从客户端收到的 Response

  

![](https://gitee.com/fuli009/images/raw/master/public/20210806182655.png)

  

    6、域控制器会根据用户名从ntds.dit数据库中找到该用户对应的hash，然后使用这个hash去加密challenge

    7、域控制器比较自己第6步加密完成后的challenge和在第4步客户端计算出的response的值，如何两者一致，那么认证成功

  

  

参考文章：

  1. https://docs.microsoft.com/zh-cn/windows/win32/secauthn/microsoft-ntlm

  2. https://blog.csdn.net/include_heqile/article/details/103455391

  3. https://www.cnblogs.com/artech/archive/2011/01/25/ntlm.html  

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

Windows NTLM认证原理及过程

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

