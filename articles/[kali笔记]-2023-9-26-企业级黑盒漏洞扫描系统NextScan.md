#  企业级黑盒漏洞扫描系统NextScan

原创 大表哥吆 [ kali笔记 ](javascript:void\(0\);)

**kali笔记** ![]()

微信号 bbskali-cn

功能介绍 发布关于kali相关文章。Debian Centos等操作系统的安全和运维。以及树莓派 ESP8266
DIY单片机等相关安全领域的文章。旨在掌握技术和原理的前提下，更好的保护自身网络安全。反对一切危害网络安全的行为，造成法律后果请自负。

____

___发表于_

收录于合集

#指纹识别 2 个

#信息收集 9 个

#kali工具 125 个

#基础教程 213 个

#web渗透测试 24 个

> 在前面的文章中，表哥讲到过很多漏洞扫描神器。本文为大家介绍一款企业级黑盒漏洞扫描系统`NextScan`(飞刃)。

# 关于

飞刃
是一套完整的企业级黑盒漏洞扫描系统，集成漏洞扫描、漏洞管理、扫描资产、爬虫等服务。拥有强大的漏洞检测引擎和丰富的插件库，覆盖多种漏洞类型和应用程序框架。

![]()

# 安装

`NextScan`的部署相对比较麻烦，所需的环境也比较多。需要mongo、redis等服务的支持。但是你也不用担心，我们直接用docker部署就行了。

    
    
    wget https://oss.17usoft.com/nextscan/download/v1.2.0/mongo.tar.gz  
    wget https://oss.17usoft.com/nextscan/download/v1.2.0/docker-compose.yaml  
    tar zxvf mongo.tar.gz  
    

 **文件说明：**

  * docker-compose.yaml ：配置文件
  * mongo：存放mongo启动和初始化文件 启动时确保两个文件在同一目录下

# 启动

    
    
    docker-compose up -d #启动  
    docker-compose stop #停止  
    docker logs 容器ID #查看日志  
    docker ps -a  #查看是否正常运行  
    

![]()启动完成后，访问http://你的宿主机ip (默认80端口)用户名：`admin`默认密码：`nextscan`

# 使用体验

 **创建项目**

![]() **配置插件**

![]() **漏洞管理**

![]() 点击详情可以看见漏洞详情、插件基本详情及处理历史。

![]() **资产管理**

记录扫描中出现过的所有url，包括爬虫和浏览器插件及代理收集到的url

![]()

![]() **节点详情**

显示cpu、内存使用率、任务空闲数等指标

![]() _更多详情，请移步官方文档：https://next-scan.ly.com/_

 **更多精彩文章 欢迎关注我们**

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

