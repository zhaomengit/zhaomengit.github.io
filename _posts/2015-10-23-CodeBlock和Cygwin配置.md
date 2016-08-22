---
layout: post
title: "Codeblock和Cygwin配置"
date: 2015-10-23
comments: true
category: 工具
tags: Cygwin
---

### 使用Codeblock和Cygwin在win7下开发Linux程序

环境:win7,codeblock13.12,Cygwin(这个木有版本貌似)

1. 下载Cygwin,然后安装,注意安装包要选上gcc,g++,make,gdb....
2. 下载Codeblock然后安装
3. 配置Codeblock

    1. Settings-->Compiler-->Global Complier Settings
    2. 选择"Cygwin Compiler", 点击"Copy"按钮

    配置如下图:
        ![配置路径](/images/Cygwin_sd.JPG)
        ![配置toolchain](/images/Cygwin_toolchain.JPG)

4. 配置gdb注册表,如果不配置调试的时候就出现以下错误:

    ```
    Cannot open file: >>/cygdrive/e/code/test/main.cpp
    At >>/cygdrive/e/code/test/main.cpp:17
    ```

5. 注册表配置代码:

    ```
    手动增加以下注册表项:
    HKEY_CURRENT_USER\\Software\\Cygnus Solutions\\Cygwin\\mounts v2

    HKEY_LOCAL_MACHINE\\Software\\Cygnus Solutions\\Cygwin\\mounts v2

    然后右击,新建字符串项,名称:cygdrive prefix 数据值:/cygdrive

    ```