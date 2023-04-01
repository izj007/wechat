#  SharpExchangeKing v1.6.0

原创 Rcoil [ RowTeam ](javascript:void\(0\);)

**RowTeam** ![]()

微信号 RowTeam

功能介绍 没有什么好介绍的消遣地

____

___发表于_

收录于合集 #King 2个

## 0x00 前言

v1.6.0 版本主要是新增一个 Persistence 选项卡，其余内容均是优化及修复。

目前 SharpExchangeKing 已编写并更新的功能有以下内容：

  1. 1. Info：基于 4 中方法获取 Exchange 服务器的信息

  2. 2. Mailbox：基于 3 种方法判断用户是否存在

  3. 3. PwdSpray：基于 EWS 接口进行密码爆破

  4. 4. Persistence：基于 EWS 接口对当前登陆邮箱进行一些设置，从而更好的邮件收取

当前新增的 Persistence 选项卡，部分技术来自三好学生师傅的博客内容，我只是将他的内容用 C#
造了轮子罢了。如果想对它的原理进行学习，那么可以翻阅三好学生师傅的文章《透透基础——继续获得Exchange用户收件箱邮件的方法》。

当前公开的版本仅仅是包含了 `Info`和`PwdSpray`
两个选项卡功能：https://github.com/RowTeam/SharpExchangeKing

## 0x01 新增

[+] 从全局地址列表获取域用户邮箱

[+] 设置当前邮箱收信箱转发：指定用户

[+] 设置当前邮箱收信箱和发信箱的委派设置：可指定用户，也可以全部用户可访问

共计 8 个功能选项

![](https://gitee.com/fuli009/images/raw/master/public/20230401182142.png)总览

使用说明：

  * •  **Auto** ：必须为 "username:password" 格式。

  * •  **Value1** ：必须是 username 的邮箱名，是当前登陆的用户邮箱名，比如 rcoil@rowteam.me。根据 Funtion 选择。

  * •  **Value2** ：根据 Funtion 选择对应值。

这个模块当前支持 8 个函数功能，它们分别是：

  * •  **GetMailLists** ：从全局地址列表获取邮箱账号数据。

  * •  **GetInboxRules** ：读取用户 Value1 规则信息，从返回结果中能够获得规则对应的 RuleID。

  * •  **AddForwardToRecipients** ：创建用户 Value1 转发邮件至用户 Value2 的规则。

  * •  **DelForwardToRecipients** ：根据 RuleID 删除用户 Value1 的指定转发规则。

  * •  **GetInboxPermissions** ：查看用户 Value1 收件箱的访问权限。

  * •  **AddDelegateEditorToInboxPermissions** ：添加用户 Value2 对用户 Value1 收件箱的完全访问权限。

  * •  **RemoveDelegateEditorToInboxPermissions** ：移除用户 Value2 对用户 Value1 收件箱的访问权限。

  * •  **UpdateFolderDefaultToPermissions** ：设置所有用户都可以访问 `username`的收信箱及发信箱。

## 0x02 更新

[u] 将目标地址设置虚拟化

[u] 重点信息使用别的颜色显示

[u] 设置选项卡图标

[u] 在 about 放置了一张图片，导致程序大小 biubiubiu 的

## 0x03 效果如下

  * • 信息获取

![]()信息获取

  * • EASDate

![](https://gitee.com/fuli009/images/raw/master/public/20230401182143.png)EASDate

  * • 密码爆破

![](https://gitee.com/fuli009/images/raw/master/public/20230401182144.png)密码爆破

## 0x04 免责声明

本工具仅面向合法授权的企业安全建设行为，例如企业内部攻防演练、漏洞验证和复测，如您需要测试本工具的可用性，请自行搭建靶机环境。

在使用本工具进行检测时，您应确保该行为符合当地的法律法规，并且已经取得了足够的授权。请勿对非授权目标使用。

如您在使用本工具的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。

## 0x05 防御检测

  1. 1. 查看单个邮件用户的转发规则

访问 Exchange Control Panel(ECP)，登录，查看 organize email -> inbox rules

  1. 1. 查看单个邮件用户的访问权限

访问 Outlook Web Access(OWA)，登录，查看 Inbox -> permissions...

  1. 1. 查看所有邮件用户的收件箱转发功能

运行 Exchange Management Shell，查看命令如下：

    
    
    Get-Mailbox|Select-Object UserPrincipalName,ForwardingAddress,ForwardingSmtpAddress

  

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

