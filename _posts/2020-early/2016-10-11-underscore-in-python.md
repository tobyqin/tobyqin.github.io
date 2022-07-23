---
title: Python中的下划线和魔法方法
date: 2016-10-11 22:26:46
tags: [python, python underscore, tips]
categories: Tech
---

下划线在 Python 中有很特别的意义。

<!-- more -->

## 开门见山

下划线在 Python 中有特殊的意义，简单来说，可以总结成三点。

1. 单下划线在前一般用于声明私有成员，比如 `_private_var`
2. 单下划线在后一般用于命名已经被保留关键字占用的变量，比如 `class_`,`type_`
3. 双下划线一般被用于 Python 内置的特殊方法或者属性，比如 `__name__`,`__file__`，有时候也被称之为魔法方法。

更多细节的讨论，可以看 StackOverflow 上的这个主题：[What is the meaning of single and double underscore before an object name?](http://stackoverflow.com/questions/1301346/the-meaning-of-a-single-and-a-double-underscore-before-an-object-name-in-python)。

> `__foo__`: this is just a convention, a way for the Python system to use names that won't conflict with user names.
>
> `_foo`: this is just a convention, a way for the programmer to indicate that the variable is private (whatever that means in Python).
>
> `__foo`: this has real meaning: the interpreter replaces this name with `_classname__foo` as a way to ensure that the name will not overlap with a similar name in another class.
>
> No other form of underscores have meaning in the Python world.
>
> There's no difference between class, variable, global, etc in these conventions.

有时候我们还能看到就仅仅命名为一个下划线的的变量，这种情况一般是这个变量不重要或者只是一个临时工，连名字都不配拥有。

```python
for _,value in func(): # 假如func每次会返回两个值，我们只关心第二个值
    use(value)
```

## 思维导图

下面是思维导图的总结

![underscore-in-python](https://image.tobyqin.cn/underscore-in-Python.png)

## 如何调用魔法方法

一些魔法方法直接和内建函数对应，这种情况下，如何调用它们是显而易见的。这有个附录可以作为调用魔法方法的参考。

| 魔法方法                          | 什么时候被调用                     | 解释                       |
| --------------------------------- | ---------------------------------- | -------------------------- |
| `__new__(cls [,...])`             | `instance = MyClass(arg1, arg2)`   | `__new__`在实例创建时调用  |
| `__init__(self [,...])`           | `instance = MyClass(arg1,arg2)`    | `__init__`在实例创建时调用 |
| `__cmp__(self)`                   | `self == other`, `self > other` 等 | 进行比较时调用             |
| `__pos__(self)`                   | `self`                             | 一元加法符号               |
| `__neg__(self)`                   | `-self`                            | 一元减法符号               |
| `__invert__(self)`                | `~self`                            | 按位取反                   |
| `__index__(self)`                 | `x[self]`                          | 当对象用于索引时           |
| `__nonzero__(self)`               | `bool(self)`                       | 对象的布尔值               |
| `__getattr__(self, name)`         | `self.name` #name 不存在           | 访问不存在的属性           |
| `__setattr__(self, name)`         | `self.name = val`                  | 给属性赋值                 |
| `__delattr__(self, name)`         | `del self.name `                   | 删除属性                   |
| `__getattribute__(self,name)`     | `self.name`                        | 访问任意属性               |
| `__getitem__(self, key)`          | `self[key]`                        | 使用索引访问某个元素       |
| `__setitem__(self, key)`          | `self[key] = val`                  | 使用索引给某个元素赋值     |
| `__delitem__(self, key)`          | `del self[key]`                    | 使用索引删除某个对象       |
| `__iter__(self)`                  | `for x in self`                    | 迭代                       |
| `__contains__(self, value)`       | `value in self, value not in self` | 使用 in 进行成员测试       |
| `__call__(self [,...])`           | `self(args)`                       | “调用”一个实例             |
| `__enter__(self)`                 | `with self as x:`                  | with 声明的上下文管理器    |
| `__exit__(self, exc, val, trace)` | `with self as x:`                  | with 声明的上下文管理器    |
| `__getstate__(self)`              | `pickle.dump(pkl_file, self)`      | Pickling                   |
| `__setstate__(self)`              | `data = pickle.load(pkl_file)`     | Pickling                   |

如果你还想了解关于魔法方法的更多细节，那么你一定不能错过：

- [魔法方法指南](http://pyzh.readthedocs.io/en/latest/python-magic-methods-guide.html)
- [Magic method guide](http://www.rafekettler.com/magicmethods.html)
