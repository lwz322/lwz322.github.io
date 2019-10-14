---
layout: article
title: How to OpenWrt
author :
mathjax: true
mermaid: true
chart: true
mathjax_autoNumber: true
tags: 网络 OpenWrt Howto
key: milestone
sidebar:
  nav: /2018/10/08/Milestone.html
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#ffffff'
  background_image:
    src: assets/background/hua.jpg
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
- 使用策略路由和iptables灵活地配置Linux路由
- 移植和修改源码以实现功能和硬件的适配

这里的Freedom就是得益于OpenWrt强大的扩展性以及可操作性实现想要的功能，尽管在官网的[Reasons to use OpenWrt](https://openwrt.org/reasons_to_use_openwrt)已经有了一些说明，本文从个人用户的视角介绍一些OpenWrt实用的软件和功能

> 关于开源的路由器系统的稳定性问题，众说纷纭，OpenWrt支持的硬件多，针对某一特定的硬件的稳定性是比较随缘的，尤其是部分路由器的硬件设计上的稳定性就一般，如果准备在稳定性要求较高的环境使用，要谨慎（使用的工具越简单越好）

## 如何开始
建议选有官方固件支持，软件支持的路由器（也就是不太建议刷仅有民间固件的那种了，不开源感觉不安全），具体可以参考官方支持的[Hardware Table](https://OpenWrt.org/toh/start)，进入某一款路由器的详情界面就可以看到支持的情况

或者是论坛，一般都会有详细的刷机教程，如国内的[恩山](https://right.com.cn/forum/portal.php)，[Koolshare](http://koolshare.cn/portal.php)
>就刷OpenWrt而言，推荐以高通（QAC）和联发科（MTK）或者软路由为主，博通CPU的因为驱动开源的不太好，所以可能会缺少无线功能

就版本的话，主要是稳定版(写这篇文章的时候最新的是18.06.4)和每日构建的版本(Snapshot)
>后者没有自带LuCI界面，需要自己安装，又因为版本太新，不是所有的软件都有已经编译好的ipk，优势在于可以体验到最新的驱动之类的

国内也有各种个人修改的版本，比较出名的：[Lean's OpenWrt source](https://github.com/coolsnowwolf/lede)，作者现在只提供源码，网上有很多编译好的版本，内置了一些常用的软件以及“魔改”，如果不想折腾太多而获得一系列的功能可以考虑，缺点就是自带的配置可能会和要做的配置冲突，因为为了统一标准，大多数人都会以原生的OpenWrt上配置，比如说一些多拨固件默认开启的负载均衡和IPv6功能有冲突

当然也可以自己编译，因为受限与路由器的存储空间和性能，固件的Linux内核被精简，部分软件也被精简了，比如说某些功能的实现就依赖于完全体的dnsmasq-full，推荐在编译时就处理好这个倚赖，对于Snapshot版本而言，官方仓库里没有预编译软件包或者系统不支持ipk安装，又或者发行版的软件仓库中收录的软件版本不合适，这些都需要自行编译解决，这里可以参考[编译OpenWrt Snapshot固件](https://lwz322.github.io/2019/08/31/Build_OpenWrt_snapshot.html)

## OpenWrt软件推荐
官方的[Ueser Guide](https://openwrt.org/docs/guide-user/start)以及[Old Wiki](https://oldwiki.archive.openwrt.org/doc/howto/start)(看起来简洁一些)从功能上对软件划分，相当全面和详细的介绍了OpenWrt的功能及其实现的软件，这里主要是推荐一下个人用过的，体验还OK的部分软件

### Aria2
这是一个跨平台的多线程下载软件，主要是支持BT，在路由器性能允许的情况下能够做到全天挂PT，并且可以通过网络共享做一个简易的NAS
>之前有一段时间官方源下载的Aria2是不支持BT的，需要自己动手编译，而1.34之后又自带BT支持了

Aria2本身只是一个命令行下载软件，而OpenWrt提供了以个LuCI的配置APP(只能拿来做些设置)luci-app-aria2，所以如果不想用命令行的话就需要一个前端界面，推荐一个[AiraNG](https://github.com/mayswind/AriaNg)

最近的OpenWrt使用luci-app-aria2启动不了Aria2，需要输入下面的命令才会在后台运行
```bash
aria2c --enable-rpc --rpc-listen-all=true --rpc-allow-origin-all -c -D
```

至于网页打开的那种AriaNg，OpenWrt的仓库那个比较老了，可以先安装一个AraiNg的IPK，之后再到AriaNG的release下载一个新版的html文件替换掉，但是如果是打开外网访问的话，AriaNg是没有什么安全措施的

另外一点就是Aria2的内存消耗太恐怖了，大量的内存被用于缓存，这对主路由是极为不利的

### luci-app-statistics
![collectd](https://img.vim-cn.com/66/cce412e04be5032bddb4aa14a0845f07241647.jpg)
强大的统计软件，和其他的包组合收集各种数据，个人的主要作用就是拿来监测网络延迟，可以提一下的就是能够结合防火墙数据收集实现复杂的流量监控，下面附上官方的WiKi
[luci-app-statistics](https://oldwiki.archive.OpenWrt.org/doc/howto/luci_app_statistics)

### iperf3
跨平台的网络测试工具，主要是拿来测速的，测试一下就知道路由器的性能是什么情况了，比如说5G-5G的传输速度，LAN-5G的传输速度

[iperf3](https://iperf.fr/iperf-download.php)，这里面还有个移动端软件值得推荐[HE.NET-Network Tools](http://networktools.he.net/)，其中也包含了iperf3网络测速感觉

如果要测试较为极限的吞吐能力可以用参数```-P 10```

至于要测试路由器到互联网的上下行速度可以参考附录中的命令行Speedtest

### mtr
mtr是ping和traceroute的结合，网络问题排查的利器

### luci-app-ddns
这个我是在做[家用宽带的IPv6配置](https://lwz322.github.io/2019/07/25/IPv6_Home.html)的时候用到的，相比与传统路由器支持数量极为有限的几个DDNS，这个简直强大太多，因为软件本身做好DDNS客户端的外围工作，至于各个DDNS供应商的适配可以由脚本完成，比如说[Sensec](https://github.com/sensec)写的[[分享]适用于OpenWRT/LEDE自带DDNS功能的阿里云脚本](https://www.right.com.cn/forum/thread-267501-1-1.html)

### luci-app-nlbwmon
![nlbwon](https://img.vim-cn.com/00/f96b33b6c0aacd64d92b54f23b755bb86f58f0.png)
对设备流量统计工具，而且是分IPv4和IPv6的，在流量统计软件里算是很美观的了

### luci-wrtbwmon
![mac](https://img.vim-cn.com/92/1b76c51f4991fd5b82f1d7c88a58efd33a917d.jpg)
OpenWrt上少有的分设备的LuCi界面下的网速监测工具，没有官方的Feed，需要自己去[Github](https://github.com/Kiougar/luci-wrtbwmon)上面下载ipk

### iftop
在Linux用于监测网卡/接口的网络连接情况，在路由器上可以显示内外网连接的地址，以及连接的速度，相比上面的LuCi界面的网速监控简直强大太多了

### Netdata
![netdata](https://img.vim-cn.com/e3/44f5fa92845bda4dc5c31193badd6c2da0f87c.jpg)
算是一个比较好看的性能监测界面了，第一次见到还是印象深刻，然而用处...对个人来说不大，效果可以看[Github](https://github.com/netdata/netdata)，在OpenWrt中直接用``opkg install netdata``就好，之后直接访问LuCI管理IP的19999端口就可以看到了，优点还是信息量大，占用低

### FRP
一个用于反向代理的软件，对于没有公网IP的网络接入来说还是挺好用的，一般需要一个拥有公网IP的服务器作为流量的中转，具体的用法，以及ipk下载可以参考[Github](https://github.com/fatedier/frp)，也有提供FRP服务器的网站可以直接用，值得一提的是，学校的教育网一般是阻挡了传入连接的，如果有在学校里搭建NAS，想要远程访问，直接用地址是行不通的，这个时候FRP就可以通过家里的公网IP服务器中转对学校的NAS进行访问，在学校如果给代理服务器穿透的话就相当于可以使用学校的网络了

## 实用的功能
脱离了简单的应用软件层面，多多少少需要点shell编程

### 交换机 Switch
这个的用途就是当路由器做路由，占用掉了墙壁内嵌的网口，但是这个时候又有设备需要直接拨号，从前的话可能就需要使用交换机了，但是考虑到现在的路由器都是通过VLAN来划分网口的，而OpenWrt对此是可以自定义的，所以只需要改下网口的VLAN ID就好，因为设备之间有差别，所以这里建议参考[OpenWrt Guide](https://OpenWrt.org/docs/guide-user/network/vlan/switch_configuration)，然后根据自己的需求做设置

### 定时任务 Cron
> **cron** is the general name for the service that runs scheduled actions. 
> **crond** is the name of the daemon that runs in the background and reads crontab files. 
> A **crontab** is a file containing jobs in the format
>minute hour day-of-month month day-of-week  command
>crontabs are normally stored by the system in /var/spool/<username>/crontab. These files are not meant to be edited directly. You can use the crontab command to invoke a text editor (what you have defined for the EDITOR env variable) to modify a crontab file.

>There are various implementations of cron. Commonly there will be per-user crontab files (accessed with the command crontab -e) as well as system crontabs in /etc/cron.daily, /etc/cron.hourly, etc.

LuCI的System->Scheduled Tasks中就是了，和Linux中的Crontab差不多，所以查下就好，算是简单实用的东西了，用来做个定时重启、运行某个脚本都是可以的,下面这个就是定时断开和连接一个PPPoE拨号
```shell
* 6 * * * /sbin/ifdown wan
40 23 * * * /sbin/ifup wan
```
需要注意的是初次运行要在启动项界面重启下cron

### 热插拔脚本 Hotplug
Hotplug功能实际上是相当的实用的，涉及到接口的热插拔时就会触发，[这篇文章](https://github.com/wywincl/hotplug)做了详细的剖析，比如说个人就写过一个针对夜间断网的桥接切换脚本（写的比较随便，有点长，见文末），更加实用的是对路由表的修改，比如自定义策略路由的时候，在接口断开重连后维护路由表173
```bash
#!/bin/sh
[ "$ACTION" = ifup ] || exit 0
ip route add 10.173.0.0/16 via 10.170.72.254 dev pppoe-VWAN31 table 173
ip route add 10.170.0.0/16 via 10.170.72.254 dev pppoe-VWAN22 table 173
ip route add 10.177.0.0/16 via 10.170.72.254 dev pppoe-VWAN22 table 173
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

- ubus中的查询命令: ubus call network.interface.wan_6 status

或者 ```ubus call network.interface dump | jsonfilter -e '$.interface[@.interface="wan_6"]'```

```shell
/sbin/ifstatus edu_6
ubus call network.interface dump | jsonfilter -e '$.interface[@.interface="wan_6"]'
```

因为返回的都是json格式的文本，因而提取信息需要解析，这里又有两种内置的方法

- libubox库的jshn软件
- jsonfilter工具

具体的使用

```shell
root@LEDE:~# source /usr/share/libubox/jshn.sh
root@LEDE:~# data=$(/sbin/ifstatus wan_6)
root@LEDE:~# json_init
root@LEDE:~# json_load "$data"
root@LEDE:~# echo $data
{ "up": true, "pending": false, "available": true, "autostart": true, "dynamic": true, "uptime": 141048, "l3_device": "pppoe-wan", "proto": "dhcpv6", "device": "pppoe-wan", "metric": 0, "dns_metric": 0, "delegation": true, "ipv4-address": [ ], "ipv6-address": [ { "address": "2001:xxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx", "mask": 64, "preferred": 171757, "valid": 258157 } ], "ipv6-prefix": [ ], "ipv6-prefix-assignment": [ ], "route": [ { "target": "2001:xxx:xxxx:xxxx::", "mask": 64, "nexthop": "::", "metric": 256, "valid": 258157, "source": "::\/0" }, { "target": "::", "mask": 0, "nexthop": "fe80::96db:daff:fe3e:8fcf", "metric": 512, "valid": 757, "source": "2001:xxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx\/64" } ], "dns-server": [ ], "dns-search": [ ], "inactive": { "ipv4-address": [ ], "ipv6-address": [ ], "route": [ ], "dns-server": [ ], "dns-search": [ ] }, "data": { "zone": "wan" } }
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
/sbin/ifstatus wan_6 | jsonfilter -e '$.route[1].nexthop'
```

另外还有一种写法
```shell
ubus call network.interface dump | jsonfilter -e '$.interface[@.interface="wan_6"].route[1].nexthop'
```

使用Jsonpath的时候如果遇到特殊字符可以给节点名加中括号来处理，获取接口的IPv6地址如下
```bash
/sbin/ifstatus wan_6 | jsonfilter -e '$["ipv6-address"][0]["address"]'
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

其中，ifaces是筛选出协议为DHCPv6的所有活动端口的接口名称，比如说wan_6，edu_6，它们的接口状态中是有IPv6的路由信息的，主要就是上级网关，而devices是上述接口的设备名称，用于添加路由表时指定转发出去的接口，加上循环也就可以批量添加路由表条目了（脚本是针对单一的上级网关的）

## 方便的外部管理工具

### SSH使用私钥认证
在版本较新的Windows 10中已经内置了OpenSSH，可以用SSH和SCP，linux发行版基本上是自带了，使用私钥登陆可以免去输密码，也方便写脚本

生成密钥，默认暂时是RSA-2048bit
```
ssh-keygen
```
之后会提示输入密钥的保存路径，如果是初次设置默认就好，否则会覆盖掉之前的文件
```
 ~: [00:42:02]
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\USER/.ssh/id_rsa):
```
完成后会在指定的目录生成指定文件名的两个文件：私钥id_rsa和公钥id_rsa.pub，公钥保存在ssh-server中就好，OpenWrt默认使用Dropbear作为SSH工具，其公钥保存在```/etc/dropbear/authorized_keys```中（如果没有这个文件就新建），把公钥的文件内容复制进去就好，或者在LuCi界面的/cgi-bin/luci/admin/system/admin找到SSH-Keys，把公钥放进去

之后只要使用的SSH客户端带有私钥就可以直接使用SSH和SCP

### 使用SCP传输文件
和SSH配套的软件，SCP从字面上理解就是安全的远程cp命令：
```bash
#复制本地多个文件
$ scp foo.txt bar.txt username@remotehost:/path/directory/

#复制远程多个文件
$ scp username@remotehost:/path/directory/\{foo.txt,bar.txt\} .
```

## 附录

### 命令行Speedtest网络测速（需要Python）
偶尔有这种需求，OpenWrt安装Python要注意空间占用
```shell
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -
```

### 脚本
夜间断网切换脚本，针对的是夜间宿舍断电，UPS只能给一台路由器供电，此时需要把另外一台路由器的PPPoE账号转移到UPS的路由上，如果中途另外一台路由器恢复网络则会断开之前切换而来的PPPoE拨号
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