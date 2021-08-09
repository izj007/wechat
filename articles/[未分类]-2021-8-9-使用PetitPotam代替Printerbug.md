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

#  使用PetitPotam代替Printerbug

阅读量    **6114** |

分享到： ![QQ空间](https://p0.ssl.qhimg.com/sdm/28_28_100/t014ba42aad7714178d.png)
![新浪微博](https://p0.ssl.qhimg.com/sdm/28_28_100/t01f0d8694dda79069d.png)
![微信](https://p0.ssl.qhimg.com/sdm/28_28_100/t01e29062a5dcd13c10.png)
![QQ](https://p0.ssl.qhimg.com/sdm/28_28_100/t010d95bee6ba3edf60.png)
![facebook](https://p0.ssl.qhimg.com/sdm/28_28_100/t01a5e75c16cffcb0ba.png)
![twitter](https://p0.ssl.qhimg.com/sdm/28_28_100/t01fc30c819f51e9cff.png)

发布时间：2021-08-09 10:00:47

[robots](/35efd2d4bd.html)

> 上帝关了一扇, 必定会再为你打开另一扇窗

## 0x00 前言

Printerbug使得拥有控制域用户/计算机的攻击者可以指定域内的一台服务器，并使其对攻击者选择的目标进行身份验证。虽然不是一个微软承认的漏洞，但是跟Net-
ntlmV1,非约束委派，NTLM_Relay,命名管道模拟这些手法的结合可以用来域内提权，本地提权，跨域等等利用。

遗憾的是，在PrintNightmare爆发之后，很多企业会选择关闭spoolss服务，使得Printerbug失效。在Printerbug逐渐失效的今天，PetitPotam来了，他也可以指定域内的一台服务器，并使其对攻击者选择的目标进行身份验证。而且在低版本(16以下)的情况底下，可以匿名触发。

## 0x01 原理

`MS-EFSR`里面有个函数EfsRpcOpenFileRaw(Opnum 0)

    
    
    long EfsRpcOpenFileRaw(
       [in] handle_t binding_h,
       [out] PEXIMPORT_CONTEXT_HANDLE* hContext,
       [in, string] wchar_t* FileName,
       [in] long Flags
     );
    

他的作用是打开服务器上的加密对象以进行备份或还原，服务器上的加密对象由`FileName` 参数指定,`FileName`的类型是UncPath。

当指定格式为`\\IP\C$`的时候，lsass.exe服务就会去访问`\\IP\pipe\srvsrv`

指定域内的一台服务器，并使其对攻击者选择的目标(通过修改FileName里面的IP参数)进行身份验证。

## 0x02 细节

###  1、通过lsarpc 触发

在[官方文档](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-
efsr/403c7ae0-1a3a-4e96-8efc-54e79a2cc451)里面，`MS-
EFSR`的调用有`\pipe\lsarpc`和`\pipe\efsrpc`两种方法，其中

  * `\pipe\lsarpc`的服务器接口必须是UUID [c681d488-d850-11d0-8c52-00c04fd90f7e]
  * `\pipe\efsrpc`的服务器接口必须是UUID [df1941c5-fe89-4e79-bf10-463657acf44d]

在我本地测试发现`\pipe\efsrpc`并未对外开放

在PetitPotam的Poc里面有一句注释`possible aussi via efsrpc (en changeant d'UUID) mais ce
named pipe est moins universel et plus rare que lsarpc ;)`，翻译过来就是

`也可以通过EFSRPC（通过更改UUID），但这种命名管道的通用性不如lsarpc，而且比LSARPC更罕见`

所以PetitPotam直接是采用lsarpc的方式触发。

###  2、低版本可以匿名触发

在08和12的环境，默认在`网络安全:可匿名访问的命名管道`中有三个`netlogon`、`samr`、`lsarpc`。因此在这个环境下是可以匿名触发的

遗憾的是在16以上这个默认就是空了，需要至少一个域内凭据。

## 0x03 利用

这篇文章的主题是使用`PetitPotam`代替`Printerbug`，因此这个利用同时也是`Printerbug`的利用。这里顺便梳理复习下`Printerbug`的利用。

###  1、结合 CVE-2019-1040，NTLM_Relay到LDAP

详情见[CVE-2019-1040](https://daiker.gitbook.io/windows-protocol/ntlm-
pian/7#5-cve-2019-1040),这里我们可以将触发源从`Printerbug`换成`PetitPotam`

###  2、Relay到HTTP

不同于LDAP是协商签名的，发起的协议如果是smb就需要修改Flag位，到HTTP的NTLM认证是不签名的。前段时间比较火的ADCS刚好是http接口，又接受ntlm认证，我们可以利用PetitPotam把域控机器用户relay到ADCS里面申请一个域控证书，再用这个证书进行kerberos认证。注意这里如果是域控要指定模板为`DomainController`

    
    
    python3 ntlmrelayx.py -t https://192.168.12.201/Certsrv/certfnsh.asp -smb2support --adcs --template "DomainController"
    

###  2、结合非约束委派的利用

当一台机器机配置了非约束委派之后，任何用户通过网络认证访问这台主机，配置的非约束委派的机器都能拿到这个用户的TGT票据。

当我们拿到了一台非约束委派的机器，只要诱导别人来访问这台机器就可以拿到那个用户的TGT，在这之前我们一般用printerbug来触发，在这里我们可以用PetitPotamlai来触发。

域内默认所有域控都是非约束委派，因此这种利用还可用于跨域。

###  3、结合Net-ntlmV1进行利用

很多企业由于历史原因，会导致LAN身份验证级别配置不当，攻击者可以将Net-Ntlm降级为V1

我们在Responder里面把Challeng设置为`1122334455667788`,就可以将Net-ntlm V1解密为ntlm hash

###  4、结合命名管道的模拟

在这之前，我们利用了printerbug放出了pipePotato漏洞。详情见[pipePotato：一种新型的通用提权漏洞](https://www.anquanke.com/post/id/204510)。

在PetitPotam出来的时候，发现这个RPC也会有之前pipePotato的问题。

## 0x04 引用

  * [[MS-EFSR]: Encrypting File System Remote (EFSRPC) Protocol](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-efsr/08796ba8-01c8-4872-9221-1000ec2eff31)–[PetitPotam](https://github.com/topotam/PetitPotam)

本文由 **御守实验室** 原创发布  
转载，请参考[转载声明](/note/repost)，注明出处：
[https://www.anquanke.com/post/id/249603](/post/id/249603)  
安全客 - 有思想的安全新媒体

[Windows](/tag/Windows) [PetitPotam](/tag/PetitPotam) __ 赞 __收藏

[ 御守实验室 ](/member/159343)

分享到： ![QQ空间](https://p0.ssl.qhimg.com/sdm/28_28_100/t014ba42aad7714178d.png)
![新浪微博](https://p0.ssl.qhimg.com/sdm/28_28_100/t01f0d8694dda79069d.png)
![微信](https://p0.ssl.qhimg.com/sdm/28_28_100/t01e29062a5dcd13c10.png)
![QQ](https://p0.ssl.qhimg.com/sdm/28_28_100/t010d95bee6ba3edf60.png)
![facebook](https://p0.ssl.qhimg.com/sdm/28_28_100/t01a5e75c16cffcb0ba.png)
![twitter](https://p0.ssl.qhimg.com/sdm/28_28_100/t01fc30c819f51e9cff.png)

|推荐阅读

[ ![](https://p5.ssl.qhimg.com/sdm/229_160_100/t01e9ef10b21fb4cc07.png)

###### 分享一个最近的一次应急溯源

2021-08-09 10:30:25 ](/post/id/248890)

[ ![](https://p2.ssl.qhimg.com/sdm/229_160_100/t010500b2188e8a5018.jpg)

###### 使用PetitPotam代替Printerbug

2021-08-09 10:00:47 ](/post/id/249603)

[ ![](https://p3.ssl.qhimg.com/sdm/229_160_100/t0131c91d129ca0ed1b.png)

###### 加密固件之依据老固件进行解密

2021-08-08 12:00:10 ](/post/id/248741)

[ ![](https://p4.ssl.qhimg.com/sdm/229_160_100/t016cc852e99271d391.png)

###### JSBot无文件攻击分析

2021-08-08 10:00:56 ](/post/id/249104)

|发表评论

[ ](/member/159343)

发表评论

|评论列表

还没有评论呢，快去抢个沙发吧~

加载更多

[ ](/member/159343)

[ 御守实验室 ](/member/159343)

中安网星御守实验室（Amulab）是隶属于中安网星的纯技术研究团队。主要职能包括横向移动研究、AD域安全研究、Windows安全研究等前瞻攻防技术预研、工具孵化，为产品输出安全能力。

文章

3

粉丝

14

__ 关注

## TA的文章

[使用PetitPotam代替Printerbug](/post/id/249603)

2021-08-09 10:00:47

[Active Directory 证书服务攻击与防御（一）](/post/id/245791)

2021-07-05 10:00:33

[利用MS-SAMR协议修改/重置用户密码](/post/id/245482)

2021-06-29 14:30:22

__

### 相关文章

  * [ 深入理解APC机制](/post/id/247813)
  * [ Windows Kernel Exploitation Notes（二）——HEVD Write-What-Where](/post/id/246289)
  * [ CVE-2021-26900 Windows Win32k提权漏洞分析](/post/id/245481)
  * [ Active Directory 证书服务攻击与防御（一）](/post/id/245791)
  * [ Windows Kernel Exploitation Notes（一）——HEVD Stack Overflow](/post/id/245528)
  * [ Win10下一个有意思的驱动引起可能性的拒绝服务攻击](/post/id/237096)
  * [ 揪出那些在Windows操作系统中注册的WFP函数](/post/id/236134)

热门推荐

[ ](/post/id/162175)

##### 文章目录

0x00 前言 0x01 原理 0x02 细节 1、通过lsarpc 触发 2、低版本可以匿名触发 0x03 利用 1、结合
CVE-2019-1040，NTLM_Relay到LDAP 2、Relay到HTTP 2、结合非约束委派的利用 3、结合Net-ntlmV1进行利用
4、结合命名管道的模拟 0x04 引用

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

