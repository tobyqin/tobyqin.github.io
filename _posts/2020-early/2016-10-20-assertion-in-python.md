---
title: Python中不尽如人意的断言Assertion
date: 2016-10-20
tags: [python, software testing]
categories: Tech
---

断言是测试的氧气，无断言，不测试。

<!-- more -->

### Python Assert 为何不尽如人意

Python 中的断言用起来非常简单，你可以在`assert`后面跟上任意判断条件，如果断言失败则会抛出异常。

```python
>>> assert 1 + 1 == 2
>>> assert isinstance('Hello', str)
>>> assert isinstance('Hello', int)

Traceback (most recent call last):
  File "<input>", line 1, in <module>
AssertionError
```

其实`assert`看上去不错，然而用起来并不爽。就比如有人告诉你程序错了，但是不告诉哪里错了。很多时候这样的`assert`还不如不写，写了我就想骂娘。直接抛一个异常来得更痛快一些。

### 改进方案 #1

一个稍微改进一丢丢的方案就是把必要的信息也放到`assert`语句后面，比如这样。

```python
>>> s = "nothin is impossible."
>>> key = "nothing"
>>> assert key in s, "Key: '{}' is not in Target: '{}'".format(key, s)

Traceback (most recent call last):
  File "<input>", line 1, in <module>
AssertionError: Key: 'nothing' is not in Target: 'nothin is impossible.'
```

看上去还行吧，但是其实写的很蛋疼。假如你是一名测试汪，有成千上万的测试案例需要做断言做验证，相信你面对以上做法，心中一定有千万只那种马奔腾而过。

### 改进方案 #2

既然你是搞测试的，相比听过不少测试框架。你猜到我要说什么了吧？对，不用测试框架里的断言机制，你是不是洒。

#### py.test

[py.test](https://pypi.python.org/pypi/pytest) 是一个轻量级的测试框架，所以它压根就没写自己的断言系统，但是它对 Python 自带的断言做了强化处理，如果断言失败，那么框架本身会尽可能多地提供断言失败的原因。那么也就意味着，用**py.test**实现测试，你一行代码都不用改。

```python
import pytest

def test_case():
    expected = "Hello"
    actual = "hello"
    assert expected == actual

if __name__ == '__main__':
    pytest.main()

"""
================================== FAILURES ===================================
__________________________________ test_case __________________________________

    def test_case():
        expected = "Hello"
        actual = "hello"
>       assert expected == actual
E       assert 'Hello' == 'hello'
E         - Hello
E         ? ^
E         + hello
E         ? ^

assertion_in_python.py:7: AssertionError
========================== 1 failed in 0.05 seconds ===========================
""""
```

#### unittest

Python 自带的[unittest](https://docs.python.org/3/library/unittest.html)单元测试框架就有了自己的断言方法 `self.assertXXX()` ，而且不推荐使用`assert XXX `语句。

```python
import unittest

class TestStringMethods(unittest.TestCase):

    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FoO')

if __name__ == '__main__':
    unittest.main()

"""
Failure
Expected :'FOO'
Actual   :'FoO'

Traceback (most recent call last):
  File "assertion_in_python.py", line 6, in test_upper
    self.assertEqual('foo'.upper(), 'FoO')
AssertionError: 'FOO' != 'FoO'
"""
```

#### ptest

我非常喜欢[ptest](https://pypi.python.org/pypi/ptest)，感谢 Karl 大神写了这么一个测试框架。ptest 中的断言可读性很好，而且智能提示也很方便你通过 IDE 轻松完成各种断言语句。

```python
from ptest.decorator import *
from ptest.assertion import *

@TestClass()
class TestCases:
    @Test()
    def test1(self):
        actual = 'foo'
        expected = 'bar'
        assert_that(expected).is_equal_to(actual)

"""
Start to run following 1 tests:
------------------------------
...
[demo.assertion_in_python.TestCases.test1@Test] Failed with following message:
...
AssertionError: Unexpectedly that the str <bar> is not equal to str <foo>.
"""
```

### 改进方案 #3

不仅仅是你和我对 Python 中的断言表示不满足，所以大家都争相发明自己的 assert 包。在这里我强烈推荐[assertpy](https://pypi.python.org/pypi/assertpy) 这个包，它异常强大而且好评如潮。

```shell
pip install assertpy
```

**看例子:**

```python
from assertpy import assert_that

def test_something():
    assert_that(1 + 2).is_equal_to(3)
    assert_that('foobar')\
        .is_length(6)\
        .starts_with('foo')\
        .ends_with('bar')
    assert_that(['a', 'b', 'c'])\
        .contains('a')\
        .does_not_contain('x')
```

从它的[github 主页](https://github.com/ActivisionGameScience/assertpy) 主页文档上你会发现它支持了几乎你能想到的所有测试场景，包括但不限于以下列表。

- Strings
- Numbers
- Lists
- Tuples
- Dicts
- Sets
- Booleans
- Dates
- Files
- Objects

而且它的断言信息简洁明了，不多不少。

```
Expected <foo> to be of length <4>, but was <3>.
Expected <foo> to be empty string, but was not.
Expected <False>, but was not.
Expected <foo> to contain only digits, but did not.
Expected <123> to contain only alphabetic chars, but did not.
Expected <foo> to contain only uppercase chars, but did not.
Expected <FOO> to contain only lowercase chars, but did not.
Expected <foo> to be equal to <bar>, but was not.
Expected <foo> to be not equal to <foo>, but was.
Expected <foo> to be case-insensitive equal to <BAR>, but was not.
```

在发现 assertpy 之前我也想写一个类似的包，尽可能通用一些。但是现在，我为毛要重新去造轮子？完全没必要！

### 总结

断言在软件系统中有非常重要的作用，写的好可以让你的系统更稳定，也可以让你有更多真正面对对象的时间，而不是在调试代码。

Python 中默认的断言语句其实还有一个作用，如果你写了一个类型相关的断言，IDE 会把这个对象当成这种类型，这时候智能提示就有如神助。

要不要把内置的断言语句换成可读性更好功能更强大的第三方断言，完全取决于实际情况。比如你真的需要验证某个东西并且很关心验证结果，那么必须不能用简单的 assert；如果你只是担心某个点可能有坑或者让 IDE 认识某个对象，用内置的 assert 既简单又方便。

总之，你自己看着办。
