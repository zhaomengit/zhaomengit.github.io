---
layout: post
title: "Python中subprocess获取stdout和stderr"
date: 2016-09-09
comments: true
category: Python
tags: Python
---

### 说明

Python中执行`shell`脚本可以通过`commands`模块中的`commands.getstatusoutput`来获取执行脚本的输出,返回的状态码,但是这个模块已经不推荐使用了,官网上执行脚本给出的解决方案是使用`subprocess`模块

### 使用subprocess

### 方案一

用subprocess的时候，获取stdout和stderr:

```python
import subprocess
p = subprocess.Popen(['tail','-10','/tmp/hosts.txt'],stdin=subprocess.PIPE,stdout=subprocess.PIPE,stderr=subprocess.PIPE,shell=False)

stdout,stderr = p.communicate()
print 'stdout : ',stdout
print 'stderr : ',stder
```

popen调用的时候会在父进程和子进程建立管道，然后我们可以把子进程的标准输出和错误输出都重定向到管道，然后从父进程取出。上面的communicate会一直阻塞，直到子进程跑完。这种方式是不能及时获取子程序的stdout和stderr。

### 方案二

主要是用来获取实时的输出信息

```python
p = subprocess.Popen("/etc/service/tops-cmos/module/hadoop/test.sh", shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
returncode = p.poll()
while returncode is None:
        line = p.stdout.readline()
        returncode = p.poll()
        line = line.strip()
        print line
print returncode
```

这里就是把错误输出重定向到PIPE对应的标准输出，也就是说现在stderr都stdout是一起的了，下面是一个while，poll回去不断查看子进程是否已经被终止，如果程序没有终止，就一直返回None，但是子进程终止了就返货状态码，甚至于调用多次poll都会返回状态码。上面的demo就是可以获取子进程的标准输出和标准错误输出。

### 注意事项

`subprocess.Popen` 函数中`stderr`如果不重新定向,会导致错误输出获取不到
