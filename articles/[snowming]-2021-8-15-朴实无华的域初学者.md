
# 0x01 定位域控

**<u>通过KDC服务确定域内机器是否为域控</u>**

>Kerberos 协议中主要有三个角色：
1. 访问服务的 Client
2. 提供服务的 Server
3. KDC(Key Distribution Center)密钥分发中心
**KDC 默认安装在域控中**，而 Client 和 Server 为域内的用户或者服务，如 web 应用、数据库服务器和邮件服务器等。Client 是否有权限访问 Server 端的服务由 KDC发放的票据来决定。


自然想到，确定一台机器是域控的方法之一：
**通过 KDC 服务。**


<u>下面这台就是一个域控：</u>


在 CS Beacon Shell 里：

```
shell net start //查看服务
```

![title](https://leanote.com/api/file/getImage?fileId=5e7ef720ab64412ae60063a2)


<u>下面这台就不是域控：</u>


![title](https://leanote.com/api/file/getImage?fileId=5e7ef7dbab64412ae60063bf)

没有 `Kerberos Key Distribution Center` 服务。虽然有个 DNS 服务，但是是 client 客户端不是 server 服务端。

---------------

**<u>通过 DNS Server 确定域控</u>**


AD 很大程度上依赖 DNS 这类解析服务，所以域内内成员机器指向的 DNS 都是域控。


<u>下面这台就是一个域控：</u>



![title](https://leanote.com/api/file/getImage?fileId=5e7efcf2ab644128e90062e4)


DNS 指向自身：本机 IP 地址是192.168.3.142，它的 DNS 也是192.168.3.142。 


<u>下面这台就不是域控：</u>

![title](https://leanote.com/api/file/getImage?fileId=5e7efc75ab644128e90062cc)


本机IP : 100.100.100.104

域控/DNS 服务器 IP：100.100.100.100


---------------

**<u>通过时间服务器确定域控</u>**

域中的默认时间服务器是 DC。

![title](https://leanote.com/api/file/getImage?fileId=5e7f0175ab64412ae6006589)


那么 `192.168.3.142` 就是域控。

<u>一个实例</u>

先通过 `net time` 命令获取域控服务器的 `FQDN`。

![title](https://leanote.com/api/file/getImage?fileId=5e7f0252ab64412ae60065b6)

然后通过 ping 域控服务器的 FQDN 获知域控 IP：

![title](https://leanote.com/api/file/getImage?fileId=5e7f037eab64412ae60065df)


------------------



# 0x02 横向移动


**<u> IPC+计划任务</u>**


进程间通信（IPC，Inter-Process Communication），指至少两个进程或线程间传送数据或者信号的一些技术或方法。

![title](https://leanote.com/api/file/getImage?fileId=5e7f2335ab644128e9006a97)
