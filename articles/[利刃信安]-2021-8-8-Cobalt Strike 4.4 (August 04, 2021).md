##  Cobalt Strike 4.4 (August 04, 2021)

[ 利刃信安 ](javascript:void\(0\);)

**利刃信安** ![]()

微信号 LRXAEGZ

功能介绍 利刃信安

____

__

收录于话题

#远程控制

2个

我很高兴地通知您 Cobalt Strike 4.4 现已推出！此版本为您提供了更多控制权，提高了 Cobalt Strike
的回避质量，并解决了我们用户要求的一些较小的更改……是的！我们添加了重新连接按钮！以下是有关最新功能的更多详细信息：

![](https://gitee.com/fuli009/images/raw/master/public/20210808101817.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210808101818.png)

 **  
**

![](https://gitee.com/fuli009/images/raw/master/public/20210808101819.png)

 **  
**

![](https://gitee.com/fuli009/images/raw/master/public/20210808101821.png)

 **  
**

![](https://gitee.com/fuli009/images/raw/master/public/20210808101822.png)

 **  
**

![](https://gitee.com/fuli009/images/raw/master/public/20210808101823.png)

 **  
**

 **用户指定的反射 DLL 加载程序**  
Cobalt Strike
在其反射加载基础上具有很大的灵活性，但它也有局限性。我们已经看到很多社区对这个领域的兴趣，因此我们进行了更改以允许您完全绕过它并改为定义您自己的反射加载过程。默认的反射加载器仍可随时使用。  

我们还扩展了 4.2 版本中最初对反射加载器所做的更改，为您提供了一个 Aggressor 脚本挂钩，允许您指定自己的反射加载器并完全重新定义 Beacon
如何加载到内存中。提供了一个攻击者脚本 API
来促进这个过程。由于这是一个如此重大的变化，我们计划跟进一个单独的博客文章来更深入地介绍这个功能。目前，您可以在此处找到更多信息。

  

 **避免用于 Beacon 元数据解析的 Localhost Sysmon Event 22**  
Beacon 启动时，它会解析元数据以发送回 Cobalt Strike。以前，Beacon 在成熟的环境中脱颖而出，因为用于解析此元数据的方法会触发
Sysmon 事件 22（DNS 查询），并且已成为每次运行时可靠地对 Beacon 进行指纹识别的方法。4.4
版本修改了此元数据的解析方式，以便不再发生这种情况。

 **用户定义sleep_mask面膜/取消屏蔽存根**  
的 **sleep_mask** 是钴击的掩蔽和揭露本身在存储器中的能力。此功能的目标是将内存检测从基于内容的签名中推开。尽管 sleep_mask
可以编码 Beacon 的数据和代码（如果代理在 RWX 内存中），静态存根仍然是基于内容的内存搜索的目标。

为了解决这个问题，我们通过可从 Cobalt Strike 武器库下载的工具包使 **sleep_mask** 存根 **可由**
用户定义。可以在此处找到有关此功能的完整详细信息以及如何使用它，就像反射加载程序的更改一样，我们计划在单独的博客文章中详细介绍。

 **重新连接按钮**  
毫无疑问，Cobalt Strike 待办事项中最需要的更改是添加了重新连接按钮。你问（经常），我们听了——但我们也让你做得更好。我们不 _只是_
给你一个重新连接按钮。如果您的 Cobalt Strike
客户端检测到团队服务器已断开连接，它将尝试自动重新连接。自动重新连接尝试将重复进行，直到重新建立连接，或者您选择停止该过程。如果断开连接是用户通过菜单、工具栏或切换栏服务器按钮启动的，则会出现重新连接按钮，这允许您手动重新连接到团队服务器。

 **新连接对话框上的别名字段**  
许多用户希望我们添加一种方法来保留在主 Cobalt Strike UI
的服务器切换栏上重命名团队服务器时指定的别名，因此我们更改了新连接对话框以促进这一点。添加新连接时，您现在可以为该团队服务器指定别名。别名是您将在
Cobalt Strike 主 UI
的切换栏上看到的内容。同样，如果您在切换栏上重命名团队服务器，这将更新该连接的别名。此更改将反映在“新建连接”对话框中。您可以选择在“新建连接”对话框中的别名和连接视图之间切换。

 **在 UI 中查看 Malleable C2 配置文件**  
我们用户要求的另一项更改是添加一种方法，可以在 Cobalt Strike UI 中查看您的团队服务器正在使用的 Malleable
C2配置文件。这个新选项可以在帮助菜单中找到。

 **c2lint 返回代码**  
我们更新了 c2lint 以在完成时返回结果代码，可以在编写脚本时对其进行解析。返回代码是：

0 如果 c2lint 完成且没有错误

1 如果 c2lint 完成并出现警告

2 如果 c2lint 完成并出现错误

3 如果 c2line 完成同时出现警告和错误

如果发现任何错误和警告，也会显示检测到的错误和警告的数量。

 **sHost 轮换策略更新**  
4.3 版本中添加的故障转移主机轮换策略已得到改进，可以在决定是否需要执行故障转移之前解析响应内容和返回代码。这种变化使策略更加可靠。

 **允许特定用户代理请求**  
我们在 **Malleable**  C2 配置文件中的 http-config 块中添加了一个 **allow_useragents**
选项，以补充4.3 版本中添加的 **block_useragents**
选项。这个新选项使您可以更好地控制要响应的用户代理。请注意，这些设置是独占的。您不能在同一个 **Malleable**  C2 配置文件中同时为
**allow_useragents** 和 **block_useragents** 指定值。

除了这些更新之外，还有许多其他较小的更改和错误修复。如需完整列表，请参阅发行说明。  

    
    
    Cobalt Strike 发行说明  
    -------------  
    欢迎使用 Cobalt Strike 4.x。以下是您想立即了解的一些事项：  
      
    1. Cobalt Strike 4.x 与 Cobalt Strike 3.x 不兼容。新站起来  
       基础设施并迁移对它的访问。不更新 3.x 基础架构  
       到 Cobalt Strike 4.x。  
      
    2. 不要将 Cobalt Strike.auth 文件从 Cobalt Strike 3.x 移动到 4.x。两个文件  
       格式不兼容。  
      
    3. 为 Cobalt Strike 3.x 编写的攻击者脚本可能需要更改才能使用   
       Cobalt Strike 4.x. 请参考本指南更新您的脚本：  
      
       https://www.cobaltstrike.com/aggressor-script/migrate.html  
      
    2021 年 8 月 4 日 - Cobalt Strike 4.4  
    -------------  
    + 添加对用户定义反射加载器的支持。  
      https://www.cobaltstrike.com/help-user-defined-reflective-loader  
    + 添加对用户定义睡眠屏蔽的支持。  
      https://www.cobaltstrike.com/help-sleep-mask-kit  
    + 产品许可和安全增强。  
    + 避免使用 localhost Sysmon 事件 22 进行 Beacon 元数据解析。  
    + 使用 sleep_mask 集验证信标是否有足够的代码洞空间。  
    + 更新 Mimikatz (2.2.0 20210724)  
    + 使用证书/子域信息更新 Cobalt Strike 更新程序  
    + 添加客户端重连选项  
    + 通过 NanoHTTPD 发送数据时添加缓冲  
    + 更新链接命令的信标帮助  
    + 更新 c2lint 以返回结果代码  
    + 向 UI 添加新对话框以查看 Malleable C2 配置文件  
    + 为用户代理过滤器添加“允许”选项；补充了 4.3 中添加的块  
    + 为服务器添加别名字段到登录对话框  
    + 为连接对话框添加别名  
    + 在 Cobalt Strike 主屏幕上的连接选项卡上添加别名  
    + 增强编码签名功能的 c2lint 和 UI 处理  
    + 增强故障转移主机轮换策略（http/s 200 响应无效数据为失败）  
    + 添加鱼叉式网络钓鱼电子邮件模板解析验证以发送客户端操作  
    + UI：连接对话框的增强请求以记住上次连接的团队服务器  
    + 为代码签名配置添加更好的 C2 linting  
    + 使用编译的 Artifact 套件构建信标时校验和失败  
    + 漏洞报告：团队服务器被过大截图轰炸时崩溃。（添加 TeamServer.prop 配置）  
    + 修复武器库构建脚本中的错误（添加 bin/bash 指令）  
    + 修复 UI 中未编辑所需表格行选择的各个位置。  
    + 修复当侦听器的主机条目末尾包含空格时的信标错误（修剪主机条目字符串）  
    + 单击屏幕截图/击键选项卡不会立即聚焦列表  
    + 修复了“listener_create_ext”攻击函数中缺少的主机轮换“策略”选项文档

  

  

    
    
    # Cobalt Strike 4.4 (August 04, 2021)   
    7af9c759ac78da920395debb443b9007fdf51fa66a48f0fbdaafb30b00a8a858	Cobalt Strike 4.4 Licensed (cobaltstrike.jar)  
      
    # Distribution Packages (released with Cobalt Strike 4.4)  
    5adf9d086a2f59be9095458f207de9e947a05696e63365a4da02acdc17caa130	Cobalt Strike MacOSX Distribution Package (20210804)  
    8331a77fb2f81ce969795466f8f441f02813789c24b47d0771ffdceddf8d91fe	Cobalt Strike Linux Distributions Package (20210804)  
    fdcc265fcf1d87bdfd0f7ea91138d7d9f8128f8ed157d427317619002aadd17d	Cobalt Strike Windows Distribution Package (20210804)

  

  

下载地址：关注群聊信息  

  

![]()

  

  

密码：利刃信安  

  

https://www.virustotal.com/gui/search/7af9c759ac78da920395debb443b9007fdf51fa66a48f0fbdaafb30b00a8a858

  

https://www.virustotal.com/gui/search/fdcc265fcf1d87bdfd0f7ea91138d7d9f8128f8ed157d427317619002aadd17d

  

https://s.threatbook.cn/search?query=7af9c759ac78da920395debb443b9007fdf51fa66a48f0fbdaafb30b00a8a858&type=sha256

  

https://s.threatbook.cn/search?query=fdcc265fcf1d87bdfd0f7ea91138d7d9f8128f8ed157d427317619002aadd17d&type=sha256

  

![](https://gitee.com/fuli009/images/raw/master/public/20210808101824.png)

  

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

Cobalt Strike 4.4 (August 04, 2021)

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

