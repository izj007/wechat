#  记一次峰回路转Getshell

[ 极梦C ](javascript:void\(0\);)

**极梦C** ![]()

微信号 gh_2353880ae4d9

功能介绍 只专注于实战的实战派。

____

___发表于_

收录于合集

以下文章来源于琴音安全 ，作者琴音安全

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM6DbRDNwxUrWRkZmN78R3ia6hqHvFxHYMiawibEQUAyKUHEA/0)
**琴音安全** .

致力于红蓝对抗，渗透测试，网络安全技术分享，CTF 研究等。

**免责声明  
**

由于传播、利用本公众号琴音安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号琴音安全及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉，谢谢！  
---  
  
 **0x01正文**

 **信息收集**

拿到目标为xxx.example.com

子域名收集一波,在进行存活探测.

子域名扫描工具,推荐使用网络空间搜索引擎,如shodan,fofa,鹰图,或者子域名挖掘机,OneForAll,subfinder等工具.

存活探测推荐工具httpx

下载链接:

  * 

    
    
    https://github.com/projectdiscovery/httpx

![]()

端口扫描无果  

发现存在有两个站点存活,A.xxx.com和B.xxx.com

我们先对A.example.com进行渗透

在A.example.com一打开还是大家熟悉的登页面,还好验证一次

![]()

尝试用户名admin,test,root页面均提示用户名不存在,想着先爆破一下用户名,掏出祖传字典直接怼,意外发生了.响应包中居然有SQL语句报错信息.

![]()

看到有注入存在想着直接上sqlmap --os-shell 叽叽叽shell失败

MySQL数据库拿shell必备条件:

  *   *   *   * 

    
    
    数据库当前用户为root权限；知道当前网站的绝对路径；PHP的GPC为 off状态；(魔术引号，GET，POST，Cookie)写入的那个路径存在写入权限。

‍转眼把目光放在B.example.com上,通过前面的信息收集发现是.net程序  

![]()

对于这种有个小技巧,可以尝试在URL后面拼接

  *   *   * 

    
    
    admin/login.aspxadmim/top.aspxadmin/main.aspx

踩狗屎运直接一个admin未授权到后台,大起大落功能都不能使用

![]()

尝试访问admin/top.aspx路径页面直接跳转到后台登录处

![]()尝试弱口令,SQL注入,修改响应包中数据均不成功,难搞

 **0x02峰回路转**

本来到这里已经没心态了，本想就此收手，突然想起来还有js文件没查看，于是便使用了F12大法,功夫不负有心人,让我找到一个接口

![]()上传txt成功这里也可以尝试截断绕过  

  * 

    
    
     ：，；、%00、’、^

主要应用于Windows上,在创建文件时不允许出现特殊字符.

![]()

直接上传成功,访问提示404,应该被杀掉了,做了一下免杀处理shell成功

![]()

到此点到为止,收工提交报告

 **0x03小结**

推荐两个收集JS敏感信息常用的工具

  *   * 

    
    
    https://github.com/hakluke/hakrawlerhttps://github.com/p1g3/JSINFO-SCAN

浏览器插件：  

 **FindSomething**

直接下载即可 ****

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

  
  

![]()

  

![]()

  

![]()

  

  
  

![]()

  

  

![]()

  

  

  
  
  
  
  

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

