#  CobaltStrike非注入截屏BOF

刨洞之眼  [ 刨洞之眼 ](javascript:void\(0\);)

**刨洞之眼** ![]()

微信号 gh_d8f9af93f3da

功能介绍 专注网络安全领域信息分享

____

___发表于_

收录于合集

前言

`Cobalt
Strike`的截屏，获取`hash`等功能都是靠反射型`dll`注入完成的，当我们使用`screenshot`功能时，都会注入到一个进程然后再执行截屏的相关代码，而进程注入是杀软监测的一个重点，比如`360`(开启核晶)，`Windows
Defender`

目前绕过的主要方法有两个:

1.修改`screenshot.dll`，将其改为不注入执行，比较麻烦

2.自己用`cs`的`api`实现一个截屏的功能

这里为了方便选择用`inline-execute`功能自己实现。代码我参考单纯师傅的项目，进行了少量优化和更改。

  

特点

1.无进程注入，所有操作发生在当前进程中，不能是控制台会话

2.将`bmp`转为`jpg`，大幅度减小截屏的大小

3.修复一个截屏时存在的一个长期痛点，即`Windows`全局缩放启动时，不能获取完整截图

4.修复两处内存泄露和`vs2019 x86`编译问题

5.添加了一个`cna`脚本方便使用

6.将去除调试信息的`ps1`文件又加了回来

  

使用效果

获取的截屏，还是在原来查看的窗口查看

    
    
    beacon> screenshot_plus  
    [*] Running screenshot without injection  
    ...

![](https://gitee.com/fuli009/images/raw/master/public/20230617194309.png)

项目地址：  

  * 

    
    
    https://github.com/baiyies/ScreenshotBOFPlus

  

![](https://gitee.com/fuli009/images/raw/master/public/20230617194310.png)  

  

加我微信好友，邀请你进交流群

  

![](https://gitee.com/fuli009/images/raw/master/public/20230617194311.png)

  

  

  

往期推荐

  
  
[

【紧急通知！】广东电信无信号

](https://mp.weixin.qq.com/s?__biz=Mzk0MTQ4NTU5OA==&mid=2247483888&idx=1&sn=ab630bfa5bd9509f2df0f58638c39a81&chksm=c2d0fa81f5a7739755c780a9a3818684e3e1bd85f7920ad226e74ad04344572b0a5b44888da5&scene=21#wechat_redirect)[

【威胁情报】某主流数据库连接软件疑似有在野漏洞利用

](https://mp.weixin.qq.com/s?__biz=Mzk0MTQ4NTU5OA==&mid=2247483854&idx=1&sn=49d0f17ffa4c3257c0a11940ecf57727&chksm=c2d0fabff5a773a930cc10d215db2110a4362a67c3bef0ee4445d5928bd0f723227b295873f5&scene=21#wechat_redirect)[

最新版Proxifier注册码分享

](https://mp.weixin.qq.com/s?__biz=Mzk0MTQ4NTU5OA==&mid=2247483842&idx=1&sn=28b89b1374b3cb9c339851bf753aa549&chksm=c2d0fab3f5a773a5ea8ede2f3463521e7b99a1c721de6b4b747b564b027c8292d05823797f10&scene=21#wechat_redirect)[

BlackHat 2023 PPT打包

](https://mp.weixin.qq.com/s?__biz=Mzk0MTQ4NTU5OA==&mid=2247483790&idx=1&sn=97b49ab75e625fbcd97dc6f85c77bfe5&chksm=c2d0fafff5a773e9cec4c12108eb97e3057a7997ee3d12398397c0ae5f6b17cc78ce42686a92&scene=21#wechat_redirect)[

【复现成功】成功构造poc让微信崩溃了！

](https://mp.weixin.qq.com/s?__biz=Mzk0MTQ4NTU5OA==&mid=2247483674&idx=1&sn=613df2ab5b54d2229c6d62515bd7928c&chksm=c2d0fa6bf5a7737d25b949f02c92dc05fc0ba6f87b3ff62a5d6c8a4e3e7fc106758360ac968a&scene=21#wechat_redirect)[

【最新消息】CobaltStrike 4.8先别用了！

](https://mp.weixin.qq.com/s?__biz=Mzk0MTQ4NTU5OA==&mid=2247483655&idx=1&sn=81db939b6f58a3613176b778ea6d5245&chksm=c2d0fa76f5a7736004df158045114b99891073e6e94874b9eb393143c96aa285f949d2c2aee6&scene=21#wechat_redirect)

文章号，欢迎关注

![](https://gitee.com/fuli009/images/raw/master/public/20230617194312.png)

  

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

