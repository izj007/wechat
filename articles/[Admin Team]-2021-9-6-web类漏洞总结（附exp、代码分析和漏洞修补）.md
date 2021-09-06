#  web类漏洞总结（附exp、代码分析和漏洞修补）

running  [ Admin Team ](javascript:void\(0\);)

**Admin Team** ![]()

微信号 Amin_Bug

功能介绍 专注于网络安全、渗透测试、反诈骗。为提升国民的网络安全意识做贡献！

____

__

收录于话题

  

  

WEB中间件

  

  

  

  * Tomcat

  

Tomcat是Apache Jakarta软件组织的一个子项目，Tomcat是一个JSP/Servlet容器，它是在SUN公司的JSWDK（Java
Server Web Development
Kit）基础上发展起来的一个JSP和Servlet规范的标准实现，使用Tomcat可以体验JSP和Servlet的最新规范。

  

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    端口号：8080攻击方法：默认口令、弱口令，爆破，tomcat5 默认有两个角色：tomcat和role1。其中账号both、tomcat、role1的默认密码都是tomcat。弱口令一般存在5以下的版本中。在管理后台部署 war 后门文件远程代码执行漏洞参考：https://paper.seebug.org/399/http://www.freebuf.com/column/159200.htmlhttp://liehu.tass.com.cn/archives/836http://www.mottoin.com/87173.html

  

  * Jboss

  

是一个运行EJB的J2EE应用服务器。它是开放源代码的项目，遵循最新的J2EE规范。从JBoss项目开始至今，它已经从一个EJB容器发展成为一个基于的
J2EE 的一个Web 操作系统（operating system for web），它体现了 J2EE 规范中最新的技术。

  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    端口：8080攻击方法：弱口令，爆破管理后台部署 war 后门反序列化远程代码执行参考：http://www.vuln.cn/6300http://mobile.www.cnblogs.com/Safe3/archive/2010/01/08/1642371.htmlhttps://www.zybuluo.com/websec007/note/838374https://blog.csdn.net/u011215939/article/details/79141624

  

  * WebLogic

  

WebLogic是美国Oracle公司出品的一个Application
Server，确切的说是一个基于JAVAEE架构的中间件，WebLogic是用于开发、集成、部署和管理大型分布式Web应用、网络应用和数据库应用的Java应用服务器。将Java的动态功能和Java
Enterprise标准的安全性引入大型网络应用的开发、集成、部署和管理之中。

  

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    端口：7001，7002攻击方法：弱口令、爆破，弱密码一般为weblogic/Oracle@123 or weblogic管理后台部署 war 后门SSRF反序列化漏洞weblogic_uac参考：https://github.com/vulhub/vulhub/tree/master/weblogic/ssrfhttps://blog.gdssecurity.com/labs/2015/3/30/weblogic-ssrf-and-xss-cve-2014-4241-cve-2014-4210-cve-2014-4.htmlhttps://fuping.site/2017/06/05/Weblogic-Vulnerability-Verification/https://bbs.pediy.com/thread-224954.htm

  

  * WebSphere

  

IBM公司一套典型的电子商务应用开发工具及运行环境。

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    端口：默认端口：908*；第一个应用就是9080，第二个就是9081；控制台9090；  
    攻击方法：控制台登录爆破很多内网 websphere 的控制台存在弱口令 / 默认口令，可以使用 admin/admin 以及 webshpere/webshpere 这种口令登录。通过该口令登录控制台后，可以部署 war 包，从而获取到 WEBSHELL 。反序列化任意文件泄露参考：  
      
    https://loudong.sjtu.edu.cn/?keyword=WebSphere&serverity=%E9%AB%98%E5%8D%B1http://www.fr1sh.com/wooyun_1/bug_detail.php?wybug_id=wooyun-2013-036803https://gist.github.com/metall0id/bb3e9bab2b7caee90cb7

  * ### Glassfish

GlassFish 是一款用于构建 Java EE的应用服务组件。

> 任意文件读取：https://www.freebuf.com/vuls/93338.html  
> 目录穿越：https://www.anquanke.com/post/id/85948

  

  

WEB框架

  

  

  * Struts 2

  

Struts2是一个优雅的,可扩展的框架,用于创建企业准备的Java Web应用程序。出现的漏洞也着实的多每爆一个各大漏洞平台上就会被刷屏。

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    可利用漏洞S2-057 CVE-2018-11776 Apache Struts 2.3 – Struts 2.3.34、 Apache Struts 2.5 – Struts 2.5.16S2-046 CVE-2017-5638 Struts 2.3.5-2.3.31,Struts 2.5-2.5.10S2-045 CVE-2017-5638 Struts 2.3.5-2.3.31,Struts 2.5-2.5.10S2-037 CVE-2016-4438 Struts 2.3.20-2.3.28.1S2-032 CVE-2016-3081 Struts 2.3.18-2.3.28S2-020 CVE-2014-0094 Struts 2.0.0-2.3.16S2-019 CVE-2013-4316 Struts 2.0.0-2.3.15.1S2-016 CVE-2013-2251 Struts 2.0.0-2.3.15S2-013 CVE-2013-1966 Struts 2.0.0-2.3.14S2-009 CVE-2011-3923 Struts 2.0.0-2.3.1.1S2-005 CVE-2010-1870 Struts 2.0.0-2.1.8.1参考：https://github.com/hktalent/myhktoolshttps://github.com/Lucifer1993/struts-scanhttps://github.com/SecureSkyTechnology/study-struts2-s2-054_055-jackson-cve-2017-7525_cve-2017-15095

  

  * Spring框架

  

Spring Framework 是一个开源的Java／Java EE全功能栈（full-stack）的应用程序框架，以Apache License
2.0开源许可协议的形式发布，也有.NET平台上的移植版本。Spring
Framework提供了一个简易的开发方式，这种开发方式，将避免那些可能致使底层代码变得繁杂混乱的大量的属性文件和帮助类。

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    可利用漏洞CVE-2010-1622CVE-2018-1274CVE-2018-1270CVE-2018-1273反序列化目录穿越参考http://www.inbreak.net/archives/377https://www.secpulse.com/archives/71762.htmlhttp://www.open-open.com/news/view/1225d07https://xz.aliyun.com/t/2261https://xz.aliyun.com/t/2252

  

  

WEB服务器漏洞

  

  

  * IIS(Windows 的 WWW 服务器)

  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    端口：80攻击方法：IIS，开启了 WebDAV，可以直接详服务器 PUT 文件短文件名枚举漏洞远程代码执行提权漏洞解析漏洞参考：https://masterxsec.github.io/2017/06/07/IIS-write-%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8/http://www.freebuf.com/articles/4908.htmlhttps://www.anquanke.com/post/id/85811

  

  * Apache

  *   *   *   * 

    
    
    端口：80攻击方法：解析漏洞目录遍历

  

  * Nginx

  *   *   *   *   *   *   * 

    
    
    端口：80攻击方法：解析漏洞目录遍历CVE-2016-1247：需要获取主机操作权限，攻击者可通过软链接任意文件来替换日志文件，从而实现提权以获取服务器的root权限。参考：https://www.seebug.org/vuldb/ssvid-92538

  * lighttpd

  *   *   *   *   * 

    
    
    端口：80攻击方法：目录遍历WEB应用常见WEB应用有邮件应用、CMS 应用，在搜索引擎上查找对应的漏洞，利用已知漏洞进行攻击。

  

  

  

邮件系统

  

  

一部分是使用腾讯企业邮箱、阿里企业邮箱的，很难有可利用的漏洞，另外一种是能独立部署的邮件系统，政企常用的邮箱应用：

  

  *   *   *   *   *   *   *   * 

    
    
    Coremail亿邮35互联TurboMailExchangeIBM Lotus参考：https://www.cnblogs.com/xiaozi/p/14481595.htm

  

  

CMS漏洞

  

  

类型：

 **p hp类cms系统：**

dedecms、帝国cms、php168、phpcms、cmstop、discuz、phpwind等

 **asp类cms系统** ：zblog、KingCMS等

 **国外的著名cms系统** ：

joomla、WordPress 、magento、drupal 、mambo。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210906083809.png)

  * 

    
    
    exp链接：https://github.com/SecWiki/CMS-Hunter

  

  

  

  

 **声明：文中技术仅用于技术研究**

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

web类漏洞总结（附exp、代码分析和漏洞修补）

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

