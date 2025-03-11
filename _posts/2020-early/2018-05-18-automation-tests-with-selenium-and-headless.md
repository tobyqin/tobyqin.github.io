---
title: 使用浏览器的HEADLESS模式进行自动化测试
categories: [Tech]
tags: [python,selenium,headless]
date: 2018-05-18
---

自动化测试的进阶内容。

<!-- more -->

## 了解HEADLESS模式

HEADLESS BROWSER 指的是不需要用户界面的浏览器，这种浏览器在自动化测试和爬虫领域有着广泛的应用。

例如你想在网页上运行一些测试，从网页抓取信息，检查浏览器访问某些资源的状态，定时截取网页等等，你需要的是浏览器处理网页但不一定需要浏览器界面，这些情况都是HEADLESS BROWSER的应用场景。

Chrome 从 59.0 开始支持HEADLESS模式（2017年5月），Firefox从 55.0 开始也支持了HEADLESS模式（2017年9月）。也就是在今年2018年的4月份，老牌的无头浏览器 [PhantomJS](http://phantomjs.org/) 的核心开发者宣布不再维护该项目，因为Chrome 和Firefox的HEADLESS模式已经足够好并可以替代PhantomJS。

## 实践 Selenium + HEADLESS

使用浏览器的HEADLESS模式进行自动化测试，你需要先满足以下前提：

- Python + Selenium 运行环境
- Chrome 59+ 或者 Firefox 55+
- ChromeDriver 或者 GeckoDriver 最新版已加入PATH

万事俱备，废话不多说我们直接上演示代码。

### Chrome版实例

```python
from time import sleep
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

options = Options()
options.add_argument('--headless')
options.add_argument('--disable-gpu') # 允许在无GPU的环境下运行，可选
options.add_argument('--window-size=1920x1080') # 建议设置

browser = webdriver.Chrome(chrome_options=options)
browser.get('https://www.baidu.com')
browser.find_element_by_id('kw').send_keys('HELLO')
browser.find_element_by_id('su').click()

sleep(1) # 简单粗暴的等待，实际项目中勿用
assert browser.title == u'HELLO_百度搜索'
browser.save_screenshot('chrome-headless-test.png')
```

### Firefox版实例

```
from time import sleep
from selenium import webdriver
from selenium.webdriver.firefox.options import Options

options = Options()
options.add_argument('--headless')
# options.add_argument('--window-size=1920x1080') # Firefox无效

browser = webdriver.Firefox(firefox_options=options)
browser.set_window_size(1280, 1024) # 启动后设置浏览器大小，但是高度会随着访问的网页变化

browser.get('https://www.baidu.com')
browser.find_element_by_id('kw').send_keys('HELLO')
browser.find_element_by_id('su').click()

sleep(1)
assert browser.title == u'HELLO_百度搜索'
browser.save_screenshot('firefox-headless-test.png')
```



## 总结

浏览器HEADLESS模式可以让程序运行的环境更贴近用户访问的真实环境，相对于模拟UserAgent等方式得出的数据也会更加准确可靠。

尤其在自动化测试领域，HEADLESS也有取代传统的带界面的自动化测试的趋势，有一些公司已经[将实践投入生产](https://about.gitlab.com/2017/12/19/moving-to-headless-chrome/)中。我们可以在调试自动化测试时使用用户界面，当部署到持续集成环境中是启用HEADLESS，并开启多线程使用并行的方式来运行测试案例，这样效率会大大提高，而且因为界面被干扰而导致测试失败的概率也会降低。

总的来说，至少在端对端的自动化测试中，HEADLESS模式没有明显的缺点，甚至可以成为网页自动化测试进化的下一个目标。

## 参考文档

- <https://developer.mozilla.org/en-US/Firefox/Headless_mode>
- <https://intoli.com/blog/running-selenium-with-headless-firefox/>
- <https://developers.google.com/web/updates/2017/04/headless-chrome>
- <https://about.gitlab.com/2017/12/19/moving-to-headless-chrome/>

