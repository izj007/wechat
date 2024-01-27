#  使用direct syscall进行反调试

原创 wondeekun [ 安全的矛与盾 ](javascript:void\(0\);)

**安全的矛与盾** ![]()

微信号 gh_b4c853063b88

功能介绍 知攻、知守、知进退；有矛、有盾、有安全。

____

___发表于_

## 前言

反调试是对抗中十分常用的技术，但是怎么把反调试做的更加隐蔽，一直都是一个难题。其实很多反调试的技巧会被调试器插件自动屏蔽掉，还有一些反调试技巧十分显而易见，直接静态patch就解决了。今天我们要讨论一个使用direct
syscall进行实现的相对比较隐蔽的反调试方法。

## 基础知识

一种非常常见的反调试技巧是使用windows
api将当前线程设置为对调试器不可见，当线程被设置为对调试器隐藏后，它将继续运行，但调试器不会接收与此线程相关的事件。但是，如果隐藏线程中存在断点，或者如果我们从调试器中隐藏主线程，则进程将崩溃，调试器将卡住。

windows有个API `NtSetInformationThread` 可以用来设置线程的优先级

![]()

但是 ThreadInformationClass 有一个没有文档化的一个值，ThreadHideFromDebugger ，是下面枚举类型没有给出的。

    
    
    typedef enum _THREADINFOCLASS {  
        ThreadBasicInformation,                            // ??  
        ThreadTimes,  
        ThreadPriority,                                    // ??  
        ThreadBasePriority,                                // ??  
        ThreadAffinityMask,                                // ??  
        ThreadImpersonationToken,                        // HANDLE  
        ThreadDescriptorTableEntry,                        // ULONG Selector + LDT_ENTRY  
        ThreadEnableAlignmentFaultFixup,                // ??  
        ThreadEventPair,                                // ??  
        ThreadQuerySetWin32StartAddress,                // ??  
        ThreadZeroTlsCell,                                // ??  
        ThreadPerformanceCount,                            // ??  
        ThreadAmILastThread,                            // ??  
        ThreadIdealProcessor,                            // ??  
        ThreadPriorityBoost,                            // ??  
        ThreadSetTlsArrayAddress,                        // ??  
        MaxThreadInfoClass  
    } THREADINFOCLASS;  
    

我们可以使用这个值设置线程从调试器中隐藏。下面就展示一下怎么使用：

    
    
    #define NtCurrentThread ((HANDLE)-2)  
    #define ThreadHideFromDebugger 0x11   
      
    typedef   
    NTSTATUS ( NTAPI *pNtSetInformationThread)(  
        IN HANDLE ThreadHandle,  
        IN THREAD_INFORMATION_CLASS ThreadInformationClass,  
        IN PVOID ThreadInformation,  
        IN ULONG ThreadInformationLength  
    );  
      
    bool AntiDebug()  
    {  
        HMODULE ntdll = LoadLibraryA("ntdll");  
        pNtSetInformationThread NtSetInformationThread = (pNtSetInformationThread)GetProcAddress( ntdll,"NtSetInformationThread");  
      
        NTSTATUS status = NtSetInformationThread(  
            NtCurrentThread,  
            (THREAD_INFORMATION_CLASS)ThreadHideFromDebugger,  
            NULL,  
            0);  
        return status >= 0;  
    }  
    

当我们不调用 antiDebug 函数之前，我们的断点是正常的

![]()

在调用call之前，断点是正常的，但是调用call之后，调试器就卡住了，然后进程就会退出。

![]()

具体原理通过逆向内核中`NtSetInformationThread` 函数的实现，发现是线程对象 EPTHREAD
中有一个未公开的标志位，此函数就是修改此标志位实现的隐藏。

## 使用direct syscall实现

此函数在用户态到底是干了什么呢？直接看一波源码；

![]()

这用户态啥也没干呀，直接进入内核了，这可以使用syscall来写了；直接使用橘哥的 `PigSyscall`
`https://github.com/evilashz/PigSyscall`；

看了一下代码，函数的字符串hash需要使用宏定义，就随后写了一个编译时计算hash的代码，以防止出现字符串明文，具体实现如下：

    
    
    template <unsigned int size>  
    class Hash_String {  
    public:  
        volatile DWORD32 value;  
        __forceinline constexpr  Hash_String(const char* string, UINT count) :value(0)  
        {  
      
            DWORD Mask = (CHAR_BIT * sizeof(value) - 1);  
      
            count &= Mask;  
      
            for (unsigned i = 0u; i < size; ++i)  
            {  
                value = string[i] + (value >> count | value << ((Mask + 1 - count) & Mask));  
            }  
      
        }  
      
    };  
      
    #define HASH(my_string) ([]{ constexpr Hash_String<(sizeof(my_string)/sizeof(char) -1)> name(my_string,0xd8); return name.value; }())  
      
        syscall.CallSyscall(HASH("NtSetInformationThread"),  
            NtCurrentThread,  
            ThreadHideFromDebugger,  
            NULL,  
            0);  
    

编译时编译器会自动将 `NtSetInformationThread` 转化为对应的hash，所以在二进制中并不会出现明文；

![]()

成功！！

## 参考

https://www.lodsb.com/ntsetinformationthread-disabling-threadhidefromdebugger

## 投稿

  * 欢迎大家私信微信Manliness_man投稿，我们将按照【个人简介】+【原文】的形式免费曝光您的文章，提高您的知名度，并有礼品赠送哦
  * 原创文章、翻译文章、复现笔记、学习体会等网络安全攻防相关文章均可

![]()

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 使用direct syscall进行反调试

原创 wondeekun [ 安全的矛与盾 ](javascript:void\(0\);)

轻触阅读原文

![]()

安全的矛与盾

赞 分享 在看

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

