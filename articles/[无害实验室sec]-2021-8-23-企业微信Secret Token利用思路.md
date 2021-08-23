##  企业微信Secret Token利用思路

[ 无害实验室sec ](javascript:void\(0\);)

**无害实验室sec** ![]()

微信号 WUHAISEC

功能介绍 不接商业培训广告

____

__

收录于话题

编者荐语：

针对企业微信Secret Token利用思路

以下文章来源于漏洞推送 ，作者kkk mr

![漏洞推送](http://wx.qlogo.cn/mmhead/Q3auHgzwzM5rcpX1zHSzDZXwKXwcHAFiaANiae0icV19exnFU74AqK7YA/0)
**漏洞推送**

专注于安全漏洞、威胁情报发掘。

在一次渗透过程中，获得了一台服务器的主机权限，但是提权未果。不得已只能翻一下主机上面的文件，看一下有没有什么可以利用的地方。在其中一个网站源码里面，翻到了这么一段代码

  *   *   * 

    
    
    <add name="CorpId" connectionString="wxxxxxxxxxxxx" />    <add name="CorpSecret" connectionString="xxxxxxxxxxx" />        <add name="CorpToken" connectionString="xxxxxxxxxx" />

百度了一下，发现是企业微信的开发密钥

![](https://gitee.com/fuli009/images/raw/master/public/20210823210407.png)

阿里云的密钥大家都知道，拿到了密钥基本上等于拿到了阿里云账号的控制权限，直接可以执行命令的。

企业微信的密钥，还是接触比较少，翻了一下企微的开发API，发现可以通过这个ak添加企微用户。

## 第一步：获取AccessToken

API地址：`https://qyapi.weixin.qq.com/cgi-
bin/gettoken?corpid=id&corpsecret=secrect`

传入获取到的id和secrect即可获取AccessToken

![](https://gitee.com/fuli009/images/raw/master/public/20210823210411.png)

## 第二步：获取部门ID

企微中会分为一个个的部分，通过企微的API我们可以获取到企业的架构和部门ID，这个在添加成员的时候用的到。

在`https://open.work.weixin.qq.com/devtool/query?e=301002`中查询上一部获取到的ak权限就能，查询到部门名称，以及部门ID

![](https://gitee.com/fuli009/images/raw/master/public/20210823210412.png)

## 第三步：添加企微成员

![](https://gitee.com/fuli009/images/raw/master/public/20210823210413.png)

往`https://qyapi.weixin.qq.com/cgi-bin/user/create?access_token=ACCESS_TOKEN`

  *   *   *   *   *   * 

    
    
    {   "userid": "zhangsan",   "name": "张三",   "department": [6],   "mobile":"1388888888"}

调用成功后，即可通过手机号登录目标企业的企业微信。可选择企业微信通讯录中选择高管、信息部门等姓名进行伪装。然后再采取社工钓鱼的方式，信任度和成功率都会高很多。

## 第四部：善后

当使用手机号登录成功以后，可通过修改成员API来修改姓名进行伪装，并且可以将手机号修改为空来防溯源。

![](https://gitee.com/fuli009/images/raw/master/public/20210823210414.png)

最后，删除成员以绝后患

![](https://gitee.com/fuli009/images/raw/master/public/20210823210415.png)

> 企业微信API文档地址
>
>
> https://qydev.weixin.qq.com/wiki/index.php?title=%E7%AE%A1%E7%90%86%E6%88%90%E5%91%98#.E6.9B.B4.E6.96.B0.E6.88.90.E5.91.98

## 最后  

 **由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

  

 **无害实验室sec
拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**

  

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

企业微信Secret Token利用思路

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

