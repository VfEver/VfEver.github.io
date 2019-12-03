---
title: Git简明手册——再谈Git版本
date: 2018-09-26 22:07:28
categories: 技术
tags:
      - Git
      - 教程
---
### 版本回退
上节我们了解到版本回退相关的几个操作，回退到上一个版本`git reset --hard HEAD^`或者通过`git reset --hard commit-id`来跳转到相应的版本。同时可以通过`git reflog`查看操作的历史纪录和注释等信息。这里我们提到的版本跳转是通过本地版本仓库的版本覆盖暂存区和工作区。这就又回到了我们第一小节讲到的三个空间：工作区、暂存区、本地版本库，那么版本回退过程中，几个空间如何回退跳转？
#### 撤销工作区的修改
首先查看一下我们的版本历史纪录：
```java
$ git log --pretty=oneline
f5da69a6bdf6a44d75f7b4088b2d26d59abe497f add go.txt
9ffb52623011006575566736bc0c387107dae5c6 Merge branch 'master' of github.com:VfEver/rushgit
3125251a15943118724f9dce6302c7351444880e Update README.md
9e7674cbd635f8e52e2434113e05479bf7281b9d Initial commit
68c86da5dc6372755fe7048847aed6cd9b8f001f add java9
```
此刻我们修改go.txt文件，随便做一些修改。然后此时我们查看一下当前的状态：
```java
$ git status
On branch master
Changes not staged for commit:
 (use "git add <file>..." to update what will be committed)
 (use "git checkout -- <file>..." to discard changes in working directory)

       modified:   go.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
可以发现提示我们可以add到暂存区。如果我们想撤销刚才对go文件的修改怎么办？只是想要工作区的内容回退到和暂存区内容一致。此时我们使用`git checkout -- file`，file就是需要回退的文件名。
```java
$ git checkout -- go.txt
$  git status
On branch master
nothing to commit, working directory clean
```
其实git checkout就是将暂存区的内容覆盖掉工作区的内容。我们通过`git status`可以看到当前工作区处于干净的状态。打开go.txt文件一看发现刚才添加的内容已经消失了。
#### 撤销暂存区的修改
刚才演示了在工作区修改没有add到暂存区的版本回退，当我们使用了add命令将修改同步到了暂存区，再想回退上一个暂存区的版本。我们实际操作一把，同样修改go.txt，add到暂存区：
```java
$ git add go.txt
$ git status
On branch master
Changes to be committed:
 (use "git reset HEAD <file>..." to unstage)

       modified:   go.txt
```
我们add之后可以看到git给出了提示：use "git reset HEAD <file>..." to unstage。提示我们使用reset HEAD file可以取消保存。
```java
$  git reset HEAD go.txt
Unstaged changes after reset:
M       go.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   go.txt
```
此刻我们再查看当前的状态，发现git提示我们要么通过add 添加到暂存区来准备commit，要么再通过上面checkout的方式来丢弃本次在工作区的修改。又回到了上面的流程。其实此时就是我们修改了工作区的内容，但是处于未同步到暂存区的状态。
#### 撤销本地版本仓库的修改
那就是我们上一节提到的`git reset --hard HEAD^`或者`git reset --hard commit-id`。
### 总结
这一小节我们详细介绍了三个空间（工作区、暂存区、本地版本库）版本的修改方式。这使得我们的开发更加方便灵活。
