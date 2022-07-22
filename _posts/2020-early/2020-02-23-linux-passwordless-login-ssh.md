---
title: Linux免密码登录SSH
categories: [Tech]
tags: [linux, ssh, shell]
date: 2020-02-23
---

无密钥登录可以更快乐一点。

<!-- more -->

第一步，生成公钥和私钥。

```sh
ssh-keygen -t rsa     ##-t rsa可以省略，默认就是生成rsa类型的密钥
```

按提示会在当前主机的 ~/.ssh 生成 id_rsa, id_rsa.pub 。

第二步，将公钥 id_rsa.pub 复制到目标主机的 ~/.ssh/authorized_keys 中。方法很多，推荐使用 ssh-copy-id

```sh
# Copy your keys to the remote machine:
ssh-copy-id username@remote_host

# Copy the given public key to the remote:
ssh-copy-id -i path/to/certificate username@remote_host

# Copy the given public key to the remote with specific port:
ssh-copy-id -i path/to/certificate -p port username@remote_host
```

也可以手动复制粘贴，要注意文件权限。

```sh
# on client machine
cat id_rsa.pub >> authorized_keys
scp authorized_keys root@192.168.1.116:/root/.ssh/

# on remote machine
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

最后测试登录即可。

```
ssh 192.168.1.1      # 使用当前用户名，如果不存在就报错
ssh root@192.168.1.1 # 使用root
```

最后贴一下口令登录验证原理。

![img](https://raw.githubusercontent.com/tobyqin/img/master/1586336-20190213210715597-629546872.png)

以及密钥登录验证原理。

![img](https://raw.githubusercontent.com/tobyqin/img/master/1586336-20190213210725731-973454197.png)

参考文章：https://www.cnblogs.com/henkeyi/p/10487553.html
