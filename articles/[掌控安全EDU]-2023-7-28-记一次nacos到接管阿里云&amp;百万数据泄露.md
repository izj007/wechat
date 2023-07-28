#  记一次nacos到接管阿里云&百万数据泄露

原创 zkaq-山屿云  [ 掌控安全EDU ](javascript:void\(0\);)

**掌控安全EDU** ![]()

微信号 ZKAQEDU

功能介绍 安全教程\高质量文章\面试经验分享，尽在#掌控安全EDU#

____

___发表于_

收录于合集

获网安教程

免费&进群

![]()  ![]()  

本文由掌控安全学院-山屿云投稿

在某次攻防当中，通过打点发现了一台nacos，经过测试之后发现可以通过弱口令进入到后台，可以查看其中的配置信息

![]()

通过翻看配置文件，发现腾讯云的AK,SK泄露，以及数据库的账号密码。操作不就来了么，直接上云！

![]()

利用CF工具加上之前的AK，SK配置信息，创建腾讯云控制台账号密码，登录后直接上云！

![]()

翻看腾讯云的资源信息，发现5个存储桶，这一波直接加大分

![]()

还可以看一下账单信息，看看购买了什么资源

![]()

在私有网络当中，发现里面有两个子网，不过上不去也就没办法

![]()

因为客户开通了短信包，同时还可接管腾讯云短信，给自己发一条看看

![]()

接管后发送短信，可以篡改内容

![]()

同时泄露数据库账号密码，接管数据库，接管数十个数据库，泄露数据高达百万

![]()

![]()

数据库中泄露小程序APPID和相关的KEY，可接管小程序

#### 后续

后续没有深入，不让打了，同时这个资产是某X部的（懂得都懂），今天开打明天喝茶

对于云安全，其实相对于我们这种脚本小子来说，你会用cf工具，那么你就畅通无阻，但是你必须得找到AK/SK，那么AK/SK是什么玩意呢，它其实就是云服务给你的账号密码(相当于账号密码)不过不能直接登上去，需要用cf工具做做操作

关于cf工具，链接在这，大家自己下载即可  
https://github.com/teamssix/cf

非常滴好用!!

  

申明：本公众号所分享内容仅用于网络安全技术讨论，切勿用于违法途径，

所有渗透都需获取授权，违者后果自行承担，与本号及作者无关，请谨记守法.

![]()  

 **没看够~？欢迎关注！**

  

  

 **分享本文到朋友圈，可以凭截图找老师领取**

上千 **教程+工具+交流群+靶场账号** 哦



![]()

 ** ** **分享后扫码 加我！**

  

 **回顾往期内容**

[Xray挂机刷漏洞](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247504665&idx=1&sn=eb88ca9711e95ee8851eb47959ff8a61&chksm=fa6baa68cd1c237e755037f35c6f74b3c09c92fd2373d9c07f98697ea723797b73009e872014&scene=21#wechat_redirect)  

[零基础学黑客，该怎么学？](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487576&idx=1&sn=3852f2221f6d1a492b94939f5f398034&chksm=fa686929cd1fe03fcb6d14a5a9d86c2ed750b3617bd55ad73134bd6d1397cc3ccf4a1b822bd4&scene=21#wechat_redirect)

[网络安全人员必考的几本证书！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247520349&idx=1&sn=41b1bcd357e4178ba478e164ae531626&chksm=fa6be92ccd1c603af2d9100348600db5ed5a2284e82fd2b370e00b1138731b3cac5f83a3a542&scene=21#wechat_redirect)  

[文库｜内网神器cs4.0使用说明书](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247519540&idx=1&sn=e8246a12895a32b4fc2909a0874faac2&chksm=fa6bf445cd1c7d53a207200289fe15a8518cd1eb0cc18535222ea01ac51c3e22706f63f20251&scene=21#wechat_redirect)  

[代码审计 |
这个CNVD证书拿的有点轻松](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503150&idx=1&sn=189d061e1f7c14812e491b6b7c49b202&chksm=fa6bb45fcd1c3d490cdfa59326801ecb383b1bf9586f51305ad5add9dec163e78af58a9874d2&scene=21#wechat_redirect)

[【精选】SRC快速入门+上分小秘籍+实战指南](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247512593&idx=1&sn=24c8e51745added4f81aa1e337fc8a1a&chksm=fa6bcb60cd1c4276d9d21ebaa7cb4c0c8c562e54fe8742c87e62343c00a1283c9eb3ea1c67dc&scene=21#wechat_redirect)

## [    代理池工具撰写 |
只有无尽的跳转，没有封禁的IP！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503462&idx=1&sn=0b696f0cabab0a046385599a1683dfb2&chksm=fa6bb717cd1c3e01afc0d6126ea141bb9a39bf3b4123462528d37fb00f74ea525b83e948bc80&scene=21#wechat_redirect)

![]()

点赞+在看支持一下吧~感谢看官老爷~

你的点赞是我更新的动力

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

