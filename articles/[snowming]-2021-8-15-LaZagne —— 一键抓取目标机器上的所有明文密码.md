# LaZagne 介绍

![title](https://leanote.com/api/file/getImage?fileId=5da00c02ab64411ce4000261)


## 功能
LaZagne 是用于获取存储在本地计算机上的大量密码的开源应用程序。<br>每个软件都使用不同的技术（纯文本、API、自定义算法、数据库等）存储其密码。LaZagne 的作用就是找到最常用的一些软件的密码。<br>LaZagne 几乎支持市面上大部分常用工具。包括浏览器、Git、SVN、Wifi、Databases 等。但是对聊天软件的支持不够本土化，主要支持一些国外的聊天软件。

## 跨平台性
LaZagne 自身基于py,跨平台性相对较好。
但是有时候如果目标机器上没有 py 环境，我们可以把 py 转换成 exe 扔到目标机器上。

## 免杀
LaZagne 本身有exe，有一定免杀效果。<br>但是为什么说可以自己py转exe呢？<br>一是我们可以用全新的环境打包（就是只装需要的包，其他的一概不用）这样可以减小一点exe程序的体积，不然生成的exe程序会非常大（9M左右？）。另外在`XP`环境下打包也可以减少一点体积。<br>二是因为时至今日LaZagne的Releases里面的exe肯定已经被各种杀软记录了md5。自己打包，至少打包出来的exe的md5是新的，从文件md5层面可以稍微的绕过杀软。

## 权限问题
> 实战中务必要在一个较高权限[最好是提权后的root或system]下运行,否则你很可能什么都抓不到。<br>实战中用过几次,主要是想用它来搜集内网机器上的各种密码,但,并不是特别靠谱,有些行为还是很容易被杀软捕捉到,自己如果不会免杀,就很头疼了。

也就是说，现在可能绕不过一些杀软的行为检测。
不过我在我本机用普通域用户的权限尝试了一下这个软件，抓出来了很多密码。但是主要是浏览器层面的。

项目里面作者提醒了：
![title](https://leanote.com/api/file/getImage?fileId=5d9ff903ab64411aeb0001c2)

如果不使用高权限管理员账号，抓不出来一些Windows秘钥或者Wifi密码。

--------------

# 用法
该工具分 Windows 版本、Linux 版本、Mac版本：
![title](https://leanote.com/api/file/getImage?fileId=5d9ffbffab64411aeb0001e2)
里面是python脚本。
也有直接的exe版本。

## 抓取所有支持软件的密码：
``` bash
laZagne.exe all
```

## 抓取特定一类软件的密码：

如，抓取浏览器：
``` bash
laZagne.exe browsers
```

## 抓取特定一个软件的密码：
如，抓取火狐：
``` bash
laZagne.exe browsers -firefox
```

## 把所有的密码写入一个文件：
- `-oN` 写成普通 txt 格式
- `-oJ` 写成 Json 格式
- `-oA` 写成所有的格式 
``` bash
laZagne.exe all -oN
laZagne.exe all -oA -output C:\Users\test\Desktop
```
- `注意：`如果在解析以多行字符串形式编写的JSON结果时遇到问题，参考这个 [issue](https://github.com/AlessandroZ/LaZagne/issues/226)。

## 获取帮助：
``` bash
laZagne.exe -h
laZagne.exe browsers -h
```
## 更改模式（2个不同模式）：
- 详细模式
``` bash
laZagne.exe all -vv
```
- 安静模式（标准输出上不会打印任何内容）
``` bash
laZagne.exe all -quiet -oA
```
## 解密域凭据：

要解密域凭据，可以通过指定用户 Windows 密码的方式来完成。否则，它将尝试将所有已找到的密码作为Windows密码来进行解密。
``` bash
laZagne.exe all -password ZapataVive
```

## 权限问题：
注意：对于wifi密码\ Windows Secrets，请以管理员权限启动它（UAC身份验证 / sudo）

## For Mac OS：
上面的通用用法是针对 Linux 和 Windows 的，也可以根据情况把 exe 换成 py 脚本。
但是，在Mac OS系统中，如果没有用户密码，则很难检索存储在计算机上的密码。因此，LaZagne 作者建议使用以下选项之一：

1. 如果知道用户密码，把用户密码作为选项值加入命令行：
``` bash
laZagne all --password SuperSecurePassword
```
2. 可以使用交互式模式，该模式将向用户提示对话框，直到密码正确为止：
``` bash
laZagne all -i
```
--------------

# 实际效果
测试的版本为`LaZagne-2.4.3`，适用于 python3 的环境。需要安装几个依赖，但是问题不大。没遇到什么麻烦。

## 普通用户权限
普通域用户权限下跑：
![title](https://leanote.com/api/file/getImage?fileId=5da00fd5ab64411aeb000236)

跑出来138个账号密码：
![title](https://leanote.com/api/file/getImage?fileId=5da01072ab64411ce4000273)

其中包括Firefox浏览器上面存过的一些密码、Google chrome passwords、Wsl（Winows10下面的Linux主机）账号密码、一些3389登录过的远程windows vps的密码等......

## 系统权限
System权限下面跑：
![title](https://leanote.com/api/file/getImage?fileId=5da011bdab64411aeb000241)
![title](https://leanote.com/api/file/getImage?fileId=5da0127dab64411ce400027c)

这个软件会分用户跑：System\Administrator\PC......
![title](https://leanote.com/api/file/getImage?fileId=5da01282ab64411aeb000245)
![title](https://leanote.com/api/file/getImage?fileId=5da0128aab64411aeb000246)

最后跑出来152个账号密码：
![title](https://leanote.com/api/file/getImage?fileId=5da01291ab64411ce400027d)

其中的东西也差不多，主要是浏览器的，可能多了一些 Windows 相关的凭据和 Dumps from memory（Keepass 和 Mimikatz method 等）。看了下，貌似没跑出来 WIFI 密码。

## Linux 环境下
在 windows 下面直接跑exe是没问题的：
![title](https://leanote.com/api/file/getImage?fileId=5da01736ab64411ce40002a2)

但是Linux下面遇到了问题：
![title](https://leanote.com/api/file/getImage?fileId=5da01777ab64411aeb000260)

跑py脚本的话依赖也有一些问题，懒得解决了，需要用的时候再解决。

---------

# 特别注意
在尝试的过程中，遇到了一个乌龙。如下图，当时惊呆了，我这不会抓到了域管的密码吧！但是这个软件其实只能抓本地的密码。退一万步讲，如果真的抓到了管理员密码，也只可能是本机的管理员的密码。更何况我在抓到这个密码的时候，用的普通用户权限，一般来说，是不可能抓到 Windows 秘钥的。

![title](https://leanote.com/api/file/getImage?fileId=5da01854ab64411aeb000266)

最终搞清楚了，这个是我3389登陆过的远程Windows vps的账号密码。也不知道为什么前面加了个域的名字的前缀。

## 总结：
laZagne 只可能抓到本机上的密码！不可能抓到域管的密码。但是呢，如果这台机器是域控，或者你用一台内网机器的密码去撞一个网段，就是另一个故事了！权限一定要注意！！
Linux 注意 `sudo`!

另外有朋友说，laZagne 的新版本抓到的密码没有旧版本的多，可以自行实验吧，感觉可能差不太多。

--------------

参考链接：
[1] [一键抓取目标机器上的所有明文密码](https://klionsec.github.io/2017/10/26/LaZagne/)， klion的博客，klion，2017年10月26日
[2] [LaZagne](https://github.com/AlessandroZ/LaZagne)

