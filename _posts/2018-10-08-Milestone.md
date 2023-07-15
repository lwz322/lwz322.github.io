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

# 关于OpenWrt
![Wireless Freedom](https://openwrt.org/lib/tpl/openwrt/images/logo.png)

从OpenWrt Logo下面的那个Wireless Freedom讲起吧，几个里程碑：
- 在没有PD的情况下，通过IPv6 NAT，使无线设备用上IPv6
- 通过IPv6代理，在校园网下实现免流量高速上网
- 用Aria2挂PT，攒下了可以用几年的上传量（用Aria2伪装成别的BT软件可能导致封号，但是当时只有一台路由器能跑Aria2这种轻量级BT软件）
- 在学校限速1Mbps的情况下把网速多拨叠加到6Mbps，逐步改进后又达到了40Mbps，60Mbps
- 最后趁舍友国庆出游，用新的负载均衡方法以及HWNAT把汇聚全宿舍端口达到400Mps
- 使用策略路由和iptables灵活地配置Linux路由
- 移植和修改源码以实现功能和硬件的适配
- 使用VLAN实现单线复用，进而聚合多个网口

这里的Freedom就是得益于OpenWrt强大的扩展性以及可操作性实现想要的功能，尽管在官网的[Reasons to use OpenWrt](https://openwrt.org/reasons_to_use_openwrt)已经有了一些说明，本文从个人用户的视角介绍一些OpenWrt实用的软件和功能

> 关于开源的路由器系统的稳定性问题，众说纷纭，OpenWrt支持的硬件多，针对某一特定的硬件的稳定性是比较随缘的，尤其是部分路由器的硬件设计的稳定性就一般，如果准备在稳定性要求较高的环境使用，要谨慎（使用的工具越简单越好）

# 如何开始
## 硬件
建议选有官方固件支持的路由器（也就是不太建议刷仅有民间固件的设备，可能导致使用体验不佳甚至是安全问题），具体可以参考官方支持的[Hardware Table](https://OpenWrt.org/toh/start)，进入某一款路由器的详情界面就可以看到支持的情况

论坛一般都会有详细的刷机教程，如国内的[恩山论坛](https://right.com.cn/forum/portal.php)的氛围就很好

### 平台差异

就刷OpenWrt的机器的CPU平台而言，推荐以高通（QAC）和联发科（MTK）或者软路由为主，博通CPU的因为驱动开源的不太好，所以可能会缺少无线功能（如果有能用的闭源驱动也不错），MTK平台官方固件可以直接使用开源的[MT76项目](https://github.com/openwrt/mt76)编译驱动，大部分也有闭源驱动可以用

> OpenWrt的无线能用但是性能大概率还是不及原厂，总之不对OpenWrt的无线期望太高

另外就是CPU架构及硬路由的特性支持了，ARM架构的算力更强，一般有针对常用加密算法的CPU指令（可以加速），老的MIPS平台如MT7621有HWNAT这种硬路由才有的转发卸载能力，能在更低的功耗下带来强劲的转发性能

关于散热和无线方案的情况，可以参考[acwifi](https://www.acwifi.net)的拆机
### 闪存及内存需求

当前我接触到的最小的是16MB闪存，一般大概还有一半的空间用来装插件，用来做家庭的主路由还是可以的；最少128MB的内存，考察23年的主流应该是256MB了；如果有更大的闪存和内存自然有更大的自由，这往往也是用来划分高中低端的标志

### 有线与无线速率

网口的数量一般看路由器的产品形态，做主路由的话还是网口越多越快越好，如果有USB口的话还可以用来扩展额外的2.5Gb网口，这几年的路由器的网口是越来越少了，5个网口基本上只存在于中高端路由器上了，至于2.5G网口也是高端路由器一般才有，当前千兆宽带（200M以上的都算）和AX 3000的WiFi6（实测一般可以跑到800Mbps~1.6Gbps）恰好卡在千兆网口左右，对于内网传输速度有要求的还是尽量2.5G网口；至于10G网口，当前支持OpenWrt的硬路由不多，而且10G电口往往发热量较大

## 固件
### 官方固件
官方编译的版本主要是稳定版（写这篇文章的时候最新的是18.06.4，23年更新到21.02.6后刷新了部分段落）和每日构建的版本(Snapshot)、以及rc版本：
- 稳定版用的最广泛，最推荐
- rc版本适合在稳定版之前尝鲜新特性，在稳定版出来之前又软件源，稳定版发布后，基本上可以用稳定版的软件源
- Snapshot版本随着代码合入更新，有最快的设备支持，适合开发者，没有自带LuCI界面，需要自己安装，软件源也是快速滚动更新，而且软件源的包相对稳定版的少

### 第三方固件
在各种论坛里面推广/自编译分享的多是这一种，如果不想折腾太多就获得一系列的功能可以考虑

最出名的：[Lean's OpenWrt source](https://github.com/coolsnowwolf/lede)，作者现在只提供源码，主要特点：
- 有更广泛的设备支持和更早的适配
- 内置了部分实用的软件，适应国情，如多拨助手等
- 功能性的“魔改”，如Full Cone NAT，DNS加速
- 更新速度比官方快，但是从在线的软件源安装插件不是很方便

当前我个人最推荐是[ImmortalWrt](https://downloads.immortalwrt.org/)，项目开始的比较晚，而且整体还是跟随官方分支，新增设备支持的速度和广度稍落后（比如闪存小的设备装插件比较吃力），但是拥有和官方一样的随版本的软件源、有适用于国内使用的插件（同样的版本的OpenWrt一般也能用ImmortalWrt的软件源）

另外还有pandorabox，据说多拨比较厉害

第三方固件的缺点是自带的配置可能会和要做的配置冲突，而为了统一标准，大多数教程都会以在官方默认设置的OpenWrt上配置为准，比如说一些多拨固件默认的负载均衡设置会导致IPv6无法使用，虽然可以解决，但是这种折腾的必要性见仁见智

### 自编译
当然也可以自己编译，因为受限与路由器的存储空间和性能，固件的Linux内核被精简，部分软件也被精简了，比如说某些功能的实现就依赖于完全体的dnsmasq-full，推荐在编译时就处理好这个倚赖；对于Snapshot版本，官方仓库里缺少部分预编译软件包，又或者软件依赖不匹配，这些都需要自行编译解决，这里可以参考[编译OpenWrt Snapshot固件](https://lwz322.github.io/2019/08/31/Build_OpenWrt_snapshot.html)，一般还是推荐使用稳定版的源码编译

自编译也可以用到闭源驱动的源码，比如K2P：[为斐讯K2P编译OpenWRT LEDE，并启用mtk闭源wifi驱动](https://www.asmodeus.cn/archives/728)就用到了[mtk-openwrt-feeds](https://github.com/Nossiac/mtk-openwrt-feeds)
### 版本选择
这里细说的话涉及到以下的方面：
- Uboot/Breed等bootloader的选择，如Breed可以超频，部分bootloader有“刷不死”的特性
- 刷机的闪存布局，这个会影响到升级的兼容性，一般来说官方OpenWrt的前后兼容性会比较好
- OpenWrt大版本的新特性，例如OpenWrt 19的客户端渲染的新LuCI，OpenWrt 20的DSA架构交换机
- 自带软件源的软件的新版本的新特性
- 开源和闭源驱动，无线固件的选择

# 软件推荐
官方的[Ueser Guide](https://openwrt.org/docs/guide-user/start)以及[Old Wiki](https://oldwiki.archive.openwrt.org/doc/howto/start)(看起来简洁一些)从功能上对软件划分，相当全面和详细的介绍了OpenWrt的功能及其实现的软件，这里主要是推荐一下个人用过的，体验还OK的部分软件

前半部分主要是OpenWrt下常用的Linux平台下的命令行工具：

### iperf3
跨平台的网络测试工具，主要是拿来测速的，测试一下就知道路由器的性能是什么情况了，比如说5G-5G的传输速度，LAN-5G的传输速度

[iperf3](https://iperf.fr/iperf-download.php)，这里面还有个移动端软件值得推荐[HE.NET-Network Tools](http://networktools.he.net/)，其中也包含了iperf3网络测速工具

如果要测试较为极限的吞吐能力可以用参数```-P 10```，默认参数测试的是上行，测试下行需要加上参数`-R`，默认服务端的一个端口同时只能一次测试一个客户端

> 至于要测试路由器到互联网的上下行速度可以参考附录中的命令行Speedtest

在最新的OpenWrt的package仓库中新加入了一个测试网络性能的命令行工具：[Netperf](https://github.com/openwrt/packages/tree/master/net/speedtest-netperf/files)

### mtr
mtr是ping和traceroute的结合，网络问题排查的利器

### iftop
在Linux用于监测网卡/接口的网络连接情况，在路由器上可以显示内外网连接的IP地址，以及连接的速度；个人的一个重要的用途就是用来查看多WAN接入的负载均衡的效果

另外因为OpenWrt一直以来缺少一个用来实时查看客户端的网络上下行速率的LuCI插件，iftop也是命令行里最主要用来监控网速的工具

### socat
在OpenWrt自带的iptables命令行和LuCI防火墙设置都支持端口转发，前者需要熟悉命令行，后者仅支持IPv4，当前公网IPv4越来越少，家用的公网访问主要还是用的IPv6，对于路由器下层的设备，可以用一行命令将端口重定向（命令是需要常驻命令行才生效的，可以用tmux里运行确保不会因为SSH断开而停止）

```bash
socat TCP6-LISTEN:123,reuseaddr,fork TCP6:[2604:xxx]:456
```
用了一段时间发现：
- 支持IPv4到IPv6的端口转发
- 新版本支持解析域名（但是是否支持实时解析就不清楚了）

这就有更大的想象空间了，比如国内不同地域的网络连接性是不同的，可以用这种方法去规避连接性的问题

### gost
相比socat的端口转发针对单一端口，gost的作用是提供一行命令构建代理服务器

### tcpdump
一个运行在命令行下的嗅探工具，它允许用户拦截和显示发送或收到过网络连接到该计算机的TCP/IP和其他数据包。具体的使用方法网上的教程很多，这里就给出抓取WiFi或者路由器上任意接口的包的一个快捷的方法：使用Wireshark的SSH远程抓取功能，在Wireshark进入界面的捕获一栏下可以看到这个功能；设置好SSH的相关参数后在Capture可以设置抓取的命令，在这里使用tcpdump：
```bash
tcpdump -U -s 0 -i br-lan -w -
```
- -U 是让数据包打印时直接输出到stdout而不是在输出缓存满后再输出
- -i 是指定需要抓取的路由器的interface
- -s 是指定包大小，参数为0表示最大
- -w 是指定输出文件 当参数为-表示输出到当前stdout

之后在Wireshark的图形化界面中就可以实时看到抓包的情况了

### FRP
一个用于反向代理的软件，在无公网IP的情况下也可以做到远程访问，一般需要一个拥有公网IP的服务器作为流量的中转，具体的用法可以参考[Github](https://github.com/fatedier/frp)，其中也提供了各个平台架构的二进制文件

更加推荐的是：[kuoruan/openwrt-frp](https://github.com/kuoruan/openwrt-frp)，其提供的ipk文件安装后的空间占用更小，更加适合在小存储空间的路由器中使用，另外作者还编写了[kuoruan/luci-app-frpc](https://github.com/kuoruan/luci-app-frpc)

因为[作者觉得没有必要](https://github.com/kuoruan/luci-app-frpc/issues/4)，所以没有写frps的luci-app，因为个人的学校的教育网阻挡了传入连接的，偶尔需要远程访问校内的资源，所以就使用FRP连接家里的公网IPv6路由器中转对学校的资源进行访问，一开始用 screen + crontab 写了一个“守护进程”，在路由器和VPS都可以用

但是还是不太优雅的，刚好看了下上面的luci-app-frpc的代码，发现用到了OpenWrt内建的procd，于是就本着学习的目的写了一个[lwz322/luci-app-frps](https://github.com/lwz322/luci-app-frps)，欢迎提issue和PR

### luci-app-statistics
![collectd](https://cdn.jsdelivr.net/gh/lwz322/pics/github.io/Collectd.jpg)
强大的统计软件，和其他的包组合收集各种数据，个人的主要作用就是拿来监测网络延迟，可以提一下的就是能够结合防火墙数据收集实现复杂的流量监控，下面附上官方的WiKi
[luci-app-statistics](https://oldwiki.archive.OpenWrt.org/doc/howto/luci_app_statistics)

### luci-app-ddns
这个我是在做[家用宽带的IPv6配置](https://lwz322.github.io/2019/07/25/IPv6_Home.html)的时候用到的，相比与传统路由器支持数量极为有限的几个DDNS，这个简直强大太多，因为软件本身做好DDNS客户端的外围工作，至于各个DDNS供应商的适配可以由脚本完成比如说[Sensec](https://github.com/sensec)写的[[分享]适用于OpenWRT/LEDE自带DDNS功能的阿里云脚本](https://www.right.com.cn/forum/thread-267501-1-1.html)

> 随着时间的推移，opkg软件源提供的ddns脚本当然会越来越广

### Aria2
这是一个跨平台的多线程下载软件，主要是支持BT，在路由器性能允许的情况下能够做到全天挂PT，并且可以通过网络共享做一个简易的NAS；当然在条件允许的情况下还是建议使用Docker版的qbittorrent
> 之前有一段时间官方源下载的Aria2是不支持BT的，需要自己动手编译，而1.34之后又自带BT支持了

注：大部分的PT不支持使用Aria2作为BT客户端使用，而Aria2也自带了伪装功能，可以伪装成为被允许的客户端，然而部分PT站是可以检测出来的（因为会涉及到流量作弊的问题），所以在PT使用Aria2是可能**被封号**的（PT账号的价值不必多说）

实测北邮人可以检测伪装，北洋园PT也加入了相关代码，参考自：[Pt 站点禁用 Aria2 客户端方法分析](https://blog.rhilip.info/archives/1010/)

Aria2本身只是一个命令行下载软件，而OpenWrt提供了一个LuCI的配置APP(只能拿来做些设置)luci-app-aria2

最近的OpenWrt使用luci-app-aria2启动不了Aria2，需要输入下面的命令才会在后台运行
```bash
aria2c --enable-rpc --rpc-listen-all=true --rpc-allow-origin-all -c -D
```
所以如果不想用命令行的话就需要一个前端界面，推荐[AiraNG](https://github.com/mayswind/AriaNg)，通过RPC与服务端通信，既可以安装到路由器上（貌似不安全），也可以放到终端上（客户端或者HTML）

OpenWrt的仓库那个AriaNG版本比较老了，可以先安装一个AraiNg的IPK，之后再到AriaNG的release下载一个新版的html文件替换掉，但是如果是打开外网访问的话，AriaNg是没有什么安全措施的

另外一点就是Aria2的内存消耗太恐怖了，大量的内存被用于缓存，这对主路由是极为不利的

### luci-app-nlbwmon
![nlbwon](https://cdn.jsdelivr.net/gh/lwz322/pics/github.io/nlbwon.png)
对设备流量统计工具，而且是分IPv4和IPv6的，在流量统计软件里算是很美观的了

### luci-wrtbwmon
![mac](https://cdn.jsdelivr.net/gh/lwz322/pics/github.io/usage.jpg)
OpenWrt上少有的分设备的LuCI界面下的网速监测工具，没有官方的Feed，需要自己去[Github](https://github.com/Kiougar/luci-wrtbwmon)上面下载ipk

### Netdata
![netdata](https://cdn.jsdelivr.net/gh/lwz322/pics/github.io/netdata.jpg)
算是一个比较好看的性能监测界面了，第一次见到还是印象深刻，然而用处...对个人来说不大，效果可以看[Github](https://github.com/netdata/netdata)，在OpenWrt中直接用``opkg install netdata``就好，之后直接访问LuCI管理IP的19999端口就可以看到了，优点还是信息量大，占用低

# 实用的内建功能
脱离了简单的应用软件层面，部分需要shell编程

### 交换机 Switch
`在较新版本的OpenWrt中，部分路由器的交换机模块迁移到了DSA架构，LuCI界面网络选项卡去掉了交换机页面，等摸索之后再补充详细说明`.{:.warning}
> [DSA架构简介](https://forum.openwrt.org/t/mini-tutorial-for-dsa-network-config/96998)

先说一个用途：当路由器做路由，占用掉了墙壁内嵌的网口，但是这个时候又有设备需要直接拨号，从前的话可能就需要使用交换机了，但是现在的大部分路由器都是通过VLAN来划分网口的，而OpenWrt对此是可以自定义的，所以只需要改下网口的VLAN ID就好，比如像下面这样，WAN口和LAN4就相当于“桥接”了，任意一口作为接入的时候，另外一个网口也可以连接电脑拨号
![](https://i.loli.net/2019/11/09/keOTDbEr3oYfc6S.png)

因为设备之间有差别，所以这里建议参考[OpenWrt Guide](https://OpenWrt.org/docs/guide-user/network/vlan/switch_configuration)，然后根据自己的需求做设置

VLAN可以实现相对高级的交换机/路由器的级联，常见的比如：
- IPTV和宽带的单线复用，单臂路由

### 定时任务 Cron
> **cron** is the general name for the service that runs scheduled actions. 
> **crond** is the name of the daemon that runs in the background and reads crontab files. 
> A **crontab** is a file containing jobs in the format
>minute hour day-of-month month day-of-week  command
>crontabs are normally stored by the system in /var/spool/<username>/crontab. These files are not meant to be edited directly. You can use the crontab command to invoke a text editor (what you have defined for the EDITOR env variable) to modify a crontab file.

>There are various implementations of cron. Commonly there will be per-user crontab files (accessed with the command crontab -e) as well as system crontabs in /etc/cron.daily, /etc/cron.hourly, etc.

LuCI的System -> Scheduled Tasks中就是了，和Linux中的Crontab差不多，所以查下就好，算是简单实用的东西了，用来做个定时重启、运行某个脚本都是可以的,下面这个就是定时断开和连接一个PPPoE拨号
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

因为OpenWrt路由器往往内存和存储空间有限，日志文件可以发到远程的机器上以供调试

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

由于ubus提供的信息非常的多，对于IPv6 NAT下的负载均衡就兼顾简洁和适用性的写法了（一般情况下也可用）

```shell
#!/bin/sh
[ "$ACTION" = ifup ] || exit 0
ifaces=$(ubus call network.interface dump | jsonfilter -e '$.interface[@.proto="dhcpv6" && @.up=true].interface')
for iface_6 in $ifaces
do
  [ "$INTERFACE" = $iface_6 ]  || continue
  ipv6_devices=$(ubus call network.interface dump | jsonfilter -e '$.interface[@.proto="dhcpv6" && @.up=true].device')
  nexthops=""
  for ipv6_dev in $ipv6_devices
  do
    VAR=$(echo '$.interface[@.device="'$ipv6_dev'"].route[1].nexthop')
    ipv6_gw=$(ubus call network.interface dump | jsonfilter -e $VAR)
    nexthops=${nexthops}"nexthop via $ipv6_gw  dev $ipv6_dev "
  done
  status=$(ip -6 route replace default metric 1 $nexthops 2>&1)
  break
done
logger -t ECMPv6 "Done $ipv6_devices $status"
```

其中，ifaces是筛选出协议为DHCPv6的所有活动端口的接口名称，比如说wan_6，edu_6，它们的接口状态中是有IPv6的路由信息的，主要就是上级网关，而devices是上述接口的设备名称，用于添加路由表时指定转发出去的接口，加上循环也就可以批量添加路由表条目了（脚本是针对单一的上级网关的）

## SSH与文件传输

### SSH使用私钥认证
在版本较新的Windows 10中已经内置了OpenSSH，可以用SSH和SCP，linux发行版基本上是自带了，使用私钥登陆可以免去输密码，也方便写脚本

生成密钥，默认暂时是RSA-2048bit
```powershell
ssh-keygen
```
之后会提示输入密钥的保存路径，如果是初次设置默认就好，否则会覆盖掉之前的文件
```powershell
 ~: [00:42:02]
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\USER/.ssh/id_rsa):
```
完成后会在指定的目录生成指定文件名的两个文件：私钥id_rsa和公钥id_rsa.pub，公钥保存在ssh-server中就好，OpenWrt默认使用Dropbear作为SSH工具，其公钥保存在```/etc/dropbear/authorized_keys```中（如果没有这个文件就新建），把公钥的文件内容复制进去就好，或者在LuCI界面的/cgi-bin/luci/admin/system/admin找到SSH-Keys，把公钥放进去

之后只要使用的SSH客户端带有私钥就可以直接使用SSH和SCP

### 使用SCP传输文件
和SSH配套的软件，SCP从字面上理解就是安全的远程cp命令：
```bash
#复制本地多个文件
$ scp foo.txt bar.txt username@remotehost:/path/directory/

#复制远程多个文件
$ scp username@remotehost:/path/directory/\{foo.txt,bar.txt\} .
```

### 使用带有文件传输功能的终端软件
一般需要额外安装一个软件
```bash
opkg install openssh-sftp-server
```
# 附录

### WiFi参数查看APP
[analiti](https://analiti.com/) 是一款Android APP，可以查看WiFi的特性支持情况：[教你用手机查看无线路由器Wi-Fi 7特性，还可查看KVR 和MU-MIMO的支持](https://www.acwifi.net/24613.html)，就2023年的支持MLO和4K QAM的新品铺开之际，这款工具显得非常强大了，使用时如果需要看到MU-MIMO等特性的支持情况，需要能够能连接到analiti的服务器（推测）

### Speedtest
偶尔有网络测速需求，OpenWrt安装Python要注意空间占用
```shell
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -
```
安装完成python之后，如果经常要用的话可以放到用户文件夹中
```shell
wget -O /usr/bin/speedtest https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
chmod +x /usr/bin/speedtest
speedtest
```

### besttrace
电脑上的很好用，可以方便的查看路由的路径（实际地名），路由器上有ARM平台的Linux版本，实测BCM4709的K3是可以运行的

### hotplug脚本
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