title: 如何克服解决Git冲突的恐惧症？（Git高级话题）
date: 2018-03-20 07:55:41
categories: Git
tags: [Git]
---
# 多分支rebase

多分支的情况下，我们往往希望得到有序的提交历史，看下面的例子：

![](https://user-gold-cdn.xitu.io/2018/3/20/16243be85bfa1d62?w=880&h=419&f=png&s=26832)

执行如下步骤进行多分支rebase：

```java
git rebase master bugFix
git rebase bugFix side
git rebase side another
git rebase another master
```

![](https://user-gold-cdn.xitu.io/2018/3/20/16243c2acbce1666?w=273&h=669&f=png&s=24669)

<!--more-->

# 两个父节点

操作符`^`与`~`符一样，后面也可以跟一个数字。

但是该操作符后面的数字与~后面的不同，并不是用来指定向上返回几代，而是指定合并提交记录的某个父提交。还记得前面提到过的一个合并提交有两个父提交吧，所以遇到这样的节点时该选择哪条路径就不是很清晰了。

Git默认选择合并提交的“第一个”父提交，在操作符^后跟一个数字可以改变这一默认行为。

废话不多说，举个例子：

![](https://user-gold-cdn.xitu.io/2018/3/20/16243ca0a2420e26?w=532&h=555&f=png&s=23445)

```java
//链式操作
git branch bugWork master~^2~
```

![](https://user-gold-cdn.xitu.io/2018/3/20/16243cb3f790e8f5?w=673&h=555&f=png&s=27729)

# 纠缠不清的分支

![](https://user-gold-cdn.xitu.io/2018/3/20/16243cf4b1a4eabc?w=264&h=517&f=png&s=18476)

如上图，现在我们的master分支是比one、two和three要多几个提交。出于某种原因，我们需要把master分支上最近的几次提交做不同的调整后，分别添加到各个的分支上。

one需要重新排序并删除C5，two仅需要重排排序，而three只需要提交一次。

执行如下命令：

```java
git checkout one
git cherry-pick C4 C3 C2
git checkout two
git cherry-pick C5 C4 C3 C2
git branch -f three C2
```

![](https://user-gold-cdn.xitu.io/2018/3/20/16243e68de653a4f?w=636&h=517&f=png&s=34988)

>相信大家对Git高级话题已经基本掌握，不妨在自己的git环境中动手试一试吧～