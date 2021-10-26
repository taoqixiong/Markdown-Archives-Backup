---

title: OpenWRT 添加 4G/5G 模块支持
author: 梓喵
abbrlink: 35234
date: 2021-10-26 14:32:00
tags:
  - OpenWRT
cover: 'https://zimiao.moe/images/cover/C8qY84q4gVNyB6ML.jpg'
categories: [折腾]
toc: true

---

## 前言

本人使用的是`SIMCom SIM8202G-M2`5G模块测试`QMI`和`RNDIS`模式，在内核版本`5.4.150`是已经支持`SIM820X-M2`无需修改内核

## Openwrt 内核设置

需要预先选择需要编译的设备，再设置内核

```bash
make kernel_menuconfig
```

选择以下选项

```bash
Device Drivers --->
 [*] USB support --->
  <*> USB Modem (CDC ACM) support
  <*> USB Serial Converter support --->
   [*] USB Generic Serial Driver
   <*> USB Qualcomm Serial modem
   <*> USB driver for GSM and CDMA modems
 [*] Networking support --->
  <*> USB Network Adapters --->
   <*> Option USB High Speed Mobile Devices
   <*> Multi-purpose USB Networking Framework
    <*> USB-to-WWAN Driver for Sierra Wireless modems
    <*> CDC Ethernet support (smart devices such as cable modems)
```

## Openwrt 软件包设置

```bash
make menuconfig
```

根据自己需求选择以下选项，可同时选择

### RNDIS

软件包名

```bash
usb-modeswitch
kmod-usb-net-rndis
```

软件包所在位置

```bash
Kernel modules --->
 USB Support --->
  <*> kmod-usb2
  <*> kmod-usb3
  <*> kmod-usb-net
   <*> kmod-usb-net-rndis
Utilities --->
 <*> usb-modeswitch
```

### QMI

软件包名

```bash
usb-modeswitch
kmod-mii
kmod-usb-net
kmod-usb-wdm
kmod-usb-net-qmi-wwan
uqmi
```

软件包所在位置

```bash
Kernel modules --->
 USB Support --->
  <*> kmod-usb2
  <*> kmod-usb3
  <*> kmod-usb-net
   <*> kmod-usb-net
   <*> kmod-usb-wdm
   <*> kmod-usb-net-qmi-wwan
 Network Devices --->
  <*> kmod-mii
Network --->
 WWAN --->
  <*> uqmi
LuCI --->
 5. Protocols --->
  <*> luci-proto-qmi
Utilities --->
 <*> usb-modeswitch
```

### NCM

NCM 模式暂时未测试，软件包设置来自 OpenWRT Wiki

软件包名称

```bash
kmod-usb-serial
kmod-usb-serial-option
kmod-usb-serial-wwan
```

软件包所在位置

```bash
Kernel modules --->
 USB Support --->
  <*> kmod-usb2
  <*> kmod-usb3
  <*> kmod-usb-serial
   <*> kmod-usb-serial-option
   <*> kmod-usb-serial-wwan
LuCI --->
 5. Protocols --->
  <*> luci-proto-ncm
```

## 编译

跟正常编译 OpenWRT 步骤进行，这里不过多赘述

## Luci 设置

### RNDIS 设置

需要先把模块设置成`RNIDS`模式，使用`AT`指令

```bash
AT+CUSBCFG=usbid,1e0e,9011
# 设置接入点(电信ctnet/联通3gnet/移动cmnet) 
AT+CGDCONT=1,"ip","3gnet"
# 重启模块
AT+CFUN=1,1
```

在`OpenWRT`的系统是日志中找到以下信息，证明模块运行在`RNDIS`模式下

```bash
rndis_host 1-1:1.0 usb0: register 'rndis_host' at usb-0000:00:14.0-1, RNDIS device, xx:xx:xx:xx:xx:xx
```

在`网络-接口`选项添加新接口

![](https://zimiao.pages.dev/35234/posts_35234_p0.png)

使用`DHCP 客户端`协议，通常选择`以太网适配器: "usbx"`

![](https://zimiao.pages.dev/35234/posts_35234_p1.png)

在新建接口的`防火墙设置`中把接口放进`wan`里面，保存&应用设置

### QMI 设置

模块默认是使用`qmi`模式，若不是可以使用以下`AT`指令切换

```bash
AT+CUSBCFG=usbid,1e0e,9001
```

在`OpenWRT`的系统是日志中找到以下信息，证明模块运行在`QMI`模式下

```bash
qmi_wwan 1-1:1.5 wwan0: register 'qmi_wwan' at usb-xxxx:xx:xx.x-x, WWAN/QMI device, xx:xx:xx:xx:xx:xx
```

并且在`/dev`中找到`cdc-wdmx`，说明模块已经被识别到

在`网络-接口`选项添加新接口

![](https://zimiao.pages.dev/35234/posts_35234_p2.png)

使用`QMI 蜂窝`协议

![](https://zimiao.pages.dev/35234/posts_35234_p3.png)

`调制解调器节点`填写找到的`/dev/cdc-wdmx`，`APN`根据 SIM 卡的运营商填写，其他无需填写

![](https://zimiao.pages.dev/35234/posts_35234_p1.png)

在新建接口的`防火墙设置`中把接口放进`wan`里面，保存&应用设置
有时无法拨号，可以尝试通过结束`uqmi`进程`killall uqmi`，或者重启模块

在概览网络处的`IPv4 WAN 状态`看到已经获取到 IP 说明已经能拨号成功了

### NCM 设置

手头暂时没支持的模块进行测试

## 参考来源

- <https://openwrt.org/docs/guide-user/network/wan/wwan/ltedongle>
- <https://openwrt.org/docs/guide-user/network/wan/wwan/ethernetoverusb_rndis>
- <https://openwrt.org/docs/guide-user/network/wan/wwan/ethernetoverusb_ncm>
