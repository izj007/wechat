#  使用内存特征检测 Cobalt Strike

黑白天  [ 黑白天实验室 ](javascript:void\(0\);)

**黑白天实验室** ![]()

微信号 HBT-SEC

功能介绍 研究学习一切网络安全相关的技术

____

__

收录于话题

Beacon 通常是反射加载到内存中，还可以配置各种内存中混淆选项以隐藏其有效负载。

  

Beacon 可以配置各种内存中混淆选项以隐藏其有效负载。例如，obfuscate-and-sleep 选项会试图在回调之间屏蔽部分 Beacon
有效负载，以专门避开基于特征的内存扫描。  

  *   * 

    
    
    Obfuscate and Sleep是一个Malleable C2选项，在Cobalt Strike 3.12.引入。启用后，Beacon将在进入Sleep状态之前在内存中混淆自身。

那么我们先使用默认的关闭Obfuscate and Sleep来查看CobaltStrike进行进程注入会的具体情况。  

![](https://gitee.com/fuli009/images/raw/master/public/20210907205440.png)

  

然后我们把进程注入到微信中。

![](https://gitee.com/fuli009/images/raw/master/public/20210907205443.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210907205444.png)

  

注入微信进程

![](https://gitee.com/fuli009/images/raw/master/public/20210907205445.png)

  

正常执行命令

![](https://gitee.com/fuli009/images/raw/master/public/20210907205446.png)

  

然后我们在目标主机中使用Process Hacker 2进行检测查看：

![](https://gitee.com/fuli009/images/raw/master/public/20210907205447.png)

查找调用 SleepEx 的线程来定位内存中的 Beacon，一般在比较活跃的之中。

![](https://gitee.com/fuli009/images/raw/master/public/20210907205448.png)

  

然后，我们可以将关联的内存区域进行分析，转到Memory查看分析这个偏移量。

![](https://gitee.com/fuli009/images/raw/master/public/20210907205449.png)

  

我们可以对比看一下Beacon的情况：

![](https://gitee.com/fuli009/images/raw/master/public/20210907205450.png)

转到Memory查看分析这个偏移量并对比一下：

![](https://gitee.com/fuli009/images/raw/master/public/20210907205451.png)

可以看到我们可以看到我们的整个beacon在内存中未加密。

  

检测这样没有加密的beacon不难，我们在最简单的做法是，从这个区域挑选一些独特的字符串并将它们用作我们的检测的特征就行。

![](https://gitee.com/fuli009/images/raw/master/public/20210907205453.png)

  *   *   *   *   *   *   *   *   *   * 

    
    
    rule cobaltstrike_beacon_strings{meta:     author ="Elastic"description ="Identifies strings used in Cobalt Strike Beacon DLL."strings:   $a = {70 6F 77 65 72 73 68 65 6C 6C 20 2D 6E 6F 70 20 2D 65 78 65 63 20 62 79 70 61 73 73 20 2D 45 6E 63 6F 64 65 64 43 6F 6D 6D 61 6E 64 20 22 25 73}condition:       any of them }

  

当然上面的我只是举个例子，在实战中还得细一点。

![]()

  

国外也有个安全研究人员给出了个yar

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    rule cobaltstrike_beacon_strings{meta:  author = "Elastic"  description = "Identifies strings used in Cobalt Strike Beacon DLL."strings:    $a = "%02d/%02d/%02d %02d:%02d:%02d"    $b = "Started service %s on %s"    $c = "%s as %s\\%s: %d"condition:   2 of the }

同时Cobalt Strike给出了一种Bypass方法

  *   * 

    
    
    # Obfuscate Beacon, in-memory, prior to sleepingset sleep_mask "true";

``  

![](https://gitee.com/fuli009/images/raw/master/public/20210907205454.png)

  

Set sleep_mask “true”; 设置使beacon在睡眠之前混淆内存中的代码,睡眠后对自己进行混淆处理

![](https://gitee.com/fuli009/images/raw/master/public/20210907205455.png)

  

可以看到在混淆内存中的代码，然后我们使用前面的规则并不能检测到了beacon

![](https://gitee.com/fuli009/images/raw/master/public/20210907205457.png)

  

其实如果你刷新几次也可以发现解密的beacon,因为在每次使用beacon，都会重新加密数据和字符串。

![](https://gitee.com/fuli009/images/raw/master/public/20210907205458.png)

  

那么我们也可以多检测几次也可以检测到：

![](https://gitee.com/fuli009/images/raw/master/public/20210907205459.png)

  

因为我现在使用的4.3的Cobalt Strike，使用 13 字节的 XOR 密钥，

![](https://gitee.com/fuli009/images/raw/master/public/20210907205500.png)

  

如果是4.2以下Cobalt strike 使用的是使用简单的单字节 XOR 混淆，使用下面的yar一样可以检测：

``

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    rule cobaltstrike_beacon_xor_strings{meta:    author = "Elastic"    description = "Identifies XOR'd strings used in Cobalt Strike Beacon DLL."strings:    $a = "%02d/%02d/%02d %02d:%02d:%02d" xor(0x01-0xff)    $b = "Started service %s on %s" xor(0x01-0xff)    $c = "%s as %s\\%s: %d" xor(0x01-0xff)condition:    2 of them}

  

这里不多讨论。

  

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

使用内存特征检测 Cobalt Strike

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

