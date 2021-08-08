##  使用nssm注册windows服务进行持久化

Leticia's Blog  [ 乌雲安全 ](javascript:void\(0\);)

**乌雲安全** ![]()

微信号 hackctf

功能介绍
乌雲安全，致力于红队攻防实战、内网渗透、代码审计、社工、安卓逆向、CTF比赛技巧、安全运维等技术干货分享，并预警最新漏洞，定期分享常用渗透工具、教程等资源。

____

__

收录于话题

#渗透技巧 14

#渗透测试工具 7

#红蓝对抗 7

#web安全 14

# 前言

NSSM是一款注册Windows系统服务的工具。将应用程序注册为windows系统服务可以使其在系统停止、重启等情况仍可自动运行，可用于持久化后门。

# 下载

http://www.nssm.cc/download

# 使用

解压并在nssm.exe目录打开cmd

使用如下命令注册服务

    
    
    nssm install 服务名  
    

填写木马路径和服务名，可以填写欺骗性较强的服务名

![](https://gitee.com/fuli009/images/raw/master/public/20210808102544.png)

在服务里设置自动启动

![](https://gitee.com/fuli009/images/raw/master/public/20210808102545.png)

重启对方机器，在msf中开启监听，成功得到shell

![](https://gitee.com/fuli009/images/raw/master/public/20210808102546.png)

作者：Leticia's Blog ，详情点击阅读原文。

 **推荐阅读**[
**![]()**](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247496904&idx=1&sn=e6c717bc2709f7c4ec8523bc681f43f3&chksm=9acd2457adbaad4169ed38ebf0d969553b6cf4dee26307f2ce6a137c52849e4ffdf06325347e&scene=21#wechat_redirect)  
 **觉得不错点个 **“赞”** 、“在看”，支持下小编**
**![](https://gitee.com/fuli009/images/raw/master/public/20210808102547.png)**

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

使用nssm注册windows服务进行持久化

最多200字，当前共字

__

发送中

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

