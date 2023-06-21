#  wmic远程执行与go版本wmiexec的bug修复

原创 楚君河  [ 雁行安全团队 ](javascript:void\(0\);)

**雁行安全团队** ![]()

微信号 YX_Security

功能介绍 四叶草安全雁行安服团队—黑客与POC的火花

____

___发表于_

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20230621160013.png)  

首先，简单介绍一下WMI， WMI是Windows
2K/XP管理系统的核心；对于其他的Win32操作系统，WMI是一个有用的插件。WMI以CIMOM为基础，CIMOM即公共信息模型对象管理器（Common
Information Model Object
Manager），是一个描述操作系统构成单元的对象数据库，为MMC和脚本程序提供了一个访问操作系统构成单元的公共接口。
总的来说，就是一个很方便我们获取系统信息的接口。

在进入内网后，如果没有防火墙的话，基本135端口都是开放的，如果我们知道UID为500的用户或者域管理员的密码，就可以通过wmic远程执行命令：

  * 

    
    
    wmic /node:192.168.1.1 /user:administrator /password:abc123!@# process call create "cmd.exe /c whoami > c:\test.txt"

但是，目前来说这种方法存在三个缺点：

1.不支持通过hash执行命令。

2.如果密码中有逗号“,”，无论是直接在cmd中执行还是写成bat，都会报错：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160015.png)

3.无法在linux下面执行。

这样就给我们的渗透过程带来不少阻碍。但是有大神已经写出各种语言的wmiexec，不过最方便的应该算golang版的，支持交叉编译，地址是这个：

https://github.com/C-Sto/goWMIExec

经过测试，发现一个问题， 当在普通环境中用500用户执行的时候，没有任何问题：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160016.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621160017.png)

但是我查看它的代码，里面虽然存在domain参数，但是没有设置获取，那么把它修改一下。 修改前：

![]()

修改后：  

![](https://gitee.com/fuli009/images/raw/master/public/20230621160019.png)

但是执行出错，爆拒绝访问错误（也就是错误5）：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160020.png)

我开始以为是域名输入错误，但是尝试test.com也不行。所以我怀疑是作者有些地方写错了。 拿出Wireshark，抓包看看。

首先用正常的wmic执行一次：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160022.png)

可以看到ntlmssp是正常地传送了域名和用户名：

![]()

接着用goWMIExec.exe执行一次：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160023.png)

发现域名无法正常传送，而且域名不为unicode字符：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160024.png)

那么大概的问题已经知道了，作者在进行ntlm认证的时候，没有将go默认的utf-8编码转换为windows的unicode编码，导致数据包在认证时无法正常识别。
尝试一下修改作者的代码。

一步一步跟进执行函数：  

![](https://gitee.com/fuli009/images/raw/master/public/20230621160025.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621160026.png)

在NewExecConfig里面domain没有经过变换就返回：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160027.png)

下一步进入NewExecer函数，我们直接跟进鉴权函数：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160028.png)

可以看到问题所在了，domain在这里没有转为unicode就直接转成byte[]进入NewSSPAuthenticate:

![](https://gitee.com/fuli009/images/raw/master/public/20230621160029.png)

那么我们可以在所有调用到NewSSPAuthenticate函数的地方，将domain先转为unicode再传入，就可以把问题解决了。
有的小伙伴就要问了，为什么不在NewExecConfig里面直接把domain转化呢？这里就说明一下，也顺便说明为什么我们能通过hash就可以登陆smb：

 **windows** **在采用 ntml** **远程鉴权的时候，不是用明文对照，而是先把密码转成 unicode** **，然后用 md4**
**加密成 hash** **（这个加密出来的就是我们经常用到的 hash** **），最后用这个 hash** **作为 hmac** **的 md5**
**的密钥，加密大写用户名 +** **域名，最后得出的结果要是跟储存的一样，就当你鉴权通过。**

所以这个域名不能先转化为unicode，否则到了最后一步就会出错。

好了，回到代码里面，作者写了一个toUnicodeS的代码，顺便拿来一用，给所有调用到NewSSPAuthenticate函数的地方都加上，原代码有两处：  

![](https://gitee.com/fuli009/images/raw/master/public/20230621160030.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621160032.png)

编译之后测试一下：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160038.png)

成功执行命令：

![](https://gitee.com/fuli009/images/raw/master/public/20230621160039.png)

只用hash也可以：

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20230621160040.png)

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

