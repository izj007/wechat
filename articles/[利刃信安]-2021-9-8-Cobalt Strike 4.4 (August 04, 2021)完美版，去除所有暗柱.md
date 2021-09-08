#  Cobalt Strike 4.4 (August 04, 2021)完美版，去除所有暗柱

原创 利刃信安 [ 利刃信安 ](javascript:void\(0\);)

**利刃信安** ![]()

微信号 LRXAEGZ

功能介绍 利刃信安

____

__

收录于话题 #远程控制 ,3个

Cobalt Strike 4.4 (August 04, 2021)完美版，去除所有暗柱  

  

    
    
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

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210908121040.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210908121046.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210908121047.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210908121049.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210908121050.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210908121051.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210908121053.png)

  

注意：使用的时候自行添加【已内置】

# TeamServer.prop

在 4.4 版本之后，您可能在启动团队服务器时注意到一条警告消息：

![](https://gitee.com/fuli009/images/raw/master/public/20210908121054.png)

丢失的文件是可选的，它的缺失不会破坏团队服务器。它包含许多可选参数，可用于自定义用于验证屏幕截图和键盘日志回调数据的设置，从而允许您调整“HotCobalt”漏洞的修复程序。您可以通过创建一个名为
TeamServer.prop 的空文件并将其保存在 Cobalt Strike 目录中来抑制警告。

一个示例 TeamServer.prop 文件可以从 Cobalt-Strike/TeamServerProp GitHub
存储库下载在这里。我们建议创建一个空的“TeamServer.prop”文件，创建该文件但使用默认设置，或者只是忽略警告。但是，如果您想更改这些设置，现在可以这样做。

默认 TeamServer.prop 文件包含以下内容：

`#Cobalt Strike Team Server Properties`

`#Fri May 07 12:00:00 CDT 2021`

`# ------------------------------------------------`

`# Validation for screenshot messages from beacons`

`# ------------------------------------------------`

`# limits.screenshot_validated=true`

`# limits.screenshot_data_maxlen=4194304`

`# limits.screenshot_user_maxlen=1024`

`# limits.screenshot_title_maxlen=1024`

`# Stop writing screenshot data when Disk Usage reaches XX%`

`# Example: Off`

`#          "limits.screenshot_diskused_percent=0"`

`# Example: Stop writing screenshot data when Disk Usage reaches 95%`

`#          "limits.screenshot_diskused_percent=95"`

`# Default:`

`# limits.screenshot_diskused_percent=95`

`# ------------------------------------------------`

`# Validation for keystroke messages from beacons`

`# ------------------------------------------------`

`# limits.keystrokes_validated=true`

`# limits.keystrokes_data_maxlen=8192`

`# limits.keystrokes_user_maxlen=1024`

`# limits.keystrokes_title_maxlen=1024`

`# Stop writing keystroke data when Disk Usage reaches XX%`

`# Example: Off`

`#          "limits.keystrokes_diskused_percent=0"`

`# Example: Stop writing keystroke data when Disk Usage reaches 95%`

`#          "limits.keystrokes_diskused_percent=95"`

`# Default:`

`# limits.keystrokes_diskused_percent=95`

  * 以“#”开头的行是注释。

  * limit.*_data_maxlen是将被处理的屏幕截图/keylog 数据的最大大小。超过此限制的回调将被拒绝。

  * limit.*_validated=false表示忽略以下三个“ …_maxlen ”设置

  * 将任何“ ..._maxlen ”设置设置为零将禁用该特定设置

  * limit.*_diskused_percent设置回调处理的阈值。当磁盘使用率超过指定百分比时回调被拒绝

    * limit.*_diskused_percent=0（零）禁用此设置

    * 有效值为 0-99

  

Cobalt Strike 4.4 现已推出。此版本为您提供了更多控制权，提高了 Cobalt Strike
的回避质量，并解决了我们用户要求的一些较小的更改……是的！我们添加了重新连接按钮！

### 用户定义的反射 DLL 加载程序

Cobalt Strike
在其反射加载基础方面具有很大的灵活性，但它确实有局限性。我们已经看到很多社区对这个领域的兴趣，所以我们做了一些改变，让你完全绕过它，而是定义你自己的反射加载过程。默认的反射加载器仍可随时使用。

我们扩展了 4.2 版本中最初对反射加载器所做的更改，为您提供了一个 Aggressor 脚本挂钩，允许您指定自己的反射加载器并完全重新定义 Beacon
加载到内存中的方式。提供了一个攻击者脚本 API
来促进这个过程。这是一个巨大的变化，我们计划跟进一个单独的博客文章来更详细地介绍这个功能。目前，您可以在此处找到更多信息。用户定义的反射加载器套件可以从
Cobalt Strike 武器库下载。

### 避免使用本地主机 Sysmon 事件 22 进行信标元数据解析

当 Beacon 启动时，它会解析元数据以发送回 Cobalt Strike。以前，Beacon
在成熟的环境中像拇指一样突出，因为用于解析此元数据的方法触发了 Sysmon 事件 22（DNS 查询），并且已成为每次运行时可靠地对 Beacon
进行指纹识别的方法。4.4 版本修改了此元数据的解析方式，以便不再发生这种情况。

### 用户定义的 sleep_mask 掩码/取消掩码 Stub

该sleep_mask是钴罢工的掩盖和揭露本身的记忆能力。此功能的目标是将内存检测从基于内容的签名中推开。尽管 sleep_mask 可以编码 Beacon
的数据和代码（如果代理在 RWX 内存中），静态存根仍然是基于内容的内存搜索的目标。

为了解决这个问题，我们通过可从 Cobalt Strike
武器库下载的工具包使sleep_mask存根可由用户定义。可以在此处找到有关此功能的完整详细信息以及如何使用它，就像反射加载程序的更改一样，我们计划在单独的博客文章中详细介绍。

### 重新连接按钮

毫无疑问，Cobalt Strike 待办事项中最受要求的更改是添加了重新连接按钮。你问了（问了，问了！），我们听了——但我们不 _只是_
给你一个重新连接按钮。如果您的 Cobalt Strike
客户端检测到团队服务器已断开连接，它将尝试自动重新连接。自动重新连接尝试将重复进行，直到重新建立连接，或者您选择停止该过程。

![](https://gitee.com/fuli009/images/raw/master/public/20210908121055.png)

如果断开连接是用户通过菜单、工具栏或切换栏服务器按钮启动的，则会出现重新连接按钮，这允许您手动重新连接到团队服务器。  

![](https://gitee.com/fuli009/images/raw/master/public/20210908121056.png)

### 功能请求

我们很乐意听取用户的意见——包括对有效方法的反馈，以及对无效事物的更改请求。除了重新连接按钮之外，我们还在此版本中进行了一些更改，以专门解决您提出的问题。

许多用户希望我们在 Cobalt Strike 主 UI
中的服务器切换栏上添加一种在重命名团队服务器时指定别名的方法，因此我们更改了“新建连接”对话框以促进这一点。添加新连接时，您现在可以为该团队服务器指定别名。别名是您将在
Cobalt Strike 主 UI
的切换栏上看到的内容。同样，如果您在切换栏上重命名团队服务器，这将更新该连接的别名。此更改将反映在“新建连接”对话框中。您可以选择在“新建连接”对话框中的别名和连接视图之间切换。

![](https://gitee.com/fuli009/images/raw/master/public/20210908121057.png)

我们用户要求的另一项更改是添加一种方法，可以在 Cobalt Strike UI 中查看您的团队服务器正在使用的 Malleable C2
配置文件。这个新选项可以在帮助菜单（Help -> Malleable C2 Profile）中找到。

![](https://gitee.com/fuli009/images/raw/master/public/20210908121058.png)

最后要提及的一个功能请求是，我们已更新 c2lint 以在完成时返回结果代码，可在编写脚本时对其进行解析。返回代码为：
如果发现任何错误和警告，也会显示检测到的错误和警告的数量。  
`  
0 if c2lint completes with no errors  
1 if c2lint completes with warnings  
2 if c2lint completes with errors  
3 if c2lint completes with both warnings and errors  
`  
  

### 漏洞修复 (CVE-2021-36798)

在 Cobalt Strike 中发现了一个拒绝服务 (DoS) 漏洞 (CVE-2021-36798)。该漏洞已在 4.4
版本范围内修复。可以在此处找到更多信息。

### 其他增强功能

4.3 版本中添加的故障转移主机轮换策略已得到改进，可以在决定是否需要执行故障转移之前解析响应内容和返回代码。这种变化使策略更加可靠。

最后要提到的一个变化是在Malleable C2 配置文件中的 http-config 块中添加了一个allow_useragents选项，以补充4.3
版本中添加的block_useragents选项。这个新选项使您可以更好地控制要响应的用户代理。请注意，这些设置是独占的。您不能在同一个 Malleable
C2 配置文件中同时为 allow_useragents 和 block_useragents 指定值。

要查看 Cobalt Strike 4.4 新功能的完整列表，请查看 发行说明。获得许可的用户可以 运行更新程序 以获取最新版本。要购买 Cobalt
Strike 或询问评估选项，请 联系我们 以获取更多信息。

下载地址：关注公众号回复CS4.4即可获取。  

![]()

利刃信安

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

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

Cobalt Strike 4.4 (August 04, 2021)完美版，去除所有暗柱

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

