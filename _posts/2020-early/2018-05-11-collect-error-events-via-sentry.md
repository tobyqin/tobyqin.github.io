---
title: Sentry - 处理异常日志的正确姿势
categories: [Tech]
tags: [python, error, logging]
date: 2018-05-11
---

在各种系统和应用里，无论你的代码再完美也还是会抛异常，出错误。今天的主角是当今比较流行的异常记录框架 - Sentry，来了解一下。

<!-- more -->

## 关于日志管理

应用越做越复杂，输出日志五花八门，有 print 的，有写 stdout 的，有写 stderr 的, 有写 logging 的，也有自定义 xxx.log 的。那么这将导致平台应用日志分布在各个地方，无法统一管理。而且可能用的还不止一种开发语言，想规范和统一日志不是一件容易的事。

## 为什么使用 Sentry

Sentry 是一个集中式日志管理系统。它具备以下优点：

- 多项目，多用户
- 界面友好
- 可以配置异常出发规则，例如发送邮件
- 支持主流语言接口

从 Sentry 的文档首页截下来的一张图，可以看到它支持目前主流的编程语言。

![sentry-supported-platform](https://image.tobyqin.cn/sentry-supported-platform.png)

## 安装和快速上手

Sentry 支持部署到本地服务器，具体可以参考以下文档：

- https://docs.sentry.io/server/installation/

但作为大多数个人开发者和中小企业，我更建议使用 Sentry 官网（https://sentry.io/）提供的云服务，你只需要注册一个Sentry账号，就可以快速享受到集中处理异常日志的服务。

Sentry 免费版可以：

- 每月 10k 错误日志上限
- 支持所有平台和语言，功能无缩水
- 无限项目数量，仅单用户访问，不提供团队功能

具体的价格表可以看这里：

- https://sentry.io/pricing/

## 开始配置 DSN

你可以认为 DSN（Data Source Name）是 Sentry 管理项目需要的`PROJECT_ID`，每个应用都需要对应一个 `PROJECT_ID`，以及用于身份认证的 `PUBLIC_KEY` 和 `SECRET_KEY`。由此组成一个这样的 DSN：

```
{PROTOCOL}://{PUBLIC_KEY}:{SECRET_KEY}@{HOST}/{PATH}{PROJECT_ID}
```

PROTOCOL 通常会是 `http` 或者 `https`，HOST 为 Sentry 服务的主机名和端口，PATH 通常为空。

在你登入 Sentry 后台之后，你可以新建一个项目，之后就可以得到类似于下面这样一个 DSN。

```
https://e055040d5@sentry.io/12345
```

有了 DSN 以后，你就可以在客户端中将错误日志上传到 Sentry 了。

## 配置客户端

这里我主要以 Python 为例，其他编程语言的客户端配置可以参考官网文档，步骤大同小异。

- https://docs.sentry.io/quickstart/

首先通过 pip 安装 Sentry SDK。

```
pip install raven --upgrade
```

然后初始化客户端。

```Python
from raven import Client

DSN = 'https://****@sentry.io/****'
client = Client(DSN)
```

最后，在你需要记录异常的代码为止调用`client.captureException()`即可。

```Python
try:
    1 / 0
except ZeroDivisionError:
    client.captureException()
```

很多时候我们的异常信息应该包含更多的上下文信息，这样对于我们做后续分析会有更多帮助，那么你可以在 Sentry 捕获异常前加入这些上下文。

```Python
try:
    processing(user, data)

except:
    client.user_context({
        'user': user.email,
        'data': json.dumps(data)
    })
    client.captureException()
```

### 一些经验之谈

当然，我们不可能在每处可能发生异常的代码为止都调用 Sentry，也不可能去修补过去的代码将 Sentry 一一植入，一个好的建议是，无论何时，你的程序都有统一的异常处理机制，最好是全局的。这样的话，你只要将 Sentry 写在全局的异常处理器即可。

另外 Sentry 还对流行的开发框架提供了特别的支持，比如 Flask，Django 等等，在这些应用中你只要配置就行，不需要你去写什么全局的异常处理（虽然写起来也不难）。

Flask 的例子：

```Python
sentry = Sentry(dsn='http://public_key:secret_key@example.com/1')

def create_app():
    app = Flask(__name__)
    sentry.init_app(app)
    return app
```

Django 的例子：

```Python
import os
import raven

INSTALLED_APPS = (
    'raven.contrib.django.raven_compat',
)

RAVEN_CONFIG = {
    'dsn': 'http://public_key:secret_key@example.com/1',
    # If you are using git, you can also automatically
    # configure the release based on the git info.
    'release': raven.fetch_git_sha(os.path.abspath(os.pardir)),
}
```

## 异常报告和提醒

一旦你完成上面的配置，以后系统发生的所有错误异常都会被自动记录到 Sentry，查看报告就是一件轻松愉快的事情了。

默认情况下，一旦异常发生，5 分钟内就会有一封邮件送到你邮箱，包含了异常信息的大致描述。

![sentry-email-alert](https://image.tobyqin.cn/sentry-email-alert.png)

当然你还可以将异常报警集成到更多系统中，比如 HICHAT，SLACK，IRC，WEBHOOKS，在 Sentry 后台提供了相应的入口。

在 Sentry 的项目 Dashboard 你可以浏览到更详细的报告，比如按照异常信息的类别进行分类和过滤，也可以统计近期异常的状态和频率，非常方便。

![sentry-dashboard](https://image.tobyqin.cn/sentry-dashboard.png)

Sentry 还提供了异常信息的聚合，同样的错误有可能在多处抛出，传统的日志统计起来就不是很方便，在 Sentry 一目了然。

另外你还可以针对异常问题进行分配和跟踪，例如指派团队的某个成员去处理某一类问题，对于长时间没有再发生的问题自动标记为解决等等。

## 总结

Sentry 还有有很多亮点，比如敏感信息过滤， release 版本跟踪，关键字查找，受影响用户统计，权限管理等。Sentry 的 plugin 模块还可以集成大量的第三方工具如： SLACK ， JIRA 。

对我们来说最大的便利就是利用日志进行错误发现和排查的效率变高了。但是，我们能不能完全依赖 Senry 呢？有几点值得探讨：

### 不是日志的替代品

Sentry 的目的是为了让我们专注于系统与程序的异常信息，目的是提高排查问题的效率，日志事件的量到达一个限制时甚至丢弃一些内容。官方也提倡正确设置 Sentry 接收的日志 level 的同时，用户也能继续旧的日志备份。

### 不是排查错误的万能工具

Sentry 是带有一定策略的问题分析工具，以样本的形式展示部分原始日志的信息。信息不全面的同时，使用过程中也可能出现 Sentry 聚合所带来的负面影响，特别是日志记录质量不够的情况下。

### 不是传统监控的替代品

与传统的监控系统相比，Sentry 更依赖于发出的日志报告，而另外一些隐藏的逻辑问题或者业务问题很可能是不会得到反馈的。

## 参考文档

- https://docs.sentry.io/quickstart/
- https://blog.csdn.net/bigsec/article/details/54091109
