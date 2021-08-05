##  COM对象挖掘及攻击利用

原创 天元实验室  [ M01N Team ](javascript:void\(0\);)

**M01N Team** ![]()

微信号 m01nteam

功能介绍 攻击对抗研究分享

____

__

收录于话题

#蓝军

4个

![](https://gitee.com/fuli009/images/raw/master/public/20210805110440.png)

**概述**

在实网攻击威胁及红蓝对抗当中，攻击方越来越多的进行白利用以完成反检测等防御规避目的，包括LOLBAS、VULNBins等可信文件形式，但Windows系统当中同样也存在很多有趣的COM对象可被用来进行命令执行、信息收集、添加计划任务等，这些对象本身不是安全漏洞，但可以滥用这些对象对抗基于行为特征和启发式特征的安全检测。在本文当中我们会分享一些COM对象挖掘和攻击利用方面的思路。

  

 **01** COM对象、CLSID与ProgID

注册表项HKEY_CLASSES_ROOT \
CLSID公开了枚举COM对象所需的所有信息，包括CLSID和ProgID。CLSID是与COM类对象关联的全局唯一标识符。ProgID是表示底层CLSID的程序员友好字符串。

  

 **CLSID转ProgID**  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110441.png)

 **ProgID转CLSID**

![](https://gitee.com/fuli009/images/raw/master/public/20210805110442.png)

 **获取COM名称**  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110443.png)  

 **02  **枚举可用COM对象的方法

 **方法一**

以下Powershell命令获得CLSID的列表。

![](https://gitee.com/fuli009/images/raw/master/public/20210805110444.png)

  

我们可以使用CLSID列表依次实例化每个对象，然后枚举每个COM对象公开的方法和属性，在没有管理特权的最坏情况下，可以使用标准用户特权来深入了解可用的COM对象。

![]()

  

 **方法二**

枚举COM对象。  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110445.png)![](https://gitee.com/fuli009/images/raw/master/public/20210805110446.png)

  

枚举COM对象方法并利用。

![](https://gitee.com/fuli009/images/raw/master/public/20210805110448.png)![](https://gitee.com/fuli009/images/raw/master/public/20210805110449.png)

  

 **方法三**

利用第三方工具OleViewDotNet可通过ProgID枚举COM对象。

![](https://gitee.com/fuli009/images/raw/master/public/20210805110450.png)

  

右击查看COM接口的Type Library。

![](https://gitee.com/fuli009/images/raw/master/public/20210805110451.png)

  

查看该COM接口中包含的接口与方法，其中包括潜在可利用函数Exec方法。

![](https://gitee.com/fuli009/images/raw/master/public/20210805110452.png)

  

通过“Create Instance”功能创建该COM接口的实例后可进一步执行其中的方法进行测试。

![](https://gitee.com/fuli009/images/raw/master/public/20210805110453.png)

  

设置方法的参数。

![](https://gitee.com/fuli009/images/raw/master/public/20210805110454.png)

  

点击invoke后即可测试该方法的功能，此处Exec方法传入的命令被成功执行，创建了新的计算器进程。

![]()

  

  

 **03  **部分可利用的COM接口

 **ProcessChain Class**

ProgID：ProcessC hainLib

CLSID：{E430E93D-09A9- 4DC5-80E3- CBB2FB9AF28E}

滥用：命令执行

代码：

![](https://gitee.com/fuli009/images/raw/master/public/20210805110456.png)

  

 **Windows Script Host Shell Object**

ProgID：WScript. Shell.1

CLSID：{72C24DD5-D70A- 438B-8A42-98424B88AFB8}

滥用：命令执行、注册表操作、获取环境变量等

代码：

![](https://gitee.com/fuli009/images/raw/master/public/20210805110457.png)

  

 **XML HTTP 3.0**  

ProgID：Msxml2. XMLHTTP. 3.0

CLSID：{F5078F35-C551- 11D3-89B9-0000F81FE221}

滥用：无文件下载执行

代码：

![]()

  

 **TaskScheduler class**

ProgID：Schedule. Service. 1

CLSID：{0F87369F-A4E5-4CFC-BD3E73E6154572DD}

滥用：命令执行

代码：

![](https://gitee.com/fuli009/images/raw/master/public/20210805110458.png)

  

 **MMC Application Class**  

ProgID：MMC20. Application

CLSID：{49B2791A-B1AE- 4C90-9B8E- E860BA07F889}

滥用：命令执行

代码：

![](https://gitee.com/fuli009/images/raw/master/public/20210805110459.png)

  

 **ShellWindows**

CLSID：{9BA05972-F6A8- 11CF-A442-00A0C90A8F39}

滥用：命令执行

代码：

![](https://gitee.com/fuli009/images/raw/master/public/20210805110500.png)

  

 **ShellBrowserWindow**

ProgID：ProcessC hainLib

CLSID：{C08AFD90-F2A1- 11D1-8455-00A0C91F3880}

滥用：命令执行

代码：

![]()

  

  

 **04  **总结

本文对于批量枚举COM接口的方法进行了探讨，通过这些方法可对系统中包含的COM对象及其方法进行快速枚举，进而挖掘具有实战利用价值的COM接口。而在提供三种枚举COM对象的方法之外，本文也对部分可利用的COM接口及其调用方式进行了分享，旨在抛砖引玉，启发蓝军研究人员对COM接口滥用的实战价值与挖掘思路的思考。

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110502.png)

 **绿盟科技M01N战队** 专注于Red
Team、APT等高级攻击技术、战术及威胁研究，涉及Web安全、终端安全、AD安全、云安全等相关领域。通过研判现网攻击技术发展方向，以攻促防，为风险识别及威胁对抗提供决策支撑，全面提升安全防护能力。

  

![]()

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

COM对象挖掘及攻击利用

最多200字，当前共字

__

发送中

写下你的留言

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

