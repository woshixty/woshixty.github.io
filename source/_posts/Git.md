---
title: 使用 git 时的一些笔记
date: 2020-11-08
tags: [git]
categories: git
---

### Git笔记

#### 新建仓库时的文本

Get started by [creating a new file](https://github.com/woshixty/xty.github.io/new/main) or [uploading an existing file](https://github.com/woshixty/xty.github.io/upload). We recommend every repository include a [README](https://github.com/woshixty/xty.github.io/new/main?readme=1), [LICENSE](https://github.com/woshixty/xty.github.io/new/main?filename=LICENSE.md), and [.gitignore](https://github.com/woshixty/xty.github.io/new/main?filename=.gitignore).

### …or create a new repository on the command line

```
echo "# xty.github.io" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:woshixty/xty.github.io.git
git push -u origin main
                
```

### …or push an existing repository from the command line

```
git remote add origin git@github.com:woshixty/xty.github.io.git
git branch -M main
git push -u origin main
```

### …or import code from another repository

You can initialize this repository with code from a Subversion, Mercurial, or TFS project.



###### (记录使用Git时遇到的一些问题)

#### 本地分支和远程分支没有建立联系

> 新建本地分支后将本地分支推送到远程库, 使用git pull 或者 git push 的时候报错

```tex
git pull命令用于从另一个存储库或本地分支获取并集成(整合)。git pull命令的作用是：取回远程主机某个分支的更新，再与本地的指定分支合并，它的完整格式稍稍有点复杂。

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

是因为本地分支和远程分支没有建立联系  (使用git branch -vv  可以查看本地分支和远程分支的关联关系)  .根据命令行提示只需要执行以下命令即可
git branch --set-upstream-to=origin/远程分支的名字 本地分支的名字  
```
