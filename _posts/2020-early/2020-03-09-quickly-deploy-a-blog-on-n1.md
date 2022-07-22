---
title: 在N1上快速部署一个博客
categories: [Tech]
tags: [tips]
date: 2020-03-09
---

前提是你已经[刷了 armbian](https://tobyqin.github.io/posts/2020-02-09/feixun-n1s-road-of-twists-and-turns/)。

<!-- more -->

然后你还需要 docker，一个命令即可。

```
curl -sSL https://get.docker.com | sh
```

接下来一句话就可以搞定 typecho。

```
docker run -d \
--name=typecho \
--restart always \
--mount type=tmpfs,destination=/tmp \
-v /data/typecho:/data \
-e PHP_TZ=Asia/Shanghai \
-e PHP_MAX_EXECUTION_TIME=600 \
-p 90:80 \
80x86/typecho:latest
```

docker 命令里**冒号左边**是本机的路径或者端口，根据情况调整。
