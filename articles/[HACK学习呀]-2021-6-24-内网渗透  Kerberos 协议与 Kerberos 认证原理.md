##  内网渗透 | Kerberos 协议与 Kerberos 认证原理

原创 WHOAMI  [ HACK学习呀 ](javascript:void\(0\);)

**HACK学习呀** ![]()

微信号 Hacker1961X

功能介绍
HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

____

__

收录于话题

#内网渗透 15

#基础知识 4

#Kerberos协议 4

![](https://gitee.com/fuli009/images/raw/master/public/20210804120610.png)

## 前言

如果你了解内网渗透，那么应该都对 IPC、黄金票据、白银票据、、PTT、PTK
这些老生常谈的词汇再熟悉不过了，对其利用也应该是了如指掌了吧。但是如果你对其背后的所使用的原理还不太了解的话，那么这篇（系列）文章你一定不能错过。

在本篇文章中，我们将对 Kerberos 协议与 Kerberos 认证原理分模块进行详细的讲解，为下篇文章中讲解 Kerberos
认证原理的安全问题做下铺垫。

## Kerberos 协议

Kerberos
协议是一种计算机网络授权协议，用来在非安全网络中，对个人通信以安全的手段进行身份认证。其设计目标是通过密钥系统为客户机与服务器应用程序提供强大的认证服务。该协议的认证过程的实现不依赖于主机操作系统的认证，无需基于主机地址的信任，不要求网络上所有主机的物理安全，并假定网络上传送的数据包可以被任意地读取、修改和插入数据。在以上情况下，
Kerberos 作为一种可信任的第三方认证服务，是通过传统的密码技术（如：共享密钥）执行认证服务的。Kerberos
协议在在内网域渗透领域中至关重要，白银票据、黄金票据、攻击域控等都离不开 Kerberos 协议。

为了让阁下能够更轻松地理解后文对认证原理的讲解，你需要先了解以下几个关键角色：

角色| 作用  
---|---  
Domain Controller| 域控制器，简称DC，一台计算机，实现用户、计算机的统一管理。  
Key Distribution Center| 秘钥分发中心，简称KDC，默认安装在域控里，包括AS和TGS。  
Authentication Service| 身份验证服务，简称AS，用于KDC对Client认证。  
Ticket Grantng Service| 票据授予服务，简称TGS，用于KDC向Client和Server分发Session Key（临时秘钥）。  
Active Directory| 活动目录，简称AD，用于存储用户、用户组、域相关的信息。  
Client| 客户端，指用户。  
Server| 服务端，可能是某台计算机，也可能是某个服务。  
  
打个比方：当 whoami 要和 bunny 进行通信的时候，whoami 就需要向 bunny 证明自己是whoami，直接的方式就是 whoami
用二人之间的秘密做秘钥加密明文文字生成密文，把密文和明文文字一块发送给 bunny，bunny
再用秘密解密得到明文，把明文和明文文字进行对比，若一致，则证明对方是 whoami。

但是网络中，密文和文字很有可能被窃取，并且只要时间足够，总能破解得到秘钥。所以不能使用这种长期有效的秘钥，要改为短期的临时秘钥。那么这个临时秘钥就需要一个第三方可信任的机构来提供，即
KDC（Key Distribution Center）秘钥分发中心。

## Kerberos 认证原理

首先我们根据以下这张图来大致描述以下整个认证过程：

![]()image-20210506012109792

1.首先 Client 向域控制器 DC 请求访问 Server，DC 通过去 AD 活动目录中查找依次区分 Client 来判断 Client
是否可信。2.认证通过后返回 TGT 给 Client，Client 得到 TGT（Ticket Granting Ticket）。3.Client
继续拿着 TGT 请求 DC 访问 Server，TGS 通过 Client 消息中的 TGT，判断 Client 是否有访问权限。4.如果有，则给
Client 有访问 Server 的权限 Ticket，也叫 ST（Service Ticket）。5.Client 得到 Ticket 后，再去访问
Server，且该 Ticket 只针对这一个 Server 有效。6.最终 Server 和 Client 建立通信。

下面讲一下详细的认证步骤，大概分为三个阶段：

•AS_REQ & AS_REP•TGS_REQ & TGS_REP•AP-REQ & AP-REP

### AS_REQ & AS_REP

该阶段是 Client 和 AS 的认证，通过认证的客户端将获得 TGT 认购权证。

![](https://gitee.com/fuli009/images/raw/master/public/20210804120611.png)image-20210506094949950

当域内某个客户端用户 Client 视图访问域内的某个服务，于是输入用户名和密码，此时客户端本机的 Kerberos 服务会向 KDC 的 AS
认证服务发送一个 `AS_REQ` 认证请求。请求的凭据是 Client 的哈希值 NTLM-Hash 加密的时间戳以及 Client-
info、Server-info 等数据，以及一些其他信息。

当 Client 发送身份信息给 AS 后，AS 会先向活动目录 AD 请求，询问是否有此 Client 用户，如果有的话，就会取出它的 NTLM-
Hash，并对 `AS_REQ` 请求中加密的时间戳进行解密，如果解密成功，则证明客户端提供的密码正确，如果时间戳在五分钟之内，则预认证成功。然后 AS
会生成一个临时秘钥 Session-Key AS，并使用客户端 Client 的 NTLM-Hash 加密 Session-key AS
作为响应包的一部分内容。此 Session-key AS 用于确保客户端和 KGS 之间的通信安全。

还有一部分内容就是 TGT：使用 KDC 一个特定账户的 NTLM-Hash 对 Session-key AS、时间戳、Client-info
进行的加密。这个特定账户就是创建域控时自动生成的 Krbtgt 用户，然后将这两部分以及 PAC 等信息回复给 Client，即 `AS_REP` 。PAC
中包含的是用户的 SID、用户所在的组等一些信息。

> AS-REP 中最核心的东西就是 Session-key 和 TGT。我们平时用 Mimikatz、kekeo、rubeus 等工具生成的凭据是
> .kirbi 后缀，Impacket 生成的凭据的后缀是 .ccache。这两种票据主要包含的都是 Session-key 和
> TGT，因此可以相互转化。

至此，Kerberos 认证的第一步完成。

### TGS_REQ & TGS_REP

该阶段是 Client 和 TGS 的认证，通过认证的客户端将获得 ST 服务票据。

![]()image-20210506095108816

客户端 Client 收到 AS 的回复 `AS_REP` 后分别获得了 TGT 和加密的 Session-Key AS。它会先用自己的 Client
NTLM-hash 解密得到原始的 Session-Key AS，然后它会在本地缓存此 TGT 和原始的 Session-Key
AS，如果现在它就需要访问某台服务器上的服务，他就需要凭借这张 TGT 认购凭证向 KGS 购买相应的 ST 服务票据（也叫Ticket）。

此时 Client 会使用 Session-Key AS 加密时间戳、Client-info、Server-info 等数据作为一部分。由于 TGT 是用
Krbtgt 账户的 NTLM-Hash 加密的，Client 无法解密，所以 Client 会将 TGT 作为另一部分继续发送给
TGS。两部分组成的请求被称为 `TGS_REQ`。

TGS 收到该请求，用 Krbtgt 用户的 NTLM-hash 先解密 TGT 得到 Session-key AS、时间戳、Client-info 以及
Server-info。再用 Session-key AS 解密第一部分内容，得到 Client-
info、时间戳。然后将两部分获取到时间戳进行比较，如果时间戳跟当前时间相差太久，就需要重新认证。TGS 还会将这个 Client 的信息与 TGT 中的
Client 信息进行比较，如果两个相等的话，还会继续判断 Client 有没有权限访问 Server，如果都没有问题，认证成功。认证成功后，KGS
会生成一个 Session-key TGS，并用 Session-key AS 加密 Session-key TGS 作为响应的一部分。此 Session-
key TGS 用于确保客户端和服务器之间的通信安全。

另一部分是使用服务器 Server 的 NTLM-Hash 加密 Session-key TGS、时间戳以及 Client-info 等数据生成的
ST。然后 TGS 将这两部分信息回复给 Client，即 `TGS_REP` 。

至此，Client 和 KDC 的通信就结束了，然后是和 Server 进行通信。

### AP-REQ & AP-REP

该阶段是 Client 和 TGS 的认证，通过认证的客户端将与服务器建立连接。

![](https://gitee.com/fuli009/images/raw/master/public/20210804120612.png)image-20210506095349321

客户端 Client 收到 `TGS_REP` 后，分别获得了 ST 和加密的 Session-Key TGS。它会先使用本地缓存了的 Session-
key AS 解密出了原始的 Session-key TGS。然后它会在本地缓存此 ST 和原始的 Session-Key
TGS，当客户端需要访问某台服务器上的服务时会向服务器发送请求。它会使用 Session-key TGS 加密 Client-
info、时间戳等信息作为一部分内容。ST 因为使用的是 Server NTLM-hash 进行的加密，无法解密，所以会原封不动发送给
Server。两部分一块发送给 Server，这个请求即是 `AP_REQ`。

Server 收到 `AP_REQ` 请求后，用自身的 Server NTLM-Hash 解密了 ST，得到 Session-Key
TGS，再解密出Client-info、时间戳等数据。然后与 ST 的Client-
info、时间戳等进行一一对比。时间戳有效时间一般时间为8小时。通过客户端身份验证后，服务器 Server 会拿着 PAC 去询问 DC
该用户是否有访问权限，DC 拿到 PAC 后进行解密，然后通过 PAC 中的 SID
判断用户的用户组信息、用户权限等信息，然后将结果返回给服务端，服务端再将此信息域用户请求的服务资源的 ACL
进行对比，最后决定是否给用户提供相关的服务。通过认证后 Server 将返回最终的 `AP-REP` 并与 Client 建立通信。

至此，Kerberos 认证流程基本结束。

## PAC

我们在前面关于 Kerberos 认证流程的介绍中提到了 PAC（Privilege Attribute
Certificate）这个东西，这是微软为了访问控制而引进的一个扩展，即特权访问证书。

在上面的认证流程中，如果没有 PAC 的访问控制作用的话，只要用户的身份验证正确，那么就可以拿到 TGT，有了 TGT，就可以拿到 ST，有了 ST
，就可以访问服务了。此时任何一个经过身份验证的用户都可以访问任何服务。像这样的认证只解决了 "Who am i?" 的问题，而没有解决 "What can
I do?" 的问题。

为了解决上面的这个问题，微软引进了PAC。即 KDC 向客户端 Client 返回 `AS_REP` 时插入了 PAC，PAC 中包含的是用户的
SID、用户所在的组等一些信息。当最后服务端 Server 收到 Client 发来的 `AP_REQ`
请求后，首先会对客户端身份验证。通过客户端身份验证后，服务器 Server 会拿着 PAC 去询问 DC 该用户是否有访问权限，DC 拿到 PAC
后进行解密，然后通过 PAC 中的 SID 判断用户的用户组信息、用户权限等信息，然后将结果返回给服务端，服务端再将此域用户请求的服务资源的 ACL
进行对比，最后决定是否给用户提供相关的服务。

> 但是在有些服务中并没有验证 PAC 这一步，这也是白银票据能成功的前提，因为就算拥有用户的 Hash，可以伪造 TGS，但是也不能制作 PAC，PAC
> 当然也验证不成功，但是有些服务不去验证 PAC，这是白银票据成功的前提。

## Kerberos 认证中的相关安全问题概述

Kerberos 认证并不是天衣无缝的，这其中也会有各种漏洞能够被我们利用，比如我们常说的
MS14-068、黄金票据、白银票据等就是基于Kerberos协议进行攻击的。下面我们便来大致介绍一下Kerberos 认证中的相关安全问题。

### 黄金票据（Golden ticket）

在 Windows 的kerberos认证过程中，Client 将自己的信息发送给 KDC，然后 KDC 使用 Krbtgt 用户的 NTLM-Hash
作为密钥进行加密，生成 TGT。那么如果获取到了 Krbtgt 的 NTLM-Hash 值，不就可以伪造任意的 TGT 了吗。因为 Krbtgt
只有域控制器上面才有，所以使用黄金凭据意味着你之前拿到过域控制器的权限，黄金凭据可以理解为一个后门。

先假设这么一种情况，原先已拿到的域内所有的账户 Hash，包括 Krbtgt
这个账户，由于有些原因导致你对域管权限丢失，但好在你还有一个普通域用户权限，碰巧管理员在域内加固时忘记重置 Krbtgt
密码，基于此条件，我们还能利用该票据重新获得域管理员权限。利用 Krbtgt 的 Hash 值可以伪造生成任意的
TGT，能够绕过对任意用户的账号策略，让用户成为任意组的成员，可用于 Kerberos 认证的任何服务。

### 白银票据（Silver ticket）

白银票据不同于黄金票据，白银票据的利用过程是伪造 TGS，通过已知的授权服务密码生成一张可以访问该服务的 TGT。因为在票据生成过程中不需要使用
KDC，所以可以绕过域控制器，很少留下日志。而黄金票据在利用过程中由 KDC 颁发 TGT，并且在生成伪造的 TGT 得 20 分钟内，TGS不会对该
TGT 的真伪进行效验。

白银票据依赖于服务账号的密码散列值，这不同于黄金票据利用需要使用 Krbtgt 账号的密码哈希值，因此更加隐蔽。

### MS14-068

这里便用到了我们之前所讲到的 PAC 这个东西，PAC 是用来验证 Client 的访问权限的，它会被放在 TGT 里发送给 Client，然后由
Client 发送给 TGS。但也恰恰是这个 PAC 造成了MS14-068这个漏洞。

该漏洞是位于 kdcsvc.dll 域控制器的密钥分发中心（KDC）服务中的 Windows 漏洞，它允许经过身份验证的用户在其获得的票证 TGT
中插入任意的 PAC 。普通用户可以通过呈现具有改变了 PAC 的 TGT 来伪造票据获得管理员权限。

### 密码喷洒攻击（Password Spraying）

在实际渗透中，许多渗透测试人员和攻击者通常都会使用一种被称为 “密码喷洒”（Password
Spraying）的技术来进行测试和攻击。对密码进行喷洒式的攻击，这个叫法很形象，因为它属于自动化密码猜测的一种。这种针对所有用户的自动密码猜测通常是为了避免帐户被锁定，因为针对同一个用户的连续密码猜测会导致帐户被锁定。所以只有对所有用户同时执行特定的密码登录尝试，才能增加破解的概率，消除帐户被锁定的概率。普通的爆破就是用户名固定，爆破密码，但是密码喷洒，是用固定的密码去跑用户名。

### AS-REP Roasting

我们前文说过，AS_REQ & AS_REP 认证的过程是 Kerberos
身份认证的第一步，该过程又被称为预身份验证。预身份验证主要是为了防止密码脱机爆破。

而如果域用户设置了选项 "Do not require Kerberos
preauthentication"（该选项默认没有开启）关闭了预身份验证的话，攻击者可以使用指定的用户去请求票据，向域控制器发送 `AS_REQ`
请求，此时域控会不作任何验证便将 TGT 票据和加密的 Session-key 等信息返回。因此攻击者就可以对获取到的加密 Session-key
进行离线破解，如果爆破成功，就能得到该指定用户的明文密码。

这种攻击方式被称作 AS-REP Roasting 攻击。

## Ending......

![](https://gitee.com/fuli009/images/raw/master/public/20210804120613.png)

本节中我们对 Kerberos 协议与 Kerberos 认证原理分模块进行详细的讲解。在下篇文章，我们将详细的讲解 Kerberos
认证原理的安全问题并演示相关的攻击过程。

![]()

  

 **推荐阅读：**

  

本月报名可以参加抽奖送暗夜精灵6Pro笔记本电脑的优惠活动  

  

[![](https://gitee.com/fuli009/images/raw/master/public/20210804120614.png)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496998&idx=1&sn=da047300e19463fc88fcd3e76fda4203&chksm=ec1ca019db6b290f06c736843c2713464a65e6b6dbeac9699abf0b0a34d5ef442de4654d8308&scene=21#wechat_redirect)

  

 **点赞，转发，在看**

  

原创投稿作者：WHOAMI

文章首发在Freebuf

![](https://gitee.com/fuli009/images/raw/master/public/20210804120615.png)

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

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

内网渗透 | Kerberos 协议与 Kerberos 认证原理

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

