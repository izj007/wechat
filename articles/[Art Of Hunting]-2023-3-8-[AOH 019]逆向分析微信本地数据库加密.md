#  [AOH 019]逆向分析微信本地数据库加密

原创 Djerryz  [ Art Of Hunting ](javascript:void\(0\);)

**Art Of Hunting** ![]()

微信号 gh_d3ebfd9e0148

功能介绍 未知的漏洞存在于我们的想象中，直到它被安全的艺术所塑造

____

___发表于_

收录于合集

#逆向 1 个

##AOH 6 个

#sqlcipher 1 个

#微信 1 个

## 一、前言

很多客户端程序采用了sqlite作为本地数据存储方式，但为了确保数据的安全，这些程序需要引入权限访问控制与密码算法组合的安全设计，以实现`拿不走、看不懂、改不了`的目标。市面上的客户端程序已经普遍采用了这种安全设计，例如PC微信，它最敏感的数据——用户的聊天数据被加密保存在本地。

本文将从逆向角度出发，分析PC微信本地数据库密钥的生成算法，并提出密钥提取利用思路，以此提升对于终端产品安全加固的攻防理解。  
  

## 二、基础建设

准备如下工具:

  * • win10 操作系统

  * • x64dbg

  * • ghidra

  * • bindiff

  * • process monitor

  * • visual studio (MSVC - cl.exe)

目标程序: wechat.exe [3.8.1.26]

## 三、动态分析

基于已知经验，在`C:\Users\用户名\Documents\WeChat Files\wxid\Msg`目录下存有db文件。

### 1\. 观察行为

  1. 1. 打开process monitor ，filter，`Path contains xInfo.db`

  2. 2. 观察到文件动作，查看stack

![](https://gitee.com/fuli009/images/raw/master/public/20230308191449.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191450.png)

基于上述，可知操作sqlite的行为在wechatwin.dll下

### 2\. x32dbg附加分析

  1. 1. 接着，使用x32dbg附加上去

![](https://gitee.com/fuli009/images/raw/master/public/20230308191452.png)

  2. 2\. 首次进入x32dbg，需要配置：调试-高级-隐藏调试器

  3. 3. 根据前面观察的stack得知在wechatwin.dll, 点击符号表，搜素，双击进入入口，并下断

![](https://gitee.com/fuli009/images/raw/master/public/20230308191453.png)

  4. 4. 当前位置也是wechatwin.dll内存布局的开始位置，右键-搜索-当前模块-字符串

  5. 5. 查找字符串特征，例如sqlite:

  6. ![](https://gitee.com/fuli009/images/raw/master/public/20230308191454.png)

  7. 6. sqlite3_prepare_v2通过搜索google或者github能得知来自于`github.com/sqlite/sqlite`或`github.com/sqlcipher/sqlcipher`

![](https://gitee.com/fuli009/images/raw/master/public/20230308191455.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191456.png)

  8. 7\. 那么到底是哪个呢? 最快的方式还是在字符串查找"sqlcipher":

![](https://gitee.com/fuli009/images/raw/master/public/20230308191457.png)

  9. 8. 很好，到此可以大致得知操作数据库文件使用的依赖库, 接下来不妨更进一步，找到sqlcipher的版本号:

    1. • 非常幸运的找到了chuj师傅的文章，我参考他的思路，即通过 sqlite3 错误处理下的特征字符串来确定版本。

    2. • 首先，利用关键字 "misuse"，在wechatwin.dll内存布局下查找字符串，双击进入:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191458.png)

    3. • 前面应该有时间前缀，跟进一下存储地址，即可看到前面2019的时间:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191459.png)

    4. • 当然，还可以在Ghidra下静态查看：Ghidra基址+(当前地址-wechatwin.dll基址) = `python3 hex(0x10000000+(0x694fa9e8-0x689D0000))` = 0x10b2a9e8

![](https://gitee.com/fuli009/images/raw/master/public/20230308191501.png)

  10. 9. 通过上述两种方式，成功拿到了特征字符串 `2019-04-16 19:49:53 884b4b7e502b4e991677b53971277adfaf0a04a284f8e483e2553d0f8315alt2`，基于这个时间之后最近的一版本，通过提交的commit可确认为3.28.0版本

  11.   

### 3\. 确定sqlite版本的巧妙之处

此处发散并交叉验证版本的准确性: 下载sqlite-amalgamation-3280000.zip

  1. 1. 首先特征字符选用的是"misuse"

  2. 2. 在sqlite3.c下，调用sqlite3MisuseError会进而调用sqlite3_log, 其中会调用sqlite3_sourceid()

![](https://gitee.com/fuli009/images/raw/master/public/20230308191503.png)

  3. 3. 接在看到其定义:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191504.png)

  4. 4\. 即3.28恰好匹配了上面逆向分析出来的特征字符，当然下载3.27或者3.29都是不匹配的！

### 4\. 补充-SQLCipher与SQLite的区别

参见chatgpt的回答，亦或参见sqlcipher对项目的描述:

    
    
    SQLCipher 是 SQLite 数据库库的独立分支，它增加了数据库文件的 256 位 AES 加密以及其他安全功能，如：  
    * 即时加密  
    * 完整性检测  
    * 内存清理  
    * 强密钥推导  
    SQLCipher 基于 SQLite 开发，并定期整合稳定的上游版本特性。虽然 SQLCipher 作为源代码树的独立版本维护，但该项目尽量减少对核心 SQLite 代码的修改。  
    SQLCipher 是 SQLite 的独立分支，它添加了数据库文件的 256 位 AES 加密和其他安全功能。

## 四、编译SQLCipher

接下来编译3.28版本的SQLCipher获取其符号表以便与wechatwin做bindiff比对

### 1\. 环境配置

参考:

  * • 环境相关问题: https://stackoverflow.com/questions/931652/error-fatal-error-c1034-windows-h-no-include-path-set

如果需要使用cl和nmake可能会想到是，配置例如:`C:\Program Files (x86)\Microsoft Visual
Studio\2019\Enterprise\VC\Tools\MSVC\14.29.30037\bin\Hostx86\x86`路径到PATH，但是根据参考链接，实际并不需要这么做，只需要在开始编译前，在cmd下执行`VsDevCmd.bat`后即可正常使用，且不会出现缺少文件头路径错误的情况。

### 2\. 安装tlc

    
    
    1. ps-管理员  
    2. cmd  
    3. "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\Tools\VsDevCmd.bat"  
    4. cd tcl源码目录  
    5. nmake -f makefile.vc  
    6. nmake -f makefile.vc install   
    7. C:\Tcl\bin下，将tclsh86t.exe重命名为tclsh.exe  
    8. 配置C:\Tcl\bin到环境变量

参考:

  * • 编译方式: https://www.cnblogs.com/gispathfinder/p/12183373.html

  * • 编译方式: https://www.tcl.tk/doc/howto/compile.html

  * • 源码下载: https://sourceforge.net/projects/tcl/files/Tcl/8.6.13/

### 3\. 编译openssl

    
    
    1. 安装perl  
    2. 下载openssl源码  
    3. ps-管理员 -> cmd  
    4. cd openssl源码目录  
    5. "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\Tools\VsDevCmd.bat"  
    6. perl Configure VC-WIN32  
    7. nmake  
    8. nmake test  
    9. nmake install  
    10. cd "C:\Program Files (x86)\OpenSSL\lib" - cmd  
    11. mklink libeay32.lib libcrypto.lib #后续编译会用到libeay32.lib-解决BUG

参考:

  * • 编译方式: https://www.cnblogs.com/obarong/p/13260321.html

  * • openssl源码下载: https://github.com/openssl/openssl/tags

  * • perl安装包下载: https://strawberryperl.com/download/5.32.1.1/strawberry-perl-5.32.1.1-32bit.msi

### 4\. 编译sqlite(可选)

参考:

  * • 编译方式: https://nxmax.github.io/development/compile-sqlcipher-on-windows/

  * • 源码下载: https://www.sqlite.org/2019/sqlite-amalgamation-3280000.zip

  * • 源码下载: https://www.sqlite.org/2019/sqlite-src-3280000.zip

#### 4.1 使用sqilte_amalgamation方式(推荐)

通过下面的编译方式生成pdb与dll，但是通过阅读代码，可以知道sqilte_amalgamation中并没有定义出sqlite3_key_v2函数:

    
    
    1. cmd - cd到源码目录  
    2. "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\Tools\VsDevCmd.bat"  
    3. cl.exe /DEBUG /Zi /sqlite.PDB sqlite3.c -DSQLITE_API=__declspec(dllexport) -link -dll -out:sqlite3.dll

#### 4.2 使用sqilte_src方式

    
    
    1. cmd - cd到源码目录  
    2. "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\Tools\VsDevCmd.bat"  
    3. nmake /f Makefile.msc sqlite3.c  
    4. cl.exe /DEBUG /Zi /sqlite.PDB sqlite3.c -DSQLITE_API=__declspec(dllexport) -link -dll -out:sqlite3.dll

### 5\. 编译sqlcipher

参考:

  * • 源码下载: https://github.com/sqlcipher/sqlcipher

  * • 编译方式: https://nxmax.github.io/development/compile-sqlcipher-on-windows/

    
    
    1. cmd - cd到源码目录  
    2. "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\Tools\VsDevCmd.bat"  
    3. git clone https://github.com/sqlcipher/sqlcipher.git  
    4. cd sqlcipher  
    5. git log --grep "3.28"  
    '''  
    commit 20c63a67a1e9a93259a1ae2ff575ab368a708227  
    Author: Stephen Lombardo <sjlombardo@zetetic.net>  
    Date:   Wed May 15 13:07:00 2019 -0400  
      
        update changelog to reflect bseline on SQLite 3.28.0  
      
    commit 6158146e65c707566a6116b3023b751f35f1eb81  
    Merge: 2babedd 801ae64  
    Author: Stephen Lombardo <sjlombardo@zetetic.net>  
    Date:   Wed May 15 12:58:11 2019 -0400  
      
        Merge sqlite-release(3.28.0) into prerelease-integration  
      
    commit 801ae64881927443ed9627228bee7d34ef4e786e  
    Author: Stephen Lombardo <sjlombardo@zetetic.net>  
    Date:   Wed May 15 12:49:41 2019 -0400  
      
        Snapshot of upstream SQLite 3.28.0  
    '''  
    6. git checkout 6158146e65c707566a6116b3023b751f35f1eb81  
    7. nmake /f Makefile.msc sqlite3.c  
    8. cl -I"C:\Program Files (x86)\OpenSSL\include" /DEBUG:FULL /ZI sqlite3.c -DSQLITE_API=__declspec(dllexport) -DSQLITE_TEMP_STORE=2 -DSQLITE_HAS_CODEC -DSQLITE_ENABLE_FTS5 /MT -link -dll -out:sqlcipher.dll -LIBPATH:"C:\Program Files (x86)\OpenSSL\lib" libcrypto.lib libssl.lib advapi32.lib user32.lib gdi32.lib

加上`/DEBUG:FULL`使生成sqlcipher.pdb.

加上`-DSQLITE_HAS_CODEC -DSQLITE_ENABLE_FTS5`
是因为如下这部分代码需要编译进来，通过xdbg看到有在用，尽量全部覆盖到：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191505.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191506.png)  

## 五、静态分析-尝试通过diff还原符号表

拿到sqlcipher.pdb后，即可开始利用diff还原符号表, 在这之前，先查看wechatwin.dll编译器:

    
    
    py -3 -m pip install peid  
      
    peid wechatwin.dll  
    得到: Borland Delphi (???)

### 1\. 配置ghidra并安装binexport插件

  1. 1. 编辑ghidraRun.bat，写入 `set MAXMEM=16G`

  2. 2. 运行ghidraRun.bat

  3. 3. 下载ghidra_BinExport.zip (https://github.com/google/binexport)解压得到 BinExport.zip

  4. 4. ghidra->file->install extensions选择BinExport.zip，即可

 **解决最新版ghidra与最新binexport不兼容:**

若下载的ghidra是10.2.2安装ghidra_BinExport最新版可能会提示版本不兼容，操作方法，首先解压BinExport.zip，修改`extension.properties`下的`version`,
将其冲10.2.0改成10.2.2即可正常安装

 **解决bindiff内存不足报错:**

新建bat文件，写入如下, 双击启动:

    
    
    START /B "" javaw -Xmx8192M -jar "C:\Program Files\BinDiff\bin\bindiff.jar"

### 2\. 分析sqlcipher.dll

  1. 1. ghidra新建项目wechat

  2. 2. 拖入编译出来的sqlcipher.dll

  3. 3. 双击载入codebrowser, 提示自动分析，点击否

  4. 4. File-Load PDB file-选择sqlcipher.pdb-Load

  5. 5. 等待右下角进度条分析完成

  6. 6. 点击左上角保存按钮，并退出codebrowser

  7. 7. 回到ghidra界面后，点击sqlcipher.dll，右键-Export-选择Binary binexport即可

在使用ghidra执行上述操作时，发现其还原函数名存在BUG,
也是这个原因困扰了半天，一直以为是插件或者bindiff或者配置不当，后面还是提了issues后得知官方已知晓该问题并将在新版本中修复：https://github.com/NationalSecurityAgency/ghidra/issues/4883
，(lll￢ω￢)

### 3\. 分析wechatwin.dll

  1. 1. 拖入`C:\Program Files (x86)\Tencent\WeChat\[3.8.1.26]`下的wechatwin.dll到ghidra的wechat项目

  2. 2. 双击载入 codebrowser, Analysis-auto analysis, 等待右下角进度条分析完成，点击左上角保存按钮，并退出codebrowser

  3. 3. 回到ghidra界面后，点击wechatwin.dll，右键-Export-选择Binary binexport即可

### 4\. 所谓关键函数

我们的目标是获取密钥，那么需要看下哪些函数的参为密钥，这样在动态调试在这些函数下断即可拿到密钥。

看到官方demo或者直接询问chatgpt:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191507.png)

其实不仅是上述，例如sqlite3_key的上层封装或者其下层定义中均会接触到key，接下来看源码，尝试都找出来,

以sqlite3_key为例，找出其上层封装:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191508.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191509.png)

在sqlcipher_check_connection和sqlcipher_codec_ctx_migrate下断都能跟踪到密钥，当然可以看出密钥是放在了ctx结构体下，继续查找ctx->pass相关操作，可以找到更多操作了密钥的函数,
不一一例举:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191510.png)

接下来找下层定义:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191511.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191512.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191514.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191515.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191516.png)

操作结构体ctx的pass属性的调用链上的函数都可以作为关键函数。

上面简单分析的了sqlite3_key函数上下游会涉及处理密钥的函数，这些函数可以记录下来，作为符号还原的关键函数，在这些函数上断点都可以拿到密钥！

### 5\. 利用bindiff还原关键函数

操作过程:

  1. 1. 双击打开bindiff，创建新项目，Diffs-New Diff-选择导出的wechatwin.dll.binexport和sqlcipher.dll.binexport，点击diff

  2. 2. 搜索上面的关键函数，有匹配的都可做对应的下断观察，发现并不准确

![](https://gitee.com/fuli009/images/raw/master/public/20230308191517.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191519.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191520.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191521.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191522.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191523.png)

3. 利用该方式还原的符号未成功找到关键函数, 有几种可能，加壳混淆、魔改，但是有些非关键函数还是非常之准确，这儿可以有的思路是通过先准确还原部分符号后，利用关联关系比如同在某个函数下，即该函数处理过密钥或者ctx结构体，进而慢慢`回溯`到关键的链上去

![](https://gitee.com/fuli009/images/raw/master/public/20230308191524.png)

  3.   

## 六、回归特征

上面符号还原的方式并未直接定位到预期的函数位置，但并不是没有帮助，也提出相关的思路，这儿先回归原始方式，尝试利用字符特征定位关键位置。

### 1\. 找到sqlite3_exec

sqlite3_exec函数是执行SQL语句的关键函数，还原其符号与地址即可绕过解密步骤直接执行SQL语句。

  1. 1. 查找字符串，找select关键字，下断 (在引用功能下需要多下断，因为无法提前知道会断在哪个查询语句的，多下断的概率就高)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191525.png)

  2. 2. 计算下偏移: Ghidra基址+(当前地址-wechatwin.dll基址) = `python3 hex(0x10000000+(0x69730F64-0x689D0000))` = 0x10d60f64, 进入Ghidra查看：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191526.png)

    1.   

到底是`FUN_10d8b440`或是`FUN_11BB0750` , 看到源码对于sqlite3_exec的定义

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308191527.png)

    2. 可以看到首参是db指针，FUN_10d8b440明显就不像，双击进去看伪C执行流程也不像

  

  3. 3. 看到图中的特征匹配情况, 确定FUN_11BB0750即sqlite3_exec

![](https://gitee.com/fuli009/images/raw/master/public/20230308191528.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191529.png)

### 2\. 找到sqlite3_open

看到源码：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191530.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191531.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191532.png)

利用特征字符串`unable to delete/modify collation`, 找到：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191535.png)

计算偏移：Ghidra基址+(当前地址-wechatwin.dll基址) = `python3
hex(0x10000000+(0x6A5AD5E3-0x689D0000))` = 0x11bdd5e3

在`FUN_11bdd540`下，查找下交叉引用：

![]()

基本确认找到openDatabase函数:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191536.png)

至此还原`FUN_11bdde90`符号为openDatabase.

当然这儿还有一个行为特征也是非常有意思的, 既可以观察`C:\Users\xxx\Documents\WeChat
Files\wxid_XXX\Msg`目录下是否突然新增`.db-shm`和`.db-
wal`文件，代表数据库被打开并执行了未commit的事务，且未执行close动作。

### 3\. 尝试找到sqlite3_key

看到ChatGPT提供的demo:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191537.png)

通过`逼近法`，可以知道sqlite3_key会在sqlite3_open之后，在sqlite3_exec之前：

方式1: 通过open找到db指针，跟踪其调用情况 -- 技术原因未实现

方式2: 执行到sqlite3_open后步进，或首次执行到sqlite3_exec后回溯堆栈

尝试用方式2找sqlite3_key:

  1. 1. 在sqlite3_open和sqlite3_exec下断点

  2. 2. 执行到sqlite3_open断下后，点击`调用堆栈`找到上层

![](https://gitee.com/fuli009/images/raw/master/public/20230308191538.png)

  3. 3. 双击上图中位置，即来到上层函数范围，且对应指令即sqlite3_open执行完后第一条被执行的指令，这儿我们下断

![](https://gitee.com/fuli009/images/raw/master/public/20230308191540.png)

  4. 4. 接着，点击`跟踪`,右键`启用运行跟踪`, 然后上方菜单栏，跟踪-自动步过

  5. 5. 看到中间只执行了一次函数调用

![](https://gitee.com/fuli009/images/raw/master/public/20230308191541.png)

  1. 6. 重复，在`6B6A5450`断点后，首先点击`步进`, 再次开启跟踪，再次自动步过

  2. 7. 这儿会跑非常久,运行的指令数超几千条，通过观察可以知道这部分逻辑实际在vmp壳下

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308191542.png)

  3. 8. 由于技术有限，通过正向的分析vmp没什么突破点，但是断在sqlite3_exec查看上层和sqlite3_open居然是一样的，那么可以推测的是执行这些下层函数的上层函数应该是做了vmp处理的，类似于在一个循环内边执行边还原部分指令

  4. 9\. 当然也对vmp上层的函数做了简单的分析，并没有好的突破点：`openDatabase函数  
vmp区域  
696F81E2  0x10d281e2  
6975C913  0x10d8c913  
6973B193  0x10d6b193  
694D2D97  0x10b02d97`

### 4\. 找到处理密钥的函数

  1. 1. 查找字符串，找pass,key等关键字，发现有`DB_KEY_STING`和`setDBKey`

  2. 2. 跟进下`setDBKey`, Ghidra基址+(当前地址-wechatwin.dll基址) = `python3 hex(0x10000000+(0x6969DDBD-0x689D0000))` = 0x10ccddbd

![](https://gitee.com/fuli009/images/raw/master/public/20230308191543.png)

  3. 3. 答案是`FUN_10dbce50`才是真正将密钥赋值到特定指针的操作，原因如下:

    1. 原因1: 跟进函数`FUN_10dbce50`回发现存在内存拷贝赋值等操作

![]()

    2. 原因2: 当动态调试断点在`FUN_10dbce50`时，看到其第一个参数的数据即密钥

![](https://gitee.com/fuli009/images/raw/master/public/20230308191544.png)  

很好，我们找到了一个处理密钥的函数

### 5\. 从sqlite3_open(openDatabase)回溯到sqlite3_key

但还不想止于此，上面的过程存在很多偶然性，无论如何都想看看sqlite3_key的真面目.

  1. 1. 静态分析sqlite3_open( `FUN_11bdde90`), 点击交叉引用

![](https://gitee.com/fuli009/images/raw/master/public/20230308191546.png)

  2. 2. 我们跟进到`int FUN_11b622f0(void)`, 继续查看交叉引用，只有在`FUN_11b62b90`内被调用过

  3. 3. 比较有意思的是`FUN_11b62b90`存在特别多特征字符串, 通过大量比对工作，可以确认`FUN_11b62b90`即函数`sqlcipher_codec_ctx_migrate`

``

    1. ![](https://gitee.com/fuli009/images/raw/master/public/20230308191547.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308191548.png)

  4. 4. 通过sqlcipher_codec_ctx_migrate的交叉引用，可以快速定位上层函数:`FUN_11bab2e0  sqlite3Pragma  
|v  
FUN_11b5e8a0  sqlcipher_codec_pragma  
|v  
FUN_11b62b90  sqlcipher_codec_ctx_migrate`

  5. 5. 此外sqlcipher_codec_ctx_migrate、sqlcipher_codec_pragma和sqlcipher_codec_ctx_migrate下层函数有 `sqlcipher_codec_ctx_get_pagesize`,`sqlite3BtreeSetPageSize`等在sqlite3_key调用链上出现的函数(当然还有很多其他函数，这儿仅本人用到的思路)，那么可以明晰，即确认这两个函数之一的符号，那么就可以通过交叉引用推导出sqlite3_key

  6. 6. 首先，尝试`sqlcipher_codec_ctx_get_pagesize`, 其在`sqlcipher_codec_pragma`下，源码表现为:

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308191549.png)

  7.   

静态观察为, 是一个数组的偏移值:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191550.png)

  8.   

  9. 为什么不是一个函数呢？我们看下`sqlcipher_codec_ctx_get_pagesize`的定义:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191551.png)

  10. 是的，其只是为了访问结构体成员变量，因此在编译过程中被简化， 因此想要找出其在逆向下的函数定义就不可行

  

  11. 7. 接着，尝试找出`sqlite3BtreeSetPageSize`,其在`sqlcipher_codec_ctx_migrate`下被调用，源码表现为:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191552.png)

    1.   

看到逆向代码, 即确认`FUN_11b66180`为 sqlite3BtreeSetPageSize:

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308191553.png)

  12. 8. 通过`sqlite3BtreeSetPageSize`的交叉引用找`codec_set_btree_to_codec_pagesize`即`FUN_11b5e830`

  13. 9. 再通过`codec_set_btree_to_codec_pagesize`的交叉引用找到`sqlite3CodecAttach` 即`FUN_11b60350`

  14. 10. 通过`sqlite3CodecAttach` 理论上应该是能找到sqlite3_key，但申请的是居然又回到了`FUN_11b622f0`

![](https://gitee.com/fuli009/images/raw/master/public/20230308191554.png)

  15.   

我想应该是藏在了unreachable block这部分也是vmp边执行边还原的代码逻辑吧，买个坑，以后技术上去再填

  

  16. 11. 无论如何找到`sqlite3CodecAttach`即`sqlite3_key`的核心，看到源码:

![](https://gitee.com/fuli009/images/raw/master/public/20230308191555.png)

  17.   

在动态调试时对其下断，成功读取到密钥:

  

![](https://gitee.com/fuli009/images/raw/master/public/20230308191557.png)

PS:

  * • 在`sqlcipher_codec_ctx_migrate`有直接用到`sqlite3CodecAttach` ，没必要向上面一样先找`sqlite3BtreeSetPageSize`，当然我也是后面审查代码才发现这一点，这也说明逆向工作的方向和思路是多种的，不要局限思维

  * • 另外sqlcipher_codec_ctx_migrate的参数为ctx结构体，而结构体有成员变量pass,即不需要跟踪到sqlite3CodecAttach，也可以直接从结构体拿到密钥

  *   

### 6\. 编写读取密钥工具

参考http://tttang.com/archive/1665/思路，在密钥内存地址上下文找特征字符串，通过计算偏移即可快速拿到偏移：

    
    
    import XXX  
      
    XXXXXXXX

###

## 七、尝试回溯密钥生成算法

  1. 1. 利用上面的脚本，我们可以在密钥数据被写入到地址之前准确预判出地址值

  2. 2. 对地址值下硬件写入断点

  3. 3. 程序跑起来，此时断点在内存拷贝的动作上，其将密钥值从另一个位置写过来

  4. 4. 查看调用堆栈，在用户代码的最上层函数下断

  5. 5. 然后我们写出如下代码，并执行, 具体暂不放出

  6. 6. 发现在该线程最上层，即最初执行态，密钥即已经存在于内存

![](https://gitee.com/fuli009/images/raw/master/public/20230308191558.png)

该线程最上层可以看到thread等标志，对应的是`call esi`:

  

  7. ![](https://gitee.com/fuli009/images/raw/master/public/20230308191559.png)

  8. 7. 接下来就是要调试多线程，在扫码登录PC微信的过程中，在拉起上述线程之前，有特别多标志性的动作，通过process monitor可以看到有注册表，网络等，可以进入kernelbase符号表，搜索`getcomputername`,`socket`等常见函数下断，发现还是找不到关键线程, 尝试制造异常或者脏数据没有好的切入点

  9. 8. 于是沉下心观察esi的情况，基本就是重复执行下面两个函数，其后key就会出现在内存`695214E0 FUN_10b514e0-> FUN_10b4d720 -> "startSubProcess. m_isRpcStarting=%d";"RpcMgrBase::startSubProcessInThread"  
                 
69E4D7D0 FUN_1147d7d0-> FUN_114791a0 ->
"XPlugin::PluginUpdater::Execute::<lambda_1>::operator ()"`

  10. 9. 常识判断应该就是初始化过程中通过RPC进程调用，把密钥计算出来再放进主进程内存空间

  11. 10. 比较有意思的是当UAC过不了时会通过执行shellExecuteExW拉RPC进程, 如果有执行这个分支条件的，可以下断然后看下完整执行命令

![](https://gitee.com/fuli009/images/raw/master/public/20230308191600.png)

  12. 11. 调试RPC需要用到这个工具，需要根据当前主机环境编译，参考: https://itm4n.github.io/fuzzing-windows-rpc-rpcview/

  13. 12. 看来要找到密钥生成算法还有比较大的工作量，先到此

  14.   

## 八、补充常用调试技巧

  1. 1. 若不想计算偏移而直接用xdbg的地址，可以修改Ghidra的基址为wechatwin.dll基址: (比较耗时且需要多次重启调试时可以暂不设置)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191601.png)

  2. 2. wechatwin.dll基址查看技巧:首先符号表进入dll内存空间, 在内存布局中转到即可找到基址

  

  3.      ![](https://gitee.com/fuli009/images/raw/master/public/20230308191602.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230308191604.png)

  4. 3. 合并DAT展示https://github.com/NationalSecurityAgency/ghidra/issues/5033

  5. 4. 不退出微信程序，且可反复从头执行，避免基址变化选择退出登录后，重新扫描，这样程序实际还是原来的，但是会走初始化的这部分业务代码![]()

  6. 5. 执行到非wechatwin.dll模块快速出来，观察xdbg最上方会提示当前PID和模块，发现非目标模块点击`运行到用户代码`而不是`运行`，原因是某些系统dll下点击`运行`会卡死而不是运行回用户空间，还有一种解决卡死的方式，点击线程，双击主进程，点击`运行`也可

  7. 6. 不进入ntdll等系统模块下的异常断点: 选项-选项-事件-取消系统断点，即可

  8. 7\. Ghidra下CTRL+E反汇编为伪代码显示

  9. 8. 找内存特定数据的地址建议用脚本做，比cheatengine效果好，cheatengine会偷偷给装一款`RAV`的杀软，挺无语，还好是在虚拟机下

  10.   

## 九、总结

本文完成如下：

  1. 1. 成功: 确定了PC微信使用sqlcipher依赖，并准确到版本号

  2. 2. 成功: 还原出sqlite操作的关键核心函数的符号:`FUN_11b62b90  sqlcipher_codec_ctx_migrate  
FUN_11b5e8a0  sqlcipher_codec_pragma  
FUN_11bab2e0  sqlite3Pragma  
FUN_11b66180  sqlite3BtreeSetPageSize  
FUN_11b5e830  codec_set_btree_to_codec_pagesize  
FUN_11b60350  sqlite3CodecAttach  
FUN_11BB0750  sqlite3_exec  
FUN_11bdde90  openDatabase`

  3. 3. 成功: 实践通过两种方式获取密钥

  4. 4. 成功: 通过脚本工具直接从内存提取密钥

  5. 5. 现有成果，可以额外达成:

    * • 利用关键函数地址直接调用执行数据库操作,绕过加解密动作

    * • 直接读取结构体数据

  6. 6. 接近达成: (还有些坑没踩完，以后填坑)

    * • 密钥生成算法

动手实践的过程学到了很多姿势, 前前后后也是花费大量的时间，还好有各位大佬无私分享的各种技术与技巧，在此特别感谢chuj师傅, respect!

## 十、参考

  * • Wechat資料庫加密逆向工程分析: https://www.osslab.com.tw/how-decrypt-wechat-sqlite/

  * • 绕过加密访问微信SQLite数据库: https://blog.51cto.com/u_12888393/2463972

  * • 微信数据库解密算法: https://codeantenna.com/a/2pKEz8O1QO

  * • 非常棒的微信逆向：https://cjovi.icu/software-testing/1650.html

  * • 看雪逆向思路: https://bbs.kanxue.com/thread-251303.htm

  * • 多个二进制分析工具各项性能进行的分析与比较: https://www.cnblogs.com/linuxsec/articles/11703039.html

  * • bindiff学习: https://ihack4falafel.github.io/Patch-Diffing-with-Ghidra/

  * • cl生成调试信息的参数差异: https://developer.aliyun.com/article/41774

  * • RPC学习1：https://xz.aliyun.com/t/11313

  * • RPC学习2：http://blog.nsfocus.net/win10-rpc/

  * • 还原符号表的方法1: https://xeldax.top/article/IDA_BINARY_SYMBOL_DIFF

  * • 还原符号表的方法2: https://zhuanlan.zhihu.com/p/371432741

  

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

