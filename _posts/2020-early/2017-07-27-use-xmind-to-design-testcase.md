---
title: xmind2testlink - 快速设计测试案例并导入TestLink
categories: [Tech]
tags: [xmind,testlink,QA,xmind2testlink]
date: 2017-07-27
---

前面我有介绍过思维导图和xmind，现在我们再往前一步，让生活再美好一些。

<!-- more -->

> 上集回顾：[你听说过思维导图吗？](https://tobyqin.github.io/posts/2017-07-26/the-power-of-mindmap/)
>
> 原文链接： <https://tobyqin.github.io/posts/2017-07-26/the-power-of-mindmap/>

本文我们将使用xmind设计测试案例，并导入到TestCase管理系统TestLink。

## 原理

Xmind生成的思维导图以 .xmind 为扩展名，其实这本质上是一个zip压缩文件。这里略带一点小知识。

> zip这一世界通用压缩标准是美国20世纪80年代著名程序员Phil Katz 发明的。当时为了对抗商业压缩软件ARC（缺钱买不起），Philip Katz 制作出了PKZIP，因为免费而且开放，越来越多的软件都内嵌支持zip，包括Windows操作系统。
>
> 你可以使用任何一种文本编辑器打开zip文件，文件的头两个字母为 PK。

xmind解压以后，里面主要由一些xml文件构成，解析content.xml 和 comment.xml 就可以获得思维导图的结构和主要文字内容。

## xmind2testlink

这是一个我使用Python实现对xmind进行解析的PyPI包，有了它你可以很方便地将xmind转化成其他系统使用的格式，比如TestLink。

### 安装和使用

使用pip可以快速安装xmind2testlink。

```
pip install xmind2testlink -U
```

安装后默认就提供了命令行转换功能，可以将xmind转成可以导入testlink的xml文件。

```
xmind2testlink /path/to/testcase.xmind
Generated: testcase.xml
```

如果你想自己编程使用中间对象，可以导入xmind_parser或者testlink_parser中的方法。

```python
from xmind2testlink.xmind_parser import *
from xmind2testlink.testlink_parser import *

# do your stuff
```
### 使用须知 v1

并不是所有的xmind都可以顺利被xmind2testlink识别，因为我是按照一定规律去分析xmind结构的，所以如果你要使用这个小工具，那么请你遵循一些简单的游戏规则。

![xmind2testlink_v1](https://github.com/tobyqin/xmind2testlink/raw/master/web/static/guide/xmind2testlink_v1.png)

输出结果：

![xmind2testlink_v1_out](https://github.com/tobyqin/xmind2testlink/raw/master/web/static/guide/xmind2testlink_v1_out.png)

如图，你的xmind应该和上图结构一致：

1. 默认的中心主题不会被转换，默认从第一层子主题开始转换。
2. 第一层子主题会被识别为 TestSuite。
3. TestSuite 的子主题 会被识别为 TestCase。
4. TestCase 的下级分支为 TestStep 和 Expected Result。
5. 你可以给 TestSuite，TestCase加上 Note，这会被识别为Summary 字段。
6. 你可以给TestCase 加上 Comment，这会被识别为 Preconception 字段。
7. 你可以使用数字Marker来为TestCase定义优先级。
8. 你可以使用感叹号`!`来注释掉不想导入的任意分支。


如果觉得太复杂了，可以下载示例的xmind文件（[Test case by xmind v1.xmind](https://github.com/tobyqin/xmind2testlink/blob/master/web/static/guide/test_case_by_xmind_v1.xmind)），看一眼就懂了。

### 使用须知 v2

在使用V1的规则一段时间后，发现不是特别xmind，xmind真正强大的地方在于发散思维整理，如果按照前面的规则使用xmind，会有很大的限制，于是我升级了xmind2testlink，称之为V2。看图：

![xmind2testlink_v2](https://github.com/tobyqin/xmind2testlink/raw/master/web/static/guide/xmind2testlink_v2.png)

输出结果：

![xmind2testlink_v2_out](https://github.com/tobyqin/xmind2testlink/raw/master/web/static/guide/xmind2testlink_v2_out.png)

基于V1，补充的规则如下：

1. 根主题必须加上一个小星星，这是用来区分V1和V2的标识。
2. 第一层子主题还是会被识别为 TestSuite。
3. 之后的主题可以自由扩展，如果一个主题被标记了priority那么意味着case到此结束。
4. 如果没有主题被标记priority，默认case取到最后一个主题。
5. 默认使用空格连接case子主题，你可以指定其他连接符（根主题的最后一个字符）。
6. TestCase 的下级分支为 TestStep 和 Expected Result。
7. 所有case子主题的Summary和Preconception会被连接起来。
8. 你可以给 TestSuite，TestCase加上 Note，这会被识别为Summary 字段。
9. `!`开头的所有主题都会被自动忽略，可以用来隐藏或者注释某些不想导入的内容。

照旧，这里有一个示例文件（[Test case by xmind v2.xmind](https://github.com/tobyqin/xmind2testlink/blob/master/web/static/guide/test_case_by_xmind_v2.xmind)），看一下就明白了。其实Github上的[英文文档](https://github.com/tobyqin/xmind2testlink)描述更清楚一下，有能力的你还是去看一下。

### 进阶用法

可能不是每个人都了解Python或者安装了Python，那么这是你可以将xmind2testlink部署成一个网站，步骤也非常简单。

```
# clone this git repo ahead
cd /path/to/xmind2testlink/web
pip install -r requirements.txt -U
python application.py

* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
* Restarting with stat
```

这时你启动浏览器就可以看到一个web版的转换界面。

![xmind2testlink web](https://raw.githubusercontent.com/tobyqin/xmind2testlink/master/web/static/guide/web.png)

这是一个由Flask写的简单程序，你可以将其部署到专门的服务器，详情请查阅官方文档。

## 小结

其实在实现一个小工具的过程中，从构思想法到实现，有很多内容和未知需要去探索。xmind2testlink 涉及到的知识点也不少，比如 PyPI 打包发布，python读取zip文件，解析xml，Flask，网站前后期，服务器部署，持续集成，单元测试等等，我个人收获不小。

如果你工作或生活也有各种想法，不如动手去做，失败了没啥大不了的，万一成功了呢。