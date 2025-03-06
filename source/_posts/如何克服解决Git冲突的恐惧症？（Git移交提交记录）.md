title: 如何克服解决Git冲突的恐惧症？（Git移交提交记录）
date: 2018-03-18 07:54:17
categories: Git
tags: [Git]
---
到现在我们已经学习了Git的基础知识，包括：

+ [如何克服解决Git冲突的恐惧症？（序）](http://hellomypastor.net/2018/03/05/%E5%A6%82%E4%BD%95%E5%85%8B%E6%9C%8D%E8%A7%A3%E5%86%B3Git%E5%86%B2%E7%AA%81%E7%9A%84%E6%81%90%E6%83%A7%E7%97%87%EF%BC%9F%EF%BC%88%E5%BA%8F%EF%BC%89/)
+ [如何克服解决Git冲突的恐惧症？（Git入门介绍）](http://hellomypastor.net/2018/03/06/%E5%A6%82%E4%BD%95%E5%85%8B%E6%9C%8D%E8%A7%A3%E5%86%B3Git%E5%86%B2%E7%AA%81%E7%9A%84%E6%81%90%E6%83%A7%E7%97%87%EF%BC%9F%EF%BC%88Git%E5%85%A5%E9%97%A8%E4%BB%8B%E7%BB%8D%EF%BC%89/)
+ [如何克服解决Git冲突的恐惧症？（Git基础篇--上）](http://hellomypastor.net/2018/03/07/%E5%A6%82%E4%BD%95%E5%85%8B%E6%9C%8D%E8%A7%A3%E5%86%B3Git%E5%86%B2%E7%AA%81%E7%9A%84%E6%81%90%E6%83%A7%E7%97%87%EF%BC%9F%EF%BC%88Git%E5%9F%BA%E7%A1%80%E7%AF%87-%E4%B8%8A%EF%BC%89/)
+ [如何克服解决Git冲突的恐惧症？（Git基础篇--下）](http://hellomypastor.net/2018/03/11/%E5%A6%82%E4%BD%95%E5%85%8B%E6%9C%8D%E8%A7%A3%E5%86%B3Git%E5%86%B2%E7%AA%81%E7%9A%84%E6%81%90%E6%83%A7%E7%97%87%EF%BC%9F%EF%BC%88Git%E5%9F%BA%E7%A1%80%E7%AF%87-%E4%B8%8B%EF%BC%89/)
+ [如何克服解决Git冲突的恐惧症？（Git分支策略）](http://hellomypastor.net/2018/03/13/%E5%A6%82%E4%BD%95%E5%85%8B%E6%9C%8D%E8%A7%A3%E5%86%B3Git%E5%86%B2%E7%AA%81%E7%9A%84%E6%81%90%E6%83%A7%E7%97%87%EF%BC%9F%EF%BC%88Git%E5%88%86%E6%94%AF%E7%AD%96%E7%95%A5%EF%BC%89/)
+ [如何克服解决Git冲突的恐惧症？（Git四大组件）](http://hellomypastor.net/2018/03/14/%E5%A6%82%E4%BD%95%E5%85%8B%E6%9C%8D%E8%A7%A3%E5%86%B3Git%E5%86%B2%E7%AA%81%E7%9A%84%E6%81%90%E6%83%A7%E7%97%87%EF%BC%9F%EF%BC%88Git%E5%9B%9B%E5%A4%A7%E7%BB%84%E4%BB%B6%EF%BC%89/)
+ [如何克服解决Git冲突的恐惧症？（Git高级篇）](http://hellomypastor.net/2018/03/15/%E5%A6%82%E4%BD%95%E5%85%8B%E6%9C%8D%E8%A7%A3%E5%86%B3Git%E5%86%B2%E7%AA%81%E7%9A%84%E6%81%90%E6%83%A7%E7%97%87%EF%BC%9F%EF%BC%88Git%E9%AB%98%E7%BA%A7%E7%AF%87%EF%BC%89/)

概念涵盖了Git 90%的功能，同样也足够满足开发者的日常需求。

> 然而, 剩余的10%在处理复杂的工作流时（或者当你陷入困惑时）可能就显示尤为重要了。

接下来要讨论的这个话题是“整理提交记录” ：开发人员有时会说“`我想要把这个提交放到这里,那个提交放到刚才那个提交的后面`”, 而接下来就讲的就是它的实现方式，看起来挺复杂, 其实是个很简单的概念。

<!--more-->

# git cherry-pick

第一个命令是`git cherry-pick`, 命令形式为:

```java
git cherry-pick <提交号>...
```

如果你想将一些提交复制到当前所在的位置（HEAD）下面的话，cherry-pick是最直接的方式了。我个人非常喜欢cherry-pick，因为它特别简单。

咱们还是通过例子来看一下！

这里有一个仓库, 我们想将 side 分支上的工作复制到 master 分支，你立刻想到了之前学过的`rebase`了吧？但是咱们还是看看 cherry-pick有什么本领吧。

```java
git cherry-pick C2 C4
```


![](https://user-gold-cdn.xitu.io/2018/3/18/1623974a20860eef?w=1024&h=768&f=gif&s=1838351)

这就是了！我们只需要提交记录C2和C4，所以Git就将被它们抓过来放到当前分支下了，就是这么简单!

# 交互式rebase

当你你知道你所需要的提交记录（并且还知道这些提交记录的哈希值）时, 用cherry-pick再好不过了，没有比这更简单的方式了。

但是如果你不清楚你想要的提交记录的哈希值呢?

幸好Git帮你想到了这一点, 我们可以利用交互式的rebase，如果你想从一系列的提交记录中找到想要的记录, 这就是最好的方法了

咱们具体来看一下：

交互式rebase指的是使用带参数`--interactive`的rebase命令, 简写为`-i`

如果你在命令后增加了这个选项, Git会打开一个UI界面并列出将要被复制到目标分支的备选提交记录，它还会显示每个提交记录的哈希值和提交说明，提交说明有助于你理解这个提交进行了哪些更改。

在实际使用时，所谓的UI窗口一般会在文本编辑器：如Vim中打开一个文件。

当rebase UI界面打开时, 你能做3件事:

+ 调整提交记录的顺序
+ 删除你不想要的提交
+ 合并提交

接下来看下具体命令：

![](https://user-gold-cdn.xitu.io/2018/3/18/1623948b8b1e9de5?w=1490&h=874&f=png&s=345569)

可以看到：

+ p, pick = use commit
+ r, reword = use commit, but edit the commit message
+ e, edit = use commit, but stop for amending
+ s, squash = use commit, but meld into previous commit
+ f, fixup = like "squash", but discard this commit's log message
+ x, exec = run command (the rest of the line) using shell
+ d, drop = remove commit

#### 拆分提交记录 edit

更改子提交pick为edit，表示需要修改此次提交；然后reset到需要拆分的上次提交，但是保留工区的内容，再依次commit工作区中的内容。

#### 合并提交记录 squash

更改子提交 pick 为squash，表示与当前提交的父提交合并。

#### 批量修改历史提交信息 reword

更改子提交 pick 为reword，表示修改历史提交信息。

#### 删除历史纪录

修改pick为drop, 或者直接删除所在的行。

>相信大家对Git移交提交记录已经基本掌握，不妨在自己的git环境中动手试一试吧～