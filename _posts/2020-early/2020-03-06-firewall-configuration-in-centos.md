---
title: CentOS里的防火墙配置
categories: [Tech]
tags: [linux, centos, firewall]
date: 2020-03-06
---

CentOS 6 以前，内置的防火墙是 iptables，升级到 7 以后就变成了 firwalld。

<!-- more -->

## iptables

1.打开、关闭、重启防火墙

```sh
chkconfig iptables on      #开启防火墙(重启后永久生效)
chkconfig iptables off     #关闭防火墙(重启后永久生效)

service iptables start     #开启防火墙(即时生效，重启后失效)
service iptables stop      #关闭防火墙(即时生效，重启后失效)

service iptables restartd  #重启防火墙
```

2.查看打开的端口

```sh
/etc/init.d/iptables status
```

3.打开某个端口(以 8080 为例)

```sh
# 开启端口
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# 保存并重启防火墙
/etc/rc.d/init.d/iptables save
/etc/init.d/iptables restart
```

4.打开`49152~65534`之间的端口

```sh
iptables -A INPUT -p tcp --dport 49152:65534 -j ACCEPT
```

同样，这里需要对设置进行保存，并重启防火墙。

5.配置修改方式

我们还可以通过修改`/etc/sysconfig/iptables`文件的方式开启端口，如下

```sh
vi /etc/sysconfig/iptables
```

然后在文件中增加一行

```sh
-A RH-Firewall-1-INPUT -m state –state NEW -m tcp -p tcp –dport 8080 -j ACCEPT
```

参数说明:

- –A 参数就看成是添加一条规则
- –p 指定是什么协议，我们常用的 tcp 协议，当然也有 udp，例如 53 端口的 DNS
- –dport 就是目标端口，当数据从外部进入服务器为目标端口
- –sport 数据从服务器出去，则为数据源端口使用
- –j 就是指定是 ACCEPT -接收 或者 DROP 不接收

## firewalld

Centos7 默认安装了 firewalld，如果没有安装的话，可以使用 `yum install firewalld firewalld-config`进行安装。

1.启动、关闭、重启防火墙

```sh
systemctl start firewalld         # 启动,
systemctl enable firewalld        # 开机启动
systemctl stop firewalld          # 关闭
systemctl disable firewalld       # 取消开机启动

firewall-cmd --reload             # 更新规则，重启防火墙
firewall-cmd --complete-reload    # 更新规则，重启服务
```

2.查看状态

```sh
systemctl status firewalld
#或者
firewall-cmd --state
```

3.查看和管理区域 zone

```sh
# 查看当前配置的区域
firewall-cmd --get-active-zones

# 查看指定接口所属区域
firewall-cmd --get-zone-of-interface=eth0

# 设置默认接口区域，无需重启，立即生效
firwalld-cmd --set-default-zone=public

# 将接口添加至public区域，需要重启防火墙
firewall-cmd --zone=public --add-interface=eth0 --permanent

# 永久删除pubic里的接口
firewall-cmd --zone=public --permanent --remove-interface=eth0

# 查看public区域开放的端口
firewall-cmd --zone=public --list-ports
```

Firewall 能将不同的网络连接归类到不同的信任级别，Zone 提供了以下几个级别

- drop: 丢弃所有进入的包，而不给出任何响应
- block: 拒绝所有外部发起的连接，允许内部发起的连接
- public: 允许指定的进入连接
- external: 同上，对伪装的进入连接，一般用于路由转发
- dmz: 允许受限制的进入连接
- work: 允许受信任的计算机被限制的进入连接，类似 workgroup
- home: 同上，类似 homegroup
- internal: 同上，范围针对所有互联网用户
- trusted: 信任所有连接

  4.恐慌模式：拒绝所有包

panic 本意是恐慌，如果服务器遭受攻击时可以打开恐慌模式来决绝所有进包和出包，也称为“禁行模式”。但是已经建立的连接不会被强制断开，只是无法通信了而已。注意，如果你是 ssh 连接上去的话，一旦打开恐慌模式就失去和服务器的连接。

```sh
# 打开恐慌模式，拒绝所有包
firewall-cmd --panic-on
# 关闭恐慌模式
firewall-cmd --panic-off
# 查看恐慌模式状态
firwalld-cmd --query-panic
```

7.防火墙规则管理（记得重启防火墙）

```sh
# 允许http和https服务
firewall-cmd --permanent --zone=external --add-service=http
firewall-cmd --permanent --zone=external --add-service=https

# 移除smtp服务
firewall-cmd --zone=public --remove-service=smtp

# 允许指定端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent

# 打开指定端口区域
firewall-cmd --zone=public --add-port=5000-6000/tcp --permanent

# 禁封 IP
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='222.222.222.222' reject"

# 禁封网段
firewall-cmd --permanent --zone=public --new-ipset=blacklist --type=hash:net
firewall-cmd --permanent --zone=public --ipset=blacklist --add-entry=222.222.222.0/24
```

##### 过滤规则

- source: 根据源地址过滤
- interface: 根据网卡过滤
- service: 根据服务名过滤
- port: 根据端口过滤
- icmp-block: icmp 报文过滤，按照 icmp 类型配置
- masquerade: ip 地址伪装
- forward-port: 端口转发
- rule: 自定义规则

其中，过滤规则的优先级遵循如下顺序

1. source
2. interface
3. firewalld.conf
