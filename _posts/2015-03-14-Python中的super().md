---
layout: post
title: "Python中super()解析"
date: 2015-03-13
comments: true
category: Python
tags: Python
---

### super

    super(type[, object-or-type]) 

返回一个代理类来执行父类或者兄弟类的方法.
如果参数`object-or-type`省略，返回的对象不会绑定，如果`object-or-type`是一个对象，那么函数`isinstance(obj, type)` 必须为`TRUE`，如果`object-or-type`，那么`issubclass(type2, type)`必须为`TRUE`

```python
class A(object):   # 只能是new-style class
    def __init__(self):
        print "enter A"
        print "leave A"

class B(A):
    def __init__(self):
        print "enter B"
        print super(B, self)
        super(B, self).__init__()
        print "leave B"

b = B()
```

结果：

    enter B
    <super: <class 'B'>, <B object>> # 看到了super执行后的类型
    enter A
    leave A
    leave B

为什么会返回兄弟类？看下面的例子：

```python
class A(object):
    def __init__(self):
        print "enter A"
        print "leave A"

class B(object):
    def __init__(self):
        print "enter B"
        print "leave B"

class C(A):
    def __init__(self):
        print "enter C"
        super(C, self).__init__()
        print "leave C"

class D(A):
    def __init__(self):
       print "enter D"
       super(D, self).__init__()
       print "leave D"

class E(B, C):
    def __init__(self):
        print "enter E"
        B.__init__(self)
        C.__init__(self)
        print "leave E"

class F(E, D):
    def __init__(self):
        print "enter F"
        E.__init__(self)
        D.__init__(self)
        print "leave F"

F()
```

输出：

```
 enter F
 enter E
 enter B
 leave B
 enter C
 enter D
 enter A
 leave A
 leave D
 leave C
 leave E
 enter D
 enter A
 leave A
 leave D
 leave F
```

类A和类D的初始化函数被重复调用了2次，C的调用为什么会调用D的`__init__`？

```
        object
        /  \
       /    A
      /    /  \    
      B   C    D
      \  /    /
       E     /
        \   /
          F
```

这里涉及一个叫做`Method resolution order`(mro)的概念，super是以`__mro__`的顺序来创建的，这个图的mro顺序是：

     F E B C D A object

调用`super(C, self).__init__()`的时候会去找C前面的，就是D了，因此调用了D的`__init__`

如果让`F E B C D A object`类构造只调用一遍，从`F-A`依次调用`super`即可

A,B，E，F类中调用

```python
class A(object):
    def __init__(self):
        print "enter A"
        super(A, self).__init__()
        print "leave A"

class B(object):
    def __init__(self):
        print "enter B"
        super(B, self).__init__()
        print "leave B"
#....
class E(B, C):
    def __init__(self):
        print "enter E"
        super(E, self).__init__()
        print "leave E"

class F(E, D):
    def __init__(self):
        print "enter F"
        super(F, self).__init__()
        print "leave F"
```
### 参考资料
* [官方文档：Unifying types and classes in Python 2.2](https://www.python.org/download/releases/2.2/descrintro/#cooperation)