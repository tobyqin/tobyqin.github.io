---
title: Linux中的任务管理器
categories: [Tech]
tags: [Linux, top, shell]
date: 2020-02-12
---

在 Linux 中有一个命令叫`top`，作用和 Windows 下的任务管理器差不多。

<!-- more -->

## top

```
top - 15:43:06 up 3 days, 17:46,  1 user,  load average: 0.00, 0.00, 0.00
Tasks: 159 total,   1 running,  97 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.2 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.1 hi,  0.1 si,  0.0 st
KiB Mem :  1882540 total,   323160 free,   691364 used,   868016 buff/cache
KiB Swap:   941268 total,   896468 free,    44800 used.  1082664 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
21090 root      20   0    8060   3432   2824 R   2.0  0.2   0:00.23 top
    1 root      20   0  157808   5764   4300 S   0.0  0.3  10:06.43 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.64 kthreadd
    3 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 rcu_gp
    4 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 rcu_par_gp
    6 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 kworker/0+
    8 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 mm_percpu+
    9 root      20   0       0      0      0 S   0.0  0.0   0:09.23 ksoftirqd+
   10 root      20   0       0      0      0 I   0.0  0.0   0:22.27 rcu_preem+
   11 root      rt   0       0      0      0 S   0.0  0.0   0:00.64 migration+
   12 root      20   0       0      0      0 S   0.0  0.0   0:00.00 cpuhp/0
   13 root      20   0       0      0      0 S   0.0  0.0   0:00.00 cpuhp/1
```

`top` 命令执行结果的前 5 行为系统整体的统计信息，其所代表的含义如下。

- 第 1 行：系统时间、运行时间、登录终端数、系统负载(三个数值分别为 1 分钟、5 分钟、15 分钟内的平均值，数值越小意味着负载越低)。跟直接敲`uptime`是一样的结果。

- 第 2 行：进程总数、运行中的进程数、睡眠中的进程数、停止的进程数、僵死的进程数。

- 第 3 行：用户占用资源百分比、系统内核占用资源百分比、改变过优先级的进程资源百分比、空闲的资源百分比等。

- 第 4 行：物理内存总量、内存使用量、内存空闲量、作为内核缓存的内存量。

- 第 5 行：虚拟内存总量、虚拟内存使用量、虚拟内存空闲量、已被提前加载的内存量。
- 第 6 行之后：就是任务列表了。

## 如果要对任务排序怎么办？

比如按内存排序，找出最占内存的任务。

1. 可以在`top`命令后面跟上排序的参数，比如：

```
# Mac OCX
top -o MEM
# Others
top -o %MEM
```

2. 也可以使用交互模式。
   1. 直接按组合键：`shift` + `m`
   2. 或者：先按`shift`+`f`，进入列调整视图，用方向键选择你要排序的列，按`s`用这一列排序，回车保存，按`q`回到任务视图。（列视图中还可用空格键来调整要显示的列）
   3. 在主界面还保留了一些快捷排序的快捷键，比如：
      1. M，内存排序，跟`shift+m`一样
      2. N，PID 排序
      3. P，%CPU 排序
      4. T，TIME+排序，CPU 使用时间

另外提一下，MacOSX 里的 top 看上去虽然和 Linux 的差不多，但很多指令是不通用的。

## 如果要查找某些任务怎么办？

1. 用方向键可以滚屏（上下左右都可以，page up down 也可以），人肉搜索
2. `top`后面用管道加`grep`，比如 `top | grep httpd`
3. 用交互模式，按`shift`+`l`(Locate)，然后输入搜索的字符，回车。按`&`搜索下一匹配处。

![image-20200212202516163](https://raw.githubusercontent.com/tobyqin/img/master/image-20200212202516163.png)

## 如果要过滤某些任务怎么办？

比如只显示`root`的任务或者某些`PID`的任务。

1. `top`启动时可以对用户进行过滤，比如 `top -u root`
2. 用`grep`可以解决一些问题，比如 `top | grep root`
3. 用交互模式，按小写`o`然后输入你要过滤的条件，比如`USER=root`, `PID<40` ，`!USER=root`等等，此时大小写是不敏感的，如果按大写`O`大小写就是敏感的。貌似没办法部分匹配，按`=`可以重置过滤条件。

![image-20200212204136480](https://raw.githubusercontent.com/tobyqin/img/master/image-20200212204136480.png)

## 如果要杀掉某些任务怎么办？

直接按`k`就好了，首先会让你输入`PID`，然后再输入`SIG`，回车搞定。

![image-20200212210832677](https://raw.githubusercontent.com/tobyqin/img/master/image-20200212210832677.png)

顺便备注一下`SIG`的参考值：

    HUP     1     终端断线
    INT     2     中断（同 Ctrl + C）
    QUIT    3     退出（同 Ctrl + \）
    TERM    15    终止
    KILL    9     强制终止
    CONT    18    继续（与STOP相反， fg/bg命令）
    STOP    19    暂停（同 Ctrl + Z）

## 这鬼东西还有什么功能？

看文档吧，这鬼东西文档写了好几十页，功能太 TM 多了，两个核心命令：

1. `man top`：在没进入 top 前你想要知道的一切都在这。
2. `?` 或者 `h`：在你进入 top 之后，这两个按键都可以给你交互方面的指导。

如果你想要更接近 UI 的任务管理，试一下`htop`吧，可以上下左右，还有快捷键写在功能旁边，新款的 Linux 都原生支持`htop`。

![image-20200212211906608](https://raw.githubusercontent.com/tobyqin/img/master/image-20200212211906608.png)
