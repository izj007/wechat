#  一个ZoomEye查询搜尽BazarLoader C2

heige  [ Seebug漏洞平台 ](javascript:void\(0\);)

**Seebug漏洞平台** ![]()

微信号 seebug_org

功能介绍 Seebug，原 Sebug 漏洞平台，洞悉漏洞，让你掌握第一手漏洞情报！

____

__

收录于话题

![](https://gitee.com/fuli009/images/raw/master/public/20210910183734.png)

  

## 作者：heige  
公众号：黑哥说安全

  

  

在《[趣访“漏洞之王”黑哥，探寻知道创宇十年网络资产测绘之路！](https://mp.weixin.qq.com/s?__biz=MjM5NzA3Nzg2MA==&mid=2649852342&idx=1&sn=a3b0d73fdb54e53d8bebcda95dd27b49&scene=21#wechat_redirect)》的采访中我提到了知道创宇在网络空间测绘领域的多个优势，其中：

  

>
> 第六，知道创宇提出了很多先进的测绘理念并实践，如黑哥提出的“动态测绘”，时刻关注数据的动态变化及趋势。前面提到的心脏流血漏洞，就是利用动态测绘的理念分析得出的各种有趣的结论。再比如2019年委内瑞拉大停电事件，
> ZoomEye是全球唯一及时响应、通过动态测绘理念完成该国的网络关键基础设施及重要信息系统测绘的工具。此外还有 “交叉测绘”
> 的思想，基于它可以完成IPv4 vs IPv6 、暗网 vs IP/域名的交叉比；通过“行为测绘”理念来实现重要基础设施的识别定位等等。
>
>
> 高端访谈，公众号：知道创宇[趣访“漏洞之王”黑哥，探寻知道创宇十年网络资产测绘之路！](https://mp.weixin.qq.com/s/dvFAVH17AnDS_r0kOG620A?scene=21)

  

  

提到了由我们提出的多个测绘理念：“动态测绘”、“交叉测绘”、“行为测绘”，在之前我有详细专题文章进行阐述：

  

  

再谈“动态测绘”

https://zhuanlan.zhihu.com/p/183952077

  

  

聊聊网络空间“交叉”测绘溯源

https://mp.weixin.qq.com/s/QTyfHbcnoMYoVUXhcbCYCw

  

  

谈谈网络空间“行为测绘”

https://mp.weixin.qq.com/s/fQatA5iyewqRBMWtpVjsRA

  

  

其中关于“行为测绘”的文章昨天才正式对外发布，同时也写了下英文文档给老外们科普科普，我担心那蹩脚的英文影响老外的理解，所以想着今天补充个例子说明，所以顺带也给国内的朋友们分享一下！

  

  

所以今天的主题就是：How to hunt more BazarLoader C2s through a ZoomEye Query?

  

  

我留意这个BazarLoader还是在@TheDFIRReport在推特9月1日发布的一个威胁情报：

  

  

  * 

    
    
    https://twitter.com/TheDFIRReport/status/1433055791964049412 #BazarLoader64.227.73.8064.225.71.198
    
    br

  

  

按照我一贯的作风，遇到IP或者新设备都忍不住往ZoomEye里一搜：

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210910183743.png)

  

  

从搜索的banner来看两个IP的443端口有着非常相似特征，唯一区别跟追踪Trickbot遇到过的一样，也就是有一个有"Connection:
close"，一个没有，考虑到特征比较明显，所以我这里先选了这个有"Connection: close"的 64.227.73.80:443

  

  

经过几次的搜索尝试，得到一个比较准确的指纹：

  

  

  * 

    
    
    "HTTP/1.1 404 Not found" +"Server: nginx" +"Content-Type: text/html; charset=UTF-8" +"Connection: close Date" -"Content-Length" -"<head>" -"Cache-Control"
    
    br

  

![](https://gitee.com/fuli009/images/raw/master/public/20210910183746.png)

  

  

因为这个只是初步分析，意在获得更多更明显的特征，肯定存在误报的情况的，这里查询出157条数据，主要集中在443端口且都是https协议（顺带书哟下这个也跟样本IP
64.227.73.80:443是一致的），所以我们继续提取了证书里特征比较明显的"issuer" 进行特征观察：

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210910183749.png)

  

  

很明显可以看出来"issuer"的证书大多数表现出来分词结构相似的情况，因为数据量不多，我肉眼手工提取了这些证书里的特征做了个特征集：

  

  

  * 

    
    
    (ssl:"System,CN" ssl:"Amadey Org,CN" ssl:"O=Global Security,OU=IT Department,CN=example.com" ssl:"NZT,CN" ssl:"O=Lero,OU=Lero" ssl:"Security,OU=Krot" ssl:"O=Shioban,OU=Shioban")
    
    br

  

  

在配合前面掌握到的https服务返回的banner特征：

  

  

  * 

    
    
     +"HTTP/1.1  404 Not found" +"Server: nginx" +"Content-Type: text/html; charset=UTF-8"
    
    br

  

  

当然结合这两个条件还存在一些误报，所以进行一些排除运算：

  

  

  * 

    
    
     -ssl:"OU=System" -ssl:digicert -"Content-Length" -"Connection: keep-alive"
    
    br

  

  

 所以最后得到的查询语句为：

  

  

  * 

    
    
     (ssl:"System,CN" ssl:"Amadey Org,CN" ssl:"O=Global Security,OU=IT Department,CN=example.com" ssl:"NZT,CN" ssl:"O=Lero,OU=Lero" ssl:"Security,OU=Krot" ssl:"O=Shioban,OU=Shioban") +"HTTP/1.1  404 Not found" +"Server: nginx" +"Content-Type: text/html; charset=UTF-8" -ssl:"OU=System" -ssl:digicert -"Content-Length" -"Connection: keep-alive"
    
    br

  *   * 

    
    
    https://www.zoomeye.org/searchResult?q=(ssl%3A%22System%2CCN%22%20ssl%3A%22Amadey%20Org%2CCN%22%20ssl%3A%22O%3DGlobal%20Security%2COU%3DIT%20Department%2CCN%3Dexample.com%22%20ssl%3A%22NZT%2CCN%22%20ssl%3A%22O%3DLero%2COU%3DLero%22%20ssl%3A%22Security%2COU%3DKrot%22%20ssl%3A%22O%3DShioban%2COU%3DShioban%22)%20%2B%22HTTP%2F1.1%20%20404%20Not%20found%22%20%2B%22Server%3A%20nginx%22%20%2B%22Content-Type%3A%20text%2Fhtml%3B%20charset%3DUTF-8%22%20-ssl%3A%22OU%3DSystem%22%20-ssl%3Adigicert%20-%22Content-Length%22%20-%22Connection%3A%20keep-alive%22br

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210910183751.png)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210910183754.png)

  

  

 一共得到254条数据，到这里实际上我们就已经实现了“一个ZoomEye查询打尽BazarLoader C2”的目标，当然少了不了数据分析的部分：

  

  

 BazarLoader C2 国家 Top 10

  

  

  * 

    
    
     美国 112 荷兰 53 德国 22 英国 15 罗马尼亚 11 捷克 8 拉脱维亚 8 摩尔多瓦 4 俄罗斯 4 法国  3
    
    br

  

  

BazarLoader C2 运营商 Top 10

  

  

  * 

    
    
    amazon.com 79digitalocean.com 68Unknown 14hosting.international 7itldc.com 7colocrossing.com 5ovh.com 5smarthost.net 5dedipath.com 4eonix.net 4
    
    br

  

  

从这些结果来看，如果你经常对僵尸网络恶意IP地址进行追踪我想这些国家你可能并不陌生，当然我们对这些数据的证书里的域名及JARM进行提取：

  

  

证书中的域名统计

  

  * 

    
    
    amadeamadey.at   46asdotaera.it      7baget.fr          1bigter.ch         3confarencastyas.it  3enjobero.ch       1example.com      33forenzik.kz      64gosterta.fr       2haner.it          3hangober.uk       5holdasdg.it       1holdertoysar.uk   4jerbek.fr         2jermegib.fr       3jersjersy.com     2kajekin.je        6komanchi.com      1ksorun.it         2laralabana.it     3maloregerto.it    6mataner.at        4monblan.ua       14munichresed.de    1nortenarasta.fr   1nztportu.pg       2ofgasrty.fr       2parismaote.fr     1perdefue.fr       7pnercon.tr        1pokilorte.es      7rosteranar.uk     1selfoder.gb       6smartoyab.it      1smartoyta.uk      1smartoytaas.it    4zalustipar.uk     3
    
    br

  

  

JARM 统计（有一些目标没有获取到JARM） :

  

  

  * 

    
    
    2ad2ad16d2ad2ad22c2ad2ad2ad2ad7329fbe92d446436f2394e041278b8b2  92ad2ad16d2ad2ad22c2ad2ad2ad2ad47321614530b94a96fa03d06e666d6d6  322ad2ad0002ad2ad22c2ad2ad2ad2adce7a321e4956e8298ba917e9f2c22849  392ad2ad0002ad2ad0002ad2ad2ad2ade1a3c0d7ca6ad8388057924be83dfc6a  25
    
    br

  

  

这里顺带提一下单一的JARM表现不是很准，误报比较多，但是它还是有一定的统计学上，数据排除等上面有一定的意义的。

  

  

详细数据见：https://pastebin.com/Y9T4KKYr

  

  

本文只是个具体例子进行说明，理论相关详见：

  

  

谈谈网络空间“行为测绘”

[https://mp.weixin.qq.com/s/fQatA5iyewqRBMWtpVjsRA](https://mp.weixin.qq.com/s?__biz=Mzg5OTU1NTEwMg==&mid=2247483713&idx=1&sn=a0fc7c244fe400cc1903de02d70f4bb7&scene=21#wechat_redirect)

  

  

题外话：

  

我觉得我搞的这些套路，真真的好用，真的！而且我也没秃顶，也没有不务正业，也还在搞技术，真的！真的！真的！

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210910183756.png)

 **往  期 热 门**

(点击图片跳转)

[![](https://gitee.com/fuli009/images/raw/master/public/20210910183758.png)

Golang的字符编码与regexp

](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650949112&idx=1&sn=d3bde1d84014cc03a34eb3a740ba5484&chksm=807907cab70e8edc53e78eff30e18c96d4d799108b3e5bc4e5f7fce32565550def7ed44630ab&scene=21#wechat_redirect)  
[![](https://gitee.com/fuli009/images/raw/master/public/20210910183759.png)

DEFCON 29 FINAL shooow-your-shell总结

](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650948525&idx=1&sn=3bf60192d8363b88eb51489900be43ee&chksm=8079019fb70e8889a953e63ec90cdd4bd1bda38b140ba1980f18e34b87169f9c9e8aeb64bdde&scene=21#wechat_redirect)  
[![](https://gitee.com/fuli009/images/raw/master/public/20210910183801.png)

红队实战攻防技术（一）

](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650948377&idx=1&sn=488ed6b6388f57b067404baa38fdf28b&chksm=8079012bb70e883dacf4f70674061f44d067717b9bb4765d564d5bb77bda031fed42329b0ce5&scene=21#wechat_redirect)  
![](https://gitee.com/fuli009/images/raw/master/public/20210910183803.png)  
 **觉得不错点个“在看”哦**
**![](https://gitee.com/fuli009/images/raw/master/public/20210910183805.png)**

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

一个ZoomEye查询搜尽BazarLoader C2

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

