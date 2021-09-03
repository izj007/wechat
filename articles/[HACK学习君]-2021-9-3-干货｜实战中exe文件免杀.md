#  干货｜实战中exe文件免杀

原创 cacker  [ HACK学习君 ](javascript:void\(0\);)

**HACK学习君** ![]()

微信号 XHacker1961

功能介绍
HACK学习，专注于网络安全攻防与黑客精神，分享技术干货，代码审计，安全工具开发，实战渗透，漏洞挖掘，网络安全资源分享，为广大网络安全爱好者和从业人员提供一个交流学习分享的平台

____

__

收录于话题 #免杀 ,2个

**0X00     资源处理法**

  

![](https://gitee.com/fuli009/images/raw/master/public/20210903130905.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210903130908.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210903130909.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210903130911.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210903130912.png)

推荐阅读：

https://www.cnblogs.com/claidx/p/7354034.html

小结：  
一定要尽量不加，不减，保证文件能用的情况下，大幅度的改。

  

 **0X01     shellcode处理法**

 **  
**

使用异或+随机数字混淆的方式，加密方式可以自定义，尽量做到小众、独创，然后只要在运行时解码就行，也可以想办法利用加载器加载加密到txt里面到shellcode或者其他加密的资源文件的加密shellcode，也可以实现绕过静态查杀。

![](https://gitee.com/fuli009/images/raw/master/public/20210903130914.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210903130915.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210903130917.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210903130918.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210903130920.png)

  

 **0X02    远程线程注入：**

  

1.远程注入流程：

在进程A中创建远程线程，将线程函数指向为LoadLibrary();

具体实现步骤：

  1. 在进程A中分配空间，存储“A.DLL”

  2. 获取LoadLibrary函数的地址

  3. 创建远程线程，执行LoadLibrary();

![](https://gitee.com/fuli009/images/raw/master/public/20210903130921.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210903130923.png)

  

 **0X03     动态绕过杀软**  

 **  
**

可以使用远程加载shellcode的方式，但是记得shellcode一定要加密处理，写成冲锋马的形式。

  

远程加载shellcode容易暴露自己的文件服务器地址，建议加一个域前置，可以用来隐藏服务器地址。

  

通过域前置的方法可以有效的隐藏我们C2地址或加载器的文件服务器。同时此类CDN地址可能存在于白名单，也可用于绕过流量监测。

![](https://gitee.com/fuli009/images/raw/master/public/20210903130924.png)

  

 **推荐阅读**  

  

https://www.cnblogs.com/zpchcbd/p/12170851.html

  

https://payloads.online/archivers/2019-11-10/2/

  

https://uknowsec.cn/posts/notes/ShellCode%E8%BF%9C%E7%A8%8B%E5%8A%A0%E8%BD%BD%E5%99%A8%E6%94%B9%E9%80%A0%E8%AE%A1%E5%88%92.html

  

 **点赞，转发，在看**

原创投稿作者：cacker

![](https://gitee.com/fuli009/images/raw/master/public/20210903130925.png)

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

干货｜实战中exe文件免杀

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

