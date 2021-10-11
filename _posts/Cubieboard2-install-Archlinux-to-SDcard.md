---

title: Cubieboard2 安装 Archlinux 到 SD 卡
author: 梓喵
abbrlink: 11941
date: 2017-03-21 21:31:00
tags:
  - Cubieboard2
cover: 'https://zimiao.moe/images/cover/fkp1ybb1l9n12fo.jpg'
categories: [折腾]
toc: true

---

最近从某宝搞到一块二手 2.5 英寸硬盘，摸出吃灰已久的 CB2 板，用来做家里的低功耗下载机，由于当时买的是 nano 版可刷的系统比较少，所以选择用 SD 卡来装系统。

## 首先格式化 SD 卡

```bash
dd if=/dev/zero of=/dev/mmcblkX bs=1M count=8
```

### 用 fdisk 为 SD 卡分区

```bash
fdisk /dev/mmcblkX
```

## 进入 fdisk 通过以下操作进行分区

1. 按`o`键，删除 SD 卡的所有分区。
2. 现在先按`n`键，然后按`p`键，设置第一个分区， 第一个扇区设置为`2048`，后面的保持默认，一路按回车键。
3. 最后按`w`键保存分区。

## 创建 ext4 格式分区

```bash
mkfs.ext4 /dev/mmcblkX
```

## 挂载分区

```bash
mkdir mnt
mount /dev/mmcblkX mnt
```

### 下载并解压系统文件

```bash
wget https://mirrors.ustc.edu.cn/archlinuxarm/os/ArchLinuxARM-armv7-latest.tar.gz
bsdtar -xpf ArchLinuxARM-armv7-latest.tar.gz -C mnt
sync
```

### 安装 U-Boot 引导程序

```bash
wget https://mirrors.ustc.edu.cn/archlinuxarm/os/sunxi/boot/cubieboard2/u-boot-sunxi-with-spl.bin
dd if=u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8
wget https://mirrors.ustc.edu.cn/archlinuxarm/os/sunxi/boot/cubieboard2/boot.scr -O mnt/boot/boot.scr
umount mnt
sync
```

最后把 SD 卡插上，连接 hdmi 线，通上电源，等待进入系统。

## ssh用户和密码

普通用户：

- user: alarm
- password: alarm

管理员：

- user: root
- password: root
