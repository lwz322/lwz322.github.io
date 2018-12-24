---
layout: article
title: How to OpenWRT
author :
mathjax: true
mermaid: true
chart: true
mathjax_autoNumber: true
tags : 网络  路由器 OpenWRT Howto
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

## 关于OpenWRT
![Wireless Freedom](https://openwrt.org/lib/tpl/openwrt/images/logo.png)
从OpenWRT Logo下面的那个Wireless Freedom讲起吧，追寻自由的里程碑
- 让无线设备用上IPv6网络，畅通的使用Google，Youtube，IPv6 PT
- 通过IPv6免流量高速上网
- 用Aria2挂PT，攒下了可以用几年的上传量
- 在学校限速1Mbps的情况下把网速多拨叠加到6Mbps
- 逐步改进又达到了40Mbps,60Mbps,有线无线双LAN叠加到达150Mbps
- 最后趁舍友国庆出游，用新的负载均衡方法以及HWNAT把汇聚全宿舍端口达到400Mps
- 接触OpenWRT开发的一些方法，开始根据自己的需求做策略路由

## 如何开始
建议选有官方固件支持，软件支持的路由器（也就是不太建议刷仅有民间固件的那种了，不开源感觉不安全），具体可以参考官方支持的[Hardware Table](https://openwrt.org/toh/start)，进入某一款路由器的详情界面就可以看到支持的情况


或者是论坛，一般都会有详细的刷机教程，如国内的[恩山](https://right.com.cn/forum/portal.php),[Koolshare](http://koolshare.cn/portal.php)
>就刷OpenWRT而言，推荐以高通（QAC）和联发科（MTK）或者软路由为主，博通CPU的因为驱动开源的不太好，所以可能会缺少无线功能

就版本的话，主要是稳定版(写这篇文章的时候最新的是18.06.1)和每日构建的版本(Snapshot)
>后者没有自带LuCI界面，需要自己安装，又因为版本太新，不是所有的软件都有已经编译好的ipk，优势在于可以体验到最新的驱动之类的

## 软件推荐
具体的操作这里就不写了，以后有时间写到博客的WiKi一栏，大部分软件使用也比较简单，官方的说明文档和教程都很好找
### Aria2
这是一个跨平台的多线程下载软件，主要是支持BT，在路由器性能允许的情况下能够做到全天挂PT，并且可以通过网络共享做一个简易的NAS
>之前有一段时间官方源下载的Aria2是不支持BT的，需要自己动手编译，而1.34之后又自带BT支持了

Aria2本身只是一个命令行下载软件，而OpenWRT提供了以个LuCI的配置APP(只能拿来做些设置)luci-app-aria2，所以如果不想用命令行的话就需要一个前端界面，推荐一个[AiraNG](https://github.com/mayswind/AriaNg)

### luci-app-statistics
强大的统计软件，和其他的包组合收集各种数据，个人的主要作用就是拿来监测网络延迟，可以提一下的就是能够结合防火墙数据收集实现复杂的流量监控，下面附上官方的WiKi
[luci-app-statistics](https://oldwiki.archive.openwrt.org/doc/howto/luci_app_statistics)

### iperf3
跨平台的网络测试工具，主要是拿来测速的，测试一下就知道路由器的性能是什么情况了，比如说5G-5G的传输速度，LAN-5G的传输速度

[iperf3](https://iperf.fr/iperf-download.php)，这里面还有个软件值得推荐[HE.NET-Network Tools](http://networktools.he.net/)

### luci-app-nlbwmon
对设备流量统计工具，而且是分IPv4和IPv6的，在流量统计软件里算是很美观的了

### luci-wrtbwmon
OpenWRT上少有的网速监测工具，没有官方的Feed，需要自己去[Github](https://github.com/Kiougar/luci-wrtbwmon)上面下载ipk
>为什么没有人做分设备的实时网速监测的可视化，就像OpenWRT的Realtime Graph一样，之后学了JavaScript我觉得我可以自己写一个

## 实用的功能

### 交换机 Switch
这个的用途就是当路由器做路由，占用掉了墙壁内嵌的网口，但是这个时候又有设备需要直接拨号，从前的话可能就需要使用交换机了，但是考虑到现在的路由器都是通过VLAN来划分网口的，而OpenWRT对此是可以自定义的，所以只需要改下网口的VLAN ID就好，因为设备之间有差别，所以这里建议参考[OpenWRT Guide](https://openwrt.org/docs/guide-user/network/vlan/switch_configuration)，然后根据自己的需求做设置

### 定时任务Crontab
LuCI的System->Scheduled Tasks中就是了，和Linux中的Crontab差不多，所以查下就好，算是简单实用的东西了，用来做个定时重启、运行某个脚本都是可以的,下面这个就是定时断开和连接一个PPPoE拨号
```shell
6 * * * * /sbin/ifdown tel
23 40 * * * /sbin/ifup tel

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

## 路由器的选购

路由器也是价差很大，推荐路由器的话还是要看需求的，因为时效性也比较强，所以只能列出指标了

### 百兆 or 千兆

这个一般都是受到硬件性能的限制，主要也就是NAT能力，所以就会有1300Mbps的无线加上百兆有线的产品了，因为那个1300Mbps是给内网用的，不需要NAT.

### 无线

在当前，5G WiFi是很有必要的，2.4G尽管穿墙更好，但是即使是在这种情况下可能依旧比不上同点位的5G WiFi的速度，至于无线信号这一点，一般看内部的硬件，有没有独立的信号放大器之类的，至于天线数量，其实那个影响反而不那么大，具体也是有实例的，也就是第二点，看买家的评价。

以上可能不够具体，但是都是路由器最最根本的硬性性能指标，具体的测试数据需要到比较高端的硬件论坛或者网站去看详细的测试，对应前面的两条就是路由器的数据吞吐量测试和无线信号测试，KoolShare论坛，Chiphell都是不错的（只是觉得部分文章从内容上来说是不错的）

### 可玩性

智能路由？其实也就是锦上添花，就以官方的固件而言，添加了一些定制好的功能，但是依然不够实用，而对于有更高需求的人，向往的还是可以自由定制的路由，况且硬路由性能普遍还是有限

就内网配置IPv6而言，自然是可以刷机的最好，要论最简单的当属可以刷OpenWRT，LEDE，Pandorabox，Padavan之类的固件的路由器

当前就OpenWRT操作起来比较方便的主要有采用MT7260 CPU的百兆路由斐讯的K2，Newifi mini等等，这些因为年代有些久远，可以收二手；性能更强的，MT7261千兆路由的K2P（暂时无线功能不可用），Newifi D1，miwifi 3G，以及有线路由ER-X等等，基本上都支持硬件转发，至于其他的没接触过就不多说了

### 更高级的路由器

再往上走就是各个品牌的高端路由了，传统的三大厂：Netgear，Linksys，Asus的高端产品价格也比较高，贵在稳定和均衡，不会用的功能很强大，能够安安静静的做个路由器....

更加高级的，一般也说是折腾路由器的人最后都会走到软路由这一步，也就是一台小主机，可玩性就更强了，做下载机，做NAS，尽管没玩过，不过听过来人说，最终都是殊途同归，路由器还是让他安安静静做个路由器吧，毕竟大多数人的要求还是稳定，包括不掉线，无线信号好、速度快、延迟低，带机量足够日常用就行。

另外一个需要考虑的就是无线覆盖问题，这个问题每年回家家里都有人反映，现阶段主流的解决方案按成本从低到高排列：无线桥接，有线桥接，无线AP，以及最近门槛降低的Mesh路由，个人推荐在家里的或者新房布线是最关键的，有线连接的成本和灵活性都要好很多，至于Mesh路由，低端的产品还是一个噱头吧，尤其是为了降低总价而选择了较低的硬件配置，反而是降低了网络的体验，而高端产品价格就很感人了。

## 最后就是一点感想
读了几年的书，偶尔会遇到一些书写得深入浅出，不同的时间节点读都有不同的感觉，写博客的目的，有这么一条，就是写一些既能够简单直观的呈现，又可以传达足够的信息量，引人探索和思考，当然了这对作者也是要相当的要求的，所以我的博客搭建的也很晚，现在也只能勉强解释清楚一些东西。

这里面最后呈现在文章中的也比较有限，而自己当初探索，学习，实践的过程依旧值得回味，同时也就从这里开始接触到了Linux和网络的实例，之后遇到了一些问题，根据原理和自己的理解也都解决的差不多了，为了节约配置的时间，也尝试写了一些Shell脚本，尽管有些成果时效性有限，不过也为一些特殊情况提供了一种解决问题的思路，暂时就这样吧，想法太多实现不完，大概之后学院开计算机网络的课程再玩吧，感觉这是学了这些东西可以用个十几年...

个人由于曾经饱受低端路由掉线的烦恼，也花了一些时间看了些文章，转了几个论坛的确很长见识，也踩过不少的坑，生活中处处都有这样那样的坑，能够认清一部分并加以规避也是一种能力吧，而把这些东西传达给身边的人，降低决策成本也算是发挥了自己在这件事情上面花的时间的剩余价值。
