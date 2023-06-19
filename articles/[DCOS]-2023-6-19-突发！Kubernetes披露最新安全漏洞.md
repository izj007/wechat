#  突发！Kubernetes披露最新安全漏洞

原创 阎锡叁  [ DCOS ](javascript:void\(0\);)

**DCOS** ![]()

微信号 indagate

功能介绍 CNCF 云原生基金会大使，CoreDNS 开源项目维护者。主要分享云原生技术、云原生架构、容器、函数计算等方面的内容，包括但不限于
Kubernetes，Containerd、CoreDNS、Service Mesh，Istio等等

____

___发表于_

收录于合集

  

![](https://gitee.com/fuli009/images/raw/master/public/20230619142639.png)

******编辑   **｜  阎锡叁********

Kubernetes社区披露两起安全事件，涉及kube-
apiserver组件，主要为准入插件的逃逸漏洞，CVE-2023-2727及CVE-2023-2728，详细内容参见下文

![](https://gitee.com/fuli009/images/raw/master/public/20230619142640.png)

 **CVE-2023-2727:  ImagePolicyWebhook准入插件的逃逸**

  

CVSS评分: CVSS:3.1/AV:N/AC:L/PR:H/UI:N/S:U/C:H/I:H/A:N

在Kubernetes中发现了一个安全问题，当使用临时容器（即ephemeral
container）时，用户能够绕过被ImagePolicyWebhook限制的镜像来启动容器。只有使用了ImagePolicyWebhook准入插件和临时容器的Kubernetes集群才会受到影响。

![](https://gitee.com/fuli009/images/raw/master/public/20230619142642.png)

###  **Am I vulnerable?**

具备以下所有条件的集群，受该漏洞影响：

  1. 使用ImagePolicyWebhook准入插件限制某些镜像的使用
  2. Pod可以使用临时容器

###  **Affected Versions**

  * kube-apiserver v1.27.0 - v1.27.2

  * kube-apiserver v1.26.0 - v1.26.5

  * kube-apiserver v1.25.0 - v1.25.10

  * kube-apiserver <= v1.24.14

###  **How do I mitigate this vulnerability?**

上述漏洞可以通过社区提供的kube-apiserver补丁来避免，该补丁可以防止临时容器使用被ImagePolicyWebhook限制的镜像。

注意：还可以使用validate webhook方式（例如Gatekeeper和Kyverno）来强制执行相同的镜像限制策略。

###  **Fixed Versions**

  * kube-apiserver v1.27.3

  * kube-apiserver v1.26.6

  * kube-apiserver v1.25.11

  * kube-apiserver v1.24.15

###  **Detection**

使用临时容器且镜像受限于ImagePolicyWebhook的 Pod 发出update的请求，会记录在apiserver的审计日志中。也可以使用
kubectl get pods 命令，查找上述容器。

###  **Acknowledgements**

这个漏洞是由Stanislav Láznička报告的，由Rita Zhang修复。

  
![](https://gitee.com/fuli009/images/raw/master/public/20230619142643.png)

 **CVE-2023-2728: ServiceAccount准入插件的逃逸**

  

CVSS评分：CVSS:3.1/AV:N/AC:L/PR:H/UI:N/S:U/C:H/I:H/A:N

在Kubernetes中发现了一个安全问题，当使用临时容器（即ephemeral
container）时，用户能够绕过ServiceAccount的限制来获取secret。只有使用了ServiceAccount准入插件且包含临时容器的的Pod的annotation有kubernetes.io/enforce-
mountable-secrets 字段的Kubernetes集群才会受到影响。

###  **Am I vulnerable?**

具备以下所有条件的集群，受该漏洞影响：

  1. 使用了ServiceAccount准入插件。大多数集群默认开启，因为在https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#serviceaccount中推荐使用该插件
  2. ServiceAccount的annotation字段包含kubernetes.io/enforce-mountable-secrets。此字段不是默认添加的

  3. Pod使用临时容器

###  **Affected Versions**

  * kube-apiserver v1.27.0 - v1.27.2

  * kube-apiserver v1.26.0 - v1.26.5

  * kube-apiserver v1.25.0 - v1.25.10

  * kube-apiserver <= v1.24.14

###  **How do I mitigate this vulnerability?**

上述漏洞可以通过社区提供的kube-apiserver补丁来避免，该补丁可以防止临时容器使用被ServiceAccount限制的secret。

###  **Fixed Versions**

  * kube-apiserver v1.27.3

  * kube-apiserver v1.26.6

  * kube-apiserver v1.25.11

  * kube-apiserver v1.24.15

###  **Detection**

使用临时容器且secret受限于ServiceAccount的 Pod
发出update的请求，会记录在apiserver的审计日志中。您还可以使用kubectl get pods查找。

由于笔者时间、视野、认知有限，本文难免出现错误、疏漏等问题，期待各位读者朋友、业界专家指正交流。

  

参考文献  

   1\. https://github.com/kubernetes/kubernetes/issues/118640

  

  

 **👇🏻  真诚推荐你关注👇🏻**

  

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

