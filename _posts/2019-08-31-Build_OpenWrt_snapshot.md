---
layout: article
title: 编译OpenWrt Snapshot固件
author :
mathjax: true
mermaid: true
chart: true
mathjax_autoNumber: true
tags: OpenWrt Howto
key: build_Openwrt
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#ffffff'
  background_image:
    src: assets/background/cooling.jpg
    gradient: 'linear-gradient(0deg, rgba(0, 0, 0 , .2), rgba(0, 0, 0, .2))'
---

K3的无线性能貌似不错，现在也有了Snapshot固件可以下载，但是官方对屏幕做的适配不多，网络上的个人编译版本多是按照个人喜好来编译的，屏幕控制，无线信号，插件，各有优缺，那为什么不自己编译一个呢
<!--more-->

## 硬件

斐讯K3 A1版

CPU: BCM4709C Cortex A9 1.4GHz 40nm制程

RAM: 512MB DDR3-1600

flash: 128MB NAND Flash

无线芯片: BMC4366 * 2, 4×4 MU-MIMO

功放: SKY2623L + PA5542

接口: 千兆 wan 口 + 千兆 lan 口 * 3 + USB 3.0

硬件上跟华硕的AC88U差不多，信号在acwifi的测试中相当强悍，做AP的效果也不错，无线方面的缺点是不支持160Mhz的频宽，无法适配市场上现有的一批廉价的2x2的1.7G网卡，以及开源驱动太弱

硬件方面“漏油”问题已经通过更换铜片+硅脂的解决了，另外就是由于芯片的制程太过落后，发热感人，容易“烧坏”硬件

很期待一台高性能的，无线强劲的OpenWrt路由器（价格要可以接受才行）

## K3的OpenWrt支持

我翻了下OpenWrt的Table of Hardware，发现已经有了Snapshot的支持，并且也没有说明硬件不可用之类的，一般来说，也就是下个stable release就支持了，距离18.04也有很长一段时间了，19.07应该发布在即，因为其他的OpenWrt设备都在履行自己的义务，拿这个体验下新系统也不错，首先我是下载了编译好的snapshot固件，感受下19.07版本的新特性：

1. LuCi界面优化

首页的风格小改，截图底部的[排版错乱](https://www.jarviswang.me/?p=1119)终于被修复了，另外就是改动了选项卡的风格，减小了选项卡的面积
![](https://img.vim-cn.com/23/dcdce4a5ca18ca785422d39343152ac28bc6e3.png)

最重要的改动不易察觉，就是优化了移动端的页面布局，解决了之前竖屏修改设置极为不便的问题
![](https://img.vim-cn.com/42/92d9dda00d3962bc209b5d3312dfaeb24a2a2e.png)

2. 无线方面

现在可以在路由器把连接的客户端断开了，然后在选项卡里都有了802.11r的选项， 另外的惊喜就是发现5G一栏居然有160Mhz的选项!! 然而Intel 9260AC的网卡没有在身边... 

![](https://img.vim-cn.com/a6/8c6d33d90e84c59457490898fac81c0eed0922.png)

看完了一些新特性来说下目前的Snapshot固件存在的问题

1. K3的屏幕信息显示不支持而且只能常亮，据说添加了[补丁](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=419328)，应该还是要自行编译安装k3screenctrl的

2. 开源的驱动的信号，可以说几乎没有，即使是在路由器旁边都只有10Mbps的速度

[社区](https://www.right.com.cn/forum/thread-466672-1-1.html)有人已经针对snapshot做了修改和编译，解决了以上两个问题，作为路由器基本上已经可以用了，但是有以下遗憾：

1. 固件基于uclibc的c库编译，官方源软件部分不能使用，我尝试的Opkg安装的大部分软件不能用

2. 自带的软件又太多，K3的发热本来就大，太多软件带来的负担更大

3. 作者的工作相当的出色，但是后续没有更新以及没有公布细节，想自己定制变得很困难（只想要个原生、纯净的版本）

最后还是要自己动手，但是完全可以参考上面的固件来编译，然后我就打开了Github，找K3的屏幕和无线方面可用的资源

## 框架

主要解决的还是屏幕和无线信号两个问题，因为参考的东西比较杂（东拼西凑），这里做个大致的描述：

### 屏幕

屏幕包括几个部分：屏幕控制的源码，luci-app设置界面，屏幕界面信息更新脚本以及综合以上的编译设置文件

在 [zxlhhyccc/Hill-98-k3screenctrl](https://github.com/zxlhhyccc/Hill-98-k3screenctrl) 已经给K3屏幕开启了7屏的基础上，使用 [K3 openwrt18.06.02](https://www.right.com.cn/forum/thread-466672-1-1.html) 固件中的```/lib/k3screenctrl/```下的sh文件做了替换

搭配的 luci-app 是根据固件的LuCi文件修改的 [lwz322/luci-app-k3screenctrl](https://github.com/lwz322/luci-app-k3screenctrl)

最后使用修改自 [lean/lede](https://github.com/lean/lede) 中的编译文件 [lwz322/k3screenctrl_build](https://github.com/lwz322/k3screenctrl_build) 编译

### 屏幕界面的情况

第一屏：升级界面

第二屏：型号，温度，MAC，软件版本

第三屏：接口

第四屏：网速以及2.4G和5G WiFi的接入客户端数量

第五屏：天气，时间

第六屏：WiFi信息：SSID和密码（可选隐藏）

第七屏：已接入终端和网速（IPv4转发）

### 无线固件部分

直接使用了[社区](https://www.right.com.cn/forum/thread-466672-1-1.html)的无线固件，我的7260AC网卡在近距离（同一个房间内）使用iperf3测试下载和上传速度，5G的实际传输速度300Mbp左右，5M，相隔两堵墙的情况下200Mbps左右，再远一点100Mbps，三堵墙就GG了，在我使用的OpenWrt的路由器中是最好的（一般隔一墙就GG），据说K3的无线功率相当的高，发热也很大，连接着两台设备的情况下，温度可以到70以上（还是再改了散热的情况下）

其他的固件可以参考[/Hill-98/phicommk3-firmware](https://github.com/Hill-98/phicommk3-firmware)做替换，固件在OpenWrt的```/lib/firmware/brcm/ ```目录下

## 编译经过

这里对网络环境有一定的需求，一般都是推荐在国外的VPS上编译，如果在本地编译需要开代理，安装和设置docker和Git就不说了

### Windows下使用Dockerfile构建编译环境

主要是避免环境之间的互相影响，这里也不推荐使用WSL，原因是WSL的文件系统依然是Windows文件系统，需要做些额外的工作，详情见文末的参考部分

```powershell
git clone https://github.com/lwz322/OpenWrt_build_Docker.git
cd OpenWrt_build_Docker
docker build -t openwt_builder .
```

这里因为使用的是透明代理，软件源选的中科大的，根据情况可以自行修改
```
sed  -i 's!http://mirrors.ustc.edu.cn!URL!g' DOCKERFILE
```

image构建完成之后运行

```powershell
docker run --rm -it openwrt_builder
```
> 这里--rm参数，在容器终止运行后自动删除容器文件，要不要添加这个个人，-it是两个参数：-i和-t。前者表示打开并保持stdout，后者表示分配一个终端（pseudo-tty）

进入容器后clone源码
```bash
git clone https://github.com/openwrt/openwrt.git
```

再git clone屏幕相关文件到package/k3目录下，回到编译目录更新feeds以及把软件包注册到编译系统中
```bash
mkdir openwrt/package/k3
cd openwrt/package/k3
git clone https://github.com/lwz322/luci-app-k3screenctrl.git
git clone https://github.com/lwz322/k3screenctrl_build.git
cd ~/openwrt

./scripts/feeds update -a  && ./scripts/feeds install -a
```
之后打开编译选项：
```bash
make menuconfig
```

- Target 选 Broadcom BCM47xx/53xx (ARM)
- Target Profile 选 PHICOMM K3
- Utilities 确保 k3screenctrl 选中，倚赖会自动选上
- LuCi->Application中的 luci-app-k3screenctrl
- snapshot默认没有的web管理界面 LuCi->Collection ->luci

为了避免可能出现的软件包倚赖和网络问题，先检查倚赖和下载源码，如果遇到问题可能需要改为```-j 1```查看具体原因，这一步需要的时间比较长（取决于网速和添加的包的数量，20Mbps大概十分钟），可以开个tmux去修改下默认的设置之类的

```bash
make defconfig
make -j 10 download
```

选择完成之后一路Exit，也可以保存配置文件，最后编译

```bash
make -j 1 V=99
```
-j 1 后面的1是线程数，第一次编译推荐用单线程，需要的时间可能

使用8700K@4.3G，Docker分配10 CPUs+4G RAM，使用十线程编译的时候需要二十分钟左右，最后生成的固件和ipk在/bin目录下，回到Windows目录下，从容器中拷贝文件就好
```
mkdir ./bin
docker cp Container_ID:/openwrt/bin/ ./bin
```
## 其他设置的修改

#### 修改LAN IP为 192.168.2.1
修改这个文件 package/base-files/files/bin/config_generate 里的 192.168.1.1 为 192.168.2.1
```bash
sed -i 's/192.168.1.1/192.168.2.1/g' package/base-files/files/bin/config_generate
```
### 默认WIFI设置

由于OpenWrt默认关闭WIFI的，如何一编译好就默认开启WIFI呢？

我们只需要修改配置文件就OK了。方法如下：

打开package/kernel/mac80211/files/lib/wifi/mac80211.sh文件，大约在文件最后有如下代码

```
config wifi-device  radio$devidx
        option type     mac80211
        option channel  ${channel}
        option hwmode   11${mode_band}
$dev_id
$ht_capab
        # REMOVE THIS LINE TO ENABLE WIFI:
        #这里↑说的很清楚了，注释掉下面一行就行了
        option disabled 1

config wifi-iface
        option device   radio$devidx
        option network  lan
        option mode     ap
        option ssid     OpenWrt    #这里修改默认SSID SSID中不允许有空格
        option encryption none
```
## 刷机

鉴于TFTP的刷机太过麻烦，参考[自编译说明](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=419328)，可以使用：
openwrt/lede中，强刷固件教程（可有效避免web页面刷机的各种问题）：
1. 上传固件到路由器，/tmp/k3.trx
2. ssh，执行命令
```bash
mtd -r write /tmp/k3.trx firmware
```

说明中还给了一些技巧：
- 修改固件的Makefile使其只编译K3的固件
```bash
sed -i 's|^TARGET_|# TARGET_|g; s|# TARGET_DEVICES += phicomm-k3|TARGET_DEVICES += phicomm-k3|' target/linux/bcm53xx/image/Makefile
```
- 在VPS打包make download之后的文件放到高性能的机器编译以节约时间

如果有需要，固件部分还是手动替换下吧

## OpenWrt下K3的性能

个人的测试以及使用感受

### 无线

当前想要作为无线路由使用的话是肯定要换无线固件的，尽管标称AC3150，参考[简说各种wifi无线协议的传输速率](https://www.acwifi.net/318.html)，K3是 4X4 MIMO + 80MHz + 1020-QAM = 2100Mbps ，结合现在的无线网卡市场（廉价的2X2 160Mhz网卡普及快），信号和速率方面其实已经难以和现在中端以上的路由器（200+）拉开差距了，相比其他的OpenWrt路由，我的结论就是K3只有信号强度有较大的实用价值，剩下的没有绝对的优势（如果不考虑外观的话）

### CPU运算性能

就以专门为ARM设计的ChaCha20算法测试，BCM4709C@1.4G的2C2T加解密的速度在60Mbp左右CPU就已经满载，温度70左右，相比现在主流的MT7621@880MHz的2C4T甚至处于下风，大概是输在了制程上，另外因为K3有块屏幕需要不断刷新，偶尔CPU的占用高，这对网络是很不利的

### Snapshot版本

使用体验取决于编译的情况，因为大部分的软件是没有预编译的IPK可以下载的，我一开始用的master分支的Snapshot版本编译，部分软件都无法正常编译，之后换了19.07的Snapshot版本好一点点，暂时可以用了，使用19.07分支的话我是先Clone仓库，再切换的分支（貌似直接Clone 19.07不行）

## 参考

[OpenWrt 编译失败的原因及解决方案](https://p3terx.com/archives/reasons-and-solutions-for-openwrt-compilation-failure-3.html)

> 因为Windows文件系统与Linux不同，是默认不区分大小写的，推荐不挂载，如果挂载的话
>
> docker run --rm -it -v $(pwd)/data:/root openwt_builder
>
> 因为powershell对$(pwd)/data不支持，需要替换成绝对路径(echo "$(pwd)\data")


> 因为OpenWrt只能构建在区分大小写的文件系统上（Build dependency: OpenWrt can only be built on a case-sensitive filesystem），所以如果Docker挂载了Windows的目录的话需要设置下：
>
>fsutil.exe file setCaseSensitiveInfo <path> enable
>
> 但是设置完成之后git clone又不可用了（尴尬，所以一般还是用docker cp拷贝文件出来方便些）