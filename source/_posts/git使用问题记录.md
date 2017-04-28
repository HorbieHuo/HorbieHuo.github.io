---
title: git使用问题记录
date: 2017-04-28 11:00:01
tags:
---

1. git pull 失败 ,提示：fatal: refusing to merge unrelated histories
    *本地使用git init命令初始化后，并关联了远程仓库，本地和远程均有commit，首次推送出现的问题*
    解决方法： **git pull origin master --allow-unrelated-histories**

2. 本地仓库关联远程仓库
    命令：**git remote add origin remote-url**