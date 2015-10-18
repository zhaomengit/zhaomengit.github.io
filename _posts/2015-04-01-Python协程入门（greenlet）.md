---
layout: post
title: "Python协程入门（greenlet）"
date: 2015-04-01
comments: true
category: Python
tags: Python
---
###一个例子

```python
from greenlet import greenlet

def test1(name):
    print name
    gr2.switch()
    print "over"

def test2():
    print "test-2"
    return "test2-fjsfjs"

    
gr1 = greenlet(test1)
gr2 = greenlet(test2)
value = gr1.switch("fjsfjs")

print gr1.parent
print gr2.parent
print greenlet.getcurrent()
print value
```

输出结果:

```
fjsfjs
test-2
<greenlet.greenlet object at 0x0000000002465508>
<greenlet.greenlet object at 0x0000000002465508>
<greenlet.greenlet object at 0x0000000002465508>
test2-fjsfjs
```

###解释

看到`test1`的最后那个输出并没有打印出来程序就已经结束了，也就是`gr2`执行完了之后，代码的执行并没有在`test1`返回，而是直接在最外层开始继续运行。。。而且返回结果也是直接到了最外层，而不是在test1中。。。

这个是为什么呢。。？
在`greenlet`模块的初始话的时候，就会先建立一个最顶层的`greenle`t对象，他属于`greenlet`层次关系中的`root`节点，接下来在创建`greenlet`的时候，如果在构造函数中并没有指明`parent`对象，那么构建出来的`greenlet`对象将会默认将当前环境所在的`greenlet`作为parent，当这个`greenlet`对象执行完成之后，将会调度其`parent`继续执行，而且会将返回参数也传递过去。

###参考

* http://blog.csdn.net/fjslovejhl/article/details/38821673
