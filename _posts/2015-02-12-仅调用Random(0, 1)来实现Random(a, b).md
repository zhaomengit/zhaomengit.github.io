---
layout: post
title: "仅调用Random(0, 1)来实现Random(a, b)"
date: 2015-02-16 16:25:06 -0700
comments: true
category: 算法
tags: 算法
---

### 题目描述
如何在只调用`RANDOM(0,1)`来实现`RANDOM(a, b)`

###步骤

1. 设 **n = b - a**
2. 找到最小的**c** 使**2^c ≥ n  (c = ⌈lgn⌉)**
3. 调用`RANDOM(0, 1)`函数**c**次,生成的0,1序列构成一个二进制数**r**
4.  如果r大于n重新开始第3步
5. 否则返回**a+r**即可
