#  云渗透 | Azure 渗透测试

大牛逼  [ 安全帮Live ](javascript:void\(0\);)

**安全帮Live** ![]()

微信号 gh_499ac9d326f5

功能介绍 安全帮 帮你学安全

____

___发表于_

收录于合集

  

## 基本信息

{% content-ref url="az-basic-information.md" %} az-basic-information.md {%
endcontent-ref %}

## Azure Pentester/红队方法论

为了审核 AZURE 环境，了解以下内容非常重要：正在使用哪些服务、正在公开什么、谁可以访问什么以及内部 Azure 服务和外部服务如何连接。

从红队的角度来看，破坏 Azure 环境的第一步是设法获取 Azure AD 的一些凭据。在这里，您对如何做到这一点有一些想法：

  * github（或类似）中的泄漏- OSINT

  * 社会工程学

  * 密码重用（密码泄露）

  * Azure 托管应用程序中的漏洞

    * `/home/USERNAME/.azure`

    * `C:\Users\USERNAME\.azure`

    * `accessTokens.json`2.30 之前的文件`az cli`\- Jan2022 -以明文形式存储访问令牌

    * 该文件`azureProfile.json`包含有关登录用户的信息。

    * `az logout`删除令牌。

    * . 中以明文形式`Az PowerShell`存储的访问令牌的旧版本。它还将ServicePrincipalSecret以明文形式存储在. 该 cmdlet可用于存储令牌。 用于删除它们。`TokenCache.dat``AzureRmContext.json``Save-AzContext`  
`Disconnect-AzAccount`

    * ****访问元数据端点的服务器端请求伪造

    * 读取本地文件

  * 第三者违约

  * 内部员工

  * 常见网络钓鱼（凭据或 Oauth 应用程序）

    * 设备代码身份验证网络钓鱼

  * Azure密码喷洒 

  

  

即使您没有破坏您正在攻击的 Azure 租户内的任何用户，您也可以从中收集一些信息：

{% content-ref url="az-unauthenticated-enum-and-initial-entry/" %} az-
unauthenticated-enum-and-initial-entry {% endcontent-ref %}

{% hint style="info" %} 在您设法获得凭据后，您需要知道这些凭据属于谁，以及他们可以访问什么，因此您需要执行一些基本的枚举：{%
endhint %}

## 基本枚举

{% hint style="info" %} 请记住，枚举中最嘈杂的部分是登录，而不是枚举本身。 {% endhint %}

### SSRF

如果您在 Azure 中的机器中发现了 SSRF，请检查此页面以获取技巧：

{% embed url=" https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-
request-forgery/cloud-ssrf " %}

### 绕过登录条件

![](https://gitee.com/fuli009/images/raw/master/public/20230320091332.png)

如果您拥有一些有效凭据但无法登录，这些是一些可能到位的常见保护措施：

  * IP白名单——你需要妥协一个有效的IP

  * 地理限制——找到用户居住的地方或公司的办公室在哪里，并从同一个城市（或至少一个国家）获得一个IP

  * 浏览器——可能只允许使用来自特定操作系统（Windows、Linux、Mac、Android、iOS）的浏览器。找出受害者/公司使用的操作系统。

  * 您还可以尝试破坏服务主体凭据，因为它们通常受限较少且其登录审核较少

绕过它后，您也许可以回到初始设置，并且您仍然可以访问。

### 我是谁

{% hint style="danger" %}在Az - AzureAD部分了解如何安装az cli、AzureAD 和 Az PowerShell
。{% endhint %}

  * 

    
    
    https://github.com/carlospolop/hacktricks-cloud/blob/master/pentesting-cloud/azure-security/az-services/az-azuread.md

  

你需要知道的第一件事就是你是谁（你在哪个环境中）：

{% tabs %} {% tab title="az cli" %}

    
    
    az account list   
    az account tenant list # Current tenant info  
    az account subscription list # Current subscription info  
    az ad signed-in-user show # Current signed-in user  
    az ad signed-in-user list-owned-objects # Get owned objects by current user  
    az account management-group list #Not allowed by default

{% endtab %}

{% tab title="AzureAD" %}

    
    
    #Get the current session state  
    Get-AzureADCurrentSessionInfo  
    #Get details of the current tenant  
    Get-AzureADTenantDetail

{% endtab %}

{% tab title="Az PowerShell" %}

    
    
    # Get the information about the current context (Account, Tenant, Subscription etc.)  
    Get-AzContext  
    # List all available contexts  
    Get-AzContext -ListAvailable  
    # Enumerate subscriptions accessible by the current user  
    Get-AzSubscription  
    #Get Resource group  
    Get-AzResourceGroup  
    # Enumerate all resources visible to the current user  
    Get-AzResource  
    # Enumerate all Azure RBAC role assignments  
    Get-AzRoleAssignment # For all users  
    Get-AzRoleAssignment -SignInName test@corp.onmicrosoft.com # For current user

{% endtab %} {% endtabs %}

### AzureAD 枚举

默认情况下，任何用户都应该有足够的权限来枚举我们、用户、组、角色、服务主体等内容……（检查默认的 AzureAD 权限）。  
您可以在这里找到指南：

{% content-ref url="az-services/az-azuread.md" %} az-azuread.md {% endcontent-
ref %}

{% hint style="info" %}
现在你已经有了一些关于你的凭据的信息（如果你是红队，希望你没有被发现）。是时候弄清楚环境中正在使用哪些服务了。  
在下一节中，您可以检查一些枚举一些常见服务的方法。{% endhint %}

## 服务主体和访问策略

Azure 服务可以具有（服务本身的）系统标识或使用用户分配的托管标识。这个身份可以有访问策略，例如，一个 KeyVault
来读取秘密。这些访问策略应该受到限制（最小特权原则），但可能具有比所需更多的权限。通常，应用服务会使用 KeyVault 来检索机密和证书。

因此探索这些身份是有用的。

## 应用服务供应链管理

Kudu 控制台登录到应用服务“容器”。

## Webshell

使用 portal.azure.com 并选择 shell，或使用 shell.azure.com 作为 bash 或
powershell。此外壳的“磁盘”作为图像文件存储在存储帐户中。

## Azure 开发运营

Azure DevOps 独立于 Azure。它有存储库、管道（yaml 或发布）、看板、wiki 等等。变量组用于存储变量值和秘密。

## 自动侦察工具

### 侦察

https://github.com/dirkjanm/ROADtools

    
    
    cd ROADTools  
    pipenv shell  
    roadrecon auth -u test@corp.onmicrosoft.com -p "Welcome2022!"  
    roadrecon gather  
    roadrecon gui

### Monkey365

https://github.com/silverhack/monkey365

{% code overflow="wrap" %}

    
    
    Import-Module monkey365  
    Get-Help Invoke-Monkey365  
    Get-Help Invoke-Monkey365 -Detailed  
    Invoke-Monkey365 -IncludeAzureActiveDirectory -ExportTo HTML -Verbose -Debug -InformationAction Continue  
    Invoke-Monkey365 - Instance Azure -Analysis All -ExportTo HTML

{% endcode %}

###

### Stormspotter

https://github.com/Azure/Stormspotter

###

    
    
    # Start Backend  
    cd stormspotter\backend\  
    pipenv shell  
    python ssbackend.pyz  
      
    # Start Front-end  
    cd stormspotter\frontend\dist\spa\  
    quasar.cmd serve -p 9091 --history  
      
    # Run Stormcollector  
    cd stormspotter\stormcollector\  
    pipenv shell  
    az login -u test@corp.onmicrosoft.com -p Welcome2022!  
    python stormspotter\stormcollector\sscollector.pyz cli  
    # This will generate a .zip file to upload in the frontend (127.0.0.1:9091)

###

### AzureHound

https://github.com/BloodHoundAD/AzureHound

###

    
    
    # You need to use the Az PowerShell and Azure AD modules:  
    $passwd = ConvertTo-SecureString "Welcome2022!" -AsPlainText -Force  
    $creds = New-Object System.Management.Automation.PSCredential ("test@corp.onmicrosoft.com", $passwd)  
    Connect-AzAccount -Credential $creds  
      
    Import-Module AzureAD\AzureAD.psd1  
    Connect-AzureAD -Credential $creds  
      
    # Launch AzureHound  
    . AzureHound\AzureHound.ps1  
    Invoke-AzureHound -Verbose  
      
    # Simple queries  
    ## All Azure Users  
    MATCH (n:AZUser) return n.name  
    ## All Azure Applications  
    MATCH (n:AZApp) return n.objectid  
    ## All Azure Devices  
    MATCH (n:AZDevice) return n.name  
    ## All Azure Groups  
    MATCH (n:AZGroup) return n.name  
    ## All Azure Key Vaults  
    MATCH (n:AZKeyVault) return n.name  
    ## All Azure Resource Groups  
    MATCH (n:AZResourceGroup) return n.name  
    ## All Azure Service Principals  
    MATCH (n:AZServicePrincipal) return n.objectid  
    ## All Azure Virtual Machines  
    MATCH (n:AZVM) return n.name  
    ## All Principals with the ‘Contributor’ role  
    MATCH p = (n)-[r:AZContributor]->(g) RETURN p  
      
    # Advanced queries  
    ## Get Global Admins  
    MATCH p =(n)-[r:AZGlobalAdmin*1..]->(m) RETURN p  
    ## Owners of Azure Groups  
    MATCH p = (n)-[r:AZOwns]->(g:AZGroup) RETURN p  
    ## All Azure Users and their Groups  
    MATCH p=(m:AZUser)-[r:MemberOf]->(n) WHERE NOT m.objectid CONTAINS 'S-1-5' RETURN p  
    ## Privileged Service Principals  
    MATCH p = (g:AZServicePrincipal)-[r]->(n) RETURN p  
    ## Owners of Azure Applications  
    MATCH p = (n)-[r:AZOwns]->(g:AZApp) RETURN p  
    ## Paths to VMs  
    MATCH p = (n)-[r]->(g: AZVM) RETURN p  
    ## Paths to KeyVault  
    MATCH p = (n)-[r]->(g:AZKeyVault) RETURN p  
    ## Paths to Azure Resource Group  
    MATCH p = (n)-[r]->(g:AZResourceGroup) RETURN p  
    ## On-Prem users with edges to Azure  
    MATCH  p=(m:User)-[r:AZResetPassword|AZOwns|AZUserAccessAdministrator|AZContributor|AZAddMembers|AZGlobalAdmin|AZVMContributor|AZOwnsAZAvereContributor]->(n) WHERE m.objectid CONTAINS 'S-1-5-21' RETURN p  
    ## All Azure AD Groups that are synchronized with On-Premise AD  
    MATCH (n:Group) WHERE n.objectid CONTAINS 'S-1-5' AND n.azsyncid IS NOT NULL RETURN n

  

### Azucar

https://github.com/nccgroup/azucar

    
    
    # You should use an account with at least read-permission on the assets you want to access  
    git clone https://github.com/nccgroup/azucar.git  
    PS> Get-ChildItem -Recurse c:\Azucar_V10 | Unblock-File  
      
    PS> .\Azucar.ps1 -AuthMode UseCachedCredentials -Verbose -WriteLog -Debug -ExportTo PRINT  
    PS> .\Azucar.ps1 -ExportTo CSV,JSON,XML,EXCEL -AuthMode Certificate_Credentials -Certificate C:\AzucarTest\server.pfx -ApplicationId 00000000-0000-0000-0000-000000000000 -TenantID 00000000-0000-0000-0000-000000000000  
    PS> .\Azucar.ps1 -ExportTo CSV,JSON,XML,EXCEL -AuthMode Certificate_Credentials -Certificate C:\AzucarTest\server.pfx -CertFilePassword MySuperP@ssw0rd! -ApplicationId 00000000-0000-0000-0000-000000000000 -TenantID 00000000-0000-0000-0000-000000000000  
      
    # resolve the TenantID for an specific username  
    PS> .\Azucar.ps1 -ResolveTenantUserName user@company.com

### MicroBurst

https://github.com/NetSPI/MicroBurst

    
    
    Import-Module .\MicroBurst.psm1  
    Import-Module .\Get-AzureDomainInfo.ps1  
    Get-AzureDomainInfo -folder MicroBurst -Verbose  
    

### PowerZure

https://github.com/hausec/PowerZure

    
    
    Connect-AzAccount  
    ipmo C:\Path\To\Powerzure.psd1  
    Get-AzureTarget  
      
    # Reader  
    $ Get-Runbook, Get-AllUsers, Get-Apps, Get-Resources, Get-WebApps, Get-WebAppDetails  
      
    # Contributor  
    $ Execute-Command -OS Windows -VM Win10Test -ResourceGroup Test-RG -Command "whoami"  
    $ Execute-MSBuild -VM Win10Test  -ResourceGroup Test-RG -File "build.xml"  
    $ Get-AllSecrets # AllAppSecrets, AllKeyVaultContents  
    $ Get-AvailableVMDisks, Get-VMDisk # Download a virtual machine's disk  
      
    # Owner  
    $ Set-Role -Role Contributor -User test@contoso.com -Resource Win10VMTest  
      
    # Administrator  
    $ Create-Backdoor, Execute-Backdoor

![](https://gitee.com/fuli009/images/raw/master/public/20230320091358.png)  

 **
**[2023零基础白帽+中级进阶渗透学习路线](http://mp.weixin.qq.com/s?__biz=MzI3NTcwNTQ2Mg==&mid=2247486851&idx=1&sn=c0259da9ccb93bf8dbc0ebabec9bdef9&chksm=eb01f69adc767f8c68eff87a81121eebc716058d9c2f654822a2ebaf1ee9d5b9b1e06f8c8076&scene=21#wechat_redirect)****
**知识星球**![](https://gitee.com/fuli009/images/raw/master/public/20230320091359.png)![](https://gitee.com/fuli009/images/raw/master/public/20230320091358.png)
想成为凯文·米特尼克一模一样的黑客教父？不存在的想成为透透之神？醒吧看了无数入门视频，依然搞不到Shell不像看《安全帮Live》![](https://gitee.com/fuli009/images/raw/master/public/20230320091402.png)

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

