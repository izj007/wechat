# 0x01 权限维持

当目标机器重启之后，驻留在 `cmd.exe`、`powershell.exe` 等进程中的 Beacon payload 就会掉，导致我们的 Beacon Shell 掉线。

可以通过 IFEO、启动项、服务等方式进行权限维持，这样机器重启之后 Beacon Shell 还会在。

本文中通过一个 Github 上的 Cobalt Strike 后渗透测试插件 Erebus 以服务的方式进行权限维持操作。

```
https://github.com/DeEpinGh0st/Erebus
```

**<u>前提：</u>**

Beacon Shell 必须是高权限，不然通过 SC 命令加服务的话不会成功。

**<u>第一步：加载 cna 脚本</u>**

`Cobalt Strike` → `Script Manager` → `Load` → Erebus 中的 Main.cna

**<u>第二步：生成 Payload 可执行文件</u>**

`Attacks` → `Packages` → `Windows Executable(S)`


![title](https://leanote.com/api/file/getImage?fileId=5e3b943aab64417c350265da)

保存为 `xiaoxue.exe`。

- `Stage` 的地方填团队服务器上的 reverse_http 监听器

**<u>第三步：上传 payload 可执行文件至目标主机</u>**

通过 Cobalt Strike 的 `File Browser` 进行上传。

![title](https://leanote.com/api/file/getImage?fileId=5e3bb000ab64417a35026732)

![title](https://leanote.com/api/file/getImage?fileId=5e3bb058ab64417a35026736)

- 这里要注意：首先上传的文件路径最好没有空格，不然可能会导致错误；其次最好上传至彩色（不是灰色的）的文件夹路径下。

**<u>第四步：通过插件添加服务</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e3bb0eaab64417a3502673b)

![title](https://leanote.com/api/file/getImage?fileId=5e3bb11eab64417a3502673f)

![title](https://leanote.com/api/file/getImage?fileId=5e3bb183ab64417c35026721)

然后就通过 `SC` 命令把此 `xiaoxue.exe` 添加进了开机启动项，从而初始了一个权限为 `SYSTEM` 的 Beacon。

其效果等同于在 Beacon 控制台中输入：

```
shell sc create "WindowsUpdate2" binpath= "cmd /c start "C:\Windows\system32\xiaoxue.exe""&&sc config "WindowsUpdate2" start= auto&&net start  WindowsUpdate2
```

同样会上线一个 Beacon Shell：

![title](https://leanote.com/api/file/getImage?fileId=5e3bb29eab64417a3502674a)

![title](https://leanote.com/api/file/getImage?fileId=5e3bb2f5ab64417a3502674f)


注意一定不要在普通用户权限下添加服务，否则不会成功：

![title](https://leanote.com/api/file/getImage?fileId=5e3b9ad4ab64417c35026623)

**<u>第五步：重启目标机器测试 Beacon 留存</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e3bb5e5ab64417c35026751)

的确只剩下这两个 Beacon。

# 0x02 在团队服务器之间传递 Beacon Shell

**<u>第一步：准备工作 —— 把 Beacon 转移到更安全的进程上</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e3bc76bab64417c35026903)

当前 Beacon 开在 `powershell.exe` 上。但是此进程比较敏感，开在此进程上不是很安全，所以换一个进程注入。

选择 `jusched.exe`（Java 更新程序），然后注入：

![title](https://leanote.com/api/file/getImage?fileId=5e3bc9a0ab64417c3502693f)

然后我们就会得到一个弹回的开在 `jusched.exe` 上的 Beacon Shell：

![title](https://leanote.com/api/file/getImage?fileId=5e3bcad7ab64417c35026952)

然后把原来那个开在 `powershell.exe` 上的进程 `Exit`、`Remove` 即可。

**<u>第二步：准备工作—— PPID 欺骗和指定临时进程派生新会话</u>**


----------------------------

2020/5/21 更新：

这一部分感觉我没写好，重新写。其实很简单，比如我想获得一个 beacon shell，让 beacon shell 藏在 `aliyundun.exe` 的进程树下，作为其子进程，获得某种程度的隐匿性，那么首先：

查看 `aliyundun.exe` 的 pid：

![title](https://leanote.com/api/file/getImage?fileId=5ec65812ab64416552000a12)

通过 `ppid` 命令设其为父进程。或者直接在 `Process List` 中对此进程右键 `set as ppid`。

然后在原 Beacon Shell （原是相对于派生来说的）中，使用 spawn 命令（`spawn [位数] [listener]`）进行派生。注意在这里的第二个参数 spawn 是监听器的名字。


![title](https://leanote.com/api/file/getImage?fileId=5ec65825ab64416552000a14)

然后就可以获得弹回来的派生的 Beacon shell。其运行进程为：

![title](https://leanote.com/api/file/getImage?fileId=5ec6594aab644167500009b7)

总之就这么简单的。

---------------------------

![title](https://leanote.com/api/file/getImage?fileId=5e3bcd8eab64417c3502697e)

目标是把 `144.*.*.70` 这台团队服务器的 Beacon Shell 传递到 `52.*.*.108` 这台团队服务器上。

> 父进程标识符（PPID）欺骗是相当吸引人的技术，因为它使得恶意应用程序能够在不同的父进程 ID 下派生新的进程。一直以来，此技术被广泛用于隐藏恶意软件，尤其是在需要某种持久性的情况下。
<br>
The Parent Process Identifier (PPID) Spoofing is a quite fascinating technique since it enables malicious applications to spawn new processes under a different parent process ID. It is been used in the wild since ever to hide malware, especially when some kind of persistence is required. Let’s see together how to implement this capability into the Meterpreter agent.
<br>
引自：[Meterpreter+PPID Spoofing-Blending into the Target Environment](https://mp.weixin.qq.com/s/RfZZXqttSWN4A4DmG82UDA)，lsh4ck

要传递的 Beacon Shell 当前运行在 `jusched.exe` 上，此进程除了本身的一个子进程，一般不会有别的子进程。所以我想把子进程开在 `chrome.exe` 进程下，比较不引人注目。使用 `ppid` 命令将 chrome.exe 设为父进程：


![title](https://leanote.com/api/file/getImage?fileId=5e3bd49fab64417c350269f5)

使用 `chrome` 的64位子进程来作为临时进程用于派生会话：

![title](https://leanote.com/api/file/getImage?fileId=5e3bd505ab64417a35026a21)

![title](https://leanote.com/api/file/getImage?fileId=5e3bd5caab64417c35026a05)

> 注：使用 `spawn` 命令来为监听器派生会话，`spawn` 命令接受两个参数，第一个是位数（x86 或 x64），第二个参数是监听器。
默认情况下，`spawn` 命令会在 `rundll32.exe` 中派生会话。但是这样（`rundll32.exe` 定期与 Internet 建立连接这种异常现象）可能会引起管理员注意，所以为了更好的隐蔽性，可以使用更适合的程序如 `Internet Explorer` 来进行会话派生。
<br>
使用 `spawnto` 命令来说明在派生新会话时候使用哪个程序。此命令第一个参数是位数，第二个参数是用于派生会话的程序的完整路径。也就是文中的 `spawnto x64 C:\Program Files (x86)\Google\Chrome\Application\chrome.exe` 这个命令。

**<u>第三步：把会话传递到另一台团队服务器上</u>**

在新的团队服务器 `52.*.*.108` 下新建 reverse_http 监听器：

![title](https://leanote.com/api/file/getImage?fileId=5e3bd863ab64417a35026a54)

在 `144.*.*.70` 这台团队服务器上欲传递的 Beacon 上右键 → `Spawn`，选择刚刚创建的监听器：

![title](https://leanote.com/api/file/getImage?fileId=5e3bda2dab64417c35026a3d)

这个操作等同于 `spawn [监听器名]`：

```
spawn new-team-server
```

然后回到新的团队服务器下，会发现会话已经传递过来了：

![title](https://leanote.com/api/file/getImage?fileId=5e3bdacfab64417c35026a45)

查看 `Process List` 发现此会话进程的确是作为 `chrome.exe` 的子进程运行的，但是将新派生会话到 `chrome.exe` 的子进程中失败了，而是开了一个默认的 `rundll32.exe`。其实这里一般是用 `iexplore.exe` 的 x86 子进程作为派生会话的临时进程（使用 ` spawn x86 c:\program files (x86)\internet explorer\iexplore.exe` 命令）。之所以使用 x86 子进程，是为了跟 x64 位父进程区分开来。

但是本文中我使用了 `spawnto x64 C:\Program Files (x86)\Google\Chrome\Application\chrome.exe` 这个命令，所以就没有跟 `chrome.exe` 父进程区分开来。因而其实使用的是 `chrome.exe` 父进程派生会话，而没有使用其子进程派生会话，所以最终的新会话开在了 `spawnto` 命令默认使用的 `rundll32.exe` 程序上。

![title](https://leanote.com/api/file/getImage?fileId=5e3bdbdaab64417c35026a53)
![title](https://leanote.com/api/file/getImage?fileId=5e3bdc16ab64417a35026a97)

**<u>总结：</u>**

将一台团队服务器上的 Beacon 传递到另一台团队服务器，最精简的步骤为：

1. `New Connection` 连接到新的团队服务器上。
2. 在新的团队服务器上开监听自身的 `reverse_http` 监听器。
3. 在旧的团队服务器上，[Beacon] → `spawn` → 选择第二步中开的监听器。
4. 会话传递成功，可在新的团队服务器中查看。

其中，可以在旧的团队服务器上通过 `ppid` 命令指定会话的父进程，也可以通过 `spawnto` 命令指定用于派生欲传递会话的进程（默认是 rundll32.exe，推荐 `c:\program files (x86)\internet explorer\iexplore.exe`）。



------

2020/5/21 更新：

其实完全可以先把会话 spawn 传递到另外的团队服务器，再作 ppid 欺骗或者是注入到其他进程，提高被动隐匿性。


--------------------

参考文档：

[1] [Youtube 视频 - Session Prepping and Session Passing (Cobalt Strike 4.0)](https://www.youtube.com/watch?v=4xnBn5ZVkKE)，Youtube，Raphael Mudge
[2] [Meterpreter+PPID Spoofing-Blending into the Target Environment](https://mp.weixin.qq.com/s/RfZZXqttSWN4A4DmG82UDA)，「靶机狂魔」公众号，lsh4ck，2020年2月10日