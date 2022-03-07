#  攻击技术研判 ｜钓鱼文档攻击利用MSI对象安装Lua环境隐藏后门

原创 天元实验室  [ M01N Team ](javascript:void\(0\);)

**M01N Team** ![]()

微信号 m01nteam

功能介绍 攻击对抗研究分享

____

__

收录于话题 #攻击技术研判 34个

![](https://gitee.com/fuli009/images/raw/master/public/20220307210529.png)

**情报背景**

近期随着俄乌冲突不断升级，Proofpoint公司观察到有组织利用已被感染盗用的乌克兰武装部队成员的邮件账号传播带有宏的XLS文档，以此对欧洲政府发起网络钓鱼攻击。该活动中使用宏代码对象Windows
Installer和MSI释放安装基于Lua语言后门的利用方式非常独特。

  

 **组织**

 **名称**

|

UNC1151  
  
---|---  
  
 **关联组织**

|

Ghostwriter  
  
 **战术标签**

|

网络钓鱼  
  
 **技术标签**

|

VBA MSI lua  
  
 **情报来源**

|

https://www.proofpoint.com/us/blog/threat-insight/asylum-ambuscade-state-
actor-uses-compromised-private-ukrainian-military-emails  
  
  

 **01** 攻击技术分析

整体攻击流程：

1\. 被盗人员电子邮件账号传播带有宏代码的XLS文档

2\. 文档被打开并启用后向攻击者的恶意地址发起连接

3\. 从连接处下载恶意MSI安装包

4\. MSI安装包首先释放执行Lua脚本所需的依赖文件，创造了一个Lua执行环境，同时释放了一个LNK文件用于持久化

5\. 当下次Windows启动的时候，Lnk将启动Lua的可执行文件去运行包含恶意代码的Lua脚本

6\. 恶意Lua脚本将获取C盘的序列信息，并每两秒一次与攻击者的C2地址通讯，执行攻击者的下达的后续请求

![](https://gitee.com/fuli009/images/raw/master/public/20220307210539.png)

  

 **1.1 独特的宏代码-WindowsInstaller**

在攻击流程2中，当受害者打开恶意文档启用宏执行，会执行一段简单但独特的宏代码。该宏通过CreateObject创建一个 Windows
Installer（msiexec.exe）对象。该对象调用Windows Installer以远程连接在攻击者控制下的IP下载安装恶意MSI包。

  

通过将UILevel设置为2，将指定Windows Installer以安全静默模式安装，对用户隐藏的安装请求与安装过程界面。

  

Windows
Installer对象的InstallProduct方法会将从URL获取的MSI安装文件，缓存到本地，然后调用msiexec.exe安装恶意MSI包。由于攻击者使用MSI包作为后续基于Lua的恶意软件的安装程序，所以通过网络钓鱼传递的宏文档进行安装部署非常合适。

  

使用宏文档来进行网络钓鱼攻击非常常见，但本次攻击者使用了一个比较少见和独特的对象Windows
Installer来实现恶意后门的部署，这种方式还比较独特和少见。从使用效果来看，该方式非常方便，不需要很多的宏代码和行为就将攻击过程从office转移到msi，只要终端安全软件针对宏代码执行windows
installer没有限制执行，就可以利用这个合法对象切入下一阶段的恶意攻击。较为有效和隐蔽。

![](https://gitee.com/fuli009/images/raw/master/public/20220307210540.png)

  

 **1.2 使用MSI部署基于Lua的恶意软件**

在将宏文档钓鱼转换到恶意MSI安装包后，攻击者也没有直接去安装恶意的二进制后门和脚本，而是先安装了一系列Lua依赖的合法文件，在这些合法文件中包含了恶意Lua脚本。通过这种方式，攻击者可以进一步去规避与安全软件在二进制文件查杀的直接对抗，通过选择恶意lua脚本的方式实现后门执行。利用MSI安装包的方式也适合去安装脚本执行环境的一系列依赖文件。之后通过安装用于Windows
启动运行的LNK文件实现持久性。

  

释放的合法Lua文件：

  * luacom.dll （LuaCom 库文件）

  * ltn12.lua （LuaSocket 脚本）

  * mime.lua 

  * http.lua 

  * url.lua 

  * tp.lua

  * socket.lua

  * tp.lua

  * core.dll

  * mime.dll

  * lua51.dll

  * sppsvc.exe(经过攻击者修改的Lua解释器，该版本不会对用户输出控制台界面)

  * <6 characters>.rbs （windows 安装回滚脚本）

  

  

释放的恶意Lua文件：

  * print.lua  

  

  

释放的恶意LNK位置

  * ~\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\Software Protection Service.lnk

  

  

恶意LNK被设置为启动释放于C:\ProgramData\\.security-soft\目录的sppsvc.exe去运行print.lua,
之后该脚本尝试从攻击者的C2获取额外的恶意Lua代码。

  

 **02  **总结

在网络钓鱼攻击中，攻击者通常会选定一些独特的方式去规避与安全软件的直接对抗。在这次攻击活动中我们看到在宏文档中运行windows
installer对象安装恶意的MSI安装包。之后利用MSI包来安装基于脚本语言的后门非常有趣，如果相关安全软件针对这两点没有完好的防护方案，攻击者就能够有机可乘。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220307210541.png)

 **绿盟科技天元实验室** 专注于新型实战化攻防对抗技术研究。

研究目标包括：漏洞利用技术、防御绕过技术、攻击隐匿技术、攻击持久化技术等蓝军技术，以及攻击技战术、攻击框架的研究。涵盖Web安全、终端安全、AD安全、云安全等多个技术领域的攻击技术研究，以及工业互联网、车联网等业务场景的攻击技术研究。通过研究攻击对抗技术，从攻击视角提供识别风险的方法和手段，为威胁对抗提供决策支撑。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220307210542.png)

 **M01N Team**

聚焦高级攻防对抗热点技术

绿盟科技蓝军技术研究战队

  

 **往期推荐**

[攻击技术研判
｜Lazarus搭载新的载荷执行与控制流劫持技术](http://mp.weixin.qq.com/s?__biz=MzkyMTI0NjA3OA==&mid=2247486868&idx=1&sn=0bb5b2b51296bcd2cfc65adfaed3f836&chksm=c187cd85f6f04493cc8ec77e904227a7f5a63db61da12eae72c07edaab3484c1231515251973&scene=21#wechat_redirect)  

![](https://gitee.com/fuli009/images/raw/master/public/20220307210543.png)

[攻击技术研判 ｜
利用SEO技术的钓鱼攻击与样本隐匿持久化技术](http://mp.weixin.qq.com/s?__biz=MzkyMTI0NjA3OA==&mid=2247486819&idx=1&sn=f45a4af8793628fa4a25e8f1b679ff27&chksm=c187cd72f6f0446403264c6df814f0145375514cd80fb8e4f81d3633972715bc95643d63f360&scene=21#wechat_redirect)  

![](https://gitee.com/fuli009/images/raw/master/public/20220307210543.png)

[攻击技术研判｜在野Web注入及证书透明度检测规避手法分析](http://mp.weixin.qq.com/s?__biz=MzkyMTI0NjA3OA==&mid=2247486777&idx=1&sn=e35345a65a62764d9182ef750b349de8&chksm=c187cd28f6f0443e1ee1cd2ad67b670d4565cc4e80d1bca43004fd9af4115d59df971c2477df&scene=21#wechat_redirect)  

![](https://gitee.com/fuli009/images/raw/master/public/20220307210543.png)

  

预览时标签不可点

收录于话题 #

 个

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

攻击技术研判 ｜钓鱼文档攻击利用MSI对象安装Lua环境隐藏后门

最多200字，当前共字

__

发送中

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

