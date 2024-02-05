#  AsyncRAT攻防技术对抗

原创 T0daySeeker  [ T0daySeeker ](javascript:void\(0\);)

**T0daySeeker** ![]()

微信号 gh_26c54b2c64aa

功能介绍 专注APT样本的木马分析、流量分析、通信模型分析、攻击场景复现等。

____

___发表于_

    
    
    文章首发地址：  
    https://xz.aliyun.com/t/13505  
    文章首发作者：  
    T0daySeeker  
    

## 概述

本篇文章为AsyncRAT远控工具剖析系列文章的第三篇，第一篇为《AsyncRAT加解密技术剖析》，第二篇为《AsyncRAT通信模型剖析及自动化解密脚本实现》，欢迎各位大佬关注并指点。

本篇文章将继续对AsyncRAT远控工具进行剖析，将从受控端角度对AsyncRAT木马使用的相关技术进行剖析，并从网络检测、终端检测、内存检测、内存马提取、内存中提取配置信息等多个角度提出临检取证的技术方案，便于全面的对AsyncRAT木马进行检测分析发现。

## 网络检测

通常，在一个单位的网络环境中，部署有大量的PC机及服务器，为了能够快速的从此网络环境中检测挖掘木马攻击行为，网络流量检测报警是最便捷直接的方式，因此，提取有效的木马通信行为特征，有利于通过网络流量方式对木马通信行为进行检测报警。

通过前两篇文章的分析梳理，可知AsyncRAT木马使用的TLS加密通信，因此只能从加密数据角度对此样本通信行为进行检测识别，通过分析，笔者梳理了以下几点可供网络流量检测报警：

  * 加密通信使用TLS1.0通信协议；
  * 通过多平台分析测试，发现若AsyncRAT控制端在WIN7系统中运行，则在AsyncRAT木马通信过程中，木马将选择TLS_RSA_WITH_AES_128_CBC_SHA (0x002f)密钥套件；
  * 木马存在心跳数据包行为：源码中指定了心跳间隔时间为10-15秒，实际测试过程中基本稳定在15秒；
  * 由于心跳数据包的最终数据载荷基本固定，因此心跳数据为固定长度，所以加密后数据长度也为固定长度；
  * TLS解密后的数据中存在部分明文字符串；

### 固定通信协议、密钥套件

相关数据包截图如下：

![]()

### 心跳数据包

相关解密数据截图如下：

![]()

相关代码截图如下：

![]()

![]()

### TLS解密数据存在明文字符串

在前期通信数据包解密过程中，笔者发现了一个有趣的情况：使用C#调用的gzip压缩代码，在对部分数据进行压缩时，会出现明文字符串的情况。

例如：

  * gzip解压前数据：（存在Packet、savePlugin、Dll字符串，逻辑上此字符串应该是解压后才会出现）

![]()

  * gzip解压后数据：

![]()

基于上述情况，笔者尝试编写了多个C#程序，用于对此现象进行模拟（C#模拟代码中，将gzip解密后的数据作为输入数据），最终发现，使用C#调用gzip压缩代码时，确实会出现明文字符串的情况。
**「（备注：数据长度是否会对其产生影响？大概阈值是多少？笔者未进行过验证，因此无法确定）」**

相关代码截图如下：

![]()

![]()

输出二进制中明文字符串截图如下：

![]()

![]()

## 终端检测

通常，若在网络检测中发现可疑木马通信行为，则需要进一步的在终端中进行核查取证，同时对木马通信行为进行证据固定。

针对终端检测角度，常见手法是基于木马特征生成对应的yara检测规则，然后使用对应的yara规则加载工具对终端进行扫描检测：

  * yara规则：可从样本中提取相关特征字符串生成yara规则；在这里，笔者将直接使用网络中关于AsyncRAT的yara检测规则（https://github.com/IrishIRL/yara-rules/blob/main/RAT_AsyncRAT.yara ）；
  * yara规则加载工具：常见的yara规则加载工具为yara.exe程序（https://github.com/VirusTotal/yara/releases ）；但在这里，笔者将使用另一款Loki工具（https://github.com/grafana/loki ）进行yara加载检测；

### yara规则

通过网络调研，笔者在网络中发现IrishIRL作者于2022年12月03日编写的RAT_AsyncRAT.yara规则，经过简单对比分析，发现相关特征均为AsyncRATClient端木马的特征字符串。

进一步对特征字符串进行分析研判，发现在AsyncRAT控制端生成AsyncRATClient端木马过程中，可在AsyncRAT控制端中开启反病毒分析功能，因此，笔者尝试对不同功能选项下生成的AsyncRATClient端木马进行特征字符串对比，发现不同功能选项下生成的AsyncRATClient端木马均存在RAT_AsyncRAT.yara规则文件中提到的相关特征字符串。

相关截图如下：

![]()

### Loki

Loki是一款简单强大的IOC和事件响应扫描器，当前仅支持在windows平台中运行，此工具除支持Yara匹配文件内容外，还支持正则匹配路径、文件名，Yara匹配进程内存，匹配扫描目录中的文件Hash，匹配进程网络行为C2，文件系统检测，进程异常检测等。

Loki在Yara匹配文件内容功能中，提供了多种告警级别（ [ALERT]（>100），
[WARNING]（60-99），[NOTICE]（0-59）），在多种告警模式下，支持多条yara规则的叠加告警，相关截图如下：

![]()

尝试使用Loki工具加载RAT_AsyncRAT.yara规则进行终端检测扫描，发现可成功对不同功能选项下生成的AsyncRATClient端木马进行告警，相关检测结果如下：

    
    
    C:\Users\admin\Desktop\loki>loki.exe  -p C:\Users\admin\Desktop\test\ --onlyrelevant --noprocscan >1.txt                                                                                
          __   ____  __ ______    
         / /  / __ \/ //_/  _/    
        / /__/ /_/ / ,< _/ /      
       /____/\____/_/|_/___/      
       YARA and IOC Scanner       
        
       by Florian Roth, GNU General Public License  
       version 0.46.1 (Python 3 release)  
        
       DISCLAIMER - USE AT YOUR OWN RISK  
      
                                                                                     
      
    -[WARNING]   
    FILE: C:\Users\admin\Desktop\test\AsyncClient.exe SCORE: 70 TYPE: EXE SIZE: 46080   
    FIRST_BYTES: 4d5a90000300000004000000ffff0000b8000000 / <filter object at 0x03268E20>   
    MD5: cfd32f4b27b21671d3ee96be07e86361   
    SHA1: f78cc04ae6c06f53b63a045e21b2259a20920d55   
    SHA256: 86e0bc6c09b35777f71c4053582ad5bd615e6e447dd3350dc19dc953d2d778f7 CREATED: Fri Jan 26 15:52:26 2024 MODIFIED: Thu Jan 25 10:57:55 2024 ACCESSED: Fri Jan 26 15:52:30 2024   
    REASON_1: Yara Rule MATCH: AsyncRAT SUBSCORE: 70   
    DESCRIPTION: AsyncRAT v0.5.7B - Remote Administration Tool, became popular across hackforums members REF: https://github.com/NYAN-x-CAT/AsyncRAT-C-Sharp AUTHOR: IrishIRL   
    MATCHES: Str1: MZ Str2: /c schtasks /create /f /sc onlogon /rl highest /tn " Str3: START "" " Str4: DEL " Str5: System.Drawing.Imaging Str6: System. ... (truncated)  
    \[WARNING]   
    FILE: C:\Users\admin\Desktop\test\AsyncClient_混淆.exe SCORE: 70 TYPE: EXE SIZE: 48640   
    FIRST_BYTES: 4d5a90000300000004000000ffff0000b8000000 / <filter object at 0x03268460>   
    MD5: b9e6ba8a2ab93a3e0fcc9b695f8e59b5   
    SHA1: e5a99670ba330d00e80b45b39ce745623ea68219   
    SHA256: aad45a159d128852e659b483f19156e773074e43557ea0a6e344bedee36bf0c5 CREATED: Wed Jan 31 14:40:10 2024 MODIFIED: Wed Jan 31 14:32:51 2024 ACCESSED: Wed Jan 31 14:40:27 2024   
    REASON_1: Yara Rule MATCH: AsyncRAT SUBSCORE: 70   
    DESCRIPTION: AsyncRAT v0.5.7B - Remote Administration Tool, became popular across hackforums members REF: https://github.com/NYAN-x-CAT/AsyncRAT-C-Sharp AUTHOR: IrishIRL   
    MATCHES: Str1: MZ Str2: /c schtasks /create /f /sc onlogon /rl highest /tn " Str3: START "" " Str4: DEL " Str5: System.Drawing.Imaging Str6: System. ... (truncated)  
    C:\Users\admin\Desktop\loki>  
    

相关截图如下：

![]()

![]()

## 内存检测

在对AsyncRAT远控工具进行研究的过程中，笔者尝试查阅了网络中大部分涉及AsyncRAT木马的相关事件报告，发现部分AsyncRAT事件报告中对AsyncRAT木马的投放方式进行了描述，相关截图如下：

![]()

基于上述AsyncRAT事件报告，笔者发现，AsyncRAT木马可能还会以无文件落地的方式存在于终端主机中，因此，单纯的终端检测可能还无法完全对其进行检测发现，所以，还需要结合内存对其进行全面的检测发现。

由于上述yara规则是基于静态文件进行检测的，因此，静态文件的特征字符串可能无法完全在内存中适用，因此，笔者尝试性的从内存中提取了部分内存字符串构造yara检测规则，并在AsyncRAT木马运行过程中启动Loki工具对其进程内存进行检测，发现可有效对其检测。

相关yara规则如下：

![]()

相关检测效果截图如下：

![]()

## 内存马提取

若通过内存检测成功发现AsyncRAT木马痕迹，则下一步需要对其内存进行证据固定，并尝试从内存中提取还原AsyncRAT木马模块，在这里，笔者推荐使用MegaDumper工具（https://github.com/CodeCracker-
Tools/MegaDumper），此工具可以轻松地直接从内存中转储.NET可执行文件，可秒杀各种.Net 内存释放壳。

尝试使用工具模拟注入AsyncRAT内存马至aspnet_compiler.exe文件进程中，相关截图如下：

![]()

使用MegaDumper工具dump AsyncRAT木马模块截图如下：

![]()

dump输出文件夹截图如下：

![]()

使用dnspy对dump提取的.NET模块进行反编译，截图如下：

![]()

## 内存中提取配置信息

若能够从内存中成功提取木马模块，则可直接从木马模块中提取并解密配置信息；若无法从内存中提取木马模块，则只能从内存中筛选提取配置信息，然后尝试性的进行解密。

基于此，笔者准备尝试看能否从内存中直接提取配置信息。进一步分析，笔者发现内存中存放配置信息处存在部分特征格式，因此可尝试编写脚本辅助对其配置信息进行提取解密：

  * 内存中存放配置信息处存在特征格式：配置信息字符串前4字节处存放了对应配置信息字符串的长度，配置信息字符串前12字节处存放了固定4字节数据（0x00 0x00 0x00 0x80）；
  * 配置信息中Key字符串长度固定：通过源码及实际样本对比，发现配置信息中Key字符串长度固定为0x2C字节；
  * 通过Key字符串特征在内存中查找存放配置信息的位置：可通过编写脚本实现辅助查找，进程内存可通过processhacker直接dump；
  * 从对应内存片段中提取配置信息并使用系列文章的第一篇《AsyncRAT加解密技术剖析》文章中的自动化解密脚本解密；

### 内存中特征格式

相关截图如下：

![]()

### Key字符串

相关代码截图如下：

![]()

### 内存片段查找

尝试编写脚本对内存片段进行查找，运行效果如下：

    
    
    F:\GolandProjects\awesomeProject30>awesomeProject30.exe  
    找到匹配:  
    起始索引: 0xcc99d, 结束索引: 0xcc9a9  
    200000000000000000000000000000f92c0000ff2c000000200000000000000000000000000000002d0000252d000000001000000000000000000000000000302d0000652d0000000000000000000000040000000000006f  
                   �,  �,                   -  %-                 0-  e-                 o  
    起始索引: 0x4c41bc, 结束索引: 0x4c41c8  
    56005600630079004d006d0052006a005200450056003000610048005a0057005300570074003100520054004e0044004f00440052004400510058005a005400620044004200580062006b0039006b005700480049003d00  
    V V c y M m R j R E V 0 a H Z W S W t 1 R T N D O D R D Q X Z T b D B X b k 9 k W H I =  
    起始索引: 0x4c7da0, 结束索引: 0x4c7dac  
    49006e00760061006c00690064004f007000650072006100740069006f006e005f0041005000490049006e00760061006c006900640046006f007200430075007200720065006e00740043006f006e007400650078007400  
    I n v a l i d O p e r a t i o n _ A P I I n v a l i d F o r C u r r e n t C o n t e x t  
    起始索引: 0x4e2acc, 结束索引: 0x4e2ad8  
    4d006900630072006f0073006f0066007400200055006e00690066006900650064002000530065006300750072006900740079002000500072006f0074006f0063006f006c002000500072006f0076006900640065007200  
    M i c r o s o f t   U n i f i e d   S e c u r i t y   P r o t o c o l   P r o v i d e r  
    起始索引: 0x4e3074, 结束索引: 0x4e3080  
    4d006900630072006f0073006f0066007400200055006e00690066006900650064002000530065006300750072006900740079002000500072006f0074006f0063006f006c002000500072006f0076006900640065007200  
    M i c r o s o f t   U n i f i e d   S e c u r i t y   P r o t o c o l   P r o v i d e r  
    起始索引: 0x4e3ea4, 结束索引: 0x4e3eb0  
    53006f006600740077006100720065005c004d006900630072006f0073006f00660074005c00570069006e0064006f007700730020004e0054005c00430075007200720065006e007400560065007200730069006f006e00  
    S o f t w a r e \ M i c r o s o f t \ W i n d o w s   N T \ C u r r e n t V e r s i o n  
    起始索引: 0x4e469c, 结束索引: 0x4e46a8  
    5300770069007400630068002e00530079007300740065006d002e004e00650074002e0044006f006e00740045006e00610062006c006500530063006800530065006e0064004100750078005200650063006f0072006400  
    S w i t c h . S y s t e m . N e t . D o n t E n a b l e S c h S e n d A u x R e c o r d  
    起始索引: 0x59a724, 结束索引: 0x59a730  
    49006e00760061006c00690064004f007000650072006100740069006f006e005f0041005000490049006e00760061006c006900640046006f007200430075007200720065006e00740043006f006e007400650078007400  
    I n v a l i d O p e r a t i o n _ A P I I n v a l i d F o r C u r r e n t C o n t e x t  
    起始索引: 0x5ed5a8, 结束索引: 0x5ed5b4  
    4d006900630072006f0073006f0066007400200055006e00690066006900650064002000530065006300750072006900740079002000500072006f0074006f0063006f006c002000500072006f0076006900640065007200  
    M i c r o s o f t   U n i f i e d   S e c u r i t y   P r o t o c o l   P r o v i d e r  
    起始索引: 0x5edf14, 结束索引: 0x5edf20  
    4d006900630072006f0073006f0066007400200055006e00690066006900650064002000530065006300750072006900740079002000500072006f0074006f0063006f006c002000500072006f0076006900640065007200  
    M i c r o s o f t   U n i f i e d   S e c u r i t y   P r o t o c o l   P r o v i d e r  
    起始索引: 0x5eedc0, 结束索引: 0x5eedcc  
    53006f006600740077006100720065005c004d006900630072006f0073006f00660074005c00570069006e0064006f007700730020004e0054005c00430075007200720065006e007400560065007200730069006f006e00  
    S o f t w a r e \ M i c r o s o f t \ W i n d o w s   N T \ C u r r e n t V e r s i o n  
    起始索引: 0x5efe54, 结束索引: 0x5efe60  
    5300770069007400630068002e00530079007300740065006d002e004e00650074002e0044006f006e00740045006e00610062006c006500530063006800530065006e0064004100750078005200650063006f0072006400  
    S w i t c h . S y s t e m . N e t . D o n t E n a b l e S c h S e n d A u x R e c o r d  
      
    F:\GolandProjects\awesomeProject30>  
    

相关截图如下：

![]()

![]()

尝试对base64字符串进行人工识别，发现AsyncClient.exe.dmp文件的0x4c41bc偏移处即为配置信息的Key字符串，因此可直接跳转至对应偏移处查看配置信息(配置信息的顺序与反编译代码中的顺序相同)，相关截图如下：

![]()

#### 代码实现

代码结构如下：

![]()

  * main.go

    
    
    package main  
      
    import (  
     "encoding/hex"  
     "fmt"  
     "io/ioutil"  
     "os"  
     "regexp"  
    )  
      
    func main() {  
     sss, _ := readFile("C:\\Users\\admin\\Desktop\\AsyncClient.exe.dmp")  
     data := hex.EncodeToString([]byte(sss))  
      
     pattern := regexp.MustCompile("00000080.{8}2c000000")  
     matches := pattern.FindAllStringIndex(data, -1)  
     if len(matches) > 0 {  
      fmt.Println("找到匹配:")  
      for _, match := range matches {  
       startIndex := match[0] / 2  
       endIndex := match[1] / 2  
       fmt.Printf("起始索引: 0x%x, 结束索引: 0x%x\n", startIndex, endIndex)  
       data_match := data[endIndex*2 : endIndex*2+0x2c*2*2]  
       fmt.Println(data_match)  
       aa, _ := hex.DecodeString(data_match)  
       fmt.Println(string(aa))  
      }  
     } else {  
      fmt.Println("未找到匹配")  
     }  
    }  
      
    func readFile(filename string) (string, error) {  
     f, err := os.Open(filename)  
     if err != nil {  
      return "", err  
     }  
     defer f.Close()  
      
     b, err := ioutil.ReadAll(f)  
     if err != nil {  
      return "", err  
     }  
      
     return string(b), nil  
    }  
    

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# AsyncRAT攻防技术对抗

原创 T0daySeeker  [ T0daySeeker ](javascript:void\(0\);)

轻触阅读原文

![]()

T0daySeeker

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

