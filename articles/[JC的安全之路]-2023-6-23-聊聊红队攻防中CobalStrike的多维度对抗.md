#  聊聊红队攻防中CobalStrike的多维度对抗

原创 JC  [ JC的安全之路 ](javascript:void\(0\);)

**JC的安全之路** ![]()

微信号 csec527

功能介绍 一个安全从业者的想法见解 ，每周随缘更新1-2篇

____

___发表于_

收录于合集

#红队相关 2 个

#CobaltStrike 1 个

#样本对抗 1 个

#安全小技巧 6 个

  

  

#

**前 言**  
  

闲来无事，聊聊CobaltStrike在攻防对抗中的一些Bypass操作。

  

 **0 1** **其之一profiles  
**

  

Profiles是CobalStrike流量混淆的基础，当然也会操纵shellcode的一些加载和执行方式，用于绕过杀毒，网上也有很多相关的profile，当然流传的是APT组织的那些肯定是重点标记的，这里简单的举例，笔者首先做一组对比测试。

  

####  **1）无profiles**

![](https://gitee.com/fuli009/images/raw/master/public/20230623141241.png)

生成的shellcode

![](https://gitee.com/fuli009/images/raw/master/public/20230623141242.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230623141243.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230623141244.png)

嗯，落地就被干碎，挺正常的，毕竟啥也没做处理

####  **2）加载profiles**

![](https://gitee.com/fuli009/images/raw/master/public/20230623141245.png)

再生成一个shellcode

![](https://gitee.com/fuli009/images/raw/master/public/20230623141246.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230623141247.png)

杀不到了

可以看到这里其实已经过了火绒的静态了，关键点在于profiles里面替换了大量的敏感字符，也就是所谓的静态特征，随意截取一段

![](https://gitee.com/fuli009/images/raw/master/public/20230623141248.png)

这里主要是对stag的处理，也就是去除被标记的特征。

  

 **0 2** **其之二Arsenal-kit**

###  ****

  

Arsenal-kit是cs官方出的一个帮助免杀的套件，可以通过简单的配置绕过查杀，而ArtifactKit是在arsenal-
kit当中的一个处理beacon的套件，通过修改加载方式等，bypass杀毒，主要还是ArtifactKit这个插件，这个插件呢主要针对一阶段马Artifact，二阶段马就没那么好的效果了

![](https://gitee.com/fuli009/images/raw/master/public/20230623141249.png)

生成这些东西还是需要做简单配置，随便改下

![](https://gitee.com/fuli009/images/raw/master/public/20230623141250.png)

简单编译一下

![](https://gitee.com/fuli009/images/raw/master/public/20230623141251.png)

然后加载插件

![](https://gitee.com/fuli009/images/raw/master/public/20230623141252.png)

再生成一个shellcode，为了有效的验证这个工具的绕过查杀的效果，我这里先取消了profiles，再做验证

![](https://gitee.com/fuli009/images/raw/master/public/20230623141253.png)

确保裸马是被杀的

![](https://gitee.com/fuli009/images/raw/master/public/20230623141254.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230623141255.png)

现在加载插件生成一个新的shellcode

![](https://gitee.com/fuli009/images/raw/master/public/20230623141256.png)

 **可以看到没有查杀了，这里放张vt的图，证明效果还是可以的**

![]()

 **  
**

  

 **0 3** **其之三自写免杀**

  

  

这里使用二阶段生成的shellcode搭配自写的免杀，很轻松就可过Defender

![](https://gitee.com/fuli009/images/raw/master/public/20230623141257.png)

最新版本Defender

![](https://gitee.com/fuli009/images/raw/master/public/20230623141258.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230623141259.png)

denfender一声不吭

![]()

火绒如下

![](https://gitee.com/fuli009/images/raw/master/public/20230623141300.png)

某擎，一声不吭嗷

![](https://gitee.com/fuli009/images/raw/master/public/20230623141301.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230623141303.png)

甚至执行个命令也无事发生

![](https://gitee.com/fuli009/images/raw/master/public/20230623141304.png)

从前的时候，会告警内存里有cs（吃了好多次亏，麻麻滴），现在告都不告了，笑死

  

![](https://gitee.com/fuli009/images/raw/master/public/20230623141305.png)

  

  

  

 **0 4** **最 终效果  
**

  

  

从实战验证可以看到在对抗主防，EDR效果上还是很不错的,攻防是一个持续的过程，虽然我挺怀疑某些安全公司的产品在摆烂。  

  

  

![]()

  

  

  

  
 **E ND**  
![](https://gitee.com/fuli009/images/raw/master/public/20230623141306.png) **扫
码关注了解更多**安全小技巧  
  

  

  

  

  
  

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

