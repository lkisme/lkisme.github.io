---
title: Home Assistant控制设备
tags:
---
本文将包含如下内容：
1. Home Assistant配置使用HACS
2. 连接控制小米设备
3. 连接控制Sonoff设备
4. 连接控制涂鸦设备
5. 连接控制自定义设备
6. Home Assistant备份与恢复

<!-- more -->

# 安装HACS
在HA的主界面点击配置（Configuration）->设备和服务（Devices&Services）->添加集成（Add integration）可以看到系统可用的集成列表，并且可以点击安装。自定义集成在安装成功后也需要在这里进行安装。
Home Assistant在启动时会扫描主目录（我的目录是/opt/docker/hassio/homeassistant）下的custom_components文件夹，识别其中的集成（Integration）并加载到可用集成列表。HA官方提供了丰富的集成，但还是有很多功能需要安装自定义集成来完成，安装自定义集成有一个比较简单的工具是HACS（Home Assistant Community Store）。
注意：HACS强依赖GitHub，它的更新、下载等动作都要去GitHub下载，如果你无法访问GitHub，可以不使用HACS，因为速度会让人崩溃。
### 1. 下载最新包
去https://github.com/hacs/integration/releases 下载最新的包，将下载的包复制到custom_components文件夹内并解压，会生成一个hacs文件夹，重启HA。
或者去https://hacs.xyz/docs/setup/download 查看文档，使用脚本下载。
### 2. 安装
在HA的主界面点击配置（Configuration）->设备和服务（Devices&Services）->添加集成（Add integration），搜索HACS，并安装。之后按照https://hacs.xyz/docs/configuration/basic 提示绑定GitHub账号。

# 连接控制小米设备
所有已经接入米家APP的设备，都可以通过此方法接入。

### 安装`Xiaomi Miot Auto`
开发者是中国人，教程很丰富，功能很强大。安装教程可以参考：https://github.com/al-one/hass-xiaomi-miot/blob/master/README_zh.md
安装好之后需要配置米家账号，一定要选”更新设备列表“的选项，之后可以选择哪些设备想用HA来控制，毕竟像我有个扫地机器人，使用HA控制就毕竟鸡肋，所以索性排除在外。
注意：`Xiaomi Miot Auto`使用了米家的公共API，对于很多不同的设备支持方式也不同，会导致偶尔连不上设备的情况，且HA的服务器和设备最好在1个网段，不然一些局域网控制的设备会连不上，偶尔设备重启换IP后也会导致连不上。如果连不上某些设备，可以尝试在配置（Configuration）->设备和服务（Devices&Services）->点击集成的重载按钮。

### 连接蓝牙设备
使用`Xiaomi Miot Auto`已经可以控制米家设备了，但是很多设备使用的是云-云对接，就是说我们本地的指令需要通过`Xiaomi Miot Auto`传输到米家后台，然后再下发到设备上，对于一般的WiFi设备还好，但是对于像温度计、门磁传感器这种蓝牙设备，首先你需要1个蓝牙网关，其次`Xiaomi Miot Auto`采用的是轮询方式查询状态，如果你想做一个开门后自动开灯的自动化，可能你门都关上了，灯还没开，这种延迟可以达到几分钟，让人奔溃。而蓝牙是广播协议，如果使HA可以直接接收蓝牙设备的广播信息，那么就不需要蓝牙网关了，而且速度会特别快。

#### 1. 安装蓝牙控制软件
在Armbian的命令行执行`armbian-config`，在跳出的界面中选择Network，选择安装Bluetooth。这一步是安装一些蓝牙相关的控制软件。
之后执行`hciconfig -a`，就可以看到蓝牙设备的信息（00:xx:xx:xx:xx:xx是蓝牙地址）：
```shell
root@arm-64:/opt/docker/hassio/homeassistant# hciconfig -a
hci0:	Type: Primary  Bus: USB
	BD Address: 00:xx:xx:xx:xx:xx  ACL MTU: 1021:8  SCO MTU: 64:1
	UP RUNNING 
	RX bytes:1005 acl:0 sco:0 events:55 errors:0
	TX bytes:3181 acl:0 sco:0 commands:55 errors:0
	Features: 0xbf 0xfe 0xcf 0xfe 0xdb 0xff 0x7b 0x87
	Packet type: DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV2 HV3 
	Link policy: RSWITCH SNIFF 
	Link mode: SLAVE ACCEPT 
	Name: 'arm-64'
	Class: 0x0c0000
	Service Classes: Rendering, Capturing
	Device Class: Miscellaneous, 
	HCI Version: 4.0 (0x6)  Revision: 0x1000
	LMP Version: 4.0 (0x6)  Subversion: 0x220e
	Manufacturer: Broadcom Corporation (15)

```
如果命令不存在，则表示蓝牙控制软件没有安装成功。如果命令执行返回为空，则表示蓝牙设备没有成功驱动，或者没有蓝牙设备。

#### 2. 安装`Passive BLE monitor integration`
可以通过HACS安装：1）在HACS中搜索ble monitor，点击安装；2）安装成功后重启；3）点击配置（Configuration）->设备和服务（Devices&Services）->添加集成（Add integration）进行添加。参考：https://github.com/custom-components/ble_monitor
此集成默认处于Passive模式，被动接收蓝牙设备消息，不会主动扫描设备，所以会更节省设备电量。
#### 2. 将蓝牙设备添加到米家APP
我目前有5个温度计、2个小米门磁、1个青萍门磁，都要先通过手机添加到米家APP，因为这些设备都支持小米的加密协议，添加到米家APP后会生成加密密钥`BLE KEY`，在`Passive BLE monitor integration`的配置中需要使用它。
如果其他非米家蓝牙设备，且广播消息未加密的，可以自动识别并添加到此集成内。
#### 3. 获取蓝牙设备的`encryption_key`
ble_monitor官方给出了4个方法获取蓝牙设备的`encryption_key`，其他的都比较复杂，如果有Windows系统，可以下载[token_extractor.exe](https://github.com/PiotrMachowski/Xiaomi-cloud-tokens-extractor/releases/latest/download/token_extractor.exe)，点击它然后根据提示输入用户名和密码，把输出的信息拷贝出来保存。其中有一行`BLE KEY`，就是之后要使用的`encryption_key`。
#### 4. 配置集成
![](https://custom-components.github.io/ble_monitor/assets/images/configuration_screen.png)
在这个配置页面选择使用的蓝牙设备，在最后一项中根据Mac地址（对照上一步中拷贝下来的设备列表）选择需要配置的设备，然后填入`encryption_key`。按照这个步骤依次把设备配置好。

如果Armbian上没有蓝牙或者蓝牙设备无法驱动，可以使用ESPHome插件来转发蓝牙广播消息。

# 连接Sonoff设备

如果你有Sonoff或者需要接入易微联APP的设备，可以安装插件：https://github.com/AlexxIT/SonoffLAN ，安装方法和其他一样，安装成功后在主目录下的`configuration.yaml`

```yaml
sonoff:
    username: xxx
    password: xxx
```
HA会在启动的时候扫描此文件和其他配置项，加载相应的集成。这个配置告诉HA要加载1个叫sonoff的集成。

# 连接控制涂鸦设备
涂鸦是智能家居行业比较大的解决方案提供商，在国内外都有很大的市场，网上买到的智能设备可能是接入涂鸦智能APP的。目前有2种方法，对于一般的设备，可以使用HA官方的Tuya集成，配置方法比较复杂，可以参考涂鸦的[官方文档](https://developer.tuya.com/cn/docs/iot/Home-assistant-tuya-intergration?id=Kb0eqjig0utdd) 。
如果官方插件暂时无法支持，可以使用[SonoffLAN](https://github.com/rospogrigio/localtuya) ，重点是使用工具找到相应的DP（Data Point），然后通过此插件修改各种DP的值。

# 连接控制自定义设备
如果是自定义的设备，可以通过MQTT连接到HA来控制。



# Home Assistant备份与恢复
点击配置（Configuration）->插件、备份和Supervisor（Add-ons、Backup&Supervisor）->备份（Backup）