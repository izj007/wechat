[ BaCde's Blog ](/)

自由工作者，劈柴喂马周游世界

  * [ __主页](/)
  * [ __文章列表](/archives/)
  * [ __汽车安全](/tag/prvJjB7Fua5//)
  * [ __安全工具](/tag/G8-CImT42BP//)
  * [ __原创小说](/tag/JJyH5v-0vnk//)
  * [ __文章标签](/tags/)
  * [ __友情链接](/friends/)

文章目录 站点概览

![](/images/avatar.png)

BaCde

[ 50 文章 ](/archives/)

[ 147 分类 ]()

[ 147 标签 ](/tags/)

__[RSS](https://bacde.me/atom.xml)

[ __](https://rapiddns.io) [ __](https://github.com/insightglacier) [
__](https://twitter.com/insightGlacier)

  *     * MQTT协议
      * 发布/订阅模式
    * MQTT的攻击点
      * MQTT的利用
    * 探测与发现
    * mqtt 安全建议
    * 小结
    * 引用

#  [ 物联网安全之MQTT协议安全 ](https://bacde.me/post/mqtt-security-part-one/)

__ 发布于 2020-08-18 | __ 分类于 [ automotive ](https://bacde.me/tag/prvJjB7Fua5/)、
[ mqtt ](https://bacde.me/tag/cu9Tg0uRbMd/)、 [ car hacking
](https://bacde.me/tag/JdYDio18R72/)、 [ mqtt-pwn ](https://bacde.me/tag/tCcj-
UHo7-S/)、 [ car ](https://bacde.me/tag/NB7AfDzqlwN/) | __ 9分钟 | __ 2351字数

MQTT 全称为 Message Queuing Telemetry Transport（消息队列遥测传输）是是ISO 标准(ISO/IEC PRF
20922)下基于发布/订阅范式的消息协议，由 IBM
发布。由于其轻量、简单、开放和易于实现的特点非常适合需要低功耗和网络带宽有限的IoT场景。比如遥感数据、汽车、智能家居、智慧城市、医疗医护等。

## MQTT协议

MQTT协议为大量计算能力有限，低带宽、不可靠网络等环境而设计，其应用非常广泛。目前支持的服务端程序也较丰富，其PHP，JAVA，Python，C，C#等系统语言也都可以向MQTT发送相关消息。
目前最新的版本为5.0版本，可以在https://github.com/mqtt/mqtt.github.io/wiki/servers
这个连接中看到支持MQTT的服务端软件。 其中hivemq中提到针对汽车厂商的合作与应用，在研究过程中会发现有汽车行业应用了MQTT协议。

![](https://p1.ssl.qhimg.com/t01f74a80563218bc7a.png)

以下列举我们关心的几项：

  1. 使用发布/订阅的消息模式，支持一对多的消息发布；
  2. 消息是通过TCP/IP协议传输；
  3. 简单的数据包格式；
  4. 默认端口为TCP的1883，websocket端口8083，默认消息不加密。8883端口默认是通过TLS加密的MQTT协议。

### 发布/订阅模式

MQTT协议中有三种角色和一个主要概念，三种角色分别是发布者（PUBLISHER）、订阅者（SUBCRIBER）、代理（BROKER），还有一个主要的概念为主题（TOPIC）。

消息的发送方被称为发布者，消息的接收方被称为订阅者，发送者和订阅者发布或订阅消息均会连接BROKER，BROKER一般为服务端，BROKER存放消息的容器就是主题。发布者将消息发送到主题中，订阅者在接收消息之前需要先“订阅主题”。每份订阅中，订阅者都可以接收到主题的所有消息。

![](https://p0.ssl.qhimg.com/t01b3844d8b306b3f7c.jpg)

其MQTT协议流程图如下： ![](https://p1.ssl.qhimg.com/t01c8644cc806e6316e.png)

这里不对协议进行过多介绍，感兴趣的大家可以结尾处的引用查看。

## MQTT的攻击点

根据其特性，可以扩展如下几个攻击点：

  1. 授权：匿名连接问题，匿名访问则代表任何人都可以发布或订阅消息。如果存在敏感数据或指令，将导致信息泄漏或者被恶意攻击者发起恶意指令；
  2. 传输：默认未加密，则可被中间人攻击。可获取其验证的用户名和密码；
  3. 认证：弱口令问题，由于可被爆破，设置了弱口令，同样也会存在安全风险；
  4. 应用：订阅端明文配置导致泄漏其验证的用户名和密码；
  5. 漏洞：服务端软件自身存在缺陷可被利用，或者订阅端或服务端解析内容不当产生安全漏洞，这将导致整个系统。

### MQTT的利用

目前已经有针对MQTT的开源利用工具，这里主要以mqtt-pwn这块工具为主。mqtt-
pwn这块工具功能强大易用。github地址为<https://github.com/akamai-threat-research/mqtt-
pwn>，使用文档地址为<https://mqtt-pwn.readthedocs.io/en/latest/>。

**工具安装**  
mqtt-pwn的安装很简单。可以直接安装到本机，也可以直接使用docker的方式。  
文本直接使用docker方式。首先确保已经安装docker和docker-compose。然后运行如下命令进行安装:

    
    
    git clone https://github.com/akamai-threat-research/mqtt-pwn.git
    cd mqtt-pwn
    docker-compose up --build --detach
    

运行：

    
    
    docker-compose ps
    docker-compose run cli
    

即可看到mqtt-pwn的界面。  
![](https://p2.ssl.qhimg.com/t0145aba7cf70a0a690.png)

**MQTT匿名访问**  
有一些MQTT的服务端软件默认是开启匿名访问，如果管理员没有网络安全意识或懒惰，只要对公网开放，任何人都可以直接访问。

使用mqtt-pwn的connect命令进行连接。`connect -h` 显示帮助信息，其他命令也是如此，使用时，多看帮助和文档，很快就可以熟悉使用。  
对于开启匿名的服务，直接`connect -o host`
即可，当然该命令也支持输入用户名和密码。如果没有显示连接异常，就表示连接成功。连接成功后，可使用`system_info` 查看系统信息。

![](https://p4.ssl.qhimg.com/t01287faaad96ebfc43.png)

接下来就可以查看topic信息等内容。这时先执行`discovery`，等待显示scan #1 has finished，接下来执行`scans -i
序号`，在执行`topics`命令即可看到topic信息。
其中`disconvery`可以使用`-t`参数设置超时时间。`topics`命令可以使用`-l`参数设置查看条数。

![](https://p5.ssl.qhimg.com/t0178206285c0ae4785.png)

可以输入`messages`查看topic的内容。使用`-l`限制条数，`-i`参数查看某个单挑消息内容等。  
![](https://p3.ssl.qhimg.com/t0160bb2c89e37fa7c6.png)

![](https://p5.ssl.qhimg.com/t016e010021b6700142.png)

**MQTT用户名密码爆破**  
metasploit带有MQTT的爆破模块，经过实际测试，效果并不理想。这里仍然以mqtt-pwn来进行介绍。  
mqtt-pwn具有bruteforce功能，并带了一个简单的字典，可以爆破MQTT的用户名和密码。

    
    
    bruteforce --host host --port -uf user_dic -pf pass_dic 
    

端口默认是1883，用户和密码字典默认会在mqtt-pwn的`resources/wordlists` 文件夹下。

例如执行`bruteforce --host
127.0.0.1`爆破。爆破成功后就可以使用上面将到的内容进行连接进行操作，在连接时加上用户名和密码选项即可。

mqtt-pwn还支持更多功能，如[Owntracks (GPS Tracker)](https://mqtt-
pwn.readthedocs.io/en/latest/plugins/owntracks.html)、[Sonoff
Exploiter](https://mqtt-
pwn.readthedocs.io/en/latest/plugins/sonoff.html)等。感兴趣的大家自己去看下文档去进行测试。

**应用中发现**  
在实际的使用场景我们可以通过中间人劫持从流量中捕获验证信息。以下为wireshark抓包内容。  
![](https://p5.ssl.qhimg.com/t014090f26a582af519.png)

除此之外，由于目前多种语言实现了mqtt的客户端，web应用中还有webscoket的mqtt。这使得可以通过web的网页源码或网络请求获得验证的信息。

![](https://p5.ssl.qhimg.com/t01c3af2e34139f7a84.png)

**服务端漏洞**  
这里列举一些历史上MQTT的漏洞供参考。

  1. [CVE-2017-7296](https://www.cvedetails.com/cve/CVE-2017-7296/)
  2. [CVE-2017-7650](https://mosquitto.org/blog/2017/05/security-advisory-cve-2017-7650/)
  3. [CVE-2018-17614](https://nvd.nist.gov/vuln/detail/CVE-2018-17614)
  4. [CVE-2019-5432](https://hackerone.com/reports/541354)
  5. [CVE-2020-13849](https://nvd.nist.gov/vuln/detail/CVE-2020-13849)
  6. [Mosquitto漏洞列表](https://mosquitto.org/security/)

## 探测与发现

功能强大的nmap是支持MQTT协议的识别的，可以直接通过nmap进行识别MQTT协议。另外，除上面提到的默认端口外，有的管理员会修改默认端口，这时也可以尝试1884，8084，8884等临近端口以进行快速探测，或前面增加数字等作为组合，如果针对单个目标，则可以探测全部端口。如果进行大规模的扫描或者提升扫描效率，则可以使用masscan、zmap、RustScan等先进性端口扫描，在使用nmap进行协议识别即可。  
nmap举例命令如下：  
`sudo nmap -p1883,8083,8883 -sS -sV --version-intensity 9 -Pn --open
target_ip`

![](https://p3.ssl.qhimg.com/t018117aa19a2c9274f.png)

nmap也有相关的MQTT
lua脚本可以使用，其MQTT版本为3.1.1。脚本地址为<https://svn.nmap.org/nmap/nselib/mqtt.lua>。

如果想要自己编写代码去进行探测，只需要根据MQTT的协议标准，通过socket即可进行收发报文。关于MQTT协议的详细内容可以查看文档，地址为<https://docs.oasis-
open.org/mqtt/mqtt/>

现有的网络空间测绘平台基本都实现了对MQTT进行探测。可直接通过这些搜索引擎获取大量对外使用MQTT协议的服务。

  * 知风  
在针对IoT和ICS探测的搜索引擎知风中搜索，直接搜索`mqtt`关键字，可以发现15万个对外开放的服务。  
![](https://p1.ssl.qhimg.com/t01e5819f5a7da4e34c.png)

  * fofa  
搜索关键字为`protocol=mqtt`，一年内有25万个对外开放。

![](https://p2.ssl.qhimg.com/t0137efdf7d38a450f0.png)

  * shodan  
搜索关键字：  
product:"MQTT"  
product:"Mosquitto"

shodan上搜索后共有超过11万个对外开放。

![](https://p5.ssl.qhimg.com/t0146755084862cf215.png)

通过以上的搜索结果，各引擎各有优劣。shodan和知风针对该协议的探测均会列出topic；而fofa从发现数量上最多，但是仅识别了协议，并未列出topic；除此之外知风系统的地理位置定位精度较高，可以定位百米范围内。

## mqtt 安全建议

  1. 请勿启用匿名访问，对服务端（BROKER）设置认证，增加用户名密码验证。
  2. 根据实际情况，优先使用加密传输数据，防止中间人攻击。
  3. 加密payload后在进行安全传输。
  4. 使用最新的服务端程序架设服务。
  5. 不要将实现的代码上传到github等代码公开平台。

## 小结

写这篇文章时，网络上关于MQTT安全的文章并不多，但是通过对其了解，仍然有不少内容可以探索，比如在工业上有MQTT网关，以及众多支持MQTT的服务端软件、加上广泛的应用场景。本文简单介绍MQTT安全的内容，还有更多的内容等待探索。感兴趣的朋友也欢迎大家多多交流讨论。

最后，提醒一下大家，在学习和研究过程中自己搭建服务进行学习。请勿对网络上的目标进行测试、破坏等活动。

## 引用

https://dzone.com/articles/exploiting-mqtt-using-lua  
https://www.hindawi.com/journals/wcmc/2018/8261746/  
https://github.com/akamai-threat-research/mqtt-pwn  
https://morphuslabs.com/hacking-the-iot-with-mqtt-8edaf0d07b9b  
https://book.hacktricks.xyz/pentesting/1883-pentesting-mqtt-mosquitto  
https://hackmd.io/@QwmL8PAwTx-bYDnry-ONpA/H1nm2tHzb?type=view  
https://ttm4175.iik.ntnu.no/prep-iot-mqtt.html  
https://mobilebit.wordpress.com/tag/mqtt/  
https://www.hivemq.com/blog/seven-best-mqtt-client-tools/  
https://nmap.org/nsedoc/lib/mqtt.html  
http://mqtt.p2hp.com/

  * **本文作者：** BaCde 
  * **本文链接：** <https://bacde.me/post/mqtt-security-part-one/>
  * **版权声明：** 本博客所有文章除特别声明外，均采用[ __BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。转载请注明出处！ 

[# automotive](https://bacde.me/tag/prvJjB7Fua5/) [#
mqtt](https://bacde.me/tag/cu9Tg0uRbMd/) [# car
hacking](https://bacde.me/tag/JdYDio18R72/) [# mqtt-
pwn](https://bacde.me/tag/tCcj-UHo7-S/) [#
car](https://bacde.me/tag/NB7AfDzqlwN/)

__[自己的一些漏洞分析文章汇总【含0day】](https://bacde.me/post/my-some-vulnerability-alert/
"自己的一些漏洞分析文章汇总【含0day】") [上一篇](https://bacde.me/post/my-some-vulnerability-
alert/ "自己的一些漏洞分析文章汇总【含0day】")

[Hacking All The Cars之CAN总线逆向](https://bacde.me/post/hacking-all-the-cars-can-
bus-reverse/ "Hacking All The Cars之CAN总线逆向")
[下一篇](https://bacde.me/post/hacking-all-the-cars-can-bus-reverse/ "Hacking All
The Cars之CAN总线逆向") __

Powered by [Gridea](https://github.com/getgridea/gridea) | © 2019-2020 Theme
By [HsxyHao](https://github.com/hsxyhao/gridea-theme-next)

Powered by [BaCde](https://www.bacde.me)

__ 0%

