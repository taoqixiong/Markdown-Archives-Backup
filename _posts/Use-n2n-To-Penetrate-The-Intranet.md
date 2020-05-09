title: 使用 n2n 穿透内网
author: 梓喵
tags:
  - 折腾
categories: []
abbrlink: 61570
cover: 'https://zimiao.moe/images/cover/v15lsuhhyivgiul.jpg'
date: 2018-02-12 15:58:00
---
# 介绍

n2n 是 2008 年 lucaderi 开发的 P2P VPN ，只要有台有公网 ip 的服务器，就能远程访问内网的电脑、路由器或者其他设备。n2n 跟 Zerotier 相类似，然而 Zerotier 的设置方面更容易些，但稳定性取决于官方的服务器，由于国内的互联网环境，Zerotier 经常掉线。虽然 Zerotier 官方提供 Moon 的设置方法，但只能用在 Linux 上，Windows 下没办法是使用 Moon 来连接；n2n 虽然设置起来麻烦一些，但稳定性取决于所选的服务器。

官方提供的源码用的是很老的库，现在编译会报错，我这里用的是 meyerd 修正过的，但两者不能互通，所以如果服务器是用官方的，客户端必须使用官方的源码来编译。另外 n2n 还有 V1 和 V2 版本，这两者也是不能互通的，我这里选择的是 V2 版本， V1 版本编译和设置方面基本相同。

# n2n 源码
## 官方旧版
```bash
https://github.com/ntop/n2n
```
## 修正版
```bash
https://github.com/meyerd/n2n
```


# Linux 编译 n2n
## 安装必要运行库

```bash
apt-get install gcc g++ cmake make libssl-dev
```

## 编译
```bash
mkdir ~/n2n/n2n_v2/build
cd ~/n2n/n2n_v2/build
cmake ..
make
```

# Linux 交叉编译 Windows 版的 n2n
## 安装 Mingw
```bash
apt-get install mingw-w64
```

## 安装 cmake
```bash
apt-get install cmake
```

## 获取 n2n 源码
```bash
git clone https://github.com/meyerd/n2n.git
```

## 修改文件
修改在 n2n_v2 中的 cmake 文件夹中的 CMakeToolchainFileMingw32.cmake 里面的

``` c
SET(CMAKE_C_COMPILER i686-mingw32-gcc)
SET(CMAKE_CXX_COMPILER i686-mingw32-g++)
```

修改为

``` c
SET(CMAKE_C_COMPILER i686-w64-mingw32-gcc)
SET(CMAKE_CXX_COMPILER i686-w64-mingw32-g++)
```

## 开始编译
```bash
mkdir ~/n2n/n2n_v2/build
cd ~/n2n/n2n_v2/build
cmake -DCMAKE_TOOLCHAIN_FILE=../cmake/CMakeToolchainFileMingw32.cmake --build ./ ../
make
```
# 使用
## 服务端
```bash
./supernode -l 端口
```

## Linux 客户端(需要权限)
```bash
./edge -d n2n0 -c 虚拟局域网名 -k 密码 -m 指定物理mac地址(可选) -a 内网IP -l 服务端IP:端口号
```

## Windows 客户端
市面上的 Windows 客户端有两个：一个是 [n2nedgegui](https://sourceforge.net/projects/n2nedgegui/)，另一个是 [n2nguien](http://www.vpnhosting.cz/n2nguien.exe)，前者用的是 V2 版本，后者 V1 和 V2 版本，但这两个用的是官方旧版的内核，如果服务端用的是修正的版本，需要把上面编译好的 edge 替换客户端安装文件夹内的 edge 。

我这里使用的是 n2nguien ，客户端安装文件夹内 V2 版是用 edge2.exe 来命名的，所以替换 V2 版的时候把名字改为 edge2.exe ，V1 版直接替换就行了。
![设置](https://i.loli.net/2018/05/02/5ae98660ae3f0.jpg)
如果使用的是 V2 版，需要在 Advanced 选择 Use n2n v2 。