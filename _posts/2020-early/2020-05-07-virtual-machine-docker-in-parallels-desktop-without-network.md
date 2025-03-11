---
title: Parallels Desktop里的虚拟机的Docker无网络
categories: [Tech]
tags: [linux, docker, vmware, macosx]
date: 2020-05-07
---

Parallel Desktop 里装了一个 CentOS，CentOS 里装了一个 Docker，有点像套娃。

<!-- more -->

安装过程很顺利，运行第一个例子也很正常，Hello World 而已。

```bash
curl -fsSL https://get.docker.com/ | sh
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl enable docker
sudo usermod -aG docker $(whoami)
sudo docker run hello-world
```

我想做什么呢？我想用原生的 Docker 来替代 MacOS 上的 Docker。

接下来就是映射本地文件到虚拟机里，配置共享就好了。

![image-20200507220741525](https://image.tobyqin.cn/image-20200507220741525.png)

然后从本地 iTerm 登录到虚拟机的命令行，切换到本地工作目录（被挂载到了`/media/psf/Home/src`）。这样就可以在本地获得一个原生的 Linux Shell，操作的还是项目里的文件。

```
ssh toby@centos-linux
cd /media/psf/Home/src/xmind2testlink/web
docker build -t xmind2testlink .
```

开始用原生的 Docker 打包镜像，发现基础镜像可以拉下来，但是安装 Python 包失败因为没有网络。

```
Step 1/6 : FROM frolvlad/alpine-python3:latest
latest: Pulling from frolvlad/alpine-python3
aad63a933944: Pull complete
071e92db37fc: Pull complete
Digest: sha256:ee37502c33d69a230096c8abcda4f293cc398d1e08d3c3b854375b209ab85fe9
Status: Downloaded newer image for frolvlad/alpine-python3:latest
 ---> dd1e5224fc24
Step 2/6 : RUN mkdir /app
 ---> Running in 1761ff57cd39
Removing intermediate container 1761ff57cd39
 ---> 7b4a6a4e13c4
Step 3/6 : WORKDIR /app
 ---> Running in cbf60703344e
Removing intermediate container cbf60703344e
 ---> 24e13fb03163
Step 4/6 : ADD . /app
 ---> b2b737f9503b
Step 5/6 : RUN pip3 install -r requirements.txt
 ---> Running in c574837c3e7f
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.VerifiedHTTPSConnection object at 0x7fafbcc4b3a0>: Failed to establish a new connection: [Errno -3] Try again')': /simple/flask/
```

在网上搜寻半天，各种配 DNS，改防火墙，改代理，一点效果都没有。最简单的测试办法：

- 在虚拟机里 ping baidu，没问题。
- 在虚拟机的 Docker 里 ping baidu，不行。
- 在虚拟机里 ping 路由或者 ip，没问题
- 在虚拟机里的 Docker 里 ping 路由或者 ip，不行。

说明主机和虚拟机的网络桥接没问题，但是虚拟机和 Docker 之间的网络不通。不管切换什么网络共享方式，都行不通。

![image-20200507222519106](https://image.tobyqin.cn/image-20200507222519106.png)

算了，我打不过你。

我打开 VMWare Workstation，把 CentOS 和 Docker 又装了一遍，上面的命令再跑一遍，行了。

MMP。

## 后记

其实 VMWare 也不是没有坑，它最坑的是需要安装 VMWare Tools 才能访问主机文件。官网的文档经久失修有误导性，便捷的办法就是用 yum 来安装。

```bash
yum install -y open-vm-tools
ls /usr/bin/vmtoolsd # 确认安装完毕
reboot
```

然后本地文件就可以在虚拟机里访问了，被挂载在 `/mnt/hgfs/tobyqin/src/`。

是不是我把 CentOS 和 Docker 再到 PD 里装一遍就好了呢？谁知道呢。
