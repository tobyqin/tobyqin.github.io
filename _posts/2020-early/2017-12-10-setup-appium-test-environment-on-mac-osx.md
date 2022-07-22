---
title: 在Mac OSX 上配置 Appium + Android 自动化测试环境
categories: [Tech]
tags: [python,appium,mac,automation]
date: 2017-12-10
---

Appium是手机端自动化测试的必备知识。

<!-- more -->

## 前提准备

开始正文之前，你需要准备好一些基本条件：

1. 安装好Mac OSX 操作系统的设备
2. 能够访问中国局域网以外资源的方法（没有也行，但很痛苦）
3. 已经安装好 [homebrew](https://brew.sh/)
4. 已经安装好 Python3.x，可以通过brew安装
5. 已经安装好 Java Runtime Environment，可以通过brew安装



## 安装 Android Studio / SDK

本文主要为了测试安卓应用，那么这里我们就需要安装Android Studio或者Android SDK。

- Android Studio - 完整的开发以及测试工具，需要梯子
- Android SDK + Platform Tools - 足够完成自动化测试，通过homebrew安装

如果只是为了自动化测试建议安装SDK足矣，使用brew命令轻松搞定。

```
brew cask install android-sdk
brew cask install android-platform-tools
```

在国内使用brew可以事先配置好国内源，速度会快很多。

如果有梯子可以选择Android Studio，安装方法很简单，官网下载后一路Next，启动后会让你把SDK等等一系列依赖都装好，一步到位。

## 准备Android 模拟器或者使用真机

如果SDK已经安装完毕，应该可以使用adb命令来检查可用的模拟器或者已经连接到电脑上的实体手机。

```
adb devices
```

如果不知道adb工具在哪，可以先在命令行里通过find搜索。

```
find /Users -name adb  # 搜索用户目录
find / -name adb  # 搜索全盘
```

找到之后可以把 adb 目录加入PATH，方便后续使用。一般在 `~/Library/Android/sdk/platform-tools/adb`，如果通过brew安装，会自动建立软链接后加入PATH。

模拟器可以使用官方模拟器（Android Studio自带），或者口碑较好的 [Genymotion](https://www.genymotion.com/)，具体请查阅其他文章，搭模拟器其实也挺不容易的。

## 安装Appium

Appium 可以通过多种方式安装。

### 方式一：使用 NodeJS 安装

首先通过brew安装nodejs：

```Shell
brew install node
```

配置国内源，在个人目录下新建一个.npmrc文件，写入：

```Shell
registry=https://registry.npm.taobao.org/
```

开始安装Appium：

```Shell
npm install -g appium
npm install -g appium-doctor
```

Appium-doctor 可以帮你诊断测试环境，建议安装。

### 方式二：下载Appium桌面版安装

官方的Appium桌面安装包可以从github下载，需要梯子。

- <https://github.com/appium/appium-desktop/releases>

下载到的dmg文件双击装载，把Appium拖到Application里完成安装。

## 安装 Appium-Client

本文只讨论Python实现的Appium测试，所以你只需要允许以下命令：

```Shell
pip install Appium-Python-Client
```

如果需要使用其他编程语言，下表供参考：

| 语言/框架                | Github版本库以及安装指南                          |
| -------------------- | ---------------------------------------- |
| Ruby                 | <https://github.com/appium/ruby_lib>     |
| Python               | <https://github.com/appium/python-client> |
| Java                 | <https://github.com/appium/java-client>  |
| JavaScript (Node.js) | <https://github.com/admc/wd>             |
| Objective C          | <https://github.com/appium/selenium-objective-c> |
| PHP                  | <https://github.com/appium/php-client>   |
| C# (.NET)            | <https://github.com/appium/appium-dotnet-driver> |
| RobotFramework       | <https://github.com/jollychang/robotframework-appiumlibrary> |

## 必要的环境变量设置

如果你已经安装了appium-doctor，那么你只要运行appium-doctor命令就可以知道你还需要设置哪些环境变量，比如：

```Shell
tobyqin@CatBook ~> appium-doctor
info AppiumDoctor Appium Doctor v.1.4.3
info AppiumDoctor ### Diagnostic starting ###
info AppiumDoctor  ✔ The Node.js binary was found at: /usr/local/bin/node
info AppiumDoctor  ✔ Node version is 7.10.0
info AppiumDoctor  ✔ Xcode is installed at: /Applications/Xcode.app/Contents/Developer
WARN AppiumDoctor  ✖ Xcode Command Line Tools are NOT installed!
info AppiumDoctor  ✔ DevToolsSecurity is enabled.
info AppiumDoctor  ✔ The Authorization DB is set up properly.
WARN AppiumDoctor  ✖ Carthage was NOT found!
info AppiumDoctor  ✔ HOME is set to: /Users/tobyqin
info AppiumDoctor  ✔ ANDROID_HOME is set to: /Users/tobyqin/Library/Android/sdk/
info AppiumDoctor  ✔ JAVA_HOME is set to: /Library/Java/JavaVirtualMachines/jdk1.8.0_112.jdk/Contents/Home
info AppiumDoctor  ✔ adb exists at: /Users/tobyqin/Library/Android/sdk/platform-tools/adb
info AppiumDoctor  ✔ android exists at: /Users/tobyqin/Library/Android/sdk/tools/android
info AppiumDoctor  ✔ emulator exists at: /Users/tobyqin/Library/Android/sdk/tools/emulator
info AppiumDoctor  ✔ Bin directory of $JAVA_HOME is set
info AppiumDoctor ### Diagnostic completed, 2 fixes needed. ###
info AppiumDoctor
info AppiumDoctor ### Manual Fixes Needed ###
info AppiumDoctor The configuration cannot be automatically fixed, please do the following first:
WARN AppiumDoctor - Please install Carthage. Visit https://github.com/Carthage/Carthage#installing-carthage for more information.
info AppiumDoctor ###
info AppiumDoctor
info AppiumDoctor Bye! Run appium-doctor again when all manual fixes have been applied!
```

其实你不一定需要把通过所有检查项，如果只是为了完成Android的Appium测试，只要确保ANDROID_HOME 和 JAVA_HOME 正确配置，另外SDK Tools 和Platform Tools都加入PATH就基本完成了。可以参考我个人目录下的.bash_profile设置：

```Shell
export ANDROID_HOME=~/Library/Android/sdk/
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_112.jdk/Contents/Home

export PATH=~/bin:$PATH:/usr/local/bin:$ANDROID_HOME/platform-tools/:$JAVA_HOME/bin
```

## 开始编写自动化测试

终于到了开始写代码的时候了，相信你已经迫不及待了，别急，你还要准备以下条件：

1. 模拟器或者测试机必须是Ready的状态，每次启动模拟器都很费时间，所以建议模拟器不要关闭，通过代码来启动模拟器是一个办法，但是时间成本有点高。
2. 如果是调试代码阶段，建议保持Appium桌面版长期运行，但是Appium和uiautomator有冲突，只能二选一。
3. Appium会根据你的测试代码去寻找符合要求的设备，如果你启动了多台虚拟机或者连接了多台实体机，请显式地在代码中指定设备名称或者版本号。

自动化测试代码例子如下，启动内置拨号软件，搜索关键字。

```Python
from appium import webdriver

desired_caps = {}
desired_caps['platformName'] = 'Android'
desired_caps['platformVersion'] = '7.1.1'
desired_caps['deviceName'] = 'Android Emulator'
desired_caps['appPackage'] = 'com.android.dialer'
desired_caps['appActivity'] = 'DialtactsActivity'

driver = webdriver.Remote('http://localhost:4723/wd/hub', desired_caps)
driver.find_element_by_id('com.android.dialer:id/search_box_collapsed').click()
search_box = driver.find_element_by_id('com.android.dialer:id/search_view')
search_box.click()
search_box.send_keys('hello toby')
```

恭喜你，解锁了移动应用测试的新成就！

## 参考

- [在 Windows 下搭建 Appium + Android 自动化测试环境](/posts/2017-05-03/setup-appium-automation-test-environment/)