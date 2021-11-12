---

title: 模拟 JoyControl 刷 Amiibo
author: 梓喵
abbrlink: 38470
date: 2020-05-17 17:14:30
tags:
  - Nintendo Switch
cover: 'https://zimiao.moe/images/cover/rv2spucgfqab95n.jpg'
categories: [折腾]
toc: true

---

## 前言

在某天上班摸鱼刷推的时候看到一条推

![](https://zimiao.pages.dev/38470/posts_38470_p0.png)

原推链接：https://bit.ly/2z0m09k

在假期时候试了下还真能用，就记录一下用法

`注意：这里不分享 amiibbo 的 bin 文件，请自行使用搜索引擎寻找`

## 准备 

首先需要准备一台带有蓝牙的 Windows/Linux 的 PC 或者树莓派4/Zero，Windows 用户需要使用`VirtualBox`来安装桌面版的 Ubuntu 版或其他发行版本，Linux 用户则无需安装虚拟机，暂时不支持 MacOS 用户，WSL/WSL2 均不支持

## VirtualBox 虚拟机设置

没有安装过桌面版 Linux 需要新增一台新的虚拟机，在Windows 设置中开启蓝牙，在虚拟机的设置中找到`USB设备`，在`USB设备筛选器`中添加`USB筛选器`找到 PC 上的蓝牙适配器并添加上去。已经安装过桌面版的Linux 可以直接在设置中添加蓝牙适配器

![](https://zimiao.pages.dev/38470/posts_38470_p1.png)

我这里是用的是苹果原装网卡的蓝牙驱动，根据自己的机器找到蓝牙驱动

安装完成并设置完后，检查蓝牙驱动是否安装并启用

## 安装运行环境

以下都是虚拟机用户和 Linux 用户需要的步骤

```bash
sudo apt-get update -qy
sudo apt-get upgrade -qy
sudo apt-get install -qy git python3-pip libglib2.0-dev libhidapi-hidraw0 libhidapi-libusb0 libdbus-1-dev
sudo pip3 install hid aioconsole crc8 dbus-python
```

## 设置蓝牙服务

```bash
sudo sed -i `s|^ExecStart=/usr/lib/bluetooth/bluetoothd.*$|ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=input|g` /lib/systemd/system/bluetooth.service
sudo systemctl daemon-reload
sudo systemctl restart bluetooth
```

## 拉取源码

最新的源码已经移除模拟nfc的功能，若想使用这个功能需要回退到支持模拟nfc的版本

```bash
git clone https://github.com/mart1nro/joycontrol ~/joycontrol
# 需要模拟nfc功能，执行下面命令回退到支持模拟nfc的版本
git reset --hard bf2e7e5
```

## 连接

这里拿动森来举例，首先用正常的手柄移动到狸端机，然后断开正常手柄的连接，在虚拟机中运行下面的命令

```bash
cd ~/joycontrol
sudo python3 ./run_controller_cli.py PRO_CONTROLLER
```

后面可以选择模拟不同的手柄：`JOYCON_R`、`JOYCON_L`和`PRO_CONTROLLER`

![](https://zimiao.pages.dev/38470/posts_38470_p2.png)

出现`Waitting for Switch to connect..`说明启动成功，稍等片刻，NS 会连接上模拟手柄，如果长时间未连接上，多次尝试重新运行命令或者重启模拟器。

## 使用

控制方法是通过输入文字来简单控制，但也可以模拟摇杆，暂时还没研究过

- `a` 对应手柄A键
- `b` 对应手柄B键
- `x` 对应手柄X键
- `y` 对应手柄Y键
- `up` 对应手柄方向键上
- `down` 对应手柄方向键下
- `left` 对应手柄方向键左
- `right` 对应手柄方向键右

刷 Amiibo 方法，先把想要刷的 Amiibo 的 bin 文件放到虚拟机的文件夹内，运行下面的命令

```bash
amiibo /path/to/amiibo.bin
```

把后面替换成你要刷的 Amiibo 的所在的文件夹和文件名
