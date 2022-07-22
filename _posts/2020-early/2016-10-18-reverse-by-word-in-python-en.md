---
title: Reverse string by word with Python
date: 2016-10-18 09:46:00
tags: [python, regex]
categories: Quiz
---

**Reverse string by word** is a very popular interview question. In python you can solve it easily with code like below.

<!-- more -->

```python
def reverse_string_by_word(s):
    lst = s.split()  # split by blank space by default
    return ' '.join(lst[::-1])

s = 'Power of Love'
print reverse_string_by_word(s)
# Love of Power

s = 'Hello    World!'
print reverse_string_by_word(s)
# World! Hello
```

We can see above implementation is good but not enough, in 2nd string we are expecting the `!` symbol should be reversed as well, and keep original blank spaces between words. (multiple spaces between `Hello` and `World` in the example)

```python
print reverse_string_by_word(s)
# Expected: !World  Hello
```

To improve the solution, a better choice should be `re` module. You might want to take a look at [`re.split()`](https://docs.python.org/2/library/re.html#re.split) method.

```python
>>> import re
>>> s = 'Hello  World!'

>>> re.split(r'\s+', s)    # will discard blank spaces
['Hello', 'World!']

>>> re.split(r'(\s+)', s)  # will keep spaces as a group
['Hello', '  ', 'World!']

>>> s = '< Welcome to EF.COM! >'

>>> re.split(r'\s+', s)  # split by spaces
['<', 'Welcome', 'to', 'EF.COM!', '>']

>>> re.split(r'(\w+)', s)  # exactly split by word
['< ', 'Welcome', ' ', 'to', ' ', 'EF', '.', 'COM', '! >']

>>> re.split(r'(\s+|\w+)', s)  # split by space and word
['<', ' ', '', 'Welcome', '', ' ', '', 'to', '', ' ', '', 'EF', '.', 'COM', '!', ' ', '>']

>>> ''.join(re.split(r'(\s+|\w+)', s)[::-1])
'> !COM.EF to Welcome <'

>>> ''.join(re.split(r'(\s+)', s)[::-1])
'> EF.COM! to Welcome <'

>>> ''.join(re.split(r'(\w+)', s)[::-1])
'! >COM.EF to Welcome< '

```

If you would like to increase the readability a little bit, replacing list slicing to `reversed()` is a choice.

```python
>>> ''.join(reversed(re.split(r'(\s+|\w+)', s)))
'> !COM.EF to Welcome <'
```

Bingo, so easy!
