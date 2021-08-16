##  APT-C-54(Gamaredon)近期技战术总结

原创 高级威胁研究院  [ 360威胁情报中心 ](javascript:void\(0\);)

**360威胁情报中心** ![]()

微信号 CoreSec360

功能介绍
360威胁情报中心是全球领先的威胁情报共享、分析和预警平台，依托360安全大脑百亿级样本，万亿级防护日志等海量安全数据，整合360漏洞挖掘、恶意代码分析、威胁情报追踪等团队的安全能力，产出高质量的安全威胁情报，驱动安全的防御、检测和响应。

____

__

收录于话题

#APT 40

#东欧地区 8

#APT-C-54 Gamaredon 4

  
  

**Gamaredon组织是具有俄罗斯背景的APT组织，长期以来针对乌克兰及周边地区进行着高频的网络攻击活动。该组织向来以活动强度大和活动频率高为主要特点，常常在短期内产生大量的威胁情报指标，且丝毫不在意暴露自身的俄罗斯相关背景。本篇报告将结合往期内容，针对该组织近期攻击活动中使用的不同载荷及暴露出的特征进行介绍。**  

  
  
 **  
** **一.  攻击组件**  
  
在去年年底我们发布的相关报告中，介绍了Gamaredon组织的不同攻击组件类型，其所利用的组件中，包括了vbs脚本、sfx自解压文件和dll文件窃取后门。经过一段时间的追踪和研究，我们发现该组织对使用的攻击载荷进行了一定程度的更新和换代，今年4月，我们就介绍了该组织新启用的一批文件窃取后门。如今，该组织的攻击载荷再度发生了变化。  
1.诱饵文档长期以来，Gamaredon通过钓鱼邮件等途径向攻击目标投递诱饵文档，今年年初各大安全厂商也连续捕获了该组织大量的文档类样本，这些样本多数内嵌远程模板文件链接，也有部分直接嵌入vba宏并引诱攻击目标启用，在后续的dropper组件中，我们也发现该组织通过感染攻击目标计算机内的文档，使原本正常的文件变成Gamaredon恶意载荷的二次传播工具。通过对文档内容的分析，我们推测出其攻击目标主要为乌克兰政府工作人员和军方人员，部分诱饵文档结合了当下的时事政治，诱惑性极强。下图为近期捕获到的一例诱饵文档，标题为卢甘斯克地区国家警察总局，可以看到左下角显示文档语言为乌克兰语：

![](https://gitee.com/fuli009/images/raw/master/public/20210816211759.png)

该文档嵌入了一处远程模板链接：  

![](https://gitee.com/fuli009/images/raw/master/public/20210816211808.png)

遗憾的是，长期以来我们捕获到的同类链接都没有下载成功，但从历史样本推断，下载成功后的模板文件应当会包含VBA宏，引诱受害者启用宏后，会释放并执行后续的downloader组件。  
  
2.Dropper(1)sfx自解压文件我们在往期有关Gamaredon的报告中披露过，该组织擅长利用sfx自解压文件结合vbs脚本和bat批处理文件完成第一阶段载荷的释放和第二阶段载荷的下载，即便其载荷的组合形式和脚本文件的风格有所变化，这一攻击形式到本文完成为止并未有明显的改变。sfx所释放的载荷除了去年提到过的vbs
downloader和指向远程html的link文件之外，我们还捕获到一例通过批处理脚本感染受害者word文档的载荷。

![]()

在本例中，文件名为245的批处理文件会调用名为21738的vbs脚本，关闭word的VBA警告，然后调用名为4854的vbs脚本，给用户计算机内.word文档添加上恶意的模板链接。  

![](https://gitee.com/fuli009/images/raw/master/public/20210816211809.png)

其中名为21738的脚本通过修改注册表来关闭VBA宏警告：  

![](https://gitee.com/fuli009/images/raw/master/public/20210816211810.png)

  
(2)C++ Dropper今年七月，我们还捕获到了Gamaredon新启用的一批c++
dropper，可能是考虑到7z自解压文件被杀软查杀的概率较高，才新编写了这一类型的dropper组件。此类组件逻辑较为简单，主要功能就是解密并执行攻击载荷，释放的载荷依旧主要由vbs脚本组成。目前我们捕获到的c++
dropper主要有以下两个类型：第一种会从资源文件中解密出载荷，然后释放执行：

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210816211811.png)

解密出的载荷包含vbs脚本和要释放的路径环境变量、文件名等。不同的部分用\x00进行分割

![](https://gitee.com/fuli009/images/raw/master/public/20210816211812.png)

Vbs脚本中涉及路径的代码也会用\x00代替，释放前再用之前计算出来的路径字符串进行填充。填充完成的完整vbs脚本会调用wscript执行，其功能为一个vbs
downloader.  

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210816211813.png)

第二种dropper将payload隐藏在文件尾，并将payload的偏移保存在dos头偏移若干字节处，本例中，偏移量为96字节。隐藏的payload由Base32算法解密。

![](https://gitee.com/fuli009/images/raw/master/public/20210816211814.png)

后续过程与前文提到的大同小异，最终释放的是一个vbs downloader。  
八月初，我们捕获到了一个特殊的样本，从运行过程来看，此样本属于前文提到的第一种dropper，从资源文件中解密payload然后调用CreateProcess执行。说它特殊，是因为在样本逻辑中发现了大量垃圾变量和无用逻辑：

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210816211815.png)

这倒是非常符合Gamaredon一向的作风，在C#版本的downloader中，也曾见过类似的逻辑。  
3.Downloader(1) vbs
downloaderVbs脚本是Gamaredon最爱使用的攻击武器之一，我们也在以往的报告中详细介绍过该组织所使用的vbs脚本的各种组合类型。今年6月，我们披露了该组织通过修改PE头的方式向含有合法数字签名的PE文件尾部嵌入vbs脚本，并调用mshta执行，这一手法在该组织上还是首次见到。

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210816211816.png)

  
(2)C# Downloader在去年我们有关Gamaredon的报告中，介绍过该组织所使用的C#
downloader。在今年捕获到的同类样本中，混淆风格发生了一定的变化，通过加入大量的垃圾try
catch结构来提高分析人员的时间成本，但功能依旧没有太大的改变。

![](https://gitee.com/fuli009/images/raw/master/public/20210816211817.png)

与之前不同的是，新捕获到的C# downloader将c2从样本中分离了出去，连接c2时需要读取同目录下的同名txt文件：  

![](https://gitee.com/fuli009/images/raw/master/public/20210816211818.png)

   因为大量垃圾信息的加入，同类型的样本最大膨胀到了80兆：  

![]()

  

(3)C++
Downloader今年七月，我们捕获到了c++版本的downloader，样本内出现了该组织所使用的文件窃取后门中的部分算法，例如通过API字符串的hash获取函数地址的算法，以及字符串的解密算法：

![](https://gitee.com/fuli009/images/raw/master/public/20210816211819.png)

样本设置好Request Header后，会通过Post向C2发起请求  

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210816211820.png)

样本会通过InternetReadFile下载payload，完成后会进行异或解密，最后调用CreateProcess创建进程执行

![](https://gitee.com/fuli009/images/raw/master/public/20210816211821.png)

4.File
Stealer去年年底我们在有关Gamaredon的报告中介绍了该组织的文件窃取后门，该类样本是dll文件，通过rundll32执行。今年四月份，我们发布的相关报告揭露了Gamaredon新启用的文件窃取后门，该类后面以exe形式存在，且在几个月的时间内更新了数个版本。我们在后续的时间中又捕获了若干文件窃取后门，从功能上看，和四月份捕获到的并没有太大区别，但是相比之下，新捕获到的后门更加精简，也更专注于“窃取文件”这一点，取消了诸如屏幕截图的功能，只创建了两个子线程：

![]()

窃取的文件类型也依旧主要为文档类文件：  

![](https://gitee.com/fuli009/images/raw/master/public/20210816211822.png)

由于该样本在创建线程之前进行了若干项的检查，与c2通信之前也会检查网络的连通性，所以此样本免杀效果较好，在刚刚上传到VirusTotal时，仅有6个杀毒引擎报毒。截止写下这段话时，VirusTotal共19个引擎报毒。  
  
 **二. TTPs指纹分析**  
  
1\.
混淆手法长期以来，Gamaredon所采用的主要策略就是通过提高分析时间成本来降低分析人员的分析效率，在一段典型的该组织使用的脚本中，字符串会被破碎然后重新拼接，同时掺杂着无意义的注释和代码：

![](https://gitee.com/fuli009/images/raw/master/public/20210816211823.png)

在近期所捕获到的vbs脚本中，掺杂的注释里时常会出现一些人名、电话、网络地址等信息，这可能是用于防止自动化脚本从vbs里提取C2域名。  
类似的通过插入垃圾代码以实现混淆的方法在C#和C++版本的downloader中也可以见到，前文中也做了举例。如此的混淆手段实在算不上高级，却能以最低的成本浪费分析人员的精力。如果说在脚本中插入垃圾代码可以通过语法高亮用人眼“过滤”，那么二进制文件中的垃圾代码就确实需要分析人员花上一些功夫了。  
2\.
加解密算法Gamaredon的二进制样本中常用的手法包括动态获取API地址和字符串加密。这样的手段谈不上新鲜，但由于该组织的在野样本数量非常大，为了避免被安全研究人员轻易捕获，该组织会将同一算法通过修改编译参数或修改代码的方式进行字节码层面的改变，以提高提取特征的难度，下图是同一时期捕获到的两个样本中的GetApiByHash函数：

![](https://gitee.com/fuli009/images/raw/master/public/20210816211824.png)

与此同时，Gamaredon所采用的加解密算法，无非是将四则运算、异或、Base64、Base32等进行组合使用如下图所示的就是近期捕获到的样本中所采用的字符串解密算法：  

![]()

类似的算法逻辑非常简单，但也增加了特征命中结果的噪点。更糟糕的是，Gamaredon更新武器载荷的频率非常高，样本算法的简单和易于改造使得该组织同一批武器的使用寿命可能仅有一至两个月，这就要求研究人员不断地追踪和分析新样本、同步相关特征以追踪新的活动。当然，这正是我们一直以来在做的工作。  
3\.
网络特征Gamaredon喜欢采用Http协议进行通信，二进制样本主要通过wininet.dll的相关导出函数实现与c2的http通信，通信过程中，Gamaredon常使用火狐的UA头：

![](https://gitee.com/fuli009/images/raw/master/public/20210816211825.png)

在通信过程中，Gamaredon通常先利用DNS解析域名的ip地址，再通过ip地址与c2进行通信，这一点无论是在二进制样本还是脚本文件都可以体现出来：  
  

![](https://gitee.com/fuli009/images/raw/master/public/20210816211826.png)

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210816211827.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210816211828.png)

  

除此之外，在文件窃取后门中，Gamaredon常常设置Http请求头中的Pragma为”no - cache”，相比较正常的请求头，该项多出了两个空格

![]()

  
4\. 网络资产

通过对Gamaredon所掌握的域名进行梳理，我们发现绝大多数域名以”.ru”
结尾，注册在”reg.ru”，这倒是很符合Gamaredon的俄罗斯相关背景，但同时也说明了该组织完全不介意暴露自身的地域背景。

![](https://gitee.com/fuli009/images/raw/master/public/20210816211829.png)

除此之外，该组织的大多数域名所指向的ip地点都在莫斯科或圣彼得堡，且都为TIMEWEB下属资产。  
有趣的是，目前我们捕获到的任何url，无论是docx中嵌入的远程模板地址，还是downloader中嵌入的下载地址，都无法正常访问和下载。我们尝试过通过东欧代理访问，也尝试过使用与样本中相同的http头，但都以失败告终，仿佛这些url在暴露出来的一瞬间就失效了。我们猜测，Gamaredon可能采用了类似白名单的机制来控制网络的访问，相关情况有待进一步的研究。  
总结  
Gamaredon组织一直以来在东欧地区保持着较为高调的活跃姿态，其下属资产和攻击载荷更新频繁，且多种多样。在众多APT组织中，Gamaredon也属于比较另类的一种，在其他组织小心翼翼的进行攻击活动时，Gamaredon似乎丝毫不在意暴露自身的地域背景和攻击意图，其使用的攻击载荷也被广泛的捕获和分析，其下属网络资产数量也非常丰富。如果说其他组织的攻击武器是“精准而优雅”，那么Gamaredon的攻击武器绝对算得上“粗犷且数量庞大”，即使是混淆的手段，也是一种非常暴力的策略：插入大量垃圾变量和垃圾逻辑。其采用的规避查杀的方法是频繁的修改样本特征，针对同类载荷，通过不同的编译参数或修改代码顺序的方式来最大程度降低被特征规则命中的可能性，与此同时，大量初始阶段的恶意脚本都在本地生成和释放，这使得大量脚本文件根本不能作为恶意指标使用，因为每个受害者机器上的样本都不同。因此，分析人员需要花费一定时间精力跟踪和调查相关活动，并对新出现的样本进行分析和归纳。我们会在未来的时间里继续研究和追踪该组织的相关活动情况。
**\- END -** **  
** **附录 IOC**  
 **1\. MD5** **(1) sfx**
0f3842c10463c1bf73c82047b230e9117b34ce7cb665072e157fa38c8d9e370f8a218c943466c2662c928965189cbe979b68d72e3b64e939320106b40329dd0b65c8a0ff681200d5494f03ae7682d7e01854bae42505bdc4c639dc97a78ea8a2e3312e533258980df3c35f9eef51e136f366b8fbc2e64553d6393d861f6125acff8355a03aa8997f1d8173e4b4e20830
**(2) C#downloader**
7fc7bac75263f4357c6283e947114052810d3ed1d755e330d9458da59ae424d27824989ef667ab27312e7966016e0cc3fdf4ab7be093e68a080b8045b48f3455
**(3) C++downloader**
4ae34503cda95fc3d11fec574f361383582417c5467a7c3a219157321fc4371db57e0679ba83424ad695174b5400276092f92cdc83a89f133cb266339268cd23
**(4) C++dropper**
84b50758618ab278f187a506c18687ca6873052c4282aeda4802128534e720b9625042cc1af1a427798ad71463d0b073b06d4123ade8cb797876512525f7fcf8b43e4f2ae26e04ab32937131d04f026d
**(5) C++File Stealer** fc40434947caee8b0b3237ca03208716  
0fdf124e0a2ccdd5348ca9965a4f8eb7cdc4eb983e5d29836b743afc060900975f4cc572801b5711635fa289421ec0806a5d0742b8a8ab084ba31ffdd2e935ed8c492bd3e854d0c2d8753dac5a375c195d40912bdf632ddefe980d428453a0609e8b06452b8bd1d7312426d1dfae215521d08e99fc8d5ec15be2b11110deecc20336d521fce938c2ff9993cdfb181bcb9623e2299f609041b3f8df6ecf387999e14e53c0bf0d3df7dc8f26e8ff2fd01b65365420a3bb5d943ecd6668dd5dbfef6ee61b9935b8483af73ddbe407c6bb4692f92cdc83a89f133cb266339268cd231cf3ff8141a065e2e9e312102a462804
**2\. 域名**
oidium.ruprivigna.ruferruminatio.ruproteusa.ruagarisi.rumyces.ruenarto.rucereusi.onlinepapiliot.onlineavirona.rucolista.ruprisonta.rumortalin.rupenicil.rutenosha.rucolidar.rumonask.rubercul.rufishardo.rumarinos.rubrontaga.rucalendas.ruonibacter.rutuberci.rurabilin.ruculosa.ruflatfisha.rukrashand.rubertis.ruteroba.ruobacter.rufelineus.onlineteriuma.rusardanal.ruminalis.rublattodea.onlineburuncha.ru  
  
![](https://gitee.com/fuli009/images/raw/master/public/20210816211830.png)
**团队介绍**![](https://gitee.com/fuli009/images/raw/master/public/20210816211830.png)
TEAM INTRODUCTION **360** **高级威胁研究院**
360高级威胁研究院是360政企安全集团的核心能力支持部门，由360资深安全专家组成，专注于高级威胁的发现、防御、处置和研究，曾在全球范围内率先捕获双杀、双星、噩梦公式等多起业界知名的0day在野攻击，独家披露多个国家级APT组织的高级行动，赢得业内外的广泛认可，为360保障国家网络安全提供有力支撑。

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

APT-C-54(Gamaredon)近期技战术总结

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

