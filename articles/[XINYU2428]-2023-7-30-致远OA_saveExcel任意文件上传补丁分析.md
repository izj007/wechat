#  致远OA_saveExcel任意文件上传补丁分析

原创 XINYU2428 [ XINYU2428 ](javascript:void\(0\);)

**XINYU2428** ![]()

微信号 gh_65c38df19d17

功能介绍 信息安全打工人的学习分享及日常记录

____

___发表于_

收录于合集

### 0x00 前言

本篇文章主要内容为，致远OA-saveExcel任意文件上传漏洞补丁分析，这是一个后台漏洞。

漏洞影响范围：V7.1-V8.1SP2；复现成功版本：V8.0SP2

  

### 0x01 补丁分析

补丁发布时间：2023年6月

补丁下载地址：https://service.seeyon.com/patchtools/tp.html#/patchList?type=%E5%AE%89%E5%85%A8%E8%A1%A5%E4%B8%81&id=167

![]()

  

下载查看

![]()

  

补丁已知的信息

    
    
    漏洞说明：上传功能未对上传的文件做控制，导致可任意上传文件  
    漏洞文件：seeyon-ctp-core.jar!/com/seeyon/ctp/common/excel/FileToExcelManagerImpl.class  
    

补丁和源文件对比

    
    
    补丁增加了两个if判断  
    1.filePath参数中不能存在：webapps、..  
    2.fileName参数后缀做了白名单：xls、xlsx、csv  
      
    漏洞方法基本可以确定为 saveExcelInBase 方法  
    

![]()

  

已知是 `saveExcelInBase` 方法了，我的习惯是从后往前看，先找到 `写入文件方法` 的调用位置。

首先确认控制文件名的参数是哪个，参数值传递大概关系如下：

out_ -> file -> filePath

![]()  
  
我们往上找 `filePath` 参数，可以看到它是方法直接接收过来的形参，可控，`fileName` 参数没有用到可忽略，`dataRecords`
用来传递写入内容，可控。![]()  

下面就是去寻找 `saveExcelInBase` 方法的调用位置即可，搜索关键字找到如下几个调用位置。

![]()

  

先看 `SafetyProtectionController` ，传递的参数个数都对不上，文件后缀也不可控，排除。  

![]()  

在看 `LogonLogController` ，该文件共有四处调用了，我们以文件后缀是否可控为切入点去做排除，一下子全排除了…，文件后缀全是写死的
`.xls`

![]()  

此时

第一想法：有遗漏的jar包没反编译，坑次坑次忙活了一会儿，搜索到几个新的 `saveExcelInBase` 方法的调用位置，但后缀仍是写死。

第二想法：我就知道没这么简单，会不会是一个任意类调用之类的，坑次坑次又忙活了两会儿，找不到。

第三想法：找人。

然后就去找了小白，直接给我一个笔记链接，打开一看，恍然大悟，不经发出感慨、

![]()

  

### 0x02 漏洞复现

![]()

![]()

    
    
    POST /seeyon/ajax.do HTTP/1.1  
    Host:   
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36 uacq  
    Content-Type: application/x-www-form-urlencoded; charset=UTF-8  
    Cookie: JSESSIONID=CB1120AE205DAFD7FF70EAF5DC1241B9  
    Content-Length: 154  
    Connection: close  
      
    method=ajaxAction&managerName=fileToExcelManager&managerMethod=saveExcelInBase&arguments=["../webapps/ROOT/ceshi.txt","",{"columnName":['HelloWorld']}]

  

 **ajax.do路由调用流程关键位置贴图**

`ajax.do` 路由经过seeyon的关键过滤器CTPSecurityFilter，会走到图中 `result.authenticate(req,
resp);` 位置。

![]()seeyon-ctp-
common.jar!/com/seeyon/ctp/common/web/filter/CTPSecurityFilter.class  

到这里系统获取了传递的前三个参数的值，此方法的作用是用来校验请求调用的类和方法是否合法。注意 `ajaxAccessInterceptor`
这个成员变量，`V8.0SP2版本` 中，默认为 `null` ，以至跳过了对即将调用的类和方法的检测。

![]()seeyon-ctp-
common.jar!/com/seeyon/ctp/common/web/filter/AjaxAuthenticator.class  

与上图对比：`V8.1SP1版本` 中，这里默认初始化了一个白名单列表，`fileToExcelManager.saveExcelInBase`
是不在白名单内的，导致到这里就被拦截，走不到后面的反射调用流程。

![]()

  

回到 `V8.0SP2版本` ，流程走到这里，就出了 `CTPSecurityFilter` 这个核心过滤器了

![]()

  

继续，来到 `AjaxController` 的 `ajaxAction` 方法

![]()seeyon-ctp-common.jar!/com/seeyon/ctp/common/service/AjaxController.class

  

进入 `this.invokeService(request, response);` ，接收参数传递给 `invokeMethod` 方法

![]()

  

接着进入 `this.invokeMethod(service, methodName, strArgs, serviceName);`
，反射调用了我们的目标方法。

![]()

  

最终来到 `FileToExcelManagerImpl` 的 `saveExcelInBase` 方法，到这里调用流程就走通了，结束。

![]()seeyon-ctp-
core.jar!/com/seeyon/ctp/common/excel/FileToExcelManagerImpl.class  
  

### 0x03 结语

  1. 使用黑白盒结合的方式去找路由规则，可以很快速的去确认一些不易看懂的路由。

  2. 想深入审计某一套代码，还是需要静下心来弄清楚它的路由分发机制；以及去分析并找到关键的过滤器、拦截器，弄清楚它们的作用及作用范围。

  3. 本次复现成功的版本为 `V8.0SP2` ，而 `V8.1SP1` 中，由于存在一个调用类、方法白名单验证，无法调用到漏洞方法，未复现成功，不排除有其它的调用方式。

  

  

说明: 文章不保证内容完全准确, 文中如有错误还请多多指出, 共同进步.

  

  

  

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

