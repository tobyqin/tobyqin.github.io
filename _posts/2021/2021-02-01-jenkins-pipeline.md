---
title: Jenkins Pipeline 一点通
categories: [Tech]
tags: [Jenkins]
date: 2021-02-01
---
本文主要介绍在生产环境中持续集成与持续部署的使用，主要通过实现 Jenkins 流水线脚本自动发布应用到 Kubernetes 集群当中。

## CI/CD 介绍

CI（Continuous Integration，持续集成）/CD（Continuous Delivery，持续交付）是一种通过在应用开发阶段引入自动化来频繁向客户交付应用的方法。CI/CD 的核心概念是持续集成、持续交付和持续部署。作为一个面向开发和运营团队的解决方案，CI/CD 主要针对在集成新代码时所引发的问题（亦称 “集成地狱”）。

具体而言，CI/CD 在整个应用生命周期内（从集成和测试阶段到交付和部署）引入了持续自动化和持续监控，这些关联的事务通常被称为 “CI/CD 管道”，由开发和运维团队以敏捷方式协同支持。

### CI 和 CD 的区别

CI/CD 中的 CI 指持续集成，它属于开发人员的自动化流程。成功的 CI 意味着应用代码的最新更改会定期构建、测试并合并到共享存储中。该解决方案可以解决在一次开发中有太多应用分支，从而导致相互冲突的问题。

CI/CD 中的 CD 指的是持续交付或持续部署，这些相关概念有时会交叉使用。两者都事关管道后续阶段的自动化，但它们有时也会单独使用，用于说明自动化程度。

持续交付通常是指开发人员对应用的更改会自动进行错误测试并上传到存储库（如 GitLab 或容器注册表），然后由运维团队将其部署到实时生产环境中，旨在解决开发和运维团队之间可见性及沟通较差的问题，因此持续交付的目的就是确保尽可能减少部署新代码时所需的工作量。

持续部署指的是自动将开发人员的更改从存储库发布到生产环境中以供客户使用，它主要为解决因手动流程降低应用交付速度，从而使运维团队超负荷的问题。持续部署以持续交付的优势为根基，实现了管道后续阶段的自动化。

CI/CD 既可能仅指持续集成和持续交付构成的关联环节，也可以指持续集成、持续交付和持续部署这三个方面构成的关联环节。更为复杂的是有时持续交付也包含了持续部署流程。

纠缠于这些语义其实并无必要，只需记得 CI/CD 实际上就是一个流程（通常表述为管道），用于在更大程度上实现应用开发的持续自动化和持续监控。

### 持续集成（CI）

现代应用开发的目标是让多位开发人员同时开发同一个应用的不同功能。但是，如果企业安排在一天内将所有分支源代码合并在一起，最终可能导致工作繁琐、耗时，而且需要手动完成。这是因为当一位独立工作的开发人员对应用进行更改时，有可能会有其他开发人员同时进行更改，从而引发冲突。

持续集成可以帮助开发人员更加频繁地将代码更改合并到共享分支或主干中。一旦开发人员对应用所做的更改被合并，系统就会通过自动构建应用并运行不同级别的自动化测试（通常是单元测试和集成测试）来验证这些更改，确保更改没有对应用造成破坏。这意味着测试内容涵盖了从类和函数到构成整个应用的不同模块，如果自动化测试发现新代码和现有代码之间有冲突，持续集成（CI）可以更加轻松地快速修复这些错误。

### 持续交付（CD）

完成持续集成中构建单元测试和集成测试的自动化流程后，通过持续交付可自动将已验证的代码发布到存储库。为了实现高效的持续交付流程，务必要确保持续交付已内置于开发管道。持续交付的目标是拥有一个可随时部署到生产环境的代码库。

在持续交付中，每个阶段（从代码更改的合并到生产就绪型构建版本的交付）都涉及测试自动化和代码发布自动化。在流程结束时，运维团队可以快速、轻松地将应用部署到生产环境中。

### 持续部署

对于一个成熟的 CI/CD 管道来说，最后的阶段是持续部署。作为持续交付（自动将生产就绪型构建版本发布到代码存储库）的延伸，持续部署可以自动将应用发布到生产环境中。由于生产之前的管道阶段没有手动门控，因此持续部署在很大程度上都得依赖于精心设计的测试自动化。

实际上，持续部署意味着开发人员对应用的更改在编写后的几分钟内就能生效，这更加便于持续接收和整合用户反馈。总而言之，所有这些 CI/CD 的关联步骤都有助于降低应用的部署风险，因此更便于以小件的方式（非一次性）发布对应用的更改。不过，由于还需要编写自动化测试以适应 CI/CD 管道中的各种测试和发布阶段，因此前期投资会很大。

## Jenkins 流水线介绍

本节主要讲解 Jenkins 的新功能 —— Jenkins 流水线（Pipeline）的使用，首先介绍流水线的概念和类型，然后讲解流水线的基本语法和一些例子。

### 什么是流水线

Jenkins 流水线（或 Pipeline）是一套插件，它支持实现并把持续提交流水线（Continuous Delivery Pipeline）集成到 Jenkins。

持续提交流水线（Continuous Delivery Pipeline）会经历一个复杂的过程：从版本控制、向用户和客户提交软件，软件的每次变更（提交代码到仓库）到软件发布（Release）。这个过程包括以一种可靠并可重复的方式构建软件，以及通过多个测试和部署阶段来开发构建好的软件（称为 Build）。

流水线提供了一组可扩展的工具，通过流水线语法对从简单到复杂的交付流水线作为代码进行建模，Jenkins 流水线的定义被写在一个文本文件中，一般为 Jenkinsfile，该文件 “编制” 了整个构建软件的过程，该文件一般也可以被提交到项目的代码仓库中，在 Jenkins 中可以直接引用。这是流水线即代码的基础，将持续提交流水线作为应用程序的一部分，像其他代码一样进行版本化和审查。创建 `Jenkinsfile` 并提交到代码仓库中的好处如下：

- 自动地为所有分支创建流水线构建过程
- 在流水线上进行代码复查 / 迭代
- 对流水线进行审计跟踪
- 流水线的代码可以被项目的多个成员查看和编辑

### Jenkins 流水线概念

流水线主要分为以下几种区块：Pipeline、Node、Stage、Step 等。

- Pipeline（流水线），Pipeline 是用户定义的一个持续提交（CD）流水线模型。流水线的代码定义了整个的构建过程，包括构建、测试和交付应用程序的阶段。另外，Pipeline 块是声明式流水线语法的关键部分。
- Node（节点），Node（节点）是一个机器，它是 Jenkins 环境的一部分，另外，Node 块是脚本化流水线语法的关键部分。
- Stage（阶段），Stage 块定义了在整个流水线的执行任务中概念不同的子集（比如 Build、Test、Deploy 阶段），它被许多插件用于可视化 Jenkins 流水线当前的状态 / 进展。
- Step（步骤），本质上是指通过一个单一的任务告诉 Jenkins 在特定的时间点需要做什么，比如要执行 shell 命令，可以使用 `sh SHELL_COMMAND`。
- 声明式流水线

在声明式流水线语法中，Pipeline 块定义了整个流水线中完成的所有工作，比如：

```Groovy

```

说明：

- agent any：在任何可用的代理上执行流水线或它的任何阶段。
- stage ('Build')：定义 Build 阶段。
- steps：执行某阶段相关的步骤。

### 脚本化流水线

在脚本化流水线语法中，会有一个或多个 Node（节点）块在整个流水线中执行核心工作，比如：

```
```

说明：

- node：在任何可用的代理上执行流水线或它的任何阶段。
- stage ('Build')：定义 build 阶段。stage 块在脚本化流水线语法中是可选的，然而在脚本化流水线中实现 stage 块，可以清楚地在 Jenkins UI 界面中显示每个 stage 的任务子集。

### 流水线示例

一个以声明式流水线的语法编写的 Jenkinsfile 文件如下：

```
```

常用参数说明：

- pipeline 是声明式流水线的一种特定语法，定义了包含执行整个流水线的所有内容和指令。
- agent 是声明式流水线的一种特定语法，指示 Jenkins 为整个流水线分配一个执行器（在节点上）和工作区。
- stage 是一个描述流水线阶段的语法块，在脚本化流水线语法中，stage（阶段）块是可选的。
- steps 是声明式流水线的一种特定语法，它描述了在这个 stage 中要运行的步骤。
- sh 是一个执行给定 shell 命令的流水线 step（步骤）。
- junit 是一个聚合测试报告的流水线 step（步骤）。
- node 是脚本化流水线的一种特定语法，它指示 Jenkins 在任何可用的代理 / 节点上执行流水线，这实际等同于声明式流水线特定语法的 agent 注：后续的例子中要用到。

上述声明式流水线等同于以下脚本式流水线：

```
```

## Pipeline 语法

本节主要从流水线的两种类型出发讲解 Pipeline 的语法。

### 声明式流水线

声明式流水线是在流水线子系统之上提供了一种更简单、更有主见的语法。

所有有效的声明式流水线必须包含在一个 Pipeline 块中，比如以下是一个 Pipeline 块的格式：

```
pipeline {/* insert Declarative Pipeline here */}
```

在声明式流水线中有效的基本语句和表达式遵循与 Groovy 的语法同样的规则，但有以下例外：

- 流水线顶层必须是一个 block，pipeline {}。
- 没有分号作为语句分隔符，每条语句都在自己的行上。
- 块只能由 Sections、Directives、Steps 或 assignment statements 组成。

### Sections

声明式流水线中的 Sections 通常包含一个或多个 agent、Stages、post、Directives 和 Steps，本节首先介绍 agent、Stages、post，有关 Directives 和 Steps 的说明见下一小节。

agent：

agent 部分指定了整个流水线或特定的部分，在 Jenkins 环境中执行的位置取决于 agent 区域的位置，该部分必须在 Pipeline 块的顶层被定义，但是 stage 级别的使用是可选的。

①参数

为了支持可能有的各种各样的流水线，agent 部分支持一些不同类型的参数，这些参数应用在 pipeline 块的顶层，或 Stage 指令内部。

any：在任何可用的代理上执行流水线或 stage。例如：agent any。

none：当在 pipeline 块的顶部没有全局 agent，该参数将会被分配到整个流水线的运行中，并且每个 stage 部分都需要包含它自己的 agent，比如：agent none。在提供了标签的 Jenkins 环境中可用代理上执行流水线或 stage。例如：`agent {label ‘my-defined-label’}`。

node：`agent {node { label ‘labelName’} }` 和 `agent { label ‘labelName’ }` 一样，但是 node 允许额外的选项（比如 customWorkspace）。

dockerfile：执行流水线或 stage，使用从源码包含的 `Dockerfile` 所构建的容器。为了使用该选项，`Jenkinsfile` 必须从多个分支流水线中加载，或者加载 `Pipelinefrom SCM`（下面章节会涉及）。通常，这是源码根目录下的 `Dockerfile:agent {dockerfile true}`。如果在其他目录下构建 `Dockerfile，使用` dir 选择：`agent { dockerfile { dir ‘someSubDir’} }`。如果 Dockerfile 有另一个名字，可以使用 filename 选项指定该文件名。也可以传递额外的参数到 `dockerbuild`，使用 `additionalBuildArgs` 选项提交，比如：`agent { dockerfile { additionalBuildArgs ‘--build-arg foo=bar’ }` }。例如一个带有 `build/Dockerfile.build` 的仓库，在构建时期望一个参数 version：

```

```

docker：使用给定的容器执行流水线或 stage。该容器将在预置的 node（节点）上，或在由 label 参数指定的节点上，动态地接受基于 Docker 的流水线。Docker 也可以接受 args 参数，该参数可能包含直接传递到 dockerrun 调用的参数及 alwaysPull 选项，alwaysPul 选项强制 dockerpull，即使镜像（image）已经存在。比如：agent {docker ‘maven:3-alpine’} 或：

```
agent{    docker{        image ‘maven:3-alpine’            label ‘my-defined-label’            args ‘-v /tmp:/tmp’    }}
```

②常见选项

label：一个字符串，该标签用于运行流水线或个别 stage。该选项对 node、docker 和 dockerfile 可用，node 必须选择该选项。

customWorkspace：一个字符串，在自定义工作区运行流水线或 stage。它可以是相对路径，也可以是绝对路径，该选项对 node、docker 和 dockerfile 可用。比如：

```

```

reuseNode：一个布尔值，默认为 false。如果是 true，则在流水线顶层指定的节点上运行该容器。这个选项对 docker 和 dockerfile 有用，并且只有当使用在个别 stage 的 agent 上才会有效。

③示例

示例 1：在 maven:3-alpine（agent 中定义）的新建容器上执行定义在流水线中的所有步骤。

```

```

示例 2：本示例在流水线顶层定义 agentnone，确保 <<../glossary#executor, an Executor>> 没有被分配。使用 agentnode 也会强制 stage 部分包含它自己的 agent 部分。在 stage ('Example Build') 部分使用 maven:3-alpine 执行该阶段步骤，在 stage ('Example Test') 部分使用 openjdk:8-jre 执行该阶段步骤。

```

```

post：

post 部分定义一个或多个 steps，这些阶段根据流水线或 stage 的完成情况而运行（取决于流水线中 post 部分的位置）。post 支持以下 post-condition 块之一：

always：无论流水线或 stage 的完成状态如何，都允许在 post 部分运行该步骤。

changed：只有当前流水线或 stage 的完成状态与它之前的运行不同时，才允许在 post 部分运行该步骤。

failure：只有当前流水线或 stage 的完成状态为失败（failure），才允许在 post 部分运行该步骤，通常这时在 Web 界面中显示为红色。

success：当前状态为成功（success），执行 post 步骤，通常在 Web 界面中显示为蓝色或绿色。

unstable：当前状态为不稳定（unstable），执行 post 步骤，通常由于测试失败或代码违规等造成，在 Web 界面中显示为黄色。

aborted：当前状态为放弃（aborted），执行 post 步骤，通常由于流水线被手动放弃触发，这时在 Web 界面中显示为灰色。

示例：一般情况下 post 部分放在流水线的底部，比如本实例，无论 stage 的完成状态如何，都会输出一条 I will always say Hello again! 信息。

```

```

stages：

stages 包含一个或多个 stage 指令，stages 部分是流水线描述的大部分工作（work）的位置。建议 stages 至少包含一个 stage 指令，用于持续交付过程的某个离散的部分，比如构建、测试或部署。

示例：本示例的 stages 包含一个名为 Example 的 stage，该 stage 执行 echo 'Hello World' 命令输出 Hello World 信息。

```

```

seps：

steps 部分在给定的 stage 指令中执行的一个或多个步骤。

示例：在 steps 定义执行一条 shell 命令。

```

```

### Directives

Directives 用于一些执行 stage 时的条件判断，主要分为 environment、options、parameters、triggers、stage、tools、input、when 等，这里仅对常用的 environment、parameters、stage 和 when 进行介绍。

environment：

environment 制定一个键 - 值对（key-value pair）序列，该序列将被定义为所有步骤的环境变量，或者是特定 stage 的步骤，这取决于 environment 指令在流水线内的位置。

该指令支持一个特殊的方法 credentials ()，该方法可用于在 Jenkins 环境中通过标识符访问预定义的凭证。对于类型为 Secret Text 的凭证，credentials () 将确保指定的环境变量包含秘密文本内容，对于类型为 Standardusernameandpassword 的凭证，指定的环境变量为 username:password，并且两个额外的环境变量将被自动定义，分别为 MYVARNAMEUSR 和 MYVARNAMEPSW。

示例：

```

```

上述示例的顶层流水线块中使用的 environment 指令将适用于流水线中的所有步骤。在 stage 中定义的 environment 指令只会适用于 stage 中的步骤。其中 stage 中的 environment 使用的是 credentials 预定义的凭证。

parameters：

parameters 提供了一个用户在触发流水线时应该提供的参数列表，这些用户指定参数的值可以通过 params 对象提供给流水线的 step（步骤）。

可用参数：

- string：字符串类型的参数，例如：parameters {string (name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }。
- booleanParam：布尔参数，例如: parameters {booleanParam (name: 'DEBUG_BUILD', defaultValue: true, description: '') }。

示例：定义 string 类型的变量，并在 steps 中引用。

```

```

stage：

stage 指定在 stages 部分流水线所做的工作都将封装在一个或多个 stage 指令中。

示例：

```

```

when：

when 指令允许流水线根据给定的条件决定是否应该执行 stage。when 指令必须包含至少一个条件。如果 when 包含多个条件，所有的子条件必须返回 True，stage 才能执行。

①内置条件

- branch：当正在构建的分支与给定的分支匹配时，执行这个 stage，例如：when {branch 'master'}。注意，branch 只适用于多分支流水线
- environment：当指定的环境变量和给定的变量匹配时，执行这个 stage，例如：when {environment name: 'DEPLOY_TO', value: 'production'}。
- expression：当指定的 Groovy 表达式评估为 True，执行这个 stage，例如：when {expression { return params.DEBUG_BUILD} }。
- not：当嵌套条件出现错误时，执行这个 stage，必须包含一个条件，例如：when {not { branch 'master'} }。
- allOf：当所有的嵌套条件都正确时执行这个 stage，必须包含至少一个条件，例如：when {allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production'} }
- anyOf：当至少有一个嵌套条件为 True 时执行这个 stage，例如：when {anyOf { branch 'master'; branch'staging'} }。

②在进入 stage 的 agent 前评估 when

默认情况下，如果定义了某个 stage 的 agent，在进入该 stage 的 agent 后，该 stage 的 when 条件才会被评估。但是可以通过在 when 块中指定 beforeAgent 选项来更改此选项。如果 beforeAgent 被设置为 True，那么就会首先对 when 条件进行评估，并且只有在 when 条件验证为真时才会进入 agent。

③示例

示例 1：当 branch 为 production 时才会执行名为 Example Deploy 的 stage。

```

```

示例 2：当 branch 为 production，environment 的 DEPLOY_TO 为 production 才会执行名为 Example Deploy 的 stage。

```

```

示例 3：当 branch 为 production 并且 DEPLOY_TO 为 production 时才会执行名为 Example Deploy 的 stage。

```

```

示例 4：当 DEPLOY_TO 等于 production 或者 staging 时才会执行名为 Example Deploy 的 stage。

```

```

示例 5：当 BRANCHNAME 为 production 或者 staging，并且 DEPLOYTO 为 production 或者 staging 时才会执行名为 Example Deploy 的 stage。

```

```

示例 6：在进行 agent 前执行判断，当 branch 为 production 时才会进行该 agent。

```

```

**steps**

Steps 包含一个完整的 script 步骤列表。Script 步骤需要 scripted-pipeline 块并在声明式流水线中执行。对于大多数用例来说，script 步骤并不是必要的。

示例：在 steps 添加 script 进行 for 循环。

```

```

**脚本化流水线**

脚本化流水线与声明式流水线一样都是建立在底层流水线的子系统上，与声明式流水线不同的是，脚本化流水线实际上是由 Groovy 构建。Groovy 语言提供的大部分功能都可以用于脚本化流水线的用户，这意味着它是一个非常有表现力和灵活的工具，可以通过它编写持续交付流水线。

脚本化流水线和其他传统脚本一致都是从 Jenkinsfile 的顶部开始向下串行执行，因此其提供的流控制也取决于 Groovy 表达式，比如：if/else 条件：

```

```

另一种方法是使用 Groovy 的异常处理支持来管理脚本化流水线的流控制，无论遇到什么原因的失败，它们都会抛出一个异常，处理错误的行为必须使用 Groovy 中的 try/catch/finally 块，例如：

```

```

## Jenkinsfile 的使用


上面讲过流水线支持两种语法，即声明式和脚本式，这两种语法都支持构建持续交付流水线。并且都可以用来在 Web UI 或 Jenkinsfile 中定义流水线，不过通常将 Jenkinsfile 放置于代码仓库中。

创建一个 Jenkinsfile 并将其放置于代码仓库中，有以下好处：

- 方便对流水线上的代码进行复查 / 迭代。
- 对管道进行审计跟踪。
- 流水线真正的源代码能够被项目的多个成员查看和编辑。

本节主要介绍 Jenkinsfile 常见的模式以及演示 Jenkinsfile 的一些特例。

**创建 Jenkinsfile**

Jenkinsfile 是一个文本文件，它包含了 Jenkins 流水线的定义并被用于源代码控制。以下流水线实现了 3 个基本的持续交付：

```
```

对应的脚本式流水线如下：

```

```

注意：不是所有的流水线都有相同的三个阶段。

**构建**

对于许多项目来说，流水线中工作（work）的开始就是构建（build），这个阶段的主要工作是进行源代码的组装、编译或打包。Jenkinsfile 文件不是替代现有的构建工具，如 GNU/Make、Maven、Gradle 等，可以视其为一个将项目开发周期的多个阶段（构建、测试、部署等）绑定在一起的粘合层。

Jenkins 有许多插件用于构建工具，假设系统为 Unix/Linux，只需要从 shell 步骤（sh）调用 make 即可进行构建，Windows 系统可以使用 bat：

```

```

说明：

- steps 的 shmake 表示如果命令的状态码为 0，则继续，为非零则失败。
- archiveArtifacts 用于捕获构建后生成的文件。

对应的脚本式流水线如下：

```

```

**测试**

运行自动化测试是任何成功的持续交付过程中的重要组成部分，因此 Jenkins 有许多测试记录、报告和可视化工具，这些工具都是由插件提供。下面的例子将使用 JUnit 插件提供的 junit 工具进行测试。

在下面的例子中，如果测试失败，流水线就会被标记为不稳定，这时 Web 界面中的球就显示为黄色。基于记录的测试报告，Jenkins 也可以提供历史趋势分析和可视化：

```

```

说明：

- 当 sh 步骤状态码为 0 时，调用 junit 进行测试。
- junit 捕获并关联匹配 */target/.xml 的 Junit XML 文件。

对应的脚本式流水线如下：

```

```

**部署**

当编译构建和测试都通过后，就会将编译生成的包推送到生产环境中，从本质上讲，Deploy 阶段只可能发生在之前的阶段都成功完成后才会进行，否则流水线会提前退出：

```

```

说明：当前 build 结果为 SUCCESS 时，执行 publish。

对应的脚本式流水线如下：

```

```

**处理 Jenkinsfile**

本节主要介绍 Jenkins 使用中如何处理 Jenkinsfile 及 Jenkinsfile 的编写方式。

**插入字符串**

Jenkins 使用与 Groovy 相同的规则进行字符串赋值，可以使用单引号或者双引号进行赋值。例如：

```
def singlyQuoted = 'Hello'def doublyQuoted = "World"
```

引用变量需要使用双引号：

```
def username = 'Jenkins'echo 'Hello Mr. ${username}'echo "I said, Hello Mr. ${username}"
```

运行结果：

```
Hello Mr. ${username}I said, Hello Mr. Jenkins
```

**使用环境变量**

Jenkins 有许多内置变量可以直接在 Jenkinsfile 中使用，比如：

- BUILDID：当前构建的 ID，与 Jenkins 版本 1.597 + 中的 BUILDNUMBER 完全相同。
- JOB_NAME：本次构建的项目名称。
- JENKINS_URL：Jenkins 完整的 URL，需要在 System Configuration 设置。

使用 env.BUILDID 和 env.JENKINSURL 引用内置变量：

```

```

对应的脚本式流水线如下：

```
Jenkinsfile (Scripted Pipeline)node {    echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"}
```

更多参数请参考：https://wiki.jenkins.io/display/JENKINS/Building+a+software+project#Buildingasoftwareproject-JenkinsSetEnvironmentVariables。

**处理凭证**

本节主要介绍在 Jenkins 的使用过程中对一些机密文件的处理方式。

机密文件、用户名密码、私密文件：

Jenkins 的声明式流水线语法有一个 credentials () 函数，它支持 secrettext（机密文件）、usernameandpassword（用户名和密码）以及 secretfile（私密文件）。

①机密文件示例

本实例演示将两个 Secret 文本凭证分配给单独的环境变量来访问 Amazon Web 服务，需要提前创建这两个文件的 credentials（下面章节会有演示），Jenkinsfile 文件的内容如下：

```

```

说明：上述示例定义了两个全局变量 AWSACCESSKEYID 和 AWSSECRETACCESSKEY，这两个变量引用的是 credentials 的两个文件，并且这两个变量均可以在 stages 直接引用（$AWSSECRETACCESSKEY 和 $AWSACCESSKEYID）。

注意：如果在 steps 中使用 echo $AWSACCESSKEY_ID，此时返回的是 **，加密内容不会被显示出来。

②用户名密码

本示例用来演示 credentials 账号密码的使用，比如使用一个公用账户访问 Bitbucket、GitLab、Harbor 等，假设已经配置完成了用户名密码的 credentials，凭证 ID 为 jenkins-bitbucket-common-creds。

可以用以下方式设置凭证环境变量：

```
environment {    BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')}
```

这里实际设置了下面的 3 个环境变量：

- BITBUCKETCOMMONCREDS，包含一个以冒号分隔的用户名和密码，格式为 username:password。
- BITBUCKETCOMMONCREDS_USR，仅包含用户名的附加变量。
- BITBUCKETCOMMONCREDS_PSW，仅包含密码的附加变量。

此时，调用用户名密码的 Jenkinsfile 如下：

```

```

注意：此时环境变量的凭证仅作用于 stage 1。

**处理参数**

声明式流水线的参数支持开箱即用，允许流水线在运行时通过 parametersdirective 接受用户指定的参数。

如果将流水线配置为 BuildWithParameters 选项用来接受参数，那么这些参数将会作为 params 变量被成员访问。假设在 Jenkinsfile 中配置了名为 Greeting 的字符串参数，可以通过 ${params.Greeting} 访问该参数，比如：

```

```

对应的脚本式流水线如下：

```
```

**处理失败**

声明式流水线默认通过 postsection 支持健壮的失败处理方式，允许声明许多不同的 postconditions，比如 always、unstable、success、failure 和 changed，具体可参考 Pipeline 的语法。

比如，以下是构建失败发送邮件通知的示例：

```

```

对应的脚本式流水线如下：

```

```

**使用多个代理**

流水线允许在 Jenkins 环境中使用多个代理，这有助于更高级的用例，例如跨多个平台执行构建、测试等。

比如，在 Linux 和 Windows 系统的不同 agent 上进行测试：

```

```

本文节选自《再也不踩坑的 Kubernetes 实战指南》。
