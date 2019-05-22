---
layout: article
title: How to OpenWrt
author :
mathjax: true
mermaid: true
chart: true
mathjax_autoNumber: true
tags: 网络  路由器 OpenWrt Howto
key: milestone
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#ffffff'
  background_image:
    src: assets/background/annapurna_dawn.jpg
    gradient: 'linear-gradient(0deg, rgba(0, 0, 0 , .2), rgba(0, 0, 0, .2))'
---
任何事物都是始于生存，发展于社会秩序，娱乐至死的过程
                      ————《只是为了好玩：Linux之父林纳斯自传》

<!--more-->

## 关于OpenWrt
![Wireless Freedom](https://openwrt.org/lib/tpl/openwrt/images/logo.png)

从OpenWrt Logo下面的那个Wireless Freedom讲起吧，几个里程碑
- 让无线设备用上IPv6网络，畅通的使用Google，Youtube，IPv6 PT
- 通过IPv6代理实现免流量高速上网
- 用Aria2挂PT，攒下了可以用几年的上传量
- 在学校限速1Mbps的情况下把网速多拨叠加到6Mbps，逐步改进后又达到了40Mbps,60Mbps
- 最后趁舍友国庆出游，用新的负载均衡方法以及HWNAT把汇聚全宿舍端口达到400Mps
- 接触OpenWrt开发的一些方法，开始根据自己的需求做策略路由

这里的Freedom就是得益于OpenWrt强大的扩展性以及可操作性实现想要的功能

## 如何开始
建议选有官方固件支持，软件支持的路由器（也就是不太建议刷仅有民间固件的那种了，不开源感觉不安全），具体可以参考官方支持的[Hardware Table](https://OpenWrt.org/toh/start)，进入某一款路由器的详情界面就可以看到支持的情况

或者是论坛，一般都会有详细的刷机教程，如国内的[恩山](https://right.com.cn/forum/portal.php),[Koolshare](http://koolshare.cn/portal.php)
>就刷OpenWrt而言，推荐以高通（QAC）和联发科（MTK）或者软路由为主，博通CPU的因为驱动开源的不太好，所以可能会缺少无线功能

就版本的话，主要是稳定版(写这篇文章的时候最新的是18.06.1)和每日构建的版本(Snapshot)
>后者没有自带LuCI界面，需要自己安装，又因为版本太新，不是所有的软件都有已经编译好的ipk，优势在于可以体验到最新的驱动之类的

## 软件推荐
具体的操作这里就不写了，以后有时间写到博客的WiKi一栏，大部分软件使用也比较简单，官方的说明文档和教程都很好找，特别需要提到的是，因为个人用的OpenWrt改动比较大，一些软件由于可定制性有限，尤其是流量统计、速度监测类的软件，部分功能是失效的

### Aria2
这是一个跨平台的多线程下载软件，主要是支持BT，在路由器性能允许的情况下能够做到全天挂PT，并且可以通过网络共享做一个简易的NAS
>之前有一段时间官方源下载的Aria2是不支持BT的，需要自己动手编译，而1.34之后又自带BT支持了

Aria2本身只是一个命令行下载软件，而OpenWrt提供了以个LuCI的配置APP(只能拿来做些设置)luci-app-aria2，所以如果不想用命令行的话就需要一个前端界面，推荐一个[AiraNG](https://github.com/mayswind/AriaNg)

### luci-app-statistics
![collectd](https://img.vim-cn.com/66/cce412e04be5032bddb4aa14a0845f07241647.jpg)
强大的统计软件，和其他的包组合收集各种数据，个人的主要作用就是拿来监测网络延迟，可以提一下的就是能够结合防火墙数据收集实现复杂的流量监控，下面附上官方的WiKi
[luci-app-statistics](https://oldwiki.archive.OpenWrt.org/doc/howto/luci_app_statistics)

### iperf3
跨平台的网络测试工具，主要是拿来测速的，测试一下就知道路由器的性能是什么情况了，比如说5G-5G的传输速度，LAN-5G的传输速度

[iperf3](https://iperf.fr/iperf-download.php)，这里面还有个软件值得推荐[HE.NET-Network Tools](http://networktools.he.net/)

### luci-app-nlbwmon
![nlbwon](https://img.vim-cn.com/00/f96b33b6c0aacd64d92b54f23b755bb86f58f0.png)
对设备流量统计工具，而且是分IPv4和IPv6的，在流量统计软件里算是很美观的了

### luci-wrtbwmon
![mac](https://img.vim-cn.com/92/1b76c51f4991fd5b82f1d7c88a58efd33a917d.jpg)
OpenWrt上少有的分设备的网速监测工具，没有官方的Feed，需要自己去[Github](https://github.com/Kiougar/luci-wrtbwmon)上面下载ipk
>为什么没有人做分设备的实时网速监测的可视化，就像OpenWrt的Realtime Graph一样，之后学了JavaScript我觉得我可以自己写一个

### Netdata
![netdata](https://img.vim-cn.com/e3/44f5fa92845bda4dc5c31193badd6c2da0f87c.jpg)
算是一个比较好看的性能监测界面了，第一次见到还是印象深刻，然而用处...对个人来说不大，效果可以看[Github](https://github.com/netdata/netdata)，在OpenWrt中直接用``opkg install netdata``就好，之后直接访问LuCI管理IP的19999端口就可以看到了，优点还是信息量大，占用低

### FRP
一个用于反向代理的软件，对于没有公网IP的网络接入来说还是挺好用的，一般需要一个拥有公网IP的服务器作为流量的中转，[Github](https://github.com/fatedier/frp)，也有提供FRP服务器的网站可以直接用

## 实用的功能

### 交换机 Switch
这个的用途就是当路由器做路由，占用掉了墙壁内嵌的网口，但是这个时候又有设备需要直接拨号，从前的话可能就需要使用交换机了，但是考虑到现在的路由器都是通过VLAN来划分网口的，而OpenWrt对此是可以自定义的，所以只需要改下网口的VLAN ID就好，因为设备之间有差别，所以这里建议参考[OpenWrt Guide](https://OpenWrt.org/docs/guide-user/network/vlan/switch_configuration)，然后根据自己的需求做设置

### 定时任务 Cron
> **cron** is the general name for the service that runs scheduled actions. 
> **crond** is the name of the daemon that runs in the background and reads crontab files. 
> A **crontab** is a file containing jobs in the format
>minute hour day-of-month month day-of-week  command
>crontabs are normally stored by the system in /var/spool/<username>/crontab. These files are not meant to be edited directly. You can use the crontab command to invoke a text editor (what you have defined for the EDITOR env variable) to modify a crontab file.

There are various implementations of cron. Commonly there will be per-user crontab files (accessed with the command crontab -e) as well as system crontabs in /etc/cron.daily, /etc/cron.hourly, etc.

In your first example you are scheduling a job via a crontab. In your second example you're using the  at command to queue a job for later execution.
LuCI的System->Scheduled Tasks中就是了，和Linux中的Crontab差不多，所以查下就好，算是简单实用的东西了，用来做个定时重启、运行某个脚本都是可以的,下面这个就是定时断开和连接一个PPPoE拨号
```shell
6 * * * * /sbin/ifdown wan
23 40 * * * /sbin/ifup wan
```
### 热插拔脚本 Hotplug
Hotplug功能实际上是相当的实用的，涉及到接口的热插拔到执行脚本 https://github.com/wywincl/hotplug 这篇文章做了详细的剖析，比如说个人就写过一个针对夜间断网的桥接切换脚本
```shell
#!/bin/sh
exec 1>>/root/wwan
exec 2>>/root/wwan
[ "$INTERFACE" = wwan ] || exit 0
status_update(){
  ifstatus=`ubus call network.interface.wwan status | jsonfilter -e "@.up"`
  telstatus=`ubus call network.interface.tel status | jsonfilter -e "@.up"`
  edustatus=`ubus call network.interface.edu status | jsonfilter -e "@.up"`
}
status_echo(){
  echo interface status:
  echo -e " WWAN \t TEL \t EDU"
  echo -e " $ifstatus \t $telstatus \t $edustatus "
}
time_init(){
  date1="23:25:00"
  date2=`date "+%H:%M:%S"`
  date3="06:25:00"
  poff=`date -d "$date1" +%s`
  now=`date -d "$date2" +%s`
  pon=`date -d "$date3" +%s`
}
head_echo(){
  echo -e "\n \n"
  echo "==============================  `date`  ================================"
  status_echo
}
status_update
time_init
time_begin=$(date "+%s")
case "$ACTION" in
  ifup)
  case "$telstatus" in
    true)
     head_echo
     if [ $poff -gt $now ] && [ $now -gt $pon ]; then
       echo "========================== MORNING SWITCH ========================="
       ubus call network.interface.tel down
       sleep 6
       status_update
       if [ "$ifstatus" = true ]; then
         echo "-------------------- MORNING SWITCH SUCESSFUL -------------------"
       else
         echo "------------------=== MORNING SWITCH FAILED ===------------------"
       fi
     else
       echo "---------------------=== MAKE WAY FOR WWAN ===---------------------"
       ubus call network.interface.tel down
       sleep 6
       status_update
       if [ "$ifstatus" = true ]; then
         echo "------------------------ SWITCH SUCESSFUL -----------------------"
       else
         echo "----------------------=== SWITCH FAILED ===----------------------"
       fi
     fi
     exit 0
     ;;
    false)
     #echo "interface wwan is up,interface.tel is down already"
     #status_echo
     exit 0
     ;;
  esac
    ;;
  ifdown)
  if [ $poff -gt $now ] && [ $now -gt $pon ]; then
    head_echo
    echo "-------------------=== WWAN is Offline DAYTIME ===--------------------"
    echo -n "WWAN is Reconnecting"
      until [ "$ifstatus" = true ]; do
        status_update
        sleep 6
        echo -n "."
        time_end=$(date "+%s")
        duration=$((time_end - time_begin))
        if [ "$ifstatus" = true ]; then
          echo ;
          echo "----------------------- WWAN Connected -------------------------"
          exit 0
        elif [ $duration -gt 30 ]; then
          echo ;
          echo "---------------- Connect TIMEOUT & Switch to EDU ---------------"
          exit 0
        fi
      done
  else
    head_echo
    echo "============================ NIGHT SWITCH ============================"
    echo -n "TEL is Connecting"
    count=100
    while [ $count -ne 0  ]; do
      ubus call network.interface.tel up
      retry=6
      sleep $retry
      count=$(($count - 1))
      time_sleep=$(((100 - count)*retry))
      status_update
      if [ "$telstatus" = true ]; then
        time_end=$(date "+%s")
        duration=$((time_end - time_begin))
        echo ;
        echo "Tried $((100-count)) times"
        echo "--------------------- NIGHT SWITCH SUCCESSFUL --------------------"
        echo "Time used: $duration seconds"
        echo "----------------------------- `date` -----------------------------"
        exit 0
      elif [ "$ifstatus" = true ]; then
        echo ;
        echo "------------------------ WWAN Reconnected --------------------------"
        echo "-----------------------====== `date` ======-----------------------"
        exit 0
      elif [ $count -eq 0 ]; then
        echo ;
        echo "---------------- Connect TIMEOUT & Switch to EDU -----------------"
        echo "Time used: $duration seconds"
        echo "----------------------------- `date` -----------------------------"
        status_update
        status_echo
        exit 0
      else
        echo -n "."
      fi
    done
    fi
  ;;
esac
```

### 系统日志 Logger
在OpenWrt中可通过```logread```命令查看运行时的log日志

使用```logger -t IPv6 "Add good IPv6 route..."```

就可以添加log并且打上标签，打标签的方便之处在于调取日志的时候可以根据标签来筛选```logread | grep IPv6```

### 消息通信 ubus

之所以会关注这个也是因为之前在改进NAT6脚本的时候需要改进路由表条目添加的部分

```shell
gw=$(ip -6 route show default | grep $iface_route | sed 's/from [^ ]* //' | head -n1)
```

而在OpenWrt WiKi中有提到在设置时应该尽量避免使用grep之类的文本命令

> For developers requiring automatic parsing of the UCI configuration, it is therefore redundant, unwise, and inefficient to use `awk` and `grep` to parse OpenWrt's config files. The `uci` utility offers all functionality with respect to modifying and parsing UCI.

但是经过个人反复的查询都没有找到替换上面的代码的UCI代码，直到无意间看到了ubus部分，发现它提供了进程间的通信，更具体的是管理网络接口和路由功能的netifid进程可以通过ubus来调用

这里就有两种等效的命令

- /sbin目录下的Shell脚本: /sbin/ifstatus

- ubus中的查询命令: ubus call network.interface dump

```shell
/sbin/ifstatus edu_6
ubus call network.interface dump | jsonfilter -e '$.interface[@.interface="edu_6"]'
```

因为返回的都是json格式的文本，因而提取信息需要解析，这里又有两种内置的方法

- libubox库的jshn软件
- jsonfilter工具

具体的使用

```shell
root@LEDE:~# source /usr/share/libubox/jshn.sh
root@LEDE:~# data=$(/sbin/ifstatus edu_6)
root@LEDE:~# json_init
root@LEDE:~# json_load "$data"
root@LEDE:~# echo $data
{ "up": true, "pending": false, "available": true, "autostart": true, "dynamic": true, "uptime": 141048, "l3_device": "pppoe-edu", "proto": "dhcpv6", "device": "pppoe-edu", "metric": 0, "dns_metric": 0, "delegation": true, "ipv4-address": [ ], "ipv6-address": [ { "address": "2001:xxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx", "mask": 64, "preferred": 171757, "valid": 258157 } ], "ipv6-prefix": [ ], "ipv6-prefix-assignment": [ ], "route": [ { "target": "2001:xxx:xxxx:xxxx::", "mask": 64, "nexthop": "::", "metric": 256, "valid": 258157, "source": "::\/0" }, { "target": "::", "mask": 0, "nexthop": "fe80::96db:daff:fe3e:8fcf", "metric": 512, "valid": 757, "source": "2001:xxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx\/64" } ], "dns-server": [ ], "dns-search": [ ], "inactive": { "ipv4-address": [ ], "ipv6-address": [ ], "route": [ ], "dns-server": [ ], "dns-search": [ ] }, "data": { "zone": "wan" } }
root@LEDE:~# json_get_var iface_up up
root@LEDE:~# echo $iface_up
1

root@LEDE:~# json_get_keys keys
root@LEDE:~# echo $keys
up pending available autostart dynamic uptime l3_device proto device metric dns_metric delegation ipv4_address ipv6_address ipv6_prefix ipv6_prefix_assignment route dns_server dns_search inactive data

root@LEDE:~# json_select route
root@LEDE:~# json_get_keys keys
root@LEDE:~# echo $keys
1 2
root@LEDE:~# json_select 2
root@LEDE:~# json_get_keys keys
root@LEDE:~# echo $keys
target mask nexthop metric valid source

root@LEDE:~# json_get_var gw nexthop
root@LEDE:~# echo $gw
fe80::96db:daff:fe3e:8fcf
```

可能是我使用的方法不太对导致代码很长，不过jsonfilter使用jsonpath来解析json，相比之下就要简短的多

```shell
/sbin/ifstatus edu_6 | jsonfilter -e '$.route[1].nexthop'
```

由于ubus提供的信息非常的多，对于IPv6 NAT下的负载均衡就兼顾简洁和适用性的写法了，对于没有负载均衡的情况下也可以使用，按照规则保存在hotplug文件夹下即可

```shell
#!/bin/sh
[ "$ACTION" = ifup ] || exit 0
ifaces=$(ubus call network.interface dump | jsonfilter -e '$.interface[@.proto="dhcpv6" && @.up=true].interface')
for iface_6 in $ifaces
do
  [ "$INTERFACE" = $iface_6 ]  || continue
  devices=$(ubus call network.interface dump | jsonfilter -e '$.interface[@.proto="dhcpv6" && @.up=true].device')
  ipv6_gw=$(ifstatus $iface_6 | jsonfilter -e '$.route[1].nexthop')
  for ipv6_dev in $devices
  do
    status=$(route -A inet6 add 2000::/3 gw $ipv6_gw dev $ipv6_dev 2>$1)
    logger -t NAT6 "Gateway: $ipv6_dev: Done $status"
  done
  exit 0
done
```

其中，ifaces是筛选出协议为ducpv6的所有活动端口的接口名称，比如说wan_6,edu_6，它们的接口状态中是有IPv6的路由信息的，主要就是上级网关，而devices是上述接口的设备名称，用于添加路由表时指定转发出去的接口，加上循环也就可以批量添加路由表条目了
