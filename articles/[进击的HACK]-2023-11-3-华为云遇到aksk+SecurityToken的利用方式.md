#  华为云遇到aksk+SecurityToken的利用方式

原创 进击的HACK  [ 进击的HACK ](javascript:void\(0\);)

**进击的HACK** ![]()

微信号 redteasec

功能介绍 1、分享网络安全知识。 2、分享IT热门事件。

____

___发表于_

收录于合集 #渗透测试 23个

前言

工作中遇到了一个接口泄露了华为云的aksk以及SecurityToken，之前没见过，趁此机会学习了解一下如何利用。

利用到华为云官方的工具obsutil，下载链接https://support.huaweicloud.com/utiltg-
obs/obs_11_0003.html。

aksk分为长期有效的和短期有效的。

长期有效的aksk一般不会变化，在js或者app反编译后可能看到。

带着SecurityToken的aksk一般请求一次变动一次。

  

## 固定aksk

这个利用方式比较普遍，可以使用华为云官方的图形化客户端OBS Browser+

下载地址：https://support.huaweicloud.com/browsertg-obs/obs_03_1003.html

![]()

利用方式和阿里云的aksk差不多，不做过多赘述

## 带着SecurityToken的aksk

  

带着SecurityToken的aksk就无法使用OBS
Browser+，此时我们可以使用官方命令行工具obsutil，下载链接https://support.huaweicloud.com/utiltg-
obs/obs_11_0003.html。

![]()  

根据需要下载即可。

通过 ./obsutil config -interactive 方式设置配置文件

    
    
    obsutil config -interactive  
    Please input your ak:  
    randyNUATN  
    Please input your sk:  
    randyITzwtQO9  
    Please input your endpoint:  
    http://obs.randy.sesame  
    Please input your token:  
    obs.randy.sesame  
    Config file url:  
    /home/randy/.obsutilconfig  
    Update config file successfully!

如果是Windows用户会在C:\Users\xxx\\.obsutilconfig目录下生成

如果出现Access Key Id you provided does not exist in our
records.，可能因为你的aksk超时了，需要重新生成配置文件。

![]()

之后就可以执行命令。

一般来说，配置了SecurityToken的aksk的权限都比较低，可以尝试增删改查的命令，确定权限。如果没有权限一般会返回403或者400。执行成功会返回200.

  

### 常见的命令

可以参考[https://mp.weixin.qq.com/s/CWIeO0aA8Spq11y3ftty9g](https://mp.weixin.qq.com/s?__biz=MzUyMDc2MDMxNg==&mid=2247489706&idx=1&sn=a4e2439aee1cbe20f1786fc22d0f10f9&scene=21#wechat_redirect)
中的快速使用中的内容，里面比较详细

操作桶

运行./obsutil mb obs://bucket-randy 命令，创建一个名为bucket-randy的新桶。

删除桶（危险操作，谨慎使用

./obsutil rm obs://bucket-randy -f

查询桶属性

./obsutil rm obs://bucket-randy -f

上传文件/文件夹（ **上传文件功能，可以尝试是否允许覆盖已有文件，如果可以覆盖也算一个漏洞**

运行./obsutil cp obsutil obs://randy/命令，将本地obsutil文件上传至randy桶中。

  

下载文件

运行./obsutil cp obs://randy/obsutil ~/Downloads/obsutil-
randy命令，将randy桶中的obsutil对象下载至本地。

  

附上

# 阿里云AK泄露之STS(SecurityToken)如何利用

[https://mp.weixin.qq.com/s/a3ZhPtqmVcdF4tWJ3Pn7Lg](https://mp.weixin.qq.com/s?__biz=MzI5Nzc3NDEyNA==&mid=2247485648&idx=1&sn=9b575adafd26c0a983baf714c06add06&scene=21#wechat_redirect)

  

## 参考链接

[https://mp.weixin.qq.com/s/CWIeO0aA8Spq11y3ftty9g](https://mp.weixin.qq.com/s?__biz=MzUyMDc2MDMxNg==&mid=2247489706&idx=1&sn=a4e2439aee1cbe20f1786fc22d0f10f9&scene=21#wechat_redirect)

  

  

  

  

  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

