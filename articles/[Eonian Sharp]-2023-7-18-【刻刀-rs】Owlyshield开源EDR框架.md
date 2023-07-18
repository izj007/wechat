#  【刻刀-rs】Owlyshield开源EDR框架

Enomothem  [ Eonian Sharp ](javascript:void\(0\);)

**Eonian Sharp** ![]()

微信号 Eonian_sharp

功能介绍 Eonian Sharp | 永恒之锋，专注APT框架、渗透测试攻击与防御的研究与开发，没有永恒的安全，但有永恒的正义之锋击破黑暗的不速之客。

____

___发表于_

收录于合集

#Rust 4 个

#工具 14 个

#刻刀系列 6 个

# 介绍

Owlyshield的可扩展性是其区别于其他EDR解决方案的关键特性。作为一个框架，您可以添加用于恶意软件检测、UEBA(用户和实体行为分析)和新奇检测的新算法。您还可以使用Owlyshield来记录和回放用于训练机器学习模型的文件活动，就像我们使用自动编码器功能一样。

Owlyshield为Linux、Windows和物联网设备提供强大而高效的端点检测和响应能力。它对文件活动的独特关注使它在检测无文件恶意软件和C&C信标方面非常有效，这些可能被其他EDR解决方案忽视。

![]()

尽管Owlyshield是一个设计用于定制和扩展的框架，但它也提供了预先构建的强大功能，可以立即使用:

  * 具有自动编码器的高级新奇检测(商业版)，
  * 使用XGBoost在Windows上实时保护勒索软件，
  * 在Linux (+IoT)和Windows上进行嵌入式训练的异常检测，
  * SELinux的自动配置，自动保护公开的应用程序。

# 应用

现实生活中的例子

Owlyshield为实时检测和响应威胁提供了强大的解决方案。以下是Owlyshield如何保护客户的三个现实例子:

攻击者利用ESXi服务器中的关键CVE部署有效载荷。Owlyshield通过分析文件活动，识别ESXi进程家族中的异常行为，检测到对ESXi服务器的攻击微弱信号，表明存在恶意进程。

使用JHipster构建的web应用程序有一个隐藏的URL，可以用来转储JVM内存，但是基础架构团队并不知道这个漏洞。Owlyshield能够通过分析文件系统中与创建转储文件相关的异常活动来检测它被利用，

来自不同国家的顾问团队访问了一个庞大而昂贵的ERP系统。其中一个拥有管理员权限，开始慢慢破坏ERP系统中的特定文件。攻击者使用这种策略使攻击看起来像是一系列的bug或小故障，而不是蓄意攻击。

# 方法

Owlyshield的安装说明可以在该项目GitHub存储库的发行版(Releases)部分找到。有关使用说明，请参阅项目的Wiki，如果您喜欢自己构建Owlyshield，请参阅贡献部分。

项目： ** _https://github.com/SitinCloud/Owlyshield_**

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

