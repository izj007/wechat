#  RPC绕过EDR的研究与落地

原创 M0nster3  [ T00ls安全 ](javascript:void\(0\);)

**T00ls安全** ![]()

微信号 T00lsSec

功能介绍
T00ls，中国最具影响力的网络安全社区，聚合安全领域最优秀的人群，低调研究潜心学习讨论各类网络安全知识，为推动中国网络安全进步与技术创新贡献力量！

____

___发表于_

收录于合集 ##T00ls月刊 3个

本文主要是根据之前写的一些RpcsDemo做一个解释说明。

## 1、前言

最近研究RPC在内网中的一些攻击面，主要是以红队视角来看，使用RPC协议有时候Bypass EDR等设备会有较好的效果，那么什么是RPC呢，RPC
代表“远程过程调用”，它不是 Windows 特定的概念。RPC
的第一个实现是在80年代在UNIX系统上实现的。这允许机器在网络上相互通信，它甚至被“用作网络文件系统（NFS）的基础”，其实简单的说就是它允许请求另一台计算机上的服务，本节内容主要是依靠Microsoft官方文档进行学习。

## 2、RPC结构相关概念

1、首先我们要理解RPC是如何进行通信的首先需要知道几个概念IDL文件，UUID，ACF文件IDL文件：为了统一客户端与服务端不同平台处理不同的实现，于是有了IDL语言。IDL文件由一个或多个接口定义组成，每一个接口定义都有一个接口头和一个接口体，接口头包含了使用此接口的信息(UUID和接口版本)，接口体包含了接口函数的原型相关细节查看。UUID：通常为一个16长度的标识符，具有唯一性，在Rpc通信模型中，UUID
提供对接口、管理器入口点向量或客户端对象等对象的唯一指定。ACF：(ACF) 的应用程序配置文件有两个部分： 接口标头，类似于 IDL
文件中的接口标头，以及一个 正文，其中包含适用于 IDL
文件的接口正文中定义的类型和函数的配置属性。2、调用过程RpcStringBindingCompose：需要先创建一个绑定句柄字符串。。RpcBindingFromStringBinding：通过绑定句柄字符串返回绑定句柄。3、存根分配和释放内存在编写RPC调用的时候，必须将函数MIDL_user_allocate和MIDL_user_free在项目的中定义。  
![](https://gitee.com/fuli009/images/raw/master/public/20230401130630.png)

## 3、相关攻击面

所有的Demo都在https://github.com/M0nster3/RpcsDemo码字不易，大家如果觉得有帮助奉献一下您的star，哈哈。

### 1、IOXID Resolver探测内网多网卡主机

我们发送一个IOXID的传输包，这个发送方式有很多种，我这里用的K8师傅的工具，用Wireshark抓包。![](https://gitee.com/fuli009/images/raw/master/public/20230401130637.png)上图中TCP的三个包就不用看了，就是很常见的TCP的三次握手，后四个包中可以如图看，主要关注的是最后一个包，前三个都是固定的，就是交互中用来协商版本之类的参数。1、先来构造第一个数据包，由于这个包是固定的可以直接Copy
Wireshark中的，如下图

    
    
    05000b03100000004800000001000000b810b810000000000100000000000100c4fefc9960521b10bbcb00aa0021347a00000000045d888aeb1cc9119fe808002b10486002000000

  
![](https://gitee.com/fuli009/images/raw/master/public/20230401130639.png)2、后续第二个是接收的数据包，直接将第三个包复制就可以

    
    
    050000031000000018000000010000000000000000000500

  
3、主要就是看我们如何剖析最后一个包，将他接收过来并且进行一个分割输出，首先我们是想要枚举他的多网卡信息，和主机信息。我们对数据包进行一个分割。是从/0x07/0x00/进行分割。![](https://gitee.com/fuli009/images/raw/master/public/20230401130640.png)结束的是在0x09/0x00/0xff这一块结束的,把我们接受的数据进行一个分割。![](https://gitee.com/fuli009/images/raw/master/public/20230401130642.png)相关代码：https://github.com/M0nster3/RpcsDemo/blob/main/OXIDINterka_network_card/OXID.go效果图![](https://gitee.com/fuli009/images/raw/master/public/20230401130644.png)

### 2、RPC SMB

RPC还可以通过不同的协议进行一个访问，例如通过SMB协议传输的RPC服务就可以通过管道进行访问，假如在做项目的时候又有个域凭证就可以进行一些RPC接口的一个调用，比较好用的一个工具是rpcclient，它是执行客户端
MS-RPC
功能的工具。相关命令的一些总结我发在了https://github.com/M0nster3/RpcsDemo/blob/main/RPC%20over%20SMB/MS-
RPC.md中，大家有需要可以去提取。

### 3、MS-SAMR的那些事

该协议支持包含用户和组的帐户存储或目录的管理功能，简单来说就是该协议主要是对Windows用户以及用户组的一些相应操作，例如添加用户，用户组等操作。官方参考.

#### 1）添加本地用户

调用的API SamrCreateUser2InDomain()可以创建一个用户.

    
    
    long SamrCreateUser2InDomain(  
       [in] SAMPR_HANDLE DomainHandle,  
       [in] PRPC_UNICODE_STRING Name,  
       [in] unsigned long AccountType,  
       [in] unsigned long DesiredAccess,  
       [out] SAMPR_HANDLE* UserHandle,  
       [out] unsigned long* GrantedAccess,  
       [out] unsigned long* RelativeId  
     );

  
在创建用户的时候通过分档来看，不能直接创建到内置域（Builtin）中，需要先创建到账户域（账户）中，如下图。关于内置域和账户域的相关内容可以参考官方链接.其实简单来说就是，账户域内的用户只能访问该账户所在计算机的资源，而内置域中的账户可以访问域的资源。

![](https://gitee.com/fuli009/images/raw/master/public/20230401130645.png)

由于使用SamrCreateUser2InDomain创建的账户存在禁用标识位，我们先需要为它Set一个属性，来清除禁用标识位。然后才可以将其加入到所在的内置域中。使用SamrSetInformationUser()
这个API为它设置。

    
    
    long SamrSetInformationUser(  
       [in] SAMPR_HANDLE UserHandle,  
       [in] USER_INFORMATION_CLASS UserInformationClass,  
       [in, switch_is(UserInformationClass)]   
         PSAMPR_USER_INFO_BUFFER Buffer  
     );

  
编写脚本有两种方式一种是直接调用MS-
SAMR协议去直接创建一个用户，微软官方给了IDL，将其编译，然后构造，这种方式调用起来比较麻烦，另一种是使用神器mimikatz打包好的包，samlib来进行调用，调用的时候将前面的samr改成sam就可以.参考微软给的官方例子.可以按照这个例子依次构造

![](https://gitee.com/fuli009/images/raw/master/public/20230401130647.png)

首先先求出来账户域Account和内置域的Builts的SID为后续添加账户以及加入到内置域中做准备。

![](https://gitee.com/fuli009/images/raw/master/public/20230401130648.png)

然后获取域对象的句柄，然后为域对象添加用户,并且清除禁用标识位，关键代码。

![](https://gitee.com/fuli009/images/raw/master/public/20230401130649.png)

到这里创建用户的准备工作就结束了，接下来，就是将用户添加到组里面，用到SamAddMemberToAlias.

    
    
    long SamrAddMemberToAlias(  
       [in] SAMPR_HANDLE AliasHandle,  
       [in] PRPC_SID MemberId  
     );

  
相应的Demo：https://github.com/M0nster3/RpcsDemo/blob/main/MS-
SAMR/AddUser/AddUser/main.c

![](https://gitee.com/fuli009/images/raw/master/public/20230401130650.png)

#### 2) Change Ntlm

调用的关键API在SamrChangePasswordUser .当我们获取到了用户名，以及密码NTLMhash，则可以使用这个API将用户的密码修改了。

    
    
    long SamrChangePasswordUser(  
       [in] SAMPR_HANDLE UserHandle,  
       [in] unsigned char LmPresent,  
       [in, unique] PENCRYPTED_LM_OWF_PASSWORD OldLmEncryptedWithNewLm,  
       [in, unique] PENCRYPTED_LM_OWF_PASSWORD NewLmEncryptedWithOldLm,  
       [in] unsigned char NtPresent,  
       [in, unique] PENCRYPTED_NT_OWF_PASSWORD OldNtEncryptedWithNewNt,  
       [in, unique] PENCRYPTED_NT_OWF_PASSWORD NewNtEncryptedWithOldNt,  
       [in] unsigned char NtCrossEncryptionPresent,  
       [in, unique] PENCRYPTED_NT_OWF_PASSWORD NewNtEncryptedWithNewLm,  
       [in] unsigned char LmCrossEncryptionPresent,  
       [in, unique] PENCRYPTED_LM_OWF_PASSWORD NewLmEncryptedWithNewNt  
     );

  
这这里遇到了一个坑，就是只用旧的Ntlm就行修改而不对LmCrossEncryptionPresent和NewLmEncryptedWithNewNt进行传参，则会输出一个C000017F的错误，如下图。

![](https://gitee.com/fuli009/images/raw/master/public/20230401130651.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230401130652.png)

我去查看一下这个错误发现是客户端使用当前密码LM hash作为加密密钥请求返回，不清楚为什么不能用当前的密码LM hash，就改了一个其他的LM
hash,关键代码。

![](https://gitee.com/fuli009/images/raw/master/public/20230401130653.png)

接下来就是编写POC，我在这里使用微软官方的提供的IDL进行编译，提供了我们需要的所有包，在我们编译好，生成exe的时候会有很多错误，直接将其都注释就好。根据RPC的调用过程首先需要进行RPC的绑定

    
    
    RPC_STATUS RpcStringBindingComposeW(  
      RPC_WSTR ObjUuid,  
      RPC_WSTR ProtSeq,  
      RPC_WSTR NetworkAddr,  
      RPC_WSTR Endpoint,  
      RPC_WSTR Options,  
      RPC_WSTR *StringBinding  
    );

  
其中的ObjUuid可以直接在提供的IDL中找到，如下图，但是发现这个例子有没有这个都可以，最主要的必须定义一个命名管道端点 \PIPE\samr。关键代码

![](https://gitee.com/fuli009/images/raw/master/public/20230401130654.png)

绑定了之后接下来就是构造SamrChangePasswordUser,如果我们不熟悉MS-SAMR我们可以倒着堆整个调用流程。

    
    
     long SamrChangePasswordUser(  
       [in] SAMPR_HANDLE UserHandle,  
       [in] unsigned char LmPresent,  
       [in, unique] PENCRYPTED_LM_OWF_PASSWORD OldLmEncryptedWithNewLm,  
       [in, unique] PENCRYPTED_LM_OWF_PASSWORD NewLmEncryptedWithOldLm,  
       [in] unsigned char NtPresent,  
       [in, unique] PENCRYPTED_NT_OWF_PASSWORD OldNtEncryptedWithNewNt,  
       [in, unique] PENCRYPTED_NT_OWF_PASSWORD NewNtEncryptedWithOldNt,  
       [in] unsigned char NtCrossEncryptionPresent,  
       [in, unique] PENCRYPTED_NT_OWF_PASSWORD NewNtEncryptedWithNewLm,  
       [in] unsigned char LmCrossEncryptionPresent,  
       [in, unique] PENCRYPTED_LM_OWF_PASSWORD NewLmEncryptedWithNewNt  
     );

  
根据上面的图，以及相关的官方文档，我们发现我们现在就需要传入一个UserHandle用户句柄，其他的就是我们需要输入的NT
hash，以及我们需要修改的新的NT hash，那么这个UserHandle需要从哪里获取呢。这时候可以翻看官方文档。发现一个API
SamrOpenUser()如下，可以为我们提供我们需要的Userhandle，这个API意思就是通过RID来获取用户句柄。

    
    
    long SamrOpenUser(  
       [in] SAMPR_HANDLE DomainHandle,  
       [in] unsigned long DesiredAccess,  
       [in] unsigned long UserId,  
       [out] SAMPR_HANDLE* UserHandle  
     );

  
继续查看这个API需要什么参数，需要一个域的句柄，所需要的访问权限查看文档，如下图，由于我们是要实现修改密码，所以我们需要一个指定修改用户密码的能力USER_CHANGE_PASSWORD，最后还需要一个RID。

![](https://gitee.com/fuli009/images/raw/master/public/20230401130656.png)

通过上面的分析，我们现在需要两个参数，一个参数是DomainHandle，另一个就是UserId.继续翻看文档发现这样一个API
SamrLookupNamesInDomainSamrLookupNamesInDomain如下就是将我们输入的用户名转化为RID，输出一个RID号，到这里我们上面所需要的两个参数中的UserId就找到了。这里需要的两个参数就是我们输入的用户名，还有和上面SamrOpenUser通向需要的的
DomainHandle。

    
    
    long SamrLookupNamesInDomain(  
       [in] SAMPR_HANDLE DomainHandle,  
       [in, range(0,1000)] unsigned long Count,  
       [in, size_is(1000), length_is(Count)]   
         RPC_UNICODE_STRING Names<li>,  
       [out] PSAMPR_ULONG_ARRAY RelativeIds,  
       [out] PSAMPR_ULONG_ARRAY Use  
     );

  
我们继续找返现 SamrOpenDomain这个API，通过SID号可以输出我们需要的域对象句柄。

    
    
     long SamrOpenDomain(  
       [in] SAMPR_HANDLE ServerHandle,  
       [in] unsigned long DesiredAccess,  
       [in] PRPC_SID DomainId,  
       [out] SAMPR_HANDLE* DomainHandle  
     );

  
到这里SamrOpenUser这个API所需要的条件就找全了。我们需要继续为SamrOpenDomain寻找它所需要输入的内容，服务器句柄，SID号这一块可以使用SamrLookupDomainInSamServer来获取我们需要的SID.这个需要一个内置域的名称，也就是上面上面添加本地用户中提到的获取内置域的名称就可以，这里填写“Builtin”以及一个服务器句柄。

    
    
    long SamrLookupDomainInSamServer(  
       [in] SAMPR_HANDLE ServerHandle,  
       [in] PRPC_UNICODE_STRING Name,  
       [out] PRPC_SID* DomainId  
     );  
    

获取服务器对象的句柄使用到的API SamrConnect5。这个API 会返回服务器对象的句柄,需要我们填入我们的服务器，直接填写机器名称就可以。

    
    
     long SamrConnect5(  
       [in, unique, string] PSAMPR_SERVER_NAME ServerName,  
       [in] unsigned long DesiredAccess,  
       [in] unsigned long InVersion,  
       [in] [switch_is(InVersion)] SAMPR_REVISION_INFO* InRevisionInfo,  
       [out] unsigned long* OutVersion,  
       [out, switch_is(*OutVersion)] SAMPR_REVISION_INFO* OutRevisionInfo,  
       [out] SAMPR_HANDLE* ServerHandle  
     );

总结一下：1、我们首先利用 SamrConnect5
获取服务器句柄。2、利用获取到的服务器句柄经过SamrLookupDomainInSamServer获取服务器SID,。3、接着利用第一步中获取的服务器句柄以及第二步中的SID利用SamrOpenDomain获取域句柄4、接下来利用获取到的域句柄利用SamrLookupNamesInDomain获取RID号5、接着利用第四步中的RID以及第三步中的域句柄利用SamrOpenUser
API获取用户句柄6、最后利用用户句柄以及之前的NT hash和需要修改的Nt
Hash调用SamrChangePasswordUser修改密码。想要修改的Nt hash 可以使用 python2 。

    
    
    import hashlib,binascii  
    print binascii.hexlify(hashlib.new("md4", "123456".encode("utf-16le")).digest())

效果图：

![](https://gitee.com/fuli009/images/raw/master/public/20230401130657.png)

完整的Demo：https://github.com/M0nster3/RpcsDemo/blob/main/MS-
SAMR/ChangeNTLM/ChangePass/main.c

### 4、MS-TSCH

[MS -
TSCH]：任务计划程序服务远程协议,用于注册和配置任务以及查询远程计算机上运行的任务的状态。顾名思义就是利用这个API可以操纵计划任务。https://learn.microsoft.com/zh-
cn/openspecs/windows_protocols/ms-
tsch/d1058a28-7e02-4948-8b8d-4a347fa64931直接来看相关API
SchRpcRegisterTask直接向服务器注册一个任务，关键的两个参数一个是我们创建服务的路径，另一个就是定义计划任务的xml。

    
    
     HRESULT SchRpcRegisterTask(  
       [in, string, unique] const wchar_t* path,  
       [in, string] const wchar_t* xml,  
       [in] DWORD flags,  
       [in, string, unique] const wchar_t* sddl,  
       [in] DWORD logonType,  
       [in] DWORD cCreds,  
       [in, size_is(cCreds), unique] const TASK_USER_CRED* pCreds,  
       [out, string] wchar_t** pActualPath,  
       [out] PTASK_XML_ERROR_INFO* pErrorInfo  
     );

  
奇怪的是我们在编写的时候总是提示我们缺少参数，如下图,我们缺少一个句柄，这个句柄就是我们写RPC时候的一个绑定句柄，这个Demo写起来就简单多了，不需要之前那么多要求，只要配置一个RPC绑定就可以了。

![](https://gitee.com/fuli009/images/raw/master/public/20230401130658.png)

本来以为很简单直接写一个绑定就可以，没想到调用之前的绑定，发现总是失败，后来查找github别人的源码发现需要多一步验证，需要实现RpcBindingSetAuthInfoExA,真是吐了。

    
    
    RPC_STATUS RpcBindingSetAuthInfoExA(  
      RPC_BINDING_HANDLE       Binding,  
      RPC_CSTR                 ServerPrincName,  
      unsigned long            AuthnLevel,  
      unsigned long            AuthnSvc,  
      RPC_AUTH_IDENTITY_HANDLE AuthIdentity,  
      unsigned long            AuthzSvc,  
      RPC_SECURITY_QOS         *SecurityQos  
    );

  
关键代码

![](https://gitee.com/fuli009/images/raw/master/public/20230401130659.png)

效果图：

![](https://gitee.com/fuli009/images/raw/master/public/20230401130700.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230401130701.png)

相关代码：https://github.com/M0nster3/RpcsDemo/blob/main/MS-
TSCH_DESK/RPCDESK/RPCDESK/main.c

### 5、MS-SCMR

[MS - SCMR]：服务控制管理器远程协议.指定服务控制管理器远程协议，用于远程管理服务控制管理器 (SCM)，这是一个启用服务配置和服务程序控制的
RPC
服务器。其实就是一个管理服务的一个RPC协议。需要调用ROpenSCManagerA、RCreateServiceA也可以创建服务，除了这个之外还可以查看很多文档，还有许多API来使用。

    
    
     DWORD ROpenSCManagerA(  
       [in, string, unique, range(0, SC_MAX_COMPUTER_NAME_LENGTH)]   
         SVCCTL_HANDLEA lpMachineName,  
       [in, string, unique, range(0, SC_MAX_NAME_LENGTH)]   
         LPSTR lpDatabaseName,  
       [in] DWORD dwDesiredAccess,  
       [out] LPSC_RPC_HANDLE lpScHandle  
     );
    
    
    DWORD RCreateServiceA(  
       [in] SC_RPC_HANDLE hSCManager,  
       [in, string, range(0, SC_MAX_NAME_LENGTH)]   
         LPSTR lpServiceName,  
       [in, string, unique, range(0, SC_MAX_NAME_LENGTH)]   
         LPSTR lpDisplayName,  
       [in] DWORD dwDesiredAccess,  
       [in] DWORD dwServiceType,  
       [in] DWORD dwStartType,  
       [in] DWORD dwErrorControl,  
       [in, string, range(0, SC_MAX_PATH_LENGTH)]   
         LPSTR lpBinaryPathName,  
       [in, string, unique, range(0, SC_MAX_NAME_LENGTH)]   
         LPSTR lpLoadOrderGroup,  
       [in, out, unique] LPDWORD lpdwTagId,  
       [in, unique, size_is(dwDependSize)]   
         LPBYTE lpDependencies,  
       [in, range(0, SC_MAX_DEPEND_SIZE)]   
         DWORD dwDependSize,  
       [in, string, unique, range(0, SC_MAX_ACCOUNT_NAME_LENGTH)]   
         LPSTR lpServiceStartName,  
       [in, unique, size_is(dwPwSize)]   
         LPBYTE lpPassword,  
       [in, range(0, SC_MAX_PWD_SIZE)]   
         DWORD dwPwSize,  
       [out] LPSC_RPC_HANDLE lpServiceHandle  
     );

  
通过创建的服务是没有开启的，这个时候我们就需要一个开启的API
RStartServiceA,准备好了所有的东西，就可以开始编写Demo。相关Demo和之前的一样搞就可以了，这里写几个注意的点。1、当我们使用官方给的IDL编写的时候有很多重命名，我们直接注释就可以，还有一些我们代码中可能用不到的方法，但是由于是使用官方的IDL编译的，所以需要我们实现一下。2、创建服务的时候只能直接将我们的EXE作为服务启动，因为不是所有程序都可以作为服务的方式运行，作为服务运行需要能返回运行情况等信息，所以有的程序添加后会，这里我提供一个方法，就是使用微软官方的程序srvany.exe1）首先将srvany.exe添加到服务中并且启动。2）将我们要执行的内容路径放入到注册表中

    
    
    reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ServiceName\Parameters /v AppDirectory /t REG_SZ /d "c:\" /f

  
3）然后将程序放入注册表

    
    
    reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ServiceName\Parameters /v Application /t REG_SZ /d "c:\xxx.exe" /f  
      
    reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ServiceName\Parameters /v AppParameters /t REG_SZ /d "如果程序需要参数则填在这里，如果不需要，清空这段文字或者整行" /f

  
效果图：

![](https://gitee.com/fuli009/images/raw/master/public/20230401130702.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230401130703.png)

这里我们将我们的shellcode执行一下,添加注册表的时候需要将servicesname改为你添加任务的名字。

![](https://gitee.com/fuli009/images/raw/master/public/20230401130705.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230401130706.png)

而且这里还是system权限。

#### 6、Seclogon Dump Lsass

这个是splinter_code
这个师傅发现的.它的原理主要是，不直接调用OpenProcess去打开进程对象，而是利用已经打开的Lsass进程句柄，从而绕过检测，然后利用RpcImpersonateClient尝试使用PID做一个调用者的伪造。关键细节可以看这个师傅的博客说的很详细了：效果图：

![](https://gitee.com/fuli009/images/raw/master/public/20230401130707.png)

需要将我们的第一步-t 1的提取出来，不然直接使用-t 2解密之后会被杀软杀了。

## 参考

https://splintercod3.blogspot.com/p/the-hidden-side-of-seclogon-
part-3.htmlhttps://splintercod3.blogspot.com/p/the-hidden-side-of-seclogon-
part-2.html https://www.anquanke.com/post/id/245482#h3-5
[https://mp.weixin.qq.com/s/ByW_tsipzLGKa9mUBImCqA](https://mp.weixin.qq.com/s?__biz=MzAwNTI1NDI3MQ==&mid=2649618174&idx=1&sn=0b754089976de73c799e7bf72c807e1b&scene=21#wechat_redirect)  
 **原文链接：**  

> https://www.t00ls.com/articles-68609.html

  

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

