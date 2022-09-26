#  互动答疑 | ASPX的webshell权限为什么比ASP的大？

专攻.NET安全的  [ dotNet安全矩阵 ](javascript:void\(0\);)

**dotNet安全矩阵** ![]()

微信号 doNetSafety

功能介绍
感谢关注"dotNet安全矩阵"，分享微软.NET安全技术知识的盛宴，这里不仅有C#安全漏洞，还有.NET反序列化漏洞研究，只要是.NET领域的安全技能均在话题范围内，风里雨里我在这里等你。

____

___发表于_

收录于合集

# 0x01 背景

dotNet安全矩阵微信群里有师傅问了一个妖娆的面试题，ASPX的webshell权限为什么比ASP的大？根据微软IIS迭代时间线大致可分为两个阶段去看这个问题。

# 0x02 IIS版本间的差异

## 2.1 IIS7.5之前

在IIS6.0 -
IIS7.5，.NET默认的账户是aspnet，隶属于Users，而此时的ASP基于IUSER账户，隶属于Guest组，对于组来说Users组的权限 >
Guest组，所以能执行或者读取的资源更多。

![](https://gitee.com/fuli009/images/raw/master/public/20220926130341.png)

## 2.2 IIS7.5之后

IIS7.5以后到现在IIS10，.NET默认的账户是iis apppool\defaultapppool，默认身份为
ApplicationPoolIdentity

![](https://gitee.com/fuli009/images/raw/master/public/20220926130343.png)

而此账户继承了Users组和IIS_IUSRS组的成员，例如执行 whoami/groups 返回结果如图

![](https://gitee.com/fuli009/images/raw/master/public/20220926130344.png)

而ASP同样也是继承于apppool\defaultapppool，返回用户组也和.NET一致，如图![](https://gitee.com/fuli009/images/raw/master/public/20220926130346.png)

## 2.3 结论

在老版本的IIS里的确ASPX的webshell权限 > ASP
Webshell；新版本的IIS里只要开启了ASP的话，两者的权限一样的，但新版的IIS默认已经关闭了ASP，用进程工具做了交叉验证，的确在IIS新版权限一致，都是基于默认的应用程序池defaultapppool

![](https://gitee.com/fuli009/images/raw/master/public/20220926130347.png)

# 0x03 星球优惠活动

为了更好地应对基于.NET技术栈的风险识别和未知威胁，dotNet安全矩阵星球从创建以来一直聚焦于.NET领域的安全攻防技术，定位于高质量安全攻防星球社区，也得到了许多师傅们的支持和信任，通过星球深度连接入圈的师傅们，一起推动.NET安全高质量的向前发展。
**星球提供50元代金劵，师傅们先到先得噢！扫描星球亮点里的二维码即可加入我们。**

![](https://gitee.com/fuli009/images/raw/master/public/20220926130348.png)

星球汇聚了各行业安全攻防技术大咖，并且每日分享.NET安全技术干货以及交流解答各类技术等问题，社区中发布 **很多高质量的.NET**
安全资源，可以说市面上很少见，都是干货。其中主题包括 **.NET
Tricks、漏洞分析、内存马、代码审计、预编译、反序列化、webshell免杀、命令执行、C#工具库** 等等，后续还会倾力打造 **专刊、视频**
等配套学习资源，循序渐进的方式引导加深安全攻防技术提高以及岗位内推等等服务。  

![](https://gitee.com/fuli009/images/raw/master/public/20220926130349.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220926130350.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220926130351.png)

dotNet安全矩阵知识星球 — 聚焦于微软.NET安全技术，关注基于.NET衍生出的各种红蓝攻防对抗技术、分享内容不限于 .NET代码审计、
最新的.NET漏洞分析、反序列化漏洞研究、有趣的.NET安全Trick、.NET开源软件分享、.
NET生态等热点话题、还可以获得阿里、蚂蚁、字节等大厂内推的机会.

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

