#  IDEA动态调试WebLogic

原创 JC0o0l [ Zer0ne安全研究 ](javascript:void\(0\);)

**Zer0ne安全研究** ![]()

微信号 JC_SecNotes

功能介绍 信安新手的成长手札

____

___发表于_

收录于合集

## IDEA动态调试WebLogic

环境：Windows 10 + Windows7(192.168.52.181) + Idea + WebLogic12.2.1.4 + Java8102

## 0x01 安装weblogic

安装成功后，在domains下的bin目录下有个startWebLogic.cmd文件![](https://gitee.com/fuli009/images/raw/master/public/20230714174941.png)

## 0x02 配置被调试端

## 0x0201 添加调试参数

### 2.1.1 方式一

在startWebLogic.cmd文件中加参数：

  * 

    
    
    set JAVA_OPTIONS=-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=9999,server=y,suspend=n

### 2.1.1 方式二

修改 `user_projects\domains\<域名>` 的 bin 目录下面的 setDomainEnv.cmd 文件，在 `if
"%debugFlag%"=="true"`前加入：

  *   * 

    
    
    debugFlag=trueDEBUG_PORT=9999

## 0x0202 启动weblogic

双击startWebLogic.cmd启动weblogic，之后使用`netstat -ano|findstr 9999`可以看到9999端口的监听

![](https://gitee.com/fuli009/images/raw/master/public/20230714174942.png)

## 0x03 配置调试端

## 0x0301 打开wlserver作为项目

复制被调试环境中的weblogic目录到调试环境中

![](https://gitee.com/fuli009/images/raw/master/public/20230714174943.png)

启动idea,选择打开一个项目，选择wlserver这个目录

![](https://gitee.com/fuli009/images/raw/master/public/20230714174944.png)

## 0x0302 添加jar包

单纯的将server/lib目录设置为library，可能会遗漏一些jar包，比如weblogic12.2.1.4环境CVE-2020-14852要用到的`oracle.eclipselink.coherence.integrated.internal.cache.LockVersionExtractor`类在toplink-
grid.jar包中，但该jar包的路径为`<weblogic安装目录>\oracle_common\modules\oracle.toplink\toplink-
grid.jar`。

### 3.2.1 寻找所有的jar包

我这里用的是everything软件

![](https://gitee.com/fuli009/images/raw/master/public/20230714174946.png)

新建一个文件夹，将所有的jar包拷贝新建的文件夹中

![](https://gitee.com/fuli009/images/raw/master/public/20230714174947.png)

### 3.2.2 添加library

![](https://gitee.com/fuli009/images/raw/master/public/20230714174948.png)

选择刚刚的weblogic_jars

![](https://gitee.com/fuli009/images/raw/master/public/20230714174950.png)

添加完之后的显示：

![](https://gitee.com/fuli009/images/raw/master/public/20230714174951.png)

## 0x0303 添加调试配置

选择调试配置![](https://gitee.com/fuli009/images/raw/master/public/20230714174952.png)添加一个remote的配置

![](https://gitee.com/fuli009/images/raw/master/public/20230714174953.png)

这里需要输入Host和Port的值，确保被调试环境的端口能够被访问![](https://gitee.com/fuli009/images/raw/master/public/20230714174954.png)

## 0x0304 启动调试

点击Debug，如果成功则会出现`Connected to the target VM, address: '192.168.52.181:9999',
transport:
'socket'`类似的显示，没成功的话可以重新运行startWebLogic.cmd文件再去debug。![](https://gitee.com/fuli009/images/raw/master/public/20230714174955.png)

## 0x04 使用方法

## 0x0401 如何搜索类或函数

双击shift键后，勾选`Include non-project
items`，之后就可以在搜索栏里搜索了![](https://gitee.com/fuli009/images/raw/master/public/20230714174956.png)

参考资料：weblogic漏洞分析系列之调试环境搭建[1]

### References

`[1]` weblogic漏洞分析系列之调试环境搭建:
_https://cangqingzhe.github.io/2020/10/30/weblogic%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90%E7%B3%BB%E5%88%97%E4%B9%8B%E8%B0%83%E8%AF%95%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/_

  

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

