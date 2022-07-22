---
title: 设置 Python Selenium 中的Log显示信息
tags: [python, selenium, logging, tips]
date: 2016-09-24 22:14:44
categories: Tech
---

Python Selenium 默认会往控制台和 Log 文件里写入大量的 DEBUG 信息，比如下面这张图。

<!-- more -->

这样的相信在测试过程中有一定帮助，但大部分情况下都是没有营养的，而且会把你自己打印的 Log 信息淹没在汪洋大海中。

![image](https://raw.githubusercontent.com/tobyqin/img/master/selenium-debug-logging.png)

如果想要停止显示或者关闭 Selenium 中的 Log，你可以通过以下代码更改其默认 LOGGER 的级别。

```python
import logging
from selenium.webdriver.remote.remote_connection import LOGGER

LOGGER.setLevel(logging.WARNING)
```

**注意：**以上代码一定要在初始化`WebDriver`前进行。
