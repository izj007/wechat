#  基于任务编排的漏扫实现(asm项目)

原创 leveryd [ leveryd ](javascript:void\(0\);)

**leveryd** ![]()

微信号 gh_8d7f6ed4daff

功能介绍 个人记录：应用安全、云/云原生安全、安全产品

____

___发表于_

收录于合集 #漏洞扫描 11个

#

# https://github.com/leveryd-asm/asm
实现了上周说的《基于任务编排玩一玩漏扫》，如果你是在资源有限的情况下做漏扫，或者想学一下k8s的运维使用，可以一键部署玩下这个项目。欢迎和我交流，我的微信是
happy_leveryd。下面向你介绍一下asm项目。

#

# asm是什么？

 **提交爬扫任务**|  **查看报警**  
---|---  
![]()| ![]()  
 **提交POC扫描任务**|  **查看任务状态**  
![]()| ![]()  
  
设计思路见
[基于任务编排玩一玩漏扫](https://mp.weixin.qq.com/s?__biz=MzkyMDIxMjE5MA==&mid=2247485271&idx=1&sn=8aba702fbb1af37100cf12fb4cf380ca&scene=21#wechat_redirect)

# 特点

 **💻  开箱即用 **内置五条工作流，只需要输入资产信息，就可以完成扫描任务 **🕸 任务编排** 基于argo-
workflow提供功能丰富、稳定的任务编排能力 **🔗  基于kubernetes
**任务编排引擎基于kubernetes调度工作容器，因此很容易实现通过水平扩展提升扫描性能；通过kubesphere可以更好地观测、运维应用 **🤖
管理控制台**
向用户提供UI界面管理资产、运营漏洞；对于开发者来说，想要在控制台新增一个模板可以很快，常规的crud操作只需要通过配置选项就能完成模块的前后端开发

# 运维指南

## 在k8s集群中一键部署

  * 安装 kubesphere

kubesphere 可以用来管理k8s集群，并且提供了`ingress controller`。

可以按照如下命令安装`v3.3.1`版本

    
    
    kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.3.1/kubesphere-installer.yaml  
    kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.3.1/cluster-configuration.yaml  
    

> 详细安装步骤参考 kubesphere

如果你不需要kubesphere，可以使用`ingress-nginx`作为`ingress controller`。

  * 安装本项目

第一次安装，需要执行`helm dependency build`下载依赖。

执行如下命令会在asm命名空间中安装本项目

    
    
    helm -n asm template ./helm | kubectl apply -n asm -f -  
    

你也可以向helm传递参数来修改安装的配置，如下命令会使用`manage.com`作为域名访问控制台

    
    
    helm -n asm template ./helm --set console_domain=manage.com  | kubectl apply -n asm -f -  
    

  * 卸载本项目

    
    
    helm -n asm template ./helm | kubectl delete -n asm -f -  
    

# 用户指南

## 怎么访问asm控制台？

绑定域名到node节点后(域名默认是`console.com`)，在`kubesphere`控制台上找到`console`
ingress访问地址，如下图所示

![]()![]()

访问服务进入到asm控制台![]()

## 怎么对某个域名做漏洞扫描？

可以选择默认的`nuclei扫描-保存结果`模板，如下图所示。输入域名后，点击`Submit`按钮，等待扫描完成即可。

![]()

默认还有其他模板，可以针对二级域名(比如`leveryd.top`)做扫描，或者针对库中的二级域名列表做扫描。你可以根据自己的需求选择合适的模板。

## 怎么运营漏洞？

你可以在asm控制台上运营漏洞。

xray扫描的漏洞也会通过webhook推送到企业微信群。

> 需要你在安装本项目时有设置`weixin_webhook_url`参数，比如`helm --set
> weixin_webhook_url=https://qyapi.weixin.qq.com/cgi-
> bin/webhook/send?key=07d4613c-45ef-46e2-9379-a7b2aade3132`

## 怎么管理扫描任务？

你可以在控制台上管理扫描任务，具体办法如下。

浏览器输入 `http://asm控制台地址/argo` 地址，进入到argo界面，点击`Submit
Workflow`按钮，选择一个模板后，创建一个扫描任务.

![]()

## 怎么管理任务模板？

你可以在控制台上管理任务模板 ，目前默认有五个工作流，功能分别是：

  * 从API获取兄弟域名-获取子域名-nuclei扫描-保存结果
  * 从API获取兄弟域名-获取子域名-katana爬虫-xray扫描-保存结果
  * 获取子域名-nuclei扫描-保存结果
  * 获取子域名-katana爬虫-xray扫描-保存结果
  * nuclei扫描-保存结果

子域名扫描用到`oneforall`、`subfinder`等工具。

当然你也可以定义自己的任务模板，可以参考`helm/templates/argo`目录下的模板文件。

  

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

