---
title: Git常用命令
date: 2024-08-11 23:09:40
categories:
- Version-Control
tags:
- Git
---

协作开发时一些常用的git命令。

<!--more-->

## Git 常用命令

### rebase

git变基到[branch]

`git rebase [branch]`

git变基后合并冲突

`git rebase --cotinue`

合并代码前先进行变基操作(避免多出一条和分支修改无关的commit)

`git pull --rebase`

git变基修改多次提交

`git rebase -i [commit-id]` 或 `git rebase -i HEAD~n`

git取消变基

`git rebase --abort`

### reset

git重置提交到[commit-id]，并强制推送

`git reset --hard [commit-id]]`

`git push -f`

### revert

git回退到某一版本并提交

`git revert [commit-id]`

`git commit -m [commit-id]`

### others

git获取远程分支内容

`git fetch -a  origin/[branch]`

本地remote分支和远程同步

`git remote prune origin`

将本次修改的内容合并到上一次提交中，并强制推送

`git commit -a --amend`

`git push -f`

## 参考文档

[Git官方中文文档](https://git-scm.com/docs/git/zh_HANS-CN)