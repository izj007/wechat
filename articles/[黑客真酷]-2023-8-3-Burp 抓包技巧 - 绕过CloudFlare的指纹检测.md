#  Burp 抓包技巧 - 绕过CloudFlare的指纹检测

原创 110hacker  [ 黑客真酷 ](javascript:void\(0\);)

**黑客真酷** ![]()

微信号 gh_ddeb734f0ee7

功能介绍 主要分享自己追逐成为一名合格黑客的成长经历，一路上有风雨、有荆棘、也有鲜花和彼岸，更有我想与你分享的满满热情，期待与你共同进步。

____

___发表于_

收录于合集

# 0x00 前言  

本系列主要分享笔者日常在Burpsuite这个工具使用方面的一些经验心得。此文背景来源于笔者某次通过BurpSuite自带的浏览器访问某个网站的时候出现永久重定向导致无法抓包的现象，出于好奇，故对此进行了一番小研究实现Bypass，虽然后面发现弄了个乌龙，但过程颇为有趣，故记录下过程并作为本次分享内容。

## 0x01 分析

实验环境

> BurpSuite Version: 2023.7

目标站点:

> http://www.xxxsir.shop/

1）使用火狐浏览器，通过SwitchyOmega插件抓包，可以实现正常抓包，对应的Response如下:

![]()

2）使用Burp自带的浏览器抓包,就会出现不断重定向的情况，不能正常访问到目标网站

history 一直显示302

![]()![]()

为了弄清楚具体请求的情况，开启Intercept is on， 查看Response

![]()

相应地返回状态码302

![]()

3）分析原因

简单比较下两个HTTP请求的数据，很容易就可以知道，差异出在了 **UA** 上面。

默认Browser UA:

    
    
    1Mozilla/5.0 (iPhone; CPU iPhone OS 14_8_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gec1ko) Mobile/15E148 MicroMessenger/8.0.37(0x1800252e) NetType/WIFI Language/en  
    

当我尝试删掉其中的一部分比如括号里面的内容就正常

![]()

至此，可以得出结果: **CloudFlare 针对 BurpSuite Browser FingerPrint 进行了拦截**

4）解决办法

利用BurpSuite的 Tools -> Proxy -> Match and replace rules 功能，开启其中的Emulate OS
模拟UA规则即可实现Bypass

![]()

## 0x02 乌龙事件

一开始我想着这种方式也太不优雅了，有时候有一些请求是我不想改动UA的，所以我寻思着，能不能有一种一劳永逸的办法呢？比如直接在Chromium程序上面改配置、硬编码之类的办法。

找到Chromium可执行程序的安装路径

    
    
    1/Applications/Burp Suite Professional.app/Contents/Resources/app/burpbrowser/114.0.5735.198/Chromium.app/Contents/MacOS  
    

搜索一下字符串看看有没有硬编码UA的部分内容，可以尝试ida patch下。

    
    
    1strings Chromium|grep "Chrome"  
    2strings Chromium|grep "Mozilla"  
    

不过很可惜的是，没有直接找到类似的字符串，由于chromium是开源的，所以我们可以直接阅读它的源码查看UA生成规则

1）获取UA头的代码

https://source.chromium.org/chromium/chromium/src/+/main:content/common/user_agent.cc

![]()

2）获取版本号代码

https://source.chromium.org/chromium/chromium/src/+/main:components/version_info/version_info.cc;l=17

![]()

基于此，我得到一个非常关键的信息，那就是 Chronium
默认的UA是存在固定特征的，比较一下那个被拦截的UA，可以看到是明显不同的，这个被拦截的UA应该是我曾经IOS上面抓包的时候获取到，应该不是BurpSuite默认的UA。

    
    
    1Mozilla/5.0 (iPhone; CPU iPhone OS 14_8_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gec1ko) Mobile/15E148 MicroMessenger/8.0.37(0x1800252e) NetType/WIFI Language/en  
    

于是，我回头查看默认的BurpBrowser的配置，发现自己似乎启用 【User-Agent Switcher and Manager】这个插件。

![]()

尝试关闭这个插件之后，获取到默认UA如下，这样子就符合UA生成的规则了。

    
    
    1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36  
    

## 0x03 思路延伸

关于 User-agent 的处理，打开 chrome 浏览器, 正常获取到的UA应该是符合Chronium的UA源码规则，具有两个动态获取的变量。

    
    
    1Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36  
    

变量对应的值分别是

os_info 对应的 -> Macintosh; Intel Mac OS X 10_15_7

product 对应的 -> Chrome/115.0.0.0  (MajorVerion.x.BuildVersion.x)

不过，需要注意地是，如果是默认安装 BurpSuite 自带的 chronium 浏览器的，你会发现无论是在 Linux、Mac
或者是Window，它的UA除了版本号，其他值是固定。

为什么会出现这个情况呢？我的猜想如下，可能是在构建时指定为Window，或者说Chromium编译的时候本身会存在这样的一个问题，但是由于编译Chromium的相关资料较少，笔者就没有再深入去验证，如果有大师傅熟悉这一块，欢迎给我留言解惑，Thanks！

![]()

基于上述的研究和实验，不难可以得到绕过这类拦截的一个思路， **那就是最大化伪装成一个正常用户**
，即UA符合常规chrome的UA规则，所以，当你下次再遇到请求被拦截的时候，这个时候，你只需要观察一下UA特征就可以拥有判断这个UA是否正常的能力了。

同时，像先前那个被CF拦截的UA，我称之为魔法黑名单UA，当你用这类UA去访问的时候，你会发现任何CF保护下的站点，你就会遭遇到拦截，至于CF为什么会这样做一个黑名单匹配，我也不知道，要不然我为什么称之为魔法呢？

## 0x04 全文总结

真正在实战中，Burp抓包遇到拦截的问题成因是非常多的，上述仅仅是最为简单的一种。不过，本文通过这一次简单的事件进行了一次较为详尽的分析和实验，虽然有乌龙，但也总结出了部分Burp的特点，并提出了一些绕过的思路，同时也留下了一些疑问。所谓成长嘛，那就让未来的自己慢慢解决历史遗留的问题吧！最后，不得不感谢每一个耐心看完本文的每一个读者。

## 0x05 Reference

https://blog.cloudflare.com/monsters-in-the-middleboxes/

https://stackoverflow.com/questions/70129432/how-to-bypass-cloudflare-
protection-with-burp

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

