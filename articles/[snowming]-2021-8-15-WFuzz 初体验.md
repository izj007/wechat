知道WFuzz这款工具是因为一次 SSRF 内网探测有 fuzz 需求，一个老哥给我推荐了这款 fuzz 工具。

WFuzz 是一个【模糊测试】工具。历史悠久（10年+），功能丰富（可用作 web 漏扫、爆破工具）...... 

WFuzz 的两大优势是：
 1. Kali 自带； 
 2. 有 `Python` 接口。

![title](https://leanote.com/api/file/getImage?fileId=5da11b3aab64411aeb0006a9)

`注:`不支持https网站
![title](https://leanote.com/api/file/getImage?fileId=5da11ca1ab64411aeb0006af)

--------------

# 爆破文件和路径
文件、路径爆破的成功与否很大程度上要依赖于使用的字典。

wfuzz自带一些字典文件，更多的字典可以参考下面两个开放的git：
- fuzzdb
- seclists
注：这两个是比wfuzz更强大的fuzz项目。

使用wfuzz暴力猜测目录的命令如下：
``` bash
$ wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://testphp.vulnweb.com/FUZZ
```

使用wfuzz暴力猜测文件的命令如下：
``` shell
$ wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://testphp.vulnweb.com/FUZZ.php
```

## 目录爆破尝试：

命令：

``` shell
wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://testphp.vulnweb.com/FUZZ
```

![title](https://leanote.com/api/file/getImage?fileId=5da120ccab64411aeb0006ce)

爆破出来的结果是：
![title](https://leanote.com/api/file/getImage?fileId=5da12142ab64411ce40006e8)
响应码会有变化。有的是301。

404 的响应码本应该是被忽略的：
![title](https://leanote.com/api/file/getImage?fileId=5da12198ab64411ce40006e9)

我们加上 `--hc 404` 再来尝试：
![title](https://leanote.com/api/file/getImage?fileId=5da121f5ab64411aeb0006d1)
![title](https://leanote.com/api/file/getImage?fileId=5da12211ab64411ce40006ec)

结果就都是想要的。

随便试几个爆破出来的路径：
![title](https://leanote.com/api/file/getImage?fileId=5da1226bab64411ce40006ee)
![title](https://leanote.com/api/file/getImage?fileId=5da1229eab64411aeb0006d3)

也可以把爆破结果写入一个文件：
![title](https://leanote.com/api/file/getImage?fileId=5da12783ab64411ce4000714)

![title](https://leanote.com/api/file/getImage?fileId=5da12a98ab64411aeb000741)

## 文件爆破尝试：
命令：
``` shell
wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://testphp.vulnweb.com/FUZZ.php
```

![title](https://leanote.com/api/file/getImage?fileId=5da128b4ab64411ce400071b)

`注：`我直接加上了`--hc 404`进行过滤，结果会明了很多。

![title](https://leanote.com/api/file/getImage?fileId=5da1291bab64411ce400071d)

注意结果里面这两列，个人感觉相当于响应包的长度。
![title](https://leanote.com/api/file/getImage?fileId=5da12957ab64411ce400071e)

查看一个0字节的页面：
![title](https://leanote.com/api/file/getImage?fileId=5da129feab64411ce4000721)

----------------

# 爆破用户名和密码
在上面发现的这个网页中，我们想尝试一下爆破：
![title](https://leanote.com/api/file/getImage?fileId=5da12bc1ab64411aeb00074d)

请求地址为：http://bembala.net/admin.php

POST请求正文为：user&password=
![title](https://leanote.com/api/file/getImage?fileId=5da12c43ab64411aeb000751)

使用wfuzz测试（使用了两个本地的字典）：
``` shell
wfuzz -w /root/userName.txt -w /root/passWord.txt -d "user=FUZZ&password=FUZ2Z" http://bembala.net/admin.php
```
`-d`参数传输POST请求正文。

注意这里的占位符不是随便设定的：
![title](https://leanote.com/api/file/getImage?fileId=5da132eeab64411ce4000758)

我之前随便把第二个占位符设为`FUZZ2`，就报错了：
![title](https://leanote.com/api/file/getImage?fileId=5da13349ab64411ce400075e)

然后开始跑：
![title](https://leanote.com/api/file/getImage?fileId=5da13375ab64411aeb000782)

但是没跑出来结果......
![title](https://leanote.com/api/file/getImage?fileId=5da132afab64411ce4000757)

可能密码并不弱或者字典不够强把。

于是换个目标来测试吧，换一个我已知的弱密码登陆页面来试试：
![title](https://leanote.com/api/file/getImage?fileId=5da137f0ab64411aeb0007ad)

所以 payload 应该为:
``` shell
wfuzz -w /root/userName.txt -w /root/passWord.txt -d "login=true&username=FUZZ&password=FUZ2Z" http://202.101.*.*:9090/login.jsp
```

然后就看到 WFUZZ 开始疯狂跑，有种无止无尽的感觉：
![title](https://leanote.com/api/file/getImage?fileId=5da139ccab64411aeb0007b4)

那怎么找出爆破成功的那个结果呢，可以看到上图中，响应码都是200，唯有对响应长度进行区分了：
![title](https://leanote.com/api/file/getImage?fileId=5da13a3aab64411aeb0007b7)

就看 Lines 一行就行。在爆破成功的结果应该就是 Lines 不等于 `148L` 的那对账号密码组合。

## 结果过滤
找了点关于结果过滤的资料，发现要使用正则：

![title](https://leanote.com/api/file/getImage?fileId=5da13aa9ab64411aeb0007bc)

![title](https://leanote.com/api/file/getImage?fileId=5da13b25ab64411aeb0007d6)

构造payload为:
``` shell
wfuzz -w /root/userName.txt -w /root/passWord.txt -d "login=true&username=FUZZ&password=FUZ2Z" --hs "148L" http://202.101.*.*:9090/login.jsp
```

本以为现在就可以过滤掉包含`148L`的结果，但是跑出来还是这样：
![title](https://leanote.com/api/file/getImage?fileId=5da13cb6ab64411aeb0007e5)

很明显正则没写对呗。找原因很快就发现，过滤的关键字不是`--hs`，而应该是`--hc`或者`--ss`。

![title](https://leanote.com/api/file/getImage?fileId=5da13d46ab64411aeb0007e9)

上图中这个博客写错了。

`注意`：过滤的关键字不是`--hs`，而应该是`--hc`或者`--ss`！

换了关键字继续跑：
![title](https://leanote.com/api/file/getImage?fileId=5da15c56ab64411ce4000886)

这里可能把服务器短期跑出来什么毛病了，所以正确密码也没有什么特殊长度和响应码。
![title](https://leanote.com/api/file/getImage?fileId=5da141d6ab64411ce40007f6)

于是我登陆了一下正确账号密码，发现正确动作应该是一个跳转，所以重新写payload，正则匹配到一个302跳转。

![title](https://leanote.com/api/file/getImage?fileId=5da142c0ab64411aeb000817)
（出处：https://www.freebuf.com/column/163553.html）

payload:
``` shell
wfuzz -w /root/userName.txt -w /root/passWord.txt -d "login=true&username=FUZZ&password=FUZ2Z" --sc 302 http://202.101.*.*:9090/login.jsp
```
![title](https://leanote.com/api/file/getImage?fileId=5da14388ab64411ce40007fb)

![title](https://leanote.com/api/file/getImage?fileId=5da14526ab64411aeb00081f)

然后居然没结果。我自己又登陆了一次正确账号和密码:
一开始登不上：
![title](https://leanote.com/api/file/getImage?fileId=5da1458aab64411aeb000822)
过了一会能登上了，赶快再跑一次(用小点的字典):
![title](https://leanote.com/api/file/getImage?fileId=5da1460fab64411ce4000806)

终于跑出来了！总之这个网站一跑就会暂时不让登陆（正确的也登不上去），但是过一会儿就好了。害我折腾了很久。

## 再次尝试

换一个健壮一点的站点尝试：

![title](https://leanote.com/api/file/getImage?fileId=5da146afab64411ce400080b)

![title](https://leanote.com/api/file/getImage?fileId=5da146d4ab64411aeb000826)

![title](https://leanote.com/api/file/getImage?fileId=5da1470dab64411aeb000827)

果断跑出来了账号密码：

![title](https://leanote.com/api/file/getImage?fileId=5da14762ab64411ce4000810)

-----------------

# 总结
WFuzz核心是使用了FUZZ作为占位符，使得它能胜任更多的扫描爆破任务。

 - `-w` 参数指定字典。 
 - `-d`参数传输POST请求正文。 
 - 多个 payload 时候，占位符为：`FUZZ`,`FUZ2Z`...,`FUZnZ`，不能自己随便自定义。
 - 【结果过滤】`--hc`或`--ss`不显示符合条件的结果。
 - 【结果过滤】`--sc`或`--sl`或`--sw`或`--sh`显示符合条件的结果。

WFuzz 的功能还有很多很多，以后会继续在真实场景中去熟悉其使用。我在这篇文章写作的时候，就是对着【史上最详[ZI]细[DUO]的wfuzz中文教程】和【wfuzz使用手册】这两篇想用什么功能就用关键字去搜索语法，现学现用、挺好使的。

--------------

### 参考链接（排名有先后）：
[1] [wfuzz使用手册](https://hack4.fun/2019/04/wfuzz%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C/)，https://hack4.fun/，TOKYOCOLQ，2019年4月23日
[2] [史上最详[ZI]细[DUO]的wfuzz中文教程（一）-初识wfuzz](https://www.freebuf.com/column/163553.html)， FREEBUF，m0nst3r,2018年3月4日
[3] [史上最详[ZI]细[DUO]的wfuzz中文教程（二）——wfuzz 基本用法](https://www.freebuf.com/column/163632.html)，FREEBUF，m0nst3r，2018年2月28日
[4] [史上最详[ZI]细[DUO]的wfuzz中文教程（三）-wfuzz高级用法](https://www.freebuf.com/column/163787.html)，FREEBUF，m0nst3r，2018年3月1日
[5] [史上最详[ZI]细[DUO]的wfuzz中文教程（四）—— wfuzz 库](https://www.freebuf.com/column/164079.html)， FREEBUF，m0nst3r，2018年3月4日
[6] [Wfuzz - Github 项目](https://github.com/xmendez/wfuzz)
[7] [WFUZZ使用教程](https://www.fuzzer.xyz/2019/03/29/WFUZZ%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/)，Ethan's Blog，Ethan，2019年3月29日
[8] [WFUZZ大法](https://wh0ale.github.io/2019/03/02/2019-3-2-WFUZZ%E5%A4%A7%E6%B3%95/)，Wh0ale's Blog，Wh0ale，2019年9月29日 ← 此博文中有错误，谨慎阅读
[9] [Scan for shellshock with wfuzz](http://edge-security.blogspot.com/2014/10/scan-for-shellshock-with-wfuzz.html)，Security on the edge，2014年10月30日


 
