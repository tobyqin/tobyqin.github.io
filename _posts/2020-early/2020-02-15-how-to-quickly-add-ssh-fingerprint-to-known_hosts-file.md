---
title: 如何快速将SSH指纹添加到known_hosts文件中
categories: [Tech]
tags: [Linux, shell, ssh, git]
date: 2020-02-15
---

每次连接新的 SSH 或者从新的域名克隆代码时，总是会提示你是否信任，需要手动确认。

<!-- more -->

```sh
git clone git@github.com:tobyqin/blog.git
Cloning into 'blog'...
The authenticity of host 'github.com (52.74.223.119)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)?
```

在 CICD 过程中，这种交互是要避免的。

方法一：你用 `ssh-keyscan`来自动扫描远程主机的指纹并添加到 known_hosts 文件。

```sh
# by host
ssh-keyscan github.com >> ~/.ssh/known_hosts
# by ip
ssh-keyscan -H 52.74.223.119 >> ~/.ssh/known_hosts
```

方法二：让 ssh 永久信任某个域名，比如这样。

```sh
ssh -o StrictHostKeyChecking=no tobyqin@github.com
Warning: Permanently added 'github.com,13.250.177.223' (RSA) to the list of known hosts.
```

这种方法不是很推荐，因为它一旦信任某个 host 后，以后就算指纹更新了也会继续连接，有可能引发中间人攻击。

> **StrictHostKeyChecking**
>
> [...] If this flag is set to “no” or “off”, ssh will automatically add new host keys to the user known hosts files and allow connections to hosts with changed hostkeys to proceed, subject to some restrictions. [...]
