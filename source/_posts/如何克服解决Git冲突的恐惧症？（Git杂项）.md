title: 如何克服解决Git冲突的恐惧症？（Git杂项）
date: 2018-03-19 07:57:02
categories: Git
tags: [Git]
---
>上篇介绍了如何克服解决Git冲突的恐惧症？（Git移交提交记录），本篇我们将介绍Git杂项。

<!--more-->

# 只取一个记录

来看一个在开发中经常会遇到的情况：我正在解决某个特别棘手的 Bug，为了便于调试而在代码中添加了一些调试命令并向控制台打印了一些信息。

这些调试和打印语句都在它们各自的提交记录里。最后我终于找到了造成这个Bug的根本原因，解决掉以后觉得沾沾自喜！

最后就差把bugFix分支里的工作合并回master分支了。

![](https://user-gold-cdn.xitu.io/2018/3/19/1623ebd4ba0a682d?w=265&h=440&f=png&s=17602)

你可以选择通过fast-forward快速合并到master分支上，但这样的话master分支就会包含我这些调试语句了。你肯定不想这样，应该还有更好的方式……

实际我们只要让Git复制解决问题的那一个提交记录就可以了。跟之前我们在“移交提交记录”中学到的一样，我们可以使用：

+ git rebase -i
+ git cherry-pick

解决上述问题，可以使用如下命令：

```java
git checkout master
git cherry-pick C4
```

![](https://user-gold-cdn.xitu.io/2018/3/19/1623ebc5edb02b72?w=521&h=383&f=png&s=18511)

# 提交的技巧1

接下来这种情况也是很常见的：你之前在newImage分支上进行了一次提交，然后又基于它创建了caption分支，然后又提交了一次。

此时你想对的某个以前的提交记录进行一些小小的调整。比如设计师想修改一下newImage中图片的分辨率，尽管那个提交记录并不是最新的了。

我们可以通过下面的方法来克服困难：

先用git rebase -i将提交重新排序，然后把我们想要修改的提交记录挪到最前，然后用commit --amend来进行一些小修改，接着再用git rebase -i来将他们调回原来的顺序，最后我们把master移到修改的最前端（用你自己喜欢的方法），就大功告成啦！

当然完成这个任务的方法不止上面提到的一种（我知道你在看cherry-pick啦），之后我们会多点关注这些技巧啦，但现在暂时只专注上面这种方法。

![](https://user-gold-cdn.xitu.io/2018/3/19/1623ec77b1f1581e?w=290&h=363&f=png&s=15262)

```java
git rebase -i caption~2 --aboveAll
git commit --amend
git rebase -i caption~2 --aboveAll
git branch -f master caption
```

![](https://user-gold-cdn.xitu.io/2018/3/19/1623ec9f9437b47d?w=557&h=398&f=png&s=17946)

最后有必要说明一下目标状态中的那几个'，我们把这个提交移动了两次，每移动一次会产生一个'；而C2上多出来的那个是我们在使用了amend参数提交时产生的，所以最终结果就是这样了。

# 提交的技巧2

我们可以使用rebase -i对提交记录进行重新排序。只要把我们想要的提交记录挪到最前端，我们就可以很轻松的用--amend修改它，然后把它们重新排成我们想要的顺序。

但这样做就唯一的问题就是要进行两次排序，而这有可能造成由rebase而导致的冲突。下面还是看看git cherry-pick是怎么做的吧。

要在心里牢记cherry-pick可以将提交树上任何地方的提交记录取过来追加到HEAD上（只要不是HEAD上游的提交就没问题）。

来看看这个例子：

![](https://user-gold-cdn.xitu.io/2018/3/19/1623ecac0755a3d9?w=290&h=363&f=png&s=15262)

```java
git checkout master
git cherry-pick C2
git commit --amend
git cherry-pick C3
```

![](https://user-gold-cdn.xitu.io/2018/3/19/1623eccb6aa62d2a?w=556&h=383&f=png&s=19597)

# Git Tag

相信通过前面的例子你已经发现了：分支很容易被人为移动，并且当有新的提交时，它也会移动。分支很容易被改变，大部分分支还只是临时的，并且还一直在变。

你可能会问了：有没有什么可以永远指向某个提交记录的标识呢，比如软件发布新的大版本，或者是修正一些重要的Bug或是增加了某些新特性，有没有比分支更好的可以永远指向这些提交的方法呢？

当然有了！Git的tag就是干这个用的啊，它们可以（在某种程度上 —— 因为标签可以被删除后重新在另外一个位置创建同名的标签）永久地将某个特定的提交命名为里程碑，然后就可以像分支一样引用了。

更难得的是，它们并不会随着新的提交而移动。你也不能检出到某个标签上面进行修改提交，它就像是提交树上的一个锚点，标识了某个特定的位置。

咱们来看看标签到底是什么样：

![](https://user-gold-cdn.xitu.io/2018/3/19/1623ed1a2fea794e?w=270&h=363&f=png&s=11100)

```java
git tag v0 C1
git tag v1 C2
```


![](https://user-gold-cdn.xitu.io/2018/3/19/1623ed0ccb0f45ee?w=266&h=363&f=png&s=14176)

# Git Describe

由于标签在代码库中起着“锚点”的作用，Git还为此专门设计了一个命令用来描述离你最近的锚点（也就是标签），它就是git describe！

Git Describe能帮你在提交历史中移动了多次以后找到方向；当你用git bisect（一个查找产生Bug的提交记录的指令）找到某个提交记录时，或者是当你坐在你那刚刚度假回来的同事的电脑前时，可能会用到这个命令。

git describe的语法是：

```java
git describe <ref>
```

<ref>可以是任何能被Git识别成提交记录的引用，如果你没有指定的话，Git会以你目前所检出的位置（HEAD）。

它输出的结果是这样的：

```java
<tag>_<numCommits>_g<hash>
```

tag表示的是离ref最近的标签，numCommits是表示这个ref与tag相差有多少个提交记录，hash表示的是你所给定的ref所表示的提交记录哈希值的前几位。

当ref提交记录上有某个标签时，则只输出标签名称。


![](https://user-gold-cdn.xitu.io/2018/3/19/1623ed57a725d7b6?w=507&h=383&f=png&s=18847)

```java
git describe master
//输出
v1_2_gC2

git describe side
//输出
v2_1_gC4
```

>相信大家对Git杂项已经基本掌握，不妨在自己的git环境中动手试一试吧～