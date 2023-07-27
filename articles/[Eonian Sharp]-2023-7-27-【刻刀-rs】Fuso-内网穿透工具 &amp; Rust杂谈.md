#  【刻刀-rs】Fuso-内网穿透工具 & Rust杂谈

Enomothem  [ Eonian Sharp ](javascript:void\(0\);)

**Eonian Sharp** ![]()

微信号 Eonian_sharp

功能介绍 Eonian Sharp | 永恒之锋，专注APT框架、渗透测试攻击与防御的研究与开发，没有永恒的安全，但有永恒的正义之锋击破黑暗的不速之客。

____

___发表于_

收录于合集

#刻刀系列 8 个

#Rust 5 个

# 声明

该公众号分享的安全工具和项目均来源于网络，仅供学术交流， **请勿直接用于任何商业场合和非法用途**
。如用于其它用途，由使用者承担全部法律及连带责任，与工具作者和本公众号无关。

# 项目

https://github.com/editso/fuso

# 介绍

Fuso（ 扶桑），一款体积小, 快速, 稳定, 高效, 轻量的内网穿透, 端口转发工具，支持多连接,级联代理,传输加密。

# 使用

    
    
    1. 端口转发  
    fuc --forward-host xxx.xxx.xxx.xxx --forward-port  
       --forward-host: 转发到的地址  
       --forward-port: 转发到的端口  
       如: 转发流量到内网 10.10.10.4:3389  
       > fuc --forward-host 10.10.10.4 --forward-port 3389  
      
    2. socks5:  
    fuc --socks --su --s5p xxx --s5u xxx  
       --su: 可选的, 开启udp转发,   
       --s5p: 可选的, 认证密码, 默认不进行密码认证  
       --s5u 可选的, 认证账号, 默认账号 anonymous  
       --socks: 可选的, 开启socks5代理, 未指定--su的情况下不会转发udp  
       如: 开启udp转发与密码认证  
       > fuc --socks --su --s5p 123 --s5u socks  
       此时, 已开启udp转发,连接密码为 "123",账号为 "socks"  
      
    3. 指定穿透成功时访问的端口  
       fuc -b xxxx  
       -b | --visit-bind-port: 可选的, 默认随机分配  
       如: 访问外网端口 8888 转发到内网 80  
       > fuc --forward-port 80 -b 8888  
         
    4. 桥接模式 注意: 目前不能转发udp  
       fuc --bridge-listen xxxx --bridge-port xxx   
       --bridge-listen | --bl: 监听地址, 默认 127.0.0.1  
       --bridge-port | --bp: 监听端口, 默认不启用桥接  
       如: 开始桥接模式,并监听在9999端口, 本机ip地址为: 10.10.10.2  
       > fuc --bridge-listen 0.0.0.0 --bridge-port 9999 # 开启桥接  
       > fuc 10.10.10.2 9999 # 建立连接  
      
       级联:   
       > fuc --bridge-listen 0.0.0.0 --bridge-port 9999 # 第一级, IP: 10.10.10.2  
        > fuc --bridge-listen 0.0.0.0 --bridge-port 9991  10.10.10.2 9999 # 第二级, IP: 10.10.10.3  
         > fuc 10.10.10.3 9991 # 最终   
      
    5. 将连接信息通知到 Telegram 或其他  
       fus --observer "program:[arguments]"  
       --observer: 建立连接或断开连接时的钩子  
       如: 使用bash脚本将连接信息通知到tg  
       > fus --observer "/bin/bash:[telegram.sh]"  
      
    6. 指定客户端与服务端通信的端口  
       fuc --channel-port 8888 ...  
    
    
       --channel-port: 可选的, 客户端与服务端通信端口, 默认随机   ds

# Demo

![]()

# 武器锈化

"刻刀-rs"系列，即是使用Rust编写的工具，将工具尽可能的锈化（rust翻译为铁锈）。

目前为止，已经推荐了多款Rust语言编写的工具，日后将不断完善rust工具化到整个渗透测试体系中。

# Rust

关于Rust，在Rustscan一文中已经详细描述，可前往进行查看。[【刻刀-
rs】Rustscan现代化高速端口扫描工具](http://mp.weixin.qq.com/s?__biz=Mzg3NzUyMTM0NA==&mid=2247484625&idx=1&sn=551a12348ee037141d6f077f90893955&chksm=cf20f47ef8577d681424def048a81ace1ec7a0e1b41dee58655c6fda327e015d20d12f04b67a&scene=21#wechat_redirect)

最近，2023-07-24 使用 Rust 重写的InfluxDB 3.0，

InfluxDB 是一个开源的、分布式的时序数据库，用于存储和分析时间序列数据, 于 2013 年由 Brian Bondy 和 Nicholas
Zakhariev 创立。InfluxDB 最初是用 Go 语言编写的。2018 年，InfluxData 决定将 InfluxDB 重写为 Rust
语言。重写 InfluxDB 的主要原因是 Rust 是一种更快、更安全、更可靠的编程语言。重写 InfluxDB 的过程非常成功。InfluxDB 3.0
在性能、可靠性和安全性方面都有显著的改进,
现在是世界上最快、最可靠的时序数据库之一https://www.influxdata.com/blog/meet-founders-who-rewrote-
in-rust/

  

  

  

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

