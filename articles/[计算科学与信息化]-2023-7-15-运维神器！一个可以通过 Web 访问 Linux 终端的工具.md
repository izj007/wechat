#  运维神器！一个可以通过 Web 访问 Linux 终端的工具

[ 计算科学与信息化 ](javascript:void\(0\);)

**计算科学与信息化** ![]()

微信号 CSandIN

功能介绍 计算机网络与安全、人工智能算法设计、计算机视觉、VR/AR技术、数据分析挖掘、编程开发以及各种软硬件技术知识分享。欢迎关注计算科学与信息化！

____

___发表于_

收录于合集

#计算机网络 344 个

#网络工程师 95 个

#linux 10 个

#web 1 个

#运维 3 个

  

**来自公众号： 开源技术专栏**

作者：OP-SRC

rtty 由客户端和服务端组成。客户端采用纯C实现，服务端采用 GO 语言实现，前端界面采用 vue 实现。使用 rtty 可以在任何地方通过 Web
访问您的设备的终端，通过 设备ID 来区分您的不同的设备。rtty 非常适合远程维护 Linux设备。  

# 特性

  * • 客户端 C 语言实现，非常小，适合嵌入式 Linux

    * •  **不支持 SSL** ：rtty(32K) + libev(56K)

    * •  **支持 SSL** ：+ libmbedtls(88K) + libmbedcrypto(241K) + libmbedx509(48k)

  * • 远程批量执行命令

  * • 支持SSL: openssl、mbedtls、CyaSSl(wolfssl)

  * • SSL 双向认证(mTLS)

  * • 非常方便的上传和下载文件

  * • 根据 设备ID 访问不同的设备

  * • 支持 HTTP 代理 访问您的设备的 Web

  * • 基于 `Xterm.js` 的全功能终端

  * • 部署简单，使用方便

# 演示

![](https://gitee.com/fuli009/images/raw/master/public/20230715091707.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230715091710.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230715091713.png)

# 部署服务端

### 安装依赖

    
    
    sudo apt install -y libev-dev libssl-dev      # Ubuntu, Debian  
    sudo pacman -S --noconfirm libev openssl      # ArchLinux  
    sudo yum install -y libev-devel openssl-devel # Centos

### 克隆 rtty 代码

    
    
    git clone --recursive https://github.com/zhaojh329/rtty.git

### 编译

    
    
    cd rtty && mkdir build && cd build  
    cmake .. && make install

### 将下面的参数替换为您自己的参数

    
    
    sudo rtty -I 'My-device-ID' -h 'your-server' -p 5912 -a -v -d 'My Device Description'

### 生成一个 token

    
    
      
    $ rttys token  
    Please set a password:******  
    Your token is: 34762d07637276694b938d23f10d7164

### 使用 token

    
    
      
    $rttys -t 34762d07637276694b938d23f10d7164

# 通过浏览器访问

使用 Web 浏览器访问您的服务器：http://your-server-host:5913，然后点击连接按钮。

或者直接连接设备，无需 Web 登录(需要在服务端配置设备白名单)

    
    
    http://your-server-host:5913/connect/devid1  
      
    http://your-server-host:5913/connect/devid2

### 从本地传输文件到远程设备

    
    
    rtty -R

### 从远程设备传输文件到本地

    
    
    rtty -S test.txt

  

# 传送门

项目地址：https://github.com/zhaojh329/rtty

网盘下载地址：https://pan.quark.cn/s/c3100155b7fa

\---END---

  

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

