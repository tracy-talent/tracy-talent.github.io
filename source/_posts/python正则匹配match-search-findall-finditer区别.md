---
title: 'python正则匹配match,search,findall,finditer区别'
date: 2020-10-31 14:08:34
tags:
	- python
categories:
	- language
	- python
---

python正则匹配常用到的函数主要主要有：

1. fullmatch/match，返回re.Match对象，Match对象调用group(0)返回整个匹配，调用group(1)或者group(2)等依次返回括号内的匹配
2. search，返回re.Match对象
3. findall，返回列表包含括号内的匹配
4. finditer，返回re.Match迭代器

示例代码如下：

```python
import re

content = '333STR1666STR299'
regex = r'[A-Z]+(\d)'

if __name__ == '__main__':
    print(re.match(regex, content)) ##content的开关不符合正则，所以结果为None。

    ##只会找一个匹配，match[0]是regex所代表的整个字符串，match[1]是第一个()中的内容，match[2]是第二对()中的内容。
    match = re.search(regex, content)
    print('\nre.search() return value: ' + str(type(match)))
    print(match.group(0), match.group(1))  

    result1 = re.findall(regex, content)
    print('\nre.findall() return value: ' + str(type(result1)))
    print(result1)
    for m in result1:
        print(m)

    result2 = re.finditer(regex, content)
    print('\nre.finditer() return value: ' + str(type(result2)))
    print(result2)
    for m in result2:
        print(m.group(0), m.group(1))  ##字符串
```

返回结果如下：

```
None

re.search() return value: <class 're.Match'>
STR1 1

re.findall() return value: <class 'list'>
['1', '2']
1
2

re.finditer() return value: <class 'callable_iterator'>
<callable_iterator object at 0x7ffa374dfad0>
STR1 1
STR2 2
```

