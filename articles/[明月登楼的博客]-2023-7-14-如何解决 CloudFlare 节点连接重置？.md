#  如何解决 CloudFlare 节点连接重置？

原创 明月登楼 [ 明月登楼的博客 ](javascript:void\(0\);)

**明月登楼的博客** ![]()

微信号 imydl-blog

功能介绍 草根博客作者明月登楼吐槽网站运营、服务器运维、独立博客成长等等各种经历心得的平台。

____

___发表于_

收录于合集

#CloudFlare 16 个

#DDoS攻击 17 个

#CC攻击 16 个

#cdn 4 个

  

明月登楼

读完需要

1分钟

速读仅需 1 分钟

最近明月再给几个客户做 CloudFlare 部署的时候经常碰到 CloudFlare 节点出现连接重置或者连接超时的问题，初步分析估计是部分
CloudFlare 节点 IP 有被封禁的嫌疑。

![](https://gitee.com/fuli009/images/raw/master/public/20230714180235.png)

就是这个"ERR_CONNECTION_RESET"（连接重置）错误

但是，CloudFlare 的节点被封禁并不多见，据明月的经验很多时候其实是国内各地 ISP
解析缓存造成的解析错误。了解国内网络状况的应该都知道目前国内被三大运营商占据着主导地位，各地区的三大 ISP 为了降低运营成本，多会缓存 DNS
解析，这也是这几年宽带越来越便宜的一个重要原因之一，所以国内宽带用的时间久了明月是建议大家清理一下电脑端的 DNS
缓存的，具体可参考【亲，你有多久没有清理过你电脑的 DNS 缓存了？】一文。

这次明月客户碰到的使用 CloudFlare 后出现“连接重置”故障，就是在让客户清理了电脑的 DNS
缓存后解决的，并且速度上还提升了不少，如果您也碰到这类问题，建议可以试一下哦！

![](https://gitee.com/fuli009/images/raw/master/public/20230714180236.png)

 ****

 **·END·** ****

 **草根博客站长交流学习群**

独立博客站长间交流、学习、互动社群

![](https://gitee.com/fuli009/images/raw/master/public/20230714180237.png)

明月登楼的博客 **『www.imydl.com』**

![](https://gitee.com/fuli009/images/raw/master/public/20230714180238.png)

明月运维服务小栈

  

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

