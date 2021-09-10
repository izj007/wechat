#  Burpsuit插件开发(Java篇)

原创 rebootORZ  [ 雷神众测 ](javascript:void\(0\);)

**雷神众测** ![]()

微信号 bounty_team

功能介绍 雷神众测，专注于渗透测试技术及全球最新网络攻击技术的分析。

____

__

收录于话题

  

**STATEMENT**

 **声明**

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  

  

 **NO.1 前言**

官方API文档：

https://portswigger.net/burp/extender/api/index.html

  * 

    
    
    Ps:也可以直接看burp suit-Extender-APIs功能下的API说明

官方的文档对于开发能力很一般的人来说可能不一定马上就能上手，所以建议找一些国人写的文档来看，方便理解。这里推荐以前乌云上的大佬写的文章进行学习，这里只找到内容，但是找不到原出处了：

  

\- 《BurpSuite插件开发指南之 Java 上篇》

https://blog.csdn.net/zhangmiaoping23/article/details/79786187

  

\- 《BurpSuite插件开发指南之 Java 下篇》  

https://blog.csdn.net/zhangmiaoping23/article/details/79786631?utm_medium=distribute.pc_relevant.none-
task-
blog-2~default~baidujs_baidulandingword~default-1.control&spm=1001.2101.3001.4242

  

\- 《BurpSuite插件开发指南之 API 篇》

https://blog.csdn.net/zhangmiaoping23/article/details/79787139

  

具体开发的时候建议先看这篇文章，会很有启发：

从头开发一个BurpSuite数据收集插件

https://www.secpulse.com/archives/124593.html

  

本文主要是记录学习的过程，文中的代码片段和概念会有些是从上文中摘抄过来，开发能力较弱，有些写的不好的地方各位可以改改。

  

  

 **NO.2  ** **开发环境**

编辑器 IDEAJ 2021.1  

电脑 Macbook 2019 pro

JDK 版本1.8.0_261

BurpSuit Professional v2.1.06

  

  

 **NO.3 准备工作**

项目中需要导入官方的SDK包，但是根据网上的说法，官方目前已经不提供SDK包了，我们可以自行从burpsuit中直接导出(需要选择一个空白目录)：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151145.png)

导出后会有一个burp文件夹，结构如下：

这些就是官方提供的所有接口了

![](https://gitee.com/fuli009/images/raw/master/public/20210910151152.png)

接下来，创建一个空白项目：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151154.png)

 ****然后在src目录下创建一个名为"burp"的package，并将前面下载下来的所有文件拷贝进去：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151155.png)

 ****至此开发前期的准备工作已完成。

  

  

 **NO.4 相关接口**

 **IBurpExtender**

****这个接口是官方规定的，在进行插件开发时必须要实现的一个接口，且实现的类的类名必须叫“BurpExtender.java”并声明为public，在这个类中，必须提供一个默认的构造方法，该类需要放在burp包中，如下：

  *   *   *   *   *   *   * 

    
    
     package burp;public class BurpExtender implements IBurpExtender{    @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){        // TODO here    }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151157.png)

 **IBurpExtenderCallbacks**

上面我们提到了，有一个默认的构造方法，即“registerExtenderCallbacks(final IBurpExtenderCallbacks
callbacks)”，该方法在插件被burpsuit加载后会被调用，这里它我们提供了一个IBurpExtenderCallbacks接口的实例，而这个接口提供了很多有用的方法。我们可以跟进这个方法看一下：

  

可以看到例如：输入输出、对其他模块的调用等，这里面都有提供。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151158.png)![](https://gitee.com/fuli009/images/raw/master/public/20210910151200.png)

 ****通过这个实例“callbacks”我们可以进行一系列操作，例如：

  *   *   *   *   *   *   * 

    
    
     package burp;public class BurpExtender implements IBurpExtender{    @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){        callbacks.setExtensionName("rebootORZ"); //设置扩展名称为 “rebootORZ”    }}

打包成jar包：

· File - Project Structure - Project Settings - Artifacts - “+” - JAR - From
modules with dependencies - 在Main Class中输入"BurpExtender" - OK

![](https://gitee.com/fuli009/images/raw/master/public/20210910151202.png)![](https://gitee.com/fuli009/images/raw/master/public/20210910151203.png)

· Build - Build Project  

· Build - Build Artifacts

· Build

![](https://gitee.com/fuli009/images/raw/master/public/20210910151204.png)

输出默认路径：  

out/artifacts/burpExterder_jar/burpExterder.jar

  

导入burpsuit效果如下：  

![](https://gitee.com/fuli009/images/raw/master/public/20210910151206.png)

 **IContextMenuFactory**

根据官方API文档的说明：  

“Extensions can implement this interface and then call
IBurpExtenderCallbacks.registerContextMenuFactory() to register a factory for
custom context menu items.”

  

“通过扩展实现该接口，然后调用 IBurpExtenderCallbacks.registerContextMenuFactory()
为自定义上下文菜单项注册工厂。”

  

简而言之，这里使用了工厂模式，我们需要先通过registerContextMenuFactory()注册一个工厂，然后通过实现IContextMenuFactory接口里面的createMenuItems()方法，就可以进行菜单的创建了。关于该接口的官方说明如下：

  

“This method will be called by Burp when the user invokes a context menu
anywhere within Burp. The factory can then provide any custom context menu
items that should be displayed in the context menu, based on the details of
the menu invocation.”

  

“当用户在 Burp 内的任何位置调用上下文菜单时，该方法将被 Burp
调用。然后，工厂可以根据菜单调用的详细信息，提供应该在上下文菜单中显示的任何自定义上下文菜单项。”

  

也就是说，要使用该接口需要先调用registerContextMenuFactory()注册一个上下文菜单项，然后才可以通过实现IContextMenuFactory接口去创建菜单，例如：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package burp;  
    import javax.swing.*;import java.util.ArrayList;import java.util.List;  
    public class BurpExtender implements IBurpExtender,IContextMenuFactory{    @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks) {        callbacks.setExtensionName("rebootORZ"); //设置扩展名称为 “rebootORZ”        callbacks.registerContextMenuFactory(this);    }  
      
        @Override    public List<JMenuItem> createMenuItems(final IContextMenuInvocation invocation) {  
            List<JMenuItem> listMenuItems = new ArrayList<JMenuItem>();        //子菜单        JMenuItem menuItem;        menuItem = new JMenuItem("子菜单测试");  
            //父级菜单        JMenu jMenu = new JMenu("rebootORZ");        jMenu.add(menuItem);        listMenuItems.add(jMenu);        return listMenuItems;    }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151207.png)

 **IContextMenuInvocation**

该接口的作用，官方的说法和网上的文章说的比较拗口，其实就是调用菜单的时候，用来获取一些相关信息，例如用来判断当前是在哪个功能里面进行调用的，是在"Intruder"还是"Repeater"等。  

其中提供了以下方法：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151208.png)

 ****例如，设置改上下文菜单只允许在Repeater模块中显示，代码如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import javax.swing.*;import java.util.ArrayList;import java.util.List;  
    public class BurpExtender implements IBurpExtender,IContextMenuFactory{    @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks) {        callbacks.setExtensionName("rebootORZ"); //设置扩展名称为 “rebootORZ”        callbacks.registerContextMenuFactory(this);    }  
        @Override    public List<JMenuItem> createMenuItems(final IContextMenuInvocation invocation) {  
            List<JMenuItem> listMenuItems = new ArrayList<JMenuItem>();  
            //判断是否是Repeater模块        if(invocation.getToolFlag() == IBurpExtenderCallbacks.TOOL_REPEATER) {            //子菜单            JMenuItem menuItem;            menuItem = new JMenuItem("子菜单测试");  
                //父级菜单            JMenu jMenu = new JMenu("rebootORZ");            jMenu.add(menuItem);            listMenuItems.add(jMenu);        }        return listMenuItems;    }}

 ****此时在其他模块中不会有该菜单：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151210.png)

 ****当在Repeater中就有了：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151211.png)

 **ICookie**

 ****这个接口用于获取 cookie 相关的信息，提供了以下方法：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151213.png)

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.io.PrintWriter;import java.util.List;  
    public class BurpExtender implements IBurpExtender, IHttpListener{  
        private PrintWriter stdout;    public IExtensionHelpers helpers;    public IRequestInfo iRequestInfo;    public List<ICookie> cookieList;    public String cookie="";  
      
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
      
            callbacks.setExtensionName("rebootORZ");        callbacks.registerHttpListener(this);        helpers = callbacks.getHelpers();        cookieList = callbacks.getCookieJarContents();  
            //设置输入输出流        this.stdout = new PrintWriter(callbacks.getStdout(), true);    }  
      
      
        public void GetCookie(IHttpRequestResponse messageInfo){  
            for(int i = 0; i<cookieList.size();i++){            String cookieName=cookieList.get(i).getName();            String cookieValue=cookieList.get(i).getValue();  
                if(i==0){                cookie = cookieName+"="+cookieValue;            }else{                cookie = cookie+"&"+cookieName+"="+cookieValue;  
                }  
            }        stdout.println(cookie+"\n");  
      
        }  
      
        @Override    public void processHttpMessage(int toolFlag, boolean messageIsRequest, IHttpRequestResponse messageInfo) {            if(messageIsRequest){  
                    this.GetCookie(messageInfo);  
                }  
        }}

 **IExtensionHelpers**

这个接口提供了很多常用的辅助方法，可以通过调用 IBurpExtenderCallbacks.getHelpers获得此接口的实例。  

  

这个接口提供的方法有非常多，对于我们编写插件比较常用的有以下：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151214.png)

 **IExtensionStateListener**  

这个接口在使用前需要先调用IBurpExtenderCallbacks.registerExtensionStateListener()
注册一个扩展的状态监听器，当扩展的状态发生改变时，监听器会收到通知并触发操作，相当于是在做善后的处理。  

  

这个接口只提供了一个卸载的方法，当然用户从burpsuit
unload这个插件的时候就会调用该方法并执行一些操作，具体操作的内容我们可以通过重写这个方法来实现。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package burp;  
    import java.io.PrintWriter;  
    public class BurpExtender implements IBurpExtender, IExtensionStateListener{  
        private PrintWriter stdout;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);  
            callbacks.setExtensionName("rebootORZ");        // 先注册扩展状态监听器        callbacks.registerExtensionStateListener(this);    }  
        // 重写 extensionUnloaded 方法    @Override    public void extensionUnloaded() {        // TODO        this.stdout.println("unload success ...");    }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151215.png)

 **IHttpListener**

使用该接口前，需要先通过 IBurpExtenderCallbacks.registerHttpListener() 注册一个 HTTP
监听器，这样所有的HTTP请求和响应都会通知这个监听器，就可以在这个监听器对HTTP数据包进行分析或修改了。  

  

该接口只提供了一个方法：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151216.png)

参数说明：  

· toolFlag
指的就是当前在哪个模块，可以根据模块筛选自己要的数据包，这个模块所对应的一个标志符，这个符号的定义在IBurpExtenderCallbacks接口中：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151217.png)

 **·  **messageIsRequest 用来判断该数据包是请求包（True）还是响应包（False）  

· messageInfo 该数据包的详细信息，是一个 IHttpRequestResponse 对象

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package burp; public class BurpExtender implements IBurpExtender, IHttpListener{     @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){         callbacks.setExtensionName("rebootORZ");        callbacks.registerHttpListener(this);    }     @Override    public void processHttpMessage(int toolFlag, boolean messageIsRequest, IHttpRequestResponse messageInfo) {         // TODO here    }}

 **IHttpRequestResponse**

这个接口用于获取/修改请求/响应的信息。  

  

提供的方法中，所有的set方法只能在消息被发送出去之前进行处理，但是在只读的上下文里面是不可以用的，因为set是写操作，而与响应有关的get方法也只能在请求发出后使用。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151218.png)![](https://gitee.com/fuli009/images/raw/master/public/20210910151219.png)

  

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.io.PrintWriter;import java.util.Arrays;import java.util.List;  
    public class BurpExtender implements IBurpExtender, IHttpListener{  
        private PrintWriter stdout;    public IExtensionHelpers helpers;    public IBurpExtenderCallbacks burpCallback;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            callbacks.setExtensionName("rebootORZ");        callbacks.registerHttpListener(this);  
            helpers = callbacks.getHelpers();  
            //设置输入输出流        this.stdout = new PrintWriter(callbacks.getStdout(), true);    }  
      
      
        private void runRequest(IHttpRequestResponse req) {        try {            byte[] rawRequest = req.getRequest();  
                IRequestInfo reqInfo = helpers.analyzeRequest(rawRequest);            // header of request should be a string            List<String> headers = reqInfo.getHeaders();            for(int h=0; h<headers.size(); h++){                //打印请求包                stdout.println(headers.get(h)+"\n");  
            } catch (Throwable e) {            PrintWriter writer = new PrintWriter(burpCallback.getStderr());            writer.write(e.getMessage());            writer.write("\n");            e.printStackTrace(writer);        }    }  
      
        @Override    public void processHttpMessage(int toolFlag, boolean messageIsRequest, IHttpRequestResponse messageInfo) {            if(messageIsRequest){  
                    this.runRequest(messageInfo);  
                }  
        }}

 **IHttpRequestResponseWithMarkers**

IHttpRequestResponseWithMarkers是 IHttpRequestResponse 接口的一个子接口，用于那些已被标记的
IHttpRequestResponse 对象，可用 IBurpExtenderCallbacks.applyMarkers()
创建一个此接口的实例，标记可用于各种情况，如指定Intruder 工具的 payload 位置，Scanner 工具的插入点或将 Scanner
工具的一些问题置为高亮。

  

此接口提供了两个分别操作请求和响应的方法：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151220.png)

 ****这两个方法的返回值均为一个整型数组列表，分别表示请求消息/响应消息标记偏移的索引对。列表中的每一项目都是一个长度为 2
的整型数组，包含标记开始和结束的偏移量。如果没有定义请求/响应标记，返回 null。

 **IHttpService**

此接口用于提供关于 HTTP 服务信息的细节。  

只提供了三个方法：

![]()

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package burp;  
    import java.io.PrintWriter;  
    public class BurpExtender implements IBurpExtender, IHttpListener{  
        private PrintWriter stdout;    public IBurpExtenderCallbacks iCallbacks;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);        callbacks.setExtensionName("rebootORZ");        callbacks.registerHttpListener(this);    }  
        @Override    public void processHttpMessage(int toolFlag, boolean messageIsRequest, IHttpRequestResponse messageInfo) {  
            IHttpService iHttpService = messageInfo.getHttpService();  
            this.stdout.println(iHttpService.getHost());        this.stdout.println(iHttpService.getPort());        this.stdout.println(iHttpService.getProtocol());    }}

 ****发送一个数据包，然后看输出：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151223.png)

 **IIntruderAttack**

 ****这个接口用于保存关于爆破的详细信息。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151224.png)

 **IInterceptedProxyMessage**

 ****此接口不能被扩展实现，它表示了已被 Burp 代理拦截的 HTTP 消息。扩展可以利用此接口注册一个 IProxyListener
以便接收代理消息的细节。

![]()

 **IIntruderPayloadGenerator**

 ****此接口被用于自定义 Intruder 工具的 payload 生成器。当需要发起一次新的 Intruder 攻击时，扩展需要注册一个
IIntruderPayloadGeneratorFactory 工厂并且必须返回此接口的一个新的实例。此接口会将当前插件注册为一个 Intruder
工具的 payload 生成器。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151225.png)

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    public class BurpExtender implements IBurpExtender, IIntruderPayloadGeneratorFactory{  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){        callbacks.setExtensionName("rebootORZ");        // 将当前插件注册为一个 Intruder 工具的 payload 生成器        callbacks.registerIntruderPayloadGeneratorFactory(this);    }  
        @Override    public String getGeneratorName() {        // 设置 payload 生成器名称        return "自定义 payload 生成器";    }  
        @Override    public IIntruderPayloadGenerator createNewInstance(IIntruderAttack attack) {        // 返回一个新的 payload 生成器的实例        return new IntruderPayloadGenerator();    }  
        // 实现 IIntruderPayloadGenerator 接口，此接口提供的方法是由 Burp 来调用的    class IntruderPayloadGenerator implements IIntruderPayloadGenerator{  
            @Override        public boolean hasMorePayloads() {            // TODO here            return false;        }  
            @Override        public byte[] getNextPayload(byte[] baseValue) {            // TODO here            return null;        }  
            @Override        public void reset() {            // TODO here  
            }    }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151226.png)

 **IIntruderPayloadGeneratorFactory**

 ****通过实现该接口，可以调用
IBurpExtenderCallbacks.registerIntruderPayloadGeneratorFactory() 注册一个自定义的
Intruder 工具的 payload 生成器。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151227.png)

 **IIntruderPayloadProcessor**

 ****通过实现该接口，可以调用 IBurpExtenderCallbacks.registerIntruderPayloadProcessor()
注册一个自定义 Intruder 工具的 payload 的处理器。此接口会将当前插件注册为一个 Intruder 工具的 payload 处理器。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151229.png)

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    public class BurpExtender implements IBurpExtender, IIntruderPayloadProcessor{  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){        callbacks.setExtensionName("rebootORZ");        // 将当前插件注册为一个 Intruder 工具的 payload 处理器        callbacks.registerIntruderPayloadProcessor(this);    }  
        // 此方法由 Burp 调用    @Override    public String getProcessorName() {        // 设置自定义 payload 处理器的名称        return "自定义 payload 处理器";    }  
        // 此方法由 Burp 调用，且会在每次使用一个 payload 发动攻击时都会调用一次此方法    @Override    public byte[] processPayload(byte[] currentPayload, byte[] originalPayload,                                 byte[] baseValue) {        // TODO here        return null;    }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151230.png)

 **IMessageEditor**

 ****此接口被用于使用 Burp 的 HTTP 消息编辑框的实例提供扩展功能，以便扩展插件可以在它自己的 UI 中使用消息编辑框，扩展插件可以通过调用
IBurpExtenderCallbacks.createMessageEditor() 获得此接口的实例。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151231.png)

 **IMessageEditorTab**

 ****扩展插件通过注册 IMessageEditorTabFactory 工厂，此工厂的 createNewInstance
返回一个当前接口的实例，Burp 将会在其 HTTP 消息编辑器中创建自定义的标签页。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151232.png)

 **IMessageEditorController**

 ****此接口被用于 IMessageEditor 获取当前显示的消息的细节。创建了 Burp 的 HTTP 消息编辑器实例的扩展插件可以有选择的实现
IMessageEditorController 接口，当扩展插件需要当前消息的其他信息时，编辑器将会调用此接口（例如：发送当前消息到其他的 Burp
工具中）。 扩展通过 IMessageEditorTabFactory 工厂提供自定义的编辑器标签页，此工厂的 createNewInstance
方法接受一个由该工厂所生成的每一个标签页的 IMessageEditorController 对象的引用，当标签页需要当前消息的其他信息时，则会调用该对象。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151233.png)

 **IMessageEditorTabFactory**

 ****通过实现该接口，可以调用 IBurpExtenderCallbacks.registerMessageEditorTabFactory()
注册一个自定义的消息编辑器标签页的工厂。扩展插件可以在 Burp 的 HTTP 编辑器中渲染或编辑 HTTP 消息。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151234.png)

示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # !javapackage burp; import java.awt.Component;import java.io.PrintWriter; public class BurpExtender implements IBurpExtender, IMessageEditorTabFactory{     public PrintWriter stdout;    public IExtensionHelpers helpers;    private IBurpExtenderCallbacks callbacks;     @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){         this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.callbacks = callbacks;        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        callbacks.registerMessageEditorTabFactory(this);    }     @Override    public IMessageEditorTab createNewInstance(            IMessageEditorController controller, boolean editable) {        // 返回 IMessageEditorTab 的实例        return new iMessageEditorTab();    }     class iMessageEditorTab implements IMessageEditorTab{         // 创建一个新的文本编辑器        private ITextEditor iTextEditor = callbacks.createTextEditor();         @Override        public String getTabCaption() {            // 设置消息编辑器标签页的标题            return "测试 MessageEditorTab";        }         @Override        public Component getUiComponent() {            // 返回 iTextEditor 的组件信息，当然也可以放置其他的组件            return iTextEditor.getComponent();        }         @Override        public boolean isEnabled(byte[] content, boolean isRequest) {            // 在显示一个新的 HTTP 消息时，启用自定义的标签页            // 通过 content 和 isRequest 也可以对特定的消息进行设置            return true;        }         @Override        public void setMessage(byte[] content, boolean isRequest) {            // 把请求消息里面的 data 参数进行 Base64 编码操作            // 这里并未处理参数中没有 data 时的异常            IParameter parameter = helpers.getRequestParameter(content, "data");            stdout.println("data = " + parameter.getValue());            iTextEditor.setText(helpers.stringToBytes(helpers.base64Encode(parameter.getValue())));        }         @Override        public byte[] getMessage() {            // 获取 iTextEditor 的文本            return iTextEditor.getText();        }         @Override        public boolean isModified() {            // 允许用户修改当前的消息            return true;        }         @Override        public byte[] getSelectedData() {            // 直接返回 iTextEditor 中选中的文本            return iTextEditor.getSelectedText();        }     }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151235.png)

 **IParameter**

 ****此接口用于操控 HTTP 请求参数，开发者通过此接口可以灵活的获取请求或响应里的参数。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151236.png)![](https://gitee.com/fuli009/images/raw/master/public/20210910151237.png)

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp; import java.io.PrintWriter;import java.util.List; public class BurpExtender implements IBurpExtender, IHttpListener{     public PrintWriter stdout;    public IExtensionHelpers helpers;    private IBurpExtenderCallbacks callbacks;     @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){         this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.callbacks = callbacks;        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        callbacks.registerHttpListener(this);    }     @Override    public void processHttpMessage(int toolFlag, boolean messageIsRequest,IHttpRequestResponse messageInfo) {        // 获取请求中的参数        if(messageIsRequest){            IRequestInfo iRequestInfo = helpers.analyzeRequest(messageInfo);            // 获取请求中的所有参数            List<IParameter> iParameters = iRequestInfo.getParameters();            for (IParameter iParameter : iParameters) {                if(iParameter.getType() == IParameter.PARAM_URL)                    stdout.println("参数：" + iParameter.getName() + " 在 URL中");                    stdout.println("参数：" + iParameter.getName() + " 的值为：" + iParameter.getValue());            }        }     }}

 ****先发送一个包：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151238.png)

 ****查看输出

![](https://gitee.com/fuli009/images/raw/master/public/20210910151239.png)

 ****获取Cookie:

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.io.PrintWriter;import java.util.List;  
    public class BurpExtender implements IBurpExtender, IHttpListener{  
        public PrintWriter stdout;    public IExtensionHelpers helpers;    private IBurpExtenderCallbacks callbacks;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.callbacks = callbacks;        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        callbacks.registerHttpListener(this);    }  
        @Override    public void processHttpMessage(int toolFlag, boolean messageIsRequest,IHttpRequestResponse messageInfo) {        // 获取请求中的参数        if(messageIsRequest){            IRequestInfo iRequestInfo = helpers.analyzeRequest(messageInfo);            // 获取请求中的所有参数            List<IParameter> iParameters = iRequestInfo.getParameters();            String cookie = "";            for (IParameter iParameter : iParameters) {                        //判断参数是否在Cookie中，如果是的话，将这些参数和值提取出来就是当前的Cookie了                if (iParameter.getType() == IParameter.PARAM_COOKIE) {                    if(cookie == ""){                        cookie = iParameter.getName()+"="+iParameter.getValue();                    }else {                        cookie = cookie + "&" + iParameter.getName()+"="+iParameter.getValue();                    }                }  
      
                }            stdout.println(cookie);        }  
        }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151240.png)

 **IProxyListener**

扩展可以实现此接口，并且可以通过调用
IBurpExtenderCallbacks.registerProxyListener()注册一个代理监听器。在代理工具处理了请求或响应后会通知此监听器。扩展插件通过注册这样一个监听器，对这些消息执行自定义的分析或修改操作。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151241.png)

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.io.PrintWriter;public class BurpExtender implements IBurpExtender, IProxyListener{  
        public PrintWriter stdout;    public IExtensionHelpers helpers;    private IBurpExtenderCallbacks callbacks;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.callbacks = callbacks;        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        callbacks.registerProxyListener(this);    }  
        @Override    public void processProxyMessage(boolean messageIsRequest,IInterceptedProxyMessage message) {        // TODO here        stdout.println("测试 IProxyListener");    }}

 ****先抓个包：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151242.png)

 ****再去查看插件的输出：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151243.png)

 **IRequestInfo**

 ****此接口被用于获取一个 HTTP 请求的详细信息。扩展插件可以通过调用 IExtensionHelpers.analyzeRequest()
获得一个 IRequestInfo 对象。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151244.png)

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.io.PrintWriter;  
    public class BurpExtender implements IBurpExtender, IHttpListener{  
        public PrintWriter stdout;    public IExtensionHelpers helpers;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        callbacks.registerHttpListener(this);    }  
        @Override    public void processHttpMessage(int toolFlag, boolean messageIsRequest,                                   IHttpRequestResponse messageInfo) {        // 打印出请求的 Url 和 响应码        if(messageIsRequest){            stdout.println(helpers.bytesToString(messageInfo.getRequest()));        }        else{            IResponseInfo responseInfo = helpers.analyzeResponse(messageInfo.getResponse());            short statusCode = responseInfo.getStatusCode();            stdout.printf("响应码 => %d\r\n", statusCode);        }    }}

 ****先发包：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151245.png)

 ****看输出：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151246.png)

 **IResponseInfo**

 ****此接口被用于获取一个 HTTP 请求的详细信息。扩展插件可以通过调用 IExtensionHelpers. analyzeResponse()
获得一个 IResponseInfo 对象。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151247.png)

 ****同理，发包后看输出：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151248.png)

 **IScanIssue**

 ****此接口用于获取 Scanner 工具扫描到的问题的细节。扩展可以通过注册一个
IScannerListener或者是通过调用IBurpExtenderCallbacks.getScanIssues()
获取扫描问题的细节。扩展同样可以通过注册 IScannerCheck
接口或者是调用IBurpExtenderCallbacks.addScanIssue() 方法来自定义扫描问题，此时扩展需要提供它对此接口的实现。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151249.png)![](https://gitee.com/fuli009/images/raw/master/public/20210910151250.png)

 **IScannerCheck**

 ****扩展可以实现此接口，之后可以通过调用 IBurpExtenderCallbacks.registerScannerCheck()注册一个自定义的
Scanner 工具的检查器。Burp 将会告知检查器执行“主动扫描”或“被动扫描”，并且在确认扫描到问题时给出报告。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151251.png)

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.io.PrintWriter;import java.util.List;  
    public class BurpExtender implements IBurpExtender, IScannerCheck{  
        public PrintWriter stdout;    public IExtensionHelpers helpers;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        callbacks.registerScannerCheck(this);    }  
        @Override    public List<IScanIssue> doPassiveScan(            IHttpRequestResponse baseRequestResponse) {        // TODO here        stdout.println("执行doPassiveScan");        return null;    }  
        @Override    public List<IScanIssue> doActiveScan(            IHttpRequestResponse baseRequestResponse,            IScannerInsertionPoint insertionPoint) {        // TODO here        stdout.println("执行doActiveScan");        return null;    }  
        @Override    public int consolidateDuplicateIssues(IScanIssue existingIssue,                                          IScanIssue newIssue) {        // TODO here        stdout.println("执行consolidateDuplicateIssues");        return 0;    }}

 ****对一个数据包进行scan-crawler and audit，当扫描执行到不同阶段时，会按照我们指定的代码输出如下内容：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151255.png)

 **IScannerInsertionPoint**

 ****此接口被用于定义一个用于Scanner工具检查器扫描的插入点。扩展可以通过注册 IScannerCheck 获得此接口实例，或者通过注册
IScannerInsertionPointProvider 创建一个 Burp 所使用的扫描检查器实例。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151256.png)

 **IScannerInsertionPointProvider**

 ****通过实现该接口，可以通过调用
IBurpExtenderCallbacks.registerScannerInsertionPointProvider() 注册自定义扫描插入点的工厂。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151257.png)

 **IScannerListener**

 ****通过实现该接口，可以通过调用 IBurpExtenderCallbacks.registerScannerListener() 注册一个
Scanner 工具的监听器。当 Scanner 工具扫描到新的问题时，会通知此监听器。扩展通过注册这样的监听器用于针对扫描问题自定义的分析和记录。

![]()

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.io.PrintWriter;  
    public class BurpExtender implements IBurpExtender, IScannerListener{  
        public PrintWriter stdout;    public IExtensionHelpers helpers;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        callbacks.registerScannerListener(this);    }  
        @Override    public void newScanIssue(IScanIssue issue) {        // TODO Auto-generated method stub        stdout.println("扫描到新的问题 :");        stdout.println("url => " + issue.getUrl());        stdout.println("详情 => " + issue.getIssueDetail());    }}

 ****使用scanner进行扫描并查看输出：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151258.png)

 **IScanQueueItem**

 ****  此接口被用于获取在 Burp 的 Scanner
工具中激活的扫描队列里的项目详情。扩展可以通过调用IBurpExtenderCallbacks.doActiveScan() 获得扫描队列项目的引用。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151259.png)

 **IScopeChangeListener**

 ****通过实现该接口，可以通过调用 IBurpExtenderCallbacks.registerScopeChangeListener() 注册一个
Target 工具下的 scope 变化监听器。当 Burp 的 Target 工具下的 scope 发生变化时，将会通知此接口。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151301.png)

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.io.PrintWriter;  
    public class BurpExtender implements IBurpExtender, IScopeChangeListener{  
        public PrintWriter stdout;    public IExtensionHelpers helpers;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        callbacks.registerScopeChangeListener(this);    }  
        @Override    public void scopeChanged() {        // 手动添加或右键菜单添加目标到 scope 列表，就会执行此方法        stdout.println("scope 有变化！");    }}

 ****添加到Scope查看插件输出：

![](https://gitee.com/fuli009/images/raw/master/public/20210910151302.png)

 **ISessionHandlingAction**

 ****通过实现该接口，可以通过调用 IBurpExtenderCallbacks.registerSessionHandlingAction()
注册一个自定义的会话操作动作。每一个已注册的会话操作动作在会话操作规则的UI中都是可用的，并且用户可以选择其中一个作为会话操作行为的规则。用户可以选择直接调用操作，也可以按照宏定义调用操作。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151303.png)

 **ITab**

 ****此接口用于自定义的标签页，调用 IBurpExtenderCallbacks.addSuiteTab() 方法可以在 Burp 的 UI
中显示自定义的标签页。

![]()

 ****示例代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
    import java.awt.Component;import java.io.PrintWriter;  
    import javax.swing.JButton;import javax.swing.JPanel;import javax.swing.SwingUtilities;  
    public class BurpExtender implements IBurpExtender, ITab{  
        public PrintWriter stdout;    public IExtensionHelpers helpers;  
        private JPanel jPanel1;    private JButton jButton1;  
        @Override    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){  
            this.stdout = new PrintWriter(callbacks.getStdout(), true);        this.helpers = callbacks.getHelpers();        callbacks.setExtensionName("rebootORZ");        SwingUtilities.invokeLater(new Runnable() {            @Override            public void run() {                //创建一个 JPanel                jPanel1 = new JPanel();                jButton1 = new JButton("点我");  
                    // 将按钮添加到面板中                jPanel1.add(jButton1);  
                    //自定义的 UI 组件                callbacks.customizeUiComponent(jPanel1);                //将自定义的标签页添加到Burp UI 中                callbacks.addSuiteTab(BurpExtender.this);            }        });    }  
        @Override    public String getTabCaption() {        // 返回自定义标签页的标题        return "rebootORZ";    }  
        @Override    public Component getUiComponent() {        // 返回自定义标签页中的面板的组件对象        return jPanel1;    }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151304.png)

 **ITempFile**

 ****此接口用于操作调用 IBurpExtenderCallbacks.saveToTempFile() 创建的临时文件。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151305.png)

 **ITextEditor**

 ****此接口用于扩展 Burp的原始文本编辑器，扩展通过调用 IBurpExtenderCallbacks.createTextEditor()
获得一个此接口的实例。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151306.png)

  

  

 **NO.5 Java Swing**

****Swing是java图形化编程中用到的一个包，而swing本身又是基于awt包扩展出来的。在Burpsuit插件编写中图形化相关知识用的并不多，所以这里只列举常用控件，并给出部分综合应用的示例代码，相关控件的使用可以从示例中去理解或者自行百度即可。

 **常用控件**

![](https://gitee.com/fuli009/images/raw/master/public/20210910151307.png)![](https://gitee.com/fuli009/images/raw/master/public/20210910151308.png)

 **示例代码**

 **SwingLoginActionExample.java**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     import javax.swing.JButton;import javax.swing.JFrame;import javax.swing.JLabel;import javax.swing.JPanel;import javax.swing.JPasswordField;import javax.swing.JTextField;import java.awt.event.ActionEvent;import java.awt.event.ActionListener;  
    public class SwingLoginActionExample {  
        public static void main(String[] args) {        // 创建 JFrame 实例        JFrame frame = new JFrame("Login Example");        // Setting the width and height of frame        frame.setSize(350, 200);        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);  
            /* 创建面板，这个类似于 HTML 的 div 标签         * 我们可以创建多个面板并在 JFrame 中指定位置         * 面板中我们可以添加文本字段，按钮及其他组件。         */        JPanel panel = new JPanel();        // 添加面板        frame.add(panel);        /*         * 调用用户定义的方法并添加组件到面板         */        placeComponents(panel);  
            // 设置界面可见        frame.setVisible(true);    }  
        private static void placeComponents(JPanel panel) {  
            /* 布局部分我们这边不多做介绍         * 这边设置布局为 null         */        panel.setLayout(null);  
            // 创建 JLabel        JLabel userLabel = new JLabel("User:");        /* 这个方法定义了组件的位置。         * setBounds(x, y, width, height)         * x 和 y 指定左上角的新位置，由 width 和 height 指定新的大小。         */        userLabel.setBounds(10,20,80,25);        panel.add(userLabel);  
            /*         * 创建文本域用于用户输入         */        JTextField userText = new JTextField(20);        userText.setBounds(100,20,165,25);        panel.add(userText);        //如果想要获取userText中用户输入的内容，不能直接在文本域这里创建监听事件，这样是获取不到内容的，需要在例如button的事件里面来获取（不知道为什么）  
      
            // 输入密码的文本域        JLabel passwordLabel = new JLabel("Password:");        passwordLabel.setBounds(10,50,80,25);        panel.add(passwordLabel);  
            /*         *这个类似用于输入的文本域         * 但是输入的信息会以点号代替，用于包含密码的安全性         */        JPasswordField passwordText = new JPasswordField(20);        passwordText.setBounds(100,50,165,25);        panel.add(passwordText);  
            // 创建登录按钮        JButton loginButton = new JButton("login");        loginButton.setBounds(10, 80, 80, 25);        panel.add(loginButton);        loginButton.addActionListener(new ActionListener() {            @Override            public void actionPerformed(ActionEvent e) {                // 进行逻辑处理即可  
                    System.out.println("用户名为：" + userText.getText());            }        });    }  
    }

![](https://gitee.com/fuli009/images/raw/master/public/20210910151309.png)

 **ListSelectionDemo.java**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     //列表监听事件Demo  
      
    import javax.swing.*;import javax.swing.event.*;import javax.swing.table.*;  
    import java.util.*;import java.awt.*;import java.awt.event.*;  
    public class ListSelectionDemo extends JPanel {    JTextArea output;    JList list;    JTable table;    String newline = "\n";    ListSelectionModel listSelectionModel;  
        public ListSelectionDemo() {        super(new BorderLayout());  
            String[] listData = { "one", "two", "three", "four",                "five", "six", "seven" };        String[] columnNames = { "French", "Spanish", "Italian" };        list = new JList(listData);  
            listSelectionModel = list.getSelectionModel();        listSelectionModel.addListSelectionListener(                new SharedListSelectionHandler());        JScrollPane listPane = new JScrollPane(list);  
            JPanel controlPane = new JPanel();        String[] modes = { "SINGLE_SELECTION",                "SINGLE_INTERVAL_SELECTION",                "MULTIPLE_INTERVAL_SELECTION" };  
            final JComboBox comboBox = new JComboBox(modes);        comboBox.setSelectedIndex(2);        comboBox.addActionListener(new ActionListener() {            public void actionPerformed(ActionEvent e) {                String newMode = (String)comboBox.getSelectedItem(); //获取当前下拉框选择的项内容                if (newMode.equals("SINGLE_SELECTION")) {                    listSelectionModel.setSelectionMode(                            ListSelectionModel.SINGLE_SELECTION);                } else if (newMode.equals("SINGLE_INTERVAL_SELECTION")) {                    listSelectionModel.setSelectionMode(                            ListSelectionModel.SINGLE_INTERVAL_SELECTION);                } else {                    listSelectionModel.setSelectionMode(                            ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);                }                output.append("----------"                        + "Mode: " + newMode                        + "----------" + newline);            }        });        controlPane.add(new JLabel("Selection mode:"));        controlPane.add(comboBox);  
            //Build output area.        output = new JTextArea(1, 10);        output.setEditable(false);        JScrollPane outputPane = new JScrollPane(output,                ScrollPaneConstants.VERTICAL_SCROLLBAR_ALWAYS,                ScrollPaneConstants.HORIZONTAL_SCROLLBAR_AS_NEEDED);  
            //Do the layout.        JSplitPane splitPane = new JSplitPane(JSplitPane.VERTICAL_SPLIT);        add(splitPane, BorderLayout.CENTER);  
            JPanel topHalf = new JPanel();        topHalf.setLayout(new BoxLayout(topHalf, BoxLayout.LINE_AXIS));        JPanel listContainer = new JPanel(new GridLayout(1,1));        listContainer.setBorder(BorderFactory.createTitledBorder(                "List"));        listContainer.add(listPane);  
            topHalf.setBorder(BorderFactory.createEmptyBorder(5,5,0,5));        topHalf.add(listContainer);        //topHalf.add(tableContainer);  
            topHalf.setMinimumSize(new Dimension(100, 50));        topHalf.setPreferredSize(new Dimension(100, 110));        splitPane.add(topHalf);  
            JPanel bottomHalf = new JPanel(new BorderLayout());        bottomHalf.add(controlPane, BorderLayout.PAGE_START);        bottomHalf.add(outputPane, BorderLayout.CENTER);        //XXX: next line needed if bottomHalf is a scroll pane:        //bottomHalf.setMinimumSize(new Dimension(400, 50));        bottomHalf.setPreferredSize(new Dimension(450, 135));        splitPane.add(bottomHalf);    }  
        /**     * Create the GUI and show it.  For thread safety,     * this method should be invoked from the     * event-dispatching thread.     */    private static void createAndShowGUI() {        //Create and set up the window.        JFrame frame = new JFrame("ListSelectionDemo");        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);  
            //Create and set up the content pane.        ListSelectionDemo demo = new ListSelectionDemo();        demo.setOpaque(true);        frame.setContentPane(demo);  
            //Display the window.        frame.pack();        frame.setVisible(true);    }  
        public static void main(String[] args) {        //Schedule a job for the event-dispatching thread:        //creating and showing this application's GUI.        javax.swing.SwingUtilities.invokeLater(new Runnable() {            public void run() {                createAndShowGUI();            }        });    }  
        class SharedListSelectionHandler implements ListSelectionListener {        public void valueChanged(ListSelectionEvent e) {            ListSelectionModel lsm = (ListSelectionModel)e.getSource();            //System.out.printf("LeadSelectionIndex is %s%n",lsm.getLeadSelectionIndex());            output.append("LeadSelectionIndex is " + lsm.getLeadSelectionIndex() + "\n");  
                int firstIndex = e.getFirstIndex();            int lastIndex = e.getLastIndex();            boolean isAdjusting = e.getValueIsAdjusting();            output.append("Event for indexes "                    + firstIndex + " - " + lastIndex                    + "; isAdjusting is " + isAdjusting                    + "; selected indexes:");  
                if (lsm.isSelectionEmpty()) {                output.append(" <none>");            } else {                // Find out which indexes are selected.                int minIndex = lsm.getMinSelectionIndex();                int maxIndex = lsm.getMaxSelectionIndex();                for (int i = minIndex; i <= maxIndex; i++) {                    if (lsm.isSelectedIndex(i)) {                        output.append(" " + i);                    }                }            }            output.append(newline);            output.setCaretPosition(output.getDocument().getLength());        }    }}

![](https://gitee.com/fuli009/images/raw/master/public/20210910151310.png)

 **表格Demo.java**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     import javax.swing.*;import java.awt.*;import java.awt.event.WindowAdapter;import java.awt.event.WindowEvent;  
    public class 表格Demo {    public 表格Demo() {        JFrame f = new JFrame();        Object[][] playerInfo = {                // 创建表格中的数据                { "王鹏", new Integer(91), new Integer(100), new Integer(191),                        new Boolean(true) },                { "朱学莲", new Integer(82), new Integer(69), new Integer(151),                        new Boolean(true) },                { "梅婷", new Integer(47), new Integer(57), new Integer(104),                        new Boolean(false) },                { "赵龙", new Integer(61), new Integer(57), new Integer(118),                        new Boolean(false) },                { "李兵", new Integer(90), new Integer(87), new Integer(177),                        new Boolean(true) },                { "李兵", new Integer(90), new Integer(87), new Integer(177),                        new Boolean(true) },                { "李兵", new Integer(90), new Integer(87), new Integer(177),                        new Boolean(true) },                { "李兵", new Integer(90), new Integer(87), new Integer(177),                        new Boolean(true) },                { "李兵", new Integer(90), new Integer(87), new Integer(177),                        new Boolean(true) },                { "李兵", new Integer(90), new Integer(87), new Integer(177),                        new Boolean(true) },                { "李兵", new Integer(90), new Integer(87), new Integer(177),                        new Boolean(true) },                { "李兵", new Integer(90), new Integer(87), new Integer(177),                        new Boolean(true) }};        // 创建表格中的横标题        String[] Names = { "姓名", "语文", "数学", "总分", "及格" };        // 以Names和playerInfo为参数，创建一个表格        JTable table = new JTable(playerInfo, Names);        // 设置此表视图的首选大小        table.setPreferredScrollableViewportSize(new Dimension(550, 100));        // 将表格加入到滚动条组件中        JScrollPane scrollPane = new JScrollPane(table);        f.getContentPane().add(scrollPane, BorderLayout.CENTER);        // 再将滚动条组件添加到中间容器中        f.setTitle("表格测试窗口");        f.pack();        f.setVisible(true);        f.addWindowListener(new WindowAdapter() {            @Override            public void windowClosing(WindowEvent e) {                System.exit(0);            }        });    }  
        public static void main(String[] args) {        表格Demo t = new 表格Demo();    }  
    }

![](https://gitee.com/fuli009/images/raw/master/public/20210910151311.png)

  

  

 **NO.6 开发实践**

 **解决问题**

****在实战中，很多站点，尤其是java站点，目录结构并非我们常见的层级目录，而是映射出来的，导致我们手中的字典经常不太符合实际需求，这个工具的用途就是从burp的历史流量中筛选我们要的目标，然后提取其所有URI进行组合，并根据我们提供的字典去探测可能存在的文件。

 **实现思路**

1.使用burpsuit IHttpRequestResponse接口，获取历史记录

2.根据用户输入的目标Host（可以是完整域名也可以部分，因为代码中用的是host.contains(str)进行匹配）匹配到的历史记录提取URI、host、协议、端口

3.例如：“/aa/bb/cc/”的URI会被处理成如下形式后存储到列表中：

“/aa”、“/aa/bb”、“/aa/bb/cc”

4.用户提供字典，获取到字典列表

ps: 这个字典不能太大，20以内最佳，因为该工具的初衷是用来发现敏感文件，例如web.xml，而不是做目录爆破。

 **核心代码**

 **burp/BurpExtender.java**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
      
    import java.awt.*;import java.util.*;import java.util.List;  
      
    public class BurpExtender implements IBurpExtender, ITab, IExtensionStateListener {  
        private FileScannerGUI fileScannerUI;  
        private IBurpExtenderCallbacks callbacks;  
      
      
        private String targetHost;  
      
      
        @Override    public void registerExtenderCallbacks(IBurpExtenderCallbacks callbacks) {        this.callbacks = callbacks;        /*            设置插件名称         */        callbacks.setExtensionName("FileScanner"); //这个是显示在插件中心的名字  
            //创建GUI界面类registerExtensionStateListener        fileScannerUI = new FileScannerGUI(BurpExtender.this);        //注册ITAB接口        callbacks.addSuiteTab(BurpExtender.this);  
            callbacks.registerExtensionStateListener(BurpExtender.this);        //加载后输出        callbacks.printOutput("Load Success...\nPowered By 安恒信息水滴实验室...\n");  
        }  
        @Override    public String getTabCaption() {        //返回标签页的名字        return "FileScanner";    }  
        @Override    public Component getUiComponent() {        //返回GUI界面主面板        return fileScannerUI.$$$getRootComponent$$$();    }  
      
        /*        获取数据包并解析  
            这里后面需要增加黑白名单功能，只获取要的目标host的路径、排除不要的目标     */  
        //String path;    public List<List<String>> getDir(String target) {        targetHost = target;        IHttpRequestResponse[] httpRequestResponses = callbacks.getProxyHistory();        IExtensionHelpers helpers = callbacks.getHelpers();  
            //创建一个List用来装所有path,这里不能用hashMap存host和path,因为host作为键的话会存在重复，对host的筛选应该放在解析数据包的时候        List<String> listPaths = new ArrayList();  
            //创建一个List用来存放host        List<String> listStr = new ArrayList();  
            // 获取所有路径        for (IHttpRequestResponse httpRequestResponse : httpRequestResponses) {            IRequestInfo requestInfo = helpers.analyzeRequest(httpRequestResponse);  
                //请求头host            String host = requestInfo.getUrl().getHost();  
                if(host.contains(targetHost)){  
      
                    //请求的URI                String fullPath = requestInfo.getUrl().getPath();  // /web/html/1.html  
                    //协议://host:端口                String str = requestInfo.getUrl().getProtocol() + "://" +requestInfo.getUrl().getHost() + ":" + requestInfo.getUrl().getPort();                listStr.add(str);  
                    //提取所有路径/aa/bb/cc ->  /aa/bb/cc 、/aa/bb 、 /aa                String[] paths = fullPath.split("/");                int len = paths.length;                if (checkPath(fullPath)) {                    for (int i = 1; i < len - 1; i++) {                        StringBuilder stringBuilder = new StringBuilder();                        for (int j = 1; j <= i; j++) {                            stringBuilder.append("/");                            stringBuilder.append(paths[j]);                        }                        stringBuilder.append("/");                        String resultPath = stringBuilder.toString();                        if (checkPath(resultPath)) { //检查路径是否符合要求                            listPaths.add(resultPath);  
                            }                    }                }            }else{  
                    continue;            }  
            }        /*            对path和host去重         */  
            HashSet set1 = new HashSet(listPaths);        // 清空list集合        listPaths.clear();        // 将去重后的元素重新添加到list中        listPaths.addAll(set1);  
            HashSet set2 = new HashSet(listStr);        // 清空list集合        listStr.clear();        // 将去重后的元素重新添加到list中        listStr.addAll(set2);  
            //将路径、没有URI的URL两个列表押入returnList列表中        List<List<String>> returnList = new ArrayList<>();        returnList.add(listPaths);        returnList.add(listStr);  
      
            return returnList;    }  
      
        @Override    public void extensionUnloaded() {        callbacks.printOutput("再见...");    }  
      
        /*        路径检查     */    private boolean checkPath(String path) {  
            if (path.length() > 256)            return false;  
            if (path.equals("/") || path.equals("//"))            return false;  
      
            return true;    }  
    }

 **/burp/FileScannerUI.java**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package burp;  
      
    import com.intellij.uiDesigner.core.GridConstraints;import com.intellij.uiDesigner.core.GridLayoutManager;  
    import javax.swing.*;import java.awt.*;import java.awt.event.ActionEvent;import java.awt.event.ActionListener;import java.io.*;  
    import java.util.List;import java.util.concurrent.ExecutorService;  
    import com.intellij.uiDesigner.core.Spacer;import utils.FixedThreadPool;import utils.ReadFileTool;import utils.HttpURLConnectionHelper;  
      
    public class FileScannerGUI {    private JPanel rootPanel;    private JTextField textField0;    private JButton jButton1;    private JButton jButton0;    private JTextField textField1;    private JButton jButton2;    private JTextArea textArea1;    private JTextArea textArea2;    private JLabel jLabel4;    private JLabel jLabel5;    private JLabel jLabel1;    private JPanel jPanel1;    private JLabel jLabel2;    private JScrollPane jscp4;    private JLabel jLabel6;    private JLabel jLabel3;    private JLabel textFieldForEmpty1;    private JLabel textFieldForEmpty2;  
      
        private List<List<String>> msg;  //用来存放获取到的path和host两个列表    private List<String> paths;  // 用来接收path列表    private List<String> str; //用来接收不带路径的url列表    private List<String> dictList; //用来存放字典转换后的列表  
      
        //创建一个文件选择器对象    private JFileChooser jFileChooser = new JFileChooser(new File("."));  
      
        JMenuItem x = new JMenuItem("Check for Struts RCE");  
        public FileScannerGUI(BurpExtender burpExtender) {  
            /*            选择文件的按钮事件         */        jButton1.addActionListener(new ActionListener() {            @Override            public void actionPerformed(ActionEvent e) {                //将文件选择器对象绑定在"选择文件"的按钮上                int status = jFileChooser.showOpenDialog(jButton1);//showOpenDialog()是打开/导入  
                    //判断是否选择了文件，如果选择了文件则去获取绝对路径                if (status == JFileChooser.FILES_ONLY) {                    File fo = jFileChooser.getSelectedFile();                    String absolutePath = fo.getAbsolutePath();                    //将文件路径输出在文本框中                    textField1.setText(absolutePath);  
                        /*                        获取到文件路径后，将文件的内容按行读取放进列表中                     */                    ReadFileTool readFileTool = new ReadFileTool();                    try {                        dictList = readFileTool.getList(absolutePath); //调用ReadFileTool类的getList方法，获取到字典列表                    } catch (FileNotFoundException fileNotFoundException) {                        fileNotFoundException.printStackTrace();                    }  
                    }  
                }        });  
      
            jButton0.addActionListener(new ActionListener() {  
                @Override            public void actionPerformed(ActionEvent e) {                /*                   HttpURLConnectionHelper实例化的代码必须写在这里，不然后面codeAndLength的调用内容是空的。                  */                HttpURLConnectionHelper httpURLConnectionHelper = new HttpURLConnectionHelper();                String targetHost = textField0.getText();                msg = burpExtender.getDir(targetHost); //getDir()的结果中有URI集合、不带URI的URL集合                paths = msg.get(0); //所有URI的集合                str = msg.get(1);  //不带URI的URL集合  
                    //先清空文本域                textArea1.setText("");                textArea2.setText("");                //将所有URI输出到textArea1                for (int i = 0; i < paths.size(); i++) {                    textArea1.append(paths.get(i) + "\n");                }  
      
                    /*                    创建后面用来发包的线程池                 */                FixedThreadPool fixedThreadPool = new FixedThreadPool();                ExecutorService threadPool = fixedThreadPool.fixedThreadPool();  
      
      
                    /*                    循环出每个URL                 */                //每个不带URI的URL                for (int i = 0; i < str.size(); i++) {                    //每个URI                    for (int j = 0; j < paths.size(); j++) {                        //字典的每一行                        for (int t = 0; t < dictList.size(); t++) {                            String url = str.get(i) + paths.get(j) + dictList.get(t);  
                                /*                                    多线程进行发包和实时更新文本域内容                             */  
                                threadPool.execute(() -> {                                httpURLConnectionHelper.doGet(url, textArea2);                                // textArea2.append(url + "   状态码：" + codeAndLength.get(0) + "  响应长度：" + codeAndLength.get(1) + "\n"+ codeAndLength.get(2) + "\n");//+ "   状态码：" + codeAndLength.get(0) + "  响应长度：" + codeAndLength.get(1) + "\n"+ codeAndLength.get(2) + "\n"                            });  
      
                            }  
                        }  
                    }  
                }        });  
      
            //按钮：保存路径        jButton2.addActionListener(new ActionListener() {            @Override            public void actionPerformed(ActionEvent e) {  
                    //将路径选择器对象绑定在"导出路径"的按钮上                int status = jFileChooser.showSaveDialog(jButton2); //showSaveDialog()是保存  
                    File f = jFileChooser.getSelectedFile();// f为选择到的目录                try {                    bufferedOutputStreamMethod(f.getAbsolutePath(), textArea1.getText());                } catch (IOException ioException) {                    ioException.printStackTrace();                }  
                }  
      
            });    }  
      
        // 文件导出函数    public static void bufferedOutputStreamMethod(String filepath, String content) throws IOException {        try (BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(                new FileOutputStream(filepath))) {            bufferedOutputStream.write(content.getBytes());        }    }  
      
        private void createUIComponents() {        // TODO: place custom component creation code here    }  
      
        {// GUI initializer generated by IntelliJ IDEA GUI Designer// >>> IMPORTANT!! <<<// DO NOT EDIT OR ADD ANY CODE HERE!        $$$setupUI$$$();    }  
        /**     * Method generated by IntelliJ IDEA GUI Designer     * >>> IMPORTANT!! <<<     * DO NOT edit this method OR call it in your code!     *     * @noinspection ALL     */    private void $$$setupUI$$$() {        rootPanel = new JPanel();        rootPanel.setLayout(new GridLayoutManager(9, 5, new Insets(0, 0, 0, 0), -1, -1));        jLabel5 = new JLabel();        jLabel5.setText("  Host 关 键 字：");        rootPanel.add(jLabel5, new GridConstraints(2, 1, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_NONE, GridConstraints.SIZEPOLICY_FIXED, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        textField0 = new JTextField();        rootPanel.add(textField0, new GridConstraints(2, 2, 1, 1, GridConstraints.ANCHOR_WEST, GridConstraints.FILL_HORIZONTAL, GridConstraints.SIZEPOLICY_WANT_GROW, GridConstraints.SIZEPOLICY_FIXED, null, new Dimension(150, -1), null, 0, false));        jLabel1 = new JLabel();        jLabel1.setText("字 典 路 径：");        rootPanel.add(jLabel1, new GridConstraints(3, 1, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_NONE, GridConstraints.SIZEPOLICY_FIXED, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        jButton1 = new JButton();        jButton1.setText("选择字典");        rootPanel.add(jButton1, new GridConstraints(3, 3, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_CAN_GROW, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        jPanel1 = new JPanel();        jPanel1.setLayout(new GridLayoutManager(1, 2, new Insets(0, 0, 0, 0), -1, -1));        rootPanel.add(jPanel1, new GridConstraints(1, 1, 1, 3, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_BOTH, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_CAN_GROW, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_CAN_GROW, null, null, null, 0, false));        jLabel4 = new JLabel();        jLabel4.setText("  注：本工具适合少量文件探测，不适合批量爆破。");        jPanel1.add(jLabel4, new GridConstraints(0, 0, 1, 2, GridConstraints.ANCHOR_WEST, GridConstraints.FILL_NONE, GridConstraints.SIZEPOLICY_FIXED, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        jButton0 = new JButton();        jButton0.setText("开  始");        rootPanel.add(jButton0, new GridConstraints(2, 3, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_CAN_GROW, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        textField1 = new JTextField();        rootPanel.add(textField1, new GridConstraints(3, 2, 1, 1, GridConstraints.ANCHOR_WEST, GridConstraints.FILL_HORIZONTAL, GridConstraints.SIZEPOLICY_WANT_GROW, GridConstraints.SIZEPOLICY_FIXED, null, new Dimension(150, -1), null, 0, false));        jLabel2 = new JLabel();        jLabel2.setText("存 在 路 径：");        rootPanel.add(jLabel2, new GridConstraints(4, 1, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_NONE, GridConstraints.SIZEPOLICY_FIXED, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        jButton2 = new JButton();        jButton2.setText("导出路径");        rootPanel.add(jButton2, new GridConstraints(4, 3, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_CAN_GROW, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        final JScrollPane scrollPane1 = new JScrollPane();        rootPanel.add(scrollPane1, new GridConstraints(4, 2, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_BOTH, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_WANT_GROW, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_WANT_GROW, null, null, null, 0, false));        textArea1 = new JTextArea();        scrollPane1.setViewportView(textArea1);        jscp4 = new JScrollPane();        rootPanel.add(jscp4, new GridConstraints(6, 1, 1, 3, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_BOTH, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_WANT_GROW, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_WANT_GROW, null, null, null, 0, false));        textArea2 = new JTextArea();        jscp4.setViewportView(textArea2);        jLabel6 = new JLabel();        jLabel6.setText("Powered By 安恒信息水滴实验室...");        rootPanel.add(jLabel6, new GridConstraints(7, 1, 1, 3, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_NONE, GridConstraints.SIZEPOLICY_FIXED, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        final JPanel panel1 = new JPanel();        panel1.setLayout(new GridLayoutManager(1, 1, new Insets(0, 0, 0, 0), -1, -1));        rootPanel.add(panel1, new GridConstraints(5, 1, 1, 3, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_BOTH, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_CAN_GROW, GridConstraints.SIZEPOLICY_CAN_SHRINK | GridConstraints.SIZEPOLICY_CAN_GROW, null, null, null, 0, false));        jLabel3 = new JLabel();        jLabel3.setText("URL：HTTP Status Code 200/302 And Content-Length > 0");        panel1.add(jLabel3, new GridConstraints(0, 0, 1, 1, GridConstraints.ANCHOR_WEST, GridConstraints.FILL_NONE, GridConstraints.SIZEPOLICY_FIXED, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        final Spacer spacer1 = new Spacer();        rootPanel.add(spacer1, new GridConstraints(2, 4, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, 1, 1, null, null, null, 1, false));        final Spacer spacer2 = new Spacer();        rootPanel.add(spacer2, new GridConstraints(4, 4, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, 1, 1, null, null, null, 1, false));        final Spacer spacer3 = new Spacer();        rootPanel.add(spacer3, new GridConstraints(3, 4, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, 1, 1, null, null, null, 1, false));        final Spacer spacer4 = new Spacer();        rootPanel.add(spacer4, new GridConstraints(5, 4, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, 1, 1, null, null, null, 1, false));        final Spacer spacer5 = new Spacer();        rootPanel.add(spacer5, new GridConstraints(6, 4, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, 1, 1, null, null, null, 1, false));        final Spacer spacer6 = new Spacer();        rootPanel.add(spacer6, new GridConstraints(7, 4, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_NONE, 1, 1, null, null, null, 1, false));        final Spacer spacer7 = new Spacer();        rootPanel.add(spacer7, new GridConstraints(5, 0, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_HORIZONTAL, 1, 1, null, null, null, 1, false));        textFieldForEmpty2 = new JLabel();        textFieldForEmpty2.setText("");        rootPanel.add(textFieldForEmpty2, new GridConstraints(8, 2, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_NONE, GridConstraints.SIZEPOLICY_FIXED, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));        textFieldForEmpty1 = new JLabel();        textFieldForEmpty1.setText("");        rootPanel.add(textFieldForEmpty1, new GridConstraints(0, 2, 1, 1, GridConstraints.ANCHOR_CENTER, GridConstraints.FILL_NONE, GridConstraints.SIZEPOLICY_FIXED, GridConstraints.SIZEPOLICY_FIXED, null, null, null, 0, false));    }  
        /**     * @noinspection ALL     */    public JComponent $$$getRootComponent$$$() {        return rootPanel;    }  
    }

 **最后效果**

 ****PS：UI有点丑，但是不影响使用，建议把swing好好学一下，最简单的就是用IDeaJ的Swing GUI
进行UI设计,拖拖拽拽最后实现功能即可。

![](https://gitee.com/fuli009/images/raw/master/public/20210910151312.png)

  

  

  

 **NO.7 总结**

 ****总的来说，burpsuit插件开发，需要先确定需求，然后可以结合IDeaJ的Swing GUI
功能，进行拖拽，将UI设计出来，最后再具体实现其中功能即可，如果是涉及到扫描、爆破，建议使用线程池进行操作，避免内存占用过高。由于前人写过的工具很多，想不明白的地方可以找前人写的工具，功能相似的插件借阅一下，找的到源码的就直接看，找不到源码的用jd-
gui反编译一下就能看了，通常来说并不会有人会对插件代码做加密。

  

  

  

 **RECRUITMENT**

 **招聘启事**

 **安恒雷神众测SRC运营（实习生）**  
————————  
【职责描述】  
1\.  负责SRC的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2\.  负责白帽子提交漏洞的漏洞审核、Rank评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3\.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4\.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5\.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
  
【任职要求】  
 1\.  责任心强，性格活泼，具备良好的人际交往能力；  
 2\.  对网络安全感兴趣，对行业有基本了解；  
 3\.  良好的文案写作能力和活动组织协调能力。

  

简历投递至

bountyteam@dbappsecurity.com.cn

 **设计师（实习生）**  

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
  
【职位要求】  
1、从事平面设计相关工作1年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；

3、精通photoshop/illustrator/coreldrew/等设计制作软件；  
4、有品牌传播、产品设计或新媒体视觉工作经历；  
  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦19楼  
学历要求：本科及以上  
工作年限：1年及以上，条件优秀者可放宽

  

简历投递至

bountyteam@dbappsecurity.com.cn

安全招聘  

————————  
  
公司：安恒信息  
岗位： **Web安全 安全研究员**  
部门：战略支援部  
薪资：13-30K  
工作年限：1年+  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
  
【岗位职责】  
1.定期面向部门、全公司技术分享;  
2.前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3.负责完成部门渗透测试、红蓝对抗业务;  
4.负责自动化平台建设  
5.负责针对常见WAF产品规则进行测试并落地bypass方案  
  
【岗位要求】  
1.至少1年安全领域工作经验；  
2.熟悉HTTP协议相关技术  
3.拥有大型产品、CMS、厂商漏洞挖掘案例；  
4.熟练掌握php、java、asp.net代码审计基础（一种或多种）  
5.精通Web Fuzz模糊测试漏洞挖掘技术  
6.精通OWASP TOP 10安全漏洞原理并熟悉漏洞利用方法  
7.有过独立分析漏洞的经验，熟悉各种Web调试技巧  
8.熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
  
【加分项】  
1.具备良好的英语文档阅读能力；  
2.曾参加过技术沙龙担任嘉宾进行技术分享；  
3.具有CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP等安全相关资质者；  
4.具有大型SRC漏洞提交经验、获得年度表彰、大型CTF夺得名次者；  
5.开发过安全相关的开源项目；  
6.具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7.个人技术博客；  
8.在优质社区投稿过文章；

  

岗位： **安全红队武器自动化工程师**  
薪资：13-30K  
工作年限：2年+  
工作地点：杭州（总部）  
  
【岗位职责】  
1.负责红蓝对抗中的武器化落地与研究；  
2.平台化建设；  
3.安全研究落地。  
  
【岗位要求】  
1.熟练使用Python、java、c/c++等至少一门语言作为主要开发语言；  
2.熟练使用Django、flask 等常用web开发框架、以及熟练使用mysql、mongoDB、redis等数据存储方案；  
3:熟悉域安全以及内网横向渗透、常见web等漏洞原理；  
4.对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5.具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
  
【加分项】  
1.有高并发tcp服务、分布式等相关经验者优先；  
2.在github上有开源安全产品优先；  
3:有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4.在freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5.具备良好的英语文档阅读能力。

  

简历投递至

bountyteam@dbappsecurity.com.cn

岗位： **红队武器化Golang开发工程师**  

薪资：13-30K  
工作年限：2年+  
工作地点：杭州（总部）  
  
【岗位职责】  
1.负责红蓝对抗中的武器化落地与研究；  
2.平台化建设；  
3.安全研究落地。  
  
【岗位要求】  
1.掌握C/C++/Java/Go/Python/JavaScript等至少一门语言作为主要开发语言；  
2.熟练使用Gin、Beego、Echo等常用web开发框架、熟悉MySQL、Redis、MongoDB等主流数据库结构的设计,有独立部署调优经验；  
3.了解docker，能进行简单的项目部署；  
3.熟悉常见web漏洞原理，并能写出对应的利用工具；  
4.熟悉TCP/IP协议的基本运作原理；  
5.对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
  
【加分项】  
1.有高并发tcp服务、分布式、消息队列等相关经验者优先；  
2.在github上有开源安全产品优先；  
3:有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4.在freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5.具备良好的英语文档阅读能力。  
  
简历投递至

bountyteam@dbappsecurity.com.cn

  

  

END

![](https://gitee.com/fuli009/images/raw/master/public/20210910151313.png)![](https://gitee.com/fuli009/images/raw/master/public/20210910151314.png)![]()

 **长按识别二维码关注我们**

  

  

  

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

Burpsuit插件开发(Java篇)

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

