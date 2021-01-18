title: 搭建 Home Assistant 智能家居环境
author: 梓喵
abbrlink: 38470
date: 2021-01-14 12:26:00
tags:
    - 智能家居
    - Home Assistant
categories: []
cover: 'https://zimiao.moe/images/cover/4YSsvyZBz7EX0By.jpg'

---

# 环境配置

## 首先安装 Docker 环境，已安装可以跳过

这里使用 Ubuntu 系统作为演示，其他系统可以根据官方文档进行安装

- [Debian](https://docs.docker.com/engine/install/debian/)
- [CentOS](https://docs.docker.com/engine/install/centos/)
- [Fedora](https://docs.docker.com/engine/install/fedora/)

移除已安装的 Docker 相关组件，以及安装必要的组件

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

信任 Docker 的 GPG 公钥

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

添加 Docker 软件源，可以把 `download.docker.com` 替换成国内镜像源，根据计算机的 CPU 框架选择合适的命令添加软件仓库

```bash
# x86_64/amd64
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# armhf
sudo add-apt-repository \
   "deb [arch=armhf] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# arm64
sudo add-apt-repository \
   "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

更新源并安装 Docker

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## 修改 Docker 镜像源

在 `/etc/docker` 文件夹内添加 `daemon.json` 文件，并添加以下内容，可以把下面的七牛云镜像源替换成其他的镜像源

```json
{
  "registry-mirrors": ["https://reg-mirror.qiniu.com"]
}
```

添加完成后重启 Docker 服务

```bash
sudo systemctl restart docker
```

# 安装 Home Assistant 环境

拉取 Home Assistant 镜像

```bash
docker pull homeassistant/home-assistant:stable
```

启动容器

```bash
docker run -d --name="home-assistant" --net=host \
       -v /path/to/hass:/config \
       -v /etc/localtime:/etc/localtime:ro \
       homeassistant/home-assistant:stable
```

过段时间打开浏览器 `http://<your_ip:8123>` 就能访问 Home Assistant 的界面

# 参考来源

- <https://docs.docker.com/engine/install/ubuntu/>
- <https://www.home-assistant.io/docs/installation/docker/>
