---
title: 使用 git 时的一些笔记
date: 2020-11-08
tags: [git]
categories: git
---

### Git笔记

#### Index

Get started by [creating a new file](https://github.com/woshixty/xty.github.io/new/main) or [uploading an existing file](https://github.com/woshixty/xty.github.io/upload). We recommend every repository include a [README](https://github.com/woshixty/xty.github.io/new/main?readme=1), [LICENSE](https://github.com/woshixty/xty.github.io/new/main?filename=LICENSE.md), and [.gitignore](https://github.com/woshixty/xty.github.io/new/main?filename=.gitignore).

#### create a new repository on the command line

```
echo "# xty.github.io" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:woshixty/xty.github.io.git
git push -u origin main     
```

#### push an existing repository from the command line

```
git remote add origin git@github.com:woshixty/xty.github.io.git
git branch -M main
git push -u origin main
```

#### import code from another repository

You can initialize this repository with code from a Subversion, Mercurial, or TFS project.



###### (记录使用Git时遇到的一些问题)

#### The current branch xty has no upstream branch.

> 情况概述：新建本地分支后将本地分支推送到远程库, 使用git pull 或者 git push 的时候报错

错误原因：本地分支和远程分支没有建立联系

git pull命令的作用是：取回远程主机某个分支的更新，再与本地的指定分支合并，它的完整格式稍稍有点复杂。

> 查看本地分支和远程分支的关联关系

```bash
git branch -vv
```

> 本地分支和远程分支建立联系,执行以下命令即可

```bash
git branch --set-upstream-to=origin/远程分支的名字 本地分支的名字
```



#### src refspec master does not match any解决办法

> 情况概述：我在GitHub新建一个仓库之后，在本地新建了一个文件夹，git init 并且git remote add origin git@github.com:woshixty/xty.github.io.git 之后，执行 git push -u origin main，发现不能提交

错误原因：目录中没有文件，空目录是不能提交上去的，新建一个README文件，再上传

执行以下命令，远程分支与本地分支建立联系，并提交到远程分支

```bash
git push --set-upstream origin master
```

