# 背景
我们在本地使用git做版本管理时，如果想看看远端的git分支上发生了什么变化，习惯的做法是访问类似"https://github.com/" 的用户界面去查看最新的变化。
本文会讲述如何在本地使用git命令去查看远端分支的变化，降低操作成本。

# git remote branch
 https://git-scm.com/book/en/v2/Git-Branching-Remote-Branches
 
理论层面的知识，都在上面的文章中了，就不再细说，下面汇总下重点：

```
git br -a
```
<img width="406" alt="image" src="https://user-images.githubusercontent.com/3232275/221393763-7740baa9-7d66-4b15-9594-f6c02e04b35d.png">

可以看到所有的remote branch，即git中心仓库上创建的所有分支。

作为对比，local branch是你在本机创建的本地分支。

![image](https://user-images.githubusercontent.com/3232275/221393802-d8aa5960-8c61-48c8-8e59-16b7bf0149be.png)

在我们的电脑上，存储了两种分支：remote branch、local branch，这个概念很重要，remote branch是存储在我们电脑上的，而不是指代的远端git服务器上的分支。

```
git fetch
```
remote branch 不会自动更新。执行 git fetch命令，会把所有中心仓库上最新的提交，下载到本机。即会更新remote branch，这时并不会更新local branch。

```
git co origin/main
```
执行上述命令，切换到origin/main这个远端分支。这时候如果你的伙伴往main分支提交了新代码，需要执行git fetch，更新之后，就可以在本地查看中心仓库上main分支的最新代码了。

## 知识运用
本地clone了一个开源项目，默认是master分支，我想切换到该项目的2.3.1版本。该怎么快速切换呢？
```
git br -a
```
<img width="395" alt="image" src="https://user-images.githubusercontent.com/3232275/221394987-6ffa556b-732c-4009-b262-b1a8088ff8b5.png">

看起来origin/2.3.1 分支，对应的就是2.3.1版本。


```
git co -b 2.3.1 origin/2.3.1
```

于是执行上述命令，就ok了。它实际上做了几步操作：
1. 创建分支2.3.1，并自动追踪origin/2.3.1分支
2. checkout 到2.3.1分支




# Tracking Branches

工作中，我们可能使用过git pull命令。在master分支(先checkout master)上执行git pull命令，效果等同于：
```
git fetch origin master
git merge origin/master master
```
即先更新origin/master分支，再merge到local master分支，这样，就把远端master分支最新代码合并到本地了。

这里有个关键点：git为啥默认把origin/master分支合并到master分支呢？如果是其他分支（举例，远端有个distribute-storage分支），能默认实现类似效果吗？

这里其实是Tracking Branches 在发挥作用。

    If you’re on a tracking branch and type git pull, Git automatically knows which server to fetch from and which branch to merge in.

```
git fetch origin distribute-storage:distribute-storage
git br -vv
```
<img width="644" alt="image" src="https://user-images.githubusercontent.com/3232275/221394223-f8aaff7f-104c-4284-83e9-34b1e99a00bf.png">

通过git fetch命令，创建本地分支distribute-storage，这时候并没有自动跟踪origin/distribute-storage分支。

怎么实现自动跟踪呢。

```
git checkout -b local-distribute-storage origin/distribute-storage
git pull
```

checkout 之后，在中心仓库有一个新的提交，然后执行git pull命令，之后本地的local-distribute-storage就自动更新了。
因为我们本地的local-distribute-storage分支在跟踪origin/distribute-storage，执行pull origin命令，相当于：
```
git fetch origin distribute-storage 
git merge origin/distribute-storage local-distribute-storage
```


# 总结

本文介绍了 remote branch 和 track branch，用好它们，提升git版本管理的效率。

