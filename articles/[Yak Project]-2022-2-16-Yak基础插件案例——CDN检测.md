#  Yak基础插件案例——CDN检测

原创 奶权  [ Yak Project ](javascript:void\(0\);)

**Yak Project** ![]()

微信号 YakLanguage

功能介绍 Yak Language Project: <del>北半球</del>最强安全研发语言 / 黑客语言

____

__

收录于话题

#cdn 2 个

#插件 12 个

#编程 15 个

  

      解释一下上次文章的招聘要求（写漏了一点）。安全类的至少符合两项即可（不是很符合，对我们感兴趣也可以），设计师也不一定非要艺术相关专业，简单来说就是我们招这些岗位，条件都不是硬性要求，你觉得你可以，对我们也感兴趣就可以来聊聊~~

  

01

  

关于CDN

  

01

介绍

  

要谈CDN，就得先从CDN以及CDN的配置先说起。  

> 内容分发网络（英语：Content Delivery Network或Content Distribution
> Network，缩写：CDN）是指一种透过互联网互相连接的电脑网络系统，利用最靠近每位用户的服务器，更快、更可靠地将音乐、图片、视频、应用程序及其他文件发送给用户，来提供高性能、可扩展性及低成本的网络内容传递给用户。——来自维基百科

  

02

优点

  

      CDN的总承载量取决于其网络节点的数量决定，通常会比单一骨干最大的带宽还要大。这使得CDN可以承载的用户数量比起传统的单一服务器多。假设现在把一台有`100Gbps`处理能力的服务器放在只有`10Gbps`带宽的数据中心，那么这时候带宽就成为了架构上了瓶颈。但是如果放到十个有`10Gbps`的地点，整个系统的处理能力就可以达到`10*10Gbps`。同时，将服务器放到不同地点还有其他好处，例如：异地备援、隐藏真实IP等。

  

03

配置

  

配置CDN一般有两种方式：

#### 第一种

CDN厂商提供一个域名，给自己需要接入CDN的域名添加一个`cname`，指向CDN厂商提供的域名。这种域名一般会有个随机前缀。

#### 第二种

把需要接入CDN的域名的`NS`记录指向CDN厂商的DNS服务器IP。

  

02

  

检测CDN

  

知道了配置原理后，我们可以很容易得到几种检测的思路。

01

CNAME指纹

  

       配置中说的第一种方法是去配置一个`CNAME`记录为CDN厂商提供的域名，这个域名通常会有一个前缀，后缀一般都是固定的。找了两个加载js库的CDN加速服务，看一下他们的`CNAME`记录。 

![](https://gitee.com/fuli009/images/raw/master/public/20220216143807.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220216143809.png)

       可以看到不同的CDN厂商的域名一般会有个共同点，就是带有cdn/dns等字眼（不一定），这就相当于CDN厂商的一个指纹。我们只需要维护一个常见CDN厂商的`CNAME`指纹字典，可以去里面查目标的CNAME记录是否为某个CDN厂商的域名。但是这种方式的缺点十分明显，就是维护指纹的成本很高，如果某个CDN厂商加了新的域名，那就需要重新添加指纹了。  

02

IP段

     我们都知道CDN厂商一般会有很多个节点，而这些节点一般是是在一些IP段里面。所以我们可以维护一个常见CDN厂商的IP段列表。 

![](https://gitee.com/fuli009/images/raw/master/public/20220216143810.png)

  

> https://github.com/al0ne/Vxscan/blob/master/lib/iscdn.py

如果目标域名的`A`记录在IP段列表中，那么我们可以暂时认为其接入了CDN。但是这样做也是明显有跟上面一样的缺点的，CDN厂商添加节点的行为会使得我们的脚本出现误报/漏报。

  

03

ASN号  

  

ASN介绍

> 自治系统或自治域（英文：Autonomous system,
> AS）是指在互联网中，一个或多个实体管辖下的所有IP网络和路由器的组合，它们对互联网执行共同的路由策略。——来自维基百科

       我们可以整理出常见CDN厂商的ASN号列表，如果目标域名`A`记录IP的ASN号在列表中，那么我们也可以暂时认为其接入了CDN。 

![](https://gitee.com/fuli009/images/raw/master/public/20220216143811.png)

      但是还是有明显的误报/漏报问题，因为谁也不知道会不会突然冒出一个新的CDN厂商被我们遇到，或者某些家大业大的大厂自己实现了CDN的接入，没有使用CDN厂商的服务。

  

04

多地区ping

  

       上面介绍的三种方式的优点就是速度快，需要检测的域名非常多时优势很大。基本就是DNS查一下然后进行各种类型指纹的匹配就行，但缺点就是误报/漏报率高。想要降低误报/漏报率可以将以上的方式做一下结合，例如精灵师傅的`OneForAll`子域名工具就结合了上面提到的方式。但是再怎么样也只是能将误报/漏报率降到最低，达不到零误报/漏报。

       前面介绍到CDN会根据地区返回不同的IP。那么我们可以准备很多个地区的服务器，同时对该域名执行ping操作。看看返回的IP是否相同，达到判断是否接入CDN的目的。

      但是这样成本非常高，好在网上有现成的服务可以使用。例如：站长之家多地区ping

![](https://gitee.com/fuli009/images/raw/master/public/20220216143812.png)

      但是在官网使用该服务一次只会请求一部分监测点进行ping操作，效率不高。所以我打算使用`yaklang`利用其探测点，重新写一个并发的版本。

  

03

  

插件编写

  

其实这个插件本质上是一个“爬虫”，我们需要去分析一下站长之家的请求流程。

大概流程如下：首先会发送一个`POST`请求到`https://ping.chinaz.com/`，获取监测点的`guid`，以及`enkey`、`checkType`等参数，作为后续请求的参数。

![]()

  

 接着带着上一步获取到的参数发送一个`POST`请求到`https://ping.chinaz.com/iframe.ashx?t=ping`

![](https://gitee.com/fuli009/images/raw/master/public/20220216143813.png)

 如果成功则返回格式如下  

    
    
    {state:1,msg:'',result:{ip:'36.152.44.96',ipaddress:'中国江苏南京 移动',responsetime:'13毫秒',ttl:'51',bytes:'32'}}  
    

反之则返回格式如下

    
    
    {state:0,msg:''}  
    

将其中需要的信息提取出来就行。

  

01

编写获取初始化参数的函数

  

       经过测试，其实后续的回调接口只需要获取`enkey`与`checkType`以及监测点的`guid`，所以我们发送对应的`POST`请求后使用正则表达式提取出需要的参数即可。（在写完插件的第二天V1师傅加了个`Xpath`库，可以更容易提取出参数了，所以这里最优解是用`Xpath`库。）
    
    
    // 字典转url query格式，带urlencode  
    dict2UrlQueryWithUrlEncode := func(d) {  
        s := make([]string)  
        for k, v := range d {  
            s = append(s, sprintf("%v=%v", k, codec.EscapeQueryUrl(v)))  
        }  
        return str.Join(s, "&")  
    }  
      
    // 获取初始化配置  
    getInitConfig := func() {  
        datas := dict2UrlQueryWithUrlEncode({  
            "host": "example.com",  
            "linetype": "电信,多线,联通,移动,其他",  
        })  
        // 请求接口获取配置  
        res, err := http.Request("POST", "https://ping.chinaz.com/", http.body(datas), http.header("Content-Type", "application/x-www-form-urlencoded"))  
        die(err)  
        resBody := string(http.GetAllBody(res))  
      
        // <div id="0e519c9d-dab8-480c-a372-c72480dd133a" class="row listw tc clearfix" linetype="1" state="0" trycount="0">  
                // <div class="col-2" name="city" serveruroup="0" data-company="[网锐]微端BGP200M/1200/月,www.wridc.com/hd.html">江苏宿迁[电信]</div>  
        // 获取监测点UUID  
        r1, _ := re.Compile(`<div id="([0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12})" class="row listw tc clearfix"[\s\S]+?<div class="col-2" name="city"[\s\S]+?>(\S+)</div>`)  
        result := r1.FindAllStringSubmatch(resBody, -1)  
        pingServerInfoList := make([]map[string]string)  
        for _, i := range result {  
            pingServerInfoList = append(pingServerInfoList, {  
                "name": i[2],  
                "guid": i[1],  
            })  
        }  
      
        // <input type="hidden" id="enkey" value="OT5JUx9bX5INGvYSBT087i8pZeO7y9et" />  
        // 获取enkey参数  
        r2, _ := re.Compile(`<input type="hidden" id="enkey" value="(\S+)" />`)  
        enkey := r2.FindStringSubmatch(resBody)  
        if len(enkey) == 0 {  
            die("get enkey failure")  
        }  
        enkey := enkey[1]  
      
        // <input type="hidden" id="checktype" value="0" />  
        // 获取checkType参数  
        r3, _ := re.Compile(`<input type="hidden" id="checktype" value="(\d+?)" />`)  
        checkType := r3.FindStringSubmatch(resBody)  
        if len(checkType) == 0 {  
            die("get checkType failure")  
        }  
        checkType := checkType[1]  
      
        return pingServerInfoList, enkey, checkType  
    }

  

02

编写执行监测点ping操作函数

  

       这里需要注意的点是，目标返回的是一个`JSONP`格式的数据，往`JSONP`回调函数里面丢的是一个js的对象，不是一个标准的`JSON`格式，所以需要对其进行一些字符串操作，使其能被转为一个`map[string]var`格式的数据。且因为我们需要启动`goroutine`进行并发操作，所以我是使用了一个`Channel`来在多个`goroutine`中安全地操作数据。
    
    
    // 字典转url query格式，带urlencode  
    dict2UrlQueryWithUrlEncode := func(d) {  
        s := make([]string)  
        for k, v := range d {  
            s = append(s, sprintf("%v=%v", k, codec.EscapeQueryUrl(v)))  
        }  
        return str.Join(s, "&")  
    }  
    // 转为合法json，并反序列化  
    convertLegalJson := func(s) {  
        // 去掉括号  
        s = str.ReplaceAll(s, "({", "{")  
        s = str.ReplaceAll(s, "})", "}")  
          
        // 加引号，改单引号  
        s = str.ReplaceAll(s, `state:`, `"state":`)  
    	s = str.ReplaceAll(s, `msg:`, `"msg":`)  
    	s = str.ReplaceAll(s, `result:`, `"result":`)  
    	s = str.ReplaceAll(s, `ip:`, `"ip":`)  
    	s = str.ReplaceAll(s, `ipaddress:`, `"ipaddress":`)  
    	s = str.ReplaceAll(s, `responsetime:`, `"responsetime":`)  
    	s = str.ReplaceAll(s, `ttl:`, `"ttl":`)  
    	s = str.ReplaceAll(s, `bytes:`, `"bytes":`)  
    	s = str.ReplaceAll(s, `'`, `"`)  
      
        // 反序列化  
        d, err := json.New(s)  
        die(err)  
        return d.Value()  
    }  
    // 让监测点开始ping操作  
    ping := func(serverInfo, enkey, checkType, target, results) {  
        datas := dict2UrlQueryWithUrlEncode({  
            "guid": serverInfo["guid"],  
            "host": target,  
            "ishost": "0",  
            "isipv6": "0",  
            "encode": enkey,  
            "checktype": checkType,  
        })  
        res, err := http.Request("POST", "https://ping.chinaz.com/iframe.ashx?t=ping", http.body(datas), http.header("Content-Type", "application/x-www-form-urlencoded"), http.timeout(20))  
        // 请求失败也返回ping失败的结果  
        if err != nil {  
            results <- {"state": 0, "msg": ""}  
            return  
        }  
        resBody := string(http.GetAllBody(res))  
        // 失败 ({state:0,msg:''})  
        // 成功 ({state:1,msg:'',result:{ip:'110.242.68.4',ipaddress:'中国河北保定顺平县 联通',responsetime:'20毫秒',ttl:'52',bytes:'32'}})  
      
        // 处理结果  
        pingInfo := convertLegalJson(resBody)  
        pingInfo["cityname"] = serverInfo["name"]  
        if pingInfo["state"] == float64(1) {  
            if str.Contains(pingInfo["result"]["responsetime"], "超时") {  
                pingInfo["result"]["responsetime"] = "超时"  
            }  
            if str.Contains(pingInfo["result"]["ttl"], "超时") {  
                pingInfo["result"]["ttl"] = "超时"  
            }  
            if pingInfo["result"]["bytes"] == "" {  
                pingInfo["result"]["bytes"] = "-"  
            }  
            pingInfo["result"]["ipaddress"] = str.Join(str.Fields(pingInfo["result"]["ipaddress"]), " ")  
        }  
        results <- pingInfo  
    }  
    

  

03

编写监测逻辑与图形化输出

  

 **第一步**  

初始化与`yakit`的连接，并解析外部参数（即需要检测的域名）。因为使用了`str.ParseStringToHosts()`，所以`target`参数可以使用`,`进行分割以支持多个目标。

    
    
    yakit.AutoInitYakit()  
      
    // 解析参数  
    targets := cli.String("target", cli.setRequired(true))  
    targetList := str.ParseStringToHosts(targets)  
    

#### 第二步

调用`func getInitConfig()`拿到所有需要的参数

    
    
    pingServerInfoList, enkey, checkType := getInitConfig()  
    serverNum := len(pingServerInfoList)  
    printf("初始化成功，共获取到%v个监测点\n", serverNum)  
    

#### 第三步

遍历`targetList`拿到每一个`target`，初始化一个当前`target`的表格，并声明一个用于收集结果的`Channel`。再遍历探测点信息`pingServerInfoList`，启动`goroutine`并发执行`func
ping()`。

    
    
    for targetIndex, target := range targetList {  
        yakit.EnableTable(target, ["监测点", "响应IP", "IP归属地", "响应时间", "TTL", "数据包大小"])  
        results := make(chan var)  
      
        // https://www.yaklang.io/docs/newforyak/concurrent  
        submitTask := func(param...) {  
            go ping(param...)  
        }  
        for _, serverInfo := range pingServerInfoList {  
            submitTask(serverInfo, enkey, checkType, target, results)  
        }  
    }  
    

       这里还有一个关于在循环中`goroutine`启动时定义域的一个坑点。需要使用一个`trick`去化解。即我们在循环中不直接启动`goroutine`，而是在循环中调用一个同步函数，在该函数中再开启`goroutine`执行异步任务。具体可以看官网这个链接：https://www.yaklang.io/docs/newforyak/concurrent

#### 第四步

从通道中拿到结果，将结果做一系列处理（如统计返回IP数量、进度条、表格等）和最重要的CDN判断后用`yakit`库进行输出。

    
    
    for targetIndex, target := range targetList {  
        yakit.EnableTable(target, ["监测点", "响应IP", "IP归属地", "响应时间", "TTL", "数据包大小"])  
        results := make(chan var)  
      
        // https://www.yaklang.io/docs/newforyak/concurrent  
        submitTask := func(param...) {  
            go ping(param...)  
        }  
        for _, serverInfo := range pingServerInfoList {  
            submitTask(serverInfo, enkey, checkType, target, results)  
        }  
        // println(<- results)  
        ips := make(map[string]int)  
        for i := 0; i < serverNum; i++ {  
            data := <- results  
         // dump(data)  
      
            // 总进度条  
            // yakit.SetProgress(( float64(i+1)*(float64(targetIndex+1)/float64(len(targetList))) )/float64(serverNum))  
            yakit.SetProgress( float64(i+1)*float64(targetIndex+1) / (float64(len(targetList)) * float64(serverNum)) )  
            // 子任务进度条  
            yakit.SetProgressEx(target, float64(i+1)/float64(serverNum))  
      
            // 统计结果、处理表格输出  
            if data["state"] == float64(1) {  
                if data["result"]["ip"] != "" {  
                    if ips[data["result"]["ip"]] == undefined {  
                        ips[data["result"]["ip"]] = 1  
                    }else {  
                        ips[data["result"]["ip"]]++  
                    }  
                    // 成功的探测点输出表格  
                    tableData := make(map[string]var)  
                    tableData["监测点"] = data["cityname"]  
                    tableData["响应IP"] = data["result"]["ip"]  
                    tableData["IP归属地"] = data["result"]["ipaddress"]  
                    tableData["响应时间"] = data["result"]["responsetime"]  
                    tableData["TTL"] = data["result"]["ttl"]  
                    tableData["数据包大小"] = data["result"]["bytes"]  
                    yakit.Output(yakit.TableData(target, tableData))  
                }  
            }  
            // printf("\r正在执行ping操作，当前：%v/%v个，成功：%v/%v个，失败：%v/%v个，总进度：%.2f%%", i+1, serverNum, success, serverNum, failure, serverNum, 100*(float64(i+1)/float64(serverNum)))  
        }  
        println()  
        if len(ips) > 1 {  
            yakit.StatusCard(sprintf("%v:IS CDN", target), "是", target)  
            yakit.StatusCard(sprintf("%v:IP Number", target), len(ips), target)  
            // println("监测点返回不同IP，可能存在CDN")  
        }else {  
            for ip := range ips {  
                yakit.StatusCard(sprintf("%v:IS CDN", target), "否", target)  
                yakit.StatusCard(sprintf("%v:IP Address", target), ip, target)  
                // printf("所有监测点返回IP一致，为%v\n", ip)  
            }  
        }  
        // break  
    }  
    

 **最终插件效果  **

![](https://gitee.com/fuli009/images/raw/master/public/20220216143814.png)

（其实还可以再给每个域名都分配一个`goroutine`，让每个目标间的监测也是异步进行的。但是考虑到可能会对接口产生较大的压力，所以就没这样做了。

  

04

  

插件地址

  

如果大家对这个CDN判断插件感兴趣的话，可以直接在插件仓库中导入米斯特的第三方`yakit-
store`插件库。地址为：https://github.com/Acmesec/yakit-store

更新后点击头部的刷新按钮

![](https://gitee.com/fuli009/images/raw/master/public/20220216143816.png)

 就可以在插件仓库或者基础安全工具中看到插件啦！

![](https://gitee.com/fuli009/images/raw/master/public/20220216143817.png)

  

05

  

END  

  

感谢奶权的投稿~~我们的投稿活动还在进行中，走过路过不要错过，参加即可获得精美小礼物，投稿还有稿费噢~进交流群和投稿添加以下微信即可。

投稿活动详情（[Yakit联合HackingClub开启有奖征稿啦~](http://mp.weixin.qq.com/s?__biz=MzAxOTAzOTU3Mw==&mid=2247484212&idx=1&sn=a391d8bbeaecd0baff71accad6b3bada&chksm=9bcc570eacbbde18637049dc30ea0ad5f3f1fce062e00b19254faf6de686bdb54c4a38362ac7&scene=21#wechat_redirect)）

官网教程：https://www.yaklang.io/products/intro

下载地址：https://github.com/yaklang/yakit

![](https://gitee.com/fuli009/images/raw/master/public/20220216143818.png)

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

Yak基础插件案例——CDN检测

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

