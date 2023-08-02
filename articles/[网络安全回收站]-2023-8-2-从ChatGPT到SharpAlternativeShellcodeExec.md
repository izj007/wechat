#  从ChatGPT到SharpAlternativeShellcodeExec

原创 yzddMr6 [ 网络安全回收站 ](javascript:void\(0\);)

**网络安全回收站** ![]()

微信号 gh_cd24c9599f5f

功能介绍 这里是yzddmr6的公众号，博客的移动端版本，存放本人写的一些垃圾文章。

____

___发表于_

收录于合集

**背景**

利用ShellCode进行免杀是一种最常见的免杀方式，但是常见的VirtualAlloc、CreateRemoteThread这些Windows
API已经被各大杀软重点监控。那么与之相对的绕过的办法就是利用一些小众的Windows
API，这些API函数往往提供了回调的功能，当它的参数是指针类型的话就可以直接执行内存当中的ShellCode，这样就绕过了敏感函数识别达到执行ShellCode的目的。

  

最近研究了一下

https://github.com/aahmad097/AlternativeShellcodeExec这个项目，该项目提供了很多绕过的API函数。在渗透测试的过程中为了逃避检测，我们常常希望实现内存加载，无文件攻击。原项目是C++写的，PE转ShellCode等方法有时候并不稳定。而C#可以直接内存加载，并且从Windows
XP以来每台Windows上都默认安装了.NET，所以就萌生了用C#重写一遍的方法，同时也可以加深对项目的理解。

但是C#调用Windows
API的时候需要额外进行函数的声明，并且要实现C++类型到C#类型的转化，这部分的工作十分的繁琐，也不感兴趣。所以就想到了用ChatGPT帮我去做转化，也算蹭一波热度吧。

  

##  **初步尝试**

先随便找了一个回调改写试试，发现就算是硬编码ShellCode+裸的API调用，C#重写后的版本也比原版本少20个左右引擎的检出。可能是原来的项目已经被各大厂商都加过规则了，C#版本的还没有，看起来有搞头。

 **C++版本**

![]()

  

 **C#版本**

![]()

  

##  **调教ChatGPT**

不得不说ChatGPT确实很牛逼，本来只是想让他转换一个函数声明，但是最后几乎替我完成了一大半的重写工作（失业警告）。少部分情况下一次就能转换出能运行的代码，大多数情况我们也只需要对生成的代码进行微调即可。不过GPT偶尔也有自作聪明，胡说八道的情况，这个时候就需要调整我们的prompt来一步步引导他。

  

###  **替换等价函数**

最开始的咒语很简单：

将以下代码用C#重写:xxxx，但是ChatGPT有时候会自作聪明的把我们想要触发的回调函数进行等价替换了，这样也就达不到我们的效果。

![]()

  

![]()

  

###  **错误的DllImport提示**

  

![]()

GPT认为EnumICMProfilesW是在mscms.dll中，但是实际上这个函数是存在于gdi32.dll中的：

https://learn.microsoft.com/en-us/windows/win32/api/wingdi/nf-wingdi-
enumicmprofilesw

![]()

###  **错误的参数位置**

在重写EnumDesktopWindows这个回调函数的时候，ChatGPT一直给出了错误的参数调用，用于回调的addr指针应该放到倒数第二个参数，但是ChatGPT一直认为要放到倒数第一个参数，怎么提示也没用，后来自己改了。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    using System;using System.Runtime.InteropServices;  
    class Program{    [DllImport("kernel32.dll", SetLastError = true)]    static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);  
        [DllImport("ntdll.dll")]    static extern void RtlMoveMemory(IntPtr dest, byte[] src, uint length);  
        [DllImport("user32.dll")]    static extern bool EnumDesktopWindows(IntPtr hDesktop, EnumWindowsProc lpfn, IntPtr lParam);  
        delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);  
        static bool MyCallback(IntPtr hWnd, IntPtr lParam)    {        // Do something with hWnd        return true;    }  
        static void Main(string[] args)    {        // alfarom256 calc shellcode        byte[] op = new byte[] { };  
            IntPtr addr = VirtualAlloc(IntPtr.Zero, (uint)op.Length, 0x1000, 0x40);        RtlMoveMemory(addr, op, (uint)op.Length);  
            if (addr != IntPtr.Zero)            EnumDesktopWindows(GetThreadDesktop(GetCurrentThreadId()), new EnumWindowsProc(MyCallback), addr);    }  
        [DllImport("user32.dll")]    static extern IntPtr GetThreadDesktop(uint dwThreadId);  
        [DllImport("kernel32.dll")]    static extern uint GetCurrentThreadId();}

  

![]()

重写后的代码也是错误的。

  

##  **整体感受**

讲完了ChatGPT的坑，其实总体体验下来感觉还是不错的，遇到不熟悉的API可以直接问他，体验比去找文档好的多。

  

![]()

  

并且不仅会给出转换后的代码，并且还会告诉你代码的大体逻辑，以及需要注意的点。

![]()

  

代码逻辑的梳理

![]()

  

遇到报错贴给他，他也会给出一定的解决方案，当然还是需要人工检验的。

![]()

利用零零碎碎的时间，终于把45个Project都重写完了，在此期间也体会到了ChatGPT在提升生产力中的应用。

  

##  **进一步优化**

在有思路的情况下，还可以用ChatGPT进一步提高免杀效果。随便改一下做个例子：

![]()

![]()

![]()

  

![]()

  

![]()

最后的代码也是可用的，弹个计算器

![]()

  

##  **无法运行的FiberContextEdit**

在重写后的45个项目中，利用FiberContextEdit方法的始终无法正常运行。问了ChatGPT后应该是跟C++中函数转换到C#后偏移量不同有关，限制于水平问题没有深入研究，有知道原因的同学可以私下里交流交流

![]()

  

![]()

项目代码已经同步到我的github，如果有问题欢迎反馈：

https://github.com/yzddmr6/SharpAlternativeShellcodeExec

  
  

  

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

