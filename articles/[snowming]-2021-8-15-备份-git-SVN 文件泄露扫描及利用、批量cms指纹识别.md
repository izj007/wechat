# Git/SVN 泄露查询工具：

Test404 HTTP Fuzzer：

> Test404 HTTP Fuzzer v4.1.1
链接：https://pan.baidu.com/s/1twrHFhpXkdScMj64KTNA_w 
提取码：49a5 
复制这段内容后打开百度网盘手机App，操作更方便哦

插件：
>链接：https://pan.baidu.com/s/17FxHYcorPR-jP9DyxtjIGw
提取码：evsf 
复制这段内容后打开百度网盘手机App，操作更方便哦
解压密码：test404

在插件文件夹中通过 `git` 和 `svn` 关键词搜到下图中的两个插件然后放入 Test404 Fuzzer 的插件文件夹：

![title](https://leanote.com/api/file/getImage?fileId=5dc12d7cab6441404f00039c)

使用的时候勾选：

![title](https://leanote.com/api/file/getImage?fileId=5dc12dfbab6441404f0003a0)

然后在 Fuzzing 界面可以开始扫描：

![title](https://leanote.com/api/file/getImage?fileId=5dc12fc7ab6441404f0003af)

实测判断 git 泄露的插件还可以，判断 svn 泄露的插件没扫出来。SVN 判断泄露的规则根据版本大于或小于1.7是不同的。这种可以自己用 Python 写，我写了个比较简单的。主要原理是有的 SVN 泄露去访问 http://域名/.svn/entries 页面会返回一个12：

![title](https://leanote.com/api/file/getImage?fileId=5dc29932ab6441404f000bea)

大家根据实际情况在此基础上修改此代码：

``` python
import requests
import re
import os


with open("test.txt","r") as f1:
    lines = f1.readlines()
    f = open('outtput1.txt','a')
    i = 1
    for line in lines:
        if i <= len(lines):
            try:
                line = line.strip()
                target_url = line + '/.svn/entries'
                #print target_url
                print '%i.scanning:%s'%(i,target_url)
                f.write('\n%i.scanning:%s'%(i,target_url))
                rq = requests.get(target_url,headers={'Connection':'close'})
                status_code = rq.status_code
                if status_code == 200:
                    if len(rq.text)>1 and len(rq.text)<10:
                        print '[# Congratulations! ]succeed:%s'%target_url
                        f.write('\n[# Congratulations! ]succeed:%s'%target_url)
            except Exception as e:
                print '[# WARNING: ]error:%s'%target_url
                f.write('\n[# WARNING: ]error:%s'%target_url)
            i += 1
            continue

    f.close()
print "scanned finished!"
```

效果展示：

![ ](https://leanote.com/api/file/getImage?fileId=5dc4cd97ab6441040d0007c0)



# 备份文件泄露扫描：

暂时没找到很好的工具，可以根据网站域名生成的。只有一个在 7kbscan WebPathBrute 里面扒出来、整理的备份字典。 

> 链接：https://pan.baidu.com/s/1bGOiykymQ8NZy5TrCsOaBA 
提取码：f3w1 
复制这段内容后打开百度网盘手机App，操作更方便哦

然后配合 7kbscan WebPathBrute 使用：

![title](https://leanote.com/api/file/getImage?fileId=5dc12f8eab6441404f0003ae)

注意文件切分，我分成了每个文件 100 行 url 的多个小文件。


更新：

感谢 @彼此 大佬给我给了一个工具：

此工具适用于 py3 环境。里面除了字典之外，可以根据网站域名生成字典进行扫描。

> 链接：https://pan.baidu.com/s/1IX49Xl87Z0dmQVHgcsBsGA 
提取码：thx7 
复制这段内容后打开百度网盘手机App，操作更方便哦

# Git 信息泄露利用：

此工具：[lijiejie- GitHack](https://github.com/lijiejie/GitHack)

能够通过泄露的.git文件夹下的文件，重建还原工程源代码。

![title](https://leanote.com/api/file/getImage?fileId=5dc13050ab6441425d0003e8)

就会在此文件夹下生成一个以此网址命名的文件夹，里面是**尽量**（比较多 File not found 和 error）还原出来的工程源代码。


然后下一步就是对还原出来的源码查找配置文件、或者进行代码审计。

# SVN 信息泄露利用：

工具的话比如 [svnExploit](https://github.com/admintony/svnExploit/)。

还比如这个 Seay-SVN（要注意法师这个工具仅适用于 SVN 版本小于 1.7，但是 svnExploit 那个都可以，主要是路径的区别，具体区别可以参考 [svnExploit](https://github.com/admintony/svnExploit/) 的 readMe 文档）：

![title](https://leanote.com/api/file/getImage?fileId=5dc52276ab6441040d000a4c)

> 链接：https://pan.baidu.com/s/1SQ1knuH5K7-s9CtPcxExDw 
提取码：w6jy 
复制这段内容后打开百度网盘手机App，操作更方便哦


手动的话参考这篇： [墨者_SVN信息泄露漏洞分析](https://www.cnblogs.com/hilfloser/p/10517856.html)

比如我现在查到了 http://www.example.com 这个网站，扫到他的 http://www.example.com/.svn/entries 页面如下：

![title](https://leanote.com/api/file/getImage?fileId=5dc51448ab6441062900097e)

既然存在 `.svn` 信息泄露，于是开始下载 `wc.db` 文件：

通过访问 http://www.example.com/.svn/wc.db 下载 `wc.db` 文件，然后用 SQLiteStudio 软件打开 wc.db文件，我们看到 `NODES` 表，看到 `local relpath` 栏和 `checksum` 栏。

![title](https://leanote.com/api/file/getImage?fileId=5dc51665ab6441062900099a)

比如我们要下载第一个文件：

- local relpath: ad_pay.html
- checksum: `$sha1$526ef3f6d93df9f7216b9ca9358f6845a78c94f5`

那么下载地址为：

https://www.example.com/.svn/pristine/52/526ef3f6d93df9f7216b9ca9358f6845a78c94f5.svn-base

具体请参考：

- [SVN源代码泄露利用工具](http://www.admintony.com/SVN%E6%BA%90%E4%BB%A3%E7%A0%81%E6%B3%84%E9%9C%B2%E5%88%A9%E7%94%A8%E5%B7%A5%E5%85%B7.html)
- [svnExploit](https://github.com/admintony/svnExploit/)

工具介绍那里讲得非常清楚了，特别是工具的 readme 那里。

因为文件众多，所以可以通过脚本批量下载或者是通过 svnExploit 这个工具。要注意重建时候的文件结构，因为下下来是 `09d830f44a4b9bc63e462dff940ff76bd7f115b0.svn-base` 这种，注意使用 `local relpath` 来替换。最终按照文件路径来重构，这样才是完整的代码项目。

# 批量 cms 指纹识别

http://whatweb.bugscaner.com

一次可以识别100个，也有自己的 API 接口。

识别出 cms 之后可以从 cms 漏洞入手。这个网站识别速度非常快。

另外潮汐 TideSec 也是比较常用的网站，只是近期关闭了。
