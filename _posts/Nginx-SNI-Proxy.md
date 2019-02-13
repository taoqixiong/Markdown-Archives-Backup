---
title: 使用 Nginx 做 SNI 反代
author: 梓喵
tags:
  - Nginx
  - 折腾
  - SNI
categories: []
abbrlink: 7056
date: 2017-10-08 20:04:00
---
从 Nginx 1.11.5 版本开始支持用做 SNI 反代，使用 Nginx 做 SNI 反代比用 SNI Proxy 配置起来更简单、更稳定。

### 安装依赖
```bash
sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev openssl
```
### 编译并安装 Nginx
获取 Nginx 源码并安装
```bash
wget https://nginx.org/download/nginx-1.12.1.tar.gz
tar zxvf nginx-1.12.1.tar.gz
cd nginx-1.12.1
./configure --with-http_v2_module \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-stream \
--with-stream_ssl_preread_module \
--with-stream_ssl_module
make
sudo make install
```
### 配置
在 nginx.conf 的顶部添加
```bash
stream {
    server {
        listen 443;
        ssl_preread on;
        resolver 8.8.8.8;
        proxy_pass $ssl_preread_server_name:$server_port;
    }
}
```
### 启动 Nginx
```bash
sudo /usr/local/nginx/sbin/nginx
```