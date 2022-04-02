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
# Home Assistant简介
按照[官网](https://www.home-assistant.io)描述，HomeAssistant是一个开源的、以本地控制和隐私保护为基础的家庭自动化系统。有以下几个特点
1. 开放
  HA是一个开源的平台系统，可以在[github](https://github.com/home-assistant)找到相关源代码，同时它有丰富的开放API，因此有了丰富的插件（如Integration、Frontend等等）支持，可以带来这些好处：1）将其他平台的设备，全部接入HA，这样就实现了家庭智能设备的统一控制，如小米、涂鸦、Sonoff、；2）将HA上的设备统一接入其他平台，如你一堆的小米设备接入苹果的HomeKit，可以实现Siri来控制小米设备；3）其他自定义设备如软路由、NAS、DIY的设备通过各种方式接入HA控制；4）提供iOS、Android、PC各平台的控制终端
2. 本地部署
  HA往往部署在本地的设备，这样有2个好处：1）速度快，比如你想做一个开门后自动开灯且播放欢迎提示音的自动化，使用云服务可能要几秒，使用家庭本地网络的HA会节省了公网网络传输的时间消耗；2）隐私，如果你比较担心家里的智能音箱、各种智能设备在偷偷监听上报数据（这个事情完全靠厂商自觉，本身智能设备就要持续上报数据，你无法判断是否上报了与其功能无关的数据），使用HA就没有这方面的苦恼了。
3. 更新频繁
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
| 磁盘/TF卡 | 16G以上：8G的存储是不够的，armbian安装加上容器大概需要7G左右；最好是大牌的如铠侠、闪迪、三星这些；也可以外接一个16g以上的SSD| 我之前用过杂牌TF卡，读写太慢会因为IO负载高最终导致系统无响应，后来用N1+铠侠64G，用了几个月感觉有点浪费，最后用了盒子原生的2+16EMMC的配置，刚好够 | 
| 网络 | 有线网卡不需要千兆，百兆足够，；无线没太大用处 | 做服务器流量不大，但要响应快，有线比较靠谱 | 
| 蓝牙 | 能驱动的板载蓝牙；USB蓝牙接收器；没有蓝牙也可以 | 蓝牙的功能很重要，很多智能设备都是蓝牙的，如果armbian系统没有可驱动的蓝牙，也可以有其他替代办法 | 


## 试过的盒子
1. R3300-L
![IMG_5931](https://user-images.githubusercontent.com/2712885/161371283-2a532ffd-d4c3-4c08-a9f4-2ae0ea84257f.jpg)
![IMG_5932](https://user-images.githubusercontent.com/2712885/161371289-b178610d-5090-44cf-8f14-56a9d3fe80bc.jpg)

最开始买了这个小红盒，由于含有一个读卡器，且通过AV孔可以不拆机按住Reset键从而直接刷机，所以还比较受欢迎。最开始刷了Openwrt+Docker-ce安装HA，因为内存只有1G，会出现上文说道的负载高无响应的问题，所以后来删除了HA，只用作Openwrt，Openwrt本身对内存的要求不高（只需要100M左右），目前一直在用，半年没重启过了，稳得很。

2. N1
![n1](https://user-images.githubusercontent.com/2712885/161371533-2e883c96-e6f3-46e6-b671-a95c30632806.jpg)
N1真的是各方面都基本够用，S905D+千兆网口+蓝牙+双频WiFi，不太有短板，怪不得这么几年过去了还是这么火，蓝牙是博通的芯片，可以Armbian直接驱动。但是如果刷Armbian会觉得它没有读卡器比较鸡肋，当然用USB外接读卡器或者直接接SSD都是好选项。N1+读卡器+64G卡用了大半年，没太大的问题，就是感觉老外接一个读卡器怪难受。

3. M401
![m401a](https://user-images.githubusercontent.com/2712885/161371740-77038928-5101-4a5b-ae7e-0f7814c70822.jpg)
![m401a2](https://user-images.githubusercontent.com/2712885/161371749-74aa6b3f-e7f3-4877-bc22-466716c837d3.jpg)
最近在网上捡了一个移动新出的盒子M401A，买的时候只看重了它2g内存+16gEmmc的配置，终于不用外接TF卡了。买来之后发现这个CPU是S905L3A，性能还不错，只是当前蓝牙还无法驱动。先是插了一个蓝牙接收器，但是蓝牙老掉线，后来索性就不要蓝牙了。
