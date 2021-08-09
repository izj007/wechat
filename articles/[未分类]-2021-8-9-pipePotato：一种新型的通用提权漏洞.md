[robots](/35efd2d4bd.html)

[ ![](/img/job/logo.svg) ](https://www.anquanke.com)

  * [首页](/)
  * 文章 

|文章分类

[安全知识](/knowledge)| [安全资讯](/news)| [安全活动](/activity)| [安全工具](/tool)|  
[招聘信息](/job)

|内容精选

[360网络安全周报](/week-list)| [安全客季刊](/discovery) |  
[专题列表](/subject-list)

|热门标签

[ 活动 ](/tag/活动)| [ 安全活动 ](/tag/安全活动)| [ CTF ](/tag/CTF)| [ 恶意软件 ](/tag/恶意软件)|
[ 每日安全热点 ](/tag/每日安全热点)| [ 网络安全热点 ](/tag/网络安全热点)| [ Web安全 ](/tag/Web安全)| [
漏洞预警 ](/tag/漏洞预警)| [ 渗透测试 ](/tag/渗透测试)| [ Pwn ](/tag/Pwn)|

  * [漏洞](/vul)
  * [SRC导航](/src)
  * ![](/img/job/new.svg) [招聘](/job-list)
  * [内容精选](/discovery)

![](https://p0.ssl.qhimg.com/t010133a1346bd31419.png)

![](https://p0.ssl.qhimg.com/t01103704213901dd1e.png) [
![](https://p0.ssl.qhimg.com/t0161d2c7f7fe89cb91.png) ](/app)

![投稿](https://p0.ssl.qhimg.com/t01e18bc83d1362b57e.png)投稿

[ 登录 ](/login) [ 注册 ](/register)

  * 主页2 个人主页
  * [ 消息 我的消息](/setting?p=message)

  * [ 设置 个人设置](/setting)
  * [ 关闭 退出登录](javascript:void\(0\))

__

  * 首页
  * 安全知识
  * 安全资讯
  * 招聘信息
  * 安全活动
  * APP下载

#  pipePotato：一种新型的通用提权漏洞

阅读量    **776612** | 评论 **12 **

分享到： ![QQ空间](https://p0.ssl.qhimg.com/sdm/28_28_100/t014ba42aad7714178d.png)
![新浪微博](https://p0.ssl.qhimg.com/sdm/28_28_100/t01f0d8694dda79069d.png)
![微信](https://p0.ssl.qhimg.com/sdm/28_28_100/t01e29062a5dcd13c10.png)
![QQ](https://p0.ssl.qhimg.com/sdm/28_28_100/t010d95bee6ba3edf60.png)
![facebook](https://p0.ssl.qhimg.com/sdm/28_28_100/t01a5e75c16cffcb0ba.png)
![twitter](https://p0.ssl.qhimg.com/sdm/28_28_100/t01fc30c819f51e9cff.png)

发布时间：2020-05-06 13:50:32

[robots](/35efd2d4bd.html)

作者：xianyu & daiker

## 0x00 影响

本地提权，对于任意windows Server 2012以上的windows
server版本(win8以上的某些windows版本也行)，从Service用户提到System 用户，在windows Server
2012，windows Server 2016，windows Server 2019全补丁的情况都测试成功了。

## 0x01 攻击流程

演示基于server 2019

  * laji.exe. msf 生成的正常木马
  * pipserver.exe 命名管道服务端，注册命名管道
  * spoolssClient.exe 打印机rpc调用客户端

首先，攻击者拥有一个服务用户，这里演示采用的是IIS服务的用户。攻击者通过pipeserver.exe注册一个名为pipexpipespoolss的恶意的命名管道等待高权限用户来连接以模拟高权限用户权限，然后通过spoolssClient.exe迫使system用户来访问攻击者构建的恶意命名管道，从而模拟system用户运行任意应用程序

## 0x02 漏洞成因

spoolsv.exe 进程会注册一个 rpc 服务,任何授权用户可以访问他，RPC 服务里面存在一个函数
RpcRemoteFindFirstPrinterChangeNotificationEx

<https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-
rprn/eb66b221-1c1f-4249-b8bc-c5befec2314d>

pszLocalMachine 可以是一个 UNC 路径(\host)，然后 system 用户会访问 \hostpipespoolss,

在文档 [https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-
rprn/9b3f8135-7022-4b72-accb-aefcc360c83b里面](https://docs.microsoft.com/en-
us/openspecs/windows_protocols/ms-rprn/9b3f8135-7022-4b72-accb-
aefcc360c83b%E9%87%8C%E9%9D%A2)

server name 是这样规范的

    
    
    SERVER_NAME = "\" host "" | ""
    

如果 SERVER_NAME 是 \127.0.0.1 ,system用户会访问 \127.0.0.1pipespoolss

问题是，如果 SERVER_NAME
是\127.0.0.1/pipe/xx,system用户会访问\127.0.0.1pipexxxpipespoolss，这个命名管道并没有注册，攻击者就可以注册这个命名管道。

当 system 用户访问这个命名管道(pipexpipespoolss)，我们就能模拟system 用户开启一个新的进程。

## 0x03 时间线

2019年12月5日 向MSRC进行反馈，分配编号`VULN-013177` `CRM:0279000283```

2019年12月5日 分配Case编号 `MSRC Case 55249`

2019年12月15日 向MSRC发邮件询求进度，微软2019年12月18日回复

2019年12月27日 MSRC 回信认为impersonate的权限需要administrator或者等同用户才拥有，Administrator-to-
kernel并不是安全问题。事实上，所有的service 用户(包括local service，network service
都具备这个权限)。我们向MSRC发邮件反馈此事

    
    
    the account used to impersonate the named pipe client use the SeImpersonatePrivilege. The SeImpersonatePrivilege is only available to accounts that have administrator or equivalent privileges. Per our servicing criteria: Administrator-to-kernel is not a security boundary.
    

2019年12月28日 MSRC 回信会处理，至今没有回信

2020年5月6日 在安全客上披露

本文由 **360灵腾安全实验室** 原创发布  
转载，请参考[转载声明](/note/repost)，注明出处：
[https://www.anquanke.com/post/id/204510](/post/id/204510)  
安全客 - 有思想的安全新媒体

[漏洞分析](/tag/漏洞分析) __ 赞 ( 25) __收藏

[ 360灵腾安全实验室 ](/member/143805)

分享到： ![QQ空间](https://p0.ssl.qhimg.com/sdm/28_28_100/t014ba42aad7714178d.png)
![新浪微博](https://p0.ssl.qhimg.com/sdm/28_28_100/t01f0d8694dda79069d.png)
![微信](https://p0.ssl.qhimg.com/sdm/28_28_100/t01e29062a5dcd13c10.png)
![QQ](https://p0.ssl.qhimg.com/sdm/28_28_100/t010d95bee6ba3edf60.png)
![facebook](https://p0.ssl.qhimg.com/sdm/28_28_100/t01a5e75c16cffcb0ba.png)
![twitter](https://p0.ssl.qhimg.com/sdm/28_28_100/t01fc30c819f51e9cff.png)

|推荐阅读

[ ![](https://p4.ssl.qhimg.com/sdm/229_160_100/t0156ce4ffa5fd81122.png)

###### 从零开始的内核eBPF之旅（1）

2021-08-09 12:00:53 ](/post/id/249211)

[ ![](https://p2.ssl.qhimg.com/sdm/229_160_100/t0110d6ccbdc9cef92e.jpg)

###### 密码学学习笔记 之 差分分析入门篇——四轮DES

2021-08-09 10:30:46 ](/post/id/249403)

[ ![](https://p5.ssl.qhimg.com/sdm/229_160_100/t01e9ef10b21fb4cc07.png)

###### 分享一个最近的一次应急溯源

2021-08-09 10:30:25 ](/post/id/248890)

[ ![](https://p2.ssl.qhimg.com/sdm/229_160_100/t010500b2188e8a5018.jpg)

###### 使用PetitPotam代替Printerbug

2021-08-09 10:00:47 ](/post/id/249603)

|发表评论

[ ](/member/143805)

发表评论

|评论列表

还没有评论呢，快去抢个沙发吧~

加载更多

[ ](/member/143805)

[ 360灵腾安全实验室 ](/member/143805)

360灵腾安全实验室（RedTeam &
0KeeTeam）隶属于360政企集团实网威胁感知部。主要职能包括红队技术、安全狩猎等前瞻攻防技术预研、工具孵化，为各核心产品输出安全能力。

文章

38

粉丝

148

__ 关注

## TA的文章

[渗透测试中的Exchange](/post/id/226543)

2020-12-24 14:30:09

[ZeroLogon的利用以及分析](/post/id/219374)

2020-10-13 16:15:30

[邮件伪造组合拳](/post/id/218889)

2020-09-29 19:46:41

[NTLM认证协议与SSP（下）——NTLM中高级进阶](/post/id/210324)

2020-07-13 10:00:07

[Citrix 从权限绕过到远程代码执行分析（CVE-2020-8193）](/post/id/210407)

2020-07-13 09:45:07

__

### 相关文章

  * [ Java安全之 Xstream 漏洞分析](/post/id/248029)
  * [ CVE-2020-26567 DSR-250N 远程拒绝服务漏洞分析](/post/id/247049)
  * [ 国外漏洞报告大赏-p站任意密码重置](/post/id/248769)
  * [ CVE-2021-33514：Netgear 多款交换机命令注入漏洞](/post/id/248135)
  * [ Chrome 在野0day：CVE-2021-30551的分析与利用](/post/id/248716)
  * [ 记一次从鸡肋SSRF到RCE的代码审计过程](/post/id/248821)
  * [ Windows内核提权漏洞CVE-2018-8120分析 - 下](/post/id/247764)

热门推荐

[ ](/post/id/162175)

##### 文章目录

0x00 影响 0x01 攻击流程 0x02 漏洞成因 0x03 时间线

![安全客Logo](https://p0.ssl.qhimg.com/t0168809c9f19b4fec6.png)

[ ![安全客](https://p0.ssl.qhimg.com/t014afa383e7a786b4a.png)
](https://zhuanlan.zhihu.com/c_118578260)

[ ](https://weibo.com/360adlab)

##### 微信二维码

×

![安全客](https://p0.ssl.qhimg.com/t0151209205b47f2270.jpg)

## 安全客

  * [关于我们](/about)
  * [加入我们](/join)
  * [联系我们](/note/contact)
  * [用户协议](/note/protocol)

## 商务合作

  * [合作内容](/note/business)
  * [联系方式](/note/contact)
  * [友情链接](/link)

## 内容须知

  * [投稿须知](https://www.anquanke.com/contribute/tips)
  * [转载须知](/note/repost)
  * 官网QQ群6：785695539 
  * 官网QQ群3：830462644(已满) 
  * 官网QQ群2：814450983(已满) 
  * 官网QQ群1：702511263(已满) 

## 合作单位

  * [ ![安全客](https://p0.ssl.qhimg.com/t01592a959354157bc0.png) ](http://www.cert.org.cn/)
  * [ ![安全客](https://p0.ssl.qhimg.com/t014f76fcea94035e47.png) ](http://www.cnnvd.org.cn/)

Copyright © 北京奇虎科技有限公司 360网络攻防实验室 安全客 All Rights Reserved
[京ICP备08010314号-66](https://beian.miit.gov.cn/)

![](https://p0.ssl.qhimg.com/t0179ac3294ef926b8c.png)

