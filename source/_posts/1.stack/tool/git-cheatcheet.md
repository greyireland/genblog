---
title: git cheatcheet
tags:
  - git
categories:
  - tool
date: 2018-09-19 16:12:32
---

# 简图
![](https://ws1.sinaimg.cn/large/e5320b2aly1fwy5q46d3lj215k10nabc.jpg)


# 常用命令
~~~
git add . //ga
git status //gs
git diff //gd
git commit -m "desc" //gcm
git push //gp
git checkout -b feature/comment //gcb
git pull origin master //gpom
~~~

## 偶尔用到
~~~
git push --set-upstream origin dev_2 
git stash
git log
git cherry-pick xxx
git init
git clone
~~~

### 很少用到
~~~
git config --global user.name "muName"
git config --global user.email "myEmail"
~~~

### GET NEW THING

#### 比对 diff

```
# 显示暂存区和工作区的代码差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD
git diff HEAD^ 比对当前内容和倒数第二次提交。

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

```

```
# 从暂存区移除一个文件
git rm file
# 从暂存区移除所有文件
git reset .
```

#### rebase 变基【为了提交好看】

#### gitnore
```
filename  //递归 忽略当前目录下所有包含此文件名的文件
dir/      //递归 忽略目录和子目录 
!filename //递归 不忽略文件名

```


### 存储
![](https://ws1.sinaimg.cn/large/e5320b2aly1g210qd00udj20l20h6do5.jpg)

- commit
- tree
- blob

差异存储，后面的可以引用前面的blob和tree