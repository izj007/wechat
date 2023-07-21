#  vue站点渗透技巧及实战到getshell

[ SafetyTeam ](javascript:void\(0\);)

**SafetyTeam** ![]()

微信号 luck_sec

功能介绍 安全开发，漏洞研究，漏洞挖掘

____

___发表于_

收录于合集

编者荐语：

实战值得学习

以下文章来源于红蓝攻防实验室 ，作者不懂安全的开发

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4mPb9KQ30dWyegDAibw5CvFkAqRGxIkiaquk7prictwuCug/0)
**红蓝攻防实验室** .

渗透测试、应急响应、红蓝对抗、web渗透、漏洞挖掘、网络安全、信息安全

**vue介绍**

  

1.Vue.js是用于构建交互式的 Web 界面的库。

2.它提供了
MVVM数据绑定和一个可组合的组件系统,具有简单、灵活的API。从技术上讲，Vue.js集中在MVVM模式上的视图模型层,并通过双向数据绑定连接视图和模型。

3.实际的DOM操作和输出格式被抽象出来成指令和过滤器。相比其它的MVVM 框架,Vue.js 更容易上手。

4.Vue.js是一个用于创建Web交互界面的库。它让你通过简单而灵活的API创建由数据驱动的UI组件。

 **vue分辨**

  

 **插件Wappalyzer**

![]()

 **插件Vue.js devtools**

默认是灰色的只要亮了就是vue

![]()

 **有手就行分辨**

总之分辨vue很好分辨 比如dom元素中有个div的id是app （基本上是）

  * 

    
    
    document.getElementById('app')

  

 **vue测试**

  

那么都知道vue在开发时代码对于开发者都是可以清晰可见方便调式 实际发布 build后 代码将会被压缩

以下示例拿奇安信的鹰图作为示范

  

 **接口地址**

我们到以这种app开头的js文件

![]()

点击左下角的{}格式化代码 就可以看见

![]()

使用搜索功能搜索相关关键字

![]()

可以看到是通过$axios发起的请求 此处就是接口地址 请求类型包括需要的构造的params的参数等等 我们就可以手工测试

 **插件扫描superSearchPlus**

  

  * 

    
    
     插件地址：https://github.com/dark-kingA/superSearchPlus

这里已经实现了获取路径 身份信息 手机号码和其他信息等等（可能不全 结合手工测试）检索完毕可以点击目录扫描 一键整理扫描的接口地址

![]()

根据扫描app**.js的地址相关拼接而成

![]()

 **破解Vue.js devtools**

  

Vue Devtools 是 Vue 官方发布的调试浏览器插件，可以安装在 Chrome 和 Firefox
等浏览器上，直接内嵌在开发者工具中，使用体验流畅。Vue Devtools 由 Vue.js 核心团队成员 Guillaume Chau 和 Evan
You 开发。

所以这个插件唯一的一点就是只能本地调式 （另外也不算破解叫绕过吧）

 **环境**

  *   * 

    
    
     Vue.js devtoolsGithub下载地址：https://github.com/vuejs/vue-devtools/tree/v5.1.1

  *   * 

    
    
    ScriptCat谷歌商店下载即

ScriptCat脚本（绕过Vue.js devtools插件实现无视开发环境调式）

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    // ==UserScript==// @name         强制开启vue 调式工具// @namespace    http://tampermonkey.net/// @version      0.1.2// @description  强制启用 Vue.js devtools 开发者工具// @author       不懂安全的开发// @match        *://*/*// @run-at       document-end// @grant        none// ==/UserScript==  
    (function() {  if (!window.__VUE_DEVTOOLS_GLOBAL_HOOK__) {    return;  }  
      let isActivated = false;  
      const getConstructor = el => {    const vm = (el || 0).__vue__;    if (vm) {      return vm.constructor.super ? vm.constructor.super : vm.constructor;    }  };  
      const getVue = () => {    let Vue = window.Vue;  
        if (!Vue) {      Vue = getConstructor(document.getElementById('app'));    }  
        if (!Vue) {      // 遍历 dom 读取可能的 vue 实例      Vue = getConstructor([...document.body.querySelectorAll('div')].find(el => el.__vue__));    }    return Vue;  };  
      const enableDevtools = () => {    if (isActivated) {      return;    }  
        const Vue = getVue();  
        if (!Vue) {      return;    }  
        isActivated = true;  
        Vue.config.devtools = true;    window.__VUE_DEVTOOLS_GLOBAL_HOOK__.emit('init', Vue);  };  setTimeout(enableDevtools, 2500);})();

![]()

![]()

启用即可

以上两个插件安装完成找个vue站点

还是用奇安信的鹰图

插件界面 以下是路由

路由简单理解就是每个页面就是一个路由 通过router的方法来驱动跳转页面

![]()

 **vuex**

  

vuex是什么？

Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式,它由五部分组成：

分别是：state,actions,mutations,getters,modules

vuex 由5部分组成

state :数据状态数据状态  

actions:可以是异步状态

mutations：唯一可以修改state数据的场所

getters: 类似于vue组件中的计算属性，对state数据进行计算（会被缓存）

modules:模块化管理store（仓库），每个模块拥有自己的 state、mutation、action、getter

简单理解就是 一次登录成功之后获取了一些用户相关信息 比如登录状态 token 手机号码 用户id 类型 菜单 权限列表 用户状态 用户权限标识 等

vuex演示我们使用360的quake

![]()

 **如何利用**

修改当前vuex的值会实时同步

就拿360的quake一个国际化 localLanguage做示范

![]()

修改en 点击右侧保存  可以看到页面实时变化

![]()

这里只是个示范 实际情况

可能这是个用户类型 用户id 用户状态 用户的vip 时间等等 因为全局状态管理 一般使用的场景在vue组件化开发中内部使用调用 内部可能会加入业务逻辑计算
最终带入这个值 来驱动页面的变化等等

 **Components、props、data功能说明**

  

 **Components：**

组件 (Component) 是 Vue.js 最强大的功能之一。组件可以扩展 HTML
元素，封装可重用的代码。在较高层面上，组件是自定义元素，Vue.js 的编译器为它添加特殊功能。在有些情况下，组件也可以表现为用 is 特性进行了扩展的原生
HTML 元素。

 **props：**

简单理解就是对某个经常使用的页面 进行封装组件化 对外提供 api（props）同时对外输出等等

可以看到 鹰图的搜索框 使用的QInput组件 （猜测封装的）

![]()

可以看到这个组件对应有个props 这个就是组件对外提供的api 在你调用组件的同时 来进行传入props中定义key的值 来驱动组件的某些功能

 **data：**

data是Vue实例中一个配置项。用来存储vue实例或者组件里面的数值。

1、字符串

2、整数

3、数组

4、对象

5、对象数组

简单来说就是当前这个vue页面定义的当前页面有效值 来进行 业务的驱动 逻辑等

![]()

如上图可直接修改内部的data值

场景 a 页面 调用b页面（b是组件）a调用页面传入了props的某个对象或数组值 通过接口 或者当前页面定义的data值 或结果来进行逻辑处理
最终渲染dom 那么这里我们就可以操作data 来进行操作 窜改值（实际自己研究）

 ****

 **实战操作到getshell**

  

fofa大法随便搜几个vue站点 开始干（没有归属的哈 [狗头]）

![]()

找到个目标 目前有这些路由 开始构造访问

开始手动访问吧 （要是多起来可就恶心了 狗头）

这里先演示 正常操作并且绕过一些加载 以及css 处理 burp 等

![]()

没权限 点击好的 直接退出 由于前端拦截器 直接给你重定向到登录了

我们的目标是不让他弹 进到页面看看有什么功能 和渗透的点

 **方案1：**  

找到这个这个弹窗css删除掉  

![]()

 **方案2：**

直接找到没有权限的包 手动改code码 ok继续

![]()

  

![]()

  

ok 干净了 挨个菜单 点点 同样你点击菜单的时候看看浏览器中加载的请求 很有可能有未授权接口等，注意观察！！

这些菜单 其实就是刚刚vue组件里面 加载的 现在已经加载到页面了

路由多的时候 实在一个一个点 我比较懒 直接 写了脚本开扫 最后看看哪个页面能利用

  

![]()

呕吼 发现个上传点页面 点进去看看

呕吼 上传成功了

![]()

此刻我是未授权 没有任何登录 一个任意上传 到手

尝试开始梭哈 上传马子 盲猜jsp站 掏出我的经典冰蝎

  

![]()

root权限 美滋滋 有人敲门了 我去开

 **其他**

关注公众号 进安全开发交流群 一起分享技术 共同学习

  

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

