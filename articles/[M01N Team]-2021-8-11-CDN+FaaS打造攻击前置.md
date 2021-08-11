##  CDN+FaaS打造攻击前置

原创 天元实验室  [ M01N Team ](javascript:void\(0\);)

**M01N Team** ![]()

微信号 m01nteam

功能介绍 攻击对抗研究分享

____

__

收录于话题

#蓝军

5个

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20210811182715.png)

**概述**

随着攻防对抗形势的白热化发展，对抗升级的过程不断对OPSEC提出更高的要求。现有CDN、域前置、云函数转发等手段进行C2保护已经具备一定的能力，但也同时存在一些缺陷，本文将结合CDN和FaaS探讨C2保护新的实现过程。

  

 **01** OPSEC之基础设施  

OPSEC（Operational
Security，即操作安全），类似“红蓝对抗”，也源自现代军事对抗实践，指审视己方操作中可能被敌方利用的关键信息，评估风险，并采取有效应对措施的过程。简单的说，蓝军作战期间同样须要有自己的安全生产标准和规范，以规避行动暴露、被阻断以及被溯源等安全风险。

  

作战基础设施是为了支撑保障完成攻击全生命周期而存在的体系化辅助工具，可以理解为攻击者的阵地战壕，需要有绵亘正面和一定纵深的特点，在当作防线掩体的同时又可以用来灵活调整战术和作战状态。所以搭建一个好的基础设施也是完善OPSEC的关键一步。C2作为基础设施中的重型作战通道，一般会出于两个基础目的使用前置技术对其进行保护，首先前置会保障C2不被定位到真实的网络地址，其次防守方在进行攻击源封堵时，前置地址可以被用来当作C2的“替死鬼”，因为比较轻量可以快速进行攻击出口的切换以降低攻击者重新部署C2或切换C2网络地址的成本。当然一个更好的C2前置技术还可能具备绵亘、伪装、监控等特点。那如何搭建一个高质量的C2攻击前置的基础设施，是我们接下来要重点探讨的内容。

  

  

 **02  **CDN or FaaS

域前置（Domain
Fronting）技术在诞生之日起就被大量的用于攻击基础设施当中，因为其滥用CDN服务加速了原本不存在的可信根域名的子域名，在保障真实C2地址不被定位的同时，可信根域名又增加流量来源的可信度，并且数量众多的CDN节点任意一个都可作为前置来转发攻击流量。而在防守方视角遇到这种情况首先会发现恶意通信地址不是固定的，被控机器会随机与一些CDN节点完成通信，所以封禁少量CDN节点地址是解决不了问题的，大面积封禁CDN节点又可能会影响正常业务，从而陷入两难境地。国际上的一些知名CDN服务商早已对域前置进行了限制，而在国内因为近两三年的演练中蓝军也大量使用该技术，一些规模较大的CDN服务商也已做出逻辑校验来限制服务被滥用的情况。

  

另外，常规的CDN业务也可用于攻击前置，而与Domain
Fronting相比唯一的区别是所使用域名是攻击者自己注册的或者控制的，而不是被滥用加速实则不存在的可信子域名。总的来说两种方法都能很好的满足C2保护的前置需求，如果非说有哪些不足的话，可能包含：

  * 所使用的域名是最显性的特征可被防守方用于标记拦截攻击流量

  * 在防守方进行探测溯源攻击基础设施的时候，无法很好的进行监测、伪装和反制

  * 域前置被CDN厂商进行限制

  * ……

  

相对CDN前置的技术，近两三年随着各大云厂商推广各种形式的Serverless服务，攻击者又转而对其中一种称之为云函数（FaaS，Function as a
Service）或函数计算的服务进行滥用，比如微软云Azure Functions、亚马逊云AWS
Lambda，国内阿里云函数计算FC、腾讯云云函数等。云函数是一段运行在云端的、轻量的、无关联的、并且可重用的代码。无需管理服务器，只需编写和上传代码，即可获得对应的数据结果。使用云函数可以使企业和开发者不需要担心服务器或底层运维设施，可以更专注代码和业务本身，也可以使代码进一步解耦，增加其重用性。简单来说就是云厂商提供了一个供代码运行的容器方便开发者便捷部署并运行应用程序，而攻击者则利用其编写了代码对攻击流量进行了转发，完成攻击前置的目的。滥用云函数作为攻击前置的方法早在2018年就被提出，2020年底又在国内火了一把，主要原因有两点，一是云函数作为一种Serverless的服务，会有一个API网关触发器配合进行使用，该网关提供了一个厂商备案过的特定域名，具备高可信度的同时又具备IP不唯一的特点；二是云厂商在推广Serverless服务时，免费提供了一定的运行空间和请求次数。但同样也因此有一些不足，比如：

  * 云函数地址及请求响应包header中包含大量特征，可被标记检测的同时也存在被溯源的可能性

  * 云函数的免费请求次数乍一看是降低了成本，但其本身并不具备请求控制的能力，所以成本是不可控的。如果有防守方再遇到直接用云函数进行攻击前置的攻击队，这里建议您可以对准该函数地址进行CC攻击

  * ……

  

  

 **03  **CDN and FaaS

基于以上所述的一些问题，我们尝试利用CDN结合函数计算搭建攻击前置，希望能有1+1>>2的效果，基本结构如下。

![](https://gitee.com/fuli009/images/raw/master/public/20210811182716.png)

  

我们以国内某云厂商之前的版本为例简单阐述搭建步骤以验证效果，过程中间存在一些不完美的地方，读者可自行进行研究完善。

  

首先创建服务和函数

![](https://gitee.com/fuli009/images/raw/master/public/20210811182717.png)

  

修改触发器

![](https://gitee.com/fuli009/images/raw/master/public/20210811182718.png)

  

    
    
     1ROSTemplateFormatVersion: '2015-09-01'  
     2Transform: 'xxxyun::Serverless-2018-04-03'  
     3Resources:  
     4  redirect:  
     5    Type: 'xxxyun::Serverless::Service'  
     6    Properties:  
     7      InternetAccess: true  
     8    rcs:  
     9      Type: 'xxxyun::Serverless::Function'  
    10      Properties:  
    11        Handler: index.main  
    12        Runtime: python3  
    13        Timeout: 60  
    14        MemorySize: 512  
    15        InstanceConcurrency: 1  
    16        EnvironmentVariables: {}  
    17        CodeUri: ./redirect/rcs  
    18      Events:  
    19        defaultTrigger:  
    20          Type: HTTP  
    21          Properties:  
    22            AuthType: anonymous  
    23            Methods:  
    24              - GET  
    25              - POST  
    

  

  

编写函数代码完成请求响应转发功能

    
    
     1# -*- coding: utf-8 -*-  
     2import logging  
     3import urllib.parse  
     4import urllib.request  
     5  
     6HTTP_STATUS = {  
     7    "200": "OK",  
     8    "403": "Forbidden",  
     9    "404": "Not Found",  
    10    "500": "Internal Server Error"  
    11}  
    12  
    13def main(environ, start_response):  
    14    header_dict = {}  
    15  
    16    html = "hello guy!".encode("utf-8")  
    17    try:  
    18      get_url = 'http://8.8.8.8' + environ['PATH_INFO'] + "?" + environ["QUERY_STRING"]   
    19    except KeyError:  
    20      get_url = 'http://8.8.8.8' + environ['PATH_INFO']   
    21  
    22    try:  
    23        request_body_size = int(environ.get('CONTENT_LENGTH', 0))  
    24    except ValueError:  
    25        request_body_size = 0  
    26  
    27    request_body = environ['wsgi.input'].read(request_body_size)  
    28  
    29    return_body = []  
    30    for key, value in environ.items():  
    31        # return_body.append("{} : {}".format(key, value))  
    32        if key.startswith('HTTP_'):  
    33            header_key = key[5:]  
    34            header_dict[header_key] = value  
    35  
    36    try:  
    37        if environ['REQUEST_METHOD'] == 'GET':  
    38            request = urllib.request.Request(get_url, headers=header_dict)  
    39  
    40        else:  
    41            request = urllib.request.Request(get_url, headers=header_dict, data=request_body)  
    42  
    43        response = urllib.request.urlopen(request)  
    44        html = response.read()  
    45        headers = []  
    46  
    47        for key, value in dict(response.headers).items():  
    48            headers.append((key, value))  
    49  
    50        status_code = str(response.status)  
    51        start_response(status_code + " " + HTTP_STATUS[status_code], headers)  
    52  
    53    except:  
    54        html = "hello guy!".encode("utf-8")  
    55  
    56    return [html]  
    

  

invoke执行测试

![](https://gitee.com/fuli009/images/raw/master/public/20210811182719.png)

  

或者使用vscode插件进行远程部署

![](https://gitee.com/fuli009/images/raw/master/public/20210811182720.png)

  

访问返回的服务url即可重定向到我们的C2服务，云函数具备多出口IP，但URL特征极为明显，这里再通过自定义域名加上CDN服务做域前置进行优化

![](https://gitee.com/fuli009/images/raw/master/public/20210811182721.png)

  

请求测试，功能正常，但是返回头中含有云函数的一些特定内容，继续优化，可以在阿里云CDN中设置缓存配置过滤返回头无关内容

![](https://gitee.com/fuli009/images/raw/master/public/20210811182722.png)![](https://gitee.com/fuli009/images/raw/master/public/20210811182723.png)

  

删除以下HTTP响应头

    
    
     1Access-Control-Expose-Headers  
     2X-Fc-Code-Checksum  
     3X-Fc-Invocation-Duration  
     4X-Fc-Invocation-Service-Version  
     5X-Fc-Max-Memory-Usage  
     6X-Fc-Request-Id  
     7Ali-Swift-Global-Savetime  
     8Via  
     9X-Cache  
    10X-Swift-SaveTime  
    11X-Swift-CacheTime  
    12Timing-Allow-Orgin  
    13EagledId  
    14X-Fc-Error-Type  
    15X-Fc-Traffic-Type  
    

  

对于C2
WEBServer中不存在的资源的访问，中转请求肯定会报404，但阿里云函数自身异常处理有问题，返回会报50x错误，我们针对此情况在CDN配置自定义页面即可，同时可增加一部分伪装性

![]()

  

C2使用自定义的C2 Profile还可以用CDN的UA控制进行访问控制

![](https://gitee.com/fuli009/images/raw/master/public/20210811182724.png)

  

最后效果

![](https://gitee.com/fuli009/images/raw/master/public/20210811182725.png)

  

  

 **04  **总结

CDN结合云函数作为攻击前置存在哪些优势呢，我们总结如下：

  * 两次转发保护，多重隐蔽性

  * 存在绕过厂商限制添加域前置的可能性

  * 避免C2请求地址是函数地址，过滤请求header头中带有的云函数特征

  * 对非C2流量及潜在攻击流量进行过滤（例如针对云函数服务的DOS攻击）

  * 在CDN上定义响应状态码跳转进行基础设施保护

  * 特殊路径可通过CDN和其他服务结合的方式进行请求分发跳转，增强基础设施在伪装性、探测监控及溯源反制等方面的能力

  

随着攻防对抗形势的白热化发展，对抗升级的过程愈发对OPSEC提出了更高的要求，而云的高速发展为业务数字化转型提供便利的同时，也方便了攻击者滥用其能力去完成高隐蔽和持续性的攻击。本文所述部分方法在截止本文发出时已被个别云厂商所修复，此次分享旨在为蓝军技术研究人员拓展思路，而云上其他各类型服务根据场景依旧存在被滥用的可能性，值得攻防研究人员和云厂商跟进关注。

  

参考

https://fortynorthsecurity.com/blog/azure-functions-functional-redirection/

https://blog.xpnsec.com/aws-lambda-redirector/

  

![](https://gitee.com/fuli009/images/raw/master/public/20210811182726.png)

 **绿盟科技M01N战队** 专注于Red
Team、APT等高级攻击技术、战术及威胁研究，涉及Web安全、终端安全、AD安全、云安全等相关领域。通过研判现网攻击技术发展方向，以攻促防，为风险识别及威胁对抗提供决策支撑，全面提升安全防护能力。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210811182727.png)

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

CDN+FaaS打造攻击前置

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

