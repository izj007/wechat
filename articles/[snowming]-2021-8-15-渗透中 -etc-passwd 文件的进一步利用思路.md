# 前言
任意文件读取或是 ssrf 等一些漏洞，我们漏洞证明的方式就是读到了 `/etc/passwd`，但是然后呢？这个文件究竟有什么用？

基础知识请参考：

[1] [Linux /etc/passwd 内容解释](http://c.biancheng.net/view/839.html)
[2] [Linux /etc/shadow 影子文件内容解析](http://c.biancheng.net/view/840.html)

# 0x01 查看 /home/用户名/.bash_history 文件

通过 `/etc/passwd` 获取了用户名之后，可以去访问特定的 `/home/用户名/.bash_history` 文件，看看 history 里面有没有泄露密码。如果里面有一些运维留下的密码，或许可以内网通用口令一把梭。

参考实例： [中国电信某业务可文件遍历(泄漏数据库配置，外网redis可连接)](https://wooyun.x10sec.org/static/bugs/wooyun-2015-091444.html)

![title](https://leanote.com/api/file/getImage?fileId=5dd3cdbfab644109b0000b64)

# 0x02 明文 root 密码提权

设想一个场景：比如 tomcat 是用 root 跑的，然后你有个 jsp 的 webshell。然后你要做的事情就是登陆 root 的 ssh。

在这种场景下，就可能需要明文 root 密码提权。

将 /etc/passwd 的 root 密码 `x` 替换为我们自己的 hash，如替换为自己 linux 里的 hash，可修改目标的 root 密码。

![title](https://leanote.com/api/file/getImage?fileId=5dd3df88ab64410bb1000cb2)

这样修改后，我就把 root 的密码改为了与 /etc/shadow 不同的新密码。/etc/shadow 里面的密码不再可用，需要用新密码登录。

那么如果读到的 /etc/passwd 已经被篡改，就可以直接对此密码密文尝试解密登录。


# 0x03 信息收集、密码复用


从 /etc/passwd 本身来看：

- 用户名：用户名本身是有信息量的。看了 passwd 大概知道开了哪些服务。比如 `mail`、`redis`、`MySQL`，可以推断这个服务器的在目标网络中的作用。对其他的普通用户名进行收集，加入自定义字典。用户名本身可能是一些数据库、后台的 web 账号或密码。
- 默认Shell：这个字段获取哪些账号可以登录、哪些账号不能登录的信息。可以用一些具有特殊含义的普通账号（如`hw`、`xunjian`）进行爆破，如果弱密码爆破成功，获得的账号密码很可能是可以通杀的。
- 主目录：这个字段我们可以获取用户路径，进而猜测一些 web 路径。


总结一下：

主要是进行以下信息收集：

- 用户名
    - 自定义字典
    - 服务器功能判断
    - 尝试 SSH 爆破
- web 路径



# 0x04 结合 redis getshell 等需要知道用户名的场景使用

Redis 未授权访问，可以通过写 SSH 密钥、直接写 webshell、写计划任务这三种方法来获取 system shell。

如果 Redis 不是起在 root 用户下，那么不管是写 SSH 密钥还是写 crontab 都需要知道用户名，`/etc/passwd` 可以看哪些账号能登录，哪些不能，所以可以从中获取可以登录的用户名结合起来使用。


# Todo：

 - 非 root 用户是否能写秘钥？这个我没有试过，等待测试。
 - 【好友评论】解密成功和有直接改文件得权限，这个概率有点低......撞密码可还行，不过尽量需要拖回去撞，规避服务器端口和登陆安全策略，也能减少日志。

 
-----------------------




**参考链接：**

1. [Linux /etc/passwd内容解释（超详细）](http://c.biancheng.net/view/839.html)，C 语言中文网
2. [Linux /etc/shadow（影子文件）内容解析（超详细）](http://c.biancheng.net/view/840.html)，C 语言中文网
3. [Linux 提权的各种姿势总结](https://mp.weixin.qq.com/s/uk0qSfGA4yaj7ioQYmln-g)，信安之路，VoltCary，2019年11月10日
4. [中国电信某业务可文件遍历(泄漏数据库配置，外网redis可连接)](https://wooyun.x10sec.org/static/bugs/wooyun-2015-091444.html)，wooyun，lijiejie，2015年1月15日