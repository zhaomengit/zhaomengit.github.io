---
layout: post
title: "Python库中functools中@waps的使用"
date: 2015-03-09
comments: true
categories: Python
tags: 函数式编程
---
###functools中@waps
使用装饰器的一个副作用就是，装饰之后函数丢失了它本来的`__name__`,`__doc__`及`__module__`属性。

```python
def my_decorator(f):
    def wrapper(*args, **kwds):
        print 'Calling decorated function'
        return f(*args, **kwds)
    return wrapper

@my_decorator
def example():
    """Docstring"""
    print 'Called example function'

example()
print example.__name__
print example.__doc__
```

输出的结果是:

    >>>Calling decorated function
    >>>Called example function
    >>>wrapper
    >>>None
    

使用functools.wraps中@wraps装饰器可以让丢失的函数属性`__name__`,`__doc__`,`__module__`恢复

```python
from functools import wraps
def my_decorator(f):
    @wraps(f)
    def wrapper(*args, **kwds):
        print 'Calling decorated function'
        return f(*args, **kwds)
    return wrapper

@my_decorator
def example():
    """Docstring"""
    print 'Called example function'

example()
print example.__name__
print example.__doc__
```

加入`@wraps`的输出结果是:

    >>>Calling decorated function
    >>>Called example function
    >>>example
    >>>Docstring
    

###@wraps实现原理
上面的例子，函数的调用顺序是:

    example() -> my_decorator(example)() -> 
    wraps(example, assigned = WRAPPER_ASSIGNMENTS, updated = WRAPPER_UPDATES)(wrapper)()

  查看wraps的实现代码是：

```python
  def wraps(wrapped, 
            assigned = WRAPPER_ASSIGNMENTS, 
            updated = WRAPPER_UPDATES):

    return partial(update_wrapper, 
                   wrapped=wrapped, 
                   assigned=assigned, 
                   updated=updated)
```

即:

    partial(update_wrapper, wrapped=example, assigned=assigned, updated=updated)(wrapper)()

**注意：调用函数`partial`之后，`wrapped=example`在函数partial参数keywords中**

`functools.partial`的大概的Python实现是：

```python
def partial(func, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = keywords.copy()
        newkeywords.update(fkeywords)
        return func(*(args + fargs), **newkeywords)
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```

因此`newfunc.func = update_wrapper`，`newfunc.args = example`

函数成为:`newfunc(wrapper)`，因此`newfunc`函数最终执行的是`update_wrapper(wrapper, wrapped = example,...)`
再看`update_wrapper`的实现.

```python
WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__doc__')
WRAPPER_UPDATES = ('__dict__',)
def update_wrapper(wrapper,
                   wrapped,
                   assigned = WRAPPER_ASSIGNMENTS,
                   updated = WRAPPER_UPDATES):
    for attr in assigned:
        setattr(wrapper, attr, getattr(wrapped, attr))
    for attr in updated:
        getattr(wrapper, attr).update(getattr(wrapped, attr, {}))
    return wrapper
```

###partial函数的使用
Python 中有个 `int([x[,base]])` 函数，作用是把字符串转换为一个普通的整型。
如果要把所有输入的二进制数转为整型，那么就要这样写 `int('11', base=2)`。这样写起来貌似不太方便，那么我们就能用 partial 来实现值传递一个参数就能转换二进制数转为整型的方法。

```python
from functools import partial
int2 = partial(int, base=2)

print int2('11') # 3
print int2('101') # 5
```

###附录：完整的实现（没有使用库函数）

```python
def partial_t(func, *args, **keywords):
    print "keywords is %s" % keywords
    print args
    def newfunc(*fargs, **fkeywords):
        newkeywords = keywords.copy()
        newkeywords.update(fkeywords)
        print fargs
        print "newkeywords is %s" % newkeywords
        return func(*(args + fargs), **newkeywords)
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc

WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__doc__')
WRAPPER_UPDATES = ('__dict__',)

def update_wrapper(wrapper,
                   wrapped,
                   assigned = WRAPPER_ASSIGNMENTS,
                   updated = WRAPPER_UPDATES):
    for attr in assigned:
        setattr(wrapper, attr, getattr(wrapped, attr))
    for attr in updated:
        getattr(wrapper, attr).update(getattr(wrapped, attr, {}))
    return wrapper

def wraps(wrapped,
          assigned = WRAPPER_ASSIGNMENTS,
          updated = WRAPPER_UPDATES):
    return partial_t(update_wrapper, wrapped=wrapped,
                   assigned=assigned, updated=updated)

def my_decorator(f):
    @wraps(f)
    def wrapper(*args, **kwds):
        print 'Calling decorated function'
        return f(*args, **kwds)
    return wrapper

@my_decorator
def example():
    """Docstring"""
    print 'Called example function'

example()
print example.__name__
print example.__doc__

```

输出结果:

```
keywords is {'wrapped': <function example at 0x0000000002649CF8>, 'assigned': ('__module__', '__name__', '__doc__'), 'updated': ('__dict__',)}
() #args
(<function wrapper at 0x0000000002649DD8>,) #fargs
newkeywords is {'wrapped': <function example at 0x0000000002649CF8>, 'assigned': ('__module__', '__name__', '__doc__'), 'updated': ('__dict__',)}
Calling decorated function
Called example function
example
Docstring
```