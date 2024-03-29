#  已复现！泛微e-cology SQL注入漏洞

原创 微步情报局  [ 微步在线研究响应中心 ](javascript:void\(0\);)

**微步在线研究响应中心** ![]()

微信号 gh_280024a09930

功能介绍 微步情报局最新威胁事件分析、漏洞分析、安全研究成果共享，探究网络攻击的真相

____

___发表于_

收录于合集

#威胁通告 38 个

#漏洞 38 个

![](https://gitee.com/fuli009/images/raw/master/public/20230714180127.png)

01 漏洞概况 ** **  

  
泛微协同管理应用平台e-
cology是一套兼具企业信息门户、知识文档管理、工作流程管理、人力资源管理、客户关系管理、项目管理、财务管理、资产管理、供应链管理、数据中心功能的企业大型协同管理平台。  
  
近日，微步漏洞团队监测到泛微e-cology官方修复了一个SQL注入漏洞。经过分析和研判，该漏洞利用难度低，可导致远程命令执行或敏感数据泄露，建议尽快修复。
**  
**

02 漏洞处置优先级（VPT）  

  

 **综合处置优先级：** **高** ****

  

 **漏洞编号**

|

微步编号

|

XVE-2023-21310  
  
---|---|---  
  
 **漏洞评估**

|

危害评级

|

高危  
  
漏洞类型

|

SQL注入、RCE  
  
公开程度

|

PoC未公开  
  
利用条件

|

无权限要求  
  
交互要求

|

0-click  
  
威胁类型

|

远程  
  
 **利用情报**

|

在野利用

|

暂无  
  
漏洞活跃度

|

中  
  
 **影响产品**

|

产品名称

|

上海泛微网络科技股份有限公司-泛微e-cology  
  
受影响版本

| 1\. 部分e-cology 8且补丁版本<10.58.0 2\. 部分e-cology 9且补丁版本<10.58.0  
  
影响范围

|

万级  
  
有无修复补丁

|

有  
  
03 漏洞复现  

  
![](https://gitee.com/fuli009/images/raw/master/public/20230714180129.png)

###

###

04 修复方案  

  

 **1、官方修复方案**  

目前官方已发布安全补丁，建议受影响用户尽快将补丁版本升级至10.58及以上。  
https://www.weaver.com.cn/cs/securityDownload.asp#  

 **  
**

###

 **2、临时修复方案**

  * 使用防护类设备对相关资产进行防护，但是存在绕过风险，请尽快前往官网下载安装升级补丁。
  * 如非必要，避免将资产暴露在互联网。

###

###

05 微步在线产品侧支持情况  

  

微步在线威胁感知平台TDP通用SQL注入检测规则支持检测该漏洞。  

![](https://gitee.com/fuli009/images/raw/master/public/20230714180130.png)

###

###

06 时间线  

  

2023.07.10 微步获取该漏洞相关情报

2023.07.10 微步发布报告  

 **  
**

 **\---End---**  

  

 ** **微步漏洞情报订阅服务****

微步漏洞情报订阅服务是由微步漏洞团队面向企业推出的一项高级分析服务，致力于通过微步自有产品强大的高价值漏洞发现和收集能力以及微步核心的威胁情报能力，为企业提供0day漏洞预警、最新公开漏洞预警、漏洞分析及评估等漏洞相关情报，帮助企业应对最新0day/1day等漏洞威胁并确定漏洞修复优先级，快速收敛企业的攻击面，保障企业自身业务的正常运转。  

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

