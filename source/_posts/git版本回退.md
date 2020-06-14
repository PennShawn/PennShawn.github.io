---
title: git版本回退
categories: 
 - 技术
tags:
 - Java
 - git 
---


#### 撤销还没有提交到远程的commit
git reset --hard HEAD^;
git reset --soft HEAD^;
git reset --mixed HEAD^; 默认参数


##### --hard
--hard参数会把上一次提交的所以数据重置
```
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git reset --hard HEAD^
HEAD is now at 754eb01 add some commments
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git status
On branch master
```

##### --soft
--soft会将当前的的HEAD指向另一个commit,但是数据不会丢失,相当于修改了代码后add了但是没有commit.
```
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git reset --soft HEAD^
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master ✚  git status
On branch master
Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   src/main/java/thread/SynchronizedMethod.java

 playcrab@localhost  ~/code/JavaCode/MavenDemo   master ✚  git commit -m "add 2"
```
##### --mixed 默认参数
不加参数或者加--mixed相当于上一次的修改没有add 和commit,也不会丢失数据
```
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git reset HEAD^
Unstaged changes after reset:
M	src/main/java/thread/SynchronizedMethod.java
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master ●  git status
On branch master
Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   src/main/java/thread/SynchronizedMethod.java

no changes added to commit (use "git add" and/or "git commit -a")
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master ●  git add .
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master ✚  git commit -m "add test"
[master 2f581e7] add test
 1 file changed, 2 insertions(+)
```


#### 撤回远程仓库的提交
当一个错误的修改提交到了远程后,我们如果只是reset那一次提交,在远程仓库还是会有那次提交的记录.
```
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git reset --hard HEAD^
HEAD is now at 2f581e7 add test
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git status
On branch master
Your branch is behind 'origin/master' by 2 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git pull
```
这时候需要用到 `revert`参数,revert能够产生一次新的提交,来是错误的提交的那一次的代码回到修改前的版本.
```
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git revert 2f581e78efb914b004203e76e1917bd023693e7b
[master 656bd96] Revert "add test"
 1 file changed, 2 deletions(-)
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
 playcrab@localhost  ~/code/JavaCode/MavenDemo   master  git push
```
