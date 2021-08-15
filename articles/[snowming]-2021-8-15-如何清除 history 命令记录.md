# 0x01 前言

事情当然是从痛失权限开始说起。

为了以后不要再次重蹈覆辙，尽量避免被管理员发现在目标主机上操作过的痕迹，我在本机（Ubuntu）进行了多次实验。我的需求是：删除我在目标机器上的所有执行过的命令，但是不要影响到这台主机上之前的所有命令记录（为了不被管理员发现）。很自然的思路是打开 `.bash_history` 进行编辑，但是为什么我发现：

1. 我打开这个对话之后，所有执行过的命令，在 `.bash_history` 文件上都无法找到，但是在 `history` 的执行结果里都可以找到。
2. 当我打开我的 `.bash_history` 这个文件和通过 `history` 命令查看我之前执行过的命令，我发现这两处的内容不一样。目测 `.bash_history` 文件里有更少的命令，而 `history` 命令的执行结果包含更多的记录。 

这是为什么呢？

我怎么样才能完全的清除自己执行过的命令记录从而不被管理员发现呢？


# 0x02 原理

先抛出一个问题：当我们打开一个 Bash shell 时，发生了什么？

经我实验发现：

1. 当我们打开 Bash shell 时，它将读取 `.bash_history` 文件的内容并将 load 到 `history` 的记录列表中。
2. 当我们键入命令， bash 将使这些命令逐条追加到 `history` 的记录列表中。
3. 再次关闭 Bash shell 时，Bash 会将此次会话中键入的条目附加到 `.bash_history` 文件中，从而将 `history` 列表保存到磁盘。注意这里之前从 `.bash_history` 中载入到 `history` 列表的命令，不会再被重复保存到 `.bash_history` 文件中。

总结一下：

- 在此次会话中键入的命令，会被写入 history 列表，相当于暂存在内存中（所以不同的用户打开 bash shell 之后，自己键入的命令部分是不同的，互相不可见）；
- 在此次会话结束后，bash 会将内存中暂存的命令存入 `.bash_history` 文件中，也就是保存到磁盘。
- 在特定的Shell会话结束之前，历史文件将永远不会更新，因此我们在当前终端窗口中所做的所有操作都不会持久保存到磁盘上。它仅驻留在内存中的历史记录列表中。



但是实际上，问题远没有这么简单。这个问题在于，影响 `.bash_history` 的写入的，除了 history 列表里面的内容，还有一些其他的环境变量：

- `HISTFILE`
- `HISTFILESIZE`
- `HISTSIZE`
- `HISTIGNORE`
- `HISTCONTROL`
- ...




``` shell
export HISTFILE=~/.bash_history
```
- `HISTFILE` 变量代表历史记录所在的文件。默认是 `~/.bash_history`。不想保存 history 信息时，可以 `unset` 这个变量。

```shell
export HISTFILESIZE=1000
```

- 这个是针对历史纪录文件(`.bash_history`)的设置，设置了历史记录文件中包含的最大行数。为该变量分配值后，历史记录文件将在必要时通过删除最旧的条目而被截断，以包含不超过该行数的行。默认值为 500。在退出交互式 shell 时，历史文件在写入后也将被截断为该大小。
- 注意：默认情况下，历史记录列表最多被截断为 1000 个条目，历史记录文件最多为 2000 个条目。

```shell
export HISTSIZE=500
```
- `HISTSIZE`，设置一个 shell session 可以存多少条 history 记录。默认情况，它设置了一个非零值。

```shell
export HISTIGNORE=export HISTIGNORE="&:[ ]*:exit"
```
- `HISTIGNORE`，代表在保存历史时忽略某些命令。例如`export HISTIGNORE="&:[ ]*:exit"`，会忽略三种类型的历史记录；分别是： 
    - &，和上一条历史记录相同的记录；
    - [ ]*，以空格开头的历史记录；
    - exit，忽略exit命令。
    - Pattern之间用`:`分隔。
- 例如，如果你不想在命令历史记录中看到 ` history` 程序，则可以执行以下操作：

``` shell
$ export HISTIGNORE="history"
$ history
...
$ history | tail -1
	581  export HISTIGNORE="history"
	582  history | tail -1
```
- 你可以在此处看到仅history在其上面的行未添加到列表中，而 `history | tail -1` 则可以看到。 也就是说仅过滤了 HISTIGNORE 列表中的具有完全匹配的行。如果你不希望以历史记录**开头**的任何行包含在命令历史记录中，则可以将 `HISTIGNORE` 变量修改为（`*` 是以xx开头的意思）：

``` shell
$ export HISTIGNORE="history*"
```


- `HISTCONTROL`，和 `HISTIGNORE` 类似，也是用来忽略某些历史记录。
    - `HISTCONTROL=ignoredups`：忽略连续重复的命令
    - `HISTCONTROL=ignorespace`：忽略以空白字符开头的命令
    - `HISTCONTROL=ignoreboth`：同时忽略以上两种
    - `HISTCONTROL=erasedups`：忽略所有历史命令中的重复命令


注意：以上所有在命令行中输入的 `export` 命令都只对单次 session 生效，如果想要让配置永久生效，可以把 `export` 命令写到 `.bash_profile` 或者全局配置文件 `/etc/profile` 里。




# 0x03 想法

那如果我们想擦除输入的命令，该从哪些地方下手呢？

> 当我们键入命令， bash 将使这些命令逐条追加到 `history` 的记录列表中。

- 注意这里：所有的输入都添加吗？哪些命令不加？
- 重复的指令？带空格的指令？可以通过对单次会话设置环境变量做到这一点。

> 再次关闭 Bash shell 时，Bash 会将此次会话中键入的条目附加到 `.bash_history` 文件中。

- 注意这里：如果提前中断 bash，这样 bash 就无法完成这一步保存任务了？

> 如果一不小心 bash shell 掉线了，相当于输入了 `exit`，那么 history 就会已经被写入了历史记录文件。

- 注意这里：如何擦除上次的记录——编辑或覆盖 `.bash_history`。

# 0x04 实践

## 隐匿思路1 —— 擦除
- 清空单次 session shell 中的用户个人输入（即 history 列表），不会影响到历史文件。
- 其他用户不会看到你此次 session shell 中的记录，所以清空是完全正当合理的。

**方法一：擦除全部 history**

在完成所有工作之后执行：
``` bash
history -cw
```

注：

- history -c   # 清除当前 shell 的历史记录，不会影响 HISTFILE
- history -w   # 用当前 shell 的历史来覆盖 HISTFILE（`.bash_history`），相当于写入。因为当前 shell 的历史中 load 了之前的历史文件，但是清除了当前 shell 的 history 记录，所以去覆盖的话就会保留之前的完整历史文件。

**方法二：擦除全部 history**


- 要从历史记录列表中清除特定条目，可以先运行 `history` 以显示完整列表，并找出有问题的条目的索引号。然后可以使用以下命令删除它：
``` shell
history -d NUMBER
```

或者，通过 grep 命令来删除：
``` shell
history | grep "part of command you want to remove"
```
上面的命令会输出历史记录中匹配的命令，每一条前面会有个数字。
![title](https://leanote.com/api/file/getImage?fileId=5de701eeab644153da004ea7)


一旦你找到你想删除的命令，执行下面的命令，从历史记录中删除那个指定的项：
``` shell
[space]history -d NUMBER
```

注：不加空格的话这条删除记录本身会被记录！


# 隐匿思路2 —— 不让记录


**方法一: 在命令前插入空格**

``` shell
export HISTCONTROL=ignorespace
export HISTCONTROL=ignoreboth
```
通过以上任意一种设置之后（但是在较多情况下，这两行设置是默认值），在命令前面插入空格，这条命令会被 shell 忽略，也就意味着它不会出现在历史记录中。


测试：

Kali 系统默认忽略以空格开头的命令和重复命令：

![title](https://leanote.com/api/file/getImage?fileId=5de88acbab644111cc001b23)

Ubuntu 系统默认忽略重复的命令（dups 指 duplicates）和以空格开头的命令：


![title](https://leanote.com/api/file/getImage?fileId=5de88baaab64410fc9001a77)


Amazon Linux 2 AMI 默认忽略重复的命令：

![title](https://leanote.com/api/file/getImage?fileId=5de88b6bab644111cc001b44)

**方法二：禁用当前会话的所有历史记录**

在开始命令行工作前简单地清除环境变量 HISTSIZE 的值即可。

``` shell
export HISTSIZE=0
```
通过此设置， `history` 列表将不能保存命令（可保存命令的条数为0）。默认情况，`HISTSIZE` 设置了一个非零值。

**方法三：丢弃当前会话的所有历史记录**

在开始命令行工作前，做如下设置：

``` shell
export HISTFILE=/dev/null
```

或

``` shell
unset HISTFILE
```

- `HISTFILE` 变量代表历史记录所在的文件。默认是 `~/.bash_history`。不想保存 history 信息时，可以 `unset` 这个变量。
- /dev/null（或称空设备）在类Unix系统中是一个特殊的设备文件，它丢弃一切写入其中的数据（但报告写入操作成功）。`command > /dev/null` 的作用是将是 command 命令的标准输出丢弃。通过指定 `HISTFILE` 为此文件，在 session 关闭时，会将此次会话中的历史纪录从内存转储到此文件，即为进行了丢弃。

**方法四：在单次会话中，禁用某一段命令记录**

有时候需求是：

- 在单次 session 中，只禁用一段时间的命令记录
- 这样上面的方法都不起作用了，因为它们会清除整个 session 中的会话历史记录
- 如何做到临时的禁用呢？

对于这样的需求，在需要禁用的部分前执行下述命令：

``` shell
[space]set +o history
```
备注：[space] 表示空格。并且由于空格的缘故，该命令本身也不会被记录。

上面的命令会临时禁用历史功能，这意味着在这命令之后你执行的所有操作都不会记录到历史中，然而这个命令之前的所有东西都会原样记录在历史列表中。

要重新开启历史功能，执行下面的命令：
``` shell
[Space]set -o history
```
它将环境恢复原状，也就是你完成了你的工作。执行上述命令之后的命令都会出现在历史中。

原理非常简单，就是通过 set 的 `-o` 选项来启用或禁用 `history` 功能。


**方法五：kill bash 进程**

通过杀死本 bash 进程，提前中断，这样 history 还未被写入历史文件中。

原理是：

默认情况下， bash 只在退出的时候更新命令历史， 而且这个「更新」是用新版直接覆盖旧版。 

这个前提下，如果你的 bash **异常退出**了——比如网络故障、 防火墙更改、或者它的进程被杀掉了等，那么会话中所有的历史记录都会丢失。


具体做法为：

在完成了所有操作之后，

``` shell
kill -9 $$
```

更进一步：

如果一个用户登录多次， 这种覆盖的机制会使得只有最后一个退出的 bash 能保存它的历史记录(一个登录的用户打开多个终端模拟器， 或者使用 screen/tmux 等工具启动多个 bash 等也在此列）。

所以或许可以使用 screen/tmux 启动多个 bash，只在前几个 bash 输入操作指令，然后先退出这些 bash，理论上也可以做到隐匿操作记录。但是我没有去尝试。

参考资料：[[译] 如何防止丢失任何 bash 历史命令?](https://felixc.at/2013/09/how-to-avoid-losing-any-history-lines/)

# 隐匿思路3 —— 擦屁股

如上两种思路中，我更倾向于在此 session 开始时候就进行的环境变量设置，因为我们的会话随时会断，这样有些在操作结束后需要执行的命令就没机会输入了。

那如果会话就那么断了，history 已经被写入历史文件了，我们如何去补救呢？

可以在会话中把 `.bash_history` 文件下载下来（因为会话中的 history 不会影响到 `.bash_history`，所以理论上，会话断掉之前随时都可以），然后编辑一下，最后准备结束的使用用 xftp 覆盖。
如果会话断掉了，就用 sftp 覆盖。

# 0x05 总结

**开始操作前：**

```
[space]export HISTCONTROL=ignorespace #默认设置
[space]export HISTCONTROL=ignoreboth #默认设置
```

```
[space]export HISTSIZE=0
```

```
[space]export HISTFILE=/dev/null
```

```
[space]unset HISTFILE
```

```
[space]set +o history
```
----------------------

**注意：**

请注意所有跟环境变量(`HISTCONTROL`，`HISTSIZE`，`HISTFILE`)有关的设置。

如果这几个参数在 `/ect/profile` 环境配置文件里做了 `readonly` 的设置，那么就无法在单次 session 的 `bash` 中通过 `export` 命令修改此环境变量的设置（csh、zsh 终端未尝试）。

在 `/etc/profile` 里面把这几个参数设为 `readonly` 的具体做法为：
``` shell
readonly PROMPT_COMMAND  HISTSIZE HISTCONTROL HISTSIZE
```

把这行加到 `/etc/profile` 文件里去，然后每次登陆终端，都会加载一次，于是终端上就执行不了 `export HISTSIZE=/dev/null` 这样的操作 ，会提示 `readonly` 权限。

一些运维人员会在生产环境里做这样的设置，对于那些做了 readonly 配置命令记录的机器，此类在 session 里面通过 `export` 临时修改环境变量的方法将无法使用。

不过对于做了这种 readonly 设置的主机，输入 `export HISTSIZE=/dev/null` 这种命令会报错，说 `readonly variable`。当我们看到此类报错，就知道是做了设置了，就转向下面的其他方法就好。为了避免此次尝试被记录，我们最好在这条尝试的 `export` 命令前也打上空格。

----------------------------



**执行操作时：**

- 在每条命令前加一个空格

这个方法的好处是：在较多情况下，命令行中默认忽略以空格开头的指令。


**操作完成后，关闭 session shell 前：**

```
[space]history -cw
```

```
[space]kill -9 $$
```

或者可以组合拳搞定所有设置（注意这里的所有设置都是仅针对单次 shell，并不是永久生效）：
``` shell
[space]unset HISTORY HISTFILE HISTSAVE HISTZONE HISTORY HISTLOG; export HISTFILE=/dev/null; export HISTSIZE=0; export HISTFILESIZE=0
```

----------------------------------------

致谢：

感谢 `Web Security&Penetration Testing` 群和`从心开始`群里的各位大佬指点。
感谢`@星火燎原`大佬关于环境变量 readonly 情况下无法更改的提点。


参考链接：

1. [the history section of the manual](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Using-History-Interactively) ，Bash 参考手册
2. [[译] 如何防止丢失任何 bash 历史命令?](https://felixc.at/2013/09/how-to-avoid-losing-any-history-lines/)，Felix's Blog，Felix，2013年9月5日
3. [如何隐藏你的 Linux 的命令行历史](https://www.cnblogs.com/276815076/p/5673133.html)，博客园，求知，2016年7月15日
4. [Bash的history命令](http://rockhong.github.io/history-in-bash.html)，Hong's Blog，Hong，2015年9月7日
5. [Linux command line tips: history and HISTIGNORE in Bash](https://www.techrepublic.com/article/linux-command-line-tips-history-and-histignore-in-bash/)，TechRepublic，Nick Gibson，2007年9月3日
6. [Linux| 用户目录下三个bash文件的作用(.bash_history,.bash_logout,.bash_profile,.bashrc)](https://blog.csdn.net/u011479200/article/details/86501366)，CSDN，YvesHe，2019年1月21日 
7. ![title](https://leanote.com/api/file/getImage?fileId=5de702e8ab644153da004ef1)

