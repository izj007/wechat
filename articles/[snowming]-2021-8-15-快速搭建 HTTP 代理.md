这篇文章更多说一些 Linux 服务器上习惯性的东西。

本文的场景是：比如我现在用公司的青云云服务器（慢的一批），我自己有个阿里云云服务器（高贵的香港区域），但是我必须把环境搭建在青云云服务器（为了持久的保存），毕竟阿里云云服务器我是按量计费一会儿就关掉了。

所以很直接的想要用阿里云作为代理，毕竟阿里云香港、这样我的青云上海也可以呼吸一下外面的新鲜空气了。


但是首先我想说一下一些习惯，当我拿到一个云服务器/VPS我会做什么？


# 0x01 history 隐匿


我不想留下任何历史命令。所以我执行的第一条命令会是：

```
#最前面有一个空格，空格的原因参考：http://blog.leanote.com/post/snowming/ad293ff667db
 unset HISTORY HISTFILE HISTSAVE HISTZONE HISTORY HISTLOG; export HISTFILE=/dev/null; export HISTSIZE=0; export HISTFILESIZE=0
```

然后一般我会在 screen 里面进行一切操作，screen 有什么好处呢?

1. 会话的持续性
2. screen 里面的 history 不会在 screen 外面被看到

# 0x02 snap 命令

[Snap常用命令总结](https://www.maixj.net/ict/snap-cmd-21775)

可以用 snap 下载的软件优先用 snap 处理，然后在使用的时候搭配 `alias` 命令即可。


# 0x03 超快速搭建 HTTP 代理服务

使用 `tinyproxy` 镜像：

```
docker pull endoffight/tinyproxy


docker run -d --name='tinyproxy' -p 7777:8888 endoffight/tinyproxy ANY
# 或
docker run -d --name='tinyproxy' -p 7777:8888 endoffight/tinyproxy 8.8.8.8
```

- 那么代理端口就是7777了
- `ANY` 的意思是所有 IP 都可以建立连接
- `ANY` 使用acl语法，所以 10.103.0.100 192.168.1.22/16都是可以的
- 保险的运行方式是先找到自己的 IP 地址，使用http://www.ip138.com/，然后把 ANY 改为自己的 IP，比如自己的家里的地址为8.8.8.8，这样可以限制可建立连接的 IP

# 0x04 使用 HTTP 代理

curl：

![title](https://leanote.com/api/file/getImage?fileId=5eb8c496ab6441030e0143d3)


然后就可以通过 wget、proxychains 等使用这个自建的 http proxy 了。或者 linux 直接设置全局代理。


# 0x05 TOR MEEK 插件


![title](https://leanote.com/api/file/getImage?fileId=5eb93d25ab6441053102667f)



![title](https://leanote.com/api/file/getImage?fileId=5eb93d4cab6441030e026935)


-----

上面的很多东西，都是整合自我以前的博文，欣慰的是这个博客写到今天，已具备字典这样的工具属性了，我平时用什么也会经常翻翻。



------------------------


参考文档：

1. [tinyproxy docker 镜像](https://hub.docker.com/r/endoffight/tinyproxy)，Docker Hub
2. [ProxyChains 教程](https://linuxhint.com/proxychains-tutorial/)
3. [wget 和 curl 设置代理服务器的命令](https://blog.csdn.net/huzhenwei/article/details/4369027)，CSDN，huzhenwei
4. [玩转 VPS 之快速搭建 HTTP 代理](https://blog.phpgao.com/vps_tinyproxy.html)，老高的技术博客，老高，2019-08-31
