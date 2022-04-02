---
title: homeassistant-on-cheap-box
tags: homeassistant docker armbian
categories: 智能家居
thumbnailImage: https://user-images.githubusercontent.com/2712885/161280108-acbbbd42-1c65-45f8-ab58-66b5face9d9c.jpg
coverImage: https://user-images.githubusercontent.com/2712885/161280108-acbbbd42-1c65-45f8-ab58-66b5face9d9c.jpg
coverCaption: "The bund"
coverMeta: out
coverSize: partial

---

本文将包含如下内容：
1. Home Assistant简介
2. 可安装Armbian的盒子简单分析
3. 安装Armbian的简要步骤
4. 安装Home Assistant的简要步骤
<!-- more -->

# Home Assistant简介 
按照[官网](https://www.home-assistant.io)描述，HomeAssistant是一个开源的、以本地控制和隐私保护为基础的家庭自动化系统。有以下几个特点
## 1. 开放
  HA是一个开源的平台系统，可以在[github](https://github.com/home-assistant)找到相关源代码，同时它有丰富的开放API，因此有了丰富的插件（如Integration、Frontend等等）支持，可以带来这些好处：1）将其他平台的设备，全部接入HA，这样就实现了家庭智能设备的统一控制，如小米、涂鸦、Sonoff、；2）将HA上的设备统一接入其他平台，如你一堆的小米设备接入苹果的HomeKit，可以实现Siri来控制小米设备；3）其他自定义设备如软路由、NAS、DIY的设备通过各种方式接入HA控制；4）提供iOS、Android、PC各平台的控制终端
## 2. 本地部署
  HA往往部署在本地的设备，这样有2个好处：1）速度快，比如你想做一个开门后自动开灯且播放欢迎提示音的自动化，使用云服务可能要几秒，使用家庭本地网络的HA会节省了公网网络传输的时间消耗；2）隐私，如果你比较担心家里的智能音箱、各种智能设备在偷偷监听上报数据（这个事情完全靠厂商自觉，本身智能设备就要持续上报数据，你无法判断是否上报了与其功能无关的数据），使用HA就没有这方面的苦恼了。
## 3. 更新频繁
  HA目前一个月要更新好几个版本，一些漏洞在持续修复，能力也在持续优化，同时有很大的社区支持，

# HA安装
官网给出4中安装方法（后2中不推荐）：
1. Home Assistant Operating System：作为操作系统直接安装在机器上
2. Home Assistant Container:容器安装，如docker
3. Home Assistant Supervised：手动安装（不推荐）
4. Home Assistant Core:手动安装（不推荐）

HA底层基于Python，基本所有的系统都可以支持，所以几乎不挑硬件，X86、ARM平台都可以，官网推荐了两种ARM的单板系统（SBC，single-board computers）:
1. Home assistant Yellow（基于树莓派）

![Home assistant Yellow](https://www.crowdsupply.com/img/7b5c/home-assistant-amber-with-cm4-no-heat-sink-top_jpg_md-xl.jpg)
2. Home assistant Blue（基于Odroid）
![Home assistant Blue](https://cdn.hardkernel.com/wp-content/uploads/2020/07/odroidn2plusA.jpg)

# 平替
这两个SBC在中国不太好买，其次价格较高（Yellow的最低价格115刀，Blue的价格可以去搜Odroid N2，树莓派今年的价格也飙上去了）100刀都可以买到一个X86的主机来做server了。仔细看一下这个Blue，他的CPU是晶晨的S922X，晶晨还有S905系列，市场容量相当大，于是买了几个来试试。

## 硬件要求

| 部件 | 要求 | 备注|
| ---- | ---- | ---- | 
| CPU | 64位的4核ARM | 32位的晶晨S805（ARMv7）应该是不行的，跑很多东西找不到合适的包。S905、S922的各种型号，全志，只要能装Armbian都可以 |
| 内存 | 1G勉强也够，最好2G | 1G内存一般可用的有800M，我是用Docker跑，跑起来HA的7个容器，内存就已经捉襟见肘了，再跑一些插件、MQTT、nodered这些容器就不够用了，内存不够用会导致系统频繁读写交换空间Zram，导致系统负载高、无响应 |
| 磁盘/TF卡 | 16G以上：8G的存储是不够的，armbian安装加上容器大概需要7G左右；最好是大牌的如铠侠、闪迪、三星这些；也可以外接一个16g以上的SSD| 我之前用过杂牌TF卡，读写太慢会因为IO负载高最终导致系统无响应，后来用N1+铠侠64G，用了几个月感觉有点浪费，最后用了盒子原生的2+16eMMC的配置，刚好够 | 
| 网络 | 有线网卡不需要千兆，百兆足够，；无线没太大用处 | 做服务器流量不大，但要响应快，有线比较靠谱 | 
| 蓝牙 | 能驱动的板载蓝牙；USB蓝牙接收器；没有蓝牙也可以 | 蓝牙的功能很重要，很多智能设备都是蓝牙的，如果armbian系统没有可驱动的蓝牙，也可以有其他替代办法 | 

由此看出，2G内存+16geMMC的配置是比较划算的，如果没有特殊需求，不需要另加其他外设。目前市场上还可以买到海思的ARM盒子，但海思芯片刷Armbian的不多，社区文档页少（可以看这个：[glinuz/hi3798mv100](https://github.com/glinuz/hi3798mv100)），尽量还是选择其他芯片。
## 试过的盒子
### 1. R3300-L

![IMG_5931](https://user-images.githubusercontent.com/2712885/161371283-2a532ffd-d4c3-4c08-a9f4-2ae0ea84257f.jpg)
![IMG_5932](https://user-images.githubusercontent.com/2712885/161371289-b178610d-5090-44cf-8f14-56a9d3fe80bc.jpg)

根据成色、是否刷机、配件不同，价格也不同，普遍在50-60左右。

最开始买了这个小红盒，由于含有一个读卡器，且通过AV孔可以不拆机按住Reset键从而直接刷机，所以还比较受欢迎。最开始刷了Openwrt+Docker-ce安装HA，因为内存只有1G，会出现上文说道的负载高无响应的问题，所以后来删除了HA，只用作Openwrt，Openwrt本身对内存的要求不高（只需要100M左右），目前一直在用，半年没重启过了，稳得很。

### 2. N1

![n1](https://user-images.githubusercontent.com/2712885/161371533-2e883c96-e6f3-46e6-b671-a95c30632806.jpg)
现在全套配件的得150+了，不是很划算，但确实N1这种各方面都不错的盒子也不多。
N1真的是各方面都基本够用，S905D+千兆网口+蓝牙+双频WiFi，不太有短板，怪不得这么几年过去了还是这么火，蓝牙是博通的芯片，可以Armbian直接驱动。但是如果刷Armbian会觉得它没有读卡器比较鸡肋，当然用USB外接读卡器或者直接接SSD都是好选项。N1+读卡器+64G卡用了大半年，没太大的问题，就是感觉老外接一个读卡器怪难受。

### 3. M401、E900V22C等新版魔百和


![m401a](https://user-images.githubusercontent.com/2712885/161371740-77038928-5101-4a5b-ae7e-0f7814c70822.jpg)
![m401a2](https://user-images.githubusercontent.com/2712885/161371952-f717f3af-c8dd-4748-9222-cd16f298e3ca.jpg)
原装全套（盒子+网线+HDMI线+蓝牙遥控）价格在70左右，盒子比较新，不太买得到单机头的
最近在网上捡了一个移动新出的盒子M401A，买的时候只看重了它2g内存+16geMMC的配置，终于不用外接TF卡了。买来之后发现这个CPU是S905L3A，性能还不错，只是当前蓝牙还无法驱动。先是插了一个蓝牙接收器，但是蓝牙老掉线，后来索性就不要蓝牙了。

### 4. 其他ARM盒子、板子
树莓派、香橙派、各种派

# 安装步骤（卡刷）
卡刷或者线刷都可以，其中卡刷简单方便，不容易损坏盒子本身（丢三码），下文以卡刷为例
### 1. 寻找Armbian镜像并写入U盘或者TF卡
根据型号在网上找到合适的镜像，注意尽量不要找太老的镜像，如果是Ubuntu尽量选1804版本的或者2004版本的，版本较老本身没有太大的问题，但随着软件更新，难免未来某些软件就不支持老版本的内核了。大家在网上找到的镜像其实都差不多，无非就是根据不同的硬件所带的dtb和驱动不同，所以如果无法找到相应的镜像，可以随便找一个较新的的镜像，修改合适的dtb就行了。
这里提供一个R3300L的镜像： https://pan.baidu.com/s/1BesJhaKfKMF1WJapmUH6_Q 提取码: vs5g 下载：Armbian_5.99_Aml-g12_Ubuntu_bionic_default_5.3.0_rtl8189ftv.tar，不带桌面，带WiFi。
持续更新的镜像下载：https://github.com/zzcand111/amlogic-s9xxx-armbian 。历史的一些镜像下载：https://users.armbian.com/balbes150/arm-64/ (5.9的内核)。
要注意，5.9的内核我已经试过写入eMMC，可以正常使用，如果是5.9以上的内核，很多人说刷入eMMC会导致无法启动。
将镜像解压出来img格式，用Rufus等软件写入TF卡或者U盘。
### 2. 更换DTB，从U盘启动Armbian
写好U盘镜像后，U盘上会有一个FAT32的名叫BOOT的分区，如果是https://users.armbian.com/balbes150/arm-64/ 下载的镜像，需要做2件事情：
1. 在extlinux/extlinux.conf文件中修改dtb文件
2. 将u-boot-s905（只针对S905) u-boot-s905x2-s922* u-boot-s905x-s912（x905[w,d,l]）中的某1个名词改为u-boot.ext

如果是其他地方下载的，只需要在BOOT分区的根目录下的uEnv.ini文件中修改dtb文件即可。
M401a、E900V22c等s905L3a的芯片使用dtb：`meson-g12a-u200.dtb`，R3300L等s905L芯片使用`meson-gxl-s905x-p212.dtb`。其他的机器找到响应的dtb文件即可。

#### U盘启动的方法
R3300L用牙签插入AV孔，按住Reset键，然后插上电源。其他盒子可以尝试root后安装从U盘启动的APP，R401可以拆开外壳，Reset键在板子背后，其他还有按遥控器的方法，可以根据型号自行搜索。从U盘启动后可以看有线网络是否驱动，这个是最重要的，一般都可以驱动。
第一次启动可能会花几分钟时间，因为系统要执行一些初始化的任务，可以插上HDMI屏幕，看一下是否成功。
### 3. 备份eMMC的系统
进入U盘的Armbian系统，执行命令ddbr，根据提示选择备份系统到U盘。

### 4. 写入eMMC
一般的CPU（ s905[x,w,l,x2,x3],s912,s922）执行命令`/root/install-aml.sh`，将系统写入eMMC。如果CPU是S905（不带任何后缀字母的版本），执行`/root/install-aml-s905-emmc.sh`。写入成功后断电重启。

### 5. 更新证书
如果下载的比较老的镜像，在更换apt源之前需要先更新本机证书：`apt install --reinstall ca-certificates`，不然由于本机证书比较旧，换源后会识别新的更新源证书无效。

### 6. 换源

```shell
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
#deb-src  focal main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
#deb-src http://ports.ubuntu.com/ focal-security main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
#deb-src http focal-updates main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
#deb-src http://ports.ubuntu.com/ focal-backports main restricted universe multiverse

```
换源后执行：`apt update`
### 7. 安装docker-ce
```shell
#创建安装目录
cd; mkdir download;cd download
#下载安装脚本
curl -fsSL https://get.docker.com -o get-docker.sh
#赋予权限
chmod a+x get-docker.sh 
#执行安装
./get-docker.sh --mirror Aliyun


```
安装完成后，修改国内镜像（加速下载镜像），更改docker目标文件夹（方便备份）
```shell
vim /etc/docker/daemon.json
#填入如下内容
{
  "bip": "172.31.0.1/24",
  "data-root": "/opt/docker/", #docker安装目录，如果eMMC不够，可以把此目录挂载到外部TF卡或者SSD
  "log-level": "warn",
  "log-driver": "json-file",
  "log-opts": {
     "max-size": "10m",
     "max-file": "5"
   },
   "registry-mirrors": [
     "https://xxxx.mirror.aliyuncs.com" #加速镜像
   ]
}
# 重启docker
systemctl restart docker
```
### 8. 安装HA
安装HA比较简单，只需要安装homeassistant-supervisor，它会自动安装所有其他镜像。根据网速、CPU、磁盘写入速度不同，安装时间不确定，大概十几分钟到1个小时之间。
```shell
#让homeassistant不做健康检查，/opt/docker为之前设置的docker安装目录
vim /opt/docker/hassio/jobs.json
写入：
{"ignore_conditions": ["healthy"]}
#安装hassio-supervisor
docker run -d --name hassio_supervisor  --privileged \
-v /var/run/docker.sock:/var/run/docker.sock \ 
-v /var/run/dbus:/var/run/dbus \
-v /opt/docker/hassio:/data \ #指定HA目录
-e SUPERVISOR_SHARE="/opt/docker/hassio" \
-e SUPERVISOR_NAME=hassio_supervisor \
-e HOMEASSISTANT_REPOSITORY="homeassistant/qemuarm-64-homeassistant" \
--restart unless-stopped homeassistant/aarch64-hassio-supervisor:2022.03.5 #2022.03.5是版本号，可以去https://hub.docker.com/r/homeassistant/aarch64-hassio-supervisor/tags 找一个最新的稳定版版本

```
想看到执行过程的，可以执行`docker logs -f hassio_supervisor`查看实时日志，有可能有些版本的docker安装好supervisor后不能上网，可以临时添加`--net host`使它可以上网下载其他镜像，完成后再去掉此选项。因为这几个HA的镜像必须在一个内网才能互相访问。

#### 9. 其他安装项
其他如MQTT、nodered可以在HA安装好后使用HAsupervisor以插件的形式安装，但如果想自己安装也可以。
```shell
#安装MQTT，需要进入镜像修改配置文件
docker run --name mqtt --restart=always -dit -p 1883:1883 -p 9001:9001 -v /mosquitto/data -v /mosquitto/log eclipse-mosquitto

#安装nodered，需要手动设置/opt/docker/node_red_data的访问权限
chmod 777 /opt/docker/node_red_data
docker run -d -it -p 1880:1880 -v /opt/docker/node_red_data:/data --name nodered nodered/node-red

```

### 10. 网页打开HA页面，
假设Armbian的IP是192.168.3.3，则电脑打开[http://192.168.3.3:8123](http://192.168.3.3:8123)查看安装状态，如果安装完成，则会出现设置密码的界面。
