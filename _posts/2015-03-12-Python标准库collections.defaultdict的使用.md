---
layout: post
title: "Python标准库collections.defaultdict的使用"
date: 2015-03-12
comments: true
categories: Python
tags: Python
---
###defaultdict Examples
`collections.defaultdict`是`dict`的一个子类，定义是

    class collections.defaultdict([default_factory[, ...]]) 

至少提供一个参数，这个参数会初始化给dict的`default_factory`属性，这个属性在`dict`中默认为`None`，如果这个返回的值为空的话会导致的引发`KeyError`异常.

```
>>> d = dict()
>>> print d['x']
Traceback (most recent call last):
  File "G:\PythonTest\test1.py", line 2, in <module>
    print d['x']
KeyError: 'x'
```

再说一下`default_factory`属性，是在`__missing__()`函数中使用的，通过构造函数初始化而来。
对于`__missing__(key)`函数：

    >>> from collections import defaultdict
    >>> print defaultdict.__missing__.__doc__
        __missing__(key) # Called by __getitem__ for missing key; pseudo-code:
            if self.default_factory is None: raise KeyError(key)
            self[key] = value = self.default_factory()
            return value

可以看出当使用`__getitem__()`方法访问一个不存在的键时(`dict[key]`这种形式实际上是`__getitem__()`方法的简化形式)，会调用`__missing__()`方法获取默认值，并将该键添加到字典中去。`dict`会使用`__getitem__()`来返回值，值不存在会返回`None`而不是`default_factory`。在`key`值不存在的条件下，如果`default_factory`不为`None`，会使用`default_factory`构造一个默认值，赋值给`dict[key]`

####example 1

```
>>> s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
>>> d = defaultdict(list)  # 默认初始化为list
>>> for k, v in s:
        # k不存在的时候会创建一个空的list给d[k],然后d[k]就返回一个list，可以进行append
...     d[k].append(v)
...
>>> d.items()
[('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]
```

####example 2
`dict.setdefault()`方法接收两个参数，第一个参数是健的名称，第二个参数是默认值。假如字典中不存在给定的键，则返回参数中提供的默认值；反之，则返回字典中保存的值。

```
>>> d = {}
>>> for k, v in s:
...     d.setdefault(k, []).append(v)
...
>>> d.items()
[('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]
```

####example 3
字符第一次出现，dict中没有键值，`default_factory`调用`int()`构造一个默认值

```
>>> s = 'mississippi'
>>> d = defaultdict(int)
>>> for k in s:
        # 统计某个字符出现的次数，如果不使用defaultdict(int)而使用dict，会出现KeyError
...     d[k] += 1
...
>>> d.items()
[('i', 4), ('p', 2), ('s', 4), ('m', 1)]
```

####example 4
可以通过函数初始化：
`itertools.repeat(value).next`是`itertools.repeat`一个函数,传入的是函数名，确保能够被调用

```
>>> def constant_factory(value):
...     return itertools.repeat(value).next
>>> d = defaultdict(constant_factory('<missing>'))
>>> d.update(name='John', action='ran')
>>> '%(name)s %(action)s to %(object)s' % d # name匹配dict中的key
'John ran to <missing>'
>>> print d
defaultdict(<method-wrapper 'next' of itertools.repeat object at 0x00000000022F8B38>, 
{'action': 'ran', 'object': '<missing>', 'name': 'John'})
```

####example 5

```
>>> s = [('red', 1), ('blue', 2), ('red', 3), ('blue', 4), ('red', 1), ('blue', 4)]
>>> d = defaultdict(set)
>>> for k, v in s:
...     d[k].add(v)
...
>>> d.items()
[('blue', set([2, 4])), ('red', set([1, 3]))]
```