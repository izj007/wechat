#  【渗透技巧】Vue应用快速debug前端页面

利刃信安  [ 利刃信安攻防实验室 ](javascript:void\(0\);)

**利刃信安攻防实验室** ![]()

微信号 LRXAGFSYS

功能介绍
利刃信安攻防实验室是一家专注于提供网络安全解决方案的专业服务商。我们能够提供包括等保、软测试、密码评估、风险评估、渗透测试等项目合作服务。此外，我们还能够提供CISP、PTE、CISSP、PTS及其他各种网络安全证书培训考证服务。

____

___发表于_

收录于合集 #渗透技巧 1个

**Vue应用快速debug前端页面**

分享一个干货。现在黑盒测试的过程中会经常遇到Vue写的页面，做扫描器也会遇到这类情况，特别是一些后台，打开就跳到登录页面。这时候我想知道这个页面有哪些功能，就只能去扒JavaScript代码，不过几百K甚至几M的代码看起来也很费劲。分享一个小知识，可以用于快速debug前端页面：如果目标使用了Vue
Router，我们可以直接从控制台拿到所有的前端路由信息。虽然大部分前端应用都是所谓的“单页应用”，但实际上Vue是作用于某个DOM元素，而不是整个页面上。比如，我们通常会将Vue作用在一个空的div节点上：

  * 

    
    
    <div id="app"></div>

所以，想要分析Vue，需要先找到这个根节点DOM。然后，根据Vue版本的不同，我们可以找到这个根节点上的Vue对象。Vue主要分两个版本，Vue2和3，在Vue2下，我们通过访问DOM.__vue__可以拿到当前元素的Vue对象；在Vue3下，通过访问DOM.__vue_app__可以拿到当前元素的Vue对象。Vue几乎所有的信息都可以在Vue对象中找到，比如我前面说的路由。我写了个小脚本，只需要在Vue开发的页面控制台中执行这个脚本，即可获得当前网站所有的前端路由列表。

 **地址**

  * 

    
    
     https://github.com/phith0n/vueinfo

 **代码**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     function findVueRoot(root) {  const queue = [root];  while (queue.length > 0) {    const currentNode = queue.shift();  
        if (currentNode.__vue__ || currentNode.__vue_app__ || currentNode._vnode) {      console.log("vue detected on root element:", currentNode);      return currentNode    }  
        for (let i = 0; i < currentNode.childNodes.length; i++) {      queue.push(currentNode.childNodes[i]);    }  }  
      return null;}  
    function findVueRouter(vueRoot) {  let router;  
      try {    if (vueRoot.__vue_app__) {      router = vueRoot.__vue_app__.config.globalProperties.$router.options.routes    } else {      if (vueRoot.__vue__.$root.$options.router.options.routes) {        router = vueRoot.__vue__.$root.$options.router.options.routes      } else if (vueRoot.__vue__._router.options.routes) {        router = vueRoot.__vue__._router.options.routes      }    }  } catch (e) {}  
      return router}  
    function walkRouter(rootNode, callback) {  const stack = [{node: rootNode, path: ''}];  
      while (stack.length) {    const { node, path} = stack.pop();  
        if (node && typeof node === 'object') {      if (Array.isArray(node)) {        for (const key in node) {          stack.push({node: node[key], path: mergePath(path, node[key].path)})        }      } else if (node.hasOwnProperty("children")) {        stack.push({node: node.children, path: path});      }    }  
        callback(path, node);  }}  
    function mergePath(parent, path) {  if (path.indexOf(parent) === 0) {    return path  }  
      return (parent ? parent + '/' : '') + path}  
    function main() {  const vueRoot = findVueRoot(document.body);  if (!vueRoot) {    console.error("This website is not developed by Vue")    return  }  
      let vueVersion;  if (vueRoot.__vue__) {    vueVersion = vueRoot.__vue__.$options._base.version;  } else {    vueVersion = vueRoot.__vue_app__.version;  }  
      console.log("Vue version is ", vueVersion)  const routers = [];  
      const vueRouter = findVueRouter(vueRoot)  if (!vueRouter) {    console.error("No Vue-Router detected")    return  }  
      console.log(vueRouter)  walkRouter(vueRouter, function (path, node) {    if (node.path) {      routers.push({name: node.name, path})    }  })  
      return routers}  
    console.table(main())  
    

 **效果图**

![]()

如图是我获取的snyk的用户页面中所有路由列表。注意的是，这里只能获取前端路由列表，并不能获取后端路由列表。但是在获取到前端路由列表后，我们就可以写个简单的爬虫爬一下所有路由，这样就可以抓到一部分后端的流量了。

注意：实际测试过程中，获取到前端路径以后会在使用这些路径时也会直接跳转到登录页，若是已经登录下的情况会好很多。

 **来源：代码审计知识星球**

  

 **阅读  10万+**

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

