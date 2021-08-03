##  为 CobaltStrike TeamServer 加上谷歌二次验证

原创 Medicean  [ 雷神众测 ](javascript:void\(0\);)

**雷神众测** ![]()

微信号 bounty_team

功能介绍 雷神众测，专注于渗透测试技术及全球最新网络攻击技术的分析。

____

__

收录于话题

  

**STATEMENT**

 **声明**

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  

  

西安安全运营中心

 **NO.1 前言**

CobaltStrike TeamServer 在认证的时候，是没有类似验证码的功能的，这就为蓝队爆破 TeamServer 密码提供了可能。

  

再加上攻防演练当中出现的 **「红队钓鱼事件」** ，团队中不乏有新鲜血液，稍有不慎就会让用户家目录下的 .aggressor.prop
文件被他人窃取，令团队成果付之东流。

  

那么如何规避这种情况呢？我们有如下几个方案考虑：

 **方案一:   直接反编译 CobaltStrike JAR 包，修改逻辑之后重打包。**

该方案的好处是可控性强，可以改的面目全非，甚至可以把配置文件中的密码保存改成加密的形式。

 **缺点也显而易见：**

· 首先CS反编译之后重打包挺让人抓狂的，不好弄。

· 其次必须要用专有的 cobaltstrike 去连接，后续如果拿到新的版本，工作量是比较大的

· 团队成员机器如果被控，攻击者完全可以偷走专有的工具和配置文件进行连接

 **所以该方案 Pass**

 **方案二：网络层动手脚**

TeamServer 监听在 127.0.0.1 接口上，通过 SSH 等其它方式建立 Socks5隧道或者 VPN，团队成员在接入 TeamServer
时，先进入到专网当中，再连接。

好处是成本低，无任何额外工作量，TeamServer 监听的端口也不会被 fofa zoomeye 这些网络空间搜索引擎扫描到。

 **缺点是** 团队成员机器被控后，攻击者完全可以该成员机器作为跳板，通过专网连接。

 **所以该方案也 Pass**

 **  
**

 **进一步分析**

将认证的过程分离开来，一部分保存在团队成员电脑上，另一部分放在团队成员的个人手机上，这就是本文要推荐的方案了，也是两步验证的核心思路了。

  

我们先来看一下 CS TeamServer 中关于认证过程的逻辑:

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package server;public class ManageUser implements Runnable {   protected TeamSocket client;   protected boolean authenticated = false;   protected String nickname = "";   protected Resources resources;   protected ManageUser.BroadcastWriter writer = null;   protected Map calls = null;   protected Thread mine = null;   public void process(Request request) throws Exception {      String reqArg0;      if (!this.authenticated && "aggressor.authenticate".equals(request.getCall()) && request.size() == 3) {         reqArg0 = request.arg(0) + ""; // 用户昵称         String password = request.arg(1) + ""; // 密码         String clientver = request.arg(2) + ""; // 客户端版本         if (!Aggressor.VERSION.equals(clientver)) { // 版本比对            this.client.writeObject(request.reply("Your client software does not match this server\nClient: " + clientver + "\nServer: " + Aggressor.VERSION));         } else if (ServerUtils.getServerPassword(this.resources, reqArg0).equals(password)) { // 密码比对            if (this.resources.isRegistered(reqArg0)) { // 用户是否已经登录了               this.client.writeObject(request.reply("User is already connected."));            } else { // 登录成功               this.client.writeObject(request.reply("SUCCESS"));               this.authenticated = true;               this.nickname = reqArg0;               Thread.currentThread().setName("Manage: " + this.nickname);               this.writer = new ManageUser.BroadcastWriter();               (new Thread(this.writer, "Writer for: " + this.nickname)).start();            }         } else {            this.client.writeObject(request.reply("Logon failure"));         }      } else if (!this.authenticated) {         this.client.close();      }    // ... 省略其它代码   }}  
    ServerUtils.getServerPassword 的内容也很简单，就是直接获取密码而已  
    package server;public class ServerUtils {   public static String getServerPassword(Resources resource, String key) {      return (String)resource.get("password");   }}

可以看到，整个流程当中，密码比对了一次，那是不是意味着我们修改了 getServerPassword 这块就完事了？

刚开始我也是这么想的，后来发现事情并没有这么简单，TeamServer 的 password 不止一处使用到了

我们先来看一下 TeamServer 初始化的代码，先从 main 函数看，main 函数从命令行中接到了参数之后，初始化 teamserver 然后调用了
teamserver 的 go 方法

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package server;public class TeamServer {   public void go() {      try {         new ProfileEdits(this.c2profile);         this.c2profile.addParameter(".watermark", this.auth.getWatermark());         this.c2profile.addParameter(".self", CommonUtils.readAndSumFi1e(TeamServer.class.getProtectionDomain().getCodeSource().getLocation().getPath()));         this.resources = new Resources(this.calls);         this.resources.put("c2profile", this.c2profile);         this.resources.put("localip", this.host);         this.resources.put("password", this.pass); // 向 resources 这个 Map 中设置 password         (new TestCall()).register(this.calls);         WebCalls webcalls = new WebCalls(this.resources);         webcalls.register(this.calls);         // ... 省略中间其它初始化的代码         SecureServerSocket secserversocket = new SecureServerSocket(this.port); // 初始化 ssl socket 对指定的端口监听         while(true) {            secserversocket.acceptAndAuthenticate(this.pass, new PostAuthentication() { // 注意这里，直接传入 password 进行认证               public void clientAuthenticated(Socket socket) {                  try {                     socket.setSoTimeout(0);                     TeamSocket teamsocket = new TeamSocket(socket);                     (new Thread(new ManageUser(teamsocket, TeamServer.this.resources, TeamServer.this.calls), "Manage: unauth'd user")).start();                  } catch (Exception ex) {                     MudgeSanity.logException("Start client thread", ex, false);                  }               }            });         }      } catch (Exception e) {         MudgeSanity.logException("team server startup", e, false);      }   }   public static void main(String[] args) {      int port = CommonUtils.toNumber(System.getProperty("cobaltstrike.server_port", "50050"), 50050);      if (!AssertUtils.TestPort(port)) {         System.exit(0);      }      Requirements.checkConsole();      Authorization authorization = new Authorization();      License.checkLicenseConsole(authorization);      MudgeSanity.systemDetail("scheme", QuickSecurity.getCryptoScheme() + "");      if (args.length != 0 && (args.length != 1 || !"-h".equals(args[0]) && !"--help".equals(args[0]))) {         if (args.length != 2 && args.length != 3 && args.length != 4) {            CommonUtils.print_error("Missing arguments to start team server\n\t./teamserver <host> <password> [/path/to/c2.profile] [YYYY-MM-DD]");         } else if (!CommonUtils.isIP(args[0])) {            CommonUtils.print_error("The team server <host> must be an IP address. " + host_help);         } else if ("127.0.0.1".equals(args[0])) {            CommonUtils.print_error("Don't use 127.0.0.1 for the team server <host>. " + host_help);         } else if ("0.0.0.0".equals(args[0])) {            CommonUtils.print_error("Don't use 0.0.0.0 for the team server <host>. " + host_help);         } else if (args.length == 2) { // 没有指定 teamserver profile 的情况            MudgeSanity.systemDetail("c2Profile", "default");            TeamServer teamserver = new TeamServer(args[0], port, args[1], Loader.LoadDefaultProfile(), authorization);            teamserver.go();         } else if (args.length == 3 || args.length == 4) { // 指定了 profile 的情况            // ... 省略中间其它初始化的代码            TeamServer teamserver = new TeamServer(args[0], port, args[1], profile, authorization); // args[1] 是 teamserver 的 password            teamserver.go(); // 调用了上面 go 方法         }      } else {         CommonUtils.print_info("./teamserver <host> <password> [/path/to/c2.profile] [YYYY-MM-DD]\n\n\t<host> is the (default) IP address of this Cobalt Strike team server\n\t<password> is the shared password to connect to this server\n\t[/path/to/c2.profile] is your Malleable C2 profile\n\t[YYYY-MM-DD] is a kill date for Beacon payloads run from this server\n");      }   }}

再跟进 SecureServerSocket 类里面，看一下 acceptAndAuthenticate 的实现

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package ssl;public class SecureServerSocket {   protected ServerSocket server;   protected boolean authenticate(Socket socket, String password, String addr) throws IOException {      DataInputStream in = new DataInputStream(socket.getInputStream());      DataOutputStream out = new DataOutputStream(socket.getOutputStream());      int var6 = in.readInt();      if (var6 != 48879) {         CommonUtils.print_error("rejected client from " + addr + ": invalid auth protocol (old client?)");         return false;      } else {         int var7 = in.readUnsignedByte();         if (var7 <= 0) {            CommonUtils.print_error("rejected client from " + addr + ": bad password length");            return false;         } else {            StringBuffer sb = new StringBuffer();            for(int i = 0; i < var7; ++i) {               sb.append((char)in.readUnsignedByte());            }            for(int i = var7; i < 256; ++i) {               in.readUnsignedByte();            }            synchronized(this.getClass()) {               CommonUtils.sleep((long)CommonUtils.rand(1000));            }            if (sb.toString().equals(password)) { // 读 password               out.writeInt(51966);               return true;            } else {               out.writeInt(0);               CommonUtils.print_error("rejected client from " + addr + ": invalid password");               return false;            }         }      }   }   public Socket acceptAndAuthenticate(final String password, final PostAuthentication postauth) {      String addr = "unknown";      try {         final Socket socket = this.server.accept();         addr = socket.getInetAddress().getHostAddress();         (new Thread(new Runnable() {            public void run() {               String var1x = "unknown";               try {                  var1x = socket.getInetAddress().getHostAddress();                  if (SecureServerSocket.this.authenticate(socket, password, var1x)) { // 调用上面的 authenticate                     postauth.clientAuthenticated(socket);                     return;                  }               } catch (Exception var4x) {                  MudgeSanity.logException("could not authenticate client from " + var1x, var4x, false);               }                // ...            }         }, "accept client from " + addr + " (auth phase)")).start();      } catch (Exception e) {         MudgeSanity.logException("could not accept client from " + addr, e, false);      }      return null;   }    // ...}

password 这个东西，除了我们在CobaltStrike Client 上点了 Connect 按钮之后调用
aggressor.authenticate 时会用到之外，还会在 Client  和 TeamServer
通信过程中也会使用，除此之外还有很多处会用到(我太懒了没去找了)，所以我们如果把 password 字段作为 2FA
参与的字段的话，要改动的地方太多了。那么，不如换个思路，把 2FA TOTP Code 放在 nickname处，那么就只需要修改
server.ManageUser 这个类的 process 方法中关于 aggressor.authenticate 处理部分的逻辑就好了。

  

  

 **NO.2 动手实现**

如何操作呢，直接反编译 cs 源码重打包的方式固然可以，麻烦是麻烦了一点，但是我们不用，因为这种方式不够通用。Java 中提供了 javaagent
这么个东西, 熟悉 Burp 的同学应该会有印象，BurpLoader 就是用 javaagent 这种方式实现的破解：

  * 

    
    
    java -Dfile.encoding=utf-8 -javaagent:burp-loader.jar -noverify -jar burpsuite_pro_v2020.11.jar

我们完全可以借助 javaagent 和 javassist 对 cobaltstrike.jar 在加载前动态修改。

-javaagent 参数是可以加很多个的，也就是说，我们可以实现一堆的服务端的 **「插件」** ，想用哪个开哪个，还不用担心破坏原 jar 包。

核心代码如下:

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    try {  if (className == null) {    return classfileBuffer;  } else if (className.equals("server/ManageUser")) { // 只修改 ManageUser 类    CtClass cls = classPool.makeClass(new ByteArrayInputStream(classfileBuffer));    CtMethod ctmethod = cls.getDeclaredMethod("process",      new CtClass[]{        classPool.get("common.Request")      });    String func = "{"      + "if (!$0.authenticated && \"aggressor.authenticate\".equals($1.getCall()) && $1.size() == 3) {"      + "   java.lang.String mnickname = $1.arg(0)+\"\";"      + "   java.lang.String mpassword = $1.arg(1)+\"\";"      + "   java.lang.String mver = $1.arg(2)+\"\";"      + "   if(mnickname.length() < 6){ $0.client.writeObject($1.reply(\"Dynamic Code Error.\"));return; };" // 用户名如果低于 6 位就直接 return      + "   java.lang.String lastcode = de.taimos.totp.TOTP.getOTP(\""+totpSecretKey+"\");" // 生成 TOTP 6位数字      + "   if(!mnickname.substring(mnickname.length()-6, mnickname.length()).equals(lastcode)) {" // 比对动态口令，如果口令没对上，就 return      + "       $0.client.writeObject($1.reply(\"Dynamic Code Error.\"));return;"      + "   }"      + "}"      + "}";    ctmethod.insertBefore(func); // 把上面的代码插入到 process 函数最前面，如果口令正确，就继续走 cs 常规的流程    return cls.toBytecode();  }} catch (Exception ex) {    ex.printStackTrace();    System.out.printf("[CSTOTPAgent] PreMain transform Error: %s\n", ex);}

为了方便使用，我们再加一点小细节，比如说每次动态生成 totp secretkey 的函数，secretkey 生成二维码的功能啥的，就比较方便了。

  

最终打包成 CSTOTP.jar 用于使用

  

  

 **NO.3  ** **使用**

1、把生成好的 CSTOTP.jar 放到服务端和 cobaltstrike.jar 相同目录

没错，不需要给团队成员分发，只用放在 server 端就行了

![](https://gitee.com/fuli009/images/raw/master/public/20210803151342.png)

2、生成 TOTP SecretKey

  * 

    
    
    java -cp CSTOTP.jar com.maxwell.Main

不要奇怪包名为啥叫麦斯威尔，我自己随便写的

![](https://gitee.com/fuli009/images/raw/master/public/20210803151343.png)

3、掏出你的手机，下载 Google Authenticator (有些叫 谷歌身份验证器)，或者随便找一个支持 2FA 的 APP
就行。然后扫描上面的二维码，或者是手动把 SecretKey 添加进去就行。

这玩意儿请务必保存好，这再被人偷了就只能活该了。攻防演练前生成好，让团队成员扫描这个二维码即可

4、生成 SecretKey 之后，根据最后一行提示，把最后一行的内容添加到 teamserver 这个文件里

![](https://gitee.com/fuli009/images/raw/master/public/20210803151344.png)

红线上的内容之前是没有的，之后我们保存

5、你之前是怎么运行 teamserver 的，现在还是怎么运行

![](https://gitee.com/fuli009/images/raw/master/public/20210803151345.png)

比如我启动的 TeamServer 密码是 qax

  

  

 **NO.4 测试效果**

 **客户端无需任何额外的文件，你该怎么用就怎么用**

 **下图是以  agscript 方式登录**

![](https://gitee.com/fuli009/images/raw/master/public/20210803151346.png)

可以看到即使输入正确的密码，也是无法登录的，只有输入正确的密码，并且在 nickname后面加上有效的 6 位动态口令，才可以成功

  

 **换 UI 方式测试**

不带 TOTP Code

![]()

nickname 后面带上 TOTP Code 之后：

![](https://gitee.com/fuli009/images/raw/master/public/20210803151347.png)

完美~

  

  

  

 **RECRUITMENT**

 **招聘启事**

公司：安恒信息  
岗位： **高级攻防专家**  
部门： **西安运营中心**  
薪资：20-30K  
工作年限：3年+  
工作地点：西安  
  
【岗位职责】  
1.定期面向部门、全公司技术分享;  
2.前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3.负责完成部门渗透测试、红蓝对抗业务;  
4.负责自动化平台建设  
5.负责针对常见WAF产品规则进行测试并落地bypass方案  
  
【岗位要求】  
1.至少3年安全领域工作经验；  
2.精通HTTP协议相关技术  
3.拥有大型产品、CMS、厂商漏洞挖掘案例；  
4.熟练掌握php、java、asp.net代码审计基础（一种或多种）  
5.精通Web Fuzz模糊测试漏洞挖掘技术  
6.精通OWASP TOP 10安全漏洞原理并熟悉漏洞利用方法  
7.有过独立分析漏洞的经验，熟悉各种Web调试技巧  
8.精通常见编程语言中的至少一种（Asp.net、Python、php、java）  
9.掌握免杀、钓鱼、域渗透等技能

 **安恒雷神众测SRC运营（实习生）**  
————————  
【职责描述】  
1\.  负责SRC的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2\.  负责白帽子提交漏洞的漏洞审核、Rank评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3\.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4\.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5\.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
  
【任职要求】  
 1\.  责任心强，性格活泼，具备良好的人际交往能力；  
 2\.  对网络安全感兴趣，对行业有基本了解；  
 3\.  良好的文案写作能力和活动组织协调能力。

  

简历投递至

bountyteam@dbappsecurity.com.cn

 **设计师（实习生）**  

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
  
【职位要求】  
1、从事平面设计相关工作1年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；

3、精通photoshop/illustrator/coreldrew/等设计制作软件；  
4、有品牌传播、产品设计或新媒体视觉工作经历；  
  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦19楼  
学历要求：本科及以上  
工作年限：1年及以上，条件优秀者可放宽

  

简历投递至

bountyteam@dbappsecurity.com.cn

安全招聘  

————————  
  
公司：安恒信息  
岗位： **Web安全 安全研究员**  
部门：战略支援部  
薪资：13-30K  
工作年限：1年+  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
  
【岗位职责】  
1.定期面向部门、全公司技术分享;  
2.前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3.负责完成部门渗透测试、红蓝对抗业务;  
4.负责自动化平台建设  
5.负责针对常见WAF产品规则进行测试并落地bypass方案  
  
【岗位要求】  
1.至少1年安全领域工作经验；  
2.熟悉HTTP协议相关技术  
3.拥有大型产品、CMS、厂商漏洞挖掘案例；  
4.熟练掌握php、java、asp.net代码审计基础（一种或多种）  
5.精通Web Fuzz模糊测试漏洞挖掘技术  
6.精通OWASP TOP 10安全漏洞原理并熟悉漏洞利用方法  
7.有过独立分析漏洞的经验，熟悉各种Web调试技巧  
8.熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
  
【加分项】  
1.具备良好的英语文档阅读能力；  
2.曾参加过技术沙龙担任嘉宾进行技术分享；  
3.具有CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP等安全相关资质者；  
4.具有大型SRC漏洞提交经验、获得年度表彰、大型CTF夺得名次者；  
5.开发过安全相关的开源项目；  
6.具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7.个人技术博客；  
8.在优质社区投稿过文章；

  

岗位： **安全红队武器自动化工程师**  
薪资：13-30K  
工作年限：2年+  
工作地点：杭州（总部）  
  
【岗位职责】  
1.负责红蓝对抗中的武器化落地与研究；  
2.平台化建设；  
3.安全研究落地。  
  
【岗位要求】  
1.熟练使用Python、java、c/c++等至少一门语言作为主要开发语言；  
2.熟练使用Django、flask 等常用web开发框架、以及熟练使用mysql、mongoDB、redis等数据存储方案；  
3:熟悉域安全以及内网横向渗透、常见web等漏洞原理；  
4.对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5.具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
  
【加分项】  
1.有高并发tcp服务、分布式等相关经验者优先；  
2.在github上有开源安全产品优先；  
3:有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4.在freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5.具备良好的英语文档阅读能力。

  

简历投递至

bountyteam@dbappsecurity.com.cn

岗位： **红队武器化Golang开发工程师**  

薪资：13-30K  
工作年限：2年+  
工作地点：杭州（总部）  
  
【岗位职责】  
1.负责红蓝对抗中的武器化落地与研究；  
2.平台化建设；  
3.安全研究落地。  
  
【岗位要求】  
1.掌握C/C++/Java/Go/Python/JavaScript等至少一门语言作为主要开发语言；  
2.熟练使用Gin、Beego、Echo等常用web开发框架、熟悉MySQL、Redis、MongoDB等主流数据库结构的设计,有独立部署调优经验；  
3.了解docker，能进行简单的项目部署；  
3.熟悉常见web漏洞原理，并能写出对应的利用工具；  
4.熟悉TCP/IP协议的基本运作原理；  
5.对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
  
【加分项】  
1.有高并发tcp服务、分布式、消息队列等相关经验者优先；  
2.在github上有开源安全产品优先；  
3:有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4.在freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5.具备良好的英语文档阅读能力。  
  
简历投递至

bountyteam@dbappsecurity.com.cn

  

  

END

![](https://gitee.com/fuli009/images/raw/master/public/20210803151348.png)![]()![](https://gitee.com/fuli009/images/raw/master/public/20210803151349.png)

 **长按识别二维码关注我们**

  

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

为 CobaltStrike TeamServer 加上谷歌二次验证

最多200字，当前共字

__

发送中

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

