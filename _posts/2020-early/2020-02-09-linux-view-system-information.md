---
title: Linux查看系统信息的命令
categories: [Tech]
tags: [Linux, shell]
date: 2020-02-09
---

查看 Linux 系统信息的一些技巧。

<!-- more -->

## 查看系统版本

```sh
uname
uname -a
cat /etc/*-release # 不同的发行版名字不太一样
```

## 查看 CPU 和内存

```sh
cat /proc/cpuinfo
cat /proc/meminfo
free -h
```

## 查看硬盘空间

```sh
du -sh /*
df -h
```

## 查看运行状态

```sh
uptime
22:49:55 up 10 min, 2 users, load average: 0.01, 0.19, 0.18
 # 当前时间+运行时间+当前登录用户数+最近1，5，15分钟的压力，越低越好，最好不要超过1
```

## 查看进程

```sh
ps aux
top
htop # 需要安装
pidof httpd # 查看进程号，一个进程可能有多个进程号
kill 1234 # 杀掉进程号1234
killall httpd # 杀掉一个进程
```

## 查看网络

```sh
ifconfig # 显示网卡IP等
netstat # 显示网络状态
```

## 查看登录信息

```sh
whoami # 当前用户名
id # 当前用户id以及组等信息
who # 当前登录在本机的用户
last # 系统曾经的登录信息
```

## 查看环境变量

```
env
printenv # 功能和env一样
env | sort # 排序
```

## 查看用户信息

```sh
users # 只显示可登录的用户名
groups
cat /etc/passwd # 所有用户
cat /etc/group # 所有组
getent passwd # 等同于 cat /etc/passwd
compgen -u # 只显示 /etc/passwd 的第一列
```

## 查看所有可用命令

```sh
# List all commands that you could run:
compgen -c

# List all aliases:
compgen -a

# List all functions that you could run:
compgen -A function

# Show shell reserved key words:
compgen -k

# Check command location
which [command]
```
