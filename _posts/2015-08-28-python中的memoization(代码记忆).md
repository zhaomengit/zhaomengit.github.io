---
layout: post
title: "Python-memoize"
date: 2015-08-28
category: Python
tags: Python
---

代码记忆是一种避免潜在的重复计算开销的方法。你通过缓存一个函数每次运行的结果来达到此目的。 
这样，下一次函数以同样的参数运行时，它将从缓存中返回结果，并不需要花费额外的时间来计算结果。

### 一个简单的例子

```python
factorial_memo = {}
def factorial(k):
    if k < 2: return 1
    if not k in factorial_memo:
        factorial_memo[k] = k * factorial(k-1)
    return factorial_memo[k]
```
通过factorial_memo这个dict来缓存结果.计算factorial(k)如果k已经计算过,在factorial_memo中会有缓存,直接返回

### 用类实现

```python
class Memoize:
    def __init__(self, f):
        self.f = f
        self.memo = {}
    def __call__(self, *args):
        if not args in self.memo:
            self.memo[args] = self.f(*args)
        return self.memo[args]
        
def factorial(k):
    if k < 2: return 1
    return k * factorial(k - 1)

factorial = Memoize(factorial)
```
### 用装饰器实现

```python
from functools import wraps

def memoize(func):
    cache = {}
    @wraps(func)
    def wrapper(k):
        if k not in cache:
            cache[k] = func(k)
        return cache[k]
    return wrapper
 
@memoize
def factorial(k):
    if k < 2: return 1
    return k * factorial(k - 1)
```
