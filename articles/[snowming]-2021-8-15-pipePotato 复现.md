# 0x01 漏洞环境搭建

![title](https://leanote.com/api/file/getImage?fileId=5eb53d4eab64417e59049741)

选择 Windows Server 2019 Data Center，起自带的 IIS 10 来做 WEB 服务。


![title](https://leanote.com/api/file/getImage?fileId=5eb4d1baab64410180045eb3)

>这里有个小插曲，web 目录下的文件访问404：<br/>
![title](https://leanote.com/api/file/getImage?fileId=5eb4f594ab64417e59049383)<br/>
究其原因发现：<br/>
1、 web 目录没有 `IIS_IUSRS`、`Users`权限；解决办法：给这两个用户加权限。如下图：
2、 主机名那里我写了名字 TEST，会导致服务器无法识别域名，改为空，这样所有的域名都可以解析。
3、服务端已经有一个 80 端口（IIS默认的网站），我的新测试网站也用的80端口，把默认网站服务停止就好了。<br/>
![title](https://leanote.com/api/file/getImage?fileId=5eb4f480ab6441018004920d)<br/>
最终 web 目录的访问权限如下:<br/>
![title](https://leanote.com/api/file/getImage?fileId=5eb4f6feab64417e590493c1)
<br/>
web 目录的选择要注意权限，否则后面无法对其写 shell。比如`C:\Windows\`这个目录不要选。<br/>
![title](https://leanote.com/api/file/getImage?fileId=5eb61778ab64414314012b2d)


咳，小插曲有点长了。然后配置 IIS 的 ASP 运行环境，参考本篇：[IIS7和IIS8如何配置ASP运行环境](https://jingyan.baidu.com/article/48a42057fdcd90a92525045f.html)

> 这中间也有小插曲：一开始访问 web 目录中的 txt、HTML 类型文件正常，访问 asp、aspx 就404。原因是因为IIS 10 不包含该通配符 MIME 映射，自己加了 asp 的拓展名 MIME 映射就好了。<br/>
参考这篇文章：[IIS已经存在文件，提示"HTTP 错误 404 - 找不到文件或目录](https://blog.csdn.net/u013394527/article/details/40186219)
![title](https://leanote.com/api/file/getImage?fileId=5eb507eeab64410180049322)
![title](https://leanote.com/api/file/getImage?fileId=5eb4ffaeab64417e5904944c)

注意使用默认的资源池：

![title](https://leanote.com/api/file/getImage?fileId=5eb617f9ab64414115012c44)


然后 web 目录的所有者要设为 `IIS_IUSRS`：

![title](https://leanote.com/api/file/getImage?fileId=5eb6184aab64414115012cea)




传①个 asp 的 shell 上去（免杀 defender 的）：


>如果遇到传上去的ASP 500错误参考这两篇调试：<br/>
1. [Windows7 IIS+ASP http500内部服务器错误（显示它的本来面目）](http://www.downcc.com/tech/3616.html)
2. [IIS7.5显示ASP的详细错误信息"500 – 内部服务器错误解决"](https://www.haoid.cn/post/14)


菜刀连上去：

![title](https://leanote.com/api/file/getImage?fileId=5eb61edbab64414115013ba5)

>这里也有我连菜刀的时候，触发了防火墙，导致菜刀连接时候一直遇到「与服务器的连接被重置」的错误。当我把防火墙和安全组都调好了，也就是 telnet 网站端口是通的，但是又遇到了一系列问题。解决问题的过程不表，比较琐碎。


![title](https://leanote.com/api/file/getImage?fileId=5eb61efcab64414314013be2)


权限是应用池的权限 `iis apppool\defaultapppool`，很低的权限需要提权。


# 0x02 提权过程

前提：

![title](https://leanote.com/api/file/getImage?fileId=5eb54a83ab6441018004963d)


> 实测现在对 [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) 这个项目直接编译会被 Windows Defender 杀：<br/>
![title](https://leanote.com/api/file/getImage?fileId=5eb63862ab644143140175a8)


于是对 [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) 这个项目编译改编，做点微小的免杀处理，使其不弹窗、过 defender。编译为 `test.exe`，上传到 web 目录下。执行效果：





![title](https://leanote.com/api/file/getImage?fileId=5eb623faab644143140146f4)

![title](https://leanote.com/api/file/getImage?fileId=5eb62653ab64414314014c3b)

上 CS：

![title](https://leanote.com/api/file/getImage?fileId=5eb63451ab64414314016c06)


额，还需要对 CS artifacts 做点免杀，这篇文章就到此为止吧。


----


# 参考文档：

1. [pipePotato复现](https://mp.weixin.qq.com/s/Qps-11vF4IAyYg-semzcug)，T00ls，zhiyu，2020/5/7
2. [Windows server 2019服务器的iis安装配置以及创建网站](http://www.winwin7.com/JC/18953.html)
