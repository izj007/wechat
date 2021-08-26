##  内网渗透 | 利用 RDP 协议搭建 Socks5 代理隧道

原创 WHOAMI  [ HACK学习呀 ](javascript:void\(0\);)

**HACK学习呀** ![]()

微信号 Hacker1961X

功能介绍
HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

____

__

收录于话题

#隧道 1

#远程桌面 3

#内网渗透 20

![](https://gitee.com/fuli009/images/raw/master/public/20210826091341.png)

## 前言

如今，在很多组织机构内部，针对 DMZ 或隔离网络区域内的计算机设备，为了限制其它接入端口风险，通常只允许这些设备开启 3389
端口，使用远程桌面来进行管理维护。那么我们能不能利用这个 3389 端口的 RDP 服务建立起一条通向内网的代理隧道呢？当然可以，下面就引出我们今天的主角
—— SocksOverRDP。

## SocksOverRDP

•项目地址：https://github.com/nccgroup/SocksOverRDP

SocksOverRDP 可以将 SOCKS 代理的功能添加到远程桌面服务，它使用动态虚拟通道，使我们能够通过开放的 RDP
连接进行通信，而无需在防火墙上打开新的套接字、连接或端口。此工具在 RDP 协议的基础上实现了 SOCKS 代理功能，就像 SSH 的 `-D`
参数一样，在建立远程连接后，即可利用 RDP 协议实现代理功能。

该工具可以分为两个部分：

![](https://gitee.com/fuli009/images/raw/master/public/20210826091342.png)image-20210805164336410

•第一部分是一个 `.dll` 文件，需要在 RDP 连接的客户端上进行注册，并在每次运行时将其加载到远程桌面客户端 mstsc
的上下文运行环境中。•第二部分是一个 `.exe` 可执行文件，它是服务端组件，需要上传到 RDP 连接的服务器并执行。

以下是该工具的运作原理：

> If the DLL is properly registered, it will be loaded by the mstsc.exe
> (Remote Desktop Client) or Citrix Receiver every time it is started. When
> the server executable runs on the server side, it connects back to the DLL
> on a dynamic virtual channel, which is a feature of the Remote Desktop
> Protocol. After the channel is set up, a SOCKS Proxy will spin up on the
> client computer, by default on 127.0.0.1:1080. This service can be used as a
> SOCKS5 Proxy from any browser or tool.

大致原理是，当 SocksOverRDP-Plugin.dll 在 RDP 客户端上被正确注册后，每次启动远程桌面时都会由 mstsc 加载。接着，当
SocksOverRDP-Server.exe 被上传到 RDP 服务端上传并执行后 ，SocksOverRDP-Server.exe
会在动态虚拟通道上回连 SocksOverRDP-Plugin.dll，这是远程桌面协议的一个功能。虚拟通道设置完成后，SOCKS 代理将在 RDP
客户端计算机上启动，默认为 127.0.0.1:1080。此服务可用作任何浏览器或工具的 SOCKS5
代理。并且服务器上的程序不需要服务器端的任何特殊特权，还允许低特权用户打开虚拟通道并通过连接进行代理。

## 通过 SocksOverRDP 搭建 SOCKS5 代理

测试环境如下：

![]()image-20210805190918631

右侧为一个内网环境，其中 Windows Server 2012 是一个 Web
服务器，有两个网卡，分别连通内外网。假设此时攻击者已经通过渗透手段拿下了这台 Web
服务器，需要设置代理进入内网继续对内网进行横向渗透。但是由于防火墙的规则等原因，只允许 TCP/UDP 3389 端口可以进行通信，所以我们只能尝试利用用
RDP 协议来建立通讯隧道。

### 攻击端

在攻击机上需要安装注册 SocksOverRDP-Plugin.dll。首先我们将 SocksOverRDP-Plugin.dll
放置到攻击机的任何目录中，但是为了方便我们可以将其放置到 `％SYSROOT％\system32\` 或 `％SYSROOT％\SysWoW64\`
目录下。

然后使用以下命令对 SocksOverRDP-Plugin.dll 进行安装注册：

    
    
    regsvr32.exe SocksOverRDP-Plugin.dll    # 注册# regsvr32.exe /u SocksOverRDP-Plugin.dll    取消注册

![](https://gitee.com/fuli009/images/raw/master/public/20210826091343.png)image-20210805171250365

如上图所示，注册成功。但是由于 SocksOverRDP 建立的 SOCKS5 代理是默认监听在 127.0.0.1:1080
上的，所以只能从攻击机本地使用，为了让攻击者的 Kali 也能使用搭建在攻击机 Windows 10 上的 SOCKS5 代理，我们需要修改其注册表，将
IP 从 127.0.0.1 改为 0.0.0.0。注册表的位置为：

    
    
    HKEY_CURRENT_USER\SOFTWARE\Microsoft\Terminal Server Client\Default\AddIns\SocksOverRDP-Plugin

![](https://gitee.com/fuli009/images/raw/master/public/20210826091344.png)image-20210805183835850

然后启动远程桌面客户端 mstsc.exe 连接目标 Web 服务器 Windows Server 2012：

![](https://gitee.com/fuli009/images/raw/master/public/20210826091345.png)image-20210805184009799

如上图所示，弹出了一个提示说 SocksOverRDP 成功启动，当服务端的可执行文件运行后即可在攻击机的 1080 端口上启动 SOCKS5 代理服务。

### 服务端

远程桌面连接成功后，将服务端组件 SocksOverRDP-Server.exe 上传到 Windows Server 2012 上：

![](https://gitee.com/fuli009/images/raw/master/public/20210826091346.png)image-20210805172727184

直接运行 SocksOverRDP-Server.exe 即可：

![](https://gitee.com/fuli009/images/raw/master/public/20210826091347.png)image-20210805172901544

此时便成功搭建了一个 SOCKS5 代理隧道，查看攻击机 Windows 10 的端口连接状态发现已经建立连接：

![](https://gitee.com/fuli009/images/raw/master/public/20210826091348.png)image-20210805173743464

然后在攻击机 Kali 上配置好 proxychains：

![]()image-20210805184447817

此时便可以通过代理访问到内网的主机 DC 了。如下所示，成功打开了 DC 的远程桌面：

    
    
    proxychains4 rdesktop 192.168.93.30

![](https://gitee.com/fuli009/images/raw/master/public/20210826091350.png)image-20210805184342519

探测内网主机 DC 的端口开放情况：

    
    
    proxychains4 nmap -sT -Pn 192.168.93.30 -p 445

![](https://gitee.com/fuli009/images/raw/master/public/20210826091352.png)image-20210805185347989

先该主机开启了 445 端口，我们可以直接用 smbexec.py 连接：

    
    
    proxychains4 python3 smbexec.py whoamianony/administrator:Whoami2021@192.168.93.30

![](https://gitee.com/fuli009/images/raw/master/public/20210826091353.png)image-20210805190126459

如上图所示，成功拿下内网主机 DC。

  

文中若有不当之处，还请各位大佬师傅们多多点评。

> 参考：
>
> https://github.com/nccgroup/SocksOverRDP

  

**![](https://gitee.com/fuli009/images/raw/master/public/20210826091354.png)**

  

 **推荐阅读：**

  

[
**内网渗透｜获取远程桌面连接记录与RDP凭据**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247498865&idx=1&sn=ffbc46a21eeefa69862acc3d0d36afe7&chksm=ec1ca94edb6b20583bf5df5c5d62e43dc2a86e924c1aae58f434e49f677b310c58a0843af370&scene=21#wechat_redirect)  

  

[ **内网渗透｜基于文件传输的 RDP
反向攻击**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247498826&idx=1&sn=0ea380bf43e25f5a4e8ad4c20451226a&chksm=ec1ca975db6b2063cc5b9191f3841cdc1f7e57088169932ee4e61e55f046737cee76a648da76&scene=21#wechat_redirect)  

  

[ **内网渗透 |
获取远程主机保存的RDP凭据密码**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247488505&idx=1&sn=fbe002f0fabce8d01adfd5f3271a85c2&chksm=ec1f46c6db68cfd015fe803c1b0ea4b492032c53e9269ae23120edba267407f69beddaf0767c&scene=21#wechat_redirect)

  

 **点赞，转发，在看**

  

原创投稿作者：WHOAMI

文章首发Freebuf

![](https://gitee.com/fuli009/images/raw/master/public/20210826091356.png)

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

内网渗透 | 利用 RDP 协议搭建 Socks5 代理隧道

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

