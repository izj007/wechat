#  漏洞预警 | Linux Kernel权限提升漏洞

浅安  [ 浅安安全 ](javascript:void\(0\);)

**浅安安全** ![]()

微信号 gh_758e256fcc72

功能介绍 网络安全学习、漏洞预警、工具分享

____

___发表于_

收录于合集 #漏洞预警 153个

**0x00 漏洞编号**

  * # CVE-2023-1829

 **0x01 危险等级**

  * 高危  

 **0x02 漏洞概述**

Linux是一种开源电脑操作系统内核。它是一个用C语言写成，符合POSIX标准的类Unix操作系统。

![](https://gitee.com/fuli009/images/raw/master/public/20230622092807.png)

 **0x03 漏洞详情**

###

###  ****

 **CVE-2023-1829** **漏洞类型：** 权限提升 **** **影响：** 用户权限提升 **简述：**
Linux内核流量控制索引过滤器中存在释放后使用漏洞，由于tcindex_delete
函数在某些情况下无法正确停用过滤器，同时删除底层结构，可能会导致双重释放该结构，本地低权限用户可利用该漏洞将其权限提升为root。

###

 **0x04 影响版本**

  * 2.6.12-rc2 <= Kernel < 6.3

 **0x05 POC**  

 ****

 **https://github.com/lanleft/CVE2023-1829** ****

 ** ** **仅供安全研究与学习之用，若将工具做其他用途，由使用者承担全部法律及连带责任，作者及发布****** ** ** ** ** **
**者************ **不承担任何法律及连带责任。** ****

 **0x06  ** **修复建议**

 ** ** ** **目前官方已发布漏洞修复版本，建议用户升级到安全版本******** ** ** ** **：********
https://www.kernel.org/

  

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

