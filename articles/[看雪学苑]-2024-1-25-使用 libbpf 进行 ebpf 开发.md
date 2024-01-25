#  使用 libbpf 进行 ebpf 开发

yfsec  [ 看雪学苑 ](javascript:void\(0\);)

**看雪学苑** ![]()

微信号 ikanxue

功能介绍 致力于移动与安全研究的开发者社区，看雪学院(kanxue.com)官方微信公众帐号。

____

___发表于_

  

    
    
    一  
    
    
      
    
    
    使用libbpf

`  
`

`bpf-bootstrap`是Linux ebpf技术的开发库，使用的要求是已经安装了ebpf的前置依赖包，并且需要CO-
RE，即内核编译选项默认内置支持了`CONFIG_DEBUG_INFO_BTF=y`这个开关用于控制BTF开启与关闭，检查这个文件`/sys/kernel/btf/vmlinux`是否存在且非空，如果非空说明BTF信息就已经开启且生成了。

  

    
    
    git clone https://github.com/libbpf/libbpf-bootstrap.git  
    git submodule update --init --recursive #拉取其他所有库  
    
    
      
    

成功执行完以上步骤后libbpf就成功拉取下来了。

  
进入到`examples/c`目录下,有几份demo代码，`minimal`是最小的框架，通过以下命令编译：

  

    
    
    sudo make minimal //编译程序  
    sudo ./minimal //运行程序 需要以root用户运行  
    
    
      
    

![]()  

  

成功将bpf加载到内核，可以看到这样就不需要太多繁琐的步骤，不需要再通过命令的方式手动将内核文件插入到内核中。只需执行一个程序即可。

  

  

    
    
    二  
    
    
      
    
    
    minimal代码分析

###  

### minimal代码内核部分分析

  

首先来分析一下minimal的代码，分为`minimal.c`和`minimal.bpf.c`前者为用户空间的代码
后者为内核空间的代码，先来分析内核空间的代码。

  

    
    
    #include <linux/bpf.h>  
    #include <bpf/bpf_helpers.h>  
      
    char LICENSE[] SEC("license") = "Dual BSD/GPL";  
      
    int my_pid = 0;  
    SEC("tp/syscalls/sys_enter_write")  //挂钩位置  
    int handle_tp(void *ctx)  
    {  
        //bpf_get_current_pid_tgid()获取内核的TGID  
        int pid = bpf_get_current_pid_tgid() >> 32;  
        if (pid != my_pid)  
            return 0;  
        bpf_printk("BPF triggered from PID %d.\n", pid);  
        return 0;  
    }  
    
    
      
    

程序声明了许可协议，这行代码定义了eBPF程序的许可证信息，使用SEC("license")指定了该程序的许可。这段信息通常包含程序的许可证类型，这有助于清楚地指明该程序的许可。

  
`SEC`是一个宏定义，`SEC`宏通常用于指定程序类型和相关的挂钩，宏中的信息通常可以填写以下内容：

  

跟踪点程序

  

可以设置为内核跟踪点，在Linux中提供了内核跟踪点，通过以下命令可以查询到。

  

    
    
    sudo ls cd /sys/kernel/debug/tracing/events   
    sudo cat /sys/kernel/debug/tracing/available_events  
    常见的写法有  
    SEC("tracepoint/syscalls/sys_enter_openat") 用来hook系统调用的sys_enter_openat()  
    
    
      
    

![]()

  

 **套接字程序**

  

这个很容易理解，直接就是`SEC("socket")`就行

  

 **XDP程序**

  

    
    
     SEC("xdp")  
    int xdp_example(struct __sk_buff *skb) {  
        // 在这里编写您的代码  
        return XDP_PASS;  
    }  
    
    
      
    

 **自定义程序**

  

    
    
     SEC("custom")  
    int custom_example(void *ctx) {  
        // 在这里编写您的代码  
        return 0;  
    }  
    
    
      
    

声明一个eBPF程序的段。

* * *

  

这段代码中，通过向`sys_enter_write`内核函数挂钩，同时hook到printf等函数的使用
通过`my_pid`限制挂钩等进程，只要当前进程向终端打印了数据，就会使用`bpf_printk()`进行打印，通过以下命令可以看到打印的信息：

  

    
    
    sudo cat /sys/kernel/debug/tracing/trace_pipe  
    
    
      
    

如下图所示：

  
![]()

###  

### minimal代码用户部分分析

  

来看看用户空间的代码，用户空间用于与内核进行交互，通过用户空间的代码可以间接控制内核。以下为`minimal`用户空间代码。

  

    
    
    #include <stdio.h>  
    #include <unistd.h>  
    #include <sys/resource.h>  
    #include <bpf/libbpf.h>  
    #include "minimal.skel.h"  
      
    static int libbpf_print_fn(enum libbpf_print_level level, const char *format, va_list args)  
    {  
        return vfprintf(stderr, format, args);  
    }  
      
    int main(int argc, char **argv)  
    {  
        struct minimal_bpf *skel;  
        int err;  
        libbpf_set_print(libbpf_print_fn);  
        //打开skeleton  
        skel = minimal_bpf__open();  
        if (!skel) {  
            fprintf(stderr, "Failed to open BPF skeleton\n");  
            return 1;  
        }  
        //将内核中设置的mypid设置成当前进程  
        skel->bss->my_pid = getpid();  
        //加载bpf 这一步之后成功加载到bpf程序  
        err = minimal_bpf__load(skel);  
        if (err) {  
            fprintf(stderr, "Failed to load and verify BPF skeleton\n");  
            goto cleanup;  
        }  
        //bpf附加到skeleton  
        err = minimal_bpf__attach(skel);  
        if (err) {  
            fprintf(stderr, "Failed to attach BPF skeleton\n");  
            goto cleanup;  
        }  
        printf("Successfully started! Please run `sudo cat /sys/kernel/debug/tracing/trace_pipe` "  
               "to see output of the BPF programs.\n");  
        for (;;) {  
            //重复打印小数点 触发hook函数  
            fprintf(stderr, ".");  
            sleep(1);  
        }  
    cleanup:  
        minimal_bpf__destroy(skel);  
        return -err;  
    }  
    
    
      
    

这段代码的主要作用是将ebpf程序加载到内核，分为了这几个步骤(打开并加载skeleton、附加skeleton(设置hook)、触发hook函数、结束后销毁skel)。  
  

代码中还设置了一个回调函数`libbpf_print_fn`，这个函数的作用是将ebpf程序中的所有debug信息显示到终端上，方便查看与调试。

  
`minimal.c`中通过skel指针可以和内核代码的数据进行交互，完成内核空间与用户空间的数据交互，将内核代码中的`my_pid`设置为当前进程。

  
与正常通过`clang`手动编译的不同的是，这份代码多了一步加载了`minimal.skel.h`头文件，这个文件是通过编译器生成出来的，可以看到在`example/c`目录下的代码都有引导这个文件`"[文件名].skel.h"`，并且也用到了类似于这样的函数`"[文件名]_bpf_open_and_load()"`，具体这段代码如何产生的，有兴趣可以自己翻一下makefile文件。

  
这样的头文件被称为ebpf骨架，骨架的作用是简化eBPF程序的开发，提供了一些通用的功能和结构，使得开发者能够更容易地创建、加载和管理eBPF程序。在`example/c`文件下创建项目为`[项目名称].c``[项目名称].bpf.c`通过以下命令生成skeleton文件。

  

    
    
    sudo cmake .   
    sudo make [项目名称]  
    sudo ./[文件名]  
    
    
      
    

### Skeleton代码介绍

  

打开`minimal.bpf.c`文件可以看到有如下结构体：

  
![]()  
  

为`maps``progs``links``bss`这几个结构体，分别描述了内核文件的所有对象。

  

    
    
    maps 是 libbpf中可以作为数据交换的结构体  
    progs 通常指代 BPF Program 对象的集合。在 BPF 中，Program 是执行具体功能的代码单元  通常为bpf程序的函数  
    links 通常指代 BPF Link 对象的集合。在 BPF 中，Link 用于将 BPF Program 关联到特定的 BPF Hook 或事件  
    bss 通常指代 BSS（Block Started by Symbol）段，这是程序中的一块内存，用于存储未初始化的全局或静态变量  
    
    
      
    

代码中还定义了`minimal_bpf__open``minimal_bpf__load``minimal_bpf__attach``minimal_bpf__destory`这几个函数。

  

  

    
    
    三  
    
    
      
    
    
    sockfilter程序分析

###  

### sockfilter用户空间代码分析

  

接下来看一个比较关心的问题，如何通过ebpf创建钩子获取并修改网络数据包，同样在`example/c`中有写到关于向socket进行挂钩的项目分别为`sockfilter.c``sockfilter.h``sockfilter.bpf.c`  
  

首先进行编译。

  

    
    
    sudo make sockfilter  
    sudo ./sockfilter  
    
    
      
    

运行后的结果如下所示，程序获得了回环网卡的所有网络信息。

  
![]()  
  

这份代码与`minimal`在流程上是有很大的区别的，依然是使用了`sockfilter_bpf__open_and_load()`加载了skel，但是不同的是，没有使用到`sockfilter_bpf__attach()`函数，而是使用了`bpf_program__fd(）`函数。这个函数的作用是，获取ebpf文件描述符，获取到了内核代码的`socket_handler()`函数的描述符，再通过`setsockopt()`函数设置了一些操作(这个的作用后文会描述)。

  
而且与`minimal`还有不同的是，多了一个`ringbuf`的东西(环形缓冲区)，环形缓冲区的作用主要是为了方便内核代码与用户空间代码进行交互
交互流程如下：

  

    
    
    是用户空间中ring_buffer__new() ring_buffer__poll() 创建环形缓冲区并设置回调函数 轮询环形缓冲区获取最新数据  
    内核空间bpf_ringbuf_reserve() bpf_ringbuf_submit() 为环形缓冲区保留事件的空间 提交数据到回调函数  
    
    
      
    

用户空间中的代码新的东西就这么多，接下来内核空间中与tc钩子的方法差不多，多的是创建了rb结构体，并放到了`.maps`中，定义了环形缓冲区的限制信息。

  
代码中先创建了初始套接字socket，并设置了`PF_PACKET`、`SOCK_RAW`、`SOCK_NONBLOCK`、`SOCK_CLOEXEC`这些属性，使用原始套接字
(raw socket) 编程，以便在链路层进行底层网络数据包的操作。在原始套接字编程中使用 **bind**
函数将套接字与指定的网络接口绑定。具体来说，它用于将套接字与指定的网络接口关联，以便监听或发送数据。_sll.sll_ifindex =
if_nametoindex(name)_用来绑定网卡。

  

    
    
    PF_PACKET 表示使用 packet socket。  
    SOCK_RAW 表示使用原始套接字，可以处理数据链路层的数据。  
    SOCK_NONBLOCK 表示以非阻塞模式打开套接字。  
    SOCK_CLOEXEC 表示在执行 exec 时关闭套接字。  
    htons(ETH_P_ALL) 表示捕获所有协议的数据包。  
    
    
      
    

`setsockopt`函数用于设置套接字选项，允许应用程序配置套接字的各种属性。以下是 setsockopt的参数：  
`sockfd`: 套接字描述符。  
`level`: 选项所属的协议层，常见的有`SOL_SOCKET`、`IPPROTO_TCP`、`IPPROTO_UDP`等。  
`optname`: 选项的名称，表示要设置的具体选项。  
`optval`: 指向包含新选项值的缓冲区的指针。  
`optlen`: 缓冲区的大小，即选项值的长度。  
  

前面通过`bpf_program__fd（）`函数获取到了hook函数的文件描述符，通过setsockopt设置挂钩，程序在使用socket后就会先通过内核代码中的`socket_handler()`函数。

  
完整的代码注释如下：

  

    
    
    #include <stddef.h>  
    #include <linux/bpf.h>  
    #include <linux/if_ether.h>  
    #include <linux/ip.h>  
    #include <linux/in.h>  
    #include <bpf/bpf_helpers.h>  
    #include <bpf/bpf_endian.h>  
    #include "sockfilter.h" //example/c 下编译自带  
      
    //定义了环形缓冲区的限制信息  
    struct {  
        __uint(type, BPF_MAP_TYPE_RINGBUF);  
        __uint(max_entries, 256 * 1024);  
    } rb SEC(".maps");  
      
    #define IP_MF	  0x2000  
    #define IP_OFFSET 0x1FFF  
      
    char LICENSE[] SEC("license") = "Dual BSD/GPL";  
      
    //判断数据包是否使用了分片技术  
    static inline int ip_is_fragment(struct __sk_buff *skb, __u32 nhoff)  
    {  
        __u16 frag_off;  
        //加载  
        bpf_skb_load_bytes(skb, nhoff + offsetof(struct iphdr, frag_off), &frag_off, 2);  
        frag_off = __bpf_ntohs(frag_off);  
        return frag_off & (IP_MF | IP_OFFSET);  
    }  
      
    //hook 协议栈  
    SEC("socket")  
    int socket_hook(struct __sk_buff *skb){  
        struct so_event *e;  
        __u8 verlen;  
        __u16 proto;  
        //定义首部字节长度  
        __u32 nhoff = ETH_HLEN;  
      
        /*  
        bpf_skb_load_bytes 是用于从数据包中加载字节的 eBPF 函数  
        skb: 指向 struct __sk_buff 结构体的指针，表示数据包的信息。  
        offset: 数据包中的偏移量，表示从哪里开始加载字节。  
        to: 指向目标缓冲区的指针，用于存储加载的字节。  
        len: 要加载的字节数。  
        */  
        bpf_skb_load_bytes(skb, 12, &proto, 2);  
        proto = __bpf_ntohs(proto);  
        //判断是不是IPv4协议  
        if (proto != ETH_P_IP)  
            return 0;  
        //判断数据包是否分片  
        if (ip_is_fragment(skb, nhoff))  
            return 0;  
      
        //为环形缓冲区保留事件的空间  
        e = bpf_ringbuf_reserve(&rb, sizeof(*e), 0);  
        if (!e)  
            return 0;  
      
        bpf_skb_load_bytes(skb, nhoff + offsetof(struct iphdr, protocol), &e->ip_proto, 1);  
        //判断是否使用了GRE协议  
        if (e->ip_proto != IPPROTO_GRE) {  
            bpf_skb_load_bytes(skb, nhoff + offsetof(struct iphdr, saddr), &(e->src_addr), 4);  
            bpf_skb_load_bytes(skb, nhoff + offsetof(struct iphdr, daddr), &(e->dst_addr), 4);  
        }  
        bpf_skb_load_bytes(skb, nhoff + 0, &verlen, 1);  
        bpf_skb_load_bytes(skb, nhoff + ((verlen & 0xF) << 2), &(e->ports), 4);  
        e->pkt_type = skb->pkt_type;  
        //skb->ifindex 记录了数据包所经过的网络接口的索引值 网卡索引号 lo ens33网卡的索引号不同  
        e->ifindex = skb->ifindex;  
        bpf_printk("index %d",skb->ifindex);  
        bpf_ringbuf_submit(e, 0);  
        return skb->len;  
    }  
    
    
      
    

### sockfilter内核代码分析

  

在内核代码中使用了`bpf_skb_load_bytes()`函数，通过偏移值获，从skb中获取信息，通过`bpf_ringbuf_reserve()`函数获取到交互数据，并进行修改后通过`bpf_ringbuf_submit()`返回给用户空间。

  
通过创建了2个结构体`skb`与`e`，skb是hook后拿到的上下文信息，e提取了skb的一些关键信息，再返回给用户空间。

  
以下是代码的全部注释：

  

    
    
    #include <argp.h>  
    #include <arpa/inet.h>  
    #include <assert.h>  
    #include <bpf/libbpf.h>  
    #include <linux/if_packet.h>  
    #include <linux/if_ether.h>  
    #include <linux/in.h>  
    #include <net/if.h>  
    #include <signal.h>  
    #include <stdio.h>  
    #include <sys/resource.h>  
    #include <sys/socket.h>  
    #include <unistd.h>  
    #include "sockfilter.h"  
    #include "shell.skel.h"  
      
    static struct env {  
        const char *interface;  
    } env;  
      
      
    static const char *ipproto_mapping[IPPROTO_MAX] = {  
        [IPPROTO_IP] = "IP",	   [IPPROTO_ICMP] = "ICMP",	  [IPPROTO_IGMP] = "IGMP",  
        [IPPROTO_IPIP] = "IPIP",   [IPPROTO_TCP] = "TCP",	  [IPPROTO_EGP] = "EGP",  
        [IPPROTO_PUP] = "PUP",	   [IPPROTO_UDP] = "UDP",	  [IPPROTO_IDP] = "IDP",  
        [IPPROTO_TP] = "TP",	   [IPPROTO_DCCP] = "DCCP",	  [IPPROTO_IPV6] = "IPV6",  
        [IPPROTO_RSVP] = "RSVP",   [IPPROTO_GRE] = "GRE",	  [IPPROTO_ESP] = "ESP",  
        [IPPROTO_AH] = "AH",	   [IPPROTO_MTP] = "MTP",	  [IPPROTO_BEETPH] = "BEETPH",  
        [IPPROTO_ENCAP] = "ENCAP", [IPPROTO_PIM] = "PIM",	  [IPPROTO_COMP] = "COMP",  
        [IPPROTO_SCTP] = "SCTP",   [IPPROTO_UDPLITE] = "UDPLITE", [IPPROTO_MPLS] = "MPLS",  
        [IPPROTO_RAW] = "RAW"  
        };  
      
    //将地址转换成点分十进制  
    static inline void ltoa(uint32_t addr, char *dst)  
    {  
        snprintf(dst, 16, "%u.%u.%u.%u", (addr >> 24) & 0xFF, (addr >> 16) & 0xFF,  
                 (addr >> 8) & 0xFF, (addr & 0xFF));  
    }  
      
    /*  
    环形缓冲区操作bpf_ringbuf_output() bpf_ringbuf_reserve() 后者可对data进行写入操作 bpf_ringbuf_submit()函数向回调函数进行提交数据    
    */  
    static int handle_event(void *ctx, void *data, size_t data_sz)  
    {  
        const struct so_event *e = data;  
        char ifname[IF_NAMESIZE];  
        char sstr[16] = {}, dstr[16] = {};  
      
        if (e->pkt_type != PACKET_HOST)  
            return 0;  
      
        if (e->ip_proto < 0 || e->ip_proto >= IPPROTO_MAX)  
            return 0;  
      
        if (!if_indextoname(e->ifindex, ifname))  
            return 0;  
      
        ltoa(ntohl(e->src_addr), sstr);  
        ltoa(ntohl(e->dst_addr), dstr);  
      
        printf("interface: %s\tprotocol: %s\t%s:%d(src) -> %s:%d(dst)\n", ifname,  
               ipproto_mapping[e->ip_proto], sstr, ntohs(e->port16[0]), dstr, ntohs(e->port16[1]));  
      
        return 0;  
    }  
      
    //创建一个socket连接 指定网卡并设置属性 为后面setsockopt提供sock描述符  
    static int open_raw_sock(const char *name)  
    {  
        struct sockaddr_ll sll;  
        int sock;  
        //创建初始SOKET连接  
        /*  
        PF_PACKET 表示使用 packet socket。  
        SOCK_RAW 表示使用原始套接字，可以处理数据链路层的数据。  
        SOCK_NONBLOCK 表示以非阻塞模式打开套接字。  
        SOCK_CLOEXEC 表示在执行 exec 时关闭套接字。  
        htons(ETH_P_ALL) 表示捕获所有协议的数据包。  
        */  
        sock = socket(PF_PACKET, SOCK_RAW | SOCK_NONBLOCK | SOCK_CLOEXEC, htons(ETH_P_ALL));  
        if (sock < 0) {  
            fprintf(stderr, "Failed to create raw socket\n");  
            return -1;  
        }  
        // 配置 socket 基础信息  
        memset(&sll, 0, sizeof(sll));  
        sll.sll_family = AF_PACKET;  
        sll.sll_ifindex = if_nametoindex(name);  
        sll.sll_protocol = htons(ETH_P_ALL);  
        //绑定socket基础信息   
        if (bind(sock, (struct sockaddr *)&sll, sizeof(sll)) < 0) {  
            fprintf(stderr, "Failed to bind to %s: %s\n", name, strerror(errno));  
            close(sock);  
            return -1;  
        }  
        return sock;  
    }  
      
    //设置打印回调  
    static int libbpf_print_fn(enum libbpf_print_level level, const char *format, va_list args)  
    {  
        return vfprintf(stderr, format, args);  
    }  
      
    static volatile bool exiting = false;  
      
    static void sig_handler(int sig)  
    {  
        exiting = true;  
    }  
      
    int main(){  
        struct shell_bpf *skel;  
        struct ring_buffer *rb = NULL;  
        int err,sock,prog_fd;  
        //用于设置 libbpf 库的错误和调试信息输出回调函数  
        libbpf_set_print(libbpf_print_fn);  
        skel = shell_bpf__open_and_load();  
        if (!skel)  
        {  
            fprintf(stderr,"Open ebpf Skeleton Error\n");  
            return -1;  
        }  
        skel->bss->protect_pid = getppid();  
        env.interface = "lo";  
        //设置 ctrl + C 卸载钩子  
        signal(SIGINT, sig_handler);  
        signal(SIGTERM, sig_handler);  
          
        //ring_buffer__new 创建环形缓冲区 用于用户态与内核态交换数据 bpf_map__fd获取文件描述符    
        /*  
        maps 是 libbpf中可以作为数据交换的结构体  
        progs 通常指代 BPF Program 对象的集合。在 BPF 中，Program 是执行具体功能的代码单元  通常为bpf程序的函数  
        links 通常指代 BPF Link 对象的集合。在 BPF 中，Link 用于将 BPF Program 关联到特定的 BPF Hook 或事件  
        bss 通常指代 BSS（Block Started by Symbol）段，这是程序中的一块内存，用于存储未初始化的全局或静态变量  
        */  
        rb = ring_buffer__new(bpf_map__fd(skel->maps.rb), handle_event, NULL, NULL);  
        if (!rb) {  
            err = -1;  
            fprintf(stderr, "Failed to create ring buffer\n");  
            goto clean;  
        }  
        //配置socket  
        sock = open_raw_sock(env.interface);  
        if (sock < 0) {  
            err = -2;  
            fprintf(stderr, "Failed to open raw socket\n");  
            goto clean;  
        }  
      
        /*  
        从内核数据中找到hook 函数的文件描述符 并通过setsockopt设置hook  
        sockfd：套接字描述符，表示要设置选项的套接字。  
        level：选项的协议层，通常使用 SOL_SOCKET 表示套接字级别选项。  
        optname：要设置的选项名，表示要进行的特定操作。  
        optval：指向存储选项值的缓冲区的指针。  
        optlen：optval 缓冲区的长度。  
        */  
        prog_fd = bpf_program__fd(skel->progs.socket_hook);  
        if (setsockopt(sock, SOL_SOCKET, SO_ATTACH_BPF, &prog_fd, sizeof(prog_fd))) {  
            err = -3;  
            fprintf(stderr, "Failed to attach to raw socket\n");  
            goto clean;  
        }  
        while (!exiting) {  
            /*  
            ring_buffer__poll 函数通常是 libbpf 库中用于轮询 BPF 环形缓冲区（Ring Buffer）的函数。该函数的主要作用是等待并获取新的事件或数据  
            ringbuf：指向 struct ring_buffer 结构的指针，表示要轮询的环形缓冲区。  
            timeout_ms：等待新事件的超时时间，以毫秒为单位。如果设置为负数，表示无限等待  
            */  
            err = ring_buffer__poll(rb, 100 /* timeout, ms */);  
            /* Ctrl-C will cause -EINTR */  
            if (err == -EINTR) {  
                err = 0;  
                break;  
            }  
            if (err < 0) {  
                fprintf(stderr, "Error polling perf buffer: %d\n", err);  
                break;  
            }  
            sleep(1);  
        }  
    clean:  
        //销毁skel  
        shell_bpf__destroy(skel);  
        //需要释放环形缓冲区  
        ring_buffer__free(rb);  
        return 0;  
    }  
    

  

  

  

![]()

  

 **看雪ID：yfsec**

https://bbs.kanxue.com/user-home-977105.htm

*本文为看雪论坛优秀文章，由 yfsec 原创，转载请注明来自看雪社区  
[![]()](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458518567&idx=3&sn=5845aa88a06560b5666eae7ce4ae039b&chksm=b18d34ad86fabdbb626f57e5bcdf1970f5e157e5d5c7240f8e4793f1fd8bef5c5413deaafa7e&scene=21#wechat_redirect)  
  

 **#** **  往期推荐  
**

1、[区块链智能合约逆向-合约创建-
调用执行流程分析](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458532403&idx=1&sn=3cb169db2b7587d7679fdb4ab1b1e7db&scene=21#wechat_redirect)

2、[在Windows平台使用VS2022的MSVC编译LLVM16](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458532326&idx=2&sn=1f474e4a32960bd62ca80b5172485589&scene=21#wechat_redirect)

3、[神挡杀神——揭开世界第一手游保护nProtect的神秘面纱](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458531968&idx=1&sn=f5d10b971479f00b4ba1b4bc43d63f21&scene=21#wechat_redirect)

4、[为什么在ASLR机制下DLL文件在不同进程中加载的基址相同](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458531931&idx=2&sn=c6d3d71c15a29a24e9fa288f963c82bc&scene=21#wechat_redirect)

5、[2022QWB final
RDP](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458531697&idx=1&sn=ce28e8201aee34f0be6a8b6a97c4d9e4&scene=21#wechat_redirect)

6、[华为杯研究生国赛
adv_lua](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458531696&idx=1&sn=31c1dabbd80a62307ad24f4c119170fe&scene=21#wechat_redirect)

  
![]()  
  
![]()

 **球分享**

![]()

 **球点赞**

![]()

 **球在看**

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 使用 libbpf 进行 ebpf 开发

yfsec  [ 看雪学苑 ](javascript:void\(0\);)

轻触阅读原文

![]()

看雪学苑

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

