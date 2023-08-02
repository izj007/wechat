#  Wfuzz - 体验百万字典爆破edu.cn域名

原创 110hacker  [ 黑客真酷 ](javascript:void\(0\);)

**黑客真酷** ![]()

微信号 gh_ddeb734f0ee7

功能介绍 主要分享自己追逐成为一名合格黑客的成长经历，一路上有风雨、有荆棘、也有鲜花和彼岸，更有我想与你分享的满满热情，期待与你共同进步。

____

___发表于_

收录于合集

# 0x01 前言  

简单记录一下自己玩弄某些工具的测试过程，提高自己对安全类暴力测试的边界和最终效果的整体认知，同时记录过程发布在自己的公众号，丰富下自己的互联网回忆录。

## 0x02 安装

首先，介绍下基于pip安装的方式

本文环境 -> 某云4h4g8m ubuntu系统

    
    
    sudo apt install python3-pip  
    pip install wfuzz  
    

不过事情没有那么顺利，反手来一个错误

![]()

执行下面命令修复一下

    
    
    sudo apt-get install libssl-dev libcurl4-openssl-dev python-dev  
    

  
![]()

安装成功的同时，为了方便调用，最好是把可执行文件安装目录添加到环境变量

    
    
    # 编辑 /etc/profile 是最直接暴力且有效的方式  
    sudo vim /etc/profile  
    # 添加路径到最后一行  
    export PATH=$PATH:/home/ubuntu/.local/bin  
    

 **当然如果你懒得折腾的话，我推荐你用docker，基本不用考虑这些事情，命令一把梭就行，简单又高效，还方便做资源控制**

    
    
    sudo docker run -v $(pwd)/wordlist:/wordlist/ -it ghcr.io/xmendez/wfuzz wfuzz  
    

## 0x03 爆破 edu.cn 一级子域名

### 0x3.1 暴力哲学

eTLD+1 是edu.cn,那么xxx.edu.cn的xxx应该就是一级子域名，凭据自己的一些经验，初步的想法呢

    
    
    [0-9a-z-]{1,7}  
    

简单计算下来就是 10+26+1 = 37,即37^7 =  94,931,877,133
似乎不太现实，后面想想得根据人性化方向来优化下，一般来说作为一级教育单位的域名，多以全拼音为主，所以我先尝试了这样的一个规则

    
    
    [a-z]{2,7}  
    

爆破量基本去到了3亿次，嗯嗯，问题不大也就是几天的事情，还是可以期待下的，说干就干(年轻，直接Killed)

1） 安装 Crunch 生成下规则字典

    
    
    crunch 2 5 abcdefghijklmnopqrstuvwxyz -o password_list.txt  
    

有一说一，我是真的放弃了2-7的字典规则，妥协，必须妥协好吧，2-5长度的规则内存占用都给我干掉2GB了，这种是指数爆炸增长的，吓人好吧，脑子真没概念。

![]()

2）Wfuzz 测试

字典已经就绪

![]()

如用如下Wfuzz 命令，相关解读如下

> 介绍下命令的参数
>
> -c 颜色输出
>
> -f 输出命令执行内容
>
> -z 处理payload
>
> -w fuzz字典
>
> \--sc Show responses with the specified code/lines/words/chars
    
    
    sudo wfuzz -c -f sub-fighter.txt -Z -w password_list.txt --sc 200,202,204,301,302,307,403 http://FUZZ.edu.cn  
    

启动的时候，可以看到 wfuzz 的CPU占用有多高，暴力测试很多时候反而非常考验CPU性能，要不然直接卡成PPT。

![]()

嗯，非常漂亮，等待了几十秒之后，很好直接把自家服务器打挂了。

![]()

为了方便定位下程序killed的原因，需要开启一下系统日志记录

    
    
    vim /etc/rsyslog.d/50-default.conf  
    

去掉这一行的注释,保存，然后重启下服务,执行`systemctl restart rsyslog`

![]()

结合输出最近的Killed信息: `dmesg | egrep -i -B100 'killed process'` (-i  ->
insensitive)

![]()

不出意外oom-kill,来得挺快， 就我那个小内存free才有2.4G，物理文本差不多2g多，载入内存处理，不炸就奇怪。

![]()

### 0x3.2 暴力优化

暴力美学固然美妙，但是也不是我那三流硬件能轻易尝试，想用时间换空间的机会都不给一点呢，自己重构一个程序说实话，没造轮子的必要性，后来想想不如参考国外的老哥写的一篇文章，试试一些top的万级字典来得实在些。

原文用的是5000的字典，但我觉得差点意思，干脆直接上11w的字典，向暴力极致美学看齐。

    
    
    wget https://ghproxy.com/https://ghproxy.com/https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/DNS/subdomains-top1million-110000.txt  
    

tmux开一个窗口

    
    
    tmux  
    

Wfuzz 执行如下命令

    
    
    wfuzz -c -f sub-educn.txt -Z -w subdomains-top1million-110000.txt --sc 200,202,204,301,302,307,403 http://FUZZ.edu.cn  
    

Reference:

https://infinitelogins.com/2020/09/02/bruteforcing-subdomains-wfuzz/

## 0x04 效果分析

1）暴力哲学开头就给我一棒，但是后面我观察暴力优化扫描11w字典的结果，为了形成一些明显的对比，我发现字符2-4的域名长度可以考虑下。

    
    
    crunch 2 4 abcdefghijklmnopqrstuvwxyz -o password_list_2_4.txt  
    sudo wfuzz -c -f sub-2_4.txt -Z -w password_list_2_4.txt --sc 200,202,204,301,302,307,403 http://FUZZ.edu.cn  
    

请求大概来到了40～50w的幅度，应该不会导致oom,  也许有人会问我为什么我会坚持这个呢？原因非常简单，非常崇尚暴力美学，能贯彻到底必须干到底。

![]()

> 通过 top 命令可以观察到，cpu一直维持在100%，内存38%，程序能够很好地运行， **内存反而起到非常决定性的作用** 。

大概经过了漫长一晚上的等待,获得最终的结果在 _346_ 条，在量级比较上面，其实是比较优秀的，平均10w的域名出100条的样子

![]()

2）处理暴力优化扫描的结果，通过 grep 和 awk 可以得到结果，可以获取到结果为89条域名。

![]()

`cat sub-educn.txt|grep -e '^[0-9]'| awk '{gsub("\"", ""); print $NF
".edu.cn"}'`

![]()

简单对数据进行一下去重比较

1）暴力Fuzz -> 346 -> httpx存活判断 -> 332    可以得出接近96%的有效率

![]()

2）暴力优化(高效小字典) -> 89 ->httpx存活判断 -> 81 可以得出91%的有效率

![]()

3）暴力Fuzz地毯式收集的方式相对于高效小字典的方式，额外命中率其实是非常高的。

    
    
    # 对结果排序输出，方便比较  
    cat domains2.txt|MoreFind -d --root|sort > sort_fuzz_nice.txt  
    cat domain.txt |MoreFind -d --root|sort > sort_fuzz_optimize.txt  
      
    # 比较两个文本文件，匹配出相同的行，grep -wf pattern_file input_file  其中 -w 是整词匹配   
    # 最终可以统计出38行相同, 其中 sort_fuzz_nice.txt 总行数为346行, sort_fuzz_optimize.txt为74行  
    cat sort_fuzz_nice.txt|MoreFind -d --root|wc -l  
    cat sort_fuzz_optimize.txt|MoreFind -d --root|wc -l  
    grep -wf sort_fuzz_nice.txt sort_fuzz_optimize.txt|wc -l  
    

  
![]()

虽然这两种方式的数据请求量不一样，但是做个粗糙的统计下也是有一定参考的价值

1）暴力Fuzz的方式减掉38个，额外率为89%。

2）暴力优化高效字典的方式减掉38个，额外率为48.6%

我们观察下，优化的10w小字典多出来的那36个域名到底长什么样子,结果出乎我的意料，一开始我想的是应该不会再出现2-4字符的。

    
    
    grep -wf sort_fuzz_nice.txt sort_fuzz_optimize.txt > diff.txt  
    grep -vwf diff.txt sort_fuzz_optimize.txt

  
![]()

但是经过比较之后发现，全排列爆破的方式却出现了明明字典里面有的子域名却似乎没有扫描出来的情况

![]()

作为一次粗糙的扫描，我也考虑到网络因素，可能不可避免会存在一些误差，但是呢，这个误差会去到什么程度呢？

于是，我重新提取那36个域名作为字典重新fuzz下，并且多次尝试，发现在时间间隔不大的环境，多次测试结果完全一样的。

    
    
    grep -vwf diff.txt sort_fuzz_optimize.txt > diff_only_optimize.txt  
    sed -i 's/\.edu\.cn//g' diff_only_optimize.txt  
    wfuzz -c -f sub-diff_check.txt -Z -w diff_only_optimize.txt --sc 200,202,204,301,302,307,403 http://FUZZ.edu.cn  
    

  
![]()

那么问题出在哪里呢？经过重新查看先前的操作，原因是出在我的 **操作失误** 和固定思维上面:

> 小字典并不是 abc，因为用了MoreFind -d --root 就提取到了 abc
>
> 真正解析的是www.abc.edu.cn 而 abc.edu.cn 是没有做dns解析的。

这样子来看，Wfuzz的性能和准确性是足够的，同时也发现自己存在的一个问题，想当然地用常规运维思维去处理一些事情，或者没有控制变量，控制相同维度，那么就会出现很多诡异的事情。

![]()

## 0x05 本文总结

区别于常规的DNS查询方式，笔者则是尝试通过 DNS -> WebFuzz
的方式将域名爆破流程复杂化，将dns压力和web请求压力1:1的方式传递，特别选用的 edu.cn
域名作为测试，也成功避免了出现泛解析的情况，故能非常有效地统计出有效域名的情况，根据结果我也感受到了暴力美学的神奇的特点--大力出奇迹！

## 0x06 题外话

想法之所以是想法，那么就一定要落地，要不然连理论都学不好的人夸夸其谈也就只能忽悠自己了，有时候做点傻事情，比如这个事情就很傻，为什么不用ksubdomain通过dns查询的方式去爆破，提高效率呢，而采用WebFuzz这种很低效的方式去做呢？从理论来说，我也不知道怎么反驳，但是呢，我觉得""存在即是合理",我必须去创造这类的尝试，当然，我也会安慰自己比如未来会在平衡某些工具扫描效率时给我提供一些参考数据。

## 0x07 后记

提起Wfuzz，FFuf 也较为人所熟知，网上搜索两者的时候，发现竟然还有一篇paper专门介绍了这两个工具的对比  ，简单解读下就是 wfuzz
内存大，速度快， ffuf 还有一段路要走的意思。感兴趣地可以去阅读一下文章:
https://www.doria.fi/handle/10024/181265

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

