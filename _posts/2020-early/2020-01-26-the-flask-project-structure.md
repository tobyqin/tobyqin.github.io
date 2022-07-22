---
title: 组织Flask项目结构
categories: [Tech]
tags: [python, flask]
date: 2020-01-26
---

![organizing](https://img.tobyqin.cn/organizing.png)

[Flask](https://palletsprojects.com/p/flask/) 是非常轻量和灵活的 Python 框架，轻量和灵活是它的优点，也是它的缺点。所以我们在使用 Flask 构建项目时就不得不慎重考虑其目录结构，以便日后扩展和维护。

<!-- more -->

这里我列举了一些常见的 Flask 项目结构，没有好坏之分，大家可以按照实际情况参考使用。

## 极简风格

```
app.py
config.py
requirements.txt
static/
templates/
```

此项目结构可以用于构建最简单的 Web 程序，一般用于 Demo 或者 POC。

## 使用 App 组织项目

相对复杂的项目可以按包（package）的方式来组织代码，不同的包对应不同的应用（app），每个应用相对独立，其中会有自治的视图（view）和模型（model）等等。

```
config.py
requirements.txt
run.py
instance/
    config.py
yourapp/
    __init__.py
    views.py
    models.py
    forms.py
    static/
    templates/
```

其实 Flask 项目做到这个程度已经差不多了，如果项目再复杂，就不太推荐使用 Flask 了，而是考虑换成[django](https://www.djangoproject.com/)或者其他更适合做大型项目的框架，死磕 Flask 最后只会适得其反。

欲知更多，可参考:

- http://exploreflask.com/en/latest/organizing.html
- https://lepture.com/en/2018/structure-of-a-flask-project
- http://flask.pocoo.org/docs/1.0/patterns/packages/
- https://www.digitalocean.com/community/tutorials/how-to-structure-large-flask-applications

## 使用 Blueprints 组织项目

更加复杂的项目可以引入 [Blueprints](http://docs.jinkan.org/docs/flask/blueprints.html) 来简化工作，这是官方推荐的 Flask 大型项目解决方案。一个 Blueprints 对象和一个 Flask 对象很类似，所以有了 Blueprints 之后你可以很方便的将大型应用拆分成多个子项目来开发和加载。Blueprints 需要注册后才能被加载，有点像插件。

Blueprints 解决了应用拆分的可能性，但怎么拆分和组合还是开发者的事情，下面几个例子可以参考一下。

### 基于逻辑功能

```
yourapp/
    __init__.py
    static/
    templates/
        home/
        control_panel/
        admin/
    views/
        __init__.py
        home.py
        control_panel.py
        admin.py
    models.py
tests/
```

### 基于职能模块

```
config.txt
requirements.txt
run.py
yourapp/
  __init__.py
  home/
    views.py
    static/
    templates/
  dash/
    views.py
    static/
    templates/
  admin/
    views.py
    static/
    templates/
  api/
    views.py
    static/
    templates/
  blog/
    views.py
    static/
    templates/
  models.py
tests/
```

### 静态模板的组织

```
facebook/
    __init__.py
    templates/
        layout.html
        home/
            layout.html
            index.html
            about.html
            signup.html
            login.html
        dashboard/
            layout.html
            news_feed.html
            welcome.html
            find_friends.html
        profile/
            layout.html
            timeline.html
            about.html
            photos.html
            friends.html
            edit.html
        settings/
            layout.html
            privacy.html
            security.html
            general.html
    views/
        __init__.py
        home.py
        dashboard.py
        profile.py
        settings.py
    static/
        style.css
        logo.png
    models.py
```

对于 Flask 应用中的静态模板，我觉得在现代应用中还需要三思。因为大多数的现代应用都会考虑用 nodejs 构建前端，使用模板语言已经属于异教徒，后期的维护和更新更是挖坑填坑的过程。

## 小结一下

对于要快速见效的项目，用 Flask 还是不错的选择，例如做个页面收集数据或者展示图表，再或者模拟几个 API 用于测试等等。但是大点的项目还是算了吧，真的，不骗你，要填的坑远比你想象的多的多。
