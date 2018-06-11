---
title: iOS逆向工程总结
date: 2018-05-02 10:48:52
tags: 走走停停,已有三月逆向了。或许路没有尽头,但并不是放弃理由~难道最美的风景不是在路上吗?文章主要是提炼小黄书的内容,详细请自行看书~
---

---

[TOC]

### iOS 逆向概念

####  1.逆向的作用

* 通过分析目标程序，拿到关键信息 ,可以归类于安全相关的逆向工程
  1. 评定安全等级
  2. 修改软件使用限制,逻辑,功能等
  3. 开发相关逆向工程

#### 2.逆向工具

|        检测工具        | 反汇编工具 | 调试工具 |  开发工具  |
| :--------------------: | :--------: | :------: | :--------: |
|     Reveal(监测UI)     |    IDA     |   lldb   | iOSOpenDev |
|   snoop-it(网络活动)   |   Hopper   |    \     |   Theos    |
| introspy(分析应用程序) |     \      |    \     |     \      |

#### 3.iOS 目录结构

+ ·/：根目录，以斜杠表示，其他所有文件和目录在根目录下展开 
+ ·/bin：“binary”的简写，存放提供用户级基础功能的二进制文件，如ls、ps等 
+ ·/boot：存放能使系统成功启动的所有文件。iOS中此目录为空”
+ ·/dev：“device”的简写，存放BSD设备文件” “每个文件代表系统的一个块设备或字符设备
+ ·/sbin：“system binaries”的简写，存放提供系统级基础功能的二进制文件，如netstat、reboot等 
+ ·/etc：“Et Cetera”的简写，存放系统脚本及配置文件，如passwd、hosts等。在iOS中，/etc是一个符号链接，实际指向/private/etc。 
+ ·/lib：存放系统库文件、内核模块及设备驱动等。iOS中此目录为空 
+ ·/mnt：“mount”的简写，存放临时的文件系统挂载点。iOS中此目录为空。 
+ ·/private：存放两个目录，分别是/private/etc和/private/var 
+ ·/tmp：临时目录。在iOS中，/tmp是一个符号链接，实际指向/private/var/tmp。 
+ ·/usr：包含了大多数用户工具和程序。/usr/bin包含那些/bin和/sbin中未出现的基础功能，如nm、killall等；/usr/include包含所有的标准C头文件；/usr/lib存放库文件。 
+ ·/var：“variable”的简写，存放一些经常更改的文件，比如日志、用户数据、临时文件等”“其中/var/mobile和/var/root分别存放了mobile用户和root用户的文件，是重点关注的目录。 
+ /Applications：存放所有的系统App和来自于Cydia的App，不包括StoreApp 
+ ·/Developer：如果一台设备连接Xcode后被指定为调试用机（如图2-4所示），Xcode就会在iOS中生成这个目录 
+ ·/Library：存放一些提供系统支持的数据 
+ ·/System/Library：iOS文件系统中最重要的目录之一，存放大量系统组件 
+ ·/System/Library/Frameworks和/System/Library/PrivateFrameworks：存放iOS中的各种framework，其中出现在SDK文档里的只是冰山一角，还有数不清的未公开功能等待我们去挖掘 
+ ·/System/Library/CoreServices里的SpringBoard.app：iOS桌面管理器（类似于Windows里的explorer），是用户与系统交流的最重要中介 
+ /User：用户目录，实际指向/var/mobile  "·/var/mobile/Media/DCIM下存放照片 " “·/var/mobile/Media/Recordings下存放录音文件；” “·/var/mobile/Library/SMS下存放短信数据库；” “·/var/mobile/Library/Mail下存放邮件数据。”  “非常重要的子目录是/var/mobile/Containers，存放StoreApp。值得注意的是，App的可执行文件在bundle与App中的数据目录被分别存放在/var/mobile/Containers/Bundle和/var/mobile/Containers/Data这两个不同目录下” 

#### 4.iOS二进制文件类型

+ Application

  + 1.bundle

    + App和framework
    + PreferenceBundle(依附于setting)

  + App目录结构

    + Info.plist(记录App的基本信息 )

      关注Bundle Identifier ,可执行文件名等有用信息

    + 可执行文件

       BundleExecutable

    + lproj目录(本地化字符串)

  + 系统App VS.StoreApp

    + 目录结构

      系统App(Cydia APP):/Applications   权限:root

      StoreApp:/var/mobile/Containers/    权限:mobile

    + 安装包格式与权限

      + 系统(cydia)App: .deb
      + storeApp:  .ipa

    + 沙盒

+ Dynamic Library

  在iOS中，lib分为static和dynamic两种:

  ​	1.其中static lib在编译阶段成为App可执行文件的一部分，会增加可执行文件的大小

  ​	2.dylib不会改变可执行文件的大小，只有当App需要用到这个dylib时，iOS才会把它加载进内存，成为App进程的一部分

  lib 本身并不是可执行文件，不能独立运行，只能为别的进程服务，而且它们寄生在别的进程里，成为了这个进程的一部分。

+ Deamon(后台) 

###OSX工具

#### 1.class-dump

+ 作用:用来dump目标文件的class信息的工具
+ 原理:利用Objective-C语言的runtime的特性，将存储在mach-O文件中的@interface和@protocol信息提取出来，并生成对应的.h文件
+ 安装使用:https://www.jianshu.com/p/1e3fe0a8c048
+ 主要命令行:class-dump -H [.app文件的路径] -o [输出文件夹路径]
+ 局限:1.没有加密的App。APPStore上的App就需要配合Dumpdecrypted破壳了 2.只能在作用于纯OC项目,对于Swift无能为力

#### 2.Theos

+ 作用:越狱工具开发包(开发工具)

+ 安装使用:https://www.cnblogs.com/ludashi/p/5714095.html

+ 主要命令行:

  1. export THEOS=theos文件所在路径(一般为/opt/theos)
  2. $THEOS/bin/nic.pl        新建工程
  3. make   编译
  4. make package   打包
  5. make install 安装

+ 语言:[Logos](http://iphonedevwiki.net/index.php/Logos)

  

#### 3.Reveal

+ 作用:监测UI布局
+ 安装教程: https://blog.csdn.net/hello_hwc/article/details/69365095
+ 主要命令行:/
+ 注意:OSX与iOS要在同一网段下

#### 4.IDA

+ 作用:反汇编工具,看源代码的
+ 下载安装:https://www.hex-rays.com/products/ida/support/download_demo.shtml
+ 教程:http://www.freebuf.com/column/157939.html
+ 主要操作:空格键 、F5等
+ 难点:汇编



#### 5.非越狱开发(MonkeyDey)

+   作用:非越狱开发
+   教程:http://bbs.iosre.com/t/topic/8696

### iOS工具集 

#### 1.Cydia Substrate  

+ 作用:绝大部分tweak正常工作的基础，它由MobileHooker、MobileLoade和Safe mode 组成。 cydia商店可直接下载

####  2.cycript 

+ 作用:脚本语言,测试(调试)运行的App
+ 安装:cydia直接下载安装
+ 教程:http://www.cycript.org/manual/、https://www.jianshu.com/p/7c41b03c9eb3
+ 语言:Objective-C与javascript

#### 3.lldb 

+ 作用:Xcode自带的调试工具
+ 安装教程:http://www.qingpingshan.com/rjbc/ios/140197.html
+ 使用教程:http://www.dllhook.com/post/51.html
+ 主要命令行:
  + 直接启动进程：debugserver -x backboard *:1234 /path/to/app/executable
  + 附加进程：./debugserver *:1234 -a "YourAPPName"
  + po(打印对象信息) p(打印值) c(继续执行) s、si(单步执行,子函数进入) 、n、ni(单步执行,子函数跳过)
  + breakpoint 断点设置,详细看书,或者是教程
  + image list -o -f 模块基地址



#### 4.dumpdecrypted  

+ 作用:AppStrore榔头,在AppStore下载的app不能直接dump,要配合dumpdecrypted才可以导出头文件信息
+ 使用安装:https://www.jianshu.com/p/039dfd040253
+ 主要命令行:DYLD_INSERT_LIBRARIES=/path/to/dumpdecrypted.dylib /path/to/executable

#### 5.OpenSSH 

+ 作用:iOS设备安装ssh服务,cydia可直接下载
+ 命令行:ssh  scp等

#### 6.usbmuxd 

+ 作用:usb的ssh服务

#### 7.iflie、Filza

+ 作用:iOS文件管理工具

#### 8.MTerminal  

+ 作用:ios 终端 输入

#### 9.syslogd 

+ 作用:系统日记







### 优秀资源文章

+ https://niyaoyao.github.io/2017/05/09/Learning-Reverse-From-Today-D4/