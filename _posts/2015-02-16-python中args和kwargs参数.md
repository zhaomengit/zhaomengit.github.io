---
layout: post
title: "Python中args和kwargs参数"
date: 2015-02-16 16:25:06 -0700
comments: true
categories: Python
---
### *args
通过使用\*args语法，一个python函数在它的参数列表中可以接受多个位置参数。 
\*args 将所有的非关键词参数组合到一个元组（tuple)里，这个元组可以在函数中访问得到。 
反过来，当\*args用在调用函数的参数列表时，它会将一个元组的参数展开成一系列的位置参数。

```python
# 形式参数示例
def function_with_many_arguments(*args):
    print args
 
# 此函数中的`args`将成为传递的所有参数的元组
# 可以在函数中像使用元组一样使用
function_with_many_arguments('hello', 123, True)
# >> ('hello', 123, True)
```

```python
# 实参示例
def function_with_3_parameters(num, boolean, string):
    print "num is " + str(num)
    print "boolean is " + str(boolean)
    print "string is " + string
 
arg_list = [1, False, 'decorators']
 
# 通过使用'*'号 arg_list将会被展开为三个位置参数
function_with_3_parameters(*arg_list)
# >> num is 1
# >> boolean is False
# >> string is decorators
```

在形式参数列表中，\*arg将放到一个名为args的元组中，在一个实参列表中，\*arg将扩展成一系统位置参数然后应用到函数中。
正如你在实参示例所看到那样，'*'符号可以和'args'之外的的名字使用。它只是一个在收缩和扩展通用列表时的约定形式。

### **kwargs
\*\*kwargs跟他的兄弟\*args相似，但是它是跟关键词参数相关而不是位置。
如果\*\*kwargs用在一个形式参数列表中，它将所有接收到的关键词参数收集进一个字典中。如果它用在一个函数的实参列表中。
它将把一个字典扩展成一系统的关键词参数。

```python
def function_with_many_keyword_args(**kwargs):
    print kwargs
    
function_with_many_keyword_args(a='apples', b='bananas', c='cantalopes')
# >> {'a': 'apples', 'b': 'bananas', 'c': 'cantalopes'}  
def multiply_name(count=0, name=''):
    print name * count
    
arg_dict = {'name': 'Brian','count': 3}
 
multiply_name(**arg_dict) # count参数为arg_dict的key,值3,name参数对应arg_dict的name,值为Brian
# >> BrianBrianBrian

def multiply_name(a, count=0, name=''):
    print a
    print name * count

arg_dict = {'name': 'BrianQ','count': 6}

multiply_name(a = 5, **arg_dict)
#输出结果:
#5 给变量a赋值了5
#BrianQBrianQBrianQBrianQBrianQBrianQ

```

