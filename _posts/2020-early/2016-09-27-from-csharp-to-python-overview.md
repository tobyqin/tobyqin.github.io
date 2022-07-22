---
title: 从C#到Python - 语言特性和概览
tags: [C#, python]
date: 2016-09-27 09:35:13
categories: Tech
---

因为工作的原因，目前主力编程语言从 C#转移到 Python，所以在此记录这两种语言的一些异同点和自己的感悟收获。本系列文章数量不限，随想随写。

<!-- more -->

## 语言特性和特点

### `C#`

C#是微软公司主推的编程语言，在 Windows 平台的首选开发语言，需要.net framework 的支持，非微软平台支持目前并不完善。主要特点是语法简单，IDE 强悍(VS 是我用过最强悍和人性化的 IDE，没有之一)，C#是强类型高级编程语言。

### `Python`

是开源的动态解释型语言，由 Guido van Rossum 于 1989 年发明。它天生具有跨平台的能力，默认集成在 MacOS 和 Linux 系统中。Windows 平台需要单独安装。主要特点是语法简洁，第三方类库丰富强大，数据处理能力异常优秀。

### 漫谈瞎扯

我使用 C#编程的时间大概有 5 年左右，对于它的各种特性还算比较了解。接触 Python 大概只有三个多月，不过三观已经被刷新。限于我个人水平，本文对 C#和 Python 特别深入的东西不会特别介绍，仅从我个人的角度来帮助 C#的程序猿认识 Python。

很明显我感觉 C#是**简单**，但 Python 是**简洁**。两个不完全一样的概念，简洁之中蕴含了简单，但是简洁也意味着信息的省略和丢失。举一些具体的例子

#### 标识语句块

C#用花括号，和大多数编程语言一样。

```csharp
using System;

namespace csharp_example
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            Console.WriteLine("Hello C#!");
        }
    }
}
```

而 Python 用的是缩进。

```python
def say_hello(name=None):
    if name is None:
        name = 'python!'

    print "hello", name

if __name__ == '__main__':
    say_hello()

```

#### 命名规则

C#对文件系统的命名空间是 System.IO。

```csharp
using System.IO;
```

Python 只有 io，只有两个字母，还是小写的！

```python
import os
```

#### 迭代语句

C#中 for 迭代是这样的，已经很简洁了。

```csharp
var list = new List<int>() { 1,, 3, 4, 5, 5, 5, 6 };
foreach (var item in list)
{
    Console.WriteLine(item);
}
```

Python 是这样的，真的不能再简洁了。

```python
list = [1, 2, 3, 4, 4, 5, 5, 6]
for i in list:
    print i
```

#### 其他

还有不少细节的地方在你接触 Python 之后也一定深有体会。

- 比如为了不切换大小写，Python 推荐使用全小写的命名规范（类命名除外）
- Python 要求省略句尾的分号
- Python 不推荐在逻辑判断后使用括号，比如 `if i > 0:` 而不是 `if (i > 0):`

这样的例子枚不胜举，如果语意可读性上来说，我比较喜欢 C#的做法，因为使用驼峰命名规则，基本上一个语句就是一小段英文。而且从命名规范上来说，C#推荐使用完整的英文单词来命名变量和类名。

Python 就不见得了，很多类库和命名都是极度简洁的，比如`pytz`, `wrapt`, `isalnum()`。最令人发指的是居然连中间的下划线也省了，比如`altsep`, `execl`, `getcwdu`, `spawnle`，尼玛，这些是什么鬼，这一点也不考虑其他人的感受，很多时候你只能 yy 或者查文档，这就是简洁的代价。

不过话说回来，正因为 Python 这也省布料，所以使用 Python 实现与 C#，JAVA 相同功能，至少可以少 20%的代码量。夸张的说法甚至 60%到 80%，我保留意见，但不得不承认是极有可能的。

最后从动态语言和非动态语言的角度简单说一下，动态语言的特点就是程序在运行时才能确定类型和行为，动态语言也叫鸭子类型`ducking typing`，源自于来自 James Whitcomb Riley 这句有名的话。

> If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.
>
> 如果它看起来像一只鸭子，游起来也像鸭子，而且还会像鸭子一样叫，那么它极有可能就是一只鸭子。

在动态语言里变量只是一个标记，具体的行为可以在程序运行时才确定，比如两个变量相加，Python 可以这样写：

```python
def add(a, b):
    return a + b
```

你不需要**也不能**指定这个 a, b 的类型，当程序在运行时，他们的相加行为会根据传入的具体类型确定。

```python
>>> add(1, 2)
3

>>> add([1,2,3], [4,5,6])
[1, 2, 3, 4, 5, 6]

>>> add('hi ', 'toby')
'hi toby'

>>> add('hi', 1)
Traceback (most recent call last):
  File "<pyshell#7>", line 1, in <module>
    add('hi', 1)
  File "<pyshell#2>", line 2, in add
    return a+b
TypeError: cannot concatenate 'str' and 'int' objects
>>>
```

反观 C#，Visual Studio 会对你的语法进行检查，没有泛型之前，你只能这样写。

```csharp
public int AddInt(int a, int b)
{
  return a + b;
}

public string AddString(string a, string b)
{
  return a + b;
}
```

有泛型和 dynamic 类型之后，情况好一些。

```csharp
public T Add<T>(T a, T b)
{
    dynamic x = a;
    dynamic y = b;
    return x + y;
}
```

然而这种和 C#本身格格不入的编码方式并不流行，而且 IDE 支持也不好。或许你能从这个小例子明白动态语言的厉害之处。

### 小结

写了那么多，希望你对 Python 有一个比较直观的印象。

两种语言各有特点，不能说谁好谁坏。具体用哪个一般只有一个原因，工作环境和项目需求。但就学习而言，如果已经熟悉 C#，转而学习 Python 还是比较简单和容易接受的，因为做的的减法。但是如果之前是 Python 而后转到 C#，就不是那么好受了，因为做的是加法。

Python 的作者有处女情节，所以处处都要追求优雅，简单，完美。想适应这种情节真的需要刷三观，费半条命。

而且写 Python 的人一般都短命，因为喜欢 Python 的人都喜欢这句话。

`Life is short, I use Python. （人生苦短，我用Python）`

> 本文源码地址：https://github.com/tobyqin/csharp_vs_python
