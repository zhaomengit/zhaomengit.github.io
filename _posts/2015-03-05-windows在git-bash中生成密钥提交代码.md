---
layout: post
title: windows下生成ssh-key提交代码
modified: 2014-12-19
categories: Linux
tags: git
comments: true
---

###官方文档
https://help.github.com/articles/generating-ssh-keys/
###注意一:

在第3步中有个命令是这样的

    # start the ssh-agent in the background
    eval 'ssh-agent -s'
    # Agent pid 59566


在windows下应该`eval 'ssh-agent -s'`这个命令是执行失败的，理应为

    eval `ssh-agent -s`

单引号 <em>'</em> 转换成 反引号 <em>`</em>

###注意二

通过命令

    ssh-add ~/.ssh/id_rsa

添加`id_rsa`登录的再次登录的时候会使用这个key，如果添加的可以的名字不为`id_rsa`，而是其他名称比如生成的可以放在`G:/myblog/gitkey`，这样需要在`.ssh`中进行配置。

方法是在`.ssh`文件夹中增加`config`文件，文件内容是：

    # Default github user(youremail@xxx.com)
    Host github.com
    HostName github.com
    User git
    IdentityFile G:/myblog/key/gitkey
