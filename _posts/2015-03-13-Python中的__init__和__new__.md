---
layout: post
title: "__init__和__new__"
date: 2015-03-13
comments: true
category: Python
tags: Python
---
###`__init__`方法
`__init__(self, […)`此方法为类的初始化方法。当构造函数被调用的时候的任何参数都将会传给它。(比如如果我们调用 x = SomeClass(10, 'foo'))，那么 `__init__` 将会得到两个参数10和foo。 `__init__` 在Python的类定义中被广泛用到。

注意：参数self是类的实例，已经创建完成，通过`__init__`添加改变属性等。
下面的例子中`__init__`没有执行，因为float类型的`__init__`忽略了参数就返回了，float是不可变的类型，所以这时候可以使用`__new__`

    >>> class inch(float):
    ...     "THIS DOESN'T WORK!!!"
    ...     def __init__(self, arg=0.0):
    ...         float.__init__(self, arg*0.0254)
    ...
    >>> print inch(12)
    12.0
    >>> 

###`__new__`方法
`__new__(cls, [...) ` `__new__` 是在一个对象实例化的时候所调用的第一个方法。它的第一个参数是这个类，其他的参数是用来直接传递给 `__init__` 方法。 `__new__` 方法相当不常用,但是它有自己的特性，特别是当继承一个**不可变**的类型比如一个tuple或者string。

    >>> class inch(float):
    ...     "Convert from inch to meter"
    ...     def __new__(cls, arg=0.0):
    ...         return float.__new__(cls, arg*0.0254)
    ...
    >>> print inch(12)
    0.3048
    >>>

###举例
`__new__`返回的一定是类对象，`__new__`执行完之后，就执行返回对象的`__int__`函数。
看下面的例子：（只是用来测试，实际不这么用）

```python
class A:
    def __init__(self):
        print "A __init__"


class Test(object):
    def __new__(cls):
        print "cls type is %s" % cls
        print "test __new__"
        return A()

    def __init__(self):
        print "test __init__"

t = Test()
print isinstance(t, A)
```

结果：

    cls type is <class '__main__.Test'>
    test __new__
    A __init__
    True

创建对象的时候调用了Test的`__new__`，然后调用了A的`__int__`

###总结：`object.__new__(cls[, ...])`

1. `__new__`是一个静态方法，第一个参数是这个类本身(如果外面调用必须是类)，其余的参数供构造函数使用
2. 返回一个实例，然后调用该实例的`__init__`方法
3. `__new__`必须返回一个对象
4. 主要用于对不可变对象（eg:int, str, 或 tuple）的定制
5. 对没有从Object继承的类没有`__new__`，因为`super()`和`__new__`等只支持`new-style class`

```python
class A:
    def __new__(cls):
        print "a __new__"

    def __init__(self):
        print "a __init__"

a = A()
# outputs: a __init__ 
```

###实现单例

```python
class Singleton(object):
    def __new__(cls, *args, **kwds):
        it = cls.__dict__.get("__it__")
        if it is not None:
            return it
        cls.__it__ = it = object.__new__(cls)
        it.init(*args, **kwds)
        return it
    def init(self, *args, **kwds):
        pass
class MySingleton(Singleton):
    def init(self):
        print "calling init"
    def __init__(self):
        print "calling __init__"

x = MySingleton()
# calling init
# calling __init__
assert x.__class__ is MySingleton
y = MySingleton()
# calling __init__
assert x is y
```

###参考资料：
1. [官方文档：Unifying types and classes in Python 2.2](https://www.python.org/download/releases/2.2/descrintro/#cooperation)
2. [Python 魔术方法指南](http://pycoders-weekly-chinese.readthedocs.org/en/latest/issue6/a-guide-to-pythons-magic-methods.html)