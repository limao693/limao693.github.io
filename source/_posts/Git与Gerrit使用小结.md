---
title: Git与Gerrit使用小结
date: 2019-01-15 14:22:16
tags:
---

[TOC]

### 切换分支报错，提示需要删除差异文件

git clean  -d  -fx ""
注： 

1. x ：表示删除忽略文件已经对git来说不识别的文件 
2. d: 删除未被添加到git的路径中的文件 
3. f: 强制执行

### git status提示.idea未同步。

Since you are locally ignoring files that are in the repository, you need to tell git that you are ignoring them。
​    git update-index --assume-unchanged <files to forget>

### 修改代码第一次提交

git checkout master                                               // 切换到master分支
git pull -r 或者 git fetch -p && git reset --hard origin/master   // 拉取最新代码，不建议使用git pull
git checkout -b branchname                                        // 新建分支branchname

### 修改代码

git add files 或 git add -A 或 git add -u 或 git add .            // 添加修改文件到暂存区     
git commit -m "commit msg"                                        // 本地提交，注意每次commit都会生成一个change-ID，即一个评审

git review                                                        // 提交到master分支
或
git push origin HEAD:refs/for/master                              // 推荐使用git review
或
git review r5_bugfix                                              // 提交到指定分支。r5_bugfix为分支名

##git add -A 提交所有变化
##git add -u 提交被修改（modified）和被删除（deleteed）文件，不包括新文件（new）
##git add . （git version 1.x）提交新文件（new）和被修改（modified）文件，不包括被删除（deleteed）文件

（git version 2.x）提交所有变化

### 添加评审人

2.2.1 使用Gerrit添加评审人
登录http://gerrit.zte.com.cn/，my --> changes 选择提交的评审单，在右侧的Reviewers 后添加评审人，等待评审结果
2.2.2 在提交评审的同时，设置评审人
git review --reviewers xxx@zte.com.cn

评审未通过，开发人员需要在本地修改代码，作为补丁提交到gerrit上

！！提醒：注意要使用commit --amend功能来修订你的提交，而非新增一个commit，新增commit会新生成一个评审单
场景一：如果本地目录已经是上次提交的代码，则可以直接修改代码，执行下面的命令提交补丁即可

git add files 或 git add -A 或 git add -u 或 git add .  
git commit --amend
git review 
场景二：如果本地目录不是上次提交的代码（比如：给以前提交的代码或者别人提交的代码打补丁），需要先获取指定的Change No对应的代码，然后修改代码，提交补丁

git review -d xx   // 重新下载之前提交过的patchset代码，并切换到新分支
修改代码
git add
git commit --amend
git review

以前整理的几个Gerrit使用场景：

开发1，建立分支a; 开发2，建立分支b

在开发过程中，a在开发到中间时，需要基于分支b的修改进行继续开发，这样就需要b提交一个review。

然后a基于这个review进行开发，操作流程如下：

brach_b_id： git review -l进行查看。

开发1在本地，git reiew -d brach_b_id ,然后 执行 git cherry-pick a，然后 git branch -D a ,最后 git checout -b a。

这样操作完成后，本地的a 就变成了基于 b的某个review的 branch了。

git进一步使用场景：上面场景中，如果，a的开发过程中，b又有修改，并且a需要基于b的新修改进行后续开发。

那么操作流程基本与上面一样：

git review -d b_1_id

git cherry-pick a

git brach -D a

git checkout -d a



不过此时更好理解的时另外一种操作：

git review -d b_1_id

git checkout a

git rebase -i  b_1_id

此时，会出现编辑界面，需要编辑本地变更的段，来应用(base)到b_1_id上。

此时，可以选择仅应用，a的变更，至于上个场景中已经base的b_id的变更，就可以丢弃了。
