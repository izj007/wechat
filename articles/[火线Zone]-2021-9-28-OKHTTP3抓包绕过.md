#  OKHTTP3抓包绕过

火线小助手  [ 火线Zone ](javascript:void\(0\);)

**火线Zone** ![]()

微信号 huoxian_zone

功能介绍 火线Zone是[火线安全平台]运营的封闭式实战安全攻防社区。

____

__

收录于话题

**前言**

  

同样是之前做的那个项目，除了加壳还用了OKhttp3，导致burp等工具无法抓到该应用的通信请求，抓瞎了一段时间，好在已经解决该问题，不过如果APP做了虚拟机识别的话，就没办法使用这种方式了。

  

 **0x01**

  

1.模拟器安装BURP证书

  

将burp证书导入模拟器中，在安全设置处将证书安装

  

![](https://gitee.com/fuli009/images/raw/master/public/20210928085226.png)

  

2.下载安装 proxifier

  

  * 

    
    
    http://www.proxifier.com/

激活码自行探索

  

3.添加代理服务器为burp监听的地址

  

![](https://gitee.com/fuli009/images/raw/master/public/20210928085227.png)

  

4.设置代理规则

  

我这里使用的是夜神模拟器，夜神的通信进程为 noxvmhandle.exe ,我们直接将这个进程添加到代理规则中就好了。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210928085228.png)

  

 **0x02**

  

现在就可以愉快的抓包了

  

![](https://gitee.com/fuli009/images/raw/master/public/20210928085229.png)

  

由于proxifier采用的是类似于注入模拟器进程转发流量的方式，就能让一些APP的防抓包机制失效，当APP没有限制模拟器使用时，这种方式比起HOOK还是会简单便捷很多。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210928085230.png)

  

【火线短视频精选】

【周度激励】2021.9.13 ～ 2021.9.19公告

  

![](https://gitee.com/fuli009/images/raw/master/public/20210928085231.png)

  

  

【相关精选文章】

  

[protobuf协议逆向解析](http://mp.weixin.qq.com/s?__biz=MzI2NDQ5NTQzOQ==&mid=2247488756&idx=1&sn=b3f599a20f25ea5648bfb55f7a91f149&chksm=eaaa9cd4dddd15c2de56a0e411e4d39194f364038dcc29b59edc31fbb476017af58f017184e0&scene=21#wechat_redirect)  

  

[Frida+FRIDA-DEXDump
实现简单脱壳](http://mp.weixin.qq.com/s?__biz=MzI2NDQ5NTQzOQ==&mid=2247488799&idx=1&sn=b609893027e6e2f431ee490e9629f446&chksm=eaaa9d3fdddd1429b9e25b40671128ddce347cd38ca73da17e8c804ac40bd94b6f37b08ccd7c&scene=21#wechat_redirect)  

  

【招人专区】

  

![](https://gitee.com/fuli009/images/raw/master/public/20210928085232.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210928085233.png)

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

OKHTTP3抓包绕过

最多200字，当前共字

__

发送中

写下你的留言

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

