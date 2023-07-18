#  JsProxy: 所到之处皆为代理节点

[ x9sec ](javascript:void\(0\);)

**x9sec** ![]()

微信号 x9sec_com

功能介绍 进行网络安全知识分享，渗透测试，红队蓝队技术分享，安全开发平台开源

____

___发表于_

收录于合集

编者荐语：

这是一个利用浏览器当代理的demo项目,让所有访问者的浏览器成为自己的代理池，所到之处皆为代理节点.

以下文章来源于Medi0cr1ty ，作者duckbubi

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM5fticjUaliaiaymrMMkEcpiaSKTtT2MsDPw1JV2OZydjcRQA/0)
**Medi0cr1ty** .

宁静在遥远处波动

**01**

项目简介

这是一个利用浏览器当代理的 demo 项目，让所有访问者的浏览器成为自己的代理池，所到之处皆为代理节点。

项目使用了以下技术栈：  

ServiceWorker + Go WebAssembly + WebSocket + Http Proxy

  

项目主要分为两个部分：

1\. 客户端：用 sw 将 wasm 程序驻留在浏览器，然后通过 ws 与服务端建立联系，执行完服务端发送的请求后传给服务端做进一步处理。

  

2\. 服务端：监听了两个端口，一个是 http 代理端口，一个是 ws 端口， http 代理端口收到请求信息后通过 ws 传给访问者浏览器的 wasm
程序来处理。

 **02**

使用说明

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # 编译wasmgit clone https://github.com/TheKingOfDuck/jsproxy.gitcd jsproxy#修改第82行中的localhost为自己的ipnano client/agent.go./build.sh# 启动http servercd servergo mod tidygo run httpserver.go# 启动主程序go run ws.go

![]()

  

![]()

  

 **03**

使用场景

水坑漏洞保护、XSS 深度利用等等。

  

 **04**

已知弊端

1\. 支持不了 socks5 ，因为浏览器不支持发送 tcp 包。

2\. 这只是随手写的 demo ，很多东西实战没有考虑进去。

  

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

