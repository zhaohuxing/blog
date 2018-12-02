---
categories: [develop]
tags: [python]
date: 2018-12-02T08:05:00Z
title: Function Annotation/f-strings Self Experience 
---

### Function Annotation

以前只知道 Python 不需要指定类型，写个方法也不需要带类型，如下所示：

```
def add(a, b):
    return a + b
```

按照如上的写法，并伴随着项目一点一点变大，很容易搞混参数类型，导致一些不必要的问题。

[Function Annotation](https://www.python.org/dev/peps/pep-3107/) 提供了很好的解决方案，可以让我们指定参数，有点像写静态语言的感觉。

```
def add(a:int, b:int) -> int:
	return a + b
```

### Literal String Interpolation 

在 Python 中，常用的字符串拼接方法有：`%-formatting`, `str.format`.

```
msg = 'disk failure'

# %-formatting
print('error: %s' % msg)

# str.format
print('error: {}'.format(msg))
```

如上，可读性是不是很差？！[Literal String Interpolation](https://www.python.org/dev/peps/pep-0498/) such as `f-strings` 解决可读性差的痛点。

```
msg = 'disk failure'

# f-strings
print(f'error: {msg}')

```
使用过程中, dict 里的值解析不出来，len() 后的值也解析不出来，有点尴尬!
