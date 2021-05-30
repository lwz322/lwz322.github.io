---
layout: article
title: 折腾两台Haswell时代的设备
author :
mathjax: true
mermaid: true
chart: true
mathjax_autoNumber: true
lightbox: true
tags: Linux
key: haswell
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#ffffff'
  background_image:
    src: assets/background/stui.jpg
    gradient: 'linear-gradient(0deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
---
Haswell架构是Intel在2013年推出的”第四代“CPU架构，Haswell刚推出那年配了自己第一台台式机，后来上大学又捡了一台Haswell的笔记本T440s，毕业后又凑了一台M73小主机...台式机至今还在家用，笔记本已到垂暮之年，修改BIOS之后装上AX200依旧是顺手的全能笔电，M73目前作为编译机和NAS使用

<!--more-->

## T440s刷BIOS
之所以要刷BIOS，是因为T440s大部分的BIOS有网卡白名单，基本完全限制住了自行升级网卡，自带的AC7260是Intel最早的2x2 AC网卡，信号和速度已经远远落后如今的AX网卡，T440s现在主要是处理器跟不上了，其他方面放到现在也是非常优秀的：
- 接口丰富（主要是有RJ45），有廉价的拓展坞
- 1.4KG的重量，同重量下优秀的键盘键程
- 支持2242和2.5inch的SSD
- 使用PD转方口可以使用PD充电头

另外还有几个比较有意思的地方：
- i7-4600U处理器可以在XTU拉4倍频，本来最高3.3GHz可以拉高到3.7GHz（实测还是要手动加些电压，加太多功耗高，容易死机，实际只见过3.6GHz）
- 可以换装72WH的外置电池
- 没有不耐用的类肤质涂层

在如今是市场里确实很难找到合适自己用的笔记本（定位高，接口丰富，不用拓展坞），所以还是想多延续下T440s的使用期，一直比较头痛的就是更换网卡的问题，之前并非没有查过刷BIOS，但是教程太复杂，时过境迁，2020年搜到有不少人都能自己刷了，下面进入正题

### 参考教程

- [t440p bypass白名单(适用于任何BIOS版本)](https://forum.51nb.com/thread-1916046-1-1.html)
- [关于T440p刷白名单+高级菜单之夹子使用补充（第一次用夹子的请看下，避免绕弯）](https://forum.51nb.com/thread-1991428-1-1.html)

虽然写的是T440p，但是实测对T440s基本适用，本文对以上的材料做下补充：
- BIOS芯片的位置，第二张图是夹上编程器的样子

![](../assets/images/bioschip.jpg)

![](../assets/images/T440s.jpg)

- 用工具读取到的BIOS芯片型号和教程可能不同

- 教程还带有解锁高级菜单的部分，但是实测貌似没什么用（主要想解锁功耗，15w下散热还是压得住的）

### 黑苹果

换了AX200的网卡之后在160MHz下实测无线可以跑满1000Mbps，另外一个在2020年的新闻就是Intel的网卡在MacOS下终于可以日常使用了：[itlwm](https://github.com/OpenIntelWireless/itlwm)，因为手头空闲的机器不多，所以就尝试给T440s上黑苹果试一试，好在Clover的EFI不难找

实测黑苹果日常使用确实比Win10流畅不少（主要是少了一些莫名其妙的高占用），但是多开还是有些吃力的

## M73

缘起于之前为了收一个小机箱和MATX主板，被学长捆绑了一颗ES版的CPU，最开始说是E3，用CPU-Z看也是志强，但是根据顶盖上的四位编号QEDH，在某宝上搜索指向的是i7-4770s的ES版本，4c8t @ 3.0GHz，TDP 83W，并且带核显；尝试过作为家用win10主机使用过，但是核显实在太鸡肋，单核心主频低导致部分应用速度比较慢

最后我还是决定装一台低功耗的Mini Server，准系统选择了Haswell时代的M73，因为相比戴尔的9020散热更好，相比HP的小主机，M73可以用Thinkpad系的方口电源也是一大优势（家里方口电源多...）

### 装机及散热测试
准系统的安装很简单，但是这里有些插曲，一个是CPU的散热问题，相比带T后缀的45W的低功耗CPU，以及家用台式机的65W标压处理器，QEDH的83W还是很吓人的，之前在B85 ATX主板上实测，解锁功耗烤机满载也确实可以跑到80W以上，所以对于65W以下的准系统M73，我选择了把顶盖到Die的导热材料换成液金，然后测试下M73能否在编译时跑满这颗CPU

这里选择CentOS下，初次编译OpenWrt作为负载，编译时，温度抵近90度，基本上就是达到默认风扇下的上限了，如果把风扇转速拉满，温度大概80左右，但是噪音太大了；使用s-tui可以在软件层面查看功耗和频率信息，CPU功耗接近60W，主频2.8GHz，因为可以用Type-C的方口诱骗线供电，所以也顺带看了下整机的功耗，基本在65W以下（用95W的电源）；编译的时间对比win10下使用WSL2的i7-8700K，M73这套平台是其两倍，可以接受的水平；另外日常待机功耗12W左右

### Docker下载机
之前用NAS时，用的比较多的就是Docker版本的qBittorrent以及网络共享功能，其实在Linux下实现这些并不难，M73用这一套的优势是相比传统NAS噪音很小，且小体型放置随意；这里使用和NAS上一样的容器镜像源，主要是qBittorrent作为下载软件，Nextcloud挂载SMB提供多种外部访问方式（体验下来还是SMB为主，Nextcloud的客户端能做的事情有限）

以下设置仅针对CentOS 7.9，其他发行版可能不同，其中Docker启动命令如下（记得建立挂卷的目录）：

```bash
docker run -d \
  --name=nextcloud \
  --network=host \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Asia/Shanghai \
  -v /home/nextcloud/config:/config \
  -v /home/nextcloud/data:/data \
  --restart unless-stopped \
  linuxserver/nextcloud
  
docker run -d \
  --name=qbittorrent \
  --network=host \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e WEBUI_PORT=8080 \
  -v /home/qbittorrent/config:/config \
  -v /home/downloads:/downloads \
  --restart unless-stopped \
  linuxserver/qbittorrent
```

### 可能遇到的问题

1. 3.1X的内核可能存在qbt启动异常的问题，```/usr/bin/qbittorrent-nox: error while loading shared libraries: libQt5Core.so.5: cannot open shared object file: No such file or directory```参考[群晖](https://post.smzdm.com/p/a7do76vd/)

2. 防火墙可能会阻止访问8080端口

   ```bash
   firewall-cmd --zone=public --add-port=443/tcp --permanent
   firewall-cmd --reload
   ```

3. nextcloud会识别首次登陆的IP之类的信息

   需要到docker指定的config目录下修改才可以换用域名或者新的IP访问

   ```bash
   vi ./nextcloud/config/www/nextcloud/config/config.php
   ```

   ```php
   <?php
   $CONFIG = array (
     'memcache.local' => '\\OC\\Memcache\\APCu',
     'datadirectory' => '/data',
     'instanceid' => 'xxx',
     'passwordsalt' => 'xxx',
     'secret' => 'xxx',
     'trusted_domains' =>
     array (
       0 => '192.168.8.114',
       1 => preg_match('/cli/i',php_sapi_name())?'127.0.0.1':$_SERVER['SERVER_NAME'],
     ),
     'dbtype' => 'sqlite3',
     'version' => '21.0.0.18',
     'overwrite.cli.url' => 'https://192.168.8.114',
     'installed' => true,
   );
   ```

### SMB共享

1. 基于系统的用户创建SMB用户

2. 设置SMB共享目录

3. 关闭SELinux（否则只能看到目录而看不到文件）

   ```
   临时关闭（不用重启机器）
   setenforce 0  
   
   修改/etc/selinux/config 文件
   将SELINUX=enforcing改为SELINUX=disabled
   重启机器即可
   ```

