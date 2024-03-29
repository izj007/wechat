#  【复现】泛微 e-cology 前台文件上传漏洞风险通告

原创 赛博昆仑CERT  [ 赛博昆仑CERT ](javascript:void\(0\);)

**赛博昆仑CERT** ![]()

微信号 gh_9ec1e14521c3

功能介绍 快速响应高危漏洞和安全事件，并为客户提供闭环的解决方案。

____

___发表于_

收录于合集 #赛博昆仑风险通告 57个

![]()

  

-赛博昆仑漏洞安全通告-

泛微 e-cology 前台文件上传漏洞风险通告![]()  
  
  

 **  
**

 **漏洞描述**

泛微协同管理应用平台(e-cology)是一套兼具企业信息门户、知识文档管理、工作流程管理、人力资源管理、客户关系管理、项目管理、财务管理、资产管理、供应链管理、数据中心功能的企业大型协同管理平台，形成了一系列的通用解决方案和行业解决方案。近日，赛博昆仑CERT监测到泛微官方发布了10.58.3补丁版本，未经授权的远程攻击者可通过发送特殊的HTTP请求来触发文件上传，最终可导致攻击者获取远程服务器权限。

 **漏洞名称**

|

泛微 e-cology前台文件上传漏洞  
  
---|---  
  
 **漏洞公开编号**

|

暂无  
  
 **昆仑漏洞库编号**

|

CYKL-2023-011493  
  
 **漏洞类型**

|

文件上传

|

 **公开时间**

|

2023-7-25  
  
 **漏洞等级**

|

高危

|

 **评分**

|

暂无  
  
 **漏洞所需权限**

|

无权限要求

|

 **漏洞利用难度**

|

低  
  
 **PoC** **状态**

|

未知

|

 **EXP** **状态**

|

未知  
  
 **漏洞细节**

|

未知

|

 **在野利用**

|

未知  
  
 ** **影响版本****

e-cology9 并且 补丁版本 < 10.58.3

e-cology8 并且 补丁版本 < 10.58.3

 ** **漏洞利用条件****

集群环境启动

 ** **漏洞复现****

赛博昆仑CERT已复现该漏洞。

![]()

#  **产品侧解决方案**

  *  **赛博昆仑洞见平台**

赛博昆仑-
洞见平台以风险运营为核心思想，结合资产、漏洞和威胁进行风险量化与风险排序，并在不中断业务运行的前提下完成威胁阻断和漏洞修复，从而实现实时的风险消除。

  *  **资产风险规则**

目前赛博昆仑-洞见平台的资产风险检测模块已加入该漏洞的检测规则并提示对应风险：

![]()

 **防护措施**

  *  **修复建议**

官方已发布修复建议，建议受影响的用户尽快升级至最新版本的补丁
v10.58.3。下载地址：https://www.weaver.com.cn/cs/securityDownload.asp#

![]()

  *  **技术业务咨询**

赛博昆仑支持对用户提供轻量级的检测规则或热补方式，可提供定制化服务适配多种产品及规则，帮助用户进行漏洞检测和修复。

赛博昆仑CERT已开启年订阅服务，付费客户(可申请试用)将获取更多技术详情，并支持适配客户的需求。

联系邮箱：cert@cyberkl.com公众号：赛博昆仑CERT

 ** **参考链接****

https://www.weaver.com.cn/cs/securityDownload.html#

 ** **时间线**** 2023年7月25日，泛微官方发布安全补丁 2023年7月26日，赛博昆仑CERT公众号发布漏洞风险通告

  

  

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

