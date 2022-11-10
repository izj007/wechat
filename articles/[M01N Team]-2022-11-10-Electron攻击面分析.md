#  Electron攻击面分析

原创 天元实验室  [ M01N Team ](javascript:void\(0\);)

**M01N Team** ![]()

微信号 m01nteam

功能介绍 攻击对抗研究分享

____

___发表于_

收录于合集 #蓝军 54个

**01** 简介

在今年Blackhat的会议上，安全研究员Aaditya Purani和Max GarreG分享了议题《Pwning Popular Desktop
apps while uncovering new attack surface on Electron》，
分享了几个Electron利用的新方法和他们挖到几个Electron应用的漏洞案例。

  

Electron是一个桌面应用程序的框架，极大的方便了跨平台应用的开发。Chromium和Node
JS是Electron的重要组成部分。因为Electron拥有直接执行Nodejs代码的能力，并且内置了Chromium内核。一个XSS漏洞，很可能就会导致RCE。

  

本文以安全研究员Aaditya Purani和Max
GarreG的思路为依据，简单聊一聊Electron结构，上下文进行和Electron框架XSS到RCE的攻击面。

  

 **02  **Electron基本构架

Electron是使用JavaScript，HTML和CSS构建跨平台的桌面应用程序的框架，可构建出兼容Mac、Windows和Linux三个平台的应用程序。

  

Electron使用Chromium完成UI渲染， 通过内置Node.js提供原生系统的能力，如文件系统和网络的访问。

![](https://gitee.com/fuli009/images/raw/master/public/20221110215726.png)

  

Electron继承了来自Chromium的多进程架构，每个Electron应用都有一个单一的主进程，作为应用程序的入口点。主进程在、Node.js环境中运行，这意味着它具有require模块和使用所有Node.js
API的能力。主进程的主要目标是使用BrowserWindow模块创建和管理应用程序窗口。

  

每个Electron应用都会为每个打开的BrowserWindow( 与每个网页嵌入 )
生成一个单独的渲染器进程。渲染器负责渲染网页内容。渲染器进程也有访问NodeJs共享库的能力，但是需要依赖系统配置。

  

渲染器进程和主进程直接存在隔离，通过进程IPC进行通信，一般我们通过XSS拿到的JS执行权限处于渲染进程之中。

![](https://gitee.com/fuli009/images/raw/master/public/20221110215727.png)

  

渲染进程的上下文也可以分为两种:

Preload.js和网页上下文，preload的上下文访问权限，一般高于网页上下文 。

  

这时有两种思路获取权限：

利用渲染进程本身进行RCE

  * 通过NodeJs共享库RCE

  * 通过chromium Nday RCE

通过IPC，影响主进程进行RCE

  * 需要主进程ipcmain，实现了危险方法

  * 需要当前执行上下文可以访问IPC

  

 **03  **Electron核心选项

而我们可以使用哪种思路，还需取决于Electron选项，以下几个是值得我们关注的选项:

  

Sandbox

  * 来自Chromium的沙盒特性，如果开启了这个选项， 渲染进程将运行在沙箱中，限制了大多数系统资源的访问，包括文件读写，新进程启动等， preload.js和网页中的js都会受到这个选项的影响

  * 值得注意的是，这个选项会随着Node Integration的开启而关闭

  * Sandbox选项从Electron 20开始默认为开启状态

  * 如果开启，我们就不能通过chromium v8 Nday RCE也不能通过渲染进程Nodejs共享库直接执行命令

  

Node Integration

  * Node集成，是否开启网页Js Nodej共享库的访问，如果开启的话，网页js将拥有直接Nodejs的执行权限，包括进程启动，文件加载等

  * preload.js Node集成是一直开启的，不受这个选项影响

  * 即使这个选项开启，上下文隔离选项开启的话，网页Js仍然无法访问Nodejs共享库

  

Context Isolation

  * 上下文隔离是Electron的一个特性，使用了与Chromium相同的Content Scripts技术来实现。确保preload脚本和网页js在一个独立的上下文环境中

  * 一旦开启，渲染页面的js中无法将引入Electron和Node的各种模块

  * 如果想在其中使用这部分功能，需要配置preload.js，使用contextBridge来暴露全局接口到渲染页面的脚本中

  * Electron 12开始默认启用

  

 **04  **Electron利用方法

我们对选项的不同组合情况进行讨论， 这里我们为了方便讨论，规定:

  * Node Integration为NI

  * Context Isolation为CISO

  * Sandbox为SBX

  

 **NI为true, CISO为 false，无沙箱**

这种情况是最简单的情况，允许了页面之间访问nodejs共享库，并且没有开启沙箱。只要我们能获取目标应用的一个XSS漏洞，就能直接通过访问NodeJS共享库，升级为XSS漏洞

  

在man.js中webPreferences中配置了nodeIntegration为true,
contextIsolation为false，默认情况下nodeIntegration为true,沙箱就会关闭

![](https://gitee.com/fuli009/images/raw/master/public/20221110215728.png)

  

script直接执行nodejs代码，即可获取shell

![](https://gitee.com/fuli009/images/raw/master/public/20221110215730.png)

  

 **NI为false, CISO为false，SBX为false**

在这种情况下，相比于第一种情况，虽然关闭了Nodejs集成，导致我们不能在web页面上下文访问Nodejs共享库。但是因为上下文隔离没有开启，web页面和preload.js处于同一上下文中，导致我们可以通过污染原型链，获取preload,js的函数，进行ipcmain调用，命令执行等

  

原型链污染获取__webpack_require__

通过如下代码进行原型链污染，可以获取到__webpack_require__函数，进而引入remote/IPC进行进一步利用

    
    
    <script>  
      const origEndWith = String.prototype.endsWith;  
      String.prototype.endsWith = function(...args) {  
        if (args && args[0] === "/electron") {  
          String.prototype.endsWith = origEndWith;  
          return true;  
        }  
        return origEndWith.apply(this, args);  
      }  
      
      const origCallMethod = Function.prototype.call;  
      Function.prototype.call = function(...args){  
        if(args[3] && args[3].name === "__webpack_require__") {  
          window.__webpack_require__ = args[3];  
          Function.prototype.call = origCallMethod;  
        }  
        return origCallMethod.apply(this, args);  
      }  
      console.log(window.__webpack_require__);  
    </script>  
    

  

版本限制

Electron<10

\- 可以使用原型链污染获取remote/IPC模块

\- Remote模块可以直接通过主进程执行node js绕过沙箱

  

Electron 10<version<14

\- 可以使用原型链污染获取remote/IPC模块

\- 需要Remote Module Explicitly Enabled，才可以使用remote模块RCE

\- 主进程IPC存在错误配置，通过进程间通信IPC，进行RCE

  

Electron >14

\- 只能通过原型链污染获取IPC模块

主进程IPC存在错误配置，通过进程间通信IPC，进行RCE

  

利用__webpack_require__进行RCE

无沙箱的情况下，获取openExternal直接执行系统命令

    
    
    window.__webpack_require__('./lib/common/api/shell.ts').default.openExternal('file:///System/Applications/Calculator.app/Contents/MacOS/Calculator')  
      
    window.__webpack_require__("module")._load("child_process").execFile("/System/Applications/Calculator.app/Contents/MacOS/Calculator")  
    

  

有沙箱的情况下,获取ipc，通过主进程的错误配置RCE

    
    
    ipc = __webpack_require__('./lib/renderer/ipc-renderer-internal.ts').ipcRendererInternal  
    a=__webpack_require__('./node modules/process/browser.is')._linkedBinding('electron_renderer_ipc')  
    

  

以下为无沙箱直接使用openExternal RCE的例子

![](https://gitee.com/fuli009/images/raw/master/public/20221110215733.png)

  

 **NI为true/false, CISO为true，SBX为false**

因为没有开启沙箱，通过Chrome渲染进程远程代码执行漏洞，就可以直接RCE。比如Chromium的CVE-2021-21220漏洞， 影响Chromium
83、86、87、88版本，如果electorn内置了Chromium就可以通过XSS，直接攻击，进行RCE。

  

 **NI:false,   CISO:true,  SBX为true**

有沙箱， 我们只能通过IPC进行攻击，但是如果我们js处于iframe之中，可能没有ipc访问权限， 这里分享一些绕过思路。

  

iframe下无ipc接口绕过

默认情况下iframe是没有preload.js暴露的ipc接口的，如果我们获取了一个Iframe的上下文XSS，就不能通过IPC的错误配置RCE。

  

这里提供了一个绕过手法nodeIntegrationInSubFrames选项，代表是否开启iframe环境的预加载脚本执行，如果开启我们就能在Iframe上下文访问contextBridge暴露的接口。

  

这个选项是一个实验选项，默认是关闭状态

![](https://gitee.com/fuli009/images/raw/master/public/20221110215735.png)

  

但是我们可以通过v8 renderer exploit Nday获得的任意内存写的能力，覆盖它为开启

![](https://gitee.com/fuli009/images/raw/master/public/20221110215736.png)

  

从而获得在Iframe上下文访问contextBridge能力，进行RCE。

  

关闭CISO,直接使用IPC，绕过限制

如果在渲染进程的contextBridge存在一些限制，导致我们无法直接执行恶意脚本,比如如下渲染进程的contextBridge限制了openPath的协议只能为http等安全协议，导致无法利用

![](https://gitee.com/fuli009/images/raw/master/public/20221110215738.png)

  

我们可以通过v8 renderer
exploit关闭CISO选项，然后通过原型链污染直接获取ipc，直接使用进程间通信访问main进程的ipcMain监听的方法，绕过contextBridge的限制

![](https://gitee.com/fuli009/images/raw/master/public/20221110215739.png)

  

关闭CISO,使用原型链污染获取remote模块进行RCE

在Electron低版本，一旦关闭了CISO,
我们也可以通过原型链污染泄露remore模块的方式，远程将js代码发送给main进程，通过main.js执行的方式进行RCE。

  

下图是示例的污染方式，获取require函数，引入remote.ts,引入remote模块，执行系统命令

![](https://gitee.com/fuli009/images/raw/master/public/20221110215741.png)

  

 **05  **总结

Electron是一个桌面应用程序的框架,具有方便的跨平台特性，大量的桌面应用都使用Electron开发，如VSCode,
Discord等。如果开发者不注重Electron的安全，开启了一些高危选项，很可能导致安全问题。本文讨论了影响Electron安全的3个比较重要的选项，Sandbox，Context
Isolation，Node Integration。以及这些选项的状态组合之下，对Electron可能具有的攻击方法。希望能起到抛砖引玉的作用。

  

![](https://gitee.com/fuli009/images/raw/master/public/20221110215744.png)

 **绿盟科技天元实验室** 专注于新型实战化攻防对抗技术研究。

研究目标包括：漏洞利用技术、防御绕过技术、攻击隐匿技术、攻击持久化技术等蓝军技术，以及攻击技战术、攻击框架的研究。涵盖Web安全、终端安全、AD安全、云安全等多个技术领域的攻击技术研究，以及工业互联网、车联网等业务场景的攻击技术研究。通过研究攻击对抗技术，从攻击视角提供识别风险的方法和手段，为威胁对抗提供决策支撑。

  

![](https://gitee.com/fuli009/images/raw/master/public/20221110215745.png)

 **M01N Team公众号**

聚焦高级攻防对抗热点技术

绿盟科技蓝军技术研究战队

![](https://gitee.com/fuli009/images/raw/master/public/20221110215746.png)

 **官方攻防交流群**

网络安全一手资讯

攻防技术答疑解惑

扫码加好友即可拉群

  

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

