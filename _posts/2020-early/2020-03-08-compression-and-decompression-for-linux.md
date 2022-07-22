---
title: Linux的压缩和解压
categories: [Tech]
tags: [linux，zip，unzip，tar]
date: 2020-03-08
---

最常用的是 zip 和 tar 命令。

<!-- more -->

## 1.zip 命令

```bash
# 将指定目录压缩成zip文件
zip -r compressed.zip /path/to/dir

# 压缩文件夹，但排除某些文件
zip -r compressed.zip path/to/dir -x path/to/exclude

# 将多个目录压缩成zip
zip -r compressed.zip /path/to/dir1 /path/to/dir2 /path/to/file

# 压缩加密码
zip -e -r compressed.zip path/to/dir

# 添加文件到已经存在的zip
zip compressed.zip path/to/file

# 删除zip里的文件
zip -d compressed.zip "foo/*.tmp"
```

zip 可以将当前文件夹压缩至当前文件夹，比如 /home/toby => /home/toby/toby.zip

## 2.unzip 命令

与 zip 命令相反，这是解压命令。

```bash
# 解压文件，用空格来接受多个文件
unzip file(s)

# 解压文件到指定目录
unzip compressed_file(s) -d /path/to/put/extracted_file(s)

# 显示zip里的文件，不解压
unzip -l file.zip
```

## 3.tar 命令

```bash
# 只是打包成 tar
tar cf target.tar file1 file2 file3

# 打包并使用gzip压缩
tar czf target.tar.gz file1 file2 file3

# 解压tar到当前目录
tar xf source.tar[.gz|.bz2|.xz]

# 解压tar到指定目录
tar xf source.tar -C directory

# 显示tar里的文件，不解压
tar tvf source.tar

# 解压tar里符合规则的文件
tar xf source.tar --wildcards "*.html"
```
