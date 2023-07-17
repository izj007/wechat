#  google hacker语句

[ 黑战士 ](javascript:void\(0\);)

**黑战士** ![]()

微信号 heizhanshi1

功能介绍 关注网络安全，为网络安全而战

____

___发表于_

收录于合集

**一、什么是谷歌Docker？**

Google Dork，也称为Google
Dorking或Google黑客攻击，是安全研究人员的宝贵资源。对于普通人来说，谷歌只是一个用于查找文本、图像、视频和新闻的搜索引擎。但是，在信息安全世界中，谷歌是一个有用的黑客工具。

有人会如何使用谷歌来入侵网站？

好吧，你不能直接使用谷歌入侵网站，但由于它具有巨大的网络爬行功能，它几乎可以索引你网站中的任何内容，包括敏感信息。这意味着您可能会在不知情的情况下暴露太多有关您的
Web 技术、用户名、密码和一般漏洞的信息。

换句话说：谷歌“Dorking”是使用谷歌使用原生谷歌搜索引擎功能来查找易受攻击的Web应用程序和服务器的做法。

除非您使用robots.txt文件阻止您网站上的特定资源，否则Google会将任何网站上存在的所有信息编入索引。从逻辑上讲，一段时间后，如果世界上的任何人都可以访问该信息，如果他们知道要搜索什么。您还可以访问Google
Hacking Database （GHDB），这是包含所有Google dorking命令的完整Google dork列表。

重要提示：虽然这些信息在互联网上是公开的，并且由谷歌在法律基础上提供和鼓励使用，但有错误意图的人可能会利用这些信息来损害您的在线形象。

请注意，当您执行此类查询时，Google也知道您是谁。出于这个原因和许多其他原因，建议仅出于良好的意图使用它，无论是用于您自己的研究还是在寻找保护您的网站免受此类漏洞侵害的方法时。

虽然一些网站站长会自行暴露敏感信息，但这并不意味着利用或利用这些信息是合法的。如果您这样做，您将被标记为网络犯罪分子。即使您使用的是VPN服务，跟踪您的浏览IP也非常容易。它并不像你想象的那么匿名。

在进一步阅读之前，请注意，如果您从单个静态 IP 连接，Google 将开始阻止您的连接。它将要求验证码挑战以防止自动查询。

![]()

 **二、受欢迎的谷歌docker语句**

谷歌的搜索引擎有自己的内置查询语言。可以运行以下查询列表来查找文件列表，查找有关竞争对手的信息，跟踪人员，获取有关SEO反向链接的信息，建立电子邮件列表，当然还可以发现Web漏洞。

  

让我们看看最受欢迎的Google Dorks以及它们的作用。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    cache：这个会向你显示任何网站的缓存版本，例如cache:securitytrails.comallintext：搜索任何网页上包含的特定文本，例如allintext: hacking toolsallintitle：与allintext完全相同，但将显示包含带有X字符的标题的页面，例如allintitle:“Security Companies”allinurl：它可用于获取其URL包含所有指定字符的结果，例如：allinurl:clientareafiletype：用于搜索任何类型的文件扩展名，例如，如果要搜索PDF文件，则可以使用：email security filetype: pdfinurl：这与 完全相同，但它只对一个关键字有用，例如allinurlinurl:adminintitle：用于搜索标题内的各种关键字，例如，将搜索以“安全”开头的标题，但“工具”可以在页面中的其他位置。intitle:security toolsinanchor：当您需要搜索任何链接上使用的确切锚文本时，这很有用，例如inanchor:“cyber security”intext：用于查找文本中包含某些字符或字符串的页面，例如intext:“safe internet”site：将显示指定域和子域的所有索引URL的完整列表，例如site:securitytrails.com*：通配符用于搜索单词前包含“任何内容”的页面，例如，将返回“如何…”设计/创建/破解等…“一个网站”。how to * a website|：这是一个逻辑运算符，例如 将显示所有包含“安全”或“提示”或这两个词的网站。“security” “tips”+：用于连接单词，可用于检测使用多个特定键的页面，例如security + trails–：减号运算符用于避免显示包含某些单词的结果，例如 将显示文本中使用“安全性”的页面，但不显示包含“跟踪”一词的页面。security -trails如果您正在寻找完整的Google运算符集，则可以关注此SEJ帖子，该帖子几乎涵盖了当今可用的所有已知dork。

  

如果您正在寻找完整的Google运算符集，则可以关注此SEJ帖子，该帖子几乎涵盖了当今可用的所有已知dork。

  

 **谷歌docker的例子**

让我们来看看一些最好的谷歌黑客的实际例子。您会惊讶于仅通过使用Google黑客技术即可从任何来源提取私人信息是多么容易。

  

 **日志文件**

日志文件是如何在任何网站中找到敏感信息的完美示例。错误日志、访问日志和其他类型的应用程序日志通常在网站的公共 HTTP
空间中发现。这可以帮助攻击者找到您正在运行的 PHP 版本，以及您的 CMS 或框架的关键系统路径。

对于这种，我们可以组合两个谷歌运算符，allintext 和 filetype，例如：

  * 

    
    
    allintext:username filetype:log

  

这将显示许多结果，其中包括所有 *.log 文件中的用户名。

在结果中，我们发现一个特定的网站显示了来自数据库服务器的SQL错误日志，其中包括关键信息：

  *   *   *   *   *   *   * 

    
    
    MyBB SQL ErrorSQL Error: 1062 - Duplicate entry 'XXX' for key 'username'Query:INSERTINTO XXX (`username`,`password`,`salt`,`loginkey`,`email`,`postnum`,`avatar`,`avatartype`,`usergroup`,`additionalgroups`,`displaygroup`,`usertitle`,`regdate`,`lastactive`,`lastvisit`,`website`,`icq`,`aim`,`yahoo`,`msn`,`birthday`,`signature`,`allownotices`,`hideemail`,`subscriptionmethod`,`receivepms`,`receivefrombuddy`,`pmnotice`,`pmnotify`,`showsigs`,`showavatars`,`showquickreply`,`showredirect`,`tpp`,`ppp`,`invisible`,`style`,`timezone`,`dstcorrection`,`threadmode`,`daysprune`,`dateformat`,`timeformat`,`regip`,`longregip`,`language`,`showcodebuttons`,`away`,`awaydate`,`returndate`,`awayreason`,`notepad`,`referrer`,`referrals`,`buddylist`,`ignorelist`,`pmfolders`,`warningpoints`,`moderateposts`,`moderationtime`,`suspendposting`,`suspensiontime`,`coppauser`,`classicpostbit`,`usernotes`)VALUES ('XXX','XXX','XXX','XXX','XXX','0','','','5','','0','','1389074395','1389074395','1389074395','','0','','','','','','1','1','0','1','0','1','1','1','1','1','1','0','0','0','0','5.5','2','linear','0','','','XXX','-655077638','','1','0','0','0','','','0','0','','','','0','0','0','0','0','0','0','')  
    

这个谷歌黑客示例向互联网暴露了当前的数据库名称，用户登录名，密码和电子邮件值。我们已将原始值替换为“XXX”。

  

易受攻击的 Web 服务器

以下Google Dork可用于检测易受攻击或被黑客入侵的服务器，这些服务器允许将“/proc/self/cwd/”直接附加到您网站的URL中。

  

  * 

    
    
    inurl:/proc/self/cwd

1

如以下屏幕截图所示，将显示易受攻击的服务器结果，以及可以从您自己的浏览器浏览的公开目录。

![]()

  

  

## 打开 FTP 服务器

Google不仅索引基于HTTP的服务器，还索引开放的FTP服务器。  
通过以下dork，您将能够探索公共FTP服务器，这些服务器通常可以揭示有趣的事情。

    
    
    intitle:"index of" inurl:ftp

![]()

  

 **SSH私钥**

SSH 私钥用于解密在 SSH 协议中交换的信息。作为一般安全规则，私钥必须始终保留在用于访问远程 SSH 服务器的系统上，并且不应与任何人共享。

  

使用以下dork，您将能够找到由Google叔叔索引的SSH私钥。

  

  * 

    
    
    intitle:index.of id_rsa -id_rsa.pub

让我们继续讨论另一个有趣的SSH Dork。

  

如果这不是您的幸运日，并且您使用的是带有 PUTTY SSH 客户端的 Windows 操作系统，请记住，此程序始终记录您的 SSH 连接的用户名。

  

在这种情况下，我们可以使用一个简单的 dork 从 PUTTY 日志中获取 SSH 用户名：

  * 

    
    
    filetype:log username putty

下面是预期的输出：

  

![]()

## 电子邮件列表

使用Google Dorks很容易找到电子邮件列表。在下面的示例中，我们将获取可能包含大量电子邮件地址的 excel 文件。

    
    
    filetype:xls inurl:"email.xls"

![]()

  

我们过滤以仅查看.edu域名，

  

  * 

    
    
    site:.edu filetype:xls inurl:"email.xls"

请记住，Google Dorks的真正力量来自您可以使用的无限组合。垃圾邮件发送者也知道这个技巧，并每天使用它来建立和发展他们的垃圾邮件列表。

  

MP3、电影和 PDF 文件

如今，几乎没有人在Spotify和qq
music出现在市场上之后下载音乐。但是，如果您是仍然下载合法音乐的经典人士之一，则可以使用此dork查找mp3文件：

  

  * 

    
    
    intitle: index of mp3

这同样适用于您可能需要的合法免费媒体文件或PDF文档：

  

  * 

    
    
    intitle: index of pdf intext: .mp4

 **天气**

谷歌黑客技术可用于获取任何类型的信息，其中包括许多连接到互联网的不同类型的电子设备。

  

在这种情况下，我们运行，可以让你获取设备传输。如果您参与气象学或只是好奇，请查看以下内容：

  

  

  

![]()

  

 **SQL 转储**

数据库配置错误是查找公开数据的一种方法。另一种方法是查找存储在服务器上并通过域/IP 访问的 SQL 转储。

  

有时，这些转储会通过网站管理员使用的错误备份机制显示在网站上，这些机制是将备份存储在网络服务器上的（假设 Google 未将其编入索引）。要查找压缩的
SQL 文件，我们使用：

  

  * 

    
    
    "index of" "database.sql.zip"

  

  

![]()

  

  

  

## WordPress 管理员

关于是否混淆WordPress登录页面的观点在双方都有争论。一些研究人员表示，这是不必要的，使用Web应用程序防火墙（WAF）等工具可以比混淆更好地防止攻击。

使用docker查找WP管理员登录页面并不太困难：![]()

  

  

 **apache2**

这可以被视为上面提到的“易受攻击的Web服务器”的子集，但是我们专门讨论Apache2，因为：

  

LAMP（Linux，Apache，MySQL，PHP）是托管应用程序/网站的流行堆栈

这些Apache服务器可能会被错误配置/遗忘或处于设置的某个阶段，使其成为僵尸网络的重要目标

查找具有以下 dork 的 Apache2 网页：

  * 

    
    
    intitle:"Apache2 Ubuntu Default Page: It works"

![]()

## phpMyAdmin

LAMP服务器上另一个有风险但经常发现的工具是phpMyAdmin软件。这个工具是另一种破坏数据的方法，因为phpMyAdmin用于通过Web管理MySQL。要使用的docker是：

    
    
    "Index of" inurl:phpmyadmin

![]()

JIRA/Kibana

Google dorks还可用于查找托管重要企业数据的Web应用程序（通过JIRA或Kibana）。

  

  *   * 

    
    
    inurl:Dashboard.jspa intext:"Atlassian Jira Project Management Software"inurl:app/kibana intext:Loading Kibana

![]()

查找 JIRA 实例的一种更简单的方法是使用 SurfaceBrowser™ 之类的工具，该工具可以识别子域以及这些子域上的应用程序（除了 JIRA
之外，还有许多其他应用程序）。

  

cpanel密码重置

另一个可以用作侦察第一步的笨蛋是托管cPanel，然后利用密码重置中的各种弱点来接管cPanel（以及托管在其上的所有网站）。为此目的，docker是：

  * 

    
    
    inurl:_cpanel/forgotpwd

  

![]()

  

防止谷歌DOCKER

有很多方法可以避免落入Google Dork的手中。

建议采取这些措施以防止您的敏感信息被搜索引擎编入索引。

使用用户和密码身份验证以及基于IP的限制来保护私人区域。

加密您的敏感信息（用户、密码、信用卡、电子邮件、地址、IP地址、电话号码等）。

对您的网站定期进行漏洞扫描，这些漏洞通常已经使用了流行的GoogleDorks查询，并且可以非常有效地检测最常见的漏洞。

在你自己的网站上运行定期的傻瓜查询，看看你是否能在坏人之前找到任何重要信息。你可以在Exploit DB dorks数据库中找到一个受欢迎的傻瓜列表。

如果您发现敏感内容暴露，请使用 Google搜索控制台请求将其移除。

使用位于根级网站目录中的 robots.txt文件阻止敏感内容。

  

 **文章结语**

谷歌是世界上最重要的搜索引擎之一。众所周知，除非我们明确否认，否则它有能力索引所有内容。

  

今天我们了解到，谷歌也可以用作黑客工具，但您可以比坏人领先一步，并定期使用它来查找自己网站中的漏洞。您甚至可以集成它并使用自定义的第三方 Google
SERPS API 运行自动扫描。

  

如果您是安全研究人员，那么在负责任地使用时，它可以成为您网络安全职责的实用工具。

  

虽然Google
Dorking可用于揭示有关您网站的敏感信息，这些信息可通过HTTP协议定位和索引，但您也可以使用SecurityTrails工具包执行完整的DNS审计。

  

  

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

