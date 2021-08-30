#  通过PELoader加载mimikatz

原创 Y4er [ ChaBug ](javascript:void\(0\);)

**ChaBug** ![]()

微信号 ChaBugSec

功能介绍 一个分享知识、结识伙伴、资源共享的公众号。

____

__

收录于话题

最近离职在家，闲着逛Twitter看到老外用dotnet core写了个peloader来加载mimikatz，学一下。

项目地址：https://github.com/secdev-01/Mimikore

![](https://gitee.com/fuli009/images/raw/master/public/20210830180306.png)

dotnet core可以打包成大文件，但是体积太大了  

    
    
    dotnet publish -r win-x64 -c Release /p:PublishSingleFile=true /p:IncludeNativeLibrariesForSelfExtract=true --self-contained true

![](https://gitee.com/fuli009/images/raw/master/public/20210830180324.png)

一个mimikatz打包下来20M，免杀有点效果，当作学习思路吧。

https://www.virustotal.com/gui/file/2a7a24e81c5672f694d2adc18a5b0c0713910d3711924a218e7cc8229e1464f1/detection

![](https://gitee.com/fuli009/images/raw/master/public/20210830180325.png)



看了看GitHub上 Casey Smith 还写了一个PELoader，可以直接用dotnet framework运行，体积比上面那个小得多。

https://github.com/rvrsh3ll/PELoader/blob/master/Loader.cs

代码差别不大，直接用就行了，这里就不提了。

一直觉得dotnet core在红队开发中应该大有可为，现在面临的最大问题就是打包体积太大，唉硬伤啊，只能等微软的进一步开发计划了。

  

凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数

![]()

Y4er

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

通过PELoader加载mimikatz

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

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

