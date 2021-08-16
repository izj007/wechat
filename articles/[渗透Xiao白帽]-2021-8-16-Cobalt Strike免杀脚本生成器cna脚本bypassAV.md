##  Cobalt Strike免杀脚本生成器|cna脚本|bypassAV

原创 雨苁  [ 渗透Xiao白帽 ](javascript:void\(0\);)

**渗透Xiao白帽** ![]()

微信号 SuPejkj

功能介绍 积硅步以致千里，积怠惰以致深渊！

____

__

收录于话题

#远控免杀

1个

目录导航

使用方法

  * 参考文章

  * cna脚本下载地址

  * 杀毒软件绕过效果检测

    * ①bypass火绒效果

    * ② bypass 卡巴斯基效果

  * 注意事项:  

 **仅用于技术交流，请勿用于非法用途。**

该插件没有什么技术含量 **，本质上利用的ps2exe.ps1脚本编译为exe**
，只是不想在命令行里操作，将其写为cna脚本，方便直接快速生成免杀的可执行文件且只有50KB， **目前支持exe、ps1文件格式** 。

 _注：建议在powershell 4.0版本以上机器安装，可向下兼容powershell 2.0。_

## 使用方法

 **在导入cna脚本之前，只需要修改当前路径$path为powershell_bypass.cna所在的真实路径即可。**

 _注意：均是两个斜杠_

![](https://gitee.com/fuli009/images/raw/master/public/20210816082823.png)

选择Cobalt Strike生成BIN文件。

![](https://gitee.com/fuli009/images/raw/master/public/20210816082833.png)

启用该cna脚本，选择指定的bin文件，点击生成恶意的ps1文件、exe可执行文件，

![](https://gitee.com/fuli009/images/raw/master/public/20210816082834.png)![](https://gitee.com/fuli009/images/raw/master/public/20210816082835.png)![]()

点击即可上线。

使用powershell 4.0上线server 2012

![](https://gitee.com/fuli009/images/raw/master/public/20210816082836.png)

使用powershell 2.0上线server 2008

![](https://gitee.com/fuli009/images/raw/master/public/20210816082837.png)

如果在webshell触发该可执行文件，需要start命令

![]()

更新日志2021/7/18

![](https://gitee.com/fuli009/images/raw/master/public/20210816082838.png)

## 参考文章

https://www.jianshu.com/p/fb078a99e0d8

https://www.jianshu.com/p/f158a9d6bdcf

## cna脚本下载地址

①GitHub: github.com/cseroad/bypassAV.zip

②云中转网盘:

bypassAV_www.ddosi.org.rar

## 杀毒软件绕过效果检测

### ①bypass火绒效果

生成木马以及运行木马全程火绒无反应.

![]()静态查杀未检测到病毒![](https://gitee.com/fuli009/images/raw/master/public/20210816082839.png)运行上线火绒未报毒![](https://gitee.com/fuli009/images/raw/master/public/20210816082840.png)读取文件火绒未报毒

### ② bypass 卡巴斯基效果

![]()静态查杀卡巴斯基未检测到病毒![](https://gitee.com/fuli009/images/raw/master/public/20210816082841.png)成功上线-
卡巴斯基未拦截

读取文件-卡巴斯基未拦截

![](https://gitee.com/fuli009/images/raw/master/public/20210816082842.png)

执行命令,卡巴斯基拦截并清除木马,同时锁定所有软件,进行木马扫描.

![]()

## 注意事项:

①导入脚本前请务必修改路径,否则无法生成木马.

![](https://gitee.com/fuli009/images/raw/master/public/20210816082843.png)

②ico图标必填,否则无法生成木马(报错)

这里选择的ico图标为卡巴斯基臭狗熊头像.

![](https://gitee.com/fuli009/images/raw/master/public/20210816082844.png)

③乱码问题

这个影响不大

![](https://gitee.com/fuli009/images/raw/master/public/20210816082845.png)

 _ ** **BY：雨苁  
****_

 _ ** **原文地址：****_

  * 

    
    
     https://www.ddosi.org/cs-bypass-av/

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

[【超详细 | 附EXP】Weblogic CVE-2021-2394
RCE漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488922&idx=1&sn=f43e3c243bbbfd2822867a3acaa8b85e&chksm=ea37eac0dd4063d63d98f935c73ce571cbfeb0e7272a6f171a28143bdb3e7134b09ea874969a&scene=21#wechat_redirect)

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

 _ **走过路过的大佬们留个关注再走呗**_![]()

 **往期文章有彩蛋哦**
**![](https://gitee.com/fuli009/images/raw/master/public/20210816082846.png)**  

![]()

一如既往的学习，一如既往的整理，一如即往的分享。![](https://gitee.com/fuli009/images/raw/master/public/20210816082847.png)  

“ **如侵权请私聊公众号删文** ”

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

Cobalt Strike免杀脚本生成器|cna脚本|bypassAV

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

