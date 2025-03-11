---
title: 通过 Github Actions 自动发布 Hexo 博客
categories: [Tech]
tags: [Hexo, GitHub-Actions, GitHub, CICD]
date: 2019-12-25
---

Github 今年推出了自己的 CI 集成方案 [Github Actions](https://github.com/features/actions)，本着玩一玩不吃亏的态度，我把原来通过 [Travis CI](https://travis-ci.org/) 的自动发布流程迁移到了 Github Actions，整个过程还是非常愉快顺利的。

<!-- more -->

## 创建博客

这部我就不展开说了，直接到 [Hexo 官网](https://hexo.io/)参考文档就可以快速开始。

我假设你已经有这么一个博客了，而且也成功手动发布过。

## 生成密钥

为了安全起见，我们为此次发布单独创建一对密钥，在本地命令行执行如下命令。

```shell
$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/github-actions-deploy
```

一路回车，生成的公钥为 `github-actions-deploy.pub`，私钥为 `github-actions-deploy`。

## 安排密钥

假设你的 Github 源文件仓库是 `blog`，静态页面仓库是 tobyqin.github.io。那么你需要将公钥配置到静态页面仓库的 `Deploy keys`，将私钥配置到源文件仓库的 `Secret`。

- `blog` > `Secrets` > `Add a new secret` > 添加密钥，命名为 `ACTION_DEPLOY_KEY`
- `tobyqin.github.io` > `Deploy keys` > `add deploy key` > 添加公钥，名字随意，允许写入权限。

## 配置博客

这一步主要是确保你的博客能够发布到正确的仓库，参考如下配置。

```yml
# _config.yml
## Docs: https://hexo.io/docs/deployment.html
deploy:
  - type: git
    repo: git@github.com:tobyqin/tobyqin.github.io.git
    branch: master
```

## 配置 Github Actions

好戏开场，切到你的`blog`仓库，选择 `Actions` 选项卡，新建一个 `Workflow`。

![image-20191225230141168](https://image.tobyqin.cn/image-20191225230141168.png)

你可以选用某个模板，比如 `Node.js`，或者完全自定义。针对我自己的博客，因为我在发布前还写了个 Python 的脚本做了一些额外的事情，所以我的 `Workflow` 大概是这样的。

```yaml
name: Deploy Blog

on: [push]

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v1

      - name: Use Python 3.x
        uses: actions/setup-python@v1
        with:
          python-version: "3.7"

      - name: Use Node.js 10.x
        uses: actions/setup-node@v1
        with:
          node-version: "10.x"

      - name: Setup
        env:
          ACTION_DEPLOY_KEY: ${{ secrets.ACTION_DEPLOY_KEY }}
          TZ: Asia/Shanghai
        run: |
          # set up private key for deploy
          mkdir -p ~/.ssh/
          echo "$ACTION_DEPLOY_KEY" | tr -d '\r' > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts

          # set git information
          git config --global user.name 'Toby@Github' # 换成你自己的邮箱和名字
          git config --global user.email 'toby.qin@live.com'

          # prepare blog
          pip install -r requirements.txt
          python blog.py prepare

          # install dependencies
          npm install -g hexo-cli
          npm install

      - name: Deploy
        run: |
          # publish my blog
          hexo clean
          hexo generate
          hexo deploy
```

可以看到这个`Workflow`的脚本还是很好理解的，先是起了一个名字，然后选择了 ubuntu 最新版作为运行系统，接着安装了 Python 和 Node.js，然后执行了一段脚本做环境配置，这里面既有 Python 又有 Node.js，最后执行了发布命令。

## 异常处理

当然这个 Workflow 我也不是一次就执行成功的，如果你需要调试的话就可以到 Github Actions 这个选项卡去看执行日志，非常的详尽和易读。

![image-20191225232123775](https://image.tobyqin.cn/image-20191225232123775.png)

## 小结

Github 终于自己动手做 CI 了，让各大友商瑟瑟发抖。而且我体验下来非常棒，比 Travis CI 集成度更好，而且 Action Workflow 使用 Yaml 来定义也十分清晰友好。

相对 Jenkins 的 Pipeline 可能少了一些图形化的支持，但功能毫不逊色。而且一个仓库是允许定义多个 Workflow 的，每个 Workflow 可以有不同的目的和触发机制，在 Workflow 中官方也提供了类似插件一样的功能，十分灵活。
