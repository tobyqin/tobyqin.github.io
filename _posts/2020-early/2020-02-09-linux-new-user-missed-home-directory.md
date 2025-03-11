---
title: Linux新建用户没有用户目录
categories: [Tech]
tags: [Linux, shell]
date: 2020-02-09
---

简而言之，用`adduser`而不是`useradd`，用`deluser`而不是`userdel`。

<!-- more -->

```

NAME
       adduser, addgroup - add a user or group to the system

SYNOPSIS
       adduser  [options] [--home DIR] [--shell SHELL] [--no-create-home] [--uid
       ID] [--firstuid ID] [--lastuid ID] [--ingroup GROUP | --gid  ID]  [--dis‐
       abled-password]  [--disabled-login]  [--gecos GECOS] [--add_extra_groups]
       user

       adduser --system [options] [--home  DIR]  [--shell  SHELL]  [--no-create-
       home] [--uid ID] [--group | --ingroup GROUP | --gid ID] [--disabled-pass‐
       word] [--disabled-login] [--gecos GECOS] user

       addgroup [options] [--gid ID] group

       addgroup --system [options] [--gid ID] group

       adduser [options] user group
```

`useradd` 和 `userdel` 的尝试：

```shell
useradd user1 # 不会创建home目录，没有回显
useradd -m user2 # 会创建home目录，没有回显

useradd -m user1 # 不会补充创建home目录，回显报错
useradd: user 'user1' already exists

# 可以通过复制home模板补救，模板在/etc/skel，有时候也被叫做骨架目录
cp /etc/skel/ /home/user1 -a
chmod 700 /home/user1  #只有owner拥有所有所有权限
chown user1:user1 /home/user1 -R # owner改成user1

# 给用户加密码
passwd user1
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully

# 删除用户
userdel -r user1 # 删除用户相关所有资源，包括home目录
userdel user2 # 保留home目录

```

当然，如果误删除了用户的 home 目录可以可以用上面的方法来修复。关于`adduser`和`deluser`的尝试：

```shell
root@aml:~# adduser user3
Adding user `user3' ...
Adding new group `user3' (1002) ...
Adding new user `user3' (1002) with group `user3' ...
Creating home directory `/home/user3' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for user3
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]Y

root@aml:~# deluser user3
Removing user `user3' ...
Warning: group `user3' has no more members.
Done.
```

可以看到，`adduser`是交互式的，回显里有完整的信息，包括`home`目录的位置和复制模板的过程，还会让你创建密码和完善用户信息。

`deluser`也差不多，告诉你删除了那些资源。

顺便备忘一下查看用户信息的命令。

1. 查看 `/etc/passwd` 文件
2. 使用`getent passwd` 命令
3. 使用 `compgen -u` 命令

**参考：**

- [linux 使用 useradd 创建的用户没有目录的解决办法](https://www.cnblogs.com/sogeisetsu/p/11401562.html)
