#  最新代码审计利器Fortify_SCA_v23.2.0/Windows/Linux/Mac版下载

原创 城北  [ 渗透安全HackTwo ](javascript:void\(0\);)

**渗透安全HackTwo** ![]()

微信号 CB-Hack

功能介绍 声明:
在此公众号文章学习和使用工具过程中，如果您在使用工具或使用该公众号的测试方法过程中存在任何非法行为，您需要自行承担后果，我们不负任何法律责任！ 公众号介绍：
分享漏洞挖掘技巧、收集各种CVE漏洞 、渗透测试工具的使用、网络安全的研究

____

___发表于_

前言

> > Fortify SCA 支持丰富的开发环境、语言、平台和框架，可对开发与生产混合环境进行安全检查。25 种编程语言 超过 911,000 个组件级
> API 可检测超过 961 个漏洞类别 支持所有主流平台、构建环境和 IDE。
>>

>> ![]()

>>

>> Fortify
SCA是一款商业软件，价格较为昂贵，因此我只找到了一个早期的版本进行试用。因为是商业软件，它有详细的使用文档，查阅非常方便。它支持一些IDE的插件功能，在安装的时候会有选项。

>>

>>  
>
>>

>> Fortify
SCA的代码审计功能依赖于它的规则库文件，我们可以下载更新的规则库，然后放置在安装目录下相应的位置。bin文件放置在安装目录下Coreconfigrules文件夹，xml文件放置在CoreconfigExternalMetadata文件夹（如果该文件夹没有则新建一个）。

>>

>> ![]()

>>

>> 打开Audit Workbench，点击Start New Project->Advanced
Scan选项就可以快速开始一个审计任务。选择需要审计的应用程序根目录，在Additional Options选项中选择使用的规则库，在Audit
Guide提出的四个问题中选择对应的选项，点击Run Scan即可。

>>

>> 支持的编程语言：

>>

>> ![]()

>>

>>  
>

  

01

# 更新介绍

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    在此版本中，Fortify 安全编码规则包可检测 33+ 种语言中的 1,403 个独特类别的漏洞，并跨越超过 100 万个单独的 API。总之，此版本包括以下内容：  
    改进了对 Android 13 的支持（支持的版本：33）Android 平台是专为移动设备设计的开源软件堆栈。Android 的一个主要组件是 Java API 框架，它向应用程序开发人员公开 Android 功能。此版本扩展了利用 Android 的 Java API 框架的 Java 或 Kotlin 编写的原生 Android 应用程序中的漏洞检测。此版本中针对 Android 应用程序引入了以下三个新的弱点类别：  
    Intent Manipulation: Implicit Internal IntentIntent Manipulation: Implicit Pending IntentIntent Manipulation: Mutable Pending IntentAndroid Jetpack （AndroidX）的初始支持 Android Jetpack 是一组库、工具和指南，可帮助开发人员更轻松地创建 Android 应用程序。Jetpack 涵盖 androidx.* 软件包，并且与平台 API 分离，这有助于促进向后兼容性并允许更频繁的更新。在此版本中，我们提供了此软件套件的初始覆盖范围。  
    Android Jetpack 的初始覆盖范围支持检测以下库中的弱点：  
    appsearch (version supported: 1.1.0-alpha03)compose.foundation (version supported: 1.5.1)compose.material (version supported: 1.5.1)compose.material3 (version supported: 1.1.2)compose.ui (version supported: 1.5.1)core (version supported: 1.12.0)credentials (version supported: 1.2.0-beta04)datastore (version supported: 1.0.0)security.crypto (version supported: 1.0.0)sqlite (version supported: 2.3.1)示例类别覆盖范围改进包括：  
    访问控制：数据库命令注入拒绝服务拒绝服务：正则表达式标头操作不安全的运输打开重定向密码管理：空密码密码管理：硬编码密码路径操作侵犯隐私资源注入服务器端请求伪造SQL 注入系统信息泄露系统信息泄露：内部MySQL Connector/Python 支持（支持版本：8.1.0）MySQL Connector/Python 是一个软件库，可促进 Python 应用程序和 MySQL 数据库之间的交互。它充当 Python 编程语言和 MySQL 数据库管理系统之间的桥梁或连接器，使开发人员能够使用 Python 代码轻松连接、查询和操作 MySQL 数据库中的数据。  
    改进的类别覆盖范围包括以下内容：  
    访问控制：数据库拒绝服务不安全传输：客户端身份验证已禁用不安全的传输：数据库不安全的传输：弱 SSL 协议密码管理密码管理：空密码密码管理：硬编码密码密码管理：弱加密路径操作服务器端请求伪造SQL 注入改进了对Django的支持（支持的版本：3.2）Django是一个用Python编写的Web框架，旨在促进安全和快速的Web开发。开发的速度和安全性是通过框架中的高度抽象来实现的，其中代码构造和生成用于大幅减少样板代码。在此版本中，我们更新了现有的 Django 覆盖范围以支持 3.2 版本之前的版本。  
    改进的覆盖范围包括以下命名空间：Django.contrib.auth.models，Django.db.models和Django.http.response。此外，改进的弱点类别覆盖范围包括以下内容：  
    云基础结构即代码 （IaC）基础结构即代码是通过代码管理和预配计算机资源的过程，而不是各种手动过程。支持的技术的覆盖范围扩大，包括用于部署到 Azure Microsoft 的 Terraform 配置，以及用于 AWS Ansible 的配置。与上述这些服务的配置相关的常见问题现在报告给开发人员。  
    Microsoft Azure Terraform 配置Terraform 是一个开源 IaC 工具，用于构建、更改和版本控制云基础架构。它使用自己的声明性语言，称为HashiCorp配置语言（HCL）。云基础架构在配置文件中编码，以描述所需状态。地形提供程序支持Microsoft Azure 基础结构的配置和管理。改进的弱点类别覆盖范围包括 Terraform 配置的以下内容：  
      
    2023 Common Weakness Enumeration (CWETM) Top 25常见的弱点枚举（CWE） 2019 年推出了 25 大最危险的软件弱点（CWE 前 25 名），取代了 SANS 前 25 名。2023 年 CWE 前 25 名于2023 年 6 月发布，使用启发式公式确定，该公式规范化了过去两年向国家漏洞数据库 （NVD） 报告的漏洞的频率和严重性。为了支持希望围绕NVD中最常报告的关键漏洞进行审核的客户，添加了Fortify分类法与2023年CWE Top 25的相关性。  
    OWASP API Security Top 10 2023  
    2023 年全球开放应用程序安全项目 （OWASP） API 安全 10 强提供了 2023年影响 API 的主要安全风险列表。它旨在提高对 API 安全弱点的认识，并教育参与 API 开发和维护的人员，例如需要保护 Web API 的开发人员、设计人员、架构师、经理和/或组织。  
    OWASP API 安全性 Top 10 侧重于影响 Web API 的弱点，它不打算单独使用，而是旨在与其他标准和最佳实践结合使用，以便彻底捕获所有相关风险。例如：它应与OWASP Top 10结合使用，以识别与输入验证（如注入）相关的问题。为了支持希望降低 Web 应用程序风险的客户，添加了强化分类法与新发布的 OWASP API 安全 2023 年 10 强的关联。  
    互联网安全中心 （CIS） 基准互联网安全中心 （CIS） 基准是社区开发的安全配置建议的集合，这些建议映射到 CIS 关键安全控制。这些建议旨在实现保护云基础架构的安全，并证明符合行业标准。CIS 基准不断更新，以适应所涵盖的 25+ 供应商产品系列不断变化的网络安全状态。支持的产品系列包括：  
    Amazon Elastic Kubernetes Service （EKS） Benchmark v1.3.0亚马逊云科技基础基准测试v2.0.0Azure Kubernetes Service （AKS） Benchmark v1.3.0谷歌云计算平台基准测试版 v2.0.0Google Kubernetes Engine （GKE） Benchmark v1.4.0Kubernetes Benchmark v1.7.1Microsoft Azure 基础基准测试版本 2.0.0智能合约弱点分类（SWC）[3]智能合约弱点分类（SWC）是一个系统框架，用于对智能合约中的漏洞进行分类和解释。它提供了一种标准化的方式来理解和解决在以太坊等区块链上运行的这些自动执行代码片段的弱点。值得注意的是，SWC 注册表的内容自 2020 年以来一直没有全面更新，导致已知的不完整、错误和重要遗漏。为了支持希望降低智能合约风险的客户，添加了强化分类法与当前版本的SWC的关联。  
    其他勘误表在此版本中，已投入资源以确保我们可以减少误报问题的数量，重构一致性，并提高客户审核问题的能力。客户还可以期望看到与以下内容相关的报告问题的变化：  
    弃用 20.x之前的 Fortify 静态代码分析器 正如 2022.4 版本所观察到的，我们将继续支持 Fortify 静态代码分析器的最后四个主要版本。因此，这将是支持 20.x 之前的 Fortify 静态代码分析器版本的规则包的最后一个版本。对于下一个版本，20.x 之前的 Fortify 静态代码分析器版本将不会加载最新的规则包。这将需要降级规则包或升级 Fortify 静态代码分析器的版本。对于未来的版本，我们将继续支持 Fortify 静态代码分析器的最后四个主要版本。  
    误报改进工作仍在继续，努力消除此版本中的误报。除了其他改进之外，客户还可以期望在以下方面进一步消除误报：  
    不安全部署：未修补的应用程序CVE-2023-25135 已发现 vBulletin 版本 5.6.0 至 5.6.8 中的预授权远程执行代码 （RCE） 漏洞。 vBulletin 是一种用于构建动态在线社区和论坛的流行软件，它不正确地清理用户提供的输入以进行未经身份验证的反序列化。此问题使攻击者能够在服务器上执行任意代码、滥用应用程序逻辑或装载拒绝服务 （DoS） 攻击。此版本包括一项检查，用于检测目标服务器上的此漏洞。  
    Prototype Pollution: Server-Side攻击者可以操纵对象的原型时，就会发生服务器端原型污染。这在基于原型的语言（如 JavaScript）中是可能的，它可以在运行时更改属性和方法。漏洞利用的严重性取决于污染对象在应用程序中的使用位置。攻击包括拒绝服务、更改应用程序配置，在某些情况下还包括远程代码执行。此版本包括检测 Web 应用程序中原型污染的检查。  
    合规报告  
    2023 年常见弱点枚举 （CWE） 前 25 名常见弱点枚举 （CWE） 前 25 个最危险的软件弱点 （CWE 前 25 名）于 2019 年推出，取代了 SANS 前 25 名。2023 年 CWE 前 25 名于 6 月发布，使用启发式公式确定，该公式规范化过去两年向国家漏洞数据库 （NVD） 报告的漏洞的频率和严重性。此 SecureBase 更新包括直接映射到 CWE 前 25 名标识的类别的检查，或通过“ChildOf”关系与前 25 名中的 CWE-ID 相关的 CWE-ID。  
    OWASP API 安全 2023 年 10 强 2023 年全球开放应用程序安全项目 （OWASP） API 安全 10 强提供了 2023年影响 API 的主要安全风险列表。它旨在提高对 API 安全漏洞的认识，并教育参与 API 开发和维护的人员，例如需要保护 Web API 的开发人员、设计人员、架构师、经理和一般组织。OWASP API 安全性前 10 名侧重于影响 Web API 的弱点，它不打算单独使用。相反，它旨在与其他标准和最佳实践结合使用，以彻底捕获所有相关风险。例如：将 OWASP API 安全 2023 年前 10 名与 OWASP 前 10 名结合使用，以识别与输入验证（如注入）相关的问题。此 SecureBase 更新包括一个新的合规性报告模板，该模板提供 OWASP API 安全性 2023 年十大类别与 WebInspect 检查之间的关联。  
    政策更新  
    2023 年 CWE 前 25 名 自定义策略以包含与 2023 年 CWE 前 25 名相关的检查已添加到 WebInspect SecureBase 受支持策略列表中。  
    OWASP API 安全性 2023 年 10 强 自定义策略以包含与 OWASP API 安全 2023年 10 强相关的检查，已添加到 WebInspect SecureBase 受支持策略列表中。此策略包含可用 WebInspect 检查的子集，使客户能够运行特定于合规性的 WebInspect 扫描。  
    其他勘误表  
    在此版本中，我们投入了资源来进一步减少误报的数量，并提高客户审核问题的能力。客户还可以期望看到与以下领域相关的报告结果的变化。  
    LDAP 注入 此版本包括对 LDAP 注入检查的改进，以减少误报并提高结果的准确性。  
    SSL 证书主机名差异 SSL 证书主机名差异检查报告内容现在包含更多详细信息，这些信息应帮助客户应用此安全问题的正确修复程序。  
    通过检查输入实现主动覆盖对于某些 WebInspect 检查，可以启用主动覆盖，以指导 WebInspect 发送针对更广泛端点的更长的攻击列表。此版本包括对这些检查的改进，使客户能够通过更改检查输入来配置主动覆盖范围，而不是向扫描策略添加单独的检查。具有主动覆盖功能的检查包括：Log4Shell、JNDI 引用注入、服务器端请求伪造、操作系统命令注入和服务器端原型污染。启用主动覆盖的检查可提供更准确的扫描，但是，重要的是要考虑到请求数和扫描时间可能会急剧增加。因此，Fortify 强烈建议您在单独的策略中启用主动覆盖的情况下运行检查，而不进行其他检查。  
    Web 服务器配置错误：未受保护的文件此版本包含一个小错误修复，以改进对 Java 相关配置文件的检测。  
    Fortify优质内容  
    研究团队在我们的核心安全智能产品之外构建、扩展和维护各种资源。  
    2023 年 CWE 前 25 名 为了配合新的相关性，此版本还包含 Fortify 软件安全中心的新报告包，支持 2023 年 CWE 前 25 名，可从 Fortify 客户支持门户的“高级内容”下下载。  
    OWASP API 安全 2023 年 10 强 为了配合新的相关性，此版本还包含 Fortify 软件安全中心的新报告包，支持 OWASP API 安全前 10 名，可从 Fortify 客户支持门户的“高级内容”下下载。

  

  

02

# 使用/安装方法

 **安装步骤：**  

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     一、解压补丁压缩包，把fortify.license和Fortify_SCA_23.1.0_windows_x64.exe、Fortify_Apps_and_Tools_23.1.0_windows_x64.exe放在同一目录，不要有中文。二、安装Fortify_SCA_23.1.0_windows_x64.exe，程序会自动找到fortify.license授权文件三、安装Fortify_Apps_and_Tools_23.1.0_windows_x64.exe，程序会自动找到fortify.license授权文件四、把fortify-common-23.1.0.0028.jar分别拷贝到 C:\Program Files\Fortify\Fortify_SCA_23.1.0\Core\lib\和C:\Program Files\Fortify\Fortify_Apps_and_Tools_23.1.0\Core\lib\ 下替换覆盖掉原来的五、解压FortifyRules_zh_CH_2023.1.1.0001(离线规则库).zip 规则库，把ExternalMetadata和rules文件夹拷贝到C:\Program Files\Fortify\Fortify_SCA_23.1.0\Core\config 下六、运行C:\Program Files\Fortify\Fortify_Apps_and_Tools_23.1.0\bin 下的auditworkbench.cmd 即可开启GUI界面七、根据需要配置扫描即可  
    2、规则库直接本地无法升级规则库，离线升级及最新中英文规则库。

注意：

  

否则激活不了

  

(REMEMBER TO PUT fortify.license IN SAME PATH OF INSTALLER)

  

This new version of Fortify SCA has split the UI tools from the installer and
made it CLI only.

  

To understand how to use it, refer to the user guide online or the
documentation inside installer archives.

  

（记住将 fortify.license 放在安装程序的同一路径中）

  

这个新版本的 Fortify SCA 将 UI 工具从安装程序中分离出来，仅使用 CLI。

  

要了解如何使用它，请参阅在线用户指南或安装程序存档中的文档。

 **扫描测试：**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     ourceanalyzer –b MyProject msbuild /t:rebuild Sample.slnsourceanalyzer –b MyProject -scan -f MyResults.fpr  
    FPRUtility.bat -information -listIssues -analyzerIssueCounts -project MyResults.fprFPRUtility.bat -information -listIssues -categoryIssueCounts -project MyResults.fpr  
    E:/sdk/boost_1_82_0/boost/function/function_base.hpp:297 (Dead Code)E:/sdk/boost_1_82_0/boost/function/function_base.hpp:297 (Poor Style: Variable Never Used)E:/sdk/boost_1_82_0/boost/function/function_base.hpp:307 (Dead Code)E:/sdk/boost_1_82_0/boost/locale/util.hpp:117 (Type Mismatch: Signed to Unsigned)E:/sdk/boost_1_82_0/boost/system/detail/system_category_message_win32.hpp:92 (Dead Code)E:/sdk/boost_1_82_0/boost/system/detail/system_category_message_win32.hpp:92 (Poor Style: Variable Never Used)E:/sdk/boost_1_82_0/boost/system/detail/system_category_message_win32.hpp:94 (Poor Style: Variable Never Used)update_tool.cpp:489 (Poor Style: Value Never Read)update_tool.cpp:494 (Poor Style: Value Never Read)

  

  

03

# 免责声明

# 获取方法

 **公众号回复20240115获取软件  
解压密码HackTwo**

 ****

![]()

 **后台回复" 星球"有优惠券，加入知识星球享受内部VIP资源，详情请点击下方链接了解（后续价格只涨不降）**

 **[点击了解-->>内部VIP知识星球福利介绍V1.2版本-
元旦优惠](http://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247484713&idx=1&sn=0fdab59445d9e0849843077365607b18&chksm=cf16a399f8612a8f6feb8362b1d946ea15ce4ff8a4a4cf0ce2c21f433185c622136b3c5725f3&scene=21#wechat_redirect)**

# 最后必看

本工具仅面向合法授权的企业安全建设行为，如您需要测试本工具的可用性，请自行搭建靶机环境。为避免被恶意使用，本项目所有收录的poc均为漏洞的理论判断，不存在漏洞利用过程，不会对目标发起真实攻击和漏洞利用。

  

在使用本工具进行检测时，您应确保该行为符合当地的法律法规，并且已经取得了足够的授权。请勿对非授权目标进行扫描。如您在使用本工具的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。本工具来源于网络，请在24小时内删除，请勿用于商业行为，自行查验是否具有后门，切勿相信软件内的广告！

  

在安装并使用本工具前，请您务必审慎阅读、充分理解各条款内容，限制、免责条款或者其他涉及您重大权益的条款可能会以加粗、加下划线等形式提示您重点注意。除非您已充分阅读、完全理解并接受本协议所有条款，否则，请您不要安装并使用本工具。您的使用行为或者您以其他任何明示或者默示方式表示接受本协议的，即视为您已阅读并同意本协议的约束，如有侵权请联系作者删除。

  

  

# 往期推荐

 **1.[内部VIP知识星球福利介绍V1.2版本-
元旦优惠](http://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247484713&idx=1&sn=0fdab59445d9e0849843077365607b18&chksm=cf16a399f8612a8f6feb8362b1d946ea15ce4ff8a4a4cf0ce2c21f433185c622136b3c5725f3&scene=21#wechat_redirect)**

**2.[最新Nessus2024.01.11.1版本漏洞扫描/探测工具下载Windows](http://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247484934&idx=1&sn=d8911f23507a2ba4944c198c9714182d&chksm=cf16a0b6f86129a05a98afb78daecde5a739e6cdd8d5d2836f0eca6389d35f614eaf5e9f4323&scene=21#wechat_redirect)**

 **3.**[
**最新xray1.9.11高级版下载Windows/Linux**](http://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247483882&idx=1&sn=e1bf597eb73ee7881ae132cc99ac0c8e&chksm=cf16a75af8612e4c73eda9f52218ccfc6de72725eb37aff59e181435de095b71e653b446c521&scene=21#wechat_redirect)  

 **4.**[
**2023HW的200+个poc发布直接下载**](http://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247484007&idx=1&sn=ebcd01a0433c35eb5de25a1963289d2b&chksm=cf16a4d7f8612dc188c97819beda444d24bb06060912b6c09cc80bbe50a646d81b3f0190d20d&scene=21#wechat_redirect)

**5.[最新Nessus2023.09漏洞扫描/探测工具下载Windows](http://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247484036&idx=1&sn=46cfe891b89cf5281506995929cabefb&chksm=cf16a434f8612d22c9c63398b405b40dc47f6c7f3687afdb7304a346aa8bae97eafc8a44c3dc&scene=21#wechat_redirect)**

  

渗透安全HackTwo  

微信号：关注公众号获取

加入星球：关注公众号获取

扫码关注 了解更多

![]()

  

喜欢的朋友可以点赞转发

![]()

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 最新代码审计利器Fortify_SCA_v23.2.0/Windows/Linux/Mac版下载

原创 城北  [ 渗透安全HackTwo ](javascript:void\(0\);)

轻触阅读原文

![]()

渗透安全HackTwo

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

