---
title: 在 MacOSX 上 准备 Jekyll 环境的坑
categories: [Tech]
tags: [Jekyll,Ruby,Bundler,Blog]
date: 2019-04-28
---

Jekyllrb还是有必要了解一下。

<!-- more -->

## 错误信息

弄了半天把Ruby和Gem环境弄好，运行 `bundle install` 后报错。

```
$ bundle install                                                                                                                                                         Traceback (most recent call last):
    1: from /Users/ivolooser/gems/bin/bundle:23:in `<main>'
/Users/ivolooser/gems/bin/bundle:23:in `load': cannot load such file -- /usr/local/lib/ruby/gems/2.6.0/gems/bundler-1.17.2/exe/bundle (LoadError)
```

## 解决办法

把需要的Bundle版本全都给它装上，现在错误的是没有1.17.2，那么开装：

```
$ gem install bundler -v '1.17.3'                                                                                                                                        Successfully installed bundler-1.17.3
Parsing documentation for bundler-1.17.3
Done installing documentation for bundler after 3 seconds
1 gem installed
```

再运行 `bundle install`，还是报错。

```
Can't find gem bundler (= 1.11.2) with executable bundle (Gem::GemNotFoundException)
```

应该是`Gemfile.lock`里写死了指定版本的Bundler才可以运行网站，那么再装：

```
$ gem install bundler -v '1.11.2'                                                                                                                                        Successfully installed bundler-1.11.2
Parsing documentation for bundler-1.12.2
Done installing documentation for bundler after 3 seconds
1 gem installed
```

再来运行 `bundle install`，搞定。

```
$ bundle install
...
Bundle complete! 10 Gemfile dependencies, 30 gems now installed.
```
