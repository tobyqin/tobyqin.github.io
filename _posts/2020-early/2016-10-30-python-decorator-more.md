---
title: Python装饰器的另类用法
categories: Tech
tags: [python, python decorator]
date: 2016-10-30
---

之前有比较系统介绍过 Python 的装饰器（请查阅《[详解 Python 装饰器](https://tobyqin.github.io/posts/2016-10-27/python-decorator/)》），本文算是一个补充。今天我们一起探讨一下装饰器的另类用法。

<!-- more -->

## 语法回顾

开始之前我们再将 Python 装饰器的语法回顾一下。

```python
@decorate
def f(...):
    pass
```

等同于:

```python
def f(...):
    pass

f = decorate(f)
```

@语法的好处在于：

- 相同的函数名只出现一次，避免了`f = decorate(f)`这样的语句。
- 可读性更高，让读代码的人一眼就明白这个函数被装饰了哪些功能。

## @call()装饰器

假设你要创建一个整数平方的列表，你可以这样写：

```python
>>> table = [0, 1, 4, 9, 16]
>>> len(table), table[3]
(5, 9)
```

也可以使用列表表达式，因为比较简单。

```python
>>> table = [i * i for i in range(5)]
>>> len(table), table[3]
(5, 9)
```

但是假如这个列表的逻辑比较复杂的时候，最好是写成一个方法，这样会更好维护。

```python
>>> def table(n):
...     value = []
...     for i in range(n):
...         value.append(i*i)
...     return value
>>> table = table(5)
```

注意看最后一句，是不是很符合装饰器的语法规则？什么情况下你会写这样的代码呢？

1. 你需要把相对复杂业务写成一个方法。
2. 这个方法和返回值可以同名，而且你不希望对外公开此方法，只公开结果。
3. 你想尽量使用装饰器。（无厘头的理由）

那么这时候`@call()`装饰器就登场了。

```python
def call(*args, **kwargs):
    def call_fn(fn):
        return fn(*args, **kwargs)
    return call_fn
```

这个装饰器会把你传入的参数送给目标函数然后直接执行。

```python
@call(5)
def table(n):
    value = []
    for i in range(n):
        value.append(i*i)
    return value

print len(table), table[3]  # 5 9
```

`@call()`装饰器适用于任何函数，你传入的参数会被直接使用然后结果赋值给同名函数。这样避免了你重新定义一个变量来存储结果。

## @list 装饰器

假如你有一个这样一个生成器函数。

```python
def table(n):
    for i in range(n):
        yield i
```

当你要生成`n=5`的序列时，可以直接调用。

```python
table = table(5)
print table  # <generator object table at 0x027DAC10>
```

使用`@call()`装饰器，也能得到一样的结果。

```python
@call(5)
def table(n):
    for i in range(n):
        yield i

print table  # <generator object table at 0x0340AC10>
```

你还可以直接将其转换成列表。（使用`list(generator_object)`函数）

```python
@list
@call(5)
def table(n):
    for i in range(n):
        yield i

print table  # [0, 1, 2, 3, 4]
```

这等同于列表表达式，但是可读性也许差了不少。例子本身只是演示了装饰器的一种用法，但不是推荐你就这样使用装饰器。你这样用也许会被其他同事拖到墙角里打死。

## 类装饰器

在 Python 2.6 以前，还不支持类装饰器。也就是说，你不能使用这样的写法。

```python
@decorator
class MyClass(object):
    pass
```

你必须这样写：

```python
class MyClass(object):
    pass

MyClass = decorator(MyClass)
```

也就是说，@语法对类是做了特殊处理的，类不一定是一个 callable 对象（尽管它有构造函数），但是也允许使用装饰器。那么基于以上语法，你觉得类装饰器能实现什么功能呢？

举一个例子，[ptest](https://pypi.python.org/pypi/ptest)中的`@TestClass()`用于声明一个测试类，其源代码大致如此。

```python
def TestClass(enabled=True, run_mode="singleline"):
    def tracer(cls):
        cls.__pd_type__ ='test'
        cls.__enabled__ = enabled
        cls.__run_mode__ = run_mode.lower()
        return cls
    return tracer
```

当我们在写一个测试类时，发生了什么？

```python
@TestClass()
class TestCases(object):
    # your test case ...

print TestCases.__dict__  # {'__module__': '__main__', '__enabled__': True, '__pd_type__': 'test', '__run_mode__': 'singleline', ...}
```

居然装饰器的参数全都变成了变成这个类的属性，好神奇！我们把语法糖一一展开。

```python
class TestCases(object):
    pass

decorator = TestClass()
print decorator  # <function tracer at 0x033128F0>

TestCases = decorator(TestCases)
print TestCases  # <class '__main__.TestCases'>

print TestCases.__dict__  # {'__module__': '__main__', '__enabled__': True, '__pd_type__': 'test', '__run_mode__': 'singleline', ...}
```

当装饰器在被使用时，`TestClass()`函数会马上被执行并返回一个装饰器函数，这个函数是一个闭包函数，保存了`enabled`和`run_mode`两个变量。另外它还接受一个类作为参数，并使用之前保存的变量为这个类添加属性，最后返回。所以经过`@TestClass()`装饰过的类都会带上`__enabled__`、`__pd_type__`以及`__run_mode__`的属性。

由此可见，类装饰器可以完成和 Java 类似的注解功能，而且要比注解强大的多。

## 后记

装饰器就是一个语法糖，当你看不懂一个装饰器时，可以考虑将其依次展开，分别带入。这个语法糖给了我们不少方便，但是也要慎用。毕竟可维护的代码才是高质量的代码。
