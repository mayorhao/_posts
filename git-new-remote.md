---
title: git如何替换远程仓库
categories: git
tags: 
- git
- 笔记
---
### git如何替换远程仓库
我们经常会遇到这样的场景，在github上新起了一个项目。想将本地仓库推送上去，但是该仓库已经关联了别的远程仓库。这时，我么只需要:
```shell
git init //重新初始化git仓库
```
然后，可以选择用：
```shell
git remote set-url origin [url]

```
当然，我们也可以先把删除掉与原仓库的关联
```shell
git remote rm origin
git remote add origin [url]
```