---
title: Windows UI自动化测试的XPATH实现 - WPATH
categories: [Tech]
tags: [Windows UIAutomation,XPATH,WPATH]
date: 2017-07-23
---

从事Windows 桌面应用自动化测试也有一些年了，现在谈这个话题并不流行。

<!-- more -->

因为除了企业级应用，很少有公司会只选择Windows桌面作为目标用户平台，一般都会考虑跨平台的浏览器解决方案，桌面应用的地位渐渐下降，这是事实。

当年初入测试行业时就被外包公司看上了，在微软的圈子里一待就是4年，时间真快。不得不说，一个大学刚毕业的毛头小子看到微软里各种技术和工具真像极了刘姥姥进大观园，那时候还没有iPhone，也没有Android，微软一统天下。

本文主要介绍一下我对Windows UI自动化的一些看法以及WPATH的实现和应用，如果你还在从事Windows桌面应用的自动化测试，应该能有一些帮助。

## 为何发明WPATH

Windows UI 自动化，顾名思义就是在Windows平台实现软件的界面自动化，比如自动打开Excel填入一些数据，输入公式，获取结果。正经的用途就是软件自动化测试，避免重复的手工操作；不正经的用途就是写外挂，各种投机取巧的工具等等。

最简单粗暴的实现方案就是录制回放，优点很明显，简单快速；缺点也一样明显，不可靠因素太多。主要的代表就是QTP，来自HP公司，这应该是很多同学都听过的一款测试工具。

进阶的方案就是使用微软提供的自动化工具集：[UI Automation](https://docs.microsoft.com/en-us/dotnet/framework/ui-automation/ui-automation-overview)。UI Automation是Microsoft .NET 3.0框架下提供的一种用于自动化测试的技术，是在MSAA基础上建立的，MSAA就是Microsoft Active Accessibility。

如果你使用过.NET 提供的UI Automation相关的类库，应该有一个直观的感受，就是非常啰嗦，举一个例子：

```csharp
AutomationElement ControlTypeComboBox = grdClassBook.FindFirst(
  TreeScope.Children,
  new PropertyCondition(AutomationElement.ControlTypeProperty, ControlType.ComboBox));

AutomationElement cellElement = ControlTypeComboBox.FindFirst(
  TreeScope.Children,
  new PropertyCondition(AutomationElement.AutomationIdProperty, "ListBox"));
```

每当你尝试去获取一个UI元素时，都需要使用`FindFirst`之类的方法去查询指定的`PropertyCondition`，而`PropertyCondition`使用起来也不简单，特别是当你需要拼接多个AND或者OR多个条件时。

```csharp
var btnCondition = new AndCondition(
  new PropertyCondition(AutomationElement.ControlTypeProperty, ControlType.Button),
  new PropertyCondition(AutomationElement.NameProperty, "ok"));
```

才两个条件就这么多代码了？你看看搞Web自动化的同学都可以用XPATH，快速定位和查询元素 `/div[@id='ok']`，多好。既然我们那么羡慕XPATH，那我们就搞一个出来，让做Windows桌面自动化的同学也可以High一把。

## WPATH实现原理

具体代码我就不在此展开，想刨根的同学可以直接移步至Github：https://github.com/tobyqin/wpath。

WPATH的主要原理就是通过反射的方式去获取当前方法或者属性的Attribute，在Attribute中我们可以定义类似于XPATH的语法，我 且称之为WPATH。最后经过表达式解析转换成对应的Find方法和Condition，举一个例子说明：

```Csharp
[WPath("/Edit[@id='txtId' or @Class='TextBox']")]
public AutomationElement EditControl
{
   get { return this.AppElement.FindByWPath(); }
}
```

当调用`FindByWPath()`时，该属性上的WPath Attribute就会被解析出来，其中的 `/`会被解析成FindFirst，`Edit`会被解析成ControlType.Edit，中括号里的条件最后被组合起来，调用的最终结果大致如下：

```csharp
public AutomationElement EditControl
{
   get
   {
     return this.AppElement.FindFirst(TreeScope.Children,
       new AndCondition(
          new PropertyCondition(AutomationElement.ControlTypeProperty, ControlType.Edit),
          new OrCondition(
            new PropertyCondition(AutomationElement.AutomationId, "txtId"),
            new PropertyCondition(AutomationElement.Class, "TextBox"))));
   }
}
```

 痛苦的感觉一下减轻许多，有没有？

## 更详细的WPATH用法

如果你要在项目中使用WPATH，可以通过nuget包安装：

```
PM> Install-Package WPath
```

### 简单说明

1. WPath 和 XPath 类似，以 '/' 开头。
2. 可以使用多个 '/' 来定位目标元素。
3. 节点名字来自于MSDN定义好的 [control type](https://docs.microsoft.com/en-us/dotnet/framework/ui-automation/ui-automation-control-types)。
4. 目前WPath支持的查询属性如下:

- `Name` (NameProperty)
- `ID` (AutomationIdProperty)
- `Class` (ClassNameProperty)
- `Enabled` (IsEnabledProperty)
- `FrameworkID` (FrameworkIdProperty)

### 举例子

> `/Group/Button`

- 获取第一个Group下的第一个Button。

> `//Button[@Name='Save']`

- 在子孙节点中获取第一个Name为 "Save" 的元素。

> `/[@Name='TabContainer']/Button[2]`

- 获取Name为 "TabContainer"的控件下的第二个Button，注意，控件类型名称可以为空。

> `/Button[@ID='AddButton' and @Name='Add']`

- 获取一个automation ID 为 'AddButton' **且** name 为 'Add' 的Button。

> `/Button[@ID='AddButton' or @Name='Add']`

- 获取一个automation ID 为 'AddButton' **或** name 为'Add'的Button。

> `/Button[first()]`

- 获取当前元素下第一个Button。

> `/Button[last()]`

- 获取当前元素下最后一个Button。

### 实际运用

推荐使用Attribute的方式进行调用，可用于类方法或者属性。

```csharp
[WPath("/Edit[@id='txtId' or @Class='TextBox']")]
public AutomationElement EditControl
{
   get { return this.AppElement.FindByWPath(); }
}

[WPath("/Button[first()]")]
public AutomationElement GetFirstButton()
{
   return this.AppElement.FindByWPath();
}
```

或者直接调用 `FindByWPath(path)` 来定位目标元素。

```csharp
var path = "/Edit[3]";
var e = this.AppElement.FindByWPath(path);
Assert.AreEqual("txtKey", e.Current.AutomationId);
Assert.AreEqual(ControlType.Edit, e.Current.ControlType);

path = "/Button[@name='OK']/Text[1]";
e = this.AppElement.FindByWPath(path);
Assert.AreEqual("OK", e.Current.Name);
Assert.AreEqual(ControlType.Text, e.Current.ControlType);
```

### 小贴士

- 元素类型节点是大小写不敏感的，比如：
  - @name = @Name
  - /edit = /Edit
- 父节点定位 `../` 目前不支持，因为有点复杂。

更多的说明建议还是去看Github中的说明文档，或者直接看[单元测试](https://github.com/tobyqin/wpath/blob/master/WPath.Tests/UnitTests.cs)。

## 后记

Windows UI 自动化的坑还是挺深的，填坑的人也不少，我推荐有需要的同学去学习和了解一下 [White](https://github.com/TestStack/White)。White是一个非常好UI Automation 封装框架，相信我，能省下你不少时间。
