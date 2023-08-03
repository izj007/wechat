#  【免杀工具】webshell免杀一键生成

原创 eikooc  [ 饼干安全区 ](javascript:void\(0\);)

**饼干安全区** ![]()

微信号 gh_c5d6786ebe26

功能介绍 专注网络安全知识分享

____

___发表于_

收录于合集 #工具分享 7个

### 0x00 前言

使用异或生成免杀webshell以及使用异或生成payload

经测试可以绕过主流防护软件webshell扫描以及杀毒软件检测

### 0x01 webshell生成

使用python3运行脚本文件，即可生成免杀webshell一句话

  * -m：webshell请求方式

  * -p：webshell密码

  * -o：webshell文件名

  * 

    
    
    python3 xorshell.py -m post -p pass -o c.php

![]()

执行之后会在当前目录下生成一个webshell文件

![]()

### 0x02 使用测试

将webshell使用D盾、安全狗、河马进行扫描

![]()

均无法扫描到webshell并且能够正常执行命令

接下来对web层面不怎么强的杀毒软件进行测试

360全家桶、电脑管家、火绒：  

![]()

VirusTotal、VirScan：

![]()

以上只是静态扫描

如果在执行命令时被WAF拦截了又该如何Bypass

使用以下项目对参数进行异或最终传入webshell执行

### 0x03 payload生成

以下是我们需要执行的payload

  * 

    
    
    pass=system(whoami);

使用脚本对参数进行异或

不需要对最后的分号进行异或

  * 

    
    
    python3 xorpass.py -e "system(whoami)"

![]()

将加密后的payload替换成我们想要执行的payload

记得将没有经过异或的分号带上

![]()

成功执行命令

再对ipconfig命令测试，进行异或加密

  * 

    
    
    python3 xorpass.py -e "system(ipconfig)"

![]()

将加密后的payload作为参数执行

![]()

成功执行命令

最终是否能够绕过WAF需要自行检测

### 0x04 下载地址

点击下方名片

回复关键字【20230803】获取下载地址

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

