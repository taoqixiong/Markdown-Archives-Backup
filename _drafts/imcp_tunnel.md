---

title: 搭建 IMCP 隧道
author: 梓喵
abbrlink: 11725
date: 2018-07-01 00:23:00
tags:

---

## 介绍


## 源码
```bash
https://github.com/friedrich/hans
```
## 编译

安装必要运行库:
```bash
apt-get install gcc g++ make git
```
编译：
```bash
cd hans
make
```
## 使用
### 服务端
#### Windows
[下载地址](https://sourceforge.net/projects/hanstunnel/files/windows/hans-win32-experimental-7d6c1dc290.zip)
```bash
hans.exe -s <服务器IP> -p <密码>
```
#### Linux
```bash
./hans -s <服务器IP> -p <密码>
```
### 客户端
服务端和客户端是共用的
#### Windows
```bash
hans.exe -c <服务器IP> -p <密码>
```
#### Linux
```bash
./hans -c <服务器IP> -p <密码>
```