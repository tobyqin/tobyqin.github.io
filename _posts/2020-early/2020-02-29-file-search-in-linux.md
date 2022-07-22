---
title: Linux中的文件搜索
categories: [Tech]
tags: [linux, find, grep]
date: 2020-02-29
---

我们经常需要搜索文件名或者文件内容。

<!-- more -->

## 搜索文件名

可以用`find`命令。

```
find < path > < expression > < cmd >
```

- `path`： 所要搜索的目录及其所有子目录。默认为当前目录。
- `expression`： 所要搜索的文件的特征。
- `cmd`： 对搜索结果进行特定的处理。

```bash
# 搜索包含指定字符串的文件名
find / -name "*Docker*"

# 无视大小写用iname
find ./ -iname "*.config"

# 忽略错误，比如没权限访问某个目录会打印一堆错误
find ./ -name "*.json" 2>/dev/null

# 搜索大于100M的文件
find / -size +100M -exec du -h {} \; 2>/dev/null

# 搜索0kb的文件并删除
find ./ -size 0 | xargs rm -f &
```

`find`命令功能非常强大，具体请查阅文档。

## 搜索文件内容

可以用 grep 命令。

```bash
# 搜索包含docker的文件，并打印命中行
grep -nr "docker" ./

# 搜索包含dokcer的文件，只打印文件名
grep -lr "docker" ./

# 只搜索文本文件，忽略二进制文件
grep -lrI "docker" ./

# 搜索匹配正则表达式的文件
egrep -lr "^docker" ./

# 搜索当前目录下的某些文件的内容
grep -lr "docker" *.py
```

`egrep` 是 `grep` 的正则表达式版本，`grep`还支持很多参数，具体请查阅文档。查找文件内容也可以用 `find` 命令。

```bash
find ./ -name "*.py" -exec grep -l "docker" {} \;
```

用 `find` 可以先对文件名或者类型先做一次过滤，再具体到内容搜索。
