---
title: Improve Assertion In Python
date: 2016-10-20
tags: [python, software testing]
categories: Tech
---

Assertion is the basics of testing.

<!-- more -->

### Why not using Python Assert

Assertion in Python is pretty simple, you can assert any condition by `assert` statement.

```python
>>> assert 1 + 1 == 2
>>> assert isinstance('Hello', str)
>>> assert isinstance('Hello', int)

Traceback (most recent call last):
  File "<input>", line 1, in <module>
AssertionError
```

It is great that `assert` can stop your application/tests when something goes wrong. However, it is not good enough that the **AssertionError** does not expose more information. In above example, we received the error message only with file name and line number, you have to start debugger to discover more.

### Improved Solution #1

An improved solution is always appending message in your assertion.

```python
>>> s = "nothin is impossible."
>>> key = "nothing"
>>> assert key in s, "Key: '{}' is not in Target: '{}'".format(key, s)

Traceback (most recent call last):
  File "<input>", line 1, in <module>
AssertionError: Key: 'nothing' is not in Target: 'nothin is impossible.'
```

Well, it fixed the problem, but it not elegant. If you are a QA engineer, you have to do a lot of assertions in thousands of test cases. With above solution, I would choose to die :-|

### Improved Solution #2

You might know about test frameworks, how do they do assertion? Yes, using test framework assertion is a nice alternation.

#### py.test

If you are running tests with [py.test](https://pypi.python.org/pypi/pytest), you can keep everything unchanged in your code, the failure message will tell you what is going on in failed assertion.

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

Python [unittest](https://docs.python.org/3/library/unittest.html) module provides assertion feature in itself, it recommends `self.assertXXX()` methods, but not `assert XXX` statements.

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

I like [ptest](https://pypi.python.org/pypi/ptest) very much, its assertion feature is more readable and smart. Thanks its author Karl :-)

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

### Improved Solution #3

It is not only you and me are frustrating on python assertion, so people created packages to replace default assertion. I strongly recommend you should have a try for [assertpy](https://pypi.python.org/pypi/assertpy) package, which is high rating and powerful.

```shell
pip install assertpy
```

**Example:**

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

From its [github home page](https://github.com/ActivisionGameScience/assertpy) you will see it supports assertion in most test scenarios.

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

The assertion message is really helpful, they looks like:

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

Before I found this package I am thinking about writing common assertion package for Labs, but now, I don't think I should spend time to invent the wheel again.

### Summary

Assertion is pretty important to a system, it can increase stability and save your time in debugging.

Replacing all built-in assertion to 3rd party assertion in your code is not a good idea, because IDE like PyCharm knows nothing about that, so it will not provide auto-completion for those assertion.

So my suggestion is, using more powerful assert functions in scenarios that you really want to verify something, keeping built-in assertion where you might fall in a pitfall, and with essential message. Keep It Simple and Stupid.
