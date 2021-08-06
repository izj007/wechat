##  资产收集工具——HGather

[ 渗透Xiao白帽 ](javascript:void\(0\);)

**渗透Xiao白帽** ![]()

微信号 SuPejkj

功能介绍 积硅步以致千里，积怠惰以致深渊！

____

__

收录于话题

以下文章来源于白泽Sec ，作者白泽Sec_ahui

![白泽Sec](http://wx.qlogo.cn/mmhead/Q3auHgzwzM7gibNcIR9f4AKibySMAatib3FVgdJkvb6ksfiaVcPn3jE5ag/0)
**白泽Sec**

白泽Sec安全团队是以信息安全技术研究为目标，致力于分享高质量原创文章、开源安全工具、交流安全技术。研究方向覆盖网络攻防、web安全、移动终端、安全开发、物联网/工控安全/AI安全等多个领域，对安全感兴趣的小伙伴可以关注我们

01

前言  

![](https://gitee.com/fuli009/images/raw/master/public/20210806180350.png)  

  

  
  
工具名称：HGather  
工具作者：白泽Sec安全团队_ahui  
在渗透测试和漏洞挖掘过程中，信息收集是一个繁琐的过程，此工具基于FOFA，自动化获取符合指定条件下的资产并整理输出，并且包含资产所属IP的C段整理，增加敏感标题高亮提示，让资产收集高效化！  
  

  

  

02

项目地址  

![](https://gitee.com/fuli009/images/raw/master/public/20210806180350.png)

  

  
  
  
  

 **https://github.com/BaizeSec/HGather**  

  
  

  

  

03

工具使用方法  

![](https://gitee.com/fuli009/images/raw/master/public/20210806180350.png)

  

  

  
  
  
  

##### 使用环境：python2

  *   *   *   *   * 

    
    
      -h, --help            show this help message and exit  -q QBASE64, --qbase64 QBASE64                        指定FOFA中的搜索条件（base64编码）  -i IP, --ip IP        指定要搜索的IP或IP段，例如：-i 192.168.0.1/24  -o OUT, --out OUT     指定要导出的文件名，例如：-o test.txt

  

  

04

使用场景  

![](https://gitee.com/fuli009/images/raw/master/public/20210806180350.png)  

  

  
  
一、SRC漏洞挖掘  
前期通过搜索主域名和部分子域名获取属于SRC厂家的证书，或者直接通过cert="京东"的base64编码()查询  
python2 HGather.py -q Y2VydD0i5Lqs5LicIg==  
二、红队信息收集  
通过对目标IP段批量模糊查询找到敏感资产，自动高亮提示  
python2 HGather.py -i 192.168.0.1/24  
  

  

  

05

部分使用截图  

![](https://gitee.com/fuli009/images/raw/master/public/20210806180350.png)

  

  

  
  
  
  

 **资产发现**

![](https://gitee.com/fuli009/images/raw/master/public/20210806180353.png)

  

 **资产整理**  

![](https://gitee.com/fuli009/images/raw/master/public/20210806180354.png)

  

  
  

  

  

06

结束语  

![](https://gitee.com/fuli009/images/raw/master/public/20210806180350.png)

  

  
  
  
  

还望各位多提宝贵建议，祝师傅们站站日穿~ **  
**

  
  

 ** **【往期推荐】****  

[
【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS免杀-Shellcode
Loader原理(python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS免杀-
分离+混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细 | 钟馗之眼】ZoomEye-
python命令行的使用](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488453&idx=1&sn=5828a0e1a2299d3ee0215f0ed4c30bf1&chksm=ea37ec9fdd406589124c67c45487be39ed1033d88c627092cf07f6d4f14ccdb9079b38dba74d&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 |
Weblogic未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附PoC】CVE-2021-2109 | Weblogic
Server远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)

[【漏洞分析 | 附EXP】CVE-2021-21985 VMware vCenter Server
远程代码执行漏洞](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487906&idx=1&sn=e35998115108336f8b7c6679e16d1d0a&chksm=ea37eef8dd4067ee13470391ded0f1c8e269f01bcdee4273e9f57ca8924797447f72eb2656b2&scene=21#wechat_redirect)

[【CNVD-2021-30167 | 附PoC】用友NC
BeanShell远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487897&idx=1&sn=6ab1eb2c83f164ff65084f8ba015ad60&chksm=ea37eec3dd4067d56adcb89a27478f7dbbb83b5077af14e108eca0c82168ae53ce4d1fbffabf&scene=21#wechat_redirect)  

##
[【奇淫巧技】如何成为一个合格的“FOFA”工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)

[记一次HW实战笔记 |
艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange
远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

 _
**走过路过的大佬们留个关注再走呗**_![](https://gitee.com/fuli009/images/raw/master/public/20210806180357.png)

 **往期文章有彩蛋哦** **![]()**  

![](https://gitee.com/fuli009/images/raw/master/public/20210806180358.png)

一如既往的学习，一如既往的整理，一如即往的分享。![]()  

“ **如侵权请私聊公众号删文** ”

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

资产收集工具——HGather

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

