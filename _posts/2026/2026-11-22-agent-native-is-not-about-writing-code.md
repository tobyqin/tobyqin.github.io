---
title: Agent Native 的终点，不是 AI 写代码
categories: [Tech]
tags: [ai, engineering, architecture]
date: 2026-11-22
---

过去一年，所有关于 AI 写代码的讨论，都在追错问题。

真正的问题从来不是 Agent 能写多少代码。

---

## 代码正在变成最便宜的东西

代码生成已经在商品化。Copilot、Cursor、Claude Code——生成能力在趋同，区别越来越小。

软件工程正在从"代码稀缺"进入"上下文稀缺"的时代。

一个 Agent 可以在几分钟内生成几千行代码。但它不知道：

- 为什么这个功能存在
- 哪条业务规则不能被破坏
- 哪个依赖动不得
- 什么样的测试结果才算证明变更安全

这些知识不在代码里。它们在设计讨论、事故复盘、历史决策、以及资深工程师的脑子里。

**未来最便宜的是代码。最昂贵的是正确的上下文。**

---

## Agent Native 的核心不是编码

很多人把 Agent Native Development 理解为"让 Agent 独立完成开发任务"。

这个定义过于表面。

Agent 真正受限的从来不是编码能力，是理解能力和验证能力。

一个成熟工程师花大量时间做的事不是写代码：理解需求、澄清边界、分析影响范围、验证设计假设、判断风险、证明结果正确。

这些都是知识工作。

所以 Agent Native 的核心问题不是：

> How do we make agents generate code?

而是：

> How do we make organizational knowledge consumable and verifiable?

---

## Repository 只是真相的一部分

很多团队开始维护 `AGENTS.md`、`CONTEXT.md`、OpenSpec、Repository Documentation。

这些实践有价值——它们在把人脑中的知识显式化。

但这里有一个根本性的误区：

**Repository 不是 Source of Truth。**

代码库只保存系统知识的一部分。另一部分在别处：Datadog 指标、Grafana Dashboard、Feature Flag 配置、AWS 状态、Incident History、Architecture Decision Record。

一个服务为什么这样设计，往往比代码本身更重要。而这个答案未必在 Repo 里。

> Repository is part of the truth, not the truth.

---

## 知识系统最难的不是沉淀，是维护

历史上几乎所有知识管理系统都失败过。Wiki、Confluence、SharePoint、Design Doc——无一例外。

原因不复杂。不是没人写，是没人维护。

当维护成本高于收益，知识迅速过期。团队开始只信代码，不信文档。知识系统失去价值。

**Documentation as Responsibility 很难成功。**

依赖工程师主动维护文档，系统大概率会衰败。这不是态度问题，是激励结构问题。

---

## 更合理的模式：Agent 维护知识，不只是消费知识

今天大多数实践的默认模式是：

```
Human writes context → Agent reads context
```

这仍然是人工驱动。

更可持续的模式应该是：

```
Human owns truth → Agent maintains truth
```

代码变更后，Agent 自动更新 OpenSpec、README、API 文档、部署说明。工程师负责审核，而不是逐字编写。

**Documentation 应该是开发的副产品，而不是独立的活动。**

只有当知识维护变得足够廉价，知识系统才能持续存在。

---

## 真正在发生变化的是知识管理

很多人以为 AI 会改变编码。

AI 更可能改变的是知识管理。

过去的软件工程：人记在脑子里，然后写成代码。

未来的软件工程：人、Agent 共同沉淀组织知识，知识同时服务于人和机器。

知识不再属于某个人，也不属于某个文档。它属于组织。

---

## 更准确的目标

我越来越倾向于认为，Agent Native Development 只是路过的站点。

真正的终点是：

> Organizational Knowledge Native Development.

几个核心原则：

1. 代码不是资产，知识才是资产。
2. 知识必须能被机器理解，不只是被人阅读。
3. 知识应该尽量靠近代码，但不局限于代码仓库。
4. Agent 不只消费知识，也持续维护知识。
5. 验证能力比生成能力更重要。
6. 目标不是减少工程师，而是减少知识丢失。

软件工程的未来，不是 AI 写出更多代码。

是把原本只存在于少数资深工程师脑中的隐性知识，持续沉淀为人和机器都能理解、验证、演化的组织知识。

这才是值得构建的东西。
