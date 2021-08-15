# 0x01 什么是 CNG 以及为什么要使用它


本来我在做使用 WIN32 API 加密的时候（比如 AES、RC4） 是通过 CSP 实现的。CSP 是一个独立的软件模块，实际上执行用于身份验证，编码和加密的密码算法。CSP 提供了一组密码学 API，使应用程序开发人员可以向基于 Windows 的应用程序添加身份验证，编码和加密。

但是现在有了新一代密码学 API，也被称为 [CNG](https://docs.microsoft.com/en-us/windows/win32/seccng/cng-portal)（`Cryprography API: Next Generation`），MSDN 号称：

>之前的 `Cryptography API` 不再推荐使用。

如：

- CryptAcquireContextA
- CryptCreateHash
- CryptDeriveKey
- ...

>新的和现有的软件应开始使用Cryptography Next Generation API。Microsoft可能会在将来的版本中删除之前的 `Cryptography API`。

本文就是使用 CNG（新一代密码学 API）来示范 AES 的使用，因为避免关联，不会给出完整代码，使用的尽量是 MSDN 上的代码或来源于其他合法渠道的公开代码示例。



# 0x02 实现 AES CBC 模式

CBC 模式是需要使用 IV 的。IV 即初始化向量，相比于 ECB 模式，CBC 模式对每个明文分组要与前一个密文分组进行异或，这样的作用是即时两个明文分组的值是相等的，经过`多字节异或`，对应的两个密文分组的值也不一定是相等的。而IV 的使用就是为了解决第一个明文分组跟谁异或的问题。

当加密第一个明文分组时，由于不存在“前一个密文分组”，因此需要事先准备一个长度为一个分组的比特序列来代替“前一个密文分组”，这个比特序列称为初始化向量（Initialization Vector），通常缩写为IV，一般来说，每次加密时都会随机产生一个不同的比特序列来作为初始化向量。


这里要注意，IV 的长度与分组的大小是一致的。而 AES 的标准分组为 128 Bits，即为 16 字节。

AES 有三个输入和一个输出：

- 输入1：IV。IV 不得大于16字节，如果用户输入大于则截断，小于则填充0。
- 输入2：Key。用于加密的密钥。标准 AES 为 16 字节。如果用户输入如果大于16字节则截断，小于则填充0。
- 输入3：明文
- 输出1：密文



完整代码为：


```
// Sample program for AES-CBC encryption using CNG


#include <windows.h>
#include <stdio.h>
#pragma comment(lib, "bcrypt.lib")


#define NT_SUCCESS(Status)          (((NTSTATUS)(Status)) >= 0)

#define STATUS_UNSUCCESSFUL         ((NTSTATUS)0xC0000001L)


#define DATA_TO_ENCRYPT  "Test Data"


const BYTE rgbPlaintext[] =
{
    0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,
    0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F
};

static const BYTE rgbIV[] =
{
    0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,
    0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F, 0x18
};

static const BYTE rgbAES128Key[] =
{
    0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,
    0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F
};

void PrintBytes(
    IN BYTE* pbPrintData,
    IN DWORD    cbDataLen)
{
    DWORD dwCount = 0;

    for (dwCount = 0; dwCount < cbDataLen;dwCount++)
    {
        printf("0x%02x, ", pbPrintData[dwCount]);

        if (0 == (dwCount + 1) % 10) putchar('\n');
    }

}

void __cdecl wmain(
    int                      argc,
    __in_ecount(argc) LPWSTR* wargv)
{

    BCRYPT_ALG_HANDLE       hAesAlg = NULL;
    BCRYPT_KEY_HANDLE       hKey = NULL;
    NTSTATUS                status = STATUS_UNSUCCESSFUL;
    DWORD                   cbCipherText = 0,
        cbPlainText = 0,
        cbData = 0,
        cbKeyObject = 0,
        cbBlockLen = 0,
        cbBlob = 0;
    PBYTE                   pbCipherText = NULL,
        pbPlainText = NULL,
        pbKeyObject = NULL,
        pbIV = NULL,
        pbBlob = NULL;

    UNREFERENCED_PARAMETER(argc);
    UNREFERENCED_PARAMETER(wargv);


    // Open an algorithm handle.
    if (!NT_SUCCESS(status = BCryptOpenAlgorithmProvider(
        &hAesAlg,
        BCRYPT_AES_ALGORITHM,
        NULL,
        0)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptOpenAlgorithmProvider\n", status);
        goto Cleanup;
    }

    // Calculate the size of the buffer to hold the KeyObject.
    if (!NT_SUCCESS(status = BCryptGetProperty(
        hAesAlg,
        BCRYPT_OBJECT_LENGTH,
        (PBYTE)&cbKeyObject,
        sizeof(DWORD),
        &cbData,
        0)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptGetProperty\n", status);
        goto Cleanup;
    }

    // Allocate the key object on the heap.
    pbKeyObject = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbKeyObject);
    if (NULL == pbKeyObject)
    {
        wprintf(L"**** memory allocation failed\n");
        goto Cleanup;
    }

    // Calculate the block length for the IV.
    if (!NT_SUCCESS(status = BCryptGetProperty(
        hAesAlg,
        BCRYPT_BLOCK_LENGTH,
        (PBYTE)&cbBlockLen,
        sizeof(DWORD),
        &cbData,
        0)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptGetProperty\n", status);
        goto Cleanup;
        
    }
    //printf("%d\n", cbBlockLen);

    // Determine whether the cbBlockLen is not longer than the IV length.
    if (cbBlockLen > sizeof(rgbIV))
    {
        wprintf(L"**** block length is longer than the provided IV length\n");
        goto Cleanup;
    }

    // Allocate a buffer for the IV. The buffer is consumed during the 
    // encrypt/decrypt process.
    pbIV = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbBlockLen);
    if (NULL == pbIV)
    {
        wprintf(L"**** memory allocation failed\n");
        goto Cleanup;
    }

    memcpy(pbIV, rgbIV, cbBlockLen);

    if (!NT_SUCCESS(status = BCryptSetProperty(
        hAesAlg,
        BCRYPT_CHAINING_MODE,
        (PBYTE)BCRYPT_CHAIN_MODE_CBC,
        sizeof(BCRYPT_CHAIN_MODE_CBC),
        0)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptSetProperty\n", status);
        goto Cleanup;
    }



    // Generate the key from supplied input key bytes.
    if (!NT_SUCCESS(status = BCryptGenerateSymmetricKey(
        hAesAlg,
        &hKey,
        pbKeyObject,
        cbKeyObject,
        (PBYTE)rgbAES128Key,
        sizeof(rgbAES128Key),
        0)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptGenerateSymmetricKey\n", status);
        goto Cleanup;
    }


    // Save another copy of the key for later.
    if (!NT_SUCCESS(status = BCryptExportKey(
        hKey,
        NULL,
        BCRYPT_OPAQUE_KEY_BLOB,
        NULL,
        0,
        &cbBlob,
        0)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptExportKey\n", status);
        goto Cleanup;
    }

    
    // Allocate the buffer to hold the BLOB.
    pbBlob = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbBlob);
    if (NULL == pbBlob)
    {
        wprintf(L"**** memory allocation failed\n");
        goto Cleanup;
    }

    if (!NT_SUCCESS(status = BCryptExportKey(
        hKey,
        NULL,
        BCRYPT_OPAQUE_KEY_BLOB,
        pbBlob,
        cbBlob,
        &cbBlob,
        0)))
    {
        
        
        wprintf(L"**** Error 0x%x returned by BCryptExportKey\n", status);
        goto Cleanup;
    }
    //PrintBytes(pbBlob,cbBlob);



    cbPlainText = sizeof(rgbPlaintext);
    pbPlainText = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbPlainText);
    if (NULL == pbPlainText)
    {
        wprintf(L"**** memory allocation failed\n");
        goto Cleanup;
    }

    memcpy(pbPlainText, rgbPlaintext, sizeof(rgbPlaintext));

    //
    // Get the output buffer size.
    //
    if (!NT_SUCCESS(status = BCryptEncrypt(
        hKey,
        pbPlainText,
        cbPlainText,
        NULL,
        pbIV,
        cbBlockLen,
        NULL,
        0,
        &cbCipherText,
        BCRYPT_BLOCK_PADDING)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptEncrypt\n", status);
        goto Cleanup;
    }

    pbCipherText = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbCipherText);
    if (NULL == pbCipherText)
    {
        wprintf(L"**** memory allocation failed\n");
        goto Cleanup;
    }

    // Use the key to encrypt the plaintext buffer.
    // For block sized messages, block padding will add an extra block.
    if (!NT_SUCCESS(status = BCryptEncrypt(
        hKey,
        pbPlainText,
        cbPlainText,
        NULL,
        pbIV,
        cbBlockLen,
        pbCipherText,
        cbCipherText,
        &cbData,
        BCRYPT_BLOCK_PADDING)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptEncrypt\n", status);
        goto Cleanup;
    }

    // Destroy the key and reimport from saved BLOB.
    if (!NT_SUCCESS(status = BCryptDestroyKey(hKey)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptDestroyKey\n", status);
        goto Cleanup;
    }
    hKey = 0;

    if (pbPlainText)
    {
        HeapFree(GetProcessHeap(), 0, pbPlainText);
    }

    pbPlainText = NULL;

    // We can reuse the key object.
    memset(pbKeyObject, 0, cbKeyObject);


    // Reinitialize the IV because encryption would have modified it.
    memcpy(pbIV, rgbIV, cbBlockLen);


    if (!NT_SUCCESS(status = BCryptImportKey(
        hAesAlg,
        NULL,
        BCRYPT_OPAQUE_KEY_BLOB,
        &hKey,
        pbKeyObject,
        cbKeyObject,
        pbBlob,
        cbBlob,
        0)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptGenerateSymmetricKey\n", status);
        goto Cleanup;
    }


    //
    // Get the output buffer size.
    //
    if (!NT_SUCCESS(status = BCryptDecrypt(
        hKey,
        pbCipherText,
        cbCipherText,
        NULL,
        pbIV,
        cbBlockLen,
        NULL,
        0,
        &cbPlainText,
        BCRYPT_BLOCK_PADDING)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptDecrypt\n", status);
        goto Cleanup;
    }

    pbPlainText = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbPlainText);
    if (NULL == pbPlainText)
    {
        wprintf(L"**** memory allocation failed\n");
        goto Cleanup;
    }

    if (!NT_SUCCESS(status = BCryptDecrypt(
        hKey,
        pbCipherText,
        cbCipherText,
        NULL,
        pbIV,
        cbBlockLen,
        pbPlainText,
        cbPlainText,
        &cbPlainText,
        BCRYPT_BLOCK_PADDING)))
    {
        wprintf(L"**** Error 0x%x returned by BCryptDecrypt\n", status);
        goto Cleanup;
    }


    if (0 != memcmp(pbPlainText, (PBYTE)rgbPlaintext, sizeof(rgbPlaintext)))
    {
        wprintf(L"Expected decrypted text comparison failed.\n");
        goto Cleanup;
    }

    wprintf(L"Success!\n");


Cleanup:

    if (hAesAlg)
    {
        BCryptCloseAlgorithmProvider(hAesAlg, 0);
    }

    if (hKey)
    {
        BCryptDestroyKey(hKey);
    }

    if (pbCipherText)
    {
        HeapFree(GetProcessHeap(), 0, pbCipherText);
    }

    if (pbPlainText)
    {
        HeapFree(GetProcessHeap(), 0, pbPlainText);
    }

    if (pbKeyObject)
    {
        HeapFree(GetProcessHeap(), 0, pbKeyObject);
    }

    if (pbIV)
    {
        HeapFree(GetProcessHeap(), 0, pbIV);
    }

}
```



# 0x03 代码分析

## 打开一个 AES 算法句柄


```
BCRYPT_ALG_HANDLE       hAesAlg = NULL;
BCryptOpenAlgorithmProvider(
        &hAesAlg,
        BCRYPT_AES_ALGORITHM,
        NULL,
        0)
```

通过 BCryptOpenAlgorithmProvider 函数加载并初始化CNG供应商，参数2指定使用 AES 算法，参数1接收算法句柄。


## 指定 CBC 模式


```
BCryptSetProperty(
        hAesAlg,
        BCRYPT_CHAINING_MODE,
        (PBYTE)BCRYPT_CHAIN_MODE_CBC,
        sizeof(BCRYPT_CHAIN_MODE_CBC),
        0)
```





## 由 key 获取 hkey

所谓生成 Key，是使用用户输入的 key，经过处理返回一个 hkey 句柄。句柄的设计思路是提醒用户在使用完之后 destroy。

```
BCryptGenerateSymmetricKey(
        hAesAlg,
        &hKey,
        pbKeyObject,
        cbKeyObject,
        (PBYTE)rgbAES128Key,
        sizeof(rgbAES128Key),
        0)
```


BCryptGenerateSymmetricKey API 是如何从用户输入的 key 返回 hKey 的？

```
NTSTATUS BCryptGenerateSymmetricKey(
  BCRYPT_ALG_HANDLE hAlgorithm,
  BCRYPT_KEY_HANDLE *phKey,
  PUCHAR            pbKeyObject,
  ULONG             cbKeyObject,
  PUCHAR            pbSecret,
  ULONG             cbSecret,
  ULONG             dwFlags
);
```

通过倒数第三个参数 `pbSecret` 传入用户输入的 key。

MSDN 如此描述此参数：

>通常，这是密码或其他可重现数据的哈希。如果传入的数据超过目标密钥大小，则数据将被截断，多余的数据将被忽略。

其实也就是截断为 16 字节。




第三个参数 `pbKeyObject` 指向接收密钥对象的缓冲区的指针。该cbKeyObject参数包含该缓冲区的大小。可以通过调用BCryptGetProperty函数以获得BCRYPT_OBJECT_LENGTH属性来获得此缓冲区的所需大小。这将提供指定算法的密钥对象的大小。


```
//机算存储 Key 对象的缓冲区的大小
BCryptGetProperty(
        hAesAlg,
        BCRYPT_OBJECT_LENGTH,
        (PBYTE)&cbKeyObject,
        sizeof(DWORD),
        &cbData,
        0)
        
// 在堆上为 key 对象分配内存
pbKeyObject = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbKeyObject);

```

## 导出 key BLOB

导出 key BLOB 的作用是当加密完之后我们就会通过 `BCryptDestroyKey(hKey)` 来 destroy hkey。

当解密时，如果再想获取 key，需要从保存的 key BLOB （也就是保存的 pbBlob 数组）中通过 `BCryptImportKey` 函数获取 hKey。

```
BCryptExportKey(
        hKey,
        NULL,
        BCRYPT_OPAQUE_KEY_BLOB,
        NULL,
        0,
        &cbBlob,
        0)
        
// Allocate the buffer to hold the BLOB.
pbBlob = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbBlob);

BCryptExportKey(
    hKey,
    NULL,
    BCRYPT_OPAQUE_KEY_BLOB,
    pbBlob,
    cbBlob,
    &cbBlob,
    0)
```


`BCryptGenerateSymmetricKey` 或 `BCryptImportKey` 这两个函数皆可以用于创建或导入对称密钥。

导出 key  的 BLOB 的时候使用了两遍 BCryptExportKey API，第一次是计算输出数组的大小，第二遍是存入导出的 key BLOB（BLOB 是包含一个或多个固定长度报头结构以及上下文特定数据的通用位序列。）。

我尝试打印了一下 key BLOB 的值，是一个 560 字节的数组。


```
560
0x30, 0x02, 0x00, 0x00, 0x4b, 0x53, 0x53, 0x4d, 0x02, 0x00,
0x01, 0x00, 0x01, 0x00, 0x00, 0x00, 0x10, 0x00, 0x00, 0x00,
0x80, 0x00, 0x00, 0x00, 0x10, 0x00, 0x00, 0x00, 0x00, 0x01,
0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b,
0x0c, 0x0d, 0x0e, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x01, 0x02, 0x03, 0x04, 0x05,
0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f,
0xd6, 0xaa, 0x74, 0xfd, 0xd2, 0xaf, 0x72, 0xfa, 0xda, 0xa6,
0x78, 0xf1, 0xd6, 0xab, 0x76, 0xfe, 0xb6, 0x92, 0xcf, 0x0b,
0x64, 0x3d, 0xbd, 0xf1, 0xbe, 0x9b, 0xc5, 0x00, 0x68, 0x30,
0xb3, 0xfe, 0xb6, 0xff, 0x74, 0x4e, 0xd2, 0xc2, 0xc9, 0xbf,
0x6c, 0x59, 0x0c, 0xbf, 0x04, 0x69, 0xbf, 0x41, 0x47, 0xf7,
0xf7, 0xbc, 0x95, 0x35, 0x3e, 0x03, 0xf9, 0x6c, 0x32, 0xbc,
0xfd, 0x05, 0x8d, 0xfd, 0x3c, 0xaa, 0xa3, 0xe8, 0xa9, 0x9f,
0x9d, 0xeb, 0x50, 0xf3, 0xaf, 0x57, 0xad, 0xf6, 0x22, 0xaa,
0x5e, 0x39, 0x0f, 0x7d, 0xf7, 0xa6, 0x92, 0x96, 0xa7, 0x55,
0x3d, 0xc1, 0x0a, 0xa3, 0x1f, 0x6b, 0x14, 0xf9, 0x70, 0x1a,
0xe3, 0x5f, 0xe2, 0x8c, 0x44, 0x0a, 0xdf, 0x4d, 0x4e, 0xa9,
0xc0, 0x26, 0x47, 0x43, 0x87, 0x35, 0xa4, 0x1c, 0x65, 0xb9,
0xe0, 0x16, 0xba, 0xf4, 0xae, 0xbf, 0x7a, 0xd2, 0x54, 0x99,
0x32, 0xd1, 0xf0, 0x85, 0x57, 0x68, 0x10, 0x93, 0xed, 0x9c,
0xbe, 0x2c, 0x97, 0x4e, 0x13, 0x11, 0x1d, 0x7f, 0xe3, 0x94,
0x4a, 0x17, 0xf3, 0x07, 0xa7, 0x8b, 0x4d, 0x2b, 0x30, 0xc5,
0x13, 0xaa, 0x29, 0xbe, 0x9c, 0x8f, 0xaf, 0xf6, 0xf7, 0x70,
0xf5, 0x80, 0x00, 0xf7, 0xbf, 0x03, 0x13, 0x62, 0xa4, 0x63,
0x8f, 0x25, 0x86, 0x48, 0x6b, 0xff, 0x5a, 0x76, 0xf7, 0x87,
0x4a, 0x83, 0x8d, 0x82, 0xfc, 0x74, 0x9c, 0x47, 0x22, 0x2b,
0xe4, 0xda, 0xdc, 0x3e, 0x9c, 0x78, 0x10, 0xf5, 0x72, 0xe3,
0x09, 0x8d, 0x11, 0xc5, 0xde, 0x5f, 0x78, 0x9d, 0xfe, 0x15,
0x78, 0xa2, 0xcc, 0xcb, 0x2e, 0xc4, 0x10, 0x27, 0x63, 0x26,
0xd7, 0xd2, 0x69, 0x58, 0x20, 0x4a, 0x00, 0x3f, 0x32, 0xde,
0xa8, 0xa2, 0xf5, 0x04, 0x4d, 0xe2, 0xc7, 0xf5, 0x0a, 0x7e,
0xf7, 0x98, 0x69, 0x67, 0x12, 0x94, 0xc7, 0xc6, 0xe3, 0x91,
0xe5, 0x40, 0x32, 0xf1, 0x47, 0x9c, 0x30, 0x6d, 0x63, 0x19,
0xe5, 0x0c, 0xa0, 0xdb, 0x02, 0x99, 0x22, 0x86, 0xd1, 0x60,
0xa2, 0xdc, 0x02, 0x9c, 0x24, 0x85, 0xd5, 0x61, 0x8c, 0x56,
0xdf, 0xf0, 0x82, 0x5d, 0xd3, 0xf9, 0x80, 0x5a, 0xd3, 0xfc,
0x86, 0x59, 0xd7, 0xfd, 0x00, 0x01, 0x02, 0x03, 0x04, 0x05,
0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd,
0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xdd, 0xa0, 0x00,
0x00, 0x00, 0x40, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
```

这个 key BLOB 是如何生成的？具体细节未知，这样的话造成了一个问题：如果用其他语言如何实现对应的加解密过程？如，使用 python 实现加密器，真正用于加密的 key 都无法计算出来。



这就是这套 API 的缺点：跨语言难以实现。





## 获取存储密文的内存缓冲区的大小

```
BCryptEncrypt(
        hKey,
        pbPlainText,
        cbPlainText,
        NULL,
        pbIV,
        cbBlockLen,
        NULL,
        0,
        &cbCipherText,
        BCRYPT_BLOCK_PADDING)
```


>BcryptEncrypt 函数的倒数第二个参数 `pcbResult` 指向ULONG变量的指针，该变量接收复制到pbOutput缓冲区的字节数。如果pbOutput（倒数第四个参数）为NULL，它将接收密文所需的大小（以字节为单位）。



## 使用生成的 key 进行 AES 加密

```
BCryptEncrypt(
        hKey,
        pbPlainText,
        cbPlainText,
        NULL,
        pbIV,
        cbBlockLen,
        pbCipherText,
        cbCipherText,
        &cbData,
        BCRYPT_BLOCK_PADDING)
```

注意这里密文会被存入 `pbCipherText` 指向的地址。

另外通过指定 `BCRYPT_BLOCK_PADDING`，默认使用  PKCS7Padding 填充模式。也可以将此参数传入0，则为 No Padding。


此函数的第五个函数 `pbIV`，需要传入在加密期间使用的 IV 的缓冲区地址。第六个参数 `cbIV` 传入的是此缓冲区的大小。

可以通过调用 BCryptGetProperty 函数以获得 `BCRYPT_BLOCK_LENGTH` 属性来获得 IV 的所需大小。这将为算法提供块的大小，这也是 IV 的大小。正确的情况下应该是默认的 16 字节。




```
//计算 IV 所需的大小
BCryptGetProperty(
    hAesAlg,
    BCRYPT_BLOCK_LENGTH,
    (PBYTE)&cbBlockLen,
    sizeof(DWORD),
    &cbData,
    0)

pbIV = (PBYTE)HeapAlloc(GetProcessHeap(), 0, cbBlockLen);
```

IV 是需要用户自己指定的，如:

```
static const BYTE rgbIV[] =
{
    0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,
    0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F
};
```

我尝试了一下指定17个字节的 IV 数组，发现会被截断为16字节。







------------------


# 致谢：

感谢 @zcgonvh 的铮铮教诲。


