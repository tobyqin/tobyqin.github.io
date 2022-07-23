---
title: 文档站点生成工具
categories: [Tech]
tags: [docs, generator, js]
date: 2020-02-22
---

写代码总是要维护文档的，最好文档和代码是在一起的。

<!-- more -->

这时候比较好的解决方案就是 Markdown 了，然后借助工具自动生成文档站点。

## GitBook

这可能是用户量最大的方案了，官方还提供了免费的托管服务，如果你的项目是开源的话可以考虑。但听说官方商业化后对免费用户不是很友好，比如插件或者命令行的支持等等。

官网：https://www.gitbook.com/

![image-20200222154308518](https://image.tobyqin.cn/image-20200222154308518.png)

## docsify

最轻量的解决方案，你只要引用一下它的 js 文件到你的主页，外加一些配置就可以渲染，这是我个人最喜欢的工具之一。docsify 最大的特点是不需要编译，实时渲染 Markdown。

官网：https://docsify.js.org/

![image-20200222154600499](https://image.tobyqin.cn/image-20200222154600499.png)

## vuepress

跟 GitBook 非常相似的，主题和插件也很丰富，不仅可以做文档，还可以做博客。

官网：https://vuepress.vuejs.org/

![image-20200222155018511](https://image.tobyqin.cn/image-20200222155018511.png)

## sphinx

Python 文档的最佳搭档，可以识别 Python 中的方法注释，非常强大。市面上大多数的 Python 工具的文档都是用它生成的。

官网：https://www.sphinx-doc.org/

![image-20200222160704364](https://image.tobyqin.cn/image-20200222160704364.png)
