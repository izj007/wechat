# 0x01 Python 标准库
什么是 python 标准库？

 - python 的标准库是随着 pyhon 安装的时候默认自带的库。
 - python 的第三方库，需要下载后安装到 python 的安装目录下，不同的第三方库安装及使用方法不同。
 - 它们调用方式是一样的，都需要用 import 语句调用。
 - 简单的说，一个是 python 默认自带不需要下载安装的库，一个是需要下载安装的库。它们的调用方式是一样的。

--------------


# 0x02 Python 标准库危险模块
 - os 
 - commands
 - subprocess 
 - pty


通过这四个危险标准库模块，我们最终的目的是**拿到 shell 权限**。

--------------------


# 0x03 OS 模块


Python 标准库中的 OS 模块是操作系统模块，提供各种各样的操作系统接口。
 
## 主要功能：

os 模块主要有以下4点功能：

- 系统相关
- 目录及文件操作
- 执行命令
- 管理进程


## 命令执行：

**1、system 函数** —— 将字符串转化成命令在服务器上运行

`system` 函数可以将字符串转化成命令在服务器上运行；其原理是每一条 system 函数执行时，其会创建一个子进程在系统上执行命令行，子进程的执行结果无法影响主进程。

**system 函数执行单条命令：**

 ![title](https://leanote.com/api/file/getImage?fileId=5db93f18ab6441112e0009e5)
 
 
 `注意：`system 函数的原理是每一条 system 函数执行时，其会创建一个子进程在系统上执行命令行，子进程的执行结果无法影响主进程；
上述原理会导致当需要执行多条命令行的时候可能得不到预期的结果；

![title](https://leanote.com/api/file/getImage?fileId=5db94442ab6441112e000a05)
![title](https://leanote.com/api/file/getImage?fileId=5db94473ab644113360009b9)

上述程序运行后会发现 `test2` 文件夹并没有创建在 `/test` 文件夹下，而是在当前的目录下；

**system 函数执行多条命令：**

为了保证 system 执行多条命令可以成功，多条命令需要在同一个子进程中运行；

![title](https://leanote.com/api/file/getImage?fileId=5db9430eab644113360009ac)


**2、popen 函数** —— 新建一个管道执行命令


**功能：**

os.popen() 方法用于从一个命令打开一个管道。
在 Unix，Windows 中有效。


**语法：**
``` shell
os.popen(command[, mode[, bufsize]])
```

**参数：**

- command —— 命令。使用的命令
- mode —— 权限。模式权限可以是 `'r'`(默认) 或 `'w'`
- bufsize —— 缓冲区大小。指明了文件需要的缓冲大小：0意味着无缓冲；1意味着行缓冲；其它正值表示使用参数大小的缓冲（大概值，以字节为单位）。负的bufsize意味着使用系统的默认值，一般来说，对于 tty 设备，它是行缓冲；对于其它文件，它是全缓冲。如果没有改参数，使用系统的默认值。

**返回值：**

返回一个文件描述符号为 `fd` 的打开的文件对象。 

**实例：**

![title](https://leanote.com/api/file/getImage?fileId=5db94988ab6441112e000a3f)

**获取执行内容：**

如果想要获取 popen 执行命令的文件内容，那么可以使用如下几个函数来获取执行内容：

- read() 读取整个文件，并将整个文件放入一个字符串变量中
- readline()  每次读取一行，返回一个字符串对象并保留当前行的内存
- readlines() 读取整个文件，并将整个文件按行解析成列表

![title](https://leanote.com/api/file/getImage?fileId=5db95790ab64411336000a65)


注意：

在想要使用 popen 搭配上面这几个 read 函数获取命令执行结果时请务必注意换行符 `\n`，进行字符串处理时需对该 `\n` 符进行处理。


**命令执行：**

![title](https://leanote.com/api/file/getImage?fileId=5db94c2eab6441112e000a55)


--------------------


# 0x04 commands 模块（仅适用于 Linux/Unix 环境）
 
Python 标准库中的 `commands` 模块属于 Unix 特定服务模块，是运行命令的实用工具。


commands 模块从2.6版开始不推荐使用：**该 commands 模块已在 Python3 中删除**。请改用 `subprocess` 模块。

![title](https://leanote.com/api/file/getImage?fileId=5db95c72ab6441112e000b02)


## 函数：

commands 模块只有`3`个方法。 commands 模块中包含 `os.popen()` 函数的装饰器函数，`os.popen()` 方法会把传入的字符串作为系统命令执行并返回此命令生成的任意输出，以及（可选的返回）命令执行的离开状态。



|方法     |    功能 |
| :--:      | :--:     | 
|commands.getstatusoutput(cmd)|返回一个元组（status，output），0为成功状态的返回值，非0为失败状态的返回值|
|commands.getoutput('cmd')|此函数只返回结果,不返回返回值 |
|commands.getstatus('file')|此函数返回 ls -ld file 的执行结果  |

该 commands 模块定义了以下函数：

- **commands.getstatusoutput(cmd)**

在一个 shell 中使用 os.popen() 方法执行 cmd 字符串并返回一个二个元素的元组（status, output）。cmd 实际上是这样运行的：`{ cmd ; } 2>&1`，因此返回的输出会包含输出或错误消息。从输出中删除尾随的换行符。可以根据 C 函数 wait() 的规则解释命令的退出状态。


- **commands.getoutput(cmd)**

与 getstatusoutput() 方法相似getstatusoutput()，除了退出状态被忽略，返回值是包含命令输出的字符串。

- **commands.getstatus(file)**（现已被弃用）

以字符串形式返回 `ls -ld file` 的输出。此函数使用了 `getoutput() ` 函数，并在参数中正确转义反斜杠和美元符号。
注：此方法从2.6版开始不推荐使用：此功能不明显且无用。在的情况下，这个名字也会引起与 getstatusoutput() 方法混淆的误会。

## 注意：

commands 是提供 **linux** 系统环境下支持使用 shell 命令的一个模块。commands 模块专门用于执行 **Linux  shell** 命令。

commands 模块通过 python 调用系统命令，只适用于 **Linux**!!!

如图，在 Windows 环境下使用 commamds，不会正常回显：

![title](https://leanote.com/api/file/getImage?fileId=5db989acab644109b900001e)
status 代表 shell 命令的返回态，如果成功的话是0；这里是1说明命令执行失败。


但是在 Linux 环境下使用 commands，就会正常回显：

![title](https://leanote.com/api/file/getImage?fileId=5db98a4bab644109b9000020)


## 命令执行：

- commands.getoutput(cmd)

![title](https://leanote.com/api/file/getImage?fileId=5db98cdbab644109b900002e)
![title](https://leanote.com/api/file/getImage?fileId=5db98d0aab64410bb700002e)

- commands.getstatusoutput(cmd)

![title](https://leanote.com/api/file/getImage?fileId=5db98e09ab644109b9000032)
![title](https://leanote.com/api/file/getImage?fileId=5db98de2ab64410bb7000037)


--------------------

# 0x05 subprocess 模块


subprocess 的目标是启动一个新的进程并与之进行通讯。


## subprocess.Popen 类：

这个模块主要就提供一个类 Popen：

``` python
class subprocess.Popen( 
      args, 
      bufsize=0, 
      executable=None,
      stdin=None,
      stdout=None, 
      stderr=None, 
      preexec_fn=None, 
      close_fds=False, 
      shell=False, 
      cwd=None, 
      env=None, 
      universal_newlines=False, 
      startupinfo=None, 
      creationflags=0
      )
```


![title](https://leanote.com/api/file/getImage?fileId=5dba40f5ab64415ac80000c5)


- `args` 参数

可以是一个字符串，可以是一个列表。

## args 参数详解：

`args` 参数可以是一个字符串，也可以是一个列表。

``` python
subprocess.Popen(["ls","vulhub"])
subprocess.Popen("ls vulhub")
```

![title](https://leanote.com/api/file/getImage?fileId=5dba43dbab64415ac80000ee)

在 **Linux** 环境下，这两个之中，后者将不会工作。因为如果是一个字符串的话，必须是程序的路径才可以。(考虑 unix 的 api 函数 exec，接受的是字符串列表)

但是下面的可以工作：

``` python
subprocess.Popen("ls vulhub", shell=True)
```

![title](https://leanote.com/api/file/getImage?fileId=5dba4750ab644158ca00016b)

这是因为它相当于：

``` shell
subprocess.Popen(["/bin/sh", "-c", "ls vulhub"])
```

都成了 sh 的参数，就可以生效了。

在 **Windows** 下，下面的 python 代码却又是可以工作的：

``` python
subprocess.Popen(["notepad.exe", "abc.txt"])
subprocess.Popen("notepad.exe abc.txt")
```

![title](https://leanote.com/api/file/getImage?fileId=5dba4896ab64415ac8000123)

![title](https://leanote.com/api/file/getImage?fileId=5dba48ecab64415ac8000128)


这是由于 Windows 下的 api 函数 CreateProcess **接受的是一个字符串**。即使是列表形式的参数，也需要先合并成字符串再传递给 api 函数。

类似上面、

``` python
subprocess.Popen("notepad.exe abc.txt",shell=True)
```

![title](https://leanote.com/api/file/getImage?fileId=5dba49b5ab64415ac8000132)

等价于

``` python
subprocess.Popen("cmd.exe /C "+"notepad.exe abc.txt",shell=True)
```

![title](https://leanote.com/api/file/getImage?fileId=5dba49f6ab644158ca00017a)


## 便利函数：

模块还提供了几个便利函数（这本身也算是很好的 Popen 的使用例子了）。

- call 方法 —— 执行程序，并等待它完成

``` python
def call(*popenargs, **kwargs):
    return Popen(*popenargs, **kwargs).wait()
```

- check_call() 方法 —— 调用前面的call，如果返回值非零，则抛出异常

``` python
def check_call(*popenargs, **kwargs):
    retcode = call(*popenargs, **kwargs)
    if retcode:
        cmd = kwargs.get("args")
        raise CalledProcessError(retcode, cmd)
    return 0
```    
    
- check_output() 方法 —— 执行程序，并返回其标准输出

``` python
def check_output(*popenargs, **kwargs):
    process = Popen(*popenargs, stdout=PIPE, **kwargs)
    output, unused_err = process.communicate()
    retcode = process.poll()
    if retcode:
        cmd = kwargs.get("args")
        raise CalledProcessError(retcode, cmd, output=output)
    return output
```


## Popen 方法：

Popen 对象提供有不少方法函数可用。

![title](https://leanote.com/api/file/getImage?fileId=5dba4deaab64415ac8000156)

但是这些方法跟执行 shell 的关系不太大，执行 shell 的方法还是主要是：

- call()


## 命令执行：

![title](https://leanote.com/api/file/getImage?fileId=5dba53f8ab644158ca0001f2)

![title](https://leanote.com/api/file/getImage?fileId=5dba4fb1ab644158ca0001bf)


![title](https://leanote.com/api/file/getImage?fileId=5dba50e1ab644158ca0001c7)


注：`getoutput()` 方法不能执行 cmd 命令。
![title](https://leanote.com/api/file/getImage?fileId=5dba5159ab644158ca0001d1)


`check_output()` 方法在 Windows 环境下执行有问题：

![title](https://leanote.com/api/file/getImage?fileId=5dba5471ab644158ca0001f7)

在 Linux 环境下执行正常：

![title](https://leanote.com/api/file/getImage?fileId=5dba5528ab64415ac80001a4)

可能是因为默认 shell 是 Linux shell。

-------------------


# 0x06 pty 模块（仅适用于 Linux/Unix 环境）


Python 标准库中的 pty 模块属于 Unix 特定服务模块，是伪终端实用程序。

## spawn 方法

``` python
pty.spawn(argv[, master_read[, stdin_read]])
```

此方法仅适用于 Linux 及 Unix 环境，如果在 Windows 环境下 import pty 方法，会获得如下报错：

![title](https://leanote.com/api/file/getImage?fileId=5dbad936ab64415ac8000478)




Linux 下获取交互式 shell：

![title](https://leanote.com/api/file/getImage?fileId=5dbad748ab64415ac800046f)


当然 Windows 环境下不可以：

![title](https://leanote.com/api/file/getImage?fileId=5dbad793ab64415ac8000470)

注：Windows 环境下有 `winpty`，但是这个 `winpty` 不属于标准库，是第三方库，还需要下载安装。
不过 Windows 似乎就没有交互式 shell 的说法，meterpreter 已经很好了。强行 Windows 下交互 shell 比较变态。

其他 Linux 环境下命令执行：

![title](https://leanote.com/api/file/getImage?fileId=5dbad868ab64415ac8000475)



-------------------


# 0x07 总结

在 python 的标准库中，可以执行 shell 的危险模块有：

- os
- commands（仅适用于 Python2，仅适用于 Linux/Unix 环境）
- subprocess
- pty（仅适用于 Linux/Unix 环境）


具体方法为：

- os.system('cmd')
- os.popen('cmd').read()
- commands.getoutput('cmd')
- commands.getstatusoutput('cmd')
- subprocess.Popen(['cmd'],shell=True)
- subprocess.call(['cmd'],shell=True)
- pty.spawn('cmd')
- python -c 'import pty;pty.spawn("/bin/sh")'
- python -c 'import pty;pty.spawn("bash")'


示例：

``` shell
import os
import subprocess
import commands
import pty

# 直接输入shell命令,以ifconfig举例
os.system('ifconfig')
os.popen('ifconfig')
commands.getoutput('ifconfig')
commands.getstatusoutput('ifconfig')
subprocess.Popen(['ifconfig'],shell=True)
subprocess.call(['ifconfig'],shell=True)
pty.spawn('ifconfig')
```


--------------------

参考链接：

[1] [python 标准库和第三方库的区别](https://blog.csdn.net/aaronthon/article/details/81714522)
[2] [Python 操作系统模块](http://www.langzi.fun/Python%20os%20%E6%A8%A1%E5%9D%97.html)
[3] [python 基础之 os.system 函数](https://www.cnblogs.com/cwp-bg/p/8465566.html)
[4] [Python os.popen() 方法 ](https://www.cnblogs.com/xishaonian/p/7612479.html)
[5] [Python os.popen() 方法](https://www.runoob.com/python/os-popen.html)，菜鸟教程
[6] [python 的 popen 函数](https://blog.csdn.net/Z_Stand/article/details/89375589)，CSDN
[7] [36.16. commands — Utilities for running commands](https://docs.python.org/2/library/commands.html)，Python 官方文档
[8] [23.1. cmd — Support for line-oriented command interpreters](https://docs.python.org/2/library/cmd.html)，Python 官方文档
[9] [cmd – Create line-oriented command processors](https://pymotw.com/2/cmd/)，PyMOTW
[10] [commands – Run external shell commands](https://pymotw.com/2/commands/)，PyMOTW
[11] [Python os模块参考手册](http://kuanghy.github.io/2015/08/02/python-os)
[12] [os 模块](https://wiki.jikexueyuan.com/project/explore-python/File-Directory/os.html)，极客学院
[13] [python之commands模块](https://blog.51cto.com/lookingdream/2313078?source=dra)，51CTO 博客
[14] [python commands 模块](https://www.cnblogs.com/lijunjiang2015/p/7816016.html)，博客园
[15] [python commands模块的用法](https://www.jianshu.com/p/d29b8a2119ad)，简书
[16] [17.1. subprocess — Subprocess management](https://docs.python.org/2/library/subprocess.html#module-subprocess)，Python 官方文档
[17] [Python模块subprocess小记](https://blog.csdn.net/wirelessqa/article/details/7778761)，CSDN
[18] [PEP 324 -- subprocess - New process module](https://www.python.org/dev/peps/pep-0324/)，Python 官方文档
[19] [pty — Pseudo-terminal utilities](https://docs.python.org/3/library/pty.html)，Python 官方文档
