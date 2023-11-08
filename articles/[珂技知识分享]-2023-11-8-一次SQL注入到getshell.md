#  一次SQL注入到getshell

原创 珂字辈  [ 珂技知识分享 ](javascript:void\(0\);)

**珂技知识分享** ![]()

微信号 kezibei001

功能介绍 分享自己的安全技术，渗透实例，IT知识，拒绝转载，坚持原创。

____

___发表于_

收录于合集

**1.     发现SQL注入**  
给一个单引号，很轻易的发现对方存在SQL注入，且爆出cms版本号和系统绝对路径。  
/index.php?s=/Home/Article/detail/id/1'.html  
OneThink 1.1.141212  
继续注入的时候，发现存在云锁，翻找两年前的waf绕过笔记，居然没有云锁。

![]()

不过这种老的基于正则的本地waf，和D盾/安全狗的弱点是差不多的。  

![]()

但是注入绕过时必须要用到/**/注释，而下面这种传参形式使用/**/注释会混淆参数，因此不可用。  
/index.php?s=/Home/Article/detail/id/1'.html  
这个时候熟悉thinkphp3.2.3的人就会熟练的将其转化为另一种支持参数传/的url模式。  
/Home/Article/detail?id=1'  
这样就能绕云锁进行SQL注入了。绕过的payload，基本都是基于利用%0A产生假注释，比如。  
\-- /*%0Aselect/**/user()  
有了SQL注入，似乎getshell近在眼前了？远没有那么简单。  
  
虽然OneThink的后台地址是固定的  
/index.php?s=/Admin/Public/login.html  
看数据库也没有什么salt。  

![]()

但实际密码校验是这样的。  
/Application/User/Model/UcenterMemberModel.class.php  

![]()

而UC_AUTH_KEY哪来的呢？首次安装随机生成的。  
/Application/User/Conf/config.php  

![]()

也就是说，即使你拥有数据库的权限，可以任意修改admin密码，只要你不能读config.php，也是无法进后台的。  
  
 **2.     历史漏洞**  
  
OneThink的历史漏洞主要分为三部分。  
一部分是经典thinkphp的exp注入，发生在登录口，因此可以直接使用exp+联合注入获取登录态。  
https://xz.aliyun.com/t/8081  
但新版已经修复了这个问题，会在exp后面加个空格，然后提示表达式错误。  

![]()

一部分是经典的thinkphp的缓存getshell  
https://forum.90sec.org/forum.php?mod=viewthread&tid=9308&page=1  
这个漏洞分两部分，一部分是将恶意代码写入Runtime目录下的缓存php文件并逃逸注释，另一部分就是预测缓存文件名，也就是md5。  
而后者在新版OneThink上无法实现了，因为缓存文件名规则也加了salt。  
/ThinkPHP/Library/Think/Cache/Driver/File.class.php  

![]()

DATA_CACHE_KEY是什么呢？和UC_AUTH_KEY是同一个值，还是安装程序时随机生成的。  
/Application/Common/Conf/config.php  

![]()

最后一部分漏洞则是OneThink自带的一个插件漏洞。  
https://www.t00ls.com/viewthread.php?tid=62565  
https://www.t00ls.com/viewthread.php?tid=63336  
这个漏洞倒是最新版都一直存在的。  

![]()

不过可惜的是，目标已经删除了这个插件。  
  
除此之外，如同大部分支持插件的cms一样，OneThink后台自定义插件的功能，也可以直接getshell。  

![]()

  
  
 **3.     堆叠注入getshell**  
如果纯粹挖0day的话，应该还是挺困难的，但我们运气好的一点在于以及拥有了一个SQL注入，而且经过测试，这个SQL注入是支持堆叠的，也就是说我们可以修改数据库的数据了。  
虽然非root无法用来读文件，也因为config.php储存salt的原因无法用来登录后台。  
那么我们的目标就简单了很多，仅仅只需要找一个任意文件读取出来，就可以读salt，改管理员密码，进后台自定义插件getshell，或者缓存getshell。  
  
然而在搜索历史漏洞的过程中，我发现了一个没有详情的文件包含漏洞。  
https://blog.csdn.net/m0_46699477/article/details/130009289  
显然我没有红队版goby，但它的漏洞点很清晰。  
/index.php/Home/Article/index?category=1  
很容易猜测出来，这里存在和exp注入登录后台一样的问题，用union select篡改表里的一些字段，达到文件包含的目的。  
新版虽然修复了exp注入，但目标环境刚好存在堆叠注入，完美代替exp+union select。  
代码非常简单，只需要控制category['template_index']即可文件包含。  

![]()

手动修改template_index试一试。  

![]()

![]()

得到的报错非常完美，但似乎被加上了html后缀，有能够上传html文件的地方吗？前端代码中发现默认引用了多个文件编辑器。其中kindeditor虽然默认支持html，但没有php代码，只有ueditor可用，它并不支持上传html文件。  

![]()

那么可以上传zip文件，用zip://协议消除最后的.html，同理也可以phar触发反序列化。  
但是根据代码来看，相对路径无法过is_file()判断，最终被拼接进TMPL_TEMPLATE_SUFFIX。如果使用绝对路径，就会直接return
$template;  
/ThinkPHP/Library/Think/View.class.php  

![]()

![]()

那么整个漏洞链条就清晰了。

  
thinkphp3报错获取绝对路径  
ueditor上传非php后缀的webshell  
堆叠注入绕云锁修改onethink_category. template_index  
/index.php/Home/Article/index?category=1文件包含getshell  
  
 **4.     其他设想**  
  
在最开始时，我以为这里仅仅只是一个is_file的判断，并没有指望可以文件包含。因此我最开始的设想是用phar://触发反序列化，由于thinkphp3+php5有着唯一一条连接mysql的反序列化链，可以用它来连接恶意mysql，达到文件读取的目的。随后读config.php获取UC_AUTH_KEY，然后再进后台或者缓存getshell。也就是。

  
thinkphp3报错获取绝对路径  
ueditor上传zip后缀的phar文件  
堆叠注入绕云锁修改onethink_category. template_index  
/index.php/Home/Article/index?category=1触发反序列化  
反序列化连接mysql  
fake mysql读取config.php  
UC_AUTH_KEY+堆叠注入修改管理员密码  
后台插件getshell

  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

