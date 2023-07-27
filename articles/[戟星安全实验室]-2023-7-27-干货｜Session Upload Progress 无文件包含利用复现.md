#  干货｜Session Upload Progress 无文件包含利用复现

原创 Me5  [ 戟星安全实验室 ](javascript:void\(0\);)

**戟星安全实验室** ![]()

微信号 gh_0a430ebdb179

功能介绍 忆享科技旗下高端的网络安全攻防服务团队.安服内容包括渗透测试、代码审计、应急响应、漏洞研究、威胁情报、安全运维、攻防演练等

____

___发表于_

收录于合集

  

![]()

**戟星 安全实验室**

  

    忆享科技旗下高端的网络安全攻防服务团队.安服内容包括渗透测试、代码审计、应急响应、漏洞研究、威胁情报、安全运维、攻防演练等  
![]()  

本文约1260字，阅读约需3分钟。

![]()

  

![]()

前言

![]()

    事情起因是审计时挖到一个文件包含漏洞，正好没找到文件写入点。借此机会学习和复现一下PHP >= 5.4 利用“ **Session Upload Progress”** **** 无文件包含姿势。

  
  
![]()

内容可控

![]()  

首先我挖到的文件包含漏洞可以包含任意路径的文件，因为没有可以写入文件的点就需要考虑无文件包含的姿势，然后便是利用了” **Session Upload
Progress**
“进行无文件包含的，这是5.4新增的一个功能用于监测文件上传的进度——使用后PHP会把该文件的详细信息(如上传时间、上传进度等)存储在session当中。现在介绍一下相关的PHP配置信息：

  

  *   * 

    
    
    ”session.upload_progress.enabled = On“时为开启”上传进度监测功能“；session.upload_progress.cleanup= On“ 表示上传文件完毕后清除该SESSION文件；

session.upload_progress.name则表示当一个上传在处理中，同时POST一个与INI中设置的session.upload_progress.name同名变量时，上传进度可以在$_SESSION中获得,上传进度的
索引是 session.upload_progress.prefix 与
session.upload_progress.name拼接在一起的值，同时注意该值会保存在SESSION文件中（在我们浏览页面的时候登录一个账号就会创建一个会话，SESSION文件就是这个登陆后会话的文件形式），因此SESSION文件的内容有部分是我们可控的可以插入php代码，具体见下图：

  

1.这是 **session.upload_progress.prefix**

![]()

  

2.这是 **session.upload_progress.name**

![]()

  

两者拼接存入SESSION文件，经过php自己处理后的SESSION文件大概长这样，观察后半部分可以发现我们提交的恶意代码。这时候我们搞定了漏洞的三分之一:文件内容可控。

  

![]()

  

二.SESS文件名可控，SESS文件名在正常创建的时候可以发现不是规律的，我们文件包含漏洞没办法猜到该文件名。

![]()

  

但是在“session.use_strict_mode =
0”（默认为0）时是可控的，该配置项意味着用户可以控制自己的会话ID。提交参数”PHPSESSID=pikachu“，PHP程序会自动创建一个我们预期的SESSION文件，也就是”sess_pikachu

  

![]()

  

显然，文件包含的文件名也可以猜到了。那么我们可以直接“include('xxx/tmp/sess_pikachu')”了吗？很可惜的是，在文件上传完毕之后所有文件上传进度信息都会清除。

  

三.session.upload_progress.cleanup与条件竞争

  

”session.upload_progress.cleanup =
on“时在文件上传完毕之后所有文件上传进度信息都会清除，因此要用到条件竞争的方式在清除内容之前占用该文件，就像平时删除文件的时候系统提示文件占用一样。

  

补充一下：生成恶意SESS文件不是无条件的，只有服务器那边开了会话我们提交PHPSESSID才会创建该文件比如在

1.代码这里开启：

  

![]()

  

2.配置文件里开启

![]()

  

接下来漏洞利用具体操作就是，我们一边不停的生成我们的恶意SESS文件，一边不停的尝试触发文件包含漏洞。

  

![]()

  

成功包含文件

  

![]()

  

  

  

  

  
  
  
  
  
  
  
  
  
  
![]()

往期回顾

![]()  
[【干货分享】对RuoYi内存马简单二开](http://mp.weixin.qq.com/s?__biz=MzkzMDMwNzk2Ng==&mid=2247509582&idx=1&sn=8565539fe2486b677efe99c8aeaf21b4&chksm=c27eac5ff50925490246f8c2f7c597274545f1105cb620b4e0039afff23061fffdbbef8f795f&scene=21#wechat_redirect)  
[【工具分享】Burp插件-
AccessKey泄露正则匹配插件](http://mp.weixin.qq.com/s?__biz=MzkzMDMwNzk2Ng==&mid=2247509551&idx=1&sn=5530f8d5a5035a58b7e01d87357d31d6&chksm=c27eac3ef50925285c4c8eea67e4e1f09e6cbaff76bc067e4fed94587602542b32917f8770a3&scene=21#wechat_redirect)  
  
[干货分享|Nacos 身份认证绕过漏洞复现-
附Nuclei脚本](http://mp.weixin.qq.com/s?__biz=MzkzMDMwNzk2Ng==&mid=2247509525&idx=1&sn=e2273a874e7f7f7f1321e0bf8726cdc0&chksm=c27eac04f50925125cfe9c122de2a15ce8d04dc66285569113ddc360a55e41286b14539ee0f2&scene=21#wechat_redirect)  
  
[干货｜从无到有学习Golang编写poc&exp](http://mp.weixin.qq.com/s?__biz=MzkzMDMwNzk2Ng==&mid=2247509619&idx=1&sn=bab688d1bff32afe3abb3890319effa1&chksm=c27eac62f5092574cfb772ff738641d548256aabaf6097c9d80e20fc646336d3eaed460b9c33&scene=21#wechat_redirect)  
  
  
  
  

 ****

 **声明**

 **  
**

     由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，戟星安全实验室及文章作者不为此承担任何责任。

    戟星安全实验室拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经戟星安全实验室允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  
  

  

![]()![]()  

![]()

戟星安全实验室

# 长按二维码 || 点击下方名片 关注我们 #

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

