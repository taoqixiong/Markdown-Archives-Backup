title: Cubieboard2 安装 Lxde 桌面
author: 梓喵
tags:
  - Cubieboard2
  - 折腾
categories: []
date: 2017-05-25 12:52:00
---
### 安装显卡驱动
```bash
pacman -S xf86-video-fbdev
```
### 安装基础的 xorg 包
```bash
pacman -S xorg-server xorg-xinit
```
### 安装 LXDM 管理器
```bash
pacman -S lxdm
```
### 设置 LXDM 开机启动：
```bash
systemctl enable lxdm
```
### 安装最小化的LXDE桌面环境：
```bash
pacman -S lxde-common
```
### 安装LXDE Session
```bash
pacman -S lxsession
```
### 安装LXDE面板
```bash
pacman -S lxpanel
```
### 安装窗口管理器
```bash
pacman -S openbox
```
### 安装LXDE环境下的终端程序
```bash
pacman -S lxterminal
```
### 安装LXDE环境下的文件管理器
```bash
pacman -S pcmanfm
```
### 在 ~/.xinitrc 中加入
```bash
exec startlxde
```
### 启动 X11 窗口系统
```bash
startx
```
### 安装常用软件：
```bash
pacman -S fcitx tar leafpad xarchiver firefox
```
### 安装常用字体
```bash
pacman -S ttf-dejavu wqy-zenhei wqy-microhei
```
### 更改时区
```bash
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock -s --localtime
hwclock -w --localtime
```