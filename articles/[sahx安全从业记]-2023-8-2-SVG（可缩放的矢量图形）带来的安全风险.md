#  SVG（可缩放的矢量图形）带来的安全风险

Sahx  [ sahx安全从业记 ](javascript:void\(0\);)

**sahx安全从业记** ![]()

微信号 gh_6a110ce6ac22

功能介绍 个人学习笔记，平时会写一些漏洞复现、靶场复现、SRC挖洞、工具推荐以及日常吐槽等。

____

___发表于_

收录于合集 #转载文章 13个

**1、概述**  

使网站嵌入可缩放矢量图形(SVG)可能就会存在代码注入的风险。本文探讨了svg的工作原理、它们带来的风险以及如何减轻这些风险。很长一段时间以来，我一直认为可缩放矢量图形(SVG)就是可以缩放的图像，因为图像元素在某种程度上是语义定义的，而不是根据像素定义的。虽然这种直觉没有错，但它也不完整。这些差异确实存在安全隐患，这篇文章的目的就是要强调这一点。作为一篇介绍性的文章，我们将研究一些表面的东西，忽略更高级的攻击，如XML外部实体注入或使用SVG字体的CSS泄露。

 **2、内容**

 **SVG是什么？**  
  
S V G(可放缩的矢量图形)是W3C(World Wide Web
ConSor—tium国际互联网标准组织)在2000年8月制定的一种新的二维矢量图形格式，也是规范中的网络矢量图形标准。W3C是作为一个国际X的工业联盟而创建的，
目的是领导整个互联网协作的发展和创新， 以实现科技的进步和共同发展。由于W3C联盟关于SVG的开发工作组的成员都是一些知名厂商，
如Adobe、苹果、Aut0De sk、Bit Fla
sh、Corel、惠普、IBM、ILOG、INSO、Macromedia、微软、Netscape、OASIS、Open Text、Quark、RAL(C C
LRC)、S un、V i S i
0、施乐等，所以SVG不是一个私有格式，而是一个开放的标准。也就是说，它并不属于任何个体的专利，而是一个通过协作、共同开发的工业标准。正是因为这点，才使得SVG能够得到更迅速的开发和应用。

SVG的特点：  

  *   *   *   * 

    
    
    基于XML采用文本来描述对象具有交互X和动态X完全支持DOM

SVG较G I F、JPEG的优势：  

  *   *   *   *   *   * 

    
    
    任意放缩文本独立较小文件超强显示效果超级颜色控制交互X和智能化

下面显示了一个如何定义SVG的最小示例：

  *   *   * 

    
    
    <svg xmlns="http://www.w3.org/2000/svg">    <circle cx="50" cy="50" r="30" fill="#DA3A00" /></svg>

将此段代码保存为html文件，并使用浏览器打开。  

![]()

需要使用xmlns属性使浏览器将XML
blob识别为要呈现的图像，而不是简单的XML文件。Circle在画布上定义一个坐标为(50,50)的圆，半径为30，填充为漂亮的橙色。

  * 

    
    
    #DA3A00：是一种颜色为亮橙色的颜色格式的hex值

SVG标准定义了许多其他形状和图形旋钮，您可以按照自己的意愿组合和设置它们。SVG还可以包括其他现有文档，如图像和脚本。例如，您可以通过animate标记使圆圈闪烁

  *   *   *   *   *   *   *   *   *   * 

    
    
    <svg xmlns="http://www.w3.org/2000/svg">    <circle cx="50" cy="50" r="30" fill="#DA3A00">    <animate            attributeName="fill"            values="#68768a;#DA3A00"            calcMode="discrete"            dur="0.2s"            repeatCount="indefinite"/>    </circle></svg>

![]()

这是一个gif  

或者通过嵌入SVG本身的JavaScript片段。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <svg width="500" height="300" xmlns="http://www.w3.org/2000/svg">    <circle id="circle" cx="50" cy="50" r="30" fill="#DA3A00" />    ＜script＞        function setColor(color) {            window.document.getElementById("circle").setAttribute("style", `fill: ${color}`)        }  
            function setOrange() {            setColor("#DA3A00")            setTimeout(setGrey, 200)        }  
            function setGrey() {            setColor("#68768a")            setTimeout(setOrange, 200)        }  
            setOrange()    ＜/script＞</svg>

保存后使用浏览器打开。  

![]()

或者我们可以把这个圆和另一个图像结合起来，像这样：

![]()

 **HTML中svg的安全风险**

这是对SVG格式功能的一个非常基本的概述。包含脚本的选项可能已经敲响了代码注入的警钟。本质上，svg可以包含有效的HTML，这打开了xss类型注入攻击的潘多拉盒子。现在，当包含用户提供的svg时，实际的风险是什么?那么，可用的特性(或可能的攻击)完全取决于SVG如何嵌入到HTML文档中。

为了便于说明，让我们假设我们创建了一个基于协作用户输入的显示思维导图的网站。每个条目都是一个SVG，发送到服务器，然后服务器向所有用户输出一个HTML文档，将用户SVG合并到各自的画布位置。

有几种方法可以嵌入用户提供的svg：

  *   *   * 

    
    
    img or image 标签iframe, embed, or object 标签内嵌

除了MDN web文档中列出的不同实际优点和缺点外，每种方法都具有不同的安全属性。让我们看一看

 **内嵌**  

将svg添加到HTML页面的最简单的方法是将它们内嵌，如下所示：

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <!DOCTYPE html><html>    <body>        <div id="outside">I'm a div inside the HTML</div>        <svg xmlns="http://www.w3.org/2000/svg" width="300" height="300">            <circle cx="150" cy="147.5" r="50" fill="#DA3A00" />            <image href="S.svg" />            ＜script＞alert("Hello from an SVG")＜/script＞            ＜script＞document.getElementById("outside").innerHTML = "Got hacked"＜/script＞        </svg>    </body></html>

这样，SVG就完全不受限制，几乎可以做任何事情。这绝对是危险的。在这种情况下，从SVG内部加载的图像会正确显示，因此从功能角度来看，思维导图会变得更加强大。但是这时会弹出一个警告。你的div被破坏了。攻击者可以冒充任何打开带有此SVG的思维导图的用户。

事实上，当在您的网页上下文中执行脚本时，攻击者可以做很多事情。这包括以下攻击：

  *   *   *   *   *   * 

    
    
    1、提取机密信息：①阅读页面上的所有内容(例如，在思维导图的标题行加载的个人信息)②读取本地存储③访问凭证，比如缺少HttpOnly标志的cookie2、使用起源的现有身份验证(如身份验证cookie)冒充用户查询API(例如更改受害者的帐户详细信息或冒充受害者并添加或编辑post-its作为他们)3、通过窗口更改SVG的父HTML文档。父句柄在同源嵌入期间可访问(例如隐藏所有便利贴，要求付款，呈现恶意页面，或重定向到一个)。

此外，内联嵌入用户生成的svg需要服务器在一定程度上处理它们。虽然不在本文的讨论范围之内，但请注意，这会增加XXE和SSRF攻击的可能性，这需要对XML结构进行服务器端处理。因此，虽然内联svg肯定会给用户带来风险，但它们也可能给作为服务器操作员的您带来风险。

 **  嵌入上下文**

svg也可以通过iframe、embed或object标签来包含，每个标签都会创建另一个浏览上下文。

  *   *   *   *   *   *   *   * 

    
    
    <!DOCTYPE html><html>    <body>        <iframe src="circleWithS.svg" width="300" height="250"></iframe>        <embed src="circleWithS.svg" width="300" height="250" />        <object data="circleWithS.svg" width="300" height="250"></object>    </body></html>

这将在更受限制的上下文中加载SVG，这仍然允许SVG获取外部资源或执行脚本。

如上所示，脚本执行并不局限于良性用途，比如让元素闪烁。幸运的是，它仅限于SVG的上下文，即加载SVG的源。也就是说，由于浏览器的同源策略安全特性，如果一个HTML文档的来源(scheme、主机和端口的三元组)不同，则不允许从一个HTML文档访问另一个HTML文档。

但是，如果SVG是从与父HTML文档(我们的思维导图)相同的来源加载的，那么攻击者就可以在页面上下文中做坏事，就好像SVG是内联包含的一样。

例如，如果我们有一个HTML文档：

  *   *   *   *   *   *   * 

    
    
    <!DOCTYPE html><html>    <body>        <div id="outside">I'm a div inside the HTML</div>        <iframe src="attacker-controlled.svg"></iframe>    </body></html>

SVG包含attacker-controlled.svg

  *   *   *   *   * 

    
    
    <svg xmlns="http://www.w3.org/2000/svg">    <foreignObject>        ＜script＞window.parent.document.getElementById("outside").innerHTML = "Got hacked"＜/script＞    </foreignObject></svg>

那么外部带有id的div将被脚本更改，就像上面的内联SVG一样。你可以在这里看到这个例子。

相反，如果图像是从不同的来源加载的，我们可以在不影响主站的情况下获得脚本功能。我们可以通过JavaScript工作，或者我们可以从SVG中加载其他图像，这在最后一种集成方法中是不可能的。这甚至可以传递地工作，因为svg不需要在image标记中加载其他图像，但可以使用foreignObject标记并在其中包含iframe、embed或对象以继续这个链

  *   *   *   *   * 

    
    
    <svg  version="1.1" xmlns="http://www.w3.org/2000/svg">    <foreignObject>        <iframe src="someOther.svg"></iframe>    </foreignObject></svg>

但是请注意，有些脚本仍然会对用户产生负面影响。想象一个嵌入的SVG，它每隔几秒钟发出警报。虽然从技术上讲，弹出式并不是源自你嵌入的思维导图，但你的用户肯定不会喜欢不断被打扰。因此，即使嵌入式svg不能直接影响站点，仍然需要小心。

 **Image标签**

最后，img和image标签受到浏览器安全控制的严格限制。它们不加载SVG文件中定义的外部内容，也不执行脚本。因此，使用它们通常是安全的，但如果需要其中一个特性，则可能不够。例如，通过使用JavaScript制作的圆圈闪烁不起作用，带S的圆圈也不起作用(即使S托管在同一个原点上)。因此，下面的HTML片段将显示基本圆圈两次(SVG将尽可能多地呈现)，但既不闪烁，也不与外部圆圈结合

  *   *   *   *   *   *   * 

    
    
    <!DOCTYPE html><html>    <body>        <img src="circleBlinkJS.svg" />        <img src="circleWithS.svg" />    </body></html>

如果我们的思维导图应用程序应该支持这些特性，或者思维导图实现是由较小的SVG构建而成的大SVG，那么这是行不通的。

  *   *   *   *   * 

    
    
    <svg xmlns="http://www.w3.org/2000/svg">    <image href="userSvg1.svg" />    <image href="userSvg2.svg" />    ...</svg>

因此，通常需要在安全性和可用性之间取得平衡。

 **减少SVG风险**

基本上，存在4种缓解措施来防御上述攻击：

  *   *   *   * 

    
    
    可以使用img或image标签。可以自定义定制内容安全策略（CSP）。可以在单独的(子)域中托管svg，并从那里嵌入它们。在接受或显示svg时，可以尝试输入验证、清理或输出编码。

如果您打算像加载普通图像一样加载SVG，而没有特定于SVG的特性，则通常希望将其加载到img或image标记中。如果这是不可能的，例如，因为您需要SVG加载的外部元素，或者您需要SVG提供的脚本功能，那么您应该更加小心。svg为设计人员和攻击者提供了一个强大的工具箱。如果需要在您的站点上显示用户控制或用户影响的svg，一个强有力的防御措施是定义一个限制svg脚本执行的CSP，例如通过script-
src 'none'或散列。

另一种缓解方法是将svg托管在单独的(子)域上，例如resources.your-
domain.org，仅用于静态内容，因为这样用户浏览器的同源策略将阻止在主站点上下文中执行任何脚本。在这种情况下，即使是script-src
'self'的CSP也会保护你(与你在同一来源上托管网站和SVG的情况相反)。这种强大而简单的深度防御措施只有在使用iframe之类的嵌入上下文时才有效，因为内联显然会使资源从主站点的源加载。

另一种选择是尝试在服务器端对SVG进行输入验证、清理或输出编码，而不包括脚本。然而，这暴露了解析XML结构没有失败的风险，也没有遇到诸如XXE攻击之类的问题。它只是更容易出错，因此应该避免。

因此，防止内联svg的唯一合理解决方案是使用定制的CSP，这取决于您的站点设置，可能很难设计。更实际和简单的方法是采用双管齐下的方法，即为静态资产使用单独的(子)域，同时使用不包括在其script-
src指令中的CSP。

 **3、结论**  

嵌入用户提供的svg会使您的网站暴露于注入攻击。这似乎是显而易见的，但回到我最初的陈述，最重要的是要记住SVG不仅仅是一个可伸缩的图像，而是一个脚本化的对象，攻击者会认为它是注入攻击的目标。如果可以的话，避免使用用户提供的svg。如果没有，则使用强大的缓解措施(如img标记、内容安全策略或跨源分离)来实现它们。(此为原文翻译而来，如有语句不通自行理解)

  * 

    
    
    原文链接：https://www.securesystems.de/blog/svg-security-risks-not-just-a-scalable-graphic/

![]()

 **免责声明：  
**

**本公众号漏洞复现文章，SRC、渗透测试等文章，仅供学习参考，请勿用于实战！！有授权情况下除外！！由于传播、利用本公众号文章所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责**

![]()

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

