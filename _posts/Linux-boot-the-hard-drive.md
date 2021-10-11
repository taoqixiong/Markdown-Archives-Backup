---

title: Linux 开机挂载硬盘
author: 梓喵
abbrlink: 18671
date: 2017-05-26 00:33:00
tags:
  - Linux
cover: 'https://zimiao.moe/images/cover/shvg4uby4qktktd.jpg'
categories: [折腾]
toc: true

---

在 /etc/fstab 中添加

```bash
/dev/sda1 /data1 ext4 defaults 0 0
```
