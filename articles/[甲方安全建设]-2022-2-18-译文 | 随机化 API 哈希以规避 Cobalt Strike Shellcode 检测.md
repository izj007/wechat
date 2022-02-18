#  译文 | 随机化 API 哈希以规避 Cobalt Strike Shellcode 检测

原创 bggsec [ 甲方安全建设 ](javascript:void\(0\);)

**甲方安全建设** ![]()

微信号 blueteams

功能介绍 甲方安全建设的点滴，共同学习，一起进步。 笔耕不辍也是对自我的督促。

____

__

收录于话题

#译文分享 9 个

#蓝队 7 个

#红队 7 个

![](https://gitee.com/fuli009/images/raw/master/public/20220218094445.png)

开卷有益 · 不求甚解

![](https://gitee.com/fuli009/images/raw/master/public/20220218094449.png)  

## 前言

在研究流行恶意软件（尤其是 Metasploit 和 Cobalt Strike）中常用的应用程序编程接口 (API) 哈希技术时，Huntress
ThreatOps 团队发现黑客坚持使用黑客工具附带的默认设置。我们的研究表明，许多检测/防病毒 (AV)
供应商已经意识到这一点，并围绕这些默认值留下的伪影构建了他们的检测逻辑。

经过一番修补和好奇，我们发现，如果对这些默认设置进行微不足道的更改，大量供应商将无法检测到其他简单且商品化的恶意软件。结果，简单的商品恶意软件突然开始接近
FUD 状态。😅

在这篇文章中，我们将深入探讨我们是如何发现这些细微变化的，以及您如何自己实施它们来测试您的检测工具。我们已经包含了一个脚本，该脚本可以自动执行此过程的大部分内容，以及一个
YARA 规则，该规则将检测使用此技术所做的大多数修改。

无论您是红队、蓝队还是介于两者之间的任何地方，我们都希望这篇博客能够为有趣的绕过和检测技术提供一些有用的见解。

如果像这样的屏幕截图让您兴奋，请继续阅读。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094454.png)图像1

 **技术 TL;DR**

我们的研究表明，大量供应商的 Cobalt Strike 和 Metasploit shellcode 检测功能基于 ROR13 API 哈希的存在。通过对
ROR13 逻辑进行微不足道的更改并相应地更新哈希，似乎可以绕过大量供应商检测，而不会破坏代码功能。

为了检测这种行为，可以修改之前检测到 ROR13 散列的 YARA 规则，以检测与典型的基于 ROR 的散列相关的代码块。这种转向检测 ROR
块可以提供比单独检测散列更强大的检测方法。

 **图形 TL;DR**

 **![]()**

## API 哈希

API 哈希是恶意软件经常使用的一种技术，用于在检测分析师或安全工程师的窥探下伪装可疑 API（本质上是函数）的使用。

传统上，如果某个软件需要调用 Windows API 的函数（例如，如果它想使用CreateFileW创建文件），则该软件需要在代码中“直接”引用 API
名称。这通常看起来像下面的屏幕截图。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094456.png)图像11

通过“直接”使用 API，API 的名称会保留在代码中。这使分析师能够轻松识别可疑代码可能在做什么。在这种情况下，该可疑操作是创建或打开文件。

当使用“直接”API 调用时，它还会将 API 保留在文件的导入表中。可以在 PeStudio
或任何其他分析工具中轻松查看此导入表，如下面的屏幕截图所示。请注意，我们还可以看到恶意软件正在使用的其他 API。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094459.png)图像24

如果您是试图隐藏恶意文件创建的攻击者，那么这两种情况都不理想。如果您可以将您的 API
使用情况隐藏在可能会看到`CreateFileW`然后去搜索可疑文件的分析师那里，那就更好了。

如果攻击者不希望他们的 API 出现在导入表中，那么另一种方法是动态加载
API（当恶意软件实际运行时）。最简单的方法是使用名为GetProcAddress的 Windows 函数。这种方法可以避免在导入表中留下可疑的
API，正如我们在上面在 PeStudio 中看到的那样。

使用动态加载的一个快速警告是，虽然原始可疑 API `CreateFileW`将不会出现在导入表中，但“ GetProcAddress
”的使用现在将出现在导入表中。看到 GetProcAddress 存在的敏锐分析师可以在调试器中运行恶意软件并找出正在加载的内容。

使用放置良好的断点，分析人员可以查看传递给GetProcAddress函数的参数并找出正在加载的内容。运行可疑代码后，调试器会显示类似这样的内容，揭示CreateFileW的用法并向分析师指示他们应该去寻找可能已经创建的可疑文件。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094509.png)图像23

避免这两种情况的常用方法是使用一种称为 API 散列的技术。

这是一种攻击者将实现他们自己版本的GetProcAddress的技术， 该版本将通过哈希而不是名称加载函数。这避免了将可疑 API
留在导入表中，并避免使用可以在调试器中轻松查看的可疑 API。

如果分析师想要了解发生了什么，他们需要熟悉 x86 汇编。

### TL;DR 要点

  * 有多种加载可疑 API 的方法；但是，大多数会为恶意软件分析师留下易于查找的指标
  * API 散列使用唯一的散列而不是函数名称。这会阻碍分析以断点处的字符串或参数为目标的函数名称

## 哈希指标

现在我们知道了为什么有人可能想要使用 API
哈希，我们可以看看在分析可疑代码时如何处理它。它相对容易识别，因为您经常会看到将随机十六进制值推入堆栈，然后立即调用寄存器值。

通常，此调用类似于call rbp，但从技术上讲，寄存器可以是任何值。下面是从使用 API 哈希的一些 Cobalt Strike shellcode
截取的屏幕截图。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094515.png)图像18

在屏幕截图中，我们可以看到在调用 rbp之前将两个十六进制值推送到寄存器。这些哈希值将被解析并用于加载恶意软件使用的可疑函数。

上面的哈希对应于 0x726774c ( LoadLibraryA ) 和 0xa779563a ( InternetOpenA )。

如果您要在这种情况下查找rbp的值 ，您会发现它指向 GetProcAddress 的“手动”实现，然后解析散列并调用相关的 API。

在高层次上，哈希解析逻辑类似于下面的伪代码。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094521.png)图4

此外，您会发现主要基于 ror13 散列算法的 Calculatehash Logic 与此类似。0xd (13)
的值在这里很重要，因为稍后我们将更改此值以生成可以绕过检测的新哈希。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094522.png)图8

这是一种简化，实际逻辑稍微复杂一些。如果您有兴趣更详细地了解逻辑，Nviso [1]和Avast [2]有一些关于该主题的精彩文章。

在使用 shellcode 中的 API 哈希分析大量恶意软件样本后，我们注意到类似的恶意软件家族通常会使用极其相似的哈希逻辑来计算和解析 API 哈希。

特别是，我们发现大多数 Cobalt Strike、Msfvenom 和 Metasploit 使用完全相同的哈希逻辑来解析 API
哈希。由于它们使用相同的逻辑，因此它们为任何给定函数生成相同的哈希值。

例如，Cobalt Strike 和 Metasploit在解析“ LoadLibraryA ”时都将使用哈希0x726774c。

### TL;DR 要点

  * API 哈希通过静态分析相对容易识别，尽管很难找到哈希解析到的内容
  * 类似的散列逻辑通常用于类似的恶意软件系列
  * 完全相同的哈希逻辑经常出现在来自 MsfVenom、Metasploit 和 Cobalt Strike 的样本中

## 再戳一点

我们最终发现，通过谷歌搜索代码中存在的哈希值，很容易识别由 Cobalt Strike 或 Metasploit 生成的 shellcode。

如果我们用谷歌搜索0x726774c ( LoadLibraryA ) 的值，我们会立即获得 Metasploit 框架（与 Cobalt Strike
共享代码）的点击率。如果我们用谷歌搜索0xa779563a ( InternetOpenA ) 的哈希值，我们会看到相同的结果。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094525.png)图像21

从这些框架生成我们自己的 shellcode 样本，我们观察到我们的有效载荷中存在的哈希值始终可以识别为 Metasploit 和 Cobalt
Strike 使用的那些。

### TL;DR 要点

  * Metasploit 和 Cobalt Strike（至少默认情况下）使用相同的 API 哈希例程，并且在使用相同的函数时会产生相同的哈希值
  * 这些哈希值引入了唯一的十六进制值，可用于使用 Google 轻松识别恶意软件系列

## YARA规则

从安全分析师或检测工程师的角度来看，这是很好的信息。在不深入研究 shellcode 和汇编的情况下，我们可以很容易地确定有效载荷可能属于
Metasploit 或 Cobalt Strike。

这让我们开始思考——如果这些哈希值对于 Cobalt Strike 和 Metasploit 等工具来说是独一无二的……如果这些哈希值足够独特以用于
YARA 规则呢？

我们从Avast [3]中找到了一篇精彩的文章，其中包含了同样的想法。他们的文章详细介绍了使用这些相同的 API 哈希来检测 Cobalt Strike 和
Metasploit shellcode。下面我们可以看到来自 Avast 的 YARA 规则，它主要依赖于我们之前确定的哈希值（以及 HTTP
stager 所需的其他哈希值）。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094527.png)图像22

针对我们的原始 Cobalt Strike 和 Metasploit shellcode（未启用任何编码器）测试这些 YARA 规则，我们确认Avast
YARA [4]规则集可靠地检测并识别了我们生成的所有有效负载。蓝队的好消息——以及来自 Avast 的威胁英特尔团队的出色工作。

### TL;DR 要点

  * shellcode 中的 API 哈希值是可用于检测的可靠指标
  * 供应商正在积极使用这些指标来检测恶意 shellcode

##  **但是如果 Shellcode 中的哈希值改变了怎么办？**

从表面上看，使用 API 哈希进行检测是一个好主意。但这让我们思考，如果这些哈希值发生变化会发生什么？

作为最初的概念验证，我们使用了我们的有效载荷并将哈希值粗略地更改为0x11111111。我们知道这会破坏
shellcode，因为哈希将不再解析——但它可以让我们检查在不存在已知 API 哈希的情况下检测到 shellcode 的效果。

我们的新 shellcode 将包含这样的哈希值来代替之前看到的实际哈希值。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094530.png)图片15

然后，我们使用 Virustotal 对 Cobalt Strike HTTP 有效负载进行了前后检查，发现 **在进行这些更改后，有 15
家供应商未能检测到 shellcode** 。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094531.png)图像26

作为概念验证，这非常有趣。但作为攻击者，这在很大程度上是无用的。在当前的修改状态下，shellcode 将不再解析哈希，也无法找到执行所需的 API —
将我们的 shellcode 变成了一个不错的数字镇纸。

### TL;DR 要点

  * 至少有 _一些_ 供应商正在使用 API 哈希来检测 Cobalt Strike 和类似的恶意软件
  * 如果更改这些默认值，至少 _某些_ 供应商将无法检测到以前检测到的有效载荷
  * 粗暴地修改 API 哈希会破坏你的代码

##  **但是，如果修改后的哈希可以正确解析呢？**

在确认我们怀疑供应商使用 API 哈希来检测 shellcode
之后，我们决定探索如果哈希被不那么粗暴地修改会发生什么，以一种仍然能够使修改后的哈希解析和执行的方式。

首先，我们需要准确了解哈希是如何生成的。我们的 ThreatOps 团队能够通过结合 Metasploit 源代码和分析 shellcode
样本中的汇编指令来发现这一点。

根据散列的工作原理，我们推测，只需对散列逻辑进行微小的更改，即可产生截然不同的散列。最后，我们决定将现有逻辑中的旋转值从0xd/13更改为0xf/15，而不是花哨的任何全新的哈希例程。

从理论上讲，这将产生全新的散列，同时保持基本相同的逻辑和散列结构。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094534.png)图像10

然后我们创建了一个脚本来根据我们的新旋转值 0xf 生成新的哈希值。这个逻辑可以在这篇文章中包含的最终脚本中找到。

在生成新的哈希值之后，我们更新了我们的 shellcode 以对应于我们的新哈希，以及我们的新 ror 值0xf。请注意，我们的 shellcode
结构在很大程度上仍然完好无损，唯一改变的是散列和旋转 (ror) 值。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094537.png)图像25

然后我们确认我们的代码仍然能够按预期运行。使用FireEye的 Speakeasy 工具大大加快了这个过程。[5]

下面我们可以看到在我们新修改的 shellcode 中仍然成功解析的 API 屏幕截图。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094538.png)图3

[结合使用来自 OAlabs 6]的 netcat 和B lo bRunner 工具，我们进行了额外的检查，以确认我们的 shellcode
仍然有效并且会按预期“调用”。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094539.png)图像19

在确认我们的代码确实仍然有效后，我们将其上传到 VirusTotal。并且发现我们还剩下两个供应商，这两个供应商在我们之前的虚拟值测试中是相同的。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094545.png)图2

这很有趣，因为它现在可以运行 Cobalt Strike shellcode——检测次数比修改前少 15 次。

为了进行完整性检查，我们使用来自 Metasploit 的 TCP 绑定 shell（未启用编码器）重新运行相同的过程。在确认代码仍然有效后，我们将其提交给
VirusTotal，发现有 26 家供应商未能检测到修改后的有效载荷。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094547.png)图像17![](https://gitee.com/fuli009/images/raw/master/public/20220218094552.png)图像16

在我们的分析过程中，有趣的是，剩下的两个供应商在修改后的有效载荷之间存在差异。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094554.png)图6

此时，我们还检查了原始 YARA 规则是否不再检测我们的有效负载。并确认他们不再被检测到。

### TL;DR 要点

  * 大量供应商正在使用默认的 ror13 哈希来检测 Cobalt Strike 和 Metasploit/Msfvenom 有效负载。
  * 修改这些哈希值对检测率有相当大的影响。
  * 正确完成后，修改这些哈希不会破坏 shellcode 功能。
  * 这种技术在 Msfvenom 和 Cobalt Strike 上都适用。因此也可能适用于其他恶意软件系列。

## 那么剩下的供应商呢？

我们决定解决剩下的两个供应商检测我们的 shellcode 的问题，而不是将其留在 2/55。

首先，我们注意到其余供应商正在检测通用 shellcode，而不是专门检测 Cobalt Strike 或
Metasploit。这使我们相信他们正在检测通用的 shellcode 指标，而不是我们恶意软件家族的任何特定内容。

我们推测以下可能是其余供应商的目标，因为它们是通常与 shellcode 相关的行为。

  * CLD/0xfc 是执行的第一条指令 - （CLD 用于重置字节/字符串复制操作中使用的方向标志）
  * 对寄存器的可疑调用（例如调用 rbp）
  * 堆栈字符串中存在库名称

为了测试，我们在剩余的有效载荷中稍微修改了这些指标。我们通过以下方式实现了这一目标

  * 将初始 CLD 指令移动到我们的 shellcode 中的另一个位置，以便它仍然执行但不再是第一条指令。（假设 CLD 在任何字符串操作之前执行，这应该对 shellcode 功能没有影响）
  * 插入 NOP/0x90 代替原始 CLD
  * 在对 LoadLibraryA 的初始调用的参数中插入一个大写字符。（由于 LoadLibraryA 不区分大小写，因此不应破坏任何功能）

下面，我们有一个修改后的shellcode的前后。请注意从“wininet”（全小写）到“wIninet”（一个大写 I）的细微变化。以及现在位于我们的
**pop rbp** 之后的 CLD
指令。![](https://gitee.com/fuli009/images/raw/master/public/20220218094557.png)

然后我们确认我们的 shellcode 仍然有效，然后将其重新提交给 VirusTotal。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094600.png)图 9

最后，我们在不破坏代码的情况下达到了 0/55 检测。

然后我们用 Antiscan 进行了检查，发现我们的 Cobalt Strike shellcode 的检测也为零——而未修改的副本有 13 次检测。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094602.png)图像20

 **TL;DR 要点**

  * 供应商肯定会使用 API 哈希来识别 Cobalt Strike shellcode
  * 删除 API 哈希将删除大部分（但不是全部）VirusTotal 检测
  * 由于缺乏哈希，一些供应商会在其他通用 shellcode 指标上进行检测
  * 我们可以修改这些剩余的指标来实现零检测

##  **自动化流程**

由于哈希替换过程可以通过字节级别的搜索和替换来实现，因此 Huntress ThreatOps 团队开发了一个脚本来自动化该过程。

这个脚本…

  * 将原始 shellcode 文件作为输入（不存在编码器）
  * 使用 1 到 255 之间的随机 ror 值自动执行哈希替换过程
  * 由于每次使用不同的 ror 值，每次运行时都会生成唯一的文件和哈希，允许为单个 shellcode 创建多个文件

我们决定不自动执行大写库名称和移动 CLD/0xfc
的过程，因此如果您希望零检测，则需要手动执行这些操作。这两项活动都可以手动完成，并且使用十六进制编辑器可以轻松完成。

为了使用该脚本，请使用 Msfvenom 或 Cobalt Strike 生成原始有效负载（确保您的输出是原始的 -
不要使用任何编码器），将其保存到原始二进制文件，然后将其作为参数传递给 Python 脚本。该脚本将使用随机 ror 值和唯一哈希处理哈希替换过程。

如何使用 msfvenom 生成简单的反向 shell 有效负载的示例。注意使用“--format raw”来避免使用编码器。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094608.png)图像13

以下是如何使用脚本修改 shellcode 文件的示例。

![](https://gitee.com/fuli009/images/raw/master/public/20220218094613.png)图5

### 此脚本的注意事项和限制

  * 此脚本仅替换散列和散列逻辑。如果您的 shellcode 中还有其他可疑指标，您可能需要找到自己的方法来隐藏它们
  * 这个脚本不是编码器，所以你仍然需要处理你的 shellcode 中的坏字符和空字节
  * 使用公共和知名的编码器（如 Shikata ga nai）将引入其自己的指标，这将不利于您

### 检测修改的 Shellcode

在确认我们用于生成新 shellcode 的脚本可以绕过通用检测后，我们随后开发了一个 YARA 规则来检测由我们的脚本生成的 shellcode。

下面我们包含了一个 YARA 副本，该副本检测了我们测试过的所有 Msfvenom 和 Cobalt Strike
有效负载，无论它们是否已被我们的脚本修改。在我们的测试中，我们的二进制文件测试集中没有出现任何误报，但如果出现误报，您可能希望修改规则以满足您的需求。

### 怎么运行的

由于现有的检测规则正在检测散列例程 _生成_ _的_ 散列（可以很容易地更改），因此该规则检测散列例程本身。这允许对 Cobalt Strike 和
Metasploit shellcode 进行更稳健的检测。

与任何检测一样，**此规则并非万无一失。**一个坚定的攻击者可以对哈希例程进行更复杂的更改，这将破坏此 YARA
规则。我们已经允许我们的规则有微小的变化，但更复杂的变化仍然会打败它。

    
    
    rule CobaltStrike_Ror_Hashing  
    {  
     //Yara rule for detecting ror based hashing used incobalt strike and msfvenom shellcode  
     meta:  
      author = "Huntress ThreatOps Team"  
      source = ""  
     strings:  
        
      //Detect "ToUpper" function  
      $toUpper = {ac 3c 61 7c ?? 2c 20}   
        
      //Detect 64-bit ROR sequence  
      $rorX64 = {41 c1 c9 ?? 41 01 c1 e2 ?? }  
        
      //Detect 32-bit ROR Sequence  
      $rorX86 = { c1 cf ?? 01 c7 e2 ??}  
        
        
     condition:  
      (filesize < 100KB) and  
      $toUpper and  
      1 of ($ror*)  
        
    }  
    

##  **最后的评论**

显然，检测并不总是完美的，一个意志坚定的攻击者总是能够偷偷溜进去。如果您是一名防守者，请确保您始终在测试和更新您的检测规则（您永远不知道什么会偷偷溜过去）。

如果您是攻击者（当然是红队成员），请不要依赖默认设置来解决问题——简单的更改可能会对您被检测到的机会产生重大影响。

最后，分别针对蓝队和红队的一些关键要点：

蓝队

  * 不断测试和更新您的检测逻辑
  * 积极威胁追捕！没有警报≠没有恶意软件
  * 搜索各种日志源——一个 AV 可能没有发现这一点，但网络流量可能会像大拇指一样突出

红队

  * 不要使用默认值！修补一切
  * 不要害怕熟悉组装！

### 脚本/YARA 规则

    
    
    rule CobaltStrike_Ror_Hashing  
    {  
     //Yara rule for detecting ror based hashing used incobalt strike and msfvenom shellcode  
     meta:  
      author = "Huntress ThreatOps Team"  
      source = ""  
     strings:  
        
      //Detect "ToUpper" function  
      $toUpper = {ac 3c 61 7c ?? 2c 20}   
        
      //Detect 64-bit ROR sequence  
      $rorX64 = {41 c1 c9 ?? 41 01 c1 e2 ?? }  
        
      //Detect 32-bit ROR Sequence  
      $rorX86 = { c1 cf ?? 01 c7 e2 ??}  
        
        
     condition:  
      (filesize < 100KB) and  
      $toUpper and  
      1 of ($ror*)  
        
    }  
    

 **主脚本（在此处查看完整脚本[7]）**

    
    
     """  
    Shellcode API detection and replacement script -  
    Written by Huntress Labs ThreatOps Team  
    Can be used to detect and/or modify hashes used by  
    cobaltstrike/msfvenom/metasploit  
    """  
      
      
    import random,sys,re  
    matchlist = []  
      
    #Read in architecture as an argument  
    try:   
        arch = str(sys.argv[1]).strip()  
        print("Architecture: " + arch)  
    except:  
        print("No input args")  
        print("python hashreplace.py <32 or 64 arch> <input file.bin>")  
    

## 译文申明

  * 文章来源为`近期阅读文章`，质量尚可的，大部分较新，但也可能有老文章。
  * `开卷有益，不求甚解`，不需面面俱到，能学到一个小技巧就赚了。
  * `译文仅供参考`，具体内容表达以及含义,  _`以原文为准`_  (译文来自自动翻译)
  * 如英文不错的，`尽量阅读原文`。(点击原文跳转)
  * `每日早读`基本自动化发布(不定期删除)，这是`一项测试`

> > > ### `最新动态: Follow Me`
>>>

>>> 微信/微博： **`red4blue`**

>>>

>>> 公众号/知乎： **`blueteams`**

>>>

>>> ![](https://gitee.com/fuli009/images/raw/master/public/20220218094615.png)  
>

![](https://gitee.com/fuli009/images/raw/master/public/20220218094617.png)

  

![]()

bggsec

![赞赏二维码]() **微信扫一扫赞赏作者** __赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

阅读原文

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

译文 | 随机化 API 哈希以规避 Cobalt Strike Shellcode 检测

最多200字，当前共字

__

发送中

[写留言](javascript:;)

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

