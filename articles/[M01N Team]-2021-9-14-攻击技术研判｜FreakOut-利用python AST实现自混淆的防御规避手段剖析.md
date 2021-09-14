#  攻击技术研判｜FreakOut-利用python AST实现自混淆的防御规避手段剖析

原创 天元实验室  [ M01N Team ](javascript:void\(0\);)

**M01N Team** ![]()

微信号 m01nteam

功能介绍 攻击对抗研究分享

____

__

收录于话题 #攻击技术研判 ,14个

![](https://gitee.com/fuli009/images/raw/master/public/20210914095629.png)

**情报背景**

今年2月份绿盟科技伏影实验室曾对名为FreakOut（又名necro）的botnet的技术细节与背后的攻击组织进行了披露与分析。Talos的安全研究人员在6月份的跟进分析报告中对其新添加的漏洞利用与代码混淆方式进行了较多的介绍。

本篇研判将重点对其利用ast模块对自身代码进行混淆的技术进行分析研判。

  

 **组织名称**

|

暂无  
  
---|---  
  
 **战术标签**

|

执行 防御规避  
  
 **技术标签**

|

python、跨平台、自修改、AST  
  
 **情报来源**

|

https://blog.talosintelligence.com/2021/06/necro-python-bot-adds-new-
tricks.htmhttps://research.checkpoint.com/2021/freakout-leveraging-newest-
vulnerabilities-for-creating-a-botnet/  

https://mp.weixin.qq.com/s/KVqgIb1flhWrZKbeeH6LQA  
  
  

 **01** 攻击技术分析

 **亮点：利用ast模块进行自混淆的多态引擎**

FreakOut的多态引擎会对自身代码中包含的类名、变量名、函数名以及字符串进行混淆，且这种混淆会在每次执行中进行，产生新的木马变种。对基于文件哈希的静态查杀具有较好的规避效果。

  

AST模块是python中进行语法分析的内置模块，能将python源码转换为抽象语法树进行分析与修改。FreakOut会利用AST模块解析原始脚本，遍历每一个语法树节点，对其中包含的类、变量名、函数名进行匹配与重命名，完成混淆之后覆盖原始脚本文件。

    
    
     1#处理混淆的函数  
     2def TBoKeAjHqmL(self):  
     3    VarName = []  
     4    FunctionNameList = []  
     5    classDef = []  
     6    #读取原始脚本  
     7    OriginalScriptHandle=open(PathToOriginalScriptFile,"rb")  
     8    ObfScriptContent=OriginalScriptFile.read()  
     9    OriginalScriptContent=ObfScriptContent  
    10    OriginalScriptFile.close()  
    11    #利用ast.parse将原始脚本解析为AST节点  
    12    p = ast.parse(OriginalScriptContent)  
    13    ASTNodeVistor().visit(p)  
    14  
    15#...混淆与加密过程  
    16  
    17vLhbvPnCn=open(PathToOriginalScriptFile,"wb")  
    18#覆盖原路径下的脚本  
    19vLhbvPnCn.write(ObfScriptContent)  
    20vLhbvPnCn.close()  
    

  

此处利用python内置函数isinstance函数对节点类型与ast内置类型进行对比实现了对象类型识别。

 **More ActionsAST内置类型**

|

 **说明**  
  
---|---  
  
ast.ClassDef

|

类定义  
  
ast.Name

|

变量定义  
  
ast.FunctionDef

|

函数定义  
  
      
    
     1#获取类的列表  
     2classDef = sorted([node.name for node in ast.walk(p) if isinstance(node, ast.ClassDef)], key=len, reverse=True)  
     3#获取变量名列表  
     4VarName = sorted([node.id for node in ast.walk(p) if isinstance(node, ast.Name) and not isinstance(node.ctx, ast.Load)], key=len, reverse=True)  
     5#获取函数名列表  
     6for FunctionDef in [n for n in p.body if isinstance(n, ast.FunctionDef)]:  
     7    FunctionNameList.append(FunctionDef.name)  
     8classDef = [node for node in ast.walk(p) if isinstance(node, ast.ClassDef)]  
     9for cwass in classDef:  
    10    for FunctionDef in [n for n in cwass.body if isinstance(n, ast.FunctionDef)]:  
    11        if FunctionDef.name != "init" and FunctionDef not in FunctionNameList:  
    12            FunctionNameList.append(FunctionDef.name)  
    

  

对于字符串的处理较为不同，利用内置加密函数对其处理之后使用zlib进行压缩并转义为字符串，与解密函数拼接以后替换原始字符串。在之后的运行过程中则会调用内置的解密函数进行解密处理。

    
    
     1for jnBeojpf in sorted(StrList, key=len, reverse=True):  
     2    if len(jnBeojpf)>=KCvsVgjSbXZ:  
     3        try:  
     4            #依据开头与末尾进行匹配，进行字符串混淆  
     5            if (jnBeojpf[0] == "'" and jnBeojpf[-1] == "'") or (jnBeojpf[0] == '"' and jnBeojpf[-1] == '"'):  
     6                ObfScriptContent=ObfScriptContent.replace(jnBeojpf, "PIfmMyYaBR(zlib.decompress(\x22"+self.AGcdigaJn(zlib.compress(PIfmMyYaBR(jnBeojpf[1:-1].decode('string_escape'))))+"\x22))")  
     7            else:  
     8                ObfScriptContent=ObfScriptContent.replace(jnBeojpf, "PIfmMyYaBR(zlib.decompress(\x22"+self.AGcdigaJn(zlib.compress(PIfmMyYaBR(eval(jnBeojpf).decode('string_escape'))))+"\x22))")  
     9        except:  
    10            pass  
    

  

虽然此处对字符串进行了混淆，但是也不可避免地保留了例如“zlib.decompress”这样的固定特征字符串，无论多少次混淆都会保留，可通过简单的字符串匹配识别，这是其当前存在的局限性。

  

目前FreakOut对python
AST模块的滥用手法还较为简单，仅限于类名、函数名、变量的改名与字符串的简单加密替换。在ASTObfuscate等开源的python混淆器项目中可见利用AST模块实现的更加复杂的混淆方式，如下图：

![]()

  

 **02** 总结

FreakOut从首次被发现后多次通过版本迭代增强利用各种防御规避技术其隐匿性。虽然在脚本中对对象进行改名、混淆字符串是针对静态检测的一种常规的隐匿手段，但是对脚本代码的混淆过程通常发生在攻击者投递载荷之前。FreakOut充分利用了python的内置模块AST的特性进行自身的解析与混淆，虽然混淆方式相对较为简单，但在较小代码量的前提下达到即时混淆的效果，在实战环境下也不失为一种高性价比的做法。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210914095630.png)

 **绿盟科技M01N战队** 专注于Red
Team、APT等高级攻击技术、战术及威胁研究，涉及Web安全、终端安全、AD安全、云安全等相关领域。通过研判现网攻击技术发展方向，以攻促防，为风险识别及威胁对抗提供决策支撑，全面提升安全防护能力。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210914095631.png)

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

攻击技术研判｜FreakOut-利用python AST实现自混淆的防御规避手段剖析

最多200字，当前共字

__

发送中

写下你的留言

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

