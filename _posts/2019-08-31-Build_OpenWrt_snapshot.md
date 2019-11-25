---
layout: article
title: 编译OpenWrt固件
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
    src: assets/background/k3_background.png
    gradient: 'linear-gradient(0deg, rgba(0, 0, 0 , .7), rgba(0, 0, 0, .7))'
---

K3的无线性能貌似不错，现在也有了Snapshot固件可以下载，但是官方对屏幕做的适配不多，网络上的个人编译版本多是按照个人喜好来编译的，屏幕控制，无线信号，插件，各有优缺，那为什么不自己编译一个呢；本文介绍了19.07版本的新特性，并在windows下使用Docker编译了K3的编译

<!--more-->

# 前言
我翻了下OpenWrt的Table of Hardware，发现K3已经有了Snapshot的支持，并且也没有说明硬件不可用之类的，一般来说，下个stable release就完全可以拿来用了；因为其他的OpenWrt设备都在履行自己的义务，拿这个体验下新系统也不错，首先我是下载了编译好的snapshot固件，感受下19.07版本的新特性

本文的编译工作基于OpenWrt官方的19.07分支的源码，参考了部分已有的K3固件，加入了屏幕组件以及闭源无线驱动，因为GPL协议，所以不提供固件，但是使用了个人修改的屏幕组件的有恩山论坛的[K3 openwrt固件](https://www.right.com.cn/forum/thread-1275902-1-1.html)，基于的lean的lede，感兴趣可以试试

# OpenWrt 19.07

2019年11月，OpenWrt 19.07 rc1发布了，明显的变化：
- 大幅更新了LuCi界面
 - 扁平 + 卡片式二级设置，更多的标签
 - 移动端网页自适应布局
 - 通过客户端渲染提高响应速度
- 默认提供更完整的功能体验，包括但不限于
 - 安装软件方面支持网页查看依赖树以及上传ipk安装
 - 8M以上的设备默认使用wpad-basic，支持802.11r无线漫游
- 替换了网页的Favcion，内核的基线全部到4.14
- 其他的就是一些Bug修复，包括讨论的比较多的openssl，DHCPv6

其他的可见于[Release Notes](https://openwrt.org/releases/19.07/notes-19.07.0-rc1)，后文也提供了几张截图作为参考

另外还有18.06.5，修复了之前Web界面上status界面错位的问题，整体风格上部分和19.07同步

## LuCi界面优化

首页的布局小改，截图底部的[排版错乱](https://www.jarviswang.me/?p=1119)终于被修复了，另外就是改动了选项卡的样式，在页面内呈现更多的信息
![](https://img.vim-cn.com/23/de476c3304bf0ff2047dd513fe03cc16b67440.png)

另外改动的还有页面便签的设定，粗看没什么意义

我觉得是最重要的改动————优化了移动端的页面布局，解决了之前竖屏修改设置极为不便的问题
![](https://img.vim-cn.com/42/92d9dda00d3962bc209b5d3312dfaeb24a2a2e.png)

其他的变动如Administration界面从之前的一个长网页改成了多标签网页，Firewall的Traffic Rules精简了Open Ports等栏目，改为Add的悬浮窗

从LEDE 17到OpenWrt 18，引入更多的色彩，突出重点，界面也变得更活泼，OpenWrt 19界面升级的重点大概是在布局上面，让内容变得更易读

## 无线方面

现在可以在路由器把连接的客户端断开了

![](https://img.vim-cn.com/c8/c941036270873b4be3a21e16b59aa2f9622fcd.png)

选项卡里默认都有了802.11r的选项，漫游和Mesh大概也是19.07的重要特性，另外的惊喜就是发现5G一栏居然有160Mhz的选项!! 使用Intel 9260AC的网卡实测，默认的无线固件下只有300Mbps的协商速率

![](https://img.vim-cn.com/a6/8c6d33d90e84c59457490898fac81c0eed0922.png)

## 其他

前段时间编译K2P的19.07的Snapshot时发现有MT7615e的驱动，K2P貌似只有一颗MT7615e，编译勾选之后也就可以用用2.4G频段，适当设置下速度还是有80Mbps的，5G有但是不能用

另外小米路由3G（R3G）从18.06升级19.07的时候发现无论使用mtd还是sysupgrade都不行，前者提示can't open device，后者则是format not support，即使是升级到可能的国度版本18.06.5还是不行，后面我查了下mt7621.mk的commit log，发现R3G自一次[commit](https://github.com/openwrt/openwrt/commit/7f00123d63584e8d7da717c89fd1df610a161983)默认不再编译tar格式的sysupgrade，故这里需要先回滚mk文件中定义r3g固件格式的几行才可以编译得到tar文件做sysupgrade

对19.07，暂时发现了默认的PPPoE貌似没有设置掉线检测，也就是实际上掉线之后不会重拨

# K3相关
## 硬件

斐讯K3 A1版
- CPU: BCM4709C Cortex A9 1.4GHz 40nm制程
- RAM: 512MB DDR3-1600
- flash: 128MB NAND Flash
- 无线芯片: BMC4366 * 2, 4×4 MU-MIMO
- 功放: SKY2623L + PA5542
- 接口: 千兆 wan 口 + 千兆 lan 口 * 3 + USB 3.0

硬件上跟华硕的AC88U差不多，信号在acwifi的测试中相当强悍，做AP的效果不错，遗憾是K3没有160MHz的频宽，无法适配市场上现有的一批廉价的2x2的1.7G网卡；硬件上明显的缺点是芯片的制程太过落后，发热感人

一直很期待一台高性能的，无线强劲的OpenWrt路由器（价格要可以接受才行），所幸“漏油”问题已经通过更换铜片+硅脂的解决了

## OpenWrt下的问题

看完了一些新特性来说下目前的Snapshot固件存在的问题

1. K3的屏幕信息显示不支持而且只能常亮，据说添加了[补丁](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=419328)，应该还是要自行编译安装k3screenctrl的

2. 开源的驱动的信号，可以说几乎没有，即使是在路由器旁边都只有10Mbps的速度，但是在调节频段之后勉强可用，就是协商速度不高，LuCi的无线界面显示的频宽仅有20MHz，考虑到稳定性一般还是不适合日常使用

[社区](https://www.right.com.cn/forum/thread-466672-1-1.html)有人已经针对snapshot做了修改和编译，解决了以上两个问题，作为路由器基本上已经可以用了，但是有以下遗憾：

1. 固件基于uclibc的c库编译，官方源软件部分不能使用，我尝试的opkg安装的大部分软件不能用

2. 自带的软件又太多，K3的发热本来就大，太多软件带来的负担更大

3. 作者的工作相当的出色，但是后续没有更新以及没有开源公布细节，想自己定制变得很困难（只想要个原生、纯净的版本）

最后还是要自己动手，但是完全可以参考上面的固件来编译，然后我就打开了Github，找K3的屏幕和无线方面可用的资源

## 解决方案

主要解决的还是屏幕和无线信号两个问题，因为参考的东西比较杂（东拼西凑），这里做个大致的描述，相关的代码都在个人的Github仓库下

### 屏幕组件

屏幕包括几个部分：屏幕控制的源码，luci-app设置界面，屏幕界面信息更新脚本以及综合以上的编译设置文件

在 [zxlhhyccc/Hill-98-k3screenctrl](https://github.com/zxlhhyccc/Hill-98-k3screenctrl) 已经给K3屏幕开启了7屏的基础上，使用 [K3 openwrt18.06.02](https://www.right.com.cn/forum/thread-466672-1-1.html) 固件中的```/lib/k3screenctrl/```下的sh文件做了替换

搭配的 luci-app 是根据固件的LuCi文件修改的 [lwz322/luci-app-k3screenctrl](https://github.com/lwz322/luci-app-k3screenctrl)

最后使用修改自 [lean/lede](https://github.com/lean/lede) 中的编译文件 [lwz322/k3screenctrl_build](https://github.com/lwz322/k3screenctrl_build) 编译

具体的界面：
- 第一屏：升级界面
- 第二屏：型号，温度，MAC，软件版本
- 第三屏：接口
- 第四屏：网速以及2.4G和5G WiFi的接入客户端数量
- 第五屏：天气，时间
- 第六屏：WiFi信息：SSID和密码（可选隐藏）
- 第七屏：已接入终端和网速

### 无线固件

直接使用了[社区](https://www.right.com.cn/forum/thread-466672-1-1.html)的无线固件，貌似和lean的仓库中的[k3-brcmfmac4366c-firmware](https://github.com/coolsnowwolf/lede/tree/master/package/lean/k3-brcmfmac4366c-firmware)一样，

其他的固件可以参考[/Hill-98/phicommk3-firmware](https://github.com/Hill-98/phicommk3-firmware)做替换，固件在OpenWrt的```/lib/firmware/brcm/ ```目录下

# 编译固件

这里对网络环境有一定的需求，一般都是推荐在国外的VPS上编译，如果在本地编译需要开代理，安装Docker和Git就不说了，如果觉得Docker和SSH的命令行对修改源码体验不好，可以使用VSCode加上Remote-Docker/SSH插件，再安装一个Terminal插件就很舒服了

## 准备工作

介绍下硬件环境的要求：

官方给出的[要求](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)是至少10-15G的存储空间和2G的内存（X86版本需要4G）

我分配了4G内存+**60G磁盘**给Docker，然而当我运行的时候发现空间剩下的只有20G
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        59G   36G   20G  65% /etc/hosts
```
实际单个固件编译的时候内存占用1G左右，编译完成的时候磁盘空间几乎被占满，以至于开另外一个目录编译报错，每个人的情况可能不同，但是运行的时候建议留足空间的余量

### Windows-Dockerfile构建环境

主要是避免环境之间的互相影响，这里也不推荐使用WSL，原因是WSL的文件系统依然是Windows文件系统，需要做些额外的工作，详情见文末的参考部分

这里还可能存在的问题就是OpenWrt官方建议不要使用root用户Git和编译固件，但是实测没有问题，保险起见也可以使用[p3terx/openwrt-build-env](https://p3terx.com/archives/build-openwrt-with-docker.html)提供的Docker，这里就从简了

```powershell
git clone https://github.com/lwz322/OpenWrt_build_Docker.git
cd OpenWrt_build_Docker
docker build -t openwrt_builder .
```

这里因为使用的是透明代理，软件源选的中科大，根据情况可以自行修改
```
sed  -i 's!http://mirrors.ustc.edu.cn!URL!g' DOCKERFILE
```

image构建完成之后运行
```powershell
docker run --rm -it openwrt_builder
```
> 这里--rm参数，在容器终止运行后自动删除容器文件，要不要添加这个个人，-it是两个参数：-i和-t。前者表示打开并保持stdout，后者表示分配一个终端（pseudo-tty）

### 下载OpenWrt源码

进入容器后clone源码，默认是master分支
```bash
git clone https://github.com/openwrt/openwrt.git
```
如果要切换到其他的分支的话，在openwrt目录中使用git checkout，例如切换到相对稳定的19.07分支
```bash
git checkout openwrt-19.07
``` 
对于18.06.4这种小分支，可以在release下载源码或者直接根据18.06.4的commit hash来切换

### 下载软件包源码

git clone屏幕相关文件到package/k3目录下，回到编译目录更新feeds以及把软件包注册到编译系统中
```bash
mkdir openwrt/package/k3
cd openwrt/package/k3
git clone https://github.com/lwz322/luci-app-k3screenctrl.git
git clone https://github.com/lwz322/k3screenctrl_build.git
cd ~/openwrt

./scripts/feeds update -a  && ./scripts/feeds install -a
```

如果有其他的包要添加到编译目录的话，把源码或者编译目录clone到package的一个文件夹下即可，如果是从其他人的OpenWrt源码中搬来的话要注意下是否有特殊的倚赖

## 编译与刷机

先介绍一个技巧：修改固件的Makefile使其只编译K3的固件，大大减少编译需要的时间
```bash
sed -i 's|^TARGET_|# TARGET_|g; s|# TARGET_DEVICES += phicomm-k3|TARGET_DEVICES += phicomm-k3|' target/linux/bcm53xx/image/Makefile
```
### 编译选项

之后打开编译选项：
```bash
make menuconfig
```

- Target 选 Broadcom BCM47xx/53xx (ARM)
- Target Profile 选 PHICOMM K3 （如果用了上述技巧的话就只有K3可选）
- Utilities 确保 k3screenctrl 选中，倚赖会自动选上
- LuCi->Application 中的 luci-app-k3screenctrl
- snapshot默认没有的web管理界面 LuCi -> Collection -> luci

其他的软件包自选，选择完成之后一路Exit，保存配置文件，为了避免可能出现的软件包倚赖和网络问题，先检查倚赖和下载选择编译的包的源码，建议最开始选最基本的包编译，成功之后的编译可以用到之前编译的中间文件

```bash
make defconfig
make download
```
这一步需要的时间比较长（取决于网速和添加的包的数量，20Mbps大概十分钟），期间可以用tmux分屏去[修改默认的设置](https://lwz322.github.io/2019/08/31/Build_OpenWrt_snapshot.html#%E5%85%B6%E4%BB%96%E8%AE%BE%E7%BD%AE%E7%9A%84%E4%BF%AE%E6%94%B9)之类的

这一步之前的所有步骤也可以在VPS完成，打包编译目录放到本地高性能的机器编译以节约时间

### Make

```bash
make -j 1 V=99
```
make的帮助中写到：

-j [N], --jobs[=N]          Allow N jobs at once; infinite jobs with no arg.

后面的N指的是可并行的任务数，第一次编译推荐用```-j1```，需要的时间可能比较长，但是出错时可以查看具体原因，```V=99``` 生成固件并显示每一步及正确性,另外还有```V=s```:生成固件忽略不影响固件主功能的错误

我是先用```-j10```做最简编译，之后再添加常用的包做二次编译，如果不需要生成完整的固件的话，单独编译包也是可以的

### 拷贝文件到Windows

使用8700K@4.3G，Docker分配10 CPUs + 4G RAM + 60G HDD，使用十线程编译的时候需要二十分钟左右，磁盘占用80%以上，最后生成的固件和ipk在/bin目录下，回到Windows目录下，从容器中拷贝文件就好（VSCode Remote右键Download即可）
```shell
mkdir ./bin
docker cp Container_ID:/openwrt/bin/ .
```

### 刷机

我用OpenWrt...鉴于TFTP的刷机太麻烦，参考[自编译说明](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=419328)：

openwrt/lede中，强刷固件教程（可有效避免web页面刷机的各种问题）：
1. 上传固件到路由器，比如：/tmp/k3.trx

2. SSH，执行命令

```bash
mtd -r write /tmp/k3.trx firmware
```

## 其他设置的修改

### 修改LAN IP为 192.168.2.1

修改这个文件 package/base-files/files/bin/config_generate 里的 192.168.1.1 为 192.168.2.1
```bash
sed -i 's/192.168.1.1/192.168.2.1/g' package/base-files/files/bin/config_generate
```
### 默认WIFI设置

OpenWrt默认关闭WIFI的，需要修改配置文件以默认开启WIFI和修改SSID呢（方便刷机后不网线就可以连上路由器）

编辑package/kernel/mac80211/files/lib/wifi/mac80211.sh文件，大约在文件最后有如下代码

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
### 修改内核版本
在后面 Snapshot安装软件 部分也有提到，有一个这样的概括
- 如果要回滚repo到之前的某个commit，请参阅git使用指南
- 如果你要修改内核发行版本，比如4.9改到4.4，请修改对应target的Makefile中的KERNEL_PATCHVER
- 如果你要修改指定内核发行版本的修订版本，比如4.9.111改到4.9.110，请修改include/kernel-version.mk

# 使用体验
## 已知的问题

- 屏幕流量统计基于iptable的IPv4 Forward，然而再开启硬件转发的时候是统计不到的
- 屏幕路由网速监测基于默认的WAN的流量
- 无线偶尔会出现Not Associate的情况，需要重启（用过的OpenWrt都有这个问题）
- 查看无线连接部分都只有20MHz的频宽（实测发现是80MHz）

## 无线

当前想要作为无线路由使用的话是肯定要换无线固件的，我的7260AC网卡在近距离（同一个房间内）使用iperf3测试下载和上传速度，5G的实际传输速度300Mbp左右，5M，相隔两堵墙的情况下200Mbps左右，再远一点100Mbps，三堵墙就GG了，在我使用的OpenWrt的路由器中是最好的（一般隔一墙就GG），对比之前家里用的荣耀路由Pro（当时比较迷信华为的产品），在同样的位置K3可以满速，而前者已经是没有信号了，据说K3的无线功率相当的高（超出国标的那种），发热也很大，连接着两台设备的情况下，温度可以到70以上（还是改了散热的情况下）

尽管标称AC3150，参考[简说各种wifi无线协议的传输速率](https://www.acwifi.net/318.html)，K3是 4X4 MIMO + 80MHz + 1024-QAM = 2100Mbps ，单设备连接下达到这个速度需要PCE-AC88这个级别的网卡，结合现在的无线网卡市场（2X2 160Mhz网卡廉价且产品多），信号和速率方面其实已经难以和现在中端以上的路由器（200+）拉开差距了，相比其他的OpenWrt路由，K3只有信号强度有较大的实用价值，剩下的没有绝对的优势（如果不考虑外观的话）

注：多设备连接的MU-MIMO的情形，现在的OpenWrt还不支持，从这方面来说OpenWrt路由器不适合做高性能的AP

## CPU运算性能

就以专门为ARM设计的ChaCha20算法以及常用的AES-256-CBC测试，BCM4709@1.4G的单线程openssl加解密速度测试结果如下
```bash
root@K3:~# openssl speed -elapsed -evp chacha20
#k3
 type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes  16384 bytes
 chacha20         41251.72k    88523.18k    93098.98k    98190.37k    92384.53k    93584.75k          
 aes-256-cbc      24429.85k    27700.11k    30198.27k    30665.23k    29156.44k    29420.67k    
```
对比近几年主流的MT7621@880MHz以及MT7620@580MHz单线程测试结果
```bash
#K2P
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes  16384 bytes
chacha20         15635.27k    25852.26k    29409.89k    30429.13k    30748.58k    30835.67k
aes-256-cbc       7234.99k     8504.77k     8930.74k     9059.51k     9071.08k     9090.13k
#Y1
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes  16384 bytes
chacha20         10284.77k    17029.01k    19675.48k    18671.27k    20179.63k    20025.49k
aes-256-cbc       4877.62k     5635.73k     5527.98k     6159.36k     5644.29k     6100.31k
```

AES的测试结果大幅领先应该是得益于架构上的优势（BCM4709是ARMv7架构，不支持AES硬件加速，但是相比MIPS还是有优势的），就是日常使用温度比较高（40nm制程落后）

## Snapshot安装软件

Snapshot版本的源码是滚动更新的（包括内核版本），而官方的仓库[releases/19.07-SNAPSHOT/](https://downloads.openwrt.org/releases/19.07-SNAPSHOT/)中的软件分为package和kmod，前者一般对内核版本的倚赖不多，但是后者对内核版本是有比较严格的要求的，这就导致，即使是官方仓库中有的软件，都会出现无法安装的情况：

比如安装的Snapshot固件的内核版本是4.14.145，而滚动更新的仓库中的固件内核版本已经是4.14.149，如果安装某些软件，就会出现下面的错误：
```shell
Installing openvpn-openssl (2.4.7-2) to root...
Collected errors:
 * satisfy_dependencies_for: Cannot satisfy the following dependencies for openvpn-openssl:
 *      kernel (= 4.14.149-1-9b3f4da08295392b7d7eca715b1ee0b8)
 * opkg_install_cmd: Cannot install package openvpn-openssl.
```
其中也不仅仅是内核版本的问题，后面的一串生成自内核的编译参数校验，也就是说只有使用之前编译4.14.145内核的版本的Git版本再去做一次编译了(或者编译的时候留下的SDK)，反正比较麻烦

因为只有一串md5码，所以另辟蹊径也是可以的：[编译Openwrt固件安装软件内核版本不一致问题解决](https://www.haiyun.me/archives/1075.html)

所以Snapshot版本并不适合经常需要加装软件的情况，但是现在有19.07 rc1可用，其软件仓库是有官方更新的

## 功耗

先说个有趣的事情，因为自带的电源适配器太占插座了，我使用了 QC2.0的手机充电器 + 诱骗器 + USB-DC线 提供一个12V的供电，然而诱骗失效了，5V的电压下，路由器居然可以正常的工作，这意味着可以使用USB-DC线接到高输出的USB插座上给K3供电，这个也和功耗有关：

实测，开机及待机状态下功率在10W左右，如果遇到CPU占用较高或者高速无线传输峰值功耗可以到18W（360插座依然强而有力），使用5V和12V USB-DC的时候出现过几次无法开机的情况，所以稳定性还是一般的

对比MT7261的K2P，峰值功耗15W，但是平时使用就只有6W左右，K3的电费有些不太划算

**参考**

[OpenWrt 编译失败的原因及解决方案](https://p3terx.com/archives/reasons-and-solutions-for-openwrt-compilation-failure-3.html)

> 因为Windows文件系统与Linux不同，是默认不区分大小写的，推荐不挂载，如果挂载的话
>```
> docker run --rm -it -v $(pwd)/data:/root openwt_builder
>```
> powershell对```$(pwd)/data```不支持，需要替换成绝对路径```(echo "$(pwd)\data")```
>
> OpenWrt只能构建在区分大小写的文件系统上（Build dependency: OpenWrt can only be built on a case-sensitive filesystem），所以如果Docker挂载了Windows的目录的话需要设置下：
>```
> fsutil.exe file setCaseSensitiveInfo <path> enable
>```
> 但是设置完成之后git clone又不可用了（尴尬，所以一般还是用docker cp拷贝文件出来方便些）