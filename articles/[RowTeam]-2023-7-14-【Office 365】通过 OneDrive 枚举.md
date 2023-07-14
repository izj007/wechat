#  【Office 365】通过 OneDrive 枚举

Rcoil  [ RowTeam ](javascript:void\(0\);)

**RowTeam** ![]()

微信号 RowTeam

功能介绍 没有什么好介绍的消遣地

____

___发表于_

收录于合集 #King 8个

## 0x00 前言

今天，我们来探讨如何通过 OneDrive 对 Office365 用户进行枚举。目前来说，OneDrive
可能是最适合用于用户枚举了，因为它有以下几个优点：

  1. 1. 未做登录尝试，也就是不需要像 Windows Exchange 那样，需要通过登录的方式进行判断。

  2. 2. 由于不需要登录，因此也不会产生日志。

  3. 3. 由于不需要登录，因此也没有速率等限制。

如果目标组织启用了 OneDrive，那么这就是一种最完美的枚举方法。但是，它也是有一个缺点的，这也会导致增加我们枚举的时间成本。

## 0x01 概述

OneDrive 是 SharePoint 的一部分。它专为个人文件存储而设计，并直接链接到 Azure/M365 帐户。每当用户登录到各种
Microsoft 服务（如 Excel 或 Word）时，都会激活 OneDrive，并创建包含用户电子邮件地址的个人 URL。更准确地说，个人 URL
实际上是帐户的 UPN 或用户主体名称。如下所示：

![](https://gitee.com/fuli009/images/raw/master/public/20230714182322.png)image.png

由于个人 URL 直接绑定到用户帐户，因此只需查找特定格式的 Web 目录即可枚举用户，类似于使用 DirBuster/dirb。

下面的图表显示了各种服务以及每个服务是否激活 OneDrive。

![](https://gitee.com/fuli009/images/raw/master/public/20230714182323.png)image.png

实际上，由于 OneDrive URL 创建的触发器数量众多，几乎任何实际使用 Azure/M365 帐户的人都将拥有 OneDrive URL。激活
OneDrive 后，将创建一个与该用户关联的唯一 URL。URL 采用以下格式：

    
    
    https://<tenant>-my.sharepoint.com/personal/<UserPrincipalName>/_layouts/15/onedrive.aspx

如下图所示，可以在图中看到 tenant
的名称为“36vqny”，用户主体名称为“rowteam@36vqny.onmicrosoft.com“的变体，点号（"."）会替换为下划线（“_”） 。

![](https://gitee.com/fuli009/images/raw/master/public/20230714182324.png)image.png

因此，可以通过这种方式，对该 URL 发出一个 HEAD 请求，就可以确定用户是否有效，判断依据也很简单：

  * • 状态码为 302，说明用户存在

当然，要确保该用户至少登录过一次才可以。没登陆过的用户是不可以通过这种方式进行判断的。

## 0x02 标识 AZURE 租户名称

但，想要使用 OneDrive 枚举，还需要知道 Azure 租户名称。Azure 租户名称是与 Azure 租户关联的短名称。

很多时候，组织的租户名称将与域名匹配。例如，"microsoft.com" 具有关联的租户
"Microsoft"。但实际上往往并非如此，有时它只是个替代名称，或组织法定全名的缩写。因此从域名直接截取的名称，不一定是对的，因此还需要识别 Azure
租户名称的方法。

但是，从 https://github.com/blacklanternsecurity/TREVORspray 这里的代码可以得到一个方法：向
`https://autodiscover-s.outlook.com/autodiscover/autodiscover.svc`发送请求。 具体如下

  * • 请求包

    
    
    POST /autodiscover/autodiscover.svc HTTP/1.1  
    Host: autodiscover-s.outlook.com  
    Accept: identity  
    Content-Length: 1188  
    Content-Type: text/xml; charset=utf-8  
    Expect: 100-continue  
    SOAPAction: "http://schemas.microsoft.com/exchange/2010/Autodiscover/Autodiscover/GetFederationInformation"  
    User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:49.0) Gecko/20100101 Firefox/49.0  
      
    <?xml version="1.0" encoding="utf-8"?>  
    <soap:Envelope  
        xmlns:exm="http://schemas.microsoft.com/exchange/services/2006/messages"  
        xmlns:ext="http://schemas.microsoft.com/exchange/services/2006/types"  
        xmlns:a="http://www.w3.org/2005/08/addressing"  
        xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xmlns:xsd="http://www.w3.org/2001/XMLSchema">  
        <soap:Header>  
            <a:Action soap:mustUnderstand="1">http://schemas.microsoft.com/exchange/2010/Autodiscover/Autodiscover/GetFederationInformation</a:Action>  
            <a:To soap:mustUnderstand="1">https://autodiscover-s.outlook.com/autodiscover/autodiscover.svc</a:To>  
            <a:ReplyTo>  
                <a:Address>http://www.w3.org/2005/08/addressing/anonymous</a:Address>  
            </a:ReplyTo>  
        </soap:Header>  
        <soap:Body>  
            <GetFederationInformationRequestMessage  
                xmlns="http://schemas.microsoft.com/exchange/2010/Autodiscover">  
                <Request>  
                    <Domain>36vqny.onmicrosoft.com</Domain>  
                </Request>  
            </GetFederationInformationRequestMessage>  
        </soap:Body>  
    </soap:Envelope>

  * • 响应内容

    
    
    HTTP/1.1 200 OK  
    Alt-Svc: h3=":443",h3-29=":443"  
    Cache-Control: private  
    Content-Type: text/xml; charset=utf-8  
    Date: Thu, 15 Jun 2023 05:15:10 GMT  
    Request-Id: 0606cbf2-d918-991d-70dc-9ecad9d1de7a  
    Server: Microsoft-IIS/10.0  
    X-Aspnet-Version: 4.0.30319  
    X-Backendhttpstatus: 200  
    X-Backendhttpstatus: 200  
    X-Beserver: PUZPR06MB4716  
    X-Calculatedbetarget: PUZPR06MB4716.apcprd06.prod.outlook.com  
    X-Calculatedfetarget: PS2PR02CU002.internal.outlook.com  
    X-Diaginfo: PUZPR06MB4716  
    X-Feefzinfo: SIN  
    X-Feproxyinfo: SI2PR06CA0016.APCPRD06.PROD.OUTLOOK.COM  
    X-Feserver: PS2PR02CA0033  
    X-Feserver: SI2PR06CA0016  
    X-Firsthopcafeefz: SIN  
    X-Proxy-Backendserverstatus: 200  
    X-Proxy-Routingcorrectness: 1  
    X-Rum-Notupdatequerieddbcopy: 1  
    X-Rum-Notupdatequeriedpath: 1  
    X-Rum-Validated: 1  
    Content-Length: 1188  
      
      
    <s:Envelope  
        xmlns:s="http://schemas.xmlsoap.org/soap/envelope/"  
        xmlns:a="http://www.w3.org/2005/08/addressing">  
        <s:Header>  
            <a:Action s:mustUnderstand="1">http://schemas.microsoft.com/exchange/2010/Autodiscover/Autodiscover/GetFederationInformationResponse</a:Action>  
            <h:ServerVersionInfo  
                xmlns:h="http://schemas.microsoft.com/exchange/2010/Autodiscover"  
                xmlns:i="http://www.w3.org/2001/XMLSchema-instance">  
                <h:MajorVersion>15</h:MajorVersion>  
                <h:MinorVersion>20</h:MinorVersion>  
                <h:MajorBuildNumber>6477</h:MajorBuildNumber>  
                <h:MinorBuildNumber>37</h:MinorBuildNumber>  
                <h:Version>Exchange2015</h:Version>  
            </h:ServerVersionInfo>  
        </s:Header>  
        <s:Body>  
            <GetFederationInformationResponseMessage  
                xmlns="http://schemas.microsoft.com/exchange/2010/Autodiscover">  
                <Response  
                    xmlns:i="http://www.w3.org/2001/XMLSchema-instance">  
                    <ErrorCode>NoError</ErrorCode>  
                    <ErrorMessage/>  
                    <ApplicationUri>outlook.com</ApplicationUri>  
                    <Domains>  
                        <Domain>36vqny.onmicrosoft.com</Domain>  
                    </Domains>  
                    <TokenIssuers>  
                        <TokenIssuer>  
                            <Endpoint>https://login.microsoftonline.com/extSTS.srf</Endpoint>  
                            <Uri>urn:federation:MicrosoftOnline</Uri>  
                        </TokenIssuer>  
                    </TokenIssuers>  
                </Response>  
            </GetFederationInformationResponseMessage>  
        </s:Body>  
    </s:Envelope>

然后从 `Domains`标签的域名列表中，找出以 `.onmicrosoft.com`结尾的域名。这样就可以获取到一个租户列表。下图中，显示租户只有一个
`36vqny`。

![](https://gitee.com/fuli009/images/raw/master/public/20230714182325.png)image.png

对 OneDrive Hosts 简单发起一个请求，如果返回 404，那么表示域名存在。

## 0x03 使用 OneDrive 枚举用户

上个章节，已经成功获取到租户名称，那么来尝试对用户进行枚举。它的原理很简单，在我们的例子中，需要以下条件：

  1. 1. 域名：36vqny.onmicrosoft.com

  2. 2. 租户名称：36vqny

  3. 3. 用户名：rowteam

需要向以下链接中发起 Get 请求

    
    
    https://36vqny-my.sharepoint.com/personal/rowteam_36vqny_onmicrosoft_com/_layouts/15/onedrive.aspx

如果返回 302 状态码，代表用户存在。

那么按照 RowTeam 的编写习惯，输入的格式应该是 `36vqny\rowteam`。 效果如下所示：

![](https://gitee.com/fuli009/images/raw/master/public/20230714182326.png)image.png

## 0x04 其他情况

在 OneDrive 枚举中，租户名称和域的确切组合很重要。正常情况下，大多数组织只会定义 1
个租户名称，也就是一个域仅有一个租户名称，那么该工具在识别它时不会有任何问题。但是，对于是多租户设置，那么我们必须做选择（选择租户/域的一个或所有组合）。

打个比方吗，如果“rowteam.me”有 3
个租户名称：“acmeRowTeam”、“acmeEurope”和“acmeAPC”，则用户可以存在于这些租户名称中的任何一个中。因此在枚举的时候，可能出现的组合为：

租户名称| 邮箱用户| OneDrive Host  
---|---|---  
acmeRowTeam| rowteam@rowteam.me| acmeRowTeam-my.sharepoint.com  
acmeEurope| rowteam@rowteam.me| acmeEurope-my.sharepoint.com  
acmeAPC| rowteam@rowteam.me| acmeAPC-my.sharepoint.com  
  
以上这些情况，你需要对 rowteam 用户使用不同的 OneDrive Host 进行测试，哪个成功就是哪个。很简单，拿 Microsoft 来举个例子

![](https://gitee.com/fuli009/images/raw/master/public/20230714182327.png)

成功找到 4 个 OneDrive Host，那么你在枚举用户的时候，就需要枚举 4
次，如果运气好，第一个就出来结果了，但当前工具的编写并非那么智能，需要自己手动输入租户名称。

当然还有其他情况：如果该组织对特定国家或地区使用的是特定的域，那么这可能会变得更加复杂。当然，只是更加复杂，但这也不是个大问题。因此，使用 OneDrive
枚举用户，它有利有弊，如果遇到这些额外的情况，就会增加我们的枚举时间。

最后，透露一条有关 OneDrive 枚举的附加信息。当尝试连接到 OneDrive 或 SharePoint 主机（例如“36vqny-
my.sharepoint.com”）时，如果收到“403”或“401”错误响应。 当用户名返回“401”时，表示 SharePoint
已配置为需要新式身份验证。 （这专门针对 SharePoint，而不是整个组织。

  * • 不需要新式身份验证的租户示例

![](https://gitee.com/fuli009/images/raw/master/public/20230714182328.png)image.png

  * • 需要新式身份验证的租户示例

![](https://gitee.com/fuli009/images/raw/master/public/20230714182329.png)image.png

可在此处找到 SharePoint 中新式身份验证的相关设置：

![](https://gitee.com/fuli009/images/raw/master/public/20230714182330.png)image.png

当然，这对于我们的枚举并没有任何的影响。

## 0x05 参考

  1. 1. https://www.trustedsec.com/blog/onedrive-to-enum-them-all/

  2. 2. https://github.com/nyxgeek/onedrive_user_enum

  3. 3. https://github.com/blacklanternsecurity/TREVORspray

  

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

