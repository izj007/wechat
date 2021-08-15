# 0x01 环境搭建


**调整内核版本：**


在 ALiyun 上面开了一台 centos7.6，然后把内核版本替换为与目标内核版本一致：



```
wget https://buildlogs.centos.org/c7.1810.u.x86_64/kernel/20190729174341/3.10.0-957.27.2.el7.x86_64/kernel-3.10.0-957.27.2.el7.x86_64.rpm
wget https://buildlogs.centos.org/c7.1810.u.x86_64/kernel/20190729174341/3.10.0-957.27.2.el7.x86_64/kernel-headers-3.10.0-957.27.2.el7.x86_64.rpm
wget https://buildlogs.centos.org/c7.1810.u.x86_64/kernel/20190729174341/3.10.0-957.27.2.el7.x86_64/kernel-devel-3.10.0-957.27.2.el7.x86_64.rpm
yum -y localinstall kernel-3.10.0-957.27.2.el7.x86_64.rpm
yum -y localinstall kernel-devel-3.10.0-957.27.2.el7.x86_64.rpm
yum -y localinstall kernel-headers-3.10.0-957.27.2.el7.x86_64.rpm
grub2-set-default "CentOS Linux (3.10.0-957.27.2.el7.x86_64) 7 (Core)"
reboot
```


![title](https://leanote.com/api/file/getImage?fileId=5eba6b7fab64410531052dff)


**获取 www 权限：**


```
yum -y update
yum -y install httpd
yum -y install php
systemctl start httpd.service
```

![title](https://leanote.com/api/file/getImage?fileId=5eba73b8ab6441030e053f2f)


向默认目录 `/var/www/html` 下写入一个 index.html，访问发现 apache 服务起成功了：


![title](https://leanote.com/api/file/getImage?fileId=5eba7459ab64410531053e02)


往 `/var/www/html` web 目录下写入一个一句话木马（PHP）:


![title](https://leanote.com/api/file/getImage?fileId=5eba76ecab64410531054241)

但是我的阿里云服务器有一个杀不掉的 watchdog，导致我菜刀/C刀/antsword 连上之后不一会儿就 `与服务器的连接被重置`（禁我的 IP 等）：

![title](https://leanote.com/api/file/getImage?fileId=5eba7cafab6441030e054f13)

还有安骑士：

![title](https://leanote.com/api/file/getImage?fileId=5eba84eeab64410531055bd8)

换冰蝎：

![title](https://leanote.com/api/file/getImage?fileId=5eba7f65ab6441030e0553f8)

emmm，也不行。阿里云666，但是不想挂个大马就完事。尝试挂阿里云 IP 继续搞。


我尝试过在这个 centos 主机环境下开代理，但是 docker 乱七八糟。centos 真用不习惯，我还是习惯用 ubuntu，于是开了个同内网环境中的 ubuntu：

![title](https://leanote.com/api/file/getImage?fileId=5eba8c32ab6441030e056caf)


因为一会儿我要跟 Proxifier 搭配用，所以使用 [esocks](https://github.com/fengyouchao/esocks) 这个项目在这个 ubuntu 主机上开个 socks5 代理:


![title](https://leanote.com/api/file/getImage?fileId=5eba8ce2ab6441030e056e17)


Proxifier 搞好了，终于挂上了阿里云作为全局代理：

![title](https://leanote.com/api/file/getImage?fileId=5eba8df0ab64410531056db8)

然后就可以畅爽使用冰蝎了：

![title](https://leanote.com/api/file/getImage?fileId=5eba9145ab644105310573c2)


至此环境搭建 OK。

# 0x02 尝试提权

打算尝试用这两个 CVE 的 EXP 来提权：

- CVE-2016-0728 pp_key
- CVE-2014-0038 timeoutpwn


![title](https://leanote.com/api/file/getImage?fileId=5eba72ddab64410531053b59)
![title](https://leanote.com/api/file/getImage?fileId=5eba72faab6441030e053df6)



先试一下 [CVE-2016-0728](https://github.com/SecWiki/linux-kernel-exploits/tree/master/2016/CVE-2016-0728)：

```
gcc cve-2016-0728.c -o exp.elf -lkeyutils -Wall /root/linux-kernel-exploits/2016/CVE-2016-0728/cve-2016-0728/keyutils.h
```

![title](https://leanote.com/api/file/getImage?fileId=5ebabc87ab6441030e05b857)

用的是交互式 shell 但是没提成功，另一个洞也没提成功：

![title](https://leanote.com/api/file/getImage?fileId=5ebac852ab6441030e05cd1b)

到这里也不讲究了，直接加了个普通用户提的。但是这些提权漏洞都是随缘，运气不好没成功。

太困了就这样吧。

----------------------

# 参考文档：

1. [CentOS7--手动升级内核到指定版本](https://blog.csdn.net/jobbofhe/article/details/105045494)：
2. [Linux提权-依赖exp篇 （第二课）.pdf](https://github.com/Micropoor/Micro8/blob/master/Linux%E6%8F%90%E6%9D%83-%E4%BE%9D%E8%B5%96exp%E7%AF%87%20%EF%BC%88%E7%AC%AC%E4%BA%8C%E8%AF%BE%EF%BC%89.pdf)
3. https://github.com/SecWiki/linux-kernel-exploits
4. https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack/
5. https://github.com/xairy/kernel-exploits