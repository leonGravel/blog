---
title: 解决docker 容器内访问宿主机“No route to host”的问题
categories: ["Devops"]
tags: ["docker","问题笔记"]
date: 2018-11-04 20:54:41

---

首先确认是否已经关闭防火墙； 请顺序运行以下命令：

<!--more-->

```
nmcli connection modify docker0 connection.zone trusted

systemctl stop NetworkManager.service

firewall-cmd --permanent --zone=trusted --change-interface=docker0

systemctl start NetworkManager.service

nmcli connection modify docker0 connection.zone trusted

systemctl restart docker.service
```

