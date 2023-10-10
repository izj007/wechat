#  准备熬夜加班？curl&libcurl 高危漏洞明日公布

流苏  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 中国网络安全行业门户

____

___发表于_

收录于合集 #深度观点 68个

![]()![]()

作者：流苏

排版：zhuo

  

近日，curl项目的作者bagder(Daniel
Stenberg)在GitHub中发布消息称，将在2023年10月11日发布curl的8.4.0版本。同时，他们还将公开两个漏洞：CVE-2023-38545和CVE-2023-38546。如下图所示：

  

![]()

图片来源于互联网

  

其中CVE-2023-38545是同时影响命令行工具 curl 和依赖库 libcurl 的高危漏洞，鉴于 curl&libcurl 使用量巨大，高危漏洞
CVE-2023-38545 又即将公开，提醒各位网安人：抓紧排查公司业务和产品使用上述两个库的情况，并做好升级准备。

  

有安全人员吐槽：这下又要开始熬夜加班了。

  
 **这可能是curl &libcurl很长时间内最严重漏洞**  
  

之所以在10月11日之前严格保密，是因为作者认为CVE-2023-38545漏洞的危险性极高，在 libcurl 官网首页也给了明显的提醒，如下图所示：

  

![]()

  

作者在发文时也表示，这个漏洞可能是curl&libcurl在相当长一段时间内最严重的漏洞，因此不能公布更多的细节，防止该漏洞被攻击者利用，唯一可以知道的就是“最近几年内发布的版本都会受到该漏洞影响”。目前在CVE官网，这两个漏洞也处于保密状态，只能看到漏洞编号，无法查询更详细的信息。

  

由于目前漏洞细节并未公开，建议可以提前排查使用到了curl/libcurl的业务，在相关漏洞细节公开后进一步根据具体的利用条件排查和修复。

  

Tanium 公司的端点安全研究总监 Melissa Bischoping 表示，curl 作为独立工具和其它软件的组成部分均得到广泛应用。curl
的广泛使用意味着组织机构应当趁此扩展其环境。虽然该漏洞可能并不影响所有的curl的版本，但鉴于该首席开发人员给出的提前通知，以及它可能具有的广泛影响，那么对于安全人员来说，即使最终并没有那么严重，但将其作为重大事件进行规划是稳妥做法。

  
 **curl是什么，为什么漏洞影响非常大？**  
  

根据公开信息，curl（客户端URL）是一个开放源代码的命令行工具，诞生于20世纪90年底末期，用于在服务器之间传输数据，并分发给几乎所有新的操作系统。curl编程用于需要通过Internet协议发送或接收数据的几乎任何地方。

  

curl支持几乎所有的互联网协议（DICT，FILE，FTP，FTPS，GOPHER，HTTP，HTTPS，IMAP，IMAPS，LDAP，LDAPS，MQTT，POP3，POP3S，RTMP，RTMPS，RTSP，SCP，SFTP，SMB，SMBS，SMTP
，SMTPS，TELNET和TFTP）。

  

换句话说，curl无处不在，可以隐藏在各种数据传输的设备中。

  

curl旨在通过互联网协议传输数据。其他所有内容均不在其范围内。它甚至不处理传输的数据，仅执行传输流程。curl可用于调试。例如使用“ curl -v
https://oxylabs.io ”可以显示一个连接请求的详细输出，包括用户代理，握手数据，端口等详细信息。

  

如果仅仅是curl存在漏洞，问题也许还没那么严重，关键是libcurl底层库同样受到该漏洞的影响。

  

事实上，libcurl
被广泛应用于各种软件和项目中，使得开发者能够在其应用程序中进行网络交互。即使没有直接引用，但也很可能使用了部分中间件，因此同样会受到影响。从这个角度来看，该漏洞的影响面会非常广，各位网安人可以加班干起来了。

  

【FreeBuf粉丝交流群招新啦！

在这里，拓宽网安边界  

甲方安全建设干货；

乙方最新技术理念；

全球最新的网络安全资讯；

群内不定期开启各种抽奖活动；

FreeBuf盲盒、大象公仔......

扫码添加小蜜蜂微信回复“加群”，申请加入群聊】

![]()

![]()  

![]()

[![]()](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247491351&idx=1&sn=07134e40b23213ca452fe119c2c769c6&chksm=ce1ce588f96b6c9ee6635f286d32a6234d4c0e54723b5cfb99d3e872fdf48d2cd13ee6c77e2f&scene=21#wechat_redirect)[![]()](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247491333&idx=1&sn=0344e1662d4fd7f4754e86ec4c36b156&chksm=ce1ce59af96b6c8c54441bf607ddf1a4bd215779aac8ea1151080b3bf135dcbb39b9cb1553b8&scene=21#wechat_redirect)[![]()](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247491295&idx=1&sn=3269d6d17c1efa20123d859ed8fc71fc&chksm=ce1ce440f96b6d561a7ab35f9dbdff4cdc50f8ba24ea64f5af972c5367d1556296fff44929fe&scene=21#wechat_redirect)  

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

