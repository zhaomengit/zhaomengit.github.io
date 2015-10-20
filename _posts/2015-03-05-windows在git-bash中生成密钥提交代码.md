---
layout: post
title: "使用git提交代码"
modified: 2014-12-19
comments: true
category: Linux
tags: git
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

###注意三

git支持很多种工作流程，我们采用的一般是这样，远程创建一个主分支，本地每人创建功能分支，日常工作流程如下：

1.去自己的工作分支

```
$ git checkout work
```

2.工作
....

提交工作分支的修改
```
$ git commit -a
```

3.回到主分支

```
$ git checkout master
```

4.获取远程最新的修改，此时不会产生冲突

```
$ git pull
```

5.回到工作分支

```
$ git checkout work
```

6.用rebase合并主干的修改，如果有冲突在此时解决

```
$ git rebase master
```

7.回到主分支

```
$ git checkout master
```

8.合并工作分支的修改，此时不会产生冲突。

```
$ git merge work
```

9.提交到远程主干

```
$ git push
```

这样做的好处是，远程主干上的历史永远是线性的。每个人在本地分支解决冲突，不会在主干上产生冲突。