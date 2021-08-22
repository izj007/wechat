##  关于Swagger-UI下的渗透实战

点击关注☞  [ 网络侦查研究院 ](javascript:void\(0\);)

**网络侦查研究院** ![]()

微信号 PCpolicesir

功能介绍 服务全国情报侦查人员，培养网络情报思维，提高网络情报侦查能力。

____

__

收录于话题

![](https://gitee.com/fuli009/images/raw/master/public/20210822090925.png)![](https://gitee.com/fuli009/images/raw/master/public/20210822090926.png)

  

# 前言

最近在测试时频繁遇到Swagger UI，趁着有空，就记录下

# 如何发现Swagger UI

正常情况下，当访问一个网页出现以下错误，可根据报错回显看出这是典型Spring的接口报错信息

![]()

另外Spring站点的默认logo是片小绿叶，可以观察下报错页的logo有没有小绿叶，或者拼接favicon.ico访问，如果出现以下图标，则有很大可能是Swagger
UI的站

![](https://gitee.com/fuli009/images/raw/master/public/20210822090927.png)

如果这些都无法实现，则可用字典去遍历Swagger UI路径，当一级目录不存在时，可尝试拼接二级目录进行访问

常见路径

    
    
    /v2/api-docs  
    /swagger-ui.html  
    /swagger  
    /api/swagger  
    /Swagger/ui/index  
    /api/swaggerui  
    /swagger/ui  
    /api/swagger/ui  
    /api/swagger-ui.html  
    /user/swagger-ui.html  
    /libs/swaggerui  
    /swagger/index.html  
    /swagger-resources/configuration/ui  
    /swagger-resources/configuration/security  
    /api.html  
    /druid/index.html  
    /sw/swagger-ui.html  
    /api/swagger-ui.html  
    /template/swagger-ui.html  
    /spring-security-rest/api/swagger-ui.html  
    /spring-security-oauth-resource/swagger-ui.html  
    /swagger/v1/swagger.json  
    /swagger/v2/swagger.json  
    /api-docs  
    /api/doc  
    /docs/  
    /doc.html  
    /v1/api-docs  
    /v3/api-docs

扫目录建议使用burp，通过返回包可看到完整的数据

![](https://gitee.com/fuli009/images/raw/master/public/20210822090928.png)

某些时候，swagger-ui.html会被禁止访问，这时候可以尝试拼接/v2/api-
docs进行访问，如果一级目录404，可以尝试拼接二级目录访问，以此类推。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090929.png)

![]()

另外，有些时候在js和HTML源码里也能发现Swagger UI的身影，具体怎么去做，还是看个人的思路和站点架构而定。

# 漏洞挖掘

由于Swagger UI接口中有详细的参数介绍，所以可以直接在swagger-
ui.html页面构造参数发包，如果该接口没有权限验证，则会造成严重的安全隐患。

发包示例：

先点击try is out，然后构造参数，excute可直接发包

![](https://gitee.com/fuli009/images/raw/master/public/20210822090930.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210822090931.png)

由于Swagger UI的安全性问题，延伸出了几个常见的漏洞

## 文件上传漏洞

类似于Swagger
UI这种上传接口，如果在部署时没做限制，一般都是任意文件上传的，当然，我们可以找一些temp、test这类上传接口，因为此类接口多数是开发过程中用作测试的，这种接口几乎都是无限上传文件类型的。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090932.png)

![]()

实战的时候遇到过一种情况，先是扫目录扫出一个uploadtest上传接口，在本地构造表单进行上传，通过fuzz找到了上传参数，但是上传成功却没返回路径，只返回了一个file_id号，根据这个id号猜测后端是以id+上传文件的绝对路径保存在数据库中，之后通过挖掘SQL注入，利用sqlnmap的
--search -C参数找到了文件路径。

## 用户身份信息泄露

某次测试的时候在Swagger
UI发现了一个api接口存在未授权访问，只要知道手机号就可得到用户身份证、真实姓名等信息，且此处存在用户名枚举漏洞，因为后台登录功能采用了手机号+身份证+密码登录的形式，所以我们可针对该api编写脚本进行利用，提取身份证号，和手机号，为后续爆破后台做准备。  
![](https://gitee.com/fuli009/images/raw/master/public/20210822090933.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210822090934.png)  
![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210822090935.png)

然后利用提取出来的api信息构造爆破字典成功登录后台  
![]()  
遇到过很多的Swagger
UI接口，很多时候其api都是裸奔的，而不少的api往往都包含着大量的敏感信息，这时候我们就可利用其缺陷，进行更深入的渗透，来把漏洞危害最大化。

## sql注入

关于注入遇到的比较少，我去fofa找了几个Swagger UI复现了下，发现跟普通注入都大同小异，只要得到注入点就能利用了

![](https://gitee.com/fuli009/images/raw/master/public/20210822090936.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210822090937.png)

## 任意文件下载

这个漏洞在Swagger
UI里也是非常常见的，可直接在API文档里搜索关键字，如：downLoad、filename、path等，之后就是构造参数测试了，这个就不多说了。

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210822090938.png)

## 实战举例

以近期做的一次项目举例。

这是个复测的站点，站点存在waf，仅有一个登录页，验证码有效，且有多因子认证，同事在第一次测试的时候是无漏洞情况。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090939.png)

![]()

接手了这个项目，简单的信息收集一番后便开搞了，因为时间关系，便迅速展开测试了。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090940.png)

站点只有一个登陆页，存在waf，且验证码有效，登陆多因子认证，经过端口探测和前端漏洞挖掘没发现突破点，后来翻查了js也没能发现有用的东西，正在一筹莫展的时候，一个报错也引起了我的注意。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090941.png)根据回显可以看出这是典型Spring
Swagger的接口报错信息，拼接下/favicon.ico，看到了熟悉的logo

  

之后都不带考虑的，直接拼接/v2/api-docs访问，然后就得到了一个Swagger-UI api文档

![]()

由于这个json接口不能直接构造参数发包，为了测试便捷性，尝试利用swagger-ui.html进行参数传递，访问/swagger-
ui.html，但该页面做了限制，无法访问到，使用/%20/swagger-ui.html进行访问，但也失败了，之后使用字典去遍历swagger-
ui路径，但也只得到了这个页面，没办法，还是得回到jsonAPI进行利用。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090942.png)

### 未授权漏洞

通过翻查文档，得到一个查询用户的api接口，点击parameters，即可得到该api接口的详细参数

![](https://gitee.com/fuli009/images/raw/master/public/20210822090943.png)

有了参数就好搞了，直接构造参数发包，开始之前，把无痕模式打开，以证明可以未授权访问，通过回显可以看到，得到了大量的用户信息，包含了手机号，邮箱等等。

    
    
    /findxxxs.action?start=1&limit=99

  

虽然得到了部分用户信息但却没包含密码，构不成什么威胁，之后再次翻找和用户有关的api，但也没多大效果。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090944.png)

![]()

之后发现了个用户枚举漏洞，由于是API，所以不用验证码验证，经过短暂的爆破，得到了几个存在的用户，但并没什么作用，因为在findxxs.action
API中也能获取这些用户名，无奈只能寻找其它的api做突破点

### 任意用户密码重置

进一步查找API发现了个密码重置API，由于这种api测试起来危害较大，为了自身安全，取得客户同意后，便进一步开始测试了。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090945.png)

经过测试，发现可利用泄露的api接口信息，构造userId参数来重置任意后台用户密码，且该api也是可未授权访问的，这样一来，漏洞危害便大大增加了。

利用api接口未授权重置userid1，userid2登录密码，此时我们还不知道userid1和userid2的用户名是什么，这时候就要用上/findxxxs.action
API接口了，因为该接口回显的用户信息包含了userid，可以根据这个userid去寻找用户名

![](https://gitee.com/fuli009/images/raw/master/public/20210822090946.png)

![]()

利用/findxxxs.action API接口寻找username

![](https://gitee.com/fuli009/images/raw/master/public/20210822090947.png)

前面说到了该后台存在多因子认证的，这时候就算我们重置了用户密码，但是还是无法登陆后台的，想要登录后台，就必须把用户的手机号给重置掉，或者寻找注册API注册登陆。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090948.png)

### 任意用户信息修改

接下来找到了一个修改用户参数的API，根据api接口信息构造参数，尝试利用未授权漏洞重置用户手机号登录后台

![]()

构造参数发包，可以看到修改成功了，然后到登陆页输入重置后的账号密码登录，却发现重置后的手机号怎么都无法收到信息，后来利用API重置为另一个手机号，重新登录获取短信，却也是毫无反应，刚开始以为是重置后的手机号需要等待一会才能生效，后来等了大半个小时，再次重新获取短信，却也是毫无反应，到此，也不得不放弃重置手机号进后台这个突破点了。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090949.png)

### 用户身份信息泄露

由于进不去后台，只能尽量在多挖掘漏洞证明其危害了，通过api漏洞挖掘，得到了一个未授权接口，能未授权获取到大量用户身份证号等敏感数据。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090950.png)

再进一步挖掘，发现大量API可未授权访问。随后利用未授权获取了大量合同数据，最大化了漏洞危害，到此，本次测试也告一段落了。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090951.png)

## 其它场景

大多数时候api是会存在认证限制，这时候我们可以寻找其它的api进行测试，Swagger
UI大多会存在数量庞大的API接口，除非所有的接口都存在认证限制，否则一旦有漏网之鱼，就能被我们所利用，可以重点关注test、temp之类接口。

![]()

当得到一个存在权限认证的Swagger UI接口时，可关注下Select a
spec，因为有些时候会存在多个接口文档，有可能当前API存在权限校验，而另一个API文档则裸奔的情况。

![](https://gitee.com/fuli009/images/raw/master/public/20210822090952.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210822090953.png)

存在spring框架的站点要关注下druid，因为Springboot会集成Druid，多半开发会混合着使用，在挖掘Swagger
UI的同是也不要忘记关注druid未授权的突破点。

此前也写过关注druid的漏洞利用，可参考druid未授权利用

# 总结

渗透需要细心，不能放错任何一个细节，或许一个小小的报错页面，就能给予我们突破的灵感，本文仅介绍了Swagger UI，类似的接口并不少，如web-
api、springtoot-cli api等等，这类接口或多或少都存在问题，对此应该如何防御与挖掘呢？

![](https://gitee.com/fuli009/images/raw/master/public/20210822090926.png)

  

  

 **END**

![](https://gitee.com/fuli009/images/raw/master/public/20210822090954.png)

  

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

关于Swagger-UI下的渗透实战

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

