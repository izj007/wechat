#  Hughes多款卫星终端存在漏洞,附Comtech过往漏洞列表

[ 太空安全 ](javascript:void\(0\);)

**太空安全** ![]()

微信号 SateSec

功能介绍 学习卫星互联网，研究卫星通信安全！

____

___发表于_

收录于合集

![]()

标题：Hughes 卫星路由器远程文件包含跨框架脚本  

咨询 ID：ZSL-2022-5743

类型：本地/远程

影响：安全绕过、敏感信息暴露、跨站点脚本

风险：(4/5)

发布日期：28.12 .2022年

受影响版本

HX200 v8.3.1.14

HX90 v6.11.0.5

HX50L v6.10.0.18

HN9460 v8.2.0.48

HN7000S v6.9.0.37

概括

HX200 是一款高性能卫星路由器，旨在使用动态分配的高带宽卫星 IP 连接来提供运营商级 IP 服务。HX200 卫星路由器提供灵活的服务质量 (QoS)
功能，可根据每个远程路由器的网络应用进行定制，例如自适应恒定比特率 (CBR) 带宽分配，为实时传输提供高质量、低抖动带宽。时间流量，例如 IP 语音
(VoIP) 或视频会议。HX200 具有集成的 IP 功能，包括 RIPv1、RIPv2、BGP、DHCP、NAT/PAT 和 DNS
服务器/中继功能，以及高性能卫星调制解调器，是一款具有集成高性能卫星路由器的全功能 IP 路由器。

![]()

![]()

描述

该路由器包含通过远程文件包含漏洞进行的跨框架脚本编写，恶意用户可能会利用该漏洞来危害受影响的系统。此漏洞可能允许未经身份验证的恶意用户滥用框架，包括
JS/HTML 代码并窃取应用程序合法用户的敏感信息。

小贩

休斯网络系统有限责任公司 - https://www.hughes.com

  

  

已测试

风网/1.0

供应商状态

不适用

概念验证

Hughes_rfi_xfs.js

speedtest.html

制作人员

Gjoko Krstic 发现的漏洞 - < gjoko@zeroscience.mk >

参考

[1] https://packetstormsecurity.com/files/170343/

[2] https://forum.it.mk/threads/hughes-satellite-router-remote-file-inclusion-
cross-frame-scripting.82755/

[3] https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-22971

[4] https://nvd.nist.gov/vuln/detail/CVE-2023-22971

[5] https://exchange.xforce.ibmcloud.com/vulnerability/245814

[6] https://www.exploit-db.com/exploits/51190

变更日志

[28.12.2022] - 初始版本

[29.12.2022] - 添加参考 [1] 和 [2]

[26.01.2023] - 添加参考 [3] 和 [4]

[10.02.2023] - 添加参考 [5]

[ 10.04.2023] - 添加了参考文献 [6]

![]()

姓名| 描述  
---|---  
CVE-2019-15652| 18.1.0 之前的 NSSLGlobal SatLink VSAT 调制解调器单元 (VMU) 设备的 Web
界面无法正确清理错误消息的输入，从而导致能够注入客户端代码。  
  
Comtech 卫星路由器过往漏洞列表

姓名

描述

  
|  
  
---|---  
  
CVE-2020-7244

|

Comtech Stampede FX-1010 7.4.3 设备允许经过身份验证的远程管理员通过导航到“轮询路由”页面并在“路由器 IP 地址”字段中输入
shell 元字符来实现远程代码执行。（在某些情况下，可以使用 comtech 帐户的 comtech 密码来实现身份验证。）  
  
CVE-2020-7243

|

Comtech Stampede FX-1010 7.4.3 设备允许经过身份验证的远程管理员通过导航到 Fetch URL 页面并在 URL 字段中输入
shell 元字符来实现远程代码执行。（在某些情况下，可以使用 comtech 帐户的 comtech 密码来实现身份验证。）  
  
CVE-2020-7242

|

Comtech Stampede FX-1010 7.4.3 设备允许经过身份验证的远程管理员通过导航到“诊断跟踪路由”页面并在“目标 IP
地址”字段中输入 shell 元字符来实现远程代码执行。（在某些情况下，可以使用 comtech 帐户的 comtech 密码来实现身份验证。）  
  
CVE-2020-5179

|

Comtech Stampede FX-1010 7.4.3 设备允许经过身份验证的远程管理员通过导航到“诊断 Ping”页面并在“目标 IP
地址”字段中输入 shell 元字符来执行任意操作系统命令。（在某些情况下，可以使用 comtech 帐户的 comtech 密码来实现身份验证。）  
  
CVE-2019-17667

|

Comtech H8 Heights 远程网关 2.5.1 设备允许通过站点名称（又名 SiteName）字段进行 XSS 和 HTML 注入。  
  
  
  
Hughes卫星路由器过往漏洞列表

姓名

描述

  
|  
  
---|---  
  
CVE-2023-22971

|

HX200 v8.3.1.14、HX90 v6.11.0.5、HX50L v6.10.0.18、HN9460 v8.2.0.48 和 HN7000S
v6.9.0.37 的 Hughes Network Systems 路由器终端中存在跨站脚本 (XSS) 漏洞，允许未经身份验证的攻击者滥用框架、包含
JS/HTML 代码并窃取应用程序合法用户的敏感信息。  
  
CVE-2021-32997

|

受影响的 Baker Hughes Bentley Nevada 产品（3500 System 1 6.x，部件号 3060/00 版本 6.98
及更早版本，3500 System 1，部件号 3071/xx 和 3072/xx 版本 21.1 HF1 及更早版本，3500 机架配置，部件号
129133-01 版本 6.4 及更早版本以及 3500/22M 固件（部件号 288055-01 版本 5.05
及更早版本）使用弱加密算法来存储和传输敏感数据，这可能使攻击者更容易获取用于访问的凭据。  
  
CVE-2016-9497

|

Hughes 高性能宽带卫星调制解调器（型号为 HN7740S DW7000
HN7000S/SM）容易受到使用备用路径或通道的身份验证绕过攻击。默认情况下，端口 1953 可通过 telnet
访问，不需要身份验证。未经身份验证的远程用户可以通过此界面访问许多管理命令，包括重新启动调制解调器。  
  
CVE-2016-9496

|

Hughes 高性能宽带卫星调制解调器，型号为 HN7740S DW7000 HN7000S/SM，缺乏认证。未经身份验证的用户可能会向
http://[ip]/com/gatewayreset 或 http://[ip]/cgi/reboot.bin 发送 HTTP GET
请求，以导致调制解调器重新启动。  
  
CVE-2016-9495

|

Hughes 高性能宽带卫星调制解调器，型号为 HN7740S DW7000
HN7000S/SM，使用硬编码凭证。可以通过使用所有设备之间共享的几个默认凭据之一来访问设备的默认 telnet 端口 (23)。  
  
CVE-2016-9494

|

Hughes 高性能宽带卫星调制解调器（型号为 HN7740S DW7000
HN7000S/SM）可能容易受到不当输入验证的影响。从基本状态网页链接到的设备的高级状态网页似乎无法正确解析格式错误的 GET
请求。这可能会导致拒绝服务。  
  
CVE-2013-6035

|

GateHouse上的固件；哈里斯 BGAN RF-7800B-VU204 和 BGAN RF-7800B-DU204；休斯网络系统 9201、9450 和
9502；国际海事卫星组织；日本广播电台 JUE-250 和 JUE-500；Thuraya
IP卫星终端不需要对TCP端口1827上的会话进行身份验证，这使得远程攻击者可以通过未指定的协议操作执行任意代码。  
  
CVE-2013-6034

|

GateHouse上的固件；哈里斯 BGAN RF-7800B-VU204 和 BGAN RF-7800B-DU204；休斯网络系统 9201、9450 和
9502；国际海事卫星组织；日本广播电台 JUE-250 和 JUE-500；Thuraya
IP卫星终端具有硬编码凭证，这使得攻击者更容易通过未知向量获得未指定的登录访问权限。  
  
CVE-2001-1225

|

Hughes Technology Mini SQL 2.0.10 至 2.0.12 允许本地用户通过在表中创建非常大的数组来导致拒绝服务，从而导致查询表时
miniSQL 崩溃。  
  
CVE-2001-0580

|

Hughes Technologies 虚拟 DNS (VDNS) 服务器 1.0 允许远程攻击者通过连接到端口
6070、发送一些数据并关闭连接来造成拒绝服务。  
  
  

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

