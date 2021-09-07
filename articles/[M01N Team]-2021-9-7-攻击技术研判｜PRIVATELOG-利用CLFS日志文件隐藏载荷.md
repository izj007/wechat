#  攻击技术研判｜PRIVATELOG-利用CLFS日志文件隐藏载荷

原创 天元实验室  [ M01N Team ](javascript:void\(0\);)

**M01N Team** ![]()

微信号 m01nteam

功能介绍 攻击对抗研究分享

____

__

收录于话题 #攻击技术研判 ,13个

![](https://gitee.com/fuli009/images/raw/master/public/20210907085913.png)

**情报背景**

Fireeye
Mandiant安全团队近期发布了针对恶意软件家族PRIVATELOG的分析报告。该报告对此恶意软件家族进行了首次披露，并对其利用CLFS日志文件隐藏恶意载荷的攻击手法进行了重点介绍。  
本文将对本次事件中出现的CLFS日志文件滥用手法及新型DLL Hollowing技术进行分析研判。

  

 **组织信息**

|

未知  
  
---|---  
  
 **战术标签**

|

载荷隐藏  
  
 **情报来源**

|

https://www.fireeye.com/blog/threat-research/2021/09/unknown-actor-using-clfs-
log-files-for-stealth.html  
  
  

 **01** 攻击技术分析

 **亮点一：利用CLFS日志文件存储恶意载荷**

CLFS
是微软为实现高性能日志存储而引入的日志框架，使用blf文件格式存储数据。由于该文件格式应用较少，文件格式细节也未广泛披露，第三方分析工具大多缺少对其的支持。这些数据可以直接通过clfsw32.dll中提供的API函数进行访问。这为攻击者滥用其隐藏恶意载荷创造了条件。

  

攻击者通过clfsw32.dll中的API函数CreateLogFile()打开用户配置文件目录下查找注册表事务日志文件，然后通过CloseAndResetLogFile()函数重置日志文件。

![](https://gitee.com/fuli009/images/raw/master/public/20210907085915.png)

  

之后再次打开日志文件，利用ReserveAndAppendLog()函数将内容写入CLFS日志的第一个容器文件。

![](https://gitee.com/fuli009/images/raw/master/public/20210907085916.png)

  

最终存储到文件中的数据如下所示：

![](https://gitee.com/fuli009/images/raw/master/public/20210907085917.png)

  

 **亮点二：利用Phantom DLL Hollowing技术执行恶意代码    **

PRIVATELOG使用了依赖于NTFS事务的技术实现DLL的执行，其注入过程类似于Phantom DLL空洞（白DLL修改技术）：

在得到目标DLL之后，恶意程序打开相应的文件句柄，利用该文件句柄创建一个Section并将其映射到本地内存空间，该内存空间可以被修改并植入shellcode。通过这个过程实现恶意代码的加载执行。



整体利用过程大致如下：

  1. 创建合法文件 C:\Windows\System32\dbghelp.dll的副本到C:\Windows\system32\WindowsPowerShell\v1.0\dbghelp.dll路径下（后续过程需要目标文件的写权限）

  2. 使用API CreateFileTransactedA() 创建副本文件的事务文件句柄

  3. 将恶意载荷通过事务文件句柄写入副本文件

  4. 使用事务文件句柄创建SEC_IMAGE属性的 Section

  5. 将Section 映射到内存

  6. 修复执行权限

  7. 解析导入函数

  8. 执行入口函数

  

DLL hollowing 的技术发展对现有的内存检测技术提出挑战，不需要分配新的内存空间， 且映射文件对象仍将指向磁盘上合法的
Microsoft签名DLL。

  

 **02** 总结

  1. 本次出现的样本创新性地利用了注册表事物日志文件进行恶意载荷数据的存储，依靠通用日志文件系统CLFS隐藏第二阶段的有效负载；

  2. 利用Phantom DLL Hollowing技术将恶意代码加载到可信内存空间执行。

  

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210907085918.png)

 **绿盟科技M01N战队** 专注于Red
Team、APT等高级攻击技术、战术及威胁研究，涉及Web安全、终端安全、AD安全、云安全等相关领域。通过研判现网攻击技术发展方向，以攻促防，为风险识别及威胁对抗提供决策支撑，全面提升安全防护能力。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210907085919.png)

 **M01N Team**

聚焦高级攻防对抗热点技术

绿盟科技蓝军技术研究战队

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

攻击技术研判｜PRIVATELOG-利用CLFS日志文件隐藏载荷

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

