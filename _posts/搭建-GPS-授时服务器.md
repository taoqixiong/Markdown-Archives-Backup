---

title: 搭建 GPS 授时服务器
author: 梓喵
date: 2025-03-12 22:00:00
tags: 
  - luckfox
  - linux
cover: 'https://pic.zimiao.moe/cover/UmlnJ32AhgoSSsW.jpg'
categories: [折腾]
toc: true
abbrlink: 14454

---

# 前言

最近入坑了~~无限垫~~，玩 ft8 这类数字无线电需要比较精确的时间，就想到搞个 GPS 授时服务器，于是就开始折腾，一开始选择树莓派 zero 2w 作为主机，结果发现开发板只有1个 uart 接口，后来翻找手头的开发板发现辛狐的 pico 系列有多个 uart 接口，并且官方有很好的支持，修改起来比较简单就选用作为主机，最后成本还不超过100元。

# 硬件及接线方式

这里选用的 Luckfox Pico Plus ，可选用幸狐的其他开发板，设置根据开发板不同设置会有所不同，系统使用的是 buildroot ，GPS模块选用的是 ATGM336H 

这里引用官网的引脚图
![LuckFox Pico Plus 引脚图](https://wiki.luckfox.com/zh/assets/images/LUCKFOX-PICO-PLUS-GPIO-b2199af217ec733d6e12695d0e79cd60.jpg)

| **开发板** | **GPS模块** |
|:-------:|:---------:|
| Pin 36 | VCC   |
| Pin 8  | GND   |
| Pin 7  | RX    |
| Pin 6  | TX    |
| Pin 9  | PPS   |

# 搭建编译环境和获取Luckfox SDK

使用系统为 Ubuntu 22.04.5

```bash
sudo apt update

sudo apt-get install -y git ssh make gcc gcc-multilib g++-multilib module-assistant expect g++ gawk texinfo libssl-dev bison flex fakeroot cmake unzip gperf autoconf device-tree-compiler libncurses5-dev pkg-config bc python-is-python3 passwd openssl openssh-server openssh-client vim file cpio rsync
```

```
# 国外网络
git clone https://github.com/LuckfoxTECH/luckfox-pico.git

# 国内网络
git clone https://gitee.com/LuckfoxTECH/luckfox-pico.git
```

具体编译方式可以参考官方wiki，本文只介绍需要修改的步骤
- https://wiki.luckfox.com/zh/Luckfox-Pico/Luckfox-Pico-RV1103/Luckfox-Pico-Plus-Mini/Luckfox-Pico-SDK

# 修改设备树

Luckfox Pico Plus 设备树文件位置`sysdrv/source/kernel/arch/arm/boot/dts/rv1103g-luckfox-pico-plus.dts`

首先需要再设备树根节点添加

```
	pps {
		compatible = "pps-gpio";
		pinctrl-names = "default";
		gpios = <&gpio1 RK_PD2 GPIO_ACTIVE_HIGH>;
		status = "okay";
	};
```

添加`GPIO`的部分

```
/**********GPIO**********/
&pinctrl {
	gpio1-pd2 {
		gpio1_pd2:gpio1-pd2 {
			rockchip,pins =	<2 RK_PD2 RK_FUNC_GPIO &pcfg_pull_none>;
		};
	};
};
```

这里使用`UART 4`接口，找到`&uart4`部分修改成以下

```
&uart4 {
	status = "okay";
	pinctrl-names = "default";
	pinctrl-0 = <&uart4m1_xfer>;
};
```

完整设备树文件

```
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2022 Rockchip Electronics Co., Ltd.
 */

/dts-v1/;

#include "rv1103.dtsi"
#include "rv1106-evb.dtsi"
#include "rv1103-luckfox-pico-ipc.dtsi"

/ {
	model = "Luckfox Pico Plus";
	compatible = "rockchip,rv1103g-38x38-ipc-v10", "rockchip,rv1103";
	
	pps {
		compatible = "pps-gpio";
		pinctrl-names = "default";
		gpios = <&gpio1 RK_PD2 GPIO_ACTIVE_HIGH>;
		status = "okay";
	};
};

/**********GPIO**********/
&pinctrl {
	gpio1-pd2 {
		gpio1_pd2:gpio1-pd2 {
			rockchip,pins =	<2 RK_PD2 RK_FUNC_GPIO &pcfg_pull_none>;
		};
	};
};

/**********SFC**********/
&sfc {
	status = "okay";
	flash@0 {
		compatible = "spi-nand";
		reg = <0>;
		spi-max-frequency = <75000000>;
		spi-rx-bus-width = <4>;
		spi-tx-bus-width = <1>;
	};
};

/**********SDMMC**********/
&sdmmc {
	max-frequency = <50000000>;
	no-sdio;
	no-mmc;
	bus-width = <4>;
	cap-mmc-highspeed;
	cap-sd-highspeed;
	disable-wp;
	pinctrl-names = "default";
	pinctrl-0 = <&sdmmc0_clk &sdmmc0_cmd &sdmmc0_det &sdmmc0_bus4>;
	status = "okay";
};

/**********ETH**********/
&gmac {
	status = "okay";
};

/**********USB**********/
&usbdrd_dwc3 {
	status = "okay";
	dr_mode = "peripheral";
};

/**********SPI**********/
/* SPI0_M0 */
&spi0 {
	status = "disabled";
	spidev@0 {
		spi-max-frequency = <50000000>;
	};
  fbtft@0 {
    spi-max-frequency = <50000000>;
  };
};

/**********I2C**********/
/* I2C3_M1 */
&i2c3 {
	status = "disabled";
	clock-frequency = <100000>;
};

/* I2C0_M2 */
&i2c0 {
	status = "disabled";
	clock-frequency = <100000>;
};

/**********UART**********/
/* UART3_M1 */
&uart3 {
	status = "disabled";
};

/* UART4_M1 */
&uart4 {
	status = "okay";
	pinctrl-names = "default";
	pinctrl-0 = <&uart4m1_xfer>;
};

/**********PWM**********/
/* PWM1_M0 */
&pwm1 {
	status = "disabled";
};
```

# 内核设置

开启以下选项

```
Device Drivers  --->
 <*> PPS support  --->
  <*>   PPS client using GPIO
```

# Buildroot 设置

开启和关闭以下选项，这里使用`chrony`代替`ntp`，就关闭掉`ntp`，`gpsd`里选项`NMEA2000`和`Navcom`为`NMEA`协议可选可不选，本文并未使用该协议

```
Target packages  --->
 Hardware handling  --->
  [*] gpsd  --->
   -*-   Navcom
   [*]   NMEA2000
  [*] minicom
  [*] pps-tools
 Networking applications  --->
  [*] chrony
  [ ] ntp
```

# 编译

根据官方的wiki的编译部分编译就行了

# 进入系统设置

通过`ssh`或者`adb`进入系统

- 默认用户：`root`
- 默认密码：`luckfox`

## 修改`gpsd`启动脚本`/etc/init.d/S50gpsd`
- `DEVICES`选项修改为`/dev/ttyS4 /dev/pps0`，这里设置gps模块连接开发板的`uart`和`pps`接口的位置
- 启动命令需添加`-n`，若模块的波特率不是`9600`者需要添加`-s <模块的波特率>`，最后启动命令为`start-stop-daemon -S -q -p $PIDFILE --exec $DAEMON -- -P $PIDFILE $DEVICES -s <模块的波特率> -n && echo "OK" || echo "Failed"`

完整启动脚本

```
#!/bin/sh
#
# Starts the gps daemon.
#

NAME=gpsd
DAEMON=/usr/sbin/$NAME
DEVICES="/dev/ttyS4 /dev/pps0"
PIDFILE=/var/run/$NAME.pid

start() {
        printf "Starting $NAME: "
        start-stop-daemon -S -q -p $PIDFILE --exec $DAEMON -- -P $PIDFILE $DEVICES -s 115200 -n && echo "OK" || echo "Failed"
}
stop() {
        printf "Stopping $NAME: "
        start-stop-daemon -K -q -p $PIDFILE && echo "OK" || echo "Failed"
        rm -f $PIDFILE
}
restart() {
        stop
        start
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart|reload)
        restart
        ;;
  *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
esac

exit $?
```

## 添加`Chrony`配置文件`/etc/chrony.conf`

这里的配置文件关闭了网络ntp获取时间，需要则把`server`前面的`#`去掉，并改为需要使用的ntp服务器

```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
# NTP服务器地址。server可以配置多个
#server ntp.aliyun.com iburst
#server ntp1.aliyun.com iburst
#server time1.cloud.tencent.com iburst
#server cn.ntp.org.cn iburst
#server ntp.ntsc.ac.cn iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
# 允许连接到此服务器同步时间的网段；0.0.0.0/0表示允许所有网段；注释该配置此服务就只能作为NTP客户端，不能作为服务器
allow 0.0.0.0/0

# Serve time even if not synchronized to a time source.
# 当配置的server不可用时，是否使用本地时间同步到客户端
local stratum 10

# Specify file containing keys for NTP authentication.
#keyfile /etc/chrony.keys

# Get TAI-UTC offset and leap seconds from the system tz database.
#leapsectz right/UTC

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
#log measurements statistics tracking

#和服务器相差多少秒内才执行同步。本地时间和服务器时间相差的秒数超过这个时间就不会同步。默认是3秒
maxdistance 94608000.0

# GPS 接口配置
# SHM refclock is shared memory driver, it is populated by GPSd and read by chrony
# it is SHM 0
# refid is what we want to call this source = NMEA
# offset = 0.000 means we do not yet know the delay
# precision is how precise this is. not 1e-3 = 1 millisecond, so not very precision
# poll 0 means poll every 2^0 seconds = 1 second poll interval
# filter 3 means take the average/median (forget which) of the 3 most recent readings. NMEA can be jumpy so we're averaging here
refclock SHM 0 refid NMEA offset 0.000 precision 1e-3 poll 0 filter 3

# PPS 接口配置
# PPS refclock is PPS specific, with /dev/pps0 being the source
# refid PPS means call it the PPS source
# lock NMEA means this PPS source will also lock to the NMEA source for time of day info
# offset = 0.0 means no offset... this should probably always remain 0
# poll 3 = poll every 2^3=8 seconds. polling more frequently isn't necessarily better
# trust means we trust this time. the NMEA will be kicked out as false ticker eventually, so we need to trust the combo
refclock PPS /dev/pps0 refid PPS lock NMEA offset 0.0 poll 3 trust
```

以上配置完成后可以重启系统，也可以重启服务`/etc/init.d/S50gpsd`和`/etc/init.d/S49chrony`

# 测试命令

## 测试 GPS 模块

使用`minicom -D /dev/ttyS4`检查 GPS 模块是否有输出，没有则检查模块接口是否连接正确

可用`gpsmon`或者`cgps`观察 gpsd 是否配置正确

## 测试 PPS 输出

使用`pps /dev/pps0`查看 PPS 接口是否有输出，一般在gps模块没有获取到信号或者没正确接线都会输出`time_pps_fetch() error -1 (Connection timed out)`，正常则输出`source 0 - assert 1741791874.907876850, sequence: 6520 - clear  0.000000000, sequence: 0`

## Chrony 检查 GPS 和 PPS 授时状态

使用`chronyc sourcestats`检查授时状态

```
Name/IP Address            NP  NR  Span  Frequency  Freq Skew  Offset  Std Dev
==============================================================================
NMEA                       15   8    14   -366.405   1426.521    +98ms  5653us
PPS                        15   8   191     +0.008      0.133    +13ns  3178ns
```

手动同步时间`chronyc -a makestep`

# 参考链接
- https://wiki.luckfox.com/zh/Luckfox-Pico/Luckfox-Pico-RV1103/Luckfox-Pico-Plus-Mini/Luckfox-Pico-SDK
- https://forums.luckfox.com/viewtopic.php?t=510
- https://www.cnblogs.com/xiaoko/p/17199281.html
- https://austinsnerdythings.com/2021/04/19/microsecond-accurate-ntp-with-a-raspberry-pi-and-pps-gps/
- https://austinsnerdythings.com/2025/02/14/revisiting-microsecond-accurate-ntp-for-raspberry-pi-with-gps-pps-in-2025/