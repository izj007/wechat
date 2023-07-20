#  怎样让shellcode在内存中始终不可见(shellGhost分析)

原创 wonderkun  [ 安全的矛与盾 ](javascript:void\(0\);)

**安全的矛与盾** ![]()

微信号 gh_b4c853063b88

功能介绍 知攻、知守、知进退；有矛、有盾、有安全。

____

___发表于_

收录于合集

# 前言

逛github的时候看到了一个项目`ShellGhost`，介绍了一种内存规避技术，可以让shellcode在进程从开始到结束都始终是不可见的。讲的还是挺邪乎的，不过我盲猜是利用异常获取CPU控制权去动态修改shellcode的内容，这就下载学习一下是怎么实现的。

# shellcode的预处理

项目提供了一个python脚本 `ShellGhost_mapping.py`
来对shellcode进行处理，主要是使用`ndisasm`对shellcode进行反汇编，然后分别获取到每一条指令的相对偏移以及长度，保存在一个数组中。

代码如下：

![]()

处理之后的结果示例如下：

![]()

RVA保存的就是指令的相对偏移，quota保存的就是当前的指令长度。记住这个表的结构，它会贯穿后续shellcode执行全过程。

最后是输出每条指令被RC4加密之后的结果，注意是每条指令单独加密，而不是所有指令一起加密。将加密后的shellcode，密钥，指令长度，以及上述的表写入到c文件中。

![]()好了，数据准备工作到此为止，接下来开始分析执行过程。

# 初始化过程

首先看一下main函数：

![]()

首先，申请的内存并不是可执行的，然后将这块内存都修改为0xcc软中断；

另外比较怪异的一点是启动的新线程的的起始地址是.text节的末尾全是 \x00 的地方。

`ResolveEndofTextSegment`会返回`.text`末尾全是`\x00`的起始地址，详细代码如下：

![]()

为什么要这么做呢？作者在注释里也解释了，靠近`entry_point`位置的代码或者地址其实是被杀毒软件重点关注的，其实在引擎中很多特征码的定位都是以`entry_point`
作为起始地址的。

最后注册了一个向量化异常处理函数`InterceptShellcodeException` ，这个处理函数是shellcode可以执行的关键。

# shellcode的执行

`InterceptShellcodeException`
异常处理函数是shellcode可以正常执行的关键，可以看到在上面创建的线程的起始地址也不是shellcode，接下来怎么控制EIP指向shellcode呢？其实精髓就是在这个异常处理函数。

首先要知道的是`.text`节末尾全是`\x00`,反汇编结果如下图所示，在这里创建线程一定会触发异常,`rax`一定不会是一个可写的地址，想一想是为什么？

![]()

因为看代码：

    
    
    hThread = CreateThread(0, 0, (LPTHREAD_START_ROUTINE)ResolveEndofTextSegment(), 0, 0, 0);

RAX此时应该保存的刚好是
`ResolveEndofTextSegment()`的返回值，text节显然是不可写的。这里处理的还是比较精妙的，需要仔细的琢磨一下。接下来执行流程就进入了异常处理函数：

![]()

首先判断异常是不是在自己需要处理的地址范围内，然后判断如果异常发生的地方在.text的默认，就将执行流(eip)修改到堆上，但是当前的堆上有两个问题：

  1. 1. 是只可读写，不可执行的

  2. 2. 全是0xcc

继续阅读代码，看是怎么处理的：

![]()

首先是调用 `ResolveInstructionByRva` 修改当前的指令为解密后的指令，此函数如下：

    
    
    // Edits current breakpoint to be the next instruction to decrypt  
    NTSTATUS ResolveInstructionByRva(PVOID pointer) //修改当前指令  
    {  
        DWORD64 rva = ResolveBufferFeature(pointer, INSTRUCTION_OPCODES_RVA); // 获取当前地址的rva  
        for (DWORD i = 0; i < instruction[ResolveBufferFeature(pointer, INSTRUCTION_OPCODES_NUMBER)].quota; i++)  
        {  
            *(BYTE*)((BYTE*)pointer + i) = *(BYTE*)((BYTE*)sh + rva + i);  
        }  
      
        return STATUS_SUCCESS;  
      
    }

其中`ResolveBufferFeature`就会利用之前存储的`instruction`找到当前指令的RVA和quota，然后进行修改。

然后对内存的这一条指令进行解密，解密结束之后修改内存的权限为可执行，然后继续执行这一条指令。

当执行下一条指令时，由于内容是`0xCC`,所以会再次触发异常，继续走上面的异常函数处理逻辑，达到了单条指令依次执行的目的。

当在执行下一条指令的时候会顺便把上一条指令给抹掉，防止在内存中保留有可见的shellcode。

![]()

相信已经讲解的够清楚，为了保险起见，还是来画个图更形象的说明一下。

![]()

# 总结

如此执行shellcode具备隐藏内存，反调试等优点，但是唯一的问题就是执行效率太低了。我虽然没有想到有什么实际的用途，但是作者的代码写的非常的工整，也可以给我们写其他的内存规避提供一些实现思路。

 **最后欢迎加入知识星球进行深度交流~**

![]()

 **咨询微信**  

![]()

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

