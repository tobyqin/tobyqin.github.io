---
title: 在Linux或者Mac OSX上查找大文件
categories: [Tech]
tags: [linux, du, tips]
date: 2020-01-30
---

在 Mac OSX 上尚且还有一些图形工具可以帮助查找和清理大文件，在 Linux 只能依靠命令行。其实也不难，这次总结一下，省的下次还去搜索。

<!-- more -->

## 万能的 du

`du` 是 Linux 和 MacOSX 都自带命令行工具，全称是 Disk Usage，这样就好记了。配合两个参数就可以搞定大多数问题。

```shell
-s      Display an entry for each specified file.  (Equivalent to -d 0)
-h      "Human-readable" output.
```

`-s`的意思就是只统计第一层目录，`-h` 就是显示可读性的统计数据，看例子。

```shell
[root@2017127313 ~]# du -sh /*
0       /bin
89M     /boot
0       /dev
39M     /etc
4.0K    /home
16K     /lost+found
4.0K    /media
4.0K    /mnt
4.0K    /opt
0       /proc
603M    /root
13M     /run
0       /sbin
4.0K    /srv
513M    /swapfile
0       /sys
616K    /tmp
1.3G    /usr
487M    /var
```

从根目录开始找最大的目录，然后一层一层递进就可以找到占用最大空间的目录或者文件。

### 限制数量和排序

当目录里文件比较多的时候，我们就要限制返回的条数和排序，比如我只想知道占用空间最大的 5 个目录，通过管道操作符就可以达到目的。

```shell
[root@2017127313 ~]# du -s /usr/* | sort -nr | head -5
598904  /usr/lib
303676  /usr/share
185836  /usr/lib64
124044  /usr/bin
44772   /usr/sbin
```

## 当前目录占用空间

`du` 还可以很方便检查当前目录占用空间。

```sh
[root@2017127313 ~]# du -sh
603M    .
```

## 比较难记的 find

`find` 是非常强大的命令，可以按文件属性进行搜索，比如检索大于 10M 的文件。

```sh
[root@50KVM-2017127313 ~]# find . -size +10M
./blog/.git/objects/pack/pack-fab187cef1cd08d186624f1e5e97e3131b20abc0.pack
./docs/.git/objects/3f/3385a6b098631a8426a720dcd56a9ed7da4183
./docs/PPT/Demo.pptx
```

如上命令是把文件名打印出来了，但文件的细节还是不清楚，这时候你需要加上更多参数。

```sh
find / -size +500M -exec du -h {} \; 2>/dev/null
```

如果还要排序，再加个管道。

```sh
find . -type f -size +100M  -print0 | xargs -0 du -h | sort -nr
```

`find` 权当是备忘吧，具体的参数我是记不住的，推荐还是用简单易懂的`du`。

## 额外的 df

`df`也是磁盘管理常用的工具之一，全称不知道是什么，从文档上看好像是 Disk space available on file system? 需要记住的参数只有一个，就是`-h`，可读性显示。

```shell
[root@2017127313 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        107M     0  107M   0% /dev
tmpfs           117M     0  117M   0% /dev/shm
tmpfs           117M   13M  105M  11% /run
tmpfs           117M     0  117M   0% /sys/fs/cgroup
/dev/vda1       4.9G  3.0G  1.7G  65% /
tmpfs            24M     0   24M   0% /run/user/0
```

这个命令就是用来看磁盘剩余空间的，当然有时候 `-ai`的参数也偶尔会被提到，用来检查`inode`的使用情况。

```shell
[root@017127313 ~]# df -ai
Filesystem     Inodes IUsed  IFree IUse% Mounted on
devtmpfs        27317   393  26924    2% /dev
tmpfs           29942     1  29941    1% /dev/shm
tmpfs           29942   491  29451    2% /run
tmpfs           29942    16  29926    1% /sys/fs/cgroup
/dev/vda1      324480 50300 274180   16% /
tmpfs           29942     2  29940    1% /run/user/0
```

如果`inode`占用到 100%了，你的磁盘就是还有空间也写不进去了，`inode`可以理解为文件的索引区吧，用来存放文件的属性等等，具体的内容会写到 `block` 区，当文件很碎的时候，`block` 可能还没满，但是 `inode`已经满了。

## 免费赠送的 free

文章的最后再送个 `free` 命令吧，这个跟文件系统有一丢丢关系，这个命令是用来显示内存的。

```sh
[root@2017127313 ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           233M         58M         28M          9M        147M        142M
Swap:          511M         21M        490M
```

`free` 是用来显示内存占用情况的，`-h` 一样是人性化显示。注意，这个命令还是看到 swap 分区的使用情况。

一般 Linux 都会配置虚拟内存，也就是用 swap 分区。很早以前内存还是很宝贵的，所以聪明的人类就划分了一部分硬盘来充当二级内存，纵然速度慢点，但是容量更大了能处理的东西就更多了。

`free` 里可以查看 swap 占用情况，但不能清理或者调整它的大小。
