---
layout: article
title: 策略路由
mathjax: true
mermaid: true
chart: true
toc: true
mode: immersive
tags : 网络 IPv6 路由器 OpenWrt 多拨
key: strategy
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#ffffff'
  background_image:
    src: assets/images/str.jpg
    gradient: 'linear-gradient(0deg, rgba(0, 0, 0 , .7), rgba(0, 0, 0, .7))'
---

如何能够使用一台路由器让多个人拥有各自独立的网络（降低购买多个路由器的开销）
如何在有多种网络接入的情况下，提高网络的体验
以上的问题都可以归结到策略路由上,通过不同层级间的策略的组合就可以实现不同的效果
<!--more-->

**关键词**：iptables(网络防火墙)，VLAN(网络虚拟化)，Route Load_Balance(路由表负载均衡),IP rule(策略路由)

## 设备和环境

路由器：Xiaomi R3G

系统：OpenWrt 18.06.1

网络：校园网PPPoE，中国联通4G

## 原理

使用Linux网络防火墙iptables对虚拟的出的Interface的数据包做标记，再结合一定的规则对标记的数据包进行路由，也就是通过自定的策略路由来实现，而需要实现这种特定需求下的策略路由就必须**自下而上**的对网络重构，具体的网络结构示意图如下：

![网络结构](https://img.vim-cn.com/b0/c66409bcbbee571fbb9432774a8685ee1d3fcb.png)

**自顶而下的对结构的解读**：

Wireless Master0 和 Wireless Master1 是两个独立的无线网络AP，桥接到不同的LAN，从属于不同的子网，有不同的路由表；
Wireless Client0 是桥接自另外一个接入点的无线网络，直接桥接到WWAN，主要目的是结合有线网络，使用Linux网络防火墙iptables的FwMark功能轮流给数据包打上标记，通过路由表级别的负载均衡来实现对网络的**加速**和数据包的内外网**分流**；
两个子网通过防火墙的空间（Firewall Zone）来划分，再对出入路由器于特定接口的数据包 使FwMark功能打上**标记**，之后再添加一张新的路由表负责被标记的数据包的路由，这样来自两个LAN空间的流量会转发到各自的WAN空间，图示的内外网转发也就是：	lan->wan	lan_2->wan_2
有线PPPoE连接单个网卡只能有一个，因为大多数路由器只有一块网卡，所以在网络接口方面需要用到虚拟网卡得到veth1，veth2...；

而虚拟网卡又坐落在VLAN之上，现在我们使用的大多数路由器都是通过VLAN来为网口进行划分的，通过设备对不同的VLAN ID的接口的标记，就可以划分出多组WAN口和LAN口，通过以上两步就可以实现单/多有线链路的接入以及对LAN口的划分；

综合以上，就是可以实现有线链路与无线桥接的负载均衡以及单个路由的复用。

**注**：

图中的Physical Interface也指Device，是相对于pppoe-wan的Interface而言的
不同的设备和系统以及设置的名称可能不同，这里仅供原理的叙述
以及对当前已有的解决方案的改进：校园网IPv6地址SLAAC分配情况下NAT的改进；


## 代码

### 添加防火墙空间

```shell
echo "add firewall zone and add rules..."

uci add firewall zone 1>/dev/null
uci set firewall.@zone[2]=zone
uci set firewall.@zone[2].name=wan_2
uci set firewall.@zone[2].input=REJECT
uci set firewall.@zone[2].output=ACCEPT
uci set firewall.@zone[2].forward=REJECT
uci set firewall.@zone[2].masq=1
uci set firewall.@zone[2].mtu_fix=1

uci add firewall zone 1>/dev/null
uci set firewall.@zone[3]=zone
uci set firewall.@zone[3].name=lan_2
uci set firewall.@zone[3].input=ACCEPT
uci set firewall.@zone[3].output=ACCEPT
uci set firewall.@zone[3].forward=ACCEPT

uci add firewall forwarding 1>/dev/null
uci set firewall.@forwarding[1]=forwarding
uci set firewall.@forwarding[1].src='lan_2'
uci set firewall.@forwarding[1].dest='wan_2'

uci commit firewall
```

### 设置无线网络

这一步是设置好两个5G频段的无线信号，也可以手动设置，设置完成之后需要开启无线网络

```shell
echo "set and add wireless radio..."

uci set wireless.default-radio1.ssid='OpenWrt_5G'
uci set wireless.default-radio1.mode='ap'
uci set wireless.default-radio1.encryption='psk-mixed'
uci set wireless.default-radio1.key='key'

uci add wireless wifi-iface
uci set wireless.@wifi-iface[3]=wifi-iface
uci set wireless.@wifi-iface[3].device='radio1'
uci set wireless.@wifi-iface[3].ssid='OpenWrt_6'
uci set wireless.@wifi-iface[3].mode='ap'
uci set wireless.@wifi-iface[3].encryption='psk-mixed'
uci set wireless.@wifi-iface[3].key='key'
uci commit wireless
echo "need to call wireless radio1 up..."
```

### 安装必要的软件

先设置好系统时间，连接网络，安装软件和依赖，完成之后删除默认的负载均衡设置

```shell
#!bin/sh

uci set system.system[0].zonename='Asia/Hong Kong'
uci set system.system[0].timezone='HKT-8'
uci commit system

uci set network.wan.proto="pppoe"
uci set network.wan.username="username"
uci set network.wan.password="password"
uci commit network

ubus call network.interface.wan up
ifstatus=`ubus call network.interface.wwan status | grep \"up\" | sed "s/\"up\"://" | sed "s/,//"`
sleep 10
if [ "$telstatus" = ture ];then
  echo "`date` interface.wan was successfully UP,and continum..."
else
  echo "...please check your internet status and try again"
  exit 0
fi

echo "try to install necessary software..."
opkg update
opkg install kmod-macvlan
opkg install kmod-ipt-nat6
opkg install luci-app-mwan3

echo "rm default setting of load_blance..."
sed -i "4,$d" /etc/config/mwan3
uci commit mwan3
```

### 添加接口

添加虚拟网卡，添加开机启动项，桥接无线与LAN口，自订防火墙规则，经过这一步之后，单路由的复用就可以使用了

```shell
ip link add link eth0.2 name veth_wan_2 type macvlan

echo "add start up script..."
sed -i '$d' /etc/rc.local
cat > /etc/rc.local << EOF

ip link add link eth0.2 name veth_wan_2 type macvlan
ip rule add fwmark 0x6 table 300
ip route add default via 10.170.72.254 dev pppoe-wan_2 table 300
exit 0
EOF

echo "add interfaces..."
uci set network.wan_2=interface
uci set network.wan_2.proto='pppoe'
uci set network.wan_2.username="username_2"
uci set network.wan_2.password="password_2"
uci set network.wan_2.defaultroute="0"
uci set network.wan_2.ifname="veth_wan_2"

uci set network.lan_2=interface
uci set network.lan_2.type='bridge'
uci set network.lan_2.proto='static'
uci set network.lan_2.ipaddr='192.168.2.1'
uci set network.lan_2.netmask='255.255.255.0'
uci set network.lan_2.ip6assign='60'
uci commit network

brctl addbr br-lan_2
brctl addif br-lan wlan1-1

uci set dhcp.lan_2=dhcp
uci set dhcp.lan_2.start='100'
uci set dhcp.lan_2.leasetime='12h'
uci set dhcp.lan_2.limit='150'
uci set dhcp.lan_2.interface='lan_2'
uci set dhcp.lan_2.ra_default='1'
uci set dhcp.lan_2.dhcpv6='server'
uci set dhcp.lan_2.ra='server'
uci set dhcp.lan_2.ra_management='1'

uci add_list firewall.@zone[2].network=wan_2
uci add_list firewall.@zone[2].network=wan_2_6
uci add_list firewall.@zone[3].network=lan_2

echo "add firewall mark..."
echo "ip6tables -t nat -I POSTROUTING -s \`uci get network.globals.ula_prefix\` -j MASQUERADE" >> /etc/firewall.user
echo "iptables -t mangle -A PREROUTING -j MARK --set-mark 6 -i pppoe_wan_2" >> /etc/firewall.user
echo "iptables -t mangle -A PREROUTING -j MARK --set-mark 6 -i br-lan_2" >> /etc/firewall.user
uci commit firewall

/etc/init.d/firewall restart

ubus call network.interface.wan_2 up
ubus call network.interface.lan_2 up

ip rule add fwmark 0x6 table 300
ip route add default via 10.170.72.254 dev pppoe-wan_2 table 300

echo "mlan setting is finished,try to enable two wireless radio and enjoy"
```

### 负载均衡设置

这里只是采用了最基本的负载均衡设置，可以自行的添加接口，这里也附上多链路路由负载均衡的配置脚本供参考

对于分流的比例`mwan3.member_$INTERFACE.metric`可以根据具体的网络状况进行调节

```shell
uci set mwan3.wan=interface
uci set mwan3.wan.enabled='1'
uci set mwan3.wan.track_ip='10.170.72.254'
uci set mwan3.wan.track_ip='223.5.5.5'
uci set mwan3.wan.reliability='1'
uci set mwan3.wan.count='1'
uci set mwan3.wan.timeout='2'
uci set mwan3.wan.interval='1'
uci set mwan3.wan.down='1'
uci set mwan3.wan.up='1'

uci set mwan3.wwan=interface
uci set mwan3.wwan.enabled='1'
uci set mwan3.wwan.track_ip='223.5.5.5'
uci set mwan3.wwan.track_ip='223.6.6.6'
uci set mwan3.wwan.reliability='1'
uci set mwan3.wwan.count='1'
uci set mwan3.wwan.timeout='2'
uci set mwan3.wwan.interval='1'
uci set mwan3.wwan.down='1'
uci set mwan3.wwan.up='1'
uci commit mwan3

uci set mwan3.member_wwan=member
uci set mwan3.member_wwan.interface="wwan"
uci set mwan3.member_wwan.metric='1'
uci set mwan3.member_wwan.weight='1'

uci set mwan3.member_wan=member
uci set mwan3.member_wan.interface="wan"
uci set mwan3.member_wan.metric='1'
uci set mwan3.member_wan.weight='1'
uci commit mwan3

uci set mwan3.load_blance=policy
uci set mwan3.load_blance.last_resort='unreachable'
uci add_list mwan3.load_blance.use_member="member_wwan"
uci add_list mwan3.load_blance.use_member="member_wan"
uci commit mwan3

uci set mwan3.default_rule=rule
uci set mwan3.default_rule.use_policy='load_blance'
uci set mwan3.default_rule.proto='all'
uci commit mwan3

/etc/init.d/mwan3 restart
```

## hotplug脚本

因为路由表的修改是会因为接口的连接/断开而变化/失效的，故需要添加热插拔脚本来维持路由表

```shell
#!/bin/sh

[ "$INTERFACE" = wan_2 ] || exit 0
[ "$ACTION" = ifup ] || exit 0
ip route add default via 10.170.72.254 dev pppoe-wan_2 table 300
logger -t dualroute "wan_2 is ip again,table is upgraded"

```
