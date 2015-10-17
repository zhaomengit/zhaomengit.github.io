---
layout: post
title: "Python中的@classmethod和@staticmethod"
date: 2015-03-08
comments: true
categories: Python
tags: 函数式编程
---
`@classmethod` 和 `@staticmethod` 相似, 但是有以下区别: `@classmethod` 的第一个参数必须是`class object`的引用, `@staticmethod` 则不需要.

看下面的例子：
###模板代码

假设一个处理日期信息的类：

```python
class Date(object):

    day = 0
    month = 0
    year = 0

    def __init__(self, day=0, month=0, year=0):
        self.day = day
        self.month = month
        self.year = year
```

这个类显然能够处理日期信息（没有时区信息，并且在UTC下显示）

函数`__init__`, 接受参数作为构造函数 `instancemethod`, 参数(self)，是不能省略的， 是新创建对象的引用

###类方法

我们使用`@classmethod`可以很好的完成很多任务.

假设我们要创建大量的`Date`类的实例，这些类的实例包含的信息来自外部的`('dd-mm-yyyy')`格式的字符串,我们不得不在其他的地方添加代码。

我们这样做：

1. 把字符串解析成包含day，month，year 3 个整数的元祖
2. 通过 `Date` 的构造函数来实例化

    day, month, year = map(int, string_date.split('-'))
    date1 = Date(day, month, year)

C++中称为重载, Python缺少这种特性，我们使用`@classmethod`来创建一个"constructor".

```python
@classmethod
def from_string(cls, date_as_string):
    day, month, year = map(int, date_as_string.split('-'))
    date1 = cls(day, month, year)
    return date1

date2 = Date.from_string('11-09-2012')
```

仔细查看以上代码的实现，有以下好处：

1. 解析字串的部分可以重复使用了
2. 封装性良好
3. `cls` 是一个持有 **class itself** 的对象, 不是一个类的实例. 如果我们继承 `Date` 类, 所有的子类中都有 `from_string`.

###静态函数

`@staticmethod`和 `@classmethod` 相似，不需要强制性的参数.

看如下使用方式：
我们需要一个字符串校验`Date`. 这个任务仍然在 `Date`中,但是不需要实例

这时候 `@staticmethod` 就有用了：

```python
@staticmethod
def is_date_valid(date_as_string):
    day, month, year = map(int, date_as_string.split('-'))
    return day <= 31 and month <= 12 and year <= 3999
```

可以看出`@staticmethod`的用法, 我们不必关心这个类是什么，像普通函数一样调用,但是没有权限访问这个对象及对象的属性和方法, 然而 `@classmethod`可以.

###其他例子
上面的例子, 使用`@classmethod` 修饰`from_string` 作为利用其它参数创建 `Date`对象的工厂，使用 `@staticmethod`同样可以做到

```python
class Date:
  def __init__(self, month, day, year):
    self.month = month
    self.day   = day
    self.year  = year

  def display(self):
    return "{0}-{1}-{2}".format(self.month, self.day, self.year)

  @staticmethod
  def millenium(month, day):
    return Date(month, day, 2000)

new_year = Date(1, 1, 2013)               # Creates a new Date object
millenium_new_year = Date.millenium(1, 1) # also creates a Date object. 

# Proof:
new_year.display()           # "1-1-2013"
millenium_new_year.display() # "1-1-2000"

isinstance(new_year, Date) # True
isinstance(millenium_new_year, Date) # True
```

`new_year`和`millenium_new_year`都是`Date`的实例.

如果你仔细观察，创建`Date`的方法是硬编码的，不管`Date`是啥。 意思就是即使`Date`是一个子类, 这个子类依然能够创建出`Date`对象 (没有子类的任何属性). 看下面的例子:

```python
class DateTime(Date):
  def display(self):
      return "{0}-{1}-{2} - 00:00:00PM".format(self.month, self.day, self.year)


datetime1 = DateTime(10, 10, 1990)
datetime2 = DateTime.millenium(10, 10)

isinstance(datetime1, DateTime) # True
isinstance(datetime2, DateTime) # False

datetime1.display() # returns "10-10-1990 - 00:00:00PM"
datetime2.display() # returns "10-10-2000" because it's not a DateTime object but a Date object. Check the implementation of the millenium method on the Date class
```

`datetime2`不是`DateTime`类型，这是应为使用`@staticmethod`修饰符的缘故。

大多情况没什么问题，但是如果你想要一个能根据调用者的类型来创建对象的工厂方法的话，需要使用`@classmethod`。

重写`Date.millenium`，只提供了改变的部分

```python
@classmethod
def millenium(cls, month, day):
    return cls(month, day, 2000)
```

确保`class`不是硬编码. `cls`能使任何子类. 返回的`object`一定是`cls`的实例. 

```python
datetime1 = DateTime(10, 10, 1990)
datetime2 = DateTime.millenium(10, 10)

isinstance(datetime1, DateTime) # True
isinstance(datetime2, DateTime) # True


datetime1.display() # "10-10-1990 - 00:00:00PM"
datetime2.display() # "10-10-2000 - 00:00:00PM"
```

这些就是使用`@classmethod`和`@staticmethod`的区别