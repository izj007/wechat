#  插件分享 | 字符串编码解码及转换拼音字典的 Xcode

原创 ettercap  [ GobySec ](javascript:void\(0\);)

**GobySec** ![]()

微信号 gobysec

功能介绍
新一代网络安全测试工具，由赵武Zwell（Pangolin、FOFA作者）打造，能够针对一个目标企业梳理最全的攻击面信息，同时能进行高效、实战化漏洞扫描，并快速的从一个验证入口点，切换到横向。

____

__

收录于话题 #插件 6个内容

![](https://gitee.com/fuli009/images/raw/master/public/20210915191223.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210915191224.png)

  

Goby社区第18 篇插件分享文章

全文共：4590 字   预计阅读时间：12 分钟

  

 **前言：** Goby
自从出了红队专版以来，就一直想体验一下他的强大，获取红队专版的途径很简单，提交插件并审核通过即可申请，因此花了一天时间研究一下插件的编写。为了能够快速的编写插件，只需要将已有的一些网站功能移植过来即可，非常简单。以下就以移植的这款
Xcode 信息编码工具为例，插件功能非常简单,主要用于学习移植插件的思路，起到抛砖引玉的作用。

  

  

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210915191225.png)
**01**

 **  插件使用** **1.1
插件效果**![](https://gitee.com/fuli009/images/raw/master/public/20210915191226.png)

 **1.2 使用方法**

首先介绍一下插件的使用方法

1\. 通过扫描弹窗页或工具栏打开该插件

![](https://gitee.com/fuli009/images/raw/master/public/20210915191228.png)

2\. 输入需要进行转码的字符串，选择相应的功能即可

![](https://gitee.com/fuli009/images/raw/master/public/20210915191229.png)

3\. 输入中文姓名，使用中文姓名转拼音功能转换成拼音字典

![](https://gitee.com/fuli009/images/raw/master/public/20210915191230.png)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210915191225.png)
**02**

 **  插件开发**

###  **2.1 选择现有插件**

谷歌浏览器中有一款很常用的插件 - FeHelper，这是一个多功能的工具集，其信息编码模块是我们在进行 Web 渗透时经常使用的，那么我们就选择将它移植到
Goby 中来。

从零开始编写是比较耗时的，幸好该项目是开源的，因此我们能很容易获取该插件的源代码并将它集成进去，这里以信息编码转换模块为例

![](https://gitee.com/fuli009/images/raw/master/public/20210915191232.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210915191233.png)

FeHelper 插件有很多模块，因此我们需要提取一下所需要的信息编码转换模块的代码

![](https://gitee.com/fuli009/images/raw/master/public/20210915191234.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210915191235.png)

  *   *   *   *   *   * 

    
    
        endecode-lib.js       依赖库    index.css       样式文件    index.html             插件UI    index.js              功能源代码    md5.js                依赖库    pako.js               依赖库

###  **2.2 分析插件结构**

首先我们下载官方的 FOFA 插件包，分析一下基础结构

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    │   .gitignore│   CHANGELOG.md                       对应插件商城更新日志，需要使用英文│   package.json              插件基础信息,指定插件名称、作者、版本信息，插件打开位置等,需要使用全英文│   README.md               对应插件商城的详情页面，同样需要使用英文描述，可以编写插件使用说明等└───src                  插件源码目录    │   extension.js           插件源代码    │   fofa.html             插件UI文件    └───assets              资源文件目录        ├───css              插件UI样式文件目录        │       base.css        ├───font            插件字体样式目录，一般不需要更改        │       iconfont.css        ├───img              图片目录，一般放图标文件等        │       fofa.png        ├───js              存放外部需要调用的js库文件等        │       jquery-3.3.1.min.js        │       jquery.base64.js        │       jquery.i18next.js         中英文翻译js文件        │       translate.js           中英文翻译js文件        ├───lib              layui文件，一般不需要更改        │   └───layui        └───translate          对应各类语言翻译所需文件目录            ├───CN            │       CHANGELOG.md    中文版更新日志            │       html.json      UI文件中文对照            │       README.md      中文版中文详情            │       translate.json    对应中文版package.json            └───EN                    html.json            UI文件英文对照

从以上分析可以看出，插件结构非常简单，与谷歌插件的文件结构非常相似，因此开发（移植）一款插件只需要往对应结构的文件里补充响应信息即可，非常简单。接下来我们按照以上方法来演示一下

###  **2.3 补充结构基础信息**

1\. 首先根据需要补充 package.json 文件信息，填写插件名称、作者、描述信息，根据图示填写即可，这里在 views 中的 command
字段会在 extension.js 文件中被调用

![](https://gitee.com/fuli009/images/raw/master/public/20210915191236.png)

2\. 在 src 目录下创建 index.html、extension.js 两个文件即可，同时创建 assets
目录，用来存放额外的资源，譬如图标，样式文件，需要调用的 js 库文件等。

![](https://gitee.com/fuli009/images/raw/master/public/20210915191237.png)

3\. 编辑 extension.js 文件，最基础的文件内容根据图示编写即可，忽略打码的文字，打码的文字仅为中英文翻译所需，根据自己的需求选择是否保留。

![](https://gitee.com/fuli009/images/raw/master/public/20210915191238.png)

###  **2.4 复制代码**

###  根据 2.3 创建好基础文件之后，可以正式开始复制代码了，首先将谷歌插件 FeHelper 中的 apps\en-decode 目录下的
index.html 复制过来覆盖掉 index.html，同时分别将 index.css、index.js 复制到 assets 目录下的 css 及
js 文件夹，修改好 index.html 中 css 的相对路径，基础工作都这里就完成了。

根据图中指向分别放到不同目录即可，此时可以重启 Goby，查看一下效果。

![](https://gitee.com/fuli009/images/raw/master/public/20210915191239.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210915191240.png)

大致框架已经有了，但是样式好像还有问题，这时候我们打开 Debug 工具，看一下缺少了哪些样式。

![](https://gitee.com/fuli009/images/raw/master/public/20210915191241.png)

通过 Debug 工具的网络模块，我们发现有几个请求是 404 状态，说明还有几个文件的相对路径有错误，修改一下即可。

![](https://gitee.com/fuli009/images/raw/master/public/20210915191242.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210915191243.png)

修改好其他文件的相对路径之后，重启 Goby 查看一下效果。

![](https://gitee.com/fuli009/images/raw/master/public/20210915191245.png)

已经没有再继续报错的请求了，测试一下功能都很完整，到这里，插件已经基本完成，只需要进行中英文翻译等工作就可以打包发布插件了。

![]()

###  **2.5 中英文翻译**

升级到新版之后，如果要成功的通过插件的审核，需要进行中英文的翻译，这个步骤也非常简单，按照以下步骤进行即可。

1\. 在 package.json 中加入 language，其他语言同理。

![](https://gitee.com/fuli009/images/raw/master/public/20210915191246.png)

2\. 修改 extension.js 文件，添加以下代码，然后将 registerCommand 处的 title 换成 let title =
getTranslate('XcodeTest') 即可。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
      let translate  = require(__dirname+'/assets/js/translate.js');  goby.bindEvent('onChangeLang',()=>{    let iframes = Array.from(document.querySelectorAll('#iframe-dia iframe'));    let iframe = iframes.find((iframe)=>{      return iframe.contentWindow.goby.id == goby.id;    })    iframe && changeLang(iframe);  })  
      function changeLang(iframe){    let title = getTranslate('XcodeTest')    goby.showIframeDia(iframe.getAttribute('src'),title , "666", "500");  }    function getTranslate(key) {    let lang = goby.getLang();    try {      let content = eval("translate[lang][key]");      if(content == undefined){        try {          let content = eval("translate['EN'][key]");          if(content == undefined){            return key;          }else{            return content;          }        } catch (error) {          console.log(error);          return key;        }      }else{        return content;      }    } catch (error) {      try {        let content = eval("translate['EN'][key]");        if(content == undefined){          return key;        }else{          return content;        }      } catch (error) {        console.log(error);        return key;      }    }  }

![](https://gitee.com/fuli009/images/raw/master/public/20210915191248.png)

3\. 修改 index.html 文件中的中文提示。

首先在 index.html 的 body 中引入以下 js 及代码,直接复制进去即可，响应的 js 文件从  FOFA 查询插件中复制即可。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
      <script src="assets/js/jquery-3.3.1.min.js"></script>  <script src="assets/js/jquery.i18next.js"></script>  <script src="assets/js/jquery.base64.js"></script>  <script src="assets/lib/layui/layui.all.js"></script>  <script>  
        let goby = parent.goby; // 获取GobyAPi    let fs = parent.require('fs');    let results = []; // 定义搜索结果数组    let net = parent.require('net')    // 字符串转base64    function encode(str) {      var base64 = $.base64.encode(str);      return base64;    }  
        function lang() {      //获取当前Languzge      let language = goby.getLang();  
          //判断翻译文件是否存在      let translateState = fs.existsSync(goby.__dirname + '/assets/translate/' + language + '/html.json');  
          //翻译文件存在则使用翻译文件,否则使用默认EN翻译      let lang = translateState ? language : 'EN';  
          let a = $.i18n.init({        lng: language, //指定语言        useCookie: false,        resGetPath: './assets/translate/' + lang + '/html.json',//语言包的路径      }, function (err, t) {        if (!err) {          $('[data-i18n]').i18n(); // 通过选择器集体翻译          return;        }        goby.showErrorMessage(err)      });    }  
        lang();  
        //当lang改变时更新页面内容    goby.bindEvent('onChangeLang', () => {      lang();    })</script>

然后修改 index.html 相应位置的中文，为相应的 html 标签添加 data-i18n 属性，按照图示修改即可。

![](https://gitee.com/fuli009/images/raw/master/public/20210915191249.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210915191250.png)

将修改好的中英文提示以 json 的格式分别加入不同语言文件夹的 html.json 中即可。

中文格式：

![](https://gitee.com/fuli009/images/raw/master/public/20210915191251.png)

英文格式：

![](https://gitee.com/fuli009/images/raw/master/public/20210915191252.png)

最后重启 Goby 查看效果。

中文效果：

![](https://gitee.com/fuli009/images/raw/master/public/20210915191253.png)

英文效果：

![](https://gitee.com/fuli009/images/raw/master/public/20210915191254.png)

###  **2.6 打包插件**

将已经移植好的插件使用 winrar 等工具打包成 zip 格式，提交插件进行审核即可



![](https://gitee.com/fuli009/images/raw/master/public/20210915191225.png)
**03**

 **  总结**

第一次编写 Goby 插件，但是 Goby 官网提供的案例非常详细，并且所提供的 API
也非常丰富，每个人都可以很轻松的编写自己所需要的插件。最后，非常感谢 Goby 为我们提供这么强大且免费的神器。

  

  

> 插件开发文档及Goby开发版下载：  
>
>
> https://gobies.org/docs.html

  

 **关于插件开发在B站都有详细的教学，欢迎大家到弹幕区合影~**

  * https://www.bilibili.com/video/BV1u54y147PF/

  

  

  
![](https://gitee.com/fuli009/images/raw/master/public/20210915191256.png)

  

  
 **更多插件分享** **：**

* * *

[ • h1ei1 | 如何快速上手 Zookeeper
未授权漏洞](http://mp.weixin.qq.com/s?__biz=MzI4MzcwNTAzOQ==&mid=2247500596&idx=1&sn=5b6a054f6987ced13e10315cb276b119&chksm=eb842894dcf3a18218741d0758af985f83a95c078e06b77bade0c0fdc08660ea664d8350fecf&scene=21#wechat_redirect)  

[• 蜉蝣 |
可进行Web路径爆破的Dirsearch](http://mp.weixin.qq.com/s?__biz=MzI4MzcwNTAzOQ==&mid=2247500656&idx=1&sn=52271e3fb524219e56f7a179f0e0042c&chksm=eb8428d0dcf3a1c627812a7937452ad1fb9a0d714c30b4a7d1eb09a980f060100f08ada5e135&scene=21#wechat_redirect)

[• Eyes | 调用 Python 脚本进行漏洞测试的
PythonCall](http://mp.weixin.qq.com/s?__biz=MzI4MzcwNTAzOQ==&mid=2247505661&idx=1&sn=1f1637cc802111d030d310b20b14102f&chksm=eb843d5ddcf3b44b59cd1a3fcce8eb509313d8d7d6a72ca9fc6cc6b618884bd295ce8417fde0&scene=21#wechat_redirect)

[• socatos | 可一键统计各版本数量的 Windows
Count](http://mp.weixin.qq.com/s?__biz=MzI4MzcwNTAzOQ==&mid=2247505698&idx=1&sn=a6a44cb03fa965e1c231dc632ef8927a&chksm=eb843c82dcf3b594ac2d75f2f65dd6859e61dc37a87a7d77996f5b9623010d5a2baaa94899fd&scene=21#wechat_redirect)

[• hututuZh |
简单免杀绕过和利用上线的GoCS](http://mp.weixin.qq.com/s?__biz=MzI4MzcwNTAzOQ==&mid=2247507887&idx=1&sn=e96ce68003cb29820272bf20ee20963c&chksm=eb84340fdcf3bd19cd30d45ea1c84076cf853f750da1dc34058bac189cd6f0db13e3858565d7&scene=21#wechat_redirect)

更多 >>  插件分享  
  

如果表哥/表姐也想把自己上交给社区（获取红队专版）![](https://gitee.com/fuli009/images/raw/master/public/20210915191257.png)，戳这里领取一份插件任务？

> https://github.com/gobysec/GobyExtension/projects

  

![](https://gitee.com/fuli009/images/raw/master/public/20210915191258.png)

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

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

插件分享 | 字符串编码解码及转换拼音字典的 Xcode

最多200字，当前共字

__

发送中

写下你的留言

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

