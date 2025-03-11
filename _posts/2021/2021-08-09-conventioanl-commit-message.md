---
title: 程序员的节操
categories: [Tech]
tags: [DevOps, Code]
date: 2021-08-09
---

从代码提交记录能看出一个程序员的节操，真的。

## 节操掉了一地

![commit messages: ProgrammerHumor](https://image.tobyqin.cn/commit-message.png)

在敏捷开发里我们提倡频繁提交代码，但是这并不意味着对提交的代码和提交记录的质量妥协。你身边有没有这样的程序员大哥大姐，在提交代码时是这样写的提交信息？

![just update](https://image.tobyqin.cn/image-20210809222633183.png)

节操仿佛掉了一地，甚至还有下面这样的。

![just shit](https://image.tobyqin.cn/image-20210810074620042.png)

![just bug fix](https://image.tobyqin.cn/image-20210810074738991.png)

## 规范化的代码提交记录

在开源社区，有这么一套规范 [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)，就是用来约定代码提交信息的格式，可以帮助大家更好的把节操抓在手里，而不是扔在地上。内容不是太长，几分钟就可以看完学会。

### 默认格式

```
<type>(<optional scope>): <subject>
empty separator line
<optional body>
empty separator line
<optional footer>
```

### 当分支合并时

```
Merge branch '<branch name>'
```

后面跟着默认的分支合并信息。

### 当撤销改动时

```
Revert "<commit headline>"
empty separator line
This reverts commit <commit hash>.
<optional reason>
```

后面跟着默认的撤销改动信息。

### 提交类型 Type

提交类型可以让其他用户了解这个改动的初衷，详细列表可以在后文的参考链接找到，下面列举一些常用的类别。

- API relevant changes

  - `feat` Commits, that adds a new feature，新功能
  - `fix` Commits, that fixes a bug，修复问题
  - `refactor` Commits, that rewrite/restructure your code, however does not change any behaviors，重构
  - `perf` Commits are special `refactor` commits, that improves performance，调优

- `style` Commits, that do not affect the meaning (white-space, formatting, missing semi-colons, etc)，修改样式
- `test` Commits, that add missing tests or correcting existing tests，测试相关
- `docs` Commits, that affect documentation only，文档相关
- `build` Commits, that affect build components like build tool, ci pipeline, dependencies, project version, 编译相关
- `ops` Commits, that affect operational components like infrastructure, deployment, backup, recovery, 基础设施相关
- `chore` Miscellaneous commits e.g. modifying `.gitignore`，其他杂项

任何的提交记录都应该以提交类型开头，如果是很重要的改动可以选择性地加上感叹号，例如：

```
refactor!: drop support for node 8
```

### 变更范围 Scope

The `scope` provides additional contextual information. 变更范围是可选的，在我们实际项目中我们在这里写入了需求卡片 ID，比如 JIRA ID，当然也可以写模块名称。

### 变更主题 Subject

The `subject` contains a succinct description of the change. 用一句话描述改动了什么，规范里建议不操作 72 个字符，这是最能体现节操的部分。

- Is a **mandatory** part of the format，在规范里这是必填项。
- Use the imperative, present tense: "change" not "changed" nor "changes"，建议语法使用现在时而不是过去时。
- Don't capitalize the first letter，首字母不要大写。
- No dot (.) at the end，末尾不要加句号。

### 变更详情 Body

The `body` should include the motivation for the change and contrast this with previous behavior. 详情里主要写变更的原因和背景，而不是写改了什么，改了什么主要还是通过 diff 来了解。

- Is an **optional** part of the format，这是可选内容。
- Use the imperative, present tense: "change" not "changed" nor "changes"，使用现在时而不是过去时。
- This is the place to mention issue identifiers and their relations，可以放需求卡片 ID 或者相关联的其他信息

### 变更注脚 Footer

The `footer` should contain any information about **Breaking Changes** and is also the place to **reference Issues** that this commit refers to. 注脚里可以放跟 Breaking Changes 之类的其他信息，或者放任何跟这次改动相关的参考信息。

- Is an **optional** part of the format
- **optionally** reference an issue by its id.
- **Breaking Changes** should start with the word `BREAKING CHANGES:` followed by space or two newlines. The rest of the commit message is then used for this.

### 具体例子 Examples

```
  feat(shopping cart): add the amazing button

  feat: remove ticket list endpoint
  refers to JIRA-1337
  BREAKING CHANGES: ticket endpoints no longer supports list all entities.

  fix: add missing parameter to service call
  The error occurred because of <reasons>.

  build(release): bump version to 1.0.0

  build: update dependencies

  refactor: implement calculation method as recursion

  style: remove empty line
```

## 优化提交记录有什么好处

好处很多，首先，你的节操又捡起来了。

其次，良好的提交记录可以提高你的口碑和声誉，我们可以随手去知名的开源项目翻阅一下，大神们不仅对代码有极高的要求，对提交记录也一样。

![tidy commit message](https://image.tobyqin.cn/conventional-git-commit.jpg)

大家在翻开其他同行的代码时，第一眼看的并不是代码，而是提交记录，他对他的好感有时候就是那么简单自然。

还有，在开源社区很多项目的 Change Log 都是自动生成的，良好的提交记录就是这些自动化技术的基础。

## 如何简化这个有点繁琐的流程

如果你遇到任何事情都有偷懒和简化的思维，恭喜你，你已经具有高级工程师的基本素质了。坦白讲这个规范其实也没那么烦，但是一定要简化的话还是有懒人给懒人写了工具。

1. IDEA 可以用 [Conventional Commit](https://plugins.jetbrains.com/plugin/13389-conventional-commit)
2. VSCode 可以用 [Conventional Commits](https://marketplace.visualstudio.com/items?itemName=vivaxy.vscode-conventional-commits)

![image](https://image.tobyqin.cn/idea-conventional-commit.gif)

我真的不是强行凑字数，它们的名字就这样。

还有一些插件更厉害了，可以让你的代码不按规范写 Commit Message 就没法提交，只要大家觉得这样没毛病，就可以把这个插件带到你的团队，这个插件叫，[Husky](https://github.com/typicode/husky)。

![image](https://image.tobyqin.cn/dog-husky.jpg)

关于 pre-commit-hook 和 husky 的使用，我们下次再讲，过了今天，愿你我的节操同在。
