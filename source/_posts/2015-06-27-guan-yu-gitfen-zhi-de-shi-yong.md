---
layout: post
title: "关于git分支的使用"
date: 2014-06-27 11:48:12 +0800
comments: true
categories: Git
---
学习记录，不正确的地方望指正！

<!--more-->
git branch 显示的是本地分支

git pull origin aRemoteBranch

git push origin aRemoteBranch

git commit 是提交到当前的本地分支


git remote显示远程服务器

git remote show aRemote显示指定的远程服务器（origin）上的远程分支


git checkout --track -b aLocalBranch aRemote/aRemoteBranch 创建一个新本地分支追踪远程分支（也可以直接 git checkout aRemoteBranch）


远程分支不可见，只能通过新建本地分支来追踪远程分支，本地分支和远程分支是两个不同的东西，也可以不同名。


checkout的时候会先检查本地分支，如果有则checkout到这个本地分支（不论它track哪个远程分支），如果没有则查看远程分支，如果有同名远程分支则新建一个本地同名分支并追踪（track）远程分支


git branch -d aLocalBranch  删除本地分支

git push ARemote :aRemoteBranch 删除远程分支

-d -> -D 强制删除


vim .git/config

可以查看当前的remote和已经track了remote分支的本地分支


可以建多个本地分支track同一个远程分支，删除本地分支对远程分支没有影响。 