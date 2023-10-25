#  win xp,2003环境上线cobaltstrike

原创 泼猴  [ 谢公子学安全 ](javascript:void\(0\);)

**谢公子学安全** ![]()

微信号 xie_sec

功能介绍 分享网络安全攻防对抗知识，作者拥有多年一线实战攻防经验，擅长内网渗透、域渗透等。《域渗透攻防指南》📚作者，国内知名攻防对抗专家。

____

___发表于_

收录于合集

#红蓝对抗 40 个

#攻防演练 14 个

  
  
  
  
  
  
  
  
  
  
  
  
**win xp,2003环境上线cobaltstrike**  
  
  
  
  
  
  
  
  
  
  
  
  
  

  

 **0** **1** **思路简介**  

作者：泼猴 @谢公子学安全  

问题分析  
  

在 I-IVV 的需求场景中，经常会遇到一些比较古老的环境，例如 windows xp、win server
2003系统，为了让整体流程更流畅丝滑，对这类场景也需要找到合理的解决方法，且解决方案应尽量贴合现存使用习惯。  
兼容问题的根本点在于，shellcode的兼容性+shellcodeloader的兼容性。  
  
1、shellcode  
  
2、loader  
  
其中最重要的是解决shellcode的问题。

  

解决思路  
  

 **1、自写stage模式的shellcode**  
  
需要学习shellcode的开发环境搭建+开发方法等  
  
参考代码如下：
https://github.com/AgeloVito/CobaltstrikeSource/blob/master/ShellcodeToC_wininet/Shellcode_wininet.cpp
@mai1zhi2

  

![]()

  

优点：shellcode自定义程度高，免杀效果好。  
  
缺点：开发成本较高，且只能适用stage模式  
  
如若开发stageless模式的shellcode还需重写beacon.dll的功能，技术、时间成本就高太多了。

  

 **2、复用低版本的shellcode**  
  
经过测试，CobaltStrike3.1.x 系列中httpListener的shellcode兼容性最高，  
  
其 stage，stageless模式的shellcode都能很好的兼容windows xp、win server 2003系统。

  

![]()

  

![]()

  

优点：获取成本低，且能复用到高版本的CobaltStrike中，不用换c2工具。  
  
缺点：shellcode免杀属性不太好（可以针对shellcode进行混淆等处理，这里不展开）。

  

 **0** **2** **具体实现**  

  

综合各种考量，我们最终选定的低版本cs为 May 4, 2019 - Cobalt Strike 3.14  
  
按照以下步骤将其复用至高版本 Cobalt Strike 中

  

 **0** **1** 配置malleable-c2 profile一致  
  

需要在profile配置中保持一致的几个点

  

stage  
http-stager.uri_x86  
http-stager.uri_x64  
  
stageless  
http-get.uri  
http-post.uri

  

其他的profile配置项参考各版本对应支持的配置就好，相同的配置选项尽量保持一致即可。

  

 **低版本**

  

![]()

  

![]()

  

![]()

  

 **高版本**

  

 **![]()**

  

![]()

  

![]()

  

 **02** 配置host头  
  

如果使用了域前置，我们需要在低版本的profile中配置host头

  

stage  
http-stager.client  
  
stageless  
http-get.client  
http-post.client  
  
http-config.header

  

![]()

  

高版本的cs中，http-config.header
不需要在profile中写为定值，在gui中配置Listener的时候设置就好，且gui中配置的Host Header也会覆盖掉profile的该项值。

  

![]()

  

 **0** **3** 复用低版本shellcode  
  

当我们部署好高版本的c2以后，只要我们遇到windows xp 、win server 2003
这类系统，这时候我们就可以在本地临时起一个低版本cs，通过设置其监听器地址与部署好的高版本监听器相同，生成基于http的x86
shellcode，再使用自写兼容windows xp 、win server 2003
系统的loader，从而达到全版本兼容的可执行程序，我们将便可将其权限上到CobaltStrike4.4上。  
  
我们的CobaltStrike4.4 HttpListener配置如下：

  

![]()

  

CobaltStrike3.14 HttpListener配置如下

  

![]()

  

![]()

  

然后根据使用需求生成 stage或者stageless形式的shellcode即可。

  

![]()

  

 **0** **3** **效果展示**  

  

![]()

  

如下图，完美兼容，权限上到了cs4.4上。

  

![]()

  

  

  
  
  
  
  
  
  
  
  
  
  
  
 **END**  
  
  
  
  
  
  
  
  
  
  
  
  
  

如果对内网感兴趣的话，可以报名参加内网渗透培训哦，详情👇🏻👇🏻👇🏻  

[2022 I-IVV
红队培训之内网渗透](http://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247490313&idx=1&sn=6d246e496eeb10bf5c405a7c6c0519b7&chksm=eaad9b34ddda12224cec9be431e1283df5625ba000116de2f8ee91fe7f27a17c83620d576275&scene=21#wechat_redirect)  

  

也可以在星球里跟我讨论交流。星球里有一千五百多位同样爱好安全技术的小伙伴一起交流！

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

