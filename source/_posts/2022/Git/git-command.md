---
title: Git常用命令
date: 2022-03-17 23:06:12
categories:
 - Git
tags:
 - Git
cover: https://s3.bmp.ovh/imgs/2022/03/087ccecd2950bc30.jpeg
---
# 前言
使用Git 命令前，先了解几个关键词的概念

*   工作区（workSpace）
*   暂缓区（stage/index）
*   本地仓库（Repository）
*   远程仓库（Remote）

![](https://s3.bmp.ovh/imgs/2022/03/64aa14707f10bfb0.png)

# Git 常见命令速查表
![](https://ftp.bmp.ovh/imgs/2020/04/5991c03f6436c41b.jpg)

## 创建版本库
* 初始化本地git仓库（创建新仓库）
>` git init `

* 配置用户名
> `git config --global user.name "xxx" `

* 配置邮件
>`git config --global user.email "xxx@xxx.com" `

## 修改与提交
* 查看当前版本状态（是否修改）
>`git status`

* 添加修改的指定文件至index(暂缓区)
>`git add "xxx"`

* 添加修改的所有文件至index(暂缓区)
>`git add .`

*  提交文件 "xxx"为本次提交的说明"
>`git commit -m 'xxx' `

* 合并上一次提交
git commit -a -m 'xxx'   ==>>（等价于） git add .   git commit -m 'xxx'
>`git commit --amend -m 'xxx'`

* 将add和commit合为一步
>` git commit -am 'xxx' `

## 查看历史提交
* 显示提交日志
> `git log `

*  显示1行日志 -n为n行 git log -5
> `git log -1`

* 显示提交日志及相关变动文件
> `git log --stat `

* 可查看该文件以前每一次push的修改内容）  
> `git log -p + xxx(文件名)`

* 只查看该文件当前这一次的push内容）
> `git log - p -1 + xxx(文件名`

* 显示HEAD提交日志
>`git show HEAD `

* 显示某个提交的详细内容
>`git show dfb02e6e4f2f7b573337763e5c0013802e392818 `

* 显示已存在的tag
>`git tag`

* 增加v2.0的tag
>`git tag -a v2.0  -m 'xxx'  `

* 显示v2.0的日志及详细内容
>`git show v2.0 `

* 显示v2.0的日志
>`git log v2.0  `

* 显示所有未添加至index的变更
>`git diff `

* 显示所有已添加index但还未commit的变更
>`git diff --cached `

## 撤销
* 撤销 add 过的文件 xxx（指定文件名）
>`git reset HEAR "xxx" `

*  撤销上一次 add 过的所有文件
>`git reset HEAR `

* 放弃单个的文件修改
>`git checkout -- xxx`

*  放弃所有的文件修改
>`git checkout .`

## 分支与标签
* 显示本地所有分支
>`git branch`

* 显示远程、本地所有分支
>`git branch -a`

* 切换至指定分支下
>`git checkout "xxx"`

* 创建新的分支
>`git branch ”xxx“   xxx为新的分支名`

* 创建新的分支并切换至当前新建的分支下
>`git check -b "xxx"`

* 删除指定分支
>`git branch -d "xxx"`

* 强制删除指定分支
>`git branch -D "xxx"`

* 删除tag
>`git tag -d "xxx"`

* 清除本地缓存分支
>`git remote prune origin`

## 合并已衍合

* 合并远程master分支至当前分支
>`git merge origin/master `

* 合并提交ff44785404a8e的修改
>`git cherry-pick "ff44785404a8e"`

* 将当前分支push到远程master分支
>`git push origin master `

* 把所有本地tags推送到远程仓库
>`git push --tags`

* 获取所有远程分支（不更新本地分支）
>`git fetch `

*  获取所有原创分支并清除服务器上已删掉的分支
>`git fetch --prune `

* 获取远程分支master并merge到当前分支
>`git pull origin master `

* 重命名文件README为README2
>`git mv README README2`

* 将当前版本重置为HEAD（通常用于merge失败回退）
>`git reset --hard HEAD `

## 暂存
* 暂存当前修改，将所有至为HEAD状态
>`git stash `

*  查看所有暂存
>`git stash list`

* 恢复暂存文件
>`git stash pop `


## 操作远程
* 本地分支推送到远程仓库
>`git push origin master `

* -u选项指定一个默认主机
>`git push -u origin`

* 不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机
>`git push --all origin `
