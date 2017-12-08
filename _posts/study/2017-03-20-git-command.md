---
layout: post
title: GIT 命令
categories: [study]
description: GIT常用命令
keywords: GIT,版本控
author:     "wenhao"
catalog:    true
tags:
    - study
---

## GIT学习

 *以后版本控制需要使用Git了，但因以前一直使用的SVN因此需要学习，现在把常用命令总结一下*  

--------
常用shell命令  
> pwd 查看文件目录结构  
  ls  显示目录下所有文件  
  cd  切换目录  
  mkdir 创建目录  
  touch 创建文件    
  rm  删除文件  
  rm -r 删除目录  
  mv 重命名（mv 原文件名 新文件名）
  cp   拷贝


配置全局用户信息
```
git config --global user.name "XX"  
git config --global user.email "XX"  
```

初始化本地仓库
```
git init
```

查看仓库状态  
```
git status
```

文件添加到仓库  
```
git add readme.txt
```

文件提交到仓库  
```
git commit (git commit -m "注释")
```

查看修改  
```
git diff
```

查看历史记录  
```
git log
git reflog
```

版本回退  
```
git reset --hard HEAD^   (回退到上一个版本）
git reset --hard HEAD~10 (回退到上10个版本）
git reset --hard ******  (回退到特定版本 ** 为版本commit id 可以通过git log查看）
```

撤销修改  
```
git checkout -- file（撤销工作区的修改，回到最近commit或者add时的状态）
git reset HEAD file （撤销缓存区的修改，使重新放回工作区，就像没有add）
```

删除文件  
```
git rm file
```

添加远程仓库  
```
git remote add origin git@*******.git
git push -u origin master
git push origin master
```

从远程库克隆  
```
git clone <git路径>
git clone -b <分支名> <git路径>
```


创建分支  
```
git branch(查看所有分支）
git checkout -b dev (创建dev分支并切换）
git branch dev(创建dev分支）
git checkout dev（切换到dev分支）
git merge dev(合并dev分支到当前分支）
git branch -d dev(删除分支)
```
