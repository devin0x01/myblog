---
title: "Git常用命令"
date: 2023-06-18T14:05:25+08:00
tags: ["Git"]
categories: []
draft: false
toc: true
---
# 1.git remote
```shell
# 关联远端仓库
git remote add origin git@github.com:git_username/repository_name.git
git remote remove origin

git remote -v
```
# 2.git branch
```shell
# 本地分支关联远程分支(目的是在执行git pull/push操作时就不需要指定对应的远程分支)
git branch --set-upstream-to=origin/master
git branch -u origin/master

# 切换分支
git checkout -b <new_branch>

# 查看所有分支
git branch -a

# 更改分支名称
git branch -m [old_branch] <new_branch>
```
# 3.git push
```shell
# 删除远程分支
git push origin --delete <branch>

# 同步到远程分支
git push -u origin <local_branch>:<remote_branch>
```
# 4.git diff
```shell
# 查看当前没有add的内容修改
git diff

# 查看已经add没有commit的改动
git diff --cached

# 查看当前没有add和commit的改动
git diff HEAD 或者
git status 再查看任意连个版本之间的改动 git diff <版本号码1> <版本号码2>

# 比较两个版本号src文件夹的差异
git diff <版本号码1> <版本号码2> src
```
# 5.代码回退
## 5.1 HEAD~与HEAD^
[What's the difference between HEAD^ and HEAD~ in Git? - Stack Overflow](https://stackoverflow.com/questions/2221658/whats-the-difference-between-head-and-head-in-git)  
[git在回退版本时HEAD~和HEAD^的作用和区别_git head^-CSDN博客](https://blog.csdn.net/albertsh/article/details/106448035)  

- Use`~`most of the time — to go back a number of generations, usually what you want
- Use`^`on merge commits — because they have two or more (immediate) parents

> Here is an illustration, by Jon Loeliger. Both commit nodes B and C are parents of commit node A. Parent commits are ordered left-to-right. (N.B. The git log --graph command displays history in the opposite order.)
```txt
G   H   I   J
 \ /     \ /
  D   E   F
   \  |  / \
    \ | /   |
     \|/    |
      B     C
       \   /
        \ /
         A

A =      = A^0
B = A^   = A^1     = A~1
C = A^2
D = A^^  = A^1^1   = A~2
E = B^2  = A^^2
F = B^3  = A^^3
G = A^^^ = A^1^1^1 = A~3
H = D^2  = B^^2    = A^^^2  = A~2^2
I = F^   = B^3^    = A^^3^
J = F^2  = B^3^2   = A^^3^2
```

## 5.2 git reset
```shell
# 撤销没有commit的修改
git checkout .

git reset --hard/mixed/soft origin/HEAD
git reset --hard/mixed/soft f52c633
```
# 6.git tag
```shell
# 查看标签
git tag

# 查看标签的版本信息
git show v1.0

# 打新标签
git tag v1.0
git tag v0.9 f52c633

# 删除标签
git tag -d v0.1

# 推送标签到远端
git push origin v1.0

# 一次性推送全部尚未推送到远程的本地标签
git push origin --tags

# 如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除，再从远端删除
git tag -d v0.9
git push origin :refs/tags/v0.9
```
# 7.git show
```shell
# 显示对应commit-id修改信息
git show f52c633
```
