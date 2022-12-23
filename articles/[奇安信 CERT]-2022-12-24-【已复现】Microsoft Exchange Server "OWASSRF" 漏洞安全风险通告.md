#  【已复现】Microsoft Exchange Server "OWASSRF" 漏洞安全风险通告

原创 QAX CERT  [ 奇安信 CERT ](javascript:void\(0\);)

**奇安信 CERT** ![]()

微信号 gh_64040028303e

功能介绍 为企业级用户提供高危漏洞、重大安全事件安全风险通告和相关产品解决方案。

____

___发表于_

收录于合集 #安全风险通告 386个

![](https://gitee.com/fuli009/images/raw/master/public/20221224002824.png)

奇安信CERT

 **致力于** 第一时间为企业级用户提供安全风险 **通告** 和 **有效** 解决方案。

  
  
  

 **安全通告**

  
  
Microsoft Exchange
Server是微软公司的一套电子邮件服务组件。除传统的电子邮件的存取、储存、转发功能外，在新版本的产品中亦加入了一系列辅助功能，如语音邮件、邮件过滤筛选和OWA（基于Web的电子邮件存取）。  
  

近日，奇安信CERT监测到CrowdStrike发布针对Microsoft Exchange Server新的利用链的技术细节，并将其命名为
**"OWASSRF"** ，其中涉及两个漏洞：

 **Microsoft Exchange Server权限提升漏洞(CVE-2022-41080)** ：经过身份认证的远程攻击者可通过Outlook
Web Application (OWA)端点获得在系统上下文中执行PowerShell的权限。

 **Microsoft Exchange Server远程代码执行漏洞(CVE-2022-41082)**
：具有执行PowerShell权限的远程攻击者可利用此漏洞在目标系统上执行任意代码。

  

组合这两个漏洞，经过身份认证的远程攻击者可通过Outlook Web Application (OWA)端点最终执行任意代码。值得注意的是，
**"OWASSRF"漏洞利用链绕过了之前Microsoft为"ProxyNotShell"提供的缓解措施。目前监测到POC及EXP已在互联网上公开，同时已发现在野利用。鉴于此漏洞利用链影响较大，建议客户尽快做好自查及防护。**

 **  
**

 **漏洞名称**

|

 **Microsoft  Exchange Server ****权限提升漏洞**  
  
---|---  
  
 **公开时间**

|

2022-11-08

|

 **更新时间**

|

2022-12-23  
  
 **CVE** **编号**

|

CVE-2022-41080

|

 **其他编号**

|

QVD-2022-43833

CNNVD-202211-2376  
  
 **威胁类型**

|

权限提升

|

 **技术类型**

|

服务端请求伪造  
  
 **厂商**

|

Microsoft

|

 **产品**

|

Exchange Server  
  
 **风险等级**  
  
 **奇安信** **CERT** **风险评级**

|

 **风险等级**  
  
 **高危**

|

 **蓝色（一般事件）**  
  
 **现时威胁状态**  
  
 **POC** **状态**

|

 **EXP** **状态**

|

 **在野利用状态**

|

 **技术细节状态**  
  
 **已发现**

|

 **已发现**

|

 **已发现**

|

 **已公开**  
  
 **漏洞描述**

|

Microsoft Exchange Server中存在权限提升漏洞，该漏洞允许经过身份认证的远程攻击者通过Outlook Web Application
(OWA)端点获得在系统上下文中执行PowerShell的权限。  
  
 **影响版本**

|

Microsoft Exchange Server 2016 Cumulative Update 22  
Microsoft Exchange Server 2019 Cumulative Update 11  
Microsoft Exchange Server 2013 Cumulative Update 23  
Microsoft Exchange Server 2019 Cumulative Update 12  
Microsoft Exchange Server 2016 Cumulative Update 23  
  
 **其他受影响组件**

|

无  
  
  

目前，奇安信CERT已成功复现 **Microsoft Exchange Server "OWASSRF"漏洞** ，截图如下：

![](https://gitee.com/fuli009/images/raw/master/public/20221224002839.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20221224002841.png)

  

  

威胁评估

 **漏洞名称**

|

 **Microsoft Exchange Server** **权限提升漏洞**  
  
---|---  
  
 **CVE** **编号**

|

CVE-2022-41080

|

 **其他编号**

|

QVD-2022-43833  
  
 **CVSS 3.1** **评级**

|

 **高危**

|

 **CVSS 3.1** **分数**

|

8.8  
  
 **CVSS** **向量**

|

 **访问途径（** **AV** **）**

|

 **攻击复杂度（** **AC** **）**  
  
网络

|

低  
  
 **所需权限（** **PR** **）**

|

 **用户交互（** **UI** **）**  
  
低权限

|

不需要  
  
 **影响范围（** **S** **）**

|

 **机密性影响（** **C** **）**  
  
不改变

|

高  
  
 **完整性影响（** **I** **）**

|

 **可用性影响（** **A** **）**  
  
高

|

高  
  
 **危害描述**

|

经过身份认证的远程攻击者可利用此漏洞获得在系统上下文中执行PowerShell的权限，配合CVE-2022-41082漏洞最终可在目标服务器上执行任意代码。  
  
  

  

处置建议

 **微软 已于11月发布此漏洞受影响版本的安全补丁，强烈建议受影响的用户尽快安装安全补丁进行防护。建议受影响用户通过以下链接进行手动更新：  
**

https://msrc.microsoft.com/update-guide/vulnerability/CVE-2022-41080

https://msrc.microsoft.com/update-guide/vulnerability/CVE-2022-41082 ****

  

 **缓解措施：**

1、若暂时无法应用补丁，建议禁用OWA来缓解此漏洞

2、禁止非管理员用户使用远程PowerShell访问

关于如何禁用单用户或多用户远程PoweShell访问可参考以下链接：

https://learn.microsoft.com/en-us/powershell/exchange/control-remote-
powershell-access-to-exchange-servers

3、确保X-Forwarded-For HTTP请求头记录真实的外部IP地址

  

另外，CrowdStrike已开发脚本用于监视IIS及Powershell日志：https://github.com/CrowdStrike/OWASSRF

  

  

产品解决方案

 **奇安信网站应用安全云防护系统已更新防护特征库**  

奇安信网神网站应用安全云防护系统已全面支持对Exchange SSRF漏洞(CVE-2022-41080)的防护。

  

 **奇安信网神网络数据传感器系统产品检测方案**

奇安信网神网络数据传感器（NDS5000/7000/9000系列）产品，已具备该漏洞的检测能力。规则ID为：7688，建议用户尽快升级检测规则库至2212221600以上。

  

 **奇安信天眼检测方案**

奇安信天眼新一代安全感知系统已经能够有效检测针对该漏洞的攻击，请将规则版本升级到3.0.1223.13685或以上版本。规则ID及规则名称：

0x10021419，Microsoft Exchange
服务端请求伪造漏洞(CVE-2022-41080)。奇安信天眼流量探针规则升级方法：系统配置->设备升级->规则升级，选择“网络升级”或“本地升级”。

  

 **奇安信自动化渗透测试系统产品解决方案**

奇安信自动化渗透测试系统在第一时间加入了该漏洞的插件规则和指纹规则，请将插件版本和指纹版本升级到20221223180110版本。规则名称：Microsoft
Exchange CVE-2022-41080
服务端请求伪造漏洞，插件版本：20221223180110，指纹版本：20221223180110。奇安信自动化渗透测试系统升级方法：系统管理->升级管理->插件升级（指纹升级），选择“网络升级”或“本地升级”。

  

  

参考资料

[1]https://unit42.paloaltonetworks.com/threat-brief-OWASSRF/  

[2]https://msrc.microsoft.com/update-guide/vulnerability/CVE-2022-41080

[3]https://msrc.microsoft.com/update-guide/vulnerability/CVE-2022-41082

[4]https://www.crowdstrike.com/blog/owassrf-exploit-analysis-and-
recommendations/

  

  

时间线

2022年12月14日，奇安信 CERT发布安全风险通告。  

  

点击 **阅读原文**

到奇安信NOX-安全监测平台查询更多漏洞详情

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

