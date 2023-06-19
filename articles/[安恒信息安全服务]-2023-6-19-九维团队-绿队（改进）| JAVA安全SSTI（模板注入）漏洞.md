#  九维团队-绿队（改进）| JAVA安全SSTI（模板注入）漏洞

原创 bingo  [ 安恒信息安全服务 ](javascript:void\(0\);)

**安恒信息安全服务** ![]()

微信号 AHXXsecurityservice

功能介绍 发布网络安全行业的热点资讯，对于网络安全服务的技术与案例呈现，安恒信息原创的安全服务彩虹架构应对网络黑客。

____

___发表于_

收录于合集

#九维技术团队 221 个

#绿队 47 个

![](https://gitee.com/fuli009/images/raw/master/public/20230619163526.png)

前言

SSTI（Server-Side Template
Injection）漏洞，也被称为模板注入漏洞。SSTI漏洞是一种在服务器端模板引擎中注入恶意代码的攻击方式。攻击者通过注入可执行代码，可以在服务器端执行任意命令，造成安全漏洞。  

  

  

 **一、java常见的模板引擎**

  

 **  
** **1.FreeMarker：**  
FreeMarker是一种流行的模板引擎，广泛应用于Java
Web开发中。它采用了类似于JSP的语法，并提供了许多丰富的内置函数和标签，使得模板编写变得十分简单和灵活。

  

 **2.Thymeleaf：**  
Thymeleaf是一种新兴的模板引擎，它可以应用于Java、Spring等多个开发框架中。与其他模板引擎不同的是，Thymeleaf支持自然模板的语法，可以在模板中直接嵌入表达式和变量，提高了模板编写的效率和可读性。

  

 **3.Velocity：**  
Velocity是另一种常见的模板引擎，它也广泛应用于Java Web开发中。与其他模板引擎不同的是，Velocity的语法十分简单，使用起来非常方便。

  

  

 **二、Freemarker  SSTI示例**

  

  
这里我们通过一个Freemarker SSTI示例来演示其攻击原理和防范方法。

  
Freemarker模板示例代码：

  *   *   *   *   *   *   *   *   *   * 

    
    
    <!DOCTYPE html><html><head>    <meta charset="UTF-8">    <title>${title}</title></head><body>    <h1>${message}</h1></body></html>

  

其中是模板引擎中的一个表达式，用于在模板中显示后端传递给模板的变量的值。我们需要在后端代码中将一个变量赋值给模板中的是FreeMarker模板引擎中的一个表达式，用于在模板中显示后端传递给模板的变量的值。我们需要在后端代码中将一个变量赋值给模板中的{}表达式，然后在模板渲染时，该表达式会被解析和渲染为变量的实际值。

  

![](https://gitee.com/fuli009/images/raw/master/public/20230619163528.png)

  

  

此外，Freemarker模板还支持内置函数，所以我们可以通过freemarker.template.utility
里的一个Execute类构造恶意代码如下：

  *   * 

    
    
    <#assign value="freemarker.template.utility.Execute"?new()> ${value("cmd /c calc.exe")}

*左右滑动查看更多  

  

通过调用Execute类的new方法来创建一个Execute对象。这个Execute对象可以用来执行任意的系统命令。

  

下面我们通过代码演示一下真实环境下SSTI：

  

创建spring boot项目并在pom文件添加依赖：

  *   *   *   * 

    
    
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-freemarker</artifactId></dependency>

  

*左右滑动查看更多

  

后端代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.example.demo;  
    import freemarker.cache.StringTemplateLoader;import freemarker.core.TemplateClassResolver;import freemarker.template.Configuration;import freemarker.template.Template;import freemarker.template.TemplateException;import org.springframework.stereotype.Controller;import org.springframework.web.bind.annotation.RequestBody;import org.springframework.web.bind.annotation.RequestMapping;import java.io.IOException;import java.io.StringWriter;import java.util.LinkedHashMap;import java.util.Map;  
    @Controllerpublic class SSTIdemo {    @RequestMapping("/Template")    public String importTemplate (@RequestBody String a) throws IOException, TemplateException {        Configuration configuration = new Configuration(Configuration.getVersion());        StringTemplateLoader stringTemplateLoader = new StringTemplateLoader();        Template template;        Map data = new LinkedHashMap();        StringWriter stringWriter = new StringWriter();  
            stringTemplateLoader.putTemplate("test.ftl", a);// 将模板内容添加到StringTemplateLoader中        configuration.setTemplateLoader(stringTemplateLoader);  
            template = configuration.getTemplate("test.ftl");        template.process(data, stringWriter);//渲染模板        System.out.println(stringWriter.toString());        return stringWriter.toString();    }}

*左右滑动查看更多

  

构造post请求加上之前构造的payload：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163529.png)  
  
---  
  
可以看到恶意代码已被执行：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163530.png)

  

  

  

 **三、过程分析**

  

  

我们来分析下执行过程，模板是通过process方法被加载的：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163531.png)

  

跟进process方法：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163532.png)

  
跳转到createProcessingEnvironment方法：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163533.png)

  
经过判断处理后最终返回了一个Environment对象，并将我们的Template对象（包含恶意payload）传入。

![](https://gitee.com/fuli009/images/raw/master/public/20230619163534.png)

  

回到process，又调用返回Environment对象的process方法，跟进：  

![](https://gitee.com/fuli009/images/raw/master/public/20230619163535.png)

  
又调用了visit方法将模板（payload）传入，跟进visit方法：

![]()

  

这里遍历读取模板的节点，读取为对象list，<#assign
value="freemarker.template.utility.Execute"?new()>为Assignment对象，进入了Assignment对象accept方法：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163536.png)

  
跟进 eval() 方法：  

![](https://gitee.com/fuli009/images/raw/master/public/20230619163537.png)

  
再跟进到_eval方法：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163538.png)

  

跟进到exec方法，通过freemarker.core.NewBI实例化了我们传入的freemarker.template.utility.Execute类：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163539.png)

  

回到freemarker.core.Environment，遍历到${value("cmd /c calc.exe")：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163540.png)

  

继续调用了DollarVariable的accept方法：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163542.png)

  
跟进calculateInterpolatedStringOrMarkup方法：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163543.png)

  

进入了和上个循环一样的流程,eval——>-_eval。

![](https://gitee.com/fuli009/images/raw/master/public/20230619163544.png)

  

然后调用了Execute对象的exec方法：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163545.png)

  

至此，可以看到poc已经成功被执行。

  

  

 **四、防御建议及小结**

  

  

 **1、防御方法**

 **  
**

从2.3.17版本开始使用

Configuration.setNewBuiltinClassResolver(TemplateClassResolver)

或new\\_builtin\\_class\\_resolver设置来限制内置函数new对类的访问。

  

此处官方提供了三个预定义的解析器：

1）UNRESTRICTED\\_RESOLVER：

简单地调用ClassUtil.forName(String)。

  

2）SAFER\\_RESOLVER：

和第一个类似，但禁止解析ObjectConstructor，Execute和freemarker.template.utility.

JythonRuntime。

  

3）ALLOWS\\_NOTHING\\_RESOLVER：

禁止解析任何类。

  

  

我们可以通过设置来防止类加载：

  * 

    
    
    configuration.setNewBuiltinClassResolver(TemplateClassResolver.ALLOWS_NOTHING_RESOLVER);

*左右滑动查看更多

  

再次发送payload可以看到模板已禁止实例化Execute类：

![](https://gitee.com/fuli009/images/raw/master/public/20230619163546.png)

  

 **2、小结**

Freemarker解析模板过程中可以内置函数创建对象，因此在存在加载模板可控的地点，可以通过构造利用Freemarker模板中的freemarker.template.utility.Execute类来执行命令。同时，在开发使用过程中应尽量避免用户直接对模板进行修改。

  

  

 **五、安全开发建议**

  

  

针对SSTL（模板注入）类模板，在开发使用过程中应尽量避免用户直接对模板进行修改，以下是安全开发的一些建议：

  

 **  1、输入验证：**

对所有用户输入进行验证，确保其符合预期格式，并过滤掉任何恶意字符。可使用正则表达式进行此操作。

  

 **  2、最小权限原则：**

确保模板解释器仅具有最低权限来运行。这样可以限制潜在攻击的影响范围。

  

 **  3、限制模板访问：**

仅允许授权用户访问模板编辑功能。这样可以防止未经授权的用户恶意修改模板。

  

 **  4、定期更新：**

确保模板引擎和相关库保持最新，以修复任何已知的安全漏洞。

  

 **  5、审计日志：**

记录所有对模板的更改和访问尝试，以便进行审计和追踪。

  

 **  6、逻辑分离：**

尽可能将模板中的逻辑与渲染分离。这样可以使代码更易于维护，并减少遭受基于模板的攻击的风险。

  

* * *

 **往期回顾**

[![](https://gitee.com/fuli009/images/raw/master/public/20230619163559.png)](http://mp.weixin.qq.com/s?__biz=MzAwMDgyNTQzMQ==&mid=2247503784&idx=2&sn=bc990c6259149fdf93ba39e2bbd0f898&chksm=9ae18e90ad960786ff7b082ba3d2afcfe716606214a3c6ad6929bdd2f8d437e14383a619f490&scene=21#wechat_redirect)

[![]()](http://mp.weixin.qq.com/s?__biz=MzAwMDgyNTQzMQ==&mid=2247519863&idx=1&sn=9495648b103e724983ad338384c57208&chksm=9ae1cf4fad96465927d4b7934653823ddfc71c8902a9abef8cdb33a681072b790c4c334511da&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20230619163600.png)](http://mp.weixin.qq.com/s?__biz=MzAwMDgyNTQzMQ==&mid=2247519826&idx=1&sn=37a4b9f93f337b294a2e249a44417d1d&chksm=9ae1cf6aad96467c624fdfb771634a36c49d379d7e638d37d36ae22705a768cb5e3f842ba73d&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20230619163601.png)](http://mp.weixin.qq.com/s?__biz=MzAwMDgyNTQzMQ==&mid=2247518401&idx=1&sn=af2f81c299b3e6720df3274ddf8239d1&chksm=9ae1c9f9ad9640ef67da5b58cdacf0affb1dbd0d0e673852081e39e13c1191c3054d541d6826&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20230619163602.png)](http://mp.weixin.qq.com/s?__biz=MzAwMDgyNTQzMQ==&mid=2247518134&idx=1&sn=38629736d9e1cbf411826680c4a20bd5&chksm=9ae1c68ead964f98fca82d803096a9c5d401bb0019fdd15dbde8cd801e73abf14db8309009f8&scene=21#wechat_redirect)

![](https://gitee.com/fuli009/images/raw/master/public/20230619163603.png)

  
 **关于安恒信息安全服务团队**
**安恒信息安全服务团队由九维安全能力专家构成，其职责分别为：红队持续突破、橙队擅于赋能、黄队致力建设、绿队跟踪改进、青队快速处置、蓝队实时防御，紫队不断优化、暗队专注情报和研究、白队运营管理，以体系化的安全人才及技术为客户赋能。**
****

![](https://gitee.com/fuli009/images/raw/master/public/20230619163605.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230619163606.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230619163607.png)

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

