---
title: git rebase的使用
categories: 
 - 技术
tags:
 - Java
 - git
---

#### 1.合并多个commit
在切出新分支开发一个功能时,可以会联系提交多次,逐步完善一个功能,我们在功能完成后,可以把同一功能的多个commit合并成一个,简化提交记录.
```
 playcrab@shenpeng  ~/code/JavaCode/MavenDemo   test ●  git add .
 playcrab@shenpeng  ~/code/JavaCode/MavenDemo   test ✚  git cm "x1"
[test 5f4c440] x1
 1 file changed, 1 insertion(+), 3 deletions(-)
 playcrab@shenpeng  ~/code/JavaCode/MavenDemo   test  git add .
 playcrab@shenpeng  ~/code/JavaCode/MavenDemo   test ✚  git cm "x2"
[test a3caaa5] x2
 1 file changed, 1 insertion(+)
 playcrab@shenpeng  ~/code/JavaCode/MavenDemo   test  git add .
 playcrab@shenpeng  ~/code/JavaCode/MavenDemo   test ✚  git cm "x3"
[test f934c01] x3
 1 file changed, 1 insertion(+)
 playcrab@shenpeng  ~/code/JavaCode/MavenDemo   test  git log
```
加入已经提交了三次`x1、x2、x3`,可以把他们合并成一次提交.
```
git rebase -i HEAD~3
```
```
  1 pick 5f4c440 x1
  2 pick a3caaa5 x2
  3 pick f934c01 x3
  4
  5 # Rebase 23c62de..f934c01 onto 23c62de (3 commands)
  6 #
  7 # Commands:
  8 # p, pick = use commit
  9 # r, reword = use commit, but edit the commit message
 10 # e, edit = use commit, but stop for amending
 11 # s, squash = use commit, but meld into previous commit
 12 # f, fixup = like "squash", but discard this commit's log message
 13 # x, exec = run command (the rest of the line) using shell
 14 # d, drop = remove commit
 15 #
 16 # These lines can be re-ordered; they are executed from top to bottom.
 17 #
 18 # If you remove a line here THAT COMMIT WILL BE LOST.
 19 #
 20 # However, if you remove everything, the rebase will be aborted.
 21 #
 22 # Note that empty commits are commented out
```
将x1保留即pick,x2和x3都挤压到x1,即squash
```
1 pick 5f4c440 x1
  2 s a3caaa5 x2
  3 s f934c01 x3
```
之后保存退出
再次查看记录
```
commit 8435c6fd88e2597e3cbcd88ea0417960b24fd82c (HEAD -> test)
Author: shenpeng <shenpeng@playcrab.com>
Date:   Thu Jun 4 18:02:08 2020 +0800

    x1

    x2

    x3

```
最后三次提交就合并成一条了.

#### 合并分支
我们从brNew上切分支,比如`brNew_home`来开发家园相关功能时,如果brNew上有了一些新的提交,需要更新到`brNew_home`上,如果直接merge brNew,那么就会多一次merge的commit.这时候就可以用rebase.
reabse会将`brNew_home`上的提交暂存起来,然后把brNew上的提交合并过来,然后一条条回复之前暂存的提交.这样,`brNew_home`上的提交就变成了一条线.
