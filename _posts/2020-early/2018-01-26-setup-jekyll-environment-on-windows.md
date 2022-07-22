---
title: 在Windows上搭建Jekyll运行环境
categories: [Tech]
tags: [Ruby,Jekyll,Windows,Hexo]
date: 2018-01-26
---

静态页面博客的鼻祖就是Jekyll。

<!-- more -->

## Jekyll 和 Hexo

静态博客目前最流行的也就是 Hexo 和 Jekyll，我一直都是 Hexo 的粉丝和用户，相对于 Jekyll 我想大多数人选择 Hexo 的原因跟我差不多：

1. 安装本地环境简单，只需要 nodejs 和一行命令即可
2. 官网文档非常优秀，极易上手
3. 众多优秀的主题可选，配置功能完善
4. 目录结构清晰易懂，内容和配置分离

简单来说，Hexo 可以让你快速开始写文字，但是 Jekyll 却不见得，Jekyll 的罪状我大致列一下：

1. 本地环境在 Windows 上坑超多，很多人在这一步就放弃了，官方文档很挫
2. 没有专门的主题目录，导致使用新主题需要覆盖安装
3. 脚本和内容混合存放，不易管理和使用
4. 托管后部署如果有错无法获取报错信息，如果没有本地环境根本没法调查

另外，据说 Jekyll 站点的生成效率也比 Hexo 慢很多，不过我相信这对大多数人都没影响，我们的文章数量也不多。

但是，Jekyll 比 Hexo 有一个让你欲罢不能的优点，那就是天然被 GitHub Pages 支持，当然国内的 Coding Pages，OSC Pages 等等产商也直接支持 Jekyll，俨然这就是一个静态页面的标准。

使用 Jekyll，你可以不用考虑编译生成的问题，写完直接 commit，几分钟后就能看到效果。然而使用Hexo，你要么在本地生成页面后上传，要么找一个持续集成的服务（比如 [Travis CI](https://travis-ci.org/)）帮你编译后上传，不是很简单。

因为 Jekyll 那个欲罢不能的原因，今天就说说如果在 Windows 上用最快的速度准备一个Jekyll环境。

参考资料： https://jekyllrb.com/docs/windows/

## 通过Bash准备Jekyll环境

如果你是 Windows 10，可以考虑使用 Bash 来运行Linux脚本和软件，详见 [Bash on Ubuntu on Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about)。

启动 Bash 后，运行以下命令即可完成 Jekyll 的准备工作：

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo apt-get install ruby2.3 ruby2.3-dev build-essential
sudo gem update
sudo gem install jekyll bundler
```

可能遇到的坑：

1. 在 Windows 10 启用 Bash 并不容易，不是所有系统版本都支持

2. 就算版本支持，你的 Windows 如果是盗版或者不完整也有可能装不上

3. 就算你能装上，这个 Bash 需要 Linux 子系统是一个大家伙，会吃掉你系统盘很多空间

4. 就算你系统盘有足够的空间，从服务器下载数GB的文件网络不一定稳定，电脑还不能随便关机


不就是个Jekyll吗？至于嘛，Bash on Windows听起来简单，做起来就难了。

## 通过 Ruby Installer 安装

Ruby 的安装包可以从 https://rubyinstaller.org/ 下载安装，如果你了解Ruby或者运气很好，选择了正确Ruby版本，那么恭喜你，一切会非常顺利。

不废话，直接说最佳答案。

1. 下载Ruby 2.3.3 x64 后双击运行安装。
2. 下载Development KIT x64 后解压到 c:\devkit 目录
3. 打开命令行工具，运行：


```bash
cd c:\devkit
ruby dk.rb init
ruby dk.rb review
ruby dk.rb install
```

4. 安装Jekyll

```bash
gem install jekyll bundler
```

## 在本地运行 Jekyll

更多命令建议参考官方文档 https://jekyllrb.com/docs/quickstart/

```bash
# 创建一个新博客 ./myblog
jekyll new myblog

# 进入新建博客目录
cd myblog

# 安装必要的依赖
bundle

# 在本地启动预览
jekyll serve

# 打开浏览器 http://localhost:4000 即可看到效果
```

## 可能遇到的坑

### 如果有错误信息

错误信息：

```
ERROR:  While executing gem ... (Encoding::UndefinedConversionError)
U+200F to IBM862 in conversion from UTF-16LE to UTF-8 to IBM862
```

解决办法：

从Ruby安装目录找到registry.rb，修改默认Encoding为UTF-8。

```ruby
LOCALE = Encoding::UTF_8
#LOCALE = Encoding.find(Encoding.locale_charmap)
```

错误信息：

```
compiling ruby_http_parser.c
compiling ryah_http_parser.c
linking shared-object ruby_http_parser.so
c:/devkit/mingw/bin/../lib/gcc/x86_64-w64-mingw32/4.7.2/../../../../x86_64-w64-mingw32/bin/ld.exe: cannot find -lgmp
collect2.exe: error: ld returned 1 exit status
make: *** [ruby_http_parser.so] Error 1
```

解决办法：

Ruby版本不对，没有gmp这个库。建议卸载掉重装2.3.3。

### 其他的一些问题

1. 不要安装Ruby2.5，除非你有强烈的需求。
2. Ruby2.4 以后，在安装完成后会让你选择 MSYS2 Devkit，但是国内的网络环境你基本上装不了。
3. Ruby2.4 以后，你可以通过 `ridk  install` 安装 Devkit，国内网络环境，难。
4. Ruby2.4 以后，缺少Jekyll需要的依赖，目前我没办法解决。

综上，不要使用Ruby2.4来准备Jekyll环境，不然你会很难过。

## 小贴士&小技巧

很多朋友可能不知道，如果你已经初始化了Hexo或者Jekyll，后面写博时就可以完全抛开本地环境，专注于内容就行。

如果你是Hexo，安装完成后并且配置了自动化部署，以后写博客只需要在`/source/_posts/`目录下新建Markdown文件，完成后提交即可。新建的文章样式可以直接从该目录复制后修改，或者你从模板目录 `/scaffolds/` 中拷贝过来也行。

如果你是Jekyll，直接从`/_posts/`新建文件即可，操作同上。

新博客的文件名最好也都符合`yyyy-mm-dd-name.md`的样式，主要是方便排序和查阅，另外Jekyll也只认识这种样式。