---

title: Linux编译SDR++和Satdump
author: 梓喵
abbrlink: 23117
date: 2024-09-08 15:00:00
tags:
  - Linux
  - 无线电
cover: 'https://pic.zimiao.moe/cover/zhJLz3GbiWcJxq5F.jpg'
categories: [折腾]
toc: true

---

# SDR++

## 安装编译环境

### 安装软件包

```bash
sudo apt install wget git build-essential cmake libfftw3-dev libglfw3-dev libvolk2-dev libzstd-dev libsoapysdr-dev libairspyhf-dev libairspy-dev libiio-dev libad9361-dev librtaudio-dev libhackrf-dev librtlsdr-dev libbladerf-dev liblimesuite-dev p7zip-full portaudio19-dev libcodec2-dev autoconf libtool xxd libudev-dev udev
```

## 安装 SDR API

### SDRPlay

```bash
wget https://www.sdrplay.com/software/SDRplay_RSP_API-Linux-3.15.2.run
chmod +x SDRplay_RSP_API-Linux-3.15.2.run
sudo ./SDRplay_RSP_API-Linux-3.15.2.run
```

### Perseus SDR

```bash
git clone https://github.com/Microtelecom/libperseus-sdr
cd libperseus-sdr
autoreconf -i
./configure
make -j$(($(nproc) + 1)) V=s
sudo make install
sudo ldconfig
```

##  编译安装

```bash
git clone https://github.com/AlexandreRouma/SDRPlusPlus
cd SDRPlusPlus
mkdir build && cd build
cmake .. -DOPT_BUILD_BLADERF_SOURCE=ON -DOPT_BUILD_LIMESDR_SOURCE=ON -DOPT_BUILD_SDRPLAY_SOURCE=ON -DOPT_BUILD_NEW_PORTAUDIO_SINK=ON -DOPT_BUILD_M17_DECODER=ON -DOPT_BUILD_PERSEUS_SOURCE=ON
make VERBOSE=1 -j$(($(nproc) + 1))
make install
```

## 制作 deb 软件包

```bash
cd ..
sh make_debian_package.sh ./build 'libfftw3-dev, libglfw3-dev, libvolk2-dev, librtaudio-dev, libzstd-dev'
```

## CLI 使用示例

### SDR++ Server

```bash
sdrpp --server --port <port> --addr <ip>
```

# Satdump

## 安装编译环境

```bash
sudo apt install wget git build-essential cmake g++ pkgconf libfftw3-dev libpng-dev libtiff-dev libjemalloc-dev libvolk2-dev libcurl4-openssl-dev libnng-dev librtlsdr-dev libhackrf-dev libairspy-dev libairspyhf-dev libglfw3-dev zenity libzstd-dev libomp-dev ocl-icd-opencl-dev
# 如果使用的是 Intel 核显需要安装 intel-opencl-icd 软件包
sudo apt-get install intel-opencl-icd
```

## 安装 SDR API
参考 SDR++ 的安装 SDR API 步骤

##  编译安装

```bash
git clone https://github.com/SatDump/SatDump
cd SatDump
mkdir build && cd build

# 如果要禁用某些 SDR，可以添加 -DPLUGIN_HACKRF_SDR_SUPPORT=OFF 或类似内容
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr ..
# 如果不需要编译 GUI 可以添加 -DBUILD_GUI=OFF
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DBUILD_GUI=OFF ..

make VERBOSE=1 -j$(($(nproc) + 1))
make install
```

## CLI 使用示例

### 实时解码 GK-2A
```bash
satdump live gk2a_lrit <gk2a_output> --source sdrplay --samplerate 2000000 --frequency 1692140000 --lna_gain 0 --if_gain 20
```
- `--source` 使用的接收设备类型
- `--samplerate` 接收的带宽
- `--frequency` 接收的频率

其他的选项和接收卫星的类型可以在 <https://docs.satdump.org/index.html> 找到

## 参考来源
- <https://github.com/AlexandreRouma/SDRPlusPlus>
- <https://github.com/SatDump/SatDump>
