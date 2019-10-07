title: Linux 开机挂载硬盘
author: 梓喵
tags:
  - Linux
  - 折腾
categories: []
abbrlink: 18671
date: 2017-05-26 00:33:00
cover: 'https://i.loli.net/2019/10/06/qljrxwAaNM5L6Uk.jpg'
---
在 /etc/fstab 中添加
```bash
/dev/sda1 /data1 ext4 defaults 0 0
```