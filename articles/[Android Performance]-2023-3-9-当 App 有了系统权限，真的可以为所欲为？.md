#  当 App 有了系统权限，真的可以为所欲为？

原创 Gracker [ Android Performance ](javascript:void\(0\);)

**Android Performance** ![]()

微信号 AndroidPerformance

功能介绍 分享 Android 系统和 App 的性能优化知识、开发技巧、Android 开发工具、性能优化工具、Android && Linux
系统开发等

____

___发表于_

收录于合集 #系统开发 4个

看到群里发了两篇文章，出于好奇，想看看这些个 App 在利用系统漏洞获取系统权限之后，都干了什么事，于是就有了这篇文章。由于准备仓促，有些 Code
没有仔细看，感兴趣的同学可以自己去研究研究，多多讨论

  1. [深蓝洞察：2022 年度最 “不可赦” 漏洞](https://mp.weixin.qq.com/s?__biz=MzkyMjM5MTk3NQ==&mid=2247484287&idx=1&sn=73ebf1ae3aee7bbe1a1e479246fbd7f7&scene=21#wechat_redirect)
  2. XXX apk 内嵌提权代码，及动态下发 dex 分析：https://github.com/davinci1010/pinduoduo_backdoor

 **关于这个 App 是如何获取这个系统权限的** ，[「 深蓝洞察 」2022 年度最 “不可赦”
漏洞](https://mp.weixin.qq.com/s?__biz=MzkyMjM5MTk3NQ==&mid=2247484287&idx=1&sn=73ebf1ae3aee7bbe1a1e479246fbd7f7&scene=21#wechat_redirect)
，这篇文章讲的很清楚，就不再赘述了：

> Android Framework 中一个核心的对象传递机制是 Parcel， 希望被通过 Parcel 传递的对象需要定义
> readFromParcel 和 writeToParcel 接口函数，并实现 Parcelable 接口。
> 理论上来讲，匹配序列化和反序列化函数应当是自反等效的，但系统 ROM 的开发者在编程过程中可能会出现不匹配的情况，例如写入的时候使用了
> writeLong， 读取的时候却使用了 readInt。
> 这类问题在运行过程中一般不会引起注意，也不会导致崩溃或错误，但在攻击者精心布局下，却可最终利用 Settings 和 system_server
> 进程，将这个微小的错误转化为 StartAnyWhere 提权。 Android 近年来累计已修复上百个这类漏洞，并在 Android 13 中对
> Parcel 机制做了改革，彻底杜绝了大部分此类攻击面。
>
> 但对于鸿蒙和绝大部分未升级到 Android 13 的设备和用户来说，他们仍处于危险之中。

我们这篇主要来看看XXX apk 内嵌提权代码，及动态下发 dex 分析 这个库里面提供的 Dex ，看看 App 到底想知道用户的什么信息？

总的来说，App 获取系统权限之后，主要做了下面几件事（正常 App 无法或者很难做到的事情），各种不把用户当人了

  1.  **自启动、关联启动相关的修改，偷偷打开或者默认打开**
  2.  **开启通知权限**
  3.  **监听通知内容**
  4.  **获取用户的使用手机的信息，包括安装的 App、使用时长、用户 ID、用户名等**
  5.  **修改系统设置**
  6.  **整一些系统权限的工具方便自己使用**

 **另外也可以看到，App 对于各个手机厂商的研究还是比较深入的，针对华为、Oppo、Vivo、Xiaomi
等厂商都有专门的处理，这个也是值得手机厂商去反向研究的**

#  0\. Dex 文件信息

XXX apk 内嵌提权代码，及动态下发 dex 分析 这个库提供的的 Dex 文件总共有 37 个，不多，也不大，慢慢看

![](https://gitee.com/fuli009/images/raw/master/public/20230309221844.png)

由于是 dex 文件，所以直接使用 https://github.com/tp7309/TTDeDroid 这个库的工具打开看即可，比如

> showjar 95cd95ab4d694ad8bdf49f07e3599fb3.dex

默认是用 jadx 打开，我们重点看内容

![](https://gitee.com/fuli009/images/raw/master/public/20230309221906.png)

打开后可以看到具体的功能逻辑，可以看到一个 dex 一般只干一件事，那我们重点看这件事的核心实现部分即可

# 1\. 通知监听和通知权限相关

# 1.1 获取 Xiaomi 手机通知内容

  1. 文件 ： 95cd95ab4d694ad8bdf49f07e3599fb3.dex
  2. 功能 ：获取用户的 Active 通知
  3. 类名 ：com.google.android.sd.biz_dynamic_dex.xm_ntf_info.XMGetNtfInfoExecutor

## 1\. 反射拿到 ServiceManager

一般我们会通过 ServiceManager 的 getService 方法获取系统的
Service，然后进行远程调用![](https://gitee.com/fuli009/images/raw/master/public/20230309221907.png)

## 2\. 通过 NotificationManagerService 获取通知的详细内容

通过 getService 传入 NotificationManagerService 获取 NotificationManager 之后，就可以调用
**getActiveNotifications** 这个方法了，然后具体拿到 Notification 的下面几个字段

  1. 通知的 Title
  2. 发生通知的 App 的包名
  3. 通知发送时间
  4. key
  5. channelID ：the id of the channel this notification posts to.

可能有人不知道这玩意是啥，下面这个图里面就是一个典型的通知

![](https://gitee.com/fuli009/images/raw/master/public/20230309221909.png)

其代码如下![](https://gitee.com/fuli009/images/raw/master/public/20230309221910.png)

可以看到 getActiveNotifications 这个方法，是 **System-only** 的，普通的 App 是不能随便读取
Notification 的，但是这个 App 由于有权限，就可以获取

![](https://gitee.com/fuli009/images/raw/master/public/20230309221912.png)

当然微信的防撤回插件使用的一般是另外一种方法，比如辅助服务，这玩意是合规的，但是还是推荐大家能不用就不用，它能帮你防撤回，他就能获取通知的内容，包括你知道的和不知道的

# 1.2. 打开 Xiaomi 手机上的通知权限（Push）

  1. 文件 ：0fc0e98ac2e54bc29401efaddfc8ad7f.dex
  2. 功能 ：可能有的时候小米用户会把 App 的通知给关掉，App 想知道这个用户是不是把通知关了，如果关了就偷偷打开
  3. 类名 ：com.google.android.sd.biz_dynamic_dex.xm_permission.XMPermissionExecutor

这么看来这个应该还是蛮实用的，你个调皮的用户，我发通知都是为了你好，你怎么忍心把我关掉呢？让我帮你偷偷打开吧![](https://gitee.com/fuli009/images/raw/master/public/20230309221917.png)

App 调用 NotificationManagerService 的 setNotificationsEnabledForPackage
来设置通知，可以强制打开通知
frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java![](https://gitee.com/fuli009/images/raw/master/public/20230309221926.png)

然后查看 NotificationManagerService 的 setNotificationsEnabledForPackage
这个方法，就是查看用户是不是打开成功了
frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java![](https://gitee.com/fuli009/images/raw/master/public/20230309221927.png)

还有针对 leb 的单独处理~ 细 ！

# 1.3. 打开 Vivo 机器上的通知权限（Push）

  1. 文件 ：2eb20dc580aaa5186ee4a4ceb2374669.dex
  2. 功能 ：Vivo 用户会把 App 的通知给关掉，这样在 Vivo 手机上 App 就收不到通知了，那不行，得偷偷打开
  3. 类名 ：com.google.android.sd.biz_dynamic_dex.vivo_open_push.VivoOpenPushExecutor

核心和上面那个是一样的，只不过这个是专门针对 vivo 手机的

![](https://gitee.com/fuli009/images/raw/master/public/20230309221928.png)

# 1.4 打开 Oppo 手机的通知权限

  1. 文件 ：67c9e686004f45158e94002e8e781192.dex
  2. 类名 ：com.google.android.sd.biz_dynamic_dex.oppo_notification_ut.OppoNotificationUTExecutor

没有反编译出来，看大概的逻辑应该是打开 App 在 oppo 手机上的通知权限

![](https://gitee.com/fuli009/images/raw/master/public/20230309221929.png)

# 1.5 Notification 监听

  1. 文件 ：ab8ed4c3482c42a1b8baef558ee79deb.dex
  2. 类名 ：com.google.android.sd.biz_dynamic_dex.ud_notification_listener.UdNotificationListenerExecutor

这个就有点厉害了，在监听 App 的 Notification 的发送，然后进行统计

![](https://gitee.com/fuli009/images/raw/master/public/20230309221931.png)

监听的核心代码![](https://gitee.com/fuli009/images/raw/master/public/20230309221932.png)

这个咱也不是很懂，是时候跟做了多年 SystemUI 和 Launcher 的老婆求助了....@史工

# 1.6 App Notification 监听

  1. 文件 ：4f260398-e9d1-4390-bbb9-eeb49c07bf3c.dex
  2. 类名 ：com.google.android.sd.biz_dynamic_dex.notification_listener.NotificationListenerExecutor

上面那个是 UdNotificationListenerExecutor ， 这个是 NotificationListenerExecutor，UD 是啥？

![](https://gitee.com/fuli009/images/raw/master/public/20230309221939.png)

这个反射调用的 setNotificationListenerAccessGranted 是个
SystemAPI，获得通知的使用权，果然有权限就可以为所欲为

![](https://gitee.com/fuli009/images/raw/master/public/20230309221940.png)![](https://gitee.com/fuli009/images/raw/master/public/20230309221941.png)

# 1.7 打开华为手机的通知监听权限

  1. 文件 ：a3937709-b9cc-48fd-8918-163c9cb7c2df.dex
  2. 类名 ：com.google.android.sd.biz_dynamic_dex.hw_notification_listener.HWNotificationListenerExecutor

华为也无法幸免，哈哈哈![](https://gitee.com/fuli009/images/raw/master/public/20230309221942.png)

# 1.8 打开华为手机通知权限

  1. 文件 ：257682c986ab449ab9e7c8ae7682fa61.dex
  2. 类名 ：com.google.android.sd.biz_dynamic_dex.hw_permission.HwPermissionExecutor

![](https://gitee.com/fuli009/images/raw/master/public/20230309221944.png)

# 2\. Backup 状态

# 2.1. 鸿蒙 OS 上 App Backup 状态相关，保活用？

  1. 文件 ：6932a923-9f13-4624-bfea-1249ddfd5505.dex
  2. 功能 ：Backup 相关

这个看了半天，应该是专门针对华为手机的，收到 IBackupSessionCallback 回调后，执行
PackageManagerEx.startBackupSession 方法

![](https://gitee.com/fuli009/images/raw/master/public/20230309221945.png)![](https://gitee.com/fuli009/images/raw/master/public/20230309221946.png)

查了下这个方法的作用，启动备份或恢复会话

![](https://gitee.com/fuli009/images/raw/master/public/20230309221947.png)

# 2.2. Vivo 手机 Backup 状态相关

  1. 文件 ：8c34f5dc-f04c-40ba-98d4-7aa7c364b65c.dex
  2. 功能 ：Backup 相关

![](https://gitee.com/fuli009/images/raw/master/public/20230309221949.png)

# 3\. 文件相关

# 3.1 获取华为手机 SLog 和 SharedPreferences 内容

  1. 文件 ： da03be2689cc463f901806b5b417c9f5.dex
  2. 类名 ：com.google.android.sd.biz_dynamic_dex.hw_get_input.HwGetInputExecutor

拿这个干嘛呢？拿去做数据分析？

![](https://gitee.com/fuli009/images/raw/master/public/20230309221950.png)

获取 SharedPreferences

![](https://gitee.com/fuli009/images/raw/master/public/20230309221951.png)

获取 slog

![](https://gitee.com/fuli009/images/raw/master/public/20230309221952.png)

# 4\. 用户数据

# 4.1 获取用户使用手机的数据

  1. 文件 ： 35604479f8854b5d90bc800e912034fc.dex
  2. 功能 ：看名字就知道是获取用户的使用手机的数据
  3. 类名 ：com.google.android.sd.biz_dynamic_dex.usage_event_all.UsageEventAllExecutor

看核心逻辑是同 usagestates 服务，来获取用户使用手机的数据，难怪我手机安装了什么 App、用了多久这些，其他 App
了如指掌![](https://gitee.com/fuli009/images/raw/master/public/20230309221953.png)

那么他可以拿到哪些数据呢？应有尽有~，包括但不限于 App 启动、退出、挂起、Service 变化、Configuration
变化、亮灭屏、开关机等，感兴趣的可以看一下：

    
    
    frameworks/base/core/java/android/app/usage/UsageEvents.java  
        private static String eventToString(int eventType) {  
            switch (eventType) {  
                case Event.NONE:  
                    return "NONE";  
                case Event.ACTIVITY_PAUSED:  
                    return "ACTIVITY_PAUSED";  
                case Event.ACTIVITY_RESUMED:  
                    return "ACTIVITY_RESUMED";  
                case Event.FOREGROUND_SERVICE_START:  
                    return "FOREGROUND_SERVICE_START";  
                case Event.FOREGROUND_SERVICE_STOP:  
                    return "FOREGROUND_SERVICE_STOP";  
                case Event.ACTIVITY_STOPPED:  
                    return "ACTIVITY_STOPPED";  
                case Event.END_OF_DAY:  
                    return "END_OF_DAY";  
                case Event.ROLLOVER_FOREGROUND_SERVICE:  
                    return "ROLLOVER_FOREGROUND_SERVICE";  
                case Event.CONTINUE_PREVIOUS_DAY:  
                    return "CONTINUE_PREVIOUS_DAY";  
                case Event.CONTINUING_FOREGROUND_SERVICE:  
                    return "CONTINUING_FOREGROUND_SERVICE";  
                case Event.CONFIGURATION_CHANGE:  
                    return "CONFIGURATION_CHANGE";  
                case Event.SYSTEM_INTERACTION:  
                    return "SYSTEM_INTERACTION";  
                case Event.USER_INTERACTION:  
                    return "USER_INTERACTION";  
                case Event.SHORTCUT_INVOCATION:  
                    return "SHORTCUT_INVOCATION";  
                case Event.CHOOSER_ACTION:  
                    return "CHOOSER_ACTION";  
                case Event.NOTIFICATION_SEEN:  
                    return "NOTIFICATION_SEEN";  
                case Event.STANDBY_BUCKET_CHANGED:  
                    return "STANDBY_BUCKET_CHANGED";  
                case Event.NOTIFICATION_INTERRUPTION:  
                    return "NOTIFICATION_INTERRUPTION";  
                case Event.SLICE_PINNED:  
                    return "SLICE_PINNED";  
                case Event.SLICE_PINNED_PRIV:  
                    return "SLICE_PINNED_PRIV";  
                case Event.SCREEN_INTERACTIVE:  
                    return "SCREEN_INTERACTIVE";  
                case Event.SCREEN_NON_INTERACTIVE:  
                    return "SCREEN_NON_INTERACTIVE";  
                case Event.KEYGUARD_SHOWN:  
                    return "KEYGUARD_SHOWN";  
                case Event.KEYGUARD_HIDDEN:  
                    return "KEYGUARD_HIDDEN";  
                case Event.DEVICE_SHUTDOWN:  
                    return "DEVICE_SHUTDOWN";  
                case Event.DEVICE_STARTUP:  
                    return "DEVICE_STARTUP";  
                case Event.USER_UNLOCKED:  
                    return "USER_UNLOCKED";  
                case Event.USER_STOPPED:  
                    return "USER_STOPPED";  
                case Event.LOCUS_ID_SET:  
                    return "LOCUS_ID_SET";  
                case Event.APP_COMPONENT_USED:  
                    return "APP_COMPONENT_USED";  
                default:  
                    return "UNKNOWN_TYPE_" + eventType;  
            }  
        }  
    

# 4.2 获取用户使用数据

  1. 文件：b50477f70bd14479a50e6fa34e18b2a0.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.usage_event.UsageEventExecutor

上面那个是 UsageEventAllExecutor，这个是 UsageEventExecutor，主要拿用户使用 App
相关的数据，比如什么时候打开某个 App、什么时候关闭某个 App，6 得很，真毒瘤

![](https://gitee.com/fuli009/images/raw/master/public/20230309221954.png)

# 4.3 获取用户使用数据

  1. 文件：1a68d982e02fc22b464693a06f528fac.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.app_usage_observer.AppUsageObserver

看样子是注册了 App Usage 的权限，具体 Code 没有出来，不好分析

![](https://gitee.com/fuli009/images/raw/master/public/20230309221956.png)

# 5\. Widget 和 icon 相关

# 5.1. Vivo 手机添加 Widget

  1. 文件：f9b6b139-4516-4ac2-896d-8bc3eb1f2d03.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.vivo_widget.VivoAddWidgetExecutor

这个比较好理解，在 Vivo 手机上加个 Widget

![](https://gitee.com/fuli009/images/raw/master/public/20230309221957.png)

# 5.2 获取 icon 相关的信息

  1. 文件：da60112a4b2848adba2ac11f412cccc7.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.get_icon_info.GetIconInfoExecutor

这个好理解，获取 icon 相关的信息，比如在 Launcher 的哪一行，哪一列，是否在文件夹里面。问题是获取这玩意干嘛？？？迷

![](https://gitee.com/fuli009/images/raw/master/public/20230309221958.png)

# 5.3 Oppo 手机添加 Widget

  1. 文件：75dcc8ea-d0f9-4222-b8dd-2a83444f9cd6.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.oppoaddwidget.OppoAddWidgetExecutor

![](https://gitee.com/fuli009/images/raw/master/public/20230309222000.png)

# 5.4 Xiaomi 手机更新图标？

  1. 文件：5d372522-b6a4-4c1b-a0b4-8114d342e6c0.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.xm_akasha.XmAkashaExecutor

小米手机上的桌面 icon 、shorcut 相关的操作，小米的同学来认领

![](https://gitee.com/fuli009/images/raw/master/public/20230309222001.png)

# 6\. 自启动、关联启动、保活相关

# 6.1 打开 Oppo 手机自启动

  1. 文件：e723d560-c2ee-461e-b2a1-96f85b614f2b.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.oppo_boot_perm.OppoBootPermExecutor

看下面这一堆就知道是和自启动相关的，看来自启动权限是每个 App 都蛋疼的东西啊

![](https://gitee.com/fuli009/images/raw/master/public/20230309222002.png)![](https://gitee.com/fuli009/images/raw/master/public/20230309222004.png)

# 6.2 打开 Vivo 关联启动权限

  1. 文件：8b56d820-cac2-4ca0-8a3a-1083c5cca7ae.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.vivo_association_start.VivoAssociationStartExecutor

看名字就是和关联启动相关的权限，vivo
的同学来领了![](https://gitee.com/fuli009/images/raw/master/public/20230309222005.png)

直接写了个节点进去

![](https://gitee.com/fuli009/images/raw/master/public/20230309222007.png)

# 6.3 关闭华为耗电精灵

  1. 文件：7c6e6702-e461-4315-8631-eee246aeba95.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.hw_hide_power_window.HidePowerWindowExecutor

看名字和实现，应该是和华为的耗电精灵有关系，华为的同学可以来看看

![](https://gitee.com/fuli009/images/raw/master/public/20230309222008.png)![](https://gitee.com/fuli009/images/raw/master/public/20230309222010.png)

# 6.4 Vivo 机型保活相关

  1. 文件：7877ec6850344e7aad5fdd57f6abf238.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.vivo_get_loc.VivoGetLocExecutor

猜测和保活相关，Vivo 的同学可以来认领一下

![](https://gitee.com/fuli009/images/raw/master/public/20230309222011.png)![](https://gitee.com/fuli009/images/raw/master/public/20230309222012.png)

# 7\. 安装卸载相关

# 7.1 Vivo 手机回滚卸载

  1. 文件：d643e0f9a68342bc8403a69e7ee877a7.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.vivo_rollback_uninstall.VivoRollbackUninstallExecutor

这个看上去像是用户卸载 App
之后，回滚到预置的版本,好吧，这个是常规操作![](https://gitee.com/fuli009/images/raw/master/public/20230309222013.png)

# 7.2 Vivo 手机 App 卸载

  1. 文件：be7a2b643d7e8543f49994ffeb0ee0b6.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.vivo_official_uninstall.OfficialUntiUninstallV3

看名字和实现，也是和卸载回滚相关的![](https://gitee.com/fuli009/images/raw/master/public/20230309222015.png)

# 7.3 Vivo 手机 App 卸载相关

  1. 文件：183bb87aa7d744a195741ce524577dd0.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.vivo_official_uninstall.VivoOfficialUninstallExecutor

同上![](https://gitee.com/fuli009/images/raw/master/public/20230309222016.png)

# 其他

# SyncExecutor

  1. 文件：f4247da0-6274-44eb-859a-b4c35ec0dd71.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.sync.SyncExecutor

没看懂是干嘛的，核心应该是 Utils.updateSid ，但是没看到实现的地方

![](https://gitee.com/fuli009/images/raw/master/public/20230309222017.png)

# UdParseNotifyMessageExecutor

  1. 文件：f35735a5cbf445c785237797138d246a.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.ud_parse_nmessage.UdParseNotifyMessageExecutor

看名字应该是解析从远端传来的 Notify
Message，具体功能未知![](https://gitee.com/fuli009/images/raw/master/public/20230309222018.png)

# 6.3 TDLogcatExecutor

  1. 文件
    1. 8aeb045fad9343acbbd1a26998b6485a.dex
    2. 2aa151e2cfa04acb8fb96e523807ca6b.dex
  2. 类名
    1. com.google.android.sd.biz_dynamic_dex.td.logcat.TDLogcatExecutor
    2. com.google.android.sd.biz_dynamic_dex.td.logcat.TDLogcatExecutor

没太看懂这个是干嘛的，像是保活又不像，后面有时间了再慢慢分析![](https://gitee.com/fuli009/images/raw/master/public/20230309222020.png)

# 6.4 QueryLBSInfoExecutor

  1. 文件：74168acd-14b4-4ff8-842e-f92b794d7abf.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.query_lbs_info.QueryLBSInfoExecutor

获取 LBS Info

![](https://gitee.com/fuli009/images/raw/master/public/20230309222021.png)

# 6.5 WriteSettingsExecutor

  1. 文件：6afc90e406bf46e4a29956aabcdfe004.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.write_settings.WriteSettingsExecutor

看名字应该是个工具类，写 Settings 字段的，至于些什么应该是动态下发的

![](https://gitee.com/fuli009/images/raw/master/public/20230309222022.png)

# 6.6 OppoSettingExecutor

  1. 文件：61517b68-7c09-4021-9aaa-cdebeb9549f2.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.opposettingproxy.OppoSettingExecutor

Setting 代理？？没看懂干嘛的，Oppo 的同学来认领，难道是另外一种形式的保活？

![](https://gitee.com/fuli009/images/raw/master/public/20230309222023.png)

# 6.7 CheckAsterExecutor

  1. 文件：561341f5f7976e13efce7491887f1306.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.check_aster.CheckAsterExecutor

Check aster ？不是很懂

![](https://gitee.com/fuli009/images/raw/master/public/20230309222024.png)

# 6.8 OppoCommunityIdExecutor

  1. 文件：538278f3-9f68-4fce-be10-12635b9640b2.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.oppo_community_id.OppoCommunityIdExecutor

获取 Oppo 用户的 ID？要这玩意干么？

![](https://gitee.com/fuli009/images/raw/master/public/20230309222025.png)

# 6.9 GetSettingsUsernameExecutor

  1. 文件：4569a29c-b5a8-4dcf-a3a6-0a2f0bfdd493.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.oppo_get_settings_username.GetSettingsUsernameExecutor

获取 Oppo 手机用户的 username，话说你要这个啥用咧？

![](https://gitee.com/fuli009/images/raw/master/public/20230309222027.png)

# 6.10 LogcatExecutor

  1. 文件：218a37ea-710d-49cb-b872-2a47a1115c69.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.logcat.LogcatExecutor

配置 Log
的参数![](https://gitee.com/fuli009/images/raw/master/public/20230309222029.png)

# 6.11 VivoBrowserSettingsExecutor

  1. 文件：136d4651-df47-41b4-bb80-2ec0ab1bc775.dex
  2. 类名：com.google.android.sd.biz_dynamic_dex.vivo_browser_settings.VivoBrowserSettingsExecutor

Vivo 浏览器相关的设置，不太懂要干嘛

![](https://gitee.com/fuli009/images/raw/master/public/20230309222030.png)

  

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

