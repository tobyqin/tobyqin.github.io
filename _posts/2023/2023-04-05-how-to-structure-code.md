---
title: 如何组织你的代码结构
categories: Tech
tags: engineering
date: 2023-04-05
---

# 如何组织你的代码结构

> 任何设计师都会告诉你，结构设计是设计过程中最重要的步骤。创造开始的前几笔，承载着其余部分的命运。— Christopher Alexander
> 

我们开始看一个程序的第一步是什么？就是它的目录结构。一起从结构开始，程序好坏，从结构上大致就能分辨出来。

## 最佳实践

以下示例可以作为 Java 项目的一种最佳实践，核心思想来源于一个博主的视频和最近看的一些文章。

```java
project-name
  .github   // 可选，Github 配置文件，如有需要
  .ci       // 推荐，存放 CI 相关的配置文件
  design    // 推荐，存放设计文档
    adr     // 推荐，架构设计决策记录，详情请查询 ADR
      adr-0001...
  docs      // 推荐，存放项目文档，一般是 README 写不下的部分，或者文档需要的图片等资源
  scripts   // 可选，存放项目使用的脚本，如果有的话
  src       // Java 国际规范，源代码目录
    main    // Java 国际规范，源代码主程序目录
      java  // Java 国际规范
        com.example             // 包名
          customer              // 领域划分，建议将领域中对象整合，方便未来管理、拆分和重构
            Customer            // 领域对象模型
            CustomerConfig      // 领域配置
            CustomerController  // 领域路由控制器
            CustomerDAO         // 领域传输对象模型
            CustomerService     // 领域服务
          db                    // 可选，DB 相关
          exception             // 可选，异常类型
            ApiError
          order                 // 其他领域示例
            Order...
          shared                // 公共类、工具类
            FileUtils
          Application           // 程序主函数入口
        resources               // Java 国际规范，程序资源目录
          application.yml
          application-dev.yml
          application-uat.yml
          application-prd.yml
    test                        // Java 国际规范，程序测试目录
      java
        customer
          CustomerServiceTest   // 测试类应按被测对象结构整理
      resources
  .gitignore// 推荐，git ignore 配置
  pom.xml   // Maven 配置文件，在 Gradle 也有等同配置
  LICENSE   // 推荐，版权文件，开源项目都应该有
  README.md // 推荐，项目介绍，一个项目的第一印象，非常重要
        
```

在 IDEA 中示例如下：

![](https://image.tobyqin.cn/202304152235299.png)

## 按包而不是按层封装

主要有以下好处：

1. 更好的模块化，这样才可能达到高内聚低耦合的目的
2. 找代码更容易，大多时候我们不是在写代码，而是在翻代码，如果找起来方便就是效率提升
3. 更高层次的抽象，按业务划分更**强调业务而不是实现细节**，文档自动化后也会轻松许多
4. 最小化影响范围，假设后期需要对项目重构或者拆分，按包封装的结构将更容易处理
5. 更容易扩展，如果某个业务以后变的很复杂，这还是在这个包内操作，那时可以再在包内分层

## 举一些例子

想象一辆汽车，在汽车的实现里，如果按功能可能是这样划分的：

1. 安全系统
2. 引擎系统
3. 操控系统
4. 燃油系统 等等

但如果按层划分会是怎么样的呢？

1. 机械系统
2. 液压系统
3. 电气系统 等等

当然这样理解和组合问题没错，但其实现的复杂度和耦合度是很难估量的。

再想象一下一个公司的结构，如果按包划分，可能是这样的（等同于按部门划分）：

1. 前台
2. 后台
3. 会计
4. 人事

如果按层划分，大概会是这样（等同于按组织结构划分）：

1. 董事会
2. 高级经理
3. 员工

按层划分之后，对象会有交集吗？可能吧。他们是同一类人，也许会在一起开会讨论。但是他们能直接对业务产生效果吗？并不会，真正业务的结果还是来自于各个部门，部门内部的交流和耦合才更加有效。这些人其实就是每个部门的接口类。

那为什么很多人还是喜欢按层的方式来组织代码呢？可能只是一个需要改掉的坏习惯吧。按层思考更省力气，但或许是对业务理解的不够透彻。

## 参考链接：

1. Java 项目结构最近实践： [https://www.youtube.com/watch?v=DoK3-9V5HRE](https://www.youtube.com/watch?v=DoK3-9V5HRE)
2. Golang 项目结构 [https://github.com/golang-standards/project-layout](https://github.com/golang-standards/project-layout)
3. Maven 标准项目结构 [https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)
4. 按包而不是按层划分项目功能 [http://www.javapractices.com/topic/TopicAction.do?Id=205](http://www.javapractices.com/topic/TopicAction.do?Id=205)