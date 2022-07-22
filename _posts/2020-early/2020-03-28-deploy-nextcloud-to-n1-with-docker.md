---
title: 用Docker部署NextCloud到N1
categories: [Tech]
tags: [docker, n1, nextcloud, nas]
date: 2020-03-28
---

只需要一个命令。

```bash
docker run -d -p 8888:80  --name nextcloud  -v /data/nextcloud/:/var/www/html/ --restart=always   --privileged=true  arm64v8/nextcloud
```

<!-- more -->

如果是部署到 U 盘，可以这样。

```bash
docker run -d -p 8888:80  --name nextcloud  -v /media/udisk/:/var/www/html/ --restart=always   --privileged=true  arm64v8/nextcloud
```
