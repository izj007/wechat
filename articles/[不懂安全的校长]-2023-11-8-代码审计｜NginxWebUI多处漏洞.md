#  代码审计｜NginxWebUI多处漏洞

原创 校长  [ 不懂安全的校长 ](javascript:void\(0\);)

**不懂安全的校长** ![]()

微信号 sectip

功能介绍 校长不懂安全，总是发些奇奇怪怪的笔记！

____

___发表于_

收录于合集

## 0x01 前言

目标信息，源码名称:nginxWebUI，下载地址:https://www.nginxwebui.cn/

![]()

因为下载下来的是jar包，所以我们通过反编译来拿到源码。

    
    
    官方说明:https://www.nginxwebui.cn/product.html  
      
    windows启动如下：  
    java -jar -Dfile.encoding=UTF-8 D:/home/nginxWebUI/nginxWebUI.jar --server.port=8080 --project.home=D:/home/nginxWebUI/  
    

## 0x02 获取源码

这里推荐的两个反编译工具

  * 1.luyten
  * 2.jd-gui

这里我使用的 **jd-gui** 这两款软件 在github 均可以搜到下载

![]()

选择 "Save All sources" 导出该项目，导出成功后 使用IDEA去打开该项目

![]()![]()

我们在这里开始远程JVM调试，一般我们java web远程debug的话  只要设置好三要素

  * 1.主机
  * 2.端口
  * 3.JDK版本

如图  服务端的地址为192.168.238.132  端口为8089 复制给出的命令行实参

![]()

    
    
    添加到启动jar包的cmd命令:  
    java  -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8089 -jar -Dfile.encoding=UTF-8 D:/home/nginxWebUI/nginxWebUI.jar --server.port=8081 --project.home=D:/home/nginxWebUI/   
    

![]()![]()

## 0x03 审计结果

#### 1\. 登录无视验证码爆破

![]()

首先去对应路由关系 清楚运行的逻辑 就能快速去挖掘了，这里登录抓包 url 为 `/adminPage/login/getAuth` 相当于为
**contoller/adminpage/Logincontoller/** 方法直接找Mapping绑定

这样对应上了就知道是怎么传参 就能找到对应源码进行审计

![]()

这边形参 用户可以直接传参 如下不为null 就可以绕过验证码生成验证的这个流程

![]()![]()

爆破name 和pass的话只要经过两次base64解码就可以

![]()

#### 2\. 权限绕过

config下目录AppFillter 类发现doFilter方法，通过全局过滤器发现这里校验不严格,可以大小写绕过。

![]()

只要是，只通过全局过滤器鉴权的路由的都可以进行未授权操作

#### 3\. 任意文件上传

![]()

通过全局搜索upload 找上传点

![]()

上传参数为file，这里直接是原生文件名。上传到了/tmp目录，不过文件名可控可以跨目录

![]()![]()

所以我们只需要网站的绝对路径就可以getshell。不知道他获取文件绝对路径用的哪个函数，直接正常登录系统看了一遍功能点，发现在conf这个类里打印了网站的绝对路径。

![]()

直接定位到该源码

![]()

不过，突然想起是jar包启动的。如果是类似tomcat的这种就能Getshell了。

![]()

#### 4\. RCE - 执行恶意jar包(1)

![]()

    
    
     @Mapping("/adminPage/main/autoUpdate")  
     public JsonResult autoUpdate(String url) {  
    //  if (!SystemTool.isLinux()) {  
    //   return renderError(m.get("commonStr.updateTips"));  
    //  }  
      
      File jar = JarUtil.getCurrentFile();  
      String path = jar.getParent() + "/nginxWebUI.jar.update";  
      LOG.info("download:" + path);  
      HttpUtil.downloadFile(url, path);  
      updateUtils.run(path);  
      return renderSuccess();  
     }  
    

获取jar包的绝对路径，然后远程下载回来。然后进入一个run方法，跟进run方法查看。

    
    
     public void run(String path) {  
      ThreadUtil.safeSleep(2000);  
      
      String newPath = path.replace(".update", "");//  
      FileUtil.rename(new File(path), newPath, true);//将下载到的文件去除后缀.update  
      
      String param = " --server.port=" + port + " --project.home=" + home;  
      
      if ("mysql".equals(type.toLowerCase())) {  
       param += " --spring.database.type=" + type //  
         + " --spring.datasource.url=" + url //  
         + " --spring.datasource.username=" + username //  
         + " --spring.datasource.password=" + password;  
      }  
      
      String cmd = null;  
      if (SystemTool.isWindows()) {  
       cmd = "java -jar -Dfile.encoding=UTF-8 " + newPath + param;  
      } else {  
       cmd = "nohup java -jar -Dfile.encoding=UTF-8 " + newPath + param + " > /dev/null &";  
      }  
      
      LOG.info(cmd);  
      RuntimeUtil.exec(cmd);  
    

 **RuntimeUtil.exec(cmd);** 最后一行执行命令，回溯到cmd命令，是 **java -jar    newpath**
是我们可控的，也就是说我们能控制他运行的jar文件，所以我们在自己的服务上放一个恶意jar包让他来下载并且执行。

#### 5\. RCE - 2

![]()

 **confController.runCmd** 这里他调用了conf控制器类的 **runCmd方法**
跟进去看，这里调用了exec方法，可以直接RCE。

![]()![]()

#### 6\. RCE - 3

以此之外，我们还可以去寻找其他调用这个runCmd方法的类，挖掘出更多的RCE

![]()

    
    
    public JsonResult cmdOver(String[] remoteId, String cmd, Integer interval) {  
      if (remoteId == null || remoteId.length == 0) {  
       return renderSuccess(m.get("remoteStr.noSelect"));  // 需要对remoteId进行传参  
      }  
      
      StringBuilder rs = new StringBuilder();  
      for (String id : remoteId) {                
       JsonResult jsonResult = null;  
       if (id.equals("local")) {   //remoteId 进入流程  
        if (cmd.contentEquals("check")) {  
         jsonResult = confController.checkBase();  
        }  
        if (cmd.contentEquals("reload")) {  
         jsonResult = confController.reload(null, null, null);  
        }  
        if (cmd.contentEquals("replace")) {  
         jsonResult = confController.replace(confController.getReplaceJson(), null);  
        }  
        if (cmd.startsWith("start") || cmd.startsWith("stop")) {   //cmd值开头为start 进入RCE  
         jsonResult = confController.runCmd(cmd.replace("start ", "").replace("stop ", ""), null); //会自动将start 替换为空    
        }  
    

![]()

#### 7\. 任意文件读取

![]()

## 0x04 结尾

承接红蓝对抗、安全众测、安全培训、CTF代打、CTF培训、PHP / JAVA / GO / Python 代码审计、渗透测试、应急响应
等等的安全项目，请联系下方微信。

![]()

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

