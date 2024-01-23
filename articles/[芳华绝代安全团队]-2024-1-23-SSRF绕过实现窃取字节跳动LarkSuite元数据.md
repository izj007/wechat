#  SSRF绕过实现窃取字节跳动LarkSuite元数据

原创 ice  [ 芳华绝代安全团队 ](javascript:void\(0\);)

**芳华绝代安全团队** ![]()

微信号 ifhsec

功能介绍 芳华绝代安全团队与君共芳华。

____

___发表于_

## 前言

LarkSuite是字节跳动旗下的一款集成办公套件，提供团队协作、通讯、日程管理等功能的企业级应用。  

![]()

SSRF 通常存在于以下功能点：  
1、消息功能：部分聊天功能会渲染任何链接的预览。  
2、文件转换：将用户可控内容转换为PDF等文件类型。  
3、文件上传：SVG文件如果在服务器端渲染可能导致SSRF。  
4、文件导入：例如Excel文档、Word文档、Zip文件等通常需要服务器端处理。修改XML文件（存在于Excel、Word中）可能导致文档在服务器端处理时出现SSRF。  
  
SSRF  
本次渗透功能点即为`文件导入`：

![]()

尝试导入 Microsoft Office 文档，若文档中的 XML
得到处理且允许外部实体，则可能会泄露文件/URL等信息。但使用多个Payload后，仍不见有效回显。  

经过进一步测试，排除利用 Office 文档中任何内容的可能性。

现在还有 2 个选项可供导入：CSV 和 Confluence ZIP 文件。CSV 不需要太多处理，故关注 Confluence 文件。

Confluence 提供了导出页面的选项。导出为 ZIP 文件的选项之一是包含每个页面的 HTML 内容。导出示例如下所示：

![]()

每个页面都表示为 HTML 文件，任何图像/附件/样式都存储在本地由 HTML 文件引用。示例片段如下，附件在 Confluence 页面中用作图像：  

![]()

每当将 Confluence zip 上传到 Lark 时，原始 Confluence 页面中包含的所有图像都会上传/添加到生成的 Lark Wiki
页面中。  
  
既然 Lark 通过路径获取内容，此时将路径更改为实际的 URL，或许 Lark 将在任何 URL 抓取图像。  
  
故开始验证，尝试修改图像源以指向 Burp Collaborator 域，如下所示：

![]()

  

将修改后的 HTML 保存并添加回 ZIP ，再导入到 Lark 中。回显如下：

![]()

Collaborator 收到了请求，且生成的 Lark Wiki 页面包含来自 Collaborator 的完整回复正文。

故验证成功，且意味着提供的 URL 不必是图像，即除内网的任何 URL 的内容都可作为附件生成至 Lark Wiki 页面中。

扩大危害  
请求 Burp Collaborator 的 IP 来自 AWS，故可尝试从元数据 URL (169.254.169.254) 中获取 AWS
凭证。修改文件如下并重复上述操作：

![]()

然而并无回显，推测程序的 SSRF 防护导致不可访问内部 IP。

进行子域名枚举再进行测试，成功概率很小但仍可尝试，然而此法失败。 **重定向** 是绕过SSRF的一个方法。

重定向实战可参考该文：  
[利用Gopher实现雅虎邮件SSRF升级至RCE命令执行](http://mp.weixin.qq.com/s?__biz=MzI4NTYwMzc5OQ==&mid=2247484131&idx=1&sn=ef648ad60ec3d08c0c39b166524afca7&chksm=ebe8e05bdc9f694d1b6af3c8e76f2e4ef9372b7bf5061d745febb2ed10c8060bff3337ed5285&scene=21#wechat_redirect)

使用重定向的渗透过程如下：

> 1、使用重定向脚本将流量从我的服务器302重定向到AWS元数据 URL
>
> 2、修改 Confluence 页面的图像 URL，使其指向我的服务器
>
> 3、保存并导入压缩文件到 Lark（如果重定向被跟随并绕过了保护机制，则渗透成功）
>
> 4、失败，重定向并未绕过SSRF防护

  

似乎山重水复疑无路了，再想想，我站在河边（服务端），对岸有flag（内网），我又不会游泳（权限），如何拿到对岸的flag呢？答案很简单，桥或摆渡人。

所以，DNS 重新绑定是值得尝试的。简要来说就是将一个域名绑定到一个合法的IP地址上，在成功建立连接后再将域名绑定到内网IP，从而绕过请求伪造保护。

https://lock.cmpxchg8b.com/rebinder.html可用于轻松生成 DNS 重新绑定域，使用它生成一个能在 Google IP
和内部 AWS 元数据 IP 之间切换的域名：

![]()

如图：

![]()

更新图像源URL：

![]()

基于DNS 重新绑定的随机性，重复执行导入操作，最终得到回显：

![]()

附件包含来自其元数据 URL 的目录列表，访问 `http://169.254.169.254/latest/meta-data/identity-
credentials/ec2/security-credentials/ec2-instance`即可得到 ec2 的 AWS 凭证。

参考链接：https://sirleeroyjenkins.medium.com  
本公众号不承担任何由于传播、利用本公众号所发布内容而造成的任何后果及法律责任。未经许可，不得转载。

* * *

  
芳华绝代安全团队现已推出Web安全渗透教程，欢迎学习：https://space.bilibili.com/602205041

![]()

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# SSRF绕过实现窃取字节跳动LarkSuite元数据

原创 ice  [ 芳华绝代安全团队 ](javascript:void\(0\);)

轻触阅读原文

![]()

芳华绝代安全团队

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

