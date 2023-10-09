#  Cobalt Strike系列｜从0开始破解

原创 黔灵山Xo.  [ TERRA星环安全团队 ](javascript:void\(0\);)

**TERRA星环安全团队** ![]()

微信号 TERRASEC

功能介绍 团队专注于漏洞挖掘、漏洞研究、红蓝对抗、CTF竞赛、溯源取证、威胁情报、代码审计、逆向分析等研究。

____

___发表于_

收录于合集 #工具分享 7个

![]()

  

  

**0x01   ****Cobalt Strike介绍**  

Cobalt
Strike(简称为CS)是一款基于java的渗透测试工具，将所有攻击都变得简单、可视化，团队成员可以连接到同一个服务器上进行多人运动，共享攻击资源。

官网地址：https://www.cobaltstrike.com

![]()

  

 **0x02**   **Cobalt Strike二开环境搭建**

最近想学习一下cs系列的二次开发，网上也有不少教程，但是在学习过程中也还是遇到不少坑，最开始是想研究下4.7版本的破解，但是4.7及以上的版本对于防破解做了很多改变，不再是双端共用了，服务端的二进制文件无疑是加大了破解难度，所以选择4.5版本进行研究学习，双端java，新手好评。

下载官方cs4.5版本jar包，记得校验sha256的值

校验地址：https://verify.cobaltstrike.com/

![]()![]()

笔者从网上下载的版本比对值是正确的，但是里面有一个签名，反编译后查看源码并没有其他后门，问题不大，不影响后续的破解学习。

![]()

  

 **首先反编译客户端源码**

使用IntelliJ IDEA自带的反编译工具java-decompiler，在idea目录的plugins\java-
decompiler\lib下,新建两个文件夹用来存放原版和反编译后的客户端文件，并把java-decompiler复制到当前工作目录

![]()

使用如下命令进行反编译

  * 

    
    
    java -cp java-decompiler.jar org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler -dsg=true Original/cobaltstrike-client.jar Decompiler

 **配置IDEA编译环境**

打开idea新建一个空白项目

![]()

在根目录新建一个lib文件夹用来存放原版的客户端。将直接解压的反编译后客户端文件夹复制进项目。

![]()

进入file->project Structure->Modules->Dependencies->Library-Java 配置lib环境

![]()

文件选择之前我们的原版jar文件，添加后选中并保存。

![]()

进入file->project Structure->Artifacts—>jar—>from modules with
dependencies创建main class，填入入口aggressor.Aggressor，该函数可以在lib->cobaltstrike-
client.jar->META-INF->MENIFEST.MF中找到

![]()![]()

再将lib->cobaltstrike-client.jar->META-INF->MENIFEST.MF替换到src目录

![]()

后续需要修改的源码直接从反编译的文件目录复制该java文件到src对应的目录下即可。修改入口文件来测试一下。

![]()![]()

进入src目录下的Aggressor文件添加一行启动输出

  * 

    
    
    JOptionPane.showMessageDialog(null, "Welcome to 泰若星环安全团队!");

![]()

修改后重新编译生成jar包 Build–>Build Artifacts–>Build

![]()

复制jar包路径添加到启动配置中

![]()

配置启动参数

  * 

    
    
    -XX:ParallelGCThreads=4 -XX:+AggressiveHeap -XX:+UseParallelGC

![]()

完成后点击运行，成功出现我们添加的内容,然后报错退出，接下来就开始破解了。

![]()

  

 **0x03**    **Cobalt Strike破解**

关于4.5版本的破解网上也有很多教程，有的是加载外部jar包动态修改，先简单的说一下破解思路，首先是修改客户端的启动校验，然后修改运行中的暗桩，两个部分。

 **所有源码的修改都要复制源文件到src的对应目录下进行修改。**

需要修改以下几个地方的代码，可直接注释。

beacon/BeaconData.java

![]()

  

beacon/CommandBuilder.java    保持var1=true就行，也可以直接注释最后的if语句

![]()![]()

  

common/Authorization.java
主要授权校验函数，直接注释掉cobaltstrike.auth文件的校验，然后重新给var4赋值一个密钥，这个值是网上找的大佬破解密钥文件后的结果。

  * 

    
    
    byte[] var4 = {1, -55, -61, 127, 0, 1, -122, -96, 45, 16, 27, -27, -66, 82, -58, 37, 92, 51, 85, -114, -118, 28, -74, 103, -53, 6, 16, -128, -29, 42, 116, 32, 96, -72, -124, 65, -101, -96, -63, 113, -55, -86, 118, 16, -78, 13, 72, 122, -35, -44, 113, 52, 24, -14, -43, -93, -82, 2, -89, -96, 16, 58, 68, 37, 73, 15, 56, -102, -18, -61, 18, -67, -41, 88, -83, 43, -103, 16, 94, -104, 25, 74, 1, -58, -76, -113, -91, -126, -90, -87, -4, -69, -110, -42, 16, -13, -114, -77, -47, -93, 53, -78, 82, -75, -117, -62, -84, -34, -127, -75, 66, 0, 0, 0, 24, 66, 101, 117, 100, 116, 75, 103, 113, 110, 108, 109, 48, 82, 117, 118, 102, 43, 86, 89, 120, 117, 119, 61, 61};

![]()

  

common/Helper.java    注释掉class文件的调用，保证var2=true

![]()

  

common/Starter.java    注释掉class文件的调用，保证var2=true

![]()![]()

  

common/Starter2.java    注释掉class文件的调用，保证var2=true

![]()

到此破解结束，不需要额外的程序，便可以直接启动客户端和服务端。

客户端启动

  * 

    
    
    java -XX:ParallelGCThreads=4 -XX:+AggressiveHeap -XX:+UseParallelGC -Duser.language=en -jar cobaltstrike.jar

服务端启动

  * 

    
    
    java -XX:ParallelGCThreads=4 -Dcobaltstrike.server_port=50050 -Dcobaltstrike.server_bindto=0.0.0.0 -Djavax.net.ssl.keyStore=./cobaltstrike.store -Djavax.net.ssl.keyStorePassword=111111 -server -XX:+AggressiveHeap -XX:+UseParallelGC -classpath ./cobaltstrike.jar -Duser.language=en server.TeamServer 192.168.1.1 1111

创建ssl证书

  * 

    
    
    keytool -keystore ./cobaltstrike.store -storepass 111111 -keypass 111111 -genkey -keyalg RSA -alias cobaltstrike -dname "CN=Major Cobalt Strike, OU=AdvancedPenTesting, O=cobaltstrike, L=Somewhere, S=Cyberspace, C=Earth"

成功运行并上线

![]()

如果想个性化可以修改以下文件内容

aggressor/AggressorClient.java  设置cs标题

![]()

  

resources/default.profile    修改配置，比如默认beacon心跳时间为1分钟，可以设置短一点

![]()

  

 **公众号后台回复cs4.5获取源码，带汉化补丁和vnc插件**

 **0x04   ** **免责声明**

  

    本文仅限于技术研究学习，切勿将文中技术细节用作非法用途，如有违者后果自负。  

 **关于我们**

  

 **“TERRA星环”安全团队**
正式成立于2020年，是贵州泰若数字科技有限公司旗下以互联网攻防技术研究为目标的安全团队。团队核心成员长期从事渗透测试、代码审计、应急响应等安服工作，多次参与国家、省级攻防演练行动，具备丰富的安服及攻防对抗经验。

团队专注于漏洞挖掘、漏洞研究、红蓝对抗、CTF夺旗、溯源取证、威胁情报、代码审计、逆向分析等研究。
**对外提供安全评估、安全培训、安全咨询、安全集成、应急响应等服务。**

  

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

