---
title: 解决 Jenkins中TFS Plugin Mapping错误的问题
date: 2016-10-19 14:29:04
tags: [tips, tfs, jenkins]
categories: Tech
---

处理 TFS 的问题备忘。

<!-- more -->

### 问题描述

Once you had update in TFS workspace for Jenkin TFS plugin, you might get error like bellow:

如果你把 Jenkins 中 TFS 插件更新过，那么你有可能会遇到 Mapping 错误的问题。

```
[workspace] $ "C:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\IDE\TF.exe" workspaces -format:brief -server:http://tfs.xxx.com:8080/tfs/Default ********
Collection: tfs.xxx.com\Default
Workspace   Owner      Computer    Comment
----------- ---------- ----------- --------------------------------------------
MyServer newUser MyServer
[workspace] $ "C:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\IDE\TF.exe" workspace -new "Hudson-My build job-MASTER;nam\newUser" -noprompt -server:http://tfs.xxx.com:8080/tfs/Default ********
The path D:\hudson\jobs\My build job\workspace is already mapped in workspace Hudson-My build job-MASTER;NAM\oldUser.
FATAL: Executable returned an unexpected result code [100]
ERROR: null
```

### 解决办法

You should follow bellow steps to fix it.

**Step 1: Remove the TFS workspace**

- From VS:

  1. Open Team Explorer
  2. Click Source Control Explorer
  3. In the nav bar of the tool window there is a drop down labeled "Workspaces".
  4. Extend it and click on the "Workspaces..." option (yeah, a bit un-intuitive)
  5. The "Manage Workspaces" window comes up. Click edit and you can add / remove / edit your workspace

- From the command line
  1. Type "tf workspace" from a developer command promt. It will bring up the "Manage Workspaces" directly!

**Step 2: Remove cache on this computer**

Manually delete all the files in the TFS cache, they located at:

- C:\Users\{UserName}\AppData\Local\Microsoft\Team Foundation\3.0\Cache

- If there is a \4.0\Cache and \5.0\Cache existed, delete them all.

你可以尝试通过以下步骤解决。

**Step 1: 删除该 TFS workspace**

- 从 Visual Studo 操作:

  1. 打开 Team Explorer
  2. 打开 Source Control Explorer
  3. 从工具栏下拉列表中找到 "Workspaces"，展开 "Workspaces..."
  4. 这时 "Manage Workspaces" 窗口会打开，在这里你可以编辑或者删除当前用户所有的 workspace

- 从命令提示符操作
  1. 在 VS 命令提示符中输入 "tf workspace" 可以看到相关命令，不行就查一下 MSDN

**Step 2: 删除 TFS 相关 Cache**

手动清除 TFS 的 Cache 文件，参考以下路径。

- C:\Users\{UserName}\AppData\Local\Microsoft\Team Foundation\3.0\Cache
- 如果 3.0 找不到就 4.0，如果 4.0 也没有就 5.0，取决于你的 VS 版本。
