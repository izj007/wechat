#  渗透测试岗位面试题(重点：渗透思路)

[ LemonSec ](javascript:void\(0\);)

**LemonSec** ![]()

微信号 lemon-sec

功能介绍 每日发布安全资讯～

____

___发表于_

收录于合集

本文主要是讲解遇到问题的各思路解决方法，不仅可做为面试题查看，在实操中收到思绪的阻碍也可作为参考.  
 **1、拿到一个待检测的站，你觉得应该先做什么？**  
1)信息收集

  *   *   *   *   *   * 

    
    
    1，获取域名的whois信息,获取注册者邮箱姓名电话等。2，查询服务器旁站以及子域名站点，因为主站一般比较难，所以先看看旁站有没有通用性的cms或者其他漏洞。3，查看服务器操作系统版本，web中间件，看看是否存在已知的漏洞，比如IIS，APACHE,NGINX的解析漏洞4，查看IP，进行IP地址端口扫描，对响应的端口进行漏洞探测，比如 rsync,心脏出血，mysql,ftp,ssh弱口令等。5，扫描网站目录结构，看看是否可以遍历目录，或者敏感文件泄漏，比如php探针6，google hack 进一步探测网站的信息，后台，敏感文件
    
    
    2）漏洞扫描    开始检测漏洞，如XSS,XSRF,sql注入，代码执行，命令执行，越权访问，目录读取，任意文件读取，下载，文件包含，    远程命令执行，弱口令，上传，编辑器漏洞，暴力破解等3）漏洞利用    利用以上的方式拿到webshell，或者其他权限4）权限提升    提权服务器，比如windows下mysql的udf提权，serv-u提权，windows低版本的漏洞，如iis6,pr,巴西烤肉，linux脏牛漏洞，linux内核版本漏洞提权，linux下的mysql system提权以及oracle低权限提权5) 日志清理6）总结报告及修复方案

 **2.判断出网站的CMS对渗透有什么意义？**  

  * 

    
    
     查找网上已曝光的程序漏洞。如果开源，还能下载相对应的源码进行代码审计。

 **3.一个成熟并且相对安全的CMS，渗透时扫目录的意义？**

  * 

    
    
     敏感文件、二级目录扫描
    
    
    站长的误操作比如：网站备份的压缩文件、说明.txt、二级目录可能存放着其他站点

 **4.常见的网站服务器容器。**

  * 

    
    
     IIS、Apache、nginx、Lighttpd、Tomcat

 **5.mysql注入点，用工具对目标站直接写入一句话，需要哪些条件？**  

  * 

    
    
     root权限以及网站的绝对路径。

 **6.目前已知哪些版本的容器有解析漏洞，具体举例。**  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     IIS 6.0/xx.asp/xx.jpg "xx.asp"是文件夹名  
    IIS 7.0/7.5默认Fast-CGI开启，直接在url中图片地址后面输入/1.php，会把正常图片当成php解析  
    Nginx版本小于等于0.8.37，利用方法和IIS 7.0/7.5一样，Fast-CGI关闭情况下也可利用。空字节代码 xxx.jpg.php  
    Apache上传的文件命名为：test.php.x1.x2.x3，Apache是从右往左判断后缀  
    lighttpdxx.jpg/xx.php，不全

  
 **7.如何手工快速判断目标站是windows还是linux服务器？**  

  * 

    
    
     linux大小写敏感,windows大小写不敏感。

 **8.为何一个mysql数据库的站，只有一个80端口开放？**  

  *   *   * 

    
    
     更改了端口，没有扫描出来。站库分离。3306端口不对外开放

 **9、3389无法连接的几种情况**

  *   *   *   *   *   *   * 

    
    
     没开放3389 端口  
    端口被修改  
    防护拦截  
    处于内网(需进行端口转发)

  
 **10.如何突破注入时字符被转义？**

  *   *   * 

    
    
     宽字符注入  
    hex编码绕过

  
 **11.在某后台新闻编辑界面看到编辑器，应该先做什么？**  

  * 

    
    
     查看编辑器的名称版本,然后搜索公开的漏洞。

 **12.拿到一个webshell发现网站根目录下有.htaccess文件，我们能做什么？**  

  *   *   *   *   *   * 

    
    
     能做的事情很多，用隐藏网马来举例子：插入<FilesMatch "xxx.jpg"> SetHandler application/x-httpd-php </FilesMatch>.jpg文件会被解析成.php文件。  
    具体其他的事情，不好详说，建议大家自己去搜索语句来玩玩。

  
 **13.注入漏洞只能查账号密码？**

  * 

    
    
     只要权限广，拖库脱到老。

 **14.安全狗会追踪变量，从而发现出是一句话木马吗？**  

  * 

    
    
     是根据特征码，所以很好绕过了，只要思路宽，绕狗绕到欢，但这应该不会是一成不变的。

 **15.access 扫出后缀为asp的数据库文件，访问乱码，如何实现到本地利用？**

  * 

    
    
     迅雷下载，直接改后缀为.mdb。

 **16.提权时选择可读写目录，为何尽量不用带空格的目录？**  

  * 

    
    
     因为exp执行多半需要空格界定参数

 **17.某服务器有站点A,B 为何在A的后台添加test用户，访问B的后台。发现也添加上了test用户？**

  * 

    
    
     同数据库。

 **18.注入时可以不使用and 或or 或xor，直接order by 开始注入吗？**  

  * 

    
    
     and/or/xor，前面的1=1、1=2步骤只是为了判断是否为注入点，如果已经确定是注入点那就可以省那步骤去。

 **19:某个防注入系统，在注入时会提示：**

  *   *   *   *   * 

    
    
     系统检测到你有非法注入的行为。已记录您的ip xx.xx.xx.xx时间:2016:01-23提交页面:test.asp?id=15提交内容:and 1=1

 **20、如何利用这个防注入系统拿shell？**

  * 

    
    
     在URL里面直接提交一句话，这样网站就把你的一句话也记录进数据库文件了 这个时候可以尝试寻找网站的配置文件 直接上菜刀链接。具体文章参见：http://ytxiao.lofter.com/post/40583a_ab36540。

 **21.上传大马后访问乱码时，有哪些解决办法？**  

  * 

    
    
     浏览器中改编码。

 **22.审查上传点的元素有什么意义？**  

  * 

    
    
     有些站点的上传文件类型的限制是在前端实现的，这时只要增加上传类型就能突破限制了。

 **23.目标站禁止注册用户，找回密码处随便输入用户名提示：“此用户不存在”，你觉得这里怎样利用？**  

  *   *   *   *   * 

    
    
     先爆破用户名，再利用被爆破出来的用户名爆破密码。  
    其实有些站点，在登陆处也会这样提示  
    所有和数据库有交互的地方都有可能有注入。

  
 **24.目标站发现某txt的下载地址** 为http://www.test.com/down/down.php?file=/upwdown/1.txt，
**你有什么思路？**  

  * 

    
    
     这就是传说中的下载漏洞！在file=后面尝试输入index.php下载他的首页文件，然后在首页文件里继续查找其他网站的配置文件，可以找出网站的数据库密码和数据库的地址。

 **25.甲给你一个目标站，并且告诉你根目录下存在/abc/目录，并且此目录下存在编辑器和admin目录。请问你的想法是？**  

  * 

    
    
     直接在网站二级目录/abc/下扫描敏感文件及目录。

 **26.在有shell的情况下，如何使用xss实现对目标站的长久控制？**  

  *   *   * 

    
    
     后台登录处加一段记录登录账号密码的js，并且判断是否登录成功，如果登录成功，就把账号密码记录到一个生僻的路径的文件中或者直接发到自己的网站文件中。(此方法适合有价值并且需要深入控制权限的网络)。  
    在登录后才可以访问的文件中插入XSS脚本。

  
 **27.后台修改管理员密码处，原密码显示为*。你觉得该怎样实现读出这个用户的密码？**  

  * 

    
    
     审查元素 把密码处的password属性改成text就明文显示了

 **28.目标站无防护，上传图片可以正常访问，上传脚本格式访问则403.什么原因？**  

  * 

    
    
     原因很多，有可能web服务器配置把上传目录写死了不执行相应脚本，尝试改后缀名绕过

 **29.审查元素得知网站所使用的防护软件，你觉得怎样做到的？**  

  * 

    
    
     在敏感操作被拦截，通过界面信息无法具体判断是什么防护的时候，F12看HTML体部 比如护卫神就可以在名称那看到<hws>内容<hws>。

 **30.在win2003服务器中建立一个 .zhongzi文件夹用意何为？**  

  * 

    
    
     隐藏文件夹，为了不让管理员发现你传上去的工具。

 **31、sql注入有以下两个测试选项，选一个并且阐述不选另一个的理由：**

  * 

    
    
     A. demo.jsp?id=2+1 B. demo.jsp?id=2-1选B，在 URL 编码中 + 代表空格，可能会造成混淆

  
 **32、以下链接存在 sql 注入漏洞，对于这个变形注入，你有什么思路？**

  *   * 

    
    
     demo.do?DATA=AjAxNg==DATA有可能经过了 base64 编码再传入服务器，所以我们也要对参数进行 base64 编码才能正确完成测试

  
 **33、发现 demo.jsp?uid=110 注入点，你有哪几种思路获取 webshell，哪种是优选？**

  *   *   * 

    
    
     有写入权限的，构造联合查询语句使用using INTO OUTFILE，可以将查询的输出重定向到系统的文件中，这样去写入 WebShell使用 sqlmap –os-shell 原理和上面一种相同，来直接获得一个 Shell，这样效率更高通过构造联合查询语句得到网站管理员的账户和密码，然后扫后台登录后台，再在后台通过改包上传等方法上传 Shell

  
 **34、CSRF 和 XSS 和 XXE 有什么区别，以及修复方式？**

  *   *   *   *   *   *   *   * 

    
    
     XSS是跨站脚本攻击，用户提交的数据中可以构造代码来执行，从而实现窃取用户信息等攻击。修复方式：对字符实体进行转义、使用HTTP Only来禁止JavaScript读取Cookie值、输入时校验、浏览器与Web应用端采用相同的字符编码。  
    CSRF是跨站请求伪造攻击，XSS是实现CSRF的诸多手段中的一种，是由于没有在关键操作执行时进行是否由用户自愿发起的确认。修复方式：筛选出需要防范CSRF的页面然后嵌入Token、再次输入密码、检验Referer  
    XXE是XML外部实体注入攻击，XML中可以通过调用实体来请求本地或者远程内容，和远程文件保护类似，会引发相关安全问题，例如敏感文件读取。修复方式：XML解析库在调用时严格禁止对外部实体的解析。

  
 **35、CSRF、SSRF和重放攻击有什么区别** **？**

  *   *   * 

    
    
     CSRF是跨站请求伪造攻击，由客户端发起SSRF是服务器端请求伪造，由服务器发起重放攻击是将截获的数据包进行重放，达到身份认证等目的

  
 **36、说出至少三种业务逻辑漏洞，以及修复方式？**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     密码找回漏洞中存在1）密码允许暴力破解、2）存在通用型找回凭证、3）可以跳过验证步骤、4）找回凭证可以拦包获取等方式来通过厂商提供的密码找回功能来得到密码。  
    身份认证漏洞中最常见的是1）会话固定攻击2） Cookie 仿冒只要得到 Session 或 Cookie 即可伪造用户身份。  
    验证码漏洞中存在1）验证码允许暴力破解2）验证码可以通过 Javascript 或者改包的方法来进行绕过

  
 **37、圈出下面会话中可能存在问题的项，并标注可能会存在的问题？**  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
     get /ecskins/demo.jsp?uid=2016031900&keyword=”hello world”HTTP/1.1Host:***.com:82User-Agent:Mozilla/5.0 Firefox/40Accept:text/css,/;q=0.1Accept-Language:zh-CN;zh;q=0.8;en-US;q=0.5,en;q=0.3Referer:http://*******.com/eciop/orderForCC/cgtListForCC.htm?zone=11370601&v=145902Cookie:myguid1234567890=1349db5fe50c372c3d995709f54c273d;uniqueserid=session_OGRMIFIYJHAH5_HZRQOZAMHJ;st_uid=N90PLYHLZGJXI-NX01VPUF46W;status=TrueConnection:keep-alive

  
 **38、给你一个网站你是如何来渗透测试的?**

  * 

    
    
     在获取书面授权的前提下。

  
 **39、sqlmap，怎么对一个注入点注入？**

  *   * 

    
    
     1）如果是get型号，直接，sqlmap -u "诸如点网址".2) 如果是post型诸如点，可以sqlmap -u "注入点网址” --data="post的参数"3）如果是cookie，X-Forwarded-For等，可以访问的时候，用burpsuite抓包，注入处用号替换，放到文件里，然后sqlmap -r "文件地址"

  
 **40、nmap，扫描的几种方式**  
 **41、sql注入的几种类型？**

  *   *   *   * 

    
    
     1）报错注入2）bool型注入3）延时注入4）宽字节注入

 **42、报错注入的函数有哪些？**  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
     10个1）and extractvalue(1, concat(0x7e,(select @@version),0x7e))】】】----------------2）通过floor报错 向下取整3）+and updatexml(1, concat(0x7e,(secect @@version),0x7e),1)4）.geometrycollection()select from test where id=1 and geometrycollection((select from(selectfrom(select user())a)b));5）.multipoint()select from test where id=1 and multipoint((select from(select from(select user())a)b));6）.polygon()select from test where id=1 and polygon((select from(select from(select user())a)b));7）.multipolygon()select from test where id=1 and multipolygon((select from(select from(select user())a)b));8）.linestring()select from test where id=1 and linestring((select from(select from(select user())a)b));9）.multilinestring()select from test where id=1 and multilinestring((select from(select from(select user())a)b));10）.exp()select from test where id=1 and exp(~(select * from(select user())a));

  
 **43、延时注入如何来判断？**

  * 

    
    
     if(ascii(substr(“hello”, 1, 1))=104, sleep(5), 1)

  
 **44、盲注和延时注入的共同点？**

  * 

    
    
     都是一个字符一个字符的判断

  
 **45、如何拿一个网站的webshell？**

  *   * 

    
    
     上传，后台编辑模板，sql注入写文件，命令执行，代码执行，一些已经爆出的cms漏洞，比如dedecms后台可以直接建立脚本文件，wordpress上传插件包含脚本文件zip压缩包等

  
 **46、sql注入写文件都有哪些函数？**

  *   *   * 

    
    
     select '一句话' into outfile '路径'select '一句话' into dumpfile '路径'select '<?php eval($_POST[1]) ?>' into dumpfile  'd:\wwwroot\baidu.com\nvhack.php';

  
 **47、如何防止CSRF?**

  *   *   * 

    
    
     1,验证referer2，验证token详细：http://cnodejs.org/topic/5533dd6e9138f09b629674fd

  
 **48、owasp 漏洞都有哪些？**

  *   *   *   *   *   *   *   *   *   * 

    
    
     1、SQL注入防护方法：2、失效的身份认证和会话管理3、跨站脚本攻击XSS4、直接引用不安全的对象5、安全配置错误6、敏感信息泄露7、缺少功能级的访问控制8、跨站请求伪造CSRF9、使用含有已知漏洞的组件10、未验证的重定向和转发

  
 **49、SQL注入防护方法？**

  *   *   *   *   * 

    
    
     1、使用安全的API2、对输入的特殊字符进行Escape转义处理3、使用白名单来规范化输入验证方法4、对客户端输入进行控制，不允许输入SQL注入相关的特殊字符5、服务器端在提交数据库进行SQL查询之前，对特殊字符进行过滤、转义、替换、删除。

  
 **50、代码执行，文件读取，命令执行的函数都有哪些？**

  *   *   *   *   *   *   * 

    
    
     1）代码执行：eval,preg_replace+/e,assert,call_user_func,call_user_func_array,create_function2）文件读取：file_get_contents(),highlight_file(),fopen(),readfile(),fread(),fgetss(), fgets(),parse_ini_file(),show_source(),file()等3)命令执行：system(), exec(), shell_exec(), passthru() ,pcntl_exec(), popen(),proc_open()

  
 **51、img标签除了onerror属性外，还有其他获取管理员路径的办法吗？**

  * 

    
    
     src指定一个远程的脚本文件，获取referer

  
 **52、img标签除了onerror属性外，并且src属性的后缀名，必须以.jpg结尾，怎么获取管理员路径。** **  
** **侵权请私聊公众号删文**  

  **热文推荐  ** ** **

  * [蓝队应急响应姿势之Linux](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523380&idx=1&sn=27acf248b4bbce96e2e40e193b32f0c9&chksm=f9e3f36fce947a79b416e30442009c3de226d98422bd0fb8cbcc54a66c303ab99b4d3f9bbb05&scene=21#wechat_redirect)  

  * [通过DNSLOG回显验证漏洞](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523485&idx=1&sn=2825827e55c1c9264041744a00688caf&chksm=f9e3f3c6ce947ad0c129566e5952ac23c990cf0428704df1a51526d8db6adbc47f998ee96eb4&scene=21#wechat_redirect)  

  * [记一次服务器被种挖矿溯源](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523441&idx=2&sn=94c6fae1f131c991d82263cb6a8c820b&chksm=f9e3f32ace947a3cdae52cf4cdfc9169ecf2b801f6b0fc2312801d73846d28b36d4ba47cb671&scene=21#wechat_redirect)  

  * [内网渗透初探 | 小白简单学习内网渗透](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523346&idx=1&sn=4bf01626aa7457c9f9255dc088a738b4&chksm=f9e3f349ce947a5f934329a78177b9ce85e625a36039008eead2fe35cbad5e96a991569d0b80&scene=21#wechat_redirect)  

  * [实战|通过恶意 pdf 执行 xss 漏洞](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523274&idx=1&sn=89290e2b7a8e408ff62a657ef71c8594&chksm=f9e3f491ce947d8702eda190e8d4f7ea2e3721549c27a2f768c3256de170f1fd0c99e817e0fb&scene=21#wechat_redirect)  

  * [免杀技术有一套（免杀方法大集结）(Anti-AntiVirus)](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523189&idx=1&sn=44ea2c9a59a07847e1efb1da01583883&chksm=f9e3f42ece947d3890eb74e4d5fc60364710b83bd4669344a74c630ac78f689b1248a2208082&scene=21#wechat_redirect)  

  * [内网渗透之内网信息查看常用命令](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522979&idx=1&sn=894ac98a85ae7e23312b0188b8784278&chksm=f9e3f5f8ce947cee823a62ae4db34270510cc64772ed8314febf177a7660de08c36bedab6267&scene=21#wechat_redirect)  

  * [关于漏洞的基础知识](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523083&idx=2&sn=0b162aba30063a4073bad24269a8dc0e&chksm=f9e3f450ce947d4699dfebf0a60a2dade481d8baf5f782350c2125ad6a320f91a2854d027e85&scene=21#wechat_redirect)  

  * [任意账号密码重置的6种方法](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522927&idx=1&sn=075ccdb91ae67b7ad2a771aa1d6b43f3&chksm=f9e3f534ce947c220664a938bc42926bee3ca8d07c6e3129795d7c8977948f060b08c0f89739&scene=21#wechat_redirect)  

  * [干货 | 横向移动与域控权限维持方法总汇](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522810&idx=2&sn=ed65a8c60c45f9af598178ed20c89896&chksm=f9e3f6a1ce947fb710ff77d8fbd721220b16673953b30eba6b10ad6e86924f6b4b9b2a983e74&scene=21#wechat_redirect)  

  * [手把手教你Linux提权](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522500&idx=2&sn=ec74a21ef0a872f7486ccac6772e0b9a&chksm=f9e3f79fce947e89eac9d9077eee8ce74f3ab35a345b1c2194d11b77d5b522be3b269b326ebf&scene=21#wechat_redirect)

  
 ** **欢迎关注LemonSec**** **觉得不错点个 **“赞”** 、“在看“**

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

