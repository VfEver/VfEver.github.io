---
title: Hexo 常用命令介绍
date: 2017-10-28 17:48:25
categories: 技术
tags:
      - Hexo
      - 教程
description: Hexo常用命令介绍
---

## Hexo 简介
[Hexo](https://hexo.io/)是一个快速、简洁且高效的博客框架。Hexo使用[Markdown](https://daringfireball.net/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

利用Hexo和GitHub搭建[Next](（http://theme-next.iissnan.com/)主题博客系统请参照Next的[官方文档](http://theme-next.iissnan.com/getting-started.html)。

## Hexo命令
Hexo等一系列搭建完成之后，可以轻松的进行博客撰写和发表。以下是一些常用的命令操作。

### 新建博客
```
$ hexo new post "article title"
```
这样，就在主目录（对应git主目录）的 source/\_post 目录里面新建了一个“article title.md”的markdown文件。可以简写为：
```
$ hexo n "article title"
```

### 清理缓存
```
$ hexo clean
```
这个命令一般用于清理项目下的缓存，用于启动项目之前。

### 启动服务
```
$ hexo server
```
用于启动服务。这是什么意思呢？相当于你可以通过浏览器浏览你用markdown写的博客网站，当然是访问的本地环境。这就类似于启动tomcat访问项目。路由器的url路径填写localhost:4000。这样，就可以在本机调试博客网站。可以简写为：
```
$ hexo s
```
关闭服务则为Ctrl+C。


### 生成静态网页

```
$ hexo generate
```
用于将新建的markdown文件解析为静态网页。一般用于新博客文章发布且启动服务之前。可以简写为：
```
$ hexo g
```
监视文件变动：
```
$ hexo generate --watch
```

### 部署
```
$ hexo deploy
```
用于将完成的博客发布挂载到我们的GitHub上。
可以简写为：
```
$ hexo d
```

### 完成后部署
两个命令是相同的：
```
$ hexo generate --deploy
$ hexo deploy --generate
```
简写为：
```
$ hexo g -d
$ hexo d -g
```

## 结语
以上是利用hexo新建发表博客的常用指令。
