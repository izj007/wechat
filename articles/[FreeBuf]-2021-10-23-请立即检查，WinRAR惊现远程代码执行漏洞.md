#  请立即检查，WinRAR惊现远程代码执行漏洞

原创 苏苏  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 国内网络安全行业门户

____

__

收录于话题

据security affairs消息，网络安全专家Igor Sak-
Sakovskiy发现了WinRAR的一个远程代码执行漏洞，漏洞编号CVE-2021-35052。

该漏洞出现在WinRAR的Windows试用版本，漏洞版本为5.70，黑客可利用该漏洞远程攻击计算机系统。

Igor Sak-
Sakovskiy表示，“这个漏洞允许攻击者拦截和修改系统发送给用户的请求，这样就可以在用户电脑上实现远程代码执行(RCE)。我们也是在WinRAR
5.70 版中偶然发现了这个漏洞。”

安全专家在安装了WinRAR后，发现它存在一个JavaScript 错误，具体表现形式是，在浏览器中弹出下图这样的错误窗口。

![](https://gitee.com/fuli009/images/raw/master/public/20211023182336.png)

经过一系列的测试之后，安全专家发现软件试用期满后，软件会开始显示错误消息，基本上是每三次执行一次。这个弹出“错误显示”的窗口是通过系统文件mshtml.dll报错来实现，它和WinRAR一样，都是用
Borland的 C++语言编写的。

安全专家使用Burp Suite作为默认的Windows代理，以此拦截消息显示时生成的流量。

WinRAR在软件试用期结束后，会通过“notifier.rarlab[.]com”来提醒用户，安全专家在分析了发送给用户的响应代码后发现，攻击者会把提醒信息修改成“301永久移动”的重定向消息，这样就可以将后续所有的请求缓存重定向到恶意域中。

安全专家还注意到，当攻击者可以访问同一网络域之后，就会发起ARP欺骗攻击，以便远程启动应用程序，检索本地主机信息并执行任意代码。

随后，我们试图拦截、修改WinRAR发送给用户的反馈信息，而不是去拦截和更改默认域，让“notifier.rarlab.com”每次都响应我们的恶意内容。我们进一步发现，如果如果响应代码被更改为“301永久移动”，并重定向缓存到我们的“attacker.com”恶意域，这样所有的请求都会转到“attacker.com”。

![](https://gitee.com/fuli009/images/raw/master/public/20211023182343.png)

最后，安全专家指出，第三方软件中的漏洞会对企业和组织产生严重风险。这些漏洞的存在，可以让攻击者访问系统的任何资源，甚至是访问托管网络中的所有资源。

“在安装应用程序之前，我们不可能确保每一个程序都没问题，因此用户的策略对于外部应用程序的风险管理至关重要，以及如何平衡应用程序的业务需求和安全风险，也依赖于用户的选择。如果使用、管理不当，很有可能会产生非常严重的后果。”

参考来源：

https://securityaffairs.co/wordpress/123652/hacking/winrar-trial-flaw.html

![]()

  

精彩推荐

  
  
  
  
  
 **
**![](https://gitee.com/fuli009/images/raw/master/public/20211023182344.png)****  

[![](https://gitee.com/fuli009/images/raw/master/public/20211023182345.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486647&idx=1&sn=13ae89f5104291b30864af54ff28ceda&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20211023182346.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486625&idx=1&sn=b68fcc53d322bb9a2e43d00a112ca40d&chksm=ce1cf63ef96b7f287356248af6a7c190268d07993c30b87488a27f8b1a6cb71f68be08ea4b78&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20211023182347.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486586&idx=1&sn=8fb235328402751e06c0578e05f3c905&scene=21#wechat_redirect)
** ** ** ** ** ** **![]()**************

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

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

请立即检查，WinRAR惊现远程代码执行漏洞

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

