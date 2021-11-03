#  一文读懂Kerberos认证流程

原创 松神  [ 红队防线 ](javascript:void\(0\);)

**红队防线** ![]()

微信号 klionsec

功能介绍 没有章法，便是最好的章法

____

__

收录于话题

**Windows 如何判断域登陆OR本地登陆**

  

当用户开机启动后或者按下“Ctrl + Alt +
Del”之后，Winlogon服务将会被调起，同时向大家展示需要输入登录用户名密码（由Gina.dll定义）,当用户输入相关信息之后，Windows应该如何判断是域用户登录呢还是本地用户登录？

  

如果是本地用户登录的话将会使用本地数据库进行认证，如果是域渗透的话将会丢给Kerberos
SSP去认证。当用户按下键盘Crt+Alt+Del后，Winlogon读取完用户的身份凭据后，把它交给本地安全机构（LSA），LSA会对凭证做一系列安全加密编码操作如MD4加密，加密结束后会通过SSPI(安全支持提供者接口，该接口负责与Kerberos和NTLM服务沟通)来判断是应该交给Ntlm处理，还是Kerberos
SSPI进行处理。LSA首先根据用户输入UPN等信息会事先把身份认证请求传递到Kerberos SSP。

  

Kerberos
SSP验证用户登入目标是本地计算机还是域，如果是域则继续向下处理，如果是本地计算机则会向SSPI返回一条错误消息，SSPi将它将这个任务交回给GINA处理。

SSPI现在发送请求到下一个安全提供程序——NTLM。NTLM SSP会将请求交给Netlogon服务针对LSAM （Local Security
Account Manager，本地安全账户管理器）数据库进行身份认证。使用NTLM SSP的身份认证过程与Windows NT系统的身份认证方法是相同的。

  

 **基础概述  
**

一个思想：Kerberos协议只认证，不鉴权，鉴权是服务账户所做。

注意事项：

  * 问：在客户端与Application Server访问阶段，用户对Application Server认证是否有权限访问时，是否会把PAC整个数据包交给KDC去做权限验证？

  * 答案：不会。Application Server仅仅会发送PAC结构中KERB_VERIFY_PAC 信息交给KDC以验证签名，将会发送以下4个字段内容。一般情况是不会去主动发送以下4个字段去询问KDC。

  * 

    
    
    MessageType ,ChecksumLength,SignatureType,SignatureLength,ChecksumAndSignature

  

 **什么是 PAC？**

答：KDC在向Kerberos客户端颁发TGT时，会向本地LSA请求生成一个特殊的数据结构，名为“特权访问证书”（Privilege Access
Certificate,PAC）,这个PAC包含为Kerberos客户端构建一个本地访问令牌所需的用户信息，他同时使用域控制器服务器的私钥和KDC服务器的私钥来进行数字签署。以防假的伪造PAC。

PAC包含了用户登陆时间，用户登录名,用户RID，用户所属信息，PAC认证次数等。【注意：签名不是加密，只是一个特征标识】

  

  

 ** **Kerberos 认证过程****

 ** **  
****

如图演示，在认证过程中共有8个步骤,其中6，7，8这三个步骤使用虚线表示，也就代表这些是可选步骤。

Kerberos协议意义在于，需要A 和 B
两个不同的一方可以向对方证明他们知道相同的密钥，而不会将密钥泄露给另一方，同时证明每一方是谁说他们是正确的，后续引入了第三方C。A和B都信任C，所以C可以作为双方的中间人。

  

![](https://gitee.com/fuli009/images/raw/master/public/20211103221101.png)

 ** ******  

 **1.KRB_AS_REQ**

  *   *   *   * 

    
    
     User hash（timestamp）：用户密码hash加密当前的时间戳Username ：用户名SPN krbtgt ：服务账户SPNUser Nonce ：用户随机数

其中User hash（timestamp）是因为默认KDC为每一个认证都开启了Pre-
Authentication，使用当前时间戳和用户Hash加密，以防任何人都过来请求TGT。

![](https://gitee.com/fuli009/images/raw/master/public/20211103221108.png)

  

  

 **2.KRB_AS_REP**

KDC收到请求包之后，KDC(DC)拥有所有人的Hash信息,
它将KRB_AS_REQ中的用户名到数据库中查找，查找到之后使用该用户名所对应的Hash解密出时间戳，并且解开User Nonce等。

* * *

  

KDC在验证用户身份没有问题后会给用户颁发2个信息，一个使用当前User用户的hash加密的基础信息，此信息包含了一个Session
key，用户随机数，以及TGT过期时间。

另外一个是TGT票据，整个TGT都是用krbtgt hash进行加密。其他人无法解密TGT，仅仅KDC自己。其中包含了用户名，Session
key，TGT过期时间，以及PAC特权。

  

![](https://gitee.com/fuli009/images/raw/master/public/20211103221109.png)

PAC除了包含用户，RID，用户信息，密码情况，登录时间等基础之外，还有两个签名KDC签名和服务签名。  

  *   *   * 

    
    
    PAC签名：KDC签名：使用DC的krbtgt 签名服务签名：谁提供服务谁就是自己的服务hash签名（这里跟AS认证，所以服务是krbtgt）

当前User请求的服务对象是都是KDC，则两个签名都是有krbtgt签发。但加密算法不同  

![](https://gitee.com/fuli009/images/raw/master/public/20211103221111.png)

  

  

 **KRB_TGS_REQ**

用户收到KDC发来的两个信息之后，其中第一个用user hash加密的内容，用户自己可以解密出相关内容，提取出 Session key，TGT
过期时间，用户随机数。但是TGT中的内容 用户没有办法解密，因为他没有krbtgt的密钥。

* * *

  

  

用户解开数据包确认TGT过期时间以及随机数等信息正常后，想要去访问某一个服务（在域环境中，某个服务需要以Kerberos身份进行认证，需要注册一个SPN）需要指定该服务SPN，这一次请求需要指定SPN，用户随机数，TGT（用krbtgt加密），以及使用Session
key加密的用户名以及时间戳。

![](https://gitee.com/fuli009/images/raw/master/public/20211103221112.png)

  

 **KRB_TGS_REP**

KDC收到User信息之后使用Session key解密出用户名与时间戳确认了身份。

* * *

  

1.把上述请求中的SPN字段SPN账户以及该服务账户所对应的Hash找出来并对相关内容进行加密形成一个TGS。

注意：TGS中包含了一个新的Service Session key，整个TGS是使用服务账户Hash进行加密。

  

2.KDC使用User与KDC之间前面使用过的Session key继续加密一部分内容，用户随机数，TGT过期时间，Service Session
key等相关内容

将第一步与第二步一起发送给User。其中Service owner hash加密的TGS 用户无法打开，仅应用服务本身可以打开。

3.TGS中包含的PAC签名有所变动，还是使用两个签名，一个签名为krbtgt签名，另外一个使用服务签名，这里服务签名也就是SPN指定的服务hash。

  *   *   * 

    
    
    NEW PAC签名KDC签名：使用DC的krbtgt 签名服务签名：Service hash 签名

  

![](https://gitee.com/fuli009/images/raw/master/public/20211103221113.png)

  

 **KRB_AP-REQ**

User收到信息后，使用自己前面与KDC通讯用的Session key解密出用户随机数，TGS过期时间以及Service Session
key之后。将TGS原封不动的发送给Application Server（AP）

  

User将使用Session key解密出Service Session key之后再使用Service Session
key加密自己的用户名以及时间戳发送给AP。

  

AP收到后，使用自己的服务hash解密出TGS中的内容，PAC，username ，TGT过期时间，Service session
key等内容。随之把解开Service session
key解密上述用户的username以及timestamp（时间戳），同时验证PAC中签名是否被篡改。

  

与此同时AP（application
server）会把PAC缓存到本地操作系统中，解析PAC中的信息然后对比ALC来决定用户是否可以访问某些资源，同时也方便下一次该此用户来访问服务时，重新请求TGT。

![](https://gitee.com/fuli009/images/raw/master/public/20211103221114.png)

  

  

到这一步基本认证就完成了。后续动作都是可选的。

  
AP（application
server）是不会主动把PAC整个数据包发送给KDC做认证，如果有场景需要仅仅也会发送PAC中的签名，也就是KERB_VERIFY_PAC
结构字段。如下图所示

![](https://gitee.com/fuli009/images/raw/master/public/20211103221116.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20211103221117.png)

  

KDC验证结束签名内容正常后会给Server返回一个RPC Code 以通知Server，但是一般情况下是不会去主动请求KDC来认证的。

  

  *   *   * 

    
    
    Tips1：User在与AS第一次认证的时候有效票据只有5分钟有效，防止爆破。所以活动目录中的机器时间需要与域控制器同步。这也是使用net time /do命令定位域控制器的一个手段。

  

  

参考链接：[MS-APDS]: Kerberos PAC Validation

参考链接：https://docs.microsoft.com/zh-cn/openspecs/windows_protocols/ms-
apds/88aacb11-6c8d-40f0-b0c3-049f1ad08447

参考链接：MS_APDS中KERB_VERIFY_PAC_message

  

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

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

一文读懂Kerberos认证流程

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

