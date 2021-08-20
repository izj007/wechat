##  渗透测试之劫持国外某云BucketName

Ed1s0n  [ 渗透云笔记 ](javascript:void\(0\);)

**渗透云笔记** ![]()

微信号 shentouyun

功能介绍
分享学习网络安全的路上，把自己所学的分享上来，帮助一些网络小白避免“踩坑”，既节省了学习时间，少走了弯道，我们也可以去回顾自己所学，分享自己所得与经验，为此我们感到光荣。

____

__

收录于话题

**首发先知社区 链接：https://xz.aliyun.com/t/10050**

 **好兄弟写的，求有账号的师傅点赞  
**

 **  
**

 **  **  

## 0\. 起因：

几天前，收到一个国外目标（公司）的渗透测试任务，时间为两周；

大概看了一下目标是类似于国内阿里云那样提供云服务的平台；

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820100037.png)

常规信息收集过后，尝试渗透三天无果... 于是下班前只能祭出我的"大杀器"---缝合怪.py。缝合了一些好用的扫描器，一键 XRAY多线程批量扫 +
自动添加任务到AWVS + 自动添加任务到arl + ...加入资产后就下班回家了。

到了第二天一看扫描结果，心里暗道不妙，md坏起来了啊。。。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820100048.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820100049.png)

扫描器里一个洞都没，goby里所有资产显示只开放两个端口80、443。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820100050.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820100051.png)

不慌，问题不大，时间还长，接下来要做的，就是整理思路，重新来过。

在重新整理之前收集到的资产中，发现测试目标的旁站有一个有趣的404页面：

  

![]()

NoSuchBucket + BucketaName

想到了 阿里云 的bucket劫持漏洞，幸福来得太突然了。

## 1\. 经过：

使用测试账号登录自己的云平台尝试进行劫持：  
1.点击对象存储服务：

![](https://gitee.com/fuli009/images/raw/master/public/20210820100052.png)  
2.点击创建桶：

![](https://gitee.com/fuli009/images/raw/master/public/20210820100054.png)  
3.桶的名字为BucketName字段:

![](https://gitee.com/fuli009/images/raw/master/public/20210820100055.png)  
4.将访问控制权限更改为公共读写:

![](https://gitee.com/fuli009/images/raw/master/public/20210820100056.png)  
5.点击对象，创建hack.txt:

![]()  
6.完成后刷新http://321.asd.com为如下:

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820100057.png)

发现BucketName字段消失了，原来的NoSuchBucket也变成了NoSuchCustomDomain，说明我们的修改对它造成了影响！  
7.NoSuchCustomDomain？那我们就来给他设置一个，点击域名管理尝试绑定域名：

![](https://gitee.com/fuli009/images/raw/master/public/20210820100058.png)  
8.访问http://321.asd.com/

![](https://gitee.com/fuli009/images/raw/master/public/20210820100059.png)  
9.访问：http://321.asd.com/hack.txt （hack.txt为我们刚才上传的）

(后期尝试上传图片，html等文件均可)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820100100.png)

劫持成功！拿来吧你！

  

![]()

## 2\. 结果：

  1. 删除该桶后，http://321.asd.com/恢复bucket字段:

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820100102.png)

  

漏洞危害：劫持Bucket，并开放匿名读取功能。可以挂黑页或引用了js文件，攻击者可以上传恶意js文件，去盗取用户信息。。。

  

  2. 还有几天时间，接着搞搞（ji xu mo yu），渗透测试一定要耐心并且专注~

  

           

  

  

  

  

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

渗透测试之劫持国外某云BucketName

最多200字，当前共字

__

发送中

写下你的留言

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

