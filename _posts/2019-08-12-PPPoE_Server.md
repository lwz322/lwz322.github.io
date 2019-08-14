---
layout: article
title: OpenWrt搭建PPPoE服务器
author :
mathjax: true
mermaid: true
chart: true
mathjax_autoNumber: true
toc: true
tags : 网络 路由器 OpenWrt
key: pppoe_server
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#ffffff'
  background_image:
    gradient: 'linear-gradient(0deg, rgba(0, 0, 0 , .7), rgba(0, 0, 0, .7))'
    src: assets/background/sunny.jpg
---

OpenWrt有搭建PPPoE服务器的软件，网络上也有教程，但难以满足实际需求：主要是在多WAN接入的情况下，使用PPPoE服务器和策略路由实现用户的分流
<!--more--> 

OpenWrt官方软件源有PPPoE服务端的包rp-pppoe-server，另外还有一个LuCi APP，但是也没有提供什么实质性的功能，所以维持一个服务的功能都要自己写，如果想要做的深入点的话应该找文档看下更好，这里都是按照个人的感觉来写的，经过短时间测试，拿来简单用下还是可以的，如果对稳定性和安全性有较高的要求，建议还是买现成的方案

## RP-PPPoE-Server的特点

- 可以选择**账号和IP关联** 比如说test用户拨号后的IP可以指定为10.170.123.123
- PPPoE Server主要提供上网认证功能，关闭服务器只会影响认证，但已经认证的客户端不会被下线
- 10.0.0.0/8到上级网关的转发方式依然是**NAT**（默认从Zone_wan出去的IPv4数据包就经历了NAT），可以理解为另外一个LAN，Forward到WAN之后再NAT出去
- 实现的过程是下级设备发出PPPoE认证广播->路由器收到广播->在监听的端口上建立pppX->PPPoE认证过程->路由器分配IP

但是在查路由表的过程中发现，这样的PPPoE服务器有一个用于PPPoE的网关，类似于LAN的网关

```bash
root@Y1:~# ip r
default via 192.168.10.1 dev wlan1 proto static src 192.168.10.178
10.170.1.44 dev ppp0 proto kernel scope link src 10.170.1.1
10.170.2.37 dev ppp1 proto kernel scope link src 10.170.1.1
192.168.10.0/24 dev wlan0 proto kernel scope link src 192.168.10.180
192.168.10.0/24 dev wlan1 proto kernel scope link src 192.168.10.178
192.168.13.0/24 dev br-lan proto kernel scope link src 192.168.13.1
root@Y1:~# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.10.1    0.0.0.0         UG    0      0        0 wlan1
10.170.1.44     *               255.255.255.255 UH    0      0        0 ppp0
10.170.2.37     *               255.255.255.255 UH    0      0        0 ppp1
192.168.10.0    *               255.255.255.0   U     0      0        0 wlan0
192.168.10.0    *               255.255.255.0   U     0      0        0 wlan1
192.168.13.0    *               255.255.255.0   U     0      0        0 br-lan
```

大致的想法就有了：给下级设备分配的不同的IP段的IP，再用入站IP的路由规则实现分流

比如走WAN1的账号IP为10.170.1.1/24，走WAN2的账号的IP段为10.170.2.1/24，以此类推

## 实验环境

一台OpenWrt 18.06.4路由器 安装RP-PPPoE-Server

路由器通过无线桥接到家里的双频AP，以两个频段的无线桥接接入，模拟多个WAN接入的场景

两台PC网线直连路由器的LAN口拨号，分配得到两个不同网段的IP和相同网段的IP


## 核心功能

### 开启PPPoE Server密码认证

修改/etc/ppp/pppoe-server-options，主要是require-chap

```
# PPP options for the PPPoE server                                                       
# LIC: GPL                                                                               
require-chap                                                                               
login                                                                                     
lcp-echo-interval 10                                                                     
lcp-echo-failure 2                                                                       
mru 1492                                                                                 
mtu 1492                                                                                
```

修改/etc/ppp/chap-secrets

（用于拨号的用户名 密码）

```
#USERNAME  PROVIDER  PASSWORD  IPADDRESS
test * test *
USER_WAN1_1 * 123456 10.170.1.101
```

下面的两个小节就是介绍下核心步骤，而实现的代码在周边功能的脚本里面

### 开启ppp的Forward

```
iptables -I FORWARD -i ppp0 -j ACCEPT
iptables -I FORWARD -o ppp0 -j ACCEPT
```

写到自定义规则里去，依次添加pppX（X=0,1,2...11）分别对应在线的PPPoE账号

### 添加路由规则

```bash
root@Y1:~# echo '521 wan_1'>>/etc/iproute2/rt_tables
#新建路由表ID与名称映射
root@Y1:~# ip rule add from 10.170.1.1/8 table wan_1 pref 32764  
#新建路由表wan_1，也就是第一个WAN，优先级32764，指定10.170.1.1/24的设备走WAN1
#之后换成全部走WAN1的地址段就好了
root@Y1:~# ip route add default via 192.168.10.1 dev wlan1 table wan_1
#新建路由表wan_1默认路由项
root@Y1:~# ip rule ls
#查看路由表
0:      from all lookup local                                                             
32764:  from 10.170.1.1/24 lookup wan_1
32766:  from all lookup main
32767:  from all lookup default
```

## 周边功能

这部分都是脚本，是维护核心功能正常运行的，按照自己的经验写的，日志和错误处理部分就没怎么写了

### PPPoE Server开机启动

写一个脚本添加到自订的启动项中（如果要求长时间稳定运行的话，需要写守护进程）

```bash
cat >/etc/init.d/pppoe-server << EOF
#!/bin/sh /etc/rc.common
START=99
 
start() {
    /usr/sbin/pppoe-server -k -T 60 -I br-lan -N 32 -L 10.170.1.1 -R 10.170.1.2
}
 
stop() {
    killall pppoe-server
}
EOF
chmod +x /etc/init.d/pppoe-server
/etc/init.d/pppoe-server enable
```
如果在启动项的页面发现还是disable的话就手动启动下，期间我遇到过启动失败的情况，那就添加到Local Startup吧

### 开机添加路由表

因为是固定的本地IP，所以路由表在开机的时候添加就好了，注意修改下Interface的名字（也就是设置的时候的Interface name，注意区分大小写，可以SSH用ifstatus name试下），添加到Local Startup中

```bash
wan_1=WAN1
wan_2=WAN2
wan_3=WAN3
wan_4=WAN4

echo '521 wan_1'>>/etc/iproute2/rt_tables
echo '522 wan_2'>>/etc/iproute2/rt_tables
echo '523 wan_3'>>/etc/iproute2/rt_tables
echo '524 wan_4'>>/etc/iproute2/rt_tables

ip rule add from 10.170.1.1/24 table wan_1 pref 32764
ip rule add from 10.170.2.1/24 table wan_2 pref 32764
ip rule add from 10.170.3.1/24 table wan_3 pref 32764
ip rule add from 10.170.4.1/24 table wan_4 pref 32764

for i in `seq 1 4`
do
  iface=$(eval echo '$wan_'"$i")
	device_wan=$(/sbin/ifstatus $iface | jsonfilter -e '$["l3_device"]')
	gw_wan=$(/sbin/ifstatus $iface | jsonfilter -e '$.route[0].nexthop')
	ip route add default via $gw_wan dev $device_wan table wan_$i
done
logger -t ROUTE "Route Table Init"
exit 0
```


### 自订防火墙规则
添加到Custom Rules中

```
for i in `seq 0 31` 
do
iptables -I FORWARD -i ppp$i -j ACCEPT
iptables -I FORWARD -o ppp$i -j ACCEPT
done
```

### 使用Hotplug脚本来处理WAN的网关变动

这里需要把四个WAN的Interface名称替换下，作用就是WAN口重新连接的时候会自动监测和替换路由表中的网关IP

```bash
#!/bin/sh
wan_1=WAN1
wan_2=WAN2
wan_3=WAN3
wan_4=WAN4

[ "$ACTION" = ifup ] || exit 0

for i in `seq 1 4`
do
	iface=$(eval echo '$wan_'"$i")
	new_gw_wan=$(/sbin/ifstatus $iface | jsonfilter -e '$.route[0].nexthop')
	device_wan=$(/sbin/ifstatus $iface | jsonfilter -e '$["l3_device"]')
	rule_gw_wan=$(ip r show table wan_$i | awk '{print $3}')
	if [ "$new_gw_wan" != "$rule_gw_wan" ]
	then
		ip route del default table wan_$i
		ip route add default via $new_gw_wan dev $device_wan table wan_$i
		logger -t ROUTE "Route Table changed"
	fi
done
```

## 已知的问题

- 一个账号被两台设备登陆的时候，后连接设备因为IP冲突，表现为无Internet连接

  因为给定的是单个的固定的IP，如果换用PPPoE动态IP的话，IP段还不清楚怎么设置

- PPPoE拨号的设备没有显示网关，但是可以从br-lan的ip以及PPP地址登录到网关

  设置个强度高点的密码，或者写防火墙规则

- 插上网线可以连接到br-lan进而不走PPPoE上网

  关掉Zone_lan到Zone_wan的Forward，这样是不会影响br-lan下的局域网的，如果看有两个网络连接不舒服的话就关闭DHCP吧
  
- PPPoE的管理问题

  这个方面OpenWrt除了有个有BUG还没什么用的LuCi APP之外就没有什么了，唯一能够看看的就是防火墙页面可以看到Forward规则走的流量，所以有条件还是上现成的解决方案吧

## 参考

核心功能方面参考的几篇文章设置上大同小异，部分我按照实际情况修改了下

[Openwrt+pppoe-server+radius笔记](http://www.ishenping.com/ArtInfo/436498.html)

[Linux/Openwrt策略路由配置使用](https://www.haiyun.me/tag/openwrtlinux%E7%AD%96%E7%95%A5%E8%B7%AF%E7%94%B1%E8%AE%BE%E7%BD%AE/)

[OpenWRT安装配置PPPoe服务器](https://www.haiyun.me/archives/openwrt-install-pppoe-server.html)
