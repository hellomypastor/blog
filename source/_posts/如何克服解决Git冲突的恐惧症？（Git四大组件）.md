title: 如何克服解决Git冲突的恐惧症？（Git四大组件）
date: 2018-03-14 07:51:40
categories: Git
tags: [Git]
---
# Git存储结构

Git有四大组件，分别是：

+ Tag
+ Commit
+ Tree
+ Blob

当git初始化后，目录下就生成了.git文件夹，存放着与git相关的所有内容，我们看下目录下具体的内容：

![](https://user-gold-cdn.xitu.io/2018/3/9/1620b296506043e3?w=1490&h=874&f=png&s=494715)

所有的组件都存放在objects文件夹中：

![](https://user-gold-cdn.xitu.io/2018/3/9/1620b2ace4e904ad?w=1452&h=314&f=png&s=178039)

<!--more-->

# Blob组件

当我们执行`git add README.md`后，文件夹内容如下：

![](https://user-gold-cdn.xitu.io/2018/3/10/1620d502f8be1603?w=1456&h=514&f=png&s=302898)

我们可以看到，目录中多了83目录，即blog组件，83目录中有文件名是一串UUID的文件，当我们执行git add将文件变为staged状态后，就会在objects目录创建一个组件，组件都是以hash的二进制方式进行存储，组件的名称为文件夹名称+文件名称，所有上面的blob组件的名字即为`83920ba13f0cd4e0046337313c1f0a1cfc676ad4`，这个名字是唯一的。

当修改README.md后再次执行git add，发现，objects目录中又多了一个blob组件：

![](https://user-gold-cdn.xitu.io/2018/3/10/1620d6371d7f7916?w=1454&h=376&f=png&s=269554)

注意：如果两个文件的内容一样的话，执行git add的时候，只会生成一个blob组件，不会是两个。blob组件是在代码提交到Stage区域的时候生成的，而且是以内容来生成一个字节码文件。

可以通过git hash-object来查询文件的hash码：

![](https://user-gold-cdn.xitu.io/2018/3/10/1620d68688672f0f?w=1456&h=106&f=png&s=66710)

# Commit组件

刚刚我们已经执行了两次git add，下面我们将变动提交，执行git commit：

```java
git commit -m "init"
```

![](https://user-gold-cdn.xitu.io/2018/3/10/1620d7266827a1f5?w=1456&h=550&f=png&s=314371)

可以看到，objects中多了两个文件夹，b6和da，这两个是什么呢？我们先用git log查看下提交日志：


![](https://user-gold-cdn.xitu.io/2018/3/10/1620d75440a806cf?w=1446&h=212&f=png&s=118173)

可以看到，commit的id为`da7b2dd822e576db1cfb0e546a9de57fc8cfbe8b`，所以da文件夹为commit组件，那么b6是什么呢？

# Tree组件

b6是tree组件，每次commit时，首先会创建commit组件，然后将涉及的文件信息创建tree组件，我们可以用git cat-file -p命令查看commit组件：

![](https://user-gold-cdn.xitu.io/2018/3/10/1620ff6716d46157?w=1456&h=240&f=png&s=208015)

可以看到，通过git cat-file -p命令查看commit组件，可以看到tree组件，我们用git cat-file -p来查看tree组件：

![](https://user-gold-cdn.xitu.io/2018/3/10/1620ffa41e6538bc?w=1454&h=106&f=png&s=99221)

可以看到，tree组件中记录了文件的基本信息。

# 底层运行流程

我们总结下git底层的运行流程：

![](https://user-gold-cdn.xitu.io/2018/3/6/161fb994093dae3b?w=638&h=359&f=jpeg&s=42249)

+ 当我们添加或者修改了文件并且add到stage区之后，会根据文件内容创建不同的blob
+ 当进行提交之后马上创建一个tree组件把需要的blob组件添加进去，之后再封装到一个commit组件中完成本次提交。
+ 在将来进行reset的时候可以直接使用git reset --hard xxxxx可以恢复到某个特定的版本
+ 在reset之后，git会根据这个commit组件的id快速的找到tree组件，然后根据tree找到blob组件，之后对仓库进行还原

>我们看到，git的整个过程都是以hash和二进制进行操作，所以git执行效率非常之高。