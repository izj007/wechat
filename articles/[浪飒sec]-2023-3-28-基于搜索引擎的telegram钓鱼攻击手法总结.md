#  基于搜索引擎的telegram钓鱼攻击手法总结

[ 浪飒sec ](javascript:void\(0\);)

**浪飒sec** ![]()

微信号 langsasec

功能介绍 浪飒，共建网络安全

____

___发表于_

收录于合集

以下文章来源于王小明的事 ，作者流水账小明

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM6N4HJ3lM50t9ezjYibKYaVMIhCF1KZZzDM7tHD1AIqibBQ/0)
**王小明的事** .

一个脚本小子的自我修行。

目前该攻击类型的主要受害者是tg中文用户，固然tg中的坏人含量相当高，但是对于像金融、贸易、区块链等这些带有跨境业务性质的甲方安全来说，个人认为这是一个不容忽视的风险点。今天能替换你员工剪贴板，明天就能给你域控下发个
lock.exe 天下大乱。

换了个方向写流水账，欢迎拍砖。

  
  

## TL;DR

![]()

搜索引擎tg钓鱼手法简要思维导图

## 搜索引擎分类

### Google

目前Google是该攻击类型的主流平台，因为telegram的使用需要XFW以外的网络，也就意味着具备该网络条件的用户一般也具备浏览谷歌的条件。截至目前为止的相当长一段时间内，直接在Google搜索
“telegram” 相关的关键字，返回的搜索结果中的前1-4条一般都会带有 “AD” 或 “赞助商”
这种标识，这些广告结果当中绝大多数都是有所图谋的恶意攻击者故意投放的 “有毒” 广告。

![]()

### Bing

Bing搜索引擎也具有付费广告服务，但是使用bing搜索引擎直接搜索 “telegram”、“纸飞机”
等关键字的话，得到的页面是被过滤和筛选后的，猜测是因为政策原因导致搜索结果均是新闻网页。

![]()

但是该屏蔽政策似乎没有兼顾所有的关键字，使用关键字 **“电报”、“电报中文版”**
等关键字仍然可以得到真实的搜索结果，其中包含广告结果以及使用SEO等手段将排名做的很靠前的假冒官网。

![]()

## 广告投放筛选条件

详细分类的话，广告投放大致分为地理位置、语言、意向群体、关键字这几类。

![]()

但是站在自动化搜寻并封禁的角度，我们的方向可以有：语言、IP归属、地理位置、关键字、时段、搜索次数、客户端UA等，尽量满足这些条件去主动迎合恶意攻击者的投放群体画像，以此来达到最佳的封禁效果。

下面仅举例几个对搜索结果影响较大的关键要素，不一一赘述。

### 语言

简体中文搜索结果

![]()

繁体中文

![]()

英文搜索结果

![]()

### 关键字

以telegram中文版为关键字

![]()

以纸飞机中文版为关键字

![]()

以电报中文版为关键字

![]()

### 客户端

直接展示移动端的搜索结果，根据部分搜索结果的网站标题可以看出，其目标群体为安卓用户，这些搜索结果使用PC桌面端的user-agent是无法触达的。

![]()

### 位置

Google的广告投放按照位置来区分可以简单的分为顶部和底部两种，顶部的广告展示位于真正的搜索结果之上比较引人注目，转化率可能会相对较好，但是容易被人忽略的底部广告位也存在telegram钓鱼攻击广告投放的情况。

![]()

  

![]()

## 中转页面

中转页面即为搜索引擎给出的搜索结果，攻击者一般会围绕 ”白利用“
的思想采用各种手段保证中转页面的合法性，下面介绍的是结合日常安全运营工作当中发现的主流中转手法。

### YouTube频道页面

YouTube是最传统、在搜索结果中最常见的中转方式，从最初出现到现在可能达到数年之久。

![]()

目前为止基本只有一种形式，即创建一个名称为 “telegram中文版”
相关的YouTube频道，频道的背景、介绍话术、头像、各种图标ico等均设置为telegram相关，给搜索者一种telegram官方制作的推广页面的感觉。

然后频道详情页面下方的超链接被设置为恶意的链接，以此引导用户浏览假冒的telegram官网等页面，直至用户下载安装假冒的telegram客户端。

![]()

  

![]()

#### 频道链接指向问题

在以往的应急过程中，大多数情况下频道页面下面的6个超链接对应的跳转链接是一致的。

需要注意的是，虽然不是很常见，但是有时会遇到六个超链接对应的跳转链接不一致的情况，所以手动排查和自动化搜集时应该注意这个问题避免遗漏。

![]()

#### YouTube搜索频道

YouTube支持根据关键字搜索频道，不过搜索结果当中作品和频道是混在一起的。根据这个思路可以直接通过搜索关键字的形式找到这些假冒的频道页，再提取详情介绍里面的超链接，批量封禁。

并且根据实际观察到的情况来看，大多数实际触达到用户的频道基本都是提前建立的，提前时间在几天到一两个月不等。个人猜测攻击者准备资源、通过审核可能也需要一定的时间，所以这种方式封禁域名有一定的预判作用，时效性的效果可能会好一些。

![]()

### Google文档

Google文档即为谷歌官方提供的在线文档服务，使用Google文档作为中转页面的手法出现时间相对较短，不过也是目前出镜率相对较高的一种中转方式。

搜索相关关键字时，可以看到推广域名是 docs.google.com。

![]()

链接对应的实际就是攻击者设计好的谷歌在线文档，文档假冒telegram推广相关主题，下载文本的超链接同样被设置为恶意地址，诱导用户继续访问假冒telegram官网或直接返回恶意telegram客户端的下载地址。

![]()

### Google Site

该攻击方式的思想与Google文档相似，均是利用Google提供的第三方服务作为中转恶意地址的 “白名单” 基础设施。Google
Site本身为Google的一款以Wiki为基础的在线网站制作系统，为Google Apps的一部分，一般被用来搭建基础的展示网页。

搜索相关关键字时，有时可以得到域名为 ”sites.google.com“ 的搜索结果。

![]()

点击链接后，可以发现攻击者使用Google sites的服务创建了一个仿冒telegram官网的web站点，最醒目处的两个按钮就是恶意客户端的下载地址。

![]()

上图展示的是为Windows PC准备的钓鱼网站，针对移动端的钓鱼攻击同样也有使用Google sites作为中转页面的案例。

![]()

### 搜索引擎语法

接触过信息安全的人可能都对Google hacking不陌生，因为早几年网上的教程基本都是从 ”信息搜集“ 开始讲，信息搜集教学当中经常就会带有Google
hacking语法教学这种环节，其中比较常见的大概是 inurl、filetype、site 这些语法，其中site、inurl
语法都有筛选指定的网站搜索结果的功能，并且比较通用的语法基本对大型的搜索引擎都是适用的。

#### Google 搜索语法

当发现搜索结果中的推广地址为 www.google.com 的时候，对应的就是攻击者利用谷歌搜索语法的结果。

![]()

点击推广的结果后，可见链接地址对应的是针对某个特定的假冒telegram官网域名的搜索引擎语法搜索结果，攻击者利用inurl语法使telegram只展示位于该假冒域名下的url搜索结果，受害用户不管点击哪个搜索结果，最终都将在攻击者的假冒官网上下载假冒的客户端。

![]()

#### Bing 搜索结果

搜索结果中可以看到域名为 www.bing.com

![]()

点击后发现攻击者同样是想通过site语法来限定搜索结果为特定的假冒官网地址

![]()

但是由于上文提到过的疑似特殊原因，使用Bing中转似乎不是一个很稳定的选择

![]()

### 直接跳转假冒官网

有时也能看到攻击者直接将自己搭建的假冒官网作为推广地址的案例

![]()

用户在搜索页面点击搜索结果后，会直接跳转到假冒官网地址

![]()

假冒的telegram官网域名当中绝大多数围绕 ”telegram“ 这个关键字，但是也能见到少数没有规律的域名作为推广结果的案例。

![]()

### 小众搜索引擎

像 www.discovertoday.co 、hk.top10quest.com 这类的域名具备搜索引擎的功能，但是又没有听说过的网站，我暂且称之为
”小众搜索引擎“，该类网站的特点是 ”也具备广告位的设定，并且广告特别多“。

![]()

Google搜索钓鱼广告中也曾出现过利用这些 ”小众搜索引擎“ 二次跳转的案例。

![]()

  

![]()

  

![]()

进入此类搜索引擎的搜索页面后，位于顶部的仍然是广告投放的恶意推广结果。

![]()

  

![]()

### 印象笔记

跟踪过程中也曾经发现印象笔记被攻击者短暂的利用过一段时间，即 www.evernote.com
的域名会出现在推广广告列表中。但是不知道出于何种原因，现在基本已经看不到这种中转方式了。

![]()

利用思路与Google文档类似，都是利用文档服务制作一个假装推广telegram的展示页面，诱导用户点击下载链接。

![]()

### telegram关键字网站

该类型搜索结果与广告投放无关，一般出现在除广告推广结果、真实telegram官网之外的位置。

![]()

此类假冒网站基本可以分为两种工作原理：

  1. 网站本身主打的就是 ”telegram官网“，通过各种SEO方式将网站排名优化的特别靠前，如果用户直接忽略掉了推广结果浏览下方搜索结果，或者用户使用了广告屏蔽浏览器插件等，就很容易点击进入此类假冒官网下载假冒的客户端。

![]()

  2.   

网站定位是主要围绕 ”telegram“
这个关键字的资讯类网站，本身并不提供下载服务，但是很容易被搜索相关资料的用户点进去，网站上一般在顶部或者其他位置放置真正的假冒官网的跳转地址，诱导用户点击。

  

  

  

  

![]()

  

![]()

### 被SEO利用的无关站点

这种情况多数是站内的search功能被黑帽SEO滥用了，最终效果就是大型的网站被利用来做黑灰关键字的推广。

![]()

### 疑似Google商店

在观察排名靠前的几个telegram关键字时，发现 “纸飞机群组” 这个关键字可以搜索到一个排名比较靠前的Google Play的结果。

![]()

软件icon跟名称来看，主打 “中文”、"福利"，并且该软件似乎也只是谷歌应用商店当中上架的非官方app之一。

![]()

关于该软件的比较有意思

![]()

![]()

![]()

![]()

## 中转页面跳转

用户在搜索引擎给出的搜索结果中点击了不可信的内容后来到中转页面，中转页面一般还会有至少一次跳转，中转页面的跳转涉及多个维度的信息，概括如下。

### 自建假冒官网

攻击者搭建的假冒telegram官网是该类型钓鱼攻击当中的主战场。中招的用户将在该页面下载假冒的客户端，走完成为受害者的最后一步；防守者也将提取官网的域名、文件下载的地址进行告警和封禁。因为前面的几个步骤当中，攻击者利用的资源都是白名单资源，防守者无法有效的拦截和封禁，只有像
”假冒官网“、”下载地址“、"C2地址" 这些自定义的东西才具备针对性治理的条件。

攻击者搭建的假冒官网在视觉上一般会模拟真实官网的形态，偶尔也有一些是自创的UI，但是最终目的一定是设法让用户点击恶意客户端的下载链接，实现钓鱼。

![]()

#### 假冒官网域名特征

假冒官网的标配就是模拟 ”telegram“ 这个关键字的相似域名，攻击者的手法基本围绕
”冷门后缀“、”字母移位“、”单词缺字“、”重复字母“、”相似字母替换“、”加单词造域名“ 这些，此处不赘述。

![]()

不过需要注意的是，有几个telegram官方的域名不是很常见，有时候会混杂在攻击者的假冒官网当中，需要关注下避免误封的问题。

    
    
    telegram.com  
    cdn4.telegram-cdn.org   
    updates.tdesktop.com   
    web.telegram.org  
    

##### 在线快速建站服务

甲方一般都遇到过使用ipfs域名、国外快速建站域名等服务发钓鱼邮件的，一般这种域名都是个壳，实际收信的地址在远端。fake tg 同理，例如：

![]()

#### 客户端选择

进入官网后一般就是客户端的选择下载，比较常见的大多数情况是攻击者主打Windows系统的假冒客户端，所以剩下的几个系统的下载地址均设置为跳转telegram官方的下载地址。

![]()

或是攻击者主打安卓系统的假冒客户端，网站尺寸都仅为移动端设置，除安卓下载地址外其他系统下载地址返回官方下载地址。

![]()

剩下的情况基本就是无论点击哪个类型的下载按钮，返回的都是攻击者设置好的下载地址，文件类型都是固定的。

![]()

另外分析过程中有遇到过比较特殊的情况，攻击者为IOS系统也准备了相应的假冒客户端，下载地址是自签app的托管页面，会要求用户信任描述文件。

![]()

![]()

![]()

#### 下载功能

受害者选择完客户端后，流程就到了下载部分。攻击者在这部分的工作主要是bypass或者保证存活、方便分发之类的工作，此处简单阐述几种情况。

##### 直接超链接

  1.   

最常见的情况就是下载按钮直接对应超链接，这种方式用户在正常的浏览形态下，把鼠标放到按钮上即可看到下载链接，只是基础的下载功能，不具备什么隐蔽性。

![]()

  

像这种跟上图中的也区别不大

  

  

![]()  

##### 隐蔽超链接

  1. 还有一种情况是鼠标放上去看不见下载地址，大概分为两种设置：下载链接也在前端只不过对应的是js的事件、用户点击后后端再返回下载地址。

![]()

##### 特殊下载域名

  1. 下载功能使用了特制的下载域名，封禁角度来说就多了一个封禁的点。

![]()

![]()

##### 特殊下载页面

  1. 经常可以看到一款自定义的下载页面，会在返回给用户下载地址后关闭，猜测可能是攻击者分发文件或者bypass chrome下载功能的手段。

![]()

##### 条件检查

  1. 假冒官网首页反调试

![]()

  2. 跳转专用的下载域名

![]()

  3. 直接访问下载地址返回404

![]()

  4. 分析后发现Windows的UA无法得到真实的下载地址

![]()

  5. UA为安卓客户端时才会返回正确跳转地址，攻击者采用这种方式来保护自身下载地址。

![]()

### 域名随机分发

实际在真正到达攻击者自己搭建的假冒telegram官网之前，用户点击中转页面的跳转链接后，也存在bypass的细节。

曾经在分析过程中发现点击假冒官网链接后， 前后两次得到的官网地址不唯一，后来发现攻击者采用了一种随机分发域名的思路，以此来保证存活率。

![]()

### 文件托管

文件托管部分理论上应该是在自建假冒官网部分中的，但是由于有时Youtune这些中转页面下部的超链接对应的就是文件托管地址，没有假冒官网这层中转步骤。以及文件托管部分也具备一定的bypass功能，所以单独在这里介绍。

#### OSS下载链接

从下载链接的维度来说的话，除攻击者自己申请的下载域名外，最常见的就是使用阿里云OSS来托管恶意客户端。

![]()

  
  

![]()

  

![]()

除阿里云oss服务外，也有使用腾讯云oss、AWS oss服务的案例。

![]()

![]()

#### 网盘等白服务

攻击者使用oss托管文件的方式虽然是使用了第三方的服务，但是oss域名绝大多数都是唯一的域名地址，所以仍然可以进行劫持或者阻断。但是有些案例当中，攻击者使用了网盘等白服务的下载直链作为下载地址，地址中的域名是服务方的不唯一域名，封禁的话就可能会影响正常业务。

##### onedrive案例

点击下载按钮后，返回了OneDrive的下载链接。

![]()

##### Filebin案例

filebin是公用的文件托管服务，此处返回的下载地址是 filebin.net

![]()

  

![]()

#### HFS等非主流渠道

有时也能够看到下载地址是基于纯IP开放的web服务，这种情况多半是攻击者自己搭建的HFS服务。

![]()

  

![]()

另外一个案例

![]()

  

![]()

##### 特征狩猎

像HFS这种情况，服务的特征还是相当明显的，我们可以使用使用FOFA等互联网资产搜索引擎来过滤出被采集到的恶意服务。

该搜索结果风格基本是单服务、单IP。

![]()

该搜索结果风格是多域名、单IP，可见攻击者比较偷懒，注册了多个垃圾域名均解析到了单个IP开启的服务。

![]()

当然了，直接根据关键字去匹，也能匹配到不少假冒官网。

![]()

## 客户端类型

客户端类型相关的信息量较少，需要注意的基本只有格式、命名两点。文件格式变换可能是出于免杀、bypass
chrome下载文件安全提醒考虑，安装包文件名则可能主要用来欺骗用户。本文更侧重于自动化封禁处理的角度，这里不再赘述。

![]()

## 常见C2特征

日常分析对C2接触不算多，此处仅介绍几个接触过的比较有代表性的场景：

  1. 常见恶意客户端的使命无非两个：释放出中文版telegram给用户使用、运行木马。目前最常见的木马运行方式即为 “白加黑”，攻击者一般会选用迅雷、某些游戏客户端等的exe组件作为加载恶意dll的载体。

  2. 对于比较传统的MSI后缀流派，7z的分析效果比较好（要看具体场景）。以某次直接释放MFC木马的假冒telegram客户端为例：

![]()以某次使用 “白加黑” 手法的Firefox钓鱼客户端为例（很多假冒tg客户端也是同样的手法）：

![]()

  3. C2木马具有复杂性提升的发展趋势。最初比较常见的木马释放流程就是安装包释放出loader，loader运行加密的shellcode，但近期分析的木马却更加复杂一些。

  

  

下图案例中的exe安装包将释放出msi安装包、压缩文件包、PE文件，msi安装包释放出downloader（downloader疑似也是一个被恶意利用的组件）。

  

![]()

downloader在有道云笔记相关的分享链接下载加密的压缩文件 A.jpg

![]()

  

  

![]()  

压缩文件下载到本地后，安装程序解压出被分割的恶意loader
dll文件，本地拼接后白加黑将loader运行起来，再解密并动态加载之前释放出的加密的恶意动态链接库。

  

木马运行后通过添加防火墙规则、复制多个自身、添加计划任务等手段进行权限维持。

![]()

并且该恶意客户端采用了大量的反沙箱设置，整个安装过程就长达五分钟。![]()

  4. 部分木马具有 “牺牲部分上线率来提升存活率” 的思想。以下案例中的恶意客户端在用户安装完毕后，并不会完成木马的上线工作。而是将用户安装后的telegram的快捷方式修改为木马上线的命令行，用户在第一次运行假冒telegram的时候才会将恶意木马解压运行起来。![]()

  5. 木马多样性趋势。最初在了解范围内的木马基本无外乎 Cobalt Strike、大灰狼 这些传统远控（病毒特征码FatalRat、Zegost这些），后来有遇到过之前没有见过的websocket协议木马。![]()

## 攻击者目的

目前该攻击方式最常见的攻击目的即替换受害者剪贴板中的钱包地址实现盗币，但是在日常分析和其他分析报道中也能看到其他目的的攻击者。

![]()

## 其他信息

### 安全厂商报道

目前已有多家国内安全厂商进行过报道，媒体类型以微信公众号为主。前段时间ESET也发表了一篇较为详尽的文章来介绍 “针对东南亚和东亚的虚假安装程序”
攻击事件，其中最有代表性的即为假冒telegram。

![]()

![]()

原文地址：https://www.welivesecurity.com/2023/02/16/these-arent-apps-youre-looking-
for-fake-installers/

### 相似攻击手法的其他软件

除telegram外，确实还存在其他多种假冒软件的攻击，如ESET文中提到的chrome、Firefox浏览器、WhatsApp、SKYPE、搜狗拼音输入法、Electrum钱包等等。

![]()

WhatsApp

![]()

  

![]()

chrome

![]()

Line

![]()

Electrum

![]()

SKYPE

![]()

  
  

![]()

  

![]()

  

![]()

另外除上面描述的软件之外，也有其他IT类软件的攻击痕迹，例如使用 “navicat汉化版” 关键字推广的ytb频道。

![]()

目前该域名已经被cloudflare拦截

![]()

### 官方的拉黑机制

虽然攻击者所使用的基础设施的反应相对迟钝，但其实也是有所动作的。

#### chrome的钓鱼网页拦截

![]()

#### Youtube频道的封禁措施

![]()

  
  

![]()

部分之前存在过的恶意YouTube频道已经404，不确定是官方的处置措施还是攻击者更换了阵地。

![]()

#### Google文档的跳转声明

似乎并非所有  

![]()

  
  

![]()

#### 被DMCA的域名

搜索时可以看到页面底部有时会显示有些搜索结果是被DMCA移除掉的

![]()

但是不能确定这些域名是被官方移除掉了，还是被用户举报的，抑或是竞争对手干的。

![]()

### 攻击者之间的竞争

比较有趣的是，在一次分析当中，我们发现攻击者会修改用户的hosts文件，屏蔽掉部分竞争对手的假冒官网域名，以此来保证自己的 "权限独享"。

![]()

### 假冒telegram的产业链

“盗币” telegram的开发已经成为公开的黑灰项目，时常能够见到该 "项目" 的广告。

![]()

![]()

### 攻击者的推广关键字

一次分析过程中，意外发现了攻击者用于统计访问的关键字，通过关键字可以看到攻击者目标人群。

    
    
    https://ia.51.la/go1?id=21515153&rt=1676951579692&rl=2048*1152&lang=zh-CN&ct=unknow&pf=1&ins=0&vd=3&ce=1&cd=24&ds=全新telegram中文汉化版已上线，其中包括telegra&ing=3&ekc=&sid=1676951478109&tt=Telegram Telegram中文版&kw=telegram, 电报telegram, telegram中文, telegram汉化, telegram中文版, telegram下载, telegram中文版安卓, telegram中文版ios&cu=https://tenetgamg.top/&pu=https://www.youtube.com/  
    

关键字：telegram, 电报telegram, telegram中文, telegram汉化, telegram中文版, telegram下载,
telegram中文版安卓, telegram中文版ios

### 攻击者有趣的话术

不知道是否是刻意利用目标受害者的民族自豪感，攻击者在假冒安卓客户端下载处，刻意设置了一个 ”华为专用版“ 下载入口，其实背后对应的下载链接与
”普通假冒安卓版“ 是一样的。

![]()  

  

### 针对已中招用户的一个筛查思路

大多数假冒的Windows客户端在安装时基本都会在 ”程序和功能“
注册表项进行新软件注册，假冒telegram客户端程序也有自己的特征，可以在EDR、准入等会统计客户端软件列表信息的地方以排查telegram关键字、排查特殊符号等思路来筛查已经安装的用户。

![]()

![]()

![]()

![]()

![]()

### 嘿客竟在我身边

微信群里偶然看到有人咨询如何解决下载exe文件时chrome报毒的问题

![]()

文件名称命名流派属于上文提到的 ”官方写实派“

![]()

下面又在咨询谷歌快照的收录问题

![]()

以及苍白的辩解

![]()

根据以上特征可以推断出此人极大概率是个假TG站长

![]()

### 漏洞

有套模板用了低版本shiro，可以打个550。安骑士守护，值得信赖。

![]()

![]()

app分发也有

![]()

![]()

### 分享下ioc

写完有一段时间了，发现近几天谷歌封禁的速度貌似快了起来，整这些活的貌似也少了不少，遂赶紧发出来蹭个尾巴。

分享一波目前收集到的域名，共计608个，其中绝大多数是围绕 fake_tg 的官网、下载域名、C2、相似攻击ioc。

未打码版放在夏老师星际黑客文档：https://txt.xj.hk/fake_t

    
    
    f[.]nkking[.]com  
    103[.]212[.]231[.]151  
    download3[.]htfoifm[.]xyz  
    tg-telegram[.]vip  
    teleprannm[.]com  
    down[.]tg-zc[.]com  
    tg-zc[.]com  
    zhlatrst-38tgle[.]org  
    jungen[.]oss-accelerate[.]aliyuncs[.]com  
    daboluo-dl[.]netlify[.]app  
    telegrem-zg[.]netlify[.]app  
    tgdsafsagoogle[.]oss-cn-hongkong[.]aliyuncs[.]com  
    tg-ch[.]com  
    tgelegramch[.]xyz  
    telegrarn[.]eu  
    tgsgp[.]oss-ap-southeast-1[.]aliyuncs[.]com  
    zh-telegram[.]net  
    rhfudncm[.]xyz  
    telegramr[.]co  
    telegrarn[.]work  
    telepanm[.]top  
    telegram[.]kiwi  
    telegam[.]health  
    uv7zzq[.]dm[.]files[.]1drv[.]com  
    telegrarn[.]one  
    taleglam[.]com  
    www[.]talesgian[.]com  
    www[.]telegram-tgp3[.]com  
    www[.]telegram-tgp5[.]com  
    telegrarn[.]in  
    telegrann[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegarn[.]org  
    teledown1[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegrma[.]cc  
    telegarsm[.]com  
    www[.]tg-telegram[.]pro  
    download90[.]srdna[.]com  
    telegram[.]training  
    ttelte[.]oss-cn-hongkong[.]aliyuncs[.]com  
    wahadp[.]com  
    download[.]dsvcfwp[.]cn  
    www[.]tg-telegram[.]org  
    789-1306961415[.]cos[.]ap-hongkong[.]myqcloud[.]com  
    www[.]telegram-com[.]cc  
    tg-telegram[.]pro  
    telegramzh[.]cc  
    download[.]telegram[.]lgbt  
    telegramzw[.]com[.]cn  
    yyds10133[.]oss-accelerate[.]aliyuncs[.]com  
    telergam[.]live  
    122121ffff[.]oss-cn-hongkong[.]aliyuncs[.]com  
    tehegiam[.]com  
    tellegram[.]group  
    telegraem[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegraem[.]cn  
    xgtele[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegraem[.]vip  
    tlssatwla[.]com  
    telegrarn[.]pw  
    www[.]telegram-o[.]cc  
    apk[.]telegramx[.]me  
    789822[.]oss-cn-hongkong[.]aliyuncs[.]com  
    tg-zw[.]com  
    teleglam-zz[.]com  
    e80255c45d527cc415b7526391f6d9f5[.]oss-accelerate[.]aliyuncs[.]com  
    hx4fne[.]gz3k[.]world  
    k4n[.]8uft[.]world  
    w[.]kuai-lian[.]vip  
    gg[.]telenet[.]vip  
    tg-telegram[.]biz  
    tg-telegram[.]org  
    2a561b9384674c115296be162e54ee6e[.]oss-accelerate[.]aliyuncs[.]com  
    telegmas[.]com  
    xkck-dl[.]netlify[.]app  
    telegremkz[.]com  
    www[.]telegremkz[.]com  
    telegrarn[.]win  
    1231313879[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegram-zz[.]com  
    telegram-download[.]cloud  
    tpcsy[.]oss-cn-hongkong[.]aliyuncs[.]com  
    teleegram[.]art  
    www[.]telegarsa[.]life  
    www[.]telegarsc[.]top  
    telegram-download[.]fun  
    telegram-download[.]cyou  
    www[.]talagem[.]com  
    talagram[.]shop  
    zsdownload1[.]oss-cn-hongkong[.]aliyuncs[.]com  
    www[.]tellagrem[.]com  
    telegrem-zg[.]com  
    telegkam[.]com  
    telegrarmcnzz[.]com  
    telegram-1313815604[.]cos[.]ap-singapore[.]myqcloud[.]com  
    telegrmam[.]org  
    www[.]telegradm[.]com  
    teyegarm[.]org  
    www[.]telegramxi[.]com  
    570c805dc6a7845a6733f91b7196bed0[.]oss-accelerate[.]aliyuncs[.]com  
    telegroms[.]com  
    www[.]teleam[.]health  
    telegream[.]tv  
    tg-zhongwen1[.]cc  
    web[.]mjlfyqrr[.]xyz  
    www[.]telegram--download[.]com  
    www[.]telegaarm[.]com  
    telepram[.]com  
    xvsdgvgsdrbg[.]oss-cn-hongkong[.]aliyuncs[.]com  
    download69[.]srdna[.]com  
    download[.]telegramc[.]xyz  
    www[.]telegramm[.]vip  
    telegloam[.]com  
    137[.]220[.]146[.]224  
    www[.]telegramvip[.]xyz  
    telegrano[.]org  
    www[.]telegramgl[.]com[.]cn  
    telegarmd[.]com  
    zstelegram[.]oss-cn-hongkong[.]aliyuncs[.]com  
    451bf881a688a12c8ff794d089531831[.]oss-accelerate[.]aliyuncs[.]com  
    www[.]tleamaa[.]com  
    telegcrem[.]com  
    telegreng[.]com  
    www[.]tellegram[.]zone  
    tellegram[.]host  
    telgram[.]health  
    btcdl[.]netlify[.]app  
    telegarmm[.]netlify[.]app  
    29e221f1ca4868201606d3[.]oss-accelerate[.]aliyuncs[.]com  
    telegcerm[.]com  
    www[.]telegeram[.]ink  
    apk-telegram[.]com  
    telegramr[.]xyz  
    telegram-a[.]org  
    www[.]telegramm[.]ink  
    telergam[.]top  
    afsaf1[.]oss-cn-hongkong[.]aliyuncs[.]com  
    www[.]telelgrzm[.]com  
    drhhrddtery2[.]oss-cn-hongkong[.]aliyuncs[.]com  
    www[.]tleamoa[.]com  
    felegram[.]lol  
    telegarm[.]one  
    telegrarm[.]art  
    tele[.]bkve[.]cn  
    m[.]teoegram[.]com  
    www[.]buchananapp[.]com  
    app[.]buchananapp[.]com  
    pc[.]buchananapp[.]com  
    upload[.]buchananapp[.]com  
    172[.]67[.]173[.]103  
    104[.]21[.]72[.]9  
    telegramdo[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegrma[.]vip  
    dows[.]kuai-lian[.]vip  
    tenetgamg[.]top  
    tgelegramzh[.]xyz  
    telegremg[.]cc  
    telegrarn[.]cc  
    www[.]telegrarn[.]cc  
    teleprann[.]com  
    telegrmas[.]com  
    telegram30-chinese[.]com  
    telegrak[.]com  
    tele[.]kuai-lian[.]vip  
    telegrrann[.]org  
    telegramr[.]org  
    tg11184[.]oss-cn-hongkong[.]aliyuncs[.]com  
    www[.]talagren[.]com  
    e06aac69edcc89ff9b1b92e9f2528ab7[.]oss-accelerate[.]aliyuncs[.]com  
    123-1306961415[.]cos[.]ap-hongkong[.]myqcloud[.]com  
    telegarn[.]xyz  
    china-teleglam[.]com  
    telegramet123[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegramgc[.]com  
    inso-a88[.]oss-cn-hongkong[.]aliyuncs[.]com  
    www[.]telegrgn[.]net  
    qsflww[.]bn[.]files[.]1drv[.]com  
    www[.]telegrarn[.]ink  
    601f3d2aa2bda7d42f51bce0782dc81b[.]oss-accelerate[.]aliyuncs[.]com  
    telegrms[.]com  
    www[.]zhtelegram[.]org  
    zhtelegram[.]org  
    www[.]telegramzh[.]co  
    telegramzh[.]co  
    www[.]telegrbm[.]net  
    telegrbm[.]net  
    www[.]telegramom[.]org  
    telegramom[.]org  
    www[.]telegram[.]family  
    telegram[.]family  
    www[.]telegramb[.]com  
    telegramb[.]com  
    hbxvrhaw[.]bar  
    www[.]telogrem[.]com  
    download[.]teleegramvv[.]xyz  
    download[.]telegronm[.]xyz  
    www[.]telegramxe[.]org  
    telegaarm[.]ink  
    www[.]telasgram[.]com  
    www[.]telegroom[.]com  
    www[.]telegrvn[.]com  
    a1com[.]oss-cn-hongkong[.]aliyuncs[.]com  
    www[.]telegaagm[.]com  
    www[.]telegram-asd[.]com  
    www[.]telegrarn[.]us  
    www[.]telegrxam[.]com  
    www[.]telegram1[.]vip  
    www[.]telegrrm[.]net  
    www[.]teleegran[.]org  
     www[.]teleegran[.]org  
    telegraan[.]org  
    teleegran[.]org  
    download[.]telegraam[.]vip  
    download[.]telegramhome[.]org  
    download2[.]rtgvbny[.]xyz  
    teracnm[.]top  
    download[.]telegramp[.]xyz  
    telegrraam[.]cuyocxs[.]cn  
    tele1[.]ouygqmq[.]cn  
    tele[.]ebimdfg[.]cn  
    teler[.]oabuynet[.]com[.]cn  
    telegramzh[.]org  
    telegrarn[.]com  
    www[.]telegcn[.]com  
    telegrarcn[.]com  
    telergem[.]org  
    down[.]tggdown[.]com  
    www[.]telegrram[.]buzz  
    www[.]telegarms[.]vip  
    bakdownload[.]srdna[.]com  
    download[.]tellegrom[.]com  
    www[.]yobestategov[.]com  
    tellegrom[.]com  
    sites[.]pqict[.]cn  
    cri-5amnsqb5sb53590h-registry[.]oss-accelerate[.]aliyuncs[.]com  
    www[.]telegrambo[.]org  
    china-teleglem[.]com  
    www[.]telegaam[.]org  
    tghka7[.]oss-ap-southeast-1[.]aliyuncs[.]com  
    zw-telegam[.]com  
    www[.]teleagrem[.]com  
    talegramn[.]org  
    txun[.]s3[.]ap-northeast-2[.]amazonaws[.]com  
    www[.]totater[.]com  
    k-telegram[.]app  
    zh-cntelegram[.]com  
    telecom-site[.]com  
    asqpqe[.]com  
    telegrame[.]online  
    www[.]cn-teiegram[.]xyz  
    www[.]telegramsu[.]com  
    telechina[.]oss-accelerate[.]aliyuncs[.]com  
    telebisa[.]com  
    abc-telegram[.]com  
    www[.]tgstpaa[.]top  
    tgbkc1[.]oss-cn-hongkong[.]aliyuncs[.]com  
    mi-telegram[.]com  
    www[.]tegygram[.]com  
    appmobi[.]online  
    tele-lyon[.]com  
    ryxsdg8[.]space  
    www[.]telegarn[.]co  
    super-telegram[.]com  
    www[.]teleramg[.]org  
    www[.]telegrems[.]com  
    www[.]telegham[.]org  
    download[.]telergems[.]com  
    telemram[.]com  
    www[.]tleagnz[.]com  
    telencgram[.]com  
    www[.]taijuaa[.]store  
    telegarm[.]shop  
    telegracm[.]cn  
    telagrcm[.]com  
    telegrams[.]cloud  
    www[.]telegramn[.]top  
    www[.]t-telegram[.]org  
    telsgrams[.]com  
    telegrams-app[.]org  
    telengrm[.]com  
    www[.]telegrampcn[.]com  
    talagram-zh[.]com  
    www[.]tglegram[.]org  
    www[.]whatsappg[.]com  
    teleagram[.]vip  
    www[.]telegrab[.]org  
    www[.]telagad[.]com  
    www[.]telagtiem[.]xyz  
    www[.]telegrabs[.]com  
    telematica-uk[.]com  
    telegrem[.]bid  
    anyrepeater[.]com  
    www[.]lrvr[.]org  
    telegram88[.]xyz  
    www[.]teleldcn[.]com  
    telegramzhcn[.]org  
    telegramst[.]com  
    www[.]telegramst[.]com  
    www[.]tgcn[.]cash  
    www[.]telegramxiazai[.]com  
    www[.]telegram-chinas[.]com  
    www[.]telegrcm[.]org  
    www[.]telegrsm[.]net  
    download3[.]fugbnh[.]xyz  
    en-telegram[.]com  
    154[.]39[.]64[.]225  
    telegarms[.]xyz  
    27[.]124[.]34[.]177  
    www[.]luckfafa[.]com  
    210[.]56[.]54[.]12  
    microsoftdefender[.]luckfafa[.]com  
    14[.]192[.]67[.]187  
    wpsupdate[.]luckfafa[.]com  
    45[.]116[.]161[.]95  
    45[.]116[.]161[.]95   
    googleupdate[.]luckfafa[.]com  
    b[.]nkking[.]com  
    www[.]nkking[.]com  
    193[.]218[.]38[.]149  
    c[.]nkking[.]com  
    193[.]218[.]38[.]148  
    d[.]nkking[.]com  
    193[.]218[.]38[.]82  
    telergam[.]xyz  
    143[.]92[.]61[.]121  
    107[.]148[.]45[.]48  
    12-16[.]pinyin-sougou[.]com  
    107[.]148[.]35[.]6  
    occ-a6[.]oss-accelerate[.]aliyuncs[.]com  
    www[.]firefoxs[.]org  
    a2net[.]oss-cn-hongkong[.]aliyuncs[.]com  
    edfbdc7fc81abad462efa6688c19482a[.]oss-accelerate[.]aliyuncs[.]com  
    api18[.]srdna[.]com  
    download88[.]srdna[.]com  
    download[.]telegran[.]fit  
    download95[.]srdna[.]com  
    download[.]telebram[.]com  
    dhdkenxke[.]xyz  
    www[.]tleamaa[.]net  
    china-telegrme[.]com  
    telegorm[.]com  
    zh[.]slqhtz[.]cn  
    www[.]telegran[.]fit  
    telebram[.]com  
    telegrgm[.]xyz  
    f2-lang[.]oss-cn-hongkong[.]aliyuncs[.]com  
    dow-a15[.]oss-cn-hongkong[.]aliyuncs[.]com  
    dow-a11[.]oss-accelerate[.]aliyuncs[.]com  
    www[.]teledown[.]org  
    cdndown[.]shop  
    laohuzhi[.]oss-cn-hongkong[.]aliyuncs[.]com  
    download82[.]srdna[.]com  
    www[.]telamvz[.]com  
    www[.]tlergam[.]xyz  
    www[.]telegraysm[.]com  
    www[.]telamse[.]com  
    www[.]telamad[.]com  
    www[.]telamsf[.]com  
    www[.]telamfs[.]com  
    telegram-android[.]org  
    telegnm[.]com  
    telegraman[.]com  
    www[.]telegracm[.]org  
    www[.]telegramxx[.]com  
    www[.]telegvn[.]com  
    www[.]telegraxm[.]net  
    www[.]telegima[.]xyz  
    www[.]telagtiem[.]com  
    www[.]telegima[.]com  
    www[.]telegima[.]top  
    www[.]cn-teledown[.]com  
    download[.]telegramm[.]work  
    www[.]telegramm[.]work  
    telegaam[.]org  
    zh-telegram[.]app  
    www[.]tgdown[.]org  
    telegram[.]me  
    www[.]telegram[.]com[.]pe  
    telagran[.]com  
    telegramgb[.]com  
    telearnm[.]com  
    telegram[.]surf  
    telegramrm[.]com  
    www[.]telegramv[.]org  
    kitgafpslmj102047[.]telegrammessenger[.]xyz  
    telegramcn[.]org  
    telegriem[.]com  
    www[.]telegramsg[.]org  
    telegramlinux[.]com  
    telegvcn[.]com  
    www[.]telegramlk[.]org  
    5c4488d628a64861d71a396bbafbf4b2[.]oss-accelerate[.]aliyuncs[.]com  
    telgm-zw[.]com  
    telegranm[.]net  
    www[.]telegnam[.]com  
    tg404[.]com  
    telegramz[.]me  
    www[.]teleincn[.]com  
    54ggssdfr[.]oss-cn-hongkong[.]aliyuncs[.]com  
    52telegram[.]com  
    www[.]telegramac[.]com  
    telegrampc[.]org  
    www[.]telebigi[.]com  
    www[.]telegkn[.]com  
    www[.]telegramce[.]com  
    download[.]telegraema[.]com  
    telegram-cn[.]org  
    www[.]tevegram[.]com  
    cn-telagrem[.]com  
    telegrammj[.]com  
    www[.]telegrgm[.]com  
    www[.]telegcm[.]com  
    www[.]telegramns[.]com  
    www[.]telegramopen[.]co  
    telegmm[.]com  
    www[.]telegramyy[.]com  
    telegramcs[.]com  
    download[.]telegramm[.]cloud  
    telegrampl[.]com  
    telegrzm[.]com  
    www[.]teleylm[.]com  
    clomidus[.]com  
    www[.]webk-telegram[.]org  
    626f8c47c30ce[.]site123[.]me  
    www[.]telegraka[.]com  
    app[.]zhaixz[.]com  
    telegramgi[.]com  
    telagrem-zn[.]com  
    www[.]telegham[.]com  
    www[.]telegdn[.]com  
    www[.]telegrfm[.]com  
    www[.]telegnm[.]com  
    www[.]telegrames[.]org  
    u5x9ckzpj7910225[.]gettelegram[.]xyz  
    f953b4a29c6253a0b43ca25601710c32[.]oss-accelerate[.]aliyuncs[.]com  
    telgram[.]cn  
    tgdowm[.]com  
    app[.]telegranyy[.]com  
    8qw2hl54c9p105654[.]telegramwindows[.]xyz  
    telegram[.]gs  
    telegpn[.]com  
    downs[.]telcp213[.]com  
    www[.]teleggam[.]com  
    www[.]telegramdh[.]com  
    telegqn[.]com  
    www[.]telegram[.]farm  
    telegrcn[.]org  
    www[.]telelgracn[.]com  
    www[.]telegkam[.]com  
    www[.]skypocn[.]com  
    telegrjm[.]com  
    wwv[.]telegramdm[.]com  
    www[.]telegrann[.]org  
    www[.]telegram-c[.]com  
    www[.]telegramstr[.]com  
    telegbam[.]com  
    aptne[.]com  
    telegrlm[.]com  
    telegram-install[.]com  
    tlegrem[.]com  
    www[.]telegranak[.]com  
    www[.]telegxn[.]com  
    www[.]telegcsm[.]com  
    www[.]telegpam[.]com  
    telegtrm[.]com  
    telegrtm[.]com  
    www[.]telepang[.]com  
    www[.]telegracn[.]org  
    ttelgram[.]com  
    telegrn[.]com  
    ww[.]telegwm[.]com  
    telegram-v7[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegwn[.]com  
    www[.]telegbm[.]com  
    wwv[.]telegrfm[.]com  
    www[.]telegrarnm[.]org  
    www[.]telegramae[.]com  
    chinesetelecn[.]vip  
    teleglon[.]com  
    www[.]telegrambu[.]com  
    tgsgp[.]oss-accelerate[.]aliyuncs[.]com  
    www[.]telegrantn[.]com  
    www[.]chinesetelecn[.]vip  
    www[.]potacn[.]com  
    www[.]telegramm[.]cloud  
    teletgarm[.]com  
    telegram-chian[.]org[.]cn  
    telegrammacos[.]org  
    zh-cn-channel[.]telagarm[.]com  
    telegarnm[.]com  
    teiegrcm[.]com  
    telegramrn[.]com  
    teleglam-cn[.]com  
    www[.]telegrampx[.]com  
    www[.]telecgram[.]org  
    www[.]telegramcl[.]com  
    www[.]teleylc[.]com  
    3s3wy68jxy311850[.]telegramlinux[.]xyz  
    telegpam[.]com  
    gettelegram[.]org  
    telegramde[.]com  
    telegrab[.]org  
    www[.]telegram-china[.]org  
    www[.]telegrhm[.]com  
    www[.]telegramcq[.]org  
    www[.]telegramdi[.]com  
    www[.]telepve[.]com  
    cn-teleglam[.]com  
    telegran[.]one  
    www[.]telegtcn[.]com  
    www[.]skypebee[.]com  
    telegvm[.]com  
    www[.]telegramau[.]com  
    telegram-w1[.]oss-cn-hongkong[.]aliyuncs[.]com  
    www[.]telegramga[.]com  
    www[.]telegrambx[.]com  
    telegram2[.]com  
    telegramapp[.]cn  
    telecnsr[.]com  
    telegramsku[.]com  
    www[.]telcp[.]com  
    chip-usa[.]com  
    www[.]telegrancf[.]com  
    www[.]teleghn[.]com  
    www[.]telgcn[.]com  
    download81[.]srdna[.]com  
    www[.]telezj[.]com  
    telegrammessenger[.]cn  
    telergems[.]com  
    www[.]harleyfanzone[.]com  
    jjkmxv[.]com  
    www[.]telegramdj[.]com  
    383f66e5f63bef91845f5a4e22ae5be5[.]oss-accelerate[.]aliyuncs[.]com  
    telegramapp[.]pro  
    telegramam[.]org  
    www[.]telegrammc[.]com  
    www[.]telegramim[.]org  
    telegfn[.]com  
    telegzm[.]com  
    telegrnam[.]com  
    telegram-24[.]com  
    www[.]telegdam[.]com  
    telegramwindows[.]org  
    www[.]telegramos[.]org  
    www[.]telegramapp[.]vip  
    telegrannn[.]com  
    www[.]telegramracn[.]org  
    tgcn[.]top  
    www[.]telegranhk[.]com  
    www[.]telegraema[.]com  
    telegram[.]965rock[.]com  
    telegram[.]wpcoder[.]cn  
    www[.]telegmramcn[.]com  
    www[.]cntegrom[.]com  
    27[.]124[.]46[.]23  
    www[.]telegramcj[.]com  
    www[.]telegvam[.]org  
    tgramarn[.]com  
    telegrammacos[.]com  
    www[.]telegramv[.]com  
    telegrambt[.]com  
    telegramwindows[.]com  
    telegrambq[.]com  
    www[.]telegfam[.]com  
    telegranzh[.]com  
    www[.]telegdm[.]com  
    www[.]telegramaz[.]com  
    telegram-china[.]org  
    telegramgg[.]com  
    www[.]telegvam[.]com  
    88smgmt1zh2105738[.]telegramwindows[.]xyz  
    www[.]telegram-zw[.]com  
    www[.]telegramgh[.]com  
    www[.]telegramgi[.]com  
    www[.]telegrampv[.]com  
    www[.]telegmn[.]com  
    telegramapp[.]io  
    www[.]fj-telegram[.]com  
    x1eq1kyc5k10454[.]telegramwindows[.]xyz  
    telegramav[.]com  
    www[.]telegramnc[.]com  
    www[.]telegron[.]com  
    www[.]telegram-fj[.]com  
    download[.]telegramm[.]wang  
    telegrems[.]wixsite[.]com  
    telemetr[.]io  
    app[.]cntanghuang[.]com  
    telegramcp[.]com  
    x-telegram[.]app  
    telegramab[.]com  
    telegramct[.]com  
    sipins[.]com  
    tleamsc[.]com  
    tgcn[.]cash  
    app[.]telegramam[.]org  
    telegram-fj[.]com  
    wwv[.]telegranyy[.]com  
    telegramorg[.]cn  
    www[.]telegramtk[.]com  
    telogarm[.]cc  
    qqjiik1slv8105558[.]telegrammacos[.]xyz  
    teleggam[.]com  
    4tewded[.]oss-cn-hongkong[.]aliyuncs[.]com  
    telegkm[.]com  
    telegramapp[.]tv  
    www[.]telegramdg[.]com  
    www[.]telegramgb[.]com  
    www[.]telesun[.]org  
    www[.]telegramml[.]com  
    

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

