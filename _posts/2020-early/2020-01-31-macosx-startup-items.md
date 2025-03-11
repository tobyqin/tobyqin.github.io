---
title: 管理MacOSX的开机启动项
categories: [Tech]
tags: [macosx, startup]
date: 2020-01-31
---

MacOSX 下的开机项有多乱，需要整理一下。

<!-- more -->

## 系统偏好

在“系统偏好设置”窗口，选择“用户与群组”，进入用户与群组窗口。选择登录项选项卡，再解锁，最后删除开机启动的应用。

## plist 文件

分别在以下 6 个目录中检查是否有与开机程序相关的 plist 文件

1. ~/Library/Preferences/ – （当前用户设置的进程）
2. ~/Library/LaunchAgents/ – （当前用户的守护进程）
3. /Library/LaunchAgents/ – （管理员设置的用户进程）
4. /Library/LaunchDaemons/ – （管理员提供的系统守护进程）
5. /System/Library/LaunchAgents/ – （Mac 操作系统提供的用户进程）
6. /System/Library/LaunchDaemons/ – （Mac 操作系统提供的系统守护进程）

### plist 中主要的字段和它的含义

- `Label <required, nsstring=“”>` Launchd 中的一个唯一标识，类似于每一个程序都有一个 ID 一样。
- `UserName <optional, string="">` 指定运行启动项的用户，只有当 Launchd 作为 root 用户运行时，此项才适用。
- `GroupName <optional, string="">` 指定运行启动项的组，只有当 Launchd 作为 root 用户运行时，此项才适用。
- `Program<optional, string="">` 这个值用来指定进程的可执行文件的路径。
- `ProgramArguments<optional,array of="" strings="">` 如果未指定`Program`时就必须指定该项，包括可执行文件文件和运行的参数。
- `KeepAlive <optional, boolean="">` 用来控制可执行文件是持续运行，还是满足具体条件之后再启动。默认值为`false`，也就是说满足具体条件之后才启动。当设置值为`true`时，表明无条件的开启可执行文件，并使之保持在整个系统运行周期内。
- `RunAtLoad <optional, boolean="”>` 标识 Launchd 在加载完该项服务之后立即启动路径指定的可执行文件。默认值为`false`。
- `SuccessfulExit <optional, boolean="”>` 此项为 `true` 时，程序正常退出时重启（即退出码为 0）；为 `false` 时，程序非正常退出时重启。此项设置时会隐含默认 `RunAtLoad = true`，因为程序需要至少运行一次才能获得退出状态。

所以不能简单的把以上目录中的 plist 删除来解决开机启动问题，这样会导致某些应用启动失败。最保险的办法是根据 plist 的文件名字，猜测它的作用，然后再配置其中的 Key。

- 如果 `KeepAlive` = false：

- - 当 `RunAtLoad` = false 时：程序只有在有需要的时候运行。
  - 当 `RunAtLoad` = true 时：程序在启动时会运行一次，然后等待在有需要的时候运行。
  - 当 `SuccessfulExit` = true / false 时：不论 `RunAtLoad` 值是什么，都会在启动时运行一次。其后根据 `SuccessfulExit` 值来决定是否重启。

- 如果 `KeepAlive` = true ：

- - 不论 `RunAtLoad`/`SuccessfulExit` 值是什么，都会启动时运行且一直保持运行状态。

如果不希望开机自动运行，则需要：

> 1. 找到对应程序的 .plist 文件
> 2. 删除 SuccessfulExit 属性。
> 3. 将 RunAtLoad / KeepAlive 均设为 false

## StartupItems

StartupItems，顾名思义，就是在系统启动过程中运行的程序，它们可以是运行完就立即终止的程序，也可以是一直持续在系统运行周期的后台进程。

StartupItems 一般存放在以下两个路径下：

1. /System/Library/StartupItems
2. /Library/StartupItems

大部分与系统相关的 StartupItems 都放在`/System/Library/StartupItems`这个路径下，它们会先于 `/Library/StartupItems` 路径下的执行，因为前者路径下的 StartupItems 提供了系统级的基础服务，而后者路径在默认情况下是不存在的，需要自己手动创建。

## References

1. https://www.jianshu.com/p/542f6359f2d4
2. https://www.zhihu.com/question/28268529
