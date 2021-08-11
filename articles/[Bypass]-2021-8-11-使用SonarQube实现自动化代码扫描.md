##  使用SonarQube实现自动化代码扫描

原创 Bypass [ Bypass ](javascript:void\(0\);)

**Bypass** ![]()

微信号 Bypass--

功能介绍 致力于分享原创高质量干货，包括但不限于：渗透测试、WAF绕过、代码审计、安全运维。 闻道有先后，术业有专攻，如是而已。

____

__

收录于话题

Sonar是一个用于代码质量管理的开源平台，通过插件机制，Sonar可与第三方工具进行集成。将Sonar引入到代码开发的过程中，提供静态源代码安全扫描能力，这无疑是安全左移的一次很好的尝试和探索。

* * *

 **1、安装Findbugs插件**  

Sonar有自己的默认的扫描规则，可通过安装Findbugs插件，来提升代码漏洞扫描能力。

（1）安装Findbugs插件进入配置→应用市场，搜索Findbugs，点击安装即可。

![](https://gitee.com/fuli009/images/raw/master/public/20210811100635.png)

在质量配置中，设置FindBugs Security Audit为默认。

![](https://gitee.com/fuli009/images/raw/master/public/20210811100642.png)

（2）代码漏洞扫描效果测试：

默认的扫描规则与FindBugs Security Audit的对比。

![](https://gitee.com/fuli009/images/raw/master/public/20210811100643.png)

 **2、IDEA集成**

通过IDEA集成Sonar，实现开发过程中就可以自动检测代码中存在的安全问题。

（1）在线安装

打开IDEA菜单，File → Settings → Plugins，搜索sonar插件，选择SonarLint进行Install，重启IDEA即可。

![]()

（2）基本使用

在IDEA中安装SonarLint插件，实现自动检测项目文件分析或者对整个项目进行分析。

![](https://gitee.com/fuli009/images/raw/master/public/20210811100644.png)

 **3、Gitlab集成**

通过Gitlab集成Sonar，就可以实现提交代码后自动邮件反馈扫描结果。

（1）在项目根目录编写.gitlab-ci.yml文件，通过GitLab-Runner实现Gitlab与Sonarqube集成。

![](https://gitee.com/fuli009/images/raw/master/public/20210811100645.png)

（2）当提交代码的时候，自动检测代码并发送报告给提交者。

![](https://gitee.com/fuli009/images/raw/master/public/20210811100646.png)

 **4、Jenkins集成**

通过Jenkins集成Sonar，就可以实现在流水线做自动化持续代码扫描。

（1）在Jenkins中，使用Pipeline流水线，拉取代码、执行打包、代码扫描。

![](https://gitee.com/fuli009/images/raw/master/public/20210811100647.png)

（2）流水线构建成功。  

![](https://gitee.com/fuli009/images/raw/master/public/20210811100648.png)

![]()

Bypass

如有帮助，请随意打赏。

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

如有帮助，请随意打赏。

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

使用SonarQube实现自动化代码扫描

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

