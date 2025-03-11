---
title: Python 基础简明教程
categories: [Tech]
tags: [Python]
date: 2021-08-07
---

这是 Python 程序设计的简明教程，假设你已经有其他高级编程语言的经验。

## 环境准备

环境准备过程中，核心要点如下：

1. 官方下载地址 http://python.org/download/，推荐使用最新版
2. 安装路径，推荐选择用户目录（默认选项）
3. 环境配置，推荐将 Python 加入 `PATH`
4. 开发工具，推荐使用 [PyCharm 社区版](https://www.jetbrains.com/pycharm/)

## 新建项目

在 Python 中新建项目的要点：

1. 选择项目保存地址，按个人习惯选择，例如 `~/workspace`
2. 选择 Python 解释器（interpreter），可以用已经存在的解释器，也可以选择一个虚拟环境。

![新建项目](https://tobyqin.cn/docs/python/images/image-20210807100110615.png)

Python 是解释型语言，解释器可以类比成 Java 的 JDK 版本。我个人非常不推荐直接使用默认的 Python 解释器，因为随着项目的开发我们会引入很多依赖包，每个项目对同一个依赖包的版本可能有所不同，这时候使用同一个解释器会造成依赖混乱，后期排查的难度非常大，那时候再去把项目分离属于没必要的成本。

如果只是学习或者做非专业项目开发，可以使用默认环境，同时请关注环境污染问题。

## 第一个程序

一行代码就可以开启 Python 世界的大门。

```python
print('hello world')
```

> 友情提示：Python 中的变量和方法名主要以小写和下划线声明，请在你的代码里遵循这样的规范。

用面向对象和动态语言来写第一个程序。

```python
class Employee(object):
    pass

if __name__ == '__main__':
    employee = Employee()
    employee.code = '007'
    employee.name = 'Toby'
```

`Employee` 对象可以不声明 `code` 和 `name` 的属性，在程序运行中再给它添加 `code` 和 `name`。

`if __name__ == '__main__’` 的意思是这是 Python 程序的入口，这行代码能让很多新手纠结半天。这其实还是简单的 `if` 判断，关键是 `__name__` 和 `__main__` 是什么鬼。

`__name__` 是 Python 模块的名字，双下划线属于 Python 进阶课程的内容，双下划线开头和结尾的变量一般属于 Python 的内置变量，`__name__` 就是其中之一，指的是当前模块的名字。但是，如果当前模块是被直接运行的模块的话，值就等于 `__main__`，否则就等于模块原本的名字。

> 想了解更多双下划线的内容，搜索 [Python 的魔法方法](https://tobyqin.cn/posts/2016-10-12/underscore-in-python/)

我们做一个简单的例子，假设有 a 和 b 两个模块。

```python
# a.py
def what_is_name():
    print('a.__name__ = ' + __name__)

if __name__ == '__main__':
    what_is_name()
```

现在直接运行脚本 `a.py`。

```bash
$ python a.py
a.__name__ = __main__
```

这时候的`__name__`返回的值就是`__main__`。我们来写一个 b 模块，调用 a 模块，然后看看这时候 a 里的`__name__`是什么。

```python
# b.py
import a

a.what_is_name()
```

现在直接运行脚本 `b.py`。

```bash
$ python b.py
a.__name__ = a
```

这个例子看懂了，这个 Python 入门的坎就过去了。哦对了，Python 是按缩进来严格划分代码块的，这个坑踩几次就记住了。

## Python 的数据类型

数据类型是编程语言里的核心概念，Python 内置的数据类型非常简单，分类如下。

| 类型       | 英文           | 关键字                       |
| ---------- | -------------- | ---------------------------- |
| 文本类型   | Text Type      | str                          |
| 数字类型   | Numeric Types  | int, float, complex          |
| 序列类型   | Sequence Types | list, tuple, range           |
| 字典类型   | Mapping Type   | dict                         |
| 集合类型   | Set Types      | set, frozenset               |
| 布尔类型   | Boolean Type   | bool                         |
| 二进制类型 | Binary Types   | bytes, bytearray, memoryview |
| 空类型     | None Type      | None                         |

实际工作中用的最多的类型主要是文本，数字，列表，字典，布尔。使用类型时不需要声明类型，Python 在运行时会根据实际值进行运算，这是动态语言的优势，但是在如此高的灵活性下对程序员的素质要求也很高。

```python
a = 'hello'               # text type
b = 1                     # numeric type

c = [100,200,'test']      # list type
print(c[2])               # use
c[1] = 99                 # assign

d = {'a': 1, 'b': c}      # dict type
print(d['a'])             # use
d['b'] = 'test'           # assign

e = (1,3,4)               # set type
print(e[1])               # use it, cannot assign

f = True                  # bool type
g = b'hello'              # byte type
h = None                  # null type
```

要判断 Python 变量的类型是什么的话，可以用两种方法，第一个是`type()`。

```python
>>> type(1)
# <class 'int'>
>>> type('hello')
# <class 'str'>
```

第二种方法是 `isinstance()`。

```python
>>> isinstance(1, int)
# True
>>> isinstance(1, str)
# False
```

## 控制流和语法

Python 是最接近自然语言的编程语言，所以 Python 的控制流非常容易上手。下面列举一下我们常用的控制流和语法。

```python
# if / else
if b > a:
  print("b > a")
else:
  print("b < a")

# while
while i < 6:
  print(i)
  i += 1

# for
fruits = ["apple", "banana", "cherry"]
for x in fruits:
  print(x)

# function
def my_function():
  print("Hello from a function")

# lambda
x = lambda a : a + 10
print(x(5))

# try / catch
try:
  print(x)
except:
  print("An exception occurred")
```

## Python 的类和构造函数

有了一点双下划线的基础后，我们就可以了解一下 Python 类的构造函数，它也是一个双下划线，名字叫 `__init__`。看一个简单的例子。

```python
class Employee(object):
    def __init__(self, code, name):
        self.code = code
        self.name = name

if __name__ == '__main__':
    employee = Employee('007', 'Toby')
    print(employee.code)
    print(employee.name)
```

`def` 是 Python 里声明方法的关键字，上面的例子我们重载 `Employee` 的构造函数，构造函数里加入了两个参数 `code` 和 `name`。你是不是觉得我在骗你，明明是三个参数，还有个 `self` 为啥不把它当参数。

我没有骗你，你看在调用的时候你只给了两个参数：`Employee('007', 'Toby')`

好吧，这是 Python 的第二关，又可以让新手挠半天头。`self` 可以理解成 Java 或者 C# 里的 `this`，从英文上看就是一个意思。我们把 `self` 可以理解成当前类的实例（instance），Python 奇怪的是所有的实例方法都要传入当前的实例（作为第一个参数）。构造函数也是一个实例方法。

假如不传入当前实例会怎么样？那么这个方法就是类的静态方法，比如这样。

```python
class Employee(object):

    @staticmethod
    def create(code, name):
        e = Employee()
        e.code = code
        e.name = name

# 调用静态方法
Employee.create('007', 'Toby')
```

其实这里要想深入需要更多篇幅，Python 类的**实例方法里的第一个参数一定是当前对象**，但不一定叫 `self`，可以叫阿猫阿狗，只是约定叫 `self`。想了解这个设计背后的初衷可以看这篇文档。

- https://medium.com/quick-code/understanding-self-in-python-a3704319e5f0

## 可变参数

现在来到 Python 世界的第三关：可变参数。可变参数有两种，一种是不需要名字的可变参数，比如我们有个很厉害的函数可以把所有的参数加起来。

```python
def add(...):
    # sum up all arguments
```

这个怎么写？在别的编程语言几乎不可能实现这样的功能，但是 Python 可以。

```python
def add(*args):
    return sum(args)

print(add(1, 2, 3, 4)) # => 10
```

这里的 `*args` 拿到的是一个数组，所有的参数都是这个数组里的元素，只有索引，没有名字。

可变参数的第二种情况是我们希望每个参数都有一个名字，这样在使用的时候会更方便。比如这个例子。

```python
class Employee(object):

    def __init__(self, **kwargs):
        print(kwargs['code'])
        print(kwargs['name'])

if __name__ == '__main__':
    employee = Employee(code='007', name='Toby', mail='toby@test.com')
```

上面的代码里我们可以通过参数名字拿到参数的值，在参数比较多的情况下会尤其有用。这个例子里我还传入了一个叫 `mail` 的参数，实际上没有使用但程序并不会报错。

那么`*args` 和 `**kwargs` 到底是什么对象呢？很简单：

- `*args` 是一个`Tuple`，元组，和数组有一点细微的差别，就是元组不可变，数组可变。
- `**kwargs` 是一个 `dict`，字典，和`json` 字符串差不多，就是一系列的 `key=value`。

当这两个可变参数放在一个函数里时，这个函数就成了一个超级函数，可以接受任意参数。

```python
class Employee(object):

    def __init__(self, *args, **kwargs):
        print(f'args = {args}')
        print(f'kwargs = {kwargs}')

        # args = ('007', 'Toby')
        # kwargs = {'phone': '123-456-789', 'mail': 'toby@mail.com'}

if __name__ == '__main__':
    employee = Employee('007', 'Toby', phone='123-456-789', mail='toby@mail.com')
```

在很多比较抽象的模块里，你可以能会很容易看到这样的超级函数。

### 必要关键字参数

顺带提一个实用小技巧，如果我们需要某些参数在调用的时候一定要传入参数名字，可以这样写。

```python
def calc(a,b,*,operator):
    ...

calc(1,2,operator='+') # OK
calc(1,2,'+') #TypeError: calc() takes 2 positional arguments but 3 were given
```

为什么说这个技巧实用吗？想象一下假如你遇到这样的代码就知道这个用法的好处了。

```python
magic('001','xxx','bbb','wtf',[1,2,3]) # 鬼知道我的参数是什么意思
```

## 使用 pip 安装依赖包

在 Python 的世界里有很多轮子，我们应该尽量避免造轮子，多去找找已经存在的轮子。pip 就是安装轮子的方法。比如我们要做网络请求相关的功能开发，可以这样安装 `requests` 依赖库。

```
pip install requests
```

如果提示 `pip` 命令找不到，你需要将 `pip` 加入 PATH，或者通过 Python 来调用 `pip`。

```
python -m pip install requests
```

现在就可以直接访问一个网址了。

```python
import requests

response = requests.get('https://api.ipify.org?format=json')
print(response.json()) # {"ip":"98.207.254.136"}
```

requests 库可以用最少和最优雅的代码来完成网络请求相关的所有操作，强烈安利一下。

```python
requests.get(url)
requests.post(url, data={'a': 1})
requests.put(url, json={'b': 2})
requests.delete(url)
```

## 操作数据库

为了完成后面的新手任务，我们安装一个连接 mysql 的依赖包。

```
pip install mysql-connector-python
```

假设你本机有个 mysql 的服务，我们现在需要一个数据库和一张数据表。在命令行工具里完成以下库和表的建立。

> 没有的话，可以安装 `brew install mysql` 启动：`mysql.server start`

```bash
$ mysql -uroot # 默认密码为空
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

# 创建和使用数据库
mysql> create database demo;
Query OK, 1 row affected (0.00 sec)

mysql> use demo;
Database changed

# 创建数据表
mysql> create table table Employee
    -> (id int primary key auto_increment,
    -> code varchar(100),
    -> name varchar(200)
    -> );
Query OK, 0 rows affected (0.00 sec)
```

### 连接数据库

数据库的配置文件一般要和代码分离，我们先写一个数据的配置文件。

```ini
# config.ini
[DB]
username=root
password=
database=demo
```

然后我们再来一个数据操作类，用来保存我们的 Employee 数据。

```python
# database.py
import configparser
import mysql.connector

class Database(object):

    def connect(self):
        parser = configparser.ConfigParser()
        parser.read('config.ini')
        db_config = dict(parser.items('DB'))
        self.connection = mysql.connector.connect(**db_config)
        self.cursor = self.connection.cursor()

    def save_employee(self, employee):
        self.connect()

        sql = "INSERT INTO employee (code, name) VALUES (%s, %s)"
        val = (employee.code, employee.name)
        self.cursor.execute(sql, val)
        self.connection.commit()

        self.close()

    def close(self):
        self.cursor.close()
        self.connection.close()
```

现在回到 Employee 模块，像这样。

```python
# employee.py
class Employee(object):

    def __init__(self, code, name):
        self.code = code
        self.name = name
```

我们来加一个 main.py 的主模块。

```python
# main.py
from model.database import Database
from model.employee import Employee

if __name__ == '__main__':
    emp = Employee('007', 'Toby')
    db = Database()
    db.save_employee(emp)
```

现在的目录结构看起来是这样的。

```
├── config.ini
├── main.py
├── model
   ├── __init__.py
   ├── database.py
   └── employee.py
```

现在来运行一下 main.py。

```
python main.py
```

去查一下数据库，数据应该已经存进去了。

```bash
mysql> select * from employee;
+----+------+------+
| id | code | name |
+----+------+------+
|  1 | 001  | Toby |
+----+------+------+
1 rows in set (0.00 sec)
```

查询数据库的方法可以这样写。

```python
# database.py
def query_employee(self):
    sql = 'select * from employee'
    self.connect()
    self.cursor.execute(sql)
    result = self.cursor.fetchall()
    self.close()

    return result
```

操作数据库记得要关闭数据库，这是初级教程，其实有更优雅的办法来处理数据的打开关闭，比如使用[Python 的装饰器](https://tobyqin.cn/posts/2016-10-27/python-decorator/)，有兴趣可以了解一下。

## 完结

OK，感谢你的阅读。你已经入门 Python 了，我猜的。
