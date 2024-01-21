#  [逆向Tips]Webpack加密算法细抠

原创 Captain0X  [ 森柒柒 ](javascript:void\(0\);)

**森柒柒** ![]()

微信号 gh_89d057f5542d

功能介绍 分享知识，让一个人的快乐变成一群人的快乐！

____

___发表于_

特别说明:文章中提及的所有知识点仅供学习，切勿用作非法用途。

![]()  

前言

昨天看到一个使用了wabpack的站点，登录处密码使用RSA加密算法，由于太久没有精抠代码，有些生手，今早专门复习webpack结构，研究出了它的规律，写下文章记录过程。
**webpack的原理其实很简单，就是写一个加载器，然后使用字典或者数组保存要加载的代 码块。**

![]()  

问题

  
那么问题只有两个：  
1.找到加载器。  
2.找到要用的模块。  

  

![]()  

实战

  

1.打开目标登录页面，输入账号密码  

![]()

  
2.发现密码是一长串加密字符，这时候，可以盲猜它就是rsa，别问为什么，经验之谈。  

![]()

3.定位加密算法位置  

![]()

![]()

  

4.根据上图可知 a.a.token.encrypt就是加密算法，那么我们断点进去encrypt方法看看  

![]()

  
  
5.这里使用B进行调用encrypt方法，我们继续追踪B在哪赋值的  

![]()

从上可看到B设置了rsa公钥  
B是由c.a赋值而来，继续跟踪c的赋值位置  

![]()

c = n.n(i)  
而i = n(863)  
  
此时，在此行代码打上断点，重新刷新页面。  

![]()

  

断点断住的同时在控制台输出n查看它的来源，我们要跟踪到n的位置，因为它是全局的加载器。  

  

![]()  

分析加载器的结构，一般是由自执行函数包裹加载器  

  

![]()

  

6.我们先把这个自执行函数赋值下来，保存到新建的rsa_enc.js文件

  

![]()

  
折叠一下js代码，把加载器后面的所有代码都删掉，因为它们是用不上的，也可以不删，防止报错，我一般都会删。  

  

  

![]()

  

观察这个自执行函数的结构，我们发现它传进去的是一个空的数组参数，因为它还没指定要加载什么模块，那接下来我们就要找到我们要加载的加密模块了。  
  

7.回到encrypt方法，鼠标点一下该方法，我们可以跟踪到它的来源

  
  

![]()

![]()

  

鼠标从这里出发，继续往上滑，直到看到最左边出现一逗号以及函数，这个逗号就是数组分隔逗号，它的形式应该是这样的：  

[模块1，模块2，模块3.....]

![]()

  

把它粘贴到rsa_enc.js文件的开头处，也就是加载器代码的前面。并把它赋值给module变量。  

![]()

  

接着把module写入到自执行函数的数组里面  

![]()

  

在内部的加载器代码下方写入一个代码l(0)，意思就是让加载器加载传进来的module模块。  

![]()

  

在代码最初的位置加上下面两行代码补充运行环境

  *   * 

    
    
    window = {};navigator = { "appName": 'Netscape' }

![]()

最后构造一个加密算法。  

  *   *   *   *   * 

    
    
    function get_rsa_encrypt(password) {    rsa_encrypt = new window.JSEncrypt;    rsa_encrypt.setPublicKey("MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDFE46eRC1GbfT7VpV0auVBleMDSRRBdO44zWF5c7DQKQSu6cUODk79D2fz6rf6DkptK5CacbHR+gfa5+apn+DjoKlJlX1bIgTzsac2EORWrnZPdOJNHZgAqnHwTbr9u9R2DQdnfBAU+slIcpaNEaoOGX5FlfnJZ03N7h/9Q5TpJwIDAQAB")    return rsa_encrypt.encrypt(password)}

![]()

这就是完整的加密算法精抠过程。  
我们使用nodejs调用试试。  

node rsa_enc.js

![]()

  
收工！  

![]()

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# [逆向Tips]Webpack加密算法细抠

原创 Captain0X  [ 森柒柒 ](javascript:void\(0\);)

轻触阅读原文

![]()

森柒柒

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

