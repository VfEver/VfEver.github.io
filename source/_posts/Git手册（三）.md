---
title:  Git简明手册——分支与版本的故事
date: 2018-09-24 15:15:39
categories: 技术
tags:
      - Git
      - 教程
---
### 分支
上一节熟悉了一下git远程仓库相关的知识。远程仓库和本地仓库一样，保存了分支与版本的各种信息。那么什么是分支与版本呢？   
所谓分支，简单理解可以把分支比喻作为生长的树，多人协同开发过程中，每个人都在自己的分支上开发，也就是树的各个树枝。这就体现了主干和分支的概念，树的主干就是我们的master分支，分支就是我们基于主干或者其他分支创建出来的分支。
#### 查看分支
那么怎么可以查看当前git目录里面的所有分支呢？命令为git branch。我们可以在当前的git仓库目录下面执行git brach查看一下。
```java
$ git branch
* master
```
可以看到master和前面带着的*，首先*代表我们当前处理的分支，这个分支叫做master。我们一般称master这个分支为主干分支，也就是我们前面提到的树的主干。master分支作为开发的主干分支，其他分支都是基于主干checkout出来进行开发（当然也可以checkout其他分支）。这个master分支对应于我们上节提到的远程仓库里面的master分支，我们起名远程仓库为origin，则它就对应origin/master。git branch -v，添加-v可以查看分支的版本号和当前提交的注释。如下：
```java
$ git branch -v
* master 9ffb526 Merge branch 'master' of github.com:VfEver/rushgit
```
可以看到当前只有一个分支master，版本号前几位为9ffb526，提交的注释为“Merge branch 'master' of github.com:VfEver/rushgit”。
#### 创建分支
创建分支命令为git branch [name]。例如git branch dev，就创建了一个名字叫做dev的分支，此时我们再用git branch -v查看一下：
```java
$ git branch dev
$ git branch -v
  dev    9ffb526 Merge branch 'master' of github.com:VfEver/rushgit
* master 9ffb526 Merge branch 'master' of github.com:VfEver/rushgit
```
可以看到现在有了两个分支，一个主干分支master，一个基于master的dev分支。此时我们还处于master分支（星号在master），两个分支的版本号和注释都是一样的。   
接下来我们通过git checkout命令切换分支。
```java
$ git checkout dev
Switched to branch 'dev'
$ git branch
* dev
  master
```
此时我们就把我们工作区分支切换到了dev。dev工作区内容和master分支的内容完全一致。接下来我们执行一次提交。
```java
$ touch go.txt
$ git add go.txt
$ git commit -m "add go.txt"
```
再切回master分支，就会发现我们在dev分支的修改并没有影响到master分支。当前master分支目录下面并没有go.txt这个文件。
```java
$ git checkout master
Switched to branch 'master'
$ ls -l
total 2
-rw-r--r-- 1 zys 197609 137 Sep 24 01:53 README.md
-rw-r--r-- 1 zys 197609  42 Oct 29  2017 java9.txt
```
在当前目录下面，我们再介绍两个命令，merged和no-merged：
```java
$ git --merged
* master
$ git --no-merged
  dev
```
--merged是指当前分支合并过哪些分支，--no-merged是指还有哪些分支没有被合并。可以看到我们刚才创建的dev分支并没有合并到master。接下来我们尝试一下分支合并。
#### 合并分支
使用git merge [name]来讲name分支合并到当前分支。如下：
```java
$ git merge dev
Updating 9ffb526..f5da69a
Fast-forward
 go.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 go.txt
$ ls -l
total 3
-rw-r--r-- 1 zys 197609 137 Sep 24 01:53 README.md
-rw-r--r-- 1 zys 197609  12 Sep 24 16:51 go.txt
-rw-r--r-- 1 zys 197609  42 Oct 29  2017 java9.txt
```
此时就可以看到go.txt这个文件出现在了master分支。将dev分支合并到master分支后，go这个文件夹出现在了我们的工作仓库当中。如何查看整体的提交记录？此时又引出一个命令：
```java
git log --graph --pretty=oneline
* f5da69a6bdf6a44d75f7b4088b2d26d59abe497f add go.txt
*   9ffb52623011006575566736bc0c387107dae5c6 Merge branch 'master' of github.com:VfEver/rushgit
|\
| * 3125251a15943118724f9dce6302c7351444880e Update README.md
| * 9e7674cbd635f8e52e2434113e05479bf7281b9d Initial commit
* 68c86da5dc6372755fe7048847aed6cd9b8f001f add java9
```
通过这个命令我们可以根据图形化的提交记录来查看分支提交树，而且分支的信息也特别明确。
#### 删除分支
删除分支就很简单了，-b参数后面添加要删除的分支名称。如下：
```java
$ git branch -d dev
Deleted branch dev (was f5da69a).
$ git branch
* master
```
### 版本
所谓版本，可以简单理解为每次commit记录。每一次commit都会生成一个commit-id，这个commit-id代表一次提交的标识，前面我们看到的git log或者git branch -v也好，在后面都跟着一串字符串，包含数字和字母，这就是commit-id。我们可以称这些commit-id为不同的版本，后续我们的版本操作基本上就是根据commit-id来进行操作。
#### 版本回退
现在我们处于master分支，由上面的命令行可以看到一共有5次版本记录，每次版本记录后面都有本次commit注释。现在我们尝试将版本回退到上一个版本，也就是从f5da69回退到9ffb526。回退版本有两种方式，通过HEAD回退和通过commit-id回退。    
1. 通过HEAD回退
```java
git reset --hard HEAD^
HEAD is now at 9ffb526 Merge branch 'master' of github.com:VfEver/rushgit
```
HEAD指向头节点，也就是f5da69节点。^代表上一个版本，是针对f5da69版本的上一个版本。此时我们可以看到命令行里面的输出为HEAD现在指向了9ffb526这个节点。那版本库究竟发生了什么？我们仔细查看一下：
```java
$  ls -l
total 2
-rw-r--r-- 1 zys 197609 137 Sep 24 01:53 README.md
-rw-r--r-- 1 zys 197609  42 Oct 29  2017 java9.txt
```
可以发现从dev分支合并过来的go.txt消失了。此时我们就完成了从f5da69回退到9ffb526这个版本。命令中一个^代表上一个版本，两个^^代表上两个版本...以此类推。当我们想回退多个版本的时候，需要写多个^？这就引出了第二种版本回退的方法。
2. 通过commit-id回退
通过commit-id我们也可以进行版本回退。现在我们处在9ffb526这个commit-id，我们想要再回退一次commit，则可以将上述命令的HEAD部分可以修改为想要回退的版本号，也就是想要回退的commit-id。如下：
```java
$ git reset --hard 3125251
HEAD is now at 3125251 Update README.md
$ git log --pretty=oneline
3125251a15943118724f9dce6302c7351444880e Update README.md
9e7674cbd635f8e52e2434113e05479bf7281b9d Initial commit
```
此时我们可以看到，master分支的当前版本已经回退到Update README.md这个注释版本。再继续说明一下，通过commit-id方式不仅可以回退到上一个版本，还可以回退到任何版本，只要能拿到想要跳转的版本commit-id。所以这个命令不仅可以回退，还可以前进到某个历史版本。比如现在我们前进到最后的一个commit，它的commit-id为f5da69a，如下：
```java
$ git reset --hard f5da69a
HEAD is now at f5da69a add go.txt
$ git log --pretty=oneline
f5da69a6bdf6a44d75f7b4088b2d26d59abe497f add go.txt
9ffb52623011006575566736bc0c387107dae5c6 Merge branch 'master' of github.com:VfEver/rushgit
3125251a15943118724f9dce6302c7351444880e Update README.md
9e7674cbd635f8e52e2434113e05479bf7281b9d Initial commit
68c86da5dc6372755fe7048847aed6cd9b8f001f add java9
```
此时再看，所有的版本记录又都回来了。但是，当我们不知道我们想要会退的commit-id时候呢？
#### 版本查询
当我们不记得我们需要回退的commit-id时，可以通过git reflog命令查看我们所有的操作记录，如下：
```java
$ git reflog
f5da69a HEAD@{0}: reset: moving to f5da69a
3125251 HEAD@{1}: reset: moving to 3125251
9ffb526 HEAD@{2}: reset: moving to HEAD^
f5da69a HEAD@{3}: merge dev: Fast-forward
9ffb526 HEAD@{4}: checkout: moving from dev to master
f5da69a HEAD@{5}: commit: add go.txt
9ffb526 HEAD@{6}: checkout: moving from master to dev
9ffb526 HEAD@{7}: pull origin master: Merge made by the 'recursive' strategy.
68c86da HEAD@{8}: commit (initial): add java9
```
通过git reflog命令可以看到我们所有的操作记录，每个操作都有一个commit-id，代表一次提交记录，前面的十六进制的号码就是当前操作记录的commit-id，所以可以通过这些commit-id进行任意穿梭。
### 总结
这个小节主要学习了版本和分支的基本操作，分支使我们协同开发更加方便，版本控制使我们对版本掌控力度更大。至此，我们算是一只脚踏进了gay交友的大门。处理之外，git还有什么好玩的操作？如何让我们使用起来更加优雅、更加专业？git怎么控制版本？.git目录是为了好看？
