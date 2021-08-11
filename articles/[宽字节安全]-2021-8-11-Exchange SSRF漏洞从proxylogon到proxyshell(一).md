##  Exchange SSRF漏洞从proxylogon到proxyshell(一)

原创 unicodeSec  [ 宽字节安全 ](javascript:void\(0\);)

**宽字节安全** ![]()

微信号 gh_2de2b9f7d076

功能介绍 二十年专注安全研究，漏洞分析

____

__

收录于话题

前几天Exchange又爆出一个新洞，类似于之前的ProxyLogon。本片文章更多注重于基础。

## 一. 调试环境

C#语言类似与Java，同样可以很轻松的反编译。在我的Java线上课程中第一节课就是讲解Java调试的原理，方法以及环境。如图![](https://gitee.com/fuli009/images/raw/master/public/20210811181054.png)

对于分析Exchange漏洞来讲，与java没有什么不同，同样搭建调试环境。Exchange类似于OA应用，只不过运行在IIS中。

首先下载dnspy，找到Exchange安装目录下的Dll文件，类似于Java的jar包。拖入dnspy中，并附加到Exchange相关的IIS进程中。

![](https://gitee.com/fuli009/images/raw/master/public/20210811181055.png)

通过附加进程，即可通过dnspy调试IIS服务器中相关的应用。在Exchange中其实存在很多应用，例如AutoDiscover，ecp等，在IIS目录中是这样的。![](https://gitee.com/fuli009/images/raw/master/public/20210811181056.png)

那么我们应该附加到哪个进程呢，可以看图![](https://gitee.com/fuli009/images/raw/master/public/20210811181057.png)

找到应用程序名，在附加到进程的进程参数中，可以看到应用程序名，点击附加即可调试该web应用。

再运行一下proxylogon的exp，即可成功触发断点。![](https://gitee.com/fuli009/images/raw/master/public/20210811181059.png)

## 二. proxylogon ssrf分析

首先我们要了解proxylogon
ssrf漏洞的分析。对于Exchange来讲，很多功能都是通过Exchange发送响应的http请求到处理的组件。而且Exchange在向自己的组件发送http请求的时候，也会自动添加认证。这也就是Exchange绕过认证的原理。

首先分析proxylogon的ssrf漏洞
Exchange对于每一个请求，都会调用`Microsoft.Exchange.FrontEndHttpProxy.dll!Microsoft.Exchange.HttpProxy.ProxyModule.SelectHandlerForUnauthenticatedRequest`去处理。

微软可能是这样想的，对于某些请求的资源，比如说js,png等资源文件，没有必要通过认证才可以访问。所以微软默认对于这类资源文件的请求是不做认证的。在`ProxyModule.SelectHandlerForUnauthenticatedRequest`中，会通过大量的if-
else去询问各个Handle能否处理。

![](https://gitee.com/fuli009/images/raw/master/public/20210811181100.png)

在`Microsoft.Exchange.HttpProxy.BEResourceRequestHandler`中，通过后缀名来判断本次请求是否为请求静态资源，并且请求中是否存在
Cookie的key为X-
BEResource，代码如图。![](https://gitee.com/fuli009/images/raw/master/public/20210811181101.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210811181102.png)

最终通过多态，调用ProxyRequestHandler的run方法。

![](https://gitee.com/fuli009/images/raw/master/public/20210811181103.png)

在`proxyRequestHandle.InternalBeginCalculateTargetBackEnd`中，通过多态，实际调用`BEResourceRequestHandler.
ResolveAnchorMailbox` 方法，获取Cookie值为X-X-
BEResource![](https://gitee.com/fuli009/images/raw/master/public/20210811181104.png)

通过`BackEndServer.FromString`方法，将Cookie通过`~`分割，分为FQDN与版本。![](https://gitee.com/fuli009/images/raw/master/public/20210811181105.png)

在`proxyRequestHandle.BeginProxyRequest`方法中，首先调用`GetTargetBackEndServerUrl`获取需要请求的地址。![](https://gitee.com/fuli009/images/raw/master/public/20210811181106.png)

可以看到，其实通过上面的那一步获取到的fqdn地址作为请求的url。

在`proxyRequestHandle.BeginProxyRequest`方法中，随后会调用`CreateServerRequest`向上一步获取到的url对象发起请求。

在`CreateServerRequest`方法中，调用`PrepareServerRequest`创建请求。`PrepareServerRequest`方法中，调用`proxyRequestHandle.ShouldBlockCurrentOAuthRequest`判断是否有权限发起该请求。![](https://gitee.com/fuli009/images/raw/master/public/20210811181107.png)

而`proxyRequestHandle.ShouldBlockCurrentOAuthRequest`判断比较糙，只是简单返回ProxyToDownLevel的值。

只有ProxyToDownLevel在GetTargetBackEndServerUrl方法中将其设置为true时才调用该方法。此方法检查用户是否已通过身份验证，如果未通过验证，则返回HTTP
401错误。
幸运的是，我们可以GetTargetBackEndServerUrl通过修改Cookie中的服务器版本来防止设置此值。如果版本大于Server.E15MinVersion，ProxyToDownLevel则为false。进行此更改后，我们成功通过了后端服务（自动发现服务）的身份验证。详细代码见上面的图

剩下的部分，就是正常的SSRF请求。不过这个SSRF相比较其他而言，还可以将Post请求体的内容，当前http的请求头，一并通过http发送。恰巧还会自己添加认证信息。

## 三. proxyshell ssrf部分分析

在微软的通告中，提到在autodiscover服务中存在一处ssrf漏洞。原理很有可能与proxyshell的原理相类似。同样我们寻找ProxyRequestHandler的子类。

![](https://gitee.com/fuli009/images/raw/master/public/20210811181108.png)

回到最开始的`Microsoft.Exchange.FrontEndHttpProxy.dll!Microsoft.Exchange.HttpProxy.ProxyModule.SelectHandlerForUnauthenticatedRequest`中，这里同样也支持调用`AutodiscoverProxyRequestHandler`![](https://gitee.com/fuli009/images/raw/master/public/20210811181109.png)

在ProxyRequestHandler.GetTargetBackEndServerUrl中，已经修复了proxylogon的漏洞。但是，可以通过GetClientUrlForProxy方法，同样可以获取SSRF的请求地址。

在GetClientUrlForProxy中，如果我们的请求符合IsAutodiscoverV2Request函数的请求，那么将会`UrlHelper.RemoveExplicitLogonFromUrlAbsoluteUri`函数移除多余的部分，并与localhost拼接为ssrf的请求地址。![](https://gitee.com/fuli009/images/raw/master/public/20210811181110.png)

![]()

其实就是从url中，移除不需要的部分。例如对于`/Autodiscover/autodiscover.json?a=unicodesec@unicodesec.com/mapi/nspi`来讲，如果explicitLogonAddress为`/Autodiscover/autodiscover.json?a=unicodesec@unicodesec.com`，那么最终请求的地址为localhost/mapi/napi

explicitLogonAddress值在`EwsAutodiscoverProxyRequestHandler.ResolveAnchorMailbox`中复制，从请求参数的Email中获取

![](https://gitee.com/fuli009/images/raw/master/public/20210811181111.png)

最终payload

![](https://gitee.com/fuli009/images/raw/master/public/20210811181112.png)

如果服务器存在漏洞，则返回当前权限。

    
    
    GET /Autodiscover/autodiscover.json?a=unicodesec@unicodesec.com/mapi/nspi HTTP/2  
    Host: localhost  
    Accept: */*  
    Cookie: Email=Autodiscover/autodiscover.json?a=unicodesec@unicodesec.com  
    Content-Length: 0  
    

下一部分将分享exp的编写方法

欢迎大家报名线上java课程，从代码审计的角度讲解基础漏洞以及Java特有的安全漏洞。

Java安全课程目录，由于篇幅原因只分享部分课程目录，前面还会有很多基础的内容，我会整理为pdf文档发给大家。

#### 第一章

Classloader 原理以及在java安全中的使用

主要讲解Classloader的原理，不同classloader的类型，双亲委派模型以及如何通过反射调用defineClass。在后面的java漏洞利用中，classloader作为将类与JVM的桥梁将会扮演很重要的角色。

##### 讲解内容

classloader的原理，以及如何动态加载一个class文件

###### 双亲委派模型

Tomcat中classloader的讲解，在tomcat中，并没有完全遵守双亲委派模型。为了实现更好的隔离，tomcat使用另外一种方式实现classloader
几种常见的classloader的使用讲解以及在不同漏洞中的利用方法

#### 第二章

Java
asm字节码的使用指南。在java反序列化的payload中，需要动态生成一个javaclass文件。可以通过手工，根据jvm指令构造。也可以通过javaassist自带的编译器构造。

##### 讲解内容

动态生成字节码在java中的作用 javaassist的使用指南 java asm的使用指南 java asm在ysoserial中的使用情况 再探Java
反射原理，使用java asm实现反射机制

#### 第四章

java反序列化协议解析， java 序列化的流协议解读 java ObjectInputstream代码解析 如何实现python读写java 流
python生成 8u20gadget 8u20 漏洞的原理 python生成反序列化Gadget的方法

#### 第七章

内存马实战与查杀。为了更好地对抗ids，waf设备。利用注入内存马技术，修改java中间件中url对应对象的路由部分。在很多waf规则防护中，一般只认为用户请求的url的后缀为jsp才会产生webshell请求，但是我们可以将请求某个静态文件的路由映射到webshell对象中去骗过waf等安全设备

##### 讲解内容：

中间件的简单架构，这里主要讲解tomcat，介绍四大组件
J2ee标准，主要是servlet这几个标准，主要是在servlet3.0中，支持动态添加servlet，filter
如何插入内存马。可能更多会讲如何跨webapp应用插入内存马 如何查杀。对于内存马，不光要会插，还要会清除

#### java 漏洞实战分析

调试分析weblogic漏洞 讲解内容     T3协议漏洞 如何调试weblogic T3协议，以及T3协议内容大致讲解 如何diff补丁包
如何判断目标服务器是否存在T3反序列化漏洞（查看黑名单等方法 如何实现T3协议回显代码（cve-2020-2555 + nashorn的回显），报错回显
T3协议如何与IIOP协议配合绕过Nat 如何应急响应weblogic服务器以及如何在不停机的情况下打热补丁 weblogic xmldecoder漏洞

欢迎对于课程有兴趣的学员积极报名参加课程，详情联系下方客服。

期待你的加入！  

![](https://gitee.com/fuli009/images/raw/master/public/20210811181113.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210811181114.png)

  

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

Exchange SSRF漏洞从proxylogon到proxyshell(一)

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

