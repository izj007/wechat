#  Cobalt Strike与YARA：我能拥有你的签名吗？

Endlessparadox  [ Z2O安全攻防 ](javascript:void\(0\);)

**Z2O安全攻防** ![]()

微信号 Z2O_SEC

功能介绍 From zero to one

____

___发表于_

收录于合集 #内网 1个

点击上方[蓝字]，关注我们

 **建议大家把公众号“Z2O安全攻防”设为星标，否则可能就看不到啦！**
因为公众号现在只对常读和星标的公众号才能展示大图推送。操作方法：点击右上角的【...】，然后点击【设为星标】即可。

![]()

# 免责声明

本文仅用于技术讨论与学习，利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者及本公众号不为此承担任何责任。

# 文章正文

译者前言：本文保留的大部分专业术语和工具名称，在不影响阅读的情况下修正了部分语句的语法顺序。相关相关方法在<https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/malleable-c2-extend_pe-
memory-indicators.htm>
这里可以快速查阅了解。原文地址为：<https://www.cobaltstrike.com/blog/cobalt-strike-and-yara-
can-i-have-your-signature/>

在过去的几年里，Beacon的YARA签名大量涌现。我们从与客户的对话中了解到，在使用Cobalt
Strike进行红队活动时，这已经成为问题，而且对于Cobalt Strike的malleable C2 options选项也变得相当迷惑。

因此，这篇博文将概括使用Beacon进行内存YARA扫描时的OPSEC，并建议采用可修改的C2配置文件，有力地规避这些类型的防御技术。

简而言之，要想在面对内存YARA扫描时保证OPSEC安全，你应该：

  * • 确保你正在使用sleep mask，并启用Arsenal kit中的sleep规避（或者使用修改/定制的 sleep mask）

  * • 启用 stage.cleanup

  * • 强烈考虑将stage.obfuscate设置为true（或在UDRL中实现类似的功能）

  * • 在使用post-ex反射DLLs时要保持警惕（取决于现有的安全控制），因为它们目前不能清理内存，因此对签名来说是成熟的（我们正在努力修复）

## YARA 签名

在试图获得特定恶意软件家族的检测时，防御者的快速胜利之一是YARA规则。一旦获得样本，就可以快速生成签名，然后大规模部署。Elastic关于如何实现有一篇优秀博客可以在这里找到：[https://www.elastic.co/blog/detecting-
cobalt-strike-with-memory-signatures。](https://www.elastic.co/blog/detecting-
cobalt-strike-with-memory-signatures%E3%80%82)

通常，这些规则/签名是由目标二进制文件中的字符串或从反转特定功能（如XOR/加密例程等）中识别的静态字节组成。请注意，这些规则也可以很简单，即找到一些任意的字节序列，在不同的版本之间似乎是一致的，例如，当针对Virus
Total运行时（例如，通过VTGrep：content:{4D
5a等...}），可靠地识别目标家族。如果这在不同的版本之间从未改变（操作者不知道），那么它可以提供一个高保真的检测签名。

当与内存扫描结合使用时，这个过程变得更加强大。除非implant（木马）采取规避行动，否则它在内存中就是毫无防备的（即它的.text/.data部分是清晰可见的），好的YARA签名将迅速识别它。在这个场景下Beacon也不例外：

![]()

此外，对于 Beacon 来说，有无数个开源的（无疑还有更多未公开的）YARA规则被EDR供应商/威胁狩猎研究员部署，用于快速扫描和识别
Beacon。因此，这可能使得在红队演练中使用 Cobalt Strike 成为一项挑战，除非采取额外的预防措施。

然而，重要的是要强调，低成本的检测通常也容易被规避。YARA签名通常可以被视为广度广泛但深度有限（即它们相对快速且低成本地生成/自动化，但对于长期检测有效性具有有限的可靠性）。通过对Beacon的可塑C2配置文件进行一些基本调整，大概率以高度自信地绕过YARA扫描。因此，本博客文章的目的是：

  * • 展示 Beacon（在其默认状态下）对 YARA 签名的察觉程度。首先，我们将通过对磁盘上的默认 Beacon 载荷进行静态扫描来进行说明，然后再考虑内存中的 YARA 扫描。

  * • 展示 如何配置Cobalt Strike的可塑性C2选项，使内存YARA扫描彻底变得多此一举。

请注意，本博客主要依赖 Elastic 公开的 Cobalt Strike YARA 规则。这是因为这是我们能找到的最全面的开源 YARA
规则集合（Elastic 在这方面的开放和透明应受到赞赏）。因此，本博客并不意味着是一个关于如何“规避”特定供应商的指南，同时也强调内存中的 YARA
扫描很可能只是 EDR（终端检测和响应）所采用的深度防御策略的一个组成部分（例如与扫描无后备内存、异常线程、进程注入等相结合）。

最后，需要注意的是，由于本博客关注内存中的YARA扫描，所有给出的示例都是针对原始的Beacon
DLL载荷。因此，在本文中，我们假设Beacon已经通过某种形式的Stage0 shellcode runner注入到内存中。Cobalt
Strike生成的其他载荷（例如默认的Cobalt Strike可执行文件，它们是简单的shellcode
runner）已经包含在产品中多年，因此也会广泛地被签名。然而，由于Arsenal工具包的目的是帮助修改这些可执行文件以绕过签名，它们超出了本博客文章的范围。

## 磁盘上的YARA扫描

为了展示YARA签名的威力，我们可以使用Elastic的开源规则针对默认的原始HTTP Beacon
DLL（存储在磁盘上）进行扫描。需要注意的是，这个场景有些刻意，因为通常当一个exe/DLL被写入磁盘（或执行）时，EDR会尝试通过PE恶意软件模型提取特征并对二进制进行分类（尽管也可以使用YARA）。这是一个完全不同的问题，并且超出了本博客文章的范围，但可以参考相关资料进行进一步阅读。然而，这个第一个示例主要旨在了解Beacon在YARA签名方面的足迹，然后我们再考虑内存中的YARA扫描用例。

我们可以使用以下命令将Elastic Cobalt Strike YARA规则对默认的原始Beacon DLL进行扫描：

    
    
    yara64.exe --print-strings Windows_Trojan_CobaltStrike.yar default_raw_beacon.dll

这产生了五个结果：

  1. 1\. 在Beacon.dll中发现的默认字符串，如下所示（相应的YARA规则是Windows_Trojan_CobaltStrike_ee756db7）：

    
    
    Windows_Trojan_CobaltStrike_ee756db7 beacon_default_raw.dll  
      
    0x2cd60:$a39: %s as %s\%s: %d  
    0x3c012:$a41: beacon.x64.dll   
    0x2df70:$a46: %s (admin)  
    0x2cec0:$a48: %s%s: %s   
    0x2cd8c:$a50: %02d/%02d/%02d %02d:%02d:%02d   
    0x2cdb8:$a50: %02d/%02d/%02d %02d:%02d:%02d  
    0x2dfb9:$a51: Content-Length: %d

如上所示,字符串通常存储在二进制文件的.data/.rdata部分，并且清楚地显示了Beacon独有的数字.

  1. 1\. 在Beacon的.text部分发现了一个 "未识别 "的代码片段（相应的YARA规则是Windows_Trojan_CobaltStrike_663fc95d）：

    
    
    Windows_Trojan_CobaltStrike_663fc95d beacon_default_raw.dll  
      
    0x195f8:$a: 48 89 5C 24 08 57 48 83 EC 20 48 8B 59 10 48 8B F9 48 8B 49 08 FF 17 33 D2 41 B8 00 80 00 00

  1. 1\. Beacon的默认睡眠屏蔽程序的代码片段（相应的YARA规则是Windows_Trojan_CobaltStrike_b54b94ac）：

    
    
    Windows_Trojan_CobaltStrike_b54b94ac beacon_default_raw.dll   
    0x3c37b:$a_x64: 4C 8B 53 08 45 8B 0A 45 8B 5A 04 4D 8D 52 08 45 85 C9 75 05 45 85 DB 74 33 45 3B CB 73 E6 49 8B F9 4C 8B 03

请注意，这个Beacon有效载荷实际上没有启用睡眠掩码，但是默认的睡眠掩码总是在.data部分打上补丁，除非指定一个自定义的。

  1. 1\. 在Beacon默认导出的反射加载器函数中发现了一个代码片段（对应的YARA规则是Windows_Trojan_CobaltStrike_f0b627fc）：

    
    
    Windows_Trojan_CobaltStrike_f0b627fc beacon_default_raw.dll 0x16ed2:$beacon_loader_x64: 25 FF FF FF 00 3D 41 41 41 00 75 1A 8B 44 24 78 25 FF FF FF 00 3D 42 42 42 00 75   
    0x18183:$beacon_loader_x64: 25 FF FF FF 00 3D 41 41 41 00 75 1A 8B 44 24 78 25 FF FF FF 00 3D 42 42 42 00 75

  1. 1\. Beacon覆盖DOS头部的反射加载器shellcode stub（相应的YARA规则是Windows_Trojan_CobaltStrike_1787eef5）：

    
    
    Windows_Trojan_CobaltStrike_1787eef5 beacon_default_raw.dll   
    0x0:$a5: 4D 5A 41 52 55 48 89 E5 48 81 EC 20 00 00 00 48 8D 1D EA FF FF FF 48 89 DF 48 81 C3 3C 6E 01 00

对于最后两个命中（4和5），重要的是要了解Beacon是一个反射加载的DLL。这意味着它在其.text节中具有一个导出的反射加载器函数。此外，为了能够直接从内存中运行，一个小的shellcode
stub被写入DOS头的开头。这个小的shellcode
stub负责跳转到导出的反射加载器函数，然后继续将Beacon引导到内存中。有关反射加载的更多背景信息，请参阅我们关于UDRL开发的博客系列以及@0xBoku的博客文章《Defining
the Cobalt Strike Reflective Loader》。

<https://www.cobaltstrike.com/blog/revisiting-the-udrl-part-1-simplifying-
development/>  
<https://securityintelligence.com/posts/defining-cobalt-strike-reflective-
loader/>

毫不奇怪，这两个组件对于防御者来说非常具有吸引力（并且在过去的几年中一直保持不变）。上面的第四个结果针对的是默认的反射加载器函数中的任意代码。下面是IDA中ReflectiveLoader()函数中的有问题的代码（25FFFFFF00解析为`and
eax, 0xFFFFFF`等）：

![]()

以下是Google提供的类似YARA规则的示例，值得注意的是，Windows
Defender也包含了对Beacon默认反射加载器的签名。我们可以通过对相同的原始载荷运行ThreatCheck来证明这一点：

![]()

如果我们在IDA中扫描这个字节序列，我们发现它是位于ReflectiveLoader函数中的代码：

![]()

上述的第五个结果（Windows_Trojan_CobaltStrike_1787eef5）是用于Beacon默认的引导反射加载的shellcode
stub的签名。下面是PE-Bear中显示的Beacon默认的shellcode stub：

![]()

Windows_Trojan_CobaltStrike_1787eef5规则具有一个字符串条件（$a5），该条件是对此shellcode的前8个字节（`{
4D 5A 41 52 55 48 89 E5 48 81 EC ?? ?? ?? ?? 48 8D 1D ?? ?? ?? ?? 48 89 DF 48
81 C3 ?? ?? ?? ?? }`）进行模糊匹配。

## 内存中的YARA扫描

在查看了Beacon对于位于磁盘上的默认原始DLL载荷对YARA暴露之后，我们现在可以转向这篇博客关注的问题：内存中的YARA扫描。与内存中的YARA扫描相关的关键是，当Beacon被反射加载到内存中时，会导致两个内存分配：原始的Beacon
DLL（实际上会执行shellcode引导程序和反射加载器函数）以及虚拟的Beacon DLL（在内存中正确加载并准备就绪）。因此，原始的Beacon
DLL 实际上会通过反射加载器函数为虚拟的Beacon DLL 分配内存，并确保其正确加载。下面的图像演示了这一过程：

![]()

Cobalt Strike的一些可塑C2选项会对原始的Beacon
DLL进行覆盖/修改（例如stage.magic_mz_x64），而其他一些选项会改变反射加载器的行为/Beacon加载到内存中的方式（例如stage.obfuscate不会复制DLL头信息到虚拟的Beacon
DLL中，stage.stomp_pe会在虚拟的Beacon
DLL头中篡改数值等等）。需要强调的重要一点是，根据设置的可塑C2选项不同，这两个内存分配可能会触发不同的YARA签名，因此需要同时考虑这两个方面。

从上面的图表可以清楚地看出，一旦将Beacon注入到内存中，它在默认状态下就会变得毫无防备。它的反射加载器存根、代码（.text部分）和字符串（.rdata/data）都是明显可见的。下面是Windbg的截图，显示了属于Beacon的字符串明文存在于与虚拟Beacon
DLL（RWX）对应的内存区域中：

![]()

（看到beacon.x64了吗？这种技术就是yara扫描发现的）

此外，由于原始DLL和虚拟DLL对应的两个内存分配，所有这些敏感区域实际上在内存中存储了两次。因此，如果我们对一个已注入默认Beacon的进程运行相同的YARA规则，我们将会得到与之前相同的YARA匹配结果，只是这一次会有重复的结果。这些重复的结果将对应于在原始Beacon
DLL和虚拟Beacon DLL中找到的相同字符串/字节。

下面的Windbg截图显示了由于将Beacon注入到内存中而产生的两个可疑内存区域（对应原始的Beacon DLL和虚拟的Beacon
DLL）。这两个区域都可以通过相同的DLL头部（MZARUH..）进行识别，这是默认反射加载器存根的起始位置。

![]()

下面的图像显示了针对该进程内存地址空间（PID
3204）运行YARA的输出结果。请注意，我们对每个规则都得到了两个匹配结果，对应于在Windbg中上面突出显示的两个内存区域：

![]()

最后，一旦Beacon启动并运行，也要注意它的运行时间行为会增加其内存占用。BeaconEye是一个很好的例子来证明这一点。它使用以下签名（https://github.com/CCob/BeaconEye/blob/master/BeaconEye.cs#L35），通过扫描堆内存中的配置来识别Beacon。

####  **Stage.transform.strrep**

作为一种基本的“深度逃避”方法，删除在Beacon的反射式DLL中发现的明显可疑的字符串（例如“beacon.x64.dll”，“ReflectiveLoader”等）是明智的。我们可以使用strrep命令来替换Beacon的反射式DLL中的字符串/字节。但请注意，上面由YARA识别出的一些字符串是Beacon的格式字符串（“%s%s:
%s”），目前需要返回数据，因此替换这些字符串可能会导致未预料的行为。

为了证明YARA签名是多么的脆弱，我们可以对我们可塑性强的C2配置文件做一些小的修改，以绕过Windows_Trojan_CobaltStrike_ee756db7规则，该规则用于识别Beacon中的默认字符串。这个规则需要识别六个可疑的字符串，正如下面的摘录所示：

    
    
    [...]   
    $a50 = "%02d/%02d/%02d %02d:%02d:%02d" ascii fullword   
    $a51 = "Content-Length: %d" ascii fullword condition:  
    6 of ($a*)

通过对我们的malleable C2配置文件做以下修改，我们可以绕过这个规则：

    
    
    stage {   
    Transform-x64 {   
               strrep "(admin)" "(adm)";   
               # If you modify these ensure you keep format string order  
                strrep "%s as %s\\%s: %d" "%s - %s\\%s: %d";   
                }   
        }

一般而言，我们无法修改所有这些默认字符串而不引发问题，对于可疑字符串的真正解决方案是使用"sleep mask kit"（我们将很快讨论）。

#### * Stage.magic mz / Stage.magic pe *

"magic_mz ***"和"magic_pe**
*"选项分别修补了Beacon的原始DLL中的MZ和PE字符。导出的反射加载器函数将使用这些魔术字节在内存中定位自身，因此这个选项也会修改反射加载器。请注意，由于这些位于DLL头部，它们将在反射加载过程中复制到虚拟的Beacon
DLL中。

需要注意的是，对于magic mz
*选项，提供的值必须是有效的（无）操作码，因为它们是作为shellcode存根的一部分将被执行的第一条指令。通常情况下，这将是一些`pop regA,
push regA`的变种，因为后面的指令会撤消前面的指令。但请参考相关指南以获取有关配置此选项的更多指导。

这些选项通常用于阻碍试图识别被注入的DLL的内存扫描器，然而，magic_mz选项可能会破坏对反射加载器存根的基本YARA签名。例如，修改MZ字节（4D
5A）将破坏此签名。然而，我们的操作自由度有限，因为我们每次只能修改几个字节，因此更强大的YARA签名仍然会触发。

####  **Stage.cleanup**

这是一个关键的选项，在任何可能的情况下都应该设置为true。正如我们之前演示过的那样，原始Beacon
DLL的初始内存分配包含了几个高度可签名的组件（例如反射加载器存根、导出的反射加载器函数等）。此外，一旦Beacon启动完成，这些组件就不再需要了。通过将"cleanup"选项设置为true，Beacon将释放这些内存，大幅降低其在内存中的占用空间（因此只剩下虚拟Beacon
DLL）。此外，清理操作足够智能，可以识别初始内存分配的方式并相应地释放它们。

请注意，如果由于某种原因您无法使用cleanup选项，@S4ntiagoP的这个BOFhttps://github.com/S4ntiagoP/freeBokuLoader示例演示了如何手动清理内存。

####  **Stage.stomppe**

即使我们将cleanup选项设置为true，DLL头部仍然会被复制到虚拟Beacon
DLL中，从而成为内存扫描的目标。stomppe选项将确保在反射加载过程中在虚拟Beacon
DLL中篡改MZ/PE和e_lfanew值。这意味着内存扫描器再次更难识别内存中是否存在注入的DLL，但也可能有助于破坏针对Beacon
DOS头的任何YARA签名（尽管该选项与magic_mz/pe类似，在YARA使用情况下受到一定限制）。

####  **Stage.obfuscate**

我们可以更进一步，要求反射加载器在复制Beacon时不包含DLL头部。这可以通过使用"stage.obfuscate"标志来实现。启用此选项，并配合cleanup选项，意味着反射加载器存根无法在内存中被找到。

此外，stage.obfuscate也会屏蔽Beacon的

  * • .text section

  * • Section names

  * • 导入表

  * • Dos/Rich Header（这在技术上没有被屏蔽，而是用随机数据覆盖）。

从4.8版本开始，stage.obfuscate从一个固定的单字节XOR密钥转移到一个随机生成的多字节XOR密钥。从规避的角度来看，这一点特别有用，因为YARA包含一个XOR修改器，可以对单字节XOR的字符串进行暴力破解。作为一个例子，这里是一个YARA规则，它在Beacon中寻找XOR的字符串。

Stage.obfuscate在下面PE-
Bear的截图中得到了证明，它比较了一个默认的Beacon和一个启用了混淆功能的Beacon（注意不同的DOS头和被屏蔽/XOR的部分名称）：

![]()

在反射加载过程中，必需的部分将解除掩码。因此，这将破坏一些针对原始/静态Beacon
DLL的YARA签名，但不会在它被加载到内存中后产生影响（例如，对.text节中的代码片段的规则仍将匹配虚拟Beacon DLL）。下面是一个简要的示例：

![]()

还要注意的是，正如这张图所示，即使启用了混淆处理，在原始Beacon DLL中仍有一些东西没有被掩盖：

  * • Reflective loader stub 反射式加载器存根

  * • Exported reflective loader function 导出的反射式加载器函数

  * • Sleep mask 睡眠掩码

  * • Strings 字符串

因此，即使是被混淆的Beacon有效载荷仍然会触发之前为这些各自组件确定的相同的YARA规则。解决这一限制的方法之一是通过在UDRL中实施自定义混淆处理。这个话题将在我们的UDRL开发博客系列的下一部分中更详细地介绍。

最后，请记住，应用于Beacon的混淆可能会使它看起来（更加）可疑，如果被丢弃到磁盘上的PE恶意软件模型（即粗略的部分名称，某些部分的更高熵，等等，都会使它看起来更加异常）。

### 修改反射性加载器引导程序

现在应该很清楚，默认的Cobalt
Strike反射式加载器存根是YARA签名的一个明显目标。然而，我们可以创建自己的DOS引导程序，并通过`stage.transform`块将其应用于Beacon的有效载荷。注意，这个例子只考虑X64有效载荷。

Beacon使用的默认64位DOS存根显示在下面（以及解释）：

    
    
    pop r10 ; MZ Header  
    push r10 ; undo action above  
    push rbp ; save the stack base pointer   
    mov rbp, rsp ; create a new stack frame   
    sub rsp, 0x20 ; create shadow space (x64 __fastcall)  
    lea rbx, [rip - 0x16] ; obtain shellcode base address   
    mov rdi, rbx ; save shellcode base address  
    add rbx, 0x16EA4 ; add file offset of ReflectiveLoader to shellcode base  
    call rbx ; call ReflectiveLoader (returns DllMain address)   
    mov r8d, 0X56A2B5F0 ; EXITFUNC value   
    push 4 ; push 4 to stack   
    pop rdx ; pop the value into rdx (second argument of call)   
    mov rcx, rdi ; move shellcode base address to rcx (first argument of call)  
    call rax ; call DllMain

下面的例子是上述stub的另一个版本

    
    
    pop r10 ; MZ Header  
    lea rbx, [rip -0x08] ; obtain the shellcode base address  
    push r10 ; undo action of MZ header  
    sub rsp, 0x28 ; create shadow space and align stack  
    mov rdi, rbx ; save shellcode base address  
    add rbx, 0xb752 ; add half file offset of ReflectiveLoader to shellcode base  
    add rbx, 0xb752 ; add half file offset of ReflectiveLoader to shellcode base   
    call rbx ; call ReflectiveLoader (returns DllMain address)  
    mov rdx, 0x04 ; move 4 into rdx (second argument of call)  
    mov rcx, rdi ; move shellcode base address to rcx (first argument of call)  
    call rax ; call DllMain

注意，默认的ReflectiveLoader实际上会调用DllMain本身，并将DllMain的地址返回给shellcode
stub。然后我们第二次调用DllMain来启动Beacon。这就是为什么你可能会在开源的UDRL中看到对DllMain的两次调用。

这个stub执行类似的步骤，但有一些指令替换，而且顺序略有不同（除了设置EXITFUNC值，实际上技术上不需要）。此外，还有许多其他使用指令替换的方法来达到类似的效果。例如，每条
"mov regA, regB "可以被替换为 "push regB; pop regA "或 "mov tmp, regB; mov regA, tmp
"等。另外，一个更自动化的方法是使用类似Nettitude的Shellcode
Mutator的东西。这实际上并不改变指令本身（它随机地在现有的shellcode中加入no-ops），但它也可以用来破坏YARA的签名。

在下面的截图中，我们使用defuse.ca来组装上面的备用shellcode存根：

![]()

请注意，由于复杂的指令编码原因，`pop r10` 实际上会组装成 41
5A，因此您需要手动将字符串字面值开头的`“\x41\x5A”`改回为`“\x4D\x5A”`（4D 5A 仍会被反汇编为 `pop
r10`）。这是因为默认的反射加载器向后搜索 MZ 头部（即 4D 5A），如果找不到它，加载器将会失败。

可以修改默认的DOS存根，并在可塑性C2配置文件中使用我们的更新版本与strrep，如下图所示：

    
    
    stage {    
           transform-x64 {  
                  strrep "\x4D\x5A\x41\x52\x55\x48\x89\xE5\x48\x81\xEC\x20\x00\x00\x00\x48\x8D\x1D\xEA\xFF\xFF\xFF\x48\x89\xDF\x48\x81\xC3\xA4\x6E\x01\x00\xFF\xD3\x41\xB8\xF0\xB5\xA2\x56\x68\x04\x00\x00\x00\x5A\x48\x89\xF9\xFF\xD0" "\x4D\x5A\x48\x8D\x1D\xF8\xFF\xFF\xFF\x41\x52\x48\x83\xEC\x28\x48\x89\xDF\x48\x81\xC3\x52\xB7\x00\x00\x48\x81\xC3\x52\xB7\x00\x00\xFF\xD3\x48\xC7\xC2\x04\x00\x00\x00\x48\x89\xF9\xFF\xD0";  
                        }  
    }

在重启Teamserver并在配置文件中加入上述strrep后，我们可以看到新的DOS存根已经应用于我们导出的Beacon有效载荷：

![]()

在下面的截图中，我们使用udrl.py（来自udrl-
vs工具包）将带有修改过的DOS头的原始Beacon有效载荷（即beacon.bin）注入内存，以测试它是否如预期那样工作：

![]()

请注意，为了与默认的反射加载器兼容，反射加载器存根需要执行以下四个操作：

  1. 1\. 获得shellcode的基本地址（即MZ头的开始）。

  2. 2\. 计算从shellcode基址到导出的ReflectiveLoader函数的偏移量。

  3. 3\. 调用ReflectiveLoader()。

  4. 4\. 使用shellcode的基址作为hinstDLL参数（第一个参数/rcx），并使用4作为原因代码参数（第二个参数/rdx），调用DllMain函数。请注意，第三个参数不是必需的。

只要按照这些步骤进行操作，您可以添加任何您需要的代码来启动和运行Beacon。但是，请尽量限制shellcode存根的大小为59字节，否则可能会导致Beacon崩溃。这是因为在这一点之后，您将覆盖e_lfanew的值（它位于DOS头部的0x3C/60位置）。反射加载器需要e_lfanew的值来向后搜索内存中的MZ和PE头部，如果破坏了它，加载器将无法正常工作。

最后，由于magic_mz选项还会修改DOS stub的开头（从而改变反射加载器寻找的字节），它与本文档中概述的strrep方法不兼容，并且会取代该方法。

### Sleep Mask 睡眠掩码

即使进行了上述所有修改，Beacon仍然容易受到针对.text/.data节的YARA规则的攻击，因为这些内容在内存中仍然是明文可见的。此外，Beacon的运行时数据（如堆内存）也面临类似的风险。

Cobalt Strike中解决这个问题的方法是使用"sleep
mask"（睡眠掩码）。其概念很简单：在Beacon进入休眠之前，它会对自身和相关内存（例如堆内存）进行掩码操作。当Beacon进行checkin时，它的内存会暂时暴露出来，但大部分时间内存会被掩码，因此任何有效的签名都无法找到它们的目标。这是需要配置的关键"Malleable
C2"选项之一。它将确保Beacon在内存中只可见一个极短的时间窗口，并为防止内存中的签名攻击提供最强大的防御。

在4.7之前，睡眠掩码只在RWX的情况下才会掩码.text部分。因此，如果stage.userwx被设置为false，Beacon的.text部分将驻留在RX内存中，不会被屏蔽。因此，.text部分有可能被之前讨论的许多YARA规则发现，所以这并不理想。

从4.7版本开始，当stage.userwx被设置为false时，我们可以配置睡眠屏蔽套件来屏蔽.text部分。当这一点被启用时，睡眠掩码将把.text保护改为RW，掩码部分，睡眠，解除掩码部分，然后把.text保护改回RX。

    
    
    set userwx "false";   
    set sleepmask "true";

在设置这些值并重新启动TeamServer后，运行在睡眠屏蔽套件中发现的构建脚本。从的Arsenal
Kit最新版本（20230315）来看，这是通过以下（示例）命令进行的：

    
    
    ./build.sh 47 WaitForSingleObject true indirect /tmp/dst

要了解每个参数的完整解释，您可以在不带任何参数的情况下运行`./build.sh`。然而，与本讨论相关的关键参数是将第三个参数（Mask_text）设置为“true”。重新编译后，将随后的.cna脚本加载到脚本管理器中。如需进一步指导，请参阅休眠掩码工具包中的README文件。

请注意，在旧版本的休眠掩码工具包中，您需要在sleepmask.c文件中将MASK_TEXT_SECTION设置为1，示例如下所示：

    
    
    /* Enable or Disable sleep mask capabilities */   
    #define MASK_TEXT_SECTION 1

### 规避式休眠掩码

到此为止，Beacon将避免使用可读写执行（RWX）内存，并且不会复制其DLL头部，在休眠时会进行掩码处理。然而，这种方法中仍存在一个薄弱环节：休眠掩码本身仍然存在于内存中并暴露出来。以下是对此问题的高层次说明：

![]()

因此，默认的休眠掩码对防御者来说仍然是一个非常诱人的目标，毫不奇怪地有很多规则专注于这个保留在内存中的足迹。例如，启用清理和休眠掩码后，对于默认休眠掩码的Windows_Trojan_CobaltStrike_b54b94ac规则将在Beacon休眠时触发，如下所示：

![]()

这个屏幕截图显示了正在休眠的Beacon线程（线程ID：900）的调用堆栈，以及针对该进程（进程ID：5944）运行相同YARA规则的结果。现在，Beacon被屏蔽了，我们之前识别出的YARA规则将不再触发，但是我们确实看到了默认的睡眠屏蔽（Windows_Trojan_CobaltStrike_b54b94ac）的预期命中。请注意，我们可以看到这个结果是在与睡眠线程调用堆栈中突出显示的无后备返回地址相同的内存区域中找到的（~0x1b4ef8c00d2）。因此，即使你使用的是默认的睡眠屏蔽，你仍然有可能被内存中的YARA扫描轻松识别出来。

显然，我们在Beacon休眠时还需要对睡眠屏蔽内存进行混淆。我们可以使用Arsenal套件中的规避性睡眠屏蔽来实现这一点。这将在休眠时使用外部机制对睡眠屏蔽进行混淆，从而破坏任何对其自身的签名，如下图所示：

![]()

这可以通过在 sleepmask.c 中设置以下内容来配置：

    
    
    #if _WIN64   
    #define EVASIVE_SLEEP 1   
    #endif

还要注意，为了在使用规避性睡眠时（特别是在进行进程注入时）避免出现任何问题，请确保通过修改evasive_sleep.c文件来启用CFG（Control
Flow Guard）绕过功能，修改如下：

    
    
    /* * Enable the CFG bypass technique which is needed to inject into processes * protected Control Flow Guard (CFG) on supported version of Windows. */   
      
    #define CFG_BYPASS 1

完成这两个步骤后，您可以再次重新构建和重新加载.cna脚本，以使更改生效。

另外，您还可以修改默认的睡眠屏蔽并重新编译它，以打破任何静态字节模式，或者应用自己定义的用户睡眠屏蔽（UDSM）。一般来说，对于睡眠屏蔽和反射式加载（参见下面的UDRLs部分），自定义是关键，并且使用自定义/未知代码显然会完全破坏预先制作的YARA签名。

### UDRLs

本文示范了Beacon的默认反射加载器函数存在许多签名。通过启用睡眠掩码、规避性睡眠和清理内存功能，
默认的反射加载器问题较小，因为我们在内存中的曝光极为有限。然而，为了避免默认的YARA签名，您可以考虑使用自定义的UDRLs。我们自己的博客系列提供了如何开发UDRL的指南，而且还有许多优秀的开源UDRL，如BokuLdr、[TitanLdr](https://github.com/realoriginal/titanldr-
ng)和AceLdr。

请注意，如果您使用自定义UDRL，上述提到的许多可塑的C2选项将被忽略。这是因为您修改/混淆Beacon的方式与反射式加载器的工作方式紧密相关。例如，如果您混淆了一个部分，您的反射式加载器需要知道如何对其进行解混淆。因此，将这些细节留给UDRL开发人员来实现是有意义的。

### Post-Ex Reflective DLLs

到目前为止，所提出的方法存在一个需要注意的地方，即与后渗透反射式DLL相关。需要注意的是，并非所有后渗透功能都作为反射式DLL运行，其中许多是以BOF（Beacon
Object File）的形式实现的。详细指南请参阅以下文档。

如果我们对一个已经注入Cobalt Strike键盘记录器的进程运行相同的YARA规则集，我们会得到两个命中结果：

  1. 1. 在键盘记录器后渗透DLL中发现的默认字符串(对应的YARA规则是 Windows_Trojan_CobaltStrike_0b58325e)：0x14767a91d42:$a2: keylogger.x64.dll

    
    
    0x14767cb2b42:$a2: keylogger.x64.dll  
    0x14767a8b568:$a4: %cE=======%c  
    0x14767cac368:$a4: %cE=======%c  
    0x14767a8b908:$a5: [unknown: %02X]  
    0x14767cac708:$a5: [unknown: %02X]  
    0x14767a91d54:$b1: ReflectiveLoader  
    0x14767cb2b54:$b1: ReflectiveLoader  
    0x14767a8b578:$b2: %c2%s%c  
    0x14767cac378:$b2: %c2%s%c  
    0x14767a8b8c0:$b3: [numlock]  
    0x14767cac6c0:$b3: [numlock]  
    0x14767a8b562:$b4: %cC%s  
    0x14767cac362:$b4: %cC%s  
    0x14767a8b658:$b5: [backspace]  
    0x14767cac458:$b5: [backspace]  
    0x14767a8b8d0:$b6: [scroll lock]  
    0x14767cac6d0:$b6: [scroll lock]  
    0x14767a8b688:$b7: [control]  
    0x14767cac488:$b7: [control]  
    0x14767a8b6f4:$b8: [left]  
    0x14767cac4f4:$b8: [left]  
    0x14767a8b6c8:$b9: [page up]  
    0x14767cac4c8:$b9: [page up]  
    0x14767a8b6d8:$b10: [page down]  
    0x14767cac4d8:$b10: [page down]  
    0x14767a8b718:$b11: [prtscr]  
    0x14767cac518:$b11: [prtscr]  
    0x14767a8b8e0:$b13: [ctrl]  
    0x14767cac6e0:$b13: [ctrl]  
    0x14767a8b6ec:$b14: [home]  
    0x14767cac4ec:$b14: [home]  
    0x14767a8b6a0:$b15: [pause]  
    0x14767cac4a0:$b15: [pause]  
    0x14767a8b670:$b16: [clear]  
    0x14767cac470:$b16: [clear]

  1. 1\. 反射式加载器存根（对应的YARA规则是Windows_Trojan_CobaltStrike_29374056）：

    
    
    Windows_Trojan_CobaltStrike_29374056 9540  
    0x14767a80000:$a1: 4D 5A 41 52 55 48 89 E5 48 81 EC 20 00 00 00 48 8D 1D EA FF FF FF 48 81 C3 10 19 00 00 FF D3   
    0x14767ca0000:$a1: 4D 5A 41 52 55 48 89 E5 48 81 EC 20 00 00 00 48 8D 1D EA FF FF FF 48 81 C3 10 19 00 00 FF D3

又一次，我们得到了重复的结果，这些结果对应于在原始的后渗透DLL和虚拟的后渗透DLL上都触发了相同的规则。

对于post-ex DLLs，我们可以使用的主要可塑的C2选项是将'post-ex.obfuscate'设置为true。该选项将会：

  * • 静态屏蔽rdata/data部分

  * • 清除模块/函数名称字符串

  * • 在运行时动态屏蔽长时间运行任务的rdata节（这种行为在不同的后渗透DLL之间可能会有所不同）

  * • 在反射加载期间不复制DLL头

  * • 避免RWX内存（因此，如果不设置该选项，post-ex DLL将默认使用RWX内存）

因此，在启用post-
ex.obfuscate之后，我们只得到了一个命中结果，即来自原始后渗透DLL的反射式加载器存根（Windows_Trojan_CobaltStrike_29374056）。这是因为DLL头部没有复制到虚拟后渗透DLL中，并且之前识别的字符串现在在原始和虚拟DLL中都被屏蔽了。

![]()

此外，一旦键盘记录器任务被终止，我们仍然会得到这个结果。实际上，两个内存分配并没有被清理，它们将一直留在内存中，直到进程终止。请注意，obfuscate标志意味着在后渗透DLL线程退出时，rdata中的字符串将被清除，但内存并没有被释放。

 **因此，post-ex反射式DLL不会正确清理内存，因此确实存在轻微的YARA签名的风险**  。需要注意的是，不同的post-
ex反射式动态链接库在行为上存在差异，这里就不详细介绍了，但关键的一点是，目前所有的动态链接库都存在这种限制。

如果您使用fork并运行（可能会产生噪音），那么这不会构成风险，但如果在进程内部使用或注入到另一个进程中进行长时间运行的任务中，则会构成风险。因此，在使用后渗透反射式DLL时要保持警惕，即使启用了混淆功能，如果您了解安全控制中涉及内存中的YARA扫描，并且如果可能的话，优先选择BOF（Beacon
Object File）等等效方法。我们计划在4.9版本中对后渗透DLL的这个限制进行全面改进。

### 结论

通过本文中提出的建议方法，Beacon现在对内存中的YARA扫描具有很强的抵抗力。它将在内存中被屏蔽，除了极短暂的检查时间之外，当它可见时，我们已经采取了尽可能多的预防措施来限制其暴露。

我们建议的可塑性C2配置文件如下所示：

    
    
    stage {   
            set userwx "false";    
            set cleanup "true";   
            set obfuscate "true";   
            set magic_mz_x64 "<CHANGEME>";   
            set magic_pe "<CHANGEME>";   
            # Alternatively, modify the DOS header via the   
            # transform.strrep approach outlined previously.   
            # For sleep mask ensure you:   
            # - Enable masking the text section (set Mask_text to true # for./build.sh or for older sleep mask kits set   
            # MASK_TEXT_SECTION to 1 in sleepmask.c).   
            # - Enable evasive sleep (#define EVASIVE SLEEP 1 in sleepmask.c).  
            # - Enable CFG bypass (#define CFG_BYPASS 1 in evasive_sleep.c).   
            # - Ensure sleep mask is recompiled after setting the above   
            # and that the .cna script is loaded into the Script Manager. set sleep_mask "true";   
            # Remove default strings found in Beacon.   
            transform-x64 {   
              strrep "ReflectiveLoader" "<CHANGEME>";   
              strrep "beacon.x64.dll" "<CHANGEME>";   
              strrep "(admin)" "(adm)";   
              etc.   
            }   
    }

请注意，这是一个用于绕过YARA签名的建议配置文件（可能还有其他许多安全控制措施，并且根据背景，可能使用其中的某些选项不是很理想）。

在这一点上，防御者的下一道防线要么是采用传统的内存扫描方法查找注入代码，要么是查找在睡眠线程中内存中未被屏蔽的的部分(这种检测技术的示例请参见https://github.com/thefLink/Hunt-
Sleeping-
Beacons)。要解决这两个问题比之前的解决难度更加复杂。此外，在4.8版本中，您可以启用堆栈欺骗来绕过后者。只需修改sleepmask.c如下所示，并重新编译/加载生成的.cna脚本，即可再次启用堆栈欺骗：

    
    
    #if EVASIVE_SLEEP  
    // #include "evasive_sleep.c"  
    #include "evasive_sleep_stack_spoof.c"  
    #endif

这包含了一个默认的堆栈欺骗，但还是建议用户自定义。欲了解更多信息，请参阅Arsenal kit中的README。

作为最后的说明，本博文中的分析结果将用于我们计划在下一个Cobalt
Strike发布版本中进行的更改。我们希望为用户提供更多关于Beacon和后渗透DLL的反射式加载过程的控制，并使用户能够轻松对抗YARA签名。

    
    
    https://xz.aliyun.com/t/12701  
    author:Endlessparadox

  

# 技术交流

### 知识星球

致力于红蓝对抗，实战攻防，星球不定时更新内外网攻防渗透技巧，以及最新学习研究成果等。常态化更新最新安全动态。专题更新奇技淫巧小Tips及实战案例。

涉及方向包括Web渗透、免杀绕过、内网攻防、代码审计、应急响应、云安全。星球中已发布 300+
安全资源，针对网络安全成员的普遍水平，并为星友提供了教程、工具、POC&EXP以及各种学习笔记等等。([点我了解详情](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247496923&idx=1&sn=523f413df2d53de64fd48fbb081be790&chksm=ceab1f9bf9dc968da974eec5e34003d8a937884d2732fda7fac190360fd2a1a5290db090d8e9&scene=21#wechat_redirect))

![]()

### 学习圈子

一个引导大家一起成长，系统化学习的圈子。如果看到这里的师傅是基础不够扎实/技术不够全面/入行安全不久/有充足时间的初学者...其中之一，那么欢迎加入我们的圈子。在圈子内会每周规划学习任务，提供资料、技术文档，供大家一起学习、交流，由浅入深、层层递进。（[点我了解详情](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247498684&idx=1&sn=71c923f29d6431f6271398d435f96ae0&chksm=ceab10fcf9dc99ea174325e6982af84d0331aeee2cb0196ac11405acd44283c18da31e77d553&scene=21#wechat_redirect)）

![]()

 ****

### 交流群

关注公众号回复“ **加群** ”，添加Z2OBot好友，自动拉你加入 **Z2O安全攻防交流群(微信群)** 分享更多好东西。
**（QQ群可直接扫码添加）**

![]()

![]()

### 关注我们

 **关注福利：**

 **回复“** **app** **" 获取   app渗透和app抓包教程**

 **回复“** **渗透字典** **" 获取 针对一些字典重新划分处理，收集了几个密码管理字典生成器用来扩展更多字典的仓库。**

 **回复“** **书籍** **"  获取 网络安全相关经典书籍电子版pdf**

 ** **回复“ 资料** **"  获取 网络安全、渗透测试相关资料文档****

 ** **  
****

点个【 在看 】，你最好看

  

  

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

