#  【蓝队篇】Weblogic反序列化攻击不依赖日志溯源攻击时间

[ 听风安全 ](javascript:void\(0\);)

**听风安全** ![]()

微信号 tingfengsec

功能介绍 潜心学安全

____

___发表于_

收录于合集

以下文章来源于希潭实验室 ，作者abc123info

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM6fmEcY2bcaelEq3UFVKWcPX4siaYoqKZHfc2DgtHsekRw/0)
**希潭实验室** .

ABC_123，2008年入行网络安全，某部委网络安保工作优秀个人，某市局特聘网络安全专家，某高校外聘讲师，希潭实验室创始人。Struts2检测工具及Weblogic
T3/IIOP反序列化工具原创作者，擅长红队攻防，代码审计，内网渗透。

![]()

**免责声明** 由于传播、利用本公众号听风安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号听风安全及作者不为 **此**
承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！

公众号现在只对常读和星标的公众号才展示大图推送，

建议大家把听风安全设为 **星标** ，否则可能就看不到啦！

\----------------------------------------------------------------------

 **Part1 前言
**WebLogic是美国Oracle公司出品一款中间件产品，在国内使用也比较广泛。从2015年开始至今，接连爆出过10几个可直接利用的Java反序列化漏洞，相关漏洞的原理也越来越复杂。每次应急响应过程中，遇到Oracle公司的产品都会特别头疼，因为其日志结构太过繁杂，相关介绍文档也很少，想弄明白是需要下一番功夫的。在日常的应急响应过程中，为了解决这个问题，我曾经反复搭建环境，反编译各种Weblogic攻击工具，跟踪其源代码，研究可以不依赖日志分析去快速解决Weblogic中间件应急响应的方法，流量监控设备通常会有各种报警，混杂在一起的时候，也很难确定准确的攻击时间。
**对于早期Weblogic反序列化攻击，我做应急响应的时候，ls
-lah看一下Weblogic几个目录下新增的文件及文件时间属性，就能判断出攻击者用的是哪一款Weblogic漏洞利用工具，而且还可以确定准确的攻击时间，根本就不用去看日志。**
下面分享一下这个技巧，后续也会花很大篇幅讲讲Weblogic的日志结构及各种漏洞的应急溯源技巧，因为牵扯到Oracle公司的产品，都会很复杂。花很大精力：
**  Part2 具体研究过程 **

  *  **虚拟机下测试**

本地搭建Weblogic的环境如下：![]()使用Weblogic反序列化工具进行攻击，查看服务器的/temp/目录，会生成一个.tmp格式的临时文件，然后查看此文件属性，得知攻击者执行命令的准确时间。![]()将此tmp文件拷贝出来，改扩展名为.jar，使用jadx-
gui工具反编译，即可看到攻击代码，证明此文件确实是攻击者遗留下来的。![]()很多weblogic反序列化利用工具为了能通过T3协议回显命令执行结果，都有类似的文件落地，也有的weblogic反序列化利用工具为了实现无文件落地回显，是通过defineClass方法，从byte[]字节码中还原一个Class对象，实现无文件落地注入构造回显，这种应急方法不再适用。

  *  **Weblogic回显的实现原理：**

应急方法讲完之后，接下来看一下相关工具为什么会向/temp/目录写文件，首先要大体讲一下T3反序列化回显的原理：Weblogic对外提供Web服务，会开放7001等端口，这个端口是Web协议、T3协议、IIOP协议并存的，而T3协议允许客户端远程调用Weblogic服务端的类，然后把执行结果再传输给客户端。因此攻击者会在本地事先编译好一个具有执行命令、上传文件等功能的java类，
**接着将编译好的文件上传至服务器上**
，通过URLClassLoader加载这个编译文件，在服务端绑定一个实例，进而实现T3下的Weblogic反序列化回显，其实有点类似于RMI远程方法调用的过程。
这里要指出，有的Weblogic利用工具向服务器写入临时文件时，并没有关闭文件句柄，导致文件会一直存在服务器上。

  *  **常见Weblogic利用工具特征  **

 **  1 **
首先看第一款实现Weblogic反序列化回显的利用工具，是由冰蝎作者rebeyond大牛编写的。我记得大概是15年年底时，冰蝎作者rebeyond第一个公布出Weblogic
T3反序列化回显方法，而且给出了相关的代码。后续的研究者的回显方法基本上与rebeyond的差不多，通过加载字节码方式实现无文件落地回显。  
我们反编译一下rebeyond的工具，看看代码的具体实现过程：![]()通过Connect按钮事件跟进，一直跟到GenPayload类的Gen方法，很明显可以看到，这款工具会向服务器的临时目录写入1vBLBK.tmp临时文件：![]()之后再通过URLClassLoader类加载这个tmp文件，在服务端绑定一个实例，进而实现T3回显。![]()
**  2 **
接下来看另一款Weblogic反序列化利用工具：![]()利用成功后，会在服务器上生成H3y5ec.tmp临时文件。![]()之后同样使用URLClassLoader类加载，实现T3回显。![]()
**  3 **
接下来看第3款Weblogic反序列化利用工具：![]()可以看到，也会向服务器写一个临时文件![]()同样是使用URLClassLoader类加载，实现T3回显：![]()

 **  Part3 总结 **

 **1.
**早期的Weblogic反序列化利用工具，为了实现T3协议回显，都会向服务器上写入一个临时文件。近几年随着无文件落地的流行，Weblogic的回显基本上都是找一个实现了defineClass方法的类，通过还原字节码方式实现回显，这种应急方法不再适用。
**2.
**应急溯源过程中，日志量通常浩如烟海，而且容易被攻击者删掉。在此情况，研究一下攻击者常用工具的原理，总结一些小技巧，在日常应急溯源工作中，会事半功倍，省去很大的工作量。

  

不可错过的往期推荐哦  

    
    
      
    
    
    ![]()
    
    [从钓鱼邮件溯源到反制上线](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247492153&idx=1&sn=ff299ee157f8dbcdc4a554bc25aac1c4&chksm=cf24dca3f85355b537b9d9df4da4490b0b44a8b3834ccc5f76cac8f16e5be6781db46b8e71ea&scene=21#wechat_redirect)  
    
    
    [蓝队分析辅助工具箱V0.58更新，兼容jdk8至jdk20|shiro解密|哥斯拉解密|各种java反编译](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491587&idx=1&sn=0dffe6cc16ebbc39d9aa446bc0b97787&chksm=cf24de99f853578fa57bad94951cdcee8776cfe572cb8158ea17e5d8fedc6cfbfd484e83157a&scene=21#wechat_redirect)  
    
    
    [记一次渗透中的Password加密爆破过程](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491576&idx=1&sn=98d4addfa549835f78a25e02c483a71d&chksm=cf272162f850a874d7ef75ce1af89e0ac9ee4f358f4a0fc03fd671a02917c2f18d4e1f5b14f4&scene=21#wechat_redirect)  
    
    
    [【蓝队篇】jsp型webshell被删情况下如何溯源攻击时间](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491565&idx=1&sn=6831842335da082a0b57086db19e1d17&chksm=cf272177f850a861ec3acf774055281465c163f0e763ae8f74e61b8b03489312af751d51fac2&scene=21#wechat_redirect)
    
    [再捉“隔壁小王”](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491554&idx=1&sn=d76f246744e3c3342d53cec71a32c132&chksm=cf272178f850a86eea4650b20ace55a6602e475be79aecaac6d0a24f75202c5935b1b2957b71&scene=21#wechat_redirect)
    
    [达梦数据库手工注入笔记](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491479&idx=1&sn=41e8beebfc19513887373c897985ca0d&chksm=cf27210df850a81ba9db2e42cd472b14392dfe9b6758d1aba1d74035ef7988daa02952e61c55&scene=21#wechat_redirect)
    
    [某运营商外网打点到内网横向渗透的全过程](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491347&idx=1&sn=732617c7dd6e4bf2abd4a0c2199906cc&chksm=cf272189f850a89f697b402633f3b4f59beaee09e26db09e2661fb9f3713b85dab939b50c3af&scene=21#wechat_redirect)
    
    [从外网绕过沙箱逃逸再到内网权限提升的一次常规渗透项目](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491299&idx=1&sn=61f20ea8070c95819343fd7b34bc1407&chksm=cf272079f850a96fa1f8ad795deb139787dda6fbb672b90c1aee70da2bf0a894dd14abf42ea8&scene=21#wechat_redirect)
    
    [一次市hvv及省hvv的思路总结](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491241&idx=1&sn=97a8d9cf665f4aeefe8eefb5020a844a&chksm=cf272033f850a925de5f68c315a005d4f048282bbf161ed03a78e431f3fa1bcb525dc0d39655&scene=21#wechat_redirect)
    
    [带防护的Windows域渗透](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247490417&idx=1&sn=963e0943a530f4f1915e7b163ad749be&chksm=cf2725ebf850acfd6a2069742aa88197c3c8901acf69831ea187524956ff8dd7b9cc94e58169&scene=21#wechat_redirect)

点击下方名片，关注我们  
觉得内容不错，就点下“ **赞** ”和“ **在看** ”  
如果不想错过新的内容推送可以设为 **星标**![]()  

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

