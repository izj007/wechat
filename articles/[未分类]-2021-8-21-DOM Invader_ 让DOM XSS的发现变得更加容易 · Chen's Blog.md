[ ![](/assets/avatar.jpg) ](https://gh0st.cn/) Back to Index

# ᴀᴜᴛʜᴏʀ: ᴠᴜʟᴋᴇʏ_ᴄʜᴇɴ

[ᴀʙᴏᴜᴛ](/about) | [ʟɪɴᴋs](/links) | [ᴀssɪsᴛ ᴛᴏᴏʟ](/AssistTool) |
[ʀɢᴘᴇʀsᴏɴ](/RGPerson) | [ʙɪɴᴀʀʏ ʟᴇᴀʀɴɪɴɢ](/Binary-Learning)

# DOM Invader: 让DOM XSS的发现变得更加容易

July 1, 2021

# DOM Invader: 让DOM XSS的发现变得更加容易

编译作者：key

文章原文：https://portswigger.net/blog/introducing-dom-invader

## 背景

大多数现代网站都使用多个JavaScript库，并且有很多行复杂的压缩代码，这使得对DOM
XSS的测试变得非常令人头痛，PortSwigger安全研究部门专门开发了DOM Invader，使对DOM XSS的测试更加容易。

Augmented DOM功能允许你方便的找到DOM XSS，使用它，DOM XSS就像是反射的XSS一样。

通过Augmented DOM功能，DOM
Invader将为你提供一个方便的树状视图，以此来显示目标的源和汇（不理解别着急，这是BurpSuite的一个定义），这极大地简化了发现DOM
XSS的过程。

废话不多说，让我们仔细看看DOM Invader能做什么：

![](https://chen-blog-oss.oss-cn-
beijing.aliyuncs.com/2021-07-01/16251134048508.jpg)

如上图所示，基于DoraBox的DOM
XSS环境，我们知道了mq7tdzib这个值是基于location.search获取的，也知道了这个值最终会通过document.write输出。

## 开始使用 DOM Invader

你需要有BurpSuite Professional / Community 2021.7，然后在BurpSuite中打开你的嵌入式浏览器，默认情况下DOM
Invader是关闭的，你需要手动开启：

  1. 通过如下图的方式你可以找到这个扩展，点击它就可以：

![](https://chen-blog-oss.oss-cn-
beijing.aliyuncs.com/2021-07-01/16251134318096.jpg)

  1. 在设置界面中选择开启它：

![](https://chen-blog-oss.oss-cn-
beijing.aliyuncs.com/2021-07-01/16251134197040.jpg)

DOM Invader 探测目标的 DOM，拦截它可能遇到的任何 JavaScript 源和汇，并将它们组织起来供你使用。

源（Sources）：表示任何允许用户控制的输入的JavaScript对象，例如：location.search；

汇（Sinks）：表示任何允许JavaScript/HTML执行的函数或设置器，例如：eval、document.write。

DOM Invader会对汇（Sinks）进行排序，使最有价值的的汇（Sinks）排列在最前面。

在DOM
Invader中，我们将大量使用金丝雀（canary，这也是BurpSuite定义的一个概念，这里你可以理解为是一个字符串），金丝雀（canary）是一个独特的字符串，用于查看用户输入在汇（Sinks）中的反映。

默认情况下，DOM Invader使用一个随机的金丝雀（canary），不过你也可以将这个值自定义为你喜欢的任何值。

## DOM Invader 工作原理

在Burp Suite的嵌入式浏览器中打开DevTools，你就可以看见一个新的`Augmented
DOM`标签，它显示任何包含金丝雀（Canary）值的源（Sources）和汇（Sinks），以及所有可用的源（Sources）和汇（Sinks）的树状视图。

当你找到一个有趣的汇（Sinks），你可以看到其中包含的值，以及堆栈跟踪，并会突出显示你的金丝雀（Canary），通过Augmented
DOM，你也可以检查你自定义的金丝雀（Canary）值是否被正确编码。

其他有用的功能包括能够搜索发送到汇（Sinks）的值，以及自动将金丝雀（Canary）注入到URL参数和表单元素中。

![](https://chen-blog-oss.oss-cn-
beijing.aliyuncs.com/2021-07-01/16251134658058.jpg)

## 源（Sources）和汇（Sinks）的清单

### Sources

    
    
    const sourcesList = [
        "location",
        "location.href",
        "location.hash",
        "location.search",
        "location.pathname",
        "document.URL",
        "window.name",
        "document.referrer",
        "document.documentURI",
        "document.baseURI",
        "document.cookie"
    ];
    

### Sinks (排名)

    
    
    const sinkRanking = {
        "jQuery.globalEval":1,
        "eval":2,
        "Function":3,
        "execScript":4,
        "setTimeout":5,
        "setInterval":6,
        "setImmediate":7,
        "msSetImmediate":7,
        "script.src":8,
        "script.textContent":9,
        "script.text":10,
        "script.innerText":11,
        "script.innerHTML":12,
        "script.appendChild":13,
        "script.append":14,
        "document.write": 15,
        "document.writeln": 16,
        "jQuery":17,
        "jQuery.$":18,
        "jQuery.constructor":19,
        "jQuery.parseHTML":20,
        "jQuery.has":20,
        "jQuery.init":20,
        "jQuery.index":20,
        "jQuery.add": 20,
        "jQuery.append": 20,
        "jQuery.appendTo": 20,
        "jQuery.after": 20,
        "jQuery.insertAfter": 20,
        "jQuery.before": 20,
        "jQuery.insertBefore": 20,
        "jQuery.html": 20,
        "jQuery.prepend": 20,
        "jQuery.prependTo": 20,
        "jQuery.replaceWith": 20,
        "jQuery.replaceAll": 20,
        "jQuery.wrap": 20,
        "jQuery.wrapAll": 20,
        "jQuery.wrapInner": 20,
        "jQuery.prop.innerHTML": 20,
        "jQuery.prop.outerHTML": 20,
        "element.innerHTML":21,
        "element.outerHTML":22,
        "element.insertAdjacentHTML":23,
        "iframe.srcdoc": 24,
        "location.href":25,
        "location.replace":26,
        "location.assign":27,
        "location":28,
        "window.open":29,
        "iframe.src":30,
        "javascriptURL":31,
        "jQuery.attr.onclick":32,
        "jQuery.attr.onmouseover":32,
        "jQuery.attr.onmousedown":32,
        "jQuery.attr.onmouseup":32,
        "jQuery.attr.onkeydown":32,
        "jQuery.attr.onkeypress":32,
        "jQuery.attr.onkeyup":32,
        "element.setAttribute.onclick":33,
        "element.setAttribute.onmouseover":33,
        "element.setAttribute.onmousedown":33,
        "element.setAttribute.onmouseup":33,
        "element.setAttribute.onkeydown":33,
        "element.setAttribute.onkeypress":33,
        "element.setAttribute.onkeyup":33,
        "createContextualFragment":34,
        "document.implementation.createHTMLDocument": 35,
        "xhr.open":36,
        "xhr.send": 36,
        "fetch": 36,
        "fetch.body": 36,
        "xhr.setRequestHeader.name": 37,
        "xhr.setRequestHeader.value": 38,
        "jQuery.attr.href":39,
        "jQuery.attr.src":40,
        "jQuery.attr.data":41,
        "jQuery.attr.action":42,
        "jQuery.attr.formaction":43,
        "jQuery.prop.href":44,
        "jQuery.prop.src":45,
        "jQuery.prop.data":46,
        "jQuery.prop.action":47,
        "jQuery.prop.formaction":48,
        "form.action":49,
        "input.formaction":50,
        "button.formaction":51,
        "button.value": 52,
        "element.setAttribute.href":53,
        "element.setAttribute.src":54,
        "element.setAttribute.data":55,
        "element.setAttribute.action":56,
        "element.setAttribute.formaction":57,
        "webdatabase.executeSql": 58,
        "document.domain":59,
        "history.pushState":60,
        "history.replaceState":61,
        "xhr.setRequestHeader":62,
        "websocket":63,
        "anchor.href":64,
        "anchor.target": 65,
        "JSON.parse": 66,
        "document.cookie":67,
        "localStorage.setItem.name": 68,
        "localStorage.setItem.value": 69,
        "sessionStorage.setItem.name": 70,
        "sessionStorage.setItem.value": 71,
        "element.outerText": 72,
        "element.innerText": 73,
        "element.textContent": 74,
        "element.style.cssText": 75,
        "RegExp":76,
        "window.name":77,
        "location.pathname": 78,
        "location.protocol": 79,
        "location.host": 80,
        "location.hostname": 81,
        "location.hash": 82,
        "location.search": 83,
        "input.value": 84,
        "input.type": 85,
        "document.evaluate": 86
    };
    

[INDEX](https://gh0st.cn/ "Back to Index") *
[>>](https://gh0st.cn/archives/2021-05-05/1 "PREV: 某VPN客户端远程下载文件执行挖掘")

</> Copyright (C) Chen's Blog (GH0ST.CN) 2016-2021

