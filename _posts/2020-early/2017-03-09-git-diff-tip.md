---
title: 理解 Git Diff 命令
date: 2017-03-09 23:04:51
tags: [git, tips]
categories: Tech
---

你确定自己会`git diff` 吗？

<!-- more -->

在 git 提交过程中，存在三大环节：

- working tree
- index file (staged)
- commit

这三大环节中，你应该有一个大概的了解：

1. **working tree**：就是你所工作在的目录，每当你在代码中进行了修改，working tree 的状态就改变了。
2. **index file**：是索引文件，它是连接`working tree`和`commit`的桥梁，每当我们使用`git add`命令来登记修改的文件后，`index file`的内容就改变了，此时`index file`就和`working tree`同步了。
3. **commit**：这是提交更改完成的最后阶段，commit 后我们的代码才真正进入了 git 仓库。我们使用`git commit`就是将`index file`里的内容提交到`commit`中。

总结一下：

- **git diff**：是查看`working tree`与`index file`的差别。
- **git diff --cached**：是查看`index file`与`commit`的差别。
- **git diff HEAD**：是查看`working tree`和`commit`的差别。（你一定没有忘记，HEAD 代表的是最近的一次 commit 的信息）

参考文档： http://www.cnblogs.com/Alight/p/3571042.html
