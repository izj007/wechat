#  SBSCAN -Spring一套带走

原创 sule01u  [ 不懂安全 ](javascript:void\(0\);)

**不懂安全** ![]()

微信号 the-avengers-5

功能介绍 周更！分享漏洞复现，挖洞技巧，面试题解。主打就是一个消除模糊，只有消除模糊我们才能理解并记住！

____

___发表于_

收录于合集 #分享 5个

# SBSCAN横空出世 - 专打spring各种不服

## 一、SBSCAN

#### 前情提要：

日常渗透过程中我们经常会遇到spring boot框架，通过资产测绘工具搜索我们也可以知道spring
boot的资产非常多，最常见的需求就是我想测试是否存在敏感信息泄漏以及测试是否存在spring的相关漏洞，不想东拼西凑找工具，不如写一个吧！

#### 这个工具能干啥：

  * • 用于检测站点是否存在Spring Boot的敏感信息泄漏

  * • 用于检测站点是否存在Spring相关的漏洞

## 二、认真的介绍挨打对象

### 1、Spring Boot 是啥？

Spring Boot
**是一个基于Spring的套件，它帮我们预组装了Spring的一系列组件，以便以尽可能少的代码和配置来开发基于Spring的Java应用程序**
。以汽车为例，如果我们想组装一辆汽车，我们需要发动机、传动、轮胎、底盘、外壳、座椅、内饰等各种部件，然后把它们装配起来。

### 2、Spring Boot Actuator又是啥？

一句话说它的故事：`Spring Boot Actuator` 提供的 endpoint 会将应用重要的信息暴露，它知道spring boot的所有秘密。

### 3、识别Spring Boot框架

微步X社区资产测绘页面导航

    
    
    # x社区资产测绘语法  
    icon_hash=0488faca4c19046b94d07c3ee83cf9d6 || body="Whitelabel Error Page"

![]()image-20231006105839906

## 三、Spring Boot框架敏感信息泄露

> 渗透测试过程中，如果遇到spring boot, 必须走一波敏感信息泄露和未授权访问相关的漏洞，真的很常见

 **版本区别：**

`Spring Boot` < 1.5：默认未授权访问所有端点

`Spring Boot` >= 1.5：默认只允许访问 `/health` 和 `/info` 端点，但是此安全性通常被应用程序开发人员禁用了

#### 常见的敏感信息泄漏端点

路径| 描述| 默认启用  
---|---|---  
auditevents| 显示当前应用程序的审计事件信息| Yes  
beans| 显示一个应用中所有Spring Beans的完整列表| Yes  
conditions| 显示配置类和自动配置类的状态及它们被应用或未被应用的原因| Yes  
configprops| 显示一个所有@ConfigurationProperties的集合列表| Yes  
env| 显示来自Spring的 ConfigurableEnvironment的属性| Yes  
flyway| 显示数据库迁移路径，如果有的话| Yes  
health| 显示应用的健康信息| Yes  
info| 显示任意的应用信息| Yes  
liquibase| 展示任何Liquibase数据库迁移路径，如果有的话| Yes  
metrics| 展示当前应用的metrics信息| Yes  
mappings| 显示一个所有@RequestMapping路径的集合列表| Yes  
scheduledtasks| 显示应用程序中的计划任务| Yes  
sessions| 允许从Spring会话支持的会话存储中检索和删除| No  
shutdown| 允许应用以优雅的方式关闭| No  
threaddump| 执行一个线程dump| Yes  
heapdump| 返回一个GZip压缩的hprof堆dump文件| Yes  
jolokia| 通过HTTP暴露JMX beans| Yes  
logfile| 返回日志文件内容| Yes  
prometheus| 以可以被Prometheus服务器抓取的格式显示metrics信息| Yes  
  
#### 潜在的漏洞点

  * • `/env`、`/actuator/env`：GET 请求 `/env` 会直接泄露环境变量、内网地址、配置中的用户名等信息；当程序员的属性名命名不规范，例如 password 写成 psasword、pwd 时，会泄露密码明文；同时有一定概率可以通过 POST 请求 `/env` 接口设置一些属性，间接触发相关 RCE 漏洞；同时有概率获得星号遮掩的密码、密钥等重要隐私信息的明文。

  * • `/refresh`、`/actuator/refresh`：POST 请求 `/env` 接口设置属性后，可同时配合 POST 请求 `/refresh` 接口刷新属性变量来触发相关 RCE 漏洞。

  * • `/restart`、`/actuator/restart`：暴露出此接口的情况较少；可以配合 POST请求 `/env` 接口设置属性后，再 POST 请求 `/restart` 接口重启应用来触发相关 RCE 漏洞。

  * • `/jolokia`、`/actuator/jolokia`：可以通过 `/jolokia/list` 接口寻找可以利用的 MBean，间接触发相关 RCE 漏洞、获得星号遮掩的重要隐私信息的明文等。

  * • `/trace`、`/actuator/httptrace`：一些 http 请求包访问跟踪信息，有可能在其中发现内网应用系统的一些请求信息详情；以及有效用户或管理员的 cookie、jwt token 等信息。

## 四、Spring相关的历史漏洞

CVE编号| CVE描述  
---|---  
CVE-2016-4977| Spring Security OAuth2 远程命令执行漏洞  
CVE-2017-4971| Spring WebFlow 远程代码执行漏洞  
CVE-2017-8046| Spring Data REST命令执行漏洞  
CVE-2018-1270| Spring-Messaging-远程命令执行漏洞  
CVE-2018-1273| Spring Data Commons 远程命令执行漏洞  
CVE-2022-22947| Spring Cloud Gateway 远程代码执行漏洞  
CVE-2022-22963| Spring Cloud Function SpEL 远程代码执行漏洞  
CVE-2022-22965| Spring远程命令执行漏洞(Spring4Shell)  
CVE-2022-22978| Spring-security 认证绕过漏洞  
  
## 五、SBSCAN帮你一键测试

![]()

image-20231006013436933

### ✈️ 工具概述

  * • 用于检测站点是否存在Spring Boot的敏感信息泄漏

  * • 用于检测站点是否存在Spring相关的漏洞

### 🏂 如何使用

    
    
    $ git clone https://github.com/sule01u/SBSCAN.git  
    $ cd SBSCAN  
    $ pip3 install -r requirements.txt  
    $ python3 sbscan.py --help

### 🪄使用效果

> 支持多线程扫描以及指定代理

> ![]()sbsscan

### 🎡 Options

    
    
          -u, --url TEXT     对单个URL进行扫描  
          -f, --file TEXT    读取文件中的url目标进行扫描  
          -p, --proxy TEXT   指定HTTP代理  
          --threads INTEGER  指定线程数量  
          --help             显示帮助信息  
    

### 🎨 使用示例

    
    
    $ python3 sbscan.py -u http://test.com  
    $ python3 sbscan.py -f url.txt  
    $ python3 sbscan.py -u http://test.com -p 1.1.1.1:8888 --threads 10

### 🧾TODO

  * •  补充进spring框架相关的其他CVE POC

  * •  增加指纹识别，减少无效目标扫描

  * •  增加每个敏感路径响应页面的关键词匹配，提升检出精准度

### 🧭工具地址：

https://github.com/sule01u/SBSCAN

 **来了就点个🌟再走吧，欢迎所有师傅PR**

 **  
**

 **欢迎关注不懂安全⬇️**

 **每一个你想学习的瞬间都是未来的你在向现在的你求救**

PS:请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。渗透的第一步是啥？拿授权拿授权拿授权！

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

