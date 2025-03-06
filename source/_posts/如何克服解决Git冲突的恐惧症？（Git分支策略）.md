title: 如何克服解决Git冲突的恐惧症？（Git分支策略）
date: 2018-03-13 07:50:25
categories: Git
tags: [Git]
---
>git默认的是master分支，试想下，如果所有的开发都在master分支，想起来都比较混乱，那么有没有比较科学的分支策略呢？本篇将介绍git的分支策略，听我慢慢道来～

# 分支分类

正常分支：

+ master：主分支
+ develop：开发分支

临时分支：

+ feature：功能分支
+ release：预发布分支
+ fixbug：修补bug分支

![](https://user-gold-cdn.xitu.io/2018/3/10/162100f7945bace0?w=840&h=885&f=png&s=121275)

<!--more-->

# 主分支

首先，代码库应该有一个、且仅有一个主分支。

所有提供给用户使用的正式版本，都在这个主分支上发布。

Git主分支的名字，默认叫做Master。

它是自动建立的，版本库初始化以后，默认就是在主分支在进行开发。

# 开发分支

主分支只用来分布重大版本，日常开发应该在另一条分支上完成。我们把开发用的分支，叫做Develop。

这个分支可以用来生成代码的最新隔夜版本（nightly）。如果想正式对外发布，就在Master分支上，对Develop分支进行"合并"（merge）。

Git创建Develop分支的命令：
    
```java
git checkout -b develop master
```
    
将Develop分支发布到Master分支的命令：

```java
# 切换到Master分支
git checkout master
# 对Develop分支进行合并
git merge --no-ff develop
```

# 功能分支

功能分支，它是为了开发某种特定功能，从Develop分支上面分出来的。开发完成后，要再并入Develop。

功能分支的名字，可以采用feature-*的形式命名。

Git创建一个功能分支：

```java
git checkout -b feature-x develop
```

开发完成后，将功能分支合并到develop分支： 

```java
git checkout develop
git merge --no-ff feature-x
```

删除feature分支：
```java
git branch -d feature-x
```

# 预发布分支

预发布分支，它是指发布正式版本之前（即合并到Master分支之前），我们可能需要有一个预发布的版本进行测试。

预发布分支是从Develop分支上面分出来的，预发布结束以后，必须合并进Develop和Master分支。

它的命名，可以采用release-*的形式。

Git创建一个预发布分支：

```java
git checkout -b release-1.2 develop
```

确认没有问题后，合并到master分支：   

```java
git checkout master
git merge --no-ff release-1.2
# 对合并生成的新节点，做一个标签
git tag -a 1.2
```

再合并到develop分支：

```java
git checkout develop
git merge --no-ff release-1.2
```

最后，删除预发布分支：

```java
git branch -d release-1.2
```

# 修补bug分支

软件正式发布以后，难免会出现bug。这时就需要创建一个分支，进行bug修补。

修补bug分支是从Master分支上面分出来的。修补结束以后，再合并进Master和Develop分支。它的命名，可以采用fixbug-*的形式。

Git创建一个修补bug分支：

```java
git checkout -b fixbug-0.1 master
```

修补结束后，合并到master分支：  

```java
git checkout master
git merge --no-ff fixbug-0.1
git tag -a 0.1.1
```

再合并到develop分支：

```java
git checkout develop
git merge --no-ff fixbug-0.1
```

最后，删除"修补bug分支"：

```java
git branch -d fixbug-0.1
```

# 多人协作的工作模式

首先，可以试图用git push origin branch-name推送自己的修改；

如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！

如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。

>这就是多人协作的工作模式，一旦熟悉了，就非常简单。