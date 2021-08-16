##  基于ProxyPool创建Proxifier代理配置文件

原创 hesc  [ moonsec ](javascript:void\(0\);)

**moonsec** ![]()

微信号 moon_sec

功能介绍 暗月博客

____

__

收录于话题

#红队

1个

**基于ProxyPool创建Proxifier代理配置文件**![](https://gitee.com/fuli009/images/raw/master/public/20210816111621.png)

  

七夕优惠活动进行中 欢迎加入学习和咨询 [2021全栈渗透测试培训
七夕优惠活动](http://mp.weixin.qq.com/s?__biz=MzAwMjc0NTEzMw==&mid=2653574542&idx=1&sn=1621b5b2ad231d9d5306bdfdbbd43b58&chksm=811b43ccb66ccadaa71b7aff4440ffbab980fefa6a7048523719723849a27c79a60204ec3328&scene=21#wechat_redirect)  

  

 **1**  
前言

    刚开始学习安全渗透，想着用御剑去扫后台目录看有没有后台登录入口。扫着扫着我发现不能访问网站，浏览器界面提示访问重定向。问群里的大神才知道是 waf把此IP的请求给拦截，那就弄个代理池。不是单一IP地址进行访问，不会被waf拦截了吧！

![](https://gitee.com/fuli009/images/raw/master/public/20210816111623.png)

  

 **2**  
环境搭建

  

    使用github开源ProxyPool项目进行免费的代理IP进行收集、进行可用性验证。将已经验证过代理链接信息保存在Redis数据库中。并且支持docker部署

ProxyPool链接：https://github.com/jhao104/proxy_pool

  

###  **2.1安装Redis 数据库**

参考链接：https://www.cnblogs.com/hunanzp/p/12304622.html

  

 **2.2Redis** **遇到的坑**

  

  *   *   *   *   *   *   *   *   *   * 

    
    
     #修改redis.conf配置文件后需要重启redis服务requirepass ""#设置非密码访问,也可以注释掉。#取消redis安全模式#bind 127.0.0.1#取消只能本地链接daemonizeno#Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程，设置为noprotected-modeno #保护模式

  

###  **2.3安装拉取ProxyPool docker**

docker 安装方法请自行百度或谷歌

#### Docker运行  

  *   * 

    
    
    dockerpull jhao104/proxy_pool --拉取镜像dockerrun --env DB_CONN=redis://:password@ip:port/db -p 5010:5010jhao104/proxy_pool:latest --运行ProxyPool

样例：  

  *   * 

    
    
    docker run --en vDB_CONN=redis://:redis数据库密码(不要设置密码，后续执行创建Proxifier配置文件python脚本，使用非认证操作)@ip:port/数据库序号（可以默认写0）-p5010:5010 jhao104/proxy_pool:latestdocker run --env DB_CONN=redis://:@192.168.10.20:6379/0 -p 5010:5010jhao104/proxy_pool:lates

![]()  

RedisDesktopManager截图

![](https://gitee.com/fuli009/images/raw/master/public/20210816111626.png)

  

###  **2.4安装Proxifier**

Proxifier下载链接

傻瓜式安装、汉化版本  

  

#### 2.5Proxifier配置

开启HTTP代理服务器设置，因为有一些代理ip是走HTTP协议的。而Proxifier默认只支持HTTPS协议

![](https://gitee.com/fuli009/images/raw/master/public/20210816111627.png)

  

![]()

### 2.6 Proxifier配置文件生成

运行环境：Python3.6

模块：Redis,json,ElementTree

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # -*- coding:utf8 -*-import redisimport jsonfrom xml.etree import ElementTree  
    def RedisProxyGet():    ConnectString = []    pool = redis.ConnectionPool(host='192.168.10.20', port=6379, db=0, decode_responses=True)    use_proxy = redis.Redis(connection_pool=pool)    key = use_proxy.hkeys('use_proxy')    for temp in key:        try:            ConnectString.append(json.loads(use_proxy.hget('use_proxy',temp)))        except json.JSONDecodeError: # JSON解析异常处理            pass    return ConnectString  
    def xmlOutputs(data):    i = 101    ProxyIDList = []# ProxifierProfile根    ProxifierProfile = ElementTree.Element("ProxifierProfile")    ProxifierProfile.set("version", str(i))    ProxifierProfile.set("platform", "Windows")    ProxifierProfile.set("product_id", "0")    ProxifierProfile.set("product_minver", "310")  
    # Options 节点    Options = ElementTree.SubElement(ProxifierProfile, "Options")  
        # Options.Resolve    Resolve = ElementTree.SubElement(Options, "Resolve")    # Options.Resolve.AutoModeDetection    AutoModeDetection = ElementTree.SubElement(Resolve, "AutoModeDetection")    AutoModeDetection.set("enabled", "false")  
        # Options.Resolve.ViaProxy    ViaProxy = ElementTree.SubElement(Resolve, "ViaProxy")    ViaProxy.set("enabled", "false")  
        # Options.Resolve.ViaProxy.TryLocalDnsFirst    TryLocalDnsFirst = ElementTree.SubElement(ViaProxy, "TryLocalDnsFirst")    TryLocalDnsFirst.set("enabled", "false")  
        # Options.Resolve.ExclusionList    ExclusionList = ElementTree.SubElement(Resolve, "ExclusionList")    ExclusionList.text = "%ComputerName%; localhost; *.local"  
        # Options.*    Encryption = ElementTree.SubElement(Options, "Encryption")    Encryption.set("mode", 'basic')    Encryption = ElementTree.SubElement(Options, "HttpProxiesSupport")    Encryption.set("enabled", 'true')    Encryption = ElementTree.SubElement(Options, "HandleDirectConnections")    Encryption.set("enabled", 'false')    Encryption = ElementTree.SubElement(Options, "ConnectionLoopDetection")    Encryption.set("enabled", 'true')    Encryption = ElementTree.SubElement(Options, "ProcessServices")    Encryption.set("enabled", 'false')    Encryption = ElementTree.SubElement(Options, "ProcessOtherUsers")    Encryption.set("enabled", 'false')  
        # ProxyList    ProxyList = ElementTree.SubElement(ProxifierProfile, "ProxyList")    for temp in data:        i += 1  # 从101开始增加        # ProxyList.Proxy        Proxy = ElementTree.SubElement(ProxyList, "Proxy")        Proxy.set("id", str(i))  
            if not temp['https']:            Proxy.set("type", "HTTP")        else:            Proxy.set("type", "HTTPS")            Proxy.text = str(i)            ProxyIDList.append(i)  
            # ProxyList.Proxy.Address        Address = ElementTree.SubElement(Proxy, "Address")        Address.text = temp['proxy'].split(":", 1)[0]  
            # ProxyList.Proxy.Port        Port = ElementTree.SubElement(Proxy, "Port")        Port.text = temp['proxy'].split(":", 1)[1]  
            # ProxyList.Proxy.Options        Options = ElementTree.SubElement(Proxy, "Options")        Options.text = "48"  
        # RuleList    ChainList = ElementTree.SubElement(ProxifierProfile, "ChainList")  
        # RuleList.Chain    Chain = ElementTree.SubElement(ChainList, "Chain")    Chain.set("id", str(i))    Chain.set("type", "simple")  
        # RuleList.Chain.Name    Name = ElementTree.SubElement(Chain, "Name")    Name.text="AgentPool"  
        # RuleList.Chain.Proxy    for temp_id in ProxyIDList:        Proxy = ElementTree.SubElement(Chain, "Proxy")        Proxy.set("enabled", "true")        Proxy.text=str(temp_id)    # RuleList    RuleList = ElementTree.SubElement(ProxifierProfile, "RuleList")  
        # Rule    Rule = ElementTree.SubElement(RuleList, "Rule")    Rule.set("enabled", "true")    Name = ElementTree.SubElement(Rule,"Name")    Applications = ElementTree.SubElement(Rule,"Applications")    Action = ElementTree.SubElement(Rule,"Action")  
        Name.text="御剑后台扫描工具.exe [auto-created]"    Applications.text="御剑后台扫描工具.exe"    Action.set("type","Direct")  
        # Rule    Rule = ElementTree.SubElement(RuleList, "Rule")    Rule.set("enabled", "true")    Name = ElementTree.SubElement(Rule,"Name")    Targets = ElementTree.SubElement(Rule,"Targets")    Action = ElementTree.SubElement(Rule,"Action")  
        Name.text="Localhost"    Targets.text="localhost; 127.0.0.1; %ComputerName%"    Action.set("type", "Direct")  
        # Rule    Rule = ElementTree.SubElement(RuleList, "Rule")    Rule.set("enabled", "true")    Name = ElementTree.SubElement(Rule, "Name")    Action = ElementTree.SubElement(Rule, "Action")    Name.text = "Default"    Action.text = "102"    Action.set("type", "Proxy")  
        tree = ElementTree.ElementTree(ProxifierProfile)    tree.write("ProxifierConf.ppx", encoding="UTF-8", xml_declaration=True)if __name__ == '__main__':    proxy_data = RedisProxyGet()    xmlOutputs(proxy_data)    print("ProxifierConf.ppx配置文件创建完成....")

创建Proxifier配置文件：

![](https://gitee.com/fuli009/images/raw/master/public/20210816111628.png)

 _redis-to-xml_

Proxifier配置文件导入：

![](https://gitee.com/fuli009/images/raw/master/public/20210816111630.png)

## 开始奔放吧

![](https://gitee.com/fuli009/images/raw/master/public/20210816111631.png)

  

 **4**  
关注  

本公众号 更新安全类文章 欢迎关注和转发  

![](https://gitee.com/fuli009/images/raw/master/public/20210816111632.png)

  

  

  
  

  

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

基于ProxyPool创建Proxifier代理配置文件

最多200字，当前共字

__

发送中

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

