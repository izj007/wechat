#  【安全入门】漏洞发现爬虫特扫&Burp插件自动化&白盒扫描

学网络安全到  [ 开源聚合网络空间安全研究院 ](javascript:void\(0\);)

**开源聚合网络空间安全研究院** ![]()

微信号 OSPtech_Cyberspace

功能介绍
山西开源聚合科技有限公司是山西省第一家研发生产销售MOOE信息安全实验室的高科技企业。公司专注于大型在线信息安全实验室的研发、销售和技术服务，为客户提供仿真在线实验软件和解决方案。公司自成立以来一直秉承“专注信息安全，立足教育”的核心理念。

____

__

收录于话题

**![](https://gitee.com/fuli009/images/raw/master/public/20211008182850.png)**

网安教育

培养网络安全人才

技术交流、学习咨询

  

  

➤ 网络爬虫项目演示测试

  

crawlergo&rad&burpsuite&awvs爬虫的对比

![](https://gitee.com/fuli009/images/raw/master/public/20211008182854.png)

  

参考程序员启航的博客：https://blog.csdn.net/aaahtml/article/details/113174227

  

 **1.crawlergo**

crawlergo是一个使用chrome
headless模式进行URL收集的浏览器爬虫。它对整个网页的关键位置与DOM渲染阶段进行HOOK，自动进行表单填充并提交，配合智能的JS事件触发，尽可能的收集网站暴露出的入口。内置URL去重模块，过滤掉了大量伪静态URL，对于大型网站仍保持较快的解析与抓取速度，最后得到高质量的请求结果集合。

安装：需要下载chromium以及linux环境

地址：https://github.com/0Kee-Team/crawlergo

  

 **2.rad**

rad，一款专为安全扫描而生的浏览器爬虫

运行rad

![](https://gitee.com/fuli009/images/raw/master/public/20211008182855.png)

  

  

    
    
    1rad.exe -t http://192.168.111.131/dvwa/index.php  
    

  

![](https://gitee.com/fuli009/images/raw/master/public/20211008182857.png)

 **  
**

 **3.avws**  

Acunetix一款商业的Web漏洞扫描程序，它可以检查Web应用程序中的漏洞，如SQL注入、跨站脚本攻击、身份验证页上的弱口令长度等。它拥有一个操作方便的图形用户界面，并且能够创建专业级的Web站点安全审核报告。新版本集成了漏洞管理功能来扩展企业全面管理、优先级和控制漏洞威胁的能力。

这里选择爬虫扫描

![](https://gitee.com/fuli009/images/raw/master/public/20211008182858.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182900.png)

  

  

➤ AVWS&burp&sqlmap爬虫扫描

  

 **1、利用Awvs配置代理联动Burp爬虫**

![](https://gitee.com/fuli009/images/raw/master/public/20211008182901.png)

  

选择爬虫扫描

![](https://gitee.com/fuli009/images/raw/master/public/20211008182902.png)

  

选择好代理设置

![](https://gitee.com/fuli009/images/raw/master/public/20211008182903.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182905.png)

  

把爬虫的请求保存到文本中

![](https://gitee.com/fuli009/images/raw/master/public/20211008182907.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182909.png)

接着使用sqlmap对文本中的请求快速扫描

    
    
    1python sqlmap.py -l .\avws_sqlmap.txt --batch -smart  
    

  

![](https://gitee.com/fuli009/images/raw/master/public/20211008182910.png)

跑完之后我们去康康结果

![](https://gitee.com/fuli009/images/raw/master/public/20211008182911.png)

  

发现是存在注入的

![](https://gitee.com/fuli009/images/raw/master/public/20211008182912.png)

  

  

➤ Burp-Suite安装插件扫描&灯塔配合goby

  

 **1.Burp-Suite插件项目地址：https://github.com/Mr-xn/BurpSuite-collections**

Burp-Suite-collections，BurpSuite 相关收集项目，插件主要是非BApp Store（商店）

![](https://gitee.com/fuli009/images/raw/master/public/20211008182913.png)

  

武装完Burp-Suite之后，我们就可以使用它进行扫描测试

![](https://gitee.com/fuli009/images/raw/master/public/20211008182918.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182919.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182920.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182921.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182922.png)

  

这里就是Burp-Suite扫描的结果信息

  

 **2.灯塔配合goby**

![](https://gitee.com/fuli009/images/raw/master/public/20211008182924.png)

 ****  

通过灯塔来进行资产收集到敏感文件或ip、域名再配合goby进行漏洞扫描

Penetration_Testing_POC，搜集有关渗透测试中用到的POC、脚本、工具、文章等姿势分享，项目地址：https://github.com/Mr-
xn/Penetration_Testing_POC

Awesome burp extensions一个非常棒的 Burp 扩展，适合那些想用很棒的插件来为他们的 Burp
实例增添趣味的人，项目地址：https://github.com/snoopysecurity/awesome-burp-extensions

  

➤信息收集&源代码进行白盒扫描

  

fottify 全名叫：Fortify SCA ，是HP的产品
，是一个静态的、白盒的软件源代码安全测试工具。它通过内置的五大主要分析引擎：数据流、语义、结构、控制流、配置流等对应用软件的源代码进行静态的分析，分析的过程中与它特有的软件安全漏洞规则集进行全面地匹配、查找，从而将源代码中存在的安全漏洞扫描出来，并给予整理报告。扫描的结果包含详细的安全漏洞信息、安全知识说明、修复意见。

下载详情参考：https://www.shungg.cn/301.html

![](https://gitee.com/fuli009/images/raw/master/public/20211008182925.png)

  

这里我们拿一个项目进行白盒扫描

![](https://gitee.com/fuli009/images/raw/master/public/20211008182926.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182928.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182929.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182930.png)

  

扫描出存在sql注入的地方

  

➤ rad联动xary进行爬虫扫描

  

先运行xray执行监听

    
    
    1.\xray.exe webscan --listen 127.0.0.1:7777 --html-output xxx.html  
    

  

![](https://gitee.com/fuli009/images/raw/master/public/20211008182931.png)

  

然后执行rad进行爬取

    
    
    1.\rad.exe -t http://testphp.vulnweb.com -http-proxy 127.0.0.1:7777  
    

  

![](https://gitee.com/fuli009/images/raw/master/public/20211008182932.png)

浏览器配置代理，监听本地7777端口

![](https://gitee.com/fuli009/images/raw/master/public/20211008182933.png)

![](https://gitee.com/fuli009/images/raw/master/public/20211008182935.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20211008182936.png)

版权声明：本文为CSDN博主「遗憾zzz」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/qq_36241198/article/details/120604274

版权声明：著作权归作者所有。如有侵权请联系删除

  

开源聚合网安训练营

战疫期间，开源聚合网络安全基础班、实战班线上全面开启，学网络安全技术、升职加薪……有兴趣的可以加入开源聚合网安大家庭，一起学习、一起成长，考证书求职加分、升级加薪，有兴趣的可以咨询客服小姐姐哦！

![](https://gitee.com/fuli009/images/raw/master/public/20211008182937.png)

加QQ（1005989737）找小姐姐私聊哦  
  

  
  
 **精选文章**  
  
[环境搭建](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379832&idx=1&sn=79689274fc87d5a12d27029276630980&chksm=f3fd2f4fc48aa659178c6cdf12cfd5f29d7182aebc4920a909da237f4b47150dab11c6005af8&scene=21#wechat_redirect)[Python](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379805&idx=1&sn=b97ec4b71200e48c7b555f26b50b3b2b&chksm=f3fd2f6ac48aa67c551db3a797b4d845a5c3bc4ded0158fd72e9d0968be8c2a7187397575932&scene=21#wechat_redirect)[学员专辑](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379757&idx=1&sn=a20137d6cb1dbb5a4e468b34292cfffc&chksm=f3fd2c9ac48aa58c53223f542f81a760c23c128692a7bd8ad61884dbfb9e3473c618751c7872&scene=21#wechat_redirect)  
[信息收集](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379625&idx=1&sn=16174780c0e0a9280f3340b343d805ec&chksm=f3fd2c1ec48aa5082127ac4c782052fed9ffad98feaf8c4bb6c10c34d753cbd313741b5e4f9e&scene=21#wechat_redirect)[CNVD](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379395&idx=1&sn=137bb0dee822cb4c335aafc56b127866&chksm=f3fd2df4c48aa4e253a8e454cd981911d89c849ed0e2d5673b087114f19192830d06544b9512&scene=21#wechat_redirect)[安全求职](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379240&idx=1&sn=acfca99c355411170fe6a6845e14092e&chksm=f3fd2a9fc48aa3890343a0e4fb9726a00385340bb32b25130560e3d51ccfdf3ae42af5e67fb1&scene=21#wechat_redirect)[渗透实战](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379325&idx=1&sn=c2eba3936b73b8026f8a34f5ac9915a0&chksm=f3fd2d4ac48aa45c6e99b03e89463d125392bac39b2379fe7c4887303304ebf62a3917fbc088&scene=21#wechat_redirect)[CVE](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650378778&idx=1&sn=75840eb5f62a03627bc4f390efdd548e&chksm=f3fd2b6dc48aa27b0c9cb543efbea8bbf0f7565ac3075a053f66095c6197a9bab1b68d2b19eb&scene=21#wechat_redirect)[高薪揭秘](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379541&idx=1&sn=e723ec96fb2e57183b57f2577e2ecc21&chksm=f3fd2c62c48aa574e41dca8b58742ea4d282425eef2deca0cda438e9af08f4156a193f7d283a&scene=21#wechat_redirect)[渗透测试工具](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379790&idx=1&sn=6bc249cb209d094badde28cab8f6ec82&chksm=f3fd2f79c48aa66ff1f62527e0a386d1bf6c1cbe39df8f1e7491abdd27d9ecfc97b96e9aa516&scene=21#wechat_redirect)[网络安全行业](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379779&idx=1&sn=a8910121532c3e5a81a808a2b344910f&chksm=f3fd2f74c48aa662bb5341cf4f7cda2bb2f53871fbf430eef88bf53549dcf95ad91bb4aabb93&scene=21#wechat_redirect)[神秘大礼包](http://mp.weixin.qq.com/s?__biz=MzI4NTE4NDAyNA==&mid=2650379753&idx=1&sn=139d0832145f76e746512e100dab4c24&chksm=f3fd2c9ec48aa588cb48350483ceeb121d23c66db9f07e0641d95d08ac8f9bdc0c279223f9eb&scene=21#wechat_redirect)
**基础教程** 我们贴心备至 **用户答疑**  QQ在线客服 **加入社群** QQ+微信等着你

![](https://gitee.com/fuli009/images/raw/master/public/20211008182938.png)

  

**我就知道你“在看”**![](https://gitee.com/fuli009/images/raw/master/public/20211008182940.png)

预览时标签不可点

收录于话题 #

个 __

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

【安全入门】漏洞发现爬虫特扫&Burp插件自动化&白盒扫描

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

