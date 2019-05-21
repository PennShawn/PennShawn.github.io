---
title: git的一些操作
categories :
- 技术
tags :
- git
---

#### 要添加一个新的功能
1.本地新建分支 
```
git checkout  -b brMissionDel
```
2.修改完代码后add并commit然后
```
git push --set-upstream origin brMissionDel
```
3.测试通过后
切到这分支merge

删除本地
git branch -d brMissionDel

删除远程
git push origin :brMissionDel


##### 查看本地未提交的commit
` git cherry -v`
```
 playcrab@shenpeng  ~/code/JavaCode/shenpeng.github.io   master  git cherry -v
+ 0c1a2f367db2cf47476c0167cb0e5d3846903a2d add logic
+ 695246c3de14265ee7efe6423cf6736425648e88 add logic 2
 playcrab@shenpeng  ~/code/JavaCode/shenpeng.github.io   master  git  show 0c1a2f367db2cf47476c0167cb0e5d3846903a2d
```

##### git撤销合并
http://blog.psjay.com/posts/git-revert-merge-commit/
`git revert -m 1 e55d0543e96b22d9cecce2dcb7eeae1a9cb53edf `

合并时也可以`git merge --abort`

##### git stash
```
git stash
git stash list 
git stash pop
```

##### git 查看本地未提交commit
`git cherry -v`

