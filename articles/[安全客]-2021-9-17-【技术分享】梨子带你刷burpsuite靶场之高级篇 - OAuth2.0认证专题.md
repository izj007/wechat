#  【技术分享】梨子带你刷burpsuite靶场之高级篇 - OAuth2.0认证专题

原创 NaTsUk0  [ 安全客 ](javascript:void\(0\);)

**安全客** ![]()

微信号 anquanbobao

功能介绍 打破黑箱 客说安全

____

__

收录于话题

#BurpSuite 5 个内容

#Web安全 14 个内容

#OAuth2.0认证漏洞 1 个内容

![](https://gitee.com/fuli009/images/raw/master/public/20210917080317.png)

高级漏洞篇-OAuth2.0认证漏洞专题

  
  

###  **什么是OAuth？**

OAuth是一种常用的授权框架，它使网站和应用程序能够请求对另一个应用程序上的用户帐户进行有限访问。值得注意的是，OAuth允许用户授予此访问权限，而无需将其登录凭据暴露给请求的应用程序。这意味着用户可以选择他们想要共享的数据，而不必将其帐户的完全控制权交给第三方。基本的OAuth过程广泛用于集成第三方功能，这些功能需要访问用户帐户中的某些数据。例如，应用程序可能使用OAuth请求访问电子邮件联系人列表，以便它可以建议人们联系。但是，同样的机制也用于提供第三方认证服务，允许用户使用他们在不同网站上的帐户登录。  
尽管OAuth 2.0是当前标准，但一些网站仍然使用旧版1a。OAuth 2.0 是从头开始编写的，而不是直接从OAuth
1.0开发的。所以两者非常不同。后文中所有的OAuth均特指OAuth2.0

###  **OAuth2.0是如何运行的？**

OAuth2.0设计初衷是在应用之间共享特定数据的访问权限，它通过在三个不同方，客户端应用程序、资源所有者、OAuth服务提供者之间定义一系列交互来运作

  * 客户端应用程序(想要访问用户数据的网站或Web应用程序)

  * 资源所有者(客户端应用程序想要访问数据的所有者)

  * OAuth服务提供者(控制用户数据和访问权限的网站或应用程序，它们提供用于与授权服务器和资源服务器进行交互的API来支持OAuth)

可以使用多种不同的方法来实现实际的OAuth流程，这些方法统称为OAuth流或授权类型，burp主要讲解其中的授权码和隐式授权，这两种授权类型包括以下几个阶段

  * 客户端应用程序请求访问用户数据的子集，指定它们要使用的授权类型以及想要哪一种权限

  * 提示用户登录OAuth服务，并询问他们是否同意请求的访问权限

  * 客户端应用程序接收一个唯一的访问令牌(Access Token)，它可以证明它们拥有所请求数据的访问权限

  * 客户端应用程序使用该令牌(Token)调用API，从资源服务器中获取相关数据



OAuth2.0授权类型（grant type）

  
  

##

这是OAuth必须了解的重要概念。

###  **什么是授权类型？**

授权类型确定OAuth流程中涉及的确切步骤顺序，授予类型还会影响客户端应用程序在每个阶段与OAuth服务的通信方式，包括访问令牌的发送方式，所以，授权类型也被称为OAuth流，在客户端应用程序开始流程之前必须配置OAuth服务为特定的授权类型，并将要使用的授权类型包含在初始授权请求中，而且每种授权类型都是处于不同的复杂性和安全性考虑的，burp主要介绍常见的授权码和隐式两个类型

###  **OAuth范围(scope)**

任何OAuth授权类型，客户端应用程序都需要指定要访问的数据和要进行的操作类型，并将这些信息包含在发送给OAuth服务的请求中的参数scope中，经常使用标准化的OpenID
Connect作用域来进行认证

###  **授权码流**

客户端应用程序和OAuth服务首先通过浏览器的重定向来交换一系列HTTP请求，一启动这个授权流程，此时会询问用户是否同意访问，如果同意则向客户端发放授权码，然后客户端与OAuth服务交换这个码来接收访问令牌，它可以用于调用API来获取相关的用户数据，这个交换过程全程是经过安全的，预置的服务器端到服务器端的通道进行的，所以该过程对于最终用户是透明的，当客户端应用程序首次向OAuth服务注册时就会建立这个安全通道，此时还会生成一个client_secret，用于在发送服务端到服务端请求时用client_secret对客户端应用程序进行认证，因为最敏感的数据不经过浏览器发送，所以这种授权类型某种程度上是最安全的，为了更直观地理解这个过程，burp给出一个序列图![](https://gitee.com/fuli009/images/raw/master/public/20210917080321.png)下面我们具体介绍每一步的内容1.授权请求客户端应用程序向OAuth服务的/authorization端点发送请求以获取指定用户数据的访问权限，该请求通常包含如下查询参数

  * client_id(强制参数，包含客户端应用程序的唯一标识符)

  * redirect_uri(将授权码发送到客户端应用程序时浏览器会被重定向到其指定的URI 也被称为回调URI或回调端点，该参数使用频率很高)

  * response_type(指定客户端应用程序期望的响应类型，对于授权码授权类型，其值应该为code)

  * scope(指定客户端应用程序要访问用户数据的哪个子集，其值可以是OAuth提供程序设置的自定义值，也可以是OpenID Connect规范定义的标准化范围)

  * state(与客户端应用程序的当前会话相关的唯一的不可预测的值，该参数还充当客户端应用程序CSRF令牌的形式)

授权请求示例如下

  *   * 

    
    
    GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1Host: oauth-authorization-server.com

2.用户登录并处理请求授权服务器收到初始请求后会将用户重定向到登录页面，该页面会提示他们登录OAuth提供程序的账户，通常为社交账号，然后会向用户提供客户端应用程序要访问的数据列表，用户可以选择是否同意此访问，值得注意的是一旦用户同意了访问的范围，在会话有效期内会自动完成该步骤3.授权码发放如果用户同意请求的访问，浏览器就会被重定向到redirect_uri指定的callback端点，其值有时也可与state参数值相同，像这样

  *   * 

    
    
    GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1Host: client-app.com

4.访问令牌请求客户端应用程序收到授权码以后需要将其兑换为访问令牌，客户端应用程序会向OAuth服务发送POST请求/token端点，全程是在透明通道中进行的，该请求除了client_id和code参数还要有client_secret和grant_type，client_secret由OAuth服务注册时分配，用于对客户端应用程序进行身份验证，grant_type告知端点客户端应用程序使用哪种授权类型，很明显，此时应为authorization_code，像这样

  *   *   *   * 

    
    
    POST /token HTTP/1.1Host: oauth-authorization-server.com…client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8

5.访问令牌发放OAuth服务验证由客户端应用程序发过来的访问令牌请求，验证通过以后会向其发放其请求范围内的访问令牌，像这样

  *   *   *   *   *   *   * 

    
    
    {  "access_token": "z0y9x8w7v6u5",  "token_type": "Bearer",  "expires_in": 3600,  "scope": "openid profile",  …}

6.API调用客户端应用程序接收到访问令牌以后就可以通过API调用OAuth服务的/userinfo端点，调用请求中会包含一个有token_type和access_token的Authorization请求头字段以表明其拥有访问权限，像这样

  *   *   * 

    
    
    GET /userinfo HTTP/1.1Host: oauth-resource-server.comAuthorization: Bearer z0y9x8w7v6u5

7.资源发放OAuth接收到API调用请求时会验证其中的access_token，确认其是否有效并且是否与所请求客户端应用程序匹配，通过验证后返回所请求的资源，像这样

  *   *   *   *   * 

    
    
    {  "username":"carlos",  "email":"carlos@carlos-montoya.net",  …}

###  **隐式流**

隐式就简单多了，客户端应用程序在用户同意后就能接收到由OAuth服务发放的访问令牌，跳过了用授权码兑换访问令牌的过程，所以相对应的，这种授权类型的安全性会大打折扣，而且全程都是经过浏览器的，风险太大了，所以该授权类型更适合单页应用程序和本机桌面应用程序，同样的burp也提供了一个很直观的序列图![](https://gitee.com/fuli009/images/raw/master/public/20210917080322.png)下面我们详细介绍每一步1.授权请求与授权码流不同的是必须要将参数response_type设置为token，像这样

  *   * 

    
    
    GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1Host: oauth-authorization-server.com

2.用户登录并处理请求该阶段与授权码流相同3.访问令牌发放虽然OAuth服务会重定向到redirect_uri指定的授权请求，但是它会将访问令牌和其他相关数据拼接到URL中发回，客户端应用程序需要利用脚本对其进行提取并存储，像这样

  *   * 

    
    
    GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1Host: client-app.com

4.API调用客户端应用程序在提取到token_type和access_token后就使用相同的方式向/userinfo端点发出请求，不过这个过程也是经过浏览器，像这样

  *   *   * 

    
    
    GET /userinfo HTTP/1.1Host: oauth-resource-server.comAuthorization: Bearer z0y9x8w7v6u5

5.资源发放资源服务器验证令牌的有效性以及其是否属于所请求的客户端应用程序，如果通过验证，则返回所请求的数据，像这样

  *   *   *   * 

    
    
    {  "username":"carlos",  "email":"carlos@carlos-montoya.net"}



OAuth认证

  
  

##

现如今OAuth已经逐渐成为身份验证的手段，在OAuth身份验证中OAuth返回的数据会用于基于SAML的单点登录(sso)，OAuth身份验证通常这样实现

  * 用户选择使用其社交媒体账户登录的选项，然后客户端应用程序使用社交媒体网站的OAuth服务来请求访问用于标识用户的数据

  * 客户端应用程序收到访问令牌后会调用/userinfo端点向资源服务器请求此数据

  * 客户端应用程序接收到数据后将使用它代替用户名来登录用户，通常使用访问令牌代替



OAuth身份验证漏洞是怎么产生的？

  
  

##

该漏洞部分是由于OAuth规范在设计上相对模糊且灵活，虽然每种授权类型的基本功能都需要一些强制性组件，但绝大多数实现是完全可选的，这就导致错误的操作会引发不良后果，还有一个原因就是普遍缺乏内置的安全功能，安全性几乎完全依赖于开发人员使用正确的配置选项组合并在顶部实施自己的其他安全性措施，有的授权类型因为全程都经过浏览器，这就导致所有的请求包都可以被拦截到，从而遭到恶意修改

识别OAuth认证

  
  

##

如果登录应用程序会重定向到第三方网站进行登录，大概率是使用了OAuth，进一步探测是通过在burp代理中观察经过浏览器的HTTP请求有哪些，无论哪一种授权类型，整个流程的第一个请求都是向/authorization端点发出的请求，其特点为URL中包含了很多OAuth使用的查询参数，值得注意的查询参数有client_id、redirect_uri、response_type，像这样

  *   * 

    
    
    GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1Host: oauth-authorization-server.com



信息收集

  
  

##

在观察开启OAuth流程的请求包中有时可以看出使用的是哪一款OAuth服务，并且API也会有详细的文档，这些文档可以提供很多信息，例如端点的名称以及配置选项，当知道授权服务器的主机名以后，向其端点发出GET请求可以得到json格式的配置信息，像这样

  *   * 

    
    
    /.well-known/oauth-authorization-server/.well-known/openid-configuration



OAuth认证漏洞的利用

  
  

##

###  **OAuth客户端应用程序的漏洞**

隐式流实现不当在隐式流中，OAuth服务会将访问令牌以URL查询参数的形式通过浏览器返回给客户端应用程序，但是为了维持会话，客户端应用程序会通过浏览器向OAuth服务发出POST请求提交这个访问令牌，但是OAuth并没有任何验证访问令牌与其标识信息是否匹配的能力，这就是隐式信任由客户端应用程序发送来的身份信息，这就使攻击者可以通过篡改标识信息让指定用户也获得所请求的权限配套靶场：OAuth隐式流中的认证绕过先用预留的用户登录，发现OAuth流程发起的请求包![](https://gitee.com/fuli009/images/raw/master/public/20210917080323.png)

  

从上图来看会重定向到/oauth-callback调用API，然后我们再看看发向/oauth-
callback的请求包，我们看看响应中有没有有价值的信息![](https://gitee.com/fuli009/images/raw/master/public/20210917080324.png)从响应来看会请求/me去验证访问权限，所以我们接着跟踪到发向/me的请求包的响应![](https://gitee.com/fuli009/images/raw/master/public/20210917080325.png)我们发现响应中有一个新的字段，从字面意思来看是邮箱验证通过了，所以后续的请求应该都是默认同意的了，所以我们找到POST请求包/authenticate，修改其中的邮箱即可，因为其中的token是通过授权的，重放包以后我们就可以登录目标用户而不需要知道它的密码了![](https://gitee.com/fuli009/images/raw/master/public/20210917080326.png)有缺陷的CSRF防护burp强烈建议非特殊原因不要缺少使用state参数，理想情况下，state参数应包含一个不可预测的值，如开启会话时某些相关信息的hash值，然后这个值以csrf令牌的形式在客户端应用程序和OAuth服务之间来回传递，但是如果整个OAuth流程不包括state参数的话就可以导致类似csrf之类的OAuth的漏洞，例如将用户绑定到属于自己的社交媒体账号上配套靶场：强行链接OAuth配置文件首先我们登录测试用户，然后进入用户中心，连接社交媒体，然后输入测试社交媒体账号，现在HTTP
History中就有了我们所有的观察样本了，我们来剖析一下，我们找到了发起OAuth流程![](https://gitee.com/fuli009/images/raw/master/public/20210917080327.png)我们看到发起OAuth流程的请求中只有访问令牌这一个查询参数，并没有像state一样的防csrf措施，所以我们只要再截获一个有效的含有访问令牌的请求就可以利用csrf将社交媒体账户与受害者账户连接起来，于是我们重新拦截一个这样的请求![](https://gitee.com/fuli009/images/raw/master/public/20210917080328.png)为了保持访问令牌的有效性，我们在拦截到包以后将访问令牌暂存然后丢掉这个包，然后我们在Eploit
Server中构造payload![](https://gitee.com/fuli009/images/raw/master/public/20210917080329.png)然后我们将payload分发给受害者以后，我们重新使用社交媒体账户登录发现登录进来的是管理员账号，我们就可以删除指定用户了![](https://gitee.com/fuli009/images/raw/master/public/20210917080330.png)

###  **缺少授权码和访问令牌**

关于OAuth最著名的漏洞可能是OAuth服务本身的配置使攻击者能窃取授权代码或访问与其他用户账户关联的令牌，窃取到有效的授权码或访问令牌后即可访问受害者的数据，甚至可以以受害者身份登录任何使用相同OAuth服务的任何客户端应用程序，一般开启OAuth流程都会由浏览器将授权码或者访问令牌发送到请求授权的redirect_uri参数指定的/callback端点，但是如果OAuth服务没有正确地验证，则攻击者可以篡改redirect_uri参数指定的/callback端点从而诱使受害者将授权码或者访问令牌发送到攻击者指定的端点，在授权码流中，攻击者甚至可以在发起OAuth流程之前获取到授权码，然后再将其发送到OAuth服务进行验证从而访问受害者账户甚至不需要知道访问令牌，只要OAuth服务与受害者之间存在有效对话，客户端应用程序就会代替攻击者完成访问令牌交换的过程，比较安全的做法就是在交换访问令牌的时候也要在请求中包含redirect_uri参数，服务器检查是否与初始授权请求中收到的匹配，如果不匹配则拒绝交换，因为这个过程对浏览器是透明的，所以攻击者无法控制这个过程

###  **配套靶场：通过redirect_uri劫持OAuth账户**

我们先用测试用户登录，观察HTTP
History发现开启OAuth请求的查询参数中有一个参数redirect_uri，该请求会将授权码等发往这个参数指定的端点，然后我们把这个请求发到repeater中，将这个参数值试着修改成Exploit
Server的域名![](https://gitee.com/fuli009/images/raw/master/public/20210917080331.png)我们在Exploit
Server中接收到了由客户端应用程序发来的授权码，说明我们是可以通过篡改参数redirect_uri值来窃取授权码的，于是我们在Exploit
Server构造payload![]()然后分发给受害者以后我们就能在日志中接收到窃取的授权码，有了授权码以后，将其附在端点/oauth-
callback的code参数中，剩下的流程会由客户端应用程序替我们完成的，将响应发到浏览器中，就成功访问目标用户页面，我们就可以删除指定用户了![](https://gitee.com/fuli009/images/raw/master/public/20210917080332.png)有缺陷的redirect_uri验证有的OAuth会通过设置redirect_uri白名单来缓解攻击者篡改redirect_uri值的攻击，但是可以通过不断测试以观察出绕过方法

  * 有的缓解方法是通过匹配字符串，所以可以通过对字符串进行删减尝试观察

  * 可以通过将其他域以参数的形式附加在redirect_uri后面可能会因为对URI解析的差异绕过白名单限制，例如`https://default-host.com &@foo.evil-user.net#@bar.evil-user.net/`

  * 还可以通过提交重复的redirect_uri参数检测是否存在服务端参数污染漏洞

  * 有时候还可能对本地URI放宽审核，以localhost开头的域名可能会轻易通过白名单检测，例如`https://oauth-authorization-server.com/?client_id=123&redirect_uri=client-app.com/callback&redirect_uri=evil-user.net`

有时候不单单只需要测试redirect_uri参数，可能更改一个参数会影响其他参数的验证通过代理页面窃取授权码和访问令牌首先要观察是否可以将redirect_uri参数更改为指向白名单域中的其他页面，例如通过目录穿越到达其他子域，例如  
`https://client-app.com/oauth/callback/../../example/path`  
对于授权码流需要找到可以访问查询参数的漏洞，而对于隐式流需要提取URL片段，由此引发的漏洞最常见之一为开放重定向，攻击者可以利用它作为代理，将受害者的授权码或访问令牌发送到攻击者指定的可以托管任何恶意脚本的域，对于隐式流，访问令牌的作用并不仅仅用于登录受害者账户，因为全程经过浏览器，所以还可以调用API获取敏感数据，除了开放重定向，还有其他的漏洞也可以将授权码或者访问令牌发送到其他域，例如

  * 处理查询参数和URL片段的危险js

  * XSS漏洞

  * HTML注入漏洞

配套靶场1：通过开放重定向窃取OAuth访问令牌首先我们登录测试用户，但是我们发现用户中心的API Key是隐藏的，于是我们对HTTP
History中相关的包都搜索一下，发现发向/me的GET请求的响应中有API
Key![](https://gitee.com/fuli009/images/raw/master/public/20210917080333.png)我们在窃取到访问令牌以后就可以通过重放这个请求包来获取它的API
Key了，于是我们将这个请求发到Repeater中，下面我们探索一下怎么窃取访问令牌，从burp的资料来看，肯定是要寻找开放重定向漏洞点来转发的，我们利用目录穿越以穿越到子域，通过寻找，我们发现Next
Post存在开放重定向![](https://gitee.com/fuli009/images/raw/master/public/20210917080334.png)我们看到了仅仅通过修改path查询参数的值就能重定向到任意域，于是我们可以把这个构造到发起OAuth流程中，这里要注意一下，我们不能把初次发起OAuth流程的请求包发到Repeater，因为这时候是还没有有效会话的，要先退出再登录进来![](https://gitee.com/fuli009/images/raw/master/public/20210917080336.png)从上图来看拼接到重定向的域名中的符号会被URL编码，所以我们要把这个响应发到浏览器才会正常地跳转，而且我们还发现响应中会包含访问令牌，于是我们在Exploit
Server构造这样的payload![](https://gitee.com/fuli009/images/raw/master/public/20210917080337.png)分发给受害者以后我们就能接收到受害者的访问令牌了，于是我们修改之前/me的请求中的Authorization中的访问令牌，就能获得administrator的API
Key了![](https://gitee.com/fuli009/images/raw/master/public/20210917080339.png)![](https://gitee.com/fuli009/images/raw/master/public/20210917080340.png)配套靶场2：通过代理页面窃取OAuth访问令牌通过测试，发现redirect_uri也存在开放重定向漏洞，然后我们发现评论功能有一个/post/comment/comment-
form，它会用postMessage方法向window.location.href属性值发送到父窗口![](https://gitee.com/fuli009/images/raw/master/public/20210917080341.png)并且允许向任何来源发送，然后我们还发现评论表单会被包含在iframe中![](https://gitee.com/fuli009/images/raw/master/public/20210917080342.png)这样我们就可以将结合开放重定向漏洞把发起OAuth流程的URL想办法拼接到iframe中以将访问令牌发送到指定的域，于是我们在Exploit
Server中构造这样的paylaod![](https://gitee.com/fuli009/images/raw/master/public/20210917080343.png)分发到受害者以后我们就能在log中看到窃取的访问令牌，然后就能通过之前已知的/me路径查看administrator的API
Key![]()![](https://gitee.com/fuli009/images/raw/master/public/20210917080344.png)

###  **有缺陷的范围验证**

对于任何OAuth流，用户都必须定义授权的范围，但是存在某些漏洞导致攻击者可以扩大授权的范围，获得更大权限的访问令牌扩大范围：授权码缺陷在授权码流中，攻击者窃取到授权码，然后攻击者可以在交换授权码或访问令牌的请求中再添加其他的范围值，如果服务器没有对篡改的范围进行验证，则服务器会利用新的范围制作访问令牌并发放给客户端应用程序  
例如，假设攻击者的恶意客户端应用程序最初使用openid电子邮件范围请求访问用户的电子邮件地址。在用户批准此请求后，恶意客户端应用程序会收到一个授权码。当攻击者控制他们的客户端应用程序时，他们可以向包含附加配置文件范围的代码/令牌交换请求添加另一个范围参数，像这样

  *   *   *   * 

    
    
    POST /tokenHost: oauth-authorization-server.com…client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8&scope=openid%20 email%20profile

如果服务器没有根据初始授权请求的范围验证这一点，它有时会使用新范围生成访问令牌并将其发送到攻击者的客户端应用程序，像这样

  *   *   *   *   *   *   * 

    
    
    {  "access_token": "z0y9x8w7v6u5",  "token_type": "Bearer",  "expires_in": 3600,  "scope": "openid email profile",  …}

然后攻击者可以使用他们的应用程序进行API调用来访问用户的个人资料数据。扩大范围：隐式流因为隐式流的所有请求都会经过浏览器，所以攻击者可以更轻而易举地在交换访问令牌的请求中添加新的范围值，只要范围值涉及的权限没有超过之前发放的权限则无需用户再次确认即可获得权限

###  **未验证的用户注册**

当通过OAuth验证用户时，客户端应用程序会隐式信任由OAuth提供的信息是正确的，一些提供OAuth服务的网站允许用户无需验证所有详细信息就能注册账户，包括电子邮件地址，攻击者可以使用与目标用户相同的详细信息向OAuth提供者注册账户，然后客户端应用程序可能会允许攻击者通过该账户以受害者的身份登录

使用OpenID Connect扩展OAuth

  
  
当用于身份验证时，OAuth通常会拓展一个OpenID Connect层，该层提供了一些与标识和身份验证用户相关的其他功能

###  **什么是OpenID Connect？**

OpenID
Connect是在OAuth的基础上扩展的专用身份和认证层，添加了一些可以更好地支持OAuth的认证的简单功能，OAuth最初并不是用来实现认证机制而是授权资源的访问权限的，后来有人错用于认证，在授予访问权限时默认在OAuth一端进行了认证，但是毕竟不是专门为认证设计的，所以设计了OpenID
Connect，它使通过OAuth进行认证的方式更加可靠和统一

###  **OpenID Connect是如何运作的？**

OpenID
Connect是在基础的OAuth之上进行扩展的，从客户端应用程序角度来看，主要区别在于存在一个额外的，标准化的范围集，以及一个额外的响应类型：id_tokenOpenID
Connect角色OpenID Connect角色与标准OAuth的角色区别在于规范使用的术语略有不同

  * 依赖方(正在请求用户身份验证的应用程序，等同于客户端应用程序)

  * 最终用户(正在进行身份验证的用户，等同于OAuth资源所有者)

  * OpenID提供者(配置为支持OpenID Connect的OAuth服务)

OpenID Connect权限声明和范围权限声明指表示资源服务器上用户信息的key:value对，所有OpenID
Connect服务使用相同的作用域集，为了使用OpenID
Connect，客户端应用程序必须在授权请求中指定范围openid，然后可以包括一个或多个其他标准范围，这些范围中每个范围都对应OpenID规范中定义的有关用户声明的子集的读取访问权限

###  **ID令牌(Token)**

OpenID Connect提供的另一个主要附加功能是id_token响应类型，返回的是一个带有json web signature(JWS)的JSON
web
token(JWT)，JWT包含基于最初请求的范围的声明列表、OAuth服务上次对用户进行身份验证的方式和时间，客户端应用程序可以使用它来确定用户时候已被充分认证，使用id_token的主要好处就是减少了客户端应用程序和OAuth服务之间需要发送的请求数量，这可以总体上提供更好的性能，无需获取访问令牌然后分别请求用户数据，包含此数据的ID令牌在用户进行身份验证后立即发送到客户端应用程序，ID令牌中传输的数据的完整性是基于JWT密码签名，虽然这样会防止一些中间人攻击，但是因为用于签名验证的加密密钥是通过同一通道传输的(有时候会暴露在/.well-
known/jwks.json中)，所以仍然可能出现某些攻击，OAuth支持多种响应类型，所以客户端应用程序可以发送具有基本OAuth响应类型和OpenID
Connect的id_token响应类型的授权请求，例如

  *   * 

    
    
    response_type=id_token tokenresponse_type=id_token code

此时ID令牌和授权码或者访问令牌会同时发送到客户端应用程序

###  **识别OpenID Connect**

最简单的办法是查找强制性的openid范围，即使登录过程可能没有使用OpenID
Connect，但是也要检查OAuth服务是否支持它，可以通过添加openid范围或将响应类型更改为id_token并观察是否会触发报错，最好查看文档以了解有用信息，还可以通过从标准端点/.well-
known/openid-configuration访问配置文件

###  **OpenID Connect漏洞**

未受保护的动态客户端注册OpenID规范概述了允许客户端应用程序向OpenID提供程序注册的标准化方法，如果支持动态客户端注册，则客户端应用程序可以向专用/registration端点发送POST请求，通常配置文件和文档中会提供该端点的名称，在请求正文中，客户端应用程序以JSON格式提交有关自身的关键信息，如经常需要包括列入白名单的重定向URI的数组，还可以提交一系列其他信息，如要公开的端点的名称，应用程序的名称等，burp给出了一个示例

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /openid/register HTTP/1.1Content-Type: application/jsonAccept: application/jsonHost: oauth-authorization-server.comAuthorization: Bearer ab12cd34ef56gh89  
    {  "application_type": "web",  "redirect_uris": [    "https://client-app.com/callback",    "https://client-app.com/callback2"    ],  "client_name": "My Application",  "logo_uri": "https://client-app.com/logo.png",  "token_endpoint_auth_method": "client_secret_basic",  "jwks_uri": "https://client-app.com/my_public_keys.jwks",  "userinfo_encrypted_response_alg": "RSA1_5",  "userinfo_encrypted_response_enc": "A128CBC-HS256",  …}

有些OAuth提供程序允许动态客户端注册而无需任何身份验证，攻击者就可以注册自己的恶意客户端应用程序，里面有些属性可以当做URI来控制，可能导致二阶SSRF漏洞配套靶场：通过Open
ID动态客户端注册的SSRF首先进入OpenID的配置文件/.well-known/openid-
configuration![](https://gitee.com/fuli009/images/raw/master/public/20210917080345.png)从配置文件我们可以看到很多端点以及支持的授权类型和可使用的签名算法，然后我们得知注册客户端的端点是/reg，于是我们以这样的示例构造请求包

  *   *   *   *   *   *   *   *   * 

    
    
    POST /reg HTTP/1.1Host: YOUR-LAB-OAUTH-SERVER.web-security-academy.netContent-Type: application/json  
    {  "redirect_uris" : [    "https://example.com"  ]}

发送以后，发现不需要进行验证就注册成功了，并将相关信息反馈在了响应中，包括我们注册成功以后的客户端ID，经过对所有触发的HTTP请求的观察发现，在请求用户是否同意授权的时候会利用img标签加载一个图片资源![](https://gitee.com/fuli009/images/raw/master/public/20210917080347.png)我们在HTTP
History会发现加载图片资源的请求，然后给我们可以通过logo_uri指定其加载图片资源的来源，于是我们对/reg请求包进行修改，示例如下

  *   *   *   *   *   *   *   *   *   * 

    
    
    POST /reg HTTP/1.1Host: YOUR-LAB-OAUTH-SERVER.web-security-academy.netContent-Type: application/json  
    {  "redirect_uris" : [    "https://example.com"  ],  "logo_uri" : "https://YOUR-COLLABORATOR-ID.burpcollaborator.net"}

测试的时候推荐使用burp
collaborator来接收请求，发送请求以后重新注册的客户端的logo地址就会被替换为我们指定的地址，于是我们从响应中复制出新的client_id替换到加载logo的请求URL中![]()我们看到burp
collaborator是可以接收到请求的，说明这个点可以向任意来源发出请求，于是我们将logo_uri替换为题目中的目标URL重新注册客户端，然后再用新的client_id加载logo资源成功获得目标字符串![](https://gitee.com/fuli009/images/raw/master/public/20210917080348.png)允许通过引用授权请求某些服务器可以有效地验证授权请求中的查询字符串，但可能无法将相同的验证充分应用于JWT中的参数，包括redirect_uri，可以在配置文件和文档中查找request_uri_parameter_supported选项以确定是否支持这个参数，也可以添加request_uri参数观察它是否有效，有时候可能未在文档中提及此功能，但是却支持该功能

如何缓解OAuth认证漏洞？

  
  

###  **对于OAuth服务提供者**

要求客户端应用程序注册有效的redirect_uri白名单尽可能使用严格的逐字节比较来验证所有传入请求中的URI，仅允许完全匹配而不是模式匹配，这样可以防止攻击者访问列入白名单的域中的其他页面强制使用参数state还应通过包含一些不可预测的特定于会话的数据(如对会话cookie做取hash值)并将其值绑定到用户的会话，这有助于保护用户免受类似CSRF的攻击，这也使攻击者更难使用任何被盗的授权码确保访问令牌发放给发出请求的同意client_id还应检查请求的范围，以确保它与最初授予令牌的范围匹配

###  **对于OAuth客户端应用程序**

确保完全了解OAuth工作原理的详细信息许多漏洞都是由于对每个阶段确切发生的情况以及如何利用这些潜在原因的简单了解导致的使用state参数将redirect_uri参数同时发送到/authorization端点和/token端点使用PKCE(RFC7638)机制提供额外的保护开发移动或本地桌面OAuth客户端应用程序时，通常无法将client_secret保持私有，PKCE(RFC7638)机制可以防止代码被拦截或泄漏在使用OpenID
Connect id_token时，确保正确验证JSON Web签名(JWS)，JSON
Web加密(JSE)和OpenID规范留意授权码加载外部图像、脚本或CSS内容时，授权码可能通过Referer头泄漏

总结

  
  
以上就是梨子带你刷burpsuite官方网络安全学院靶场(练兵场)系列之高级漏洞篇 –
OAuth2.0认证漏洞专题的全部内容啦，本专题主要讲了OAuth的关键基础原理以及扩展组件、由OAuth机制的缺陷可能产生的漏洞、利用，最后从两个角度介绍了关于这些漏洞的防护建议等，本专题内容较多，请大家耐心阅读并一一动手解题哦，感兴趣的同学可以在评论区进行讨论，嘻嘻嘻。  

系列结语

  
  
至此，梨子带你刷burpsuite官方网络安全学院靶场(练兵场)系列的三个大篇章，共21个专题内容就全部介绍完毕了。这是梨子第一次写文章，而且就完成了整个系列，还是非常有成就感的呢，希望这个系列可以让初学者了解到Web安全的魅力并能够通俗易懂地熟悉常见的Web安全漏洞。梨子建议大家一定要自己注册一个burp账号亲自去做每一道靶场，看每一个知识点，这样对于大家理解是非常有帮助的。21篇文章，内容很多，干货很多，希望大家能够耐心地阅读。梨子非常喜欢交朋友，所以想和梨子交流的可以私信梨子哦。最后，容我在这炫耀一下，本系列共197道靶场，梨子为全球第80个全通的用户，嘻嘻嘻，废话不多说上截图![](https://gitee.com/fuli009/images/raw/master/public/20210917080349.png)非常感谢大家的耐心阅读，你们的支持让梨子觉得在安全圈很开心，感谢有你们。![](https://gitee.com/fuli009/images/raw/master/public/20210917080350.png)

    
    
      
    - 结尾 -  
    精彩推荐[【技术分享】梨子带你刷burpsuite靶场之高级篇 - HTTP请求走私专题](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649751885&idx=1&sn=d6ee55c39a0000a98acd713ac7eb94a3&chksm=88933122bfe4b8345a50b27c4255533549873b34d1138d52b01f16925313ac54405d77ba3533&scene=21#wechat_redirect)  
    [【技术分享】反制爬虫之Burp Suite RCE](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649751822&idx=1&sn=2d36443ca50d76244016ecf4a9451a53&chksm=88933161bfe4b8777ec506375b5c1f3195b2f7a520606455ab35a89499a3530f13cf24646dd5&scene=21#wechat_redirect)  
    [【技术分享】Linux内核中利用msg_msg结构实现任意地址读写](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649751747&idx=1&sn=cbd84a56d46a2a03e4dc619676f2ae19&chksm=889330acbfe4b9ba93ca6536df2031eb246ac68056a895bb999150776861c709765eb72a630c&scene=21#wechat_redirect)  
    [【技术分享】浅谈angr的缓解状态爆炸策略](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649751699&idx=1&sn=c674fac8da0e07f0a3780297a8faf0c4&chksm=889330fcbfe4b9eaf54ff0ee5bd3c04eeb7595b0053ef8ff7abf4582846240737dc49da56c59&scene=21#wechat_redirect)![](https://gitee.com/fuli009/images/raw/master/public/20210917080351.png)
    
    
     **戳“阅读原文”查看更多内容**

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

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

【技术分享】梨子带你刷burpsuite靶场之高级篇 - OAuth2.0认证专题

最多200字，当前共字

__

发送中

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

