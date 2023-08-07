#  一种JDBC Attack的新方式

原创 pyn3rd  [ 跳跳糖社区 ](javascript:void\(0\);)

**跳跳糖社区** ![]()

微信号 tttangsec

功能介绍
跳跳糖是一个安全社区，旨在为安全人员提供一个能让思维跳跃起来的交流平台。希望你在这里能够找到灵感，找到志同道合的人。https://tttang.com/

____

___发表于_

收录于合集 #Java安全 35个

******点击** **蓝** **字  / 关注我们******

#### 背景

抽空在h1上看到了一个和JDBC Attack有关的案例，于是就简单看了下。

![]()image.png

https://hackerone.com/reports/1547877

这个案例中提到的Aiven是一家提供Saas化数据管理服务的云厂商，包括Apache
Kafka、Postgres、MySQL、Redis，所以自然就存在JDBC连接的场景。

由于漏洞已经修复，无法从控制台上看到相关的功能。不过可以根据漏洞作者的描述了解整个漏洞的利用过程：

  * • Aiven的Apache Kafka Connect Connector支持包括JDBC Sink Connector和HTTP Sink Connector等多种Sink Connector。

  * • 通过控制台对服务日志的查看，作者发现支持Jolokia，并且监听在`localhost:6725`

  * • HTTP Sink Connector允许发送HTTP请求到`localhost`并且Jolokia监听在本地的 `localhost:6725`

  * • Jolokia暴露了MBean `com.sun.management:type=DiagnosticCommand`, 而该Mbean存在一个operation `jvmtiAgentLoad`

  * • 由于HTTP Sink Connector没有校验目标是不是本地，所以可以通过`jvmtiAgentLoad` 这个operation可以加载一个本地的jar文件。

  * • 至于如何上传，作者通过JDBC Sink Connector的功能，利用SQLite JDBC driver在JDBC连接时创建一个数据库文件，再把恶意jar内容写入数据库的表里，SQLite的数据文件都是存在本地，可以通过JDBC连接的URL指定数据库文件的具体路径。

  * • 最后通过`jvmtiAgentLoad`加载指定的恶意jar，既数据库文件。

#### JVMTI和Instrument

上文在介绍漏洞利用过程时，提到Jolokia暴露的MBean `com.sun.management:type=DiagnosticCommand`
和它的operation `jvmtiAgentLoad`，这里涉及到另一个概念，叫`JVMTI`。

还是先了解下基本概念：`JVMTI（JVM Tool Interface）`是 Java 虚拟机所提供的 native
编程接口。JVMTI只是一套接口，我们要开发JVM工具就需要写一个Agent程序来使用这些接口。Agent程序其实就是一个C/C++语言编写的动态链接库。

所以加载恶意.so文件的方法可以实现RCE，不过Java在JDK
5开始引入了Instrument机制。利用Instrument接口，可以通过Java代码调用libinstrument的动态库与JVMTI接口进行交互，从而不需要再开发native的动态连结库文件，更加方便。

说到Instrument机制，它包含两种方式的整合形式，一种是main方法启动前执行，另一种是main方法内部通过attach来进行加载。

  * • premain（Agent模式）: 目标应用main方法启动前

    
    
    java -javaagent:/path/to/javaagent.jar -jar application.jar

其中，`-javaagent`需要在-jar的前面，如果在后面，不生效。

    
    
    public static void premain(String agentArgs, Instrumentation inst); public static void premain(String agentArgs);

premain方式相对简单，就是有一个javaagent的jar包，然后，在启动命令上把这个jar加上去之后，就会在启动main方法之前先运行这个premain方法。需要注意的是，要想使这个jar包知道启动哪一个premain方法，我们还需要在manifest文件里面进行定义。定义menifast的方法也有两种，一种是直接编写menifast文件，还有一种使用maven的插件进行编写。

  * • agentmain（Attach模式）: 目标应用之外，用一个attach应用将javaagent.jar注入到目标应用中

    
    
    public static void agentmain(String agentArgs, Instrumentation inst); public static void agentmain(String agentArgs);

attach方式相对要麻烦一些，需要单独起一个应用（或者使用一个另外的线程），通过`VirturalMachine.list()`找到所有运行的`VirtualMachineDescriptor`，匹配到目标应用之后，再把javaagent.jar注入到目标应用里面去。

有了以上这些知识就不难理解，我们可以直接通过`com.sun.management:type=DiagnosticCommand`
的`jvmtiAgentLoad`operation，使用attach的方式把我们的恶意agent注入进去，从而达到不重启应用也可以实现RCE。

#### 实现恶意Java Agent

其实非常的简单，因为JDK里提供了`premain`和`agentmain`两个静态方法，直接使用就好。这里使用的`agentmain` 直接attach

![]()image-20221112145203669.png

如果是手工创建MANIFEST.MF，需要指定Agent-Class，最后打成jar包

![]()image-20221112145446722.png

这里需要注意的是如果随便load一个文件，会存在以下报错

    
    
    "Agent_OnAttach is not available in /tmp/ext.so "

![]()image-20221112150842312.png

原因是JVMTI底层是通过`Agent_OnAttach`作为入口函数，然后执行以下流程，最终加载Java agent

  1. 1. 获取JNIEnv，保证已经成功attach到Java进程

  2. 2. 创建并初始化JPLISAgent、设置VMInit监听（不会触发了），逻辑与OnLoad相同

  3. 3. 读取Agent-Class并加载

  4. 4. 读取META-INFO相关配置，设置mRetransformEnvironment ClassFileLoadHook监听，逻辑与OnLoad相同

  5. 5. 创建InstrumentationImpl实例

  6. 6. 设置mNormaltransformEnvironment ClassFileLoadHook监听

  7. 7. 执行AgentMain方法

所以最终一个简单的恶意Java agent的代码如下：

    
    
    public class JavaAgent {  
        private static final String RCE_COMMAND = "open -a calculator";  
      
        public static void cmd() {  
            try {  
                Runtime.getRuntime().exec(RCE_COMMAND);  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
        }  
        public static void agentmain(String args, Instrumentation inst) {  
            System.out.println("In JavaAgent Agentmain");  
            cmd();  
        }  
    }

#### Jar文件的特性

这个漏洞利用了Jar文件的特性，说到特性就得先了解Jar的定义。以下是Oracle官方的定义

![]()image.png

两点有帮助的信息：

  * • Jar是基于Zip文件的一种文件格式

  * • Jar对文件名的要求不严格

所以我们可以像对Zip文件一样，把jar文件嵌入在其他文件里，而不影响其正常使用。

#### 演示测试效果

因为官方早已经修复了该漏洞，我在本地搭建了一个环境，演示以上两点的效果：

  * • 把恶意的Java agent写入SQLite数据库

  * • 在JDBC连接时，如果之前数据库文件不存在，会通过DriverManager的getConnection方法会自动创建一个SQLite数据库的文件 xxx.jar（文件后缀名对数据库的读取没有任何影响），并通过相应的SQL语句创建表。

  * • 把恶意的Java agent以Blob数据类型写入数据库，演示中是把打好的恶意agent.jar写入SQLite数据库文件test.jar

![]()image.png

  * • 加载恶意Java agent

    
    
    http://127.0.0.1:8099/actuator/jolokia/exec/com.sun.management:type=DiagnosticCommand/jvmtiAgentLoad/!/tmp!/test.jar

成功attach进去了恶意的Java agent，完成了RCE

![]()image.png

 ** ** **  
******

 **推荐阅读：**[初探HTTP Request
Smuggling](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247496564&idx=1&sn=e6aff4cd4026e06ae60866d1c77626b5&chksm=c172e2e5f6056bf37701382cc8024d2a03dd21467cc61b32e96adaea0d5bf442d039321af123&scene=21#wechat_redirect)
**  
**[2022蓝帽杯遇见的 SUID 提权
总结篇](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247495880&idx=1&sn=9f00650cb32efcf7fe5823d23501ec34&chksm=c172e159f605684fe29e76aeff68d8a4be8ee1d1f219139cc797c70e90300fd7e6c511ffd4f2&scene=21#wechat_redirect)[CobaltStrike
beacon二开指南](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247495740&idx=1&sn=885dc33cdb7f6468dcf526a0307d786d&chksm=c172e1adf60568bbb94ae398b01733beff4c9e196e47a7747c253c0990b06884c4addf46b406&scene=21#wechat_redirect)[Edge浏览器-
通过XSS获取高权限从而RCE](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247495016&idx=1&sn=be8e88c5d4508f93f5eb8479e76db5d8&chksm=c172fcf9f60575ef67b8434b2237d63ecb35a7f87dd92ef49f5a9184a513b7852d295065b68c&scene=21#wechat_redirect)[The
End of
AFR?](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247494920&idx=1&sn=59eb4c16e2047faf931198223d0f5575&chksm=c172fc99f605758fd36d8890a7413d6b3ec78bc4fe9552c2c57ed15b990e3d90ff2c1ffd3b39&scene=21#wechat_redirect)

* * *

跳跳糖是一个安全社区，旨在为安全人员提供一个能让思维跳跃起来的交流平台。

跳跳糖持续向广大安全从业者征集高质量技术文章，可以是漏洞分析，事件分析，渗透技巧，安全工具等等。通过审核且发布将予以500RMB-1000RMB不等的奖励，具体文章要求可以查看“投稿须知”。阅读更多原创技术文章，戳“阅读全文”

 ** ** **  
******

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

