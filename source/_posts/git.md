---
title: git
date: 2021-12-14 18:38:15
tags:
---



### git init

> 初始化本地仓库，创建 `.git` 文件夹



### git add

> 将文件添加到暂存区



### git commit

> 将暂存区的文件提交到当前分支（本地仓库）

#### 常用命令

+ `-m <info>`：添加提交的说明



### git push

> 将当前分支的内容提交到远程分支



### git log

> 查询提交的日志（按时间近到远排序），不包括已经被删除的commit记录和reset操作

#### 常用命令

+ `--pretty=oneline`：只显示一行（id + 提交的信息）



### git reflog

> 显示所有的操作记录，包括提交和回退的操作



### git branch

> 查看本地所以分支

+ `-r`：查看所有远程分支
+ `-d <branch>`：删除指定branch分支
+ `--set-upstream-to=origin/dev`：将本地dev和远程dev链接



### git fetch

> 更新远程的所有分支到本地



### git reset

> 版本回退
>
> git的版本回退是将 hard 指针指向另一个版本，因此速度极快

#### 常用命令

+ `--hard <commit id>`：id为commit id，可以通过 `git log` 或者 `git reflog` 查看，不需要补全全部，git会自动去查找对应的id。
+ `HEAD <file>`：将暂存区的修改撤销，放回工作区



### git checkout

> 

#### 常用命令

+ `-- <file>`：把指定文件在工作区的修改全部撤销
+ `-b <branch>`：创建并切换分支



切换分支的原理：创建分支时创建一个新的指针指向当前分支的Head（如果指定commit id，指向对应的commit提交），切换分支就是将Head指向新的分支的指针



### git switch

> 切换、创建分支

#### 常用命令

+ `<branch>`：切换分支，不存在的分支会报错
+ `-c <branch>`：创建并切换分支
+ `-c <branch> <commit id>`：以一个commit创建切换分支



### git merge

#### 常用命令

+ `<branch>`：把branch分支合并到当前分支（需要处理可能存在的冲突），默认会使用 `Fast forward` （删除分支后，会丢掉分支的信息）模式
+ `--no-ff -m <info> <branch>`：强制禁用 `Fast forward`



### git stash

> 将工作区文件储藏
>
> 当前分支开发到一半，不想或者说没法提交，需要切换分支

#### 常用命令

+ `apply`：恢复stash储藏的文件，但是不删除stash的内容
+ `drop`：删除stash的内容
+ `pop`：apply + drop
+ `list`：查看所有的stash



### git cherry-pick

> 将一次提交所在的修改复制到当前分支

#### 常用命令

+ `<commit id>`



### git remote

> 操作本地和远程的的关系

#### 常见命令

+ `add origin <url>`：本地仓库与远程仓库绑定
+ `rm origin`

*origin：可以当成远程仓库的别名，使用链接太过冗长。*



### git rebase

> 将杂乱的 git log梳理成一条直线（谨慎使用）



### git tag

> 为commit打上标签，使得查找更加方便

#### 常用命令

+ `<name>`
+ `<name> <commit id>`：为指定的commit打上tag标签
+ `-d <name>`：删除特定的tag标签



### git show

> 查看标签的具体信息

#### 常用命令

+ `<tag name>`
