#  【紧跟时事】Apache Solr 远程代码执行漏洞

原创 浪飒 [ 浪飒sec ](javascript:void\(0\);)

**浪飒sec** ![]()

微信号 langsasec

功能介绍 浪飒，共建网络安全

____

___发表于_

收录于合集

#漏洞通告 2 个

#Solr 1 个

## 免责声明

> 本公众号所发布的文章及工具只限交流学习，本公众号不承担任何责任！如有侵权，请告知我们立即删除。

收不到推送的小伙伴，记得 **星标** 公众号哦

## 废话

大哥刚发的 [漏洞通告 | Apache Solr
远程代码执行漏洞](https://mp.weixin.qq.com/s?__biz=Mzg5MTc3ODY4Mw==&mid=2247503360&idx=1&sn=29f1738205f8ce6e4c05b868e6596691&scene=21#wechat_redirect)

  

![]()

微步大哥说上传恶意配置文件进行攻击

![]()

我来猜下大致上传路径：

## FOFA语法

> app="APACHE-Solr" && title=="Solr Admin"

## 开始

首先从大哥的提示可以知道，利用路径大概就在`/api/schema-
designer/*`路径下，并且solr本身可以默认未授权访问的，还有版本大概在8.10-9.3

![]()

找到一个资产如下，未授权访问：

![]()

访问`api/schema-designer`路径

![]()

这是一个JSON格式的响应数据，其中包含了可用的子路径及其对应的请求方法。根据提供的数据，以下是可用的子路径及其对应的请求方法：

  * GET /schema-designer//configs：获取配置信息

  * POST /schema-designer//file：上传文件

  * GET /schema-designer//file：获取文件

  * GET /schema-designer//diff：获取差异

  * GET /schema-designer//collectionsForConfig：获取配置的集合信息

  * GET /schema-designer//sample：获取示例数据

  * GET /schema-designer//download/*：下载指定路径的文件

  * GET /schema-designer//info：获取信息

  * GET /schema-designer//query：查询数据

  * POST /schema-designer//add：添加数据

  * POST /schema-designer//prep：准备数据

  * POST /schema-designer//analyze：分析数据

  * PUT /schema-designer//cleanup：清理数据

  * PUT /schema-designer//update：更新数据

  * PUT /schema-designer//publish：发布数据

根据通告可以发现需要创建Schema Designer,利用该功能才能上传文件

找到这个功能：

![]()

  

访问后下图可知，这里可以分别创建新的Schema和上传文件。

![]()

新建Schema抓包，如下图，其中test即为新建Schema的名称

![]()

请求如下：

  *   *   *   *   *   *   *   *   *   * 

    
    
     POST /api/schema-designer/prep?_=1698050271144&configSet=test&copyFrom=_default&wt=json HTTP/1.1 Host:  Content-Length: 0 Accept: application/json, text/plain, */* X-Requested-With: XMLHttpRequest User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36 Content-Type: application/json;charset=utf-8 Accept-Encoding: gzip, deflate Accept-Language: zh-CN,zh;q=0.9 Connection: close

选择文件并点击分析上传，这里我选了个图片：

图中提示图片不允许

![]()

改了后缀还是图片不允许

![]()

我直接把Content Type删掉，提示我说只允许CSV/TSV，XML，JSON

![]()

下面我把Content Type改成json的形式，出现了JSON参数错误

![]()

  

但这个我觉得应该就是利用的地方，微步大哥利用时的报错和我数据包的末尾几乎相似，我猜可能是在分析的同时执行了上传文件内的恶意代码。

![]()

![]()

那如果没猜错的话

这个上传的数据包就可以是如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /api/schema-designer/analyze?_=1698050271144&configSet=test&languages=*&wt=json HTTP/1.1 Host:  Content-Length: 218300 Accept: application/json, text/plain, */* X-Requested-With: XMLHttpRequest User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36 Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryzGHoYcY5St7YFia9 Accept-Encoding: gzip, deflate Accept-Language: zh-CN,zh;q=0.9 Connection: close  ------WebKitFormBoundaryzGHoYcY5St7YFia9 Content-Disposition: form-data; name="file"; filename="solr.in或者solr.sh 根据系统" Content-Type: application/json  这里是恶意代码 ------WebKitFormBoundaryzGHoYcY5St7YFia9--

最终的利用应该就是先发送新建Schema的数据包，再发送上传的数据包

## 总结

我瞎说的，我其实根本不懂，别骂我，要是有真POC告诉我咋利用再骂我

![]()

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

