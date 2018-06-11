---
title: wifi密码破解
date: 2016-12-29 13:54:23
tags:
---

那天回了老家,没有网络,真是回归自然~今天就来说说mac电脑如何如何破解wifi~让没有网络的你,可以偷别人家的wifi,想想就激动~

##### 首先来首歌
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="https://music.163.com/outchain/player?type=2&id=116352&auto=1&height=66"></iframe>


### 文章说了以下几个方面

- 安装aircrack
- 利用aircrack 进行wifi抓包
- 配合字典开始wifi破解


### 安装aircrack
##### 1.首先安装homebrew

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```

##### 2.用homebrew安装aircrack

```
brew install aircrack-ng
```


### 利用aircrack 进行wifi抓包

##### 1.airport 监测到的附近 wifi 信息

```
airport -s
```
然后就能看到一下的信息

![wifi](1.png)

SSID 是 wifi名称，RSSI 是信号强度，CHANNEL 是信道。

##### 2.接着挑一个信号强的进行抓包

```
sudo airport en1 sniff  1
```

其中en1是电脑的网卡,大部分就是en1 或en0,等待几分钟~

##### 3.接着前往`/tmp`文件夹查看生成后缀为.cap的包,然后再桌面新建一个文件夹(假设我新建的文件夹名字为wifi),把抓到的后缀.cap的问价放进文件夹里面,重新命名为01.cap.


### 配合字典开始wifi破解

##### 1.谷歌(百度)找到密码字典
##### 2.把字典也放进上述所创建的wifi文件夹中,重命名为01.text
##### 3.终端cd 到这个文件夹里,然后查看wifi抓包情况

```
cd ~/Desktop/wifi
aircrack-ng -w 01.txt 01.cap
```

![4](4.png)

其中,Encryption中（0 handshake）是抓包失败，（1 handshake）则是抓包成功。看到第几行抓包成功，则在「Index number of target network ?」后敲回车：第几行 例如(11) <br/>
如果cap文件内全是（0 handshake），就按 command + c 组合键退出。重新回到「sudo airport en1 sniff 1」这步进行监听抓包。抓包成功率受到 wifi 信号强弱、电脑与路由器距离远近、路由器是否正处在收发数据状态的影响。总之多试几次、监听时间适当延长些，可以大大提高成功率。<br />
如果可以的话,会进入

![2](2.png)

接下来等待破解结果就行了，中断破解过程可以直接按 command + c 组合键退出。破解过程所需时间长短受电脑硬件配置、字典体积大小的影响。如果01.txt字典破解失败，则可以换其它字典进行破解，直到破解成功。
使用一个好的字典是很重要的，一个9位的纯数字字典大概1G多，结果经过几个小时的破解，如果密码是987654321就很令人郁闷了，所以最好准备几个常用的wifi密码字典，可以大大提高成功率和节省时间。常用字典可以直接百度Google搜索下载。<br />
如果成功了就会出现

![3](3.jpg)

然而我却一直卡在抓包那里,一直没成功过,心塞~

主要参考是[这里](http://chaishiwei.com/blog/562.html)



