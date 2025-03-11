---
title: 说说Python中的单元测试
categories: [Tech]
tags: [python, unittest, pytest]
date: 2018-04-08
---

单元测试是每种编程语言必学的课题，是保护开发者的强力护盾，每个程序员都在时间允许的情况下尽可能多的写单元测试，今天我们不讨论其必要性，只抛砖引玉聊一聊 Python 中的单元测试，本文仅代表个人看法。

<!-- more -->

## 标准库中难以忍受的 unittest

很多时候我们总是认为标准库里的带的总是精挑细选的，如果不经过仔细打磨怎么可能入选为一等公民？但我要告诉你，Python 标准库里的单元测试框架真不是最好的，随着你对 Python 的熟悉你甚至会讨厌这个 unittest。

Python 一直崇尚简单，优雅，高效地完成事情，当你写完一个函数需要测试一下时，使用标准库的 unittest 你需要做这些事情：

- 新建单元测试脚本
- 导入单元测试依赖
- **继承单元测试类**
- 实现单元测试方法

具体的实例代码如下：

```python
import unittest

class IntegerArithmeticTestCase(unittest.TestCase):
    def testAdd(self):  # test method names begin with 'test'
        self.assertEqual((1 + 2), 3)
        self.assertEqual(0 + 1, 1)

if __name__ == '__main__':
    unittest.main()
```

看上去还行，不是很难。但是渐渐地你会吐槽：

- 为啥我要新建一个文件来写测试？
- 为啥我要继承一个类来写测试？
- 为啥我要用 unittest 的 Assertion 来做断言？
- 为啥 unitunit 的命名规则跟最佳实践不一样（`mixedCase` vs `lower_case`）？

要回答以上问题，答案只有一个：[历史原因](https://www.quora.com/Will-Pythons-unittest-module-become-pythonic-anytime-soon)。

很久很久以前，Python 从 Java 借鉴了单元测试框架，包括命名规则和实现方式，一直沿用至今。不得不说这个框架没啥毛病，该有的功能的都有，想做的事都可以做，但是用起来总是没有爽的感觉。

但是为啥伟大的社区力量为啥不把这个框架改的爽一点呢？没办法，我估计是为了世界和平，你要知道 Python 这个庞然大物能健康地活着，后面有无数的类库和方法在支撑，而这些类库和方法都被单元测试保护着，如果修改了单元测试框架导致兼容性问题，就成了千古罪人。

## 见识简洁的单元测试 pytest

Python 中很多大牛其实都有严重的强迫症，追求简洁和优雅的代码。必然的，他们会抛弃标准库中的 unittest，使用或者发明自己心仪的单元测试框架。

正如其名，pytest 是一个无数人推荐并在使用的 Python 单元测试框架，它使用起来非常简单，只要你的方法名以 `test` 开头就可以，你可以和需要测试的方法放在一起，亦或是新建一个文件来专门整理单元测试，都可以。

```python
def your_func():
  pass

def test_your_func():
  assert result
```

这样的设计，就让你写单元测试成了顺手拈来的事，假如你写完了一个方法，想看看是否工作，在旁边直接写上一个`test` 开头的方法，稍微准备一下数据就可以验证这个方法好不好用，岂不妙哉？

> The idioms that pytest first introduced brought a change in the Python community because they made it possible for test suites to be written in a very compact style, or at least far more compact than was ever possible before. Pytest basically introduced the concept that Python tests should be plain Python functions instead of forcing developers to include their tests inside large test classes.

pytest 的发明让大家意识到单元测试原来可以这么轻松和随意，完全没有必要去继承一个所谓的测试类或者按照复杂的规则才能开始书写测试代码，这也是我选择和推荐它的理由。

当然，如果原来你的单元测试时 unittest 写的话，pytest 其实也是[有可能兼容的](https://docs.pytest.org/en/latest/unittest.html)的。

pytest 能够识别 `unittest.TestCase` 子类中的测试方法，如果文件名符合 `test_*.py` 或者 `*_test.py` 这样的规则。

并且大多数 `unittest` 的功能都是被支持的，例如：

- `@unittest.skip` 装饰器;
- `setUp/tearDown`;
- `setUpClass/tearDownClass()`;

我觉得，pytest 有以下优点：

- 上手和使用足够简单
- 当 case 失败时信息足够丰富和直观，比如最后导致失败的变量值会打印出来
- 更丰富的运行参数
- 可以使用 `assert` 而不是 `self.assert*`
- 被广大 IDE 支持，社区资源丰富，用户群体大

## 让单元测试和 IDE 无缝集成

毕竟我们大多数人都不是神，不能用记事本写代码，IDE 才是我们正确搬砖的方式。Python 的首选 IDE 毋庸置疑就是 JetBrain 公司出品的 [PyCharm](https://www.jetbrains.com/pycharm/download/)。

在 PyCharm 中只要你将默认的单元测试驱动改成 pytest，就可以在任意`test`开头的方法上通过右键菜单运行或者调试这个测试案例，非常方便。

![更改PyCharm设置](https://image.tobyqin.cn/pytest-pycharm-settings.png)

![右键菜单运行或者调试](https://image.tobyqin.cn/pytest-context-run.png)

如果你要运行当前文件的所有测试，只要从非`test`方法的其他区域点击右键即可。或者修改任意已经运行过的 Configuration，添加你想要的参数，比如最多运行挂 3 个 case 就终止测试等等。

![自定义Run Configuration](https://image.tobyqin.cn/pytest-configuration.png)

## 闲话和总结

单元测试的重要性大家都知道，大名鼎鼎的 TDD 应该都听过，但是真正在实践的少之又少。

究其原因，一些人会说时间写代码都不够，哪还有空写单元测试。还有一些人就是嫌麻烦，在绝大多数编程语言里单元测试都是需要单独建立工程和目录的，写单元测试需要很多基础工作要做，本以为顺手就可以写的单元测试，实际上需要费九牛二虎之力还是在搭架子，太沮丧了。

Python 的动态特性和灵活性让它有可能让单元测试超级简单，有可能你认为单元测试还是不要和业务代码混合在一起的好，那就多辛苦一点新建一个文件导入要测试的方法，写一个 `test` 开头的方法即可，不算太难，不要找推辞的理由。

最后我的个人观点，单元测试其实还有一个非常重要的作用，就是替代函数文档注释。比如你写了一个函数，使用起来可能有那么一点复杂，你可以给它写一份清晰的注释文档，但是千言万语不如给我来个例子，单元测试可以充当例子的角色，什么样的输入，输出结果如何，一目了然。

希望从今天起，你的代码也都有单元测试。
