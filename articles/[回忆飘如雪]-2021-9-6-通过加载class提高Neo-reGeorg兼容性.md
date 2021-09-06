#  通过加载class提高Neo-reGeorg兼容性

原创 c0ny1 [ 回忆飘如雪 ](javascript:void\(0\);)

**回忆飘如雪** ![]()

微信号 gv7_me

功能介绍 记录工作生活所思所想

____

__

收录于话题 #安全开发 ,1个

**  0x01 **

 **背景**

  

一大早就看到L-codes师傅发消息说，Neo-reGeorg jsp服务端又出现问题了，印象里已经不是一两次了。大部分都是兼容性问题，这次也不例外。

![](https://gitee.com/fuli009/images/raw/master/public/20210906120658.png)是时候设计一个一劳永逸的方案了。  
 **  0x02 ** **分析原因**  
我们知道jsp从被访问到运行，经历如下阶段。![](https://gitee.com/fuli009/images/raw/master/public/20210906120703.png)本案例中发现tomcat
work目录下已经存在了tunnel_jsp.java,但是没有tunnel_jsp.class，说明阶段1已经过。结合页面报错信息，在2阶段时Tomcat内置的编译器JDTCompiler，编译报错了。  
  
检查tunnel_jsp.java代码并没有语法错误，尝试使用javac编译，编译成功！看来JDTCompiler与javac实现逻辑并不同，而且没有javac强大。![](https://gitee.com/fuli009/images/raw/master/public/20210906120705.png)编译成功之后我再访问tunnel.jsp页面不再报错了。可见提高一个.jsp的兼容，无非就是让它在各个中间件下成功变成一个.class。而这个过程与具体中间件的jsp转换器的解析机制，java编译器的编译机制和servlet-
api的版本息息相关。  
  
那么我们是不是可以把Neo-reGeorg的服务端代码提取变成class字节码，然后jsp来加载和调用，来提高这个过程的成功率呢？。 **  
** **总之核心思想就是把尽可能多的业务逻辑变成最终可运行的java字节码，同时尽可能的减少jsp代码，少用api少用语法糖少用特性。**  
 **  0x03 ** **编码实现**  
我们先来移植服务端模版代码为java代码。直接新建一个NeoreGeorg.java，将jsp中的方法直接copy,主体代码的移植需要注意2个问题。  
第一、参数提炼问题。我们需要把模版变化的地方，提取出来作为参数，比如X-CMD这样的指令，POST request read filed这样的提示，Neo-
reGorg需要通过随机替换它们实现流量加密。  
第二、参数传递问题。参数可以通过构造方法或者自定义方法传递进来，但是这样要多写些反射代码。本着jsp代码越少越好原则，使用每个类都有的equal(java.lang.Object)方法。![](https://gitee.com/fuli009/images/raw/master/public/20210906120707.png)为了兼容更多的jdk版本我们这里选择使用1.5编译，同时为了class体积更小，可以使用-g:none去掉调试信息。![](https://gitee.com/fuli009/images/raw/master/public/20210906120710.png)jsp部分很简单，定义一个classloader用于加载class，然后将该class
newInstance进行调用。有二个点可以简单讲讲。  
第一，class字节码的存储方式问题。本着少用api的原则，我直接用byte数组存储。当然如果字节码太多，可能会有The code of method
_jspService(...) is exceeding the 65535 bytes limit报错问题，推荐用hex编码解决。  
第二，全局存储class对象问题。推荐使用application对象，而不是session对象进行存储，否则遇到负载的情况就麻烦了。![](https://gitee.com/fuli009/images/raw/master/public/20210906120712.png)经过测试在各个中间件下稳定运行，顺手给L-
codes师傅一个pr。  
 **  0x04 ** **总结**  
其实这个方法可以使用很多jsp脚本的改造，比如内存马注入jsp，jsp大马，蚁剑一句话木马等等。大家可以照猫画虎，自行修改。  

![]()

c0ny1

如有帮助，打赏c0ny1一瓶1664吧!

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

如有帮助，打赏c0ny1一瓶1664吧!

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

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

通过加载class提高Neo-reGeorg兼容性

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

