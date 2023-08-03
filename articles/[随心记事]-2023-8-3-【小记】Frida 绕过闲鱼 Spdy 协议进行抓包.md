#  【小记】Frida 绕过闲鱼 Spdy 协议进行抓包

原创 心态与度量 [ 随心记事 ](javascript:void\(0\);)

**随心记事** ![]()

微信号 Remember-Things

功能介绍 分享、记录、共同进步成长

____

___发表于_

收录于合集

#逆向分析 4 个

#安卓 1 个

**SPDY 介绍 - 来自维基百科**

* * *

  

SPDY（发音如英语：speedy），一种开放的网络传输协议，由Google开发，用来发送网页内容。基于传输控制协议（TCP）的应用层协议。SPDY也就是HTTP/2的前身。Google最早是在Chromium中提出的SPDY协议[1]。被用于Google
Chrome浏览器中来访问Google的SSL加密服务。SPDY并不是首字母缩略字，而仅仅是"speedy"的缩写。SPDY现为Google的商标[2]。HTTP/2的关键功能主要来自SPDY技术，换言之，SPDY的成果被采纳而最终演变为HTTP/2。
****

SPDY并不是一个标准协议，但SPDY的开发组推动SPDY成为正式标准，而成为了互联网草案[3]。后来SPDY未能单独成为正式标准，不过SPDY开发组的成员全程参与了HTTP/2的制定过程。Google
Chrome[4]、Mozilla Firefox、Safari、Opera、Internet
Explorer[5]等主要浏览器均已经或曾经支持SPDY协议。SPDY协议类似于HTTP，但旨在缩短网页的加载时间和提高安全性。SPDY协议通过压缩、多路复用和优先级来缩短加载时间[1]。HTTP/2协议完成之后，Google认为SPDY可以功成身退了[6]，于是最终Google
Chrome淘汰对SPDY的支持，全面改为采用HTTP/2。  

![]()

 **这一段是为了水够300字  ![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Yellowdog.png)**  

 **正常抓包**  

* * *

  

![]()

![]()

  

![]()

  

 **定位相关函数 - Hook 点**

* * *

  

肯定搜索 spdy 相关的啦

![]()

点进去查看函数引用，发现这些有的甚至函数引用为 0。点击加载更多，看到了不一样的包名：  

![]()

![]()

老规矩，看一下函数引用：

![]()

顺藤摸瓜点进来：

![]()

 **编写 Hook  代码  
**

* * *

  

尝试Hook一下。

这个函数返回了布尔值，hook后直接返回 false 就行了。  

![]()

![]()

  

 **再次抓包**

* * *

  


![]()

![]()

可以看到，搞定了

 **文末结语**

* * *

  

水字数  

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

