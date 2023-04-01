#  ChatGPTScan:使用ChatGPTScan批量进行代码审计

原创 cokeBeer  [ 御林安全 ](javascript:void\(0\);)

**御林安全** ![]()

微信号 yulinSec

功能介绍 电子科技大学御林安全工作室官方公众号

____

___发表于_

收录于合集

  

  

  
  

**声明：**
本文内容仅供学习交流，所有因传播利用文章内相关技术造成的不良后果均由使用者本人负责，与本公众号和文章作者无关。如需转载，请注明出处，未经作者允许不得随意删改本文内容。

  
  

  

  

  

  

  

  

  

  

  

  

  

  

  

  

 **背景**

最近 chatgpt 杀进安全圈，除过各位师傅介绍的使用方法外，微软也公布了Copilot。

在网络安全应用方面，现在chatgpt 的优势是对于代码/日志/混淆等文本类信息有着较好的理解能力。

例如在代码审计时，常常会遇到一些大型项目，里面有很多复杂的函数。使用 chatgpt 辅助代码审计，可以快速理解这些函数的作用，提高代码审计效率。

这里尝试使用 chatgpt 实现一个代码审计平台，让 chatgpt 帮我挖洞。

  

 **收获**

添加了约 2w 个 github 开源项目进行扫描 （ 因为 api 成本问题，截止目前，扫描了一部分 ）， 在 github 某  1k star 和
1.2k star 的 开源项目中扫描到了两个高危问题，目前已经提交修复，并获得 CVE 编号

  

 **OPENAI API 简介**

为了方便发送代码和获取结果，以及后续实现自动化扫描功能，我们使用 openai 提供的 api 接口来和 chatgpt 交互。

下面先补充一些 api 接口的基本知识。目前 openai 开放了 api
接口，方便用户基于chatgpt的能力开发第三方程序。已经注册的用户访问管理面板可以创建 API key

  * 

    
    
    https://platform.openai.com/account/api-keys

![](https://gitee.com/fuli009/images/raw/master/public/20230401181954.png)

openai 也有官方文档提供了 API key 的使用教程。

  * 

    
    
    https://platform.openai.com/docs/api-reference

例如，我们可以通过 curl 请求，在 Authorization 头中带上 API key，和 openai 的 api
交互。这里我将OPENAI_API_KEY 写入了环境变量，直接读取发送。

  *   * 

    
    
    curl https://api.openai.com/v1/models \  -H "Authorization: Bearer $OPENAI_API_KEY"

![](https://gitee.com/fuli009/images/raw/master/public/20230401181956.png)

响应报文会列出可以使用的模型。

这里我们主要使用 gpt-3.5-turbo 这个模型，它也是目前使用最多的模型。

openai 也提供了封装好的 python 库。首先使用如下指令安装。

  * 

    
    
    pip install openai

然后可以编写一段简单的代码进行测试

  *   *   *   *   *   *   *   * 

    
    
    import openaiopenai.api_key = "sk-XXXXXXXXX"openai.proxy = "http://127.0.0.1:7890"result = openai.ChatCompletion.create(            model="gpt-3.5-turbo",            messages=[{"role": "user", "content": "Say this is a test!"}]        )print(result.choices[0].message.content)

运行之后即可输出 This is a test!。

这里 message 是一个数组，里面存放历史对话信息。

和网页版的 chatgpt 不同，api 版本可以一次性发送所有的历史对话信息，包括用户的历史提问和 chatgpt 的历史回答。

role 字段预设的角色有三个，user, system 和 assistant。其中 user 代表用户说的内容。system 代表系统说的内容，对于
chatgpt 后续的回答具有指示作用。assistant 代表 chatgpt 的回答。

将三个角色的说话内容加入mssages数组里，即可将整个对话的背景信息发送给 chatgpt。

  

  

 **原理**

  

在实现 ChatGPTScan 前，我们先分析一下在 web 版 chatgpt 进行代码审计的基本流程

首先发送一条请求，让 chatgpt 扮演一个安全专家

![](https://gitee.com/fuli009/images/raw/master/public/20230401181957.png)

然后将准备好的漏洞代码片段发送给 chatgpt，并且提出让它分析代码，找出漏洞然后根据严重程度分级

![](https://gitee.com/fuli009/images/raw/master/public/20230401181958.png)

chatgpt输出如下

![](https://gitee.com/fuli009/images/raw/master/public/20230401182000.png)

可以看到 chatgpt 比较清晰地输出了漏洞存在的位置，漏洞描述，以及给漏洞标注的危害等级

其中对于明显存在漏洞的代码，标注了高危；对于存在漏洞但是可能无法利用的代码，标注了中危；对于做了防护的代码，标注了低危

从上面的交互过程中可以总结出，我们需要发送给 chatgpt 的内容分成三个部分

第一，系统指令部分，告诉 chatgpt 扮演一个网络安全专家，引导 chatgpt 进入情景

第二，代码内容部分，将需要分析的代码发送给 chatgpt

第三，用户需求部分，告诉 chatgpt 需要做怎样的分析，以及以怎样的格式输出

设计 ChatGPT 代码审计的 prompt， 可以从这个三段式的结构入手

  

 **实现效果**

下面展示一下实现好的效果。项目后端采用 python + openai 库，前端采用 vue3 + element-plus

架构上， 扫描器和 web 管理界面分离， 可以支持 web 管理界面远程控制扫描器。

功能上， 实现了运行管理，项目列表，扫描结果，用户管理和密钥管理模块。

性能上，采用多线程加速扫描，理论 api 调用频率可以达到 openai 限制频率。

主要模块截图如下：

运行管理：这里可以监控扫描器运行状态，启动和停止扫描器。

![](https://gitee.com/fuli009/images/raw/master/public/20230401182001.png)

项目列表: 这里可以查看添加的项目信息，支持多种搜索条件。

![](https://gitee.com/fuli009/images/raw/master/public/20230401182002.png)

扫描结果：这里可以查看所有扫描结果，支持多种搜索条件，ChatGPT 生成的扫描报告会以 markdown 的形式渲染出来，方便查看。这里也在指令中要求
ChatGPT 明确地用 VulnRh3r3 关键字标注出大概率存在漏洞的报告，方便定位搜索有效漏洞报告。

![](https://gitee.com/fuli009/images/raw/master/public/20230401182003.png)

用户管理：可以查看用户状态，修改密码。

![](https://gitee.com/fuli009/images/raw/master/public/20230401182004.png)

密钥管理：可以动态添加和删除密钥, 并且查看余额。

![](https://gitee.com/fuli009/images/raw/master/public/20230401182005.png)

  

 **结语**

目前 gpt-3.5-turbo 的 context limit 是 4096个 token，也就是说 messages 所有内容加上 chagpt
最后给出的回答不能超过 4096 个 token，不然就会报错。

4096 个 token 支持的代码片段还比较小，只能审计一些较简单的漏洞。

后续 gpt-4 模型的 context limit 为 8k 个 token，gpt-4-32k 模型的 context limit 为 32k 个
token。

增大的 context limit 会更有利于代码审计，可以期待一下。

命令行版本的ChatGPTScanner已经开源：https://github.com/YulinSec/ChatGPTScanner

完整项目后续会在 github 账号 YulinSec 上开源，欢迎持续关注。

  

 _ **END**_

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230401182009.png)
**扫码关注我们** 御 梦 而 生   如 鹿 归 林  

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

