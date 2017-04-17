---
layout: post
title: "openat()函数避免TOCTTOU"
date: 2017-04-17
comments: true
category: Linux
tags: Linux
---

## openat()函数

`openat`函数是POSIX.1最新版本中新增的一类函数之一，希望解决两个问题。

第一，让线程可以使用相对路径名打开目录中的文件，而不再只能打开当前工作目录。 
第二，可以避免time-of-check-to-time-of-use（TOCTTOU）错误。

TOCTTOU错误的基本思想是：

如果有两个基于文件的函数调用，其中第二个调用依赖于第一个调用结果，那么程序是脆弱的。如果两个调用并不是原子操作，在两个函数调用之间文件可能改变了，这样也就造成了第一个调用的结果就不再有效，使得程序最终的结果是错误的。文件系统命名空间中的TOCTTOU错误通常处理的就是那些覆盖文件系统权限的小把戏，这些小把戏通过骗取特权程序降低特权文件的权限控制或者让特权文件打开一个安全漏洞等方式进行

## 例子:

实际代码:

```c
if (access("file", W_OK) != 0) {
   exit(1);
}

fd = open("file", O_WRONLY);
// Actually writing over /etc/passwd
write(fd, buffer, sizeof(buffer));
```

攻击代码:

```c
// 
//
// After the access check
symlink("/etc/passwd", "file");
// Before the open, "file" points to the password database
//
//
```

## 分析
大量的TOCTOU攻击被阻止; 那些依赖于目录被重命名（或可能被删除）而是使用新名称（例如，一些其他位置的符号链接）的攻击被阻止了，因为文件相对于原始目录继续创建，而不是使用重命名或被的替换目录。

## 参考

* [how-can-openat-avoid-tocttou-errors](http://stackoverflow.com/questions/36708171/how-can-openat-avoid-tocttou-errors)

