---
title: 一行代码停止或删除所有Docker容器
categories: [Tech]
tags: [docker, container]
date: 2020-02-06
---

一行代码就可以停止或者删除所有的 [Docker](http://www.docker.io/) 容器。

<!-- more -->

```sh
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```
