#  内网渗透软件-BloodHound使用指南

熊陶  [ 平安集团安全应急响应中心 ](javascript:void\(0\);)

**平安集团安全应急响应中心** ![]()

微信号 PSRC_Team

功能介绍 平安集团安全应急响应中心隶属于平安科技，是外部用户向平安集团反馈各产品和业务安全漏洞的平台，也是平安科技加强与安全界和同仁合作交流的渠道之一。

____

___发表于_

收录于合集

#PSRC 141 个

#漏洞分析 20 个

  

**> >>> ** **0x01  基本信息**

  

软件名称：BloodHound软件类型：内网渗透软件版本：4.1.1下载地址：https://github.com/BloodHoundAD/BloodHound/releases/tag/4.1.1同类软件：ADExplorer

##  

##  **> >>> ** **0x02 软件描述**

  
BloodHound
通过图与线的形式，将域内用户、计算机、组、会话、ACL,以及域内所有的相关用户、组、计算机、登录信息、访问控制策略之间的关系，直观地展现在红队成员面前，为他们更便捷地分析域内情况、更快速地在域内提升权限提供条件。  
BloodHound可以帮助蓝队成员更好地对己方网络系统进行安全检查，以及保证域的安全性。BloodHound使用图形理论，在活动目录环境中自动理清大部分人员之间的关系和细节。使用BloodHound,可以快速、深人地了解活动目录中用户之间的关系，获取哪些用户具有管理员权限、哪些用户对所有的计算机都具有管理员权限、哪些用户是有效的用户组成员等信息。  
BloodHound可以在域内导出相关信息，将采集的数据导人本地Neo4j数据库，并进行展示和分析。Neo4j是一款NoSQL图形数据库，它将结构化数据存储在网络内而不是表中。BloodHound正是利用Neo4j的这种特性，通过合理的分析，直观地以节点空间的形式表达相关数据的。Neo4j和MySQL及其他数据库一样，拥有自己的查询语言Cypher
Query Language。因为Neo4j是一款非关系型数据库，所以，要想在其中进行查询，同样需要使用其特有的语法。

##  

##  **> >>> ** **0x03 软件安装**

  
该软件正常是安装在本地，日常内网渗透过程中，是需要将渗透的内网域信息导出来，然后通过该工具进行分析。  
1\. BloodHound 使用neo4j数据库，需要配备java环境。  
2.安装neo4j数据库可以在https://neo4j.com/download-
center/#community地址中下载.exe文件进行安装（PS：建议下载.exe文件进行安装，如果使用源码文件进行安装对java版本要求很严格，很容易跑不起来）  
3.安装好neo4j数据库后，使用地址https://github.com/BloodHoundAD/BloodHound/releases/tag/4.1.1下载BloodHound工具，填入默认用户名密码neo4j/neo4j。  
![](https://gitee.com/fuli009/images/raw/master/public/20220923141402.png)  
如果neo4j数据库安装失败，登录页面会提示数据库未安装。

##  

##  **> >>> ** **0x04 使用说明**

  
1\. 正常安装neo4j数据库和BloodHound后，BloodHound即可通过用户密码进行登录。  
![](https://gitee.com/fuli009/images/raw/master/public/20220923141403.png)  
首次登录该页面会是空白的。  
2\. 需用使用https://github.com/BloodHoundAD/BloodHound工具对域信息进行收集。  
使用命令SharpHound.exe -c all对域信息进行收集并打包成zip文件。  
  
![](https://gitee.com/fuli009/images/raw/master/public/20220923141405.png)  
3\. 将打包的域信息导入进BloodHound软件中去，导入后就可以看到域相关的信息了。  
  

![](https://gitee.com/fuli009/images/raw/master/public/20220923141407.png)

##  **  
**

##  **> >>> ** **0x05  软件使用**

  
BloodHound软件的一些核心功能。

  * Find all Domain Admins：查询所有域管理员
  * Map Domain Trusts：映射域信任
  * Find Computers with Unsupported Operating Systems 查找操作系统不受支持的计算机
  * Find Shortest Paths to Domain Admins：查找到达域管理员的最短路径
  * Find Principals with DCSync Rights：查找具有DCSync权限的主体
  * Users with Foreign Domain Group Membership：具有外部域组成员身份的用户
  * Groups with Foreign Domain Group Membership：具有外部域组成员身份的组
  * Shortest Paths to Unconstrained Delegation Systems：无约束委托系统的最短路径
  * Shortest Paths from Kerberoastable Users：Kerberoastable用户的最短路径
  * Shortest Paths to Domain Admins from kerberoastable Users：从Kerberoathable用户到域管理员的最短路径
  * Shortest Path from Owned Principals：拥有主体的最短路径
  * Shortest Paths to Domain Admins from Owned Principals：从所属主体到域管理员的最短路径
  * Shortest Paths to High Value Targets：高价值目标的最短路径

  
1.举例Find all Domain Admins说明。  
  
![](https://gitee.com/fuli009/images/raw/master/public/20220923141409.png)  
即可获取到所有域管理员，也可以点击图形中的域管理员，对其进行详细信息查看。  
  
![](https://gitee.com/fuli009/images/raw/master/public/20220923141410.png)  
2.举例Shortest Paths to High Value Targets，该功能比较常用，在日常渗透过程中可以更好的辅助进行横向渗透。  
  
![](https://gitee.com/fuli009/images/raw/master/public/20220923141413.png)  
该功能将域环境中的所有节点，所有用户、域控制器等信息都表示出来了。我们也可以将鼠标移到到任意一台机器或者一个节点上，它会出现红色的流信息，流向的节点就表示该机器可去往的节点。  
  
![](https://gitee.com/fuli009/images/raw/master/public/20220923141415.png)  
更好的辅助我们进行横向渗透。

  

  
  

银河实验室

![](https://gitee.com/fuli009/images/raw/master/public/20220923141416.png)

银河实验室（GalaxyLab）是平安集团信息安全部下一个相对独立的安全实验室，主要从事安全技术研究和安全测试工作。团队内现在覆盖逆向、物联网、Web、Android、iOS、云平台区块链安全等多个安全方向。官网：http://galaxylab.pingan.com.cn/

  

  

往期回顾

  

技术

[](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652140750&idx=1&sn=907ceb3795a13e8b54910f67e67e8938&chksm=f320d86ec4575178eec0aed8e50d4a1c4ad91aa2edfde855fcf0bfa0d80301f33459c688a07f&scene=21#wechat_redirect)[水坑钓鱼研究](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652142558&idx=1&sn=73637d6e2fa4ffeee0be92dee413368d&chksm=f320d77ec4575e6847bb18257ad65150870cff5a8f4160d0c3aa5d72beff22d8a5569f3c7e28&scene=21#wechat_redirect)  

技术  

[S2-062漏洞原理分析](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652142452&idx=1&sn=f682bb48510b36243f29813f9fc8c5f6&chksm=f320d7d4c4575ec27ee173ed4be2870e4bf6cfe7d62991059578926442d5c898a25602c7887e&scene=21#wechat_redirect)  

技术

[利用Outlook宏创建基于邮件触发的后门](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652142488&idx=1&sn=3b8db434dd0b2031543e3d592b9a0ac8&chksm=f320d738c4575e2e5cbce7e96055abb61343e46895c9fd7e42eaa2a618aa08efe54310a1e877&scene=21#wechat_redirect)  

技术

[技术解析 |
JWT弱点的利用方式](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652142351&idx=1&sn=8c5d2bcad52df27c2f0ae5beb58fa41e&chksm=f320d7afc4575eb96fde8ea53681d2fd119f10d3decf3bbd5e5e6eaccc72ab255e1909f9d315&scene=21#wechat_redirect)  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20220923141417.png)

 **活动专区**

公开转发此文至朋友圈即可参与抽奖。

 **抽奖方式：发送关键词 **“ PSRC2022** **”** 至本微信公众号后台获取抽奖链接，**点击链接抽奖，9月9日12:00自动开奖。

*注意：兑奖时请发送当日朋友圈转发截图，不可设置分组！

奖品：PSRC中秋月饼礼盒1个

![](https://gitee.com/fuli009/images/raw/master/public/20220923141418.png)

  

  
  
  
  

 **点赞、分享，感谢你的阅读 ▼ **

  

 **▼  点击阅读原文，进入官网**

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

