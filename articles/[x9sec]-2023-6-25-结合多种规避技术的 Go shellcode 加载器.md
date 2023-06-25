#  结合多种规避技术的 Go shellcode 加载器

upzhu  [ x9sec ](javascript:void\(0\);)

**x9sec** ![]()

微信号 x9sec_com

功能介绍 进行网络安全知识分享，渗透测试，红队蓝队技术分享，安全开发平台开源

____

___发表于_

收录于合集

  
**声明：**
该公众号大部分文章来自作者日常学习笔记，也有部分文章是经过作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本  
公众号无关。  
---  
  
  

  

 **项目简介**

  

 **Hades** 是一个概念验证加载器，它结合了几种规避技术，旨在绕过现代AV / EDR常用的防御机制。

  

 **项目描述**

##  

##  用法

最简单的方法可能是使用  `make`.

    
    
    git clone https://github.com/f1zm0/hades && cd hades  
    make

然后，您可以将可执行文件带到 x64 Windows 主机并运行它  `.\hades.exe [options]`.

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    PS > .\hades.exe -h  
      '||'  '||'     |     '||''|.   '||''''|   .|'''.|   ||    ||     |||     ||   ||   ||  .     ||..  '   ||''''||    |  ||    ||    ||  ||''|      ''|||.   ||    ||   .''''|.   ||    ||  ||       .     '||  .||.  .||. .|.  .||. .||...|'  .||.....| |'....|'  
              version: dev [11/01/23] :: @f1zm0  
    Usage:  hades -f <filepath> [-t selfthread|remotethread|queueuserapc]  
    Options:  -f, --file <str>        shellcode file path (.bin)  -t, --technique <str>   injection technique [selfthread,
    
    
     remotethread, queueuserapc]  
    

###

###  **例：**

注入生成的外壳代码  `calc.exe` 使用 队列用户APC 技术：

  * 

    
    
    .\hades.exe -f calc.bin -t queueuserapc

##

##  **展示**

具有系统调用 RVA 排序的用户模式挂钩旁路 （ `NtQueueApcThread` 与 弗里达跟踪 和 自定义处理程序 挂钩）

![](https://gitee.com/fuli009/images/raw/master/public/20230625200236.png)  

使用间接系统调用的检测回调旁路（注入的 DLL 来自  调用检测  的系统 jackullrich ）

![](https://gitee.com/fuli009/images/raw/master/public/20230625200237.png)  

##

##  **附加说明**

###

###  直接系统调用版本

在最新版本中，直接系统调用功能已被 acheron 提供的间接系统调用所取代
acheron。如果出于某种原因要使用使用直接系统调用的以前版本的加载程序，则需要显式传递  `direct_syscalls`
标记，编译器将确定需要从构建中包含和排除哪些文件。

  * 

    
    
    GOOS=windows GOARCH=amd64 go build -ldflags "-s -w" -tags='direct_syscalls' -o dist/hades_directsys.exe cmd/h
    
    
      
    

  

  

 ****

 **下载地址**

 **项目地址：https://github.com/AabyssZG/WebShell-Bypass-Guide**

 **  
**

 **点击最下方名片进入公众号**  

 **  
**

# **  往期推荐**  

[爬网站JS文件，自动fuzz
api接口，指定api接口](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485069&idx=1&sn=e94bad6d0f927441b7cb5c533d2e369d&chksm=fcedb22acb9a3b3cfb57e9d40f8898c51fa4c9fa3562bd112b5d489f9cba5cf81637d1558b53&scene=21#wechat_redirect)  

[生成各类免杀webshell适合市面上常见的几种webshell管理工具、冰蝎、蚁剑、哥斯拉](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485058&idx=1&sn=ac47be9a9d0c6aba0ec991e5213a903e&chksm=fcedb225cb9a3b3312a895ec7da3819216fde632546b644f943a6b31c3b6efa6db3624f6edf7&scene=21#wechat_redirect)  

[一款新的webshell管理工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485049&idx=1&sn=e02753cc60ea620882710bf9cf7b23b0&chksm=fcedb2decb9a3bc82b6dc1d9da9bc9c112ceedf608732c8d3338b04c4ffa5b44ebe28355272c&scene=21#wechat_redirect)  

[浏览器插件--
右键检测图片是否存在Exif漏洞](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485033&idx=1&sn=184c7486a3fb0f06b27885e881146561&chksm=fcedb2cecb9a3bd82564162a5a013261acad3b244ba67af527b211175701b5ebc617f731fafe&scene=21#wechat_redirect)  

[一个自动化通用爬虫
用于自动化获取网站的URL](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485003&idx=1&sn=914d8bcd37af1e43ea293b20a6b7976e&chksm=fcedb2eccb9a3bfa7174dfba94b4b85570ab818f875acbcef26163c173ae7c67c4c153f6d191&scene=21#wechat_redirect)  

[一个查询IP地理信息和CDN服务提供商的离线终端工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484994&idx=1&sn=55a0986b58d7bff15e8f55fd4b495204&chksm=fcedb2e5cb9a3bf3bc6d55f4e292fa43b4cb563012593b0f7832c040e0616f2aac53975dc31b&scene=21#wechat_redirect)  

[红队批量脆弱点搜集工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484965&idx=1&sn=70e48cdcb7683b227bc3f8d8efd05a6a&chksm=fcedb282cb9a3b942c93edec939bf9943a66d7345c40916e3c17a369ace2d0a9eed83ccf5ae8&scene=21#wechat_redirect)  

[Spring漏洞综合利用工具——Spring_All_Reachable](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484989&idx=1&sn=bfa97e5279e921e26631b9f93a75eefe&chksm=fcedb29acb9a3b8c178d47f04eaee53aabeb8e27b2c06e12d7f8a0916ea288b2a7ad94f4da3e&scene=21#wechat_redirect)  

[利用字符集编码绕过waf的burpsuite插件](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484894&idx=1&sn=33cec3b0baa230be290d7c62935d0dd5&chksm=fcedb179cb9a386f103c044a2fcb481e31efc4358c808b38c7d4cd961ec99cbd72ac80175b8a&scene=21#wechat_redirect)  

[云安全-
AK/SK泄露利用工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484850&idx=1&sn=785ebf6284c536262a08a6713cbcac8a&chksm=fcedb115cb9a38030cfe0024ecd722b3269d6eeddcd0845433f73ab68660ed776019b7a1f649&scene=21#wechat_redirect)  

[可以自定义规则的密码字典生成器](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484816&idx=1&sn=80a19335226b0e242a36a85486c541fc&chksm=fcedb137cb9a382100c0fbb51875de2a1183f46acccced953d4ee927c3f627d8d8e4a3f660d4&scene=21#wechat_redirect)  

[MySql、Oracle、金仓、达梦、神通等数据库、Redis等管理工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484787&idx=1&sn=766263e71666745fca6a1fe71b016170&chksm=fcedb1d4cb9a38c2cccc8c58afb34b31677974540d45ad9db5f032c0eba6c7ff26527c0dbd2c&scene=21#wechat_redirect)  

[JNDI/LDAP注入利用工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484743&idx=1&sn=42224dd27e62cb0d7a5cbdebf4dbbf19&chksm=fcedb1e0cb9a38f6b091c5a753c80b411bb92197ea02439a0493886ad55b7203fed057ec7718&scene=21#wechat_redirect)  

[jmx未授权访问 弱口令批量检测 GUI工具 -
jmxbfGUI](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484738&idx=1&sn=3d7cf4b2ec5bc5a92de3db9c57fe036e&chksm=fcedb1e5cb9a38f3e8d8fce41bfd4ac22d001c075bfe2f1425f1a90643ede8b93bce377fa787&scene=21#wechat_redirect)  

[基于Golang的高并发子域名收集工具——FindSubs](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484731&idx=1&sn=cedd1f26bfe4227ae8932c77163d6938&chksm=fcedb19ccb9a388a5a3a5cb2ce9dcbfe0cc19e48846c590014d2894707f4f10fbd37579575bb&scene=21#wechat_redirect)  

[用友NC反序列化漏洞payload生成工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484724&idx=1&sn=d78ecac847f9245ac0275968fec10fa7&chksm=fcedb193cb9a38851d51cc14f0a07a659bfed23a1faab5a326260241eed995fe7c0378c13c05&scene=21#wechat_redirect)  

[一种另辟蹊径的免杀执行系统命令的木马](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484697&idx=1&sn=5daa5b99580bffd7e3db48bebd8e4fa2&chksm=fcedb1becb9a38a841f3fe2f6c6cdbfed5728b9171abcb1eee7645d6659d59e3a5148e7a6b7d&scene=21#wechat_redirect)  

[高并发红队打点横向移动内网渗透扫描器](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484625&idx=1&sn=24e58636f61c0f4542ea0aac467f774f&chksm=fcedb076cb9a396043968b283c67962c72b195f11646afd66db8d4b799701d066528838b3c14&scene=21#wechat_redirect)  

[一款go
shellcode免杀加载器](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484617&idx=1&sn=1b67b44ccc3f1e8056ce9ac15d4d90e4&chksm=fcedb06ecb9a3978f20b354be309e0f4ebbbc6d34ed283e937e872ec6a92564e13110b483b61&scene=21#wechat_redirect)  

[目录扫描+403状态绕过工具——dirsearch_bypass403](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484589&idx=1&sn=76a011ac81b8751e29374283b7dc2f94&chksm=fcedb00acb9a391c83816dfb653ea260e15bf80341585917d1bd892081e618f4ab430d404390&scene=21#wechat_redirect)  

[跨平台空间资产测绘——superSearchPlus](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484580&idx=1&sn=3beacb4e755b075c6eaf1cf805887719&chksm=fcedb003cb9a3915567143f81f553865ed6a872972b921c62093d5faa9de98d631f534ac426f&scene=21#wechat_redirect)  

[免杀php木马](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484572&idx=1&sn=fcaefb5ae7793ad3576913b769a536a6&chksm=fcedb03bcb9a392dd11898b6f918b4a0daf23c414bff586ead9a7fc0d850f5e045aa025632f4&scene=21#wechat_redirect)  

  

  

加入微信群  

![](https://gitee.com/fuli009/images/raw/master/public/20230625200239.png)

  

  

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

