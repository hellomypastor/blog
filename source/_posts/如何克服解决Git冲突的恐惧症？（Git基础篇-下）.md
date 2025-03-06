title: 如何克服解决Git冲突的恐惧症？（Git基础篇--下）
date: 2018-03-11 07:48:00
categories: Git
tags: [Git]
---
在上一篇中，介绍了git的初始化配置配置、获取帮助、初始化仓库、跟踪新文件、提交、忽略某些文件，以及分支，具体文章：[如何克服解决Git冲突的恐惧症？（Git基础篇--上）](http://hellomypastor.net/2018/03/07/%E5%A6%82%E4%BD%95%E5%85%8B%E6%9C%8D%E8%A7%A3%E5%86%B3Git%E5%86%B2%E7%AA%81%E7%9A%84%E6%81%90%E6%83%A7%E7%97%87%EF%BC%9F%EF%BC%88Git%E5%9F%BA%E7%A1%80%E7%AF%87-%E4%B8%8A%EF%BC%89/)，本篇将介绍分支合并相关的`git merge`与`git rebase`。

# merge

分支合并的方法一就是git merge，通过图示更容易理解：

执行命令如下：

```java
git merge bugFix
git checkout bugFix
git merge master
```

执行过程如下：


![](https://user-gold-cdn.xitu.io/2018/3/8/1620636c82b68567?w=1024&h=768&f=gif&s=1432430)

# rebase

分支合并的方法二就是git rebase，通过图示更容易理解：

执行命令如下：

```java
git rebase master
//下面两步只是示例，不建议使用
git checkout master
git rebase bugFix
```

执行过程如下：

![](https://user-gold-cdn.xitu.io/2018/3/8/16206385e6ce291b?w=1024&h=768&f=gif&s=2051942)

<!--more-->

# merge与rebase的对比

Merge好在它是一个安全的操作。现有的分支不会被更改，避免了rebase潜在的缺点，另一方面，这同样意味着每次合并上游更改时feature分支都会引入一个外来的合并提交。如果master非常活跃的话，这或多或少会污染你的分支历史。虽然高级的git log选项可以减轻这个问题，但对于开发者来说，还是会增加理解项目历史的难度。

Rebase最大的好处是你的项目历史会非常整洁。首先，它不像git merge 那样引入不必要的合并提交。其次，rebase导致最后的项目历史呈现出完美的线性——你可以从项目终点到起点浏览而不需要任何的Fork。这让你更容易使用git log、git bisect和gitk来查看项目历史。

不过，这种简单的提交历史会带来两个后果：安全性和可跟踪性。如果你违反了Rebase黄金法则，重写项目历史可能会给你的协作工作流带来灾难性的影响。此外，rebase不会有合并提交中附带的信息——你看不到feature分支中并入了上游的哪些更改。

# rebase黄金法则

当你理解rebase是什么的时候，最重要的就是什么时候不能用rebase。git rebase的黄金法则便是，绝不要在公共的分支上使用它。

# rebase冲突解决

假设有两个分支，master与bugFix：

master分支的README.md文件内容如下：

```java
史培培
```

bugFix分支的README.md文件内容如下：

```java
码上论剑

欢迎关注我的公众号

http://hellomypastor.net
```

在bugFix分支执行如下命令：

```java
git pull --rebase
```

发现冲突：

```java
<<<<<<< HEAD
史培培
=======
码上论剑

欢迎关注我的公众号

http://hellomypastor.net
>>>>>>> init
```

解决冲突之后，执行：

```java
git add README.md
git rebase --continue
```

这样就解决冲突了，是不是很简单？

# 建议

用pull --rebase，而不用pull（默认merge），这样的话在pull的时候就自行在本地解决两路冲突，而不是merge的时候麻烦的多路merge，这才是git的正确使用方式。


> 相信大家对git的基础已经基本掌握，不妨在自己的git环境中动手试一试，下篇将讲述《Git分支管理策略》，主要介绍git分支的管理相关内容，敬请期待～