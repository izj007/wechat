#  工具推荐|一款使用Golang开发的高并发网络扫描、服务探测工具。

Adminisme  [ Z2O安全攻防 ](javascript:void\(0\);)

**Z2O安全攻防** ![]()

微信号 Z2O_SEC

功能介绍 From zero to one

____

___发表于_

收录于合集

免责声明

  
  

 **本文仅用于技术讨论与学习，利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

 **只供对已授权的目标使用测试，对未授权目标的测试作者不承担责任，均由使用本人自行承担。**

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308202417.png)

文章正文

  
  

  

一款使用 **Golang** 开发且适用于攻防演习 **内网横向信息收集** 的 **高并发** 网络扫描、服务探测工具。

## 🍭Property

  * • 多平台支持（Windows、Mac、Linux、Cobalt Strike）

  * • 存活IP探测（支持TCP、ICMP两种模式）

  * • 超快的端口扫描

  * • 服务和应用版本检测功能，内置指纹探针采用:nmap-service-probes[1]

  * • Web服务（http、https）信息探测

  * • 扫描结果兼容INFINITY攻防协同平台（暂不公开）

## 🎉First Game

总结诸多实战经验，考虑到实战过程中会出现和存在复杂的环境、红蓝对抗过程中常用的内存加载无文件落地执行等，因此 **ServerScan** 设计了
**轻巧版** 、 **专业版** 、支持 **Cobalt Strike跨平台beacon: Cross
C2[2]的动态链接库**，**以及支持INFINITY攻防协同平台的专用版**。便于在不同的Shell环境中可以轻松自如地使用：如：Windows
Cmd、Linux Console、远控Console、WebShell等，以及Cobalt
Strike联动使用cna脚本文件加载，实现内网信息快速收集，为下一步横向移动铺路。

 **轻巧版：**

参数形式简单、扫描速度快、耗时短、文件体积小、适合在网络情况较好的条件情况下使用。

 **专业版：**

支持参数默认值、支持自定义扫描超时时长、支持扫描结果导出、适合在网络条件较苛刻的情况下使用。

 **动态链接库：**

为支持Cobalt Strike跨平台beacon，无文件落地执行，无文件执行的进程信息，基于轻巧版本进行动态链接库编译，扫描超时时长为1.5秒。

### 💻for Linux or Windows

  * •

#### 轻巧版

    * •  **for PortScan** **Usage：**

![](https://gitee.com/fuli009/images/raw/master/public/20230308202418.png)
Air_scan_use

 **Scanning：**

![](https://gitee.com/fuli009/images/raw/master/public/20230308202420.png)
Air_scan1

    * •  **for Service and Version Detection** **Usage：**

![](https://gitee.com/fuli009/images/raw/master/public/20230308202421.png)
Air_scan_probes_use

 **Scanning：**

![](https://gitee.com/fuli009/images/raw/master/public/20230308202422.png)
Air_scan_probes

  * •

#### 专业版

    * •  **for PortScan** **Usage：**

![](https://gitee.com/fuli009/images/raw/master/public/20230308202423.png)
Pro_scan_use

 **Scanning：**

![](https://gitee.com/fuli009/images/raw/master/public/20230308202426.png)
Pro_scan

    * •  **for Service and Version Detection** **Usage：**

![](https://gitee.com/fuli009/images/raw/master/public/20230308202427.png)
Pro_scan_probes_use

 **Scanning：**

![](https://gitee.com/fuli009/images/raw/master/public/20230308202428.png)
Pro_scan_probes

### 🎮for Cobalt Strike

  * •  **Windows** 于Cobalt Strike已经内置了PortScan，因此目前Windows仅支持利用cna上传对应版本的ServerScan可执行文件到服务器进行扫描。

    * •  **for Service and Version Detection** Interact:

![](https://gitee.com/fuli009/images/raw/master/public/20230308202430.png)serverscan_windows![](https://gitee.com/fuli009/images/raw/master/public/20230308202431.png)serverscan2_windows

  * •  **Cobalt Strike跨平台beacon** ServerScan的优势在于跨平台，在Hook师傅的帮（jiān）助（dū）下目前已经基本适配了Cross C2[3]的Linux、Mac OS两大平台，为了提高隐匿性减少文件特征，目前支持内存加载可执行程序和动态链接库调用，您只需在安装了Cross C2的Cobalt Strike中导入对应的.cna脚本，即可实现ServerScan与Cobalt Strike跨平台beacon联动，具体使用参考。

    * •  **for PortScan** Interact:

![](https://gitee.com/fuli009/images/raw/master/public/20230308202432.png)portscan_console

Targets结果集自动导入:

![](https://gitee.com/fuli009/images/raw/master/public/20230308202433.png)portscan_targets

services结果集自动导入:

![](https://gitee.com/fuli009/images/raw/master/public/20230308202434.png)portscan_services

    * •  **for Service and Version Detection** Interact:

![](https://gitee.com/fuli009/images/raw/master/public/20230308202435.png)serverscan_console

Targets结果集自动导入:

![](https://gitee.com/fuli009/images/raw/master/public/20230308202437.png)serverscan_targets

services结果集自动导入:

![](https://gitee.com/fuli009/images/raw/master/public/20230308202438.png)serverscan_services

  

  
 **后台回复“** **20230308** **" 获取该工具   **

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308202417.png)

技术交流

  

  
  

知识星球

  
  

  

 **致力于红蓝对抗，实战攻防，星球不定时更新内外网攻防渗透技巧，以及最新学习研究成果等。常态化更新最新安全动态。专题更新奇技淫巧小Tips及实战案例。  
**

 **涉及方向包括Web渗透、免杀绕过、内网攻防、代码审计、应急响应、云安全。星球中已发布 200+
安全资源，针对网络安全成员的普遍水平，并为星友提供了教程、工具、POC &EXP以及各种学习笔记等等。**

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308202440.png)

 **  
**

交流群

  
  

关注公众号回复“ **加群** ”，添加Z2OBot 小K自动拉你加入 **Z2O安全攻防交流群** 分享更多好东西。

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20230308202441.png)![](https://gitee.com/fuli009/images/raw/master/public/20230308202442.png)

  

  
  
  
  

关注我们

  
  

  

 **关注福利：**  

 **  
**

 **回复“** **app** **" 获取   app渗透和app抓包教程**

 **回复“** **渗透字典** **" 获取 针对一些字典重新划分处理，收集了几个密码管理字典生成器用来扩展更多字典的仓库。**

 **回复“** **书籍** **"  获取 网络安全相关经典书籍电子版pdf**

 ** **回复“ 资料** **"  获取 网络安全、渗透测试相关资料文档****

  

 ****

往期文章

  
  

[
**我是如何摸鱼到红队的**](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247489526&idx=1&sn=f52e17f5eb4790395a88a8e64c2a15e4&chksm=cea8fcb6f9df75a03fcf710a7369d8e761a1f8e5ae882709292c49103f22eefeb3a534388cd3&scene=21#wechat_redirect)  

[
**命令执行漏洞[无]回显[不]出网利用技巧**](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247488092&idx=1&sn=07ad5a0c09715ca03362bdcacc17f1e1&chksm=cea8f91cf9df700a68c747e2d7185efad50f380f93257293b9b96b5d87dc52cb2082560b0f0f&scene=21#wechat_redirect)  

[
**MSSQL提权全总结**](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247488864&idx=1&sn=4888dcec05c0c5c8bc9d3b34136930c0&chksm=cea8fe20f9df7736e03b8544955533ac32671b845ff61d24dc51b5d787960d28256a833350d0&scene=21#wechat_redirect)  

[ **Powershell 免杀过 defender
火绒，附自动化工具**](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247484469&idx=1&sn=bdac380ee95fd0ef72581a3b60da1443&chksm=cea8ef75f9df66630e2148be842428802b27bcee1cb09748cbc200046742b8052cb6f873e115&scene=21#wechat_redirect)  

[
**一篇文章带你学会容器逃逸**](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247486670&idx=1&sn=6822dd40dcc20121fce0c42188fcd49a&chksm=cea8e78ef9df6e987896bdbd6b7a96c48992782657680574d7015d72984e7ca67a25a0aeea5d&scene=21#wechat_redirect)  

[ **域渗透 |
kerberos认证及过程中产生的攻击**](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247483720&idx=1&sn=157c5a4c172315af343bd253bc66da7c&chksm=cea8ea08f9df631e96faca7437a2b1e1900973dfd3b8d3501a666afcd8b05d6d16b8642aabc7&scene=21#wechat_redirect)  

[
**通过DCERPC和ntlmssp获取Windows远程主机信息**](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247488097&idx=1&sn=cf1e6965c78c9b4636ba15d2bd752655&chksm=cea8f921f9df7037e1e54ae11e03e18562da87309414c0fac3d8bc581bebe69d96cc5fdfce1d&scene=21#wechat_redirect)

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

