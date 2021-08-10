##  高危漏洞狙击框架:woodpecker-framework

原创 c0ny1 [ 回忆飘如雪 ](javascript:void\(0\);)

**回忆飘如雪** ![]()

微信号 gh_31665e4938a5

功能介绍 记录工作生活所思所想

____

__

收录于话题

#woodpecker-framework

1个

**  0x01 **

 **简介**

  

woodpecker-
framework是一款高危漏洞综合利用框架，目的是可以狙击高危漏洞，拿到权限！其设计是由我在日常红队外围打点经验抽象得来。它的每个模块和外围打点的主要流程是一一对应的。

  

比如遇到一个具体的外围应用，渗透测试的流程是：

  

1\. 探测当前应用所有攻击面和风险点 （信息探测模块）

2\. 使用poc探测漏洞是否存在 (精准检测模块)

3\. 通过exp拿下webshell (深度利用模块)

4\. 遇到奇葩环境漏洞环境自动化无法打死，需要人工生成payload （荷载生成模块）

5\. 人工构造payload时经常需要做一些常规操作，比如把Class变成BCEL编码，runtime.exec命令变形等等 （辅助模块）

  

下面围绕weblogic和shiro这两个高频漏洞应用来介绍详细每个模块。

  

  

 **  0x02 **

 **信息探测模块（InfoDetector）**

  

  

 **信息探测模块的任务是寻找当前应用最薄弱的点。**
显然有用的信息是判断的重要依据。这里探测的信息不是什么操作系，中间件，cms之类的指纹识别。而是针对具体应用的攻击面和风险点的探测，比如weblogic就会探测如下信息。

  

1\. weblogic是那个版本

2\. 协议是否开启t3/iiop协议

3\. web端口是否可以访问到console，wls，async之类的组件

  

![](https://gitee.com/fuli009/images/raw/master/public/20210810165707.png)

顺便值得一提的是，我们探测t3/iiop协议的时候，还需要探测它们是否被设置为禁止连接，不然探测出open也是无法利用的。如上图的t3开启了但是配置了如下过滤。

![](https://gitee.com/fuli009/images/raw/master/public/20210810165714.png)

 **这些信息有什么用呢？** 当然是让我们知道面前这个weblogic的薄弱点在哪里，后续攻击的计划应该是:t3和iiop系列漏洞不用测试了，wls-
wsat组件的xmldecoder反序列化漏洞可以看看。  

  

  

 **  0x03 **

 **精准检测模块(POC)**

 **  
**

 **  
**

 **精准检测模块的任务是使用poc去判断漏洞是否存在。** 显然精准是这个模块关注的问题，我们的原则是误报可以原谅，但是漏报坚决杜绝。

  

那现实如此复杂的漏洞环境，怎么实现精准检查呢？woodpecker插件的检测原则是尽可能的实现以下所有检测方案。

  

1\. 回显检测

2\. dnslog检测

3\. 间接检查

4\. 写文件检测

5\. 触发补丁检测

6\. 延时检测

7\. 特定特征检测

8\. ....

  

![](https://gitee.com/fuli009/images/raw/master/public/20210810165715.png)

这里我细说下3,5和7这三个方案，其他方案顾名思义。

  

间接检测是不通过直接触发漏洞来检测，而是通过其他方面间接来验证。举2个例子，shiro
key的检测由开始的通过回显，dnslog之类的直接检测变成了现在统计rememberMe个数。weblogic漏洞检测则可通过下载黑明单class来验证是否被修复。这些方法很巧妙，在漏检中有四两拨千斤的作用。

  

1. [一种另类的 shiro 检测方式](https://mp.weixin.qq.com/s?__biz=MzIzOTE1ODczMg==&mid=2247485052&idx=1&sn=b007a722e233b45982b7a57c3788d47d&scene=21#wechat_redirect)

2.  [红蓝必备 你需要了解的weblogic攻击手法](https://mp.weixin.qq.com/s?__biz=MzUzNTEyMTE0Mw==&mid=2247484559&idx=1&sn=a9054c0a433cb288df2820363a889446&scene=21#wechat_redirect)

  

触发补丁检测就是提交可触发补丁的payload，然后看是否拦截来确定漏洞是否修复。比如CVE-2019-2725我们就可以发送带<class>标签的payload，若如下提示非法标签说明漏洞修复了。

![](https://gitee.com/fuli009/images/raw/master/public/20210810165716.png)

特定特征检测就是通过respone的某些特征可以知道漏洞是否修复，比如CVE-2020-14882/3漏洞修复后的响应如下,那咱们就可以通过repsoen状态码为500,返回包中存在The
server encountered an unexpected condition which prevented it from fulfilling
the request.提示来判断。

![](https://gitee.com/fuli009/images/raw/master/public/20210810165718.png)

  

  

 **  0x04 **

 **深度利用模块(Exploit)**

  

  

 **深度利用模块的任务是发挥漏洞的最大利用价值**
。比如一个RCE可以干的事情很多，命令执行，写文件，读文件，反弹shell，注入内存马，开启bindshell等等。不过最后我梳理了下，很多功能都是有交集的，比如反弹shell可以通过命令执行来反弹，读文件可以通过webshell来读。所以在红队行动中，真正对我们有用的一般是三个功能，woodpecker插件编写的原则上要求深度利用模块必须实现这3个功能，并保证稳定性。

  

1\. 写文件

2\. 命令回显

3\. 注入内存马

  

![](https://gitee.com/fuli009/images/raw/master/public/20210810165720.png)

  

  

 **  0x05 **

 **荷载生成模块(Payload generator)**

  

  

 **荷载生成模块的任务是帮助红队人员快速生成自定义payload。**
自动化并不能解决所有问题，当遇到奇葩环境时就需要人工介入。比如当shiro漏洞遇到未知中间件时，可能无法回显也无法注入内存马，这时就需要人工构造payload了。但是每次都要先生成序列化数据，设置key，选择加密模式，非常浪费时间。而woodpecker
shiro漏洞插件的荷载生成模块可以一键生成。

![](https://gitee.com/fuli009/images/raw/master/public/20210810165724.png)

  

  

 **  0x06 **

 **辅助模块(Helper)**

  

  

 **该模块的任务是将漏洞检测和利用中经常要进行的操作自动化，节省时间。**

  

比如在java命令执行漏洞中无法使用带有管道符的命令，需要我们去转换下命令。当然有Jackson_T这样的在线网站，这里我编写成了本地插件。

  

https://github.com/woodpecker-appstore/runtime-exec-encoder

![](https://gitee.com/fuli009/images/raw/master/public/20210810165725.png)

同时如果想通过命令执行漏洞写一个shell的话，往往需要转义下，这个过程也是比较繁琐的。可以使用EchoToFileConverter插件来解决。

  

https://github.com/woodpecker-appstore/EchoToFileConverter

![](https://gitee.com/fuli009/images/raw/master/public/20210810165726.png)

  

  

 **  0x07 **

 **最后的话**

  

如果你比较认同这样的设计，并有能力编写插件。欢迎到github提交pr或者插件。

  

1\. 框架主页 https://woodpecker.gv7.me

2\. 框架仓库 https://github.com/woodpecker-framework

3\. 插件仓库 http://github.com/woodpecker-appstore

![]()

c0ny1

如有帮助，打赏c0ny1一瓶1664吧!

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

如有帮助，打赏c0ny1一瓶1664吧!

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

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

高危漏洞狙击框架:woodpecker-framework

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

