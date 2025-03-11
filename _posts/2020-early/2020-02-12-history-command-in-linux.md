---
title: Linux中的history命令
categories: [Tech]
tags: [Linux, shell]
date: 2020-02-12
---

`history`是用来显示命令历史的命令。

<!-- more -->

```
root@aml:~# history
    1  which git
    2  cd /
    3  ls -l
    4  ifconfig
    5  alias
    ...
```

1. 默认记忆 1000 个历史，这些命令保存在家目录的`~/.bash_history`里。
2. `history #`列出最近的`#`条命令，例如 `history 5`
3. `history -c`会将当前 shell 里的命令历史记录。
4. `history -d #`会删除第`#`条历史命令，例如`history -d 10`删除第 10 条历史。
5. `history -w`会将当前 shell 里的命令历史写进 `.bash_history`，注销后也会自动写入。
6. `!#`用来执行第`#`条命令，例如`!5`就是执行第 5 条历史命令。
7. `!-#`用来执行倒数第`#`条命令，例如`!-2`就是执行倒数第 2 条命令。
8. `!command`用来执行**最近历史**里的以`command`开头的命令，例如`!ls`会执行最近的`ls`命令包括参数。
9. `!!` 用来执行上一条历史命令。
10. `!$`可以取到上一条命令的参数，假如刚刚执行完`vi hello.txt`再 `cat !$` ，等同于 `cat hello.txt`。
11. `echo "export $HISTSIZE=500" >> /etc/profile` 修改当前 shell 缓存的历史上限。
12. `echo "export $HISTFILE=~/.history" >> /etc/profile` 修改保存历史的文件名字。
13. `echo "export $HISTFILESIZE=1000" >> /etc/profile` 修改最大保存历史命令上限。
14. `echo "export HISTTIMEFORMAT='%F %T" >> /etc/profile` 修改历史文件的内容格式，带上时间戳。
15. 以上修改需要 `source /etc/profile`才能生效，不过更建议修改个人目录下的 `.bash_profile`。
