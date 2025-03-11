---
title: 让vi在保存文件时获得sudo权限
categories: [Tips]
tags: [vim, linux]
date: 2020-02-23
---

改完文件后发现没权限保存？可以临时补救一下。

<!-- more -->

```
:w !sudo tee %
```

完了之后还要强制退出一下。

```
:q!
```

额外赠送两个非常好用的快捷键（非编辑模式，一般先按`Ecs`）：

- 按住`Shift`，再按`zz`：保存退出
- 按住`Shift`，再按`zq`：不保存退出
