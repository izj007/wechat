#  【技术分享】Burpsuite插件改造计划

原创 白猫  [ 安全客 ](javascript:void\(0\);)

**安全客** ![]()

微信号 anquanbobao

功能介绍 打破黑箱 客说安全

____

__

收录于话题

#BurpSuite ,5

#开发 ,1

#Java ,5

![](https://gitee.com/fuli009/images/raw/master/public/20210914095127.png)作者：天启@涂鸦智能安全实验室  
![]()

前言

之前逛GitHub的时候看到一个老同事写的一个插件，于是就多喵了两眼，然后发现其实这个插件还是很实用的！于是就安装使用了！简而言之该插件就是利用正则来从数据包中匹配，如果匹配到了规则则高亮显示！该插件也支持规则导入导出，综合来讲在实战中就非常有用的！比如去年大杀四方的shiro反序列化，它的数据包的特征就非常明显即：响应数据包字段中rememberMe=deleteMe，我们只需要写个正则，然后就可以自动化对数据包做反序列化无损化检测了！该插件的官方GitHub地址：https：//github.com/gh0stkey/HaE，使用效果如下图所示![](https://gitee.com/fuli009/images/raw/master/public/20210914095128.png)但是我在使用的过程中遇到一个很严重的问题：如果规则比较多的话，颜色就会重叠，而且我们也不能直观地看到到底匹配的什么规则！于是我就修改了一下源码将其改造成了我想要的样子！下面则是记录了我是如何从burpsuite的hello
world开始到改造插件的过程！本文使用的所有代码址：https：//github.com/ba1ma0/burpsuite_extension![](https://gitee.com/fuli009/images/raw/master/public/20210914095129.png)
![]()

入门

老规矩我们先从helloworld走起，官方已经给出了教程：https：//portswigger.net/burp/extender/writing-
your-first-burp-suite-extension![]()官方的教程我总结一下可以分为三步：

  *   *   * 

    
    
    从burpsuite中导出api文件。创建Java项目。导出Java代码的jar包安装运行。

下面就针对这三点我我介绍一下以便读者可以跟着这个自己写出helloworld，然后再改造插件，让其成为我们想要的样子。
**第一步从burpsuite中导出API文件**
第一步打开burpsuite，从Extender->APIS->SaveInterface到一个本地的文件件中![](https://gitee.com/fuli009/images/raw/master/public/20210914095131.png)

###

 **第二步创建Java项目** 创建Java项目的时候注意以下两点：

  *   * 

    
    
    创建一个burp文件夹用于存放上一步导出的API文件。在Burp文件夹中创建名为BurpExtender.java文件

创建之后我们在BurpExtender.java中写几句简单的代码即可，如下图所示![](https://gitee.com/fuli009/images/raw/master/public/20210914095132.png)

###

 **第三步导出jar包运行**
代码写完之后，我们需要将我们写的Java导出为jar包，这样方便我们在burpsuite上安装运行.接下来我用的是IntelliJ IDEA
Community版本作为演示.首先第一步打开File->Project Structure->Project
Settings->Artifacts->JAR->From modules with
dependencies...，如下图所示![](https://gitee.com/fuli009/images/raw/master/public/20210914095133.png)![](https://gitee.com/fuli009/images/raw/master/public/20210914095134.png)第二步选择Module为burp，并且Main
class地方为空即可![](https://gitee.com/fuli009/images/raw/master/public/20210914095135.png)第三步首先Build->Build
project![](https://gitee.com/fuli009/images/raw/master/public/20210914095136.png)接着Build->Build
Artifacts…->burp：jar->Build输出插件的jar包如下图所示![](https://gitee.com/fuli009/images/raw/master/public/20210914095137.png)![](https://gitee.com/fuli009/images/raw/master/public/20210914095138.png)最后安装并运行burpsuite插件运行成功，至此我们的入门阶段完成了！接下来就开始进阶部分：如何改造别人的插件让其成为我们想要的样子！![](https://gitee.com/fuli009/images/raw/master/public/20210914095139.png)

  

![]()

进阶

首先我们在脑海中要有一个概念，我们想要的效果是将颜色和对应的正则描述name全部在burpsuite中显现出来！而且此时正则和颜色是一一对应的，也就说我们如果能搞清楚颜色color生成逻辑，大概率也是能够按照相同规律生成对应的正则描述name，并且将其显示出来的！所以第一步我们就先在代码中全局搜索一下color如下图所示，还记得入门部分我们提到的BurpExtender.java吗?该文件是插件的入口文件，可以将功能类同于main函数！我们要搞懂color的生成逻辑，最开始的要从这个入口文件中查起！![](https://gitee.com/fuli009/images/raw/master/public/20210914095141.png)进入该入口文件之后，我发现color第一被使用出现在114行，被函messageInfo.setHighlight(color)
直接处理那我们就直接跟进该函数进去看看，而进去之后发现了一个意外之喜.为什么这么说呢?文章前面说过，我们需要将颜色和正则描述全部显示出来，而burpsuite展示界面唯一一个让用户修改的列是comment，我们在IHttpRequestResponse.java直接看到了官方已经将修改comment的接口setComment(String
comment)定义好了，那么我们只需要调用即可！![](https://gitee.com/fuli009/images/raw/master/public/20210914095143.png)![](https://gitee.com/fuli009/images/raw/master/public/20210914095144.png)那接下来我们就只需要了解color的生成以及调用逻辑，然后按照相同的逻辑同时生成对应的name即可，最后再调用setComment(String
comment)就能实现我们想要的结果！接下来我捋了一下所有跟color有关的变量并且最终定位到了最原始的变量obj如下图所示![](https://gitee.com/fuli009/images/raw/master/public/20210914095145.png)变量obj又诞生于ec.matchRegex中，那我们继续跟进进入ExtractContent.java，进入之后我们发现这里就是最原始的color产生的地方，且color被存放在一个map中，obj其实就是存放color的map，所以要想要想实现color和name一一对应地展示出来，只要在此处将name和color对应关系一起保存在map中返回即可，原作者这里其实是没有将这个name一起返回的，我们在这里添加几行代码将其返回即可如下图所示了解了obj的产生过程，接下来我们看一下他是怎么被调用的，处理obj的是da.highlight，所以直接跟进去看一下实现逻辑！进去之后发现他的实现逻辑其实很简单，就是从obj中取出colo！color和name其实是一一对应的，所以我就按照相同的逻辑，写了跟原作者一模一样的函数将color和name按照name：color
的形式一起返回来如下所示![](https://gitee.com/fuli009/images/raw/master/public/20210914095146.png)colorList和commentList是在相同的时间和相同的地方一起产生的！那我们按照相同的顺序从他们中取出和color和name此时两个变量也肯定是一一对应的！取出两个变量之后我们再调用burpsuite官方提供的修改color和comment的两个接口就能实现我们想要的结果！![](https://gitee.com/fuli009/images/raw/master/public/20210914095147.png)一切修改完之后按照上面导出jar的流程直接导出jar运行，最终实现我们想要的结果![](https://gitee.com/fuli009/images/raw/master/public/20210914095148.png)  
![]()

后言

通过这次修改代码，我发现良好的代码规范真的很重要，尤其是这种协同的开源项目！原作者的代码非常规范，所以当我我在调试，追踪的时候就很舒服！本文作者能力有限，文章若有纰漏请读者不吝赐教！  
WeChat：5ed0c42e63c9c2145990351ccaec4da5![](https://gitee.com/fuli009/images/raw/master/public/20210914095150.png)

    
    
      
    - 结尾 -  
    精彩推荐[报警就“撕票”，Ragnar Locker勒索软件顶风作案](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649751972&idx=1&sn=c50a9aab8fa409e597272fb5585b3009&chksm=889331cbbfe4b8dd98034fc6b86200c85770cb660cc9f1adcaf1b4c871500b137493c2575343&scene=21#wechat_redirect)  
    [【技术分享】梨子带你刷burpsuite靶场之高级篇 - OAuth2.0认证专题](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649751971&idx=1&sn=fb65344a6b846aad3afc82fe42c1b17e&chksm=889331ccbfe4b8dae0c6d8b755511f85b80e3c39831d7b0c1ac4c6d57f0959cb0e18c7e23ba5&scene=21#wechat_redirect)  
    [【技术分享】梨子带你刷burpsuite靶场之高级篇 - HTTP请求走私专题](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649751885&idx=1&sn=d6ee55c39a0000a98acd713ac7eb94a3&chksm=88933122bfe4b8345a50b27c4255533549873b34d1138d52b01f16925313ac54405d77ba3533&scene=21#wechat_redirect)  
    [【技术分享】反制爬虫之Burp Suite RCE](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649751822&idx=1&sn=2d36443ca50d76244016ecf4a9451a53&chksm=88933161bfe4b8777ec506375b5c1f3195b2f7a520606455ab35a89499a3530f13cf24646dd5&scene=21#wechat_redirect)![](https://gitee.com/fuli009/images/raw/master/public/20210914095151.png)
    
    
     **戳“阅读原文”查看更多内容**

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

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

【技术分享】Burpsuite插件改造计划

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

