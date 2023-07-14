#  雷石｜yonyouNC命令执行Bypass测试分享

原创 xueqi  [ 雷石安全实验室 ](javascript:void\(0\);)

**雷石安全实验室** ![]()

微信号 leishianquan1

功能介绍 雷石安全实验室以团队公众号为平台向安全工作者不定期分享渗透、APT、企业安全建设等新鲜干货，团队公众号致力于成为一个实用干货分享型公众号。

____

___发表于_

收录于合集

#技术专栏 211 个

#雷石安全运营 2 个

#网络安全 14 个

![](https://gitee.com/fuli009/images/raw/master/public/20230714174740.png)

**前言**

  
![](https://gitee.com/fuli009/images/raw/master/public/20230714174741.png)
在渗透和攻防中，多次遇到用友NC系统。一直以来都是用别人的工具，自己从来没分析研究过用友的源码和利用，遇到一些场景或需要利用漏洞执行命令时踩了很多坑。最近便通过灰盒测试简单分析了下NC6的命令执行，并尝试bypass。
**测试环境：** win server+NC6.3，win server+NC6.5

  

  

 **01.**

 **NC6.3**

  
首先是用友NC6.3，无论是bsh.servlet.BshServlet命令执行还是反序列化执行系统命令，都会遇到dir、echo命令执行失败。通过监控进程，发现未调用cmd，猜测是被过滤。

![](https://gitee.com/fuli009/images/raw/master/public/20230714174742.png)

  

当命令执行成功时，会调用系统cmd执行。

![](https://gitee.com/fuli009/images/raw/master/public/20230714174743.png)

  

通过测试，发现加上cmd/c能够成功调用cmd并执行

![](https://gitee.com/fuli009/images/raw/master/public/20230714174745.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230714174746.png)

  

尝试echo命令写入文件，也没有问题，完全正常写入exec("cmd /c echo ccc > nc63.txt");

![](https://gitee.com/fuli009/images/raw/master/public/20230714174747.png)

  

 **NC6.5**

 **02.**

  
在NC6.5中，发现在NC6.3中的方法已不再适用。增加了转义，特殊符号全部失效。例如通过echo写入需要用到的“>”
，会被双引号转义成字符串而非命令。被转义：可以看到 cmd和特殊字符都被添加了双引号包裹

![](https://gitee.com/fuli009/images/raw/master/public/20230714174748.png)

  

失败尝试：

![](https://gitee.com/fuli009/images/raw/master/public/20230714174749.png)

  

 **尝试Bypass：** 首先是执行无参数的命令，观察发现上面使用cmd /c
时，cmd被加了双引号，但还是成功调用cmd命令。1.在cmd下测试命令加上双引号还是会正常执行。2.NC6.5中会以空格分界，对含有特殊字符的字符串添加引号。通过测试，发现上面两个特写。用如下方式，将执行结果通过尖括号输出到文本。

![]()

  

成功执行方式：

![](https://gitee.com/fuli009/images/raw/master/public/20230714174750.png)

  

 **命令执行带参数：** 如果出现命令需要有参数的场景怎么办？win中的系统命令大多都是用斜线“/” 来声明参数名。经过尝试，发现可以直接省去空格连写:

![](https://gitee.com/fuli009/images/raw/master/public/20230714174751.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230714174752.png)

  

 **写文件：** 但在实战中还是要写入webshell的，就要用到echo 和">"符号。这一步确实让我折腾了一番，后来想到用闭合双引号方式来干扰转义。

![](https://gitee.com/fuli009/images/raw/master/public/20230714174754.png)

  

经过测试,用如下方式写入的缺点是会多出空格和引号：

![](https://gitee.com/fuli009/images/raw/master/public/20230714174755.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230714174756.png)

  

 **小结**

  
![](https://gitee.com/fuli009/images/raw/master/public/20230714174741.png)

会用利用脚本是一回事，复现了漏洞是一回事，能否在攻防中利用漏洞并拿到权限又是另一回事。

看似有了exp复现漏洞就是掌握了，但在实战中总会遇到不同环境、不同场景，也会出现不同状况，还是要掌握漏洞原理才能逐个攻破。

  

 **推荐阅读**

  
![](https://gitee.com/fuli009/images/raw/master/public/20230714174741.png)

#
[·白加黑免杀制作（详细）](https://mp.weixin.qq.com/s?__biz=Mzg5MDg0NzUzMw==&mid=2247483769&idx=1&sn=72470857b2f9eb1f1fee11e901bc9873&scene=21#wechat_redirect)

·文章预览

![](https://gitee.com/fuli009/images/raw/master/public/20230714174759.png)

  

 **往期回顾**

  

 **01**

[Electron应用调试技巧分享‍‍](http://mp.weixin.qq.com/s?__biz=MzI5MDE0MjQ1NQ==&mid=2247525561&idx=1&sn=a4fa8c403724908f24ee13704c2825e8&chksm=ec264521db51cc37fd98b36c309b3dd01115acb5ea4afe5236c167776d18944741b2de8e3b67&scene=21#wechat_redirect)

 **02**

[Python安全工具开发思路分享](http://mp.weixin.qq.com/s?__biz=MzI5MDE0MjQ1NQ==&mid=2247525460&idx=1&sn=8ef28ea0172aff9bc154e4efdd4bf017&chksm=ec2645ccdb51ccda8d610ce9c23a51e5f392cc232e1e3741ebfb72eb99f0301da793785f19fd&scene=21#wechat_redirect)

 **03**

[一个好用的RPC框架](http://mp.weixin.qq.com/s?__biz=MzI5MDE0MjQ1NQ==&mid=2247525440&idx=1&sn=1692dc4c8eab9c541d940d7bb9b7a8be&chksm=ec2645d8db51ccceb2021615db11f7ca120951b0af8039f3c9fa4805c3e2e91060bfcc92302e&scene=21#wechat_redirect)

![](https://gitee.com/fuli009/images/raw/master/public/20230714174800.png)

 **雷石安全实验室**

  

商务咨询：

0571-87031601

商务邮箱：

mtn@motanni.com

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

