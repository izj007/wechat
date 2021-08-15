# 0x01 前言

事情的起因是看到了这样一段话:

![title](https://leanote.com/api/file/getImage?fileId=5df5d130ab644137c5001e1b)

众所周知，`RID` 是 `SID` 的最后一位。这里面说到了用户有 RID，那么也就是说用户有 `SID`，这个是自然的，继续往下看。

然后又说 `/groups` 参数指定用户所属组的 `RID`，这也就是说组也有 `SID`。也可以理解吧，因为 `SID` 就是一个安全身份标识符，自然可以标识很多东西。

那么看下我的域内的一台 windows server 2008 r2 上面的 `SID`：

![title](https://leanote.com/api/file/getImage?fileId=5df5d767ab644139ca001f04)

嗯，看上去没毛病。但是跟这里描述的不一样啊：

![title](https://leanote.com/api/file/getImage?fileId=5df5d79cab644139ca001f0e)

既然说了组，那么就大胆猜测一个命令吧：

```
wmic group get name,sid
```

然后就开始变得有意思起来了，查询到的 `SID` 有长有短：

![title](https://leanote.com/api/file/getImage?fileId=5df5d828ab644139ca001f2b)

那么这些 SID 为什么有的长有的短呢？


# 0x02 探索

经查资料，发现短的 `SID` 是微软内置（built-in）的一些：

[Well-known security identifiers in Windows operating systems](https://support.microsoft.com/en-us/help/243330/well-known-security-identifiers-in-windows-operating-systems)

如果仅仅理解为，短的是内置，长的不是内置，似乎太过肤浅。

那么来进行一点点的探索，观察发现：**短的 `SID` 始终固定，长的 `SID` 具体内容会变化**。

跟微软文档进行对比发现：短的 `SID` 是根据操作系统始终固定的：

![title](https://leanote.com/api/file/getImage?fileId=5df5dcb9ab644137c5002052)


那这些短的 `SID` 对应的对象是什么呢?

其实是**「本地组」**。


![title](https://leanote.com/api/file/getImage?fileId=5df5de9eab644137c50020a7)

其实也很好理解。组是 windows 自带的机制，每一台机器都必定会有本地组，但是不一定会有域组。所以本地组的 `SID` 是固定的。


那那些长的 `SID` 对应的对象又是什么呢?

先看同一个域内的两台机器：


![title](https://leanote.com/api/file/getImage?fileId=5df5dfafab644139ca00207f)

![title](https://leanote.com/api/file/getImage?fileId=5df5e1d0ab644137c500212f)


发现红框和绿框内的有区别，但是黄框里面的 SID 是一样的。其实原因很明显：红框和绿框内的是本机用户对应的 SID，黄框部分是域用户对应的 `SID`。

![title](https://leanote.com/api/file/getImage?fileId=5df5e383ab644139ca00213e)

再比较下另一个域的机器上查看 `SID` 的情况，域用户（`krbtgt`）的 SID 和前一个域里面的也不一样：

![title](https://leanote.com/api/file/getImage?fileId=5df5db74ab644139ca001fbb)

域组的 `SID` 呢，可以想见，必定是长的 `SID`，因为域不是计算机自带的，所以必定不是内置的。

# 0x03 总结

![title](https://leanote.com/api/file/getImage?fileId=5df5e43fab644139ca002165)

在 Windows 里面，不仅用户有本地和域的概念，组也是。因为也就是用户加入了本地组或域组，才有了本地用户和域用户之分。

- 查看本地组：`net localgroup`
- 查看域组：`net group`，注意此条命令只能在 DC 上执行：

[非域控机器执行]
![title](https://leanote.com/api/file/getImage?fileId=5df5e50cab644137c50021b5)

[DC执行]
![title](https://leanote.com/api/file/getImage?fileId=5df5e54eab644137c50021c1)



----------

参考链接：
1. [Windows SID 理解](https://www.cnblogs.com/jackydalong/p/3262241.html)，博客园，笑剑钝，2013年8月16日  

