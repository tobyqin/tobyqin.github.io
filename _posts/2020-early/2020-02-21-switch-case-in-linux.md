---
title: Linux中的Switch Case
categories: [Tech]
tags: [Linux, bash, tips, shell]
date: 2020-02-21
---

如果`if`判断超过 3 次，那么可以考虑换成`switch case`了。

<!-- more -->

语法如下：

```sh
case EXPRESSION in

  PATTERN_1)
    STATEMENTS
    ;;

  PATTERN_2)
    STATEMENTS
    ;;

  PATTERN_N)
    STATEMENTS
    ;;

  *)
    STATEMENTS
    ;;
esac
```

`case`里还可以有一些语法：

```
?() - zero or one occurrences of pattern，匹配0次或1次
*() - zero or more occurrences of pattern，匹配0次或多次
+() - one or more occurrences of pattern，匹配1次或多次
@() - one occurrence of pattern，匹配其中的某一项
!() - anything except the pattern，匹配指定模式外的情况
```

举例说明：

```sh
# call functions based on arguments
case "$arg" in
    a*             ) foo;;    # matches anything starting with "a"
    b?             ) bar;;    # matches any two-character string starting with "b"
    c[de]          ) baz;;    # matches "cd" or "ce"
    me?(e)t        ) qux;;    # matches "met" or "meet"
    @(a|e|i|o|u)   ) fuzz;;   # matches one vowel
    m+(iss)?(ippi) ) fizz;;   # matches "miss" or "mississippi" or others
    * ) bazinga;; # catchall, matches anything not matched above
esac
```

实际上用起来不会那么高级，大概会是这样：

```sh
case "$ENV" in
    *DEV         ) xxx;;
    *QA|*UAT     ) yyy;;
    PROD         ) zzz;;
    * ) xxx;;
esac
```

如果要忽略大小写，就先把变量转一下再放到`case`里。

```sh
ENV=$( tr '[:upper:]' '[:lower:]' <<<"$ENV" )
```
