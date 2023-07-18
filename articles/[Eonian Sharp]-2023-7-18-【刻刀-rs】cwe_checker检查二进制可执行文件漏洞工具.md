#  【刻刀-rs】cwe_checker检查二进制可执行文件漏洞工具

Enomothem  [ Eonian Sharp ](javascript:void\(0\);)

**Eonian Sharp** ![]()

微信号 Eonian_sharp

功能介绍 Eonian Sharp | 永恒之锋，专注APT框架、渗透测试攻击与防御的研究与开发，没有永恒的安全，但有永恒的正义之锋击破黑暗的不速之客。

____

___发表于_

收录于合集

#刻刀系列 6 个

#Rust 4 个

#工具 14 个

# 什么是cwe_checker？

cwe_checker
是一套检查，用于检测常见错误类，例如空指针取消引用和缓冲区溢出。这些错误类别的正式名称为常见弱点枚举(CWE)。检查基于各种分析技术，从简单的启发式到基于抽象解释的数据流分析。其主要目标是帮助分析人员快速找到潜在易受攻击的代码路径。它的主要关注点是
Linux 和 Unix 操作系统上常见的 ELF 二进制文件。cwe_checker 使用Ghidra将二进制文件分解为一种通用的中间表示形式，并在此
IR 上实现自己的分析。因此，分析可以在 Ghidra 可以反汇编的大多数 CPU 架构上运行，这使得 cwe_checker
成为固件分析的重要工具。以下论点应该说服您尝试一下 _cwe_checker_ ：

  * 设置非常简单，只需构建 Docker 容器即可！
  * 它分析多种 CPU 架构的 ELF 二进制文件，包括 x86、ARM、MIPS 和 PPC
  * 由于其基于插件的架构，它是可扩展的
  * 它是可配置的，例如将分析应用于新的 API
  * 查看用 Ghidra 注释的结果
  * cwe_checker 可以作为插件集成到FACT中

![]()

# 使用Docker镜像

最简单的方法是从Github 容器注册表中提取最新的 Docker 镜像：

  * 生成基于当前主分支的图像。

    
    
    docker pull ghcr.io/fkie-cad/cwe_checker:latest

  * 生成基于最新稳定发行版本的映像。

    
    
    docker pull ghcr.io/fkie-cad/cwe_checker:stable

  * 生成基于 v0.7 稳定发行版的映像。但是，建议在发布后立即切换到较新的稳定版本，因为稳定版本之间的改进可能非常显着。

    
    
    docker pull ghcr.io/fkie-cad/cwe_checker:v0.7

如果你想自己构建docker镜像，只需运行

    
    
    docker build -t cwe_checker .

通过这种方式，您还可以为基于 ARM 的 PC（例如较新的 Apple Mac）构建本机 Docker 映像。预构建的 Docker 镜像目前仅基于
x86。

# 本地安装

必须安装以下依赖项才能在本地构建和安装 _cwe_checker_ ：

  * Rust >= 1.69
  * Ghidra >= 10.2

运行make all GHIDRA_PATH=/path/to/ghidra_folder（插入本地 Ghidra 安装的正确路径）来编译并安装
cwe_checker。如果省略该GHIDRA_PATH参数，安装程序将在您的文件系统中搜索 Ghidra 的本地安装。

# 用法

cwe_checker将二进制文件作为输入，基于二进制文件的静态分析运行多项检查，然后输出在分析过程中发现的 CWE
警告列表。如果你使用官方的docker镜像，只需运行

    
    
    docker run --rm -v /PATH/TO/BINARY:/input ghcr.io/fkie-cad/cwe_checker /input

如果您在本地安装了 _cwe_checker_ ，请运行

    
    
    cwe_checker BINARY

您可以通过位于 的配置文件调整大多数检查的行为src/config.json。如果修改它，请添加命令行标志--
config=src/config.json以告诉 _cwe_checker_ 使用修改后的文件。有关其他可用命令行标志的信息，您可以将该--
help标志传递给 _cwe_checker_ 。如果您使用稳定版本，还可以查看在线文档以获取更多信息。

# 对于裸机二进制文件

cwe_checker 为分析裸机二进制文件提供实验支持。为此，需要通过--bare-metal-
config命令行选项提供裸机配置文件。可以在以下位置找到此类配置文件的示例bare_metal/stm32f407vg.json （该文件是为
STM32F407VG MCU 创建和测试的）。有关更多信息，请查看在线文档。

# 文档和测试

我们的测试套件的测试二进制文件可以使用make compile_test_files（需要安装 Docker！）来构建。然后可以使用 运行测试套件make
test。源代码文档可以使用make documentation. 对于稳定版本，可以在此处找到文档。

# 实施检查

到目前为止，已实施以下分析：

  * CWE-78：操作系统命令注入（当前在标准运行中禁用）
  * CWE-119及其变体CWE-125和CWE-787：缓冲区溢出
  * CWE-134：使用外部控制的格式字符串
  * CWE-190：整数溢出或环绕
  * CWE-215：通过调试信息暴露信息
  * CWE-243：在不更改工作目录的情况下创建 chroot Jail
  * CWE-332：PRNG 中的熵不足
  * CWE-367：检查时间使用时间 (TOCTOU) 竞争条件
  * CWE-416：释放后使用及其变体CWE-415：双重释放
  * CWE-426：不受信任的搜索路径
  * CWE-467：在指针类型上使用 sizeof()
  * CWE-476：空指针取消引用
  * CWE-560：使用 umask() 和 chmod 样式参数
  * CWE-676：使用潜在危险功能
  * CWE-782：暴露的 IOCTL 访问控制不足
  * CWE-789：内存分配大小值过大

请注意，由于捷径和静态分析的性质以及过度近似，误报和误报都是可以预料的。您可以在特定于检查的文档页面上找到有关每项检查的内部工作原理以及误报和漏报的已知原因的信息。

# 集成到其他工具中

cwe_checker 附带了一个 Ghidra 脚本，它解析 cwe_checker 的输出并在反汇编器中注释找到的
CWE，以便于手动分析。该脚本位于ghidra_plugin/cwe_checker_ghidra_plugin.py，文件中包含使用说明。![]()cwe_checker
也作为插件集成到FACT中。如果您想将 cwe_checker 集成到您自己的分析工具链中，您可以使用命令行标志（与或命令行选项--
json组合）以易于解析的 JSON 输出格式生成 CWE 警告。--quiet``--out=...cwe_checker
内部如何工作？使用构建文档cargo doc --open --document-private-items --no-deps将为您提供有关
cwe_checker
内部结构的更多信息。然而，最好的文档仍然是源代码本身。如果您有疑问，请务必在我们的讨论页面上提问！我们不断努力改进可扩展性和文档，您的问题将帮助我们实现这一目标！
_要快速/初步了解其内部结构，您还可以查看doc_ 文件夹中 _cwe_checker_ 上的会议演示幻灯片。到目前为止，我们在以下会议上展示了
cwe_checker：

  * 通过 2019 年 SALT（幻灯片）
  * Black Hat USA 2019（幻灯片）
  * Black Hat USA 2022（幻灯片）

![]()

![]()

![]()

![]()

![]()

# 参考

https://github.com/fkie-cad/cwe_ch‍e‍ckerhttps://fkie-
cad.github.io/cwe_checker/doc/html/cwe_checker_lib/index.html  
  

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

