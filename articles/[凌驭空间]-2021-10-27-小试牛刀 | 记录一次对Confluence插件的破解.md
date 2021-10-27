#  小试牛刀 | 记录一次对Confluence插件的破解

原创 EvilChen [ 凌驭空间 ](javascript:void\(0\);)

**凌驭空间** ![]()

微信号 OVERSPACE_TEAM

功能介绍 「我们希望拥有驾驭网络空间的安全能力，并以此能力回馈网络空间，贡献自己的力量。」

____

__

收录于话题

## 记录一次对Confluence插件的破解

这是一款基于Confluence开发的水印插件。

## 源码获取

你可以通过Confluence的Marketplace下载到插件，插件是obr格式的，没见过，但经验熟悉的一般都知道这种插件包直接以压缩包格式打开即可（
**或者你可以用binwalk来看看** ）

  

![](https://gitee.com/fuli009/images/raw/master/public/20211027083906.png)

  

使用压缩包格式打开，发现存在Jar文件，直接把Jar文件拖出来分析/破解。

## 分析/破解过程

使用Luyten这个工具反编译获取代码直接进行分析，由于没写过Confluence插件，所以不知道入口在哪（ **懒得翻文档**
），直接找类似于配置文件去看下，找到一个 **atlassian-plugin.xml** 文件：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211027083907.png)

  

根据这文件内容的大概意思便知道这就是插件功能实现的地方，接着在反编译的代码中继续跟进即可。

  

不过在这里我没有看具体代码，直接全局搜索license、key、valid等关键词寻找关键方法，这里发现 **hasValidLicense**
函数被其他地方调用，根据中文意思，也能猜出这是 **校验程序许可证是否有效** 的地方：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211027083908.png)

  

所以我们可以直接入手修改这个函数的返回值，可以看见该函数是返回布尔类型的，所以在这里我们可以将函数的返回值修改为：

  

  * 

    
    
    return true;

  

这样在理论上就可以让所有的许可证都过校验，也就完成了破解的整个步骤。

  

但这都只是停留在理论层，我们需要付诸于行动，你可以选择导出源码然后进行修复、修改、打包...但是为了图方便，我打算借助工具 **jbytemod**
直接修改字节码的方式进行修改，但是在这里，我不了解Java字节码的语法，为了节约时间，直接以源码对照字节码的方式来现场看一下。

  

首先我们要知道 **return true;** 的字节码是什么，所以我们可以先直接全局搜索就可以找到代码中已经实现的方法：  

  

![](https://gitee.com/fuli009/images/raw/master/public/20211027083909.png)

  

然后再使用 **jbytemod** 找到对应的字节码片段即可：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211027083911.png)

  

这时候再多看几处相同的代码你就会发现这段字节码就对应着 **return true** ;：

  

  *   * 

    
    
    iconst_1ireturn

  

所以我们按照这个格式替换掉 **hasValidLicense** 函数的字节码即可：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211027083912.png)

  

最后保存到Jar文件，并替换原Jar文件到obr文件中即可（压缩包替换的方式）。

  

## 导入插件

接着我们需要验证下当前的破解是否成功，首先访问自己的Confluence平台导入修改好插件，其次随便输入一段认证码内容，最后发现输入完成之后即可直接使用该插件的全部功能：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211027083913.png)

  

综上所述，我们幸运的完成了这一次的破解。  

## 声明

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，凌驭空间（OverSpace）安全团队及文章作者不为此承担任何责任。

  

凌驭空间（OverSpace）安全团队拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经凌驭空间（OverSpace）安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  

![](https://gitee.com/fuli009/images/raw/master/public/20211027083914.png)

  

![]()

EvilChen

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

小试牛刀 | 记录一次对Confluence插件的破解

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

