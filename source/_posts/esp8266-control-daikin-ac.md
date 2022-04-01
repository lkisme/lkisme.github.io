---
title: 大金空调面板Hack添加WiFi支持
date: 2022-03-29 14:59:12
tags: homeassistant 大金 智能家居
categories: 智能家居
metaAlignment: center
thumbnailImage: https://user-images.githubusercontent.com/2712885/161278001-c51e14f9-c5c0-44f2-9dce-0bb5f0f5542b.jpg 
thumbnailImagePosition: bottom
coverImage: https://user-images.githubusercontent.com/2712885/161278001-c51e14f9-c5c0-44f2-9dce-0bb5f0f5542b.jpg 

coverCaption: "The bund"
coverMeta: out
coverSize: partial

---


## 背景
装修时家里装了大金的1拖2中央空调，当时可以选“金制家中”的套餐，只要加1000块，结果脑抽没选。入住之后马上感觉到不爽的地方：
1. 无法远程控制：夏天下班回家后，家里还是很热，立马开空调也要过几分钟才会凉下来，冬天同理。
2. 无法远程控制：冬天要起床开关空调，
3. 其他扩展功能：定时关定时开只能在面板上选择，且只能以半小时为单位。
4. 指示灯的问题：晚上睡觉指示灯一直亮着，烦人

<!-- more -->

## 可选方案
首先是想买一套金制家中，大概就是一个Wi-Fi网关+几个传感器，这个主要问题是要装在外机上（大误），于是放弃了，继续搜索可以接入米家控制的设备，大概有三种：
1. 大金官方的方案：WiFi控制器（金制家中）
1. vrf面板，价格在300-600之间，要替换掉自带面板，特点是安装简单，且可以卖掉原版的线控，回一波血
2. 线控网关，原理和#1一样，接在空调内机的P1/P2上，价格在300-400之间。安装比较麻烦
3. 集控网关，原理和金制家中一样，接在F1/F2上，可以控制所有内机。价格在1000-2000之间，安装难度和#2一样，特点是一台机器可以控制所有内机，如果家里内机较多，可以考虑这个
4. DIY：难度较高

## 原理分析

下图是线控网关的安装拓扑

<img width="645" alt="xiankong" src="https://user-images.githubusercontent.com/2712885/144000776-37a099fd-2056-4140-bc03-1bcf7cb4c0b5.png">
对于大金空调的控制，大概的方式可以分为2种

### F1/F2
单台外机：空调网关的F1/F2连接大金中央空调的室内机或者室外机的“F1/F2”。
多台外机：空调网关的F1/F2连接大金中央空调室外机的“F1F2”。

金制家中应该就是这种方案，此外集控网关也是这种方案，优点是一台机器控制所有空调。
![jikong](https://user-images.githubusercontent.com/2712885/144000705-39cbc944-6fdf-4b57-98e2-7dacc64d3f5c.jpg)


### P1/P2
基于HBS协议的载波通信。没有太深入的了解，大概就是这两根线既可以提供电源（压差大概15V多一点），又可以载波通信（高20V，低10V）。国内可以买到的很多线控网关都是做了一个HBS的解析器+控制器。网上也有开源的方案，但要自己做硬件，于是放弃。
![hbs-chip](https://user-images.githubusercontent.com/2712885/144000648-b6d5cf7c-6c57-4ea9-88b1-d5709a9bb5c8.png)

所有的线控都是内机->解码->控制器->网关这样的链路。而自带的面板只是缺了「网关」这一环，所以我们可以制作一个可以控制面板的带有网关功能的设备，就可以轻松实现联网控制空调了。


## 实操

首先我买了一个线控网关，接起来真的费劲，接线图如下所示：

其原理上边讲了，相当于一个内机接了2个遥控，第二个遥控配上了WiFi网关。

第二个内机不想再买线控网关了，觉得这看起来功能很简单啊，于是在网上搜相关的方案，搜到了这个：https://lecoding.com/post/2020/daikin-remote-controller-hack/ ，这偏文章里通过红外来控制原始的控制面板，但有以下几个问题：如果需要远程控制，需要再加一个红外网关。而我最近在研究8266，手里拿着锤子看见所有东西都是钉子，于是开始改造。但改造失败，因为接上8266之后面板就会重启，之后会不断的重启，另外由于没有断电操作，把面板上的电源指示灯LED烧了，额外解决了最开始的#4个诉求。。。

![front_before](https://user-images.githubusercontent.com/2712885/144226767-0e6ff656-9adb-4948-8b66-f4ee0aca2bbd.jpg)

改造计划暂时搁置，最近又在英文世界里冲浪，找找老外的解决方案，找到了一个使用Arduino来解决问题的：[AIRDUINO | ADDING NETWORK REMOTE CONTROL TO THE DAIKIN VRV WITH THE BRC1D61 OR SIMILAR THERMOSTATS](https://mattala.com.au/2016/08/14/airduino-network-enabling-the-daikin-vrv-with-the-brc1d61-or-similar-thermostats/)，于是又燃起了改造的激情，这次增加了读取LED状态的接线，这样就能精确控制空调了。接线如下：
![front_done](https://user-images.githubusercontent.com/2712885/144226833-8ad835fd-67be-4d16-a9c5-cabd103a09f9.jpg)

![back](https://user-images.githubusercontent.com/2712885/144226809-9030fd06-a16e-4151-8d8b-39529d08844e.jpg)

### 电源问题

接上之后好了几个小时，面板又重启了，另外一接上8266，就能听到滋滋的声音，应该是面板上15V转5V的变压器受不了额外的载荷，发出了以此抖动的声音。于是不得已，又从相邻的卧室灯开关的接线盒拉出来了220V的线：


## 最终方案
1. 220V接AC-DC电源变3.3V接8266电源，将8266和电源塞进了面板背后的86接线盒内
2. 面板的GND接8266的GND
3. 面板的2条引线分别串联20K左右的电阻，接8266的GPIO引脚
4. HA配置如下所示

```
  - platform: mqtt
    name: "卧室空调"
    unique_id: switch.wifi.bedroom_ac
    payload_on: "1"
    payload_off: "0"
    state_topic: daikin/bedroom/state
    command_topic: daikin/bedroom/command
    availability_topic: daikin/bedroom/availability
    json_attributes_topic: daikin/bedroom/attrs
```


家里有一台N1装了HASS+NodeRed+MQTT，自制了一个离线的语音控制器放在卧室，这样就可以开关（灯、暖气、空调、其他）基本靠吼了。

### 最终形态
![最终形态](https://user-images.githubusercontent.com/2712885/160620943-1db84d92-4889-4da0-9465-6e537c43c9e3.jpg)

