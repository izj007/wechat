#  使用Zuthaka管理不同APT和其他后渗透工具

Alpha_h4ck  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 国内网络安全行业门户

____

__

收录于话题

![](https://gitee.com/fuli009/images/raw/master/public/20211022213844.png)

Zuthaka是一款开源的应用程序，旨在帮助红队研究人员完成安全分析与管理任务。Zuthaka可以简化很多操作任务，比如说针对不同APT和其他后渗透工具的管理等等。

除此之外，Zuthaka还是一个免费的开源协作C&C整合框架，可以允许广大开发人员专注于其命令控制服务器的核心功能和目标。

当前的C2生态系统正在迅速发展，以适应现代红队作战和多样化需求。这也给安全专业人员带来了很大的额外工作量以及开销。创建C2系统已经是一项艰巨的任务了，而且大多数可用的C2系统系统都缺乏直观且易于使用的Web界面。

因此，Zuthaka便应运而生。Zuthaka可以提供简化的API，用于快速、清晰地集成C2，并通过统一的红队操作界面为多个C2实例提供集中管理。

 **下面给出的是Zuthaka的框架结构：**

![](https://gitee.com/fuli009/images/raw/master/public/20211022213849.png)

## Zuthaka组件

 **Zuthaka由以下优秀工具和框架组成：**

> Django Rest Framework
>
> Redis
>
> ReactJS
>
> Nginx
>
> Docker
>
> PostgreSQL

## 目前支持的C2

> Covenant
>
> Empire

## 开始使用

Zuthaka由一个前端和一个后端组成。前端负责提供UI界面，其中包括API处理管理器、文件管理器、Shell后渗透模块和常规的C2操作。后端负责处理Zuthaka实例化C2的一致性和可用性问题，并部署Redis作为消息代理，以异步处理代理UI和Nginx服务器中的每个元素。

### 依赖组件

如需自动化部署必要的关键基础设施（Zuthaka的前端和后端、Nginx、Redis），我们需要安装一个可用的Docker实例。

### 工具安装

如需构建完整的Zuthaka项目，首先我们需要下载并安装项目依赖组件：

    
          * 
    
    
    
    git clone https://github.com/pucara/zuthaka

如需使用特定的服务开启项目，则需要利用到Docker-Compose文件：

    
          * 
    
    
    
    docker-compose up

## 许可证协议

本项目的开发与发布遵循BSD-3开源许可证协议。

## 项目地址

>  **Zuthaka：** 【点击阅读原文】

## 参考资料

https://docs.zuthaka.com/

![](https://gitee.com/fuli009/images/raw/master/public/20211022213850.png)  

  

精彩推荐

  
  
  
  
  
 **
**![](https://gitee.com/fuli009/images/raw/master/public/20211022213851.png)****  

[![](https://gitee.com/fuli009/images/raw/master/public/20211022213852.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486647&idx=1&sn=13ae89f5104291b30864af54ff28ceda&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20211022213853.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486625&idx=1&sn=b68fcc53d322bb9a2e43d00a112ca40d&chksm=ce1cf63ef96b7f287356248af6a7c190268d07993c30b87488a27f8b1a6cb71f68be08ea4b78&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20211022213855.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486586&idx=1&sn=8fb235328402751e06c0578e05f3c905&scene=21#wechat_redirect)
** ** ** ** ** **
**![](https://gitee.com/fuli009/images/raw/master/public/20211022213856.png)**************

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

使用Zuthaka管理不同APT和其他后渗透工具

最多200字，当前共字

__

发送中

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

