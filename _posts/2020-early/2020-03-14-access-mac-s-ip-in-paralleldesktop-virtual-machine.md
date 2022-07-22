---
title: 在ParallelDesktop虚拟机中访问Mac的IP
categories: [Tech]
tags: [paralledesktop, tips]
date: 2020-03-14
---

假设在 Mac 主机开了一个 http 的服务。

<!-- more -->

```
$ python -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

我们可以找到 PD 的网络设置，看到 DHCP 的地址。

![image-20200314154520597](https://img.tobyqin.cn/image-20200314154520597.png)

这里是 10.211.55.1，那么宿主机就是 2 号位。

![image-20200314154808932](https://img.tobyqin.cn/image-20200314154808932.png)

当然，如果你的机器联网了，也可以用路由器分配的地址。

```
$ ifconfig | grep 192
	inet 192.168.1.3 netmask 0xffffff00 broadcast 192.168.1.255
```

![image-20200314154939087](https://img.tobyqin.cn/image-20200314154939087.png)

最后，如果你知道你的 Mac 的机器名是什么（hostname），也可以用机器名来访问。

![image-20200314155350759](https://img.tobyqin.cn/image-20200314155350759.png)
