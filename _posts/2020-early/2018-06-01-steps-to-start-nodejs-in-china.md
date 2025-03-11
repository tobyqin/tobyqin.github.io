---
title: NodeJS起步两三事
categories: [Tech]
tags: [nodejs,mirror,proxy]
date: 2018-06-01
---

主要是为了备忘，开始接触NodeJS有一段时间，断断续续，年纪也大了时间一长容易忘事情，汗。

<!-- more -->

## 安装Node

直接到[官网下载](https://nodejs.org/en/download/)LTS版本安装即可，没必要追新功能用最新版。安装Node基本没什么坑，记得加到PATH就好。

Windows双击安装，macOS推荐使用brew安装，完成后在命令行里测试一下。

```shell
$ node -v
```

后期如果没有啥breaking的API改动基本也不用升级。

## 必要的配置

NodeJS的包管理器`npm`，如果在墙内及有可能在使用过程中很不稳定，一般推荐使用国内的镜像源，目前最知名的也就是淘宝家的了。

>  https://npm.taobao.org/

### .npmrc

你可以通过修改`~/.npmrc`来设置默认的包源:

```ini
registry=https://registry.npm.taobao.org/
```

### cnpm

你也可以安装 `cnpm` 来代替 `npm` ：

```
npm install cnpm -g
```

之后的大部分`npm` 都可以直接用 `cnpm` 代替（发布相关的除外）：

```
cnpm install <package>
```

### nrm

其实我还推荐你了解另外一种切换源的方式 `nrm`:

```
nmp install nrm -g
```

使用方法如下，超级简单：

```
$ nrm ls

* npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
  
$ nrm use taobao

  Registry has been set to: https://registry.npm.taobao.org/

```

使用 `nrm ls` 列出可以切换的源，然后`nrm use <name>` 瞬间切换，爽！

### proxy

镜像源毕竟是copy的，同步状态有可能你不甚满意，最近taobao的源可能还有以下错误：

```
npm ERR! registry error parsing json
```

作为一个资深码农你也许有一个本地代理（SS懂？）让你无障碍访问国际互联网，那么你可以这么做：

```
npm config set proxy http://server:port
npm config set https-proxy http://server:port
```

配置好 `npm` 的代理后你又可以开心地玩耍了。

## Node入门须知

三分钟入门NodeJS，如果你已经有其他语言的编程经验，是可以的。

NodeJS的核心可执行程序`node[.exe]`你可以简单理解成代码解释器，类似于Java的虚拟机，C#的.net Framework，Python里的`python[.exe]`，作用是把你的代码翻译成计算机行为。

```
node hello.js
```

以上就是运行NodeJS代码的不二法则。

### 项目启动

很多知名框架都会提供所谓的脚手架命令，但是对于萌新玩家，我是不建议直接去使用脚手架的，这些脚手架做出来的目录结构对于萌新玩家来说是懵逼的，虽然有可能都是最佳实践的结果，但没有1，2，3步骤和手拉手，萌新也许直接退出游戏了。

新手真正的入门法则还是踏踏实实按照各种教程一步一步走，如果教程不好就换，没学会走路就想跑是不可能的。一般项目是这样开始的：

1. 明确项目需求（略）
2. 创建项目目录
3. 初始化项目信息
4. 安装必要的依赖
5. 模块划分，编码
6. 系统集成，测试
7. 项目发布，维护升级

这个流程用NodeJS来实践大致是这样的：

```shell
# create project 
mkdir [project-name]
cd [project-name]

# init project
npm init
...

# install dependencies
npm install <package...>

# install dev dependencies
npm install <package...> -D

# coding
...

# run and test
npm run build
npm run start
npm run test
npm run stop

# publish project
npm publish .
```

前面一节全都是在介绍`npm`，通过这个例子你应该明白它在NodeJS中的重要性了吧，所以让你的`npm`好用意义非凡。

例子中安装了两次依赖可能你有点困惑，可以这样理解，第一次依赖是项目运行时必要的依赖`--save-prod`，在发布时必须安装的；第二次依赖主要是用于开发或者测试的`--save-dev`，比如某些debug包或者test框架，方便开发才需要的包，在项目部署时没必要安装。其实还可能安装第三种依赖`--save-optional`，比如是用来做数据分析或者别的enhancement的，目前Node支持区分对待这三类依赖包。

### 语法演示

NodeJS里的JS就是JavaScript，属于动态语言，命名规则类似Java，但是语法更接近各种动态语言，比如Python或者Ruby。上手不难，了解一下标准库和数据基本类型，在IDE的加持下你就可以开始写代码了。

```js
// index.js 项目的入口文件

var fs = require('fs');  // 导入标准库模块
var _ = require('lodash'); // 导入已安装的包

var m = require('./hello');  // 导入同目录下的模块
import hello from './test'; // 导入模块的部分对象

m.say('yo..'); // 调用模块方法
hello('toby');

fs.copyFileSync(src,dst) // 调用标准库方法
```

模块的编写范例：

```js
// hello.js

var a = 'hello world!'; // 模块的全局变量不会被导出

function say(word) {
    console.warn('say ' + word);
}

function hello(word) {
    console.log('hello, ' + word);
}

// 导出就是对 module.exports 进行赋值
module.exports = {
    say: say,
    hello: hello
}
```

## 包管理小贴士

对于`npm`包的安装的位置我们来了解一下，简单来说就是这个包所作用的范围。如果安装包不带`-g`参数那么默认就是安装在当前目录。

```
npm install <package>
# => will install to ./node_modules
```

这个包也就只有在当前目录（项目）里可用，这样方便做到环境隔离，不同项目可以用同样包的不同版本等等。

### -g

如果安装过程中带了`-g`参数那么就意味这个包是全局（global）安装的，在系统的任何位置都是可用的。

```
npm install <package> -g
# => will install to /usr/local/lib/node_modules
# => or %AppData%\npm\node_modules
```

一般的包安装都是不用带`-g`的，除了一些工具类的包，例如第一节介绍那些，这样系统干净一些，也可以避免全局包污染项目导致各种灵异事件。

### --depth

如果要查看安装了哪些global包可以用这个命令：

```
npm list -g --depth=0
```

如果不加`--depth`，哎我去真没法看，也不知设计这个`list`的人为啥不默认带上这个参数，脑子被门夹了。为了方便你可以配上以下两个alias。

```
alias ng="npm list -g --depth=0 2>/dev/null"
alias nl="npm list --depth=0 2>/dev/null"
```

### ncu

使用NodeJS就是要面对各种各样的包，有些包升级很勤快，你也想升级怎么办呢？哪些是outdated哪些不是，`npm`的升级命令真的很难用（记）哎。我推荐你了解一下`npm-check-updates`：

```
npm install npm-check-updates -g
```

这么好用的东西当然要全局安装啦，用法很简单，检查当前项目所有包的更新状态：

```
$ cd my-project
$ ncu
Using /Users/tobyqin/src/blog/package.json
[..................] | :
The following dependencies are satisfied by their declared version range, but the installed versions are behind. You can install the latest versions without modifying your package file by using npm update. If you want to update the dependencies in your package file anyway, run ncu -a.

 hexo                   ^3.5.0  →  ^3.7.1
 hexo-generator-search  ^2.1.1  →  ^2.2.5
 hexo-server            ^0.3.1  →  ^0.3.2
```

使用 `ncu -a`就可以一键更新项目里的所有包，当然它也提供了一些过滤参数，你可以[查阅文档](https://www.npmjs.com/package/npm-check-updates)。如果要看全局的包有没有可以更新的，试试`ncu -g`，非常方便。

有一点要注意，虽然命令是`ncu`但是包名不是，因为`ncu`的包名已经被一个天气预报的包占用了，无语。

## 总结

现在写NodeJS项目已经很容易了，JS经过了两三年的野蛮生长，诞生了成千上万个包（框架），其中不乏精品。

作为搬砖的码农，经过一顿`npm install`操作就可以做出一个不错的demo，你还犹豫什么，赶快上车吧！