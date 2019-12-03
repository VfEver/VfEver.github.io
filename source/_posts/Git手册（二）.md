---
title: Git简明手册——远程仓库
date: 2018-09-23 23:34:31
categories: 技术
tags:
      - Git
      - 教程
---
### Git远程仓库简介
上一节讲到了Git的本地开发方式，类似于本地一个版本仓库，存储在本机的磁盘中。对于自己开发当然可以直接搞，本地版本管理，本地提交等等。但是对于多人协同工作，就需要建立一个远程存储代码和版本历史的仓库，我们可以成这个仓库为远程仓库，存储我们代码的提交历史，维护代码变动。     
由于我们前一小结提到过，git是一个分布式的版本控制工具，也就是说，即使存在远程仓库，本地也会保留一个完整的版本记录。
#### 建立远程仓库
首先要有远程仓库，所以我们先去github上面手动创建一个远程仓库，创建细节不再描述，创建完成之后我们可以拿到对应这个git远程仓库的ssh链接：git@github.com:VfEver/rushgit.git。好了，第一步就简单的完成了。
#### 本地仓库与远程仓库连接
建立了远程仓库，那么如何让本地仓库与远程仓库建立连接呢？这个连接又是什么意思呢？  
连接指的就是本地与远程的一个纽带，一个相互关系。本地的修改可以提交到远程仓库，也可以从远程仓库同步代码到本地代码仓库当中。建立连接有两种方式：  
1. 本地还未初始化git仓库，可以直接git clone。如下：   
```java
$ git clone git@github.com:VfEver/rushgit.git
Cloning into 'rushgit'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
Checking connectivity... done.
```
此时相当于把远程仓库的所有东西全部copy到本地，或者说down到了本地。那么当前的这个目录就是本地仓库，并且和远程仓库的master分支（分支下一节介绍）一致。
2. 当本地已经存在一个仓库，此时只是想把本地仓库和远程仓库做一个关联，如下：
```java
$  git remote add origin git@github.com:VfEver/rushgit.git
```
git remote add [名称] [地址]   
这样就将本地的仓库与远程的分支建立了连接，我们将远程的仓库起名为origin，当然也可以起名为其他的名字，默认规范为origin，表明这个仓库为远程源仓库。我们创建仓库的时候初始化了一个README.md文档，本地是没有这个文件的，所以我们需要把远程仓库的东西同步到我们本地代码，此时就需要引入接下来的命令。
3. 将远程仓库变动同步到本地仓库   
将远程仓库变动同步到本地仓库，就是使用git pull一个命令，同时需要跟上远程仓库的名称和分支。上面建立关联的过程中我们使用了origin这个名称，分支一直使用master。所以命令以及输出如下：    
```java
$ git pull origin master
From github.com:VfEver/rushgit
 * branch            master     -> FETCH_HEAD
Merge made by the 'recursive' strategy.
 README.md | 3 +++
 1 file changed, 3 insertions(+)
 create mode 100644 README.md
```
此时我们可以看到pull一个动作将远程的README.md拉取到了本地仓库，并且作了一个merge动作，就是将本地和远程的文件在本地仓库做一个合并。在多人协作或者频繁更改某些文件的时候，是会出现冲突的。由于此次远程和本地的修改不是同一个文件所以并没有冲突。
4. 将本地仓库变动同步到远程仓库   
刚才我们将远程的变动拉到了本地仓库，那么如何将本地的修改变动同步到远程仓库呢？此时就需要一个推送的命令。push可以将本地的修改推送到远程。push的时候也需要我们将远程仓库名（origin）和分支（master）添加上。命令和输出如下：   
```java
$ git push origin master
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 537 bytes | 0 bytes/s, done.
Total 5 (delta 0), reused 0 (delta 0)
To git@github.com:VfEver/rushgit.git
   3125251..9ffb526  master -> master
```
通过Writing objects：100%可以看到本地的修改已经全部push到了远程master分支。此时我们再通过git log命令查看一下版本记录：
```java
$ git log --pretty=oneline
9ffb52623011006575566736bc0c387107dae5c6 Merge branch 'master' of github.com:VfEver/rushgit  --merge远程
3125251a15943118724f9dce6302c7351444880e Update README.md  --远程更新记录
9e7674cbd635f8e52e2434113e05479bf7281b9d Initial commit    --远程初始化记录
68c86da5dc6372755fe7048847aed6cd9b8f001f add java9         --本地commit记录
```
### 总结
至此，git两个重量级操作已经搞完了，pull拉取和push推送。git协同操作最基本的两个操作就是这两个。掌握了这两个命令，已经算是初步掌握了协同开发的基本技能，但是这也是两个最简单的两个操作。至于后面的分支管理策略、冲突解决，将会在下一小节继续描述。
