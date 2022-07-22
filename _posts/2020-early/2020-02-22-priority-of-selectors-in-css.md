---
title: CSS中选择器的优先级
categories: [Tech]
tags: [css, selector, html]
date: 2020-02-22
---

CSS 选择器很灵活，弄不懂它的优先级可能会被坑的很惨。

<!-- more -->

CSS 选择器的优先级官方的说法应该叫特殊性（Specificity），特殊性越高，自然优先级越高。下面是特殊性说明：

1. `！important` 特殊性最高，详情访问重要性
2. 对于内联样式，加`1000`
3. 对于选中器中给定的 ID 属性值，加`0100`
4. 对于选择器中给定的类属性值，属性选择或伪类，加`0010`
5. 对于选择器中给定的元素选择器和伪元素，加`0001`
6. 结合符和通配符选择器对特殊性没有任何贡献，加`0000`

用图片表示就是这样的：

![css specificity](https://raw.githubusercontent.com/tobyqin/img/master/2020-02/20180527004805952)

或者这样的：

![css selector priority](https://raw.githubusercontent.com/tobyqin/img/master/image-20200222175158998.png)

再补充一个实际的计算例子：

![CSS selector](https://raw.githubusercontent.com/tobyqin/img/master/image-20200222175423591.png)

或者这个例子：

![css-selector-example](https://raw.githubusercontent.com/tobyqin/img/master/css-selector-example.png)
