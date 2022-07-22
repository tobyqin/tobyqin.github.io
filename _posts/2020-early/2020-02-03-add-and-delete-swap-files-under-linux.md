---
title: Linux 添加和删除 Swap 文件
categories: [Tech]
tags: [linux, swap]
date: 2020-02-03
---

Linux 内核为了提高读写效率与速度，会将文件在内存中进行缓存，Swap 是通过磁盘文件的形式给系统增加虚拟内存的解决方案。

<!-- more -->

所以 Swap 速度肯定比真实内存慢，但是可以让系统可以处理超过自身内存瓶颈的任务。默认情况下，系统会用完物理内存后才用虚拟内存。

## 检查 Swap

先检查一下系统里有没有存在的 Swap 文件，如果返回的信息概要是空的，则表示 Swap 文件不存在。

```bash
swapon -s
```

free 命令可以确定 swap 文件是否在在使用。

```bash
free -h
```

## 创建 Swap

一般 Swap 文件的大小是内存的 2 倍，如果内存 1G，Swap 应该就是 2G。

```bash
fallocate -l 2G /swapfile
```

如果这个命令失败就用`dd`，但是需要计算字节数。

> swap 文件的大小单位为 M。将该值乘以 1024 得到块大小。例如，64MB 的 swap 文件的块大小是 65536。

下面使用 dd 命令来创建 Swap 文件。

```bash
dd if=/dev/zero of=/swapfile bs=1024 count=4194304
```

参数说明：

- if=文件名：输入文件名，缺省为标准输入。即指定源文件。`< if=input file >`
- of=文件名：输出文件名，缺省为标准输出。即指定目的文件。`< of=output file >`
- bs=bytes：同时设置读入/输出的块大小为 bytes 个字节
- count=blocks：仅拷贝 blocks 个块，块大小等于 bs 指定的字节数。

最后，赋予 Swap 文件适当的权限：

```bash
chown root:root /swapfile
chmod 0600 /swapfile
```

## 激活 Swap

创建好 Swap 文件，还需要格式化后才能使用。运行命令：

```bash
mkswap /swapfile
```

激活 Swap ，运行命令：

```bash
swapon /swapfile
```

如果要机器重启的时候自动挂载 Swap ，那么还需要修改 fstab 配置。用 vim 打开 `/etc/fstab` 文件，在其最后添加如下一行：

```bash
/swapfile   swap   swap    defaults 0 0
```

当下一次系统启动时，新的 swap 文件就打开了。

在 Linux 系统中，可以通过查看 `/proc/sys/vm/swappiness` 内容的值来确定系统对 Swap 分区的使用原则。当 `swappiness` 内容的值为 0 时，表示最大限度地使用物理内存，物理内存使用完毕后，才会使用 Swap 分区。当 `swappiness` 内容的值为 100 时，表示积极地使用 Swap 分区，并且把内存中的数据及时地置换到 Swap 分区。 默认值为 0，表示需要在物理内存使用完毕后才会使用 Swap 分区。

```bash
 ## 查看默认的swappiness参数
 cat  /proc/sys/vm/swappiness
 ## 临时修改
 sysctl -w  vm.swappiness=10
 ## 永久修改
 vi + /etc/sysctl.conf
 # 添加 vm.swappiness=10
 ## 让配置生效  sysctl -p
```

## 删除 Swap

先卸载 Swap 分区，后从 fastab 中删除，最后删除文件。

```bash
swapoff /swapfile
# remove swap configuration from /etc/fstab
rm -rf /swapfile
```

## 参考链接

- https://blog.csdn.net/wangjunjun2008/article/details/50681115
- https://www.cnblogs.com/operationhome/p/10571166.html
