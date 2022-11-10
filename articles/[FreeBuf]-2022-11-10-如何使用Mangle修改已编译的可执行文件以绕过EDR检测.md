#  如何使用Mangle修改已编译的可执行文件以绕过EDR检测

原创 Alpha_h4ck  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 中国网络安全行业门户

____

___发表于_

收录于合集

##
**![](https://gitee.com/fuli009/images/raw/master/public/20221110202522.png)**

## **  
**

## **  关于Mangle **

  
Mangle是一款功能强大的代码处理和安全测试工具，该工具基于Golang开发，可以帮助广大研究人员从各个方面对已编译好的可执行程序（.exe或DLL）进行修改，从而实现EDR检测绕过。  

##  **  工具运行机制 **

  
Mangle可以删除基于字符串的入侵威胁指标（IoC），并将其替换为随机字符，然后通过增加文件大小来避免EDR检测，而且还可以通过合法文件来克隆代码签名证书。在整个过程中，Mangle可以帮助加载器绕过磁盘和内存扫描工具的检测。  

##  **  工具安装 **

  
首先，该工具基于Golang开发，因此我们需要在本地设备上安装并配置好Golang环境。接下来，使用下列命令将该项目源码拉取到本地，然后安装该工具所需的依赖组建，并编译项目代码：

    
          * 
    
    
    
    go get github.com/Binject/debug/pe

  
然后，使用下列命令构建项目源码：

    
          * 
    
    
    
    go build Mangle.go

##  **  
**

##  **  工具使用 **

  

![](https://gitee.com/fuli009/images/raw/master/public/20221110202531.png)

###  

###  **参数解释**

> -C 字符串：包含需要克隆的证书路径；-I 字符串：原始文件路径；-M 字符串：编辑PE文件以替换/去除Go标识符指定的字符串；-O
> 字符串：新文件名称；-S 整数：需要增加多少文件大小；

###  **  
**

###  **字符串**

  
Mangle可以获取研究人员提供的可执行文件并寻找那些安全产品可能会搜索或触发安全警报的已知字符串。这些字符串并不是唯一的检测因素，因为反病毒产品一般会将这些字符串和其他（遥测）数据结合起来检测。而Mangle可以找到这些已知的字符串，并用随机值替换掉字符串的十六进制值，然后移除原始字符串。需要注意的是，这种替换方式并不会改变文件大小，这样可以防止文件报错。  
字符串修改样例：修改前。

![](https://gitee.com/fuli009/images/raw/master/public/20221110202532.png)

  
字符串修改样例：修改后。  

![](https://gitee.com/fuli009/images/raw/master/public/20221110202533.png)

###  

###  **文件体积增加**

  
几乎所有EDR都无法扫描磁盘或内存中超过一定大小的文件，因为大文件需要更长的时间来查看、扫描或监视，而EDR不希望通过降低用户的生产率来影响性能。Mangle通过在文件末尾创建空字节（零）填充来增加文件体积，这样可以确保文件内的任何内容都不会受到影响。建议将大小增加95-100
MB，不建议制作2 GB或以上的文件。  

![](https://gitee.com/fuli009/images/raw/master/public/20221110202534.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221110202535.png)

###  **证书克隆**

  
Mangle还可以从一个文件中获取合法代码签名证书的完整链和所有属性，并将其复制到另一个文件。其中包括签名日期、反签名和其他可测量的属性：

![](https://gitee.com/fuli009/images/raw/master/public/20221110202537.png)

##  **  许可证协议 **

  
本项目的开发与发布遵循MIT开源许可证协议。

##  

##  **  项目地址 **

  
 **Mangle** ：https://github.com/optiv/Mangle

##  

##  **参考资料：**

> https://github.com/Tylous/Limelighterhttps://github.com/Binject/debug

![](https://gitee.com/fuli009/images/raw/master/public/20221110202538.png)  
  

精彩推荐

  
  
  
  
  
  
  
[![](https://gitee.com/fuli009/images/raw/master/public/20221110202539.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247489554&idx=1&sn=7dc76862d96516013375f712c9bdfcf1&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20221110202541.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247489510&idx=1&sn=55f589464f2ffbf2817523f3282175ba&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20221110202542.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247489388&idx=1&sn=f0e8fed4afd1dc82ac6af789e5b13626&scene=21#wechat_redirect)![](https://gitee.com/fuli009/images/raw/master/public/20221110202543.png)

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

