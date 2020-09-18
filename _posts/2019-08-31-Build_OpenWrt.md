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

K3的无线性能貌似不错，现在也有了Snapshot固件可以下载，但是官方对屏幕做的适配不多，网络上的个人编译版本多是按照个人喜好来编译的，屏幕控制，无线信号，插件，各有优缺，那为什么不自己编译一个呢；本文介绍了19.07版本的新特性，并在windows下使用Docker编译了K3的固件；最后讨论了下内核模块相关的vermagic的问题

<!--more-->

# 前言
我翻了下OpenWrt的Table of Hardware，发现K3已经有了Snapshot的支持，并且也没有说明硬件不可用之类的，一般来说，下个stable release就完全可以拿来用了；因为其他的OpenWrt设备都在履行自己的义务，拿这个体验下新系统也不错，首先我是下载了编译好的snapshot固件，感受下19.07版本的新特性

本文的编译工作基于OpenWrt官方的19.07分支的源码，参考了部分已有的K3固件，加入了屏幕组件以及闭源无线驱动，因为GPL协议，所以不提供固件，但是使用了个人修改的屏幕组件的有恩山论坛的[K3 openwrt固件](https://www.right.com.cn/forum/thread-1275902-1-1.html)，基于的lean的lede，感兴趣可以试试

# OpenWrt 19.07

2019年11月，OpenWrt 19.07 rc1发布了，明显的变化：
- 大幅更新了LuCI界面
 - 扁平化 + 卡片式的二级设置页面
 - 移动端网页自适应布局（响应式设计）
 - 通过客户端渲染提高响应速度（主要的luci-app由lua迁移到js）
- 默认提供更完整的功能体验，包括但不限于
 - 安装软件方面支持网页查看依赖树以及上传ipk安装
 - 8M以上的设备默认使用wpad-basic，支持802.11r无线漫游
- 替换了网页的Favcion，内核的基线全部到4.14
- 其他的就是一些Bug修复，包括讨论的比较多的openssl，DHCPv6

其他的可见于[Release Notes](https://openwrt.org/releases/19.07/notes-19.07.0-rc1)，后文也提供了几张截图作为参考

另外还有18.06.5，修复了之前Web界面上status界面错位的问题，整体风格上部分和19.07同步

## LuCI优化

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

# 编译固件

这里对网络环境有一定的需求，一般都是推荐在国外的VPS上编译，如果在本地编译需要开代理，安装Docker和Git就不说了，如果觉得Docker和SSH的命令行对修改源码体验不好，可以使用VSCode加上Remote-Docker/SSH插件，再安装一个Terminal插件就很舒服了

部分表述还是以OpenWrt官方的[文档](https://openwrt.org/docs/guide-developer/build-system/use-buildsystem)为准

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
对于18.06.4之类的发行版，可以在release下载源码或者切换到相应的tag，如：
```bash
git checkout v18.06.4
```

### 下载软件包源码

**注意**：在2020年8月的提交[683bcbd](https://github.com/openwrt/openwrt/blob/a14f5bb4bd263c21e103f13279d0c2ff03e48fe5/target/linux/bcm53xx/image/Makefile#L391)之后，TARGET的名称由phicomm-k3变为了phicomm_k3，另外menuconfig上bcm53xx还需要选择subtarget，进而对Makefile产生了一些影响，博客和仓库均做了适配，建议使用较新版本源码编译时更新下k3screenctrl_build的Makefile:
```Makefile
DEPENDS:=@(TARGET_bcm53xx_generic_DEVICE_phicomm_k3||TARGET_bcm53xx_generic_DEVICE_phicomm-k3||TARGET_bcm53xx_DEVICE_phicomm-k3)
```

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

先介绍一个技巧：修改固件的Makefile使其只编译K3的固件，大大减少编译需要的时间，此处相对之前也做了TARGET名称变化的兼容

```bash
sed -i 's|^TARGET_|# TARGET_|g; s|# TARGET_DEVICES += phicomm-k3|TARGET_DEVICES += phicomm-k3| ; s|# TARGET_DEVICES += phicomm_k3|TARGET_DEVICES += phicomm_k3|' target/linux/bcm53xx/image/Makefile
```
### 编译选项

之后打开编译选项：
```bash
make menuconfig
```

- Target 选 Broadcom BCM47xx/53xx (ARM)
- 在2020年9月的Master分支上，menuconfig上bcm53xx还需要选择subtarget，generic即可
- Target Profile 选 PHICOMM K3 （如果用了上述技巧的话就只有K3可选）
- Utilities 确保 k3screenctrl 选中，倚赖会自动选上
- LuCI -> Application 中的 luci-app-k3screenctrl
- snapshot默认没有的web管理界面 LuCI -> Collection -> luci

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
如果之前刷过OpenWrt系统仅作小版本的升级的话，使用luci升级也是可以的

## 其他设置的修改

### 修改LAN IP为 192.168.2.1

修改这个文件 package/base-files/files/bin/config_generate 里的 192.168.1.1 为 192.168.2.1
```bash
sed -i 's/192.168.1.1/192.168.2.1/g' package/base-files/files/bin/config_generate
```
### 默认WIFI设置

OpenWrt默认关闭WIFI的，需要修改配置文件以默认开启WIFI和修改SSID呢（方便刷机后不网线就可以连上路由器）

编辑 package/kernel/mac80211/files/lib/wifi/mac80211.sh 文件，大约在文件最后有如下代码

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
在逛论坛时看到了这样的概括：
- 如果要回滚repo到之前的某个commit，请参阅git使用指南
- 如果你要修改内核发行版本，比如4.9改到4.4，请修改对应target的Makefile中的KERNEL_PATCHVER
- 如果你要修改指定内核发行版本的修订版本，比如4.9.111改到4.9.110，请修改include/kernel-version.mk

在后面一节会提到，内核版本后接的一段参数

## 内核版本问题

Snapshot版本的源码是滚动更新的（包括内核版本），而官方的仓库[releases/19.07-SNAPSHOT/](https://downloads.openwrt.org/releases/19.07-SNAPSHOT/)中的软件分为package和kmod，前者一般对内核版本的倚赖不多，但是后者对内核版本是有严格的要求的，这就导致，即使是官方仓库中有的软件，都会出现无法安装的情况：

比如安装的Snapshot固件的内核版本是4.14.145，而滚动更新的仓库中的固件内核版本已经是4.14.149，如果安装某些软件，就会出现下面的错误：
```shell
Installing openvpn-openssl (2.4.7-2) to root...
Collected errors:
 * satisfy_dependencies_for: Cannot satisfy the following dependencies for openvpn-openssl:
 *      kernel (= 4.14.149-1-9b3f4da08295392b7d7eca715b1ee0b8)
 * opkg_install_cmd: Cannot install package openvpn-openssl.
```
其中```4.14.149-1-9b3f4da08295392b7d7eca715b1ee0b8```是 Kernel_version + vermagic ，前者就是内核版本，后者与内核的编译参数有关

### vermagic

[编译Openwrt固件安装软件内核版本不一致问题解决](https://www.haiyun.me/archives/1075.html)，其中就给出了vermagic的由来，在新版本的代码中：

```bash
grep '=[ym]' $(LINUX_DIR)/.config.set | LC_ALL=C sort | mkhash md5 > $(LINUX_DIR)/.vermagic
```

上面的文章给出的解决方法就是强行替换和官方仓库一致的vermagic，感觉不够优雅

之后翻了下OpenWrt官方论坛，找到了维护者的一次回复：

  > vermagic is a hash calculated from
  >
  > - all compilation options related to kernel, and
  > - names of all kernel modules enabled in the kernel compilation .config (either =y or =m)
  >
  > In practice, any change to the config makes the modules officially incompatible.
  >
  > - It is possible to compile individual modules later for a firmware if you have also compiled the static SDK and use that SDK for the kmod compilation (like eduperez does not mwlwifi driver kmods here in the forum)
  > - It is practically impossible to compile a new kernel or full firmware and then try to use older kmods with opkg. The vermagic will differ

说的算是比较清楚了（相对于没有找到config.set的具体生成方式），下面介绍几种个人试过的方法，先说明的是，暂时没有找到直接编译得到vermagic和官方一致的最优方法，至于imagebuilder，暂时还没用过

### 编译vermagic相同的固件

对于某些情况来说是不可避免的，如
- 编译能够使用官方的软件仓库的固件
- 在使用官方固件的情况下，编译兼容的包，如kmod-nf-fullconenat
- 在编译vermagic与官方不一致的固件的情况下，需要再次编译兼容的包

对于第一种情况，在一篇不久前更新的博文中找到了较为满意的方法：[How to compile OpenWrt and still use the official repository](https://hamy.io/post/0015/how-to-compile-openwrt-and-still-use-the-official-repository/)

> As part of building kernel, *.config.set* file is created. This file includes all the applied kernel settings.
>
> To get the same `vermagic` value, not only we need to use the same exact OpenWrt source version, but also build our image with the same exact set of kernel settings and modules.
>
> In a nutshell, `config.seed` includes all the required changes for building the same image again. In other words, it contains all the changes that’s been applied to an image compared to the default configuration.

```config.seed```文件在新版的仓库中就是```config.buildinfo```，暂时没尝试，因为即使是编译所有的内核模块但是不安装，编译一次固件依然需要**耗费大量时间**

对于第二种情况，第一种方法依然可用，细微的调整在于，编译菜单不能直接勾选内核模块，而是在使用官方的config.buildinfo文件的情况下，单独编译内核模块

更见快捷的方法是使用官方提供的SDK，对K3可以在[19.07.1/targets/bcm53xx/generic/](http://mirrors.ustc.edu.cn/lede/releases/19.07.1/targets/bcm53xx/generic/)中找到```openwrt-sdk-19.07.1-bcm53xx_gcc*.tar.xz```，在不修改设置的情况下，编译出来的包的vermagic与官方固件是一致的

第三种情况多见于自编译固件，因为：
- 对于各路论坛下载到的固件，往往没有放出修改后的源码，config也不得而知
- 一般自编译固件的过程和上文差不多，不会注意原始的config文件

至于方法都是一样的，自编译固件可以留下编译时的config以及SDK，方便再次编译兼容的包

### 强制修改vermagic
还有一种非常普遍的情况就是，使用的固件既没有SDK，也没留下config

如果确实没有对内核设置之类的做太多的修改的话，使用中文博客中的方法：
> 编译后指定内核版本：
> ```bash
> sed -i 's/eac88df3cb49b94d68ac3bc78be57f95/3051dee8f07064b727e9d57fbfeb05ec/' /usr/lib/opkg/status
> ```

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