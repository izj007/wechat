#  TongWeb历史补丁分析及利用方法

原创 LambdaTeam [ Lambda小队 ](javascript:void\(0\);)

**Lambda小队** ![]()

微信号 LambdaTeam

功能介绍 一群热爱网络安全的热血青年，一支做好事不留名的Lambda小队

____

___发表于_

收录于合集

#红队实战 4 个

#代码审计 8 个

#漏洞复现 5 个

  

_**免责声明：**_

 __
_本公众号致力于安全研究和红队攻防技术分享等内容，本文中所有涉及的内容均不针对任何厂商或个人，同时由于传播、利用本公众号所发布的技术或工具造成的任何直接或者间接的后果及损失，均由使用者本人承担。请遵守中华人民共和国相关法律法规，切勿利用本公众号发布的技术或工具从事违法犯罪活动。最后，文中提及的图文若无意间导致了侵权问题，请在公众号后台私信联系作者，进行删除操作。_

  
  
 **0x00 简介**  

应用服务器 TongWeb v7 全面支持 JavaEE7 及 JavaEE8
规范，作为基础架构软件，位于操作系统与应用之间，帮助企业将业务应用集成在一个基础平台上，为应用高效、稳定、安全运行提供关键支撑，包括便捷的开发、随需应变的灵活部署、丰富的运行时监视、高效的管理等。  

 **0x01  架构**  

参考[1]，TongWeb7 提供三种管理工具，分别是管理控制台和第三方 JMX 工具 JConsole 还有命
令行。管理控制台和命令行提供应用组件和资源的管理等功能，JConsole是基于 JMX 的 GUI 工具，提供 JVM、MBeans 等信息。

![]()

TongWeb7应用服务器目录结构说明:

目录名称

|

说明  
  
---|---  
  
Agent

|

代理服务器注册节点（标准版不存在该目录）  
  
apache-activemq

|

activemq 所在目录  
  
applications

|

系统应用所在目录  
  
asdp

|

asdp 防御攻击目录（安全版本提供）  
  
autodeploy

|

服务器默认提供的自动部署监听目录  
  
bīn

|

服务器启动，停止等脚本文件所在目录  
  
conf

|

服务器的配置文件所在目录  
  
deployment

|

已部署应用的应用程序目录  
  
domain_template

|

TongWeb 域服务器模板  
  
TongDataGrid

|

分布式缓存目录(标准版不存在该目录)  
  
lib

|

服务器运行所需的类文件所在目录，主要以Jar形式存在  
  
logs

|

服务器存放日志文件的目录，日志文件包括访问日志文件和服 务器日志文件等  
  
Agent

|

代理服务器注册节点（标准版不存在该目录）  
  
native

|

Apr native 在不同平台所需要的库文件  
  
samples

|

应用服务器的示例目录，示例包括 EJB、WEB 等模块  
  
service

|

自启动服务器目录  
  
persistence

|

存放各类监视量的持久化文件  
  
snapshot

|

存放服务器生成的快照文件  
  
temp

|

服务器产生的临时文件以及应用预编译文件所在的目录  
  
tools

|

应用服务器所带的工具目录  
  
  

 **0x02  补丁历史**  

  * SR-001  

##

TongWeb7041至7045产品补丁升级说明

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    升级包提交日期：2022.06.09  
    升级包名称：TongWeb7041_LD5_001.zipTongWeb7042_LD5_002.zipTongWeb7043_LD5_003.zipTongWeb7044_LD5_004.zipTongWeb7045_LD5_005.zip  
    产品：TongWeb7  
    版本号：TongWeb7041TongWeb7042TongWeb7043TongWeb7044TongWeb7045  
    补丁号：TongWeb7041_LD5_001TongWeb7042_LD5_002TongWeb7043_LD5_003TongWeb7044_LD5_004TongWeb7045_LD5_005  
    补丁说明：TongWeb7041_LD5_001补丁适用于TongWeb7.0.4.1TongWeb7042_LD5_002补丁适用于TongWeb7.0.4.2TongWeb7043_LD5_003补丁适用于TongWeb7.0.4.3TongWeb7044_LD5_004补丁适用于TongWeb7.0.4.4TongWeb7045_LD5_005补丁适用于TongWeb7.0.4.5  
    升级包解决的问题：1.修复Log任意下载漏洞；2.修复管理控制台未授权访问漏洞；3.修复命令执行漏洞；4.修复console存在命令行执行漏洞；5.修复任意文件删除漏洞；6.修复访问日志-任意文件写入&路径穿越漏洞；

 **0x03  漏洞分析**  

漏洞细节已发小密圈，扫描文末二维码加入

  

  *  **Log任意下载**  

##

  

Diff漏洞版本和补丁发现在Logfinder#downloadLog方法新增了校验，判断文件名必须以.log结尾且包含server字符串。

![]()

该方法的调用位于LogShowController#downloadLog，接收前端传递的names参数传入方法的调用。

![]()

#### 漏洞利用

由于在读取文件前会判断路径和原有的日志路径是否一致，因此只能读取日志目录下的文件（/data/TongWeb{version}/logs/），返回的是一个压缩包文件。

![]()

  

  * ###  **未授权访问漏洞**

Diff漏洞版本和补丁发现在SecurityController#saveUser方法新增了鉴权和授权的判断，猜测该方法可能存在未授权调用的地方可添加用户。

![]()

  * ###  **命令执行漏洞**

实际上这是一个Spring框架的HttpInvokerServiceExporter反序列化漏洞，因为默认就是通过读取请求输入流进行反序列化所以导致的漏洞，而补丁则新增了白名单，只允许反序列化白名单内的包下的类。

远程调用的配置（applications/console/WEB-INF/classes/service-remotecall.xml）：

![]()

![]()

  *   *   *   *   *   * 

    
    
    // 白名单列表privatestatic final String[] whitelist = new String[] {"org.springframework.remoting.support.RemoteInvocation","[Ljava.lang", "[B", "com.tongweb","java.util", "java.lang", "javax.management" };

#### 漏洞利用

URLDNS

![]()

  * ###  **日志任意文件写入漏洞**

补丁的修复了com.tongweb.console.webcontainer.controller.AccessLogController#update在更新日志时对某些属性的检查，例如不允许日志文件日期格式包含X-
Forwarded-For，不允许日志模式包含X-Forwarded-For，对日志文件名的前缀和后缀进行检查。

![]()

对于前缀和后缀的检查：

![]()

AccessLogController#update最终会调用AccessLogService#updateAccessLog方法通过JMX更新属性。

![]()

调用JMXUtil#setAttributes设置属性：

![]()

![]()

漏洞利用方式参考Spring框架数据绑定的RCE（CVE-2022-22965），实际情况下该漏洞需要经过Console的鉴权是一个后台漏洞，并且修改日志格式化会对业务造成影响，故鸡肋。

 **加入我们  
**  

后台回复“加群”或“小助手”，或扫描下方二维码加入我们的付费圈子，一起进步吧

  

![]()

![]()

  

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

