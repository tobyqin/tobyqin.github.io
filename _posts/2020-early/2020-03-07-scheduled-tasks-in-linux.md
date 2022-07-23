---
title: Linux里的计划任务
categories: [Tech]
tags: [Linux,Cron,Crontab]
date: 2020-03-07
---

cron 是 Linux 内置的计划任务程序。

crontab 是 cron 定期执行任务所需的列表文件，注意通过 crontab 命令来修改。

anacron 可以看做是 cron 的补充程序，可以每月，每周，每天执行某些任务。

<!-- more -->

## cron 服务

cron 服务的守护进程是 crond。

```
启动：service crond start
停止：service crond stop
重启：service crond restart
查看状态：service crond status
重新载入配置：service crond reload
```

在 CentOS7 也可以用 `systemctl start crond` 来管理服务。默认情况下 cron 服务应该是开机自动运行的，如果没有可以 enable 一下。

```
systemctl enable crond
```

## crontab 命令

```sh
crontab -u     # 设定某个用户的cron服务，一般root用户在执行这个命令的时候需要此参数
crontab -l     # 列出某个用户cron服务的详细内容
crontab -r     # 删除某个用户的cron服务，这个命令最没用还容易按错！！！
crontab -e     # 编辑某个用户的cron服务
crontab <file> # 将 <file> 恢复至crontab

# 查看自己的cron设置
crontab -l
# 编辑自己的cron设置
crontab -e
# root想删除toby的cron设置
crontab -u toby -r
```

你也可以直接修改 crontab 的配置文件：

```
系统配置文件：
/etc/crontab
系统级任务，一般为空，如果anacron不存在有可能会把anacron 类似的配置写到这里

用户配置文件：
/var/spool/cron/[user]
比如 /var/spool/cron/toby
```

crontab 文件的内容：

```sh
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# demo task
01 * * * * root echo hello
```

前半部分用于声明环境变量，这四个变量是固定的，但值可以改。后半部分就是具体的任务，建议任务前用#号加以注释，方便以后管理。关于 cron 语法，可以参考其他文档或自行搜索：

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

crontab 配置修改后不用重启服务，Linux 会自动加载最新的改动。每次任务执行完毕后会将执行日志写到 `/var/log/cron`，可以通过 tail 命令排错。

### 注意事项

要经常备份 crontab 文件：

```
crontab -l > ~/mycrontab
```

恢复你的备份：

```
crontab ~/mycrontab
```

环境变量可以在具体命令前加载：

```
0 * * * * . /etc/profile;/bin/sh /var/my.sh
```

定时重启的任务需要 root 权限：

```
0 0 * * * root /sbin/reboot
```

## anacron 命令

anacron 算是 crontab 补充。假如你的服务器因为某些原因关机了，crontab 里配置的任务就错过了，例如你一个月备份一次数据库，刚好要备份那天服务器宕机了，当你重启后这个任务又要重新计算了，因为错过了备份时间。

但如果你把备份任务写到 anacron 里，服务器重启后依然会去执行你的任务。anacron 会通过计算记录文件的时间戳来判断上次任务是否已经执行，anacron 没那么灵活，只能按天，周，月配置任务。

```
anacron 配置文件：
/etc/anacrontab
```

cat 一下这个配置文件：

```
SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1	5	cron.daily		nice run-parts /etc/cron.daily
7	25	cron.weekly		nice run-parts /etc/cron.weekly
@monthly 45	cron.monthly		nice run-parts /etc/cron.monthly
```

可以看到所有放入 `/etc/cron.{daily，weekly，monthly}` 目录中的脚本都会在指定时间执行，而且不用担心服务器万一关机的情况。
