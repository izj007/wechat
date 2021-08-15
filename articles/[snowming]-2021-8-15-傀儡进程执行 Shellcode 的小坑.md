>- 因为蚂蚁笔记官方图片服务器极其不稳定，如果本文出现图片加载不出的问题，请转到此链接查看： 
https://shimo.im/docs/8XJKxH9PxqgKp33c/ 《傀儡进程执行 Shellcode 的小坑》，可复制链接后用石墨文档 App 或小程序打开


三好学生在[傀儡进程的实现与检测](https://3gstudent.github.io/3gstudent.github.io/%E5%82%80%E5%84%A1%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B8%8E%E6%A3%80%E6%B5%8B/)一文里面提到了傀儡进程。但是在那篇文章中，其实是 process hollowing/RunPE 技术。众所周知，所谓进程，就是pe文件的执行在内存中的映射。那么 process hollowing/RunPE 技术就是把一个挂起进程的内存数据清空，然后将一个 PE 文件映射到内存，再将进程的入口点替换为 PE 文件在内存中的起始地址，最后恢复进程状态，执行 PE 文件。


但这种所谓的 process hollowing/RunPE 技术对于 shellcode 注入来说有点过于重量级，因为镂空一个 PE 文件写入 shellcode 动静较大。所谓的镂空就是使用 NtUnmapViewOfSection 卸载正在执行的PE文件在内存中的映像。所以在进程 shellcode 注入的时候，更好地选择可能是不镂空进程，而直接修改进程的 EIP/RIP ，指向 shellcode 在内存中的起始地址。


所以调用链大概是这样的：

1. 创建挂起进程（使用 `CREATE_SUSPENDED`标志调用 `CreateProcess` API）；或者使用 SuspendThread 函数挂起目标线程。
2. VirtualAllocEx函数申请一个可读、可写、可执行的内存。
3. 调用WriteProcessMemory将Shellcode数据写入刚申请的内存中。
4. 调用GetThreadContext，设置获取标志为CONTEXT_FULL，即获取新进程中所有线程的上下文。
5. 修改线程上下文中EIP/RIP的值为申请的内存的首地址，通过SetThreadContext函数设置回主线程中。
6. 调用ResumeThread恢复主线程。
      
**总之也就是远线程注入+修改远线程的 EIP/RIP。**


其中要注意的是，我们要通过GetThreadContext获得所有寄存器的信息(保存在结构体_CONTEXT中)，然后通过 _CONTEXT 结构获取执行指针。


- x64系统的进程入口地址为 `CONTEXT.RIP`；
- x86系统的进程入口地址为 `CONTEXT.EIP`。


代码实现中可以通过宏来进行定义：


```
#ifdef _WIN64
	lpAddr = newContext.Rip;
#else 
	lpAddr = newContext.Eip;
#endif
```

但是最终在成品 Demo 中，我遇到了一个问题，就是 32-bit 的 shellcode 可以成功执行，但是 64-bit 的 shellcode 无法成功执行。测试得出直到 GetThreadContext 都是成功执行的，但是到 SetThreadContext 远线程就会挂掉。


调试发现：

![title](https://images-cdn.shimo.im/EDzweTRrwz42a1OM__thumbnail.png)



查看远线程的主进程执行指针，Rip 高8位地址没有修改成功。回头看我的代码：

```
#ifdef _WIN64
    #define XIP Rip
#else
    #define XIP Eip
#endif

	newContext.XIP = DWORD(lpAddr);
```

强转为 DWORD 使得64位系统的8字节地址损失了4字节的数据，所以设置 Rip 时候有8-bit的数据丢失了。

于是就知道如何修改了，写一个宏将 `Rip` 强转为 `DWORD64`/`unsigned long long` 类型；将 `Eip` 强转为 `DWORD` 类型。

另外补充一点，使用傀儡进程的方式执行 shellcode，普通的 shellcode 执行完进程会挂掉，因为一般 shellcode 的结尾都是 TerminateProcess，但是一般远控生成的 shellcode 不会，因为一旦流程控制权限转交给 shellcode，最终会不断执行死循环，维持通信，这样会一直挂住进程，进程不会挂掉。






---------------


# 拓展阅读：

1. [傀儡进程的实现与检测](https://3gstudent.github.io/3gstudent.github.io/%E5%82%80%E5%84%A1%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B8%8E%E6%A3%80%E6%B5%8B/)， 三好学生的博客，3gstudent，2017-11-30
2. [SetContext Hijack Thread](https://idiotc4t.com/code-and-dll-process-injection/setcontext-hijack-thread)，idiotc4t's blog，idiotc4t，2020-06

