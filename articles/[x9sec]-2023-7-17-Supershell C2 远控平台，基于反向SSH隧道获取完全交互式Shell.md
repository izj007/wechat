#  Supershell C2 远控平台，基于反向SSH隧道获取完全交互式Shell

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

#### Supershell是一个通过WEB服务访问的C2远控平台，通过建立反向SSH隧道，获取完全交互式Shell，支持多平台架构Payload

##  

  

 **项目描述**

  

##  **功能特点**

  * 支持团队并发协作，一个浏览器使用所有功能

  * 支持多种系统架构的反弹Shell客户端Payload，集成压缩和免杀

  * 支持客户端断线自动重连

  * 支持全平台完全交互式Shell，支持在浏览器中使用Shell，支持分享Shell

  * 支持回连客户端列表管理

  * 内置文件服务器

  * 支持文件管理

  * 支持内存注入，即文件不落地执行木马（内存马）

  * 支持Windows安装反弹Shell服务和Linux尝试迁移uid与gid

##

##  **支持平台**

支持生成的客户端Payload系统架构：

 **android**|  **darwin**|  **dragonfly**|  **freebsd**|  **illumos**|
**linux**|  **netbsd**|  **openbsd**|  **solaris**|  **windows**  
---|---|---|---|---|---|---|---|---|---  
amd64| amd64| amd64| 386| amd64| 386| 386| 386| amd64| 386  
arm64| arm64|  
| amd64|  
| amd64| amd64| amd64|  
| amd64  
  
|  
|  
| arm|  
| arm| arm| arm|  
| dll  
  
|  
|  
| arm64|  
| arm64| arm64| arm64|  
|  
  
  
|  
|  
|  
|  
| ppc64le|  
| mips64|  
|  
  
  
|  
|  
|  
|  
| s390x|  
|  
|  
|  
  
  
|  
|  
|  
|  
| so|  
|  
|  
|  
  
  
 **其中以下系统架构不支持加壳压缩：**

  *   *   *   *   *   * 

    
    
     freebsd/*android/arm64linux/s390xlinux/sonetbsd/*openbsd/*

##

##  **架构图**

#

# ![]()

##

##  **快速构建**

 **1、下载最新release源码，解压后进入项目目录**

    
    
     wget https://github.com/tdragon6/Supershell/releases/download/latest/Supershell.tar.gz  
    
    
    
    tar -zxvf Supershell.tar.gz  
    

`cd Supershell  
`

 **2、修改配置文件config.py**  

其中登录密码`pwd`、jwt密钥`global_salt`、共享密码`share_pwd`必须修改，注意Python语法：`String`类型和`Int`类型，密码为明文密码的32位md5值

    
    
    # web登录和会话配置信息    
    user = 'tdragon6'    
    pwd = 'b7671f125bb2ed21d0476a00cfaa9ed6' # 明文密码 tdragon6 的md5    
        
    # jwt加密盐    
    global_salt = 'Be sure to modify this key' # 必须修改，不然可以伪造jwt token直接登录    
        
    # 会话保持时间，单位：小时    
    expire = 48    
        
        
    # 共享远控shell的共享密码    
    share_pwd = 'b7671f125bb2ed21d0476a00cfaa9ed6' # 明文密码 tdragon6 的md5    
        
    # 共享shell会话保持时间，单位：小时  
    

`share_expire = 24`  

 **3、确保8888和3232端口没有占用**  

（若占用，请修改docker-compose.yml文件nginx和rssh服务对外暴露端口），执行docker-compose命令

`docker-compose up -d  
`

 **4、访问管理平台，使用config.py配置的` user` / `pwd` 登录**

    
    
    http://公网IP:8888  
    

  

##  **部分功能演示**

  

声明：功能演示时的受害者主机采用谜团靶场，部署Supershell服务的VPS主机为临时申请使用，请不要尝试对演示中暴露的任何IP进行攻击，该VPS主机之前与之后的任何行为与本作者无关。

###

### 客户端生成

#

# ![]()  

  

  

 ****

 **下载地址**

 **项目地址：** https://github.com/tdragon6/Supershell

 **添加管理微信进内部交流群**  

 **  
**

![]()

 **  
**

# **  往期推荐**  

[WebSocket
内存马/Webshell，一种新型内存马/WebShell技术](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485218&idx=1&sn=253e80ffe166a1d6f2d49e4ce3a3d9aa&chksm=fcedb385cb9a3a93f2af164a433ea7450fbc42de90b1e155ed6002083def4686410395dc4d53&scene=21#wechat_redirect)  

[Chunsou（春蒐），Web指纹资产识别，风险收敛以及企业互联网资产风险摸查识别工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485204&idx=1&sn=6fb5b82803fe00de48be2400a9246149&chksm=fcedb3b3cb9a3aa5208f0e8c45a260cc482fb73ff40afa45f745a7eb45a389a48792e8c0518c&scene=21#wechat_redirect)  

[Nacos Hessian
反序列化漏洞利用工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485190&idx=1&sn=c53db63cfaf46560d3369806321bc275&chksm=fcedb3a1cb9a3ab754c81a351965d1551b591ec5626af76aa5673d2400a9171729de39f80e9e&scene=21#wechat_redirect)  

[myscan
集成多个框架的漏洞扫描工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485181&idx=1&sn=2f91540e572397e15627cddb2ba24c4d&chksm=fcedb25acb9a3b4c2f7ee781fbca46ee1385afe194581f801c8721c72911a51211ac7779d173&scene=21#wechat_redirect)  

[Burp的JS
API接口过滤插件](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485160&idx=1&sn=55fb551de62132f0aacaff2821b24eb2&chksm=fcedb24fcb9a3b59e8bebafb9189e912a5e844c4c176df200e4cd4459510a7f8580a8f8aaac5&scene=21#wechat_redirect)  

[通达OA检测工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485131&idx=1&sn=bccb90bc296d87908f073ac8f6c317a0&chksm=fcedb26ccb9a3b7aeeb7c6beec9ba5fe7af4168ff731dfe1429c2daaebe3425d888b7da1eb80&scene=21#wechat_redirect)  

[从零学习Webshell免杀手册](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485091&idx=1&sn=591916b62674003afd2f57bba780cef5&chksm=fcedb204cb9a3b1209e510ed72ad42984a6ef5b62736164bd5474ff28d81b99ae9f9368ad0e2&scene=21#wechat_redirect)  

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

