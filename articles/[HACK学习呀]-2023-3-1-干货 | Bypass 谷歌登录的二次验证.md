#  干货 | Bypass 谷歌登录的二次验证

HACK学习  [ HACK学习呀 ](javascript:void\(0\);)

**HACK学习呀** ![]()

微信号 Hacker1961X

功能介绍
HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

____

___发表于_

收录于合集

#身份验证 2 个

#谷歌 1 个

#漏洞挖掘 77 个

# Bypass 谷歌登录的二次验证

大家好， 今天的文章将非常有趣，因为我们将讨论一种方法，我们可以使用这种方法通过欺骗受害者来轻松绕过“ Google 双因素身份验证”。

![](https://gitee.com/fuli009/images/raw/master/public/20230301093742.png)

# 首先获取凭证

要绕过任何谷歌帐户的双因素身份验证，您必须首先拥有该帐户的用户名和密码，还必须使用网络钓鱼和社会工程来获取凭据。

但问题是我们如何做到这一切？这一切都很容易做到，你只需要使用一个名为“ Advphishing ”的工具

你就可以 通过使用假的 WhatsApp 号码轻松获取受害者的帐户用户名、密码甚至 OTP 。

![](https://gitee.com/fuli009/images/raw/master/public/20230301093806.png)

# 我们会做什么 ？

通常当我们第一次尝试从 google chrome 登录我们的 google 帐户时，它会让我们做一些安全过程来查明那个人是否是正确的人。

谷歌为我们提供了几个成功登录帐户的功能，所有这些功能都有一个名为“点击通知继续”的双因素身份验证功能，其中包含攻击者的设备信息，提醒受害者不允许攻击者登录他的帐户。

所以我们只需要将我们的设备信息替换成受害者正在使用的设备信息，我们就可以对受害者进行诈骗。

因此，在本教程中，你将了解如何通过欺骗受害者来绕过双因素身份验证。

# 社会工程学

真正的步骤从这里开始，我们现在将使用社会工程技术来捕获受害者的设备信息。它很容易实现。

一旦受害者点击您提供的链接，您就可以轻松获得有关其设备的所有深层信息。

![](https://gitee.com/fuli009/images/raw/master/public/20230301093810.png)

# 输入找到的凭据

让我们转到 Google 帐户并输入凭据，但输入密码后不要提交。

![](https://gitee.com/fuli009/images/raw/master/public/20230301093811.png)

# 使用Burpsuite 工具

kali linux操作系统预装的Web应用渗透测试顶级工具，你需要打开它，但我们不设置代理就无法使用它，所以你必须先配置代理。

一切都完成后， “打开”拦截模式，然后转到谷歌帐户并单击 “下一步”。设备信息始终存储在“ User-
Agent”参数中，我们需要将其替换为从足迹中找到的受害设备信息。让我们改变它。

![](https://gitee.com/fuli009/images/raw/master/public/20230301093812.png)

好的 ，我们已经更改了从足迹中获得的所有信息。更改后，转发请求。

![](https://gitee.com/fuli009/images/raw/master/public/20230301093814.png)

我们必须再次遵循我们在上一步中完成的相同过程。更改后转发请求并“关闭”拦截模式。

![](https://gitee.com/fuli009/images/raw/master/public/20230301093816.png)

注意 您必须在很快的时间内完成这两个步骤。

一旦您转发请求，就会向受害者手机发送一条通知警报，并要求允许该帐户登录受害者正在使用的设备。现在受害者会认为请求一定来自我的设备并允许他登录。

绕过

![](https://gitee.com/fuli009/images/raw/master/public/20230301093818.png)

So Easy！我们使用社会工程学技术接管了google帐户是多么容易。

![](https://gitee.com/fuli009/images/raw/master/public/20230301093820.png)

  

 **推荐阅读：**

 **  
**

 **[实战 |
记一次SSRF攻击内网的实战案例](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247504980&idx=1&sn=d8ee8cc63ce8bb937891c0942c59d2e0&chksm=ec1c816bdb6b087d4b4315f81c87c6a8b8a7a9cfe60f42f79bbb9a14877016b6b607f95d962e&scene=21#wechat_redirect)  
**

  

 **[实战 |
记两次应急响应小记](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247508721&idx=1&sn=6bd3d0e1354da8b22cb12cd42587c3d8&chksm=ec1cf7cedb6b7ed8a701c0f460539e5039ea6e35e22c87cd34a6b738651c5d6a9040048faa66&scene=21#wechat_redirect)  
**

  

 **[干货 |
Wordpress网站渗透方法指南](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247508079&idx=1&sn=668535e1e2e29683403cf6dea91d2ad6&chksm=ec1cf550db6b7c46859ff1ab8129eaf46f82a9c9e427be6a3a94ffd9827a0873bf5afc8d209f&scene=21#wechat_redirect)  
**

  

[ **实战 |
记一次CTF题引发的0day挖掘**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247509402&idx=1&sn=af99609cb6685118b96616a8011ac252&chksm=ec1cf0a5db6b79b38ea240c755ccf10b94f3b0c19326297c569ab69abba57e7e8f8093063e66&scene=21#wechat_redirect)  

  

[ **2023年零基础+进阶系统化白帽黑客学习 |
2月份最新版**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247511350&idx=1&sn=4164f4acdcc6216919ebe85b91cccd88&chksm=ec1cf809db6b711f9cc83d2c20f81e815b4bef5150d03c8afc688ca5083290e3507f34f621b5&scene=21#wechat_redirect)  

  

[ **实战 |
记一次邮件系统C段引发的SQL手注和内网渗透**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247509307&idx=1&sn=3b09b33609b5fb4476dae0e84e489d4a&chksm=ec1cf004db6b79127cc7b5fb1a3606054440feb48208a60a759f29ffff0f0659f19826b9ad48&scene=21#wechat_redirect)  

  

 **点赞，转发，在看**

  

![](https://gitee.com/fuli009/images/raw/master/public/20230301093822.png)

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

