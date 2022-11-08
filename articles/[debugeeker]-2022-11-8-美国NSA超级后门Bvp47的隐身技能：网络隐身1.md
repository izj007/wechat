#  美国NSA超级后门Bvp47的隐身技能：网络隐身1

原创 debugeeker [ debugeeker ](javascript:void\(0\);)

**debugeeker** ![]()

微信号 gh_ed0f7fa0bfe0

功能介绍 安全攻防技术 大厂经历分享 程序员职业规划

____

___发表于_

收录于合集

#Linux内核 4 个

#后门 4 个

#美国 3 个

#恶意软件 18 个

  

> 本文是基于Bvp47技术报告(PDF)和`Linux`内核写的。
>
> 这篇主要是讲述`Bvp47`的网络信息在主机侧的隐身，下篇会讲述到它在整个网络的隐身技能

如何在`Linux`系统隐藏网络信息呢？

平时，我们查看网络信息，一般是使用`netstat`命令。那它是从哪里获取这些网络信息？

使用`strace
netstat`就可以知道它是从`/proc/net`和`/proc/<pid>/fd`两个目录获取信息。比如要知道1号进程监听了哪些`IPV4`的`tcp
socket`,就是要从`/proc/1/fd`下使用`readlink`获取是`socket`类型`fd`的`inode`，再通过这个`inode`在`/proc/net/tcp`里找，如果该`inode`对应记录的`st`列为`0A`，那么该`fd`就是监听`socket`，而它的源宿地址分别在`local_address`和`rem_address`这两列。

按照前面两篇的讲述，隐藏网络信息，也就两种方法：

  1. 用户态劫持`readlink`,`read`之类的系统调用
  2. 内核态劫持网络相关的内核函数

`Bvp47`挂钩了内核里这些网络相关的函数：

  * tcp4_seq_show - 显示TCP监听和连接信息
  * udp4_seq_show - 显示udp监听和连接信息
  * packet_seq_show - 显示packet socket信息
  * unix_seq_show - 显示unix socket信息
  * listening_get_next - 枚举监听信息
  * established_get_next - 枚举连接信息
  * sys_bind - 绑定地址
  * unix_accept - unix socket接收新连接
  * sys_read - 读取句柄
  * sys_write - 写句柄
  * sys_sendto/sys_sendmsg - 向远程发送信息
  * sys_recvfrom/sys_recvmsg - 接收消息
  * sock_init_data - 初始化socket并加入到链表
  * tcp_time_wait - tcp连接关闭时的回收
  * sys_connect - 连接远程
  * sys_dup/sys_dup2 - 复制句柄
  * get_raw_socket - 获取raw socket链表

从上面`netstat`的例子，可以知道`ipv4`的信息是从`/proc/net/tcp`里获得的，而这个虚拟文件是由哪个函数来写的呢？

`/proc/net/tcp`文件的字段是这样：

    
    
     sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode        
    

通过在内核代码里搜索，可以看到是`net/ipv4/tcp_ipv4.c`的`tcp4_seq_show`函数。所以`Bvp47`挂钩`tcp4_seq_show`过滤掉它的连接和监听端口。

同理，这也是为什么`Bvp47`挂钩`udp4_seq_show`,`unix_seq_show`,`packet_seq_show`的原因。

如果仔细阅读内核代码，会发现`/proc/net/tcp`的信息获取流程中包含着`listening_get_next`和`established_get_next`。

其实，从这几个文件读取的信息，也是指纹检测。现在的`HIDS`一般还会用行为检测。

在没有设置`SO_REUSEADDR`标志情况下，端口的监听是独占式的，意味着，如果一个端口被某个进程监听了，其它进程就无法再监听这个端口。

对任意一个端口调用一次`bind`操作，如果它返回错误码是`EADDRINUSE`,就说明这个端口被监听，假设在`/proc/net/tcp`又找不到它的信息，说明这个端口是被隐藏起来了。

由于端口的范围也是固定的，就是0-65535，那么只需要对端口进行枚举来`bind`一把，再对比`netstat`的结果，就知道有没有端口隐藏了。

所以，`Bvp47`要挂钩`sys_bind`从而隐藏自己的端口监听，从而实现即使端口被重复监听，也不会显示出来。

根据盘古实验室的《Bvp47技术报告》，黑客机器`Bvp47`会向DMZ的`Bvp47`发端口敲门包，DMZ的`Bvp47`就会向它连接。在现实网络中，往往是很深的，除了DMZ区，还有业务区，DB区之类。这些区域的机器往往不会和外网直接连接的，所以，恶意软件需要做一些代理节点，像黑客机器的`Bvp47`也会入侵投放到这些区域，成为代理节点。

那么，问题来了，假设`Bvp47`代理节点监听了某端口，由于它挂钩了`sys_bind`来隐藏了，允许端口重复监听。那么，可以该代理节点里进行对端口枚举监听，毕竟范围是从0-65535，用`nc`就可以实现了，把所有端口全部监听，肯定会覆盖到`Bvp47`的端口。如果其它主机的`Bvp47`向代理节点发起连接，那么，该代理节点的`nc`就会接收`Bvp47`的连接请求和数据传输。

所以，`Bvp47`需要对`sys_accept`,`unix_accept`,
`sys_sendto/sys_sendmsg`,`sys_recvfrom/sys_recvmsg`,`sys_read`,`sys_write`这些都要挂钩来隐藏信息。由于挂钩这些函数，会涉及到`socket`在内核本身数据结构，所以`sock_init_data`,`sys_connect`,
`tcp_time_wait`都会需要挂钩。

由于`Bvp47`
不可能实现所有功能，它往往会调用系统一些工具，那么当它调用时，就会派生子进程，而子进程是会继承它的句柄，如果这些句柄的`inode`是指向不存在的网络信息，这就暴露了它的存在。由于句柄继承有一些是通过`sys_dup/sys_dup2`来进行，所以它需要对这些函数挂钩来过滤掉它的信息。

`get_raw_socket`这个函数挂钩，我真的没办法看不出它的用处。它是由`ioctl`来调用，用于虚拟机方面，是宿主机`vhost_net`后端这一块范围。希望有大佬能够指点一下。谢谢！

  

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

