#  Burp Collaborator的奇淫技巧

原创 瓦都尅 [ 小宝的安全学习笔记 ](javascript:void\(0\);)

**小宝的安全学习笔记** ![]()

微信号 blackTechOfBaby

功能介绍 记录并分享自己学习的安全知识和总结思考

____

___发表于_

收录于合集

#SRC 20 个

#WAF 3 个

#Dnslog 1 个

## 0x00 前言

测试的时候总是被`WAF`或办公安全产品拦，网上很多要注册/登录，所以就想着自己搭建一个，方便使用。部署的时候发现一个很奇妙的技巧。

## 0x01 发现过程

部署好后，直接`curl`测试的时候发现了`header`的特征

    
    
    HTTP/1.1 200 OK  
    Server: Burp Collaborator https://burpcollaborator.net/  
    X-Collaborator-Version: 4  
    Content-Type: text/html  
    Content-Length: 55  
    

通用特征？

直接去`Fofa`搜了下，发现很多，随便找了个就能直接使用。

1、配置 `Burp Collaborator Server`

![]()

点击 `Run health check` 检查下

![]()

若通过，即可愉快的白嫖了

![]()

随便找了几个，都能使用，太香了～

 **注意：使用过程中请一定要注意脱敏和数据安全！**

##  0x03 FoFa指纹

    
    
    server="Burp Collaborator https://burpcollaborator.net/"  
    

技巧：可以直接找域名，也可以加个`https`协议，然后在证书里面找域名。

## 0x04 思考

为什么DNS解析啥的什么都不在我这，我直接配置下就能获取到这个域名的DNS等记录呢？

想了下，我唯一能想的通的解释就是：`Burp Collaborator` 为了能够在页面进行展示，会要求 `Server` 将相关记录
按照某种格式转到某个地方，然后 `Client` 去这个地方取数据，对数据进行格式化后展示。

若有大佬研究过，可以告知下小弟。

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

