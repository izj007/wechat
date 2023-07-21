#  【蓝队篇】jsp型webshell被删情况下如何溯源攻击时间

[ 听风安全 ](javascript:void\(0\);)

**听风安全** ![]()

微信号 tingfengsec

功能介绍 潜心学安全

____

___发表于_

收录于合集

以下文章来源于希潭实验室 ，作者ABC_123

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

# Part1前言

在日常的蓝队溯源工作、感染加密勒索病毒后的应急排查工作中，查找攻击者遗留的webshell是一种常规手段，一旦webshell文件被找到后，可以反推出很多信息，最重要的是能确定攻击者攻击时间，以此攻击时间为轴心开展溯源工作会事半功倍。但攻击者经常会把webshell文件删除，并且清理掉所有的访问日志，这种情况下应该怎么溯源确定上传webshell的攻击时间呢？其实对于jsp型或jspx型webshell来说，还是有办法的，因为java的webshell在编译过程中会生成很多临时文件，一直留存在服务器中。

# Part2技术研究过程

首先本地搭建一个tomcat，模拟攻击者行为，上传一个jsp的webshell

![]()

接下来模拟攻击者行为，把log111.jsp文件删除掉，那么怎么找到这个shell遗留的蛛丝马迹呢？  
查看tomcat中间件的\work\Catalina\localhost_\org\apache\jsp
目录，仍然是可以发现这个shell的编译过程中产生的几个文件的，这3个文件攻击者一般不会删除，也不会更改文件的时间属性。所以，这些jsp型webshell文件在编译过程中生成的class文件的时间属性，往往是比较准确的，而jsp文件的时间属性，很多攻击者会改成与web应用部署一样的时间，去迷惑蓝队工作人员。原理：客户端访问某个jsp
、jspx文件时，Tomcat容器或者Weblogic容器会将 jsp 文件编译成java文件和class文件，这两份文件均会存储在容器的某个目录中。  
如下图所示，可以确定攻击者的攻击时间了：

![]()

如果需要进一步验证此文件是正常文件还是webshell文件，就需要使用jadx-
gui工具对此class文件进行反编译了。如下图所示，很清楚看到了冰蝎webshell的代码特征。  

![]()

注：虚拟机环境证明，tomcat中间件即使重启后，这些编译产生的临时文件也是会一直存在的。  
接下来在weblogic中间件上进行同样的操作，制作一个war包上传到weblogic环境中：

![]()

  

![]()

接下来将log.jsp删除后，试着查看在weblogic中间件下还有什么蛛丝马迹可以查询，发现在/jsp_servlet/目录下，还是有几个webshell编译生成的文件，这里与tomcat中间件不同的是，路径中的/hwr7e2/是随机生成的一个目录，需要根据经验具体问题具体分析：  

![]()

除此之外，还可以找到上传的war包，一样可以确定攻击时间，一般在这个目录下边：\user_projects\domains\base_domain\servers\AdminServer\upload，即使删掉jsp文件，重启weblogic后，这个war包文件也会一直存在。

![]()

# Part3总结  

  1. jsp、jspx型webshell文件被删除后，可以通过查找编译生成的class文件的方式去确定攻击时间。如果攻击者将webshell时间属性改掉，也可以通过此方法获取真实的攻击时间。

  2. 对于tomcat、weblogic中间件，除非攻击者删除编译生成的文件，否则重启后这些文件也会一直留存在Web服务器中，成为溯源攻击者的一个重要证据。

  3. 即使攻击者非常细心，把这些class文件全部删干净了，那么借助一些取证工具或者专业设备，还是可以溯源出来的，这个将来会专门写一篇文章讲解。

  
  
不可错过的往期推荐哦  

    
    
      
    
    
    ![]()
    
    [再捉“隔壁小王”](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491554&idx=1&sn=d76f246744e3c3342d53cec71a32c132&chksm=cf272178f850a86eea4650b20ace55a6602e475be79aecaac6d0a24f75202c5935b1b2957b71&scene=21#wechat_redirect)
    
    [达梦数据库手工注入笔记](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491479&idx=1&sn=41e8beebfc19513887373c897985ca0d&chksm=cf27210df850a81ba9db2e42cd472b14392dfe9b6758d1aba1d74035ef7988daa02952e61c55&scene=21#wechat_redirect)  
    
    
    [记一次源码泄露引发的惨案](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491431&idx=1&sn=c4aecf32ee8a7269c27b2b2d1416787b&chksm=cf2721fdf850a8eb4db7a006f997af13456e06e8b2fd458eee6c7b0ab27091e178283ac7c335&scene=21#wechat_redirect)  
    
    
    [某运营商外网打点到内网横向渗透的全过程](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491347&idx=1&sn=732617c7dd6e4bf2abd4a0c2199906cc&chksm=cf272189f850a89f697b402633f3b4f59beaee09e26db09e2661fb9f3713b85dab939b50c3af&scene=21#wechat_redirect)
    
    [从外网绕过沙箱逃逸再到内网权限提升的一次常规渗透项目](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491299&idx=1&sn=61f20ea8070c95819343fd7b34bc1407&chksm=cf272079f850a96fa1f8ad795deb139787dda6fbb672b90c1aee70da2bf0a894dd14abf42ea8&scene=21#wechat_redirect)
    
    [一次市hvv及省hvv的思路总结](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247491241&idx=1&sn=97a8d9cf665f4aeefe8eefb5020a844a&chksm=cf272033f850a925de5f68c315a005d4f048282bbf161ed03a78e431f3fa1bcb525dc0d39655&scene=21#wechat_redirect)
    
    [带防护的Windows域渗透](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247490417&idx=1&sn=963e0943a530f4f1915e7b163ad749be&chksm=cf2725ebf850acfd6a2069742aa88197c3c8901acf69831ea187524956ff8dd7b9cc94e58169&scene=21#wechat_redirect)  
    
    
    [渗透实战|NPS反制之绕过登陆验证](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247490397&idx=1&sn=ddf394bb24bcfa7063d9a31d7d65af4f&chksm=cf2725c7f850acd1143f58335fbf8efe146574d6f1cfdce68c374c4cc283fe4b640f6a17a131&scene=21#wechat_redirect)
    
    [干货｜从无到有学习Golang编写poc&exp](http://mp.weixin.qq.com/s?__biz=Mzg3NzIxMDYxMw==&mid=2247490364&idx=1&sn=7da66c070a672a4e8ec6e3b5c4090e6b&chksm=cf2725a6f850acb0a30ae6a084902675519e19243c4bb0f91ea8754f7e161cd87521af195492&scene=21#wechat_redirect)

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

