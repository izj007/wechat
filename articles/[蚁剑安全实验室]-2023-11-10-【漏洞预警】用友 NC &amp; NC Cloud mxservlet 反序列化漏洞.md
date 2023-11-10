#  【漏洞预警】用友 NC & NC Cloud mxservlet 反序列化漏洞

原创 Permit  [ 蚁剑安全实验室 ](javascript:void\(0\);)

**蚁剑安全实验室** ![]()

微信号 AntSwordSec

功能介绍 专注于网络安全技术分享，漏洞情报、漏洞复现、渗透测试、src挖掘、应急响应、逆向、安全开发、红蓝对抗、内网横向等。

____

___发表于_

收录于合集

#漏洞情报 20 个

#漏洞预警 22 个

**免责声明：**
**该文章仅用于技术讨论与学习。请勿利用文章所提供的相关技术从事非法测试，若利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果均与文章作者及本账号无关。**

 **漏洞名称**

用友 NC & NC Cloud mxservlet 反序列化漏洞

  

 **漏洞评分**

8.8（高危）  

CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H/

 **漏洞详情**

在用友 NC 和 NC Cloud
系统中，由于系统未将用户传入的序列化数据进行过滤就直接执行了反序列化操作，并且结合系统本身存在的反序列化利用链，从而最终造成了命令执行的漏洞后果。

漏洞编号  
|  
| 漏洞类型  
| 反序列化漏洞  
---|---|---|---  
POC状态| 未知  
| 漏洞细节  
| 未知  
EXP状态  
| 未知| 在野利用  
| 未知  
  
  
  

 **影响版本**

NC56 / NC57 / NC63 / NC65 / NCC1903 / NCC1909 / NCC2005

  

 **安全版本**

NC56 补丁名称：NC56mxservlet反序列化补丁

NC57 补丁名称：57mxservlet反序列化补丁

NC63 补丁名称：63mxservlet反序列化补丁

NC65 补丁名称：65mxservlet反序列化补丁

NCC1903 补丁名称：1903mxservlet反序列化补丁

NCC1909 补丁名称：1909mxservlet反序列化补丁

NCC2005 补丁名称：MxServlet反序列化命令执行

 **修复建议**

 **  
**

更新用友  NC 和 NC Cloud 系统对应补丁，然后重启服务：

https://security.yonyou.com/#/patchInfo?foreignKey=8d74c5c5e62748c096a4493b52005ba0

 **参考链接**

https://security.yonyou.com/#/noticeInfo?id=429

  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

