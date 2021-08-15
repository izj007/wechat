# 0x01 问题描述

遇到了同如下的问题，在网上没有找到答案：

![title](https://leanote.com/api/file/getImage?fileId=5f46504bab644105f500190b)

# 0x02 原理


PE 的头文件中有一处信息是关于证书的，就是 `Certificate Table`：


![title](https://leanote.com/api/file/getImage?fileId=5f464acaab644105f50018d8)

这个 `Certificate Table` 位于哪里呢？
位于：`NT Optional Header` →  `DATA_DIRECTORY` →  `Certificate Table`


>下图都出自 MSDN 文档：[Optional Header Data Directories (Image Only)](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#optional-header-data-directories-image-only)

![title](https://leanote.com/api/file/getImage?fileId=5f464cf6ab644107f200194c)

![title](https://leanote.com/api/file/getImage?fileId=5f464d51ab644105f50018ea)

![title](https://leanote.com/api/file/getImage?fileId=5f464d6eab644107f200194f)

让我们来看3个 .exe 文件，分别看他们的 `Certificate Table`：

有证书的 exe：

![title](https://leanote.com/api/file/getImage?fileId=5f464eb0ab644105f50018f6)


没证书的 exe 之一：

![title](https://leanote.com/api/file/getImage?fileId=5f464efcab644105f50018f9)


没证书的 exe 之二：

![title](https://leanote.com/api/file/getImage?fileId=5f464f42ab644105f50018fd)


应该可以看出来区别了：

没有证书的 exe，其 `Certificate Table` 的 `VirtualAddress` 和 `Size` 字段对应的值都为 0。很好理解吧，没有证书，那证书的大小和文件偏移肯定为0鸭。


# 0x03 代码实现

使用 Python 解析 PE 的库 `pefile`。

先查看 NT 可选头中的 DATA_DIRECTORY 数组：


```
def hv_cer():
    PEfile_Path = "D:\A\\abexcm1.exe"
    pe = pefile.PE(PEfile_Path)
    print pe.OPTIONAL_HEADER.DATA_DIRECTORY
```

得到的结果是：

![title](https://leanote.com/api/file/getImage?fileId=5f465194ab644105f5001910)



这对应着这些表：



![title](https://leanote.com/api/file/getImage?fileId=5f4651e1ab644105f5001914)
![title](https://leanote.com/api/file/getImage?fileId=5f465213ab644105f5001918)

> 图出自：[Optional Header Data Directories (Image Only)](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#optional-header-data-directories-image-only)， MSDN


可以看到我们要找的是此数组中的第5个结构体，数组下标4的元素 `[IMAGE_DIRECTORY_ENTRY_SECURITY]`。打印出来看看：


```
def hv_cer():
    PEfile_Path = "D:\A\\abexcm1.exe"
    pe = pefile.PE(PEfile_Path)
    print pe.OPTIONAL_HEADER.DATA_DIRECTORY[4].name
    print pe.OPTIONAL_HEADER.DATA_DIRECTORY[4]
```

![title](https://leanote.com/api/file/getImage?fileId=5f4652f0ab644105f500191f)

到此就非常好说了，我使用了 python 的 re 模块的正则搜索来判断那两处是否为0：

```
def hv_cer(PEfile_Path):
    pe = pefile.PE(PEfile_Path)
    IMAGE_DIRECTORY_ENTRY_SECURITY = str(pe.OPTIONAL_HEADER.DATA_DIRECTORY[4])
    pattern1 = r'VirtualAddress:\s+0x0\b'
    pattern2 = r'Size:\s+0x0\b'
    if (re.search(pattern1, IMAGE_DIRECTORY_ENTRY_SECURITY) is None and re.search(pattern2, IMAGE_DIRECTORY_ENTRY_SECURITY) is None):
        print('{0} have certificate!'.format(PEfile_Path))
```

>注意这里有一个坑：re 模块因为单行\多行模式的原因，影响了正则中 `^` 和 `$` 的行为，用 `\b` 单词边界来匹配 pattern 的末尾比较好。
参考：[Python正则表达式中的re.S，re.M，re.I的作用](https://www.cnblogs.com/feifeifeisir/p/10627474.html)

# 0x04 测试结果

对于带数字签名的 exe `strings64.exe`：

![strings64.exe](https://leanote.com/api/file/getImage?fileId=5f4654a2ab644107f200198b)


![title](https://leanote.com/api/file/getImage?fileId=5f4654dcab644107f200198f)


对于不带数字签名的 exe `wusa.exe`：


![title](https://leanote.com/api/file/getImage?fileId=5f465529ab644107f2001993)


![](https://leanote.com/api/file/getImage?fileId=5f46554aab644105f5001933)


附上成品代码：

```
import os, string, re
import pefile


def hv_cer(PEfile_Path):
    pe = pefile.PE(PEfile_Path)
    IMAGE_DIRECTORY_ENTRY_SECURITY = str(pe.OPTIONAL_HEADER.DATA_DIRECTORY[4])
    pattern1 = r'VirtualAddress:\s+0x0\b'
    pattern2 = r'Size:\s+0x0\b'
    if (re.search(pattern1, IMAGE_DIRECTORY_ENTRY_SECURITY) is None and re.search(pattern2, IMAGE_DIRECTORY_ENTRY_SECURITY) is None):
        print('{0} have certificate!'.format(PEfile_Path))
```

-------------

# 参考文档：



1. [恶意文件分析系统中的数字签名验证](http://blog.nsfocus.net/digital-signature-with-malware-analysis/)，绿盟技术博客，2015-07-29，参考 PE 查找证书的原理
2. [Optional Header Data Directories (Image Only)](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#optional-header-data-directories-image-only)， MSDN， 参考 PE 结构的权威资料
3. [Python读写PE文件模块pefile](http://www.pythonclub.org/modules/pefile#dokuwiki__top)，Python 俱乐部，参考 pefile 库的用法
4. [erocarrera/pefile](https://github.com/erocarrera/pefile)，Github pefile 库项目地址
5. [pefile/ReadingResourceStrings.md](https://github.com/erocarrera/pefile/blob/wiki/ReadingResourceStrings.md)，Github，pefile 库的一个比较有用的用法示例
6. [Python正则表达式中的re.S，re.M，re.I的作用](https://www.cnblogs.com/feifeifeisir/p/10627474.html)，博客园，傻白甜++，2019-3-30，参考 Python re 模块多行模式对于正则符号 $ ^ 的影响
7. [PE Format Manipulation with PEFile](https://axcheron.github.io/pe-format-manipulation-with-pefile/)，BreakInSecurity，Alexandre CHERON，2017-12-12，参考 pefile 库的详细用法