#  系列 | 58集团IAST/RASP调研与实践：IAST调研

原创 wangyuhao  [ 58安全应急响应中心 ](javascript:void\(0\);)

**58安全应急响应中心** ![]()

微信号 wubasrc

功能介绍 58安全应急响应中心

____

__

收录于话题 #技术文章 5个内容

![](https://gitee.com/fuli009/images/raw/master/public/20211015084835.png)

**_背景_**

  
IAST（InteractiveApplication Security
Testing,交互式应用安全测试）、RASP（RuntimeApplicationSelf-
Protection,应用运行时自我保护），是两种在程序运行时提供安全能力的产品，由于这两款产品可以获取到程序内部运行中的状态，在程序运行状态视角，可以更准确的进行安全测试、及在运行时的入侵检测。目前针对于Java应用狭义上的IAST、RASP都是通过向应用字节码中插桩增强应用能力，增加漏洞检测逻辑或运行自我保护逻辑。本系列文章，主要介绍我们针对于IAST/RASP的技术调研与实践。包括：对IAST及RASP的调研，调研阶段面向现有的一些产品，对一些关键技术点进行分析总结。后期主要面向IAST/RASP的工程化落地。本文是IAST/RASP调研系列文章第一篇，主要介绍针对于Java场景实现IAST安全检测能力的方案。  
  
  
  
  
  

IAST简介

  
IAST-
交互式应用安全测试，是指在程序运行时，通过与应用交互的方式，收集程序中运行信息进行安全测试的手段。广义的IAST分为三类：①代理模式；②active插桩；③passive插桩。代理模式可以理解为现在的被动流量黑盒。active插桩通过与黑盒配合，黑盒触发检测流量，通过流量中的POC与插入的探针收集的信息结合，进行安全测试。Passive插桩模式下，不主动触发流量，只通过插桩的探针进程序运行信息收集，流量的触发主要通过开发或测试人员。![](https://gitee.com/fuli009/images/raw/master/public/20211015084842.png)本文主要针对第三种模式，passive插桩模式的IAST进行调研。  
  
  
  
  
  

 **如何实现插桩**

  
 **1、什么是插桩?**  
保证被测程序原有逻辑完整性的基础上在程序中插入一些探针。通过探针的执行并抛出程序运行的特征数据，通过对这些数据的分析，可以获得程序的控制流和数据流信息，进而得到逻辑覆盖等动态信息，从而实现测试目的的方法。  
总结一下对于探针的需求就是：(1)插入的探针逻辑不能影响原代码的逻辑完整性。(2)能够收集程序运行时的特征，用作测试的分析。  
 **2、JAVA中通过字节码增强插桩**  
![](https://gitee.com/fuli009/images/raw/master/public/20211015084843.png)  
在java从源代码，到编译字节码，再到运行时加载到jvm运行这个过程中，在(2)阶段，通过修改类的字节码，不直接修改java源代码，插入不影响源代码逻辑的字节码，来实现插桩的监听逻辑，即不影响源代码的逻辑完整性。  
如下，有这样一段代码，直观的观察一下字节码增强：

  *   *   *   *   *   *   *   * 

    
    
    package com.test;  
    public void a(Objecto){  
      //业务逻辑  System.out.println(o);  }

编译后对应的字节码为：

  *   *   *   *   *   *   * 

    
    
    public void print(java.lang.Object);  
      Code:  0:getstatic     #2 //Field java/lang/System.out:Ljava/io/PrintStream;  3:aload_1  4:invokevirtual #3//Method java/io/PrintStream.println:(Ljava/lang/Object;)V  7:return

现在如果有一个需求，在不影响Java源代码的基础上，输出o对象是否为null。即插入System.out.println(o==null);对应的字节码到该函数现有逻辑中。插入该段逻辑后的字节码变为：  

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public void print(java.lang.Object);  
      Code:    0:getstatic     #2 //Field java/lang/System.out:Ljava/io/PrintStream;  3:aload_1    4:ifnonnull     11    7:iconst_18:goto          12    11:iconst_0    12:invokevirtual #3//Method java/io/PrintStream.println:(Z)V   15:getstatic     #2//Field java/lang/System.out:Ljava/io/PrintStream;   18:aload_1   19:invokevirtual #4//Method java/io/PrintStream.println:(Ljava/lang/Object;)V   22:return

  
  
  
  
  
  

通过插桩实现污点分析

  
污点分析是安全测试中一个重要的能力，主要通过数据流向，确定外界输入是否会影响一些关键函数的执行。常见的注入类问题都可以通过污点分析进行测试，确定外界不可信输入是否可控制代码实际的执行逻辑。通过IAST实现污点分析，是IAST应该具有的一项重要的能力。
**1、通过插桩实现污点分析**

火线洞态IAST是一款优秀的IAST开源产品，通过对洞态的调研，梳理出对应的污点分析算法。以下通过一个简单示例进行污点分析的算法的介绍。

如下，有一段需要分析的java代码逻辑，是一段SQL注入的伪代码，通过外界获取参数id的输入，进行sql拼接，最终进行了sql的执行。![](https://gitee.com/fuli009/images/raw/master/public/20211015084844.png)

在污点分析的模型中，会有source、propagate、sanitizer、sink四个节点。简化模型，突出算法，没有在代码中体现sanitizer的净化。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //Stringid = request.getParameter("id");  
    String getParameter(Stringvar1){  
      //getParameter的业务逻辑  ....        //插入污点追踪逻辑    //inttaintedHash = System.identityHashCode(s)    //TaintedPool.add(taintedHash);    ....        returns  }

先对source点进行增强，针对getParameter函数，在return之前，函数本身的业务逻辑之后，插入污点分析逻辑。对于函数返回的s字符串对象，通过identityHashCode函数计算对象hash，用该hash表示对象唯一标识，并把hash放入TaintedPool中，表示该对象已经被污点标记。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //Stringsql1 = sql.concat(id);  
    public Stringconcat(String str){    //concat的业务逻辑  ...    //插入污点追踪逻辑  //intstrHash = System.identityHashCode(str)    //if(TaintedPool.contain(strHash)){      // int sHash = System.identityHashCode(s);    // TaintedPool.add(sHash);  //}    ...      returns  }

对于propagate,判断传入的str的hash是否在污点池里，如果在污点池里，把该函数的返回值的hash放到污点池里。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //ResultSetrs = st.executeQuery(sql);  
    ResultSet executeQuery(String sql) throws SQLException{  
      //插入污点追踪逻辑    //intsqlHash = System.identityHashCode(sql)    //if(TaintedPool.contain(sqlHash)){    //   Vul.Report(sqlHash,...);    //}        //executeQuery的业务逻辑    ...    returns  }

对于sink点，判断出入参数是否在TintedPool里，如果存在则上报漏洞。整个算法对source、propagate、sanitizer、sink中各个阶段的函数插入监听逻辑，监听污点source传入，propagate、sanitizer中进行传播或净化，sink监听是否传入受污染函数，如果sink监听到污点对象传入，则上报漏洞。  
 **2、规则抽象**  
对污点分析的四个阶段，定义source、propagate、sanitizer、sink四类增强类型，然后通过增加函数到各个阶段的规则中，添加各个阶段的的规则。  
![](https://gitee.com/fuli009/images/raw/master/public/20211015084846.png)  
（1）函数签名: 增强的具体函数。（2）污点输入:
对于该函数，污点是从哪里传入的，可以是O或pn。pn为该函数的哪个参数传入、O为当前类的base对象(比如
"x".concat("s")，字符串"x"也有可能是污点传入的位置)。（3）污点输出:
对于该函数，污点在哪里输出，可以为O、Pn或R，R返回值。（4）深度:
由于增强面向的可以是接口、抽象类或具体类。可以通过增强其子类，进而做到Hook到所以实现类。所以可以选择：当前类、当前类及子类、仅子类。（5）类型:
污点分析模型中的四个环节，source、propagate、sanitizer、sink。（6）漏洞类型:
sink或sanitizer对应的漏洞类型，被净化的污点数据，需要配合漏洞类型查看流到sink点，是否净化成功。  
  
  
  
  

SCA

  
通过Javaagent，实现SCA的收集，可以通过获取应用classpath下所依赖的Jar包，来收集应用的依赖信息，结合SCA信息可以进行更精确的安全检测。  
  
  
  
  

Skywalking实现IAST调研

  
Skywalking是apache下开源的APM(Application Performance
Management)系统，有监控、trace(调用链跟踪)、分布式诊断等功能，支持多种语言实现agent。Skywalking的javaagent也是基于字节码增强技术来实现监控逻辑的嵌入。其重要的trace功能，与污点分析的业务类似。Skywalking的trace功能是基于OpenTracing标准实现的，一个trace就代表着对一个请求的完整跟踪。追踪一个请求在单个或一组应用中的执行顺序，对关键的节点进行切面收集执行响应时间、监听运行时状态(关键函数入参、返回值等)。关键节点可以通过插件指定对应的具体函数，如在应用边界的队列生产、消费函数、数据库的客户端调用、RPC的客户端调用与服务端执行等，也可以是应用内具体的一个内部函数的执行情况。  
![](https://gitee.com/fuli009/images/raw/master/public/20211015084847.png)  
对于一次请求trace，有一个唯一的traceID会传到执行到的关键节点，且跟随着trace的传播也可以传递上下文信息跟随trace流动，可以把对应的污点分析上下文放入到trace的上下文中，通过添加插件来覆盖对应的source、propagate、sanitizer、sink对应的函数，从而达到污点分析的功能。

原理上可行，能够满足安全检测的需求。是否可以基于skywalking实现IAST，还需要考虑到工程化实现，如trace的业务是否适合污点分析，及兼容的开发成本，还有agent对性能严格的要求等。在编写demo时也遇到了skywalking默认不支持修改Jre的类，如果需要绑定，就可能需要修改插件外的代码等问题。工程化不在此次调研的范围中，对于安全检测能力，以下的技术要点可以给IAST实现带来一些启发:

1、Skywalking的插件，除了通过指定类的全限定名以及函数签名来指定待增强的函数，还可以通过Java注解来指定待增强的类及函数。这个给规则抽象增加了注解的维度。如58内部的RPC服务，就是通过注解来指定服务端逻辑或者客户端逻辑的。2、Skywalking的trace功能，提供了跨线程、跨进程的跟踪，及上下文的传递，跨进程的跟踪，实现了在一组应用中进行污点分析的可能，不只局限于单个应用内的分析。这在内部服务丰富，调用关系复杂的场景下是一个重要的需求，因为source点和sink点可能存在于一组服务中的任意位置。  
![](https://gitee.com/fuli009/images/raw/master/public/20211015084848.png)  
Skywalking是如何实现跨进程的trace的?比如在一个HTTP的请求过程中，通过对HTTP客户端进行增强，把此次trace上下文等信息封装进自定义的HTTP头中，通过HTTP协议传到服务端，HTTP服务端增强HTTP请求处理逻辑，把trace信息从HTTP请求头中拆出，然后进行后续的逻辑。这样就达到了trace跨应用的的传递。其他的跨进程trace同理。  
  
  
  
  
  

IAST、SAST、DAST能力互补

  
目前58集团已经现有较为成熟的DAST、SAST。增加IAST，是增强了哪些维度的能力。以下是简单的列举了几个维度上的对比。![](https://gitee.com/fuli009/images/raw/master/public/20211015084849.png)  
（1）控制流的覆盖：在SAST视角直接面向代码逻辑，可以穷举完所有逻辑分支，拿到控制流。DAST、IAST是运行触发有限的请求输入，不能做到完全的覆盖。  
![](https://gitee.com/fuli009/images/raw/master/public/20211015084850.png)  
（2）脏数据: SAST静态分析，不产生脏数据。IAST被动监听运行时状态，不产生脏数据。DAST产生脏数据。（3）动态特性:
IAST、DAST对于动态特性，优于SAST。比如JAVA反射，或者调用Native code。（4）主动、被动:
IAST只能做到被动监听，SAST/DAST可以做到主动扫描。（5）应用运行时内部信息收集:
IAST能够收集到应用运行时信息，SAST/DAST不能。（6）误报：SAST为了保证soundness，在一些维度上是存在误报的，如核心的指向分析。IAST/DAST相对于SAST误报低。

（7）漏报：SAST在源码层面，能拿到所有的控制流等信息，所以漏报相对于IAST/DAST较低。IAST较DAST能拿到程序运行时状态，漏报优于DAST。

针对不同维度的能力对比，就是不同系统之间能力上的互补：（1）针对控制流覆盖，可以通过SAST给DAST提供控制流覆盖更完善的POC。（2）DAST给IAST提供主动触发业务执行的请求。（3）IAST收集代码内执行的信息，给DAST判定提供更详细的信息，也可以进行FUZZ，优化POC。（4）通过DAST/IAST结合，验证SAST的误报。  
  
  
  
  
  

具体实现

  
由于面向Java来实现IAST，所以需要Java这个平台提供实现IAST的基础能力。

 **1、Javaagent技术**

JDK1.5 之后引入的新特性，此特性为用户提供了在 JVM 将字节码文件读入内存之后，JVM 使用对应的字节流在 Java 堆中生成一个 Class
对象之前，用户可以对其字节码进行修改的能力，从而 JVM 也将会使用用户修改过之后的字节码进行 Class
对象的创建。Javaagent有两种加载方式:（1）premain: 在运行java应用时，通过java
-javaagent:/path/to/agent.jar，agent需要实现premain函数，premain逻辑在main函数逻辑之前执行。（2）attach:
对于已经在运行的java应用，通过指定Jvm的PID进行加载，需要实现AgentLauncher类以及agentmain函数。 **2.字节码增强**
字节码增强的工具有很多，包括ASM、byte-buddy、Javassist等。  
  
  
  
  
  

总  结

  
  
针对于IAST的核心功能污点分析，火线洞态提供了完整的污点分析算法及规则的抽象。针对于内部服务复杂的甲方场景，同样也需要skywalking这种全局的视角实现跨应用的分析。此次调研汲取这些优秀的设计特点，为后续IAST建设提供检测能力的技术支撑。  
  
  
  
  
  

参  考

  
(1)https://baike.baidu.com/item/%E7%A8%8B%E5%BA%8F%E6%8F%92%E6%A1%A9/242087?fr=aladdin(2)https://mp.weixin.qq.com/s/GpiiLaaygPU1oPIk2YGQRg(3)https://www.aqniu.com/learn/46910.html(4)https://github.com/HXSecurity/DongTai(5)https://github.com/apache/skywalking  
  
  
分享收藏点赞在看

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

系列 | 58集团IAST/RASP调研与实践：IAST调研

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

