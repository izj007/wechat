#  实战之低版本tomcat-getshell

原创 遥遥 [ 极梦C ](javascript:void\(0\);)

**极梦C** ![]()

微信号 gh_2353880ae4d9

功能介绍 只专注于实战的实战派。

____

___发表于_

收录于合集

前言

某次红蓝对抗中,发现目标低版本的tomcat弱口令，但是常规的war包无法上传。

  

祝各位表哥国庆节假期快乐。

  

环境介绍

![](https://gitee.com/fuli009/images/raw/master/public/20220929133211.png)![](https://gitee.com/fuli009/images/raw/master/public/20220929133212.png)

linux tomcat6.0

  

  

过程-上传war包

![](https://gitee.com/fuli009/images/raw/master/public/20220929133211.png)  

直接修改后缀,尝试war包上传。

  

![]()  

  

  

发现未成功部署，显示false。点击start也无法成功部署。 当然也无法访问。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133214.png)

  

思考是否是war的生成方式不对。

这里采取了标准的war生成。

War包构造 Jar cvf jmc.war ./

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133215.png)  

  

发现war包成功部署。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133217.png)  

  

  

但是访问502报错。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133218.png)  

  

继续尝试,这次更改jsp为txt文件进行尝试。 发现txt文件是可以正常被访问。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133219.png)  

那么问题就出在了jsp文件上面。或者是目录问题。

这里尝试过1.8jdk的马子 各种低版本的。 命令执行的马子 大马等等都不可以

这里尝试多层目录，发现也是不行。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133220.png)  

  

成功上传

![](https://gitee.com/fuli009/images/raw/master/public/20220929133211.png)![](https://gitee.com/fuli009/images/raw/master/public/20220929133212.png)

在搜索相关文章的时候,发现一位表哥讨论了axis2和tomcat的上传问题,这里引起了注意。

这里axis2成功部署的话,可以利用axis进行文件上传。

是利用aar后缀进行部署的。这里github找到一个别的师傅的aar包。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133223.png)  

  

成功部署。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133224.png)  

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133225.png)  

成功获取shell

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133226.png)  

### 解决了上次低版本没有拿到shell的问题。

首发freebuf

  

  

  
  

  

  

关于文章/Tools获取方式:请关注交流群或者知识星球。

  

  

  

  

  
  
  
  
  
  

关于交流群：因为某些原因，更改一下交流群的获取方式

  

  1. 请点击联系我们->联系官方->客服小助手添加二维码拉群 。    

![](https://gitee.com/fuli009/images/raw/master/public/20220929133227.png)

  

  

  

  

  
  

关于知识星球的获取方式

  

  1. 后台回复发送 "知识星球"，即可获取知识星球二维码。不定时发送免费名额。

  2. 如若上述方式不行，请点击联系我们->联系官方->客服小助手添加二维码进入星球 。    

![](https://gitee.com/fuli009/images/raw/master/public/20220929133227.png)

  

  

  

  

  

  

  

免责声明

  

          

  

本公众号文章以技术分享学习为目的。

由于传播、利用本公众号发布文章而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号及作者不为此承担任何责任。

一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！

  

  

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20220929133229.png)

  

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

