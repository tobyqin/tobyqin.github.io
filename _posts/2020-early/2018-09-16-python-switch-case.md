---
title: 用 Python 实现简单的 switch/case 语句
categories: [Tech]
tags: [Python, Quiz]
date: 2018-09-16
---

在Python中是没有Switch / Case语句的，很多人认为这种语句不够优雅灵活，在Python中用字典来处理多条件匹配问题字典会更简单高效，对于有一定经验的Python玩家不得不承认，的确如此。

<!-- more -->

但今天我们还是来看看如果一定要用Python来Switch / Case，可以怎么玩。

## 语法约束

我们先定义一下Switch/Case应该怎么表达，为了简单我们可以让它长成这样。

```python
def cn():
    print('cn')

def us():
    print('us')

switch(lang).case('cn',cn)
			.case('us',us)
   			.default(us)
```

## 类实现一

通过以上约束，我们可以把switch当成一个类来实现，传入的参数在构造函数里处理，然后再分别实现case和default方法即可。

```python
class switch(object):
    def __init__(self, case_path):
        self.switch_to = case_path
        self._invoked = False

    def case(self, key, method):
        if self.switch_to == key and not self._invoked:
            self._invoked = True
            method()

        return self

    def default(self, method):
        if not self._invoked:
            self._invoked = True
            method()
```

在构造函数中我们记住了`case_path` 和执行状态`_invoked`，在`case()`里如果当前的`key`和`switch_to`匹配并且函数没有被执行过，那么就更新`_invoked`并执行对应的方法。在`default()`里检查一下`_invoked`，如果从没执行过，那么就调用`default`分支的函数。

看上去还不错，我们来试用一下。

```python
switch('cn').case('cn',cn).case('us',us).default(fail)
>>> cn
switch('us').case('cn',cn).case('us',us).default(fail)
>>> cn
switch('jp').case('cn',cn).case('us',us).default(fail)
>>> fail
switch('cn').case('cn',cn).case('us',us)
>>> cn
```

让我们来看几个奇葩一点的case。

```python
# duplicate case
switch('us').case('us',cn).case('us',us).default(fail)
>>> cn

def cn() return 'cn'
def us() return 'us'

# return value
result = switch('cn').case('cn',cn).case('us',us)
result
>>> <python_switch_case.switch object at 0x11034fb70>
```

发现了没有，上面的实现不会处理重复的case，当然你可以加强一下case方法，最好是抛出异常，其他编程语言通常都这样做。

第二个问题，你希望从case里拿到返回值，像上面的写法是没希望了，因为扔掉了。我们可以考虑在switch类里加一个result的变量来保存执行结果。

```python
class switch(object):
    def __init__(self, case_path):
        ...
        self.result = None

    def case(self, key, method):
        ...
        self.result = method()
    ...
```

在调用结束后，就可以通过`result`拿到结果了。

```python
_ = switch('cn').case('cn',cn).case('us',us)
_.result
>>> cn
```

## 类实现二

我大概在网上搜了一下，你还可以参考[Brian Beck](http://code.activestate.com/recipes/410692/)通过类来实现Swich/Case。

```python
class switch(object):
    def __init__(self, value):
        self.value = value
        self.fall = False

    def __iter__(self):
        """Return the match method once, then stop"""
        yield self.match
        raise StopIteration

    def match(self, *args):
        """Indicate whether or not to enter a case suite"""
        if self.fall or not args:
            return True
        elif self.value in args:
            self.fall = True
            return True
        else:
            return False


c = 'z'
for case in switch(c):
    if case('a'): pass  # only necessary if the rest of the suite is empty
    if case('c'): pass
    # ...
    if case('y'): pass
    if case('z'):
        print("c is lowercase!")
        break
    if case('A'): pass
    # ...
    if case('Z'):
        print("c is uppercase!")
        break
    if case():  # default
        print("I dunno what c was!")
```

这种实现相对复杂一点，而且用起来也不是很舒服，又需要for又需要if（还不如直接if/else痛快）。当然也有好处，就是可以把相同结果的case放一起，而且case里可以写更多东西，不仅仅是一个方法名。

## 写在最后

最后我们还是回到Python推崇的方法来处理switch/case问题，一般我们可以通过字典来处理这种多分支的问题，举例说明。

```python
MAPPING = {
    'cn': cn,
    'us': us
}

lang = 'cn'
result = MAPPING.get(lang, default=us)
```

是不是一目了然，不仅易于阅读也易于维护。在字典中key是唯一的，value可以是任意类型的数据，可以是类或者是方法，所以足够灵活。