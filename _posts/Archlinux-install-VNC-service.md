---
title: Archlinux 安装 VNC 服务端
author: 梓喵
tags:
  - Archlinux
  - 折腾
categories: []
abbrlink: 51311
date: 2017-05-29 01:40:00
---
### 安装 tigervnc 
```bash
pacman -S tigervnc
```
### 创建环境和密码文件 
```bash
vncserver
```
### 关闭 VNC 服务 
```bash
vncserver -kill :1
```
### 编辑 xstartup 文件 
把 ~/.vnc/xstartup 文件删除，然后创建新的 xstartup 文件，如果使用 LXDE 作为桌面环境添加以下代码
```bash
#!/bin/sh
export XKL_XMODMAP_DISABLE=1
exec startlxde
```
### 启动 VNC 服务 
分辨率自行修改
```bash
vncserver -geometry 1280x720 -alwaysshared -dpi 96 :1
```