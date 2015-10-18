---
layout: post
title: "Python-Decorator"
date: 2015-03-06 16:25:06 -0700
comments: true
category: Python
tags: 函数式编程
---
### 函数（Functions）是第一个对象
在Python中， 一切都是对象。这意味着即使一个函数被其他对象所包装，我们仍然可以通过这个对象名来进行调用。 举个列子:

```python
def traveling_function():
    print "Here I am!"
 
function_dict = {
    "func": traveling_function
}
 
trav_func = function_dict['func']
trav_func()
# >> Here I am!
```

`traveling_function`尽管在`function_dictdictionary`中被指定为`func`这个`key`的`value`， 但是我们仍然可以正常的使用。

### 在高阶函数中使用第一类函数

我们可以以类似其它对象的方式传递对象。它可以是字典的值、列表的项或是另一个对象的属性。
那么，我们不能将函数以参数的方式传递给另一个函数么？可以！函数作为参数传递给高阶函数。

```python
def self_absorbed_function():
    return "I'm an amazing function!"
 
def printer(func):
    print "The function passed to me says: " + func()
 
# Call `printer` and give it `self_absorbed_function` as an argument
printer(self_absorbed_function)
# >> The function passed to me says: I'm an amazing function!
```

上面就是将函数作为参数传递另一个函数，并进行处理的示例。用这种方式，我们可以创造很多有趣的函数，例如 `decorator`。

### Decorator 基础

从心而论，`decorator` 只是将函数作为参数的函数。通常，它的作用是返回封装好的经过修饰的函数。
下面这个简单的身份识别 `decorator` 可以让我们了解 `decorator` 是如何工作的。

```python
def identity_decorator(func):
    def wrapper():
        func()
    return wrapper
 
 
def a_function():
    print "I'm a normal function."
 
# `decorated_function` is the function that `identity_decorator` returns, which
# is the nested function, `wrapper`
decorated_function = identity_decorator(a_function)
 
# This calls the function that `identity_decorator` returned
decorated_function()
# >> I'm a normal function
```

在这里，`identity_decoratordoes` 并没有修改其封装的函数。它仅仅是返回了这个函数，
调用了作为参数传递给 `identity_decorator` 的函数。这个 `decorator` 毫无意义！
有趣的是在 `identity_decorator` 中，虽然函数没有传递给`wrapper` ，它依然那可以被调用，这是因为闭包原理。

#### 闭包原理

闭包是个花哨的术语，意思是当一个函数被声明，在其被声明的词法环境中，它都可以被引用。
上例中，当 `wrapper` 被定义时，它就访问了本地环境中的函数变量。一旦 `identity_decorato`r 返回，
你就只能通过 `decorated_function` 访问函数。在 `decorated_function` 的闭包环境之外，函数并非以变量形式存在的。

### 简单的 decorator

让我们来创建一个简单的,有点实际功能的Decorator. 这个Decorator所做的就是记录他所包装的方法被调用的次数。

```python
def logging_decorator(func):
    def wrapper():
        wrapper.count += 1
        print "The function I modify has been called {0} times(s).".format(
              wrapper.count)
        func()
    wrapper.count = 0
    return wrapper
 
 
def a_function():
    print "I'm a normal function."
 
modified_function = logging_decorator(a_function)
 
modified_function()
# >> The function I modify has been called 1 time(s).
# >> I'm a normal function.
 
modified_function()
# >> The function I modify has been called 2 time(s).
# >> I'm a normal function.
```

我们说过一个`Decorator`会修改另外一个方法, 这个例子可能会帮助你理解这其中的意思. 
正如你在这个例子中所看到的一样, `logging_decorator`所返回的新方法和`a_function`很相似，只是多增加了日志功能。

在这个例子中，`logging_decorator`接收一个方法作为参数, 返回另一个包装过的方法. 
每当`logging_decorator`返回的方法被调用的时候, 他会为`wrapper.count`加1, 打印`wrapper.count`的值, 
然后再调用`logging_decorator`所包装的方法.

你可能会问为什么我们要把`counter`作为`wrapper`的属性而不是一个普通的变量呢.
包装器的闭包特性不是可以让我们访问其本地作用域中所申明的变量吗? 是的, 但是有一个小问题.
在Python中, 闭包完整的提供了对方法作用域链中变量的读权限，但只为同样作用域中的可变对象(比如, 列表, 字典等)提供了写权限. 
然而, Python中的整数类型是不可变的, 所以我们无法对`wrapper`中的整数变量加1. 
取而代之的是，我们把`counter`作为`wrapper`的一个属性，它就变成了一个可变对象，这样我们就可以对它进行自增操作了。

###Decorator语法

在上一个例子中，我们看到一个Decorator可以接受一个方法作为参数，然后在该方法上再包装上其他方法。
一旦你熟悉了装饰器(Decorator), Python还为你提供了一个特定的语法使得它看上去更直观，更简单。

```python
# In the previous example, we used our decorator function by passing the
# function we wanted to modify to it, and assigning the result to a variable
 
def some_function():
    print "I'm happiest when decorated."
 
# Here we will make the assigned variable the same name as the wrapped function
some_function = logging_decorator(some_function)

# We can achieve the exact same thing with this syntax:
 
@logging_decorator
def some_function():
    print "I'm happiest when decorated."
```

Decorator语法的简要工作原理:

1. 当Python的解释器看到这个被装饰的方法时，先编译这个方法(`some_function`), 然后先给它赋一个名字 `some_function`.
2. 这个方法(`some_function`)再被传入装饰方法`(decorator function)logging_decorator`中
3. 装饰方法`(decorator function)logging_decorator`返回的新方法替代原来的`some_function`方法, 与`some_function`这个名字绑定.

记住这些步骤，再来仔细看一下`identity_decoratora`方法和它注释.

```python
def identity_decorator(func):
    # Everything here happens when the decorator LOADS and is passed
    # the function as described in step 2 above
    def wrapper():
        # Things here happen each time the final wrapped function gets CALLED
        func()
    return wrapper
```

希望这里的注释能起到一定的引导作用. 只有在装饰器所返回的方法中的指令才会在每次调用的时候被执行. 
在被返回函数外的指令只会被执行一次-- 在第二步 当装饰器第一次接受一个方法的时候。

###类的装饰器
上面我们说到装饰器是修饰函数的函数。凡事总有个但是。我们还可以用它来修饰类或方法。
虽然一般不会这么用它。但有些情况下用来替代元类也未尝不可。

```python
foo = ['important', 'foo', 'stuff']
 
 
def add_foo(klass):
    klass.foo = foo
    return klass
 
 
@add_foo
class Person(object):
    pass
 
brian = Person()
 
print brian.foo
# >> ['important', 'foo', 'stuff']
```

现在任何从`Person`实例出来的对象都会包含 foo 属性，注意到我们的装饰器函数没，它没有返回一个函数，而是一个类。
所以赶紧更新一下刚才我们对装饰器的定义吧：装饰器是一个可以修饰函数，类或方法的函数。
### 类作为装饰器
装饰器不仅仅可以装饰一个类，它可以作为一个类来使用。一个装饰器的唯一需求是他的返回值可以被调用。 这意味着当你调用一个对象时它必须实现`__call__`这个魔幻般的在幕后调用的方法。函数设置了这个隐式方法。让我们重新建立 `identity_decorators`作为一个类，然后来看它是怎么运作的。

```python
class IdentityDecorator(object):
    def __init__(self, func):
        self.func = func
 
    def __call__(self):
        self.func()
 
 
@IdentityDecorator
def a_function():
    print "I'm a normal function."
 
a_function()
# >> I'm a normal function
```

* 当IdentityDecorator装饰器装饰了`a_function`,它表现仅仅是一个装饰器也是一个函数。这个片段相当于这个例子的装饰语法： `a_function = IdentityDecorator(a_function)`. 这个类装饰器被调用并实例化，然后把它作为参数传递给它所装饰的函数。
* 当`IdentityDecorator`实例化，它的初始化函`__init__`与当做参数传递进来的装饰函数一起调用。 这种情况下，它所做的一切就是分派给这个函数一个属性，随后可以被其它方法访问。
* 最后,当`a_function`（真实的返回的`IdentityDecorator`对象包装了`a_function`）被调用，这个对象的`__call__` 方法被引用进来，由于这仅仅是一个有标识的装饰器，它可以简单的调用它所装饰的函数。

让我们再一次更新我们的装饰器:装饰模式可以做为修改函数、方法或者类来被调用。

###带参数的装饰器

有时你需要根据不同的情况改变装饰器的行为，这可以通过传递参数来完成。

```python
from functools import wraps
 
def argumentative_decorator(gift):
    def func_wrapper(func):
        @wraps(func)
        def returned_wrapper(*args, **kwargs):
            print "I don't like this " + gift + " you gave me!"
            return func(gift, *args, **kwargs)
        return returned_wrapper
    return func_wrapper
 
@argumentative_decorator("sweater")
def grateful_function(gift):
    print "I love the " + gift + "! Thank you!"
 
grateful_function()
# >> I don't like this sweater you gave me!
# >> I love the sweater! Thank you!
```

让我们看看如果我们不用装饰器语法，装饰器函数是怎么运作的：

```python
# If we tried to invoke without an argument:
grateful_function = argumentative_function(grateful_function)
 
# But when given an argument, the pattern changes to:
grateful_function = argumentative_decorator("sweater")(grateful_function)
```

主要的要关注的是当给定一些参数，装饰器会首先被引用并带有这些参数——就像平时包装过的函数并不在此列。 然后这个函数调用返回值， 装饰器已经包装的这个函数已经传递给初始化后的带参数的装饰器的返回函数。（这种情况下，返回值是`(argumentative_decorator("swearter"))`.

1. 解释器到达装饰过的函数, 编译`grateful_function`, 并把它绑定给`grateful_fucntion`这个名字.
2. `argumentativ_decorator`被调用, 并传递参数`sweater`, 返回`func_wrapper`. 
3. `func_wrapper`被调用, 并传入`grateful_function`作为参数, `func_wrapper`返回`returned_wrapper`.
4. 最后,`returned wrapper`被替代为原始的函数`grateful_function`, 然后被绑定到`grateful function`这个名字下.

###带可选参数的装饰器
有许多方法使用带可选参数的装饰器。这取决于你是要用一个位置参数还是关键字参数，或者两个都用。在使用上可能有一点点不同。下面就是其中的一种方法:

```python
from functools import wraps
GLOBAL_NAME = "Brian"
 
def print_name(function=None, name=GLOBAL_NAME):
    def actual_decorator(function):
        @wraps(function)
        def returned_func(*args, **kwargs):
            print "My name is " + name
            return function(*args, **kwargs)
        return returned_func
 
    if not function:  # User passed in a name argument
        def waiting_for_func(function):
            return actual_decorator(function)
        return waiting_for_func
 
    else:
        return actual_decorator(function)

@print_name
def a_function():
    print "I like that name!"
 
@print_name(name='Matt')
def another_function():
    print "Hey, that's new!"
 
a_function()
# >> My name is Brian
# >> I like that name!
 
another_function()
# >> My name is Matt
# >> Hey, that's new!
```

如果我们需要传`name`到`print_name`方法里面，他将会和之前的`argumentative_decoratorin`效果相同。也就是说，第一个`print_name`将会把`name`作为它的参数。函数在第一次请求时返回的值将会传递到函数里。

如果没有向`print_name`传`name`的参数，将会报缺少修饰的错。它将会像单参数函数一样发送请求。
`print_name`有这两种可能。他要检查收到的参数是不是一个被包装的函数。

如果不是的话，返回`waiting_for_func` 函数来请求被包装的函数。如果收到的是一个函数参数，它将会跳过中间的步骤，立刻请求`actual_decorator`。

###链式装饰器
装饰器的最后一个特性吧：链式。你可以在任意给定的函数中放置多个装饰器。 它使用一种类似用多继承来构造类的方式来构造函数。但是最好不要过于追求这种方式。

```python
@print_name('Sam')
@logging_decorator
def some_function():
    print "I'm the wrapped function!"
 
some_function()
# >> My name is Sam
# >> The function I modify has been called 1 time(s).
# >> I'm the wrapped function!
```

当你将装饰器链接在一起时，他们在放置在栈中顺序是从下往上。被包装的函数，`some_fuction`，编译之后，
传递给在它之上的第一个装饰器（`loging_decorator`).然后第一个装饰器的返回值传递给下一个。它将以这样的式传递给链中的每一个装饰器。
因为我们这里用到的装饰器都是打印一个值然后返回传递给它们的函数。
这意味着在链中的最后一个装饰器，`print_name`，当被包装（装饰）的函数调用时，将打印整个输出的第一行。

###总结
我认为decorator最大的好处之一是它能让你从高些的层次进行抽象。 使用decorator，也能实现代码重用，从而节省时间，简化调试，使得反射更容易。
###参考
* [Python Decorator 和函数式编程](http://www.oschina.net/translate/decorators-and-functional-python)
