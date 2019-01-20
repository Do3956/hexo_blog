---
title: git多人协作流程
date: 2018-12-20 22:02:41

tags:
- git

categories:
- git

---

> git是一个管理代码的工具，具体的介绍及使用方法可以查看下面链接，在此不做叙述。
- https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001373962845513aefd77a99f4145f0a2c7a7ca057e7570000

> 本文主要讲述团队协作中，git的使用流程。

## 1. 每做一个新功能时，先从master拉一个新分支
`git checkout -b 你的名字/功能名称` 如 `git checkout -b ltj/test_git_add_branch`
## 2. 功能开发...提交代码到本地分支
## 3. 上传本地分支到远程分支
`git push origin ltj/test_git_add_branch remotes/origin/ltj/test_git_add_branch`

## 4. rebase，和master代码同步
（相当于把master合并到当前分支，但更友好）
`git fetch`

`git rebase origin/master `

如果有冲突，需要解决冲突，提交，之后再继续rebase

`git add .  ; git rebase --continue`

这个过程一直持续循环到没有冲突为止

将rebase后的代码推送到远程分支

`git push origin ltj/test_git_add_branch`

如果rebase一直报错，可以强行推送到远程，但不建议用

`git push origin ltj/test_git_add_branch -f`

## 5. 发出合并请求，等master合并该分支
到gitlab上进行页面操作

## 6. 过一段时间确认无误后
删除远程分支
`git push origin --delete ltj/test_git_add_branch`

删除本地分支
`git branch -D ltj/fix_paging`


## 可能遇到的问题：
- 合并冲突 https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840202368c74be33fbd884e71b570f2cc3c0d1dcf000
