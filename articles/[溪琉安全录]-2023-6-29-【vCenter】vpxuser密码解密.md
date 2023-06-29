#  【vCenter】vpxuser密码解密

原创 Chris  [ 溪琉安全录 ](javascript:void\(0\);)

**溪琉安全录** ![]()

微信号 gh_e1ff7c735f30

功能介绍 不定时更新网络安全相关的知识，同时也欢迎朋友们来投稿。

____

___发表于_

收录于合集

#凭据获取 3 个

#内网渗透 7 个

#实战攻防 9 个

#数据库 2 个

#vCenter 1 个

![](https://gitee.com/fuli009/images/raw/master/public/20230629201609.png)

****

**0x01 前言**

在内网渗透过程中，集权设备往往是Red
Team喜欢关注的地方，vCenter又有小域控的称号，当获取到vCenter之后权限不高，或者想要获取更多权限的时候，可以关注一下vpxuser账户，这很可能是安全基线检查中会被忽略到的一个点。本文记录了获取vCenter服务器权限之后，解密vpxuser账户明文密码的一个过程  

 **0x02 vpxuser  
**

  

vpxuser是ESXI和vCenter第一次连接时自动创建的高权限用户，vCenter使用OpenSSL密码库作为随机来源，每30天生成一个新的vpxuser密码，密码长度为32个字符，该用户是被vpxd进程创建的，用来管理ESXI和vCenter之间的通信  

  

vpxuser密文解密的过程如下：  

1.登录vCenter服务器的PostgresDB数据库

2.找到服务器的vpxuser加密的密码

3.对vpxuser密码逆向解密

4.利用CVE-2021-22015去提升到解密所需要的高权限（root权限忽略）

5.解密密码登录目标服务器  
 **0x03 获取密文**  

###  **PostgresDB的凭据**

首先需要找到含有数据库密码的凭据文件

  *   * 

    
    
    /etc/vmware-vpx/vcdb.propertiesC:\ProgramData\VMware\vCenterServer\cfg\vmware-vps\vcdb.properties

![](https://gitee.com/fuli009/images/raw/master/public/20230629201611.png)

使用数据库凭据登录数据库  

  * 

    
    
    /opt/vmware/vpostgres/current/bin/psql -d VCDB -U vc -w <Password>

  
![](https://gitee.com/fuli009/images/raw/master/public/20230629201612.png)

###  **vpxv_vms表相关信息**

该数据库存放了有关ESXi和vCenter的信息，其中vpxv_vms表是虚拟机的一些信息  
vpxv_vms字段如下

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    vmid namevmgroupidhostid                                configfilename                                         |        vmuniqueidresource_group_idmem_size_mbnum_vcpuboot_timesuspend_timepower_stateguest_osguest_familyguest_statememory_reservationmemory_overheadcpu_reservationdns_nameip_addressvmmware_tooltools_versionnum_nicnum_diskis_templateannotationsuspend_intervalaggr_commited_storage_spaceaggr_uncommited_storage_spaceaggr_unshared_storage_spacestorage_space_updated_time

查询vpx的虚拟机信息

  * 

    
    
    SELECT vmid,name,configfilename,guest_state,is_template FROM vpxv_vms;

 **![](https://gitee.com/fuli009/images/raw/master/public/20230629201614.png)  
**寻找正在运行的虚拟机

  * 

    
    
     SELECT vmid,name,configfilename,guest_state,is_template FROM vpxv_vms where guest_state='running';

 **![](https://gitee.com/fuli009/images/raw/master/public/20230629201615.png)  
**

 **获取vpxuser密文**  

从数据库获取vpxuser的密文

  * 

    
    
    SELECT user_name, password FROM vc.vpx_host;

 **![](https://gitee.com/fuli009/images/raw/master/public/20230629201617.png)  
**

 **0x04 加密算法**  

这里用到的是AES_256_CBC加密算法，还需要获取密钥key和iv初始化向量密钥key是保存在/etc/vmware-
vpx/ssl/symkey.dat文件中的![](https://gitee.com/fuli009/images/raw/master/public/20230629201618.png)  
  
iv是密文base64解密之后的前16位  
0746243b3b7d8f0c19bfd62d10bac3ea

![](https://gitee.com/fuli009/images/raw/master/public/20230629201620.png)

  

key和iv都确定之后 开始解密

需要解密的密文是base64之后去掉前16位之后的剩余部分

![](https://gitee.com/fuli009/images/raw/master/public/20230629201621.png)

成功解密  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230629201622.png)

  

使用python脚本解密如下

![](https://gitee.com/fuli009/images/raw/master/public/20230629201624.png)

  

公众号后台回复"vpxuser"获取工具链接  

 ****  
 ****

 **** **0x05 参考链接  
**

https://www.pentera.io/blog/information-disclosure-in-vmware-vcenter/  
https://www.pentera.io/blog/vscalation-cve-2021-22015-local-privilege-
escalation-in-vmware-vcenter-pentera-labs/  

https://mp.weixin.qq.com/s/Q91wk8BIUO5cAp6_7_pt1A  

![](https://gitee.com/fuli009/images/raw/master/public/20230629201625.png)

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

