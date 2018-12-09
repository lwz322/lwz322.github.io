---
layout: article
title: How to OpenWRT
author :
mathjax: true
mermaid: true
chart: true
mathjax_autoNumber: true
tags : 网络  路由器 OpenWRT
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
任何事物都是始于生存，发展于社会秩序，娱乐至死的过程————《只是为了好玩——Linux之父林纳斯自传》

<!--more-->

## 关于OpenWRT
[Wireless Freedom](https://openwrt.org/lib/tpl/openwrt/images/logo.png)
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


或者是论坛，一般都会有详细的刷机教程，如国内的[恩山](https://right.com.cn/forum/portal.php)或者[Koolshare](http://koolshare.cn/portal.php)
>就刷OpenWRT而言，推荐以高通（QAC）和联发科（MTK）或者软路由为主，博通CPU的因为驱动开源的不太好，所以可能会缺少无线功能

就版本的话，主要是稳定版(写这篇文章的时候最新的是18.06.1)和每日构建的版本(Snapshot)，
>后者没有自带LuCI界面，需要自己安装，又因为版本太新，不是所有的软件都有已经编译好的ipk，优势在于可以体验到最新的驱动之类的

## 软件推荐
具体的操作这里就不写了，因为官方的说明文档和教程都很好找
### Aria2
这是一个跨平台的多线程下载软件，主要是支持BT，在路由器性能允许的情况下能够做到全天挂PT，并且可以通过网络共享做一个简易的NAS
>之前有一段时间官方源下载的Aria2是不支持BT的，需要自己动手编译，而1.34之后又自带BT支持了

### luci-app-statistics
强大的统计软件，和其他的包组合收集各种数据，个人的主要作用就是拿来监测网络延迟

### iperf3
跨平台的网络测试工具，主要是拿来测速的，测试一下就知道路由器的极限是什么情况了

### luci-app-nlbwmon
对设备流量统计工具，而且是分IPv4和IPv6的

### luci-wrtbwmon
OpenWRT上少有的网速监测工具，没有官方的Feed，需要自己去下载ipk
https://github.com/Kiougar/luci-wrtbwmon



## 路由器的选购

路由器也是价差很大，推荐路由器的话还是要看需求的

首先自然是百兆和千兆的选择

这个一般都是受到硬件性能的限制，主要也就是NAT能力，所以就会有1300Mbps的无线加上百兆有线的产品了，因为那个1300Mbps是给内网用的，不需要NAT.

再者就是无线的情况

在当前，5G WiFi是很有必要的，2.4G尽管穿墙更好，但是即使是在这种情况下可能依旧比不上同点位的5G WiFi的速度，至于无线信号这一点，一般看内部的硬件，有没有独立的信号放大器之类的，至于天线数量，其实那个影响反而不那么大，具体也是有实例的，也就是第二点，看买家的评价。

以上可能不够具体，但是都是路由器最最根本的硬性性能指标，具体的测试数据需要到比较高端的硬件论坛或者网站去看详细的测试，对应前面的两条就是路由器的数据吞吐量测试和无线信号测试，KoolShare论坛，Chiphell都是不错的

再说可玩性

智能路由？其实也就是锦上添花，实用的功能不多，硬路由性能普遍还是有限

就以上的内网配置IPv6而言，自然是可以刷机的最好，要论最简单的当属可以刷OpenWRT，LEDE，Pandorabox，Padavan之类的固件的路由器，另外

当前就OpenWRT操作起来比较方便的主要有采用MT7260 CPU的百兆路由斐讯的K2，Newifi mini等等，这些因为年代有些久远，可以收二手，性能更强，MT7261千兆路由的K2P（暂时无线功能不可用），Newifi D1，miwifi 3G，以及有线路由ER-X等等，基本上都支持硬件转发，至于其他的没接触过就不多说了

再往上走就是各个品牌的高端路由了，传统的三大厂：Netgear，Linksys，Asus的高端产品价格也比较高，贵在稳定和均衡，不会用的功能很强大，能够安安静静的做个路由器....

更加高级的，一般也说是折腾路由器的人最后都会走到软路由这一步，也就是一台小主机，可玩性就更强了，做下载机，做NAS,尽管没玩过，不过听过来人说，最终都是殊途同归，路由器还是让他安安静静做个路由器吧，毕竟大多数人的要求还是稳定，包括不掉线，无线信号好、速度快、延迟低，带机量足够日常用就行。

另外还有一种情况就是无线覆盖面积问题，这个问题每年回家家里都有人反映，现阶段主流的解决方案按成本从低到高排列：无线桥接，有线桥接，无线AP，以及最近门槛降低的Mesh路由，个人推荐在家里的或者新房布线是最关键的，有线连接的成本和灵活性都要好很多，至于Mesh路由，低端的产品还是一个噱头吧，尤其是为了降低总价而选择了较低的硬件配置，反而是降低了网络的体验，而高端产品价格就很感人了。
