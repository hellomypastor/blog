title: 如何克服解决Git冲突的恐惧症？（Git基础篇--上）
date: 2018-03-07 07:46:38
categories: Git
tags: [Git]
---
# 初始化配置
我们安装了git之后，都要先配置以下git工作环境。git提供了git config的工具，专门用来配置或读取相应的工作环境变量。

配置：

```java
git config --global user.name "hellomypastor"
git config --global user.email 18013963220@163.com
```

查看配置：

```java
git config --list //方式一
git config -l //方式二
```

这些配置一般会存在三个地方：

+ /etc/gitconfig：全局配置（针对所有用户）
+ ~/.gitconfig：全局配置（针对某个用户）
+ .git/config：局部配置（针对某个目录/项目）

<!--more-->

# 获取帮助

git help可以获取帮助，使用如下：

```java
git commit --help //方式一
git help commit //方式二
```

# 在工作目录中初始化新仓库

git init可以将任何目录转化为git版本库，使用方法如下：

```java
git init
```

初始化后，在当前目录下会出现一个名为 .git 的目录，所有 Git 需要的数据和资源都存放在这个目录中。

# 跟踪新文件

初始化后，我们在目录中增加一个README.md文件，如果要跟踪这个文件，那么执行如下命令：

```java
git add README.md
```

执行后，我们可以执行git status，可以看到，README.md已被跟踪起来：

```java
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   README.md
```

# 提交

```java
git commit -m "init version"
[master (root-commit) 4dfc094] init version
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

执行完，执行git status查看状态：

```java
 On branch master
nothing to commit, working tree clean
```

下面我用简单gif动图进行示例git commit的效果：

执行命令如下：

```java
git commit -m "c2"
git commit -m "c3"
```

执行过程如下：

![](https://user-gold-cdn.xitu.io/2018/3/7/162009ecae2b78b5?w=1024&h=768&f=gif&s=1022937)

# 忽略某些文件

一般我们总会有些文件无需纳入Git 的管理，也不希望它们总出现在未跟踪文件列表，比如说编译文件、日志、配置文件、环境文件等等，我们可以创建一个名为 .gitignore 的文件，列出要忽略的文件模式，看一个例子：

```java
# 此为注释 – 将被 Git 忽略
# 忽略所有 .a 结尾的文件
*.a
# 但 lib.a 除外
!lib.a
# 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
/TODO
# 忽略 build/ 目录下的所有文件
build/
# 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
doc/*.txt
# 忽略 doc/ 目录下所有扩展名为 txt 的文件
doc/**/*.txt
```

# 分支

分支相关命令如下：

```java
//查看分支
git branch
* master
//新建分支
git branch bugFix
  bugFix
* master
//新建分支并切换到分支
git checkout -b bugFix
* bugFix
  master
```

下面我用简单gif动图进行示例git commit的效果：

执行命令如下：

```java
git branch bugFix
git commit -m "c2"
git checkout bugFix
git commit -m "c3"
```

执行过程如下：

![](https://user-gold-cdn.xitu.io/2018/3/7/16200d5bc77b46e9?w=1024&h=768&f=gif&s=1120392)

> 相信大家对git的基础命令已经基本掌握，不妨在自己的git环境中动手试一试，下篇将讲述《Git基础篇--下》，主要介绍git merge与git rebase，敬请期待～