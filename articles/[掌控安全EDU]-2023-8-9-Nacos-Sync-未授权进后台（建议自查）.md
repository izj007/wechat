#  Nacos-Sync-未授权进后台（建议自查）

原创 zkaq-杳若  [ 掌控安全EDU ](javascript:void\(0\);)

**掌控安全EDU** ![]()

微信号 ZKAQEDU

功能介绍 安全教程\高质量文章\面试经验分享，尽在#掌控安全EDU#

____

___发表于_

收录于合集

获网安教程

免费&进群

![]()  ![]()  

本文由掌控安全学院-杳若投稿

## 漏洞成因

没进行权限校验。

## 影响范围

Nacos-Sync 3.0

## 发现方式

### 一、fofa发现

    
    
    title="nacos" &amp;&amp; title=="Nacos-Sync"  
    

![]()

### 二、路径拼接

    
    
    /#/serviceSync  
    

## 利用方式

访问之后直接是进入后台的样子~

![]()

## 修复方式

对于鉴权类型的漏洞，主要的修复方式是全局增加鉴权

  

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

