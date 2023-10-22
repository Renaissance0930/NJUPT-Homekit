# NJUPT-Homekit
南邮寝室智能家居合集
[TOC]



### 准备工作

一台路由器(构建内网环境)

一个能运行docker的东西，如Linux开发板，软路由，NAS，树莓派，服务器等(建议1G RAM以上)

一些你需要集成的智能家居设备，如台灯，空调遥控器，音箱，传感器等(不限品牌协议，也可以**手搓**)

------

### 环境搭建

前期初始化过程省略

进入开发板命令行，输入一键安装CasaOS命令

```
curl -fsSL https://get.casaos.io | sudo bash
```

输入开发板IP地址进入CasaOS，IP地址可以在路由器后台查到

我这里是

```
192.168.1.11
```

![](https://github.com/Renaissance0930/NJUPT-Homekit/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-10-22%20150139.png)

点击App Store进入软件目录

下拉找到Home Assistant并安装，等待安装完成后打开，进行初始化操作

### 添加智能家居设备

通常有智能家居设备接入Home Assistant所在的局域网时，Home Assistant会自动识别新设备并显示在通知栏中，也可以手动接入

#### 米家

打开Home Assistant，进入主界面选择 **配置>设备与服务>添加集成**，在搜索框中搜索 **xiaomi**，在结果中点击 **Xiaomi**，选择 **Xiaomi Miio**

![](https://github.com/Renaissance0930/NJUPT-Homekit/blob/main/2.png)

输入米家账号密码即可添加绑定米家账号的米家设备

#### Homekit

添加Homekit设备的方法和米家类似，主要介绍如何将所有绑定在Home Assistant的设备添加进Apple家庭中

在添加集成界面搜索**Apple**，在结果中点击**HomeKit Bridge**，将所有域都勾选上(默认全选)，点击提交，选择区域完成后会在通知栏会出现一个二维码

![](https://github.com/Renaissance0930/NJUPT-Homekit/blob/main/3.png)

打开Apple设备上的家庭，点击右上角的加号选择添加设备，扫描通知栏的二维码跟随软件指引即可将所有设备集成到Apple家庭中

#### 其他协议

和米家类似，理论上市面上所有协议都支持

### 本文重点：使用ESP8266手工制作传感器等设备

#### 准备工作

ESP8266开发板一个

传感器可选：

光照传感器：TSL2591
温湿度气压传感器：BME280
紫外线传感器：LTR390
可燃气体传感器：SGP30

杜邦线若干

#### 安装ESP固件编辑刷写工具

在App Store内找到添加软件源

![](https://github.com/Renaissance0930/NJUPT-Homekit/blob/main/1.png)

在输入框内添加以下链接

```
https://casaos-appstore.paodayag.dev/coolstore.zip
```

更新完软件目录后，在App Store内找到**EspHome**并安装

#### 固件编写

打开EspHome选择**NEW DEVICE>CONTINUE**，给设备起名>SKIP

在新建的设备配置上点击EDIT

清空所有代码粘贴下方的代码(注意更改WiFi信息)

```
esphome:
  name: my-esp

esp8266:
  board: nodemcuv2

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:
  password: "1272645ecfce18cb0a23b93e0cbd80ec"

wifi:
  #在下方引号内输入WiFi名称
  ssid: "123"
  #在下方引号内输入WiFi密码
  password: "12345678"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "My-Esp Fallback Hotspot"
    password: "ah6ooaqooNr9"

captive_portal:


i2c:
  sda: D2
  scl: D1
  scan: True


output:
  - platform: esp8266_pwm
    pin: GPIO2
    frequency: 200Hz
    id: led_onboard
    inverted: True

light:
  - platform: monochromatic
    name: "LED On Board"
    output: led_onboard
 
sensor:
  - platform: bme280
    temperature:
      name: "BME280 Temperature"
      oversampling: 16x
    pressure:
      name: "BME280 Pressure"
    humidity:
      name: "BME280 Humidity"
    address: 0x76
    update_interval: 5s
  
  - platform: tsl2591
    name: "This little light of mine"
    id: "my_tls2591"
    address: 0x29
    update_interval: 5s
    gain: auto
    device_factor: 53
    glass_attenuation_factor: 14.4
    visible:
      name: "TSL2591 visible light"
    infrared:
      name: "TSL2591 infrared light"
    full_spectrum:
      name: "TSL2591 full spectrum light"
    calculated_lux:
      id: i_lux
      name: "TSL2591 Lux"
  
  - platform: ltr390
    uv:
      name: "UV Index"
    light:
      name: "Light"
    address: 0x53
    update_interval: 5s
  
  - platform: sgp30
    eco2:
      name: "eCO2"
      accuracy_decimals: 1
    tvoc:
      name: "TVOC"
      accuracy_decimals: 1
    store_baseline: yes
    address: 0x58
    update_interval: 1s
```

SAVE保存后点击INSTALL，选择Manual download等待固件编译，根据设备CPU性能可能需要几分钟，编译完成后会自动下载到电脑

#### 硬件连接

```
BME280              ESP8266
VCC   ------------ 3.3V
GND   ------------ GND
SDA   ------------ D2
SCL   ------------ D1

TSL25911            ESP8266
VIN   ------------ 3.3V
GND   ------------ GND
SCL   ------------ D1
SDA   ------------ D2

LTR390
VIN   ------------ 3.3V
GND   ------------ GND
SCL   ------------ D1
SDA   ------------ D2
```



#### 固件烧录

将ESP8266连接到电脑上，打开固件烧录软件**ESPHome-Flasher-1.4.0-Windows-x64.exe**，Serial port选择ESP8266所在端口(一般就一个选项)

点击**Firmware**旁边的**Browse**，找到刚刚下载的编译好的固件，点击**Flash ESP**，即可将固件烧录进开发板，随后开发板会自动联网，随后在日志栏可以看到输出的网络和传感器信息

#### 接入Home Assistant

主界面选择 **配置>设备与服务>添加集成**，在搜索框中搜索 **ESPHome**，点击**ESPHome**进入配置

输入esp8266的IP地址，可以在路由器后台找到，点击提交即可将所有传感器接入Home Assistant

#### DIY空调遥控器

具体可参考我仓库的另一个项目：[南邮寝室空调控制](https://github.com/Renaissance0930/NJUPT-AC-IRcontrol)（与本项目的ESP传感器互斥，需要重新购买一块ESP8266）

### CasaOS进阶

在Bilibili上有很多关于CasaOS玩法的介绍，在CasaOS的App Store中有Alist，Jellyfin，内网穿透等很多应用，在图形化界面一键安装到docker，避免了命令行界面的繁琐

### Home Assistant进阶

在电脑网页上查看各个设备的信息会非常不方便，Home Assistant在手机上也有相应的APP，

苹果设备可以在自带APP Store上搜索Home Assistant下载，进入软件只需要输入Home Assistant的IP地址即可完成绑定

安卓设备则需要在Google Play商店内搜索Home Assistant下载，使用方法和苹果设备相同
