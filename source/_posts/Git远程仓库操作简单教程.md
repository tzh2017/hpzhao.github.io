---
title: Git远程仓库操作简单教程
date: 2017-02-25 17:17:31
tags: Git
categories: Git

---
# 准备工作
本教程只涉及远程仓库的一些基本操作，不涉及更细节的问题。
首先在Github上创建了一个`git_usage`的项目。然后创建了两个分支，一个分支是*master*分支，该分支下有一个名为`m1.txt`的文件，另一个分支是*b1*分支，该分支目录下有一个名为`b1.txt`的文件。
# 从远程仓库到本地仓库

## 克隆项目: git clone

首先是从远程服务器克隆项目。默认的远程主机是`origin`。

```Git
git clone git@github.com:hpzhao/git_usage.git
```

## 查看分支: git branch

接下来先查看下本地分支:  

```git
git branch
```
我们可以得到如下图的结果：

<image src='/images/git_branch.png' width="40%" height="20%" />

我们看到只有一个master分支。因为git clone默认只会拷贝master分支的数据到本地，所以本地不会有其他分支，但是会有远程分支的索引。接下来我们利用`-a`参数来查看所有的分支：  

```git
git branch -a
```

结果如下所示：  


<image src='/images/git_branch-a.png' width="50%" height="40%" />

这里我们看到了前缀有`remote`的分支。这就是未同步到本地的分支。下面会讲解如何将远程分支同步到本地。

## 从远程仓库获取分支：git checkout

我们需要用`git checkout`来建立新的分支并获取相应远程分支的数据。 

```
git checkout --track origin/b1
```

得到的结果如下图：

<image src='/images/git_checkout-track.png' width="60%" height="60%" />

这样我们就得到远程分支`b1`。**注意**：`--track`是git 2.6版本之后的参数，如果是旧版本的git，需要自己命名本地分支：

```git
git checkout b1 origin/b1
```

## 同步远程分支最新变动: git fetch和git pull

如果远程服务器的内容变更了之后如何得到最新的变动呢？常用的命令有`git pull`和`git fetch`两种。我们在远程服务器的`master`分支下添加`m2.txt`文件。那么我们首先用`git fetch`来演示效果：

```git
git fetch
```

我们可以得到如图的结果：

<image src='/images/git_fetch.png' width="60%" height="60%" />

然后查看本地`master`分支下的文件，发现并没有新增文件。这也就是说`git fetch`文件并没有真正获取远程仓库的数据，这条命令只是在更新远程分支的索引，包括commit id。而`git pull`会直接将数据同步到本地，并且会进行`merge`操作。所以一般来说用`git fetch`更加安全，因为可以手工地决定合并哪些内容。


# 从本地仓库到远程仓库

## 创建本地分支

我们首先创建一个远程仓库没有的本地分支`b2`。

```git
git checkout -b b2
```

## 将本地分支推送到远程服务器

我们将`b2`分支推送到远程服务器。

```git
git push origin b2:b2
```

## 将本地分支与远程分支推送关联

就是你当前分支执行`git push`默认的关联的远程分支。

```
git push --set-upstream origin hpzhao_resume
```

# 参考资料

+ [Git分支-远程分支](https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)
+ [git pull和git fetch区别](http://www.huxiusong.com/?p=365)