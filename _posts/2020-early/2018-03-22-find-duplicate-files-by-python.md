---
title: 使用Python查找目录中的重复文件
categories: [Tech]
tags: [python]
date: 2018-03-22
---

是这样的，电脑上的堆积的照片有点多，而且重复的照片被放在了不同的目录，占用的空间越来越大，数量也多得已经不太适合人工分辨整理，写个Python脚本来处理吧。

<!-- more -->

## 文件的唯一标识 - MD5

假如你要处理的重复文件有不同的文件名，最简单的办法就是通过MD5来确定两个文件是不是一样的。

```Python
def md5sum(filename, blocksize=65536):
    hash = hashlib.md5()
    with open(filename, "rb") as f:
        for block in iter(lambda: f.read(blocksize), b""):
            hash.update(block)
    return hash.hexdigest()
```

这个方法可以快速获得一个文件的MD5值，`blocksize` 可以根据文件大小和CPU性能调整，一般选择的值约等于文件的平均大小。

## 保存所有文件标识和路径

接下来遍历所有文件，使用MD5作为key，路径作为value，保存起来。

```Python
dup = {}

def build_hash_dict(dir_path, pattern='*.jpg'):
    
    def save(file):
        hash = md5sum(file)
        if hash not in dup.keys():
            dup[hash] = [file]
        else:
            dup[hash].append(file)

    p = Path(dir_path)
    for item in p.glob('**/' + pattern):
        save(str(item))
```

## 处理重复文件

最后一步非常简单，把上一步建立的字典做一个简单的过滤就能找到重复文件。

```python
def get_duplicate():
    return {k: v for k, v in dup.items() if len(v) > 1}

for hash, files in get_duplicate().items():
    print("{}: {}".format(hash, files))
```

接下来你可以根据自己的需要删除或者保留某个路径下的文件，本文到此为止。

>  完整的脚本代码： https://gist.github.com/tobyqin/9299d27bdb429ffaa7713ed760a44fbb